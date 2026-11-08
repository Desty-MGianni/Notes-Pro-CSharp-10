---
title: Note Annexe au Chapitre 12
publish: true
---
# Chapitre 12: Délégués, événements et expressions Lambda –– Sous le Capot

>[!note] Cette note à été généré par Gemini

Ce document détaille l'architecture interne des délégués, la génération CIL des événements, ainsi que la gestion mémoire des fermetures (*closures*) et des classes d'affichage (*display classes*).

---

## 1. Structure Interne d'un Délégué (`System.MulticastDelegate`)

Lorsqu'un délégué est déclaré en C# (`public delegate void MonDelegue(int x);`), le compilateur génère une classe scellée (`sealed`) qui hérite implicitement de `System.MulticastDelegate` (qui hérite elle-même de `System.Delegate`).

### Champs internes du CLR
Chaque instance de délégué sur le tas managé contient trois champs majeurs :
* **`_target` (`object`)** : Référence vers l'instance de la classe contenant la méthode cible (vaut `null` s'il s'agit d'une méthode statique).
* **`_methodPtr` (`IntPtr`)** : Pointeur de fonction natif vers la méthode à exécuter.
* **`_invocationList` (`object[]` / `Delegate[]`)** : Tableau maintenant la liste ordonnée des délégués lors d'un enchaînement via `+=`.

### Comportement Multicast avec valeur de retour
Lorsqu'un délégué multicast possédant une valeur de retour (ex. `Func<int>`) est invoqué :
1. Le CLR énumère et exécute séquentiellement chaque délégué de `_invocationList`.
2. Chaque méthode s'exécute, mais la valeur de retour du délégué global est systématiquement écrasée par le résultat de la **dernière méthode exécutée**.

---

## 2. Traduction CIL des Événements (`event`)

La déclaration d'un événement `public event Action OnChangement;` ne crée pas directement une variable accessible publiquement. Le compilateur C# génère sous le capot :

1. **Un champ délégué privé de sous-jacent** : `private Action OnChangement;`.
2. **Deux accesseurs publics** :
   * `add_OnChangement(Action value)` : Appelé lors de l'abonnement `+=`. Il utilise `System.Delegate.Combine`.
   * `remove_OnChangement(Action value)` : Appelé lors du désabonnement `-=`. Il utilise `System.Delegate.Remove`.
3. **Sécurité Multithread (`Interlocked`)** : Les accesseurs générés entourent les appels `Delegate.Combine` et `Delegate.Remove` de la méthode thread-safe `System.Threading.Interlocked.CompareExchange` pour éviter les conditions de course (*race conditions*) lors des abonnements concurrents.

---

## 3. Fermetures (*Closures*) et Classes d'Affichage (*Display Classes*)

Lorsqu'une expression lambda ou une méthode anonyme référence une variable locale de la méthode englobante, le compilateur effectue une **capture de variable**.

### Le mécanisme de la *Display Class*
* Le compilateur génère une classe interne cachée (ex. `<>c__DisplayClass0_0`).
* La variable locale capturée est promue au rang de **champ public** de cette classe d'affichage.
* L'instance de la classe d'affichage est allouée sur le **tas managé (*heap*)**, ce qui prolonge la durée de vie de la variable au-delà de l'exécution de la méthode englobante.

### Le piège de la boucle `for`
Dans une boucle `for (int i = 0; i < 3; i++)`, une lambda qui capture `i` référence le champ unique de la classe d'affichage. À la fin de la boucle, le champ vaut `3`. Tous les délégués stockés liront la valeur courante `3` au moment de leur exécution.

### Optimisation : Lambdas Statiques (`static`)
Depuis C# 9.0, l'ajout du modificateur `static` à une expression lambda (`static x => x * 2`) interdit la capture de variables locales. Le compilateur garantit ainsi l'absence de classe d'affichage et évite toute allocation implicite sur le tas.

---
title: Note Annexe au Chapitre 9
publish: true
---
# Chapitre 9 : Durée de vie des objets et Garbage Collection — Sous le Capot

>[!note] Cette note à été généré par Gemini

Ce document détaille les mécanismes internes du CLR, de la mémoire managée et du Garbage Collector (GC) pour la gestion de la durée de vie des objets en C# (.NET 6 / C# 10), d'après *Pro C# 10 with .NET 6* d'Andrew Troelsen et *Pro .NET Memory Management* de Konrad Kokosa.

---

## 1. Le Tas Managé (*Managed Heap*) & Allocation `newobj`

* **Pointeur d'allocation contiguë (`Next Object Pointer`)** : Contrairement aux tas natifs (C/C++) qui gèrent des listes de blocs libres avec fragmentation, le tas managé alloue la mémoire de façon séquentielle. L'instruction CIL `newobj` déplace simplement un pointeur d'allocation, rendant l'allocation sur le tas aussi rapide qu'une allocation sur la pile (*stack*).
* **Générations d'objets (Gen 0, Gen 1, Gen 2 & LOH)** :
  * **Gen 0 & Gen 1 (Générations éphémères)** : Stockent les objets récents à courte durée de vie.
  * **Gen 2** : Héberge les objets à longue durée de vie ayant survécu à plusieurs collectes.
  * **Large Object Heap (LOH)** : Reçoit directement les objets de taille $\ge 85\ 000$ octets. Par défaut, le LOH est nettoyé par balayage (*sweep*) sans compactage automatique pour éviter d'importants déplacements mémoire.

---

## 2. Racines d'Application (*Application Roots*) & Algorithme Mark-and-Compact

1. **Identification des racines** : Le GC identifie les objets vivants (*reachable*) en inspectant :
   * Les registres CPU et variables locales sur la pile d'exécution.
   * Les membres statiques des classes chargées.
   * Les handles du GC (GCHandles).
2. **Phase de Marquage (*Marking Phase*)** : Le GC parcourt le graphe d'objets à partir des racines et positionne un bit de présence.
3. **Phase de Compactage (*Compacting Phase*)** : Les objets survivants sont déplacés pour former un bloc mémoire contigu, et tous les pointeurs de référence sont réalignés dynamiquement par le CLR.

---

## 3. Mécanique Interne de Finalisation (`Finalization List` vs `Freachable Queue`)

Lorsqu'un objet définit un finaliseur (`~MaClasse()`), le CLR lui applique un traitement spécial :

1. **Allocation & Registration** : Un pointeur vers l'objet est inscrit dans la **Finalization List** du CLR lors de son instanciation (ce qui emprunte un chemin d'allocation plus lent).
2. **Cycle 1 du GC** : Quand l'objet n'est plus accessible par les racines de l'application, le GC repère sa présence dans la *Finalization List*. Il ne libère pas sa mémoire, mais déplace sa référence vers la **Freachable Queue** (*Finalization Reachable*).
3. **Résurrection Temporaire** : Étant enraciné par la file *Freachable*, l'objet est considéré comme vivant et **promu à la génération supérieure** (ex. Gen 0 $\rightarrow$ Gen 1).
4. **Thread de Finalisation** : Un thread dédié de haute priorité du CLR parcourt la file *Freachable* et exécute la méthode `Finalize()` de chaque objet.
5. **Cycle 2 du GC** : L'objet n'étant plus présent dans la file *Freachable*, la collecte suivante libère enfin la mémoire managée.
   * **Conséquence** : La finalisation exige au minimum **deux cycles complets de Garbage Collection** pour restituer la mémoire au système.

---

## 4. Le Pattern de Libération (*Dispose Pattern*) & `GC.SuppressFinalize`

* **`IDisposable` & bloc `using`** : L'interface `IDisposable` permet une libération **déterministe** et immédiate des ressources non managées (fichiers, sockets, handles OS). Le compilateur rabaisse la syntaxe `using` en un bloc `try/finally` appelant `.Dispose()`.
* **Le rôle de `GC.SuppressFinalize(this)`** :
  * Lorsque la méthode publique `Dispose()` est appelée explicitement par le développeur, elle exécute `GC.SuppressFinalize(this)`.
  * Le CLR positionne immédiatement le bit `BIT_SBLK_FINALIZER_RUN` dans l'en-tête de l'objet (*Object Header*).
  * Lors des collectes ultérieures, le GC ignore la finalisation. L'objet **ne passe pas par la file `Freachable`** et sa mémoire est restituée dès le premier cycle de collecte.

---

## 5. Collecte Agressive des Racines (*Eager Root Collection*) & `GC.KeepAlive`

* **Inlining & Optimisation JIT (Mode Release)** : Le compilateur JIT ne lie pas la durée de vie d'une variable locale à la fin de son bloc lexical (`}`), mais à sa dernière utilisation réelle dans le code.
* **Résurrection / Destruction précoce** : Un objet peut être considéré comme mort et collecté par le GC **pendant qu'une de ses propres méthodes d'instance s'exécute encore**, si l'instruction courante n'utilise plus le pointeur `this`.
* **Rôle de `GC.KeepAlive(this)`** : Pour empêcher la finalisation prématurée d'une ressource native pendant un appel P/Invoke ou une méthode longue, `GC.KeepAlive(this)` force le JIT à maintenir la racine active jusqu'à cet appel.

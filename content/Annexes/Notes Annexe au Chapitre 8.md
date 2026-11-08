---
title: Note Annexe au Chapitre 8
publish: true
---
# Chapitre 8 : Implémentation des interfaces — Sous le Capot

>[!note] Cette note à été généré par Gemini

Ce document détaille les mécanismes internes au niveau du compilateur C#, du CIL et du CLR concernant les interfaces en C# 10 / .NET 6.

---

## 1. Méthodes d'interface par défaut (*Default Interface Implementation*)

Introduite avec C# 8.0, cette fonctionnalité modifie profondément la résolution des méthodes dans le CLR.

```csharp
public interface IVehicule
{
    void Klaxonner() => Console.WriteLine("Pouët !");
}
public class Voiture : IVehicule { }
```

* **Mécanique CIL & Accessibilité** : La méthode d'interface par défaut est compilée directement dans les métadonnées de l'interface. Elle n'est **pas injectée** dans la table des méthodes (*vtable*) de la classe `Voiture`.
* **Restriction au niveau du compilateur** : L'expression `v.Klaxonner()` (où `v` est de type `Voiture`) provoque l'erreur `CS1061`. Pour appeler l'implémentation par défaut, le compilateur exige un appel via le type de l'interface : `((IVehicule)v).Klaxonner()`.
* **Support Runtime (.NET Core 3.0+)** : Contrairement au simple sucre syntaxique, les méthodes d'interface par défaut ont nécessité des modifications du moteur d'exécution (CLR) pour gérer la résolution dynamique des tables d'interfaces (`itbl`). Elles ne sont donc pas supportées sur le legacy .NET Framework.

---

## 2. Transtypage des structures (`struct`) vers une interface et Boxing

Lorsqu'une structure de type valeur implémente une interface :

```csharp
public interface ICompteur { void Incrementer(); }
public struct CompteurStruct : ICompteur
{
    public int Valeur;
    public void Incrementer() { Valeur++; }
}
```

* **Boxing automatique sur le tas (*Heap*)** : L'affectation `ICompteur itf = c;` prend la structure `c` située sur la pile (*Stack*) et en crée une copie empaquetée (*boxed*) sur le tas managé.
* **Divergence d'état** : L'appel `itf.Incrementer()` exécute la méthode sur la copie empaquetée située sur le tas (qui passe à 2). La variable d'origine `c` sur la pile reste inchangée (valeur 1).
* **Évitement du Boxing via les contraintes génériques** : Pour appeler une méthode d'interface sur un `struct` sans provoquer de boxing, on utilise une méthode générique contrainte :
  ```csharp
  public void Traiter<T>(T compteur) where T : ICompteur
  {
      compteur.Incrementer(); // Pas de boxing ! Le JIT génère du code spécialisé.
  }
  ```

---

## 3. Implémentation explicite d'interface

Lorsqu'une classe implémente une interface de manière explicite (`void IInterface.Methode()`) :

* **Accessibilité CIL** : Le compilateur C# émet la méthode avec la visibilité `private` et le drapeau `final virtual` dans la *vtable* de la classe.
* **Isolation d'API** : La méthode est complètement masquée de l'API publique de la classe concrète et n'est accessible que lorsqu'elle est référencée via le type d'interface exact.

---

## 4. Performance : `vtable` vs `itbl` (*Interface Dispatch Table*)

* **Appel virtuel classique (`virtual`/`override`)** : Utilise un index fixe direct dans la table des méthodes virtuelles de la classe (*vtable*). L'appel est extrêmement rapide.
* **Appel d'interface (`callvirt` sur interface)** : Nécessite la consultation d'une table d'interfaces (*Interface Dispatch Table* ou `itbl`) pour localiser la méthode correspondant à l'interface dans l'instance concrète. Le JIT utilise une technique d'optimisation appelée *Inline Caching* (ou *Stubs Dispatch*) pour minimiser cet surcoût à l'exécution.

---
title: Note Annexe au Chapitre 6
publish: true
---
# Chapitre 6 : Héritage et Polymorphisme — Sous le Capot

>[!note] Cette note à été généré par Gemini

Ce document détaille les mécanismes internes du compilateur C#, du langage intermédiaire CIL (*Common Intermediate Language*) et du moteur d'exécution CLR (*Common Language Runtime*) abordés dans le Chapitre 6 de *Pro C# 10*.

---

## 1. Instructions CIL `call` vs `callvirt`

Lorsqu'une méthode est appelée en C#, le compilateur émet deux types d'instructions d'appel principales en CIL :

```csharp
public class Animal
{
    public void Manger() { } // Méthode non virtuelle
    public virtual void Crier() { } // Méthode virtuelle
}
```

* **`call`** :
  * Utilisé pour appeler des méthodes statiques, des méthodes de structures (`struct`), ou des méthodes de base via le mot-clé `base.MaMethode()`.
  * Ne vérifie pas la nullité de l'instance (`this`). Si l'instance est `null`, l'instruction s'exécute quand même jusqu'à ce qu'un champ de l'instance soit accédé.
  * Utilisé explicitement sur `base.MaMethode()` pour éviter une résolution virtuelle dynamique qui provoquerait une récursion infinie.

* **`callvirt`** :
  * Utilisé pour toutes les méthodes virtuelles et pour la quasi-totalité des méthodes d'instance non virtuelles générées par le compilateur C#.
  * **Vérification de nullité automatique** : Avant d'exécuter la méthode, le CLR vérifie si la référence `this` est `null`. Si c'est le cas, il lève immédiatement une `NullReferenceException`.
  * **Résolution dynamique (*vtable*)** : Pour les méthodes marquées `virtual`/`override`, le CLR parcourt la table des méthodes virtuelles (*vtable*) de l'instance réelle sur le tas à l'exécution.

---

## 2. Masquage (`new`) vs Redéfinition (`override`) & Drapeau `newslot`

Le CIL gère la redéfinition et le masquage des méthodes via des emplacements spécifiques (*slots*) dans la table des méthodes :

```csharp
public class Parent
{
    public virtual void Afficher() => Console.WriteLine("Parent");
}

public class Enfant : Parent
{
    public new void Afficher() => Console.WriteLine("Enfant");
}
```

* **Redéfinition avec `override`** :
  * Le compilateur conserve le même slot dans la *vtable* attribué par la classe parente.
  * La méthode de l'enfant remplace le pointeur de la méthode parente. L'appel est polymorphique quel que soit le type statique de la variable.

* **Masquage avec `new` (`newslot`)** :
  * Le compilateur attribue le drapeau CIL **`newslot`** à la méthode `Enfant.Afficher()`.
  * Cela crée une **nouvelle entrée indépendante** dans la table des méthodes de l'objet, sans écraser le slot de `Parent.Afficher()`.
  * Résultat : Si la variable est de type `Parent` (`Parent p = new Enfant();`), le CLR exécute le slot de `Parent`. Si la variable est de type `Enfant` (`Enfant e = e;`), le CLR exécute le nouveau slot de `Enfant`.

---

## 3. Implémentation Explicite d'Interface

Lorsqu'une classe implémente une interface de manière explicite :

```csharp
public interface IAffichable
{
    void Afficher();
}

public class Document : IAffichable
{
    void IAffichable.Afficher() => Console.WriteLine("Document");
}
```

1. **Accessibilité restreinte au compilateur** : Le compilateur C# marque la méthode générée comme `private` (ou `final virtual` au niveau CIL), empêchant l'accès direct via une variable de type `Document` (`doc.Afficher()` produit une erreur de compilation).
2. **Accès via le type de l'interface** : La méthode est accessible uniquement lorsque l'instance est référencée sous le type de l'interface (`((IAffichable)doc).Afficher()`).
3. **Résolution des conflits** : Ce mécanisme permet à une même classe d'implémenter deux interfaces possédant des méthodes de même signature sans ambiguïté.

---

## 4. Implémentation d'Interfaces par des `struct` & Boxing Automatique

Les types valeurs (`struct`) peuvent implémenter des interfaces en C#, mais cela comporte une contrainte mémoire importante :

```csharp
public struct Compteur : IIncrementable
{
    public int Valeur { get; set; }
    public void Incrementer() => Valeur++;
}

Compteur c = new Compteur();
IIncrementable i = c; // BOXING AUTOMATIQUE !
```

* **Le piège du Boxing sur le tas** : Une variable de type interface est obligatoirement un type référence (un pointeur vers un objet sur le tas).
* Lors de l'affectation `IIncrementable i = c;`, le CLR recopie la `struct` située sur la pile (*stack*) vers un nouvel objet boxed sur le tas (*heap*).
* **Conséquence sur la mutabilité** : Appeler `i.Incrementer()` modifie la copie boxed sur le tas, tandis que la structure originale `c` sur la pile reste inchangée !

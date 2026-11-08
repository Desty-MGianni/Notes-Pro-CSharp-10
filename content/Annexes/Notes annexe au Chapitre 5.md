---
title: Note Annexe au Chapitre 5
publish: true
---
# Chapitre 5 : Comprendre l'encapsulation — Sous le capot

>[!note] Cette note à été généré par Geminni

Ce document explore les mécanismes internes du runtime (CLR), du compilateur et du bytecode CIL (_Common Intermediate Language_) sous-jacents aux concepts d'encapsulation de C# 10 et .NET 6 traités dans le chapitre 5.

---

## 1. Propriétés en lecture seule à l'initialisation (Setters `init`) & `IsExternalInit`

Lorsque vous définissez une propriété avec un accesseur `init` :

```cs
public class Personne
{
    public string Prenom { get; init; }
    public string Nom { get; init; }
}

```

- **Signature CIL & Métadonnées** : Le compilateur génère une méthode setter standard (`set_Prenom`), mais la décore avec un modificateur requis : `modreq(System.Runtime.CompilerServices.IsExternalInit)`.
- **Contrôle du compilateur** : Le compilateur C# inspecte ce modificateur `modreq` et garantit que les affectations ne sont autorisées qu'au sein des initialiseurs d'objets ou dans le scope du constructeur (`.ctor`).
- **Exécution au Runtime** : À l'exécution, l'appel se déroule comme une méthode classique sans surcoût de verrouillage au runtime, tout en offrant une immuabilité stricte lors de la compilation.

---

## 2. Types `record`, égalité de valeur et code synthétisé

La déclaration d'un `record` positionnel :

```cs
public record Personne(string Nom, int Age);
```

Amène le compilateur C# à générer automatiquement une classe complète contenant :

1. **Champs sous-jacents & Propriétés** : Création de `<Nom>k__BakcingField` et `<Age>k__BackingField` 1. avec les drapeaux CIL `private initonly`, exposés via des propriétés auto-implémentées avec accesseurs `init`.
2. **Méthodes d'égalité de valeur** : Redéfinition de `Equals(object)`, implémentation de `IEquatable<Personne>`, - et surcharge des opérateurs `==` et `!=` pour effectuer une comparaison champ par champ basée sur la valeur plutôt que sur l'adresse mémoire (`object.ReferenceEquals`).
- **Déconstruction** : Synthèse d'une méthode `Deconstruct(out string Nom, out int Age)` permettant la déconstruction positionnelle (`var (nom, age) = personne;`).
- **Mutation non destructive (expressions `with`)** : Synthèse d'une méthode de clonage virtuelle (généralement `<Clone>$()` qui crée une copie superficielle (_shallow copy_) de l'instance avant d'y appliquer les modifications de propriétés.

---

## 3. Constructeurs statiques (`.cctor`) & `beforefieldinit`

Dans les métadonnées CIL, les constructeurs statiques sont nommés `.cctor` (_class constructor_).

- **Initialisation statique en ligne** : Si des champs statiques sont initialisés dès leur déclaration sans constructeur statique explicite :

```cs
public class CompteEpargne  
{  
    public static double TauxInteret = 0.04;  
}  
```

Le compilateur ajoute le drapeau `beforefieldinit` dans les métadonnées de la classe. Cela autorise le compilateur JIT à exécuter l'initialisation statique à n'importe quel moment précédant le premier accès à un champ statique.

- **Constructeur statique explicite** :

```cs
public class CompteEpargne  
{  
    public static double TauxInteret;  
    static CompteEpargne()  
    {  
        TauxInteret = 0.04;  
    }  
}  
```

Déclarer un constructeur `static` explicite supprime le drapeau `beforefieldinit`. Le CLR garantit alors que `.cctor` s'exécute _exactement_ juste avant le tout premier accès à un membre statique ou d'instance.

---

## 4. Littéraux `const` vs Champs `static readonly`

- **`const`** :
    - Doit pouvoir être déterminé dès la compilation.
    - Le compilateur intègre la valeur littérale directement dans les instructions IL appelantes (ex: `ldc.i4.s 50`), ce qui signifie qu'aucune mémoire n'est allouée au runtime pour ce symbole.
    - Les appels inter-assemblies intègrent la valeur littérale en ligne, ce qui peut provoquer des incohérences de versioning si la constante change sans recompiler les projets appelants.
- **`static readonly`** :
    - La valeur est évaluée dynamiquement au runtime (dans le `.cctor` ou l'initialiseur statique en ligne).
    - Alloué dans la région mémoire statique de l'AppDomain.
    - Pour les types références, `readonly` ne protège que le pointeur de référence lui-même, et non l'état interne de l'objet pointé sur le tas (_heap_).
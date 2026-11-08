---
title: Note Annexe au Chapitre 11
publish: true
---
# Chapitre 11: Fonctionnalité avancé de C# –– Sous le Capot

>[!note] Cette note à été généré par Gemini

Ce document détaille les mécanismes internes du CLR, des métadonnées CIL et du compilateur C# pour les fonctionnalités avancées du langage (indexeurs, surcharge d'opérateurs, méthodes d'extension, types anonymes et code non sécurisé).

---

## 1. Indexeurs (`this[...]`) et l'Attribut `[IndexerName]`

* **Traduction CIL** : En Common Intermediate Language (CIL), la notion d'indexeur n'existe pas en tant que concept natif. Le compilateur C# traduit un indexeur en une propriété spéciale nommée par défaut **`Item`**, générant les méthodes d'accès `get_Item(...)` et `set_Item(...)`.
* **Personnalisation du nom de métadonnées (`[IndexerName]`)** :
	  ```csharp
	  [System.Runtime.CompilerServices.IndexerName("Element")]
	  public string this[int index] => _data[index];
	  ```
	L'application de cet attribut force le compilateur à nommer la propriété `Element` (`get_Element` / `set_Element`) dans les métadonnées de l'assembly, assurant la compatibilité avec les langages .NET ne supportant pas la syntaxe entre crochets `[]`.

---

## 2. Surcharge d'Opérateurs et Conversions Personnalisées

Les opérateurs et les routines de conversion sont traduits par le compilateur C# sous forme de méthodes statiques spéciales avec des noms réservés par la spécification CLI :

| Synthaxe C#                                   | Méthode CIL générée                     |
| :-------------------------------------------- | :-------------------------------------- |
| `public static Type operator +(...)`          | `public static Type op_Addition(...)`   |
| `public static bool operator ==(...)`         | `public static bool op_Equality(...)`   |
| `public static implicit operator Target(...)` | `public static Target op_Implicit(...)` |
| `public static explicit operator Target(...)` | `public static Target op_Explicit(...)` |

* **Opérateurs symétriques** : Le compilateur C# impose que les opérateurs opposés (`==` et `!=`, `<` et `>`) soient surchargés par paires obligatoires afin de garantir la cohérence sémantique.

---

## 3. Méthodes d'Extension et Résolution d'Appel sur `null`

* **Abaissement par le compilateur (*Lowering*)** : Une méthode d'extension est une méthode statique ordinaire contenue dans une classe statique, décorée de l'attribut `[ExtensionAttribute]`.
* **Instruction CIL `call` (Statique)** :
  L'expression `monInstance.MaMethodeExtension()` est rabaissée en `MaClasseStatique.MaMethodeExtension(monInstance)`.
* **Comportement avec `null`** :
  Parce qu'il s'agit d'un appel statique (instruction `call` et non `callvirt`), le CLR **n'effectue aucune vérification de nullité** sur l'instance appelante avant d'entrer dans la méthode. L'accès à une méthode d'extension avec une référence `null` ne lève donc pas de `NullReferenceException` à l'appel.

---

## 4. Types Anonymes (*Anonymous Types*)

Lorsque vous instanciez un type anonyme :

```csharp
var person = new { Name = "Alice", Age = 30 };
```

Le compilateur C# génère à la volée une classe générique interne (`internal sealed class <>f__AnonymousType0<...>`):

1. **Propriétés Immuables** : Chaque propriété est générée avec un accesseur `init`-only et un champ sous-jacent `private readonly`.
2. **Égalité basée sur les valeurs (`Equals`)** : Le compilateur redéfinit `Equals(object)` et `GetHashCode()` pour comparer chaque propriété individuellement.
3. **Absence de surcharge de `==`** : **Attention** — Le compilateur ne surcharge pas les opérateurs `==` et `!=`. L'expression `a == b` effectue donc une comparaison d'égalité de référence (`object.ReferenceEquals`), renvoyant `false` pour deux instances anonymes distinctes sur le tas, même si toutes leurs propriétés sont identiques.

---

## 5. Code Non Sécurisé (`unsafe`) et Épinglage Mémoire (`fixed`)

* **Pointeurs bruts** : Le bloc `unsafe` permet l'utilisation directe de pointeurs bruts (`*`, `&`, `->`), désactivant la vérification de sécurité des accès mémoire par le CLR.
* **Épinglage (*Pinning*) avec `fixed`** :
  ```csharp
  fixed (int* p = &monTableau[0]) { ... }
  ```
  Pendant le déroulement du bloc `fixed`, le compilateur indique au Garbage Collector d'épingler l'objet en mémoire (en ajoutant un GC Handle de type *Pinned*). Cela empêche le GC de déplacer l'objet sur le tas managé lors des phases de compaction mémoire.

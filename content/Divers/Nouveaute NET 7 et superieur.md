---
title: "Nouveauté .NET 7 et supérieur"
publish: true
---

Les informations de cette note proviennent en partie de Gemini.

# .NET 7/ C# 11

## Littéraux de chaîne bruts

C’est sans doute la fonctionnalité préférée des développeurs. Elle permet d'écrire du texte multi-ligne ou du JSON sans avoir à échapper les guillemets ou les caractères spéciaux.

```cs
// Plus besoin de \" ou de \n !
string json = """
{
    "Model": "Hexagon",
    "Color": "Red",
    "Points": 6
}
""";
```

## Motifs de liste

**La correspondance de motif s'est étendu aux tableaux et aux listes**. Vous pouvez maintenant tester la structure d'une collection très facilement.

```cs
int[] numbers = { 1, 2, 3 };
if (numbers is [1, 2, 3]) // Vérifie le contenu exact
{
Console.WriteLine("C'est la séquence 1-2-3");
}
if (numbers is [1, ..]) // Vérifie si ça commence par 1
{
Console.WriteLine("Commence par 1, suivi de n'importe quoi");
}
```

## Les membres requis

Avant, pour forcer l'initialisation d'une propriété, il fallait passer par un constructeur complexe. Désormais, le mot-clé `required` oblige l'utilisateur à remplir la propriété lors de la création de l'objet.

```cs
public class Car
{
    public required string PetName { get; init; } // Obligatoire !
    public int MaxSpeed { get; set; }
}

// Erreur de compilation si on oublie PetName :
var myCar = new Car { MaxSpeed = 200 }; 
```

## Les Attributs génériques

C'est une avancée technique pour les frameworks. Auparavant, vous deviez passer un `Type` à un attribut via `typeof()`. Maintenant, vous pouvez utiliser la syntaxe générique `<T>`.

```cs
// Ancien style : [MyAttribute(typeof(string))]
// Nouveau style C# 11 :
[MyAttribute<string>]
public void MyMethod() { }
```

## Les chaînes littérales UTF-8

Pour les performances web (très utile en 2026), vous pouvez déclarer des chaînes directement en format UTF-8 (tableau de bytes) en ajoutant le suffixe `u8`

```cs
ReadOnlySpan<byte> data = "Hello"u8;
```

## Les structures automatique par défaut (expliqué dans le livre)

Dans les anciennes versions, vous deviez initialiser **chaque** champ d'une `struct` dans le constructeur. En C# 11, le compilateur initialise automatiquement à leur valeur par défaut les champs que vous oubliez.

# .NET 8 / C# 12

## Les constructeurs primaires (expliqué dans le livre)

Voir dans le livre (expliqué pour les structures dans [[Chapitre 4#Le constructeur primaires de structure (Nouveauté C 12)|Chapitre 4]] mais présent dans tout le livre).

## Expressions de collection (expliqué dans le livre)

C'est une unification de la syntaxe. Au lieu de jongler entre `new int[] { }`, `new List<int> { }` ou `stackalloc`, vous utilisez une syntaxe unique avec des crochets `[]`.

```cs
int[] a = [1, 2, 3];
List<string> b = ["un", "deux", "trois"];
Span<char> c = ['a', 'b', 'c'];

// Le "Spread Operator" (..) pour combiner des listes
int[] combined = [..a, 4, 5, 6]; // Résultat: [1, 2, 3, 4, 5, 6]
```

## Alias pour n'importe quel type

> Même fonctionnalité que TypeScript

Vous pouviez déjà créer des alias pour les classes, mais C# 12 permet de le faire pour **tous** les types, y compris les tuples, les pointeurs ou les tableaux.

```cs
using Point = (int x, int y); // Alias pour un tuple
using GradeList = int[];      // Alias pour un tableau

Point p = (10, 20);
```

## Paramètres par défaut pour lambdas

Tout comme pour les méthodes classiques, vous pouvez maintenant donner une valeur par défaut aux paramètres d'une fonction anonyme (lambda).

```cs
var Incremente = (int value, int step = 1) => value + step;

Console.WriteLine(Incremente(5));    // Affiche 6
Console.WriteLine(Incremente(5, 10)); // Affiche 15
```

## Tableaux en ligne

C'est une fonctionnalité avancée pour la performance. Elle permet de créer un tableau de taille fixe directement dans une `struct`. C'est principalement utilisé par les développeurs de bibliothèques système pour éviter les allocations mémoire.

# .NET 9 / C# 13

## L'évolution de `params` (Plus que des tableaux)

C'est le changement le plus pratique au quotidien. Avant C# 13, le mot-clé `params` ne fonctionnait qu'avec les **tableaux** (`T[]`).

- **Maintenant :** Vous pouvez l'utiliser avec n'importe quel type de collection supporté par les expressions de collection (C# 12), comme `ReadOnlySpan<T>`, `IEnumerable<T>`, ou `List<T>`.
- **Bénéfice :** Cela évite des allocations de mémoire inutiles sur le "tas" (heap) si vous utilisez des `Span`.

```cs
// Valide en C# 13
void PrintNames(params ReadOnlySpan<string> names) 
{
    foreach (var name in names) Console.WriteLine(name);
}

PrintNames("Alice", "Bob", "Charlie"); // Très performant !
```

## Le nouvel objet `Lock`

Pendant 20 ans, on utilisait `lock(unObjet)`. C# 13 introduit un type dédié : `System.Threading.Lock`.

- **Pourquoi ?** L'ancien `lock` reposait sur des objets de synchronisation lourds. Le nouveau type `Lock` est beaucoup plus rapide et offre une syntaxe plus propre avec le mot-clé `using`.

```cs
private readonly Lock _myLock = new();

void DoWork()
{
    lock (_myLock) // Utilise le nouveau mécanisme optimisé
    {
        // Code sécurisé
    }
}
```

## Séquence d'échappement `\e`

Une petite nouveauté qui va plaire à l'utilisateur de **Neovim** que vous êtes !

- **Détail :** Jusqu'ici, pour représenter le caractère "Escape" (ESC), il fallait utiliser son code Unicode `\u001b`. C# 13 introduit le raccourci `\e`.
- **Utilité :** Très pratique pour envoyer des codes de couleur ou des commandes au terminal.

## Améliorations du "Method Group Natural Type"

Parfois, le compilateur C# avait du mal à deviner le type d'une méthode que vous passiez en paramètre (comme dans les expressions LINQ).

- **Changement :** C# 13 a affiné la manière dont il déduit le type. Le code "casse" moins souvent lors de l'utilisation de délégués ou de fonctions anonymes complexes.

## Propriétés et Indexeurs Partiels

Le mot-clé `partial` (que vous connaissez pour les classes) s'étend maintenant aux propriétés et aux indexeurs.

- **Utilité :** C'est surtout pour les **générateurs de code** (Source Generators). Un générateur peut définir une partie de la propriété, et vous définissez l'autre.

# .NET 10 / C# 14

## Les *files based apps*

**Avec .NET 10, la nécessité d'un projet n'est plus nécessaire. Il sera possible d'avoir un fichier *.cs* qui fonctionnera comme un exécutable.** (comme pour le Python et JavaScript)


Pour ce faire, on crée le programme, on entre quelque chose dedans. la commande `dotnet run nomDuFichier` fera fonctionné le programme  fo f fera fonctionné le programme.

`--` permettra de séparer les arguments des options

### Intégrer d'autres fichiers dans

Importer un package Nuget:
[Lien Microsoft](https://learn.microsoft.com/en-us/dotnet/core/sdk/file-based-apps)

Inclure des fichiers (classes, struct, ...) *.cs* dans un script .NET (Toujours en Preview de .NET 11, mais sera intégré dans .NET 10)
[Lien video YouTube sur le sujet](https://www.youtube.com/watch?v=VLXXXYQ8SEA)

### Sur les systèmes `Unix` (`Linux` et `MacOS`)

On peut maintenant créer des programmes exécutables de la même manière qu'un script `bash` !!!!!!

```cs
#!/usr/bin/env dotnet

Console.WriteLine("Hello, world!");
```

On peut ensuite utilise la commande `chmod +x` pour ajouté la permission d'être exécuté.

On peut rentré des arguments à la fonctions comme illustré [[Chapitre 3#Traitement des arguments de ligne de commande (MaJ C 9.0)|ici dans le livre]].

## L'assignation Null-conditionnelle (`?.=`)
Jusqu'à la version 13, C# permettait d'utiliser `?.` pour **lire** une valeur ou **appeler** une méthode, mais **pas pour l'assignation directe** à gauche d'un signe égal.

- **Lecture** (Autorisée depuis C# 6) :  `string name = customer?.Name;`
- **Appel** (Autorisée depuis C# 6) : `customer?.Reset();`
- **Assignation** (INTERDITE avant C# 14) : `customer?.Name = "Joe";`

**Désormais, l'assignation null-conditionnelle est valide à gauche de l'opérateur `=`.**

## Le mot-clé `field` (Field-backed properties)

Vous pouvez enfin accéder au champ de sauvegarde (_backing field_) généré par le compilateur sans le déclarer manuellement.

```cs
public string Name { 
    get => field; 
    set => field = value ?? throw new ArgumentNullException(); 
}
```

## Les Extension Members complets

Les extensions ne sont plus limitées aux méthodes. Vous pouvez désormais créer des **propriétés d'extension**, des opérateurs d'extension et des membres statiques d'extension.

## Paramètres lambda simples avec modificateurs

Vous pouvez ajouter des modificateurs de paramètres, tels que `scoped`, `ref`, `in`, `out` ou `ref` `readonly`, aux paramètres d'expression lambda sans spécifier le type de paramètre :

```cs
delegate bool TryParse<T>(string text, out T result);
// ...
TryParse<int> parse1 = (text, out result) => Int32.TryParse(text, out result);
```

Auparavant, l'ajout de modificateurs n'était autorisé que si les déclarations de paramètres incluaient leurs types. La déclaration précédente exigeait que tous les paramètres soient typés.

```cs
TryParse<int> parse2 = (string text, out int result) => Int32.TryParse(text, out result);
```

Le modificateur `params` exige toujours une liste de paramètres explicitement typée.

Pour en savoir plus sur ces changements, consultez l'article sur les expressions lambda dans la [documentation](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/lambda-expressions#input-parameters-of-a-lambda-expression).


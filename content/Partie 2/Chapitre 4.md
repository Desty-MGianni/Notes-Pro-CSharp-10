---
title: "Chapitre 4: Principes Fondamentaux de la Programmation - Partie 2"
publish: true
---

# <big><big><big><b><font color =green>Principes Fondamentaux de la Programmation C#, partie 2</font></b></big></big></big>

Ce chapitre reprend là où le [[Chapitre 3#Résumé du chapitre|Chapitre 3]] s'est arrêté et complète votre étude des aspects fondamentaux du langage de programmation C#. Vous commencerez par **étudier les détails de la manipulation des tableaux à l'aide de la syntaxe C# et découvrirez les fonctionnalités contenues dans le type de classe `System.Array` associé.**

Ensuite, vous examinerez divers détails concernant la **construction des méthodes C#**, ==en explorant les mots-clés `out`, `ref` et` params`==. Au cours de cette étape, vous examinerez également le rôle des ==paramètres facultatifs et nommés==. Je termine la discussion sur les méthodes en abordant la ==surcharge des méthodes==.

Ce chapitre aborde ensuite la **construction des types d'énumération et de structure, y compris un examen détaillé de la distinction entre un *type de valeur* et un *type de référence*. Ce chapitre se termine par l' examen du rôle des types de données *annulbles* et des opérateurs associés.

Une fois ce chapitre terminé, vous serez parfaitement prêt à découvrir les capacités *orientées objet de C#*, à partir du [[Chapitre 5]].

# Comprendre les Tableaux C#

Comme vous le savez sans doute déjà, **un tableau est un ensemble d'éléments de données auxquels on accède à l'aide d'un index numérique.** Plus précisément, ==un tableau est un ensemble de points de données contigus du même type (un tableau de`int`, un tableau de `SportsCar`, etc.)==. La déclaration, le remplissage et l'accès à un tableau avec C# sont tous assez simples. Pour illustrer cela, créez un nouveau projet d'application console nommé *FunWithArrays* qui contient une méthode d'aide nommée `SimpleArrays()` comme suit :

>Chaque chapitre possède sa solution, avec dedans tout les projets du chapitre. Pour lancer tout les projets du chapitre, utilisez [[Launcher.cs]]

```cs
Console.Title = "Fun with Arrays";
Console.WriteLine("***** Fun with Arrays *****");
SimpleArrays();
Console.ReadLine();

static void SimpleArrays()
{
	 Console.WriteLine("==== Simple Array Creation");
	
    // Créé un tableau de 3 entiers.
    int[] myInts = new int[3];
	
    // Crée un tableau de 100 object chaînes, indexé de 0 à 99.
    string[] booksOnDotNet = new string[100];
    Console.WriteLine();
}

```

Examinez attentivement les commentaires du code précédent. **Lorsque vous déclarez un tableau C# à l'aide de cette syntaxe, le nombre utilisé dans la déclaration du tableau représente le nombre total d'éléments, et non la limite supérieure**. Notez également que ==la limite inférieure d'un tableau commence toujours à 0==. Ainsi, lorsque vous écrivez ```int[] myInts = new int[3]```j, vous obtenez un tableau contenant trois éléments, indexés aux positions `0`, `1` et `2`.

Après avoir défini une variable de tableau, vous pouvez remplir les éléments index par index, comme indiqué ici dans la méthode `SimpleArrays()` mise à jour :

```cs
static void SimpleArrays()
{
    Console.WriteLine("==== Simple Array Creation");

    // Créé et rempli un tableau de 3 entiers
    int[] myInts = new int[3];
    myInts[0] = 100;
    myInts[1] = 200;
    myInts[2] = 300;

    // Maintenant on affiche chaque valeurs.
    foreach (int i in myInts)
    {
        Console.WriteLine(i);
    }
    Console.WriteLine();
}
```

>[!Info:]- Valeurs par défaut d'un tableau 
>**Sachez que si vous déclarez un tableau mais que vous ne remplissez pas explicitement chaque index, chaque élément sera défini à la valeur par défaut du type de données** (par exemple, un tableau de `bool`s sera défini à `false` ou un tableau d'`int`s sera défini à $0$.

## Examen de la syntaxe d'initialisation des tableaux C# 

En plus de remplir un tableau élément par élément, vous pouvez remplir les éléments d'un tableau à l'aide de la syntaxe d'initialisation des tableaux C#. Pour ce faire, **spécifiez chaque élément du tableau entre accolades (`{}`). Cette syntaxe peut être utile lorsque vous créez un tableau de taille connue et que vous souhaitez spécifier rapidement les valeurs initiales**. Par exemple, considérez les déclarations de tableaux alternatives suivantes :

```cs
static void ArrayInitialization()
{
    Console.WriteLine("==== Array Initialization ====");

    // Syntaxe d'initialisation de tableau en utilisant le mot clé new.
    string[] stringArray = new string[] { "one", "two", "three" };
    Console.WriteLine($"stringArry has {stringArray.Length} elements");
    
    // Syntaxe d'initialisation de tableau sans utilisé le mot clé new.
    bool[] boolArray = { false, false, true };
    Console.WriteLine($"{nameof(boolArray)} has {boolArray.Length} elements");

    // Syntaxe d'initialisation de tableau avec le mot clé new et une taille.
    int[] intArray = new int[4] { 20, 22, 23, 0 };
    Console.WriteLine($"{nameof(intArray)} has {intArray.Length} elements.");

    // Syntaxe d'initialisation de tableau entre crochets (C# 12)
    double[] doubleArray = [3.1, 34.5, 23.6];
    Console.WriteLine($"{nameof(doubleArray)} has {doubleArray.Length} elements");
}
```

**Quand on utilise les accolades `{}`, il n'est pas nécessaire de spécifié la taille du tableau** (vu quand on a construit la variable `stringArray`). **Notez aussi que le mot-clé `new` est optionnel.**

Dans le cas de la déclaration `intArray`, rappelez-vous à nouveau que la valeur numérique spécifiée représente le nombre d'éléments dans le tableau, et non la valeur de la limite supérieure. ==S'il y a une incompatibilité entre la taille déclarée et le nombre d'éléments initialisés== (que vous ayez trop ou trop peu d'initialiseurs), ==vous obtenez une erreur de compilation==. Voici un exemple :

```cs
// OUPS! décalage entre la tailles et les éléments!
int[] intArray = new int[2] { 20, 22, 23, 0 };
```

## Comprendre les tableaux locaux implicitement typés

Au [[Chapitre 3#Comprendre les variables locales implicitement typées|Chapitre 3]], vous avez découvert les variables locales implicitement typées. **Rappelez-vous que le mot-clé `var` vous permet de définir une variable dont le type sous-jacent est déterminé par le compilateur**. De la même manière, ==le mot-clé `var` peut être utilisé pour définir des tableaux locaux implicitement typés==. Grâce à cette technique, vous pouvez allouer une nouvelle variable de tableau sans spécifier le type contenu dans le tableau lui-même (==notez que vous devez utiliser le mot-clé `new` lorsque vous utilisez cette approche==).

```cs
static void DeclareImplecitArrays()
{
    Console.WriteLine("==== Implicit Array Initialization");

    // a est réellement un int[]
    var a = new[] { 1, 10, 100, 1000 };
    Console.WriteLine($"{nameof(a)} is a: {a.ToString()}"); 

    // b est réellement un double[]
    var b = new[] { 1, 1.5, 2, 2.5 };
    Console.WriteLine($"{nameof(b)} is a: {b}");

    // c est réellement un string[]
    var c = new[] { "hello", null, "world" };
    Console.WriteLine($"{nameof(c)} is a: {c}");
    Console.WriteLine();

}
```

>[!tip] Petite simplification
Depuis C# 6.0, on peut retirer l'appel à `ToString()` pour les membres. Surtout en C# 14! l'appel à `ToString()` est moins performant.

Bien sûr, tout comme lorsque vous allouez un tableau à l'aide de la syntaxe C# explicite, **les éléments de la liste d'initialisation du tableau doivent être du même type sous-jacent** (par exemple, tous des `int`s, tous des `string`s ou toutes des `SportsCar`s). **Contrairement à ce que vous pourriez attendre, un tableau local de type implicite n'est pas défini par défaut sur `System.Object` ; ainsi, le code suivant génère une erreur de compilation :**

```cs
// Erreur! Types mélangés!
var d = new[] {1, "one", 2, "two", false}
```

## Définition d'un tableau d'objets

Dans la plupart des cas, lorsque vous définissez un tableau, vous le faites en ==spécifiant le type explicite d'élément qui peut se trouver dans la variable du tableau==. Bien que cela semble assez simple, il y a une particularité notable. Comme vous le comprendrez au [[Chapitre 6|Chapitre 6]]**, `System.Object` est la classe de base ultime de tous les types (*y compris les types de données fondamentaux*) dans le système de types .NET Core**. Compte tenu de cela, si vous deviez définir un tableau de types de données `System.Object`, les sous-éléments pourraient être n'importe quoi. Considérez la méthode `ArrayOfObjects()` suivante :

```cs
static void ArrayOfObjects()
{
    Console.WriteLine("==== Array of Objects ====");
    // Un tableau d'objets peut Être n'importe quoi.
    object[] myObjects =
    {
        10,
        false,
        new DateTime(1997, 10, 23),
        "Form & Void",
    };

    foreach (object obj in myObjects)
    {
        Console.WriteLine($"type: {obj.GetType()}, Value: {obj}");
    }
    Console.WriteLine();
}

```

> C'est le comportement des ==listes== en *Python* et *JavaScript* par exemple.

Ici, lorsque vous parcourez le contenu de `myObjects`, vous affichez le type sous-jacent de chaque élément à l'aide de la méthode `GetType()` de `System.Object`, ainsi que la valeur de l'élément actuel. Sans entrer dans les détails concernant `System.Object.GetType()` à ce stade du texte, comprenez simplement que **cette méthode peut être utilisée pour obtenir le nom complet de l'élément** (le [[Chapitre 17|chapitre 17]] examine en détail le sujet des informations de type et des services de réflexion). La sortie suivante montre le résultat de l'appel à `ArrayOfObjects()` :

```
==== Array of Objects ====
type: System.Int32, Value: 10
type: System.Boolean, Value: False
type: System.DateTime, Value: 10/23/1997 12:00:00 AM
type: System.String, Value: Form & Void
```

## Travailler avec des tableaux multidimensionnels

Outre les tableaux unidimensionnels que vous avez vus jusqu'à présent, **C# prend en charge deux types de tableaux multidimensionnels. Le premier est un *tableau rectangulaire*, qui est simplement un tableau à plusieurs dimensions, *où chaque ligne a la même longueur.**** Pour déclarer et remplir un tableau rectangulaire multidimensionnel, procédez comme suit :

```cs
static void RectMultidimentionalArray()
{
    Console.WriteLine("==== Rectangular Multidimensional Array ====");

    // Un tableau en 2 dimension rectangulaire (matrice)
    int[,] myMatrix;
    myMatrix = new int[3, 4];

    // Populate (3 * 4) array.
    for (int i = 0; i < 3; i++)
    {
        for (int j = 0; j < 4; j++)
        {
            myMatrix[i, j] = i * j;
        }
    }

    // Affiche le tableau (3 lignes x 4 colonnes).
    for (int i = 0; i < 3; i++)
    {
        for (int j = 0; j < 4; j++)
        {
            Console.Write(myMatrix[i, j] + "\t");
        }
        Console.WriteLine();
    }
    Console.WriteLine();
}

```

Le deuxième type de tableau multidimensionnel est appelé *tableau dentelé (jagged array)*. Comme son nom l'indique, **un tableau dentelé contient un certain nombre de tableaux internes, chacun pouvant avoir une limite supérieure différente**. Voici un exemple :

```cs
static void JaggedMultidimensionalArray()
{
    Console.WriteLine("==== Jagged Multidimensional Array ===");

    // Déclaration d'un tableau irrégulier à plusieurs dimensions 
    // (c.-à-d. un tableau de tableau)
    // Ici, on a un tableau contenant 5 tableau de taille différents.
    int[][] myJagArray = new int[5][];

    // Création du tableau irrégulier.
    for (int i = 0; i < myJagArray.Length; i++)
    {
        myJagArray[i] = new int[i + 7];
    }
    // Affiche chaque ligne (souvenez-vous, chaque élément est par défaut 0)
    for (int i = 0; i < 5; i++)
    {
        for (int j = 0; j < myJagArray[i].Length; j++)
        {
            Console.Write(myJagArray[i][j] + " ");
        }
        Console.WriteLine();
    }
    Console.WriteLine();
}
```

Le résultat de l'appel de chacune des méthodes `RectMultidimensionalArray()` et `JaggedMultidimensionalArray()` est présenté ci-dessous :

```
==== Rectangular Multidimensional Array ====
0       0       0       0
0       1       2       3
0       2       4       6

==== Jagged Multidimensional Array ===
0 0 0 0 0 0 0
0 0 0 0 0 0 0 0
0 0 0 0 0 0 0 0 0
0 0 0 0 0 0 0 0 0 0
0 0 0 0 0 0 0 0 0 0 0
```

## Utilisation de tableaux comme arguments ou valeurs de retour 

==Après avoir créé un tableau, vous pouvez le transmettre comme argument ou le recevoir comme valeur de retour d'un membre==. Par exemple, la méthode `PrintArray()` suivante prend un tableau d'entiers et affiche chaque membre sur la console, tandis que la méthode `GetStringArray()` remplit un tableau de chaînes et le renvoie à l'appelant :

```cs
static void PrintArray(int[] myInts)
{
    for (int i = 0; i < myInts.Length; i++)
    {
        Console.WriteLine($"Item {i} is {myInts[i]}");
    }
}

static string[] GetStringArray()
{
    string[] theStrings = ["Hello", "from", "GetStringArray"];
    return theStrings;
}
```

Ces méthodes peuvent être invoquées comme vous vous y attendez.

```cs
static void PassAndReceiveArrays()
{
    Console.WriteLine("==== Arrays As Parameters And Return Values ====");

    // Envoit un tableau comme paramètre.
    int[] ages = [20, 22, 23, 0];
    PrintArray(ages);

    // Reçoit un tableau comme valeur de retour.
    string[] strs = GetStringArray();
    foreach (string s in strs)
    {
        Console.WriteLine(s);
    }
    Console.WriteLine();
}

```

À ce stade, vous devriez être à l'aise avec le processus de définition, de remplissage et d'analyse du contenu d'une variable de tableau C#. ==Pour compléter le tableau, examinons maintenant le rôle de la classe `System. Array`==.

## Utilisation de la classe de base `System.Array` 

**Chaque tableau que vous créez tire une grande partie de ses fonctionnalités de la classe `System.Array`**. Grâce à ces membres communs, vous pouvez exploiter un tableau à l'aide d'un modèle objet cohérent. Le [[#Tableau 4-1 Sélection de membre de `System.Array`|Tableau 4-1]] présente quelques-uns des membres les plus intéressants (consultez la documentation pour plus de détails).

##### Tableau 4-1: Sélection de membres de `System.Array`

| Membre de la Classe `Array` | Description                                                                                                                                                                                                                                                             |
| --------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Clear()`                   | Cette méthode statique définit une plage d'éléments du tableau à des valeurs vides (`0` pour les nombres, `null` pour les références d'objet, `false` pour les booléens).                                                                                               |
| `CopyTo()`                  | Cette méthode est utilisée pour copier des éléments du tableau source dans le tableau de destination.                                                                                                                                                                   |
| `Length`                    | Cette propriété renvoie le nombre d'éléments dans le tableau.                                                                                                                                                                                                           |
| `Rank`                      | Cette propriété renvoie le nombre de dimensions du tableau actuel.                                                                                                                                                                                                      |
| `Reverse()`                 | Cette méthode statique inverse le contenu d'un tableau unidimensionnel.                                                                                                                                                                                                 |
| `Sort()`                    | Cette méthode statique trie un *tableau unidimensionnel de types primitif*. **Si les éléments du tableau implémentent l'interface `IComparer`, vous pouvez également trier vos types personnalisés** (voir [[Chapitre 8\|Chapitre 8]] et [[Chapitre 10\|Chapitre 10]]). |

Voyons quelques-uns de ces membres en action. La méthode d'assistance suivante utilise les méthodes statiques `Reverse()` et `Clear()` pour transmettre des informations sur un tableau de types `string`  à la console :

```cs
static void SystemArrayFuctionality()
{
    Console.WriteLine("==== Working With System.Array ====");
    
    // Ititialise les éléments lors de la déclaration.
    string[] gothicBands = [ "Tones on Tail", "Bauhaus", "Sisters of Mercy" ];

    // Affiche le nom dans l'ordre déclaré.
    Console.WriteLine("-> Here is the array:");
    for (int i = 0; i < gothicBands.Length; i++)
    {
        Console.Write(gothicBands[i] + ", ");
    }
    Console.WriteLine('\n');
	
    // On les renverse ...
    Array.Reverse(gothicBands);
    Console.WriteLine("-> The reversed array:");

    // ... et on les affiches.
    for (int i = 0; i < gothicBands.Length; i++)
    {
        // Affiche un nom.
        Console.Write(gothicBands[i] + ", ");
    }
    Console.WriteLine('\n');
    
    // Nettoie tout sauf le premier élément.
    Console.WriteLine("-> Cleared out all but one...");
    Array.Clear(gothicBands, 1, 2);
	
    for (int i = 0; i < gothicBands.Length; i++)
    {
        Console.Write(gothicBands[i] + ", ");
    }
    Console.WriteLine();
}
```

Si vous appelez cette méthode, vous obtiendrez le résultat affiché ici :

```
==== Working With System.Array ====
-> Here is the array:
Tones on Tail, Bauhaus, Sisters of Mercy,

-> The reversed array:
Sisters of Mercy, Bauhaus, Tones on Tail,

-> Cleared out all but one...
Sisters of Mercy, , ,
```

**Notez que de nombreux membres de `System.Array` sont définis comme membres statiques et sont donc appelés au niveau de la classe** (par exemple, les méthodes `Array.Sort()` et `Array.Reverse()`). ==Ces méthodes nécessite d'ajouter l'objet dans les paramètres==. **D'autres membres de `System.Array`** (comme la propriété `Length`) **sont liés au niveau de l'objet** ; ==vous pouvez donc invoquer le membre directement sur le tableau (`nomTableau.Length` par exemple)==.

## Utilisation des indices et des plages (Nouveauté C# 8.0, MaJ C# 10.0)

>[!Tip] Cette section est l'équivalent C# des *slices* en *Python*.

Pour simplifier l'utilisation des séquences (y compris les tableaux), **C# 8 introduit deux nouveaux types et deux nouveaux opérateurs à utiliser avec les tableaux**.

- `System.Index` représente un index dans une séquence.
- `System.Range` représente une sous-plage d'indices.
- L'opérateur d'indexation (`^`) indique que l'index est relatif à la fin de la séquence.
- L'opérateur de plage (`..`) indique le début et la fin d'une plage comme opérandes.

> [!note] les indices et les plages peuvent être utilisés avec des tableaux, des chaînes de caractères, `Span<T>`, `ReadOnlySpan<T>` et (ajouté dans C# 10) `IEnumerable<T>`.

Comme vous l'avez déjà vu, **les tableaux sont indexés à partir de zéro (`0`)**. ==La fin d'une séquence correspond à la longueur de la séquence – 1==. La boucle `for` précédente, qui a imprimé le tableau `gothicBands`, peut être mise à jour comme suit :

```cs
for (int i = 0; i < gothicBands.Length; i++)
{
	Index idx = i;
	// Affiche un nom.
	Console.Write(gothicBands[idx] + ", ");
}
```

**L'opérateur d'*indexation depuis la fin(`^`)* vous permet de spécifier le nombre de positions à partir de la fin de la séquence, en commençant par la longueur**. N'oubliez pas que le dernier élément d'une séquence est inférieur d'une unité à la longueur réelle ; `^0` provoquerait donc une erreur. ==Le code suivant affiche le tableau à l'envers== :

```cs
// Notez la différence dans la boulce for.
for (int i = 1; i <= gothicBands.Length; i++)
{
    // ^ va dabord se référer à la fin, puis à l'avant dernier,...
    // Ici, aucune modification n'est effectué dans le tableau,
    // contrairement à Array.Reverse().
    Index idx = ^i;
    Console.Write(gothicBands[idx] + ", ");
}
Console.WriteLine('\n');
```

L'opérateur de plage (`..`) spécifie un index de début et de fin et permet d'accéder à une sous-séquence d'une liste. **Le début de la plage est inclusif et la fin est exclusive**. ==Par exemple, pour extraire les deux premiers membres du tableau, créez des plages allant de 0 (le premier membre) à 2 (un de plus que la position d'index souhaitée)==.

```cs
foreach (var itm in gothigBands[0..2])
{
	// print a name
	Console.WriteLine(itm + ", ");
}
Consolel.WriteLine('\n');
```

Les plages peuvent également être transmises à une séquence à l’aide du nouveau type de données `Range`, comme indiqué ici :

```cs
static void UsingRangeDatatypeAndForeach(string[] gothicBands)
	 Console.WriteLine(" -> using Range datatype and foreach");
    Range r = 0..2; // La fin de l'intervale est exclusif.
    foreach (var itm in gothicBands[r])
    {
        // Affiche un nom.
        Console.Write(itm + ", ");
    }
    Console.WriteLine('\n');
```

Les plages peuvent être **définies à l'aide d'entiers ou de variables `Index`. Le même résultat se produit avec le code suivant :

```cs
Console.WriteLine("-> mix Index and Range datatypes");
Index idx1 = 0;
Index idx2 = 2;
Range r2 = idx1..idx2; // La fin de l'intervale est expclusif.
foreach (var itm in gothicBands[r2])
{
    // Affiche un nom.
    Console.Write(itm + ", ");
}
Console.WriteLine('\n');
```

**Si le début de la plage est omis, le début de la séquence est utilisé. Si la fin de la plage est omis, la longueur de la plage est utilisée. Cela ne provoque pas d'erreur, car la valeur à la fin de la plage est exclusive**. Dans l'exemple précédent de trois éléments dans un tableau, toutes les plages représentent le même sous-ensemble.

```cs
// Pour la varialbe gothicBands,
// les trois notations suivantes sont exactement identiques:
gothicBands[..]
gothicBands[0..^0]
gothicBands[0..3]
```

**La méthode d'extension `ElementAt()` (dans l'espace de noms `System.Linq`) récupère l'élément du tableau à l'emplacement spécifié.** ==Nouveauté de .NET 6/C# 10 : l'utilisation de l'opérateur `^` est prise en charge pour obtenir un élément à la distance spécifiée de la fin du tableau==. Le code suivant récupère l'avant-dernière bande de la liste :

```cs
var band = gothicBands.ElementAt(^2);
Console.WriteLine(
    $"Second to last element of {nameof(gothicBands)}: {band}"
);
```

>[!note]   La prise en charge des paramètres `Index` et `Range` a été ajoutée à `LINQ`. Voir le [[Chapitre 13|Chapitre 13]] pour plus d'informations.

# Comprendre les méthodes

Examinons en détail la définition des méthodes. Les méthodes sont définies par un modificateur d'accès et un type de retour (ou `void` pour l'absence de retour) et peuvent prendre ou non des paramètres. **Une méthode qui renvoie une valeur à l'appelant est généralement appelée *fonction*, tandis que les méthodes qui ne renvoient pas de valeur sont généralement appelées *méthodes*.

> [!warning] Attention:
> 
> Cette définition entre méthode est fonction est une distinction ancienne en C#. De par ca conception, le C# fonctionne seulement avec des *objets*, et donc des *classes*. Ce qui veut dire que tout bloc de code regroupé en un nom est une ==méthodes==. https://stackoverflow.com/questions/155609/whats-the-difference-between-a-method-and-a-function

>[!tip]- Détail supplémentaire sur le sujet (Gemini)
>D'un point de vue **technique et puriste**, la distinction d'Andrew Troelsen est tout à fait correcte et historique :
>
>1. **Fonction** : Un bloc de code qui prend des entrées, effectue un calcul et **renvoie** un résultat (ex: `Math.Sqrt`).
>2. **Procédure (ou Méthode void)** : Un bloc de code qui effectue une action (effet de bord) mais **ne renvoie rien** (ex: `Console.WriteLine`).
>
>### Dans le langage courant (C# moderne)
>
>Cependant, dans la pratique quotidienne des développeurs .NET, on utilise presque exclusivement le mot **"Méthode"** pour les deux cas. Pourquoi ? Parce qu'en Programmation Orientée Objet (POO), toute fonction qui appartient à une classe est officiellement une "méthode".
>
> ### Pourquoi cette nuance est-elle dans le livre ?
>
>Troelsen souligne cette différence car elle vient du monde de la programmation procédurale (comme le Pascal ou le Fortran) où l'on distinguait strictement les `Functions` des `Procedures`.
>
>En C#, la distinction est purement **sémantique** :
>
> - Si vous dites "J'appelle une fonction", on s'attend à ce que vous récupériez une valeur (`var x = F();`).
>- Si vous dites "J'appelle une méthode", cela couvre tout le spectre.
>
>**En résumé :** Oui, c'est correct, mais c'est une distinction de vocabulaire plutôt que de syntaxe. Pour le compilateur, tout est une "Member Method".

>[!note] Les modificateurs d'accès pour les classes et les méthodes sont couvert dans le [[Chapitre 5#Comprendre les modificateurs d'accès (MaJ C 7.2)|Chapitre 5]]. Les paramètre de méthodes sont couvert dans la section suivante.

À ce stade du texte, chacune de vos méthodes a le format de base suivant :

```cs
// Souvenez vous qu'une méthoce statique peut être appelé directement
// sans créer d'instance de classe.
// static typeRetour NomMéthode(list paramètres) { /* Implémentation */ }
static int Add(int x, int y)
{
    return x + y;
}

```

**Comme vous le verrez dans les prochains chapitres, les méthodes peuvent être implémentées dans le cadre de classes, de structures ou (nouveauté de C# 8) d'interfaces.**

## Comprendre les membres à corps d'expression

Vous avez déjà découvert les méthodes simples qui renvoient des valeurs, comme la méthode `Add()`. C# 6 a introduit ==les membres à corps d'expression qui raccourcissent la syntaxe des méthodes monolignes==. Par exemple, `Add()` peut être réécrit avec la syntaxe suivante :

```cs
static int Add(int x, int y) => x + y;
```

C'est ce que l'on appelle communément le *sucre syntaxique*, ce qui signifie que le code IL généré n'est pas différent. Il s'agit simplement d'une autre façon d'écrire la méthode. Certains la trouvent plus facile à lire, d'autres non ; à vous de choisir (ou à votre équipe) le style que vous préférez.

>[!note]- Ne vous inquiétez pas de l'opérateur `=>`. 
>Il s'agit de l'*opérateur lambda*, traitée en détail au [[Chapitre 12|chapitre 12]]. Ce chapitre explique également le fonctionnement précis des membres d'expression. 
>>[!tip] Dans ce cadre précis, considérez-les simplement comme un raccourci pour écrire des instructions sur une seule ligne.
>

## Comprendre les fonctions locales (Nouveauté C# 7.0,  MaJ C# 9.0)

Une fonctionnalité introduite en C# 7.0 est *la possibilité de créer des méthodes à l'intérieur de méthodes, officiellement appelées fonctions locales*. **Une fonction locale est une fonction déclarée à l'intérieur d'une autre fonction. Elle doit être privée, peut être statique en C# 8.0 (voir la section suivante) et ne prend pas en charge la surcharge**. Les fonctions locales prennent en charge l'imbrication : **une fonction locale peut contenir une autre fonction locale déclarée à l'intérieur**.

Pour comprendre comment cela fonctionne, créez un projet d'application console nommé *FunWithLocalFunctions*. Par exemple, ==imaginons que vous souhaitiez étendre l'exemple `Add()` utilisé précédemment pour **inclure la validation des entrées**==. Il existe plusieurs façons d'y parvenir, et une méthode simple consiste à **ajouter la validation directement dans la méthode `Add()`**. Continuons sur cette lancée et mettons à jour l'exemple précédent comme suit (le commentaire représentant la logique de validation) :

```cs
static int Add(int x, int y)
{
	//Effectuer des validations ici.
	return x + y;
}

```

Comme vous pouvez le constater, il n'y a pas de changements majeurs. Il y a juste un commentaire indiquant que le code réel doit faire quelque chose. **Et si vous vouliez séparer la raison réelle de la méthode** (renvoyer la somme des arguments) **de la validation des arguments ? Vous pourriez créer des méthodes supplémentaires et les appeler depuis la méthode `Add()`**. Mais cela nécessiterait de créer une autre méthode, uniquement utilisable par une autre méthode. C'est peut-être excessif. **Les fonctions locales permettent d'effectuer d'abord la validation, puis d'encapsuler l'objectif réel de la méthode définie dans la méthode `AddWrapper()`, comme illustré ici** :

```cs

static int AddWrapper(int x, int y)
{
    //Effecture des validations ici.
    return Add();

    int Add() => x + y;
}
```

**La méthode `Add()` contenue ne peut être appelée que depuis la méthode `AddWrapper()` englobante**. Vous vous demandez sans doute : « Qu'est-ce que cela m'a apporté ?» La réponse pour cet exemple précis est, tout simplement, peu (voire rien). **Mais que se passerait-il si `AddWrapper()` devait exécuter la fonction `Add()` depuis plusieurs emplacements ? Vous devriez maintenant commencer à comprendre l'intérêt d'une fonction locale pour la réutilisation du code, non exposée ailleurs que là où elle est nécessaire**. Vous découvrirez encore plus d'avantages avec les fonctions locales lorsque nous aborderons les méthodes d'itérateur personnalisées (chapitre 8) et les méthodes asynchrones ([[Chapitre 15|Chapitre 15]]).

>[!tip] Petite précision sur les fonctions locales:
>La fonction locale `AddWrapper()` est un exemple de fonction locale imbriquée. ***Rappelons que les fonctions déclarées dans les instructions de niveau supérieur sont créées comme des fonctions locales.*** La fonction locale `Add()` se trouve dans la fonction locale `AddWrapper()`. Cette fonctionnalité n'est généralement pas utilisée en dehors des exemples pédagogiques, mais si vous avez besoin d'imbriquer des fonctions locales, sachez que C# la prend en charge.

**C# 9.0 a mis à jour les fonctions locales pour permettre l'ajout d'attributs à une fonction locale, à ses paramètres et à ses paramètres de type, comme dans l'exemple suivant** (ne vous souciez pas de l'attribut `NotNullWhen`, qui sera abordé plus loin dans ce chapitre) :

```cs
// Dans le fichier .csproj, les Nullables sont désactivés
// Pour que l'alerte sur la redondance disparaisse.
#nullable enable
private static void Process(string?[] lines, string mark)
{
	foreach (var line in lines)
	{
		// La logique de traîtement.
	}
	
	bool isValid([NotNullWhen(true)] string? line)
	{
		return !string.IsNullOrEmpty(line) && line.Length >= mark.Length;
	}
}
```

## Comprendre les fonctions locales statiques (Nouveauté C# 8.0) 

**Une amélioration des fonctions locales introduite en C# 8 est la possibilité de déclarer une fonction locale comme statique**. Dans l'exemple précédent, la fonction locale `Add()` référençait directement les variables de la fonction principale. Cela pouvait entraîner des effets secondaires inattendus, car la fonction locale peut modifier les valeurs des variables. 

Pour voir cela en action, créez une nouvelle méthode appelée `AddWrapperWithSideEffect()`, comme illustré ici :

```cs
static int AddWrapperWithSideEffects(int x, int y)
{
    // Effectuer des validations ici
    return Add();

    int Add()
    {
		x += 1;
		return x + y;
    }
}
```

Bien sûr, cet exemple est tellement simple qu'il ne se produirait probablement pas dans du code réel. Pour éviter ce type d'erreur, ajoutez le modificateur `static` à la fonction locale. Cela empêche la fonction locale d'accéder directement aux variables de la méthode *parente*, ce qui provoque l'exception du compilateur:

```
CS8421 : « Une fonction locale statique ne peut pas contenir de référence à ‘<nom de la variable>’. »
```

**La version améliorée de la méthode précédente est présentée ici **:

```cs
static int AddWrapperWithStatic(int x, int y)
{
	// Effectuer des validations ici.
	return Add(x, y);
	
	static int Add(int x, int y)
	{
		return x + y;
	}
}
```

# Comprendre les paramètres de méthode

==Les paramètres de méthode servent à transmettre des données à un appel de méthode==. Dans les sections suivantes, vous découvrirez en détail comment les méthodes (et leurs appelants) traitent les paramètres.

## Comprendre les modificateurs de paramètres de méthode

**Par défaut, un paramètre est passé à une fonction *par valeur***. Autrement dit, si vous n'indiquez pas de modificateur de paramètre sur un argument, ==une copie des données est transmise à la fonction==. Comme expliqué plus loin dans ce chapitre, ==les données copiées dépendent du type du paramètre : *valeur* ou *référence.

Bien que la définition d'une méthode en C# soit assez simple, *vous pouvez utiliser quelques méthodes pour contrôler la manière dont les arguments lui sont transmis*, comme indiqué dans le [[#Tableau 4-2 Les modificateurs de paramètres C|Tableau 4-2]]*.

##### Tableau 4-2: Les modificateurs de paramètres C# 

| Modificateur de paramètre | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `(None)`                  | Si un paramètre de type valeur n'est pas marqué d'un modificateur, il est considéré comme transmis par valeur, ce qui signifie que la méthode appelée reçoit une copie des données d'origine. Les types référence sans modificateur sont transmis par référence.                                                                                                                                                                                                                     |
| `out`                     | Les paramètres de sortie doivent être assignés par la méthode appelée et sont donc transmis par référence. Si la méthode appelée ne parvient pas à assigner les paramètres de sortie, une erreur de compilation est générée.                                                                                                                                                                                                                                                         |
| `ref`                     | La valeur est initialement attribuée par l'appelant et peut éventuellement être modifiée par la méthode appelée (les données étant également transmises par référence). Aucune erreur de compilation n'est générée si la méthode appelée ne parvient pas à attribuer un paramètre `ref`.                                                                                                                                                                                             |
| `in`                      | Nouveauté dans C# 7.2, le modificateur `in` indique qu'un paramètre `ref` est en lecture seule par la méthode appelée.                                                                                                                                                                                                                                                                                                                                                               |
| `params`                  | Ce modificateur de paramètre vous permet d'envoyer un nombre variable d'arguments sous forme de paramètre logique unique. Une méthode ne peut avoir qu'un seul modificateur params, et celui-ci doit être le paramètre final de la méthode. Vous n'aurez peut-être pas besoin d'utiliser le modificateur params très souvent ; cependant, sachez que de nombreuses méthodes des bibliothèques de classes de base utilisent cette fonctionnalité du langage C#. (comme `*` en Python) |

Pour illustrer l'utilisation de ces mots-clés, créez un projet d'application console nommé
*FunWithMethods*. Voyons maintenant le rôle de chaque mot-clé.

## Comprendre le comportement par défaut du passage de paramètres

**Lorsqu'un paramètre ne possède pas de modificateur, le comportement des types valeur est de passer le paramètre par valeur et celui des types référence est de passer le paramètre par référence.**

>[!Info] les type de donnée *valeur* et *références* sont expliqué plus tard dans ce chapitre.

### Comportement par défaut des types de valeurs

==Par défaut, un paramètre de type valeur est transmis à une fonction *par valeur*==. Autrement dit, **si vous ne marquez pas l'argument avec un modificateur, une copie des données est transmise à la fonction**. Ajoutez la méthode suivante au fichier *Program.cs*, qui traite deux types de données numériques transmis par valeur :

```cs
// Des arguments de type valeur sont passé par valeur par défaut
static int Add(int x, int y)
{
	int ans = x + y;
	// L'appelant ne verra pas ces changements comme
	// on modifie une copie des donnée originales.
	x = 10_000;
	y = 88_888;
	return ans; 
}
```

**Les données numériques appartiennent à la catégorie des *types valeur***. Par conséquent, ==si vous modifiez les valeurs des paramètres dans la portée du membre, l'appelant n'en sera absolument pas conscient, car vous modifiez les valeurs sur une *copie* de ses données d'origine==.

```cs
Console.Title = "Fun with Methods";
Console.WriteLine("***** Fun with Methods *****\n");

// Envois deux variable dedans par valeur.
int x = 9, y = 10;

Console.WriteLine($"Before call: X: {x}, Y: {y}");
Console.WriteLine($"Answer is : {Add(x, y)}");
Console.WriteLine($"After call: X: {x}, Y: {y}");

Console.ReadLine();
```

Comme on pouvait s'y attendre, les valeurs de `x` et `y` restent identiques avant et après l'appel à `Add()`, comme le montre le résultat suivant, car les points de données ont été envoyés par valeur. Ainsi, **toute modification de ces paramètres dans la méthode `Add()` n'est pas visible par l'appelant, car la méthode `Add()` opère sur une copie des données**.

```
***** Fun with Methods *****

Before call: X: 9, Y: 10
Answer is : 19
After call: X: 9, Y: 10
```

### Comportement par défaut des types référence

==Par défaut, un paramètre de type référence est envoyé dans une méthode *par référence pour ces propriétés, mais par valeur pour lui-même*==. Ce point est abordé en détail plus loin dans ce chapitre, après la discussion sur les types valeur et référence.

>[!Important] Précision sur le type de donnée `string`
> Même si le type de données `string` est techniquement un type référence, comme expliqué au [[Chapitre 3#Travailler avec des donnée `string`|Chapitre 3]], il s'agit d'un cas particulier. ==Lorsqu'un paramètre `string` ne possède pas de modificateur, il est transmis *par valeur*==.

## Utilisation du modificateur `out` (MaJ C# 7.0)

Ensuite, vous pouvez utiliser les paramètres de sortie. **Les méthodes définies pour accepter des paramètres de sortie (via le mot-clé `out`) doivent leur attribuer une valeur appropriée avant de quitter leur portée** (à défaut, ==des erreurs de compilation se produiront==).

À titre d'illustration, voici une version alternative de la méthode `Add()` qui renvoie la somme de deux entiers à l'aide du modificateur `out` en C# (notez que la valeur de retour physique de cette méthode est désormais `void`) :

```cs
// Output paremeter must be assigned by the called method.
static void AddUsingOutParam(int x, int y, out int ans)
{
	ans = x + y;
}
```

L'appel d'une méthode avec des paramètres de sortie nécessite également l'utilisation du modificateur `out`. ==Cependant, il n'est pas nécessaire d'assigner une valeur aux variables locales passées en sortie avant de les transmettre comme arguments de sortie== (sinon, leur valeur d'origine est perdue après l'appel). ==Si le compilateur autorise l'envoi de données apparemment non assignées, c'est parce que la méthode appelée *doit* effectuer une assignation==. Pour appeler la méthode `Add` mise à jour, créez une variable de type `int` et utilisez le modificateur `out` lors de l'appel, comme ceci :

```cs
int ans;
AddUsingOutParam(90, 90, out ans);
```

**Depuis C# 7.0, les paramètres `out` n'ont plus besoin d'être déclarés avant leur utilisation. Autrement dit, ils peuvent être déclarés directement dans l'appel de méthode, comme ceci** :

```cs
AddUsingOutParam(90, 90, out int ans);
```

Le code suivant est un exemple d'appel de méthode avec une déclaration dans la méthode du paramètre `out`:

```cs
// Il n'est pas nécessaire d'initialiser les variables locales
// utilisées comme paramètres de sortie, à condition d'être utilisées 
// comme arguments de sortie lors de leur première utilisation.
// C# 7 permet de déclarer les paramètres de sortie dans l'appel de méthode.
AddUsingOutParam(90, 90, out int ans);
Console.WriteLine("90 + 90 = {0}", ans);
Console.ReadLine();

```

**L'exemple précédent est purement illustratif**; il n'y a aucune raison de renvoyer la valeur de votre somme à l'aide d'un paramètre de sortie. **Cependant, le modificateur `out` de C# est utile : il permet à l'appelant d'obtenir plusieurs sorties à partir d'une seule invocation de méthode.**

```cs
// Returning multiple output parameters.
static void FillTheseValues(out int a, out string b, out bool c)
{
	a = 9;
	b = "Enjoy your string.";
	c = true;
}
```

**L'appelant pourra invoquer la méthode `FillTheseValues()`.** N'oubliez pas d'utiliser le modificateur `out` lors de l'invocation et de l'implémentation de la méthode.

```cs
FillTheseValues(out int i, out string str, out bool b);
Console.WriteLine("Int is: {0}", i);
Console.WriteLine("String is: {0}", str);
Console.WriteLine("Boolean is: {0}", b);
Console.ReadLine();
```

>[!Info] C# 7 a également introduit les *tuples*, une autre façon de renvoyer plusieurs valeurs à partir d'un appel de méthode. Vous en apprendrez davantage à ce sujet plus loin dans ce chapitre.

**N'oubliez jamais qu'une méthode définissant des paramètres de sortie doit leur attribuer une valeur valide avant de quitter la portée de la méthode**. Par conséquent, ==le code suivant entraînera une erreur de compilation, car le paramètre de sortie n'a pas été attribué dans la portée de la méthode== :

```cs
static void ThisWontCompile(out int a)
{
	Console.WriteLine("Error! Forget to assign output arg!");
}
```

### Supprimer/ignorer les paramètres de sortie (Nouveauté 7.0)

Si la valeur d'un paramètre `out` ne vous intéresse pas, **vous pouvez utiliser un *rebut* comme espace réservé. Les rebuts sont des variables temporaires et factices, intentionnellement inutilisées**. Elles ne sont pas assignées, n'ont pas de valeur et **peuvent même ne pas allouer de mémoire**. ==Cela peut améliorer les performances et rendre votre code plus lisible==. **Les rebuts peuvent être utilisés avec des paramètres de sortie, avec des tuples** (plus loin dans ce chapitre), **avec des correspondance de motifs** ([[Chapitre 6|chapitres 6]] et [[Chapitre 8|chapitre 8]]), **ou même comme variables autonomes**.

Par exemple, si vous souhaitez obtenir la valeur de l'`int` de l'exemple précédent, mais que les deux autres paramètres ne vous intéressent pas, vous pouvez écrire le code suivant :

```cs
// Ceci ne récupèrent seulement la valeur de a, 
// et ignore les deux autre paramètres.
FillTheseValues(out int a, out _, out _);
```

==Notez que la méthode appelée continue de définir les valeurs des trois paramètres== ; les deux derniers sont simplement ignorés au retour de l'appel de méthode. 

## Le modificateur `out` dans les constructeurs et les initialiseurs (Nouveauté C# 7.3)

C# 7.3 a étendu les emplacements autorisés pour l'utilisation du paramètre `out`. **Outre les méthodes, les paramètres des constructeurs, les initialiseurs de champs et de propriétés, ainsi que les clauses de requête peuvent tous être décorés avec le modificateur `out`**. Des exemples seront examinés plus loin dans ce livre.

## Utilisation du modificateur `ref`

Examinons maintenant l'utilisation du modificateur de paramètre `ref` en C#. **Les paramètres de type référence sont nécessaires pour permettre à une méthode d'opérer sur (et généralement de modifier les valeurs) divers points de données déclarés dans la portée de l'appelant (comme une routine de tri ou d'échange).** ==Notez la distinction entre paramètres de sortie et paramètres de référence== :

- **Les paramètres de sortie n'ont pas besoin d'être initialisés avant d'être transmis à la méthode**. En effet, la méthode doit assigner des paramètres de sortie avant de quitter.

- **Les paramètres de référence doivent être initialisés avant d'être transmis à la méthode**. En effet, vous transmettez une référence à une variable existante. ==Si vous ne l'assignez pas à une valeur initiale, cela équivaudrait à une opération sur une variable locale non assignée==.

Examinons l'utilisation du mot-clé `ref` par une méthode qui permute deux variables de type `string` (bien sûr, deux types de données quelconques peuvent être utilisés ici, notamment int, bool, float, etc.).

```cs
// Paramètre par références.
static void SwapStrings(ref string s1, ref string s2)
{
	// Autre manière d'échancer des valeurs:
	// string tempStr = s1;
	// s1 = s2;
	// s2 = tempStr;
	(s1, s2) = (s2, s1);
}
```

Cette méthode peut être appelé comme suit:

```cs
Console.WriteLine("***** Fun with Methods *****");
string str1 = "Flip";
string str2 = "Flop";
Console.WriteLine("Before: {0}, {1} ", str1, str2);
SwapStrings(ref str1, ref str2);
Console.WriteLine("After: {0}, {1} ", str1, str2);
Console.ReadLine();
```

Ici, l'appelant a attribué une valeur initiale aux données de chaîne locales (`str1` et `str2`). Après le retour de `SwapStrings()`, `str1` contient désormais la valeur `"Flop"`, tandis que `str2` renvoie la valeur `"Flip"`.

```
before: Flip, Flop
After: Flop, Flip
```

## Utilisation du modificateur `in` (Nouveauté C# 7.2)

**Le modificateur `in` transmet une valeur par référence** (==pour les types valeur **ET** référence==) **et empêche la méthode appelée de modifier les valeurs.** Cela indique clairement une intention de conception dans votre code et ==permet potentiellement de réduire la pression mémoire==. Lorsque les types valeur sont transmis par valeur, ils sont copiés (en interne) par la méthode appelée. ==Si l'objet est volumineux (comme un `struct` volumineux), la surcharge liée à la création d'une copie pour une utilisation locale peut être importante==. De plus, ==même lorsque les types référence sont transmis sans modificateur, ils peuvent être modifiés par la méthode appelée==. **Ces deux problèmes peuvent être résolus grâce au modificateur `in`**.

En reprenant la méthode `Add()` décrite précédemment, deux lignes de code modifient les paramètres, mais n'affectent pas les valeurs de la méthode appelante. Ces valeurs ne sont pas affectées car la méthode `Add()` copie les variables `x` et `y` pour une utilisation locale. ==Bien que la méthode appelante n'ait aucun effet secondaire, que se passerait-il si la méthode `Add()` était modifiée comme suit ?==:

```cs
static int Add2(int x, int y)
{
	x = 10_000;
	y = 88_888;
	int ans = x + y;
	return ans;
}
```

L'exécution de ce code renvoie alors `98888`, quels que soient les nombres envoyés à la méthode. Il s'agit manifestement d'un problème. Pour corriger cela, mettez à jour la méthode comme suit :

```cs
static int AddReadOnly(in int x, in int y)
{
	//Error CS8331 Cannot assign to variable 'in int' because it is a readonly variable
	//x = 10000;
	//y = 88888;
	int ans = x + y;
	return ans;
}
```

Lorsque le code tente de modifier les valeurs des paramètres, le compilateur génère l'erreur CS8331, indiquant que les valeurs ne peuvent pas être modifiées en raison du modificateur `in`.

## Utilisation du modificateur `params`

> L'équivalent du paramètre `*args` dans les fonction Python.

**C# prend en charge l'utilisation de tableaux de paramètres grâce au mot-clé `params`. Ce mot-clé permet de passer à une méthode un nombre variable de paramètres de même type (ou de classes liées par héritage) comme *paramètre logique unique***. De plus, les arguments marqués avec le mot-clé `params` ==peuvent être traités si l'appelant envoie un tableau fortement typé ou une liste d'éléments délimitée par des virgules==. Oui, cela peut prêter à confusion ! Pour clarifier les choses, supposons que vous souhaitiez créer une fonction permettant à l'appelant de passer un nombre quelconque d'arguments et de renvoyer la moyenne calculée.

Si vous prototypez cette méthode pour prendre un tableau de `double`, cela obligera l'appelant à d'abord définir le tableau, puis à le remplir, et enfin à le passer à la méthode. Cependant, si vous définissez `CalculateAverage()` pour prendre un paramètre de type `double[]`, l'appelant peut simplement passer une liste de doubles délimitée par des virgules. La liste des doublons sera regroupée en coulisses dans un tableau de `double`.

```cs
// Renvois la moyenne d'un certain nombre de doubles.
static double CalculateAverage(params double[] values)
{
	Console.WriteLine("You sent me {0} doubles.", values.Length);
	
	double sum = 0;
	if(values.Length == 0)
	{
		return sum;
	}
	for (int i = 0; i < values.Length; i++)
	{
		sum += values[i];
	}
	return (sum / values.Length);
}
```

Cette méthode a été définie pour prendre un tableau de paramètres contenant des `double`. **En fait, elle dit : « Envoyez-moi n'importe quel nombre de doubles (y compris zéro) et je calculerai la moyenne ».** Ceci étant dit, vous pouvez appeler la fonction `CalculateAverage()` de l'une des manières suivantes :

```cs
// On envoit une liste de doubles délimité par des virgules.
double average;
average = CalculateAverage(4.0, 3.2, 5.7, 64.22, 87.2);
Console.WriteLine("Average of data is: {0}", average);

// ... ou on envoit un tableau de double.
double[] data = { 4.0, 3.2, 5.7 };
average = CalculateAverage(data);
Console.WriteLine("Average of data is: {0}", average);

// La moyenne de 0 est 0!
Console.WriteLine("Average of data is: {0}", CalculateAverage());
Console.ReadLine();
```

**Si vous n'avez pas utilisé le modificateur `params` dans la définition de `CalculateAverage()`, la première invocation de cette méthode entraînerait une erreur du compilateur, car celui-ci rechercherait une version de `CalculateAverage()` prenant cinq arguments `double`**.

>[!Important] 
>Pour éviter toute ambiguïté, C# exige qu'une méthode ne prenne en charge qu'un seul argument `params`, qui ==doit être le dernier argument de la liste de paramètres==.

Comme vous pouvez l'imaginer, **cette technique n'est qu'une commodité pour l'appelant, puisque le tableau est créé par le runtime .NET Core si nécessaire**. ==Dès que le tableau est dans la portée de la méthode appelée, vous pouvez le traiter comme un tableau .NET Core complet contenant toutes les fonctionnalités du type de bibliothèque de classes de base `System.Array`==. Considérez le résultat suivant :

```
You sent me 5 doubles.
Average of data is: 32.864
You sent me 3 doubles.
Average of data is: 4.3
You sent me 0 doubles.
Average of data is: 0
```

## Définir des paramètres optionnels

C# permet de créer des méthodes acceptant des *arguments optionnels*. Cette technique permet à l'appelant d'**invoquer une seule méthode en omettant les arguments jugés inutiles, à condition que les valeurs par défaut spécifiées lui conviennent**.

Pour illustrer l'utilisation d'arguments optionnels, supposons que vous disposiez d'une méthode nommée `EnterLogData()`, qui définit un seul paramètre *optionnel*.

```cs
static void EnterLogData(string message, string owner = "Programmer")
{
	Console.WriteLine($"Error: {message}");
	Console.WriteLine($"Owner of Error: {owner}");
}
```

Ici, la valeur par défaut `« Programmer »` a été attribuée à l'argument `string` final via une affectation dans la définition du paramètre. Ceci permet d'appeler `EnterLogData()` de deux manières.

```cs
EnterLogData("Oh no! Grid can't find data");
EnterLogData("Oh no! I can't find the payroll data", "CFO");
Console.ReadLine();
```

Étant donné que la première invocation de `EnterLogData()` ne spécifiait pas de second argument de type `string`, vous constateriez que le programmeur est responsable de la perte des données de la grille, tandis que le directeur financier a égaré les données de paie (comme spécifié par le second argument du second appel de méthode).

**Il est important de noter que la valeur attribuée à un paramètre facultatif doit être connue à la compilation et ne peut être résolue à l'exécution** (si vous tentez de le faire, vous obtiendrez des erreurs de compilation !). À titre d'exemple, supposons que vous souhaitiez mettre à jour `EnterLogData()` avec le paramètre facultatif supplémentaire suivant :

```cs
// Erreur! La valeur par défaut pour un argument optionnel 
// doit être connu lors de la compilation.
static void EnterLogData(string message, string owner = "Programmer", DateTime timeStamp =
DateTime.Now)
{
	Console.WriteLine("Error: {0}", message);
	Console.WriteLine("Owner of Error: {0}", owner);
	Console.WriteLine("Time of Error: {0}", timeStamp);
}
```

==La compilation échouera, car la valeur de la propriété `Now` de la classe `DateTime` est résolue à l'exécution et non à la compilation==.

> [!warning] 
> Pour éviter toute ambiguïté, les paramètres optionnels doivent toujours être placés à la fin de la signature d'une méthode. **C'est une erreur du compilateur de placer les paramètres optionnels avant les paramètres non optionnels**.

## Utilisation des arguments nommés (MaJ C# 7.2)

Une autre fonctionnalité du langage C# est la prise en charge des *arguments nommés*. **Ces arguments vous permettent d'invoquer une méthode en spécifiant les valeurs des paramètres dans l'ordre de votre choix.** Ainsi, plutôt que de transmettre les paramètres uniquement par leur position (comme c'est souvent le cas), **vous pouvez spécifier chaque argument par son nom à l'aide d'un opérateur deux-points**. Pour illustrer l'utilisation des arguments nommés, supposons que vous ayez ajouté la méthode suivante au fichier *Program.cs* :

```cs
static void DisplayFancyMessage(
    ConsoleColor textColor,
    ConsoleColor backgroundColor,
    string message
)
{
    // Stocker les anciennes couleurs pour
    // les restaurer après avoir afficher le message.
    ConsoleColor oldTextColor = Console.ForegroundColor;
    ConsoleColor oldBackgroundColor = Console.BackgroundColor;

    // Définit les nouvelles couleus et affiche le message.
    Console.ForegroundColor = textColor;
    Console.BackgroundColor = backgroundColor;
    Console.WriteLine(message);

    // Restaure les couleurs précédentes.
    Console.ForegroundColor = oldBackgroundColor;
    Console.BackgroundColor = oldBackgroundColor;
}

```

Étant donné la manière dont `DisplayFancyMessage()` a été écrite, on s'attendrait à ce que l'appelant invoque cette méthode en passant deux variables `ConsoleColor` suivies d'un type `string`. Cependant, **avec des arguments nommés, les appels suivants sont parfaitement acceptables** :

```cs
DisplayFancyMessage(
    message: "Wow! Very Fancy indeed!",
    textColor: ConsoleColor.DarkGreen,
    backgroundColor: ConsoleColor.DarkMagenta
);
DisplayFancyMessage(
    backgroundColor: ConsoleColor.Gray,
    message: "Testing",
    textColor: ConsoleColor.DarkYellow
);
Console.ReadLine();
```

**Les règles d'utilisation des arguments nommés ont été légèrement mises à jour avec C# 7.2. Avant cette version, si vous appelez une méthode utilisant des paramètres positionnels, vous devez les lister avant tout paramètre nommé. Avec C# 7.2 et les versions ultérieures, les paramètres nommés et non nommés peuvent être mélangés s'ils sont à la bonne position**.

> [!tip] Bonnes pratiques
> Ce n'est pas parce qu'il est possible de combiner des arguments nommés et des arguments positionnels en C# 7.2 et versions ultérieures que c'est une bonne idée. **Ce n'est pas parce que c'est possible que c'est obligatoire!**

Le code suivant est un exemple:

```cs
// Ceci est OK, comme les arguments positionnels sont listés
// avant les argument nommés.
DisplayFancyMessage(
    ConsoleColor.Blue,
    message: "Testing...",
    backgroundColor: ConsoleColor.White
);

// Ceci est OK, tout le arguments sont dans le bonne ordre.
DisplayFancyMessage(
    textColor: ConsoleColor.White,
    backgroundColor: ConsoleColor.Blue,
    "Testing..."
);

// Ceci est une ERREUR, car les arguments positionnels sont
// listés après les arguments nommés.
DisplayFancyMessage(
    message: "Testing...",
    backgroundColor: ConsoleColor.White,
    ConsoleColor.Blue
);
```

Cette restriction mise à part, vous vous demandez peut-être ==quand vous aurez besoin d'utiliser cette fonctionnalité du langage. Après tout, si vous devez spécifier trois arguments à une méthode, pourquoi inverser leurs positions ?==

**Eh bien, il s'avère que si vous avez une méthode qui définit des arguments optionnels, cette fonctionnalité peut s'avérer utile**. Supposons que `DisplayFancyMessage()` a ==été réécrit pour prendre en charge les arguments optionnels, car vous avez attribué des valeurs par défaut appropriées==.

```cs
static void DisplayFancyMessage(ConsoleColor textColor = ConsoleColor.Blue,
	ConsoleColor backgroundColor = ConsoleColor.White,
	string message = "Test Message")
{
	...
}
```

Étant donné que chaque argument possède une valeur par défaut, **les arguments nommés permettent à l'appelant de spécifier uniquement les paramètres pour lesquels il ne souhaite pas recevoir les valeurs par défaut**. Ainsi, si l'appelant souhaite que la valeur `«Hello!»` apparaisse en bleu sur fond blanc, il peut simplement spécifier ce qui suit :

```cs
DisplayFancyMessage(message: "Hello!");
```

Si l'appelant souhaite voir `"Test Message"` s'afficher sur fond vert avec du texte bleu, il peut utiliser la commande suivante :

```cs
DisplayFancyMessage(backgroundColor: ConsoleColor.Green);
```

Comme vous pouvez le constater, les arguments optionnels et les paramètres nommés ont tendance à fonctionner de concert. Pour conclure votre analyse de la création de méthodes C#, je dois aborder le sujet de la *surcharge de méthodes*.

## Comprendre la surcharge de méthodes

Comme d'autres langages orientés objet modernes, C# permet à une méthode d'être *surchargée*. **En termes simples, lorsque vous définissez un ensemble de méthodes portant le même nom et différant par le nombre (ou le type) de paramètres, la méthode en question est dite surchargée**.

Pour comprendre l'utilité de la surcharge, imaginez-vous un développeur Visual Basic 6.0 (VB6) à l'ancienne. Supposons que vous utilisiez VB6 pour créer un ensemble de méthodes renvoyant la somme de différents types de données entrantes (`Integer`, `double`, etc.). ==VB6 ne prenant pas en charge la surcharge de méthodes, vous devrez définir un ensemble unique de méthodes ayant pour fonction principale la même fonction== (renvoyer la somme des arguments).

```VB
' Example de code VB6.
Public Function AddInts(ByVal x As Integer, ByVal y As Integer) As Integer
	AddInts = x + y
End Function

Public Function AddDoubles(ByVal x As Double, ByVal y As Double) As Double
	AddDoubles = x + y
End Function

Public Function AddLongs(ByVal x As Long, ByVal y As Long) As Long
	AddLongs = x + y
End Function
```

Non seulement un tel code peut devenir difficile à maintenir, mais ==l'appelant doit désormais connaître précisément le nom de chaque méthode==. **Grâce à la surcharge, vous pouvez autoriser l'appelant à appeler une seule méthode nommée `Add()`**. ==Là encore, l'essentiel est de s'assurer que chaque version de la méthode possède un ensemble d'arguments distinct== (**les méthodes qui ne diffèrent que par leur type de retour ne sont pas suffisamment uniques**).

>[!note] Précision avec les génériques
>Comme expliqué au [[Chapitre 10|Chapitre 10]], il est possible de créer des méthodes génériques qui optimisent le concept de surcharge. Grâce aux génériques, vous pouvez définir des *espaces réservés* pour l'implémentation d'une méthode, spécifiés lors de l'appel du membre concerné.

Pour tester cela, créez un projet d'application console nommé `FunWithMethodOverloading`. Ajoutez une classe nommée `AddOperations.cs` et modifiez le code comme suit.

```cs
namespace FunWithMethodOverloading;

// Code C#
// Surcharge de la méthode Add()

public static class AddOperations
{
    // Surcharge de la méthode Add()
    public static int Add(int x, int y)
    {
        return x + y;
    }

    public static double Add(double x, double y)
    {
        return x + y;
    }

    public static long Add(long x, long y)
    {
        return x + y;
    }
}


```

Remplacez le code dans le fichier `Program.cs` par ce qui suit :

```cs
using static FunWithMethodOverloading.AddOperations;

Console.Title = "Fun with Method Overloading";
Console.WriteLine("***** Fun With Method Overloading *****\n");

// Appel de la version int de Add()
Console.WriteLine(Add(10, 10));

// Appel de la version long de Add() (avec l'usage des séparateur de chiffre.)
Console.WriteLine(Add(900_000_000_000, 900_000_000_000));

// Appel de la version double de Add()
Console.WriteLine(Add(4.3, 4.4));

Console.ReadLine();

```

>[!Note] 
> L'instruction `using static` sera abordée au [[Chapitre 5#Importer des membres statiques via le mot-clé `using` en C|Chapitre 5]]. Pour l'instant, considérez-la comme un raccourci clavier pour utiliser des méthodes contenant une classe statique nommée `AddOperations` dans l'espace de noms `FunWithMethodOverloading`.
>>[!tip] Petit couac avec neovim
>> *Roslyn* noircit le `using static` car il ne fait pas bien la différence entre un `using` normal et `using static`.
>>
>> Pour lui, vu que fichier *Program.cs* et *AddOperations.cs* sont dans le même espace de noms, il n'est pas nécessaire de l'importer. Pourtant, si on supprime la ligne, alors les appels à `Add()` renverront une erreur.

**Les instructions de niveau supérieur appelaient trois versions différentes de la méthode `Add()`, chacune utilisant un type de données différent**.

Visual Studio et Visual Studio Code facilitent l'appel de méthodes surchargées. **Lorsque vous saisissez le nom d'une méthode surchargée (comme votre fidèle `Console.WriteLine()`), IntelliSense liste chaque version de la méthode concernée. Notez que vous pouvez parcourir chaque version d'une méthode surchargée à l'aide des touches fléchées haut et bas.**

![[Figure 4.2.png|Visual Studio Code IntelliSense pour les méthodes surchargées]]

>[!tip] Avec ma configuration neovim
>la manipulation clavier en mode *normal* `gd` permet d'afficher dirrectement la définition du membre utilisé.

Si votre surcharge comporte des paramètres optionnels, le compilateur choisira la méthode la plus adaptée au code appelant, en fonction des arguments nommés et/ou positionnels. Ajoutez la méthode suivante :

```cs
static int Add(int x, int y, int z = 0)
{
	return x + (y * z);
}
```

 **Si l'argument optionnel n'est pas transmis par l'appelant, le compilateur utilisera la première signature** (celle sans le paramètre optionnel). ==Bien qu'il existe des règles pour l'emplacement des méthodes, il est généralement déconseillé de créer des méthodes qui ne diffèrent que par leurs paramètres optionnels==.

**Enfin, `in`, `ref` et `out` ne sont pas considérés comme faisant partie de la signature pour la surcharge de méthode lorsque plusieurs modificateurs sont utilisés**. Autrement dit, les surcharges suivantes génèrent une erreur de compilation :

```cs
static int Add(ref int x) { /* */ }
static int Add(out int x) { /* */ }
```

**Cependant, si une seule méthode utilise `in`, `ref` ou `out`, le compilateur peut distinguer les signatures.** Ceci est donc autorisé :

```cs
static int Add(ref int x) { /* */ }
static int Add(int x) { /* */ }
```

Ceci conclut l'examen initial de la construction de méthodes à l'aide de la syntaxe C#. Voyons maintenant comment construire et manipuler des énumérations et des structures.

## Vérification de la valeur `null` des paramètres (MaJ C# 10.0)

**Si un paramètre de méthode est nullable** (par exemple, un type de référence comme `string`) **et est requis par le corps de la méthode, il est considéré comme une bonne pratique de programmation de vérifier que le paramètre n'est pas null avant de l'utiliser.** S'il est `null`, la méthode doit lever une exception `ArgumentNullException`. Prenons l'exemple de la mise à jour suivante de la méthode `EnterLogData()` qui fait exactement cela :

```cs
static void EnterLogData(string message, string owner = "Programmer")
{
    // On veut lancer une erreur si message est nul.
    if (message == null)
    {
        throw new ArgumentNullException(message);
    }

    Console.WriteLine($"Error: {message}");
    Console.WriteLine($"Owner of Error: {owner}");
}

```

**Introduite dans C# 10, l'exception `ArgumentNullException` possède une méthode d'extension permettant de réaliser cela en une seule ligne de code** :

```cs
static void EnterLogData(string message, string owner = "Programmer")
{
    // Depuis C# 10: un méthode d'extension à été ajouté.
    ArgumentNullException.ThrowIfNull(message);
    Console.WriteLine($"Error: {message}");
    Console.WriteLine($"Owner of Error: {owner}");
}
```

>[!note] les exceptions seront couverte dans le [[Chapitre 7|Chapitre 7]] et les méthodes d'extensions dans le [[Chapitre 11|Chapitre 11]]

**L'activation des types de référence anulable** (abordée plus loin dans ce chapitre) **permet de garantir que les types de référence requis ne sont pas `null`**.

# Comprendre le type de donnée `enum`

Rappelons que, comme indiqué au [[Chapitre 1#Comprendre le Common Type System (CTS)|Chapitre 1]], le système de types .NET Core est composé de *classes*, de *structures*, d'*énumérations*, d'*interfaces* et de *délégués*. Pour explorer ces types, étudions le rôle de l'*énumération* (ou simplement, `enum`) à l'aide d'un nouveau projet d'application console nommé *FunWithEnums*.

>[!warning] Ne confondez pas les termes « *énumération* » et « *énumérateur* » ; ce sont des concepts totalement différents. 
>Une *énumération* est un type de données personnalisé composé de **paires nom-valeur**.<br>
>Un *énumérateur* est une classe ou une structure qui implémente une interface .net Core nommée `IEnumerable`. 
>Cette interface est généralement implémentée sur les classes de collection, ainsi que sur la classe `System.Array`. Comme vous le verrez au [[Chapitre 8|Chapitre 8]], les objets prenant en charge `IEnumerable` peuvent fonctionner dans la boucle `foreach`.

Lors de la création d'un système, ==il est souvent pratique de créer un ensemble de noms symboliques correspondant à des valeurs numériques connues==. Par exemple, si vous créez un système de paiement, vous pouvez désigner le type d'employés à l'aide de constantes telles que vice-président, directeur, entrepreneur et employé. C# prend en charge les énumérations personnalisées pour cette raison. Par exemple, voici une énumération nommée `EmpTypeEnum` (**vous pouvez la définir dans le même fichier que vos instructions de niveau supérieur, si elle est placée à la fin du fichier**) :

```cs
Console.Title = "Fun With Enums";
Console.WriteLine("+++++ Fun With Enums *****");

// Les fonctions locales vont ici:

Console.ReadLine();

// Une énumération personnalisée.
enum EnpTypeEnum
{
    manager, // = 0
    Grunt, // = 1
    Contractor, // = 2
    VicePresident, // = 3
}

```

>[!Note] par convention:
>les type `enum` sont suffixé avec *Enum*. Ce n'est pas nécessaire mais rend le code plus lisible.

L'énumération `EmpTypeEnum` définit quatre constantes nommées, **correspondant à des valeurs numériques discrètes**. **Par défaut, le premier élément est défini à zéro (`0`), suivi d'une progression $n+1$**. ==Vous êtes libre de modifier la valeur initiale à votre guise. Par exemple, s'il était judicieux de numéroter les membres d'`EmpTypeEnum` de `102` à `105`, vous pourriez procéder comme suit== :

```cs
// Commence avec 102
enum EnpTypeEnum
{
	manager = 102,
	Grunt,         // = 103
	Contractor,    // = 104
	VicePresident  // = 105
}

```

**Les énumérations ne doivent pas nécessairement suivre un ordre séquentiel ni posséder de valeurs uniques**. Si (pour une raison ou une autre) il est judicieux d'établir votre `EmpTypeEnum` comme indiqué ici, le compilateur continue à fonctionner :

```cs
// Les éléments d'une énumération n'ont pas besoin d'être séquentiels !
enum EnpTypeEnum
{
	Manager = 10,
	Grunt = 1,
	Contractor = 100,
	VicePresident = 9
}
```

## Contrôler le stockage sous-jacent d'une énumération

Par défaut, le type de stockage utilisé pour stocker les valeurs d'une énumération est un `System.Int32` (`int` C#) ; vous pouvez toutefois le modifier à votre guise. **Les énumérations C# peuvent être définies de manière similaire pour tous les types système principaux** (`byte`, `short`, `int` ou `long`). Par exemple, si vous souhaitez définir la valeur de stockage sous-jacente d'`EmpTypeEnum` comme un `byte` plutôt qu'un `int`, vous pouvez écrire ce qui suit :

```cs
// Cette fois, EmpTypeEnum mappe à un byte sous-jacent.
enum EnpTypeEnum :byte
{
    Manager = 10,
    Grunt = 1,
    Contractor = 100,
    VicePresident = 9,
}
```

==Modifier le type sous-jacent d'une énumération peut s'avérer utile si vous développez une application .NET Core destinée à être déployée sur un périphérique à faible mémoire et que vous devez économiser cette dernière autant que possible==. **Bien sûr, si vous configurez votre énumération pour utiliser un `byte` comme stockage, chaque valeur doit être comprise dans sa plage !** Par exemple, la version suivante d'`EmpTypeEnum` entraînera une erreur de compilation, car la valeur `999` ne peut pas tenir dans la plage d'un `byte` :

```cs
// Erreur à la compilation! 999 est trop grand pour un byte
enum EmpTypeEnum : byte
{
	Manager = 10,
	Grunt = 1,
	Contractor = 100,
	VicePresident = 999
}
```

## Déclaration des variables d'énumération

Une fois la plage et le type de stockage de votre énumération définis, **vous pouvez l'utiliser à la place des nombres magiques**. ==Les énumérations n'étant rien d'autre qu'un type de données défini par l'utilisateur, vous pouvez les utiliser comme valeurs de retour de fonction, paramètres de méthode, variables locales, etc==. Supposons que vous disposiez d'une méthode nommée `AskForBonus()`, prenant une variable `EmpTypeEnum` comme unique paramètre. En fonction de la valeur du paramètre entrant, vous afficherez une réponse appropriée à la demande de prime.

```cs
Console.Title = "Fun With Enums";
Console.WriteLine("+++++ Fun With Enums *****");

// Crée une variable EnptypeEnum.
EmpTypeEnum emp = EmpTypeEnum.Contractor;
AskForBonus(emp);
Console.ReadLine();

// Enumerations comme paramètres.
static void AskForBonus(EmpTypeEnum e)
{
    switch (e)
    {
        case EmpTypeEnum.Manager:
            Console.WriteLine("How about stock options instead?");
            ;
            break;
        case EmpTypeEnum.Grunt:
            Console.WriteLine($"You have ggot to be kidding...");
            break;
        case EmpTypeEnum.Contractor:
            Console.WriteLine($"You already get enough cash...");
            break;
        case EmpTypeEnum.VicePresident:
            Console.WriteLine($"VERY GOOD, sir!");
            break;
    }
}
```

Notez que lorsque vous attribuez une valeur à une variable d'énumération, **vous devez définir le nom de l'énumération (`EmpTypeEnum`) sur la valeur (`Grunt`)**. ==Les énumérations étant un ensemble fixe de paires nom-valeur, il est illégal de définir une variable d'énumération sur une valeur qui n'est pas définie directement par le type énuméré==.

```cs
static void ThisMethodWillNotCompile()
{
    // Erreur! SalesManager n'est pas dans l'enum EmpTypeEnum!
    EmpTypeEnum emp = EmpTypeEnum.SalesManager;

    // Erreur ! Oublié de limiter la valeur Grunt à l'énumération EmpTypeEnum !
    emp = Grunt;
}
```

## Utilisation du type `System.Enum`

L'intérêt des énumérations .NET Core réside dans le fait qu'elles tirent leurs fonctionnalités du type de classe `System.Enum`. **Cette classe définit plusieurs méthodes permettant d'interroger et de transformer une énumération donnée**. ==Une méthode utile est la méthode statique `Enum.GetUnderlyingType()`, qui, comme son nom l'indique, renvoie le type de données utilisé pour stocker les valeurs du type énuméré== (`System.Byte` dans le cas de la déclaration `EmpTypeEnum` actuelle).

```cs
// Affiche le contenant de l'énumération.
Console.WriteLine(
    $"{nameof(EmpTypeEnum)} uses a "
        + $"{Enum.GetUnderlyingType(emp.GetType())} for storage"
);
```

**La méthode `Enum.GetUnderlyingType()` demande de passer un `System.Type` comme premier paramètre**. Comme expliqué en détail au [[Chapitre 17|Chapitre 17]], **`Type` représente la description des métadonnées d'une entité .NET Core donnée**.

==Une manière possible pour obtenir des métadonnées (comme indiqué précédemment) consiste à utiliser la méthode `GetType()`, commune à tous les types des bibliothèques de classes de base .NET Core. Une autre approche consiste à utiliser l'opérateur `typeof` en C#. L'un des avantages de cette méthode est qu'il n'est pas nécessaire de disposer d'une variable de l'entité dont vous souhaitez obtenir la description des métadonnées==.

```cs
// la même chose, en utilisant typeof() au lieu de object.GetType()
Console.WriteLine(
    $"{nameof(EmpTypeEnum)} uses a "
        + $"{Enum.GetUnderlyingType(typeof(EmpTypeEnum))}"
);
```

## Découverte dynamique des paires nom-valeur d'une énumération

Outre la méthode `Enum.GetUnderlyingType()`, **toutes les énumérations C# prennent en charge une méthode nommée `ToString()`, qui renvoie le nom de la chaîne de la valeur de l'énumération actuelle**. Le code suivant en est un exemple :

```cs
EnpTypeEnum emp = EnpTypeEnum.Contractor;
...
// Affiche "emp is a contractor".
Console.WriteLine($"{nameof(emp)} is a {emp.ToString()}");
Console.ReadLine();
```

**Si vous souhaitez connaître la valeur d'une variable d'énumération donnée, plutôt que son nom, vous pouvez simplement convertir la variable enum en fonction du type de stockage sous-jacent.** Voici un exemple :

```cs
EnpTypeEnum emp = EnpTypeEnum.Contractor;
...
// Prits out "Contractor = 100".
Console.WriteLine($"{emp.ToString()} = {(byte)emp}");
Console.ReadLine();
```

>[!Note] 
>La méthode statique `Enum.Format()` offre des options de formatage plus précises en spécifiant l'indicateur de format souhaité. Consultez la [documentation Microsoft](https://learn.microsoft.com/en-us/dotnet/standard/base-types/enumeration-format-strings) pour obtenir la liste complète des indicateurs de formatage.
>>[!warning] Comme vu [[#Comprendre les tableaux locaux implicitement typés|précédemment]], il est fortement recommandé de ne plus utilisé l'appel à `ToString()` ou `Format()`, surtout pour les interpolations de chaînes. L'exemple est gardé pour correspondre au livre.

`System.Enum` définit également une autre méthode statique nommée `GetValues()`. Cette méthode renvoie une instance de `System.Array`. **Chaque élément du tableau correspond à un membre de l'énumération spécifiée**. Prenons l'exemple de la méthode suivante, qui affichera chaque paire nom-valeur de toute énumération passée en paramètre :

```cs
// Cette méthode affichera les détails de n'importe quel Enum
static void EvaluateEnum(Enum e)
{
    Console.WriteLine($"=> Information about {e.GetType().Name}");

    Console.WriteLine(
        $"Underlying storage type: {Enum.GetUnderlyingType(e.GetType())}"
    );
    
    // Récupère toute les paire nom-valeur pour le paramètre entrant.
    Array enumData = Enum.GetValues(e.GetType());
    Console.WriteLine($"This enum has {enumData.Length} memebers");

    // Maintenant affiche le nom (string) et la valeur associée
    // En utilisant l'indicateur de formatage D (voir Chapitre 3)
    for (int i = 0; i < enumData.Length; i++)
    {
	    Console.WriteLine("Name: {0}, Value: {0:D}", enumData.GetValue(i));
    }
}
```

Pour tester cette nouvelle méthode, mettez à jour votre code afin de créer des variables de plusieurs types d'énumération déclarés dans l'espace de noms `System` (ainsi qu'une énumération `EmpTypeEnum` pour plus de précision). Le code suivant en est un exemple :

```cs
Console.WriteLine("***** Fun with Enums *****");
...
EmpTypeEnum e2 = EmpTypeEnum.Contractor;

// Ces types sont des enumérations situés dans l'espace de noms System.
DayOfWeek day = DayOfWeek.Monday;
ConsoleColor cc = ConsoleColor.Gray;

EvaluateEnum(e2);
EvaluateEnum(day);
EvaluateEnum(cc);

Console.ReadLine();
```

Une partie de la sortie est montrée ici: 

```
=> Information about DayOfWeek
Underlying storage type: System.Int32
This enum has 7 members.
Name: Sunday, Value: 0
Name: Monday, Value: 1
Name: Tuesday, Value: 2
Name: Wednesday, Value: 3
Name: Thursday, Value: 4
Name: Friday, Value: 5
Name: Saturday, Value: 6
```

Comme vous le verrez au fil de ce texte, **les énumérations sont largement utilisées dans les bibliothèques de classes de base de .NET Core. Lorsque vous utilisez une énumération, n'oubliez jamais que vous pouvez interagir avec les paires nom-valeur grâce aux membres de `System.Enum`.**

## Utilisation des énumérations, l'attribut `Flags` et des opérations binaires

**Les opérations binaires offrent un mécanisme rapide pour traiter des nombres binaires au niveau du bit**. Le [[#Tableau 4-3 Opérations au niveau du bit.|Tableau 4-3]] présente les opérateurs binaires C#, leur fonction et un exemple pour chacun d'eux.

##### Tableau 4-3: Opérations au niveau du bit.

| Opérateur                | Opération                                                       | Example                             |
| ------------------------ | --------------------------------------------------------------- | ----------------------------------- |
| `&` (ET)                 | Copie un bit si il existe dans les deux opérandes               | `0110 & 0100` = `0100` ($4$)        |
| `\|` (OU)                | Copie un bit si il existe dans au moins un des opérandes        | `0110 \| 0100` = `0110` ($6$)       |
| `^` (OU EXCLUSIF)        | Copie un bit si il existe dans seulement un des deux opérandes. | `0110 ^ 0100` = `0010` ($2$)        |
| `~` (complément à un)    | Bascule le bit.                                                 | `~0110` = $-7$ (dus à l'*overflow*) |
| `<<` (Décalage à gauche) | Décale les bits à gauche                                        | `0110 << 1` = `1100` ($12$)         |
| `>>` (Décalage à droite) | Décale les bits à droite.                                       | `0110 >> 1` = `0011` ($3$)          |

Pour les visualiser en action, créez un projet d'application console nommé *FunWithBitwiseOperations*. Mettez à jour le fichier *Program.cs* avec le code suivant :

```cs
using FunWithBitwiseOperations;

Console.WriteLine("***** Fun With Bitwise Operations *****");

Console.WriteLine($"6 & 4 = {6 & 4} | {Convert.ToString(6 & 4,2)}");
Console.WriteLine($"6 | 4 = {6 | 4} | {Convert.ToString(6 | 4,2)}");
Console.WriteLine($"6 ^ 4 = {6 ^ 4} | {Convert.ToString(6 ^ 4,2)}");
Console.WriteLine($"6 << 1 = {6 << 1} | {Convert.ToString(6 << 1,2)}");
Console.WriteLine($"6 >> 1 = {6 >> 1} | {Convert.ToString(6 >> 1,2)}");
// l'appel à ToString() présent dans le livre est supprimé 
// pour correspondre au C# moderne.
Console.WriteLine($"~6 = {~6} | {Convert.ToString(~6,2)}");
Console.WriteLine($"Int.MaxValue {Convert.ToString(int.MaxValue,2)}");
Console.ReadLine();
```

Quand vous exécuter le programme, vous verrez le résultat suivant:

```
***** Fun With Bitwise Operations *****
6 & 4 = 4 | 100
6 | 4 = 6 | 110
6 ^ 4 = 2 | 10
6 << 1 = 12 | 1100
6 >> 1 = 3 | 11
~6 = -7 | 11111111111111111111111111111001
Int.MaxValue 1111111111111111111111111111111
```

Maintenant que vous connaissez les bases des opérations bit à bit, il est temps de les appliquer aux énumérations. Ajoutez un nouveau fichier nommé *ContactPreferenceEnum.cs* et modifiez le code comme suit :

```cs
namespace FunWithBitwiseOperations;

[Flags]
public enum ContactPreferenceEnum
{
	None = 1,
	Email = 2,
	Phone = 4,
	Text = 8
}
```

>[!warning] Il **FAUT** assigné les valeurs en puissance de 2 quand on utilise l'attribut [Flags].

**Notez l'attribut `Flags`. Il permet de combiner plusieurs valeurs d'une énumération en une seule variable.** Par exemple, `Mail` et  `Phone` peuvent être combinés comme suit :

```cs
ContactPreferenceEnum emailAndPhone =
   ContactPreferenceEnum.Email | ContactPreferenceEnum.Phone;
```

>[!tip] On peut aussi déclarer un membre d'énumération avec cette méthodologie:
>```cs
>[Flags]
>public enum ContactPreferenceEnum
>{
>    ...
>    EmailAndPhone = Email | Phone,
>}
>```

**Cela vous permet de vérifier is une des valeurs existe dans la valeur combinée.** Par exemple, si vous souhaitez vérifier la valeur de la variable `ContactPreference` dans la variable `emailAndPhone`, vous pouvez utiliser le code suivant :

```cs
Console.WriteLine($"What does {nameof(emailAndPhone)} variable equals to?");
Console.WriteLine(
    $"None? {(emailAndPhone | ContactPreferenceEnum.None) == emailAndPhone}"
);
Console.WriteLine(
    $"Email? {(emailAndPhone | ContactPreferenceEnum.Email) == emailAndPhone}"
);
Console.WriteLine(
    $"Phone? {(emailAndPhone | ContactPreferenceEnum.Phone) == emailAndPhone}"
);
Console.WriteLine(
    $"Text? {(emailAndPhone | ContactPreferenceEnum.Text) == emailAndPhone}"
);
```

Une fois exécuté, le message suivant s'affiche dans la fenêtre de la console :

```
None? False
Email? True
Phone? True
Text? True
```

# Comprendre les structures (`struct`)

Maintenant que vous comprenez le rôle des types d'énumération, examinons l'utilisation des *structures .NET Core* (ou simplement *structs*). **Les types de structure sont particulièrement adaptés à la modélisation d'entités mathématiques, géométriques et autres entités « atomiques » dans votre application**. ==Une structure (comme une énumération) est un type défini par l'utilisateur ; cependant, les structures ne sont pas simplement une collection de paires nom-valeur. Ce sont plutôt des types pouvant **contenir un nombre illimité de champs de données et de membres agissant sur ces champs**==.

>[!note]- Si vous avez des connaissances en OOP
>Vous pouvez considérer une structure comme un « *type de classe léger* », car les structures permettent de définir un type compatible avec l'encapsulation, **mais ne peuvent pas servir à construire une famille de types apparentés**. ==Elles ne peuvent pas hériter d'autres types de classes ou de structures et ne peuvent pas constituer la base d'une classe==. L'héritage est abordé au [[Chapitre 5#Comprendre le rôle de l'héritage|Chapitre 5]]. ==Les structures peuvent implémenter des interfaces==, abordées au [[Chapitre 8|Chapitre 8]]. Pour construire une famille de types apparentés par héritage, vous devrez utiliser des types de classe.

En apparence, définir et utiliser des structures est simple, mais comme on dit, le diable est dans les détails. Pour comprendre les bases des types de structures, créez un projet nommé *FunWithStructures*. **En C#, les structures sont définies à l'aide du mot-clé `struct`**. Définissez une nouvelle structure nommée `Point`, qui définit deux variables membres de type `int` et un ensemble de méthodes pour interagir avec ces données.

```cs
struct Point
{
    // Champs de la structure.
    public int X;
    public int Y;

    // Ajoute 1 aux positions (X, Y).
    public void Increment()
    {
        X++;
        Y++;
    }
    
    // Soustrait 1 aux positions (X, Y).
    public void Decrement()
    {
        X--;
        Y--;
    }

    // Affiche la position actuelle.
    public void Display()
    {
        Console.WriteLine($"X = {X}, Y = {Y}");
    }
    
    // La méthode Display réécrite.
    // En suivant toutes les recommendation de C# 14
    public readonly void Display() => Console.WriteLine($"X = {X}, Y = {Y}");

}
```

Ici, vous avez défini vos deux champs entiers (`X` et `Y`) à l'aide du mot-clé `public`, un modificateur de contrôle d'accès (le [[Chapitre 5#Comprendre les modificateurs d'accès (MaJ C 7.2)|chapitre 5]] poursuit cette discussion). **Déclarer des données avec le mot-clé `public` garantit à l'appelant un accès direct aux données d'une variable `Point` donnée** (via l'opérateur point(`.`)).

>[!Tip] Bonnes pratiques
>Définir des données publiques au sein d'une classe ou d'une structure est généralement considéré comme une mauvaise pratique. *Il est préférable de définir des données privées, accessibles et modifiables via des propriétés publiques*. Ces détails seront examinés au [[Chapitre 5#Utilisation des propriétés dans une définition de classe|chapitre 5]].

Voici le code qui utilise le type `Point` pour un essai.:

```cs
Console.Title = "A First Look at Structures";
Console.WriteLine("**** A First Look at Structures\n");

// Crée un Point initial.
Point myPoint;
myPoint.X = 349;
myPoint.Y = 76;
myPoint.Display();

// Ajuste les valeurs de X et Y.
myPoint.Increment();
myPoint.Display();

Console.ReadLine();

```

Le résultat est conforme à ce que vous attendez.

```
*****  A First Look at Structures *****

X = 349, Y = 76
X = 350, Y = 77
```

==Il existe certaines règles concernant les structures==. **Premièrement, une structure ne peut hériter d'une autre classe ou type de structure et ne peut servir de base à une classe**. ==Les structures peuvent implémenter des interfaces==.

## Création de variables de structure

Pour créer une variable de structure, **plusieurs options s'offrent à vous**. ==Il suffit de créer une variable `Point` et d'affecter chaque donnée de champ public avant d'appeler ses membres==. **Si vous *n'affectez pas* chaque donnée de champ public** (`X` et `Y` dans ce cas) **avant d'utiliser la structure, vous recevrez une erreur de compilation**.

```cs
// Erreur! n'as pas assigné de valeur à Y.
Point p1;
p1.X = 10;
p1.Display();

// OK! chaque champs est assignés avant l'utilisation.
Point p2;
p2.X = 10;
p2.Y = 10;
p2.Display();
```

**Vous pouvez également créer des variables de structure à l'aide du mot-clé C# `new`, qui invoquera le *constructeur par défaut* de la structure**. Par définition, ==un constructeur par défaut ne prend aucun argument. L'avantage d'invoquer le constructeur par défaut d'une structure est que chaque donnée de champ est automatiquement définie à sa valeur par défaut==.

```cs
// Assigne les valeurs par défaut à chaque champs
// en utilisant le constructeur par défaut.
Point p1 = new Point();

// Affiche X=0, Y=0.
p1.Display();
```

## Les constructeurs de structure (MaJ C# 10)

**Il est également possible de concevoir une structure avec un *constructeur personnalisé***. Cela vous permet de ==spécifier les valeurs des données de champ lors de la création des variables, plutôt que de devoir définir chaque donnée membre champ par champ==. Le [[Chapitre 5#Comprendre les constructeurs|chapitre 5]] fournira une analyse détaillée des constructeurs ; à titre d'illustration, mettez à jour la structure `Point` avec le code suivant :

```cs
struct Point
{
    // Champs de la structure.
    public int X;
    public int Y;

    // Un constructeur personnalisé
    public Point(int xPos, int yPos)
    {
		X = xPos;
		Y = yPos;
    }
    ...
}
```

Avec cela, vous pouvez maintenant créer des variable `Point` comme suit:

```cs
// Appel du constructeur personnalisé.
Point p2 = new Point(50, 60);
// Affiche X=50, Y=60.
p2.Display();

```

Avant C# 10, il était impossible de déclarer un constructeur sans paramètre (c.-à-d. par défaut) sur une structure, car cela était pris en charge par l'implémentation des types de structures. **Désormais, vous pouvez créer des variables de type `Point`, comme suit** :

```cs
// Ici, contrairement à l'exemple de p1,
// on appelle le constructeur sans paramètres
// et non pas le constructeur par défaut.
Point p3 = new Point()
p3.Display()
```

Quel que soit le constructeur choisi, avant C# 10, **il était impossible de déclarer un constructeur sans paramètre (c.-à-d. par défaut) sur une structure**, car cela était prévu à l'implémentation des types de structure. **Désormais, c'est possible, à condition d'attribuer une valeur à tous les types de valeur avant la fin du code du constructeur**. Ainsi, vous pouvez mettre à jour la structure `Point` comme suit :

```cs
struct Point
{
// Omis pour brièveté.
	// constructeur sans paramètres (C#10)
	public Point()
	{
		X = 0;
		Y = 0;
	}
	
	// Un constructeur personnalisés.
	public Point(int xPos, int yPos)
	{
		X = xPos;
		Y = yPos;
	}
	...
}
```

>[!note] C#10 et .NET 6 introduisent le `record struct`, qui sera couvert dans le [[Chapitre 5|chapitre 5]]

### Le constructeur primaires de structure (Nouveauté C# 12)

> Texte en partie généré par Gemini.

Avec la sortie de .NET 8 / C# 12, les `struct` bénéficie des *constructeurs primaires*. C'est une évolution d'une fonctionnalité qui existait déjà pour les `record` depuis C# 9, mais qui a été étendue aux types classiques pour réduire le code répétitif (*boilerplate*).

>[!note] Le type  `record` sera abordé dans le [[Chapitre 5|Chapitre 5]].

```cs
// Nouveauté C# 12: le constructeur primaire
struct Point(int xPos, int yPos)
{
    public int X = xPos;
    public int Y = yPos;

    // C# 10 : Le constructeur sans paramètres DOIT appeler le primaire
    public Point()
        : this(0, 0) { }

    // Le constructeur personnalisé (int, int) est devenu inutile
    // car le constructeur primaire le remplace déjà.

    public void Increment()
    {
        X++;
        Y++;
    }

    public void Decrement()
    {
        X--;
        Y--;
    }

    public readonly void Display() => Console.WriteLine($"X = {X}, Y = {Y}");
}
```

>[!Attention]
>1. **Appel obligatoire** : Le constructeur `public Point()` ne peut plus initialiser `X` et `Y` directement dans son corps ; ==il doit déléguer le travail au constructeur primaire avec `: this(0, 0)`== (expliqué en détail dans le [[Chapitre 5#Comprendre le rôle du mot-clé `this`|Chapitre 5]].
>2. **Capture des paramètres** : Dans une `struct` (contrairement aux `class`), les paramètres `xPos` et `yPos` sont *capturés*. Si tu modifies `X` plus tard dans `Increment()`, ==la valeur originale de `xPos` reste inchangée dans la mémoire de la structure==.

## Utilisation des initialiseurs de champs (Nouveauté C# 10) 

Nouveauté de C# 10 : **les champs de structure peuvent être initialisés dès leur déclaration**. Modifiez le code comme suit : initialise `X` avec la valeur $5$ et `Y` avec la valeur $7$ :

```cs
struct Point
{
	// Champs de la structure.
	public int X = 5;
	public int Y = 7;
	
	// Ommis pour brièveté.
}
```

Avec cette mise à jour, les constructeur sans paramètres n'ont désormais plus besoin d'initialiser les champs `X` et `Y`.

```cs
struct Point
{
	// Ommis pour brièveté.
	// Construdteur sans paramètres.
	public Point() { }
	// Ommis pour brièveté.
}
```

## Utilisation des structures en lecture seule (C# 7.2)

Les structures peuvent également être marquées comme étant en lecture seule si elles doivent être *immuables*. **Les objets immuables doivent être configurés dès la construction et, comme ils ne peuvent pas être modifiés, ils peuvent être plus performants**. ==Lorsqu'une structure est déclarée en lecture seule, toutes ses propriétés doivent également l'être==. Mais vous vous demandez peut-être comment définir une propriété (puisque toutes les propriétés doivent figurer sur une `struct`) si elle est en lecture seule ? La réponse est que **la valeur doit être définie lors de la construction de la structure**.

Mettez à jour la classe `point` avec l'exemple suivant :

```cs
readonly struct ReadOnlyPoint
{
    // Propriétés de la structure.
    // La typo du livre à été corigés
    public int X { get; }
    public int Y { get; }

    // Affiche la position actuelle et le nom.
    public void Display()
    {
        Console.WriteLine($"X = {X}, Y = {Y}");
    }

    public ReadOnlyPoint(int xPos, int yPos)
    {
        X = xPos;
        Y = yPos;
    }
}
```

Les méthodes `Increment` et `Decrement` ont été supprimées, car les variables sont en lecture seule. **Notez également les deux propriétés `X` et `Y`. Au lieu de les définir comme des *champs*, elles sont créées comme des *propriétés automatiques en lecture seule***. Les propriétés automatiques sont abordées au [[Chapitre 5#Comprendre les propriétés automatiques|Chapitre 5]].

## Utilisation des membres en lecture seule (Nouveauté C# 8.0)

Nouveauté de C# 8.0 : **vous pouvez déclarer les champs individuels d'un `struct` en `readonly`**. ==Cette fonctionnalité est plus précise que de rendre la structure entière en lecture seule. Le modificateur `readonly` peut être appliqué aux méthodes, aux propriétés et aux accesseurs de propriété==({`get` et `set`}). Ajoutez le code de structure suivant à votre ficher, en dehors de *Crogram.cs*:

```cs
struct PointWithReadOnly
{
    // Champs de la structure.
    public int X;
    public readonly int Y;
    public readonly string Name;

    // Affiche la position actuelle et le nom.
    public readonly void Display()
    {
        Console.WriteLine($"X = {X}, Y = {Y}, Name = {Name}");
    }

    // Un constructuer personnalisé.
    public PointWithReadOnly(int xPos, int yPos, string name)
    {
        X = xPos;
        Y = yPos;
        Name = name;
    }
}
```

Pour utiliser cette nouvelle structure, ajouter le code suivant dans la déclaration de haut niveau:

```cs
PointWithReadOnly p4 = new(50, 60, "Point w/RO");
p4.Display();
```

## Utilisation des `ref struct` (Nouveauté C# 7.2)

Également ajouté en C# 7.2, **le modificateur `ref` peut être utilisé lors de la définition d'une structure**. ==Cela nécessite que toutes les instances de la structure soient allouées sur la pile (*stack*) et ne puissent pas être assignées comme propriété d'une autre classe==. La raison technique est que les structures `ref` ne peuvent pas être référencées depuis le tas (*heap*). ==La différence entre la pile et le tas est abordée dans la section suivante==.

>[!Warning]
> Voici quelques limitations supplémentaires des structures `ref` :
>
> - Elles ne peuvent pas être affectées à une variable de type `object` ou `dynamic`, et ne peuvent pas être de type interface.
> - Elles ne peuvent pas implémenter d'interfaces.
> - Elles ne peuvent pas être utilisées comme propriété d'une structure non-`ref`.
> - Elles ne peuvent pas être utilisées dans des méthodes asynchrones, des itérateurs, des expressions lambda ou des fonctions locales.

Le code suivant, qui crée une structure simple puis tente de créer une propriété dans cette structure, typée sur une structure `ref`, ne compilera pas :

```cs
struct NormalPoint
{
	// Ceci ne compilera pas
	public PointWithRef PropPointer {get; set;}
}
```

Notez que les modificateurs `readonly` et `ref` peuvent être combinés pour bénéficier des avantages et des restrictions des deux.

## Utilisation des `ref struct` jetables (Nouveauté C# 8.0)

Comme indiqué dans la section précédente, **les structures `ref` (et les structures `ref` en lecture seule) ne peuvent pas implémenter d'interface et ne peuvent donc pas implémenter `IDisposable`**. ==Nouveauté de C# 8.0 : les structures `ref` et les structures `ref` en lecture seule peuvent être rendues jetables en ajoutant une méthode publique void `Dispose()`==.

Ajoutez la définition de structure suivante au fichier *Program.cs* :

```cs
ref struct DisposableRefStruct
{
    public int X;
    public readonly int Y;

    public readonly void Display()
    {
        Console.WriteLine($"X = {X}, Y = {Y}");
    }

    // Un constructeur personnalisé
    public DisposableRefStruct(int xPos, int yPos)
    {
        X = xPos;
        Y = yPos;
        Console.WriteLine("Created!");
    }

    public void Dispose()
    {
        // Nettoyer les ressources ici.
        Console.WriteLine("Disposed!");
    }
}
```

Ensuite, ajoutez ce qui suit à la fin des instructions de niveau supérieur pour créer et supprimer la nouvelle structure :

```cs
var s = new DisposableRefStruct(50, 60);
s.Display();
s.Dispose();
```

>[!Info] La durée de vie et leur élimination des `Object` sont traitées en profondeur dans le [[Chapitre 9|Chapitre 9]] 

**Pour approfondir votre compréhension de l'allocation de pile (*stack*) et de tas (*heap*), vous devez explorer la distinction entre un type de valeur .NET Core et un type de référence .NET Core.**

# Comprendre les types de valeur et les types de référence

>[!warning] Attention: 
>La discussion suivante sur les types valeur et référence suppose que vous possédez une expérience en programmation orientée objet. 
>
>Dans le cas contraire, vous pouvez passer directement à la section [[#Comprendre les types nullable en C]] de ce chapitre et y revenir après avoir lu le [[Chapitre 5#Définir les piliers de la POO|Chapitre 5]] et le [[Chapitre 6|Chapitre 6]].

Contrairement aux tableaux, aux chaînes ou aux énumérations, les structures C# n'ont pas de représentation portant le même nom dans la bibliothèque .NET Core (**il n'existe pas de classe `System.Structure`**), **mais sont implicitement dérivées de `System.ValueType`**. ==Le rôle de `System.ValueType` est de garantir que le type dérivé== (par exemple, toute structure) ==est alloué sur la *stack*, plutôt que sur le *heap* récupéré par le *garbage collector*==. En d'autres termes, **les données allouées sur la *stack* peuvent être créées et détruites rapidement, car leur durée de vie est déterminée par la portée qui les définit. Les données allouées sur le *heap*, en revanche, sont surveillées par le *garbage collector* .NET Core et leur durée de vie est déterminée par de nombreux facteurs, qui seront examinés au [[Chapitre 9|Chapitre 9]]**.

Fonctionnellement, **le seul but de `System.ValueType` est de *surcharger* les méthodes virtuelles définies par `System.Object` afin d'utiliser une sémantique basée sur les valeurs plutôt que sur les références**. Comme vous le savez peut-être, ==la surcharge consiste à modifier l'implémentation d'une méthode virtuelle (ou éventuellement abstraite) définie dans une classe de base==. La classe de base de `ValueType` est `System.Object`. ==En fait, les méthodes d'instance définies par `System.ValueType` sont identiques à celles de `System.Object`==.

```cs
// Les structures et les énumérations étendent implicitement System.ValueType.
public abstract class ValueType : object
{
	public virtual bool Equals(object obj);
	public virtual int GetHashCode();
	public Type GetType();
	public virtual string ToString();
}
```

**Étant donné que les types valeur utilisent une sémantique basée sur les valeurs, la durée de vie d'une structure** (qui inclut tous les types de données numériques [`int`, `float`], ainsi que toute `enum` ou `struct`) **est prévisible. Lorsqu'une variable `struct` sort de la portée définie, elle est immédiatement supprimée de la mémoire**.

```cs
// Les structures locales sont retirées de la pile
// lorsqu'une méthode se termine.
static void LocalValueType()
{
	// Souvenir! "int" est réellement une structure System.Int32.
	int i = 0;
	
	// Souvenir! Point est un type de structure.
	Point p = new Point();
}  // "i" et "p" sont retiré de la pile ici!
```

## Utilisation des types de valeur, des types de référence et de l'opérateur d'affectation

**Lorsque vous affectez un type de valeur à un autre, une copie membre par membre des données du champ est réalisée**. Dans le cas d'un type de données simple tel que `System.Int32`, le seul membre à copier est la valeur numérique. Cependant, ==dans le cas de votre `Point`, les valeurs `X` et `Y` sont copiées dans la nouvelle variable de structure==. À titre d'exemple, créez un nouveau projet d'application console nommé *FunWithValueAndReferenceTypes*, puis copiez votre définition de `Point` précédente dans votre nouvel espace de noms. Ajoutez ensuite la fonction locale suivante à vos instructions de niveau supérieur :

>[!note]- Voici la structure `Point` pour cette section:
>```cs
>namespace FunWithValueAndReference;
>
>struct Point(int xPos, int yPos, string name = "")
>{
> 	// Champs de la structure.
> 	public int X = xPos;
> 	public readonly int Y = yPos;
> 	public readonly string Name = name;
>	
> 	// Affiche la position actuelle et le nom.
> 	public readonly void Display()
> 	{
> 		   Console.WriteLine($"{GetType().Name}: X = {X}, Y = {Y}, Name = {Name}");
> 	}
>}
>```

```cs
using FunWithValueAndReference;

Console.Title = "Fun with Value and Reference";
Console.WriteLine("***** Fun with Value and Reference *****");

ValueTypeAssignment();
ReferenceTypeAssignment();

// L'affectation de deux types de valeurs intrinsèques donne
// deux variables indépendantes sur la pile
static void ValueTypeAssignment()
{
    Console.WriteLine("=> Assigning value types\n");

    Point p1 = new Point(10, 10);
    Point p2 = p1;

    // Affiche les deux points.
    Console.Write($"{nameof(p1)}: ");
    p1.Display();
    Console.Write($"{nameof(p2)}: ");
    p2.Display();

    // Changement de p1.X et affiche encore. p2.X n'est pas changé.
    p1.X = 100;

    Console.WriteLine("\n=> Changed p1.X\n");

    Console.Write($"{nameof(p1)}: ");
    p1.Display();
    Console.Write($"{nameof(p2)}: ");
    p2.Display();

    Console.WriteLine();
}
```

Ici, vous avez créé une variable de type `Point` (nommée `p1`) qui est ensuite affectée à un autre `Point` (`p2`). **`Point` étant un type valeur, vous disposez de deux copies de ce type sur la *stack*, chacune pouvant être manipulée indépendamment**. Par conséquent, ==lorsque vous modifiez la valeur de `p1.X`, la valeur de `p2.X` n'est pas affectée==.

```
=> Assigning value types

X = 10, Y = 10, Name =
X = 10, Y = 10, Name =

=> Changed p1.X

X = 100, Y = 10, Name =
X = 10, Y = 10, Name =
```

Contrairement aux types valeur, **lorsque vous appliquez l'opérateur d'affectation aux types référence** (c'est-à-dire à toutes les instances de classe), **vous redirigez ce vers quoi pointe la variable de référence en mémoire**. Par exemple, créez un nouveau type de classe nommé `PointRef`, possédant les mêmes membres que la structure `Point`, à l'exception du constructeur renommant celui-ci.

```cs
// Les classes sont toujours de type par référence.
// Rappel: les constructeurs primaires de C# 12
// Fonctionne aussi pour les classes.
class PointRef
{
    // Les même membres que la structure Point...
    // Assurez-vous de changer le nom de votre constructeur en PointRef !
    public int X;
    public readonly int Y;
    public readonly string Name;

    public PointRef(int xPos, int yPos, string name = "")
    {
        X = xPos;
        Y = yPos;
        Name = name;
    }

    // Pour les classes, le modificateur readonly
    // ne s'applique UNIQUEMENT que sur les champs.
    public void Display() =>
        Console.WriteLine($"{GetType().Name}: X = {X}, Y = {Y}, Name = {Name}");
}
```

Utilisez maintenant votre type `PointRef` dans la nouvelle méthode suivante. Notez qu'au-delà de l'utilisation de la classe `PointRef`, et non de la structure `Point`, le code est identique à celui de la méthode `ValueTypeAssignment()`.

```cs
static void ReferenceTypeAssignment()
{
    Console.WriteLine("=> Assigning value types\n");

    PointRef p1 = new PointRef(10, 10);
    PointRef p2 = p1;

    // Affiche les deux points.
    Console.Write($"{nameof(p1)}: ");
    p1.Display();
    Console.Write($"{nameof(p2)}: ");
    p2.Display();

    // Modifie p1.X et affiche encore.
    // p2.X sera modifié (PointRef et un type de référence).
    p1.X = 100;

    Console.WriteLine("\n=> Changed p1.X\n");

    Console.Write($"{nameof(p1)}: ");
    p1.Display();
    Console.Write($"{nameof(p2)}: ");
    p2.Display();

    Console.WriteLine();
}
```

**Dans ce cas, vous avez deux références pointant vers le même objet sur le *heap* géré. Par conséquent, lorsque vous modifiez la valeur de `X` à l'aide de la référence `p1`, `p2.X` renvoie la même valeur**. En supposant que vous ayez appelé cette nouvelle méthode, votre résultat devrait ressembler à ceci :

```
=> Assigning Reference types

X = 10, Y = 10, Name =
X = 10, Y = 10, Name =

=> Changed p1.X

X = 100, Y = 10, Name =
X = 100, Y = 10, Name =
```

## Utilisation de types de valeur contenant des types de référence

Maintenant que vous comprenez mieux les différences fondamentales entre les types de valeur et les types de référence, examinons un exemple plus complexe. ==Supposons que vous disposiez du type de référence (classe) suivant qui maintiens un `string` d'information pouvant être définie à l'aide d'un constructeur personnalisé== :

```cs
// Utilisation du constructeur primaire
// pour la concision.
class ShapeInfo(string info)
{
	public string InfoString = info;
}
```

Supposons maintenant que vous souhaitiez inclure une variable de ce type de classe dans un type de valeur nommé `Rectangle`. **Pour permettre à l'appelant de définir la valeur de la variable membre `ShapeInfo` interne, vous devez également fournir un constructeur personnalisé**. Voici la définition complète du type `Rectangle` :

```cs
namespace FunWithValueAndReference;

// Ici, j'utilise le constructeur personnalisé
// pour un soucis de simplicité et de rester proche
// du livre.
struct Rectangle
{
    // La structure Rectangle contient un membre de type référence.
    public ShapeInfo RectInfo;
    public int RectTop, RectLeft, RectBottom, RectRight;

    public Rectangle(string info, int top, int left, int bottom, int right)
    {
        RectInfo = new ShapeInfo(info);

        RectTop = top;
        RectBottom = bottom;
        RectLeft = left;
        RectRight = right;
    }

    public readonly void Display()
    {
        Console.WriteLine(
            $"String = {RectInfo.InfoString}, Top = {RectTop}"
                + $", Bottom = {RectBottom}, Left = {RectLeft}, Right = {RectRight}"
        );
    }
}
```

À ce stade, vous avez inclus un type référence dans un type valeur. La question cruciale devient alors : « **Que se passe-t-il si vous affectez une variable `Rectangle` à une autre ?**» Compte tenu de vos connaissances sur les types valeur, vous auriez raison de supposer que les données entières (qui sont effectivement une structure, `System.Int32`) devraient être une entité indépendante pour chaque variable `Rectangle`. Mais qu’en est-il du type référence interne ? L’*état* de l’objet sera-t-il entièrement copié, ou la référence à cet objet sera-t-elle copiée ? Pour répondre à cette question, définissez la méthode suivante et appelez-la :

```cs
static void ValueTypeContainingRefType()
{
	Console.WriteLine("-> Creating r1");
	Rectangle r1 = new Rectangle("First Rect", 10, 10, 50, 50);
	
	// assigne maintenant un nouveau Rectance à r1
	Console.WriteLine("-> Assigning r2 to r1");
	Rectangle r2 = r1;

	// Change des valeurs de r2	
	Console.WriteLine("-> Changing values of r2");
	r2.RectInfo.InfoString = "This is new info!";
	r2.RectBottom = 4444;

	// Affiche les valeurs de chaques rectangles.	
	r1.Display();
	r2.Display();
}
```

La sortie est montré ici bas:

```
-> Creating r1
-> Assigning r2 to r1
-> Changing values of r2
String = This is new info!, Top = 10, Bottom = 50, Left = 10, Right = 50
String = This is new info!, Top = 10, Bottom = 4444, Left = 10, Right = 50
```

>[!warning] Cette partie est très importante!

Comme vous pouvez le constater, ==lorsque vous modifiez la valeur de la chaîne d'information à l'aide de la référence `r2`, la référence `r1` affiche la même valeur==. **Par défaut, lorsqu'un type de valeur contient d'autres types de référence, l'affectation produit une copie des références. Ainsi, vous disposez de deux structures indépendantes, chacune contenant une référence pointant vers le même objet en mémoire** (c'est-à-dire une *copie superficielle (shallow copy)*). Pour effectuer une *copie profonde (deep copy)*, où l'état des références internes est entièrement copié dans un nouvel objet, une approche consiste à implémenter l'interface `ICloneable` (comme vous le ferez au [[Chapitre 8|Chapitre 8]]).

## Passage de types référence par valeur

Comme indiqué précédemment dans ce chapitre, **les types référence ou types valeur peuvent être passés en paramètres aux méthodes**. ==Cependant, le passage par référence d'un type référence (comme une classe) diffère grandement d'un passage par valeur.==. Pour comprendre cette distinction, supposons que vous ayez une classe `Person` simple définie dans un nouveau projet d'application console nommé *FunWithRefTypeValTypeParams*, comme suit :

```cs
class Person
{
	public string personName;
	public int personAge;
	
	// Constructeurs.
	public Person() { }
	
	public Person(string name, int age)
	{
		personName = name;
		personAge = age;
	}
	
	public void Display()
	{
		Console.WriteLine($"Name: {personName}, Age: {personAge}");
	}
}
```

Et si vous créiez une méthode permettant à l'appelant d'envoyer l'objet `Person` par valeur (==notez l'absence de modificateurs de paramètres, tels que `out` ou `ref`==) ?

```cs
static void SendApersonByValue(Person p)
{
	// Change l'âge de "p" ?
	p.personAge = 99;

	// Est-ce que l'appelant voie ce réassignement ?	
	p = new Person("Nikki", 99);
}
```

Notez comment la méthode `SendAPersonByValue()` tente de réaffecter la référence `Person` entrante à un nouvel objet `Person`, ainsi que de modifier certaines données d'état. Testons maintenant cette méthode avec le code suivant :

```cs
// Passer des type référence par valeur.

Console.Title = $"Passing {nameof(Person)} Object by Value";
Console.WriteLine("***** Passing Person Object By Value *****");

Person fred = new("Fred", 12);

Console.WriteLine("\nBefore by value call, Person is:");
fred.Display();

SendApersonByValue(fred);

Console.WriteLine("\nAfter by value call, Person is:");
fred.Display();

Console.ReadLine();
```

Voici le résultat de cet appel :

```
***** Passing Person Object By Value *****

Before by value call, Person is:
Name: Fred, Age: 12

After by value call, Person is:
Name: Fred, Age: 99
```

**Comme vous pouvez le constater, la valeur de `personAge` a été modifiée**. Ce comportement, évoqué précédemment, devrait être plus clair maintenant que vous comprenez le fonctionnement des types de référence. Puisque vous avez pu modifier l'état de la personne entrante, qu'est-ce qui a été copié ? La réponse : une copie de la référence à l'objet de l'appelant. **Par conséquent, comme la méthode  `SendAPersonByValue()` pointe vers le même objet que l'appelant, il est possible de modifier les données d'état de l'objet. En revanche, il est impossible de réaffecter l'objet vers lequel *pointe* la référence**.

>[!warning] Important
>>[!Tldr] Pour résumer 
>>Par défaut, C# passe n'importe quel type de membre **par valeur** (énumérations, structures, classes, records, entiers, octets, objects, ...) aux méthodes.

## Passer des types de référence par référence

Supposons maintenant que vous disposez d'une méthode `SendAPersonByReference()` qui transmet un type référence *par référence* (notez le modificateur de paramètre `ref`).

```cs
static void SendAPersonByReference(ref Person p)
{
    // change une donnée de "p".
    p.personAge = 555;
    
    // "p" pointent maintenant vers un nouvel object dans le heap!
    p = new Person("Nikki", 999);
}
```

Comme on pouvait s'y attendre, cela offre une flexibilité totale quant à la façon dont l'appelé peut manipuler le paramètre entrant. **Non seulement l'appelé peut modifier l'état de l'objet, mais il peut également, s'il le souhaite, réaffecter la référence à un nouvel objet `Person`**. Considérons maintenant le code mis à jour suivant :

```cs
Console.WriteLine("***** Passing Person Ojbect By Reference *****");

Person mel = new Person("Mel", 23);

Console.WriteLine("\nBefore by ref call, Person is:");
mel.Display();

SendAPersonByReference(ref mel);

Console.WriteLine("\nAfter by ref call, Person is:");
mel.Display();

Console.ReadLine();
```

Notez la sortie suivante :

```
***** Passing Person Ojbect By Reference *****

Before by ref call, Person is:
Name: Mel, Age: 23

After by ref call, Person is:
Name: Nikki, Age: 999
```

==Comme vous pouvez le constater, un objet nommé `Mel` revient après l'appel sous la forme d'un objet nommé `Nikki`, car la méthode a pu modifier le point de la référence entrante en mémoire==. 

>[!tip] **La règle d'or à retenir lors du passage de types de référence est la suivante** :
>
>- Si un type de référence est transmis par référence, l'appelé peut modifier les valeurs des données d'état de l'objet, ainsi que l'objet auquel il fait référence.
>
>- Si un type de référence est transmis par valeur, l'appelé peut modifier les valeurs des données d'état de l'objet, mais pas l'objet auquel il fait référence.

## Derniers détails concernant les types de valeur et les types de référence

Pour conclure, consultez le [[#Tableau 4-4 Comparaison entre les types de valeur et les types de référence.|Tableau 4-4]], qui résume les principales distinctions entre les types de valeur et les types de référence.

>[!note]- Note personnel
>Dans mes notes nommée "*Notes code & CLI*", dans la partie C#, J'ai représenté en image la différence entre type valeurs et type de référence pour plus de clarté.

##### Tableau 4-4: Comparaison entre les types de valeur et les types de référence.
| Question                                                       | Type valeur                                                                                                                           | Type référence                                                                                                                                       |
| -------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| Où sont allouée les objets ?                                   | Dans la pile (*stack*)                                                                                                                | Dans le tas (*heap*) géré  (*garbage collector*)                                                                                                     |
| Comment une variable est-elle représentée ?                    | Les variable de type valeur sont une copie locale                                                                                     | Les variables de type référence pointent vers la mémoire occupée par l'instance qui l'occupe.                                                        |
| Quel est le type de base ?                                     | Étend implicitement `System.ValueType`.                                                                                               | Peut dériver de tout autre type (sauf `System.ValueType`), si ce type n'est pas `sealed` (plus de détails à ce sujet au [[Chapitre 6\|Chapitre 6]]). |
| Ce type peut-il servir de base à d’autres types ?              | Non. Les type de valeur sont tout le temps `sealed` et ne peuvent pas être hérité.                                                    | Oui. Si le type n'est pas scellé, il peut servir de base à d'autres types.                                                                           |
| Quel est le comportement par défaut du passage de paramètres ? | Les variables sont transmises par valeur (c'est-à-dire qu'une copie de la variable est transmise à la fonction appelée).              | **Pour les types de référence, la référence (l'adresse mémoire) est copiée par valeur**.                                                             |
| Ce type peut-il surchargé `System.Object.Finalize()` ?         | Non.                                                                                                                                  | Oui, indirectement (plus de détail dans le [[Chapitre 9\|Chapitre 9]])                                                                               |
| Puis-je définir des constructeurs pour ce type ?               | Oui, mais le constructeur par défaut est réservé (c'est-à-dire que vos constructeurs personnalisés doivent tous avoir des arguments). | Mais bien sur!.                                                                                                                                      |
| Quand les variables de ce type meurent-elles ?                 | Lorsqu'ils sortent de la portée définie.                                                                                              | Lorsque l'objet est récupéré par le *garbage collector* (voir [[Chapitre 9\|Chapitre 9]]).                                                           |

**Malgré leurs différences, les types de valeur et les types de référence peuvent tous deux implémenter des interfaces et prendre en charge un nombre illimité de champs, de méthodes, d'opérateurs surchargés, de constantes, de propriétés et d'événements**.

# Comprendre les types nullable en C# 

Examinons le rôle du *type de données annulable* à l'aide d'un projet d'application console nommé `FunWithNullableValueTypes`. **Comme vous le savez, les types de données C# ont une plage fixe et sont représentés comme un type dans l'espace de noms `System`.** Par exemple, le type de données `System.Boolean` peut se voir attribuer une valeur de l'ensemble {`true`, `false`}. ==Rappelons que tous les types de données numériques (ainsi que le type de données `boolean`) sont des *types valeur*==. **La valeur `null` ne peut jamais être attribuée aux types valeur, car elle sert à établir une référence d'objet vide**.

```cs
// Erreurs du compilateur!
// Les types valeur ne peuvent pas ètre assigné à null!
bool myBool = null;
int myInt = null;
```

C# prend en charge le concept de *types de données annulables*. En termes simples, **un type nullable peut représenter toutes les valeurs de son type sous-jacent, plus la valeur `null`**. Ainsi, ==si vous déclarez un `bool` nullable, une valeur de l'ensemble {`true`, `false`, `null`} peut lui être attribuée==. **Cela peut s'avérer extrêmement utile pour les bases de données relationnelles, car il est fréquent de rencontrer des colonnes indéfinies dans les tables de base de données. Sans le concept de type de données nullable, il est impossible en C# de représenter facilement une donnée numérique sans valeur**.

Pour définir un type de variable nullable, **le point d'interrogation (`?`) est ajouté au type de données sous-jacent**. ==Comme pour une variable non nullable, les variables nullables locales doivent se voir attribuer une valeur initiale avant de pouvoir être utilisées==.

```cs
static void LocalNullableVariable()
{
    // Définition de quelque variable local annulable.
    int? nullableInt = 10;
    double? nullableDouble = 3.14;
    bool? nullableBool = null;
    char? nullableChar = 'a';
    int?[] arrayOfNullableInts = new int?[10];
}
```

## Utilisation des types de valeurs nullable

**C#, la notation avec le suffixe `?` est un raccourci pour créer une instance du type de *structure générique* `System.Nullable<T>` **. Elle est également utilisée pour créer des types de référence Nullable** (abordés dans la section suivante), **bien que le comportement soit légèrement différent**. ==Bien que nous n'abordions les génériques qu'au [[Chapitre 10|Chapitre 10]], il est important de comprendre que le type `System.Nullable<T>` fournit un ensemble de **membres utilisables par tous les types nullable**==.

Par exemple, ==vous pouvez déterminer par programmation si une valeur `null` a bien été assignée à la variable nullable grâce à la propriété `HasValue` ou à l'opérateur `!=`==. **La valeur assignée d'un type nullable peut être obtenue directement ou via la propriété `Value`.** En fait, le suffixe `?` n'étant qu'un raccourci pour utiliser `Nullable<T>`, vous pouvez implémenter votre méthode `LocalNullableVariables()` comme suit :

```cs
static void LocalNullableVariablesUsingNullable()
{
    // Définition de quelque variable local annulable
    // en utilisant Nullable<T>.
    Nullable<int> nullableInt = 10;
    Nullable<double> nullableDouble = 3.14;
    Nullable<bool> nullableBool = null;
    Nullable<char> nullableChar = 'a';
    Nullable<int>[] arrayOfNullableInts = new Nullable<int>[10];
}
```

Comme indiqué précédemment, **les types de données annulables peuvent être particulièrement utiles lors de l'interaction avec des bases de données, car les colonnes d'une table de données peuvent être intentionnellement vides (c.-à-d. non-définies)**. Prenons l'exemple de la classe suivante, qui simule l'accès à une base de données dont la table contient deux colonnes potentiellement annulables. ==Notez que la méthode `GetIntFromDatabase()` n'affecte pas de valeur à la variable membre `int?`, tandis que `GetBoolFromDatabase()` attribue une valeur valide au membre `bool?`==.

```cs
class DatabaseReader
{
    // Champs avec des données annulables.
    public int? numericValue = null;
    public bool? boolValue = true;

    // Notez le type de retour annulable.
    public int? GetIntFromDatabase() 
    { return numericValue; }

    // Notez le type de retour annulable.
    public bool? GetBoolFromDatabas() 
	 { return boolValue; }
}

```

Examinons maintenant le code suivant, qui appelle chaque membre de la classe `DatabaseReader` et découvre les valeurs attribuées à l'aide des membres `HasValue` et `Value`, ainsi qu'à l'aide de l'opérateur d'égalité C# (« pas égal » pour être précis) :

```cs
Console.Title = "Fun With Nullables Value Types";
Console.WriteLine("***** Fun with Nullable Value Types *****\n");

DatabaseReader dr = new();

// Récupère int de "la base de donnée"
int? i = dr.GetIntFromDatabase();
if (i.HasValue)
    Console.WriteLine($"Value of '{nameof(i)}' is: {i.Value}");
else
    Console.WriteLine($"Value of '{nameof(i)}' is undefined.");

// Récupère bool de "la base de donnée"
bool? b = dr.GetBoolFromDatabase();
if (b != null)
    Console.WriteLine($"Value of '{nameof(b)}' is: {b.Value}");
else
    Console.WriteLine($"Value of '{nameof(b)}' is undefined.");

Console.ReadLine();
```

## Utilisation des types de référence annulable (Nouveauté C# 8, MaJ C# 10)

==Une fonctionnalité importante ajoutée à C# 8 est la prise en charge des types de référence annulable. **En fait, ce changement est si important que .NET Framework n'a pas pu être mis à jour pour la prendre en charge, ce qui explique en partie la prise en charge de C# 8 uniquement dans .NET Core 3.0 et versions ultérieures**==.

==Lorsque vous créez un projet dans .NET Core 3.0/3.1 ou .NET 5, les types de référence fonctionnent de la même manière qu'avec C# 7==. Ceci afin d'éviter de casser des milliards de lignes de code existant dans l'écosystème pré-C# 8. ==Les développeurs doivent activer les types de référence annulable dans leurs applications (*.csproj*)==.

**C# 10 et .NET 6 (et versions ultérieures) modifient le comportement par défaut et activent les types de référence annulable dans tous les modèles de projet.**

Les types de référence annulable suivent en grande partie les mêmes règles que les types de valeur annulable. Les types de référence non-annulables doivent se voir attribuer une valeur non `null` à l'initialisation et ne peuvent pas être modifiés plus tard en une valeur `null`. **Les types de référence annulables peuvent être `null`, mais doivent néanmoins se voir *attribuer une valeur* avant leur première utilisation (soit une instance réelle, soit la valeur `null`)**.

**Les types de référence annulables utilisent le même symbole (`?`) pour indiquer qu'ils sont annulables.** ==Cependant, il ne s'agit pas d'un raccourci pour utiliser `System.Nullable<T>`, car seuls les types valeur peuvent être utilisés à la place de `T`==. Pour rappel, les génériques et les contraintes sont traités au [[Chapitre 10|Chapitre 10]].

### Activation des types référence annulables (MaJ C# 10.0)

La prise en charge des types de référence annulables est contrôlée par la définition d'un contexte nullable. Ce contexte ==peut être aussi vaste qu'un projet entier== (en mettant à jour le fichier *.csproj*) ==ou aussi restreint que quelques lignes== (en utilisant des directives du compilateur). **Deux contextes peuvent également être définis** :

- *Contexte d'annotation nullable* : active/désactive l'annotation nullable (`?`) pour les types de référence annulables.
- **Contexte d'avertissement nullable** : cela active/désactive les avertissements du compilateur pour les types de référence annulables.

Pour les visualiser en action, créez une application console nommée `FunWithNullableReferenceTypes`. Ouvrez le fichier projet(*.csproj*). **Comme indiqué précédemment, en C# 10, chaque nouveau projet active par défaut les types de référence Nullable**. Notez le nœud `<Nullable>` dans la liste du fichier projet (==toutes les options disponibles sont présentées dans le [[#Tableau 4-5 Valeurs pour `Nullable` dans le fichier *.csproj*|Tableau 4-5]])== :

```Xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net10.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

</Project>
```

##### Tableau 4-5: Valeurs pour `Nullable` dans le fichier *.csproj*

| Valeurs       | Annotations | Avertissements |
| ------------- | ----------- | -------------- |
| `enable`      | ✅           | ✅              |
| `warnings`    | ❌           | ✅              |
| `annotations` | ✅           | ❌              |
| `disable`     | ❌           | ❌              |

###### Légende des symboles

- ✅ -> activé
- ❌ -> désactivé
 
Comme on peut s'y attendre, l'élément `<Nullable>` du fichier projet affecte l'ensemble du projet. ==Pour contrôler des parties plus petites du projet, utilisez les directives du compilateur présentées dans le [[#Tableau 4-6 Valeurs pour la directive de compilateur ` nullable`|Tableau 4-6]]==.

##### Tableau 4-6: Valeurs pour la directive de compilateur `#nullable`

| Valeur                | Annotations | Avertissements |
| --------------------- | ----------- | -------------- |
| `enable`              | ✅           | ✅              |
| `disable`             | ❌           | ❌              |
| `resotre`             | *.csproj*   | *.csproj*      |
| `disable warnings`    | ❓           | ❌              |
| `enable warnings`     | ❓           | ✅              |
| `restore warning`     | ❓           | *.csproj*      |
| `disable annotations` | ❌           | ❓              |
| `enable annotations`  | ✅           | ❓              |
| `restore annotations` | *.csproj*   | ❓              |

###### Légende des symboles

- ✅ -> activé
- ❌ -> désactivé
- ❓ -> pas affecté
- *.csproj* -> Remise à zéro aux paramètres du projet.

>[!Important]  Le fonctionnements par défaut des projets de ce livre.
>Comme mentionné précédemment, avec l'introduction de C# 10/.NET 6, les types de référence annulables (*NRT*s en anglais) sont activés par défaut. ==Dans la suite de ce livre, les NRTs sont *désactivés* dans tous les exemples de code, sauf indication contraire dans l'exemple==. **Cela ne signifie pas que vous ne devez pas utiliser les NRTs** ; c'est une décision que vous devez prendre en fonction des exigences de votre projet. ==Je les ai désactivés afin que les exemples de ce livre soient centrés sur l'objectif pédagogique spécifique==.

### Types de référence Nullable en action

**En grande partie à cause de l'importance de ce changement, les types annulable ne génèrent des erreurs que s'ils sont mal utilisés.** Ajoutez la classe suivante au fichier *Program.cs* :

```cs
public class TestClass
{
	public string Name { get; set; }
	public int Age { get; set; }
}
```

Comme vous pouvez le constater, il s'agit d'une classe normale. L'annulabilité intervient lorsque vous utilisez cette classe dans votre code. Prenons les déclarations suivantes :

```cs
string? nullableString = null;
TestClass? myNullableClass = null;
```

**Le paramétrage du fichier projet transforme l'ensemble du projet en contexte nullable. Ce contexte nullable permet aux déclarations des types `string` et `TestClass` d'utiliser l'annotation nullable (`?`).** La ligne de code suivante génère un avertissement (`CS8600`) en raison de l'affectation d'une valeur `null` à un type non nullable dans un contexte nullable :

```cs
// warning CS8600 Converting null literal or possible null value to a non-nullable type.
TestClass myNonNullableClass = myNullableClass;
```

**Pour un contrôle plus fin de l'emplacement des contextes annulables dans votre projet, vous pouvez utiliser les directives du compilateur** (comme indiqué [[Chapitre 4#Comprendre les fonctions locales (Nouveauté C 7.0, MaJ C 9.0)|précédemment]]) **pour activer ou désactiver le contexte.** Le code suivant désactive le contexte nullable (défini au niveau du projet), puis le réactive en restaurant les paramètres du projet :

```cs
#nullable disable
TestClass anotherNullableclass = null;

// Avertissement CS8632 : l’annotation pour les types référence nullables ne doit
// être utilisée que dans du code sans annotations '#nullable'.
TestClass? badDefinition = null;

// Avertissement CS8632 : l’annotation pour les types référence nullables ne doit
// être utilisée que dans du code sans annotations '#nullable'.
string? anotherNullableString = null;
#nullable restore
```

Ajoutez la méthode d'exemple précédente `EnterLogData()` dans les instructions de niveau supérieur de notre projet actuel :

```cs
static void EnterLogData(string message, string owner = "programmer")
{
	ArgumentNullException.ThrowIfNull(message);
	Console.WriteLine($"Error: {message}");
	Console.WriteLine($"Owner of the error: {owner}");
}
```

Comme les types de référence annulables sont activés dans ce projet, les paramètres `message` et `owner` ne sont pas annulables. Le paramètre `owner` a une valeur par défaut ; il ne sera donc jamais `null`. En revanche, le paramètre `message` n'a pas de valeur par défaut; son appel suivant génère un avertissement du compilateur :

```cs
// Avertissement CS8625 Impossible de convertir une valeur littérale nulle en un type de référence non nullable.
EnterLogData(null);
```

Il est probable que vous ne transmettiez pas explicitement une valeur nulle, mais plus probablement une variable qui se trouve être nulle :

```cs
string? msg = null;
// Avertissement CS8604 : Argument de référence null possible pour le paramètre 'message'
// dans 'void EnterLogData(string message, string owner = "programmer")'
EnterLogData(msg);
```

==Cela ne résout pas le problème des valeurs nulles transmises à une méthode, mais permet une certaine vérification de votre code à la compilation==.

**Enfin, les types de référence annulables ne possèdent pas les propriétés `HasValue` et `Value`, car celles-ci sont fournies par `System.Nullable<T>`**.

### Considérations relatives à la migration

Lorsque vous migrez votre code de C# 7 vers C# 8, C# 9 ou C# 10 et que vous souhaitez utiliser des types de référence annulables, vous pouvez utiliser une combinaison des paramètres du projet et des directives du compilateur pour gérer votre code.

==Il est courant de commencer par activer les avertissements et désactiver les annotations annulables pour l'ensemble du projet==. Ensuite, au fur et à mesure que vous nettoyez certaines zones de code, ==utilisez les directives du compilateur pour activer progressivement les annotations==. **N'oubliez pas qu'avec C# 10, les types de référence annulables sont activés par défaut dans les modèles de projet**.

### Transformer les avertissements nullables en erreurs

==Lorsque vous êtes prêt à valider des types de référence annulables, vous pouvez *configurer les avertissements nullables en erreurs*. La méthode la plus simple pour un projet est d'ajouter ce qui suit au fichier de projet== :

```xml
<PropertyGroup>
	<WarningsAsErrors>CS8604,CS8625</WarningsAsErrors>
</PropertyGroup>
```

En fait, **si vous souhaitez traiter tous les avertissements liés aux types de référence annulables comme des erreurs, vous pouvez utiliser la syntaxe suivante** :

```xml
<PropertyGroup>
	<WarningsAsErrors>Nullable</WarningsAsErrors>
</PropertyGroup>
```

>[!note] Note:
>==Vous pouvez transformer n'importe quel avertissement en erreur, et pas seulement ceux concernant les types de référence annulables==. Vous pouvez également transformer tous les avertissements en erreurs en utilisant: 
> ```xml
> <TreatWarningsAsErrors>true</TreatWarningsAsErrors>
> ```
> au lieu du nœud `WarningsAsErrors` dans votre fichier projet.

## Opérer sur les types annulables

C# propose plusieurs opérateurs pour travailler avec les types annulables. Les sections suivantes portent sur l'opérateur de *coalescence null*, l'opérateur d'*affectation null* et l'opérateur *conditionnel null*. ==Pour ces exemples, Pour ces examples, retournez au projet `FunWithNullableValueTypes`==.

### L'opérateur de coalescence null

L'opérateur de *coalescence null* est un autre aspect à prendre en compte : **toute variable susceptible de contenir une valeur null peut utiliser l'opérateur C# `??`, appelé formellement opérateur de coalescence null**. ==Cet opérateur permet d'affecter une valeur à un type nullable si la valeur récupérée est effectivement `null`==. Dans cet exemple, supposons que vous souhaitiez affecter un entier nullable local à `100` si la valeur renvoyée par `GetIntFromDatabase()` est `null` (bien sûr, cette méthode est programmée pour toujours renvoyer `null`, mais vous avez sans doute saisi l'idée générale). Revenez au projet `FunWithNullableValueTypes` (et définissez-le comme projet de démarrage) et saisissez le code suivant :

```cs
Console.Title = "Fun With Nullables Value Types";
Console.WriteLine("***** Fun with Nullable Value Types *****\n");

DatabaseReader dr = new DatabaseReader();

// Si la valeur retournée par GetIntFromDatabase() est nulle,
// assigne 100 à la variable locale.
int myData = dr.GetIntFromDatabase() ?? 100;
Console.WriteLine($"Value of {nameof(myData)}: {myData}");
Console.ReadLine();
```

**L'avantage de l'opérateur `??` est qu'il offre une version plus compacte d'une condition if/else traditionnelle.** Cependant, si vous le souhaitez, vous auriez pu créer le code fonctionnellement équivalent suivant pour garantir que si une valeur est nulle, elle sera bien définie sur `100` :

```cs
// Version plus longue sans utiliser l'opérateur ??.
int? moreData = dr.GetIntFromDatabase();
if (!moreData.HasValue) moreData = 100;
Console.WriteLine($"Value of {nameof(moreData)}: {moreData}");
Console.ReadLine();
```

### L'opérateur d'affectation à coalescence nulle (Nouveauté C# 8.0)

S'appuyant sur l'opérateur à coalescence nulle, C# 8 a introduit l'opérateur d'*affectation à coalescence nulle (`??=`).* **Cet opérateur affecte le membre de gauche au membre de droite uniquement si le membre de gauche est `null`**. Par exemple, saisissez le code suivant :

```cs
// Opérateur d'assignation à coalescence null.
int? nullableInt = null;
nullableInt ??= 12;
nullableInt ??= 14;
Console.WriteLine($"Value of {nameof(nullableInt)}: {nullableInt}");
```

La variable `nullableInt` est initialisée à `null`. La ligne suivante lui attribue la valeur `12`, car le membre de gauche est effectivement `null`. La ligne suivante n'attribue pas la valeur `14`, car elle n'est pas `null`.

### L'opérateur conditionnel à coalescence nul

**Lors de l'écriture de logiciels, il est courant de vérifier les paramètres entrants, qui sont des valeurs renvoyées par les membres de type** (méthodes, propriétés, indexeurs), **par rapport à la valeur `null`**. Par exemple, ==supposons que vous ayez une méthode qui prend un `string[]` comme unique paramètre. Par sécurité, vous pouvez tester la valeur `null` avant de continuer. Ainsi, vous éviterez une erreur d'exécution si le tableau est vide==. Voici une méthode classique pour effectuer une telle vérification :

```cs
static void TesterMethod(string[] args)
{
   // We should chekc for null before accessing the array data!
	if (args != null)
		Console.WriteLine($"You sent me {args.Length} arguments.");
}
```

Ici, vous utilisez une *portée conditionnelle* pour garantir que la propriété `Length` du tableau de `string` ne sera pas accessible si le tableau est `null`. Si l'appelant n'a pas créé de tableau de données et a appelé votre méthode de cette manière, vous êtes toujours en sécurité et ne déclencherez pas d'erreur d'exécution :

```cs
TesterMethod(null);
```

C# inclut l'opérateur conditionnel `null` (**un point d'interrogation placé après le type d'une variable, mais avant un opérateur d'accès**) afin de simplifier la vérification des erreurs. ==Plutôt que de créer explicitement une instruction conditionnelle pour vérifier la valeur `null`, vous pouvez désormais écrire ce qui suit== :

```cs
static void TesterMethod2(string[] args)
{
	// We should check for null before accessing the array data!
	Console.WriteLine($"You sent me {args?.Length} arguments");
}
```

> Cette manière de procéder est similaire à JavaScript.

Dans ce cas, vous n'utilisez pas d'instruction conditionnelle. **Vous ajoutez plutôt l'opérateur `?` directement après la variable de tableau de `string`**. Si la variable est `null`, son appel à la propriété `Length` ne déclenchera pas d'erreur à l'exécution. ==Si vous souhaitez afficher une valeur réelle, vous pouvez utiliser l'opérateur de coalescence nulle pour attribuer une valeur par défaut==, comme suit :

```cs
Console.WriteLine($"You sent me {args?.Length ?? 0} arguments.");
```

**Il existe d'autres domaines de programmation où l'opérateur conditionnel null de C# 6.0 sera très utile, notamment pour travailler avec des délégués et des événements**. Ces sujets sont abordés plus loin dans le livre (voir le [[Chapitre 12|Chapitre 12]]) et vous y découvrirez de nombreux autres exemples.

# Comprendre les tuples (Nouveauté / MaJ C# 7.0)

Pour conclure ce chapitre, examinons le rôle des tuples à l'aide d'un projet d'application console nommé *FunWithTuples*. **Comme mentionné précédemment, une façon d'utiliser les paramètres `out` est de récupérer plusieurs valeurs à partir d'un appel de méthode. Une autre méthode consiste à utiliser une construction légère appelée *tuple***.

==Les tuples sont des structures de données légères contenant plusieurs champs. Ils ont été ajoutés au langage C# 6, mais de manière extrêmement limitée==. L'implémentation de C# 6 présentait également un problème potentiellement important : chaque champ est implémenté comme un type référence, ce qui pouvait engendrer des problèmes de mémoire et/ou de performances (en raison du boxing/unboxing).

>[!tip]
>Le *boxing* se produit lorsqu'un type de valeur est stocké comme variable de référence (stocké dans le *heap*), et l'*unboxing* se produit lorsque le type de valeur est renvoyé à une variable de type valeur (stockée sur le *stack*). Le boxing et l'unboxing, ainsi que leurs implications en termes de performances, sont traités en détail au [[Chapitre 10|Chapitre 10]].

==En C# 7, les tuples utilisent le nouveau type de données `ValueTuple` au lieu des types référence, ce qui permet d'économiser une quantité importante de mémoire==. Le type de données `ValueTuple` crée différentes structures en fonction du nombre de propriétés d'un tuple. **C# 7 offre également la possibilité d'attribuer un nom spécifique à chaque propriété d'un tuple (comme pour les variables), ce qui améliore considérablement son utilisation.**

**Voici deux points importants à prendre en compte pour les tuples :**
- Les champs ne sont pas validés.
- Vous ne pouvez pas définir vos propres méthodes.

Ils sont conçus comme un simple mécanisme de transport de données léger.

>[!warning] Important
>La notation :
>```cs
>(string a, int b) variable = ("Foo", 1)
>```
>et la notation :
>```cs
>Tuple<string a, int b> variable = new("Foo", 1)
>```
>Ne sont pas identique sous le capot!
>
>La première notation renvois comme type `System.ValueTuple` (C# 7+) tandis que la deuxième renvoie `System.Tuple` (avant C# 7).

## Prise en main des tuples

Assez de théorie. Passons à l'écriture de code ! Pour créer un tuple, **placez simplement les valeurs à lui assigner entre parenthèses**, comme suit :

```cs
("a", 5, "c")
```

***Notez qu'elles ne doivent pas nécessairement être du même type de données***. ==La construction entre parenthèses permet également d'assigner le tuple à une variable (vous pouvez également utiliser le mot-clé `var`; le compilateur assignera les types de données pour vous)==. Pour assigner l'exemple précédent à une variable, les deux lignes suivantes ont le même résultat. La variable values ​​sera un tuple contenant deux propriétés de type `string` et une propriété de type `int`.

```cs
(string, int, string) values = ("a", 5, "c");
var values = ("a", 5, "c");
```

**Par défaut, le compilateur attribue à chaque propriété le nom `ItemX`, où `X` représente la position de base 1 dans le tuple**. Dans l'exemple précédent, les noms des propriétés sont `Item1`, `Item2` et `Item3`. Leur accès se fait comme suit :

```cs
Console.WriteLine($"First item: {values.Item1}");
Console.WriteLine($"Second item: {values.Item2}");
Console.WriteLine($"Third item: {values.Item3}");
```

**Des noms spécifiques peuvent également être ajoutés à chaque propriété du tuple, à droite ou à gauche de l'instruction**. Bien que l'attribution de noms aux deux extrémités de l'instruction ne constitue pas une erreur de compilation, le côté droit sera ignoré et seuls les noms de gauche seront utilisés. Les deux lignes de code suivantes illustrent la configuration des noms à gauche et à droite pour obtenir le même résultat :

```cs
(string FirstLetter, int TheNumber, string SecondLetter) valuesWithNames = ("a", 5, "c");

// Cette syntax est la plus courrament utilisée.
var valuesWithNames2 = (FirstLetter: "a", TheNumber: 5, SecondLetter: "c");
```

Les propriétés du tuple sont désormais accessibles via les noms de champs et la notation `ItemX`, comme illustré dans le code suivant :

```cs
Console.WriteLine($"First item: {valuesWithNames.FirstLetter}");
Console.WriteLine($"Second item: {valuesWithNames.TheNumber}");
Console.WriteLine($"Third item: {valuesWithNames.SecondLetter}");

// Utiliser la notation item fonctionne aussi!
Console.WriteLine($"First item: {valuesWithNames2.Item1}");
Console.WriteLine($"Second item: {valuesWithNames2.Item2}");
Console.WriteLine($"Third item: {valuesWithNames2.Item3}");
```

>[!tip] Important
> **Notez que pour définir les noms à droite, vous devez utiliser le mot-clé `var` pour déclarer la variable**.
>
> Définir spécifiquement les types de données (même sans noms personnalisés) force le compilateur à utiliser le côté gauche, à assigner les propriétés avec la notation `ItemX` et à ignorer les noms personnalisés définis à droite. **Les deux exemples suivants ignorent les noms `Custom1` et `Custom2`** :
>
>```cs
>(int, int) example = (Custom1:5, Custom2:7);
>(int Field1, int Field2) example2 = (Custom1:5, Custom2:7);
>```

Il est également important de souligner que ==les noms de champs personnalisés n'existent qu'à la compilation et ne sont pas disponibles lors de l'inspection du tuple à l'exécution par *réflexion*== (la réflexion est abordée au [[Chapitre 17#Comprendre la réflexion|Chapitre 17]]).

**Les tuples peuvent également être imbriqués dans d'autres tuples. Puisque chaque propriété d'un tuple est un type de données et qu'un tuple est un type de données**, le code suivant est parfaitement légitime :

```cs
Console.WriteLine("=> Nested Tuples");
var nt = (5, 4, ("a", "b"));
```

## Utilisation des noms de variables inférés (MaJ C# 7.1)

Une mise à jour des tuples dans C# 7.1 permet désormais à C# d'inférer les noms de variables des tuples, comme illustré ici :

```cs
Console.WriteLine("=> Inferred Tuple Names");
var foo = new {Prop1 = "first", Prop2 = "second"};
var bar = (foo.Prop1, foo.Prop2);
Console.WriteLine($"{bar.Prop1};{bar.Prop2}");
```

## Comprendre l'égalité/inégalité des tuples (Nouveauté C# 7.3)

Une fonctionnalité ajoutée à C# 7.3 est l'égalité (`==`) et l'inégalité (`!=`) des tuples. **Lors des tests d'inégalité, les opérateurs de comparaison effectuent des conversions implicites sur les types de données au sein des tuples, notamment en comparant les tuples et/ou propriétés annulables et non annulables**. Cela signifie que les tests suivants fonctionnent parfaitement, malgré la différence entre `int` et `long` :

```cs
// conversions améliorée.
var left = (a: 5, b: 10);
(int? a, int? b) nullableMembers = (5, 10);
Console.WriteLine(left == nullableMembers); // Aussi true

// Les type de donnée converties à droite sont (long, long)
(long a, long b) longTuple = (5, 10);
Console.WriteLine(left == longTuple); // Aussi true

// Comparaison effectuée sur des tuples (long, long)
(long a, int b) longFirst = (5, 10);
(int a, long b) longSecond = (5, 10);
Console.WriteLine(longFirst == longSecond); // Aussi true
```

**Les tuples contenant des tuples peuvent également être comparés, mais seulement s'ils ont la même forme**. Il est impossible de comparer un tuple de trois propriétés `int` avec un autre tuple de deux `int` et un `tuple`.

## Comprendre les tuples comme valeurs de retour de méthode

==Plus tôt dans ce chapitre, les paramètres de sortie (`out`) étaient utilisés pour renvoyer plusieurs valeurs lors d'un appel de méthode==. Il existe d'autres moyens de procéder, comme la création d'une classe ou d'une structure dédiée au renvoi de ces valeurs. Cependant, si cette classe ou structure doit servir de transport de données pour une seule méthode, cela représente un travail et un code supplémentaires qui n'ont pas besoin d'être développés. **Les tuples sont parfaitement adaptés à cette tâche, sont légers et faciles à déclarer et à utiliser.**

Voici un exemple de la section sur les paramètre `out`. Il renvoie trois valeurs, mais nécessite trois paramètres transmis comme mécanismes de transport pour le code appelant.

```cs
static void FillTheseValue(out int a, out string b, out bool c)
{
	a = 9;
	b = "Enjoy you string.";
	c = true;
}
```

**En utilisant un tuple, vous pouvez supprimer les paramètres et récupérer les trois valeurs**.

```cs
static (int a, string b, bool c) FillTheseValues()
{
	return (9, "Enjoy your string", true);
}
```

Appeler cette méthode est aussi simple que d’appeler n’importe quelle autre méthode.

```cs
Console.WriteLine("=> Tuples as Return Value");
// Déconstruction du tuple retounré par la méthode.
var (a, b, c) = FillTheseValues();
Console.WriteLine($"Int is: {a}");
Console.WriteLine($"String is: {b}");
Console.WriteLine($"Bool is: {c}");
```

> Petite mise en bouche, la *déconstruction de tuples* sera abordé un peu plus loin dans ce chapitre.

Un meilleur exemple serait de décomposer un nom complet en ses différentes parties (prénom, deuxième prénom, nom). Le code suivant prend un nom complet et renvoie un tuple contenant ses différentes parties :

```cs
static (string first, string middle, string last) SplitNames(string fullName)
{
    var splited = fullName.Split(" ");
    if (splited.Length > 3)
        throw new Exception("The string isn't compose of 3 elements.");
    return (splited[0], splited[1], splited[2]);
}
```

##  Ignorer des valeurs (discards) de tuples

Suite à l'exemple `SplitNames()`, supposons que vous sachiez que vous n'avez besoin que du prénom et du nom et que le deuxième prénom ne vous importe pas. **En fournissant les noms de variables pour les valeurs à renvoyer et en remplissant les valeurs inutiles avec un trait de soulignement (`_`), vous pouvez affiner la valeur renvoyée comme suit** :

```cs
var (first, _, last) = SplitNames("Philip F Japikse");
Console.WriteLine($"{first}:{last}");
```

**La valeur du deuxième prénom du tuple est jeté**.

>[!Note]- Que devient la valeur de `_`  Dans le détail? (Réponse de Gemini)
>
> **Il n'est plus possible d'accéder à la valeur** !
>
>>[!tldr] Au niveau de votre code (Logique) : OUI, c'est jeté.
>>
>> Une fois que la ligne de déconstruction est passée, il n'existe **aucun moyen** de récupérer cette valeur. Elle n'a pas d'adresse, pas de nom, pas de référence. Pour vous, elle a disparu.
>
>> [!tldr] Au niveau du compilateur (Optimisation) : OUI, c'est jeté.
>>
>>Le compilateur C# est très malin : s'il voit un `_`, il ne génère même pas l'instruction pour stocker cette valeur dans une variable locale. Il "saute" l'étape de rangement.
>
>> [!tldr] Au niveau de la RAM (Mécanique) : PAS TOUT DE SUITE.
>>
>>La valeur reste physiquement dans la "boîte" (le tuple) que la méthode `SplitNames` a renvoyée, jusqu'à ce que cette boîte soit nettoyée :
>>
>> - Si c'est un **`ValueTuple`** (*struct*) : Elle sera effacée dès que vous sortez de la méthode actuelle (nettoyage de la pile "*classique*").
>> - Si c'est un **`Tuple`** (*object*) : Elle sera effacée quand le **Garbage Collector** passera ramasser l'objet entier.

## Comprendre les expressions `switch`  de correspondance de motifs de tuples (Nouveauté C# 8.0)

Maintenant que vous avez une compréhension approfondie des tuples, il est temps de revoir l'expression switch avec les tuples du [[Chapitre 3#Utilisation des expressions `switch` (Nouveauté C 8.0)|Chapitre 3]]. Voici à nouveau l'exemple :

```cs
// Utilisation des tuples dans une expression switch
static string RockPaperScissors(string first, string second)
{
	return (first, second) switch
	{
		("rock", "paper") => "Paper wins.",
		("rock", "scissors") => "Rock wins.",
		("paper", "rock") => "Paper wins.",
		("paper", "scissors") => "Scissors wins.",
		("scissors", "rock") => "Rock wins.",
		("scissors", "paper") => "Scissors wins.",
		(_, _) => "Tie.",
	};
}
```

Dans cet exemple, les deux paramètres sont convertis en un tuple lorsqu'ils sont passés à l'expression switch. Les valeurs pertinentes sont représentées dans l'expression `switch`, et les autres cas sont traités par le tuple final, composé de deux valeurs ignorée.

**La signature de la méthode `RockPaperScissors()` pourrait également être écrite pour accepter un tuple**, comme ceci :

```cs
static string RockPaperScissors(
	(string first, string second) val
)
{
	return val switch
	{
		// Omis pour brièveté.
	};
}
```

## Déconstruction des tuples (MaJ C# 10.0)

La *déconstruction* est le terme utilisé pour séparer les propriétés d'un tuple afin de les utiliser individuellement. **Les exemple `SplitNames()` et `FillTheseValues()` font exactement cela. Les premières et dernières variables ont été accédées indépendamment de toute construction de tuple. Les variables peuvent être initialisées lors de la déconstruction du tuple, ou pré-initialisées.** Les exemples suivants illustrent les deux modèles :

```cs
(int X, int Y) myTuple = (4, 5);
int x = 0;
int y = 0;
(x, y) = myTuple;
Console.WriteLine($"X is: {x}");
Console.WriteLine($"Y is: {y}");
(int x1, int y1) = myTuple;
Console.WriteLine($"x1 is: {x1}");
Console.WriteLine($"y1 is: {y1}");
```

**Nouveauté dans C# 10, l'affectation et la déclaration peuvent être mélangées**, comme le montre l'exemple suivant :

```cs
int x2 = 0;
(x2, int y2) = myTuple;
Console.WriteLine($"x2 is: {x2}");
Console.WriteLine($"y2 is: {y2}");
```

**Ce modèle peut également être utile pour déconstruire des types personnalisés**. Prenons une version abrégée de la structure `Point` utilisée précédemment dans ce chapitre. Une nouvelle méthode nommée `Deconstruct()` a été ajoutée pour renvoyer les propriétés individuelles de l'instance `Point` sous forme de tuple avec les propriétés `XPos` et `YPos`.

```cs
struct Point
{
	// Champs del a structure.
	public int X;
	public int Y;
	
	// Un constructeur personnalisée.
	public Point(int xPos, int yPos)
	{
		X = xPos;
		Y = yPos;
	}
	
	public (int xPos, int yPos) Deconstruct() => (X, Y);
}
```

Notez la nouvelle méthode `Deconstruct()`. **Cette méthode peut porter n'importe quel nom, mais par convention, elle est généralement appelée `Deconstruct()`**. ==Un seul appel de méthode permet ainsi d'obtenir les valeurs individuelles de la structure en renvoyant un tuple==.

```cs
Point p = new Point(7, 5);
var pointValues = p.Deconstruct();
Console.WriteLine($"X is: {pointValues.xPos}");
Console.WriteLine($"Y is: {pointValues.yPos}");
```

### Déconstruction de tuples avec la correspondance de motifs positionnels (Nouveauté C# 8.0)

Lorsque les tuples disposent d'une méthode `Deconstruct()` accessible, la déconstruction peut se faire implicitement sans avoir à appeler la méthode `Deconstruct()`. Le code suivant illustre cette déconstruction implicite :

```cs
Point p2 = new Point(8,3);
int xp2 = 0;
int yp2 = 0;
(xp2,yp2) = p2;
Console.WriteLine($"XP2 is: {xp2}");
Console.WriteLine($"YP2 is: {yp2}");
```

>[!warning] Grosse erreur dans le livre pour cette exemple.
>
> L'extrait de code précédent n'est possible **seulement** si le compilateur détecte une méthode `Deconstruct()` correcte. Cependant, la signature de la méthode `Decuonstruct()` utilisé dans cette structure ne **fonctionne pas!**
>
> Pour que la ligne  `(xp2, yp2) = p2` fonctionne (et ne renvoient pas d'erreur!), il faut utiliser une signature avec *deux paramètre `out`*.
>
> ```cs
> public readonly void Deconstruct(out int x, out int y) => (x, y) = (X, Y);
>```
>
> L'auteur à, pour cette exemple, confondu une **méthode qui retourne un tuple** et la **méthode spéciale de déconstruction** attendue par le compilateur C#.
>
> Cette explication survient juste après pour un autre exemple, mais est **obligatoire** pour que cet exemple, de la manière dont le livre le présente, fonctionne.

**De plus, la déconstruction peut être utilisée dans une expression switch basée sur un tuple**. En utilisant l'exemple `Point`, le code suivant utilise le tuple généré et ses valeurs pour la clause `when` de chaque expression :

```cs
static string GetQuadrant1(Point p)
{
	return p.Deconstruct() switch
	{
		(0, 0) => "Origin",
		var (x, y) when x > 0 && y > 0 => "One",
		var (x, y) when x < 0 && y > 0 => "Two",
		var (x, y) when x < 0 && y < 0 => "Three",
		var (x, y) when x > 0 && y < 0 => "Four",
		var (_, _) => "Border",
	};
}
```

==Si la méthode `Deconstruct()` est définie avec deux paramètres `out`, l'expression switch déconstruira automatiquement le point==. Ajoutez une autre méthode `Deconstruct()` à `Point` comme suit :

```cs
public void Deconstruct(out int xPos, out int yPos)
{
	(xPos, yPos) = (X, Y);
}
```

Vous pouvez maintenant mettre à jour (ou ajouter une nouvelle) méthode `GetQuadrant()` à ceci :

```cs
static string GetQuadrant2(Point p)
{
	return p switch
	{
		(0, 0) => "Origin",
		var (x, y) when x > 0 && y > 0 => "One",
		var (x, y) when x < 0 && y > 0 => "Two",
		var (x, y) when x < 0 && y < 0 => "Three",
		var (x, y) when x > 0 && y < 0 => "Four",
		var (_, _) => "Border",
	};
}
```

Le changement est très subtil. **Au lieu d'appeler `p.Deconstruct()`, seule la variable `Point` est utilisée dans l'expression `switch`**.

# Résumé du chapitre

Ce chapitre a commencé par une **étude des tableaux**. Nous avons ensuite abordé les mots-clés C# permettant de créer des **méthodes personnalisées**. ==Rappelons que, par défaut, les paramètres sont passés par valeur==; cependant, ==vous pouvez passer un paramètre par référence en le marquant avec `ref` ou `out`==. Vous avez également découvert le rôle des **paramètres facultatifs ou nommés**, ainsi que la manière de **définir et d'invoquer des méthodes utilisant des tableaux de paramètres**.

Après avoir abordé la **surcharge de méthodes**, ce chapitre a principalement abordé la définition des **énumérations et des structures en C#** et leur ==représentation dans les bibliothèques de classes de base .NET Core==. Vous avez également étudié plusieurs détails concernant **les types valeur et référence**, notamment ==leur comportement lorsqu'ils sont passés en paramètres à des méthodes== et l'**interaction avec les types de données annulables et les variables potentiellement annulables** (par exemple, les variables de type référence et les variables de type valeur annulables) à l'aide des opérateurs `?`, `??` et `??=`.

La dernière section du chapitre a abordé une fonctionnalité attendue depuis longtemps en C# : **les tuples**. Après avoir compris leur nature et leur fonctionnement, vous les avez utilisés pour ==renvoyer plusieurs valeurs à partir de méthodes, ainsi que pour décomposer des types personnalisés==.

Au [[Chapitre 5|chapitre 5]], vous commencerez à approfondir les détails du développement orienté objet.
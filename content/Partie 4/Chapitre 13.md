---
title: "Chapitre 13: De LINQ aux Objets"
publish: true
---

# <big><big><big><b><font color =green>De LINQ aux Objets</font></b></big></big></big>

==Quel que soit le type d'application que vous créez avec la plateforme .NET, votre programme aura certainement besoin d'accéder à des données lors de son exécution==. Ces données peuvent se trouver à de nombreux emplacements, notamment dans des fichiers XML, des bases de données relationnelles, des collections en mémoire et des tableaux primitifs. **Historiquement, selon l'emplacement des données, les programmeurs devaient utiliser des API différentes et sans lien entre elles**. ***==La technologie LINQ (Language Integrated Query), introduite dans .NET 3.5, offre une méthode concise, symétrique et fortement typée pour accéder à une grande variété de sources de données==***. Dans ce chapitre, vous commencerez votre exploration de LINQ en vous concentrant sur *LINQ to Objects*.

Avant d'aborder LINQ to Objects proprement dit, ==la première partie de ce chapitre passe brièvement en revue les principales constructions de programmation C# qui permettent d'utiliser LINQ==. Au fil de ce chapitre, **vous constaterez que les variables locales implicitement typées, la syntaxe d'initialisation des objets, les expressions lambda, les méthodes d'extension et les types anonymes vous seront très utiles (voire parfois indispensables).**

Après avoir passé en revue cette infrastructure de base, ==le reste du chapitre vous présentera le modèle de programmation LINQ et son rôle dans la plateforme .NET==. **Vous y découvrirez le rôle des opérateurs de requête et des expressions de requête, qui vous permettent de définir des instructions interrogeant une source de données afin d'obtenir le jeu de résultats demandé**. Vous créerez également de nombreux exemples LINQ interagissant avec des données contenues dans des tableaux ainsi que divers types de collections (génériques et non génériques) et **vous comprendrez les assemblies, les espaces de noms et les types qui représentent l'API *LINQ to Objects***.

>[!check] Ce chapitre est ultra important !
>Les informations contenues dans ce chapitre constituent le fondement des sections et chapitres suivants de cet ouvrage, notamment Parallel LINQ ([[Chapitre 15#Requêtes LINQ parallèles (PLINQ)|Chapitre 15]]) et Entity Framework Core ([[Chapitres 21|Chapitres 21 à 23]]).

# Constructions de programmation spécifiques à LINQ

De manière générale, **LINQ peut être considéré comme un langage de requêtes fortement typé, intégré directement à la grammaire de C#**. **==Avec LINQ, vous pouvez créer un nombre illimité d'expressions dont l'apparence et le comportement sont similaires à ceux d'une requête SQL de base de données==**. Cependant, **une requête LINQ peut être appliquée à n'importe quel type de source de données, y compris des sources sans aucun lien avec une base de données relationnelle au sens strict**.

>[!note]
>Bien que les requêtes LINQ puissent ressembler aux requêtes SQL, leur syntaxe *n'est pas* identique. En fait, de nombreuses requêtes LINQ semblent être l'exact opposé d'une requête de base de données similaire ! **Si vous tentez de transposer directement LINQ à SQL, vous risquez fort d'être frustré**. Pour éviter toute confusion, **Considérez les requêtes LINQ comme des structures uniques qui, par coïncidence, ont un air de SQL**. 

Lors de l'introduction de LINQ sur la plateforme .NET dans la version 3.5, les langages C# et VB ont été enrichis de nombreuses nouvelles constructions de programmation pour prendre en charge la technologie LINQ. Plus précisément, **le langage C# utilise les fonctionnalités LINQ suivantes :**

- Variables locales à typage implicite
- Syntaxe d'initialisation des objets et des collections
- Expressions lambda
- Méthodes d'extension
- Types anonymes

***==Ces fonctionnalités ont déjà été abordées en détail dans différents chapitres de cet ouvrage. Toutefois, pour démarrer, reprenons rapidement chaque fonctionnalité une par une, afin de bien comprendre les concepts abordés.==***

>[!note]
>Étant donné que les sections suivantes reprennent des notions déjà abordées dans le livre, je n'ai pas inclus de projet de code C# pour ce contenu.

## Typage implicite des variables locales

==Au [[Chapitre 3#Comprendre les variables locales implicitement typées|Chapitre 3]], vous avez découvert le mot-clé `var` en C#==. **Ce mot-clé vous permet de définir une variable locale sans spécifier explicitement son type de données**. **==La variable est néanmoins fortement typée, car le compilateur déterminera le type de données correct en fonction de l'affectation initiale==**. Rappelez-vous cet exemple de code du [[Chapitre 3#Comprendre les variables locales implicitement typées|Chapitre 3]] :

```cs
static void DeclareImplicitVars()
{
    // Les variables locales implicitement typées
    var myInt = 0;
    var myBool = true;
    var myString = "Time, marches on...";

    // Affiche le type sous-jacent
    Console.WriteLine($"myInt is a: {myInt.GetType().Name}");
    Console.WriteLine($"myBool is a: {myBool.GetType().Name}");
    Console.WriteLine($"myString is a: {myString.GetType().Name}");
}
```

**Cette fonctionnalité du langage est utile, et souvent indispensable, lors de l'utilisation de LINQ**. Comme vous le verrez dans ce chapitre, ==de nombreuses requêtes LINQ renvoient une séquence de types de données, qui ne sont connus qu'à la compilation==. **==Étant donné que le type de données sous-jacent n'est connu qu'à la compilation de l'application, il est évidemment impossible de déclarer une variable explicitement !==**

## Syntaxe d'initialisation des objets et des collections

==Le [[Chapitre 5#Comprendre l'initialisation des objets|Chapitre 5]] a exploré le rôle de la syntaxe d'initialisation des objets, qui permet de créer une classe ou une structure et de définir simultanément un nombre quelconque de ses propriétés publiques==. **Il en résulte une syntaxe compacte** (et pourtant agréable à lire) **permettant de rendre vos objets immédiatement utilisables**. Rappelons également, comme vu au [[Chapitre 10#Comprendre la syntaxe d'initialisation des collections|Chapitre 10]], que **==le langage C# permet d'utiliser une syntaxe similaire pour initialiser des collections d'objets==**. Prenons l'exemple de l'extrait de code suivant, qui utilise la syntaxe d'initialisation des collections pour remplir une `List<T>` d'objets `Rectangle`, chacun d'eux contenant deux objets Point pour représenter une position (`x`, `y`) :

```cs
List<Rectangle> myListOfRects = new List<Rectangle>
{
	new Rectangle {
		TopLeft = new Point { X = 10, Y = 10 },
		BottomRight = new Point { X = 200, Y = 200}
	},
	new Rectangle {
		TopLeft = new Point { X = 2, Y = 2 },
		BottomRight = new Point { X = 100, Y = 100}
	},
	new Rectangle {
		TopLeft = new Point { X = 5, Y = 5 },
		BottomRight = new Point { X = 90, Y = 75}
	}
};
```

**Bien que l'utilisation de la syntaxe d'initialisation des collections/objets ne soit jamais obligatoire, elle permet d'obtenir un code plus compact**. De plus, ==cette syntaxe, combinée au typage implicite des variables locales, permet de déclarer un type anonyme, ce qui est utile lors de la création d'une projection LINQ==. Vous découvrirez les projections LINQ plus loin dans ce chapitre.

## Expressions lambda

==L'opérateur lambda C# (`=>`) a été étudié en détail au [[Chapitre 12#Comprendre les expressions lambda|Chapitre 12]]==. Rappelons que **cet opérateur permet de construire une expression lambda, utilisable chaque fois que vous appelez une méthode nécessitant un délégué fortement typé comme argument**. **==Les expressions lambda simplifient considérablement l'utilisation des délégués en réduisant la quantité de code à écrire manuellement==**. Une expression lambda peut être utilisée de la manière suivante :

```cs
( ArgumentsÀTraiter ) => { DéclarationsÀTraiter }
```

***==Au [[Chapitre 12#Comprendre les expressions lambda|Chapitre 12]], je vous ai présenté trois approches différentes pour interagir avec la méthode `FindAll()` de la classe générique `List<T>`. Après avoir travaillé avec le délégué `Predicate<T>` brut et une méthode anonyme C#, vous êtes finalement arrivés à l'itération (très concise) suivante avec cette expression lambda==*** :

```cs
void LambdaExpressionSyntax()
{
    // Crée une liste d'entiers
    List<int> list = new List<int>();
    list.AddRange(new int[] { 20, 1, 4, 8, 9, 44 });

	// Expression lambda C#.
    List<int> evenNumbers = list.FindAll(i => (i % 2) == 0);

    Console.WriteLine("Here are your even numbers:");
    foreach (int evenNumber in evenNumbers)
    {
        Console.Write($"{evenNumber}\t");
    }
    Console.WriteLine();
}
```

**==Les lambdas seront utiles lors de la manipulation du modèle objet sous-jacent de LINQ==**. Comme vous le découvrirez bientôt, **les opérateurs de requête LINQ en C# ne sont qu'une notation abrégée pour appeler des méthodes classiques sur une classe nommée `System.Linq.Enumerable`**. ==Ces méthodes requièrent généralement des délégués (**en particulier le délégué `Func<>`**) comme paramètres, qui servent à traiter vos données pour produire le jeu de résultats attendu==. Grâce aux lambdas, vous pouvez simplifier votre code et permettre au compilateur de déduire le délégué sous-jacent.

## Méthodes d'extension

==Les méthodes d'extension C# permettent d'ajouter de nouvelles fonctionnalités aux classes existantes sans avoir à créer de sous-classe==. De plus, **elles permettent d'ajouter de nouvelles fonctionnalités aux classes et structures scellées, qu'une pourraient de toute façon jamais être héritées**. Rappelons-nous du [[Chapitre 11#Définition des méthodes d'extension|Chapitre 11]] : **==lors de la création d'une méthode d'extension, le premier paramètre est qualifié par le mot-clé `this` et indique le type étendu==**. Rappelons également que **==les méthodes d'extension doivent toujours être définies dans une classe statique et doivent donc être déclarées à l'aide du mot-clé `static`.==** Voici un exemple :

```cs
namespace MyExtensions;

static class ObjectExtensions
{
	// Définit une méthode d'extension pour System.Object.
	public static void DisplayDefiningAssembly(this object obj)
	{ 
		Console.WriteLine("{0} lives here :\n\t->{1}\n", 
			obj.GetType().Name,
			Assembly.GetAssembly(obj.GetType())
		);
	}
}
```

**Pour utiliser cette extension, une application doit importer l'espace de noms qui la définit (et éventuellement ajouter une référence à l'assembly externe)**. ==Il suffit ensuite d'importer l'espace de noms et de coder.==

```cs
// Puisque tout étend System.Object, toutes les classes et structures
// peuvent utiliser cette extension.
int myInt = 12345678;
myInt.DisplayDefiningAssembly();

System.Data.DataSet d = new System.Data.DataSet();
d.DisplayDefiningAssembly();
```

***==Lorsque vous travaillez avec LINQ, vous aurez rarement, voire jamais, besoin de créer manuellement vos propres méthodes d'extension==***. Cependant, **lors de la création d'expressions de requête LINQ, vous utiliserez de nombreuses méthodes d'extension déjà définies par Microsoft**. En fait, **==chaque opérateur de requête LINQ en C# est une notation abrégée pour effectuer un appel manuel sur une méthode d'extension sous-jacente, généralement définie par la classe utilitaire `System.Linq.Enumerable`.==**

## Types anonymes

La dernière fonctionnalité du langage C# que j'aimerais aborder rapidement est celle des types anonymes, que nous avons explorée au [[Chapitre 11#Comprendre les types anonymes|Chapitre 11]]. *==Cette fonctionnalité permet de modéliser rapidement la « structure » ​​des données en autorisant le compilateur à générer une nouvelle définition de classe lors de la compilation, à partir d'un ensemble de paires nom-valeur==*. **Rappelons que ce type sera composé selon une sémantique basée sur les valeurs, et que chaque méthode virtuelle de `System.Object` sera redéfinie en conséquence**. ==Pour définir un type anonyme, déclarez une variable implicitement typée et spécifiez la structure des données à l'aide de la syntaxe d'initialisation d'objet.==

```cs
// Créer un type anonyme composé d'un autre.
var purchaseItem = new {
	TimeBought = DateTime.Now,
	ItemBought =
		new {Color = "Red", Make = "Saab", CurrentSpeed ​​= 55},
	Price = 34.000
};
```

***==LINQ utilise fréquemment des types anonymes pour projeter dynamiquement de nouvelles formes de données.==*** Par exemple, ==supposons que vous ayez une collection d'objets `Person` et que vous souhaitiez utiliser LINQ pour obtenir des informations sur l'âge et le numéro de sécurité sociale de chaque personne==. **Grâce à une projection LINQ, vous pouvez autoriser le compilateur à générer un nouveau type anonyme contenant ces informations.**

# Comprendre le rôle de LINQ

Ceci conclut notre bref aperçu des fonctionnalités du langage C# qui permettent à LINQ de déployer toute sa puissance. Mais, **pourquoi utiliser LINQ ?** En tant que développeurs, il est indéniable qu'une part importante de notre temps de programmation est consacrée à l'acquisition et à la manipulation de données. Lorsqu'on parle de « données », on pense facilement aux informations contenues dans des bases de données relationnelles. Cependant, les données se trouvent également fréquemment dans des documents XML ou de simples fichiers texte.

Les données peuvent se trouver à bien d'autres endroits que ces deux types d'environnements courants. Par exemple, imaginons que vous ayez un tableau ou un type générique `List<T>` contenant 300 entiers et que vous souhaitiez obtenir un sous-ensemble répondant à un critère donné (par exemple, uniquement les éléments pairs ou impairs du conteneur, uniquement les nombres premiers, uniquement les nombres non répétitifs supérieurs à 50). Ou peut-être utilisez-vous les API de réflexion et avez-vous besoin d'obtenir uniquement les métadonnées de chaque classe dérivée d'une classe parente au sein d'un tableau de types. En effet, les données sont omniprésentes.

Avant .NET 3.5, interagir avec différents types de données nécessitait l'utilisation d'API très diverses. Prenons l'exemple du [[#Tableau 13-1 Manière de manipuler des types de donnée variés|Tableau 13-1]], qui illustre plusieurs API courantes permettant d'accéder à divers types de données (et vous pouvez certainement trouver de nombreux autres exemples).

##### Tableau 13-1: Manière de manipuler des types de donnée variés

>[!info] Une partie des informations présenté seront expliqués un peu plus loin !

| La donnée que l'on veut      | Comment l'obtenir (C# 10)                                                             | Comment l'obtenir (C# 13/14)                                           |
| ---------------------------- | ------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| **Données relationnelles**   | *System.Data.dll*, *System.Data.SqlClient.dll,* etc.                                  | `Microsoft.Data.SqlClient` (Plus moderne et sécurisé)                  |
| **Données de documents XML** | *System.Xml.dll*                                                                      | `System.Xml.Linq` (LINQ to XML est la norme privilégiée)               |
| **Tableaux de métadonnées**  | L'espace de noms `System.Reflection`                                                  | `System.Reflection` + **Source Generators** (analyse à la compilation) |
| **Collections d'objets**     | Les espace de noms`System.Array` et `System.Collections`/`System.Collections.Generic` | `System.Colections.Generic` et `System.Collecitons.Immutable`          |

>[!warning] Précisions importantes 
>
>- **XML :** Bien que *System.Xml.dll* soit toujours présent, l'utilisation de **LINQ to XML** (`XDocument`, `XElement`) est beaucoup plus intuitive et performante pour manipuler des documents que l'ancien `XmlDocument`.
>- **Métadonnées :** En C# 13, on essaie d'éviter la réflexion à l'exécution (trop lente) au profit des **Source Generators**. Cela permet au compilateur d'inspecter votre code et d'en générer du nouveau automatiquement.
>- **Collections :** La grande tendance moderne est l'utilisation des collections **Immutables**. Une fois créées, elles ne changent plus, ce qui évite de nombreux bugs dans les applications asynchrones ou multi-threadées.
>- **Performance :** C# 13 met l'accent sur les types `Span<T>` et `ReadOnlySpan<T>` pour manipuler les tableaux et les chaînes de caractères de manière ultra-rapide sans allocation mémoire inutile.

##### Tableau 13-2:  Nouveaux type de données manipulable (C# 11 +)

| La donnée que l'on veut       | Comment l'obtenir (Moderne)                | Note technologique                                      |
| ----------------------------- | ------------------------------------------ | ------------------------------------------------------- |
| **Données JSON**              | `System.Text.Json`                         | Remplace souvent le XML aujourd'hui.                    |
| **Flux asynchrones**          | `IAsyncEnumerable<T>`                      | Traitement de données "en direct" (Async LINQ).         |
| **Bases NoSQL**               | Drivers spécifiques (ex: `MongoDB.Driver`) | Utilise LINQ pour interroger des docs non-relationnels. |
| **Données haute performance** | `Span<T>` / `ReadOnlySpan<T>`              | Pour les calculs critiques (C# 13/14).                  |

Bien sûr, ces approches de manipulation de données sont tout à fait valables. En effet, vous pouvez (et allez) utiliser directement ADO.NET, les espaces de noms XML, les services de réflexion et les différents types de collections. Cependant, ==le problème fondamental est que chacune de ces API fonctionne de manière isolée, ce qui limite considérablement les possibilités d'intégration==. *==Certes, il est possible (par exemple) d'enregistrer un `DataSet` ADO.NET au format XML et de le manipuler ensuite via les espaces de noms `System.Xml`, mais la manipulation des données reste néanmoins assez asymétrique==*.

**L'API LINQ vise à fournir une méthode cohérente et symétrique permettant aux programmeurs d'obtenir et de manipuler des « données » au sens large du terme**. Grâce à LINQ, ==vous pouvez créer directement dans le langage de programmation C# des constructions appelées *expressions de requête*==. Ces expressions de requête sont basées sur de nombreux opérateurs de requête conçus pour ressembler (sans toutefois être tout à fait identiques) à une expression SQL. 

**La particularité, cependant, est qu'une expression de requête peut être utilisée pour interagir avec de nombreux types de données, même des données sans lien avec une base de données relationnelle**. **==À proprement parler, « LINQ » est le terme utilisé pour décrire cette approche globale d'accès aux données==**. Toutefois, **selon l'endroit où vous appliquez vos requêtes LINQ, vous rencontrerez différents termes, tels que :**

• *LINQ to Objects* : ce terme désigne l'application de requêtes LINQ aux tableaux et collections.
• *LINQ to XML* : ce terme désigne l'utilisation de LINQ pour manipuler et interroger des documents XML.
• *LINQ to Entities* : cet aspect de LINQ vous permet d'utiliser des requêtes LINQ au sein de l'API ADO.NET Entity Framework (EF) Core.
• *Parallel LINQ (ou PLINQ)* : ceci permet le traitement parallèle des données renvoyées par une requête LINQ. 

**Aujourd'hui, LINQ fait partie intégrante des bibliothèques de classes de base .NET, des langages managés et de Visual Studio lui-même.**

>[!tip]- Si on veut communiquer avec une base de donnée PostgreSQL, il faut utilisé *Entity Framework Core* ([[Chapitre 21|Chapitre 21]])
>C'est la méthode la plus courante aujourd'hui. Elle permet d'utiliser **LINQ** pour interroger votre base PostgreSQL sans écrire de SQL manuellement.
>
>- **Le Package NuGet :** `Npgsql.EntityFrameworkCore.PostgreSQL`
>- **L'espace de noms :** `using Microsoft.EntityFrameworkCore;`
>- **Le concept :** Vous manipulez des classes C# (vos modèles) et le "Provider" traduit vos requêtes LINQ en SQL PostgreSQL.
>- **Avantage :** C'est ici que la puissance de LINQ prend tout son sens.

## Les expressions LINQ sont fortement typées

**Il est également important de souligner qu'une expression de requête LINQ** (contrairement à une instruction SQL traditionnelle) *est fortement typée*. **==Par conséquent, le compilateur C# vous obligera à respecter la syntaxe et s'assurera que ces expressions sont syntaxiquement correctes==**. Grâce à *roslyn*, les IDE peuvent utiliser des métadonnées pour des fonctionnalités utiles comme IntelliSense, la saisie semi-automatique, etc.

## Les assemblies LINQ principaux

Les fonctionnalités LINQ sont définies dans l'espace de noms **`System.Linq`**. Dans les projets modernes, cet espace de noms est souvent inclus par défaut grâce aux **Implicit Usings** (directives `global using` générées par les *templates* du SDK), rendant les méthodes d'extension LINQ disponibles partout dans votre code sans import manuel.

# Application de requêtes LINQ aux tableaux primitifs

Pour commencer à explorer *LINQ to Objects*, créons une application qui appliquera des requêtes LINQ à différents tableaux. Créez un projet d'application console nommé *LinqOverArray* et définissez une méthode d'assistance statique nommée `QueryOverStrings()` dans le fichier *Program.cs*. Dans cette méthode, créez un tableau de chaînes de caractères contenant six éléments environ (ici, j'ai listé une liste de jeux vidéo de ma bibliothèque). Assurez-vous d'avoir au moins deux entrées contenant des valeurs numériques et quelques-unes contenant des espaces.

```cs
static void QueryOverStrings()
{
    // Supposons que nous ayons un tableau de strings.
    // Collections syntax (C# 12)
    string[] currentVideoGames =
    [
        "Morrowind",
        "Uncharted 2",
        "Fallout 3",
        "Daxter",
        "System Shock 2",
    ];
}
```

Maintenant, mettez à jour le fichier *Program.cs* pour appeler la fonction `QueryOverStrings()`.

```cs
Console.Title = "Fun with LINQ to Objects";
Console.WriteLine("***** Fun with LINQ to Objects *****\n");

QueryOverStrings();

Console.ReadLine();
```

**Lorsqu'on travaille avec un tableau de données, il est courant d'en extraire un sous-ensemble en fonction d'un critère donné**. ==Par exemple, vous souhaitez peut-être obtenir uniquement les sous-éléments contenant un nombre== (ex. : System Shock 2, Uncharted 2 et Fallout 3), ==un certain nombre de caractères, ou encore ceux ne contenant pas d'espaces== (ex. : Morrowind ou Daxter). **Bien qu'il soit possible d'effectuer ces tâches manuellement avec les classes de type `System.Array`, les expressions de requête LINQ simplifient grandement le processus.**

Supposons que vous souhaitiez extraire du tableau uniquement les éléments contenant un espace et les trier par ordre alphabétique ; vous pouvez alors construire l'expression de requête LINQ suivante :

```cs
static void QueryOverStrings()
{
    // Supposons que nous ayons un tableau de strings.
    // Collections syntax (C# 12)
    string[] currentVideoGames =
    [
        "Morrowind",
        "Uncharted 2",
        "Fallout 3",
        "Daxter",
        "System Shock 2",
    ];

    // Construit une expression de requête pour trouver
    // les éléments du tableau qui contiennent un espace.
    IEnumerable<string> subset =
        from g in currentVideoGames
        where g.Contains(' ')
        orderby g
        select g;

    // Affiche les résultats
    foreach (string s in subset)
    {
        Console.WriteLine($"Item: {s}");
    }
}
```

**Notez que l'expression de requête créée ici utilise les opérateurs de requête LINQ suivants : `from`, `in`, `where`, `orderby` et `select`**. Vous approfondirez la syntaxe des expressions de requête plus loin dans ce chapitre. Cependant, **==vous pouvez déjà interpréter cette instruction comme suit : « Rendez-moi les éléments de `currentVideoGames` contenant un espace, triés par ordre alphabétique. »==**

Ici, chaque élément correspondant aux critères de recherche est nommé `g` (comme dans « game ») ; toutefois, n'importe quel nom de variable C# valide conviendrait.

```cs
IEnumerable<string> subset =
	from game in currentVideoGames
	where game.Contains(' ')
	orderby game
	select game;
```

Notez que **la séquence renvoyée est stockée dans une variable nommée `subset`, de type implémentant la version générique de `IEnumerable<T>`, où `T` est de type `System.String`** (après tout, vous interrogez un tableau de chaînes de caractères). ==Une fois l'ensemble de résultats obtenu, il vous suffit d'afficher chaque élément à l'aide d'une boucle `foreach` standard==. Si vous exécutez votre application, vous obtiendrez le résultat suivant :

```
***** Fun with LINQ to Objects *****

Item: Fallout 3
Item: System Shock 2
Item: Uncharted 2
```

## Utilisation des méthodes d'extension (une fois de plus)

**La syntaxe LINQ utilisée précédemment** (et dans le reste de ce chapitre) **est appelée expressions de requête LINQ**. Il s'agit d'un format similaire à SQL, mais légèrement différent. ***==Une autre syntaxe, utilisant des méthodes d'extension, sera celle employée dans la plupart des exemples de ce livre==***. 

Créez une nouvelle méthode nommée `QueryOverStringsWithExtensionMethods()` et saisissez le code suivant :

```cs
static void QueryOverStringsWithExtensionsMethods()
{
    // Supposons que nous ayons un tableau de strings.
    // Collections syntax (C# 12)
    string[] currentVideoGames =
    [
        "Morrowind",
        "Uncharted 2",
        "Fallout 3",
        "Daxter",
        "System Shock 2",
    ];

    // Construit une expression de requête pour trouver
    // les éléments du tableau qui contiennent un espace.
    IEnumerable<string> subset = currentVideoGames
        .Where(g => g.Contains(' '))
        .OrderBy(g => g)
        .Select(g => g);

    // Affiche les résultats
    foreach (string s in subset)
    {
        Console.WriteLine($"Item: {s}");
    }
}
```

Tout est identique à la méthode précédente, à l'exception de de l'expression de requête. ==Cette méthode utilise la syntaxe d'extension.== **Cette syntaxe utilise des expressions lambda au sein de chaque méthode pour définir l'opération**. Par exemple, le lambda de la méthode `Where()` définit la condition (où une valeur contient un espace). **==Tout comme dans la syntaxe d'expression de requête, la lettre utilisée pour indiquer la valeur évaluée dans la lambda est arbitraire==** ; j'aurais pu utiliser « v » pour les jeux vidéo.

***==Bien que les résultats soient les mêmes==*** (l'exécution de cette méthode produit la même sortie que la méthode précédente utilisant l'expression de requête), **vous constaterez bientôt que le *type* de l'ensemble de résultats est légèrement différent**. **==Dans la plupart des cas (voire la quasi-totalité), cette différence ne pose aucun problème et les formats peuvent être utilisés indifféremment==**.

## Sans LINQ (une fois de plus)

Certes, *==LINQ n'est jamais obligatoire==*. ==Si vous le souhaitez, vous pouvez obtenir le même résultat en vous passant complètement de LINQ et en utilisant des primitives de programmation telles que les instructions `if` et les boucles `for`==. Voici une méthode qui produit le même résultat que la méthode `QueryOverStrings()`, mais de manière beaucoup plus verbeuse :

```cs
static void QueryOverStringsLongHand()
{
    // Collections syntax (C# 12)
    string[] currentVideoGames =
    [
        "Morrowind",
        "Uncharted 2",
        "Fallout 3",
        "Daxter",
        "System Shock 2",
    ];

    string[] gamesWithSpaces = new string[5];

    for (int i = 0; i < currentVideoGames.Length; i++)
    {
        if (currentVideoGames[i].Contains(' '))
        {
            gamesWithSpaces[i] = currentVideoGames[i];
        }
    }

    // Maintenant on les trie.
    Array.Sort(gamesWithSpaces);

    // Affiche le résultat
    foreach (string s in gamesWithSpaces)
    {
        if (s != null)
        {
            Console.WriteLine($"Item: {s}");
        }
    }
    Console.WriteLine();
}
```

*==Bien que je sois certain que vous puissiez trouver des moyens d'améliorer la méthode précédente, il n'en reste pas moins que les requêtes LINQ peuvent être utilisées pour simplifier radicalement le processus d'extraction de nouveaux sous-ensembles de données à partir d'une source==*. Plutôt que de construire des boucles imbriquées, une logique `if`/`else` complexe, des types de données temporaires, etc., **le compilateur C# effectuera le travail fastidieux à votre place, une fois que vous aurez créé une requête LINQ appropriée**.

## Réflexion sur un ensemble de résultats LINQ

Supposons maintenant que le fichier *Program.cs* définisse une fonction d'assistance supplémentaire nommée `ReflectOverQueryResults()` qui affichera divers détails de l'ensemble de résultats LINQ (==notez que le paramètre est un `System.Object` pour tenir compte des différents types d'ensembles de résultats==).

```cs
static void ReflectOverQueryResults(
    object resultSet,
    string queryType = "Query Expressions"
)
{
    Console.WriteLine($"**** Info about your query using {queryType} ****");
    Console.WriteLine($"resultSet is of type: {resultSet.GetType().Name}");
    Console.WriteLine(
        $"resultSet location: {resultSet.GetType().Assembly.GetName().Name}"
    );
}
```

Mettez à jour le cœur de la méthode `QueryOverStrings()` comme suit :

```cs
// Construit une expression de requête pour trouver
// les éléments du tableau qui contiennent un espace.
IEnumerable<string> subset =
	from game in currentVideoGames
	where game.Contains(' ')
	orderby game
	select game;

ReflectOverQueryResults(subset);

// Affiche les résultats
foreach (string s in subset)
{
	Console.WriteLine($"Item: {s}");
}
```

~~**Lorsque vous exécuterez l'application, vous constaterez que la variable de sous-ensemble est en réalité une instance du type générique `OrderedEnumerable<TElement, TKey>`** (représenté par ```OrderedEnumerable`2```), **qui est un type abstrait interne résidant dans l'assembly `System.Linq.dll`.**~~

>[!warning] Ce qui a changé en .NET 10
>
>Dans les versions précédentes (jusqu'à .NET 9), LINQ utilisait souvent des classes comme `OrderedEnumerable`. .NET 10 a subi ce que certains appellent une "greffe de cerveau" pour améliorer radicalement les performances.
>
>- **Nouveau type interne :** Votre ```OrderedIterator`2``` est le nouveau moteur de tri. Il est conçu pour être beaucoup plus rapide et consommer moins de mémoire que l'ancien `OrderedEnumerable`.
>- **Optimisation du "Short-circuiting" :** Si vous chaînez d'autres méthodes après votre tri (comme un `.First()` ou `.Contains()`), ce nouvel itérateur est capable de "regarder à l'intérieur" de la requête pour éviter de trier tout le tableau si ce n'est pas nécessaire.
>- **Localisation :** Comme vous l'avez noté, il réside toujours dans l'assembly logique **`System.Linq`**, ce qui confirme que l'architecture modulaire de .NET moderne (Core/5/6+) est maintenue
>
>>[!success] Même si les noms des classes internes changent :
>>
>>1. **L'interface est la même :** Votre variable `subset` implémente toujours `IOrderedEnumerable<string>`. C'est la seule chose qui compte pour votre code.
>>2. **Le comportement est identique :** Le principe de l'exécution différée (Lazy Evaluation) expliqué par Troelsen reste le pilier central de LINQ.
>>3. **L'encapsulation :** Le fait que vous deviez "fouiller" pour voir `OrderedIterator` prouve que l'abstraction de Microsoft fonctionne : ils peuvent réécrire tout le moteur sans que vous ayez à changer une seule ligne de votre `foreach`.
>
>---
>>[!example]- **L'explication concrète de pourquoi Microsoft fonctionne de cette manière est expliqué dans cette video :** 
>>
>>La vidéo est sortie avant les modifications apporté par .NET 10, mais l'explication tient toujours
>>
>> ![Video YouTube sur LINQ](https://youtu.be/xKr96nIyCFM?t=4628) 
>---
>*En .NET 10, certaines opérations comme `.OrderBy(...).First()` peuvent être jusqu'à **250x plus rapides** qu'en .NET 9 grâce à ces nouveaux itérateurs qui évitent les tris complets inutiles.*

```
***** Fun with LINQ to Objects *****

**** Info about your query using Query Expressions ****
resultSet is of type: OrderedIterator`2
resultSet location: System.Linq
```

Apportez la même modification à la méthode `QueryOverStringsWithExtensionMethods()`, en ajoutant toutefois `"Extension Methods"` comme deuxième paramètre.

```cs
// Construit une expression de requête pour trouver
// les éléments du tableau qui contiennent un espace.
IEnumerable<string> subset = currentVideoGames
	.Where(g => g.Contains(' '))
	.OrderBy(g => g)
	.Select(g => g);

ReflectOverQueryResults(subset, "Extension Methods");

// Affiche les résultats
foreach (string s in subset)
{
	Console.WriteLine($"Item: {s}");
}
```

Lorsque vous exécutez l'application, vous constaterez que la variable `subset` est une instance de type `SelectIPartitionIterator`. **Si vous supprimez `Select(g=>g)` de la requête, vous retrouverez une instance de type `OrderedEnumerable<TElement, TKey>` (voir note précédente)**. Qu'est-ce que cela signifie ? Pour la plupart des développeurs, pas grand-chose (voire rien). **Les deux types dérivent de `IEnumerable<T>`, les deux peuvent être parcourus de la même manière, et les deux peuvent créer une liste ou un tableau à partir de leurs valeurs.**

```
**** Info about your query using Extension Methods ****
resultSet is of type: IteratorSelectIterator`2
resultSet location: System.Linq
```


>[!warning]- Le résultat avec un .NET plus récent est différent par rapport au livre (Généré par Gemini)
>le nom exact `IteratorSelectIterator` est le fruit de la **réécriture complète de LINQ** entamée dans .NET 7 et stabilisée dans les versions suivantes (.NET 8, 9 et 10). L'objectif était de remplacer les vieux types "Enumerable" par des types "Iterator" plus légers.
>
>### Quel est le bénéfice ?
>
>L'intérêt de passer de `OrderedEnumerable` à un `IteratorSelectIterator` (ou tout autre type d'itérateur moderne) tient en trois mots : **Vitesse**, **Mémoire**, **Composition**.
>
>#### 1. Élimination des allocations (Memory)
>
>Les anciens types LINQ créaient beaucoup d'objets intermédiaires. Ces nouveaux "Iterators" sont conçus pour :
>
>- Réduire la pression sur le Garbage Collector (GC).
>- Utiliser moins d'espace sur la mémoire "Tas" (Heap).
>
>#### 2. Composition de pipelines (Chaining)
>
>Au lieu d'avoir un objet pour le `Where`, un autre pour le `OrderBy` et un autre pour le `Select`, .NET tente de "fusionner" ces étapes. `IteratorSelectIterator` est une sorte de **"wrapper" optimisé** qui dit à .NET : _"Je sais qu'on a déjà filtré/trié avant, je vais juste appliquer la transformation finale pendant que je parcours les données"_.
>
>#### 3. Loop Bound Checks
>
>Ces itérateurs sont écrits pour aider le compilateur JIT (Just-In-Time) à supprimer les vérifications de limites de tableaux inutiles, ce qui rend le `foreach` final presque aussi rapide qu'une boucle `for` classique sur un tableau.
>
>### Pourquoi avez-vous ce type et pas l'autre ?
>
>- **Exemple 1 (Query Expression) :** Le compilateur voit `select game` (une simple sélection de l'objet lui-même). Il est assez intelligent pour se dire : *"Je n'ai pas besoin de créer un itérateur de sélection, je reste sur le `OrderedIterator`"*.
>- **Exemple 2 (Extension Method) :** En écrivant `.Select(g => g)`, vous appelez explicitement une méthode. Même si votre fonction est "identitaire" (elle ne change rien), .NET crée ce `IteratorSelectIterator` pour encapsuler l'appel à votre délégué.
>---
>*Dans la vraie vie, si vous n'avez pas besoin de transformer la donnée (ex: transformer un `User` en `string`), **ne mettez pas le** `.Select(g => g)` **final**. Cela évitera la création de cet `IteratorSelectIterator` et votre code sera légèrement plus performant en restant sur le `OrderedIterator` pur.*

## LINQ et les variables locales implicitement typées

Bien que le programme d'exemple actuel permette de déterminer assez facilement que l'ensemble de résultats peut être capturé sous forme d'énumération de l'objet chaîne (par exemple, `IEnumerable<string>`), **il n'est pas évident que subset soit réellement de type `OrderedEnumerable<TElement, TKey>` (`OrderedIterator` pour les versions modernes).**

**==Étant donné que les ensembles de résultats LINQ peuvent être représentés par un grand nombre de types dans divers espaces de noms LINQ, il serait fastidieux de définir le type approprié pour contenir un ensemble de résultats, car dans de nombreux cas, le type sous-jacent peut ne pas être évident, voire inaccessible directement depuis votre code (et comme vous le verrez, dans certains cas, le type est généré à la compilation)==**.

Pour illustrer davantage ce point, considérez la méthode d'assistance supplémentaire suivante définie dans le fichier *Program.cs* :

```cs
static void QueryOverInts()
{
    // Collection expression (C# 12)
    int[] numbers = [10, 20, 30, 40, 1, 2, 3, 8];

    // Affiche seulement les éléments avec une valeur < 10.
    IEnumerable<int> subset = from i in numbers where i < 10 select i;

    ReflectOverQueryResults(subset);
    
    foreach (int i in subset)
    {
        Console.WriteLine($"Item: {i}");
    }
    Console.WriteLine();
}
```

Dans ce cas, la variable de sous-ensemble est d'un type sous-jacent complètement différent. **Cette fois, le type implémentant l'interface `IEnumerable<int>` est une classe de bas niveau nommée `WhereArrayIterator<T>`.

```
Item: 1
Item: 2
Item: 3
Item: 8
**** Info about your query using Query Expressions ****
resultSet is of type: ArrayWhereIterator`1
resultSet location: System.Linq
```

>[!warning]- Différence là aussi avec les versions modernes (Gemini)
>Le livre mentionne `WhereArrayIterator`. C'était une classe "spécialisée" : il y avait un itérateur pour les tableaux (`WhereArrayIterator`), un pour les listes (`WhereListIterator`), etc.
>
>Dans les versions récentes (.NET 8/9/10), Microsoft a refactorisé le code pour utiliser **`ArrayWhereIterator<T>`**.
>
>- **L'objectif :** Uniformiser la nomenclature interne (le nom suit maintenant le pattern `[Source][Opération][Type]`).
>- **La différence :** Ce n'est pas qu'un changement de nom. Le nouvel `ArrayWhereIterator` est beaucoup plus performant car il utilise souvent des **Spans** en interne pour parcourir votre tableau `int[]` sans l'allouer à nouveau.
>
> ### Pourquoi "Array" au début ?
>
>LINQ est devenu extrêmement intelligent. Lorsque vous faites `from i in numbers` :
>
>- Si `numbers` est un **tableau** (`int[]`), LINQ choisit `ArrayWhereIterator`.
>- Si `numbers` était une **List**, vous verriez probablement `ListWhereIterator`.
>
> **Le bénéfice :** Un tableau est un bloc de mémoire continu. `ArrayWhereIterator` le sait et utilise des optimisations de processeur (comme la vectorisation SIMD dans certains cas) qu'un itérateur générique ne pourrait pas utiliser.
>
> **Si vous changez la première ligne en utilisant une `List<int>` au lieu d'un tableau `int[]`, votre reflet change aussi pour devenir `ListWhereIterator`**
>
>---
>**Votre console (.NET 10) :** *Vous montre l'implémentation de pointe. Le nom a changé car le code a été réécrit pour être plus rapide, mais le rôle de la classe reste le même.*

**Étant donné que le type sous-jacent exact d'une requête LINQ n'est pas toujours évident, ces premiers exemples ont représenté les résultats de la requête sous forme de variable `IEnumerable<T>`, où `T` est le type de données dans la séquence renvoyée** ( `string`, `int`, etc.). Cependant, *==cette approche reste assez lourde==*. De plus, ==étant donné que `IEnumerable<T>` étend l'interface non générique `IEnumerable`, il serait également possible de capturer le résultat d'une requête LINQ comme suit :==

```cs
System.Collections.IEnumerable subset =
	from i in numbers
	where i < 10
	select i;
```

***==Heureusement, le typage implicite simplifie considérablement les choses lors de l'utilisation de requêtes LINQ.==***

```cs
static void QueryOverInts()
{
    // Collection expression (C# 12)
    int[] numbers = [10, 20, 30, 40, 1, 2, 3, 8];

    // Utilise le typage implicit ici...
    var subset = from i in numbers where i < 10 select i;

    // ...et ici
    foreach (var i in subset)
    {
        Console.WriteLine($"Item: {i}");
    }
    ReflectOverQueryResults(subset);
}
```

En règle générale, **il est toujours préférable d'utiliser le typage implicite lors de la capture des résultats d'une requête LINQ**. ==N'oubliez pas cependant que== (dans la plupart des cas) ==la valeur de retour réelle est un type implémentant l'interface générique `IEnumerable<T>`==.

**Le type exact sous-jacent** (`OrderedEnumerable<TElement, TKey>`, `WhereArrayIterator<T>`, etc.) **est sans importance et il n'est pas nécessaire de le découvrir**. Comme illustré dans l'exemple de code précédent, vous pouvez simplement utiliser le mot-clé `var` dans une boucle `foreach` pour parcourir les données récupérées.

>[!tip]- Quand est-ce que cela devient important de "savoir" ?
>
>Même si on ne manipule pas ces types directement, comprendre ce qu'ils font est devenu un atout pour le développeur moderne dans deux cas :
>
>- **Le "Performance Profiling" :** Si votre application est lente, regarder ces types via la réflexion ou un profiler (comme *DotTrace*) vous permet de comprendre pourquoi. Par exemple, voir un `IteratorSelectIterator` vous indique que vous avez peut-être un `.Select()` inutile qui ralentit la boucle.
>- **L'optimisation mémoire :** En .NET 10, savoir que `ArrayWhereIterator` utilise des `Spans` ou évite des allocations vous permet de dormir tranquille : vous savez que LINQ est devenu "gratuit" (ou presque) en termes de ressources, ce qui n'était pas le cas il y a 10 ans.

## LINQ et méthodes d'extension

Bien que cet exemple ne vous demande pas d'écrire directement de méthodes d'extension, vous les utilisez en réalité de manière transparente en arrière-plan. **Les expressions de requête LINQ permettent d'itérer sur des conteneurs de données implémentant l'interface générique `IEnumerable<T>`**. *==Cependant, le type de classe `System.Array`==* (utilisé pour représenter les tableaux de chaînes et d'entiers) *==n'implémente pas ce contrat.==*

```cs
// Le type System.Array ne semble pas implémenter
// l'infrastructure correcte pour les expressions de requête !
public abstract class Array : ICloneable, IList,
	IStructuralComparable, IStructuralEquatable
{
	...
}
```

**Bien que `System.Array` n'implémente pas directement l'interface `IEnumerable<T>`, il bénéficie indirectement des fonctionnalités requises de ce type** (ainsi que de nombreux autres membres spécifiques à LINQ) **via la classe statique `System.Linq.Enumerable`**.

**==Cette classe utilitaire définit un grand nombre de méthodes d'extension génériques==** (telles que `Aggregate<T>()`, `First<T>()`, `Max<T>()`, etc.), **==que `System.Array` (et d'autres types) acquièrent en arrière-plan==**. Ainsi, si vous appliquez l'opérateur point à la variable locale `currentVideoGames`, vous constaterez la présence de nombreux membres absents de la définition formelle de `System.Array`.

>[!tip]- Ce qui à changé sous le capot
>### L'implémentation "magique" du Runtime
>
>En réalité, dans le .NET moderne, le **CLR (Common Language Runtime)** injecte automatiquement `IEnumerable<T>` (et d'autres interfaces comme `IList<T>`) dans les tableaux à l'exécution. 
>
>Si vous faites `myArray is IEnumerable<string>`, cela renverra `true`. C'est une petite manipulation du moteur .NET pour s'assurer que les tableaux s'intègrent parfaitement partout.
>
>### L'optimisation par "Spécialisation"
>
>C'est le point le plus important pour **.NET 10** :
>
>- **Avant :** Toutes les méthodes d'extension de `System.Linq.Enumerable` traitaient les tableaux comme n'importe quelle séquence (`IEnumerable<T>`). C'était lent car cela demandait beaucoup de "cast" et d'allocations.
>- **Maintenant :** Microsoft a ajouté des **chemins rapides (Fast Paths)**. Quand vous appelez `.First()` ou `.Max()` sur un tableau, LINQ détecte immédiatement qu'il s'agit d'un `System.Array` et court-circuite la logique générique pour utiliser une boucle `for` ultra-rapide sur la mémoire directe du tableau.
>
>### Les méthodes "intrinsèques"
>
>Depuis .NET 7/8, certaines méthodes comme `Max()` ou `Sum()` ont été déplacées ou optimisées directement dans les bibliothèques de base pour utiliser des instructions processeur (SIMD). Le tableau ne se contente plus de "bénéficier" de LINQ, il collabore activement avec lui pour la performance.
>
>---
>*En .NET 10, ce n'est plus juste une "classe utilitaire" qui dépanne, c'est une intégration chirurgicale où le compilateur et le runtime travaillent ensemble pour que `myArray.Where(...)` soit presque aussi performant qu'un code écrit à la main.*
>
>>[!example] Comment vérifier à l'exécution les interfaces implémentée par un type
>>```cs
>>string[] currentVideoGames = { "Morrowind", "Uncharted 2", "Fallout 3" };
>>
>>Console.WriteLine("--- Interfaces de System.Array ---");
>>foreach (var i in currentVideoGames.GetType().GetInterfaces())
>>{
>>    Console.WriteLine($"Interface : {i.Name}");
>>}
>>```
>
>>[!info] Pourquoi le livre dit-il qu'il ne l'implémente pas "directement" ?
>>
>>C'est une précision de puriste : si vous allez voir le code source de la classe `Array` sur [GitHub](https://github.com/dotnet/runtime), vous verrez qu'elle n'implémente que les versions **non-génériques** (`IEnumerable`, `IList`). Les versions génériques (avec `<T>`) sont ajoutées au moment où vous compilez et exécutez votre programme pour un type spécifique (comme `string[]`).

## Le rôle de l'exécution différée

>[!check] Cette sous-section est encore plus vraie avec les versions de .NET récentes. 

Un autre point important concernant les expressions de requête LINQ est que, **lorsqu'elles renvoient une séquence, elles ne sont réellement évaluées qu'après itération sur la séquence résultante**. Formellement, on parle d'*exécution différée.* **==L'avantage de cette approche est que vous pouvez appliquer la même requête LINQ plusieurs fois au même conteneur et être assuré d'obtenir les résultats les plus récents==**. Prenons l'exemple de la mise à jour suivante de la méthode `QueryOverInts()` :

```cs
static void QueryOverInts()
{
    // Collection expression (C# 12)
    int[] numbers = [10, 20, 30, 40, 1, 2, 3, 8];

    var subset = from i in numbers where i < 10 select i;

    // L'instruction LINQ est évaluée ici !
    // i.e: le programme fait les calculs maintenant.
    foreach (var i in subset)
    {
        Console.WriteLine($"{i} < 10");
    }

    // Change des données dans le tableau
    numbers[0] = 4;

    // Nouvelle évaluation!
    foreach (var j in subset)
    {
        Console.WriteLine($"{j} < 10");
    }
    
    Console.WriteLine();
    ReflectOverQueryResults(subset);
}
```

>[!note] 
>Lorsqu'une instruction LINQ sélectionne un seul élément (à l'aide de `First`/`FirstOrDefault`, `Single`/`SingleOrDefault` ou de toute autre méthode d'agrégation), la requête est exécutée immédiatement. Les méthodes `First`, `FirstOrDefault`, `Single` et `SingleOrDefault` sont présentées dans la section suivante. Les méthodes d'agrégation sont abordées plus loin dans ce chapitre.

>[!tip] Ce qu'il se passe sous le capot en .NET 10
>
>Bien que l'exécution soit immédiate, .NET 10 a ajouté des optimisations pour que ce soit plus rapide :
>
>1. **Le court-circuitage :** Si vous faites `.Where(condition).First()`, l'itérateur moderne ne va pas filtrer toute la liste. Il va s'arrêter dès qu'il trouve le **premier** élément qui correspond à la condition et fermer la boucle immédiatement.
>2. **L'accès direct :** Si vous appelez `.First()` sur un tableau (`int[]`), .NET 10 ne passe plus par un itérateur. Il vérifie simplement si la taille du tableau est > 0 et vous donne l'élément `[0]`.

##### Tableau 13-3: Temps d'exécution et type de retour selon les méthodes LINQ

| Méthode                              | Type de retour         | Exécution             |
| ------------------------------------ | ---------------------- | --------------------- |
| `Where`, `Select`, `OrderBy`, `Take` | `IEnumerable<T>`       | **Différée** (Lazy)   |
| `First`, `Last`, `Single`, `Any`     | `T` ou `bool`          | **Immédiate** (Eager) |
| `Count`, `Sum`, `Average`, `Max`     | `int`, `double`, etc.  | **Immédiate** (Eager) |
| `ToList`, `ToArray`, `ToDictionary`  | `List<T>`, `T[]`, etc. | **Immédiate** (Eager) |

Si vous exécutiez à nouveau le programme, vous obtiendriez le résultat suivant. ==Remarquez que lors de la deuxième itération sur la séquence demandée, un élément supplémentaire apparaît, car vous avez attribué à la première élément du tableau une valeur inférieure à dix==.

```
1 < 10
2 < 10
3 < 10
8 < 10
4 < 10
1 < 10
2 < 10
3 < 10
8 < 10
```

L'un des avantages de Visual Studio est que, si vous définissez un point d'arrêt (*breakpoint*) avant l'évaluation d'une requête LINQ, vous pouvez en visualiser le contenu pendant une session de débogage. Il suffit de placer le curseur de la souris sur la variable de résultat LINQ (sous-ensemble dans l'image suivante). Vous aurez alors la possibilité d'évaluer la requête à ce moment-là en développant l'option « Affichage des résultats ».

![[Figure 13.1.png|Débogger des expressions LINQ]]

### `DefaultIfEmpty` (Nouveauté C# 10.0)

**Nouveauté de C# 10, la méthode `DefaultIfEmpty()` renvoie les éléments de la séquence ou une valeur par défaut si la séquence est vide**. L'exécution de la requête est différée jusqu'à ce que la liste soit parcourue. L'exemple suivant illustre `DefaultIfEmpty()` en action :

```cs
static void DefaultWhenEmpty()
{
    Console.WriteLine("Default When Empty");

    // Collection expression (C# 12)
    int[] numbers = [10, 20, 30, 40, 1, 2, 3, 8];

    // Retourne tout les nombres
    foreach (var i in numbers.DefaultIfEmpty(-1))
    {
        Console.WriteLine($"{i},");
    }
    Console.WriteLine();

    // Renvoit -1 car la séquence est vide.
    foreach (
        var i in (from i in numbers where i > 99 select i).DefaultIfEmpty(-1)
    )
    {
        Console.WriteLine($"{i},");
    }
    Console.WriteLine();
}
```

## Le rôle de l'exécution immédiate

**Lorsque vous devez évaluer une expression LINQ renvoyant une séquence en dehors du contexte d'une boucle `foreach`, vous pouvez appeler n'importe quelle méthode d'extension définie par le type `Enumerable`, comme `ToArray<T>()`, `ToDictionary<TSource,TKey>()` et `ToList<T>()`**. ***==Ces méthodes exécutent une requête LINQ à l'instant précis où vous les appelez afin d'obtenir un aperçu des données. Une fois cette opération effectuée, l'aperçu des données peut être manipulé indépendamment.==***

De plus, **==si vous recherchez un seul élément, la requête est exécutée immédiatement. `First()` renvoie le premier élément de la séquence==** (*==et doit toujours être utilisé avec `OrderBy()` ou `OrderByDescending()`==*). **==`FirstOrDefault()` renvoie la valeur par défaut du type d'élément dans la liste s'il n'y en a aucune à renvoyer, par exemple lorsque la séquence d'origine est vide ou que la clause `Where()` filtre tous les éléments==**. **==La fonction `Single()` renvoie également le premier élément de la séquence==** (*==selon `Orderby()`/`OrderByDescending()`, ou l'ordre des éléments s'il n'y a pas de clause de tri==*). **==Comme son homologue, `SingleOrDefault()` renvoie la valeur par défaut du type d'élément s'il n'y a aucun élément dans la séquence==** (==ou si tous les enregistrements sont filtrés par une clause `Where`==). **Si aucun enregistrement n'est renvoyé, `First()` et `Single()` lèvent une exception indiquant qu'aucun enregistrement n'a été renvoyé, tandis que `FirstOrDefault()` et `SingleOrDefault()` renvoient simplement `null`**. ***==La différence entre `First()`/`FirstOrDefault()` et `Single()`/`SingleOrDefault()` est que `Single()`/`SingleOrDefault()` lèvera une exception si la requête renvoie plus d'un élément.==***

```cs
static void ImmediateExecution()
{
    // Ici, j'utilise les méthodes d'extension, contrairement au livre
    // qui utilise la syntaxe de requête.
    Console.WriteLine("Immediate Execution");

    // Collection expression (C# 12)
    int[] numbers = [10, 20, 30, 40, 1, 2, 3, 8];

    // Obtient le premier élément dans l'ordre de la séquence
    int number = numbers.Select(i => i).First();
    Console.WriteLine($"First is {number}");

    // Obtient le premier élément dans l'ordre de la requête
    number = numbers.OrderBy(i => i).Select(i => i).First();
    Console.WriteLine($"First is {number}");

    // Obtient le seul élément qui correspond à la requête.
    number = numbers.Where(i => i > 30).Select(i => i).Single();
    Console.WriteLine($"Single is {number}");

    // Renvois null si rien n'est obtenu
    number = numbers.Where(i => i > 99).Select(i => i).FirstOrDefault();
    number = numbers.Where(i => i > 99).Select(i => i).SingleOrDefault();

    try
    {
        // Lève une exception si aucun enregistrement n'est renvoyé.
        number = numbers.Where(i => i > 99).Select(i => i).First();
    }
    catch (Exception ex)
    {
        Console.WriteLine($"An exception occurred: {ex.Message}");
    }

    try
    {
        // Lève une exception si aucun enregistrement n'est renvoyé.
        number = numbers.Where(i => i > 99).Select(i => i).Single();
    }
    catch (Exception ex)
    {
        Console.WriteLine($"An exception occurred: {ex.Message}");
    }

    try
    {
    
        // Lève une exception si plus qu'un seul élément
        // répondent à la requête.
        number = numbers.Where(i => i > 10).Select(i => i).Single();
    }
    catch (Exception ex)
    {
        Console.WriteLine($"An exception occurred: {ex.Message}");
    }

    // Récupère les données MAINTENANT sous forme d'int[].
    // Collection expression (C# 12)
    // + opérateur de propagation (plus optimisé que ToArray())
    int[] subsetAsIntArray = [.. numbers.Where(i => i < 10).Select(i => i)];

    // Récupère les données MAINTENANT sous forme de List<int>.
    // Collection expression (C# 12)
    // + opérateur de propagation (mème syntaxe pour toutes les collections)
    List<int> subsetAsListOfInt =
    [
        .. numbers.Where(i => i < 10).Select(i => i),
    ];
}
```

>[!warning] Si on ne transforme pas la donnée, **supprimez le `Select()` lors de l'utilisation des méthodes d'extensions**.
>
>>[!tip]- Raccourcis possible en C# moderne avec les exemples listé si-dessus.
>>
>>**Comme les éléments constituant le tableau ne sont pas modifié** (avec une méthode `Select()` dont la signature de la fonction de rappelle est différente de `x => x`) Les méthodes `First()`, `Single()` et leurs dérivées prennent un délégué en paramètre, permettant de supprimer les appels à `Where()` comme ceci:
>>```cs
>>static void ImmediateExecution()
>>{
>>    Console.WriteLine("* Imediate Execution *");
>>
>>    int[] numbers = [10, 20, 30, 40, 1, 2, 3, 8];
>>
>>    int number = numbers.First();
>>    Console.WriteLine($"First is {number}");
>>
>>    number = numbers.OrderBy(i => i).First();
>>    Console.WriteLine($"First is {number}");
>>
>>    number = numbers.Single(i => i > 30);
>>    Console.WriteLine($"Single is {number}");
>>
>>    number = numbers.FirstOrDefault(i => i > 99, -1);
>>    number = numbers.SingleOrDefault(i => i > 99, -1);
>>
>>    try
>>    {
>>        number = numbers.First(i => i > 99);
>>    }
>>    catch (Exception ex)
>>    {
>>        Console.WriteLine($"An exception occured: {ex.Message}");
>>    }
>>    try
>>    {
>>        number = numbers.Single(i => i > 99);
>>    }
>>    catch (Exception ex)
>>    {
>>        Console.WriteLine($"An exception occured: {ex.Message}");
>>    }
>>
>>    try
>>    {
>>        // Lève une exception si plus qu'un seul élément
>>        // répondent à la requête.
>>        number = numbers.Single(i => i > 10);
>>    }
>>    catch (Exception ex)
>>    {
>>        Console.WriteLine($"An exception occured: {ex.Message}");
>>    }
>>
>>    int[] subsetAsIntArray = [.. numbers.Where(i => i < 10)];
>>    List<int> subsetAListOfInt = [.. numbers.Where(i => i < 10)];
>>}
>>```

Notez que l'expression LINQ entière est placée entre parenthèses afin de la convertir dans le type sous-jacent approprié (quel qu'il soit) pour appeler les méthodes d'extension d'`Enumerable`[^1]. 

Rappelez-vous également du [[Chapitre 10#Spécification des paramètres de type pour les membres génériques|Chapitre 10]] : **lorsque le compilateur C# peut déterminer sans ambiguïté le paramètre de type d'un générique, il n'est pas nécessaire de le spécifier**. Vous pouvez donc également appeler `ToArray()` (ou `ToList()`) comme suit :

```cs
int[] subsetAsIntArray =
	(from i in numbers where i < 10 select i).ToArray();
```

**==L'utilité de l'exécution immédiate est évidente lorsqu'il est nécessaire de renvoyer les résultats d'une requête LINQ à un appelant externe==**. Et, comme par hasard, c'est précisément le sujet du prochain chapitre.

### Définition de la valeur par défaut pour les méthodes `[First/Last/Single]OrDefault` (Nouveauté C# 10)

**Les méthodes `FirstOrDefault()`, `SingleOrDefault()` et `LastOrDefault()` ont été mises à jour dans .NET 6/C# 10 pour permettre la spécification de la valeur par défaut lorsque la requête ne renvoie aucun élément.** ==La version de base de ces méthodes définit automatiquement la valeur par défaut== (`0` pour un nombre, `null` pour une classe, etc.). **Désormais, vous pouvez définir la valeur par défaut par programmation lorsqu'aucun enregistrement n'est renvoyé.**

Prenons l'exemple simple suivant. La requête LINQ d'exemple ne renvoie aucun enregistrement. Au lieu de renvoyer la valeur par défaut zéro, chaque méthode renvoie un nombre négatif différent.

```cs
static void SettingDefaults()
{
    int[] numbers = Array.Empty<int>();
    var query = numbers.Where(i => i > 100);
    
    var number = query.FirstOrDefault(-1);
    Console.WriteLine(number);
   
    number = query.SingleOrDefault(-2);
    Console.WriteLine(number);

    number = query.LastOrDefault(-3);
    Console.WriteLine(number);
}
```

# Retourner le résultat d'une requête LINQ

**Il est possible de définir un champ au sein d'une classe (ou structure) dont la valeur est le résultat d'une requête LINQ**. Cependant, **==pour ce faire, vous ne pouvez pas utiliser le typage implicite (car le mot-clé `var` ne peut pas être utilisé pour les champs), et la cible de la requête LINQ ne peut pas être une donnée d'instance; elle doit donc être statique==**. Compte tenu de ces limitations, vous aurez rarement besoin d'écrire un code comme celui-ci :

```cs
class LINQBasedFieldsAreClunky
{
	 private static string[] currentVideoGames =
	{
		"Morrowind", 
		"Uncharted 2",
		"Fallout 3", 
		"Daxter", 
		"System Shock 2"
	};
	
	// Impossible d'utiliser le typage implicite ici ! 
	// Le type du sous-ensemble doit être connu !
	private IEnumerable<string> subset =
		from g in currentVideoGames
		where g.Contains(" ")
		orderby g
		select g;
	
	public void PrintGames()
	{ 
		foreach (var item in subset)
		{ 
			Console.WriteLine(item);
		}
	}
}
```

**Souvent, les requêtes LINQ sont définies dans la portée d'une méthode ou d'une propriété**. De plus, pour simplifier votre programmation, **==la variable utilisée pour stocker le jeu de résultats sera stockée dans une variable locale à typage implicite à l'aide du mot-clé `var`==**. Or, *==rappel du [[Chapitre 3#Comprendre les restrictions relatives aux variables de type implicite|Chapitre 3]] que les variables à typage implicite ne peuvent pas être utilisées pour définir les paramètres, les valeurs de retour ou les champs d'une classe ou d'une structure==*.

Partant de ce constat, vous pourriez vous demander ==comment renvoyer le résultat d'une requête à un appelant externe==. La réponse est : **cela dépend**. **==Si votre jeu de résultats est composé de données fortement typées, comme un tableau de chaînes de caractères ou une `List<Car>`, vous pouvez abandonner l'utilisation du mot-clé `var` et utiliser un type `IEnumerable<T>` ou un type `IEnumerable` approprié (car `IEnumerable<T>` étend `IEnumerable`)==**. Prenons l'exemple suivant pour une nouvelle application console nommée *LinqRetValues* :

```cs
Console.Title = "LINQ Return Values";
Console.WriteLine("***** LINQ Return Values *****\n");

IEnumerable<string> subset = GetStringSubset();

foreach (string item in subset)
{
    Console.WriteLine(item);
}

static IEnumerable<string> GetStringSubset()
{
    string[] colors =
    [
        "Light Red",
        "Green",
        "Yellow",
        "Dark Red",
        "Red",
        "Purple",
    ];

    // Notez que subset est un objet compatible avec IEnumerable<string>.
    IEnumerable<string> theRedColors =
        from c in colors
        where c.Contains("Red")
        select c;
    return theRedColors;
}
```

Le résultat est comme attendu : 

```
***** LINQ Return Values *****

Light Red
Dark Red
Red
```

## Retourner des résultats LINQ par exécution immédiate

**Cet exemple fonctionne comme prévu uniquement parce que la valeur de retour de `GetStringSubset()` et la requête LINQ à l'intérieur de cette méthode sont fortement typées**. ==Si vous aviez utilisé le mot-clé `var` pour définir la variable de sous-ensemble, il serait permis de retourner la valeur *uniquement* si la méthode est toujours prototypée pour retourner `IEnumerable<string>`== (et si la variable locale implicitement typée est effectivement compatible avec le type de retour spécifié).

***==Comme il est quelque peu fastidieux de manipuler `IEnumerable<T>`, vous pouvez utiliser l'exécution immédiate==***. Par exemple, **au lieu de retourner `IEnumerable<string>`, vous pouvez simplement retourner un tableau de chaînes (`string[]`), si vous transformez la séquence en un tableau fortement typé**. Considérez cette nouvelle méthode du fichier *Program.cs*, qui fait exactement cela :

```cs
static string[] GetStringSubsetAsArray()
{
    string[] colors =
    [
        "Light Red",
        "Green",
        "Yellow",
        "Dark Red",
        "Red",
        "Purple",
    ];

    var theRedColors = from c in colors where c.Contains("Red") select c;

    // Mappez les résultats dans un tableau.
    // syntaxe C# 12, plus optimisé que .ToArray()
    return [.. theRedColors];
}
```

Ainsi, **==l'appelant peut ignorer que son résultat provient d'une requête LINQ et simplement travailler avec le tableau de `string` comme prévu==**. Voici un exemple :

```cs
foreach (string item in GetStringSubsetAsArray())
{
    Console.WriteLine(item);
}
```

**L'exécution immédiate est également cruciale lorsqu'on tente de renvoyer à l'appelant les résultats d'une projection LINQ**. ***==Nous aborderons ce sujet plus loin dans ce chapitre==***. Voyons maintenant comment appliquer des requêtes LINQ à des objets de collection génériques et non génériques.

# Application de requêtes LINQ aux objets de collections

Outre l'extraction de résultats à partir d'un simple tableau de données, **les expressions de requête LINQ peuvent également manipuler les données au sein des membres de l'espace de noms `System.Collections.Generic`, tels que le type `List<T>`**. Créez un nouveau projet d'application console nommé *LinqOverCollections* et définissez une classe `Car` de base qui conserve la vitesse actuelle, la couleur, la marque et le surnom, comme illustré dans le code suivant :

```cs
namespace LinqOverCollections;

class Car
{
    public string PetName { get; set; } = "";
    public string Color { get; set; } = "";
    public int Speed { get; set; }
    public string Make { get; set; } = "";
}
```

Maintenant, dans vos instructions de niveau supérieur, définissez une variable locale `List<T>` de type `Car`, et utilisez la syntaxe d'initialisation d'objet pour remplir la liste avec quelques nouveaux objets `Car`.

```cs
using System.Collections;
using LinqOverCollections;

Console.Title = "LINQ over Generic Collections";
Console.WriteLine("***** LINQ over Generic Collections *****\n");

// Crée une List<T> d'objet Car
List<Car> myCars = new List<Car>()
{
    new Car
    {
        PetName = "Henry",
        Color = "Silver",
        Speed = 100,
        Make = "BMW",
    },
    new Car
    {
        PetName = "Daisy",
        Color = "Tan",
        Speed = 90,
        Make = "BMW",
    },
    new Car
    {
        PetName = "Mary",
        Color = "Black",
        Speed = 55,
        Make = "VW",
    },
    new Car
    {
        PetName = "Clunker",
        Color = "Rust",
        Speed = 5,
        Make = "Yugo",
    },
    new Car
    {
        PetName = "Melvin",
        Color = "White",
        Speed = 43,
        Make = "Ford",
    },
};

Console.ReadLine();
```

## Accéder aux sous-objets contenus

**Appliquer une requête LINQ à un conteneur générique est identique à le faire avec un simple tableau, car *LINQ to Objects* peut être utilisé sur tout type implémentant `IEnumerable<T>`**. Cette fois, votre objectif est de construire une expression de requête pour sélectionner uniquement les objets `Car` de la liste `myCars` dont la vitesse est supérieure à $55$. Une fois le sous-ensemble obtenu, vous afficherez le nom de chaque objet `Car` en appelant la propriété `PetName`.

Supposons que vous disposiez de la méthode auxiliaire suivante (prenant un paramètre `List<Car>`), appelée depuis les instructions de niveau supérieur :

```cs
static void GetFastCars(List<Car> myCars)
{
    // Trouve tous les objets Car dans la List<>,
    // où la vitesse est supérieure à 55.
    var fastCars = from c in myCars where c.Speed > 55 select c;

    foreach (var car in fastCars)
    {
        Console.WriteLine($"{car.PetName} is going too fast!");
    }
}
```

**==Remarquez que votre requête ne récupère que les éléments de la liste `List<T>` dont la propriété `Speed` est supérieure à 55==**. Si vous exécutez l'application, vous constaterez que `Henry` et `Daisy` sont les deux seuls éléments correspondant aux critères de recherche.

**Pour créer une requête plus complexe, vous pouvez rechercher uniquement les BMW dont la propriété `Speed` est supérieure à 90**. Dans ce cas, **==il vous suffit de créer une instruction booléenne composée à l'aide de l'opérateur `&&` en C#==**.

```cs
static void GetFastBMWs(List<Car> myCars)
{
    // Troube les BMW raipides
    // Utilisation des méthodes d'extension
    // (pas besoin de Select dans ce cas précis).
    var fastBMW = myCars.Where(c => c.Speed > 90 && c.Make == "BMW");

    foreach (var car in fastBMW)
    {
        Console.WriteLine($"{car.PetName} is going too fast!");
    }
}
```

Dans ce cas, le seul surnom imprimé est `Henry`.

## Application des requêtes LINQ aux collections non génériques

**Rappelons que les opérateurs de requête LINQ sont conçus pour fonctionner avec tout type implémentant `IEnumerable<T>`** (directement ou via des méthodes d'extension). *==Étant donné que `System.Array` dispose de cette infrastructure nécessaire, il peut paraître surprenant que les conteneurs hérités==* (non génériques) *==de `System.Collections` n'en bénéficient pas==*. Heureusement, **il est toujours possible d'itérer sur les données contenues dans des collections non génériques grâce à la méthode d'extension générique `Enumerable.OfType<T>()`.**

==Lors de l'appel à `OfType<T>()` depuis un objet collection non générique (tel qu'un `ArrayList`), il suffit de spécifier le type de l'élément dans le conteneur pour extraire un objet `IEnumerable<T>` compatible==. **Dans le code, vous pouvez stocker ce point de données à l'aide d'une variable implicitement typée.**

Considérez la nouvelle méthode suivante, qui remplit un `ArrayList` avec un ensemble d'objets `Car` (veillez à importer l'espace de noms `System.Collections` dans votre fichier *Program.cs*) :

```cs
void LINQOverArrayList()
{
    Console.Title = "LINQ over ArrayList";
    Console.WriteLine("***** LINQ over ArrayList *****\n");

    // Voici une collection non-générique d'objet Car
    ArrayList myCars = new ArrayList()
    {
        new Car
        {
            PetName = "Henry",
            Color = "Silver",
            Speed = 100,
            Make = "BMW",
        },
        new Car
        {
            PetName = "Daisy",
            Color = "Tan",
            Speed = 90,
            Make = "BMW",
        },
        new Car
        {
            PetName = "Mary",
            Color = "Black",
            Speed = 55,
            Make = "VW",
        },
        new Car
        {
            PetName = "Clunker",
            Color = "Rust",
            Speed = 5,
            Make = "Yugo",
        },
        new Car
        {
            PetName = "Melvin",
            Color = "White",
            Speed = 43,
            Make = "Ford",
        },
    };

    // Transforme l'ArrayList en un type compatible IEnumerable<T>
    var myCarsEnum = myCars.OfType<Car>();

    // Crée une expression de requête ciblant le type compatible.
    var fastCars = from c in myCarsEnum where c.Speed > 55 select c;
    foreach (var car in fastCars)
    {
        Console.WriteLine($"{car.PetName} is going too fast!");
    }
}
```

Comme les exemples précédents, cette méthode, lorsqu'elle est appelée depuis les instructions de niveau supérieur, n'affichera que les noms `Henry` et `Daisy`, selon le format de la requête LINQ.

## Filtrage des données avec `OfType<T>()`

>[!success] Très utile, avec les types non-génériques ET génériques

Comme vous le savez, ==les types non génériques peuvent contenir n'importe quelle combinaison d'éléments, car les membres de ces conteneurs== (comme `ArrayList`) ==sont prototypés pour recevoir des `System.Objects`==. Par exemple, supposons qu'une `ArrayList` contienne divers éléments, dont seulement une partie est numérique. **Si vous souhaitez obtenir un sous-ensemble ne contenant que des données numériques, vous pouvez utiliser `OfType<T>()`, car cette fonction filtre chaque élément dont le type est différent du type donné lors des itérations**.

```cs
static void OfTypeAsFilter()
{
    // Extrait les entiers de l'ArrayList
    ArrayList myStuff = new ArrayList();
    myStuff.AddRange(
        new object[] { 10, 400, 8, false, new Car(), "string data" }
    );
    var myInts = myStuff.OfType<int>();

    // Affiche 10, 400 et 8
    foreach (int i in myInts)
    {
        Console.WriteLine($"Int value: {i}");
    }
}
```

>[!tip]- Sous le capot (Réflexion .NET 10)
>
>Si vous passiez `myInts` dans une méthode de réflexion (comme fait précédement), vous verriez un type nommé **`OfTypeIterator<TResult>`**.
>
>Comme pour les autres itérateurs :
>
>1. **Exécution différée :** Le filtrage ne se produit que pendant le `foreach`.
>2. **Optimisation :** .NET 10 utilise des instructions de test de type ultra-rapides pour que le coût du filtrage soit presque nul.

À ce stade, vous avez eu l'occasion d'appliquer des requêtes LINQ à des tableaux, des collections génériques et des collections non génériques. Ces conteneurs contenaient à la fois des types primitifs C# (entiers, chaînes de caractères) et des classes personnalisées. La prochaine étape consiste à découvrir de nombreux opérateurs LINQ supplémentaires permettant de construire des requêtes plus complexes et utiles.

# Étude des opérateurs de requêtes LINQ en C#

C# définit un grand nombre d'opérateurs de requête prêts à l'emploi. Le [[#Tableau 13-3 Opérateurs de requêtes LINQ courants|Tableau 13-3]] répertorie certains des opérateurs de requête les plus couramment utilisés. ==Outre la liste partielle d'opérateurs présentée dans le [[#Tableau 13-3 Opérateurs de requêtes LINQ courants|Tableau 13-3]], **la classe `System.Linq.Enumerable` fournit un ensemble de méthodes qui ne possèdent pas de notation abrégée directe pour les opérateurs de requête C#, mais sont exposées en tant que méthodes d'extension**==. **Ces méthodes génériques peuvent être appelées pour transformer un ensemble de résultats de diverses manières** (`Reverse<>()`, `ToArray<>()`, `ToList<>()`, etc.). **Certaines servent à extraire des singletons d'un ensemble de résultats, d'autres à effectuer diverses opérations d'ensembles** (`Distinct<>()`, `Union<>()`, `Intersect<>()`, etc.), **et d'autres encore à agréger les résultats** (`Count<>()`, `Sum<>()`, `Min<>()`, `Max<>()`, etc.).

##### Tableau 13-3: Opérateurs de requêtes LINQ courants

| Opérateur de requête                 | Description                                                                                                                                                            |
| ------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `from`, `in`                         | Utilisé pour définir la structure de base de toute expression LINQ, permettant d'extraire un sous-ensemble de données d'un conteneur approprié.                        |
| `where`                              | Utilisé pour définir une restriction quant aux éléments à extraire d'un conteneur                                                                                      |
| `select`                             | Utilisé pour sélectionner une séquence dans le conteneur.                                                                                                              |
| `join`, `on`, `equals`, `into`       | Effectue des jointures en fonction d'une clé spécifiée. Notez que ces jointures n'ont pas nécessairement de lien avec les données d'une base de données relationnelle. |
| `orderby`, `ascending`, `descending` | Permet de trier le sous-ensemble résultant par ordre croissant ou décroissant.                                                                                         |
| `groupby`                            | Génère un sous-ensemble de données regroupées selon une valeur spécifiée.                                                                                              |

Pour commencer à explorer des requêtes LINQ plus complexes, créez un nouveau projet d'application console nommé *FunWithLinqExpressions*. Ensuite, vous devez définir un tableau ou une collection de données d'exemple. Pour ce projet, vous créerez un tableau d'objets `ProductInfo`, défini dans le code suivant :

```cs
namespace FunWithLinqExpressions;

class ProductInfo
{
    public string Name { get; set; } = "";
    public string Description { get; set; } = "";
    public int NumberInStock { get; set; } = 0;

    public override string ToString() =>
        $"Name={Name}, Description={Description}, Number in Stock={NumberInStock}";
}
```

Remplissez maintenant un tableau avec un lot d'objets `ProductInfo` dans votre code appelant.

```cs
using FunWithLinqExpressions;

Console.Title = "Fun with Query Expressions";
Console.WriteLine("***** Fun with Query Expressions *****\n");

// Ce tableau sera la base de nos test...
ProductInfo[] itemsInStock = new[]
{
    new ProductInfo
    {
        Name = "Mac's Coffee",
        Description = "Coffee with TEETH",
        NumberInStock = 24,
    },
    new ProductInfo
    {
        Name = "Milk Maid Milk",
        Description = "Milk cow's love",
        NumberInStock = 100,
    },
    new ProductInfo
    {
        Name = "Pure Silk Tofu",
        Description = "Bland as Possible",
        NumberInStock = 120,
    },
    new ProductInfo
    {
        Name = "Crunchy Pops",
        Description = "Cheezy, peppery goodness",
        NumberInStock = 2,
    },
    new ProductInfo
    {
        Name = "RipOff Water",
        Description = "From the tap to your wallet",
        NumberInStock = 100,
    },
    new ProductInfo
    {
        Name = "Classic Valpo Pizza",
        Description = "Everyone loves pizza!",
        NumberInStock = 73,
    },
};

// On appellera différentes méthode ici!

Console.ReadLine();
```

## Syntaxe de sélection de base

Étant donné que la correction syntaxique d'une expression de requête LINQ est validée à la compilation, *==il est important de noter que l'ordre des opérateurs est crucial==*. **En résumé, toute expression de requête LINQ est construite à l'aide des opérateurs `from`, `in` et `select`**. Voici le modèle général à suivre :

```cs
var résultat =

from objetCorrespondant in contenant
select objetCorrespondant;
```

==L'élément suivant l'opérateur `from` représente un élément correspondant aux critères de la requête LINQ, qui peut être nommé comme vous le souhaitez==. ==L'élément suivant l'opérateur `in` représente le conteneur de données à interroger== (un tableau, une collection, un document XML, etc.). Voici une requête simple, qui se contente de sélectionner tous les éléments du conteneur (comportement similaire à une instruction SQL `SELECT *` dans une base de données). Prenons l'exemple suivant :

```cs
static void SelectEverything(ProductInfo[] products)
{
    // Récupère tout!
    Console.WriteLine("All product details:");
    var allProducts = from p in products select p;

    foreach (var prod in allProducts)
    {
        // Pas besion de .ToString() (différent du livre)
        Console.WriteLine(prod);
    }
}
```

Honnêtement, *==cette requête n'est pas très utile, étant donné que votre sous-ensemble est identique à celui des données du paramètre entrant==*. Si vous le souhaitez, vous pouvez extraire uniquement les valeurs « Nom » de chaque voiture en utilisant la syntaxe de sélection suivante :

```cs
static void ListProductName(ProductInfo[] products)
{
    // Maintenant on récupère le nom du produit.
    Console.WriteLine("Only product names:");
    var names = from p in products select p.Name;

    foreach (var n in names)
    {
        Console.WriteLine($"Name: {n}");
    }
}
```

## Obtention de sous-ensembles de données

**Pour obtenir un sous-ensemble spécifique d'un conteneur, vous pouvez utiliser l'opérateur `where`**. Dans ce cas, le modèle général devient le code suivant :

```cs
var resultat =
	from objet
	in contenant
	where ExpressionBooléenne
	select objet;
```

**==Notez que l'opérateur `where` attend une expression qui renvoie une valeur booléenne==**. Par exemple, pour extraire de l'argument `ProductInfo[]` uniquement les articles dont le stock est supérieur à $25$ unités, vous pouvez écrire le code suivant :

```cs
static void GetOverStock(ProductInfo[] products)
{
    Console.WriteLine("The overstock item!");

    // Récupère seulement les éléments ou l'on en possède
    // plus que 25 dans le stock.
    var overstock = from p in products where p.NumberInStock > 25 select p;

    foreach (ProductInfo c in overstock)
    {
      Console.WriteLine(c.ToString());
    }
}
```

Comme indiqué précédemment dans ce chapitre, **lors de la construction d'une clause `where`, il est permis d'utiliser tous les opérateurs C# valides pour créer des expressions complexes**. Par exemple, reprenons cette requête qui extrait uniquement les BMW roulant à au moins 100 km/h :

```cs
// Récupère les BMW qui roule à au moins 100 km/h.
var onlyFastBMWs =
	from c
	in myCars
	where c.Make == "BMW" && c.Speed ​​>= 100
	select c;
```

## Pagination des données

**Pour obtenir un certain nombre d'enregistrements, vous pouvez utiliser les méthodes `Take()`/`TakeWhile()`/`TakeLast()` et `Skip()`/`SkipWhile()`/`SkipLast()`**. Les premières méthodes renvoient le nombre d'enregistrements spécifié (`Take()`), tous les enregistrements pour lesquels la condition est vraie (`TakeWhile()`), ou le dernier nombre d'enregistrements spécifié (`TakeLast()`). Les secondes méthodes ignorent le nombre d'enregistrements spécifié (`Skip()`), ignorent tous les enregistrements pour lesquels la condition est vraie (`SkipWhile()`), ou ignorent le dernier nombre d'enregistrements spécifié (`SkipLast()`).

**==Ces méthodes sont exposées via l'interface `IEnumerable`==**. Par conséquent, **bien que vous ne puissiez pas utiliser les opérateurs de pagination directement sur les instructions LINQ, vous pouvez les utiliser sur le résultat de l'instruction LINQ.**

Commencez par créer une nouvelle méthode nommée `PagingWithLinq()`. Le premier exemple récupère les trois premiers enregistrements de la liste, puis transmet le résultat à une fonction locale pour affichage. **Les méthodes de pagination (comme la clause `where`) participent à une exécution différée; le type de retour est donc également un `IEnumerable<T>`**. Voici la méthode avec le premier exemple et la fonction auxiliaire locale :

```cs
static void PagingWithLINQ(ProductInfo[] products)
{
    Console.WriteLine("Paging operations");

    IEnumerable<ProductInfo> list = (from p in products select p).Take(3);
    OutputResults("The first 3", list);

    static void OutputResults(string message, IEnumerable<ProductInfo> products)
    {
        Console.WriteLine(message);
        foreach (ProductInfo c in products)
        {
            Console.WriteLine(c);
        }
    }
}
```

**La méthode `TakeWhile()` traite les enregistrements tant qu'une condition est vraie**. ==Cette condition est passée à la méthode sous forme d'expression lambda==, comme ceci :

```cs
list = (from p in products select p).TakeWhile(x => x.NumberInStock > 20);
OutputResults("All while number in stock > 20", list);
```

**Le code précédent renvoie les trois premiers éléments de la liste, car le quatrième enregistrement** (« Crunchy Pops ») **ne contient que deux articles en stock**. Il est important de noter que *==la méthode cesse de traiter les enregistrements lorsque la condition n'est pas remplie, même s'il reste deux articles qui auraient satisfait à la condition==*. **==Si vous devez prendre en compte tous les articles, ajoutez une clause `orderby` à la liste avant d'appeler `TakeWhile()`.==**

La méthode `TakeLast()` traite le dernier nombre d'enregistrements spécifié.

```cs
list = (from p in products select p).TakeLast(2);
OutputResults("The last 2", list);
```

**Les méthodes `Skip()` et `SkipWhile()` fonctionnent de la même manière, à la différence qu'elles ignorent des enregistrements au lieu de les traiter**. L'exemple suivant ignore les trois premiers enregistrements, puis renvoie le reste :

```cs
list = (from p in products select p).Skip(3);
OutputResults("Skipping the first 3", list);
```

Pour ignorer les enregistrements dont le nombre en stock est supérieur à $20$, utilisez le code suivant. **Notez que le même problème de tri se pose avec `SkipWhile()` et `TakeWhile()`**. **==Il est préférable d'utiliser ces deux méthodes lorsque les enregistrements sont triés en conséquence.==**

```cs
list = (from p in products select p).SkipWhile(x => x.NumberInStock > 20);
OutputResults("Skip while number in stock > 20", list);
```

La méthode `SkipLast()` prend tous les enregistrements sauf le dernier nombre spécifié :

```cs
list = (from p in products select p).SkipLast(2);
OutputResults("All but the last 2", list);
```

***==Ces méthodes peuvent être combinées pour véritablement « paginer » les données==***. Pour ignorer trois enregistrements et n'en conserver que deux, exécutez le code suivant :

```cs
list = (from p in products select p).Skip(3).Take(2);
OutputResults("Skip 3 last 2", list);
```

## Pagination des données avec des plages (Nouveauté C# 10.0)

==La prise en charge des plages dans la méthode Take() a été ajoutée dans .NET 6/C# 10, permettant la pagination sans avoir à utiliser simultanément les méthodes `Take()` et `Skip()`==. Notez que les méthodes TakeWhile() et SkipWhile() n'ont pas été mises à jour pour accepter les plages.

>[!tip] En .NET 10, utiliser `Take(..10)` sur un tableau est devenu **extrêmement performant** car LINQ utilise désormais l'opérateur de slicing natif du langage sous le capot (ce qui évite de créer un itérateur complet).

Voici une nouvelle méthode nommée `PagingWithRanges()` qui répète les appels à `Take()` et `Skip()` de l' exemple précédent, en utilisant des plages pour récupérer les mêmes données :

```cs
static void PagingWithRanges(ProductInfo[] products)
{
    Console.WriteLine("Paging Operations");

    IEnumerable<ProductInfo> list = (from p in products select p).Take(..3);
    OutputResults("The first 3", list);

    list = (from p in products select p).Take(3..);
    OutputResults("Skipping the first 3", list);

    list = (from p in products select p).Take(^2..);
    OutputResults("The last 2", list);

    list = (from p in products select p).Take(3..5);
    OutputResults("Skip 3 take 2", list);

    list = (from p in products select p).Take(..^2);
    OutputResults("Skip the last 2", list);

    static void OutputResults(string message, IEnumerable<ProductInfo> products)
    {
        Console.WriteLine(message);
        foreach (ProductInfo c in products)
        {
            Console.WriteLine(c);
        }
    }
}
```

## Pagination des données avec des segments (Nouveauté C# 10.0)

**La méthode `Chunk()` est une nouvelle méthode de pagination ajoutée dans .NET 6/C# 10**. **==Cette méthode prend un paramètre (taille) et divise la source en un objet `Enumerable` d'objets `Enumerable`==**. Par exemple, ==si l'on prend la liste `ProductInfo` et que l'on applique `Chunk(2)`, la valeur de retour est une liste de trois listes, chaque liste interne contenant deux enregistrements.== **Si la liste ne peut pas être divisée de manière égale, la dernière liste contiendra moins d'enregistrements.**

Voici une nouvelle méthode nommée `PagingWithChunks()` qui illustre la méthode `Chunk()` :

```cs
static void PagingWithChunks(ProductInfo[] products)
{
    Console.WriteLine("Chunking Operations");

    IEnumerable<ProductInfo[]> chunks = products.Chunk(size: 2);
    int counter = 0;
    foreach (var chunk in chunks)
    {
        // Combinaison de l'interpolation et de la concaténation.
        OutputResults($"Chunk #{++counter}", chunk);
    }
    static void OutputResults(string message, IEnumerable<ProductInfo> products)
    {
        Console.WriteLine(message);
        foreach (ProductInfo c in products)
        {
            Console.WriteLine(c);
        }
    }
}
```

## Projection de nouveaux types de données

**Il est également possible de projeter de nouveaux formats de données à partir d'une source de données existante**. Supposons que vous souhaitiez utiliser le paramètre entrant `ProductInfo[]` et obtenir un ensemble de résultats ne contenant que le nom et la description de chaque article. **==Pour ce faire, vous pouvez définir une instruction `select` qui génère dynamiquement un nouveau type anonyme.==**

```cs
static void GetNamesAndDescriptions(ProductInfo[] products)
{
    Console.WriteLine("Names and Descriptions:");

    // Création d'un type anonyme projetant les propriétés
    // Name et Descriptions de la classe ProductInfo.
    // Utilisation de Tuples possible 
    // (plus léger en mémoire que le type anonyme)
    //var nameDesc = from p in products select (p.Name, p.Description);
    var nameDesc = from p in products select new { p.Name, p.Description };


    foreach (var item in nameDesc)
    {
        // On pourrait également utiliser directement
        // les propriétés Nom et Description.
        Console.WriteLine(item.ToString());
    }
}
```

*==N'oubliez jamais que lorsqu'une requête LINQ utilise une projection, vous ne pouvez pas connaître le type de données sous-jacent, car celui-ci est déterminé à la compilation==*. Dans ce cas, **le mot-clé `var` est obligatoire**. De plus, rappelez-vous qu'**il est impossible de créer des méthodes avec des valeurs de retour implicitement typées**. Par conséquent, la méthode suivante ne compilerait pas :

>[!tip] Ce problème n'est pas présent si on utilise un `tuple` au lieu d'un type anonyme !

```cs
static var GetProjectedSubset(ProductInfo[] products)
{
	var nameDesc =
		from p in products select new { p.Name, p.Description };
	return nameDesc; // Non !
}
```

**==Lorsque vous devez renvoyer des données projetées à un appelant, une solution consiste à transformer le résultat de la requête en un objet `System.Array` à l'aide de la méthode d'extension `ToArray()`==**. Ainsi, si vous deviez mettre à jour votre expression de requête comme suit :

```cs
// La valeur de retour est maintenant un Array
static Array GetProjectedSubset(ProductInfo[] products)
{
    var nameDesc = from p in products select new { p.Name, p.Description };

    // Mappe l'ensemble d'objet anonymes en un objet Array
    return nameDesc.ToArray();
}
```

vous pouvez invoquer et traiter les données comme suit :

```cs
Array objs = GetProjectedSubset(itemsInStock);
foreach (object o in objs)
{
	Console.WriteLine(o); // Appelle toString() pour tout les objets anonymes.
}
```

Notez que **vous devez utiliser un objet `System.Array` littéral et non la syntaxe de déclaration de tableau C#, étant donné que vous ne connaissez pas le type sous-jacent puisque vous travaillez sur une classe anonyme générée par le compilateur !** Notez également que ==vous ne spécifiez pas le paramètre de type de la méthode générique `ToArray<T>()`, car, là encore, vous ne connaissez le type de données sous-jacent qu'à la compilation, ce qui est trop tard pour votre usage.==

*==Le problème évident est la perte de tout typage fort, car chaque élément de l'objet `Array` est supposé être de type `Object`==*. Néanmoins, **lorsque vous devez renvoyer un ensemble de résultats LINQ résultant d'une projection sur un type anonyme, la transformation des données en un type `Array` (ou un autre conteneur approprié via d'autres membres du type `Enumerable`) est obligatoire.**

## Projection vers différents types de données

Outre la projection vers des types anonymes, **vous pouvez projeter les résultats de votre requête LINQ vers un autre type concret**. ==Ceci permet un typage statique et l'utilisation de `IEnumerable<T>` comme ensemble de résultats.== Pour commencer, créez une version plus simple de la classe `ProductInfo`.

```cs
namespace FunWithLinqExpressions;

class ProductInfoSmall
{
    public string Name { get; set; } = "";
    public string Description { get; set; } = "";

    public override string ToString() =>
        $"Name={Name}, Description={Description}";
}
```

***==La prochaine étape consiste à projeter les résultats de la requête dans une collection d'objets `ProductInfoSmall`, au lieu de types anonymes==***. Ajoutez la méthode suivante à votre classe :

```cs
static void GetNamesAndDescriptionsTyped(ProductInfo[] products)
{
    Console.WriteLine("Names and Descriptions");
    IEnumerable<ProductInfoSmall> nameDesc =
        from p in products
        select new ProductInfoSmall
        {
            Name = p.Name,
            Description = p.Description,
        };

    foreach (ProductInfoSmall item in nameDesc)
    {
        Console.WriteLine(item);
    }
}
```

==Avec les projections LINQ, vous avez le choix entre plusieurs méthodes== (objets anonymes ou fortement typés). Votre décision dépend entièrement de vos besoins métier.

## Obtenir le nombre d'éléments avec Enumerable

Lorsque vous projetez de nouveaux lots de données, vous pouvez avoir besoin de connaître précisément le nombre d'éléments renvoyés dans la séquence. **Pour déterminer le nombre d'éléments renvoyés par une expression de requête LINQ, utilisez simplement la méthode d'extension `Count()` de la classe `Enumerable`**. Par exemple, la méthode suivante trouvera tous les objets `string` d'un tableau local dont la longueur est supérieure à six caractères :

```cs
static void GetCountFromQuery()
{
    string[] currentVideoGames =
    {
        "Morrowind",
        "Uncharted 2",
        "Fallout 3",
        "Daxter",
        "System Shock 2",
    };

    // Récupère le nombre d'éléments de la requête.
    int numb = (
        from g in currentVideoGames
        where g.Length > 6
        select g
    ).Count();

    // Affiche le nombre d'éléments
    Console.WriteLine($"{numb} items honor the LINQ query.");
}
```

## Obtenir le nombre d'éléments sans énumérer l'objet (Nouveauté C# 10.0)

**Introduite dans .NET 6/C# 10, la méthode `TryGetNonEnumeratedCount()` tente d'obtenir le nombre total d'éléments d'un `IEnumerable` sans parcourir la liste**. *==Si le parcours de la liste est nécessaire pour obtenir le compte, la méthode échoue==*. Le code suivant illustre un comptage simple qui n'entraîne pas de problème de performance en parcourant toute la liste :

```cs
static void GetUnenumerateCount(ProductInfo[] products)
{
    Console.WriteLine("Get Unenumerated Count");

    IEnumerable<ProductInfo> query = from p in products select p;
    var result = query.TryGetNonEnumeratedCount(out int count);
    if (result)
    {
        Console.WriteLine($"Total Count: {count}");
    }
    else
    {
        Console.WriteLine("Try Get Count Failed");
    }
}
```

Le code mis à jour suivant échoue lors de l'appel à la fonction locale `GetProducts()`, car le `yield return` doit être énuméré :

```cs
static void GetUnenumerateCount(ProductInfo[] products)
{
    Console.WriteLine("Get Unenumerated Count");

	...

    var newResult = GetProduct(products)
        .TryGetNonEnumeratedCount(out int newCount);
    if (newResult)
    {
        Console.WriteLine($"Total Count: {newCount}");
    }
    else
    {
        Console.WriteLine("Try Get Count Failed");
    }

    static IEnumerable<ProductInfo> GetProduct(ProductInfo[] products)
    {
        // On peut aussi utiliser la propriété Length.
        for (int i = 0; i < products.Count(); i++)
        {
            yield return products[i];
        }
    }
}
```

## Inversion des résultats

**Vous pouvez inverser l'ordre des éléments d'un ensemble de résultats très simplement grâce à la méthode d'extension `Reverse()` de la classe `Enumerable`**. Par exemple, la méthode suivante sélectionne tous les éléments du paramètre `ProductInfo[]` entrant, en ordre inverse :

```cs
static void ReverseEverything(ProductInfo[] products)
{
    Console.WriteLine("Product in reverse:");
    var allProducts = from p in products select p;
    foreach (var prod in allProducts.Reverse())
    {
        Console.WriteLine(prod.ToString());
    }

    // Avec les ranges (slices)
    // for (int i = 1; i <= products.Length; i++)
    // {
    //     // On utilise l'index 'hat' seul, sans le range '..'
    //     var item = products[^i];
    //     Console.WriteLine(item);
    // }
}
```

## Expressions de tri

Comme vous l'avez vu dans les premiers exemples de ce chapitre, **une expression de requête peut utiliser l'opérateur `orderby` pour trier les éléments du sous-ensemble selon une valeur spécifique**. ==Par défaut, le tri est croissant== ; ainsi, un tri par chaîne de caractères donne un ordre alphabétique, un tri par données numériques un ordre croissant, etc. ==Si vous souhaitez afficher les résultats par ordre décroissant, il suffit d'inclure l'opérateur `descending`==. Considérez la méthode suivante :

```cs
static void AlphabetizeProductNames(ProductInfo[] products)
{
    Console.WriteLine("Ordered by Name:");

    // Récupère les noms de produits, dans l'ordre alphabétique.
    var subset = from p in products orderby p.Name select p;
    foreach (var p in subset)
    {
        Console.WriteLine(p.ToString());
    }
}
```

**==Bien que l'ordre croissant soit l'ordre par défaut, vous pouvez clarifier vos intentions en utilisant l'opérateur `ascending`.==**

```cs
var subset = from p in products orderby p.Name ascending select p;
```

**Si vous souhaitez obtenir les éléments par ordre décroissant, vous pouvez le faire via l'opérateur `descending`.**

```cs
var subset = from p in products orderby p.Name descending select p;
```

>[!tip] quand on utilise les méthodes d'extensions, Il faut utilisé `OrderDescending()` ou `OrderByDescending()`

## LINQ comme un meilleur outil pour les diagrammes de Venn

>[!info]- C'est quoi un diagramme de Venn ?
> montre toutes les relations logiques possibles dans une collection finie de différents ensembles ([Lien Wikipédia](https://fr.wikipedia.org/wiki/Diagramme_de_Venn))
>![[Diagramme de Venn.png|Diagramme de Venn|480]]

**La classe `Enumerable` prend en charge un ensemble de méthodes d'extension permettant d'utiliser deux requêtes LINQ** (ou plus) **comme base pour trouver les unions, les différences, les concaténations et les intersections de données**. Prenons l'exemple de la méthode d'extension `Except()`, qui renvoie un ensemble de résultats LINQ contenant la différence entre deux conteneurs, ici la valeur `"Yugo"`.

```cs
static void DisplayDiff()
{
    // Collection expression (C# 12)
    List<string> myCars = ["Yugo", "Aztec", "BMW"];
    List<string> yourCars = ["BMW", "Saab", "Aztec"];

    var carDiff = (from c in myCars select c).Except(
        from c2 in yourCars
        select c2
    );

    Console.WriteLine("Here is what you dont have, but I do:");
    foreach (string s in carDiff)
    {
      Console.WriteLine(s); // Affiche Yugo
    }
}
```

**La méthode `Intersect()` renvoie un ensemble de résultats contenant les données communes à un ensemble de conteneurs**. Par exemple, la méthode suivante renvoie la séquence `Aztec` et `BMW` :

```cs
static void DisplayIntersection()
{
    // Collection expression (C# 12)
    List<string> myCars = ["Yugo", "Aztec", "BMW"];
    List<string> yourCars = ["BMW", "Saab", "Aztec"];

    // Récupère les membres communs aux deux liste.
    var carIntersect = (from c in myCars select c).Intersect(
        from c2 in yourCars
        select c2
    );

    Console.WriteLine("here is what we have in common:");
    foreach (string s in carIntersect)
    {
        Console.WriteLine(s); // Affiche Aztec et BMW
    }
}
```

**La méthode `Union()`, comme son nom l'indique, renvoie un ensemble de résultats incluant tous les membres d'un lot de requêtes LINQ**. Comme pour toute union, ***==vous ne trouverez pas de valeurs répétées si un membre commun apparaît plusieurs fois==***. Par conséquent, la méthode suivante affichera les valeurs `Yugo`, `Aztec`, `BMW` et `Saab` :

```cs
static void DisplayUnion()
{
    // Collection expression (C# 12)
    List<string> myCars = ["Yugo", "Aztec", "BMW"];
    List<string> yourCars = ["BMW", "Saab", "Aztec"];

    // Récupère l'union de ces deux conteneurs
    var carUnion = (from c in myCars select c).Union(
        from c2 in yourCars
        select c2
    );
    
    Console.WriteLine("Here is all the different cars:");
    
    foreach (string s in carUnion)
    {
        Console.WriteLine(s);
    }
}
```

**Enfin, la méthode d'extension `Concat()` renvoie un ensemble de résultats qui est une concaténation directe d'ensembles de résultats LINQ**. Par exemple, la méthode suivante affiche les résultats suivants : `Yugo`, `Aztec`, `BMW`, `BMW`, `Saab` et `Aztec` :

```cs
static void DisplayConcat()
{
    List<string> myCars = ["Yugo", "Aztec", "BMW"];
    List<string> yourCars = ["BMW", "Saab", "Aztec"];
    var carConcat = (from c in myCars select c).Concat(
        from c2 in yourCars
        select c2
    );
    
    Console.WriteLine("Here is everything:");
    
    // Affiche :
    // Yugo Aztec BMW BMW Saab Aztec.
    foreach (string s in carConcat)
    {
        Console.WriteLine(s);
    }
}
```

### Diagrammes de Venn avec sélecteurs (Nouveauté C# 10.0)

**Un nouvel ensemble de méthodes a été introduit dans .NET 6/C# 10, permettant d'utiliser un sélecteur pour trouver les unions, les différences et les intersections de données**. ==Les sélecteurs utilisent une propriété spécifique des objets dans les listes pour déterminer l'action à effectuer.==

>[!info]- Ce qu'il se cache derrière le terme *sélecteur*
> Microsoft et l'auteur du livre utilise le terme *sélecteur* pour désigner l'utilisation faite d'une fonction/méthode. Ici, notre but est dé sélectionner des éléments selon un critère. Pour effectué cela, les méthodes présenté dans le paragraphe suivant utilise des délégués `Func<>`.

**==La méthode d'extension `ExceptBy()` utilise le sélecteur pour supprimer du premier ensemble les enregistrements où la valeur du sélecteur existe dans le second ensemble==**. La méthode suivante crée deux tableau de tuples et utilise la propriété `Age` comme sélecteur. `Claire` et `Pat` ont le même âge que `Lindsey`, mais les valeurs `Age` de `Francis` et `Ashley` ne figurent pas dans la seconde liste. Le résultat de la méthode d'extension `ExceptBy()` est donc `Francis` et `Ashley` :

```cs
static void DisplayDiffBySelector()
{
    var first = new (string Name, int Age)[]
    {
        ("Francis", 20),
        ("Lindsay", 30),
        ("Ashley", 40),
    };

    var second = new (string Name, int Age)[]
    {
        ("Claire", 30),
        ("Pat", 30),
        ("Drew", 33),
    };

    // la méthode ExceptBy prend une collection + un délégué Func.
    // (ici, une expression lambda).
    var result = first.ExceptBy(
        second.Select(x => x.Age),
        product => product.Age
    );

    Console.WriteLine("Except for by selector:");
    foreach (var item in result)
    {
        Console.WriteLine(item); // { ("Francis", 20), ("Ashley", 40) };
    }
}
```

**La méthode `IntersectBy()` renvoie un ensemble de résultats contenant les données communes à un ensemble de conteneurs, en fonction du sélecteur**. La méthode suivante renvoie le tuple `Lindsey`. Notez que même si `Claire` et `Pat` ont également la même valeur `Age` que Lindsey, elles ne sont pas renvoyées car ==la méthode `IntersectBy()` ne renvoie qu'un seul résultat par valeur de sélecteur.==

```cs
static void DisplayInteresctionBySelector()
{
    var first = new (string Name, int Age)[]
    {
        ("Francis", 20),
        ("Lindsay", 30),
        ("Ashley", 40),
    };

    var second = new (string Name, int Age)[]
    {
        ("Claire", 30),
        ("pat", 30),
        ("Drew", 33),
    };

    var result = first.IntersectBy(
        second.Select(x => x.Age),
        person => person.Age
    );

    Console.WriteLine("Intersection by selector:");
    foreach (var item in result)
    {
        Console.WriteLine(item); // { ("Francis", 20), ("Ashley", 40) };
    }
}
```

**==Inverser l'appel entre la première et la deuxième liste produit toujours un seul résultat, qui est le premier tuple de la deuxième liste avec la valeur de sélecteur de $30$.==**

```cs
var result = second.IntersectBy(
	first.Select(x => x.Age),
	person => person.Age
); // Renvoit { ("Claire", 30) }
```

**La méthode `UnionBy()` renvoie un ensemble de résultats comprenant toutes les valeurs du sélecteur, ainsi que le premier élément des listes combinées dont la valeur correspond à la liste**. ==Dans la méthode suivante, notez que bien que chaque valeur `Age` soit représentée dans le résultat, `Claire` et `Pat` ne le sont pas, car leur `Age` est déjà représenté par `Lindsey` :==

```cs
static void DisplayUnionBySelector()
{
    var first = new (string Name, int Age)[]
    {
        ("Francis", 20),
        ("Lindsay", 30),
        ("Ashley", 40),
    };

    var second = new (string Name, int Age)[]
    {
        ("Claire", 30),
        ("pat", 30),
        ("Drew", 33),
    };

    var result = first.UnionBy(second, person => person.Age);
    Console.WriteLine("Union by selector:");
    foreach (var item in result)
    {
        Console.WriteLine(item); // { ("Lindsay", 30) }
    }
}
```

>[!tip]- Pourquoi la méthode `UnionBy()` n'as pas la même signature que les deux autre méthodes ?
>
>- **`ExceptBy` et `InterectBy` sont des filtres :** On part de `First` et on retire des trucs. On a juste besoin de savoir "quoi" retirer (les clés).
>- **`UnionBy` est un Générateur :** On veut créer une nouvelle liste qui contient des éléments venant de `First` **ET** de `Second`. Si `UnionBy` ne prenait que des clés pour `Second`, il ne pourrait pas ajouter les objets complets de la deuxième liste au résultat final !
>
>---
> *La signature de `ExceptBy` et de `IntersectBy` est beaucoup plus proche de la logique **ensembliste** (mathématique) et de ce que l'on fait en SQL.*

==Ces nouvelles méthodes, `ExceptBy()`, `IntersectBy()` et `UnionBy()`, ajoutent beaucoup de puissance à votre arsenal de listes, mais comme vous l'avez vu, elles ne se comportent pas toujours comme prévu.

## Suppression des doublons

Lorsque vous appelez la méthode d'extension `Concat()`, vous pouvez très bien obtenir des entrées redondantes dans le résultat récupéré, ce qui peut être exactement ce que vous souhaitez dans certains cas. Cependant, dans d'autres cas, **vous pourriez vouloir supprimer les entrées en double dans vos données. Pour ce faire, il suffit d'appeler la méthode d'extension `Distinct()`, comme illustré ici :**

```cs
static void DisplayConcatNoDups()
{
    // Collection expression (C# 12)
    List<string> myCars = ["Yugo", "Aztec", "BMW"];
    List<string> yourCars = ["BMW", "Saab", "Aztec"];

    var carConcat = (from c in myCars select c).Concat(
        from c2 in yourCars
        select c2
    );

    // Affiche:
    // Yugo Aztec BMW Saab.
    foreach (string s in carConcat.Distinct())
    {
        Console.WriteLine(s);
    }
}
```

### Suppression des doublons avec des sélecteurs (Nouveauté C# 10.0)

**Une autre nouveauté de .NET 6/C# 10 pour la manipulation des listes est la méthode `DistinctBy()`**. **==Cette méthode==**, comme ses homologues `UnionBy()`, `IntersectBy()` et `ExceptBy()`, **==utilise un sélecteur pour fonctionner==**. L'exemple suivant extrait les enregistrements distincts par `Age`. Notez que la liste finale ne contient que `Lindsey` parmi les trois tuples dont la valeur pour `Age` est $30$. ***==Le tuple retenu est celui qui apparaît en premier dans la liste.==***

>[!info]- Le comportement décrit à la fin du paragraphe est le même qu'en SQL.
>La seule différence est sémantique :
>
>- **SQL `DISTINCT`** : Compare **toute la ligne** (toutes les colonnes).
>- **LINQ `DistinctBy()`** : Ne compare que la **clé** que tu as choisie (l'âge), mais te rend l'objet **entier**.

>[!tip] `DistinctBy` est une **optimisation massive** : il utilise une table de hachage interne (`HashSet`) pour marquer les clés déjà vues et passer à la suite sans jamais stocker les doublons.

>[!warning] L'utilisation de `Concat` et `Distinct` de l'exemple de code précédent ne fonctionnera pas avec comme base un `ProductInfo[]`. 
>La méthode d'extension `Distinct()` **compare les adresses mémoire (références)**. `DistinctBy(p => p.Age)` Compare uniquement la propriété. si une égalité arrive, alors il gardera la première ocurence.

```cs
static void DisplayConcatNoDupsBySelector()
{
    var first = new (string Name, int Age)[]
    {
        ("Francis", 20),
        ("Lindsay", 30),
        ("Ashley", 40),
    };

    var second = new (string Name, int Age)[]
    {
        ("Claire", 30),
        ("pat", 30),
        ("Drew", 33),
    };

    var result = first.Concat(second).DistinctBy(x => x.Age);

    Console.WriteLine("Distinct by selector:");
    foreach (var item in result)
    {
        Console.WriteLine(item); // { (Francis", 20), ("Lindsey", 30), ("Ashley", 40), ("Drew", 33)}
    }
}
```

## Opérations d'agrégation LINQ

**Les requêtes LINQ peuvent également être conçues pour effectuer diverses opérations d'agrégation sur l'ensemble de résultats**. **==La méthode d'extension `Count()` est un exemple d'agrégation==**. ==D'autres possibilités incluent l'obtention d'une moyenne, d'un maximum, d'un minimum ou d'une somme de valeurs à l'aide des membres `Max()`, `Min()`, `Average()` ou `Sum()` de la classe `Enumerable`. Voici un exemple simple :==

```cs
static void AggregateOps()
{
    double[] winterTemps = { 2.0, -21.3, 8, -4, 9, 8.2 };

    // Exemples d'aggrégation divers
    Console.WriteLine($"Max temp: {(from t in winterTemps select t).Max()}");
    Console.WriteLine($"Min temp: {(from t in winterTemps select t).Min()}");
    Console.WriteLine(
        $"Average temp: {(from t in winterTemps select t).Average()}"
    );
    Console.WriteLine(
        $"Sum of all temps: {(from t in winterTemps select t).Sum()}"
    );
}
```

### Agrégation avec sélecteurs (Nouveauté C# 10.0)

**Deux nouvelles fonctions d'agrégation ont été introduites dans .NET 6/C# 10 : `MaxBy()` et `MinBy()`**. En utilisant le tableau `ProductInfo` mentionnée précédemment, voici une méthode qui récupère les valeurs maximale et minimale à partir de la propriété `NumberInStock` :

>[!info] Microsoft a continué d'enrichir LINQ. 
>Avec **.NET 9** (qui accompagne C# 13), deux nouvelles méthodes d'agrégation utilisant des sélecteurs (ou des groupements internes) ont été introduites. Elles sont particulièrement puissantes car elles évitent d'utiliser un `GroupBy` complet, ce qui améliore grandement les performances.
>
>Voici les nouveautés majeures :
>
> 1. `CountBy()` (Nouveauté .NET 9)
>
>	C'est le compagnon idéal pour compter rapidement des occurrences sans créer de structures de données lourdes.
>
>	- **L'intention :** "Compte combien j'ai de produits pour chaque catégorie."
>
> 2. `AggregateBy()` (Nouveauté .NET 9)
>
>	C'est la méthode "ultime" pour les calculs complexes groupés. Elle permet de faire des sommes, des moyennes ou des calculs personnalisés par clé en un seul passage.
>
>	- **L'intention :** "Calcule le total en stock pour chaque catégorie."


```cs
static void AggregateOpsBySelector(ProductInfo[] products)
{
    // Max et Min renvoient un seul objet
    Console.WriteLine(
        $"Max by In Stock: {products.MaxBy(x => x.NumberInStock)}"
    );
    Console.WriteLine(
        $"Min by In Stock: {products.MinBy(x => x.NumberInStock)}"
    );

    /* 
	    Nouvelles méthodes d'aggrégation ajoutée avec .NET 9 (C# 13) 
	*/
    
    // CountBy - Compte les occurence par valeurs.
    foreach (var count in products.CountBy(x => x.NumberInStock))
    {
        Console.WriteLine($"Quantity {count.Key}: present {count.Value} times");
    }

    // AgregateBy - Totalise par critère.
    var aggregate = products.AggregateBy(
        keySelector: p => p.Name.Contains("Milk"),
        seed: 0,
        func: (currentTotal, p) => currentTotal + p.NumberInStock
    );
    foreach (var entry in aggregate)
    {
        string label = entry.Key ? "Dairy products" : "Other products";
        Console.WriteLine($"{label}: {entry.Value} items in total");
    }
}
```

>[!tip] Explication de chaque paramètre de la méthode d'extension `AggregateBy`
>- **Le `keySelector` :** Ici, on ne pointe pas juste une propriété, on crée une **logique de groupe** à la volée (`p.NumberInStock < 50`). C'est la force de LINQ.
>- **Le `seed` :** C'est la valeur initiale. Si tu faisais une multiplication, tu mettrais `1`. Pour une somme, on met `0`.
>- **L'accumulateur (Callback) `func` :** C'est ici que la magie opère. Pour chaque article, .NET regarde sa catégorie (Critique ou Sain) et met à jour le total correspondant.

# Représentation interne des requêtes LINQ

Bien que ce document ne constitue pas une référence LINQ complète, ==les exemples précédents de ce chapitre devraient vous fournir les connaissances nécessaires pour maîtriser la construction d'expressions de requêtes LINQ==. **Vous trouverez d'autres exemples plus loin dans cet ouvrage, notamment dans les chapitres consacrés à Entity Framework Core**. Pour conclure cette première approche de LINQ, la suite de ce chapitre explorera en détail **==les interactions entre les opérateurs de requêtes LINQ en C# et le modèle objet sous-jacent==**.

À ce stade, vous avez été initié à la construction d'expressions de requêtes à l'aide de divers opérateurs de requêtes C# (tels que `from`, `in`, `where`, `orderby` et `select`). Vous avez également constaté que certaines fonctionnalités de l'API LINQ to Objects ne sont accessibles que par l'appel de méthodes d'extension de la classe `Enumerable`. **En réalité, lors de la compilation des requêtes LINQ, le compilateur C# traduit tous les opérateurs LINQ C# en appels de méthodes de la classe `Enumerable`**. ***==De nombreuses méthodes de la classe `Enumerable` ont été prototypées pour accepter des délégués comme arguments.==***

**Beaucoup de ces méthodes requièrent un délégué générique nommé `Func<>`, qui vous a été présenté lors de l'étude des délégués génériques au [[Chapitre 12#Les délégués génériques `Action<>` et `Func<>`|Chapitre 12]]**. Prenons l'exemple de la méthode `Where()` de la classe `Enumerable`, appelée automatiquement lorsque vous utilisez l'opérateur de requête LINQ `where` en C#.

```cs
// Surcharges de la méthode Enumerable.Where<T>().
// Notez que le deuxième paramètre est de type System.Func<>.

public static IEnumerable<TSource> Where<TSource>(
	this IEnumerable<TSource> source,
	System.Func<TSource,int,bool> predicate)

public static IEnumerable<TSource> Where<TSource>(
	this IEnumerable<TSource> source,
	System.Func<TSource,bool> predicate)
```

***==Le délégué `Func<>`==*** (comme son nom l'indique) ***==représente un modèle pour une fonction donnée avec un maximum de $16$ arguments et une valeur de retour==***. Si vous examiniez ce type à l'aide de l'explorateur d'objets de Visual Studio (ou voir le code source de C# [ici](https://github.com/dotnet/dotnet/blob/b0f34d51fccc69fd334253924abd8d6853fad7aa/src/runtime/src/libraries/System.Private.CoreLib/src/System/Function.cs)), vous remarqueriez différentes formes du délégué `Func<>`. Voici un exemple :

```cs
// Les différents formats du délégué Func<>.

...

public delegate TResult Func<T1,T2,T3,T4,TResult>(T1 arg1, T2 arg2, T3 arg3, T4 arg4)

public delegate TResult Func<T1,T2,T3,TResult>(T1 arg1, T2 arg2, T3 arg3)

public delegate TResult Func<T1,T2,TResult>(T1 arg1, T2 arg2)

public delegate TResult Func<T1,TResult>(T1 arg1)

public delegate TResult Func<TResult>()
```

**Étant donné que de nombreux membres de `System.Linq.Enumerable` requièrent un délégué en entrée, lors de leur invocation, vous pouvez soit créer manuellement un nouveau type délégué et définir les méthodes cibles nécessaires, soit utiliser une méthode anonyme C#, soit définir une expression lambda appropriée**. **==Quelle que soit l'approche choisie, le résultat est identique.==**

Bien qu'il soit vrai que l'utilisation des opérateurs de requête LINQ C# soit de loin la méthode la plus simple pour construire une expression de requête LINQ, **==examinons chacune de ces approches possibles afin de mettre en évidence le lien entre les opérateurs de requête C# et le type `Enumerable` sous-jacent.==**

## Création d'expressions de requête avec les opérateurs de requête (Revisité)

Pour commencer, créez un nouveau projet d'application console nommé *LinqUsingEnumerable*. Le fichier *Program.cs* définira une série de méthodes d'assistance statiques (chacune étant appelée dans les instructions de niveau supérieur) afin d'illustrer les différentes manières de créer des expressions de requête LINQ.

La première méthode, `QueryStringsWithOperators()`, offre la méthode la plus simple pour créer une expression de requête et est identique au code présenté dans l'exemple *LinqOverArray* plus haut dans ce chapitre.

```cs
static void QueryStringWithOperators()
{
    Console.Title = "Using Query Operators";
    Console.WriteLine("***** Using Query Operators *****\n");

    string[] currentVideoGames =
    {
        "Morrowind",
        "Uncharted 2",
        "Fallout 3",
        "Daxter",
        "System Shock 2",
    };

    var subset =
        from game in currentVideoGames
        where game.Contains(' ')
        orderby game
        select game;

    // Affiche le résultat
    foreach (string s in subset)
    {
        Console.WriteLine($"Item: {s}");
    }
}
```

**L'avantage évident de l'utilisation des opérateurs de requête C# pour construire des expressions de requête est que les délégués `Func<>` et les appels sur le type `Enumerable` sont abstraits de votre code, car c'est au compilateur C# qu'il incombe d'effectuer cette traduction**. Certes, la **==construction d'expressions LINQ à l'aide de divers opérateurs de requête (`from`, `in`, `where` ou `orderby`) est l'approche la plus courante et la plus simple.==**

## Création d'expressions de requête à l'aide du type `Enumerable` et des expressions lambda

**N'oubliez pas que les opérateurs de requête LINQ utilisés ici sont simplement des versions abrégées permettant d'appeler diverses méthodes d'extension définies par le type `Enumerable`**. Prenons l'exemple de la méthode `QueryStringsWithEnumerableAndLambdas()` suivante, qui traite le tableau de chaînes local en utilisant directement les méthodes d'extension `Enumerable` :

```cs
static void QueryStringsWithEnumerableAndLambdas()
{
    Console.Title = "Using Enumerable / Lambda Expressions";
    Console.WriteLine("***** Using Enumerable / Lambda Expressions *****\n");

    string[] currentVideoGames =
    {
        "Morrowind",
        "Uncharted 2",
        "Fallout 3",
        "Daxter",
        "System Shock 2",
    };

    // Créez une expression de requête à l'aide des méthodes d'extension
    // accordées au tableau via le type Enumerable.
    // Pas besoin d'utiliser la méthode d'extension Select() si on ne modifie pas la donnée.
    var subset = currentVideoGames
        .Where(game => game.Contains(' '))
        .OrderBy(game => game);

    foreach (string s in subset)
    {
        Console.WriteLine($"Item: {s}");
    }
}
```

Ici, vous commencez par appeler la méthode d'extension `Where()` sur le tableau de chaînes de caractères `currentVideoGames`. **Rappelons que la classe `Array` reçoit cette méthode via une extension fournie par `Enumerable`**. **La méthode `Where()` requiert un paramètre délégué de type `System.Func<T1, TResult>`. Le premier paramètre de ce délégué représente les données compatibles `IEnumerable<T>` à traiter** (un tableau de chaînes de caractères dans ce cas), **tandis que le second paramètre représente le résultat de la méthode, obtenu à partir d'une instruction unique passée à l'expression lambda**.

Dans cet exemple de code, **==la valeur de retour de la méthode `Where()` est masquée, mais en réalité, vous manipulez un type `Enumerable`==**. À partir de cet objet, **vous appelez la méthode générique `OrderBy()`, qui requiert également un paramètre délégué `Func<>`.** ==Cette fois-ci, vous transmettez simplement chaque élément l'un après l'autre via une expression lambda appropriée==. **L'appel à `OrderBy()` renvoie une nouvelle séquence ordonnée des données initiales.**

Enfin, vous appelez la méthode `Select()` sur la séquence renvoyée par `OrderBy()`, ce qui produit l'ensemble final de données, stocké dans une variable implicitement typée nommée `subset`.

==Il est vrai que cette requête LINQ== longue » ==est un peu plus complexe à analyser que l'exemple précédent d'opérateur de requête LINQ en C#==. Cette complexité est sans doute due en partie à l'enchaînement des appels à l'aide de l'opérateur point. Voici la même requête, chaque étape étant décomposée en segments distincts (comme vous pouvez le deviner, vous pouvez décomposer la requête globale de différentes manières) :

```cs
static void QueryStringsWithEnumerableAndLambdas2()
{
    Console.Title = "Using Enumerable / Lambda Expressions";
    Console.WriteLine("***** Using Enumerable / Lambda Expressions *****\n");

    string[] currentVideoGames =
    {
        "Morrowind",
        "Uncharted 2",
        "Fallout 3",
        "Daxter",
        "System Shock 2",
    };

    // On décompose les opérations.
    var gamesWithSpaces = currentVideoGames.Where(game => game.Contains(' '));
    var orderedGames = gamesWithSpaces.OrderBy(game => game);
    // Pas beson d'appeler Select() car on ne modifie pas la donnée.
    //var subset = orderedGames.Select(game => game);

    // Affiche le résultat
    foreach (string s in subset)
    {
        Console.WriteLine($"Item: {s}");
    }
}
```

Comme vous le constaterez peut-être, ==la construction d'une expression de requête LINQ à l'aide des méthodes de la classe `Enumerable` est plus verbeuse que l'utilisation des opérateurs de requête C#==. De plus, **étant donné que les méthodes de `Enumerable` requièrent des délégués comme paramètres, il vous faudra généralement écrire des expressions lambda pour permettre le traitement des données d'entrée par la cible déléguée sous-jacente**.

>[!tip] Comme expliqué dans les commentaire des exemples de code, Il n'est plus nécessaire d'appeler `Select()` quand on ne modifie pas les donnée, permettant une optimisation importante ainsi qu'un code plus court.

## Création d'expressions de requête à l'aide du type `Enumerable` et des méthodes anonymes 

==Étant donné que les expressions lambda C# sont simplement des notations abrégées pour l'utilisation des méthodes anonymes==, considérons la troisième expression de requête créée dans la fonction d'assistance `QueryStringsWithAnonymousMethods()`, illustrée ici :

```cs
static void QueryStringsWithAnonymousMethods()
{
    Console.Title = "Using Anonymous Methods";
    Console.WriteLine("***** Using Anonymous Methods *****\n");

    string[] currentVideoGames =
    {
        "Morrowind",
        "Uncharted 2",
        "Fallout 3",
        "Daxter",
        "System Shock 2",
    };

    // Construit le délégué Func<> nécessaire utilisant les méthodes anonymes.
    Func<string, bool> searchFilter = delegate(string game)
    {
        return game.Contains(' ');
    };

    Func<string, string> itemToProcess = delegate(string s)
    {
        return s;
    };

    // Transmets les délégués aux méthodes de Enumerable.
    // Pas beson d'appeler Select() car on ne modifie pas la donnée.
    // Encore plus voyant avec cette syntaxe.
    var subset = currentVideoGames.Where(searchFilter).OrderBy(itemToProcess);

    // Affiche le résultat
    foreach (string s in subset)
    {
        Console.WriteLine($"Item: {s}");
    }
}
```

==Cette itération de l'expression de requête est encore plus verbeuse, car vous créez manuellement les délégués `Func<>` utilisés par les méthodes `Where()`, `OrderBy()` et `Select()` de la classe `Enumerable`==. **L'avantage, c'est que la syntaxe de méthode anonyme permet de centraliser tout le traitement des délégués dans une seule définition de méthode**. Néanmoins, *==cette méthode est fonctionnellement équivalente aux méthodes `QueryStringsWithEnumerableAndLambdas()` et `QueryStringsWithOperators()` créées dans les sections précédentes.==*

>[!tip] Bien qu'elles soient équivalentes en termes de **résultat**, il y a une micro-différence de performance :
>
>- **Lambdas :** Le compilateur C# moderne les optimise agressivement (souvent en les transformant en méthodes statiques pour éviter des allocations).
>- **Méthodes anonymes :** Elles sont un peu moins "flexibles" pour le compilateur et peuvent parfois forcer une allocation de classe de clôture (_closure_) un peu plus lourde, mais c'est négligeable pour 99 % des applications.

## Création d'expressions de requête à l'aide du type `Enumerable` et des délégués bruts

**Enfin, si vous souhaitez créer une expression de requête de manière détaillée, vous pouvez éviter
l'utilisation de lambdas/de la syntaxe des méthodes anonymes et créer directement des cibles de délégué pour chaque type `Func<>`**. Voici la version finale de votre expression de requête, modélisée dans un nouveau type de classe nommé `VeryComplexQueryExpression` :

```cs
class VeryComplexQueryExpression
{
    public static void QueryStringsWithRawDelegate()
    {
        Console.Title = "Using Raw Delegate";
        Console.WriteLine("***** Using Raw Delegate *****\n");

        string[] currentVideoGames =
        {
            "Morrowind",
            "Uncharted 2",
            "Fallout 3",
            "Daxter",
            "System Shock 2",
        };

        // Construit les délégués Func<> nécessaires.
        Func<string, bool> searchFilter = new Func<string, bool>(Filter);
        Func<string, string> itemToProcess = new Func<string, string>(
            ProcessItem
        );

        // Passe les délégués dans les méthodes d'extension Enumerable
        // Pas besoin d'appeler la méthode Select()
        // quand on ne modifie pas la donnée.
        var subset = currentVideoGames
            .Where(searchFilter)
            .OrderBy(itemToProcess);

        // Affiche le résultat
        foreach (string game in subset)
        {
            Console.WriteLine($"Item: {game}");
        }
    }

    // Cible des délégués
    public static bool Filter(string game) => game.Contains(' ');

    public static string ProcessItem(string game) => game;
}
```

Vous pouvez tester cette itération de votre logique de traitement de chaînes en appelant cette méthode dans les instructions de niveau supérieur du fichier *Program.cs*, comme suit :

```cs
VeryComplexQueryExpression.QueryStringsWithRawDelegates();
```

**Si vous exécutiez maintenant l'application pour tester chaque approche possible, il ne serait pas surprenant que le résultat soit identique, quel que soit le chemin emprunté**. ***==Gardez à l'esprit les points suivants concernant la représentation interne des expressions de requête LINQ==*** :

- Les expressions de requête sont créées à l'aide de divers opérateurs de requête C#.
- Les opérateurs de requête sont simplement des notations abrégées pour appeler des méthodes d'extension définies par le type `System.Linq.Enumerable`.
- De nombreuses méthodes d'`Enumerable` requièrent des délégués (`Func<>` en particulier) comme paramètres.
- Toute méthode nécessitant un paramètre de délégué peut recevoir une expression lambda.
- Les expressions lambda sont simplement des méthodes anonymes (ce qui améliore considérablement la lisibilité).
- Les méthodes anonymes sont des notations abrégées pour allouer un délégué brut et construire manuellement une méthode cible de délégué.

Ouf ! C'était peut-être un peu plus technique que vous ne le souhaitiez, mais j'espère que cette discussion vous a aidé à comprendre le fonctionnement interne des si conviviaux opérateurs de requête C#.

# Résumé du chapitre

**LINQ est un ensemble de technologies connexes visant à fournir une méthode unique et symétrique d'interaction avec diverses formes de données**. Comme expliqué dans ce chapitre, **==LINQ peut interagir avec tout type implémentant l'interface `IEnumerable<T>`, y compris les tableaux simples ainsi que les collections de données génériques et non génériques==**.

Comme vous l'avez constaté, **l'utilisation des technologies LINQ se fait grâce à plusieurs fonctionnalités du langage C#**. Par exemple, ***==étant donné que les expressions de requête LINQ peuvent renvoyer un nombre quelconque d'ensembles de résultats, il est courant d'utiliser le mot-clé `var` pour représenter le type de données sous-jacent==***. **Les expressions lambda, la syntaxe d'initialisation d'objets et les types anonymes peuvent tous être utilisés pour construire des requêtes LINQ fonctionnelles et concises.**

**==Plus important encore, vous avez vu comment les opérateurs de requête LINQ en C# sont simplement des notations abrégées pour appeler les membres statiques du type `System.Linq.Enumerable`==**. Comme indiqué, **la plupart des membres de `Enumerable` fonctionnent sur des types délégués `Func<T>`, qui peuvent prendre des adresses de méthodes littérales, des méthodes anonymes ou des expressions lambda comme entrée pour évaluer la requête.**

[^1]: Ce paragraphe se reporte à l'exemple contenu dans le livre qui utilise la syntaxe de requête.

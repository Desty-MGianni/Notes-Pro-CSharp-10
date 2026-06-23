---
title:  "Chapitre 8: Travailler avec des Interfaces"
publish: true
---

# <big><big><big><b><font color =green>Travailler avec des Interfaces</font></b></big></big></big>

Ce chapitre approfondit vos connaissances actuelles en développement orienté objet en abordant le thème de la **programmation par interfaces**. Vous apprendrez à ==définir et à implémenter des interfaces et comprendrez les avantages de la création de types prenant en charge plusieurs comportements==. Vous découvrirez également plusieurs sujets connexes, tels que ==l'obtention de références d'interface, l'implémentation d'interfaces explicites et la construction de hiérarchies d'interfaces==. Vous étudierez aussi **plusieurs interfaces standard définies dans les bibliothèques de classes de base .NET Core**. Sont également abordées ==les nouvelles fonctionnalités introduites dans C# 8 concernant les interfaces, notamment les méthodes d'interface par défaut, les membres statiques et les modificateurs d'accès==. Comme vous le verrez, **vos classes et structures personnalisées peuvent implémenter ces interfaces prédéfinies pour prendre en charge plusieurs comportements utiles, tels que le clonage d'objets, l'énumération d'objets et le tri d'objets**.

# Comprendre les types d'interface

Pour commencer ce chapitre, permettez-moi de vous donner une définition formelle du *type interface*, qui a évolué avec l'introduction de C# 8.0. **Avant C# 8.0, une interface n'était rien de plus qu'un ensemble nommé de *membres abstraits***. Rappelons-nous du [[Chapitre 6#Comprendre les classes abstraites|Chapitre 6]] que les méthodes abstraites sont des protocoles purs, en ce sens qu'elles ne fournissent pas d'implémentation par défaut. **Les membres spécifiques définis par une interface dépendent du comportement précis qu'elle modélise. Autrement dit, une interface exprime un *comportement* qu'une classe ou une structure donnée peut choisir de prendre en charge**. De plus, comme vous le verrez dans ce chapitre, ==une classe ou une structure peut prendre en charge autant d'interfaces que nécessaire, et donc prendre en charge (en substance) plusieurs comportements==.

**La fonctionnalité des méthodes d'interface par défaut, introduite dans C# 8.0, permet aux méthodes d'interface de contenir une implémentation qui peut être redéfinie ou non par la classe implémentant l'interface**. Nous y reviendrons plus tard dans ce chapitre. 

Comme vous pouvez l'imaginer, **les bibliothèques de classes de base .NET Core sont fournies avec de nombreuses interfaces prédéfinies implémentées par diverses classes et structures**. Par exemple, comme vous le verrez au [[Chapitre 20|Chapitre 20]], ADO.NET est fourni avec plusieurs fournisseurs de données permettant de communiquer avec un système de gestion de base de données particulier. Ainsi, sous ADO.NET, vous disposez de nombreux objets de connexion parmi lesquels choisir (`SqlConnection`, `OleDbConnection`, `OdbcConnection`, etc.). **De plus, les fournisseurs de bases de données tiers (ainsi que de nombreux projets open source) fournissent des bibliothèques .NET pour communiquer avec un grand nombre d'autres bases de données** (MySQL, Oracle, etc.), **qui contiennent toutes des objets implémentant ces interfaces**.

***==Bien que chaque classe de connexion ait un nom unique, soit définie dans un espace de noms différent et (dans certains cas) soit regroupée dans un assembly différent, toutes les classes de connexion implémentent une interface commune nommée `IDbConnection`.==***

```cs
// L'interface IDbConnection définit un ensemble commun
// de membres pris en charge par tous les objets de connexion.
public interface IDbConnection : IDisposable
{
	// Methodes
	IDbTransaction BeginTransaction();
	IDbTransaction BeginTransaction(IsolationLevel il);
	void ChangeDatabase(string databaseName);
	void Close();
	IDbCommand CreateCommand();
	void Open();

	// Propiétés
	string ConnectionString { get; set;}
	int ConnectionTimeout { get; }
	string Database { get; }
	ConnectionState State { get; }
}
```

>[!tip] Bonne pratique
>Par convention, les noms d'interfaces .NET commencent par un `I` majuscule. Lorsque vous créez vos propres interfaces personnalisées, il est recommandé de faire de même.

Ne vous préoccupez pas pour l'instant des détails concernant le fonctionnement de ces membres. Retenez simplement que ==l'interface `IDbConnection` définit un ensemble de membres communs à toutes les classes de connexion ADO.NET. De ce fait, vous avez la garantie que chaque objet de connexion prend en charge des membres tels que `Open()`, `Close()`, `CreateCommand()`, etc==. De plus, étant donné que **les membres d'interface sont toujours abstraits, chaque objet de connexion est libre d'implémenter ces méthodes à sa manière**.

Au fil de votre lecture, vous découvrirez des dizaines d'interfaces fournies avec les bibliothèques de classes de base .NET Core. Comme vous le verrez, **ces interfaces peuvent être implémentés dans vos propres classes et structures personnalisées afin de définir des types étroitement intégrés au framework**. De plus, une fois que vous aurez compris l'utilité des interfaces, vous trouverez certainement des raisons de créer les vôtres.

## Types d'interface vs. Classes de base abstraites

Compte tenu de votre travail au [[Chapitre 6#Comprendre les classes abstraites|Chapitre 6]], ==le type interface peut sembler similaire à une classe de base abstraite==. Rappelons que **lorsqu'une classe est marquée comme abstraite, elle *peut* définir un nombre quelconque de membres abstraits afin de fournir une interface polymorphe à tous les types dérivés**. Cependant, même ==lorsqu'une classe définit un ensemble de membres abstraits, elle est également libre de définir un nombre quelconque de constructeurs, de données de champ, de membres non abstraits== (avec implémentation), etc. Les interfaces (avant C# 8.0) ne contenaient *seulement* des définitions de membres. **Désormais, avec C# 8, les interfaces peuvent contenir des définitions de membres (comme des membres abstraits), des membres avec des implémentations par défaut (comme des méthodes virtuelles) et des membres statiques**. ==Il n'y a que deux différences majeures : les interfaces ne peuvent pas avoir de constructeurs non statiques, et une classe peut implémenter plusieurs interfaces. Nous reviendrons sur ce deuxième point plus loin==.

**L'interface polymorphe établie par une classe parente abstraite souffre d'une limitation majeure** : ==seuls les *types dérivés* prennent en charge les membres définis par la classe parente abstraite==. Cependant, ==dans les systèmes logiciels de grande envergure, il est courant de développer plusieurs hiérarchies de classes sans parent commun autre que `System.Object`==. Étant donné que les membres abstraits d'une classe de base abstraite ne s'appliquent qu'aux types dérivés, **il est impossible de configurer des types dans différentes hiérarchies pour prendre en charge la même interface polymorphe**. Pour commencer, créez un nouveau projet d'application console nommé *CustomInterfaces*. Ajoutez la classe abstraite suivante au projet :

```cs
namespace CustomInterfaces;

public abstract class ClonableType
{
    // Seuls les types dérivés peuvent prendre en charge cette
    // « interface polymorphe ». Les classes des autres
    // hiérarchies n'ont pas accès à ce membre abstrait.
    public abstract object Clone();
}
```

Compte tenu de cette définition, **seuls les membres étendant `CloneableType` peuvent implémenter la méthode `Clone()`**. Si vous ==créez un nouvel ensemble de classes n'étendant pas cette classe de base, vous ne pourrez pas bénéficier de cette interface polymorphe==. De plus, n'oubliez pas que **C# ne prend pas en charge l'héritage multiple pour les classes**. Par conséquent, si vous souhaitez créer une `MiniVan` qui soit à la fois une `Car` et une `CloneableType`, cela est impossible.

```cs
// Non ! L'héritage multiple n'est pas possible en C#.
// pour les classes.
public class MiniVan : Car, CloneableType
{
}
```

Comme vous pouvez l'imaginer, **les interfaces viennent à la rescousse**. Une fois définie, ==une interface peut être implémentée par n'importe quelle classe ou structure, dans n'importe quelle hiérarchie, et au sein de n'importe quel espace de noms ou assembly== (écrit dans n'importe quel langage de programmation .NET Core). Comme vous pouvez le constater, **les interfaces sont hautement polymorphes.** Prenons l'exemple de l'interface standard .NET Core nommée `ICloneable`, définie dans l'espace de noms `System`. Cette interface définit une unique méthode nommée `Clone()`.

```cs
public interface ICloneable
{
    object Clone();
}
```

**Si vous examinez les bibliothèques de classes de base de .NET Core, vous constaterez que de nombreux types apparemment sans lien entre eux** (`System.Array`, `System.Data.SqlClient.SqlConnection`, `System.OperatingSystem`, `System.String`, etc.) **implémentent tous cette interface**. Bien que ces types n'aient pas de parent commun (autre que `System.Object`), ==vous pouvez les traiter de manière polymorphe grâce au type d'interface `ICloneable`.

Pour commencer, supprimez le code du fichier *Program.cs* et ajoutez ce qui suit :

```cs
using CustomInterfaces;

Console.Title = "A First Look at Interfaces";
Console.WriteLine("**** A First Look at Interfaces ****\n");

CloneableExample();
```

Ensuite, ajoutez la fonction locale suivante, nommée `CloneMe()`, à vos instructions de niveau supérieur. ==Cette fonction prend un paramètre d'interface `ICloneable`, qui accepte tout objet implémentant cette interface==. Voici le code de la fonction :

```cs
using CustomInterfaces;

Console.Title = "A First Look at Interfaces";
Console.WriteLine("**** A First Look at Interfaces ****\n");

CloneableExample();

static void CloneableExample()
{
    // Toutes ces classes prennent en charge l'interface ICloneable.
    string myStr = "Hello";
    OperatingSystem unixOS = new OperatingSystem(
        PlatformID.Unix,
        new Version()
    );

    // Par conséquent, ils peuvent tous être passés à une méthode prenant ICloneable.
    CloneMe(myStr);
    CloneMe(unixOS);

    static void CloneMe(ICloneable c)
    {
        // Cloner ce que nous obtenons et imprimer le nom.
        object theClone = c.Clone();
        Console.WriteLine($"Your clone is a: {theClone.GetType().Name}");
    }
}
```

Lorsque vous exécutez cette application, **le nom de chaque classe s'affiche dans la console via la méthode `GetType()` héritée de `System.Object`**. Comme expliqué en détail au [[Chapitre 17|Chapitre 17]], ==cette méthode permet de comprendre la composition de n'importe quel type à l'exécution==. Quoi qu'il en soit, le résultat du programme précédent est affiché ci-dessous :

```
**** A First Look at Interfaces ****

Your clone is a: String
Your clone is a: OperatingSystem
```

==Une autre limitation des classes de base abstraites est que *chaque type dérivé* doit composer avec l'ensemble des membres abstraits et fournir une implémentation==. Pour illustrer ce problème, reprenons la hiérarchie des formes définie au [[Chapitre 6#Comprendre l'interface polymorphe|Chapitre 6]]. Supposons que vous ayez défini une nouvelle méthode abstraite dans la classe de base `Shape`, nommée `GetNumberOfPoints()`, qui permet aux types dérivés de renvoyer le nombre de points nécessaires au rendu de la forme.

```cs
namespace CustomInterfaces;

// La classe de base abstraite de la hiérarchie
abstract class Shape
{
	...
	
    // Chaque classe dérivée doit désormais prendre en charge cette méthode !
    public abstract byte GetNumberOfPoints();
}
```

De toute évidence, seule la classe `Hexagon` possède des points. Cependant, avec cette mise à jour, chaque classe dérivée (`Circle`, `Hexagon` et `ThreeDCircle`) doit désormais fournir une implémentation concrète de cette fonction, même si cela n'a pas de sens. Là encore, le type interface offre une solution. **Si vous définissez une interface représentant le comportement «posséder des points», vous pouvez simplement l'intégrer au type `Hexagon`, sans modifier les classes `Circle` et `ThreeDCircle`**.

>[!note]
>Les modifications apportées aux interfaces dans C# 8 sont probablement les changements les plus importants apportés à une fonctionnalité existante du langage dont je me souvienne. Comme décrit précédemment, les nouvelles capacités des interfaces les rapprochent considérablement des fonctionnalités des classes abstraites, avec la possibilité supplémentaire pour une classe d'implémenter plusieurs interfaces. Mon conseil : il faut faire preuve de prudence et de bon sens. Ce n'est pas parce qu'on peut faire quelque chose qu'on doit le faire.

# Définition d'interfaces personnalisées
Maintenant que vous comprenez mieux le rôle global des types d'interface, ==voyons un exemple de définition et d'implémentation d'interfaces personnalisées==. Copiez les fichiers *Shape.cs,* *Hexagon.cs,* *Circle.cs* et *ThreeDCircle.cs* de la solution *Shapes* créée au [[Chapitre 6#Comprendre l'interface polymorphe|Chapitre 6]]. Ensuite, renommez l'espace de noms qui définit vos types centrés sur les formes en `CustomInterfaces` (afin d'éviter d'avoir à importer les définitions d'espace de noms dans votre nouveau projet). Insérez ensuite un nouveau fichier nommé *IPointy.cs* dans votre projet.

Au niveau syntaxique, **une interface est définie à l'aide du mot-clé C# `interface`. Contrairement à une classe, les interfaces ne spécifient jamais de classe de base** (pas même `System.Object`; cependant, comme vous le verrez plus loin dans ce chapitre, ==une interface peut spécifier des interfaces de base==). **Avant C# 8.0, les membres d'une interface ne spécifiaient jamais de modificateur d'accès** (car tous les membres d'une interface sont implicitement `public` et `abstract`). **Nouveauté de C# 8.0 : il est désormais possible de définir des membres `private`, `internal`, `protected` et même `static`**. Nous y reviendrons plus tard. Pour commencer, voici un exemple d’interface personnalisée définie en C# :

```cs
namespace CustomInterfaces;

// Cette interface défini le comportement d'"avoir des points".
public interface IPointy
{
    // Implicitement public et asbstrait.
    byte GetNumberOfPoints();
}
```

**En C# 8, les interfaces ne peuvent pas définir de champs de données ni de constructeurs non statiques**. ==Par conséquent, la version suivante de `IPointy` entraînera diverses erreurs de compilation==:

```cs
// Aïe ! Des erreurs à profusion !
public interface IPointy
{
    // Erreur ! Les interfaces ne peuvent pas contenir de champs de données !
    public int numbOfPoints;

    // Erreur ! Les interfaces ne possèdent pas de constructeurs non statiques !
    public IPointy()
    {
        numbOfPoints = 0;
    }
}
```

Dans tous les cas, ==cette interface `IPointy` initiale définit une seule méthode==. Les interfaces peuvent également définir un nombre quelconque de *prototypes de propriétés*. Par exemple, nous pourrions mettre à jour l'interface `IPointy` pour utiliser une propriété en lecture-écriture (commentée) et une propriété en lecture seule. La propriété `Points` remplace la méthode `GetNumberOfPoints()`.

```cs
namespace CustomInterfaces;

// Le comportement de la propriété « pointy » en lecture seule.
public interface IPointy
{
    // Implicitement publique et abstraite.
    //byte GetNumberOfPoints();

    // Une propriété en lecture-écriture dans une interface ressemblerait à :
    //string PropName { get; set; }

    // tandis qu'une propriété en lecture seule 
    // dans une interface ressemblerait à :
    byte Points { get; }
}
```

>[!info] 
>Les types d'interface peuvent également contenir des définitions d'événements (voir [[Chapitre 12#Comprendre les événements C|Chapitre 12]]) et d'indexeurs (voir [[Chapitre 11#Comprendre les méthodes d'indexation|Chapitre 11]]).

**Les types d'interface sont assez inutiles en eux-mêmes, car on ne peut pas allouer des types d'interface comme on le ferait pour une classe ou une structure**.

```cs
// Aïe ! Allocation de types d'interface interdite.
IPointy p = new IPointy(); // Erreur de compilation !
```

Les interfaces n'apportent pas grand-chose tant qu'elles ne sont pas implémentées par une classe ou une structure. ==Ici, `IPointy` est une interface qui exprime le comportement consistant à «posséder des points»==. L'idée est simple : certaines classes de la hiérarchie des formes possèdent des points (comme `Hexagon`), tandis que d'autres (comme le `Circle`) n'en possèdent pas.

# Implémentation d'une interface

 **Lorsqu'une classe (ou structure) choisit d'étendre ses fonctionnalités en prenant en charge des interfaces, elle le fait à l'aide d'une liste séparée par des virgules dans la définition du type**. ==Notez que la classe de base directe doit être le premier élément après l'opérateur deux-points==. **Si votre type de classe hérite directement de `System.Object`, vous pouvez simplement lister l'interface (ou les interfaces) prise en charge par la classe, car le compilateur C# étendra vos types à partir de `System.Object` si vous ne spécifiez rien d'autre**. Par ailleurs, ==étant donné que les structures héritent toujours de `System.ValueType`== (voir [[Chapitre 4#Comprendre les types de valeur et les types de référence|Chapitre 4]]), ==il suffit de lister chaque interface directement après la définition de la structure==. Considérez les exemples suivants :

```cs
// Cette classe hérite de System.Object et
// implémente une seule interface.
public class Pencil : IPointy
{...}

// Cette classe hérite également de System.Object
// et implémente une seule interface.
public class SwitchBlade : object, IPointy
{...}

// Cette classe hérite d'une classe de base personnalisée
// et implémente une seule interface.
public class Fork : Utensil, IPointy
{...}

// Cette structure hérite implicitement de System.ValueType et
// implémente deux interfaces.
public struct PitchFork : ICloneable, IPointy
{...}
```

==Il faut bien comprendre que l'implémentation d'une interface est une opération binaire pour les éléments d'interface qui n'incluent pas d'implémentation par défaut==. Le type support ne peut pas choisir les membres qu'il implémentera. ==Étant donné que l'interface `IPointy` définit une seule propriété en lecture seule, cela ne représente pas une contrainte majeure==. Cependant, **si vous implémentez une interface qui définit dix membres** (comme l'interface `IDbConnection` présentée précédemment), **le type est alors responsable de détailler les dix membres abstraits**.

Pour cet exemple, insérez un nouveau type de classe nommé `Triangle` qui "est un" `Shape` et prend en charge l'interface `IPointy`. Notez que l'implémentation de la propriété `Points` en lecture seule (implémentée à l'aide de la syntaxe de membre avec expression) renvoie simplement le nombre correct de points (trois).

```cs
namespace CustomInterfaces;

// Nouvelle classe dérivée de Shape appellée Triangle.
class Triangle : Shape, IPointy
{
    public Triangle() { }

    public Triangle(string name)
        : base(name) { }

    public override void Draw()
    {
        Console.WriteLine($"Drawing {PetName} the Triangle");
    }

    // Implémentation de IPointy
    // public byte Points
    // {
    //    get {return 3;}
    // }
    public byte Points => 3;
}
```

Maintenant, mettez à jour votre type `Hexagon` existant pour qu'il prenne également en charge le type d'interface `IPointy`.

```cs
namespace CustomInterfaces;

// Hexagon implémente maintenant IPointy
class Hexagon : Shape, IPointy
{
    public Hexagon() { }

    public Hexagon(string name)
        : base(name) { }

    public override void Draw()
    {
        Console.WriteLine($"Drawing {PetName} the Hexagon");
    }

    // Implémentation de IPointy
    public byte Points => 6;
}
```

Pour résumer, le diagramme de classes Visual Studio présenté dans l'image suivante illustre les classes compatibles avec `IPointy`, utilisant la notation en sucette. ==Remarquez que `Circle` et `ThreeDCircle` n’implémentent pas `IPointy`, car ce comportement n’est pas pertinent pour ces classes==.

![[Figure 8.1.png|La hiérarchie des Shapes, désormais avec des interfaces]]

>[!note]-
>Pour afficher/masquer le nom des interfaces dans le concepteur de classe, faites un clic-droit sur l'icône de l'interface et sélectionner l'option Réduire ou Développer.

# Appel des membres d'interface au niveau objet

Maintenant que vous disposez de classes prenant en charge l'interface `IPointy`, la question suivante est de savoir **comment interagir avec cette nouvelle fonctionnalité**. ==La méthode la plus simple consiste à appeler directement les membres d'une interface== (à condition qu'ils ne soient pas implémentés explicitement; vous trouverez plus de détails dans la section [[#Implémentation explicite d'interface]]). Par exemple, considérez le code suivant :

```cs
using CustomInterfaces;

Console.Title = "Fun with Interfaces";
Console.WriteLine("**** Fun with Interfaces ****\n");

 // Appelle la propriété Points défini par IPointy.
Hexagon hex = new Hexagon();
Console.WriteLine("Points: {0}", hex.Points);
Console.ReadLine();
```

Cette approche fonctionne parfaitement dans ce cas, étant donné que vous comprenez que le type `Hexagon` a implémenté l'interface en question et possède donc une propriété `Points`. Cependant, **il arrive parfois que vous ne puissiez pas déterminer quelles interfaces sont prises en charge par un type donné**. Par exemple, ==supposons que vous ayez un tableau contenant 50 types compatibles avec `Shape`, dont seuls certains prennent en charge `IPointy`==. Évidemment, ==si vous tentez d'accéder à la propriété `Points` d'un type qui n'a pas implémenté `IPointy`, vous recevrez une erreur==. Alors, comment déterminer dynamiquement si une classe ou une structure prend en charge l'interface appropriée ?

**Une façon de déterminer à l'exécution si un type prend en charge une interface spécifique consiste à utiliser une conversion explicite**. ==Si le type ne prend pas en charge l'interface demandée, vous recevez une exception `InvalidCastException`==. Pour gérer cette possibilité, utilisez une gestion structurée des exceptions comme dans l'exemple suivant :

```cs
...
// Gérer une éventuelle exception InvalidCastException.
Circle c = new Circle("Lisa");
IPointy itfPt = null;
try
{
    itfPt = (IPointy)c;
    Console.WriteLine(itfPt.Points);
}
catch (InvalidCastException e)
{
    Console.WriteLine(e.Message);
}
Console.ReadLine();
}
```

Bien qu'il soit possible d'utiliser une logique `try`/`catch` et de croiser les doigts, **l'idéal serait de déterminer quelles interfaces sont prises en charge avant même d'appeler les membres de l'interface. Voyons deux façons de procéder**.

## Obtention de références d'interface : le mot-clé `as`

**Vous pouvez déterminer si un type donné prend en charge une interface en utilisant le mot-clé `as`**, introduit au [[Chapitre 6#Utilisation du mot clé C `as`|Chapitre 6]]. ==Si l'objet peut être traité comme l'interface spécifiée, une référence à cette interface vous est renvoyée. Sinon, une référence nulle vous est renvoyée. Par conséquent, assurez-vous de vérifier la valeur avant de poursuivre.==

```cs
Console.Title = "Fun with Interfaces";
Console.WriteLine("**** Fun with Interfaces ****\n");

...

// Peut on traité hex2 comme un IPointy ?
Hexagon hex2 = new Hexagon("Peter");
IPointy itfPt2 = hex2 as IPointy;
if (itfPt2 != null)
{
    Console.WriteLine($"Points: {itfPt2.Points}");
}
else
{
    Console.WriteLine("OOPS! Not Pointy...");
}

Console.ReadLine();
```

**Notez que lorsque vous utilisez le mot-clé `as`, vous n'avez pas besoin d'utiliser la logique `try/catch`**; ==si la référence n'est pas `null`, vous savez que vous appelez une référence d'interface valide==.

## Obtention de références d'interface : Le mot-clé `is` (MaJ C# 7.0)

**Vous pouvez également vérifier si une interface est implémentée à l'aide du mot-clé `is`** (également présenté au [[Chapitre 6#Utilisation du mot-clé `is` (Nouveauté C 7.0)|Chapitre 6]]). ==Si l'objet en question n'est pas compatible avec l'interface spécifiée, la valeur `false` est renvoyée==. **Si vous fournissez un nom de variable dans l'instruction, le type est affecté à la variable, ce qui évite d'effectuer la vérification de type et la conversion de type**. L'exemple précédent est mis à jour ici :

```cs
Console.Title = "Fun with Interfaces";
Console.WriteLine("**** Fun with Interfaces ****\n");

...

if (hex2 is IPointy itfPt3)
    Console.WriteLine($"Points: {itfPt3.Points}");
else
    Console.WriteLine("OOPS! Not Pointy...");

Console.ReadLine();
```

# Implémentations par défaut (Nouveauté C# 8.0)

**Comme mentionné précédemment, C# 8.0 a introduit la possibilité d'avoir une implémentation par défaut pour les méthodes et propriétés d'interface**. Ajoutez une nouvelle interface nommée `IRegularPointy` pour représenter un polygone de forme régulière. Le code est présenté ici.

```cs
namespace CustomInterfaces;

interface IRegularPointy : IPointy
{
    int SideLength { get; set; }
    int NumberOfSides { get; set; }
    int Perimeter => SideLength * NumberOfSides;
}
```

Ajoutez une nouvelle classe nommée *Square.cs* au projet, héritez de la classe de base `Shape` et implémentez l'interface `IRegularPointy`, comme suit :

```cs
namespace CustomInterfaces;

class Square : Shape, IRegularPointy
{
    public Square() { }

    public Square(string name)
        : base(name) { }

    // Draw() vient de la classe de base Shape
    public override void Draw()
    {
        Console.WriteLine($"Drawing {PetName} the Square");
    }

    // Ceci vient de l'interface IPointy
    public byte Points => 4;

    // Ceux-ci viennent de l'interface IRegularPointy
    public int SideLength { get; set; }
    public int NumberOfSides { get; set; }
    // Notez que la propiété Perimeter n'est pas implémentée.
}
```

==Nous avons ici, sans le vouloir, introduit le premier inconvénient de l'utilisation d'implémentations par défaut dans les interfaces==. **La propriété `Perimeter`, définie dans l'interface `IRegularPointy`, n'est pas définie dans la classe `Square`, la rendant inaccessible depuis une instance de `Square`**. Pour le constater, créez une nouvelle instance de la classe `Square` et affichez les valeurs correspondantes dans la console, comme ceci :

```cs
Console.Title = "Fun with Interfaces";
Console.WriteLine("**** Fun with Interfaces ****\n");

...

var sq = new Square("Boxy") { NumberOfSides = 4, SideLength = 4 };
sq.Draw();

// Ceci ne compilera pas
// Console.WriteLine($"{sq.PetName} has {sq.NumberOfSides} of length {sq.SideLength} and a Perimeter of {sq.Perimeter}");
```

**L'instance `Square` doit être explicitement convertie en interface `IRegularPointy`** (==puisque c'est là que se trouve l'implémentation==), **et la propriété `Perimeter` sera alors accessible**. Mettez à jour le code comme suit :

```cs
Console.WriteLine(
    $"{sq.PetName} has {sq.NumberOfSides} of length {sq.SideLength} and a Perimeter of {((IRegularPointy)sq).Perimeter}"
);
```

==Une solution pour contourner ce problème consiste à toujours coder selon l'interface d'un type==. Modifiez la définition de l'instance `Square` en `IRegularPointy` au lieu de `Square`, comme ceci :

```cs
IRegularPointy sq = new Square("Boxy") { NumberOfSides = 4, SideLength = 4 };
```

**Le problème avec cette approche (dans notre cas) est que la méthode `Draw()` et la propriété `PetName` ne sont pas définies dans l'interface, ce qui provoque des erreurs de compilation**.

Bien qu'il s'agisse d'un exemple simple, ==il illustre l'un des problèmes liés aux interfaces par défaut. Avant d'utiliser cette fonctionnalité dans votre code, assurez-vous d'évaluer les conséquences pour le code appelant de devoir connaître l'emplacement de l'implémentation==.

>[!note]- Les bénéfices de cette fonctionnalité (avec Gemini)
>Le livre se concentre sur une des contraintes majeures de l'implémentation par défaut des interfaces: le non héritages des implémentations.
>
> Cependant, les avantages sont les suivant:
> 
> 1. Évolution des APIs sans "Breaking Changes"
>	C'est le bénéfice n°1. Avant C# 8, si vous ajoutiez une méthode à une interface utilisée par des milliers de personnes, leur code ne compilait plus.
>
>	- **Bénéfice :** Vous pouvez ajouter une nouvelle fonctionnalité à une interface existante tout en fournissant un code "par défaut". Les anciennes classes continuent de fonctionner sans être modifiées.
>
>2. Éviter les "Extension Methods" polluantes
>	Auparavant, pour ajouter du comportement à une interface sans la modifier, on utilisait des méthodes d'extension.
>
>	- **Bénéfice :** Le code reste **à l'intérieur** de l'interface. Cela permet de garder une meilleure encapsulation et d'accéder à des membres protégés de l'interface si nécessaire.
>
>3. Support de l'interopérabilité (Android/iOS)
>	C# devait évoluer pour supporter les langages comme **Java** ou **Swift** (via Xamarin/MAUI) qui possédaient déjà cette fonctionnalité.
>	
>	- **Bénéfice :** Cela permet de mieux "mapper" les APIs mobiles natives vers le monde .NET.
>
>4. Code plus compact (Traits)
>	On peut utiliser les interfaces comme des "Traits" (un concept venant d'autres langages).
>
>	- **Bénéfice :** Vous pouvez "composer" le comportement d'une classe en lui faisant implémenter plusieurs interfaces qui ont chacune une logique par défaut.
>
>Les implémentations par défauts sont un outil de **maintenance d'architecture** plutôt qu'un outil de développement quotidien. Elles servent à faire survivre les interfaces sur le long terme sans forcer tous les utilisateurs à réécrire leur code.

# Constructeurs et membres statiques (Nouveauté C# 8.0)

**Une autre nouveauté des interfaces en C# 8.0 est la possibilité d'utiliser des constructeurs et des membres statiques, qui fonctionnent de la même manière que les membres statiques des définitions de classe, mais sont définis dans les interfaces**. Mettez à jour l'interface `IRegularPointy` avec un exemple de propriété statique et de constructeur statique.

```cs
namespace CustomInterfaces;

interface IRegularPointy : IPointy
{
    int SideLength { get; set; }
    int NumberOfSides { get; set; }
    int Perimeter => SideLength * NumberOfSides;

    // Membres statique sont aussi permis en C# 8
    static string ExampleProperty { get; set; }

    static IRegularPointy() => ExampleProperty = "Foo";
}
```

**Les constructeurs statiques doivent être sans paramètre et ne peuvent accéder qu'aux propriétés et méthodes statiques**. Pour accéder à la propriété statique de l'interface, ajoutez le code suivant aux instructions de niveau supérieur :

```cs
Console.WriteLine($"Example property: {IRegularPointy.ExampleProperty}");
IRegularPointy.ExampleProperty = "Updated";
Console.WriteLine($"Example property: {IRegularPointy.ExampleProperty}");
```

Notez que ==la propriété statique doit être appelée depuis l'interface et non depuis une variable d'instance==.

# Interfaces comme paramètres

**Les interfaces étant des types valides, vous pouvez créer des méthodes qui prennent des interfaces comme paramètres, comme illustré par la méthode `CloneMe()` présentée plus haut dans ce chapitre**. Dans cet exemple, supposons que vous ayez défini une autre interface nommée `IDraw3D`.

```cs
namespace CustomInterfaces;

// Modélise la capacité à rendre un texte en 3D époustouflante.
public interface IDraw3D
{
    void Draw3D();
}
```

Supposons maintenant que deux de vos trois formes (`ThreeDCircle` et `Hexagon`) aient été configurées pour prendre en charge ce nouveau comportement.

```cs
// ThreeDCircle supporte IDraw3D
class ThreeDCircle : Circle, IDraw3D
{

	...

    public void Draw3D() =>
        Console.WriteLine($"Drawing {PetName} the 3D circle in 3D");
}
```

```cs
// Hexagon implémente maintenant IPointy et IDraw3D
class Hexagon : Shape, IPointy, IDraw3D
{

	...
	
    public void Draw3D() =>
        Console.WriteLine($"Drawing {PetName} the Hexagon in 3D");
}
```

L'image suivante présente le diagramme de classes Visual Studio mis à jour.

![[Figure 8.2.png|La hiérarchie Shapes mise à jour]]

**Si vous définissez maintenant une méthode prenant une interface `IDraw3D` comme paramètre, vous pouvez lui passer n'importe quel objet implémentant `IDraw3D`**. ==Si vous tentez de lui passer un type ne prenant pas en charge l'interface requise, vous obtiendrez une erreur de compilation==. Prenons l'exemple de la méthode suivante définie dans votre fichier *Program.cs* :

```cs
// Je dessinerai tous ceux qui supporte IDraw3D.
static void DrawIn3D(IDraw3D itf3d)
{
    Console.WriteLine("-> Drawing IDraw3D compatible type");
    itf3d.Draw3D();
}
```

Vous pouvez désormais vérifier si un élément du tableau `Shape` prend en charge cette nouvelle interface et, si c'est le cas, le transmettre à la méthode `DrawIn3D()` pour traitement.

```cs
Console.Title = "Fun with Interfaces";
Console.WriteLine("**** Fun with Interfaces ****\n");

...
Shape[] myShapes =
{
    new Hexagon(),
    new Circle(),
    new Triangle("Joe"),
    new Circle("JoJo"),
};
for (int i = 0; i < myShapes.Length; i++)
{
	// Puis-je te dessiner en 3D ?
    if (myShapes[i] is IDraw3D s)
    {
        DrawIn3D(s);
    }
}

Console.ReadLine();
```

Voici le résultat de l'application mise à jour. Notez que seul l'objet `Hexagon` s'affiche en 3D, car les autres éléments du tableau `Shape` n'implémentent pas l'interface `IDraw3D`.

```
**** Fun with Interfaces ****

...
-> Drawing IDraw3D compatible type
Drawing noName the Hexagon in 3D
```

# Utilisation des interfaces comme valeurs de retour

**Les interfaces peuvent également servir de valeurs de retour pour les méthodes**. Par exemple, vous pouvez écrire une méthode qui prend un tableau d'objets `Shape` et renvoie une référence au premier élément compatible avec `IPointy`.

```cs
// Cette méthode renvoie le premier objet du
// tableau qui implémente l'interface IPointy.
static IPointy FindFirstPointyShape(Shape[] shapes)
{
    foreach (Shape s in shapes)
    {
        if (s is IPointy ip)
        {
            return ip;
        }
    }
    return null;
}
```

You could interact with this method as follows:

```cs
Console.Title = "Fun with Interfaces";
Console.WriteLine("**** Fun with Interfaces ****\n");

...

Shape[] myShapes =
{
    new Hexagon(),
    new Circle(),
    new Triangle("Joe"),
    new Circle("JoJo"),
};
...
// Récupérer le premier élément pointu.
IPointy firstPointyItem = FindFirstPointyShape(myShapes);

// Par sécurité, utilisation de l'opérateur conditionnel null.
Console.WriteLine($"The item has {firstPointyItem?.Points} points");
```

# Tableaux de types d'interface

**Rappelons qu'une même interface peut être implémentée par plusieurs types, même s'ils n'appartiennent pas à la même hiérarchie de classes et n'ont pas de classe parente commune autre que `System.Object`**. Ceci permet de réaliser des constructions de programmation puissantes. Par exemple, supposons que vous ayez développé trois nouveaux types de classes dans votre projet actuel : l'un modélisant des ustensiles de cuisine (via les classes `Knife` et `Fork`) et l'autre modélisant du matériel de jardinage (à la manière de `PitchFork`). Le code correspondant à ces classes est présenté ici, et le diagramme de classes mis à jour est illustré dans l'image suivante :

```cs
// Fork.cs
namespace CustomInterfaces;

class Fork : IPointy
{
    public byte Points => 4;
}
```

```cs
// PitchFork.cs
namespace CustomInterfaces;

class PitchFork : IPointy
{
    public byte Points => 3;
}
```

```cs
// Knife.cs
namespace CustomInterfaces;

class Knife : IPointy
{
    public byte Points => 1;
}
```

![[Figure 8.3.png|Rappelons que les interfaces peuvent être « branchées » sur n’importe quel type dans n’importe quelle partie d’une hiérarchie de classes.]]

Si vous avez défini les types `PitchFork`, `Fork` et `Knife`, vous pouvez maintenant définir un tableau d'objets compatibles avec `IPointy`. **Comme ces membres prennent tous en charge la même interface, vous pouvez parcourir le tableau et traiter chaque élément comme un objet compatible avec `IPointy`, quelle que soit la diversité globale des hiérarchies de classes**.

```cs
...

// Ce tableau ne peut contenir que des types qui
// implémentent l'interface IPointy.
IPointy[] myPointyObjects =
{
    new Hexagon(),
    new Knife(),
    new Triangle(),
    new Fork(),
    new PitchFork(),
};
foreach (IPointy i in myPointyObjects)
{
    Console.WriteLine($"Object has {i.Points} points.");
}

Console.ReadLine();
```

==Pour bien souligner l'importance de cet exemple, retenez ceci : **lorsqu'on a un tableau d'une interface donnée, ce tableau peut contenir n'importe quelle classe ou structure qui implémente cette interface**==.

# Implémentation d'interfaces avec Visual Studio ou Visual Studio Code

>[!tip] Cette fonctionnalité vient de *roslyn*, et est donc disponible pour *neovim*.
>Pour cela, il faut avoir configuré l'éditeur pour accéder aux "*code actions*" (dans mon cas: `leader => c => a`)

Bien que la programmation par interfaces soit une technique puissante, ==leur implémentation peut nécessiter une quantité importante de code==. **Les interfaces étant des ensembles nommés de membres abstraits, vous devez définir et implémenter chaque méthode d'interface pour chaque type prenant en charge ce comportement**. Par conséquent, ==si vous souhaitez implémenter une interface définissant cinq méthodes et trois propriétés, vous devez implémenter les huit membres (sinon, vous obtiendrez des erreurs de compilation)==.

Comme vous pouvez l'espérer, ==Visual Studio et Visual Studio Code proposent divers outils facilitant l'implémentation des interfaces==. À titre de test simple, insérez une classe finale nommée `PointyTestClass` dans votre projet actuel. Lorsque vous ajoutez une interface telle que `IPointy` (ou toute autre interface) à un type de classe, ==vous avez peut-être remarqué qu'une fois le nom de l'interface saisi== (ou lorsque vous placez le curseur de la souris dessus dans la fenêtre de code), ==Visual Studio et Visual Studio Code affichent une ampoule, accessible également avec la combinaison de touches Ctrl+point (.)==. En cliquant sur cette ampoule, une liste déroulante s'affiche, vous permettant d'implémenter l'interface (voir les deux images suivantes).

![[Figure 8.4.png|Implémentation automatique d'interfaces à l'aide de Visual Studio Code]]

![[Figure 8.5.png|Implémentation automatique des interfaces à l'aide de Visual Studio]]

**Vous remarquerez que deux options vous sont proposées. La seconde (implémentation explicite de l'interface) sera examinée dans la section suivante**. Pour l'instant, sélectionnez la première option. Visual Studio/Visual Studio Code a alors généré un code de base que vous pouvez modifier. (==Notez que l'implémentation par défaut lève une exception `System.NotImplementedException`, que vous pouvez bien sûr supprimer.==)

```cs
namespace CustomInterfaces;

class PointyTestClass : IPointy
{
    public byte Points => throw new NotImplementedException();
}
```

>[!note] Refactorisation en interface
>Les IDE utilisant *roslyn* prennent également en charge la refactorisation par extraction d'interface, accessible via l'option Extraire l'interface du menu *code actions*. Cela vous permet d'extraire une nouvelle définition d'interface à partir d'une définition de classe existante. Par exemple, vous pourriez être en train d'écrire une classe lorsque vous réalisez que vous pouvez généraliser son comportement dans une interface (et ainsi ouvrir la possibilité d'implémentations alternatives).

# Implémentation explicite d'interfaces

Comme indiqué précédemment dans ce chapitre, **une classe ou une structure peut implémenter un nombre quelconque d'interfaces**. De ce fait, ==il est toujours possible d'implémenter des interfaces contenant des membres identiques et, donc, de rencontrer un conflit de noms==. Pour illustrer différentes manières de résoudre ce problème, créez un nouveau projet d'application console nommé *InterfaceNameClash*. Concevez ensuite trois interfaces représentant différents emplacements où un type implémentant ces interfaces pourrait afficher sa sortie.

```cs
namespace InterfaceNameClash;

// Dessiner une image sur un formulaire.
public interface IDrawToForm
{
    void Draw();
}
```

```cs
namespace InterfaceNameClash;

// Dessiner dans la mémoire tampon.
public interface IDrawToMemory
{
  void Draw();
}
```

```cs
namespace InterfaceNameClash;

// Rendu vers l'imprimante.
public interface IDrawToPrinter
{
    void Draw();
}
```

**Notez que chaque interface définit une méthode nommée `Draw()`, avec la même signature**. ==Si vous souhaitez maintenant prendre en charge chacune de ces interfaces sur un seul type de classe nommé `Octagon`, le compilateur autorisera la définition suivante :==

```cs
namespace InterfaceNameClash;

class Octagon : IDrawToForm, IDrawToMemory, IDrawToPrinter
{
    public void Draw()
    {
        // Logique de dessin partagée.
        Console.WriteLine("Drawing the Octagon...");
    }
}
```

Bien que le code compile sans problème, un souci subsiste. En effet, **fournir une seule implémentation de la méthode `Draw()` ne permet pas d'effectuer des actions spécifiques selon l'interface obtenue à partir d'un objet `Octagon`**. ==Par exemple, le code suivant appellera la même méthode `Draw()`, quelle que soit l'interface obtenue :==

```cs
using InterfaceNameClash;

Console.Title = "Fun with Interface Name Clashes";
Console.WriteLine("**** Fun with Interface Name Clashes ****\n");

Octagon oct = new Octagon();

// Ces deux appels "convoque" la
// même méthode Draw() !

// Notation abrégée si vous n'avez pas besoin
// de la variable d'interface pour une utilisation ultérieure.
((IDrawToPrinter)oct).Draw();

// On pourrait également utiliser le mot-clé « is ».
if (oct is IDrawToMemory dtm)
{
    dtm.Draw();
}
Console.ReadLine();
```

Il est clair que le code nécessaire pour afficher une image dans une fenêtre est très différent de celui nécessaire pour l'afficher sur une imprimante réseau ou dans une zone de mémoire. **Lorsque vous implémentez plusieurs interfaces ayant des membres identiques, vous pouvez résoudre ce type de conflit de noms en utilisant une syntaxe d'implémentation d'interface explicite**. Prenons l'exemple de la mise à jour suivante du type `Octagon` :

```cs
namespace InterfaceNameClash;

class Octagon : IDrawToForm, IDrawToMemory, IDrawToPrinter
{
    // Associer explicitement les implémentations de Draw()
    // à une interface donnée.
    void IDrawToForm.Draw()
    {
        Console.WriteLine("Drawing to form...");
    }

    void IDrawToMemory.Draw()
    {
        Console.WriteLine("Drawing to memory...");
    }

    void IDrawToPrinter.Draw()
    {
        Console.WriteLine("Drawing to a printer...");
    }
}
```

Comme vous pouvez le constater, **lors de l'implémentation explicite d'un membre d'interface, le modèle général se réduit à ceci** :

```cs
typeRetour NomInterface.NomMéthode(params){}
```

Notez que lorsque vous utilisez cette syntaxe, ==vous ne devez pas spécifier de modificateur d'accès ; les membres explicitement implémentés sont automatiquement `private`==. Par exemple, la syntaxe suivante est incorrecte :

```cs
// Erreur! Pas de modificateur d'accès!
public void IDrawToForm.Draw()
{
	Console.WriteLine("Drawing to form...");
}
```

Comme les membres implémentés explicitement sont toujours implicitement privés, ils ne sont plus accessibles au niveau de l'objet. En effet, ==si vous appliquez l'opérateur point à un type `Octagon`, vous constaterez qu'IntelliSense n'affiche aucun membre `Draw()`==. Comme prévu, **vous devez utiliser un casting explicite pour accéder à la fonctionnalité requise**. Le code précédent, dans les instructions de niveau supérieur, utilise déjà un casting explicite et fonctionne donc avec les interfaces explicites.

```cs
using InterfaceNameClash;

Console.Title = "Fun with Interface Name Clashes";
Console.WriteLine("**** Fun with Interface Name Clashes ****\n");

Octagon oct = new Octagon();

// Nous devons maintenant utiliser le casting
// pour accéder aux membres de Draw().
IDrawToForm itfForm = (IDrawToForm)oct;
itfForm.Draw();

// Notation abrégée si vous n'en avez pas besoin de
// la variable d'interface pour une utilisation ultérieure.
((IDrawToForm)oct).Draw();

// On pourrait aussi utiliser le mot-clé « is ».
if (oct is IDrawToMemory dtm)
{
    dtm.Draw();
}
Console.ReadLine();
```

**Bien que cette syntaxe soit très utile pour résoudre les conflits de noms, vous pouvez utiliser une implémentation d'interface explicite afin de masquer les membres plus « avancés » au niveau de l'objet**. Ainsi, ==lorsque l'utilisateur de l'objet applique l'opérateur point, il ne verra qu'un sous-ensemble des fonctionnalités globales du type==. Toutefois, ceux qui ont besoin des comportements plus avancés peuvent extraire l'interface souhaitée via un conversion explicite.

### Explication plus poussée pour l'exemple de code précédent (Avec l'aide de Gemini)

**comment le code peut-il appeler une méthode `private` via une interface** ?

#### Le concept de `private` est relatif

Quand on dit que l'implémentation explicite est **privée**, cela signifie qu'elle est privée **du point de vue de la classe** `Octagon`.

- Si vous tapez `oct.`, votre IDE ne vous proposera pas `Draw()`.
- Le compilateur interdit l'accès direct pour éviter les collisions de noms.

Cependant, du point de vue du **Runtime (CLR)**, cette méthode est marquée d'une manière spéciale : elle est **"Publiquement accessible via l'interface"**.

#### Toujours un `Octagon`

- L'objet en mémoire ne change jamais. Il reste un `Octagon` avec toute sa structure.
- Le **Casting** (`(IDrawToForm)oct`) ne transforme pas l'objet. Il change simplement la **table de méthodes** (vtable) que le programme consulte.

#### Comment le Runtime fait-il le lien ?

C'est là que la magie du .NET opère via une structure appelée la **Interface Map** :

1. Chaque classe possède une table qui liste : "Pour telle interface, voici quelle méthode de ma classe il faut exécuter".
2. Quand vous faites `itfForm.Draw()`, le runtime dit : *"Je sais que cet objet est un `Octagon`. Je regarde dans sa carte d'interfaces pour `IDrawToForm`. Ah ! La méthode `Draw()` correspond à ce bloc de code précis (l'implémentation explicite)."*
3. Le Runtime ignore alors les modificateurs d'accès (`public`/`private`) car il suit le contrat de l'interface qui, lui, est **public**.

#### Exemple de vérification (C# 14 / .NET 10)

Vous pouvez vérifier que l'objet reste un `Octagon` même après le cast :

```cs
IDrawToForm itfForm = (IDrawToForm)oct;

// Le type reste Octagon !
Console.WriteLine(itfForm.GetType().Name); // Affiche "Octagon"

// On peut toujours revenir en arrière (C# 14 syntax)
if (itfForm is Octagon backToOct)
{
    Console.WriteLine("C'est toujours mon polygone à 8 côtés !");
}
```

#### Ce qu'il faut retenir

- L'implémentation explicite est **privée pour la classe** mais **publique pour l'interface**.
- Le **Runtime** utilise une table de correspondance interne pour sauter directement au bon code.
- L'objet ne change pas de nature, il change juste de **rôle**.

# Conception de hiérarchies d'interfaces

**Les interfaces peuvent être organisées selon une hiérarchie**. ***==À l'instar d'une hiérarchie de classes, lorsqu'une interface étend une interface existante, elle hérite des membres abstraits définis par le ou les parents==***. **Avant C# 8, les interfaces dérivées n'héritaient jamais d'implémentation concrète**. ==Elles se contentaient d'étendre leur propre définition avec des membres abstraits supplémentaires==. **En C# 8, les interfaces dérivées héritent des implémentations par défaut, en plus d'étendre leur définition et d'ajouter potentiellement de nouvelles implémentations par défaut**.

**Les hiérarchies d'interfaces s'avèrent utiles pour étendre les fonctionnalités d'une interface existante sans impacter le code existant**. À titre d'exemple, créez un nouveau projet d'application console nommé *InterfaceHierarchy*. Concevons maintenant un nouvel ensemble d'interfaces axées sur le rendu, dont `IDrawable` est la racine.

```cs
namespace InterfaceHierarchy;

interface IDrawable
{
    void Draw();
}
```

Étant donné que `IDrawable` définit un comportement de dessin de base, vous pouvez maintenant créer une interface dérivée qui étend cette interface en lui permettant de réaliser des rendus dans des formats modifiés. Voici un exemple :

```cs
namespace InterfaceHierarchy;

public interface IAdvancedDraw : IDrawable
{
    void DrawInBoundingBox(int top, int left, int bottom, int right);

    void DrawUpsideDown();
}
```

Compte tenu de cette conception, ==si une classe devait implémenter `IAdvancedDraw`, elle serait désormais tenue d'implémenter chaque membre défini en amont de la chaîne d'héritage== (en particulier, les méthodes `Draw()`, `DrawInBoundingBox()` et `DrawUpsideDown()`).

```cs
namespace InterfaceHierarchy;

public class BitmapImage : IAdvancedDraw
{
    public void Draw()
    {
        Console.WriteLine("Drawing...");
    }

    public void DrawInBoundingBox(int top, int left, int bottom, int right)
    {
        Console.WriteLine("Drawing in a box...");
    }

    public void DrawUpsideDown()
    {
        Console.WriteLine("Drawing upside down!");
    }
}
```

**Désormais, lorsque vous utilisez `BitmapImage`, vous pouvez appeler chaque méthode au niveau de l'objet (car elles sont toutes `public`), ainsi qu'extraire explicitement une référence à chaque interface prise en charge via un cast.

```cs
using InterfaceHierarchy;

Console.Title = "Simple Interface Hierarchy";
Console.WriteLine("**** Simple Interface Hierarchy ****\n)");

// Appel au niveau de l'objet.
BitmapImage myBitmap = new BitmapImage();
myBitmap.Draw();
myBitmap.DrawInBoundingBox(10, 10, 100, 150);
myBitmap.DrawUpsideDown();

// Obtenir explicitement IAdvancedDraw.
if (myBitmap is IAdvancedDraw iAdvDraw)
    iAdvDraw.DrawUpsideDown();

Console.ReadLine();
```

## Hiérarchies d'interfaces avec implémentations par défaut (Nouveauté C# 8.0) 

**Lorsque les hiérarchies d'interfaces incluent également des implémentations par défaut, les interfaces en aval peuvent choisir de reprendre l'implémentation de l'interface de base ou de créer une nouvelle implémentation par défaut**. Mettez à jour l'interface `IDrawable` comme suit :

```cs
namespace InterfaceHierarchy;

public interface IDrawable
{
    void Draw();
    int TimeToDraw() => 5;
}
```

Ensuite, mettez à jour les instructions de niveau supérieur comme suit :

```cs
using InterfaceHierarchy;

Console.Title = "Simple Interface Hierarchy";
Console.WriteLine("**** Simple Interface Hierarchy ****\n)");

...

if (myBitmap is IAdvancedDraw iAdvDraw)
{
    iAdvDraw.DrawUpsideDown();
    Console.WriteLine($"Time to draw: {iAdvDraw.TimeToDraw()}");
}

Console.ReadLine();
```

**Non seulement ce code compile, mais il renvoie la valeur $5$ pour la méthode `TimeToDraw()`**. ==Ceci est dû au fait que les implémentations par défaut sont automatiquement héritées des interfaces descendantes==. **La conversion de `BitMapImage` vers l'interface `IAdvancedDraw` permet d'accéder à la méthode `TimeToDraw()`, même si l'instance `BitMapImage` n'a pas accès à l'implémentation par défaut**. Pour le vérifier, saisissez le code suivant et observez l'erreur de compilation :

```cs
// Ceci ne compilera pas
myBitmap.TimeToDraw();
```

==Si une interface en aval souhaite fournir sa propre implémentation par défaut, elle doit masquer l'implémentation en amont==. Par exemple, si la méthode `IAdvancedDraw` `TimeToDraw()` prend $15$ unités pour dessiner, mettez à jour l'interface avec la définition suivante :

```cs
namespace InterfaceHierarchy;

public interface IAdvancedDraw : IDrawable
{
    void DrawInBoundingBox(int top, int left, int bottom, int right);
    void DrawUpsideDown();
    new int TimeToDraw() => 15;
}
```

Bien entendu, la classe `BitMapImage` peut également implémenter la méthode `TimeToDraw()`. Contrairement à la méthode `TimeToDraw()` de `IAdvancedDraw`, la classe n'a qu'à ***implémenter*** la méthode, sans la masquer.

```cs
namespace InterfaceHierarchy;

public class BitmapImage : IAdvancedDraw
{
	...
	
    public int TimeToDraw() => 12;
}
```

**Lors de la conversion de l'instance `BitmapImage` vers l'interface `IAdvancedDraw` ou `IDrawable`, la méthode sur l'instance est toujours exécutée**. Ajoutez ce code aux instructions de niveau supérieur :

```cs
...
//Appelle toujours la méthode sur l'instance :
Console.WriteLine("**** Calling Implemented TimeToDraw ****\n)");
Console.WriteLine($"Time to draw: {myBitmap.TimeToDraw()}");
Console.WriteLine($"Time to draw: {((IDrawable)myBitmap).TimeToDraw()}");
Console.WriteLine($"Time to draw: {((IAdvancedDraw)myBitmap).TimeToDraw()}");
```

Voici les résultats :

```
...
**** Calling Implemented TimeToDraw ****

Time to draw: 12
Time to draw: 12
Time to draw: 12
```

> Pour garder le comportement où les interfaces gardes leurs propres logique:
> 1. Ne pas implémenter la méthode dans la classe.
> 2. L'implémentation explicite dans la classe (comme pour l'exemple des conflits de nom de méthode)

## Héritage multiple avec les types d'interface

**Contrairement aux classes, une interface peut étendre plusieurs interfaces de base, ce qui vous permet de concevoir des abstractions puissantes et flexibles**. Créez un nouveau projet d'application console nommé *MyInterfaceHierarchy.* Voici un autre ensemble d'interfaces modélisant diverses abstractions de rendu et de forme. Notez que l'interface `IShape` étend à la fois `IDrawable` et `IPrintable`.

```cs
//IDrawable.cs
namespace MyInterfaceHierarchy;

// L'héritage multiple pour les types d'interface 
// est tout à fait acceptable.
interface IDrawable
{
    void Draw();
}
```

```cs
//IPrintable.cs
namespace MyInterfaceHierarchy;

interface IPrintable
{
    void Print();

    void Draw(); // <-- Attention, risque de conflit de noms !
}
```

```cs
//IShape.cs
namespace MyInterfaceHierarchy;

// Héritage multiple d'interfaces. OK !
interface IShape : IDrawable, IPrintable
{
    int GetNumberOfSides();
}
```

L'image suivante illustre la hiérarchie d'interface actuelle.

![[Figure 8.6.png|Contrairement aux classes, les interfaces peuvent étendre plusieurs types d'interface.]]

À ce stade, la question à un million de dollars est : « ==Si vous avez une classe prenant en charge `IShape`, combien de méthodes devra-t-elle implémenter ?== » La réponse : cela dépend. **Si vous souhaitez fournir une implémentation simple de la méthode `Draw()`, vous n’aurez besoin que de trois membres, comme illustré dans le type `Rectangle` suivant** :

```cs
namespace MyInterfaceHierarchy;

class Rectangle : IShape
{
    public int GetNumberOfSides() => 4;

    public void Draw() => Console.WriteLine("Drawing...");

    public void Print() => Console.WriteLine("Printing...");
}
```

**Si vous préférez des implémentations spécifiques pour chaque méthode `Draw()`** (ce qui, dans ce cas, serait plus logique), **vous pouvez résoudre le conflit de noms en utilisant une implémentation d'interface explicite, comme illustré dans le type `Square` suivant** :

```cs
namespace MyInterfaceHierarchy;

class Square : IShape
{
    // Utilisation d'une implémentation explicite
    // pour gérer les conflits de noms de membres.
    void IPrintable.Draw()
    {
        // Dessine pour l'impimante...
    }

    void IDrawable.Draw()
    {
        // Dessine à l'écran...
    }

    public void Print()
    {
        // Affiche...
    }

    public int GetNumberOfSides() => 4;
}
```

Idéalement, à ce stade, vous devriez être plus à l'aise avec la définition et l'implémentation d'interfaces personnalisées en C#. Pour être honnête, la programmation par interfaces peut prendre un certain temps à maîtriser. Si vous avez encore quelques doutes, c'est tout à fait normal. Sachez toutefois que **les interfaces sont un aspect fondamental du framework .NET Core**. ==Quel que soit le type d'application que vous développez== (applications web, interfaces graphiques de bureau, bibliothèques d'accès aux données, etc.), ==l'utilisation des interfaces fera partie intégrante du processus==. En résumé, retenez que **les interfaces peuvent s'avérer extrêmement utiles dans les cas suivants** :

- **Vous disposez d'une hiérarchie unique où seul un sous-ensemble des types dérivés prend en charge un comportement commun**.
- **Vous devez modéliser un comportement commun présent dans plusieurs hiérarchies, sans classe parente commune autre que `System.Object`.**

Maintenant que vous avez approfondi la création et l'implémentation d'interfaces personnalisées, ==le reste de ce chapitre examine plusieurs interfaces prédéfinies contenues dans les bibliothèques de classes de base .NET Core==. Comme vous le verrez, ==vous pouvez implémenter des interfaces .NET Core standard sur vos types personnalisés afin de garantir leur intégration transparente au framework==.

# Les interfaces `IEnumerable` et `IEnumerator`

Pour commencer l’étude du processus d’implémentation des interfaces .NET Core existantes, examinons d’abord le rôle de `IEnumerable` et `IEnumerator`. **Rappelons que C# prend en charge le mot-clé `foreach` qui permet d’itérer sur le contenu de n’importe quel tableau**.

```cs
// Parcourir un tableau d'éléments.
int[] myArrayOfInts = {10, 20, 30, 40};

foreach(int i in myArrayOfInts)
{
	Console.WriteLine(i);
}
```

Bien qu'il puisse sembler que seuls les types tableaux puissent utiliser cette construction, en réalité, **tout type prenant en charge une méthode nommée `GetEnumerator()` peut être évalué par la boucle `foreach`**. Pour illustrer cela, commencez par créer un nouveau projet d'application console nommé *CustomEnumerator*. Ensuite, copiez les fichiers *Car.cs* et *Radio.cs* définis dans l'exemple *SimpleException* du [[Chapitre 7#L'exemple le plus simple possible|Chapitre 7]] dans le nouveau projet. Assurez-vous de mettre à jour les espaces de noms des classes en `CustomEnumerator`.

Insérez maintenant une nouvelle classe nommée `Garage` qui stocke un ensemble d'objets `Car` dans un `System.Array`.

```cs
using System.Collections;

namespace CustomEnumerator;

// Garage contient un enseble d'objet Car
public class Garage
{
    private Car[] carArray = new Car[4];

    // Remplir avec quelques objets Car au démarrage.
    public Garage()
    {
        carArray[0] = new Car("Rusty", 30);
        carArray[1] = new Car("Clunker", 55);
        carArray[2] = new Car("Zippy", 30);
        carArray[3] = new Car("Fred", 30);
    }
}
```

Idéalement, ==il serait pratique de parcourir les sous-éléments de l'objet `Garage` à l'aide de la boucle `foreach`, comme pour un tableau de valeurs de données==. Mettez à jour le fichier *Program.cs* comme suit :

```cs
using System.Collections;
using CustomEnumerator;

// Cela semble résonable ...
Console.Title = "Fun with IEnumerable / IEnumerator";
Console.WriteLine("**** Fun with IEnumerable / IEnumerator ****\n");

Garage carLot = new Garage();

// Remettre chaque voiture de la collection ?
foreach (Car c in carLot)
{
    Console.WriteLine("{0} is going {1} MPH", c.PetName, c.CurrentSpeed);
}
Console.ReadLine();
```

Malheureusement, **le compilateur vous informe que la classe `Garage` n'implémente pas de méthode nommée `GetEnumerator()`**. ==Cette méthode est formalisée par l'interface `IEnumerable`, que l'on trouve dans l'espace de noms `System.Collections`==.

>[!note]
>Au [[Chapitre 10#L'espace de noms `System.Collections.Generic`|Chapitre 10]], vous découvrirez le rôle des génériques et l'espace de noms `System.Collections.Generic`. Comme vous le verrez, cet espace de noms contient des versions génériques de `IEnumerable`/`IEnumerator` qui offrent une méthode plus sûre pour parcourir les éléments.

**Les classes ou structures qui prennent en charge ce comportement indiquent qu'elles peuvent exposer les éléments contenus à l'appelant** (dans cet exemple, le mot-clé `foreach` lui-même). Voici la définition de cette interface standard :

```cs
// Cette interface informe l'appelant
// que les éléments de l'objet peuvent être énumérés.
public interface IEnumerable
{ 
	IEnumerator GetEnumerator();
}
```

Comme vous pouvez le constater, **la méthode `GetEnumerator()` renvoie une référence à une autre interface nommée `System.Collections.IEnumerator`**. ==Cette interface fournit l'infrastructure permettant à l'appelant de parcourir les objets internes contenus dans le conteneur compatible `IEnumerable`==.

```cs
// Cette interface permet à l'appelant d'
// obtenir les éléments d'un conteneur.
public interface IEnumerator
{ 
	bool MoveNext (); // Avance la position interne du curseur.
	object Current { get; } // Obtient l'élément courant (propriété en lecture seule).
	void Reset (); // Réinitialise le curseur avant le premier élément.
}
```

Si vous souhaitez mettre à jour le type `Garage` pour prendre en charge ces interfaces, vous pouvez procéder de manière plus longue et implémenter chaque méthode manuellement. ==Bien que vous soyez libre de fournir des versions personnalisées de `GetEnumerator()`, `MoveNext()`, `Current` et `Reset()`, il existe une solution plus simple==. **Étant donné que le type `System.Array`** (ainsi que de nombreuses autres classes de collections) **implémente déjà `IEnumerable` et `IEnumerator`, vous pouvez simplement déléguer la requête à `System.Array` comme suit** (notez que vous devrez importer l'espace de noms `System.Collections` dans votre fichier de code) :

>[!info] 
>`IEnumearator` accepte les données nullable, même si le contrat de l'interface montre le contraire !
>
>![Video YouTube Microsoft](https://youtu.be/xKr96nIyCFM?t=1758)
>

```cs
using System.Collections;

namespace CustomEnumerator;

// Garage contient un enseble d'objet Car
public class Garage : IEnumerable
{
    // System.Array implémente déjà IEnumerator! 
    private Car[] carArray = new Car[4];

    // Remplir avec quelques objets Car au démarrage.
    public Garage()
    {
        carArray[0] = new Car("Rusty", 30);
        carArray[1] = new Car("Clunker", 55);
        carArray[2] = new Car("Zippy", 30);
        carArray[3] = new Car("Fred", 30);
    }

    // Retourne l'IEnumerator de l'objet tableau.
    public IEnumerator GetEnumerator() => carArray.GetEnumerator();

}
```

Après avoir mis à jour votre type `Garage`, ==vous pouvez l'utiliser sans risque dans la boucle `foreach` de C#==. De plus, ==étant donné que la méthode `GetEnumerator()` est publique, l'utilisateur de l'objet peut également interagir avec le type `IEnumerator`==.

```cs
// Utilisation manuelle de IEnumerator.
// Ne fonctionne pas avec une implémentation d'interface explicite.
IEnumerator i = carLot.GetEnumerator();
i.MoveNext();
Car myCar = (Car)i.Current;
Console.WriteLine("{0} is going {1} MPH", myCar?.PetName, myCar?.CurrentSpeed);
```

Ce faisant, l’utilisateur occasionnel de l’objet ne trouvera pas la méthode `GetEnumerator()` de `Garage`, tandis que la construction `foreach` obtiendra l’interface en arrière-plan lorsque cela sera nécessaire.

## Création de méthodes d'itération avec le mot-clé `yield`

==Il existe une autre façon de créer des types compatibles avec la boucle `foreach` via des itérateurs==. En clair, **un itérateur est un membre qui spécifie comment les éléments internes d'un conteneur doivent être retournés lors de leur traitement par `foreach`**. Pour illustrer cela, créez un nouveau projet d'application console nommé *CustomEnumeratorWithYield* et insérez les types `Car`, `Radio` et `Garage` de l'exemple précédent (en renommant vos définitions d'espace de noms pour correspondre au projet actuel). Ensuite, modifiez le type `Garage` existant comme suit :

```cs
using System.Collections;

namespace CustomEnumeratorWithYield;

// Garage contient un enseble d'objet Car
public class Garage : IEnumerable
{
	// Méthode itératrice
    public IEnumerator GetEnumerator()
    {
        foreach (Car c in carArray)
        {
            yield return c;
        }
    }
}
```

**Notez que cette implémentation de `GetEnumerator()` itère sur les sous-éléments à l'aide d'une boucle `foreach` interne et renvoie chaque `Car` à l'appelant en utilisant la syntaxe `yield return`**. ==Le mot-clé `yield` est utilisé pour spécifier la ou les valeurs à renvoyer à la boucle `foreach` de l'appelant==. **Lorsque l'instruction `yield return` est atteinte, l'emplacement actuel dans le conteneur est enregistré et l'exécution reprend à partir de cet emplacement lors du prochain appel de l'itérateur**.

==Les méthodes d'itération ne sont pas tenues d'utiliser le mot-clé `foreach` pour renvoyer leur contenu==. Il est également possible de définir cette méthode d'itération comme suit :

```cs
public IEnumerator GetEnumerator()
{
	yield return carArray[0];
	yield return carArray[1];
	yield return carArray[2];
	yield return carArray[3];
}
```

Dans cette implémentation, ==notez que la méthode `GetEnumerator()` renvoie explicitement une nouvelle valeur à l'appelant à chaque itération==. Dans cet exemple, cela n'a guère de sens, car si vous ajoutiez d'autres objets à la variable membre `carArray`, votre méthode `GetEnumerator()` serait alors désynchronisée. Néanmoins, ==cette syntaxe peut s'avérer utile lorsque vous souhaitez renvoyer des données locales depuis une méthode pour un traitement par la syntaxe `foreach`.==

### Clauses de garde avec fonctions locales (Nouveauté C# 7.0)

**Aucun code de la méthode `GetEnumerator()` n'est exécuté avant le premier parcours des éléments** (ou l'accès à un élément quelconque). ==Cela signifie que si une exception survient avant l'instruction `yield`, elle ne sera pas levée lors du premier appel à la méthode, mais seulement lors du premier appel à `MoveNext()`==.

Pour tester cela, mettez à jour la méthode `GetEnumerator` comme suit :

```cs
public IEnumerator GetEnumerator()
{
    //Cette exception ne sera levée que lorsque MoveNext() sera appelé
    throw new Exception("This won't get called");
    foreach (Car c in carArray)
    {
        yield return c;
    }
}
```

Si vous appeliez la fonction de cette manière sans rien faire d'autre, l'exception ne serait jamais levée :

```cs
using System.Collections;
using CustomEnumeratorWithYield;

Console.Title = "Fun with the Yield Keyword";
Console.WriteLine("**** Fun with the Yield Keyword ****\n");

Garage carLot = new Garage();
IEnumerator carEnumerator = carLot.GetEnumerator();

Console.ReadLine();
```

**Ce n'est qu'à l'appel de `MoveNext()` que le code s'exécutera et que l'exception sera levée**. Selon les besoins de votre programme, ==cela peut parfaitement convenir. Mais ce n'est pas toujours le cas==. Votre méthode `GetEnumerator` **peut contenir une clause de garde qui doit s'exécuter lors de son premier appel**. Par exemple, supposons que la liste soit extraite d'une base de données. ==Vous pourriez vouloir vérifier que la connexion à la base de données peut être ouverte au moment de l'appel de la méthode, et non lors de l'itération sur la liste. Ou encore, vous pourriez vouloir vérifier la validité des paramètres d'entrée de la méthode `Iterator`== (traitée ci-après).

Rappelez-vous, comme vu au [[Chapitre 4#Comprendre les fonctions locales (Nouveauté C 7.0, MaJ C 9.0)|Chapitre 4]], la fonctionnalité des fonctions locales de C# 7 ; ==les fonctions locales sont des fonctions privées à l'intérieur d'autres fonctions==. **En déplaçant le `yield return` dans une fonction locale renvoyée par le corps principal de la méthode, le code des instructions de niveau supérieur** (avant le renvoi de la fonction locale) **est exécuté immédiatement. La fonction locale est exécutée lors de l'appel à `MoveNext()`**.

Mettez à jour la méthode comme suit :

```cs
public IEnumerator GetEnumerator()
{
    // Cette exception sera levée immédiatement
    throw new Exception("Cette fonction sera appelée");
    
    return ActualImplementation();
    
    // Fonction locale et implémentation réelle de IEnumerator
    IEnumerator ActualImplementation()
    {
        foreach (Car c in carArray)
        {
            yield return c;
        }
    }
}
```

Testez cela en mettant à jour le code appelant comme suit :

```cs
using System.Collections;
using CustomEnumeratorWithYield;

Console.Title = "Fun with the Yield Keyword";
Console.WriteLine("**** Fun with the Yield Keyword ****\n");

Garage carLot = new Garage();
try
{
    // Erreur à ce moment
    var carEnumerator = carLot.GetEnumerator();
}
catch (Exception)
{
    Console.WriteLine($"Exception occurred on GetEnumerator");
}
Console.ReadLine();
```

==Suite à la mise à jour de la méthode `GetEnumerator()`, l'exception est levée immédiatement et non plus lors de l'appel à `MoveNext()`.==

## Créer d'un itérateur nommé

Il est également intéressant de noter que **le mot-clé `yield` peut techniquement être utilisé dans n'importe quelle méthode, quel que soit son nom**. ==Ces méthodes== (appelées techniquement *itérateurs nommés*) ==sont également uniques en ce qu'elles peuvent accepter un nombre quelconque d'arguments==. Lors de la création d'un itérateur nommé, **sachez que la méthode renverra l'interface `IEnumerable`, et non le type compatible `IEnumerator` attendu**. À titre d'exemple, vous pouvez ajouter la méthode suivante au type `Garage` (en utilisant une fonction locale pour encapsuler la fonctionnalité d'itération) :

```cs
public IEnumerable GetTheCars(bool returnReversed)
{
    // Effecture des vérification d'erreurs
    return ActualImplementation();
    
    IEnumerable ActualImplementation()
    {
        // Retourne les objets dans l'ordre inverse.
        if (returnReversed)
        {
            for (int i = carArray.Length; i != 0; i--)
            {
                yield return carArray[i - 1];
            }
        }
        // En C# moderne, on retire le else et on effectue la bloucle directement
        else
        {
            // Retourne les objets comme placé dans le tableau.
            foreach (Car c in carArray)
            {
                yield return c;
            }
        }
    }
}
```

==Notez que la nouvelle méthode permet à l'appelant d'obtenir les sous-éléments dans l'ordre séquentiel, ainsi que dans l'ordre inverse, si le paramètre entrant a la valeur `true`==. Vous pouvez maintenant interagir avec votre nouvelle méthode comme suit (veillez à commenter l'instruction `throw new exception`  dans la méthode `GetEnumerator()`) :

```cs
using System.Collections;
using CustomEnumeratorWithYield;

Console.Title = "Fun with the Yield Keyword";
Console.WriteLine("**** Fun with the Yield Keyword ****\n");

Garage carLot = new Garage();

// Récupère les éléments à l'aide de GetEnumerator().
foreach (Car c in carLot)
{
    Console.WriteLine($"{c.PetName} is going {c.CurrentSpeed} Km/h");
}

Console.WriteLine();

// Récupère les éléments (en sens inverse !)
// à l'aide d'un itérateur nommé.
foreach (Car c in carLot.GetTheCars(true))
{
    Console.WriteLine($"{c.PetName} is going {c.CurrentSpeed} Km/h");
}

Console.ReadLine();
```

Comme vous le savez sans doute, **les itérateurs nommés sont des constructions utiles, car un seul conteneur personnalisé peut définir plusieurs façons de demander l'ensemble retourné**.

Pour conclure cette présentation de la création d'objets énumérables, n'oubliez pas que **pour que vos types personnalisés fonctionnent avec le mot-clé `foreach` de C#, le conteneur doit définir une méthode nommée `GetEnumerator()`, formalisée par l'interface `IEnumerable`**. ==L'implémentation de cette méthode est généralement réalisée en la déléguant simplement au membre interne qui conserve les sous-objets== ; cependant, **il est également possible d'utiliser la syntaxe `yield return` pour fournir plusieurs méthodes "itérateur nommé"**.

# L'interface `ICloneable`

>[!warning]- L'interface `ICloneable` est considérée comme une **mauvaise pratique** par Microsoft et de nombreux experts car elle est **ambiguë**.
>
> ### L'ambiguïté de `ICloneable`
>
> L'interface ne possède qu'une méthode, `Clone()`, mais elle ne précise pas si cette copie doit être superficielle (_shallow_) ou profonde (_deep_). 
>
>- **Shallow Copy :** On crée un nouvel objet, mais les champs qui sont des références pointent toujours vers les **mêmes objets** en mémoire que l'original.
>- **Deep Copy :** On crée un nouvel objet et on duplique également **tous les objets imbriqués** de manière récursive. Les deux objets sont totalement indépendants.
>
>**Le risque :** Si vous utilisez une bibliothèque tierce qui expose `ICloneable`, vous ne savez jamais quel type de copie vous allez obtenir sans lire la documentation spécifique.
>	
>>[!example]- Sources
>>[Microsoft Learn | IClonable interface (System)](https://learn.microsoft.com/en-us/dotnet/api/system.icloneable?view=net-10.0#:~:text=ComVisibleAttribute-,Remarks,.NET)
>>[Stack OverFlow | Why should I implement IClonable in C#?](https://stackoverflow.com/questions/699210/why-should-i-implement-icloneable-in-c#:~:text=Although%20the%20ICloneable%20interface%20would,%2C%20don't%20use%20it.)
>>[Medium | Object Copy in C#](https://medium.com/@wa541964/object-copy-in-c-645ed2f2dea6#:~:text=Key%20takeaway:,City%20%7B%20get;%20set;%20%7D)
>>[dotnet-guide | How to Implement Object Cloning in .NET](https://www.dotnet-guide.com/how-do-you-implement-cloning-in-dot-net.html#:~:text=Shallow%20Copy%20vs%20Deep%20Copy,won't%20affect%20the%20original.&text=Notice%20how%20changing%20the%20Name,when%20they%20expect%20complete%20independence.)
>>[C# Corner | Shallow Copy and Deep Copy Using C#](https://www.c-sharpcorner.com/UploadFile/56fb14/shallow-copy-and-deep-copy-of-instance-using-C-Sharp/#:~:text=%2D%2D%2D%2D%2D%2D%2D,it%20using%20the%20following%20code:)
>>[GeeksforGeeks | Difference between Shallow and Deep copy of a class](https://www.geeksforgeeks.org/blogs/difference-between-shallow-and-deep-copy-of-a-class/#:~:text=Shallow%20Copy%20stores%20the%20references,object%20in%20the%20original%20object.)
>
> ### Le piège de `MemberwiseClone()`
>
>Beaucoup de développeurs implémentent `ICloneable` en utilisant simplement la méthode `MemberwiseClone()` héritée de `System.Object`.
> - **Attention :** `MemberwiseClone()` effectue toujours une **copie superficielle** (_shallow copy_). Si votre objet contient des listes ou d'autres classes, le clone partagera les mêmes données que l'original.
>	
>>[!example]- Sources
>> [Medium | Implement ICloneable as a Senior](https://medium.com/@iamprovidence/implement-icloneable-as-a-senior-327f31de5f25#:~:text=Firstly%2C%20you%20need%20to%20inherit,.IdInfo.IdNumber;%20//%202)
>>[Stack Overflow | Proper way to implement ICloneable](https://stackoverflow.com/questions/21116554/proper-way-to-implement-icloneable#:~:text=Comments,-Add%20a%20comment&text=Give%20your%20base%20class%20a,inherited%20version%20of%20Clone()%20.&text=If%20you%20want%20to%20skip,have%20to%20override%20CreateClone()%20.)
>>[Microsoft Learn | Object.MemberwiseClone Method (System)](https://learn.microsoft.com/en-us/dotnet/api/system.object.memberwiseclone?view=net-10.0#:~:text=The%20MemberwiseClone%20method%20creates%20a,of%20the%20field%20is%20performed.)
>>[Medium | Deep Copy vs Shallow Copy](https://medium.com/codeelevation/deep-copy-vs-shallow-copy-one-tiny-difference-huge-consequences-15bd9067e6a0)
>>[Coding Helmet | The ICloneable Controversy](https://codinghelmet.com/articles/implement-icloneable-or-not#:~:text=System.,right%20and%20who%20is%20wrong.)
>
> ### Les alternatives modernes (Recommandé)
>
>Pour éviter la confusion, préférez ces approches :
>
>- **Records (C# 9+) :** Utilisez `record` avec le mot-clé `with`. C'est natif, rapide et très lisible, bien que cela fasse par défaut une copie superficielle (mais sécurisée par l'immutabilité).
>- **Constructeur de copie :** Créez un constructeur qui prend une instance existante pour initialiser une nouvelle instance. C'est explicite et vous contrôlez la profondeur du clone.
>- **Sérialisation :** Pour une copie **profonde automatique**, vous pouvez sérialiser l'objet en JSON puis le désérialiser. C'est plus lent mais garantit l'indépendance totale.
>- **Méthodes explicites :** Nommez vos méthodes `DeepCopy()` ou `ShallowCopy()` au lieu d'utiliser `ICloneable.Clone()`. L'intention est alors claire pour celui qui utilise votre code	
>
>>[!example]- Sources
>> [poppastring | Shallow Copy and Deep Copy](https://www.poppastring.com/blog/c-shallow-copy-and-deep-copy#:~:text=In%20order%20to%20provide%20deep,other%20ways%20to%20do%20this)
>> [dotnet-guide | How to Implement Object Cloning in .NET](https://www.dotnet-guide.com/how-do-you-implement-cloning-in-dot-net.html#:~:text=Shallow%20Copy%20vs%20Deep%20Copy,won't%20affect%20the%20original.&text=Notice%20how%20changing%20the%20Name,when%20they%20expect%20complete%20independence.)
>> [Level Up Coding | Prototype Pattern in .NET C#](https://levelup.gitconnected.com/prototype-pattern-in-net-c-from-deep-cloning-to-modern-object-copying-2025-aec86dbfeea7#:~:text=Some%20developers%20implemented%20shallow%20copy,obtain%20a%20completely%20independent%20copy.)
>> [Stack Overflow | Proper way to implement ICloneable](https://stackoverflow.com/questions/21116554/proper-way-to-implement-icloneable#:~:text=Comments,-Add%20a%20comment&text=Give%20your%20base%20class%20a,inherited%20version%20of%20Clone()%20.&text=If%20you%20want%20to%20skip,have%20to%20override%20CreateClone()%20.)
>> [Stack Overflow | Why should I implement ICloneable in C# ?](https://stackoverflow.com/questions/699210/why-should-i-implement-icloneable-in-c#:~:text=Although%20the%20ICloneable%20interface%20would,%2C%20don't%20use%20it.)

Comme vous vous en souvenez peut-être du [[Chapitre 6#Tableau 6-1 Membres principaux de `System.Object`|Chapitre 6]], **`System.Object` définit une méthode nommée `MemberwiseClone()`. Cette méthode permet d'obtenir une copie superficielle** (*shallow copy*) **de l'objet courant**. ==Les utilisateurs de l'objet n'appellent pas directement cette méthode, car elle est protégée==. Toutefois, **un objet donné peut appeler cette méthode lui-même lors du processus de *clonage***. Pour illustrer cela, créez un nouveau projet d'application console nommé *CloneablePoint* qui définit une classe nommée `Point`.

```cs
namespace ClonablePoint;

// Une classe nommée Point
public class Point
{
    public int X { get; set; }
    public int Y { get; set; }

    public Point(int xPos, int yPos)
    {
        X = xPos;
        Y = yPos;
    }

    public Point() { }

    // Surcharge Object.ToString().
    public override string ToString() => $"X = {X}; Y = {Y}";
}
```

Compte tenu de vos connaissances actuelles sur les types référence et les types valeur (voir [[Chapitre 4#Comprendre les types de valeur et les types de référence|Chapitre 4]]), ==vous savez que si vous affectez une variable référence à une autre, vous avez deux références pointant vers le même objet en mémoire==. Ainsi, l'opération d'affectation suivante crée deux références au même objet `Point` dans le tas; les modifications effectuées à l'aide de l'une ou l'autre référence affectent le même objet dans le tas :

```cs
using ClonablePoint;

Console.Title = "Fun with Object Cloning";
Console.WriteLine("**** Fun with Object Cloning ****\n");

// Deux références pour le même objet
Point p1 = new Point(50, 50);
Point p2 = p1;
p2.X = 0;

Console.WriteLine(p1);
Console.WriteLine(p2);

Console.ReadLine();
```

==Lorsque vous souhaitez permettre à votre type personnalisé de renvoyer une copie identique de lui-même à l'appelant==, **vous pouvez implémenter l'interface standard `ICloneable`**. Comme indiqué au début de ce chapitre, ce type définit une unique méthode nommée `Clone()`.

```cs
public interface ICloneable
{
	object Clone();
}
```

Bien entendu, **l'implémentation de la méthode `Clone()` varie selon vos classes**. Cependant, ==sa fonctionnalité de base reste généralement la même : copier les valeurs de vos variables membres dans une nouvelle instance d'objet du même type et la renvoyer à l'utilisateur==. À titre d'exemple, considérons la mise à jour suivante de la classe `Point` :

```cs
namespace ClonablePoint;

// Une classe nommée Point
public class Point : ICloneable
{
    public int X { get; set; }
    public int Y { get; set; }

    public Point(int xPos, int yPos)
    {
        X = xPos;
        Y = yPos;
    }

    public Point() { }

    // Surcharge Object.ToString().
    public override string ToString() => $"X = {X}; Y = {Y}";

    // Retourne une copie de l'objet actuelle
    public object Clone() => new Point(this.X, this.Y);
}
```

Vous pouvez ainsi créer des copies exactes et autonomes du type `Point`, comme l'illustre le code suivant :

```cs
using ClonablePoint;

Console.Title = "Fun with Object Cloning";
Console.WriteLine("**** Fun with Object Cloning ****\n");

...

// Notez que Clone() renvoie un objet simple.
// Vous devez effectuer une conversion explicite
// pour obtenir le type dérivé.
Point p3 = new Point(100, 100);
Point p4 = (Point)p3.Clone();

// Modifie p4.X (ce qui ne modifiera pas p3.X).
p4.X = 0;

// Affiche chaque objet.
Console.WriteLine(p3);
Console.WriteLine(p4);

Console.ReadLine();
```

**Bien que l'implémentation actuelle de Point soit satisfaisante, il est possible de l'optimiser légèrement**. ==Comme le type `Point` ne contient aucune variable de type référence interne, vous pouvez simplifier l'implémentation de la méthode `Clone()` comme suit==.

```cs
// Copie chaque champ du membre Point, membre par membre.
public object Clone() => this.MemberwiseClone();
```

 **Notez toutefois que si le `Point` contenait des variables membres de type référence, `MemberwiseClone()` copierait les références à ces objets** ( c.-à-d. *copie superficielle*/*shallow copy*). **Pour une véritable *copie profonde* (*deep copy*), vous devrez créer une nouvelle instance de chaque variable de type référence lors du clonage**. Voyons un exemple.

## Exemple de clonage plus élaboré

==Supposons maintenant que la classe `Point` contienne une variable membre de type référence `PointDescription`==. **Cette classe gère le nom convivial d'un point ainsi qu'un numéro d'identification exprimé sous la forme d'un `System.Guid`** (un identificateur global unique `[GUID]` est un nombre statistiquement unique de 128 bits). Voici l'implémentation :

```cs
namespace ClonablePoint;

// Cette classe décrit un point.
public class PointDescription
{
    public string PetName { get; set; }
    public Guid PointID { get; set; }

    public PointDescription()
    {
        PetName = "No-name";
        PointID = Guid.NewGuid();
    }
}
```

Les premières mises à jour de la classe `Point` incluaient la modification de la méthode `ToString()` pour prendre en compte ces nouvelles données d'état, ainsi que la définition et la création du type de référence `PointDescription`. Pour permettre à l'extérieur d'attribuer un nom familier au `Point`, vous mettez également à jour les arguments passés au constructeur surchargé.

```cs
namespace ClonablePoint;

// Une classe nommée Point
public class Point : ICloneable
{
    public int X { get; set; }
    public int Y { get; set; }

    public PointDescription desc = new PointDescription();

    public Point(int xPos, int yPos, string petName)
    {
        X = xPos;
        Y = yPos;
        desc.PetName = petName;
    }

    public Point(int xPos, int yPos)
    {
        X = xPos;
        Y = yPos;
    }

    public Point() { }

    // Surcharge Object.ToString().
    public override string ToString() =>
        $"X = {X}; Y = {Y}; Name: {desc.PetName};\nID = {desc.PointID}\n";

    // Retourne une copie de l'objet actuel
    public object Clone() => this.MemberwiseClone();
}
```

==Notez que vous n'avez pas encore mis à jour votre méthode `Clone()`==. Par conséquent, ==lorsque l'utilisateur de l'objet demande un clone avec l'implémentation actuelle, une copie superficielle (membre par membre) est effectuée==. À titre d'exemple, supposons que vous ayez mis à jour le code appelant comme suit :

```cs
using ClonablePoint;

Console.Title = "Fun with Object Cloning";
Console.WriteLine("**** Fun with Object Cloning ****\n");

...

Console.WriteLine("Cloned p3 and stored new Point in p4");
Point p3 = new Point(100, 100, "Jane");
Point p4 = (Point)p3.Clone();

Console.WriteLine("Before modification:");
Console.WriteLine("p3: {0}", p3);
Console.WriteLine("p4: {0}", p4);

p4.desc.PetName = "My new Point";
p4.X = 9;

Console.WriteLine("\nChanged p4.desc.petName and p4.X");
Console.WriteLine("After modification:");
Console.WriteLine("p3: {0}", p3);
Console.WriteLine("p4: {0}", p4);

Console.ReadLine();
```

==Remarquez dans la sortie suivante que, bien que les types de valeurs aient effectivement été modifiés, les types de référence internes conservent les mêmes valeurs, car ils «pointent» vers les mêmes objets en mémoire== (plus précisément, notez que le nom familier des deux objets est maintenant `"My new Point"`).

```
**** Fun with Object Cloning ****

...

Cloned p3 and stored new Point in p4
Before modification:
p3: X = 100; Y = 100; Name: Jane;
ID = 03fb99c0-06d0-4937-98c4-269ecbfbce2d

p4: X = 100; Y = 100; Name: Jane;
ID = 03fb99c0-06d0-4937-98c4-269ecbfbce2d


Changed p4.desc.petName and p4.X
After modification:
p3: X = 100; Y = 100; Name: My new Point;
ID = 03fb99c0-06d0-4937-98c4-269ecbfbce2d

p4: X = 9; Y = 100; Name: My new Point;
ID = 03fb99c0-06d0-4937-98c4-269ecbfbce2d
```

**Pour que votre méthode `Clone()` effectue une *copie profonde* complète des types de référence internes, vous devez configurer l'objet renvoyé par `MemberwiseClone()` afin de prendre en compte le nom du point courant** (==le type `System.Guid` étant en réalité une structure, les données numériques sont bien copiées==). Voici une implémentation possible :

```cs
// Il faut maintenant ajuster le membre PointDescription.
public object Clone()
{
    // On commence par obtenir une copie superficielle.
    Point newPoint = (Point)this.MemberwiseClone();
    
    // On complète ensuite les données manquantes.
    PointDescription currentDesc = new PointDescription();
    currentDesc.PetName = this.desc.PetName;
    newPoint.desc = currentDesc;
    
    return newPoint;
}

// Version moin verbeuse
    // Il faut maintenant ajuster le membre PointDescription.
    // public object Clone()
    // {
        // On commence par obtenir une copie superficielle.
        //Point newPoint = (Point)MemberwiseClone();

        // On complète ensuite les données manquantes.
        // PointDescription currentDesc = new() { PetName = desc.PetName };
        // newPoint.desc = currentDesc;

        // return newPoint;
    // }
```

**Si vous relancez l'application et consultez le résultat** (illustré ci-après), **vous constaterez que le `Point` renvoyé par `Clone()` copie bien ses variables membres de type référence interne** (notez que le nom de l'animal est désormais unique pour `p3` et `p4`).

```
**** Fun with Object Cloning ****

...

Cloned p3 and stored new Point in p4
Before modification:
p3: X = 100; Y = 100; Name = Jane;
ID = 51f64f25-4b0e-47ac-ba35-37d263496406

p4: X = 100; Y = 100; Name = Jane;
ID = 0d3776b3-b159-490d-b022-7f3f60788e8a


Changed p4.desc.petName and p4.X
After modification:
p3: X = 100; Y = 100; Name = Jane;
ID = 51f64f25-4b0e-47ac-ba35-37d263496406

p4: X = 9; Y = 100; Name = My new Point;
ID = 0d3776b3-b159-490d-b022-7f3f60788e8a
```

==Pour résumer le processus de clonage, si vous avez une classe ou une structure ne contenant que des types valeur, implémentez votre méthode `Clone()` en utilisant `MemberwiseClone()`==. Cependant, **si vous avez un type personnalisé qui gère d'autres types référence, vous pouvez créer un nouvel objet qui considère chaque variable membre de type référence pour obtenir une** « copie profonde ».

>[!tip] Il est important de retourner voir le warning au début de cette section car **le paragraphe précédent ne fait plus partie des "bonnes pratiques" de C#**.

# L'interface `IComparable`

>[!warning]- Performance et Sécurité des Types
>L'implémentation de l'interface classique `IComparable` (non-générique) comporte des pièges techniques liés à l'architecture .NET :
>- **Coût du Boxing :** Puisque `CompareTo(object obj)` accepte un type `object`, comparer des types valeurs (comme `struct`) force le passage des données de la **Pile** vers le **Tas**. Ce processus, appelé **Boxing**, dégrade fortement les performances lors de tris sur de grandes listes.
>- **Manque de Sécurité :** La signature accepte n'importe quel objet. Une erreur de type ne sera détectée qu'à l'exécution par une exception, et non à la compilation.
>
>**Recommandation  :**
>1. Implémentez toujours la version générique **`IComparable<T>`** pour éviter le boxing et garantir la sécurité.
>2. Utilisez l'**implémentation explicite** pour masquer la méthode `object IComparable.CompareTo` tout en restant compatible avec les anciens composants du framework.


**L'interface `System.IComparable` spécifie un comportement permettant de trier un objet en fonction d'une clé spécifiée**. Voici sa définition formelle :

```cs
// Cette interface permet à un objet de spécifier sa
// relation avec d'autres objets similaires.
public interface IComparable
{ 
	int CompareTo(object o);
}
```

>[!note] **La version générique de cette interface (`IComparable<T>`) offre une méthode plus sûre en termes de typage pour gérer les comparaisons entre objets**. Vous étudierez les génériques au [[Chapitre 10|Chapitre 10]].

Créez un nouveau projet d'application console nommé *ComparableCar*, copiez les classes `Car` et `Radio` de l'exemple *SimpleException* du [[Chapitre 7#L'exemple le plus simple possible|Chapitre 7]] et renommez l'espace de noms de chaque fichier en `ComparableCar`. Mettez à jour la classe `Car` en ajoutant une nouvelle propriété représentant un identifiant unique pour chaque voiture et un constructeur modifié.

```cs
using System.Collections;

namespace ComparableCar;

class Car
{
	...
	
    public int CarID { get; set; }

	...

    public Car(string name, int speed, int id)
    {
        CurrentSpeed = speed;
        PetName = name;
        CarID = id;
    }

	...
}
```

Supposons maintenant que vous ayez un tableau d'objets `Car` comme suit dans vos instructions de niveau supérieur :

```cs
global using System.Collections;
using ComparableCar;

Console.Title = "Fun with Object Sorting";
Console.WriteLine("***** Fun with Object Sorting *****\n");

// Crée un tableau d'objet Car
Car[] myAutos =
[
    new("Rusty", 80, 1),
    new("Mary", 40, 234),
    new("Viper", 40, 34),
    new("Mel", 40, 4),
    new("Chucky", 40, 5),
];

Console.ReadLine();
```

**La classe `System.Array` définit une méthode statique nommée `Sort()`**. ==Lorsque vous appelez cette méthode sur un tableau de types intrinsèques== (`int`, `short`, `string`, etc.), ==vous pouvez trier les éléments du tableau par ordre numérique ou alphabétique, car ces types de données intrinsèques implémentent l'interface `IComparable`==. Cependant, que se passe-t-il si vous transmettez un tableau de types `Car` à la méthode `Sort()` comme suit ?

```cs
// Trier mes voitures ? pas encore!
Array.Sort(MyAutos);
```

**Si vous exécutez ce test, vous obtiendrez une exception d'exécution, car la classe `Car` ne prend pas en charge l'interface nécessaire**. Lorsque vous créez des types personnalisés, ==vous pouvez implémenter `IComparable` pour permettre le tri des tableaux de vos types==. **Lorsque vous détaillerez la méthode `CompareTo()`, il vous appartiendra de définir la base de l'opération de tri**. Pour le type `Car`, l'identifiant interne `CarID` semble être le candidat le plus logique.

```cs
using System.Collections;

namespace ComparableCar;

// L'itération de la voiture peut être commandée
// en fonction de l'identifiant de la voiture.
public class Car : IComparable
{

	...
	
    // Implémentation de IComparable
    int IComparable.CompareTo(object obj)
    {
        if (obj is Car temp)
        {
            if (this.CarID > temp.CarID)
            {
                return 1;
            }
            if (this.CarID < temp.CarID)
            {
                return -1;
            }
            return 0;
        }
        throw new ArgumentException("parameter is not a Car!");
    }
}
```

Comme vous pouvez le constater, **la logique de `CompareTo()` consiste à comparer l'objet entrant à l'instance actuelle en fonction d'un point de données spécifique**. ==La valeur de retour de `CompareTo()` permet de déterminer si ce type est inférieur, supérieur ou égal à l'objet avec lequel il est comparé== (voir [[#Tableau 8-1 Valeurs de retours de la méthode `CompareTo()`|Tableau 8-1]]).

##### Tableau 8-1: Valeurs de retours de la méthode `CompareTo()`

| Valeur de retour             | Description                                                         |
| ---------------------------- | ------------------------------------------------------------------- |
| Tout nombre inférieur à zéro | Cette instance précède l'objet spécifié dans l'ordre de tri.        |
| Zéro                         | Cette instance est égale à l'objet spécifié                         |
| Tout nombre supérieur à zéro | Cette instance apparaît après l'objet spécifié dans l'ordre de tri. |

**Vous pouvez simplifier l'implémentation précédente de `CompareTo()` étant donné que le type de données `int` de C#** (qui est simplement une notation abrégée pour `System.Int32`) **implémente `IComparable`**. Vous pourriez implémenter la méthode `CompareTo()` de la voiture comme suit :

```cs
int IComparable.CompareTo(object obj)
{
    if (obj is Car temp)
    {
        return CarID.CompareTo(temp.CarID);
    }
    throw new ArgumentException("parameter is not a Car!");
}
```

Dans les deux cas, pour que votre type `Car` comprenne comment se comparer à des objets similaires, vous pouvez écrire le code utilisateur suivant :

```cs
global using System.Collections;
using ComparableCar;

Console.Title = "Fun with Object Sorting";
Console.WriteLine("**** Fun with Object Sorting ****\n");

...

// Affiche le tableau actuel
Console.WriteLine("Here is the unordered set of cars:");
foreach (Car c in myAutos)
{
    Console.WriteLine($"{c.CarID} {c.PetName}");
}

// Maintenant, on les trie en utilisant IComparable!
Array.Sort(myAutos);
Console.WriteLine();

// Affiche le tableau triée.
Console.WriteLine("Here is the ordered set of cars:");
foreach (Car c in myAutos)
{
    Console.WriteLine($"{c.CarID} {c.PetName}");
}
Console.ReadLine();
```

Voici le résultat de l'extrait de code précédent :

```
**** Fun with Object Sorting ****

Here is the unordered set of cars:
1 Rusty
234 Mary
34 Viper
4 Mel
5 Chucky

Here is the ordered set of cars:
1 Rusty
4 Mel
5 Chucky
34 Viper
234 Mary
```

## Spécification de plusieurs ordres de tri avec `IComparer`

Dans cette version du type `Car`, vous avez utilisé l'identifiant comme base pour l'ordre de tri. ==Une autre conception aurait pu utiliser le surnom  comme base de l'algorithme de tri== (pour lister les voitures par ordre alphabétique). Maintenant, **que faire si vous vouliez créer une voiture qui puisse être triée *à la fois* par identifiant et par surnom** ? Si ce type de comportement vous intéresse, **vous devez vous familiariser avec une autre interface standard nommée `IComparer`, définie dans l'espace de noms `System.Collections` comme suit** :

```cs
// Une manière générale de comparer deux objets.
interface IComparer
{
	int Compare(object o1, object o2);
}
```

>[!note] **La version générique de cette interface (`IComparer<T>`) offre une méthode plus sûre en termes de typage pour gérer les comparaisons entre objets**. Vous étudierez les génériques au [[Chapitre 10|Chapitre 10]].

Contrairement à l'interface `IComparable`, **`IComparer` n'est généralement *pas* implémentée sur le type que vous souhaitez trier** (par exemple, la classe `Car`). **Vous implémentez plutôt cette interface sur un nombre quelconque de classes auxiliaires, une pour chaque critère de tri** (surnom, identifiant de la voiture, etc.). ==Actuellement, le type `Car` sait déjà se comparer aux autres voitures grâce à son identifiant interne. Par conséquent, permettre à l'utilisateur de trier un tableau d'objets `Car` par nom d'animal nécessitera une classe auxiliaire supplémentaire implémentant `IComparer`==. Voici le code :

```cs
namespace ComparableCar;

// Cette classe d'aide est utilisée pour trier
// un tableau de Car par leurs suronms.
public class PetNameComparer : IComparer
{
    // Teste le surnom de chaque objets.
    int IComparer.Compare(object o1, object o2)
    {
        if (o1 is Car t1 && o2 is Car t2)
        {
            return string.Compare(
                t1.PetName,
                t2.PetName,
                StringComparison.OrdinalIgnoreCase
            );
        }
        throw new ArgumentException("Parameter is not a Car!");
    }
}
```

Le code utilisateur de l'objet peut utiliser cette classe d'assistance. ==`System.Array` possède plusieurs méthodes `Sort()` surchargées, dont une qui accepte un objet implémentant `IComparer`==.

```cs

...

// Maintenant, on les trient par surnoms
Array.Sort(myAutos, new PetNameComparer());
Console.WriteLine();


// Affiche le tableau trié.
Console.WriteLine("Ordering by pet name:");
foreach (Car c in myAutos)
{
    Console.WriteLine($"{c.CarID} {c.PetName}");
}
Console.ReadLine();
```

## Propriétés personnalisées et types de tri personnalisés

Il est important de noter que **vous pouvez utiliser une propriété statique personnalisée pour faciliter le tri des types `Car` par l'utilisateur selon un critère de tri spécifique**. ==Supposons que la classe `Car` ait ajouté une propriété statique en lecture seule nommée `SortByPetName` qui renvoie une instance d'un objet implémentant l'interface `IComparer`== (`PetNameComparer`, dans ce cas; assurez-vous d'importer `System.Collections`).

```cs
// On supporte maintenant une propriété personnalisée
// pour retourner l'object correcte IComparer.
public class Car : IComparable
{
	...
	
    // Propriété pour retourner l'objet PetNameComparer
    public static IComparer SortByPetName => (IComparer)new PetNameComparer();
    
    ...
}
```

==Le code utilisateur de l'objet peut désormais trier par nom d'animal de compagnie à l'aide d'une propriété fortement associée==, au lieu de simplement "devoir savoir" utiliser le type de classe autonome `PetNameComparer`.

```cs
// Le tri par surnom a été amélioré.
Array.Sort(myAutos, Car.SortByPetName);
```

**Idéalement, à ce stade, vous devriez non seulement savoir définir et implémenter vos propres interfaces, mais aussi comprendre leur utilité**. ==Bien entendu, les interfaces sont présentes dans tous les principaux espaces de noms .NET Core, et vous continuerez à travailler avec diverses interfaces standard dans la suite de cet ouvrage==.

# Résumé du chapitre

**Une interface peut être définie comme un ensemble nommé de membres abstraits**. ==On considère généralement une interface comme un comportement pouvant être pris en charge par un type donné==. **Lorsque deux classes ou plus implémentent la même interface, vous pouvez traiter chaque type de la même manière (polymorphisme basé sur les interfaces), même si les types sont définis au sein de hiérarchies de classes distinctes**.

C# fournit le mot-clé `interface` pour vous permettre de définir une nouvelle interface. **Comme vous l'avez vu, un type peut prendre en charge autant d'interfaces que nécessaire, à l'aide d'une liste séparée par des virgules**. De plus, ==il est possible de créer des interfaces dérivées de plusieurs interfaces de base==.

Outre la possibilité de créer vos propres interfaces, **les bibliothèques .NET Core définissent plusieurs interfaces standard** (c'est-à-dire fournies par le framework). Comme vous l'avez vu, ==vous pouvez créer des types personnalisés implémentant ces interfaces prédéfinies afin d'obtenir plusieurs caractéristiques utiles telles que le clonage, le tri et l'énumération==.

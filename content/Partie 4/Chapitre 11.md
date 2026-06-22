---
title: "Chapitre 11: Fonctionnalités Avancées du Language"
publish: true
---

# <big><big><big><b><font color =green>Fonctionnalités Avancées du Langage C#</font></b></big></big></big>

Dans ce chapitre, vous approfondirez votre compréhension du langage de programmation C# en explorant plusieurs sujets plus avancés. Pour commencer, vous apprendrez à implémenter et à utiliser une *méthode d'indexation*. **Ce mécanisme C# vous permet de créer des types personnalisés offrant un accès aux sous-éléments internes à l'aide d'une syntaxe similaire à celle des tableaux**. Après avoir appris à créer une méthode d'indexation, **vous verrez comment surcharger différents opérateurs (`+`, `-`, `<`, `>`, etc.) et comment créer des routines de conversion explicites et implicites personnalisées pour vos types** (et vous comprendrez pourquoi cela peut s'avérer utile).

Ensuite, **==vous étudierez des sujets particulièrement utiles lors de l'utilisation d'API centrées sur LINQ==** (bien que vous puissiez les utiliser en dehors du contexte de LINQ), notamment **les méthodes d'extension et les types anonymes**.

Pour conclure, vous apprendrez à **créer un contexte de code «non sécurisé» pour manipuler directement des pointeurs non managés**. Bien qu'il soit vrai que l'utilisation de pointeurs dans les applications C# soit peu fréquente, comprendre comment les utiliser peut s'avérer utile dans certaines circonstances impliquant des scénarios d'interopérabilité complexes.

# Comprendre les méthodes d'indexation

En tant que programmeur, vous connaissez certainement le processus d'accès aux éléments individuels contenus dans un tableau simple à l'aide de l'opérateur d'indexation (`[]`). Voici un exemple :

```cs
// Boucle sur les arguments de la ligne de commande entrants
// en utilisant l'opérateur d'index.
for(int i = 0; i < args.Length; i++)
{
	Console.WriteLine("Args: {0}", args[i]);
}

// Déclarer un tableau d'entiers locaux.
int[] myInts = { 10, 9, 100, 432, 9874};

// Utilisez l'opérateur d'index pour accéder à chaque élément.
for(int j = 0; j < myInts.Length; j++)
{
	Console.WriteLine("Index {0} = {1} ", j, myInts[j]);
}
Console.ReadLine();
```

Ce code n'a rien de révolutionnaire. Cependant, **le langage C# offre la possibilité de concevoir des classes et des structures personnalisées indexables comme un tableau standard**, en définissant une *méthode d'indexation*. ==Cette fonctionnalité est particulièrement utile lors de la création de classes de collections personnalisées== (génériques ou non génériques).

Avant d'examiner comment implémenter un indexeur personnalisé, voyons-en un exemple concret. Supposons que vous ayez ajouté la prise en charge d'une méthode d'indexation au type `PersonCollection` personnalisé développé au [[Chapitre 10#Les problèmes des collections non génériques|Chapitre 10]] (plus précisément, dans le projet *IssuesWithNonGenericCollections*). Bien que vous n'ayez pas encore ajouté l'indexeur, observez son utilisation suivante dans un nouveau projet d'application console nommé *SimpleIndexer* :

```cs
using System.Data;
using SimpleIndexer;

// Les indexeurs permettent d'accéder aux objets sous forme de tableau.
Console.Title = "Fun with Indexers";
Console.WriteLine("***** Fun with Indexers *****\n");

PersonCollection myPeople = new PersonCollection();

// Ajoute des objets avec la syntaxe d'indexeur.
myPeople[0] = new Person("Homer", "Simpson", 40);
myPeople[1] = new Person("Marge", "Simpson", 38);
myPeople[2] = new Person("Lisa", "Simpson", 9);
myPeople[3] = new Person("Bart", "Simpson", 7);
myPeople[4] = new Person("Maggie", "Simpson", 2);

// Maintenant, obtenez et affichez chaque élément à l'aide de l'indexeur.
for (int i = 0; i < myPeople.Count; i++)
{
    Console.WriteLine($"Person number: {i}");
    Console.WriteLine($"Name: {myPeople[i].FirstName} {myPeople[i].LastName}");
    Console.WriteLine($"Age: {myPeople[i].Age}");
    Console.WriteLine();
}
```

Comme vous pouvez le constater, **les indexeurs vous permettent de manipuler la collection interne de sous-objets comme un tableau standard**. La question principale est maintenant : comment configurer la classe `PersonCollection` (ou toute autre classe ou structure personnalisée) pour prendre en charge cette fonctionnalité ? **==Un indexeur est représenté par une définition de propriété C# légèrement modifiée. Dans sa forme la plus simple, un indexeur est créé à l’aide de la syntaxe `this[]`==**. Voici la mise à jour nécessaire pour la classe `PersonCollection` :

```cs
using System.Collections;

namespace SimpleIndexer;

public class PersonCollection : IEnumerable
{
    private ArrayList arPeople = new ArrayList();
    
    ...

    // Ajoute un indexeur à la définition de classe existante.
    public Person this[int pos]
    {
        get => arPeople[pos] as Person;
        set => arPeople.Insert(pos, value);
    }
    
    ...
}
```

Hormis l'utilisation du mot-clé `this` entre parenthèses, l'indexeur ressemble à n'importe quelle autre déclaration de propriété C#. Par exemple, ==le rôle de la portée `get` est de renvoyer l'objet correct à l'appelant==. Ici, **vous déléguez la requête à l'indexeur de l'objet `ArrayList`, car cette classe prend également en charge un indexeur**. ==La portée `set` permet d'ajouter de nouveaux objets `Person`== ; **ceci est réalisé en appelant la méthode `Insert()` de `ArrayList`**.

**Les indexeurs constituent une forme supplémentaire de sucre syntaxique, étant donné que cette fonctionnalité peut également être obtenue à l'aide de méthodes publiques « normales » telles que `AddPerson()` ou `GetPerson()`**. Néanmoins, ==lorsque vous prenez en charge les méthodes d'indexation sur vos types de collections personnalisés, elles s'intègrent parfaitement aux bibliothèques de classes de base .NET==.

Bien que la création de méthodes d'indexation soit assez courante lors de la création de collections personnalisées, **n'oubliez pas que les types génériques vous offrent cette fonctionnalité nativement**. Prenons l'exemple de la méthode suivante, qui utilise une `List<T>` générique d'objets `Person`. Notez que vous pouvez simplement utiliser l'indexeur de `List<T>` directement. Voici un exemple :

```cs
static void UseGenericListOFPeople()
{
    List<Person> myPeople = new List<Person>();
    myPeople.Add(new Person("Lisa", "Simpson", 9));
    myPeople.Add(new Person("Bart", "Simpson", 7));

    // Change la première personnne avec indexeur
    myPeople[0] = new Person("Maggie", "Simpson", 2);

    // Maintenant, obtenez et affichez chaque élément à l'aide de l'indexeur.
    for (int i = 0; i < myPeople.Count; i++)
    {
        Console.WriteLine($"Person number: {i}");
        Console.WriteLine(
            $"Name: {myPeople[i].FirstName} {myPeople[i].LastName}"
        );
        Console.WriteLine($"Age: {myPeople[i].Age}");
        Console.WriteLine();
    }
}
```

## Indexation des données à l'aide de chaînes de caractères

La classe `PersonCollection` actuelle définit un indexeur permettant à l'appelant d'identifier les sous-éléments à l'aide d'une valeur numérique. **Notez toutefois que cela n'est pas une exigence d'une méthode d'indexation**. ==**Supposons que vous préfériez stocker les objets `Person` dans un `System.Collections.Generic.Dictionary<TKey, TValue>` plutôt que dans un `ArrayList`. Étant donné que les types `Dictionary` permettent d'accéder aux types contenus à l'aide d'une clé (telle que le prénom d'une personne), vous pouvez définir un indexeur comme suit**== :

```cs
using System.Collections;

namespace SimpleIndexer;

// Utilisation de la version générique de IEnumerable (plus moderne)
public class PersonCollectionStringIndexer : IEnumerable<Person>
{
	// Syntaxe permise depuis C# 12 (Collection Expression)
    private Dictionary<string, Person> listPeople = [];

    // Cet indexeur retourne une personne basée sur un index string.
    public Person this[string name]
    {
        // plus besoin de cast car on utilise une classe générique
        // comme contennant (Dictionary <TKey, TValue>).
        get => listPeople[name];
        set => listPeople[name] = value;
    }

    public void ClearPeople()
    {
        listPeople.Clear();
    }

    public int Count => listPeople.Count;

    // On renvoit un objet Person, et non un tuple de (string, Person)
    public IEnumerator<Person> GetEnumerator() =>
        listPeople.Values.GetEnumerator();

    IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();
}
```

L'appelant pourrait désormais interagir avec les objets `Person` contenus, comme illustré ici :

```cs
PersonCollectionStringIndexer myPeopleStrings = new();
myPeopleStrings["Homer"] = new Person("Homer", "Simpson", 40);
myPeopleStrings["Marge"] = new Person("Marge", "Simpson", 38);

// Récupère "Homer" et affiche les données
Person homer = myPeopleStrings["Homer"];
Console.WriteLine(homer);

Console.ReadLine();
```

Là encore, ==si vous utilisiez directement le type générique `Dictionary<TKey, TValue>`, vous bénéficieriez de la fonctionnalité de la méthode d'indexation nativement, sans avoir à créer une classe personnalisée non générique prenant en charge un indexeur de chaînes==. Toutefois, **il est important de comprendre que le type de données de tout indexeur dépendra de la manière dont le type de collection sous-jacent permet à l'appelant de récupérer les sous-éléments**.

## Surcharge des méthodes d'indexation

**Les méthodes d'indexation peuvent être surchargées sur une même classe ou structure**. Ainsi, s'il est pertinent de permettre à l'appelant d'accéder aux sous-éléments à l'aide d'un index numérique ou d'une valeur de type chaîne, ==vous pouvez définir plusieurs indexeurs pour un même type==. Par exemple, dans ADO.NET (l'API native d'accès aux bases de données de .NET), la classe `DataSet` prend en charge une propriété nommée `Tables`, qui renvoie un type `DataTableCollection` fortement typé. Il s'avère que `DataTableCollection` définit trois indexeurs pour obtenir et définir des objets `DataTable` : l'un par position ordinale et les autres par un nom de chaîne convivial et un espace de noms conteneur facultatif, comme illustré ici :

```cs
public sealed class DataTableCollection : InternalDataCollectionBase
{
	...
	// Indexeurs surchargées.
	public DataTable this[int index] { get; }
	public DataTable this[string name] { get; }
	public DataTable this[string name, string tableNamespace] { get; }
}
```

**Il est courant que les types des bibliothèques de classes de base prennent en charge les méthodes d'indexation**. Par conséquent, même si votre projet actuel ne nécessite pas la création d'indexeurs personnalisés pour vos classes et structures, ==sachez que de nombreux types prennent déjà en charge cette syntaxe.==

## Indexeurs multidimensionnels
 
 Vous pouvez également **créer une méthode d'indexation prenant plusieurs paramètres**. Supposons que vous ayez une ==collection personnalisée qui stocke des sous-éléments dans un tableau 2D==. Dans ce cas, vous pouvez définir une méthode d'indexation comme suit :

```cs
public class SomeContainer
{ 
	private int[,] my2DintArray = new int[10, 10];

	public int this[int row, int column]
	{ /* Récupérer ou définir une valeur du tableau 2D */ }
}
```

~~Encore une fois, à moins de créer une classe de collection personnalisée très stylisée, vous n'aurez que rarement besoin de créer un indexeur multidimensionnel.~~ Néanmoins, ADO.NET démontre une fois de plus l'utilité de cette construction. L'objet `DataTable` d'ADO.NET est essentiellement une collection de lignes et de colonnes, semblable à une feuille de papier millimétré ou à la structure générale d'une feuille de calcul Microsoft Excel.

Bien que les objets `DataTable` soient généralement remplis automatiquement par un «adaptateur de données» associé, le code suivant illustre comment créer manuellement un `DataTable` en mémoire contenant trois colonnes (pour le prénom, le nom et l'âge de chaque enregistrement). Notez qu'une fois une seule ligne ajoutée au `DataTable`, vous utilisez un indexeur multidimensionnel pour explorer chaque colonne de cette première (et unique) ligne. (Si vous suivez ce tutoriel, vous devrez importer l'espace de noms `System.Data` dans votre fichier de code.)

>[!tip] Les conclusion de l'auteur sur la rareté de cet utilisation est fausse
>Les indexeurs multidimensionnels sont **vitaux** dans plusieurs domaines modernes :
>
>- **Data Science et IA** : Si vous manipulez des matrices ou des tenseurs (avec des bibliothèques comme _TensorFlow.NET_ ou _NumSharp_), vous passez votre temps à écrire `matrix[x, y]`.
>- **Développement de Jeux Vidéo (Unity / Godot)** : Pour gérer une carte de jeu, une grille d'inventaire ou un moteur de voxels, l'indexeur multidimensionnel (`grid[x, y, z]`) est la syntaxe standard.
>- **Traitement d'images** : Accéder à un pixel via `image[x, y]` est beaucoup plus naturel que d'appeler une méthode `GetPixel(x, y)`.
>

>[!warning] **L'example de code suivant fourni par le livre ne montrent pas l'utilisation d'un indexeur multidimentionnelle.**
>
>L'auteur utilise un abus de language et "confond" les *indexeurs multidimentionnelles* et les *indexeurs en cascade*.
>
>- **`SomeContainer[0, 0]`** : C’est un **vrai indexeur multidimensionnel**. Il n'y a qu'**un seul appel** d'indexeur qui prend deux paramètres séparés par une virgule. La classe `SomeContainer` contrôle directement la grille.
>- **`myTable.Rows[0][0]`** : C'est un **tableau de tableaux** (ou indexeurs en cascade). Il y a **deux appels successifs**. Le premier `[0]` extrait un objet `DataRow`, puis le second `[0]` interroge cet objet extrait.

```cs
static void MultiIndexerWithDataTable()
{
    // Crée une DataTable simple avec 3 colonnes
    DataTable myTable = new DataTable();
    myTable.Columns.Add(new DataColumn("FirstName"));
    myTable.Columns.Add(new DataColumn("LastName"));
    myTable.Columns.Add(new DataColumn("Age"));

    // Ajoute maintenant une ligne dans le tableau.
    myTable.Rows.Add("Mel", "Appleby", 60);

    // Utilisation de l'indexeur en cascade (erreur dans le livre) 
    // pour avoir les détails de la première ligne
    Console.WriteLine($"First Name: {myTable.Rows[0][0]}");
    Console.WriteLine($"Last Name: {myTable.Rows[0][1]}");
    Console.WriteLine($"Age: {myTable.Rows[0][2]}");
}
```

Sachez que vous explorerez en profondeur ADO.NET à partir du [[Chapitre 20|Chapitre 20]]. Si une partie du code précédent vous semble inconnue, n’ayez crainte. ==L’objectif principal de cet exemple est de montrer que les méthodes d’indexation peuvent gérer plusieurs dimensions et, utilisées correctement, simplifier l’interaction avec les sous-objets contenus dans des collections personnalisées==.

## Définitions d'indexeurs sur les types d'interface

**Des indexeurs peuvent être définis sur un type d'interface .NET donné afin de permettre aux types de support de fournir une implémentation personnalisée**. Voici un exemple simple d'interface définissant un protocole pour obtenir des objets chaîne de caractères à l'aide d'un indexeur numérique :

```cs
public interface IStringContainer
{
	string this[int index] { get; set; }
}
```

Avec cette définition d'interface, toute classe ou structure qui implémente cette interface doit désormais prendre en charge un indexeur en lecture-écriture qui manipule les sous-éléments à l'aide d'une valeur numérique. Voici une implémentation partielle d'une telle classe :

```cs
class SomeClass : IStringContainer
{
	private List<string> myStrings = new List<string>();
	public string this[int index]
	{
		get => myStrings[index];
		set => myStrings.Insert(index, value);
	}
}
```

Ceci conclut le premier sujet majeur de ce chapitre. **==Examinons maintenant une fonctionnalité du langage qui vous permet de créer des classes ou des structures personnalisées qui réagissent de manière unique aux opérateurs intrinsèques de C#==**. **Permettez-moi ensuite de vous présenter le concept de *surcharge d'opérateurs.***

# Comprendre la surcharge de l'opérateur

>[!tip] Vision moderne sur cette section (en lien avec [[#Réflexions finales concernant la surcharge d'opérateurs|cette sous-section]])
>c'est une fonctionnalité puissante, mais elle est devenue une **niche**. Elle est considéré comme un **sucre syntaxique** réservé aux types "atomiques" (ceux qui se comportent comme des nombres ou des valeurs pures).
>
>Pourquoi on en fait "moins" ?
>
>1. **Les Records (C# 9+)** :
>	- Les `record` et `record struct` gèrent automatiquement l'égalité par valeur (`==`). Plus besoin de surcharger manuellement les opérateurs d'égalité dans la plupart des cas !
>2. **LINQ** :
>	- On manipule des collections avec des méthodes (`.Where`, `.Select`) plutôt que par des opérateurs.
>3. **Generic Math (C# 11)** :
>	- On définit maintenant des interfaces implémentant la surcharge. (`INumber<T>`, `IAdditionOperators<T, T, T>`, ...) 

**C#, comme tout langage de programmation, possède un ensemble prédéfini de jetons permettant d'effectuer des opérations de base sur les types intrinsèques**. Par exemple, ==vous savez que l'opérateur `+` peut être appliqué à deux entiers pour obtenir un entier plus grand==.

```cs
// L'opérateur + avec les entiers.
int a = 100;
int b = 240;
int c = a + b; // c vaut maintenant 340
```

Encore une fois, ce n'est pas une révélation, mais avez-vous déjà remarqué que ==le même opérateur `+` peut être appliqué à la plupart des types de données intrinsèques de C# ?== Par exemple, considérez ce code :

```cs
// Opérateur + avec les chaînes de caractères.
string s1 = "Hello";
string s2 = " world!";
string s3 = s1 + s2; // s3 vaut maintenant "Hello World!"
```

**L'opérateur `+` fonctionne de manière spécifique selon les types de données fournis** (chaînes de caractères ou entiers, dans ce cas). ==Appliqué à des types numériques, il renvoie la somme des opérandes. En revanche, appliqué à des chaînes de caractères, il renvoie la concaténation de ces chaînes.==

**Le langage C# permet de créer des classes et des structures personnalisées qui réagissent également de manière unique aux mêmes opérateurs de base** (comme l'opérateur `+`). Bien que **==tous les opérateurs C# ne soient pas surchargeables, un grand nombre le sont==**, comme illustré dans le [[#Tableau 11-1 Surchargeabilité des opérateurs C|Tableau 11-1]].

##### Tableau 11-1: Surchargeabilité des opérateurs C#

| Opérateur C#                                                 | Surchargeablilté                                                                                                                                                                                                            |
| ------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `+`, `-`, `!`, `~`, `++`, `--` , `true`, `false`             | Ces opérateurs unaires peuvent être surchargés. En C#, si `true` ou `false` est surchargé, les deux doivent l'être.                                                                                                         |
| `+`, `-`, `*`, `/`, `%`, `&`, `\|`, `^`, `<<`, `>>`          | Ces opérateur binaires peuvent être surchargés.                                                                                                                                                                             |
| `==`, `!=`, `<`, `>`, `<=`, `>=`                             | Ces opérateurs de comparaison peuvent être surchargés. En C#, il est obligatoire de surcharger simultanément les opérateurs de type « like » (c’est-à-dire `<` et `>`, `<=` et `>`=, `==` et `!=`).                         |
| `[]`                                                         | L'opérateur `[]` ne peut pas être surchargé. Comme vous l'avez vu précédemment dans ce chapitre, la construction d'indexeur offre cependant la même fonctionnalité.                                                         |
| `()`                                                         | L'opérateur `()` ne peut pas être surchargé. Comme vous le verrez plus loin dans ce chapitre, cependant, des méthodes de conversion personnalisées offrent les mêmes fonctionnalités.                                       |
| `+=`, `-=`, `*=`, `/=`, `%=`, `&=`,`\|=`, `^=`, `<<=`, `>>=` | Les opérateurs d'affectation abrégés ne peuvent pas être surchargés ; cependant, vous les obtenez gratuitement lorsque vous surchargez l'opérateur binaire associé.                                                         |
| `??`, `??=`                                                  | Les opérateur de coalescences nulles ne peuvent pas être surchargée. C# protège la sémantique fondamentale du langage. L'affectation (`=`) et la vérification de nullité (`??`) sont des piliers de la sécurité du runtime. |

## Surcharge des opérateurs binaires

Pour illustrer le processus de surcharge des opérateurs binaires, supposons que la classe `Point` simple suivante soit définie dans un nouveau projet d’application console nommé *OverloadedOps* :

```cs
namespace OverloadedOps;
// Une simple classe C# de tous les jours.
public class Point
{

  public int X { get; set; }
  public int Y { get; set; }

  public Point(int xPos, int yPos)
  {
      X = xPos;
      Y = yPos;
  }

  public override string ToString() => $"[{this.X}, {this.Y}]";
}
```

Logiquement, il est pertinent d'« additionner » des `Point`s. Par exemple, si vous additionnez deux variables de type `Point`, vous obtenez un nouveau point qui correspond à la somme des valeurs `X` et `Y`. Bien sûr, il peut également être utile de soustraire un point d'un autre. **Idéalement, vous devriez pouvoir écrire le code suivant** :

```cs
using OverloadedOps;

// Ajouter et soustraire deux Point ?
Console.Title = "Fun with Overloaded Operators";
Console.WriteLine("***** Fun with Overloaded Operators *****\n");

// Créer deux points.
Point ptOne = new Point(100, 100);
Point ptTwo = new Point(40, 40);
Console.WriteLine("ptOne = {0}", ptOne);
Console.WriteLine("ptTwo = {0}", ptTwo);

// Additionner les points pour en former un plus grand ?
Console.WriteLine("ptOne + ptTwo : {0}", ptOne + ptTwo);

// Soustraire les points pour en former un plus petit ?
Console.WriteLine("ptOne - ptTwo : {0}", ptOne - ptTwo);
Console.ReadLine();
```

Cependant, **en l'état actuel de votre objet `Point`, vous obtiendrez des erreurs de compilation, car le type `Point` ne sait pas comment réagir aux opérateurs `+` et `-`**. Pour qu'un type personnalisé réagisse de manière spécifique aux opérateurs intrinsèques, ==C# fournit le mot-clé `operator`, que vous ne pouvez utiliser qu'avec le mot-clé `static`==. Lorsque vous surchargez un opérateur binaire (tel que `+` et `-`), ==vous passerez généralement deux arguments du même type que la classe définissante== (un `Point` dans cet exemple), comme illustré dans la mise à jour de code suivante :

```cs
namespace OverloadedOps;

// Un type Point plus intelligent.
public class Point
{
	...
	
    // Surcharge de l'opérateur +
    public static Point operator +(Point p1, Point p2) =>
        new Point(p1.X + p2.X, p1.Y + p2.Y);

    // Surcharge de l'opérateur -
    public static Point operator -(Point p1, Point p2) =>
        new Point(p1.X - p2.X, p1.Y - p2.Y);

	...
}
```

**La logique de l'opérateur `+` est simplement de renvoyer un nouvel objet `Point` basé sur la somme des champs des paramètres `Poin`t entrants**. Ainsi, lorsque vous écrivez `pt1 + pt2`, vous pouvez imaginer en interne l'appel suivant à la méthode statique de l'opérateur `+` :

```cs
// Pseudo-code: Point p3 = Point.operator+ (p1, p2)
Point p3 = p1 + p2;
```

De même, p1–p2 correspond à ce qui suit :

```cs
// Pseudo-code: Point p4 = Point.operator- (p1, p2)
Point p4 = p1 - p2;
```

Grâce à cette mise à jour, votre programme se compile désormais et vous pouvez ajouter et soustraire des objets Point, comme le montre la sortie suivante :

```
***** Fun with Overloaded Operators *****

ptOne = [100, 100]
ptTwo = [40, 40]
ptOne + ptTwo : [140, 140]
ptOne - ptTwo : [60, 60]
```

Lors de la surcharge d'un opérateur binaire, **==il n'est pas obligatoire de fournir deux paramètres du même type==**. Si cela s'avère pertinent, **l'un des arguments peut être différent**. Par exemple, voici un opérateur surchargé `+` qui permet d'obtenir un nouveau `Point` basé sur un ajustement numérique :

```cs
public class Point
{
	...
	
    public static Point operator +(Point p1, int change) =>
        new Point(p1.X + change, p1.Y + change);

    public static Point operator +(int change, Point p1) =>
        new Point(p1.X + change, p1.Y + change);

	...
}
```

==Notez que les *deux versions* de la méthode sont nécessaires si vous souhaitez que les arguments soient passés dans n'importe quel ordre==. (Autrement dit, vous ne pouvez pas définir une seule méthode et espérer que le compilateur prenne automatiquement en charge l'autre.) Vous pouvez désormais utiliser ces nouvelles versions de l'opérateur `+` comme suit :

```cs
// Affiche [110, 110].
Point biggerPoint = ptOne + 10;
Console.WriteLine("ptOne + 10 = {0}", biggerPoint);

// Affiche [120, 120].
Console.WriteLine("10 + biggerPoint = {0}", 10 + biggerPoint);
Console.WriteLine();
```

## Qu’en est-il des opérateurs `+=` et `–=` ?

Si vous venez du C++ et que vous découvrez le C#, vous regretterez peut-être la perte de la surcharge des opérateurs d’affectation abrégés (+=, -=, etc.). Rassurez-vous. **En C#, ces opérateurs sont simulés automatiquement si un type surcharge l’opérateur binaire correspondant**. Ainsi, puisque la structure `Point` surcharge déjà les opérateurs `+` et `-`, vous pouvez écrire :

```cs
// La surcharge des opérateurs binaires génère un opérateur raccourci gratuit.
// Opérateur += gratuit
Point ptThree = new Point(90, 5);
Console.WriteLine("ptThree = {0}", ptThree);
Console.WriteLine("ptThree += ptTwo: {0}", ptThree += ptTwo);

// Opérateur -= gratuit
Point ptFour = new Point(0, 500);
Console.WriteLine("ptFour = {0}", ptFour);
Console.WriteLine("ptFour -= ptThree: {0}", ptFour -= ptThree);
```

## Surcharge des opérateurs unaires

C# permet également de surcharger divers opérateurs unaires, tels que `++` et `--`. **Lorsque vous surchargez un opérateur unaire, vous devez également utiliser le mot-clé `static` avec le mot-clé `operator`** ; cependant, ==dans ce cas, il vous suffit de passer un seul paramètre du même type que la classe/structure définissante==. Par exemple, si vous souhaitez mettre à jour l'objet `Point` avec les opérateurs surchargés suivants :

```cs
public class Point
{
	...
	
    // Ajoute 1 aux valeurs X/Y du point entrant
    public static Point operator ++(Point p1) => 
        new Point(p1.X + 1, p1.Y + 1);

    // Soustrait 1 des valeurs X/Y du point entrant.
    public static Point operator --(Point p1) => 
        new Point(p1.X - 1, p1.Y - 1);

	...
}
```

Vous pouvez incrémenter et décrémenter les valeurs x et y du point comme ceci :

```cs
// Applique les opérateurs unaires ++ et -- à un Point.
Point ptFive = new Point(1, 1);
Console.WriteLine("++ptFive = {0}", ++ptFive); // [2, 2]
Console.WriteLine("--ptFive = {0}", --ptFive); // [1, 1]

// Applique les mêmes opérateurs
// pour la post-incrémentation/décrémentation.
Point ptSix = new Point(20, 20);
Console.WriteLine("ptSix++ = {0}", ptSix++); // [20, 20]
Console.WriteLine("ptSix-- = {0}", ptSix--); // [21, 21]
Console.ReadLine();
```

Remarquez que dans l'exemple de code précédent, **vous appliquez les opérateurs personnalisés `++` et `--` de deux manières différentes**. En C++, il est possible de surcharger séparément les opérateurs d'incrémentation/décrémentation avant et après la modification. Ceci n'est pas possible en C#. Cependant, **la valeur de retour de l'incrémentation/décrémentation est automatiquement gérée « correctement » sans frais supplémentaires** (c'est-à-dire que, **==pour un opérateur `++` surchargé, `pt++` a la valeur de l'objet non modifié comme valeur dans une expression, tandis que `++pt` a la nouvelle valeur appliquée avant son utilisation dans l'expression==**).

## Surcharge des opérateurs d'égalité

>[!warning] Cette sous-section est désuète depuis l'introduction des `record` (voir [[Chapitre 5#Égalité des valeurs avec les types d'enregistrements|Chapitre 5]] ainsi que le *callout* au début de la section) 

Comme vous vous en souvenez peut-être du [[Chapitre 6#Redéfinition de `System.Object.Equals()`|Chapitre 6]], ==la méthode `System.Object.Equals()` peut être redéfinie pour effectuer des comparaisons basées sur les valeurs== (plutôt que sur les références) ==entre types référence==. **Si vous choisissez de redéfinir `Equals()`** (et la méthode souvent associée `System.Object.GetHashCode()`), **il est très simple de surcharger les opérateurs d'égalité (`==` et `!=`)**. À titre d'exemple, voici le type `Point` mis à jour :


>[!warning] Le code présenté dans le livre n'est pas une bonne pratique ! C'est un raccourcis pour un but éducatif
> 1. Le piège de la `NullReferenceException`
>
>	Si on écris ce code :
>	
>	```cs
>	Point p1 = null;
>	Point p2 = new Point(1, 1);
>	if (p1 == p2) { ... } // CRASH !
>	```
>	
>	L'opérateur va tenter d'appeler `.Equals()` sur `p1`, qui est `null`. Le programme va planter immédiatement. Un opérateur `==` doit **toujours** être capable de gérer des opérandes nuls sans planter.
>	
> 2. L'inefficacité du `ToString()`
>
>	Comparer deux objets en convertissant tout en chaîne de caractères (`o.ToString() == this.ToString()`) est extrêmement lent et gourmand en mémoire.
>
>	- Cela crée deux nouvelles chaînes de caractères sur le tas (heap) à chaque comparaison.
>	- Si tu tries une liste de 10 000 points, tu vas créer 20 000 chaînes de caractères juste pour comparer des nombres !
>	- **La bonne pratique** : Comparer directement les champs numériques (`this.X == other.X && this.Y == other.Y`).

```cs
public class Point
{

	...
	
    // Les méthodes suivantes sont différentes par rapport au livre
    // Ces méthodes sont plus robustes et plus "pro"
    public override bool Equals(object o)
    {
        if (o is not Point other)
        {
            return false;
        }
        // beaucoup plus rapide que ToString
        return this.X == other.X && this.Y == other.Y;
    }

    public override int GetHashCode()
    {
        // Utilise HashCode.Combine (disponible depuis .NET 5+)
        // C'est la manière la plus performante
        // et sûre d'éviter les collisions.
        return HashCode.Combine(X, Y);
    }

    // Surchargeons maintenant les opérateurs == et !=.
    public static bool operator ==(Point p1, Point p2)
    {
        // Utilise ReferenceEquals pour gérer
        // les cas où les deux sont null
        if (ReferenceEquals(p1, p2))
            return true;

        // Si l'un est null (mais pas les deux, testé au-dessus)
        // alors c'est faux
        if (p1 is null || p2 is null)
            return false;

        return p1.Equals(p2);
    }

    public static bool operator !=(Point p1, Point p2) => !(p1 == p2);
    
    ...
}
```

Compte tenu de cela, vous pouvez maintenant utiliser votre classe `Point` comme suit :

```cs
...
// Utilisation les opérateurs d'égalité surchargés.
Console.WriteLine("ptOne == ptTwo : {0}", ptOne == ptTwo);
Console.WriteLine("ptOne != ptTwo : {0}", ptOne != ptTwo);
```

Comme vous pouvez le constater, il est assez intuitif de comparer deux objets à l'aide des opérateurs bien connus `==` et `!=`, plutôt que d'appeler `Object.Equals()`. **Si vous surchargez les opérateurs d'égalité pour une classe donnée, n'oubliez pas que C# exige que si vous redéfinissez l'opérateur `==`, vous redéfinissiez également l'opérateur `!=`** (si vous l'oubliez, ==le compilateur vous le signalera==).

## Surcharge des opérateurs de comparaison

Au [[Chapitre 8#L'interface `IComparable`|Chapitre 8]], vous avez appris à implémenter l'interface `IComparable` pour comparer la relation entre deux objets similaires. **Vous pouvez également surcharger les opérateurs de comparaison (`<`, `>`, `<=` et` >=`) pour la même classe**. Comme pour les opérateurs d'égalité, ==C# exige que si vous surchargez `<`, vous devez également surcharger `>`. Il en va de même pour les opérateurs `<=` et `>=`==. Si le type `Point` surcharge ces opérateurs de comparaison, l'utilisateur de l'objet pourra alors comparer des `Point`s, comme suit :

```cs
...
// Utilisation des opérateurs < et > surchargés.
Console.WriteLine("ptOne < ptTwo : {0}", ptOne < ptTwo);
Console.WriteLine("ptOne > ptTwo : {0}", ptOne > ptTwo);
```

En supposant que vous ayez implémenté l'interface `IComparable` (ou mieux encore, son équivalent générique), la surcharge des opérateurs de comparaison est triviale. Voici la définition de classe mise à jour :

>[!Attention] 
>L'implémentation de la méthode `CompareTo` à été modifié par rapport au livre car sa logique n'était pas correcte.

```cs
// Point est aussi comparable avec les opérateurs de comparaisons
public class Point : IComparable<Point>
{
	...

    // Utilise la comparaison de tuples :
    // compare X, puis Y si les X sont égaux.
    // C'est l'approche standard (ordre lexicographique)
    // pour garantir un tri cohérent.
    public int CompareTo(Point other) =>
        (this.X, this.Y).CompareTo((other.X, other.Y));

    // Surcharge des opérateur de comparaisons
    public static bool operator <(Point p1, Point p2) => p1.CompareTo(p2) < 0;

    public static bool operator >(Point p1, Point p2) => p1.CompareTo(p2) > 0;

    public static bool operator <=(Point p1, Point p2) => p1.CompareTo(p2) <= 0;

    public static bool operator >=(Point p1, Point p2) => p1.CompareTo(p2) >= 0;
}
```

## Réflexions finales concernant la surcharge d'opérateurs

Comme vous l'avez vu, C# permet de créer des types qui réagissent de manière unique à divers opérateurs intrinsèques et bien connus. ***==Avant de modifier toutes vos classes pour prendre en charge ce comportement, assurez-vous que les opérateurs que vous vous apprêtez à surcharger ont une certaine logique==***.

Par exemple, supposons que vous ayez surchargé l'opérateur de multiplication pour la classe `MiniVan`. Que signifierait exactement la multiplication de deux objets `MiniVan` ? Pas grand-chose. En fait, ==il serait déroutant pour vos collègues de voir l'utilisation suivante d'objets `MiniVan`== :

```cs
// Heuh?! C'est très loin d'être intuitif...
MiniVan newVan = myVan * yourVan;
```

**La surcharge d'opérateurs est généralement utile uniquement lors de la création de types de données atomiques**. *Les vecteurs, les matrices, le texte, les points, les formes, les ensembles*, etc., ==sont de bons candidats pour la surcharge d'opérateurs==. Les personnes, les gestionnaires, les voitures, les connexions aux bases de données et les pages web ne le sont pas. ***==En règle générale, si un opérateur surchargé rend plus difficile la compréhension du fonctionnement d'un type par l'utilisateur, il vaut mieux ne pas l'utiliser. Utilisez cette fonctionnalité avec discernement==***.

>[!example] Dans [[Note annexe au chapitre 11|cette note]], une implémentation de l'interface `INumber<T>` est proposé permettant l'implémentation, entre autre, des surcharges d'opérateurs

# Comprendre les conversions de types personnalisés
 
>[!tip]- En C# moderne, elle est considérée comme une "épée à double tranchant".
>De nos jours, on préfère souvent d'autres approches plus explicites:
>1. **Méthodes d'extension** : `monObjet.ToViewModel()`.
>2. **Constructeurs de copie** : `new ViewModel(monObjet)`.
>3. **Bibliothèques de Mapping** : Comme [AutoMapper](https://automapper.org/), très utilisé en entreprise pour convertir des objets de base de données en objets pour l'interface (DTOs).

Examinons maintenant un sujet étroitement lié à la surcharge d'opérateurs : les conversions de types personnalisées. Pour préparer le terrain, revoyons rapidement la notion de conversions explicites et implicites entre les données numérique et les types de classes associés.

## Rappel : Conversions numériques

==Concernant les types numériques intrinsèques== (`sbyte`, `int`, `float`, etc.), ==une conversion explicite est nécessaire lorsque vous tentez de stocker une valeur plus grande dans un conteneur plus petit, car cela pourrait entraîner une perte de données==. En clair, **c’est votre façon d’indiquer au compilateur : « Ne me touchez pas, je sais ce que je fais »**. À l’inverse, ==une conversion implicite s’effectue automatiquement lorsque vous tentez de placer un type plus petit dans un type de destination qui ne provoquera pas de perte de données==.

```cs
int a = 123;
long b = a;      // Conversion implicite de int en long.
int c = (int) b; // Conversion explicite de long en int.
```

## Rappel : Conversions entre types de classes apparentés

Comme indiqué au [[Chapitre 6#Comprendre les mécanismes fondamentaux de l'héritage|Chapitre 6]], **les types de classes peuvent être liés par héritage classique (relation « est un »)**. Dans ce cas, ==le processus de conversion C# permet de remonter et de descendre dans la hiérarchie des classes==. Par exemple, une classe dérivée peut toujours être convertie implicitement en un type de base. Toutefois, ==si vous souhaitez stocker un type de classe de base dans une variable dérivée, vous devez effectuer une conversion explicite, comme ceci== :

```cs
// Deux types de classes liés.
class Base{}
class Derived : Base{}

// Conversion implicite entre la classe dérivée et la classe de base.
Base myBaseType;
myBaseType = new Derived();

// Conversion explicite nécessaire pour stocker la référence à la classe de base
// dans le type dérivé.
Derived myDerivedType = (Derived)myBaseType;
```

**Cette conversion explicite fonctionne car les classes `Base` et `Derived` sont liées par héritage classique et `myBaseType` est construite comme une instance de `Derived`**. Cependant, ==si `myBaseType` est une instance de `Base`, la conversion lève une exception `InvalidCastException`==. En cas de doute sur l'échec de la conversion, **il est recommandé d'utiliser le mot-clé `as`, comme expliqué au** [[Chapitre 6#Utilisation du mot clé C `as`|Chapitre 6]]. Voici un exemple remanié pour illustrer ceci :

```cs
// Conversion implicite entre la classe dérivée et la classe de base.
Base myBaseType2 = new();

// Lève une exception InvalidCastException
//Derived myDerivedType2 = (Derived)myBaseType2;

// Aucune exception, myDerivedType2 est null
Derived myDerivedType2 = myBaseType2 as Derived;
```

Cependant, ==que se passe-t-il si vous avez deux types de classes dans des hiérarchies différentes, sans parent commun== (autre que `System.Object`) ==et nécessitant des conversions ?== Étant donné qu'ils ne sont pas liés par héritage classique, les opérations de conversion classiques sont inutiles (et vous obtiendriez une erreur de compilation !).

Dans le même ordre d'idées, considérons les types valeur (structures). ==Supposons que vous ayez deux structures nommées `Square` et `Rectangle`==. **Étant donné que les structures ne peuvent pas tirer parti de l'héritage classique== (car elles sont toujours `sealed`), vous n'avez pas de moyen naturel de convertir entre ces types apparemment liés**.

==Bien que vous puissiez créer des méthodes auxiliaires dans les structures== (telles que `Rectangle.ToSquare()`), **C# vous permet de créer des routines de conversion personnalisées qui permettent à vos types de répondre à l'opérateur de conversion `()`**. Par conséquent, **==si vous avez correctement configuré les structures, vous pourrez utiliser la syntaxe suivante pour convertir explicitement entre elles==** :

```cs
// Convertis un Rectangle en en Square! 
Rectangle rect = new Rectangle
{
	Width = 3;
	Height = 10;
}
Square sq = (Square)rect;
```

## Création de routines de conversion personnalisées

Commencez par créer un nouveau projet d'application console nommé *CustomConversions*. **C# propose deux mots-clés, `explicit` et `implicit`, permettant de contrôler le comportement des types lors d'une tentative de conversion**. Supposons que vous ayez les définitions de structure suivantes :

```cs
// Rectangle.cs
namespace CustomConversions;

public struct Rectangle
{
    public int Width { get; set; }
    public int Height { get; set; }

    public Rectangle(int w, int h)
    {
        Width = w;
        Height = h;
    }

    public readonly void Draw()
    {
        for (int i = 0; i < Height; i++)
        {
            for (int j = 0; j < Width; j++)
            {
                Console.Write('*');
            }
            Console.WriteLine();
        }
    }

    public override readonly string ToString() =>
        $"[Width = {Width}; Height = {Height}]";
}
```

```cs
// Square.cs
namespace CustomConversions;

public struct Square
{
    public int Length { get; set; }

    public Square(int l)
        : this()
    {
        Length = l;
    }

    public readonly void Draw()
    {
        for (int i = 0; i < Length; i++)
        {
            for (int j = 0; j < Length; j++)
            {
                Console.Write('*');
            }
            Console.WriteLine();
        }
    }

    public override readonly string ToString() => $"[Length = {Length}]";

    // Les objets Rectangle peuvent être convertit
    // explicitement en des objet Square
    public static explicit operator Square(Rectangle r)
    {
        return new Square { Length = r.Height };
    }
}
```

==Notez que cette itération du type `Square` définit un opérateur de conversion explicite==. À l'instar du processus de surcharge d'un opérateur, **les routines de conversion utilisent le mot-clé `operator` de C#, conjointement avec les mots-clés `explicit` ou `implicit`, et doivent être définies comme `static`**. ==Le paramètre d'entrée est l'entité à partir de laquelle vous effectuez la conversion, tandis que le type de l'opérateur est l'entité de *destination*.==

Dans ce cas, on suppose qu'un carré (figure géométrique dont tous les côtés sont de même longueur) peut être obtenu à partir de la hauteur d'un rectangle. Ainsi, vous pouvez convertir un rectangle en carré comme suit :

```cs
using CustomConversions;

Console.Title = "Fun with Conversions";
Console.WriteLine("***** Fun with Conversions *****\n");

// Crée un Rectangle
Rectangle r = new(15, 4);
Console.WriteLine(r.ToString());
r.Draw();

Console.WriteLine();

// Convertit r en en Square
// basé sur la propriété Height de l'objet Rectangle.
Square s = (Square)r;
Console.WriteLine(s.ToString());
s.Draw();

Console.ReadLine();

```

Vous pouvez voir le résultat ici :

```
***** Fun with Conversions *****

[Width = 15; Height = 4]
***************
***************
***************
***************

[Length = 4]
****
****
****
****
```

Bien qu'il ne soit pas forcément très utile de convertir un `Rectangle` en `Square` dans le même contexte, supposons que vous ayez une fonction conçue pour prendre des paramètres de type `Square`.

```cs
// Cette méthode requiert un objet de type Square.
static void DrawSquare(Square sq)
{
    Console.WriteLine(sq.ToString());
    sq.Draw();
}
```

**En utilisant votre opération de conversion explicite sur le type `Square`, vous pouvez maintenant transmettre des types `Rectangle` pour le traitement à l'aide d'un cast explicite**, comme ceci :

```cs
...
// Convertit un Rectangle en un Square pour invoquer la méthode
Rectangle rect = new Rectangle(10, 5);
DrawSquare((Square)rect);
Console.ReadLine();
```

## Conversions explicites supplémentaires pour le type `Square` 

Maintenant que vous pouvez convertir explicitement des rectangles en carrés, examinons quelques conversions explicites supplémentaires. Étant donné qu'un carré est symétrique, il peut être utile de fournir une routine de conversion explicite permettant à l'appelant de convertir un entier en carré (dont la longueur du côté sera, bien sûr, égale à l'entier d'entrée). De même, que diriez-vous de mettre à jour la classe `Square` afin l'appelant puisse convertir un carré en entier ? Voici la logique d'appel :

```cs
...
// Convertit un int en Square
Square sq2 = (Square)90;
Console.WriteLine($"sq2 = {sq2}");

// Convertit un Square vers un int.
int side = (int)sq2;
Console.WriteLine($"Side length of sp2 = {side}");
```

Voici la mise à jour de la classe `Square` :

```cs
namespace CustomConversions;

public struct Square
{
	...
	
    public static explicit operator Square(int sideLength) =>
        new Square { Length = sideLength };

    public static explicit operator int(Square s) => s.Length;
}
```

Honnêtement, ==convertir un `Square` en entier n'est peut-être pas l'opération la plus intuitive== (ni la plus utile. Après tout, il y a de fortes chances que vous puissiez simplement passer ces valeurs à un constructeur). Cependant, **cela met en évidence un point important concernant les routines de conversion personnalisées : le compilateur se fiche de ce que vous convertissez depuis ou vers quoi vous convertissez, si votre code est syntaxiquement correct**.

Ainsi, comme pour la surcharge d'opérateurs, ==ce n'est pas parce que vous pouvez créer une opération de conversion explicite pour un type donné que vous *devriez* le faire==. **Généralement, cette technique sera surtout utile lors de la création de types de structure, étant donné qu'ils ne peuvent pas participer à l'héritage classique** (où la conversion est automatique).

## Définition des routines de conversion implicite

Jusqu'à présent, vous avez créé diverses opérations de conversion *explicites* personnalisées. Mais qu'en est-il de la conversion *implicite* suivante ?

```cs
...
Square s3 = new Square {Longueur = 83};

// Tentative de conversion implicite ?
Rectangle rect2 = s3;
```

**Ce code ne compilera pas, car vous n'avez pas fourni de routine de conversion implicite pour le type `Rectangle`**. Voici le problème : **==il est interdit de définir des fonctions de conversion explicites et implicites sur le même type si elles ne diffèrent ni par leur type de retour ni par leurs paramètres==**. Cela peut sembler une limitation ; cependant, ==le second problème est que lorsqu'un type définit une routine de conversion *implicite*, l'appelant peut utiliser la syntaxe de conversion explicite !==

Vous êtes perdu ? Pour y voir plus clair, ajoutons une routine de conversion implicite à la structure `Rectangle` en utilisant le mot-clé `implicit` de C# (notez que le code suivant suppose que la largeur du `Rectangle` résultant est calculée en multipliant le côté du `Square` par 2) :

```cs
namespace CustomConversions;

public struct Rectangle
{
	...
	
    public static implicit operator Rectangle(Square s)
    {
      return new Rectangle {
        Height = s.Length,
        Width = s.Length * 2 // Supposons que la longueur du nouveau rectangle soit (Longueur x 2).
      };
    }
}
```

Grâce à cette mise à jour, vous pouvez désormais convertir entre les types, comme suit :

```cs
// Conversion implicite OK!
Square s3 = new Square { Length = 7 };
Rectangle rect2 = s3;
Console.WriteLine($"rect 2 = {rect2}");

// Syntaxe de conversion explicite toujours OK!
Square s4 = new Square { Length = 3 };
Rectangle rect3 = (Rectangle)s4;
Console.WriteLine($"rect3 = {rect3}");

Console.ReadLine();
```

Ceci conclut notre présentation de la définition des routines de conversion personnalisées. Comme pour les opérateurs surchargés, **n'oubliez pas que cette syntaxe est simplement une notation abrégée pour les fonctions membres « normales », et qu'à ce titre, elle est toujours facultative**. Cependant, ==utilisées correctement, les structures personnalisées s'intègrent plus naturellement, car elles peuvent être traitées comme de véritables types de classes liés par héritage==.

# Comprendre les méthodes d'extension

.NET 3.5 a introduit le concept de *méthodes d'extension*, qui **permettent d'ajouter de nouvelles méthodes ou propriétés à une classe ou une structure, sans modifier directement le type d'origine**. Dans quel cas cela peut-il être utile ? Prenons quelques exemples.

Supposons que vous ayez une classe en production. ==Avec le temps, il devient évident que cette classe doit prendre en charge de nouveaux membres==. **Si vous modifiez directement la définition de la classe, vous risquez de rompre la compatibilité ascendante avec les anciennes bases de code qui l'utilisent, car elles n'ont peut-être pas été compilées avec la dernière version de la définition de classe**. Une solution pour garantir la compatibilité ascendante consiste à créer une nouvelle classe dérivée de la classe parente existante; cependant, vous avez alors deux classes à maintenir. **Comme chacun sait, la maintenance du code est la partie la moins attrayante du métier d'ingénieur logiciel.**

Considérons maintenant cette situation. ==Supposons que vous ayez une structure== (ou peut-être une classe scellée) ==et que vous souhaitiez ajouter de nouveaux membres pour qu'elle se comporte de manière polymorphe dans votre système==. **Comme les structures et les classes scellées ne peuvent pas être étendues, votre seul choix est d'ajouter les membres au type, au risque une fois de plus de rompre la compatibilité ascendante**.

***==Grâce aux méthodes d'extension, vous pouvez modifier les types sans créer de sous-classe et sans modifier le type directement. Le hic, c'est que la nouvelle fonctionnalité n'est offerte à un type que si les méthodes d'extension ont été référencées pour être utilisées dans votre projet actuel==***.

## Définition des méthodes d'extension

**Lorsque vous définissez des méthodes d'extension, la première contrainte est qu'elles doivent être définies dans une classe statique** (voir [[Chapitre 5#Comprendre le mot-clé `static`|Chapitre 5]]); par conséquent, ==chaque méthode d'extension doit être déclarée avec le mot-clé `static`.==

La seconde contrainte est que **toutes les méthodes d'extension sont marquées comme telles en utilisant le mot-clé `this` comme modificateur sur le premier (et uniquement le premier) paramètre de la méthode en question**. ==Le paramètre qualifié « this » représente l'élément étendue.==

Pour illustrer cela, créez un nouveau projet d'application console nommé *ExtensionMethods*. Supposons maintenant que vous écriviez une classe nommée `MyExtensions` qui définit deux méthodes d'extension. La première méthode permet à tout objet d'utiliser une nouvelle méthode nommée `DisplayDefiningAssembly()` qui utilise des types de l'espace de noms de réflexion du système pour afficher le nom de l'assembly contenant le type en question.

>[!note]
>Vous étudierez formellement l'API de réflexion au [[Chapitre 17|Chapitre 17]]. Si vous découvrez le sujet, retenez simplement que la **réflexion vous permet de découvrir la structure des assemblys, des types et des membres de type lors de l'exécution.**

La seconde méthode d'extension, nommée `ReverseDigits()`, permet à tout entier d'obtenir une nouvelle version de lui-même où sa valeur est inversée chiffre par chiffre. Par exemple, si un entier de valeur $1234$ appelle `ReverseDigits()`, l'entier retourné aura la valeur $4321$. Considérez l'implémentation de classe suivante (==veillez à importer l'espace de noms `System.Reflection` si vous suivez cet exemple)== :

```cs
using System.Reflection;

namespace ExtensionMethods;

static class MyExtensions
{
    // Cette méthode permet à tout objet d'afficher l'assemblage
    // dans lequel il est défini.
    public static void DisplayDefiningAssembly(this object obj)
    {
        Console.WriteLine(
            $"{obj.GetType().Name} lives here: => {Assembly.GetAssembly(obj.GetType()).GetName().Name}\n"
        );
    }

    // Cette méthode permet d'inverser les chiffres de n'importe quel entier.
    // Par exemple, 56 renverra 65.
    public static int ReverseDigits(this int i)
    {
        // Convertir l'entier en chaîne de caractères, puis
        // récupérer tous les caractères.
        char[] digits = i.ToString().ToCharArray();

        // On inverse les objet dans le tableau
        Array.Reverse(digits);

        // On remet dans un string
        string newDigits = new string(digits);

        // Enfin, renvoyez la chaîne modifiée sous forme d'entier.
        return int.Parse(newDigits);
    }
}
```

>[!tip]-
> La méthode d'extension `ReverseDigits()` présenté est très pédagogique mais très inefficace!
> 
> L'approche plus performante la plus proche à celle du livre est la suivante:
> ```cs
> public static int ReverseDigits(this int i)
>{
>	// On travaille sur la pile (stack) avec Span
>	Span<char> s = stackalloc char[11]; // Un int a max 10 chiffres + signe
>	if (!i.TryFormat(s, out int charsWritten)) return 0;
>
>	Span<char> actualDigits = s.Slice(0, charsWritten);
>	actualDigits.Reverse();
>
>	return int.Parse(actualDigits);
>}
>```
>
> **L'approche la plus performantes est algorithmique:**
>
> ```cs
> public static int ReverseDigits(this int i)
>{
>	int reverse = 0;
>	while (i > 0)
>	{
>		// On prend le dernier chiffre (modulo) et on l'ajoute au résultat
>		reverse = (reverse * 10) + (i % 10);
>		i /= 10;
>	}
>	return reverse;
>}
>```

**Notez à nouveau que le premier paramètre de chaque méthode d'extension est qualifié par le mot-clé `this`, avant de définir son type**. **==Le premier paramètre d'une méthode d'extension représente toujours le type étendu==**. ==Étant donné que `DisplayDefiningAssembly()` a été prototypée pour étendre `System.Object`, chaque type possède désormais ce nouveau membre, car `Object` est le type parent de tous les types sur la plateforme .NET==. Cependant, ==`ReverseDigits()` a été prototypée pour étendre uniquement les types entiers; par conséquent, si un type autre qu'un entier tente d'appeler cette méthode, une erreur de compilation se produira==.

>[!note]
>Il faut comprendre qu'une méthode d'extension donnée peut avoir plusieurs paramètres, mais **seul le premier paramètre peut être qualifié avec ceci**. Les paramètres supplémentaires seraient traités comme des paramètres d'entrée normaux pour être utilisés par la méthode.

## Appel des méthodes d'extension

Maintenant que ces méthodes d'extension sont en place, considérez l'exemple de code suivant qui applique la méthode d'extension à différents types dans les bibliothèques de classes de base :

```cs
using MyExtensionMethods;

Console.Title = "Fun with Extension Methods";
Console.WriteLine("***** Fun with Extension Methods *****\n");

// Le type entier a pris une nouvelle identité !
int myInt = 12345678;
myInt.DisplayDefiningAssembly();

// Le type DataSet aussi !
System.Data.DataSet d = new System.Data.DataSet();
d.DisplayDefiningAssembly();

// Utilisation de la nouvelle fonctinnalité des entiers
Console.WriteLine($"Value of myInt: {myInt}");
Console.WriteLine($"Reversed digits of myInt: {myInt.ReverseDigits()}");

Console.ReadLine();
```

Voici le résultat :

```
***** Fun with Extension Methods *****

Int32 lives here: => System.Private.CoreLib

DataSet lives here: => System.Data.Common

Value of myInt: 12345678
Reversed digits of myInt: 87654321
```

## Importation des méthodes d'extension

==Lorsque vous définissez une classe contenant des méthodes d'extension, elle sera sans aucun doute définie dans un espace de noms==. **Si cet espace de noms est différent de celui utilisant les méthodes d'extension, vous devrez utiliser le mot-clé C# attendu `using`**. Ainsi, votre fichier de code aura accès à toutes les méthodes d'extension du type étendu. **Il est important de s'en souvenir car si vous n'importez pas explicitement le bon espace de noms, les méthodes d'extension ne seront pas disponibles pour ce fichier de code C#**.

En effet, bien qu'il puisse sembler à première vue que les méthodes d'extension soient globales, **==elles sont en réalité limitées aux espaces de noms qui les définissent ou à ceux qui les importent==**. Rappelez-vous que vous avez encapsulé la classe `MyExtensions` dans un espace de noms nommé `MyExtensionMethods`, comme suit :

```cs
namespace MyExtensionMethods;
static class MyExtensions
{
	...
}
```

**Pour utiliser les méthodes d'extension de la classe, vous devez importer explicitement l'espace de noms `MyExtensionMethods` , comme nous l'avons fait dans les instructions de niveau supérieur utilisées pour exécuter les exemples.**

## Extension de types implémentant des interfaces spécifiques

Vous avez vu jusqu'ici comment étendre des classes (et, indirectement, des structures suivant la même syntaxe) avec de nouvelles fonctionnalités via des méthodes d'extension. **Il est également possible de définir une méthode d'extension qui ne peut étendre qu'une classe ou une structure implémentant l'interface appropriée**. Par exemple, vous pourriez déclarer : « ==Si une classe ou une structure implémente `IEnumerable<T>`, alors ce type reçoit les nouveaux membres suivants==. » Bien sûr, il est possible d'exiger qu'un type prenne en charge n'importe quelle interface, y compris vos propres interfaces personnalisées.

Pour illustrer cela, créez un nouveau projet d'application console nommé *InterfaceExtensions*. **L'objectif est d'ajouter une nouvelle méthode à tout type implémentant `IEnumerable`, ce qui inclut tout tableau et de nombreuses classes de collections non génériques** (rappelons du [[Chapitre 10#Tableau 10-4 Interfaces clés prises en charge par les classes de `System.Collections.Generic`|Chapitre 10]] que l'interface générique `IEnumerable<T>` étend l'interface non générique `IEnumerable`). Ajoutez la classe d'extension suivante à votre nouveau projet :

```cs
namespace IntefaceExtensions;

static class AnnoyingExtensions
{
    public static void PrintDataAndBeep(
        this System.Collections.IEnumerable iterator
    )
    {
        foreach (var item in iterator)
        {
            Console.WriteLine(item);
            Console.Beep();
        }
    }
}
```

**Étant donné que la méthode `PrintDataAndBeep()` peut être utilisée par toute classe ou structure implémentant `IEnumerable`, vous pouvez la tester avec le code suivant :**

```cs
using IntefaceExtensions;

Console.Title = "Extending Interface Compatible types";
Console.WriteLine("***** Extending Interface Compatible types *****\n");

// System.Array implémente IEnumerable!
// Syntaxe C# 12 (collection expression)
string[] data =
[
    "wow",
    "this",
    "is",
    "sort",
    "of",
    "annoying",
    "but",
    "in",
    "a",
    "weird",
    "way",
    "fun!",
];
data.PrintDataAndBeep();

// List<T> implémente IEnumerable!
// Syntaxe C# 12 (collection expression)
List<int> myInts = [10, 15, 20];
myInts.PrintDataAndBeep();

Console.ReadLine();
```

>[!warning] Sur MacOS, c'est complètement normal si on entend qu'un seul bip, c'est lié à comment le système gère les alertes sonore (notifications).

Ceci conclut votre étude des méthodes d'extension C#. N'oubliez pas que **cette fonctionnalité du langage peut s'avérer utile chaque fois que vous souhaitez étendre les fonctionnalités d'un type sans créer de sous-classe** (ou sans possibilité de sous-classe si le type est scellé), **dans le cadre du polymorphisme**. Comme vous le verrez plus loin, **==les méthodes d'extension jouent un rôle clé pour les API LINQ. En effet, vous constaterez que, dans les API LINQ, l'un des éléments les plus fréquemment étendus est une classe ou une structure implémentant (surprise !) la version générique de `IEnumerable`==**.

## Prise en charge de la méthode d'extension `GetEnumerator` (Nouveauté C# 9.0)

==Avant C# 9.0, pour utiliser `foreach` sur une classe, la méthode `GetEnumerator()` devait être définie directement dans cette classe==. **Avec C# 9.0, la méthode `foreach` examine les méthodes d'extension de la classe et, si une méthode `GetEnumerator()` est trouvée, elle l'utilise pour obtenir l'`IEnumerator` de cette classe**. Pour observer ce comportement, créez une application console nommée *ForEachWithExtensionMethods* et ajoutez-y des versions simplifiées des classes `Car` et `Garage` du [[Chapitre 8#Les interfaces `IEnumerable` et `IEnumerator`|Chapitre 8]].

```cs
// Car.cs
namespace ForEachWithExtensionMethods;

class Car
{
    // Propriétés de Car
    public int CurrentSpeed { get; set; } = 0;
    public string PetName { get; set; } = "";

    // Constructeurs
    public Car() { }

    public Car(string name, int speed)
    {
        CurrentSpeed = speed;
        PetName = name;
    }
}
```

```cs
// Garage.cs
namespace ForEachWithExtensionMethods;

class Garage
{
    public Car[] CarsInGarage { get; set; }

    public Garage()
    {
        CarsInGarage = new Car[4];
        CarsInGarage[0] = new Car("Rusty", 30);
        CarsInGarage[1] = new Car("Clunker", 55);
        CarsInGarage[2] = new Car("Zippy", 30);
        CarsInGarage[3] = new Car("Fred", 30);
    }
}
```

Notez que la classe `Garage` n'implémente pas `IEnumerable` et ne possède pas de méthode `GetEnumerator()`. La méthode `GetEnumerator()` est ajoutée par la classe `GarageExtensions`, comme illustré ici :

```cs
using System.Collections;

namespace ForEachWithExtensionMethods;

static class GarageExtensions
{
    public static IEnumerator GetEnumerator(this Garage g) =>
        g.CarsInGarage.GetEnumerator();
}
```

Le code permettant de tester cette nouvelle fonctionnalité est identique à celui utilisé pour tester la méthode `GetEnumerator()` au [[Chapitre 8#Les interfaces `IEnumerable` et `IEnumerator`|Chapitre 8]]. Mettez à jour le fichier *Program.cs* comme suit :

```cs
using ForEachWithExtensionMethods;

Console.Title = "Support for Extension Method GetEnumerator";
Console.WriteLine("***** Support for Extension Method GetEnumerator *****\n");

Garage carLot = new Garage();

// Donner chaque voiture de la collection ?
foreach (Car c in carLot)
{
    Console.WriteLine("{0} is going {1} km/h", c.PetName, c.CurrentSpeed);
}
```

Vous constaterez que le code fonctionne, affichant dans la console la liste des voitures et leur vitesse.

```
***** Support for Extension Method GetEnumerator *****

Rusty is going 30 km/h
Clunker is going 55 km/h
Zippy is going 30 km/h
Fred is going 30 km/h
```

>[!note]
>Cette nouvelle fonctionnalité présente un inconvénient potentiel : des classes qui n’étaient pas censées être accessibles par `foreach` peuvent désormais l’être.

# Comprendre les types anonymes

>[!info]- L'utilisation moderne des types anonymes
>Les types anonymes sont très utilisés pour tout ce qui va toucher à l'API LINQ ([[Chapitre 13#Types anonymes|Chapitre13]]. Pout les autres usage, les tuples ([[Chapitre 4#Comprendre les tuples (Nouveauté / MaJ C 7.0)|Chapitre 4]]) et les records ([[Chapitre 5#Le type de donnée `record` (Nouveauté C 9.0)|Chapitre 5]]) sont préférés pour transférer des données temporaires car les types anonymes ont deux gros défaut:
>
>- **Portée locale uniquement** : Vous ne pouvez pas retourner un type anonyme d'une méthode (la méthode devrait retourner `object`, et vous perdriez tout l'intérêt du typage).
>- **Lecture seule** : Ils sont immuables (ce qui est bien), mais moins flexibles que d'autres structures.


En tant que programmeur orienté objet, vous connaissez les avantages de définir des classes pour représenter l'état et les fonctionnalités d'un élément donné que vous cherchez à modéliser. En effet, lorsqu'il s'agit de définir une classe destinée à être réutilisée dans différents projets et qui offre de nombreuses fonctionnalités via un ensemble de méthodes, d'événements, de propriétés et de constructeurs personnalisés, la création d'une nouvelle classe C# est une pratique courante.

Cependant, **il arrive que l'on souhaite définir une classe simplement pour modéliser un ensemble de points de données encapsulées** (et liés d'une manière ou d'une autre), **sans méthodes, événements ou autres fonctionnalités spécialisées**. De plus, **que se passe-t-il si ce type n'est utilisé que par quelques méthodes de votre programme ? Définir une classe complète, comme illustré ci-après, serait plutôt fastidieux si l'on sait pertinemment qu'elle ne sera utilisée qu'à quelques endroits**. Pour illustrer ce point, voici ==un aperçu des étapes à suivre pour créer un type de données « simple » suivant une sémantique classique basée sur les valeurs== :

```cs
class SomeClass
{ 
	// Définir un ensemble de variables membres privées...

	// Créer une propriété pour chaque variable membre...
	
	// Redéfinir ToString() pour gérer les variables membres clés...
	
	// Redéfinir GetHashCode() et Equals() pour gérer l'égalité basée sur les valeurs...
}
```

Comme vous pouvez le constater, ce n'est pas forcément si simple. ==Non seulement vous devez écrire une quantité conséquente de code, mais vous devez également gérer une classe supplémentaire dans votre système==. **Pour des données temporaires comme celles-ci, il serait utile de créer rapidement un type de données personnalisé**. Par exemple, supposons que vous deviez créer une méthode personnalisée qui reçoit un ensemble de paramètres. Vous souhaitez utiliser ces paramètres pour créer un nouveau type de données utilisable dans la portée de cette méthode. De plus, vous souhaitez afficher rapidement ces données à l'aide de la méthode `ToString()` classique et peut-être utiliser d'autres membres de `System.Object`. Vous pouvez faire exactement cela en utilisant la syntaxe des types anonymes.

## Définition d'un type anonyme

**Lorsque vous définissez un type anonyme, vous utilisez le mot-clé `var`** (voir [[Chapitre 3#Comprendre l'utilité des variables locales implicitement typées|Chapitre 3]]) **conjointement avec la syntaxe d'initialisation d'objet** (voir [[Chapitre 5#Comprendre l'initialisation des objets|Chapitre 5]]). **L'utilisation du mot-clé `var` est indispensable car le compilateur génère automatiquement une nouvelle définition de classe lors de la compilation** (et vous ne verrez jamais le nom de cette classe dans votre code C#). ==La syntaxe d'initialisation permet d'indiquer au compilateur de créer des champs privés et des propriétés== (en lecture seule) ==pour le type nouvellement créé.==

À titre d'exemple, créez un nouveau projet d'application console nommé *AnonymousTypes*. Ajoutez ensuite la méthode suivante à votre fichier `Program.cs`, qui crée un nouveau type à la volée à partir des données de paramètres reçues :

```cs
static void BuildAnonymousType(string make, string color, int currSp)
{
    // Construit un type anonyme à partir des arguments entrants.
    var car = new
    {
        Make = make,
        Color = color,
        Speed = currSp,
    };

    // Notez que vous pouvez maintenant utiliser ce type
    // pour accéder aux données des propriétés !
    Console.WriteLine(
        "You have a {0} {1} going {2} km/h",
        car.Color,
        car.Make,
        car.Speed
    );

    // Les types anonymes possèdent des implémentations
    // personnalisées de chaque méthode virtuelle
    // de System.Object. Par exemple :
    Console.WriteLine("ToString() == {0}", car.ToString());
}
```

**Notez qu'un type anonyme peut également être créé directement dans le code, en plus de l'encapsuler dans une fonction**, comme illustré ici :

```cs
Console.Title = "Fun with Anonymous types";
Console.WriteLine("***** Fun with Anonymous types *****\n");

// Créer un type anonyme représentant une voiture.
var myCar = new
{
    Color = "Bright Pink",
    Make = "Saab",
    CurrentSpeed = 55,
};

// Affiche maintentant la couleurs et le fabricant
Console.WriteLine($"My car is a {myCar.Color} {myCar.Make}");

// Appelle maintenant notre méthode d'assistance
// pour construire un type anonyme via les arguments.
BuildAnonymousType("BMW", "Black", 90);

Console.ReadLine();
```

À ce stade, **retenez simplement que les types anonymes permettent de modéliser rapidement la « structure » ​​des données avec un minimum de surcharge**. Cette technique consiste simplement à ==créer à la volée un nouveau type de données, qui prend en charge une encapsulation minimale via des propriétés et se comporte selon une sémantique basée sur les valeurs==. Pour comprendre ce dernier point, **voyons comment le compilateur C# construit les types anonymes à la compilation et, plus précisément, comment il redéfinit les membres de `System.Object`**.

## La représentation interne des types anonymes

**Tous les types anonymes sont automatiquement dérivés de `System.Object` et prennent donc en charge chacun des membres fournis par cette classe de base**. Vous pouvez ainsi appeler `ToString()`, `GetHashCode()`, `Equals()`, ou `GetType()` sur l'objet `myCar`, dont le typage est implicite. Supposons que votre fichier *Program.cs* définisse la fonction d'assistance statique suivante :

```cs
static void ReflectOverAnonymousType(object obj)
{
    Console.WriteLine($"obj is an instance of: {obj.GetType().Name}");
    Console.WriteLine(
        $"Base class of {obj.GetType().Name} is {obj.GetType().BaseType}"
    );
    Console.WriteLine($"obj.ToString() == {obj.ToString()}");
    Console.WriteLine($"obj.GetHashCode() == {obj.GetHashCode()}");
    Console.WriteLine();
}
```

Supposons maintenant que vous invoquiez cette méthode en passant l'objet `myCar` comme paramètre, comme ceci :

```cs
Console.Title = "Fun with Anonymous types";
Console.WriteLine("***** Fun with Anonymous types *****\n");

// Créer un type anonyme représentant une voiture.
var myCar = new
{
    Color = "Bright Pink",
    Make = "Saab",
    CurrentSpeed = 55,
};

// Réflète ce que le compilateur a généré.
ReflectOverAnonymousType(myCar);

...

Console.ReadLine();
```

Le résultat ressemblera à ceci :

```
***** Fun with Anonymous types *****

obj is an instance of: <>f__AnonymousType0`3
Base class of <>f__AnonymousType0`3 is System.Object
obj.ToString() == { Color = Bright Pink, Make = Saab, CurrentSpeed = 55 }
obj.GetHashCode() == -1467690625
```

Tout d'abord, notez que, dans cet exemple, l'objet `myCar` est de type `<>f__AnonymousType...` (votre nom peut être différent). ==N'oubliez pas que le nom de type attribué est entièrement déterminé par le compilateur et n'est pas directement accessible dans votre code C#.==

Plus important encore, **notez que chaque paire nom-valeur définie à l'aide de la syntaxe d'initialisation d'objet est associée à une propriété en lecture seule portant le même nom et à un champ de stockage privé correspondant, réservé à l'initialisation**. Le code C# suivant illustre **==approximativement**== la classe générée par le compilateur pour représenter l'objet `myCar` (que vous pouvez vérifier à l'aide d'*ildasm.exe)* :

>[!info] Le code suivant à été fait à la main par l'auteur du livre pour une question de pédagogie.

```CIL
.class private sealed '<>f__AnonymousType0'3'<'<Color>j__TPar',
  '<Make>j__TPar', <CurrentSpeed>j__TPar>'
  extends [System.Runtime][System.Object]
{
	// init-only fields.
	private initonly <Color>j__TPar <Color>i__Field;
	private initonly <CurrentSpeed>j__TPar <CurrentSpeed>i__Field;
	private initonly <Make>j__TPar <Make>i__Field;
	
	// Default constructor.
	public <>f__AnonymousType0(<Color>j__TPar Color,
	<Make>j__TPar Make, <CurrentSpeed>j__TPar CurrentSpeed);
	
	// Overridden methods.
	public override bool Equals(object value);
	public override int GetHashCode();
	public override string ToString();
	
	// Read-only properties.
	<Color>j__TPar Color { get; }
	<CurrentSpeed>j__TPar CurrentSpeed { get; }
	<Make>j__TPar Make { get; }
}
```

## Implémentation de `ToString()` et `GetHashCode()`

**Tous les types anonymes héritent automatiquement de `System.Object` et disposent d'une version redéfinie des méthodes `Equals()`, `GetHashCode()` et `ToString()`**. L'implémentation de `ToString()` construit simplement une chaîne de caractères à partir de chaque paire nom-valeur. Voici un exemple :

```cs
public override string ToString()
{
	StringBuilder builder = new StringBuilder();
	builder.Append("{ Color = ");
	builder.Append(this.<Color>i__Field);
	builder.Append(", Make = ");
	builder.Append(this.<Make>i__Field);
	builder.Append(", CurrentSpeed = ");
	builder.Append(this.<CurrentSpeed>i__Field);
	builder.Append(" }");
	return builder.ToString();
}
```

>[!tip] Le code du dessus est la réelle implémentation qu'utilisait le compilateur C# avant .NET 5.
>Le compilateur utilise maintenant des appels à `System.Object.ToString()` avec un appel final à `String.Format()`.

**L'implémentation de `GetHashCode()` calcule une valeur de hachage en utilisant les variables membres de chaque type anonyme comme entrée du type `System.Collections.Generic.EqualityComparer<T>`. Grâce à cette implémentation de `GetHashCode()`, deux types anonymes produiront la même valeur de hachage s'ils possèdent le même ensemble de propriétés auxquelles ont été attribuées les mêmes valeurs**. De ce fait, les types anonymes sont parfaitement adaptés à une utilisation dans un conteneur `Hashtable`

>[!tldr] TLDR plus précision moderne
>Deux objets identiques dans une même session d'exécution auront le même HashCode. Cependant, **ce HashCode peut être différent d'une exécution à l'autre (redémarrage du programme)** ou d'une version de .NET à l'autre. Microsoft a introduit ce comportement (non-déterminisme) pour des raisons de sécurité afin de prévenir les attaques par collision de hashs.

## Sémantique de l'égalité pour les types anonymes

Bien que l'implémentation des méthodes `ToString()` et `GetHashCode()` surchargées soit simple, vous vous demandez peut-être ==comment la méthode `Equals()` a été implémentée==. Par exemple, si vous définissiez deux variables « voitures anonymes » spécifiant les mêmes paires nom-valeur, ces deux variables seraient-elles considérées comme égales ? Pour observer les résultats directement, mettez à jour votre classe *Program.cs* avec la nouvelle méthode suivante :

```cs
static void EqualityTest()
{
    // Créez 2 classes anonymes avec des paires nom/valeur identiques.
    var firstCar = new
    {
        Color = "Bright Pink",
        Make = "Saab",

        CurrentSpeed = 55,
    };

    var secondCar = new
    {
        Color = "Bright Pink",
        Make = "Saab",

        CurrentSpeed = 55,
    };

	Console.WriteLine();
	
    // Sont-ils considérés comme égaux lors de l'utilisation de Equals() ?
    if (firstCar.Equals(secondCar))
    {
        Console.WriteLine("Same anonymous object!");
    }
    else
    {
        Console.WriteLine("Not the same anonymous object!");
    }
    
    // Sont-ils considérés comme égaux lorsqu'on utilise == ?
    if (firstCar == secondCar)
    {
        Console.WriteLine("Same anonymous object!");
    }
    else
    {
        Console.WriteLine("Not the same anonymous object!");
    }
    
    // Ces objets sont-ils du même type sous-jacent ?
    if (firstCar.GetType().Name == secondCar.GetType().Name)
    {
        Console.WriteLine("We are both the same type!");
    }
    else
    {
        Console.WriteLine("We are different types!");
    }
    
    // Affiche tous les détails.
    Console.WriteLine();
    ReflectOverAnonymousType(firstCar);
    ReflectOverAnonymousType(secondCar);
}
```

Lorsque vous appelez cette méthode, le résultat peut être quelque peu surprenant.

```
...

My car is a Bright Pink Saab
You have a Black BMW going 90 km/h
ToString() == { Make = BMW, Color = Black, Speed = 90 }

Same anonymous object!
Not the same anonymous object!
We are both the same type!

obj is an instance of: <>f__AnonymousType0`3
Base class of <>f__AnonymousType0`3 is System.Object
obj.ToString() == { Color = Bright Pink, Make = Saab, CurrentSpeed = 55 }
obj.GetHashCode() == 1789772488

obj is an instance of: <>f__AnonymousType0`3
Base class of <>f__AnonymousType0`3 is System.Object
obj.ToString() == { Color = Bright Pink, Make = Saab, CurrentSpeed = 55 }
obj.GetHashCode() == 1789772488
```

Lorsque vous exécuterez ce code de test, ==vous constaterez que le premier test conditionnel, où vous appelez `Equals()`, renvoie `true`== et, par conséquent, le message « Même objet anonyme ! » s'affiche à l'écran. Cela s'explique par le fait que **la méthode `Equals()` générée par le compilateur utilise une sémantique basée sur les valeurs lors du test d'égalité** (par exemple, en vérifiant la valeur de chaque champ des objets comparés).

Cependant, ==le second test conditionnel, qui utilise l'opérateur d'égalité C# (`==`), renvoie `false`.== Cela peut sembler contre-intuitif au premier abord. **Ce résultat s'explique par le fait que les types anonymes ne reçoivent pas de versions surchargées des opérateurs d'égalité C# (`==` et `!=`)**. Par conséquent, lorsque vous testez l'égalité de types anonymes à l'aide des opérateurs d'égalité C# (plutôt que de la méthode `Equals()`), ==ce sont les références, et non les valeurs conservées par les objets, qui sont testées.==

Enfin, ==dans le dernier test conditionnel== (où vous examinez le nom du type sous-jacent), ==vous constatez que les types anonymes sont des instances du même type de classe généré par le compilateur== (dans cet exemple, `<>f AnonymousType...`) **car `firstCar` et `secondCar` ont les mêmes propriétés (`Color`, `Make` et `CurrentSpeed`).**

Ceci illustre un point important, bien que subtil : **le compilateur ne génère une nouvelle définition de classe que lorsqu'un type anonyme contient des noms uniques**. Ainsi, **==si vous déclarez des types anonymes identiques==** (c'est-à-dire portant les mêmes noms) **==au sein du même assembly, le compilateur ne génère qu'une seule définition de type anonyme.==**

## Types anonymes contenant d'autres types anonymes

**Il est possible de créer un type anonyme composé d'autres types anonymes**. Par exemple, supposons que vous souhaitiez modéliser un bon de commande comprenant un horodatage, un prix et le véhicule acheté. Voici un nouveau type anonyme (légèrement plus sophistiqué) représentant une telle entité :

```cs
...

var purchaseItem = new
{
    TimeBought = DateTime.Now,
    ItemBought = new
    {
        Color = "Red",
        Make = "Saab",
        CurrentSpeed = 55,
    },
    Price = 34.000,
};

ReflectOverAnonymousType(purchaseItem);

Console.ReadLine();
```

À ce stade, vous devriez comprendre la syntaxe utilisée pour définir les types anonymes, mais vous vous demandez peut-être encore où (et quand) utiliser cette nouvelle fonctionnalité du langage. **En clair, les déclarations de types anonymes devraient être utilisées avec parcimonie, généralement uniquement lors de l'utilisation de la suite de technologies LINQ** (voir [[Chapitre 13|Chapitre 13]]). Il ne faudrait jamais abandonner l'utilisation de classes/structures fortement typées simplement pour le plaisir de le faire, compte tenu ==des nombreuses limitations des types anonymes, notamment les suivantes== :

- Vous ne contrôlez pas le nom du type anonyme.
- Les types anonymes héritent toujours de `System.Object`.
- Les champs et propriétés d'un type anonyme sont toujours en lecture seule.
- Les types anonymes ne prennent pas en charge les événements, les méthodes personnalisées, les opérateurs personnalisés ni les surcharges personnalisées.
- Les types anonymes sont toujours implicitement scellés.
- Les types anonymes sont toujours créés à l'aide du constructeur par défaut.

**Toutefois, lors de la programmation avec l'ensemble de technologies LINQ, vous constaterez que, dans de nombreux cas, cette syntaxe peut s'avérer utile lorsque vous souhaitez modéliser rapidement la forme générale d'une entité plutôt que sa fonctionnalité.**

>[!tip]- Autre cas pratique pour cette syntaxe
>En plus de l'API LINQ, les types anonymes peut être utilisé avec des convertisseurs JSON, et c'est d'ailleurs l'un de leurs cas d'usage les plus fréquents aujourd'hui. Bien que les "entrailles" diffèrent, les bibliothèques de sérialisation comme `System.Text.Json` ou `Newtonsoft.Json` savent parfaitement lire les types anonymes par réflexion.
>- **Prototypage rapide :** Idéal pour tester une structure de données avant de figer une classe.
>- **Performance :** Depuis .NET 6, [System.Text.Json](https://learn.microsoft.com/en-us/dotnet/standard/serialization/system-text-json/overview) est extrêmement optimisé pour transformer ces objets en chaînes de caractères.
>- **LINQ :** Vous pouvez transformer une liste d'objets complexes en une liste de types anonymes simplifiés en une seule ligne, puis la convertir directement en JSON pour un client JavaScript.
>
>>[!warning] Limites importantes
>>
>>- **Lecture seule :** Comme mentionné plus haut, une fois le JSON désérialisé dans un type anonyme, vous ne pourrez pas modifier ses valeurs.
>>- **Noms de propriétés :** Si votre JSON utilise `item_bought` (snake_case) et votre type anonyme `ItemBought` (PascalCase), la conversion échouera sans configurer de [JsonSerializerOptions](https://learn.microsoft.com/en-us/dotnet/api/system.text.json.jsonserializeroptions).

# Utilisation des types pointeurs

Et maintenant, le dernier sujet de ce chapitre, qui sera probablement le moins utilisé de toutes les fonctionnalités C# dans la plupart de vos projets .NET.

>[!note] 
>Dans les exemples qui suivent, je suppose que vous avez des connaissances de base en manipulation de pointeurs en C++. Si ce n'est pas le cas, vous pouvez ignorer ce sujet. L'utilisation de pointeurs ne sera pas une tâche courante dans la plupart des applications C#.

==Au [[Chapitre 4#Comprendre les types de valeur et les types de référence|Chapitre 4]], vous avez appris que la plateforme .NET définit deux grandes catégories de données : les types valeur et les types référence==. **==En réalité, il existe une troisième catégorie : les types pointeur==**. **Pour manipuler les types pointeur, vous disposez d’opérateurs et de mots clés spécifiques qui vous permettent de contourner le système de gestion de la mémoire du runtime .NET et de prendre le contrôle** (voir [[#Tableau 11-2 Opérateurs et mots-clés C centrés sur les pointeurs|Tableau 11-2]]).

##### Tableau 11-2: Opérateurs et mots-clés C# centrés sur les pointeurs

| Opérateur/mot-clé                | Description                                                                                                                                                                                                                                                  |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `*`                              | Cet opérateur sert à créer une variable pointeur (c’est-à-dire une variable qui représente une adresse mémoire directe). Comme en C/C++, ce même opérateur est utilisé pour l’indirection de pointeurs.                                                      |
| `&`                              | Cet opérateur permet d'obtenir l'adresse d'une variable en mémoire.                                                                                                                                                                                          |
| `->`                             | Cet opérateur est utilisé pour accéder aux champs d'un type représenté par un pointeur (la version non sécurisée de l'opérateur point C#).                                                                                                                   |
| `[]`                             | Cet opérateur (dans un contexte non sécurisé) vous permet d'indexer l'emplacement pointé par une variable pointeur (si vous êtes programmeur C/C++, vous vous souviendrez de l'interaction entre une variable pointeur et l'opérateur []).                   |
| `++`, `--`                       | Dans un contexte non sécurisé, les opérateurs d'incrémentation et de décrémentation peuvent être appliqués aux types pointeurs.                                                                                                                              |
| `+`, `-`                         | Dans un contexte non sécurisé, les opérateurs d'addition et de soustraction peuvent être appliqués aux types pointeurs.                                                                                                                                      |
| `==`, `!=`, `<`, `>`, `<=`, `=>` | Dans un contexte non sécurisé, les opérateurs de comparaison et d'égalité peuvent être appliqués aux types pointeurs.                                                                                                                                        |
| `Stackalloc`                     | Dans un contexte non sécurisé, le mot-clé `stackalloc` peut être utilisé pour allouer des tableaux C# directement sur la pile.                                                                                                                               |
| `Fixed`                          | Dans un contexte non sécurisé, le mot-clé `fixed` peut être utilisé pour fixer temporairement une variable afin que son adresse puisse être trouvée. (voir [[Chapitre 9#Faire le lien avec la gestion de la mémoire en C (Boot.dev et Gemini)\|Chapitre 9]]) |

Avant d'entrer dans les détails, je tiens à rappeler que vous aurez *rarement, voire jamais*, besoin d'utiliser les types pointeurs. ==Bien que C# vous permette de manipuler les pointeurs, sachez que le runtime .NET ignore totalement vos intentions==. Par conséquent, **si vous manipulez mal un pointeur, vous serez seul responsable des conséquences**. Compte tenu de ces avertissements, ==dans quels cas précis auriez-vous besoin de travailler avec des types pointeurs ?== Il existe deux situations courantes.

- Vous cherchez à optimiser certaines parties de votre application en manipulant directement la mémoire en dehors de la gestion du runtime .NET 5.
- Vous appelez des méthodes d'une *.dll* C ou d'un serveur COM qui requièrent des types pointeur comme paramètres. Même dans ce cas, vous pouvez souvent contourner les types pointeur et privilégier le type `System.IntPtr` et les membres de `System.Runtime.InteropServices.Marshal`.

**Lorsque vous décidez d'utiliser cette fonctionnalité du langage C#, vous devez informer le compilateur C# de vos intentions en autorisant votre projet à prendre en charge le «code non sécurisé»**. Créez un nouveau projet d'application console, nommé *UnsafeCode*, et configurez-le pour prendre en charge le code non sécurisé en ajoutant ce qui suit au fichier *UnsafeCode.csproj* :

### Activer le code non sécurisé avec Visual Studio 2022

Visual Studio 2022 propose une interface graphique pour définir cette propriété. Accédez à la page des propriétés de votre projet, puis à l’onglet « Générer » et cochez la case « Autoriser le code non sécurisé ». Voir l'image suivante.

![[Figure 11.1.png|Activer le code non sécurisé avec Visual Studio 2022]]

Pour sélectionner la configuration de compilation des paramètres, survolez la case à cocher ou le côté gauche de l'étiquette « Code non sécurisé » ; une icône d'engrenage apparaîtra. Cliquez dessus pour choisir où le paramètre sera appliqué. Voir l'image suivante.

![[Figure 11.2.png|Spécification de/des configurations pour le paramètre de code non sécurisé]]

## Le mot-clé `unsafe`

Lorsque vous souhaitez manipuler des pointeurs en C#, **vous devez déclarer explicitement un bloc de code non sécurisé à l'aide du mot-clé `unsafe`** (tout code non marqué avec le mot-clé `unsafe` est automatiquement considéré comme sûr). Par exemple, le fichier `Program.cs` suivant déclare une portée de code non sécurisé au sein des instructions de niveau supérieur sûres : 

```cs
using UnsafeCode;

Console.Title = "Calling Method with Unisafe Code";
Console.WriteLine("***** Calling Method with Unisafe Code *****\n");

unsafe
{
    // Travaille avec des type pointeurs ici!
}
// Ne peut pas travailler avec des types pointeurs ici!
```

En plus de déclarer la portée du code non sécurisé au sein d'une méthode, **vous pouvez créer des structures, des classes, des types, membres et des paramètres « non sécurisés »**. Voici quelques exemples à méditer (inutile de définir les types `Node` ou `Node2` dans votre projet actuel) :

```cs
namespace UnsafeCode;

// Cette structure entière est « non sécurisée » et ne peut
// être utilisée que dans un contexte non sécurisé.
unsafe struct Node
{
    public int Value;
    public Node* Left;
    public Node* Right;
}

// Cette structure est sûre, mais les membres Node2*
// ne le sont pas. Techniquement, vous pouvez accéder à « Value » depuis
// un contexte non sécurisé, mais pas à « Left » et « Right ».
public struct Node2
{
    public int Value;

    // Ces éléments ne sont accessibles que dans un contexte non sécurisé !
    public unsafe Node2* Left;
    public unsafe Node2* Right;
}
```

**Les méthodes** (statiques ou d'instance) **peuvent également être marquées comme non sécurisées**. Par exemple, supposons que vous sachiez qu'une méthode statique utilise la logique des pointeurs. Pour garantir que cette méthode ne puisse être appelée que depuis un contexte non sécurisé, vous pouvez la définir comme suit :

```cs
static unsafe void SquareIntPointer(int* myIntPointer)
{
    // Mettre la valeur au carré juste pour un test.
    *myIntPointer *= *myIntPointer;
}
```

La configuration de votre méthode exige que l'appelant invoque `SquareIntPointer()` comme suit :

```cs
unsafe
{
    // Travaille avec des type pointeurs ici!
    int myInt = 10;

    // OK, parce que on est dasn un contexte non sécurisé.
    SquareIntPointer(&myInt);
    Console.WriteLine($"{nameof(myInt)}: {myInt}");
}

// Ne peut pas travailler avec des types pointeurs ici!

int myInt2 = 5;

// Erreur de compilation ! Doit se trouver
// dans un contexte non sécurisé !
SquareIntPointer(&myInt2);
Console.WriteLine("myInt: {0}", myInt2);
```

**Si vous préférez ne pas imposer à l'appelant d'encapsuler l'appel dans un contexte non sécurisé, vous pouvez encapsuler toutes les instructions de niveau supérieur dans un bloc `unsafe`**. Si vous utilisez une méthode `Main()` comme point d'entrée, ==vous pouvez mettre à jour `Main()` avec le mot-clé `unsafe`. Dans ce cas, le code suivant compilerait== :

```cs
static unsafe void Main(string[] args)
{
	int myInt2 = 5;
	SquareIntPointer(&myInt2);
	Console.WriteLine("myInt: {0}", myInt2);
}
```

Si vous exécutez cette version du code, vous obtiendrez le résultat suivant :

```
myInt: 25
```

>[!warning] Attention
>Il est important de noter que le terme *unsafe* a été choisi à dessein. Accéder directement à la pile et manipuler des pointeurs peuvent entraîner des problèmes inattendus pour votre application et la machine sur laquelle elle s'exécute. Si vous devez utiliser du code non sécurisé, redoublez de vigilance.

## Utilisation des opérateurs `*` et `&`

Après avoir établi un contexte non sécurisé, **vous pouvez créer des pointeurs vers des types de données à l'aide de l'opérateur `*` et obtenir l'adresse de la donnée pointée à l'aide de l'opérateur `&`**. Contrairement au C et au C++, en **==C#, l'opérateur `*` s'applique uniquement au type sous-jacent, et non comme préfixe au nom de chaque variable pointeur==**. Par exemple, considérez le code suivant, qui illustre les manières correcte et incorrecte de déclarer des pointeurs vers des variables entières :

```cs
// Non ! Ceci est incorrect en C# !
int *pi, *pj;

// Oui ! C'est ainsi que fonctionne C#.
int* pi, pj;
```

Considérons la méthode non sécurisé suivante :

```cs
static unsafe void PrintValueAndAddress()
{
    int myInt;

    // Définit un pointeur int et l'assigne à l'addresse de myInt.
    int* ptrToInt = &myInt;
    // Attribue une valeur à myInt en utilisant l'indirection par pointeur.
    *ptrToInt = 123;

    // Affiche des données
    Console.WriteLine($"Value of myInt {myInt}");
    Console.WriteLine($"Address of myInt {(long)&ptrToInt:X}");
}
```

>[!tip]
> En C#, contrairement au C, le type "pointeur" (`int*`) n'est pas implicitement convertible en `string` ou en `object`.
>
> De plus, pour récupérer l'adresse complète (non tronqué) sur les machine modernes, il faut convertir en `long` au lieu d'un `int`.

Si vous exécutez cette méthode depuis le bloc non sécurisé, vous obtiendrez le résultat suivant :

```
**** Print Value And Address ****
Value of myInt: 123
Address of myInt: 16D3C9C70
```

## Fonction d'échange non sécurisée (et sécurisée)

Bien sûr, déclarer des pointeurs vers des variables locales simplement pour leur assigner une valeur (comme dans l'exemple précédent) n'est jamais nécessaire et pas toujours utile. **Pour illustrer un exemple plus concret de code non sécurisé, supposons que vous souhaitez créer une fonction d'échange utilisant l'arithmétique des pointeurs**.

```cs
// Valeurs pour l'échange.
int i = 10,
    j = 20;

// Échange les valeur "de manière sécurisée"
Console.WriteLine("\n***** Safe swap *****");
Console.WriteLine($"Values before safe swap: i = {i}, j = {j}");
SafeSwap(ref i, ref j);
Console.WriteLine($"Values after safe swap: i = {i}, j = {j}");

// Échange les valeurs « de manière non sécurisée ».
Console.WriteLine("\n***** Unsafe swap *****");
Console.WriteLine($"Values before unsafe swap: i = {i}, j = {j}");
unsafe
{
    UnsafeSwap(&i, &j);
}
Console.WriteLine($"Values after unsafe swap: i = {i}, j = {j}");
Console.ReadLine();
```

## Accès aux champs via des pointeurs (l'opérateur `->`)

Supposons maintenant que vous ayez défini une structure de point simple et sûre, comme suit :

```cs
struct Point
{
    public int x;
    public int y;

    public override string ToString() => $"({x}, {y})";
}
```

**Si vous déclarez un pointeur vers un type `Point`, vous devrez utiliser l'opérateur d'accès aux champs de pointeur (représenté par `->`) pour accéder à ses membres publics**. Comme indiqué dans le [[#Tableau 11-2 Opérateurs et mots-clés C centrés sur les pointeurs|Tableau 11-2]], **il s'agit de la version non sécurisée de l'opérateur point standard sûr (`.`)**. En fait, ==en utilisant l'opérateur d'indirection de pointeur (`*`), il est possible de déréférencer un pointeur pour (à nouveau) appliquer la notation de l'opérateur point==. Consultez la méthode non sécurisée :

```cs
static unsafe void UsePointerToPoint()
{
    // Accès aux membres via un pointeur
    Point point;
    Point* p = &point;
    p->x = 100;
    p->y = 200;
    Console.WriteLine(p->ToString());

    // Accès aux membres via l'indirection par pointeur.
    Point point2;
    Point* p2 = &point2;
    (*p2).x = 100;
    (*p2).y = 200;
    Console.WriteLine((*p2).ToString());
}
```

## Le mot-clé `stackalloc`

**Dans un contexte non sécurisé, vous pouvez avoir besoin de déclarer une variable locale qui alloue de la mémoire directement à partir de la pile d'appels** (et qui, par conséquent, ==n'est pas soumise au ramasse-miettes .NET==). Pour ce faire, **C# fournit le mot-clé `stackalloc`, qui est l'équivalent C# de la fonction `_alloca` de la bibliothèque d'exécution C**. Voici un exemple simple :

```cs
static unsafe string UnsafeStackAlloc()
{
    char* p = stackalloc char[52];

    for (int k = 0; k < 52; k++)
    {
        p[k] = (char)(k + 65);
    }
    return new string(p);
}
```

>un type `char` possède une taille de 2 octets. Le code précédent alloue 52 emplacements mémoire de 2 octets chacun sur la pile (*stack*).

## Fixer un type via le mot-clé `fixed`

Comme vous l'avez vu dans l'exemple précédent, ==l'allocation d'un bloc de mémoire dans un contexte non sécurisé peut être facilitée par le mot-clé `stackalloc`==. **De par la nature même de cette opération, la mémoire allouée est libérée dès que la méthode d'allocation a terminé son exécution** (puisque la mémoire est récupérée de la pile). Cependant, ==prenons un exemple plus complexe==. Lors de notre étude de l'opérateur `->`, vous avez créé un type valeur nommé `Point`. Comme pour tous les types valeur, la mémoire allouée est dépilée une fois que la portée d'exécution est terminée. Supposons, pour les besoins de la démonstration, que Point soit défini comme un type référence, comme ceci :

```cs
class PointRef // Renommé et retapé.
{
    public int x;
    public int y;

    public override string ToString() => $"({x}, {y})";
}
```

Comme vous le savez, **si l'appelant déclare une variable de type `Point`, la mémoire est allouée sur le tas, géré par le ramasse-miettes**. La question cruciale est alors : « ==Que se passe-t-il si un contexte non sécurisé souhaite interagir avec cet objet (ou tout autre objet présent sur le tas) ?== » Étant donné que le ramasse-miettes peut s'effectuer à tout moment, imaginez les problèmes rencontrés lors de l'accès aux membres de `Point` au moment précis où un balayage du tas est en cours. **Théoriquement, il est possible que le contexte non sécurisé tente d'interagir avec un membre qui n'est plus accessible ou qui a été repositionné sur le tas après avoir survécu à un balayage générationnel** (ce qui est un problème évident).

**Pour verrouiller une variable de type référence en mémoire contre un contexte non sécurisé, C# fournit le mot-clé `fixed`**. **==L'instruction `fixed` définit un pointeur vers un type managé et « épingle » cette variable pendant l'exécution du code. Sans fixed, les pointeurs vers des variables managées seraient peu utiles, car le ramasse-miettes pourrait déplacer les variables de manière imprévisible==**. (En fait, le compilateur C# n'autorise pas l'affectation d'un pointeur à une variable managée, sauf dans une instruction `fixed`.) 

Par conséquent, **si vous créez un objet `PointRef` et souhaitez interagir avec ses membres, vous devez écrire le code suivant** (sinon, vous obtiendrez une erreur de compilation) :

```cs
static unsafe void UseAndPinPoint()
{
    PointRef pt = new PointRef { x = 5, y = 6 };

    // Épingle pt en place pour qu'il ne soit pas
    // déplacé par le GC.
    fixed (int* p = &pt.x)
    {
        // Utilisation de la variable int* ici!
    }

    // pt est maintenant détaché et prêt à être collecté
    // par le GC une fois la méthode terminée.
    Console.WriteLine($"Point is {pt}");
}
```

En résumé, **le mot-clé `fixed` permet de construire une instruction qui verrouille une variable de référence en mémoire, de sorte que son adresse reste constante pendant toute la durée de l'instruction** (ou du bloc de portée). **Chaque fois que vous interagissez avec un type référence dans le contexte de code non sécurisé, il est impératif de verrouiller la référence.**

## Le mot-clé `sizeof`

Le dernier mot-clé C# relatif aux opérations non sécurisées à considérer est `sizeof`. Comme en C/C++, **le mot-clé `sizeof` en C# permet d'obtenir la taille en octets d'un type de données intrinsèque, mais pas d'un type personnalisé, sauf dans un contexte non sécurisé**. Par exemple, ==la méthode suivante n'a pas besoin d'être déclarée « non sécurisée » car tous les arguments du mot-clé `sizeof` sont des types intrinsèques== :

```cs
static void UseSizeOfOperator()
{
	Console.WriteLine("The size of short is {0}.", sizeof(short));
	Console.WriteLine("The size of int is {0}.", sizeof(int));
	Console.WriteLine("The size of long is {0}.", sizeof(long));
}
```

Toutefois, si vous souhaitez obtenir la taille de votre structure `Point` personnalisée, vous devez mettre à jour cette méthode comme suit (notez que le mot-clé `unsafe` a été ajouté) :

```cs
static void UseSizeOfOperator()
{
    Console.WriteLine("The size of short is {0}.", sizeof(short));
    Console.WriteLine("The size of int is {0}.", sizeof(int));
    Console.WriteLine("The size of long is {0}.", sizeof(long));

    unsafe
    {
        Console.WriteLine("The size of Point is {0}", sizeof(Point));
    }
}
```

Ceci conclut notre présentation de certaines des fonctionnalités avancées du langage de programmation C#. Pour être sûrs que nous sommes tous sur la même longueur d'onde, je tiens à préciser que **la plupart de vos projets .NET n'auront probablement jamais besoin d'utiliser directement ces fonctionnalités** (notamment les pointeurs). Néanmoins, comme vous le verrez dans les chapitres suivants, **certains sujets sont très utiles, voire indispensables, pour travailler avec les API LINQ, en particulier les méthodes d'extension et les types anonymes.**

# Résumé du chapitre

Ce chapitre avait pour but d'approfondir votre compréhension du langage de programmation C#. **Vous avez d'abord exploré diverses techniques avancées de construction de types** (méthodes d'indexation, opérateurs surchargés et routines de conversion personnalisées).

Ensuite, **vous avez examiné le rôle des méthodes d'extension et des types anonymes**. ==Comme vous le verrez plus en détail au [[Chapitre 13|Chapitre 13]], ces fonctionnalités sont utiles lors de l'utilisation d'API centrées sur LINQ (mais vous pouvez les utiliser partout dans votre code, si nécessaire)==. Rappelons que **les méthodes anonymes permettent de modéliser rapidement la structure d'un type, tandis que les méthodes d'extension permettent d'ajouter de nouvelles fonctionnalités aux types, sans avoir besoin de créer de sous-classes.**

Vous avez consacré le reste de ce chapitre à l'étude d'un petit nombre de mots-clés moins connus (`sizeof`, `unsafe`, etc.) et, ce faisant, **vous avez appris à manipuler les types pointeurs bruts**. Comme indiqué tout au long de l'étude des types pointeurs, **==la plupart de vos applications C# n'auront jamais besoin de les utiliser.==**

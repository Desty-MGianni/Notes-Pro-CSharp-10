---
title: "Chapitre 6: Comprendre l'Héritage et le Polymorphisme"
publish: true
---

# <big><big><big><b><font color =green>Comprendre l'Héritage et le Polymorphisme</font></b></big></big></big>

Le [[Chapitre 5]] a examiné le premier pilier de la POO : ==l’encapsulation==. Vous y avez appris à construire un type de classe unique et bien défini, avec des constructeurs et divers membres (champs, propriétés, méthodes, constantes et champs en lecture seule). **Ce chapitre se concentrera sur les deux autres piliers de la POO : l’héritage et le polymorphisme.**

Vous apprendrez d’abord à construire des familles de classes apparentées grâce à l’*héritage*. Comme vous le verrez, cette forme de réutilisation de code vous ==permet de définir des fonctionnalités communes dans une classe parente, fonctionnalités qui peuvent être exploitées et, éventuellement, modifiées par les classes enfants==. Vous apprendrez également à établir une *interface polymorphe* dans les hiérarchies de classes ==à l’aide de membres virtuels et abstraits, ainsi que le rôle du casting explicite==.

**Le chapitre se conclura par l’examen du rôle de la classe parente ultime dans les bibliothèques de classes de base .NET : `System.Object`.**

# Comprendre les mécanismes fondamentaux de l'héritage

Rappelons-nous du [[Chapitre 5#Définir les piliers de la POO|Chapitre 5]] que l'héritage est un aspect de la POO qui facilite la réutilisation du code. Plus précisément, **la réutilisation du code se présente sous deux formes : l'héritage** (==la relation "est un"==) **et le modèle de confinement/délégation** (==la relation "possède un"==). Commençons ce chapitre par examiner le modèle d'héritage classique de la relation "est un".

==Lorsque vous établissez des relations "est un" entre des classes, vous créez une dépendance entre deux ou plusieurs types de classes==. **L'idée de base de l'héritage classique est que de nouvelles classes peuvent être créées en utilisant des classes existantes comme point de départ**.

Pour commencer par un exemple simple, créez un nouveau projet d'application console nommé
*BasicInheritance*. Supposons maintenant que vous ayez conçu une classe nommée `Car` qui modélise certains détails de base d'une automobile.

```cs
namespace BasicInheritance;

// Une classe de base simple.
class Car
{
    public readonly int MaxSpeed;
    private int _currSpeed;

    public Car(int max)
    {
        MaxSpeed = max;
    }

    public Car()
    {
        MaxSpeed = 55;
    }

    public int Speed
    {
        get => _currSpeed;
        set
        {
            _currSpeed = value;
            if (_currSpeed > MaxSpeed)
                _currSpeed = MaxSpeed;
        }
    }
}
```

Notez que la classe `Car` utilise des services d'encapsulation pour contrôler l'accès au champ privé `currSpeed` à l'aide d'une propriété publique nommée `Speed`. Vous pouvez maintenant tester votre type `Car` comme suit :

```cs
using BasicInheritance;

Console.Title = "Basic Inheritance";
Console.WriteLine("**** Basic Inheritance ****\n");

// Crée un objet Car, assigne la propriété Speed et le champ _currSpeed.
Car myCar = new Car(80) { Speed = 50 };

// Affiche la vitesse actuelle (currSpeed)
Console.WriteLine($"My car is going {myCar.Speed} km/h");

Console.ReadLine();
```

## Spécification de la classe parente d'une classe existante

Supposons que vous souhaitiez créer une nouvelle classe nommée `MiniVan`. À l'instar d'un objet `Car` classique, vous souhaitez définir la classe `MiniVan` pour qu'elle prenne en charge les données de vitesse maximale, de vitesse actuelle et une propriété nommée `Speed` permettant à l'utilisateur de modifier l'état de l'objet. **Les classes `Car` et `MiniVan` sont clairement liées; en effet, on peut dire qu'un `MiniVan` est un type de `Car`**. ==La relation "est un"== (formellement appelée *héritage classique*) ==vous permet de créer de nouvelles définitions de classes qui étendent les fonctionnalités d'une classe existante==.

**La classe existante qui servira de base à la nouvelle classe est appelée *classe de base*, *superclasse* ou *classe parente***. ==Le rôle d'une classe de base est de définir toutes les données et tous les membres communs aux classes qui l'étendent==. **Les classes qui étendent la classe sont formellement appelées *classes dérivées* ou *classes enfants***. ==En C#, vous utilisez l'opérateur deux-points (`:`) dans la définition de la classe pour établir une relation "est un" entre les classes==. Supposons que vous ayez créé la nouvelle classe `MiniVan` suivante :

```cs
namespace BasicInheritance;

// Minivan "est un" Car
class MiniVan: Car
{
}
```

Actuellement, cette nouvelle classe ne définit aucun membre. Alors, qu'avez-vous gagné en
étendant votre `MiniVan` à partir de la classe de base `Car` ? **En clair, les objets `MiniVan` ont désormais accès à chaque membre public défini dans la classe parente**.

>[!warning] Note sur les constructeurs 
Bien que les constructeurs soient généralement définis comme publics, **une classe dérivée n'hérite jamais des constructeurs de sa classe parente**. Les constructeurs servent uniquement à créer la classe dans laquelle ils sont définis, **même s'ils peuvent être appelés par une classe dérivée grâce au chaînage de constructeurs**. Ce point sera abordé prochainement.

Compte tenu de la relation entre ces deux types de classes, vous pouvez maintenant utiliser la classe `MiniVan` comme suit :

```cs
using BasicInheritance;

Console.Title = "Basic Inheritance";
Console.WriteLine("**** Basic Inheritance ****\n");

...

// Maintenant on crée un objet MiniVan
MiniVan myVan = new MiniVan { Speed = 50 };
Console.WriteLine($"My van is going {myVan.Speed} km/h");

Console.ReadLine();
```

**Notez encore une fois que, même si vous n'avez ajouté aucun membre à la classe `MiniVan`, vous avez un accès direct à la propriété publique `Speed` ​​de votre classe parente et avez donc réutilisé du code**. **==Cette approche est bien meilleure que de créer une classe `MiniVan` possédant les mêmes membres que `Car`, comme une propriété `Speed`==**. *==Si vous dupliquiez du code entre ces deux classes, vous devriez alors maintenir deux ensembles de code, ce qui serait certainement une perte de temps==*.

**N'oubliez jamais que l'héritage préserve l'encapsulation** ; par conséquent, le code suivant provoque une erreur de compilation, car ==les membres privés ne sont jamais accessibles à partir d'une référence d'objet== :

```cs
// Crée un objet MiniVan
MiniVan myVan = new MiniVan();
myVan.Speed = 10;
Console.WriteLine($"My van is going {myVan.Speed} km/h");

// Erreur! Ne peut pas accéder aux membre privés.
myVan._currSpeed = 55;
Console.ReadLine();
```

Par ailleurs, ==même si la classe `MiniVan` définissait son propre ensemble de membres, elle ne pourrait toujours pas accéder aux membres privés de la classe de base `Car`==. **Rappelons que les membres `private` ne sont accessibles que par la classe qui les définit**. Par exemple, la méthode suivante dans `MiniVan` provoquerait une erreur de compilation :

```cs
namespace BasicInheritance;

// Minivan hérite de Car
class MiniVan : Car
{
    public void TestMethod()
    {
        // OK! Peut accéder aux membres publique
        // d'un parent depuis un type dérivé
        Speed = 10;
		
        // Erruer! Ne peut pas accéder aux membres privés
        // d'un parent depuis un type dérivé.
        _currSpeed = 10;
    }
}
```

## Concernant les classes de base multiples

En parlant de classes de base, **il est important de noter que C# exige qu'une classe donnée possède exactement une classe de base directe**. ==Il est impossible de créer un type de classe qui hérite directement de deux classes de base ou plus== (cette technique, prise en charge en C++ non managé, est connue sous le nom d'*héritage multiple*, ou simplement *MI*). **Si vous tentiez de créer une classe spécifiant deux classes parentes directes, comme illustré dans le code suivant, vous obtiendriez des erreurs de compilation** :

```cs
// Illégale! C# ne permet pas
// l'héritage multiple pour les classes!
class WontWork
	: BaseClassOne, BaseClassTwo
{}
```

>[!tldr]- Une explication complète de pourquoi C#  ne permet pas l'héritage multiple
>Le problème du diamant en C# est une ambiguïté qui survient lorsqu'une classe hérite de deux classes dérivées d'une même classe de base. Il en résulte une hiérarchie en forme de diamant où la classe dérivée ne peut déterminer quelle implémentation d'une méthode utiliser. C# prévient ce problème par conception, car il ne prend pas en charge l'héritage multiple de classes, garantissant ainsi qu'une classe ne peut hériter que d'une seule classe de base.
>
>>[!note]- Le cas des Interfaces (Expliqué dans le [[Chapitre 8#Héritage multiple avec les types d'interface|Chapitre 8]])
>>Avec C# 8.0 et les versions ultérieures, l'introduction d'implémentations par défaut pour les méthodes d'interface a réintroduit la possibilité d'un problème du diamant lorsqu'une classe implémente plusieurs interfaces partageant des méthodes par défaut avec la même signature. Pour résoudre ce problème, C# exige que la classe implémentant l'interface redéfinisse explicitement la méthode conflictuelle ou oblige l'appelant à convertir l'objet vers l'interface spécifique pour accéder à l'implémentation par défaut. 

Comme vous le verrez au [[Chapitre 8#Implémentation d'une interface|Chapitre 8|]], **la plateforme .NET Core permet à une classe ou structure donnée d'implémenter un nombre quelconque d'interfaces discrètes**. Ainsi, ==un type C# peut présenter plusieurs comportements tout en évitant les complexités liées à l'implémentation multiple==. Grâce à cette technique, vous pouvez créer des hiérarchies d'interfaces sophistiquées qui modélisent des comportements complexes (voir également le [[Chapitre 8#Hiérarchies d'interfaces avec implémentations par défaut (Nouveauté C 8.0)|Chapitre 8]]).

## Utilisation du mot-clé `sealed`

**C# fournit un autre mot-clé, `sealed`, qui empêche l'héritage**. ==Lorsqu'une classe est marquée comme `sealed`, le compilateur vous empêche d'en hériter==. Par exemple, supposons que vous ayez décidé qu'il est inutile d'étendre davantage la classe `MiniVan`.

```cs
// La classe MiniVan ne peut pas être étendue! (héritée)
sealed class MiniVan : Car
{
}
```

Si vous (ou un collègue) tentiez de dériver de cette classe, vous recevriez une erreur de compilation.

```cs
// Erreur! Ne peut pas étendre
// une classe marquée avec le mot clé sealed!
class DeluxeMiniVan
	: MiniVan
{
}
```

Le plus souvent, ==sceller une classe est particulièrement pertinent lors de la conception d'une classe utilitaire==. **Par exemple, l'espace de noms `System` définit de nombreuses classes scellées, comme la classe `String`**. Ainsi, tout comme pour le `MiniVan`, **si vous tentez de créer une nouvelle classe étendant `System.String`, vous obtiendrez une erreur de compilation**.

```cs
// Une autre erreur! Ne peut pas étendre
// une classe marquée comme scellée.
class MyString
	: String
{
}
```

>[!important]
>Au [[Chapitre 4#Comprendre les structures (`struct`)|Chapitre 4]], vous avez appris que **les structures C# sont toujours implicitement scellées** (voir [[Chapitre 4#Tableau 4-4 Comparaison entre les types de valeur et les types de référence.|Tableau 4-4]]). Par conséquent, ==il est impossible de dériver une structure d'une autre, une classe d'une structure, ou une structure d'une classe==. Les structures ne peuvent servir qu'à modéliser des types de données autonomes, atomiques et définis par l'utilisateur. Si vous souhaitez exploiter la relation "est un", vous devez utiliser des classes.
>>[!info] **Nouveautés C# 14**
>>Bien que l'héritage ne soit pas autorisé, C# 14 introduit les **blocs d'extension** (extension members). Cela vous permet d'enrichir des structures existantes (comme `int` ou `string`) avec de nouvelles propriétés, indexeurs ou opérateurs statiques, sans avoir besoin d'héritage.

Comme vous vous en doutez, de nombreux autres détails concernant l'héritage seront abordés dans la suite de ce chapitre. **Pour l'instant, retenez simplement que l'opérateur deux-points permet d'établir des relations entre classes de base et classes dérivées, tandis que le mot-clé `sealed` empêche tout héritage ultérieur**.

# Revisite des diagrammes de classes dans Visual Studio

Au [[Chapitre 2#Utilisation de l'outil de diagramme de classes visuel|Chapitre 2]], j'ai brièvement mentionné que Visual Studio permet d'établir visuellement les relations entre classes de base et classes dérivées lors de la conception. Pour exploiter cette fonctionnalité de l'IDE, la première étape consiste à ajouter un nouveau fichier de diagramme de classes à votre projet actuel. Pour ce faire, accédez au menu Projet -> Ajouter un nouvel élément et cliquez sur l'icône Diagramme de classes (dans l'image suivante, j'ai renommé le fichier *ClassDiagram1.cd* en *Cars.cd*).

![[Figure 6.1.png|Insérer un nouveau diagram de classe]]

Après avoir cliqué sur le bouton Ajouter, une surface de conception vierge s'affiche. Pour ajouter des types à un concepteur de classes, faites glisser chaque fichier depuis l'Explorateur de solutions vers la surface. Notez également que si vous supprimez un élément du concepteur visuel (en le sélectionnant et en appuyant sur la touche Suppr / del), cela ne détruira pas le code source associé, mais supprimera simplement l'élément de la surface de conception. L'image suivante illustre la hiérarchie des classes actuelle.

![[Figure 6.2.png|Le concepteur de classes visuelles de Visual Studio]]

Au-delà de la simple visualisation des relations entre les types au sein de votre application, rappelez-vous du [[Chapitre 2#Utilisation de l'outil de diagramme de classes visuel|Chapitre 2]] : vous pouvez également créer de nouveaux types et renseigner leurs membres à l’aide de la boîte à outils Concepteur de classes et de la fenêtre Détails de la classe.

N’hésitez pas à utiliser ces outils visuels tout au long de cet ouvrage. Toutefois, veillez toujours à analyser le code généré afin de bien comprendre le fonctionnement de ces outils.

# Comprendre le deuxième pilier de la POO : les détails de l’héritage

Maintenant que vous avez vu la syntaxe de base de l’héritage, créons un exemple plus complexe et ==découvrons les nombreux détails de la construction des hiérarchies de classes==. Pour ce faire, vous réutiliserez la classe `Employee` que vous avez conçue au [[Chapitre 5#Encapsulation utilisant des accesseurs et des mutateurs traditionnels|Chapitre 5]]. Commencez par créer un nouveau projet d’application console C# nommé *Employees*.

Ensuite, copiez les fichiers *Employee.cs,* *Employee.Core.cs* et *EmployeePayTypeEnum.cs* que vous avez créés dans l’exemple *EmployeeApp* du [[Chapitre 5#Encapsulation utilisant des accesseurs et des mutateurs traditionnels|Chapitre 5]] dans le projet *Employees*.

>[!warning] Les deux versions du code (avant et après C# 12) seront conservé pour toutes les classes mais seulement les version avant C# 12 seront illustrée au fur et à mesure des notes. **À la fin de la section, la version récente sera illustrée.**

>[!info]- La copie de fichier selon les époques de C#
>Avant .NET Core, il était nécessaire de référencer les fichiers dans le fichier *.csproj* pour pouvoir les utiliser dans un projet C#. **Avec .NET Core, tous les fichiers présents dans le répertoire courant sont automatiquement inclus dans votre projet**. Il suffit de copier les deux fichiers de l'autre projet dans le répertoire du projet courant pour qu'ils soient inclus dans votre projet.

Avant de créer des classes dérivées, deux points sont à prendre en compte. La classe `Employee` d'origine ayant été créée dans un projet nommé *EmployeeApp*, elle est encapsulée dans un espace de noms .NET Core portant le même nom. Le [[Chapitre 16#Définition des espaces de noms personnalisés (MaJ C 10.0)|Chapitre 16]] examinera les espaces de noms en détail ; toutefois, ==par souci de simplicité, renommez l'espace de noms actuel (dans les trois emplacements de fichiers) en `Employees` afin qu'il corresponde au nom de votre nouveau projet==.

```cs
// Veillez à modifier le nom de l'espace de noms dans les deux fichier C#!
namespace Employees;
partial class Employee
{...}
```


>[!warning] Si vous avez supprimé le constructeur par défaut lors des modifications apportées à la classe Employee au [[Chapitre 5#Comprendre les classes partielles|Chapitre 5]], assurez-vous de le réintégrer dans la classe.

Le deuxième détail consiste à ==supprimer tout le code commenté des différentes itérations de la classe Employee venant du chapitre précédent==.

>[!note]- Vérifier le code
>Pour vérifier que votre projet fonctionne correctement, compilez-le et exécutez-le en saisissant `dotnet run` dans une invite de commandes (dans le répertoire de votre projet) ou en appuyant sur Ctrl+F5 si vous utilisez Visual Studio. Le programme ne fera rien à ce stade ; cela vous permettra toutefois de vous assurer qu’il ne contient aucune erreur de compilation.

==Votre objectif est de créer une famille de classes modélisant différents types d'employés au sein d'une entreprise==. Supposons que vous souhaitiez exploiter les fonctionnalités de la classe `Employee` pour créer deux nouvelles classes : `SalesPerson` et `Manager`. **La nouvelle classe `SalesPerson` "est un" employé (tout comme la classe `Manager`)**. Rappelons que, ==selon le modèle d'héritage classique, les classes de base (telles que `Employee`) servent à définir des caractéristiques générales communes à tous les descendants. Les sous-classes (telles que `SalesPerson` et `Manager`) étendent ces fonctionnalités générales tout en ajoutant des fonctionnalités plus spécifiques==.

Dans cet exemple, vous supposerez que la classe `Manager` étend la classe `Employee` en enregistrant le nombre d'options d'achat d'actions, tandis que la classe `SalesPerson` gère le nombre de ventes réalisées. Insérez un nouveau fichier de classe (*Manager.cs*) qui définit la classe `Manager` avec la propriété automatique suivante :

```cs
namespace Employees;

// Les managers doivent savoir leurs nombre de stock options.
class ManagerOldSyntax : EmployeeOldSyntax 
{
    public int StockOptions { get; set; }
}
```

Ensuite, ajoutez un autre nouveau fichier de classe (*SalesPerson.cs*) qui définit la classe `SalesPerson` avec une propriété automatique appropriée.

```cs
namespace Employees;

// Les personnes des ventes doivent savoir leurs nombre de ventes.
class SalesPersonOldSyntax : EmployeeOldSyntax
{
    public int SalesNumber { get; set; }
}

```

Maintenant que vous avez établi une relation "est un", **les classes `SalesPerson` et `Manager` ont automatiquement hérité de tous les membres publics de la classe de base `Employee`**. Pour illustrer cela, mettez à jour vos instructions de niveau supérieur comme suit :

```cs
using Employees;

Console.Title = "The Employee Class Hierarchy";
Console.WriteLine("**** The Employee Class Hierarchy ****\n");

// Crée un objet d'une sous-classe qui a accès aux
// fonctinnalitées de la classe de base.
SalesPerson fred = new SalesPerson
{
    Age = 31,
    Name = "Fred",
    SalesNumber = 50,
};
```

## Appel des constructeurs de la classe de base avec le mot-clé `base`

**Actuellement, les classes `SalesPerson` et `Manager` ne peuvent être créées qu'à l'aide du constructeur par défaut** (voir [[Chapitre 5#Revisite du constructeur par défaut|Chapitre 5]]). Supposons maintenant que vous ayez ajouté un nouveau constructeur à sept arguments à la classe `Manager`, qui s'appelle comme suit :

```cs
// Assume que Manager a un construceur qui correspont à cette signature:
// (string fullName, int age, int empId,
// float currPay, string ssn, int numbOfOpts)
Manager chucky = new Manager("Chucky", 50, 92, 100000, "333-23-2322", 9000);
```

==Si vous examinez la liste des paramètres, vous constaterez clairement que la plupart de ces arguments doivent être stockés dans les variables membres définies par la classe de base `Employee`==. Pour ce faire, vous pouvez implémenter ce constructeur personnalisé sur la classe `Manager` comme suit :

```cs
public ManagerOldSyntax(
    string fullName,
    int age,
    int empId,
    float currPay,
    string ssn,
    int numbOfOpts
)
{
    // Cette propriété est définie par la classe Manager
    StockOptions = numbOfOpts;

    // Assigne les paramètres entrant en utilisant
    // les propriétés héritée de la classe parent.
    Id = empId;
    Age = age;
    Name = fullName;
    Pay = currPay;
    PayType = EmployeePayTypeEnum.Salaried;
    
    // OUPS! Ceci sera une erreur de compilation
    // si la propriété SSN est en lecture seule!
    SocialSecurityNumber = ssn;
}
```

Le premier problème de ==cette approche est que si vous définissez une propriété comme étant en lecture seule== (par exemple, la propriété `SocialSecurityNumber`), ==vous ne pouvez pas lui assigner le paramètre `string` entrant, comme le montre la dernière instruction de code de ce constructeur personnalisé==.

Le second problème est que ==vous avez indirectement créé un constructeur assez inefficace, étant donné qu'en C#, sauf indication contraire, le constructeur par défaut d'une classe de base est appelé automatiquement avant l'exécution de la logique du constructeur dérivé==. Après cela, **l'implémentation actuelle accède à de nombreuses propriétés publiques de la classe de base `Employee` pour établir son état**. Ainsi, vous avez ==effectué huit accès== (six propriétés héritées et deux appels de constructeur) ==lors de la création d'un objet `Manager`==!

**Pour optimiser la création d'une classe dérivée, il est conseillé d'implémenter les constructeurs de votre sous-classe de manière à appeler explicitement un constructeur personnalisé approprié de la classe de base, plutôt que le constructeur par défaut**. De cette façon, ==vous réduisez le nombre d'appels aux membres d'initialisation hérités (ce qui permet de gagner du temps de traitement)==. Tout d'abord, assurez-vous que votre classe parente `Employee` possède le constructeur à six arguments suivant :

```cs
// Ajout (si nécesaire) dans la classe de base Employee
public EmployeeOldSyntax(
    string name,
    int age,
    int id,
    float pay,
    string ssn,
    EmployeePayTypeEnum payType
)
{
    Name = name;
    Age = age;
    Id = id;
    Pay = pay;
    // J'ai changé la propiété en init-only.
    SocialSecurityNumber = ssn;
    PayType = payType;
}
```


```cs
public ManagerOldSyntax(
    string fullName,
    int age,
    int empId,
    float currPay,
    string ssn,
    int numbOfOpts
)
    : base(fullName, age, empId, currPay, ssn, EmployeePayTypeEnum.Salaried)
{
    // Cette propriété est définie par la classe Manager
    StockOptions = numbOfOpts;
}
```

**Ici, le mot-clé `base` est ajouté à la signature du constructeur** (de la même manière que pour ==chaîner les constructeurs d'une même classe à l'aide du mot-clé `this`, comme expliqué au [[Chapitre 5#Enchaîner des appels de constructeurs avec `this`|Chapitre 5]]==), **ce qui indique toujours qu'un constructeur dérivé transmet des données au constructeur de la classe parente immédiate**. Dans ce cas, ==vous appelez explicitement le constructeur à six paramètres défini par `Employee`, évitant ainsi des appels inutiles lors de la création de la classe enfant==. De plus, vous avez **ajouté un comportement spécifique à la classe `Manager` : le type de rémunération est toujours défini sur `Salaried`**. Le constructeur personnalisé de `SalesPerson` est presque identique, à l'exception du type de rémunération, défini sur `Commission`.

```cs
// en règles générale, toute les classes dérivées doivent
// appelés explicitement un constructeur apropriée
// venant de la classe de base.
public SalesPersonOldSyntax(
    string fullName,
    int age,
    int empId,
    float currPay,
    string ssn,
    int numbOfSales
)
    : base(
        fullName,
        age,
        empId,
        currPay,
        ssn,
        EmployeePayTypeEnum.Commission
    )
{
    // Ceci nous appartient!
    SalesNumber = numbOfSales;
}
```

>[!Note] 
>Vous pouvez utiliser le mot-clé `base` lorsqu'une sous-classe souhaite accéder à un membre public ou protégé défini par une classe parente. L'utilisation de ce mot-clé ne se limite pas à la logique du constructeur. Vous verrez des exemples d'utilisation de `base` de cette manière lors de l'étude du polymorphisme, plus loin dans ce chapitre.

Enfin, **n'oubliez pas que lorsqu'un constructeur personnalisé est ajouté à la définition d'une classe, le constructeur par défaut est supprimé silencieusement**. Par conséquent, ==veillez à redéfinir le constructeur par défaut pour les types `SalesPerson` et `Manager`==. Voici un exemple :

```cs
// Rajouter le constructeur par défaut
// dans la classe Manager aussi.
public SalesPerson() {}
```

## Protection des secrets de famille : le mot-clé `protected`

Comme vous le savez déjà, **les éléments publics sont accessibles directement de partout, tandis que les éléments privés ne sont accessibles qu’à la classe qui les a définis**. Rappelons-nous du [[Chapitre 5#Comprendre les modificateurs d'accès (MaJ C 7.2)|Chapitre 5]] : C#, à l’instar de nombreux autres langages orientés objet modernes, propose ==un mot-clé supplémentaire pour définir l’accessibilité des membres : `protected`==.

Lorsqu’une classe de base définit des données ou des membres protégés, ==elle établit un ensemble d’éléments accessibles directement à toute classe descendante==. Si vous souhaitez autoriser les classes enfants `SalesPerson` et `Manager` à accéder directement au secteur de données défini par `Employee`, vous pouvez modifier la définition de la classe `Employee` d’origine (dans le fichier *EmployeeCore.cs*) comme suit :

>Le code utilisant les nouvelles syntaxes utilisent des propriétés automatiques.

```cs
// Les données d'état (champs) protected
public partial class EmployeeOldSyntax
{
    // Les classes dérivées peuvent maintenant accédé
    // dirrectement à ces informations.
    protected string EmpName;
    protected int EmpId;
    protected int EmpAge;
    protected float CurrPay;
    protected string EmpSSN;
    protected EmployeePayTypeEnum EmpPayType;
    protected DateTime EmpHireDate;

	...
	
}
```

>[!note] Par convention
>Les membres protégés sont nommés en *PascalCase* (`EmpName`) et non en *underscore-camelCase* (`_empName`). ==Il ne s'agit pas d'une exigence du langage, mais d'une convention de codage courante==. Si vous décidez de modifier les noms comme je l'ai fait ici, assurez-vous de renommer toutes les méthodes de support de vos propriétés pour correspondre aux propriétés protégées en *PascalCase*.

==L'avantage de définir des membres protégés dans une classe de base est que les types dérivés n'ont plus besoin d'accéder indirectement aux données via des méthodes ou propriétés publiques==. **L'inconvénient potentiel est que lorsqu'un type dérivé a un accès direct aux données internes de sa classe parente, il est possible de contourner accidentellement les règles métier existantes, définies dans les propriétés publiques**. En définissant des membres protégés, ==vous instaurez un climat de confiance entre la classe parente et la classe enfant, car le compilateur ne détectera aucune violation des règles métier de votre type==.

Enfin, il est important de comprendre que, **pour l'utilisateur de l'objet, les données protégées sont considérées comme privées** (puisque l'utilisateur est extérieur à la famille). Par conséquent, l'opération suivante est illégale :

```cs
// Erreur! Ne peut pas accédér aux données protégés depuis le code client.
Employee emp = new Employee();
emp.EmpName = "Fred";
```

>[!note]- Précision sur les champs protégés
> Bien que les données de champs `protected` puissent enfreindre l'encapsulation, il est tout à fait sûr (et utile) de définir des méthodes `protected`. Lors de la construction de hiérarchies de classes, il est courant de définir un ensemble de méthodes réservées à l'usage exclusif des types dérivés et non destinées à être utilisées par le monde extérieur.

## Ajout d'une classe `sealed`

**Rappelons qu'une classe scellée ne peut pas être étendue par d'autres classes**. Comme mentionné précédemment, cette technique est le plus souvent utilisée lors de la conception d'une classe utilitaire. Toutefois, **lors de la construction de hiérarchies de classes, vous pourriez constater qu'une certaine branche de la chaîne d'héritage doit être "bloquée", car il est inutile d'étendre davantage la lignée**. Par exemple, supposons que vous ayez ajouté une nouvelle classe à votre programme (`PtSalesPerson`) qui étend le type `SalesPerson` existant. L'image suivante illustre la mise à jour actuelle.

![[Figure 6.3.png|La classe PtSalesPerson]]

`PtSalesPerson` est une classe représentant, bien sûr, un vendeur à temps partiel. Supposons, pour les besoins de la démonstration que vous souhaitiez empêcher tout autre développeur de créer une sous-classe de `PtSalesPerson`. Pour empêcher l'extension d'une classe, utilisez le mot-clé `sealed`.

```cs
sealed class PtSalesPersonOldSyntax : SalesPersonOldSyntax
{
    public PtSalesPersonOldSyntax(
        string fullName,
        int age,
        int empId,
        float currPay,
        string ssn,
        int numbOfSales
    )
        : base(fullName, age, empId, currPay, ssn, numbOfSales) { }
    // Supposons que d'autres membres soient présents...
}
```

## Comprendre l'héritage avec les types d'enregistrement (Nouveauté C# 9.0)

Les nouveaux types d'enregistrement C# 9.0 prennent également en charge l'héritage. Pour explorer cette fonctionnalité, suspendez votre travail dans le projet *Employees* et créez une nouvelle application console nommée *RecordInheritance*.

### Héritage pour les types d'enregistrements avec propriétés standard

Ajoutez deux nouveaux fichiers nommés *Car.cs* et *MiniVan.cs*, et ajoutez-y le code de définition d'enregistrement dans leurs fichier respectif :

```cs
// Car.cs
namespace RecordInheritance;

// Type enregistrement Car.
public record Car
{
    public string Make { get; init; }
    public string Model { get; init; }
    public string Color { get; init; }

    public Car(string make, string model, string color)
    {
        Make = make;
        Model = model;
        Color = color;
    }
}
```

```cs
// MiniVan.cs
namespace RecordInheritance;

// Type d'enregistrement MiniVan
public sealed record MiniVan : Car
{
    public int Seating { get; init; }

    public MiniVan(string make, string model, string color, int seating)
        : base(make, model, color)
    {
        Seating = seating;
    }
}
```

Notez qu'==il n'y a pas beaucoup de différence entre ces exemples utilisant des types d'enregistrement et les précédents exemples utilisant des classes==. **Le modificateur d'accès `sealed` sur le type d'enregistrement empêche d'autres types d'enregistrement d'en hériter**. Bien que non utilisé dans les exemples présentés, le modificateur d'accès `protected` sur les propriétés et les méthodes se comporte de la même manière qu'avec l'héritage de classes. ==Vous constaterez également que les sujets suivants de ce chapitre traitent des types d'enregistrement hérités==. En effet, ==les types d'enregistrement ne sont qu'un type particulier de classe== (comme expliqué en détail au [[Chapitre 5#Le type de donnée `record` (Nouveauté C 9.0)|Chapitre 5]]).

Les types d'enregistrement incluent également des conversions implicites vers leur classe de base, comme le montre le code suivant :

```cs
using RecordInheritance;

Console.Title = "Record Type Inheritance!";
Console.WriteLine("**** Record Type Inheritance! ****\n");

Car c = new Car("Honda", "Pilot", "Blue");
MiniVan m = new MiniVan("Honda", "Pilot", "Blue", 10);

Console.WriteLine($"Checking MiniVan is-a Car:{m is Car}");
```

Comme on pouvait s'y attendre, le résultat de la vérification `m` est que `Car` renvoie vrai, comme le montre la sortie suivante :

```
**** Record Type Inheritance! ****

Checking MiniVan is-a Car:True
```

Il est important de noter que même si les types enregistrement sont des classes spécialisées, **l'héritage croisé est impossible entre classes et enregistrements**. Autrement dit, **les classes ne peuvent pas hériter des types enregistrement, et les types enregistrement ne peuvent pas hériter des classes**. Prenons l'exemple de code suivant : les deux derniers exemples ne compileront pas.

```cs
namespace RecordInheritance;

public class TestClass { }

public record TestRecord { }
// Les classes ne peuvent pas hérité des types enregistrements.
// public class Test2 : TestRecord { }

// Les types enregistrements ne peuvent pas hérité des classes.
// public record Test2 : TestClass { }
```

### Héritage pour les types d'enregistrements avec paramètres positionnels

**L'héritage fonctionne également avec les types d'enregistrements positionnels**. L'enregistrement dérivé déclare les paramètres positionnels pour tous les paramètres de l'enregistrement de base. **L'enregistrement dérivé ne les masque pas, mais les utilise à partir de l'enregistrement de base**. ==L'enregistrement dérivé crée et initialise uniquement les propriétés qui ne sont pas présentes dans l'enregistrement de base==.

Pour observer ce comportement, créez un fichier nommé *PositionalRecordTypes.cs* dans votre projet. Ajoutez-y le code suivant :

```cs
namespace RecordInheritance;

public record PositionalCar(string Make, string Model, string Color);

public record PositionalMiniVan(
    string Make,
    string Model,
    string Color,
    int seating
) : PositionalCar(Make, Model, Color);

public record MotorCycle(string Make, string Model);

public record Scooter(string Make, string Model) : MotorCycle(Make, Model);

public record FancyScooter(string Make, string Model, string FancyColor)
    : Scooter(Make, Model);
```

Ajoutez le code suivant pour démontrer ce que vous savez déjà est vrai : que les types d’enregistrements positionnels fonctionnent exactement de la même manière que les types d’enregistrements :

```cs
PositionalCar pc = new PositionalCar("Honda", "Pilot", "Blue");
PositionalMiniVan pm = new PositionalMiniVan("Honda", "Pilot", "Blue", 10);
Console.WriteLine(
    $"Checking PositionalMiniVan is-a PositionalCar:{pm is PositionalCar}"
);
```

```
**** Record Type Inheritance! ****

Checking MiniVan is-a Car:True
Checking PositionalMiniVan is-a PositionalCar:True
```

### Mutation non destructive avec types d'enregistrements hérités

Lors de la création de nouvelles instances de types d'enregistrements à l'aide de l'expression `with`, le type d'enregistrement résultant est identique au type d'exécution de l'opérande. Prenons l'exemple suivant :

```cs
MotorCycle mc = new FancyScooter("Harley", "Lowrider", "Gold");
Console.WriteLine($"mc is a FancyScooter: {mc is FancyScooter}");

MotorCycle mc2 = mc with { Make = "Harley", Model = "Lowrider" };
Console.WriteLine($"mc2 is a FancyScooter: {mc2 is FancyScooter}");
```

Dans ces deux exemples, le type d'exécution des instances est `FancyScooter`, et non `MotorCycle` :

```
**** Record Type Inheritance! ****

...

mc is a FancyScooter: True
mc2 is a FancyScooter: True
```

### Égalité avec les types d'enregistrements hérités

Rappelons ([[Chapitre 5#Égalité des valeurs avec les types d'enregistrements|Chapitre 5]]) que les types d'enregistrements utilisent la sémantique des valeurs pour déterminer l'égalité. **Il convient de noter que le type d'enregistrement est un élément important dans le calcul de l'égalité**. Prenons l'exemple des types `MotorCycle` et `Scooter` vus précédemment :

```cs
public record MotorCycle(string Make, string Model);

public record Scooter(string Make, string Model) : MotorCycle(Make, Model);
```

Abstraction faite du fait que les classes héritées étendent généralement les classes de base, ==ces exemples simples définissent deux types d'enregistrement différents possédant les mêmes propriétés==. **Lors de la création d'instances avec les mêmes valeurs pour les propriétés, le test d'égalité échoue car il s'agit de types différents**. Prenons comme exemple le code et les résultats suivants :

```cs
MotorCycle mc3 = new MotorCycle("Harley", "Lowrider");
Scooter sc = new Scooter("Harley", "Lowrider");
Console.WriteLine($"MotorCycle and Scooter are equal: {Equals(mc3, sc)}");
```

```
**** Record Type Inheritance! ****

...

MotorCycle and Scooter are equal: False
```

==La raison pour laquelle les deux ne sont pas égaux est que la vérification d'égalité avec les types d'enregistrements utilise le *type à l'exécution*, et non le *type déclaré*==. L'exemple suivant illustre ce point :

```cs
MotorCycle mc3 = new MotorCycle("Harley", "Lowrider");
MotorCycle scMotorCycle = new Scooter("Harley", "Lowrider");
Console.WriteLine(
    $"MotorCycle and Scooter Motorcycle are equal: {Equals(mc3, scMotorCycle)}"
);
```

Notez que les variables `mc3` et `scMotorCycle` sont toutes deux déclarées comme étant de type enregistrement `MotorCycle`. Malgré cela, leurs types ne sont pas égaux, car leurs types d'exécution diffèrent :

```
**** Record type inheritance! ****

...

MotorCycle and Scooter Motorcycle are equal: False
```

### Comportement du déconstructeur avec les types d'enregistrements hérités

**La méthode `Deconstruct()` d'un enregistrement dérivé renvoie les valeurs de toutes les propriétés positionnelles du type déclaré à la compilation**. ==Dans ce premier exemple, la propriété `FancyColor` n'est pas déconstruite car le type à la compilation est `MotorCycle`==.

```cs
MotorCycle mc = new FancyScooter("Harley", "Lowrider","Gold");
var (make1, model1) = mc; // Ne déconstruit pas FancyColor
```

Toutefois, **si la variable est convertie en type dérivé, toutes les propriétés positionnelles du type dérivé sont déconstruites**, comme illustré ici :

```cs
MotorCycle mc = new FancyScooter("Harley", "Lowrider", "Gold");
var (make2, model2, fancyColor2) = (FancyScooter)mc;
```

# Programmation pour la délégation/l'inclusion

Rappelons que **la réutilisation de code se présente sous deux formes**. ==Vous venez d'explorer la relation classique "est un"==. Avant d'examiner le troisième pilier de la POO (le polymorphisme), ==examinons la relation "possède un"== (également connue sous le nom de modèle de *délégation/inclusion* ou *agrégation*). Revenons au projet *Employees* : créez un nouveau fichier nommé *BenefitPackage.cs* et ajoutez-y le code permettant de modéliser un ensemble d'avantages sociaux, comme suit :

```cs
namespace Employees;

// Ce nouveau type fonctionnera comme une classe contenue.
class BenefitPackage
{
    // Supposons que nous ayons d'autres membres qui représentent
    // les prestations dentaires/de santé, etc.
    public double ComputePayDeduction() => 125.0;
}

```

==Il serait évidemment assez étrange d'établir une relation "est un" entre la classe `BenefitPackage` et les types d'employés==. (`Employee` "est un" `BenefitPackage` ? J'en doute fort.) Cependant, **il devrait être clair qu'une relation quelconque entre les deux peut être établie. En bref, vous souhaitez exprimer l'idée que chaque employé "possède un" `BenefitPackage`**. Pour ce faire, vous pouvez mettre à jour la définition de la classe `Employee` comme suit :

```cs
// les objet Employees possèdent des bénéfices
partial class EmployeeOldSyntax
{
    // Contient un objet BenefitPackage.
    protected BenefitPackage EmpBenefits = new BenefitPackage();

...

}
```

À ce stade, vous avez réussi à encapsuler un autre objet. **Cependant, exposer les fonctionnalités de cet objet au monde extérieur nécessite une délégation**. ==La *délégation* consiste simplement à ajouter des membres publics à la classe conteneur qui utilisent les fonctionnalités de l'objet contenu==.

Par exemple, vous pouvez mettre à jour la classe `Employee` pour exposer l'objet `EmpBenefits` contenu à l'aide d'une propriété personnalisée, et utiliser ses fonctionnalités en interne grâce à une nouvelle méthode nommée `GetBenefitCost()`.

```cs
partial class EmployeeOldSyntax
{
    // Contient un objet BenefitPackage.
    protected BenefitPackage EmpBenefits = new BenefitPackage();

	...

    // Expose l'objet à travers une propriété personnalisée.
    public BenefitPackage Benefits
    {
        get { return EmpBenefits; }
        set { EmpBenefits = value; }
    }
    
    ...
    
    // Expose certain comportements de l'objet bénéfice.
    public double GetBenefitCost() => EmpBenefits.ComputePayDeduction();
```

Dans le code mis à jour ci-dessous, notez comment vous pouvez interagir avec le type interne `BenefitsPackage` défini par le type `Employee` :

```cs
using Employees;

Console.Title = "The Employee Class Hierarchy";
Console.WriteLine("**** The Employee Class Hierarchy ****\n");

...

Manager chucky = new Manager("Chucky", 50, 92, 100000, "333-23-2322", 9000);
double cost = chucky.GetBenefitCost();
Console.WriteLine($"Benefit Cost: {cost}");

Console.ReadLine();
```

## Comprendre les définitions de types imbriqués
Le [[Chapitre 5#Utilisation des modificateurs d'accès et des types imbriqués|Chapitre 5]] a brièvement abordé le concept de types imbriqués, une variante de la relation "possède un" que vous venez d'examiner. **En C# (ainsi que dans d'autres langages .NET), il est possible de définir un type (`enum`, `class`, `interface`, `struct` ou `delegate`) directement dans la portée d'une classe ou d'une structure**. ==Dans ce cas, le type imbriqué== (***==ou "interne"==***) ==est considéré comme un membre de la classe imbriquée== (***==ou "externe"==***) ==et, du point de vue de l'environnement d'exécution, peut être manipulé comme n'importe quel autre membre== (champs, propriétés, méthodes et événements). La syntaxe utilisée pour imbriquer un type est relativement simple.

```cs
public class OuterClass
{
	// Un type imbriquée publique peut être utilisé par n'importe qui.
	public class PublicInnerClass {}
	
	// Un type imbriquée privé ne peut être accédé 
	// que par les membres de la classe contenante.
	private class PrivateInnerClass {}
}
```

Bien que la syntaxe soit relativement claire, ==comprendre pourquoi on souhaiterait procéder ainsi n'est pas forcément évident==. Pour comprendre cette technique, il convient de considérer les caractéristiques suivantes de l'imbrication de types :

- Les types imbriqués **permettent un contrôle total du niveau d'accès du type interne, car ils peuvent être déclarés privés** (rappelons que les classes non imbriquées ne peuvent pas être déclarées avec le mot-clé `private`).
-  **Un type imbriqué étant membre de la classe conteneur, il peut accéder aux membres privés de cette dernière**.
- Souvent, un type imbriqué sert uniquement d'auxiliaire à la classe externe et n'est pas destiné à être utilisé par le monde extérieur.

Lorsqu'un type imbrique un autre type de classe, ==il peut créer des variables membres de ce type, comme pour n'importe quelle donnée==. Cependant, **si vous souhaitez utiliser un type imbriqué en dehors du type conteneur, vous devez le qualifier par la portée du type imbriqué**. Prenons l'exemple du code suivant :

```cs
// Crée et utilise la classe interne publique. OK!
OuterClass.PublicInnerClass inner;
inner = new OuterClass.PublicInnerClass();

// Erreur de compilation! Ne peut pas accéder à la classe privée.
OuterClass.PrivateInnerClass inner2;
inner2 = new OuterClass.PrivateInnerClass();
```

Pour utiliser ce concept dans l’exemple de l’employé, supposons que vous ayez maintenant imbriqué `BenefitPackage` directement dans le type de classe `Employee`.

```cs
partial class EmployeeOldSyntax
{
    public class BenefitPackage
    {
        // Supposons que nous ayons d'autres membres qui représentent
        // les prestations dentaires/de santé, etc.
        public double ComputePayDeduction() => 125.0;
    }
	
	...
	
}
```

**Le niveau d'imbrication est illimité**. Par exemple, supposons que vous souhaitiez créer une énumération nommée `BenefitPackageLevel`, qui répertorie les différents niveaux d'avantages sociaux qu'un employé peut choisir. Pour garantir la corrélation étroite entre `Employee`, `BenefitPackage` et `BenefitPackageLevel`, vous pouvez imbriquer l'énumération comme suit :

```cs
// EmployeeOldSyntax imbrique BenefitPackage.
partial class EmployeeOldSyntax
{
    // BenefitPackage imbrique BenefitPackageLevel.
    public class BenefitPackage
    {
        public enum BenefitPackageLevel
        {
            Standard,
            Gold,
            Platinum,
        }

        // Supposons que nous ayons d'autres membres qui représentent
        // les prestations dentaires/de santé, etc.
        public double ComputePayDeduction() => 125.0;
    }

	...
	
}
```

En raison des relations d'imbrication, notez comment vous devez utiliser cette énumération :

```cs
// Définit mes droits sociaux.
Employee.BenefitPackage.BenefitPackageLevel myBenefitLevel = Employee
    .BenefitPackage
    .BenefitPackageLevel
    .Platinum;
```

Excellent ! À ce stade, vous avez été initié à un certain nombre de mots-clés (et de concepts) qui vous permettent de ==construire des hiérarchies de types apparentés via l'héritage classique, l'inclusion et les types imbriqués==. Si les détails ne sont pas encore parfaitement clairs, ne vous inquiétez pas. Vous construirez de nombreuses autres hiérarchies au cours de la suite de cet ouvrage. Passons maintenant au dernier pilier de la POO : le polymorphisme.

# Comprendre le troisième pilier de la POO : la prise en charge du polymorphisme de C#

Souvenez vous que la classe de base `Employee` définissait une méthode nommée `GiveBonus()`, qui était initialement implémentée comme suit (avant sa mise à jour pour utiliser le modèle de propriété) :

```cs
public partial class Employee
{
	public void GiveBonus(float amount) => _currPay += amount;
	
	...
}
```

==Comme cette méthode a été définie avec le mot-clé `public`, vous pouvez désormais attribuer des primes aux vendeurs et aux responsables== (ainsi qu'aux vendeurs à temps partiel).

```cs
using Employees;

Console.Title = "The Employee Class Hierarchy";
Console.WriteLine("**** The Employee Class Hierarchy ****\n");

// Donne à chaque employées un bonus ?
Manager chucky = new Manager("Chucky", 50, 92, 100000, "333-23-2322", 9000);
chucky.GiveBonus(300);
chucky.DisplayStats();
Console.WriteLine();

SalesPerson fran = new SalesPerson("Fran", 43, 93, 3000, "932-32-3232", 31);
fran.GiveBonus(200);
fran.DisplayStats();
Console.ReadLine();
```

Le problème avec ==la conception actuelle est que la méthode `GiveBonus()`, héritée publiquement, fonctionne identiquement pour toutes les sous-classes==. Idéalement, la prime d'un vendeur ou d'un vendeur à temps partiel devrait prendre en compte le nombre de ventes. Les cadres pourraient par exemple bénéficier d'options d'achat d'actions supplémentaires en plus d'une augmentation de salaire. Face à ce constat, une question intéressante se pose : «Comment des types apparentés peuvent-ils répondre différemment à une même requête ?» Encore une fois, excellente question !

## Utilisation des mots-clés `virtual` et `override`

==Le polymorphisme permet à une sous-classe de définir sa propre version d'une méthode définie par sa classe de base, grâce au processus appelé *surcharge de méthode*==. Pour adapter votre conception actuelle, vous devez comprendre la signification des mots-clés `virtual` et `override`. **Si une classe de base souhaite définir une méthode qui peut être** (mais n'est pas obligée d'être) **redéfinie par une sous-classe, elle doit la marquer avec le mot-clé `virtual`**.

```cs
partial class Employee
{
	// Cette méthode peut maintenant être "surchargée" par une classe dérivée.
	public virtual void GiveBonus(float amount)
	{
		Pay += amount;
	}
	
	...
}
```

>Les méthodes qui ont été marquées avec le mot-clé `virtual` sont (sans surprise) appelées *méthodes virtuelles*.

**Lorsqu'une sous-classe souhaite modifier les détails d'implémentation d'une méthode virtuelle, elle utilise le mot-clé `override`**. Par exemple, les classes `SalesPerson` et `Manager` peuvent redéfinir `GiveBonus()` comme suit (==supposons que `PTSalesPerson` ne redéfinisse pas `GiveBonus()` et hérite donc simplement de la version définie par `SalesPerson`==) :

```cs
// SalesPerson.cs
class SalesPersonOldSyntax : EmployeeOldSyntax
{
	
	...
	
	// Un bonus de vendeur est influencé par ces nombre de ventes.
	public override void GiveBonus(float amount) 
	{
		// Utilisation de la déclaration switch 
		// pattern matching comme dans le chapitre 3.
		int salesBonus = SalesNumber switch
		{
			= 0 and <= 100 => 10,
			= 101 and <= 200 => 15,
			_ => 20
		};
		
		base.GiveBonus(amount * salesBonus);
	}

```

```cs
// Manager.cs
class ManagerOldSyntax : EmployeeOldSyntax
{
	...
	
    public override void GiveBonus(float amount)
    {
        base.GiveBonus(amount);
        Random r = new Random();
        StockOptions += r.Next(500);
    }
```

**Notez que chaque méthode surchargée peut exploiter le comportement par défaut grâce au mot-clé `base`**.

**Ainsi, vous n'avez pas besoin de réimplémenter entièrement la logique de `GiveBonus()`, mais vous pouvez réutiliser** (et éventuellement étendre) **le comportement par défaut de la classe parente**.

Supposons également que la méthode `DisplayStats()` actuelle de la classe `Employee` soit déclarée virtuellement.

```cs
public virtual void DisplayStats()
{
    Console.WriteLine($"Name: {Name}");
    Console.WriteLine($"Age: {Age}");
    Console.WriteLine($"ID: {EmpId}");
    Console.WriteLine($"Pay: {Pay}");
    Console.WriteLine($"SSN: {SocialSecurityNumber}");
}
```

Ainsi, ==chaque sous-classe peut redéfinir cette méthode pour afficher le nombre de ventes== (pour les vendeurs) ==et les options d'achat d'actions actuelles== (pour les responsables). Par exemple, considérons la version de la méthode `DisplayStats()` de la classe `Manager` (la classe `SalesPerson` implémenterait `DisplayStats()` de manière similaire pour afficher le nombre de ventes).

```cs
// Manager.cs
public override void DisplayStats()
{
    base.DisplayStats();
    Console.WriteLine($"Number of Stock Options: {StockOptions}");
}
```

```cs
// SalesPerson.cs
public override void DisplayStats()
{
    base.DisplayStats();
    Console.WriteLine($"Number of Sales: {SalesNumber}");
}
```

Maintenant que chaque sous-classe peut interpréter la signification de ces méthodes virtuelles pour elle-même, **chaque instance d'objet se comporte comme une entité plus indépendante**.

```cs
using Employees;

Console.Title = "The Employee Class Hierarchy";
Console.WriteLine("**** The Employee Class Hierarchy ****\n");

// Un meilleur système de bonus!
Manager chucky = new Manager("Chucky", 50, 92, 100000, "333-23-2322", 9000);
chucky.GiveBonus(300);
chucky.DisplayStats();
Console.WriteLine();

SalesPerson fran = new SalesPerson("Fran", 43, 93, 3000, "932-32-3232", 31);
fran.GiveBonus(200);
fran.DisplayStats();
Console.ReadLine();
```

Le résultat suivant illustre un possible test d'exécution de votre application jusqu'à présent :

```
**** The Employee Class Hierarchy ****

Name: Chucky
Age: 50
ID: 92
Pay: 100000
SSN: 333-23-2322
Number of Stock Options: 9490

Name: Fran
Age: 93
ID: 43
Pay: 3000
SSN: 932-32-3232
Number of Sales: 31
```

## Raccourcis pour surcharger les membres virtuels

Comme vous l'avez peut-être déjà remarqué, ==lorsque vous redéfinissez un membre, vous devez vous souvenir du type de chaque paramètre, sans oublier le nom de la méthode et les conventions de passage des paramètres== (`ref`, `out` et `params`). Visual Studio et Visual Studio Code proposent une fonctionnalité pratique pour redéfinir un membre virtuel. **Si vous saisissez le mot `override` dans la portée d'un type de classe** (==puis appuyez sur la barre d'espace==), ***roslyn* affichera automatiquement la liste de tous les membres surchargeable définis dans vos classes parentes, à l'exception des méthodes déjà redéfinies**.

==Lorsque vous sélectionnez un membre et appuyez sur la touche Entrée, l'IDE remplit automatiquement le stub de la méthode==. **Notez qu'une instruction de code appelant la version du membre virtuel de votre classe parente est également générée** (vous pouvez supprimer cette ligne si elle n'est pas nécessaire). Par exemple, si vous utilisez cette technique pour redéfinir la méthode `DisplayStats()`, vous trouverez probablement le code généré automatiquement suivant :

```cs
public override void DisplayStats()
{
	base.DisplayStats();
}
```

>[!tip] Cette fonctionnalité vient de *roslyn*, et est donc possible avec *neovim* sans l'ajout de l'appel à la méthode "héritée".

## Scellement des membres virtuels (MaJ C# 10.0)

Rappelons que **le mot-clé `sealed` peut être appliqué à un type de classe pour empêcher d'autres types d'étendre son comportement par héritage**. Comme vous vous en souvenez peut-être, vous avez scellé `PtSalesPerson` car vous avez supposé qu'il était inutile pour d'autres développeurs d'étendre davantage cette lignée d'héritage.

Par ailleurs, ==il arrive que vous ne souhaitiez pas sceller une classe entière, mais simplement empêcher les types dérivés de redéfinir certaines méthodes virtuelles==. Par exemple, supposons que vous ne vouliez pas que les vendeurs à temps partiel reçoivent des primes personnalisées. Pour empêcher la classe `PTSalesPerson` de redéfinir la méthode virtuelle `GiveBonus()`, vous pouvez sceller cette méthode dans la classe `SalesPerson` comme suit :

```cs
class SalesPersonOldSyntax : EmployeeOldSyntax
{
	...
	
    public override sealed void GiveBonus(float amount)
    {
	    ...
	}
}
```

Ici, la classe `SalesPerson` a bien redéfini la méthode virtuelle `GiveBonus()` définie dans la classe `Employee`; cependant, elle l'a explicitement marquée comme scellée. Par conséquent, **si vous tentiez de redéfinir cette méthode dans la classe `PtSalesPerson`, vous obtiendriez des erreurs de compilation, comme illustré dans le code suivant** :

```cs
sealed class PTSalesPerson : SalesPerson
{
	...

	// Erreur de compilation! Ne peux pas surchargé cette méthode
	// dans la classe PTSalesPerson, comme elle est scellée.
	public override void GiveBonus(float amount)
	{
	}
}
```

Nouveauté de C# 10 : **la méthode `ToString()` d’un enregistrement peut être scellée, empêchant ainsi le compilateur de synthétiser une méthode `ToString()` pour tout type d’enregistrement dérivé**. Reprenons l’exemple de `CarRecord` du [[Chapitre 5#Le type de donnée `record` (Nouveauté C 9.0)|Chapitre 5]] : remarquez la méthode `ToString()` scellée :

```cs
public record CarRecord
{
    public string Make { get; init; }
    public string Model { get; init; }
    public string Color { get; init; }

    public CarRecord() { }

    public CarRecord(string make, string model, string color)
    {
        Make = make;
        Model = model;
        Color = color;
    }

    public sealed override string ToString() =>
        $"This is a {Color} {Make} {Model}";
}
```

## Comprendre les classes abstraites

Actuellement, la classe de base `Employee` a été conçue pour fournir divers membres de données à ses descendants, ainsi que deux méthodes virtuelles (`GiveBonus()` et `DisplayStats()`) qui peuvent être redéfinies par un descendant donné. Bien que cela soit tout à fait correct, ==la conception actuelle a un effet secondaire assez étrange : il est possible de créer directement des instances de la classe de base `Employee`==.

```cs
// Qu'est ce que cela veut dire exactement ?
Employee X = new Employee();
```

**Dans cet exemple, la seule véritable utilité de la classe de base `Employee` est de définir des membres communs à toutes les sous-classes**. Il est fort probable que vous n'ayez pas prévu que quiconque puisse créer une instance directe de cette classe, car ==le type `Employee` lui-même est un concept trop général==. Par exemple, si je vous abordais en disant « Je suis un employé », je parie que votre première question serait : « Quel *sorte* d'employé êtes-vous ? Êtes-vous consultant, formateur, assistant administratif, correcteur ou conseiller à la Maison Blanche ? »

Étant donné que de nombreuses classes de base ont tendance à être des entités assez nébuleuses, ==une conception bien plus judicieuse pour cet exemple est d'empêcher la création directe d'un nouvel objet `Employee` dans le code==. En C#, **vous pouvez imposer cette restriction par programmation en utilisant le mot-clé `abstract` dans la définition de la classe, créant ainsi une *classe de base abstraite***.

```cs
// Met à jour la class Employee comme asbstract
// pour évitér les instanciation directe.
abstract partial class Employee
{
	...
}
```

Si vous tentez maintenant de créer une instance de la classe `Employee`, une erreur de compilation se produit.

```cs
// Erreur ! Impossible de créer une instance d'une classe abstraite !
Employee X = new Employee();
```

À première vue, définir une classe dont on ne peut pas créer directement une instance peut sembler étrange. Cependant, ==rappelons que les classes de base== (abstraites ou non) ==sont utiles car elles contiennent toutes les données communes et les fonctionnalités des types dérivés==. Grâce à cette forme d'abstraction, vous pouvez modéliser le fait que l'"idée" d'un employé est tout à fait valable; il ne s'agit simplement pas d'une entité concrète. **Il faut également comprendre que, même si vous ne pouvez pas créer *directement* une instance d'une classe abstraite, celle-ci est tout de même assemblée en mémoire lors de la création des classes dérivées**. Ainsi, ==il est parfaitement normal (et courant) que les classes abstraites définissent un nombre quelconque de constructeurs qui sont appelés indirectement lors de l'allocation des classes dérivées==.

À ce stade, vous avez construit une hiérarchie d'employés assez intéressante. Vous ajouterez davantage de fonctionnalités à cette application plus tard dans ce chapitre, lors de l'**étude des règles de conversion de type en C#**. En attendant, l'image suivante illustre l'essentiel de votre conception actuelle.

![[Figure 6.4.png|La hiérarchie Employee]]

## Comprendre l'interface polymorphe

**Lorsqu'une classe est définie comme classe de base abstraite (via le mot-clé `abstract`), elle peut définir un nombre quelconque de *membres abstraits***. Les membres abstraits sont utilisés pour définir un membre qui ne fournit *pas* d'implémentation par défaut, mais dont chaque classe dérivée *doit* tenir compte. Ce faisant, ==vous imposez une *interface polymorphe* à chaque descendant, les laissant gérer la tâche de fournir les détails de vos méthodes abstraites==.

**En résumé, l'interface polymorphe d'une classe de base abstraite fait simplement référence à son ensemble de méthodes virtuelles et abstraites**. ==C'est bien plus intéressant qu'il n'y paraît, car cette caractéristique de la POO permet de créer des applications logicielles facilement extensibles et flexibles==. Pour illustrer cela, vous allez implémenter (et légèrement modifier) ​​la hiérarchie des formes brièvement examinée au [[Chapitre 5#Définir les piliers de la POO|Chapitre 5]] lors de la présentation des piliers de la POO. Pour commencer, créez un nouveau projet d'application console C# nommé *Shapes*.

Dans l'image suivante, notez que les types `Hexagon` et `Circle` héritent chacun de la classe de base `Shape`. Comme toute classe de base, `Shape` définit un certain nombre de membres (une propriété `PetName` et une méthode `Draw()`, dans ce cas) qui sont communs à tous les descendants.

![[Figure 6.5.png|La hiérarchie Shapes]]e

Tout comme pour la hiérarchie `Employee`, vous devriez pouvoir constater que vous ne souhaitez pas autoriser l'utilisateur de l'objet à créer directement une instance de `Shape`, car il s'agit d'un concept trop abstrait. De même, ==pour empêcher la création directe du type `Shape`, vous pouvez le définir comme une classe abstraite==. De plus, ==étant donné que vous souhaitez que les types dérivés répondent de manière unique à la méthode `Draw()`, déclarons-la `virtual` et définissons une implémentation par défaut==. **Notez que le constructeur est déclaré protégé afin qu'il ne puisse être appelé que depuis les classes dérivées**.

```cs
namespace Shapes;

// La classe de base abstraite de la hiérarchie
abstract class Shape
{
    protected Shape(string name = "noName")
    {
        PetName = name;
    }

    public string PetName { get; set; }

    // Une seule méthode virtuelle
    public virtual void Draw()
    {
        Console.WriteLine("Inside Shape.Draw()");
    }
}
```

Notez que ==la méthode virtuelle `Draw()` fournit une implémentation par défaut qui affiche simplement un message vous informant que vous appelez la méthode `Draw()` dans la classe de base `Shape`==. **Rappelons que lorsqu'une méthode est marquée avec le mot-clé `virtual`, elle fournit une implémentation par défaut dont tous les types dérivés héritent automatiquement**. ==Si une classe enfant le souhaite, elle peut redéfinir la méthode, mais elle n'y est pas obligée==. Considérons maintenant l'implémentation suivante des types `Circle` et `Hexagon` :

```cs
// Circle.cs
namespace Shapes;

// Circle NE SURCHARGE PAS Draw()
class Circle : Shape
{
    public Circle() { }

    public Circle(string name)
        : base(name) { }
}
```

```cs
// Hexagon.cs
namespace Shapes;

// Hexagon SURCHARGE Draw()
class Hexagon : Shape
{
    public Hexagon() { }

    public Hexagon(string name)
        : base(name) { }

    public override void Draw()
    {
        Console.WriteLine($"Drawing {PetName} the Hexagon");
    }
}
```

L'utilité des méthodes abstraites devient limpide lorsqu'on se souvient que **les sous-classes ne sont jamais tenues de redéfinir les méthodes virtuelles** (comme dans le cas de `Circle`). Par conséquent, si vous créez une instance des types `Hexagon` et `Circle`, vous constaterez que `Hexagon` sait se dessiner correctement ou, du moins, afficher un message approprié dans la console. `Circle`, en revanche, est pour le moins déconcerté.

```cs
using Shapes;

Console.Title = "Fun with Polymorphism";
Console.WriteLine("**** Fun with Polymorphism ****\n");

Hexagon hex = new Hexagon("Beth");
hex.Draw();

Circle cir = new Circle("Cindy");

// Appelle l'implémentation de la classe de base!
cir.Draw();

Console.ReadLine();
```

Considérons maintenant le résultat suivant du code précédent :

```
**** Fun with Polymorphism ****

Drawing Beth the Hexagon
Inside Shape.Draw()
```

Il est clair que la hiérarchie actuelle n'est pas conçue de manière optimale. **Pour forcer chaque classe enfant à redéfinir la méthode `Draw()`, vous pouvez définir `Draw()` comme une méthode abstraite de la classe `Shape`**, ce qui, par définition, **implique qu'aucune implémentation par défaut n'est fournie**. En C#, ==pour déclarer une méthode comme abstraite, on utilise le mot-clé `abstract`==. Notez que les méthodes abstraites ne fournissent aucune implémentation.

```cs
namespace Shapes;

// La classe de base abstraite de la hiérarchie
abstract class Shape
{
    protected Shape(string name = "noName")
    {
        PetName = name;
    }

    public string PetName { get; set; }

    // Une seule méthode virtuelle
    // Force toutes les classes enfants à
    // définir une implémentation à la méthode.
    public abstract void Draw();
}

```

>[!warning] Les méthodes abstraites ne peuvent être définies que dans des classes abstraites. Toute autre tentative entraînera une erreur de compilation.

**Les méthodes marquées `abstract` sont de purs protocoles**. ==Elles définissent simplement le nom, le type de retour== (le cas échéant) ==et l'ensemble de paramètres== (si nécessaire). Ici, la classe abstraite `Shape` informe les types dérivés : 

>"il existe une méthode nommée `Draw()` qui ne prend aucun argument et ne retourne rien. Si vous héritez de moi, vous devez vous en charger."

**De ce fait, vous êtes désormais tenu de redéfinir la méthode `Draw()` dans la classe `Circle`**. Si vous ne le faites pas, ==`Circle` est également considéré comme un type abstrait non instanciable qui doit être annoté avec le mot-clé `abstract`== (ce qui est évidemment inutile dans cet exemple). Voici la mise à jour du code :

```cs
namespace Shapes;

// Si on n'implémente pas la méthode abstraite Draw(), Circle
// devra alors être considéré comme abstraite,
// et devra donc être marquée abstract
class Circle : Shape
{
    public Circle() { }

    public Circle(string name)
        : base(name) { }

    public override void Draw()
    {
        Console.WriteLine($"Drawing {PetName} the Circle");
    }
}
```

==La réponse courte est que vous pouvez désormais considérer que tout objet dérivant de `Shape` possède bien une version unique de la méthode `Draw()`==. Pour illustrer pleinement le concept de polymorphisme, prenons l'exemple du code suivant :

```cs
using Shapes;

Console.Title = "Fun with Polymorphism";
Console.WriteLine("**** Fun with Polymorphism ****\n");

...

// Crée un tableau d'objet compatible avec Shape
Shape[] myShape =
{
    new Hexagon(),
    new Circle(),
    new Hexagon("Mick"),
    new Circle("Beth"),
    new Hexagon("Linda"),
};

// Parcourir chaque élément et interagir 
// avec l'interface polymorphe.
foreach (Shape s in myShape)
{
  s.Draw();
}
Console.ReadLine();
```

Voici le résultat du code modifié :

```
**** Fun with Polymorphism ****

...

Drawing noName the Hexagon
Drawing noName the Circle
Drawing Mick the Hexagon
Drawing Beth the Circle
Drawing Linda the Hexagon
```

Ce code illustre parfaitement le polymorphisme. ==Bien qu'il soit impossible de créer *directement* une instance d'une classe de base abstraite (`Shape`), vous pouvez librement stocker des références à n'importe quelle sous-classe possédant une variable de base abstraite==. Ainsi, ==lors de la création d'un tableau de `Shapes`, celui-ci peut contenir n'importe quel objet dérivant de la classe de base `Shape`== (toute tentative d'insertion d'objets incompatibles avec `Shape` dans le tableau générera une erreur de compilation).

**Étant donné que tous les éléments du tableau `myShapes` dérivent bien de `Shape`, vous savez qu'ils prennent tous en charge la même «interface polymorphe»** (==ou, plus simplement, qu'ils possèdent tous une méthode `Draw()`==). Lors de l'itération dans le tableau des références `Shape`, **le type sous-jacent est déterminé à l'exécution. À ce stade, la version correcte de la méthode `Draw()` est appelée en mémoire**.

**Cette technique simplifie également l'extension de la hiérarchie existante en toute sécurité**. Par exemple, ==supposons que vous ayez dérivé d'autres classes de la classe de base abstraite `Shape`== (`Triangle`, `Square`, etc.). **Grâce à l'interface polymorphe, le code à l'intérieur de votre boucle `foreach` n'aurait pas besoin d'être modifié le moins du monde, car le compilateur impose que seuls les types compatibles avec `Shape` soient placés dans le tableau `myShapes`**.

## Comprendre le masquage de membre

C# offre une fonctionnalité inverse à la redéfinition de méthode : le *masquage*. Formellement, ==si une classe dérivée définit un membre identique à un membre défini dans une classe de base, la classe dérivée a masqué la version de la classe parente==. En pratique, **ce risque est maximal lorsque vous héritez d'une classe que vous** (ou votre équipe) **n'avez pas créée vous-même** (par exemple, lors de l'achat d'un logiciel tiers). À titre d'exemple, supposons que vous receviez une classe nommée `ThreeDCircle` d'un collègue (ou camarade) qui définit une sous-routine nommée `Draw()` sans argument.

```cs
namespace Shapes;

class TreeDCircle
{
    public void Draw()
    {
      Console.WriteLine("Drawing a 3D Circle");
    }

}
```

Vous considérez que` ThreeDCircle` "est un" `Circle`, vous dérivez donc de votre type `Circle` existant.

```cs
namespace Shapes;

class TreeDCircle : Circle
{
    public void Draw()
    {
        Console.WriteLine("Drawing a 3D Circle");
    }
}
```

Après la recompilation, vous trouverez l'avertissement suivant :

```
warning CS0114: 'TreeDCircle.Draw()' hides inherited member 'Circle.Draw()'. To make the current member override that implementation, add the override keyword. Otherwise add the new keyword.
```

**Le problème est que vous avez une classe dérivée (`ThreeDCircle`) qui contient une méthode identique à une méthode héritée**. Pour résoudre ce problème, plusieurs options s'offrent à vous. ==Vous pouvez simplement mettre à jour la version de `Draw()` de la classe enfant à l'aide du mot-clé `override`== (comme suggéré par le compilateur). Avec cette approche, le type `ThreeDCircle` peut étendre le comportement par défaut de la classe parente selon les besoins. Cependant, ==si vous n'avez pas accès au code définissant la classe de base== (comme c'est souvent le cas avec les bibliothèques tierces), ==vous ne pourrez pas modifier la méthode `Draw()` en tant que membre virtuel, car vous n'avez pas accès au fichier de code !==

**Une autre solution consiste à ajouter le mot-clé `new` au membre `Draw()` concerné de la classe dérivée** (`ThreeDCircle`, dans cet exemple). Cela ==indique explicitement que l’implémentation du type dérivé est intentionnellement conçue pour ignorer effectivement la version du parent== (encore une fois, dans le monde réel, cela peut être utile si un logiciel externe entre en conflit avec votre logiciel actuel).

```cs
namespace Shapes;

// Cette classe étend Circle et masque la méthode héritée Draw()
class TreeDCircle : Circle
{
    // Masque n'importe quel implémentation de Draw() au dessus de moi.
    public new void Draw()
    {
        Console.WriteLine("Drawing a 3D Circle");
    }
}
```

**Vous pouvez également appliquer le mot-clé `new` à tout type de membre hérité d'une classe de base** (champ, constante, membre statique ou propriété). Par exemple, supposons que `ThreeDCircle` souhaite masquer la propriété héritée `PetName`.

```cs
namespace Shapes;

// Cette classe étend Circle et masque la méthode héritée Draw()
class TreeDCircle : Circle
{

    // Masque la propriété PetName au dessus de moi.
    public new string PetName { get; set; }

    // Masque n'importe quel implémentation de Draw() au dessus de moi.
    public new void Draw()
    {
        Console.WriteLine("Drawing a 3D Circle");
    }
}
```

Enfin, notez qu'**il est toujours possible de déclencher l'implémentation de la classe de base d'un membre masqué en utilisant une conversion explicite**, comme décrit dans la section suivante. Le code suivant en donne un exemple :

```cs
...
// Ceci appelle la méthode Draw() de ThreeDCircle
ThreeDCircle o = new ThreeDCircle();
o.Draw();

// Ceci appelle la méthode draw du parent!
((Circle)o).Draw();
```

# Comprendre les règles de conversion de classes de base et dérivées

Maintenant que vous pouvez créer une famille de types de classes apparentés, ==vous devez apprendre les *règles des opérations de conversion* de classes==. Pour ce faire, revenons à la hiérarchie des employés créée précédemment dans ce chapitre et ajoutons quelques nouvelles méthodes au fichier *Program.cs* (si vous suivez ce tutoriel, ouvrez le projet `Employees`). Comme décrit plus loin dans ce chapitre, **la classe de base ultime du système est `System.Object`. Par conséquent, tout est de type `Object` et peut être traité comme tel**. De ce fait, il est possible de stocker une instance de n'importe quel type dans une variable objet.

```cs
static void CastingExamples()
{
    // Un Manager "est un" System.Object, donc on peut stocker
    // une référence Manager dans une variable object sans problème.
    object frank = new Manager("Frank Zappa", 9, 3000, 40000, "111-11-1111", 5);
}
```

Dans le projet `Employees`, les types `Managers`, `SalesPerson` et `PtSalesPerson` héritent tous de `Employee`. **Vous pouvez donc stocker n'importe lequel de ces objets dans une référence de classe de base valide**. Par conséquent, les instructions suivantes sont également valides :

```cs
static void CastingExamples()
{
    // Un Manager "est un" System.Object, donc on peut stocker
    // une référence Manager dans une variable Object sans problème.
    object frank = new Manager(
        "Frank Zappa",
        9,
        3000,
        40000,
        "111-11-1111",
        5
    );

    // Un Manager "est un" Employee aussi.
    Employee moonUnit = new Manager(
        "MoonUnit Zappa",
        2,
        3001,
        20000,
        "101-11-1321",
        1
    );

    // Une PtSalesperson "est un" Salesperson.
    SalesPerson jill = new PtSalesPerson(
        "Jill",
        834,
        3002,
        100000,
        "111-12-1119",
        90
    );
}
```

**La première règle de la conversion de type entre classes est que lorsque deux classes sont liées par une relation "est un", il est toujours possible de stocker un objet dérivé dans une référence à la classe de base**. ==Formellement, on parle de *transtypage implicite*, car cela fonctionne tout simplement grâce aux lois de l'héritage==. Ceci permet des constructions de programmation puissantes. Par exemple, supposons que vous ayez défini une nouvelle méthode dans votre fichier *Program.cs* actuel.

```cs
static void GivePromotion(Employee emp)
{
    // Augmenter les salaires...
    // Créer de nouvelles places de parking dans le garage de l'entreprise...
    Console.WriteLine("{0} was promoted!", emp.Name);
}
```

Étant donné que cette méthode prend un seul paramètre de type `Employee`, **vous pouvez effectivement lui passer directement n'importe quel descendant de la classe `Employee`, compte tenu de la relation "est un"**.

```cs
static void CastingExamples()
{
    // Un Manager "est un" System.Object, donc on peut stocker
    // une référence Manager dans une variable Object sans problème.
    object franck = new Manager(
        "Frank Zappa",
        9,
        3000,
        40000,
        "111-11-1111",
        5
    );

    // Un Manager "est un" Employee aussi.
    Employee moonUnit = new Manager(
        "MoonUnit Zappa",
        2,
        3001,
        20000,
        "101-11-1321",
        1
    );
    GivePromotion(moonUnit);

    // Une PtSalesperson "est un" Salesperson.
    SalesPerson jill = new PtSalesPerson(
        "Jill",
        834,
        3002,
        100000,
        "111-12-1119",
        90
    );
    GivePromotion(jill);
}
```

Le code précédent compile grâce à la conversion implicite du type de la classe de base (`Employee`) vers le type dérivé. **Cependant, que se passe-t-il si vous souhaitez également promouvoir Frank Zappa (actuellement stocké dans une référence `System.Object générale`) ?** Si vous passez directement l'objet `frank` à cette méthode, ==vous obtiendrez une erreur de compilation comme suit== :

```cs
object frank = new Manager("Frank Zappa", 9, 3000, 40000, "111-11-1111", 5);
// Erreur !
GivePromotion(frank);
```

Le problème est que ==vous tentez de passer une variable qui n'est pas déclarée comme `Employee` mais comme un objet plus général de type `System.Object`==. Étant donné que `object` est plus haut dans la chaîne d'héritage que `Employee`, ==le compilateur n'autorisera pas de conversion implicite, afin de garantir la sécurité des types de votre code==.

==Même si vous pouvez déterminer que la référence à l'objet pointe vers une classe compatible avec `Employee` en mémoire, le compilateur ne le peut pas, car cela ne sera connu qu'à l'exécution==. **Vous pouvez satisfaire le compilateur en effectuant une conversion explicite**. ==C'est la deuxième règle de la conversion : vous pouvez, dans de tels cas, effectuer une conversion descendante explicite à l'aide de l'opérateur de conversion C#==. Le modèle de base à suivre pour effectuer une conversion explicite ressemble à ce qui suit :

```cs
(TypeCible)objetActuel
```

>[!note]- Rappel du [[Chapitre 3#Rétrécissement et élargissement des conversions de types de données|Chapitre 3]]
>Ce qui est décrit dans cette section est le fonctionnement générale de C# pour le casting (explicite ou implicite) de type de donnée. Ce qui à été expliquée dans le [[Chapitre 3#Rétrécissement et élargissement des conversions de types de données|Chapitre 3]] est une section de cela, appliquée aux type `System` de valeurs.

Ainsi, pour transmettre la variable objet à la méthode `GivePromotion()`, vous pouvez écrire le code suivant :

```cs
static void CastingExamples()
{
	// OK!
    object frank = new Manager(
        "Frank Zappa",
        9,
        3000,
        40000,
        "111-11-1111",
        5
    );
	GivePromotion((Manager)frank);
	
	...
}
```

## Utilisation du mot clé C# `as`

**Notez que le casting explicite est évalué à l'exécution, et non à la compilation**. Supposons, par exemple, que votre projet *Employees* contienne une copie de la classe `Hexagon` créée précédemment dans ce chapitre. Par souci de simplicité, vous pouvez ajouter la classe suivante au projet actuel :

```cs
namespace Shapes;

class Hexagon
{
    public void Draw()
    {
        Console.WriteLine("Drawing a hexagon!");
    }
}
```

**Bien que convertir l'objet `Employee` en un objet `Shape` n'ait absolument aucun sens, un code comme celui-ci pourrait compiler sans erreur** :

```cs
// Aïe ! On ne peut pas convertir frank en Hexagon, mais ça compile sans problème !
object frank = new Manager();
Hexagon hex = (Hexagon)frank;
```

**Vous recevriez toutefois une erreur d'exécution, ou plus formellement, une exception d'exécution**. Le [[Chapitre 7|Chapitre 7]] examinera en détail la gestion structurée des exceptions ; cependant, ==il convient de souligner, pour le moment, que lors d'une conversion explicite, vous pouvez intercepter une éventuelle conversion invalide à l'aide des mots clés `try` et `catch`== (voir le [[Chapitre 7|Chapitre 7]] pour plus de détails).

```cs
// Détecter une éventuelle conversion invalide.
object frank = new Manager();
Hexagon hex;
try
{
    hex = (Hexagon)frank;
}
catch (InvalidCastException ex)
{
    Console.WriteLine(ex.Message);
}
```

Bien évidemment, il s'agit d'un exemple artificiel; vous ne vous soucieriez jamais de convertir entre ces types dans cette situation. Cependant, ==supposons que vous ayez un tableau de types `System.Object`, dont seuls quelques-uns contiennent des objets compatibles avec `Employee`. Dans ce cas, vous souhaitez déterminer si un élément du tableau est compatible dès le départ et, si oui, effectuer la conversion==.

**C# fournit le mot-clé `as` pour déterminer rapidement à l'exécution si un type donné est compatible avec un autre**. Lorsque vous utilisez le mot-clé `as`, vous pouvez déterminer la compatibilité en vérifiant si la valeur de retour est `null`. Considérez ce qui suit :

```cs
// Utilise "as" pour tester la compatibilité.
object[] things = [new Hexagon(), false, new Manager(), "Last thing"];

foreach (object item in things)
{
    Hexagon h = item as Hexagon;
    if (h == null)
    {
        Console.WriteLine("Item is not a hexagon");
    }
    else
    {
        h.Draw();
    }
}
```

Ici, vous parcourez chaque élément du tableau d'objets, en vérifiant sa compatibilité avec la classe `Hexagon`. **Si** (et seulement si !) **vous trouvez un objet compatible avec `Hexagon`, vous appelez la méthode `Draw()`. Sinon, vous indiquez simplement que les éléments ne sont pas compatibles**.

## Utilisation du mot-clé C# `is` (MaJ C# 7.0 C# 9.0)

**En plus du mot-clé `as`, le langage C# propose le mot-clé `is` pour déterminer si deux éléments sont compatibles**. ==Contrairement au mot-clé `as`, cependant, le mot-clé `is` renvoie `false`, plutôt qu'une référence `null`, si les types sont incompatibles==. Actuellement, la méthode `GivePromotion()` est conçue pour accepter tout type dérivé de `Employee`. Prenons l'exemple de la mise à jour suivante, qui vérifie désormais précisément le "type d'employé" transmis :

```cs
static void GivePromotion(Employee emp)
{
    // Augmenter les salaires...
    // Créer de nouvelles places de parking dans le garage de l'entreprise...
    Console.WriteLine("{0} was promoted!", emp.Name);
    if (emp is SalesPerson)
    {
        Console.WriteLine($"{emp.Name} made {((SalesPerson)emp).SalesNumber}");
        Console.WriteLine();
    }
    else if (emp is Manager)
    {
        Console.WriteLine($"{emp.Name} had {((Manager)emp).StockOptions}");
        Console.WriteLine();
    }
}
```

Ici, vous effectuez une **vérification à l'exécution pour déterminer à quoi pointe réellement en mémoire la référence de classe de base entrante**. Après avoir déterminé si vous avez reçu un type `SalesPerson` ou `Manager`, ==vous pouvez effectuer une conversion explicite pour accéder aux membres spécialisés de la classe==. **Notez également qu'il n'est pas nécessaire d'encapsuler vos opérations de conversion dans une structure `try`/`catch`, car vous savez que la conversion est sûre si vous entrez dans la portée de l'une ou l'autre des instructions if, étant donné votre vérification conditionnelle**.

Nouveauté de C# 7.0 : **le mot-clé `is` peut également affecter le type converti à une variable si la conversion réussit**. ==Cela simplifie la méthode précédente en évitant le problème de « double conversion »==. Dans l'exemple précédent, la première conversion est effectuée lors de la vérification de la correspondance des types, et si c'est le cas, la variable doit être convertie une seconde fois. Considérez cette mise à jour de la méthode précédente :

```cs
static void GivePromotion(Employee emp)
{
    Console.WriteLine("{0} was promoted!", emp.Name);
    // Vérifie si c'est un SalesPerson, si oui, assigne à une variable s
    if (emp is SalesPerson s)
    {
        Console.WriteLine($"{s.Name} made {s.SalesNumber}");
        Console.WriteLine();
    }

    // Vérifie si c'est un manager, si oui, assigne à une variable m
    else if (emp is Manager m)
    {
        Console.WriteLine($"{m.Name} had {m.StockOptions}");
        Console.WriteLine();
    }
}
```

**C# 9.0 a introduit des fonctionnalités supplémentaires de correspondance de modèles (*pattern matching*)**, traitées au [[Chapitre 3#Améliorations apportées à la correspondance de motifs (C 9.0)|Chapitre 3]]. **Ces correspondances de modèles mises à jour peuvent être utilisées avec le mot-clé `is`**. Par exemple, pour vérifier si l'instance n'est ni un `Manager` et ni un `SalesPerson`, utilisez le code suivant :

```cs
if (emp is not Manager and not SalesPerson)
{
    Console.WriteLine($"Unable to promote {emp.Name}. Wrong employee type");
    Console.WriteLine();
}
```

### Utilisation du mot-clé `is` (Nouveauté C# 7.0)

**Le mot-clé `is` peut également être utilisé conjointement avec l'espace réservé de variable `discard`**. Si vous souhaitez créer une boucle `catchall` dans votre instruction `if` ou `switch`, vous pouvez procéder comme suit :

```cs
if (obj is var _)
{
  // Faire quelque chose
}
```

**Cela correspondra à tout, alors faites attention à l'ordre dans lequel vous utilisez le comparateur avec le discard**. La méthode `GivePromotion()` mise à jour est présentée ici :

```cs
static void GivePromotion(Employee emp)
{
    // Vérifie si c'est un SalesPerson, si oui, assigne à une variable s
    if (emp is SalesPerson s)
    {
        Console.WriteLine($"{s.Name} made {s.SalesNumber}");
        Console.WriteLine();
    }
    // Vérifie si c'est un manager, si oui, assigne à une variable m
    else if (emp is Manager m)
    {
        Console.WriteLine($"{m.Name} had {m.StockOptions}");
        Console.WriteLine();
    }
    else if (emp is var _)
    {
        Console.WriteLine($"Unable to promote {emp.Name}. Wrong employee type");
        Console.WriteLine();
    }
}
```

**La dernière instruction `if` interceptera toute instance d'`Employee` qui n'est ni un `Manager`, ni un `SalesPerson`, ni un `PtSalesPerson`**. ==N'oubliez pas que vous pouvez effectuer un cast vers une classe de base==; ainsi, le `PtSalesPerson` sera enregistré comme un `SalesPerson`.

## Retour sur la correspondance de motifs (Nouveauté C# 7.0)

Le [[Chapitre 3#Exécution de la correspondance de modèles dans les instructions switch (Nouveauté C 7.0, MaJ C 9.0)|Chapitre 3]] a présenté la fonctionnalité de filtrage par motif (*pattern matching*) de C# 7, ainsi que les mises à jour apportées par C# 9.0. Maintenant que vous maîtrisez le casting, voici un exemple plus pertinent. **L'exemple précédent peut désormais être facilement mis à jour pour utiliser une instruction `switch` de filtrage par motifs**, comme suit :

```cs
static void GivePromotion(Employee emp)
{
    Console.WriteLine($"{emp.Name} was promoted!");
    switch (emp)
    {
        case SalesPerson s:
            Console.WriteLine($"{s.Name} made {s.SalesNumber} sale(s)!");
            break;
        case Manager m:
            Console.WriteLine(
                $"{m.Name} had {m.StockOptions} stock options..."
            );
            break;
    }
    Console.WriteLine();
}
```

**Lorsqu'on ajoute une clause `when` à l'instruction `case`, la définition complète de l'objet après conversion est disponible**. Par exemple, ==la propriété `SalesNumber` existe uniquement dans la classe `SalesPerson` et non dans la classe `Employee`==. **Si la conversion dans la première instruction `case` réussit, la variable `s` contiendra une instance de la classe `SalesPerson`**. L'instruction `case` pourrait alors être mise à jour comme suit :

```cs
case SalesPerson s when s.SalesNumber > 5:
```

Ces nouveaux ajouts aux instructions `is` et `switch` apportent des ==améliorations appréciables qui contribuent à réduire la quantité de code nécessaire à la mise en correspondance==, comme l'ont démontré les exemples précédents.

### Utilisation des variables jetables avec les instructions `switch` (Nouveauté C# 7.0)

**Les *variables jetables* peut également être utilisée dans les instructions `switch`**, comme illustré dans le code suivant :

```cs
static void GivePromotion(Employee emp)
{
    Console.WriteLine($"{emp.Name} was promoted!");
    switch (emp)
    {
        case SalesPerson s when s.SalesNumber > 5:
            Console.WriteLine($"{s.Name} made {s.SalesNumber} sale(s)!");
            break;
        case Manager m:
            Console.WriteLine(
                $"{m.Name} had {m.StockOptions} stock options..."
            );
            break;
        case Employee _:
            Console.WriteLine(
                $"Unable to promote {emp.Name}. Wrong employee type"
            );
            break;
    }
    Console.WriteLine();
}
```

**Chaque type de saisie entrant est déjà `Employee`, donc la dernière condition est donc toujours vraie**. Cependant, comme nous l'avons vu lors de l'introduction de la correspondance de modèles au [[Chapitre 3#Exécution de la correspondance de modèles dans les instructions switch (Nouveauté C 7.0, MaJ C 9.0)|Chapitre 3]], une fois qu'une correspondance est trouvée, l'instruction `switch` est annulée. ==Ceci démontre l'importance de respecter l'ordre des instructions==. **Si la dernière condition était placée en haut, aucun employé ne serait jamais promu.**

# Comprendre la classe parente ultime: `System.Object`

 Pour conclure ce chapitre, j’aimerais **examiner en détail la classe parente principale : `Object`**. Lors de votre lecture de la section précédente, vous avez peut-être remarqué que les classes de base de vos hiérarchies (`Car`, `Shape`, `Employee`) ne spécifient jamais explicitement leurs classes parentes.

```cs
// Qui est la classe parent de Car ?
class Car
{...}
```

**Dans l'univers .NET Core, chaque type hérite d'une classe de base nommée `System.Object`, représentable par le mot-clé C# `object` (o minuscule)**. ==La classe `Object` définit un ensemble de membres communs à chaque type du framework. En fait, lorsqu'une classe n'est pas explicitement définie, le compilateur la fait automatiquement dériver de `Object`==. Pour plus de clarté, vous pouvez définir des classes dérivées de `Object` comme suit (cependant, cela n'est pas nécessaire) :

```cs
// Ici, on dérive explicitement de System.Object.
class Car : object
{...}
```

Comme toute classe, ==`System.Object` définit un ensemble de membres==. Dans la définition C# formelle suivante, **notez que certains de ces éléments sont déclarés `virtual`, ce qui indique qu’un membre donné peut être redéfini par une sous-classe, tandis que d’autres sont marqués comme `static`** (et sont donc appelés au niveau de la classe) :

```cs
public class Object
{
    // membres Virtual
    public virtual bool Equals(object obj);

    protected virtual void Finalize();

    public virtual int GetHashCode();

    public virtual string ToString();

    // Niveau de l'instance, membres non-virtual.
    public Type GetType();

    protected object MemberwiseClone();

    // Membres Static
    public static bool Equals(object objA, object objB);

    public static bool ReferenceEquals(object objA, object objB);
}
```

Le [[#Tableau 6-1 Membres principaux de `System.Object`|Tableau 6-1]] présente un aperçu des fonctionnalités offertes par certaines des méthodes que vous êtes le plus susceptible d’utiliser.

##### Tableau 6-1: Membres principaux de `System.Object`

| Méthode d'instance de la classe Object | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| -------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Equals()`                             | ==Par défaut, cette méthode renvoie `true` uniquement si les éléments comparés font référence au même élément en mémoire==. Ainsi, `Equals()` est utilisée pour **comparer les références d'objets, et non l'état de l'objet**. ==Généralement, cette méthode est redéfinie pour renvoyer `true` uniquement si les objets comparés ont les mêmes valeurs d'état interne== (c'est-à-dire une sémantique basée sur les valeurs). **Notez que si vous redéfinissez `Equals()`, vous devez également redéfinir `GetHashCode()`, car ces méthodes sont utilisées en interne par les types `Hashtable` pour récupérer les sous-objets du conteneur**.<br><br>==Rappelez-vous également du [[Chapitre 4#Comprendre les types de valeur et les types de référence\|Chapitre 4]] que la classe `ValueType` redéfinit cette méthode pour toutes les structures, afin qu'elles fonctionnent avec des comparaisons basées sur les valeurs==. |
| `Finalize()`                           | Pour l'instant, retenez que ==cette méthode (lorsqu'elle est surchargée) est appelée pour libérer les ressources allouées avant la destruction de l'objet==. J'aborde plus en détail les services de récupération de mémoire de CoreCLR au [[Chapitre 9#Types de récupération de mémoire\|Chapitre 9]].                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `GetHashCode()`                        | Cette méthode renvoie un `int` qui identifie une instance d'objet spécifique.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `ToString()`                           | **Cette méthode renvoie une représentation sous forme de chaîne de caractères de cet objet, au format `<espace de noms>.<nom du type>`** (appelé *nom pleinement qualifié*). ==Cette méthode est souvent redéfinie par une sous-classe pour renvoyer une chaîne de caractères tokenisée composée de paires nom-valeur représentant l'état interne de l'objet, plutôt que son nom pleinement qualifié==.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `GetType()`                            | **Cette méthode renvoie un objet `Type` qui décrit entièrement l'objet auquel vous faites actuellement référence**. En bref, il s'agit d'une ==méthode d'identification de type à l'exécution== (RTTI) ==disponible pour tous les objets== (voir le [[Chapitre 17#La classe `System.Type`\|Chapitre 17]] pour plus de détails).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `MemberwiseClone()`                    | **Cette méthode permet de renvoyer une copie membre par membre de l'objet courant, ce qui est souvent utilisé lors du clonage d'un objet** (voir [[Chapitre 8\|Chapitre 8]]).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |

Pour illustrer certains comportements par défaut de la classe de base `Object`, créez un projet d'application console C# final nommé *ObjectOverrides*. Insérez un nouveau type de classe C# contenant la définition de classe vide suivante pour un type nommé `Person` :

```cs
namespace ObjectOverrides;

// Souvenez vous! Person étend Object.
class Person { }
```

Mettez maintenant à jour vos instructions de niveau supérieur pour interagir avec les membres hérités de `System.Object` comme suit :

```cs
using ObjectOverrides;

Console.Title = "Fun with System.Object";
Console.WriteLine("**** Fun with System.Object ****\n");

Person p1 = new Person();

// Utiliser les membres hérités de System.Object.
Console.WriteLine("ToString: {0}", p1.ToString());
Console.WriteLine("Hash code: {0}", p1.GetHashCode());
Console.WriteLine("Type: {0}", p1.GetType());

// Créer d'autre références à p1.
Person p2 = p1;
object o = p2;

// Les références pointent-elles vers le même objet en mémoire ?
if (o.Equals(p1) && p2.Equals(o))
{
    Console.WriteLine("Same instance!");
}
Console.ReadLine();
```

Voici le résultat du code actuel :

```
**** Fun with System.Object ****

ToString: ObjectOverrides.Person
Hash code: 58225482
Type: ObjectOverrides.Person
Same instance!
```

**Notez que l'implémentation par défaut de `ToString()` renvoie le nom complet du type actuel (`ObjectOverrides.Person`)**. Comme vous le verrez plus loin lors de l'étude de la création d'espaces de noms personnalisés au [[Chapitre 16#Définition des espaces de noms personnalisés (MaJ C 10.0)|Chapitre 16]], ==chaque projet C# définit un « espace de noms racine », qui porte le même nom que le projet lui-même==. Ici, vous avez créé un projet nommé *ObjectOverrides*; ainsi, le type `Person` et le fichier *Program.cs* sont tous deux placés dans l'espace de noms `ObjectOverrides`.

**Le comportement par défaut de `Equals()` est de vérifier si deux variables pointent vers le même objet en mémoire**. Ici, vous créez une nouvelle variable `Person` nommée `p1`. À ce stade, un nouvel objet `Person` est placé sur le tas managé. p2 est également de type Person. ==Cependant, vous ne créez pas une *nouvelle* instance, mais vous assignez plutôt cette variable à une référence à `p1`==. Par conséquent, ==`p1` et `p2` pointent toutes deux vers le même objet en mémoire, de même que la variable `o`== (de type object, ajoutée par précaution). **Étant donné que `p1`, `p2` et `o` pointent tous vers la même adresse mémoire, le test d'égalité réussit**.

Bien que le comportement standard de `System.Object` puisse convenir dans de nombreux cas, **il est fréquent que vos types personnalisés redéfinissent certaines de ces méthodes héritées**. À titre d'exemple, mettez à jour la classe `Person` pour prendre en charge des propriétés représentant le prénom, le nom et l'âge d'une personne, chacune pouvant être définie par un constructeur personnalisé.

```cs
namespace ObjectOverrides;

// Souvenez vous! Person étend Object.
class Person
{
    public string FirstName { get; set; } = "";
    public string LastName { get; set; } = "";
    public int Age { get; set; }

    public Person(string fName, string lName, int age)
    {
        FirstName = fName;
        LastName = lName;
        Age = age;
    }

    public Person() { }
}
```

## Surcharge de `System.Object.ToString()`

De nombreuses classes (et structures) que vous créez peuvent **bénéficier de la surcharge de la méthode `ToString()` pour renvoyer une chaîne de caractères représentant textuellement l'état actuel du type**. ==Cela peut s'avérer très utile pour le déboggage== (entre autres raisons). La manière dont vous choisissez de construire cette chaîne est une question de préférence personnelle ; cependant, **une approche recommandée consiste à séparer chaque paire nom-valeur par des points-virgules et à encadrer la chaîne entière entre crochets** (de nombreux types des bibliothèques de classes de base .NET Core suivent cette approche). Voici un exemple de surcharge de la méthode `ToString()` pour votre classe `Person` :

```cs
public override string ToString() =>
    $"[First Name: {FirstName}; Last Name: {LastName}; Age: {Age}]";
```

Cette implémentation de `ToString()` est assez simple, étant donné que la classe `Person` ne possède que trois données d'état. Cependant, ==n'oubliez jamais qu'une redéfinition correcte de `ToString()` doit également prendre en compte toutes les données définies en amont dans la chaîne d'héritage``.

**Lorsque vous redéfinissez `ToString()` pour une classe étendant une classe de base personnalisée, la première étape consiste à obtenir la valeur de `ToString()` de votre parent à l'aide du mot-clé `base`**. Une fois les données de chaîne de caractères de votre parent obtenues, vous pouvez ajouter les informations personnalisées de la classe dérivée.

## Redéfinition de `System.Object.Equals()`

**Redéfinissons également le comportement de `Object.Equals()` pour qu'il fonctionne avec une sémantique basée sur les valeurs**. Rappelons que, ==par défaut, `Equals()` renvoie `true` uniquement si les deux objets comparés référencent la même instance d'objet en mémoire==. Pour la classe `Person`, il peut être utile d'implémenter `Equals()` pour qu'il renvoie `true` si les deux variables comparées contiennent les mêmes valeurs d'état (par exemple, prénom, nom et âge).

==Tout d'abord, notez que l'argument entrant de la méthode `Equals()` est un `System.Object` générique==. De ce fait, **votre première priorité est de vous assurer que l'appelant a bien passé un objet `Person` et, par mesure de sécurité supplémentaire, de vérifier que le paramètre entrant n'est pas une référence `null`**. 

Une fois que vous avez vérifié que l'appelant vous a bien passé un objet `Person` alloué, ==une approche pour implémenter `Equals()` consiste à effectuer une comparaison champ par champ entre les données de l'objet entrant et les données de l'objet courant==.

```cs
public override bool Equals(object? obj)
{
    // Avant C# 9.0
    // if (!(obj is Person temp))
    if (obj is not Person temp)
        return false;
    if (
        temp.FirstName == this.FirstName
        && temp.LastName == this.LastName
        && temp.Age == this.Age
    )
        return true;
    return false;
}
```

Ici, vous comparez les valeurs de l'objet entrant à vos valeurs internes (notez l'utilisation du mot-clé `this`). Si les noms et l'âge sont identiques, vous avez deux objets avec les mêmes données d'état et, par conséquent, la fonction renvoie `true`. Dans tous les autres cas, elle renvoie `false`.

==Bien que cette approche fonctionne, on imagine aisément la complexité de la tâche : implémenter une méthode `Equals()` personnalisée pour des types complexes pouvant contenir des dizaines de champs de données==. Un raccourci courant consiste à **utiliser sa propre implémentation de `ToString()`**. ==Si une classe possède une implémentation correcte de `ToString()` qui prend en compte toutes les données de champ dans la hiérarchie d'héritage, il suffit de comparer les données de chaîne de l'objet== (en vérifiant la présence de valeurs nulles).

```cs
// Plus besoin de convertir « obj » en Person,
// car tout possède une méthode ToString().
public override bool Equals(object obj) => obj?.ToString() == ToString();
```

Notez ==dans ce cas qu'il n'est plus nécessaire de vérifier si l'argument entrant est du type correct== (une personne, dans cet exemple), ==car tout en .NET prend en charge la méthode `ToString()`==. Mieux encore, ==il n'est plus nécessaire d'effectuer une vérification d'égalité propriété par propriété, car il suffit désormais de tester la valeur renvoyée par `ToString()`==.

>[!warning] C'est un très mauvaise pratique !!!
>Ici, l'auteur du livre démontre deux choses:
>- Faire comprendre le concept de **la classe de base ultime**.
>- Grâce à l'héritage, il existe un **"contrat universel"** : tout objet en .NET garantit la présence de `ToString()`, `Equals()`, et `GetType()`.
>>[!example] Petite analogie de pourquoi c'est une mauvaise pratique (généré par Gemini)
>>Imaginez que vous travaillez dans une usine de recyclage de verre.
>>
>>- Le commentaire du livre dit : _"Plus besoin de vérifier si c'est une bouteille, car tout ce qui entre ici peut être broyé."_
>>- Techniquement, c'est vrai. Mais si quelqu'un jette un bloc de métal (un autre type d'objet), votre broyeuse va essayer de le traiter. Elle ne "plantera" peut-être pas tout de suite, mais le résultat final (le verre recyclé) sera gâché.

## Redéfinition de `System.Object.GetHashCode()`

**Lorsqu'une classe redéfinit la méthode `Equals()`, vous devez également redéfinir l'implémentation par défaut de `GetHashCode()`**. En résumé, **un code de hachage est une valeur numérique qui représente un objet dans un état particulier**. ==Par exemple, si vous créez deux variables `string` contenant la valeur `Hello`, vous obtiendrez le même code de hachage==. Cependant, ==si l'une des chaînes était entièrement en minuscules (`hello`), vous obtiendriez des codes de hachage différents==.

**Par défaut, `System.Object.GetHashCode()` utilise l'emplacement mémoire actuel de votre objet pour calculer la valeur de hachage**. Toutefois, ==si vous créez un type personnalisé que vous prévoyez de stocker dans un type `Hashtable`== (dans l'espace de noms `System.Collections`), ==vous devez toujours redéfinir ce membre, car le `Hashtable` appellera en interne les méthodes `Equals()` et `GetHashCode()` pour récupérer l'objet correct==.

>[!info]- Plus précisément
>La classe `System.Collections.Hashtable` appelle `GetHashCode()` en interne pour avoir une idée générale de l'emplacement de l'objet, mais un appel (interne) ultérieur à `Equals()` détermine la correspondance exacte.

Bien que vous n'alliez pas placer votre objet `Person` dans un `System.Collections.Hashtable` dans cet exemple, ==par souci d'exhaustivité, redéfinissons la méthode `GetHashCode()`==. **De nombreux algorithmes permettent de créer un code de hachage** ; certains sophistiqués, d'autres plus simples. ==La plupart du temps, vous pouvez générer une valeur de code de hachage en utilisant l'implémentation `GetHashCode()` de `System.String`==.

==Étant donné que la classe `String` possède déjà un algorithme de hachage robuste qui utilise les données de caractères du `string` pour calculer une valeur de hachage, si vous pouvez identifier une donnée de champ dans votre classe qui doit être unique pour toutes les instances== (comme un numéro de sécurité sociale), ==il vous suffit d'appeler `GetHashCode()` sur cette donnée==. Ainsi, si la classe Person définit une propriété `SSN`, vous pouvez écrire le code suivant :

```cs
class Person
{
	...
	
	public string SSN { get; } = "";
    public Person(string fName, string lName, int age, string ssn)
    {
        FirstName = fName;
        LastName = lName;
        Age = age;
        SSN = ssn;
    }
    
    ...
    
    // Retourne un code haché basé sur des donnée de caractères uniques.
    public override int GetHashCode() => SSN.GetHashCode();
}
```

**Si vous utilisez une propriété en lecture-écriture comme base du code de hachage, vous recevrez un avertissement**. ==Une fois l'objet créé, le code de hachage doit être immuable==. Dans l'exemple précédent, la propriété `SSN` ne possède qu'une méthode `get`, ce qui la rend en lecture seule et elle ne peut être définie que dans le constructeur.

**Si vous ne trouvez pas de données de chaîne uniques, mais que vous avez redéfini la méthode `ToString()`** (qui respecte la convention de lecture seule), **appelez `GetHashCode()` sur votre propre représentation sous forme de chaîne**.

```cs
// Retoure un code haché basé sur la valeur ToString() de l'objet.
public override int GetHashCode() => ToString().GetHashCode();
```

## Test de votre classe `Person` modifiée

Maintenant que vous avez redéfini les membres `virtual` de `Object`, mettez à jour les instructions de niveau supérieur pour tester vos modifications.

```cs
using ObjectOverrides;

Console.Title = "Fun with System.Object";
Console.WriteLine("**** Fun with System.Object ****\n");

// NOTE: Nous voulons que ces valeurs soient identiques pour tester
// les méthodes Equals() et GetHashCode().
Person p1 = new Person("Homer", "Simpson", 50, "111-11-1111");
Person p2 = new Person("Homer", "Simpson", 50, "111-11-1111");

// Obtenir la version sous forme de string des objets.
Console.WriteLine("p1.ToString() = {0}", p1.ToString());
Console.WriteLine("p2.ToString() = {0}", p2.ToString());

// Test de la méthode Equals() surchargée.
Console.WriteLine("p1 = p2?: {0}", p1.Equals(p2));

// Test des codes de hachage.
// Utilisation du hachage de SSN.
Console.WriteLine(
    "Same hash codes?: {0}",
    p1.GetHashCode() == p2.GetHashCode()
);
Console.WriteLine();

// Change la valeur dans Age pour p2 et test encore.
p2.Age = 45;
Console.WriteLine("p1.ToString() = {0}", p1.ToString());
Console.WriteLine("p2.ToString() = {0}", p2.ToString());
Console.WriteLine("p1 = p2?: {0}", p1.Equals(p2));

// Utilise toujours le hachage de SSN.
Console.WriteLine(
    "Same hash codes?: {0}",
    p1.GetHashCode() == p2.GetHashCode()
);
Console.ReadLine();
```

Le résultat est affiché ici :

```
**** Fun with System.Object ****

p1.ToString() = [First Name: Homer; Last Name: Simpson; Age: 50]
p2.ToString() = [First Name: Homer; Last Name: Simpson; Age: 50]
p1 = p2?: True
Same hash codes?: True

p1.ToString() = [First Name: Homer; Last Name: Simpson; Age: 50]
p2.ToString() = [First Name: Homer; Last Name: Simpson; Age: 45]
p1 = p2?: False
Same hash codes?: True
```

> La dernière ligne montre directement le pourquoi l'avertissement précédent est nécessaire.

## Utilisation des membres statiques de `System.Object`

Outre les membres d'instance que vous venez d'examiner, **`System.Object` définit deux membres statiques qui testent également l'égalité par valeur ou par référence**. Considérez le code suivant :

```cs
static void StaticMembersOfObject()
{
    // Membres static de System.Object.
    Person p3 = new Person("Sally", "Jones", 4);
    Person p4 = new Person("Sally", "Jones", 4);
    Console.WriteLine("P3 and P4 have same state: {0}", object.Equals(p3, p4));
    Console.WriteLine(
        "P3 and P4 are pointing to same object: {0}",
        object.ReferenceEquals(p3, p4)
    );
}
```

**Ici, il vous suffit de transmettre deux objets** (de n'importe quel type) **et de laisser la classe `System.Object` déterminer automatiquement les détails**. 

Le résultat (lorsqu'elle est appelée depuis les instructions de niveau supérieur) est affiché ici :

```
***** Fun with System.Object *****

...

P3 and P4 have same state: True
P3 and P4 are pointing to same object: False
```

# Résumé du chapitre

Ce chapitre a exploré **le rôle et les détails de l'héritage et du polymorphisme**. Au fil des pages, vous avez découvert de nombreux nouveaux mots-clés et jetons permettant de prendre en charge chacune de ces techniques. Par exemple, ==rappelez-vous que le jeton deux-points est utilisé pour définir la classe parente d'un type donné==. **Les types parents peuvent définir un nombre quelconque de membres virtuels et/ou abstraits afin d'établir une interface polymorphe**. ==Les types dérivés redéfinissent ces membres à l'aide du mot-clé `override`==.

Outre la construction de nombreuses hiérarchies de classes, ce chapitre a également **examiné comment effectuer des conversions explicites entre types de base et types dérivés et s'est conclu par une exploration détaillée de la classe parente globale dans les bibliothèques de classes de base .NET : `System.Object`**.

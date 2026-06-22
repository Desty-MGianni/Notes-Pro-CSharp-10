---
title: "Chapitre 5: Comprendre l'Encapsulation"
publish: true
---

# <big><big><big><b><font color =green>Comprendre l'Encapsulation</font></b></big></big></big>

Dans les chapitres [[Chapitre 3|3]] et [[Chapitre 4|4]], vous avez étudié plusieurs constructions syntaxiques fondamentales, courantes dans toute application .NET Core que vous développez. *Vous commencerez ici à examiner les fonctionnalités orientées objet de C#*. **La première étape consiste à examiner le processus de création de types de classes bien définis, prenant en charge un nombre illimité de *constructeurs***. Après avoir compris les bases de la définition des classes et de l'allocation d'objets, **la suite de ce chapitre examinera le rôle de l'*encapsulation***. Vous apprendrez ainsi à définir les propriétés de classe et à comprendre les détails du mot-clé `static`, de la syntaxe d'initialisation d'objet, des champs en lecture seule, des données constantes et des classes partielles.

# Présentation du type classe C# 

Sur la plateforme .NET, le *type de classe* est l'une des constructions de programmation les plus fondamentales. Formellement, ==une classe est un type défini par l'utilisateur, composé de données de champ== (souvent appelées *variables de membres*) ==et de membres qui agissent sur ces données== (tels que les constructeurs, les propriétés, les méthodes, les événements, etc.). **Collectivement, l'ensemble des données de champ représente l' "état" d'une instance de classe (autrement appelée *objet*).** ==La puissance des langages orientés objet, comme C#, réside dans le fait qu'en regroupant les données et les fonctionnalités associées dans une définition de classe unifiée, vous pouvez modéliser votre logiciel à partir d'entités du monde réel==.

Pour commencer, créez un nouveau projet d'application console C# nommé *SimpleClassExample*. Ensuite, insérez un nouveau fichier de classe (nommé *Car.cs*) dans votre projet. Dans ce nouveau fichier, ajoutez l'espace de noms suivant :

```cs
namespace SimpleClassExample;
```

En C#, une classe est définie à l'aide du mot-clé `class`. Voici la déclaration la plus simple possible (veillez à ajouter la déclaration de classe après l'espace de noms `SimpleClassExample`) :

```cs
class Car
{
}
```

Après avoir défini un type de classe, **vous devrez considérer l'ensemble des variables membres qui représenteront son état**. Par exemple, ==vous pourriez décider que les voitures utilisent un type de données `int` pour représenter leur vitesse actuelle et un type de données `string` pour représenter leur nom d'animal familier==. Compte tenu de ces notes de conception initiales, mettez à jour votre classe `Car` comme suit :

```cs
class Car
{
    // L'"état" de l'objet Car. 
    public string petName;
    public int curSpeed;
}
```

**Notez que ces variables membres sont déclarées à l'aide du modificateur d'accès `public`**. Les membres publics d'une classe ==sont directement accessibles dès la création d'un objet de ce type==. Rappelons que **le terme *objet* désigne une instance d'un type de classe donné, créée à l'aide du mot-clé `new`**.

>[!Tip] Bonnes pratiques
>**Les données de champ d'une classe doivent rarement (*voire jamais*) être définies comme publiques**. Pour préserver l'intégrité de vos données d'état, ==il est préférable de les définir comme `private` (ou éventuellement `protected`) et d'autoriser un accès contrôlé via les *propriétés*== (comme indiqué plus loin dans ce chapitre). 
>
>Cependant, pour simplifier au maximum ce premier exemple, les données publiques conviennent parfaitement.

Après avoir défini l'ensemble des variables membres représentant l'état de la classe, l'étape de conception suivante consiste à définir les membres qui modélisent son comportement. Dans cet exemple, la classe `Car` définira une méthode nommée `SpeedUp()` et une autre nommée `PrintState()`. Mettez à jour votre classe comme suit :

```cs
class Car
{
    // L'"état" de l'objet Car.
    public string petName;
    public int currSpeed;

    // Les fonctionnalité de l'objet Car.
    // Ici, on utilise la syntaxe de coprs de membres
    // expliqué dans le chapitre 4.
    public void PrintState() =>
        Console.WriteLine($"{petName} is going {currSpeed} km/h.");

    public void SpeedUp(int delta) => currSpeed += delta;
}
```

`PrintState()` est plus ou moins une fonction de diagnostic qui affiche simplement l'état actuel d'un objet `Car` dans la fenêtre de commande. `SpeedUp()` augmente la vitesse de l'objet `Car` de la valeur spécifiée par le paramètre `int` entrant. Mettez à jour vos instructions de niveau supérieur dans le fichier *Program.cs* avec le code suivant :

```cs
using SimpleClassExample;

Console.Title = "Fun with Class Types";
Console.WriteLine("***** Fun with Class Types *****\n");

// Alloue et configure un objet Car.
Car myCar = new Car();
myCar.petName = "Henry";
myCar.currSpeed = 10;

// accelère la voiture quelque fois et affiche 
// le nouvel état de l'objet.
for (int i = 0; i <= 10; i++)
{
    myCar.SpeedUp(5);
    myCar.PrintState();
}
Console.ReadLine();
```

Après avoir exécuté votre programme, vous constaterez que la variable `Car` (`myCar`) conserve son état actuel tout au long de l'exécution de l'application, comme illustré dans le résultat suivant :

```
***** Fun with Class Types *****

Henry is going 15 KPH.
Henry is going 20 KPH.
Henry is going 25 KPH.
Henry is going 30 KPH.
Henry is going 35 KPH.
Henry is going 40 KPH.
Henry is going 45 KPH.
Henry is going 50 KPH.
Henry is going 55 KPH.
Henry is going 60 KPH.
Henry is going 65 KPH.
```

## Allocation d'objets avec le mot-clé `new`

Comme indiqué dans l'exemple de code précédent, **les objets doivent être alloués en mémoire à l'aide du mot-clé `new`**. Si vous n'utilisez pas le mot-clé `new` et tentez d'utiliser votre variable de classe dans une instruction de code ultérieure, vous recevrez une erreur de compilation. Par exemple, l'instruction de niveau supérieur suivante ne compilera pas :

```cs
Console.WriteLine("***** Fun with Class Types *****\n");
// Erreur de compilation! Oubli du mot clé 'new' pour créer un objet!.
Car myCar;
myCar.petName = "Fred";
```

Pour créer correctement un objet à l'aide du mot-clé `new`, vous pouvez définir et allouer un objet `Car` sur une seule ligne de code.

```cs
Console.WriteLine("***** Fun with Class Types *****\n");
// Définition d'une variable locale = Allocation mémoire
Car myCar = new Car();
// Assignation de "Fred" au champ petName
myCar.petName = "Fred";
```

Alternativement, si vous souhaitez définir et allouer une instance de classe sur des lignes de code distinctes, vous pouvez procéder comme suit :

```cs
Console.WriteLine("***** Fun with Class Types *****\n");
// Définition d'une variable Car.
Car myCar;
// Allocation mémoire de la taille d'un objet Car.
myCar = new Car();
// Assignation de "Fred" au champ petName.
myCar.petName = "Fred";
```

Ici, la première instruction de code déclare simplement une référence à un objet `Car` pas encore déterminé. **Ce n'est qu'après avoir assigné une référence à un objet que cette référence pointe vers un objet valide en mémoire**.

Quoi qu'il en soit, à ce stade, vous disposez d'une classe simple qui définit quelques points de données et quelques opérations de base. Pour améliorer les fonctionnalités de la classe `Car` actuelle, vous devez comprendre le rôle des *constructeurs*.

>[!note]- Petit parallèle avec le C (avec une partie générée par Gemini)
> le mot clé `new` est une version moderne et complexe de la fonction `calloc`. en C, `calloc` permet d'allouer une plage mémoire, avec la taille déterminée par le développeur. En plus, `calloc` nettoie toute les addresses en les remettant à la valeur `0` (ce que la fonction `malloc` ne fait pas).
>
> Même si le nettoyage de la mémoire est identique, `new` fait deux étapes supplémentaires cruciales que `calloc` ignore totalement : 
>
>1. **L'appel au Constructeur** : `calloc` se contente de rendre un bloc de mémoire propre. `new` exécute du code (le constructeur) pour donner un état logique à l'objet.
>2. **L'enregistrement des métadonnées** : C# ajoute un "en-tête" invisible à chaque objet (Type Handle et Sync Block Index) pour que le Garbage Collector et le système de types sachent ce que c'est. `calloc` ne manipule que des octets bruts.

# Comprendre les constructeurs

Étant donné que les objets possèdent un état (représenté par les valeurs des variables membres), *un programmeur souhaite généralement attribuer des valeurs pertinentes aux données des champs de l'objet avant utilisation*. Actuellement, la classe `Car` exige que les champs `petName` et `currSpeed` ​​soient attribués champ par champ. Dans notre exemple, cela ne pose pas de problème majeur, car nous n'avons que deux points de données publics. Cependant, **il n'est pas rare qu'une classe gère des dizaines de champs. Il serait donc peu judicieux d'écrire 20 instructions d'initialisation pour définir 20 points de données !**

Heureusement, C# prend en charge l'utilisation de *constructeurs*, ==qui permettent d'établir l'état d'un objet dès sa création==. **Un constructeur est une méthode spéciale d'une classe appelée indirectement lors de la création d'un objet à l'aide du mot-clé `new`**. Cependant, contrairement à une méthode « normale », **les constructeurs n'ont jamais de valeur de retour (*même pas `void`*) et portent toujours le même nom que la classe qu'ils construisent**.

## Comprendre le rôle du constructeur par défaut

Chaque classe C# est fournie avec un *constructeur par défaut* "gratuit" que vous pouvez redéfinir si nécessaire. **Par définition, un constructeur par défaut ne prend jamais d'arguments**. Après avoir alloué le nouvel objet en mémoire, ==le constructeur par défaut garantit que toutes les données des champs de la classe sont définies avec une valeur par défaut appropriée== (voir le [[Chapitre 3#Valeur littérale par défaut (Nouveauté C 7.1)|Chapitre 3]] pour plus d'informations sur les valeurs par défaut des types de données C#).

**Si ces affectations par défaut ne vous conviennent pas, vous pouvez redéfinir le constructeur par défaut selon vos besoins**. Par exemple, mettez à jour votre classe C# Car comme suit :

>[!note]- Dichotomie entre les exemples et le code réel allant avec le livre.
> Les exemples de constructeurs montrés plus bas seront différent du code. Les exemples viennent du livre tandis que le code (le projet *SimpleClassExample*) utilise les syntaxes plus moderne (une note survenant plus loin expliquera en détail ses différences)

```cs
class Car
{
	// L'état d'un objet Car.
	public string petName;
	public int currSpeed;
	
	// Un constructeur par défaut personnalisé.
	public Car()
	{
		petName = "Chuck";
		currSpeed = 10;
	}
	...
}
```

Dans ce cas, vous forcez tous les objets `Car`  à porter le nom `Chuck` et à avoir une vitesse de 10 km/h. Vous pouvez ainsi créer un objet `Car` avec les valeurs par défaut suivantes :

```cs
Console.WriteLine("***** Fun with Class Types *****\n");
...
// Invoking the default constructor
Car chuck = new Car();

// Prints "Chuck is going 10 KPH."
chuck.PrintState();
```

## Définition de constructeurs personnalisés

**Généralement, les classes définissent des constructeurs supplémentaires en plus de ceux par défaut**. Ce faisant, ==vous fournissez à l'utilisateur de l'objet un moyen simple et cohérent d'initialiser l'état d'un objet directement lors de sa création==. Prenons l'exemple de la mise à jour suivante de la classe `Car`, qui prend désormais en charge trois constructeurs :

```cs
class Car
{
	// L'"état" de l'objet Car
	public string petName;
	public int currSpeed;
	
	// Un constructeur par défaut personnalisé.
	public Car()
	{
		petName = "Chuck";
		currSpeed = 10;
	}
	
	// Ici, currSpeed recevra la vaeur par défaut
	// du type de donnée int (zéro) 
	public Car(string pn)
	{
		petName = pn;
	}
	
	// Laisse l'appelant définir l'état complet de l'objet Car.
	public Car(string pn, int cs)
	{
		petName = pn;
		currSpeed = cs;
	}
	
	// Les fonctionnalité de l'objet Car.
	// Ici, on utilise la syntaxe de coprs de membres
	// expliqué dans le chapitre 4.
	public void PrintState() =>
		Console.WriteLine($"{name} is going {speed} km/h.");
	
	public void SpeedUp(int delta) => speed += delta;
}
```

**Gardez à l'esprit que ce qui différencie un constructeur d'un autre (aux yeux du compilateur C#) réside dans le nombre et/ou le type de ses arguments**. Rappelez vous du [[Chapitre 4#Comprendre la surcharge de méthodes|Chapitre 4]], ==lorsque vous définissez une méthode portant le même nom et dont le nombre ou le type d'arguments diffère, vous *surchargez* la méthode==. **Ainsi, la classe `Car` a surchargé le constructeur afin de proposer plusieurs façons de créer un objet dès sa déclaration**. Quoi qu'il en soit, vous pouvez désormais créer des objets `Car` à l'aide de n'importe quel constructeur public. Voici un exemple :

```cs
Console.WriteLine("***** Fun with Class Types *****\n");
...
// Crée un objet Car appelé "Chuck" allant à 10 km/h.
Car chuck = new();
chuck.PrintState();

// Crée un objet Car appelé "Mary" allant à 0 km/h.
Car mary = new Car("Mary");
mary.PrintState();

// Crée un objet Car appelé "Daisy" allant à 75 km/h.
Car daisy = new Car(75, "daisy");
daisy.PrintState();
```

>[!tip] Rappel
>De la même manière que pour les `struct`s, le constructeur primaire est aussi utilisable pour les classes. Voir le [[Chapitre 4#Le constructeur primaires de structure (Nouveauté C 12)|Chapitre 4]] pour les détails.

### Constructeurs comme membres à corps d'expression (Nouveauté C# 7.0)

C# 7 a ajouté des utilisations supplémentaires pour le style de membre à corps d'expression. **Les constructeurs, finaliseurs et accesseurs get/set sur les propriétés et les indexeurs acceptent désormais la nouvelle syntaxe**. Ainsi, les constructeurs précédents peuvent être écrits ainsi :

```cs
// Ici, currSpeed recevra la valeur
// par défaut pour un int (zéro)
public Car(string pn) => petName = pn;
```

**Le deuxième constructeur personnalisé ne peut pas être converti en expression, car les membres contenant une expression doivent être des méthodes *sur une seule ligne***.

### Constructeurs avec paramètres `out` (Nouveauté C# 7.3)

**Les constructeurs (ainsi que les initialiseurs de champs et de propriétés, abordés plus loin) peuvent utiliser des paramètres `out` à partir de C# 7.3**. Pour un exemple simple, ajoutez le constructeur suivant à la classe `Car` :

```cs
public Car(string pn, int cs, out bool inDanger)
{
	 petName = pn;
	 currSpeed = cs;
	 if (cs > 100)
	     inDanger = true;
	 else
		 inDanger = false;
}
```

**Toutes les règles des paramètres de sortie doivent être respectées. Dans cet exemple, une valeur doit être attribuée au paramètre `inDanger` avant la fin du constructeur.**

## Revisite du constructeur par défaut

Comme vous venez de l'apprendre, **toutes les classes disposent d'un constructeur par défaut gratuit**. Insérez un nouveau fichier dans votre projet, nommé *Motorcycle.cs*, et ajoutez ce qui suit pour définir une classe `Motorcycle` :

```cs
namespace SimpleClassExample;

class Motorcycle
{
    public void PopWheely()
    {
		Console.WriteLine("Yeeeeee Haaaaeeewwww!");
    }   
}
```

Vous pouvez désormais créer une instance du type `Motorcycle` via le constructeur par défaut prêt à l'emploi.

```cs
Console.WriteLine("***** Fun with Class Types *****\n");
...
Motorcycle mc = new Motorcycle();
mc.PopWheely();
```

Cependant, ==dès que vous définissez un constructeur personnalisé avec un nombre quelconque de paramètres, le constructeur par défaut est silencieusement supprimé de la classe et n'est plus disponible==. Imaginez : **si vous ne définissez pas de constructeur personnalisé, le compilateur C# vous attribue une valeur par défaut pour permettre à l'utilisateur de l'objet d'allouer une instance de votre type avec les valeurs par défaut correctes pour le champ**. ==Cependant, lorsque vous définissez un constructeur unique, le compilateur suppose que vous avez pris les choses en main==.

**Quand vous déclarer des constructeurs personnalisé, vous DEVEZ déclarer le constructeur par défaut.** Voici quelques modifications apportées à la catégorie `Motorcycle` :


Par conséquent, si vous souhaitez permettre à l'utilisateur de l'objet de créer une instance de votre type avec le constructeur par défaut, ainsi qu'avec votre constructeur personnalisé, ==vous devez redéfinir *explicitement* le constructeur par défaut==. À cette fin, il faut comprendre que **dans la grande majorité des cas, l'implémentation du constructeur par défaut d'une classe est volontairement vide, car vous avez simplement besoin de pouvoir créer un objet avec des valeurs par défaut**. Prenons l'exemple de la mise à jour suivante de la classe `Motorcycle`

```cs
	class Motorcycle
	{
	public int driverIntensity;
	public void PopWheely()
	{
		for (int i = 0; i <= driverIntensity; i++)
			Console.WriteLine("Yeeeeee Haaaaeeewwww!");
	}
	
	// Redéfini manuellement le constructeur par défaut, qui
	// définira toutes les valeurs par défaut des membres
	public Motorcycle() { }
	
	// Notre constructeur personnalisé. 
	public Motorcycle(int intensity) => 
		driverIntensity = intensity;
	}
```

>[!Tip] Petite astuce
>Maintenant que vous comprenez mieux le rôle des constructeurs de classe, voici un raccourci pratique. Visual Studio et Visual Studio Code fournissent tous deux le *snippet* `ctor`. Lorsque vous saisissez `ctor` et appuyez sur la touche *Tab*, l'IDE définit automatiquement un constructeur par défaut personnalisé. Vous pouvez ensuite ajouter des paramètres personnalisés et une logique d'implémentation.

# Comprendre le rôle du mot-clé `this`

C# fournit un mot-clé `this` qui ==donne accès à l'instance de classe actuelle==. Ce mot-clé peut notamment **servir à résoudre une ambiguïté de portée, qui peut survenir lorsqu'un paramètre entrant porte le même nom qu'un champ de données de la classe**. ==Cependant, vous pouvez simplement adopter une convention de nommage qui évite cette ambiguïté==; pour illustrer cette utilisation du mot-clé `this`, mettez à jour votre classe `Motorcycle` avec un nouveau champ de chaîne (nommé `name`) pour représenter le nom du conducteur. Ensuite, ajoutez une méthode nommée `SetDriverName()` implémentée comme suit :

```cs
class Motorcycle
{
    public int driverIntensity;

    // Nouveau membre représentant le nom du conducteur.
    public string name;
    public void SetDriverName(string name) => name = name;
    ...
}
```

==Bien que ce code va compilé, le compilateur C# affichera un message d'avertissement vous informant que vous avez réaffecté une variable à elle-même !== Pour illustrer cela, modifiez votre code pour appeler `SetDriverName()`, puis affichez la valeur du champ `name`. Vous pourriez être surpris de constater que la valeur du champ `name` est une chaîne vide !

```cs
// Make a Motorcycle with a rider named Tiny?
Motorcycle c = new Motorcycle(5);
c.SetDriverName("Tiny");
c.PopWheely();
Console.WriteLine($"Rider name is {c.name}"); // Prints an empty name value!
```

Le problème est que l'implémentation de `SetDriverName()` s'*attribue à lui même le paramètre entrant*, ==car le compilateur suppose que le nom fait référence à la variable actuellement dans la portée de la méthode plutôt qu'au champ nom de la portée de la classe==. **Pour indiquer au compilateur que vous souhaitez définir le champ de données nom de l'objet actuel sur le paramètre nom entrant, utilisez simplement cette méthode pour lever l'ambiguïté**.

```cs
public void SetDriverName(string name) => this.name = name;
```

**En l'absence d'ambiguïté, vous n'êtes pas obligé d'utiliser le mot-clé `this` pour accéder aux champs ou aux membres de données**. Par exemple, si vous renommez le membre de type `string` `name` en `driverName` (ce qui nécessitera également la mise à jour des instructions de niveau supérieur), l'utilisation de `this` est facultative, car il n'y a plus d'ambiguïté de portée.

```cs
class Motorcycle
{
 	 public int driverIntensity;
 	 public string driverName;
 	
 	 public void SetDriverName(string name)
 	 {
 		 // Ces deux déclarations sont techniquement les mêmes.
 		 driverName = name;
 		 this.driverName = name;
 	 }
 	 ...
}
```

**Même si l'utilisation du mot-clé `this` dans des situations non ambiguës présente peu d'intérêt, il peut néanmoins s'avérer utile lors de l'implémentation de membres de classe, car des IDE tels que Visual Studio et Visual Studio Code activent *IntelliSense* lorsqu'il est spécifié. Cela peut s'avérer utile si vous avez oublié le nom d'un membre de classe et souhaitez en retrouver rapidement la définition.**

>[!tip] convention de nommage.
>Une pratique courante consiste à *commencer les noms des variables privées* (ou internes) *de classe par un trait de soulignement*  (par exemple, `_driverName`) *afin qu'IntelliSense affiche toutes vos variables en haut de la liste*. Dans notre exemple trivial, tous les champs sont publics ; cette convention de nommage ne s'applique donc pas. Dans la suite du livre, vous verrez les variables privées et internes nommées avec un trait de soulignement.

## Enchaîner des appels de constructeurs avec `this`

Le mot-clé `this` permet également de concevoir une classe à l'aide d'une technique appelée *enchaînement de constructeurs.* **Ce modèle de conception est utile lorsqu'une classe définit plusieurs constructeurs.** ==Étant donné que les constructeurs valident souvent les arguments entrants pour appliquer diverses règles métier, il est fréquent de trouver une logique de validation redondante dans l'ensemble des constructeurs d'une classe==. Prenons l'exemple de la mise à jour de `Motorcycle` suivante :

```cs
class Motorcycle
{
	 public int driverIntensity;
	 public string driverName;
	
	 public Motorcycle(){}
	
	// Logique de constructeur redondantes.
	public Motorcycle(int intensity)
	{
		if (intensity > 10)
		{
			intensity = 10;
		}
		driverIntensity = intensity;	
	}
	
	public Morotcycle (int intensity, string name)
	{
		if (intensity > 10)
		{
			intensity = 10;
		}
		driverIntensity = intensity;
		drivername = name;
	 }
 	 ...
}
```

==Ici== (peut-être pour garantir la sécurité du pilote), ==chaque constructeur s'assure que le niveau d'intensité ne dépasse jamais 10==. **Bien que cela soit correct, il existe des instructions de code redondantes dans deux constructeurs. Ce n'est pas idéal, car vous devez désormais mettre à jour le code à plusieurs endroits si vos règles changent** (par exemple, si l'intensité ne doit pas dépasser 5 au lieu de 10).

==Une solution pour améliorer la situation actuelle consiste à définir une méthode dans la classe `Motorcycle` qui validera le ou les arguments entrants. Ainsi, chaque constructeur pourrait appeler cette méthode avant d'effectuer les affectations de champs==. Bien que cette approche permette d'isoler le code à mettre à jour lorsque les règles métier changent, vous êtes confronté à la redondance suivante :

```cs
class Motorcycle
{
	public int driverIntensity;
	public string driverName;
	
	// Constructeurs.   
	public Motorcycle() { }
	
	// Rétablir le constructeur par défaut, qui
	// définira toutes les valeurs par défaut des membres de données.
	public Motorcycle(int intensity)
	{
		SetIntensity(intensity);
	}
	
	public Motorcycle(int intensity, string name)
	{
		SetIntensity(intensity);
		driverName = name;
	}
	
	public void SetIntensity(int intensity)
	{
		if (intensity > 10)
		{
			intensity = 10;
		}
		driverIntensity = intensity;
	}
	...
}
```

Une approche plus propre consiste à désigner le constructeur acceptant le *plus grand nombre d'arguments* comme « constructeur maître » et à confier à son implémentation la logique de validation requise. ==Les autres constructeurs peuvent utiliser le mot-clé `this` pour transmettre les arguments entrants au constructeur maître et fournir les paramètres supplémentaires nécessaires==. De cette façon, vous n'avez à vous soucier que de la maintenance d'un seul constructeur pour l'ensemble de la classe, les autres étant pratiquement vides.

Voici l'itération finale de la classe `Motorcycle` (avec un constructeur supplémentaire à titre d'illustration). Lors de l'enchaînement des constructeurs, notez que le mot-clé `this` est suspendu à la déclaration du constructeur (via un opérateur deux-points), hors de la portée du constructeur lui-même.

```cs
class Motorcycle
{
	public int driverIntensity;
	public string driverName;
	
	// Enchaînement de constructeurs.
	public Motorcycle() { }
	public Motorcycle(int intensity) : this(intensity, "") { }
	public Motorcycle(string name) : this(0, name) { }
	
	// Ceci est le constructeur "maitre" qui fera tout travail.
	public Motorcycle(int intensity, string name)
	{
		if (intensity > 10)
			intensity = 10;
		
		driverIntensity = intensity;
		driverName = name;
	}
	...
}
```

Il est important de comprendre que l'utilisation du mot-clé `this` pour enchaîner les appels de constructeur *n'est jamais obligatoire*. Cependant, ==cette technique permet d'obtenir une définition de classe plus facile à maintenir et plus concise. Elle simplifie également vos tâches de programmation, car le travail principal est délégué à un seul constructeur== (généralement celui qui possède le plus de paramètres), tandis que les autres constructeurs se renvoient la balle.

>[!note] Rappel
>Comme indiqué au [[Chapitre 4#Définir des paramètres optionnels|Chapitre 4]], que **C# prend en charge les paramètres optionnels**. *En utilisant des paramètres optionnels dans vos constructeurs de classes, vous obtiendrez les mêmes avantages que le chaînage de constructeurs avec moins de code*. Vous verrez comment procéder dans un instant.

> Cette façon de définir les constructeurs est ce qui donnera naissance aux *constructeurs primaires*.

## Observation du flux du constructeur

Enfin, sachez qu'une fois qu'un constructeur a transmis des arguments au constructeur maître désigné (et que ce dernier a traité les données), le constructeur invoqué initialement par l'appelant termine l'exécution des instructions de code restantes. Pour plus de clarté, mettez à jour chaque constructeur de la classe Motorcycle avec un appel approprié à `Console.WriteLine()`.

```cs
namespace SimpleClassExample;

class Motorcycle
{
	public int driverIntensity;
	public string driverName;
	
	// Enchaînement de constructeurs
	public Motorcycle()
	{
		Console.WriteLine("In default constructor");
	}
	
	public Motorcycle(int intensity) : this(intensity, "")
	{
		Console.WriteLine("In constructor taking an int");
	}
	
	public Motorcycle(string name) : this(0, name)
	{
		Console.WriteLine("In constructor taking a string");
	}
	
	// Ceci est le constructeur "maître" qui fera tout le traveil.
	public Motorcycle(int intensity, string name)
	{
		Console.WriteLine("In main constructor");
		
		if (intensity > 10)
			intensity = 10;
		
		driverIntensity = intensity;
		driverName = name;
	}
	
	public void SetIntensity(int intensity)
	{
		if (intensity > 10)
		{
			intensity = 10;
		}
		driverIntensity = intensity;
	}
	
	public void SetDriverName(string name)
	{
		// Ces deux déclarations sont techniquement les mêmes.
		driverName = name;
		this.driverName = name;
	}
	
	public void PopWheely()
	{
		for (int i = 0; i <= driverIntensity; i++)
			Console.WriteLine("Yeeeeee Haaaaeeewwww!");
	}
}
```

Assurez-vous maintenant que vos instructions de niveau supérieur exercent un objet `Motorcycle` comme suit :

```cs
Console.WriteLine("***** Fun with Motorcycle *****\n");

// Crée un objet Motorcycle.
Motorcycle c = new Motorcycle(5);
c.SetDriverName("Tiny");
c.PopWheely();
Console.WriteLine($"Rider name is {c.driverName}");
Console.ReadLine();
```

Avec cela, réfléchissez à la sortie du code précédent :

```
**** Fun with Motorcycle ****
In main constructor
In constructor taking an int
Yeeeeee Haaaaeeewwww!
Yeeeeee Haaaaeeewwww!
Yeeeeee Haaaaeeewwww!
Yeeeeee Haaaaeeewwww!
Yeeeeee Haaaaeeewwww!
Yeeeeee Haaaaeeewwww!
Rider name is Tiny
```

Comme vous pouvez le constater, la logique du constructeur est la suivante :

>[!tldr] Ré-explication (les mots du livre sont trop complexes pour l'opération)
>1. On crée un objet en appellant le constructeur qui prend un seul `int`
>2. **Avant de faire quoique-ce-soit**, ce constructeur appel le constructeur principale (celui qui prend un `int` et un `string`).
>3. Le code se trouvant dans le constructeur principale est exécuté.
>4. Une fois le code exécuté, on retourne dans le constructeur du départ et on exécute le code se trouvant dans celui ci.

==L'avantage du chaînage de constructeurs est que ce modèle de programmation fonctionne avec toutes les versions du langage C# et de la plateforme .NET==. Cependant, si vous visez .NET 4.0 et versions ultérieures, vous pouvez simplifier davantage vos tâches de programmation en utilisant des *arguments optionnels comme alternative au chaînage de constructeurs traditionnel*.

## Revisite des arguments optionnels

Au [[Chapitre 4#Définir des paramètres optionnels|Chapitre 4]], vous avez découvert les arguments optionnels et nommés. Rappelons que les arguments optionnels permettent de définir des valeurs par défaut pour les arguments entrants. **Si l'appelant est satisfait de ces valeurs par défaut, il n'est pas obligé de spécifier une valeur unique ; cependant, il peut le faire pour fournir à l'objet des données personnalisées.** Prenons l'exemple de la version suivante de `Motorcycle`, qui offre désormais plusieurs façons de construire des objets à l'aide d'une définition de constructeur unique :

```cs
class Motorcycle
{
	public int driverIntensity;
	public string driverName;
	
	// Un seul constructeur utilisant des arguments optionnels.
	public Motorcycle(int intensity = 0, string name = "")
	{
		if (intensity > 10)
			intensity = 10;
		
		driverIntensity = intensity;
		driverName = name;
	}
	...
}
```

Grâce à ce constructeur unique, vous pouvez désormais créer un objet `Motorcycle` avec zéro, un ou deux arguments. **Rappelons que la syntaxe des arguments nommés permet d'ignorer les paramètres par défaut acceptables** (voir [[Chapitre 3#Utilisation des types de données intrinsèques et l'opérateur `new` (MaJ C 9.0)|Chapitre 3]]).

```cs
static void MakeSomeBikes()
{
    // driverName = "", driverIntensity = 0
    Motorcycle m1 = new Motorcycle();
    Console.WriteLine(
        "Name= {0}, Intensity= {1}",
        m1.driverName,
        m1.driverIntensity
    );

    // driverName = "Tiny", driverIntensity = 0
    Motorcycle m2 = new Motorcycle(name: "Tiny");
    Console.WriteLine(
        "Name= {0}, Intensity= {1}",
        m2.driverName,
        m2.driverIntensity
    );

    // driverName = "", driverIntensity = 7
    Motorcycle m3 = new Motorcycle(7);
    Console.WriteLine(
        "Name= {0}, Intensity= {1}",
        m3.driverName,
        m3.driverIntensity
    );
}
```

Quoi qu'il en soit, à ce stade, vous pouvez définir une classe avec des données de champ (ou variables membres) et diverses opérations telles que des méthodes et des constructeurs. Formalisons maintenant le rôle du mot-clé `static`.

>[!warning] La grande différences avec l'utilisation du constructeur primaire
>Si on déclare des valeurs par défaut dans un constructeur primaire, alors **il ne faut pas** (saut si on doit implémenter une logique spécifique dans le constructeur) **déclarer manuellement les constructeur personnalisé**.
>
> Voici, si on retirait les appel à `Console.WriteLine()` dans les constructeurs, l'entièreté de la classe `Motorcyle`
> ```cs
> // Notre constructeur personnalisé (primaire)
>class MotorcycleCSharp12(int intensity = 0, string name = "")
>{
>    // Logique de validation à l'assignation
>    public int driverIntensity = (intensity > 10) ? 10 : intensity;
>    public string driverName = name;
>
>    Pas besoin de redéfinir Motorcycle() ou Motorcycle(int)
>    // SAUF si vous voulez absolument le WriteLine.
>    // Dans ce cas, il faut enlever les valeurs par défaut du haut.
>
>    public void SetDriverName(string name) => driverName = name;
>
>    // Une logique différente pour faire la même chose qu'une boucle for
>    // en utiliseant la syntaxe LINQ (Chapitre 9)
>    public void PopWheely() =>
>        Enumerable
>            .Range(0, driverIntensity + 1)
>            .ToList()
>            .ForEach(_ => Console.WriteLine("Yeeeeee Haaaaeeewwww!"));
>}
>```

# Comprendre le mot-clé `static`

Une classe C# ==peut définir un nombre illimité de *membres statiques*==, déclarés à l'aide du mot-clé `static`. **Dans ce cas, le membre concerné doit être invoqué directement au niveau de la classe, plutôt que depuis une variable de référence d'objet**. Pour illustrer cette distinction, ==prenons l'exemple de `System.Console`. Comme vous l'avez vu, la méthode `WriteLine()` n'est pas invoquée au niveau de l'objet, comme illustré ici==:

```cs
// Erreur de compilation! WriteLine() n'est pas une méthode au niveau de l'objet!
Console c = new Console();
c.WriteLine("I can't be printed...");
```

Au lieu de cela, préfixez simplement le nom de la classe au membre statique `WriteLine()`.

```cs
// Correcte! WriteLine() est une méthode statique.
Console.WriteLine("Much better! Thanks...);
```

En termes simples, **les membres statiques sont des éléments considérés** (par le concepteur de la classe) **comme tellement courants qu'il n'est pas nécessaire de créer une instance de la classe avant de les invoquer**. Bien que toute classe puisse définir des membres statiques, ==on les trouve assez fréquemment dans les *classes utilitaires*==. ==**Par définition, une classe utilitaire est une classe qui ne conserve aucun état au niveau objet et n'est pas créée avec le mot-clé `new`**==. En revanche, une classe utilitaire expose toutes ses fonctionnalités sous forme de membres au niveau classe (dits statiques).

Par exemple, si vous utilisez l'Explorateur d'objets de Visual Studio (via le menu Affichage ➤ Explorateur d'objets) pour afficher l'espace de noms `System`, vous constaterez que tous les membres des classes `Console`, `Math`, `Environnement` et `GC` (entre autres) exposent toutes leurs fonctionnalités via des membres statiques. Ce ne sont là que quelques exemples de classes utilitaires présentes dans les bibliothèques de classes de base .NET Core.

Encore une fois, **sachez que les membres statiques ne se trouvent pas uniquement dans les classes utilitaires** ; ==ils peuvent faire partie de n'importe quelle définition de classe==. N'oubliez pas que les membres statiques promeuvent un élément donné au niveau de la classe plutôt qu'au niveau de l'objet. Comme vous le verrez dans les sections suivantes, le mot-clé `static` peut s'appliquer aux cas suivants :

- Données d'une classe
- Méthodes d'une classe
- Propriétés d'une classe
- Un constructeur
- Définition complète de la classe
- En conjonction avec le mot-clé `using` en C#

Voyons chacune de nos options, en commençant par le concept de données statiques.

## Définition des données de champ statiques

==La plupart du temps, lors de la conception d'une classe, les données sont définies comme des données d'instance, ou, autrement dit, comme des données non statiques==. Lorsque vous définissez des données d'instance, vous savez que chaque fois que vous créez un nouvel objet, celui-ci conserve sa propre copie indépendante des données. En revanche, **lorsque vous définissez les données statiques d'une classe, la mémoire est partagée par tous les objets de cette catégorie**.

Pour comprendre la distinction, créez un projet d'application console nommé *StaticDataAndMembers*. Insérez ensuite un fichier nommé *SavingsAccount.cs* dans votre projet, puis créez une classe nommée `SavingsAccount`. Commencez par définir une variable d'instance (pour modéliser le solde actuel) et un constructeur personnalisé pour définir le solde initial.

```cs
namespace StaticDataAndMembers;

// Une classe simple de compte épargne (saving account)
class SavingsAccount
{
    // Donnée au niveau de l'instance.
    public double currBalance;

    public SavingsAccount(double currBalance)
    {
        this.currBalance = currBalance;
    }
}

```

Lors de la création d'objets `SavingsAccount`, la mémoire du champ `currBalance` est allouée à chaque objet. Ainsi, vous pouvez créer cinq objets `SavingsAccount` différents, chacun avec son propre solde. De plus, si vous modifiez le solde d'un compte, les autres objets ne sont pas affectés.

**Les données statiques**, en revanche, **sont allouées une fois et partagées entre tous les objets de la même catégorie**. Ajoutez une variable statique nommée `currInterestRate` à la classe `SavingsAccount`, dont la valeur par défaut est `0,04`.

```cs
// A simple savings account class.
class SavingsAccount
{
   // A static point of data.
   public static double currInterestRade = 0.04;
   // Instance-level data.
   public double currBalance;
   public SavingsAccount(double currBalance)
   {
      this.currBalance = currBalance;
   }
}
```

Créez trois instances de `SavingsAccount` dans les déclarations de niveau supérieur, comme suit :

```cs
using StaticDataAndMembers;

Console.Title = "Fun with Static Data";
Console.WriteLine("**** Fun with Static Data ****\n");

SavingsAccount s1 = new(50);
SavingsAccount s2 = new SavingsAccount(100);
SavingsAccount s3 = new SavingsAccount(currBalance: 10000.75);
Console.ReadLine();
```

L’allocation des données en mémoire ressemblerait à la figure ci-dessous: 

![[Figure 5.1.png|Les données statiques sont allouées une seule fois et partagées entre toutes les instances de la classe.]]

Ici, on suppose que toutes les instances de `SavingAccount` doivent avoir le même taux d'intérêt (`currInterestRate`). **Puisque les données statiques sont partagées par tous les objets d'une même catégorie, si vous les modifiez, tous les objets verront la nouvelle valeur lors de leur prochain accès aux données statiques, car ils consultent tous le même emplacement mémoire**. Pour comprendre comment modifier (ou obtenir) des données statiques, il est nécessaire d'examiner le rôle des méthodes statiques.

## Définition des méthodes statiques

Mettons à jour la classe `SavingsAccount` pour définir deux méthodes statiques. La première méthode statique (`GetInterestRate()`) renvoie le taux d'intérêt actuel, tandis que la seconde méthode statique (`SetInterestRate()`) permet de modifier le taux d'intérêt.

```cs
namespace StaticDataAndMembers;

// Une classe simple de compte épargne (saving account)
class SavingsAccount
{
	// Un point de donnée statique.
	public static double currInterestRate = 0.04;
	
	// Donnée au niveau de l'instance.
	public double currBalance;
	
	public SavingsAccount(double currBalance)
	{
		this.currBalance = currBalance;
	}
	
	// Membres statiques pour récupérer / assigner le taux d'interêt.
	public static void SetInterestRate(double newRate) =>
		currInterestRate = newRate;
	
	public static double GetInterestRate() => currInterestRate;
}
```

Maintenant, observez l’utilisation suivante :

```cs
using StaticDataAndMembers;

Console.Title = "Fun with Static Data";
Console.WriteLine("**** Fun with Static Data ****\n");

SavingsAccount s1 = new(50);
SavingsAccount s2 = new SavingsAccount(100);

// Affiche le taux d'intérêt.
Console.WriteLine($"Interest rate is: {SavingsAccount.GetInterestRate()}");

// Création d'un nouvel objet, ceci ne reset pas le taux d'intéret.
SavingsAccount s3 = new SavingsAccount(currBalance: 10000.75);

Console.WriteLine($"Interest rate is: {SavingsAccount.GetInterestRate()}");
Console.ReadLine();
```

La sortie du code précédent est affichée ici :

```
**** Fun with Static Data ****

Interest rate is: 0.04
Interest rate is: 0.04
```

Comme vous pouvez le constater, lorsque vous créez de nouvelles instances de la classe `SavingsAccount`, la valeur des données statiques n'est pas réinitialisée, car *CoreCLR (Core Common Language Runtime)* les alloue en mémoire une seule fois. Après cela, tous les objets de type `SavingsAccount` utilisent la même valeur pour le champ statique `currInterestRate`.

Lors de la conception d'une classe C#, l'un des défis est de déterminer les données à définir comme membres statiques et celles qui ne le doivent pas. **Bien qu'il n'existe pas de règle absolue, rappelez-vous qu'un champ de données statique est partagé par tous les objets de ce type. Par conséquent, si vous définissez un point de données que tous les objets doivent partager, le mot-clé `static` est la solution idéale**.

Imaginez ce qui se passerait si la variable de taux d'intérêt n'*était pas définie avec le mot-clé `static`*. Cela signifierait que chaque objet `SavingsAccount` aurait sa propre copie du champ `currInterestRate`. Supposons maintenant que vous ayez créé 100 objets `SavingsAccount` et que vous deviez modifier le taux d'intérêt. Cela nécessiterait d'appeler la méthode `SetInterestRate()` 100 fois ! Ce ne serait clairement pas une méthode efficace pour modéliser des « données partagées ». Là encore, les données statiques sont idéales lorsqu'une valeur est commune à tous les objets de cette catégorie.

>[!attention]  
>Le fait qu'un membre statique référence des membres non statiques dans son implémentation constitue une erreur de compilation. Par ailleurs, l'utilisation du mot-clé `this` sur un membre statique est une erreur, car cela implique un objet !

## Définition de constructeurs statiques

Un constructeur classique permet de définir la valeur des données d'instance d'un objet lors de sa création. **Cependant, que se passerait-il si vous tentiez d'affecter la valeur d'un point de données statique dans un constructeur classique ? Vous pourriez être surpris de constater que la valeur est réinitialisée à chaque création d'objet.**

À titre d'exemple, supposons que vous ayez mis à jour le constructeur de la classe `SavingsAccount` comme suit (notez également que vous n'affectez plus le champ `currInterestRate` en ligne) :

```cs
class SavingsAccount
{
	public static double currInterestRate;
	
	// Notez que notre constructeur assigne
	// la valeur statique currInterestRate.
	public double currBalance;
	
	public SavingsAccount(double currBalance)
	{
		currInterestRate = 0.04; // Ceci est une donnée statique.
		this.currBalance = currBalance;
	}
	...
}
```

Supposons maintenant que vous ayez rédigé le code suivant dans les instructions de niveau supérieur :

```cs
// Crée un compte
SavingsAccount s1 = new(50);

// Affiche le taux d'intérêt actuelle.
Console.WriteLine($"Interest Rate is: {SavingsAccount.GetInterestRate()}");

// Essaye de changer le taux d'intérêt via une propriété.
SavingsAccount.SetInterestRate(0.08);
// Crée un deuxième compte.
SavingsAccount s2 = new(100);
// devrait afficher 0.08 ... pas vrai??
Console.WriteLine($"Interest rate is : {SavingsAccount.GetInterestRate()}");
Console.ReadLine();
```

Si vous avez exécuté le code précédent, **vous constaterez que la variable `currInterestRate` est réinitialisée à chaque création d'un nouvel objet `SavingsAccount` et qu'elle est toujours définie à `0,04`**. ==Définir la valeur des données statiques dans un constructeur d'instance classique est clairement contre-productif. Chaque fois que vous créez un nouvel objet, les données de classe sont réinitialisées==. Une approche pour définir un champ statique consiste à utiliser la syntaxe d'initialisation des membres, comme vous l'avez fait initialement.

```cs
class SavingsAccount
{
	public double currBalance;
	
	// Point de donnée statique.
	public static double currInterestRate = 0.04;
	...
}
```

**Cette approche garantit que le champ statique n'est affecté qu'une seule fois, quel que soit le nombre d'objets créés. Cependant, que se passe-t-il si la valeur de vos données statiques doit être obtenue à l'exécution ?** Par exemple, dans une application bancaire classique, la valeur d'une variable de taux d'intérêt serait lue depuis une base de données ou un fichier externe. ***==L'exécution de telles tâches nécessite généralement une portée de méthode, telle qu'un constructeur, pour exécuter les instructions de code==***.

C'est pourquoi **==C# vous permet de définir un constructeur statique, ce qui vous permet de définir en toute sécurité les valeurs de vos données statiques==**. Considérez la mise à jour suivante de votre classe :

```cs
class SavingsAccount
{
	public static double currInterestRate;
	public double currBalance;
	
	public SavingsAccount(double currBalance)
	{
		this.currBalance = currBalance;
	}
	
	// Un constructeur statique!
	static SavingsAccount()
	{
		Console.WriteLine("In static constructor!");
		currInterestRate = 0.04;
	}
	...
```

**En termes simples, un constructeur statique est un constructeur spécial idéal pour initialiser les valeurs de données statiques lorsque la valeur est inconnue à la compilation** (par exemple, pour lire la valeur depuis un fichier externe, une base de données, générer un nombre aléatoire, etc.). Si vous ré-exécutiez le code précédent, vous obtiendriez le résultat attendu. **Notez que le message `In static constructor` ne s'affiche qu'une seule fois, car *CoreCLR* appelle tous les constructeurs statiques avant la première utilisation (et ne les rappelle plus jamais pour cette instance de l'application)**.

```
***** Fun with Static Data *****

In static constructor!
Interest Rate is: 0.04
Interest Rate is: 0.08
```

==Voici quelques points intéressants concernant les constructeurs statiques== :

- Une classe donnée ne peut définir qu'un seul constructeur statique. Autrement dit, *==le constructeur statique ne peut pas être surchargé==*.
- *==Un constructeur statique ne prend pas de modificateur d'accès et ne peut accepter aucun paramètre==*.
- **Un constructeur statique s'exécute une seule fois, quel que soit le nombre d'objets de ce type créés.**
- ==L'environnement d'exécution appelle le constructeur statique lorsqu'il crée une instance de la classe ou avant d'accéder au premier membre statique invoqué par l'appelant==.
-  Le constructeur statique s'exécute avant tout constructeur de niveau instance.

**Avec cette modification, lorsque vous créez de nouveaux objets `SavingsAccount`, la valeur des données statiques est préservée**, car le membre statique n'est défini qu'une seule fois dans le constructeur statique, **quel que soit le nombre d'objets créés**.

## Définition de classes statiques

Il est également possible d'appliquer le mot-clé `static` directement au niveau de la classe. **Lorsqu'une classe est définie comme statique, elle ne peut pas être créée avec le mot-clé `new` et ne peut contenir que des membres ou des champs de données marqués du mot-clé `static`. Dans le cas contraire, des erreurs de compilation se produisent**.

>[!tip]- Rappel 
>Une classe (ou structure) qui expose uniquement des fonctionnalités statiques est souvent appelée *classe utilitaire*. Lors de la conception d'une classe utilitaire, il est recommandé d'appliquer le mot-clé `static` à la définition de la classe.

À première vue, cette fonctionnalité peut paraître étrange, car une classe impossible à créer ne semble pas très utile. Cependant, **si vous créez une classe ne contenant que des membres statiques et/ou des données constantes, elle n'a aucunement besoin d'être allouée !** Par exemple, créez une classe nommée `TimeUtilClass` et définissez-la comme suit :

```cs
namespace StaticDataAndMembers;

// Les classes statiques ne peuvent contenir
// seulement des membres statique!
static class TimeUntilClass
{
    public static void PrintTime() =>
        Console.WriteLine(DateTime.Now.ToShortTimeString());

    public static void PrintDate() =>
        Console.WriteLine(DateTime.Today.ToShortDateString());
}
```

**Étant donné que cette classe a été définie avec le mot-clé `static`, vous ne pouvez pas créer d'instance de `TimeUtilClass` avec le mot-clé `new`.** En effet, toutes les fonctionnalités sont exposées au niveau de la classe. Pour tester cette classe, ajoutez les instructions suivantes aux instructions de niveau supérieur :

```cs
static void StaticClass()
{
   // Ceci compile parfaitement!
   TimeUntilClass.PrintDate();
   TimeUntilClass.PrintTime();
  
   // Erreur du compilateur! Ne peut pas créer 
   // d'instante d'une classe statique!
   TimeUntilClass u = new TimeUntilClass();
   
   Console.ReadLine();
}
```

## Importer des membres statiques via le mot-clé `using` en C# 

==C# 6 a ajouté la prise en charge de l'importation de membres statiques avec le mot-clé `using`==. À titre d'exemple, considérons le fichier C# qui définit actuellement la classe utilitaire. **Comme vous appelez la méthode `WriteLine()` de la classe `Console`, ainsi que les propriétés `Now` et `Today` de la classe `DateTime`, vous devez disposer d'une instruction using pour l'espace de noms `System`.** Les membres de ces classes étant tous statiques, vous pouvez modifier votre fichier de code avec les directives using statiques suivantes :

```cs
// Importe les membres statiques des classes Console et DateTime.
using static System.Console;
using static System.DateTime;
```

Grâce à ces « importations statiques », le reste de votre fichier de code peut utiliser directement les membres statiques des classes `Console` et `DateTime`, sans avoir à préfixer la classe de définition. Par exemple, vous pouvez mettre à jour votre classe utilitaire comme suit :

```cs
// Importe les membres statique des classes Console et DateTime.
using static System.Console;
using static System.DateTime;

namespace StaticDataAndMembers;

// Les classes statiques ne peuvent contenir
// seulement des membres statique!
static class TimeUntilClass
{
    public static void PrintTime() => WriteLine(Now.ToShortTimeString());

    public static void PrintDate() => WriteLine(Today.ToShortDateString());
}
```

Un exemple plus réaliste de simplification de code avec l'importation de membres statiques pourrait impliquer une classe C# utilisant largement la classe `System.Math` (ou une autre classe utilitaire). **Puisque cette classe ne contient que des membres statiques, il serait plus simple d'utiliser une instruction `using` statique pour ce type, puis d'appeler directement les membres de la classe `Math` dans votre fichier de code**.

Cependant, soyez conscient que l'utilisation excessive d'instructions `using` statiques peut entraîner une confusion. **Premièrement, que se passe-t-il si plusieurs classes définissent une méthode `WriteLine()` ?** ==Le compilateur est confus, tout comme les autres personnes qui lisent votre code==. **Deuxièmement, à moins que les développeurs ne soient familiers avec les bibliothèques de code .NET Core, ils pourraient ignorer que `WriteLine()` est un membre de la classe `Console`.** ==À moins de remarquer l'ensemble des importations statiques en haut d'un fichier de code C#, ils pourraient ne pas savoir où cette méthode est réellement définie==. Pour ces raisons, je limiterai l'utilisation des instructions d'utilisation statiques dans ce texte.

Quoi qu'il en soit, à ce stade du chapitre, vous devriez être à l'aise avec la définition de types de classes simples contenant des constructeurs, des champs et divers membres statiques (et non statiques). Maintenant que vous comprenez les bases de la construction de classes, vous pouvez étudier formellement les trois piliers de la programmation orientée objet.

# Définir les piliers de la POO

**Tous les langages orientés objet** (C#, Java, C++, Visual Basic, etc.) **doivent respecter ces trois principes fondamentaux, souvent appelés les piliers de la programmation orientée objet** (POO) :

- *Encapsulation* : Comment ce langage masque-t-il les détails d’implémentation interne d’un objet et préserve-t-il l’intégrité des données ?
- *Héritage* : Comment ce langage favorise-t-il la réutilisation du code ?
- *Polymorphisme* : Comment ce langage permet-il de traiter des objets apparentés de manière similaire ?

Avant d’approfondir chaque pilier, il est important de comprendre leurs rôles fondamentaux. Voici un aperçu de chaque pilier, qui sera examiné en détail dans la suite de ce chapitre et le suivant.

## Comprendre le rôle de l'encapsulation

Le premier pilier de la POO est l'*encapsulation*. ==Cette caractéristique se résume à la capacité du langage à masquer les détails d'implémentation inutiles à l'utilisateur de l'objet==. Par exemple, supposons que vous utilisiez une classe nommée `DatabaseReader`, qui possède deux méthodes principales : `Open()` et `Close()`.

```cs
// Assumez que cette class encapsule les détails 
// de l'ouvertue et la fermeture d'une base de donnée.
DatabaseReader dbReader = new DatabaseReader();
dbReader.Open(@"C:\AutoLot.mdf");

// Fais quelque chose avec les données du ficher et ferme le fichier.
dbReader.close();
```

La classe fictive `DatabaseReader` encapsule les détails internes de la localisation, du chargement, de la manipulation et de la fermeture d'un fichier de données. Les programmeurs apprécient l'encapsulation, car ce pilier de la POO simplifie les tâches de codage. **Il n'y a pas besoin de se soucier des nombreuses lignes de code qui s'exécutent en arrière-plan pour exécuter le travail de la classe `DatabaseReader`**. Il suffit de créer une instance et d'envoyer les messages appropriés (par exemple, « Ouvrez le fichier `AutoLot.mdf` situé sur mon disque C »).

**La ​​protection des données est étroitement liée à la notion d'encapsulation de la logique de programmation.** ==Idéalement, les données d'état d'un objet devraient être spécifiées à l'aide du mot-clé `private`, `internal` ou `protected`. De cette façon, le monde extérieur doit demander poliment pour modifier ou obtenir la valeur sous-jacente==. C'est une bonne chose, car les **points de données déclarés publiquement peuvent facilement être corrompus** (idéalement par accident plutôt que intentionnellement !). Vous examinerez cet aspect de l'encapsulation plus en détail dans un instant.

## Comprendre le rôle de l'héritage

Le pilier suivant de la POO, l'*héritage*, ==se résume à la capacité du langage à créer de nouvelles définitions de classe à partir de définitions de classe existantes==. En substance, l'**héritage permet d'étendre le comportement d'une classe de base (ou parent) en héritant des fonctionnalités principales dans la sous-classe dérivée (également appelée classe enfant).**

![[Figure 5.2.png|La relation "est un".]]

Le diagramme peut être interprété comme « Un `hexagon` est un `Shape` qui est un `Object` ». **Lorsque des classes sont liées par ce type d'héritage, des *relations « est-un »* sont établies entre les types. Cette relation est appelée *héritage****.

**On peut supposer ici que `Shape` définit un certain nombre de membres communs à tous les descendants** (par exemple, une valeur représentant la couleur de la forme et d'autres valeurs représentant la hauteur et la largeur). **Étant donné que la classe `Hexagon` étend `Shape`, elle hérite des fonctionnalités principales définies par `Shape` et `Object`, et définit également des détails supplémentaires relatifs aux `Hexagon` (quels qu'ils soient).**

>[!important] Sur les plateformes .NET et .NET Core, `System.Object` **est toujours le parent le plus élevé de toute hiérarchie de classes, ce qui définit certaines fonctionnalités générales pour tous les types** (décrites en détail au [[Chapitre 6|Chapitre 6]]).

Il existe une autre forme de réutilisation de code en POO : le modèle de confinement/délégation, également appelé relation « possède » ou *agrégation*. **Cette forme de réutilisation ne sert pas à établir de relations parent-enfant. Elle permet à une classe de définir une variable membre d'une autre classe et d'exposer indirectement ses fonctionnalités (si nécessaire) à l'utilisateur de l'objet**.

Par exemple, supposons que vous modélisiez une automobile. Vous pourriez vouloir exprimer l'idée qu'une voiture "possède" une radio. Il serait illogique de tenter de dériver la classe `Car` d'une `Radio` ou inversement (une `car` est-elle une `Radio` ? Je ne pense pas !). On a plutôt deux classes indépendantes qui travaillent ensemble, la classe `Car` créant et exposant les fonctionnalités de `Radio`.

```cs
namespace OopExamples;

class Radio
{
    public void Power(bool turnOn)
    {
        Console.WriteLine($"Radio on: {turnOn}");
    }
}
```

```cs
namespace OopExamples;

class Car
{
    // Car possède une instance de Radio.
    private Radio myRadio = new Radio();

    public void TurnOnRadio(bool onOff)
    {
        // Déléguer l'appel à l'objet interne.
        myRadio.Power(onOff);
    }
}
```

Notez que l’utilisateur de l’objet n’a aucune idée que la classe `Car` utilise un objet `Radio` interne.

```cs
using OopExamples;

Console.Title = "OOP Examples";
Console.WriteLine("**** OOP Examples ****\n");

// L'appel est transféré à radio en interne.
Car viper = new();
viper.TurnOnRadio(false);
```

## Comprendre le rôle du polymorphisme

Le dernier pilier de la POO est le *polymorphisme*. **Cette caractéristique définit la capacité d'un langage à traiter des objets apparentés de manière similaire**. Plus précisément, ==ce principe d'un langage orienté objet permet à une classe de base de définir un ensemble de membres (formellement appelé *interface polymorphe*) accessibles à tous les descendants==. **L'interface polymorphe d'une classe est construite à l'aide d'un nombre quelconque de membres *virtuels* ou *abstraits*** (voir le [[Chapitre 6|Chapitre 6]] pour plus de détails).

**En résumé, *un membre virtuel* est un membre d'une classe de base qui définit une implémentation par défaut qui peut être modifiée (ou, plus formellement, *surchargée*) par une classe dérivée**. ==À l'inverse, une *méthode abstraite* est un membre d'une classe de base qui ne fournit pas d'implémentation par défaut, mais fournit une signature==. Lorsqu'une classe dérive d'une classe de base définissant une méthode abstraite, elle *doit* être surchargée par un type dérivé. Dans les deux cas, lorsque des types dérivés remplacent les membres définis par une classe de base, ils redéfinissent essentiellement leur façon de répondre à la même requête.

Pour prévisualiser le polymorphisme, détaillons la hiérarchie des formes illustrée précédemment. **Supposons que la classe `Shape` ait défini une ==méthode virtuelle== nommée `Draw()` qui ne prend aucun paramètre. Étant donné que chaque forme doit s'afficher de manière unique, les sous-classes telles que `Hexagon` et `Circle` sont libres de remplacer cette méthode à leur guise.**

![[Figure 5.3.png|Polymorphisme classique]]

Après avoir conçu une interface polymorphe, vous pouvez commencer à formuler diverses hypothèses dans votre code. **Par exemple, sachant que `Hexagon` et `Circle` dérivent d'un parent commun (`Shape`), un tableau de type `Shape` pourrait contenir tout ce qui dérive de cette classe de base.** ==De plus, puisque `Shape` définit une interface polymorphe pour tous les types dérivés (la méthode `Draw()` dans cet exemple), vous pouvez supposer que chaque membre du tableau possède cette fonctionnalité==.

Considérez le code suivant, qui indique à un tableau de types dérivés de `Shape` de s'afficher à l'aide de la méthode `Draw()` :

```cs
Shape[] myShapes = new Shape[3];
myShapes[0] = new Hexagon();
myShapes[1] = new Circle();
myShapes[2] = new Hexagon();

foreach (Shape s in myShapes)
{
	// Utilisation de l'interface polymorphique!
	s.Draw();
}
Console.ReadLine();
```

Ceci conclut notre rapide aperçu des piliers de la POO. Maintenant que vous avez la théorie en tête, le reste de ce chapitre explore plus en détail la gestion de l'encapsulation en C#, en commençant par les modificateurs d'accès. Le [[Chapitre 6|Chapitre 6]] abordera les détails de l'héritage et du polymorphisme.

# Comprendre les modificateurs d'accès (MaJ C# 7.2)

**Lorsque vous travaillez avec l'encapsulation, vous devez toujours considérer quels aspects d'un type sont visibles par les différentes parties de votre application. Plus précisément, les types** (classes, interfaces, structures, énumérations et délégués) **ainsi que leurs membres** (propriétés, méthodes, constructeurs et champs) **sont définis à l'aide d'un mot-clé spécifique pour contrôler la visibilité de l'élément par les autres parties de votre application**. Bien que C# définisse de nombreux mots-clés, pour contrôler l'accès, ils diffèrent selon leur application (type ou membre). Le [[#Tableau 5-1 Les modificateurs d'accès C|Tableau 5-1]] décrit le rôle de chaque modificateur d'accès et son application.

##### Tableau 5-1: Les modificateurs d'accès C# 

| Modificateur d'accès | Applicable à                          | Description                                                                                                                                                                                                                                                    |
| -------------------- | ------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `public`             | Types ou membres du type              | Les éléments publics ne sont soumis à aucune restriction d'accès. Un membre public est accessible depuis un objet, ainsi que toute classe dérivée. Un type public est accessible depuis d'autres assemblies externes.                                          |
| `private`            | Membres de type<br>ou types imbriqués | Les éléments privés ne sont accessibles que par la classe (ou la structure) qui les définit.                                                                                                                                                                   |
| `protected`          | Membres de type<br>ou types imbriqués | Les éléments protégés peuvent être utilisés par la classe qui les définit et par toute classe enfant. Ils ne sont pas accessibles en dehors de la chaîne d'héritage.                                                                                           |
| `internal`           | Types ou membres du type              | Les éléments internes ne sont accessibles que dans l'assemblage actuel. D'autres assemblages peuvent être explicitement autorisés à voir les éléments internes.                                                                                                |
| `protected internal` | Membres de type<br>ou types imbriqués | Lorsque les mots-clés `protected` et `internal` sont combinés sur un élément, celui-ci est accessible au sein de l'assemblage de définition, de la classe de définition et des classes dérivées, à l'intérieur ou à l'extérieur de l'assemblage de définition. |
| `private protected`  | Membres de type<br>ou types imbriqués | (nouveauté C# 7.2) Lorsque les mots-clés `private` et `protected` sont combinés sur un élément, celui-ci est accessible au sein de la classe de définition et par les classes dérivées du même assemblage.                                                     |
|                      |                                       |                                                                                                                                                                                                                                                                |

> [!tip] Explication simplifiée de `internal`  
> Le modificateur d'accès `internal` limite l'accès aux membres **uniquement** aux appelants situés dans le même projet (le même **assemblage**).
> 
> **Un assemblage = un projet** (le fichier `.dll` ou `.exe` fabriqué lors de la compilation. Voir [[Chapitre 1#Aperçu des assembly .NET|Chapitre 1]]).
>
> Si un appelant se trouve dans un autre projet, il ne peut pas voir les membres `internal`, même s'il référence explicitement le fichier `.csproj`. Il ne verra que ce qui est `public`.
> 
>>[!example] **L'exception**: 
>>Un projet peut accorder une "permission spéciale" à un autre assemblage spécifique (*via l'attribut `InternalsVisibleTo`*) pour lui permettre de voir ses membres `internal`. C'est très utilisé pour permettre aux projets de **tests unitaires** d'accéder au code sans le rendre public pour tout le monde. Le [[Chapitre 16#Exposition des types internes à d'autres assemblies|Chapitre 16]] montrent cette fonctionnalité.

Dans ce chapitre, nous nous intéressons uniquement aux mots-clés `public` et `private`. Les chapitres suivants examineront le rôle des modificateurs `internal protected` (utiles pour la création de bibliothèques de code et de tests unitaires) ainsi que du modificateur `protected` (utile pour la création de hiérarchies de classes).

## Utilisation des modificateurs d'accès par défaut

==Par défaut, les membres de type sont *implicitement `private`*, tandis que les types sont *implicitement `internal`*==. **Ainsi, la définition de classe suivante est automatiquement définie comme `internal`, tandis que le constructeur par défaut du type est automatiquement défini comme `privé`** (cependant, comme vous pouvez vous en douter, un constructeur de classe privé est rarement souhaité) :

```cs
namespace OopExamples;

// Une classe interne avec un constructeur privé.
class Radio
{
    Radio(){}
}
```

==Si vous souhaitez être explicite, vous pouvez ajouter ces mots-clés vous-même sans aucun problème== (au-delà de quelques frappes supplémentaires).

```cs
namespace OopExamples;

// Une classe interne avec un constructeur privé.
internal class Radio
{
    private Radio() { }
}
```

**Pour permettre à d'autres parties d'un programme d'invoquer des membres d'un objet, vous devez les définir avec le mot-clé `public` (ou éventuellement avec le mot-clé `protected`, que vous découvrirez dans le chapitre suivant)**. ==De plus, si vous souhaitez exposer la classe `Radio` à des assemblies externes== (là encore, utile pour la création de solutions plus volumineuses ou de bibliothèques de code), ==vous devrez ajouter le modificateur `public`==.

```cs
// Une classe publique avec un constructeur par défaut publique.
public class Radio
{
	public Radio(){}
}
```

## Utilisation des modificateurs d'accès et des types imbriqués

Comme indiqué dans le [[#Tableau 5-1 Les modificateurs d'accès C|Tableau 5-1]], les modificateurs d'accès `private`, `protected`, `protected internal` et `private protected` peuvent être appliqués à un type imbriqué. Le [[Chapitre 6|Chapitre 6]] examinera l'imbrication en détail. **Ce qu'il faut savoir, à ce stade, c'est qu'un type imbriqué est un type déclaré directement dans la portée d'une classe ou d'une structure**. À titre d'exemple, voici une énumération privée (nommée `CarColor`) imbriquée dans une classe publique (nommée `SportsCar`) :

```cs
namespace oopexamples;

public class SportsCar
{
    // OK! les membres imbriqués peuvent être marqué privés.
    private enum CarColor
    {
        Red,
        Green,
        Blue,
    }
}
```

Ici, **il est permis d'appliquer le modificateur d'accès `private` au type imbriqué. Cependant, les types non imbriqués** (comme `SportsCar`) **ne peuvent être définis qu'avec le modificateur `public` ou `internal`**. Par conséquent, la définition de classe suivante est illégale :

```cs
// Erreur! Les types non-imbriqué ne peuvent pas être marquée comme privés!
private class SpportyCar
{}
```

# Comprendre le premier pilier : les services d’encapsulation de C# 

**Le concept d’encapsulation repose sur l’idée que les données d’un objet ne doivent pas être directement accessibles depuis une instance d’objet.** Plutôt, ==les données de classe sont définies comme `private`==. Si l’utilisateur souhaite modifier l’état d’un objet, il le fait indirectement via des membres publics. Pour illustrer la nécessité des services d’encapsulation, supposons que vous ayez créé la définition de classe suivante :

```cs
// Une classe avec un seul champ publique
class Book
{
    public int numberOfPages;
}
```

**Le problème avec les données publiques est qu'elles ne peuvent pas comprendre si la valeur à laquelle elles sont assignées est valide au regard des règles métier du système**. Comme vous le savez, la plage supérieure d'un `int` C# est assez large ($2\;147\;483\;647$). Par conséquent, le compilateur autorise l'assignation suivante :

```cs
// Euuh. c'est une sacrée mini-nouvelle!
Book miniNovel = new Book();
miniNovel.numberOfPages = 30_000_000;
```

Bien que vous n'ayez pas dépassé les limites d'un type de données `int`, il est clair qu'un mini-roman avec $30\;000\;000$ pages est un peu excessif. ==Comme vous pouvez le constater, les champs publics ne permettent pas de capturer les limites logiques== supérieures (ou inférieures). ==Si votre système actuel possède une règle métier stipulant qu'un livre doit comporter entre 1 et 1 000 pages, vous ne pouvez pas l'appliquer par programmation==. De ce fait, les champs `public` n'ont généralement pas leur place dans une définition de classe de "professionnel".

>[!tip] Plus précisément, les membres d'une classe représentant l'état d'un objet ne doivent pas être marqués comme `public`. Comme vous le verrez plus loin dans ce chapitre, les constantes publiques et les champs publics en lecture seule sont très utiles.

**L'encapsulation permet de préserver l'intégrité des données d'état d'un objet**. ==Plutôt que de définir des champs publics== (ce qui peut facilement favoriser la corruption des données), ==il est conseillé de prendre l'habitude de définir des *données privées*, manipulées indirectement par l'une des deux techniques suivantes== :

- Vous pouvez définir une paire de méthodes publiques d'accès (`get`) et de mutation (`set`).
- Vous pouvez définir une *propriété publique*.

Quelle que soit la technique choisie, ==l'objectif est qu'une classe bien encapsulé protège ses données et cache son fonctionnement aux regards indiscrets. C'est ce que l'on appelle souvent la *programmation boîte noire*==. L'avantage de cette approche réside dans le fait qu'**un objet est libre de modifier l'implémentation d'une méthode donnée en arrière-plan**. Il le fait sans perturber le code existant qui l'utilise, **à condition que les paramètres et les valeurs de retour de la méthode restent constants**.

## Encapsulation utilisant des accesseurs et des mutateurs traditionnels

Dans les pages suivantes de ce chapitre, vous construirez une classe assez complète modélisant un employé standard. Pour commencer, créez un projet d'application console nommé *EmployeeApp* et un fichier de classe nommé *Employee.cs*. Mettez à jour la classe `Employee` avec l'espace de noms, les champs, les méthodes et les constructeurs suivants :

```cs
namespace EmployeeApp;

public class Employee
{
    // Données de champs
    private string _empNmae;
    private int _empId;
    private float _currentPay;

    // Constructeurs
	 public Employee() { }
    public Employee(string name, int id, float pay)
    {
       _empNmae = name;
       _empId = id;
       _currentPay = pay;
    }

    // Méthodes.
    public void GiveBonus(float amount) => _currentPay += amount;
    public void DisplayStats()
    {
        Console.WriteLine($"Name: {_empNmae}");
        Console.WriteLine($"ID: {_empId}");
        Console.WriteLine($"Pay: {_currentPay}");
    }
}
```

**Notez que les champs de la classe `Employee` sont actuellement définis avec le mot-clé `private`. De ce fait, les champs `_empName`, `_empId` et `_currPay` ne sont pas directement accessibles depuis une variable objet**. Par conséquent, la logique suivante dans votre code entraînerait des erreurs de compilation :

```cs
using EmployeeApp;

Employee emp = new Employee();

// Erreur! Ne peut pas dirrectement accédér 
// aux membres privées d'un objet.
emp._empName = "Marv";
```

Si vous souhaitez que le monde extérieur interagisse avec le nom complet d'un employé, **une approche traditionnelle consiste à définir un accesseur (méthode `get`) et un mutateur (méthode `set`)**. ***==Le rôle d'une méthode `get` est de renvoyer à l'appelant la valeur actuelle des données d'état sous-jacentes. Une méthode `set` permet à l'appelant de modifier la valeur actuelle des données d'état sous-jacentes, à condition que les règles métier définies soient respectées==***.

À titre d'illustration, encapsulons le champ `empName`. Pour ce faire, ajoutez les méthodes publiques suivantes à la classe `Employee`. Notez que la méthode `SetName()` effectue un test sur les données entrantes pour s'assurer que la chaîne contient `15` caractères maximum. Dans le cas contraire, une erreur s'affiche sur la console et l'exécution est terminée sans modification du champ `empName`.

>[!tip] Bonne pratique
>S'il s'agissait d'une classe de production, vous vérifieriez probablement également la longueur en caractères du nom d'un employé dans votre logique de constructeur. Ignorez ce détail pour le moment, car vous corrigerez ce code dans quelques instants lorsque vous examinerez la syntaxe des propriétés.

```cs
class employee
{
	// Donnée de champs
	 private string _empName;
	 ...
	
    // Accesseur (méthode Get)
    public string GetName() => _empName;

    // Mutateur (méthode Set)
    public void SetName(string name)
    {
        // Effectue une vérification de la
        // valeur entrante avant l'assignement.
        if (name.Length > 15)
            Console.WriteLine("Error! Name length exceeds 15 characters");
        else
            _empName = name;
    }
}
```

Cette technique nécessite deux méthodes portant un nom unique pour fonctionner sur un même point de données. Pour tester vos nouvelles méthodes, mettez à jour votre méthode de code comme suit :

```cs
using EmployeeApp;

Console.Title = "Fun with Encapsulation";
Console.WriteLine("***** Fun with Encapsulation *****\n");

Employee emp = new("Marvin", 456, 30_000);
emp.GiveBonus(1000);
emp.DisplayStats();

// Utilisation des méthode get/set pour intéragin avec le nom de l'objet.
emp.SetName("Marv");
Console.WriteLine($"Employee is named: {emp.GetName()}");
Console.ReadLine();
```

En raison du code de votre méthode `SetName()`, si vous tentiez de spécifier plus de 15 caractères (voir ci-dessous), le message d'erreur codé en dur s'afficherait sur la console :

```cs
// Plus long que 15 caractères! Une erreur sera affiché à la console.
Employee emp2 = new Employee();
emp.SetName("Xena the warrior princess");

Console.ReadLine();
```

Jusqu'ici, tout va bien. Vous avez encapsulé le champ privé `empName` à l'aide de deux méthodes publiques nommées `GetName()` et `SetName()`. **Pour encapsuler davantage les données de la classe `Employee`, vous devrez ajouter diverses méthodes supplémentaires (telles que `GetID()`, `SetID()`, `GetCurrentPay()` et `SetCurrentPay()`). Chacune des méthodes de mutation pourrait également comporter plusieurs lignes de code pour vérifier des règles métier supplémentaires**. Bien que cela soit tout à fait possible, le langage C# propose une notation alternative utile pour encapsuler les données de classe.

## Encapsulation à l'aide de propriétés

Bien qu'il soit possible d'encapsuler les données d'un champ à l'aide des méthodes `get` et `set` traditionnelles, **les langages .NET Core préfèrent appliquer les données d'état d'encapsulation à l'aide de *propriétés***. ==Tout d'abord, il faut comprendre que les propriétés ne sont qu'un conteneur pour les méthodes d'accès et de mutation « réelles », nommées respectivement `get` et `set`==. **Par conséquent, en tant que concepteur de classe, vous pouvez toujours exécuter toute logique interne nécessaire avant d'attribuer une valeur** (par exemple, mettre la valeur en majuscules, supprimer les caractères illégaux, vérifier les limites d'une valeur numérique, etc.).

Voici la classe `Employee` mise à jour, qui impose désormais l'encapsulation de chaque champ à l'aide de la syntaxe de propriété plutôt que des méthodes `get` et `set` traditionnelles :

```cs
namespace EmployeeApp;

public class Employee
{
    // Données de champs
    private string _empName;
    private int _empId;
    private float _currPay;

    // Propriétés
    public string Name
    {
        get { return _empName; }
        set
        {
            if (value.Length > 15)
                Console.WriteLine("Error! Name length exceeds 15 characters");
            else
                _empName = value;
        }
    }

    // On pourrait ajouter des régles métiers supplémentaires
    // à ces ensembles de propriétés; cependant, ce n'est pas
    // nécessaires pour cette exemple.
    public int Id
    {
        get => _empId;
        set => _empId = value;
    }
    public float Pay
    {
        get => _currPay;
        set => _currPay = value;
    }
   ...
}
```

**Une propriété C# est composée en définissant une portée `get` (accesseur) et une portée `set` (mutateur) directement dans la propriété elle-même**. Notez que la propriété spécifie le type de données qu'elle encapsule par ce qui semble être une valeur de retour. **Notez également que, contrairement à une méthode, les propriétés n'utilisent pas de parenthèses (même vides) lors de leur définition.** Tenez compte du commentaire suivant concernant votre propriété `Id` actuelle :

```cs
// "int" représente le type de donnée que cette propriété encapsule.
public int Id   // Notez le manque de parenthèses.
{
	get { return _empId; }
	set { _empId = value; }
}
```

**Dans la portée définie d'une propriété, vous utilisez un jeton nommé `valeur`, qui représente la valeur entrante utilisée pour assigner la propriété par l'appelant.** ==Ce jeton n'est pas un véritable mot-clé C#, mais un mot-clé contextuel. Lorsque la valeur du jeton est dans la portée définie de la propriété, elle représente toujours la valeur assignée par l'appelant et son type de données sous-jacent est toujours identique à celui de la propriété elle-même==. Ainsi, remarquez comment la propriété `Name` peut toujours tester la plage de la chaîne :

```cs
   public string Name
   {
      get => _empNmae;
      set
      {
          // Ici, value est réellement un string.
         if (value.Length > 15)
         {
            Console.WriteLine("Error! Name length exceeds 15 characters!");
         }
         else
         {
            _empNmae = value;
         }
      }
   }
```

==Une fois ces propriétés définies, l'appelant a l'impression d'obtenir et de définir un *point de données public*. Cependant, le bloc d'obtention et de définition correct est appelé en arrière-plan afin de préserver l'encapsulation==.

```cs
using EmployeeApp;

Console.WriteLine("***** Fun with Encapsulation *****\n");
Employee emp = new Employee("Marvin", 456, 30000);
emp.GiveBonus(1000);
emp.DisplayStats();

// Réinitialise en ensuite accède la propriété Name.
emp.Name = "Marv";
Console.WriteLine("Employee is named: {0}", emp.Name);
Console.ReadLine();
```

**Les propriétés (==contrairement aux méthodes d'accès et de mutation==) facilitent également la manipulation de vos types, car elles peuvent répondre aux opérateurs intrinsèques de C#.** À titre d'exemple, supposons que le type de classe `Employee` possède une variable membre privée interne représentant l'âge de l'employé. Voici la mise à jour correspondante (notez l'utilisation du chaînage de constructeurs) :

```cs
class Employee
{
	...
	// Nouveaux champs et propriété
	private int _empAge;
	public int Age
	{
		get { return _empAge; }
		set { _empAge = value;}
	}

	// constructeurs mis à jour.
	public Employee() {}
	public Employee(string name, int id, float pay)
		:this(name, 0, id, pay){}
	public Employee(string name, int age, int id, float pay)
	{
		_empName = name;
		_empId = id;
		_empAge = age;
		_currPay = pay;
	}
	// Méthode DisplayStats() mis à jour pour prendre en compte l'âge.
	public void DisplayStats()
	{
		Console.WriteLine("Name: {0}", _empName);
		Console.WriteLine("ID: {0}", _empId);
		Console.WriteLine("Age: {0}", _empAge);
		Console.WriteLine("Pay: {0}", _currPay);
	}
}
```

>[!info] Rappel
>L'exemple montrée ici est différent du code venant du projet (constructeur primaire pas utilisé dans l'exemple).

Supposons maintenant que vous ayez créé un objet `Employee` nommé Joe. Le jour de son anniversaire, vous souhaitez incrémenter son âge d'une unité. En utilisant les méthodes d'accès et de mutation traditionnelles, vous devrez écrire du code comme suit :

```cs
Employee joy = new Employee();
joe.SetAge(joe.GetAge() + 1);
```

Cependant, si vous encapsulez `empAge` à l'aide d'une propriété nommée `Age`, vous pouvez simplement écrire ceci :

```cs
Employee joe = new();
joe.Age++;
```

### Propriétés comme membres à corps d'expression (Nouveauté C# 7.0)

Comme mentionné précédemment, **les accesseurs `get` et `set` de propriétés peuvent également être écrits comme membres à corps d'expression**. Les règles et la syntaxe sont identiques : ==les méthodes sur une seule ligne peuvent être écrites avec la nouvelle syntaxe==. Ainsi, la propriété `Age` pourrait s'écrire ainsi :

```cs
public int Age
{
	get => _empAge;
	set => _empAge = value;
}
```

**Les deux syntaxes compilent vers le même IL ; le choix de la syntaxe à utiliser vous appartient donc entièrement**. ==Dans ce texte, vous verrez un mélange des deux styles afin de les rendre plus visibles, et non par souci d'adhésion à un style de code spécifique==.

## Utilisation des propriétés dans une définition de classe

Les propriétés, et plus particulièrement la partie `Set` d'une propriété, sont des emplacements courants pour empaqueter les règles métier de votre classe. Actuellement, la classe `Employee` possède une propriété `Name` qui garantit que le nom ne dépasse pas 15 caractères. Les autres propriétés (`Id`, `Pay` et `Age`) peuvent également être mises à jour avec toute logique pertinente.

**Bien que cela soit pertinent, il convient également de considérer ce qu'un constructeur de classe fait généralement en interne. Il prend les paramètres entrants, vérifie la validité des données, puis effectue les affectations aux champs privés internes**. Actuellement, votre constructeur principal ne teste pas la validité des données de chaîne entrantes ; vous pouvez donc mettre à jour ce membre comme suit :

```cs
public Employee(string name, int age, int id, float pay)
{
	// Hum, ça ressemble à un problème...
	if (name.Length > 15)
	{
		 Console.WriteLine("Error! Name length exceeds 15 characters!");
	}
	else
	{
		 _empName = name;
	}
	_empId = id;
	_empAge = age;
	_currPay = pay;
}
```

Je suis sûr que vous comprenez le problème de cette approche. **La propriété `Name` et votre constructeur principal effectuent la même vérification d'erreurs. Si vous vérifiiez également les autres points de données, vous auriez beaucoup de code dupliqué**. ==Pour rationaliser votre code et centraliser toutes vos vérifications d'erreurs, il est judicieux d'utiliser systématiquement les propriétés de votre classe lorsque vous devez obtenir ou définir des valeurs==. Prenons l'exemple du constructeur mis à jour suivant :

```cs
public Employee(string name,int age, int id, float pay)
{
	 Name = name;
	 Age = age;
	 Id = id;
	 Pay = pay;
}
```

Outre la mise à jour des constructeurs pour utiliser des propriétés lors de l'attribution de valeurs, **il est recommandé d'utiliser des propriétés tout au long de l'implémentation d'une classe afin de garantir l'application constante de vos règles métier. Dans de nombreux cas, la seule référence directe aux données privées sous-jacentes se fait au sein même de la propriété**. Dans cette optique, voici votre classe `Employee` mise à jour :

- En utilisant la syntaxe plus classique (avant C# 12)

```cs
namepsace EmployeeApp
public class Employee
{
    // Champs de données.
    private string _empName;
    private int _empId;
    private float _currPay;
    private int _empAge;

    // Propriétés.
    public string Name
    {
        get => _empName;
        set
        {
            // Ici, la valeur est réellement un string.
            if (value.Length > 15)
            {
                Console.WriteLine("Error! Name length exceeds 15 characters!");
            }
            else
            {
                _empName = value;
            }
        }
    }
    public int Id
    {
        get => _empId;
        set => _empId = value;
    }
    public float Pay
    {
        get => _currPay;
        set => _currPay = value;
    }
    public int Age
    {
        get => _empAge;
        set => _empAge = value;
    }

    // Constructeurs
    public Employee() { }

    public Employee(string name, int id, float pay)
        : this(name, 0, id, pay) { }

    public Employee(string name, int age, int id, float pay)
    {
        Name = name;
        Age = age;
        Id = id;
        Pay = pay;
    }

    // Méthodes
    public void GiveBonus(float amount) => Pay += amount;

    public void DisplayStats()
    {
        Console.WriteLine($"Name: {Name}");
        Console.WriteLine($"Age: {Age}");
        Console.WriteLine($"ID: {Id}");
        Console.WriteLine($"Pay: {Pay}");
    }

    public string GetName() => _empName;
    public void SetName(string name) => _empName = name;
}

```

- En utilisant la syntaxe moderne (C# 14+)

```cs
namespace EmployeeApp;

// Constructeur primaire du C# 12
public class Employee(string name, int id, int age, float pay)
{
    // Spécificité de C# 13/14 :
    // On utilise 'field' pour la validation sans déclarer de variable privée.
    // L'assignation "= name" à la fin appelle le SETTER
    // (comportement spécifique au mot-clé field).
    public string Name
    {
        get;
        set
        {
            if (value.Length > 15)
                Console.WriteLine("Error! Name length exceeds 15 characters");
            else
                field = value;
        }
    } = name;
    public int Id { get; set; } = id;
    public int Age { get; set; } = age;
    public float Pay { get; set; } = pay;

    // Constructeurs secondaires
    public Employee()
        : this("Unknown", 0, 0, 0.0f) { }

    public Employee(string name, int id, float pay)
        : this(name, id, 0, pay) { }

    public void DisplayStats() =>
        Console.WriteLine($"Name: {Name}, ID: {Id}, Pay: {Pay:C}");
}
```

>[!info] Ce code ne déclare pas de champs privés, ce qui sera expliqué plus loins dans ce chapitre.

## Propriétés en lecture seule

Lors de l'encapsulation de données, vous souhaiterez peut-être configurer une *propriété en lecture seule*. **Pour ce faire, omettez simplement le bloc `set`.** Par exemple, supposons que vous ayez une nouvelle propriété nommée `SocialSecurityNumber`, qui encapsule une variable de chaîne privée nommée `empSSN`. Pour la rendre en lecture seule, vous pouvez écrire ceci :

```cs
public string SocialSecurityNumber 
{ 
	get { return _empSSN; }
}
```

**Les propriétés qui ne disposent que d'un getter peuvent également être simplifiées à l'aide des membres du corps de l'expression. La ligne suivante est équivalente au bloc de code précédent** :

```cs
public string SocialSecurityNumber => _empSSN;
```

Supposons maintenant que le constructeur de votre classe contienne un nouveau paramètre permettant à l'appelant de définir le numéro de sécurité sociale de l'objet. La propriété `SocialSecurityNumber` étant en lecture seule, vous ne pouvez pas définir la valeur comme suit :

```cs
public Employee(string name,int age, int id, float pay, string ssn)
{
	Name = name;
	Age = age;
	Id = id;
	Pay = pay;

	// OUPS! Ceci n'est plus possible si la propriété est en lecture seule.
	SocialSecurityNumber = ssn;
}
```

**À moins que vous ne souhaitiez redéfinir la propriété en lecture-écriture (ce que vous ferez bientôt), votre seule option avec les propriétés en lecture seule serait d'utiliser la variable membre `empSSN` sous-jacente dans votre logique de constructeur, comme suit** :

```cs
public Employee(string name,int age, int id, float pay, string ssn)
{
	...
	// Check incoming ssn parameter as required and then set the value.
	_empSSN = ssn;
}
```

## Propriétés en écriture seule

Si vous souhaitez configurer votre *propriété en écriture seule*, omettez le bloc `get`, comme ceci :

```cs
public int Id { set => _empId = value; }
```

## Mélange de méthodes `Get`/`Set` privées et publiques sur les propriétés

**Lors de la définition des propriétés, le niveau d'accès des méthodes `Get` et `Set` peut être différent**. Pour revenir au numéro de sécurité sociale, si l'objectif est d'empêcher sa modification depuis l'extérieur de la classe, déclarez la méthode `Get` comme `public` et la méthode `Set` comme `private`, comme suit :

```cs
public string SocialSecurityNumber 
{ 
    get => _empSSN; 
    private set => _empSSN = value; 
} 
```

**Notez que cela modifie la propriété de lecture seule à lecture-écriture. La différence réside dans le fait que l'écriture est masquée pour tout élément extérieur à la classe de définition**.

## Revisite du mot-clé `static` : Définition des propriétés statiques

Plus tôt dans ce chapitre, vous avez examiné le rôle du mot-clé `static`. Maintenant que vous comprenez l'utilisation de la syntaxe des propriétés C#, vous pouvez formaliser les propriétés statiques. ==Dans le projet `StaticDataAndMembers` créé plus tôt dans ce chapitre, votre classe `SavingsAccount` disposait de deux méthodes statiques publiques pour obtenir et définir le taux d'intérêt==. Cependant, **il serait plus courant d'encapsuler ce point de données dans une propriété statique**. Voici un exemple (notez l'utilisation du mot-clé `static`) :

```cs
// Une classe simple de compte épargne (saving account)
class SavingsAccount
{
    // Donnée au niveau de l'instance
    public double currBalance;

    // Point de donnée statiques (au niveau de la classe)
    private static double currInterestRate = 0.04;
    
    // Une propriété statique
    public static double InterestRate
    {
        get => currInterestRate;
        set => currInterestRate = value;
    }

	...
}
```

Si vous voulez utiliser cette propriété aux endroit où les anciennes méthodes statiques était utilisé, vous pourriez mettre à jour votre code comme suit:

```cs
// Affiche le taux d'intérèt via la propriété.
Console.WriteLine("Interest Rate is: {0}", SavingsAccount.InterestRate);
```

## Correspondance de motifs avec des motifs de propriété (Nouveauté C# 8.0)

**Le motif de propriété correspond à une expression lorsque le résultat de l'expression est non nul et que chaque motif imbriqué correspond à la propriété ou au champ correspondant du résultat de l'expression**. Autrement dit, ==le motif de propriété permet de faire correspondre les propriétés d'un objet==. Pour configurer l'exemple, ajoutez un nouveau fichier (*EmployeePayTypeEnum.cs*) au projet `EmployeeApp` pour énumérer les types de rémunération des employés, comme suit :

```cs
namespace EmployeeApp;

public enum EmployeePayTypeEnum
{
    Hourly,
    Salaried,
    Commission,
}
```

Mettez à jour la classe `Employee` avec une propriété pour le type de salaire et initialisez-la depuis le constructeur. Les modifications de code correspondantes sont listées ici :

```cs

...
private EmployeePayTypeEnum _payType;
public EmployeePayTypeEnum PayType
{
	 get => _payType;
	 set => _payType = value;
}

public Employee(string name, int id, float pay, string empSsn)
	: this(name,0,id,pay, empSsn, EmployeePayTypeEnum.Salaried)
{}

public Employee(string name, int age, int id,
	float pay, string empSsn, EmployeePayTypeEnum payType)
{
	 Name = name;
	 Id = id;
	 Age = age;
	 Pay = pay;
	 SocialSecurityNumber = empSsn;
	 PayType = payType;
}
...
```

Maintenant que tous les éléments sont en place, la méthode `GiveBonus()` peut être mise à jour en fonction du type de rémunération de l'employé. Les employés avec des commissions reçoivent 10 % de la prime, les employée payé à l'heure reçoivent l'équivalent de 40 heures de la prime au prorata, et les salariés reçoivent le montant saisi. La méthode `GiveBonus()` mise à jour est présentée ici :

```cs
public void GiveBonus(float amount)
{
    Pay = this switch
    {
        { PayType: EmployeePayTypeEnum.Commission } => Pay += .10F * amount,
        { PayType: EmployeePayTypeEnum.Hourly } => Pay +=
            40F * amount / 2080,
        { PayType: EmployeePayTypeEnum.Salaried } => Pay += amount,
        _ => Pay += 0,
    };
}
```

==Comme pour les autres instructions `switch` utilisant la correspondance de motifs, l'instruction `switch` doit soit contenir une instruction cas fourre-tout, soit lever une exception si aucune des instructions cas n'est remplie==.

Pour tester cela, ajoutez le code suivant aux instructions de niveau supérieur :

```cs
Employee emp3 = new Employee("Marvin", 45, 123, 1000, "111-11-1111", EmployeePayTypeEnum.Salaried);
Console.WriteLine(emp3.Pay);
emp3.GiveBonus(100);
Console.WriteLine(emp3.Pay);
```

**Plusieurs propriétés peuvent être utilisées dans le modèle**. Supposons que vous souhaitiez vous assurer que chaque employé recevant une prime a plus de 18 ans. Vous pouvez mettre à jour la méthode comme suit :

```cs
public void GiveBonus(float amount)
{
	 Pay = this switch
	 {
		 { Age: >= 18, PayType: EmployeePayTypeEnum.Commission } => Pay += 
		     .10F * amount,
		 { Age: >= 18, PayType: EmployeePayTypeEnum.Hourly } => Pay += 
		     40F * (amount/2080F),
		 { Age: >= 18, PayType: EmployeePayTypeEnum.Salaried } => Pay += 
		     amount,
		_ => Pay += 0
	 };
}
```

**Les modèles de propriétés peuvent être imbriqués pour naviguer dans la chaîne de propriétés. Pour illustrer cela, ajoutez une propriété publique pour `HireDate`, comme ceci :**

```cs
private DateTime _hireDate;
public DateTime HireDate 
{
    get => _hireDate; 
    set => _hireDate = value;
}
```

Ensuite, mettez à jour l'instruction switch pour vérifier que l'année d'embauche de chaque employé est postérieure à 2020 afin de lui donner droit à la prime :

```cs
public void GiveBonus(float amount)
{
    Pay = this switch
    {
        {
            Age: > -18,
            PayType: EmployeePayTypeEnum.Commission,
            HireDate: { Year: > 2020 }
        } => Pay += .10F * amount,
        {
            Age: >= 18,
            PayType: EmployeePayTypeEnum.Hourly,
            HireDate: { Year: > 2020 }
        } => Pay += 40F * (amount / 2080F),
        {
            Age: >= 18,
            PayType: EmployeePayTypeEnum.Salaried,
            HireDate: { Year: > 2020 }
        } => Pay += amount,
        _ => Pay += 0,
    };
}
```

## Modèles de propriétés étendus (Nouveauté C# 10.0)

Nouveauté de C# 10 : **les modèles de propriétés étendus peuvent être utilisés à la place de l'imbrication des propriétés en aval. Cette mise à jour corrige l'exemple précédent**, comme illustré ici :

```cs
public void GiveBonus(float amount)
{
    Pay = this switch
    {
        {
            Age: > -18,
            PayType: EmployeePayTypeEnum.Commission,
            // Simplification possible depuis C# 10
            HireDate.Year: > 2020
        } => Pay += .10F * amount,
        {
            Age: >= 18,
            PayType: EmployeePayTypeEnum.Hourly,
            // Simplification possible depuis C# 10
            HireDate.Year: > 2020
        } => Pay += 40F * (amount / 2080F),
        {
            Age: >= 18,
            PayType: EmployeePayTypeEnum.Salaried,
            // Simplification possible depuis C# 10
            HireDate.Year: > 2020
        } => Pay += amount,
        _ => Pay += 0,
    };
}
```

# Comprendre les propriétés automatiques

==Lorsque vous créez des propriétés pour encapsuler vos données, il est fréquent de constater que les portées définies contiennent du code pour appliquer les règles métier de votre programme==. Cependant, **dans certains cas, vous n'aurez besoin d'aucune logique d'implémentation autre que la simple obtention et la définition de la valeur. Cela signifie que vous risquez d'obtenir beaucoup de code, comme suit** :

```cs
// Type de véhicule d'employé utilisant la syntaxe de propriété standard.
class Car
{
	private string carName = "";
	public string PetName
	{
		get { return carName; }
		set { carName = value; }
	}
}
```

Dans ces cas, **définir plusieurs fois des champs de sauvegarde privés et des définitions de propriétés simples peut s'avérer complexe**. Par exemple, ==si vous modélisez une classe nécessitant neuf points de données de champ privés, vous finissez par créer neuf propriétés associées, qui ne sont rien de plus que de simples enveloppes pour les services d'encapsulation==.

Pour simplifier le processus d'encapsulation simple des données de champ, vous pouvez utiliser la *syntaxe de propriété automatique*. Comme son nom l'indique, **cette fonctionnalité décharge la définition d'un champ de sauvegarde privé et du membre de propriété C# associé  au compilateur grâce à une nouvelle syntaxe**. Par exemple, créez un nouveau projet d'application console nommé *AutoProps* et ajoutez un nouveau fichier de classe nommé *Car.cs*. Considérons maintenant cette refonte de la classe `Car`, qui utilise cette syntaxe pour créer rapidement trois propriétés :

```cs
namespace AutoProps;
class Car
{
   // Automatic properties! No need to define backing fields.
   public string PetName { get; set; }
   public int Speed { get; set; }
   public string Color { get; set; }
}
```

>[!tip] Une série de *snippets* est disponible comme raccourcis. `prop` générera une propriété automatique.

Lors de la définition de propriétés automatiques, **il suffit de spécifier le modificateur d'accès, le type de données sous-jacent, le nom de la propriété et les portées `get`/`set` vides**. ==À la compilation, votre type sera doté d'un champ de sauvegarde privé autogénéré et d'une implémentation appropriée de la logique `get`/`set`==.

>[!info]-
> Le nom du champ de sauvegarde privé généré automatiquement n'est pas visible dans votre code C#. La seule façon de le voir est d'utiliser un outil tel qu'*ildasm.exe*.

**Depuis C# version 6, il est possible de définir une propriété automatique en lecture seule en omettant la portée définie.** Les propriétés automatiques en lecture seule ne peuvent être définies que dans le constructeur. **En revanche, il est impossible de définir une propriété en écriture seule**. Pour plus de clarté, considérez ce qui suit :

```cs
// Propriété en lecture-seule? C'est OK!
public int MyReadOnlyProp { get; }

// Propriété en écriture-seule? Erreur!
public int MyWriteOnlyProp { set; }
```

## Interaction avec les propriétés automatiques

**Comme le compilateur définit le champ de sauvegarde privé à la compilation** (et que ces champs ne sont pas directement accessibles en C#), **les propriétés automatiques définissant la classe devront toujours utiliser la syntaxe de propriété pour obtenir et définir la valeur sous-jacente**. Ceci est important à noter, car ==de nombreux programmeurs utilisent directement les champs privés dans une définition de classe, ce qui est impossible dans ce cas==. Par exemple, si la classe `Car` fournissait une méthode `DisplayStats()`, elle devrait implémenter cette méthode en utilisant le nom de la propriété.

```cs
namespace AutoProps;

class Car
{
    // Propriétés automatiques! Pas besoin de définir
    // des champs de stockage.
    public string PetName { get; set; }
    public int Speed { get; set; }
    public string Color { get; set; }

    public void DisplayStats()
    {
        Console.WriteLine($"Car Name: {PetName}");
        Console.WriteLine($"Speed: {Speed}");
        Console.WriteLine($"Color: {Color}");
    }
}
```

Lorsque vous utilisez un objet défini avec des propriétés automatiques, vous pourrez attribuer et obtenir les valeurs en utilisant la syntaxe de propriété attendue.

```cs
using AutoProps;

Console.Title = "Fun with Automatic Properties";
Console.WriteLine("**** Fun with Automatic Properties ****\n");

Car c = new Car()
{
    PetName = "Frank",
    Speed = 55,
    Color = "Red",
};

Console.WriteLine($"Your car is named {c.PetName}? That's odd...");
c.DisplayStats();

Console.ReadLine();

```

## Propriétés automatiques et valeurs par défaut

==Lorsque vous utilisez des propriétés automatiques pour encapsuler des données numériques ou booléennes, vous pouvez utiliser les propriétés de type générées automatiquement directement dans votre code, car les champs de sauvegarde masqués se verront attribuer une valeur par défaut sûre== (`false` pour les booléens et `0` pour les données numériques). Cependant, **sachez que si vous utilisez la syntaxe de propriété automatique pour encapsuler une autre variable de classe, le type de référence privé masqué sera également défini sur `null`** (ce qui peut s'avérer problématique si vous n'y prêtez pas attention).

Insérons dans votre projet actuel un nouveau fichier de classe nommé *Garage.cs*, qui utilise deux propriétés automatiques (bien sûr, une classe garage réelle peut gérer une collection d'objets `Car` ; cependant, ignorez ce détail ici).

```cs
namespace AutoProps;

class Garage
{
    // Le champ de stockage caché de type int est assigné à zéro!
    public int NumberOfCars { get; set; }

    // Le champ de stockage caché de type Car est assigné à null!
    public Car MyAuto { get; set; }
}
```

==Étant donné les valeurs par défaut de C# pour les données de champ, vous pourriez afficher la valeur de `NumberOfCars` telle quelle== (la valeur zéro lui étant automatiquement attribuée), ==mais si vous appelez directement `MyAuto`, vous recevrez une exception de référence nulle à l'exécution==, car **la variable membre `Car` utilisée en arrière-plan n'a pas été affectée à un nouvel objet**.

```cs
...
Garage g = new Garage();

// OK, affiche la valeur par défault (0)
Console.WriteLine($"Number of Cars: {g.NumberOfCars}");

// Erreur à l'exécution! Le champ de stockage est actuellement null!
Console.WriteLine(g.MyAuto.PetName);
```

Pour résoudre ce problème, vous pouvez mettre à jour les constructeurs de classe afin de garantir que l'objet s'exécute de manière sécurisée. Voici un exemple :

```cs
class Garage
{
    // Le champ de stockage caché de type int est assigné à zéro!
    public int NumberOfCars { get; set; }

    // Le champ de stockage caché de type Car est assigné à null!
    public Car MyAuto { get; set; }

    // Doit utiliser les constructeurs pour surcharger
    // les valeurs par defauts assigné aux champs cachés.
    public Garage()
    {
        MyAuto = new Car();
        NumberOfCars = 1;
    }

    public Garage(Car car, int number)
    {
        MyAuto = car;
        NumberOfCars = number;
    }
}
```

Grâce à cette modification, vous pouvez désormais placer un objet `Car` dans l'objet `Garage` comme suit :

```cs
using AutoProps;

Console.Title = "Fun with Automatic Properties";
Console.WriteLine("**** Fun with Automatic Properties ****\n");

// Crée une instance de Car.
Car c = new Car()
{
    PetName = "Frank",
    Speed = 55,
    Color = "Red",
};

Console.WriteLine($"Your card is named {c.PetName}? That's odd...");
c.DisplayStats();

Console.ReadLine();

// Mets l'instance de Car dans le Garage.
Garage g = new() { MyAuto = c };
Console.WriteLine($"Number of Cars in garage: {g.NumberOfCars}");
Console.WriteLine($"Your car is named: {g.MyAuto.PetName}");

Console.ReadLine();

```

## Initialisation des propriétés automatiques

**Bien que l'approche précédente fonctionne, depuis la sortie de C# 6, une fonctionnalité du langage simplifie l'attribution de la valeur initiale à une propriété automatique**. Rappelons que, depuis le début de ce chapitre, une valeur initiale peut être attribuée directement à un champ de données d'une classe lors de sa déclaration. Voici un exemple :

```cs
class Car
{
	private int numberOfDoors = 2;
}
```

**De la même manière, C# vous permet désormais d'assigner une valeur initiale au champ de sauvegarde sous-jacent généré par le compilateur**. ==Cela vous évite d'avoir à ajouter des instructions de code dans les constructeurs de classe pour garantir que les données de propriété s'exécutent correctement==.

Voici une version mise à jour de la classe `Garage` qui initialise les propriétés automatiques avec des valeurs appropriées. Notez qu'il n'est plus nécessaire d'ajouter de logique à votre constructeur de classe par défaut pour effectuer des assignations sécurisées. Dans cette itération, vous assignez directement un nouvel objet `Car` à la propriété `MyAuto`.

```cs
namespace AutoProps;

class Garage
{
   // The hidden int backing field is set to zero!
   public int NumberOfCars { get; set; } = 1;

   // The hidden Car backing f ield is set to null!
   public Car MyAuto { get; set; } = new Car();

   // Must use constructor to override default values
   // assigned to hidden fields.
   public Garage() {}
   public Garage(Car car, int number)
   {
      MyAuto = car;
      NumberOfCars = number;
   }
}
```

**Comme vous le constaterez sans doute, les propriétés automatiques sont une fonctionnalité intéressante du langage de programmation C#, car elles permettent de définir plusieurs propriétés pour une classe grâce à une syntaxe simplifiée**. Sachez toutefois que ==si vous créez une propriété nécessitant du code supplémentaire au-delà de l'obtention et de la définition du champ privé sous-jacent== (comme la logique de validation des données, l'écriture dans un journal d'événements, la communication avec une base de données, etc.), ==vous devrez définir manuellement un type de propriété .NET Core « normal »==. **Les propriétés automatiques C# ne font jamais plus que fournir une simple encapsulation pour une donnée privée sous-jacente (générée par le compilateur)**.

# Comprendre l'initialisation des objets

Pour simplifier la création et l'utilisation d'un objet, C# propose une *syntaxe d'initialisation d'objet*. **Grâce à cette technique, il est possible de créer une nouvelle variable objet et de lui assigner plusieurs propriétés et/ou champs publics en quelques lignes de code**. ==Syntactiquement, un initialiseur d'objet se compose d'une liste de valeurs spécifiées, séparées par des virgules et encadrées par les tokens `{` et `}`==. Chaque élément de la liste d'initialisation correspond au nom d'un champ public ou d'une propriété publique de l'objet initialisé.

Pour observer cette syntaxe en pratique, créez un nouveau projet d'application console nommé *ObjectInitializers*. Prenons ensuite l'exemple d'une classe simple nommée `Point`, créée à l'aide de propriétés automatiques (ce qui n'est pas obligatoire pour la syntaxe d'initialisation d'objet, mais permet d'écrire un code plus concis).

```cs
class Point
{
    public int X { get; set; }
    public int Y { get; set; }

    public Point(int xVal, int yVal)
    {
        X = xVal;
        Y = yVal;
    }

    public Point() { }

    public void DisplayStats()
    {
        Console.WriteLine("{0}, {1}", X, Y);
    }
}

```

Voyez maintenant comment créer des objets `Point` en utilisant l'une des approches suivantes :

```cs
using ObjectInitializers;

Console.Title = "Fun with Object Init Syntax";
Console.WriteLine("***** Fun with Object Init Syntax *****\n");

// Crée un Point en assignant chaque propriété manuellement.
Point firstPoint = new Point();
firstPoint.X = 10;
firstPoint.Y = 10;
firstPoint.DisplayStats();

// Ou crée un Point avec un constructeur personnalisé.
Point anotherPoint = new Point(20, 20);
anotherPoint.DisplayStats();

// Ou crée un Point en utilisant
// la syntaxe d'initialisation d'objet
Point finalPoint = new Point { X = 30, Y = 30 };
finalPoint.DisplayStats();
Console.ReadLine();
```

**La variable `Point` finale n'utilise pas de constructeur personnalisé** (comme on le ferait traditionnellement), **mais affecte directement des valeurs aux propriétés publiques X et Y**. En coulisses, ==le constructeur par défaut du type est appelé, puis les valeurs des propriétés spécifiées sont initialisées==. Ainsi, la syntaxe d'initialisation d'objet est une notation abrégée de la syntaxe utilisée pour créer une variable de classe à l'aide d'un constructeur par défaut et pour définir
les données d'état propriété par propriété.

>[!Attention] 
>Il est important de se rappeler que le processus d'initialisation de l'objet utilise implicitement le setter de propriété. **Si le setter de propriété est déclaré `private`, cette syntaxe ne peut pas être utilisée**.

## Utilisation des setters d'initialisation (Nouveauté C# 9.0)

Une nouvelle fonctionnalité de C# 9.0 est celle des setters d'initialisation. **Ces setters permettent de définir la valeur d'une propriété lors de l'initialisation, mais une fois la construction de l'objet terminée, la propriété devient en lecture seule**. ==Ces types de propriétés sont dits *immuables*==. Ajoutez un nouveau fichier de classe nommé *ReadOnlyPointAfterCreation.cs* à votre projet et ajoutez le code suivant :

```cs
namespace ObjectInitializers;

class ReadOnlyPointAfterCreation
{
    public int X { get; init; }
    public int Y { get; init; }

    public void DisplayStats()
    {
        Console.WriteLine($"InitOnlySetter: [{X}, {Y}]");
    }

    public ReadOnlyPointAfterCreation(int xVal, int yVal)
    {
        X = xVal;
        Y = yVal;
    }

    public ReadOnlyPointAfterCreation() { }
}
```

Utilisez le code suivant pour tester cette nouvelle classe:

```cs
// Crée un Point en lecture seule après construction.
ReadOnlyPointAfterCreation firstReadonlyPoint = new(20, 20);
firstReadonlyPoint.DisplayStats();

// Ou crée un Point en utilisant la syntaxe d'initialisation d'objet.
ReadOnlyPointAfterCreation secondReadonlyPoint = new() { X = 30, Y = 30 };
secondReadonlyPoint.DisplayStats();
```

==Remarquez que rien n'a changé par rapport au code que vous avez écrit pour la classe `Point`==, à l'exception bien sûr du nom de la classe. **La différence réside dans le fait que les valeurs de `X` et `Y` ne peuvent plus être modifiées une fois la classe créée**. Par exemple, le code suivant ne compilera pas :

```cs
// Les deux prochaines lignes ne compileront pas (erreur CS8852):
secondReadonlyPoint.X = 10;
secondReadonlyPoint.Y = 10;
```

## Appel de constructeurs personnalisés avec la syntaxe d'initialisation 

Les exemples précédents initialisaient les types `Point` en appelant implicitement le constructeur par défaut sur le type.

```cs
// Ici, le constructeur par défaut est appelé implicitement.
Point finalPoint = new Point { X = 30, Y = 30 };
```

Pour être clair, il est permis d'appeler explicitement le constructeur par défaut comme suit :

```cs
// Ici, le constructeur par défaut est appelé explicitement.
Point finalPoint = new Point() { X = 30, Y = 30 };
```

**Sachez que lorsque vous créez un type à l'aide de la syntaxe d'initialisation, vous pouvez appeler *n'importe quel* constructeur défini par la classe**. Votre type `Point` définit actuellement un constructeur à deux arguments pour définir la position (*x*, *y*). ==Par conséquent, la déclaration `Point` suivante donne une valeur `X` de $100$ et une valeur `Y` de $100$, même si les arguments du constructeur spécifient les valeurs $10$ et $16$ :

```cs
// Appel un constructeur personnalisé
Point pt = new Point(10, 16) { X = 100, Y = 100 };
```

Compte tenu de la définition actuelle de votre type `Point`, ==l'appel du constructeur personnalisé avec la syntaxe d'initialisation est peu utile== (et plutôt verbeux). Cependant, ==si votre type `Point` fournit un
nouveau constructeur permettant à l'appelant de définir une couleur== (via une énumération personnalisée nommée `PointColor`), ==l'utilité de la combinaison de constructeurs personnalisés et de la syntaxe d'initialisation d'objet devient évidente==.

Ajoutez une nouvelle classe nommée *PointColorEnum.cs* à votre projet et ajoutez le code suivant pour créer une énumération pour la couleur :

```cs
namespace ObjectInitializers;

enum PointColorEnum
{
    LightBlue,
    BloodRed,
    Gold,
}
```

Mettez maintenant à jour la classe `Point` comme suit:

```cs
namespace ObjectInitializers;

class Point
{
    public int X { get; set; }
    public int Y { get; set; }
    public PointColorEnum Color { get; set; }

    public Point(int xVal, int yVal)
    {
        X = xVal;
        Y = yVal;
        Color = PointColorEnum.Gold;
    }

    public Point(PointColorEnum ptColor)
    {
        Color = ptColor;
    }

    public Point()
        : this(PointColorEnum.BloodRed) { }

    public void DisplayStats()
    {
        Console.WriteLine("{0}, {1}", X, Y);
        Console.WriteLine("Point is {0}", Color);
    }
}
```

Avec ce nouveau constructeur, vous pouvez maintenant créer un point de couleur or (positionné en $90$, $20$) comme suit:

```cs
// Appel un constructeur personnalisé plus intéressant
// avec la syntaxe d'initialisation d'objet.
Point goldPoint = new Point(PointColorEnum.Gold) { X = 90, Y = 20 };
goldPoint.DisplayStats();
```

## Initialisation des données avec la syntaxe d'initialisation

Comme mentionné brièvement plus haut dans ce chapitre (et examiné en détail au [[Chapitre 6#Programmation pour la délégation/l'inclusion|Chapitre 6]]), ==la relation « a un » vous permet de composer de nouvelles classes en définissant des variables membres de classes existantes==. Par exemple, supposons que vous ayez maintenant une classe `Rectangle`, qui utilise le type `Point` pour représenter ses coordonnées coin supérieur gauche/coin inférieur droit.

Étant donné que les propriétés automatiques initialisent tous les champs des variables de classe à `null`, vous implémenterez cette nouvelle classe en utilisant la syntaxe de propriété « traditionnelle ».

```cs
namespace ObjectInitializers;

class Rectangle
{
    private Point topLeft = new Point();
    private Point bottomRight = new Point();

    public Point TopLeft
    {
        get => topLeft;
        set => topLeft = value;
    }
    public Point Bottomright
    {
        get => bottomRight;
        set => bottomRight = value;
    }

    public void DisplayStats()
    {
        Console.WriteLine(
            "[TopLeft: {0}, {1}, {2} BottomRight: {3}, {4}, {5}]",
            topLeft.X,
            topLeft.Y,
            topLeft.Color,
            bottomRight.X,
            bottomRight.Y,
            bottomRight.Color
        );
    }
}
```

En utilisant la syntaxe d'initialisation d'objet, vous pouvez créer une nouvelle variable `Rectangle` et définir des `Point` internes comme suit :

```cs
// Création et initialisation d'une variable Rectangle
Rectangle myRect = new()
{
    TopLeft = new Point { X = 10, Y = 10 },
    Bottomright = new Point { X = 200, Y = 200 },
};
```

L'avantage de la syntaxe d'initialisation d'objet réside encore une fois dans la ==réduction du nombre de frappes au clavier== (en supposant qu'il n'existe pas de constructeur approprié). Voici l'approche traditionnelle pour créer un objet `Rectangle` similaire :

```cs
// Approche à l'ancienne
Rectangle r = new Rectangle();

Point p1 = new Point();
p1.X = 10;
p1.Y = 10;

r.TopLeft = p1;

Point p2 = new Point();
p2.X = 200;
p2.Y = 200;

r.Bottomright = p2;

```

Bien que la syntaxe d'initialisation des objets puisse sembler nécessiter un certain temps d'adaptation, une fois que vous serez à l'aise avec le code, vous serez très satisfait de la rapidité avec laquelle vous pourrez établir l'état d'un nouvel objet sans le moindre problème.

# Utilisation des données de champs constants et en lecture seule

==Parfois, vous avez besoin d'une propriété que vous ne souhaitez absolument pas modifier, également appelée *immuable*==, que ce soit depuis sa compilation ou après sa définition lors de la construction du programme. Nous avons déjà exploré un exemple avec les setters réservés à l'initialisation (mot clé `init`). Nous allons maintenant examiner les constantes et les champs en lecture seule.

## Comprendre les données de champs constants

**C# offre le mot-clé `const` pour définir des données constantes, qui ne peuvent plus être modifiées après leur affectation initiale**. Comme vous pouvez l'imaginer, ==cela peut s'avérer utile lorsque vous définissez un ensemble de valeurs connues à utiliser dans vos applications, et qui sont logiquement liées à une classe ou une structure donnée==.

Supposons que vous développiez une classe utilitaire nommée `MyMathClass` qui doit définir une valeur pour `pi` (que vous supposerez égale à 3,14 par souci de simplicité). Commencez par créer un nouveau projet d'application console nommé *ConstData* et ajoutez un fichier nommé *MyMathClass.cs*. Étant donné que vous ne souhaitez pas autoriser d'autres développeurs à modifier cette valeur dans le code, `pi` peut être modélisé par la constante suivante :

```cs
namespace ConstData;

class MyMathClass
{
    public const double PI = 3.14;
}
```

Mettez à jour le code dans le fichier *Program.cs* pour correspondre à cela:

```cs
using ConstData;

Console.Title = "Fun with Const";
Console.WriteLine("**** Fun with Const ****\n");

Console.WriteLine($"The value of PI is: {MyMathClass.PI}");

// Erreur! Ne peut pas changer une constante!
// MyMathClass.PI = 3.144444;

Console.ReadLine();

```

==Notez que vous faites référence aux données constantes définies par `MyMathClass` en utilisant le préfixe du nom de la classe== (c.-à-d., `MyMathClass.PI`). En effet, **les champs constants d'une classe sont implicitement *`static`***. Toutefois, **il est permis de définir et d'accéder à une variable constante locale dans la portée d'une méthode ou d'une propriété**. Voici un exemple :

```cs
static void LocalConstStringVariable()
{
    // Un point de donnée constante locale peut être dirrectement accédé.
    const string fixedString = "Fixed string Data";

    Console.WriteLine(fixedString);
    // Erreur!
    // FixedString = "This will not work!"
}
```

Peu importe où vous définissez une donnée constante, **il est essentiel de se rappeler que la valeur initiale de la constante doit être spécifiée lors de sa définition**. ==L'affectation de la valeur de pi dans le constructeur d'une classe, comme illustré dans le code suivant, provoque une erreur de compilation== :

```cs
namespace ConstData;

class MyMathClass
{
    // Essaye d'assigné PI dans le constructeur?
    public const double PI;
    public MyMathClass()
    {
        // Pas possible - dois assigné au moment de la déclaration.
        PI = 3.14;
    }
}
```

### Les chaînes de caractères interpolées constantes (Nouveauté C# 10.0)

Introduites dans C# 10, **les valeurs de type `const string` peuvent utiliser l'interpolation de chaînes dans leurs instructions d'affectation, à condition que tous les composants utilisés soient également des chaînes constantes**. À titre d'exemple simple, ajoutez le code suivant aux instructions de niveau supérieur :

```cs
static void ConstInterpolatedString()
{
    Console.WriteLine("=> Constant String Interpolation");
    const string foo = "Foo";
    const string bar = "Bar";
    const string foobar = $"{foo}{bar}";
    Console.WriteLine(foobar);
}

```

## Comprendre les champs en lecture seule

Étroitement liée aux données constantes est la notion de *champ de donnée en lecture seule* (à ne pas confondre avec une propriété en lecture seule). **Comme une constante, un champ en lecture seule ne peut être modifié après son affectation initiale, sous peine de provoquer une erreur de compilation**. ***==Cependant, contrairement à une constante, la valeur affectée à un champ en lecture seule peut être déterminée à l'exécution et, par conséquent, peut légalement lui être affectée dans la portée d'un constructeur, mais nulle part ailleurs==***.

**==Ceci peut s'avérer utile lorsque la valeur d'un champ est inconnue jusqu'à l'exécution, par exemple parce qu'il est nécessaire de lire un fichier externe pour obtenir cette valeur, tout en s'assurant qu'elle ne changera pas par la suite==**. À titre d'illustration, prenons l'exemple de la mise à jour suivante de la classe `MyMathClass` :

```cs
class MyMathClass
{

    // Les champs le lecture seules peuvent être assigné
    // dans un constructeur, mais nulle part ailleur.
    public readonly double PI = 3.14;

    public MyMathClass()
    {
        PI = 3.14;
    }
}
```

**Là encore, toute tentative d'affectation à un champ marqué `readonly` en dehors de la portée d'un constructeur, entraîne une erreur de compilation :**

```cs
class MyMathClass
{
    public readonly double PI = 3.14;

    public MyMathClass()
    {
        PI = 3.14;
    }

    // Erreur!
    public void ChangePI()
    {
        PI = 3.14;
    }
}
```

## Comprendre les champs `static` en lecture seule

Contrairement aux champs constants, ==les champs en lecture seule ne sont pas statiques par défaut==. Par conséquent, **si vous souhaitez exposer `PI` au niveau de la classe, vous devez utiliser explicitement le mot-clé `static`**. ==Si vous connaissez la valeur d'un champ statique en lecture seule à la compilation, l'affectation initiale ressemble à celle d'une constante== 

>[!note]
cependant, dans ce cas, il serait plus simple d'utiliser directement le mot-clé `const`, puisque vous affectez la valeur du champ de données lors de sa déclaration).

```cs
// MyMathClass.cs
class MyMathClass
{
    public static readonly double PI = 3.14;
}
```

```cs
// Program.cs
using ConstData;

Console.Title = "Fun with Const";
Console.WriteLine("**** Fun with Const ****\n");

Console.WriteLine($"The value of PI is: {MyMathClass.PI}");
Console.ReadLine();
```

**Toutefois, si la valeur d'un champ statique en lecture seule n'est connue qu'à l'exécution, vous devez utiliser un constructeur statique comme décrit précédemment dans ce chapitre.**

```cs
namespace ConstData;

class MyMathClass
{
    public static readonly double PI;

    static MyMathClass()
    {
        PI = 3.14;
    }
}
```

# Comprendre les classes partielles

Lorsqu'on travaille avec des classes, il est important de ==comprendre le rôle du mot-clé `partial` en C#==. **Ce mot-clé permet de répartir une classe unique sur plusieurs fichiers de code**. ==Lorsque vous générez des classes *Entity Framework Core* à partir d'une base de données, elles sont toutes créées en tant que classes partielles==. Ainsi, tout code que vous avez écrit pour compléter ces fichiers n'est pas écrasé, ==à condition qu'il se trouve dans des fichiers de classe distincts marqués avec le mot-clé `partial`==. **Une autre raison est que votre classe peut être devenue, au fil du temps, difficile à gérer**. Dans ce cas, ==comme étape intermédiaire vers sa refactorisation, vous pouvez la diviser en classes partielles==.

**En C#, vous pouvez répartir une classe unique sur plusieurs fichiers de code afin d'isoler le code répétitif des membres plus utiles** (et complexes). Pour illustrer l'utilité des classes partielles, ouvrez le projet `EmployeeApp` que vous avez créé précédemment dans ce chapitre, puis ouvrez le fichier `Employee.cs` pour le modifier. Comme vous vous en souvenez peut-être, ce fichier unique contient le code de tous les aspects de la classe.

```cs
class Employee
{
    // Donnée de champs

    // Constructeurs

    // Methodes

    // Propriétés
}
```

En utilisant des classes partielles, vous pouvez déplacer (par exemple) les propriétés, les constructeurs et les données des champs dans un nouveau fichier nommé `Employee.Core.cs` (le nom du fichier est sans importance). ==La première étape consiste à ajouter le mot-clé `partial` à la définition de la classe actuelle et à découper le code à placer dans le nouveau fichier.

```cs
// Employee.cs
public partial class Employee
{
    // Constructeurs
    public Employee() { }

    public Employee(string name, int id, float pay, string ssn)
        : this(name, 0, id, pay, ssn, EmployeePayTypeEnum.Salaried) { }

    public Employee(
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
        // Propriété en lecture seule, il faut assigné le champs à la place.
        _empSSN = ssn;
        PayType = payType;
    }

    // Méthodes
    public void GiveBonus(float amount)
    {
        Pay = this switch
        {
            {
                Age: > -18,
                PayType: EmployeePayTypeEnum.Commission,
                HireDate: { Year: > 2020 }
            } => Pay += .10F * amount,
            {
                Age: >= 18,
                PayType: EmployeePayTypeEnum.Hourly,
                HireDate: { Year: > 2020 }
            } => Pay += 40F * (amount / 2080F),
            {
                Age: >= 18,
                PayType: EmployeePayTypeEnum.Salaried,
                HireDate: { Year: > 2020 }
            } => Pay += amount,
            _ => Pay += 0,
        };
    }

    public void DisplayStats()
    {
        Console.WriteLine($"Name: {Name}");
        Console.WriteLine($"Age: {Age}");
        // Propriété en écriture seule, il faut lire le champs à la place.
        Console.WriteLine($"ID: {_empId}");
        Console.WriteLine($"SSN: {SocialSecurityNumber}");
        Console.WriteLine($"Pay: {Pay}");
    }

    public string GetName() => _empName;

    public void SetName(string name) => _empName = name;
}

```

Ensuite, ==en supposant que vous ayez inséré un nouveau fichier de classe dans votre projet, vous pouvez déplacer les champs de données et les propriétés vers ce nouveau fichier à l'aide d'une simple opération de couper-coller==. De plus, vous devez ajouter le mot-clé `partial` à cette partie de la définition de la classe. Voici un exemple :

```cs
// Employee.Core.cs
public partial class Employee
{
    // Champs de données.
    private string _empName;
    private int _empId;
    private float _currPay;
    private int _empAge;
    private string _empSSN;
    private EmployeePayTypeEnum _payType;
    private DateTime _hireDate;

    // Propriétés.
    public string Name
    {
        get => _empName;
        set
        {
            // Ici, la valeur est réellement un string.
            if (value.Length > 15)
            {
                Console.WriteLine("Error! Name length exceeds 15 characters!");
            }
            else
            {
                _empName = value;
            }
        }
    }

    // Propriété en écriture seule
    public int Id
    {
        set { _empId = value; }
    }

    public float Pay
    {
        get { return _currPay; }
        set { _currPay = value; }
    }
    public int Age
    {
        get { return _empAge; }
        set { _empAge = value; }
    }

    // Propriété en lecture seule
    public string SocialSecurityNumber
    {
        get { return _empSSN; }
    }
    public EmployeePayTypeEnum PayType
    {
        get => _payType;
        set => _payType = value;
    }
    public DateTime HireDate
    {
        get => _hireDate;
        set => _hireDate = value;
    }
}
```

>[!note]  N'oubliez pas que **chacune des classes** partielles doit être marquée avec le mot-clé `partial` !

>[!warning]- Pour les constructeurs primaires (C# 12+)
>
>1. Les **paramètres** du constructeur primaire ne peuvent être déclarés que sur **une seule** des parties de la classe (généralement le fichier principal).
>2. On peux déclarer des propriétés ou des champs dans **n'importe quel** fichier `partial`, mais ils ne pourront être initialisés par les paramètres du constructeur primaire que s'ils sont dans le **même fichier** (ou via un bloc d'initialisation dans ce même fichier).

Après avoir compilé le projet modifié, vous ne devriez constater aucune différence. Le concept de classe partielle se concrétise uniquement lors de la conception. **Une fois l'application compilée, l'assembly ne contient qu'une seule classe**. ==La seule condition pour définir des types partiels est que leur nom== (`Employee` dans ce cas) ==soit identique et défini **dans le même espace de noms .NET Core**==.

==Comme évoqué précédemment concernant les instructions de niveau supérieur, toute méthode dans ces instructions doit être une fonction locale==. **Les instructions de niveau supérieur sont implicitement définies dans une classe partielle `Program`, ce qui permet de créer une autre classe partielle `Program` pour y placer des méthodes classiques**.

Créez une application console nommée *FunWithPartials* et ajoutez un fichier de classe nommé *Program.Partial.cs*. Mettez à jour le code comme suit :

```cs
public partial class Program
{
    public static string SayHello() => "Hello";
}

```

Vous pouvez maintenant appeler cette méthode depuis vos instructions de niveau supérieur dans le fichier *Program.cs*, comme ceci :

```cs
Console.Title = "Fun with Partials";
Console.WriteLine("**** Fun with Partials ****\n");

Console.WriteLine(SayHello());
```

Le choix de la méthode est une question de préférence.

# Le type de donnée `record` (Nouveauté C# 9.0)

Nouveauté de C# 9.0, ==les *types enregistrement* sont un type référence spécial qui fournit des méthodes synthétisées pour l'égalité utilisant la sémantique des valeurs et l'encapsulation des données==. Les types `record` peuvent être créés avec des propriétés immuables ou standard. Pour commencer à expérimenter avec les enregistrements, créez une nouvelle application console nommée *FunWithRecords*. Considérez la classe `Car` suivante, modifiée à partir des exemples précédents de ce chapitre :

```cs
namespace FunWithRecords;

class Car
{
    public string Make { get; set; }
    public string Model { get; set; }
    public string Color { get; set; }

    public Car() { }

    public Car(string make, string model, string color)
    {
        Make = make;
        Model = model;
        Color = color;
    }
}
```

Comme vous le savez déjà, ==une fois que vous avez créé une instance de cette classe, vous pouvez modifier n'importe laquelle de ses propriétés à l'exécution==. Si les propriétés de la classe `Car` précédente doivent être immuables, vous pouvez modifier leurs définitions de propriétés pour utiliser des setters réservés à l'initialisation, comme ceci :

```cs
public string Make { get; init; }
public string Model { get; init; }
public string Color { get; init; }
```

Pour tester cette nouvelle classe, le code suivant crée deux instances de la classe `Car` : ==l’une par initialisation d’objet et l’autre par le constructeur personnalisé==. Mettez à jour le fichier *Program.cs* comme suit :

```cs
using FunWithRecords;

Console.Title = "Fun with Records";
Console.WriteLine("**** Fun with Records ****\n");

UseCarClass();

static void UseCarClass()
{
    Console.WriteLine("*************** CLASS ***************");
    // Utilisation de l'initialisation d'objet.
    Car myCar = new Car
    {
        Make = "Honda",
        Model = "Pilot",
        Color = "Blue",
    };
    Console.WriteLine("My car: ");
    DisplayCarStats(myCar);
    Console.WriteLine();

    // Utilisation du constructeur personnalisé.
    Car anotherMyCar = new Car("Honda", "Pilot", "Blue");
    Console.WriteLine("Another variable for my car: ");
    DisplayCarStats(anotherMyCar);
    Console.WriteLine();

    // Erreur de compilation si la propriété est modifiée.
    // myCar.Color = "Red";
    Console.ReadLine();

    static void DisplayCarStats(Car c)
    {
        Console.WriteLine("Car Make: {0}", c.Make);
        Console.WriteLine("Car Model: {0}", c.Model);
        Console.WriteLine("Car Color: {0}", c.Color);
    }
}
```

Comme prévu, les deux méthodes de création d'objets fonctionnent, les propriétés s'affichent et toute tentative de modification d'une propriété après la construction de l'objet provoque une erreur de compilation.

## Types `record` immuables avec syntaxe de propriété standard

==Créer un type d'enregistrement `Car` immuable à l'aide de la syntaxe de propriété standard est similaire à la création de classes avec des propriétés immuables==. Pour voir cela en pratique, ajoutez un nouveau fichier nommé *CarRecord.cs* à votre projet et ajoutez le code suivant :

```cs
using FunWithRecords;

record CarRecord
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
}
```

>[!info]- Facultatif
>Les types d'enregistrement permettent d'utiliser le mot-clé `class` pour les distinguer des `recrod struct`, mais ce mot-clé est facultatif. Par conséquent, `record class` et `record` signifie la même chose. 

Vous pouvez vérifier que le comportement est identique à celui de la classe `Car` avec les paramètres d'initialisation uniquement en exécutant le code suivant dans *Program.cs* :

```cs
static void UseCarRecord()
{
    Console.WriteLine("*************** RECORDS ***************");
    // Utilisation de l'initialisation d'objets.
    CarRecord myCarRecord = new CarRecord
    {
        Make = "Honda",
        Model = "Pilot",
        Color = "Blue",
    };
    Console.WriteLine("My car: ");
    DisplayCarRecordStats(myCarRecord);
    Console.WriteLine();

    // Utilisation du constructeur personnalisé.
    CarRecord anotherMyCarRecord = new CarRecord("Honda", "Pilot", "Blue");
    Console.WriteLine("Another variable for my car: ");
    Console.WriteLine(anotherMyCarRecord.ToString());
    Console.WriteLine();

    // Erreur de compilation si la propriété est changée.
    // myCarRecord.Color = "Red";
    Console.ReadLine();

    static void DisplayCarRecordStats(CarRecord c)
    {
        Console.WriteLine("Car Make: {0}", c.Make);
        Console.WriteLine("Car Model: {0}", c.Model);
        Console.WriteLine("Car Color: {0}", c.Color);
    }
}
```

>Attention, il y a une coquille dans le livre (la signature et le nom de la méthode `DisplayCarRecordStats` n'est pas correcte).

Bien que nous n'ayons pas encore abordé l'égalité (section suivante) ni l'héritage (chapitre suivant) avec les enregistrements, **ce premier aperçu des enregistrements ne semble pas apporter d'avantages significatifs**. L'exemple actuel `Car`  inclut tout le code d'infrastructure auquel nous sommes habitués. **Une différence notable toutefois dans le résultat : la méthode `ToString()` est optimisée pour les types enregistrements, comme le montre l'exemple de résultat suivant** :

```
*************** RECORDS ***************
My car:
Car Make: Honda
Car Model: Pilot
Car Color: Blue

Another variable for my car:
CarRecord { Make = Honda, Model = Pilot, Color = Blue }
```

## Types d'enregistrements immuables avec syntaxe positionnelle

Voici une définition mise à jour (et ==fortement abrégée==) de l'enregistrement `Car` :

```cs
record CarRecord(string Make, string Model, string Color);
```

Qualifié de type d'*enregistrement positionnel*, ==le constructeur définit les propriétés de l'enregistrement, et tout le reste du code de gestion a été supprimé==. **Trois points sont à prendre en compte lors de l'utilisation de cette syntaxe** : 

1) *==L'initialisation d'objets des types d'enregistrement avec la syntaxe de définition compacte est impossible==* (Si un constructeur par défaut n'est pas déclaré! Voir les explications précédentes sur [[Chapitre 4#Le constructeur primaires de structure (Nouveauté C 12)|les constructeurs primaires]]).
2) L'enregistrement doit être construit avec les propriétés correctement positionnées 
3) La casse des propriétés dans le constructeur est directement traduite en casse des propriétés du type d'enregistrement.

**On peut confirmer que les propriétés `Make`, `Model` et `Color` de l'enregistrement `Car` ​​sont toutes des propriétés d'initialisation uniquement en consultant un extrait du code IL**. On remarque la présence de champs de stockage pour chaque paramètre passé au constructeur, chacun étant associé aux modificateurs `private` et `initonly`.

```CIL
.class private auto ansi beforefieldinit FunWithRecords.CarRecord
	extends [System.Runtime]System.Object
	implements class [System.Runtime]System.IEquatable`1<class FunWithRecords.CarRecord>
{
	.field private initonly string '<Make>k__BackingField'
	.field private initonly string '<Model>k__BackingField'
	.field private initonly string '<Color>k__BackingField'
...
}
```

>[!info]- Différence entre le code IL du livre et le code désassemblé actuel
>Entre C# 10 et C# 14 (et le processeur de language *roslyn*), les codes IL modernes sont beaucoup plus verbeux que les exemples IL illustrée dans le livre. C'est un comportement normal.

**Lorsqu'on utilise la syntaxe positionnelle, les types d'enregistrement fournissent un constructeur principal qui correspond aux paramètres positionnels de la déclaration d'enregistrement.**

### Déconstruction des types d'enregistrements mutables

**Les types d'enregistrements utilisant des paramètres positionnels fournissent également une méthode `Deconstruct()` avec un paramètre `out` pour chaque paramètre positionnel de la déclaration**. Le code suivant crée un nouvel enregistrement à l'aide du constructeur fourni, puis décompose les propriétés en variables distinctes :

```cs
static void UseDeconstruct()
{
    Console.WriteLine("*************** DECONSTRUCT ***************");
    CarRecord myCarRecord = new CarRecord("Honda", "Pilot", "Blue");
    myCarRecord.Deconstruct(
        out string make,
        out string model,
        out string color
    );
    Console.WriteLine($"Make: {make}, Model: {model}, Color: {color}");
}
```

Notez que si les propriétés publiques de l'enregistrement respectent la casse de la déclaration, ==les variables `out` de la méthode `Deconstruct()` doivent seulement respecter la position des paramètres==. **Modifier les noms des variables dans la méthode `Deconstruct()` renvoie toujours `Make`, `Model` et `Color`, dans cet ordre :**

```cs
static void UseDeconstruct()
{
	...
    myCarRecord.Deconstruct(out string a, out string b, out string c);
    Console.WriteLine($"Make: {a}, Model: {b}, Color: {c}");
}
```

**La syntaxe des tuples peut également être utilisée lors de la déconstruction d'enregistrements**. Notez l'ajout suivant à l'exemple :

```cs
// Appel implicitement la méthode Deconstruct().
var (make, model, color) = myCarRecord;
Console.WriteLine($"Make: {make}, Model: {model}, Color: {color}");
```

>[!important] **La méthode `deconstruct` est un concept universelle et peut être crée aussi pour les `struct`, les `record` et les `class`.**
>
> L'avantage du type `record` est que son implémentation est déjà faite par la tuyauterie interne de C# !
>>[!warning] **Complément important :**
>>
>>- **Pas d'interface ([[Chapitre 8]]) :** C# utilise le _Duck Typing_ (ou approche par motif). Si la méthode s'appelle `Deconstruct` avec des paramètres `out`, ça fonctionne.
>>- **Méthodes d'extension ([[Chapitre 11#Comprendre les méthodes d'extension|Chapitre 11]]) :** On peut étendre des types tiers. Exemple : ajouter un `Deconstruct` à la classe `System.DateTime` pour extraire d'un coup _(jour, mois, année)_.

## Types d'enregistrements mutables
C# prend également en charge les types d'enregistrements mutables grâce à l'utilisation de setters standard (et non uniquement `init`). Voici un exemple :

```cs
record CarRecord
{
	public string Make { get; set; }
	public string Model { get; set; }
	public string Color { get; set; }

	public CarRecord() {}
	public CarRecord(string make, string model, string color)
	{
		Make = make;
		Model = model;
		Color = color;
	}
}
```

**==Bien que cette syntaxe soit prise en charge, les types d'enregistrements sont destinés à être utilisés pour des modèles de données immuables.==**

## Égalité des valeurs avec les types d'enregistrements

Dans l'exemple de la classe `Car`, les deux instances de `Car` ont été créées avec les mêmes données. ==On pourrait penser que ces deux classes sont égales==, comme le vérifie la ligne de code suivante :

```cs
static void UseClass()
{
    Console.WriteLine("*************** CLASS ***************");
    // Utilisation de l'initialisation d'objet.
    Car myCar = new Car
    {
        Make = "Honda",
        Model = "Pilot",
        Color = "Blue",
    };
	
	...
    
    // Utilisation du constructeur personnalisé.
    Car anotherMyCar = new Car("Honda", "Pilot", "Blue");
	
	...

    Console.WriteLine($"Cars are the same? {myCar.Equals(anotherMyCar)}");
    Console.ReadLine();
	
	...
}
```

**Cependant, ils ne sont pas égaux**. Rappelons que ==les types enregistrement sont un type spécialisé de classe, et que les classes sont des types référence==. **Pour que deux types référence soient égaux, ils doivent pointer vers le même objet en mémoire**. À titre de test supplémentaire, vérifiez si les deux objets `Car` pointent vers le même objet.

```cs
Console.WriteLine($"Cars are the same reference? {ReferenceEquals(myCar, anotherMyCar)}");
```

L'exécution du programme à nouveau produit ce résultat (abrégé) :

```
Cars are the same? False
CarRecords are the same? False
```

==Les types d'enregistrement se comportent différemment==. **Ils redéfinissent implicitement les méthodes `Equals`, `==` et `!=`, et ==deux enregistrements de type enregistrement sont considérés comme égaux s'ils contiennent les mêmes valeurs et sont du même type, comme s'il s'agissait d'instances de types valeur==**. Considérez le code suivant et les résultats obtenus :

```cs
// Compare les deux instances CarRecord avec les méthodes Equals et ReferenceEquals.
Console.WriteLine(
    $"CarRecords are the same? {myCarRecord.Equals(anotherMyCarRecord)}"
);
Console.WriteLine(
    $"CarRecords are the same reference? {ReferenceEquals(myCarRecord, anotherMyCarRecord)}"
;

// compare les deux instances CarRecord avec les opérateur de comparaison == et !=.
Console.WriteLine(
    $"CarRecords are the same? {myCarRecord == anotherMyCarRecord}"
);
Console.WriteLine(
    $"CarRecords are not the same? {myCarRecord != anotherMyCarRecord}"
);
```

```
*************** RECORDS ***************
My car:
Car Make: Honda
Car Model: Pilot
Car Color: Blue

Another variable for my car:
CarRecord { Make = Honda, Model = Pilot, Color = Blue }

CarRecords are the same? True
CarRecords are the same reference? False
CarRecords are the same? True
CarRecords are not the same? False
```

**Notez qu'elles sont considérées comme égales, même si les variables pointent vers deux variables différentes en mémoire.**

## Copie de types d'enregistrements à l'aide d'expressions

Avec les types d'enregistrements, ==l'affectation d'une instance de type d'enregistrement à une nouvelle variable crée un pointeur vers la même référence, ce qui correspond au comportement des classes==. Le code suivant illustre ce comportement :

```cs
static void CopyRecord()
{
    Console.WriteLine("*************** COPY ***************");
    
    CarRecord myCarRecord = new CarRecord("Honda", "Pilot", "Blue");
    CarRecord anotherMyCarRecord = new CarRecord("Honda", "Pilot", "Blue");
    
    CarRecord carRecordCopy = anotherMyCarRecord;
    Console.WriteLine("Car Record copy results");
    Console.WriteLine(
        $"CarRecords are the same? {carRecordCopy.Equals(anotherMyCarRecord)}"
    );
    Console.WriteLine(
        $"CarRecords are the same reference? {ReferenceEquals(carRecordCopy,
anotherMyCarRecord)}"
    );
}
```

==Lorsqu'ils sont exécutés, les deux tests renvoient la valeur « vrai », prouvant qu'ils sont identiques en valeur et en référence.==

**Pour créer une copie conforme** (*deepcopy*) **d'un enregistrement avec une ou plusieurs propriétés modifiées** (*modification non destructive*), **C# 9.0 introduit les expressions `with`**. ==Dans la construction `with`, les propriétés à mettre à jour sont spécifiées avec leurs nouvelles valeurs, et les propriétés non spécifiées sont copiées superficiellement==. Examinez l'exemple suivant :

```cs
static void CopyRecord()
{
    Console.WriteLine("*************** COPY ***************");
    
    CarRecord myCarRecord = new CarRecord("Honda", "Pilot", "Blue");
    CarRecord anotherMyCarRecord = new CarRecord("Honda", "Pilot", "Blue");
	
	...    

    CarRecord ourOtherCar = myCarRecord with { Model = "Odyssey" };
    Console.WriteLine("My copied car:");
    Console.WriteLine(ourOtherCar.ToString());
    Console.WriteLine("Car Record copy using with expression results");
    Console.WriteLine(
        $"CarRecords are the same? {ourOtherCar.Equals(myCarRecord)}"
    );
    Console.WriteLine(
        $"CarRecords are the same? {ReferenceEquals(ourOtherCar, myCarRecord)}"
    );
}
```

Le code ==crée une nouvelle instance du type `CarRecord`, en copiant les valeurs `Make` et `Color` de l'instance `myCarRecord` et en définissant `Model` sur la chaîne `Odyssey`==. Le résultat de ce code est affiché ici :

```
My copied car:
CarRecord { Make = Honda, Model = Odyssey, Color = Blue }
Car Record copy using with expression results
CarRecords are the same? False
CarRecords are the same? False
```


>[!important] L'expression `with` est disponible pour les **Records**, les **Structs** (depuis C#10) et les **Types Anonymes**, mais pas pour les classes.

**En utilisant l'expression `with`, vous pouvez facilement convertir des types d'enregistrements complexes en nouvelles instances de types d'enregistrements avec des valeurs de propriétés mises à jour**.

# Type de donnée `record struct` (Nouveauté C# 10.0)

**Nouveauté de C# 10.0, les *structures d'enregistrement* sont l'équivalent, pour les types valeur, des types enregistrement**. ==Elles peuvent également utiliser des paramètres positionnels ou la syntaxe standard des propriétés et offrent l'égalité des valeurs, la mutation non destructive et une mise en forme d'affichage intégrée==. Pour commencer à expérimenter avec les enregistrements, créez une nouvelle application console nommée *FunWithRecordStructs*. **Une différence majeure entre les structures d'enregistrement et les enregistrements est qu'une structure d'enregistrement est mutable par défaut**. ==Pour rendre une structure d'enregistrement immuable, vous devez ajouter le modificateur `readonly`.==

## Structures d'enregistrement mutables

Pour créer une structure d'enregistrement, reprenons l'exemple de la structure `Point`. L'exemple suivant montre comment créer deux types de structures d'enregistrement différents : l'un utilisant la syntaxe positionnelle et l'autre les propriétés standard.

```cs
namespace FunWithRecordStruct;

public record struct Point(double X, double Y, double Z);

// Déclare le constructeur par défaut comme constructeur primaire
public record struct PointWithPropertySyntax()
{
    public double X { get; set; } = default;
    public double Y { get; set; } = default;
    public double Z { get; set; } = default;

    // constructeur personnalisé qui appelle
    // le constructeur primaire, qui est le constructeur par defaut.
    public PointWithPropertySyntax(double x, double y, double z)
        : this()
    {
        X = x;
        Y = y;
        Z = z;
    }
}
```

>[!info]- Précision sur le constructeur primaire
>Les constructeur primaires ont été introduit pour les `record`. Soit avec C# 9.0. C# 12 à permis cette syntaxe pour les `class` et `struct` ordinaires.

Le code suivant illustre la mutabilité des deux types de structures d'enregistrement ainsi que la méthode améliorée `ToString()` :

```cs
using FunWithRecordStructs;

Console.Title = "Fun with Record Structs";
Console.WriteLine("**** Fun with Record Structs ****\n");

var rs = new Point(2, 4, 6);
Console.WriteLine(rs.ToString());
rs.X = 8;
Console.WriteLine(rs.ToString());

var rs2 = new PointWithPropertySyntax
{
    X = 2,
    Y = 4,
    Z = 6,
};
Console.WriteLine(rs2.ToString());
rs2.X = 8;
Console.WriteLine(rs2.ToString());

Console.ReadLine();
```

## Structures d'enregistrement immuables

Les deux exemples de structures d'enregistrement précédents peuvent être rendus immuables en ajoutant le mot-clé `readonly` :

```cs
// La même chose que Point, avec le mot clé readonly en plus.
public readonly record struct ReadOnlyPoint(double X, double Y, double Z);

// Différence notable avec PointWithPropertySyntax:
// Les propriété sont déclarées init-only.
public readonly record struct ReadOnlyPointWithPropertySyntax()
{
    public double X { get; init; } = default;
    public double Y { get; init; } = default;
    public double Z { get; init; } = default;

    public ReadOnlyPointWithPropertySyntax(double x, double y, double z)
        : this()
    {
        X = x;
        Y = y;
        Z = z;
    }
}
```

Vous pouvez confirmer qu'ils sont désormais immuables (imposé par le compilateur) avec le code suivant:

```cs
var rors = new ReadOnlyPoint(2, 4, 6);

// Erreur de compilation:
// rors.X = 8;

var rors2 = new ReadOnlyPointWithPropertySyntax
{
    X = 2,
    Y = 4,
    Z = 6,
};

// Erreur de compilation:
// rors2.X = 8;
```

## Déconstruction des structures d'enregistrement

**À l'instar des types d'enregistrement, les structures d'enregistrement utilisant la syntaxe positionnelle proposent également une méthode `Deconstruct()`**. ==Le comportement est identique à celui des structures d'enregistrement mutables et immuables==. Le code suivant crée un nouvel enregistrement à l'aide du constructeur fourni, puis décompose ses propriétés en variables distinctes :

```cs
Console.WriteLine("Deconstruction: ");

var (x1, y1, z1) = rs;
Console.WriteLine($"X: {x1}, Y: {y1}, Z: {z1}");

var (x2, y2, z2) = rors;
Console.WriteLine($"X: {x2}, Y: {y2}, Z: {z2}");

rs.Deconstruct(out double x3, out double y3, out double z3);
Console.WriteLine($"X: {x3}, Y: {y3}, Z: {z3}");

rors.Deconstruct(out double x4, out double y4, out double z4);
Console.WriteLine($"X: {x4}, Y: {y4}, Z: {z4}");
```

##### Tableau 5-2: Différences entre `record` et `record struct`
| Caractéristique        | `record` (ou `record class`)     | `record struct`                        |
| ---------------------- | -------------------------------- | -------------------------------------- |
| **Nature**             | Type Référence (Classe)          | Type Valeur (Structure)                |
| **Lieu de stockage**   | Le Tas (_Heap_)                  | La Pile (_Stack_)                      |
| **Par défaut**         | Immuable (`init`)                | Mutable (`set`)                        |
| **Héritage**           | Peut hériter d'un autre `record` | Impossible d'hériter (sauf interfaces) |
| **Pression sur le GC** | Oui (crée des objets à nettoyer) | Non (nettoyage instantané)             |

# Résumé du chapitre

**L'objectif de ce chapitre était de vous présenter le rôle du type classe C# et le nouveau type enregistrement de C# 9.0**. Comme vous l'avez vu, ==les classes peuvent avoir un nombre quelconque de *constructeurs* permettant à l'utilisateur de l'objet d'établir l'état de l'objet lors de sa création==. Ce chapitre a également illustré plusieurs techniques de conception de classes (et les mots-clés associés). ==Le mot-clé `this` permet d'accéder à l'objet courant==. Le mot-clé `static` permet de définir des champs et des membres liés au niveau de la classe* (et non de l'objet). ==Le mot-clé `const`, le modificateur `readonly` et les setters `init-only` permettent de définir une donnée qui ne peut plus être modifiée après l'affectation initiale ou la construction de l'objet==. **Les types `record` sont un type de classe particulier, immuable par défaut, qui se comporte comme un type valeur lors de la comparaison d'une instance du même type**. **Les structures d'enregistrement sont des types valeur mutables par défaut et offrent les mêmes fonctionnalités d'égalité et de modification non destructive que les types enregistrement**. 

**L'essentiel de ce chapitre a exploré en détail le premier pilier de la programmation orientée objet : l'encapsulation**. Vous avez découvert : ==les modificateurs d'accès en C# et le rôle des propriétés de type, la syntaxe d'initialisation des objets et les classes partielles==. Maintenant que vous maîtrisez ces notions, vous pouvez aborder le chapitre suivant, où vous apprendrez à construire une famille de classes apparentées en utilisant l'héritage et le polymorphisme.

---
title: "Chapitre 12: Délégués, Événemenmts et Expressions Lambda"
publish: true
---

# <big><big><big><b><font color =green>Délégués, Événements et Expressions Lambda</font></b></big></big></big>

Jusqu'à présent, la plupart des applications que vous avez développées ajoutaient divers fragments de code à *Program.cs* sous forme d'instructions de niveau supérieur, qui, d'une manière ou d'une autre, envoyaient des requêtes *à* un objet donné. Cependant, **de nombreuses applications exigent qu'un objet puisse communiquer avec l'entité qui l'a créé à l'aide d'un mécanisme de rappel**. Bien que les mécanismes de rappel puissent être utilisés dans n'importe quelle application, **ils sont particulièrement critiques pour les interfaces utilisateur graphiques, car les contrôles** (comme un bouton) **doivent appeler des méthodes externes dans les circonstances appropriées** (lorsqu'on clique sur le bouton, lorsque la souris entre dans la zone du bouton, etc.).

**Sous la plateforme .NET, le type** *délégué* **est le moyen privilégié de définir et de répondre aux rappels au sein des applications. Essentiellement, le type délégué .NET est un objet typé qui « pointe » vers une méthode ou une liste de méthodes pouvant être appelées ultérieurement**. Contrairement à un pointeur de fonction C++ traditionnel, **les délégués sont des classes qui prennent en charge nativement la multidiffusion.**

>[!note]
>Dans les versions précédentes de .NET, les délégués exposaient l'appel de méthode asynchrone avec `BeginInvoke()`/`EndInvoke()`. Bien que ces méthodes soient toujours générées par le compilateur, elles ne sont plus prises en charge par .NET. En effet, le modèle `IAsyncResult()`/`BeginInvoke()` utilisé par les délégués a été remplacé par le modèle asynchrone basé sur les tâches. Pour plus d'informations sur l'exécution asynchrone, consultez le [[Chapitre 15|Chapitre 15]].

Dans ce chapitre, **vous apprendrez à créer et à manipuler des types délégués, puis vous étudierez le mot-clé `event` de C#, qui simplifie l'utilisation des types délégués**. Vous découvrirez également plusieurs fonctionnalités du langage C# axées sur les délégués et les événements, notamment les ==méthodes anonymes et les conversions de groupes de méthodes.==

Ce chapitre se conclut par l'étude des *expressions lambda*. **Grâce à l'opérateur lambda C# (`=>`), vous pouvez spécifier un bloc d'instructions** (et les paramètres à leur transmettre) **partout où un délégué fortement typé est requis**. Comme vous le verrez, **une expression lambda est en réalité une méthode anonyme déguisée et offre une approche simplifiée de l'utilisation des délégués**. De plus, **cette même opération (comme dans .NET Framework 4.6 et versions ultérieures) permet d'implémenter une méthode ou une propriété en une seule instruction avec une syntaxe concise**.

## Comprendre le type délégué

Avant de définir formellement les délégués, prenons un peu de recul. ==Historiquement, l'API Windows utilisait fréquemment des pointeurs de fonction de style C pour créer des entités appelées *fonctions de rappel (Callback functions)*, ou simplement *rappels (Callback)*==. **Grâce aux rappels, les développeurs pouvaient configurer une fonction pour qu'elle renvoie un résultat à une autre fonction de l'application**. Cette approche permettait aux développeurs Windows de gérer les clics de boutons, les déplacements de la souris, la sélection de menus et les communications bidirectionnelles entre deux entités en mémoire.

Dans les frameworks .NET et .NET Core (et .NET 5+), les rappels sont implémentés de manière sûre et orientée objet grâce aux *délégués*. **Un délégué est un objet sûr qui pointe vers une autre méthode (ou éventuellement une liste de méthodes) de l'application, qui peut être appelée ultérieurement. Plus précisément, un délégué conserve trois informations importantes.**

- L'*adresse* de la méthode appelée
- Les *paramètres* (le cas échéant) de cette méthode
- Le *type de retour* (le cas échéant) de cette méthode

>[!note] Les délégués .NET peuvent pointer vers des méthodes statiques ou d'instance.

Une fois l'objet délégué créé et doté des informations nécessaires, il peut dynamiquement invoquer les méthodes qu'il désigne lors de l'exécution.

## Définition d'un type délégué en C#

>[!warning] Cette sous-section n'est plus la manière privilégié pour déclarer des délégués. (Gemini)
>**En C# moderne, on utilisera 95% du temps d'autre syntaxes qui seront abordé plus loin dans ce chapitre.**
>
Il reste néanmoins trois scénarios où déclarer son propre délégué est préférable :
>
>- **La clarté sémantique :** Si tu as une signature complexe, `delegate void CalculTransaction(int id, decimal montant, string devise, DateTime date)` est plus lisible que les alternatives utilisées actuellement.
>- **Les paramètres `ref`, `out` ou `params` :** Les types `Action` et `Func` ne supportent pas ces modificateurs. Si tu as besoin d'un paramètre `ref`, tu **dois** déclarer ton propre `delegate`.
>- **Les méthodes génériques avec contraintes :** Parfois, pour des APIs de très bas niveau ou des frameworks très spécifiques.

**Pour créer un type délégué en C#, utilisez le mot-clé `delegate`**. Le nom de votre type délégué est libre. Cependant, **==vous devez définir le délégué de manière à ce qu'il corresponde à la signature de la ou des méthodes auxquelles il fera référence==**. Par exemple, le type délégué suivant (nommé `BinaryOp`) peut faire référence à toute méthode renvoyant un entier et prenant deux entiers comme paramètres d'entrée (vous créerez et utiliserez vous-même ce délégué plus loin dans ce chapitre, alors patientez un peu pour le moment) :

```cs
// Ce délégué peut pointer vers n'importe quelle méthode,
// prenant deux entiers et renvoyant un entier.
public delegate int BinaryOp(int x, int y);
```

**Lorsque le compilateur C# traite les types délégués, il génère automatiquement une classe scellée dérivant de `System.MulticastDelegate`. Cette classe (associée à sa classe de base, `System.Delegate`) fournit l'infrastructure nécessaire au délégué pour conserver une liste de méthodes à appeler ultérieurement**. Par exemple, si vous examiniez le délégué `BinaryOp` à l'aide d'*ildsm.exe*, vous trouveriez certains détails comme indiqué ici (vous pouvez compiler cet exemple complet dans un instant si vous souhaitez le vérifier vous-même) :

```CIL
// -------------------------------------------------------
// 	TypDefName: BinaryOp  (02000003)
// 	Flags     : [Public] [AutoLayout] [Class] [Sealed] [AnsiClass]  (00000101)
// 	Extends   : 01000010 [TypeRef] System.MulticastDelegate
// 	Method #1 (06000003) 
// 	-------------------------------------------------------
// 		MethodName: .ctor (06000003)
// 		Flags     : [Public] [HideBySig] [ReuseSlot] [SpecialName] [RTSpecialName] [.ctor]  (00001886)
// 		RVA       : 0x00000000
// 		ImplFlags : [Runtime] [Managed]  (00000003)
// 		CallCnvntn: [DEFAULT]
// 		hasThis 
// 		ReturnType: Void
// 		2 Arguments
// 			Argument #1:  Object
// 			Argument #2:  I
// 		2 Parameters
// 			(1) ParamToken : (08000002) Name : object flags: [none] (00000000)
// 			(2) ParamToken : (08000003) Name : method flags: [none] (00000000)
// 
// 	Method #2 (06000004) 
// 	-------------------------------------------------------
// 		MethodName: Invoke (06000004)
// 		Flags     : [Public] [Virtual] [HideBySig] [NewSlot]  (000001c6)
// 		RVA       : 0x00000000
// 		ImplFlags : [Runtime] [Managed]  (00000003)
// 		CallCnvntn: [DEFAULT]
// 		hasThis 
// 		ReturnType: I4
// 		2 Arguments
// 			Argument #1:  I4
// 			Argument #2:  I4
// 		2 Parameters
// 			(1) ParamToken : (08000004) Name : x flags: [none] (00000000)
// 			(2) ParamToken : (08000005) Name : y flags: [none] (00000000)
// 
// 	Method #3 (06000005) 
// 	-------------------------------------------------------
// 		MethodName: BeginInvoke (06000005)
// 		Flags     : [Public] [Virtual] [HideBySig] [NewSlot]  (000001c6)
// 		RVA       : 0x00000000
// 		ImplFlags : [Runtime] [Managed]  (00000003)
// 		CallCnvntn: [DEFAULT]
// 		hasThis 
// 		ReturnType: Class System.IAsyncResult
// 		4 Arguments
// 			Argument #1:  I4
// 			Argument #2:  I4
// 			Argument #3:  Class System.AsyncCallback
// 			Argument #4:  Object
// 		4 Parameters
// 			(1) ParamToken : (08000006) Name : x flags: [none] (00000000)
// 			(2) ParamToken : (08000007) Name : y flags: [none] (00000000)
// 			(3) ParamToken : (08000008) Name : callback flags: [none] (00000000)
// 			(4) ParamToken : (08000009) Name : object flags: [none] (00000000)
// 
// 	Method #4 (06000006) 
// 	-------------------------------------------------------
// 		MethodName: EndInvoke (06000006)
// 		Flags     : [Public] [Virtual] [HideBySig] [NewSlot]  (000001c6)
// 		RVA       : 0x00000000
// 		ImplFlags : [Runtime] [Managed]  (00000003)
// 		CallCnvntn: [DEFAULT]
// 		hasThis 
// 		ReturnType: I4
// 		1 Arguments
// 			Argument #1:  Class System.IAsyncResult
// 		1 Parameters
// 			(1) ParamToken : (0800000a) Name : result flags: [none] (00000000)
// 
```

Comme vous pouvez le constater, **la classe `BinaryOp` générée par le compilateur définit trois méthodes publiques. `Invoke()` est la méthode clé en .NET, car elle permet d'appeler chaque méthode gérée par l'objet délégué de manière synchrone**. Cela signifie que ==l'appelant doit attendre la fin de l'appel avant de poursuivre son exécution==. Curieusement, **il n'est pas toujours nécessaire d'appeler explicitement la méthode synchrone `Invoke()` depuis votre code C#**. Comme vous le verrez plus loin, **`Invoke()` est appelée en arrière-plan lorsque vous utilisez la syntaxe C# appropriée.**

>[!note]
>Bien que les méthodes `BeginInvoke()` et `EndInvoke()` soient générées, elles ne sont pas prises en charge lors de l'exécution de votre code sous .NET Core ou .NET 5+. Cela peut être frustrant, car vous n'obtiendrez pas d'erreur de compilation, mais une erreur d'exécution si vous les utilisez.

Comment le compilateur détermine-t-il précisément la méthode `Invoke()` ? Pour comprendre ce processus, voici l’essentiel du type de classe `BinaryOp` généré par le compilateur (les éléments spécifiés par le type délégué défini sont indiqués en gras italique) :

```cs
sealed class BinaryOp : System.MulticastDelegate
{
	public int Invoke(int x, int y);
	
	...
}
```

Tout d'abord, notez que les paramètres et le type de retour définis pour la méthode `Invoke()` correspondent exactement à la définition du délégué `BinaryOp`.

Prenons un autre exemple. Supposons que vous ayez défini un type délégué pouvant pointer vers n'importe quelle méthode renvoyant un `string` et recevant trois paramètres d'entrée de type `System.Boolean`.

```cs
public delegate string MyDelegate (bool a, bool b, bool c);
```

Cette fois-ci, la classe générée par le compilateur se décompose comme suit :

```cs
sealed class MyDelegate : System.MulticastDelegate
{
	public string Invoke(bool a, bool b, bool c);

	...
}
```

**Les délégués peuvent également « pointer vers » des méthodes comportant un nombre quelconque de paramètres `out` ou `ref`** (ainsi que des paramètres de type tableau marqués avec le mot-clé `params`). Par exemple, considérons le type de délégué suivant :

```cs
public delegate string MyOtherDelegate(
	out bool a, ref bool b, int c);
```

La signature de la méthode `Invoke()` est conforme aux attentes.

En résumé, **la définition d'un type délégué en C# aboutit à une classe scellée avec une méthode générée par le compilateur dont les types des paramètres et de retour sont basés sur la déclaration du délégué**. Le pseudocode suivant illustre approximativement le modèle de base :

```cs
// C'est seulement du pseudo-code!
public sealed class DelegateName : System.MulticastDelegate
{
	public delegateReturnValue Invoke(allDelegateInputRefAndOutParams);
}
```

## Les classes de base `System.MulticastDelegate` et `System.Delegate`

Ainsi, **lorsque vous créez un type à l'aide du mot-clé `delegate` en C#, vous déclarez indirectement un type de classe qui dérive de `System.MulticastDelegate`**. ==Cette classe permet aux classes descendantes d'accéder à une liste contenant les adresses des méthodes gérées par l'objet délégué, ainsi que plusieurs méthodes supplémentaires== (et quelques opérateurs surchargés) pour interagir avec la liste d'invocation. Voici quelques membres de `System.MulticastDelegate` :

```cs
public abstract class MulticastDelegate : Delegate
{
	// Renvoie la liste des méthodes « pointées ».
	public sealed override Delegate[] GetInvocationList();
	
	// Opérateurs surchargés.
	public static bool operator ==
	(MulticastDelegate d1, MulticastDelegate d2);
	public static bool operator !=
	(MulticastDelegate d1, MulticastDelegate d2);
	
	// Utilisé en interne pour gérer la liste des méthodes conservées par le délégué.
	private IntPtr _invocationCount;
	private object _invocationList;
}
```

`System.MulticastDelegate` hérite de fonctionnalités supplémentaires de sa classe parente, `System.Delegate`. Voici un aperçu partiel de la définition de la classe :

```cs
public abstract class Delegate : ICloneable, ISerializable
{
	// Méthodes pour interagir avec la liste des fonctions.
	public static Delegate Combine(params Delegate[] delegates);
	public static Delegate Combine(Delegate a, Delegate b);
	public static Delegate Remove(
	  Delegate source, Delegate value);
	public static Delegate RemoveAll(
	  Delegate source, Delegate value);
	
	// Opérateurs surchargés.
	public static bool operator ==(Delegate d1, Delegate d2);
	public static bool operator !=(Delegate d1, Delegate d2);
	
	// Propriétés qui exposent la cible du délégué.
	public MethodInfo Method { get; }
	public object Target { get; }
}
```

**Il est important de comprendre que vous ne pouvez jamais hériter directement de ces classes de base dans votre code** (cela provoquerait une erreur de compilation). Cependant, ==lorsque vous utilisez le mot-clé `delegate`, vous créez indirectement une classe qui est un `MulticastDelegate`==. Le [[#Tableau 12-1 Sélection de membres de `System.MulticastDelegate`/`System.Delegate`|Tableau 12-1]] répertorie les membres principaux communs à tous les types de délégués.

##### Tableau 12-1: Sélection de membres de `System.MulticastDelegate`/`System.Delegate`

| Membre                     | Description                                                                                                                                                                                                                                                   |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Method`                   | Cette propriété renvoie un objet `System.Reflection.MethodInfo` qui représente les détails d'une méthode statique gérée par le délégué.                                                                                                                       |
| `Target`                   | Si la méthode à appeler est définie au niveau de l'objet (plutôt qu'une méthode statique), `Target` renvoie un objet représentant la méthode gérée par le délégué. Si la valeur renvoyée par `Target` est nulle, la méthode à appeler est un membre statique. |
| `Combine()`                | Cette méthode statique ajoute une méthode à la liste gérée par le délégué. En C#, vous appelez cette méthode à l'aide de l'opérateur surchargé `+=` comme notation abrégée.                                                                                   |
| `GetInvocationList()`      | Cette méthode renvoie un tableau d'objets `System.Delegate`, chacun représentant une méthode pouvant être invoquée.                                                                                                                                           |
| `Remove()` / `RemoveAll()` | Ces méthodes statiques suppriment une méthode (ou toutes les méthodes) de la liste d'appel du délégué. En C#, la méthode Remove() peut être appelée indirectement à l'aide de l'opérateur surchargé `-=`.                                                     |

# L’exemple le plus simple possible de délégué

Il est vrai que les délégués peuvent prêter à confusion lors d’une première utilisation. Pour commencer, prenons l’exemple d’une application console simple (nommée *SimpleDelegate*) qui utilise le type de délégué `BinaryOp` que vous avez déjà vu. Voici le code complet, suivi de son analyse :

```cs
// SimpleMath.cs
namespace SimpleDelegate;

// Cette classe contient les méthodes auxquelles
// BinaryOp va pointer.
public class SimpleMath
{
    public static int Add(int x, int y) => x + y;

    public static int Subtract(int x, int y) => x - y;
}
```

```cs
// Program.cs
using SimpleDelegate;

Console.Title = "Simple Delegate Example";
Console.WriteLine("***** Simple Delegate Example *****\n");

// Créer un objet délégué BinaryOp qui
// « pointe vers » SimpleMath.Add().
BinaryOp b = new BinaryOp(SimpleMath.Add);

// Appel indirect de la méthode Add() via l'objet délégué.
Console.WriteLine($"10 + 10 is {b(10, 10)}");
Console.ReadLine();

// Les définitions de type supplémentaires doivent
// être placées à la fin des instructions de niveau supérieur.
// Ce délégué peut pointer vers n'importe quelle méthode,
// prenant deux entiers et renvoyant un entier.
public delegate int BinaryOp(int x, int y);
```

>[!note] 
>Souvenez vous du [[Chapitre 3#Utiliser les déclaration de haut niveaux (Nouveauté C 9.0)|Chapitre 3]] que les déclarations de type additionnels (dans cet exemple le délégué `BinaryOp`) doit venir *après toutes* les déclarations de haut niveau.

Notez à nouveau le format de la déclaration du type délégué `BinaryOp`; ==il spécifie que les objets délégués `BinaryOp` peuvent pointer ***vers n’importe quelle méthode*** prenant deux entiers et renvoyant un entier== (le nom de la méthode pointée est sans importance). Ici, vous avez créé une classe nommée `SimpleMath`, qui définit deux méthodes statiques correspondant au modèle défini par le délégué `BinaryOp`.

**Pour assigner la méthode cible à un objet délégué donné, il suffit de passer le nom de la méthode au constructeur du délégué.**

```cs
// Créer un objet délégué BinaryOp qui
// « pointe vers » SimpleMath.Add().
BinaryOp b = new BinaryOp(SimpleMath.Add);
```

À ce stade, vous pouvez invoquer le membre pointé en utilisant une syntaxe qui ressemble à un appel de fonction direct.

```cs
// La méthode Invoke() est bien appelée ici !
Console.WriteLine($"10 + 10 is {b(10, 10)}");
```

**En interne, le runtime appelle la méthode `Invoke()` générée par le compilateur sur votre
classe dérivée de `MulticastDelegate`**. Vous pouvez le vérifier vous-même en ouvrant votre assembly dans *ildasm.exe* et en examinant le code CIL de la méthode `Main()`.

```CIL
.method private hidebysig static void  '<Main>$'(string[] args) cil managed
{
	...
	
	IL_0041:  callvirt   instance int32 BinaryOp::Invoke(int32, int32)

	...
} // end of method Program::'<Main>$'

...
```

**En C#, il n'est pas nécessaire d'appeler explicitement la méthode `Invoke()` dans votre code**. Comme `BinaryOp` peut pointer vers des méthodes prenant deux arguments, ==l'instruction suivante est également autorisée :==

```cs
Console.WriteLine("10 + 10 is {0}", b.Invoke(10, 10));
```

Rappelons que les délégués .NET sont *fortement typés*. Par conséquent, **si vous tentez de créer un objet délégué pointant vers une méthode qui ne correspond pas au modèle, vous obtiendrez une erreur de compilation**. À titre d'exemple, supposons que la classe `SimpleMath` définisse désormais une méthode supplémentaire nommée `SquareNumber()`, qui prend un entier unique en entrée.

```cs
public class SimpleMath
{
	...
    public static int SquareNumber(int a) => a * a;
}
```

Étant donné que le délégué `BinaryOp` ne peut pointer que vers des méthodes prenant deux entiers et renvoyant un entier, le code suivant est invalide et ne compilera pas :

```cs
// Erreur de compilation ! La méthode ne correspond pas au modèle de délégation !
BinaryOp b2 = new BinaryOp(SimpleMath.SquareNumber);
```

## Exploration d'un objet délégué

Ajoutons une méthode statique (nommée `DisplayDelegateInfo()`) au fichier *Program.cs* pour enrichir l'exemple actuel. Cette méthode affichera les noms des méthodes gérées par un objet délégué, ainsi que le nom de la classe définissant la méthode. Pour ce faire, vous parcourrez le tableau `System.Delegate` renvoyé par `GetInvocationList()`, en appelant les propriétés `Target` et `Method` de chaque objet.

```cs
static void DisplayDelegateInfo(Delegate delObj)
{
    // Affiche le nom de chaque membre
    // dans la list d'invoquation du délégué
    foreach (Delegate d in delObj.GetInvocationList())
    {
        Console.WriteLine($"Method name: {d.Method}");
        Console.WriteLine($"Type name: {d.Target}");
    }
}
```

En supposant que vous ayez mis à jour votre méthode `Main()` pour appeler cette nouvelle méthode auxiliaire, comme indiqué ici :

```cs
BinaryOp b = new BinaryOp(SimpleMath.Add);
DisplayDelegateInfo(b);
```

Vous trouverez le résultat suivant :

```
***** Simple Delegate Example *****

Method name: Int32 Add(Int32, Int32)
Type name:
10 + 10 is 20
```

==Notez que le nom de la classe cible (`SimpleMath`) n'est actuellement *pas* affiché lors de l'appel de la propriété `Target`==. Cela est dû au fait que **votre délégué `BinaryOp` pointe vers une méthode statique et, par conséquent, aucun objet n'est disponible pour y faire référence !** Toutefois, si vous modifiez les méthodes `Add()` et `Subtract()` pour qu'elles soient non statiques (en supprimant simplement les mots-clés `static`), vous pourrez créer une instance de la classe `SimpleMath` et spécifier les méthodes à appeler à l'aide de la référence d'objet.

```cs
using SimpleDelegate;

Console.Title = "Simple Delegate Example";
Console.WriteLine("***** Simple Delegate Example *****\n");

// Les délégués peuvent aussi pointer vers des méthodes d'instances.
SimpleMath m = new SimpleMath();
BinaryOp b = new BinaryOp(m.Add);
DisplayDelegateInfo(b);

// Appel indirect de la méthode Add() via l'objet délégué.
// La méthode Invoke() est bien appelée ici!
Console.WriteLine($"10 + 10 is {b(10, 10)}");
Console.ReadLine();
...
```

Dans ce cas, vous trouverez le résultat affiché ici :

```
***** Simple Delegate Example *****

Method name: Int32 Add(Int32, Int32)
Type name: SimpleDelegate.SimpleMath
10 + 10 is 20
```

# Envoi de notifications d'état d'objet à l'aide de délégués

==De toute évidence, l'exemple précédent de `SimpleDelegate` était purement illustratif, car il n'y aurait aucune raison valable de définir un délégué simplement pour additionner deux nombres==. Pour une utilisation plus réaliste des types délégués, **utilisons-les pour définir une classe `Car` capable d'informer des entités externes de l'état actuel de son moteur.** Pour ce faire, suivez les étapes suivantes :

1. Définissez un nouveau type de délégué qui servira à envoyer des notifications à l'appelant.
2. Déclarez une variable membre de ce délégué dans la classe `Car`.
3. Créez une fonction d'assistance dans la classe `Car` permettant à l'appelant de spécifier la méthode à appeler.
4. Implémentez la méthode `Accelerate()` pour invoquer la liste d'appels du délégué dans les circonstances appropriées.

Pour commencer, créez un nouveau projet d'application console nommé *CarDelegate*. Ensuite, définissez une nouvelle classe `Car` qui ressemble initialement à ceci :

```cs
namespace CarDelegate;

public class Car
{
    // Donnée d'état interne
    public int CurrentSpeed { get; set; }
    public int MaxSpeed { get; set; } = 100;
    public string PetName { get; set; }

    // Est-ce que la voiture est morte ou vivante ?
    private bool _carIsDead;

    // Constructeurs de classe
    public Car() { }

    public Car(string name, int maxSp, int currSp)
    {
        CurrentSpeed = currSp;
        MaxSpeed = maxSp;
        PetName = name;
    }
}
```

Considérons maintenant les mises à jour suivantes, qui traitent des trois premiers points :

```cs
public class Car
{
	...
	
    // 1) Définit un type délegué
    public delegate void CarEngineHandler(string msgForCaller);

    // 2) Définit une variable de membre pour ce délégué.
    private CarEngineHandler _listOfHandlers;

    // Ajouter une fonction d'enregistrement pour l'appelant.
    public void RegisterWithCarEngine(CarEngineHandler methodTocall)
    {
        _listOfHandlers = methodTocall;
    }
}
```

>[!warning] Comme expliqué [[#Définition d'un type délégué en C|ici]], ce n'est plus la façon de faire pour déclarer de délégués de nos jours

Dans cet exemple, notez que **vous définissez les types délégués directement dans la portée de la classe `Car`, ce qui n'est certes pas nécessaire, mais renforce l'idée que le délégué fonctionne naturellement avec cette classe**. ==Le type délégué, `CarEngineHandler`, peut pointer vers n'importe quelle méthode prenant une chaîne de caractères en entrée et une valeur de retour void.==

Ensuite, notez que **vous déclarez une variable membre privée de votre type délégué (nommée `_listOfHandlers`) et une fonction auxiliaire (nommée `RegisterWithCarEngine()`) qui permet à l'appelant d'assigner une méthode à la liste d'invocation du délégué**.

>[!note]
>À proprement parler, vous auriez pu définir votre variable membre déléguée comme publique, évitant ainsi la nécessité de créer des méthodes d'enregistrement supplémentaires. Toutefois, en définissant la variable membre déléguée comme privée, vous imposez l'encapsulation des services et offrez une solution plus sûre au niveau des types. Vous reviendrez sur le risque que représentent les variables membres déléguées publiques plus loin dans ce chapitre, lors de l'étude du mot-clé `event` en C#.

À ce stade, vous devez créer la méthode `Accelerate()`. **Rappelons que l'objectif est de permettre à un objet `Car` d'envoyer des messages relatifs au moteur à tout écouteur abonné**. Voici la mise à jour :

```cs
// 4) Implémentez la méthode Accelerate() pour appeler la liste
// d'invocation du délégué dans les circonstances appropriées.
public void Accelerate(int delta)
{
	// Si cette voiture est "morte", on envoit un message de mort.
	if (_carIsDead)
	{
		_listOfHandlers?.Invoke("Sorry, this car is dead...");
	}
	else
	{
		CurrentSpeed += delta;
		// Cette voiture est-elle « presque morte » ?
		if (10 == MaxSpeed - CurrentSpeed)
		{
			_listOfHandlers?.Invoke("Careful buddy! Gonna blow!");
		}

		if (CurrentSpeed >= MaxSpeed)
		{
			_carIsDead = true;
		}
		else
		{
			Console.WriteLine($"currentSpeed = {CurrentSpeed}");
		}
	}
}
```

Notez que **vous utilisez la syntaxe de propagation de la valeur nulle lorsque vous tentez d'appeler les méthodes gérées par la variable membre `listOfHandlers`**. En effet, ==il incombera à l'appelant d'allouer ces objets en appelant la méthode auxiliaire `RegisterWithCarEngine()`==. **Si l'appelant n'appelle pas cette méthode et que vous tentez d'appeler la liste d'appel du délégué, une exception `NullReferenceException` sera levée à l'exécution**. L'infrastructure du délégué étant maintenant en place, observez les mises à jour du fichier *Program.cs,* affichées ici :

```cs
using CarDelegate;

Console.Title = "Delegates as Event Enablers";
Console.WriteLine("***** Delegates as Event Enablers *****\n");

// D'abord, créer un objet Car
Car c1 = new Car("SlugBug", 100, 10);

// Maintenant, Indique à la voiture quelle méthode utiliser
// lorsqu'elle souhaite nous envoyer des messages.
c1.RegisterWithCarEngine(new Car.CarEngineHandler(OnCarEngineEvent));

// Accélère (cela va déclencher l'événement).
Console.WriteLine("**** Speeding up ****");
for (int i = 0; i < 6; i++)
{
    c1.Accelerate(20);
}
Console.ReadLine();

// Ceci est la cible pour les événements à venir
static void OnCarEngineEvent(string msg)
{
    Console.WriteLine("\n*** Message From Car Object ***");
    Console.WriteLine($"=> {msg}");
    Console.WriteLine("********************\n");
}
```

Le code commence par la création d'un nouvel objet `Car`. ==Puisque vous souhaitez être informé des événements du moteur, l'étape suivante consiste à appeler votre fonction d'enregistrement personnalisée, `RegisterWithCarEngine()`==. Rappelons que **cette méthode attend une instance du délégué imbriqué `CarEngineHandler`, et comme pour tout délégué, vous spécifiez une « méthode cible », aussi appelé *abonné*, en tant que paramètre du constructeur**. L'astuce dans cet exemple est que ==la méthode en question se trouve dans le fichier *Program.cs*== ! Notez encore une fois que **la méthode `OnCarEngineEvent()` correspond parfaitement au délégué associé, car elle prend un `string` en entrée et ne renvoie rien**. Considérez la sortie de l'exemple actuel :

```
***** Delegates as Event Enablers *****

**** Speeding up ****
currentSpeed = 30
currentSpeed = 50
currentSpeed = 70

*** Message From Car Object ***
=> Careful buddy! Gonna blow!
********************

currentSpeed = 90

*** Message From Car Object ***
=> Sorry, this car is dead...
********************
```

## Activation du multicast

appelons que ==les délégués .NET possèdent la capacité intégrée de *multicast*==. Autrement dit, **un objet délégué peut gérer une liste de méthodes à appeler, et non une seule**. Pour ajouter plusieurs méthodes à un objet délégué, **il suffit d'utiliser l'opérateur surchargé `+=`, plutôt qu'une affectation directe**. Pour activer le multicast sur la classe `Car`, vous pouvez modifier la méthode `RegisterWithCarEngine()` comme suit :

```cs
public class Car
{
	...
	
    // Prise en charge du multicast désormais disponible !
    // Notez que nous utilisons maintenant l'opérateur += et non
    // l'opérateur d'affectation (=).
    public void RegisterWithCarEngine(CarEngineHandler onEngineEvent)
    {
        _listOfHandlers += onEngineEvent;
    }

	...
}
```

**Lorsque vous utilisez l'opérateur `+=` sur un objet délégué, le compilateur le traduit en un appel à la méthode statique `Delegate.Combine()`**. ==En réalité, vous pourriez appeler `Delegate.Combine()` directement; cependant, l'opérateur `+=` offre une alternative plus simple==. Il n'est pas nécessaire de modifier votre méthode `RegisterWithCarEngine()` actuelle, mais voici un exemple d'utilisation de `Delegate.Combine()` plutôt que de l'opérateur `+=` :

```cs
public void RegisterWithCarEngine( CarEngineHandler methodToCall )
{
	if (_listOfHandlers == null)
	{
		_listOfHandlers = methodToCall;
	}
	else
	{
		_listOfHandlers =
		  Delegate.Combine(_listOfHandlers, methodToCall)
		    as CarEngineHandler;
	}
}
```

**Dans tous les cas, l'appelant peut désormais enregistrer plusieurs cibles pour la même notification de rappel**. Ici, le second gestionnaire affiche le message entrant en majuscules, à des fins d'affichage uniquement:

```cs
using CarDelegate;

Console.Title = "Delegates as Event Enablers";
Console.WriteLine("***** Delegates as Event Enablers *****\n");

// D'abord, créer un objet Car
Car c1 = new Car("SlugBug", 100, 10);

// Maintenant, Indique à la voiture quelle méthode utiliser
// lorsqu'elle souhaite nous envoyer des messages.
// Enregistre plusieurs plusieurs abonné pour la notification.
c1.RegisterWithCarEngine(new Car.CarEngineHandler(OnCarEngineEvent));
c1.RegisterWithCarEngine(new Car.CarEngineHandler(OnCarEngineEvent2));

// Accélère (cela va déclencher l'événement).
Console.WriteLine("**** Speeding up ****)");
for (int i = 0; i < 6; i++)
{
    c1.Accelerate(20);
}
Console.ReadLine();

// On a maintenant deux méthodes qui seront appellée par l'objet Car
// quand on enverra une notification
static void OnCarEngineEvent(string msg)
{
    Console.WriteLine("\n*** Message From Car Object ***");
    Console.WriteLine($"=> {msg}");
    Console.WriteLine("********************\n");
}

static void OnCarEngineEvent2(string msg)
{
    Console.WriteLine($"=> {msg.ToUpper()}");
}
```

## Suppression de cibles de la liste d'invocation d'un délégué

**La classe `Delegate` définit également une méthode statique `Remove()` qui permet à l'appelant de supprimer dynamiquement une méthode de la liste d'invocation d'un objet délégué**. Cela simplifie la *désinscription* de l'appelant d'une notification donnée lors de l'exécution. **Bien qu'il soit possible d'appeler directement `Delegate.Remove()` dans le code, les développeurs C# peuvent utiliser l'opérateur `-=` comme notation abrégée pratique**. Ajoutons une nouvelle méthode à la classe `Car` qui permet à l'appelant de supprimer une méthode de la liste d'invocation.

```cs
public class Car
{
	...
    // methodToCall à été changé par onEngineEvent pour se rapprocher du
    // monde professionnel, tout en gardant une description pédagogique
    // (comparé à "handler" / "action" / "callback")
    public void UnregisterWithCarEngine(CarEngineHandler onEngineEvent)
    {
        _listOfHandlers -= onEngineEvent;
    }
}
```

Avec les mises à jour actuelles de la classe `Car`, vous pouvez désactiver la réception de la notification moteur sur le deuxième gestionnaire en mettant à jour le code appelant comme suit :

```cs
using CarDelegate;

Console.Title = "Delegates as Event Enablers";
Console.WriteLine("***** Delegates as Event Enablers *****\n");

// D'abord, créer un objet Car
Car c1 = new Car("SlugBug", 100, 10);

// Maintenant, Indique à la voiture quelle méthode utiliser
// lorsqu'elle souhaite nous envoyer des messages.
// Enregistre plusieurs plusieurs abonné pour la notification.
c1.RegisterWithCarEngine(new Car.CarEngineHandler(OnCarEngineEvent));

// Cette fois, conservez l'objet délégué,
// afin de pouvoir le désinscrire ultérieurement.
Car.CarEngineHandler handler2 = new Car.CarEngineHandler(OnCarEngineEvent2);
c1.RegisterWithCarEngine(handler2);

// Accélère (cela va déclencher l'événement).
Console.WriteLine("**** Speeding up ****");
for (int i = 0; i < 6; i++)
{
    c1.Accelerate(20);
}

// Désabonnement du deuxième gestionnaire
c1.UnregisterWithCarEngine(handler2);

// On ne verra plus le message en Majuscule!
Console.WriteLine("**** Speeding up ****");
for (int i = 0; i < 6; i++)
{
    c1.Accelerate(20);
}

Console.ReadLine();
```

Une différence dans ce code est que cette fois-ci, **vous créez un objet `Car.CarEngineHandler` et le stockez dans une variable locale afin de pouvoir l'utiliser ultérieurement pour vous désinscrire de la notification**. Ainsi, la deuxième fois que vous accélérez l'objet `Car`, vous ne voyez plus la version en majuscules des données du message entrant, car vous avez supprimé cette cible de la liste d'invocation du délégué.

## Syntaxe de conversion du groupe de méthodes

Dans l'exemple `CarDelegate` précédent, vous avez explicitement créé des instances de l'objet délégué `Car.CarEngineHandler` pour vous enregistrer et vous désinscrire des notifications du moteur.

```cs
Console.Title = "Delegates as Event Enablers";
Console.WriteLine("***** Delegates as Event Enablers *****\n");

Car c1 = new Car("SlugBug", 100, 10);
c1.RegisterWithCarEngine(new Car.CarEngineHandler(OnCarEngineEvent));

Car.CarEngineHandler handler2 = new Car.CarEngineHandler(OnCarEngineEvent2);
c1.RegisterWithCarEngine(handler2);
```

Certes, si vous devez appeler un membre hérité de `MulticastDelegate` ou de `Delegate`, créer manuellement une variable déléguée est la méthode la plus simple. Cependant, **dans la plupart des cas, il n'est pas nécessaire de conserver l'objet délégué**. En général, ==vous n'avez besoin de l'utiliser que pour passer le nom de la méthode comme paramètre du constructeur==.

==Pour simplifier, C# propose un raccourci appelé *conversion de groupe de méthodes*==. **Cette fonctionnalité vous permet de fournir directement le nom de la méthode, plutôt qu'un objet délégué, lors de l'appel de méthodes prenant des délégués comme arguments**.

>[!note]
>Comme vous le verrez plus loin dans ce chapitre, vous pouvez également utiliser la syntaxe de conversion de groupe de méthodes pour simplifier la manière dont vous vous abonné à un événement C#.

À titre d'exemple, considérons les mises à jour suivantes du fichier *Program.cs*, qui utilise le groupe de méthodes conversion pour s'inscrire et se désinscrire des notifications du moteur :


```cs

...

Console.Title = "Method Group Conversion";
Console.WriteLine("***** Method Group Conversion *****\n");

Car c2 = new Car();

// Abonnement avec simplement le nom de la méthode.
c2.RegisterWithCarEngine(OnCarEngineEvent);

Console.WriteLine("**** Speeding up ****");
for (int i = 0; i < 6; i++)
{
    c2.Accelerate(20);
}

// Désabonnemnt avec simplement le nom de la méthode.
c2.UnregisterWithCarEngine(OnCarEngineEvent);

// Plus aucune notification!
for (int i = 0; i < 6; i++)
{
    c2.Accelerate(20);
}

Console.ReadLine();
```

**Notez que vous n'allouez pas directement l'objet délégué associé, mais que vous spécifiez simplement une méthode correspondant à la signature attendue du délégué** (une méthode ne renvoyant aucune valeur et prenant une seule chaîne de caractères, dans ce cas). ==Sachez que le compilateur C# vérifie toujours la sécurité des types==. Par conséquent, si la méthode `OnCarEngineEvent()` ne prenait pas une chaîne de caractères et ne renvoyait pas aucune valeur, une erreur de compilation serait générée.

### Method Group Conversion vs Instances de Délégués (Gemini)

Le passage d'une méthode à un délégué cache une étape de création d'objet que le compilateur gère pour vous.

#### La "Method Group Conversion" (Magie du Compilateur)

Lorsque vous écrivez `c1.Register(OnCarEngineEvent)`, vous ne passez pas réellement "une méthode" en paramètre. **En C#, une méthode n'est pas un objet.**

- Le compilateur détecte que `Register` attend un type `CarEngineHandler`.
- Il génère automatiquement le code `new CarEngineHandler(OnCarEngineEvent)`.
- **Résultat :** C'est un raccourci syntaxique (sucre syntaxique) qui rend le code plus propre sans changer la mécanique de base.

#### Création d'objets et Mémoire

Chaque fois que vous utilisez le nom d'une méthode pour un abonnement ou un désabonnement **sans utiliser de variable**, vous créez une **nouvelle instance** de délégué.

1. **Abonnement :** `c1.Register(OnCarEngineEvent)`
    - _Mémoire :_ Création de l'**Instance A** du délégué. Elle est ajoutée à la liste.
2. **Désabonnement :** `c1.Unregister(OnCarEngineEvent)`
    - _Mémoire :_ Création de l'**Instance B** du délégué.
    - _Action :_ **Le CLR compare A et B. Comme ils pointent vers la même méthode, il supprime A de la liste et jette B à la poubelle (Garbage Collector).**

#### Pourquoi l' approche `handler2` est différente

En stockant le délégué dans une variable (`Car.CarEngineHandler handler2 = ...`), vous contrôlez précisément l'instance :

- Vous créez **une seule instance** en mémoire.
- Vous passez exactement la **même référence** au `+=` et au `-=`.
- C'est la méthode la plus performante (pas d'allocation d'objet temporaire lors du désabonnement) et la plus sûre si vous travaillez avec des méthodes complexes ou des lambdas.

#### Ce qui a évolué autour (de .NET 7 à .NET 10)

Bien que la mécanique soit la même, la manière dont on écrit le code a continué de s'affiner depuis .NET 6 :

##### **.NET 7 : Optimisation des "Method Groups"**

Avant .NET 7, faire `c1.Register(OnCarEngineEvent)` créait un nouvel objet délégué **à chaque fois** que le code passait sur cette ligne.

- **Ajustement :** Le compilateur a commencé à "mettre en cache" (cacher) l'instance du délégué si la méthode est statique.
- **Impact :** Moins de travail pour le Garbage Collector. Votre code devient plus performant sans que vous ayez à le modifier.

##### **.NET 8 : Function Pointers & Native AOT**

.NET 8 a mis l'accent sur la compilation native (AOT - Ahead Of Time).

- **Ajustement :** Les délégués ont été optimisés pour être plus "légers" lors de la compilation en code machine direct.
- **Impact :** Le démarrage des applications est plus rapide, et l'invocation des événements coûte moins de cycles CPU.

##### **.NET 9 : Inline Arrays et Amélioration du Multicast**

.NET 9 a peaufiné la manière dont les listes d'invocation (quand vous avez plusieurs abonnés avec `+=`) sont parcourues.

- **Ajustement :** Amélioration de la localité des données dans la mémoire cache.
- **Impact :** Si vous avez 50 abonnés sur une même voiture, l'appel de l'événement est beaucoup plus fluide.

##### **.NET 10 Intelligence de Désabonnement (Static Analysis) et Zero-Allocation**

- **Ajustement :** L'analyseur de code (Roslyn) est devenu beaucoup plus agressif. Il peut maintenant vous avertir via un "Warning" si vous essayez de faire un `-=` sur une Lambda anonyme (ce qui, comme nous l'avons vu, ne fait rien).
- **Impact :** Moins de bugs de "fuite de mémoire" où un abonné reste accroché parce que le désabonnement a échoué silencieusement.

- **Performance (Zero-Allocation) :** Le compilateur C# 14 (livré avec .NET 10) est encore plus intelligent pour optimiser les _Method Groups_. Dans certains cas, il parvient à mettre en cache l'instance du délégué pour éviter de créer un nouvel objet en mémoire lors de l'abonnement dans une boucle, réduisant la pression sur le Garbage Collector.
- **Weak Event Patterns :** Pour éviter les fuites de mémoire (lorsqu'un abonné n'est pas désinscrit et empêche la suppression de l'objet), .NET 10 généralise des approches plus sécurisées, mais le `-=` manuel reste la norme pour le contrôle total.

# Comprendre les délégués génériques

Au [[Chapitre 10#L'espace de noms `System.Collections.Generic`|Chapitre 10]], j'ai mentionné que C# permet de définir des types de délégués génériques. Par exemple, supposons que vous souhaitiez définir un type de délégué capable d'appeler n'importe quelle méthode ne renvoyant aucune valeur et prenant un seul paramètre. Si l'argument en question peut varier, vous pouvez le modéliser à l'aide d'un paramètre de type. Pour illustrer cela, considérez le code suivant dans un nouveau projet d'application console nommé *GenericDelegate* :

```cs
Console.Title = "Generic Delegate";
Console.WriteLine("***** Generic Delegate *****\n");

// Abonnement des cibles
MyGenericDelegate<string> strTarget = new MyGenericDelegate<string>(
    StringTarget
);
strTarget("Some string data");

// Utilisation de la syntaxe comversion de groupe de méthode
MyGenericDelegate<int> intTarget = IntTarget;
intTarget(9);
Console.ReadLine();

static void StringTarget(string arg)
{
    Console.WriteLine($"arg in uppercase is: {arg.ToUpper()}");
}

static void IntTarget(int arg)
{
    Console.WriteLine($"++arg is: {++arg}");
}

// Ce délégué générique peut représenté n'importe quel
// méthode retournant void et prenant un seul paramètre de type T
public delegate void MyGenericDelegate<T>(T arg);
```

**Notez que `MyGenericDelegate<T>` définit un unique paramètre de type qui représente l'argument à transmettre à la cible du délégué**. ==Lors de la création d'une instance de ce type, vous devez spécifier la valeur du paramètre de type, ainsi que le nom de la méthode que le délégué appellera==. Ainsi, si vous spécifiez un type chaîne de caractères, vous transmettez une valeur de type chaîne de caractères à la méthode cible.

```cs
// Abonnement des cibles
// Crée une instance de MyGenericDelegate<T>
// avec string comme paramètre de type.
MyGenericDelegate<string> strTarget = new MyGenericDelegate<string>(StringTarget);
strTarget("Some string data");
```

Compte tenu du format de l'objet `strTarget`, la méthode `StringTarget()` doit désormais prendre une seule chaîne de caractères comme paramètre.

```cs
static void StringTarget(string arg)
{
    Console.WriteLine($"arg in uppercase is: {arg.ToUpper()}");
}
```

## Les délégués génériques `Action<>` et `Func<>`

>[!check] Très important 
>Cette sous-section du livre est très importante, car elle est la racine de toutes les autre technologie / framework lié à C# (EF.Core, ASP.NET, Blazor, MAUI, WinUI, Avalonia UI)

==Au cours de ce chapitre, vous avez constaté que pour utiliser des délégués afin d'activer les rappels dans vos applications, vous suivez généralement les étapes décrites ici== :

1. Définissez un délégué personnalisé correspondant au format de la méthode appelée.
2. Créez une instance de votre délégué personnalisé en lui passant le nom de la méthode comme argument du constructeur.
3. Appelez la méthode indirectement, via un appel à la méthode `Invoke()` sur l'objet délégué.

==Lorsque vous adoptez cette approche, vous vous retrouvez généralement avec plusieurs délégués personnalisés qui ne seront peut-être jamais utilisés au-delà de la tâche en cours== (par exemple, `MyGenericDelegate<T>`, `CarEngineHandler`, etc.). Bien qu'il puisse certainement être nécessaire d'avoir un type de délégué personnalisé et nommé de manière unique pour votre projet, le nom exact du type de délégué est parfois sans importance. ==Dans de nombreux cas, vous souhaitez simplement un délégué qui prend un ensemble d'arguments et qui peut éventuellement renvoyer une valeur autre que `void`.== **Dans ces cas, vous pouvez utiliser les types de délégués intégrés `Action<>` et `Func<>` du framework**. Pour illustrer leur utilité, créez un nouveau projet d'application console nommé *ActionAndFuncDelegates*.

**Le délégué générique `Action<>` est défini dans l'espace de noms `System`, et vous pouvez utiliser ce
délégué générique pour « pointer » vers une méthode qui prend jusqu'à** *16 arguments* (ce qui devrait être suffisant !) **et qui renvoie `void`**. ==Rappelons que, puisque `Action<>` est un délégué générique, vous devrez également spécifier le type sous-jacent de chaque paramètre.==

Mettez à jour votre fichier *Program.cs* pour définir une nouvelle méthode statique prenant trois paramètres uniques (environ). Voici un exemple :

```cs
// Ceci est une cible pour le délégué Action<> (Abonné)
static void DisplayMessage(string msg, ConsoleColor txtColor, int printCount)
{
    // Définir la couleur du texte de la console.
    ConsoleColor prev = Console.ForegroundColor;
    Console.ForegroundColor = txtColor;

    for (int i = 0; i < printCount; i++)
    {
      Console.WriteLine(msg);
    }

    // Restaure la couleur
    Console.ForegroundColor = prev;
}
```

**Désormais, plutôt que de créer manuellement un délégué personnalisé pour transmettre le flux du programme à la méthode `DisplayMessage()`, vous pouvez utiliser le délégué `Action<>` prêt à l'emploi, comme ceci :**

```cs
Console.Title = "Fun with Action and Func";
Console.WriteLine("***** Fun with Action and Func *****\n");

// Utilisation du délégué Action<> pour pointer à DisplayMessage.
Action<string, ConsoleColor, int> actionTarget = DisplayMessage;

actionTarget("Action Message!", ConsoleColor.Yellow, 5);

Console.ReadLine();
```

Comme vous pouvez le constater, **l'utilisation du délégué `Action<>` vous évite de définir un type de délégué personnalisé**. Cependant, **==n'oubliez pas que le type de délégué `Action<>` ne peut pointer que vers des méthodes renvoyant `void`==**. **Si vous souhaitez pointer vers une méthode renvoyant une valeur** (et ne voulez pas vous embêter à écrire vous-même le délégué personnalisé), **vous pouvez utiliser `Func<>`**.

**Le délégué générique `Func<>` peut pointer vers des méthodes qui** (comme `Action<>`) **acceptent jusqu'à 16 paramètres et une valeur de retour personnalisée**. Pour illustrer cela, ajoutez la méthode suivante au fichier *Program.cs* :

> [!Tip] La Grande Ligne de Faille : `void` vs `Return Type` (généré par Gemini)
> En C#, la différence entre une procédure (`void`) et une fonction (avec retour) n'est pas qu'une question de style, c'est une contrainte matérielle et logique qui dicte tout le système de types.
>
> ---
>
> ### 1. Au niveau du Processeur (La réalité binaire)
> Lorsqu'une méthode est exécutée, le processeur utilise des **registres** (comme `RAX`) et la **Pile (Stack)** pour gérer les données.
> - **Méthode avec retour :** Le contrat stipule que la méthode *doit* laisser une valeur dans le registre de sortie avant de rendre la main. L'appelant attend cette valeur.
> - **Méthode `void` :** Le processeur ignore le registre de sortie. Si on essayait de lire le retour d'une méthode `void`, on récupèrerait un "déchet" (une valeur résiduelle en mémoire), ce qui corromprait la logique du programme.
>
> ### 2. Pourquoi `Action<>` et `Func<>` existent-ils séparément ?
> Cette séparation "physique" se répercute sur les délégués génériques à cause d'une limitation du langage :
> - **Le problème de `void` :** En C#, `void` n'est pas un type (on ne peut pas créer une `List<void>`).
> - **L'impossibilité d'unifier :** Comme on ne peut pas passer `void` comme paramètre générique à un `Func<T>`, Microsoft a dû créer `Action`.
> - **Analogie :** `Func` est une boîte avec une fente de sortie. `Action` est une boîte scellée. On ne peut pas utiliser l'une pour l'autre car le circuit électrique (le code machine) n'est pas câblé de la même façon.
>
> ### 3. Le "Dead-on Match" de Troelsen
> Pour qu'un délégué pointe vers une méthode, il faut que leurs "empreintes digitales" (signatures) soient identiques :
> 1. **Mêmes paramètres :** Pour que la pile soit lue correctement.
> 2. **Même type de retour :** Pour que le registre de sortie soit géré correctement.
> 
> > [!IMPORTANT] Règle d'or
> > - **PROCÉDURE (`void` / `Action`)** = Instruction (On donne un ordre). Idéal pour les événements (ex: `CarIsExploding`).
> > - **FONCTION (`T` / `Func`)** = Expression (On pose une question). Idéal pour les calculs ou les transformations.
>
>### 4. La Spécialisation Logique : Le cas du Prédicat
>
>Bien qu'un Prédicat soit techniquement une Fonction (il retourne un `bool`), il occupe une place à part dans l'architecture LINQ.
>
>- **Le contrat sémantique :** Alors qu'une `Func` classique transforme (ex: `int` -> `string`), le Prédicat **valide**. Il répond à une question fermée : _Est-ce que cet élément appartient à ma sélection ?_
>- **L'optimisation .NET :** Le type `Predicate<T>` existe historiquement, mais LINQ utilise `Func<T, bool>`. Pourquoi ? Pour permettre la **composition**. On peut enchaîner des fonctions, mais on "finit" souvent par un prédicat pour filtrer la donnée avant qu'elle ne sorte du pipeline.
>
>### 5. La "Méthode Itératrice" : Le moteur de l'exécution différée
>
>Il existe une catégorie "hybride" qui ne rentre pas dans le moule classique de la Pile (Stack) : les méthodes utilisant `yield return`.
>
>- **Le saut quantique :** Contrairement à une fonction classique qui s'exécute de haut en bas et "meurt" après le `return`, une méthode itératrice **"suspend"** son exécution.
>- **L'état préservé :** Elle ne retourne pas une valeur, mais un **Itérateur** (comme le `ArrayWhereIterator` que nous avons vu). Elle se souvient d'où elle s'est arrêtée.
>- **Conséquence matérielle :** Le compilateur transforme ces méthodes en **machines à états** (State Machines). Ce ne sont plus de simples appels de fonctions sur la pile, mais des objets complexes qui vivent sur le **Tas (Heap)**.
>
>---
> _Note : Dans les langages comme Rust ou Kotlin, `void` est remplacé par un type réel (`Unit`), ce qui permet d'unifier ces deux concepts. C# reste fidèle à ses racines C++ avec cette séparation stricte._

##### Tableau 12-2 : Les différents types de méthode et les rôles

| Concept       | Type C#       | Rôle Matériel                                | Intention Logique                        |
| ------------- | ------------- | -------------------------------------------- | ---------------------------------------- |
| **PROCÉDURE** | `Action`      | Modifie la mémoire/registres sans retour.    | **ORDRE** : "Fais ceci."                 |
| **FONCTION**  | `Func`        | Laisse une valeur dans le registre `RAX`.    | **TRANSFORMATION** : "Deviens cela."     |
| **PRÉDICAT**  | `Predicate`   | Laisse un drapeau (0 ou 1) dans le registre. | **VALIDATION** : "Est-ce vrai ?"         |
| **ITÉRATEUR** | `IEnumerable` | Crée une machine à états sur le Tas.         | **GÉNÉRATION** : "Donne-moi le suivant." |


> [!IMPORTANT]- Faire le lien entre le [[Chapitre 11#Utilisation des types pointeurs|Chapitre 11]] et les délégués
> ## Le Délégué : L'unique "Pointeur de Fonction" du monde managé
> Dans un environnement non managé (C/C++), on utilise des **pointeurs de fonction** bruts (adresses mémoire). C'est puissant, mais extrêmement dangereux : une adresse erronée provoque un *Segmentation Fault* ou corrompt la mémoire.
>
> ### 1. Un pointeur sous haute surveillance
> Le délégué est la réponse de C# pour conserver cette puissance tout en restant dans le **Managed World** :
> - **Encapsulation :** Le délégué ne contient pas juste une adresse (`0x1234`), il enveloppe l'adresse de la méthode ET une référence vers l'instance de l'objet cible.
> - **Vérification au JIT :** Contrairement aux pointeurs bruts, le moteur d'exécution (CLR) valide la signature de la méthode avant l'appel. Si les types ne correspondent pas, le code ne s'exécute même pas.
>
> ### 2. Pourquoi est-ce la "seule" voie ?
> En C# managé, vous n'avez pas le droit de manipuler directement les adresses mémoire (sauf dans un bloc `unsafe`). Le délégué est donc **l'unique abstraction autorisée** par le CLR pour :
> - Passer du code en paramètre.
> - Stocker une méthode dans une variable.
> - Implémenter des rappels (callbacks) sans risquer de corrompre le *Heap* ou la *Stack*.
>
> ### 3. Le rempart contre la corruption
> Parce que le délégué est "Type Safe", il garantit qu'aucun appel ne débordera de la mémoire allouée. Il transforme ce qui pourrait être un crash système majeur en une simple exception gérée, préservant ainsi l'intégrité de tout le processus managé.

```cs
// Cible du délégué Func<>.
static int Add(int x, int y)
{
    return x + y;
}
```

Plus tôt dans ce chapitre, je vous ai demandé de créer un délégué `BinaryOp` personnalisé pour « pointer vers » les méthodes d'addition et de soustraction. Vous pouvez toutefois simplifier votre démarche en utilisant une version de `Func<>` qui prend trois paramètres de type. ==Notez que le *dernier* paramètre de type de `Func<>` est *toujours* la valeur de retour de la méthode==. Pour bien comprendre ce point, supposons que le fichier *Program.cs* définisse également la méthode suivante :

```cs
static string SumToString(int x, int y)
{
    return (x + y).ToString();
}
```

Le code appelant peut désormais appeler chacune de ces méthodes, comme suit :

```cs
Func<int, int, int> funcTarget = Add;
int result = funcTarget.Invoke(40, 40);
Console.WriteLine($"40 + 40 = {result}");

Func<int, int, string> funcTarget2 = SumToString;
string sum = funcTarget2.Invoke(90, 300);
Console.WriteLine($"90 + 300 = {sum}");
```

Quoi qu'il en soit, étant donné que `Action<>` et `Func<>` vous évitent de définir manuellement un délégué personnalisé, ==vous vous demandez peut-être s'il faut les utiliser systématiquement==. La réponse, comme souvent en programmation, est : « cela dépend ». **Dans de nombreux cas, `Action<>` et `Func<>` seront la solution privilégiée**. Cependant, ==si vous avez besoin d'un délégué avec un nom personnalisé qui, selon vous, reflète mieux votre domaine d'application, la création d'un délégué personnalisé se résume à une simple instruction de code==. Vous découvrirez les deux approches dans la suite de ce texte.

Ceci conclut notre aperçu initial du type délégué. Passons maintenant au sujet connexe du mot-clé `event` en C#.

# Comprendre les événements C#

>[!success] Cette section est très importante. Comme pour `Action<>` et `Func<>`, les événements sont utilisé par tout les framework de C#.

**Les délégués sont des constructions intéressantes, car elles permettent aux objets en mémoire d'engager une conversation bidirectionnelle**. Cependant, ==travailler directement avec les délégués peut impliquer la création de code répétitif== (définition du délégué, déclaration des variables membres nécessaires, création de méthodes d'abonnement et de désabonnement personnalisées pour préserver l'encapsulation, etc.).

De plus, ==lorsque vous utilisez directement les délégués comme mécanisme de rappel de votre application, si vous ne définissez pas les variables membres du délégué d'une classe comme privées, l'appelant aura un accès direct aux objets délégués==. **Dans ce cas, l'appelant pourrait réaffecter la variable à un nouvel objet délégué** (supprimant ainsi la liste actuelle des fonctions à appeler) **et, pire encore, il pourrait invoquer directement la liste d'appels du délégué**. Pour illustrer ce problème, créez une nouvelle application console nommée *PublicDelegateProblem* et ajoutez la réécriture (et la simplification) suivante de la classe `Car` de l'exemple *CarDelegate* précédent :

```cs
namespace PublicDelegateProblem;

public class Car
{
    public delegate void CarEngineHandler(string msgForCaller);

    // Maintenant un membre publique!
    public CarEngineHandler ListOfHandlers;

    // Déclenchez simplement la notification Exploded.
    public void Accelerate(int delta) =>
        ListOfHandlers?.Invoke("Sorry, this car is dead...");
}
```

**Notez que vous n'avez plus de variables membres déléguées privées encapsulées avec des méthodes d'enregistrement personnalisées**. ==Ces membres étant désormais publics, l'appelant peut accéder directement à la variable membre `ListOfHandlers`, la réaffecter à de nouveaux objets `CarEngineHandler` et invoquer le délégué à sa guise.==

```cs
using PublicDelegateProblem;

Console.Title = "Agh! No Encapsulation!";
Console.WriteLine("***** Agh! No Encapsulation! *****\n");

// Crée un objet Car
Car myCar = new Car();

// On a accès directement au délégué!
myCar.ListOfHandlers = CallWhenExploded;
myCar.Accelerate(10);

// On peut maintenant l'assigner à un tout nouvel objet...
// c'est pour le moins déroutant.
myCar.ListOfHandlers = CallHereToo;
myCar.Accelerate(10);

// L'appelant peut aussi invoquer directement le délégué!
myCar.ListOfHandlers.Invoke("hee, hee, hee...");
Console.ReadLine();

static void CallWhenExploded(string msg)
{
    Console.WriteLine(msg);
}

static void CallHereToo(string msg)
{
    Console.WriteLine(msg);
}
```

***==Exposer des membres délégués publics rompt l'encapsulation, ce qui peut non seulement rendre le code difficile à maintenir (et à déboguer), mais aussi exposer votre application à des risques de sécurité !==*** Voici le résultat de l'exemple actuel :

```
***** Agh! No Encapsulation! *****

Sorry, this car is dead...
Sorry, this car is dead...
hee, hee, hee...
```

Il est évident que vous ne souhaitez pas donner à d'autres applications la possibilité de modifier ce vers quoi pointe un délégué ou d'appeler ses membres sans votre autorisation. **C'est pourquoi il est courant de déclarer des variables membres de délégué privées**.

## Le mot-clé `event` de C#

**Pour simplifier l'ajout ou la suppression de méthodes à la liste d'appels d'un délégué sans avoir à créer de méthodes personnalisées, C# fournit le mot-clé `event`**. Lors du traitement de ce mot-clé par le compilateur, ==les méthodes d'abonnement et de désabonnement, ainsi que les variables membres nécessaires à vos types délégués, sont automatiquement fournies==. Ces variables membres sont *toujours* déclarées privées et ne sont donc pas directement exposées par l'objet déclenchant l'événement. **Le mot-clé `event` permet de simplifier l'envoi de notifications à des objets externes par une classe personnalisée.**

Définir un événement se fait en deux étapes. Premièrement, **il faut définir un type délégué** (ou en réutiliser un existant) **qui contiendra la liste des méthodes à appeler lors du déclenchement de l'événement**. Ensuite, **il faut déclarer l'événement** (à l'aide du mot-clé `event` de C#) **en fonction du type délégué associé.**

Pour illustrer le mot-clé `event`, créez une application console nommée *CarEvents*. Dans cette itération de la classe `Car`, vous définirez deux événements : `AboutToBlow` et `Exploded`. Ces événements sont associés à un seul type de délégué nommé `CarEngineHandler`. Voici les mises à jour initiales de la classe `Car` :

```cs
public class Car
{
	...
	
    // Ce délégué travaille conjointement avec les événements Car
    public delegate void CarEngineHandler(string msgForCaller);

    // 2) Cette objet peut envoyé ces événements
    public event CarEngineHandler Exploded;
    public event CarEngineHandler AboutToBlow;

	...
}
```

Envoyer un événement à l'appelant est aussi simple que de spécifier l'événement par son nom, ainsi que tous les paramètres requis tels que définis par le délégué associé. **Pour s'assurer que l'appelant s'est bien enregistré auprès de l'événement, il est conseillé de vérifier que l'événement n'a pas une valeur `null` avant d'appeler la méthode du délégué**. En tenant compte de ces points, voici la nouvelle version de la méthode `Accelerate()` de l'objet `Car` :

```cs
public void Accelerate(int delta)
{
	// Si cette voiture est "morte", enclenche l'événement Exploded 
	if (_carIsDead)
	{
		Exploded?.Invoke("Sorry, this car is dead...");
	}
	else
	{
		CurrentSpeed += delta;
		// Cette voiture est-elle « presque morte » ?
		if (10 == MaxSpeed - CurrentSpeed)
		{
			AboutToBlow?.Invoke("Careful buddy! Gonna blow!");
		}
		
		// Toujours OK!
		if (CurrentSpeed >= MaxSpeed)
		{
			_carIsDead = true;
		}
		else
		{
			Console.WriteLine($"currentSpeed = {CurrentSpeed}");
		}
	}
}
```

Vous avez ainsi configuré la voiture pour envoyer deux événements personnalisés sans avoir à définir de fonctions d'enregistrement personnalisées ni à déclarer de variables membres déléguées. ==Vous verrez l'utilité de cette nouvelle automobile dans un instant, mais examinons d'abord l'architecture événementielle plus en détail.==

## Les événements sous le capot

**Lorsque le compilateur traite le mot-clé `event` en C#, il génère deux méthodes cachées : l'une préfixée par `add_` et l'autre par `remove_`**. ==Chaque préfixe est suivi du nom de l'événement C#==. Par exemple, l'événement `Exploded` génère deux méthodes cachées nommées `add_Exploded()` et `remove_Exploded()`. Si vous examiniez les instructions CIL de `add_AboutToBlow()`, vous trouveriez un appel à la méthode `Delegate.Combine()`. Voici un extrait de code CIL :

```CIL
  .method public hidebysig specialname instance void 
          add_AboutToBlow(class CarDelegate.Car/CarEngineHandler 'value') cil managed
  {
	...
	
    IL_000b:  call       class [System.Runtime]System.Delegate [System.Runtime]System.Delegate::Combine(class [System.Runtime]System.Delegate, class [System.Runtime]System.Delegate)
    
    ...
  
  } // end of method Car::add_AboutToBlow
```

Comme vous pouvez vous y attendre, `remove_AboutToBlow()` appellera `Delegate.Remove()` en votre nom.

```CIL
  .method public hidebysig specialname instance void 
          remove_AboutToBlow(class CarDelegate.Car/CarEngineHandler 'value') cil managed
  {
  
	...
	
    IL_001e:  call       !!0 [System.Threading]System.Threading.Interlocked::CompareExchange<class CarDelegate.Car/CarEngineHandler>(!!0&, !!0, !!0)
    
    ...
  
  } // end of method Car::remove_AboutToBlow

```

Enfin, le code CIL représentant l'événement lui-même utilise les directives `.addon` et `.removeon` pour mapper les noms des méthodes `add_XXX()` et `remove_XXX()` correctes à invoquer.

```CIL
  .event CarDelegate.Car/CarEngineHandler AboutToBlow
  {
    .addon instance void CarDelegate.Car::add_AboutToBlow(class CarDelegate.Car/CarEngineHandler)
    .removeon instance void CarDelegate.Car::remove_AboutToBlow(class CarDelegate.Car/CarEngineHandler)
  } // end of event Car::AboutToBlow
```

Maintenant que vous comprenez comment créer une classe capable d'envoyer des événements C# (et que vous savez que les événements ne sont rien de plus qu'un gain de temps lors de la saisie), ==la question suivante est de savoir comment écouter les événements entrants du côté de l'appelant.==

## Écoute des événements entrants

**Les événements C# simplifient également l'enregistrement des gestionnaires d'événements côté appelant**. ==Au lieu de devoir spécifier des méthodes d'assistance personnalisées, l'appelant utilise simplement les opérateurs `+=` et `-=`== (ce qui déclenche la méthode appropriée `add_XXX()` ou `remove_XXX()` en arrière-plan). Pour vous enregistrer à un événement, suivez le modèle présenté ici :

```cs
// NameOfObject.NameOfEvent +=
//    new RelatedDelegate(functionToCall);
Car.CarEngineHandler d = new Car.CarEngineHandler(CarExplodedEventHandler);
myCar.Exploded += d;
```

Pour vous détacher d'une source d'événements, utilisez l'opérateur `-=` selon le modèle suivant :

```cs
// NameOfObject.NameOfEvent -=
//    new RelatedDelegate(functionToCall);
myCar.Exploded -= d;
```

Notez que vous pouvez également utiliser la syntaxe de conversion de groupe de méthodes avec les événements (avec ces qualité et ses défauts. Voir [[#Method Group Conversion vs Instances de Délégués (Gemini)|Ici]]) :

```cs
Car.CarEngineHandler d = CarExplodedEventHandler;
myCar.Exploded += d;
```

Compte tenu de ces schémas très prévisibles, voici le code d'appel remanié, utilisant désormais la syntaxe d'enregistrement d'événements C# :

```cs
using CarEvents;

Console.Title = "Fun with Events";
Console.WriteLine("***** Fun with Events *****\n");

Car c1 = new Car("SlugBug", 100, 10);

// Abonnement des gestionnaires.
c1.AboutToBlow += CarIsAlmostDoomed;
c1.AboutToBlow += CarAboutToBlow;

Car.CarEngineHandler d = CarExploded;
c1.Exploded += d;

Console.WriteLine("***** Speeding up *****");
for (int i = 0; i < 6; i++)
{
    c1.Accelerate(20);
}

// désabonne la méthode CarExploded
// de la liste d'appel.
c1.Exploded -= d;

Console.WriteLine("\n***** Speeding up *****");
for (int i = 0; i < 6; i++)
{
    c1.Accelerate(20);
}
Console.ReadLine();

static void CarAboutToBlow(string msg)
{
    Console.WriteLine(msg);
}
static void CarIsAlmostDoomed(string msg)
{
    Console.WriteLine("=> Critical Message from Car: {0}", msg);
}
static void CarExploded(string msg)
{
    Console.WriteLine(msg);
}
```

>[!warning]- Abonner une méthode à la création d'objet.
>Envoyé les méthodes dans les constructeurs pour les abonné dés la création d'un objet est une **mauvaise pratique** pour plusieurs raisons:
>
>- **Fuites de mémoire** : Si vous passez une méthode d'un objet tiers au constructeur, l'objet `Car` garde une référence forte vers cet objet. Cela empêche le Garbage Collector de nettoyer les objets.
>- **Problème de cycle de vie** : L'objet n'est pas encore "complètement construit" que déjà il tente d'interagir avec des délégués externes.
>- **Manque de flexibilité** : L'abonnement devient "obligatoire". L'intérêt des événements est justement de permettre à l'utilisateur de l'objet de choisir s'il veut écouter ou non, et quand.
>- **Violation du SRP** : Le constructeur doit servir à initialiser l'état interne (nom, vitesse), pas à gérer les dépendances de notification externes.
>
>### Quand passer des méthodes au constructeur ?
>
>Si vous voulez vraiment forcer une action dès la création, on n'utilise généralement pas d'événements mais le **Pattern Strategy** ou l'**Injection de Dépendances** :
>
>- **Interface/Action** : Passez une `Action<string>` ou une interface si le comportement est _obligatoire_ pour le fonctionnement de la classe.
>- **Événement** : Utilisez `+=` si le comportement est _optionnel_ ou peut changer durant la vie de l'objet.

## Simplification de l'enregistrement des événements avec Visual Studio

Visual Studio facilite l'enregistrement des gestionnaires d'événements. Lorsque vous utilisez la syntaxe `+=` lors de l'enregistrement d'un événement, une fenêtre IntelliSense s'affiche et vous invite à appuyer sur la touche Tab pour saisir automatiquement l'instance du délégué associée (voir image suivante), capturée à l'aide de la *syntaxe de conversion des groupes de méthodes*.

![[Figure 12.1.png|Sélection de délégué IntelliSense]]

Après avoir appuyé sur la touche Tab, l'IDE générera automatiquement la nouvelle méthode, comme indiqué dans l'image suivante.

![[Figure 12.2.png|Format cible délégué IntelliSense]]

Notez que le code factice est au format correct de la cible du délégué (notez que cette méthode a été déclarée statique car l'événement a été enregistré dans une méthode statique).

```cs
static void NewCar_AboutToBlow(string msg)
{
	throw new NotImplementedException();
}
```

## Création d'arguments d'événements personnalisés

À vrai dire, ==il existe une dernière amélioration possible à apporter à la version actuelle de la classe Car, qui reflète le modèle d'événements recommandé par Microsoft==. En explorant les événements envoyés par un type donné dans les bibliothèques de classes de base, vous constaterez que **le premier paramètre du délégué sous-jacent est un `System.Object`, tandis que le second paramètre est un descendant de `System.EventArgs`**.

==L'argument `System.Object` représente une référence à l'objet ayant émis l'événement== (par exemple, `Car`), ==tandis que le second paramètre contient des informations relatives à l'événement en question==. La classe de base `System.EventArgs` représente un événement n'envoyant aucune information personnalisée.

```cs
public class EventArgs
{
	public static readonly EventArgs Empty;
	public EventArgs();
}
```

**Pour les événements simples, vous pouvez transmettre directement une instance de `EventArgs`**. Cependant, **==si vous souhaitez transmettre des données personnalisées, vous devez créer une classe appropriée dérivée de `EventArgs`==**. Dans cet exemple, supposons que vous ayez une classe nommée `CarEventArgs`, qui stocke une chaîne de caractères représentant le message envoyé au destinataire.

```cs
namespace CarEvents;

public class CarEventArgs : EventArgs
{
    public readonly string msg;

    public CarEventArgs(string message)
    {
        msg = message;
    }
}
```

Vous pouvez maintenant mettre à jour la définition du type délégué `CarEngineHandler` comme suit (les événements restent inchangés) :

```cs
public class Car
{
	...
	
    public delegate void CarEngineHandler(object sender, CarEventArgs e);
	
	...
	
}
```

**Désormais, lors du déclenchement des événements depuis la méthode `Accelerate()`, vous devrez fournir une référence à l'objet `Car` actuelle (via le mot-clé `this`) et une instance du type `CarEventArgs`**. Par exemple, considérez la mise à jour partielle suivante :

```cs
public void Accelerate(int delta)
{
	// Si cette voiture est "morte", enclenche l'événement Exploded
	if (_carIsDead)
	{
		Exploded?.Invoke(
			this,
			new CarEventArgs("Sorry, this car is dead...")
		);
	}
	else
	{
		CurrentSpeed += delta;
		// Cette voiture est-elle « presque morte » ?
		if (10 == MaxSpeed - CurrentSpeed)
		{
			AboutToBlow?.Invoke(
				this,
				new CarEventArgs("Careful buddy! Gonna blow!")
			);
		}
		...
	}
}
```

**Du côté de l'appelant, il vous suffit de mettre à jour vos gestionnaires d'événements pour recevoir les paramètres entrants et récupérer le message via le champ en lecture seule**. Voici un exemple :

```cs
static void CarAboutToBlow(object sender, CarEventArgs e)
{
    Console.WriteLine($"{sender} says: {e.msg}");
}
```

**Si le destinataire souhaite interagir avec l'objet ayant envoyé l'événement, vous pouvez effectuer un cast explicite de `System.Object`**. À partir de cette référence, vous pouvez utiliser n'importe quel membre public de l'objet ayant envoyé la notification d'événement. 

```cs
static void CarAboutToBlow(object sender, CarEventArgs e)
{
    // Pour être sur, effectue une vérification à
    // l'exécution avant de convertir.
    if (sender is Car c)
    {
        Console.WriteLine($"Critical message from {c.PetName}: {e.msg}");
    }
}
```

>[!tip] Pourquoi l'argument `sender` est typé `object` et non pas une classe plus restrictive ?
>- **Interopérabilité avec les frameworks .NET**. Ils sont tous basé sur cette signature:
>```cs
>voidHandler(objet sender, EventArgs e)
>```
>- **Flexibilité pour le re-raise**. Un événement peut être retransmis par un intermédiaire
>```cs
>button.Click += (s, e) => myControl.RaiseEvent(s, e); // s n'est pas myControl
>```
>- **Héritage**. Si une classe fille hérite et lève le même événement, le sender typé fortement serait incorrect.
>>[!Example] Quand c'est acceptable ?
>>Dans des systèmes **fermés** où tu contrôles tout et que tu n'exposeras jamais l'événement à l'extérieur :
>>
>>```csharp
>>public event Action<MyClass, MyEventArgs> MyEvent;
>>```
>>
>>Certains frameworks modernes comme **Reactive Extensions (Rx)** ou les **source generators** adoptent d'ailleurs des approches plus typées.


## Le délégué générique `EventHandler<T>`

**Étant donné que de nombreux délégués personnalisés prennent un objet comme premier paramètre et un descendant d'`EventArgs` comme second, vous pouvez simplifier davantage l'exemple précédent en utilisant le type générique `EventHandler<T>`, où `T` est votre type `EventArgs` personnalisé**. Considérez la mise à jour suivante du type `Car` (==remarquez que vous n'avez plus besoin de définir de type de délégué personnalisé==) :

```cs
public class Car
{
	...
	
    public event EventHandler<CarEventArgs> Exploded;
    public event EventHandler<CarEventArgs> AboutToBlow;

	...
}
```

**Le code appelant pourrait alors utiliser `EventHandler<CarEventArgs>` partout où vous aviez précédemment spécifié `CarEventHandler`** (ou, une fois de plus, utiliser la conversion de groupe de méthodes).

```cs
using CarEvents;

Console.Title = "Fun with Events";
Console.WriteLine("***** Fun with Events *****\n");

Car c1 = new Car("SlugBug", 100, 10);

// Abonnement des gestionnaires.
c1.AboutToBlow += CarIsAlmostDoomed;
c1.AboutToBlow += CarAboutToBlow;

EventHandler<CarEventArgs> d = CarExploded;
c1.Exploded += d;
...
```

>[!tip] Autre possibilité depuis C# 12: Utiliser `using` pour générer un nom pour un type
>```cs
>using CarHandler = Action<object, CarEventArgs>;
>class Car
>{
>	...
>	public event CarHandler TireExploded;
>	public event CarHandler EngineExploded;
>	public event CarHandler SpeedUp;
>
>```

Parfait ! Vous avez maintenant découvert les aspects essentiels de l’utilisation des délégués et des événements en C#. Bien que **ces informations vont vous être utiles pour la quasi-totalité de vos besoins en matière de rappels**, nous conclurons ce chapitre par un aperçu de quelques simplifications finales, notamment les méthodes anonymes et les expressions lambda.

>[!tip]- Petit résumé des délégués génériques et des événements
> ### `Action<...>` : L'exécutant (Void)
> 
>C'est un délégué qui pointe vers une méthode qui **fait quelque chose** mais **ne renvoie rien** (`void`).
>
>- **Usage** : Pour des rappels (_callbacks_) simples.
>- **Exemple** : `Action<string> logger = msg => Console.WriteLine(msg);`
>- **Variantes** : Peut prendre de 0 à 16 paramètres (`Action`, `Action<T1>`, `Action<T1, T2>`, etc.).
>
> ### `Func<..., TResult>` : Le calculateur (Return)
> 
>C'est un délégué qui pointe vers une méthode qui **renvoie une valeur**.
>	
>- **Usage** : Pour transformer des données ou obtenir un résultat.
>- **Exemple** : `Func<int, int, int> add = (a, b) => a + b;` (Prend deux `int`, renvoie un `int`).
>- **Règle** : Le **dernier** type dans les chevrons `<...>` est toujours le type de retour.
>
> ### `EventHandler<T>` : Le messager officiel (Events)
>
>C'est un délégué spécialisé, rigoureusement conçu pour le **Pattern Événement** de .NET.
>	
>- **Signature forcée** : Il impose toujours deux paramètres : `(object? sender, T e)`.
>- **Usage** : Exclusivement pour les `event`.
>- **Exemple** : `public event EventHandler<CarEventArgs>? Exploded;`**
>
>### Pourquoi utiliser `EventHandler<T>` plutôt qu'une `Action<object, T>` ?
>
>Même si techniquement une `Action<object, T>` a la même tête, on préfère `EventHandler` pour deux raisons :
>
>1. **Sémantique** : Quand un développeur voit `EventHandler`, il sait tout de suite qu'il s'agit d'un événement standard.
>2. **Contraintes** : `EventHandler<T>` oblige `T` à hériter de `EventArgs` (dans les versions plus anciennes) ou suit au moins la convention qui permet aux outils (comme les designers visuels ou les générateurs de code) de fonctionner correctement.
>
>>[!warning] Il ne faut pas lié des délégué `Func<>` à des événements! (Gemini)
>>### Le problème du retour (`Return Value`)
>>Un événement est fait pour être **Multicast** (plusieurs abonnés). Si tu as trois abonnés à un `Func<int, int>` :
>>
>>- L'abonné A renvoie `10`.
>>- L'abonné B renvoie `20`.
>>- L'abonné C renvoie `30`.
>>
>>Quand tu déclenches l'événement : `int result = CalculateSomething?.Invoke(5);` **Seule la valeur du dernier abonné (C) sera conservée.** Les résultats de A et B sont perdus dans le vide.
>>
>>C'est pour cela que les événements utilisent presque toujours `void` (ou `EventHandler`) : on notifie, on n'attend pas de réponse.
>>
>>### L'encapsulation `event` vs `delegate`
>>
>>Il ne faut pas confondre le **type de délégué** (`Func`, `Action`) et le **mot-clé** `event`.
>>
>>- Si tu déclares `public Func<int, int> OnAction;` (sans le mot-clé `event`) : n'importe qui peut écraser tous les abonnés en faisant `c1.OnAction = ...` au lieu de `+=`.
>>- Si tu ajoutes le mot-clé `event`, tu protèges la liste d'appel, mais tu te retrouves avec le problème du point n°1 si tu n'utilises pas `void`.
>>
>> ### Conclusion
>>Techniquement, rien ne t'interdit de lier un `Func` à un `event`. Le compilateur l'acceptera car il va simplement générer :
>>
>>1. Un délégué privé `Func<int, int>`.
>>2. Des méthodes `add_CalculateSomething` et `remove_CalculateSomething`.
>>
>>Mais tu te battras contre la logique même du langage dès que tu auras plus d'un abonné.

## Comprendre les méthodes anonymes en C#

Comme vous l'avez vu, **lorsqu'un appelant souhaite écouter des événements entrants, il doit définir une méthode personnalisée dans une classe** (ou une structure) **dont la signature correspond à celle du délégué associé**. Voici un exemple :

```cs
Sometype t = new SomeType()

// Supposons que « SomeDelegate » puisse pointer vers des méthodes ne prenant aucun
// argument et ne retournant aucune valeur
t.SomeEvent += new SomeDelegate(MyEventHandler);

// Généralement appelée uniquement par l'objet SomeDelegate.
static void MyEventHandler()
{
	// Fait quelque chose quand l'événement est déclenché
}
```

Cependant, **à bien y réfléchir, les méthodes telles que `MyEventHandler()` sont rarement destinées à être appelées par une autre partie du programme que le délégué appelant**. ==En termes de productivité, définir manuellement une méthode distincte à appeler par l'objet délégué est un peu fastidieux== (bien que non rédhibitoire).

Pour pallier ce problème, **il est possible d'associer un événement directement à un bloc d'instructions de code lors de l' son enregistrement**. Formellement, ==ce code est appelé *méthode anonyme*==. Pour illustrer la syntaxe, commencez par créer une nouvelle application console nommée *AnonymousMethods*, puis copiez les classes *Car.cs* et *CarEventArgs.cs* du projet *CarEvents* dans le nouveau projet (en veillant à modifier leur espace de noms en `AnonymousMethods`). Mettez à jour le code du fichier *Program.cs* pour qu'il corresponde à ce qui suit, lequel gère les événements envoyés par la classe `Car` à l'aide de méthodes anonymes, plutôt que de gestionnaires d'événements nommés :

```cs
using AnonymousMethods;

Console.Title = "Anonymous Methods";
Console.WriteLine("***** Anonymous Methods *****\n");

Car c1 = new Car("SlugBug", 100, 10);

// Enregistre les gestionnaires d'événements en tant que méthodes anonymes.
c1.AboutToBlow += delegate
{
    Console.WriteLine("Eek! Going too fast!");
};

c1.AboutToBlow += delegate(object sender, CarEventArgs e)
{
    Console.WriteLine($"Message from Car: {e.msg}");
};

c1.Exploded += delegate(object sender, CarEventArgs e)
{
    Console.WriteLine($"Fatal message from Car: {e.msg}");
};

// Ceci va éventuellement déclencher les événements.
for (int i = 0; i < 6; i++)
{
    c1.Accelerate(20);
}

Console.ReadLine();
```

>[!note]
>L'accolade fermante d'une méthode anonyme doit se terminer par un point-virgule. Si vous ne le faites pas, une erreur de compilation sera générée.

Notez encore une fois que **le code appelant n'a plus besoin de définir de gestionnaires d'événements statiques spécifiques tels que `CarAboutToBlow()` ou `CarExploded()`**. Les méthodes anonymes sont désormais définies directement au moment où l'appelant traite l'événement à l'aide de la syntaxe `+=`. La syntaxe de base d'une méthode anonyme correspond au pseudocode suivant :

```cs
UnType t = new UnType();
t.UnEvent += delegate (argumentDéléguéDéfiniOptionellement)
{ /* déclaration */ };
```

Lors du traitement du premier événement `AboutToBlow` dans l'exemple de code précédent, notez que vous ne spécifiez pas les arguments transmis par le délégué.

```cs
c1.AboutToBlow += delegate
{
	Console.WriteLine("Eek! Going too fast!");
};
```

À proprement parler, ==vous n'êtes pas tenu de recevoir les arguments entrants d'un événement spécifique==. Toutefois, **si vous souhaitez utiliser les arguments entrants possibles, vous devrez spécifier les paramètres prototypés par le type délégué** (comme illustré dans la seconde gestion des événements `AboutToBlow` et `Exploded`). Voici un exemple :

```cs
c1.AboutToBlow += delegate(object sender, CarEventArgs e)
{
    Console.WriteLine($"Message from Car: {e.msg}");
};
```

## Accès aux variables locales

Les méthodes anonymes sont intéressantes car **elles peuvent accéder aux variables locales de la méthode qui les définit**. Formellement, ces variables sont appelées *variables externes* de la méthode anonyme. Il convient de mentionner les points importants suivants concernant l'interaction entre la portée d'une méthode anonyme et la portée de la méthode qui la définit :

- Une méthode anonyme ==ne peut pas accéder aux paramètres `ref` ou `out` de la méthode qui la définit.==
- Une méthode anonyme ==ne peut pas avoir de variable locale portant le même nom qu'une variable locale de la méthode englobante.==
- Une méthode anonyme ==peut accéder aux variables d'instance== (ou aux variables statiques, selon le cas) ==de la classe englobante.==
- Une méthode anonyme ==peut déclarer des variables locales portant le même nom que les variables membres de la classe englobante== (les variables locales ont une portée distincte et masquent les variables membres de la classe englobante).

**Supposons que vos instructions de niveau supérieur définissent un entier local nommé `aboutToBlowCounter`. Dans les méthodes anonymes qui gèrent l'événement `AboutToBlow`, vous incrémenterez ce compteur de un et afficherez le résultat avant la fin de l'exécution des instructions.**

```cs
using AnonymousMethods;

Console.Title = "Anonymous Methods";
Console.WriteLine("***** Anonymous Methods *****\n");

int aboutToBlowCounter = 0;

Car c1 = new Car("SlugBug", 100, 10);

// Enregistre les gestionnaires d'événements en tant que méthodes anonymes.
c1.AboutToBlow += delegate
{
    aboutToBlowCounter++;
    Console.WriteLine("Eek! Going too fast!");
};

c1.AboutToBlow += delegate(object sender, CarEventArgs e)
{
    aboutToBlowCounter++;
    Console.WriteLine($"Message from Car: {e.msg}");
};

...

// Ceci va éventuellement déclencher les événements.
for (int i = 0; i < 6; i++)
{
    c1.Accelerate(20);
}

Console.WriteLine($"aboutToBlow event was fired {aboutToBlowCounter} times.");

Console.ReadLine();
```

Après avoir exécuté ce code mis à jour, vous constaterez que la dernière instruction `Console.WriteLine()` indique que l'événement `AboutToBlow` a été déclenché deux fois.

## Utilisation de `static` avec les méthodes anonymes (Nouveauté C# 9.0)

==L'exemple précédent illustrait l'interaction de méthodes anonymes avec des variables déclarées en dehors de la portée de la méthode elle-même==. **Bien que cela puisse correspondre à votre intention, cela enfreint le principe d'encapsulation et peut introduire des effets de bord indésirables dans votre programme**. Rappelons-nous du [[Chapitre 4#Comprendre les fonctions locales (Nouveauté C 7.0, MaJ C 9.0)|Chapitre 4]] que ==les fonctions locales peuvent être isolées du code qui les contient en les déclarant comme `static`==, comme dans l'exemple suivant :

```cs
static int AddWrapperWithStatic(int x, int y)
{ 
	// Effectuer une validation ici
	return Add(x, y);
	
	static int Add(int x, int y)
	{ 
		return x + y;
	}
}
```

Nouveauté de C# 9.0 : **les méthodes anonymes peuvent désormais être déclarées statiques afin de préserver l’encapsulation et de garantir qu’elles ne peuvent introduire aucun effet de bord dans le code qui les contient**. Par exemple, consultez la méthode anonyme mise à jour ici :

```cs
c1.AboutToBlow += static delegate
{
    // Ceci cause une erreur de compilation parce que c'est marqué static.
    aboutToBlowCounter++;
    Console.WriteLine("Eek! Going too fast!");
};
```

Le code précédent ne compilera pas car les méthodes anonymes tentent d'accéder à la variable déclarée en dehors de sa portée. Pour corriger l'erreur de compilation, commentez (ou supprimez) la ligne qui met à jour le compteur :

```cs
c1.AboutToBlow += static delegate
{
    // aboutToBlowCounter++;
    Console.WriteLine("Eek! Going too fast!");
};
```

## Valeurs ignorées avec les méthodes anonymes (Nouveauté C# 9.0)

**Les valeurs ignorées, introduites au [[Chapitre 4#Ignorer des valeurs (discards) de tuples|Chapitre 4]], ont été mises à jour dans C# 9.0 pour être utilisées comme paramètres d'entrée pour les méthodes anonymes, avec une exception**. Étant donné que le caractère de soulignement (`_`) était un identificateur de variable valide dans les versions précédentes de C#, ==il faut au moins deux valeurs ignorées utilisées avec la méthode anonyme pour qu'elles soient traitées comme telles==.

Par exemple, le code suivant crée un délégué pour un `Func` qui prend deux entiers et en renvoie un autre. Cette implémentation ignore les variables passées en paramètre et renvoie $42$ :

```cs
Func<int, int, int> constant = delegate(int _, int _)
{
    return 42;
};

Console.WriteLine("constant(3,4)={0}", constant(3, 4));
```

# Comprendre les expressions lambda

Pour conclure votre étude de l'architecture événementielle .NET, vous allez examiner les *expressions lambda* C#. **Comme expliqué précédemment, C# permet de gérer les événements « en ligne » en assignant un bloc d'instructions de code directement à un événement à l'aide de méthodes anonymes, plutôt que de créer une méthode autonome appelée par le délégué sous-jacent**. **==Les expressions lambda ne sont rien d'autre qu'une manière concise de créer des méthodes anonymes et, en fin de compte, de simplifier l'utilisation du type délégué .NET==**.

Pour préparer votre étude des expressions lambda, créez un nouveau projet d'application console nommé *LambdaExpressions*. Pour commencer, considérez la méthode `FindAll()` de la classe générique `List<T>`. Cette méthode peut être appelée lorsque vous devez extraire un sous-ensemble d'éléments de la collection et son prototype est le suivant :

```cs
// Méthode de System.Collections.Generic.List<T>
public List<T> FindAll(Predicate<T> match)
```

**Comme vous pouvez le constater, cette méthode renvoie une nouvelle `List<T>` représentant le sous-ensemble de données**. Notez également que **==le seul paramètre de `FindAll()` est un délégué générique de type `System.Predicate<T>`==**. **Ce type délégué peut pointer vers n'importe quelle méthode renvoyant un booléen et prend un seul paramètre de type `bool` comme unique paramètre d'entrée.**

```cs
// Ce délégué est utilisé par la méthode FindAll()
// pour extraire le sous-ensemble.
public delegate bool Predicate<T>(T obj);
```

==Lorsque vous appelez `FindAll()`, chaque élément de la `List<T>` est transmis à la méthode pointée par l'objet `Predicate<T>`==. **==L'implémentation de cette méthode effectuera des calculs pour déterminer si les données entrantes correspondent aux critères requis et renverra `true` ou `false`==**. **Si cette méthode renvoie `true`, l'élément sera ajouté à la nouvelle `List<T>` représentant le sous-ensemble** (vous suivez ?).

Avant de voir comment les expressions lambda peuvent simplifier l'utilisation de `FindAll()`, résolvons le problème en notation longue, en utilisant directement les objets délégués. Ajoutez une méthode (nommée `TraditionalDelegateSyntax()`) dans votre fichier *Program.cs* qui interagit avec le type `System.Predicate<T>` pour trouver les nombres pairs dans une `List<T>` d'entiers.

```cs
using LambdaExpressions;

Console.Title = "Fun with Lambdas";
Console.WriteLine("***** Fun with Lambdas *****\n");

TraditionalDelegateSyntax();

Console.ReadLine();

void TraditionalDelegateSyntax()
{
    // Crée une liste d'entiers
    List<int> list = new List<int>();
    list.AddRange(new int[] { 20, 1, 4, 8, 9, 44 });

    // Appelle FindAll() en utilisant
    // la syntaxe de délégué traditionnel. (abonnement)
    Predicate<int> callback = isEvenNumber;
    List<int> evenNumbers = list.FindAll(callback);

    Console.WriteLine("Here are your even numbers:");
    foreach (int evenNumber in evenNumbers)
    {
        Console.Write($"{evenNumber}\t");
    }
    Console.WriteLine();
}

// Cible pour le délégué Predicate<T>
bool isEvenNumber(int i)
{
    // Est-ce un nombre paire ?
    return (i % 2) == 0;
}
```

Ici, vous avez une méthode (`IsEvenNumber()`) qui vérifie si le paramètre entier entrant est pair ou impair grâce à l'opérateur modulo C#, `%`. Si vous exécutez votre application, vous verrez les nombres $20$, $4$, $8$ et $44$ s'afficher dans la console.

**Bien que cette approche classique de l'utilisation des délégués fonctionne comme prévu, la méthode `IsEvenNumber()` n'est appelée que dans certains cas, notamment lors de l'appel à `FindAll()`, ce qui vous oblige à définir une méthode complète**. ==Vous pourriez en faire une fonction locale, mais l'utilisation d'une méthode anonyme simplifierait considérablement votre code==. Voici un exemple de la nouvelle méthode du fichier *Program.cs* :

```cs
void AnonymousMethodSyntax()
{
    // Crée une liste d'entiers
    List<int> list = new List<int>();
    list.AddRange(new int[] { 20, 1, 4, 8, 9, 44 });

    // Maintenant, on utilise une méthode anonyme.
    List<int> evenNumbers = list.FindAll(
        delegate(int i)
        {
            return (i % 2) == 0;
        }
    );

    Console.WriteLine("Here are your even numbers:");
    foreach (int evenNumber in evenNumbers)
    {
        Console.Write($"{evenNumber}\t");
    }
    Console.WriteLine();
}
```

Dans ce cas, **plutôt que de créer directement un objet délégué `Predicate<T>` puis d'écrire une méthode autonome, vous pouvez intégrer une méthode de manière anonyme**. ==Bien que cela représente un pas dans la bonne direction, vous devez toujours utiliser le mot-clé `delegate` (ou un `Predicate<T>` fortement typé) et vous devez vous assurer que la liste des paramètres correspond parfaitement.==

```cs
List<int> evenNumbers = list.FindAll(
	delegate(int i)
	{
		return (i % 2) == 0;
	}
);
```

Les *expressions lambda* permettent de simplifier encore davantage l'appel à `FindAll()`. **Lorsque vous utilisez la syntaxe lambda, aucune trace de l'objet délégué sous-jacent n'est conservée**. Prenons l'exemple de la nouvelle méthode suivante dans le fichier *Program.cs* :

```cs
void LambdaExpressionSyntax()
{
    // Crée une liste d'entiers
    List<int> list = new List<int>();
    list.AddRange(new int[] { 20, 1, 4, 8, 9, 44 });

    // Maintenant, on utilise une expression lambda.
    List<int> evenNumbers = list.FindAll(i => (i % 2) == 0);

    Console.WriteLine("Here are your even numbers:");
    foreach (int evenNumber in evenNumbers)
    {
        Console.Write($"{evenNumber}\t");
    }
    Console.WriteLine();
}
```

Dans ce cas, **notez l'instruction de code plutôt étrange passée à la méthode `FindAll()`, qui est en fait une expression lambda**. Dans cette version de l'exemple, ==il n'y a aucune trace du délégué `Predicate<T>`== (ni du mot-clé `delegate`, d'ailleurs). ==Vous avez uniquement spécifié l'expression lambda.==

```cs
i => (i % 2) == 0;
```

Avant d'analyser cette syntaxe, **il est important de comprendre que les expressions lambda peuvent être utilisées partout où vous auriez utilisé une méthode anonyme ou un délégué fortement typé** (généralement avec beaucoup moins de frappes). ==En interne, le compilateur C# traduit l'expression en une méthode anonyme standard utilisant le type délégué `Predicate<T>`== (ce qui peut être vérifié avec *ildasm.exe* ou *reflector.exe*). Plus précisément, l'instruction de code suivante :

```cs
// Cette expression lambda...
List<int> evenNumbers = list.FindAll(i => (i % 2) == 0);
```

est compilé en le code C# approximatif suivant :

```cs
// ...devient cette méthode anonyme suivante
List<int> evenNumbers = list.FindAll(delegate (int i)
{
	return (i % 2) == 0;
});
```

>[!important] 
>Il est effectivement vrai qu'une expression lambda est traduit en une méthode anonymes. Cependant, les expressions lambda ne génère pas tout à fait le même code CIL que les méthodes anonymes.
>
> **ll faut à tout les coups (99.9% des cas) utiliser des expressions lambda car elle sont bien mieux optimisées**
>
>>[!example]- Les deux seules example ou les méthodes anonymes sont légitimes:
>> - Ignorer les paramètres (Discards)
>> 
>> 	La syntaxe `delegate` permet d'ignorer **tous** les paramètres d'un coup, sans même avoir à les nommer avec des discards (`_`).
>> 	
>> 	- Méthode anonyme :
>> 
>> 	```cs
>> 	// Peu importe si l'événement a 1, 2 ou 10 paramètres, ça compile.
>>	c1.AboutToBlow += delegate { Console.WriteLine("BOOM!"); };
>>	```
>>
>>	- Lambda:
>>
>>	```cs
>>	// Obligé de spécifier au moins un discard
>>	c1.AboutToBlow += (_) => Console.WriteLine("BOOM!");
>>	```
>>
>> - La lisibilité sur de très gros blocs de code
>> 
>> 	La lambda est conçue pour être "légère" (`=>`). Si tu dois écrire un bloc de code de 20 lignes à l'intérieur d'un abonnement (ce qui est déjà un signe que tu devrais probablement créer une méthode nommée), la syntaxe `delegate { ... }` peut parfois paraître plus "structurelle" et claire pour certains développeurs habitués aux anciens standards.

## Analyse d'une expression lambda

==Une expression lambda s'écrit en définissant d'abord une liste de paramètres, suivie du jeton `=>`== (le jeton C# de l'opérateur lambda, utilisé dans le *calcul lambda*), ==puis d'un ensemble d'instructions== (ou d'une seule instruction) ==qui traiteront ces arguments==. De manière générale, une expression lambda peut être comprise comme suit :

```cs
ArgumentsÀTraiter => DéclarationPourLesTraiter
```

Dans la méthode `LambdaExpressionSyntax()`, les choses se décomposent comme suit :

```cs
// "i" est notre liste de paramètres.
// "(i % 2) == 0" est l'instruction qui traite "i".
List<int> evenNumbers = list.FindAll(i => (i % 2) == 0);
```

**Les paramètres d'une expression lambda peuvent être typés explicitement ou implicitement**. ==Actuellement, le type de données sous-jacent représentant le paramètre `i`== (un entier) ==est déterminé implicitement==. **Le compilateur peut déduire que `i` est un entier en fonction du contexte de l'expression lambda globale et du délégué sous-jacent**. Cependant, ***==il est également possible de définir explicitement le type de chaque paramètre de l'expression en encadrant le type de données et le nom de la variable par des parenthèses==***, comme suit :

```cs
// Maintenant, on déclare explicitement le type de paramètre.
List<int> evenNumbers = list.FindAll((int i) => (i % 2) == 0);
```

Comme vous l'avez constaté, **si une expression lambda possède un seul paramètre implicitement typé, les parenthèses peuvent être omises dans la liste des paramètres**. ==Si vous souhaitez une utilisation cohérente des paramètres lambda, vous pouvez toujours placer la liste des paramètres entre parenthèses==, ce qui donne l'expression suivante :

```cs
List<int> evenNumbers = list.FindAll((i) => (i % 2) == 0);
```

Enfin, ==notez que l'expression n'est actuellement pas placée entre parenthèses (vous avez bien sûr placé l'instruction modulo entre parenthèses pour garantir son exécution avant le test d'égalité)==. **Les expressions lambda permettent d'encapsuler l'instruction** comme suit :

```cs
// Maintenant, encapsulez également l'expression.
List<int> evenNumbers = list.FindAll((i) => ((i % 2) == 0));
```

Maintenant que vous avez vu les différentes manières de construire une expression lambda, ==comment pouvez-vous lire cette instruction lambda en termes compréhensibles== ? En laissant de côté les mathématiques brutes, l’explication suivante répond à ce besoin :

```cs
// Ma liste de paramètres (ici, un entier unique nommé i)
// sera traitée par l'expression (i % 2) == 0.
List<int> evenNumbers = list.FindAll((i) => ((i % 2) == 0));
```

## Traitement des arguments dans plusieurs instructions

La première expression lambda était une instruction unique qui s'évaluait finalement en un booléen. Cependant, **comme vous le savez, de nombreuses cibles de délégués doivent exécuter plusieurs instructions de code**. ==C'est pourquoi C# vous permet de construire des expressions lambda contenant plusieurs instructions en spécifiant un bloc de code à l'aide des accolades standard==. Considérez l'exemple de mise à jour suivant de la méthode `LambdaExpressionSyntax()` :

```cs
void LambdaExpressionSyntax()
{
    // Crée une liste d'entiers
    List<int> list = new List<int>();
    list.AddRange(new int[] { 20, 1, 4, 8, 9, 44 });

    // Maintenant, on traite chaque argument dans un groupe de 
    // plusieurs instructions.
    List<int> evenNumbers = list.FindAll(
        (i) =>
        {
            Console.WriteLine($"Value of I is currently {i}");
            bool isEven = (i % 2) == 0;
            return isEven;
        }
    );

    Console.WriteLine("Here are your even numbers:");
    foreach (int evenNumber in evenNumbers)
    {
        Console.Write($"{evenNumber}\t");
    }
    Console.WriteLine();
}
```

**Dans ce cas, la liste de paramètres** (encore une fois, un seul entier nommé `i`) **est traitée par un ensemble d'instructions de code**. ==Outre les appels à `Console.WriteLine()`, l'instruction modulo a été divisée en deux instructions de code pour une meilleure lisibilité==. En supposant que chacune des méthodes que vous avez examinées dans cette section soit appelée depuis vos instructions de niveau supérieur :

```cs
Console.WriteLine("***** Fun with Lambdas *****\n");

TraditionalDelegateSyntax();
AnonymousMethodSyntax();
LambdaExpressionSyntax();

Console.ReadLine();
```

Vous obtiendrez le résultat suivant :

```
***** Fun with Lambdas *****

Here are your even numbers:
20      4       8       44
Here are your even numbers:
20      4       8       44
Value of I is currently 20
Value of I is currently 1
Value of I is currently 4
Value of I is currently 8
Value of I is currently 9
Value of I is currently 44
Here are your even numbers:
20      4       8       44
```


## Expressions lambda avec plusieurs (ou zéro) paramètres

Les expressions lambda vues jusqu'à présent dans ce chapitre traitaient un seul paramètre. Cependant, ce n'est pas une obligation, car **une expression lambda peut traiter plusieurs arguments (ou aucun)**. Pour illustrer le premier cas, celui des arguments multiples, ajoutez l'exemple suivant du type `SimpleMath` :

```cs
namespace LambdaExpressions;

public class Simplemath
{
    public delegate void MathMessage(string msg, int result);
    private MathMessage _mmDelegate;

    public void SetMathHandler(MathMessage target)
    {
        _mmDelegate = target;
    }

    public void Add(int x, int y)
    {
        _mmDelegate?.Invoke("Adding has completed!", x + y);
    }
}
```

**Notez que le type délégué MathMessage attend deux paramètres**. Pour les représenter sous forme d'expression lambda, le code pourrait s'écrire comme suit :

```cs
// S'enregistre auprès du délégué en tant qu'expression lambda.
Simplemath m = new Simplemath();
m.SetMathHandler(
    (msg, result) =>
    {
        Console.WriteLine($"Message: {msg}; Result: {result}");
    }
);

// Ceci exécutera l'expression lambda
m.Add(10, 10);

Console.ReadLine();
```

**Ici, vous tirez parti de l'inférence de type, car les deux paramètres n'ont pas été fortement typés par souci de simplicité**. Vous pouvez toutefois appeler `SetMathHandler()` comme suit :

```cs
Simplemath m = new Simplemath();
m.SetMathHandler(
    (string msg, int result) =>
    {
        Console.WriteLine($"Message: {msg}; Result: {result}");
    }
);
```

Enfin, ==si vous utilisez une expression lambda pour interagir avec un délégué ne prenant aucun paramètre, vous pouvez le faire en fournissant une paire de parenthèses vides comme paramètre==. Ainsi, en supposant que vous ayez défini le type de délégué suivant :

```cs
public delegate string VerySimpleDelegate();
```

```cs
// Affiche "Enjoy your string!" dans la console.
VerySimpleDelegate d = new VerySimpleDelegate(() =>
{
    return "Enjoy your string!";
});
Console.WriteLine(d());
```

En utilisant la nouvelle syntaxe d'expression, la ligne précédente peut s'écrire ainsi :

```cs
VerySimpleDelegate d2 = new VerySimpleDelegate(() => "Enjoy you string!");
```

ce qui peut également se résumer ainsi :

```cs
VerySimpleDelegate d3 = () => "Enjoy your string!";
```

## Utilisation du mot clé `static` avec les expressions lambda (Nouveauté C# 9.0)

**Les expressions lambda étant un raccourci pour les délégués, il est compréhensible qu'elles prennent également en charge le mot-clé `static` (avec C# 9.0) ainsi que les instructions `discard` (traitées dans la section suivante)**. Ajoutez ce qui suit à vos instructions de niveau supérieur :

```cs
Func<int, int, bool> DoWork = (x, y) =>
{
    outerVariable++;
    return true;
};

DoWork(3, 4);
Console.WriteLine($"Outer variable now = {outerVariable}");
```

Lorsque ce code est exécuté, il produit le résultat suivant :

```
***** Fun with Lambdas *****

...
Outer variable now = 1
```

**Si vous définissez la lambda comme statique, vous obtiendrez une erreur de compilation car l'expression tente de modifier une variable déclarée dans une portée extérieure**.

```cs
var outerVariable = 0;

Func<int, int, bool> DoWork = static (x, y) =>
{
    // Erreur de compilation vu que cela accède 
    // à une variable en dehors de la portée.
    // outerVariable++;
    return true;
};
```

### Les avantages de cette syntaxe

- **Performance (Allocation Zéro)** : Comme elle ne capture rien, le compilateur peut transformer la lambda en une méthode statique pure. Cela permet souvent au compilateur de réutiliser la même instance de délégué (caching), évitant ainsi des allocations inutiles sur le tas (Heap).
- **Sécurité (Prévention des fuites)** : Elle empêche la capture accidentelle d'objets volumineux ou de `this` (l'instance de classe actuelle), ce qui pourrait maintenir des objets en mémoire et causer des **Memory Leaks**.
- **Intention claire** : Elle indique explicitement aux autres développeurs : "Cette fonction est pure et ne dépend pas de l'état de l'objet".

>[!warning] Ce qui est INTERDIT (Erreurs de compilation)
>
>Une static lambda ne peut pas accéder à :
>
>- `this` ou `base`.
>- Des variables locales définies avant la lambda.
>- Des champs ou propriétés d'instance de la classe.
>```cs
>int bonus = 10;
>// ❌ ERREUR : Impossible de capturer 'bonus' dans une static lambda
>Func<int, int> calc = static x => x + bonus; 
>```
>>[!success] Ce qui reste AUTORISÉ
>>
>>- Accéder à des membres **statiques** de la classe.
>>- Accéder à des constantes (`const`).

>[!tip] Les cas d'usage idéal sont les événements et l'API LINQ

## Ignorer des valeurs (discards) avec les expressions lambda (Nouveauté C# 9.0)

**Comme pour les délégués** (et C# 9.0), **les variables d'entrée d'une expression lambda peuvent être remplacées par des valeurs ignorées si elles sont inutiles**. ==La même restriction s'applique==. Le caractère de soulignement (`_`) étant un identificateur de variable valide dans les versions précédentes de C#, ***==il doit y avoir au moins deux valeurs ignorées lors de l'utilisation de l'expression lambda==***.

```cs
var outerVariable = 0;

Func<int, int, bool> DoWork = (x,y) =>
{
	outerVariable++
	return true;
}
DoWork(_,_);
Console.WriteLine($"Outer variable now = {outerVariable}");
```

## Adaptation de l'exemple *CarEvents* à l'aide d'expressions lambda

Étant donné que l'objectif principal des expressions lambda est de fournir une manière claire et concise de définir une méthode anonyme (et donc indirectement de simplifier l'utilisation des délégués), adaptons le projet *CarEventArgs* créé précédemment dans ce chapitre. Voici une version simplifiée du fichier *Program.cs* de ce projet, qui utilise la syntaxe des expressions lambda (plutôt que les délégués bruts) pour intercepter chaque événement envoyé par l'objet `Car` :

```cs
using CarEventsWithLambdas;

Console.Title = "More fun with Lambdas";
Console.WriteLine("***** More fun with Lambdas *****\n");

// Crée un objet Car comme d'habitude.
Car c1 = new Car("SlugBug", 100, 10);

// Accroche (abonne) aux événements avec des lambdas!
c1.AboutToBlow += (sender, e) => Console.WriteLine(e.msg);
c1.Exploded += (sender, e) => Console.WriteLine(e.msg);

// Accélère (cela va généré des événements).
Console.WriteLine("\n***** Speeding up *****");
for (int i = 0; i < 6; i++)
{
    c1.Accelerate(20);
}

Console.ReadLine();
```

## Expressions lambda et membres à corps d'expression (MaJ C# 7.0)

Maintenant que vous comprenez les expressions lambda et leur fonctionnement, ==le fonctionnement interne des membres à corps d'expression devrait être beaucoup plus clair==. Comme mentionné au [[Chapitre 4#Comprendre les membres à corps d'expression|Chapitre 4]], **depuis C# 6, il est possible d'utiliser l'opérateur `=>` pour simplifier l'implémentation des membres**. Plus précisément, **==si vous avez une méthode ou une propriété==** (en plus d'un opérateur personnalisé ou d'une routine de conversion ; voir le [[Chapitre 11#Comprendre les conversions de types personnalisés|Chapitre 11]]) **==qui ne comporte qu'une seule ligne de code, vous n'êtes pas tenu de définir une portée avec des accolades==**. **Vous pouvez alors utiliser l'opérateur lambda et écrire un membre à corps d'expression**. ==En C# 7, vous pouvez également utiliser cette syntaxe pour les constructeurs de classe, les finaliseurs== (traités au [[Chapitre 9#Création d'objets finalisables|Chapitre 9]]) ==et les accesseurs `get` et `set` sur les membres de propriété.==

**Attention cependant, cette nouvelle syntaxe simplifiée peut être utilisée partout, même lorsque votre code n'a aucun lien avec les délégués ou les événements**. Par exemple, si vous deviez créer une classe simple pour additionner deux nombres, vous pourriez écrire ce qui suit :

```cs
class SimpleMath
{
	public int Add(int x, int y)
	{
		return x + y;
	}
	public void PrintSum(int x, int y)
	{
		Console.WriteLine(x + y);
	}
}
```

Vous pouvez également écrire un code comme celui-ci :

```cs
class SimpleMath
{
	public int Add(int x, int y) => x + y;
	public void PrintSum(int x, int y) => Console.WriteLine(x + y);
}
```

Idéalement, à ce stade, vous devriez comprendre le rôle global des expressions lambda et comment elles offrent une méthode fonctionnelle pour manipuler les méthodes anonymes et les types délégués. ==Bien que l'opérateur lambda (`=>`) puisse nécessiter un temps d'adaptation, gardez toujours à l'esprit qu'une expression lambda peut se décomposer en l'équation simple suivante== :

```cs
ArgumentsÀTraiter =>
{
	//InstructionsÀTraiter
}
```

==Ou, si l'on utilise l'opérateur `=>` pour implémenter un membre de type sur une seule ligne, cela donnerait quelque chose comme ceci :==

```cs
MembreDeType => InstructionDeCodeUnique
```

**Il convient de souligner que le modèle de programmation LINQ utilise également beaucoup les expressions lambda pour simplifier votre code**. Vous étudierez LINQ à partir du [[Chapitre 13|Chapitre 13]].

# Résumé du chapitre

Dans ce chapitre, ==vous avez examiné plusieurs manières dont plusieurs objets peuvent participer à une conversation bidirectionnelle==. **Vous avez d'abord étudié le mot-clé `delegate` en C#, utilisé pour construire indirectement une classe dérivée de `System.MulticastDelegate`**. Comme vous l'avez vu, **un objet délégué conserve la méthode à appeler lorsqu'il reçoit l'instruction de le faire**.

==Vous avez ensuite examiné le mot-clé `event` en C#, qui, utilisé conjointement avec un type délégué, peut simplifier l'envoi de vos notifications d'événements aux appelants en attente==. Comme le montre le code CIL résultant, **le modèle d'événements .NET correspond à des appels masqués sur les types `System.Delegate`/`System.MulticastDelegate`**. **Dans ce contexte, le mot-clé `event` en C# est purement optionnel, car il vous permet simplement de gagner du temps de saisie**. De plus, ==vous avez constaté que l'opérateur conditionnel nulle (`?.`) de C# 6.0 simplifie la manière de déclencher des événements en toute sécurité vers toute partie intéressée.==

Ce chapitre a également exploré une fonctionnalité du langage C# appelée *méthodes anonymes*. **Grâce à cette construction syntaxique, vous pouvez associer directement un bloc d'instructions de code à un événement donné**. Comme vous l'avez vu, ==les méthodes anonymes peuvent ignorer les paramètres transmis par l'événement et accéder aux « variables externes » de la méthode qui les définit==. **Vous avez également examiné une méthode simplifiée pour enregistrer des événements à l'aide de la conversion de groupes de méthodes.**

***==Enfin, vous avez abordé l'opérateur lambda C#, `=>`==***. Comme illustré, **cette syntaxe est une notation abrégée (et optimisée) idéale pour écrire des méthodes anonymes, permettant de transmettre une pile d'arguments à un groupe d'instructions pour traitement**. ==Toute méthode de la plateforme .NET prenant un objet délégué comme argument peut être remplacée par une expression lambda correspondante, ce qui simplifie généralement considérablement votre code==.

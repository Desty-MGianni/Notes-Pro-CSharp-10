---
title: "Chapitre 7: Comprendre la Gestion Structurée des Exceptions"
publish: true
---

# <big><big><big><b><font color =green>Comprendre la Gestion Structurée des Exceptions</font></b></big></big></big>

Dans ce chapitre, vous apprendrez à ==gérer les anomalies d'exécution dans votre code C# grâce à la gestion structurée des exceptions==. Vous découvrirez les mots-clés C# permettant de gérer ces anomalies (`try`, `catch`, `throw`, `finally`, `when`), ainsi que la **distinction entre les exceptions de niveau application et les exceptions de niveau système, et le rôle de la classe de base `System.Exception`**. Cette discussion vous mènera à la ==création d'exceptions personnalisées et, enfin, à un aperçu des outils de déboggage de Visual Studio axés sur les exceptions==.

# Ode aux erreurs, aux bugs et aux exceptions

Malgré ce que notre ego (parfois démesuré) peut nous faire croire, aucun programmeur n'est parfait. **Écrire des logiciels est une tâche complexe, et compte tenu de cette complexité, il est fréquent que même les meilleurs logiciels soient livrés avec *divers problèmes***. ==Parfois, le problème est dû à un code défectueux== (comme un dépassement de capacité d'un tableau). ==D'autres fois, il est causé par une saisie utilisateur erronée qui n'a pas été prise en compte dans le code source de l'application== (par exemple, un champ de saisie de numéro de téléphone auquel on a attribué la valeur `Chucky`). Or, **quelle que soit la cause du problème, le résultat est le même : l'application ne fonctionne pas comme prévu**. Afin de contextualiser la discussion à venir sur la gestion structurée des exceptions, permettez-moi de définir trois termes couramment utilisés en lien avec les anomalies.

- Les *bugs* : **Il s’agit, en termes simples, d’erreurs commises par le programmeur**. Par exemple, supposons que vous programmiez en C++ non managé. Si vous oubliez de libérer la mémoire allouée dynamiquement, ce qui entraîne une fuite de mémoire, vous avez un bogue.

- *Erreurs utilisateur* : **Les erreurs utilisateur, quant à elles, sont généralement causées par la personne qui exécute votre application, plutôt que par ses créateurs**. Par exemple, un utilisateur final qui saisit une chaîne de caractères mal formée dans une zone de texte peut très bien générer une erreur si vous ne gérez pas cette entrée incorrecte dans votre code.

- *Exceptions* : **Les exceptions sont généralement considérées comme des anomalies d’exécution difficiles, voire impossibles, à prendre en compte lors de la programmation de votre application**. Parmi les exceptions possibles, on peut citer la tentative de connexion à une base de données qui n’existe plus, l’ouverture d’un fichier XML corrompu ou la tentative de connexion à une machine actuellement hors ligne. Dans chacun de ces cas, le programmeur (ou l’utilisateur final) a peu de contrôle sur ces circonstances "exceptionnelles".

Compte tenu de ces définitions, il devrait être clair que **la gestion structurée des exceptions .NET est une technique permettant de gérer les exceptions d'exécution**. Cependant, ==même pour les bugs et les erreurs utilisateur qui vous ont échappé, l'environnement d'exécution générera souvent une exception correspondante qui identifie le problème en question==. À titre d'exemple, ==les bibliothèques de classes de base .NET définissent de nombreuses exceptions, telles que `FormatException`, `IndexOutOfRangeException`, `FileNotFoundException`, `ArgumentOutOfRangeException`, etc.

**Dans la nomenclature .NET, une exception prend en compte les bugs, les entrées utilisateur incorrectes et les erreurs d'exécution, même si les programmeurs peuvent considérer chacun de ces éléments comme un problème distinct**. Cependant, ==avant d'aller plus loin, formalisons le rôle de la gestion structurée des exceptions et voyons en quoi elle diffère des techniques traditionnelles de gestion des erreurs==.

>[!note]
>Afin de rendre les exemples de code utilisés dans ce livre aussi clairs que possible, je ne traiterai pas toutes les exceptions possibles qui pourraient être levées par une méthode donnée dans les bibliothèques de classes de base. Dans vos projets de production, vous devriez, bien sûr, faire un usage intensif des techniques présentées dans ce chapitre.

# Le rôle de la gestion des exceptions .NET

**Avant .NET, la gestion des erreurs sous Windows était un ensemble disparate de techniques**. De nombreux programmeurs implémentaient leur propre logique de gestion des erreurs au sein d'une application donnée. ==Pae exemple, une équipe de développement pouvait définir un ensemble de constantes numériques représentant des erreurs connues et les utiliser comme valeurs de retour de méthodes. À titre d'exemple, considérons le code C partiel suivant== :

```c
/* Un mécanisme de gestion des erreurs très typique du C. */
#define E_FILENOTFOUND 1000

int UseFileSystem()
{
	// Supposons qu'il se passe quelque chose dans cette fonction
	// qui provoque la valeur de retour suivante.
	return E_FILENOTFOUND;
}

void main()
{
	int retVal = UseFileSystem();
	(retVal == E_FILENOTFOUND)
	printf("Cannot find file...");
}
```

Cette approche est loin d'être idéale, car **la constante `E_FILENOTFOUND` n'est guère plus qu'une valeur numérique et est loin d'être une aide précieuse pour résoudre le problème**. Idéalement, ==il faudrait regrouper le nom de l'erreur, un message descriptif et d'autres informations utiles sur cette erreur dans un seul et même élément bien défini== (ce qui est précisément le cas lors de la gestion structurée des exceptions). Outre les techniques spécifiques utilisées par les développeurs, ==l'API Windows définit des centaines de codes d'erreur accessibles via des directives== `#define`, des `HRESULT` et de nombreuses variantes du booléen simple (`bool`, `BOOL`, `VARIANT_BOOL`, etc.).

**Le problème évident de ces anciennes techniques réside dans leur manque flagrant d'homogénéité**. ==Chaque approche est plus ou moins adaptée à une technologie, un langage, voire un projet donnés==. Pour mettre fin à cette situation, **la plateforme .NET propose une technique standard pour envoyer et intercepter les erreurs d'exécution : la gestion structurée des exceptions**. L'avantage de cette approche est que ==les développeurs disposent désormais d'une approche unifiée de la gestion des erreurs, commune à tous les langages ciblant la plateforme .NET==. Ainsi, la manière dont un programmeur C# gère les erreurs est syntactiquement similaire à celle d'un programmeur VB, ou d'un programmeur C++ utilisant C++/CLI.

De plus, **la syntaxe utilisée pour lever et intercepter les exceptions entre les assemblages et les machines est identique**. Par exemple, ==si vous utilisez C# pour créer un service RESTful ASP.NET Core, vous pouvez lever une erreur JSON vers un appelant distant, en utilisant les mêmes mots-clés qui vous permettent de lever une exception entre les méthodes d'une même application==. 

Un autre avantage des exceptions .NET est que, **plutôt que de recevoir une valeur numérique obscure, les exceptions sont des objets contenant une description lisible du problème, ainsi qu'un aperçu détaillé de la pile d'appels (*stacktrace*) qui a déclenché l'exception**. De plus, ==vous pouvez fournir à l'utilisateur final des liens d'aide le dirigeant vers une URL fournissant des informations sur l'erreur, ainsi que des données personnalisées définies par le programmeur==.

## Les éléments constitutifs de la gestion des exceptions .NET

**La programmation avec une gestion structurée des exceptions implique l'utilisation de quatre entités interdépendantes** :

- Un type de classe représentant les détails de l'exception
- Un membre qui *lève* une instance de la classe d'exception vers l'appelant dans les circonstances appropriées
- Un bloc de code côté appelant qui invoque le membre susceptible de lever une exception
- Un bloc de code côté appelant qui traitera (ou *interceptera*) l'exception, le cas échéant

Le langage de programmation C# offre **cinq mots-clés** (`try`, `catch`, `throw`, `finally` et `when`) **qui permettent de lever et de gérer des exceptions**. ==L'objet représentant le problème en question est une classe étendant `System.Exception` (ou une classe dérivée)==. Partant de ce constat, examinons le rôle de cette classe de base centrée sur les exceptions.

## La classe de base `System.Exception`

**Toutes les exceptions dérivent de la classe de base `System.Exception`, qui dérive elle-même de `System.Object`**. Voici la définition principale de cette classe (notez que certains de ces membres sont virtuels et peuvent donc être redéfinis par les classes dérivées) :

```cs
public class Exception : ISerializable
{
	// Constructeur public
	public Exception(string message, Exception innerException);
	public Exception(string message);
	public Exception();
	
	...

	// Methodes
	public virtual Exception GetBaseException();
	public virtual void GetObjectData(SerializationInfo info,
	  StreamingContext context);

	// Propriétés
	public virtual IDictionary Data { get; }
	public virtual string HelpLink { get; set; }
	public int HResult {get;set;}
	public Exception InnerException { get; }
	public virtual string Message { get; }
	public virtual string Source { get; set; }
	public virtual string StackTrace { get; }
	public MethodBase TargetSite { get; }
}
```

Comme vous pouvez le constater, ==de nombreuses propriétés définies par `System.Exception` sont en lecture seule==. En effet, les types dérivés fournissent généralement des valeurs par défaut pour chaque propriété. Par exemple, le message par défaut du type `IndexOutOfRangeException` est `"L’index se trouvait en dehors des limites du tableau."`

Le [[#Tableau 7-1 Membres principaux du type `System.Exception`|Tableau 7-1]] décrit les membres les plus importants de System.Exception.


##### Tableau 7-1: Membres principaux du type `System.Exception`

| Propriétés de `System.Exception` | Description                                                                                                                                                                                                                                                                                   |
| -------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Data`                           | Cette propriété en lecture seule récupère une collection de paires clé-valeur (représentée par un objet implémentant `IDictionary`) qui ==fournissent des informations supplémentaires, définies par le programmeur, sur l'exception==. **Par défaut, cette collection est vide**.            |
| `HelpLink`                       | **Cette propriété obtient ou définit une URL vers un fichier d'aide ou un site web décrivant l'erreur en detail**.                                                                                                                                                                            |
| `InnerException`                 | **Cette propriété en lecture seule permet d'obtenir des informations sur les exceptions précédentes ayant provoqué l'exception actuelle**. Ces exceptions précédentes sont enregistrées en les passant au constructeur de l'exception la plus récente.                                        |
| `Message`                        | **Cette propriété en lecture seule renvoie la description textuelle d'une erreur donnée**. ==Le message d'erreur lui-même est défini comme paramètre du constructeur==.                                                                                                                       |
| `Source`                         | **Cette propriété permet d'obtenir ou de définir le nom de l'assembly, ou de l'objet, qui a levé l'exception actuelle**.                                                                                                                                                                      |
| `StackTrace`                     | **Cette propriété en lecture seule contient une chaîne de caractères identifiant la séquence d'appels ayant déclenché l'exception**. Comme vous pouvez l'imaginer, cette propriété est ==utile lors du déboggage ou si vous souhaitez consigner l'erreur dans un journal d'erreurs externe==. |
| `TargetSite`                     | **Cette propriété en lecture seule renvoie un objet `MethodBase`, qui décrit de nombreux détails sur la méthode ayant levé l'exception** (l'appel à `ToString()` permettra d'identifier la méthode par son nom).                                                                              |

# L'exemple le plus simple possible

Pour illustrer l'utilité de la gestion structurée des exceptions, vous devez créer une classe qui lèvera une exception dans des circonstances appropriées (ou, pourrait-on dire, *exceptionnelles*). Supposons que vous ayez créé un nouveau projet d'application console C# (nommé *SimpleException*) qui définit deux types de classes (`Car` et `Radio`) associé par la relation "possède un". Le type `Radio` définit une méthode unique qui active ou désactive la radio.

```cs
namespace SimpleException;

class Radio
{
    public void TurnOn(bool on) =>
        Console.WriteLine(on ? "Jamming..." : "Quiet time...");
}
```

Outre l'utilisation de la classe `Radio` via la délégation, la classe `Car` (présentée ci-après) est définie de telle sorte que si l'utilisateur accélère un objet `Car` au-delà d'une vitesse maximale prédéfinie (spécifiée par une constante nommée `MaxSpeed`), son moteur explose, rendant l'objet `Car` inutilisable (état capturé par une variable membre booléenne privée nommée `_carIsDead`).

De plus, le type `Car` possède quelques propriétés pour représenter la vitesse actuelle et un "surnom" fourni par l'utilisateur, ainsi que divers constructeurs pour initialiser un nouvel objet `Car`. Voici la définition complète (avec commentaires) :

```cs
namespace SimpleException;

class Car
{
    // Constante pour la vitesse maximale.
    public const int MaxSpeed = 100;

    // Propriété de la voiture
    public int CurrentSpeed { get; set; } = 0;
    public string PetName { get; set; } = "";

    // Est-ce que la voiture est toujours opérationnelle?
    private bool _carIsDead;

    // Une voiture possède une radio
    private readonly Radio _theMusicBox = new Radio();

    // Constructeurs
    public Car() { }

    public Car(string name, int speed)
    {
        CurrentSpeed = speed;
        PetName = name;
    }

    public void CrankTunes(bool state)
    {
        // Déléguer la requête à l'objet interne.
        _theMusicBox.TurnOn(state);
    }

    // Vérifier si la voiture a surchauffé.
    // Écrite de manière plus moderne par rapport au livre.
    public void Accelerate(int delta)
    {
        if (_carIsDead)
        {
            Console.WriteLine($"{PetName} is out of order...");
            return;
        }

        CurrentSpeed += delta;
        if (CurrentSpeed > MaxSpeed)
        {
            Console.WriteLine($"{PetName} has overheated!");
            CurrentSpeed = 0;
            _carIsDead = true;
        }
        else
        {
            Console.WriteLine($"=> CurrentSpeed = {CurrentSpeed}");
        }
    }
}
```

Ensuite, mettez à jour votre fichier `Program.cs` pour forcer un objet `Car` à dépasser la vitesse maximale prédéfinie (définie à $100$ dans la classe `Car`), comme indiqué ici :

```cs
using System.Collections;
using SimpleException;

Console.Title = "Simple Exception Example";
Console.WriteLine("**** Simple Exception Example ****\n");

Console.WriteLine("=> Creating a car and stepping on it!");

Car myCar = new("Zippy", 20);
myCar.CrankTunes(true);
for (int i = 0; i < 10; i++)
{
    myCar.Accelerate(10);
}
Console.ReadLine();
```

Quand on exécute le code, vous verrez le résultat suivant :

```
**** Simple Exception Example ****

=> Creating a car and stepping on it!
Jamming...
=> CurrentSpeed = 30
=> CurrentSpeed = 40
=> CurrentSpeed = 50
=> CurrentSpeed = 60
=> CurrentSpeed = 70
=> CurrentSpeed = 80
=> CurrentSpeed = 90
=> CurrentSpeed = 100
Zippy has overheated!
Zippy is out of order...
```

## Lever une exception générale

Maintenant que vous disposez d'une classe `Car` fonctionnelle, je vais vous montrer la méthode la plus simple pour lever une exception. ==L'implémentation actuelle de la méthode `Accelerate()` affiche simplement un message d'erreur si l'appelant tente d'accélérer `Car` au-delà de sa limite supérieure.

Pour modifier cette méthode afin qu'elle lève une exception si l'utilisateur tente d'accélérer le véhicule après qu'il a atteint sa limite supérieure, **vous devez créer et configurer une nouvelle instance de la classe `System.Exception`, en définissant la valeur de la propriété `Message`** (en lecture seule) **via le constructeur de la classe**. ==Lorsque vous souhaitez renvoyer l'objet exception à l'appelant, utilisez l'instruction `throw()` de C#==. Voici la mise à jour du code correspondante pour la méthode `Accelerate()` :

```cs
// Cette fois, lève une exception si l'utilisateur
// accelère au dela de MaxSpeed
public void Accelerate(int delta)
{
    if (_carIsDead)
    {
        Console.WriteLine($"{PetName} is out of order...");
        return;
    }
    
    CurrentSpeed += delta;
    
    if (CurrentSpeed > MaxSpeed)
    {
        CurrentSpeed = 0;
        _carIsDead = true;
        // utilise le mot clé "throw" pour lever une exception
        throw new Exception($"{PetName} has overheated!");
    }
    
    Console.WriteLine($"=> CurrentSpeed = {CurrentSpeed}");
}
```

Avant d'examiner comment l'appelant intercepterait cette exception, penchons-nous sur quelques points importants. Premièrement, **lorsque vous levez une exception, il vous appartient toujours de déterminer précisément ce qui constitue l'erreur en question et à quel moment une exception doit être levée**. Ici, ==vous partez du principe que si le programme tente d'augmenter la vitesse d'un objet `Car` au-delà de sa valeur maximale, un objet `System.Exception` devrait être levé pour indiquer que la méthode `Accelerate()` ne peut pas continuer== (ce qui peut être une hypothèse valide ou non ; il vous appartiendra d'en juger en fonction de l'application que vous développez). 

Autrement, **vous pouvez implémenter la méthode `Accelerate()` pour une récupération automatique, sans avoir besoin de lever une exception**. En règle générale, ==les exceptions ne devraient être levées que lorsqu'une condition critique est atteinte== (par exemple, un fichier nécessaire introuvable, une connexion impossible à une base de données, etc.) ==et non utilisées comme mécanisme de contrôle du flux logique==. **Déterminer précisément ce qui justifie le déclenchement d'une exception est un aspect de conception auquel vous devez toujours faire face**. Dans le cadre de cet exemple, supposons que demander à une voiture en panne d'accélérer soit une raison suffisante pour lever une exception.

Deuxièmement, **notez que le dernier bloc `else` a été supprimé de la méthode**. ==Lorsqu'une exception est levée== (soit par le framework, soit manuellement à l'aide d'une instruction `throw()`), ==le contrôle est rendu à la méthode appelante (ou par le bloc `catch` dans un bloc `try-catch`). Cela rend le dernier bloc `else` inutile==. Libre à vous de le conserver pour des raisons de lisibilité, et selon vos normes de codage.

Quoi qu'il en soit, si vous deviez relancer l'application à ce stade en utilisant la logique précédente dans les instructions de niveau supérieur, l'exception finirait par être levée. Comme le montre la sortie suivante, **le résultat de ne pas gérer cette erreur est loin d'être idéal, car vous recevez un message d'erreur détaillé suivi de l'arrêt du programme** (avec votre chemin d'accès et vos numéros de ligne spécifiques) :

```
**** Simple Exception Example ****

=> Creating a car and stepping on it!
Jamming...
=> CurrentSpeed = 30
=> CurrentSpeed = 40
=> CurrentSpeed = 50
=> CurrentSpeed = 60
=> CurrentSpeed = 70
=> CurrentSpeed = 80
=> CurrentSpeed = 90
=> CurrentSpeed = 100

Unhandled exception. System.Exception: Zippy has overheated!
	at SimpleException.Car.Accelerate(Int32 delta) in [path to file]\Car.cs:line 52
	at SimpleException.Program.Main(String[] args) in [path to file]\Program.cs:line 16
```

## Gestion des exceptions

>[!note]- Pour ceux qui découvrent C# après une expérience en Java
>Il est important de comprendre que les membres de type ne sont pas prototypés avec l'ensemble des exceptions qu'ils peuvent lever (autrement dit, .NET Core ne prend pas en charge les exceptions vérifiées). Que cela soit un avantage ou un inconvénient, vous n'êtes pas tenu de gérer toutes les exceptions levées par un membre donné.

Puisque la méthode `Accelerate()` lève désormais une exception, **l'appelant doit être prêt à la gérer le cas échéant**. ==Lorsque vous appelez une méthode susceptible de lever une exception, vous utilisez un bloc `try`/`catch`. Une fois l'objet exception intercepté, vous pouvez accéder à ses membres pour extraire les détails du problème==.

**Le traitement de ces données vous appartient**. ==Vous pouvez, par exemple, les consigner dans un fichier de rapport, les enregistrer dans le journal des événements, les envoyer par e-mail à un administrateur système ou les afficher à l'utilisateur final==. Ici, vous afficherez simplement le contenu dans la console :

```cs
using System.Collections;
using SimpleException;

Console.Title = "Simple Exception Example";
Console.WriteLine("**** Simple Exception Example ****\n");

Console.WriteLine("=> Creating a car and stepping on it!");

Car myCar = new("Zippy", 20);
myCar.CrankTunes(true);

// Accélère la voiture, dépassant la vitesse
// maximale pour lever l'exception.
try
{
    for (int i = 0; i < 10; i++)
    {
        myCar.Accelerate(10);
    }
}
catch (Exception e)
{
    // Gestion de l'exception levée.
    Console.WriteLine("\n*** Erreur! ***");
    Console.WriteLine($"Method: {e.TargetSite}");
    Console.WriteLine($"Message: {e.Message}");
    Console.WriteLine($"Source: {e.Source}");
}

// L'erreur a été gérée, le traitement
// se poursuit avec l'instruction suivante.
Console.WriteLine("\n**** Out of Exception Logic ****");
Console.ReadLine();
```

En résumé, **un bloc `try` est une section d'instructions susceptible de lever une exception lors de son exécution. Si une exception est détectée, l'exécution du programme est dirigée vers le bloc `catch` approprié. En revanche, si le code contenu dans un bloc `try` ne déclenche pas d'exception, le bloc `catch` est ignoré et tout se déroule normalement**. L'exemple de sortie suivant illustre l'exécution de ce programme :

```
**** Simple Exception Example ****

=> Creating a car and stepping on it!
Jamming...
=> CurrentSpeed = 30
=> CurrentSpeed = 40
=> CurrentSpeed = 50
=> CurrentSpeed = 60
=> CurrentSpeed = 70
=> CurrentSpeed = 80
=> CurrentSpeed = 90
=> CurrentSpeed = 100

*** Erreur! ***
Method: Void Accelerate(Int32)
Message: Zippy has overheated!
Source: SimpleException

**** Out of Exception Logic ****
```

Comme vous pouvez le constater, **une fois une exception gérée, l'application peut reprendre son exécution à partir du point situé après le bloc `catch`**. Dans certains cas, **une exception donnée peut être suffisamment critique pour justifier l'arrêt de l'application**. ==Cependant, dans de nombreux cas, la logique du gestionnaire d'exceptions garantit la poursuite de l'application== (même si certaines fonctionnalités peuvent être légèrement réduites, par exemple l'impossibilité de se connecter à une source de données distante).

## Lever une exception en tant qu'expression (Nouveauté C# 7.0)

Avant C# 7, la méthode `throw()` était une instruction, ce qui signifiait qu'une exception ne pouvait être levée que là où les instructions étaient autorisées. **Avec C# 7.0 et versions ultérieures, `throw()` est également disponible en tant qu'expression et peut être appelée partout où les expressions sont autorisées**.

Voici un exemple fourni par Gemini pour comprendre cette nouveauté:

```cs
// On vérifie le delta avant même de l'ajouter
public void Accelerate(int delta)
{
    int speedToAdd = (delta < 100) ? delta : throw new Exception("Accélération trop dangereuse !");
    CurrentSpeed += speedToAdd;
}
```

# Configuration de l'état d'une exception

==Actuellement, l'objet `System.Exception` configuré dans la méthode `Accelerate()` définit simplement une valeur exposée à la propriété `Message`== (via un paramètre du constructeur). Comme indiqué précédemment dans le tableau [[#Tableau 7-1 Membres principaux du type `System.Exception`|Tableau 7-1]], **la classe Exception fournit également plusieurs membres supplémentaires** (`TargetSite`, `StackTrace`, `HelpLink` et `Data`) **qui peuvent s'avérer utiles pour mieux caractériser le problème**. Afin d'enrichir l'exemple actuel, examinons plus en détail ces membres au cas par cas.

## Propriété `TargetSite`

**La propriété `System.Exception.TargetSite` permet d'obtenir divers détails sur la méthode qui a levé une exception donnée**. Comme illustré dans l'exemple de code précédent, l'==affichage de la valeur de `TargetSite` affichera le type de retour, le nom et les types des paramètres de la méthode ayant levé l'exception==. Cependant, **`TargetSite` ne renvoie pas une simple chaîne de caractères, mais un objet `System.Reflection.MethodBase` fortement typé. Ce type permet de recueillir de nombreux détails concernant la méthode fautive, ainsi que la classe qui la définit**. À titre d'exemple, supposons que la logique catch précédente ait été mise à jour comme suit :

```cs

...

catch (Exception e)
{
    // Gestion de l'exception levée.
    Console.WriteLine("\n*** Error! ***");
    Console.WriteLine($"Method: {e.TargetSite}");
    Console.WriteLine($"Class defining member: {e.TargetSite.DeclaringType}");
    Console.WriteLine($"Member type: {e.TargetSite.MemberType}");
    Console.WriteLine($"Message: {e.Message}");
    Console.WriteLine($"Source: {e.Source}");
}

// L'erreur a été gérée, le traitement
// se poursuit avec l'instruction suivante.
Console.WriteLine("\n**** Out of Exception Logic ****");
Console.ReadLine();
```

Cette fois-ci, ==vous utilisez la propriété `MethodBase.DeclaringType` pour déterminer le nom complet de la classe ayant généré l'erreur== (`SimpleException.Car`, dans ce cas), ==ainsi que la propriété `MemberType` de l'objet `MethodBase` pour identifier le type de membre== (propriété ou méthode) ==à l'origine de cette exception==. Dans ce cas, le bloc catch affichera le résultat suivant :

```

...
*** Error! ***
Method: Void Accelerate(Int32)
Class defining member: SimpleException.Car
Member type: Method
Message: Zippy has overheated!
Source: SimpleException
...

```

## La propriété `StackTrace`

**La propriété `System.Exception.StackTrace` permet d'identifier la séquence d'appels ayant entraîné l'exception**. Notez que ==vous ne devez jamais définir la valeur de StackTrace, car elle est établie automatiquement lors de la création de l'exception==. À titre d'exemple, supposons que vous ayez mis à jour votre bloc catch`.

```cs
catch(Exception e)
{
	...
	Console.WriteLine($"Stack: {e.StackTrace}");
}
```

Si vous exécutiez le programme, vous verriez la trace de pile suivante s'afficher dans la console (vos numéros de ligne et chemins d'accès aux fichiers peuvent différer, bien sûr) :

```
Stack: at SimpleException.Car.Accelerate(Int32 delta)
in [path to file]\car.cs:line 57 at <Program>$.<Main>$(String[] args)
in [path to file]\Program.cs:line 20
```

**La chaîne renvoyée par `StackTrace` documente la séquence d'appels ayant entraîné la levée de cette exception**. Remarquez comment ==le numéro de ligne le plus bas de cette chaîne identifie le premier appel de la séquence, tandis que le numéro de ligne le plus haut identifie l'emplacement exact du membre fautif==. De toute évidence, ces informations peuvent s'avérer très utiles lors du déboggage ou de la journalisation d'une application, car elles permettent de « suivre le flux » de l'origine de l'erreur.

## La propriété `HelpLink`

Bien que les propriétés `TargetSite` et `StackTrace` permettent aux programmeurs de comprendre une exception donnée, ces informations sont peu utiles à l'utilisateur fina Comme vous l'avez déjà vu, ==la propriété `System.Exception.Message` permet d'obtenir des informations lisibles par l'utilisateur et affichables à l'utilisateur actuel==. De plus, **la propriété `HelpLink` peut être configurée pour rediriger l'utilisateur vers une URL spécifique ou un fichier d'aide standard contenant des informations plus détaillées**. ==Par défaut, la valeur gérée par la propriété `HelpLink` est une chaîne vide==. Mettez à jour l'exception à l'aide de l'initialisation d'objet pour fournir une valeur plus pertinente. Voici les mises à jour pertinentes pour la méthode `Car.Accelerate()` :

```cs
public void Accelerate(int delta)
{
    if (_carIsDead)
    {
        Console.WriteLine($"{PetName} is out of order...");
        return;
    }

    CurrentSpeed += delta;
    if (CurrentSpeed > MaxSpeed)
    {
        CurrentSpeed = 0;
        _carIsDead = true;

        // utilise le mot clé "throw" pour lever une exception
        // retour à l'appelant.
        throw new Exception($"{PetName} has overheated!")
        {
            HelpLink = "http://www.CarsRUs.com",
        };
    }

    Console.WriteLine($"=> CurrentSpeed = {CurrentSpeed}");
}
```

La logique de gestion des intercepteurs pourrait maintenant être mise à jour pour imprimer ces informations de lien d'aide comme suit :

```cs
catch (Exception e)
{
    // Gestion de l'exception levée.
    Console.WriteLine("\n*** Error! ***");
    ...
    Console.WriteLine($"Help Link: {e.HelpLink}");
}
```

## La propriété `Data`
**La propriété `Data` de `System.Exception` permet de renseigner un objet exception avec des informations auxiliaires pertinentes** (telles qu'un horodatage). ==La propriété `Data` renvoie un objet implémentant l'interface `IDictionary`, définie dans l'espace de noms `System.Collections`==. Le [[Chapitre 8|Chapitre 8]] examine le rôle de la programmation par interfaces, ainsi que l'espace de noms `System.Collections`. Pour l'instant, retenez simplement que ==les collections de dictionnaires permettent de créer un ensemble de valeurs récupérées à l'aide d'une clé spécifique. Observez la prochaine mise à jour de la méthode `Car.Accelerate()`== :

```cs
public void Accelerate(int delta)
{
    if (_carIsDead)
    {
        Console.WriteLine($"{PetName} is out of order...");
        return;
    }

    CurrentSpeed += delta;
    if (CurrentSpeed > MaxSpeed)
    {
        CurrentSpeed = 0;
        _carIsDead = true;

        // utilise le mot clé "throw" pour lever une exception
        // retour à l'appelant.
        throw new Exception($"{PetName} has overheated!")
        {
            HelpLink = "https://www.google.com",
            Data =
            {
                { "TimeStamp", $"The car exploded at {DateTime.Now}" },
                { "Cause", "You have a lead foot" },
            },
        };
    }

    Console.WriteLine($"=> CurrentSpeed = {CurrentSpeed}");
}
```

**Pour parcourir correctement les paires clé-valeur, assurez-vous d'avoir ajouté une directive `using` pour l'espace de noms `System.Collections`, car vous utiliserez un type `DictionaryEntry` dans le fichier contenant la classe implémentant vos instructions de niveau supérieur**.

```cs
using System.Collections;
```

Ensuite, vous devez mettre à jour la logique de gestion des exceptions pour vérifier que la valeur renvoyée par la propriété `Data` n'est pas `null` (la valeur par défaut). Enfin, ==utilisez les propriétés `Key` et `Value` du type `DictionaryEntry` pour afficher les données personnalisées dans la console==.

```cs
catch (Exception e)
{
    // Gestion de l'exception levée.
    Console.WriteLine("\n*** Error! ***");
    
	...
	
    Console.WriteLine("\n-> Custom Data:");
    foreach (DictionaryEntry de in e.Data)
    {
        Console.WriteLine($"\t->{de.Key}: {de.Value}");
    }
}
```

Voici le résultat final que vous obtiendrez :

```
**** Simple Exception Example ****

=> Creating a car and stepping on it!
Jamming...
=> CurrentSpeed = 30
=> CurrentSpeed = 40
=> CurrentSpeed = 50
=> CurrentSpeed = 60
=> CurrentSpeed = 70
=> CurrentSpeed = 80
=> CurrentSpeed = 90
=> CurrentSpeed = 100

*** Error! ***
Method: Void Accelerate(Int32)
Class defining member: SimpleException.Car
Member type: Method
Message: Zippy has overheated!
Source: SimpleException
Stack:    at SimpleException.Car.Accelerate(Int32 delta) in /Volumes/GM Disk 1/Code/Livre C#/SimpleException/Car.cs:line 53
   at Program.<Main>$(String[] args) in /Volumes/GM Disk 1/Code/Livre C#/SimpleException/Program.cs:line 18
Help Link: https://www.google.com

-> Custom Data:
        ->TimeStamp: The car exploded at 4/18/2026 11:24:20 AM
        ->Cause: You have a lead foot

**** Out of Exception Logic ****
```

==La propriété `Data` est utile car elle permet d'intégrer des informations personnalisées concernant l'erreur en question, sans avoir à créer une nouvelle classe étendant la classe de base `Exception`==. Cependant, **aussi utile que soit la propriété `Data`, il est courant que les développeurs créent des classes d'exceptions fortement typées, qui gèrent les données personnalisées à l'aide de propriétés fortement typées**.

==Cette approche permet à l'appelant d'intercepter un type d'exception spécifique, plutôt que de devoir explorer une collection de données pour obtenir des informations supplémentaires==. Pour comprendre comment procéder, il est nécessaire d'**examiner la distinction entre les exceptions système et les exceptions applicative**.

# Exceptions système (`System.SystemException`)

Les bibliothèques de classes de base .NET définissent de nombreuses classes qui héritent de `System.Exception`. Par exemple, l'**espace de noms `System` définit des objets d'exception de base tels que `ArgumentOutOfRangeException`, `IndexOutOfRangeException`, `StackOverflowException`, etc**. ==D'autres espaces de noms définissent des exceptions qui reflètent le comportement de cet espace de noms. Par exemple, `System.Drawing.Printing` définit les exceptions d'impression, `System.IO` définit les exceptions d'entrée/sortie, `System.Data` définit les exceptions liées aux bases de données, et ainsi de suite==.

**Les exceptions levées par la plateforme .NET sont** (à juste titre) **appelées exceptions système**. ==Ces exceptions sont généralement considérées comme des erreurs fatales et non récupérables==. **Les exceptions système héritent directement d'une classe de base nommée `System.SystemException`, qui hérite elle-même de `System.Exception`** (qui hérite de `System.Object`).

```cs
public class SystemException : Exception
{
	// Multitude de constructeurs.
}
```

**Étant donné que le type `System.SystemException` n'ajoute aucune fonctionnalité au-delà d'un ensemble de constructeurs personnalisés, vous pourriez vous demander pourquoi `SystemException` existe**. En résumé, ==lorsqu'un type d'exception hérite de System.`SystemException`, vous pouvez déterminer que c'est le runtime .NET qui a levé l'exception, et non le code source de l'application en cours d'exécution==. Vous pouvez le vérifier très facilement à l'aide du mot-clé `is`.

```cs
// Vrai! NullReferenceException "est un" SystemException
NullReferenceException nullRefEx = new NullReferenceException();
Console.WriteLine(
    "NullReferenceException is-a SystemException? : {0}",
    nullRefEx is SystemException
);
```

# Exceptions au niveau de l'application (`System.ApplicationException`)

>[!warning] Relique du passé
>L'utilisation de `System.ApplicationException` est aujourd'hui considérée comme une **mauvaise pratique** (ou "relique" du passé) et ne doit plus être utilisée pour la création d'exceptions personnalisées.
>
>À l'origine de .NET, Microsoft avait prévu que les exceptions du système dérivent de `SystemException` et que celles des développeurs dérivent d'`ApplicationException` pour les différencier. Cependant : 
>
>- **Perte de sens** : De nombreuses exceptions du framework lui-même ont fini par dériver d'`ApplicationException` par erreur, rendant la distinction inutile.
>- **Complexité inutile** : Cela ajoute un niveau d'héritage sans apporter de fonctionnalité supplémentaire.
>- **Recommandation officielle** : Microsoft conseille désormais explicitement d'ignorer `ApplicationException` et de dériver de `Exception`
>
>Sources :
>- Gemini
>- [StackOverflow](https://stackoverflow.com/questions/5685923/what-is-applicationexception-for-in-net)
>- [Miscrosoft (guideline déja présente depuis .NET 5)](https://learn.microsoft.com/en-us/dotnet/api/system.applicationexception?view=net-5.0)
>



**Étant donné que toutes les exceptions .NET sont des types de classe, vous pouvez créer vos propres exceptions spécifiques à l'application**. Cependant, ==comme la classe de base `System.SystemException` représente les exceptions levées par l'environnement d'exécution, vous pourriez être tenté de penser qu'il faut faire dériver vos exceptions personnalisées du type `System.Exception`==. C'est possible, **mais vous pouvez également dériver de la classe `System.ApplicationException`**.

```cs
public class ApplicationException : Exception
{
	// Multitude de constructeurs.
}
```

**À l'instar de `SystemException`, `ApplicationException` ne définit aucun membre supplémentaire au-delà d'un ensemble de constructeurs**. Fonctionnellement, **le seul but de `System.ApplicationException` est d'identifier la source de l'erreur**. ==Lorsqu'une exception dérivant de `System.ApplicationException` est gérée, on peut supposer qu'elle a été levée par le code source de l'application en cours d'exécution, et non par les bibliothèques de classes de base .NET Core ou le moteur d'exécution .NET==.

## Création d'exceptions personnalisées, 1ère partie

>[!info] 
>Les exemples de codes utiliseront les guidelines "modernes" de Microsoft. De ce fait, les exceptions personnalisées dériveront de `System.Exception`.

==Bien qu'il soit toujours possible de lever des instances de `System.Exception` pour signaler une erreur d'exécution== (comme illustré dans le premier exemple), **il est parfois avantageux de créer une exception fortement typée qui représente les spécificités de votre problème actuel**. Par exemple, supposons que vous souhaitiez créer une exception personnalisée (nommée `CarIsDeadException`) pour représenter l'erreur liée à l'accélération d'une voiture en panne. La première étape consiste à dériver une nouvelle classe de `System.Exception`/`System.ApplicationException` (==par convention, tous les noms de classes d'exception se terminent par le suffixe `Exception`)==.

>[!note]
>**En règle générale, toutes les classes d'exceptions personnalisées doivent être définies comme des classes publiques** (rappelons que le modificateur d'accès par défaut d'un type non imbriqué est `internal`). En effet, les exceptions sont souvent transmises en dehors des limites de l'assembly et doivent donc être accessibles au code appelant.

Créez un nouveau projet d'application console nommé *CustomException*, copiez les fichiers *Car.cs* et *Radio.cs* existants dans ce nouveau projet, et remplacez l'espace de noms définissant les types `Car` et `Radio` par `CustomException`. Ensuite, ajoutez un fichier nommé *CarIsDeadException.cs* et insérez la définition de classe suivante :

```cs
namespace CustomException;

// Cette exception personnnalisée décrit les détails
// de la condition "voiture est morte".
public class CarIsDeadException : Exception { }
```

Comme pour toute classe, **vous pouvez inclure autant de membres personnalisés que nécessaire, appelables dans le bloc `catch` de la logique appelante**. ==Vous pouvez également redéfinir les membres virtuels définis par vos classes parentes. Par exemple, vous pouvez implémenter `CarIsDeadException` en redéfinissant la propriété virtuelle `Message`==. 

**De plus, au lieu de remplir un dictionnaire de données (via la propriété `Data`) lors de la levée de l'exception, le constructeur permet à l'expéditeur de fournir un horodatage et la raison de l'erreur**. Enfin, l'horodatage et la cause de l'erreur peuvent être obtenus à l'aide de propriétés fortement typées.

```cs
namespace CustomException;

// Cette exception personnnalisée décrit les détails
// de la condition "voiture est morte".
public class CarIsDeadException : Exception
{
    private string _messageDetails = string.Empty;
    public DateTime ErrorTimeStamp { get; set; }
    public string CauseOfError { get; set; }

    public CarIsDeadException() { }

    public CarIsDeadException(string message, string cause, DateTime time)
    {
        _messageDetails = message;
        CauseOfError = cause;
        ErrorTimeStamp = time;
    }

    // Surcharge la propriété Exception.Message
    public override string Message => $"Car Error Message: {_messageDetails}";
}
```

Ici, **la classe `CarIsDeadException` conserve un champ privé** (`_messageDetails`) **qui représente des données relatives à l'exception actuelle, et qui peut être initialisé à l'aide d'un constructeur personnalisé**. Lever cette exception depuis la méthode `Accelerate()` est simple. Il suffit d'allouer, de configurer et de lever une exception de type `CarIsDeadException` plutôt qu'une `System.Exception`.

```cs
public void Accelerate(int delta)
{
    if (_carIsDead)
    {
        Console.WriteLine($"{PetName} is out of order...");
        return;
    }
    CurrentSpeed += delta;
    if (CurrentSpeed > MaxSpeed)
    {
        CurrentSpeed = 0;
        _carIsDead = true;
        
        // Utilisation de l'exception personnalisée
        throw new CarIsDeadException(
            $"{PetName} has overheated!",
            "You have a lead foot",
            DateTime.Now
        )
        {
            HelpLink = "https://www.google.com",
        };
    }
    Console.WriteLine($"=> CurrentSpeed = {CurrentSpeed}");
}
```

Pour intercepter cette exception entrante, votre portée de capture peut maintenant être mise à jour pour intercepter un type spécifique `CarIsDeadException` (cependant, étant donné que `CarIsDeadException` "est un" `System.Exception`, il est toujours permis d'intercepter également une `System.Exception`).

```cs
using CustomException;

Console.Title = "Fun with Custom Exceptions";
Console.WriteLine("**** Fun with Custom Exceptions ****\n");

Car myCar = new Car("Rusty", 90);

try
{
    // Déclenche l'exception
    myCar.Accelerate(50);
}
catch (CarIsDeadException e)
{
    Console.WriteLine(e.Message);
    Console.WriteLine(e.ErrorTimeStamp);
    Console.WriteLine(e.CauseOfError);
}
Console.ReadLine();
```

Maintenant que vous comprenez le processus de base de la création d'une exception personnalisée, il est temps d'approfondir ces connaissances.

## Création d'exceptions personnalisées, 2ème partie

==Le type `CarIsDeadException` actuel a redéfini la propriété virtuelle `System.Exception.Message` pour configurer un message d'erreur personnalisé et a fourni deux propriétés personnalisées pour prendre en compte des données supplémentaires==. En réalité, **vous n'êtes cependant pas obligé de redéfinir la propriété virtuelle `Message`, car vous pouvez simplement transmettre le message entrant au constructeur du parent comme suit** :

```cs
namespace CustomException;

public class CarIsDeadException : Exception
{
    public DateTime ErrorTimeStamp { get; set; }
    public string CauseOfError { get; set; }

    public CarIsDeadException() { }

    // Transmettre le message au constructeur parent.
    public CarIsDeadException(string message, string cause, DateTime time)
        : base(message)
    {
        CauseOfError = cause;
        ErrorTimeStamp = time;
    }
}
```

Notez que cette fois-ci, vous *n'avez pas* défini de membre de champs de type `string` pour représenter le message et vous *n'avez pas* redéfini la propriété `Message`. **Vous transmettez simplement le paramètre au constructeur de votre classe de base**. Avec cette conception, **une classe d'exception personnalisée n'est guère plus qu'une classe portant un nom unique et dérivant de `System.ApplicationException` (avec des propriétés supplémentaires si nécessaire), sans aucune redéfinition de la classe de base**.

Ne soyez pas surpris si la plupart (voire la totalité) de vos classes d'exception personnalisées suivent ce modèle simple. **Bien souvent, le rôle d'une exception personnalisée n'est pas nécessairement de fournir des fonctionnalités supplémentaires par rapport à celles héritées des classes de base, mais de fournir un *type fortement nommé* qui identifie clairement la nature de l'erreur afin que le client puisse fournir une logique de gestion différente pour différents types d'exceptions**.

## Création d'exceptions personnalisées, 3ème partie

==Pour créer une classe d'exception personnalisée parfaitement conforme aux bonnes pratiques, assurez-vous que votre exception personnalisée effectue les opérations suivantes== :

- Dériver de la classe `Exception`
- Définit un constructeur par défaut
- Définit un constructeur qui initialise la propriété `Message` héritée.
- Définit un constructeur pour gérer les "exceptions internes".

Pour compléter votre étude de la création d'exceptions personnalisées, voici la version finale de `CarIsDeadException`, qui prend en compte chacun de ces constructeurs spéciaux (les propriétés seraient telles que présentées dans l'exemple précédent) :

```cs
public class CarIsDeadException : Exception
{
    private string _messageDetails = String.Empty;
    public DateTime ErrorTimeStamp { get; set; }
    public string CauseOfError { get; set; }

    public CarIsDeadException() { }

    public CarIsDeadException(string cause, DateTime time)
        : this(cause, time, string.Empty) { }

    public CarIsDeadException(string cause, DateTime time, string message)
        : this(cause, time, message, null) { }

    public CarIsDeadException(
        string cause,
        DateTime time,
        string message,
        System.Exception inner
    )
        : base(message, inner)
    {
        CauseOfError = cause;
        ErrorTimeStamp = time;
    }
}
```

Suite à cette mise à jour de votre exception personnalisée, mettez à jour la méthode `Accelerate()` comme suit :

```cs
throw new CarIsDeadException(
    "You have a lead foot",
    DateTime.Now,
    $"{PetName} has overheated!"
)
{
    HelpLink = "https://www.google.com",
};
```

Étant donné que la création d'exceptions personnalisées conformes aux bonnes pratiques .NET Core ne diffère en réalité que par leur nom, vous serez ravi d'apprendre que Visual Studio fournit un modèle d'extrait de code nommé `Exception` qui génère automatiquement une nouvelle classe d'exception conforme aux bonnes pratiques .NET. Pour l'activer, saisissez exc dans l'éditeur et appuyez sur la touche Tabulation (dans Visual Studio, appuyez deux fois sur la touche Tabulation).

>[!important]- Pour Neovim
>*Explication de l'exemple est générée par Gemini*.
>
>le snippet `exception` est aussi disponible. Voici son "implémentation" :
>```cs
>[System.Serializable]
>public class MyException : System.Exception
>{
>    public MyException() { }
>
>    public MyException(string message)
>        : base(message) { }
>
>    public MyException(string message, System.Exception inner)
>        : base(message, inner) { }
>
>    protected MyException(
>        System.Runtime.Serialization.SerializationInfo info,
>        System.Runtime.Serialization.StreamingContext context
>    )
>        : base(info, context) { }
>}
>
>```
>
>L'attribut `[System.Serialisable]` indique au moteur .NET que l'objet (votre exception) peut être converti en un flux de données (binaire, XML, JSON) pour être transporté.
>
>**Pourquoi est-ce là ?** Imaginez que votre code s'exécute sur un serveur, mais qu'il est appelé par un client à distance (via WCF ou d'anciennes technologies .NET). Si une erreur survient sur le serveur, .NET doit "emballer" l'exception, l'envoyer sur le réseau et la "déballer" chez le client.
>
> ==Cependant==: 
>En .NET 8, Microsoft a officiellement marqué la sérialisation binaire comme **obsolète** (SYSLIB0051). Si tu compiles ce snippet dans un projet récent, tu recevras probablement un avertissement (Warning).
>
>- Si vous apprenez le C# et suivez le livre : **Gardez-le**, c'est formateur.
>- Si vous créez une application moderne (API Web, App Mobile) : Vous pouvez généralement **supprimer** l'attribut et le 4ème constructeur. Le snippet de base (les 3 premiers constructeurs) suffit amplement.

# Gestion de plusieurs exceptions

==Dans sa forme la plus simple, un bloc `try` ne contient qu'un seul bloc `catch`==. En pratique, cependant, **il arrive souvent que les instructions d'un bloc `try` puissent déclencher plusieurs exceptions**. Créez un nouveau projet d'application console C# nommé *ProcessMultipleExceptions* ; copiez les fichiers *Car.cs,* *Radio.cs* et *CarIsDeadException.cs* de l'exemple *CustomException* précédent dans ce nouveau projet, puis mettez à jour vos espaces de noms en conséquence.

Maintenant, ==mettez à jour la méthode `Accelerate()` de la classe `Car` afin qu'elle lève également une exception `ArgumentOutOfRangeException`== prédéfinie de la bibliothèque de classes de base si vous lui transmettez un paramètre invalide (c'est-à-dire toute valeur inférieure à zéro). ==Notez que le constructeur de cette classe d'exception prend le nom de l'argument fautif comme première chaîne de caractères, suivi d'un message décrivant l'erreur==.

```cs
public  void Accelerate(int delta)
{
    // Tester la validité de l'argument avant de continuer.
    if (delta < 0)
    {
        throw new ArgumentOutOfRangeException(
            nameof(delta),
            "Speed must be greater than zero"
        );
    }
    
    ...
}
```

>[!note] 
>L'opérateur `nameof()` renvoie une chaîne de caractères représentant le nom de l'objet, dans cet exemple la variable delta. Il s'agit d'une méthode plus sûre pour faire référence aux objets, méthodes et variables C# lorsque la version sous forme de chaîne est requise.

La logique de gestion des exceptions peut désormais répondre spécifiquement à chaque type d'exception.

```cs
using ProcessMultipleException;

Console.Title = "Handling Multiple Exceptions";
Console.WriteLine("**** Handling Multiple Exceptions ****\n");

Car myCar = new Car("Rusty", 90);
try
{
    // Déclanche l'exception "argument en dehors de la limite"
    myCar.Accelerate(-10);
}
catch (CarIsDeadException e)
{
    Console.WriteLine(e.Message);
}
catch (ArgumentOutOfRangeException e)
{
    Console.WriteLine(e.Message);
}
Console.ReadLine();
```

**Lorsque vous écrivez plusieurs blocs `catch`, sachez que lorsqu'une exception est levée, elle sera traitée par le premier bloc `catch` approprié**. Pour illustrer précisément ce que signifie "*premier bloc catch approprié*", imaginez que vous ayez ajouté à la logique précédente un bloc catch supplémentaire qui tente de gérer toutes les exceptions, autres que `CarIsDeadException` et `ArgumentOutOfRangeException`, en interceptant une exception `System.Exception` générale comme suit:

```cs
// Ce code ne compilera pas!
using ProcessMultipleException;

Console.Title = "Handling Multiple Exceptions";
Console.WriteLine("**** Handling Multiple Exceptions ****\n");

Car myCar = new Car("Rusty", 90);
try
{
    // Déclanche l'exception "argument en dehors de la limite"
    myCar.Accelerate(-10);
}
catch (Exception e)
{
    // Traiter toutes les autres exceptions ?
    Console.WriteLine(e.Message);
}
catch (CarIsDeadException e)
{
    Console.WriteLine(e.Message);
}
catch (ArgumentOutOfRangeException e)
{
    Console.WriteLine(e.Message);
}
Console.ReadLine();
```

Cette logique de gestion des exceptions **génère des erreurs de compilation**. ==Le problème est que le premier bloc `catch` peut gérer toute exception dérivée de `System.Exception`== (étant donné la relation "est un"), ==y compris les types `CarIsDeadException` et `ArgumentOutOfRangeException`==. Par conséquent, **les deux derniers blocs `catch` sont inaccessibles !**

**La règle générale à retenir est de s'assurer que vos blocs `catch` sont structurés de telle sorte que le premier `catch` soit l'exception la plus spécifique** (c.-à-d. ==le type le plus dérivé dans une chaîne d'héritage de types d'exceptions==), **et que le dernier `catch` soit réservé à l'exception la plus générale** (c.-à-d. la classe de base d'une chaîne d'héritage d'exceptions donnée, ici `System.Exception`).

Ainsi, si vous souhaitez définir un bloc `catch` qui gérera toutes les erreurs autres que `CarIsDeadException` et `ArgumentOutOfRangeException`, vous pouvez écrire le code suivant :

```cs
// Ce code se compilera sans problème.
using ProcessMultipleException;

Console.Title = "Handling Multiple Exceptions";
Console.WriteLine("**** Handling Multiple Exceptions ****\n");

Car myCar = new Car("Rusty", 90);
try
{
    // Déclanche l'exception "argument en dehors de la limite"
    myCar.Accelerate(-10);
}
catch (CarIsDeadException e)
{
    Console.WriteLine(e.Message);
}
catch (ArgumentOutOfRangeException e)
{
    Console.WriteLine(e.Message);
}
// Ceci interceptera toute autre exception
// autre que CarIsDeadException ou
// ArgumentOutOfRangeException.
catch (Exception e)
{
    // Traiter toutes les autres exceptions ?
    Console.WriteLine(e.Message);
}
Console.ReadLine();
```

>[!tip] Dans la mesure du possible
>**Privilégiez toujours la capture de classes d'exceptions spécifiques plutôt qu'une exception générale de type `System.Exception`**. Bien que cela puisse sembler simplifier les choses à court terme (vous pourriez penser : « Ah ! Cela capte toutes les autres choses qui ne m'intéressent pas. »), à long terme, ==vous pourriez vous retrouver avec des plantages inattendus à l'exécution, car une erreur plus grave n'aurait pas été gérée directement dans votre code==. **N'oubliez pas qu'un bloc `catch` final qui gère une exception de type `System.Exception` a tendance à être très général**.

## Instructions catch générales

C# prend également en charge une portée `catch` "générale" qui ne reçoit pas explicitement l'objet exception levé par un membre donné.

```cs
// Un catch générique
Console.Title = "Handling Multiple Exceptions";
Console.WriteLine("**** Handling Multiple Exceptions ****\n");

Car myCar = new Car("Rusty", 90);
try
{
	myCar.Accelerate(90);
}
catch
{
	Console.WriteLine("Something bad happened...");
}
Console.ReadLine();
```

Il est évident que ce n'est pas la méthode la plus informative pour gérer les exceptions, car vous n'avez aucun moyen d'obtenir des données pertinentes sur l'erreur survenue (telles que le nom de la méthode, la pile d'appels ou un message personnalisé). Néanmoins, C# autorise une telle construction, ce qui peut s'avérer utile lorsque vous souhaitez gérer toutes les erreurs de manière générale.

## Re-lever les exceptions
**Lorsqu'une exception est interceptée, la logique d'un bloc `try` peut la *relancer* vers l'appelant précédent dans la pile d'appels**. ==Pour ce faire, il suffit d'utiliser `throw()` dans un bloc `catch`. Cela transmet l'exception vers le haut de la chaîne d'appels, ce qui peut s'avérer utile si votre bloc `catch` ne gère que partiellement l'erreur==.

```cs
// Se décharger de la responsabilité.
... 
try
{
	// Accélérer la logique du véhicule...

} 
catch(CarIsDeadException e)
{
	// Effectuer un traitement partiel de cette erreur et se décharger de la responsabilité.
	throw;
}
...
```

## Exceptions internes

Comme vous pouvez vous en douter, ==il est tout à fait possible de déclencher une exception pendant que vous en gérez une autre==. Par exemple, supposons que vous gériez une exception `CarIsDeadException` dans un bloc `catch` particulier et que, pendant ce processus, vous tentiez d'enregistrer la trace de la pile dans un fichier nommé *carErrors.txt* sur votre lecteur `C:` sur Windows ou `home` pour Linux et MacOS. (Les instructions `global using` implicites vous donnent accès à l'espace de noms `System.IO` et à ses types centrés sur les E/S).

```cs
...
catch (CarIsDeadException e)
{
    // Tentative d'ouverture d'un fichier 
    // nommé carErrors.txt situé dans home.
    FileStream fs = File.Open(@"~/carErrors.txt", FileMode.Open);
}
...
```

**Si le fichier spécifié est introuvable, l'appel à `File.Open()` génère un `FileNotFoundException`!** ==Plus loin dans cet ouvrage, vous découvrirez l'espace de noms `System.IO` et apprendrez à déterminer par programmation l'existence d'un fichier sur le disque dur avant même de tenter de l'ouvrir== (évitant ainsi l'exception). Cependant, pour rester concentrés sur le sujet des exceptions, supposons qu'une exception ait été levée.

**Lorsqu'une exception survient pendant le traitement d'une autre, il est recommandé d' enregistrer le nouvel objet exception comme une "exception interne" au sein d'un nouvel objet du même type que l'exception initiale**. (C'est un peu long !) ==La raison pour laquelle il est nécessaire d'allouer un nouvel objet pour l'exception en cours de gestion est que le seul moyen de documenter une exception interne est via un paramètre de constructeur==. Prenons l'exemple du code suivant :

```cs
// Mise à jour du gestionnaire d'exception.
catch (CarIsDeadException e)
{
    try
    {
        FileStream fs = File.Open(@"./carErrors.txt", FileMode.Open);
        Console.WriteLine(e.Message);
    }
    catch (Exception e2)
    {
        // Ceci provoque une erreur de compilation
        // - InnerException est en lecture seule
        //e.InnerException = e2;

        // Lève une exception qui enregistre la nouvelle exception,
        // ainsi que le message de la première exception.
        throw new CarIsDeadException(
            cause: e.CauseOfError,
            time: e.ErrorTimeStamp,
            message: e.Message,
            inner: e2
        );
    }
}

```

Notez que, dans ce cas, j'ai ==passé l'objet `FileNotFoundException` comme quatrième paramètre au constructeur `CarIsDeadException`==. **Après avoir configuré ce nouvel objet, vous le transmettez à l'appelant suivant dans la pile d'appels, en l'occurrence les instructions de niveau supérieur**.

**Étant donné qu'il n'y a pas d'appelant suivant après les instructions de niveau supérieur pour intercepter l'exception, vous verrez à nouveau une boîte de dialogue d'erreur**. Tout comme le fait de relancer une exception, ==l'enregistrement des exceptions internes n'est généralement utile que si l'appelant est capable d'intercepter l'exception correctement dès le départ==. Si tel est le cas, la logique de capture de l'appelant peut utiliser la propriété `InnerException` pour extraire les détails de l'objet exception interne.

## Le bloc `finally`

**Un bloc `try`/`catch` peut également définir un bloc `finally` optionnel. Ce bloc `finally` a pour but de *garantir l’exécution systématique* d’un ensemble d’instructions, exception (de tout type) ou non**. Par exemple, supposons que vous souhaitiez toujours éteindre l’autoradio avant de quitter le programme, quelle que soit l’exception gérée.

```cs
Console.WriteLine("***** Gestion de plusieurs exceptions *****\n");
Car myCar = new Car("Rusty", 90);

myCar.CrankTunes(true);
try
{
    // Accélérer la logique de la voiture.
}
catch (CarIsDeadException e)
{
    // Traiter CarIsDeadException.
}
catch (ArgumentOutOfRangeException e)
{
    // Traiter ArgumentOutOfRangeException.
}
catch (Exception e)
{
    // Traiter toute autre exception.
}
finally
{
    // Cela se produira toujours. Exception ou pas.
    myCar.CrankTunes(false);
}
Console.ReadLine();
```

**Si vous n'aviez pas inclus de bloc `finally`, la radio ne serait pas désactivée en cas d'exceptions** (ce qui pourrait être problématique ou non). ==Dans un scénario plus concret, lorsque vous devez libérer des objets, fermer un fichier ou vous déconnecter d'une base de données (ou autre), un bloc `finally` garantit un emplacement pour un nettoyage approprié==.

## Filtres d'exceptions

**C# 6 a introduit une nouvelle clause pouvant être placée dans la portée d'un bloc `catch`, via le mot-clé `when`**. ==L'ajout de cette clause permet de s'assurer que les instructions d'un bloc `catch` ne sont exécutées que si une condition est vraie==. **Cette condition doit être évaluée à un booléen** (vrai ou faux) **et peut être obtenue soit par une simple instruction dans la définition de `when`, soit en appelant une méthode supplémentaire**. ==En résumé, cette approche permet d'ajouter des filtres à la logique de gestion des exceptions.==

Prenons l'exemple suivant : une clause `when` a été ajoutée au gestionnaire d'exceptions `CarIsDeadException` afin de garantir que le bloc `catch` ne soit jamais exécuté le vendredi (un exemple artificiel, certes, mais qui souhaite une panne de voiture juste avant le week-end ?). Notez que l'expression booléenne de la clause `when` doit être placée entre parenthèses.

```cs
catch (CarIsDeadException e)
    when (e.ErrorTimeStamp.DayOfWeek != DayOfWeek.Friday)
{
    // Cette nouvelle ligne ne sera affiché qui si la comparaison est vrai.
    Console.WriteLine("Catching car is dead!");
	
	...
}
```

Bien que cet exemple soit très artificiel, ==une utilisation plus réaliste d'un filtre d'exceptions consiste à intercepter les `SystemExceptions`==. Par exemple, supposons que votre code enregistre des données dans la base de données; une exception générale est alors levée. En examinant le message et les données de l'exception, vous pouvez créer des gestionnaires spécifiques en fonction de la cause de l'exception.

# Déboggage des exceptions non gérées avec Visual Studio

Visual Studio propose plusieurs outils pour vous aider à débogguer les exceptions non gérées. Supposons que vous ayez augmenté la vitesse d'un objet `Car` au-delà de sa valeur maximale, mais que vous n'ayez pas pris la peine d'encapsuler votre appel dans un bloc `try`.

```cs
Car myCar = new Car("Rusty", 90);
myCar.Accelerate(100);
```

Si vous lancez une session de déboggage dans Visual Studio (via le menu Débogguer ➤ Démarrer le déboggage), Visual Studio s'interrompt automatiquement au moment où l'exception non interceptée est levée. Mieux encore, une fenêtre s'affiche indiquant la valeur de la propriété `Message`.

![[Figure 7.1.png|Déboggage des exceptions personnalisées non gérées avec Visual Studio]]

>[!note]-
>Si vous ne gérez pas une exception levée par une méthode des bibliothèques de classes de base .NET, le débogueur Visual Studio s'arrête à l'instruction qui a appelé la méthode fautive.

Si vous cliquez sur le lien Afficher les détails, vous trouverez les détails concernant l'état de l'objet.

![[Figure 7.2.png|Affichage des détails des exceptions]]

# Résumé du Chapitre
Ici, vous avez **examiné le rôle de la gestion structurée des exceptions**. ==Lorsqu'une méthode doit envoyer un objet d'erreur à l'appelant, elle alloue, configure et lève une exception de type spécifique dérivé de `System.Exception` via `throw()`==. **L'appelant peut gérer toute exception entrante possible à l'aide du mot-clé `catch` de C# et d'une boucle `finally` facultative**. Depuis C# 6.0, ==la possibilité de créer des filtres d'exceptions à l'aide du mot-clé `when` facultatif a été ajoutée, et C# 7 a étendu les emplacements à partir desquels il est possible de lever des exceptions==.

**Lorsque vous créez vos propres exceptions personnalisées, vous créez finalement un type de classe dérivant de `System.ApplicationException` (`System.Exception` pour les codebases récent), qui désigne une exception levée par l'application en cours d'exécution**. En revanche, ==les objets d'erreur dérivant de `System.SystemException` représentent des erreurs critiques (et fatales) levées par le runtime .NET==. Enfin, ce chapitre a illustré divers outils de Visual Studio permettant de créer des exceptions personnalisées (conformément aux bonnes pratiques .NET) ainsi que de débogguer ces exceptions.

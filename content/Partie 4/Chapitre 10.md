---
title: "Chapitre 10: Collections et Génériques"
publish: true
---

# <big><big><big><b><font color =green>Collections et Génériques</font></b></big></big></big>

**Toute application créée avec la plateforme .NET Core devra gérer la maintenance et la manipulation d'un ensemble de données en mémoire**. Ces données peuvent provenir de diverses sources, notamment une base de données relationnelle, un fichier texte local, un document XML, un appel de service web ou encore des données saisies par l'utilisateur.

Lors de la première version de la plateforme .NET, **les programmeurs utilisaient fréquemment les classes de l'espace de noms `System.Collections` pour stocker et manipuler les données utilisées dans une application**. Dans .NET 2.0, ==le langage de programmation C# a été amélioré pour prendre en charge les génériques; avec cette évolution, un nouvel espace de noms a été introduit dans les bibliothèques de classes de base : `System.Collections.Generic`.==

Ce chapitre vous donnera un aperçu des ==différents espaces de noms et types de collections (génériques et non génériques)== présents dans les bibliothèques de classes de base de .NET Core. Comme vous le verrez, **les conteneurs génériques sont souvent préférés à leurs homologues non génériques car ils offrent généralement une meilleure sécurité des types et des performances accrues**. Après avoir appris à créer et à manipuler les éléments génériques du framework, le reste de ce chapitre examinera ==comment construire vos propres méthodes génériques et types génériques==. Ce faisant, vous découvrirez le rôle des *contraintes* (et le mot-clé C# correspondant `where`), qui vous permettent de créer des classes extrêmement sûres en termes de types.

# Motivations pour les classes `Collection`

**Le conteneur le plus primitif pour stocker les données d'une application est sans aucun doute le tableau**. Comme vous l'avez vu au [[Chapitre 4#Comprendre les Tableaux C|Chapitre 4]], ==les tableaux C# permettent de définir un ensemble d'éléments de même type== (y compris un tableau d'objets `System.Array`, qui représente essentiellement un tableau de données de tout type) ==**d'une taille maximale fixe**==. Rappelez-vous également du [[Chapitre 4#Utilisation de la classe de base `System.Array`|Chapitre 4]] : ==toutes les variables de type tableau C# bénéficient d'une grande partie des fonctionnalités de la classe `System.Array`==. Pour un bref rappel, considérez le code suivant, qui crée un tableau de données textuelles et manipule son contenu de différentes manières :

```cs
// Créer un tableau de chaînes de caractères.
string[] strArray = {"First", "Second", "Third" };

// Afficher le nombre d'éléments du tableau à l'aide de la propriété Length.
Console.WriteLine("Ce tableau contient {0} éléments.", strArray.Length);
Console.WriteLine();

// Afficher le contenu à l'aide d'un énumérateur.
foreach (string s in strArray)
{
	Console.WriteLine("Élément du tableau : {0}", s);
}
Console.WriteLine();

// Inverser le tableau et afficher à nouveau.
Array.Reverse(strArray);
foreach (string s in strArray)
{ 
	Console.WriteLine("Élément du tableau : {0}", s);
} 
Console.ReadLine();
```

Bien que les tableaux simples puissent être utiles pour gérer de petites quantités de données de taille fixe, ==il existe de nombreuses autres situations où une structure de données plus flexible est nécessaire, comme un conteneur dont **la taille peut varier dynamiquement** ou un conteneur pouvant contenir uniquement des objets répondant à un critère spécifique== (par exemple, uniquement les objets héritant d'une classe de base spécifique ou uniquement les objets implémentant une interface particulière). **Lorsque vous utilisez un tableau simple, n'oubliez jamais qu'il est créé avec une «taille fixe».** Si vous créez un tableau de trois éléments, vous n'obtiendrez que trois éléments ; par conséquent, le code suivant provoquera une exception d'exécution (une exception `IndexOutOfRangeException`, pour être précis) :

```cs
// Créer un tableau de chaînes de caractères.
string[] strArray = { "Premier", "Deuxième", "Troisième" };

// Tentative d'ajout d'un nouvel élément à la fin ? Erreur d'exécution !
strArray[3] = "nouvel élément ?";

...
```

>[!note] Il est possible de modifier la taille d'un tableau à l'aide de la méthode générique `Resize()<T>`. Cependant, cela entraîne la copie des données dans un nouvel objet tableau et peut s'avérer inefficace.

==Pour pallier les limitations d'un tableau simple, les bibliothèques de classes de base .NET Core incluent un certain nombre d'espaces de noms contenant des *classes de collection*==. **Contrairement à un tableau C# simple, les classes de collection sont conçues pour se redimensionner dynamiquement à la volée lors de l'insertion ou de la suppression d'éléments**. De plus, ==nombre d'entre elles offrent une sécurité de type accrue et sont hautement optimisées pour traiter les données de manière économe en mémoire==. Au fil de ce chapitre, vous constaterez rapidement qu'une classe de collection peut appartenir à l'une des deux grandes catégories:

- **Collections non génériques** (principalement dans l'espace de noms `System.Collections`)
- **Collections génériques** (principalement dans l'espace de noms `System.Collections.Generic`)

**Les collections non génériques sont généralement conçues pour fonctionner avec des types `System.Object` et sont donc, des conteneurs faiblement typés** (cependant, ==certaines collections non génériques ne fonctionnent qu'avec un type de données spécifique, comme les objets chaînes de caractères==). En revanche, **les collections génériques sont beaucoup plus sûres au niveau des types, étant donné que vous devez spécifier le «type de type» qu'elles contiennent lors de leur création**. Comme vous le verrez, ==le signe distinctif de tout élément générique est le «paramètre de type» marqué par des chevrons== (par exemple, `List<T>`). Vous examinerez les détails des génériques (y compris leurs nombreux avantages) un peu plus loin dans ce chapitre. Pour l'instant, examinons quelques-uns des principaux types de collections non génériques dans les espaces de noms `System.Collections` et `System.Collections.Specialized`.

## L’espace de noms `System.Collections`

Lors du lancement de la plateforme .NET, les programmeurs utilisaient fréquemment les classes de collections non génériques de l’espace de noms `System.Collections`, qui contient un ensemble de classes permettant de gérer et d’organiser de grandes quantités de données en mémoire. Le [[#Tableau 10-1 Types utiles de `System.Collections`|Tableau 10-1]] présente certaines des classes de collections les plus couramment utilisées de cet espace de noms et les interfaces principales qu’elles implémentent.

##### Tableau 10-1: Types utiles de `System.Collections`

| Classe `System.Collections` | Description                                                                                                                                                               | Interfaces clés implémentées                                 |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------ |
| `ArrayLiist`                | Représente une collection d'objets de taille dynamique, listés dans l'ordre séquentiel.                                                                                   | `IList`, `ICollection`, `IEnumerable`, et `ICloneable`       |
| `BitArray`                  | Gère un tableau compact de valeurs binaires, représentées sous forme de booléens, où `true` indique que le bit est activé (1) et `false` indique qu'il est désactivé (0). | `ICollection`, `IEnumerable`, et `ICloneable`                |
| `Hashtable`                 | Représente une collection de paires clé-valeur organisées selon le code de hachage de la clé                                                                              | `IDictionary`, `ICollection`, `IEnumerable`, et `ICloneable` |
| `Queue`                     | Représente une collection d'objets standard de type premier entré, premier sorti (FIFO).                                                                                  | `ICollection`, `IEnumerable`, et `ICloneable`                |
| `SortedList`                | Représente une collection de paires clé-valeur qui sont triées par clés et accessibles par clé et par index                                                               | `IDictionary`, `ICollection`, `IEnumerable`, et `ICloneable` |
| `Stack`                     | Une pile LIFO (dernier entré, premier sorti) offrant les fonctionnalités d'empilement, de dépilement et de consultation.                                                  | `ICollection`, `IEnumerable`, et `ICloneable`                |

**Les interfaces implémentées par ces classes de collections offrent un aperçu précieux de leur fonctionnalité globale**. Le [[#Tableau 10-2 Interfaces clés prises en charge par les classes de `System.Collections`|Tableau 10-2]] décrit la nature générale de ces interfaces clés, dont certaines ont été abordées directement au [[Chapitre 8#Les interfaces `IEnumerable` et `IEnumerator`|Chapitre 8]].

##### Tableau 10-2: Interfaces clés prises en charge par les classes de `System.Collections`

| Interfaces `System.Collections` | Description                                                                                                                                           |
| ------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ICollection`                   | Définit les caractéristiques générales (par exemple, la taille, le nombre et la sécurité du thread) pour tous les types de collections non génériques |
| `ICloneable`                    | Permet à l'objet implémentant de renvoyer une copie de lui-même à l'appelant.                                                                         |
| `IDictionary`                   | Permet à un objet de collection non générique de représenter son contenu à l'aide de paires clé-valeur.                                               |
| `IEnumerate`                    | Renvoie un objet implémentant l'interface `IEnumerator` (voir l'entrée suivante du tableau).                                                          |
| `IEnumerator`                   | Permet l'itération de type `foreach` sur les éléments de la collection                                                                                |
| `IList`                         | Permet d'ajouter, de supprimer et d'indexer des éléments dans une liste séquentielle d'objets.                                                        |

### Exemple illustratif : Comprendre les `ArrayList`

Selon votre expérience, vous avez peut-être ==déjà utilisé== (ou implémenté) ==certaines de ces structures de données classiques telles que les piles, les files et les listes==. Si ce n’est pas le cas, je vous fournirai des détails supplémentaires sur leurs différences lorsque vous étudierez leurs équivalents génériques un peu plus loin dans ce chapitre. En attendant, voici un exemple de code utilisant un objet `ArrayList` :

```cs
// Vous devez importer System.Collections pour accéder à l'ArrayList.
using System.Collections;

ArrayList strArray = new ArrayList();
strArray.AddRange(new string[] { "First", "Second", "Third" });

// Afficher le nombre d'éléments dans l'ArrayList.
Console.WriteLine("Cette collection contient {0} éléments.", strArray.Count);
Console.WriteLine();

// Ajouter un nouvel élément et afficher le nombre actuel.
strArray.Add("Fourth!");
Console.WriteLine("Cette collection contient {0} éléments.", strArray.Count);

// Afficher le contenu.
foreach (string s in strArray)
{ 
	Console.WriteLine("Entrée : {0}", s);
} 
Console.WriteLine();
```

==Notez que vous pouvez ajouter (ou supprimer) des éléments à la volée et que le conteneur se redimensionne automatiquement.==

Comme vous vous en doutez, **la classe `ArrayList` possède de nombreux membres utiles, outre la propriété `Count` et les méthodes `AddRange()` et `Add()`**. Consultez la documentation .NET Core pour plus de détails. Par ailleurs, les autres classes de `System.Collections` (Stack, Queue, etc.) sont également entièrement documentées dans le système d'aide de .NET Core (lien [ici](https://learn.microsoft.com/en-us/dotnet/api/system.collections?view=net-10.0)).

==Cependant, il est important de souligner que la plupart de vos projets .NET Core n'utiliseront probablement pas les classes de collections de l'espace de noms `System.Collections`==. En effet, **il est aujourd'hui beaucoup plus courant d'utiliser les classes génériques correspondantes de l'espace de noms `System.Collections.Generic`**. C'est pourquoi je ne commenterai pas (et ne fournirai pas d'exemples de code) les autres classes non génériques de `System.Collections`.

>[!tldr]-
>Ne perdez pas trop de temps à mémoriser les membres de `System.Collections` (ArrayList, Queue, Stack, Hashtable).
>
>**Concentrez-vous sur `System.Collections.Generic`**, car :
>- Ils sont plus performants.
>- Ils fatiguent moins le Garbage Collector.
>- Ils sont le standard de l'industrie depuis .NET 2.0 (2005 !).

## Présentation de l'espace de noms `System.Collections.Specialized`

`System.Collections` n'est pas le seul espace de noms .NET Core contenant des classes de collections non génériques. L'espace de noms `System.Collections.Specialized` définit plusieurs types de collections spécialisés (veuillez excuser la redondance). ==Le [[#Tableau 10-3 Classes utiles de `System.Collections.Specialized`|Tableau 10-3]] présente certains des types les plus utiles de cet espace de noms dédié aux collections, tous non génériques.==

##### Tableau 10-3: Classes utiles de `System.Collections.Specialized`

| Type `System.Collections.Specialized` | Description                                                                                                                                                                           |
| ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `HybridDictionary`                    | Cette classe implémente `IDictionary` en utilisant un `ListDictionary` lorsque la collection est petite, puis en passant à une `Hashtable` lorsque la collection devient volumineuse. |
| `ListDictionary`                      | Cette classe est utile pour gérer un petit nombre d'éléments (une dizaine) susceptibles d'évoluer. Elle utilise une liste simplement chaînée pour stocker les données.                |
| `StringCollection`                    | Cette classe offre une méthode optimale pour gérer de grandes collections de chaînes de caractères.                                                                                   |
| `BitVector32`                         | Elle fournit une structure simple qui stocke les valeurs booléennes et les petits entiers sur 32 bits de mémoire.                                                                     |

Au-delà de ces types de classes concrets, **cet espace de noms contient également de nombreuses interfaces et classes de base abstraites supplémentaires que vous pouvez utiliser comme point de départ pour créer des classes de collections personnalisées**. ==Bien que ces types «spécialisés» puissent répondre précisément aux besoins de vos projets dans certaines situations, je ne commenterai pas leur utilisation ici==. Encore une fois, **dans de nombreux cas, vous constaterez probablement que l’espace de noms `System.Collections.Generic` fournit des classes aux fonctionnalités similaires et présentant des avantages supplémentaires**.

>[!note]
>**Il existe deux espaces de noms supplémentaires axés sur les collections** (`System.Collections.ObjectModel` et `System.Collections.Concurrent`) **dans les bibliothèques de classes de base .NET Core**. ==Vous étudierez le premier espace de noms plus loin dans ce chapitre, une fois que vous maîtriserez le sujet des génériques==. **`System.Collections.Concurrent` fournit des classes de collections parfaitement adaptées à un environnement multithread** (voir le [[Chapitre 15#La relation Processus/Domaine d'application/Contexte/Thread|Chapitre 15]] pour plus d'informations sur le multithreading).

# Les problèmes des collections non génériques

Bien que de nombreuses applications .NET et .NET Core performantes aient été développées au fil des ans à l'aide de ces classes (et interfaces) de collections non génériques, ==l'expérience a montré que leur utilisation peut engendrer un certain nombre de problèmes==.

Le premier problème est que **l'utilisation des classes `System.Collections` et `System.Collections.Specialized` peut produire un code peu performant, notamment lors de la manipulation de données numériques** (par exemple, des types valeur). Comme vous le verrez prochainement, ==CoreCLR doit effectuer de nombreuses opérations de transfert de mémoire lorsque vous stockez des structures dans une classe de collection non générique prototypée pour fonctionner sur `System.Objects`, ce qui peut nuire à la vitesse d'exécution==.

Le second problème est que **la plupart des classes de collections non génériques ne sont pas sûres au niveau des types car** (encore une fois) **elles ont été développées pour fonctionner sur `System.Objects` et peuvent donc contenir n'importe quel type**. Si un développeur devait créer une collection hautement sécurisée (par exemple, un conteneur pouvant contenir des objets implémentant uniquement une certaine interface), ==la seule option réaliste était de créer manuellement une nouvelle classe de collection==. **Cette opération n'était pas excessivement laborieuse, mais elle s'avérait néanmoins un peu fastidieuse**.

Avant d'aborder l'utilisation des génériques dans vos programmes, ==il est utile d'examiner de plus près les problèmes des classes de collections non génériques== ; cela vous permettra de mieux comprendre les problèmes que les génériques étaient destinés à résoudre. Si vous souhaitez suivre, créez un nouveau projet d'application console nommé *IssuesWithNonGenericCollections*. Ensuite, assurez-vous d'importer l'espace de noms `System.Collections` en haut du fichier *Program.cs* et supprimez le reste du code.

```cs
using System.Collections;
```

## La question des performances

Comme vous vous en souvenez peut-être du [[Chapitre 4#Comprendre les types de valeur et les types de référence|Chapitre 4]], ==la plateforme .NET Core prend en charge deux grandes catégories de données : les types valeur et les types référence==. Étant donné que .NET Core définit deux grandes catégories de types, **il peut arriver que vous ayez besoin de représenter une variable d’une catégorie comme une variable de l’autre**. Pour ce faire, **==C# fournit un mécanisme simple, appelé *boxing*, permettant de stocker les données d’un type valeur dans une variable référence==**. Supposons que vous ayez créé une variable locale de type `int` dans une méthode appelée `SimpleBoxUnboxOperation`. **Si, au cours de votre application, vous deviez représenter ce type valeur comme un type référence, vous devriez effectuer un «boxing» de la valeur**, comme suit :

```cs
static void SimpleBoxUnboxOperation()
{
    // Créer une variable de type ValueType (int).
    int myInt = 25;

    // Convertir l'entier en référence d'objet.
    object boxedInt = myInt;
}
```

**Le boxing peut être formellement définie comme le processus d'assignation explicite d'un type de valeur à une variable `System.Object`**. ==Lorsqu'une valeur est empaquetée / emballée, le CoreCLR alloue un nouvel objet sur le tas et y copie la valeur du type de valeur== (`25`, dans ce cas). ==Ce qui vous est retourné est une référence à l'objet nouvellement alloué sur le tas.==

**==L'opération inverse est également possible à travers l'*unboxing*)==**. **Le déballage est le processus de conversion de la valeur contenue dans la référence d'objet en un type de valeur correspondant sur la pile**. **==Syntactiquement parlant, une opération de déballage ressemble à une opération de conversion classique (cast)==**. ==Cependant, la sémantique est très différente. Le CoreCLR commence par vérifier que le type de données de réception est équivalent au type encapsulé, et si c'est le cas, il copie la valeur dans une variable locale sur la pile==. Par exemple, les opérations d'encapsulation suivantes fonctionnent correctement, étant donné que le type sous-jacent de `boxedInt` est bien un `int` :

```cs
static void SimpleBoxUnboxOperation()
{
    // Créer une variable de type ValueType (int).
    int myInt = 25;

    // Empqqueter l'entier en une référence d'objet.
    object boxedInt = myInt;

    // Désempaqueter la référence en un entier correspondant.
    int unboxedInt = (int)boxedInt;
}
```

Lorsque le compilateur C# rencontre une syntaxe d'empaquetage/désempaquetage, il génère du code CIL contenant les codes d'opération d'empaquetage/désempaquetage. Si vous examiniez votre assembleur compilé à l'aide d'*ildasm.exe*, vous trouveriez ce qui suit :

```CIL
.method assembly hidebysig static void 
        '<<Main>$>g__SimpleBoxUnboxOperation|0_0'() cil managed
{
  .custom instance void [System.Runtime]System.Runtime.CompilerServices.CompilerGeneratedAttribute::.ctor() = ( 01 00 00 00 ) 
  // Code size       19 (0x13)
  .maxstack  1
  .locals init (int32 V_0,
           object V_1,
           int32 V_2)
  IL_0000:  nop
  IL_0001:  ldc.i4.s   25
  IL_0003:  stloc.0
  IL_0004:  ldloc.0
  IL_0005:  box        [System.Runtime]System.Int32
  IL_000a:  stloc.1
  IL_000b:  ldloc.1
  IL_000c:  unbox.any  [System.Runtime]System.Int32
  IL_0011:  stloc.2
  IL_0012:  ret
} // end of method Program::'<<Main>$>g__SimpleBoxUnboxOperation|0_0'
```

N'oubliez pas que, **contrairement à une conversion classique, vous ==*devez*== déballer les données dans un type approprié**. ==Si vous tentez de déballer des données dans un type incorrect, une exception `InvalidCastException` sera levée==. Pour une sécurité optimale, **il est recommandé emballer chaque opération de déballage dans un bloc `try`/`catch`**; cependant, ==cela serait fastidieux à implémenter pour chaque opération==. Prenons l'exemple de la mise à jour de code suivante, qui générera une erreur car vous tentez de déballer un `int` en un `long` :

```cs
static void SimpleBoxUnboxOperation()
{
    // Créer une variable de type ValueType (int).
    int myInt = 25;

    // Convertir l'entier en une référence d'objet.
    object boxedInt = myInt;

    // Déconvertir avec un type de données incorrect pour déclencher
    // une exception d'exécution.
    try
    {
        long unboxedLong = (long)boxedInt;
    }
    catch (InvalidCastException ex)
    {
        Console.WriteLine(ex.Message);
    }
}
```

À première vue, le boxing/unboxing peut sembler une fonctionnalité du langage plutôt banale, voire plus académique que pratique. Après tout, il est rare d'avoir besoin de stocker un type de valeur local dans une variable objet locale, comme illustré ici. Cependant, **il s'avère que le processus d'emballage/désemballage est très utile car il permet de considérer que tout peut être traité comme un `System.Object`, tandis que CoreCLR gère à votre place les détails liés à la mémoire**.

==Voyons une application pratique de ces techniques. Nous allons examiner la classe `System.Collections.ArrayList` et l'utiliser pour stocker un lot de données numériques== (allouées sur la pile). Les membres pertinents de la classe `ArrayList` sont listés ci-dessous. Notez qu'ils sont prototypés pour opérer sur des données `System.Object`. Considérons maintenant les méthodes `Add()`, `Insert()` et `Remove()`, ainsi que l'indexeur de classe.

```cs
public class ArrayList : IList, ICloneable
{
	...

	public virtual int Add(object? value);
	public virtual void Insert(int index, object? value);
	public virtual void Remove(object? obj);
	public virtual object? this[int index] {get; set; }
}
```

>[!tip]
> L'indexeur `this[int index]` est le membre principal de l'interface **`IList`**. C'est ce qui différencie une "Liste" d'un simple "Ensemble" (Set) ou d'une "File" (Queue) : la capacité d'accéder directement à n'importe quel élément par sa position.
>
> Cette propriété ce comportent comme une méthode avec comme paramètre un `int` et qui englobe en lui le getter set le setter. l'indexeur est le code permettant d'écrire le code suivant:
>  ```cs
>object item = maListe[0]; 
>```
>>[!example] Pourquoi cette syntax ?
>>Le créateur du C# (Anders Hejlsberg) voulait que l'utilisateur de la classe ait l'impression de manipuler un **tableau**, pas d'appeler une fonction.
>>- Si on utilisait des parenthèses `myList(0)`, cela ressemblerait à une **méthode**.
>>- En utilisant des crochets `myList[0]`, cela ressemble à de la **donnée**.
>>
>>C'est une décision purement **esthétique** pour le développeur qui utilise votre classe. Mais pour vous qui l'écrivez, c'est un hybride étrange.

**`ArrayList` a été conçu pour manipuler des objets, qui représentent des données allouées sur le tas**. ==Il peut donc sembler étrange que le code suivant se compile et s'exécute sans générer d'erreur== :

```cs
static void WorkWithArrayList()
{
    // Les types valeur sont automatiquement encapsulés lorsqu'ils
    // sont passés à une méthode demandant un objet.
    ArrayList myInts = new ArrayList();
    myInts.Add(10);
    myInts.Add(20);
    myInts.Add(35);
}
```

==Bien que vous transmettiez directement des données numériques aux méthodes nécessitant un objet, l'environnement d'exécution convertit automatiquement les données allouées sur la pile==. Par la suite, **si vous souhaitez récupérer un élément de l'`ArrayList` à l'aide de l'indexeur de type, vous devez désassembler l'objet alloué sur le tas en un entier alloué sur la pile à l'aide d'une opération de conversion de type. ==N'oubliez pas que l'indexeur de l'`ArrayList` renvoie des `System.Objects`, et non des `System.Int32`==**.

```cs
static void WorkWithArrayList()
{
    // Les types valeur sont automatiquement encapsulés lorsqu'ils sont
    // passés à un membre demandant un objet.

    ArrayList myInts = new ArrayList();
    myInts.Add(10);
    myInts.Add(20);
    myInts.Add(35);

    // L'encapsulation a lieu lorsqu'un objet est reconverti en
    // données basées sur une pile.
    int i = (int)myInts[0];

    // Maintenant, il est réencapsulé, car WriteLine() 
    // requiert des types objet !
    Console.WriteLine("Value of your int: {0}", i);
}
```

Notez encore une fois que **le `System.Int32` alloué sur la pile est encapsulé avant l'appel à `ArrayList.Add()`, afin de pouvoir être passé en paramètre le `System.Object` requis**. ==Notez également que le `System.Object` est déballée en `System.Int32` une fois récupéré de l'`ArrayList` via l'opération de conversion, pour être à nouveau emballée lorsqu'il est passé à la méthode `Console.WriteLine()`, car cette méthode opère sur des variables `System.Object`.

>[!important] Le Boxing et `Console.WriteLine` : Ancien vs Nouveau
>
>Bien que le résultat visuel soit identique, la gestion mémoire diffère selon la syntaxe utilisée :
>
>- **Syntaxe Composite :** `Console.WriteLine("Valeur : {0}", i);`
>	- **Boxing :** **OUI**.
>	- **Pourquoi ?** Cette méthode appelle la surcharge `WriteLine(string, object)`. Un `int` (Type Valeur) doit être converti en `object` (Type Référence) pour être accepté. Cela crée une allocation inutile sur le tas (_Heap_).
>- **Interpolation (C# 10+) :** `Console.WriteLine($"Valeur : {i}");`
>	- **Boxing :** **NON** (dans la plupart des cas).
>	- **Pourquoi ?** Le compilateur utilise un **Interpolated String Handler**. Grâce aux génériques (`<T>`), l'entier est traité directement sans jamais être transformé en objet.
>
>**Conseil de performance :** Pour éviter le boxing dans les vieilles versions ou avec la syntaxe `{0}`, utilisez explicitement **`i.ToString()`**. Cela alloue une chaîne, mais évite l'opération coûteuse de boxing de l'entier lui-même.

L'emballage et le déballage sont pratiques du point de vue du programmeur, mais ==cette approche simplifiée du transfert de mémoire pile/tas entraîne des problèmes de performance== (**tant en termes de vitesse d'exécution que de taille du code**) ==et un manque de sécurité des types==. Pour comprendre ces problèmes de performance, considérez les étapes suivantes qui doivent avoir lieu pour emballer et déballer un simple entier :

1. Un nouvel objet doit être alloué sur le tas géré.
2. La valeur des données stockées sur la pile doit être transférée à cet emplacement mémoire.
3. Lors du déballage, la valeur stockée dans l'objet sur le tas doit être transférée à nouveau sur la pile.
4. L'objet désormais inutilisé sur le tas sera (finalement) collecté par le ramasse-miettes.

Bien que cette méthode `WorkWithArrayList()` ne provoque pas de ralentissement majeur en termes de performances, ==son impact pourrait se faire sentir si une `ArrayList` contenait des milliers d'entiers que votre programme manipule régulièrement==. En réalité, **vous pourriez manipuler des données stockées dans une pile, dans un conteneur, sans aucun problème de performance**. Idéalement, ==il serait préférable de ne pas avoir à se soucier de l'extraction de données de ce conteneur à l'aide de blocs `try`/`catch`== (c'est précisément ce que permettent les génériques).

## La question de la sécurité des types

J'ai abordé la question de la sécurité des types lors de la présentation des opérations de déballage. **Rappelons que vous devez déballer vos données dans le même type de données que celui déclaré avant le déballage**. Cependant, ==il existe un autre aspect de la sécurité des types à prendre en compte dans un environnement sans génériques : le fait que **la majorité des classes de `System.Collections` peuvent généralement contenir n'importe quel type de données, car leurs membres sont prototypés pour opérer sur des `System.Objects`**==. Par exemple, cette méthode construit une `ArrayList` de données aléatoires sans lien entre elles :

```cs
static void ArrayListOfRandomObjects()
{
    // L'ArrayList peut contenir n'importe quoi.
    ArrayList allMyObjects = new ArrayList();
    allMyObjects.Add(true);
    allMyObjects.Add(
        new OperatingSystem(PlatformID.MacOSX, new Version(10, 0))
    );
    allMyObjects.Add(66);
    allMyObjects.Add(3.14);
}
```

**Dans certains cas, vous aurez besoin d'un conteneur extrêmement flexible capable de contenir absolument n'importe quoi** (comme illustré ici). Cependant, ==la plupart du temps, vous préférerez un conteneur à *typage statique (type safe)* qui ne peut traiter qu'un type particulier de données==. Par exemple, **vous pourriez avoir besoin d'un conteneur ne pouvant contenir que des connexions à une base de données, des bitmaps ou des objets compatibles avec `IPointy`**.

**Avant l'avènement des génériques, la seule façon de résoudre ce problème de *typage statique* était de créer manuellement une classe de collection personnalisée** (fortement typée). Supposons que vous souhaitiez créer une collection personnalisée ne pouvant contenir que des objets de type `Person`.

```cs
namespace IssuesWithNonGenericCollections;

public class Person
{
    public int Age { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }

    public Person() { }

    public Person(string firstName, string lastName, int age)
    {
        Age = age;
        FirstName = firstName;
        LastName = lastName;
    }

    public override string ToString()
    {
        return $"Name: {FirstName} {LastName}, Age: {Age}";
    }
}
```

==Pour créer une collection ne pouvant contenir que des objets `Person`, **vous pouvez définir une variable membre `ArrayList` de `System.Collections`**==. au sein d'une classe nommée `PersonCollection` et configurer tous les membres pour qu'ils opèrent sur des objets `Person` fortement typés, plutôt que sur des objets `System.Object`. Voici un exemple simple (une collection personnalisée en production pourrait prendre en charge de nombreux membres supplémentaires et étendre une classe de base abstraite de l'espace de noms `System.Collections` ou `System.Collections.Specialized`) :

```cs
using System.Collections;

namespace IssuesWithNonGenericCollections;

public class PersonCollection : IEnumerable
{
    private ArrayList arPeople = new ArrayList();

    // Cast pour l'appelant.
    public Person GetPerson(int pos) => (Person)arPeople[pos];

    // Insérer uniquement des objets Person.
    public void AddPerson(Person p)
    {
        arPeople.Add(p);
    }

    public void ClearPeople()
    {
        arPeople.Clear();
    }

    // Property
    public int Count => arPeople.Count;

    // Prise en charge de l'énumération Foreach.
    public IEnumerator GetEnumerator() => arPeople.GetEnumerator();
}
```

**Notez que la classe `PersonCollection` implémente l'interface `IEnumerable`, permettant une itération de type `foreach` sur chaque élément contenu**. Notez également que ==vos méthodes `GetPerson()` et `AddPerson()` ont été prototypées pour fonctionner uniquement sur des objets `Person`==, et non sur des bitmaps, des chaînes de caractères, des connexions à une base de données ou d'autres types d'éléments. **Ces types étant définis, la sécurité des types est assurée, car le compilateur C# pourra détecter toute tentative d'insertion d'un type de données incompatible**. Mettez à jour les instructions `using` dans *Program.cs* comme suit et ajoutez la méthode `UserPersonCollection()` à la fin de votre code :

```cs
using System.Collections;
using IssuesWithNonGenericCollections;

...

static void UsePersonCollection()
{
    Console.Title = "Custom Person Collection";
    Console.WriteLine("**** Custom Person Collection ****\n");

    PersonCollection myPeople = new PersonCollection();
    myPeople.AddPerson(new Person("Homer", "Simpson", 40));
    myPeople.AddPerson(new Person("Marge", "Simpson", 38));
    myPeople.AddPerson(new Person("Lisa", "Simpson", 9));
    myPeople.AddPerson(new Person("Bart", "Simpson", 7));
    myPeople.AddPerson(new Person("Maggie", "Simpson", 2));

    // Ceci provoquerait une erreur de compilation !
    // myPeople.AddPerson(new Car());
    foreach (Person p in myPeople)
    {
        Console.WriteLine(p);
    }
}
```

==**Bien que les collections personnalisées garantissent la sécurité des types, cette approche vous oblige à créer une collection personnalisée (quasi identique) pour chaque type de données unique que vous souhaitez contenir**==. Ainsi, si vous avez besoin d'une collection personnalisée qui ne fonctionne que sur les classes dérivées de la classe de base `Car`, vous devez créer une classe de collection très similaire.

```cs
using System.Collections;

public class CarCollection : IEnumerable
{
	private ArrayList arCars = new ArrayList();
	
	// Cast pour l'appelant.
	public Car GetCar(int pos) => (Car) arCars[pos];

	// Insérer uniquement des objets Voiture.
	public void AddCar(Car c)
	{
		arCars.Add(c);
	}
	
	public void ClearCars()
	{
		arCars.Clear();
	}
	
	public int Count => arCars.Count;
	
	// Prise en charge de l'énumération Foreach.
	IEnumerator IEnumerable.GetEnumerator() => arCars.GetEnumerator();
}
```

**Cependant, une classe de collection personnalisée ne résout en rien le problème des pénalités liées à l'emballage/désemballage**. ==Même si vous créiez une collection personnalisée nommée `IntCollection` conçue pour fonctionner uniquement avec des éléments de type `System.Int32`, vous devriez allouer un objet pour stocker les données== (par exemple, `System.Array` et `ArrayList`).

```cs
using System.Collections;

public class IntCollection : IEnumerable
{ 
	private ArrayList arInts = new ArrayList();

	// Récupère un entier (effectue un déballage!).
	public int GetInt(int pos) => (int)arInts[pos];
	
	// Insère un entier (effectue un emballage!).
	public void AddInt(int i)
	{
		arInts.Add(i);
	} 
	
	public void ClearInts()
	{
		arInts.Clear();
	
	} 
	
	public int Count => arInts.Count;
	
	IEnumerator IEnumerable.GetEnumerator() => arInts.GetEnumerator();
}
```

**Quel que soit le type choisi pour stocker les entiers, il est impossible d'échapper au dilemme du boxing en utilisant des conteneurs non génériques.

## Premier aperçu des collections génériques `<T>`

**==L'utilisation de classes de collections génériques corrige tous les problèmes précédents, notamment les pénalités liées au boxing/unboxing et le manque de sécurité des types==**. De plus, **la nécessité de créer une classe de collection (générique) personnalisée devient beaucoup plus rare**. ==Plutôt que de devoir créer des classes uniques pouvant contenir des personnes, des voitures et des entiers, vous pouvez utiliser une classe de collection générique et spécifier le type==.

Prenons l'exemple de la méthode suivante (ajoutée à la fin du fichier *Program.cs*), qui utilise la classe générique `List<T>` (dans l'espace de noms `System.Collections.Generic`) pour contenir différents types de données de manière fortement typée. (Ne vous préoccupez pas des détails de la syntaxe générique pour le moment) :

```cs
static void UseGnericList()
{
    // Utilisation d'une syntaxe plus récente
    // (collection expression) comparé au livre
    Console.Title = "Fun with Generics";
    Console.WriteLine("**** Fun with Generics ****\n");

    // Cette List<> ne peut contenir que des objets Person.
    List<Person> morePeople = [new Person("Frank", "Black", 50)];
    Console.WriteLine(morePeople[0]);

    // Cette List<> ne peut contenir que des entiers.
    List<int> moreInts = [10, 2];

    int sum = moreInts[0] + moreInts[1];

    // Erreur de compilation ! Impossible d'ajouter un objet Person
    // à une liste d'entiers !
    // moreInts.Add(new Person());
}
```

Le premier objet `List<T>` ne peut contenir que des objets `Person`. Par conséquent, **vous n'avez pas besoin d'effectuer de conversion de type (cast) lors de la récupération des éléments du conteneur, ce qui rend cette approche plus sûre en termes de types**. Le second objet `List<T>` ne peut contenir que des entiers, tous alloués sur la pile ; autrement dit, il n'y a pas de boxing ou de unboxing caché, contrairement à ce qui se passe avec les `ArrayList` non génériques. ==Voici une liste succincte des avantages offerts par les conteneurs génériques par rapport à leurs homologues non génériques== :

- Les génériques **offrent de meilleures performances** car ils n'entraînent ==pas de pénalités de boxing ou de unboxing lors du stockage de types valeur==.
- Les génériques sont **sûrs en termes de types** car ils ne peuvent contenir que le type spécifié.
- Les génériques **réduisent considérablement le besoin de créer des types de collections personnalisés** car ==vous spécifiez le « type de type » lors de la création du conteneur générique==.

# Le rôle des paramètres de type générique

Vous trouverez des classes, interfaces, structures et délégués génériques dans l'ensemble des bibliothèques de classes de base .NET Core, et ceux-ci peuvent faire partie de n'importe quel espace de noms .NET Core. Sachez également que **les génériques ont bien plus d'utilisations que la simple définition d'une classe de collection**. ==Vous verrez d'ailleurs de nombreux génériques différents utilisés dans la suite de cet ouvrage, pour diverses raisons==.

>[!note] 
>Seules les classes, les structures, les interfaces et les délégués peuvent être écrits de manière générique ; les types énumérés ne le peuvent pas.

**Lorsque vous voyez un élément générique listé dans la documentation .NET Core** ou dans l'explorateur d'objets de Visual Studio, **vous remarquerez une paire de chevrons contenant une lettre ou un autre jeton**. L'image suivante illustre l'explorateur d'objets de Visual Studio présentant plusieurs éléments génériques situés dans l'espace de noms `System.Collections.Generic`, notamment la classe `List<T>` mise en évidence.

![[Figure 10.1.png|Éléments génériques prenant en charge les paramètres de type]]

**Formellement, ces jetons sont appelés paramètres de type** ; cependant, pour simplifier, ==on peut les appeler simplement des espaces réservés==. Le symbole `<T>` se lit «de T». Ainsi, `IEnumerable<T>` se lit «`IEnumerable` de T» ou, autrement dit, «`IEnumerable` de type T».

>[!tip] Bonnes pratiques
>**Le nom d'un paramètre de type** (espace réservé) **est sans importance** et son choix revient au développeur qui a créé l'élément générique. ==Cependant, on utilise généralement *T* pour représenter les types, *TKey* ou *K* pour les clés et *TValue* ou *V* pour les valeurs==.

Lorsque vous créez un objet générique, implémentez une interface générique ou appelez un membre générique, **il vous incombe de fournir une valeur au paramètre de type**. Vous trouverez de nombreux exemples dans ce chapitre et tout au long de la suite du texte. Pour commencer, examinons les bases de l'interaction avec les types et les membres génériques.

## Spécification des paramètres de type pour les classes/structures génériques

Lorsque vous créez une instance d'une classe ou d'une structure générique, vous spécifiez le paramètre de type lors de la déclaration de la variable et lors de l'appel du constructeur. Comme vous l'avez vu dans l'exemple de code précédent, `UseGenericList()` a défini deux objets `List<T>`.

```cs
// Cette List<> ne peut contenir que des objets Person.
List<Person> morePeople = new List<Person>();

// Cette List<> ne peut contenir que des entiers.
List<int> moreInts = new List<int>();
```

Vous pouvez lire la première ligne de l'extrait précédent comme « une `List<>` de `T`, où `T` est de type `Person` ». Ou, plus simplement, vous pouvez la lire comme « ==une liste d'objets `Person`== ». **Une fois le paramètre de type d'un élément générique spécifié, il ne peut plus être modifié** (n'oubliez pas que les génériques garantissent la sécurité des types). ==Lorsque vous spécifiez un paramètre de type pour une classe ou une structure générique, toutes les occurrences de l'espace réservé sont remplacées par la valeur que vous avez fournie==.

Si vous consultiez la déclaration complète de la classe générique `List<T>` à l'aide de l'explorateur d'objets de Visual Studio, vous verriez que l'espace réservé `T` est utilisé dans toute la définition du type `List<T>`. Voici un extrait :

```cs
// Un listing partiel de la classe List<T>
namespace System.Collections.Generic;
public class List<T> : IList<T>, IList, IReadOnlyList<T>
{
	...
	
	public void Add(T item);
	public void AddRange(IEnumerable<T> collection);
	public ReadOnlyCollection<T> AsReadOnly();
	public int BinarySearch(T item);
	public bool Contains(T item);
	public void CopyTo(T[] array);
	public int FindIndex(System.Predicate<T> match);
	public T FindLast(System.Predicate<T> match);
	public bool Remove(T item);
	public int RemoveAll(System.Predicate<T> match);
	public T[] ToArray();
	public bool TrueForAll(System.Predicate<T> match);
	public T this[int index] { get; set; }
}
```

Lorsque vous créez une `List<T>` spécifiant des objets `Person`, c'est comme si le type `List<T>` était défini comme suit :

```cs
namespace System.Collections.Generic;

public class List<Person>
	: IList<Person>, IList, IReadOnlyList<Person>
{
	...
	
	public void Add(Person item);
	public void AddRange(IEnumerable<Person> collection);
	public ReadOnlyCollection<Person> AsReadOnly();
	public int BinarySearch(Person item);
	public bool Contains(Person item);
	public void CopyTo(Person[] array);
	public int FindIndex(System.Predicate<Person> match);
	public Person FindLast(System.Predicate<Person> match);
	public bool Remove(Person item);
	public int RemoveAll(System.Predicate<Person> match);
	public Person[] ToArray();
	public bool TrueForAll(System.Predicate<Person> match);
	public Person this[int index] { get; set; }
}
```

**Bien sûr, lorsque vous créez une variable générique `List<T>`, le compilateur ne crée pas littéralement une nouvelle implémentation de la classe `List<T>`. Il accède uniquement aux membres du type générique que vous utilisez effectivement**.

## Spécification des paramètres de type pour les membres génériques

**Il est tout à fait acceptable qu'une classe ou une structure non générique prenne en charge des propriétés génériques**. ==Dans ce cas, vous devrez également spécifier la valeur d'espace réservé lors de l'appel de la méthode==. **==Par exemple, `System.Array` prend en charge plusieurs méthodes génériques. Plus précisément, la méthode statique non générique `Sort()` possède désormais une version générique nommée `Sort<T>()`==**. Prenons l'exemple de l'extrait de code suivant, où `T` est de type `int` :

```cs
int[] myInts = { 10, 4, 2, 33, 93 };
// Spécifie l'espace réservé pour la méthode générique
// Sort<>().
Array.Sort<int>(myInts);
foreach (int i in myInts)
{
    Console.WriteLine(i);
}
```

>Le compilateur choisit automatiquement la version générique sans que l'on ait à l'écrire. Ce qui veut dire que le l'appel à `Sort()` peut être écrit comme suit, tout en utilisant la version générique sous le capot: `Array.Sort(myInts)`;

## Spécification des paramètres de type pour les interfaces génériques

**Il est courant d'implémenter des interfaces génériques lors de la création de classes ou de structures devant prendre en charge divers comportements du framework** (par exemple, le clonage, le tri et l'énumération). Au [[Chapitre 8#Les interfaces `IEnumerable` et `IEnumerator`|Chapitre 8]], vous avez découvert plusieurs interfaces non génériques, telles que `IComparable`, `IEnumerable`, `IEnumerator` et `IComparer`. Rappelons que l'interface non générique `IComparable` était définie comme suit :

```cs
public interface IComparable
{
	int CompareTo(object obj);
}
```

Au [[Chapitre 8#L'interface `IComparable`|Chapitre 8]], vous avez également implémenté cette interface sur votre classe `Car` pour permettre le tri dans un tableau standard. **Cependant, le code nécessitait plusieurs vérifications à l'exécution et opérations de conversion car le paramètre était un `System.Object`.**

```cs
public class Car : IComparable
{
	...
	
	int IComparable.CompareTo(object obj)
	{
	    if (obj is Car temp)
	    {
	        return CarID.CompareTo(temp.CarID);
	    }
	    throw new ArgumentException("parameter is not a Car!");
	}
}
```

Supposons maintenant que vous utilisiez l'équivalent générique de cette interface.

```cs
public interface IComparable<T>
{
	int CompareTo(T obj);
}
```

Dans ce cas, votre code d'implémentation sera considérablement épuré.

```cs
public class Car : IComparable<Car>
{
	
	...
	
    // Implémentation de IComparable<Car>
    int IComparable<Car>.CompareTo(Car obj)
    {
        if (this.CarID > obj.CarID)
        {
            return 1;
        }
        else if (this.CarID < obj.CarID)
        {
            return -1;
        }
        return 0;
    }
}
```

**Ici, il n'est pas nécessaire de vérifier si le paramètre entrant est un `Car`, car il ne peut s'agir que d'un object `Car`**. ==Si quelqu'un transmettait un type de données incompatible, une erreur de compilation se produirait==. Maintenant que vous comprenez mieux comment interagir avec les éléments génériques, ainsi que le rôle des paramètres de type (aussi appelés espaces réservés), vous êtes prêt à examiner les classes et les interfaces de l'espace de noms `System.Collections.Generic`.

# L'espace de noms `System.Collections.Generic`

Lorsque vous développez une application .NET Core et que vous avez besoin de gérer des données en mémoire, les classes de `System.Collections.Generic` répondront très probablement à vos besoins. ==Au début de ce chapitre, j'ai brièvement mentionné certaines des interfaces non génériques principales implémentées par les classes de collections non génériques==. Sans grande surprise, **l'espace de noms `System.Collections.Generic` définit des remplacements génériques pour bon nombre d'entre elles**.

En fait, **vous pouvez trouver un certain nombre d'interfaces génériques qui étendent leurs homologues non génériques**. Cela peut sembler étrange ; cependant, **==ce faisant, les classes implémentant ces interfaces prendront également en charge les fonctionnalités héritées présentes dans leurs homologues non génériques==**. Par exemple, `IEnumerable<T>` étend `IEnumerable`. Le tableau 10-4 répertorie les interfaces génériques principales que vous rencontrerez lorsque vous travaillerez avec les classes de collections génériques.

##### Tableau 10-4: Interfaces clés prises en charge par les classes de `System.Collections.Generic`

| Interfaces de `System.Collections.Generic`<br> | Description                                                                                                                                                                        |
| ---------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ICollection<T>`                               | Définit les caractéristiques générales (par exemple, la taille, l'énumération et la sécurité des threads) pour tous les types de collections génériques.                           |
| `IComparer<T>`                                 | Définit une méthode de comparaison avec des objets.                                                                                                                                |
| `IDictionary<TKey, TValue>`                    | Permet à un objet de collection générique de représenter son contenu à l'aide de paires clé-valeur.                                                                                |
| `IEnumerable<T>`<br>`IAsyncEnumerable<T>`      | Renvoie l'interface `IEnumerator<T>` pour un objet donné. `IAsyncEnumerable` (nouveauté de C# 8.0) est abordée au [[Chapitre 15#Flux asynchrones (Nouveauté C 8.0)\|Chapitre 15]]. |
| `IEnumerator<T>`                               | Permet une itération de type `foreach` sur une collection générique.                                                                                                               |
| `IList<T>`                                     | Fournit un comportement permettant d'ajouter, de supprimer et d'indexer des éléments dans une liste séquentielle d'objets.                                                         |
| `ISet<T>`                                      | Fournit un comportement permettant d'ajouter, de supprimer et d'indexer des éléments dans une liste séquentielle d'objets.                                                         |

**L’espace de noms `System.Collections.Generic` définit également plusieurs classes qui implémentent bon nombre de ces interfaces clés**. Le [[#Tableau 10-5 Les classes de `System.Collections.Generic`|Tableau 10-5]] décrit certaines classes couramment utilisées de cet espace de noms, les interfaces qu’elles implémentent et leurs fonctionnalités de base.

##### Tableau 10-5 Les classes de `System.Collections.Generic`

| Classe Générique                 | Interfaces clés prises en charge                                                                                         | Description                                                                               |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------- |
| `Dictionary<TKey, TValue>`       | `ICollection<T>`, `IDictionary<TKey, TValue>`, `IEnumerable<T>`                                                          | Ceci représente une collection générique de clés et de valeurs.                           |
| `LinkedList<T>`                  | `ICollection<T>`, `IEnumerable<T>`                                                                                       | Ceci représente une liste doublement chaînée.                                             |
| `List<T>`                        | `ICollection<T>`, `IEnumerable<T>`, `IList<T>`                                                                           | Il s'agit d'une liste séquentielle d'éléments redimensionnable dynamiquement.             |
| `Queue<T>`                       | `ICollection` (il ne s'agit pas d'une faute de frappe ; c'est l'interface de collection non générique), `IEnumerable<T>` | Il s'agit d'une implémentation générique d'une liste premier entré, premier sorti (FIFO). |
| `SortedDictionary<TKey, TValue>` | `ICollection<T>`, `IDictionary<TKey, TValue>`, `IEnumerable<T>`                                                          | Il s'agit d'une implémentation générique d'un ensemble trié de paires clé-valeur.         |
| `SortedSet<T>`                   | `ICollection<T>`, `IEnumerable<T>`, `ISet<T>`                                                                            | Ceci représente une collection d'objets maintenue dans un ordre trié, sans doublons.      |
| `Stack<T>`                       | `ICollection` (il ne s'agit pas d'une faute de frappe ; c'est l'interface de collection non générique), `IEnumerable<T>` | Il s'agit d'une implémentation générique d'une liste dernier entré, premier sorti. (LIFO) |

>[!info] Note sur une nouveauté de C#11 / .NET 7
>Bien que les interfaces n'aient pas changé, Microsoft a ajouté des **interfaces de portée plus fine** dans les versions récentes pour améliorer la flexibilité sans briser les contrats : 
>
>- `IReadOnlyCollection<T>` : Implémentée par toutes les classes citée dans le [[#Tableau 10-5 Les classes de `System.Collections.Generic`|Tableau 10-5]], elle permet d'accéder à `Count` de manière générique sans les méthodes de modification interdites.
>- **Optimisations internes** : Dans .NET 7/8, des méthodes comme `Enqueue` ont été optimisées au niveau du runtime pour être encore plus rapides, mais la structure des interfaces est restée stable pour éviter de casser des millions de lignes de code existantes.

>[!tip] Le conseil d'architecture "Pro" (Crée avec Gemini)
Dans le livre, vous verrez souvent des exemples où des méthodes renvoient des `List<T>`. En pratique professionnelle :
>
>- **En entrée** : Utilisez `IEnumerable<T>` (si vous ne lisez qu'une fois) ou `IReadOnlyCollection<T>` (si vous avez besoin de la taille).
>- **En sortie** : Renvoyez `IReadOnlyCollection<T>` pour garantir aux utilisateurs de votre code qu'ils ne peuvent pas modifier vos données internes.
>
>```cs
>// Exemple de bonne pratique
>public IReadOnlyCollection<string> GetNames() 
>{
>	return _internalList; // _internalList est une List<string>
>}
>```

==L'espace de noms `System.Collections.Generic` définit également de nombreuses classes et structures auxiliaires qui fonctionnent conjointement avec un conteneur spécifique==. Par exemple, le type `LinkedListNode<T>` représente un nœud au sein d'une `LinkedList<T>` générique, l'exception `KeyNotFoundException` est levée lors de la tentative de récupération d'un élément d'un conteneur à l'aide d'une clé inexistante, etc. **Consultez la [documentation .NET Core](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic?view=net-10.0) pour plus de détails sur l'espace de noms `System.Collections.Generic`**.

Votre prochaine étape consiste à apprendre à utiliser certaines de ces classes de collections génériques. Avant cela, permettez-moi toutefois de vous **présenter une fonctionnalité du langage C# (introduite pour la première fois dans .NET 3.5) qui simplifie la manière d'alimenter les conteneurs de collections génériques (et non génériques) avec des données**.

## Comprendre la syntaxe d'initialisation des collections

Au [[Chapitre 4#Examen de la syntaxe d'initialisation des tableaux C|Chapitre 4]], vous avez découvert la *syntaxe d'initialisation des objets*, qui vous ==permet de définir les propriétés d'une nouvelle variable lors de sa création==. La syntaxe d'*initialisation des collections* est étroitement liée à cette notion. **Cette fonctionnalité du langage C# permet de remplir de nombreux conteneurs** (tels que `ArrayList` ou `List<T>`) **avec des éléments en utilisant une syntaxe similaire à celle utilisée pour remplir un tableau simple**. Créez une nouvelle application console .NET Core nommée *FunWithCollectionInitialization*. Supprimez le code généré dans *Program.cs* et ajoutez les instructions `using` suivantes :

```cs
using System.Collections;
using System.Drawing;
```

>[!warning] Attention 
>Vous ne pouvez appliquer la syntaxe d'initialisation de collection qu'aux classes qui prennent en charge une méthode `Add()`, qui est formalisée par les interfaces `ICollection<T>`/`ICollection`.
>>[!tip] Cette note n'est plus exacte.
>>Pour qu'une classe puisse utiliser la syntaxe `{ val1, val2 }`, elle n'a besoin que de deux choses :
>>
>>1. Implémenter **`IEnumerable`** (pour que le compilateur sache que c'est une collection).
>>2. Avoir une méthode **`Add`** accessible (soit une méthode d'instance, soit, et c'est la nouveauté, une **méthode d'extension**).
>>
>>==Le cas des Expressions de Collection (C# 12)==
>>
>>Si vous utilisez une version très récente (au-delà de ce que couvre probablement votre édition C# 10), C# 12 a introduit les **Collection Expressions** `[]`.
>>```cs
>>List<int> list = [1, 2, 3];
>>int[] array = [1, 2, 3];
>>Span<int> span = [1, 2, 3];
>>```
>>Cette nouvelle syntaxe ne repose pas du tout sur la méthode `Add()`, mais sur des mécanismes internes beaucoup plus performants (comme le "Inline Array").
>
>>[!tldr] Ce qu'il faut retenir :  
>>- **Historiquement** : Il fallait `ICollection`.
>>- **Techniquement (Aujourd'hui)** : Il faut `IEnumerable` + une méthode nommée `Add`.
>>- **L'exception Queue/Stack** : Comme nous l'avons vu plus haut, `Queue<T>` et `Stack<T>` n'ont pas de méthode `Add` (elles ont `Enqueue`/`Push`). Par conséquent, **vous ne pouvez pas** utiliser la syntaxe `{ }` avec elles, même si elles sont dans `System.Collections.Generic` !
>>	```cs
>>	// ❌ ERREUR : Queue ne possède pas de méthode Add
>>	Queue<int> q = new Queue<int> { 1, 2, 3 }; 
>>	```

Considérez les exemples suivants :

```cs
// Initialisation d'un tableau standard.
int[] myArrayOfInts = { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };

// Initialisation d'une List<> générique d'entiers.
List<int> myGenericList = new List<int> { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };

// Initialisation d'une ArrayList contenant des données numériques.
ArrayList myList = new ArrayList { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };
```

**==Si votre conteneur gère une collection de classes ou une structure, vous pouvez combiner la syntaxe d'initialisation d'objets avec la syntaxe d'initialisation de collections pour obtenir du code fonctionnel==**. Vous vous souvenez peut-être de la classe `Point` du [[Chapitre 5#Comprendre l'initialisation des objets|Chapitre 5]], qui définissait deux propriétés nommées `X` et `Y`. Si vous vouliez créer une `List<T>` générique d'objets `Point`, vous pourriez écrire ce qui suit :

```cs
List<Point> myListOfPoints = new List<Point>
{
	new Point { X = 2, Y = 2 },
	new Point { X = 3, Y = 3 },
	new Point { X = 4, Y = 4 }
};
foreach (var pt in myListOfPoints)
{
	Console.WriteLine(pt);
}
```

**L’avantage de cette syntaxe est qu’elle vous permet d’économiser de nombreuses frappes**. Bien que les accolades imbriquées puissent rendre la lecture difficile si la mise en forme ne vous importe pas, ==imaginez la quantité de code nécessaire pour remplir la liste de rectangles suivante (`List<T>`) sans la syntaxe d’initialisation des collections.==

```cs
List<Rectangle> myListOfRects = new List<Rectangle>
{
	new Rectangle {
	  Height = 90, Width = 90,
	  Location = new Point { X = 10, Y = 10 }},
	new Rectangle {
	  Height = 50,Width = 50,
	  Location = new Point { X = 2, Y = 2 }},
};
foreach (var r in myListOfRects)
{
	Console.WriteLine(r);
}
```

## Utilisation de la classe `List<T>`

Créez un nouveau projet d'application console nommé *FunWithGenericCollections.* Ajoutez un nouveau fichier, nommé *Person.cs,* et ajoutez le code suivant (qui est identique au code de la classe *Person* précédente) :

```cs
namespace FunWithGenericCollections;

public class Person
{
    public int Age { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }

    public Person() { }

    public Person(string firstName, string lastName, int age)
    {
        Age = age;
        FirstName = firstName;
        LastName = lastName;
    }

    public override string ToString()
    {
        return $"Name: {FirstName} {LastName}, Age: {Age}";
    }
}
```

Supprimez le code généré dans *Program.cs* et ajoutez l'instruction using suivante :

```cs
using FunWithGenericCollections;
```

La première classe générique que vous allez examiner est `List<T>`, que vous avez déjà vue une ou deux fois dans ce chapitre. **La classe `List<T>` sera sans doute le type que vous utiliserez le plus fréquemment dans l'espace de noms `System.Collections.Generic`**. ==Ce type vous permet de redimensionner dynamiquement le contenu du conteneur==. Pour illustrer les bases de ce type, considérez la méthode suivante dans votre fichier *Program.cs*, qui utilise `List<T>` pour manipuler l'ensemble d'objets `Person` présenté précédemment dans ce chapitre ; vous vous souvenez peut-être que ces objets `Person` définissaient trois propriétés (`Age`, `FirstName` et `LastName`) et une implémentation personnalisée de `ToString()` :

```cs
using FunWithGenericCollections;

UseGenericList();

static void UseGenericList()
{
    // Créer une liste d'objets Person, remplie avec
    // la syntaxe d'initialisation de collection/objet.
    List<Person> people = new List<Person>()
    {
        new Person
        {
            FirstName = "Homer",
            LastName = "Simpson",
            Age = 47,
        },
        new Person
        {
            FirstName = "Marge",
            LastName = "Simpson",
            Age = 45,
        },
        new Person
        {
            FirstName = "Lisa",
            LastName = "Simpson",
            Age = 9,
        },
        new Person
        {
            FirstName = "Bart",
            LastName = "Simpson",
            Age = 8,
        },
    };

    // Affiche le # d'objet dans la liste.
    Console.WriteLine($"Items in list: {people.Count}");

    // Énumérer la liste.
    foreach (Person p in people)
    {
        Console.WriteLine(p);
    }

    // Insère un nouvel objet Person
    Console.WriteLine("\n-> Inserting new person.");
    people.Insert(
        2,
        new Person
        {
            FirstName = "Maggie",
            LastName = "Simpson",
            Age = 2,
        }
    );
    Console.WriteLine($"Items in list: {people.Count}");

    // Copie les données dans un nouveau tableau.
    Person[] arrayOfPeople = people.ToArray();
    foreach (Person p in arrayOfPeople)
    {
        Console.WriteLine($"First Names: {p.FirstName}");
    }
}
```

Ici, vous utilisez la syntaxe d'initialisation de collection pour remplir votre `List<T>` avec des objets, ==comme une notation abrégée pour appeler `Add()` *plusieurs fois*==. Après avoir affiché le nombre d'éléments dans la collection (et énuméré chaque élément), **vous appelez `Insert()`. Comme vous pouvez le voir, `Insert()` vous permet d'insérer un nouvel élément dans la `List<T>` à un index spécifié.**

Enfin, **notez l'appel à la méthode `ToArray()`, qui renvoie un tableau d'objets `Person` basé sur le contenu de la `List<T>` d'origine**. À partir de ce tableau, vous parcourez à nouveau les éléments en utilisant la syntaxe d'indexation du tableau. Si vous appelez cette méthode depuis vos instructions de niveau supérieur, vous obtenez la sortie suivante :

```
**** Fun with Generic Collection ****

Items in list: 4
Name: Homer Simpson, Age: 47
Name: Marge Simpson, Age: 45
Name: Lisa Simpson, Age: 9
Name: Bart Simpson, Age: 8

-> Inserting new person.
Items in list: 5
First Names: Homer
First Names: Marge
First Names: Maggie
First Names: Lisa
First Names: Bart
```

La classe `List<T>` définit de nombreux autres membres intéressants ; consultez la [documentation](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.list-1?view=net-10.0) pour plus d’informations. Ensuite, examinons quelques collections plus génériques, notamment `Stack<T>`, `Queue<T>` et `SortedSet<T>`. Cela devrait vous permettre de bien comprendre vos options de base pour stocker les données de votre application personnalisée.

## Utilisation de la classe `Stack<T>`

**La classe `Stack<T>` représente une collection qui gère ses éléments selon le principe du dernier entré, premier sorti (LIFO)**. Comme vous pouvez vous y attendre, **==`Stack<T>` définit des méthodes nommées `Push()` et `Pop()` pour ajouter ou retirer des éléments de la pile==**. La méthode suivante crée une pile d'objets `Person` :

```cs
static void UseGenericStack()
{
    Stack<Person> stackOfPeople = new();
    stackOfPeople.Push(
        new Person
        {
            FirstName = "Homer",
            LastName = "Simpson",
            Age = 47,
        }
    );
    stackOfPeople.Push(
        new Person
        {
            FirstName = "Marge",
            LastName = "Simpson",
            Age = 45,
        }
    );
    stackOfPeople.Push(
        new Person
        {
            FirstName = "Lisa",
            LastName = "Simpson",
            Age = 9,
        }
    );

    // Maintenant, regardez l'élément du haut, faites-le sauter,
    // puis regardez à nouveau.
    Console.WriteLine($"First person is: {stackOfPeople.Peek()}");
    Console.WriteLine($"Popped off {stackOfPeople.Pop()}");
    Console.WriteLine($"\nFirst person is: {stackOfPeople.Peek()}");
    Console.WriteLine($"Popped off {stackOfPeople.Pop()}");
    Console.WriteLine($"\nFirst person is: {stackOfPeople.Peek()}");
    Console.WriteLine($"Popped off {stackOfPeople.Pop()}");

    try
    {
        Console.WriteLine($"First person is: {stackOfPeople.Peek()}");
        Console.WriteLine($"Popped off {stackOfPeople.Pop()}");
    }
    catch (InvalidOperationException ex)
    {
        Console.WriteLine($"\nError! {ex.Message}");
    }
}
```

Ici, vous créez une pile contenant trois personnes, ajoutées dans l'ordre de leurs prénoms : Homer, Marge et Lisa. **Lorsque vous consultez la pile, vous verrez toujours l'objet du haut en premier** ; par conséquent, ==le premier appel à `Peek()` révèle le troisième objet `Person`==. Après une série d'appels à `Pop()` et `Peek()`, la pile finit par se vider, auquel cas tout appel supplémentaire à `Peek()` et `Pop()` lève une exception système. Vous pouvez voir le résultat ici :

```
**** Fun with Generic Collection ****

...

First person is: Name: Lisa Simpson, Age: 9
Popped off Name: Lisa Simpson, Age: 9

First person is: Name: Marge Simpson, Age: 45
Popped off Name: Marge Simpson, Age: 45

First person is: Name: Homer Simpson, Age: 47
Popped off Name: Homer Simpson, Age: 47

Error! Stack empty.
```

## Utilisation de la classe `Queue<T>`

**Les `Queues` sont des conteneurs qui garantissent un accès aux éléments selon le principe du premier arrivé, premier servi**. Malheureusement, nous autres humains, sommes soumis aux files d'attente toute la journée : à la banque, au cinéma, au café le matin. Si vous devez modéliser un scénario où les éléments sont traités selon le principe du premier arrivé, premier servi, **la classe `Queue<T>` répondra parfaitement à vos besoins**. Outre les fonctionnalités offertes par les interfaces prises en charge, la classe `Queue` définit les membres clés présentés dans le [[#Tableau 10-6 Membres du type `Queue<T>`|Tableau 10-6]].

##### Tableau 10-6: Membres du type `Queue<T>`

| Sélection de membres de  `Queue<T>` | Description                                                               |
| ----------------------------------- | ------------------------------------------------------------------------- |
| `Dequeue()`                         | Supprime et renvoie l'objet au début de `Queue<T>`                        |
| `Enqueue()`                         | Ajoute un objet à la fin de `Queue<T>`                                    |
| `Peek()`                            | Renvoie l'objet situé au début de la file (`Queue<T>`) sans le supprimer. |

Passons maintenant à la pratique. Vous pouvez commencer par réutiliser votre classe `Person` et créer un objet `Queue<T>` qui simule une file d'attente de personnes souhaitant commander un café.

```cs
void UseGenericQueue()
{
    // Crée une Queue avec 3 personnes.
    Queue<Person> peopleQ = new();
    peopleQ.Enqueue(
        new Person
        {
            FirstName = "Homer",
            LastName = "Simpson",
            Age = 47,
        }
    );
    peopleQ.Enqueue(
        new Person
        {
            FirstName = "Marge",
            LastName = "Simpson",
            Age = 45,
        }
    );
    peopleQ.Enqueue(
        new Person
        {
            FirstName = "Lisa",
            LastName = "Simpson",
            Age = 9,
        }
    );

    // Aperçu de la première personne dans la file.
    Console.WriteLine($"{peopleQ.Peek().FirstName} is first in line!");

    // Retirer chaque Person de la file.
    GetCoffee(peopleQ.Dequeue());
    GetCoffee(peopleQ.Dequeue());
    GetCoffee(peopleQ.Dequeue());

    // Essai d'encore retirer un élément dans la file?
    try
    {
        GetCoffee(peopleQ.Dequeue());
    }
    catch (InvalidOperationException e)
    {
        Console.WriteLine($"Error! {e.Message}");
    }

    static void GetCoffee(Person p)
    {
        Console.WriteLine("{0} got coffee!", p.FirstName);
    }
}
```

Ici, vous insérez trois éléments dans la classe `Queue<T>` à l'aide de sa méthode `Enqueue()`. L'appel à `Peek()` vous permet de visualiser (mais pas de supprimer) le premier élément présent dans la file d'attente. Enfin, l'appel à `Dequeue()` supprime l'élément de la file et l'envoie à la fonction auxiliaire `GetCoffee()` pour traitement. **Notez que si vous tentez de supprimer des éléments d'une file d'attente vide, une exception d'exécution est levée**. Voici le résultat que vous obtenez lors de l'appel de cette méthode :

```
**** Fun with Generic Collection ****
...

Homer is first in line!
Homer got coffee!
Marge got coffee!
Lisa got coffee!
Error! Queue empty.
```

## Utilisation de la classe `PriorityQueue<TElement, TPriority>` (Nouveauté C# 10)

**Introduite dans .NET 6/C# 10**, la classe `PriorityQueue` fonctionne comme la classe `Queue<T>`, à **la différence que chaque élément en file d'attente se voit attribuer une priorité**. ==Lorsque des éléments sont retirés de la file d'attente, ils sont extraits de la priorité la plus basse à la plus élevée==. L'exemple suivant met à jour l'exemple précédent de file d'attente pour utiliser une `PriorityQueue` :

```cs
void UsePriorityQueue()
{
    Console.WriteLine("* Fun with Generic Priority Queues *\n");

    PriorityQueue<Person, int> peopleQ = new();

    peopleQ.Enqueue(
        new Person
        {
            FirstName = "Lisa",
            LastName = "Simpson",
            Age = 9,
        },
        1
    );
    peopleQ.Enqueue(
        new Person
        {
            FirstName = "Homer",
            LastName = "Simpson",
            Age = 47,
        },
        3
    );
    peopleQ.Enqueue(
        new Person
        {
            FirstName = "Marge",
            LastName = "Simpson",
            Age = 45,
        },
        3
    );
    peopleQ.Enqueue(
        new Person
        {
            FirstName = "Bart",
            LastName = "Simpson",
            Age = 12,
        },
        2
    );

    while (peopleQ.Count > 0)
    {
        Console.WriteLine(peopleQ.Dequeue().FirstName); //Affiche Lisa
        Console.WriteLine(peopleQ.Dequeue().FirstName); //Affiche Bart
        Console.WriteLine(peopleQ.Dequeue().FirstName); //Affiche Marge ou Homer
        Console.WriteLine(peopleQ.Dequeue().FirstName); //Affiche l'autre élément de priorité 3
    }
}
```

**Si plusieurs éléments ont la priorité la plus basse, l'ordre de défilement n'est pas garanti**. Comme le montre l'exemple de code, ==le troisième appel à `Dequeue()` renverra soit `Homer`, soit `Marge`, car ils ont tous deux une priorité de $3$==. Le quatrième appel renverra alors l'autre personne. **Si l'ordre exact est important, vous devez vous assurer que les valeurs de chaque priorité sont uniques**.

## Utilisation de la classe `SortedSet<T>`

**La classe `SortedSet<T>` est utile car elle garantit automatiquement le tri des éléments de l'ensemble lors de l'insertion ou de la suppression d'éléments**. Cependant, ==vous devez indiquer précisément à la classe `SortedSet<T>` comment vous souhaitez trier les objets, en lui passant comme argument du constructeur un objet implémentant l'interface générique `IComparer<T>`==.

Commencez par créer une nouvelle classe nommée `SortPeopleByAge`, implémentant `IComparer<T>`, où `T` est de type `Person`. **Rappelons que cette interface définit une unique méthode nommée `Compare()`, dans laquelle vous pouvez implémenter la logique de comparaison nécessaire**. Voici une implémentation simple de cette classe :

```cs
namespace FunWithGenericCollections;

class SortPeopleByAge : IComparer<Person>
{
    public int Compare(Person firstPerson, Person secondPerson)
    {
        if (firstPerson?.Age > secondPerson?.Age)
        {
            return 1;
        }
        else if (firstPerson?.Age < secondPerson?.Age)
        {
            return -1;
        }
        return 0;
    }
}
```

Ajoutez maintenant la nouvelle méthode suivante qui illustre l'utilisation de `SortedSet<Person>` :

```cs
static void UseSortedSet()
{
    Console.WriteLine("* Fun with Generic Sorted Set *\n");

    // Créez des personnages d'âges différents.
    SortedSet<Person> setOfPeople = new SortedSet<Person>(new SortPeopleByAge())
    {
        new Person
        {
            FirstName = "Homer",
            LastName = "Simpson",
            Age = 47,
        },
        new Person
        {
            FirstName = "Marge",
            LastName = "Simpson",
            Age = 45,
        },
        new Person
        {
            FirstName = "Lisa",
            LastName = "Simpson",
            Age = 9,
        },
        new Person
        {
            FirstName = "Bart",
            LastName = "Simpson",
            Age = 8,
        },
    };

    // Notez que les objets sont triés par âge !
    foreach (Person p in setOfPeople)
    {
        Console.WriteLine(p);
    }
    Console.WriteLine();

    // Ajouter quelques nouveaux personnages, d'âges variés.
    setOfPeople.Add(
        new Person
        {
            FirstName = "Saku",
            LastName = "Jones",
            Age = 1,
        }
    );
    setOfPeople.Add(
        new Person
        {
            FirstName = "Mikko",
            LastName = "Jones",
            Age = 32,
        }
    );

    // Toujours triée par âge !
    foreach (Person p in setOfPeople)
    {
      Console.WriteLine(p);
    }
    Console.WriteLine();
}
```

**Lorsque vous exécutez votre application, la liste des objets est désormais toujours triée en fonction de la valeur de la propriété `Age`, quel que soit l'ordre dans lequel vous avez inséré ou supprimé des objets**.

```
**** Fun with Generic Collection ****

...

* Fun with Generic Sorted Set *

Name: Bart Simpson, Age: 8
Name: Lisa Simpson, Age: 9
Name: Marge Simpson, Age: 45
Name: Homer Simpson, Age: 47

Name: Saku Jones, Age: 1
Name: Bart Simpson, Age: 8
Name: Lisa Simpson, Age: 9
Name: Mikko Jones, Age: 32
Name: Marge Simpson, Age: 45
Name: Homer Simpson, Age: 47
```

>[!warning] Il n'est pas possible de modifié le tri utilisé une fois que l'ensemble est initialisé !
>Comme l'implémentation sous-jacente utilise un *Red-Black Tree* (voir les notes Boot.Dev), il compare à chaque ajout d'élément. L'arbre deviendrait invalide si on changeait la méthode de tri alors que des éléments sont déjà trié dans l'ensemble.

## Utilisation de la classe `Dictionary<TKey, TValue>`

**Une autre collection générique pratique est le type `Dictionary<TKey, TValue>`, qui permet de stocker un nombre quelconque d'objets, identifiables par une clé unique**. Ainsi, ==au lieu d'obtenir un élément d'une `List<T>` à l'aide d'un identifiant numérique== (par exemple, « Donnez-moi le deuxième objet »), **==vous pouvez utiliser la clé textuelle unique (par exemple, « Donnez-moi l'objet que j'ai identifié comme Homer »)==**

Comme pour les autres objets de collection, **vous pouvez remplir un `Dictionary<TKey, TValue>` en appelant manuellement la méthode générique `Add()`**. Cependant, ==vous pouvez également le remplir en utilisant la syntaxe d'initialisation de collection==. Soyez conscient que lors du remplissage de cet objet de collection, **les noms de clés doivent être uniques. Si vous spécifiez par erreur la même clé plusieurs fois, une exception d'exécution sera levée.**

Considérez la méthode suivante qui remplit un `Dictionary<K, V>` avec différents objets. Notez que lors de la création de l'objet `Dictionary<TKey,TValue>`, vous spécifiez le type de clé (`TKey`) et le type d'objet sous-jacent (`TValue`) comme arguments du constructeur. Dans cet exemple, vous utilisez un `string` comme clé et un objet de type `Person` comme valeur. **Notez également que vous pouvez combiner la syntaxe d'initialisation d'objet avec celle d'initialisation de collection.**

```cs
static void UseDictionary()
{
    Console.WriteLine("* Fun with Generic Dictionaries *\n");
    // Remplir à l'aide de la méthode Add()
    Dictionary<string, Person> peopleA = new Dictionary<string, Person>();
    peopleA.Add(
        "Homer",
        new Person
        {
            FirstName = "Homer",
            LastName = "Simpson",
            Age = 47,
        }
    );
    peopleA.Add(
        "Marge",
        new Person
        {
            FirstName = "Marge",
            LastName = "Simpson",
            Age = 45,
        }
    );
    peopleA.Add(
        "Lisa",
        new Person
        {
            FirstName = "Lisa",
            LastName = "Simpson",
            Age = 9,
        }
    );

    // Récupérer Homer
    Person homer = peopleA["Homer"];
    Console.WriteLine(homer);

    // Peupler avec la syntaxe d'initialisation
    Dictionary<string, Person> peopleB = new Dictionary<string, Person>()
    {
        {
            "Homer",
            new Person
            {
                FirstName = "Homer",
                LastName = "Simpson",
                Age = 47,
            }
        },
        {
            "Marge",
            new Person
            {
                FirstName = "Marge",
                LastName = "Simpson",
                Age = 45,
            }
        },
        {
            "Lisa",
            new Person
            {
                FirstName = "Lisa",
                LastName = "Simpson",
                Age = 9,
            }
        },
    };

    // Récupère Lisa
    Person lisa = peopleB["Lisa"];
    Console.WriteLine(lisa);
    Console.WriteLine();
}
```

**Il est également possible de remplir un `Dictionary<TKey,TValue>` à l'aide d'une syntaxe d'initialisation spécifique à ce type de conteneur** (appelée, sans surprise, *initialisation de dictionnaire*). ==Comme pour la syntaxe utilisée pour remplir l'objet `personB` dans l'exemple de code précédent, vous définissez toujours une portée d'initialisation pour l'objet collection==. **Cependant, vous pouvez utiliser l'indexeur pour spécifier la clé et l'assigner à un nouvel objet**, comme ceci :

```cs
// Initialisation du dictionnaire.
Dictionary<string, Person> peopleC = new Dictionary<string, Person>()
{
    ["Homer"] = new Person
    {
        FirstName = "Homer",
        LastName = "Simpson",
        Age = 47,
    },
    ["Marge"] = new Person
    {
        FirstName = "Marge",
        LastName = "Simpson",
        Age = 45,
    },
    ["Lisa"] = new Person
    {
        FirstName = "Lisa",
        LastName = "Simpson",
        Age = 9,
    },
};
```

>[!important] Quelle est la différence subtile entre les deux ? (Gemini)
>
>Bien que le résultat final soit souvent identique, le compilateur ne traduit pas ces deux syntaxes de la même manière en arrière-plan :
>
>- **La syntaxe par indexation (`["clé"] = ...`)** appelle le dictionnaire de cette façon : `dico["clé"] = valeur;`. Si une clé identique existe déjà lors de l'initialisation, elle va simplement **écraser** la valeur précédente sans planter.
>- **La syntaxe par accolades (`{ "clé", ... }`)** appelle en réalité la méthode `.Add("clé", valeur)`. Si vous mettez par mégarde deux fois la même clé dans votre liste d'initialisation, le programme lèvera immédiatement une exception (`ArgumentException`) au démarrage.

# L'espace de noms `System.Collections.ObjectModel`

Maintenant que vous comprenez comment utiliser les principales classes génériques, ==nous allons examiner brièvement un espace de noms supplémentaire centré sur les collections== : `System.Collections.ObjectModel`. **Cet espace de noms est relativement petit et contient quelques classes**. Le [[#Tableau 10-7 Membres utiles de `System.Collections.ObjectModel`|Tableau 10-7]] répertorie **les deux classes que vous devriez absolument connaître**.

##### Tableau 10-7: Membres utiles de `System.Collections.ObjectModel`

| Types `System.collections.ObjectModel` | Description                                                                                                                                                                |
| -------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ObservableCollection<T>`<br><br>      | Représente une collection de données dynamiques qui fournissent des notifications quand des éléments sont ajoutés, sont retirés ou quand la liste complète est rafraîchie. |
| `ReadOnlyObservableCollection<T>`      | Représente une version en lecture seule de `ObservableCollection<T>`                                                                                                       |

**La classe `ObservableCollection<T>` est utile, car elle a la capacité d'informer les objets externes lorsque son contenu a changé d'une manière ou d'une autre** (comme vous pouvez le deviner, travailler avec `ReadOnlyObservableCollection<T>` est similaire mais en lecture seule).

## Utilisation de `ObservableCollection<T>`

Créez un nouveau projet d'application console nommé *FunWithObservableCollections* et importez l'espace de noms `System.Collections.ObjectModel` dans votre fichier de code C# initial. **À bien des égards, l'utilisation d'`ObservableCollection<T>` est identique à celle de `List<T>`, car ces deux classes implémentent les mêmes interfaces de base**. ==La particularité de la classe `ObservableCollection<T>` réside dans la prise en charge d'un événement nommé `CollectionChanged`==. Cet événement est déclenché lorsqu'un nouvel élément est inséré, lorsqu'un élément est supprimé (ou déplacé), ou lorsque la collection entière est modifiée.

>Les événements et délégués sont expliqué en détail dans le [[Chapitre 12|Chapitre 12]]

**Comme tout événement, `CollectionChanged` est défini à l'aide d'un délégué, en l'occurrence `NotifyCollectionChangedEventHandler`**. Ce délégué peut appeler toute méthode prenant un objet comme premier paramètre et un `NotifyCollectionChangedEventArgs` comme second. Considérez le code suivant, qui remplit une collection observable contenant des objets `Person` et configure l'événement `CollectionChanged` :

```cs
using System.Collections.ObjectModel;
using System.Collections.Specialized;
using FunWithObservableCollections;

Console.Title = "Fun with Observable Collections";
Console.WriteLine("**** Fun with Observable Collections ****\n");

// Créez une collection à observer
// et ajoutez quelques objets Personne.
ObservableCollection<Person> people = new ObservableCollection<Person>()
{
    new Person
    {
        FirstName = "Peter",
        LastName = "Murphy",
        Age = 52,
    },
    new Person
    {
        FirstName = "Kevin",
        LastName = "Key",
        Age = 48,
    },
};

// Associer l'événement CollectionChanged.
people.CollectionChanged += people_CollectionChanged;

static void people_CollectionChanged(
    object sender,
    NotifyCollectionChangedEventArgs e
)
{
    throw new NotImplementedException();
}
```

**Le paramètre entrant `NotifyCollectionChangedEventArgs` définit deux propriétés importantes : `OldItems` et `NewItems`**. ==Elles vous fournissent la liste des éléments présents dans la collection avant le déclenchement de l’événement, et celle des nouveaux éléments concernés par la modification==. Toutefois, **il est conseillé d’examiner ces listes uniquement dans les circonstances appropriées.** Rappelons que l’événement `CollectionChanged` peut se déclencher lors de l’ajout, la suppression, le déplacement ou la réinitialisation d’éléments. **Pour identifier l’action qui a déclenché l’événement, vous pouvez utiliser la propriété `Action` de `NotifyCollectionChangedEventArgs`**. ==Cette propriété peut être comparée à n’importe quel membre de l’énumération `NotifyCollectionChangedAction` :==

```cs
public enum NotifyCollectionChangedAction
{
	Add = 0,
	Remove = 1,
	Replace = 2,
	Move = 3,
	Reset = 4,
}
```

Voici une implémentation du gestionnaire d'événements `CollectionChanged` qui parcourra les anciens et nouveaux ensembles lorsqu'un élément a été inséré ou supprimé de la collection en question (notez l'utilisation de `System.Collections.Specialized`) :

```cs
using System.Collections.Specialized;

...

static void people_CollectionChanged(
    object sender,
    NotifyCollectionChangedEventArgs e
)
{
    // Quelle action a provoqué l'événement ?
    Console.WriteLine($"Action for this event: {e.Action}");

    // Ils ont retiré quelque chose
    if (e.Action == NotifyCollectionChangedAction.Remove)
    {
        Console.WriteLine("Here are the OLD items:");
        foreach (Person p in e.OldItems)
        {
            Console.WriteLine(p);
        }
        Console.WriteLine();
    }
    // Ils ont ajouté quelque chose
    else if (e.Action == NotifyCollectionChangedAction.Add)
    {
        // Afficher maintenant les NOUVEAUX éléments insérés.
        Console.WriteLine("Here are the NEW items:");
        foreach (Person p in e.NewItems)
        {
            Console.WriteLine(p);
        }
        Console.WriteLine();
    }
}
```

Maintenant, mettez à jour votre code d'appel pour ajouter et supprimer un élément.

```cs
// Ajouter un nouvel élément.
people.Add(new Person("Fred", "Smith", 32));

// Supprimer un élément.
people.RemoveAt(0);
```

Lorsque vous exécuterez le programme, vous obtiendrez un résultat similaire à celui-ci :

```
**** Fun with Observable Collections ****

Action for this event: Add
Here are the NEW items:
Name: Fred Smith, Age: 32

Action for this event: Remove
Here are the OLD items:
Name: Peter Murphy, Age: 52
```

>[!tip]- l'événement `CollectionChanged` ne renvoie pas l'état complet de la liste, mais seulement le **"diff"** (la différence) causé par l'action qui vient d'avoir lieu.
>Pour voir toute la collection dans la méthode de rappel (*callback*) `people_CollectionChanged()`
>```cs
>static void people_CollectionChanged(object sender, NotifyCollectionChangedEventArgs e)
>{
>	Console.WriteLine($"Action: {e.Action}");
>
>	// On "cast" le sender pour récupérer la collection complète
>	var fullCollection = sender as ObservableCollection<Person>;
>
>	Console.WriteLine("Current state of the WHOLE collection:");
>	foreach (var p in fullCollection)
>	{
>		Console.WriteLine($" -> {p.FirstName}");
>	}
>}
>```

>[!warning] Un détail important sur le `null`
>
>Faites attention : selon l'action, certaines propriétés de `e` sont **nulles**.
>
>- Si `Action == Add`, `e.OldItems` est **null**.
>- Si `Action == Remove`, `e.NewItems` est **null**.  
>
>**C'est pour cela que le livre utilise des `if/else` ou un `switch`.**

Ceci conclut l'examen des différents espaces de noms centrés sur les collections. Pour terminer ce chapitre, vous allez maintenant étudier comment créer vos propres méthodes génériques et types génériques personnalisés.

# Création de méthodes génériques personnalisées

Bien que la plupart des développeurs utilisent généralement les types génériques existants dans les bibliothèques de classes de base, il est également possible de **créer vos propres membres génériques et types génériques personnalisés**. Voyons comment intégrer des génériques personnalisés à vos projets. ==La première étape consiste à créer une méthode d'échange générique==. Commencez par créer une nouvelle application console nommée *CustomGenericMethods*.

**Lorsque vous créez des méthodes génériques personnalisées, vous obtenez une version améliorée de la surcharge de méthodes traditionnelle**. Au [[Chapitre 4#Comprendre la surcharge de méthodes|Chapitre 4]], vous avez appris que la surcharge consiste à définir plusieurs versions d'une même méthode, qui diffèrent par le nombre ou le type de paramètres.

==Bien que la surcharge soit une fonctionnalité utile dans un langage orienté objet, un problème se pose== : **on peut facilement se retrouver avec une multitude de méthodes qui font essentiellement la même chose**. Par exemple, supposons que vous ayez besoin de créer des méthodes permettant d'échanger deux données à l'aide d'une simple routine d'échange. Vous pourriez commencer par créer une nouvelle classe statique avec une méthode pouvant manipuler des entiers, comme ceci :

```cs
namespace CustomGenericMethods;

static class SwapFunctions
{
    // Échange 2 entiers.
    static void Swap(ref int a, ref int b)
    {
        // Utilisation de tuple plutôt que de
        // créer une variable temporaire
        (a, b) = (b, a);
    }
}
```

Jusqu'ici tout va bien. ==Mais supposons maintenant que vous ayez également besoin d'échanger deux objets `Person` ; cela nécessiterait la création d'une nouvelle version de `Swap()`.==

```cs
// Échange 2 objet Person
static void Swap(ref Person a, ref Person b)
{
    (a, b) = (b, a);
}
```

Vous voyez sans doute où je veux en venir. **Si vous deviez également échanger des nombres à virgule flottante, des bitmaps, des voitures, des boutons, etc., vous devriez créer encore plus de méthodes, ce qui deviendrait un véritable cauchemar de maintenance**. Vous pourriez créer une seule méthode (non générique) qui opère sur des paramètres d'objet, mais vous vous retrouveriez alors confronté à tous les problèmes que nous avons examinés précédemment dans ce chapitre, notamment le boxing, l'unboxing, le manque de sécurité des types, les conversions explicites, etc.

**==Lorsqu'un groupe de méthodes surchargées ne diffère que par ses arguments d'entrée, c'est le signe que les génériques peuvent vous simplifier la vie==**. Prenons l'exemple de la méthode générique `Swap<T>()` suivante, qui peut échanger deux types `T` quelconques :

```cs
// Cette méthode échangera 2 object de n'importe quel type
// comme spécifié par le paramètre de type <T>.
static void Swap<T>(ref T a, ref T b)
{
    Console.WriteLine("You sent the Swap() method a {0}", typeof(T));
    // Utilisation de tuple plutôt que de
    // créer une variable temporaire
    (a, b) = (b, a);
}
```

**Remarquez comment une méthode générique est définie en spécifiant le type des paramètres après le nom de la méthode, mais avant la liste des paramètres**. Ici, ==vous indiquez que la méthode `Swap<T>()` peut opérer sur deux paramètres quelconques de type `<T>`==. Pour plus de clarté, vous affichez également le nom du type de l'espace réservé fourni dans la console à l'aide de l'opérateur `typeof()` de C#. Considérez maintenant le code d'appel suivant, qui échange des entiers et des chaînes de caractères :

```cs
using CustomGenericMethods;

Console.Title = "Fun with Custom Generic Methods";
Console.WriteLine("**** Fun with Custom Generic Methods ****\n");

// Échange deux entiers
int a = 10,
    b = 90;
Console.WriteLine("Before swap: {0}, {1}", a, b);
SwapFunctions.Swap<int>(ref a, ref b);
Console.WriteLine("After swap: {0}, {1}", a, b);
Console.WriteLine();

// Échange 2 chaînes.
string s1 = "Hello",
    s2 = "There";
Console.WriteLine("Before swap: {0} {1}!", s1, s2);
SwapFunctions.Swap<string>(ref s1, ref s2);
Console.WriteLine("After swap: {0} {1}!", s1, s2);
```

Le résultat ressemble à ceci :

```
**** Fun with Custom Generic Methods ****

Before swap: 10, 90
You sent the Swap() method a System.Int32
After swap: 90, 10

Before swap: Hello There!
You sent the Swap() method a System.String
After swap: There Hello!
```

## Inférence des paramètres de type

**Lorsque vous appelez des méthodes génériques telles que `Swap<T>`, vous pouvez omettre le paramètre de type si (et seulement si) la méthode générique requiert des arguments, car le compilateur peut inférer le paramètre de type à partir des paramètres membres**. Par exemple, vous pouvez échanger deux valeurs `System.Boolean` en ajoutant le code suivant à vos instructions de niveau supérieur :

```cs
// Le compilateur déduira System.Boolean.
bool b1 = true,
    b2 = false;
Console.WriteLine($"Before swap: {b1}, {b2}");
SwapFunctions.Swap(ref b1, ref b2);
Console.WriteLine($"After swap: {b1}, {b2}");
```

Même si le compilateur peut déterminer le paramètre de type correct en fonction du type de données utilisé pour déclarer `b1` et `b2`, ~~vous devriez prendre l'habitude de toujours spécifier explicitement le paramètre de type~~ (détails sur ceci plus loin).

```cs
SwapFunctions.Swap<bool>(ref b1, ref b2);
```

Cela indique clairement à vos collègues programmeurs que cette méthode est bien générique. De plus, **l'inférence des paramètres de type ne fonctionne que si la méthode générique possède au moins un paramètre**. Par exemple, supposons que vous ayez la méthode générique suivante dans votre fichier *Program.cs* :

```cs
static void DisplayBaseClass<T>()
{
    // BaseType est une méthode utilisée en réflexion,
    // qui sera étudiée au chapitre 17
    Console.WriteLine(
        "Base class of {0} is: {1}.",
        typeof(T),
        typeof(T).BaseType
    );
}
```

Dans ce cas, **==vous devez fournir le paramètre de type lors de l'appel==**.

```cs

...

// Doit fournir un paramètre de type si
// la méthode ne prend pas de paramètres.
DisplayBaseClass<int>();
DisplayBaseClass<string>();

// Erreur de compilation ! Aucun paramètre ? Doit fournir un espace réservé !
// DisplayBaseClass();
Console.ReadLine();
```

Bien entendu, les méthodes génériques n'ont pas besoin d'être statiques comme dans ces exemples. Toutes les règles et options des méthodes non génériques s'appliquent également.

>[!warning] Grosse différences de philosophie avec le C# moderne (avec Gemini)
>**L'habitude de tout spécifier explicitement s'est largement perdue au profit de la concision.**
>
>### 1. Pourquoi le livre donne ce conseil ?
>
>- **Pédagogie** : En tant qu'apprenant, il veut que vous soyez conscient de ce que le compilateur fait. Écrire `<int>` vous force à "voir" la mécanique générique.
>- **Clarté académique** : Dans un livre, cela évite toute confusion pour le lecteur qui ne verrait pas immédiatement le type des variables `b1` et `b2` déclarées dix lignes plus haut.
>
>### 2. Pourquoi les développeurs modernes ne le font plus ?
>
>- **L'arrivée de Roslyn** : Depuis C# 6, le compilateur est devenu extrêmement performant pour nous dire "Hé, c'est redondant !". Roslyn indique aux IDE de griser systématiquement ces paramètres de type, poussant les développeurs à les supprimer pour nettoyer le code.
>- **Moins de bruit visuel** : Dans un projet professionnel de 100 000 lignes de code, supprimer les types redondants réduit la charge cognitive. On se concentre sur l'intention du code, pas sur sa tuyauterie.
>- **L'évolution de la philosophie du langage** : C# a beaucoup évolué vers une syntaxe plus légère (comme le mot-clé `var` ou les _target-typed new_). L'inférence est maintenant considérée comme un pilier de la productivité.
>
>### 3. La règle d'or en entreprise
>
>Aujourd'hui, la norme dans la plupart des équipes de développement (et dans les guides de style comme ceux de Microsoft ou Google) est la suivante :
>
>- **Laissez le compilateur faire le travail.**
>- N'écrivez le type que si l'inférence échoue ou si le code devient vraiment ambigu à lire pour un humain.

# Création de structures et de classes génériques personnalisées

Maintenant que vous savez comment définir et appeler des méthodes génériques, il est temps de vous intéresser à la construction d'une structure générique (le processus de création d'une classe générique est identique) au sein d'un nouveau projet d'application console nommé *GenericPoint*. Supposons que vous ayez créé une structure `Point` générique prenant en charge un paramètre de type unique représentant le stockage sous-jacent des coordonnées (*x*, *y*). L'appelant peut alors créer des types `Point<T>` comme suit :

```cs
using GenericPoint;

// Point utilisant des ints
Point<int> p = new Point<int>(10, 10);

// Point utilisant des doubles
Point<double> p2 = new Point<double>(5.4, 3.3);

// Point utilisant des strings
Point<string> p3 = new Point<string>("i","3i");
```

==Créer un point à l'aide de chaînes de caractères peut sembler étrange au premier abord, mais prenons le cas des nombres imaginaires==. Dans ce cas, utiliser des chaînes de caractères pour les valeurs `X` et `Y` d'un point peut se justifier. Quoi qu'il en soit, cela illustre la puissance des génériques. Voici la définition complète de `Point<T>` :

```cs
namespace GenericPoint;

// Une structure Point générique
// Les champs
public struct Point<T> {

    // Donnée d'état générique
    private T _xPos;
    private T _yPos;

    // Constructeur générique
    public Point(T xVal, T yVal)
    {
        _xPos = xVal;
        _yPos = yVal;
    }

    // Propriétés génériques
    public T X { get => _xPos; set => _xPos = value; }

    public T Y { get => _yPos; set => _yPos = value; }

    public override string ToString() => $"[{_xPos}, {_yPos}]";
}
```

Comme vous pouvez le constater, `Point<T>` exploite son paramètre de type dans la définition des données de champ, des arguments du constructeur et des définitions de propriétés.

## Expressions de valeur par défaut avec les génériques

**Avec l'introduction des génériques, le mot-clé `default` en C# possède une double fonction.** ==Outre son utilisation dans une instruction `switch`, il permet d'attribuer à un paramètre de type sa valeur par défaut==. **Ceci est utile car un type générique ne connaît pas les valeurs réelles des paramètres à l'avance, et ne peut donc pas présumer avec certitude de la valeur par défaut**. Les valeurs par défaut d'un paramètre de type sont les suivantes :

- **Les valeurs numériques ont pour valeur par défaut `0`.**
- **Les types référence ont pour valeur par défaut `null`.**
- **Les champs d'une structure sont initialisés à `0`  pour les types valeur ou à `null` pour les types référence.**

==Pour réinitialiser une instance de `Point<T>`, vous pouvez définir directement les valeurs `X` et `Y` à 0==. **Ceci suppose que l'appelant fournisse uniquement des données numériques.** Qu'en est-il de la version chaîne ? **==C'est là que la syntaxe `default(T)` s'avère utile. Le mot-clé `default` réinitialise une variable à la valeur par défaut de son type de données==**. Ajoutez une méthode appelée `ResetPoint()` comme suit :

```cs
// Réinitialise les champs à la valeur par défaut du paramètre de type.
// Le mot-clé « default » est surchargé en C#.
// Lorsqu'il est utilisé avec les génériques, il représente
// la valeur par défaut d'un paramètre de type.
public void ResetPoint()
{
    _xPos = default(T);
    _yPos = default(T);
}
```

Maintenant que vous avez la méthode `ResetPoint()` en place, vous pouvez pleinement utiliser les méthodes de la structure `Point<T>`.

```cs
using GenericPoint;

Console.Title = "Fun with Generic Structures";
Console.WriteLine("***** Fun with Generic Structures *****\n");

// Point utilisant des ints.
Point<int> p = new Point<int>(10, 10);
Console.WriteLine("p.ToString()={0}", p.ToString());
p.ResetPoint();
Console.WriteLine("p.ToString()={0}", p.ToString());
Console.WriteLine();

// Point utilisant des doubles.
Point<double> p2 = new Point<double>(5.4, 3.3);
Console.WriteLine("p2.ToString()={0}", p2.ToString());
p2.ResetPoint();
Console.WriteLine("p2.ToString()={0}", p2.ToString());
Console.WriteLine();

// Point utilisant des strings.
Point<string> p3 = new Point<string>("i", "3i");
Console.WriteLine("p3.ToString()={0}", p3.ToString());
p3.ResetPoint();
Console.WriteLine("p3.ToString()={0}", p3.ToString());

Console.ReadLine();
```

Voici le résultat :

```
**** Fun with Generic Structures ****

p.ToString()=[10, 10]
p.ToString()=[0, 0]

p2.ToString()=[5.4, 3.3]
p2.ToString()=[0, 0]

p3.ToString()=[i, 3i]
p3.ToString()=[, ]
````

## Expressions littérales par défaut (Nouveauté C# 7.1)

**Outre la définition de la valeur par défaut d'une propriété, C# 7.1 a introduit les expressions littérales par défaut. Cela permet de se dispenser de spécifier le type de la variable dans l'instruction par défaut**. Mettez à jour la méthode `ResetPoint()` comme suit :

```cs
public void ResetPoint()
{
    _xPos = default;
    _yPos = default;
}
```

**L'expression par défaut ne se limite pas aux variables simples, mais peut également s'appliquer aux types complexes.** Par exemple, pour créer et initialiser la structure `Point`, vous pouvez écrire :

```cs
// Utilisation default
Point<string> p4 = default;
Console.WriteLine("p4.ToString()={0}", p4.ToString());
Console.WriteLine();
Point<int> p5 = default;
Console.WriteLine("p5.ToString()={0}", p5.ToString());
```

## Correspondance de modèles avec les génériques (Nouveauté C# 7.1)

Une autre nouveauté de C# 7.1 est **la possibilité d'effectuer une correspondance de modèles avec les génériques**. Prenons l'exemple de la méthode suivante, qui vérifie le type de données sur lequel est basée l'instance `Point` (ceci est certes incomplet, mais suffisant pour illustrer le concept) :

```cs
static void PatternMatching<T>(Point<T> p)
{
    switch (p)
    {
        case Point<string> pString:
            Console.WriteLine("Point is based on {0}", typeof(T));
            return;
        case Point<int> pInt:
            Console.WriteLine("Point is based on {0}", typeof(T));
            return;
    }
}
```

Pour tester le code de correspondance de modèles, mettez à jour les instructions de niveau supérieur comme suit :

```cs
// Utilisation default
Point<string> p4 = default;
Point<int> p5 = default;
PatternMatching(p4);
PatternMatching(p5);
```

## Manipulation de types générique nullable

 Les classes ou les types `Nullable<T>` peuvent avoir comme valeur pour `T` `null`.

Imaginez une méthode générique qui cherche une valeur dans une liste. Si elle ne trouve rien, vous voulez renvoyer une valeur de sécurité.

```cs
public T GetValueOrDefault<T>(T? input)
{
    // Si input est null, on renvoie la valeur par défaut du type T (0, null, false...)
    return input ?? default; 
}
```

L'opérateur **`??`** (null-coalescing) sert à dire : _"Si la valeur à gauche est nulle, utilise celle de droite"_. Combiné avec `default`, il permet de sécuriser le retour d'une méthode générique.

Ceci est d'une grande utilité car elle permet d'**éviter les exceptions**. ==Au lieu de laisser un code planter sur une `NullReferenceException`, vous forcez un retour "propre"== (ex: `0` pour un `int`). De plus cette opérateur permet **plus de flexibilité**. ==Comme vous ne savez pas si `T` est une classe (type référence) ou une structure (type valeur), `default` s'adapte automatiquement.==

### Le piège du "Nullable" (nouveauté C# 8.0)

Depuis C# 8, il y a une subtilité. ==Si vous n'avez pas de **contraintes** sur votre type `T`, le compilateur peut râler car il ne sait pas si `T` accepte d'être `null`.==

C'est pour cela que vous verrez souvent ceci juste après dans votre livre :

```cs
public T FindElement<T>(IEnumerable<T> collection) where T : class
{
    T element = // ... logique de recherche ...
    return element ?? default; // Ici default sera forcément null car T est une classe
}
```

sachez qu'il existe aussi l'**opérateur d'affectation** :

```cs
_xPos ??= default; // Si _xPos est null, on lui assigne la valeur par défaut.
```


# Les contraintes de paramètres de type

Comme l'illustre ce chapitre, **tout élément générique possède au moins un paramètre de type que vous devez spécifier lors de votre interaction avec ce type ou membre générique**. Cela vous permet à lui seul de développer du code sûr en termes de types; cependant, **==vous pouvez également utiliser le mot-clé `where` pour définir avec une extrême précision les caractéristiques d'un paramètre de type donné==**.

Grâce à ce mot-clé, **vous pouvez ajouter un ensemble de contraintes à un paramètre de type donné, que le compilateur C# vérifiera à la compilation**. Plus précisément, vous pouvez contraindre un paramètre de type comme décrit dans le [[#Tableau 10-8 Contraintes possibles pour les paramètres de types générique|Tableau 10-8]].

##### Tableau 10-8: Contraintes possibles pour les paramètres de types générique

| Contraintes de générique     | Description                                                                                                                                                                                                                                                                                          |
| ---------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `where T : struct`           | Le paramètre de type `<T>` doit être un **type valeur non-nullable** (ce qui implique d'avoir `System.ValueType` dans sa chaîne d'héritage, d'être alloué sur la pile et d'avoir une sémantique de valeur pure), à l'exclusion des énumérations.                                                     |
| `where T : class`            | Le paramètre de type `<T>` doit être un **type référence** (classe, interface, délégué ou tableau). Cela garantit que `T` peut être assigné à `null` et qu'il est géré par le Garbage Collector sur le tas (Heap).                                                                                   |
| `where T : Enum`             | *Nouveauté C# 7.2*: Le paramètre de type `<T>` doit être une **énumération**. Cela garantit que `T` est un type valeur représentant un ensemble de constantes nommées et permet d'accéder aux méthodes statiques de la classe `System.Enum` sans transtypage (casting).\|                            |
| `where T : new()`            | Le paramètre de type `<T>` doit posséder un **constructeur public sans paramètres**. Cette contrainte est indispensable pour pouvoir écrire `new T()` à l'intérieur de la classe générique. Elle doit obligatoirement figurer en **dernière position** dans la liste des contraintes d'un même type. |
| `where T :  NomClasseDeBase` | Le paramètre de type `<T>` doit être dérivé de la classe spécifiée par `NomDeLaClasseDeBase`.                                                                                                                                                                                                        |
| `where T : NomInterface`     | Le paramètre de type `<T>` doit implémenter l'interface spécifiée par `NomInterface`. Vous pouvez séparer plusieurs interfaces par une liste délimitée par des virgules.                                                                                                                             |

~~À moins d'avoir besoin de créer des collections personnalisées extrêmement sûres en termes de types, vous n'aurez probablement jamais besoin d'utiliser le mot-clé `where` dans vos projets C#.~~ Quoi qu'il en soit, les quelques exemples de code (partiels) suivants illustrent comment utiliser le mot-clé `where`.

## Exemples d'utilisation du mot-clé `where`

Supposons que vous ayez créé une classe générique personnalisée et que ==vous souhaitiez vous assurer que le type du paramètre possède un constructeur par défaut==. Cela peut s'avérer utile ==lorsque la classe générique personnalisée doit créer des instances de `T`, car le constructeur par défaut est le seul constructeur potentiellement commun à tous les types==. De plus, contraindre `T` de cette manière permet une vérification à la compilation; **si `T` est un type référence, le programmeur a pensé à redéfinir le constructeur par défaut dans la définition de la classe** (vous vous souviendrez peut-être que le constructeur par défaut est supprimé dans les classes lorsque vous définissez le vôtre).

```cs
// MyGenericClass hérite de object, tandis que
// les éléments contenus doivent avoir un constructeur par défaut.
public class MyGenericClass<T> where T : new()
{
	...
}
```

Notez que la clause `where` spécifie le type du paramètre soumis à contrainte, suivie de l'opérateur deux-points. ==**Après l'opérateur deux-points, vous listez chaque contrainte possible**== (ici, un constructeur par défaut). Voici un autre exemple :

```cs
// MyGenericClass hérite de object, tandis que
// les éléments contenus doivent être une classe implémentant IDrawable
// et doivent prendre en charge un constructeur par défaut.
public class MyGenericClass<T> where T : class, IDrawable, new()
{
	...
}
```

Dans ce cas, **`T` doit répondre à trois exigences**. Premièrement, ==il doit s'agir d'un type référence== (et non d'une structure), comme indiqué par le jeton `class`. Deuxièmement, `T` ==doit implémenter l'interface `IDrawable`==. Troisièmement, il ==doit également posséder un constructeur par défaut==. **Les contraintes multiples sont listées sous forme de liste séparée par des virgules; cependant, il est important de noter que ==la contrainte `new()` doit toujours être listée en dernier==** ! Par conséquent, le code suivant ne compilera pas :

```cs
// Erreur ! La contrainte new() doit être listée en dernier !
public class MyGenericClass<T> where T : new(), class, IDrawable
{
	...
}
```

**Si vous créez une classe de collection générique personnalisée qui spécifie plusieurs paramètres de type, vous pouvez spécifier un ensemble unique de contraintes pour chacun, en utilisant des clauses `where` distinctes.**

```cs
// <K> doit étendre SomeBaseClass et posséder un constructeur par défaut,
// tandis que <T> doit être une structure et implémenter l'interface
// générique IComparable.
public class MyGenericClass<K, T> where K : SomeBaseClass, new()
	where T : struct, IComparable<T>
{
	...
}
```

~~Vous rencontrerez rarement des cas où vous devrez créer une classe de collection générique entièrement personnalisée~~; cependant, vous **pouvez également utiliser le mot-clé `where` sur les méthodes génériques**. Par exemple, si vous souhaitez spécifier que votre méthode générique `Swap<T>()` ne peut opérer que sur des structures, vous modifierez la méthode comme suit :

```cs
// Cette méthode échange toute structure, mais pas les classes.
static void Swap<T>(ref T a, ref T b) where T : struct
{
	...
}
```

Notez que ==si vous deviez contraindre la méthode `Swap()` de cette manière, vous ne pourriez plus échanger des objets `string` (comme le montre l'exemple de code) car `string` est un type référence.==

## Absence de contraintes sur les opérateurs

>[!success] Toute cette explication du livre n'est plus correcte depuis C# 11 / .NET 7
>Tout les détails à la fin de la section. 
>

Je souhaite faire une dernière remarque concernant les méthodes génériques et les contraintes, en conclusion de ce chapitre. Vous serez peut-être surpris d'apprendre que **lors de la création de méthodes génériques, une erreur de compilation se produira si vous appliquez des opérateurs C# (`+`, `-`, `*`, etc.) aux paramètres de type**. Imaginez par exemple l'utilité d'une classe capable d'additionner, de soustraire, de multiplier et de diviser des types génériques.

```cs
// Erreur de compilation ! Impossible d'appliquer
// des opérateurs aux paramètres de type !
public class BasicMath<T>
{ 
	public T Add(T arg1, T arg2)
	{ return arg1 + arg2; }
	public T Subtract(T arg1, T arg2)
	{ return arg1 - arg2; }
	public T Multiply(T arg1, T arg2)
	{ return arg1 * arg2; }
	public T Divide(T arg1, T arg2)
	{ return arg1 / arg2; }
}
```

Malheureusement, la classe `BasicMath` précédente ne compilera pas. Bien que cela puisse sembler une restriction majeure, il faut se rappeler que les génériques sont génériques. Bien entendu, les données numériques peuvent être utilisées avec les opérateurs binaires de C#. Cependant, à titre d'exemple, si `<T>` était une classe ou une structure personnalisée, le compilateur pourrait supposer que la classe prend en charge les opérateurs `+`, `-` , `*` et `/`. Idéalement, C# permettrait de contraindre un type générique par les opérateurs pris en charge, comme dans cet exemple :

```cs
// Code illustratif uniquement !
public class BasicMath<T> where T : operator +, operator -,
operator *, operator /
{ 
	public T Add(T arg1, T arg2)
	{ return arg1 + arg2; }
	public T Subtract(T arg1, T arg2)
	{ return arg1 - arg2; }
	public T Multiply(T arg1, T arg2)
	{ return arg1 * arg2; }
	public T Divide(T arg1, T arg2)
	{ return arg1 / arg2; }
}
```

~~Malheureusement, les contraintes d'opérateurs ne sont pas prises en charge par la version actuelle de C#.~~ Cependant, **==il est possible d'obtenir l'effet souhaité en définissant une interface qui prend en charge ces opérateurs (les interfaces C# peuvent définir des opérateurs !) et en spécifiant ensuite une contrainte d'interface de la classe générique==**. Quoi qu'il en soit, ceci conclut notre première approche de la création de types génériques personnalisés. Au [[Chapitre 12|Chapitre 12]], je reprendrai le sujet des génériques en examinant le type délégué.

### Les voeux de l'auteur ont été exhaussé depuis C# 11

Ce qui était impossible dans votre livre est devenu une réalité grâce aux **Generic Math**.

Microsoft a introduit les **Static Abstracts in Interfaces**. Cela permet de définir des opérateurs dans des interfaces. Désormais, vous pouvez écrire votre classe `BasicMath` comme ceci :

```cs
// T doit être un nombre (int, double, decimal, etc.)
public class BasicMath<T> where T : INumber<T>
{
    public T Add(T arg1, T arg2) => arg1 + arg2;
    public T Multiply(T arg1, T arg2) => arg1 * arg2;
}
```

Depuis C# 11, .NET met à disposition la nouvelle interface `INumber<T>`, qui regroupe tous les opérateurs mathématiques (`+`, `-`, `*`, `/`). Cela fonctionne sans aucun boxing. C'est aussi rapide que si vous écriviez du code spécifique pour `int` ou `float`. La classe `BasicMath` fonctionne maintenant instantanément pour tous les types numériques du framework et même pour vos propres types personnalisés s'ils implémentent `INumber`.

# Résumé du chapitre

Ce chapitre a commencé par **examiner les types de collections non génériques de `System.Collections` et `System.Collections.Specialized`**, et ==notamment les différents problèmes associés à de nombreux conteneurs non génériques, tels que le manque de sécurité des types et la surcharge d'exécution liée aux opérations d'emballage et de déballage==. Comme mentionné précédemment, **c'est précisément pour ces raisons que les programmes .NET modernes utilisent généralement les classes de collections génériques disponibles dans `System.Collections.Generic` et `System.Collections.ObjectModel`**.

Comme vous l'avez vu, **==un élément générique vous permet de spécifier des espaces réservés==** (paramètres de type) **==lors de la création de l'objet (ou de son appel, dans le cas des méthodes génériques)==**. Bien que vous utilisiez le plus souvent les types génériques fournis dans les bibliothèques de classes de base .NET, **vous pourrez également créer vos propres types génériques (et méthodes génériques). Ce faisant, vous pouvez spécifier autant de contraintes que nécessaire (à l'aide du mot-clé `where`) afin d'accroître la sécurité des types et de garantir que les opérations s'effectuent sur des types dont les caractéristiques sont connues et qui possèdent certaines capacités de base**.

Enfin, n'oubliez pas que ***==les génériques sont présents à de nombreux endroits dans les bibliothèques de classes de base .NET==***. Ici, nous ==nous sommes concentrés sur les collections génériques==. Cependant, **en parcourant la suite de ce livre** (et en explorant la plateforme par vous-même), **vous découvrirez certainement des classes, des structures et des délégués génériques dans un espace de noms donné**. De plus, soyez attentif aux membres génériques d'une classe non générique !

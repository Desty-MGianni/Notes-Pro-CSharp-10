---
title: "Chapitre 17: Réflextion de Type, Liaison Tardive, Attributs et Types Dynamiques"
publish: true
---

# <big><big><big><b><font color =green>Réflexion de Type, Liaison Tardive,  Attributs et Types Dynamiques</font></b></big></big></big>

Comme indiqué au [[Chapitre 16#Le rôle des assemblages .NET|Chapitre 16]], ==les assemblies constituent l'unité de déploiement de base dans l'univers .NET==. À l'aide de l'explorateur d'objets intégré à Visual Studio (et de nombreux autres EDI), vous pouvez examiner les types au sein des assemblies référencés par un projet. De plus, **des outils externes tels que** *ildasm.exe* **vous permettent d'explorer le code CIL sous-jacent, les métadonnées de type et le manifeste d'assembly d'un binaire .NET donné**. Outre cette exploration des assemblies .NET au moment de la conception, **vous pouvez également obtenir ces mêmes informations par *programmation* à l'aide de l'espace de noms `System.Reflection`**. À cette fin, ***==la première étape de ce chapitre consiste à définir le rôle de la réflexion et la nécessité des métadonnées .NET.==***

Les sections suivantes abordent plusieurs sujets étroitement liés, qui reposent sur les services de réflexion. Par exemple, **==vous apprendrez comment un client .NET peut utiliser le chargement dynamique et la liaison tardive pour activer des types qu'il ne connaît pas à la compilation==**. **Vous apprendrez également à insérer des métadonnées personnalisées dans vos assemblies .NET à l'aide d'attributs système et d'attributs personnalisés**. Pour mieux comprendre ces sujets (qui peuvent paraître complexes), ***==le chapitre se termine par une démonstration de la création de plusieurs « objets enfichables » que vous pouvez intégrer à une application console extensible.==***

**==Ce chapitre vous présentera également le mot-clé `dynamic` de C# et vous expliquera comment les appels faiblement typés sont mappés à l'objet en mémoire approprié grâce au Dynamic Language Runtime (DLR)==**. Après avoir assimilé les services fournis par le DLR, **vous verrez des exemples d'utilisation des types dynamiques pour simplifier les appels de méthodes à liaison tardive** (via les services de réflexion) **et faciliter la communication avec les bibliothèques COM existantes.**

>[!note]
>Ne confondez pas le mot-clé `dynamic` de C# avec le concept d'assembly dynamique (voir [[Chapitre 18|Chapitre 18]]). Bien que vous puissiez utiliser le mot-clé `dynamic` lors de la création d'un assembly dynamique, il s'agit en définitive de deux concepts indépendants.

# L’importance des métadonnées de type

**La capacité à décrire pleinement les types** (classes, interfaces, structures, énumérations et délégués) **à l’aide de métadonnées est un élément clé de la plateforme .NET**. **==De nombreuses technologies .NET, telles que la sérialisation d’objets, exigent la capacité de découvrir le format des types à l’exécution==**. De plus, ==l’interopérabilité entre langages, de nombreux services de compilation et les fonctionnalités IntelliSense d’un EDI reposent tous sur une description concrète du *type*.==

**Rappelons que l'utilitaire *ildasm.exe* permet de consulter les métadonnées de type d'un assembly**. Dans le fichier *CarLibrary.il* généré (au [[Chapitre 16#Exploration des métadonnées de type|Chapitre 16]]), accédez à la section `METAINFO` pour afficher toutes les métadonnées de *CarLibrary*. Un court extrait est présenté ci-dessous :

```CIL
// ================================= M E T A I N F O ================================================

// ===========================================================
// ScopeName : CarLibrary.dll
// MVID      : {bef64307-e668-4558-b026-70c0b59636cc}
// 	CustomAttribute #1 (0c000004)
// 	-------------------------------------------------------
// 		CustomAttribute Type: 0a00000b
// 		CustomAttributeName: System.Runtime.CompilerServices.RefSafetyRulesAttribute :: instance void .ctor(int32)
// 		Length: 8
// 		Value : 01 00 0b 00 00 00 00 00                          >                <
// 		ctor args: (11)
// 
// ===========================================================
// Global functions
// -------------------------------------------------------
// 
// Global fields
// -------------------------------------------------------
// 
// Global MemberRefs
// -------------------------------------------------------
// 
// TypeDef #1 (02000002)
// -------------------------------------------------------
// 	TypDefName: CarLibrary.Car  (02000002)
// 	Flags     : [Public] [AutoLayout] [Class] [Abstract] [AnsiClass] [BeforeFieldInit]  (00100081)
// 	Extends   : 0100000D [TypeRef] System.Object
// 	Field #1 (04000001)
// 	-------------------------------------------------------
// 		Field Name: <PetName>k__BackingField (04000001)
// 		Flags     : [Private]  (00000001)
// 		CallCnvntn: [FIELD]
// 		Field type:  String
...
```

Comme vous pouvez le constater, **les métadonnées de type .NET sont verbeuses** (le format binaire est beaucoup plus compact). En fait, si je devais lister l'intégralité des métadonnées de l'assembly *CarLibrary.dll*, cela occuperait plusieurs pages. Étant donné le gaspillage considérable de papier que cela représenterait, contentons-nous d'examiner quelques éléments clés des métadonnées de l'assembly *CarLibrary.dll*.

>[!note]
>Ne vous attardez pas trop sur la syntaxe exacte de chaque élément de métadonnées .NET dans les sections suivantes. L'important est de retenir que les métadonnées .NET sont très descriptives et listent chaque type défini en interne (et référencé en externe) présent dans une base de code donnée.

## Affichage des métadonnées (partielles) de l'énumération `EngineStateEnum`

**Chaque type défini dans l'assembly courant est documenté à l'aide d'un jeton `TypeDef #n`** (`TypeDef` signifiant  *définition de type*). **==Si le type décrit utilise un type défini dans un assembly .NET distinct, le type référencé est documenté à l'aide d'un jeton `TypeRef #n`==** (`TypeRef` signifiant *référence de type*). **Un jeton `TypeRef` est un pointeur vers la définition complète des métadonnées du type référencé dans un assembly externe**. **==En résumé, les métadonnées .NET sont un ensemble de tables qui identifient clairement toutes les définitions de type (`TypeDef`s) et les types référencés (`TypeRef`s), et qui peuvent toutes être consultées à l'aide==*** d'*ildasm.exe*.

Concernant *CarLibrary.dll*, un `TypeDef` correspond à la description des métadonnées de `CarLibrary`. L'énumération `EngineStateEnum` (votre numéro peut différer; **==la numérotation des `TypeDef` dépend de l'ordre de traitement du fichier par le compilateur C#==**).

```CIL
// TypeDef #2 (02000003)
// -------------------------------------------------------
// 	TypDefName: CarLibrary.EngineStateEnum  (02000003)
// 	Flags     : [Public] [AutoLayout] [Class] [Sealed] [AnsiClass]  (00000101)
// 	Extends   : 01000011 [TypeRef] System.Enum
// 	Field #1 (04000005)
// 	-------------------------------------------------------
// 		Field Name: value__ (04000005)
// 		Flags     : [Public] [SpecialName] [RTSpecialName]  (00000606)
// 		CallCnvntn: [FIELD]
// 		Field type:  I4
// 
// 	Field #2 (04000006)
// 	-------------------------------------------------------
// 		Field Name: EngineAlive (04000006)
// 		Flags     : [Public] [Static] [Literal] [HasDefault]  (00008056)
// 		CallCnvntn: [FIELD]
// 		Field type:  ValueClass CarLibrary.EngineStateEnum
// 
// 	Field #3 (04000007)
// 	-------------------------------------------------------
// 		Field Name: EngineDead (04000007)
// 		Flags     : [Public] [Static] [Literal] [HasDefault]  (00008056)
// 		CallCnvntn: [FIELD]
// 		Field type:  ValueClass CarLibrary.EngineStateEnum
// 
...
```

>[!info] Selon les versions de .NET et de *ildasm*, le contenu de METAINFO peut varier.

**Ici, le jeton `TypDefName` sert à définir le nom du type donné, qui est en l'occurrence l'énumération personnalisée `CarLibrary.EngineStateEnum`**. **Le jeton de métadonnées `Extends` sert à documenter le type de base d'un type .NET donné** (ici, le type par référence, `System.Enum`). **Chaque champ d'une énumération est marqué à l'aide du jeton `Field #n`.**

>[!note] 
>Bien qu'elles ressemblent à des fautes de frappe, `TypDefName` ne contient pas le *e* et `DefltValue` ne contient pas le *au*  auxquels on pourrait s'attendre.

## Affichage (partiel) des métadonnées du type `Car`

Voici un extrait de la classe `Car` illustrant les points suivants :
- Définition des champs en termes de métadonnées .NET
- Documentation des méthodes via les métadonnées .NET
- Représentation d'une propriété automatique dans les métadonnées .NET

```CIL
// TypeDef #1 (02000002)
// -------------------------------------------------------
// 	TypDefName: CarLibrary.Car  (02000002)
// 	Flags     : [Public] [AutoLayout] [Class] [Abstract] [AnsiClass] [BeforeFieldInit]  (00100081)
// 	Extends   : 0100000D [TypeRef] System.Object
// 	Field #1 (04000001)
// 	-------------------------------------------------------
// 		Field Name: <PetName>k__BackingField (04000001)
// 		Flags     : [Private]  (00000001)
// 		CallCnvntn: [FIELD]
// 		Field type:  String
//
...
// 	Method #1 (06000001) 
// 	-------------------------------------------------------
// 		MethodName: get_PetName (06000001)
// 		Flags     : [Public] [HideBySig] [ReuseSlot] [SpecialName]  (00000886)
// 		RVA       : 0x00002050
// 		ImplFlags : [IL] [Managed]  (00000000)
// 		CallCnvntn: [DEFAULT]
// 		hasThis 
// 		ReturnType: String
// 		No arguments.
...
// 	Method #2 (06000002) 
// 	-------------------------------------------------------
// 		MethodName: set_PetName (06000002)
// 		Flags     : [Public] [HideBySig] [ReuseSlot] [SpecialName]  (00000886)
// 		RVA       : 0x00002058
// 		ImplFlags : [IL] [Managed]  (00000000)
// 		CallCnvntn: [DEFAULT]
// 		hasThis 
// 		ReturnType: Void
// 		1 Arguments
// 			Argument #1:  String
// 		1 Parameters
// 			(1) ParamToken : (08000001) Name : value flags: [none] (00000000)
...
// 	Property #1 (17000001)
// 	-------------------------------------------------------
// 		Prop.Name : PetName (17000001)
// 		Flags     : [none] (00000000)
// 		CallCnvntn: [PROPERTY]
// 		hasThis 
// 		ReturnType: String
// 		No arguments.
// 		Setter    : (06000002) set_PetName
// 		Getter    : (06000001) get_PetName
// 		0 Others
```

Tout d'abord, **notez que les métadonnées de la classe `Car` indiquent la classe de base du type (`System.Object`) et incluent divers indicateurs décrivant la construction de ce type** (par exemple, `[Public]`, `[Abstract]`, etc.). **Les méthodes** (telles que le constructeur de `Car`) **sont décrites par leurs paramètres, leur valeur de retour et leur nom.**

Notez comment **==une propriété automatique génère un champ de stockage privé==** (nommé `<PetName>k__BackingField`) **==et deux méthodes (dans le cas d'une propriété en lecture/écriture) nommées, dans cet exemple, `get_PetName()` et `set_PetName()`==**. Enfin, la propriété est associée aux méthodes internes `get`/`set` à l'aide des jetons `Getter`/`Setter` des métadonnées .NET.

## Examen d'une référence de type

**Rappelons que les métadonnées d'un assembly décrivent non seulement l'ensemble des types internes** (`Car`, `EngineStateEnum`, etc.), **mais aussi tous les types externes référencés par les types internes**. Par exemple, étant donné que *CarLibrary.dll* a défini deux énumérations, ***==vous trouverez un bloc `TypeRef` pour le type `System.Enum`==***, comme suit :

```CIL
// TypeRef #17 (01000011)
// -------------------------------------------------------
// Token:             0x01000011
// ResolutionScope:   0x23000001
// TypeRefName:       System.Enum
```

## Documentation de l'assembly de définition

Le fichier *CarLibrary.il* permet également de consulter les métadonnées .NET qui décrivent l'assembly lui-même à l'aide du jeton `Assembly`. Voici un extrait du manifeste de *CarLibrary.dll*. ==Notez que la version correspond à la version de l'assembly définie pour la bibliothèque.==

```CIL
// Assembly
// -------------------------------------------------------
// 	Token: 0x20000001
// 	Name : CarLibrary
// 	Public Key    :
// 	Hash Algorithm : 0x00008004
// 	Version: 1.0.0.0
// 	Major Version: 0x00000001
// 	Minor Version: 0x00000000
// 	Build Number: 0x00000000
// 	Revision Number: 0x00000000
// 	Locale: <null>
// 	Flags : [none] (00000000)
// 	CustomAttribute #1 (0c000005)
```

## Documentation des assemblies référencés

**Outre le jeton `Assembly` et l'ensemble des blocs `TypeDef` et `TypeRef`, les métadonnées .NET utilisent également les jetons `AssemblyRef #n` pour documenter chaque assembly externe**. Étant donné que ==chaque assembly .NET référence l'assembly de la bibliothèque de classes de base `System.Runtime`, vous trouverez un `AssemblyRef` pour `System.Runtime`, comme illustré dans le code suivant :

```CIL
// AssemblyRef #1 (23000001)
// -------------------------------------------------------
// 	Token: 0x23000001
// 	Public Key or Token: b0 3f 5f 7f 11 d5 0a 3a 
// 	Name: System.Runtime
// 	Version: 10.0.0.0
// 	Major Version: 0x0000000a
// 	Minor Version: 0x00000000
// 	Build Number: 0x00000000
// 	Revision Number: 0x00000000
// 	Locale: <null>
// 	HashValue Blob:
// 	Flags: [none] (00000000)
```

## Documentation des chaînes littérales

Le dernier point important concernant les métadonnées .NET est que chaque chaîne littérale de votre code est documentée sous le jeton `User Strings`.

>[!tip] Grande disparité entre le code IL du livre et celui généré moi-même. Ici, je garde celui du livre.

```CIL
// User Strings
// -------------------------------------------------------
// 70000001 : (23) L"CarLibrary Version 2.0!"
// 70000031 : (13) L"Quiet time..."
// 7000004d : (11) L"Jamming {0}"
// 70000065 : (32) L"Eek! Your engine block exploded!"
// 700000a7 : (34) L"Ramming speed! Faster is better..."
```

>[!note]
>Comme illustré dans la liste de métadonnées précédente, gardez toujours à l'esprit que toutes les chaînes de caractères sont clairement documentées dans les métadonnées de l'assemblage. Cela pourrait avoir d'énormes conséquences en matière de sécurité si vous utilisiez des chaînes de caractères littérales pour représenter des mots de passe, des numéros de carte de crédit ou d'autres informations sensibles.

Vous vous demandez peut-être (dans le meilleur des cas) : « ==Comment puis-je exploiter ces informations dans mes applications ?== » ou (dans le pire des cas) : « ==Pourquoi devrais-je me soucier des métadonnées ?== » **Pour répondre à ces deux points de vue, permettez-moi de vous présenter les services de réflexion .NET. Sachez que l’utilité des sujets abordés dans les pages suivantes pourrait vous paraître un peu obscure jusqu’à la fin de ce chapitre. Alors, patience !**

>[!note]
>Vous trouverez également plusieurs jetons `CustomAttribute` affichés par la section `METAINFO`, qui documentent les attributs appliqués au sein du code source. Vous découvrirez le rôle des attributs .NET plus loin dans ce chapitre.

# Comprendre la réflexion

Dans l'univers .NET, **la** *réflexion* **est le processus de découverte des types à l'exécution**. ==Grâce aux services de réflexion, vous pouvez obtenir par programmation les mêmes métadonnées que celles générées par *ildasm.exe,* à l'aide d'un modèle objet convivial==. Par exemple, **la réflexion vous permet d'obtenir la liste de tous les types contenus dans un assembly *.dll* ou *.exe* donné, y compris les méthodes, les champs, les propriétés et les événements définis par un type donné**. ***==Vous pouvez également découvrir dynamiquement l'ensemble des interfaces prises en charge par un type donné, les paramètres d'une méthode et d'autres détails connexes==*** (classes de base, informations d'espace de noms, données du manifeste, etc.).

**Comme tout espace de noms, `System.Reflection`** (défini dans *System.Runtime.dll*) **contient plusieurs types associés.** Le [[#Tableau 17-1 Un échantillon de membres de l'espace de noms `System.Reflection`|Tableau 17-1]] répertorie certains des éléments essentiels à connaître.

##### Tableau 17-1: Un échantillon de membres de l'espace de noms `System.Reflection`


| Type            | Description                                                                                                                                          |
| --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Assembly`      | Cette classe abstraite contient des membres qui vous permettent de charger, d'examiner et de manipuler un assembly.                                  |
| `AssemblyName`  | Cette classe  vous permet de découvrir de nombreux détails sur l'identité d'une assemblée (informations de version, informations culturelles, etc.). |
| `EventInfo`     | Cette classe abstraite contient des informations relatives à un événement donné.                                                                     |
| `FieldInfo`     | Cette classe abstraite contient des informations pour un champ donné.                                                                                |
| `MemberInfo`    | Il s'agit de la classe de base abstraite qui définit les comportements communs aux types `EventInfo`, `FieldInfo`, `MethodInfo` et `PropertyInfo`.   |
| `MethodInfo`    | Cette classe abstraite contient des informations pour une méthode donnée.                                                                            |
| `Module`        | Cette classe abstraite vous permet d'accéder à un module donné au sein d'un assemblage multi-fichier.                                                |
| `ParameterInfo` | Cette classe contient des informations pour un paramètre donné.                                                                                      |
| `PropertyInfo`  | Cette classe abstraite contient des informations relatives à une propriété donnée.                                                                   |

Pour comprendre comment exploiter l'espace de noms `System.Reflection` afin de lire par programmation les métadonnées .NET, **il est nécessaire de se familiariser au préalable avec la classe `System.Type`.**

## La classe `System.Type`

>[!warning] Bien que les bases de `System.Type` dans le livre restent valables, dans le développement moderne (.NET 8+), on cherche souvent à **éviter** la réflexion au profit des générateurs de source ou des `UnsafeAccessor` pour gagner en rapidité.

**La classe `System.Type` définit des membres permettant d’examiner les métadonnées d’un type, dont un grand nombre renvoient des types de l’espace de noms `System.Reflection`**. Par exemple, **==`Type.GetMethods()` renvoie un tableau d’objets `MethodInfo`, `Type.GetFields()` renvoie un tableau d’objets `FieldInfo`, etc==**. ***==L’ensemble complet des membres exposés par `System.Type` est très vaste==*** ; toutefois, le [[#Tableau 17-2 Sélection de membres de `System.Type`|Tableau 17-2]] offre un aperçu partiel des membres pris en charge par `System.Type` (voir la [documentation .NET](https://learn.microsoft.com/en-us/dotnet/api/system.type?view=net-10.0) pour plus de détails).

##### Tableau 17-2: Sélection de membres de `System.Type`

| Membre                                                                                                                                                                                                                             | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `IsAbstract`<br>`IsArray`<br>`IsClass`<br>`IsCOMObject`<br>`IsEnum`<br>`IsGenericTypeDefinition`<br>`IsGenericParameter`<br>`IsInterface`<br>`IsPrimitive`<br>`IsNestedPrivate`<br>`IsNestedPublic`<br>`IsSealed`<br>`IsValueType` | Ces propriétés (entre autres) vous permettent de découvrir un certain nombre de<br>caractéristiques fondamentales du type auquel vous faites référence (par exemple, s'il s'agit d'une entité abstraite, d'un<br>tableau, d'une classe imbriquée, etc.).                                                                                                                                                                                                                                                                                                             |
| `GetConstructors()`<br>`GetEvents()`<br>`GetFields()`<br>`GetInterfaces()`<br>`GetMembers()`<br>`GetMethods()`<br>`GetNestedTypes()`<br>`GetProperties()`                                                                          | Ces méthodes (entre autres) vous permettent d'obtenir un tableau représentant<br>les éléments (interface, méthode, propriété, etc.) qui vous intéressent. Chaque<br>méthode renvoie un tableau associé (par exemple, `GetFields()` renvoie un tableau `FieldInfo`, `GetMethods()` renvoie un tableau `MethodInfo`, etc.). **Notez que chacune de ces méthodes possède une forme singulière (par exemple, `GetMethod()`, `GetProperty()`, etc.) qui vous permet de récupérer un élément spécifique par son nom, plutôt qu'un tableau de tous les éléments associés.** |
| `FindMembers()`                                                                                                                                                                                                                    | Cette méthode renvoie un tableau `MemberInfo` en fonction des critères de recherche.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| `GetType()`                                                                                                                                                                                                                        | Cette méthode statique renvoie une instance de `Type` à partir d'un nom de type sous forme de chaîne.                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| `InvokeMember()`                                                                                                                                                                                                                   | Cette méthode permet la « liaison tardive » d'un élément donné. Vous en apprendrez davantage sur la liaison tardive plus loin dans ce chapitre.                                                                                                                                                                                                                                                                                                                                                                                                                      |

## Obtenir une référence de type avec `System.Object.GetType()`

Vous pouvez obtenir une instance de la classe `Type` de différentes manières. Cependant, *==vous ne pouvez pas créer directement un objet `Type` avec le mot-clé `new`, car `Type` est une classe abstraite==*. **Rappelons que `System.Object` définit une méthode nommée `GetType()`, qui renvoie une instance de la classe `Type` représentant les métadonnées de l'objet courant.**

```cs
// Obtenir les informations de type à partir d'une instance de SportsCar.
SportsCar sc = new SportsCar();
Type t = sc.GetType();
```

Bien entendu, ***==cette approche ne fonctionnera que si vous connaissez le type à analyser==*** (`SportsCar` dans ce cas) ***==au moment de la compilation et si vous disposez actuellement d'une instance de ce type en mémoire==***. **Compte tenu de cette restriction, il est logique que des outils comme** *ildasm.exe* **n'obtiennent pas d'informations de type en appelant directement `System.Object.GetType()` pour chaque type, car** *ildasm.exe* **n'a pas été compilé avec vos assemblies personnalisés.**

## Obtenir une référence de type avec `typeof()`

**La méthode suivante pour obtenir des informations de type consiste à utiliser l'opérateur `typeof` de C#**, comme ceci :

```cs
// Obtenir le type avec typeof.
Type t = typeof(SportsCar);
```

Contrairement à `System.Object.GetType()`, **l'opérateur `typeof` est utile car il n'est pas nécessaire de créer au préalable une instance d'objet pour extraire les informations de type**. *==Cependant, votre code doit toujours avoir une connaissance, à la compilation, du type que vous souhaitez examiner, car `typeof` attend le nom fortement typé du type.==*

## Obtenir une référence de type avec `System.Type.GetType()`

**Pour obtenir des informations de type de manière plus flexible, vous pouvez appeler la méthode statique `GetType()` de la classe `System.Type` et spécifier le nom complet** (sous forme de chaîne de caractères) **du type que vous souhaitez examiner**. **==Grâce à cette méthode, vous n'avez pas besoin de connaître le type à la compilation, car `Type.GetType()` prend une instance de la classe omniprésente `System.String`.==**

>[!note]
>Lorsque j'affirme qu'il n'est pas nécessaire de connaître le type à la compilation lors de l'appel à `Type.GetType()`, je fais référence au fait que cette méthode accepte n'importe quelle chaîne de caractères (et non une variable fortement typée). Bien sûr, il vous faudra toujours connaître le nom du type sous forme de chaîne de caractères !

**La méthode `Type.GetType()` a été surchargée pour vous permettre de spécifier deux paramètres booléens, l’un détermine si une exception doit être levée si le type est introuvable, et l’autre qui définit la sensibilité à la casse de la chaîne.** À titre d’exemple, considérez ce qui suit :

```cs
// Obtenir les informations de type à l'aide de la méthode statique Type.GetType()
// (ne pas lever d'exception si SportsCar est introuvable et ignorer la casse).
Type t = Type.GetType("CarLibrary.SportsCar", false, true);
```

Dans l'exemple précédent, **notez que la chaîne transmise à `GetType()` ne mentionne pas l'assembly contenant le type.** Dans ce cas, ==on suppose que le type est défini dans l'assembly en cours d'exécution==. ***==Cependant, lorsque vous souhaitez obtenir les métadonnées d'un type dans un assembly externe, le paramètre de type chaîne est formaté comme suit : nom complet du type, suivi d'une virgule, puis du nom convivial==*** (le nom de l'assembly sans information de version) ***==de l'assembly contenant le type, comme ceci==*** :

```cs
// Obtenir les informations de type d'un type au sein d'un assembly externe.
Type t = Type.GetType("CarLibrary.SportsCar, CarLibrary");
```

**Sachez également que la chaîne passée à `Type.GetType()` peut contenir un signe plus (`+`) pour indiquer un type imbriqué**. Supposons que vous souhaitiez obtenir les informations de type d'une énumération (`SpyOptions`) imbriquée dans une classe nommée `JamesBondCar`. Pour ce faire, vous écririez ce qui suit :

```cs
// Obtenir les informations de type pour une énumération imbriquée
// dans l'assembly courant.
Type t = Type.GetType("CarLibrary.JamesBondCar+SpyOptions");
```

# Création d'un visualiseur de métadonnées personnalisé

Pour illustrer le processus de réflexion de base (et l'utilité de `System.Type`), créons un projet d'application console nommé *MyTypeViewer*. **Ce programme affichera les détails des méthodes, propriétés, champs et interfaces prises en charge** (ainsi que d'autres points d'intérêt) **pour tout type de *System.Runtime.dll*** (==rappelons que toutes les applications .NET ont un accès automatique à cette bibliothèque de classes du framework==) **ou d'un type au sein de *MyTypeViewer***. Une fois l'application créée, veillez à importer l'espace de noms `System.Reflection`.

```cs
// Besion d'importer cette espace de noms 
// pour effectuer n'importe quel réflextion.
using System.Reflection;
```

## Réflexion sur les méthodes

Plusieurs méthodes statiques seront ajoutées au fichier *Program.cs*. Chacune prend un paramètre de type `System.Type` et ne renvoie aucune valeur. La première est `ListMethods()`, qui (comme vous pouvez le deviner) affiche le nom de chaque méthode définie par le type fourni. Notez que `Type.GetMethods()` renvoie un tableau d'objets `System.Reflection.MethodInfo`, que l'on peut parcourir avec une boucle `foreach` standard, comme suit :

```cs
// Affiche le nom des méthodes du type
static void ListMethods(Type t)
{
    Console.WriteLine("***** Methods *****");
    MethodInfo[] mi = t.GetMethods();

    foreach (MethodInfo m in mi)
    {
        Console.WriteLine($"-> {m.Name}");
    }
    Console.WriteLine();
}
```

**Ici, vous affichez simplement le nom de la méthode à l'aide de la propriété `MethodInfo.Name`**. Comme vous pouvez le deviner, **`MethodInfo` possède de nombreux membres supplémentaires qui vous permettent de déterminer si la méthode est statique, virtuelle, générique ou abstraite**. De plus, **==le type `MethodInfo` vous permet d'obtenir la valeur de retour et l'ensemble des paramètres de la méthode==**. Vous améliorerez l'implémentation de `ListMethods()` dans un instant.

***==Si vous le souhaitez, vous pouvez également créer une requête LINQ appropriée pour énumérer les noms de chaque méthode==***. Rappelez-vous du [[Chapitre 13#Réflexion sur un ensemble de résultats LINQ|Chapitre 13]] : ***LINQ to Objects* vous permet de créer des requêtes fortement typées qui peuvent être appliquées à des collections d'objets en mémoire**. En règle générale, ==chaque fois que vous trouvez des blocs de boucle ou de logique de programmation décisionnelle, vous pouvez utiliser une requête LINQ associée==. Par exemple, vous pouvez réécrire la méthode précédente avec LINQ comme ceci :

```cs
static void ListMethods(Type t)
{
    Console.WriteLine("***** Methods *****");
    var methodNames = from n in t.GetMethods() orderby n.Name select n.Name;

    // En utilisant les méthode d'extensions LINQ:
    //var methodNames = t.GetMethods().OrderBy(m => m.Name).Select(m => m.Name);
    foreach (var name in methodNames)
    {
        Console.WriteLine($"-> {name}");
    }
    Console.WriteLine();
}
```

## Réflexions sur les champs et les propriétés

**L'implémentation de `ListFields()` est similaire**. ==La seule différence notable réside dans l'appel à `Type.GetFields()` et le tableau `FieldInfo` résultant==. Là encore, pour simplifier, vous n'affichez que le nom de chaque champ à l'aide d'une requête LINQ.

```cs
// Affiche le nom des champs du type
static void ListFields(Type t)
{
    Console.WriteLine("***** Fields *****");
    //var fieldNames = from f in t.GetFields() orderby f.Name select f.Name;
    var fieldNames = t.GetFields().OrderBy(f => f.Name).Select(x => x.Name);
    foreach (string name in fieldNames)
    {
        Console.WriteLine($"-> {name}");
    }
    Console.WriteLine();
}
```

La logique d’affichage des propriétés d’un type est similaire.

```cs
// Affiche le nom des propriétés du type
static void ListProps(Type t)
{
    Console.WriteLine("***** Properties *****");
    var propNames = from p in t.GetProperties() orderby p.Name select p.Name;
    //var propNames = t.GetProperties().OrderBy(p => p.Name).Select(p => p.Name);
    foreach (var name in propNames)
    {
        Console.WriteLine($"-> {name}");
    }
    Console.WriteLine();
}
```

## Réflexion sur les interfaces implémentés

Vous allez maintenant créer une méthode nommée `ListInterfaces()` qui affichera les noms des interfaces prises en charge par le type entrant. **(À noter : l’appel à `GetInterfaces()` renvoie un tableau de `System.Types` !** Cela est logique puisque ==les interfaces sont, en effet, des types.==

```cs
// Affiche les interfaces implémentées
static void ListInterfaces(Type t)
{
    Console.WriteLine("***** Interfaces *****");
    //var ifaces = from i in t.GetInterfaces() orderby i.Name select i;
    var ifaces = t.GetInterfaces().OrderBy(i => i.Name);

    foreach (Type i in ifaces)
    {
        Console.WriteLine($"-> {i.Name}");
    }
	Console.WriteLine();
}
```

>[!note]
>Sachez que la plupart des méthodes « get » de `System.Type` (`GetMethods()`, `GetInterfaces()`, etc.) ont été surchargées pour vous permettre de spécifier des valeurs issues de l'énumération `BindingFlags`. Cela offre un contrôle plus précis sur les éléments à rechercher (par exemple, uniquement les membres statiques, uniquement les membres publics, inclure les membres privés, etc.). Consultez la [documentation](https://learn.microsoft.com/en-us/dotnet/api/system.reflection.bindingflags?view=net-10.0) pour plus de détails.

## Affichage de diverses informations

Le dernier mais pas des moindres, **vous disposez d'une dernière méthode d'assistance qui affichera simplement diverses statistiques** (==indiquant si le type est générique, quelle est la classe de base, si le type est scellé, etc==.) **concernant le type entrant.**

```cs
// Juste pour être sûr.
static void ListVariousStats(Type t)
{
    Console.WriteLine("***** Various Statistics *****");
    Console.WriteLine($"Base class is {t.BaseType}");
    Console.WriteLine($"Is type abstract? {t.IsAbstract}");
    Console.WriteLine($"Is type sealed? {t.IsSealed}");
    Console.WriteLine($"Is type generic? {t.IsGenericTypeDefinition}");
    Console.WriteLine($"is type a class type? {t.IsClass}");
    Console.WriteLine();
}
```

## Ajout des instructions de niveau supérieur

**Les instructions de niveau supérieur du fichier** *Program.cs* **invitent l'utilisateur à saisir le nom complet d'un type**. Une fois ces données obtenues, vous les transmettez à la méthode `Type.GetType()` et envoyez la valeur extraite de `System.Type` à chacune de vos méthodes auxiliaires. Ce processus se répète jusqu'à ce que l'utilisateur appuie sur Q pour quitter l'application.

```cs
Console.Title = "Welcome to MyTypeViewer";
Console.WriteLine("***** Welcome to MyTypeViewer *****\n");
string typeName = "";

do
{
    Console.WriteLine("Enter a type name to evaluate");
    Console.Write("Or enter Q to quit: ");

    // Récupère le nom du type
    typeName = Console.ReadLine();

    // Est-ce que l'utilisateur veut partir ?
    if (typeName.Equals("Q", StringComparison.OrdinalIgnoreCase))
    {
        break;
    }

    // Essaye d'afficher le type
    try
    {
        Type t = Type.GetType(typeName);

        // Les .NET modernes ont changé ou se trouve la classe Console
        // Pour éviter à l'utilisateur d'entrer deux fois la même chose:
        if (
            t == null
            && typeName.Equals(
                "System.Console",
                StringComparison.OrdinalIgnoreCase
            )
        )
        {
            t = typeof(System.Console);
        }
        Console.WriteLine("");
        ListVariousStats(t);
        ListFields(t);
        ListProps(t);
        ListMethods(t);
        ListInterfaces(t);
    }
    catch
    {
        Console.WriteLine("Sorry, cant find the type");
    }
} while (true);

```

À ce stade, *MyTypeViewer.exe* est prêt à être testé. Par exemple, exécutez votre application et saisissez les noms complets suivants (***==notez que `Type.GetType()` exige des noms de chaînes sensibles à la casse==***) :

- `System.Int32`
- `System.Collections.ArrayList`
- `System.Threading.Thread`
- `System.Void`
- `System.Math`

Voici par exemple un extrait du résultat obtenu en spécifiant `System.Math` :

```
***** Various Statistics *****
Base class is System.Object
Is type abstract? True
Is type sealed? True
Is type generic? False
is type a class type? True

***** Fields *****
-> E
-> PI
-> Tau

***** Properties *****

***** Methods *****
-> Abs
-> Abs
-> Abs
-> Abs
-> Abs
-> Abs
-> Abs
-> Abs
-> Acos
-> Acosh
-> Asin
-> Asinh
-> Atan
-> Atan2

...

-> Sign
-> Sin
-> SinCos
-> Sinh
-> Sqrt
-> Tan
-> Tanh
-> ToString
-> Truncate
-> Truncate

***** Interfaces *****

```

==Remarquez la présence de plusieurs occurrences de `Abs`==. Cela s'explique par la présence d'***==au moins une surcharge pour la méthode `Abs()`==***. **Le code sera prochainement complété pour afficher les paramètres et les types de retour.**

>[!warning] Attention lors de la publication de ce programme
>
>Si vous essayer de publier ce programme (comme lors du [[Chapitre 16#Publication d'applications console (MaJ .NET 5/6)|Chapitre 16]]), le code ne fonctionnera pas !
>
>### Le conflit entre Réflexion et Optimisation
>
>Lorsque vous publiez avec des options de réduction de taille (souvent liées au paramètre `PublishTrimmed`), le compilateur analyse votre code pour supprimer tout ce qui n'est pas utilisé.
>
>- **Le problème :** Le compilateur regarde votre code source et voit que vous n'appelez jamais explicitement `System.String.ToUpper()` par exemple. Il décide donc de **supprimer** les métadonnées de `System.String` pour gagner de la place.
>- **L'effet :** Quand votre programme tourne et que vous tapez "System.String", la réflexion essaie d'accéder à des membres qui ont été physiquement supprimés de l'exécutable final.
>
>### Pourquoi `ReadyToRun` déclenche des warnings ?
>
>Même si vous n'êtes pas en `AOT` complet, `ReadyToRun` avec des options d'optimisation tente de prédire les types nécessaires. La réflexion est par nature "imprévisible" pour un compilateur (il ne sait pas ce que l'utilisateur va taper au clavier).
>
>### Comment régler ça ?
>
>Pour un outil de diagnostic comme le vôtre, l'optimisation agressive est votre ennemie. Dans votre fichier `.csproj`, assurez-vous de :
>
>1. Désactiver le trimming : `<PublishTrimmed>false</PublishTrimmed>`
>2. Ou "préserver" des bibliothèques entières (mais cela augmentera la taille du fichier)

## Réflexions sur les types statiques

**Si vous saisissez `System.Console` dans la méthode précédente, une exception sera levée dans la première méthode auxiliaire car la valeur de t sera `null`**. *==Les types statiques ne peuvent pas être chargés à l'aide de la méthode `Type.GetType(typeName)`==*. **Vous devez utiliser un autre mécanisme : la fonction `typeof` de `System.Type`**. Mettez à jour le programme pour gérer le cas particulier de `System.Console` comme suit :

```cs
if (
	t == null
	&& typeName.Equals(
		"System.Console",
		StringComparison.OrdinalIgnoreCase
	)
)
{
	t = typeof(System.Console);
}
```

>[!info] Le livre prend un raccourcis pédagogique ! (avec Gemini)
>
>### Le vrai problème : Statique ou pas ?
>
>En réalité, `Type.GetType()` **peut** tout à fait charger des classes statiques. Le problème de `System.Console` n'est pas qu'elle est `static`, mais qu'elle réside dans un **assembly différent** de celui du cœur du runtime (`System.Private.CoreLib`).
>
>- Si vous créez votre propre classe statique `MyStaticClass` dans votre projet, `Type.GetType("MyStaticClass")` fonctionnera parfaitement.
>- Le livre utilise cet argument pour vous amener vers la solution `typeof()`, qui est la plus simple à ce stade de votre lecture.
>
>### Pourquoi `typeof` est la solution "magique" ici ?
>
>Comme nous l'avons vu pour votre question sur la publication (Publish) :
>
>- `Type.GetType(string)` est **dynamique** : Le compilateur ne sait pas ce que vous allez taper. Il ne peut pas garantir que le type sera là.
>- `typeof(System.Console)` est **statique** (résolu à la compilation) : Cela force le programme à charger l'assembly `System.Console.dll` dès le départ. Une fois l'assembly chargé en mémoire, le programme "connaît" enfin ce type.

## Réflexion sur les types génériques

**Lorsque vous appelez `Type.GetType()` pour obtenir les métadonnées des types génériques, vous devez utiliser une syntaxe spéciale**. ***==Cette syntaxe implique un caractère d'accent grave suivi d'une valeur numérique représentant le nombre de paramètres de type pris en charge par le type==***. Par exemple, pour afficher les métadonnées de `System.Collections.Generic.List<T>`, vous devez transmettre la chaîne suivante à votre application :

```
System.Collections.Generic.List`1
```

**Ici, la valeur numérique $1$ est utilisée, car `List<T>` ne possède qu'un seul paramètre de type**. **==En revanche, pour analyser `Dictionary<TKey, TValue>`, indiquez la valeur $2$==** :
`
```
System.Collections.Generic.Dictionary`2
```

>[!info] C'est la même orthographe que celle trouvé dans le code IL (visible avec *ildasm*).

## Réflexion sur les paramètres et les valeurs de retour des méthodes

Jusqu'ici tout va bien ! Nous allons maintenant apporter une légère amélioration à l'application. Plus précisément, **vous allez mettre à jour la fonction d'assistance `ListMethods()` afin qu'elle affiche non seulement le nom d'une méthode donnée, mais aussi son type de retour et les types de ses paramètres**. ***==Le type `MethodInfo` fournit la propriété `ReturnType` et la méthode `GetParameters()` pour cela==***. Dans le code modifié ci-dessous, ==notez que vous construisez une chaîne de caractères contenant le type et le nom de chaque paramètre à l'aide d'une boucle `foreach` imbriquée== (sans utiliser LINQ) :

```cs
// Affiche le nom des méthodes du type
static void ListMethods(Type t)
{
    Console.WriteLine("***** Methods *****");
    // Syntaxe moderne et mieux optimisé (C# 12)
    MethodInfo[] mi = [.. t.GetMethods().OrderBy(m => m.Name)];
    foreach (MethodInfo m in mi)
    {
        // Récupère le type de retour
        string retVal = m.ReturnType.FullName;
        string paramInfo = "( ";
        // Récupère les paramètres
        foreach (ParameterInfo pi in m.GetParameters())
        {
            paramInfo += string.Format($"{pi.ParameterType} {pi.Name}");
        }
        paramInfo += " )";

        // Maintenant affiche la signature basique de la méthode.
        Console.WriteLine($"-> {retVal} {m.Name} {paramInfo}");
    }
    Console.WriteLine();
}
```

**Si vous exécutez maintenant cette application mise à jour, vous constaterez que les méthodes d'un type donné sont beaucoup plus détaillées et que le mystère des méthodes répétées est résolu**. ==Si vous saisissez `System.Math` dans le programme, les deux méthodes `Abs()`== (et toutes les autres méthodes) ==afficheront le type de retour et le(s) paramètre(s)==.

```
***** Methods *****
...
->System.Double Abs ( System.Double value )
->System.Single Abs ( System.Single value )
...
```

**L'implémentation actuelle de `ListMethods()` est utile, car elle permet d'examiner directement chaque paramètre et type de retour de méthode à l'aide du modèle objet `System.Reflection`**. ***==À titre de raccourci, notez que tous les types `XXXInfo` (`MethodInfo`, `PropertyInfo`, `EventInfo`, etc.) ont redéfini la méthode `ToString()` pour afficher la signature de l'élément demandé==***. Ainsi, ==vous pouvez également implémenter `ListMethods()` comme suit== (toujours en utilisant LINQ, en sélectionnant simplement tous les objets `MethodInfo`, plutôt que seulement les valeurs de `Name`) :

```cs
// Affiche le nom des méthodes du type
static void ListMethods(Type t)
{
    Console.WriteLine("***** Methods *****");
    var methods = t.GetMethods().OrderBy(m => m.Name);
    foreach (var m in methods)
    {
        Console.WriteLine($"-> {m}");
    }
    Console.WriteLine();
}
```

Intéressant, n'est-ce pas ? De toute évidence, **l'espace de noms `System.Reflection` et la classe `System.Type` vous permettent d'explorer de nombreux aspects d'un type, au-delà de ce que** *MyTypeViewer* **affiche actuellement**. Comme vous pouvez l'espérer, **==vous pouvez obtenir les événements d'un type, la liste des paramètres génériques d'un membre donné et glaner des dizaines d'autres détails.==**

**Néanmoins, vous avez créé un explorateur d'objets** (relativement performant). *==La principale limitation de cet exemple est l'impossibilité d'explorer au-delà de l'assembly actuel==* (*MyTypeViewer*) *==et des assemblies des bibliothèques de classes de base toujours référencées==*. Cela soulève la question : « ==Comment créer des applications capables de charger (et d'explorer) des assemblies non référencés à la compilation ?== » Bonne question !

# Chargement dynamique d'assemblies

==Il peut arriver que vous ayez besoin de charger des assemblies à la volée par programmation, même s'ils ne sont pas référencés dans le manifeste==. **Formellement, le chargement d'assemblies externes à la demande est appelé** *chargement dynamique*.

***==`System.Reflection` définit une classe nommée `Assembly`==***. **Grâce à cette classe, vous pouvez charger dynamiquement un assembly et découvrir ses propriétés**. **==La classe `Assembly` fournit des méthodes permettant de charger des assemblies depuis le disque par programmation.==**

Pour illustrer le chargement dynamique, créez un projet d'application console nommé *ExternalAssemblyReflector*. ==Votre tâche consiste à écrire un code qui invite à saisir le nom d'un assembly== (sans extension) ==à charger dynamiquement==. Vous transmettrez la référence de l'assembly à une méthode auxiliaire nommée `DisplayTypes()`, qui affichera simplement les noms de chaque classe, interface, structure, énumération et délégué contenant l'assembly. Le code est d'une simplicité remarquable.

```cs
using System.Reflection;

Console.Title = "External Assembly Viewer";
Console.WriteLine("***** External Assembly Viewer *****\n");

string asmName = "";
Assembly asm = null;

do
{
    Console.WriteLine("Enter an assembly to evaluate");
    Console.Write("or enter Q to quit: ");

    // Récupère le nom de l'assembly.
    asmName = Console.ReadLine();

    // Est-ce que l'utilisateur veut quitter le programme ?
    if (asmName.Equals("Q", StringComparison.OrdinalIgnoreCase))
    {
        break;
    }

    // Essaye de charger l'assembly
    try
    {
        asm = Assembly.LoadFrom(asmName);
        DisplayTypesInAsm(asm);
    }
    catch
    {
        Console.WriteLine("Sorry, can't find assembly");
    }
} while (true);

void DisplayTypesInAsm(Assembly asm)
{
    Console.WriteLine("\n***** Types in Assembly *****");
    Console.WriteLine($"-> {asm.FullName}");
    Type[] types = asm.GetTypes();
    foreach (Type t in types)
    {
        Console.WriteLine($"Type: {t}");
    }
    Console.WriteLine();
}
```

***==Si vous souhaitez analyser le fichier==*** *CarLibrary.dll*, ***==vous devrez copier le fichier binaire==*** *CarLibrary.dll* (du chapitre précédent) ***==dans le répertoire du projet (si vous utilisez Visual Studio Code) ou dans le répertoire `\bin\Debug\net6.0` (si vous utilisez Visual Studio) de l'application==*** *ExternalAssemblyReflector* ***==pour exécuter ce programme.==*** Saisissez `CarLibrary` (l'extension est facultative) lorsque vous y êtes invité ; le résultat devrait ressembler à ceci :

>[!warning] **Il faut utiliser le DLL C#** (CIL) **et non le résultat d'un `dotnet publish` car la réflection à besoin des métadonnées contenue dans le CIL.**

>[!tip] Rappel, ici, on cherche à charger du code sans que le programme ne connaisse l'assembly ! (Il ne faut pas faire `dotnet add package` par exemple)

>[!warning] la méthode `Assembly.LoadFrom()` dépend du *working directory*. C'est ce que l'auteur induit quand il liste les chemin d'accès ou il faut copier *CarLibrary.dll*.

```
***** External Assembly Viewer *****

Enter an assembly to evaluate
or enter Q to quit: CarLibrary.dll

***** Types in Assembly *****
->CarLibrary, Version=1.0.4.0, Culture=neutral, PublicKeyToken=null
Type: CarLibrary.Car
Type: CarLibrary.EngineStateEnum
Type: CarLibrary.MiniVan
Type: CarLibrary.MusicMediaEnum
Type: CarLibrary.MyInternalClass
Type: CarLibrary.SportsCar
```

**La méthode `LoadFrom` peut également accepter un chemin absolu vers l'assembly à afficher** (par exemple, `C:\MyApp\MyAsm.dll`). **==Grâce à cette méthode, vous pouvez spécifier le chemin complet vers votre projet d'application console==**. Ainsi, si *CarLibrary.dll* se trouve sous `C:\MyCode`, vous pouvez saisir `C:\MyCode\CarLibrary.dll` (l'extension est nécessaire).

# Réflexions sur les assemblies du framework

**La méthode `Assembly.Load()` possède plusieurs surcharges**. ==Une variante permet de spécifier une valeur de culture== (pour les assemblies localisés), ==ainsi qu'un numéro de version et une valeur de jeton de clé publique== (pour les assemblies du framework). **L'ensemble des éléments identifiant un assembly est appelé *nom d'affichage***. **==Le format d'un nom d'affichage est une chaîne de paires nom-valeur séparées par des virgules, commençant par le nom convivial de l'assembly, suivi de qualificateurs optionnels==** (pouvant apparaître dans n'importe quel ordre). ***==Voici le modèle à suivre==*** (==les éléments optionnels apparaissent entre parenthèses==) :

```
Nom (, Version = majeur.mineur.build.révision) (, Culture = jeton culture) (,PublicKeyToken = jeton clé publique)
```

Lors de la création d'un nom d'affichage, **==la convention `PublicKeyToken=null` indique que la liaison et la correspondance avec un assembly non fortement nommé sont requises==**. ==De plus, `Culture=""` indique une correspondance avec la culture par défaut de la machine cible==. Voici un exemple :


```cs
// Charger la version 1.0.0.1 de CarLibrary en utilisant la culture par défaut.
Assembly a =
Assembly.Load("CarLibrary, Version=1.0.0.1, PublicKeyToken=null, Culture=\"\"");
// En C#, les guillemets doivent être échappés avec des barres obliques inverses.
```

**Notez également que l'espace de noms `System.Reflection` fournit le type `AssemblyName`, qui permet de représenter les informations de chaîne précédentes dans une variable objet pratique**. **Généralement, cette classe est utilisée conjointement avec `System.Version`, qui encapsule le numéro de version d'un assembly**. Une fois le nom d'affichage défini, il peut être transmis à la méthode surchargée `Assembly.Load()`, comme ceci :

```cs
// Utilisez AssemblyName pour définir le nom d'affichage.
AssemblyName asmName;
asmName = new AssemblyName();
asmName.Name = "CarLibrary";
Version v = new Version("1.0.0.1");
asmName.Version = v;
Assembly a = Assembly.Load(asmName);
```

==Pour charger un assembly .NET Framework (et non .NET), le paramètre `Assembly.Load()` doit spécifier une valeur `PublicKeyToken`==. **Avec .NET, ce n'est plus nécessaire, du fait de la diminution de l'utilisation des noms forts.** Par exemple, supposons que vous ayez un nouveau projet d'application console nommé *FrameworkAssemblyViewer* qui contient une référence au package `Microsoft.EntityFrameworkCore`. Pour rappel, tout cela peut être réalisé avec l'interface de ligne de commande (CLI) .NET.

```bash
dotnet new console -n FrameworkAssemblyViewer
dotnet sln Chapter17_AllProjects.slnx add 
dotnet add FrameworkAssemblyViewer package Microsoft.EntityFrameworkCore
```

Rappelons que lorsqu'une autre assembly est référencée, une copie de cette assembly est copiée dans le répertoire de sortie du projet référençant. Compilez le projet à l'aide de l'interface de ligne de commande.

```cs
using System.Reflection;

Console.Title = "The Framework Assembly Reflector App";
Console.WriteLine("***** The Framework Assembly Reflector App *****\n");

// Charge Microsoft.EntityFrameworkCore.dll
var displayName =
    "Microsoft.EntityFrameworkCore, Version=10.0.7, Culture=neutral, PublicKeyToken=adb9793829ddae60";
Assembly asm = Assembly.Load(displayName);
DisplayInfo(asm);
Console.WriteLine("Done!");
Console.ReadLine();

static void DisplayInfo(Assembly a)
{
    AssemblyName asmNameInfo = a.GetName();
    Console.WriteLine("***** Info about Assembly *****");
    Console.WriteLine($"Asm Name: {asmNameInfo.Name}");
    Console.WriteLine($"Asm Version: {asmNameInfo.Version}");
    Console.WriteLine($"Asm Culture: {asmNameInfo.CultureInfo.DisplayName}");
    Console.WriteLine("\nHere are the public enums:");
    // Use a LINQ query to find the public enums.
    var publicEnums = a.GetTypes().Where(p => p.IsEnum && p.IsPublic);
    foreach (var pe in publicEnums)
    {
        Console.WriteLine(pe);
    }
}
```

À ce stade, vous devriez comprendre **==comment utiliser certains des membres principaux de l'espace de noms `System.Reflection` pour découvrir les métadonnées à l'exécution==**. Bien sûr, ==vous n'aurez probablement pas souvent besoin (voire jamais) de créer des navigateurs d'objets personnalisés==. Cependant, **les services de réflexion constituent la base de nombreuses activités de programmation courantes, notamment la liaison tardive.**

# Comprendre la liaison tardive

En termes simples, la *liaison tardive* est **une technique permettant de créer une instance d'un type donné et d'appeler ses membres à l'exécution sans avoir à connaître son existence lors de la compilation**. **==Lorsque vous développez une application qui effectue une liaison tardive avec un type dans un assembly externe, vous n'avez aucune raison de définir une référence à cet assembly; par conséquent, le manifeste de l'appelant ne contient aucune référence directe à l'assembly.==**

À première vue, l'intérêt de la liaison tardive n'est pas évident. ==Il est vrai que si vous pouvez effectuer une liaison précoce à un objet== (par exemple, en ajoutant une référence à l'assembly et en allouant le type à l'aide du mot-clé `new` de C#)==, vous devriez privilégier cette méthode==. En effet, **la liaison précoce permet de détecter les erreurs à la compilation, et non à l'exécution**. **==Néanmoins, la liaison tardive joue un rôle crucial dans toute application extensible==**. Vous aurez l'occasion de développer un tel programme « extensible » plus loin dans ce chapitre, dans la section "[[#Création d'une application extensible]]". En attendant, examinons le rôle de la classe `Activator`.

## La classe `System.Activator`

**La classe `System.Activator` est essentielle au processus de liaison tardive .NET**. ***==Pour l'exemple suivant, vous vous intéresserez uniquement à la méthode `Activator.CreateInstance()`, qui permet de créer une instance d'un type via la liaison tardive==***. ==Cette méthode a été surchargée à de nombreuses reprises afin d'offrir une grande flexibilité==. **La variante la plus simple de la méthode `CreateInstance()` prend un objet `Type` valide décrivant l'entité que vous souhaitez allouer en mémoire à la volée.**

Créez un nouveau projet d'application console nommé *LateBindingApp* et modifiez le fichier *Program.cs* comme suit :

```cs
using System.Reflection;

// Ce programme chargera une bibliothèque externe,
// et créera un objet en utilisant la liaison tardive.
Console.Title = "Fun with Late Binding";
Console.WriteLine("***** Fun with Late Binding *****\n");

// Essaye de charger une copie locale de CarLibrary.
Assembly a = null;
try
{
    a = Assembly.LoadFrom("CarLibrary.dll");
}
catch (FileNotFoundException ex)
{
    Console.WriteLine(ex.Message);
}

if (a != null)
{
    CreateUsingLateBinding(a);
}
Console.ReadLine();

static void CreateUsingLateBinding(Assembly asm)
{
    try
    {
        // Récupère les métadonnées pour le type MiniVan
        Type miniVan = asm.GetType("CarLibrary.MiniVan");

        // Crée une instance de MiniVan à la volée
        object obj = Activator.CreateInstance(miniVan);
        Console.WriteLine($"Created a {obj} using late binding!");
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.Message);
    }
}

```

Avant d'exécuter cette application, ***==vous devrez placer manuellement une copie de==*** *CarLibrary.dll* ***==dans le dossier du projet (ou le dossier `bin\Debug\net6.0` si vous utilisez Visual Studio) de cette nouvelle application==***.

>[!note]
>N’ajoutez pas de référence à *CarLibrary.dll* pour cet exemple ! Tout l’intérêt de la liaison tardive est que vous essayez de créer un objet qui n’est pas connu au moment de la compilation.

**Notez que la méthode `Activator.CreateInstance()` renvoie un `System.Object` plutôt qu'un `MiniVan` fortement typé**. Par conséquent, *==si vous appliquez l'opérateur point à la variable `obj`, vous ne verrez aucun membre de la classe `MiniVan`==*. À première vue, vous pourriez penser qu'il est possible de résoudre ce problème avec une conversion explicite, comme ceci :

```cs
// Cast pour accéder aux membres de MiniVan ?
// Non ! Erreur de compilation !
object obj = (MiniVan)Activator.CreateInstance(minivan);
```

Cependant, **comme votre programme n'a pas ajouté de référence à** *CarLibrary.dll*, **vous ne pouvez pas utiliser le mot-clé C# `using` pour importer l'espace de noms `CarLibrary` et, par conséquent, vous ne pouvez pas utiliser un type `MiniVan` lors de l'opération de conversion**. N'oubliez pas que ==l'intérêt principal de la liaison tardive est de créer des instances d'objets pour lesquels aucune information n'est disponible à la compilation. Dès lors, comment pouvez-vous appeler les méthodes sous-jacentes de l'objet MiniVan stocké dans la référence System.Object ?== **==La réponse, bien sûr, est d'utiliser la réflexion.==**

## Appel de méthodes sans paramètres

==Supposons que vous souhaitiez appeler la méthode `TurboBoost()` de la `MiniVan`==. Comme vous vous en souvenez peut-être, cette méthode met le moteur à l'arrêt et affiche un message d'information. **La première étape consiste à obtenir un objet `MethodInfo` pour la méthode `TurboBoost()` à l'aide de `Type.GetMethod()`**. ==À partir de cet objet `MethodInfo`, vous pouvez ensuite appeler `MiniVan.TurboBoost` à l'aide de `Invoke()`==. **==`MethodInfo.Invoke()` exige que vous lui fournissiez tous les paramètres à passer à la méthode représentée par `MethodInfo`==**. **Ces paramètres sont représentés par un tableau de types `System.Object` (car les paramètres d'une méthode donnée peuvent être de nature diverse).**

**Étant donné que `TurboBoost()` ne requiert aucun paramètre, vous pouvez simplement passer `null`** (ce qui signifie que cette méthode n'a pas de paramètres). Mettez à jour votre méthode `CreateUsingLateBinding()` comme suit :

```cs
static void CreateUsingLateBinding(Assembly asm)
{
    try
    {
        // Récupère les métadonnées pour le type MiniVan
        Type miniVan = asm.GetType("CarLibrary.MiniVan");

        // Crée une instance de MiniVan à la volée
        object obj = Activator.CreateInstance(miniVan);
        Console.WriteLine($"Created a {obj} using late binding!");

        // Récupère l'information pour TurboBoost.
        MethodInfo mi = miniVan.GetMethod("TurboBoost");

        // Appelle la méthode ('null' pour aucun pramètre)
        mi.Invoke(obj, null);
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.Message);
    }
}
```

À ce stade, vous verrez apparaître dans la console le message indiquant que votre moteur a explosé.

## Appel de méthodes avec paramètres

Lorsque vous souhaitez utiliser la liaison tardive pour appeler une méthode nécessitant des paramètres, **vous devez regrouper les arguments sous forme de tableau d'objets faiblement typés**. La version de la classe `Car` qui possède une radio et la méthode suivante :

```cs
public void TurnOnRadio(bool musicOn, MusicMediaEnum mm)
	=> MessageBox.Show(musicOn ? $"Jamming {mm}" : "Quiet time...");
```

Cette méthode prend deux paramètres : un booléen indiquant si le système audio du véhicule doit être activé ou désactivé, et une énumération représentant le type de lecteur de musique. Rappelons que cette énumération est structurée comme suit :

```cs
public enum MusicMediaEnum
{
	musicCd, // 0
	musicTape, // 1
	musicRadio, // 2
	musicMp3 // 3
}
```

Voici une nouvelle méthode du fichier *Program.cs*, qui appelle `TurnOnRadio()`. ***==Notez que vous utilisez les valeurs numériques sous-jacentes de l'énumération MusicMediaEnum pour spécifier un lecteur multimédia « radio ».==***

```cs
static void InvokeMethodWithArgsUsingLateBinding(Assembly asm)
{
    try
    {
        // D'abord, récupére une description des métadonnées de SportsCar
        Type sport = asm.GetType("CarLibrary.SportsCar");

        // Maintenant, crée une instance de SportsCar.
        object obj = Activator.CreateInstance(sport);
        // Appelle TurnOnRadio() avec les arguments.
        MethodInfo mi = sport.GetMethod("TurnOnRadio");
        // Syntaxe permise par C# 12
        mi.Invoke(obj, [true, 2]);
        //mi.Invoke(obj, new object[] {true, 2});
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.Message);
    }
}
```

Idéalement, à ce stade, vous devriez percevoir les liens entre la réflexion, le chargement dynamique et la liaison tardive. Bien entendu, **l'API de réflexion offre de nombreuses fonctionnalités supplémentaires par rapport à ce qui a été abordé ici, mais vous devriez être en mesure d'approfondir le sujet si vous le souhaitez.**

==Vous vous demandez peut-être encore quand utiliser ces techniques dans vos propres applications==. **L'application extensible présentée plus loin dans ce chapitre devrait apporter des éclaircissements sur ce point**; cependant, le prochain sujet d'étude est le rôle des attributs .NET.

# Comprendre le rôle des attributs .NET

Comme illustré au début de ce chapitre, ***==l'un des rôles d'un compilateur .NET est de générer des métadonnées pour tous les types définis et référencés==***. Outre ces métadonnées standard contenues dans tout assembly, **la plateforme .NET offre aux programmeurs la possibilité d'intégrer des métadonnées supplémentaires à un assembly à l'aide d'*attributs***. **==En résumé, les attributs sont simplement des annotations de code qui peuvent être appliquées à un type donné==** (classe, interface, structure, etc.), **==un membre==** (propriété, méthode, etc.), **==un assembly ou un module.==**

**Les attributs .NET sont des types de classe qui étendent la classe de base abstraite `System.Attribute`**. ==En explorant les espaces de noms .NET, vous découvrirez de nombreux attributs prédéfinis que vous pourrez utiliser dans vos applications==. De plus, **vous pouvez créer des attributs personnalisés pour préciser le comportement de vos types en créant un nouveau type dérivant de `Attribute`**. ***==La bibliothèque de classes de base .NET fournit des attributs dans différents espaces de noms==***. Le [[#Tableau 17-3 En petit échantillon d'attributs prédéfinis|Tableau 17-3]] présente un aperçu de certains attributs prédéfinis, mais *==la liste est loin d'être exhaustive.==*

##### Tableau 17-3: En petit échantillon d'attributs prédéfinis

| Attribut         | Description                                                                                                                                                                                                                                       |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `[CLSCompliant]` | Imposera à l'élément annoté la conformité aux règles de la spécification de langage commun (CLS). Rappelons que les types conformes à la CLS sont garantis d'être utilisés de manière transparente dans tous les langages de programmation .NET.* |
| `[DllImport]`    | Permet au code .NET d'appeler n'importe quelle bibliothèque de code non managée basée sur C ou C++, y compris l'API du système d'exploitation sous-jacent.                                                                                        |
| `[Obsolete]`     | Signale un type ou un membre obsolète. Si d'autres programmeurs tentent d'utiliser un tel élément, ils recevront un avertissement du compilateur décrivant leur erreur.                                                                           |

Sachez que **lorsque vous appliquez des attributs dans votre code, les métadonnées intégrées sont essentiellement inutiles jusqu'à ce qu'un autre logiciel les analyse explicitement**. ==Dans le cas contraire, le texte contenant les métadonnées intégrées à l'assembly est ignoré et totalement inoffensif.==

## Consommateurs d'attributs

==Comme vous pouvez l'imaginer, le .NET Framework est fourni avec de nombreux utilitaires qui recherchent divers attributs==. **Le compilateur C#** (*csc.exe*) **est préprogrammé pour détecter la présence de divers attributs lors de la compilation**. Par exemple, ***==si le compilateur C# rencontre l'attribut `[CLSCompliant]`, il vérifie automatiquement l'élément attributé pour s'assurer qu'il n'expose que des constructions conformes à CLS==***. Autre exemple : ==si le compilateur C# détecte un élément attributé avec l'attribut `[Obsolete]`, il affiche un avertissement.==

**Outre les outils de développement, de nombreuses méthodes des bibliothèques de classes de base .NET sont préprogrammées pour analyser des attributs spécifiques**. **==Le [[Chapitre 19|Chapitre 19]] présente la sérialisation XML et JSON, toutes deux utilisant des attributs pour contrôler le processus de sérialisation.==**

Enfin, **vous pouvez créer des applications programmées pour interagir avec vos propres attributs personnalisés, ainsi qu'avec tout attribut des bibliothèques de classes de base .NET**. ==Vous pouvez ainsi créer un ensemble de « mots clés » interprétés par un ensemble spécifique d'assemblies.

## Utilisation des attributs en C#

Pour illustrer l'application des attributs en C#, créez un projet d'application console nommé
*ApplyingAttributes* et ajoutez une référence au package NuGet `System.Text.Json`. Mettez à jour le fichier *Program.cs* en y incluant les directives using globales suivantes :

```cs
global using System.Text.Json.Serialization;
global using System.Xml.Serialization;
```

**Supposons que vous souhaitiez créer une classe nommée `Motorcycle` pouvant être exportée au format JSON. Si vous avez un champ qui ne doit pas être exporté au format JSON, vous pouvez utiliser l'attribut `[JsonIgnore]`.**

```cs
namespace ApplyingAttributes;

public class MotorCycle
{
    [JsonIgnore]
    public float weightOfCurrentPassengers;

    // Ces champs sont toujous sérialisable
    public bool hasRadioSystem;
    public bool hasHeadSet;
    public bool hasSissyBar;
}
```

>[!note]
>un attribut s'applique à l'élément "juste après".

À ce stade, ne vous préoccupez pas du processus de sérialisation des objets (le [[Chapitre 19|Chapitre 19]] traite des détails). **Notez simplement que pour appliquer un attribut, son nom est entre crochets.**

**Comme vous pouvez l'imaginer, un élément peut être associé à plusieurs attributs**. ==Prenons l'exemple d'une classe C# hérité== (`HorseAndBuggy`) ==qui utilisait un espace de noms XML personnalisé==. **Le code a évolué, et la classe est désormais obsolète**. **==Plutôt que de supprimer la définition de la classe (et risquer de casser le logiciel existant), vous pouvez la marquer avec l'attribut `[Obsolete]`==**. **Pour appliquer plusieurs attributs à un élément, utilisez simplement une liste séparée par des virgules, comme ceci :**

```cs
namespace ApplyingAttributes;

[
    XmlRoot(Namespace = "http;//www.MyCompany.com"),
    Obsolete("Use another vehicle!")
]
public class HorseAndBuggy
{
    // ...
}
```

Autre solution : vous pouvez également appliquer plusieurs attributs à un seul élément en les empilant comme suit :

```cs
namespace ApplyingAttributes;

[XmlRoot(Namespace = "http;//www.MyCompany.com")]
[Obsolete("Use another vehicle!")]
public class HorseAndBuggy
{
    // ...
}
```

>[!success] La dernière est la version préférable pour du C# moderne (plus lisible)

## Notation abrégée des attributs C#

**Si vous avez consulté la documentation .NET, vous avez peut-être remarqué que le nom de classe réel de l'attribut `[Obsolete]` est `ObsoleteAttribute`, et non `Obsolete`**. **==Par convention, tous les attributs .NET==** (y compris les attributs personnalisés que vous pouvez créer) **==sont suffixés par le jeton `Attribute`==**. **Cependant, pour simplifier l'application des attributs, le langage C# n'exige pas la saisie du suffixe `Attribute`**. Ainsi, ==l'itération suivante du type `HorseAndBuggy` est identique à l'exemple précédent== (elle nécessite simplement quelques frappes supplémentaires) :

```cs
[XmlRootAttribute(Namespace = "http://www.MyCompany.com")]
[ObsoleteAttribute("Use another vehicle!")]
public class HorseAndBuggy
{
	// ...
}
```

**Notez que ceci est une fonctionnalité offerte par C#. Tous les langages .NET ne prennent pas en charge cette syntaxe d'attribut abrégée.**

## Spécification des paramètres du constructeur pour les attributs

**Notez que l'attribut `[Obsolete]` peut accepter ce qui semble être un paramètre de constructeur**. ***==Si vous consultez la définition formelle de l'attribut `[Obsolete]`, vous constaterez que cette classe fournit effectivement un constructeur recevant un `System.String`==***.

```cs
public sealed class ObsoleteAttribute : Attribute
{
	public ObsoleteAttribute();
	public ObsoleteAttribute(string? message);
	public ObsoleteAttribute(string? message, bool error);
	public string? Message { get; }
	public bool IsError { get; }
	public string DiagnosticId { get; set; }
	public string UrlFormat { get; set; }
}
```

Il est important de comprendre que **lorsque vous fournissez des paramètres de constructeur à un attribut, celui-ci *n'est pas* alloué en mémoire tant que les paramètres ne sont pas pris en compte par un autre type ou un outil externe**. ==Les données de type chaîne définies au niveau de l'attribut sont simplement stockées dans l'assembly sous forme de métadonnées.==

## L'attribut obsolète en pratique
Maintenant que `HorseAndBuggy` est marqué comme obsolète, si vous deviez allouer une instance de ce type :

```cs
using ApplyingAttributes;

Console.WriteLine("Hello, World!");
HorseAndBuggy mule = new HorseAndBuggy();
```

**Vous constaterez qu'un avertissement du compilateur est émis**. **==Il s'agit de l'avertissement `CS0618`, et le message comprend les informations transmises à l'attribut.==**

```
'HorseAndBuggy' is obsolete: 'Use another vehicle!'
```

*Roslyn* prend également en charge IntelliSense, qui obtient ses informations **par lui-même sur le projet actuel**, et via la **réflexion pour les *.dll* externes**. La première image présente les résultats de l'attribut Obsolete dans Visual Studio via IntelliSense, et la deuxième image présente le message plus détaillé dans l'éditeur de code de Visual Studio. **Notez que *roslyn* utilisent le terme déprécié au lieu d'obsolète.

>[!info] Précision pour neovim
>Tree-sitter s'occupe de la **forme** (le texte), Roslyn s'occupe du **fond** (la logique et les types). La réflexion n'est qu'un petit outil que Roslyn utilise parfois pour lire les DLL compilées des bibliothèques externes.

![[Figure 17.1.png|Attributs en action dans Visual Studio]]

![[Figure 17.2.png|Survoler les types obsolètes dans la fenêtre de l'éditeur Visual Studio]]

Les images suivantes montrent les résultats de l'attribut `Obsolete` dans Visual Studio Code.

![[Figure 17.3.png|Les attributs en action dans Visual Studio Code]]

![[Figure 17.4.png|Survoler les types obsolètes dans l'éditeur Visual Studio Code]]

Enfin, les images suivante montrent aussi les même résultat de l'attribut `Obsolete` dans Neovim

![[Figure 17.3.5.png|Attribut en action dans neovim]]

![[Figure 17.4.5.png|Survoler (diagnostics) les types obsolètes dans l'éditeur Neovim]]

Idéalement, à ce stade, vous devriez comprendre les points clés suivants concernant les attributs .NET :

- Les attributs sont des classes dérivées de `System.Attribute`.
- Les attributs génèrent des métadonnées intégrées.
- Les attributs sont inutilisables tant qu’un autre agent (y compris les EDI) ne les utilise pas.
- En C#, les attributs sont appliqués à l’aide de crochets.

Ensuite, **==examinons comment créer vos propres attributs personnalisés et un logiciel personnalisé qui reflète les métadonnées intégrées.==**

# Création d'attributs personnalisés

**La première étape consiste à créer une nouvelle classe dérivée de `System.Attribute`**. ==Dans la continuité du thème automobile abordé dans cet ouvrage, supposons que vous ayez créé un nouveau projet de bibliothèque de classes C# nommé *AttributedCarLibrary*.

Cet assembly définira plusieurs véhicules, chacun étant décrit par un attribut personnalisé nommé `VehicleDescriptionAttribute`, comme suit :

```cs
namespace AttributedCarLibrary;

// Un attribut personnalisé
public sealed class VehicleDescriptionAttribute : Attribute
{
    public string Description { get; set; }

    public VehicleDescriptionAttribute() { }

    public VehicleDescriptionAttribute(string desciption) =>
        Description = desciption;
}
```

Comme vous pouvez le constater, **`VehicleDescriptionAttribute` conserve une donnée de type `string` à l'aide d'une propriété automatique (`Description`)**. ==Hormis le fait que cette classe hérite de `System.Attribute`, sa définition ne présente rien de particulier.==

>[!note]
>**Pour des raisons de sécurité, il est recommandé, selon les bonnes pratiques .NET, de concevoir tous les attributs personnalisés comme scellés (`sealed`)**.
>
>En effet, *Roslyn* proposent un extrait de code nommé `Attribute` qui génère automatiquement une nouvelle classe dérivée de `System.Attribute` dans votre fenêtre de code.

## Application d'attributs personnalisés

**Étant donné que `VehicleDescriptionAttribute` hérite de `System.Attribute`, vous pouvez désormais annoter vos véhicules comme bon vous semble**. À des fins de test, ajoutez les classes suivantes à votre nouvelle bibliothèque de classes :

```cs
// MotorCycle.cs
namespace AttributedCarLibrary;

// Assigne une description à l'aide d'une "propriété nommée"
[VehicleDescription(Description = "My rocking Harley")]
public class MotorCycle { }
```

```cs
// HorseAndBuggy.cs
namespace AttributedCarLibrary;

[Obsolete("Use another vehicle!")]
[VehicleDescription("The old gray mare, she ain't what she used to be...")]
public class HorseAndBuggy { }
```

```cs
// Winnebago.cs
namespace AttributedCarLibrary;

[VehicleDescription("A very long, slow, but feature-rich auto")]
public class Winnebago { }
```

## Syntaxe des propriétés nommées

==Notez que la description de `Motorcycle` se voit attribuer une description à l'aide d'une nouvelle syntaxe d'attribut appelée *propriété nommée*==. **Dans le constructeur du premier attribut `[VehicleDescription]`, vous définissez les données de chaîne sous-jacentes à l'aide de la propriété `Description`**. **Si cet attribut est utilisé par un agent externe, la valeur est transmise à la propriété `Description`** (**==la syntaxe des propriétés nommées n'est autorisée que si l'attribut fournit une propriété .NET accessible en écriture==**).

==En revanche, les types `HorseAndBuggy` et `Winnebago`== n'utilisent pas la syntaxe des propriétés nommées et ==transmettent simplement les données de chaîne via le constructeur personnalisé==. **Dans tous les cas, une fois l'assembly `AttributedCarLibrary` compilé, vous pouvez utiliser *ildasm.exe* pour afficher les descriptions des métadonnées injectées pour votre type**. Par exemple, ce qui suit montre la description intégrée de la classe `Winnebago` :

```CIL
// 	CustomAttribute #1 (0c000014)
// 	-------------------------------------------------------
// 		CustomAttribute Type: 06000008
// 		CustomAttributeName: AttributedCarLibrary.VehicleDescriptionAttribute :: instance void .ctor(class System.String)
// 		Length: 45
// 		Value : 01 00 28 41 20 76 65 72  79 20 6c 6f 6e 67 2c 20 >  (A very long, <
//                       : 73 6c 6f 77 2c 20 62 75  74 20 66 65 61 74 75 72 >slow, but featur<
//                       : 65 2d 72 69 63 68 20 61  75 74 6f 00 00          >e-rich auto     <
// 		ctor args: ("A very long, slow, but feature-rich auto")
```

## Restriction de l'utilisation des attributs

**Par défaut, les attributs personnalisés peuvent être appliqués à presque tous les aspects de votre code** (méthodes, classes, propriétés, etc.). Ainsi, **==si cela s'avère pertinent, vous pouvez utiliser `VehicleDescription` pour qualifier des méthodes, des propriétés ou des champs==** (entre autres).

```cs
namespace AttributedCarLibrary;

[VehicleDescription("A very long, slow, but feature-rich auto")]
public class Winnebago
{
    [VehicleDescription("My rocking CD player")]
    public void PlayMusic(bool On) { }
}
```

==Dans certains cas, ce comportement est exactement celui que vous recherchez==. Cependant, **il peut arriver que vous souhaitiez créer un attribut personnalisé applicable uniquement à certains éléments de code**. ***==Si vous souhaitez limiter la portée d'un attribut personnalisé, vous devrez appliquer l'attribut `[AttributeUsage]` à la définition de votre attribut personnalisé==***. **L'attribut `[AttributeUsage]` vous permet de fournir n'importe quelle combinaison de valeurs** (via une opération `OR`) **issues de l'énumération `AttributeTargets`**, comme ceci :

```cs
// Cette énumération définit les cibles possibles d'un attribut.
public enum AttributeTargets
{
	All, Assembly, Class, Constructor,
	Delegate, Enum, Event, Field, GenericParameter,
	Interface, Method, Module, Parameter,
	Property, ReturnValue, Struct
}
```

**De plus, `[AttributeUsage]` vous permet également de définir, en option, une propriété nommée (`AllowMultiple`) qui indique si l'attribut peut être appliqué plusieurs fois au même élément** (la valeur par défaut est `false`). De même, **`[AttributeUsage]` vous permet de déterminer si l'attribut doit être hérité par les classes dérivées à l'aide de la propriété nommée `Inherited` (la valeur par défaut est `true`).

**Pour indiquer que l'attribut `[VehicleDescription]` ne peut être appliqué qu'une seule fois à une classe ou une structure, vous pouvez mettre à jour la définition de `VehicleDescriptionAttribute` comme suit** :

```cs
// Cette fois-ci, nous utilisons l'attribut AttributeUsage
// pour annoter notre attribut personnalisé.
[AttributeUsage(
    AttributeTargets.Class | AttributeTargets.Struct,
    Inherited = false
)]
public sealed class VehicleDescriptionAttribute : Attribute
{
	...
}
```

*==Ainsi, si un développeur tente d'appliquer l'attribut `[VehicleDescription]` à autre chose qu'une classe ou une structure, une erreur de compilation lui est signalée.==*

# Attributs au niveau de l'assembly

**Il est également possible d'appliquer des attributs à tous les types d'un assembly donné à l'aide de la balise `[assembly:]`**. **==Par exemple, supposons que vous souhaitiez garantir que chaque membre public de chaque type public défini dans votre assembly est conforme à la spécification CLS==**. Pour ce faire, **ajoutez simplement l'attribut au niveau de l'assembly suivant en haut de n'importe quel fichier de code source C#, comme ceci** (en dehors de toute déclaration d'espace de noms) :

```cs
[asssembly: CLSCompliant]
namespace AttributedCarLibrary;
```

>[!note]
>Tous les attributs de niveau de l'assembly ou de niveau de module doivent être listés en dehors de la portée de tout espace de noms.

## Utilisation d'un fichier séparé pour les attributs d'assembly

**Une autre approche consiste à ajouter un nouveau fichier à votre projet, nommé par exemple** *AssemblyAttributes.cs* (==n'importe quel nom convient, mais il doit indiquer la fonction du fichier==), **et à y placer vos attributs d'assembly**. Dans le .NET Framework, ==il était recommandé d'utiliser *AssemblyInfo.cs*==; *==cependant, avec .NET (Core), ce fichier n'est plus utilisable==*. **Placer les attributs dans un fichier séparé facilitera la recherche des attributs appliqués au projet par les autres développeurs.**

>[!note]
>.NET a introduit deux changements importants. Premièrement, le fichier *AssemblyInfo.cs* est désormais généré automatiquement à partir des propriétés du projet et n'est pas destiné aux développeurs. Deuxièmement (et c'est un changement lié), **de nombreux attributs d'assembly** (Version, Company, etc.) **ont été remplacés par des propriétés du fichier projet.**

Créez un fichier nommé *AssemblyAttributes.cs* et mettez-le à jour comme suit :

```cs
// Liste maintenant les attributs au niveau de l'assembly ou du module.
// Appliquer la conformité CLS à tous les types publics de cet assembly.
[assembly: CLSCompliant(true)]
```

**Si vous ajoutez maintenant un peu de code qui ne respecte pas la spécification CLS** (comme un point exposé de données non signées), **vous recevrez un avertissement du compilateur.**

```cs
public class Winnebago
{
    public ulong notCompliant;
}
```

# Utilisation du fichier projet pour les attributs d'assembly

Comme indiqué au [[Chapitre 16#Utilisation d'un attribut d'assembly|Chapitre 16]] avec l'attribut `InternalsVisibleToAttribute`, **les attributs d'assembly peuvent également être ajoutés au fichier projet**. ***==Cependant, seuls les attributs de paramètre à chaîne unique peuvent être utilisés de cette manière==***. ==Cela s'applique aux propriétés configurables dans l'onglet `Package` des propriétés du projet.

>[!warning] Seulement les attributs "standards" que Microsoft a pré-programmé peuvent être mis dans le fichier *.csproj*

>[!note]
>Au moment de la rédaction de ce document, une discussion est en cours sur le dépôt GitHub de msBuild concernant l'ajout de la prise en charge des paramètres non-chaînes de caractères. Cela permettrait d'ajouter l'attribut `CLSCompliant` directement dans le fichier projet, et non plus dans un fichier *.cs.

**Définissez certaines propriétés** (telles que `Auteurs` et `Description`) en cliquant avec le bouton droit sur le projet dans l'Explorateur de solutions, en sélectionnant Propriétés, puis en cliquant sur `Package`. **Ajoutez également l'attribut `InternalsVisibleToAttribute`, comme vous l'avez fait au [[Chapitre 16#Utilisation du fichier projet|Chapitre 16]]. Votre fichier projet devrait maintenant ressembler à ceci :

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net10.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>disable</Nullable>
    <Authors>Gianni Mucci</Authors>
    <Descrption>This is a simple car library with attributes</Descrption>
  </PropertyGroup>

</Project>
```

Après avoir compilé votre projet, accédez au répertoire `\obj\Debug\net10.0` et recherchez le fichier
*AttributedCarLibrary.AssemblyInfo.cs*. Ouvrez-le ; vous y verrez ces propriétés sous forme d’attributs (malheureusement, le format n’est pas très lisible).

```cs
using System;
using System.Reflection;

[assembly: System.Reflection.AssemblyCompanyAttribute("Gianni Mucci")]
[assembly: System.Reflection.AssemblyConfigurationAttribute("Debug")]
[assembly: System.Reflection.AssemblyFileVersionAttribute("1.0.0.0")]
[assembly: System.Reflection.AssemblyInformationalVersionAttribute("1.0.0+fed92f5413945f9a51ec8416e8c940ca58b235b7")]
[assembly: System.Reflection.AssemblyProductAttribute("AttributedCarLibrary")]
[assembly: System.Reflection.AssemblyTitleAttribute("AttributedCarLibrary")]
[assembly: System.Reflection.AssemblyVersionAttribute("1.0.0.0")]
```

>[!tip]- Pourquoi `VersionAttribute` génère "+..." en plus de la version ?
>
>Depuis .NET 8, le SDK inclut par défaut une fonctionnalité appelée **Source Link**. Son but est de lier un binaire (.dll) à son code source exact sur GitHub ou GitLab.
>
>- Si vous déployez une application et qu'un bug survient, le `AssemblyInformationalVersion` vous permet de savoir exactement **quelle version du code** a généré ce binaire, même si vous avez oublié de changer le numéro `1.0.0`
>
>Si vous trouvez cela gênant ou si vous ne voulez pas exposer vos hashs Git, vous pouvez désactiver ce comportement dans votre **`.csproj`** :
>```xml
><PropertyGroup>
>	<IncludeSourceRevisionInInformationalVersion>
>	false
>	</IncludeSourceRevisionInInformationalVersion>
</PropertyGroup>
>```

Pour conclure sur les attributs d'assemblage, vous pouvez désactiver la génération de la classe *AssemblyInfo.cs* si vous souhaitez gérer vous-même le processus.

>[!info] En résumé
>
>- **Simple texte** (Nom, Version, Description) ➡️ **`.csproj`**
>- **Paramètres typés** (Booléens, Enums, Types complexes) ➡️ **Fichier `.cs`**

# Réflexion sur les attributs à l'aide de la liaison anticipée

**N'oubliez pas qu'un attribut est inutile tant qu'un autre logiciel n'a pas pris connaissance de ses valeurs**. **==Une fois l'attribut détecté, ce logiciel peut entreprendre les actions nécessaires==**. Comme toute application, ==ce logiciel peut détecter la présence d'un attribut personnalisé par liaison anticipée ou tardive==. **Pour utiliser la liaison anticipée, l'application cliente doit disposer d'une définition de l'attribut** (ici, `VehicleDescriptionAttribute`) **au moment de la compilation**. **==L'assembly *AttributedCarLibrary* définissant cet attribut personnalisé comme une classe publique, la liaison anticipée est la meilleure option.==**

Pour illustrer le processus de réflexion sur les attributs personnalisés, ajoutez à la solution un nouveau projet d'application console C# nommé *VehicleDescriptionAttributeReader*. Ensuite, ajoutez une référence au projet *AttributedCarLibrary*. À l'aide de l'interface de ligne de commande (CLI), exécutez les commandes suivantes (chaque commande doit figurer sur une ligne distincte) :

```bash
dotnet new console -n VehicleDescriptionAttributeReader
dotnet sln Chapter17_AllProjects.sln add VehicleDescriptionAttributeReader
dotnet add VehicleDescriptionAttributeReader reference AttributedCarLibrary
```

Mettez à jour le fichier *Program.cs* avec le code suivant :

```cs
using AttributedCarLibrary;

Console.Title = "Value of VehicleDEscriptionAttribute";
Console.WriteLine("***** Value of VehicleDEscriptionAttribute *****\n");

ReflectOnAttributesUsingEarlyBinding();
Console.ReadLine();

void ReflectOnAttributesUsingEarlyBinding()
{
    // Récupère un Type représentant le Winnebago
    Type t = typeof(Winnebago);

    // Récupère tous les attributs de Winnebago
    object[] customAtts = t.GetCustomAttributes(false);

    // Affiche la description
    foreach (VehicleDescriptionAttribute v in customAtts)
    {
        Console.WriteLine($"-> {v.Description}\n");
    }
}
```

**==La méthode `Type.GetCustomAttributes()` renvoie un tableau d'objets représentant tous les attributs appliqués au membre représenté par le type==** (==le paramètre booléen détermine si la recherche doit remonter la chaîne d'héritage==). **Une fois la liste des attributs obtenue, parcourez chaque classe `VehicleDescriptionAttribute` et affichez la valeur obtenue par la propriété `Description`.**

# Utilisation de la liaison tardive pour analyser les attributs

L'exemple précédent utilisait la liaison anticipée pour afficher les données de description du véhicule de type `Winnebago`. **Ceci était possible car le type de classe `VehicleDescriptionAttribute` était défini comme membre `public` dans l'assembly `AttributedCarLibrary`. Il est également possible d'utiliser le chargement dynamique et la liaison tardive pour analyser les attributs.**

Ajoutez un nouveau projet nommé *VehicleDescriptionAttributeReaderLateBinding* à la solution, définissez-le comme projet de démarrage et copiez *AttributedCarLibrary.dll* dans le dossier du projet (ou `\bin\Debug\net10.0` si vous utilisez Visual Studio). Ensuite, mettez à jour votre fichier *Program.cs* comme suit :

```cs
using System.Reflection;

Console.Title = "Value of VehicleDescriptionAttribute";
Console.WriteLine("***** Value of VehicleDescriptionAttribute *****\n");
ReflectAttributesUsingLateBinding();
Console.ReadLine();

static void ReflectAttributesUsingLateBinding()
{
    try
    {
        // Charge la copie local de AttributeCarLibrary
        Assembly asm = Assembly.LoadFrom("AttributedCarLibrary.dll");

        // Obtient les informations de type de l'attribut
        // VehicleDescriptionAttribute.
        Type vehicleDesc = asm.GetType(
            "AttributedCarLibrary.VehicleDescriptionAttribute"
        );

        // Obtenir les informations de type de la propriété Description.
        PropertyInfo propDesc = vehicleDesc.GetProperty("Descrption");

        // Récupère tous les type de l'assembly
        Type[] types = asm.GetTypes();

        // Parcourir chaque attribut VehicleDescriptionAttribute
        // et afficher la description en utilisant la liaison tardive.
        foreach (Type t in types)
        {
            object[] objs = t.GetCustomAttributes(vehicleDesc, false);
            foreach (object o in objs)
            {
                Console.WriteLine(
                    $"-> {t.Name}: {propDesc.GetValue(o, null)}\n"
                );
            }
        }
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.Message);
    }
}
```

==Si vous avez pu suivre les exemples de ce chapitre, ce code devrait être== (plus ou moins) ==compréhensible==. **Le seul point important est l'utilisation de la méthode `PropertyInfo.GetValue()`, qui permet de déclencher l'acesseur de la propriété**. Voici le résultat de l'exemple actuel :

```
***** Value of VehicleDescriptionAttribute *****

-> HorseAndBuggy: The old gray mare, she ain't what she used to be...

-> MotorCycle: My rocking Harley

-> Winnebago: A very long, slow, but feature-rich auto
```

# Mise en perspective de la réflexion, de la liaison tardive et des attributs personnalisés

Même si vous avez vu de nombreux exemples de ces techniques en action, ==vous vous demandez peut-être encore quand utiliser la réflexion, le chargement dynamique, la liaison tardive et les attributs personnalisés dans vos programmes==. Il est vrai que ces sujets peuvent sembler un peu théoriques (ce qui peut être un avantage ou un inconvénient, selon votre point de vue). **Pour mieux comprendre ces concepts et les appliquer à une situation concrète, un exemple concret est nécessaire**. **==Imaginez que vous faites partie d'une équipe de développement qui crée une application répondant aux exigences suivantes==** :

***Le produit doit pouvoir être extensible à l'aide d'outils tiers supplémentaires.***

==Que signifie exactement *extensible* ?== **Prenons l’exemple des environnements de développement intégrés** (IDE) Visual Studio ou Visual Studio Code. **Lors du développement de cette application, divers « points d’ancrage » ont été insérés dans le code source pour permettre à d’autres éditeurs de logiciels d’intégrer (ou de brancher) des modules personnalisés à l’IDE**. *==De toute évidence, les équipes de développement de Visual Studio/Visual Studio Code n’avaient aucun moyen de référencer des assemblies .NET externes qui n’avaient pas encore été développés (donc pas de liaison anticipée)==*. ***==Comment une application peut-elle alors fournir les points d’ancrage nécessaires ?==*** Voici une solution possible à ce problème :

1. Premièrement, **une application extensible doit fournir un mécanisme de saisie permettant à l'utilisateur de spécifier le module à intégrer** (par exemple, une boîte de dialogue ou une option en ligne de commande). Ceci nécessite un *chargement dynamique*.

2. Deuxièmement, **une application extensible doit pouvoir déterminer si le module prend en charge les fonctionnalités requises** (par exemple, un ensemble d'interfaces nécessaires) pour être intégré à l'environnement. Ceci nécessite la *réflexion*.

3. Enfin, **une application extensible doit obtenir une référence à l'infrastructure requise** (par exemple, un ensemble de types d'interfaces) **et appeler les membres pour déclencher les fonctionnalités sous-jacentes**. Ceci peut nécessiter une *liaison tardive*.

**==En clair, si l'application extensible a été préprogrammé pour interroger des interfaces spécifiques, elle peut déterminer à l'exécution si le type peut être activé==**. ==Une fois ce test de vérification réussi, le type en question peut prendre en charge des interfaces supplémentaires qui offrent une structure polymorphe à ses fonctionnalités==. C'est précisément l'approche adoptée par l'équipe Visual Studio et, contrairement à ce que vous pourriez penser, elle est très simple !

# Création d'une application extensible

Dans les sections suivantes, je vous présenterai un exemple illustrant le processus de création d'une application extensible grâce aux fonctionnalités d'assemblies externes. Pour vous guider, cette application extensible comprend les assemblies suivants :

- *CommonSnappableTypes.dll* : Cet assembly contient les définitions de types qui seront utilisées par chaque composant logiciel enfichable et directement référencées par l'application Windows Forms/Avalonia.

- *CSharpSnapIn.dll* : Un composant logiciel enfichable écrit en C#, qui exploite les types de *CommonSnappableTypes.dll*.

- *VBSnapIn.dll* : Un composant logiciel enfichable écrit en Visual Basic, qui exploite les types de *CommonSnappableTypes.dll*.

- *MyExtendableApp.exe* : Une application console extensible par les fonctionnalités de chaque composant logiciel enfichable.

**Cette application utilisera le chargement dynamique, la réflexion et la liaison tardive pour acquérir dynamiquement les fonctionnalités des assemblies dont elle n'a aucune connaissance préalable.**

>[!note]
>Vous vous dites peut-être :  "Mon patron ne m’a jamais demandé de développer une application console", et vous avez probablement raison ! Les applications métier développées en C# appartiennent généralement à la catégorie des clients intelligents (WinForms ou WPF), des applications web/services RESTful (ASP.NET Core) ou des processus sans interface graphique (Azure Functions, services Windows, etc.). Nous utilisons des applications console pour nous concentrer sur les concepts spécifiques de cet exemple, en l’occurrence le chargement dynamique, la réflexion et la liaison tardive. Plus loin dans cet ouvrage, vous découvrirez de « vraies » applications destinées aux utilisateurs, utilisant WPF/Avalonia et ASP.NET Core.

## Création d'une solution `ExtendableApp` multiprojet

Jusqu'à présent dans ce livre, la plupart des applications étaient des projets autonomes, à quelques exceptions près (comme la précédente). Ce choix visait à simplifier et à cibler les exemples. **Cependant, dans le développement réel, on travaille généralement avec plusieurs projets au sein d'une même solution.**

### Création de la solution et des projets avec l'interface de ligne de commande (CLI)

Pour commencer à utiliser la CLI, saisissez les commandes suivantes afin de créer une solution, les bibliothèques de classes et une application console, ainsi que les références du projet :

```bash
dotnet new sln -n Chapter17_ExtendableApp

dotnet new classlib -n CommonSnappableTypes
dotnet sln Chapter17_ExtendableApp.slnx add CommonSnappableTypes

dotnet new classlib -n CSharpSnapIn 
dotnet sln Chapter17_ExtendableApp.slnx add CSharpSnapIn
dotnet add CSharpSnapin reference CommonSnappableTypes

dotnet new classlib -lang vb -n VBSnapIn
dotnet sln Chapter17_ExtendableApp.slnx add VBSnapIn
dotnet add VBSnapIn reference CommonSnappableTypes

dotnet new console -n MyExtendableApp
dotnet sln Chapter17_ExtendableApp.slnx add MyExtendableApp
dotnet add MyExtendableApp reference CommonSnappableTypes
```

#### Ajout d'événements post-compilation aux fichiers projet

**Lors de la compilation d'un composant** (que ce soit depuis la ligne de commande `dotnet` ou un IDE), **certains événements peuvent être interceptés**. Pour notre architecture extensible, **nous souhaitons copier automatiquement les assemblies des composants logiciels enfichables vers le répertoire du projet de l'application console** (`MyExtendableApp`) après chaque compilation réussie.

Pour y parvenir de manière propre et multiplateforme (sans utiliser de commandes spécifiques à un système comme `copy` ou `cp`), ouvrez les fichiers de vos composants (_CSharpSnapIn.csproj_ et _VBSnapIn.vbproj_) et ajoutez ce bloc de code à la toute fin, juste avant la balise de fermeture `</Project>` :

```xml
<Target Name="PostBuild" AfterTargets="PostBuildEvent">
    <!-- Copie nativement le binaire compilé vers le dossier de l'application principale -->
    <Copy SourceFiles="$(TargetPath)" DestinationFolder="$(ProjectDir)../MyExtendableApp/" />
</Target>
```

### Création de la solution et des projets avec Visual Studio

Par défaut, Visual Studio nomme la solution comme le premier projet créé dans cette solution. Vous pouvez toutefois facilement modifier le nom de la solution, comme illustré dans l'image suivante.

![[Figure 17.5.png|Création du projet CommonSnappableTypes et de la solution ExtendableApp]]

Pour créer la solution *ExtendableApp*, commencez par sélectionner Fichier -> Nouveau projet pour ouvrir la boîte de dialogue Nouveau projet. Sélectionnez Bibliothèque de classes et saisissez le nom `CommonSnappableTypes`. Avant de cliquer sur OK, saisissez le nom de la solution ExtendableApp, comme illustré dans l'image précédente.

Pour ajouter un autre projet à la solution, cliquez avec le bouton droit sur le nom de la solution (*ExtendableApp*) dans l'Explorateur de solutions (ou cliquez sur Fichier -> Ajouter -> Nouveau projet) et sélectionnez Ajouter -> Nouveau projet. Lorsque vous ajoutez un autre projet à une solution existante, la boîte de dialogue Ajouter un nouveau projet est légèrement différente : les options de la solution n'apparaissent plus, vous ne verrez donc que les informations du projet (nom et emplacement). Nommez le projet de bibliothèque de classes *CSharpSnapIn* et cliquez sur Créer.

Ensuite, ajoutez une référence au projet *CommonSnappableTypes* depuis le projet *CSharpSnapIn*. Pour ce faire dans Visual Studio, cliquez avec le bouton droit sur le projet *CSharpSnapIn* et sélectionnez Ajouter -> Référence de projet. Dans la boîte de dialogue Gestionnaire de références, sélectionnez Projets -> Solution dans le volet gauche (si ce n'est pas déjà fait) et cochez la case en regard de *CommonSnappableTypes*.

Répétez l'opération pour une nouvelle bibliothèque de classes Visual Basic (*VBSnapIn*) qui référence le projet *CommonSnappableTypes*.

Le dernier projet à ajouter est une application console .NET nommée *MyExtendableApp*. Ajoutez une référence au projet *CommonSnappableTypes* et définissez l'application console comme projet de démarrage de la solution. Pour ce faire, cliquez avec le bouton droit sur le projet *MyExtendableApp* dans l'Explorateur de solutions et sélectionnez Définir comme projet de démarrage.

#### Configuration des dépendances de génération du projet

Lorsque Visual Studio reçoit la commande d'exécuter une solution, les projets de démarrage et tous les projets référencés sont générés si des modifications sont détectées ; toutefois, les projets non référencés ne le sont pas. Ce comportement peut être modifié en configurant les dépendances du projet. Pour ce faire, cliquez avec le bouton droit sur la solution dans l'Explorateur de solutions, sélectionnez Ordre de génération du projet, et, dans la boîte de dialogue qui s'affiche, sélectionnez l'onglet Dépendances et sélectionnez *MyExtendableApp*.

Notez que le projet *CommonSnappableTypes* est déjà sélectionné et que la case à cocher est décochée. En effet, il est référencé directement. Cochez également les cases des projets *CharpSnapIn* et *VBSnapIn*, comme illustré dans l'image suivante.

![[Figure 17.6.png|Accéder au menu contextuel de l'ordre de construction du projet]]

Désormais, à chaque compilation du projet *MyExtendableApp*, les projets *CSharpSnapIn* et *VBSnapIn* sont également compilés.

#### Ajout d'événements post-compilation

Ouvrez les propriétés du projet *CSharpSnapIn* (clic droit sur l'Explorateur de solutions, puis sélectionnez Propriétés) et accédez à la page Événements de génération (C#). Cliquez sur le bouton Modifier les événements post-compilation, puis sur Macros >>. Vous pouvez alors consulter les macros disponibles, qui font toutes référence à des chemins d'accès et/ou des noms de fichiers. L'avantage d'utiliser ces macros dans les événements de génération est qu'elles sont indépendantes de la machine et fonctionnent avec des chemins relatifs. Par exemple, je travaille dans un répertoire nommé `c-sharp-wf\code\chapter17`. Vous utilisez probablement un répertoire différent. Grâce aux macros, MSBuild utilisera toujours le chemin d'accès correct relatif aux fichiers *.csproj.

Dans la zone Post-compilation, saisissez le texte suivant (sur deux lignes) :

```cmd
copy $(TargetPath) $(SolutionDir)MyExtendableApp\$(OutDir)$(TargetFileName) /Y
copy $(TargetPath) $(SolutionDir)MyExtendableApp\$(TargetFileName) /Y
```

Procédez de la même manière pour le projet *VBSnapIn*, en remplaçant la page de propriétés par « Compiler ». À partir de là, cliquez sur le bouton « Événements de génération ».

Une fois ces commandes d'événements post-génération ajoutées, chaque assembly sera copié dans les répertoires de projet et de sortie de MyExtendableApp à chaque compilation.

## Création de `CommonSnappableTypes.dll`

**Dans le projet** *CommonSnappableTypes*, **supprimez le fichier** *Class1.cs* **par défaut, ajoutez un nouveau fichier d'interface nommé** *IAppFunctionality.cs* **et mettez-le à jour comme suit** :

```cs
namespace CommonSnappableTypes;

public interface IAppFunctionality
{
    void DoIt();
}
```

**Ajoutez un fichier de classe nommé** *CompanyInfoAttribute.cs* et mettez-le à jour comme suit :

```cs
namespace CommonSnappableTypes;

[AttributeUsage(AttributeTargets.Class)]
public sealed class CompanyInfoAttribute : System.Attribute
{
    public string CompanyName { get; set; }
    public string CompanyUrl { get; set; }
}
```

**L'interface `IAppFunctionality` fournit une interface polymorphe pour tous les composants logiciels enfichables pouvant être utilisés par l'application extensible**. ==Cet exemple étant purement illustratif, vous fournissez une seule méthode nommée `DoIt()`.==

**Le type `CompanyInfoAttribute` est un attribut personnalisé qui peut être appliqué à tout type de classe souhaitant être intégré au conteneur**. Comme vous pouvez le constater dans la définition de cette classe, **==`[CompanyInfo]` permet au développeur du composant logiciel enfichable de fournir des informations de base sur son point d'origine.==**

## Création du composant logiciel enfichable C#

Dans le projet *CSharpSnapIn*, supprimez le fichier *Class1.cs* et ajoutez un nouveau fichier nommé *CSharpModule.cs*. Mettez à jour le code comme suit :

```cs
using CommonSnappableTypes;

namespace CsharpSnapIn;

[CompanyInfo(CompanyName = "FooBar", CompanyUrl = "www.FooBar.com")]
public class CsharpModule : IAppFunctionality
{
    public void DoIt()
    {
        Console.WriteLine("You have just used the C# snap-in!");
    }
}
```

**Notez que j'ai choisi d'utiliser une implémentation d'interface explicite** (voir [[Chapitre 8#Implémentation explicite d'interfaces|Chapitre 8]]) **pour la prise en charge de l'interface `IAppFunctionality`**. Cela n'est pas obligatoire ; toutefois, **l'idée est que seule la partie du système qui a besoin d'interagir directement avec ce type d'interface est l'application hôte. En implémentant explicitement cette interface, la méthode `DoIt()` n'est pas directement exposée par le type `CSharpModule`.

## Création du composant logiciel enfichable Visual Basic

Passons au projet *VBSnapIn* : supprimez le fichier *Class1.vb* et ajoutez un nouveau fichier nommé *VBSnapIn.vb*. Le code est (encore une fois) volontairement simple.

```vb
Imports CommonSnappableTypes

<CompanyInfo(CompanyName:="Chucky's Software", CompanyUrl:="www.ChuckySoft.com")>
Public Class VBSnapIn
  Implements IAppFunctionality
 
  Public Sub DoIt() Implements CommonSnappableTypes.IAppFunctionality.DoIt
    Console.WriteLine("You have just used the VB snap in!")
  End Sub
End Class
```

Notez que l'application d'attributs dans la syntaxe de Visual Basic requiert des chevrons (`< >`) plutôt que des crochets (`[ ]`). Notez également que le mot-clé `Implements` est utilisé pour implémenter des types d'interface sur une classe ou une structure donnée.

## Ajout du code pour `ExtendableApp`

==Le dernier projet à mettre à jour est l'application console C#== (*MyExtendableApp*). **Après avoir ajouté l'application console** *MyExtendableApp* **à la solution et l'avoir définie comme projet de démarrage, ajoutez une référence au projet** *CommonSnappableTypes*, **mais pas aux projets** *CSharpSnapIn.dll* **ni** *VBSnapIn.dll*.

Commencez par remplacer les instructions `using` en haut de la classe *Program.cs* par le code suivant :

```cs
using System.Reflection;
using CommonSnappableTypes;
```

**La méthode `LoadExternalModule()` effectue les tâches suivantes :**

- Charge dynamiquement l’assembly sélectionné en mémoire
- Détermine si l’assembly contient des types implémentant l’interface `IAppFunctionality`
- Crée le type à l’aide de la liaison tardive

***==Si un type implémentant `IAppFunctionality` est trouvé, la méthode `DoIt()` est appelée puis transmise à la méthode `DisplayCompanyData()` pour afficher des informations supplémentaires provenant du type reflété.==***

```cs
static void LoadExternalModule(string assemblyName)
{
    Assembly theSnapInAsm = null;
    try
    {
        // Chargement dynamique de l'assembly sélectionné
        theSnapInAsm = Assembly.LoadFrom(assemblyName);
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.Message);
    }
    var theClassType = theSnapInAsm
        .GetTypes()
        .Where(t => t.IsClass && (t.GetInterface("IAppFunctionality") != null))
        .ToList();

    // Différence par rapport au livre pour la lisibilité et la performance.
    if (theClassType.Count == 0)
    {
        Console.WriteLine("Nothing implements IAppFunctionality");
    }

    // Maintenant, crée l'objet et appelle la méthode DoIt()
    foreach (Type t in theClassType)
    {
        // Utilisation de la liaison tardive pour créér le type
        IAppFunctionality itfApp = (IAppFunctionality)
            theSnapInAsm.CreateInstance(t.FullName, true);
        itfApp?.DoIt();

        // Affiche les company info.
        DisplayCompanyData(t);
    }
}
```

**La dernière étape consiste à afficher les métadonnées fournies par l'attribut `[CompanyInfo]`**. Créez la méthode `DisplayCompanyData()` comme suit. Notez que cette méthode prend un seul paramètre de type `System.Type`.

```cs
static void DisplayCompanyData(Type t)
{
    // Récupère [CompanyInfo] data
	// Ici, utilise la version générique pour éviter le boxing 
	// causé par l'ancienne API de réflexion
    var compInfo = t.GetCustomAttributes<CompanyInfoAttribute>(false);

    // Affiche les données
    foreach (CompanyInfoAttribute c in compInfo)
    {
        Console.WriteLine(
            $"More info about {c.CompanyName} can be found at {c.CompanyUrl}"
        );
    }
}
```

Enfin, mettez à jour les instructions de niveau supérieur comme suit :

```cs
Console.Title = "Welcome to MyTypeViewer";
Console.WriteLine("***** Welcome to MyTypeViewer *****\n");
string typeName = "";
do
{
    Console.WriteLine("Enter a snapin to load");
    Console.Write("or enter Q to quit: ");

    // Récupère le nom du type
    typeName = Console.ReadLine();

    // Est-ce que l'utilisateur veut quitter le programme ?
    if (typeName.Equals("Q", StringComparison.OrdinalIgnoreCase))
    {
        break;
    }

    // Essaye d'affiche le type.
    try
    {
        LoadExternalModule(typeName);
    }
    catch (Exception ex)
    {
        Console.WriteLine("Sorry, can't find snapin");
    }
} while (true);
```

**L'exemple d'application est maintenant terminé. Pour l'exécuter, saisissez `VBSnapIn.dll` ou `CSharpSnapIn.dll` pour observer le programme en action**. **==Notez que si C# est sensible à la casse, la casse n'a pas d'importance lors de l'utilisation de la réflexion==**. *CSharpSnapIn.dll* et *csharpsnapin.dll* fonctionnent indifféremment.

Ceci conclut notre présentation de la liaison tardive. Nous allons maintenant explorer la dynamique en C#.

>[!tip] **le paradoxe** de cet exercice
Voici l'explication de ce mystère architectural :
>
>### Oui, c'est de la liaison tardive (Late Binding)... pour l'implémentation
>
>L'application `MyExtendableApp` ne connaît pas et **n'a pas de référence vers `CsharpSnapIn.dll`**.
>
>- L'application découvre cette DLL sur le disque au moment de l'exécution (Runtime).
>- Elle instancie la classe `CsharpModule` par son nom sous forme de chaîne de caractères (`t.FullName`).
>- C'est de la liaison tardive pure pour la **création** de l'objet.
>
>### Mais c'est de la liaison précoce (Early Binding)... pour l'interface !
>
>Le piège est ici : votre projet `MyExtendableApp` **a une référence** vers le projet commun nommé `CommonSnappableTypes.dll` (qui contient l'interface `IAppFunctionality`).
>
>C'est pour cela que vous pouvez écrire :
>
>```cs
>IAppFunctionality itfApp = (IAppFunctionality)theSnapInAsm.CreateInstance(...);
>```
>
>### Pourquoi l'auteur fait-il ce mélange ? (L'architecture hybride)
>
>Faire du "100 % Late Binding" (sans aucune interface commune) obligerait à appeler la méthode `DoIt()` via la réflexion avec `MethodInfo.Invoke()`. C'est lourd, lent et propice aux erreurs de frappe.
>
>Dans le monde professionnel, on utilise cette approche hybride appelée **Architecture par Plugins (ou Extensions)** :
>
>1. **Le Contrat (Early)** : L'Hôte et le Plugin connaissent tous les deux une DLL tierce très légère contenant uniquement des interfaces (`CommonSnappableTypes`).
>2. **L'Implémentation (Late)** : L'Hôte ne connaît absolument pas les détails des plugins (`CsharpSnapIn`, `VBSnapIn`). Il les charge dynamiquement.
>
>C'est exactement comme cela que fonctionnent les extensions de **Neovim** ou de **VS Code** : l'éditeur fournit une API fixe (l'interface), et les développeurs créent des plugins que l'éditeur charge sans les connaître à l'avance.

# Le rôle du mot-clé `dynamic` en C#

>[!tip] Permet de fonctionner comme le Python ou le JavaScript

**Au** [[Chapitre 3#Comprendre les variables locales implicitement typées|Chapitre 3]], **vous avez découvert le mot-clé `var`, qui permet de définir des variables locales de telle sorte que leur type de données sous-jacent soit déterminé à la compilation, en fonction de l'affectation initiale** (rappelons que on parle alors de typage implicite). ==Une fois cette affectation initiale effectuée, vous disposez d'une variable fortement typée, et toute tentative d'affectation d'une valeur incompatible entraînera une erreur de compilation.==

Pour commencer votre exploration du mot-clé `dynamic` en C#, créez un nouveau projet d'application console nommé *DynamicKeyword*. Ajoutez ensuite la méthode suivante dans votre fichier *Program.cs* et vérifiez que la dernière instruction dé-commentée provoque bien une erreur de compilation :

```cs
static void ImplicitlyTypedVariable()
{
    // a est de type List<int>
    var a = new List<int> {90};
    // Ceci serait une erreur de compilation
    // a = "Hello";
}
```

**Comme vous l'avez vu au** [[Chapitre 13#Typage implicite des variables locales|Chapitre 13]], **le typage implicite est utile avec LINQ, car de nombreuses requêtes LINQ renvoient des énumérations de classes anonymes (via des projections) que vous ne pouvez pas déclarer directement dans votre code C#**. Cependant, ==même dans ces cas, la variable typée implicitement est, en réalité, fortement typée==. Dans l'exemple précédent, la variable a est fortement typée comme étant `List<int>`.

Par ailleurs, comme vous l'avez appris au [[Chapitre 6#Comprendre la classe parente ultime `System.Object`|Chapitre 6]], **`System.Object` est la classe parente la plus élevée du .NET Core Framework et peut représenter absolument n'importe quel type**. **==Si vous déclarez une variable de type `object`, vous disposez d'une donnée fortement typée==**; *==cependant, ce à quoi elle fait référence en mémoire peut différer selon l'affectation de la référence==*. **Pour accéder aux membres pointés par la référence d'objet en mémoire, vous devez effectuer une conversion explicite.**

Supposons que vous ayez une classe simple nommée `Person` qui définit deux propriétés automatiques (`FirstName` et `LastName`) encapsulant toutes deux un `string`:

```cs
namespace DynamicKeyword;

class Person
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
}
```

Observez maintenant le code suivant :

```cs
static void UseObjectVariable()
{
    // Crée une nouvelle instance de la classe Person
    // et l'assigne à une variable de type System.Object
    object o = new Person() { FirstName = "Mike", LastName = "Larson" };

    // Il faut convertir l'objet en Person pour accéder
    // aux propriétés de Person.

    Console.WriteLine($"Person's first name is {((Person)o).FirstName}");
}
```

**Revenons-en au mot-clé `dynamic`. De manière générale, on peut le considérer comme une
forme spécialisée de `System.Object`, permettant d'assigner n'importe quelle valeur à un type de données dynamique**. ==À première vue, cela peut paraître extrêmement déroutant, car il semble exister trois manières de définir des données dont le type sous-jacent n'est pas directement indiqué dans le code==. Par exemple, cette méthode :

```cs
static void PrintThreeStrings()
{
    var s1 = "Greetings";
    object s2 = "From";
    dynamic s3 = "Minneapolis";

    Console.WriteLine($"{nameof(s1)} is of type: {s1.GetType()}");
    Console.WriteLine($"{nameof(s2)} is of type: {s2.GetType()}");
    Console.WriteLine($"{nameof(s3)} is of type: {s3.GetType()}");
}
```

afficherait le résultat suivant si cette fonction était appelée depuis vos instructions de niveau supérieur :

```
s1 is of type: System.String
s2 is of type: System.String
s3 is of type: System.String
```

==Qu'est-ce qui distingue fondamentalement une variable dynamique d'une variable déclarée implicitement ou via une référence d'objet système ?== *Elle n'est pas *fortement typée*. Autrement dit, **les données dynamiques ne sont pas *statiquement typées***. ***==Pour le compilateur C#, une donnée déclarée avec le mot-clé `dynamic` peut se voir attribuer n'importe quelle valeur initiale et être réaffectée à n'importe quelle nouvelle valeur==*** (potentiellement sans rapport) ***==durant sa durée de vie==***. Prenons l'exemple de la méthode suivante et de son résultat :

```cs
static void ChangeDynamicDataType()
{
    // Déclare seul un point de donnée dynamic nommé t
    dynamic t = "Hello!";
    Console.WriteLine($"t is of type: {t.GetType()}");

    t = false;
    Console.WriteLine($"t is of type: {t.GetType()}");

    t = new List<int>();
    Console.WriteLine($"t is of type: {t.GetType()}");
}
```

```
t is of type: System.String
t is of type: System.Boolean
t is of type: System.Collections.Generic.List`1[System.Int32]
```

**À ce stade de votre étude, sachez que le code précédent se compilerait et s'exécuterait
de manière identique si vous déclariez la variable `t` comme un `System.Object`**. **==Cependant, comme vous le verrez bientôt, le mot-clé `dynamic` offre de nombreuses fonctionnalités supplémentaires.==**

## Appel des membres sur des données déclarées dynamiquement

**Étant donné qu'une variable dynamique peut prendre n'importe quel type à la volée** (tout comme une variable de type `System.Object`), vous vous demandez peut-être comment appeler ses membres (propriétés, méthodes, indexeurs, enregistrement auprès d'événements, etc.). **Syntactiquement parlant, rien ne change. Il suffit d' appliquer l'opérateur point à la variable de données dynamique, de spécifier un membre public et de fournir les arguments (si nécessaires).**

***==Cependant (et c'est un point crucial), la validité des membres que vous spécifiez n'est pas vérifiée par le compilateur !==*** N'oubliez pas que, **contrairement à une variable définie comme `System.Object`, les données dynamiques ne sont pas statiquement typées**. **==Ce n'est qu'à l'exécution que vous saurez si les données dynamiques que vous avez appelées prennent en charge un membre spécifié, si vous avez passé les bons paramètres, si vous avez correctement orthographié le membre, etc==**. Ainsi, aussi étrange que cela puisse paraître, la méthode suivante compile parfaitement :

```cs
static void InvokeMembersOnDynamicData()
{
    dynamic textData1 = "Hello";
    Console.WriteLine(textData1.ToUpper());

    // On s'attendrait à des erreurs de compilation ici !
    // Mais la compilation se déroule sans problème.
    Console.WriteLine(textData1.toupper());
    Console.WriteLine(textData1.Foo(10, "ee", DateTime.Now));
}
```

==Remarquez que le deuxième appel à `WriteLine()` tente d'appeler une méthode nommée `toupper()` sur le point de données dynamiques== (notez la casse incorrecte : il devrait s'agir de `ToUpper()`). **Comme vous pouvez le constater, `textData1` est de type `string`, et par conséquent, vous savez qu'il ne possède pas de méthode de ce nom en minuscules**. *==De plus, `string` ne possède certainement pas de méthode nommée `Foo()` qui accepte des objets `int`, `string` et `DateTime` !==*

**Néanmoins, le compilateur C# est satisfait. Cependant, si vous appelez cette méthode depuis `Main()`, vous obtiendrez des erreurs d'exécution comme la suivante :**

```
Unhandled Exception: Microsoft.CSharp.RuntimeBinder.RuntimeBinderException:
'string' does not contain a definition for 'toupper'
```
 
 **Une autre différence notable entre l'appel de membres sur des données dynamiques et des données fortement typées, est que lorsque vous appliquez l'opérateur point à une donnée dynamique, l'IntelliSense** (*roslyn*) **ne s'affiche pas**. L'IDE vous permet de saisir n'importe quel nom de membre. ***==Cele signifie que vous devez être extrêmement vigilant lorsque vous écrivez du code C# sur de tels points de données. Toute faute d'orthographe ou erreur de majuscule dans le nom d'un membre provoquera une erreur d'exécution, plus précisément une instance de la classe `RuntimeBinderException`.

**e'exception `RuntimeBinderException` représente l'erreur qui sera levée si vous tentez d'appeler un membre sur un type de données dynamique qui n'existe pas** (comme dans le cas des méthodes `toupper()` et `Foo()`). **La ​​même erreur sera levée si vous spécifiez des données de paramètre incorrectes pour un membre qui existe.**

**==Étant donné la grande volatilité des données dynamiques, lorsque vous appelez des membres d'une variable déclarée avec le mot-clé `dynamic` de C#, vous pouvez encapsuler les appels dans un bloc `try/catch` approprié et gérer l'erreur de manière propre**==, comme ceci :

```cs
static void InvokeMembersOnDynamicData()
{
    dynamic textData1 = "Hello";

    try
    {
        Console.WriteLine(textData1.ToUpper());
        Console.WriteLine(textData1.toupper());
        Console.WriteLine(textData1.Foo(10, "ee", DateTime.Now));
    }
    catch (Microsoft.CSharp.RuntimeBinder.RuntimeBinderException ex)
    {
        Console.WriteLine(ex.Message);
    }
}
```

**==Si vous appelez à nouveau cette méthode, vous constaterez que l'appel à `ToUpper()`==** (notez le T et le U majuscules) **==fonctionne correctement; cependant, vous verrez alors s'afficher des données d'erreur dans la console.==**

```
HELLO
'string' does not contain a definition for 'toupper'
```

Bien sûr, le processus consistant à encapsuler tous les appels de méthodes dynamiques dans un bloc try/catch est plutôt fastidieux. Si vous soignez votre orthographe et le passage de vos paramètres, cela n'est pas nécessaire. Cependant, la gestion des exceptions est pratique lorsque vous ne savez pas à l'avance si un membre sera présent sur le type cible.

## Portée du mot-clé `dynamic`

Rappelons que *==les données implicitement typées==* (déclarées avec le mot-clé `var`) *==ne sont possibles que pour les variables locales dans une portée membre==*. *==Le mot-clé `var` ne peut jamais être utilisé comme valeur de retour, paramètre ou membre d'une classe/structure==*. ***==Ce n'est pas le cas du mot-clé `dynamic`==***. Prenons l'exemple de la définition de classe suivante :

```cs
 namespace DynamicKeyword;

class VeryDynamicClass
{
    // Un champ dynamique
    private static dynamic _myDynamicField;

    // Une propriété dynamique
    public dynamic DynamicProperty { get; set; }

    // Un type de retour dynamique et un paramètre de type dynamique
    public dynamic DynamicMethod(dynamic dynamicParam) 
    {
      // Une variable locale dymamique
      dynamic dynamicLocalVar = "Local variable";

      int myInt = 10;

      if (dynamicParam is int)
      {
        return dynamicLocalVar;
      }
      else
      {
        return myInt;
      }
    }
}
```

**Vous pouvez désormais appeler les membres publics comme prévu ; cependant, comme vous travaillez avec des méthodes et des propriétés dynamiques, vous ne pouvez pas être absolument certain du type de données !** ==Bien que la définition de `VeryDynamicClass` puisse ne pas être utile dans une application réelle, elle illustre le champ d’application de ce mot-clé C#.==

## Limites du mot-clé `dynamic`

Bien que de nombreux éléments puissent être définis à l'aide du mot-clé `dynamic`, *==son utilisation présente certaines limitations==*. ==Ses limitations ne sont pas rédhibitoires, mais sachez qu'un élément de données dynamique ne peut pas utiliser d'expressions lambda ni de méthodes anonymes C# lors de l'appel d'une méthode==. Par exemple, le code suivant générera toujours des erreurs, même si la méthode cible accepte un paramètre délégué de type chaîne de caractères et ne renvoie aucune valeur :

```cs
dynamic a = GetDynamicObject();

// Erreur ! Les méthodes sur des données dynamiques ne peuvent pas utiliser de lambdas !
a.Method(arg => Console.WriteLine(arg));
```

**==Pour contourner cette restriction, vous devrez interagir directement avec le délégué sous-jacent, en utilisant les techniques décrites au [[Chapitre 12]]==**. *==Une autre limitation est qu'un point de données dynamique ne peut pas interpréter les méthodes d'extension==* (voir [[Chapitre 11#Comprendre les méthodes d'extension|Chapitre 11]]). **Malheureusement, cela inclut également les méthodes d'extension provenant des API LINQ**. Par conséquent, *==une variable déclarée avec le mot-clé `dynamic` a une utilité limitée dans LINQ to Objects et les autres technologies LINQ.==*

```cs
dynamic a = GetDynamicObject();

// Erreur ! Les données dynamiques ne trouvent pas la méthode d'extension Select() !
var data = from d in a select d;
```

## Utilisation pratique du mot-clé `dynamic`

**Étant donné que les données dynamiques ne sont pas fortement typées, ne sont pas vérifiées à la compilation, ne peuvent pas déclencher IntelliSense et ne peuvent pas être la cible d'une requête LINQ, vous avez tout à fait raison de supposer que l'utilisation du mot-clé `dynamic` sans raison particulière constitue une mauvaise pratique de programmation.**

**==Cependant, dans certains cas, le mot-clé `dynamic` peut réduire considérablement la quantité de code que vous devez écrire manuellement==**. Plus précisément, **si vous développez une application .NET qui utilise intensivement la liaison tardive** (par réflexion), **le mot-clé `dynamic` peut vous faire gagner du temps de saisie**. ==De même, si vous développez une application .NET qui doit communiquer avec des bibliothèques COM héritées== (telles que les produits Microsoft Office), ==vous pouvez simplifier considérablement votre code grâce au mot-clé `dynamic`==. **Enfin, les applications web développées avec ASP.NET Core utilisent fréquemment le type `ViewBag`, qui peut également être accessible de manière simplifiée grâce au mot-clé `dynamic`.**

>[!note]
>L'interaction COM est strictement un paradigme Windows et supprime les capacités multiplateformes de votre application.
>
>L'utilisation du `ViewBag` est aujourd'hui considérée comme une **mauvaise pratique** (legacy). La communauté et Microsoft recommandent exclusivement les **ViewModels fortement typés** ou `ViewData` pour des raisons de performance et de sécurité au moment de la compilation. De plus, avec l'essor des API minimales (Minimal APIs) et de Blazor, l'utilisation de `dynamic` dans le web .NET a presque disparu.

>[!warning] Le plus grand changement depuis la sortie du livre concerne les optimisations de publication (.NET 8/9/10) :
>
>- Le mot-clé `dynamic` a besoin du compilateur complet embarqué dans l'application (DLR) pour résoudre les noms de méthodes au moment de l'exécution.
>- Par conséquent, **`dynamic` est totalement incompatible avec la compilation Native AOT** (Ahead-of-Time). Si votre fichier `.csproj` contient `<PublishAot>true</PublishAot>`, le code utilisant `dynamic` lèvera des exceptions ou refusera de compiler.


**Comme pour tout « raccourci », il est important d'en peser le pour et le contre**. ==L'utilisation du mot-clé `dynamic` représente un compromis entre la concision du code et la sécurité des types==. ==Bien que C# soit un langage fortement typé par nature, vous pouvez activer (ou désactiver) les comportements dynamiques pour chaque appel de fonction==. **==N'oubliez jamais que vous n'avez jamais besoin d'utiliser le mot-clé `dynamic`==**. ==Vous pouvez toujours obtenir le même résultat en écrivant du code alternatif à la main (et généralement beaucoup plus long).==

# Le rôle du Dynamic Language Runtime

>[!danger]- Cette section n'est plus vraiment d'actualité (Gemini)
>
Voici pourquoi le DLR est devenu un concept secondaire aujourd'hui :
>
> ### La fin de l'ère des "Langages Dynamiques" sur .NET
>
>Le DLR a été créé à la fin des années 2000 dans un but précis : permettre à des langages dynamiques comme **IronPython** ou **IronRuby** de tourner efficacement sur le runtime .NET, et permettre à C# d'interagir avec eux [1].
>
>- **Le constat actuel :** Ces projets sont aujourd'hui quasi abandonnés ou très marginaux. Plus personne ne développe de nouvelles applications en combinant C# et IronPython.
>
>### Le DLR est l'ennemi de la stratégie actuelle de Microsoft
>
>Depuis .NET 8 et l'arrivée à maturité de .NET 10, la priorité absolue de Microsoft est la **performance brute**, le **Trimming** (l'élagage du code) et le **Native AOT** (la compilation native sans machine virtuelle).
>
>- **Le problème du DLR :** Le DLR génère du code de manière dynamique dans la mémoire RAM pendant que l'application tourne (via des arbres d'expression).
>- **La conséquence :** Cela nécessite d'embarquer un mini-compilateur dans votre application, ce qui rend le binaire très lourd et **totalement incompatible** avec le déploiement Native AOT.
>
>### Par quoi a-t-il été remplacé ?
>
>Là où le DLR servait à simplifier la réflexion ou l'évaluation de code à la volée, le C# moderne utilise désormais des approches basées sur le principe de la **compilation préalable** :
>
>- **Les Source Generators (C# 9+) :** Le code dynamique est généré _pendant_ que vous écrivez (au moment du `dotnet build`), ce qui permet d'avoir du code fortement typé, ultra-rapide et vérifiable par IntelliSense.
>- **Les Expressions Lambda compilées :** Pour les frameworks qui ont encore besoin de dynamisme (comme Entity Framework), on compile des `Expression<Func<T>>` directement en code IL au démarrage, sans passer par la surcouche lourde du DLR.

Maintenant que vous comprenez mieux ce que sont les « données dynamiques », voyons comment elles sont traitées. **Depuis la sortie de .NET 4.0, le Common Language Runtime (CLR) a été complété par un environnement d’exécution associé appelé Dynamic Language Runtime**. Le concept d’« environnement d’exécution dynamique » n’est certainement pas nouveau. En effet, ==de nombreux langages de programmation tels que JavaScript, LISP, Ruby et Python l’utilisent depuis des années==. **En résumé, un environnement d’exécution dynamique permet à un langage dynamique de découvrir les types entièrement à l’exécution, sans vérification à la compilation.**

>[!note]
>Bien qu'une grande partie du DLR ait été portée vers .NET Core (à partir de la version 3.0), la parité des fonctionnalités entre le DLR dans .NET 6 et .NET 4.8 n'a pas été atteinte.

Si vous avez une expérience des langages fortement typés (y compris C#, sans types dynamiques), la notion d'un tel environnement d'exécution peut sembler peu attrayante. Après tout, ***==on préfère généralement recevoir des erreurs à la compilation plutôt qu'à l'exécution, autant que possible==***. Néanmoins, **les langages/environnements d'exécution dynamiques offrent des fonctionnalités intéressantes**, notamment :

-  Une base de code extrêmement flexible. Vous pouvez refactoriser le code sans avoir à modifier de nombreux types de données.
- Un moyen simple d'interagir avec divers types d'objets créés sur différentes plateformes et dans différents langages de programmation.
- Un moyen d'ajouter ou de supprimer des membres d'un type, en mémoire, à l'exécution.

**L'un des rôles du DLR est de permettre à divers langages dynamiques de s'exécuter avec l'environnement d'exécution .NET et de leur offrir une possibilité d'interagir avec d'autres codes .NET.** Ces langages évoluent dans un univers dynamique, où le type est découvert uniquement à l'exécution. Et pourtant, ces langages ont accès à la richesse des bibliothèques de classes de base .NET. Mieux encore, leurs bases de code peuvent interagir avec C# (et vice versa), grâce à l'inclusion du mot-clé `dynamic`.

>[!note]
>Ce chapitre n'abordera pas l'intégration de DLR avec les langages dynamiques.

## Rôle des arbres d'expression

**Le DLR utilise des arbres d'expression pour traduire de manière neutre la signification d'un appel dynamique**. Par exemple, prenons le code C# suivant :

```cs
dynamic d = GetSomeData();
d.SuperMethod(12);
```

**Dans cet exemple, le DLR construit automatiquement un arbre d'expression qui indique, en substance** : « **==Appelle la méthode nommée `SuperMethod` sur l'objet `d`, en lui passant le nombre `12` comme argument.==** » ==Ces informations (formellement appelées *charge utile*) sont ensuite transmises au module d'exécution approprié, qui peut être le module dynamique C#, ou même (comme expliqué ci-après) des objets COM hérités.==

**À partir de là, la requête est convertie en la structure d'appel requise pour l'objet cible**. **==L'avantage de ces arbres d'expression==** (outre le fait qu'il n'est pas nécessaire de les créer manuellement) **==est qu'ils permettent d'écrire une instruction C# fixe sans se soucier de la cible sous-jacente.==**

## Recherche dynamique à l'exécution des arbres d'expressions

**Comme expliqué précédemment, le DLR transmet les arbres d'expressions à un objet cible** ; ==toutefois, cette distribution est influencée par plusieurs facteurs==. Si le type de données dynamiques pointe en mémoire vers un objet COM, l'arbre d'expressions est envoyé à une interface COM de bas niveau nommée `IDispatch`. Comme vous le savez peut-être, cette interface était la méthode employée par COM pour intégrer son propre ensemble de services dynamiques. Les objets COM peuvent cependant être utilisés dans une application .NET sans recourir au DLR ni au mot-clé `dynamic` C#. Cependant, cette approche (comme vous le verrez) tend à complexifier considérablement le code C#.

**Si les données dynamiques ne pointent pas vers un objet COM, l'arbre d'expressions peut être transmis à un objet implémentant l'interface `IDynamicObject`**. ==Cette interface est utilisée en interne pour permettre à un langage, tel qu'IronRuby, de convertir un arbre d'expressions DLR en spécificités Ruby==. Enfin, **si les données dynamiques pointent vers un objet qui n'est pas un objet COM et qui n'implémente pas `IDynamicObject`, il s'agit d'un objet .NET standard**. **==Dans ce cas, l'arbre d'expression est transmis au moteur de liaison C# pour traitement==**. **Le processus de conversion de l'arbre d'expression en spécificités .NET fait appel aux services de réflexion.**

**Une fois l'arbre d'expression traité par un moteur de liaison donné, les données dynamiques sont résolues en leur type de données en mémoire réel, après quoi la méthode appropriée est appelée avec les paramètres nécessaires**. Voyons maintenant quelques cas pratiques d'utilisation du DLR, en commençant par la simplification des appels .NET à liaison tardive.

# Simplification des appels tardifs grâce aux types dynamiques

>[!warning] Le cas de la Réflexion / Liaison tardive : Remplacé par le C# moderne (Gemini)
>
>- **Ce que dit le livre :** `dynamic` réduit le code de liaison tardive (évite les `MethodInfo.Invoke`).
>- **La réalité moderne :** C'est vrai, mais cela a un coût massif en performance (le DLR - _Dynamic Language Runtime_ - alloue beaucoup de mémoire). Dans le C# moderne, pour éviter la réflexion verbeuse, on utilise :
>    1. Des **interfaces partagées** (comme votre exercice de Plug-in), ce qui rend `dynamic` inutile.
>    2. L'attribut **`[UnsafeAccessor]`** (introduit en .NET 8), qui permet d'accéder à des membres privés à vitesse native sans l'infrastructure lourde de `dynamic`]

==L'utilisation du mot-clé `dynamic` peut s'avérer utile lors de l'utilisation de services de réflexion, en particulier lors d'appels de méthodes à liaison tardive==. Plus tôt dans ce chapitre, **==vous avez vu quelques exemples d'utilisation de ce type d'appel de méthode, notamment lors de la création d'une application extensible==**. **Vous avez alors appris à utiliser la méthode `Activator.CreateInstance()` pour créer un objet dont vous ne connaissez aucune information à la compilation** (hormis son nom d'affichage). Vous pouvez ensuite utiliser les types de l'espace de noms `System.Reflection` pour appeler des membres via la liaison tardive. Rappelez-vous l'exemple précédent :

```cs
static void CreateUsingLateBinding(Assembly asm)
{
    try
    {
        // Récupère les métadonnées pour le type MiniVan
        Type miniVan = asm.GetType("CarLibrary.MiniVan");

        // Crée une instance de MiniVan à la volée
        object obj = Activator.CreateInstance(miniVan);
        Console.WriteLine($"Created a {obj} using late binding!");

        // Récupère l'information pour TurboBoost.
        MethodInfo mi = miniVan.GetMethod("TurboBoost");

        // Appelle la méthode ('null' pour aucun pramètre)
        mi.Invoke(obj, null);
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.Message);
    }
}
```

==Bien que ce code fonctionne comme prévu, vous conviendrez sans doute qu'il est un peu lourd. Vous devez utiliser manuellement la classe `MethodInfo`, interroger manuellement les métadonnées, etc==. Voici une version de cette même méthode, utilisant cette fois le mot-clé `dynamic` de C# et le DLR :

```cs
static void CreateUsingLateBinding(Assembly asm)
{
    try
    {
        // Récupère les métadonnées pour le type MiniVan
        Type miniVan = asm.GetType("CarLibrary.MiniVan");

        // Crée une instance de MiniVan à la volée et appelle la méthode!
        dynamic obj = Activator.CreateInstance(miniVan);
        obj.TuboBooost();
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.Message);
    }
}
```

En déclarant la variable obj à l'aide du mot-clé `dynamic`, le gros du travail de réflexion est effectué pour vous, grâce au DRL.

## Utilisation du mot-clé `dynamic` pour passer des arguments
 
 ==L'utilité du DLR devient encore plus évidente lorsque vous devez effectuer des appels tardifs sur des méthodes prenant des paramètres==. **Lorsque vous utilisez des appels de réflexion « longs », les arguments doivent être packagé sous forme de tableau d'objets, qui sont passés à la méthode `Invoke()` de `MethodInfo`.**

Pour illustrer cela avec un nouvel exemple, commencez par créer un nouveau projet d'application console C# nommé *LateBindingWithDynamic*. Ensuite, ajoutez un projet de bibliothèque de classes nommé *MathLibrary*. Renommez le fichier initial `Class1.cs` du projet *MathLibrary* en *SimpleMath.cs*, et implémentez la classe comme suit :

```cs
namespace MathLibrary;

public class SimpleMath
{
    public int Add(int x, int y)
    {
        return x + y;
    }
}
```

Mettez à jour le fichier *MathLibrary.csproj* avec le code suivant (pour copier l'assembly compilé vers le dossier cible `LateBindingWithDymamic`) 

```xml
  <Target Name="PostBuild" AfterTargets="PostBuildEvent">
    <Copy SourceFiles="$(TargetPath)" DestinationFolder="$(ProjectDir)../LateBindingWithDynamic/" />
  </Target>
```

De retour dans le projet *LateBindingWithDynamic* et le fichier *Program.cs*, mettez à jour les instructions using comme suit :

```cs
using System.Reflection;
using Microsoft.CSharp.RuntimeBinder;
```

Ensuite, ajoutez la méthode suivante au fichier *Program.cs*, qui appelle la méthode `Add()` en utilisant des appels d'API de réflexion classiques :

```cs
static void AddWithReflection()
{
    Assembly asm = Assembly.LoadFrom("MathLibrary.dll");
    try
    {
        // Récupère les métadonnée du type SimpleMath
        Type math = asm.GetType("MathLibrary.SimpleMath");

        // Crée un SimpleMath à la volée
        object obj = Activator.CreateInstance(math);

        // Récupère les informations pour Add
        MethodInfo mi = math.GetMethod("Add");

        // Appelle la méthode (avec des paramètres)
        object[] args = { 10, 70 };
        Console.WriteLine($"Result is {mi.Invoke(obj, args)}");
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.Message);
    }
}
```

Considérons maintenant la simplification de la logique précédente grâce au mot-clé `dynamic`, via la nouvelle méthode suivante :

```cs
static void AddWithDynamic()
{
    Assembly asm = Assembly.LoadFrom("MathLibrary.dll");
    try
    {
        // Récupère les métadonnée dy type SimpleMath
        Type math = asm.GetType("MathLibrary.SimpleMath");

        // Crée un objet SimpleMath à la volée
        dynamic obj = Activator.CreateInstance(math);

        // Notez la facilité avec laquelle on appelle Add().
        Console.WriteLine($"Result is: {obj.Add(10, 70)}");
    }
    catch (RuntimeBinderException ex)
    {
        Console.WriteLine(ex.Message);
    }
}
```

Pas mal du tout ! Si vous appelez les deux méthodes, vous obtiendrez un résultat identique. ~~Cependant, l'utilisation du mot-clé `dynamic` vous a permis de gagner un temps précieux~~. Grâce aux données définies dynamiquement, vous n'avez plus besoin de regrouper manuellement les arguments sous forme de tableau d'objets, d'interroger les métadonnées de l'assembly ni de définir d'autres détails de ce type. Si vous développez une application qui utilise intensivement le chargement dynamique/la liaison tardive, je suis certain que vous comprenez l'importance de ces gains de code au fil du temps.

# Résumé du chapitre

**==La réflexion est un aspect intéressant d'un environnement orienté objet robuste==**. **Dans l'univers .NET, les clés des services de réflexion s'articulent autour de la classe `System.Type` et de l'espace de noms `System.Reflection`**. Comme vous l'avez vu, **==la réflexion est le processus qui consiste à examiner un type en détail lors de l'exécution afin de comprendre qui, quoi, où, quand, pourquoi et comment d'un élément donné.==**

**La liaison tardive est le processus de création d'une instance d'un type et d'appel de ses membres sans connaissance préalable de leurs noms spécifiques**. ==La liaison tardive résulte souvent directement du *chargement dynamique*, qui permet de charger un assembly .NET en mémoire par programmation==. Comme illustré dans l'exemple d'application extensible de ce chapitre, **il s'agit d'une technique puissante utilisée aussi bien par les développeurs que par les utilisateurs d'outils.**

***==Ce chapitre a également examiné le rôle de la programmation par attributs. Lorsque vous ajoutez des attributs à vos types, vous augmentez les métadonnées de l'assembly sous-jacent. ==***

**Le mot-clé `dynamic` permet de définir des données dont l'identité n'est connue qu'à l'exécution**. ==Lors du traitement par le Dynamic Language Runtime (DLR), l'« arbre d'expression » créé automatiquement est transmis au module de liaison du langage dynamique approprié, où la charge utile est décompressée et envoyée au membre d'objet correspondant.==

~~L'utilisation des données dynamiques et du DLR simplifie considérablement les tâches complexes de programmation C#, notamment l'intégration de bibliothèques COM dans vos applications .NET.~~

***==Bien que ces fonctionnalités puissent simplifier votre code, n'oubliez pas que les données dynamiques rendent votre code C# beaucoup moins sûr au niveau des types et plus vulnérable aux erreurs d'exécution==***. **Pesez soigneusement le pour et le contre de l'utilisation des données dynamiques dans vos projets C# et effectuez des tests en conséquence !**

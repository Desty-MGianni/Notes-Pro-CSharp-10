---
title: "Chapitre 1: Introduction à C# et .NET 6"
publish: true
---

# <big><big><big><b><font color =green>Introduction à C# et .NET 6</font></b></big></big></big>

La plateforme .NET de Microsoft et le langage de programmation C# ont été **officiellement lancés vers 2002** et sont rapidement devenus incontournables dans le développement logiciel moderne. La plateforme .NET permet à un grand nombre de langages de programmation (dont C#, VB.NET et F#) d'interagir entre eux. ==Un programme écrit en C# peut être référencé par un autre programme écrit en VB.NET==. Nous reviendrons sur cette interopérabilité plus loin dans ce chapitre.

En __2016__, Microsoft annonce `.NET Core`. Il remplace `.NET Framework` permettant l'utilisation des programmes écrit en C# d'être utilisé nativement par d'autre systèmes d'exploitation que Windows. (Développement sur MacOS par exemple).  

**Microsoft lance C# 10 et .NET 6 le 8 Novembre 2021.**

>[!Info]-
> Un programme écrit en C# peut être référencé (utilisé) par un autre programme utilisant la plateforme .NET.

Le but du livre est double:
1. Explication claire et très détaillé de la syntaxe et de la sémantique de C#.
2. Illustrer l'usage de nombreux framework de développement .NET 
    - `ADO.NET` et `Entity Framework Core` pour l'accès aux bases de données
    - `WPF` pour les interfaces utilisateurs
    - `ASP.NET Core` pour les applications web et API en REST ([Voir vidéo ici](https://youtu.be/7iHl71nt49o?si=nO_Sg4u9rKUFVj4P))

Ce premier chapitre pose les bases conceptuelle du reste du livre. Il fait une présentation générale de certains éléments lié a .NET comme:

- Les Assemblies (équivalent des module en Python)
- Le Common Intermediate Language (`CIL`)
- La compilation Juste-à-temps (`JIT`)

En plus de découvrir certains mots-clés du langage de programmation C#, vous comprendrez également la relation entre le *runtime .NET*, le Common Type System (`CTS`) et le Common Language Specification (`CLS`).

Ce chapitre propose aussi un aperçu des fonctionnalité fournie par les bibliothèques de classes de base .NET, parfois abrégé en `BCL`, mais aussi un aperçu de la nature indépendante de `C#` et de la plateforme `.NET`.

>[!tldr]- Dans le livre: 
> Beaucoup de fonctionnalité surligné dans ce chapitre (et à travers le livre) s'applique aussi pour la plateforme précédente *.NET Framework*.

# Explorons des bénéfice clés de la plateforme .NET

Le framework .NET est une plateforme logicielle permettant de créer des applications et des services web pour les systèmes d'exploitation Windows, iOS et Linux, ainsi que des applications WinForms et WPF sur les systèmes d'exploitation Windows.

pour préparer le terrain, voici un aperçu rapide de quelque fonctionnalité fournies par `.NET`:

- *Supporte plusieurs languages de programmation:* (`C#`, `F#` et `VB.NET`)
<br>
- *Un moteur d'exécution (runtime engine) commun à tous les language .NET:* Un aspect de ce moteur est un set de type bien défini que tout les langages de la plateforme comprennent.
<br>
- *L'intégration des langagues:* .NET prend en charge l'héritage inter-langage, la gestion des exceptions inter-langage et le déboggage inter-langage du code. Par exemple, vous pouvez définir une classe de base en C# et étendre ce type dans Visual Basic.
<br>
- *Une librairie de classe de base exhaustive:* Cette librairie propose des milliers de types prédéfini qui permettent de créer de nouvelles librairies, de simple application terminal, des application avec interface graphique et des sites web d'entreprise.
<br>
- *Un modèle de déploiement simplifié:* les bibliothèques .NET ne sont pas enregistrées dans le registre du système. De plus, la plateforme .NET permet à plusieurs versions du framework ainsi qu'à plusieurs applications de coexister harmonieusement sur une même machine.
<br>
- *Prise en charge étendue de la ligne de commande:* l'interface de ligne de commande .NET (`CLI`) est une chaîne d'outils multiplateforme permettant de développer et de packager des applications .NET. Des outils supplémentaires peuvent être installés (globalement ou localement) en plus des outils standard fournis avec le ==SDK .NET==.

# Comprendre le cycle de vie des versions de .NET

1. `LTS` pour Long Term Support:
    - Trois ans après la version initiale
    - Un an de support de maintenance après la version LTS suivante

2. `STS`pour Standard Term Support:
    - 6 mois après un précédent `STS` ou `LTS`

.NET 6 est sorti en Novembre 2021 en tant que `LTS`. Il à été supporté jusqu'en Novembre 2024.

pour voir la politique de support des plateforme .NET et .NET Core: https://dotnet.microsoft.com/en-us/platform/support/policy/dotnet-core

# Aperçu des blocs de construction de la plateforme .NET

Parlons des sujets clés et interdépendant:
- Le Runtime .NET
- `CTS`
- `CLS`

**Du point de vue d'un développeur, `.NET` peut être compris comme un environnement d'exécution avec une librairie de classe de base exhaustive**. La couche runtime contient un ensemble d'implémentation minimal qui sont lié spécifiquement au système d'exploitation (Windows, MacOS/iOS, Linux), à l'architecture (x86, x64, ARM) ainsi qu'aux type de base de .NET.

Un autre bloc de la plateforme `.NET` est le *Common Type System (`CTS`)*.
La Spécification `CTS` décrit complètement tous les type de données et toutes les construction de programmation prise en charge par le runtime, spécifie comment ces entités peuvent interagir entre elle, et détaille comment ils sont représenté dans le format de méta-donnée .NET. (Plus d'information sur les metadata dans ce chapitre, voir chapitre [[Chapitre 17]] pour l'explication détaillée).

==Comprenez qu'un language .NET donnée peut ne pas supporté toute les fonctionnalité défini par le `CTS`==. Le *Common Language Specification (`CLS`)* est une spécification reliée qui définit les sous-ensemble des types de donnée et des construction de programmation que tout les language de programmation .NET peuvent supporter.

-> Si on crée un programme qui n'exploite que des fonctionnalités *"CLS compliant"* , alors tous les autre language de programmation de la plateforme .NET pourront l'utiliser.

-> Au contraire, si on utilise des fonctions qui ne sont pas *CLS compliant*, alors on ne peut pas garantir que les language `.NET` pourrons interagir avec notre librairie de code `.NET`.

Plus loin dans le chapitre, on verra comment demander au compilateur si notre programme est *CLS compliant*.

## Le rôle de la librairie de classe de base

La plateforme .NET fournit également un ensemble de bibliothèques de classes de base (`BCL`) disponibles pour tous les langages de programmation .NET. Non seulement cette bibliothèque de classes de base encapsule diverses type de donnée primitives telles que :
- les threads 
- les entrées/sorties de fichiers (I/O) 
- les systèmes de rendu graphique 
- l'interaction avec divers périphériques matériels externes

Elle prend également en charge un certain nombre de services requis par la plupart des applications du monde réel.

Les bibliothèques de classes de base définissent des types qui peuvent être utilisés pour créer tout type d'application logicielle et pour permettre aux composants de l'application d'interagir entre eux.

### Le rôle du standard .NET

Le nombre de classes de base est plus élevé dans `.NET Framework` que dans `.NET (Core)` car il y'a 14 ans de différences entre leurs sortie.

La disparité entre les deux à créer des problèmes quand on a voulu passer du code .NET Framework en du code .NET Core.

La solution (et nécessité) pour l'Interopérabilité entre `.NET Framework 4.8.2` / `.NET Core 3.1` est `.NET Standard`.

.NET Standard et une spécification qui définis la disponibilité d'une API .NET et des `BCL` qui doivent être disponible pour chaque implémentation. Le standard active les scénarios suivants:

- Définis un ensemble uniforme d' API `BCL` pour toutes les implementation .NET. indépendant de la charge de travail.
- Permet aux développeurs de produire des bibliothèques portables utilisables dans toutes les implémentations .NET à l'aide de ce même ensemble d'API
- Réduit, voire élimine, la compilation conditionnelle des sources partagées due aux API .NET, uniquement pour les API du système d'exploitation 

Le tableau figurant dans la documentation Microsoft (https://docs.microsoft.com/en-us/dotnet/standard/net-standard) présente les différentes compatibilités entre .NET Framework et .NET. Il est utile pour les versions antérieures de C#. Cependant, C# 9+ ne fonctionnera que sur .NET 5+ ou .NET Standard 2.1, et .NET Standard 2.1 n'est pas disponible pour .NET Framework.

## Ce que C# apporte

`C#` à une syntaxe proche du `java` car ils sont membre de la *famille de language C* (C, C++, Objective-C). En vérité, beaucoup de constructions syntaxique sont modelé sur du Visual Basic (`VB`) et du `C++`.

Par exemple, tout comme`VB`, `C#` prend en charge la notion de *propriétés* de classe (par opposition aux *méthodes getter et setter* traditionnelles) et les paramètres facultatifs. Tout comme `C++`, `C#` vous permet de *surcharger les opérateurs*, ainsi que de créer des structures, des *énumérations* et des callback (via des *délégués*).

De plus, au fur et à mesure que vous avancerez dans ce texte, vous vous rendrez rapidement compte que C# prend en charge un certain nombre de fonctionnalités, telles que les *expressions lambda* et les *types anonymes*, que l'on trouve traditionnellement dans divers langages fonctionnels (par exemple, LISP ou Haskell).

 De plus, avec l'avènement du Language Integrated Query (`LINQ`), C# prend en charge un certain nombre de constructions qui le rendent tout à fait unique dans le paysage de la programmation. Néanmoins, l'essentiel de C# est en effet influencé par les langages basés sur C.

`C#` étant un hybride de nombreux langages, le résultat est un produit dont la syntaxe est aussi claire que celle de `Java` et qui offre à peu près autant de puissance et de flexibilité que C++. Voici une liste partielle des fonctionnalités essentielles de C# que l'on retrouve dans toutes les versions du langage :

- Aucun pointeur requis ! Les programmes C# n'ont généralement pas besoin de manipulation directe de pointeurs (Toujours possible, comme indiqué au [[Chapitre 11]]).
<br>
- Une gestion de la mémoire automatique avec un collecteur de déchets (*garbage collection*)
<br>
- Constructions syntaxiques formelles pour les classes, les interfaces, les structures, les énumérations et les délégués. 
<br>
- Possibilité de *surchargé les opérateurs* comme en `C++`.
<br>
- Prise en charge de la programmation basée sur les attributs. Ce type de développement vous permet d'annoter les types et leurs membres afin de mieux qualifier leur comportement. Par exemple, si vous marquez une méthode avec l'attribut *[Obsolete]*, les programmeurs verront s'afficher votre message d'avertissement personnalisé s'ils tentent d'utiliser le membre décoré.

### Principales fonctionnalités des versions précédentes

À partir de C# 7, j'ai commencé à ajouter dans les en-têtes de section la version à laquelle les fonctionnalités ont été ajoutées (==Nouveauté C# 7.x==, ou ==MaJ C# 7.x==).

### Nouvelle fonctionnalité dans C# 10

C# 10, sorti le 8 novembre 2021 avec .NET 6, ajoute les fonctionnalités suivantes :

- `record struct`
- Améliorations du type de donnée `struct`
- `global using` et `global implicit using`
- les `namespaces`au niveau des fichiers
- Améliorations du `pattern matching` de propriétés
- Chaînes interpolées constantes
- Améliorations des expressions `lambda`
- Améliorations du type `record`
- Affectation et déclaration dans le destructeur.
- Suppression des faux avertissements sur les affectations définies

## Code géré vs code non géré

Il est important de noter que le langage C# ne peut être utilisé que pour créer des logiciels hébergés sous le runtime .NET (vous ne pouvez en aucun cas utiliser C# pour créer un serveur COM natif ou une application de type C/C++ non gérée).

Officiellement, le terme utilisé pour décrire le code ciblant le runtime .NET est *"managed code"*. L'unité binaire qui contient le code géré est appelée *assembly* (nous reviendrons dessus plus en détail sur dans un instant). À l'inverse, le code qui ne peut pas être directement hébergé par le runtime .NET est appelé *unmanaged code*.

Comme mentionné précédemment, la plateforme .NET peut fonctionner sur divers systèmes d'exploitation. Il est donc tout à fait possible de créer une application C# sur un ordinateur Windows et d'exécuter le programme sur un appareil iOS à l'aide du runtime .NET. De même, vous pouvez créer une application C# sous Linux à l'aide de Visual Studio Code et exécuter le programme sous Windows. Avec Visual Studio pour Mac, vous pouvez également créer des applications .NET sur un Mac pour les exécuter sous Windows, macOS ou Linux.

Le code non géré reste accessible à partir d'un programme C#, mais il vous limite alors à une cible de développement et de déploiement spécifique.

# Utiliser des langages de programmation supplémentaire compatible .avec .NET

Il faut savoir que C# n'est pas le seul langage pouvant être utilisé pour créer des applications .NET. Les applications .NET peuvent généralement être créées avec C#, Visual Basic et F#, qui sont les trois langages directement pris en charge par Microsoft.

# Aperçu des assembly .NET

Quel que soit le langage .NET que vous choisissez pour programmer, sachez que même si les binaires .NET
ont la même extension de fichier que les binaires Windows non gérés (*xxx.dll*), ils n'ont absolument aucune similitude interne. Plus précisément, les binaires .NET ne contiennent pas d'instructions spécifiques à une plate-forme, mais plutôt un langage intermédiaire (IL) indépendant de la plate-forme et des métadonnées de type.

>[!Note]- Les différents noms du language *IL*
>IL est également connu sous le nom de *Microsoft Intermediate Language* (MSIL) ou encore sous le nom de *Common Intermediate Language* (CIL).
>
>Ainsi, lorsque vous lirez la documentation .NET, sachez que les termes IL, MSIL et CIL décrivent tous essentiellement le même concept. Dans cet ouvrage, j'utiliserai l'abréviation CIL pour désigner ce jeu d'instructions de bas niveau.

Quand le compilateur .NET créé un fichier *.dll*, le résultat binaire est nommé *assembly*. Les détails concernant les assembly .NET seront dans le [[Chapitre 16]] cependant pour faciliter la compréhension de ce point, il faut comprendre les 4 propriétés basique de ce format de fichier.

D'abord, contrairement aux `assembly .NET Framework` qui peuvent être soit un fichier *.exe*  ou un fichier *.dll*. ==Les projets `.NET` sont toujours compilés dans un fichier *.dll*, même si le projet fournit un exécutable==. Les assemblages .NET exécutables sont exécutés à l'aide de la commande ```dotnet <nom de l'assemblage>.dll```. Nouveauté dans .NET Core 3.0 (et versions ultérieures), la commande ```dotnet.exe``` est copiée dans le répertoire de compilation et renommée ```<nom de l'assemblage>.exe```. L'exécution de cette commande appelle automatiquement le fichier ```dotnet <nom de l'assembly>.dll```, exécutant l'équivalent de ```dotnet <nom de l'assembly>.dll```. **Le fichier *.exe* portant le nom de votre projet n'est pas réellement le code de votre projet; il s'agit d'un raccourci pratique pour exécuter votre application**.

Mis à jour dans .NET 6, votre application peut être réduit à un fichier unique qui peut être exécuté directement. Même si ce fichier unique ressemble à un exécutable natif de type C++ et se comporte comme tel, il s'agit en réalité d'un fichier unique créé pour des raisons pratiques. Il contient tous les fichiers nécessaires à l'exécution de votre application, voire même le runtime .NET lui-même ! Mais sachez que votre code s'exécute toujours dans un conteneur géré, comme s'il était publié sous forme de plusieurs fichiers.

Deuxièmement, un assembly contient du code `CIL`, qui est conceptuellement similaire au `bytecode` de `Java`, en ce sens qu'il n'est pas compilé en instructions spécifiques à la plate-forme tant que cela n'est pas absolument nécessaire. En général, « absolument nécessaire » correspond au moment où un bloc d'instructions `CIL` (tel qu'une implémentation de méthodes) est référencé pour être utilisé par le `runtime .NET`.

Troisièmement, les assemblies contiennent également des métadonnées qui décrivent en détail les caractéristiques de chaque *type de donnée* dans le binaire. Par exemple, si vous avez une classe nommée `SportsCar`, les métadonnées de type décrivent des détails tels que la classe de base de `SportsCar`, spécifient les *interfaces* implémentés par `SportsCar` (le cas échéant) et fournissent une description complète de chaque membre pris en charge par le type `SportsCar`. Les métadonnées .NET sont toujours présentes dans un *assembly* et sont générées automatiquement par le compilateur de langage.

Enfin, outre les métadonnées `CIL` et de `type de donnée`, les *assembly* eux-mêmes sont également décrits à l'aide de métadonnées, officiellement appelées *manifest*. Le manifeste contient des informations sur la version actuelle de l'*assembly*, **des informations culturelles (utilisées pour localiser les ressources de chaînes et d'images)** et une **liste de tous les assemblages externes référencés qui sont nécessaires à une exécution correcte**. Vous examinerez divers outils pouvant être utilisés pour examiner les types, les métadonnées et les informations du manifeste d'un assemblage au cours des prochains chapitres.

## Le rôle du CIL

Examinons plus en détail le code `CIL`, les métadonnées de type et le manifeste d'assemblage. **Le CIL est un langage qui se situe au-dessus de tout jeu d'instructions spécifique à une plate-forme particulière**. Par exemple, le code C# suivant modélise une calculatrice triviale. Ne vous préoccupez pas pour l'instant de la syntaxe exacte, mais remarquez le format de la méthode `Add()` dans la classe `Calc`.

```cs
//Calc.cs
Calc c = new Calc();
int ans = c.Add(10, 84);

Console.WriteLine("10 + 84 is {0}.", ans);

//Wait for user to press the Enter key
Console.ReadLine();

// The C# calculator.
class Calc
{
	public int Add(int addend1, int addend2)
    {
        return addend1 + addend2;
    }
}
```

Compiler ce code produit un fichier d'assemblage *.dll*  qui contient un manifeste, les instruction `CIL`, et les métadonnées décrivant chaque aspects des classes `Calc` et `program`.

>[!Note] Le [[Chapitre 2#Restauration de packages, compilation et exécution de programmes|Chapitre 2]] examine comment utiliser les `IDE` pour compiler des fichier code.

Par exemple, si vous deviez générer le code `IL` à partir de cet assemblage à l'aide de *ildasm.exe* (examiné plus loin dans ce chapitre), vous constateriez que la méthode `Add()` est représentée à l'aide du code `CIL` comme suit :

```CIL
.method public hidebysig instance int32 Add(int32 addend1, int32 addend2) cil managed
{
    // Method begins at RVA 0x2090
    // Code size 9 (0x9)
    .maxstack 2
    .locals /*11000002*/ init (int32 V_0)
    IL_0000: /* 00 | */ nop
    IL_0001: /* 03 | */ ldarg.1
    IL_0002: /* 04 | */ ldarg.2
    IL_0003: /* 58 | */ add
    IL_0004: /* 0A | */ stloc.0
    IL_0005: /* 2B | 00 */ br.s IL_0007
    IL_0007: /* 06 | */ ldloc.0
    IL_0008: /* 2A | */ ret
} // end of method Calc::Add
```

Ne vous inquiétez pas si vous ne comprenez pas grand-chose au `CIL` résultant de cette méthode, car le
[[Chapitre 18]] décrit les bases du langage de programmation `CIL`. **Ce qu'il faut retenir, c'est que le compilateur C# génère du `CIL`, et non des instructions spécifiques à une plate-forme**.

Rappelez-vous que cela vaut pour tous les compilateurs .NET. À titre d'illustration, supposons que vous ayez créé cette même application à l'aide de `Visual Basic` plutôt que `C#`.

```vb
' Calc.vb
Module Program
' This class contains the app's entry point.
    Sub Main(args As String())
        Dim c As New Calc
        Dim ans As Integer = c.Add(10, 84)

        Console.WriteLine("10 + 84 is {0}", ans)
        'Wait for user to press the Enter key before shutting down
        Console.ReadLine()
    End Sub
End Module

' The VB.NET calculator.
Class Calc
    Public Function Add(ByVal addend1 As Integer, ByVal addend2 As Integer) As Integer
        Return addend1 + addend2
    End Function
End Class
```

Si on examine le Code `CIL` pour la méthode `add()`, on remarquera des instructions similaires (légèrement modifié par le compilateur Visual Basic).

```CIL
.method public instance int32 Add(int32 addend1, int32 addend2) cil managed
{
    // Code size 9 (0x9)
    .maxstack 2
    .locals init (int32 V_0)
    IL_0000: nop
    IL_0001: ldarg.1
    IL_0002: ldarg.2
    IL_0003: add.ovf
    IL_0004: stloc.0
    IL_0005: br.s IL_0007
    IL_0007: ldloc.0
    IL_0008: ret
} // end of method Calc::Add
```

## Les bénéfices du CIL

1. Les language sont inter-opérant 
2. Indépendant des plateformes (OS)

## Compilation du CIL en instructions spécifiques à chaque plate-formes

On utilise un compilateur juste-à-temps (`JIT` ou `jitter`) pour compiler notre code `CIL` en instructions CPU. Le. runtime .NET utilise un compilateur `JIT` pour chaque processeur ciblant l'environnement d'exécution, chacun étant optimisé pour la plate-forme sous-jacente.

Par exemple, si vous développez une application .NET destinée à être déployée sur un appareil mobile (tel qu'un téléphone iOS ou Android), le `jitter` correspondant est parfaitement adapté pour fonctionner dans un environnement à faible mémoire. En revanche, si vous déployez votre assembly sur un serveur back-end d'entreprise (où la mémoire est rarement un problème), le jitter sera optimisé pour fonctionner dans un environnement à mémoire élevée. De cette manière, **les développeurs peuvent écrire un seul corps de code qui peut être compilé et exécuté efficacement par JIT sur des machines ayant des architectures différentes**.

De plus, comme un `jitter` donné compile les instructions `CIL` en code machine correspondant, il met en cache
les résultats dans la mémoire d'une manière adaptée au système d'exploitation cible. Ainsi, si un appel est effectué vers une méthode nommée `PrintDocument()`, les instructions `CIL` sont compilées en instructions spécifiques à la plate-forme lors de la première invocation et conservées en mémoire pour une utilisation ultérieure. *Par conséquent, la prochaine fois que `PrintDocument()` est appelé, il n'est pas nécessaire de re-compiler le CIL*.

### Pré-compilation du CIL en instructions spécifiques à la plateforme

Il existe dans .NET un utilitaire appelé *crossgen.exe*, qui peut être utilisé pour pré-JIT votre code. Heureusement, dans .NET Core 6, la capacité à produire des assemblages « prêts à l'emploi » est intégrée au framework. Nous y reviendrons plus en détail dans la suite de cet ouvrage.

## Le rôle des métadonnée des types de donnée .NET

En plus des instructions `CIL`, un assemblage .NET contient des métadonnées complètes, exhaustives et précises qui décrivent chaque type (par exemple: *classe*, *structure*, *énumération*) défini dans le binaire, ainsi que les membres de chaque type (par exemple, *propriétés*, *méthodes*, *événements*). **Heureusement, c'est toujours au compilateur (et non au programmeur) qu'il incombe d'émettre les métadonnées de type les plus récentes et les plus performantes**. Les métadonnées .NET étant extrêmement méticuleuses, les assemblages sont des entités entièrement auto-descriptives.

Pour illustrer le format des métadonnées de type .NET, examinons les métadonnées qui ont été générées pour la méthode `Add()` de la classe `C#` `Calc` que vous avez examinée précédemment <small>(les métadonnées générées pour la version Visual Basic de la méthode Add() sont similaires, nous n'examinerons donc que la version C#)</small>.

```CIL
// TypeDef #2 (02000003)
// -------------------------------------------------------
//   TypDefName: Calc (02000003)
//   Flags : [NotPublic] [AutoLayout] [Class] [AnsiClass] [BeforeFieldInit] (00100000)
//   Extends : 0100000D [TypeRef] System.Object
//   Method #1 (06000003)
// -------------------------------------------------------
//     MethodName: Add (06000003)
//     Flags : [Public] [HideBySig] [ReuseSlot] (00000086)
//     RVA : 0x00002090
//     ImplFlags : [IL] [Managed] (00000000)
//     CallCnvntn: [DEFAULT]
//     hasThis
//     ReturnType: I4
//     2 Arguments
//       Argument #1: I4
//       Argument #2: I4
//     2 Parameters
//         (1) ParamToken : (08000002) Name : addend1 flags: [none] (00000000)
//         (2) ParamToken : (08000003) Name : addend2 flags: [none] (00000000)
```

Les métadonnées sont utilisées par de nombreux aspects de l'environnement d'exécution .NET, ainsi que par divers outils de développement. **Par exemple, la fonctionnalité *IntelliSense* fournie par des outils tels que *Visual Studio* est rendue possible par la lecture des métadonnées d'un assemblage au moment de la conception**. Les métadonnées sont également utilisées par divers utilitaires de navigation d'objets, des outils de déboggage et le compilateur C# lui-même. Il est certain que les métadonnées constituent l'épine dorsale de nombreuses technologies .NET, notamment la *réflexion*, la *liaison tardive* et la *sérialisation d'objets*. Le [[Chapitre 17]] formalisera le rôle des métadonnées .NET.

## Le rôle du manifeste d'assembly

Enfin, n'oubliez pas q'==un assemblage .NET contient également des métadonnées qui décrivent l'assemblage lui-même (techniquement appelé *manifeste*)==. Entre autres détails, ==le manifeste documente tous les assemblages externes nécessaires au bon fonctionnement de l'assemblage actuel, le numéro de version de l'assemblage, les informations de copyright, etc==. Comme pour les métadonnées de type, *c'est toujours au compilateur qu'il revient de générer le manifeste de l'assemblage*. Voici quelques détails pertinents du manifeste généré lors de la compilation du fichier de code `Calc.cs` présenté plus haut dans ce chapitre (certaines lignes ont été omises par souci de concision) :

```CIL
.assembly extern /*23000001*/ System.Runtime
{
    .publickeytoken = (B0 3F 5F 7F 11 D5 0A 3A ) 
    .ver 6:0:0:0
    // .?_....:
}
.assembly extern /*23000002*/ System.Console
{
    .publickeytoken = (B0 3F 5F 7F 11 D5 0A 3A ) 
    .ver 6:0:0:0
    // .?_....:
}
.assembly /*20000001*/ Calc.Cs
{
    .hash algorithm 0x00008004
    .ver 1:0:0:0
}
.module Calc.Cs.dll
.imagebase 0x00400000
.file alignment 0x00000200
.stackreserve 0x00100000
.subsystem 0x0003 // WINDOWS_CUI
.corflags 0x00000001 // ILONLY
```

**En résumé, le manifeste documente l'ensemble des assemblages externes requis** par `Calc.dll` (via la directive `.assembly extern`) **ainsi que diverses caractéristiques de l'assemblage lui-même** (par exemple, le numéro de version, le nom du module). Le [[Chapitre 16]] examinera plus en détail l'utilité des données du manifeste.

# Comprendre le Common Type System (CTS)

Dans le monde de .NET, un *type* est simplement un terme générale qui réfère à un membre de l'ensemble contenant:

- *classe* -> class
- *interface*, -> interface
- *structure*, -> struct
- *enumeration* -> enum
- *délégué* -> delegate

Lorsque vous créez des solutions à l'aide d'un langage .NET, vous êtes susceptible d'interagir avec bon nombre de ces types. Par exemple, votre *assembly* peut définir une classe unique qui implémente plusieurs interfaces. Il se peut qu'une des méthodes d'interface prenne un type d'énumération comme paramètre d'entrée et renvoie une structure à l'appel.

Rappelons que le `CTS` est une **spécification formelle qui documente la manière dont les types doivent être définis afin d'être hébergés par le runtime .NET**. En général, les seules personnes qui s'intéressent de près au fonctionnement interne du `CTS` sont celles qui développent des outils et/ou des compilateurs destinés à la plateforme .NET. *Il est toutefois important que tous les programmeurs .NET apprennent à utiliser les cinq types définis par le CTS dans le langage de leur choix*. Voici un bref aperçu.

## Types de Classe CTS

Chaque langage .NET prend en charge, au minimum, la notion de type de classe, qui est la pierre angulaire de la *programmation orientée objet (POO)*. Une classe peut être composée d'un nombre quelconque de membres (tels que des constructeurs, des propriétés, des méthodes et des événements) et de points de données (champs). En C#, les classes sont déclarées à l'aide du mot-clé class, comme suit :

```cs
// A C# class type with 1 method.
class Calc
{
    public int Add(int addend1, int addend2)
    {
	    return addend1 + addend2;
    }
}
```

Le [[Chapitre 5#Présentation du type de classe C|Chapitre 5]] vous permettra d'aborder officiellement l'étude des type *class* en `C#`, cependant, le [[#Tableau 1-1: Les caractéristiques des classes CTS| Tableau 1-1]] présente un certain nombre de caractéristiques relative aux type *class*.

##### Tableau 1-1: Les caractéristiques des classes CTS

| Caractéristique de classes                          | Signification                                                                                                                                                                                                                                         |
| --------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Est ce que la classe est `sealed`?                  | Les classes scellé ne peuvent pas être utilisé comme classe de base pour les autres classes.                                                                                                                                                          |
| Est ce que la classe implémentent une *interface* ? | **Une *interface* est une collection de membres `abstract` qui propose un contrat entre l'objet et l'utilisateur de l'objet**. Le `CTS` permet à une classes d'implémenter un nombre illimité d'*interface*.                                          |
| Est ce que la classes est `abstract` ou non ?       | Les classes *abstract* ne peuvent pas être instanciée mais elle servent à définir le comportement commun d'un *type dérivé*.                                                                                                                          |
| Quel est la visibilité de la classe ?               | Chaque classe doit être configurée avec un mot-clé de visibilité comme `public`,`internal`. **En gros, cela permet de contrôler si la classe peut être utilisée par des assemblages externes ou uniquement à partir de l'assemblage qui la définit**. |

## Types d'Interface CTS

Les interfaces ne sont rien d'autre qu'un ensemble nommé de définitions de membres abstraits et/ou (introduit dans C# 8) d'implémentations par défaut, qui sont implémentés (facultativement dans le cas des implémentations par défaut) par une classe ou une structure donnée. En C#, les types d'interface sont définis à l'aide du mot-clé `interface`. Par convention, toutes les interfaces .NET commencent par une lettre majuscule *I*, comme dans l'exemple suivant :

```cs
// A C# interface type is usually
// declared as public, to allow types in other
// assemblies to implement their behavior.

public interface IDraw
{
    void Draw();
}
```

En elles-mêmes, les interfaces sont peu utiles. Cependant, lorsqu'une classe ou une structure implémente une interface donnée à sa manière, vous pouvez demander l'accès à la fonctionnalité fournie à l'aide d'une référence d'interface avec le *polymorphisme (Un des principe de la POO)*. La programmation basée sur les interfaces sera abordée en détail au [[Chapitre 8]].

## Type de Structure CTS

Le concept de structure est également formalisé dans le `CTS`. Si vous avez des connaissances en C, vous serez heureux d'apprendre que ces *types définis par l'utilisateur (UDT)* ont survécu dans le monde .NET (même s'ils se comportent un peu différemment sous le capot). **En termes simples, une structure peut être considérée comme un type de classe léger ayant une sémantique basée sur les valeurs (value types)**. Pour plus de détails sur les subtilités des structures, voir le [[Chapitre 4#Comprendre les structures (`struct`)|Chapitre 4]]. En général, les structures sont particulièrement adaptées à la modélisation de données géométriques et mathématiques et sont créées en `C#` à l'aide du mot-clé `struct`, comme suit :

```cs
// A C# structure type.
struct Point
{
    // Structures can contain fields.
    public int xPos 
    public int yPos;
    
    // Structures can contain parameterized constructors.
    public Point(int x, int y)
    { 
        xPos = x; 
        yPos = y;
    }
    
    // Structures may define methods.
    public void PrintPosition()
    {
        Console.WriteLine("({0}, {1})", xPos, yPos);
    }
}
```

## Types d'Énumération CTS

Les *énumérations* sont une construction de programmation pratique qui vous permet de regrouper des *paires nom-valeur*. Par exemple, supposons que vous créiez une application de jeu vidéo qui permet au joueur de choisir parmi trois catégories de personnages (sorcier, guerrier ou voleur). Plutôt que de garder une trace de simples valeurs numériques pour représenter chaque possibilité, vous pouvez créer une énumération fortement typée à l'aide du mot-clé `enum`.

```cs
// A C# enumeration type.
enum CharacterTypeEnum
{
    Wizard = 100,
    Fighter = 200,
    Thief = 300
}
```

Par défaut, l'espace de stockage utilisé pour contenir chaque élément est un entier 32 bits ; cependant, il est possible de modifier cet emplacement de stockage si nécessaire (l'attribut [flags] permet d'utiliser le décalage de bit pour définir la valeur associé au nom). De plus, le CTS exige que les types énumérés dérivent d'une classe de base commune, `System.Enum`. Comme vous le verrez au [[Chapitre 4#Comprendre le type de donnée `enum`|Chapitre 4]], cette classe de base définit un certain nombre de membres intéressants qui vous permettent d'extraire, de manipuler et de transformer les paires nom-valeur sous-jacentes par programmation.

## Types de Délégué CTS

Les délégués sont l'équivalent .NET d'un ==pointeur de fonction sécurisé en language C==. La principale différence réside dans le fait qu'un délégué .NET est une classe dérivée de `System.MulticastDelegate`, plutôt qu'un simple pointeur vers une adresse mémoire brute. En C#, les délégués sont déclarés à l'aide du mot-clé `delegate`.

```cs
// This C# delegate type can "point to" any method
// returning an int and taking two ints as input.
delegate int BinaryOp(int x, int y);
```

Les délégués sont essentiels lorsque vous souhaitez permettre à un objet de transférer un appel vers un autre objet et constituent la base de l'architecture événementielle .NET. Comme vous le verrez dans les [[Chapitre 12]] et [[Chapitre 14]], les délégués prennent en charge de manière intrinsèque le *multicast* (c'est-à-dire le transfert d'une requête vers plusieurs destinataires) et les appels de méthode *asynchrones* (c'est-à-dire l'appel de la méthode sur un thread secondaire).

## Types de Membres CTS

Maintenant que vous avez passé en revue chacun des types formalisés par le `CTS`, sachez que la plupart des types acceptent un nombre quelconque de membres. D'un point de vue formel, un membre de type est contraint par l'ensemble suivant: 

- *constructeur* -> constructor
- *finaliseur* -> finalizer ou desctructor
- *constructeur statique* -> static constructor
- *type imbriqué* -> nested type
- *opérateur* -> operator
- *méthode* -> method
- *propriété* -> property
- *indexeur* -> indexer
- *champ* -> fields
- *champ en lecture seule* -> read only fields
- *constante* -> const
- *événement* -> event

**Le `CTS` définit divers ornements qui peuvent être associés à un membre donné. Par exemple, chaque
membre a un *trait de visibilité* donné (par exemple, *public*, *privé*, *protégé*). Certains membres peuvent être déclarés comme *abstraits* (pour imposer un comportement polymorphe sur les types dérivés) ou *virtuels* (pour définir une implémentation prédéfinie, mais remplaçable). De plus, la plupart des membres peuvent être configurés comme *statiques* (liés au niveau de la classe) ou *instance* (liés au niveau de l'objet)**. La création de membres de type est examinée au cours des prochains chapitres.

>[!Info]-
>comme décrit dans le [[Chapitre 10]], le language C# supporte aussi la création de type de donnée *générique* ainsi que de membre *générique*.

## Types de données intrinsèques CTS

Le dernier aspect du `CTS` à prendre en compte pour l'instant est qu'il établit un ensemble bien défini de
types de données fondamentaux. Bien qu'un langage donné dispose généralement d'un mot-clé unique utilisé pour déclarer un type de données fondamental, tous les mots-clés des langages .NET se résolvent finalement au même type `CTS` défini dans un assembly nommé *mscorlib.dll*. Consultez le [[#Tableau 1-2 Les type de données intrinsèques CTS|tableau 1-2]], qui documente la manière dont les types de données `CTS` clés sont exprimés en `VB.NET` et `C#`.
##### Tableau 1-2: Les type de données intrinsèques CTS

> Aussi appelé primitifs

| Type de donnée CTS | Mot-clé VB | Mot-clé C# |
| ------------------ | ---------- | ---------- |
| `System.Byte`      | Byte       | byte       |
| `System.SByte`     | SByte      | sbyte      |
| `System.Int16`     | Short      | short      |
| `System.Int32`     | Integer    | int        |
| `System.Int64`     | Long       | long       |
| `System.UInt16`    | UShort     | ushort     |
| `System.UInt32`    | UInteger   | uint       |
| `System.UInt64`    | ULong      | ulong      |
| `System.Single`    | Single     | float      |
| `System.Double`    | Double     | double     |
| `System.Object`    | Object     | object     |
| `System.Char`      | Char       | char       |
| `System.String`    | String     | string     |
| `System.Decimal`   | Decimal    | decimal    |
| `System.Boolean`   | Boolean    | bool       |

Étant donné que les mots-clés uniques d'un langage géré ne sont que des notations abrégées pour un type réel dans le *namespace* du système, vous n'avez plus à vous soucier des conditions d'*overflow* et d'*underflow* pour les données numériques ou de la manière dont les chaînes et les booléens sont représentés en interne dans différents langages. Considérez les extraits de code suivants, qui définissent des variables numériques 32 bits en `C#` et `Visual Basic`, en utilisant des mots-clés de langage ainsi que le type de données `CTS` formel :

```cs
// Define some "ints" in C#.
int i = 0;
System.Int32 j = 0;
```

```vb
' Define some "ints" in VB.
Dim i As Integer = 0
Dim j As System.Int32 = 0
```

# Comprendre le Common Language Specification (CLS)

Comme vous le savez, différents langages expriment les mêmes constructions de programmation en termes uniques et spécifiques à chaque langage. Par exemple, en C#, vous désignez la concaténation de chaînes à l'aide de l'opérateur plus (+), tandis qu'en VB, vous utilisez généralement l'esperluette (&). Même lorsque deux langages distincts expriment le même idiome programmatique (par exemple, une fonction sans valeur de retour), il y a de fortes chances que la syntaxe apparaisse très différente en surface.

```cs
// C# method returning nothing.
public void MyMethod()
{
    // Some interesting code...
}
```

```vb
' VB method returning nothing.
Public Sub MyMethod()
    ' Some interesting code...
End Sub
```

Du au fait que les deux language utilise le `CIL`, l**es deux compilateurs (*csc.exe* ou *vbc.exe*) émettent les mêmes jeux d'instructions**. Cependant, les langages peuvent également différer en ce qui concerne leur niveau global de fonctionnalité. Par exemple, un langage .NET peut ou non disposer d'un mot-clé pour représenter les données non signées et peut ou non prendre en charge les types de pointeurs. Compte tenu de ces variations possibles, l'idéal serait de disposer d'une base de référence à laquelle tous les langages .NET devraient se conformer.

**Le CLS est un ensemble de règles qui décrivent en détail l'ensemble minimal et complet de fonctionnalités qu'un compilateur .NET donné doit prendre en charge pour produire du code pouvant être hébergé par le runtime .NET, tout en étant accessible de manière uniforme par tous les langages ciblant la plateforme .NET**. À bien des égards, *le CLS peut être considéré comme un sous-ensemble de l'ensemble des fonctionnalités définies par le CTS*.

Le `CLS` est en fin de compte un ensemble de règles auxquelles les développeurs de compilateurs doivent se conformer s'ils veulent que leurs produits fonctionnent de manière transparente dans l'univers `.NET`. Chaque règle se voit attribuer un nom simple (par exemple, règle `CLS 6`) et décrit comment cette règle affecte ceux qui développent les compilateurs ainsi que ceux qui (d'une manière ou d'une autre) interagissent avec eux. La crème de la crème du `CLS` est la règle 1.

	Rule 1: CLS rules apply only to those parts of a type that are exposed outside the defining assembly.

Compte tenu de cette règle, vous pouvez (à juste titre) en déduire que les autres règles du `CLS` ne s'appliquent pas à la logique utilisée pour construire le fonctionnement interne d'un type .NET. **Les seuls aspects d'un type qui doivent être conformes au `CLS` sont les définitions des membres elles-mêmes (c'est-à-dire les conventions de nommage, les paramètres et les types de retour)**. La logique d'implémentation d'un membre peut utiliser un nombre illimité de techniques non `CLS`, car le monde extérieur ne verra pas la différence.

À titre d'illustration, la méthode `C#` `Add()` suivante n'est pas conforme au `CLS`, car les paramètres et les valeurs de retour utilisent des données non signées (ce qui n'est pas une exigence du `CLS`) :

```cs
public class Calc
{
    // Exposed unsigned data is not CLS compliant!
    public ulong Add(ulong addend1, ulong addend2)
    {
        return addend1 + addend2;
    }
}
```

Cependant, considérez le code suivant qui utilise des données non signées en interne dans une méthode :

```cs
public class Calc
{
    public int Add(int addend1, int addend2)
    {
        // As this ulong variable is only used internally,
        // we are still CLS compliant.
        ulong temp = 0;
        ...
        return addend1 + addend2;
    }
}
```

La classe reste conforme aux règles du `CLS` et peut être assurée que tous les langages `.NET` sont capables d'appeler la méthode `Add()`.

Bien sûr, en plus de la *règle 1*, le `CLS` définit de nombreuses autres règles. Par exemple, le CLS décrit la manière dont un langage donné doit représenter les chaînes de texte, la manière dont les énumérations doivent être représentées en interne (le type de base utilisé pour le stockage), la manière de définir les membres statiques, etc. Heureusement, vous n'avez pas besoin de mémoriser ces règles pour devenir un développeur .NET compétent. Encore une fois, dans l'ensemble, une compréhension approfondie des spécifications CTS et CLS n'intéresse généralement que les créateurs d'outils/compilateurs.

## Garantir la conformité CLS

Comme vous le verrez au fil de cet ouvrage, `C#` définit un certain nombre de constructions de programmation qui ne sont pas conformes à `CLS`. La bonne nouvelle, cependant, est que vous pouvez demander au compilateur C# de vérifier la conformité `CLS` de votre code à l'aide d'un seul attribut .NET.

```cs
// Demandez au compilateur C# de vérifier la conformité CLS.
[assembly: CLSCompliant(true)]
```

Le [[Chapitre 17]] aborde en détail la programmation basée sur les attributs. Pour l'instant, il suffit de comprendre que l'attribut [CLSCompliant] demande au compilateur C# de vérifier chaque ligne de code par rapport aux règles du `CLS`. **Si des violations du `CLS` sont détectées, vous recevez un avertissement du compilateur et une description du code incriminé**.

# Comprendre le Runtime .NET

Outre les spécifications `CTS` et `CLS`, la dernière pièce du puzzle à prendre en compte est le `runtime .NET`. D'un point de vue programmation, le terme *runtime* peut être compris comme un ==ensemble de services nécessaires à l'exécution d'une unité de code compilée donnée==. Par exemple, lorsque les développeurs Java déploient un logiciel sur un nouvel ordinateur, ils doivent s'assurer que la machine virtuelle Java (JVM) a été installée sur la machine afin de pouvoir exécuter leur logiciel. 

La plateforme .NET offre un autre système d'exécution. La principale différence entre le runtime .NET et les autres runtimes que je viens de mentionner est que le runtime .NET fournit une couche d'exécution unique et bien définie qui est partagée par tous les langages et toutes les plateformes .NET.

# Distinguer Assembly, Espace de Noms et Type

Nous comprenons tous l'importance des bibliothèques de codes. L'intérêt des bibliothèques de frameworks est de fournir aux développeurs un ensemble bien défini de codes existants qu'ils peuvent exploiter dans leurs applications. Cependant, le langage C# n'est pas fourni avec une bibliothèque de codes spécifique. Les développeurs C# exploitent plutôt les bibliothèques .NET, qui sont indépendantes du langage. Afin de bien organiser tous les types dans les bibliothèques de classes de base, la plateforme .NET utilise largement le concept d'**espace de noms** (*namespace*).

**Un espace de noms est un regroupement de types sémantiquement liés contenus dans un assemblage ou éventuellement répartis sur plusieurs assemblages liés**. Par exemple, l'espace de noms `System.IO` contient des types liés aux E/S de fichiers, l'espace de noms `System.Data` définit des types de base de données de base, etc. Il est important de souligner qu'un seul assemblage peut contenir un nombre illimité d'espaces de noms, chacun pouvant contenir un nombre illimité de types. 

La principale différence entre cette approche et une bibliothèque spécifique à un langage réside dans le fait que tout langage ciblant le runtime .NET utilise les mêmes espaces de noms et les mêmes types. Par exemple, les deux programmes suivants illustrent l'application omniprésente Hello World, écrite en C# et VB :

>[!Note] 
>le code suivant utilise la version C# 9 de la classe `Program` avec une méthode `void Main()` pour aider à illustrer cet exemple. Les nouveaux patrons en C# 10 utilise la *déclaration de haut niveau* (abordé dans le [[Chapitre 3#Utiliser les déclaration de haut niveaux (Nouveauté C 9.0)|Chapitre 3]]) et des déclaration *global implicit using* (abordé plus tard dans ce chapitre).

```cs
// Hello World in C#.
using System;

public class MyApp
{
    static void Main()
    {
        Console.WriteLine("Hi from C#");
    }
}
```

```vb
' Hello World in VB.
Imports System
Public Module MyApp
    Sub Main()
        Console.WriteLine("Hi from VB")
    End Sub
End Module
```

**Les deux language utilisent la classe `Console` définie dans le namespace `System`.**

Une fois que vous maîtrisez le langage de programmation .NET de votre choix, votre prochain objectif en tant que développeur .NET est clairement d'apprendre à connaître la multitude de types définis dans les (nombreux) espaces de noms .NET. **L'espace de noms le plus fondamental à comprendre au départ est celui nommé `System`. Cet espace de noms fournit un ensemble de types essentiels dont vous aurez besoin à maintes reprises en tant que développeur .NET. En fait, vous ne pouvez pas créer d' application C# fonctionnelle sans faire au moins référence à l'espace de noms System, car les types de données de base (par exemple, System.Int32, System.String) y sont définis**. Le [[#Tableau 1-3 Un échantillon de namespace .NET|tableau 1-3]] présente un aperçu de certains (mais certainement pas tous) des espaces de noms .NET regroupés par fonctionnalité.

##### Tableau 1-3 Un échantillon de namespace .NET

| .NET namespace                                                     | Description                                                                                                                                                                                                                                                                                      |
| ------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `System`                                                           | Dans `system`, vous trouverez de nombreux types utiles traitant des données intrinsèques, des calculs mathématiques, de la génération de nombres aléatoires, des variables d'environnement et de la gestion mémoire, ainsi qu'un certain nombre d'exceptions et d'attributs couramment utilisés. |
| `System.Collection` `System.Collection.Generic`                    | Ces espaces de noms définissent un certain nombre de types de conteneurs standard, ainsi que des types de base et des interfaces qui vous permettent de créer des *collections personnalisées*.                                                                                                  |
| `System.Data`<br>`System.Data.Common` `System.Data.sqlClient`      | Ces espaces de noms sont utilisé pour s'interfacer avec des bases de donnée relationnelles en utilisant *ADO.NET*                                                                                                                                                                                |
| `System.IO`<br>`System.IO.Compression`  <br>`System.IO.Ports`      | Ces espaces de noms définissent de nombreux types utiles pour travaillé avec des *I/O de fichiers*, *compression de donnée* et la *manipulation de port*.                                                                                                                                        |
| `System.Reflection` `System.reflextion.Emit`                       | Ces espaces de noms définissent des types qui prennent en charge la découverte de types à l'exécution ainsi que la création dynamique de types. Ces espaces de noms sont destinés à être utilisés par des applications qui nécessitent une découverte dynamique de types                         |
| `System.Runtime.InteropServices`                                   | Cet espace de noms fournit des fonctionnalités ==permettant aux types .NET d'interagir avec du code non géré== (par exemple, des *DLL* basées sur `C` et des *serveurs COM*) et vice versa.                                                                                                      |
| `System.Drawing`<br>`System.Windows.Forms`                         | Ces espaces de noms définissent les types utilisés pour créer des applications de bureau à l'aide de la boîte à outils d'interface utilisateur d'origine de .NET (Windows Forms).                                                                                                                |
| `System.Windows` `System.Windows.Controls` `system.Windows.Shapes` | l'espace de nom `System.Windows` est la racine de nombreux espaces de nom qui sont utilisé et dans les application *WPF* et dans les application *Windows Forms*                                                                                                                                 |
| `System.Linq`<br>`System.Linq.Expression`                          | Ces espaces de noms définissent les types utilisés lors de la programmation avec l'API LINQ.                                                                                                                                                                                                     |
| `System.AspNetCore`                                                | Il s'agit de l'un des nombreux espaces de noms qui vous permettent de créer des *applications web* `ASP.NET Core` et des *services RESTful*.                                                                                                                                                     |
| `System.Threading` `System.Threading.Tasks`                        | Ces espaces de noms définissent de nombreux types permettant de créer des applications multithread capables de répartir les charges de travail sur plusieurs coeurs ou plusieurs processeurs.                                                                                                    |
| `System.Security`                                                  | La sécurité est un aspect intégré de l'univers .NET. Dans les espaces de noms centrés sur la sécurité, vous trouverez de nombreux types traitant des *autorisations*, de la *cryptographie*, etc.                                                                                                |
| `System.Xml`                                                       | Les espaces de noms centrés sur `XML` contiennent de nombreux types utilisés pour interagir avec les données *.xml*.                                                                                                                                                                             |

## Accéder à un espace de noms par programmation

Il convient de rappeler qu'un espace de noms n'est rien d'autre qu'un moyen pratique pour nous, simples humains, de comprendre et d'organiser logiquement des types connexes. Reprenons l'exemple de l'espace de noms `System`. De votre point de vue, vous pouvez supposer que `System.Console` représente une classe nommée `Console` contenue dans un espace de noms appelé `System`. Cependant, **aux yeux du runtime .NET, ce n'est pas le cas. Le moteur d'exécution ne voit qu'une seule classe nommée System.Console**. 

En C#, le mot-clé `using` simplifie le processus de référence aux types définis dans un espace de noms particulier. Voici comment cela fonctionne. Pour revenir à l'exemple de programme *Calc* présenté plus haut dans ce chapitre, il y a une seule instruction `using` en haut du fichier.

```cs
using System;
```

Cette instruction est un raccourci permettant d'activer cette ligne de code :

```cs
Console.WriteLine("10 + 84 = {0}", ans);
```

Sans l'instruction using, le code devrait être écrit comme suit :

```cs
System.Console.WriteLine("10 + 84 = {0}", ans);
```

Bien que la définition d'un type à l'aide du nom complet offre une meilleure lisibilité, je pense que vous conviendrez que le mot-clé `using` de C# réduit le nombre de frappes. Dans ce texte, nous éviterons d'utiliser des noms complets (sauf en cas d'ambiguïté manifeste à résoudre) et opterons pour l'approche simplifiée du mot-clé `using` de C#.

**Cependant, n'oubliez jamais que le mot-clé `using` est simplement une notation abrégée pour spécifier le nom complet d'un type, et que les deux approches aboutissent au même CIL sous-jacent** (étant donné que le code CIL utilise toujours des noms complets) **et n'ont aucun effet sur les performances ou la taille de l'assembly**. C'est pourquoi nous n'avons pas de raison de ne pas utiliser le mot-clé `using`.

## La Déclaration Global Using (Nouveauté C# 10)

Lorsque vous développez des applications C# plus complexes, vous aurez très probablement des espaces de noms qui se répètent dans plusieurs fichiers. Introduits dans C# 10, les espaces de noms peuvent être référencés globalement, puis être automatiquement disponibles dans tous les fichiers du projet. Il suffit d'ajouter le mot-clé `global` devant vos instructions `using`, comme ceci :

```cs
global using System;
```

>[!warning] toutes les déclarations `global using` doivent venir avant toute déclaration n'utilisant pas `global`.

Il est recommandé de placer les instructions `global using` avec vos instructions de niveau supérieur
(abordées au [[Chapitre 3]]) ou dans un fichier complètement séparé (tel que GlobalUsings.cs) pour une meilleure visibilité. Vous trouverez de nombreux exemples à ce sujet tout au long de ce texte.

En plus de placer les instructions using globales dans Program.cs (ou dans un fichier séparé), elles peuvent être placées dans le fichier de projet de l'application (fichier *.csproj*) en utilisant le format suivant :

```xml
<ItemGroup>
    <Using Include=”System.Text” />
    <Using Include=”System.Text.Encodings.Web” />
    <Using Include=”System.Text.Json” />
    <Using Include=”System.Text.Json.Serialization” />
</ItemGroup>
```

### Les déclarations Implicit Global Using (Nouveauté C# 10)

Une autre nouvelle fonctionnalité incluse dans .NET 6/C# 10 concerne les instructions using globales implicites. Les instructions *using globales implicites* fournies par .NET 6 varient en fonction du type d'application que vous développez. Le [[#Tableau 1-4 Un échantillon d'espace noms .NET|tableau 1-4]] répertorie les types d'applications et les espaces de noms inclus.

##### Tableau 1-4: Un échantillon d'espace noms .NET

| Type d'application .NET                     | Les espaces noms couvert par la déclaration `implicit global using`                                                                                                                                                                                                                                                                                         |
| ------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Client (Microsoft.NET.Sdk)`                | `System`<br>`System.Collection.Generic`<br>`System.IO`<br>`System.Linq`<br>`System.Net.Http`<br>`System.Threading`<br>`System.Threading.Tasks`                                                                                                                                                                                                              |
| `Web (Microsoft.NET.Sdk.Web)`               | Tout dans `Microsoft.NET.Sdk` plus:<br>`System.NET.Http.Json`<br>`Microsoft.AspNetCore.Builder`<br>`Microsoft.AspNetCore.Hosting`<br>`Microsoft.AspNetCore.Http`<br>`Microsoft.AspNetCore.Routing`<br>`Microsoft.Extensions.Configuration`<br>`Microsoft.Extensions.DepedencyInjection`<br>`Microsoft.Extensions.Hosting`<br>`Microsoft.Extensions.Logging` |
| `Worker Service (Microsoft.NET.Sdk.Worker)` | Tout dans `Microsoft.NEt.Sdk` plus:<br>`Microsoft.Extensions.Configuration`<br>`Microsoft.Extensions.DepedencyInjection`<br>`Microsoft.Extentions.Hosting`<br>`Microsoft.Extensions.Logging`                                                                                                                                                                |

La grande majorité des modèles de projet C# 10 activent par défaut les instructions using implicites globales à l'aide de l'élément `ImplicitUsings` dans le groupe `Property` principal du projet (fichier *.csproj*). Pour désactiver ce paramètre, mettez à jour le fichier de projet comme suit :

```xml
<PropertyGroup>
    ...
    <ImplicitUsings>disable</ImplicitUsings>
    ...
</PropertyGroup>
```

Pour voir les déclarations d'utilisation globales dans votre projet, recherchez le fichier *"nom du project".GlobalUsings.g.cs* dans le dossier `\obj\Debug\net6.0`. Pour le projet *Calc.cs*, voici le code généré :

```cs
// <auto-generated/>
global using global::System;
global using global::System.Collections.Generic;
global using global::System.IO;
global using global::System.Linq;
global using global::System.Net.Http;
global using global::System.Threading;
global using global::System.Threading.Tasks;
```

## Espaces de noms au niveau du fichier (Nouveauté C# 10)

Autre nouveauté de C# 10, les espaces de noms au niveau du fichier suppriment la nécessité d'encadrer votre code entre accolades lorsque vous le placez dans un espace de noms personnalisé. Prenons l'exemple suivant de la classe `Calculator`, contenue dans l'espace de noms `CalculatorExamples`. Avant C# 10, pour placer une classe dans un espace de noms, il fallait la déclaration de l'espace de noms, une accolade ouvrante, le code (Calculator), puis une accolade fermante. Dans l'exemple:

```cs
namespace CalculatorExamples
{
    class Calculator()
    {
        ...
    }
}
```

À mesure que votre code devient plus complexe, cela peut ajouter beaucoup de code et d'indentation supplémentaires. Avec les espaces de noms au niveau du fichier, le code suivant permet d'obtenir le même effet :

```cs
namespace CalculatorExamples

class Calculator()
{
    ...
}
```

>[!Note]- 
>Les espaces de noms personnalisé sont abordé en profondeur dans le [[Chapitre 16]].

## Référencement d'assemblages externes

Les versions antérieures du .NET Framework utilisaient un emplacement d'installation commun pour les bibliothèques du framework, appelé *Global Assembly Cache (GAC)*. Au lieu d'avoir un emplacement d'installation unique, .NET n'utilise pas le *GAC*. Chaque version (y compris les versions mineures) est installée dans son propre emplacement (par version) sur l'ordinateur. Sous Windows, chaque version du runtime et du SDK est installée dans `c:\ProgramFiles\dotnet`.

L'ajout d'assemblages dans la plupart des projets .NET s'effectue en ajoutant des ==packages NuGet== (abordés plus loin dans ce texte). Cependant, les applications .NET ciblant (et développées sur) Windows ont toujours accès aux bibliothèques COM.

Pour qu'un assemblage ait accès à un autre assemblage que vous créez (ou que quelqu'un a créé pour vous), vous devez ajouter une référence de votre assemblage à l'autre assemblage et avoir un accès physique à l'autre assemblage. Selon l'outil de développement que vous utilisez pour créer vos applications .NET, vous disposerez de différents moyens pour indiquer au compilateur les assemblages que vous souhaitez inclure pendant le cycle de compilation.

# Exploration d'un assembly à l'aide de *ildasm.exe*

Si vous commencez à vous sentir un peu dépassé à l'idée de devoir maîtriser tous les espaces de noms de la plateforme .NET, rappelez-vous simplement que **ce qui rend un espace de noms unique, c'est qu'il contient des types qui sont d'une manière ou d'une autre liés sémantiquement**. Par conséquent, ==si vous n'avez pas besoin d'une interface utilisateur au-delà d'une simple application console, vous pouvez oublier les espaces de noms desktop et web (entre autres). Si vous développez une application de peinture, les espaces de noms de base de données ne vous concernent probablement pas beaucoup==. Vous apprendrez au fil du temps quels sont les espaces de noms les plus pertinents pour vos besoins en matière de programmation.

L'utilitaire *Intermediate Language Disassembler (ildasm.exe)* vous permet de **créer un document texte représentant un assemblage .NET et d'en examiner le contenu, y compris le manifeste associé, le code CIL et les métadonnées de type**. Cet outil vous permet d'approfondir vos connaissances sur la manière dont le code C# est "mappé" vers le CIL et, au final, vous aide à comprendre le fonctionnement interne de la plateforme .NET. Même si vous n'avez pas besoin d'utiliser ildasm.exe pour devenir un programmeur .NET compétent, je vous recommande vivement de lancer cet outil de temps en temps afin de mieux comprendre comment votre code C# est mappé vers les concepts d'exécution.

> [!important]
> L'utilitaire *ildasm.exe* n'est plus fourni avec le runtime .NET 6. Pour pouvoir récupérer l'outil: 
> 1.  Allez sur https://www.nuget.org/packages/Microsoft.NETCore.ILDAsm/ et télécharger à l'ancienne le programme.
> 2. Avec .NET CLI : ```dotnet add package Microsoft.NETCore.ILDAsm --version du runtime```
>    >[!warning] pour pouvoir l'utiliser sans devoir toujours ajouter le chemin d'accès. il faut l'ajouter manuellement dans le `PATH`
>    
>    >[!warning] Pour MacOS, if faut utiliser des `-` au lieu des `/` pour les options disponible pour la commande `ildasm`

Une fois *ildasm.exe* chargé sur votre ordinateur, vous pouvez exécuter le programme à partir de la ligne de commande sans aucun argument pour afficher les commentaires d'aide. Vous devez au minimum spécifier l'assembly à extraire du CIL.

Voici un exemple de ligne de commande :

- Pour Windows :
	```cmd
	ildasm /all /METADATA /out=csharp.il calc.cs.dll
	```
- Pour MacOS / Linux :
	```bash
	ildasm -all -metadata -out=csharp.il calc.cs.dll
	```

Cela créera un fichier nommé csharp.il exportant toutes les données disponibles dans le fichier. C'est le fichier d'où proviennent les exemples IL précédents.

# Résumé du chapitre

L'objectif de ce chapitre était de présenter le cadre conceptuel nécessaire à la compréhension du reste de cet ouvrage. J'ai commencé par examiner un certain nombre de limites et de complexités inhérentes aux technologies antérieures à .NET, puis j'ai présenté une vue d'ensemble de la manière dont .NET et C# tentent de simplifier la situation actuelle.

==NET se résume essentiellement à un moteur d'exécution (le .NET Runtime) et à des bibliothèques de classes de base. Le runtime est capable d'héberger n'importe quel binaire .NET (alias assembly) qui respecte les règles du code géré. Comme vous l'avez vu, les assemblies contiennent des instructions CIL (en plus des métadonnées de type et du manifeste de l'assembly) qui sont compilées en instructions spécifiques à la plate-forme à l'aide d'un compilateur juste-à-temps. En outre, vous avez exploré le rôle de la spécification de langage commun(`CLS`) et du système de types commun(`CTS`).==

Dans le chapitre suivant, vous découvrirez les environnements de développement intégrés courants que vous pouvez utiliser pour créer vos projets de programmation C#. Vous serez heureux d'apprendre que dans ce livre, vous utiliserez des IDE entièrement gratuits (et riches en fonctionnalités), ce qui vous permettra de commencer à explorer l'univers .NET sans dépenser un centime.
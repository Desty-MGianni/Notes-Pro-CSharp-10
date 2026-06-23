---
title: "Chapitre 16: Création et Configuration des Bibliothèques de Classes"
publish: true
---

# <big><big><big><b><font color =green>Création et Configuration des Bibliothèques de Classes</font></b></big></big></big>

**Dans la plupart des exemples présentés jusqu'ici dans ce livre, vous avez créé des applications exécutables « autonomes », dans lesquelles toute la logique de programmation était regroupée dans un seul assembly** (*.dll*) **et exécutée à l'aide de** *dotnet* (ou d'une copie de *dotnet* portant le nom de l'assembly). Ces assemblies utilisaient peu de choses autre que les bibliothèques de classes de base .NET. Bien que certains programmes .NET simples puissent être construits en utilisant uniquement les bibliothèques de classes de base, **==il est probable que vous==** (ou vos collègues) **==ayez souvent recours à l'isolation de la logique de programmation réutilisable dans des bibliothèques de classes *personnalisées*==** (fichiers *.dll*), **==partageables entre les applications.==**

***==Dans ce chapitre, vous commencerez par apprendre en détail le partitionnement des types dans les espaces de noms .NET. Ensuite, vous étudierez en profondeur les bibliothèques de classes .NET, la différence entre .NET et .NET Standard, la configuration des applications, la publication d'applications console .NET et la création de packages NuGet réutilisables pour vos bibliothèques.==***

# Définition des espaces de noms personnalisés (MaJ C# 10.0)

>[!tip] Toutes les allusions à un fichier *.exe* sera remplacé par *.dll*. Voir [[Chapitre 1#Aperçu des assembly .NET|Chapitre 1]]

Avant d'aborder le déploiement et la configuration de la bibliothèque, **la première étape consiste à comprendre en détail le packaging de vos types personnalisés dans des espaces de noms .NET**. ==Jusqu'à présent, vous avez créé de petits programmes de test exploitant les espaces de noms existants de l'univers .NET== (`System`, notamment). Cependant, **lorsque vous développez des applications plus importantes avec de nombreux types, il peut être utile de regrouper vos types apparentés dans des espaces de noms personnalisés**. **==En C#, cela se fait à l'aide du mot-clé `namespace`==**. ***==Définir explicitement des espaces de noms personnalisés est encore plus important lors de la création d'assemblies partagés, car les autres développeurs devront référencer la bibliothèque et importer vos espaces de noms personnalisés pour utiliser vos types==***. **==Les espaces de noms personnalisés évitent également les conflits de noms en séparant vos classes personnalisées des autres classes personnalisées qui pourraient porter le même nom.==**

Pour examiner ces questions concrètement, commencez par créer un nouveau projet d'application console .NET nommé *CustomNamespaces*. Supposons maintenant que vous développiez une collection de classes géométriques nommées `Square`, `Circle` et `Hexagon`. Compte tenu de leurs similitudes, vous souhaitez les regrouper dans un espace de noms unique appelé `CustomNamespaces.MyShapes` au sein de l'assembly *CustomNamespaces.dll*.

>[!tip] Bonne pratique
>Vous êtes libre d'utiliser le nom de votre choix pour vos espaces de noms, mais la convention d'appellation est généralement similaire à : `NomDeLaSociété.NomDuProduit.NomDeLAssembly.Chemin`.
Bien que le compilateur C# ne rencontre aucun problème avec un seul fichier de code C# contenant plusieurs types,

**Bien que le compilateur C# ne rencontre aucun problème avec un seul fichier de code C# contenant plusieurs types, cela peut s'avérer problématique dans un environnement de travail collaboratif**. *==Si vous travaillez sur la classe `Circle` et que votre collègue doit travailler sur la classe `Hexagon`, vous devrez travailler à tour de rôle sur le fichier monolithique, sous peine de faire face à des conflits de fusion==* (comme un `git merge` par exemple) *==difficiles à résoudre==* (ou du moins chronophages). 

**==Une meilleure approche consiste à placer chaque classe dans son propre fichier, avec une définition d'espace de noms==**. ***==Pour garantir que chaque type est regroupé dans le même ensemble logique, il suffit d'encapsuler les définitions de classe dans le même espace de noms==***. ==L'exemple suivant utilise la syntaxe des espaces de noms *antérieure à C# 10*, où chaque déclaration d'espace de noms encadrait son contenu par une accolade ouvrante et fermante, comme ceci== :

```cs
// Circle.cs
namespace CustomNamespaces.MyShapes
{
	// Classe Circle 
	public class Circle { /* Méthodes intéressantes... */ 
	}
}
```

```cs
// Hexagon.cs
namespace CustomNamespaces.MyShapes
{
	// Classe Hexagon 
	public class Hexagon { /* Des méthodes plus intéressantes... */ 
	}
}
```

```cs
// Square.cs
namespace CustomNamespaces.MyShapes
{
	// Classe Square 
	public class Square { /* Des méthodes encore plus intéressantes...*/
	}
}
```

**L'une des nouveautés de C# 10 est l'ajout des espaces de noms de portée fichier**. **==Cela élimine le besoin d'utiliser des accolades ouvrantes et fermantes pour encadrer le contenu==**. **Il suffit de déclarer l'espace de noms, et tout ce qui suit cette déclaration est inclus dans l'espace de noms**. L'exemple de code suivant produit le même résultat que l'exemple avec les espaces de noms encadrant leur contenu :

```cs
// Circle.cs
namespace CustomNamespaces.MyShapes;

// classe Circle
public class Circle { /* Des méthodes intéressantes... */
}
```

```cs
// Hexagon.cs
namespace CustomNamespaces.MyShapes;

// classe Hexagon
public class Hexagon { /* Des méthodes plus intéressantes... */
}
```

```cs
// Square.cs
namespace CustomNamespaces.MyShapes;

// Classe Square
public class Square { /* Méthode encore plus intéressante... */
}
```

**==Cette modification apportée par l'équipe C# (ainsi que les instructions `using` globales et implicites) a permis d'éliminer une grande partie du code répétitif nécessaire dans les versions de C# antérieures à C# 10==**. En fait, lors de la mise à jour du livre pour cette version, j'ai pu supprimer en moyenne deux pages par chapitre, simplement en supprimant les instructions `using` et les accolades (désormais) inutiles !

**Notez comment l’espace de noms `CustomNamespaces.MyShapes` sert de « conteneur » conceptuel à ces classes**. **==Lorsqu’un autre espace de noms==** (tel que `CustomNamespaces`) **==souhaite utiliser des types situés dans un espace de noms distinct, vous utilisez le mot-clé `using`, comme vous le feriez avec les espaces de noms des bibliothèques de classes de base .NET==**, comme suit :

```cs
// Utilise les types définis dans l'espace de noms MyShapes.
using CustomNamespaces.MyShapes;

Hexagon h = new();
Circle c = new();
Square s = new();
```

**Dans cet exemple, on suppose que les fichiers C# définissant l'espace de noms `CustomNamespaces.MyShapes` font partie du même projet d'application console; autrement dit, tous les fichiers sont compilés en un seul assembly**. ***==Si vous aviez défini l'espace de noms `CustomNamespaces.MyShapes` dans un assembly externe, vous auriez également dû ajouter une référence à cette bibliothèque pour que la compilation réussisse==***. Vous découvrirez tous les détails de la création d'applications utilisant des bibliothèques externes dans ce chapitre.

## Résolution des conflits de noms avec les noms pleinement qualifiés

***==Techniquement, l'utilisation du mot-clé `using` en C# n'est pas obligatoire pour faire référence à des types définis dans des espaces de noms externes==***. Vous pouvez utiliser le nom pleinement qualifié du type, qui, comme vous vous en souvenez peut-être du [[Chapitre 1#Accéder à un espace de noms par programmation|Chapitre 1]], est le nom du type préfixé par l'espace de noms qui le définit. Voici un exemple :

```cs
// Notez que l'on n'importe plus CustomNamespaces.MyShapes !
CustomNamespaces.MyShapes.Hexagon h = new CustomNamespaces.MyShapes.Hexagon();
CustomNamespaces.MyShapes.Circle c = new CustomNamespaces.MyShapes.Circle();
CustomNamespaces.MyShapes.Square s = new CustomNamespaces.MyShapes.Square();
```

**En règle générale, il n'est pas nécessaire d'utiliser un nom pleinement qualifié**. Non seulement cela exige plus de frappes, mais ***==cela ne change absolument rien à la taille du code ni à la vitesse d'exécution. En fait, en CIL, les types sont toujours définis avec leur nom pleinement qualifié==***. Dans cette optique, le mot-clé `using` de C# est simplement un gain de temps de saisie.

**Cependant, les noms pleinement qualifiés peuvent être utiles** (**==et parfois nécessaires==**) **pour éviter d'éventuels conflits de noms lors de l'utilisation de plusieurs espaces de noms contenant des types portant le même nom**. ==Supposons que vous ayez un nouvel espace de noms nommé `CustomNamespaces.My3DShapes`, qui définit les trois classes suivantes==, capables de rendre une forme en 3D époustouflante :

```cs
//Circle.cs
namespace CustomNamespaces.My3DShapes;
// 3D Circle class.
public class Circle { }
```

```cs
//Hexagon.cs
namespace CustomNamespaces.My3DShapes;
// 3D Hexagon class.
public class Hexagon { }
```

```cs
//Square.cs
namespace CustomNamespaces.My3DShapes;
// 3D Square class.
public class Square { }
```

==Si vous mettez à jour les instructions de niveau supérieur comme indiqué ci-dessous, plusieurs erreurs de compilation se produisent, **car les deux espaces de noms définissent des classes portant le même nom :**==

```cs
// Utilise les types définis dans l'espace de noms MyShapes.
using CustomNamespaces.My3DShapes;
using CustomNamespaces.MyShapes;

Hexagon h = new(); // Erreur de compilation
Circle c = new();  // Erreur de compilation
Square s = new();  // Erreur de compilation
```

**L’ambiguïté peut être résolue en utilisant le nom pleinement qualifié du type**, comme ceci :

```cs
// On a maintenant résolu l'ambiguité.
CustomNamespaces.My3DShapes.Hexagon h = new();
CustomNamespaces.My3DShapes.Circle c = new();
CustomNamespaces.MyShapes.Square s = new();
```

## Résolution des conflits de noms avec des alias

>[!tip] comme `as` dans une déclaration `include` en Python


**Le mot-clé `using` en C# permet également de créer un alias pour le nom complet d'un type**. ==Ce faisant, vous définissez un jeton qui remplace le nom complet du type lors de la compilation==. La définition d'alias offre une seconde méthode pour résoudre les conflits de noms. Voici un exemple :

```cs
using CustomNamespaces.MyShapes;
using CustomNamespaces.My3DShapes;

// Résout l'ambiguïté d'un type à l'aide d'un alias personnalisé.
using The3DHexagon = My3DShapes.Hexagon;

// Ceci crée en réalité une classe My3DShapes.Hexagon
The3DHexagon h1 = new The3DHexagon();
```

Il existe une autre syntaxe (**==plus courante==**) qui permet de créer un alias pour un espace de noms
au lieu d'un type. Par exemple, vous pouvez créer un alias pour l'espace de noms `CustomNamespaces.My3DShapes` et créer une instance de l'hexagone 3D comme suit :

```cs
using ThreeD = CustomNamespaces.My3DShapes;
ThreeD.Hexagon h2 = new ThreeD.Hexagon();
```

>[!note]
>Sachez que l'utilisation excessive d'alias C# pour les types peut rendre le code source confus. Si les autres programmeurs de votre équipe ignorent l'existence de ces alias personnalisés, ils pourraient avoir des difficultés à localiser les types réels dans le(s) projet(s).

## Création d'espaces de noms imbriqués

**Lors de l'organisation de vos types, vous pouvez définir des espaces de noms à l'intérieur d'autres espaces de noms**. **==Les classes de base le font à de nombreux endroits pour offrir des niveaux d'organisation des types plus profonds==**. ==Par exemple, l'espace de noms `IO` est imbriqué dans `System` pour donner `System.IO`==. ***==En fait, vous avez déjà créé des espaces de noms imbriqués dans l'exemple précédent==***. **Les espaces de noms multiparties (`CustomNamespaces.MyShapes` et `CustomNamespaces.My3DShapes`) sont imbriqués sous l'espace de noms racine, `CustomNamespaces`.**

>[!tip] 
>Déclarer `A.B` crée automatiquement `A` comme parent aux yeux du système, même si `A` n'est défini seul dans aucun fichier. Le point (`.`) crée la hiérarchie automatiquement dans l'esprit du compilateur.

**Comme vous l'avez déjà vu tout au long de ce livre, les modèles de projet .NET ajoutent le code initial des applications console dans un fichier nommé** *Program.cs*. Ce fichier contient un espace de noms portant le nom du projet et une seule classe, `Program`. **Cet espace de noms de base est appelé l'espace de noms *racine***. ***==Dans notre exemple actuel, l'espace de noms racine créé par le modèle .NET est `CustomNamespaces`==***. **Pour imbriquer les espaces de noms `MyShapes` et `My3DShapes` dans l'espace de noms racine, il existe trois options. La première consiste simplement à imbriquer le mot-clé namespace, comme ceci**(en utilisant la syntaxe antérieure à C# 10) :

```cs
namespace CustomNamespaces
{
	namespace MyShapes
	{
		// Class Circle 
		public class Circle
		{
		/* Methodes intéressantes... */
		}
	}
}
```

La deuxième option, utilisant C# 10 et supérieur utilise un espace de noms de portée fichier suivi d'un espace de noms de portée bloc pour l'espace de noms imbriqué :

>[!danger] L'auteur à fait une erreur: Cette option n'est pas légale en C# !
>```cs
>namespace CustomNamespaces;
>namespace MyShapes
>{
>	// Circle class
>	public class Circle
>	{
>	/* Interesting methods... */
>	}
>}
>```

**La troisième option (et la plus courante) consiste à utiliser la notation pointée dans la définition de l'espace de noms, comme nous l'avons fait dans les exemples de classes précédents** :

```cs
// Circle.cs
namespace CustomNamespaces.MyShapes;

// classe Circle
public class Circle { /*  Méthodes intéressantes... */
}
```

Les espaces de noms n'ont pas besoin de contenir directement des types. Cela permet aux développeurs d'utiliser les espaces de noms pour fournir un niveau de portée supplémentaire.

>[!tip] Bonne pratique
>Il est courant de regrouper les fichiers d'un espace de noms par répertoire. L'emplacement d'un fichier dans l'arborescence des répertoires n'a aucun impact sur les espaces de noms. Cependant, cela rend l'arborescence des espaces de noms plus claire pour les autres développeurs. Par conséquent, de nombreux développeurs et outils d'analyse statique de code s'attendent à ce que les espaces de noms correspondent à l'arborescence des dossiers.

## Modifier l’espace de noms racine avec Visual Studio 2022

Comme indiqué précédemment, ***==lorsque vous créez un projet C#==*** avec Visual Studio (ou l’interface de ligne de commande .NET), ***==le nom de l’espace de noms racine de votre application est identique au nom du projet==***. Désormais, lorsque vous utilisez Visual Studio pour insérer de nouveaux fichiers de code via le menu Projet > Ajouter un nouvel élément, les types sont automatiquement inclus dans l’espace de noms racine et le chemin d’accès au répertoire est ajouté. Si vous souhaitez modifier le nom de l’espace de noms racine, accédez simplement à l’option « Espace de noms par défaut » dans l’onglet Application/Général de la fenêtre des propriétés du projet (voir l'image suivante).

![[Figure 16.1.png|Configurer l'espace de nom racine/par défaut dans Visual Studio]]

>[!note]
>Les pages de propriétés de Visual Studio font toujours référence à l'espace de noms racine comme étant l'espace de noms *par défaut*. Vous verrez ensuite pourquoi je le désigne comme l'espace de noms *racine*.

## Modifier l'espace de noms racine à l'aide du fichier projet

Si vous n'utilisez pas Visual Studio (ou même avec Visual Studio), **vous pouvez également configurer l'espace de noms racine en modifiant le fichier projet** (*.csproj*). Avec les projets .NET, la modification du fichier projet dans Visual Studio est aussi simple que un double-clic sur le fichier projet dans l'Explorateur de solutions (ou un clic droit sur le fichier projet dans l'Explorateur de solutions, puis la sélection de « Modifier le fichier projet »). **Une fois le fichier ouvert, mettez à jour le `PropertyGroup` principal en ajoutant le nœud `RootNamespace`**, comme ceci :

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net10.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>disable</Nullable>
    <RootNamespace>CustomNamespace2</RootNamespace>
  </PropertyGroup>

</Project>
```

Jusqu'ici tout va bien. ==Maintenant que vous avez vu quelques détails sur la façon d'organiser vos types personnalisés dans des espaces de noms bien structurés, passons rapidement en revue les avantages et le format de l'assembly .NET==. Ensuite, vous approfondirez la création, le déploiement et la configuration de vos bibliothèques de classes personnalisées.

# Le rôle des assemblages .NET

**Les applications .NET sont construites en assemblant un nombre quelconque d'assemblies**. **==En termes simples, un assembly est un fichier binaire versionné et auto-descriptif hébergé par le runtime .NET==**. ***==Bien que les assembies .NET possèdent les mêmes extensions de fichier==*** (*.exe* ou *.dll*) ***==que les anciens binaires Windows, leur fonctionnement interne diffère sensiblement de celui de ces fichiers==***. Avant d'approfondir ce point, **examinons quelques-uns des avantages offerts par le format assembly.**

## Les assemblages favorisent la réutilisation du code (Interopérabilité)

Au fil des chapitres précédents, lors de la création de vos projets d'application console, ==il pouvait sembler que toutes les fonctionnalités de l'application étaient contenues dans l'assembly exécutable que vous construisiez==. Vos applications exploitaient de nombreux types contenus dans les bibliothèques de classes de base .NET, toujours accessibles.

**Comme vous le savez peut-être, une** *bibliothèque de code* (également appelée bibliothèque de classes) **est un fichier** *.dll* **contenant des types destinés à être utilisés par des applications externes**. **==Lorsque vous créez des assemblies exécutables, vous exploitez sans aucun doute de nombreuses bibliothèques de code système et personnalisées==**. **Sachez toutefois qu'une bibliothèque de code n'a pas nécessairement l'extension** *.dll*. ***==Il est tout à fait possible (bien que rare) qu'un assembly exécutable utilise des types définis dans un fichier exécutable externe==***. Dans cette optique, **un fichier** *.exe* **référencé peut également être considéré comme une bibliothèque de code.**

**Quelle que soit la manière dont une bibliothèque de code est empaquetée, la plateforme .NET permet de réutiliser les types de façon indépendante du langage**. Par exemple, ==vous pouvez créer une bibliothèque de code en C# et la réutiliser dans n'importe quel autre langage de programmation .NET==. Il est possible non seulement d'allouer des types entre les langages, mais aussi d'en hériter. ==Une classe de base définie en C# peut être étendue par une classe écrite en Visual Basic. Les interfaces définies en F# peuvent être implémentées par des structures définies en C#, et ainsi de suite==. L'idée est que lorsque vous commencez à décomposer un exécutable monolithique en de nombreux assemblies .NET, vous obtenez une forme de réutilisation du code indépendante du langage.

## Les assemblages définissent une limite de type

**Rappelons que le *nom pleinement qualifié* d'un type est composé du préfixe de son espace de noms** (par exemple, `System`) **à son nom** (par exemple, `Console`). **==À proprement parler, cependant, l'assembly dans lequel réside un type contribue à définir son identité==**. Par exemple, **si vous avez deux assemblies aux noms uniques** (disons, *MyCars.dll* et *YourCars.dll*) **qui définissent tous deux un espace de noms (`CarLibrary`) contenant une classe nommée `SportsCar`, ils sont considérés comme des types uniques dans l'univers .NET.**

## Les assemblages sont des unités versionnables

**Les assemblies .NET se voient attribuer un numéro de version numérique en quatre parties, au format** *`<major>.<minor>.<build>.<revision>`*. (***==Si vous ne spécifiez pas explicitement de numéro de version, l'assembly se voit automatiquement attribuer la version 1.0.0.0, conformément aux paramètres par défaut des projets .NET.==***) **Ce numéro permet à plusieurs versions du même assembly de coexister harmonieusement sur une même machine.**

## Les assemblages sont auto-descriptifs.

**Les assemblies sont considérés comme auto-descriptifs, notamment parce qu'ils enregistrent dans leur manifeste tous les assemblies externes auxquels ils doivent pouvoir accéder pour fonctionner correctement**. Rappelons-nous du [[Chapitre 1#Le rôle du manifeste d'assembly|Chapitre 1]] qu'***==un manifeste est un ensemble de métadonnées décrivant l'assembly lui-même==*** (nom, version, assemblies externes requis, etc.).

Outre les données du manifeste, **un assembly contient des métadonnées décrivant la composition** (noms des membres, interfaces implémentées, classes de base, constructeurs, etc.) **de chaque type contenu**. **==Grâce à cette documentation détaillée, le runtime .NET ne consulte pas le registre du système pour déterminer son emplacement==** (une rupture radicale avec le modèle de programmation COM traditionnel de Microsoft). **Cette séparation du registre est l'un des facteurs qui permettent aux applications .NET de s'exécuter sur d'autres systèmes d'exploitation que Windows, et de prendre en charge plusieurs versions de .NET sur la même machine.**

Comme vous le découvrirez dans ce chapitre, ***==le runtime .NET utilise un système entièrement nouveau pour déterminer l'emplacement des bibliothèques de code externes.==***

# Comprendre le format d'un assembly .NET

Maintenant que vous avez découvert plusieurs avantages offerts par les assemblies .NET, intéressons-nous à leur fonctionnement interne. **D'un point de vue structurel, un assembly .NET** (*.dll* ou *.exe*) **se compose des éléments suivants** :

- En-tête de fichier du système d'exploitation (par exemple, Windows)
-  En-tête de fichier CLR
- Code CIL
- Métadonnées de type
- Manifeste d'assembly
- Ressources incorporées facultatives

Si les deux premiers éléments** (en-têtes du système d'exploitation et du CLR) **sont des blocs de données que vous pouvez généralement ignorer, ils méritent néanmoins un bref examen. Voici une présentation de chaque élément.

## Installation des outils de profilage C++

>[!tip] Toute les informations extraites montrée par l'auteur sont possible avec *ildasm*
>**Ce que `dumpbin` fait (et pas `ildasm`)** :
>
>- **Inspection du code machine pur** : Si on compile du C++ en code assembleur natif (x64), `dumpbin` peut le lire. `ildasm` ne comprend que le CIL (MSIL).
>- **Détails du Linker** : Il montre comment les sections du fichier sont alignées sur le disque pour Windows.
>- **Imports natifs** : Il montre quelles DLL système Windows (comme `kernel32.dll`) sont appelées directement par l'enveloppe du fichier.
>
>**Tout ce que `dumpbin` fait sur la structure du fichier, la commande `objdump` (Disponible avec les *Command Line Tools* sur macOS) le fait aussi, et souvent mieux.**

Les sections suivantes utilisent l'utilitaire *dumpbin.exe*, fourni avec les outils de profilage C++. Pour les installer, saisissez « outils de profilage C++ » dans la barre de recherche rapide, puis cliquez sur l'invite d'installation, comme illustré dans l'image suivante

![[Figure 16.2.png|Installer les outils de profilage C++ dans Lancement Rapide]]

Cela ouvrira le programme d'installation de Visual Studio avec les outils sélectionnés. Vous pouvez également lancer manuellement le programme d'installation de Visual Studio et sélectionner les composants illustrés dans l'image suivante.

![[Figure 16.3.png|Installer les outils de profilage C++]]

## En-tête du fichier système d'exploitation

L'en-tête du fichier système d'exploitation indique que l'assembly peut être chargé et manipulé par
le système d'exploitation cible (ici, Windows). Ces données d'en-tête identifient également le type d'
application (console, interface graphique ou bibliothèque de code *.dll*) hébergée par le système d'exploitation.

Ouvrez le fichier *CarLibrary.dll* (==dans le dépôt du livre ou créé plus loin dans ce chapitre==) à l'aide de l'utilitaire *dumpbin.exe*/*objdump* (via le terminal) avec l'option `headers` :

**Windows**

```cmd
dumpbin /headers CarLibrary.dll
```

**MacOS**

```bash
objdump -h CarLibrary.dll
```

Ceci affiche les informations d'en-tête du système d'exploitation de l'assembly (présentées ci-dessous lors de la compilation pour Windows). Voici les informations d'en-tête Windows (partielles) pour *CarLibrary.dll* :

>Le résultat est très similaire avec `objdump`

```
Dump of file carlibrary.dll
PE signature found
File Type: DLL

FILE HEADER VALUES
		14C machine (x86)
		3 number of sections
	877429B3 time date stamp
		0 file pointer to symbol table
		0 number of symbols
		E0 size of optional header
		2022 characteristics
			Executable
			Application can handle large (>2GB) addresses
			DLL
...
```

**Rappelez-vous que la plupart des programmeurs .NET n'auront jamais à se soucier du format des données d'en-tête intégrées à un assembly .NET**. ***==À moins que vous ne développiez un nouveau compilateur pour le langage .NET==*** (auquel cas ces informations vous seraient utiles), ***==vous pouvez ignorer les détails techniques des données d'en-tête==***. **Sachez toutefois que ces informations sont utilisées en interne lorsque le système d'exploitation charge l'image binaire en mémoire.**

## En-tête CLR

>[!important] C'est ce qui différencie un binaire natif (C/C++) d'un binaire managé (.NET)

***==L'en-tête CLR est un bloc de données que tous les assemblies .NET doivent prendre en charge==*** (et prennent en charge, grâce au compilateur C#) ***==pour être hébergés par le runtime .NET==***. **En résumé, cet en-tête définit de nombreux indicateurs permettant au runtime de comprendre la structure du fichier managé**. Par exemple, ==des indicateurs permettent d'identifier l'emplacement des métadonnées et des ressources dans le fichier, la version du runtime utilisée pour la compilation de l'assembly, la valeur de la clé publique== (facultative), etc. Exécutez *dumpbin.exe* avec l'option `/clrheaders`.

>[!info] Ici, j'affiche le résultat de `objdump -p`

```
...

The Data Directory
Entry 0 00000000 00000000 Export Directory [.edata (or where ever we found it)]
Entry 1 00002908 0000004f Import Directory [parts of .idata]
Entry 2 00004000 00000610 Resource Directory [.rsrc]
Entry 3 00000000 00000000 Exception Directory [.pdata]
Entry 4 00000000 00000000 Security Directory
Entry 5 00006000 0000000c Base Relocation Directory [.reloc]
Entry 6 00002814 00000054 Debug Directory
Entry 7 00000000 00000000 Description Directory
Entry 8 00000000 00000000 Special Directory
Entry 9 00000000 00000000 Thread Storage Directory [.tls]
Entry a 00000000 00000000 Load Configuration Directory
Entry b 00000000 00000000 Bound Import Directory
Entry c 00002000 00000008 Import Address Table Directory
Entry d 00000000 00000000 Delay Import Directory
Entry e 00002008 00000048 CLR Runtime Header
Entry f 00000000 00000000 Reserved

The Import Tables:
  lookup 00002930 time 00000000 fwd 00000000 name 0000294a addr 00002000
  
...
```

***==Encore une fois, en tant que développeur .NET, vous n'aurez pas à vous soucier des détails techniques des informations d'en-tête CLR d'un assembly==***. **Retenez simplement que chaque assembly .NET contient ces données, utilisées en arrière-plan par le runtime .NET lors du chargement des données d'image en mémoire**. Concentrez-vous maintenant sur des informations bien plus utiles pour vos tâches de programmation quotidiennes.

## Code CIL, métadonnées de type et manifeste d'assembly

**Un assembly contient du code CIL, un langage intermédiaire indépendant de la plateforme et du processeur**. À l'exécution, **==le CIL interne est compilé à la volée à l'aide d'un compilateur JIT (Just-In-Time), selon les instructions spécifiques à la plateforme et au processeur**==. **Grâce à cette conception, les assemblies .NET peuvent s'exécuter sur diverses architectures, appareils et systèmes d'exploitation**. (Bien qu'il soit possible de mener une vie productive sans maîtriser les détails du langage de programmation CIL, le [[Chapitre 18|Chapitre 18]] propose une introduction à sa syntaxe et à sa sémantique.)

**Un assembly contient également des métadonnées qui décrivent entièrement le format des types qu'il contient, ainsi que celui des types externes référencés par cet assembly**. ==Le runtime .NET utilise ces métadonnées pour localiser les types== (et leurs membres) ==dans le binaire, les organiser en mémoire et faciliter les appels de méthodes distantes==. ***==Vous découvrirez les détails du format des métadonnées .NET au [[Chapitre 17|Chapitre 17]] lors de votre étude des services de réflexion.==***

**Un assembly doit également contenir un manifeste associé (également appelé métadonnées d'assembly)**. **==Le manifeste documente chaque module de l'assembly, établit sa version et documente les assemblies externes référencés par l'assembly courant==**. **Comme vous le verrez tout au long de ce chapitre, le CLR utilise largement le manifeste d'un assembly lors de la localisation des références à des assemblies externes.**

## Ressources d'assemblage optionnelles

**Enfin, un assembly .NET peut contenir un nombre quelconque de ressources incorporées, telles que des icônes d'application, des fichiers image, des clips audio ou des tables de chaînes**. En fait, ***==la plateforme .NET prend en charge les assemblies satellites qui ne contiennent que des ressources localisées==***. **==Cela peut s'avérer utile si vous souhaitez segmenter vos ressources en fonction d'une culture spécifique==** (anglais, allemand, etc.) **==afin de développer des logiciels internationaux==**. *==La création d'assemblies satellites n'est pas abordée dans ce document==*. Consultez la [documentation .NET](https://learn.microsoft.com/en-us/dotnet/core/extensions/create-satellite-assemblies) pour plus d'informations sur les assemblies satellites et la localisation si cela vous intéresse.

# Bibliothèques de classes vs. Applications console

**Jusqu'à présent, les exemples de ce livre étaient presque exclusivement des applications console .NET.** Si vous êtes développeur .NET Framework, ces exemples sont similaires aux applications console .NET, la principale différence résidant dans le processus de configuration (qui sera abordé ultérieurement) et, bien sûr, dans le fait qu'elles s'exécutent sur .NET Core. **Les applications console possèdent un point d'entrée unique (une méthode `Main()` spécifique ou des instructions de niveau supérieur), peuvent interagir avec la console et peuvent être lancées directement depuis le système d'exploitation**. ==Autre différence: les applications console .NET sont lancées à l'aide de l'hôte d'applications .NET (*dotnet.exe / dotnet*).

**Les bibliothèques de classes, quant à elles, n'ont pas de point d'entrée et ne peuvent donc pas être lancées directement**. ***==Elles servent à encapsuler la logique, les types personnalisés, etc., et sont référencées par d'autres bibliothèques de classes et/ou applications console==***. Autrement dit, **les bibliothèques de classes servent à contenir les éléments abordés dans la section  "[[#Le rôle des assemblages .NET]]".**

# Bibliothèques de classes .NET Standard vs. .NET (Core)

***==Les bibliothèques de classes .NET (y compris .NET Core/.NET 5/.NET 6) s'exécutent sur .NET Core==***, *==et les bibliothèques de classes .NET Framework s'exécutent sur .NET Framework==*. **Bien que cela soit assez simple, un problème se pose**. Imaginez que votre organisation dispose d'une importante base de code .NET Framework, fruit de plusieurs années de développement (potentiellement) de votre équipe. Il existe probablement une quantité importante de code partagé, utilisé par les applications que vous avez développées au fil des ans. Il peut s'agir de la journalisation centralisée, de la génération de rapports ou de fonctionnalités spécifiques à un domaine.

Vous (et votre organisation) souhaitez maintenant migrer vers le nouveau .NET pour tous les nouveaux développements. Qu'en est-il de tout ce code partagé ? **L'effort de réécriture de tout votre code existant en assemblies .NET 6 pourrait être considérable, et tant que toutes vos applications n'auront pas été migrées vers .NET 6, vous devrez prendre en charge deux versions (une dans .NET Framework et une dans .NET 6)**. *==Cela paralyserait complètement la productivité==*. **==Heureusement, les concepteurs de .NET (Core) ont anticipé ce cas de figure==**. **.NET Standard est un nouveau type de projet de bibliothèque de classes introduit avec .NET Core 1.0 et pouvant être référencé par les applications .NET Framework ainsi que par les applications .NET (Core)**. Cependant, *==avant de vous emballer, sachez qu'il y a un hic avec .NET 5 et .NET 6. Nous y reviendrons plus en détail==*.

**Chaque version de .NET Standard définit un ensemble commun d'API que toutes les versions de .NET doivent prendre en charge** (.NET, .NET Core, Xamarin, etc.) **pour être conformes à la norme**. Par exemple, ==si vous créez une bibliothèque de classes en tant que projet .NET Standard 2.0, elle peut être référencée par .NET 4.61+ et .NET Core 2.0+ (ainsi que par différentes versions de Xamarin, Mono, Universal Windows Platform et Unity).==

**Cela signifie que vous pouvez déplacer le code de vos bibliothèques de classes .NET Framework vers des bibliothèques de classes .NET Standard 2.0, et ces dernières peuvent être partagées par les applications .NET (Core) et .NET Framework.** **==C'est bien mieux que de gérer des copies dupliquées du même code, une pour chaque framework.==**

*==Mais attention ! Chaque version de .NET Standard représente le plus petit dénominateur commun des frameworks qu'elle prend en charge==*. Autrement dit, **plus la version est ancienne, moins vous pouvez faire de choses dans votre bibliothèque de classes**. ==Bien que les projets .NET Framework et .NET 6 puissent référencer une bibliothèque .NET Standard 2.0, vous ne pouvez pas utiliser un nombre important de fonctionnalités C# 8.0 dans une bibliothèque .NET Standard 2.0, ni les nouvelles fonctionnalités de C# 9.0 ou ultérieures. Vous devez utiliser .NET Standard 2.1 pour une prise en charge complète de C# 8.0 et versions ultérieures.== ~~De plus, .NET 4.8 (la dernière version du .NET Framework d'origine) ne prend en charge que .NET Standard 2.0.~~

>[!tip] La [Documentation](https://learn.microsoft.com/en-us/dotnet/standard/net-standard?tabs=net-standard-2-1) de Microsoft est très claire sur le sujet.

**Cela reste un bon moyen d'exploiter du code existant dans des applications plus récentes jusqu'à .NET Core 3.1 inclus, mais ce n'est pas une solution miracle**. ***==Avec l'unification des frameworks==*** (.NET, Xamarin, Mono, etc.) ***==avec .NET 6, l'utilité de .NET Standard diminue progressivement.==***

# Configuration des applications avec des fichiers de configuration

**Bien qu'il soit possible de conserver toutes les informations nécessaires à votre application .NET dans le code source, la possibilité de modifier certaines valeurs à l'exécution est essentielle dans la plupart des applications importantes**. **==L'une des options les plus courantes consiste à utiliser un ou plusieurs fichiers de configuration fournis==** (ou déployés) **==avec l'exécutable de votre application.==**

==Le framework .NET s'appuyait principalement sur des fichiers XML nommés *app.config* (ou *web.config* pour les applications ASP.NET) pour la configuration==. **Bien que les fichiers de configuration XML puissent toujours être utilisés, la méthode la plus courante pour configurer les applications .NET est l'utilisation de fichiers JSON** (JavaScript Object Notation).

>[!warning] 
>Dans .NET 10, le support du XML est devenu une option "héritée" (legacy). Pour un projet **Avalonia** ou **Console** moderne sur macOS, vous ne devriez même pas voir la trace d'un fichier `.config`.

>[!note]
>Si vous n'êtes pas familier avec JSON, il s'agit d'un format de paires nom-valeur où chaque objet est encadré par des accolades. Les valeurs peuvent également être des objets utilisant le même format de paires nom-valeur. Le [[Chapitre 20|Chapitre 20]] traite en détail de la manipulation des fichiers JSON.

Pour illustrer le processus, créez une nouvelle application console .NET nommée *FunWithConfiguration* et ajoutez la référence de package suivante à votre projet :

```cs
dotnet add FunWithConfiguration package Microsoft.Extensions.Configuration
dotnet add FunWithConfiguration package Microsoft.Extensions.Configuration.Binder
dotnet add FunWithConfiguration package Microsoft.Extensions.Configuration.Json
```

**Cela ajoute une référence au sous-système de configuration, au sous-système de configuration .NET basé sur un fichier JSON, et aux extensions de liaison pour la configuration dans votre projet**. Commencez par ajouter un nouveau fichier JSON à votre projet, nommé *appsettings.json*. Mettez à jour le fichier projet pour vous assurer que le fichier est toujours copié dans le répertoire de sortie lors de la compilation du projet.

>[!info]- Explication plus détaillées
>Dans notre code, on va utiliser l'interface `IConfiguration`. Cette interface fait partie de la BCL (Librairie de Classe de Base), mais son implémentation spécifique (comme la lecture du JSON) nécessite des packages externes. l'interface est présente dans le SDK, mais elle est  "vide" : elle définit comment accéder aux données, mais pas où les trouver. 
>
Les packages que l'auteur fait ajouter sont les **implémentations**
>
>- **`Microsoft.Extensions.Configuration.Json`** : C'est le "moteur" qui sait comment ouvrir un fichier `.json` et le transformer en dictionnaire pour `IConfiguration`.
>- **`Microsoft.Extensions.Configuration.Binder`** : C'est l'outil qui permet de mapper les données du JSON vers des objets C# (classes).
>

```xml
<ItemGroup>
	<None Update="appsettings.json">
		<CopyToOutputDirectory>Always</CopyToOutputDirectory>
	</None>
</ItemGroup>
```

>[!tip] Bonnes pratiques
>Il est préférable d'utiliser `PreserveNewest` comme valeur au lieu de `Always`.
>
>>[!info]-
>>Avec `Always`, Chaque fois que l'on compile ou lance une application, .NET copie le fichier `appsettings.json` dans le dossier `bin/Debug/...`. 
>>
>>Il écrase le fichier de destination même si rien n'a changé. Cela force parfois le système à considérer que le projet a été modifié, ce qui peut ralentir légèrement les compilations répétitives (surtout sur de gros projets). De plus. Si on lance une  application en mode "Watch" (avec `dotnet watch`), l'option `Always` peut parfois provoquer une boucle infinie
>> 
>>Avec `PreserveNewest`, NET compare la date de modification. Il ne copie le fichier dans le dossier de sortie **que si vous l'avez modifié** depuis la dernière compilation.
>>
>>C'est plus "intelligent" et plus rapide. C'est le standard utilisé par la plupart des développeurs expérimentés pour éviter des copies inutiles.

Enfin, mettez à jour le fichier *appsettings.json* pour qu'il corresponde à ce qui suit :

```json
{
  "CarName":  "Suzy"
}
```

**La dernière étape pour ajouter la configuration à votre application consiste à lire le fichier de configuration et à récupérer la valeur de `CarName`**. Mettez à jour le fichier *Program.cs* comme suit :

```cs
using Microsoft.Extensions.Configuration;

IConfiguration config = new ConfigurationBuilder()
    .SetBasePath(Directory.GetCurrentDirectory())
    .AddJsonFile("appsettings.json", true, true)
    .Build();
```

>[!warning] Pour ce projet, **il faut absolument se trouver dans le dossier racine du projet** (pas le dossier où se trouve tout les projets du livre) **pour que le programme fonctionne** (donne les résultat souhaitable).

**Le nouveau système de configuration commence par un `ConfigurationBuilder`. Le chemin d'accès à partir duquel la configuration recherchera les fichiers ajoutés est défini avec la méthode `SetBasePath()`**. Ensuite, **==le fichier de configuration est ajouté avec la méthode `AddJsonFile()`, qui prend trois paramètres==**. ==Le premier paramètre est le chemin d'accès et le nom du fichier==. Comme ce fichier se trouve au même emplacement que le chemin de base, la chaîne ne contient aucune information de chemin d'accès, seulement le nom du fichier. ==Le deuxième paramètre indique si le fichier est optionnel (`true`) ou obligatoire (`false`), et le dernier paramètre détermine si la configuration doit utiliser un observateur de fichiers pour détecter les modifications du fichier (`true`) ou ignorer les modifications pendant l'exécution (`false`)==. **La dernière étape consiste à créer une instance de `IConfiguration` à partir de la configuration à l'aide de la méthode `Build()`. Cette instance donne accès à toutes les valeurs configurées.**

>[!note]
>Les observateurs de fichiers sont traités au [[Chapitre 20|Chapitre 20]].

Une fois que vous disposez d'une instance de `IConfiguration`, vous pouvez récupérer les valeurs des fichiers de configuration de la même manière qu'en appelant le `ConfigurationManager` dans .NET 4.8. Ajoutez le code suivant à la fin de la méthode `Main()`, et lorsque vous exécuterez l'application, la valeur s'affichera dans la console :

```cs
Console.WriteLine($"My car's name is {config["CarName"]}");
```

**Si le nom de la requête n'existe pas dans la configuration, le résultat sera `null`**. **==Le code suivant s'exécute toujours sans exception ; il n'affiche simplement pas de nom sur la première ligne et affiche "True" sur la deuxième ligne :==**

```cs
Console.WriteLine($"My car's name is {config["CarName2"]}");
Console.WriteLine($"CarName2 is null? {config["CarName2"] == null}");
```


**Il existe également une méthode `GetValue()` (et sa version générique `GetValue<T>()`) qui permet de récupérer des valeurs primitives à partir de la configuration**. L'exemple suivant illustre ces deux méthodes pour obtenir le `CarName` :

```cs
Console.WriteLine($"My car's name is {config.GetValue(typeof(string), "CarName")}");
Console.WriteLine($"My car's name is {config.GetValue<string>("CarName")}");
```

**Ces méthodes renvoient la valeur par défaut** (par exemple, `null` pour les types référence, $0$ pour les types numériques) **si le nom demandé n'existe pas**. Le code suivant renvoie $0$ pour la propriété `CarName2` :

```cs
Console.WriteLine($"My car's name is {config.GetValue<int>("CarName2")}");
```

***==Ces méthodes lèveront une exception si la valeur trouvée pour le nom ne peut pas être convertie intrinsèquement au type de données demandé==***. Par exemple, tenter de convertir la propriété `CarName` en `int` lèvera une `InvalidOperationException`, comme le montre le code suivant :

```cs
try
{
    Console.WriteLine($"My car's name is {config.GetValue<int>("CarName")}");
}
catch (InvalidOperationException ex)
{
    Console.WriteLine($"An exception occured: {ex.Message}");
}
```

>[!note]
>La méthode GetValue() est conçue pour fonctionner avec des types primitifs. Pour les types complexes, utilisez les méthodes `Bind()` ou `Get()`/`Get<T>()`, décrites dans la section "[[#Utilisation des objets (MaJ C 10.0)|Utilisation des objets]]"

## Plusieurs fichiers de configuration

**Il est possible d'ajouter plusieurs fichiers de configuration au système**. ==Lorsque plusieurs fichiers sont utilisés, leurs propriétés s'additionnent, sauf en cas de conflit entre les noms des paires nom-valeur==. ***==En cas de conflit, le dernier fichier ajouté prévaut==***. Pour observer ce comportement, ajoutez un fichier nommé *appsettings.development.json* et configurez le projet pour qu'il le copie systématiquement (à chaque changement fait dans le fichier, voir note sur le sujet) dans le répertoire de sortie.

```xml
<ItemGroup>
  <None Update="appsettings.development.json">
	<CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
  </None>
</ItemGroup>
```

Mettez à jour le JSON comme suit :

```json
{
  "CarName": "Logan"
}
```

Mettez maintenant à jour le code qui crée l'instance de l'interface `IConfiguration` comme suit :

```cs
IConfiguration config = new ConfigurationBuilder()
    .SetBasePath(Directory.GetCurrentDirectory())
    .AddJsonFile("appsettings.json", true, true)
    .AddJsonFile("appsettings.development.json", true, true)
    .Build();
```

Lorsque vous exécuterez ce programme, **==vous constaterez que le nom de la voiture est bien `"Logan"`, et non`" Suzy"`.==**

## Utilisation des objets (MaJ C# 10.0)

Notre fichier JSON d'exemple est pour l'instant extrêmement simple et ne contient qu'une seule paire nom-valeur. **Dans les projets réels, la configuration d'une application est généralement plus complexe qu'une simple propriété**. Mettez à jour le fichier *appsettings.development.json* comme suit, ce qui ajoute un nouvel objet `Car` au JSON :

```json
{
  "CarName": "Logan",
  "Car": {
    "Make": "Honda",
    "Color": "Blue",
    "PetName": "Dad's Taxi"
  }
}
```

**Pour accéder aux valeurs JSON à plusieurs niveaux, la clé de recherche est la hiérarchie du JSON, chaque niveau étant séparé par des deux-points (`:`)**. **==Par exemple, la clé de la propriété `Make` de l'objet `Car` est `Car:Make`==**. Mettez à jour les instructions de niveau supérieur comme suit pour obtenir et afficher toutes les propriétés de `Car` :

```cs
Console.Write($"My car object is a {config["Car:Color"]} ");
Console.WriteLine($"{config["Car:Make"]} named {config["Car:PetName"]}");
```

**Au lieu de parcourir l'arborescence des noms, il est possible de récupérer des sections entières à l'aide de la méthode `GetSection()`**. ==Une fois la section obtenue, vous pouvez accéder à ses valeurs en utilisant le format de nom simple==, comme illustré dans l'exemple suivant :

```cs
IConfigurationSection section = config.GetSection("Car");
Console.Write($"My car object is a {section["Color"]} ");
Console.WriteLine($"{section["Make"]} named {section["PetName"]}");
```

Pour conclure, **concernant la manipulation d'objets, vous pouvez utiliser la méthode `Bind()` pour lier des valeurs de configuration à une instance existante d'un objet ou la méthode `Get()` pour créer une nouvelle instance d'objet**. ***==Ces méthodes sont similaires à `GetValue()` mais fonctionnent avec des types non primitifs==***. Pour commencer, créez une classe `Car` simple à la fin des instructions de niveau supérieur :

```cs
public class Car
{
    public string Make { get; set; }
    public string Color { get; set; }
    public string PetName { get; set; }
}
```

Ensuite, créez une nouvelle instance de la classe `Car`, puis appelez `Bind()` sur la section en passant l'instance `Car` :

```cs
var c = new Car();
section.Bind(c);
Console.Write($"My car object is a {c.Color} ");
Console.WriteLine($"{c.Make} named {c.PetName}");
```

**Si la section n'est pas configurée, la méthode `Bind()` ne mettra pas à jour l'instance mais conservera toutes les propriétés telles qu'elles existaient avant l'appel à `Bind()`**. Le code suivant laissera la couleur définie sur `Red` et les autres propriétés nulles :

```cs
var notFoundCar = new Car { Color = "Red" };
config.GetSection("Car2").Bind(notFoundCar);
Console.Write($"My car object is a {c.Color} ");
Console.WriteLine($"{notFoundCar.Make} named {notFoundCar.PetName}");
```

**La méthode `Get()` crée une nouvelle instance du type spécifié à partir d'une section de la configuration**. ==La version non générique de la méthode renvoie un objet ; il est donc nécessaire de convertir la valeur de retour vers le type spécifique avant de l'utiliser==. Voici un exemple d'utilisation de la méthode `Get()` pour créer une instance de la classe `Car` à partir de la section `Car` de la configuration :

```cs
var carFromGet = config.GetSection(nameof(Car)).Get(typeof(Car)) as Car;
Console.Write($"My car object is a {carFromGet.Color} ");
Console.WriteLine($"{carFromGet.Make} named {carFromGet.PetName}");
```

**Si la section nommée est introuvable, la méthode `Get()` renvoie `null` :**

```cs
var notFoundCarFromGet = config.GetSection("Car2").Get(typeof(Car));
Console.WriteLine($"The not found car is null? {notFoundCarFromGet == null}");
```

**La version générique renvoie une instance du type spécifié sans conversion de type**. Si la section est introuvable, la méthode renvoie `null`.

```cs
// Renvoie une instance de Car
var carFromGet2 = config.GetSection(nameof(Car)).Get<Car>();
Console.Write($"My car object is a {carFromGet2.Color} ");
Console.WriteLine($"{carFromGet2.Make} named {carFromGet2.PetName}");

// Renvoie null
var notFoundCarFromGet2 = config.GetSection("Car2").Get<Car>();
Console.WriteLine($"The not found car is null? {notFoundCarFromGet2 == null}");
```

**Les méthodes `Bind()` et `Get()`/`Get<T>()` utilisent la réflexion** (traitée dans le chapitre suivant) **pour faire correspondre les noms des propriétés publiques de la classe aux noms de la section de configuration, ==sans tenir compte de la casse.==** Par exemple, ***==si vous mettez à jour *appsettings.development.json* comme suit (notez le changement de casse de la propriété `petName`), le code précédent fonctionnera toujours :==***

```json
{
	"CarName": "Suzy",
	"Car": {
		"Make":"Honda",
		"Color": "Blue",
		"petName":"Dad's Taxi" // petName au lieu de PetName
	}
}
```

**Si une propriété de la configuration n'existe pas dans la classe** (ou ***==si son nom est orthographié différemment==***), **alors cette valeur de configuration particulière est ignorée (par défaut)**. Si vous mettez à jour le JSON comme suit, les propriétés `Make` et `Color` sont renseignées, mais la propriété `PetName` de l'objet Car ne l'est pas :

```json
{
  "CarName": "Logan",
  "Car": {
    "Make": "Honda",
    "Color": "Blue",
    "PetNameForCar": "Dad's Taxi"
  }
}
```

**Les méthodes `Bind()`, `Get()` et `Get<T>()` peuvent accepter un objet `Action<BinderOptions>` pour affiner le processus de mise à jour (`Bind()`) ou d'instanciation (`Get()`/`Get<T>()`) des instances de classe.** La classe `BinderOptions` est présentée ici :

```cs
public class BinderOptions
{
	public bool BindNonPublicProperties { get; set; } //Defaults to false
	public bool ErrorOnUnknownConfiguration { get; set; } //Defaults to false
}
```

**Si l'option `ErrorOnUnknownConfiguration` est définie sur `true`, une exception `InvalidOperationException` sera levée si la configuration contient un nom inexistant dans le modèle**. Avec la configuration renommée (`PetNameForCar`), l'appel suivant lève l'exception indiquée dans l'exemple de code :

```cs
try
{
    _ = config
        .GetSection(nameof(Car))
        .Get<Car>(t => t.ErrorOnUnknownConfiguration = true);
}
catch (InvalidOperationException ex)
{
    Console.WriteLine($"An exception occured: {ex.Message}");
}
/*
Error message: 'ErrorOnUnknownConfiguration' was set on the provided BinderOptions, but the
following properties were not found on the instance of Car: 'PetNameForCar'
*/
```

**L'autre option permet de lier des propriétés non publiques**. ***==Par défaut, les deux propriétés sont désactivées==***. **Si des propriétés non publiques doivent être liées à partir de la configuration, définissez `BindNonPublicProperties`**, comme ceci :

```cs
var notFoundCarFromGet3 = config
    .GetSection(nameof(Car))
    .Get<Car>(t => t.BindNonPublicProperties = true);
```

**Nouveauté de C# 10 : la méthode `GetRequiredSection()` lève une exception si la section n’est pas configurée**. ==Par exemple, le code suivant lèvera une exception car la section `Car2` est absente de la configuration== :

```cs
try
{
    config.GetRequiredSection("Car2").Bind(notFoundCar);
}
catch (InvalidOperationException ex)
{
    Console.WriteLine($"An exception occured: {ex.Message}");
}
```

>[!info]
>Plus tard, quand vous ferez de l'**ASP.NET Core** ou du **Worker Service**, vous verrez que vous n'aurez plus besoin d'ajouter ces packages manuellement. Ils sont inclus par défaut dans le "Metapackage" du Web.
>
>Mais pour une **Application Console** (comme dans votre exercice actuel), vous devez effectivement les ajouter un par un, car le modèle console est le plus dépouillé possible.

## Options de configuration supplémentaires

**Outre la configuration par fichiers, vous pouvez utiliser des variables d'environnement, Azure Key Vault, des arguments de ligne de commande et bien d'autres options**. Nombre d'entre elles sont intégrées à ASP.NET Core, comme vous le verrez plus loin dans cet ouvrage. Vous pouvez également consulter la documentation .NET pour plus d'informations sur l'utilisation d'autres méthodes de configuration des applications.

>[!warning] L'auteur suggère peut-être de mettre vos chaînes de connexion directement dans le fichier de config. **Ne le faites pas**.
>
>Utilisez l'outil **Secret Manager** (`dotnet user-secrets`) qui stocke vos données sensibles dans un dossier local de votre Mac (`~/.microsoft/usersecrets/`), totalement en dehors de votre dossier de projet et de Git

# Création et utilisation d'une bibliothèque de classes .NET

**Pour commencer à explorer l'univers des bibliothèques de classes .NET, vous allez d'abord créer un assembly** *.dll* (nommé *CarLibrary*) **contenant un petit ensemble de types publics**. Si vous utilisez Visual Studio, nommez le fichier de solution de manière significative (différente de *CarLibrary*). Vous ajouterez deux projets à la solution plus loin dans cette section. Si vous utilisez Visual Studio Code, vous n'avez pas besoin de solution, bien que la plupart des développeurs trouvent utile de regrouper les projets associés dans une seule solution.

Pour rappel, vous pouvez créer et gérer des solutions et des projets via l'interface de ligne de commande .NET (CLI). Utilisez la commande suivante pour créer la solution et la bibliothèque de classes :

```bash
dotnet new classlib -n CarLibrary
dotnet sln Chapter16_AllProjects.slnx add CarLibrary
```

>[!note] 
>L'interface de ligne de commande .NET dispose d'un système d'aide performant. Pour obtenir de l'aide sur une commande, ajoutez l'option `-h` à la fin de celle-ci. Par exemple, pour afficher tous les modèles, saisissez `dotnet new -h`. Pour obtenir plus d'informations sur la création d'une bibliothèque de classes, saisissez `dotnet new classlib -h`.

**Maintenant que vous avez créé votre projet et votre solution, vous pouvez les ouvrir** dans Visual Studio (ou Visual Studio Code) **pour commencer à générer les classes**. Après avoir ouvert la solution, supprimez le fichier *Class1.cs* généré automatiquement.

La conception de votre bibliothèque automobile commence avec les énumérations `EngineStateEnum` et `MusicMediaEnum`. Ajoutez deux fichiers à votre projet, nommés *MusicMediaEnum.cs* et *EngineStateEnum.cs,* et ajoutez-y le code suivant (à compléter)

```cs
namespace CarLibrary;

// Quel type de lecteur de musique possède cette voiture ?
public enum MusicMediaEnum
{
    MusicCd,
    MusicTape,
    MusicRadio,
    MusicMp3,
}
```

```cs
namespace CarLibrary;

// Représentent les états du moteur.
public enum EngineStateEnum
{
    EngineAlive,
    EngineDead,
}
```

Ensuite, insérez un nouveau fichier de classe C# dans votre projet, nommé *Car.cs*, qui contiendra une classe de base abstraite nommée `Car`. Cette classe définit diverses données d'état via une syntaxe de propriétés automatique. Cette classe possède également une unique méthode abstraite nommée `TurboBoost()`, qui utilise l'énumération `EngineStateEnum` pour représenter l'état actuel du moteur de la voiture. Mettez à jour le fichier avec le code suivant :

```cs
namespace CarLibrary;

// La classe de base abstraite dans la hiérarchie.
public abstract class Car
{
    public string PetName { get; set; }
    public int CurrentSpeed { get; set; }
    public int MaxSpeed { get; set; }

    protected EngineStateEnum State = EngineStateEnum.EngineAlive;
    public EngineStateEnum EngineState => State;
    public abstract void TurboBoost();

    protected Car() { }

    protected Car(string name, int maxSpeed, int currentSpeed)
    {
        PetName = name;
        MaxSpeed = maxSpeed;
        CurrentSpeed = currentSpeed;
    }
}
```

Supposons maintenant que vous ayez deux classes descendantes directes de la classe `Car`, nommées `MiniVan` et `SportsCar`. Chacune, redéfinit la méthode abstraite `TurboBoost()` en affichant un message approprié dans la console. Insérez deux nouveaux fichiers de classe C# dans votre projet, nommés respectivement *MiniVan.cs* et *SportsCar.cs*. Mettez à jour le code de chaque fichier avec le code correspondant.

```cs
// SportsCar.cs
namespace CarLibrary;

public class SportsCar : Car
{
    public SportsCar() { }

    public SportsCar(string name, int maxSpeed, int currentSpeed)
        : base(name, maxSpeed, currentSpeed) { }

    public override void TurboBoost()
    {
        Console.WriteLine("Ramming speed! Faster is better...");
    }
}
```

```cs
// MiniVan.cs
namespace CarLibrary;

public class MiniVan : Car
{
    public MiniVan() { }

    public MiniVan(string name, int maxSpeed, int currentSpeed)
        : base(name, maxSpeed, currentSpeed) { }

    public override void TurboBoost()
    {
        // Les minivans ont de faibles performances en turbo !
        State = EngineStateEnum.EngineDead;
        Console.WriteLine("Eek! Your engine block exploded!");
    }
}
```

## Exploration du manifeste

Avant d'utiliser *CarLibrary.dll* depuis une application cliente, ==examinons la structure interne de la bibliothèque de code==. **Si vous avez compilé ce projet, exécutez** *ildasm.exe* **sur l'assembly compilé**. Si vous ne possédez pas *ildasm.exe* (présenté précédemment dans cet ouvrage), vous le trouverez également dans le répertoire du chapitre dans le dépôt [GitHub](https://github.com/Apress/pro-c-sharp-10/tree/main/Chapter_16) de ce livre.

```bash
ildasm -metadata CarLibrary/bin/debug/net10.0/CarLibrary.dll -out=CarLibrary/CarLibrary.il
```

La section `Manifest` des résultats désassemblés commence par `//Metadata version: 4.0.30319`. ***==Suit immédiatement la liste de tous les assemblys externes requis par la bibliothèque de classes==***, comme indiqué ici :

>[!tip] Il n'est pas nécessaire d'utiliser `-metadata` pour avoir les données affiché plus loin.

```CIL
.assembly extern System.Runtime
{
  .publickeytoken = (B0 3F 5F 7F 11 D5 0A 3A )                         // .?_....:
  .ver 10:0:0:0
}
.assembly extern System.Console
{
  .publickeytoken = (B0 3F 5F 7F 11 D5 0A 3A )                         // .?_....:
  .ver 10:0:0:0
}
```

**Chaque bloc externe `.assembly` est qualifié par les directives `.publickeytoken` et `.ver`**. ***==L'instruction `.publickeytoken` est présente uniquement si l'assembly a été configuré avec un nom fort==***. Le jeton `.ver` définit l'identifiant de version numérique de l'assembly référencé.

>[!note]
>Les versions précédentes du .NET Framework reposaient fortement sur une convention de nommage stricte, impliquant l'utilisation d'une combinaison de clés publique/privée. Cette convention était requise sous Windows pour qu'un assembly soit ajouté au Global Assembly Cache (GAC), mais son utilité a considérablement diminué avec l'arrivée de .NET Core.

**Après les références externes, vous trouverez plusieurs jetons `.custom` qui identifient des attributs au niveau de l'assembly** (==certains générés par le système, mais aussi des informations de copyright, le nom de l'entreprise, la version de l'assembly, etc.==). Voici une liste (très) partielle de cette portion de données du manifeste :

```CIL
.assembly CarLibrary
{
	...
	.custom instance void ... TargetFrameworkAttribute ...
	.custom instance void ... AssemblyCompanyAttribute ...
	.custom instance void ... AssemblyConfigurationAttribute ...
	.custom instance void ... AssemblyFileVersionAttribute ...
	.custom instance void ... AssemblyProductAttribute ...
	.custom instance void ... AssemblyTitleAttribute ...
	...
}
```

Ces paramètres peuvent être définis soit à l'aide des pages de propriétés de Visual Studio, soit en modifiant le fichier projet et en y ajoutant les éléments appropriés. Pour modifier les propriétés du package dans Visual Studio 2022, cliquez avec le bouton droit sur le projet dans l'Explorateur de solutions, sélectionnez Propriétés, puis accédez au menu Package dans le volet gauche de la fenêtre. Ceci affiche la boîte de dialogue illustrée dans l' image suivante. Par souci de concision, l'image suivante ne représente qu'une partie de cette boîte de dialogue.

![[Figure 16.4.png|Modification des informations d'assemblage à l'aide de la fenêtre Propriétés de Visual Studio 2022]]

>[!note] 
>L'écran « Package » comporte trois champs de version différents. La version de l'assembly et la version du fichier utilisent le même schéma, basé sur le versionnage sémantique (https://semver.org). Le premier nombre correspond à la version majeure, le deuxième à la version mineure et le troisième au numéro de correctif. Le quatrième nombre indique généralement le numéro de build. Le numéro de version du package doit respecter le versionnage sémantique, en utilisant uniquement les espaces réservés `{major}.{minor}.{patch}`. Le versionnage sémantique autorise une extension alphanumérique à la version, séparée par un tiret au lieu d'un point (par exemple : 1.0.0-rc). Ceci indique les versions incomplètes, telles que les versions bêta et les versions candidates.
>
>La version d'un package définit la version du package NuGet (le packaging NuGet est abordé plus en détail ultérieurement dans ce chapitre). La version de l'assembly est utilisée par .NET lors de la compilation et de l'exécution pour localiser, lier et charger les assemblies. La version Fichier est utilisée uniquement par l'Explorateur Windows et non par .NET.

**Une autre façon d'ajouter les métadonnées à votre assembly consiste à le faire directement dans le fichier projet** *.csproj*. ***==La mise à jour suivante du `PropertyGroup` principal dans le fichier projet a le même effet que le remplissage du formulaire illustré dans l'image précédente==***. **==Notez que la version du package est simplement appelée « Version » dans le fichier projet.==**

```xml
<PropertyGroup>
  <TargetFramework>net10.0</TargetFramework>
  <ImplicitUsings>enable</ImplicitUsings>
  <Nullable>disable</Nullable>
  <Copyright>Copyright 2026</Copyright>
  <Authors>Gianni Mucci</Authors>
  <Company>No Name</Company>
  <Product>Pro C# 10.0</Product>
  <PackageId>CarLibrary</PackageId>
  <Description>This is an awesome library for cars.</Description>
  <AssemblyVersion>1.0.0.1</AssemblyVersion>
  <FileVersion>1.0.0.2</FileVersion>
  <Version>1.0.3</Version>
</PropertyGroup>
```

>[!note] 
>Les autres entrées de l'image précédent (et la liste des fichiers projet) sont utilisées lors de la génération des packages NuGet à partir de votre assembly. Ce point est abordé plus loin dans ce chapitre.

## Exploration du CIL

**Rappelons qu'un assembly ne contient pas d'instructions spécifiques à une plateforme**; **==il contient plutôt des instructions CIL (Common Intermediate Language) indépendantes de la plateforme==**. **Lorsque le runtime .NET charge un assembly en mémoire, le CIL sous-jacent est compilé** (***==à l'aide du compilateur JIT==***) **en instructions compréhensibles par la plateforme cible**. Par exemple, la méthode `TurboBoost()` de la classe `SportsCar` est représentée par le code CIL suivant :

```CIL
  .method public hidebysig virtual instance void 
          TurboBoost() cil managed
  {
    // Code size       13 (0xd)
    .maxstack  8
    IL_0000:  nop
    IL_0001:  ldstr      "Ramming speed! Faster is better..."
    IL_0006:  call       void [System.Console]System.Console::WriteLine(string)
    IL_000b:  nop
    IL_000c:  ret
  } // end of method SportsCar::TurboBoost
```

**Comme pour les autres exemples CIL de ce livre, la plupart des développeurs .NET n'ont pas besoin de s'intéresser de près aux détails**. **==Le [[Chapitre 18|Chapitre 18]] fournit davantage d'informations sur sa syntaxe et sa sémantique, ce qui peut s'avérer utile lors de la création d'applications plus complexes nécessitant des services avancés, tels que la construction dynamique d'assemblies.==**

## Exploration des métadonnées de type

Avant de développer des applications utilisant votre bibliothèque .NET personnalisée, examinez les métadonnées des types dans l'assembly *CarLibrary.dll.* Par exemple, voici la définition de type `TypeDef` de l'énumération `EngineStateEnum` :

```CIL
 TypeDef #2 (02000003)
 -------------------------------------------------------
 	TypDefName: CarLibrary.EngineStateEnum  (02000003)
 	Flags     : [Public] [AutoLayout] [Class] [Sealed] [AnsiClass]  (00000101)
 	Extends   : 01000011 [TypeRef] System.Enum
 	Field #1 (04000005)
 	-------------------------------------------------------
 		Field Name: value__ (04000005)
 		Flags     : [Public] [SpecialName] [RTSpecialName]  (00000606)
 		CallCnvntn: [FIELD]
 		Field type:  I4
 
 	Field #2 (04000006)
 	-------------------------------------------------------
 		Field Name: EngineAlive (04000006)
 		Flags     : [Public] [Static] [Literal] [HasDefault]  (00008056)
 		CallCnvntn: [FIELD]
 		Field type:  ValueClass CarLibrary.EngineStateEnum
 
 	Field #3 (04000007)
 	-------------------------------------------------------
 		Field Name: EngineDead (04000007)
 		Flags     : [Public] [Static] [Literal] [HasDefault]  (00008056)
 		CallCnvntn: [FIELD]
 		Field type:  ValueClass CarLibrary.EngineStateEnum
 
```

Comme expliqué dans le chapitre suivant, **les métadonnées d'un assembly sont un élément important de la plateforme .NET et constituent la base de nombreuses technologies** (sérialisation d'objets, ***==liaison tardive==***, applications extensibles, etc.). Quoi qu'il en soit, maintenant que vous avez examiné l'assembly *CarLibrary.dll*, vous pouvez créer des applications clientes utilisant vos types.

## Création d'une application cliente C#

**Comme chaque type du projet** *CarLibrary* **est déclaré avec le mot-clé `public`, d'autres applications .NET peuvent également les utiliser**. **==Rappelons que vous pouvez aussi définir des types avec le mot-clé `internal` en C#==** (***==il s'agit d'ailleurs du mode d'accès par défaut des classes en C#==***). **Les types `internal` ne peuvent être utilisés que par l'assembly dans lequel ils sont définis**. *==Les clients externes ne peuvent ni voir ni créer les types marqués avec le mot-clé `internal`.==*

>[!note]
>L'exception à la règle interne est lorsqu'un assembly autorise explicitement l'accès à un autre assembly à l'aide de l'attribut InternalsVisibleTo, que nous aborderons prochainement.

>[!info]- Evolutions avec les versions modernes de C#. (Gemini)
>
>## La règle du `public` vs `internal` (Toujours vrai)
>
>C'est le pilier de l'encapsulation en .NET.
>
>- **Default access modifier :** Une classe déclarée sans modificateur (ex: `class Car {}`) est toujours **internal** par défaut. C'est une sécurité : on ne rend public que ce qui est nécessaire.
>- **Visibilité :** Un type `internal` reste invisible pour le projet client qui ajoute une référence à votre DLL.
><br>
>
>---
>
> ## Ce qui a "évolué" (Nuances modernes)
>
> ### Le mode d'accès `file` (C# 11)
>
>C'est le plus gros changement récent. Avant, `internal` était le niveau le plus restrictif pour une classe. Depuis C# 11, il existe le modificateur `file` :
>
>```cs
>file class HiddenTechnique { ... }
>```
>
>Une classe `file` n'est visible **que dans le fichier .cs où elle est écrite**. Même les autres classes du même projet (du même assembly) ne peuvent pas la voir. C'est encore plus restrictif qu'internal.
>
> ### Les "Internal Visible To" simplifiés
>
>Le livre mentionne `InternalsVisibleTo`. Dans les versions modernes de .NET, on l'utilise massivement pour les **tests unitaires**.
>
>- Auparavant, on l'ajoutait souvent dans un fichier `AssemblyInfo.cs`.
>- Aujourd'hui, on l'ajoute souvent directement dans le fichier projet `.csproj` de la bibliothèque :
>
>```xml
><ItemGroup>
>  <AssemblyAttribute Include="System.Runtime.CompilerServices.InternalsVisibleTo">
>    <_Parameter1>CarLibrary.Tests</_Parameter1>
>  </AssemblyAttribute>
></ItemGroup>
>```
>
>
>### Les interfaces avec membres statiques (C# 11)
>
>Un changement subtil : vous pouvez maintenant avoir des membres `static abstract` dans une interface. Si l'interface est `public`, mais qu'un type impliqué est `internal`, le compilateur vous bloquera. La cohérence de visibilité est devenue plus complexe avec ces nouvelles fonctionnalités.
>
>---
>
>## Pourquoi c'est encore plus important aujourd'hui ?
>
>Avec l'avènement des **Microservices** et des bibliothèques **NuGet** légères, la gestion du `internal` est cruciale pour éviter les "breaking changes".
>
>- Si vous mettez tout en `public`, vous ne pouvez plus rien changer sans casser le code de vos utilisateurs.
>- En gardant le maximum de choses en `internal`, vous gardez la liberté de modifier votre logique interne (le code CIL dans votre DLL) sans que le client (l'application qui utilise la DLL) ne s'en aperçoive.

Pour utiliser les fonctionnalités de votre bibliothèque, créez un nouveau projet d'application console C# nommé *CSharpCarClient* dans la même solution que *CarLibrary*. Vous pouvez le faire à l'aide de Visual Studio (clic droit sur la solution et sélectionnez Ajouter -> Nouveau projet) ou à l'aide de la ligne de commande (trois lignes, exécutées séparément).

```bash
dotnet new console -n CSharpCarClient
dotnet add CSharpCarClient reference CarLibrary
dotnet sln Chapter16_AllProjects.slnx add CSharpCarClient
```

**Les commandes précédentes ont créé l'application console, ajouté une référence au projet** *CarLibrary* **pour le nouveau projet et l'ont ajoutée à votre solution.**

>[!note]
>La commande `add reference` crée une *référence de projet*. Ceci est pratique pour le développement, car *CSharpCarClient* utilisera toujours la dernière version de *CarLibrary*. ==Vous pouvez également référencer un assembly *directement*==. Les références directes sont créées en référençant la bibliothèque de classes compilée.

Si la solution est toujours ouverte dans Visual Studio, vous constaterez que le nouveau projet apparaît dans l'Explorateur de solutions sans aucune intervention de votre part.

La dernière modification à effectuer consiste à cliquer avec le bouton droit sur *CSharpCarClient* dans l'Explorateur de solutions et à sélectionner « Définir comme projet de démarrage ». Si vous n'utilisez pas Visual Studio, vous pouvez exécuter le nouveau projet en lançant la commande `dotnet run` dans le répertoire du projet.

>[!note]
>Vous pouvez également définir la référence de projet dans Visual Studio en cliquant avec le bouton droit sur le projet *CSharpCarClient* dans l’Explorateur de solutions, en sélectionnant Ajouter -> Référence, puis en sélectionnant le projet *CarLibrary* dans le nœud du projet.

**À ce stade, vous pouvez créer votre application cliente pour utiliser les types externes**. Mettez à jour le fichier *Program.cs* comme suit :

```cs
// N'oubliez pas d'importer l'espace de noms CarLibrary!
using CarLibrary;

Console.Title = "C# CarLibrary Client App";
Console.WriteLine("***** C# CarLibrary Client App *****\n");

// Crée une voiture de sport
SportsCar viper = new SportsCar("Viper", 240, 40);
viper.TurboBoost();

// Créé un minivan
MiniVan mv = new MiniVan();
mv.TurboBoost();

Console.WriteLine("Done. Press any key to terminate");
Console.ReadLine();
```

**Ce code ressemble trait pour trait à celui des autres applications développées jusqu'ici dans ce livre**. ==La seule différence réside dans le fait que l'application cliente C# utilise désormais des types définis dans une bibliothèque personnalisée distincte==. Exécutez votre programme et vérifiez que les messages s'affichent correctement.

==Vous vous demandez peut-être ce qui s'est passé exactement lorsque vous avez référencé le projet *CarLibrary*==. **Lorsqu'une référence de projet est effectuée, l'ordre de compilation de la solution est ajusté afin que les projets dépendants** (*CarLibrary* dans cet exemple) **soient compilés en premier, et que le résultat de cette compilation soit copié dans le répertoire de sortie du projet parent** (*CSharpCarLibrary*). ==La bibliothèque cliente compilée référence la bibliothèque de classes compilée==. ***==Lorsque le projet client est recompilé, la bibliothèque dépendante l'est également, et la nouvelle version est à nouveau copiée dans le dossier cible.==***

>[!note]
>Si vous utilisez Visual Studio, cliquez sur le bouton « Afficher tous les fichiers » dans l’Explorateur de solutions pour visualiser tous les fichiers de sortie et vérifier la présence de la bibliothèque *CarLibrary* compilée. Si vous utilisez Visual Studio Code, accédez au répertoire `bin/debug/net10.0` dans l’onglet Explorateur.

**Lorsqu'une** *référence directe* **est utilisée au lieu d'une** *référence de projet*, **la bibliothèque compilée est également copiée dans le répertoire de sortie de la bibliothèque cliente, mais seulement au moment où la référence est établie**. *==Sans la référence de projet, les projets peuvent être compilés indépendamment l'un de l'autre, et les fichiers risquent de se désynchroniser==*. ***==En bref, si vous développez des bibliothèques dépendantes==*** (comme c'est généralement le cas pour les projets logiciels réels), ***==il est préférable de référencer le projet et non sa sortie.==***

## Création d'une application cliente Visual Basic

Rappelons que la plateforme .NET permet aux développeurs de partager du code compilé entre différents langages de programmation. Pour illustrer l'indépendance de la plateforme .NET vis-à-vis du langage, créons un autre projet d'application console (*VisualBasicCarClient*), cette fois-ci en Visual Basic (notez que chaque commande tient sur une seule ligne).

```bash
dotnet new console -lang vb -n VisualBasicCarClient
dotnet add VisualBasicCarClient reference CarLibrary
dotnet sln Chapter16_AllProjects.slnx add VisualBasicCarClient
```

**Comme C#, Visual Basic vous permet de lister chaque espace de noms utilisé dans le fichier courant**. Cependant, ==Visual Basic utilise le mot-clé `Imports` au lieu du mot-clé `using` de C#.== Ajoutez donc l'instruction `Imports` suivante dans le fichier de code *Program.vb* :

```vb
Imports CarLibrary
Module Program
    Sub Main()
    End Sub
End Module
```

Notez que la méthode `Main()` est définie dans un module Visual Basic. En résumé, les modules, sont une notation Visual Basic permettant de définir une classe ne pouvant contenir que des méthodes statiques (un peu comme une classe statique en C#). Pour utiliser les types `MiniVan` et `SportsCar` avec la syntaxe Visual Basic, mettez à jour votre méthode `Main()` comme suit :

```vb
Sub Main(args As String())
	Console.Title = "VB CarLibrary Client App"
	Console.WriteLine("***** VB CarLibrary Client App *****\n")
	
	' Les variables locales sont déclarées à l'aide du mot-clé Dim.
	Dim myMiniVan As New MiniVan()
	myMiniVan.TurboBoost()
	
	Dim mySportsCar As New SportsCar()
	mySportsCar.TurboBoost()
end Sub
```

Lorsque vous compilez et exécutez votre application (==assurez-vous de définir *VisualBasicCarClient* comme projet de démarrage si vous utilisez Visual Studio==), vous verrez à nouveau s'afficher une série de boîtes de dialogue. De plus, cette nouvelle application cliente possède sa propre copie locale de *CarLibrary.dll* située dans le dossier `bin\Debug\netVersion`.

## Héritage interlangage en pratique

**L'un des aspects les plus intéressants du développement .NET est la notion d'héritage interlangage**. Pour illustrer cela, créons une nouvelle classe Visual Basic qui hérite de `SportsCar` (écrite en C#). Commencez par ajouter un nouveau fichier de classe nommé *PerformanceCar.vb* à votre application Visual Basic. Mettez à jour la définition initiale de la classe en héritant du type `SportsCar` à l'aide du mot-clé `Inherits`. Ensuite, redéfinissez la méthode abstraite `TurboBoost()` à l'aide du mot-clé `Overrides`, comme ceci :

```vb
Imports CarLibrary

Public Class PerformanceCar
    Inherits SportsCar

    Public Overrides Sub TurboBoost()
        Console.WriteLine("Zero to 60 in a cool 4.8 seconds...")
    End Sub

End Class
```

Pour tester ce nouveau type de classe, mettez à jour la méthode `Main()` du module comme suit :

```vb
Sub Main()
	...
	
	Dim dreamCar As New PerformanceCar()
	' Utilisation d'une propriété héritée.
	dreamCar.PetName = "Hank"
	dreamCar.TurboBoost()
	Console.ReadLine()
End Sub
```

**Notez que l'objet `dreamCar` peut appeler n'importe quel membre public** (comme la propriété `PetName`) **trouvé en amont de la chaîne d'héritage, même si la classe de base a été définie dans un langage et un assembly complètement différents** ! **==La possibilité d'étendre les classes au-delà des limites des assemblies, de manière indépendante du langage, est un aspect naturel du cycle de développement .NET==**. Cela facilite l'utilisation de code compilé par des personnes qui préfèrent ne pas développer leur code partagé en C#.

## Exposition des types internes à d'autres assemblies

Comme mentionné précédemment, ***==les classes internes ne sont visibles que par les autres objets de l'assembly où elles sont définies==***. **L'exception à cette règle est lorsque la visibilité est explicitement accordée à un autre projet. **

Commencez par ajouter une nouvelle classe nommée `MyInternalClass` au projet *CarLibrary*, puis mettez à jour le code comme suit :

```cs
namespace CarLibrary;

internal class MyInternalClass { }
```

>[!note] 
>Pourquoi exposer les types internes ? Cela se fait généralement pour les tests unitaires et d'intégration. Les développeurs souhaitent pouvoir tester leur code sans nécessairement l'exposer en dehors de l'assembly.

### Utilisation d'un attribut d'assembly

Le [[Chapitre 17|Chapitre 17]] traitera des attributs en détail, mais pour l'instant, ouvrez la classe *Car.cs* du projet *CarLibrary* et ajoutez l'attribut et l'instruction `using` suivants :

```cs
using System.Runtime.CompilerServices;

[assembly: InternalsVisibleTo("CSharpCarClient")]

namespace CarLibrary;
```

**L'attribut `InternalsVisibleTo` prend le nom du projet autorisé à consulter la classe dont l'attribut est défini**. Notez que ***==les autres projets ne peuvent pas demander cette autorisation==***; **elle doit être accordée par le projet contenant les types internes.**

>[!note]
>Les versions précédentes du .NET Framework permettaient de placer des attributs au niveau de l'assembly dans la classe *AssemblyInfo.cs*, qui existe toujours dans .NET mais est générée automatiquement et n'est pas destinée à être utilisée par les développeurs.

>[!info]
>L'attribut `[assembly: ...]` est un "attribut global". Il ne s'applique pas à la classe ou au namespace qui suit, mais à l'intégralité du fichier binaire (la DLL) une fois compilé. C'est pour cela qu'on le place traditionnellement tout en haut du fichier, en dehors de tout namespace

Vous pouvez maintenant mettre à jour le projet *CSharpCarClient* en ajoutant le code suivant aux instructions de niveau supérieur :

```cs
var InternalClassInstance = new MyInternalClass();
```

Cela fonctionne parfaitement. Essayez maintenant de faire la même chose dans la méthode `Main` de *VisualBasicCarClient*.

```vb
' Ne compilera pas
'Dim internalClassInstance = New MyInternalClass()
```

**La bibliothèque** *VisualBasicCarClient* **n'ayant pas l'autorisation d'accéder à ses éléments internes, la ligne de code précédente ne compilera pas.**

### Utilisation du fichier projet

>[!success] Méthode recommandé pour les version moderne de .NET

**Une autre façon d'obtenir le même résultat consiste à utiliser les fonctionnalités mises à jour du fichier projet .NET**. ==Commentez l'attribut que vous venez d'ajouter== et ouvrez le fichier projet de *CarLibrary*. Ajoutez l'élément suivant `ItemGroup` dans le fichier projet :

```xml
<ItemGroup>
  <AssemblyAttribute Include="System.Runtime.CompilerServices.InternalsVisibleToAttribute">
	<_Parameter1>CsharpCarclient</_Parameter1>
  </AssemblyAttribute>
</ItemGroup>
```

***==Cela permet d'obtenir le même résultat que l'utilisation de l'attribut sur une classe et, à mon avis, c'est une meilleure solution, car les autres développeurs le verront directement dans le fichier projet au lieu de devoir chercher dans tout le projet.==***

# NuGet et .NET Core

**NuGet est le gestionnaire de packages pour le .NET Framework et .NET Core**. **==Il permet de partager des logiciels dans un format compréhensible par les applications .NET et constitue le mécanisme par défaut pour charger .NET et ses composants associés==** (ASP.NET Core, EF Core, etc.). ==De nombreuses organisations regroupent leurs assemblies standard pour des fonctionnalités transversales== (comme la journalisation et la gestion des erreurs) ==dans des packages NuGet afin de les intégrer à leurs applications métiers.==

## Packager des assemblies avec NuGet

**Pour voir cela en pratique, nous allons transformer** *CarLibrary* **en package NuGet, puis l'utiliser depuis les deux applications clientes.**

Les propriétés du package NuGet sont accessibles depuis les pages de propriétés du projet. Faites un clic droit sur le projet CarLibrary et sélectionnez Propriétés. Accédez à la page Package et consultez les valeurs que nous avons saisies précédemment pour personnaliser l'assembly. D'autres propriétés peuvent être définies pour le package NuGet (par exemple, l'acceptation du contrat de licence et les informations du projet telles que l'URL et l'emplacement du dépôt).

>[!note]
>Toutes les valeurs de l'interface utilisateur de la page « Package » de Visual Studio peuvent être saisies manuellement dans le fichier projet, mais il est nécessaire de connaître les mots clés. Il est conseillé d'utiliser Visual Studio au moins une fois pour tout renseigner, puis vous pourrez modifier le fichier projet manuellement. Vous trouverez également toutes les propriétés autorisées dans la [documentation](https://learn.microsoft.com/en-us/nuget/quickstart/create-and-publish-a-package-using-the-dotnet-cli?toc=/dotnet/fundamentals/toc.json&bc=/dotnet/breadcrumb/toc.json) .NET.

Pour cet exemple, il n'est pas nécessaire de définir de propriétés supplémentaires, si ce n'est de cocher la case « Générer le package NuGet lors de la compilation » ou de mettre à jour le fichier projet avec les informations suivantes :

```xml
<PropertyGroup>
    <TargetFramework>net10.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>disable</Nullable>
    <Copyright>Copyright 2026</Copyright>
    <Authors>Gianni Mucci</Authors>
    <Company>No Name</Company>
    <Product>Pro C# 10.0</Product>
    <PackageId>CarLibrary</PackageId>
    <Description>This is an awesome library for cars.</Description>
    <AssemblyVersion>1.0.0.1</AssemblyVersion>
    <FileVersion>1.0.0.2</FileVersion>
    <Version>1.0.3</Version>
    <GeneratePackageOnBuild>true</GeneratePackageOnBuild>
</PropertyGroup>
```

>[!warning] Dans les versions plus anciennes, on avait tendance à tout déclarer. Aujourd'hui, le SDK suit une règle de cascade. Si vous ne mettez que `<Version>`, le SDK fait ceci pour vous

**Cela entraînera la reconstruction du package à chaque compilation du logiciel**. ***==Par défaut, le package sera créé dans le dossier `bin/Debug` ou `bin/Release`, selon la configuration sélectionnée.==***

**Il est également possible de créer des packages en ligne de commande, et l'interface de ligne de commande** (CLI) **offre davantage d'options que Visual Studio**. Par exemple, ==pour créer le package et le placer dans un répertoire nommé `Publish`, saisissez les commandes suivantes== (dans le répertoire du projet *CarLibrary*). La première commande génère l'assembly, et la seconde crée le package NuGet.

```bash
dotnet build -c Release
dotnet pack -o ./Publish -c Release
```

>[!tip]- Explication des deux commandes. (Gemini)
>
>La première commande compile votre code source en binaire (CIL).
>
>- **`build`** : Analyse le code, vérifie les erreurs et génère les fichiers `.dll`.
>- **`-c Release`** (Configuration) : C'est le point crucial. Par défaut, .NET compile en mode `Debug`.
>    - Le mode **Release** demande au compilateur d'activer les **optimisations**.
>    - Il supprime les informations de débogage superflues pour rendre la DLL plus légère et plus rapide.
>    - C'est cette version que vous devez distribuer.
>
>La deuxième commande crée le "paquet" qui sera envoyé sur NuGet.
>
>- **`pack`** : Elle ne se contente pas de compiler. Elle prend la DLL (et ses métadonnées) et l'enveloppe dans un fichier unique avec l'extension **`.nupkg`** (un NuGet Package). Ce fichier est en réalité une archive compressée contenant votre code et un fichier de description (`.nuspec`).
>- **`-o .\Publish`** (Output) : Elle indique où enregistrer le fichier `.nupkg` généré. Ici, il sera placé dans un dossier nommé `Publish` à la racine de votre projet.
>- **`-c Release`** : Elle s'assure que le paquet contient bien la version optimisée de votre bibliothèque et non la version de test (Debug).

**Le fichier** *CarLibrary.1.0.3.nupkg* **se trouve maintenant dans le répertoire `Publish`**. ***==Pour en consulter le contenu, ouvrez-le avec un utilitaire de compression==*** (tel que 7-Zip). ==Vous pourrez alors visualiser l'intégralité du contenu, y compris l'assemblage, ainsi que des métadonnées supplémentaires.==

## Référencement des packages NuGet

Vous vous demandez peut-être d'où proviennent les packages ajoutés dans les exemples précédents. **L'emplacement des packages NuGet est contrôlé par un fichier XML nommé** *NuGet.config*. Ce fichier se trouve dans le répertoire `%appdata%\NuGet`(pour Windows) / `~/.nuget` (pour MacOS/Linux). **==Il s'agit du fichier principal. Ouvrez-le : vous y trouverez plusieurs sources de packages.==**

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration> 
  <packageSources>
     <add key="nuget.org" value="https://api.nuget.org/v3/index.json" protocolVersion="3" />
  </packageSources>
</configuration>
```

La liste de fichiers précédente affiche deux sources. La première pointe vers NuGet.org, le plus grand dépôt de packages NuGet au monde. ==La seconde se trouve sur votre disque local et est utilisée par Visual Studio comme cache de packages.==

>Comme affiché dans l'exemple, sur mon mac, je n'ai pas la seconde source.

**Il est important de noter que les fichiers *NuGet.Config* sont cumulatifs par défaut**. ***==Pour ajouter des sources supplémentaires sans modifier la liste pour l'ensemble du système, vous pouvez ajouter d'autres fichiers==*** *NuGet.Config*. **Chaque fichier est valide pour le répertoire dans lequel il est placé ainsi que pour tout sous-répertoire**. Ajoutez un nouveau fichier nommé *NuGet.Config* **==dans le répertoire de la solution==** (*==pas dans le dossier du projet==*) et mettez à jour son contenu comme suit :

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
	<add key="local-packages" value="./Publish" />
  </packageSources>
</configuration>
```

**Vous pouvez également réinitialiser la liste des paquets en ajoutant `<clear/>` dans le nœud `<packageSources>`**, comme ceci :

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <clear />
    <add key="local-packages" value=".\CarLibrary\Publish" />
    <add key="NuGet" value="https://api.nuget.org/v3/index.json" />
  </packageSources>
</configuration>
```

>[!note]
>Si vous utilisez Visual Studio, vous devrez redémarrer l'IDE pour que les paramètres mis à jour du fichier *nuget.config* soient pris en compte.

**Supprimez les références dans projets** *CSharpCarClient* **et** *VisualBasicCarClient*, **puis ajoutez les références aux packages comme ceci (depuis le répertoire de la solution) :**

```bash
dotnet add CSharpCarClient package CarLibrary
dotnet add VisualBasicCarClient package CarLibrary
```

**Une fois les références définies, compilez la solution et examinez le résultat dans les répertoires cibles** (`bin\Debug\new10.0`). ***==Vous y trouverez le fichier==*** *CarLibrary.dll* ***==et non le fichier==*** *CarLibrary.nupkg*. En effet, **==.NET décompresse le contenu et ajoute les assemblies contenus en tant que références directes.==**

Définissez maintenant l'un des clients comme projet de démarrage et exécutez l'application : elle fonctionnera comme auparavant.

==Mettez ensuite à jour le numéro de version de *CarLibrary* à `1.0.4` et recompilez-le==. Dans le répertoire `Publish`, **vous trouverez désormais deux packages NuGet** *CarLibrary*. **Si vous exécutez à nouveau les commandes d'ajout de package, le projet sera mis à jour pour utiliser la nouvelle version. Si vous préférez l'ancienne version, la commande d'ajout de package permet d'ajouter des numéros de version pour un package spécifique.**

# Publication d'applications console (MaJ .NET 5/6)

Maintenant que vous avez votre application C# CarClient (et son assembly *CarLibrary* associé), **comment la mettre à disposition de vos utilisateurs ?** ==Le processus de packaging de votre application et de ses dépendances est appelé *publication*==. ==La publication d'applications .NET Framework nécessitait l'installation du framework sur la machine cible, puis il suffisait de copier l'exécutable et les fichiers associés pour exécuter votre application sur une autre machine==.

Comme vous pouvez vous y attendre, les applications .NET peuvent également être publiées de manière similaire, ce que l'on appelle un déploiement *dépendant du framework*. Cependant, ==les applications .NET peuvent aussi être publiées en tant qu'*application autonome*, sans nécessiter l'installation de .NET !==

**Lors de la publication d'applications autonomes, vous devez spécifier l'identificateur d'exécution cible (RID) dans le fichier projet ou via les options de ligne de commande**. **==L'identificateur d'exécution sert à packager votre application pour un système d'exploitation spécifique==**. ***==Lorsqu'un identificateur d'exécution est spécifié, le processus de publication est par défaut autonome==***. Pour obtenir la liste complète des identificateurs d'exécution (RID) disponibles, consultez le catalogue .NET RID à l'adresse : https://docs.microsoft.com/en-us/dotnet/core/rid-catalog.

**Les applications peuvent être publiées directement depuis Visual Studio ou à l'aide de l'interface de ligne de commande .NET (CLI)**. **==La commande pour la CLI est : `dotnet publish`. Pour afficher toutes les options, utilisez : `dotnet publish -h`==**. Le [[#Tableau 16-1 Quelque options pour publier une application|Tableau 16-1]] présente les options courantes utilisées lors de la publication en ligne de commande.

>[!warning] Publier une application != envoyer l'application sur les serveurs NuGet !
>
> Ici, le terme **publier** signifie que l'on prépare le produit fini pour l'utilisateur final.
> 
> - **Transformation :** On passe d'un assemblage universel (CIL) à un exécutable natif (Code Machine / ASM) optimisé pour un processeur spécifique (ex: mon M4 via le RID `osx-arm64`).
> - **Autonomie :** On peut inclure tout le moteur .NET dans l'exécutable (`--self-contained`), rendant l'appli installable sur une machine vierge.
> - **Format :** Sur macOS, cela génère un binaire "Mach-O" exécutable directement sans la commande `dotnet`.

>[!tip]- Le concept reste le même, mais Microsoft a apporté des améliorations majeures depuis **.NET 6** pour rendre les fichiers plus petits et les performances plus élevées. 
>
>### Le "Native AOT" (Le plus gros changement)
>
>Depuis .NET 7 et 8, vous pouvez publier en **Native AOT** (Ahead-Of-Time).
>
>- **Avant :** "Autonome" (Self-contained) signifiait copier l'application + toutes les DLL du runtime .NET dans un dossier. Le JIT compilait encore le code à l'exécution.
>- **Maintenant :** Le code est compilé directement en **code machine pur** (binaire natif) lors de la publication.
>- **Résultat :** Un seul fichier exécutable, démarrage instantané, et pas de JIT sur la machine cible. C'est l'idéal pour un **Mac avec Apple Silicon**.
>
>### Changement du RID par défaut
>
>Depuis .NET 8, il y a eu une simplification des **RID** (Runtime Identifiers).
>
>- Auparavant, on utilisait des RID très précis comme `osx.12-arm64`.
>- Désormais, .NET préfère des RID plus "génériques" pour améliorer la compatibilité, comme **`osx-arm64`** (pour votre Mac) ou **`linux-x64`**.
>- Publication en un seul fichier (Single File)
>
>L'option `PublishSingleFile` a été grandement améliorée. Sous .NET 6, certains fichiers pouvaient encore rester à côté. Sous .NET 8/9, l'extraction est plus propre et la compression est bien meilleure.
>
>### Nouvelles options courantes (Voir [[#Tableau 16-1.5 Mise à jour .NET 8 et 9|Tableau 16-1.5]])
>
>### Exemple pour les Mac avec Apple Sillicon
>
>Si vous voulez créer une version de votre `CarClient` que vous pourriez donner à un ami sur Mac (qui n'a pas .NET installé), vous taperiez :
>```bash
>dotnet publish -c Release -r osx-arm64 --self-contained true -p:PublishSingleFile=true
>```
>
>---
>*Sous .NET 6, une application "Hello World" autonome pesait environ 60-70 Mo. Avec les nouvelles options de **Trim** et de **AOT** des versions récentes, on peut descendre en dessous de 10-15 Mo.*

##### Tableau 16-1: Quelque options pour publier une application

| Options / Propriété MSBuild                                                              | Description                                                                                                                                                                                          |
| ---------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `--use-current-runtime`                                                                  | Utiliser l'environnement d'exécution actuel comme environnement d'exécution cible.                                                                                                                   |
| `-o, --output <OUTPUT_DIR>`                                                              | Répertoire de sortie dans lequel placer les artefacts publiés.                                                                                                                                       |
| `--self-contained`                                                                       | Publiez le runtime .NET avec votre application afin qu'il ne soit pas nécessaire de l'installer sur la machine cible. Cette option est activée par défaut si un identifiant de runtime est spécifié. |
| `--no-self-contained`                                                                    | Publiez le runtime .NET avec votre application afin qu'il ne soit pas nécessaire de l'installer sur la machine cible. Cette option est activée par défaut si un identifiant de runtime est spécifié. |
| `-r <IDENTIFIANT_RUNTIME>--runtime` <br>`<IDENTIFIANT_RUNTIME>`                          | L'environnement d'exécution cible pour la publication. Ce paramètre est utilisé lors de la création d'un déploiement autonome. Par défaut, l'application publiée dépend d'un framework.              |
| `-c debug \| release--configuration debug \| release`                                    | Configuration de publication. La valeur par défaut pour la plupart des projets est `debug`.                                                                                                          |
| `-v, --verbosity <d\|detailed\|diag\|`<br>`diagnostic\|m\|minimal\|n\|normal\|q\|quiet>` | Définissez le niveau de verbosité de MSBuild. Les valeurs autorisées sont : q/silencieux, m/minimal, n/normal, d/détaillé et diag/diagnostic.                                                        |

>[!info] Toute les commande affiché prennent une seule ligne !

##### Tableau 16-1.5: Mise à jour .NET 8 et 9

| Options / Propriété MSBuild                   | Description                                                                                           |
| --------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| `-p:PublishSingleFile=true`                   | Regroupe l'application et ses dépendances dans un seul fichier exécutable.                            |
| `-p:PublishTrimmed=true`                      | Supprime le code IL inutilisé afin de réduire considérablement la taille du fichier de l'application. |
| `-p:PublishAot=true`                          | Compile en AOT natif pour une exécution directe du code natif sans JIT.                               |
| `-p:PublishReadyToRun=true`                   | Précompile les modules pour un démarrage plus rapide de l'application.                                |
| `-p:EnableCompressionInSingleFile=true`       | Compresse le fichier binaire unique pour réduire la taille du déploiement.                            |
| `p:IncludeNativeLibrariesForSelfExtract=true` | Inclut des bibliothèques natives dans un seul fichier pour gérer les dépendances natives.             |
| **`--no-build`**                              | Ignore l'étape de compilation, en supposant qu'une compilation préalable ait eu lieu.                 |

## Publication d'applications dépendantes du framework

**Lorsqu'aucun identifiant d'exécution n'est spécifié, le déploiement dépendant du framework est le comportement par défaut de la commande `dotnet publish`**. Pour empaqueter votre application et les fichiers requis, il vous suffit d'exécuter la commande suivante dans l'interface de ligne de commande :

`dotnet publish`

>[!note] 
>**Avant .NET 8**: La commande `publish` utilise la configuration par défaut de votre projet, qui est généralement `Debug`
>
>**Après .NET 8**: Microsoft a décidé que `publish` devait logiquement produire une version prête pour la production. Par conséquent, **`dotnet publish` utilise désormais `Release` par défaut**.

>[!warning] Le petit piège depuis .NET 8/9
>
>Bien que ce soit le comportement par défaut, le SDK moderne est devenu "intelligent". Si tu as déjà défini un **RID** (comme `osx-arm64`) dans ton fichier `.csproj`, la commande `dotnet publish` pourrait décider toute seule de passer en mode **autonome**.

**Cela place votre application et ses fichiers de support** (six fichiers au total) **dans le répertoire `bin\Debug\netVersion\publish`**. ==En examinant les fichiers ajoutés à ce répertoire, vous verrez deux fichiers *.dll*== (*CarLibrary.dll* et *CSharpCarClient.dll*) ==qui contiennent tout le code de l'application==. 

Pour rappel:

- le fichier *CSharpCarClient.exe* est une version packagée de *dotnet.exe* configurée pour lancer *CSharpCarClient.dll*. 
- Le fichier *CSharpCarClient.deps.json* liste toutes les dépendances de l'application 
- Le fichier *CSharpCarClient.runtimeconfig.json* spécifie le framework cible et sa version.
- Le dernier fichier est le fichier de débogage de *CSharpCarClient*.

Pour créer une version de publication (qui sera placée dans le répertoire `bin\release\netVersion\publish`), saisissez la commande suivante :

>Voir Note sur le comportement par défaut de la commande (avant et après .NET 8)

```bash
dotnet publish -c release
```

## Publication d'applications autonomes

À l'instar des déploiements dépendants d'un framework, **==les déploiements autonomes incluent l'intégralité du code de votre application et des assemblies référencés, mais également les fichiers .NET Runtime requis par votre application==**. **Pour publier votre application de manière autonome, vous devez inclure un identifiant d'exécution dans la commande (==ou dans votre fichier projet==)**.

**Saisissez la commande CLI suivante, qui place la sortie dans un dossier nommé** `selfcontained`.

>[!important] Différencier selon les Mac
>
>- `osx-arm64`: Mac possédant un processeur **Apple Silicon** (M1, M2,...)
>- `osx-x64`: Mac possédant un processeur Intel (Avant 2020)


```bash
dotnet publish -r osx-arm64 -o selfcontained --self-contained true
```

>[!note] 
>**En .NET 6** : 
>l’option `--self-contained true` n’est plus nécessaire. Si un identificateur d’exécution est spécifié, la commande de publication utilisera le processus autonome.
>
>**Depuis .NET 8 (et donc en .NET 10) :**  
Microsoft a fait marche arrière. Désormais, même si tu spécifies un RID, le comportement par défaut est resté **Framework-dependent** (dépendant du framework).

**Cela place votre application et ses fichiers de support** ($226$ fichiers au total) **dans le répertoire autonome**. **==Si vous avez copié ces fichiers sur un autre ordinateur dont l'identifiant d'exécution correspond, vous pouvez exécuter l'application même si le runtime .NET 6 n'est pas installé.==**

>[!tip]- Le fait d'avoir 2 fois les mêmes fichier est le comportement normal de la commande. (Gemini)
>1. Le dossier `osx-arm64/` : L'aire de préparation
>
>C'est le résultat de la commande **`dotnet build`** (appelée automatiquement par publish).
>
>- Pour fonctionner en mode autonome (`self-contained`), le compilateur doit d'abord rassembler toutes les DLL du runtime .NET dans le dossier de sortie pour vérifier que tout est compatible.
>- C'est ton **environnement de développement** pour cette architecture spécifique.
>
>2. Le dossier `publish/` : Le colis de livraison
>
>C'est le résultat spécifique de la commande **`dotnet publish`**.
>
>- Le SDK **copie** les fichiers depuis le dossier parent vers le dossier `publish`.
>- **Pourquoi copier au lieu de simplement utiliser le dossier parent ?** Parce que l'étape de publication peut inclure des transformations que le dossier de build ne doit pas subir :
>    - Le **Trimming** (suppression du code mort dans les DLL).
>    - La **Compression**.
>    - La signature des fichiers pour l'App Store.
>    - Le regroupement en **SingleFile**.
>
>3. Est-ce "normal" en .NET 10 ?
>
>Oui, c'est le fonctionnement historique du SDK. .NET privilégie la **sécurité du déploiement** (avoir un dossier `publish` parfaitement propre et isolé) au détriment de l'espace disque pendant la phase de build.
>
>---
>*Pou arrêter d'avoir ces 200+ fichiers en double qui polluent ton arborescence `bin`, voir la section suivante*

### Publication d'applications autonomes dans un seul fichier

**Dans la plupart des cas, déployer $226$ fichiers** (pour une application qui affiche quelques lignes de texte) **n'est probablement pas la méthode la plus efficace pour mettre votre application à disposition des utilisateurs**. Heureusement, .NET 5 a considérablement amélioré la possibilité de ***==publier votre application et les fichiers d'exécution multiplateformes dans un seul fichier==***. **Ce processus est encore amélioré avec .NET 6, éliminant ainsi la nécessité d'avoir les bibliothèques natives en dehors de l'exécutable unique, ce qui était nécessaire avec .NET 5 pour la publication sous Windows.**

La commande suivante crée un package de déploiement autonome en un seul fichier pour les systèmes d'exploitation MacOS 64 bits et place les fichiers résultants dans un dossier nommé `singlefile` .

```bash
dotnet publish -r osx-arm64 -o singlefile --self-contained true -p:PublishSingleFile=true
```

**Lorsque vous examinez les fichiers créés, vous trouverez un seul exécutable** (*CSharpCarClient.exe*) **et un fichier de débogage** (*CSharpCarClient.pdb*). ***==Alors que le processus de publication précédent générait un grand nombre de petits fichiers, la version unique de==*** *CSharpCarClient.exe* ***==pèse 60 Mo !==*** **La création de cette publication unique a regroupé les 226 fichiers dans un seul. La réduction du nombre de fichiers a été compensée par une augmentation de leur taille.**

**==Pour inclure le fichier de symboles de débogage dans le fichier unique (afin d'en faire un fichier véritablement unique), mettez à jour votre commande comme suit :==**

```bash
dotnet publish -r osx-arm64 -o singlefile --self-contained true -p:PublishSingleFile=true -p:DebugType=embedded
```

**Vous avez maintenant un seul fichier contenant tout, mais il est encore assez volumineux**. ==Une option pour réduire sa taille est la compression==. La sortie peut être compressée pour gagner de l'espace, *==mais cela risque d'affecter encore plus le temps de démarrage de votre application==*. Pour activer la compression, utilisez la commande suivante (sur une seule ligne) :

```bash
dotnet publish -r osx-arm64 -o singlefile --self-contained true -p:PublishSingleFile=true -p:DebugType=embedded -p:EnableCompressionInSingleFile=true:
```

**L'équipe .NET a travaillé sur le trimming des fichiers lors du processus de publication ces dernières années**. ***==Avec .NET 6, cette fonctionnalité est désormais disponible et prête à l'emploi==***. **==Le processus d'optimisation des fichiers détermine ce qui peut être supprimé du runtime en fonction de ce que votre application utilise==**. Si ce processus est maintenant opérationnel, c'est notamment parce que **l'équipe a annoté le runtime lui-même afin de supprimer les faux avertissements qui étaient fréquents dans .NET 5**. Nouveauté de .NET 6 : **le processus d'optimisation ne se contente pas de rechercher les assemblies pouvant être supprimés ; il recherche également les membres inutilisés**. Utilisez la commande suivante pour optimiser le fichier de sortie unique (sur une seule ligne) :

```bash
dotnet publish -r osx-arm64 -o singlefile --self-contained true -p:PublishSingleFile=true -p:DebugType=embedded -p:EnableCompressionInSingleFile=true -p:PublishTrimmed=true
```

**La dernière étape de ce processus consiste à publier votre application prête à l'emploi**. **==Cela peut améliorer le temps de démarrage, car une partie de la compilation JIT est effectuée à l'avance (AOT) lors de la publication.==**

```bash
dotnet publish -r osx-arm64 -o singlefile --self-contained true -p:PublishSingleFile=true -p:DebugType=embedded -p:EnableCompressionInSingleFile=true -p:PublishTrimmed=true -p:PublishReadyToRun=true
```

>[!info] l'option `PublishAot` est encore plus radical car il convertit tout en code machine.
>Il faut donc choisir soit `PublishAot` ou `PublishReadyToRun`. les deux ne sont pas cumulable.

La taille finale de notre application est de $11$ Mo, bien inférieure aux $60$ Mo initiaux. Enfin, ***==tous ces paramètres peuvent être configurés dans le fichier projet de votre application==***, comme ceci :

```xml
<PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net10.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>disable</Nullable>
    <PublishSingleFile>true</PublishSingleFile>
    <SelfContained>true</SelfContained>
    <RuntimeIdentifier>osx-arm64</RuntimeIdentifier>
    <PublishTrimmed>true</PublishTrimmed>
    <DebugType>embedded</DebugType>
    <EnableCompressionInSingleFile>true</EnableCompressionInSingleFile>
    <PublishReadyToRun>true</PublishReadyToRun>
  </PropertyGroup>
```

Une fois ces valeurs définies dans le fichier projet, la ligne de commande devient beaucoup plus courte :

```bash
dotnet publish -o singlefilefinal2
```

# Comment .NET localise les assemblies

Jusqu'à présent dans ce livre, **tous les assemblies que vous avez créés étaient directement liés (==à l'exception de l'exemple NuGet que vous venez de terminer==)**. **Vous avez ajouté soit une référence de projet, soit une référence directe entre projets**. ***==Dans ces cas (ainsi que dans l'exemple NuGet), l'assembly dépendant a été copié directement dans le répertoire cible de l'application cliente==***. Localiser l'assembly dépendant ne pose aucun problème, puisqu'il se trouve sur le disque à proximité de l'application qui en a besoin.

**Mais qu'en est-il du .NET Framework ? Où est-il situé ? Les versions précédentes de .NET installaient les fichiers du framework dans le Global Assembly Cache (GAC), et toutes les applications .NET Framework savaient où les localiser.**

**Cependant, le GAC empêche la présence simultanée de plusieurs fichiers dans .NET Core**. Il n'existe donc pas de dépôt unique pour les fichiers d'exécution et le framework. **Les fichiers qui composent le framework sont installés ensemble dans `C:\Program Files\dotnet` (sous Windows), séparés par version (`/usr/local/share/dotnet` pour Linux/macOS)**. **==En fonction de la version de l'application==** (telle que spécifiée dans le fichier *.csproj*), **==les fichiers d'exécution et de framework nécessaires sont chargés pour une application à partir du répertoire de la version spécifiée.==**

>[!warning] La mort du GAC (Global Assembly Cache)
>
>Dans le passé (avant 2016), le **GAC** était un dossier système "sacré" où l'on installait les DLL.
>
>- **Le problème :** Si une mise à jour cassait une DLL dans le GAC, **toutes** les applications du PC plantaient. C'était l'enfer des dépendances ("DLL Hell").
>- **La solution moderne :** .NET Core (puis .NET 6/10) a supprimé le GAC. Chaque application est désormais isolée.

**Plus précisément, lorsqu'une version du runtime est lancée, l'hôte du runtime fournit un ensemble de chemins de sondage qu'il utilisera pour trouver les dépendances d'une application**. **==Il existe cinq propriétés de sondage==** (chacune d'entre elles optionnelle), comme indiqué dans le [[#Tableau 16-2|Tableau 16-2]].

>[!warning] Ce qui a changé (La nuance moderne)
>
>- **Priorité au fichier `.deps.json`** : Sous .NET 10, le runtime fait beaucoup moins de "devinettes". Au lieu de scanner des dossiers au hasard (ce qui ralentit le démarrage), il lit le fichier JSON pour savoir exactement où se trouve `CarLibrary.dll`.
>- **Le "Shared Framework"** : Sur ton Mac, si tu n'es pas en mode `self-contained`, le chemin de sondage pointe prioritairement vers `/usr/local/share/dotnet/shared/`. C'est là que vivent les DLL communes à toutes les applications.
>- **Sécurité accrue** : .NET restreint désormais les chemins de sondage pour éviter qu'une DLL malveillante placée dans un dossier temporaire ne soit chargée à la place de la tienne.

>[!tip]- Le processus de localilsation
>1. **Dossier local :** Il cherche d'abord `CarLibrary.dll` juste à côté de l'exécutable.
>2. **Shared Framework :** S'il ne trouve pas une DLL (comme `System.Console`), il va voir dans le dossier `/usr/local/share/dotnet/shared/` correspondant à la version définie dans ton fichier `.runtimeconfig.json`.
>3. **Dossier NuGet :** Pour les paquets tiers, il peut aller voir dans ton cache local `~/.nuget/packages`.

##### Tableau 16-2: Propriétés de sondage des applications

| Option                          | Description                                                                                      |
| ------------------------------- | ------------------------------------------------------------------------------------------------ |
| `TRUSTED_PLATFORM_ASSEMBLIES`   | Liste des chemins d'accès aux fichiers d'assemblage de la plateforme et de l'application         |
| `PLATFORM_RESOURCE_ROOTS`       | Liste des chemins de répertoire dans lesquels rechercher les ensembles de ressources satellites  |
| `NATIVE_DLL_SEARCH_DIRECTORIES` | Liste des chemins de répertoires dans lesquels rechercher les bibliothèques non gérées (natives) |
| `APP_PATHS`                     | Liste des chemins de répertoire dans lesquels rechercher les assemblys gérés                     |
| `APP_NI_PATHS`                  | Liste des chemins de répertoire dans lesquels rechercher les images natives des assemblys gérés  |

Pour afficher les chemins d'accès par défaut, créez une application console .NET nommée *FunWithProbingPaths*. Mettez à jour le fichier *Program.cs* en y ajoutant les instructions suivantes :

```cs
Console.Title = "Fun with Probing Paths";
Console.WriteLine("*** Fun with Probing Paths ***");
Console.WriteLine($"TRUSTED_PLATFORM_ASSEMBLIES: ");

// Utilisez ':' pour les peteformes différentes de Windows
var list = AppContext
    .GetData("TRUSTED_PLATFORM_ASSEMBLIES")
    .ToString()
    .Split(':');
foreach (var dir in list)
{
    Console.WriteLine(dir);
}
Console.WriteLine();
Console.WriteLine(
    $"PLATFORM_RESOURCE_ROOTS: {AppContext.GetData("PLATFORM_RESOURCE_ROOTS")}"
);
Console.WriteLine();
Console.WriteLine(
    $"NATIVE_DLL_SEARCH_DIRECTORIES: {AppContext.GetData("NATIVE_DLL_SEARCH_DIRECTORIES")}"
);
Console.WriteLine();
Console.WriteLine($"APP_PATHS: {AppContext.GetData("APP_PATHS")}");
Console.WriteLine();
Console.WriteLine($"APP_NI_PATHS: {AppContext.GetData("APP_NI_PATHS")}");
Console.WriteLine();
Console.ReadLine();
```

Lorsque vous exécutez cette application, la plupart des valeurs proviennent de la variable `TRUSTED_PLATFORM_ASSEMBLIES`. Outre l'assembly créé pour ce projet dans le répertoire cible, vous verrez une liste de bibliothèques de classes de base provenant du répertoire d'exécution actuel, `C:\Program Files\dotnet\shared\Microsoft.NETCore.App\6.0.0` (votre numéro de version peut être différent).

**Chaque fichier directement référencé par votre application est ajouté à la liste, ainsi que tous les fichiers d'exécution nécessaires à son fonctionnement**. ==La liste des bibliothèques d'exécution est alimentée par un ou plusieurs fichiers *.deps.json* chargés avec le runtime .NET==. **==Plusieurs fichiers de ce type se trouvent dans le répertoire d'installation du SDK==** (utilisés pour la compilation du logiciel) **==et du runtime==** (utilisés pour l'exécution du logiciel). ==Dans notre exemple simple, le seul fichier utilisé est *Microsoft.NETCore.App.deps.json*.==

>[!info]
**Diagnostic du Runtime :** Les propriétés comme `APP_PATHS` sont souvent vides dans les applications .NET modernes car la localisation des assemblies est désormais gérée par le fichier de description des dépendances (`.deps.json`) plutôt que par des variables d'environnement dynamiques.

**À mesure que votre application gagne en complexité, la liste des fichiers dans `TRUSTED_PLATFORM_ASSEMBLIES` s'allonge**. Par exemple, ==si vous ajoutez une référence au package `Microsoft.EntityFrameworkCore`, la liste des assemblies requis s'allonge==. Pour le démontrer, saisissez la commande suivante dans la console du Gestionnaire de packages (dans le même répertoire que le fichier *.csproj) :

```bash
dotnet add FunWithProbingPath package Microsoft.EntityFrameworkCore
```

**Une fois le package ajouté, relancez l'application et constatez le nombre accru de fichiers listés.** **==Bien que vous n'ayez ajouté qu'une seule nouvelle référence, le package `Microsoft.EntityFrameworkCore` possède des dépendances, qui sont ajoutées à la liste des fichiers de confiance.==**

# Résumé du chapitre

**Ce chapitre a examiné le rôle des bibliothèques de classes .NET** (également appelées *dll* .NET). Comme vous l'avez vu, **==les bibliothèques de classes sont des fichiers binaires .NET contenant une logique réutilisable dans différents projets.==**

**Vous avez appris en détail le partitionnement des types dans les espaces de noms .NET et la différence entre .NET et .NET Standard, commencez la configuration d'applications et vous avez approfondi la composition des bibliothèques de classes**. Vous avez ensuite ***==appris à publier des applications console .NET. Enfin, vous avez appris à empaqueter vos applications à l'aide de NuGet.==***

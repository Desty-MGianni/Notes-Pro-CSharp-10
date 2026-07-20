---
title: "Chapitre 2: Construire des Applications C#"
publish: true
---

# <big><big><big><b><font color =green>Construire des Applications C# </font></b></big></big></big>

Pour commencer à programmer en C#, il faut installer le *Kit de Développement Logiciel (SDK)* que 'on veut / à besoin).

>  `.NET 10` est une version LTS tandis que `.NET 11` est une version STS.

Pour télécharger le SDK, il y a deux façons:
1. Allez sur www.dot.net et télécharger "à la main" 
2. avec Homebrew (MacOS): ```brew install --cask dotnet```

>**le SDK installe les runtimes .*NET Core* et *ASP.NET Core* pour le système d'exploitation ainsi que le processeur de la machine.**

>[!Info]-
>Sur windows, le *SDK* installera aussi le runtime *.NET Desktop*. 
>Plus d'informations plus loin dans ce chapitre.

## Comprendre le système de numérotation des versions .NET

Au moment de la traduction de cet article, le *SDK .NET* est à la version *10.0.201* (avril 2026). **Les deux premiers chiffres (10.0) indiquent la version la plus élevée du runtime que vous pouvez cibler**. Dans ce cas, il s'agit de 10.0. Cela signifie que le SDK prend également en charge le développement pour une version inférieure du runtime, telle que .NET 5 ou .NET Core 3.1. ==Le chiffre suivant (2) correspond à la bande de fonctionnalités trimestrielle. Comme nous sommes actuellement dans le deuxième trimestre de l'année depuis la sortie, il s'agit d'un 2==. Les deux derniers chiffres (01) indiquent la version du correctif. Cela est un peu plus clair si vous ajoutez un séparateur dans la version dans votre esprit et que vous considérez la version actuelle comme *10.0.2.01*.

## Confirmation de l'installation de .NET

Pour confirmer l'installation du SDK et des runtimes, **ouvrez une fenêtre de commande et utilisez l'interface de ligne de commande (CLI) .NET**, ```dotnet.exe``` (**juste `dotnet` pour les versions plus récente**). Le CLI propose des options et des commandes SDK. Les commandes permettent notamment de créer, de compiler, d'exécuter et de publier des projets et des solutions. Vous trouverez des exemples de ces commandes plus loin dans ce document. Dans cette section, nous allons examiner les options SDK, qui sont au nombre de quatre, comme le montre le [[#Tableau 2-1 Options de .NET CLI SDK|Tableau 2-1]].

###### Tableau 2-1: Options de .NET CLI SDK

| Option            | Description                             |
| ----------------- | --------------------------------------- |
| `--version`       | Affiche la version de .NET SDK utilisée |
| `--info`          | Affiche les information .NET            |
| `--list-runtimes` | Affiche les runtimes installés          |
| `--list-sdks`     | Affiche les SDK installés               |

L'option `--version` affiche la version la plus récente du SDK installée sur votre ordinateur ou la version spécifiée dans un fichier *global.json* situé dans votre répertoire actuel ou au-dessus. Pour vérifier la version actuelle du SDK .NET installée sur votre ordinateur, entrez la commande suivante :

```bash
dotnet --version
```

>Pour ce livre, le résultat doit être 6.0.100 (ou supérieur).

Pour afficher tous les runtimes .NET Core installés sur votre ordinateur, entrez la commande suivante :

```bash
dotnet --list-runtimes
```

**Il existe trois runtimes différents** :

- `Microsoft.AspNetCore.App` (pour créer des applications ASP.NET Core)
- `Microsoft.NETCore.App` (le runtime de base pour .NET)
- `Microsoft.WindowsDesktop.App` (pour créer des applications WinForms et WPF)

Si vous utilisez un système d'exploitation Windows, chacun d'entre eux doit être en version 6.0.0 (ou supérieure). Si vous n'utilisez pas Windows, vous n'aurez besoin que des deux premiers, `Microsoft.NETCore.App` et `Microsoft.AspNetCore.App`, qui doivent également être en version 6.0.0 (ou supérieure).

Enfin, pour afficher tous les SDK installés, entrez la commande suivante :

```bash
dotnet --list-sdks
```

>Là encore, la version doit être 6.0.100 (ou supérieure).

### Vérification des mises à jour

Nouveauté de .NET 6, **l'interface CLI dispose d'une nouvelle commande qui vérifie les mises à jour disponibles pour les versions installées des SDK et des runtimes .NET/.NET Core**. Cette commande est rétrocompatible, elle vérifie donc également les mises à jour pour .NET Core 3.1. Elle vous informe également si l'un des SDK ou runtimes installés n'est plus pris en charge (comme les versions 2.x). Pour vérifier les versions, entrez la commande suivante :

```bash
dotnet sdk check
```

>[!Attention]
>La commande ne mettra à jour aucune des versions pour vous, elle se contentera de signaler les statuts.
>
>Pour mettre à jour, suivez la même procédure décrite ci-dessus pour télécharger et installer la/les nouvelles versions.

### Utiliser une version antérieure du SDK .NET (Core)

Il peut parfois être utile de s'assurer que vous utilisez une version antérieure du SDK .NET. Par exemple, vous développez vos applications de production à l'aide de .NET 6. Une première version candidate de .NET 7 est disponible, et vous souhaitez commencer à l'essayer sans mettre en péril votre travail de production. Bien que Microsoft affirme que vous pouvez développer des versions précédentes d'applications .NET avec un SDK plus récent, de nombreux développeurs et organisations ne sont pas à l'aise avec les versions candidates, et encore moins avec les versions bêta/prévisualisation précoces.

==Si vous devez associer votre projet à une version antérieure du SDK .NET, vous pouvez le faire à l'aide d'un fichier *global.json*==. Pour créer ce fichier, vous pouvez utiliser cette commande, qui associe le dossier actuel et tous les sous-dossiers à la version 5.0.400 du SDK :

```bash
dotnet new globaljson --sdk-version 5.0.400
```

Cela va créer un fichier *global.json* qui ressemble à ca:

```json
{
  "sdk": {
	"version": "5.0.400"
  }
}
```

L'exécution de `dotnet --version` dans ce répertoire (ou dans n'importe quel sous-répertoire) renverra 5.0.400.

>[!Attention] Pour que cette explication fonctionne, il faut avoir télécharger la version voulue de .NET avant.

#### Utiliser une version antérieure du SDK .NET sans avoir à l'installer

Il y a deux façons simples pour pouvoir utiliser, par exemple, un `.NET 10.0.201` installé dans la machine peut compiler et exécuter un programme qui cible `.NET 8.0.409.

La première méthode consiste à modifier le fichier project (*.csproj*) avec la version de .NET ciblé ainsi qu'une option `RollForward`.

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>
  
  <PropertyGroup>
    <RollForward>Major</RollForward>
  </PropertyGroup>
</Project>
```

La deuxième façon utilise les mêmes options, la différences est que l'on utilise le même fichier *global.json* vu précédemment avec l'option `RollForward` ajouté:

```json
{
  "sdk": {
    "version": "8.0.409",
    "rollForward": "Major"
  }
}
```

>[!tip] Voir [documentation Microsoft](https://learn.microsoft.com/en-us/dotnet/core/versions/selection#values) pour plus de détails

# Création d'applications .NET Core avec Visual Studio

>[!Failure] Visual Studio 2022 (ou 2026) ne sont disponible que sur la plateforme **Windows**

>[!Attention] 
>À l'heure de la 1ère relecture (Avril 2026), Visual Studio 2026 est sorti. Cependant, les exemples et explications resteront pour la version 2022.
>
>Comme j'utilise un Mac, je n'ai pas accès à Visual Studio, seulement Visual Studio Code.

3 versions disponible pour Visual Studio 2022:

- Visual Studio 2022 Community (gratuit)
- Visual Studio 2022 Professional (payant)
- Visual Studio 2022 Enterprise (payant)

Les éditions Community et Professional sont essentiellement identiques. La différence la plus significative réside dans le modèle de licence. La licence Community est destinée à une utilisation open source, académique et dans les petites entreprises. Les éditions Professional et Enterprise sont des produits commerciaux dont la licence couvre tout type de développement, y compris le développement en entreprise. Comme on peut s'y attendre, l'édition Enterprise offre de nombreuses fonctionnalités supplémentaires par rapport à l'édition Professional.

>[!Note]-
>Voir www.visualstudio.com pour les modalités de liscences.
>>[!warning] Le lien pour Visual Studio 2022 ne fonctionne plus.

Toutes les éditions de Visual Studio sont livrées avec des éditeurs de code sophistiqués, des débogueurs intégrés, des concepteurs d'interface graphique pour les applications de bureau, et bien plus encore. Comme elles partagent toutes un ensemble commun de fonctionnalités, la bonne nouvelle est qu'il est facile de passer de l'une à l'autre et de se sentir à l'aise avec leur fonctionnement de base.

Pour ce livre, vous devrez installer les charges de travail suivantes :

- Développement .NET pour ordinateurs de bureau
- ASP.NET et développement Web
- Stockage et traitement des données

Dans l'onglet « Composants individuels », sélectionnez également Class Designer et Git pour Windows (tous sous « Outils de code »). Une fois que vous les avez tous sélectionnés, cliquez sur Installer. Vous disposerez ainsi de tout ce dont vous avez besoin pour travailler sur les exemples présentés dans ce livre.

>[!Info] Moi, Gianni, j'ai passer la création d'un nouveau projet sur Visual Studio car je ne l'utilise pas et c'est très simple.

Une fois le projet créé, vous verrez que le fichier de code C# initial (nommé *Program.cs*) s'est ouvert dans l'éditeur de code. Le modèle initial ne contient qu'un *commentaire (la ligne commençant par //)* et une seule ligne de code qui affiche `"Hello, World!"` dans la console :

```cs
// See https://aka.ms/new-console-template for more information
Console.WriteLine("Hello, World!");
```

Ces deux lignes de code sont appelées *instructions de niveau supérieur (top level statement)* et ==servent de point d'entrée dans l'application. Le chapitre suivant traite en détail des instructions de niveau supérieur et des points d'entrée dans l'application==. Pour l'instant, sachez simplement que c'est là que commence l'exécution de votre application et qu'elle se termine lorsque toutes les lignes ont été exécutées. 

Remplacez le commentaire et la ligne de code unique par ce qui suit :

```cs
// Set up Console UI (CUI)
Console.Title = "My Rocking App";
Console.ForegroundColor = ConsoleColor.Yellow;
Console.BackgroundColor = ConsoleColor.Blue;
Console.WriteLine("*************************************");
Console.WriteLine("***** Welcome to My Rocking App *****");
Console.WriteLine("*************************************");
Console.BackgroundColor = default;

// Wait for Enter key to be pressed.
Console.ReadLine();
```

> [!Info]- Remarque:
>  Lorsque vous tapez, Visual Studio tente de compléter les mots à votre place. Cette fonctionnalité s'appelle **IntelliSense** (aide à la complétion de code) et est intégrée à Visual Studio et Visual Studio Code.

Ici, vous utilisez la classe `Console` définie dans l'espace de noms `System`. L'espace de noms `System` est inclus dans les instructions *using implicites globales*, il n'est donc pas nécessaire de le spécifier explicitement. Ce programme ne fait rien de très intéressant, mais notez l'appel final à `Console.ReadLine()`. Il sert simplement à garantir que l'utilisateur doit appuyer sur une touche pour fermer l'application.

### Modification du framework .NET Core cible

Lors de la création de ce projet, vous avez sélectionné la version de .NET que vous souhaitiez utiliser. Si vous avez choisi la mauvaise version (ou si vous souhaitez la modifier pour une autre raison), double-cliquez sur le nom du projet dans l'Explorateur de solutions. Cela ouvre le fichier de projet dans l'éditeur (cette fonctionnalité a été introduite avec Visual Studio 2019 et .NET Core). Vous pouvez également modifier le fichier de projet en cliquant avec le bouton droit sur le nom du projet dans l'Explorateur de solutions et en sélectionnant « Modifier le fichier de projet ». Vous verrez alors s'afficher ce qui suit :

```xml
<Project Sdk="Microsoft.NET.Sdk">
    <PropertyGroup>
        <OutputType>Exe</OutputType>
        <TargetFramework>net6.0</TargetFramework>
        <ImplicitUsings>enable</ImplicitUsings>
        <Nullable>enable</Nullable>
    </PropertyGroup>
</Project>
```

Pour changer la version de .NET, par exemple la version 5, il faut simplement changer la valeur de `TargetFramewrok`:

```xml
<Project Sdk="Microsoft.NET.Sdk">
    <PropertyGroup>
        ...
        <TargetFramework>net5.0</TargetFramework>
        ...
    </PropertyGroup>
</Project>
```

### Utilisation des fonctionnalités de C# 10

Dans les versions de .NET et du .NET Framework, la version de C# prise en charge par un projet pouvait être modifiée. Depuis la sortie de .NET Core 3.0 (et de chaque version .NET ultérieure), la version de C# utilisée est liée à la version .NET Core/.NET. Pour les projets .NET 6.0, la version du langage est verrouillée sur C# 10. Le [[#Tableau 2-2 Versions de C et Les Framework associé|Tableau 2-2]] répertorie les frameworks cibles (.NET, .NET Core, .NET Standard et .NET Framework) et la version C# par défaut utilisée.

###### Tableau 2-2: Versions de C# et Les Framework associé

| Framework Cible | Version | Version par défaut de C# |
| --------------- | ------- | ------------------------ |
| .NET            | 6.x     | C# 10                    |
| .NET            | 5.x     | C# 9                     |
| .NET Core       | 3.x     | C# 8                     |
| .NET Core       | 2.x     | C# 7.3                   |
| .NET Standard   | 2.1     | C# 8                     |
| .NET Standard   | 2.0     | C# 7.3                   |
| .NET Standard   | 1.x     | C# 7.3                   |
| .NET Framework  | tous    | C# 7.3                   |

### Exécution et déboggage de votre projet

Pour exécuter votre programme et voir le résultat, appuyez sur la combinaison de touches *Ctrl + F5*. Une fois que vous aurez fait cela, vous verrez une *fenêtre de terminal*  apparaître à l'écran avec votre message personnalisé (et coloré). **Sachez que lorsque vous « exécutez » votre programme avec *Ctrl+F5*, vous contournez le débogueur intégré**.

> [!Note]-
> Il est possible d'exécuter le programme dans le terminal: ```dotnet run``` quand on est dans le même   dossier ou se situe le ficher *.csproj*. ==Cette commande va automatiquement fabriquer (*build*) le projet==.
> 
> Quand on ne se situe pas au même niveau que le project: 
> ```dotnet run --project <chemin_vers_projet>```

Si vous avez besoin de débogguer votre code (ce qui sera certainement important lors de la création de programmes plus volumineux), la première étape consiste à définir des points d'arrêt au niveau de l'instruction de code que vous souhaitez examiner (==Ajouter des points rouges à coté des numéros de lignes.==)

Si vous appuyez maintenant sur la touche *F5* votre programme s'arrêtera à chaque point d'arrêt. Comme vous pouvez vous y attendre, vous pouvez interagir avec le déboggueur à l'aide des différents boutons de la barre d'outils et des options de menu de l'IDE. Une fois que vous avez évalué tous les points d'arrêt, l'application se fermera une fois les instructions terminées.

> [!Info]-
> Les IDE Microsoft disposent de débogueurs sophistiqués, et vous découvrirez diverses techniques au fil des chapitres à venir. Pour l'instant, sachez simplement que lorsque vous êtes en session de débogage, un grand nombre d'options utiles apparaissent dans le menu Déboguer. Prenez le temps de le vérifier par vous-même.

### Utilisation de l'Explorateur de solutions

Si vous regardez à droite de l'éditeur de texte, vous verrez la fenêtre Explorateur de solutions, qui affiche quelques éléments importants. Tout d'abord, ==remarquez que l'assistant de nouveau projet a créé une solution en même temps que le projet unique==. Cela peut prêter à confusion au début, car ils ont tous deux reçu le même nom (SimpleCSharpConsoleApp) **L'idée ici est qu'une « solution » peut contenir plusieurs projets qui fonctionnent tous ensemble**. Par exemple, votre solution peut inclure trois *bibliothèques de classes*, une application *WPF* et un service web *ASP. NET Core*. Les premiers chapitres de ce livre ne contiennent presque toujours qu'un seul exemple de code ; cependant, lorsque vous créerez des exemples plus complexes, vous verrez comment ajouter de nouveaux projets à votre solution initiale.

>[!Note]-
>Sachez que lorsque vous sélectionnez la solution dans la fenêtre Explorateur de solutions, le système de menus de l’EDI affiche des options différentes de celles proposées lorsque vous sélectionnez un projet. Si vous vous demandez où est passé un élément de menu, vérifiez que vous n’avez pas sélectionné le mauvais nœud par erreur.

### Utilisation de l'outil de diagramme de classes visuel

Visual Studio vous permet également de concevoir des classes et d'autres types (tels que des interfaces ou des délégués) de manière visuelle. Le diagramme de classes fournit des outils qui vous permettent de créer, d'afficher et de modifier les objets de votre projet et leurs relations avec d'autres objets. À l'aide de cet outil, vous pouvez ajouter (ou supprimer) visuellement des membres à (ou d'un) type et voir vos modifications reflétées dans le fichier C# correspondant. De plus, lorsque vous modifiez un fichier C# donné, les changements sont reflétés dans le diagramme de classes.

> [!info] Remarque : 
> cet ouvrage n'utilise l'outil Class Diagram (Diagramme de classes) qu'occasionnellement pour mettre en évidence certains concepts. Il est présenté ici par souci d'exhaustivité, et le choix de l'utiliser ou d'utiliser l'éditeur de texte vous appartient entièrement. La grande majorité des exemples utilisent l'éditeur de texte de Visual Studio/Visual Studio Code.

Pour accéder aux outils de conception visuelle de classes, la première étape consiste à insérer un nouveau fichier de diagramme de classes. Pour ce faire, sélectionnez le projet dans l'Explorateur de solutions, puis activez l'option de menu Projet ➤ Ajouter un nouvel élément et localisez le type Diagramme de classes. 

![[Figure 2.8.png|Insertion d'un fichier de diagramme de classes dans le projet actuel]]

Au départ, le concepteur sera vide ; cependant, vous pouvez glisser-déposer des fichiers depuis votre fenêtre Solution Explorer sur la surface ou cliquer avec le bouton droit sur la surface de conception pour créer de nouvelles classes. Pour commencer, créez une nouvelle classe dans votre projet en cliquant avec le bouton droit sur le projet et en sélectionnant Add ➤ Class. Dans la boîte de dialogue Ajouter un élément – `SimpleCSharpConsoleApp`, sélectionnez `Class` et nommez-la *Car.cs*.

![[Figure 2.9.png|La boîte de dialogue Ajouter un nouvel élément]]

> Cette fonctionnalité n'est disponible nativement que pour Visual Studio, pour VSCode, il faut installer une extension https://marketplace.visualstudio.com/items?itemName=pierre3.csharp-to-plantuml

Mettez à jour le code comme suit pour créer une classe `Car` (vous apprendrez tout sur les classes dans les prochains chapitres) :

```cs
namespace SimpleCSharpConsoleApp;
public class Car
{
	public string PetName { get; set; }
    public string Make { get; set; }
}
```

Après avoir enregistré le fichier, faites glisser le fichier *Car.cs* depuis l'Explorateur de solutions vers le diagramme de classes. Une fois que vous aurez fait cela, vous obtiendrez une représentation visuelle de la classe. Si vous cliquez sur l'icône en forme de flèche pour un type donné, vous pouvez afficher ou masquer les membres du type. Sous le diagramme de classes se trouve la fenêtre Détails de la classe, qui affiche les spécificités du diagramme de classes sélectionné. 

![[Figure 2.10.png|Le visualiseur de diagrammes de classes]]

>[!note] Notez que la barre d'outils du concepteur de classes vous permet d'affiner les options d'affichage de l'interface.

La fenêtre *Class Details* affiche non seulement les détails de l'élément actuellement sélectionné dans le diagramme, mais vous permet également de modifier les membres existants et d'en insérer de nouveaux à la volée.

La boîte à outils *Class Designer* vous permet d'insérer visuellement de nouveaux types (et de créer des relations entre ces types) dans votre projet. (Notez qu'un diagramme de classes doit être actif pour afficher cette boîte à outils.) L'EDI crée alors automatiquement les nouvelles définitions de types C# en arrière-plan.

![[Figure 2.11.png|La boîte à outils du designer de classe]]

À titre d'exemple, faites glisser une nouvelle classe depuis la boîte à outils du concepteur de classes vers votre concepteur de classes. Nommez cette classe `Make` avec un accès `public` et sélectionnez « *Créer un nouveau fichier* ». Cela créera un nouveau fichier C# nommé *Make.cs* qui sera automatiquement ajouté à votre projet. Ensuite, dans la fenêtre *Détails de la classe*, ajoutez une propriété de type `public string` nommée `Name`.

![[Figure 2.12.png|Ajout d'une propriété via la fenêtre Détails de la classe]]

Si vous consultez maintenant la définition C# de la classe `Make`, vous constaterez qu'elle a été mise à jour en conséquence :

```cs
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

namespace SimpleCSharpConsoleApp
{
	public class Make
	{
		public int Name
		{
			get => default;
			set { }
		}
	}
}
```

>[!note] Ne vous préoccupez pas des instructions `using` supplémentaires ni de la syntaxe des propriétés. Tout cela sera abordé dans les chapitres suivants.

Maintenant, réactivez le fichier du concepteur et faites glisser une nouvelle classe sur celui-ci. Nommez-la `SportsCar`. Cliquez sur l'icône *Héritage* dans la boîte à outils du concepteur de classes, puis sur l'icône `SportsCar`. Ensuite, cliquez sur l'icône de la classe `Car`. Si vous avez suivi correctement ces étapes, vous venez de créer la classe `SportsCar` *dérivée* de la classe `Car`.

![[Figure 2.13.png|Dérivé visuellement d'une classe existante]]

>[!info] Le concept d’héritage sera étudié en détail au [[Chapitre 6#Comprendre les mécanismes fondamentaux de l'héritage|Chapitre 6]].

Pour compléter cet exemple, mettez à jour la classe `SportsCar` générée avec une méthode publique nommée `GetPetName()`, écrite comme suit :

```cs
public class SportsCar : Car
{
	public string GetPetName()
	{
		PetName = "red";
		return PetName;
	}
}
```

Comme prévu, le concepteur affiche la méthode ajoutée à la classe `SportsCar`.

Ceci conclut votre première présentation de Visual Studio. Passons maintenant au dernier-né de la famille Visual Studio : Visual Studio Code.

# Construire des Applications .NET Core avec Visual Studio Code

VSCode est gratuit, open source et cross-platform. De plus, il supporte une myriade de language et accepte des *extensions* pour pouvoir personnaliser son expérience selon ses besions.

https://code.visualstudio.com/download

Après avoir installé VS Code, vous devrez ajouter l'extension C# disponible ici :

https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp

## Faire un tour de VS Code

Essayons Visual Studio Code en créant la même application console .NET 6 à partir de l'exemple Visual Studio.

### Création de solutions et de projets

Deux manières différentes (pour les versions récentes):

1. Quand l'extension C# est installée, 
    1. allez dans l'onglet à gauche `Explorer`
    2. Cliquer sur Créer un nouveau projet .NET
    Cela fournira un équivalent à Visual Studio dans la manière de procéder

2. Naviguer dans la fenêtre de l'explorateur jusqu'à l'emplacement souhaité
    1. [cmd + j] pour ouvrir le terminal (il se positionnera ou on à besoin)
    2. Entrez cette commande:
	
		```bash
		dotnet new sln -n SimpleCSharpConsoleApp -o .\VisualStudioCode
		```

Cela crée un nouveau fichier de solution nommé (*-n*) *SimpleCSharpConsoleApp* dans un sous-répertoire (du répertoire actuel) nommé *VisualStudioCode*. Lorsque vous utilisez Visual Studio Code avec une application à projet unique, il n'est pas nécessaire de créer un fichier de solution. ==Visual Studio est centré sur les solutions; Visual Studio Code est centré sur le code==. Nous avons créé un fichier de solution afin de reproduire le processus de l'exemple Visual Studio.

> [!warning] les examples fournis utilisent les séparateur de dossier Windows. Il faut utiliser les séparateurs selon le système d'exploitation utilisé.

Ensuite, créez une nouvelle application console C# 9/.NET 5 (`-f net6.0`) nommée (`-n`) *SimpleCSharpConsoleApp* dans un sous-répertoire (`-o`) du même nom (notez que cette commande doit être saisie sur une seule ligne) :

```bash
dotnet new console -lang c# -n SimpleCSharpConsoleApp -o .\VisualStudioCode\
SimpleCSharpConsoleApp -f net6.0
```

Enfin, ajoutez le projet nouvellement créé à la solution à l'aide de la commande suivante :

```bash
dotnet sln .\VisualStudioCode\SimpleCSharpConsoleApp.sln add .\VisualStudioCode\
SimpleCSharpConsoleApp
```

> [!note] Ceci n'est qu'un petit aperçu des capacités de l'interface de ligne de commande. La commande `dotnet -h` permet de voir toutes les commande disponibles de .NET CLI

### Exploration de l'espace de travail Visual Studio Code

![[Figure 2.14.png|L'espace de travail Visual Studio Code]]

Comme vous pouvez le voir, l'espace de travail Visual Studio Code est axé sur le code, mais offre également de nombreuses fonctionnalités supplémentaires pour vous aider à améliorer votre productivité. *L'explorateur (1)* est un explorateur de fichiers intégré et est sélectionné dans la figure. *Le contrôle de source (2) s'intègre à Git*. L'icône de *déboggage (3)* lance le déboggueur approprié (==une fois que l'extension correcte est installée==). La suivante est *le gestionnaire d'extensions(4 [Il y a une typo, c'est l'icône juste en dessous.])*. Le gestionnaire d'extensions est contextuel et fera des recommandations en fonction du type de code dans le répertoire ouvert et les sous-répertoires.

L'*éditeur de code (5)* est doté d'un codage couleur et **prend en charge IntelliSense**. La *carte du code (6)* affiche la carte de l'ensemble de votre fichier de code, et la *fenêtre Problèmes/Sortie/Console de déboggage/Terminal (7)* reçoit la sortie des sessions de déboggage et accepte les entrées de l'utilisateur.

### Restauration de packages, compilation et exécution de programmes

La CLI .NET 6 dispose de toutes les fonctionnalités nécessaires pour créer et compiler des solutions et des projets, ajouter et restaurer des packages *NuGet*, et exécuter des applications. Pour restaurer tous les packages NuGet requis pour votre solution et votre projet, entrez la commande suivante dans la fenêtre du terminal (ou dans une fenêtre de commande en dehors de VS Code), en veillant à exécuter la commande à partir du même répertoire que le fichier de solution :

```bash
dotnet restore
```

>[!note] Utiliser `dotnet build` restaure aussi les packages `NuGet`

Pour restaurer et compiler tous les projets de votre solution, exécutez la commande suivante dans le terminal/la fenêtre de commande (en vous assurant à nouveau que la commande est exécutée dans le même répertoire que le fichier de solution) :

```bash
dotnet build
```

>[!TIp]- les commandes `restore` et `build` liés avec des solutions :
>Lorsque `dotnet restore` et `dotnet build` sont exécutés dans un répertoire contenant un fichier de solution (*.sln ou .slnx*), tous les projets de la solution sont concernés. Les commandes peuvent également être exécutées sur un seul projet en exécutant la commande dans le répertoire du fichier de projet C# (*.csproj*).

Pour exécuter votre projet sans déboggage, exécutez la commande CLI .NET suivante dans le même répertoire que le fichier de projet (*SimpleCSharpConsoleApp.csproj*) :

```bash
dotnet run
```

#### La différence sémantique entre restaurer et compiler (Gemini)

 **Restaurer**
 
- *Objectif* : obtenir tous les packages externes, bibliothèques et références de projet nécessaires dont dépend votre code.
- *Fonction* : télécharge ces fichiers à partir d'un gestionnaire de packages (tel que *NuGet*) et les rend disponibles pour votre projet, en résolvant les dépendances et en s'assurant qu'ils sont correctement référencés.
- *Quand l'utiliser* : avant la compilation, en particulier lors de la configuration d'un projet sur une nouvelle machine, ou après l'ajout d'une nouvelle dépendance. De nombreuses commandes de compilation, telles que dotnet build, exécutent implicitement une restauration si nécessaire. 

**Compiler** 

- *Objectif* : compiler le code source du projet et produire une application ou une bibliothèque exécutable. 
- *Fonction* : prend le code, le combine avec les dépendances restaurées, puis le traduit dans un format lisible par machine (par exemple, un fichier exécutable). 
- *Quand l'utiliser* : pour créer le produit final à partir du code source une fois que toutes les dépendances nécessaires sont en place. 

**En résumé**

Considérez cela comme suit :
- La restauration : c'est comme rassembler tous les ingrédients d'une recette. 
- La compilation : c'est comme cuisiner ces ingrédients pour obtenir un plat fini. 
Vous ne pouvez pas faire un gâteau (compiler) si vous n'avez pas d'abord de la farine et du sucre (restaurer). 

### Débogguer son projet

Dans les nouvelles versions de VSCode (ou quand on utilise plusieurs debugger), il faut le choisir. Pour cela:

1. Allez dans l'onglet `Run and Debug`
2. Cliquer sur le bouton
3. Sélectionner l'option avec le nom du project dedans.

# Trouver la documentation .NET Core et C#

Les documentations C# et .NET Core sont toutes deux excellentes, très lisibles et regorgent d'informations utiles. Compte tenu du nombre considérable de types .NET prédéfinis (qui se comptent par milliers), vous devez être prêt à retrousser vos manches et à vous plonger dans la documentation fournie. Vous pouvez consulter l'intégralité de la documentation Microsoft ici :

https://docs.microsoft.com/en-us/dotnet/csharp/

Les sections que vous utiliserez le plus dans la première moitié de ce livre sont la documentation C# et la documentation .NET Core, disponibles aux emplacements suivants :

https://docs.microsoft.com/en-us/dotnet/csharp/
https://docs.microsoft.com/en-us/dotnet/core/

# Résumé du chapitre

L'objectif de ce chapitre était de vous fournir les informations nécessaires pour configurer votre environnement de développement avec le SDK et les runtimes .NET 6, ainsi que de vous présenter Visual Studio 2022 Community Edition et Visual Studio Code. Si vous souhaitez créer des applications .NET Core multiplateformes, vous disposez de trois choix . *Visual Studio* (Windows uniquement) et *Visual Studio Code* (multiplateforme) sont tous fournis par Microsoft, et *Rider* fournie par l'entreprise *JetBrains*. ==La création d'applications WPF ou WinForms nécessite toujours Visual Studio sur un ordinateur Windows==

>[!info] *MAUI* est une option qui permet de créer des applications à interface graphique multi-plateforme, bien que très peu utilisées.

---
title: "Chapitre 3: Principes Fondamentaux de la Programmatio - Partie 1"
publish: true
---

# <big><big><big><b><font color =green>Principes Fondamentaux de la Programmation C#, Partie 1</font></b></big></big></big>

Ce chapitre marque le début de votre étude formelle du langage de programmation C# en présentant un certain nombre de sujets concis et indépendants que vous devez maîtriser pour explorer le framework .NET Core. **La première chose à faire est de comprendre comment créer l'objet application de votre programme et d'examiner la composition du point d'entrée d'un programme exécutable : la méthode `Main()` ainsi qu'une nouvelle fonctionnalité de C# 9.0, les *instructions de niveau supérieur***. Ensuite, vous étudierez les ==types de données fondamentaux de C# (et leurs équivalents dans l'espace de noms `System`), notamment les classes `System.String` et `System.Text.StringBuilder`.==

Une fois que vous connaîtrez les détails des types de données fondamentaux de .NET Core, ==vous examinerez plusieurs techniques de conversion de types de données==, notamment les opérations de rétrécissement, les opérations d'élargissement (*upcast* et *downcast*) et l'**utilisation des mots-clés `checked` et `unchecked`**. 

Ce chapitre examinera également **le rôle du mot-clé `var` de C#**, qui vous permet de définir implicitement une variable locale. ==Comme vous le verrez plus loin dans cet ouvrage, le typage implicite est extrêmement utile, voire parfois obligatoire, lorsque vous travaillez avec l'ensemble de technologies `LINQ`==. Vous terminerez ce chapitre en examinant rapidement les *mots-clés et opérateurs C#* qui vous permettent de contrôler le flux d'une application à l'aide de diverses constructions de **boucles** et de **décisions**.

# Analyse d'un programme C# simple (MaJ C# 10)

C# exige que toute la logique du programme soit contenue dans une définition de type (rappel du [[Chapitre 1#Comprendre le Common Type System (CTS)|Chapitre 1]] que le type est un terme général désignant un membre de l'ensemble {*classe, interface, structure, énumération, délégué*}). **Contrairement à de nombreux autres langages, il n'est pas possible en C# de créer des fonctions globales ou des points de données globaux**. Au contraire, ==tous les membres de données et toutes les méthodes doivent être contenus dans une définition de type==. Pour commencer, créez une nouvelle solution vide nommée *Chapter3_AllProject.sln* qui contient une application console C# nommée *SimpleCSharpApp*.

<small><small>Dans Visual Studio, sélectionnez le modèle Blank Solution (Solution vide) dans l'écran « Create a new project » (Créer un nouveau projet). Lorsque la solution s'ouvre, cliquez avec le bouton droit sur la solution dans Solution Explorer et sélectionnez Add ➤ New Project (Ajouter ➤ Nouveau projet). Sélectionnez « Application console C# » parmi les modèles, nommez-la SimpleCSharpApp, puis cliquez sur Suivant. Sélectionnez .NET 6.0 pour le framework, puis cliquez sur Créer.</small></small>

Pour créer une solution et une application console, puis ajouter cette application console à la solution, à partir de la ligne de commande (ou de la fenêtre du terminal Visual Studio Code), exécutez la commande suivante :

```bash
dotnet new sln -n Chapter3_AllProjects
dotnet new console -lang c# -n SimpleCSharpApp -o .\SimpleCSharpApp -f net6.0
dotnet sln .\Chapter3_AllProjects.sln add .\SimpleCSharpApp
```

>[!Info] Depuis .NET 10:
>La création de solution utilise l'extension de fichier *.slnx* au lieu de *.sln*. C'est un changement important car maintenant les solution utilise le *XML*, comme le fichier project (*.csproj*).

>[!warning] Dans VSCode:
>Si plusieurs solutions sont présentes, il faut faire attention à bien ouvrir la solution ou le projet est référencé, sinon l'IDE ne pourra pas build et débugger les bons projets.

Dans le projet créé, vous verrez un fichier (nommé *Program.cs*) contenant une ligne de code.

```cs
Console.WriteLine("Hello, World!");
```

Si vous débutez avec C#, cette ligne semble assez simple. Elle affiche le message « Hello, World ! » dans la fenêtre de sortie standard de la console. *Avant C# 10, il fallait beaucoup plus de code pour obtenir le même effet. Pour créer le même programme dans les versions de C# antérieures à C# 10, vous deviez écrire ce qui suit* :

```cs
using System;
namespace SimpleCSharpApp
{
    class Program
    {
        static void Main(string[] args)
        {
	        Console.WriteLine("Hello World!");
        }
    }
}
```

La classe `Console` est contenue dans l'espace de noms `System`, et avec les espaces de noms globaux implicites fournis par .NET 6/C# 10, **l'instruction `using System;` n'est plus nécessaire**. La ligne suivante crée un espace de noms personnalisé (abordé au [[Chapitre 16]]) pour encapsuler la classe Program. **L'espace de noms et la classe `Program` peuvent tous deux être supprimés grâce à la fonctionnalité d'instruction de niveau supérieur introduite dans C# 9** (abordée brièvement). Cela nous ramène à la ligne de code unique permettant d'écrire le message dans la console.

==Pour l'instant, afin de couvrir certaines variations importantes du point d'entrée dans les applications C#, nous utiliserons l'ancien style de code (plus verbeux) au lieu de la version simplifiée de C# 10==. Dans ce contexte, mettez à jour la méthode `Main()` de votre classe `Program` avec les instructions de code suivantes :

```cs
class Program
{
    static void Main(string[] args)
    {
        // Display a simple message to the user.
        Console.WriteLine("***** My First C# App *****");
        Console.WriteLine("Hello World!");
        Console.WriteLine();
        
        // Wait for Enter key to be pressed before shutting down.
        Console.ReadLine();
    }
}
```

> [!warning] Important:
> **C# est un language sensible à la casse**, `Main` et `main` ne sont pas les mêmes choses, et `ReadLine` n'est pas la même chose que `Readline`.
> 
> Sachez que tous les mots-clés C# sont en minuscules (par exemple, `public`, `lock`, `class`, `dynamic`), tandis que les espaces de noms, les types et les noms de membres commencent (*par convention*) par une majuscule initiale et que la première lettre de tout mot intégré est en majuscule (*CamelCase*) ,par exemple, `Console.WriteLine`,  `System.Windows.MessageBox`, `System.Data.SqlClient`). **En règle générale, lorsque vous recevez une erreur de compilation concernant des « symboles non définis », vérifiez d'abord l'orthographe et la casse !**

Le code précédent contient une définition pour un type de classe qui prend en charge une seule méthode nommée `Main()`. **Par défaut, les modèles de projet C# qui n'utilisent pas d'instructions de niveau supérieur nomment la classe contenant la méthode `Main()` `Program`** ; cependant, vous êtes libre de modifier cela si vous le souhaitez. ==**Avant C# 9.0, chaque application C# exécutable** (programme console, programme Windows Desktop ou service Windows) **devait contenir une classe définissant une méthode `Main()`, utilisée pour indiquer le point d'entrée de l'application**==.
 
Formellement parlant, ==la classe qui définit la méthode `Main()` est appelée *objet d'application*==. ==Il est possible qu'une seule application exécutable ait *plusieurs objets d'application*== (ce qui peut être utile lors de la réalisation de tests unitaires), ==mais le compilateur doit alors savoir quelle méthode `Main()` doit être utilisée comme point d'entrée==. ==Cela peut être fait via l'élément `<StartupObject>` dans le fichier de projet (*.csproj*)==.

Notez que la signature de `Main()` est agrémentée du mot-clé `static`, qui sera examiné en détail au [[Chapitre 5#Comprendre le mot-clé `static`|Chapitre 5]]. Pour l'instant, il suffit de comprendre que ==**les membres statiques ont une portée au niveau de la classe** (plutôt qu'au niveau de l'objet) **et peuvent donc être invoqués sans qu'il soit nécessaire de créer au préalable une nouvelle instance de classe**==. 

En plus du mot-clé `static`, cette méthode `Main()` possède ==un seul paramètre==, qui se trouve être un ==tableau de chaînes (`string[] args`)==. Bien que vous ne vous souciez pas actuellement de traiter ce tableau, **ce paramètre peut contenir un nombre quelconque d'arguments de ligne de commande entrants** (***vous verrez comment y accéder dans un instant***). Enfin, cette méthode `Main()` a été configurée avec une ==valeur de retour `void`, ce qui signifie que vous ne définissez pas explicitement de valeur de retour à l'aide du mot-clé `return` avant de quitter la portée de la méthode==.

La logique de la classe `Program` se trouve dans `Main()`. Ici, vous utilisez la classe `Console`, qui est définie dans l'espace de noms `System`. ==Parmi ses membres figure la méthode statique `WriteLine()`, qui, comme vous pouvez le supposer, envoie une chaîne de texte et un retour chariot vers la sortie standard (*stdout*)==. Vous appelez également `Console.ReadLine()` pour vous assurer que le programme reste visible. 

<small>Lorsque vous exécutez des applications .NET Core Console avec Visual Studio (en mode Debug ou Release), la fenêtre de console reste visible par défaut. Ce comportement peut être modifié en activant le paramètre « Fermer automatiquement la console lorsque le débogage s'arrête » qui se trouve sous Outils ➤ Options ➤ Débogage. La méthode Console.ReadLine() permet de garder la fenêtre ouverte si le programme est exécuté à partir de l'Explorateur Windows en double-cliquant sur le fichier *.exe du produit. Vous en apprendrez davantage sur la classe System.Console dans quelques instants.</small>

### Comment passer d'une déclaration de haut-niveau à bas niveau

deux Manières différentes sur VSCode:

1. Lors de la création d'un projet dans VSCode (pas le CLI), on peut, à la fin des menu, lors de la finalisation, on peut choisir `Do not use top-level statement`. Il faut mettre sur `true`
	![[Figure 3.1.png|Basculer entre déclaration de haut/bas niveau]]

2. Quand le fichier utilise une déclaration de haut niveau, cliquer sur l'icône lampe :![[lampe IntelliSense.png|]] et sélectionner `Convert to 'Program.main' style program` ou `Convert to top-level statement`  selon la déclaration présente (le raccourcis *cmd + .* est disponible)

## Utilisation des variantes de la méthode `Main()` (MaJ C# 7.1)

Par défaut, le modèle de projet console .NET ==génère une méthode `Main()` qui a une valeur de retour `void` et un *tableau* de type `string` comme seul paramètre d'entrée(`string[]`)==. Cependant, **ce n'est pas la seule forme possible de `Main()`**. Il est possible de construire le point d'entrée de votre application en utilisant l'une des signatures suivantes (==en supposant qu'elle soit contenue dans une définition de classe ou de structure C#==) :

```cs
// int return type, array of strings as the parameter.
static int Main(string[] args)
{
    // Must return a value before exiting!
    return 0;
}

// No return type, no parameters.
static void Main()
{
}

// int return type, no parameters.
static int Main()
{
    // Must return a value before exiting!
    return 0;
}
```

**Avec la sortie de C# 7.1, la méthode `Main()` peut être asynchrone**. La programmation asynchrone est abordée au [[Chapitre 15]], mais pour l'instant, sachez qu'il existe quatre signatures supplémentaires.

```cs
static Task Main()
static Task Main(string[])
static Task<int> Main()
static Task<int> Main(string[])
```

>[!note]- Notes sur la méthode `Main`:
>La méthode `Main()` peut également être définie comme publique plutôt que privée. Notez que le modificateur d'accès est présumé privé si vous ne fournissez pas de modificateur d'accès spécifique. Les modificateurs d'accès sont décrits en détail au [[Chapitre 5#Comprendre les modificateurs d'accès (MaJ C 7.2)|Chapitre 5]].

Évidemment, votre choix quant à la manière de construire `Main()` dépendra de trois questions: 
- Premièrement, souhaitez-vous renvoyer une valeur au système lorsque `Main()` est terminé et que votre programme se termine ? 
    - Si oui, vous devez renvoyer un type de données `int` (`byte`) plutôt que `void`. 
- Deuxièmement, avez-vous besoin de traiter des paramètres de ligne de commande fournis par l'utilisateur? 
    - Si oui, ils seront stockés dans le tableau de chaînes (`string[]`). 
- Enfin, avez-vous besoin d'appeler du code *asynchrone* à partir de la méthode `Main()` ? 

Nous examinerons les deux premières options plus en détail après avoir présenté les instructions de niveau supérieur. 

>Les options asynchrones seront abordées au [[Chapitre 15]].

## Utiliser les déclaration de haut niveaux (Nouveauté C# 9.0)

S'il est vrai qu'avant C# 9.0, toutes les applications C# .NET Core devaient disposer d'une méthode `Main()`, C# 9.0 a introduit des instructions de niveau supérieur qui ==éliminent la plupart des formalités liées au point d'entrée de l'application C#==. La classe (`Program`) et les méthodes `Main()` peuvent toutes deux être supprimées. ==Pour voir cela en action, mettez à jour la classe *Program.cs* comme suit== :

```cs
// Display a simple message to the user.
Console.WriteLine("***** My First C# App *****");
Console.WriteLine("Hello World!");
Console.WriteLine();

// Wait for Enter key to be pressed before shutting down.
Console.ReadLine();
```

Vous verrez que lorsque vous exécuterez le programme, vous obtiendrez le même résultat ! **Il existe certaines règles concernant l'utilisation des instructions de niveau supérieur** :

- **Un seul fichier de l'application peut utiliser des instructions de niveau supérieur.**
- ==Lorsque vous utilisez des instructions de niveau supérieur, le programme ne peut pas avoir de point d'entrée déclaré==.
- Les instructions de niveau supérieur ne peuvent pas être incluses dans un espace de noms.
- ==Les instructions de niveau supérieur accèdent toujours à un tableau de chaînes de caractères d'arguments==.
- Les instructions de niveau supérieur renvoient un code d'application (voir la section suivante) à l'aide d'un retour.
- **Les fonctions qui auraient été déclarées dans la classe `Program` deviennent des fonctions locales pour les instructions de niveau supérieur**. (Les fonctions locales sont abordées au [[Chapitre 4#Comprendre les fonctions locales (Nouveauté C 7.0, MaJ C 9.0)|Chapitre 4]]).
- **Les instructions de niveau supérieur sont compilées dans une classe nommée `Program`, ce qui permet d'ajouter une classe `Program` *partielle* pour contenir des méthodes régulières**. Les classes partielles sont abordées au [[Chapitre 5#Comprendre les classes partielles|Chapitre 5]]. 
- Des types supplémentaires peuvent être déclarés après toutes les instructions de niveau supérieur. ==Tout type déclaré avant la fin des instructions de niveau supérieur entraînera une erreur de compilation==.

**En arrière-plan, le compilateur comble les lacunes. En examinant le code IL généré pour le code mis à jour, vous verrez le `TypeDef` suivant pour le point d'entrée dans l'application**:

```CIL
// TypeDef #1 (02000002)
// -------------------------------------------------------
//      TypDefName: <Program>$ (02000002)
//      Flags : [NotPublic] [AutoLayout] [Class] [Abstract] [Sealed] [AnsiClass] [BeforeFieldInit] (00100180)
//      Extends : 0100000D [TypeRef] System.Object
//      Method #1 (06000001) [ENTRYPOINT]
// -------------------------------------------------------
//          MethodName: <Main>$ (06000001)
```

Comparons cela avec le `TypeDef` du point d'entrée vu dans le [[Chapitre 1#Le rôle des métadonnée des types de donnée .NET|Chapitre 1]]:

```CIL
// TypeDef #1 (02000002)
// -------------------------------------------------------
//      TypDefName: CalculatorExamples.Program (02000002)
//      Flags : [NotPublic] [AutoLayout] [Class] [AnsiClass] [BeforeFieldInit] (00100000)
//      Extends : 0100000C [TypeRef] System.Object
//      Method #1 (06000001) [ENTRYPOINT]
// -------------------------------------------------------
//          MethodName: Main (06000001)
```

Remarquez que dans l'exemple du [[Chapitre 1#Le rôle des métadonnée des types de donnée .NET|chapitre 1]], la valeur `TypDefName` est indiquée comme l'espace de noms `CalculatorExamples` plus le nom de la classe `Program`, et la valeur `MethodName` est `Main`. Dans l'exemple mis à jour utilisant des instructions de niveau supérieur, le compilateur a renseigné les valeurs `<Program>$` pour `TypDefName` et `<Main>$` pour le nom de la méthode.

## Spécification d'un code d'erreur d'application (MaJ C# 9.0)

Bien que la grande majorité de vos méthodes `Main()` (ou instructions de niveau supérieur) renvoient la valeur void comme valeur de retour, ==la possibilité de renvoyer un `int` (ou `Task<int>`) permet à C# de rester cohérent avec les autres langages basés sur C==. Par convention, le renvoi de la valeur *0* indique que le programme s'est *terminé avec succès*, tandis qu'une autre valeur (telle que *-1*) représente une *condition d'erreur* (==notez que la valeur 0 est automatiquement renvoyée, même si vous construisez une méthode Main() prototypée pour renvoyer void==).

Lorsque vous utilisez des instructions de niveau supérieur, si le code en cours d'exécution renvoie un entier (`int`), celui-ci constitue le code de retour. Si rien n'est explicitement renvoyé, la valeur 0 est tout de même renvoyée, comme lors de l'utilisation explicite d'une méthode Main().

**Sous le système d'exploitation *Windows*, la valeur de retour d'une application est stockée dans une variable d'environnement système nommée `%ERRORLEVEL%`**

**Sous les systèmes d'exploitation basés sur *UNIX*, la variable d'environnement qui stocke la valeur de retour est `$?`.**

>[!info]- Thèmes de terminal 
>Si un thème de terminal est installé, il est possible d'afficher directement ces variables dans le prompt, comme l'image suivante (*Powerlevel10K*):
>
>![[Code de sortie.png|Thème de terminal (ici P10K)]]

 Si vous deviez créer une application qui lance par programmation un autre exécutable (un sujet examiné au [[Chapitre 18|chapitre 18]]), vous pouvez obtenir la valeur de `%ERRORLEVEL%` ou `$?` à l'aide de la propriété `ExitCode` du processus lancé.

Étant donné que la valeur de retour d'une application est transmise au système au moment où l'application se termine, **il n'est évidemment pas possible pour une application d'obtenir et d'afficher son code d'erreur final pendant son exécution**. Cependant, pour illustrer comment afficher ce niveau d'erreur à la fin du programme, commencez par mettre à jour les instructions de niveau supérieur comme suit :

```cs
// Note we are explicitly returning an int, rather than void.
// Display a message and wait for Enter key to be pressed.
Console.WriteLine("***** My First C# App *****");
Console.WriteLine();
Console.WriteLine("Hello World!");
Console.WriteLine();
Console.ReadLine();

// Return an arbitrary error code.
return -1;
```

Si le programme utilise toujours une méthode `Main()` comme point d'entrée, modifiez la signature de la méthode pour qu'elle renvoie `int` au lieu de `void`, comme suit :

```cs
static int Main()
{
    ...
}
```

**Maintenant, capturons la valeur de retour du programme à l'aide d'un fichier batch**. 

### Pour windows -> cmd

À l'aide de l'Explorateur Windows, accédez au dossier contenant votre fichier de projet (par exemple, *C:\SimpleCSharpApp*) et ajoutez un nouveau fichier texte (nommé *SimpleCSharpApp.cmd*) à ce dossier. Mettez à jour le contenu du dossier comme suit (si vous n'avez jamais créé de fichiers *.cmd* auparavant, ne vous préoccupez pas des détails) :

```cmd
@echo off
rem A batch file for SimpleCSharpApp.exe
rem which captures the app's return value.

dotnet run
@if "%ERRORLEVEL%" == "0" goto success
:fail
    echo This application has failed!
    echo return value = %ERRORLEVEL%
    goto end
:success
    echo This application has succeeded!
    echo return value = %ERRORLEVEL%
    goto end
:end
echo All Done.
```

### Pour MacOS -> bash ou zsh

Avec Finder ou le terminal, accéder au dossier contenant le fichier *.csproj* et ajouter un nouveau fichier *SimpleCSharpApp* (avec l'extension de fichier *.sh* ou *zsh*). Dans le fichier, entrez:

```bash
#!/usr/bin/env zsh

# Un script zsh pour SimpleCSharpApp.exe
# qui capture la valeur de retour de l'app.

dotnet run

return=$?   # capture l'exitcode
# les opérations suivantes (if, []) retournent des codes de sorties aussi, il faut donc la captuer avant de la comparer

if [ $return -eq 0 ]; then
    echo "This application has succeeded!"
    echo "return value = $return"
else
    echo "This application has failed!"
    echo "return value = $return"
fi

echo "All Done."
```

>Ici, on stock `$?` (qui est la variable du dernier code de retours) sinon à chaque commande cette valeur change, tandis que l'on veut seulement la valeur de retour de la commande `dotnet run`

À ce stade, ouvrez une invite de commande (ou utilisez le terminal Visual Studio Code) et accédez au dossier contenant votre nouveau fichier *.cmd* / *.zsh*. Exécutez le fichier en tapant son nom et en appuyant sur la touche Entrée. ==Vous devriez obtenir le résultat suivant, à condition que vos instructions de niveau supérieur (ou méthode Main()) renvoient -1==. Si les instructions de niveau supérieur (ou la méthode `Main()`) renvoyaient 0, le message « Cette application a réussi ! » s'afficherait dans la console.

```
***** My First C# App *****

Hello World!

This application has failed!
Return value = -1
All Done.
```

L'équivalent *PowerShell (.ps1)* est le suivant:

```PowerShell
dotnet run

if ($LastExitCode -eq 0) {
    Write-Host "This application has succeeded!"
} else{
    Write-Host "This application has failed!"
}
Write-Host "All Done."
```

>[!Note]- Erreur lors de l'exécution du script *.ps1* 
>Si vous recevez une erreur de stratégie de sécurité lors de l'exécution du script PowerShell, vous pouvez définir la stratégie pour autoriser les scripts locaux non signés en exécutant la commande suivante dans PowerShell :
>```
set-executionpolicy -executionpolicy remotesigned -scope currentuser
>```

>[!warning] En utilisant ce type de script, la variable retourné aura pour valeur `0` dans le terminal car le dernier processus s'est exécuté sans erreur (le script *.cmd* /*.sh* / *.ps1* / *zsh*).

*La grande majorité (sinon la totalité) de vos applications C# utiliseront `void` comme valeur de retour de `Main()`*, qui, comme vous vous en souvenez, renvoie implicitement le code d'erreur `0`. À cette fin, les méthodes `Main()` utilisées dans ce texte (au-delà de l'exemple actuel) renverront `void`.

## Traitement des arguments de ligne de commande (MaJ C# 9.0)

 Maintenant que vous comprenez mieux la valeur de retour de la méthode Main() ou des instructions de niveau supérieur, ==examinons le tableau de données de type chaîne `string`. Supposons qu'on traite tous les paramètres de ligne de commande possibles==. **Pour ce faire, vous pouvez utiliser une boucle `for` en C#. (Notez que les constructions d'itération en C# seront examinées plus en détail vers la fin de ce chapitre.)**

```cs
// Display a message and wait for Enter key to be pressed.
Console.WriteLine("***** My First C# App *****");
Console.WriteLine("Hello World!");
Console.WriteLine();
// Process any incoming args.
for (int i = 0; i < args.Length; i++)
{
    Console.WriteLine("Arg: {0}", args[i]);
}
Console.ReadLine();
// Return an arbitrary error code.
return 0;
```

>[!note] Notez que cet exemple utilise des instructions globales, et non une méthode `Main()`. La mise à jour de la méthode `Main()` pour accepter le paramètre `args` sera abordée prochainement.

Une fois encore, en examinant le CIL généré pour le programme à l'aide d'instructions de niveau supérieur, notez que la méthode `<Main>$` accepte un tableau de chaines (`string`) nommé `args`, comme indiqué ici (abrégé pour gagner de la place) :

```CIL
// =============== CLASS MEMBERS DECLARATION ===================

.class private auto ansi beforefieldinit Program
       extends [System.Runtime]System.Object
{
  .custom instance void [System.Runtime]System.Runtime.CompilerServices.CompilerGeneratedAttribute::.ctor() = ( 01 00 00 00 ) 
  .method private hidebysig static int32 '<Main>$'(string[] args) cil managed
  {
    .entrypoint
    // Code size       70 (0x46)
    ...
  
  } // end of method Program::.ctor

} // end of class Program
```

Si le programme utilise toujours une méthode `Main()` comme point d'entrée, assurez-vous que la signature de la méthode accepte un paramètre de type `string[]` nommé `args`, comme suit :

```cs
static int Main(string[] args)
{
    ...
}
```

Maintenant, **vous vérifiez si le tableau de chaînes contient un certain nombre d'éléments à l'aide de la propriété `Length` de `System.Array`**. Comme vous le verrez au [[Chapitre 4#Comprendre les Tableaux C|Chapitre 4]], **tous les tableaux C# sont en fait des alias de la classe `System. Array` et partagent donc un ensemble commun de membres**. Lorsque vous parcourez chaque élément du tableau, sa valeur est affichée dans la fenêtre de la console. ***La fourniture des arguments dans la ligne de commande est tout aussi simple, comme illustré ici*** :

```bash
dotnet run /arg1 -arg2 
```

```
***** My First C# App *****
Hello World!

Arg: /arg1
Arg: -arg2
```

> [!note]  Pour que ce qui suit `run` soit considéré comme **un seul `string`, il faut utilisé des `"` ou des `'`**. 

Comme alternative à la boucle `for` standard, vous pouvez itérer sur un tableau de `string` entrant à l'aide du mot-clé C# `foreach`. Voici quelques exemples d'utilisation (mais, encore une fois, vous verrez plus loin dans ce chapitre les spécificités des constructions de boucles):

```cs
// Notez qu'il n'est pas nécessaire de vérifier la taille du tableau lorsque vous utilisez « foreach ».
// Traitez les arguments entrants à l'aide de foreach.
foreach (string arg in args)
{
    Console.WriteLine("Arg: {0}", arg);
}
Console.ReadLine();
return 0;
```

==Enfin, vous pouvez également accéder aux arguments de ligne de commande à l'aide de la *méthode statique* `GetCommandLineArgs()` du type `System.Environment`==. La valeur renvoyée par cette méthode est un tableau de `string`. ==La première entrée contient le nom de l'application elle-même (le chemin d'accès)==, tandis que les autres éléments du tableau contiennent les arguments de ligne de commande individuels.

>[!tip]- La différence entre `GetCommandLineArgs()` et `args`
> La méthode `GetCommandLineArgs` ne reçoit pas les arguments de l'application via la méthode `Main()` et ne dépend pas du paramètre `string[] args`.
>>[!note] Le code suivant n'utilise pas la déclaration de haut niveau pour prouver ce point.

```cs
class Program
{
    static int Main()
    {
        // Récupérer les arguments en utilisant System.Environment.
        string[] theArgs = Environment.GetCommandLineArgs();
        foreach (string arg in theArgs)
        {
            Console.WriteLine("Arg: {0}", arg);
        }
        Console.ReadLine();
        return 0;
    }
}
```

```
Arg: /Volumes/GM Disk 1/Code/Livre C#/SimpleCSharpApp/bin/Debug/net10.0/SimpleCSharpApp.dll
Arg: /arg1
Arg: -arg2
```

Bien sûr, c'est à vous de déterminer à quels arguments de ligne de commande votre programme répondra (le cas échéant) et comment ils doivent être formatés (par exemple avec un préfixe `-` ou `/`). **Ici, on a simplement passé une série d'options qui ont été affichées directement à l'invite de commande**. ==Supposons toutefois que vous créiez un nouveau jeu vidéo et que vous ayez programmé votre application pour traiter une option nommée `-godmode`==. Si l'utilisateur démarre votre application avec ce drapeau, vous savez qu'il s'agit en fait d'un tricheur et vous pouvez prendre les mesures appropriées.

# Membres supplémentaires de la classe `System.Environment` (MaJ C# 10.0)

**La classe `Environment` expose un certain nombre de méthodes extrêmement utiles au-delà de `GetCommandLineArgs()`**. Plus précisément, ==cette classe vous permet d'obtenir un certain nombre de détails concernant le système d'exploitation qui héberge actuellement votre application .NET à l'aide de divers membres statiques==. Pour illustrer l'utilité de `System.Environment`, mettez à jour votre code afin d'appeler une méthode locale nommée `ShowEnvironmentDetails()`.

```cs
// Méthode locale dasn la déclaration de haut niveau.
ShowEnvironmentDetails();

Console.ReadLine();
return -1;
}
```

Implémentez cette méthode après vos instructions de niveau supérieur pour appeler divers membres du type `Environment` :

```cs
static void ShowEnvironmentDetails()
{
    // Affiche les disques de cette machine
    // et d'autres détails intéressants.
    foreach (string drive in Environment.GetLogicalDrives())
    {
        // Dont show the MacOS backups.
        if (drive.Contains("timemachine"))
            continue;

        Console.WriteLine("Drive: {0}", drive);
    }
    Console.WriteLine();
    Console.WriteLine("OS: {0}", Environment.OSVersion);
    Console.WriteLine("Number of processors: {0}",
    Environment.ProcessorCount);
    Console.WriteLine(".NET Core Version: {0}",
    Environment.Version);
}
```

La sortie suivante montre un exemple d'exécution de test de cette méthode :

```
***** My First C# App *****
hello World!
Drive: ...
Drive: ...
Drive: ...
...

OS: Unix 26.3.1
Number of processors: 10
.NET Core Version: 10.0.5

```

Le type `Environment` définit des membres autres que ceux indiqués dans l'exemple précédent. Le [[#Tableau 3-1 Sélection de Propriétés de `System.Environment`|Tableau 3-1]] présente certaines propriétés supplémentaires intéressantes; toutefois, veillez à consulter la [documentation](https://learn.microsoft.com/en-us/dotnet/api/system.environment?view=net-10.0) en ligne pour obtenir des informations complètes.

##### Tableau 3-1: Sélection de Propriétés de `System.Environment`

| Propriété                | Description                                                                                                                                                 |
| ------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ExitCode`               | Obtient ou définit le code de sortie de l'application.                                                                                                      |
| `Is64BitOperatingSystem` | Renvoie une valeur booléenne indiquant si la machine hôte exécute un système d'exploitation 64 bits.                                                        |
| `MachineName`            | Obtient le nom de la machine actuelle                                                                                                                       |
| `NewLine`                | Récupère le symbole de nouvelle ligne pour l'environnement actuel (/n par exemple).                                                                         |
| `ProcessId` (C# 10)      | Obtient l'identifiant unique du processus actuel.                                                                                                           |
| `ProcessPath` (C# 10)    | Renvoie le chemin d'accès de l'exécutable qui a lancé le processus en cours d'exécution ;<br>renvoie `null` lorsque le chemin d'accès n'est pas disponible. |
| `SystemDirectory`        | Renvoie le chemin d'accès complet vers le répertoire système.                                                                                               |
| `UserName`               | Renvoie le nom de l'utilisateur qui a lancé cette application.                                                                                              |
| `Version`                | Renvoie un objet Version qui représente la version de la plateforme .NET Core.                                                                              |

# Utilisation de la classe `System.Console`

Presque toutes les applications d'exemple créées au cours des premiers chapitres de ce livre utilisent abondamment la classe `System.Console`. S'il est vrai qu'une interface utilisateur console (*CUI*) peut ne pas être aussi attrayante qu'une interface utilisateur graphique (*GUI*) ou une application web, le fait de limiter les premiers exemples aux programmes console vous permettra de rester concentré sur la syntaxe de C# et les aspects fondamentaux de la plateforme .NET 6, plutôt que de vous occuper des complexités liées à la création d'interfaces graphiques ou de sites web.

>L'accès à la classe `Console` est désormais implicitement fourni par les instructions `global using` fournies par .NET 6, ce qui rend inutile l'ajout de l'instruction `using System`; qui était requise dans les versions précédentes de .NET

Comme son nom l'indique, la classe `Console` encapsule les manipulations des flux d'entrée, de sortie et d'erreur pour les applications basées sur la console (*stdin*, *stdout* et *stderr* sur les systèmes basés sur *Unix*). Le [[#Tableau 3-2 Sélection de membres de la classe `Console`|tableau 3-2]] répertorie certains membres intéressants (mais certainement pas tous). Comme vous pouvez le constater, la classe `Console` fournit certains membres qui peuvent enrichir une application en ligne de commande simple, tels que la possibilité de modifier les couleurs d'arrière-plan et de premier plan et d'émettre des bips sonores (à différentes fréquences ![^1]).

##### Tableau 3-2: Sélection de membres de la classe `Console`

| Membre                                                           | Description                                                                                                                                                                         |
| ---------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Beep()`                                                         | Cette méthode force la console à émettre une tonalité  (d'une certaine fréquence et durée pour windows)                                                                             |
| `BackgroundColor`<br>`ForegroundColor`                           | Ces propriétés définissent les couleurs d'arrière-plan/de premier plan pour la sortie actuelle.Ils peuvent être assignés à n'importe quel membre de l'*énumération* `ConsoleColor`. |
| `BufferHeight`<br>`BufferWidgh`                                  | Ces propriétés contrôlent la hauteur et la largeur de la zone de tampon de la console.                                                                                              |
| `Title`                                                          | Cette propriété récupère ou assigne le titre de la console acutelle.                                                                                                                |
| `WindowHeight` <br>`WindowWidth` <br>`WindowTop`<br>`WindowLeft` | Ces propriétés contrôlent les dimensions de la console par rapport au<br>tampon établi.                                                                                             |
| `Clear()`                                                        | Cette méthode efface la mémoire tampon établie et la zone d'affichage de la console.                                                                                                |

## Exécution d'entrées et sorties (E/S) de base avec la classe Console

En plus des membres du [[#Tableau 3-2 Sélection de membres de la classe `Console`|tableau 3-2]], le type `Console` définit un ensemble de méthodes permettant de capturer les entrées et les sorties, qui sont toutes statiques et sont donc appelées en préfixant le nom de la classe (Console) au nom de la méthode. **Comme vous l'avez vu, `WriteLine()` envoie une chaîne de texte (y compris un retour chariot) vers le flux de sortie (*stdin*)**. ==La méthode `Write()` transfère le texte vers le flux de sortie sans retour chariot==. **La méthode `ReadLine()` vous permet de recevoir des informations du flux d'entrée jusqu'à ce que la touche Entrée soit enfoncée**, ==tandis que `Read()` est utilisé pour capturer un seul caractère du flux d'entrée==.

Pour illustrer une E/S simple à l'aide de la classe `Console`, créez un nouveau projet d'application console nommé *BasicConsoleIO* et ajoutez-le à votre solution à l'aide des commandes CLI suivantes 

```
dotnet new console -lang c# -n BasicConsoleIO -o .\BasicConsoleIO
dotnet sln .\Chapter3_AllProjects.sln add .\BasicConsoleIO
```

Remplacez le code Program.cs par ce qui suit :

```cs
Console.WriteLine("***** Basic Console I/O *****");
GetUserData();
Console.ReadLine();

static void GetUserData()
{
}
```

>[!tip]- Les extraits de codes.
>Visual Studio et Visual Studio Code prennent tous deux en charge un certain nombre d'«extraits de code» (*snippets*) qui insèrent du code une fois activés. ==L'extrait de code `cw` est très utile dans les premiers chapitres de ce texte, car il se développe automatiquement en `Console.WriteLine()` !== Pour le tester par vous-même, tapez *cw* quelque part dans votre code et appuyez sur la touche Tab.
>>[!example] Dans Visual Studio Code, vous appuyez une fois sur la touche Tab ; dans Visual Studio, vous devez appuyer deux fois sur la touche Tab.


Implémentez cette méthode après les instructions de niveau supérieur avec une logique qui invite l'utilisateur à fournir certaines informations et renvoie chaque élément vers le flux de sortie standard (*stdout*). ==Par exemple, vous pouvez demander à l'utilisateur son nom et son âge== (qui seront traités comme des valeurs textuelles pour plus de simplicité, plutôt que comme les valeurs numériques attendues), ==comme sui==t :

```cs
static void GetUserData()
{
    // Récupérer le nom et l'age.
    Console.Write("Please enter your name: ");
    string userName = Console.ReadLine();
    Console.Write("Please enter your age: ");
    string userAge = Console.ReadLine();

    // Change la couleur du texte, pour le fun.
    ConsoleColor prevColor = Console.ForegroundColor;
    Console.ForegroundColor = ConsoleColor.Yellow;

    // Affiche dans la console.
    Console.WriteLine("Hello {0}! You are {1} years old", userName, userAge);

    // Restaure la couleur du texte précédentes.
    Console.ForegroundColor = prevColor;
}

```

Sans surprise, lorsque vous exécutez cette application, les données d'entrée sont affichées dans la console (en utilisant une couleur personnalisée pour démarrer !).

## Formatage de la sortie console

Au cours de ces premiers chapitres, vous avez peut-être remarqué de nombreuses occurrences de jetons tels que `{0}` et `{1}` intégrés dans divers littéraux de chaîne. La plateforme .NET 6 prend en charge un style de **formatage de chaîne** ==légèrement similaire à l'instruction `printf()` du langage C==. En termes simples, lorsque vous définissez une chaîne littérale contenant des segments de données dont la valeur n'est connue qu'au moment de l'exécution, vous pouvez spécifier un espace réservé dans la chaîne littérale à l'aide de cette syntaxe avec des accolades. **Au moment de l'exécution, les valeurs transmises à `Console.WriteLine()` sont substituées à chaque espace réservé**.

Le premier paramètre de `WriteLine()` représente une chaîne littérale (`string`) contenant des espaces réservés facultatifs désignés par `{0}`, `{1}`, `{2}`, etc. Notez que le premier numéro ordinal d'un espace réservé entre accolades ==commence toujours par 0==. Les autres paramètres de `WriteLine()` sont simplement les valeurs à insérer dans les espaces réservés respectifs.

>[!Attention]
>Si vous avez plus d'espaces réservés entre accolades que d'arguments de remplissage, vous recevrez une exception de format lors de l'exécution. Cependant, si vous avez plus d'arguments de remplissage que d'espaces réservés, les arguments de remplissage inutilisés sont ignorés.

Il est permis qu'un espace réservé donné se répète dans une chaîne donnée. Par exemple, si vous êtes fan des Beatles et que vous souhaitez créer la chaîne `"9, Number 9, Number 9"` , vous écririez ceci :

```cs
Console.WriteLine("{0}, Number {0}, Number {0}", 9);
```

**Sachez également qu'il est possible de positionner chaque espace réservé à n'importe quel endroit dans une chaîne littérale**, sans qu'il soit nécessaire de suivre une séquence croissante. Prenons par exemple l'extrait de code suivant :

```cs
// Affiche: 20, 10, 30
Console.WriteLine("{1}, {0}, {2}", 10, 20, 30);
```

Les chaînes peuvent également être formatées à l'aide de l'*interpolation de chaînes*, qui sera abordée plus loin dans ce chapitre.

## Formatage des données numériques

Si vous avez besoin d'un formatage plus élaboré pour les données numériques, chaque espace réservé peut contenir, en option, divers caractères de formatage. Le [[#Tableau 3-3 Caractères de format numérique .NET Core|Tableau 3-3]] présente les options de formatage les plus courantes.

##### Tableau 3-3: Caractères de format numérique .NET Core

| Caractère de format<br> de nombre | Description                                                                                                                                              |
| --------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `C` ou `c`                        | **Utilisé pour formater la devise**. ==Par défaut, l'indicateur sera préfixé au symbole culturel local== (un signe dollar [$] pour l'anglais américain). |
| `D` ou `d`                        | **Utilisé pour formater les nombres décimaux**. Ce drapeau peut également spécifier le nombre minimum de chiffres utilisés pour compléter la valeur.     |
| `E` ou `e`                        | **Utilisé pour la notation exponentielle**. La casse détermine si la constante exponentielle est en majuscule (`E`) ou en minuscule (`e`).               |
| `F` ou `f`                        | **Utilisé pour le formatage en virgule fixe**. Cet indicateur peut également spécifier le nombre minimum de chiffres utilisés pour compléter la valeur.  |
| `G` ou `g`                        | **Représente le caractère générique**. Ce caractère peut être utilisé pour formater un nombre au format fixe ou exponentiel.                             |
| `N` ou `n`                        | **Utilisé pour le formatage numérique de base (avec des virgules)**.                                                                                     |
| `X` ou `x`                        | **Utilisé pour le formatage hexadécimal.** Si vous utilisez un X majuscule, votre format hexadécimal contiendra également des caractères majuscules.     |
| `P` ou `p`                        | **Utilisé pour le formatage pourcentage**. (1 sera affiché 100%, 3 -> 300%,...)                                                                          |

**Ces caractères de format sont ajoutés à une valeur de remplacement donnée à l'aide du symbole deux-points** (par exemple, `{0:C}`, `{1:d}`, `{2:X}`). Pour illustrer cela, mettez à jour les instructions de niveau supérieur afin d'appeler une nouvelle fonction d'aide nommée `FormatNumericalData()`. ==Implémentez cette méthode dans votre fichier *Program.cs* afin de formater une valeur numérique fixe de différentes manières==.

```cs
// Nous allons faire usage de certain caractères de format:
static void FormatNumericalData()
{
    Console.WriteLine("The value 99999 in various formats:");
    Console.WriteLine("c format: {0:c}", 99999);
    Console.WriteLine("d9 format: {0:d9}", 99999);
    Console.WriteLine("f3 format: {0:f3}", 99999);
    Console.WriteLine("n format: {0:n}", 99999);

    // Nottez que la majuscule ou la minuscule dans le formattage
    // détermine si les lettres affiché seront minuscules ou majuscile
    Console.WriteLine("E format: {0:E}", 99999);
    Console.WriteLine("e format: {0:e}", 99999);
    Console.WriteLine("X format: {0:X}", 99999);
    Console.WriteLine("x format: {0:x}", 99999);
}
```

La sortie suivante montre le résultat de l'appel à la méthode `FormatNumericalData()` :

```
The value 99999 in various formats:
c format: € 99.999,00
d9 format: 000099999
f3 format: 99999,000
n format: 99.999,000
E format: 9,999900E+004
e format: 9,999900e+004
X format: 1869F
x format: 1869f
```

>[!example] Exemple de formatage:
>Vous trouverez d'autres exemples de formatage dans ce texte, mais si vous souhaitez approfondir le sujet, consultez la rubrique « Formatting Types » (Types de formatage) dans la [documentation](https://learn.microsoft.com/en-us/dotnet/standard/base-types/standard-numeric-format-strings)

## Formatage des données numériques au-delà des applications console

En note finale, comprenez que l'utilisation des caractères de formatage de chaîne n'est pas limitée aux programmes console. **Cette même syntaxe de formatage peut être utilisée lors de l'appel de la méthode statique `string.Format()`**. ==Cela peut être utile lorsque vous devez composer des données textuelles lors de l'exécution pour les utiliser dans n'importe quel type d'application== (par exemple, une application GUI de bureau, une application web ASP.NET, ...)

**La méthode `string.Format()` renvoie un nouvel objet chaîne, qui est formaté selon les
indicateurs fournis**. Le code suivant formate une chaîne en hexadécimal :

```cs
// Utilisation de string.Format() pour formatter une chaine de caractères littérale.
string userMessage = string.Format("100000 in hex is {0:x}", 100000);
```

# Travailler avec les types de données système et les mots-clés C# correspondants

Comme tout langage de programmation, **C# définit des *mots-clés* pour les types de données fondamentaux, qui sont utilisés pour représenter les variables locales, les variables membres de données de classe, les valeurs de retour de méthode et les paramètres**. Le [[#Tableau 3-4 Tous les types de donnée primitifs de C|Tableau 3-4]] répertorie ==chaque type de données système, sa plage, le mot-clé C# correspondant et la conformité du type avec la spécification `CLS`==. 

>Tous les types système se trouvent dans l'espace de noms `System` ne sont pas dans le tableau pour plus de lisibilité.

##### Tableau 3-4: Tous les types de donnée *primitifs* de C#

| Abréviation C# | CLS Compliant? | Type Système | Portée                                                                             | Description                                             |
| -------------- | -------------- | ------------ | ---------------------------------------------------------------------------------- | ------------------------------------------------------- |
| `bool`         | Oui            | `Boolean`    | true ou false                                                                      | Représente la véracité ou la fausseté.                  |
| `sbyte`        | Non            | `SByte`      | De  $-128$ à $127$                                                                 | Entier de 8 bits signé                                  |
| `byte`         | Oui            | `Byte`       | De $0$ à $255$                                                                     | Entier de 8 bits non signé                              |
| `short`        | Oui            | `Int16`      | De $–32\;768$ à $32\;767$                                                          | Entier de 16 bits signé                                 |
| `ushort`       | Non            | `UInt16`     | De $0$ à $65\;535$                                                                 | Entier de 16 bits non signé                             |
| `int`          | Oui            | `Int32`      | De $–2\;147\;483\;648$ à $2\;147\;483\;647$                                        | Entier de 32 bits signé                                 |
| `uint`         | Non            | `UInt32`     | De $0$ à $4\;294\;967\;295$                                                        | Entier de 32 bits non signé                             |
| `long`         | Oui            | `Int64`      | De $–9\;223\;372\;036\;854\;775\;808$ à $9\;223\;372\;036\;854\;775\;807$          | Eniter de 64 bits signé                                 |
| `ulong`        | Non            | `UInt64`     | De $0$ à $18\;446\;744\;073\;709\;551\;615$                                        | Entier de 64 bits non signé                             |
| `char`         | Oui            | `Char`       | De `U+0000` à  `U+ffff`                                                            | Caractère unique Unicode 16 bits                        |
| `float`        | Oui            | `Single`     | Epsilon: $±1.4\times10^{-45}$<br>Min/Max$–3.4 \times10^{38}$ à $+3.4\times10^{38}$ | Nombre à virgule flottante 32 bits                      |
| `double`       | Oui            | `Double`     | Epsilon: $5.0 \times10^{-324}$  <br>Min/Max: $±1.7\times10^{308}$                  | Nombre à virgule flottante 64 bits                      |
| `decimal`      | Oui            | `Decimal`    | Nbre à virgule: $±1.0\times10^{-28}$<br>Min/Max: $±7.9\times10^{28}$               | Nombre 128 bits signé                                   |
| `string`       | Oui            | `String`     | Limité par la mémoire système.                                                     | Représente un ensemble de caractères Unicode.           |
| `object`       | Oui            | `Object`     | Peut stocké tout type de donnée dans une variable objet                            | La classe de base de tous les types dans l'univers .NET |

>[!tip]- Rappel du [[Chapitre 1#Comprendre le Common Language Specification (CLS)|Chapitre 1]]: 
>Le code .NET Core `CLS-Compliant` peut être utilisé part tous les autre language de programmation .NET. Si vous exposez des données de votre programme qui ne sont pas `CLS-compliant`, les autres language .NET pourrait ne pas être en mesure de l'utiliser.

## Comprendre la déclaration et l'initialisation des variables

Lorsque vous déclarez une variable locale (par exemple, une variable dans la portée d'un membre), vous le faites en **spécifiant le type de données**, **suivi du nom de la variable**. Pour commencer, créez un nouveau projet d'application console nommé *BasicDataTypes* et ajoutez-le à la solution à l'aide des commandes suivantes :

```
dotnet new console -lang c# -n BasicDataTypes -o ./BasicDataTypes
dotnet sln ./chapter3_allProjects.sln add ./BasicDataTypes
```

Mettez à jour le code comme suit :

```cs
using System.Numerics;

Console.WriteLine("***** Fun with Basic Data Types *****");
```

Maintenant, ajoutez la fonction locale statique suivante et appelez-la à partir des instructions de niveau supérieur :

```cs
static void LocalVarDeclarations()
{
    Console.WriteLine("=> Data Declaration");
    // Les variables locals sont déclarées comme ceci:
    // typeDonnée nomVariable;
    int myInt;
    string myString;
    Console.WriteLine();
}
```

==Sachez que l'utilisation d'une variable locale avant de lui avoir attribué une valeur initiale constitue une **erreur de compilation (dans l'éditeur, des avertissement sont affiché en jaune)**==. Il est donc recommandé d'attribuer une valeur initiale à vos points de données locaux au moment de leur déclaration. Vous pouvez le faire sur une seule ligne ou en séparant la déclaration et l'attribution en deux instructions de code.

```cs
static void LocalVarDeclarations()
{
    Console.WriteLine("=> Data Declaration");
    // Local variables are declared as so:
    // dataType varName;
    int myInt = 0;
    string myString;
    myString = "This is my character data";
    
    Console.WriteLine();
}
```

**Il est également possible de déclarer plusieurs variables du même type sous-jacent sur une seule ligne de code, comme dans les trois variables booléennes suivantes :**

```cs
static void LocalVarDeclarations()
{
    Console.WriteLine("=> Data Declarations:");
    int myInt = 0;
    string myString;
    myString = "This is my character data";

    // Declare 3 booléen sur une seule ligne.
    bool b1 = true, b2 = false, b3 = b1;

    Console.WriteLine();
}
```

Étant donné que le mot-clé `bool` en C# est simplement une notation abrégée pour la structure `System.Boolean`, ==il est également possible d'allouer n'importe quel type de données en utilisant son nom complet== (bien sûr, cela vaut également pour tous les mots-clés de type de données en C#). **Voici l'implémentation finale de `LocalVarDeclarations()`**, qui illustre différentes façons de déclarer une variable locale :

```cs
static void LocalVarDeclarations()
{
    Console.WriteLine("=> Data Declaration");
    // Les variables locals sont déclarées comme ceci:
    // typeDonnée nomVariable;
    int myInt = 0;
    
    string myString;
    myString = "this is my character data";

    // Declare 3 bools on a single line.
    bool b1 = true, b2 = false, b3 = b1;

    // Utilisation du type de donnée System.Boolean pour déclarer un bool.
    System.Boolean b4 = false;
    Console.WriteLine("Your data: {0}, {1}, {2}, {3}, {4}, {5}",
        myInt, myString, b1, b2, b3, b4);
    Console.WriteLine();
}
```

### Valeur littérale par défaut (Nouveauté C# 7.1)

**La valeur littérale `default` attribue à une variable la valeur par défaut pour son type de données**. Cela fonctionne pour les types de données standard ainsi que pour les classes personnalisées ([[Chapitre 5#Présentation du type de classe C|Chapitre 5]]) et les types génériques ([[Chapitre 10#Premier aperçu des collections génériques `<T>`|Chapitre 10]]). Créez une nouvelle méthode nommée `DefaultDeclarations()` et ajoutez le code suivant :

```cs
static void DefaultDeclarations()
{
    Console.WriteLine("=> Default Declarations:");
    int myInt = default;
    Console.WriteLine(myInt);
}
```

## Utilisation des types de données intrinsèques et l'opérateur `new` (MaJ C# 9.0)

**Tous les types de données intrinsèques prennent en charge ce que l'on appelle un *constructeur par défaut*** (voir [[Chapitre 5#Comprendre le rôle du constructeur par défaut|Chapitre 5]]). ==Cette fonctionnalité vous permet de créer une variable, avec l'aide du mot-clé `new`, qui attribue automatiquement à la variable sa valeur par défaut== :

- Les variables `bool` sont définies sur `false`.
- Les données *numériques* sont définies sur `0` (ou `0,0` dans le cas des types de données à virgule flottante).
- Les variables `char` sont définies sur ==un seul caractère vide==.
- Les variables `BigInteger` sont définies sur `0`.
- Les variables `DateTime` sont définies sur `1/1/0001 12:00:00 AM`.
- Les références `object` (y compris `string`) sont définies sur `null`.

>[!note] Le type de données `BigInteger` mentionné dans la liste précédente sera expliqué dans un instant.

Bien qu'il soit plus fastidieux d'utiliser le mot-clé `new` lors de la création d'une variable de type de données de base, le code C# suivant est syntaxiquement bien formé :

```cs
static void NewingDataTypes()
{
    Console.WriteLine("=> Using new to create variables:");
    bool b = new bool();           // Assigné à false.
    int i = new int();             // Assingé à 0.
    double d = new double();       // Assigné à 0.
    DateTime dt = new DateTime();  // Assigné à 1/1/0001 12:00:00 AM
    Console.WriteLine("{0}, {1}, {2}, {3}", b, i, d, dt);
    Console.WriteLine();
}

```

**C# 9.0 a ajouté un raccourci pour créer des instances de variables. Ce raccourci consiste simplement à utiliser le mot-clé `new()` sans le type de données. La version mise à jour de `NewingDataTypes` est présentée ici :**

```cs
static void NewingDataTypes()
{
    Console.WriteLine("=> Using new to create variables:");
    bool b = new();       // Assigné à false.
    int i = new();        // Assingé à 0.
    double d = new();     // Assigné à 0.
    DateTime dt = new();  // Assigné à 1/1/0001 12:00:00 AM
    Console.WriteLine("{0}, {1}, {2}, {3}", b, i, d, dt);
    Console.WriteLine();
}
```

## Comprendre la hiérarchie des classes de types de données

**Il est intéressant de noter que même les types de données primitifs .NET sont organisés dans une *hiérarchie de classes***. Si vous découvrez le monde de l'héritage, vous trouverez tous les détails dans le [[Chapitre 6#Comprendre les mécanismes fondamentaux de l'héritage|Chapitre 6]]. En attendant, ==retenez simplement que les types situés au sommet d'une hiérarchie de classes fournissent certains comportements par défaut qui sont accordés aux types dérivés==. La relation entre ces types système de base peut être comprise comme illustré dans l'image ci-présente:.

![[Figure 3.2.png|La hiérarchie des classes des types System]]

**Notez que chaque type dérive en fin de compte de `System.Object`, qui définit un ensemble de méthodes** (par exemple, `ToString()`, `Equals()`, `GetHashCode()`) **communes à tous les types des bibliothèques de classes de base .NET Core** (ces méthodes sont décrites en détail au [[Chapitre 6#Comprendre la classe parente ultime `System.Object`|Chapitre 6]]).

**Notez également que de nombreux types de données numériques dérivent d'une classe nommée `System.ValueType`**. Les descendants de `ValueType` sont automatiquement alloués sur la *pile (Stack)* et ont donc une durée de vie prévisible et sont très efficaces. En revanche, les types qui n'ont pas `System.ValueType` dans leur chaîne d'héritage (tels que `System.Type`, `System.String`, `System.Array`, `System.Exception` et `System.Delegate`) ne sont pas alloués dans le *stack*, mais sur le *tas (heap)*, collecté par le ramasse-miettes (==garbage collector==).

>Vous trouverez plus d'informations sur cette distinction au [[Chapitre 4#Comprendre les types de valeur et les types de référence|Chapitre 4]]

Sans trop s'attarder sur les détails de `System.Object` et `System.ValueType`, il suffit de comprendre que, ==comme un mot-clé C#== (tel que `int`) ==est simplement une notation abrégée pour le type système correspondant== (dans ce cas, `System.Int32`), ==la syntaxe suivante est parfaitement légale, étant donné que `System.Int32`== (le int C#) ==dérive finalement de `System.Object` et peut donc invoquer n'importe lequel de ses membres publics==, comme l'illustre cette fonction d'aide supplémentaire :

```cs
static void ObjectFunctionality()
{
    Console.WriteLine("=> System.Object Functionality:");

    // Un entier C# est réellement une abréviation pour System.Int32,
    // qui hérite des membres suivant de System.Object.
    Console.WriteLine("12.GetHashCode() = {0}", 12.GetHashCode());
    Console.WriteLine("12.Equals(23) = {0}", 12.Equals(23));
    Console.WriteLine("12.ToString() = {0}", 12.ToString());
    Console.WriteLine("12.GetType() = {0}", 12.GetType());
    Console.WriteLine();
}
```

Si vous appeliez cette méthode à partir des instructions de niveau supérieur, vous obtiendriez le résultat suivant :

```
=> System.Object Functionality:
12.GetHashCode() = 12
12.Equals(23) = False
12.ToString() = 12
12.GetType() = System.Int32
```

## Comprendre les membres des types de données numériques

Pour continuer à expérimenter avec les types de données primitifs de C#, ==il faut comprendre que les types numériques de .NET Core prennent en charge les propriétés `MaxValue` et `MinValue` qui fournissent des informations sur la plage qu'un type donné peut stocker==. En plus des propriétés `MinValue`/`MaxValue`, un type numérique donné peut définir d'autres membres utiles. Par exemple, ==le type `System.Double` vous permet d'obtenir les valeurs pour *epsilon* et l'*infini*== (ce qui peut intéresser ceux d'entre vous qui ont un penchant pour les mathématiques). Pour illustrer cela, considérez la fonction d'aide suivante :

```cs
static void DataTypeFunctionality()
{
    Console.WriteLine("=> Data type Functionality:");
    Console.WriteLine("Max of int: {0}", int.MaxValue);
    Console.WriteLine("Min of int: {0}", int.MinValue);
    Console.WriteLine("Max of double: {0}", double.MaxValue);
    Console.WriteLine("Min of double: {0}", double.MinValue);
    Console.WriteLine("double.Epsilon: {0}", double.Epsilon);
    Console.WriteLine("double.PositiveInfinity: {0}",
        double.PositiveInfinity);
    Console.WriteLine("double.NegativeInfinity: {0}",
        double.NegativeInfinity);
    Console.WriteLine();
}
```

**Lorsque vous définissez un nombre entier littéral (tel que `500`), le runtime attribuera par défaut le type de données `int`. De même, les données littérales à virgule flottante (telles que `55,333`) seront attribuées par défaut au type `double`. Pour définir le type de données sous-jacent comme `long`, utilisez le suffixe `l` ou `L` (`4L`). Pour déclarer une variable `float`, utilisez le suffixe `f` ou `F` après la valeur numérique brute (`5,3F`), et utilisez le suffixe `m` ou `M` après un nombre à virgule flottante pour déclarer un nombre décimal (`300,5M`). Cela devient plus important lors de la déclaration implicite de variables, qui sera abordée plus loin dans ce chapitre**.

## Comprendre les membres de `System.Boolean`

Ensuite, considérons le type de données `System.Boolean`. **La seule affectation valide qu'un bool C# peut prendre provient du set: `{true || false}`**. Compte tenu de ce point, il devrait être clair que `System.Boolean` ne prend pas en charge un ensemble de propriétés `MinValue`/`MaxValue`, ==mais plutôt `TrueString`/`FalseString` (qui produit respectivement la chaîne « True » ou « False », respectivement)==. Voici un exemple :

```cs
Console.WriteLine("bool.FalseString: {0}", bool.FalseString);
Console.WriteLine("bool.TrueString: {0}", bool.TrueString);
```

## Comprendre les membres de `System.Char`

**Les données textuelles C# sont représentées par les mots-clés `string` et `char`, qui sont des notations abrégées simples pour `System.String` et `System.Char`, tous deux *Unicode* sous le capot**. Comme vous le savez peut-être déjà, ==une chaîne représente un ensemble contigu de caractères (par exemple, « Bonjour »), tandis que le caractère peut représenter un seul emplacement dans une chaîne (par exemple, « H »)==. 

Le type `System.Char` vous offre de nombreuses fonctionnalités au-delà de la capacité de contenir un seul point de données de caractères. ==À l'aide des méthodes statiques de `System.Char`, vous pouvez déterminer si un caractère donné est numérique, alphabétique, un signe de ponctuation ou autre==. Considérez la méthode suivante :

```cs
static void CharFunctionality()
{
    Console.WriteLine("=> char type Functionality:");
    char myChar = 'a';
    Console.WriteLine("char.IsDigit('a'): {0}", char.IsDigit(myChar));
    Console.WriteLine("char.IsLetter('a'): {0}", char.IsLetter(myChar));
    Console.WriteLine($"{nameof(char.IsDigit)}({3}) = {char.IsDigit('3')}");
    Console.WriteLine("char.IsWhiteSpace('Hello There', 5): {0}",
        char.IsWhiteSpace("Hello There", 5));
    Console.WriteLine("char.IsWhiteSpace('Hello There', 6): {0}",
        char.IsWhiteSpace("Hello There", 6));
    Console.WriteLine("char.IsPunctuation('?'): {0}",
        char.IsPunctuation('?'));
    Console.WriteLine();
}
```

Comme illustré dans la méthode précédente, de nombreux membres de `System.Char` ont deux conventions d'appel : ==un caractère unique ou une chaîne avec un index numérique qui spécifie la position du caractère à tester==.

## Convertir des valeurs à partir de données de type chaîne 

**Les types de données .NET permettent de générer une variable de leur type sous-jacent à partir d'un équivalent textuel (*parsing*)**. Cette technique peut s'avérer extrêmement utile lorsque vous souhaitez **convertir** certaines données saisies par l'utilisateur (telles qu'une sélection dans une liste déroulante d'une interface graphique) en une valeur numérique. Considérez la logique d'analyse suivante dans une méthode nommée `ParseFromStrings()` :

```cs
static void ParseFromStrings()
{
    Console.WriteLine("=> Data type parsing:");
    bool b = bool.Parse("True");
    Console.WriteLine("Value of b: {0}", b);
    double d = double.Parse("99.884");
    Console.WriteLine("Value of d: {0}", d);
    int i = int.Parse("8");
    Console.WriteLine("Value of i: {0}", i);
    char c = Char.Parse("w");
    Console.WriteLine("Value of c: {0}", c);
    Console.WriteLine();
}
```

## Utilisation de `TryParse` pour convertir les valeurs à partir de données de type chaîne

**L'un des problèmes du code précédent est qu'une exception sera levée si la chaîne ne peut pas être convertie correctement en type de données approprié**. Par exemple, le code suivant échouera lors de l'exécution :

```cs
bool b = bool.Parse("Hello");
```

==Une solution consiste à encapsuler chaque appel à `Parse()` dans un bloc `try-catch`== (la gestion des exceptions est traitée en détail au [[Chapitre 7#Les éléments constitutifs de la gestion des exceptions .NET|Chapitre 7]]), ce qui peut ajouter beaucoup de code, ==ou à utiliser une instruction `TryParse()`==. **L'instruction `TryParse()` prend un paramètre `out`** (le modificateur out est décrit en détail au [[Chapitre 4#Utilisation du modificateur `out` (C 7.0)|Chapitre 4]]) **et renvoie une valeur booléenne si la conversion a réussi**. Créez une nouvelle méthode nommée `ParseFromStringWithTryParse()` et ajoutez le code suivant :

```cs
static void ParseFromStringsWithTryParse()
{
    Console.WriteLine("=> Data type parsing with TryParse:");
    if (bool.TryParse("True", out bool b))
    {
        Console.WriteLine("Value of b: {0}", b);
    }
    else
    {
        Console.WriteLine("Default value of b: {0}", b);
    }

    string value = "Hello";
    if (double.TryParse(value, out double d))
    {
        Console.WriteLine("Value of d: {0}", d);
    }
    else
    {
        Console.WriteLine("Failed to convert the input ({0}) to a double and the variable was " +
            "assigned the default {1} ", value, d);
    }
    Console.WriteLine();
}
```

Si vous débutez en programmation et que vous ne savez pas comment fonctionnent les instructions `if/else`, elles sont expliquées en détail plus loin dans ce chapitre. **L'élément important à retenir de l'exemple précédent est que si une chaîne peut être convertie au type de données demandé, la méthode `TryParse()` renvoie `true` et attribue la valeur analysée à la variable transmise à la méthode. Si la valeur ne peut pas être analysée, la variable se voit attribuer sa valeur par défaut, et la méthode `TryParse()` renvoie `false`**.

## Utilisation de `System.DateTime` et `System.TimeSpan` (MaJ C# 10.0)

L'espace de noms `System` définit quelques types de données utiles pour lesquels il n'existe pas de mots-clés C#, tels que les structures `DateTime` et `TimeSpan`.

**Le type `DateTime` contient des données qui représentent une date (mois, jour, année) et une heure spécifiques, qui peuvent toutes deux être formatées de différentes manières à l'aide des membres fournis. La structure `TimeSpan` vous permet de définir et de transformer facilement des unités de temps à l'aide de divers membres.**

```cs
static void UseDatesAndTimes()
{
    Console.WriteLine("=> Dates and Times:");

    // Ce constructeur prend (années, mois, jours).
    DateTime dt = new DateTime(2024, 10, 23);

    // Quelle jour du mois est-ce?
    Console.WriteLine("the day of {0} is {1}", dt.Date, dt.DayOfWeek);

    // Le mois est maintenant décembre
    dt = dt.AddMonths(2);
    Console.WriteLine("Daylight savings: {0}", dt.IsDaylightSavingTime());

    // Ce constructeur prend (Heures, minutes, secondes).
    TimeSpan ts = new TimeSpan(4, 30, 0);
    Console.WriteLine(ts);

    // Soustrait 15 minutes à l'intervale de temps (TimeSpan) actuelle et
    // Affiche le résultat.
    Console.WriteLine(ts.Subtract(new TimeSpan(hours: 0, minutes: 15, seconds: 0)));
}
```

==Les structures ( `struct` ) `DateOnly` et `TimeOnly` ont été ajoutées dans .NET 6/C# 10, et chacune représente la moitié du type `DateTime`==. **Le `struct` `DateOnly` s'aligne sur le type `Date` de *SQL Server*, et le `struct` `TimeOnly` s'aligne sur le type `Time` de *SQL Server***. Le code suivant montre les nouveaux types en action :

```cs
static void UseDatesAndTimes()
{
    Console.WriteLine("=> Dates and Times:");
...

    DateOnly d = new DateOnly(2021,07,21);
    Console.WriteLine(d);

    TimeOnly t = new TimeOnly(13,30,0,0);
    Console.WriteLine(t);
}
```

## Travailler avec l'espace de noms `System.Numerics`

L'espace de noms `System.Numerics` définit un `struct` nommée `BigInteger`. Comme son nom l'indique, **le type de données `BigInteger` peut être utilisé lorsque vous avez besoin de représenter des valeurs numériques gigantesques, qui ne sont pas limitées par une limite supérieure ou inférieure fixe**.

>[!Note]- Note sur le type `Complex`:
>L'espace de noms `System.Numerics` définit un deuxième `struct` nommée `Complex`, qui vous permet de modéliser mathématiquement des données numériques complexes (==par exemple, des unités imaginaires, des données réelles, des tangentes hyperboliques==). Consultez la [documentation](https://learn.microsoft.com/fr-fr/dotnet/api/system.numerics.complex?view=net-8.0) si vous êtes intéressé.
>
>```cs
>static void UseComplex()
>{
>    // Code en plus du livre, pour utiliser les nombres complexes.
>    Console.WriteLine("=> Use Complex:");
>    Complex c = new(23.43, 1);
>    c += new Complex(3.57, 3);
>    Console.WriteLine($"Real part of c: {c.Real:F4}");
>    Console.WriteLine($"Imaginary part of c: {c.Imaginary:F4}");
>    Console.WriteLine($"Phase of c id rad: {c.Phase:F4}");
>    Console.WriteLine($"Magnitude of c: {c.Magnitude:F4}");
>    Console.WriteLine($"Phase of c in deg (done manually): {c.Phase * (180 / Math.PI):F4}");
>    Console.WriteLine("Phase of c in deg (with RadiansToDegrees method): {0:F4}",
>        double.RadiansToDegrees(c.Phase));
>}
>```

Même si la plupart de vos applications .NET Core n'auront probablement jamais besoin d'utiliser la structure `BigInteger`, si vous avez besoin de définir une valeur numérique très grande, la première étape consiste à ajouter la *directive* `using` suivante au fichier :

```cs
// BigInteger lives here!
using System.Numerics;
```

À ce stade, vous pouvez créer une variable `BigInteger` à l'aide de opérateur `new`. **Dans le constructeur, vous pouvez spécifier une valeur numérique, y compris des données à virgule flottante**. ==Cependant, C# type implicitement les nombres sans virgule flottant comme `int` et les nombres à virgule flottante comme `double`==. Comment, alors, pouvez-vous définir `BigInteger` sur une valeur massive sans *déborder (Overflow)* les types de données par défaut utilisés pour les valeurs numériques brutes ?

**L'approche la plus simple consiste à définir la valeur numérique massive comme un *littéral de texte***, qui peut être converti en variable `BigInteger` via la méthode statique `Parse()`. *Si nécessaire, vous pouvez également passer un tableau d'octets (`byte[]`) directement au constructeur de la classe `BigInteger`.

>[!Attention] Changer la valeur d'une variable `BigInteger`.
>Une fois que vous avez attribué une valeur à une variable `BigInteger`, vous ne pouvez plus la modifier, car les données sont ***immuables***.
>
>Cependant, la classe `BigInteger` définit un certain nombre de membres qui renvoient de nouveaux objets `BigInteger` en fonction des modifications apportées à vos données (comme la méthode statique `Multiply()` utilisée dans l'exemple de code suivant).

Dans tous les cas, après avoir défini une variable `BigInteger`, vous constaterez que cette classe définit des membres similaires à ceux d'autres types de données primitifs C# (par exemple, `float`, `int`). De plus, la classe `BigInteger` définit plusieurs membres statiques qui vous permettent d'appliquer des expressions mathématiques de base (telles que l'addition et la multiplication) aux variables `BigInteger`. Voici un exemple d'utilisation de la classe `BigInteger` :

```cs
static void UseBigInteger()
{
    Console.WriteLine("=> Use BigInteger:");
    BigInteger biggy =
        BigInteger.Parse("9999999999999999999999999999999999999999999999");
    
    Console.WriteLine("Value of biggy is {0}", biggy);
    Console.WriteLine("Is biggy an even value?: {0}", biggy.IsEven);
    Console.WriteLine("Is biggy a power of two?: {0}", biggy.IsPowerOfTwo);
    BigInteger reallyBig = BigInteger.Multiply(biggy,
        BigInteger.Parse("8888888888888888888888888888888888888888888"));
    Console.WriteLine("Value of reallyBig is {0}", reallyBig);
}
```

Il est également important de noter que le type de données `BigInteger` *répond aux opérateurs mathématiques intrinsèques de C#*, tels que +, - et * . **Par conséquent, plutôt que d'appeler `BigInteger.Multiply()` pour multiplier deux grands nombres, vous pouvez écrire le code suivant :

```cs
BigInteger reallyBig2 = biggy * reallyBig;
```

À ce stade, j'espère que vous comprenez que les mots-clés C# représentant les types de données de base ont un type correspondant dans les bibliothèques de classes de base .NET Core, chacune exposant une fonctionnalité fixe. Bien que je n'aie pas détaillé chaque membre de ces types de données, vous êtes bien placé pour approfondir les détails comme vous le souhaitez. ==N'oubliez pas de consulter la documentation .NET Core pour obtenir des informations complètes sur les différents types de données .NET. Vous serez probablement surpris par le nombre de fonctionnalités intégrées==.

## Utilisation des séparateurs de chiffres (Nouveauté C# 7.0)

Parfois, lorsque vous attribuez des nombres importants à une variable numérique, il y a plus de chiffres que l'œil ne peut en suivre. **C# 7.0 a introduit le trait de soulignement ( `_` ) comme séparateur de chiffres** (pour les types `int`, `long`, `décimal`, `double` ou `hex`). **C# 7.2 permet aux valeurs hexadécimales** (==et au nouveau littéral binaire, abordé ci-après==) **de commencer par un trait de soulignement, après la déclaration d'ouverture**. Voici un exemple d'utilisation du nouveau séparateur de chiffres :

```cs
static void DigitSeparator()
{
    Console.WriteLine("=> Use Digit Separator:");
    Console.Write("Integer: ");
    Console.WriteLine(123_456);
    Console.Write("Long: ");
    Console.WriteLine(123_456_789L);
    Console.Write("Float: ");
    Console.WriteLine(123_456.1234F);
    Console.Write("Double: ");
    Console.WriteLine(123_456.12);
    Console.Write("Decimal: ");
    Console.WriteLine(123_456.12M);
    // Mis à jour en 7.2, les nombres hexadécimaux 
    // peuvent commencer avec _
    Console.Write("Hex: ");
    Console.WriteLine(0x_00_00_FF);
}
```

## Utilisation des littéraux binaires (Nouveauté C# 7.0/7.2)

**C# 7.0 introduit un nouveau *littéral* pour les valeurs binaires, par exemple pour créer des masques de bits**. Le nouveau séparateur de chiffres fonctionne avec les littéraux binaires, et C# 7.2 permet aux nombres binaires et hexadécimaux de commencer par un underscore. **Désormais, les nombres binaires peuvent être écrits comme vous le souhaitez**. Voici un exemple :

```cs
0b_0001_0000
```

Voici une méthode qui montre comment utiliser les nouveaux littéraux avec le séparateur de chiffres :

```cs
static void BinaryLiterals()
{
    //Mis à jour en 7.2, les nomvre binaires peuvent commencer avec _
    Console.WriteLine("=> Use Binary Literals:");
    Console.WriteLine("Sixteen: {0}", 0b_0001_0000);
    Console.WriteLine("Thirty Two: {0}", 0b_0010_0000);
    Console.WriteLine("Sixty Four: {0}", 0b_0100_0000);
}

```
# Travailler avec des donnée `string`

`System.String` fournit un certain nombre de méthodes que l'on peut attendre d'une telle classe utilitaire, notamment des ==méthodes qui renvoient la longueur des données de caractères, recherchent des sous-chaînes dans la chaîne actuelle et convertissent en majuscules/minuscules et vice versa==. Le [[#Tableau 3-5 Sélection de membres de `System.String`|Tableau 3-5]] répertorie certains des membres intéressants (mais en aucun cas tous).

##### Tableau 3-5: Sélection de membres de `System.String`

| Membre de la classe String | Description                                                                                                                                                                                    |
| -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Lengh`                    | Retourne la longueur de la chaîne de caractère.                                                                                                                                                |
| `Compare()`                | Cette méthode statique compare deux chaînes.                                                                                                                                                   |
| `Contains()`               | Détermine si une chaîne contient une sous-chaîne spécifique                                                                                                                                    |
| `Equals()`                 | Teste si deux objets chaînes contiennent les mêmes caractères.                                                                                                                                 |
| `Format()`                 | Cette méthode statique formate une chaîne en utilisant des autre [[#Travailler avec les types de données système et les mots-clés C correspondants\|primitif]]                                 |
| `Insert()`                 | Insert une chaîne dans une chaîne donnée.                                                                                                                                                      |
| `PadLeft()` / `PadRight()` | Ces méthodes sont utilisées pour remplir une chaîne avec certains caractères.                                                                                                                  |
| `Remove()` / `Replace()`   | Ces méthodes sont utilisé pour récupéré une copie de la chaîne avec les modifications. (caractères supprimés ou remplacés).                                                                    |
| `Split()`                  | Cette méthode retourne un tableau de chaîne contenant les sous-chaînes dans cette instance que sont délimités par des éléments spécifiés dans un tableau de caractère ou un tableau de chaîne. |
| `Trim()`                   | Cette méthode supprime toutes les occurrences d'un ensemble de caractères spécifiés au<br>début et à la fin de la chaîne actuelle.                                                             |
| `ToUpper()` / `ToLower()`  | Ces méthodes créent une copie de la chaîne de caractère en majuscule ou minuscule, respectivement.                                                                                             |

## Effectuer des manipulations de base sur les `String`

L'utilisation des membres de `System.String` se fait comme vous pouvez vous y attendre. Il suffit de déclarer une variable `string` et d' utiliser les fonctionnalités fournies via l'opérateur *point*(`.`). **Notez que certains membres de `System.String` sont des membres statiques et sont donc appelés au niveau de la classe** (plutôt qu'au niveau de l'objet). 

Supposons que vous ayez créé un nouveau projet d'application console nommé `FunWithStrings` et que vous l'ayez ajouté à votre solution. Effacez le code existant et ajoutez ce qui suit :

```cs
using System.Runtime.CompilerServices;
using System.Text;

BasicStringFunctionality();

static void BasicStringFunctionality()
{
    Console.WriteLine("=> Basic String functionality:");
    string firstName = "Freddy";
    Console.WriteLine("Value of firstName: {0}", firstName);
    Console.WriteLine("firstName has {0} characters.", firstName.Length);
    Console.WriteLine("firstName in uppercase: {0}", firstName.ToUpper());
    Console.WriteLine("firstName in lowercase: {0}", firstName.ToLower());
    Console.WriteLine("firstName contains the letter y?: {0}",
        firstName.Contains("y"));
    Console.WriteLine("New first name: {0}", firstName.Replace("dy", ""));
    Console.WriteLine();
}
```

Il n'y a pas grand-chose à dire ici, car cette méthode invoque simplement divers membres, tels que `ToUpper()` et `Contains()`, sur une variable de chaîne locale pour produire divers formats et transformations. Voici le résultat initial :

```
=> Basic String functionality:
Value of firstName: Freddy
firstName has 6 characters.
firstName in uppercase: FREDDY
firstName in lowercase: freddy
firstName contains the letter y?: True
firstName after replace: Fred
```

Bien que ce résultat puisse sembler peu surprenant, le résultat obtenu en appelant la méthode `Replace()` est un peu trompeur. En réalité, la variable `firstName` n'a pas changé du tout; **vous recevez plutôt une nouvelle chaîne dans un format modifié**. ==Vous reviendrez sur la nature immuable des chaînes dans quelques instants

## Effectuer une concaténation de `String`

On peut ajouter des chaînes entre elles avec l'opérateur c# `+`. Considérer cette méthode d'aide suivante:

```cs
static void StringConcatenation()
{
    Console.WriteLine("=> String concatenation:");
    string s1 = "Programming the ";
    string s2 = "PsychoDrill (PTP)";
    string s3 = s1 + s2;
    Console.WriteLine(s3);
    Console.WriteLine();
}
```

le symbole `+` est traité par le compilateur comme un appel à la méthode `String.Concat()`. On peut donc effectuer les mêmes opérations qu'au dessus en appelant directement `String.Concat()`, mais c'est beaucoup plus long à écrire et ne change rien sous le capot de C#.

```cs
static void StringConcatenation()
{
    Console.WriteLine("=> String concatenation:");
    string s1 = "Programming the ";
    string s2 = "PsychoDrill (PTP)";
    string s3 = String.Concat(s1, s2);
    Console.WriteLine(s3);
    Console.WriteLine();
}
```

>[!tip]- Simplification des appels des membre de la classe `String`.
>Les EDI modernes (avec le nouvel analyseur de code *roslyn*) indique que l'on peut simplifier l'appel `String.Concat()` par cet appel -> `string.Concat()`. Cette simplification est du même registre que pour l'appel de la méthode `Parse` (vu [[#Convertir des valeurs à partir de données de type chaîne|précédement]]): `int.Parse()` au lieu de `Int32.Parse()`
>
>>[!quote] [Documentation miscrosoft](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/strings/#:~:text=In%20C%23%2C%20the%20string%20keyword,%2C%20manipulating%2C%20and%20comparing%20strings.):
>>En C#, le mot-clé `string` est un alias de `String` ; par conséquent, `String` et `string` sont équivalents. **Utilisez l'alias `string` fourni, car il fonctionne même sans `System`**. La classe `String` offre de nombreuses méthodes pour créer, manipuler et comparer des chaînes de caractères en toute sécurité.

## Utilisation des caractères d'échappement

Comme dans les autres langages basés sur le C, les littéraux de chaîne C# peuvent contenir divers **caractères d'échappement, qui déterminent la manière dont les données de caractère doivent être imprimées dans le flux de sortie**. ==Chaque caractère d'échappement commence par une barre oblique inversée, suivie d'un jeton spécifique==. Si vous n'êtes plus très au fait de la signification de ces caractères d'échappement, le [[#Tableau 3-6 Caractères d'échappement pour les chaînes littérales.|Tableau 3-6]] répertorie les options les plus courantes.

##### Tableau 3-6: Caractères d'échappement pour les chaînes littérales.

| Caractère | Desciption                                                                                                            |
| --------- | --------------------------------------------------------------------------------------------------------------------- |
| `\'`      | Insère un apostrophe dans la chaîne.                                                                                  |
| `\"`      | Insère un guillemet dans la chaîne.                                                                                   |
| `\\`      | Insère un backslash. Utile quand il faut garder en mémoire un chemin d'accès.                                         |
| `\a`      | Déclenche une alerte système (bip). Pour les application console, cela peut être un indice sonore pour l'utilisateur. |
| `\n`      | Insère un saut de ligne (systèmes basés sur Unix).                                                                    |
| `\r\n`    | Insère un saut de ligne (système pas basés sur Unix).                                                                 |
| `\r`      | Insère un retour à la ligne.                                                                                          |
| `\t`      | Insère une tabulation horizontale dans la chaîne littérale.                                                           |
| `\v`      | Insère une tabulation verticale dans la chaîne littérale. (*Peut ne pas fonctionner sur Windows.*)                    |

Par exemple, pour imprimer une chaîne contenant une tabulation entre chaque mot, vous pouvez utiliser le caractère d'échappement `\t`. Ou supposons que vous souhaitiez créer une chaîne littérale contenant des guillemets, une autre définissant un chemin d'accès à un répertoire et une dernière chaîne littérale insérant trois lignes vides après l'impression des données de caractère. Pour ce faire sans erreur de compilation, vous devrez utiliser les caractères d'échappement `\", \\ et \n`. La méthode suivante illustre cette procédure :

```cs
static void EscapeChars()
{
    Console.WriteLine("=> Ecape characters:");
    string strWithTabs = "Model\tColor\tSpeed\tPet Name";
    Console.WriteLine(strWithTabs);

	Console.WriteLine("Everyone loves \"Hello World\"");
    Console.WriteLine("C:\\MyApp\\bin\\Debug");

    // Ajoute un total de 4 lignes vides (3 avec les échappements, 1 venant de WriteLine)
    Console.WriteLine("All finished \n\n\n ");
    Console.WriteLine();
}
```

Remarquez dans le [[#Tableau 3-6 Caractères d'échappement pour les chaînes littérales.|Tableau 3-6]] qu'il existe une différence lors de la création d'une nouvelle ligne en fonction du système d'exploitation sur lequel le code s'exécute. **La propriété `NewLine` du type statique `Environment` ajoute le ou les codes d'échappement appropriés pour ajouter des lignes vides dans votre texte**. Considérez l'ajout suivant à la méthode `EscapeChars()` :

```cs
static void EscapeChars()
{
    ...
    // Ajoute 4 lignes vide de plus
	Console.WriteLine("All finished for real this time.{0}{0}{0}",
	    Environment.NewLine);
}
```

## Utiliser l'interpolation de chaînes

La syntaxe des accolades illustrée dans ce chapitre (`{0}`, `{1}`, etc.) existe dans la plateforme .NET depuis la version 1.0. **Depuis la sortie de C#6, les développeur C# peuvent utiliser une syntaxe alternative pour construire des `string` qui possèdent des espaces dédiés. Cela s'appelle l'interpolation de chaînes**. Bien que le résultat de l'opération soit identique à la syntaxe traditionnelle de formatage de chaînes, ==cette nouvelle approche vous permet d'intégrer directement les variables elles-mêmes, plutôt que de les ajouter sous forme de liste délimitée par des virgules==.

Considérez la méthode supplémentaire suivante de votre classe `Program` nommée `StringInterpolation()`, qui crée une variable de chaîne à l'aide de chaque approche :

```cs
static void StirngInterpolation()
{
    Console.WriteLine("=> String Interpolation:");

    // Des variables locales que l'on viendra intercalé dans la plus grandes chaînes.
    int age = 4;
    string name = "Soren";

    // Utilisation de la syntaxe avec des accolades.
    string greetings = string.Format("Hello {0} you are {1} years old.", 
	    name, 
	    age
	);
    Console.WriteLine(greetings);

    // Utilisation de l'interpolation des chaînes
    string greeting2 = $"Hello {name} you are {age} years old.";
    Console.WriteLine(greeting2);
}
```

**Notez que la variable `greetings2` commence, avant les guillemets, par un dollar (`$`).**  Avec l'interpolation, ==on peut directement mettre le nom de la variable dans les accolades. C'est beaucoup plus lisible car on n'a pas besoin de faire des "*va et viens*" pour savoir quel numéro correspond à quelle variable==.

**Il y a un autre aspect intéressant avec cette nouvelle syntaxe: les accolades sont de véritable portée, ce qui veut dire que l'on peut utiliser l'opérateur (`.`) pour changer l'état des variables. Prenons un exemple:**

```cs
string greeting = string.Format("Hello {0} you are {1} years old.", 
	name.ToUpper(), 
	age
);
string greeting2 = $"Hello {name.ToUpper()} you are {age} years old.";
```

Ici, j'ai mis le nom en majuscules via un appel à `ToUpper()`. ==Notez que dans l'approche par interpolation de chaîne, vous n'ajoutez pas de terminateur point-virgule lorsque vous appelez cette méthode==. **Dans ce cas, vous ne pouvez pas utiliser la portée des accolades comme une portée de méthode complète contenant de nombreuses lignes de code exécutable**. ==Vous pouvez plutôt invoquer un seul membre de l'objet à l'aide de l'opérateur point et définir une expression générale simple telle que {age += 1}==.

Toujours bon à notez que l'interpolation de chaîne est compatible avec les *caractères d'échappement.*

```cs
string greeting = string.Format("\tHello {0} you are {1} years old.", 
	name.ToUpper(), 
	age
);
string greeting2 = $"\tHello {name.ToUpper()} you are {age} years old.";
```

### Améliorations des performances (MaJ C# 10).

Lorsque vous utilisez l'interpolation de chaînes ==dans les versions antérieures à C# 10, le compilateur convertit en arrière-plan l'instruction de chaîne interpolée en un appel `Format()`==. Prenons par exemple une version raccourcie de l'exemple précédent et intégrons-la dans la méthode `Main()` d'une application console .NET 5 (C# 9) nommée `CSharp9Strings`.

>[!note] Voir le [[Chapitre 2#Utiliser une version antérieure du SDK .NET sans avoir à l'installer|Chapitre 2]] pour comment changer la version de .NET à laquelle on veut compiler


```cs
using System;

namespace CSharp9Strings
{
   class Program
    {
        static void Main(string[] args)
        {
            int age = 4;
            string name = "Soren";
            string greeting = string.Format("\tHello {0} you are {1} years old.", name.
            ToUpper(), age);
            string greetings = $"\tHello {name.ToUpper()} you are {age} years old.";
        }
    }
}
```

Lorsque la méthode `Main()` est examinée avec *ILdasm*, vous pouvez voir que l'appel à `Format()` et les appels d'interpolation de chaîne sont implémentés comme les mêmes appels `Format()` dans l'IL :

> Les lignes démarrant avec `IL` avec une tabulation sont les appels à la méthode `Format()`.

```CIL
.method private hidebysig static void Main(string[] args) cil managed
{
    .maxstack 3
    .entrypoint
    .locals init (int32 V_0, string V_1, string V_2)
    IL_0000: nop
    IL_0001: ldc.i4.4
    IL_0002: stloc.0
    IL_0003: ldstr "Soren"
    IL_0008: stloc.1
    IL_0009: ldstr "\tHello {0} you are {1} years old."
    IL_000e: ldloc.1
    IL_000f: callvirt instance string [System.Runtime]System.String::ToUpper()
    IL_0014: ldloc.0
    IL_0015: box [System.Runtime]System.Int32
        IL_001a: call string [System.Runtime]System.String::Format(string, object, object)
    IL_001f: stloc.2
    IL_0020: ldstr "\tHello {0} you are {1} years old."
    IL_0025: ldloc.1
    IL_0026: callvirt instance string [System.Runtime]System.String::ToUpper()
    IL_002b: ldloc.0
    IL_002c: box [System.Runtime]System.Int32
        IL_0031: call string [System.Runtime]System.String::Format(string, object, object)
    IL_0036: stloc.3
    IL_0037: ret
} // end of method Program::Main
```

**Le problème réside dans les performances**. Lorsque `Format()` est appelé lors de l'exécution, ==la méthode analyse la chaîne de format pour trouver les littéraux, les éléments de format, les spécificateurs et les alignements, ce que le compilateur a déjà fait lors de la compilation==. Tous les éléments sont transmis sous forme de `System.Object`, ce qui signifie que les types de valeurs sont encadrés. ==S'il y a plus de trois paramètres, un tableau est alloué==. **Outre les problèmes de performances, `Format()` ne fonctionne qu'avec les types de référence**.

Un changement majeur dans C# 10 est que tout le travail qui peut être effectué au moment de la compilation est conservé dans l'IL à l'aide de `DefaultInterpolatedStringHandler` et de ses méthodes. **Voici le code C# 10 équivalent dans lequel les chaînes interpolées sont converties**:

```cs
static void StringInterpolationWithDefaultInterpolatedStringHandler()
{
	Console.WriteLine("=> String Interpolation Under the Covers:\a");
    int age = 4;
    string name = "Soren";
    var builder = new DefaultInterpolatedStringHandler(3,2);
    builder.AppendLiteral("\tHello ");
    builder.AppendFormatted(name);
    builder.AppendLiteral(" you are ");
    builder.AppendFormatted(age);
    builder.AppendLiteral(" years old.");
    var greeting = builder.ToStringAndClear();
    Console.WriteLine(greeting);
}
```

**La version du constructeur `DefaultInterpolatedStringHandler` utilisée dans cet exemple prend en paramètre deux entiers. Le premier est le nombre de littéraux, et le second est le nombre de variables. Cela permet à l'instance de faire une estimation éclairée de la quantité de mémoire à allouer**. Les littéraux sont ajoutés à l'aide de la méthode `AppendLiteral()`, et les variables sont ajoutées à l'aide de la méthode `AppendFormatted()`.

==Les tests de performance ont montré une amélioration significative des performances dans la gestion des chaînes de caractères en C# 10 lorsque votre code contient des interpolations de chaînes, ce qui est une bonne nouvelle==. **La très bonne nouvelle, c'est que vous n'avez pas besoin d'écrire tout ce code supplémentaire. Le compilateur s'occupe de tout lors de la compilation des interpolations de chaînes**.

## Définir des chaînes bruts (C# 8.0)

**Lorsque vous préfixez une chaîne littérale avec le symbole `@`, vous créez ce que l'on appelle une chaîne textuelle (*Verbatim*)**. En utilisant des chaînes littérales, ==vous désactivez le traitement des caractères d'échappement d'un littéral et affichez une chaîne telle quelle==. **Cela peut être très utile lorsque vous travaillez avec des chaînes représentant des chemins d'accès à des répertoires et à des réseaux**. Par conséquent, plutôt que d'utiliser les caractères d'échappement `\\`, vous pouvez simplement écrire ce qui suit :

```cs
// La chaînes de caractères suivante est affiché de manière brut.
// Ainsi, tous les caractères d'échappements sont affichés.
Console.WriteLine(@"C:\MyApp\bin\Debug");
```

Notez aussi que les chaines verbatim peuvent être utilisé pour préservé les espaces/tabulations pour les chaînes qui s'étendent sur plusieurs lignes.

```cs
// les caractères d'espacements sont préservés avec les chaînes bruts.
string myLongString = @"This is a very
    very
        very
            long string";
Console.WriteLine(myLongString);
```

On peut insérer des des apostrophes en les doublant comme ceci:

```cs
Console.WriteLine(@"Cerebrus said ""Darrr! Pret-ty sun sets""");
```

**Les chaines verbatim peuvent aussi être des chaines interpolées, en spécifiant les deux opérateur (`$` et `@`)**.

```cs
string interp = "interpolation";
string myLongString2 = $@"This is a very
    very
        long string with {interp}";
```

>[!Note]
>*Nouveauté C# 8*: l'ordre n'a pas d'importance. `@$` ou `$@` fonctionnera.

## Travailler avec l'égalité des chaînes.

Comme cela sera expliqué en détail au [[Chapitre 4#Comprendre les types de valeur et les types de référence|Chapitre 4]], un **type de référence est un objet alloué dans le tas**(*heap*), **géré avec ramasse-miettes** (*Garbage Collector*). ==Par défaut, lorsque vous effectuez un test d'égalité sur des types de référence (via les opérateurs C# `= =` et `!=`), vous obtenez la valeur `true` si les références pointent vers le même objet en mémoire==. Cependant, même si le type de données `string` est effectivement un *type de référence*, ==les opérateurs d'égalité ont été redéfinis pour comparer les valeurs des objets chaîne, et non l'objet en mémoire auquel ils font référence==. [^2]

```cs
static void StringEquality()
{
    Console.WriteLine("=> String Equality:");
    string s1 = "Hello!";
    string s2 = "Yo!";
    Console.WriteLine("s1 = {0}", s1);
    Console.WriteLine("s2 = {0}", s2);
    Console.WriteLine();
    // Test d'égalité pour ces chaînes.
    Console.WriteLine("s1 == s2: {0}", s1 == s2);
    Console.WriteLine("s1 == Hello!: {0}", s1 == "Hello!");
    Console.WriteLine("s1 == HELLO!: {0}", s1 == "HELLO!");
    Console.WriteLine("s1 == hello!: {0}", s1 == "hello!");
    Console.WriteLine("s1.Equals(s2): {0}", s1.Equals(s2));
    Console.WriteLine("Yo!.Equals(s2): {0}", "Yo!".Equals(s2));
    Console.WriteLine();
}
```

==Les opérateurs d'égalité C# effectuent par défaut un test d'égalité caractère par caractère, sensible à la casse et insensible à la culture, sur les objets chaîne==. Par conséquent, « Hello! » n'est pas égal à « HELLO! », qui est également différent de « hello! ». ==De plus, en gardant à l'esprit le lien entre `string` et `System.String`, notez que vous pouvez tester l'égalité à l'aide de la méthode `Equals()` de `String` ainsi que des opérateurs d'égalité(`= =`) ==. Enfin, étant donné que chaque littéral de chaîne (tel que `"Yo !"`) est une instance `System.String` valide, ==vous pouvez accéder à des fonctionnalités centrées sur les chaînes à partir d'une séquence fixe de caractères==.

### Modification du comportement de comparaison des chaînes

Comme mentionné précédemment, les opérateurs d'égalité de chaînes (`Compare()`, `Equals()` et `==` ) ainsi que la fonction `IndexOf()` sont ==par défaut sensibles à la casse et insensibles à la culture==. Cela peut poser un problème si votre programme ne tient pas compte de la casse. ==Une façon de contourner ce problème consiste à tout convertir en majuscules ou en minuscules, puis à effectuer la comparaison, comme ceci== :

```cs
if (firstString.ToUpper() == secondString.ToUpper())
{
    //Do something
}
```

==Cela crée une copie de chaque chaîne avec toutes les lettres en minuscules. Cela ne pose probablement pas de problème dans la plupart des cas, mais cela peut nuire aux performances avec une chaîne très longue, voire entraîner un échec en fonction de la culture==. **Une bien meilleure pratique consiste à utiliser les surcharges des méthodes énumérées précédemment qui prennent une valeur de l'*énumération* `StringComparison` pour contrôler exactement la manière dont les comparaisons sont effectuées**. Le [[#Tableau 3-7 Les valeurs de l'énumération `StringComparison`|Tableau 3-7]] décrit les valeurs `StringComparison`.

##### Tableau 3-7: Les valeurs de l'énumération `StringComparison`

| Opérateur d'égalité/comparaison C# | Description                                                                                                                               |
| ---------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| `CurrentCulture`                   | Compare les chaînes à l'aide de règles de tri sensibles à la culture et à la culture actuelle.                                            |
| `CurrentCultureIgnoreCase`         | Compare les chaînes à l'aide de règles de tri sensibles à la culture et à la culture actuelle et ignore la casse des chaînes comparées.   |
| `IvariantCulture`                  | Compare les chaînes à l'aide de règles de tri sensibles à la culture et de la culture invariante.                                         |
| `InvariantCultureIgnoreCase`       | Compare les chaînes à l'aide de règles de tri sensibles à la culture et à la culture invariante et ignore la casse des chaînes comparées. |
| `Ordinal`                          | Compare les chaînes à l'aide des règles de tri ordinal (binaire).                                                                         |
| `OrdinalIgnoreCase`                | Compare les chaînes à l'aide de règles de tri ordinal (binaire) et ignore la casse des chaînes comparées.                                 |

Pour voir l'effet de l'utilisation de l'option `StringComparison`, créez une nouvelle méthode nommée `StringEqualitySpecifyingCompareRules()` et ajoutez le code suivant :

```cs
static void StringEqualitySpecifyingCompareRules()
{
    Console.WriteLine("=> String Equality (Case Insensitive):");
    string s1 = "Hello!";
    string s2 = "HELLO!";
    Console.WriteLine("s1 = {0}", s1);
    Console.WriteLine("s2 = {0}", s2);

    Console.WriteLine();

    // Vérifie les résultats du changement de règle
    // par défault de la comparaison..
    Console.WriteLine($"Default rules: s1={s1}, s2={s2}, s1.Equals(s2): {s1.Equals(s2)}");
    Console.WriteLine(
        $"Ignore case: s1.Equals(s2, StringComparison.OrdinalIgnoreCase): {s1.Equals(s2, StringComparison.OrdinalIgnoreCase)}"
    );
    Console.WriteLine(
        $"Ignore case, Invariant Culture: s1.Equals(s2, StringComparison.InvariantCultureIgnoreCase): {s1.Equals(s2, StringComparison.InvariantCultureIgnoreCase)}"
    );

    Console.WriteLine();

    Console.WriteLine($"Default rules: s1={s1}, s2={s2} s1.IndexOf(\'E\'): {s1.IndexOf('E')}");
    Console.WriteLine(
        $"Ignore case: s1.IndexOf(\'E\', StringComparison.OrdinalIgnoreCase):{s1.IndexOf('E', StringComparison.OrdinalIgnoreCase)}"
    );
    Console.WriteLine(
        $"Ignore case, Invariant Culture: s1.IndexOf(\"E\", StringComparison.InvariantCultureIgnoreCase): {s1.IndexOf('E', StringComparison.InvariantCultureIgnoreCase)}"
    );

    Console.WriteLine();
}
```
   
Bien que les exemples présentés ici soient simples et utilisent les mêmes lettres dans la plupart des cultures, ==si votre application doit prendre en compte différents ensembles culturels, l'utilisation des options `StringComparison` est indispensable==.

## Les chaînes sont immuables

L'un des aspects intéressants de `System.String` est qu'après avoir attribué une valeur initiale à un objet chaîne, **les données de caractères ne peuvent plus être modifiées**. À première vue, cela peut sembler être un mensonge éhonté, étant donné que vous réattribuer toujours des chaînes à de nouvelles valeurs et que le type `System.String` définit un certain nombre de méthodes qui semblent modifier les données de caractères d'une manière ou d'une autre (*comme la conversion en majuscules et en minuscules*). Cependant, **si vous examinez de plus près ce qui se passe en arrière-plan, vous remarquerez que les méthodes du type `string` vous renvoient en fait un nouvel objet `string` dans un format modifié**.

```cs
static void StringsAreImmutable()
{
   Console.WriteLine("=> Immutable Strings:\a");
   // Assigne la valeur initiale à la chaîne
   string s1 = "This is my string.";
   Console.WriteLine("s1 = {0}", s1);
   // s1 majuscule ?
   string upperstring = s1.ToUpper();
   Console.WriteLine("upperstring = {0}", upperstring);
   // Non! s1 est au même format!
   Console.WriteLine("s1 = {0}", s1);
}
```

**Si vous examinez la sortie, vous pouvez vérifier que l'objet `string`original (`s1`) n'est pas mis en majuscule quand on appelle `ToUpper()`**. Plutôt, on nous est retourné une *copie* du `string` dans un format modifié.

```
=> Immutable Strings:
s1 = This is my string.
upperstring = THIS IS MY STRING.
s1 = This is my string.
```

La même loi d'immuabilité reste vraie quand on utilise l'opérateur d'assignation C# (`=`). Pour illustrer, Implémentez la méthode suivante `StringsAreImmutable2()`:

```cs
static void StringsAreImmutable2()
{
   Console.WriteLine("=> Immutable Strings 2:\a");
   string s2 = "My other string";
   s2 = "New string value";
   Console.WriteLine(s2);
   Console.WriteLine();
}
```

Maintenant, compilez votre application et exécutez *ildasm.exe* (voir [[Chapitre 1#Exploration d'un assembly à l'aide de ildasm.exe|Chapitre 1]]). La sortie suivante montre ce que vous obtiendriez si vous génériez du code `CIL` pour la méthode `StringsAreImmutable2()` :

```CIL
.method assembly hidebysig static void '<<Main>$>g__StringsAreImmutable2()|0,8'()
cil managed
{
    // Code size 32 (0x20)
    .maxstack 1
    .locals init (string V_0)
    IL_0000: nop
...
    IL_000c: ldstr "My other string"
    IL_0011: stloc.0
    IL_0012: ldstr "New string value"
    IL_0017: stloc.0
    IL_0018: ldloc.0
    IL_0013: nop
...
    IL_0014: ret
} // end of method Program::StringsAreImmutable2
```

Bien que vous n'ayez pas encore examiné les détails de bas niveau du `CIL`, ==notez les deux appels à l'*opcode* `ldstr` (*load string*)==. **En termes simples, l'opcode `ldstr` du `CIL` charge un nouvel objet `string` dans le tas (*heap*) géré**. L'objet chaîne précédent qui contenait la valeur `"My other string"` sera finalement supprimé par le *garbage collector*.

Que devez-vous retenir exactement de cette information ? **En résumé, la classe `string` peut être inefficace et entraîner un code trop lourd ==si elle est mal utilisée, en particulier lors de la concaténation de chaînes ou du traitement de grandes quantités de données textuelles==**. Si vous devez représenter des données de caractères de base telles que le numéro de sécurité sociale américain, les prénoms ou noms de famille, ou de simples fragments de texte utilisés dans votre application, la classe `string` est le choix idéal.

**Cependant, si vous développez une application qui utilise intensivement des données textuelles changeant fréquemment (comme un programme de traitement de texte), il serait malavisé de représenter les données de traitement de texte à l'aide d' objets `string`, car vous finirez très certainement (et souvent indirectement) par créer des copies inutiles de données chaîne**. Alors, que doit faire un programmeur ? Bonne question.

## Utiliser le type `System.Text.StringBuilder`

Étant donné que le type `string` peut s'avérer inefficace lorsqu'il est utilisé sans discernement, **les bibliothèques de classes de base .NET Core fournissent l'espace de noms `System.Text`. Cet espace de noms (relativement petit) contient une classe nommée `StringBuilder`**. À l'instar de la classe `System.String`, `StringBuilder` définit des méthodes qui vous permettent, par exemple, ==de remplacer ou de formater des segments==. Lorsque vous souhaitez utiliser ce type dans vos fichiers de code C#, la première étape consiste à vous assurer que l'espace de noms suivant est importé dans votre fichier de code:

```cs
// StringBuilder lives here
using System.Text;
```

La particularité de `StringBuilder` réside dans le fait que lorsque vous appelez des membres de ce type, **vous modifiez directement les données de caractères internes de l'objet (ce qui le rend plus efficace), sans obtenir de copie des données dans un format modifié**. Lorsque vous créez une instance de `StringBuilder`, ==vous pouvez fournir les valeurs de démarrage initiales de l'objet via l'un des nombreux constructeurs==. Si vous débutez dans le domaine des constructeurs, sachez simplement que **les constructeurs vous permettent de créer un objet avec un état initial lorsque vous appliquez le mot-clé `new`**. Considérez l'utilisation suivante de `StringBuilder` :

```cs
static void FunWithStringBuilder()
{
   Console.WriteLine("=> Using the StringBuilder: ");
   StringBuilder sb = new StringBuilder("**** Fantastic Games ****");
   sb.Append("\n");
   sb.AppendLine("Half Life");
   sb.AppendLine("The Elder Scrolls 3: Morrowind");
   sb.AppendLine("Deus Ex" + "2");
   sb.AppendLine("System Shock");
   Console.WriteLine(sb.ToString());
   sb.Replace("2", " Invisible War");
   Console.WriteLine(sb.ToString());
   Console.WriteLine($"sb has {sb.Length} chars.");
   Console.WriteLine();
}
```

Ici, j'ai construit un `StringBuilder` défini sur la valeur initiale `**** Fantastic Games ****` . Comme vous pouvez le voir, j'ajoute des éléments au tampon interne (*buffer*) et je peux remplacer ou supprimer des caractères à ma guise. **Par défaut, un `StringBuilder` ne peut contenir initialement qu'une chaîne de 16 caractères ou moins (mais s'étendra automatiquement si nécessaire)** ; ==cependant, cette valeur de départ par défaut peut être modifiée via un argument de constructeur supplémentaire==.

```cs
// Crée un StringBuilder avec une taille initiale de 256 caractères.
StringBuilder sb = new StringBuilder("**** Fantastic Games ****", 256);
```

==Si vous ajoutez plus de caractères que la limite spécifiée, l'objet `StringBuilder` copiera ses données dans une nouvelle instance et augmentera la taille du tampon de la limite spécifiée==.

# Rétrécissement et élargissement des conversions de types de données

Maintenant que vous comprenez comment utiliser les types de données intrinsèques C#, examinons le sujet connexe de la conversion des types de données. Supposons que vous ayez un nouveau projet d'application console nommé *TypeConversions* et que vous l'ayez ajouté à votre solution. Mettez à jour le code pour qu'il corresponde à ce qui suit :

```cs
Console.Title = "Fun With Type Conversions";
Console.WriteLine("**** Fun with Type Conversions ****");

// Ajoute deux shorts et affiche le résultat. 
short numb1 = 9, numb2 = 10;
Console.WriteLine($"{numb1} + {numb2} = {Add(numb1, numb2)}");

static int Add(int x, int y)
{
   return x + y;
}
```

La méthode `Add()` doit recevoir 2 `int`. Elle reçoit 2 `short` mais ne renvois pas d'erreur. La raison est parce qu'un `short` est "*contenu*" dans un `int`. Les valeur maximal et minimales d'un `short` est plus petite que celle d'un `int`. Donc il n'y a pas de perte d'information possible. Le compilateur va implicitement "*élargir*" le `short` en un `int`.

**Le terme employé pour cette opération est "*upcast*".**

En se référant au [[#Tableau 3-4 Tous les types de donnée primitifs de C|tableau 3-4]], on peut voir qu'un `short` à une taille de 16 bits alors qu'un `int` à une taille de 32 bits.

> [!Info]- Les conversions autorisés selon les types.
> Consultez les «[Tables de conversion de types](https://learn.microsoft.com/en-us/dotnet/standard/base-types/conversion-tables)» dans la documentation .NET Core si vous souhaitez connaître les conversions autorisées (élargissement et rétrécissement, abordés ci-après) pour chaque type de données C#.

Bien que cet élargissement implicite ait joué en votre faveur dans l'exemple précédent, ==cette «*fonctionnalité*» peut parfois être à l'origine d'erreurs de compilation==. **Supposons, par exemple, que vous ayez attribué à `numb1` et` numb2` des valeurs qui, une fois additionnées, dépassent la valeur maximale d'un `short`. Supposons également que vous stockiez la valeur de retour de la méthode `Add()` dans une nouvelle variable locale de type `short`, plutôt que d'imprimer directement le résultat dans la console.

```cs
Console.Title = "Fun With Type Conversions";
Console.WriteLine("**** Fun with Type Conversions ****");

// Erreur du compilateur en dessous.
short numb1 = 30_000, numb2 = 30_000;
short answer = Add(numb1, numb2);
Console.WriteLine($"{numb1} + {numb2} = {answer}");

static int Add(int x, int y)
{
   return x + y;
}
```

Dans ce cas, le compilateur renvois l'erreur suivante:

```
Cannot implicitly convert type 'int' to 'short'. An explicit conversion exists (are you
missing a cast?)
```

Le problème est: bien que la méthode `Add()` soit capable de renvoyer un entier de valeur $60\;000$ (qui s'inscrit dans la plage d'un `System.Int32`), cette valeur ne peut pas être stockée dans un `short`, car **elle dépasse les limites de ce type de données** ==Formellement parlant, le `CoreCLR` n'a pas pu appliquer d'opération de réduction==. Comme vous pouvez le deviner, ==le rétrécissement est l'opposé logique de l'élargissement, dans la mesure où une valeur plus grande est stockée dans une variable de type de données plus petit==.

**Il est important de souligner que toutes les conversions de rétrécissement entraînent une erreur de compilation, même lorsque vous pouvez raisonnablement penser que la conversion de rétrécissement devrait effectivement réussir**. Par exemple, le code suivant entraîne également une erreur de compilation :

```cs
// Une autre erreur de compilation!
static void NarrowingAttempt()
{
    byte myByte = 0;
    int myInt = 200;
    myByte = myInt;
    
    Console.WriteLine("Value of myByte: {0}", myByte);
}
```

Ici, la valeur contenue dans la variable `int` (`myInt`) se situe dans la plage d'un octet ; par conséquent, vous vous attendez à ce que l'opération de réduction n'entraîne pas d'erreur d'exécution. Cependant, **étant donné que C# est un langage conçu dans un souci de sécurité des types**(*Type Safe*)**, vous obtenez effectivement une erreur de compilation**.

==Lorsque vous souhaitez informer le compilateur que vous êtes prêt à accepter une perte de données éventuelle due à une opération de rétrécissement, vous devez appliquer un casting explicite à l'aide de l'opérateur de cast C#, `()`==. Considérez la mise à jour suivante du fichier *Program.cs* :

```cs
Console.Title = "Fun With Type Conversions";
Console.WriteLine("**** Fun with Type Conversions ****");

short numb1 = 30000,
    numb2 = 30000;

// Cast explicite d'un int vers un short (et prmet une perte de données)
short answer = (short)Add(numb1, numb2);

Console.WriteLine("{0} + {1} = {2}", numb1, numb2, answer);
NarrowingAttempt();
Console.ReadLine();

static int Add(int x, int y)
{
    return x + y;
}

static void NarrowingAttempt()
{
    byte myByte = 0;
    int myInt = 200;

    // Cast explicite d'un int vers un short (pas de perte de données)
    myByte = (byte)myInt;
    Console.WriteLine("Value of myByte: {0}", myByte);
}
```

==À ce stade, le code compile; cependant, le résultat de l'addition est complètement incorrect.==

```
**** Fun with Type Conversions ****
30000 + 30000 = -5536
Value of myByte: 200
```

Comme vous venez de le voir, un casting explicite (aussi appelé *Downcast*) vous permet de forcer le compilateur à appliquer une conversion restrictive, même si cela peut entraîner une perte de données. ==Dans le cas de la méthode `NarrowingAttempt()`, cela ne posait pas de problème, car la valeur $200$ peut s'inscrire parfaitement dans la plage d'un `byte`==. **Cependant, dans le cas de l'addition des deux shorts dans le code, le résultat final est tout à fait inacceptable ($30 000 + 30 000 = –5536 ?$).**

**Si vous développez une application dans laquelle la perte de données est toujours inacceptable, C# fournit les mots-clés `checked` et `unchecked` pour garantir que la perte de données ne passe pas inaperçue**.

## Utilisation du mot-clé `checked`

Commençons par apprendre le rôle du mot-clé `checked`. Supposons que vous ayez une nouvelle méthode dans `Program` qui tente d'ajouter deux `byte`, chacun ayant reçu une valeur inférieure à la valeur maximale ($255$). Si vous deviez ajouter les valeurs de ces types (en convertissant l'`int` renvoyé en `byte`), vous supposeriez que le résultat serait la somme exacte de chaque membre.

```cs
static void ProcessByte()
{
   byte b1 = 100, b2 = 250;
   byte sum = (byte)Add(b1, b2);

   // sum chould hold the value 350. However, we find the value 94.
   Console.WriteLine($"sum = {sum}");
}
```

**Si vous visualisez le résultat de cette application, vous serez peut-être surpris de constater que sum contient la valeur $94$ (au lieu de $350$ comme prévu). La raison est simple: Étant donné qu'un `System.Byte` ne peut contenir qu'une valeur comprise entre $0$ et $255$ (inclus, pour un total de $256$ emplacements), `sum` contient désormais la valeur de dépassement ($350 – 256 = 94$)**. ==Par défaut, si vous ne prenez aucune mesure corrective, les conditions de dépassement/sous-dépassement se produisent sans erreur==.

> [!Note] Pour une question de compréhension (Gianni)
> dépassement -> *Overflow*
> sous-dépassement -> *Underflow*

**Pour gérer les conditions d'*overflow* ou d'*underflow* dans votre application, vous avez deux options**. **La première consiste à** utiliser votre intelligence et vos compétences en programmation pour **gérer manuellement toutes les conditions d'overflow/underflow**. Bien sûr, le problème avec cette technique est le simple fait que vous êtes humain, et même vos meilleurs efforts peuvent aboutir à des erreurs qui vous ont échappé.

**Heureusement, C# fournit le mot-clé `checked`**. ==Lorsque vous encapsulez une instruction (ou un bloc d'instructions) dans la portée du mot-clé `checked`, le compilateur C# émet des instructions `CIL` supplémentaires qui testent les conditions de dépassement pouvant survenir lors de l'addition, la multiplication, la soustraction ou la division de deux types de données numériques==.

**Si un débordement s'est produit, vous recevrez une exception d'exécution : `System.OverflowException`**. Le [[Chapitre 7#Tableau 7-1 Membres principaux du type `System.Exception`|Chapitre 7]] examinera tous les détails de la gestion structurée des exceptions et l'utilisation des mots-clés `try` et `catch`. Sans trop s'attarder sur les détails à ce stade, observez la mise à jour suivante :

```cs
static void ProcessByte()
{
    byte b1 = 100,
        b2 = 250;

    // Cette fois, on dit au compilateur d'ajouter du code CIL
    // pour lever une erreur si le dépassement/sous-dépassement
    // prend lieu.
    try
    {
        byte sum = checked((byte)Add(b1, b2));
        Console.WriteLine($"sum = {sum}");
    }
    catch (OverflowException ex)
    {
        Console.WriteLine(ex.Message);
    }
}

```

==Notez que la valeur renvoyée par `Add()` a été encapsulée dans la portée du mot-clé `checked`==. Comme la somme est supérieure à un octet, cela déclenche une exception d'exécution. Notez le message d'erreur affiché via la propriété `Message`.

```
Arithmetic operation resulted in an overflow.
```

**Si vous souhaitez forcer la vérification des dépassements de capacité sur un bloc d'instructions de code, vous pouvez le faire en définissant une portée `checked` comme suit :**

```cs
try
{
	checked
	{
		byte sum = (byte)Add(b1, b2);
		Console.WriteLine("sum = {0}", sum);
	}
}
catch (OverflowException ex)
{
	Console.WriteLine(ex.Message);
}
```

Dans les deux cas, le code en question sera automatiquement évalué pour détecter d'éventuelles conditions de débordement, ce qui déclenchera une exception de débordement si tel est le cas.

## Définition de la vérification des dépassements de capacité à l'échelle du projet (fichier  *.csproj* )

Si vous créez une application qui ne doit jamais permettre de dépassement de capacité silencieux, vous pourriez vous retrouver dans la situation fastidieuse de devoir encapsuler de nombreuses lignes de code dans la portée du mot-clé `checked`. **Comme alternative, le compilateur C# prend en charge l'indicateur `-checked`. Lorsqu'il est activé, toutes vos opérations arithmétiques seront évaluées pour détecter les débordements sans qu'il soit nécessaire d'utiliser le mot-clé `checked` de C#.** Si un débordement est détecté, vous recevrez toujours une exception d'exécution. Pour définir cela pour l'ensemble du projet, entrez ce qui suit dans le fichier de projet :

```xml
  <PropertyGroup>
    <CheckForOverflowUnderFlow>true</CheckForOverflowUnderFlow>
  </PropertyGroup>
```

## Utilisation du mot-clé `unchecked`

Maintenant, en supposant que vous ayez activé ce paramètre à l'échelle du projet, que devez-vous faire si vous avez un bloc de code où la perte de données est acceptable ? Étant donné que l'indicateur `checked` évalue toute la logique arithmétique, **C# fournit le mot-clé `unchecked` pour désactiver le déclenchement d'une exception d'overflow au cas par cas**. L'utilisation de ce mot-clé est identique à celle du mot-clé `checked`, en ce sens que vous pouvez spécifier une seule instruction ou un bloc d'instructions.

```cs
static void ProcessByte()
{
    byte b1 = 100;
    byte b2 = 250;
    
    // En supposant que checked est activé au niveau du projet,
    // ce block ne déclenchera pas
    // une exception lors de l'exécution.
    unchecked
    {
        byte sum = (byte)(b1 + b2);
        Console.WriteLine("sum = {0} ", sum);
    }
}
```

Pour résumer les mots-clés `checked` et `unchecked` en C#, ==rappelez-vous que le comportement par défaut du runtime .NET Core est d'ignorer les dépassements arithmétiques==. **Lorsque vous souhaitez gérer de manière sélective des instructions discrètes, utilisez le mot-clé `checked`. Si vous souhaitez intercepter les erreurs de dépassement dans toute votre application, activez l'indicateur `/checked`. Enfin, le mot-clé `unchecked` peut être utilisé si vous avez un bloc de code où l'overflow est acceptable (et ne doit donc pas déclencher d'exception d'exécution)**.

>[!Note]
>Dans .NET CLI, il n'est plus possible d'appeler le compilateur de de lui envoyé l'option `/checked`. Avec *MSBuild* (`dotnet build`), l'équivalent serait ceci:
>```bash
>dotnet build -p:CheckForOverflowUnderflow=true
>```
>
>Il est donc plus pratique d'utilisé le fichier project pour effectué cette opération.

# Comprendre les variables locales implicitement typées

Jusqu'à présent dans ce chapitre, lorsque vous avez défini des variables locales, vous avez explicitement spécifié le type de données sous-jacent de chaque variable déclarée.


```cs
static void DelcareExplicitVars()
{
   // Les variables locales explicitement typées
   // sont déclarées comme suit:
   // typeDonnée nomVariable = valeurInitiale

   int myInt = 0;
   bool myBool = true;
   string myString = "Time, marches on...";
}
```

Bien que beaucoup diront qu'il est généralement recommandé de spécifier explicitement le type de données de chaque variable, **le langage C# permet de *typer implicitement* les variables locales à l'aide du mot-clé `var`**. Le mot-clé `var` peut être utilisé à la place de la spécification d'un type de données particulier (tel que `int`, `bool` ou `string`). **Lorsque vous procédez ainsi, le compilateur déduit automatiquement le type de données sous-jacent en fonction de la valeur initiale utilisée pour initialiser le point de données local**.

Pour illustrer le rôle du typage implicite, créez un nouveau projet d'application console nommé *ImplicitlyTypedLocalVars* et ajoutez-le à votre solution. Mettez à jour le code dans `Program.cs` comme suit :

```cs
Console.Title = "Fun with Implicit Typing";
Console.WriteLine("**** Fun with Implicit Typing ****");

DeclareImplicitVars();

static void DeclareImplicitVars()
{
    // Les variables locales implicitement typées
    // sont déclarées comme suit:
    // var nomVariable = valeurInitiale
    var myInt = 0;
    var myBool = true;
    var myString = "Time, marches on...";
}
```

>[!Info]- La sémantique derrière `var`
>À proprement parler, `var` n'est pas un mot-clé C#. Il est possible de déclarer des variables, des paramètres et des champs nommés `var` sans erreur de compilation. Cependant, lorsque le token `var` est utilisé comme type de données, il est traité contextuellement comme un mot-clé par le compilateur.

**dans ce cas, le compilateur est capable de déduire, à partir de la valeur initialement attribuée, que `myint` est en fait un `system.int32`, `mybool` un `system.boolean` et `mystring` est bien un `system.string`.** ==vous pouvez le vérifier en affichant le nom du type via la réflexion==. comme vous le verrez plus en détail au [[Chapitre 17|chapitre 17]], ==la réflexion consiste à déterminer la composition d'un type lors de l'exécution==. Par exemple, à l'aide de la réflexion, vous pouvez déterminer le type de données d'une variable locale implicitement typée. Mettez à jour votre méthode avec les instructions de code suivantes:

```cs
static void DeclareImplicitVars()
{
    // Les variables locales implicitement typées
    // sont déclarées comme suit:
    // var nomVariable = valeurInitiale
    var myInt = 0;
    var myBool = true;
    var myString = "Time, marches on...";

    // Affiche le type sous-jacent
    Console.WriteLine($"myInt is a: {myInt.GetType().Name}");
    Console.WriteLine($"myBool is a: {myBool.GetType().Name}");
    Console.WriteLine($"myString is a: {myString.GetType().Name}");
}
```

>Sachez que vous pouvez utiliser ce typage implicite pour n'importe quel type, y compris les tableaux, les types génériques (voir [[Chapitre 10#Premier aperçu des collections génériques `<T>`|Chapitre 10]]) et vos propres types personnalisés. Vous verrez d'autres exemples de typage implicite au cours de ce livre.

En déclarant la méthode `DeclareImplicitVars()`, vous trouvez comme sortie ceci:

```
**** Fun with Implicit Typing ****
myInt is a: Int32
myBool is a: Boolean
myString is a: String
```

## Déclaration implicite des nombres

Comme indiqué précédemment, les nombres entiers sont par défaut des `int`, et les nombres à virgule flottante sont par défaut des `double`. Créez une nouvelle méthode nommée `DeclareImplicitNumerics()` et ajoutez le code suivant pour illustrer la déclaration implicite des nombres :

```cs
static void DeclaraImplicitNumerics()
{
   // Variables numériques implicitement typées.
   var myUInt = 0u;
   var myInt = 0;
   var myLong = 0L;
   var myDouble = 0.5;
   var myFloat = 0.5F;
   var myDecimal = 0.5M;

   // Affiche le type sous-jacent.
   Console.WriteLine("myUInt is a: {0}", myUInt.GetType().Name);
   Console.WriteLine("myInt is a: {0}", myInt.GetType().Name);
   Console.WriteLine("myLong is a: {0}", myLong.GetType().Name);
   Console.WriteLine("myDouble is a: {0}", myDouble.GetType().Name);
   Console.WriteLine("myFloat is a: {0}", myFloat.GetType().Name);
   Console.WriteLine("myDecimal is a: {0}", myDecimal.GetType().Name);
}
```

## Comprendre les restrictions relatives aux variables de type implicite

**Il existe diverses restrictions concernant l'utilisation du mot-clé `var`**. ==Tout d'abord, le typage implicite s'applique uniquement aux variables locales dans une méthode ou une propriété==. **Il est illégal d'utiliser le mot-clé `var` pour définir des valeurs de retour, des paramètres ou des données de champ d'un type personnalisé.** Par exemple, la définition de classe suivante entraînera diverses erreurs de compilation :

```cs
class ThisWillNeverCompile
{
	// Erreur! var ne peut pas être utilisé comme type de champ.
	private var myInt = 10;

  // Erreur! var ne peut pas être utilisé comme type de retour
	// ou type de paramètre!.
	public var MyMethod(var x, var y){}
}
```

**De plus, les variables locales déclarées avec le mot-clé `var` *doivent* se voir attribuer une valeur initiale au moment exact de leur déclaration et *ne peuvent pas* se voir attribuer la valeur initiale `null`.** Cette dernière restriction est logique, étant donné que le compilateur ne peut pas déduire le type de mémoire vers lequel la variable pointerait en se basant uniquement sur la valeur `null`.

```cs
  // Erreur! Une valeur doit lui être assignée.
  var myData;

  // Erreur! Une valeur doit lui être assignée au moment exacte de la déclaration!
  var myInt;
  myInt = 0;

// Erreur! Ne peut pas assigné null comme valeur initiale. 
  var myObj = null;
```

**Il est toutefois permis d'assigner une variable locale déduite à `null` après son assignation initiale (à condition qu'il s'agisse d'un type de référence).**

```cs
// Ok si SportCar est un type de référence.
var myCar = new SportCar();
myCAr = null

```

De plus, il est permis d'assigner la valeur d'une variable typée implicitement à la valeur d'une autre variable, implicitement ou non.

```cs
// Aussi OK!
var myINt = 0;
var anotherInt = myInt;

string myString = "Wake up!";
var myData = myString;
```

**Il est également permis de renvoyer une variable locale implicitement typée à l'appelant, à condition que le type de retour de la méthode soit le même type sous-jacent que le point de données défini par `var`**.

```cs
static int GetAnInt()
{
	var retVal = 9;
	return retVal;
}
```

## Les données implicitement typées sont des données fortement typées 

Sachez que le typage implicite des variables locales donne lieu à des *données fortement typées*. Par conséquent, **l'utilisation du mot-clé `var` *n'est pas* la même technique que celle utilisée avec les langages de script** (tels que JavaScript ou Perl) ou le type de données `COM Variant`, où une variable peut contenir des valeurs de différents types au cours de son existence dans un programme (souvent appelé typage dynamique (*dynamic typing*)).

>[!note] C# autorise le typage dynamique à l'aide d'un mot-clé appelé `dynamic`. Vous découvrirez cet aspect du langage au [[Chapitre 17|Chapitre 17]].

Au contraire, l'inférence de types **conserve l'aspect fortement typé du langage C#** et n'affecte que la déclaration des variables au moment de la compilation. Après cela, **le point de données est traité comme s'il avait été déclaré avec ce type ; l'attribution d'une valeur d'un type différent à cette variable entraînera une erreur de compilation**.

```cs
static void ImplicitTypingIsStrongTyping()
{
    // Le compilateur sait que "s" est un System.String.
    var s = "This variable can only hold string data!";
    s = "This is fine...";
    
    // Peut invoquer n'importe quel membre du type sous-jacent.
    string upper = s.ToUpper();
    
    // Erreur! Ne peut pas assiger des données numériques à une chaînes.
    s = 44;
}
```

## Comprendre l'utilité des variables locales implicitement typées

Maintenant que vous connaissez la syntaxe utilisée pour déclarer des variables locales implicitement typées, vous vous demandez certainement quand utiliser cette construction. ==Tout d'abord, utiliser `var` pour déclarer des variables locales simplement pour le plaisir de le faire n'apporte pas grand-chose==. ==Cela peut prêter à confusion pour les autres personnes qui lisent votre code, car il devient plus difficile de déterminer rapidement le type de données sous-jacent et, par conséquent, de comprendre la fonctionnalité globale de la variable==. **Donc, si vous savez que vous avez besoin d'un `int`, déclarez un `int`!**

Cependant, comme vous le verrez à partir du [[Chapitre 13|Chapitre 13]], **la technologie `LINQ` utilise des *expressions de requêtes* qui peuvent produire des ensembles de résultats créés dynamiquement en fonction du format de la requête elle-même.** ==Dans ces cas, le typage implicite est extrêmement utile, car vous n'avez pas besoin de définir explicitement le type qu'une requête peut renvoyer, ce qui serait littéralement impossible à faire dans certains cas==. Sans vous attarder sur l'exemple de code `LINQ` suivant, voyez si vous pouvez déterminer le type de données sous-jacent de `subset` :

```cs
static void LinqQuerryOverInts()
{
    int[] numbers = { 10, 20, 30, 40, 1, 2, 3, 8 };

    // Requête LINQ!
    var subset = from i in numbers where i < 10 select i;

    Console.Write("Values in subset: ");
    foreach (var i in subset)
    {
        Console.Write("{0}", i);
    }
    Console.WriteLine();

    // Hmm...de quel type est subset?
    Console.WriteLine($"subset is a: {subset.GetType().Name}");
    Console.WriteLine($"subset is defined in: {subset.GetType().Name}");
    Console.WriteLine();
}

```

Vous pensez peut-être que le type de données sous-ensemble est un tableau d'entiers `int[]`. Cela semble être le cas, mais **en réalité, il s'agit d'un type de données `LINQ` de bas niveau que vous ne connaîtriez jamais si vous n'utilisez pas `LINQ` depuis longtemps ou si vous n'ouvrez pas l'image compilée dans *ildasm.exe***. ==La bonne nouvelle, c'est que lorsque vous utilisez `LINQ`, vous vous souciez rarement (voire jamais) du type sous-jacent de la valeur de retour de la requête ; vous attribuez simplement la valeur à une variable locale implicitement typée==.

**En fait, on pourrait dire que le seul cas où vous utiliseriez le mot-clé `var` serait pour définir les données renvoyées par une requête `LINQ`**. ==N'oubliez pas : si vous savez que vous avez besoin d'un int, déclarez simplement un int ! La plupart des développeurs considèrent que la sur-utilisation du typage implicite (via le mot-clé var) est un mauvais style dans le code de production==.

# Utilisation des constructions d'itération en C#

Tous les langages de programmation fournissent des moyens de répéter des blocs de code jusqu'à ce qu'une condition de fin soit remplie. Quel que soit le langage que vous avez utilisé par le passé, je pense que les instructions d'itération C# ne devraient pas poser trop de problèmes et ne nécessitent que peu d'explications. **C# fournit les quatre constructions d'itération suivantes :**

- boucle `for`
- boucle `foreach`/`in`
- boucle `while`
- boucle `do`/`while`

Examinons rapidement chaque construction de boucle à tour de rôle, à l'aide d'un nouveau projet d'application console nommé *IterationsAndDecisions*.

> [!Note]
> Je vais rester bref et concis dans cette section du chapitre, car je pars du principe que vous avez déjà utilisé des mots-clés similaires (`if`, `for`, `switch`, etc.) dans votre langage de programmation actuel. 
>>[!example] La documentation
>>Si vous avez besoin de plus d'informations, consultez les rubriques [Instructions d'itération (référence C#)](https://learn.microsoft.com/fr-fr/dotnet/csharp/language-reference/statements/iteration-statements), [Instructions de saut (référence C#)](https://learn.microsoft.com/fr-fr/dotnet/csharp/language-reference/statements/jump-statements) et [Instructions de sélection (référence C#)](https://learn.microsoft.com/fr-fr/dotnet/csharp/language-reference/statements/selection-statements)  dans la documentation C#.

## Utilisation de la boucle `for`

Lorsque vous devez itérer un bloc de code un nombre fixe de fois, l'instruction `for` offre une grande flexibilité. En substance, vous pouvez spécifier le nombre de fois qu'un bloc de code doit se répéter, ainsi que la condition de fin. Sans trop insister sur ce point, voici un exemple de syntaxe :

```cs
// Une boucle for basique
static void ForLoopExample()
{
	// Note! "i" est seulement visible dans la portée de la boucle for.
	for(int i = 0; i < 4; i++)
	{
		Console.WriteLine("Number is: {0} ", i);
	}
	// "i" n'est pas visible ici.
}
```

Toutes vos anciennes astuces C, C++ et Java restent valables lorsque vous créez une instruction `for` en C#. Vous pouvez créer des conditions de fin complexes, créer des boucles infinies, boucler en sens inverse (via l'opérateur `--`) et utiliser les mots-clés de saut `goto`, `continue` et `break`.

## Utilisation de la boucle `foreach`

Le mot-clé `foreach` de C# vous permet d'itérer sur tous les éléments d'un conteneur ==sans avoir à tester une limite supérieure==. Contrairement à une boucle `for`, cependant, **la boucle foreach parcourt le conteneur uniquement de manière linéaire** ($n+1$) (vous ne pouvez donc pas revenir en arrière dans le conteneur, sauter un élément sur trois, etc.).

Cependant, lorsque vous avez simplement besoin de parcourir une collection élément par élément, la boucle `foreach` est le choix idéal. ==Voici deux exemples utilisant `foreach` : l'un pour parcourir un tableau de chaînes de caractères et l'autre pour parcourir un tableau d'entiers==. Notez que le type de données avant le mot-clé `in` représente le type de données dans le conteneur.

```cs
// Parcourir les éléments du tableau à l'aide de foreach.
static void ForEachLoopExample()
{
    string[] carTypes = { "Ford", "BMW", "Yugo", "Honda" };
    foreach (string c in carTypes)
    {
        Console.WriteLine(c);
    }
	
    int[] myInts = { 10, 20, 30, 40 };
    foreach (int i in myInts)
    {
        Console.WriteLine(i);
    }
}
```

**L'élément après le mot-clé `in` peut être un tableau simple (comme ici) ou, plus précisément, toute classe implémentant l'interface `IEnumerable`**. Comme vous le verrez au [[Chapitre 10#L'espace de noms `System.Collections.Generic`|Chapitre 10]], les bibliothèques de classes de base .NET Core fournissent un certain nombre de collections contenant des implémentations de types de données abstraits (*ADT*) courants. **N'importe lequel de ces éléments (tel que le générique `List<T>`) peut être utilisé dans une boucle `foreach`.**

## Utilisation du typage implicite dans les constructions `foreach`

Il est également possible d'utiliser le typage implicite dans une construction de boucle `foreach`. Comme on peut s'y attendre, **le compilateur déduira correctement le « type de type » approprié**. Rappelez-vous l'exemple de la méthode `LINQ` présenté plus haut dans ce chapitre. **Étant donné que vous ne connaissez pas le type de données sous-jacent exact de la variable `subset`, vous pouvez itérer sur le jeu de résultats à l'aide du typage implicite**.

```cs
static void LinqQueryOverInts()
{
    int[] numbers = { 10, 20, 30, 40, 1, 2, 3, 8 };
	
    // Requête LINQ!
    var subset = from i in numbers where i < 10 select i;
    Console.Write("Values in subset: ");
	
    foreach (var i in subset)
    {
        Console.Write("{0} ", i);
    }
}
```

## Utilisation des structures de boucle `while` et `do`/`while` 

**La boucle `while` est utile si vous souhaitez exécuter un bloc d'instructions jusqu'à ce qu'une condition de fin soit atteinte.** Dans le cadre d'une boucle `while`, ==vous devez vous assurer que cet événement de fin est bien défini, sinon vous serez pris dans une boucle sans fin==. Dans l'exemple suivant, le message `In while loop` sera affiché en continu jusqu'à ce que l'utilisateur mette fin à la boucle en entrant `yes` à l'invite de commande :

```cs
static void WhileLoopExample()
{
    string userIsDone = "";
	
    // Test sur une copie en minuscule de la chaîne.
    while (userIsDone.ToLower() != "yes")
    {
        Console.WriteLine("In while loop");
        Console.Write("Are you done? [yes] [no]: ");
        userIsDone = Console.ReadLine();
    }
}
```

La boucle `do/while` est étroitement liée à la boucle `while`. ==Tout comme une boucle `while` simple, `do/while` est utilisée lorsque vous devez effectuer une action un nombre indéterminé de fois==. **La différence réside dans le fait que les boucles `do/while` garantissent l'exécution du bloc de code correspondant au moins une fois**. ==En revanche, il est possible qu'une boucle `while` simple ne s'exécute jamais si la condition de fin est fausse dès le départ==.

```cs
static void DoWhileLopExample()
{
    string userIsDone = "";
	
    do
    {
        Console.WriteLine("In do/while loop");
        Console.Write("Are you done? [yes] [no]: ");
        userIsDone = Console.ReadLine();
    } while (userIsDone.ToLower() != "yes"); // Notez le point-virgule!
}
```

# Une brève discussion sur la portée

Comme tous les langages basés sur le C (C#, Java, etc.), **une portée est créée à l'aide d'accolades**. Vous l'avez déjà vu dans de nombreux exemples jusqu'à présent, notamment dans les espaces de noms, les classes et les méthodes. **Les constructions d'itération et de décision fonctionnent également dans une portée,** comme dans l'exemple suivant :

```cs
for(int i = 0; i < 4; i++)
{
	Console.WriteLine("Number is {0}", i);
}
```

**Pour ces constructions** (à la fois dans la section précédente et dans la section suivante), **il est permis de ne pas utiliser d'accolades. En d'autres termes, le code suivant est exactement le même que l'exemple précédent** :

```cs
for(int i = 0; i < 4; i++)
	Console.WriteLine("Number is {0}", i);
```

**Bien que cela soit autorisé, ce n'est généralement pas une bonne idée**. ==Le problème ne vient pas de l'instruction à une ligne, mais de l'instruction qui s'étend sur plusieurs lignes==. **Sans les accolades, des erreurs pourraient être commises lors de l'extension du code dans les constructions d'itération/décision**. Par exemple, les deux exemples suivants ne sont pas identiques :

```cs
for(int i = 0; i < 4; i++)
{
	Console.WriteLine("Number is: {0} ", i);
	Console.WriteLine("Number plus 1 is: {0} ", i+1)
}
for(int i = 0; i < 4; i++)
	Console.WriteLine("Number is: {0} ", i);
	Console.WriteLine("Number plus 1 is: {0} ", i+1)
```

Si vous avez de la chance (comme dans cet exemple), ==la ligne de code supplémentaire génère une erreur de compilation, car la variable `i` n'est définie que dans la portée de la boucle `for`==. **Si vous n'avez pas de chance, vous exécutez du code qui n'est pas signalé comme une erreur de compilation, mais qui est une erreur logique, plus difficile à trouver et à débuguer.**

# Utilisation des constructions décisionnelles et des opérateurs relationnels/d'égalité

Maintenant que vous savez itérer sur un bloc d'instructions, le concept suivant consiste à contrôler le flux d'exécution du programme. **C# définit deux constructions simples pour modifier le flux de votre programme, en fonction de diverses éventualités** :

- La déclaration `if/else`
- La déclaration `switch`

>[!Note]- préambule: 
>C# 7 étend l'instructions `is`  et la déclaration `switch` grâce à une technique appelée  *pattern matching*  (correspondance de motifs). Les principes de base de l'impact de ces extensions sur les instructions `if/else` et `switch` sont présentés ici à des fins d'exhaustivité. Ces extensions seront plus faciles à comprendre après avoir lu le [[Chapitre 6#Comprendre les règles de conversion de classes de base et dérivées|Chapitre 6]], qui traite des règles relatives aux classes de base/classes dérivées, au casting et à l'opérateur standard `is`.

## Utilisation de l'instruction `if/else`

==Tout d'abord, l'instruction `if/else`. Contrairement à C et C++, l'instruction `if/else` en C# ne fonctionne qu'avec des expressions booléennes, et non avec des valeurs arbitraires telles que -1 ou 0==.

## Utilisation des opérateurs d'égalité et relationnels

Les instructions `if/else` en C# impliquent généralement l'utilisation des opérateurs C# présentés dans le [[#Tableau 3-8 les opérateurs relationnels et d'égalité|Tableau 3-8]] pour obtenir une valeur booléenne littérale.

##### Tableau 3-8: les opérateurs relationnels et d'égalité

| Opérateur | Example             | Description                                                                                             |
| --------- | ------------------- | ------------------------------------------------------------------------------------------------------- |
| `==`      | `if(age == 30`      | Renvois `true` seulement si les deux expressions sont les mêmes                                         |
| `!=`      | `if("Foo != myStr)` | Renvois `true` seulement si les deux expressions sont différentes.                                      |
| `<`       | `if(bonus < 2000)`  | Renvois `true` seulement si l'expression A (`bonus`) est plus petite que l'expression B (`2000`)        |
| `>`       | `if(bonus > 2000)`  | Renvois `true` seulement si l'expression A (`bonus`) est plus grande que l'expression B (`2000`)        |
| `<=`      | `if(bonus <= 2000)` | Renvois `true` seulement si l'expression A (`bonus`) est plus petite ou égale à l'expression B (`2000`) |
| `>=`      | `if(bonus >= 2000)` | Renvois `true` seulement si l'expression A (`bonus`) est plus grande ou égale à l'expression B (`2000`) |

==Une fois encore, les programmeurs C et C++ doivent savoir que les anciennes astuces consistant à tester une condition pour une valeur différente de zéro ne fonctionnent pas en C#==. Supposons que vous souhaitiez vérifier si la chaîne sur laquelle vous travaillez comporte plus de zéro caractère. Vous pourriez être tenté d'écrire ceci :

```cs
static void IfElseExample()
{
    // Ceci est illégal, étant donné que Length retourne un int,
    // pas un bool.
    string stringData = "My textual data";
    if (stringData.Length)
    {
        Console.WriteLine("string is greater than 0 characters");
    }
    else
    {
        Console.WriteLine("string is not greater than 0 characters");
    }
    Console.WriteLine();
}
```

Si vous voulez utilisez la propriété `String.Length` pour déterminer la véracité ou la fausseté, vous devrez modifier votre expression conditionnelle pour aboutir à un *Booléen*

```cs
// Légale, du fait que la résolution est soit vrai ou faux
if (stringData.Length > 0)
{
    Console.WriteLine("string is greater than 0 characters");
}
```

## Utilisation de `if/else` avec la correspondance de motifs (Nouveauté C# 7.0)

==Nouveauté dans C# 7.0, la *correspondance de motifs* est autorisée dans les instructions `if/else`.== **La correspondance de motifs permet au code d'inspecter un objet à la recherche de certains traits et propriétés et de prendre des décisions en fonction de l'existence (ou non) de ces propriétés et traits**. Ne vous inquiétez pas si vous débutez dans la programmation orientée objet ; la phrase précédente sera expliquée en détail dans les chapitres suivants. Sachez simplement (pour l'instant) que **vous pouvez vérifier le type d'un objet à l'aide du mot-clé `is`, attribuer cet objet à une variable si le motif correspond, puis utiliser cette variable**.

La méthode `IfElsePatternMatching` examine deux variables d'objet et détermine si elles sont de type `string` ou `int`, puis affiche les résultats dans la console :

```cs
static void IfElsePatternMatching()
{
   Console.WriteLine("=== If Else Pattern Matching ===");
   object testItem1 = 123;
   object testItem2 = "Hello";
   if (testItem1 is string myStringValue1)
   {
      Console.WriteLine($"{myStringValue1} is a string");
   }
   else if (testItem1 is int myValue1)
   {
      Console.WriteLine($"{myValue1} is an int");
   }
   if (testItem2 is string myStringValue2)
   {
      Console.WriteLine($"{myStringValue2} is a string");
   }
   else if (testItem2 is int myValue2)
   {
      Console.WriteLine($"{myValue2} is an int");
   }
   Console.WriteLine();
}
```

>[!tip] Petite précision
>Dans le code précédent, les variables situé après le mot clé `is` ==ne sont crée seulement si la correspondance de motif réussit==, de la même manière que pour une variable de sortie d'une méthode `TryParse`, vu [[#Utilisation de `TryParse` pour convertir les valeurs à partir de données de type chaîne|précédement]].

## Améliorations apportées à la correspondance de motifs (C# 9.0)

C# 9.0 a introduit toute une série d'améliorations à la correspondance de motifs, comme le montre le [[#Tableau 3-9 Améliorations de la correspondance de motifs|Tableau 3-9]].

##### Tableau 3-9: Améliorations de la correspondance de motifs

| Motif                         | Description                                                                                                     |
| ----------------------------- | --------------------------------------------------------------------------------------------------------------- |
| `Type pattern`                | Vérifie si une variable est un type                                                                             |
| `Parenthesized pattern`       | Applique ou souligne la priorité des combinaisons de motifs                                                     |
| `Conjuctive` (and) `pattern`  | Nécesite que les deux modèles corresponde                                                                       |
| `Disjunctive` (or) `patterns` | Nécessite que l'un ou l'autre des modèles corresponde                                                           |
| `Negated` (not) `patterns`    | Nécessite un motif qui ne correspond pas                                                                        |
| `Relational patterns`         | Nécessite que la valeur saisie soit inférieure à, inférieure ou égale à, supérieure à, ou supérieure ou égale à |
| `Pattern combinator`          | Permet d'utiliser plusieurs motifs ensemble.                                                                    |

La fonction `IfElsePatternMatchingUpdatedInCSharp9()` mise à jour montre ces nouveaux modèles en action :

```cs
static void IfElsePatternMatchingUpdatedInCSharp9()
{
    Console.WriteLine("======= C#9 If Else Pattern Matching Improvements =======");
    object testItem1 = 123;
    Type t = typeof(string);
    char c = 'f';

    //Type patterns
    if (t is Type)
    {
        Console.WriteLine($"{nameof(t)}: {t} is a Type");
    }

    // Relational, Conjuctive and Disjunctive patterns
    if (c is >= 'a' and <= 'z' or >= 'A' and <= 'Z')
    {
        Console.WriteLine($"{nameof(c)}: {c} is a character");
    }

    // Parenthisized patterns
    if (c is (>= 'a' and <= 'z') or (>= 'A' and <= 'Z') or '.' or ',')
    {
        Console.WriteLine($"{nameof(c)}: {c} is a character or separator");
    }

    // Negative patterns
    if (testItem1 is not string)
    {
        Console.WriteLine($"{nameof(testItem1)}: {testItem1} is not a string");
    }
    if (testItem1 is not null)
    {
        Console.WriteLine($"{nameof(testItem1)}: {testItem1} is not null");
    }

	 Console.WriteLine();
}
```

## Utilisation de l'opérateur conditionnel (MaJ C# 7.2 et 9.0)

L'opérateur conditionnel (`?:`), aussi appelé ***opérateur ternaire***, est ==une méthode abrégée pour écrire une instruction `if/else` simple==. La syntaxe fonctionne comme suit:

```
condition ? première_expression : seconde_expression;
```

La condition est le test conditionnel (la partie `if` de l'instruction `if/else`). **Si le test est réussi, le code situé immédiatement après le point d'interrogation (`?`) est exécuté. Si le test n'est pas évalué comme vrai, le code après les deux points (la partie else de l'instruction `if/else`) est exécuté.** L'exemple de code précédent peut être écrit à l'aide de l'opérateur conditionnel comme suit :

```cs
static void ExecuteIfElseUsingConditionalOperator()
{
   string stringData = "My textual data";
   Console.WriteLine(stringData.Length > 0
   ? "string is greater than 0 characters"
   : "string is not greater than 0 characters");
   Console.WriteLine();
}
```

**Il existe certaines restrictions concernant l'opérateur conditionnel.** Tout d'abord, ==les deux types `première_expression` et `seconde_expression` doivent pouvoir être convertis implicitement l'un vers l'autre== ou, *nouveauté dans C# 9.0*, ==chacun doit pouvoir être converti implicitement vers un type cible==. Deuxièmement, ==l'opérateur conditionnel ne peut être utilisé que dans les instructions d'affectation==. 

>[!info] Le code suivant entraînera cette erreur de compilation:
>```
>Seules les expressions d'affectation, d'appel, d' incrémentation, de décrémentation et de nouvel objet peuvent être utilisées comme instruction
>```

```cs
stringData.Length > 0
	? Console.WriteLine("string is greater than 0 characters);
	: Console.WriteLine("string is not greater than 0 characters);
```

==Nouveauté dans C# 7.2, l'opérateur conditionnel peut être utilisé pour renvoyer une référence au résultat de la condition==. Prenons l'exemple suivant, qui utilise deux formes de l'opérateur conditionnel par référence :

```cs
static void ConditionalRefExample()
{
    Console.WriteLine("======= Conditional Ref Example =======");
    var smallArray = new int[] { 1, 2, 3, 4, 5 };
    var largeArray = new int[] { 10, 20, 30, 40, 50 };
	
    int index = 7;
    ref int refValue = ref (index < 5 ? ref smallArray[index] : ref largeArray[index - 5]);
	
    refValue = 0;
    index = 2;
    
    // Le même que la ligne précédente mais sans l'utilisation de la variable refValue.
    (index < 5 ? ref smallArray[index] : ref largeArray[index - 5]) = 100;
	
    Console.WriteLine(string.Join(" ", smallArray));
    Console.WriteLine(string.Join(" ", largeArray));
}
```

Si vous n'êtes pas familier avec le mot-clé `ref`, ne vous inquiétez pas trop pour l'instant, car il sera abordé en détail dans le chapitre suivant. En résumé, le premier exemple renvoie une référence à l'emplacement du tableau vérifié avec la condition et attribue la variable `refValue` à cette référence. **Considérez la référence de manière conceptuelle comme un point vers l'emplacement dans le tableau et non comme la valeur réelle de la position du tableau. Cela permet de modifier directement la valeur du tableau à cette position en modifiant la valeur attribuée à la variable.** Le résultat de la définition de la valeur de la variable `refValue` à `0` modifie les valeurs du deuxième tableau en `10,20,0,40,50`. Le deuxième exemple met à jour la deuxième valeur du premier tableau à `100`, ce qui donne `1,2,100,4,5`.

>[!tip] Le mot clé `ref`, qui sera abordé plus en profondeur dans le [[Chapitre 4#Utilisation du modificateur `ref`|Chapitre 4]] est l'équivalent d'un *pointeur* en C, avec la différence qu'en C#, tout est automatique (grâce en partie au *garbage collector*).

## Utilisation des opérateurs logiques

Une instruction `if` peut également être composée d'expressions complexes et contenir des instructions `else` pour effectuer des tests plus complexes. La syntaxe est identique à celle du C (et du C++) et du Java. Pour créer des expressions complexes, C# propose un ensemble d'opérateurs logiques, comme indiqué dans le [[#Tableau 3-10 Les opérateurs logiques C|Tableau 3-10]].

##### Tableau 3-10: Les opérateurs logiques C# 

| Opérateur | Example                            | Description                                                                       |
| --------- | ---------------------------------- | --------------------------------------------------------------------------------- |
| &&        | if (age == 30 && name == "Fred")   | Opérateur `AND` (*ET*). Retourne `true` si toute les expressions sont vraie.      |
| \| \|     | if (age == 30 \|\| name == "Fred") | Opérateur `OR` (*OU*). Retourne `true` si au moins une des expressions est vraie. |
| `!        | if (!myBool)                       | Opérateur `NOT` (*NON*). Retourne `true` si faux ou `false` si vrai.              |

>[!tip]- les équivalent à un caractère (`&` et `|`)
>Les opérateurs `&&` et `||` provoquent tous deux un « court-circuit » lorsque cela est nécessaire. Cela signifie qu'une ==fois qu'une expression complexe a été jugée fausse, les sous-expressions restantes ne seront pas vérifiées==. **Si vous souhaitez que toutes les expressions soient testées, vous pouvez utiliser les opérateurs `&` et `|` associés.**
>> Cela veut dire que, par exemple:
>> ```
>> if (condition1 & condition2)
>> ```
>> Ce code vérifiera les deux conditions, même si la première est `false`.

## Utilisation de l'instruction `switch`

L'autre structure de sélection simple offerte par C# est l'instruction `switch`. Comme dans d'autres langages basés sur C, **l'instruction `switch` vous permet de gérer le flux du programme en fonction d'un ensemble prédéfini de choix.** Par exemple, la logique suivante affiche un message spécifique en fonction de l'une des deux sélections possibles (**le cas `default` gère une sélection non valide**) :

```cs
// Switch sur une valeur numérique.
static void SwitchExample()
{
   Console.WriteLine("======= Switch Example =======");
   Console.WriteLine("1 [C#], 2 [VB]");
   Console.Write("Please pick your language preference: ");

   string langChoice = Console.ReadLine();
   int n = int.Parse(langChoice);

   switch (n)
   {
      case 1:
         Console.WriteLine("Good choice, C# is a fine language.");
         break;
      case 2:
         Console.WriteLine("VB: OOP, multithreading, and more!");
         break;
      default:
         Console.WriteLine("Well...good luck with that!");
         break;
   }
}
```

>[!Attention]
>C# exige que **chaque cas (y compris le cas `default`) contenant des instructions exécutables soit terminé d'un `return`,  `break` ou d'un `goto` afin d'éviter de passer à l'instruction suivante**.

Une fonctionnalité intéressante de l'instruction `switch` en C# est qu'elle permet **d'évaluer des données de type chaîne en plus des données numériques.** En fait, ==toutes les versions de C# peuvent évaluer les types de données `char`, `string`, `bool`, `int`, `long` et `enum`==. Comme vous le verrez dans la section suivante, ==C# 7 ajoute des fonctionnalités supplémentaires==. Voici une instruction `switch` mise à jour qui évalue une variable de type `string` :

```cs
static void SwitchOnStringExample()
{
   Console.WriteLine("======= Switch On String Example =======");

   Console.WriteLine("C# or VB");
   Console.Write("Please pick your language preference");

   string langChoice = Console.ReadLine();
   switch (langChoice.ToUpper())
   {
      case "C#":
         Console.WriteLine("Good choice, C# is a fine language.");
         break;
      case "VB":
         Console.WriteLine("VB: OOP, multithreading and more!");
         break;
      default:
         Console.WriteLine("Well...good luck with that!");
         break;
   }
}
```

**Il est également possible d'activer un type de données énumération**. Comme vous le verrez au [[Chapitre 4#Utilisation du type `System.Enum`|Chapitre 4]], ==le mot-clé C# `enum` vous permet de définir un ensemble personnalisé de *paires nom-valeur*==. Pour vous mettre en appétit, considérez la fonction d'aide finale suivante, qui effectue un test `switch` sur l'énumération `System.DayOfWeek`. Vous remarquerez certaines syntaxes que je n'ai pas encore examinées, mais concentrez-vous sur la question du passage dans l'`enum` elle-même; les éléments manquants seront complétés au fil des chapitres à venir.

```cs
static void SwitchOnEnumExample()
{
    Console.WriteLine("======= Switch On Enum Example =======");
    Console.Write("Enter your favorite day of the week: ");
    DayOfWeek favDay;

    // Non utilisation de la méthode TryParse pour montrer une déclaration
    // switch sans cas default.
    try
    {
        favDay = (DayOfWeek)Enum.Parse(typeof(DayOfWeek), Console.ReadLine());
    }
    catch (Exception)
    {
        Console.WriteLine("Bad input!");
        return;
    }
    switch (favDay)
    {
        case DayOfWeek.Sunday:
            Console.WriteLine("Football!!");
            break;
        case DayOfWeek.Monday:
            Console.WriteLine("Another day, another dollar");
            break;
        case DayOfWeek.Tuesday:
            Console.WriteLine("At least it is not Monday");
            break;
        case DayOfWeek.Wednesday:
            Console.WriteLine("A fine day.");
            break;
        case DayOfWeek.Thursday:
            Console.WriteLine("Almost Friday...");
            break;
        case DayOfWeek.Friday:
            Console.WriteLine("Yes, Friday rules!");
            break;
        case DayOfWeek.Saturday:
            Console.WriteLine("Great day indeed.");
            break;
    }
    Console.WriteLine();
}
```

==Il n'est pas possible de passer d'une instruction `case` à une autre==, mais **que se passe-t-il si plusieurs instructions `case` produisent le même résultat ?** Heureusement, ==elles peuvent être combinées==, comme le montre l'extrait de code suivant:

```cs
case DayOfWeek.Saturday:
case DayOfWeek.Sunday:
	Console.WriteLine("It's the weekend!");
	break;
```

Si un code était inclus entre les instructions case, le compilateur générerait une erreur. **Tant qu'il s'agit d'instructions consécutives, comme indiqué précédemment, les instructions `case` peuvent être combinées pour partager du code commun.**

En plus des instructions `return` et `break` présentées dans les exemples de code précédents, l'instruction **`switch` prend également en charge l'utilisation d'un `goto` pour quitter une condition `case` et exécuter une autre instruction `case`**. Bien que cela soit pris en charge, ==cette pratique est généralement considérée comme un anti-modèle et n'est généralement pas utilisée==. Voici un exemple d'utilisation de l'instruction `goto` dans un bloc `switch` :

```cs
static void SwitchWithGoto()
{
   var foo = 5;
   switch (foo)
   {
      case 1:
         // Do something
         goto case 2;
      case 2:
         // Do something
         break;
      case 3:
         // Do something
         goto default;
      default:
         //default action
         break;
   }
}
```

## Exécution de la correspondance de modèles dans les instructions switch (Nouveauté C# 7.0, MaJ C# 9.0)

Avant C# 7, les expressions de correspondance dans les instructions `switch` se limitaient à comparer une variable à des valeurs constantes, parfois appelées *constant pattern*. ==Dans C# 7, les instructions `switch` peuvent également utiliser le *type pattern*, dans lequel les instructions `case` peuvent évaluer le *type* de la variable vérifiée et les expressions `case` ne sont plus limitées aux valeurs constantes ==. La règle selon laquelle chaque instruction `case` doit se terminer par un retour ou une rupture s'applique toujours ; ==cependant, les instructions `goto` ne sont pas prises en charge avec le modèle de type==.

>[!Note:]- Note pour les débutant en *POO*.
>Si vous débutez dans la programmation orientée objet, cette section peut vous sembler un peu confuse. Tout deviendra clair au [[Chapitre 6#Retour sur la correspondance de motifs (Nouveauté C 7.0)|Chapitre 6]], lorsque vous revisiteriez les nouvelles fonctionnalités de correspondance de modèles de C# 7 dans le contexte des classes et des classes de base. Pour l'instant, contentez-vous de comprendre qu'il existe une nouvelle façon puissante d'écrire des instructions `switch`.

Ajouter une autre méthode nommée `ExecutePatternMatchingSwitch()` et ajoutez-y le code suivant:

```cs
static void ExecutePatternMatchingSwitch()
{
    Console.WriteLine("======= Execute Pattern Matching Switch =======");
	
    Console.Write("1 [Interger (5)], 2 [String (\"Hi\")], 3 [Decimal (2.5)]");
    string userChoise = Console.ReadLine();
    object choice;
    // Ceci est une instruction switch à modèle constant 
    // standard pour configurer l'exemple.
    switch (userChoise)
    {
        case "1":
            choice = 5;
            break;
        case "2":
            choice = "Hi";
            break;
        case "3":
            choice = 2.5M;
            break;
        default:
            choice = 5;
            break;
    }
    // Ceci est nouveau, l'instruction switch 
    // de correspondance de motifs
    switch (choice)
    {
        case int i:
            Console.WriteLine("You choice is an Integer.");
            break;
        case string s:
            Console.WriteLine("Your choice is a string.");
            break;
        case decimal d:
            Console.WriteLine("Your choice is a decimal.");
            break;
        default:
            Console.WriteLine("Your choice is something else.");
            break;
    }
    Console.WriteLine();
}

```

La première instruction `switch` utilise le modèle constant standard et est incluse uniquement pour configurer cet exemple (triviale). 

==Dans la deuxième instruction `switch`, la variable est de type `objet` et, en fonction de la saisie de l'utilisateur, peut être analysée en un type de données `int`, `string` ou `decimal`==. En fonction du type de la variable, différentes instructions `case` sont associées. **En plus de vérifier le type de données, une variable est attribuée dans chacune des instructions `case` (à l'exception du cas par `défaut`)**. Mettez à jour le code comme suit pour utiliser les valeurs dans les variables :

```cs
switch (choice)
{
	case int i:
		Console.WriteLine($"You choice is an Integer {i}.");
		break;
	case string s:
		Console.WriteLine($"Your choice is a string {s}.");
		break;
	case decimal d:
		Console.WriteLine($"Your choice is a decimal {d}.");
		break;
	default:
		Console.WriteLine("Your choice is something else.");
		break;
}
```

En plus d'évaluer le type de l'expression de correspondance, **des clauses `when` peuvent être ajoutées aux instructions `case` pour évaluer les conditions sur la variable. Dans cet exemple, en plus de vérifier le type, la valeur du type converti est également vérifiée pour une correspondance :

```cs
static void ExecutePatternMatchingSwitchWithWhen()
{
    Console.WriteLine("======= Execute Pattern Matching Switch With When =======");

    Console.WriteLine("1 [C#], 2 [VB]");
    Console.Write("Please pick your language preference: ");

    // On utilise le type object et non string parce qu'il n'y a pas d'héritage entre string et int.
    object langChoice = Console.ReadLine();
    var choice = int.TryParse(langChoice.ToString(), out int c) ? c : langChoice;

    switch (choice)
    {
        case int i when i == 2:
        case string s when s.Equals("VB", StringComparison.OrdinalIgnoreCase):
            Console.WriteLine("VB: OOP, multithreading, and more!");
            break;
        case int i when i == 1:
        case string s when s.Equals("C#", StringComparison.OrdinalIgnoreCase):
            Console.WriteLine("Good choice, C# is a fine language.");
            break;
        default:
            Console.WriteLine("Well...good luck with that!");
            break;
    }
    Console.WriteLine();
}
```

>[!example]- Explication du code précédent 
>On vérifie si on a entré un chiffre ou si on a entré le "nom" du cas (`C#` ou `VB`). 
>
>Si la saisie est un chiffre (`TryParse`), alors on convertit en `int`, sinon, on le garde tel quel (`string`). Ensuite, la logice du bloc `switch` est la suivante:
>
> 1) Dans le cas ou `choice` est un int 
> 	1) *ET QUAND* la valeur est `2` 
> 2) **OU** Dans le cas ou `choice` est un `string` 
> 	1) *ET QUAND* la valeur est `"VB"`, 
> 	2) -> On affiche `"VB: OOP, multithreading, and more!"`
> 3) Dans le cas ou `choice` est un int
>	1) *ET QUAND* la valeur est `1`
> 4) **OU** Dans le cas ou `choice` est un `string`
> 	1) *ET QUAND* la valeur est `C#`
> 	2) -> On affiche `"Good choice, C# is a fine language."`
> 5) Par défault
> 	1) -> On affiche `"Well...good luck with that!"`

==Cela ajoute une nouvelle dimension à l'instruction `switch`, car l'ordre des instructions `case` est désormais significatif==. Avec le modèle constant, chaque instruction `case` devait être unique. Avec le modèle de type, ce n'est plus le cas. Par exemple, le code suivant correspondra à chaque entier dans la première instruction `case` et n' exécutera jamais la deuxième ou la troisième (en fait, le code suivant échouera à la compilation) :

```cs
switch (choice)
{
	case int i:
		// do something
		break;
	case int i when i == 0:
		// do semething
		break;
	case int i when i == -1:
		// do semething
		break;
```

Lors de la sortie initiale de C# 7, il y avait un petit problème avec la correspondance de motifs lors de l'utilisation de types génériques. Ce problème a été résolu avec C# 7.1. Les types génériques seront abordés au [[Chapitre 10#Premier aperçu des collections génériques `<T>`|Chapitre 10]].

>[!Info] Toute les améliorations apporté par C# 9.0 concernant les correspondance de motifs sont aussi disponible pour les instruction `switch`
>

## Utilisation des expressions `switch` (Nouveauté C# 8.0)

**Les expressions switch sont une nouveauté de C# 8. Elles permettent d'affecter une variable dans une instruction concise**. Prenons la version C# 7 de cette méthode qui prend une couleur et renvoie la valeur hexadécimale du nom de la couleur :

```cs
static string FromRainboxClassic(string colorBand)
{
   switch (colorBand)
   {
      case "Red":
         return "#FF0000";
      case "Orange":
         return "#FF7F00";
      case "Yellow":
         return "#FFFF00";
      case "Green":
         return "#00FF00";
      case "Blue":
         return "#0000FF";
      case "Indigo":
         return "#4B0082";
      case "Violet":
         return "#9400D3";
      default:
         return "#FFFFFF";
   }
}
```

**Avec les nouvelles expressions `switch` dans C# 8, la méthode précédente peut être écrite comme suit, ce qui est beaucoup plus concis :**

```cs
 static string FromRainbow(string colorBand)
{
   return colorBand switch
   {
      "Red" => "#FF0000",
      "Orange" => "#FF7F00",
      "Yellow" => "#FFFF00",
      "Green" => "#00FF00",
      "Blue" => "#0000FF",
      "Indigo" => "#4B0082",
      "Violet" => "#9400D3",
      _ => "#FFFFFF",
   };
}
```

Cet exemple comporte de nombreux éléments à décortiquer, ==des expressions lambda (`=>`) aux expressions de suppression (`_`). Tous ces éléments seront abordés plus en détail dans les chapitres suivants==, tout comme cet exemple. 

Il y a un autre exemple avant de terminer le sujet des expressions `switch`, et il concerne **les tuples**. Les tuples sont abordés en détail au [[Chapitre 4#Comprendre les tuples (C 7.0)|Chapitre 4]], alors pour l'instant, considérez un tuple comme une construction simple contenant plusieurs valeurs et définie par des parenthèses, comme ce tuple qui contient un `string` et un `int` :

```cs
(string, int)
```

**Dans l'exemple suivant, les deux valeurs transmises à la méthode `RockPaperScissors` sont converties en un `tuple`, puis l'expression `switch` évalue les deux valeurs dans une seule expression. Ce modèle permet de comparer plusieurs valeurs au cours d'une instruction `switch`.**

```cs
static string RockPaperScissors(string first, string second)
{
   return (first, second) switch
   {
      ("rock", "paper") => "Paper wins.",
      ("rock", "scissors") => "Rock wins.",
      ("paper", "rock") => "Paper wins.",
      ("paper", "scissors") => "Scissors wins.",
      ("scissors", "rock") => "Rock wins.",
      ("scissors", "paper") => "Scissors wins.",
      (_, _) => "Tie.",
   };
}
```

Pour appeler cette méthode, ajoutez les lignes de code suivantes aux instructions de niveau supérieur :

```cs
Console.WriteLine(RockPaperScissors("paper","rock"));
Console.WriteLine(RockPaperScissors("scissors","rock"));
```

Cet exemple sera repris au [[Chapitre 4#Comprendre les tuples (Nouveauté / MaJ C 7.0)|Chapitre 4]] lors de l'introduction des `tuples`.

>[!Note] Les modèles de propriété ont également été introduits dans C# 8.0 et sont abordés au [[Chapitre 4#Comprendre les expressions `switch` de correspondance de motifs de tuples (Nouveauté C 8.0)|Chapitre 4]].

# Résumé du chapitre

L'objectif de ce chapitre était de vous présenter les nombreux aspects fondamentaux du langage de programmation C#. Vous avez examiné les constructions courantes dans toute application que vous pourriez être amené à créer. **Après avoir étudié le rôle d'un objet d'application, vous avez appris que chaque programme exécutable C# doit avoir un type définissant une méthode `Main()`, soit explicitement, soit à l'aide d'instructions de niveau supérieur. Cette méthode sert de point d'entrée au programme.**

Ensuite, vous avez approfondi les détails des types de données intégrés à C# et compris que **chaque mot-clé de type de données (par exemple, `int`) est en réalité une notation abrégée pour un type complet dans l'espace de noms `System`** (`System.Int32`, dans ce cas). ==De ce fait, chaque type de données C# possède un certain nombre de membres intégrés==. Dans le même ordre d'idées, vous avez également découvert **le rôle de l'*élargissement (Upcast)* et du *rétrécissement (Downcast)*, ainsi que celui des mots-clés `checked` et `unchecked`**. 

Le chapitre s'est terminé par la présentation du rôle du **typage implicite à l'aide du mot-clé `var`**. Comme indiqué, le typage implicite est **particulièrement utile lorsque vous travaillez avec le modèle de programmation `LINQ`**. Enfin, vous avez rapidement examiné les **différentes constructions d'itération et de décision prises en charge par C#**.

Maintenant que vous comprenez certains des principes de base, le chapitre suivant ([[Chapitre 4|Chapitre 4]]) complètera votre examen des fonctionnalités principales du langage. Vous serez alors prêt à examiner les fonctionnalités orientées objet de C# à partir du [[Chapitre 5|Chapitre 5]].

[^1]: Seulement pour Windows
[^2]: Pour une explication imagée, voir dans le vault ***Notes Code et CLI***. ou la [documentation .NET](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/reference-types#the-string-type)
---
title: "Chapitre 14: Processus, Domaines d'applications et Contextes de Chargements"
publish: true
---

# <big><big><big><b><font color =green>Processus, Domaines d'Application et Contextes de Chargement</font></b></big></big></big>


Dans ce chapitre, vous explorerez en détail **la manière dont un assembly est hébergé par le runtime et comprendrez la relation entre les processus, les domaines d'application et les contextes d'objets.**

En résumé, ==les *domaines d'application* (ou *AppDomains*) sont des subdivisions logiques au sein d'un processus donné, hébergeant un ensemble d'assemblys .NET Core associés==. Comme vous le verrez, ==un AppDomain est lui-même subdivisé en *limites contextuelles*, qui servent à regrouper des objets .NET Core similaires==. Grâce à la notion de contexte, le runtime peut garantir que les objets ayant des exigences particulières sont traités de manière appropriée.

Bien que la plupart de vos tâches de programmation quotidiennes n'impliquent pas de manipulation directe des processus, des AppDomains ou des contextes d'objets, **==la compréhension de ces concepts est essentielle pour travailler avec les nombreuses API .NET Core, notamment le multithreading, le traitement parallèle et la sérialisation d'objets.==**

>[!warning]- `AppDomain`, Relique du .NET Framework
>Une grande partie de ce chapitre détaille les `AppDomains`. Si on développes sur **macOS/LInux** avec **.NET 6+**:
>
>- **Le concept est "gelé" :** Dans l'univers moderne (Mac/Linux/Cloud), le concept de domaines d'application multiples dans un seul processus a été abandonné.
>- **Unicité forcée :** Sous .NET moderne, chaque processus possède **un seul et unique** `AppDomain`. Les tentatives d'en créer de nouveaux (`AppDomain.CreateDomain`) lèveront une exception `PlatformNotSupportedException`.
>- **Utilité actuelle :** L'auteur en parle pour expliquer comment .NET gérait l'isolation du code autrefois. 
>
>Aujourd'hui, on utilise les `AssemblyLoadContexts` (qui seront abordés plus loin) pour accomplir des tâches similaires de manière plus légère.
>
>---
>*Lire les sections sur les `AppDomains` permet de comprendre la théorie de l'isolation, mais il ne faut pas reproduire les exemples de création de domaines sur un Mac/Linux.*

# Le rôle d'un processus

Le concept de « processus » existait déjà dans les systèmes d'exploitation modernes bien avant la sortie des plateformes .NET/.NET Core. En termes simples, ==un *processus* est un programme en cours d'exécution==. Plus formellement, **un processus est un concept du système d'exploitation utilisé pour décrire un ensemble de ressources** (telles que les bibliothèques de code externes et le thread principal) **et les allocations de mémoire nécessaires à une application en cours d'exécution**. **==Pour chaque application .NET Core chargée en mémoire, le système d'exploitation crée un processus distinct et isolé pour toute sa durée de vie==**. 

**Grâce à cette approche d'isolation des applications, l'environnement d'exécution est beaucoup plus robuste et stable, car la défaillance d'un processus n'affecte pas le fonctionnement des autres**. De plus, ==les données d'un processus ne sont pas directement accessibles par un autre processus, sauf si vous utilisez des outils spécifiques tels que `System.IO.Pipes` ou la classe `MemoryMappedFile`==. De ce fait, **on peut considérer le processus comme une limite fixe et sécurisée pour une application en cours d'exécution**. 

***==Chaque processus se voit attribuer un identifiant unique (PID) et peut être chargé et déchargé indépendamment par le système d'exploitation selon les besoins==*** (ainsi que par programmation). Comme vous le savez peut-être, les utilitaires système, tels que le Moniteur d'activité sur macOS, le Gestionnaire des tâches sur Windows ou la commande `top` dans un terminal, vous permettent de consulter diverses statistiques concernant les processus exécutés sur une machine donnée, notamment leur PID et leur nom d'image (voir les images suivantes).

![[Figure 14.1.png|Gestionnaire de tâches Windows]]

![[Figure 14.1.5.png|Moniteur d'activité MacOS]]

## Le rôle des threads

Chaque processus contient un « thread » initial qui sert de point d'entrée à l'application. Le [[Chapitre 15|Chapitre 15]] examine en détail la création d'applications multithreadées sous la plateforme .NET Core; cependant, pour faciliter la compréhension des sujets abordés ici, quelques définitions sont nécessaires. **Un *thread* est un chemin d'exécution au sein d'un processus**. **Formellement, le premier thread créé par le point d'entrée d'un processus est appelé *thread principal***. **==Tout programme .NET Core==** (application console, service Windows, application WPF, etc.) **==marque son point d'entrée avec la méthode `Main() `ou un fichier contenant des instructions de niveau supérieur==** (qui est converti en une classe `Program` et une méthode `Main()`, comme démontré précédemment dans cet ouvrage). **Lorsque ce code est invoqué, le thread principal est créé automatiquement.**

==Les processus qui contiennent un seul thread principal d'exécution sont intrinsèquement *thread-safe*, étant donné qu'il n'y a qu'un seul thread pouvant accéder aux données de l'application à un instant donné==. Cependant, *==un processus monothread==* (en particulier un processus basé sur une interface graphique) *==semblera souvent un peu lent à répondre à l'utilisateur si ce thread unique effectue une opération complexe==* (comme l'impression d'un fichier texte volumineux, l'exécution d'un calcul mathématique intensif ou la tentative de connexion à un serveur distant situé à des milliers de kilomètres).

**Compte tenu de cet inconvénient potentiel des applications monothread, les systèmes d'exploitation pris en charge par .NET Core** (ainsi que la plateforme .NET Core elle même) **permettent au thread principal de créer des threads secondaires supplémentaires** (également appelés *threads de travail*) **grâce à quelques fonctions d'API telles que `CreateThread()`**. **==Chaque thread (principal ou secondaire) constitue un chemin d'exécution unique au sein du processus et dispose d'un accès concurrent à toutes les données partagées.==**

>[!tip] Le **thread principal** est la "ligne de vie" du processus (dans 95% des cas). Il est le point d'entrée (`Main`) et sa fin déclenche la destruction du processus par l'OS.
>
>>[!info]- Précision sémantique
>>Le processus reste en vie tant qu'au moins **un** "Foreground Thread" (généralement le thread principal) est en cours d'exécution.

Comme vous l'aurez deviné, **les développeurs créent généralement des threads supplémentaires pour améliorer la réactivité globale du programme**. ==Les processus multithread donnent l'illusion que de nombreuses activités se déroulent simultanément==. Par exemple, **==une application peut créer un thread de travail pour effectuer une tâche gourmande en ressources==** (comme l'impression d'un fichier texte volumineux). **==Pendant que ce thread secondaire travaille, le thread principal reste réactif aux entrées utilisateur, ce qui permet au processus global d'offrir des performances potentiellement supérieures==**. *==Cependant, ce n'est pas toujours le cas==* : **l'utilisation d'un trop grand nombre de threads dans un seul processus peut en réalité *dégrader* les performances, car le processeur doit alterner entre les threads actifs du processus (ce qui prend du temps).**

***==Sur certaines machines, le multithreading est généralement une illusion fournie par le système d'exploitation==***. **Les machines qui hébergent un seul processeur** (==non hyperthreadé==) **ne peuvent pas gérer littéralement plusieurs threads simultanément**. **==En réalité, un seul processeur exécute un thread pendant une unité de temps==** (appelée *tranche de temps*), **==en fonction notamment du niveau de priorité du thread==**. **Lorsqu'une tranche de temps est écoulée, le thread en cours est suspendu pour permettre à un autre thread de s'exécuter**. ***==Pour qu'un thread conserve en mémoire son état avant d'être interrompu, chaque thread dispose de la possibilité d'écrire dans le stockage local de thread (TLS) et d'une pile d'appels distincte==***, comme illustré dans l'image suivante

>[!info]- Notes pour les processeur Apple Silicon
> L'Hyper-threading n'existe pas sur Apple Silicon (ratio 1:1 entre cœurs et threads réels). Cependant, l'**illusion du multithreading** (Time Slicing) reste vraie dès que le nombre de threads logiciels dépasse le nombre de cœurs physiques de ma puce M4.

![[Figure 14.2.png|La relation processus/thread]]

>[!info]- Précision sur l'image
>- **Shared data** -> le tas (*heap*)
>- **Call stack** -> la pile (*stack*) : où se trouve es variables locales (`int x = 5;`).
>- **Le Thread B ne peut jamais lire la pile du Thread A**
>- **TLS** -> *stockage local de thread* : - zone mémoire spécifique pour stocker des données qui sont globales à un thread mais invisibles pour les autres.

Si le concept de threads vous est nouveau, ne vous souciez pas des détails. Pour l'instant, retenez simplement qu'un **thread est un chemin d'exécution unique au sein d'un processus**. ==Chaque processus possède un thread principal== (créé via le point d'entrée de l'exécutable) ==et peut contenir des threads supplémentaires créés par programmation==.

# Interaction avec les processus à l'aide de .NET Core

Bien que les processus et les threads ne soient pas nouveaux, **la manière d'interagir avec ces primitives sous la plateforme .NET Core a considérablement évolué** (et c'est tant mieux). ==Afin de vous familiariser avec le monde de la création d'assemblages multithread== (voir le [[Chapitre 15|Chapitre 15]]), ==commençons par examiner comment interagir avec les processus à l'aide des bibliothèques de classes de base de .NET Core.==

**L'espace de noms `System.Diagnostics` définit plusieurs types qui vous permettent d'interagir par programmation avec les processus et divers types liés au diagnostic, tels que le journal des événements système et les compteurs de performances**. **==Dans ce chapitre, nous nous intéresserons uniquement aux types centrés sur les processus définis dans le [[#Tableau 14-1 Sélection de membre de l'espace de noms `System.Diagnostics`|Tableau 14-1]].==**

##### Tableau 14-1: Sélection de membre de l'espace de noms `System.Diagnostics`

| Types centrés sur les processus de l'espace de noms `System.Diagnostics` | Description                                                                                                                                                                                                                                  |
| ------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Process`                                                                | La classe `Process` permet d'accéder aux processus locaux et distants et vous permet de démarrer et d'arrêter des processus par programmation.                                                                                               |
| `ProcessModule`                                                          | Ce type représente un module (*.dll* ou *.exe*) chargé dans un processus. Il est important de noter que le type `ProcessModule` peut représenter *n'importe quel* module : des binaires COM, .NET ou C traditionnels.                        |
| `ProcessModuleCollection`                                                | Cela fournit une collection fortement typée d'objets `ProcessModule`.                                                                                                                                                                        |
| `ProcessStartInfo`                                                       | Ceci spécifie un ensemble de valeurs utilisées lors du démarrage d'un processus via la méthode `Process.Start()`.                                                                                                                            |
| `ProcessThread`                                                          | Ce type représente un thread au sein d'un processus donné. Notez que `ProcessThread` est un type utilisé pour diagnostiquer l'ensemble des threads d'un processus et non pour créer de nouveaux threads d'exécution au sein de ce processus. |
| `ProcessThreadCollection`                                                | Cela fournit une collection fortement typée d'objets `ProcessThread`.                                                                                                                                                                        |

**La classe `System.Diagnostics.Process` vous permet d'analyser les processus exécutés sur une machine donnée** (locale ou distante). **==La classe `Process` fournit également des membres permettant de démarrer et d'arrêter des processus par programmation, d'afficher (ou de modifier) ​​le niveau de priorité d'un processus et d'obtenir la liste des threads actifs et/ou des modules chargés au sein d'un processus donné==**. Le [[#Tableau 14-2 Sélection de propriété du type `Process`|Tableau 14-2]] répertorie certaines des propriétés clés de `System.Diagnostics.Process`.

##### Tableau 14-2: Sélection de propriété du type `Process`

| Propriété         | Description                                                                                                                                                                                                                                                                                                               |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ExitTime`        | Cette propriété récupère l'horodatage associé au processus qui s'est terminé (représenté par un type `DateTime`).                                                                                                                                                                                                         |
| `Handle`          | Cette propriété renvoie le handle (représenté par un `IntPtr`) associé au processus par le système d'exploitation. Cela peut s'avérer utile lors du développement d'applications .NET qui doivent communiquer avec du code non managé (voir [[Chapitre 9#Rappel sur le Code Non Managé (*Unmanaged Code*)\|Chapitre 9]]). |
| `Id`              | Cette propriété récupère l'identifiant du processus (PID) associé.                                                                                                                                                                                                                                                        |
| `MachineName`     | Cette propriété récupère le nom de l'ordinateur sur lequel le processus associé s'exécute.                                                                                                                                                                                                                                |
| `MainWindowTitle` | `MainWindowTitle` récupère le titre de la fenêtre principale du processus (si le processus n'a pas de fenêtre principale, vous recevez une chaîne vide).                                                                                                                                                                  |
| `Modules`         | Cette propriété donne accès au type fortement typé `ProcessModuleCollection`, qui représente l'ensemble des modules (*.dll* ou *.exe*) chargés dans le processus actuel.                                                                                                                                                  |
| `ProcessName`     | Cette propriété reçoit le nom du processus (qui, comme vous le supposez, est le nom de l'application elle-même).                                                                                                                                                                                                          |
| `Responding`      | Cette propriété reçoit une valeur indiquant si l'interface utilisateur du processus<br>répond aux entrées de l'utilisateur (ou est actuellement « bloquée »).                                                                                                                                                             |
| `StartTime`       | Cette propriété récupère l'heure de démarrage du processus associé (via un type `DateTime`).                                                                                                                                                                                                                              |
| `Threads`         | Cette propriété récupère l'ensemble des threads qui s'exécutent dans le processsus associé (représenté par une collection d'objets `ProcessThread`).                                                                                                                                                                      |

En plus des propriétés que nous venons d'examiner, **`System.Diagnostics.Process` définit également quelques méthodes utiles** (voir [[#Tableau 14-3 Sélection de méthode du type `Process`|Tableau 14-3]]).

##### Tableau 14-3: Sélection de méthode du type `Process`

| Méthode               | Description                                                                                                                        |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| `CloseMainWindow()`   | Cette méthode met fin à un processus doté d'une interface utilisateur en envoyant un message de fermeture à sa fenêtre principale. |
| `GetCurrentProcess()` | Cette méthode statique renvoie un nouvel objet `Process` qui représente le processus actuellement actif.                           |
| `GetProcesses()`      | Cette méthode statique renvoie un tableau de nouveaux objets `Process` s'exécutant sur une machine donnée.                         |
| `Kill()`              | Cette méthode interrompt immédiatement le processus associé.                                                                       |
| `Start()`             | Cette méthode démarre un processus.                                                                                                |

## Énumération des processus en cours d'exécution

Pour illustrer la manipulation des objets `Process` (veuillez excuser la redondance), créez un projet d'application console C# nommé *ProcessManipulator*. Ensuite, définissez la méthode d'assistance statique suivante dans le fichier *Program.cs* :

```cs
static void ListAllRunningProcesses()
{
    // Récupères tout les processus sur la machine locale, ordonnée par PID.
    var runningProcs =
        from proc in Process.GetProcesses(".")
        orderby proc.Id
        select proc;

    // Affiche le PID et le nom de chaque processus.
    foreach (var p in runningProcs)
    {
        Console.WriteLine($"-> PID: {p.Id}\tName: {p.ProcessName}");
    }
    Console.WriteLine("****************************\n");
}
```

**La méthode statique `Process.GetProcesses()` renvoie un tableau d'objets `Process` représentant les processus en cours d'exécution sur la machine cible** (**==la notation `"."` représente l'ordinateur local==**). ==Une fois le tableau d'objets `Process` obtenu, vous pouvez appeler n'importe quel membre listé dans le [[#Tableau 14-2 Sélection de propriété du type `Process`|Tableau 14-2]] ainsi que le [[#Tableau 14-3 Sélection de méthode du type `Process`|Tableau 14-3]]==. Ici, vous affichez simplement le PID et le nom de chaque processus, triés par PID. Mettez à jour les instructions de niveau supérieur comme suit :

>[!tip] La méthode `GetProcess()` Possède une surcharge sans paramètre, référant à la machine locale (`"."`).

```cs
using System.Diagnostics;

Console.Title = "Fun with Processes";
Console.WriteLine("***** Fun with Processes *****\n");

ListAllRunningProcesses();
Console.ReadLine();
```

Lorsque vous exécuterez l'application, vous verrez les noms et les PID de tous les processus sur votre ordinateur local. Chaque output sera probablement différente.

```
-> PID: 0       Name:
-> PID: 1       Name: launchd
-> PID: 350     Name: logd
-> PID: 351     Name: smd
-> PID: 352     Name: UserEventAgent
-> PID: 354     Name: fseventsd
-> PID: 355     Name: mediaremoted
-> PID: 358     Name: systemstats
-> PID: 360     Name: accessoryupdaterd
-> PID: 361     Name: uarpassetmanagerd
-> PID: 362     Name: configd
-> PID: 363     Name: endpointsecurityd
-> PID: 364     Name: powerd
-> PID: 365     Name: IOMFB_bics_daemon
-> PID: 367     Name: amfid
-> PID: 369     Name: remoted
-> PID: 371     Name: keybagd
-> PID: 372     Name: softwareupdated
-> PID: 373     Name: corespeechd_system

...

-> PID: 93722   Name: VBCSCompiler
-> PID: 93729   Name: mdworker_shared
-> PID: 93730   Name: mdworker_shared
-> PID: 93776   Name: Brave Browser Helper (Renderer)
-> PID: 93777   Name: mdworker_shared
-> PID: 93783   Name: dotnet
-> PID: 93784   Name: Launcher
-> PID: 93785   Name: dotnet
-> PID: 93793   Name: ProcessManipulator
-> PID: 99061   Name: STExtractionService.privileged
****************************
```

## Analyse d'un processus spécifique

Outre la possibilité d'obtenir la liste complète des processus en cours d'exécution sur une machine donnée, **la méthode statique `Process.GetProcessById()` permet d'obtenir un objet `Process` unique via son PID**. *==Si vous tentez d'accéder à un PID inexistant, une exception `ArgumentException` est levée==*. Par exemple, si vous souhaitez obtenir un objet `Process` représentant un processus dont le PID est $30592$, vous pouvez écrire le code suivant :

```cs
static void GetSpecificProcess()
{
    try
    {
        var theProc = Process.GetProcessById(30952);
        Console.WriteLine(theProc?.ProcessName);
    }
    catch (ArgumentException ex)
    {
        Console.WriteLine(ex.Message);
    }
}
```

À ce stade, vous avez appris à obtenir la liste de tous les processus, ainsi qu'un processus spécifique sur une machine via une recherche par PID. **==Bien qu'il soit utile de connaître les PID et les noms des processus, la classe `Process` vous permet également de découvrir l'ensemble des threads et des bibliothèques actuellement utilisés par un processus donné==**. Voyons comment procéder.

## Analyse de l’ensemble des threads d’un processus

>[!danger] **Limitations de macOS** (avec Gemini)
> **La classe `ProcessThread` est très limitée sur macOS**. Les statistiques temporelles (`StartTime`, `ExecutionTime`, `TotalProcessorTime`, `UserProcessorTime`, `PrivilegedProcessorTime`,...) ne sont pas exposées par le runtime .NET sur macOS. **==Seules les informations structurelles (`ID`, `Priority`) sont fiables.==**
> 
>**Il n'y a pas de méthode directe dans `System.Diagnostics` pour obtenir le `StartTime` d'un thread spécifique sous macOS**. La BCL essaie d'être universelle, mais les systèmes Unix gèrent l'isolation des threads de manière très différente de Windows.
>
>Sous Linux, presque toutes les informations sur les processus et les threads sont exposées via un système de fichiers virtuel appelé **`/proc`**. Pour chaque thread, il existe un fichier texte contenant son `StartTime`, sa priorité, etc.
>
>Apple utilise un noyau appelé **Darwin** (basé sur Mach et BSD). ***==Pour des raisons de sécurité et de performance, Apple restreint l'accès aux métadonnées des threads==***. Pour obtenir le `StartTime` d'un thread sur Mac, il faut appeler des API privées ou très complexes du noyau (XNU)
>
>**Microsoft a jugé que l'effort n'en valait pas la chandelle :** Comme ces API changent souvent avec les versions de macOS (pour renforcer la sécurité), maintenir ce code dans .NET serait un cauchemar technique.
>>[!Attention]  
>>certaines propriétés non supportées ne lèvent pas de `PlatformNotSupportedException` mais retournent silencieusement une valeur par défaut (`0`, `DateTime.MinValue`...). 
>>
>>Il faut toujours vérifier le comportement sur macOS avec `RuntimeInformation.IsOSPlatform(OSPlatform.OSX)` avant d'utiliser des données de `ProcessThread` en production.

**L’ensemble des threads est représenté par la collection fortement typée `ProcessThreadCollection`, qui contient un certain nombre d’objets `ProcessThread` individuels**. Pour illustrer cela, ajoutez la fonction d’assistance statique suivante à votre application :

```cs
static void EnumThreadForPid(int pID)
{
    Process theProc = null;
    try
    {
        theProc = Process.GetProcessById(pID);
    }
    catch (ArgumentException ex)
    {
        Console.WriteLine(ex.Message);
        return;
    }

    // Affiche les statistiques de chaque thread du processus spécifié.
    Console.WriteLine($"Here are the threads used by: {theProc.ProcessName}");
    ProcessThreadCollection theThreads = theProc.Threads;

    foreach (ProcessThread pt in theThreads)
    {
        string info =
            $@"-> Thread ID: {pt.Id}	Start Time: {pt.StartTime.ToShortTimeString()}	Priority: {pt.PriorityLevel}";
        Console.WriteLine(info);
    }
    Console.WriteLine("****************************\n");
}
```

Comme vous pouvez le constater, **la propriété `Threads` du type `System.Diagnostics.Process` donne accès à la classe `ProcessThreadCollection`**. ==Ici, vous affichez l'ID, l'heure de début et le niveau de priorité de chaque thread du processus spécifié par le client==. À présent, mettez à jour les instructions de niveau supérieur de votre programme afin de inviter l'utilisateur à saisir un PID à examiner, comme suit :

>[!warning]- Comme précisé au début de la sous-section, cette exemple de code ne fonctionne pas sur macOS. (voir plus)
>>[!tip] Comment restreindre les fonctionnalité selon le système d'exploitation ?
>>
>> 1)  **L'utilisation de l'attribut `SupportedOsPlatform()` au niveau de la méthode**
>> ```cs
>> [SupportedOSPlatform("windows")]
>> [SupportedOSPlatform("linux")] 
>> static void PrintThreadInfo(ProcessThread pt) 
>> { 
>> 	// Pas de if nécessaire ! string info = $"-> Thread ID: {pt.Id} Start Time: {pt.StartTime:t}"; 
>> }
>> ```
>>
>>  La particularité de ceci est que la vérification doit s'effectué à un niveau au dessus, avant d'appeler la méthode.
>>  
>> 2) **L'appel à la méthode `RuntimeInformation.IsOSPlatform()`**
>>
>>```cs
>>if (RuntimeInformation.IsOSPlatform(OSPlatform.Linux) 
>>	!! RuntimeInformation.IsOsPlatform(OSPlatform.Windows
>>) { ... }
>>```
>>
>> la méthode `IsOSPlatform` prend un argument de type d'énumération `OSPlatform`.
>>
>> 3) **L'appel aux méthodes `OperatingSystem.IsXXX`**
>> 
>>```cs
>>if (OperatingSystem.IsWindows() || OperatingSystem.IsLinux())
>>{...}
>>```
>>> [!info] 
>>> Les méthodes `OperatingSystem.IsXXX` sont des wrapper officiel .NET de `RuntimeInformation.IsOsPlatform(OsPlatform.XXX)`
>>
>> **Si on utilise "l'inverse", ou que l'on ajoute une étape d'abstraction** (avec un wrapper par exemple), *roslyn* **ne saura pas faire le lien et donc affichera quand même les avertissements.**

```cs
...
// Demande à l'utilisateur un PID et affiche l'ensemble des threads actifs.
Console.WriteLine("***** Enter PID of process to investigate *****");
Console.Write("PID: ");
string pID = Console.ReadLine();
int theProcID = int.Parse(pID);

EnumThreadsForPid(theProcID);
Console.ReadLine();
```

Lorsque vous exécutez votre programme, vous pouvez désormais saisir l'identifiant de processus (PID) de n'importe quel processus sur votre machine et afficher les threads utilisés par ce processus. ==La sortie suivante présente une liste partielle des threads utilisés par le PID $3804$ sur ma machine, qui héberge Edge :==

```
***** Enter PID of process to investigate *****
PID: 3804
Here are the threads used by: msedge
-> Thread ID: 3464 Start Time: 01:20 PM Priority: Normal
-> Thread ID: 19420 Start Time: 01:20 PM Priority: Normal
-> Thread ID: 17780 Start Time: 01:20 PM Priority: Normal
-> Thread ID: 22380 Start Time: 01:20 PM Priority: Normal
-> Thread ID: 27580 Start Time: 01:20 PM Priority: -4
...
************************************
```

**Le type `ProcessThread` possède d'autres membres importants**, outre `Id`, `StartTime` et `PriorityLevel`. Le [[#Tableau 14-4 Sélection de membres du type `ProcessThread`|Tableau 14-4]] présente certains de ces membres.

##### Tableau 14-4: Sélection de membres du type `ProcessThread`

| Membre               | Description                                                                                          |
| -------------------- | ---------------------------------------------------------------------------------------------------- |
| `CurrentPriority`    | Obtient la priorité actuelle du thread                                                               |
| `Id`                 | Obtient l'identifiant unique du fil de discussion                                                    |
| `IdealProcessor`     | Définit le processeur préféré pour l'exécution de ce thread.                                         |
| `PriorityLevel`      | Obtient ou définit le niveau de priorité du thread                                                   |
| `ProcessorAffinity`  | Définit les processeurs sur lesquels le thread associé peut s'exécuter.                              |
| `StartAddress`       | Obtient l'adresse mémoire de la fonction appelée par le système d'exploitation qui a lancé ce thread |
| `StartTime`          | Obtient l'heure à laquelle le système d'exploitation a démarré le thread                             |
| `ThreadState`        | Obtient l'état actuel de ce thread                                                                   |
| `TotalProcessorTime` | Obtient la durée totale pendant laquelle ce thread a utilisé le processeur.                          |
| `WaitReason`         | Obtient la raison pour laquelle le thread est en attente                                             |

>[!warning] Sur macOS, Seulement `Id` est utilisable ! (et `BasePriority`, qui n'est pas recensé dans le tableau)

***==Avant de poursuivre votre lecture, sachez que le type `ProcessThread` n'est pas l'entité utilisée pour créer, suspendre ou arrêter des threads sous la plateforme .NET Core. `ProcessThread` sert plutôt à obtenir des informations de diagnostic sur les threads  actifs au sein d'un processus en cours d'exécution==***. **Vous découvrirez comment créer des applications multithread à l'aide de l'espace de noms `System.Threading` au [[Chapitre 15#L’espace de noms `System.Threading`|Chapitre 15]].**

## Analyse de l’ensemble des modules d’un processus

Ensuite, voyons comment parcourir le nombre de modules chargés hébergés dans un processus donné. ==Dans le contexte des processus, un *module* est un terme générique désignant une *.dll* (ou *.exe*) hébergée par un processus spécifique==. **En accédant à la collection `ProcessModuleCollection` via la propriété `Process.Modules`, vous pouvez énumérer tous les modules hébergés dans un processus : bibliothèques .NET Core, COM ou C traditionnelles**. Voici une fonction d'assistance supplémentaire qui permet d'énumérer les modules d'un processus spécifique en fonction de son PID :

```cs
static void EnumModsForPid(int pID)
{
    Process theProc = null;
    try
    {
        theProc = Process.GetProcessById(pID);
    }
    catch (ArgumentException ex)
    {
        Console.WriteLine(ex.Message);
        return;
    }
    Console.WriteLine(
        "Here are the loaded modules for: {0}",
        theProc.ProcessName
    );
    ProcessModuleCollection theMods = theProc.Modules;
    foreach (ProcessModule pm in theMods)
    {
        string info = $"-> Mod Name: {pm.ModuleName}";
        Console.WriteLine(info);
    }
    Console.WriteLine("************************************\n");
}
```

>[!danger] Conclusion sur l'inspection (Threads/Modules) sur Mac
Les exemples du livre concernant `Process.Threads` et `Process.Modules` sont incompatible avec macOS.
>
>- Sur **macOS**, le système d'exploitation considère l'inspection de la structure interne d'un processus comme une menace de sécurité.
>- **Résultat :** Le code ne plante pas (pas d'exception), mais il renvoie une liste vide ou limitée à l'exécutable lui-même.
>- **Utilité :** Pour un dev Mac, ces propriétés ne sont quasiment jamais utilisées. On utilise les outils natifs d'Apple (**Instruments** ou **Xcode**) pour inspecter les modules chargés.

Pour observer un exemple de résultat, examinons les modules chargés pour le processus hébergeant le programme d'exemple actuel (`ProcessManipulator`). Pour ce faire, exécutez l'application, identifiez le PID attribué à *ProcessManipulator.exe* (via le Gestionnaire des tâches) et transmettez cette valeur à la méthode `EnumModsForPid()`. Vous serez peut-être surpris de voir la liste des fichiers *.dll* utilisés pour un simple projet d'application console (*GDI32.dll*, *USER32.dll*, *ole32.dll*, etc.). Voici un aperçu partiel des modules chargés (modifié pour plus de concision) :

```
Here are (some of) the loaded modules for: ProcessManipulator
Here are the loaded modules for: ProcessManipulator
-> Mod Name: ProcessManipulator.exe
-> Mod Name: ntdll.dll
-> Mod Name: KERNEL32.DLL
-> Mod Name: KERNELBASE.dll
-> Mod Name: USER32.dll
-> Mod Name: win32u.dll
-> Mod Name: GDI32.dll
-> Mod Name: gdi32full.dll
-> Mod Name: msvcp_win.dll
-> Mod Name: ucrtbase.dll
-> Mod Name: SHELL32.dll
-> Mod Name: ADVAPI32.dll
-> Mod Name: msvcrt.dll
-> Mod Name: sechost.dll
-> Mod Name: RPCRT4.dll
-> Mod Name: IMM32.DLL
-> Mod Name: hostfxr.dll
-> Mod Name: hostpolicy.dll
-> Mod Name: coreclr.dll
-> Mod Name: ole32.dll
-> Mod Name: combase.dll
-> Mod Name: OLEAUT32.dll
-> Mod Name: bcryptPrimitives.dll
-> Mod Name: System.Private.CoreLib.dll
...
************************************
```

## Démarrage et arrêt de processus par programmation

**Les derniers aspects de la classe `System.Diagnostics.Process` examinés ici sont les méthodes `Start()` et `Kill()`**. Comme leur nom l'indique, **==ces méthodes permettent de lancer et d'arrêter un processus par programmation==**. Par exemple, considérons la méthode d'assistance statique `StartAndKillProcess()` suivante.

>[!note] Selon les paramètres de sécurité de votre système d'exploitation, vous devrez peut-être disposer des droits d'administrateur pour démarrer de nouveaux processus.

>[!warning]  Attention
>Selon le système d'exploitations, les noms de fichier/extensions ne sont pas les mêmes. De plus la commande système pour ouvrir un programme n'est pas le même (sur les système)
>
>Sur Windows, si tu fais `Process.Start("notepad.exe")`, l'OS cherche l'exécutable. Sur Mac, on utilise souvent une commande système appelée `open`.
>
>- **Windows :** `Process.Start("msedge.exe", "http://google.com")`
>- **Mac :** `Process.Start("open", "-a 'Microsoft Edge.' http://google.com")`
>
>Dans les versions modernes de .NET, `UseShellExecute` est à `false` par défaut. Si on veut ouvrir un fichier avec son application par défaut (comme un `.txt` avec TextEdit), on dois l'activer explicitement.

**Pour Windows**:

```cs
static void StartAndKillProcess()
{
	Process proc = null;
	
	// Lance Edge et va sur Facebook !
	try
	{
		proc = Process.Start(@"C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe",
		"www.facebook.com");
	}
	catch (InvalidOperationException ex)
	{
		Console.WriteLine(ex.Message);
	}
	
	Console.Write("--> Hit enter to kill {0}...", proc.ProcessName);
	Console.ReadLine();
	
	// Tuez tous les processus msedge.exe.
	try
	{
		foreach (var p in Process.GetProcessesByName("MsEdge"))
		{
			p.Kill(true);
		}
	}
	catch (InvalidOperationException ex)
	{
		Console.WriteLine(ex.Message);
	}
}
```

---

**Pour macOS**:

>[!info] 
>Avec cette méthode sur macOS, l'application Microsoft Edge envois des informations dans `stdout` et `stderr` en plein milieu du programme. Il est possible de rediriger les flux avec des propriétés présentes dans la classe `ProcessStartInfo` (voir le détail de la classe plus loin).

```cs
static void StartAndKillProcess()
{
    Process proc = null;

    // Lance Edge et va sur YouTube !
    try
    {
		proc = Process.Start(
			"open",
			"-a \"Brave Browser\" https://www.youtube.com"
		);
    }
    catch (InvalidOperationException ex)
    {
        Console.WriteLine(ex.Message);
    }

    Console.Write("--> Hit enter to kill {0}...", proc.ProcessName);
    Console.ReadLine();

    // Tuez tous les processus msedge.exe.
    try
    {
        foreach (var p in Process.GetProcessesByName("Microsoft Edge"))
        {
            p.Kill(true);
        }
    }
    catch (InvalidOperationException ex)
    {
        Console.WriteLine(ex.Message);
    }

```

**La méthode statique `Process.Start()` a été surchargée à plusieurs reprises**. **==Au minimum, vous devrez spécifier le chemin d'accès et le nom du fichier du processus à lancer==**. ==Cet exemple utilise une variante de la méthode `Start()` qui vous permet de spécifier des arguments supplémentaires à transmettre au point d'entrée du programme, ici, la page web à charger.==

**Après l'appel à la méthode `Start()`, une référence au processus nouvellement activé vous est renvoyée**. **==Pour arrêter le processus, il suffit d'appeler la méthode `Kill()` au niveau de l'instance==**. Dans cet exemple, ==Microsoft Edge lançant de nombreux processus, une boucle est utilisée pour arrêter tous les processus lancés==. **Les appels à `Start()` et `Kill()` sont également placés dans un bloc `try`/`catch` afin de gérer les éventuelles erreurs `InvalidOperationException`**. ***==Ceci est particulièrement important lors de l'appel à la méthode `Kill()`, car cette erreur sera levée si le processus a déjà été arrêté avant l'appel à `Kill()`==***.

>[!note] 
>Lors de l'utilisation du .NET Framework (avant .NET Core), la méthode `Process.Start()` permettait de démarrer le processus en spécifiant soit le chemin complet et le nom du fichier, soit le raccourci système (par exemple, *msedge*). **Avec .NET Core et la prise en charge multiplateforme, il est désormais obligatoire de spécifier le chemin complet et le nom du fichier**. ==Les associations système peuvent être exploitées grâce à `ProcessStartInfo`, comme expliqué dans les deux sections suivantes.==

## Contrôle du démarrage des processus à l'aide de la classe `ProcessStartInfo`

**La méthode `Process.Start()` permet également de passer un objet de type `System.Diagnostics.ProcessStartInfo` afin de spécifier des informations supplémentaires concernant le démarrage d'un processus donné**. Voici une définition partielle de `ProcessStartInfo` (voir la [documentation](https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.processstartinfo?view=net-10.0) ou le [code source de C#](https://source.dot.net/#System.Diagnostics.Process/System/Diagnostics/ProcessStartInfo.cs,7caf1ab108119a18) pour plus de détails) :

```cs
public sealed class ProcessStartInfo : object
{
	public ProcessStartInfo();
	public ProcessStartInfo(string fileName);
	public ProcessStartInfo(string fileName, string arguments);
	public string Arguments { get; set; }
	public bool CreateNoWindow { get; set; }
	public StringDictionary EnvironmentVariables { get; }
	public bool ErrorDialog { get; set; }
	public IntPtr ErrorDialogParentHandle { get; set; }
	public string FileName { get; set; }
	public bool LoadUserProfile { get; set; }
	public SecureString Password { get; set; }
	public bool RedirectStandardError { get; set; }
	public bool RedirectStandardInput { get; set; }
	public bool RedirectStandardOutput { get; set; }
	public Encoding StandardErrorEncoding { get; set; }
	public Encoding StandardOutputEncoding { get; set; }
	public bool UseShellExecute { get; set; }
	public string Verb { get; set; }
	public string[] Verbs { get; }
	public ProcessWindowStyle WindowStyle { get; set; }
	public string WorkingDirectory { get; set; 
}
```

>[!danger]- **Restrictions sur les systèmes basés sur Unix**
>les propriétés suivantes de `ProcessStartInfo` ne sont disponible UNIQUEMENT sur Windows:
>- `PasswordInClearText`
>- `Domain`
>- `LoadUserProfile`
>- `UseCrendentialsForNetworkingOnly`
>- `Password`
>- `CreateNewProcessGroup`

Pour illustrer comment optimiser le démarrage de votre processus, voici une version modifiée de `StartAndKillProcess()`, qui chargera Microsoft Edge et accédera à www.youtube.com, en utilisant l'association Windows MsEdge :

**Pour Windows:**

```cs
static void StartAndKillProcess()
{
	Process proc = null;
	// Lance Microsoft Edge et accédez à YouTube en mode maximisé.
	try
	{
		ProcessStartInfo startInfo = new
			ProcessStartInfo("MsEdge", "www.facebook.com");
		startInfo.UseShellExecute = true;
		proc = Process.Start(startInfo);
	}
	catch (InvalidOperationException ex)
	{
		Console.WriteLine(ex.Message);
	}
	...
}
```

---

**Pour macOS**:

```cs
static void StartAndKillProcess()
{
    Process proc = null;

    // Lance Edge et va sur YouTube avec la fenètre maximisée !
    try
    {
        ProcessStartInfo startInfo = new ProcessStartInfo(
            "open",
            "-a \"Microsoft Edge\" https://www.youtube.com"
        )
        {
            UseShellExecute = true,
        };
        proc = Process.Start(startInfo);
    }
    catch (InvalidOperationException ex)
    {
        Console.WriteLine(ex.Message);
    }

    Console.Write("--> Hit enter to kill {0}...", proc.ProcessName);
    Console.ReadLine();

    // Tuez tous les processus msedge.exe.
    try
    {
        foreach (var p in Process.GetProcessesByName("Microsoft Edge"))
        {
            p.Kill(true);
        }
    }
    catch (InvalidOperationException ex)
    {
        Console.WriteLine(ex.Message);
    }
}
```

Dans .NET Core, **la propriété `UseShellExecute` est définie par défaut sur `false`, tandis que dans les versions précédentes de .NET, elle est définie par défaut sur `true`**. *==C'est pourquoi l'ancienne version de `Process.Start()`, présentée ici, ne fonctionne plus sans utiliser `ProcessStartInfo` et définir la propriété `UseShellExecute` sur `true`==* :

```cs
Process.Start("msedge")
```

### Protocoles et Associations (Windows vs macOS) par Gemini

Le livre explique comment utiliser des associations système (comme `microsoft-edge:`) pour lancer des applications. Sur **macOS**, ce concept existe sous le nom de **URL Schemes** et se comporte de manière quasi identique, mais avec une mécanique sous-jacente différente.

#### Le mécanisme "URL Scheme"

- **Sur Windows :** C'est une clé dans le Registre qui lie un protocole à un `.exe`.
- **Sur macOS :** C'est défini dans le fichier `Info.plist` à l'intérieur du paquet `.app`. Lors de l'installation, Edge dit à macOS : _"Je suis le responsable pour tout ce qui commence par `microsoft-edge:`"_.

Pour utiliser ces protocoles sur Mac, la propriété `UseShellExecute = true` est **obligatoire**. Elle délègue la résolution du nom au Shell de macOS (qui appelle ensuite les _Launch Services_).

#### la stratégie de lancement universelles sur macOS

Voici comment classer les méthodes de lancement par ordre de fiabilité décroissante :

- **Le plus fiable (Standard Mac) :** `open -a "NomDeL'App"`. 
	- Marche avec 100% des applications.
- **Le plus verbeux (Binaire direct) :** `/Applications/App.app/Contents/MacOS/Binaire`. 
	- Très puissant (donne le contrôle total), mais peut polluer ta console avec des logs. 
	- C'est aussi la seule méthode qui permet de **récupérer un objet `Process` fonctionnel** (avec un PID stable et la possibilité de faire `.Kill()`) pour un contrôle total depuis le code C#
- **Le plus spécifique (Protocole) :** `nom-app:argument`. 
	- Ne marche que si l'app a été codée pour ça (Edge, Slack, Spotify).

*==Si on essaies d'utiliser la syntaxe `nom-app:argument` avec un programme qui n'a pas enregistré de protocole, le programme plantera ou ne fera rien du tout.==*

Cette méthode par "protocole" ,**quand elle est disponible** est souvent plus robuste que d'écrire le chemin complet (`/Applications/...`) car :

- Elle fonctionne même si l'utilisateur a déplacé l'application Edge.
- Elle évite de gérer les espaces et les guillemets dans les chemins de fichiers.
- Elle permet au système de gérer lui-même le lancement de l'application associée.

Bien que Troelsen parle d'associations Windows, .NET moderne et macOS supportent la même logique via les **URL Schemes**. ***==Utiliser `microsoft-edge:` sur Mac est une alternative élégante à la commande `open -a` quand elle est disponible.==***


>[!warning] **Les Verbes OS (Windows vs Mac)**  
La propriété `ProcessStartInfo.Verb` est une fonctionnalité **Windows uniquement** 
>
>- **Sur Mac :** .NET ne peut pas accéder aux actions du menu contextuel du Finder via cette propriété.
>- **Solution :** Pour simuler des verbes sur macOS, il faut passer des arguments spécifiques à la commande `open`. Par exemple, utiliser l'argument `-R` pour simuler le verbe "Explore" (révéler dans le Finder).
>
>Bien que `.Verbs` soit compilable sous .NET 10, l'affectation d'un `Verb` suivie d'un `Process.Start()` avec `UseShellExecute = true` provoque un **plantage immédiat** sur macOS.
>
>- **Raison :** L'implémentation Unix de .NET ne sait pas mapper les verbes Windows vers des actions macOS.
>- **Solution :** Toujours encapsuler l'usage des Verbes dans un `if (OperatingSystem.IsWindows())`.

Outre l'utilisation des raccourcis système pour lancer des applications, **vous pouvez également tirer parti des associations de fichiers avec `ProcessStartInfo`**. ***==Sous Windows==***, ==un clic droit sur un document Word vous permet de le modifier ou de l'imprimer==. Utilisons `ProcessStartInfo` pour déterminer les actions disponibles et les exploiter afin de manipuler le processus. 

Créez une nouvelle méthode avec le code suivant :

```cs
static void UseApplicationVerbs()
{
    int i = 0;
    // Ajuste le chemin d'accès et le nom vers un document de la machine
    ProcessStartInfo si = new ProcessStartInfo(@"./TestPage.docx");
    foreach (var verb in si.Verbs)
    {
        Console.WriteLine($"  {i++}. {verb}");
    }
    si.WindowStyle = ProcessWindowStyle.Minimized;
    si.Verb = "Edit";
    si.UseShellExecute = true;
    Process.Start(si);
}
```

>[!warning] La propriété `windowStyle` ne fonctionne pas sur macOS !

Lorsque vous exécutez ce code, la première partie affiche tous les verbes disponibles pour un document Word, comme le montre l'exemple suivant :

```
***** Fun with Processes *****

...

0. Edit
1. OnenotePrintto
2. Open
3. OpenAsReadOnly
4. Print
5. Printto
6. ViewProtected
```

Après avoir défini `WindowStyle` sur `Maximized`, le verbe est défini sur `Edit`, ce qui ouvre le document en mode édition. Si vous définissez le verbe sur `Print`, le document sera envoyé directement à l'imprimante. 

Maintenant que vous comprenez le rôle des processus (principalement sur Windows, on va pas se le cacher) et comment interagir avec eux depuis du code C#, vous êtes prêt à explorer le concept de domaine d'application .NET.

>[!note] 
>Le répertoire dans lequel l'application s'exécute dépend de la manière dont vous lancez l'exemple d'application. **Si vous utilisez la commande CLI `dotnet run`, le répertoire courant est celui où se trouve le fichier projet** (*.csproj*). ==Si vous utilisez Visual Studio, le répertoire courant sera celui de l'assembly compilé, c'est-à-dire `.\bin\debug\net6.0` (dépendant de la version de .NET)==. **Vous devrez adapter le chemin d'accès au document Word en conséquence.**

# Comprendre les domaines d'application .NET

>[!failure] Fossile du passé
>Le livre consacre une section aux AppDomains pour l'historique, mais en .NET Core+, l'isolation se fait via **AssemblyLoadContext**. Sur macOS, on ne peux pas créer de nouveaux domaines; on vit dans le `Default AppDomain`.
>>[!success] Le seul moment où l'on utiliseras encore `AppDomain` dans le code, c'est pour accéder aux informations du domaine actuel (le seul qui existe) :
>>```cs
>>// Pour savoir où ton app est exécutée et quel est son nom
AppDomain current = AppDomain.CurrentDomain;
Console.WriteLine($"Nom du domaine : {current.FriendlyName}");
>>```

Sous les plateformes .NET et .NET Core, les exécutables ne sont pas hébergés directement au sein d'un processus Windows, comme c'est le cas pour les applications non managées traditionnelles. Les exécutables .NET et .NET Core sont plutôt hébergés par une partition logique au sein d'un processus appelée *domaine d'application*. Cette partition d'un processus Windows traditionnel offre plusieurs avantages, dont voici quelques exemples :

- Les domaines d'application sont un élément clé de la nature indépendante du système d'exploitation de la plateforme .NET Core, car cette division logique masque les différences de représentation d'un exécutable chargé par un système d'exploitation sous-jacent.

- Les domaines d'application sont beaucoup moins gourmands en ressources de traitement et de mémoire qu'un processus complet. Ainsi, CoreCLR peut charger et décharger les domaines d'application beaucoup plus rapidement qu'un processus formel et améliorer considérablement l'évolutivité (*scalability*) des applications serveur.

>[!note] 
>La prise en charge des AppDomains a été modifiée dans .NET Core. Dans .NET Core, il n'existe qu'un seul AppDomain. **La création de nouveaux AppDomains n'est plus prise en charge car elle nécessite une prise en charge au niveau de l'exécution et est généralement coûteuse à créer**. L'`ApplicationLoadContext` (traité plus loin dans ce chapitre) assure l'isolation des assemblys dans .NET Core.

## La classe `System.AppDomain`

***==La classe `AppDomain` est largement obsolète avec .NET Core==***. Bien que la plupart des fonctionnalités restantes soient conçues pour faciliter la migration de .NET 4.x vers .NET Core, ==elles peuvent encore s'avérer utiles, comme expliqué dans les deux sections suivantes.==

## Interaction avec le domaine d'application par défaut

>[!success] fonctionne parfaitement sur macOS

**Votre application a accès au domaine d'application par défaut via la propriété statique `AppDomain.CurrentDomain`**. Grâce à ce point d'accès, **==vous pouvez utiliser les méthodes et propriétés d'`AppDomain` pour effectuer des diagnostics d'exécution.==**

Pour apprendre à interagir avec le domaine d'application par défaut, commencez par créer un nouveau projet d'application console nommé *DefaultAppDomainApp* et **==désactivez la possibilité de valeurs nulles dans le fichier projet==**. Ensuite, mettez à jour votre fichier *Program.cs* avec le code suivant, qui affichera simplement des informations sur le domaine d'application par défaut, à l'aide de plusieurs membres de la classe `AppDomain` :

```cs
using System.Reflection;
using System.Runtime.Loader;

Console.Title = "Fun with the Default AppDomain";
Console.WriteLine("***** Fun with the Default AppDomain *****\n");

DisplayDADStats();
Console.ReadLine();

static void DisplayDADStats()
{
    // Obtient l'accès au domaine d'application du thread actuel.
    AppDomain defaultAD = AppDomain.CurrentDomain;

    // Affiche divers données à propos du domaine
    Console.WriteLine($"Name of the domain: {defaultAD.FriendlyName}");
    Console.WriteLine($"ID of domain in process: {defaultAD.Id}");
    Console.WriteLine(
        $"Is this the default domain?: {defaultAD.IsDefaultAppDomain()}"
    );
    Console.WriteLine(
        $"Base directory of this domain: {defaultAD.BaseDirectory}"
    );
    Console.WriteLine("Setup information for this domain:");
    Console.WriteLine(
        $"\t Application Base: {defaultAD.SetupInformation.ApplicationBase}"
    );
    Console.WriteLine(
        $"\t Target Framework: {defaultAD.SetupInformation.TargetFrameworkName}"
    );
}
```

Le résultat de cet exemple est présenté ici :

```
***** Fun with the Default AppDomain *****

Name of the domain: DefaultAppDomainApp
ID of domain in process: 1
Is this the default domain?: True
Base directory of this domain: ... /DefaultAppDomainApp/bin/Debug/net10.0/
Setup Information for this domain:
	Application Base: ... /DefaultAppDomainApp/bin/Debug/net10.0/
	Target Framework: .NETCoreApp,Version=v10.0
```

**Notez que le nom du domaine d'application par défaut sera identique au nom de l'exécutable qu'il contient** (*DefaultAppDomainApp.dll,* dans cet exemple). Notez également que **la valeur du répertoire de base , qui sera utilisée pour rechercher les assemblies privés requis en externe, correspond à l'emplacement actuel de l'exécutable déployé.**

## Énumérer des assemblys chargés
>[!success] Fonctionne parfaitement sur macOS

**Il est également possible de découvrir tous les assemblies .NET Core chargés dans un domaine d'application donné à l'aide de la méthode `GetAssemblies()` au niveau de l'instance**. Cette méthode renvoie un tableau d'objets `Assembly` (voir [[Chapitre 17#Comprendre la réflexion|Chapitre 17]]). Pour ce faire, vous devez avoir ajouté l'espace de noms `System.Reflection` à votre fichier de code (comme indiqué précédemment dans cette section).

À titre d'exemple, définissez une nouvelle méthode nommée `ListAllAssembliesInAppDomain()` dans le fichier *Program.cs*. Cette méthode auxiliaire récupère tous les assemblies chargés et affiche le nom convivial et la version de chacun.

```cs
static void ListAllAssembliesInAppDomain()
{
    // Accéder au domaine d'application du thread courant.
    AppDomain defaultAD = AppDomain.CurrentDomain;

    // maintenant, on récupère toute les assemblies
    // chargées dans le domaine d'application par défaut.
    Assembly[] loadedAssemblies = defaultAD.GetAssemblies();
    Console.WriteLine(
        $"***** Here are the assemblies loaded in {defaultAD.FriendlyName} *****"
    );

    foreach (Assembly a in loadedAssemblies)
    {
        Console.WriteLine(
            $"-> Name, Version: {a.GetName().Name}:{a.GetName().Version}"
        );
    }
}
```

Si vous avez mis à jour vos instructions de niveau supérieur pour appeler ce nouveau membre, vous constaterez que le domaine d'application hébergeant votre exécutable utilise actuellement les bibliothèques .NET Core suivantes :

```
...

***** Here are the assemblies loaded in DefaultAppDomainApp
-> Name, Version: System.Private.CoreLib:10.0.0.0
-> Name, Version: DefaultAppDomainApp:1.0.0.0
-> Name, Version: System.Runtime:10.0.0.0
-> Name, Version: System.Console:10.0.0.0
-> Name, Version: System.Runtime.InteropServices:10.0.0.0
-> Name, Version: System.Collections:10.0.0.0
-> Name, Version: System.Memory:10.0.0.0
-> Name, Version: Microsoft.Win32.Primitives:10.0.0.0
-> Name, Version: System.Threading:10.0.0.0
```

**Sachez que la liste des assemblies chargés peut changer à tout moment lorsque vous écrivez du code C#**. Par exemple, supposons que vous ayez mis à jour votre méthode `ListAllAssembliesInAppDomain()` pour utiliser une requête LINQ, qui triera les assemblies chargés par nom, comme suit :

```cs
static void ListAllAssembliesInAppDomain()
{
    // Accéder au domaine d'application du thread courant.
    AppDomain defaultAD = AppDomain.CurrentDomain;

    // maintenant, on récupère toute les assemblies
    // chargées dans le domaine d'application par défaut.
    var loadedAssemblies = defaultAD
        .GetAssemblies()
        .OrderBy(x => x.GetName().Name);
        
    Console.WriteLine(
        $"***** Here are the assemblies loaded in {defaultAD.FriendlyName}"
    );

    foreach (Assembly a in loadedAssemblies)
    {
        Console.WriteLine(
            $"-> Name, Version: {a.GetName().Name}:{a.GetName().Version}"
        );
    }
}
```

Si vous exécutiez à nouveau le programme, vous constateriez que *System.Linq.dll* a également été chargé en mémoire.

```
***** Here are the assemblies loaded in DefaultAppDomainApp
-> Name, Version: DefaultAppDomainApp:1.0.0.0
-> Name, Version: Microsoft.Win32.Primitives:10.0.0.0
-> Name, Version: System.Collections:10.0.0.0
-> Name, Version: System.Console:10.0.0.0
-> Name, Version: System.Linq:10.0.0.0
-> Name, Version: System.Memory:10.0.0.0
-> Name, Version: System.Private.CoreLib:10.0.0.0
-> Name, Version: System.Runtime:10.0.0.0
-> Name, Version: System.Runtime.InteropServices:10.0.0.0
-> Name, Version: System.Threading:10.0.0.0
```

# Isolation des assemblages avec les contextes de chargement d'application

Comme vous venez de le voir, ==les domaines d'application sont des partitions logiques utilisées pour héberger les assemblies .NET Core==. De plus, un domaine d'application peut être subdivisé en plusieurs contextes de chargement. Conceptuellement, un contexte de chargement crée un périmètre pour le chargement, la résolution et, éventuellement, le déchargement d'un ensemble d'assemblies. En résumé, un contexte de chargement .NET Core permet à un domaine d'application d'établir un « emplacement spécifique » pour un objet donné.

>[!note] Je remplace la note de l'auteur par une note plus moderne
>- **AppDomain :** Essentiel pour la **théorie**. C'est le périmètre de sécurité de mon application. Je dois savoir qu'il existe, même si je ne le manipule plus (génère des exceptions).
>- **Load Context :** Essentiel pour la **mécanique**. C'est lui qui gère réellement les fichiers sur mon Mac. L'auteur le considère comme "optionnel" car .NET le gère très bien tout seul sans notre intervention.

**La classe `AssemblyLoadContext` permet de charger des assemblies supplémentaires dans leurs propres contextes**. **Pour illustrer cela, commencez par ajouter un projet de bibliothèque de classes nommé *ClassLibary1* à votre solution actuelle. À l'aide de l'interface de ligne de commande .NET Core, exécutez les commandes suivantes dans le répertoire contenant votre solution actuelle** :

```bash
dotnet new classlib -n ClassLibrary1 -f net10.0
dotnet sln Chapter14_AllProjects.slnx add ClassLibrary1
```

==Ensuite, ajoutez une référence du projet *DefaultAppDomainApp* au projet *ClassLibrary1* en exécutant la commande CLI suivante :==

```bash
dotnet add DefaultAppDomainApp reference ClassLibrary1
```

Si vous utilisez Visual Studio, cliquez avec le bouton droit sur le nœud de la solution dans l'Explorateur de solutions, sélectionnez Ajouter -> Nouveau projet, puis ajoutez une bibliothèque de classes .NET Core nommée *ClassLibrary1*. Le projet est ainsi créé et ajouté à votre solution. Ensuite, ajoutez une référence à ce nouveau projet en cliquant avec le bouton droit sur le projet *DefaultAppDomainApp* et en sélectionnant Ajouter -> Référence de projet. Dans le volet gauche, sélectionnez Projets -> Solution, puis cochez la case *ClassLibrary1*, comme illustré dans l'image suivante.

![[Figure 14.3.png|Ajouter la référence de projet dans Visual Studio]]

Dans cette nouvelle bibliothèque de classes, ajoutez une classe `Car`, comme suit :

```cs
namespace ClassLibrary1;

public class Car
{
    public string PetName { get; set; }
    public string Make { get; set; }
    public int Speed { get; set; }
}
```

Une fois ce nouvel assembly en place, assurez-vous que les instructions `using` suivantes figurent en haut du fichier *Program.cs* du projet *DefaultAppDomainApp* :

```cs
using System.Reflection;
using System.Runtime.Loader;
```

La méthode suivante dans les instructions de niveau supérieur de *DefaultAppDomainApp* est la méthode `LoadAdditionalAssembliesDifferentContexts()`, illustrée ici :

```cs
static void LoadAdditionalAssembliesDifferentContexts()
{
    // On génére le chemin d'accés à l'assemblage en prenant
    // le dossier de base, puis en ajoutant le nom de l'assemblage.
    var path = Path.Combine(
        AppDomain.CurrentDomain.BaseDirectory,
        "ClassLibrary1.dll"
    );

    AssemblyLoadContext lc1 = new AssemblyLoadContext("NewContext1", false);
    var cl1 = lc1.LoadFromAssemblyPath(path);
    var c1 = cl1.CreateInstance("ClassLibrary1.Car");

    AssemblyLoadContext lc2 = new AssemblyLoadContext("NewContext2", false);
    var cl2 = lc2.LoadFromAssemblyPath(path);
    var c2 = cl2.CreateInstance("ClassLibrary1.Car");

    Console.WriteLine(
        "*** Loading Additional Assemblies in Different Contexts ***"
    );

    Console.WriteLine($"Assembly1.Equals(Assembly2) {cl1.Equals(cl2)}");
    Console.WriteLine($"Assembly1 == Assembly2 {cl1 == cl2}");
    Console.WriteLine($"Class1.Equals(Class2) {c1.Equals(c2)}");
    Console.WriteLine($"Class1 == Class2 {c1 == c2}");
}
```

La première ligne utilise la méthode statique `Path.Combine` pour construire le répertoire de l'assembly *ClassLibrary1*.

>[!note] 
>Vous vous demandez peut-être ==pourquoi vous avez créé une référence à un assembly qui sera chargé dynamiquement==. Ceci est afin de garantir que, lors de la compilation du projet, l'assembly *ClassLibrary1* soit également compilé et se trouve dans le même répertoire que *DefaultAppDomainApp*. **Il s'agit simplement d'une commodité pour cet exemple. Il n'est pas nécessaire de référencer un assembly que vous chargerez dynamiquement.**

**Ensuite, le code crée un nouvel `AssemblyLoadContext` nommé `NewContext1`** (premier paramètre de la méthode) **et ne prend pas en charge le déchargement** (deuxième paramètre). **==Ce `LoadContext` sert à charger l'assembly *ClassLibrary1*, puis à créer une instance de la classe `Car`==**. **Si une partie de ce code vous est inconnue, elle sera expliquée plus en détail au [[Chapitre 17#Comprendre la liaison tardive|Chapitre 17]]**. Le processus est répété avec un nouvel `AssemblyLoadContext`, puis les assemblies et les classes sont comparés. Lorsque vous exécuterez cette nouvelle méthode, vous verrez le résultat suivant :

```
*** Loading Additional Assemblies in Different Contexts ***
Assembly1.Equals(Assembly2) False
Assembly1 == Assembly2 False
Class1.Equals(Class2) False
Class1 == Class2 False
```

**Cela démontre que le même assembly a été chargé deux fois dans le domaine de l'application**. Les ==classes sont également différentes, comme prévu.==

Ensuite, ajoutez une nouvelle méthode qui chargera l'assembly à partir du même `AssemblyLoadContext`.

```cs
static void LoadAdditionalAssembliesSameContexts()
{
    var path = Path.Combine(
        AppDomain.CurrentDomain.BaseDirectory,
        "ClassLibrary1.dll"
    );

    AssemblyLoadContext lc1 = new AssemblyLoadContext(null, true);

    var cl1 = lc1.LoadFromAssemblyPath(path);
    var c1 = cl1.CreateInstance("ClassLibrary1.Car");

    var cl2 = lc1.LoadFromAssemblyPath(path);
    var c2 = cl2.CreateInstance("ClassLibrary1.Car");

    Console.WriteLine(
        "*** Loading Additional Assemblies in Same Context ***"
    );

    Console.WriteLine($"Assembly1.Equals(Assembly2) {cl1.Equals(cl2)}");
    Console.WriteLine($"Assembly1 == Assembly2 {cl1 == cl2}");
    Console.WriteLine($"Class1.Equals(Class2) {c1.Equals(c2)}");
    Console.WriteLine($"Class1 == Class2 {c1 == c2}");
}
```

**La principale différence dans ce code est qu'un seul `AssemblyLoadContext` est créé**. ==Désormais, lorsque l'assembly *ClassLibrary1* est chargé deux fois, le second assembly est simplement un pointeur vers la première instance de l'assembly==. L'exécution du code produit la sortie suivante :

```
...

*** Loading Additional Assemblies in Different Contexts ***
Assembly1.Equals(Assembly2) True
Assembly1 == Assembly2 True
Class1.Equals(Class2) False
Class1 == Class2 False
```

# Résumé des processus, des domaines d'application et des contextes de chargement

À ce stade, vous devriez avoir une bien meilleure idée de la manière dont un assembly .NET Core est hébergé par le runtime. Si les pages précédentes vous ont semblé un peu trop techniques, rassurez-vous. **==Dans la plupart des cas, .NET Core gère automatiquement les détails des processus, des domaines d'application et des contextes de chargement==**. **Ces informations constituent une base solide pour comprendre la programmation multithread sur la plateforme .NET Core.**

Un *AppDomain* est une **isolation logique à l'intérieur d'un seul processus**. En .NET Core/.NET 5+, il n'y en a plus **q'un seul par processus**

Un *processus* c'est une **instance d'un programme en cours d'exécution**, avec sa propre mémoire isolée. Ce n'est pas exactement une assembly : une assembly c'est le fichier `.dll`/`.exe` sur disque, le processus c'est ce fichier **chargé et exécuté** par l'OS. Un processus peut charger plusieurs assemblies.

Un *thread* est une **unité d'exécution** gérée par l'OS. Plusieurs threads peuvent tourner sur un seul cœur (via le scheduling de l'OS), et un seul thread peut utiliser plusieurs cœurs via le JIT. ***==La relation threads/cœurs est uniquement gérée que par l'OS.==***

Enfin, un *context d'exécution* représente le **"bagage"** qu'un thread transporte avec lui. Quand un thread démarre ou qu'une continuation `async` reprend, il a besoin de savoir dans quel contexte il s'exécute. Ce contexte contient :

- **SecurityContext** : les permissions de l'utilisateur courant
- **SynchronizationContext** : sur quel thread reprendre après un `await` (crucial en UI)
- **Culture/Locale** : langue et format de dates
- **`AsyncLocal<T>`** : données locales à un flux async

## Hiérarchie
```
OS
└── Process (programme en mémoire, isolation totale)
    └── AppDomain (isolation logique, 1 seul en .NET moderne)
        └── Thread (unité d'exécution)
            └── Execution Context
```

# Résumé du chapitre 

L'objectif de ce chapitre était d'**examiner précisément comment une application .NET Core est hébergée par la plateforme .NET Core**. Comme vous l'avez constaté, ==la notion traditionnelle de processus a été modifiée en interne pour répondre aux besoins du CoreCLR==. **Un processus unique (manipulable par programmation via le type `System.Diagnostics.Process`) est désormais composé d'un domaine d'application, qui représente des limites isolées et indépendantes au sein d'un processus**.

**==Un domaine d'application peut héberger et exécuter un nombre quelconque d'assemblages associés==**. ***==De plus, un domaine d'application unique peut contenir un nombre quelconque de contextes de chargement pour une isolation accrue des assemblies==***. **Grâce à ce niveau d'isolation supplémentaire, le CoreCLR garantit la bonne gestion des objets nécessitant des besoins spécifiques.**

>[!info] Pourquoi l'auteur parle de "besoins spécifiques" à la fin ?
>C'est la partie la plus abstraite. Le "Load Context" permet par exemple de charger une DLL, de l'utiliser pour un calcul, puis de la **décharger** totalement pour libérer de la RAM. **Sans le "Load Context", une DLL chargée reste bloquée jusqu'à la fermeture du programme. C'est ça, la "gestion spécifique".**

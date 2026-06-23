---
title: "Chapitre 15: Programmation Multithread, Parallèle et Asynchrone"
publish: true
---

# <big><big><big><b><font color =green>Programmation Multithread, Parallèle et Asynchrone</font></b></big></big></big>

>[!success]- Cette vidéo YouTube venant de Microsoft est excellente pour comprendre comment fonctionne ce chapitre "sous le capot"
>![Video YouTube sur async await venant de Microsoft](https://www.youtube.com/watch?v=R-z2Hv-7nxk)

Personne n'apprécie de travailler avec une application lente à l'exécution. De plus, personne n'aime démarrer une tâche dans une application (par exemple, en cliquant sur un élément de la barre d'outils) qui empêche d'autres parties du programme d'être aussi réactives que possible. *==Avant la sortie de .NET (et .NET Core), la création d'applications capables d'effectuer plusieurs tâches nécessitait généralement l'écriture de code C++ complexe utilisant les API de gestion des threads Windows==*. **==Heureusement, la plateforme .NET/.NET Core offre plusieurs façons de créer des logiciels capables d'effectuer des opérations complexes sur des chemins d'exécution uniques, avec beaucoup moins de problèmes.==**

**Ce chapitre commence par définir la nature générale d'une « application multithread »**. Ensuite, vous découvrirez **l'espace de noms de gestion des threads d'origine, disponible depuis .NET 1.0, et plus précisément `System.Threading`**. Ici, ==vous examinerez différents types== (`Thread`, `ThreadStart`, etc.) ==qui vous permettent de créer explicitement des threads d'exécution supplémentaires et de synchroniser vos ressources partagées, ce qui contribue à garantir le partage de données non volatile entre plusieurs threads.==

**La suite de ce chapitre examinera trois techniques plus récentes que les développeurs .NET Core
peuvent utiliser pour créer des logiciels multithreadés** : **==la bibliothèque parallèle de tâches==** (TPL), **==Parallel LINQ==** (PLINQ) **==et les mots-clés asynchrones intrinsèques de C#==** (`async` et `await`), **===relativement nouveaux depuis C# 6==**. Comme vous le verrez, ***==ces fonctionnalités peuvent considérablement simplifier la création d'applications logicielles multithreadées réactives.==***

>[!warning]- Multithreading & .NET moderne sur macOS (Avec Gemini)
>
>Ce chapitre sur le multithreading et l'asynchronisme est **quasi-universel**. Le code fonctionnera de la même manière sur Mac que sur Windows grâce aux abstractions de .NET Core / .NET 5+.
>
>### Évolutions Modernes (.NET 9+)
>
>- **Le mot-clé `lock` optimisé** : En C# 13, la nouvelle classe `System.Threading.Lock` est utilisé comme objet de verrouillage. Le compilateur l'utilise pour surpasser les performances de l'ancien `Monitor`.
>- **Priorité au "Slim"** : Privilégiez systématiquement `SemaphoreSlim` au `Semaphore` classique. Sur macOS, il évite des appels système coûteux au noyau BSD.
>- **Async-First** : La synchronisation moderne se fait via `SemaphoreSlim` ou `Task` plutôt que via des threads manuels, car ils ne bloquent pas les ressources du système.
>- **Réduction de l'overhead d'abstraction** : Le compilateur JIT de .NET 10 est bien plus efficace pour optimiser les appels virtuels et les boucles `foreach`, réduisant drastiquement le coût lié à l'utilisation des interfaces et des délégués.
>- **Allocations sur la pile (Stack Allocation)** : Pour les petits tableaux et objets de types valeurs, .NET 10 favorise désormais l'allocation sur la pile plutôt que sur le tas (heap), ce qui soulage le Garbage Collector et booste la performance.
>
>### Spécificités macOS vs Windows
>
>- **Ordonnancement (Scheduling)** : Les `ThreadPriority` sont des suggestions. Le noyau macOS (BSD) gère les priorités différemment ; les changements peuvent être moins visibles que sous Windows.
>- **Mutex Nommés** : Les mutex système globaux (utilisés pour les instances d'applications uniques) sont capricieux sur Mac. Préférez un **verrou sur fichier** pour une compatibilité totale.
>- **Précision du Temps** :
>    - `Stopwatch` : Fiable via `ElapsedMilliseconds`, mais les "ticks" bruts varient selon le matériel Apple Silicon/Intel.
>    - `Timers` : Souvent plus réactifs sur Mac pour les faibles délais, mais moins constants sous une charge CPU extrême.
>
>### Éléments obsolètes (À ignorer sur Mac)
>
>- **`Thread.Abort()`** : Ne fonctionne pas (lance une exception). Utilisez uniquement `CancellationToken`.
>- **ApartmentState (STA/MTA)** : Concept purement lié à l'architecture COM de Windows. Inutile sur macOS.
>- **`Thread.Suspend/Resume`** : Obsolètes, dangereux et à bannir de votre code moderne.
>
>---
>**À retenir** : *Le chapitre 14 (AppDomains) est un vestige de Windows. Le chapitre 15 est votre **fondation réelle** pour le cloud, le mobile et le desktop moderne.*

# La relation Processus/Domaine d'application/Contexte/Thread

Au [[Chapitre 14#Le rôle des threads|Chapitre 14]], un *thread* a été défini comme un chemin d'exécution au sein d'une application exécutable. **Bien que de nombreuses applications .NET Core puissent fonctionner de manière optimale en mode monothread, le thread principal d'un assembly** (==créé par le runtime lors de l'exécution du point d'entrée de l'application==) **peut créer des threads secondaires à tout moment pour effectuer des tâches supplémentaires**. **==La création de threads supplémentaires permet de concevoir des applications plus réactives==** (***==mais pas nécessairement plus rapides sur les machines mono-cœur==***).

**L'espace de noms `System.Threading` a été introduit avec .NET 1.0 et propose une approche pour la création d'applications multithread**. ==La classe `Thread` est sans doute le type principal, car elle représente un thread donné==. **Si vous souhaitez obtenir par programmation une référence au thread exécutant actuellement un membre donné, il suffit d'appeler la propriété statique `Thread.CurrentThread`, comme ceci :

```cs
static void ExtractExecutingThread()
{
	// Récupère le thread en cours d'exécution
	// de cette méthode.
	Thread currThread = Thread.CurrentThread;
}
```

***===Rappelons qu'avec .NET Core, il n'existe qu'un seul AppDomain==***. Bien qu'il soit impossible de créer des AppDomains supplémentaires, **l'AppDomain d'une application peut contenir plusieurs threads exécutés simultanément**. **==Pour obtenir une référence à l'AppDomain hébergeant l'application, appelez la méthode statique `Thread.GetDomain()`==**, comme ceci :

```cs
static void ExtractAppDomainHostingThread()
{
	// Obtient le domaine d'application hébergeant le thread courant.
	AppDomain ad = Thread.GetDomain();
}
```

**Un thread peut être déplacé dans un contexte d'exécution à tout moment, et il peut être relocalisé au sein d'un nouveau contexte d'exécution à la discrétion du runtime .NET Core**. **==Pour obtenir le contexte d'exécution actuel d'un thread, utilisez la propriété statique `Thread.CurrentThread.ExecutionContext`==**, comme ceci :

```cs
static void ExtractCurrentThreadExecutionContext()
{
	// Obtient le contexte d'exécution dans lequel le
	// thread courant opère.
	ExecutionContext ctx =
		Thread.CurrentThread.ExecutionContext;
}
```

Là encore, ==le runtime .NET Core gère le déplacement des threads vers (et hors de) leurs contextes d'exécution==. **En tant que développeur .NET Core, vous pouvez généralement ignorer où un thread donné aboutit. Néanmoins, vous devriez connaître les différentes manières d'obtenir les primitives sous-jacentes.**

## Le problème de la concurrence

**L'un des nombreux « plaisirs »** (ou plutôt les aspects pénibles) **de la programmation multithread est le faible contrôle sur la manière dont le système d'exploitation ou l'environnement d'exécution sous-jacent utilise ses threads**. Par exemple, *==si vous créez un bloc de code qui crée un nouveau thread d'exécution, vous ne pouvez pas garantir son exécution immédiate==*. ==Ce code indique seulement au système d'exploitation/environnement d'exécution d'exécuter le thread dès que possible== (généralement lorsque le planificateur de threads le traite).

De plus, ==étant donné que les threads peuvent être déplacés entre les limites de l'application et du contexte selon les besoins de l'environnement d'exécution, vous devez être attentif aux aspects de votre application qui sont sensibles à la *volatilité des threads*== (par exemple, accessibles par plusieurs threads) ==et quels opérations sont *atomiques*== (les opérations sensibles à la volatilité des threads sont les dangereuses !).

Pour illustrer ce problème, ==supposons qu'un thread appelle une méthode d'un objet spécifique. Supposons maintenant que ce thread reçoive l'instruction du planificateur de threads de suspendre son activité pour permettre à un autre thread d'accéder à la même méthode du même objet.==

**Si le thread initial n'a pas terminé son opération, le second thread entrant risque de consulter un objet partiellement modifié**. ==À ce stade, le second thread lit en réalité des données erronées, ce qui entraînera inévitablement des bogues extrêmement étranges== (***==et difficiles à trouver==***), ==encore plus difficiles à reproduire et à déboguer.==

**Les opérations atomiques, en revanche, sont toujours sûres dans un environnement multithread**. Malheureusement, *==peu d'opérations des bibliothèques de classes de base .NET Core sont garanties atomiques==*. ***==Même l'action d'assigner une valeur à une variable membre n'est pas atomique !==*** **À moins que la documentation .NET Core n'indique explicitement qu'une opération est atomique, vous devez la considérer comme volatile d'un thread à l'autre et prendre des précautions.**

## Le rôle de la synchronisation des threads

==Il est désormais clair que les programmes multithreadés sont par nature assez instables, car de nombreux threads peuvent opérer sur les ressources partagées (plus ou moins) simultanément==. **Pour protéger les ressources d'une application contre toute corruption, les développeurs .NET Core doivent utiliser diverses primitives de gestion des threads (telles que les verrous, les moniteurs et l'attribut `[Synchronization]` ou les mots-clés du langage) afin de contrôler l'accès entre les threads en cours d'exécution. **

**==Bien que la plateforme .NET Core ne puisse pas éliminer complètement les difficultés liées à la création d'applications multithreadées robustes, le processus a été considérablement simplifié==**. **Grâce aux types définis dans l'espace de noms `System.Threading`, la bibliothèque parallèle de tâches (TPL) et les mots-clés `async` et `await` du langage C#, vous pouvez gérer plusieurs threads facilement. **

Avant d'aborder l'espace de noms `System.Threading`, la bibliothèque TPL et les mots clés `async` et `await` de C#, ==vous commencerez par examiner comment le type délégué .NET Core peut être utilisé pour appeler une méthode de manière asynchrone==. **Bien que, depuis .NET 4.6, les nouveaux mots clés `async` et `await` de C# offrent une alternative plus simple aux délégués asynchrones, il est néanmoins important de savoir comment interagir avec du code utilisant cette approche** (croyez-moi, de nombreux programmes en production utilisent des délégués asynchrones).

# L’espace de noms `System.Threading`

**Sous les plateformes .NET et .NET Core, l’espace de noms `System.Threading` fournit des types permettant la construction directe d’applications multithread**. Outre les types permettant d’interagir avec un thread du runtime .NET Core, **cet espace de noms définit des types permettant d’accéder au pool de threads géré par le runtime .NET Core, une classe `Timer` simple** (sans interface graphique) **et de nombreux types utilisés pour fournir un accès synchronisé aux ressources partagées**. Le [[#Tableau 15-1 Types principaux de l'espace de noms `System.Threading`|Tableau 15-1]] répertorie certains des membres importants de cet espace de noms. (Pour plus de détails, consultez la [documentation](https://learn.microsoft.com/en-us/dotnet/api/system.threading?view=net-10.0) du SDK .NET Core.)

##### Tableau 15-1:  Types principaux de l'espace de noms `System.Threading`

| Type                       | Description                                                                                                                                                                                                                   |
| -------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Interlocked`              | Ce type permet d'effectuer des opérations atomiques sur les variables partagées par plusieurs threads.                                                                                                                        |
| `Monitor`                  | Ce type assure la synchronisation des objets multithreadés à l'aide de verrous et de signaux d'attente. Avant .NET 9, le mot-clé `lock` de C# utilise un objet `Monitor` en interne. (***==Voir note suivant le tableau==***) |
| `Mutex`                    | Cette primitive de synchronisation peut être utilisée pour la synchronisation entre les limites des domaines d'application.                                                                                                   |
| `ParameterizedThreadStart` | Ce délégué permet à un thread d'appeler des méthodes qui acceptent un nombre quelconque d'arguments.                                                                                                                          |
| `Semaphore`                | Ce type vous permet de limiter le nombre de threads pouvant accéder simultanément à une ressource. (***==Voir note suivant le tableau==***)                                                                                   |
| `Thread`                   | Ce type représente un thread qui s'exécute au sein du runtime .NET Core. Grâce à ce type, vous pouvez créer des threads supplémentaires dans le domaine d'application d'origine.                                              |
| `ThreadPool`               | Ce type vous permet d'interagir avec le pool de threads géré par le runtime .NET Core au sein d'un processus donné.                                                                                                           |
| `ThreadPriority`           | Cette énumération représente le niveau de priorité d'un thread (Très élevé, Normal, etc.).                                                                                                                                    |
| `ThreadStart`              | Ce délégué permet de spécifier la méthode à appeler pour un thread donné. Contrairement au délégué `ParameterizedThreadStart`, les cibles de ThreadStart doivent toujours avoir le même prototype.                            |
| `ThreadState`              | Cette énumération spécifie les états valides qu'un thread peut prendre (`Running`, `Aborted`, etc.).                                                                                                                          |
| `Timer`                    | Ce type fournit un mécanisme permettant d'exécuter une méthode à intervalles spécifiés. (***==Voir première note du chapitre==***)                                                                                            |
| `TimerCallback`            | Ce type de délégué est utilisé conjointement avec les types Timer.                                                                                                                                                            |

>[!important]- .NET 9 ajoute le type `Lock`, qui est environ 25 % plus performant que l'ancien mécanisme basé sur des objets.
>
>En réalité, dans les versions les plus récentes de C# (C# 13+), le mot-clé `lock` a été amélioré pour utiliser intelligemment la nouvelle classe `System.Threading.Lock`.
>
>Avant .NET 9, le mot-clé `lock` utilisait la classe `Monitor` en arrière-plan. Désormais, si vous passez une instance de la nouvelle classe `Lock` au mot-clé, le compilateur génère un code beaucoup plus performant.
>
>- **Ancienne méthode (toujours valide) :**
>
>	```cs
>	private object _syncRoot = new(); 
>	lock(_syncRoot) { /* ... */ } // Utilise Monitor (plus lent)
>	```
>
>- **Nouvelle méthode (recommandée)**
>
>	```cs
>	private readonly System.Threading.Lock _myLock = new();
>	lock(_myLock) { /* ... */ } // Utilise la nouvelle classe Lock (plus rapi
>	```
>	
>La classe `System.Threading.Lock` a été introduite pour plusieurs raisons :
>
>- **Performance :** Elle est plus légère et évite certains surcoûts liés à l'allocation d'objets.
>- **Clarté :** Utiliser un objet de type `Lock` au lieu d'un simple `object` rend l'intention du code explicite.
>- **Syntaxe `using` :** Elle permet d'utiliser le pattern `using` pour libérer le verrou automatiquement, ce qui est parfois plus propre que le bloc `lock` classique.
>
>**Les limites (Quand ne pas l'utiliser)**
>
>La classe `Lock` ne remplace pas les besoins complexes :
>
>- **Asynchronisme :** Ni le mot-clé `lock`, ni la classe `Lock` ne fonctionnent avec `await`. Pour cela, vous devez toujours utiliser **`SemaphoreSlim`**.
>- **Compatibilité :** Si vous travaillez sur une version plus ancienne que .NET 9, cette classe n'existe tout simplement pas.

>[!important]- .NET 9 ajoute le type `SemaphoreSlim`.
>règle d'or que vous retrouverez dans tous ces ouvrages :
>
>- `Semaphore` (le vieux) = Synchronisation entre plusieurs **processus** (ex: deux instances de votre .exe). **Très couteux en performance (surtout sur macOS)**
>- `SemaphoreSlim` (le moderne) = Synchronisation entre plusieurs **threads/tasks** dans une seule application. C'est celui que vous utiliserez 99% du temps car **beaucoup plus léger et performant**

# La classe `System.Threading.Thread`

==Le type le plus primitif de l'espace de noms `System.Threading` est `Thread`==. **Cette classe représente un wrapper orientée objet autour d'un chemin d'exécution donné au sein d'un AppDomain**. **==Ce type définit également plusieurs méthodes==** (statiques et d'instance) **==permettant de créer de nouveaux threads dans le domaine d'application courant, ainsi que de suspendre, d'arrêter et de détruire un thread==**. Consultez la liste des principaux membres statiques dans le [[#Tableau 15-2 Membres statique clé du type `Thread`|Tableau 15-2]].

##### Tableau 15-2: Membres statique clé du type `Thread`

| Membre statique    | Description                                                                                                                                                                                              |
| ------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ExecutionContext` | **Cette propriété en lecture seule renvoie des informations relatives au fil logique d'exécution, notamment les contextes de sécurité, d'appel, de synchronisation, de localisation et de transaction.** |
| `CurrentThread`    | Cette propriété en lecture seule renvoie une référence au thread en cours d'exécution.                                                                                                                   |
| `Sleep()`          | Cette méthode suspend le thread en cours d'exécution pour un temp défini.                                                                                                                                |

La classe `Thread` prend également en charge plusieurs membres au niveau de l'instance, dont certains sont présentés dans le [[#Tableau 15-3 Sélection de membres d'instance du type `Thread`|Tableau 15-3]].

##### Tableau 15-3: Sélection de membres d'instance du type `Thread`

| Membre au niveau de l'objet | Description                                                                                                                |
| --------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| `IsAlive`                   | Renvoie une valeur booléenne indiquant si ce thread a été démarré (et n'a pas encore été terminé ou abandonné).            |
| `IsBackground`              | Obtient ou définit une valeur indiquant si ce thread est un « thread d'arrière-plan » (plus de détails dans un instant).   |
| `Name`                      | Permet de définir un nom convivial pour le thread.                                                                         |
| `Priority`                  | `get` ou `set` la priorité d'un thread, à laquelle peut être attribuée une valeur issue de l'énumération `ThreadPriority`. |
| `ThreadState`               | Gets the state of this thread, which may be assigned a value from the<br><br>ThreadState enumeration.                      |
| `Abort()`                   | Instructs the .NET Core Runtime to terminate the thread as soon as possible.                                               |
| `Interrupt()`               | Interrupts (e.g., wakes) the current thread from a suitable wait period.                                                   |
| `Join()`                    | Blocks the calling thread until the specified thread (the one on which Join() is<br><br>called) exits.                     |
| `Resume()`                  | Resumes a thread that has been previously suspended.                                                                       |
| `Start()`                   | Instructs the .NET Core Runtime to execute the thread ASAP.                                                                |
| `Suspend()`                 | Suspends the thread. If the thread is already suspended, a call to Suspend() has<br><br>no effect.                         |

>[!note]
>**Interrompre ou suspendre un thread actif est généralement considéré comme une mauvaise idée**. Dans ce cas, il existe un risque (même minime) qu'un thread « fuie » sa charge de travail lorsqu'il est interrompu ou terminé.

##### Tableau 15-4: État et propriétés de la classe Thread (Gemini)

| Membre           | Statut Moderne      | Rôle / Pertinence sur macOS                                                                                                    |
| ---------------- | ------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| **Start()**      | ✅ Indispensable     | Lance l'exécution physique du thread.                                                                                          |
| **Join()**       | ✅ Utile             | Attend que le thread se termine avant de continuer.                                                                            |
| **Name**         | ✅ Recommandé        | **Indispensable pour le débuggage.** Permet d'identifier le thread dans la fenêtre "Threads" de VS Code ou Visual Studio.      |
| **IsBackground** | ⚠️ Partiel          | Définit si le thread doit être tué quand l'app se ferme. Les `Tasks` modernes sont presque toutes en background par défaut.    |
| **IsAlive**      | ℹ️ Consultatif      | Utilisé pour savoir si le thread a démarré et n'est pas encore fini. Souvent remplacé par le suivi d'une `Task`.               |
| **Priority**     | 📉 Faible           | Une "suggestion" à l'OS. Sur macOS, l'impact est minime car le noyau gère l'ordonnancement de façon autonome.                  |
| **ThreadState**  | 🔍 Debug uniquement | Un drapeau (enum) qui indique l'état (Running, WaitSleepJoin, etc.). Très complexe à utiliser correctement en logique de code. |
| **Interrupt()**  | ⚠️ Rare             | Réveille un thread endormi via une exception. On préfère `CancellationToken`.                                                  |
| **Abort()**      | ❌ **Obsolète**      | Ne fonctionne plus. **Utilisez `CancellationToken`.**                                                                          |
| **Suspend()**    | ❌ **Obsolète**      | Bloque le thread arbitrairement. **À bannir.**                                                                                 |
| **Resume()**     | ❌ **Obsolète**      | Relance un thread suspendu. **À bannir.**                                                                                      |

## Obtention de statistiques sur le thread d'exécution courant

**Rappelons que le point d'entrée d'un assembly exécutable** (c'est-à-dire ==les instructions de niveau supérieur ou la méthode `Main()`==) **s'exécute sur le thread principal**. Pour illustrer l'utilisation de base du type `Thread`, supposons que vous ayez un nouveau projet d'application console nommé *ThreadStats*. Comme vous le savez, **==la propriété statique `Thread.CurrentThread` récupère un objet `Thread` représentant le thread en cours d'exécution. Une fois que vous avez obtenu le thread courant, vous pouvez afficher diverses statistiques==**, comme ceci :

```cs
Console.Title = "Primary Thread Stats";
Console.WriteLine("***** Primary Thread Stats *****\n");

// Obtient le nom et le thread actuel
Thread primaryThread = Thread.CurrentThread;
primaryThread.Name = "ThePrimaryThread";

// Affiche quelque donnée à propos de ce thread.
Console.WriteLine($"Id of current thread: {primaryThread.ManagedThreadId}");
Console.WriteLine($"Thread Name: {primaryThread.Name}");
Console.WriteLine($"Has thread started?: {primaryThread.IsAlive}");
Console.WriteLine($"Priority Level: {primaryThread.Priority}");
Console.WriteLine($"Thread State: {primaryThread.ThreadState}");

Console.ReadLine();
```

Voici le résultat actuel :

```
***** Primary Thread Stats *****

Id of current thread: 1
Thread Name: ThePrimaryThread
Has thread started?: True
Priority Level: Normal
Thread State: Running
```

## Propriété `Name`

Notez que la classe `Thread` prend en charge une propriété appelée `Name`. ==Si vous ne définissez pas cette valeur, `Name` renverra un `string.Empty`==. Toutefois, **une fois que vous avez attribué un nom convivial à un objet Thread donné, vous pouvez grandement simplifier vos opérations de débogage**. Si vous utilisez Visual Studio, vous pouvez accéder à la fenêtre Threads pendant une session de débogage (sélectionnez Déboguer -> Fenêtres -> Threads lorsque le programme est en cours d'exécution). Comme vous pouvez le voir sur l'image suivante, vous pouvez rapidement identifier le thread que vous souhaitez diagnostiquer.

![[Figure 15.1.png|Débogger un thread avec Visual Studio]]

## Propriété `Priority`

Notez ensuite que le type `Thread` définit une propriété nommée `Priority`. ==Par défaut, tous les threads ont un niveau de priorité `Normal`==. Toutefois, **vous pouvez le modifier à tout moment pendant la durée de vie du thread à l'aide de la propriété `Priority` et de l'énumération `System.Threading.ThreadPriority` associée**, comme ceci :

```cs
public enum ThreadPriority
{
	Lowest,
	BelowNormal,
	Normal, // Valeur par défaut.
	AboveNormal,
	Highest
}
```

**Si vous attribuez à un thread une priorité différente de la valeur par défaut (`ThreadPriority.Normal`), sachez que vous n'aurez aucun contrôle direct sur le moment où le planificateur de threads bascule entre eux**. ==Le niveau de priorité d'un thread indique au runtime .NET Core l'importance de son activité==. Ainsi, *==un thread avec la valeur `ThreadPriority.Highest` ne se verra pas nécessairement attribuer la priorité la plus élevée==*.

De même, *==si le planificateur de threads est occupé par une tâche donnée==* (par exemple, la synchronisation d'un objet, le changement de thread ou le déplacement de threads), *==le niveau de priorité sera très probablement modifié en conséquence==*. Cependant, **toutes choses égales par ailleurs, le runtime .NET Core lira ces valeurs et indiquera au planificateur de threads comment répartir au mieux le temps d'exécution**. ***==Les threads ayant une priorité identique devraient chacun recevoir le même temps d'exécution pour effectuer leur travail.==***

***==Dans la plupart des cas, vous aurez rarement (voire jamais) besoin de modifier directement le niveau de priorité d'un thread==***. **==En théorie, il est possible d'augmenter le niveau de priorité d'un ensemble de threads, empêchant ainsi les threads de priorité inférieure de s'exécuter à leur niveau requis==** (faites donc preuve de prudence).

# Création manuelle de threads secondaires

Pour créer par programmation des threads supplémentaires afin d'exécuter une unité de travail, suivez ce processus prévisible en utilisant les types de l'espace de noms `System.Threading` :

1. Créez une méthode qui servira de point d'entrée pour le nouveau thread.
2. Créez un délégué `ParameterizedThreadStart` (ou `ThreadStart`) en passant l'adresse de la méthode définie à l'étape 1 au constructeur.
3. Créez un objet `Thread` en passant le délégué `ParameterizedThreadStart`/`ThreadStart` comme argument du constructeur.
4. Définissez les caractéristiques initiales du thread (`Name`, `Priority`, etc.).
5. Appelez la méthode `Thread.Start()`. Cela démarrera le thread à la méthode référencée par le délégué créé à l'étape 2 dès que possible.

Comme indiqué à l'étape 2, **vous pouvez utiliser deux types de délégués distincts pour « pointer » vers la méthode que le thread secondaire exécutera**. **==Le délégué `ThreadStart` peut pointer vers n'importe quelle méthode ne prenant aucun argument et ne retournant aucune valeur==**. Ce délégué peut s'avérer ***==utile lorsque la méthode est conçue pour s'exécuter simplement en arrière-plan sans interaction supplémentaire.==**

*==La limitation de `ThreadStart` est qu'il est impossible de lui transmettre des paramètres de traitement==*. Cependant, **le type de délégué `ParameterizedThreadStart` accepte un unique paramètre de type `System.Object`**. Étant donné que tout objet peut être représenté par un `System.Object`, **==vous pouvez transmettre un nombre quelconque de paramètres via une classe ou une structure personnalisée==**. ==Notez toutefois que les délégués `ThreadStart` et `ParameterizedThreadStart` ne peuvent pointer que vers des méthodes qui renvoient `void`.

>[!tip] Toutes ces étapes peuvent être écrite en une ou deux lignes:
>```cs
>Thread t = new Thread(() => MaMethode(message, nombre)).Start();
>```
>

## Utilisation du délégué `ThreadStart`

Pour illustrer le processus de création d'une application multithread (et démontrer son utilité), supposons que vous ayez un projet d'application console nommé *SimpleMultiThreadApp* qui permet à l'utilisateur de choisir si l'application s'exécutera sur un seul thread principal ou répartira sa charge de travail sur deux threads distincts.

**En supposant que vous ayez importé l'espace de noms `System.Threading`, la première étape consiste à définir une méthode pour exécuter le travail du thread secondaire** (éventuellement). Afin de nous concentrer sur le fonctionnement des programmes multithread, cette méthode affichera simplement une séquence de nombres dans la console, avec une pause d'environ deux secondes à chaque affichage. Voici la définition complète de la classe `Printer` :

```cs
namespace SimpleMutliThreadApp;

public class Printer
{
    public void PrintNumbers()
    {
        // Affiche les information du thread.
        Console.WriteLine(
            $"->{Thread.CurrentThread.Name} is executing {nameof(this.PrintNumbers)}"
        );

        // Affiche des nombres:
        Console.Write("Your numbers: ");
        for (int i = 0; i < 10; i++)
        {
            Console.Write($"{i}, ");
            Thread.Sleep(2000);
        }
        Console.WriteLine();
    }
}
```

Dans le fichier *Program.cs*, vous ajouterez des instructions de niveau supérieur pour inviter l'utilisateur à déterminer si un ou deux threads seront utilisés pour exécuter l'application. Si l'utilisateur choisit un seul thread, vous appellerez simplement la méthode `PrintNumbers()` dans le thread principal. En revanche, si l'utilisateur spécifie deux threads, vous créerez un délégué `ThreadStart` pointant vers `PrintNumbers()`, passerez ce délégué au constructeur d'un nouvel objet `Thread`, et appellerez `Start()` pour informer le runtime .NET Core que ce thread est prêt à être traité. Voici l'implémentation complète :

```cs
using SimpleMutliThreadApp;

Console.Title = "The Amazing Thread App";
Console.WriteLine("***** The Amazing Thread App *****\n");

Console.Write("Do you want [1] or [2] threads? ");

// code différent par rapport au livre, on convertit en entier
// et on jette la valeur de retour de la méthode TryParse().
_ = int.TryParse(Console.ReadLine(), out int threadCount);

// Nomme le thread actuel
Thread primaryThread = Thread.CurrentThread;
primaryThread.Name = "Primary";

// Affiche les information du thread.
Console.WriteLine($"-> {Thread.CurrentThread.Name} is executing Main()");

// Créer une classe de travail.
Printer p = new Printer();

switch (threadCount)
{
    case 1:
        p.PrintNumbers();
        break;
    case 2:
        // On crée maintenant un thread.
        Thread backgroundThread = new Thread(new ThreadStart(p.PrintNumbers));
        backgroundThread.Name = "Secondary";
        backgroundThread.Start();
        break;
    default:
        Console.WriteLine("I dont know what you want... You get 1 thread.");
        goto case 1;
}

// Effecture des tâches additionnelles.
Console.WriteLine("This is on the main thread, and we are finished.");
Console.ReadLine();
```

**Si vous exécutez ce programme avec un seul thread, vous constaterez que la sortie console finale n'affichera le message qu'une fois la séquence de nombres entièrement imprimée**. Comme vous faites une pause d'environ deux secondes après chaque nombre, l'expérience utilisateur sera loin d'être optimale. **En revanche, si vous utilisez deux threads, la boîte de message s'affiche instantanément, car un objet `Thread` unique est chargé d'imprimer les nombres dans la console.**

## Utilisation du délégué `ParameterizedThreadStart`

***==Rappelons que le délégué `ThreadStart` ne peut pointer que vers des méthodes ne renvoyant aucune valeur (`void`) et ne prenant aucun argument==***. Bien que cela puisse convenir dans certains cas, **si vous souhaitez transmettre des données à la méthode s'exécutant sur le thread secondaire, vous devrez utiliser le type de délégué `ParameterizedThreadStart`**. Pour illustrer cela, créez un nouveau projet d'application console nommé *AddWithThreads*. Sachant que `ParameterizedThreadStart` peut pointer vers n'importe quelle méthode prenant un paramètre `System.Object`, vous allez créer un type personnalisé contenant les nombres à additionner, comme ceci :

```cs
namespace AddWithThreads;

class AddParams
{
    public int a,
        b;

    public AddParams(int numb1, int numb2)
    {
        a = numb1;
        b = numb2;
    }
}
```

Ensuite, créez une méthode dans le fichier *Program.cs* qui prendra un paramètre `AddParams` et affichera la somme des deux nombres concernés, comme suit :

```cs
void Add(object data)
{
    if (data is AddParams ap)
    {
        Console.WriteLine(
            $"ID of thread in Add(): {Environment.CurrentManagedThreadId}"
        );

        Console.WriteLine($"{ap.a} + {ap.b} = {ap.a + ap.b}");
    }
}
```

Le code dans *Program.cs* est simple. Il suffit d'utiliser `ParameterizedThreadStart` au lieu de `ThreadStart`, comme ceci :

```cs
using AddWithThreads;

Console.Title = "Adding with Thread Objects";
Console.WriteLine("***** Adding with Thread Objects *****\n");

Console.WriteLine(
    $"ID of thread in Main(): {Environment.CurrentManagedThreadId}"
);

// Crée un objet AddParams à passer dans le second thread.
AddParams ap = new AddParams(10, 10);
Thread t = new Thread(new ParameterizedThreadStart(Add));
t.Start(ap);

// Force une attente pour permettre à l'autre thread de se terminer.
Thread.Sleep(5);
Console.ReadLine();
```

>[!warning] **Ne jamais utiliser `Sleep` pour synchroniser des threads.** C'est ce qu'on appelle un "anti-pattern".
>L'auteur introduit ce `Sleep` pour te montrer ses limites, avant de t'enseigner **`Join()`** (que nous avons vu ensemble) ou les **`AutoResetEvent`** plus loin dans le chapitre

## La classe `AutoResetEvent`

**Dans ces premiers exemples, il n'existe pas de méthode simple pour savoir quand le thread secondaire a terminé son exécution**. *==Dans le dernier exemple, la méthode `Sleep` a été appelée avec une durée arbitraire pour laisser le temps à l'autre thread de se terminer==*. **==Une méthode simple et sûre pour forcer un thread à attendre la fin d'un autre consiste à utiliser la classe `AutoResetEvent`==**. **Dans le thread qui doit attendre, créez une instance de cette classe et passez la valeur `false` au constructeur pour indiquer que vous n'avez pas encore été notifié**. **Ensuite, au moment où vous souhaitez attendre, appelez la méthode `WaitOne()`**. Voici la mise à jour de la classe *Program.cs*, qui effectuera précisément cette opération à l'aide d'une variable membre statique `AutoResetEvent` :

```cs
using AddWithThreads;

AutoResetEvent _waitHandle = new AutoResetEvent(false);

Console.Title = "Adding with Thread Objects";
Console.WriteLine("***** Adding with Thread Objects *****\n");

Console.WriteLine(
    $"ID of thread in Main(): {Environment.CurrentManagedThreadId}"
);

// Crée un objet AddParams à passer dans le second thread.
AddParams ap = new AddParams(10, 10);
Thread t = new Thread(new ParameterizedThreadStart(Add));
t.Start(ap);

// Attend ici jusqu'à ce qu'on soit notifié!
_waitHandle.WaitOne();

Console.ReadLine();
...
```

**Lorsque l'autre thread aura terminé son traitement, il appellera la méthode `Set()` sur la même instance du type `AutoResetEvent`.**

```cs
void Add(object data)
{
    if (data is AddParams ap)
    {
        Console.WriteLine(
            $"ID of thread in Add(): {Environment.CurrentManagedThreadId}"
        );

        Console.WriteLine($"{ap.a} + {ap.b} = {ap.a + ap.b}");

        // Dit aux autres thread qu'il a fini!
        _waitHandle.Set();
    }
}
```

## Threads de premier plan et threads d'arrière-plan

Maintenant que vous avez vu comment créer par programmation de nouveaux threads d'exécution à l'aide de l'espace de noms `System.Threading`, formulons la distinction entre les threads de premier plan et les threads d'arrière-plan :

- Les *threads de premier plans* (*Foreground threads*) ==peuvent empêcher l'arrêt de l'application en cours==. **Le runtime .NET Core n'arrêtera pas une application** (c'est-à-dire qu'**==il ne déchargera pas le domaine d'application hôte==**) **tant que tous les threads de premier plan n'auront pas terminé leur exécution.**

- Les *threads d'arrière-plan* (*Background threads*, aussi parfois appelés *threads démons*) ==sont considérés par le runtime .NET Core comme des chemins d'exécution jetables qui peuvent être ignorés à tout moment== (***==même s'ils sont en cours d'exécution==***). Ainsi, **si tous les threads de premier plan sont terminés, tous les threads d'arrière-plan sont automatiquement arrêtés lors du déchargement du domaine d'application**.

**Il est important de noter que les threads de premier plan et d'arrière-plan *ne sont pas* synonymes de threads principaux et de threads de travail**. **==Par défaut, chaque thread créé via la méthode `Thread.Start()` est automatiquement un thread de premier plan==**. Cela signifie que ==le domaine d'application ne sera pas déchargé tant que tous les threads d'exécution n'auront pas terminé leurs tâches==. **==Dans la plupart des cas, ce comportement est exactement celui souhaité.==**

**Supposons toutefois que vous souhaitiez appeler `Printer.PrintNumbers()` sur un thread secondaire qui doit se comporter comme un thread d'arrière-plan**. Là encore, ==cela signifie que la méthode pointée par le type `Thread`== (via le délégué `ThreadStart` ou `ParameterizedThreadStart`) ==doit pouvoir s'arrêter en toute sécurité dès que tous les threads de premier plan ont terminé leurs tâches==. **==Configurer un tel thread est aussi simple que de définir la propriété `IsBackground` sur `true`.==**

>[!tip]- La distinction entre **Foreground** (premier plan) et **Background** (arrière-plan) est **toujours techniquement exacte** dans .NET 9, mais sa pertinence pratique a beaucoup diminué.
>
>Pourquoi on s'en occupe moins ?
>
> **A. L'hégémonie du Thread Pool**  
>Aujourd'hui, on crée rarement des threads manuellement via `new Thread()`. On utilise `Task.Run()` ou le `ThreadPool`.
>- **Fait crucial** : Tous les threads provenant du pool de threads .NET sont **automatiquement des Background Threads**.
>- Conséquence : Avec le code moderne, votre application ne reste jamais "bloquée" par une tâche de fond oubliée.
>
> **B. L'abandon de la gestion manuelle**  
>Dans le passé, on passait un thread en `IsBackground = true` pour s'assurer que l'application se ferme proprement. Aujourd'hui, on préfère utiliser un **`CancellationToken`**. Au lieu de laisser l'OS tuer le thread brutalement, on demande gentiment au thread de s'arrêter lui-même avant la fermeture.

Créez une nouvelle application console nommée `BackgroundThreads` et copiez le fichier `Printer.cs` dans le nouveau projet. Mettez à jour l'espace de noms de la classe `Printer` en `BackgroundThreads`, comme ceci :

```cs
namespace Backgroundthreads;

public class Printer
{
	...
}
```

Mettez à jour le fichier *Program.cs* pour qu'il corresponde à ce qui suit :

```cs
using Backgroundthreads;

Console.Title = "Background Threads";
Console.WriteLine("***** Background Threads *****\n");

Printer p = new Printer();
Thread bgroundThread = new Thread(new ThreadStart(p.PrintNumbers));

// Ceci est maintenant un thread en arrière plan
bgroundThread.IsBackground = true;
bgroundThread.Start();
```

***==Notez que ce code n'appelle pas `Console.ReadLine()` pour forcer l'affichage de la console jusqu'à ce que vous appuyiez sur la touche Entrée==***. Par conséquent, **lorsque vous exécutez l'application, elle s'arrête immédiatement car l'objet `Thread` a été configuré comme un thread d'arrière-plan**. ==Étant donné que le point d'entrée d'une application== (instructions supérieurs comme ici ou méthode `Main()`) ==déclenche la création du thread principal de premier plan==, ***==dès que la logique du point d'entrée est terminée, le domaine d'application est déchargé avant que le thread secondaire puisse terminer son travail.==***

Cependant, **si vous commentez la ligne qui définit la propriété `IsBackground`, vous constaterez que chaque nombre s'affiche dans la console, car tous les threads de premier plan doivent terminer leur travail avant que le domaine d'application ne soit déchargé du processus hôte.**

**==Dans la plupart des cas, configurer un thread pour qu'il s'exécute en arrière-plan peut être utile lorsque le thread de travail en question effectue une tâche non critique qui n'est plus nécessaire une fois la tâche principale du programme terminée==**. Par exemple, vous pourriez créer une application qui interroge un serveur de messagerie toutes les quelques minutes pour vérifier la présence de nouveaux courriels, met à jour les conditions météorologiques actuelles ou effectue toute autre tâche non critique.

# Le problème de la concurrence

**Lorsque vous développez des applications multithread, votre programme doit garantir que toute donnée partagée est protégée contre la possibilité que plusieurs threads modifient sa valeur**. ==Étant donné que tous les threads d'un AppDomain ont un accès concurrent aux données partagées de l'application==, **imaginez ce qui pourrait se produire si plusieurs threads accédaient au même point de données**. ==Comme le planificateur de threads force les threads à suspendre leur travail de manière aléatoire==, **que se passe-t-il si le thread A est interrompu avant d'avoir terminé son travail ? Le thread B lit alors des données instables.**

Pour illustrer le problème de la concurrence, créons un autre projet d'application console nommé *MultiThreadedPrinting*. Cette application utilisera à nouveau la classe `Printer` créée précédemment, mais cette fois, la méthode `PrintNumbers()` forcera le thread courant à s'interrompre pendant une durée aléatoire.

```cs
namespace MultiThreadedPrinting;

public class Printer
{
    public void PrintNumbers()
    {
        // Affiche les information du thread.
        Console.WriteLine(
            $"->{Thread.CurrentThread.Name} is executing {nameof(this.PrintNumbers)}()"
        );

        // Affiche des nombres:
        for (int i = 0; i < 10; i++)
        {
            // Mets le thread en veille pendant une durée aléatoire.
            Random r = new Random();
            Thread.Sleep(1000 * r.Next(5));
            Console.Write($"{i}, ");
        }
        Console.WriteLine();
    }
}
```

**Le code appelant est chargé de créer un tableau de dix objets `Thread`** (nommés de manière unique), ==chacun d'eux effectuant des appels sur la même instance de l'objet `Printer` comme suit :==

```cs
using MultiThreadedPrinting;

Console.Title = "Synchronizing Threads";
Console.WriteLine("***** Synchronizing Threads *****\n");

Printer p = new Printer();

// Crée 10 threads qui pointeront tous vers la même méthode
// et vers le même objet.
Thread[] threads = new Thread[10];
for (int i = 0; i < threads.Length; i++)
{
    threads[i] = new Thread(new ThreadStart(p.PrintNumbers))
    {
        Name = $"Worker thread #{i}",
    };
}

// Maintenant, on démarre chaque threads
foreach (Thread t in threads)
{
    t.Start();
}
Console.ReadLine();
```

Avant d'examiner quelques tests, ==récapitulons le problème==. **Le thread principal de ce domaine d'application crée dix threads de travail secondaires**. **==Chaque thread de travail est chargé d'appeler la méthode `PrintNumbers()` sur la même instance de `Printer`==**. *==Étant donné que vous n'avez pris aucune précaution pour verrouiller les ressources partagées de cet objet (la console), il est fort probable que le thread courant soit interrompu avant que la méthode `PrintNumbers()` n'ait pu afficher tous les résultats==*. **Comme vous ne savez pas exactement quand** (ni même si) **cela se produira, vous obtiendrez forcément des résultats imprévisibles**. Par exemple, vous pourriez obtenir le résultat affiché ici :

```
***** Synchronizing Threads *****

->Worker thread #0 is executing PrintNumbers()
->Worker thread #1 is executing PrintNumbers()
->Worker thread #3 is executing PrintNumbers()
->Worker thread #4 is executing PrintNumbers()
->Worker thread #2 is executing PrintNumbers()
->Worker thread #5 is executing PrintNumbers()
->Worker thread #6 is executing PrintNumbers()
->Worker thread #7 is executing PrintNumbers()
->Worker thread #8 is executing PrintNumbers()
->Worker thread #9 is executing PrintNumbers()
0, 0, 0, 0, 0, 1, 0, 1, 1, 1, 0, 0, 0, 0, 1, 2, 3, 1, 1, 1, 1, 2, 4, 5, 1, 2, 3, 4, 2, 2, 2, 2, 3, 2, 2, 3, 6, 7, 8, 4, 3, 3, 4, 2, 5, 6, 7, 3, 4, 3, 3, 9,
3, 4, 4, 5, 6, 5, 4, 5, 4, 5, 8, 5, 4, 6, 6, 7, 8, 7, 5, 5, 6, 9,
5, 6, 6, 7, 6, 7, 6, 9,
7, 7, 8, 8, 8, 8, 7, 9,
8, 9,
7, 8, 9,
9,
9,
8, 9,
9,
```

**Exécutez maintenant l'application plusieurs fois et examinez le résultat. Il sera très probablement différent à chaque fois.**

>[!note] 
>Si vous ne parvenez pas à générer des résultats imprévisibles, augmentez le nombre de threads de 10 à 100 (par exemple) ou ajoutez un appel à `Thread.Sleep()` dans votre programme. Vous finirez par rencontrer le problème de concurrence.

Il y a manifestement des problèmes. **Alors que chaque thread demande à l'objet `Printer` d'afficher les données numériques, le planificateur de threads permute allègrement les threads en arrière-plan**. *==Il en résulte une sortie incohérente==*. ***==Vous avez besoin d'un moyen d'imposer par programmation un accès synchronisé aux ressources partagées==***. **==Comme vous vous en doutez, l'espace de noms `System.Threading` fournit plusieurs types dédiés à la synchronisation==**. **Le langage de programmation C# fournit également un mot-clé permettant précisément de synchroniser les données partagées dans les applications multithread.**

## Synchronisation avec le mot-clé `lock` en C#

**La première technique permettant de synchroniser l'accès aux ressources partagées est le mot-clé `lock` en C#**. **==Ce mot-clé permet de définir un ensemble d'instructions qui doivent être synchronisées entre les threads==**. Ainsi, ==les threads entrants ne peuvent pas interrompre le thread courant et l'empêcher de terminer son exécution==. **Le mot-clé `lock` requiert la spécification d'un *jeton*** (voir note suivante) **qu'un thread doit acquérir pour entrer dans la portée verrouillée**. Pour verrouiller une méthode privée au niveau de l'instance, il suffit de passer une référence au type courant, comme suit :

>[!tip]- Rappel 
>Comme rabâché dans tout le chapitre, le mot clé `lock`, qui ne supportait alors que des jetons de type `System.Object`, à été amélioré. Depuis .NET 9, il permet l'utilisation d'un tout nouveau type de donnée `System.Threading.Lock` comme jeton.
>
>Ce nouveau type, en étant utilisé comme jeton dans l'instruction `lock`, permet une amélioration des performances considérables. 

```cs
private void SomePrivateMethod()
{
	// Utiliser l'objet courant comme jeton de thread.
	lock(this)
	{
		// Tout le code dans cette portée est thread-safe.
	}
}
```

**Toutefois, si vous verrouillez une portion de code au sein d'un membre *public*, il est plus sûr** (**==et constitue une bonne pratique==**) **de déclarer une variable membre d'objet privée servant de jeton de verrouillage, comme ceci** :

```cs
public class Printer
{
	// Verrouiller le jeton.
	// AVANT C# 13
	//private object threadLock = new object();
	// Maintenant:
    Lock threadLock = new Lock();

	
	public void PrintNumbers()
	{
		// Utiliser le jeton de verrouillage.
		lock (threadLock)
		{
			...
		}
	}
}
```

Dans tous les cas, **si vous examinez la méthode `PrintNumbers()`, vous constaterez que la ressource partagée à laquelle les threads tentent d'accéder est la fenêtre de la console**. **==Limitez toutes les interactions avec le type Console à une portée de verrouillage==**, comme suit :

```cs
public void PrintNumbers()
{
    // Utilisation de l'objet privé comme jeton de vérouillage
    // Ici, l'objet est de type Lock (nouveau C# 13)
    lock (threadLock)
    {
        // Affiche les information du thread.
        Console.WriteLine(
            $"->{Thread.CurrentThread.Name} is executing {nameof(this.PrintNumbers)}()"
        );
        // Affiche des nombres:
        Console.Write("Your numbers: ");
        for (int i = 0; i < 10; i++)
        {
            // Mets le thread en veille pendant une durée aléatoire.
            Random r = new Random();
            Thread.Sleep(1000 * r.Next(5));
            Console.Write($"{i}, ");
        }
        Console.WriteLine();
    }
}
```

Ce faisant, vous avez effectivement conçu une méthode permettant au thread courant de terminer sa tâche. Une fois qu'un thread entre dans un contexte de verrouillage, le jeton de verrouillage (ici, une référence à l'objet courant) est inaccessible aux autres threads jusqu'à ce que le verrou soit libéré après la sortie du contexte de verrouillage. Ainsi, si le thread A a obtenu le jeton de verrouillage, les autres threads ne peuvent entrer dans aucun contexte utilisant le même jeton de verrouillage tant que le thread A n'a pas libéré le jeton de verrouillage.

```
*****Synchronizing Threads *****

-> Worker thread #0 is executing PrintNumbers()
Your numbers: 0, 1, 2, 3, 4, 5, 6, 7, 8, 9,
-> Worker thread #1 is executing PrintNumbers()
Your numbers: 0, 1, 2, 3, 4, 5, 6, 7, 8, 9,
-> Worker thread #3 is executing PrintNumbers()
Your numbers: 0, 1, 2, 3, 4, 5, 6, 7, 8, 9,
-> Worker thread #2 is executing PrintNumbers()
Your numbers: 0, 1, 2, 3, 4, 5, 6, 7, 8, 9,
-> Worker thread #4 is executing PrintNumbers()
Your numbers: 0, 1, 2, 3, 4, 5, 6, 7, 8, 9,
-> Worker thread #5 is executing PrintNumbers()
Your numbers: 0, 1, 2, 3, 4, 5, 6, 7, 8, 9,
-> Worker thread #7 is executing PrintNumbers()
Your numbers: 0, 1, 2, 3, 4, 5, 6, 7, 8, 9,
-> Worker thread #6 is executing PrintNumbers()
Your numbers: 0, 1, 2, 3, 4, 5, 6, 7, 8, 9,
-> Worker thread #8 is executing PrintNumbers()
Your numbers: 0, 1, 2, 3, 4, 5, 6, 7, 8, 9,
-> Worker thread #9 is executing PrintNumbers()
Your numbers: 0, 1, 2, 3, 4, 5, 6, 7, 8, 9,
```

## Synchronisation à l'aide du type `System.Threading.Monitor`

**L'instruction `lock` en C# est une notation abrégée pour interagir avec la classe `System.Threading.Monitor`** (***==voir note suivante==***). ==Une fois traitée par le compilateur C#, une portée de verrouillage se traduit par ce qui suit== (que vous pouvez vérifier à l'aide de *ilasm.exe* / *ilspycmd*) :

**En utilisant un `System.Object` comme jeton**:

```cs
public void PrintNumbers()
{
	Monitor.Enter(threadLock);
	try
	{
		// Affiche les informations du thread
		Console.WriteLine("-> {0} is executing PrintNumbers()", Thread.CurrentThread.Name);
		
		// Affiche les nombres.
		Console.Write("Your numbers: ");
		for (int i = 0; i < 10; i++)
		{
			Random r = new Random();
			Thread.Sleep(1000 * r.Next(5));
			Console.Write("{0}, ", i);
		}
		Console.WriteLine();
	}
	finally
	{
		Monitor.Exit(threadLock);
	}
}
```

>[!tip]- **En utilisant un `System.Threading.Lock` comme jeton**:
>
>```cs
>public void PrintNumbers()
>{
>	using (threadLock.EnterScope())
>	{
>		Console.WriteLine($"->{Thread.CurrentThread.Name} is executing {"PrintNumbers"}()");
>		for (int i = 0; i < 10; i++)
>		{
>			Random random = new Random();
>			Thread.Sleep(1000 * random.Next(5));
>			Console.Write($"{i}, ");
>		}
>		Console.WriteLine();
>	}
>}
>```

Tout d'abord, **notez que la méthode `Monitor.Enter()` est la destinataire finale du jeton de thread que vous spécifiez comme argument du mot-clé `lock`**. Ensuite, **==tout le code exécuté dans la portée de `lock` est placé dans un bloc `try`==**. **Le bloc `finally` correspondant garantit la libération du jeton de thread (via la méthode `Monitor.Exit()`), quelle que soit l'exception d'exécution éventuelle**. ==Si vous modifiiez le programme `MultiThreadPrinting` pour utiliser directement le type `Monitor`== (comme illustré ci-dessus), ==vous constateriez que le résultat est identique.==

Maintenant, **étant donné que le mot-clé `lock` semble nécessiter moins de code que l'utilisation explicite du type `System.Threading.Monitor`, vous vous demandez peut-être quels sont les avantages d'utiliser directement le type `Monitor`**. ==La réponse courte est : le contrôle==. **==Si vous utilisez le type `Monitor`, vous pouvez demander au thread actif d'attendre pendant une certaine durée==** (via la méthode statique `Monitor.Wait()`), **==informer les threads en attente de la fin de l'exécution du thread courant==** (via les méthodes statiques `Monitor.Pulse()` et `Monitor.PulseAll()`), etc. 

**==Comme vous pouvez l'imaginer, dans la plupart des cas, le mot-clé `lock` de C# conviendra parfaitement. Toutefois, si vous souhaitez en savoir plus sur les autres membres de la classe `Monitor`, consultez la [documentation](https://learn.microsoft.com/en-us/dotnet/api/system.threading.monitor?view=net-10.0) .NET Core.==**

## Synchronisation à l'aide du type `System.Threading.Interlocked`

**Bien que cela puisse paraître surprenant, tant qu'on n'a pas examiné le code CIL sous-jacent, les affectations et les opérations arithmétiques simples *ne sont pas atomiques***. **==C'est pourquoi l'espace de noms `System.Threading` fournit un type qui permet d'opérer atomiquement sur un point de données unique, avec une demande mémoire moindre qu'avec le type `Monitor`==**. La classe `Interlocked` définit les membres statiques clés présentés dans le [[#Tableau 15-5 Sélection de membre statique du type `System.Threading.Interlocked`|Tableau 15-5]].

##### Tableau 15-5: Sélection de membre statique du type `System.Threading.Interlocked`

| Membre              | Description                                                                                                              |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| `CompareExchange()` | Teste en toute sécurité l'égalité de deux valeurs et, si elles sont égales, échange l'une des valeurs avec la troisième. |
| `Decrement()`       | Décrémente une valeur de 1 en toute sécurité                                                                             |
| `Exchange()`        | Échange deux valeurs en toute sécurité                                                                                   |
| `Increment()`       | Incrémente une valeur de 1 en toute sécurité                                                                             |

Bien que cela ne paraisse pas évident au premier abord, **==la modification atomique d'une seule valeur est assez courante dans un environnement multithread==**. ==Supposons que vous ayez un code qui incrémente une variable membre entière nommée `intVal`==. Plutôt que d'écrire du code de synchronisation comme le suivant :

```cs
int intVal = 5;
object myLockToken = new();
lock(myLockToken)
{
	intVal++;
}
```

**Vous pouvez simplifier votre code grâce à la méthode statique `Interlocked.Increment()`**. **==Il suffit de passer la variable à incrémenter par référence==**. ==Notez que la méthode `Increment()` modifie non seulement la valeur du paramètre d'entrée, mais renvoie également la nouvelle valeur.==

```cs
intVal = Interlocked.Increment(ref intVal);
```

**Outre les méthodes `Increment()` et `Decrement()`, le type `Interlocked` permet d'affecter atomiquement des données numériques et des objets**. Par exemple, pour affecter la valeur $83$ à une variable membre, il est possible d'éviter l'utilisation d'une instruction de verrouillage explicite (ou d'une logique `Monitor` explicite) et d'utiliser la méthode `Interlocked.Exchange()`, comme suit :

```cs
var myInt = 27;
Interlocked.Exchange(ref myInt, 83);
Console.WriteLine(myInt);
```

**Enfin, si vous souhaitez tester l'égalité de deux valeurs et modifier le point de comparaison de manière thread-safe, vous pouvez utiliser la méthode `Interlocked.CompareExchange()` comme suit :**

```cs
// Si la valeur de i est actuellement 83, change myInt à 99.
Interlocked.CompareExchange(ref myInt, 99, 83);
```

# Utiliser les callbacks de la classe  `Timer`

==De nombreuses applications nécessitent l'appel d'une méthode spécifique à intervalles réguliers==. Par exemple, **==une application peut avoir besoin d'afficher l'heure actuelle dans une barre d'état via une fonction auxiliaire==**. Autre exemple : **==une application peut vouloir appeler une fonction auxiliaire périodiquement pour effectuer des tâches de fond non critiques, telles que la vérification des nouveaux messages==**. **Dans ce genre de situations, vous pouvez utiliser le type `System.Threading.Timer` avec un délégué associé nommé `TimerCallback`.**

À titre d'exemple, supposons que vous ayez un projet d'application console (*TimerApp*) qui affichera l'heure actuelle chaque seconde jusqu'à ce que l'utilisateur appuie sur une touche pour fermer l'application. La première étape consiste à écrire la méthode qui sera appelée par le type `Timer`.

```cs
Console.Title = "Working with Timer type";
Console.WriteLine("***** Working with Timer type *****\n");

static void PrintTime(object state)
{
    // Effectue la même chose que l'appel à ToLongTimeString()
    // un t minuscule ferait la méme chose que ToShortTimeString().
    Console.WriteLine($"Time is: {DateTime.Now:T}");
}
```

**Notez que la méthode `PrintTime()` possède un seul paramètre de type `System.Object` et ne renvoie `void`**. ***==Ce paramètre est obligatoire, car le délégué `TimerCallback` ne peut appeler que des méthodes correspondant à cette signature==***. **La valeur passée à la cible de votre délégué `TimerCallback` peut être de n'importe quel type d'objet** (dans le cas de l'exemple d'e-mail, ce paramètre pourrait représenter le nom du serveur Microsoft Exchange avec lequel interagir pendant le processus). **==Notez également que, puisque ce paramètre est bien un `System.Object`, vous pouvez passer plusieurs arguments à l'aide d'un `System.Array` ou d'une classe/structure personnalisée.==**

**L'étape suivante consiste à configurer une instance du délégué `TimerCallback` et à la passer à l'objet `Timer`**. ==Outre la configuration d'un délégué `TimerCallback`, le constructeur `Timer` vous permet de spécifier les informations de paramètre facultatives à passer à la cible du délégué== (définie comme un `System.Object`), ==l'intervalle d'interrogation de la méthode et le délai d'attente== (en millisecondes) ==avant le premier appel==. Voici un exemple :

```cs
Console.Title = "Working with Timer type";
Console.WriteLine("***** Working with Timer type *****\n");

// Crée le délégué pour le type Timer.
TimerCallback timeCB = new TimerCallback(PrintTime);

// Établit les paramètre de l'objet.
Timer t = new Timer(
    timeCB, // L'objet délégué TimerCallback.
    null,   // Toute information à transmettre à la méthode appelée (ici, aucune).
    0,      // Quantité de temps à attendre avant de commencer (en ms).
    1000.   // Intervale de temp entre les appels (en ms).
);

Console.WriteLine("Hit enter key to terminate...");
Console.ReadLine();
static void PrintTime(object state)
{
    // Effectue la même chose que l'appel à ToLongTimeString()
    // un t minuscule ferait la méme chose que ToShortTimeString().
    Console.WriteLine($"Time is: {DateTime.Now:T}");
}
```

Dans ce cas, la méthode `PrintTime()` sera appelée environ toutes les secondes et ne lui transmettra aucune information supplémentaire. Voici le résultat :

```
***** Working with Timer type *****

Hit enter key to terminate...
Time is: 7:16:25 PM
Time is: 7:16:26 PM
Time is: 7:16:27 PM
Time is: 7:16:28 PM
Time is: 7:16:29 PM
Time is: 7:16:30 PM
Time is: 7:16:31 PM
Time is: 7:16:32 PM
Time is: 7:16:33 PM
Time is: 7:16:34 PM
Time is: 7:16:35 PM
Time is: 7:16:36 PM
Time is: 7:16:37 PM
Time is: 7:16:38 PM
Time is: 7:16:39 PM
Time is: 7:16:40 PM
```

>[!info] La culture utilisé sur mon mac est `en_US`. .NET récupère cette information depuis le terminal, il est donc modifiable et par programmation, et par les paramètres du terminal.

**Si vous souhaitez transmettre des informations à l'intention de la cible du délégué, il vous suffit de remplacer la valeur nulle du deuxième paramètre du constructeur par les informations appropriées**, comme ceci :

```cs
// Établit les paramètre de l'objet.
Timer t = new Timer(timeCB, "Hello From .NET 10.0", 0, 1000);
```

Vous pouvez alors obtenir les données entrantes comme suit :

```cs
static void PrintTime(object state)
{
    // Effectue la même chose que l'appel à ToLongTimeString()
    // un t minuscule ferait la méme chose que ToShortTimeString().
    Console.WriteLine(
        $"Time is: {DateTime.Now:T}, Param is: {state.ToString()}"
    );
}
```

## Utilisation d'une défausse autonome (Nouveauté C# 7.0)

**Dans l'exemple précédent, la variable `Timer` n'est utilisée dans aucun chemin d'exécution**; ==elle peut donc être remplacée par une défausse==, comme suit :

```cs
// Établit les paramètre de l'objet.
_ = new Timer(
    timeCB, // L'objet délégué TimerCallback.
    "Bonjour depuis C# 10.0", // Informations à transmettre à la 
            //méthode appelée (null si aucune information).
    0,      // Délai d'attente avant le démarrage
            // (en millisecondes).
    1000    // Intervalle de temps entre les appels (en millisecondes)
); 
```

# Comprendre le Thread Pool

==Le prochain sujet relatif aux threads que nous aborderons dans ce chapitre est le rôle du pool de threads d'exécution==. ***==La création d'un nouveau thread a un coût==*** ; **par conséquent, pour des raisons d'efficacité, le pool de threads conserve les threads créés (mais inactifs) jusqu'à ce qu'ils soient nécessaires**. **==Pour vous permettre d'interagir avec ce pool de threads en attente, l'espace de noms `System.Threading` fournit la classe `ThreadPool`.==**

**Si vous souhaitez mettre en file d'attente un appel de méthode pour qu'il soit traité par un thread de travail du pool, vous pouvez utiliser la méthode `ThreadPool.QueueUserWorkItem()`**. ==Cette méthode a été surchargée pour vous permettre de spécifier un `System.Object` optionnel pour les données d'état personnalisées, en plus d'une instance du délégué `WaitCallback`.==

```cs
public static class ThreadPool
{
	...
	public static bool QueueUserWorkItem(WaitCallback callBack);
	public static bool QueueUserWorkItem(WaitCallback callBack, object state);
}
```

**Le délégué `WaitCallback` peut pointer vers n'importe quelle méthode prenant un `System.Object` comme unique paramètre** (représentant les données d'état facultatives) **et ne retournant aucune valeur(`void`)**. Notez que ==si vous ne fournissez pas de `System.Object`, lors de l'appel à `QueueUserWorkItem()`, le runtime .NET Core transmet automatiquement la valeur `null`==. Pour illustrer la mise en file d'attente des méthodes utilisées par le pool de threads du runtime .NET Core, considérez le programme suivant (dans une application console nommée *ThreadPoolApp*), qui utilise à nouveau le type `Printer` (veillez à mettre à jour l'espace de noms en `ThreadPoolApp`). **Dans ce cas, cependant, vous ne créez pas manuellement un tableau d'objets `Thread`; vous affectez plutôt des membres du pool à la méthode `PrintNumbers()`**.

```cs
using ThreadPoolApp;

Console.Title = "Fun with the .NET Core Runtime Thread Pool";
Console.WriteLine("***** Fun with the .NET Core Runtime Thread Pool *****\n");

Console.WriteLine(
    $"Main thread started. ThreadID = {Environment.CurrentManagedThreadId}"
);

Printer p = new Printer();

WaitCallback workItem = new WaitCallback(PrintTheNumbers);

// Met en file d'attente la méthode 10 fois
for (int i = 0; i < 10; i++)
{
    ThreadPool.QueueUserWorkItem(workItem, p);
}
Console.WriteLine("All tasks queued");
Console.ReadLine();

static void PrintTheNumbers(object state)
{
    Printer task = (Printer)state;
    task.PrintNumbers();
}
```

==À ce stade, vous vous demandez peut-être s'il serait avantageux d'utiliser le pool de threads géré par le runtime .NET Core, plutôt que de créer explicitement des objets `Thread`==. **==Voici quelques avantages de l'utilisation du pool de threads :==**

- Le pool de threads gère efficacement les threads en minimisant le nombre de threads qui doivent être créés, démarrés et arrêtés.
- En utilisant le pool de threads, vous pouvez vous concentrer sur votre problème métier plutôt que sur l'infrastructure de threads de l'application.

*==Cependant, la gestion manuelle des threads est parfois préférable. Voici un exemple :==*

- Si vous avez besoin de threads de premier plan ou si vous devez définir la priorité d'un thread. Les threads du pool sont toujours des threads d'arrière-plan avec une priorité par défaut (`ThreadPriority.Normal`).
- Si vous avez besoin d'un thread avec une identité fixe pour l'interrompre, le suspendre ou le retrouver par son nom.

Ceci conclut votre exploration de l'espace de noms `System.Threading`. Bien entendu, **la compréhension des sujets présentés jusqu'ici dans ce chapitre** (***==en particulier lors de votre analyse des problèmes de concurrence==***) **sera extrêmement précieuse lors de la création d'une application multithread**. Fort de ces bases, **==vous allez maintenant vous intéresser à plusieurs nouveaux sujets liés aux threads, introduits dans .NET 4.0 et repris dans .NET Core==**. Pour commencer, vous examinerez le rôle d'un modèle de threads alternatif : la bibliothèque parallèle de tâches (TPL).

# Programmation parallèle avec la bibliothèque parallèle de tâches

>[!danger]- **L'auteur utilise un projet WPF pour illustrer cette partie du chapitre** (Gemini)
> Comme ce n'est pas exécutable sur macOS, Je vais utilisé **AvaloniaUI**. L'installation est très simple ([disponible ici](https://docs.avaloniaui.net/docs/get-started/install-avalonia)).
>
>AvaloniaUI est le successeur spirituel de WPF pour le monde multiplateforme (Mac/Linux). Voici ce qui change concrètement dans apprentissage :
>
> ### XAML : Le langage de description
>
>- **WPF** : Utilise des balises très spécifiques à Windows.
>- **Avalonia** : Utilise un dialecte XAML (AXAML), presque identique à 90 %.
>- **Ce qui change** : Dans les example de code, si vous voyez `<StackPanel>` ou `<Button>` dans le livre, cela fonctionnera exactement de la même manière dans Avalonia. Les bases du design sont les mêmes.
>
> ### Le "Code-behind" (C#)
>
>C'est ici que l'auteur va introduire des concepts comme les **Événements de clic**.
>
>- **WPF** : `public void MyButton_Click(object sender, RoutedEventArgs e)`
>- **Avalonia** : La signature est identique. Vous ne serez pas dépaysé.
>
> ### La gestion du Thread UI (Le point critique)
>
>Cette section va nous apprendre qu'un thread secondaire **n'a pas le droit** de modifier directement un élément de l'interface (comme changer le texte d'un label).
>
>- **WPF** : Utilise un objet nommé `Dispatcher`.
>- **Avalonia** : Utilise un objet nommé `Dispatcher` également (dans le namespace `Avalonia.Threading`).
>- **La logique** : La méthode `Dispatcher.UIThread.Post(...)` d'Avalonia remplace le `Dispatcher.Invoke(...)` de WPF. Le concept reste le même : "Envoyer une instruction au thread principal".
>
>---
>*Le livre va demander de modifier des propriétés que l'on ne trouverera pas au même endroit dans les namespaces. Mais la **logique de la TPL** (utiliser `Task.Run` pour ne pas geler la fenêtre) est rigoureusement identique.*

À ce stade du chapitre, ==vous avez examiné les objets de l'espace de noms `System.Threading` qui vous permettent de créer des logiciels multi-threadés==. **Depuis la sortie de .NET 4.0, Microsoft a introduit une nouvelle approche du développement d'applications multi-threadées à l'aide d'une bibliothèque de programmation parallèle appelée** *bibliothèque parallèle de tâches* (TPL). **==Grâce aux types `System.Threading.Tasks`, vous pouvez créer du code parallèle précis et évolutif sans avoir à manipuler directement les threads ou le pool de threads.==**

**Cela ne signifie pas pour autant que vous n'utiliserez pas les types `System.Threading` lorsque vous utiliserez la TPL**. **==Les deux bibliothèques de gestion des threads peuvent fonctionner ensemble de manière tout à fait naturelle==**. C'est d'autant plus vrai que **l'espace de noms `System.Threading` fournit toujours la plupart des primitives de synchronisation que vous avez examinées précédemment** (`Monitor`, `Interlocked`, etc.). Cependant, ***==vous constaterez probablement que vous préférez travailler avec la bibliothèque TPL plutôt qu'avec l'espace de noms `System.Threading` d'origine, étant donné que le même ensemble de tâches peut être effectué de manière plus simple.==***

## L’espace de noms `System.Threading.Tasks`

***==Collectivement, les types `System.Threading.Tasks` sont appelés la bibliothèque parallèle de tâches (TPL).==*** **La TPL répartit automatiquement et dynamiquement la charge de travail de votre application sur les processeurs disponibles, à l’aide du pool de threads d’exécution**. **==La TPL gère le partitionnement du travail, la planification des threads, la gestion de l’état et d’autres détails de bas niveau==**. **Vous pouvez ainsi optimiser les performances de vos applications .NET Core tout en vous affranchissant de la complexité liée à la gestion directe des threads.**

## Le rôle de la classe `Parallel`

**Une classe clé de la bibliothèque TPL est `System.Threading.Tasks.Parallel`**. ***==Cette classe contient des méthodes permettant de parcourir une collection de données (plus précisément, un objet implémentant `IEnumerable<T>`) en parallèle, principalement grâce à deux méthodes statiques principales : `Parallel.For()` et `Parallel.ForEach()`, chacune définissant de nombreuses surcharges.==***

Ces méthodes vous permettent de créer un ensemble d'instructions de code qui seront traitées en parallèle. **==En principe, ces instructions correspondent à la même logique que celle utilisée dans une boucle classique==** (via les mots-clés C# `for` ou `foreach`). **L'avantage est que la classe `Parallel` alloue des threads du pool de threads (et gère la concurrence) pour vous.**

**Les deux méthodes nécessitent la spécification d'un conteneur compatible avec `IEnumerable` ou `IEnumerable<T>` contenant les données à traiter en parallèle**. Ce conteneur peut être un simple tableau, une collection non générique (comme `ArrayList`), une collection générique (comme `List<T>`) ou les résultats d'une requête LINQ.

De plus, **vous devrez utiliser les délégués `System.Func<T>` et `System.Action<T>` pour spécifier la méthode cible qui sera appelée pour traiter les données**. Vous avez déjà rencontré le délégué `Func<T>` au [[Chapitre 13#Création d'expressions de requête à l'aide du type `Enumerable` et des délégués bruts|Chapitre 13]], lors de votre étude de LINQ to Objects. ==Rappelons que `Func<T>` représente une méthode qui peut avoir une valeur de retour donnée et un nombre variable d'arguments==. ==Le délégué `Action<T>` est similaire à `Func<T>`, en ce sens qu'il permet de pointer vers une méthode prenant un certain nombre de paramètres. Cependant, `Action<T>` spécifie une méthode qui ne peut retourner que `void`.==

**Bien qu'il soit possible d'appeler les méthodes `Parallel.For()` et `Parallel.ForEach()` et de leur passer un objet délégué `Func<T>` ou `Action<T>` fortement typé, vous pouvez simplifier votre programmation en utilisant une méthode anonyme C# appropriée, ou une expression lambda.**

## Parallélisme de données avec la classe `Parallel`

**La première utilisation de la bibliothèque TPL consiste à effectuer du parallélisme de données**. En termes simples, **==ce terme désigne l'itération sur un tableau ou une collection de manière parallèle à l'aide des méthodes `Parallel.For()` ou `Parallel.ForEach()`==**. Supposons que vous deviez effectuer des opérations d'entrée/sortie de fichiers gourmandes en ressources. Plus précisément, vous devez charger un grand nombre de fichiers *.jpg*/*.png* en mémoire, les retourner et enregistrer les données d'image modifiées dans un nouvel emplacement. 

Dans cet exemple, vous verrez comment réaliser cette même tâche à l'aide d'une interface utilisateur graphique. Vous pourrez ainsi examiner l'utilisation des « délégués anonymes » pour permettre à des threads secondaires de mettre à jour le thread principal de l'interface utilisateur (également appelé thread d'interface utilisateur).

>[!note] 
>Lors du développement d'une application d'interface utilisateur graphique (GUI) multithread, les threads secondaires, ne peuvent jamais accéder directement aux contrôles de l'interface utilisateur. En effet, les contrôles (boutons, zones de texte, étiquettes, barres de progression, etc.) sont liés au thread qui les a créés. L'exemple suivant, illustre une méthode permettant aux threads secondaires d'accéder aux éléments d'interface utilisateur de manière thread-safe. Vous découvrirez une approche plus simplifiée en examinant les mots-clés `async` et `await` de C#.

#### WPF

Pour illustrer cela, créez une nouvelle application Windows Presentation Foundation (le modèle est abrégé en Application WPF (.NET Core)) nommée *DataParallelismWithForEach*. Pour créer le projet à l'aide de l'interface de ligne de commande et l'ajouter à la solution du chapitre, saisissez la commande suivante :

```
dotnet new wpf -lang c# -n DataParallelismWithForEach -o .\DataParallelismWithForEach -f net6.0
dotnet sln .\Chapter15_AllProjects.sln add .\DataParallelismWithForEach
```

>[!note] 
>**Windows Presentation Foundation (WPF) est exclusivement compatible avec Windows** dans cette version de .NET Core et sera traité en détail dans les chapitres [[Chapitre 24|24]] à [[Chapitre 28|28]]. Si vous n'avez jamais utilisé WPF ou si vous n'avez pas accès à un ordinateur Windows, tout le nécessaire pour cet exemple est listé ici. Si vous préférez suivre une solution déjà implémentée, vous trouverez DataParallelismWithForEach dans le dossier du chapitre 15 du dépôt GitHub. Le développement WPF fonctionne avec Visual Studio Code, bien qu'il ne prenne pas en charge le concepteur. Pour une expérience de développement plus riche, je vous recommande d'utiliser Visual Studio 2022 pour les exemples WPF de ce chapitre.

Au moment de la rédaction de ce document, les modèles de projet WPF ne prennent pas en charge les instructions `using` implicites globales. Mettez à jour le `PropertyGroup` principal dans le fichier *DataParallelismWithForEach.csproj* comme suit, ce qui désactive également les types référence pouvant être nuls :

>[!tip] Le point soulevé par l'auteur sur les `global using` implicite n'est plus d'actualité

```xml
<PropertyGroup>
	<OutputType>WinExe</OutputType>
	<TargetFramework>net6.0-windows</TargetFramework>
	<ImplicitUsings>enable</ImplicitUsings>
	<Nullable>disable</Nullable>
	<UseWPF>true</UseWPF>
</PropertyGroup>
```

Double-cliquez sur le fichier *MainWindow.xaml* dans l'Explorateur de solutions et remplacez le code XAML par le suivant :

>[!note]
>**XAML(Microsoft) / AXAML(Avalonia) = XML Structuré.**  
Chaque balise = Une classe C#.  
Chaque attribut = Une propriété de cette classe.  
L'extension change, mais les règles du XML (balises fermantes, nesting, casse) restent les mêmes.

```xml
<Window x:Class="DataParallelismWithForEach.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
  xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
  xmlns:local="clr-namespace:DataParallelismWithForEach"
  mc:Ignorable="d"
  Title="Fun with TPL" Height="400" Width="800">
  <Grid>
    <Grid.RowDefinitions>
      <RowDefinition Height="Auto" />
      <RowDefinition Height="*" />
      <RowDefinition Height="Auto" />
    </Grid.RowDefinitions>
    <Label Grid.Row="0" Grid.Column="0">
      Feel free to type here while the images are processed...
    </Label>
    <TextBox Grid.Row="1" Grid.Column="0" Margin="10,10,10,10" />
    <Grid Grid.Row="2" Grid.Column="0">
      <Grid.ColumnDefinitions>
        <ColumnDefinition Width="Auto" />
        <ColumnDefinition Width="*" />
        <ColumnDefinition Width="Auto" />
      </Grid.ColumnDefinitions>
      <Button Name="cmdCancel" Grid.Row="0" Grid.Column="0" Margin="10,10,0,10"
        Click="cmdCancel_Click">
        Cancel
      </Button>
      <Button Name="cmdProcess" Grid.Row="0" Grid.Column="2" Margin="0,10,10,10"
        Click="cmdProcess_Click">
        Click to Flip Your Images!
      </Button>
    </Grid>
  </Grid>
</Window>
```

***==Encore une fois, ne vous préoccupez pas de la signification ni du fonctionnement du balisage; vous consacrerez beaucoup de temps à WPF plus loin dans ce livre==***. L’interface graphique de l’application se compose d’une zone de texte multiligne et d’un bouton unique (nommé `cmdProcess`). La zone de texte permet de saisir des données pendant que le traitement s’effectue en arrière-plan, illustrant ainsi le caractère non bloquant de la tâche parallèle.

**Pour cet exemple, un package NuGet supplémentaire (`System.Drawing.Common`) est requis**. Pour l’ajouter à votre projet, saisissez la ligne suivante (sur une seule ligne) dans l’invite de commandes (dans le même répertoire que votre fichier de solution) ou dans la console du Gestionnaire de packages de Visual Studio :

```bash
dotnet add DataParallelismWithForEach package System.Drawing.Common
```

Ouvrez le fichier *MainWindow.xaml.cs* (double-cliquez dessus dans Visual Studio; vous devrez peut-être développer la flèche à côté de *MainWindow.xaml*), et ajoutez les instructions `using` suivantes en haut du fichier :

```cs
using System.Drawing;
using System.Windows;
using System.IO;
```

>[!note] 
>Vous devez modifier la chaîne de caractères transmise à l'appel de méthode `Directory.GetFiles()` suivant afin qu'elle pointe vers un chemin d'accès sur votre ordinateur contenant des fichiers image (par exemple, un dossier personnel de photos de famille). Pour votre commodité, j'ai inclus quelques exemples d'images (fournies avec le système d'exploitation Windows) dans le code d'exemple de ce chapitre.

```cs
public partial class MainWindow : Window
{ 
	public MainWindow()
	{ 
		InitializeComponent();
	}
	
	private void cmdCancel_Click(object sender, EventArgs e)
	{
		// Ceci sera mis à jour prochainement
	}
	
	private void cmdProcess_Click(object sender, EventArgs e)
	{ 
		this.Title = $"Starting...";
		ProcessFiles();
		this.Title = "Processing Complete";
	}
	
	private void ProcessFiles()
	{
		// Charger tous les fichiers *.jpg et créer un nouveau dossier pour les
		// données modifiées.
		// Obtenir le chemin du répertoire d'exécution du fichier
		// Pour le débogage sous VS 2022, le répertoire courant est <répertoire_projet>\bin\debug\ net6.0-windows
		// Pour VS Code ou « dotnet run », le répertoire courant est <répertoire_projet>
		var basePath = Directory.GetCurrentDirectory();
		var pictureDirectory =
		Path.Combine(basePath, "TestPictures");
		var outputDirectory =
		Path.Combine(basePath, "ModifiedPictures");
		// Supprimer les fichiers existants
		if (Directory.Exists(outputDirectory))
		{
			Directory.Delete(outputDirectory, true);
		}
		Directory.CreateDirectory(outputDirectory);
		string[] files = Directory.GetFiles(
			pictureDirectory, 
			"*.jpg", 
			SearchOption.AllDirectories
		);
		// Traitement bloquant des données image.
		foreach (string currentFile in files)
		{ 
			string filename = Path.GetFileName(currentFile);
			// Affichage de l'ID du thread traitant l'image courante.
			this.Title = $"Traitement de {filename} sur le thread {Environment.CurrentManagedThreadId}";
			using (Bitmap bitmap = new Bitmap(currentFile))
			{ 
				bitmap.RotateFlip(RotateFlipType.Rotate180FlipNone);
				bitmap.Save(Path.Combine(outputDirectory, filename));
			}
		}
	}
}
```

>[!note] 
>Si vous recevez une erreur indiquant que `Path` est une référence ambiguë entre `System.IO.Path` et `System.Windows.Shapes.Path`, supprimez l'utilisation de `System.Windows.Shapes` ou ajoutez `System.IO` à `Path` : `System.IO.Path.Combine(...)`.

#### Avalonia

Pour illustrer cela, créez une nouvelle application Avalonia (le modèle est abrégé en avalonia.app) nommée *DataParallelismWithForEach*. Pour créer le projet à l'aide de l'interface de ligne de commande et l'ajouter à la solution du chapitre, saisissez la commande suivante :

```
dotnet new avalonia.app -n DataParallelismWithForEach 
dotnet sln Chapter15_AllProjects add DataParallelismWithForEach
```

>[!note]
>AvaloniaUI est le successeur cross-plateforme de WPF et il sera et sera traité en détail dans les chapitres [[Chapitre 24|24]] à [[Chapitre 28|28]]. Avalonia possède une extension sur VSCode, permettant d'avoir un designer graphique.

**Avalonia à pris le parti pris de ne pas prendre en charge les implicit `global using` pour éviter les conflits de nommage avec les type Microsoft**. La meilleure approche est de créer un fichier *GlobalUsings.cs* à la racine du projet et d'ajouter les espaces de noms voulu.

```cs
global using System;
global using System.Collections.Generic;
global using System.Linq;
global using System.Threading;
global using System.Threading.Tasks;
global using Avalonia;
global using Avalonia.Controls;
global using Avalonia.Threading;
```

Double-cliquez sur le fichier *MainWindow.axaml* dans l'Explorateur de solutions et remplacez le code AXAML par le suivant :

```xml
<Window xmlns="https://github.com"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
  xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
  xmlns:local="using:DataParallelismWithForEach"
  mc:Ignorable="d"
  x:Class="DataParallelismWithForEach.MainWindow"
  Title="Fun with TPL" Height="400" Width="800">

  <Grid>
    <Grid.RowDefinitions>
      <RowDefinition Height="Auto" />
      <RowDefinition Height="*" />
      <RowDefinition Height="Auto" />
    </Grid.RowDefinitions>

    <TextBlock Grid.Row="0" Margin="10">
      Feel free to type here while the images are processed...
    </TextBlock>

    <TextBox Grid.Row="1" Margin="10" AcceptsReturn="True" />

    <Grid Grid.Row="2">
      <Grid.ColumnDefinitions>
        <ColumnDefinition Width="Auto" />
        <ColumnDefinition Width="*" />
        <ColumnDefinition Width="Auto" />
      </Grid.ColumnDefinitions>

      <Button Name="cmdCancel" Grid.Column="0" Margin="10, 10, 0, 10"
        Click="cmdCancel_Click">
        Cancel
      </Button>

      <Button Name="cmdProcess" Grid.Column="2" Margin="10"
        Click="cmdProcess_Click">
        Click to Flip Your Images!
      </Button>
    </Grid>
  </Grid>
</Window>
```

***==Encore une fois, ne vous préoccupez pas de la signification ni du fonctionnement du balisage; vous consacrerez amplement de temps à Avalonia plus loin dans cet ouvrage==***. L’interface graphique de l’application se compose d’une zone de texte multi-ligne et d’un bouton unique (nommé `cmdProcess`). La zone de texte permet la saisie de données pendant l’exécution du traitement en arrière-plan, illustrant ainsi le caractère non bloquant de la tâche parallèle.

Pour cet exemple, un package NuGet supplémentaire (`SixLabors.ImageSharp`) est requis. **Pour l'ajouter à votre projet, saisissez la ligne suivante** (sur une seule ligne) **dans l'invite de commandes** (dans le même répertoire que votre fichier de solution) ou dans la console du Gestionnaire de packages de Visual Studio :

```bash
dotnet add package SixLabors.ImageSharp
```

Ouvrez le fichier *MainWindow.axaml.cs* (double-cliquez dessus dans Visual Studio ; vous devrez peut-être développer la flèche à côté de *MainWindow.axaml*), et ajoutez les instructions `using` suivantes en haut du fichier :

```cs
using SixLabors.ImageSharp; 
using SixLabors.ImageSharp.Processing;
```

>[!note] 
>Vous devez modifier la chaîne de caractères transmise à l'appel de méthode `Directory.GetFiles()` suivant afin qu'elle pointe vers un chemin d'accès sur votre ordinateur contenant des fichiers image (par exemple, un dossier personnel de photos de famille). Pour votre commodité, j'ai inclus quelques exemples d'images (fournies avec le système d'exploitation Windows) dans le code d'exemple de ce chapitre.

```cs
using System.IO
using System.Reflection;
using Avalonia.Interactivity;
using SixLabors.ImageSharp;
using SixLabors.ImageSharp.Processing;
// Conflit de nom entre Avalonia.Image et SixLabors.ImageSharp.Image
using SharpImage = SixLabors.ImageSharp.Image;

namespace DataParallelismWithForEach;

public partial class MainWindow : Window
{
    public MainWindow()
    {
        InitializeComponent();
    }

    private void cmdCancel_Click(object? Sender, RoutedEventArgs e)
    {
        //Ceci sera mis à jour prochainement
    }

    private void cmdProcess_Click(object? sender, RoutedEventArgs e)
    {
        this.Title = "Starting...";
        ProcessFiles();
        this.Title = "Processing Complete";
    }


    private void ProcessFiles()
    {
        // Charge tous les fichiers *.png et crée un nouveau dossier pour les
        // données modifiées.

        // Récupère le nom de l'assembly (ici: "DataParallelismWithForEach").
        string? projectName = Assembly.GetExecutingAssembly().GetName().Name;
        // Obtient le répertoire de travail (Working Directory) actuel.
        // Attention : Ce répertoire dépend de l'endroit d'où
        // l'application est lancée (Terminal vs IDE).

        var basePath = Directory.GetCurrentDirectory();
        var pictureDirectory = Path.Combine(basePath, projectName!, "assets");
        var outputDirectory = Path.Combine(
            basePath,
            projectName!,
            "modifiedAssets"
        );

        // Nettoie le dossier si il existe déjà.
        if (Directory.Exists(outputDirectory))
        {
            Directory.Delete(outputDirectory, true);
        }
        // Recrée le dossier de sortie des images
        Directory.CreateDirectory(outputDirectory);

        // Récupère tous les fichier .png contenu dans le dossier (récursif).
        string[] files = Directory.GetFiles(
            pictureDirectory,
            "*.png",
            SearchOption.AllDirectories
        );

        // Traiter les données d'image de manière bloquante.
        foreach (string currentFile in files)
        {
            string filename = Path.GetFileName(currentFile);

            // Affiche l'ID du thread traitant l'image.
            this.Title =
                $"Processing {filename} on thread {Environment.CurrentManagedThreadId}";
            using (SharpImage image = SharpImage.Load(currentFile))
            {
                image.Mutate(x => x.Flip(FlipMode.Horizontal));
                image.Save(Path.Combine(outputDirectory, filename));
            }
        }
    }
}
```

>[!note]
 Il y a un conflit de noms entre `Avalonia.Controls.Image` (UI) et `SixLabors.ImageSharp.Image` (Data). Une solution élégante est d'utiliser un alias `using SharpImage = ...` pour lever l'ambiguïté.

>[!tip] **Si le programme crash, c'est une bonne nouvelle !**
>La gestion du thread principal par macOS est différent par rapport à Windows. Ici, le système d'exploitation envoie des signaux à ton application pour savoir si elle est toujours vivante. Comme ton thread est "coincé" dans la boucle `foreach`, il ne répond pas.
>
>macOS considère que l'application est corrompue ou bloquée et envoie un signal `SIGABRT` (Abort) pour tuer le processus. C'est le fameux crash "Thread 0 crashed".
>
> **La solution pour résoudre les crashs est d'utiliser de threads secondaires.** **==Cette section du chapitre permet d'apprendre la méthode plus automatique pour créer des threads secondaire==** (*==au lieu de les créer manuellement==*).

#### Commun

Notez que la méthode `ProcessFiles()` effectuera une rotation de chaque fichier *.jpg* / *.png* dans le répertoire spécifié. ***==Actuellement, tout le traitement s'effectue sur le thread principal de l'exécutable. Par conséquent, lorsque vous cliquerez sur le bouton de traitement des fichiers, le programme semblera bloqué (ou crash sur macOS)==***. De plus, la barre d'adresse de la fenêtre indiquera également que le même thread principal traite le fichier, car nous n'avons qu'un seul thread d'exécution.

**Pour traiter les fichiers sur autant de cœurs de processeur que possible, vous pouvez réécrire la boucle `foreach` actuelle en utilisant `Parallel.ForEach()`**. **==Rappelez-vous que cette méthode a été surchargée de nombreuses fois; cependant, dans sa forme la plus simple, vous devez spécifier l'objet compatible `IEnumerable<T>` contenant les éléments à traiter==** (ici, le tableau de chaînes de caractères contenant les fichiers) **==et un délégué `Action<T>` pointant vers la méthode qui effectuera le traitement.==**

Voici la mise à jour correspondante, utilisant l'opérateur lambda C# à la place d'un objet délégué `Action<T>` littéral. **Notez que vous *commentez* actuellement la ligne de code qui affichait l'ID du thread exécutant le fichier image actuel. Consultez la section suivante pour en connaître la raison.**

>[!info] Même avec ces modifications, le programme plantera toujours sur macOS

```cs
	...
// Traiter les données d'image en parallèle !
Parallel.ForEach(files, currentFile =>
	{
		// Simule une charge te travail plus importante
		// (pour voir les bonds en performances)
		Thread.Sleep(1000);

		string filename = Path.GetFileName(currentFile);
		// Cette instruction pose problème ! Voir la section suivante.
		// this.Title = $"Traitement de {filename} sur le thread
		//     {Environment.CurrentManagedThreadId}";
		
		...
	}
);
```

## Accès aux éléments d'interface utilisateur sur les threads secondaires

==Vous remarquerez que j'ai commenté la ligne de code précédente qui mettait à jour le titre de la fenêtre principale avec l'ID du thread en cours d'exécution==. Comme indiqué précédemment, **les contrôles d'interface graphique ont une « affinité de thread » avec le thread qui les a créés**. *==Si des threads secondaires tentent d'accéder à un contrôle qu'ils n'ont pas créé directement, cela provoquera une exception d'exécution==*. Cependant, ==cela n'est pas systématique (**sauf sur macOS**) et aucune exception n'est levée==. ***==Les problèmes de gestion des threads dépendent de nombreuses variables et sont généralement intermittents et très difficiles à reproduire.==***

>[!note] 
>Je tiens à réaffirmer le point précédent : lors du débogage d’une application multithread, il est parfois possible de détecter les erreurs survenant lorsqu’un thread secondaire accède à un contrôle créé sur le thread principal. Cependant, souvent, à l’exécution, l’application peut sembler fonctionner correctement (ou générer une erreur immédiatement). Tant que vous n’aurez pas pris les précautions nécessaires (détaillées ci-après), votre application risque de générer une erreur d’exécution dans de telles circonstances.

**Une approche permettant à ces threads secondaires d'accéder aux contrôles de manière thread-safe consiste à utiliser une autre technique basée sur les délégués, et plus précisément un délégué *anonyme***. 

#### WPF

**La classe parente `Control` dans WPF définit un objet `Dispatcher` qui gère les éléments de travail d'un thread**. Cet objet possède une méthode nommée `Invoke()` qui prend un `System.Delegate` en entrée. **==Vous pouvez appeler cette méthode lorsque vous êtes dans un contexte de développement impliquant des threads secondaires afin de mettre à jour l'interface utilisateur du contrôle concerné de manière thread-safe==**. Bien qu'il soit possible d'écrire directement tout le code du délégué nécessaire, la plupart des développeurs utilisent la syntaxe d'expression comme alternative plus simple. Voici la mise à jour correspondante du contenu, avec la ligne de code précédemment commentée :

```cs
// Aïe ! Cela ne fonctionnera plus !
//this.Title = $"Traitement de {filename} sur le thread {Environment.CurrentManagedThreadId}";

// Appele la méthode sur l'objet Form pour permettre aux threads secondaires 
// d'accéder aux contrôles de manière thread-safe.
Dispatcher?.Invoke(() =>
{ 
	this.Title = $"Traitement de {filename}";
});
using (Bitmap bitmap = new Bitmap(currentFile))
{ 
	bitmap.RotateFlip(RotateFlipType.Rotate180FlipNone);
	bitmap.Save(Path.Combine(outputDirectory, filename));
}

// Simule une charge te travail plus importante
// (pour voir les bonds en performances)
Thread.Sleep(2000);
```

Maintenant, si vous exécutez le programme, le TPL répartira bien la charge de travail sur plusieurs threads du pool de threads, en utilisant autant de cœurs de processeur que possible. Cependant, comme le titre est toujours mis à jour depuis le thread principal, le code de mise à jour du titre n'affiche plus le thread courant, et vous ne verrez rien si vous saisissez du texte dans la zone de texte tant que toutes les images n'auront pas été traitées ! La raison est que le thread d'interface utilisateur principal est toujours bloqué, attendant que tous les autres threads aient terminé leur traitement.

#### Avalonia

En **Avalonia**, contrairement au WPF classique, on n'accède pas au `Dispatcher` via une propriété de la classe `Control`, mais via la classe ***==statique==*** `Dispatcher.UIThread` située dans l'espace de noms `Avalonia.Threading`.

Cet objet gère la file d'attente des travaux du thread principal. Il possède une méthode nommée **`Post()`** (ou `InvokeAsync()`) qui prend un délégué (souvent une expression lambda) en entrée. Vous devez appeler cette méthode lorsque vous travaillez avec des threads secondaires (comme dans une `Task` ou un `Parallel.ForEach`) pour demander au thread UI de mettre à jour l'interface utilisateur. Bien qu'il soit possible de déclarer un délégué explicitement, l'utilisation d'une **expression lambda** est la norme moderne pour garantir que l'application ne plante pas avec une erreur de type `Abort()` sur macOS.

```cs
Dispatcher.UIThread.Post(() =>
{
	this.Title = $"Processing {filename}";
});
using (SharpImage image = SharpImage.Load(currentFile))
{
	image.Mutate(x => x.Flip(FlipMode.Horizontal));
	image.Save(Path.Combine(outputDirectory, filename));
}

// Simule une charge te travail plus importante
// (pour voir les bonds en performances)
Thread.Sleep(2000);

```

## La classe `Task`

>[!success] Cette sous-section est la solution aux crashs sur macOS.

**La classe `Task` vous permet d'appeler facilement une méthode sur un thread secondaire et peut être utilisée comme une alternative simple à l'utilisation de délégués asynchrones**. Mettez à jour le gestionnaire `Click` de votre contrôle `Button` comme suit :

```cs
private void cmdProcess_Click(object? sender, RoutedEventArgs e)
{
	this.Title = $"Starting...";
	// Lancer une nouvelle « tâche » pour traiter les fichiers.
	Task.Factory.StartNew(ProcessFiles);
	// Peut également s'écrire ainsi :
	//Task.Factory.StartNew(() => ProcessFiles());
}
```

**La propriété `Factory` de `Task` renvoie un objet `TaskFactory`**. **==Lorsque vous appelez sa méthode `StartNew()`, vous lui transmettez un délégué `Action<T>`==** (ici, ==masqué par une expression lambda== appropriée) **==qui pointe vers la méthode à invoquer de manière asynchrone==**. Grâce à cette petite mise à jour, vous constaterez désormais que le titre de la fenêtre indique quel thread du pool de threads traite un fichier donné et, mieux encore, la zone de texte peut recevoir des entrées, car le thread d'interface utilisateur n'est plus bloqué.

## Gestion des demandes d'annulation

**Une amélioration possible de l'exemple actuel consiste à permettre à l'utilisateur d'interrompre le traitement des données d'image, via un second bouton `Cancel`** (dont le nom est tout à fait approprié). **Heureusement, les méthodes `Parallel.For()` et `Parallel.ForEach()` prennent toutes deux en charge l'annulation à l'aide de** *jetons d'annulation* (*Cancellation tokens*). **==Lorsque vous appelez des méthodes sur `Parallel`, vous pouvez lui transmettre un objet `ParallelOptions`, qui contient un objet `CancellationTokenSource`.==**

Commencez par définir la nouvelle variable membre privée suivante dans votre classe dérivée, de type `CancellationTokenSource`, nommée `_cancelToken`:

>[!info] Il existe deux jetons d'annulation. [Video avec un des développeur du language](https://www.youtube.com/watch?v=h1GvSPaRQ-U)
>
>- `CancellationToken`: C'est un `struct` qui sert à **observer** si une annulation à été déclenché et ainsi passer l'information dans la chaîne d'appel (*call stack*). C'est le **jeton** que l'on passe à tout ceux qui en ont besoin.
>- `CancellationTokenSource`: C'est une `class` qui elle permet de **générer** l'information d'annulation, elle est la **source**
>
>Le `CancellationToken` possède une propriété `IsCancellationRequested` pour vérifier manuellement l'état, ou la méthode `ThrowIfCancellationRequested()` pour stopper net l'exécution.
>
>La classe `CancellationTokenSource` possède la propriété `Token`, c'est cela que l'on envois dans les méthodes.

```cs
public partial class MainWindow : Window
{
    // Nouvelle variable au niveau de Window
    private CancellationTokenSource _cancelToken = new();
	..
}
```

Mettez à jour l'événement `Click` sur le bouton "*Cancel*" avec le code suivant :

```cs
private void cmdCancel_Click(object? Sender, RoutedEventArgs e)
{
	// Ce sera utilisé pour dire à tous les thread de travail d'arrêter.
	_cancelToken.Cancel();
}
```

Les véritables modifications doivent maintenant être apportées à la méthode `ProcessFiles()`. Voici l'implémentation finale :

#### WPF

```cs
private void ProcessFiles()
{
	// Utilise une instance de ParallelOptions
	// por stocker le CancellationToken
	ParallelOptions parOpts = new ParallelOptions();
	parOpts.CancellationToken = _cancelToken.Token;
	parOpts.MaxDegreeOfParallelism = System.Environment.ProcessorCount;
	
	// Charger tous les fichiers *.jpg et créer un nouveau dossier pour les données modifiées.
	string[] files = Directory.GetFiles(
		`@".\TestPictures", 
		"*.jpg", 
		SearchOption.AllDirectories
	);
	string outputDirectory = @".\ModifiedPictures";
	Directory.CreateDirectory(outputDirectory);
	try
	{
		// Traiter les données d'image en parallèle !
		Parallel.ForEach(files, parOpts, currentFile => 
		{
			parOpts.CancellationToken.ThrowIfCancellationRequested();
		
			string filename = Path.GetFileName(currentFile);
		
			Dispatcher?.Invoke(() =>
				{
				this.Title =
					$"Processing {filename} on thread {Environment.CurrentManagedThreadId}";
				}
			);
			using (Bitmap bitmap = new Bitmap(currentFile))
			{
				bitmap.RotateFlip(RotateFlipType.Rotate180FlipNone);
				bitmap.Save(Path.Combine(outputDirectory, filename));
			}
		});
		Dispatcher?.Invoke(()=>this.Title = "Done!");
	}
	catch (OperationCanceledException ex)
	{
		Dispatcher?.Invoke(()=>this.Title = ex.Message);
	}
}
```

#### Avalonia

```cs
private void ProcessFiles()
{
	// Charge tous les fichiers *.png et crée un nouveau dossier pour les
	// données modifiées.
	string? projectName = Assembly.GetExecutingAssembly().GetName().Name;
	var basePath = Directory.GetCurrentDirectory();
	var pictureDirectory = Path.Combine(basePath, projectName!, "assets");
	var outputDirectory = Path.Combine(
		basePath,
		projectName!,
		"modifiedAssets"
	);

	// Utilise une instance de ParallelOptions
	// por stocker le CancellationToken
	ParallelOptions parOpts = new ParallelOptions();
	parOpts.CancellationToken = _cancelToken.Token;
	parOpts.MaxDegreeOfParallelism = Environment.ProcessorCount;

	// Nettoie le dossier si il existe déjà.
	if (Directory.Exists(outputDirectory))
	{
		Directory.Delete(outputDirectory, true);
	}
	// Recrée le dossier de sortie des images
	Directory.CreateDirectory(outputDirectory);

	// Récupère tous les fichier .png contenu dans le dossier (récursif).
	string[] files = Directory.GetFiles(
		pictureDirectory,
		"*.png",
		SearchOption.AllDirectories
	);

	try
	{
		// Traiter les données d'image en parallèle !
		Parallel.ForEach(
			files,
			parOpts,
			currentFile =>
			{
				// Génère l'exception d'annulage.
				parOpts.CancellationToken.ThrowIfCancellationRequested();

				string filename = Path.GetFileName(currentFile);

				Dispatcher.UIThread.Post(() =>
				{
					this.Title =
						$"Processing {filename} on thread {Environment.CurrentManagedThreadId}";
				});

				using (SharpImage image = SharpImage.Load(currentFile))
				{
					image.Mutate(x => x.Flip(FlipMode.Horizontal));
					image.Save(Path.Combine(outputDirectory, filename));
				}

				// Simule une charge te travail plus importante
				// (pour voir les bonds en performances)
				Thread.Sleep(2000);
			}
		);
		Dispatcher.UIThread.Post(() => this.Title = "Done!");
	}
	catch (OperationCanceledException ex)
	{
		Dispatcher.UIThread.Post(() => this.Title = ex.Message);
	}
}
```

#### Commun

**Notez que vous commencez la méthode en configurant un objet `ParallelOptions` et en définissant la propriété `CancellationToken` pour utiliser le jeton `CancellationTokenSource`**. Notez également que ==lorsque vous appelez la méthode `Parallel.ForEach()`, vous transmettez l'objet `ParallelOptions` comme deuxième paramètre.==

**Dans la boucle, vous appelez la méthode `ThrowIfCancellationRequested()` sur le jeton, ce qui garantit que si l'utilisateur clique sur le bouton "*cancel*", tous les threads s'arrêtent et vous êtes notifié par une exception d'exécution**. *==Lorsque vous interceptez l'erreur `OperationCanceledException`, vous définissez le texte de la fenêtre principale avec le message d'erreur.==*

>[!note] 
>Nouveauté de C# 10 : la classe Parallel propose une nouvelle méthode `async` nommée `ForEachAsync()`. Elle sera abordée plus loin dans ce chapitre.

## Parallélisme des tâches avec la classe Parallel

**Outre le parallélisme des données, la bibliothèque TPL permet également de lancer facilement un nombre quelconque de tâches asynchrones à l'aide de la méthode `Parallel.Invoke()`**. **==Cette approche est plus simple que l'utilisation des membres de `System.Threading`==**; toutefois, **si vous avez besoin d'un contrôle plus précis sur l'exécution des tâches, vous pouvez renoncer à `Parallel.Invoke()` et utiliser directement la classe `Task`, comme dans l'exemple précédent.**

Pour illustrer le parallélisme des tâches, créez une application console nommée *MyEBookReader* et assurez-vous que les espaces de noms `System.Text` et `System.Net` sont importés en haut du fichier *Program.cs* (cet exemple est une modification d'un exemple utile de la documentation .NET Core). ==Vous allez récupérer un livre numérique public depuis le site du Projet Gutenberg== (www.gutenberg.org) ==et exécuter ensuite une série de tâches longues en parallèle.==

Le livre est téléchargé dans la méthode `GetBook()`, comme illustré ici :

```cs
using System.Net;
using System.Text;

string _theEBook = "";
GetBook();
Console.WriteLine("Downloading book...");
Console.ReadLine();

void GetBook()
{
    //NOTE: WebClient est obsolète.
    //Nous reprendrons cet exemple avec HttpClient lors de notre
    //discussion sur async/await.
    using WebClient wc = new WebClient();
    wc.DownloadStringCompleted += (s, eArgs) =>
    {
        _theEBook = eArgs.Result;
        Console.WriteLine("Download complete.");
        GetStats();
    };

    // Le livre numérique du Projet Gutenberg :
    // « Un conte de deux villes », de Charles Dickens

    // Il se peut que vous deviez l’exécuter deux fois si vous n’avez
    // jamais visité le site auparavant, car lors de la première
    // visite, une boîte de dialogue apparaît et interrompt ce code.
    wc.DownloadStringAsync(
        new Uri("https://www.gutenberg.org/files/98/98-0.txt")
    );
}
```

==La classe `WebClient` fait partie de `System.Net`==. **Cette classe fournit des méthodes pour envoyer des données à une ressource identifiée par un URI et en recevoir**. **==Il s'avère que nombre de ces méthodes possèdent une version asynchrone, comme `DownloadStringAsync()`==**. **Cette méthode crée automatiquement un nouveau thread à partir du pool de threads du runtime .NET Core**. ==Une fois les données obtenues, `WebClient` déclenche l'événement `DownloadStringCompleted`, que vous gérez ici à l'aide d'une expression lambda C#==. Si vous appeliez la version synchrone de cette méthode (`DownloadString()`), le message « Téléchargement en cours » ne s'afficherait pas avant la fin du téléchargement.

>[!warning] Attention
Le type `WebClient` est obsolète et a été remplacé par le `HttpClient`. Nous reprendrons cet exemple en utilisant la classe `HttpClient` dans la section « [[#Appels asynchrones utilisant le modèle `async`/`await`]] » de ce chapitre.

Ensuite, **la méthode `GetStats()` est implémentée pour extraire les mots individuels contenus dans la variable `theEBook`, puis transmettre le tableau de chaînes à quelques fonctions auxiliaires pour traitement**, comme suit :

```cs
void GetStats()
{
    // Récupère les mot du livre numérique.
    string[] words = _theEBook.Split(
        // Syntaxe plus simple (C# 12).
        [' ', '\u000A', ',', '.', ';', ':', '-', '?', '/'],
        StringSplitOptions.RemoveEmptyEntries
    );

    // Maintenant, trouve le 10ème mot le plus présent.
    string[] tenMostCommon = FindTenMostCommon(words);

    // Récupère le mot le plus long.
    string longestWord = FindLongestWord(words);

    // Maintenant que toutes les tâches sont accomplies,
    // construit un string pour montrer toute les stats.
    StringBuilder bookStats = new StringBuilder("Ten Most Common Words are:\n");
    foreach(string s in tenMostCommon)
    {
      bookStats.AppendLine(s);
    }

    bookStats.AppendLine($"Longest word is: {longestWord}");
    bookStats.AppendLine();
    Console.WriteLine(bookStats.ToString(), "Book info");
}
```

La méthode `FindTenMostCommon()` utilise une requête LINQ pour obtenir une liste d'objets de type chaîne qui apparaissent le plus souvent dans le tableau de chaînes, tandis que `FindLongestWord()` localise, eh bien, le mot le plus long.

```cs
string[] FindTenMostCommon(string[] words)
{
    var frequencyOrder =
        from word in words
        where word.Length > 6
        group word by word into g
        orderby g.Count() descending
        select g.Key;

    // Syntaxe plus simple et optimisé (C# 12)
    string[] commonWords = [.. frequencyOrder.Take(10)];
    return commonWords;
}

string FindLongestWord(string[] words)
{
    return (
            from w in words
            orderby w.Length descending
            select w
        ).FirstOrDefault() ?? string.Empty;
}

```

L'exécution de ce projet peut prendre un temps considérable, en fonction du nombre de cœurs de votre processeur et de sa vitesse globale. Vous devriez ensuite obtenir le résultat suivant (voir ci-dessous):

```
Downloading book...
Download complete.
Ten Most Common Words are:
Defarge
himself
through
Manette
nothing
another
business
looking
prisoner
Cruncher
Longest word is: undistinguishable
```

**Vous pouvez vous assurer que votre application utilise tous les cœurs du processeur disponibles sur la machine hôte en appelant les méthodes `FindTenMostCommon()` et `FindLongestWord()` en parallèle**. Pour ce faire, modifiez votre méthode `GetStats()` comme suit :

```cs
void GetStats()
{
    // Récupère les mot du livre numérique.
    string[] words = _theEBook.Split(
        // Syntaxe plus simple (C# 12).
        [' ', '\u000A', ',', '.', ';', ':', '-', '?', '/'],
        StringSplitOptions.RemoveEmptyEntries
    );
    string[]? tenMostCommon = null;
    string longestWord = string.Empty;

    Parallel.Invoke(
        () =>
        {
            // Maintenant, trouve le 10ème mot le plus présent.
            tenMostCommon = FindTenMostCommon(words);
        },
        () =>
        {
            // Récupère le mot le plus long.
            longestWord = FindLongestWord(words);
        }
    );

    // Maintenant que toutes les tâches sont accomplies,
    // construit un string pour montrer toute les stats.
    StringBuilder bookStats = new StringBuilder("Ten Most Common Words are:\n");
    foreach (string s in tenMostCommon!)
    {
        bookStats.AppendLine(s);
    }

    bookStats.AppendLine($"Longest word is: {longestWord}");
    bookStats.AppendLine();
    Console.WriteLine(bookStats.ToString(), "Book info");
}
```

**La méthode `Parallel.Invoke()` attend un tableau de délégués `Action<>` en paramètre, que vous avez fourni indirectement à l'aide d'expressions lambda(ici deux)**. Là encore, **==bien que le résultat soit identique, l'avantage est que la bibliothèque TPL utilisera désormais tous les processeurs disponibles sur la machine pour appeler chaque méthode en parallèle, si possible.==**

# Requêtes LINQ parallèles (PLINQ)

Pour conclure votre présentation de la bibliothèque TPL, **sachez qu'il existe une autre façon d'intégrer des tâches parallèles à vos applications .NET Core**. ==Vous pouvez utiliser un ensemble de méthodes d'extension permettant de construire une requête LINQ qui exécutera sa charge de travail en parallèle (si possible)==. Logiquement, les requêtes LINQ conçues pour s'exécuter en parallèle sont appelées *requêtes PLINQ*.

**À l'instar du code parallèle écrit avec la classe `Parallel`, PLINQ peut ignorer votre demande de traitement parallèle de la collection si nécessaire**. **==Le framework PLINQ a été optimisé de nombreuses manières, notamment pour déterminer si une requête serait plus performante en mode synchrone.==**

À l'exécution, **PLINQ analyse la structure globale de la requête et, si celle-ci est susceptible de bénéficier d'une parallélisation, elle s'exécutera de manière concurrente**. En revanche, *==si la parallélisation d'une requête risque de dégrader les performances, PLINQ exécute simplement la requête de manière séquentielle==*. **Si PLINQ a le choix entre un algorithme parallèle potentiellement coûteux et un algorithme séquentiel peu coûteux, il choisit par défaut l'algorithme séquentiel.**

Les méthodes d'extension nécessaires se trouvent dans la classe `ParallelEnumerable` de l'espace de noms `System.Linq`. Le [[#Tableau 15-6 Sélection de membre de la classe `ParallelEnumerable`|Tableau 15-6]] présente quelques extensions PLINQ utiles.

##### Tableau 15-6: Sélection de membre de la classe `ParallelEnumerable`

| Membre                      | Description                                                                                                                                                                                              |
| --------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `AsParallel()`              | Indique que le reste de la requête doit être parallélisé, si possible.                                                                                                                                   |
| `WithCancellation()`        | Indique que PLINQ doit surveiller périodiquement l'état du jeton d'annulation fourni et annuler l'exécution si une demande est formulée.                                                                 |
| `WithDegreeOfParallelism()` | Spécifie le nombre maximal de processeurs que PLINQ doit utiliser pour<br>paralléliser la requête                                                                                                        |
| `ForAll()`                  | Permet de traiter les résultats en parallèle sans les fusionner au préalable avec le thread consommateur, comme ce serait le cas lors de l'énumération d'un résultat LINQ à l'aide du mot-clé `foreach`. |

Pour voir PLINQ en action, créez une application console nommée *PLINQDataProcessingWithCancellation*. **Au démarrage du traitement, le programme lancera une nouvelle tâche qui exécutera une requête LINQ analysant un grand tableau d'entiers, à la recherche des éléments pour lesquels $x \div 3 == 0$ est `true`**. Voici une ==version non parallèle== de la requête :

```cs
Console.WriteLine("Start any key to start processing");
Console.ReadKey();

Console.WriteLine("Processing");
Task.Factory.StartNew(ProcessIntData);
Console.ReadLine();

void ProcessIntData()
{
    // Obtenir un très grand tableau d'entiers.
    // (Syntaxe C# 12)
    int[] source = [.. Enumerable.Range(1, 10_000_000)];

    // Trouver les nombres pour lesquels num % 3
    // == 0 est vrai, et les retourner
    // par ordre décroissant.
    // (On compbine avec la syntaxe de C# 12)
    int[] modThreeIsZero =
    [
        .. from num in source
        where num % 3 == 0
        orderby num descending
        select num,
    ];
    
	// Important, pour bien voir le comportement.
	Thread.Sleep(5000);

    Console.WriteLine(
        $"Found {modThreeIsZero.Count()} numbers that match query!"
    );
}
```

## Opter vers une requête PLINQ

Pour indiquer à la bibliothèque de traitement de (TPL) d'exécuter cette requête en parallèle (si possible), utilisez la méthode d'extension `AsParallel()` comme suit :

```cs
// (On compbine avec la syntaxe de C# 12)
int[] modThreeIsZero =
[
	.. from num in source.AsParallel()
	where num % 3 == 0
	orderby num descending
	select num,
];
```

**Remarquez que le format général de la requête LINQ est identique à celui vu dans les chapitres précédents**. Cependant, **==en incluant un appel à `AsParallel()`, la bibliothèque TPL tentera de répartir la charge de travail sur n'importe quel processeur disponible.==**

## Annulation d'une requête PLINQ

**Il est également possible d'utiliser un objet `CancellationTokenSource` pour indiquer à une requête PLINQ d'arrêter son traitement dans les conditions appropriées** (généralement suite à une intervention de l'utilisateur). Déclarez un objet `CancellationTokenSource` au niveau de la classe, nommé `_cancelToken`, et mettez à jour la méthode statements de niveau supérieur pour qu'elle reçoive une entrée de l'utilisateur. Voici la mise à jour de code correspondante :

```cs
CancellationTokenSource _cancelToken = new();

do
{
    Console.WriteLine("Start any key to start processing");
    Console.ReadKey();

    Console.WriteLine("Processing");
    Task.Factory.StartNew(ProcessIntData);

    Console.Write(" Enter Q to quit: ");
    string answer = Console.ReadLine();
    // Est ce que l'utilisateur veut partir ?
    if (answer.Equals('Q', StringComparison.OrdinalIgnoreCase))
    {
        _cancelToken.Cancel();
        break;
    }
} while (true);

Console.ReadLine();
```

**Maintenant, indiquez à la requête PLINQ qu'elle doit surveiller une demande d'annulation entrante en chaînant les appels à la méthode d'extension `WithCancellation()` et en lui passant le jeton**. De plus, ***==vous devrez encapsuler cette requête PLINQ dans un bloc `try`/`catch` approprié et gérer l'exception éventuelle==***. Voici la version finale de la méthode `ProcessIntData()` :

```cs
void ProcessIntData()
{
    // Obtenir un très grand tableau d'entiers.
    // (Syntaxe C# 12)
    int[] source = [.. Enumerable.Range(1, 10_000_000)];

    // Trouver les nombres pour lesquels num % 3 == 0 est vrai,
    // et les retourner par ordre décroissant.
    // (On compbine avec la syntaxe de C# 12)
    int[] modThreeIsZero = null;

    try
    {
        modThreeIsZero =
        [
            .. from num in source
                .AsParallel()
                .WithCancellation(_cancelToken.Token)
            where num % 3 == 0
            orderby num descending
            select num,
        ];
        // Important, pour bien voir le comportement.
        Thread.Sleep(5000);
        Console.WriteLine(
            $"Found {modThreeIsZero.Count()} numbers that match query!"
        );
    }
    catch (OperationCanceledException ex)
    {
        Console.WriteLine(ex.Message);
    }
}
```

# Appels asynchrones utilisant le modèle `async`/`await`

J'ai abordé de nombreux sujets dans ce chapitre (plutôt long). Il est certain que la création, le débogage et la compréhension d'applications multithread complexes représentent un défi, quel que soit le framework. *==Bien que la bibliothèque TPL, PLINQ et le type délégué puissent simplifier les choses dans une certaine mesure==* (surtout par rapport à d'autres plateformes et langages), *==les développeurs doivent néanmoins maîtriser les subtilités de diverses techniques avancées.==*

***==Depuis la sortie de .NET 4.5, le langage de programmation C# a été mis à jour avec deux nouveaux mots-clés qui simplifient encore davantage l'écriture de code asynchrone==***. Contrairement à tous les exemples de ce chapitre, **lorsque vous utilisez les nouveaux mots-clés `async` et `await`, le compilateur génère automatiquement une grande partie du code de gestion des threads, en utilisant de nombreux membres des espaces de noms `System.Threading` et `System.Threading.Tasks`.**

## Premier aperçu des mots-clés `async` et `await` en C# (MaJ C# 7.1 et 9.0)

**Le mot-clé `async` de C# permet de spécifier qu'une méthode, une expression lambda ou une méthode anonyme doit être appelée de manière asynchrone**. En effet, **==en marquant simplement une méthode avec le modificateur `async`, le runtime .NET Core créera un nouveau thread d'exécution pour traiter la tâche==**. De plus, **lors de l'appel d'une méthode asynchrone, le mot-clé `await` suspendra automatiquement le thread courant jusqu'à la fin de la tâche, permettant ainsi au thread appelant de poursuivre son exécution.**

Pour illustrer cela, créez une application console nommée `FunWithCSharpAsync` et ajoutez une méthode nommée `DoWork()`, qui force le thread appelant à attendre cinq secondes. Voici le résultat :

```cs
Console.Title = "Fun with Async";
Console.WriteLine(" Fun with Async ===>");
Console.WriteLine(DoWork());
Console.WriteLine("Completed");
Console.ReadLine();

static string DoWork()
{
    Thread.Sleep(5000);
    return "Done with work!";
}
```

Maintenant, compte tenu du travail effectué dans ce chapitre, vous savez que si vous deviez exécuter le programme, vous devriez attendre cinq secondes avant que quoi que ce soit d'autre ne puisse se produire. S'il s'agissait d'une application graphique, l'écran entier serait bloqué jusqu'à la fin du traitement.

**Si vous deviez utiliser l'une des techniques présentées précédemment dans ce chapitre pour rendre votre programme plus réactif, vous auriez beaucoup de travail devant vous**. **==Cependant, depuis .NET 4.5, vous pouvez écrire le code C# suivant :==**

```cs
...

string message = await DoWorkASync();
Console.WriteLine(message);

...

static async Task<string> DoWorkASync()
{
    return await Task.Run(() =>
    {
        Thread.Sleep(5000);
        return "Done with work!";
    });
}
```

***==Si vous utilisez une méthode `Main()` comme point d'entrée===** (au lieu d'instructions de niveau supérieur), ***==vous devez marquer la méthode comme `async`, une fonctionnalité introduite dans C# 7.1.==***

```cs
static async Task Main(string[] args)
{
	...
	string message = await DoWorkAsync();
	Console.WriteLine(message);
	...
}
```

>[!note] 
>La possibilité d'utiliser le mot-clé `async` sur la méthode `Main()` est une nouveauté de C# 7.1. **Les instructions de niveau supérieur, ajoutées dans C# 9.0, sont implicitement `async`.**

==Notez le mot-clé `await` avant le nom de la méthode qui sera appelée de manière asynchrone==. Ceci est important: **si vous décorez une méthode avec le mot-clé `async` mais qu'elle ne contient pas au moins un appel de méthode interne basé sur `await`, vous créez en réalité un appel de méthode synchrone** (==vous recevrez d'ailleurs un avertissement du compilateur à ce sujet==).

**Notez maintenant que vous devez utiliser la classe `Task` de l'espace de noms `System.Threading.Tasks` pour refactoriser vos méthodes `Main()` (si vous utilisez `Main()`) et `DoWork()`** (cette dernière est ajoutée sous le nom `DoWorkAsync()`). ***==Concrètement, au lieu de retourner directement une valeur spécifique (un objet `string` dans l'exemple actuel), vous retournez un objet `Task<T>`, où le paramètre de type générique `T` est la valeur de retour sous-jacente==***. **Si la méthode ne retourne aucune valeur (comme la méthode `Main()`), alors au lieu de `Task<T>`, vous utilisez simplement `Task`**. 

**L'implémentation de `DoWorkAsync()` renvoie désormais directement un objet `Task<T>`, qui est la valeur de retour de `Task.Run()`**. ***==La méthode `Run()` accepte un délégué `Func<>` ou `Action<>`, et comme vous le savez à ce stade du texte, vous pouvez simplifier le code en utilisant une expression lambda==***. En résumé, votre nouvelle version de `DoWorkAsync()` indique essentiellement ce qui suit :

***Lorsque vous m'appellerez, j'exécuterai une nouvelle tâche. Cette tâche mettra le thread appelant en veille pendant cinq secondes, et une fois terminée, elle me renverra une chaîne de caractères. Je placerai cette chaîne dans un nouvel objet `Task<string>` et le renverrai à l'appelant.***

Après avoir traduit cette nouvelle implémentation de `DoWorkAsync()` dans un langage plus naturel (poétique), ==vous comprenez mieux le rôle réel du jeton `await`==. **Ce mot-clé modifie toujours une méthode qui renvoie un objet `Task`**. **==Lorsque le flux d'exécution atteint le jeton `await`, le thread appelant est suspendu dans cette méthode jusqu'à la fin de l'appel==**. ==S'il s'agissait d'une application graphique, l'utilisateur pourrait continuer à utiliser l'interface utilisateur pendant l'exécution de la méthode `DoWorkAsync()`.==

##### Tableau 15-7: Juxtaposition entre la méthode manuelle et `async`/`await`/

| Ce que tu as fait manuellement              | Son remplaçant moderne                                |
| ------------------------------------------- | ----------------------------------------------------- |
| `Parallel.Invoke(() => DoA(), () => DoB())` | `await Task.WhenAll(Task.Run(DoA), Task.Run(DoB))`    |
| `Dispatcher.UIThread.Post(...)`             | Rien ! `await` revient sur le thread UI tout seul.    |
| `string result;` (déclarée avant)           | `string result = await Task.Run(...);`                |
| `_cancelToken.Cancel()`                     | Reste le même (on passe toujours le token aux Tasks). |

## `SynchronizationContext` et `async`/`await`


**La définition officielle de `SynchronizationContext` est celle d'une classe de base fournissant un contexte multithread sans synchronisation**. Bien que cette définition initiale soit peu explicite, ==la [documentation](https://learn.microsoft.com/en-us/dotnet/api/system.threading.synchronizationcontext?view=net-10.0) officielle== précise :

***Le but du modèle de synchronisation implémenté par cette classe est de permettre aux opérations internes asynchrones/synchrones du Common Language Runtime de fonctionner correctement avec différents modèles de synchronisation.***

Cette affirmation, combinée à vos connaissances sur le multithreading, éclaire le sujet. **Rappelons que
les applications GUI** (WinForms, WPF, Avalonia) **n'autorisent pas les threads secondaires à accéder directement aux contrôles; ils doivent déléguer cet accès**. ***==Nous avons déjà vu l'objet `Dispatcher` dans l'exemple WPF/Avalonia==***. **Pour les applications console n'utilisant pas WPF/Avalonia, cette restriction ne s'applique pas**. Ce sont là les différents modèles de synchronisation auxquels il est fait référence. ==Dans cette optique, examinons plus en détail le `SynchronizationContext`.==

**Le `SynchronizationContext` est un type qui fournit une méthode `Post` virtuelle, prenant un délégué à exécuter de manière asynchrone**. **==Ceci offre aux frameworks un modèle pour gérer correctement les requêtes asynchrones==** (dispatch pour Avalonia/WPF/WinForms, exécution directe pour les applications non GUI, etc.). **Il permet de mettre en file d'attente une unité de travail dans un contexte et de comptabiliser les opérations asynchrones en cours. **

Comme nous l'avons vu précédemment, **lorsqu'un délégué est mis en file d'attente pour une exécution asynchrone, il est planifié pour s'exécuter sur un thread distinct**. ***==Ce détail est géré par le runtime .NET Core. Cela se fait généralement à l'aide du pool de threads géré par le runtime .NET Core, mais il est possible de le remplacer par une implémentation personnalisée.==***

**==Bien que ces opérations d'infrastructure puissent être gérées manuellement par le code, le modèle `async`/`await` prend en charge la majeure partie du travail==**. **Lorsqu'une méthode asynchrone est attendue, elle exploite les implémentations de `SynchronizationContext` et de `TaskScheduler` du framework cible**. ***==Par exemple, si vous utilisez `async`/`await` dans une application WPF/Avalonia, le framework WPF/Avalonia gère la distribution du délégué et le rappel à la machine à états lorsque la tâche attendue est terminée afin de mettre à jour les contrôles en toute sécurité.==***

>[!tip] Résumé du sujet
Dans le  code Avalonia, on a utilisé :  
>`Dispatcher.UIThread.Post(() => this.Title = "...");`
>
>Le `SynchronizationContext` est en fait **l'abstraction** de cela.
>
>- **Manuel** : On écrit `Dispatcher` pour forcer le retour sur l'UI.
>- **Automatique (`async/await`)** : Le `SynchronizationContext` le fait pour nous en arrière-plan. on écrit juste `this.Title = "..."` après un `await` et ça marche.

## Le rôle de `ConfigureAwait`

Maintenant que vous comprenez mieux le `SynchronizationContext`, il est temps d'aborder le rôle de la méthode `ConfigureAwait()`. **Par défaut, l'attente d'une tâche entraîne l'utilisation d'un contexte de synchronisation**. **==Lors du développement d'applications GUI==** (WinForms, WPF, Avalonia), **==;ce comportement est souhaitable==**. *==Cependant, si vous développez du code pour une application non GUI, la surcharge liée à la mise en file d'attente du contexte initial lorsqu'il n'est pas nécessaire peut potentiellement engendrer des problèmes de performance.==*

Pour observer ce comportement, modifiez vos instructions de niveau supérieur comme suit :

```cs
Console.Title = "Fun with Async";
Console.WriteLine(" Fun with Async ===>");

// Console.WriteLine(DoWork());

string message = await DoWorkASync();
Console.WriteLine($"0 - {message}");
string message1 = await DoWorkASync().ConfigureAwait(false);
Console.WriteLine($"1 - {message1}");
```

Le bloc de code original utilise le `SynchronizationContext` fourni par le framework (ici, le runtime .NET Core). Il est équivalent à l'appel de `ConfigureAwait(true)`. Le second exemple ignore le contexte actuel et le planificateur. 

**L'équipe .NET Core recommande, lors du développement de code applicatif** (WinForms, WPF, Avalonia, etc.), **de conserver le comportement par défaut**. **==Si vous écrivez du code non applicatif==** (par exemple, du code de bibliothèque), **==utilisez `ConfigureAwait(false)`==**. ***==La seule exception est ASP.NET Core (traité à partir du [[Chapitre 29|Chapitre 29]])==***. ==ASP.NET Core ne crée pas de `SynchronizationContext` personnalisé; par conséquent, `ConfigureAwait(false)` n'apporte aucun avantage lors de l'utilisation d'autres frameworks.==

## Conventions de nommage pour les méthodes asynchrones

Vous avez certainement remarqué le changement de nom de `DoWork()` à `DoWorkAsync()`, mais pourquoi ce changement ? Supposons que la nouvelle version de la méthode s’appelle toujours `DoWork()` ; cependant, le code appelant a été implémenté comme suit :

```cs
// OUPS! Pas de mot clé await ici!
string message = DoWork();
```

*==Vous avez bien marqué la méthode avec le mot-clé `async`, mais vous avez omis d'utiliser le mot-clé `await` comme décorateur avant d'appeler la méthode `DoWork()`==*. À ce stade, **vous obtiendrez des erreurs de compilation, car la valeur de retour de `DoWork()` est un objet `Task`, que vous tentez d'assigner directement à une variable de type `string`**. ***==N'oubliez pas que le jeton `await` extrait la valeur de retour interne contenue dans l'objet `Task`. Puisque vous n'avez pas utilisé ce jeton, il y a une incompatibilité de types.==***

>[!note]
>Une méthode **"attendable"** (*awaitable*) est simplement une méthode qui retourne un `Task` ou un `Task<T>`

**Étant donné que les méthodes renvoyant des objets `Task` peuvent désormais être appelées de manière non bloquante grâce aux jetons `async` et `await`, il est recommandé d'ajouter le suffixe "async" à toute méthode renvoyant un `Task`**. **==Ainsi, les développeurs connaissant cette convention de nommage disposent d'un rappel visuel indiquant que le mot-clé `await` est requis s'ils souhaitent appeler la méthode dans un contexte asynchrone.==**

>[!note]
>Les gestionnaires d'événements pour les contrôles d'interface graphique (tels qu'un gestionnaire de `Click` sur un bouton) ainsi que les méthodes d'action dans les applications de style MVC qui utilisent les mots-clés `async`/`await` ne suivent pas cette convention d'appellation (par convention — veuillez excuser la redondance !).

## Méthodes `async` ne renvoyant aucune donnée

Actuellement, votre méthode `DoWorkAsync()` renvoie un `Task<string>`, qui contient des « données réelles » pour l’appelant et qui seront obtenues de manière transparente via le mot-clé `await`. Cependant, **comment créer une méthode asynchrone qui ne renvoie rien ?** ==Bien qu’il existe deux manières de procéder, il n’y a en réalité qu’une seule bonne façon de faire==. Tout d’abord, examinons les problèmes liés à la définition d’une méthode asynchrone `void`.

### Méthodes asynchrones de type `void`

Voici un exemple de méthode asynchrone qui utilise `void` comme type de retour au lieu de `Task` :

```cs
static async void MethodReturningVoidAsync()
{
    await Task.Run(() =>
    {
        /* Travaillrer ici... */
        Thread.Sleep(4_000);
    });
    Console.WriteLine("Fire and forget void method completed");
}
```

**Si vous appeliez cette méthode, elle s'exécuterait indépendamment, sans bloquer le thread principal**. Le code suivant affichera le message "Completed" avant le message de la méthode `MethodReturningVoidAsync()` :

```cs
MethodReturningVoidAsync();
Console.WriteLine("Completed");
Console.ReadLine();
```

Bien que cela puisse sembler une option viable pour les scénarios « lancer et oublier », ==un problème plus important se pose==. *==Si la méthode lève une exception, celle-ci ne peut être traitée que dans le contexte de synchronisation de la méthode appelante==*. Mettez à jour la méthode comme suit :

```cs
static async void MethodReturningVoidAsync()
{
    await Task.Run(() =>
    {
        /* Travaillrer ici... */
        Thread.Sleep(4_000);
        throw new Exception("Something bad happened");
    });
    Console.WriteLine("Fire and forget void method completed");
}
```

Par sécurité, placez l'appel à cette méthode dans un bloc `try-catch` et exécutez à nouveau le programme :

```cs
try
{
    MethodReturningVoidAsync();
}
catch (Exception ex)
{
    Console.WriteLine(ex.Message);
}
```

*==Non seulement le bloc `catch` ne capture pas l'exception, mais celle-ci est placée dans le contexte d'exécution du thread==*. Par conséquent, même si cela peut sembler une bonne idée pour des scénarios « lancer et oublier », **il vaut mieux espérer qu'aucune exception ne soit levée dans la méthode `async void`, sous peine de voir toute votre application planter.**

### Méthodes asynchrones `void` utilisant `Task`

**La meilleure solution consiste à utiliser `Task` au lieu de `void` dans votre méthode**. Mettez à jour la méthode comme suit :

```cs
static async Task MethodReturningVoidTaskAsync()
{
    await Task.Run(() =>
    {
        /* Travailler ici... */
        Thread.Sleep(4_000);
    });
    Console.WriteLine("Void method completed");
}
```

Si vous appelez la méthode sans le mot-clé `await`, le même résultat se produira que dans l'exemple précédent :

```cs
MethodReturningVoidTaskAsync();
Console.WriteLine("Void method complete");
```

Mettez à jour la méthode `MethodReturningVoidTaskAsync()` pour qu'elle lève une exception :

```cs
static async Task MethodReturningVoidTaskAsync()
{
    await Task.Run(() =>
    {
        /* Travailler ici... */
        Thread.Sleep(4_000);
        throw new Exception("Some bad happened");
    });
    Console.WriteLine("Void method completed");
}
```

Encadrez maintenant l'appel à cette méthode dans un bloc `try-catch` :

```cs
try
{
    MethodReturningVoidTaskAsync();
}
catch (Exception ex)
{
    Console.WriteLine(ex.Message);
}
```

**Lorsque vous exécutez le programme et qu'une exception est levée, deux choses intéressantes se produisent**. Premièrement, **==le programme ne plante pas complètement==**, et deuxièmement, **==le bloc catch n'intercepte pas l'exception==**. **Lorsqu'une exception est levée par une méthode `Task`/`Task<T>`, elle est capturée et placée dans l'objet `Task`**. ***==Lors de l'utilisation de `await`, l'exception==*** (ou `AggregateException`) ***==est disponible pour être gérée.==***

Mettez à jour le code appelant pour qu'il attende (`await`) la méthode, et le bloc `catch` fonctionnera désormais comme prévu :

```cs
try
{
    await MethodReturningVoidTaskAsync();
}
catch (Exception ex)
{
    Console.WriteLine(ex.Message);
}
```

**En résumé, évitez de créer des méthodes asynchrones de type `void` et utilisez plutôt des méthodes asynchrones de type `Task`**. ***==Le choix de les attendre ou non relève de votre stratégie métier, mais dans tous les cas, au moins votre application ne plantera pas !==***

## Méthodes asynchrones avec plusieurs attentes

Il est tout à fait possible qu'une méthode `async` contienne plusieurs `await` dans son implémentation. L'exemple suivant illustre ce comportement :

```cs
static async Task MultipleAwaits()
{
    await Task.Run(() =>
    {
        Thread.Sleep(2_000);
    });
    Console.WriteLine("Done with the first task!");

    await Task.Run(() =>
    {
        Thread.Sleep(2_000);
    });
    Console.WriteLine("Done with the second task!");

    await Task.Run(() =>
    {
        Thread.Sleep(2_000);
    });
    Console.WriteLine("Done with the third task!");
}
```

Ici encore, **chaque tâche se contente de suspendre temporairement le thread courant; cependant, n'importe quelle unité de travail pourrait être représentée par ces tâches** (appel d'un service web, lecture d'une base de données, etc.).

**==Une autre option consiste à ne pas attendre chaque tâche individuellement, mais à les attendre toutes ensemble et à retourner une fois toutes les tâches terminées==**. **Ce scénario est plus probable, par exemple lorsqu'il y a trois actions** (vérification des e-mails, mise à jour du serveur, téléchargement de fichiers) **qui doivent être effectuées par lots, mais qui peuvent être réalisées en parallèle**. Voici le code mis à jour à l'aide de la méthode `Task.WhenAll()` :

```cs
static async Task MultipleAwaitsAsync()
{
    await Task.WhenAll(
        Task.Run(() =>
        {
            Thread.Sleep(2_000);
            Console.WriteLine("Done with the first task!");
        }),
        Task.Run(() =>
        {
            Thread.Sleep(2_000);
            Console.WriteLine("Done with the first task!");
        }),
        Task.Run(() =>
        {
            Thread.Sleep(2_000);
            Console.WriteLine("Done with the first task!");
        })
    );
}
```

Lorsque vous exécutez le programme maintenant, vous constatez que les trois tâches s'exécutent dans l'ordre décroissant du temps de veille.

```
Fun with Async ===>
...
Done with the second task!
Done with the third task!
Done with the first task!
```

**Il existe également une méthode `WhenAny()`, qui signale qu'une des tâches est terminée**. ==Cette méthode renvoie la première tâche terminée==. Pour illustrer l'utilisation de `WhenAny()`, modifiez la dernière ligne de `MultipleAwaitsAsync` comme suit :

```cs
static async Task MultipleAwaitsAsync()
{
    await Task.WhenAny(
        Task.Run(() =>
        {
            Thread.Sleep(2_000);
            Console.WriteLine("Done with the first task!");
        }),
        Task.Run(() =>
        {
            Thread.Sleep(1_000);
            Console.WriteLine("Done with the second task!");
        }),
        Task.Run(() =>
        {
            Thread.Sleep(1_000);
            Console.WriteLine("Done with the third task!");
        })
    );
}
```

Lorsque vous faites cela, le résultat devient ceci :

```
Fun with Async ===>
Done with the third task!
Completed
Done with the second task!
Done with the first task!
```

>[!info] 
>Les ordinateurs actuels sont si rapide et .NET est tellement optimisé maintenant que les deux tâche de 1s finissend presque au même instant (à la nanoseconde). Le temps que `WhenAny` se réveille, les deux message sont déja dans la console

**Chacune de ces méthodes fonctionne également avec un tableau de tâches**. Pour illustrer cela, créez une nouvelle méthode nommée `MultipleAwaitsTake2Async()`. Dans cette méthode, créez une `List<Task>`, ajoutez-y les trois tâches, puis appelez `Task.WhenAll()` ou `Task.WhenAny()`.

```cs
static async Task MultipleAwaitsTake2Async()
{
    var tasks = new List<Task>();
    tasks.Add(
        Task.Run(() =>
        {
            Thread.Sleep(2_000);
            Console.WriteLine("Done with the first task!");
        })
    );
    tasks.Add(
        Task.Run(() =>
        {
            Thread.Sleep(1_000);
            Console.WriteLine("Done with the second task!");
        })
    );
    tasks.Add(
        Task.Run(() =>
        {
            Thread.Sleep(1_000);
            Console.WriteLine("Done with the third task!");
        })
    );

    // await Task.WhenAny(tasks)
    await Task.WhenAll(tasks);
}
```

## Appel de méthodes asynchrones depuis des méthodes synchrones

Chacun des exemples précédents utilisait le mot-clé `async` pour renvoyer le thread au code appelant pendant l'exécution de la méthode asynchrone. **Pour rappel, le mot-clé `await` ne peut être utilisé que dans une méthode marquée comme `async`**. ==Que faire si vous ne pouvez pas== (ou ne souhaitez pas) ==marquer une méthode comme asynchrone ?==

Heureusement,**il existe des moyens d'appeler des méthodes asynchrones dans un contexte synchrone**. *==Malheureusement, la plupart sont à éviter==*. ==La première option consiste simplement à omettre le mot-clé `await`, permettant ainsi au thread d'origine de poursuivre son exécution pendant que la méthode asynchrone s'exécute sur un thread séparé, sans jamais renvoyer de résultat à l'appelant==. **==Ce comportement est similaire à l'exemple précédent d'appel de méthodes `Task` asynchrones==**. *==Toutes les valeurs renvoyées par la méthode sont perdues et les exceptions sont ignorées.==*

Cette solution peut convenir à vos besoins. ==Sinon, trois options s'offrent à vous==. La première consiste à **utiliser la propriété `Result` de l'objet `Task<T>` ou la méthode `Wait()` des méthodes `Task`**. ==Si la méthode échoue, toute exception est encapsulée dans une `AggregateException`, ce qui peut compliquer la gestion des erreurs==. **Vous pouvez également appeler `GetAwaiter().GetResult()`.** **==Cette méthode se comporte de la même manière que les appels à `Wait()` et `Result`, à la légère différence près : les exceptions ne sont pas encapsulées dans une `AggregateException`==**. Bien que les méthodes `GetAwaiter().GetResult()` fonctionnent aussi bien avec les méthodes renvoyant une valeur qu'avec celles n'en renvoyant pas, *==elles sont signalées dans la documentation comme « non destinées à un usage externe », ce qui signifie qu'elles pourraient être modifiées ou supprimées ultérieurement.==*

**Bien que ces deux options semblent être des alternatives inoffensives à l'utilisation de `await` dans une méthode asynchrone, leur utilisation présente un problème plus grave**. *==Appeler `Wait()`, `Result` ou `GetAwaiter().GetResult()` bloque le thread appelant, traite la méthode asynchrone sur un autre thread, puis revient au thread appelant, mobilisant deux threads pour effectuer le travail==*. Pire encore, *==chacune de ces opérations peut provoquer des blocages, surtout si le thread appelant provient de l'interface utilisateur de l'application.==*

**==Pour faciliter la détection et la correction du code `async`/`await` incorrect (et des conventions de nommage inappropriées), ajoutez le package `Microsoft.VisualStudio.Threading.Analyzers` au projet==**. **Ce package ajoute des analyseurs qui généreront des avertissements du compilateur en cas de code de gestion des threads incorrect, notamment des conventions de nommage inappropriées**. Pour observer ce comportement, ajoutez le code suivant aux instructions de niveau supérieur :

```cs
_ = DoWorkASync().Result;
_ = DoWorkASync().GetAwaiter().GetResult();
```

Cela provoque l'avertissement suivant du compilateur :

```
VSTHRD002 Synchronously waiting on tasks or awaiters may cause deadlocks. Use await or
JoinableTaskFactory.Run instead.
```

>[!info] Le message d'avertissement (ainsi que le code) généré n'est plus le même sur les version plus moderne.

***==Non seulement un avertissement du compilateur est affiché, mais la solution recommandée est également fournie !==*** Pour utiliser la classe `JoinableTaskFactory`, vous devez ajouter le package NuGet `Microsoft.VisualStudio.Threading` et l'instruction using suivante en haut du fichier *Program.cs* :

>[!warning]- Les packages présenté et leurs utilisation dans un environnement .NET moderne
>
>Le package `Microsoft.VisualStudio.Threading.Analyzers` est considéré comme **best practice**. 
>
>Les analyseurs de base de .NET (Roslyn) détectent les erreurs flagrantes, mais le package `Microsoft.VisualStudio.Threading.Analyzers` va beaucoup plus loin. Il détecte des erreurs subtiles que le compilateur standard ignore.
>- **L'oubli de `ConfigureAwait(false)`** : Très important pour éviter les deadlocks dans les bibliothèques.
>- **Le "Async Void"** : Il vous grondera si vous créez une méthode `async void` (sauf pour les gestionnaires d'événements).
>- **Le nommage** : Il vous rappellera d'ajouter le suffixe `Async` à vos méthodes.
>- **Les blocages cachés** : Il détectera si vous utilisez `.Result` ou `.Wait()` au lieu de `await`.
>
>Aujourd'hui, .NET intègre nativement de plus en plus d'analyseurs. Cependant, le package de Visual Studio reste la **référence absolue** pour le code professionnel car il est plus strict
>
>---
>la classe **`JoinableTaskFactory`** (JTF) venant de `Microsoft.VisualStudio.Threading` est un outil très spécifique.
>
>**Dans l'écosystème Visual Studio / WPF**, c'est la solution standard si vous écrivez des extensions pour Visual Studio ou si vous travaillez sur de vieux projets WPF très complexes.
>
>**Dans le développement .NET moderne (Avalonia, ASP.NET Core)**,  *==on essaie de l'éviter.==*
>
>En 2026, la philosophie de .NET est le **"Async all the way"** (l'asynchronisme de bout en bout).
>
>- **Le danger** : Utiliser JTF ajoute une couche de complexité énorme à votre projet.
>- **La réalité** : Dans 99 % des cas, vous pouvez simplement rendre votre méthode parente `async` et utiliser `await`. C'est plus propre, plus lisible et plus performant.

```cs
using Microsoft.VisualStudio.Threading;
```

La classe `JoinableTaskFactory` a besoin d'un `JoinableTaskContext` dans son constructeur :

```cs
JoinableTaskFactory joinableTaskFactory = new JoinableTaskFactory(
    new JoinableTaskContext()
);
```

Grâce à cela, vous pouvez utiliser la méthode `Run()` pour exécuter en toute sécurité une méthode asynchrone, telle que la méthode `DoWork()`, depuis un contexte synchrone :

```cs
string message2 = joinableTaskFactory.Run(async () => await DoWorkAsync());
```

Comme vous le savez, la méthode `DoWork()` renvoie `Task<string>`, et cette valeur est bien renvoyée par la méthode `Run()`. **Vous pouvez également appeler des méthodes qui renvoient simplement `Task`**, comme suit :

```cs
joinableTaskFactory.Run(async () =>
{
	await MethodReturningVoidTaskAsync();
	await SomeOtherAsyncMethod();
});
```

>[!note]
>Bien que ces packages contiennent « Visual Studio » dans leur nom, ils ne dépendent pas de Visual Studio. Ce sont des packages .NET qui peuvent être utilisés avec ou sans Visual Studio installé.

## Utilisation de `await` dans les blocs `catch` et `finally`

**C# 6 a introduit la possibilité de placer des appels `await` dans les blocs `catch` et `finally`**. ***==La méthode elle-même doit être `async` pour ce faire==***. L'exemple de code suivant illustre cette fonctionnalité :

```cs
static async Task<string> MethodWithTryCatch()
{
    try
    {
        // Travaille.
        return "Hello";
    }
    catch (Exception ex)
    {
        await LogTheError();
        throw;
    }
    finally
    {
        await DoMagicCleanUp();
    }
}
```

## Types de retour asynchrones généralisés (Nouveauté C# 7.0)

Avant C# 7, les seules options de retour pour les méthodes asynchrones étaient `Task`, `Task<T>` et `void`. **==C# 7 autorise des types de retour supplémentaires, s'ils suivent le modèle asynchrone==**. **`ValueTask` en est un exemple concret**. Pour voir cela en pratique, créez un code comme celui-ci :

```cs
static async ValueTask<int> ReturnAnInt()
{
    await Task.Delay(1_000);
    return 5;
}
```

**Les mêmes règles s'appliquent à `ValueTask` qu'à `Task`, car `ValueTask` est simplement une tâche pour les types valeur au lieu de forcer l'allocation d'un objet sur le tas.**

## Fonctions locales avec `async`/`await` (Nouveauté C# 7.0)

Les fonctions locales ont été introduites au [[Chapitre 4#Comprendre les fonctions locales (Nouveauté C 7.0, MaJ C 9.0)|Chapitre 4]] et utilisées tout au long de cet ouvrage. Elles peuvent également s'avérer utiles pour les méthodes asynchrones. Pour en démontrer l'intérêt, il faut d'abord identifier le problème. Ajoutez une nouvelle méthode nommée `MethodWithProblems()` et insérez le code suivant :

```cs
static async Task MethodWithProblems(int firstParam, int secondParam)
{
    Console.WriteLine("Enter");
    await Task.Run(() =>
    {
        // Appelle une méthode qui dure longtemps
        Thread.Sleep(4_000);
        Console.WriteLine("First Complete");
        // Appelle une autre méthode qui dure longtems qui échoue
        // parce que le second paramètre est hors de portée.
        Console.WriteLine("Something bad happened");
    });
}
```

Le scénario est le suivant : ==la seconde tâche de longue durée échoue en raison de données d'entrée invalides==. **Vous pouvez (et devriez) ajouter des vérifications au début de la méthode, mais ==comme celle-ci est entièrement asynchrone, il n'y a aucune garantie quant au moment où les vérifications seront exécutées==**. **Il serait préférable que les vérifications soient effectuées immédiatement avant que le code appelant poursuive son exécution**. Dans la mise à jour suivante, les vérifications sont effectuées de manière synchrone, puis la fonction privée est exécutée de manière asynchrone :

```cs
static async Task MethodWithProblemsFixed(int firstParam, int secondParam)
{
    Console.WriteLine("Enter");
    if (secondParam < 0)
    {
        Console.WriteLine("Bad data");
        return;
    }

    await ActualImplementation();

    async Task ActualImplementation()
    {
        // Appelle une méthode qui dure longtemps
        Thread.Sleep(4_000);
        Console.WriteLine("First Complete");

        // Appelle une autre methode qui dure longtemps qui reussit.
        Thread.Sleep(1_000);
        Console.WriteLine("Second Complete");
    }
}
```

## Annulation des opérations `async`/`await`

L'annulation est également possible avec le modèle `async`/`await` et beaucoup plus simple qu'avec `Parallel.ForEach`. Pour illustrer cela, nous utiliserons le même projet WPF/Avalonia que précédemment dans ce chapitre. Vous pouvez soit réutiliser ce projet, soit ajouter une nouvelle application WPF (.NET Core)/Avalonia à la solution et ajouter le package nécessaire au projet selon le framework en exécutant les commandes CLI suivantes :

#### WPF

```
dotnet new wpf -n PictureHandlerWithAsyncAwait -o .\PictureHandlerWithAsyncAwait
dotnet sln Chapter15_AllProjects.sln add PictureHandlerWithAsyncAwait
dotnet add PictureHandlerWithAsyncAwait package System.Drawing.Common
```

#### Avalonia

```
dotnet new Avalonia.app -n PictureHandlerWithAsyncAwait
dotnet sln Chapter15_AllProjects.sln add PictureHandlerWithAsyncAwait
dotnet add PictureHandlerWithAsyncAwait package SixLabors.ImageSharp
```

#### Commun

Si vous utilisez Visual Studio, procédez comme suit : cliquez avec le bouton droit sur le nom de la solution dans l’Explorateur de solutions, sélectionnez Ajouter -> Projet, puis nommez-le *PictureHandlerWithAsyncAwait*. Assurez-vous de définir le nouveau projet comme projet de démarrage en cliquant avec le bouton droit sur son nom et en sélectionnant Définir comme projet de démarrage dans le menu contextuel. Ajoutez ensuite le package NuGet nécessaire au traitement de l'image selon le framework. (voir commandes précédentes)

Remplacez le XAML/AXAML pour qu'il corresponde au projet WPF/Avalonia précédent (*DataParallelismWithForEach*), à l'exception du titre : remplacez-le par `"Picture Handler with Async/Await"`. Veillez également à mettre à jour le `PropertyGroup` principal dans le fichier projet afin de désactiver les types de référence nullables.

```xml
<PropertyGroup>
	...
	<Nullable>disable</Nullable>
	...
</PropertyGroup>
```

#### WPF

Dans le fichier *MainWindow.xaml.cs*, assurez-vous que les instructions `using` suivantes sont en place :

```cs
using System.IO;
using System.Windows;
using System.Drawing;
```

#### Avalonia

Dans le fichier *MainWindow.axaml.cs*, assurez-vous que les instructions `using` suivantes sont en place :

```cs
using System.IO;
using System.Reflection;
using Avalonia.Interactivity;
using SixLabors.ImageSharp;
using SixLabors.ImageSharp.Processing;
// Conflit de nom entre Avalonia.Image et SixLabors.ImageSharp.Image
using SharpImage = SixLabors.ImageSharp.Image;
```

#### Commun

Ensuite, ajoutez une variable de classe pour `CancellationToken` et ajoutez le gestionnaire d'événements du bouton "Cancel" :

>[!warning] Attention
>Dans les paramètres des méthodes `cmdCancel_Click` et `cmdProcess_Click`:
>- **WPF**: `EventArgs e`
>- **Avalonia**: `RoutedEventArgs e` 

```cs
private CancellationTokenSource _cancelToken = null;

private void cmdCancel_Click(object sender, RoutedEventArgs e)
{
  _cancelToken.Cancel();
}
```

**==Le processus est identique à l'exemple précédent==** : récupérer le répertoire des images, créer le répertoire de sortie, puis récupérer les fichiers image, les faire pivoter et les enregistrer dans le nouveau répertoire. **Au lieu d'utiliser `Parallel.ForEach()`, cette nouvelle version utilise des méthodes asynchrones, dont la signature accepte un `CancellationToken` en paramètre**. Saisissez le code suivant :

>[!tip] La gestion des exception est un petit peu différente par rapport au livre (affichage plus propre des annulation dans le terminal).

```cs
private async void cmdProcess_Click(object sender, RoutedEventArgs e)
{
	_cancelToken = new CancellationTokenSource();

	string projectName = Assembly.GetExecutingAssembly().GetName().Name;

	var basePath = Directory.GetCurrentDirectory();
	var pictureDirectory = Path.Combine(basePath, projectName!, "assets");
	var outputDirectory = Path.Combine(
		basePath,
		projectName,
		"modifiedAssets"
	);

	// Nettoie le dossier si il existe déjà.
	if (Directory.Exists(outputDirectory))
	{
		Directory.Delete(outputDirectory, true);
	}
	// Recrée le dossier de sortie des images
	Directory.CreateDirectory(outputDirectory);

	// Récupère tous les fichier .png contenu dans le dossier (récursif).
	string[] files = Directory.GetFiles(
		pictureDirectory,
		"*.png",
		SearchOption.AllDirectories
	);

	try
	{
		foreach (string currentFile in files)
		{
			try
			{
				await ProcessFileAsync(
					currentFile,
					outputDirectory,
					_cancelToken.Token
				);
			}
			catch (OperationCanceledException)
			{
				throw;
			}
		}
	}
	catch (OperationCanceledException ex)
	{
		Console.WriteLine(ex.Message);
	}
	catch (Exception ex)
	{
		Console.WriteLine(ex.Message);
		throw;
	}

	_cancelToken = null;
	this.Title = "Processing complete";
}
```

**Après la configuration initiale, le code parcourt les fichiers et appelle `ProcessFileAsync()` de manière asynchrone pour chaque fichier**. L'appel à `ProcessFileAsync()` est placé dans un bloc `try`/`catch`, et le `CancellationToken` est transmis à la méthode `ProcessFile()`. Si `Cancel()` est exécuté sur le `CancellationTokenSource` (par exemple, lorsque l'utilisateur clique sur le bouton "Cancel"), une exception `OperationCanceledException` est levée.

>[!note] 
>Le bloc `try`/`catch` peut se trouver n'importe où dans la chaîne d'appels (comme vous le verrez bientôt). Le choix de le placer au premier appel ou dans la méthode asynchrone elle-même dépend des préférences et des besoins de l'application.

La dernière méthode à ajouter est la méthode `ProcessFileAsync()`.

>[!warning] Petite différence entre WPF et Avalonia
>- **WPF** : `Dispatcher?.Invoke()`
>- **Avalonia**: `Dispatcher.UIThread.Post()`

```cs
private async Task ProcessFileAsync(
	string currentFile,
	string outputDirectory,
	CancellationToken token
)
{
	string filename = Path.GetFileName(currentFile);

	using (SharpImage image = SharpImage.Load(currentFile))
	{
		try
		{
			await Task.Run(
				() =>
				{
					Dispatcher.UIThread.Post(() =>
					{
						this.Title = $"Processing {filename}";
					});

					image.Mutate(x => x.Flip(FlipMode.Horizontal));
					image.Save(Path.Combine(outputDirectory, filename));

					// Simule une charge te travail plus importante
					// (pour voir les bonds en performances)
					Thread.Sleep(2000);
				},
				// On transmet le token dans la tâche (garder la chaîne)
				token
			);
		}
		catch (OperationCanceledException)
		{
			throw;
		}
	}
}
```

**Cette méthode utilise une autre surcharge de la commande `Task.Run`, prenant le `CancellationToken` comme paramètre**. La commande `Task.Run` est placée dans un bloc `try`/`catch` (comme le code appelant) au cas où l'utilisateur cliquerait sur le bouton "Cancel".

### Annulation des opérations `async`/`await` avec `WaitAsync()` (Nouveauté C# 10.0)

Nouveauté de C# 10.0 : **les appels asynchrones peuvent être annulés à l’aide d’un jeton d’annulation et/ou après l’expiration d’un délai, grâce à la méthode `WaitAsync()`**. Les exemples suivants (==situés dans le projet *FunWithCSharpAsync* du code de ce chapitre==) illustrent les trois cas d’utilisation de cette nouvelle fonctionnalité :

```cs
CancellationTokenSource tokenSource = new CancellationTokenSource();
_ = await DoWorkAsync().WaitAsync(TimeSpan.FromSeconds(10));
_ = await DoWorkAsync().WaitAsync(tokenSource.Token);
_ = await DoWorkAsync().WaitAsync(TimeSpan.FromSeconds(10), tokenSource.Token);
```

### Annulation des opérations `async`/`await` dans les appels synchrones

**La méthode `Wait()` peut également accepter un jeton d'annulation lors de l'appel de méthodes asynchrones depuis une méthode non asynchrone**. ==Ceci peut être utilisé avec ou sans délai d'attente. Si un délai d'attente est spécifié, il doit être exprimé en millisecondes.==

```cs
CancellationTokenSource tokenSource = new CancellationTokenSource();
MethodReturningTaskOfVoidAsync().Wait(tokenSource.Token);
MethodReturningTaskOfVoidAsync().Wait(10000,tokenSource.Token);
```

Vous pouvez également utiliser `JoinableTaskFactory` et la nouvelle méthode `WaitAsync()` lors d'un appel depuis du code synchrone :

```cs
JoinableTaskFactory joinableTaskFactory2 = new JoinableTaskFactory(
    new JoinableTaskContext()
);
CancellationTokenSource tokenSource2 = new CancellationTokenSource();
joinableTaskFactory2.Run(async () =>
{
    await MethodReturningVoidTaskAsync().WaitAsync(tokenSource2.Token);
    await MethodReturningVoidTaskAsync()
        .WaitAsync(TimeSpan.FromSeconds(10), tokenSource2.Token);
});
```

## Flux asynchrones (Nouveauté C# 8.0)

**Nouveauté de C# 8.0 : les flux** (traités plus en détail au [[Chapitre 19|Chapitre 19]]) **peuvent être créés et consommés de manière asynchrone**. Une méthode qui renvoie un flux asynchrone

- Est déclarée avec le modificateur `async`
- Renvoie un `IAsyncEnumerable<T>`
- Contient des instructions `yield return` (traitées au [[Chapitre 8#Création de méthodes d'itération avec le mot-clé `yield`|Chapitre 8]]) pour renvoyer successivement les éléments du flux asynchrone

Prenons l'exemple suivant :

```cs
static async IAsyncEnumerable<int> GenerateSequence()
{
    for (int i = 0; i < 20; i++)
    {
        await Task.Delay(100);
        yield return i;
    }
}
```

**==La méthode est déclarée comme asynchrone, renvoie un `IAsyncEnumerable<int>` et utilise `yield return` pour renvoyer les entiers d'une séquence==**. Pour appeler cette méthode, ajoutez ce qui suit à votre code appelant :

```cs
await foreach (var number in GenerateSequence())
{
    Console.WriteLine(number);
}
```

## La méthode `Parallel.ForEachAsync()` (Nouveauté C# 10.0)

**Nouveauté de C# 10 : la classe `Parallel` possède une nouvelle méthode `ForEachAsync()`, asynchrone** (comme son nom l’indique), **qui fournit également une méthode asynchrone pour le corps de la requête**. ==Dans le projet *DataParallelismWithForEach*, créez une nouvelle méthode asynchrone nommée `ProcessFilesAsync()` et copiez le code de la méthode `ProcessFiles()`.

**Une fois le code copié, remplacez l’appel précédent à `Parallel.ForEach()` par `await Parallel.ForEach()`**. ==Les deux premiers paramètres (`files`, `parOpts`) sont identiques dans les deux appels==. ***==La différence réside dans le corps de la requête, qui prend deux paramètres : le fichier courant et le jeton d’annulation.==***

```cs
// Vieux code:
//Parallel.ForEach(files, parOpts, currentFile =>
// Nouveau code:
await Parallel.ForEachAsync(files, parOpts, async (currentFile, token) =>
```

**Le jeton d'annulation transmis dans le corps de la requête est identique à celui de l'instance `ParallelOptions`**. **==Cela signifie que nous pouvons vérifier le paramètre de jeton pour les requêtes d'annulation demandées, au lieu de celui de l'instance `ParallelOptions`==** :

```cs
// Vieux code:
//parOpts.CancellationToken.ThrowIfCancellationRequested();
// Nouveau code:
token.ThrowIfCancellationRequested();
```

La méthode complète est présentée ici :

>[!warning] Se référer aux section où l'on a créer les méthodes pour voir les différences entre **WPF** et **Avalonia**.

```cs
private async Task ProcessFilesAsync()
{
	// Charge tous les fichiers *.png et crée un nouveau dossier pour les
	// données modifiées.
	string? projectName = Assembly.GetExecutingAssembly().GetName().Name;
	var basePath = Directory.GetCurrentDirectory();
	var pictureDirectory = Path.Combine(basePath, projectName!, "assets");
	var outputDirectory = Path.Combine(
		basePath,
		projectName!,
		"modifiedAssets"
	);

	// Nettoie le dossier si il existe déjà.
	if (Directory.Exists(outputDirectory))
	{
		Directory.Delete(outputDirectory, true);
	}
	// Recrée le dossier de sortie des images
	Directory.CreateDirectory(outputDirectory);
	
	// Récupère tous les fichier .png contenu dans le dossier (récursif).
	string[] files = Directory.GetFiles(
		pictureDirectory,
		"*.png",
		SearchOption.AllDirectories
	);
	
	// Utilise une instance de ParallelOptions
	// por stocker le CancellationToken
	ParallelOptions parOpts = new ParallelOptions();
	parOpts.CancellationToken = _cancelToken.Token;
	parOpts.MaxDegreeOfParallelism = Environment.ProcessorCount;

	try
	{
		// Traiter les données d'image en parallèle !
		await Parallel.ForEachAsync(
			files,
			parOpts,
			async (currentFile, token) =>
			{
				// Génère l'exception d'annulage.
				token.ThrowIfCancellationRequested();

				string filename = Path.GetFileName(currentFile);

				Dispatcher.UIThread.Post(() =>
				{
					this.Title =
						$"Processing {filename} on thread {Environment.CurrentManagedThreadId}";
				});

				using (SharpImage image = SharpImage.Load(currentFile))
				{
					image.Mutate(x => x.Flip(FlipMode.Horizontal));
					image.Save(Path.Combine(outputDirectory, filename));
				}

				// Simule une charge te travail plus importante
				// (pour voir les bonds en performances)
				Thread.Sleep(2000);
			}
		);
		Dispatcher.UIThread.Post(() => this.Title = "Done!");
	}
	catch (OperationCanceledException ex)
	{
		Dispatcher.UIThread.Post(() => this.Title = ex.Message);
	}
}
```

Après cela, pour appeler la méthode `ProcessFileAsync()` :

```cs
private void cmdProcess_Click(object? sender, RoutedEventArgs e)
{
	// Réinitialiser le token pour permettre un nouveau cycle
	_cancelToken = new CancellationTokenSource();

	this.Title = $"Starting...";
	// Lancer une nouvelle « tâche » pour traiter les fichiers.
	// Task.Factory.StartNew(ProcessFiles);
	ProcessFilesAsync().Wait();
	// Peut également s'écrire ainsi :
	//Task.Factory.StartNew(() => ProcessFiles());
	//this.Title = "Processing Complete";
}
```

## Mise à jour de l'application de lecture de livres avec `async`/`await`

**Maintenant que vous comprenez le modèle `async`/`await`, mettons à jour l'application de lecture de livres pour utiliser `HttpClient` au lieu de `WebClient`. Créez une nouvelle méthode nommée `GetBookAsync()`.**

```cs
async Task GetBookAsync()
{
    HttpClient client = new HttpClient();
    _theEBook = await client.GetStringAsync(
        "https://www.gutenberg.org/files/98/98-0.txt"
    );
    Console.WriteLine("Download complete.");
    GetStats();
}
```

**==Après avoir créé une nouvelle instance de la classe `HttpClient`, le code appelle la méthode `GetStringAsync()` pour effectuer une requête "HTTP GET" afin de récupérer le texte depuis le site web==**. Comme vous pouvez le constater, **`HttpClient` offre une méthode beaucoup plus concise pour effectuer des requêtes HTTP**. ***==La classe `HttpClient` sera étudiée plus en détail dans les chapitres consacrés à ASP.NET Core.==***

## Récapitulatif sur async et await

Cette section contenait de nombreux exemples; voici les points clés :

- Les méthodes (ainsi que les expressions lambda ou les méthodes anonymes) peuvent être marquées avec le mot-clé `async` pour permettre à la méthode de s'exécuter de manière non bloquante.

- ***==Les méthodes (ainsi que les expressions lambda ou les méthodes anonymes) marquées avec le mot-clé `async` s'exécutent de manière synchrone jusqu'à ce que le mot-clé `await` soit rencontré.==***

- Une seule méthode `async` peut avoir plusieurs contextes `await`.

- **Lorsque l'expression `await` est rencontrée, le thread appelant est suspendu jusqu'à ce que la tâche attendue soit terminée**. Entre-temps, le contrôle est rendu à l'appelant de la méthode.

- Le mot-clé `await` masque l'objet `Task` retourné, donnant l'impression de retourner directement la valeur de retour sous-jacente. Les méthodes sans valeur de retour retournent simplement `void`.

- La vérification des paramètres et la gestion des erreurs doivent être effectuées dans la section principale de la méthode, la partie asynchrone étant déplacée dans une fonction privée. 

- **==Pour les variables de pile, l'objet `ValueTask` est plus efficace que l'objet `Task`, qui peut entraîner des opérations de boxing et d'unboxing.==**

- ***==Par convention, les méthodes appelées de manière asynchrone doivent être précisées avec le suffixe "Async"==***.

# Résumé du chapitre

Ce chapitre a débuté par l'examen du rôle de l'espace de noms `System.Threading`. Comme vous l'avez appris, **lorsqu'une application crée des threads d'exécution supplémentaires, le programme en question peut effectuer de nombreuses tâches simultanément** (en apparence). **==Vous avez également examiné plusieurs méthodes permettant de protéger les blocs de code sensibles aux threads afin d'éviter que les ressources partagées ne deviennent des unités inutilisables contenant des données erronées.==**

Ce chapitre a ensuite examiné de **nouveaux modèles de développement multithread introduits avec .NET 4.0, notamment la bibliothèque parallèle de tâches (TPL) et PLINQ**. J'ai conclu ce chapitre en abordant le rôle des mots clés `async` et `await`. **Comme vous l'avez constaté, ces mots clés utilisent de nombreux types du framework TPL en arrière-plan; cependant, le compilateur effectue la majeure partie du travail de création du code complexe de gestion des threads et de synchronisation.**

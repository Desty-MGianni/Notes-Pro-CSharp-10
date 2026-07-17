---
title: "Chapitre 19: Entrées/Sorties de Fichiers et Sérialisation d'Objets"
publish: true
---
# <big><big><big><b><font color =green>Entrées/Sorties de Fichiers et Sérialisation d'Objets</font></b></big></big></big>

>[!important] Tri précis entre ce qui est obsolète et ce qui est toujours recommandé (Gemini)
>
>## Ce qui est obsolète (À NE PLUS UTILISER)
>
>- **`BinaryFormatter` :** Si le livre vous montre comment sérialiser un objet en binaire avec cette classe, ignorez-le. Elle a été **définitivement supprimée et totalement bloquée** de .NET en raison de failles de sécurité critiques et insolubles. Elle ne fonctionnera plus du tout dans vos projets modernes.
>- **`SoapFormatter` :** Totalement obsolète. Le format SOAP n'est plus utilisé dans l'écosystème .NET moderne pour la sérialisation d'objets locaux.
>
> ## Ce qui est hérité mais à éviter (Legacy)
>
>- **`XmlSerializer` :** Toujours fonctionnel et utile si vous devez interagir avec d'anciens systèmes XML. Cependant, évitez de l'utiliser pour de nouveaux projets.
>- **`System.Runtime.Serialization` (DataContractSerializer) :** Utilisé principalement à l'époque de WCF. Il est aujourd'hui relégué au second plan.
>
>## Ce qui est ultra-pertinent et moderne
>
>- **L'espace de noms `System.IO` :** Les classes fondamentales comme `File`, `Directory`, `Path`, `FileStream`, `StreamReader`, et `StreamWriter` n'ont pas changé. Elles constituent toujours la base de la manipulation de fichiers en C#.
>- **`System.Text.Json` :** C'est le roi absolu de la sérialisation moderne dans .NET. Si le livre aborde cette classe à la fin du chapitre, concentrez toute votre attention dessus. Elle est optimisée, sécurisée, et hautement compatible avec le déploiement Native AOT via les fameux _Source Generators_.
>
>---
>
*Apprenez la gestion des flux de fichiers (`Streams`) de ce chapitre, mais faites l'impasse sur la sérialisation binaire. Utilisez exclusivement `System.Text.Json` pour sauvegarder vos objets sur le disque.*

Lors de la création d'applications de bureau, la possibilité d'enregistrer des informations entre les sessions utilisateur est courante. **Ce chapitre examine plusieurs sujets liés aux entrées/sorties (E/S) du point de vue du framework .NET.** ***==La première étape consiste à explorer les types principaux définis dans l'espace de noms `System.IO` et à apprendre à modifier par programmation la structure des répertoires et des fichiers d'une machine==***. **==L'étape suivante consiste à explorer différentes méthodes de lecture et d'écriture dans des magasins de données de type caractère, binaire, chaîne de caractères et en mémoire.==**

**Après avoir appris à manipuler les fichiers et les répertoires à l'aide des types d'E/S principaux, vous examinerez la sérialisation d'objets**. ***==Vous pouvez utiliser la sérialisation d'objets pour conserver et récupérer l'état d'un objet vers (ou depuis) ​​n'importe quel type dérivé de `System.IO.Stream`.==***

>[!note] Seulement pour Windows
>Pour pouvoir exécuter chacun des exemples de ce chapitre, démarrez Visual Studio avec des droits d'administrateur (cliquez avec le bouton droit sur l'icône Visual Studio et sélectionnez « Exécuter en tant qu'administrateur »). Si vous ne le faites pas, vous risquez de rencontrer des exceptions de sécurité lors de l'accès au système de fichiers de l'ordinateur.

# Exploration de l'espace de noms `System.IO`

>[!important] 
>Les assemblies utilisé dans les versions de .NET moderne sont différents (Voir [le code source de .NET](https://source.dot.net/)).

**Dans le framework .NET Core, l'espace de noms `System.IO` est la partie des bibliothèques de classes de base dédiée aux services d'entrée/sortie (E/S) basés sur les fichiers (et la mémoire)**. Comme tout espace de noms, `System.IO` définit un ensemble de classes, d'interfaces, d'énumérations, de structures et de délégués, dont la plupart se trouvent dans *mscorlib.dll*. Outre les types contenus dans *mscorlib.dll*, l'assembly *System.dll* définit des membres supplémentaires de l'espace de noms `System.IO`.

**De nombreux types de l'espace de noms `System.IO` permettent la manipulation programmatique de répertoires et de fichiers physiques. Cependant, d'autres types prennent en charge la lecture et l'écriture de données dans des tampons de chaînes (*string buffers*), ainsi que dans des emplacements mémoire bruts.** Le [[#Tableau 19-1 Membres clé de l'espace de noms `System.IO`|Tableau 19-1]] présente les classes principales (non abstraites), offrant une vue d'ensemble des fonctionnalités de `System.IO`.

##### Tableau 19-1: Membres clé de l'espace de noms `System.IO`

| Type de classes E/S non abstraites | Description                                                                                                                                                                                                                                                                  |
| ---------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `BinaryReader`<br>`BinaryWriter`   | Ces classes vous permettent de stocker et de récupérer des types de données primitifs (entiers, booléens, chaînes de caractères, etc.) sous forme de valeur binaire.                                                                                                         |
| `BufferedStream`                   | Cette classe fournit un espace de stockage temporaire pour un flux d'octets que vous pourrez enregistrer ultérieurement.                                                                                                                                                     |
| `Directory`<br>`DirectoryInfo`     | Ces classes permettent de manipuler la structure de répertoires d'une machine. Le type `Directory` expose des fonctionnalités via des membres statiques, tandis que le type `DirectoryInfo` expose des fonctionnalités similaires à partir d'une *référence d'objet* valide. |
| `DriveInfo`                        | Ce cours fournit des informations détaillées concernant les disques utilisés par une machine donnée.                                                                                                                                                                         |
| `File`<br>`FileInfo`               | Ces classes permettent de manipuler l’ensemble des fichiers d’une machine. Le type `File` expose des fonctionnalités via des *membres statiques*, tandis que le type `FileInfo` expose des fonctionnalités similaires à partir d’une *référence d’objet* valide.             |
| `FileStream`                       | Cette classe vous offre un accès aléatoire aux fichiers (par exemple, des capacités de recherche) avec des données représentées sous forme de flux d'octets.                                                                                                                 |
| `FileSystemWatcher`                | Cette classe vous permet de surveiller la modification des fichiers externes dans un répertoire spécifié.                                                                                                                                                                    |
| `MemoryStream`                     | Cette classe permet un accès aléatoire aux données en flux continu stockées en mémoire plutôt que dans un fichier physique.                                                                                                                                                  |
| `Path`                             | Cette classe effectue des opérations sur les types `System.String` contenant des informations de chemin de fichier ou de répertoire de manière indépendante de la plateforme.                                                                                                |
| `StreamWriter`<br>`StreamReader`   | Ces classes permettent de stocker (et de récupérer) des informations textuelles dans (ou depuis) ​​un fichier. Elles ne prennent pas en charge l'accès aléatoire aux fichiers.                                                                                               |
| `StringWriter`<br>`StringReader`   | À l'instar des classes `StreamReader`/`StreamWriter`, ces classes fonctionnent également avec des informations textuelles. Cependant, le stockage sous-jacent est un tampon de chaînes plutôt qu'un fichier physique.                                                        |

**Outre ces types de classes concrètes, `System.IO` définit plusieurs énumérations, ainsi qu'un ensemble de classes abstraites** (par exemple, `Stream`, `TextReader` et `TextWriter`), **qui définissent une interface polymorphe partagée par toutes les classes descendantes**. Vous découvrirez plusieurs de ces types dans ce chapitre.

# Les types `Directory(Info)` et `File(Info)`

**`System.IO` fournit quatre classes permettant de manipuler des fichiers individuels et d'interagir avec la structure de répertoires d'une machine**. Les deux premiers types, `Directory` et `File`, exposent des opérations de création, de suppression, de copie et de déplacement à l'aide de divers membres statiques. Les types `FileInfo` et `DirectoryInfo`, étroitement liés, exposent des fonctionnalités similaires sous forme de méthodes d'instance (il est donc nécessaire de les allouer avec le mot-clé `new`). Les classes `Directory` et `File` héritent directement de `System.Object`, tandis que `DirectoryInfo` et `FileInfo` dérivent du type abstrait `FileSystemInfo`.

**Les classes `FileInfo` et `DirectoryInfo` sont généralement plus appropriées pour obtenir des informations complètes sur un fichier ou un répertoire** (par exemple, la date de création ou les autorisations de lecture/écriture), **car leurs membres renvoient généralement des objets fortement typés**. En revanche, ***==les membres des classes `Directory` et `File` renvoient généralement de simples chaînes de caractères plutôt que des objets fortement typés==***. Il ne s'agit toutefois que d'une indication ; **==dans de nombreux cas, vous pouvez obtenir le même résultat en utilisant `File`/`FileInfo` ou `Directory`/`DirectoryInfo`.==**

## La classe de base abstraite `FileSystemInfo`

**Les types `DirectoryInfo` et `FileInfo` héritent de nombreux comportements de la classe de base abstraite `FileSystemInfo`**. **==Le plus souvent, vous utilisez les membres de la classe `FileSystemInfo` pour découvrir les caractéristiques générales==** (telles que la date de création, divers attributs, etc.) **==d'un fichier ou d'un répertoire donné==**. Le [[#Tableau 19-2 Propriété de `FileSystemInfo`|Tableau 19-2]] répertorie certaines propriétés essentielles.

##### Tableau 19-2: Propriétés de `FileSystemInfo`

| Propriété        | Description                                                                                                                                                                                                            |
| ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Attributes`     | Obtient ou définit les attributs associés au fichier courant qui sont représentés par l'énumération `FileAttributes` (par exemple, le fichier ou le répertoire est-il en lecture seule, chiffré, caché ou compressé ?) |
| `CreationTime`   | Obtient ou définit la date de création du fichier ou du répertoire courant.                                                                                                                                            |
| `Exists`         | Détermine si un fichier ou un répertoire donné existe                                                                                                                                                                  |
| `Extension`      | Récupère l’extension d’un fichier                                                                                                                                                                                      |
| `FullName`       | Obtient le chemin complet du répertoire ou du fichier                                                                                                                                                                  |
| `LastAccessTime` | Obtient ou définit la date et l'heure du dernier accès au fichier ou répertoire courant.                                                                                                                               |
| `LastWriteTime`  | Obtient ou définit la date et l'heure de la dernière écriture dans le fichier ou le répertoire courant.                                                                                                                |
| `Name`           | Obtient le nom du fichier ou du répertoire courant                                                                                                                                                                     |

**`FileSystemInfo` définit également la méthode `Delete()`**. ***==Celle-ci est implémentée par les types dérivés pour supprimer un fichier ou un répertoire donné du disque dur. De plus, vous pouvez appeler `Refresh()` avant d'obtenir les informations d'attribut afin de vous assurer que les statistiques relatives au fichier (ou répertoire) courant ne sont pas obsolètes.==***

# Utilisation du type `DirectoryInfo`

Le premier type d'E/S créable que vous examinerez est la classe `DirectoryInfo`. **Cette classe contient un ensemble de membres utilisés pour créer, déplacer, supprimer et énumérer les répertoires et sous-répertoires**. En plus des fonctionnalités fournies par sa classe de base (`FileSystemInfo`), `DirectoryInfo` offre les principaux membres détaillés dans le [[#Tableau 19-3 Membres clés du type `DirectoryInfo`|Tableau 19-3]].

##### Tableau 19-3: Membres clés du type `DirectoryInfo`

| Membre                               | Description                                                                                            |
| ------------------------------------ | ------------------------------------------------------------------------------------------------------ |
| `Create()`<br>`CreateSubdirectory()` | Crée un dossier (ou un ensemble de sous-dossier) à partir d'un nom de chemin.                          |
| `Delete()`                           | Supprime un dossier et tout son contenu                                                                |
| `GetDirectories()`                   | Renvoie un tableau d'objets `DirectoryInfo` représentant tous les sous-dossiers du dossier courant.    |
| `GetFiles()`                         | Récupère un tableau d'objets `FileInfo` représentant un ensemble de fichiers dans le dossier spécifié. |
| `MoveTo()`                           | Déplace un dossier et son contenu vers un nouveau chemin d'accès                                       |
| `Parent`                             | Récupère le dossier parent de ce dossier                                                               |
| `Root`                               | Obtient la partie racine d'un chemin d'accès                                                           |

**Pour utiliser le type `DirectoryInfo`, spécifiez un chemin de répertoire particulier comme paramètre du constructeur**. Utilisez la notation pointée (`.`) pour accéder au répertoire de travail courant (le répertoire de l'application en cours d'exécution). Voici quelques exemples :

```cs
// Se lier au répertoire de travail courant.
DirectoryInfo dir1 = new DirectoryInfo(".");

// Se connecter à C:\Windows,
// en utilisant une chaîne de caractères exacte.
DirectoryInfo dir2 = new DirectoryInfo(@"C:\Windows");
```

>[!info] L'équivalent à `C:\Windows` sur macOS est `/System`

>[!tip] La nécessité ou non des chaînes verbatim (`@`)
>
>Comme sur windows le séparateur utilisé pour les chemin d'accès est le même que pour échapper des caractères (`\`). 
>
>**Sur les système Unix (Linux/macOS), il n'est pas nécessaire car ces systèmes utilise `/` !**

Dans le second exemple, vous supposez que le chemin passé au constructeur (`C:\Windows`) existe déjà sur la machine physique. Cependant, *==si vous tentez d'accéder à un répertoire inexistant, une exception `System.IO.DirectoryNotFoundException` est levée==*. Par conséquent, ***==si vous spécifiez un répertoire qui n'a pas encore été créé, vous devez appeler la méthode `Create()` avant de poursuivre==***, comme ceci :

```cs
// Se connecter à un répertoire inexistant, puis le créer.
DirectoryInfo dir3 = new DirectoryInfo("~/MyCode/Testing");
dir3.Create();
```

**La syntaxe de chemin utilisée dans l'exemple précédent est spécifique à Windows**. ~~Si vous développez des applications .NET pour différentes plateformes, vous devez utiliser les constructions `Path.VolumeSeparatorChar` et `Path.DirectorySeparatorChar`, qui généreront les caractères appropriés en fonction de la plateforme~~. Mettez à jour le code précédent comme suit :

>[!warning]- Très mauvaise pratique (Gemini)
>
>1. **Le piège de la lettre de lecteur (`C`) :** Même si on utilisez `Path.VolumeSeparatorChar` (qui donne `:` sur Windows), la lettre `C` reste écrite en dur. Sur macOS, le chemin va se transformer en `C:/MyCode/Testing`. Le système va chercher un dossier nommé littéralement `C` à la racine de votre Mac, ce qui n'existe pas et va lever une erreur.
>2. **Une syntaxe illisible :** Utiliser ces constantes imbriquées dans une chaîne interpolée (`$@""`) rend le code lourd et difficile à maintenir.
>
>## Les deux solutions possible
>
>**La solution moderne : `Path.Combine()` (Déjà vu précédement)**
>
Depuis plusieurs versions de .NET, la règle absolue pour créer un chemin multiplateforme est d'utiliser la méthode statique **`Path.Combine()`**.
>
>Elle gère automatiquement les séparateurs (`\` ou `/`) selon que l'application tourne sur Windows, macOS ou Linux, et s'adapte parfaitement si vous commencez par une racine Unix (`/`).
>
>**La solution professionnelle : `Environment.GetFolderPath()`**
>
>En conditions réelles, on n'écrit jamais de racine en dur. On demande au framework .NET de trouver les dossiers spéciaux de l'utilisateur (comme le dossier _Documents_ ou _Home_), ce qui garantit une compatibilité totale sans se soucier de l'OS:
>```cs
>// Récupère "/Users/votre_nom/Documents" sur Mac ou "C:\Users\nom\Documents" sur Windows
string documentsPath = Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments);
>
>// Assemble le chemin de manière 100% sécurisée
>DirectoryInfo dirInfo = new DirectoryInfo(Path.Combine(documentsPath, "MyCode", "Testing"));
>```

```cs
DirectoryInfo dir3 = new DirectoryInfo(
$@"C{Path.VolumeSeparatorChar}{Path.DirectorySeparatorChar}MyCode{Path.DirectorySeparator
Char}Testing");
```

Après avoir créé un objet `DirectoryInfo`, vous pouvez explorer le contenu du répertoire sous-jacent à l'aide des propriétés héritées de `FileSystemInfo`. Pour voir cela en pratique, créez un nouveau projet d'application console nommé *DirectoryApp* et mettez à jour votre fichier C# pour importer `System` et `System.IO`. Mettez à jour votre fichier *Program.cs* avec la nouvelle méthode statique suivante qui crée un nouvel objet `DirectoryInfo` mappé à `C:\Windows` (**==adaptez votre chemin si nécessaire==**), affichant plusieurs statistiques intéressantes :

>[!tip] Rappel
>Avec les version modernes de .NET, un fichier *GlobalUsings.cs* est généré dans le dossier `obj`, avec dedans `System` et `System.IO`

```cs
Console.Title = "Fun with Directory(Info)";
Console.WriteLine("***** Fun with Directory(Info) *****\n");

ShowSystemDirectoryInfo();

static void ShowSystemDirectoryInfo()
{
    // Affiche les informations du répertoire. 
    // Si vous n'utilisez pas Windows, 
    // veuillez indiquer un autre répertoire.
    DirectoryInfo dir = new($@"{Path.VolumeSeparatorChar}System");
    Console.WriteLine("**** Directory Info ****");
    Console.WriteLine($"FullName: {dir.FullName}");
    Console.WriteLine($"Name: {dir.Name}");
    Console.WriteLine($"Parent: {dir.Parent}");
    Console.WriteLine($"Creation: {dir.CreationTime}");
    Console.WriteLine($"Attributes: {dir.Attributes}");
    Console.WriteLine($"root: {dir.Root}");
    Console.WriteLine("*************************\n");
  
}

Console.ReadLine();
```

## Énumération des fichiers avec le type `DirectoryInfo`

Outre l'obtention des informations de base d'un répertoire existant, **vous pouvez étendre l'exemple actuel pour utiliser certaines méthodes du type `DirectoryInfo`**. Vous pouvez notamment utiliser la méthode `GetFiles()` pour obtenir des informations sur tous les fichiers *.jpg* situés dans le répertoire `C:\Windows\Web\Wallpaper` (**==remplacez ce répertoire si nécessaire par un répertoire contenant des images sur votre ordinateur==**).

La méthode `GetFiles()` renvoie un tableau d'objets `FileInfo`, chacun exposant les détails d'un fichier particulier (vous découvrirez tous les détails du type `FileInfo` plus loin dans ce chapitre). Créez la méthode statique suivante dans le fichier *Program.cs* :

>[!warning] Ici, on exécute le programme depuis le dossier parent de `DirectoryApp` avec la commande `dotnet run --project ...`. C'est pour cela que le chemin d'accès est `./DirectoryApp/Assets`

```cs
static void DisplayImageFiles()
{
    DirectoryInfo dir = new(Path.Combine(".", "DirectoryApp", "Assets"));
    // Récupère tous les fichiers avec l'extension .png
    var imgFiles = dir.GetFiles("*.png", SearchOption.AllDirectories);

    // Combien ont été trouvé ?
    Console.WriteLine($"Found {imgFiles.Length} *.png files\n");

    // Maintenant on affiche les infos pour chaque fichier
    foreach (var f in imgFiles)
    {
        Console.WriteLine("*********************");
        Console.WriteLine($"File name: {f.Name}");
        Console.WriteLine($"File size: {f.Length}");
        Console.WriteLine($"Creation: {f.CreationTime}");
        Console.WriteLine($"Attributes: {f.Attributes}");
        Console.WriteLine("*********************\n");
    }
}
```

**Notez que vous spécifiez une option de recherche lorsque vous appelez `GetFiles()` ; cela permet de rechercher dans tous les sous-répertoires du répertoire racine**. Après avoir exécuté l'application, vous verrez la liste de tous les fichiers correspondant au modèle de recherche.

## Création de sous-répertoires avec le type `DirectoryInfo`

**Vous pouvez étendre par programmation une structure de répertoires à l'aide de la méthode `DirectoryInfo.CreateSubdirectory()`**. Cette méthode permet de **==créer un sous-répertoire unique, ainsi que plusieurs sous-répertoires imbriqués, en un seul appel de fonction==**. Cet exemple illustre comment procéder, en étendant la structure de répertoires du répertoire d'exécution de l'application (indiqué par un point) avec des sous-répertoires personnalisés :

```cs
static void ModifyAppDirectory()
{
	// Différent par rapport au livre car je lance les programmes 
	// depuis le dossier parent.
    var dir = new DirectoryInfo("DirectoryApp");

    // Crée `/MyFolder` depuis le dossier de l'application
    dir.CreateSubdirectory("MyFolder");

    // Crée `/MyFolder2/Data` depuis le dossier de l'application
    dir.CreateSubdirectory(Path.Combine("MyFolder2", "Data"));
}
```

**Il n'est pas nécessaire de capturer la valeur de retour de la méthode `CreateSubdirectory()`, mais vous devez savoir qu'un objet `DirectoryInfo` représentant l'élément nouvellement créé est renvoyé en cas de succès**. Voici une mise à jour possible de la méthode précédente :

```cs
static void ModifyAppDirectory()
{
    var dir = new DirectoryInfo("DirectoryApp");

    // Crée `/MyFolder` depuis le dossier de l'application
    dir.CreateSubdirectory("MyFolder");

    // Crée `/MyFolder2/Data` depuis le dossier de l'application
    // On capture l'objet DirectoryInfo retournée
    var myDataFolder = dir.CreateSubdirectory(
        Path.Combine("MyFolder2", "Data")
    );
    Console.WriteLine($"New folder is: {myDataFolder}");
}
```

Si vous appelez cette méthode à partir des instructions de niveau supérieur et que vous examinez votre répertoire Windows à l'aide de l'Explorateur Windows, vous constaterez que les nouveaux sous-répertoires sont bien présents.

# Utilisation du type `Directory`

Vous avez vu le type `DirectoryInfo` en action ; vous êtes maintenant prêt à découvrir le type `Directory`. **Dans la plupart des cas, les membres statiques de `Directory` reproduisent les fonctionnalités offertes par les membres d'instance définis par `DirectoryInfo`**. Rappelez-vous cependant que ==les membres de `Directory` renvoient généralement des *chaînes de caractères*, plutôt que des objets `FileInfo`/`DirectoryInfo` fortement typés.==

Examinons maintenant certaines fonctionnalités du type `Directory`. Cette dernière fonction auxiliaire affiche les noms de tous les lecteurs mappés sur l'ordinateur actuel (à l'aide de la méthode `Directory.GetLogicalDrives()`) et utilise la méthode statique `Directory.Delete()` pour supprimer les sous-répertoires `\MyFolder` et `\MyFolder2\Data` créés précédemment.

```cs
static void FunWithDirectoryType()
{
    // Liste toute les disques présent dans l'ordinateur
    // Sur macOS, je dois filtres les backup de timemachine
    var drives = Directory
        .GetLogicalDrives()
        .Where(d => !d.Contains("timemachine"));
    Console.WriteLine("Here are your drives:");
    foreach (var s in drives)
        Console.WriteLine($"---> {s}");

    // Supprime ce qu'il vient d'être crée
    Console.WriteLine("\nPress enter to delete directories");
    Console.ReadLine();
    try
    {
        Directory.Delete(Path.Combine("DirectoryApp", "MyFolder"));

        // Le deuxième paramètre spécifie si vous voulez
        // détruire touts les sous-dossiers
        Directory.Delete(Path.Combine("DirectoryApp", "MyFolder2"), true);
    }
    catch (IOException ex)
    {
        Console.WriteLine(ex.Message);
    }
}
```



# Utilisation de la classe `DriveInfo`

==L'espace de noms `System.IO` fournit une classe nommée `DriveInfo`==. **À l'instar de `Directory.GetLogicalDrives()`, la méthode statique `DriveInfo.GetDrives()` permet de connaître les noms des lecteurs d'une machine**. Contrairement à `Directory.GetLogicalDrives()`, **`DriveInfo` fournit de nombreuses autres informations (par exemple, le type de lecteur, l'espace libre disponible et le nom du volume)**. Prenons l'exemple du fichier *Program.cs* suivant, défini dans un nouveau projet d'application console nommé *DriveInfoApp* :

```cs
Console.Title = "Fun with DriveInfo";
Console.WriteLine("***** Fun with DriveInfo *****\n");

// Récupère les infos sur tous les disques
// Il faut filtrer les backups timemachine sur macOS
var myDrives = DriveInfo
    .GetDrives()
    .Where(d => !d.Name.Contains(".timemachine"));

// Maintenant on affiche les infos
foreach (var d in myDrives)
{
    Console.WriteLine($"Name: {d.Name}");
    Console.WriteLine($"Type: {d.DriveType}");
    // Vérifie si le disque est montée (mount)
    if (d.IsReady)
    {
        Console.WriteLine($"Free space: {d.TotalFreeSpace}");
        Console.WriteLine($"Format: {d.DriveFormat}");
        Console.WriteLine($"Label: {d.VolumeLabel}");
    }
    Console.WriteLine();
}
Console.ReadLine();
```

Voici une sortie possible: 

```
Name: / 
Type: Fixed 
Free space: 76246011904 
Format: apfs 
Label: / 

Name: /dev 
Type: Ram 
Free space: 0 
Format: devfs 
Label: /dev

...
```

À ce stade, vous avez exploré certains comportements fondamentaux des classes `Directory`, `DirectoryInfo` et `DriveInfo`. Vous allez maintenant apprendre à créer, ouvrir, fermer et supprimer les fichiers qui composent un répertoire donné.

# Utilisation de la classe `FileInfo`

Comme illustré dans l'exemple précédent de *DirectoryApp*, **la classe `FileInfo` vous permet d'obtenir des informations sur les fichiers existants sur votre disque dur** (par exemple, la date de création, la taille et les attributs du fichier) **et facilite la création, la copie, le déplacement et la suppression de fichiers**. Outre les fonctionnalités héritées de `FileSystemInfo`, vous trouverez des membres spécifiques à la classe `FileInfo`, décrits dans le [[#Tableau 19-4 Membres importants de `FileInfo`|Tableau 19-4]].

##### Tableau 19-4: Membres importants de `FileInfo`

| Membre          | Description                                                                                                                   |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| `AppendText()`  | Crée un objet `StreamWriter` (décrit plus loin) qui ajoute du texte à un fichier                                              |
| `CopyTo()`      | Copie un fichier existant dans un nouveau fichier                                                                             |
| `Create()`      | Crée un nouveau fichier et renvoie un objet `FileStream` (décrit plus loin) pour interagir avec le fichier nouvellement créé. |
| `CreateText()`  | Crée un objet `StreamWriter` qui écrit un nouveau fichier texte                                                               |
| `Delete()`      | Supprime le fichier auquel une instance de `FileInfo` est liée.                                                               |
| `Directory`     | Obtient une instance du dossier parent                                                                                        |
| `DirectoryName` | Obtient le chemin complet vers le dossier parent                                                                              |
| `Length`        | Obtient la taille du fichier actuel                                                                                           |
| `MoveTo()`      | Déplace un fichier spécifié vers un nouvel emplacement, avec la possibilité de spécifier un nouveau nom de fichier.           |
| `Name`          | Obtient le nom du fichier                                                                                                     |
| `Open()`        | Ouvre un fichier avec différents privilèges de lecture/écriture et de partage.                                                |
| `OpenRead()`    | Crée un objet `FileStream` en lecture seule                                                                                   |
| `OpenText()`    | Crée un objet `StreamReader` (décrit plus loin) qui lit à partir d'un fichier texte existant.                                 |
| `OpenWrite()`   | Crée un objet `FileStream` en écriture seule                                                                                  |

***==Notez que la plupart des méthodes de la classe `FileInfo` renvoient un objet spécifique d'entrée/sortie==*** (**par exemple, `FileStream` et `StreamWriter`**) ***==qui vous permet de commencer à lire et à écrire des données dans (ou à lire depuis) ​​le fichier associé dans différents formats==***. Vous découvrirez ces types dans un instant ; cependant, avant de voir un exemple concret, **==il vous sera utile d'examiner différentes manières d'obtenir un descripteur de fichier à l'aide du type de la classe `FileInfo`.==**

## La méthode `FileInfo.Create()`

Les exemples suivants se trouvent dans une application console nommée *SimpleFileIO*. Une façon de créer un **==descripteur de fichier==** consiste à utiliser la méthode `FileInfo.Create()`, comme ceci :

```cs
Console.Title = "Simple IO with the File type";
Console.WriteLine("***** Simple IO with the File type *****\n");

// Changez vers un dossier dans la machine où vous avez les
// droits de lecture/écriture (ou lancer en administrateur)

// Ici je récupère le chemin vers Home car .NET ne comprend pas
// le symbole ~
var homePath = Environment.GetFolderPath(Environment.SpecialFolder.UserProfile);
var fileName = Path.Combine(homePath, "temp", "test.dat");

// Crée un nouveau fichier
var f = new FileInfo(fileName);
FileStream fs = f.Create();

// Utilisation de l'objet FileStream...

// On ferme le flux de fichier.
fs.Close();
```

>[!note] 
>Ces exemples peuvent nécessiter l'exécution de Visual Studio en tant qu'administrateur, selon vos autorisations utilisateur et la configuration de votre système.

**Notez que la méthode `FileInfo.Create()` renvoie un objet `FileStream`, qui expose des opérations d'écriture/lecture synchrones et asynchrones sur le fichier sous-jacent** (nous y reviendrons plus en détail). ***==Sachez que l'objet `FileStream` renvoyé par `FileInfo.Create()` accorde un accès complet en lecture/écriture à tous les utilisateurs.==***

Notez également qu'après avoir terminé avec l'objet `FileStream` actuel, *==vous devez fermer le handle pour libérer les ressources du flux non managé sous-jacent==*. **==Étant donné que `FileStream` implémente l'interface `IDisposable`, vous pouvez utiliser la portée `using` en C# pour permettre au compilateur de générer la logique de libération==** (voir le [[Chapitre 9#Réutilisation du mot-clé `using` en C|Chapitre 9]] pour plus de détails), comme ceci :

```cs
...

// Englobe le flux de fichier dans une déclaration using
// Définit une portée using pour les E/S du fichier
var f1 = new FileInfo(filename);
using (FileStream fs1 = f1.Create())
{
// Utilisation de l'objet FileStream...
}
f1.Delete();
```

>[!note]
>Presque tous les exemples de ce chapitre utilisent des instructions `using`. J'aurais pu utiliser la nouvelle syntaxe de déclaration `using`, mais j'ai choisi de ne pas le faire dans cette réécriture afin de concentrer les exemples sur les composants `System.IO` que nous étudions.

## La méthode `FileInfo.Open()`

**Vous pouvez utiliser la méthode `FileInfo.Open()` pour ouvrir des fichiers existants, ainsi que pour créer de nouveaux fichiers avec une bien plus grande précision qu'avec `FileInfo.Create()`. Cela fonctionne car `Open()` prend généralement plusieurs paramètres pour préciser exactement comment parcourir le fichier à manipuler. Une fois l'appel à `Open()` terminé, un objet `FileStream` est renvoyé. Prenons l'exemple suivant :

```cs
...

// Crée un nouveau fichier via FileInfo.Open().
var f2 = new FileInfo(fileName);
using (
    FileStream fs2 = f2.Open(
        FileMode.OpenOrCreate,
        FileAccess.ReadWrite,
        FileShare.None
    )
)
{
    // Utilisation de l'objet FileStream...
}
f2.Delete();
```

***==Cette version surchargée de la méthode `Open()` requiert trois paramètres. Le premier paramètre de la méthode `Open()` spécifie le type d'opération d'entrée/sortie (par exemple, créer un nouveau fichier, ouvrir un fichier existant et ajouter du contenu à un fichier), que vous spécifiez à l'aide de l'énumération `FileMode`==*** (voir le [[#Tableau 19-5 Membres de l'énumération `FileMode`|Tableau 19-5]] pour plus de détails).

```cs
public enum FileMode
{
    CreateNew,
    Create,
    Open,
    OpenOrCreate,
    Truncate,
    Append,
}
```

##### Tableau 19-5: Membres de l'énumération `FileMode`

| Membre         | Description                                                                                                                                                                                                         |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `CreateNew`    | Demande au système d'exploitation de créer un nouveau fichier. Si celui-ci existe déjà, une exception `IOException` est levée.                                                                                      |
| `Create`       | Demande au système d'exploitation de créer un nouveau fichier. S'il existe déjà, il sera écrasé.                                                                                                                    |
| `Open`         | Ouvre un fichier existant. Si le fichier n'existe pas, une exception `FileNotFoundException` est levée.                                                                                                             |
| `OpenOrCreate` | Ouvre le fichier s'il existe ; sinon, un nouveau fichier est créé.                                                                                                                                                  |
| `Truncate`     | Ouvre un fichier existant et le tronque à une taille de 0 octet.                                                                                                                                                    |
| `Append`       | Ouvre un fichier, se place à la fin de celui-ci et lance les opérations d'écriture (cette option ne peut être utilisée qu'avec un flux en écriture seule). Si le fichier n'existe pas, un nouveau fichier est créé. |

***==Vous utilisez le deuxième paramètre de la méthode `Open()`, une valeur de l'énumération `FileAccess`, pour déterminer le comportement de lecture/écriture du flux sous-jacent==***, comme suit :

```cs
public enum FileAccess
{
    Read,
    Write,
    ReadWrite,
}
```

***==Enfin, le troisième paramètre de la méthode `Open()`, `FileShare`, spécifie comment partager le fichier avec d'autres gestionnaires de fichiers==***. Voici les noms principaux :

```cs
public enum FileShare
{
    None,
    Read,
    Write,
    ReadWrite,
    Delete,
    Inheritable,
}
```

## Les méthodes `FileInfo.OpenRead()` et `FileInfo.OpenWrite()`

**La méthode `FileInfo.Open()` permet d'obtenir un descripteur de fichier de manière flexible, mais la classe `FileInfo` propose également des membres nommés `OpenRead()` et `OpenWrite()`**. Comme vous pouvez l'imaginer, **==ces méthodes renvoient un objet `FileStream` correctement configuré en lecture seule ou en écriture seule, sans qu'il soit nécessaire de fournir diverses valeurs d'énumération==**. ==À l'instar de `FileInfo.Create()` et `FileInfo.Open()`, `OpenRead()` et `OpenWrite()` renvoient un objet `FileStream`.

**Notez que la méthode `OpenRead()` nécessite que le fichier existe déjà**. ***==Le code suivant crée le fichier, puis ferme le `FileStream` afin qu'il puisse être utilisé par la méthode `OpenRead()` :==**

```cs
f3.Create().Close();
```

Voici l'exemple complet:

```cs
...

// Récupère un objet FileStream avec les persmissions de lecture seule
var f3 = new FileInfo(fileName);
// Le fichier dois exister avant d'utiliser OpenRead
f3.Create().Close();
using (FileStream readOnlyStream = f3.OpenRead())
{
    // Utilisation de l'objet FileStream...
}
f3.Delete();

// Maintenant récupère un objet FileStream avec les persmissions
// d'écriture seule.
FileInfo f4 = new(fileName);
using (FileStream writeOnlyStram = f4.OpenWrite())
{
    // Utilisation de l'objet FileStream...
}
f4.Delete();
```

## La méthode `FileInfo.OpenText()`

**`OpenText()` est une autre méthode d'ouverture du type `FileInfo`**. ==Contrairement à `Create()`, `Open()`, `OpenRead()` ou `OpenWrite()`, la méthode `OpenText()` renvoie une instance du type `StreamReader`, et non une instance du type `FileStream`.==

```cs
...

// Récupère un objet StreamReader
FileInfo f5 = new(fileName);
// Le Fichier doit exister avec d'usilier OpenText
f5.Create().Close();
using (StreamReader sReader = f5.OpenText())
{
    // Utilisation de l'objet StreamReader...
}
f5.Delete();
```

Comme vous le verrez prochainement, **le type `StreamReader` offre un moyen de lire des données de caractères à partir du fichier sous-jacent.**

## Les méthodes `FileInfo.CreateText()` et `FileInfo.AppendText()`

Les deux dernières méthodes `FileInfo` qui nous intéressent ici sont `CreateText()` et `AppendText()`. Toutes deux renvoient un objet `StreamWriter`, comme illustré ici :

```cs
var f6 = new FileInfo(fileName);
using (StreamWriter sWriter = f6.CreateText())
{
    // Utilisation de l'objet StreamWriter...
}
f6.Delete();

var f7 = new FileInfo(fileName);
using (StreamWriter sWriterAppend = f7.AppendText())
{
    // Utilisation de l'objet StreamWriter...
}
f7.Delete();
```

Comme vous pouvez le deviner, **le type `StreamWriter` permet d'écrire des données de caractères dans le fichier sous-jacent.**

# Utilisation du type `File`

**Le type `File` possède plusieurs membres statiques pour offrir des fonctionnalités presque identiques à celles du type `FileInfo`**. Comme `FileInfo`, **==`File` fournit les méthodes `AppendText()`, `Create()`, `CreateText()`, `Open()`, `OpenRead()`, `OpenWrite()` et `OpenText()`==**. **Dans de nombreux cas, vous pouvez utiliser les types `File` et `FileInfo` de manière interchangeable**. Notez que `OpenText()` et `OpenRead()` nécessitent que le fichier existe déjà. Pour voir cela en pratique, vous pouvez simplifier chacun des exemples `FileStream` précédents en utilisant le type `File` à la place, comme ceci : 

```cs
...

// Utilisation de File au lieu de FileInfo
using (FileStream fs8 = File.Create(fileName))
{
    // Utilisation de l'objet FileStream...
}
File.Delete(fileName);

// Crée un nouveau fichier via File.Open
using (
    FileStream fs9 = File.Open(
        fileName,
        FileMode.OpenOrCreate,
        FileAccess.ReadWrite,
        FileShare.None
    )
)
{
    // Utilisation de l'objet FileStream...
}

// Récupère un objet FileStream avec les permissions de lecture seule.
using (FileStream readOnlyStream = File.OpenRead(fileName)) { }
File.Delete(fileName);

// Récupère un objet FileStream avec les permissions d'écriture seule
using (FileStream writeOnlyStream = File.OpenWrite(fileName)) { }

// Récupère un objet StreamReader
using (StreamReader sreader = File.OpenText(fileName)) { }
File.Delete(fileName);

// Récupère des objets StreamWriter
using (StreamWriter swriter = File.CreateText(fileName)) { }
File.Delete(fileName);
using (StreamWriter swriterAppend = File.AppendText(fileName)) { }
File.Delete(fileName);
```

## Membres additionnels centré sur `File`

Le type `File` prend également en charge quelques membres, présentés dans le [[#Tableau 19-6 Méthodes du type `File`|Tableau 19-6]], qui peuvent grandement simplifier les processus de lecture et d'écriture de données textuelles.

##### Tableau 19-6: Méthodes du type `File`

| Méthode           | Description                                                                                                                         |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| `ReadAllBytes()`  | Ouvre le fichier spécifié, renvoie les données binaires sous forme de tableau d'octets, puis ferme le fichier.                      |
| `ReadAllLines()`  | Ouvre un fichier spécifié, renvoie les données de caractères sous forme de tableau de chaînes de caractères, puis ferme le fichier. |
| `ReadAllText()`   | Ouvre un fichier spécifié, renvoie les données de caractères sous forme de `System.String`, puis ferme le fichier.                  |
| `WriteAllBytes()` | Ouvre le fichier spécifié, y écrit le tableau d'octets, puis ferme le fichier.                                                      |
| `WriteAllLines()` | Ouvre un fichier spécifié, y écrit un tableau de chaînes de caractères, puis ferme le fichier.                                      |
| `WriteAllText()`  | Ouvre un fichier spécifié, y écrit les données de caractères d'une chaîne de caractères spécifiée, puis ferme le fichier.           |

**Vous pouvez utiliser ces méthodes du type `File` pour lire et écrire des lots de données en quelques lignes de code seulement**. Mieux encore, **==chacune de ces méthodes ferme automatiquement le descripteur de fichier sous-jacent==**. Par exemple, le code suivant enregistre les données de type chaîne dans un nouveau fichier (et le charge en mémoire) avec une grande simplicité :

```cs
Console.WriteLine("***** Simple I/O with the File Type");
string[] myTasks =
[
    "Fix bathroom sink",
    "Call Dave",
    "Call Mom and Dad",
    "Play Games",
];

// Écris toues les donnée dans le répoertoire
// depuis lequel on lance le projet
File.WriteAllLines("./tasks.txt", myTasks);

// Lis les données en retour et les affiches.
foreach (var task in File.ReadAllLines("./tasks.txt"))
    Console.WriteLine($"TODO: {task}");
Console.ReadLine();
File.Delete("./tasks.txt");
```

La leçon à retenir est que **pour obtenir rapidement un descripteur de fichier, le type `File` permet de gagner du temps. Cependant, créer au préalable un objet `FileInfo` présente l'avantage de pouvoir examiner le fichier à l'aide des membres de la classe de base abstraite `FileSystemInfo`.**

# La classe abstraite `Stream`

À ce stade, vous avez vu plusieurs façons d'obtenir des objets `FileStream`, `StreamReader` et `StreamWriter`, mais vous n'avez pas encore lu ni écrit de données dans un fichier à l'aide de ces types. Pour comprendre comment faire, ==vous devrez vous familiariser avec le concept de flux==. **Dans le domaine des entrées/sorties, un *flux* représente un ensemble de données circulant entre une source et une destination**. ==Les flux offrent un moyen commun d' interagir avec une *séquence d'octets*, quel que soit le périphérique== (fichier, connexion réseau ou imprimante, par exemple) ==qui stocke ou affiche ces octets.==

**La classe abstraite `System.IO.Stream` définit plusieurs membres qui prennent en charge les interactions synchrones et asynchrones avec le support de stockage** (==par exemple, un fichier sous-jacent ou un emplacement mémoire==).

>[!Note]
>Le concept de flux ne se limite pas aux entrées/sorties de fichiers. En effet, les bibliothèques .NET offrent un accès aux flux aux réseaux, aux emplacements mémoire et à d'autres abstractions centrées sur les flux.

**Là encore, les classes dérivées de `Stream` représentent les données sous forme de flux brut d'octets** ; par conséquent, *==manipuler directement des flux bruts peut s'avérer complexe==*. ==Certains types dérivés de `Stream` prennent en charge la *recherche (seek)*, qui désigne le processus permettant d'obtenir et d'ajuster la position actuelle dans le flux==. Le [[#Tableau 19-7 Membres abstraits de `Stream`|Tableau 19-7]] vous aide à comprendre les fonctionnalités offertes par la classe Stream en décrivant ses membres principaux.

##### Tableau 19-7: Membres abstraits de `Stream`

| Membre                                       | Description                                                                                                                                                                                                                                                                               |
| -------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `CanRead`<br>`CanWrite`<br>`CanSeek`         | Détermine si le flux actuel prend en charge la lecture, la recherche et/ou l'écriture.                                                                                                                                                                                                    |
| `Close()`                                    | Ferme le flux courant et libère les ressources (telles que les sockets et les descripteurs de fichiers) qui lui sont associées. En interne, cette méthode est un alias de la méthode `Dispose()` ; par conséquent, *fermer un flux* est fonctionnellement équivalent à *libérer un flux*. |
| `Flush()`                                    | Met à jour la source de données ou le référentiel sous-jacent avec l'état actuel du tampon, puis efface ce dernier. Si un flux n'implémente pas de tampon, cette méthode est sans effet.                                                                                                  |
| `Length`                                     | Renvoie la longueur du flux en octets.                                                                                                                                                                                                                                                    |
| `Position`                                   | Determines the position in the current stream.                                                                                                                                                                                                                                            |
| `Read()`<br>`ReadByte()`<br>`ReadAsync()`    | Lit une séquence d'octets (ou un seul octet) à partir du flux actuel et avance la position actuelle dans le flux du nombre d'octets lus.                                                                                                                                                  |
| `Seek()`                                     | Définit la position dans le flux actuel.                                                                                                                                                                                                                                                  |
| `SetLength()`                                | Définit la longueur du flux actuel.                                                                                                                                                                                                                                                       |
| `Write()`<br>`WriteByte()`<br>`WriteAsync()` | Écrit une séquence d'octets (ou un seul octet) dans le flux actuel et fait avancer la position actuelle dans ce flux du nombre d'octets écrits.                                                                                                                                           |

# Utilisation de `FileStream`

**La classe `FileStream` fournit une implémentation des membres abstraits `Stream` d'une manière adaptée au traitement de flux de fichiers**. ***==Il s'agit d'un flux primitif ; il ne peut lire ou écrire qu'un seul octet ou un tableau d'octets==***. Cependant, *==vous n'aurez que rarement besoin d'interagir directement avec les membres du type `FileStream`==*. **Vous utiliserez probablement plutôt divers *wrappers de flux*, qui facilitent la manipulation de données textuelles ou de types .NET**. ==Néanmoins, il peut être utile d'expérimenter avec les capacités de lecture/écriture synchrones du type `FileStream`.==

Supposons que vous ayez un nouveau projet d'application console nommé *FileStreamApp* (et vérifiez que l'espace de noms `System.Text` est bien importé dans votre fichier de code C# initial). ==Votre objectif est d'écrire un simple message texte dans un nouveau fichier nommé *myMessage.dat*==. Cependant, ***==comme `FileStream` ne peut traiter que des octets bruts, vous devrez encoder le type `System.String` en un tableau d'octets correspondant==***. Heureusement, **==l'espace de noms `System.Text` définit un type nommé `Encoding` qui fournit des membres permettant d'encoder et de décoder des chaînes de caractères vers (ou depuis) ​​un tableau d'octets.==**

**Une fois encodé, le tableau d'octets est enregistré dans le fichier à l'aide de la méthode `FileStream.Write()`. Pour relire les octets en mémoire, vous devez réinitialiser la position interne du flux (à l'aide de la propriété `Position`) et appeler la méthode `ReadByte()`**. Enfin, vous affichez le tableau d'octets bruts et la chaîne décodée dans la console. Voici le code complet :

```cs
using System.Text;

Console.Title = "Fun with FileStreams";
Console.WriteLine("***** Fun with FileStreams *****\n");

// obtient un objet FileStream
using var fStream = File.Open("myMessage.dat", FileMode.Create);

// Encode un string en en tableau d'octet (byte)
string msg = "Hello!";
byte[] msgAsByteArray = Encoding.Default.GetBytes(msg);

// Écris le tableau d'octet dans le fichier
fStream.Write(msgAsByteArray, 0, msgAsByteArray.Length);

// Réinitialise la position interne du flux
fStream.Position = 0;

// Lis les types provenant du ficher et l'affiche dans le terminal
Console.Write("Your message as an array of bytes: ");
byte[] bytesFromFile = new byte[msgAsByteArray.Length];
for (int i = 0; i < bytesFromFile.Length; i++)
{
    // Cette méthode lis octet par octet
    // (Contrairement à FileStream.Read())
    bytesFromFile[i] = (byte)fStream.ReadByte();
    // Formatage de la sortie console en Hexadécimal
    // (PLus claire sur les valeurs de sortie)
    Console.Write($"{bytesFromFile[i]:X}");

    /*
     * Rappel :
     *
     * .NET et les languages modernes utilise
     * l'encodage UTF-8, signifiant qu'un caractère
     * est défini par 2 bytes (octets)
     *
     * La propriété Default de la classe Encoding 
     * est donc défini sur UTF-8
     *
     */
}

// Affiche le message décodé
Console.Write("\nDecoded Message: ");
Console.WriteLine(Encoding.Default.GetString(bytesFromFile));
Console.ReadLine();

File.Delete("myMessage.dat");
```

> Comme on n'ouvre qu'un seule fichier dans tout le programme, j'ai décidé d'utilisé la syntaxe `using` avec une portée globale.

**Cet exemple remplit le fichier avec des données, mais il met également en évidence le principal inconvénient de l'utilisation directe du type `FileStream`** : *==il exige de manipuler des octets bruts==*. ***==D'autres types dérivés de `Stream` fonctionnent de manière similaire. Par exemple, si vous souhaitez écrire une séquence d'octets dans une zone mémoire, vous pouvez allouer un `MemoryStream`==***. Comme mentionné précédemment, **l'espace de noms `System.IO` fournit plusieurs types de lecture et d'écriture qui encapsulent les détails d'utilisation des types dérivés de `Stream`.**

# Utilisation des `StreamWriters` et `StreamReaders`

**Les classes `StreamWriter` et `StreamReader` sont utiles pour lire et écrire des données de type caractère (par exemple, des chaînes de caractères)**. ==Par défaut, elles fonctionnent avec des caractères Unicode ; vous pouvez toutefois modifier ce comportement en fournissant une référence à un objet `System.Text.Encoding` correctement configuré==. Pour simplifier, supposons que l’encodage Unicode par défaut convienne.

**`StreamReader` hérite d’un type abstrait nommé `TextReader`, tout comme le type apparenté `StringReader`** (abordé plus loin dans ce chapitre). ***==La classe de base `TextReader` fournit un ensemble limité de fonctionnalités à chacun de ces descendants ; elle permet notamment de lire et d’observer un flux de caractères.==***

**Le type `StreamWriter` (ainsi que `StringWriter`, que vous étudierez plus loin dans ce chapitre) hérite d’une classe de base abstraite nommée `TextWriter`**. ***==Cette classe définit des membres permettant aux types dérivés d’écrire des données textuelles dans un flux de caractères donné. ==***

Pour vous aider à comprendre les capacités d'écriture de base des classes `StreamWriter` et `StringWriter`, le [[#Tableau 19-8 Membres clé de `TextWriter`|Tableau 19-8]] décrit les membres principaux de la classe de base abstraite `TextWriter`.

##### Tableau 19-8: Membres clé de `TextWriter`

| Membre                              | Description                                                                                                                                                                                                                       |
| ----------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Close()`                           | Cette méthode ferme le processus d'écriture et libère les ressources associées. Le tampon est alors automatiquement vidé (cette opération est fonctionnellement équivalente à l'appel de la méthode `Dispose()`).                 |
| `Flush()`                           | Cette méthode efface tous les tampons du processus d'écriture actuel et entraîne l'écriture de toutes les données mises en mémoire tampon sur le périphérique sous-jacent ; toutefois, elle ne ferme pas le processus d'écriture. |
| `NewLine`                           | Cette propriété indique la constante de nouvelle ligne pour la classe d'écriture dérivée. Le terminateur de ligne par défaut pour le système d'exploitation Windows est un retour chariot suivi d'un saut de ligne (`\r\n`).      |
| `Write()`<br>`WriteAsync()`         | Cette méthode surchargée écrit des données dans le flux de texte sans constante de nouvelle ligne.                                                                                                                                |
| `WriteLine()`<br>`WriteLineAsync()` | Cette méthode surchargée écrit des données dans le flux de texte avec une constante de nouvelle ligne.                                                                                                                            |

>[!Note]
Les deux derniers membres de la classe `TextWriter` vous sembleront probablement familiers. Pour rappel, le type `System.Console` possède les membres `Write()` et `WriteLine()` qui envoient des données textuelles vers le périphérique de sortie standard. En fait, la propriété `Console.In` encapsule un `TextReader`, et la propriété `Console.Out` encapsule un `TextWriter`.

**La classe dérivée `StreamWriter` fournit une implémentation appropriée pour les méthodes `Write()`, `Close()` et `Flush()`, et définit la propriété supplémentaire `AutoFlush`**. **==Lorsqu'elle est définie sur `true`, cette propriété force `StreamWriter` à vider toutes les données à chaque opération d'écriture==**. ==Notez que vous pouvez obtenir de meilleures performances en définissant `AutoFlush` sur `false`, à condition d'appeler systématiquement `Close()` une fois l'écriture terminée avec un `StreamWriter`.==

## Écriture dans un fichier texte

Pour observer le type `StreamWriter` en action, créez un nouveau projet d'application console nommé *StreamWriterReaderApp*. Le code suivant crée un nouveau fichier nommé *reminders.txt* dans le répertoire d'exécution courant, à l'aide de la méthode `File.CreateText()`. À l'aide de l'objet `StreamWriter` obtenu, vous pouvez ajouter des données textuelles au nouveau fichier.

```cs
Console.Title = "Fun with StreamWriter / StreamReader";
Console.WriteLine("***** Fun with StreamWriter / StreamReader *****\n");

// Récupère une StreamWriter et écrit des donnée string
using (StreamWriter writer = File.CreateText("reminders.txt"))
{
    writer.WriteLine("Don't forget Mother's Day this year...");
    writer.WriteLine("Don't forget Father's Day this year...");
    writer.WriteLine("Don't forget these numbers:");
    for (int i = 0; i < 10; i++)
        writer.Write(i + " ");
    // Insert une un nouvelle ligne
    writer.Write(writer.NewLine);
}
Console.WriteLine("Created file and wrote some toughts...");
Console.ReadLine();
File.Delete("reminders.txt");
```

Après avoir exécuté ce programme, vous pouvez examiner le contenu de ce nouveau fichier. **Vous trouverez ce fichier dans le répertoire racine de votre projet** (Visual Studio Code) **ou dans le dossier `bin\Debug\net10.0`** (Visual Studio) **car vous n'avez pas spécifié de chemin absolu lors de l'appel à `CreateText()` et l'emplacement du fichier est par défaut le répertoire d'exécution de l'assembly.**

## Lecture d'un fichier texte

Vous apprendrez ensuite à lire des données à partir d'un fichier par programmation en utilisant le type `StreamReader` correspondant. Rappelons que cette classe hérite de la classe abstraite `TextReader`, qui offre les fonctionnalités décrites dans le [[#Tableau 19-9 Membres Clé de `TextReader`|Tableau 19-9]].

##### Tableau 19-9: Membres Clé de `TextReader`

| Membre                              | Description                                                                                                                                                         |
| ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Peek()`                            | Renvoie le prochain caractère disponible (exprimé sous forme d'entier) sans modifier la position du lecteur. La valeur $-1$ indique que vous êtes à la fin du flux. |
| `Read()`<br>`ReadAsync()`           | Lis les données à partir d'un flux d'entrée.                                                                                                                        |
| `ReadBlock()`<br>`ReadBlockAsync()` | Lis un nombre maximal spécifié de caractères à partir du flux actuel et écrit les données dans une mémoire tampon, en commençant à un index spécifié.               |
| `ReadLine()`<br>`ReadLineAsync()`   | Lis une ligne de caractères du flux actuel et renvoie les données sous forme de chaîne (une chaîne vide (`null`) indique la fin du fichier (`EOF`)).                |
| `ReadToEnd()`<br>`ReadToEndAsync()` | Lis tous les caractères de la position actuelle jusqu'à la fin du flux et les renvoie sous forme d'une seule chaîne de caractères.                                  |

Si vous étendez maintenant l'application exemple actuelle pour utiliser un `StreamReader`, vous pouvez lire les données textuelles du fichier *reminders.txt*, comme indiqué ici :

```cs
Console.Title = "Fun with StreamWriter / StreamReader";
Console.WriteLine("***** Fun with StreamWriter / StreamReader *****\n");

...

// Maintenant on lis les données du fichier
Console.WriteLine("Here are your thoughts:\n");
using (StreamReader reader = File.OpenText("reminders.txt"))
{
    string input;
    while ((input = reader.ReadLine()) != null)
        Console.WriteLine(input);
}
Console.ReadLine();
```

Après avoir exécuté le programme, vous verrez les données des caractères contenues dans le fichier *reminders.txt* affichées dans la console.

## Création directe de types `StreamWriter`/`StreamReader`

**L'un des aspects déroutants de l'utilisation des types dans `System.IO` est qu'il est souvent possible d'obtenir un résultat identique par différentes approches**. Par exemple, ==vous avez déjà vu qu'il est possible d'utiliser la méthode `CreateText()` pour obtenir un `StreamWriter` avec `File` ou `FileInfo`==. ***==Il se trouve qu'il est également possible de travailler avec les `StreamWriters` et les `StreamReaders` d'une autre manière : en les créant directement==***. Par exemple, vous pourriez adapter l'application actuelle comme suit :

```cs
Console.Title = "Fun with StreamWriter / StreamReader";
Console.WriteLine("***** Fun with StreamWriter / StreamReader *****\n");

...

// Récupère un StreamWriter et écrit des donnée string
using (StreamWriter writer = new StreamWriter("reminders.txt"))
{
    //...
}

// Maintenant lis les donnée de fichier
using (StreamReader sr = new StreamReader("reminders.txt"))
{
    //...
}
```

**Bien qu'il puisse être un peu déroutant de voir autant d'approches apparemment identiques pour les entrées/sorties de fichiers, gardez à l'esprit qu'il en résulte une plus grande flexibilité.** Quoi qu'il en soit, vous êtes maintenant prêt à examiner le rôle des classes `StringWriter` et `StringReader`, étant donné que vous avez vu comment déplacer des données de type caractère vers et depuis un fichier donné à l'aide des types `StreamWriter` et `StreamReader`.

# Utilisation de `StringWriter` et `StringReader`

**Vous pouvez utiliser les types `StringWriter` et `StringReader` pour traiter les informations textuelles comme un flux de caractères en mémoire**. ==Cela peut s'avérer utile lorsque vous souhaitez ajouter des informations textuelles à une mémoire tampon sous-jacente==. Le projet d'application console suivant (nommé *StringReaderWriterApp*) illustre ce principe en écrivant un bloc de données de type chaîne dans un objet `StringWriter`, plutôt que dans un fichier sur le disque dur local (==n'oubliez pas d'importer `System.Text`==) :

```cs
using System.Text;

Console.Title = "Fun with StringWriter / StringReader";
Console.WriteLine("***** Fun with StringWriter / StringReader *****\n");

// Crée un StringWriter et émet des caractères vers la mémoire
using (StringWriter strWriter = new())
{
    strWriter.WriteLine("Don't forget Mother's Day this year...");
    // Récupère une copie du contenu (enregistré dans un string)
    // et affich dans la console
    Console.WriteLine($"content of StringWriter:\n{strWriter}");
}
Console.ReadLine();
```

**Les classes `StringWriter` et `StreamWriter` dérivent toutes deux de la même classe de base (`TextWriter`), leur logique d'écriture est donc similaire**. Cependant, ***==compte tenu de la nature de `StringWriter`, il est important de noter que cette classe permet d'utiliser la méthode `GetStringBuilder()` suivante pour extraire un objet `System.Text.StringBuilder`==*** :

```cs
// Crée un StringWriter et émet des caractères vers la mémoire
using (StringWriter strWriter = new())
{
    strWriter.WriteLine("Don't forget Mother's Day this year...");
    Console.WriteLine($"Content of StringWriter:\n{strWriter}");

    // Récupère l'objet Stringbuilder interne
    StringBuilder sb = strWriter.GetStringBuilder();
    sb.Insert(0, "Hey!! ");
    // Plus nécessaire d'appeler ToString
    Console.WriteLine($"-> {sb}");
    // Syntaxe intéressante !
    sb.Remove(0, "Hey!! ".Length);
}
Console.ReadLine();
```

**Quand vous voulez lire depuis un flux de données de type caractère, vous pouvez utiliser la classe `StringReader` correspondante**. ==Cette classe fonctionne== (comme prévu) ==de manière identique à la classe `StreamReader`==. ***==En fait, la classe `StringReader` se contente de redéfinir les membres hérités pour lire un bloc de données de type caractère, plutôt qu'un fichier, comme illustré ici :==***

```cs
using System.Text;

Console.Title = "Fun with StringWriter / StringReader";
Console.WriteLine("***** Fun with StringWriter / StringReader *****\n");

// Crée un StringWriter et émet des caractères vers la mémoire
using (StringWriter strWriter = new())
{
	...

    Console.WriteLine("Content with StringReader:\n");
    using (StringReader strReader = new(strWriter.ToString()))
    {
        string input = null;
        while ((input = strReader.ReadLine()) != null)
            Console.WriteLine(input);
    }
}
Console.ReadLine();
```

# Utilisation des classes `BinaryWriter` et `BinaryReader`

>[!success] Toujours d'actualité et important. À ne pas confondre avec `Binaryformatter`

**Les derniers ensembles lecteur/écrivain que vous examinerez dans cette section sont `BinaryReader` et `BinaryWriter`. Tous deux héritent directement de `System.Object`**. ***==Ces types vous permettent de lire et d'écrire des types de données discrets dans un flux sous-jacent, au format binaire compact==***. **La classe `BinaryWriter` définit une méthode `Write()` hautement surchargée pour placer un type de données dans le flux sous-jacent**. Outre le membre `Write()`, **==`BinaryWriter` fournit des membres supplémentaires qui vous permettent d'obtenir ou de définir le type dérivé du flux ; elle offre également la prise en charge de l'accès aléatoire aux données==** (voir le [[#Tableau 19-10 Membre clé de `BinaryWriter`|Tableau 19-10]]).

##### Tableau 19-10: Membre clé de `BinaryWriter`
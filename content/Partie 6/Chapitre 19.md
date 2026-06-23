---
title: "Chapitre 19: Entrées/Sorties de Fichiers et Sérialisation d'Objets"
publish: false
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

**De nombreux types de l'espace de noms `System.IO` permettent la manipulation programmatique de répertoires et de fichiers physiques. Cependant, d'autres types prennent en charge la lecture et l'écriture de données dans des tampons de chaînes, ainsi que dans des emplacements mémoire bruts.** Le [[#Tableau 19-1 Membres clé de l'espace de noms `System.IO`|Tableau 19-1]] présente les classes principales (non abstraites), offrant une vue d'ensemble des fonctionnalités de `System.IO`.

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

`System.IO` fournit quatre classes permettant de manipuler des fichiers individuels et d'interagir avec la structure de répertoires d'une machine. Les deux premiers types, Directory et File, exposent des opérations de création, de suppression, de copie et de déplacement à l'aide de divers membres statiques. Les types FileInfo et DirectoryInfo, étroitement liés, exposent des fonctionnalités similaires sous forme de méthodes d'instance (il est donc nécessaire de les allouer avec le mot-clé new). Les classes Directory et File héritent directement de System.Object, tandis que DirectoryInfo et FileInfo dérivent du type abstrait FileSystemInfo.

Les classes FileInfo et DirectoryInfo sont généralement plus appropriées pour obtenir des informations complètes sur un fichier ou un répertoire (par exemple, la date de création ou les autorisations de lecture/écriture), car leurs membres renvoient généralement des objets fortement typés. En revanche, les membres des classes Directory et File renvoient généralement de simples chaînes de caractères plutôt que des objets fortement typés. Il ne s'agit toutefois que d'une indication ; dans de nombreux cas, vous pouvez obtenir le même résultat en utilisant File/FileInfo ou Directory/DirectoryInfo.

## La classe de base abstraite `FileSystemInfo`

Les types DirectoryInfo et FileInfo héritent de nombreux comportements de la classe de base abstraite FileSystemInfo. Le plus souvent, vous utilisez les membres de la classe FileSystemInfo pour découvrir les caractéristiques générales (telles que la date de création, divers attributs, etc.) d'un fichier ou d'un répertoire donné. Le tableau 19-2 répertorie certaines propriétés essentielles.

##### Tableau 19-2: Propriété de `FileSystemInfo`

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

`FileSystemInfo` définit également la méthode `Delete()`. Celle-ci est implémentée par les types dérivés pour supprimer un fichier ou un répertoire donné du disque dur. De plus, vous pouvez appeler `Refresh()` avant d'obtenir les informations d'attribut afin de vous assurer que les statistiques relatives au fichier (ou répertoire) courant ne sont pas obsolètes.

# Utilisation du type `DirectoryInfo`

Le premier type d'E/S créable que vous examinerez est la classe `DirectoryInfo`. Cette classe contient un ensemble de membres utilisés pour créer, déplacer, supprimer et énumérer les répertoires et sous-répertoires. En plus des fonctionnalités fournies par sa classe de base (`FileSystemInfo`), `DirectoryInfo` offre les principaux membres détaillés dans le tableau 19-3.








---
publish: true
---
# <big><big><big><b><font color =green>Présentation d'Entity Framework Core</font></b></big></big></big>

 >[!tip]- Compte rendu moderne 
>## Ce qui est TRÈS pertinent (À maîtriser absolument)
>
>- **Le concept d'ORM (Object-Relational Mapper)** : Comprendre comment EF Core traduit vos classes C# en tables de base de données, et vos objets en lignes. C’est le cœur du sujet. 
>- **La classe `DbContext`** : C'est la pièce maîtresse d'EF Core. Elle représente votre session avec la base de données et suit les modifications de vos objets. 
>- **`DbSet<T>`** : Les collections au sein de votre `DbContext` qui mappent directement vos tables (ex: `DbSet<Car>`). 
>- **`OnModelCreating()` et l'API Fluent** : C'est ici que vous configurez finement vos tables en C#. C'est crucial car cela va remplacer les attributs (Data Annotations) qui sont moins flexibles. 
>- **`SaveChanges()` et la gestion des transactions** : Comprendre comment EF Core regroupe (batch) vos modifications pour les envoyer efficacement à la base de données. 
>
>
>## Ce qui est à ADAPTER (Le piège SQL Server)
>
>Le livre est écrit exclusivement pour SQL Server. En lisant ce chapitre avec **PostgreSQL**, vous devez faire trois ajustements majeurs : 
>
> 1. Le package de base de données (Provider)
>
>- **Dans le livre** : On vous demandera d'installer `Microsoft.EntityFrameworkCore.SqlServer`.
>- **Pour votre cas** : Ignorez-le. Installez uniquement le fournisseur officiel PostgreSQL : **`Npgsql.EntityFrameworkCore.PostgreSQL`**.
>
> 2. La configuration du `DbContext`
>
>Dans la méthode `OnConfiguring` (ou lors de l'injection de dépendances), le livre utilise `.UseSqlServer(connectionString)`. Vous devrez le remplacer par :
>
>
>```cs
>optionsBuilder.UseNpgsql(connectionString);
>```

Le chapitre précédent a examiné les principes fondamentaux d'ADO.NET. Comme vous l'avez vu, ==ADO.NET permet aux développeurs .NET de travailler avec des données relationnelles.== **Bien qu'ADO.NET soit un outil** *efficace* **pour la manipulation de données, il n'est pas nécessairement** *efficient*. **L'efficience dont je parle concerne l'efficience du développeur.** Pour améliorer cette efficience, **Microsoft a introduit un nouveau framework d'accès aux données appelé** *Entity Framework* (ou simplement *EF*) **dans .NET 3.5 Service Pack 1. **

***==EF offre la possibilité d'interagir avec les données des bases de données relationnelles à l'aide d'un modèle objet qui correspond directement aux objets métier (ou objets de domaine) de votre application.==*** Par exemple, **au lieu de traiter un lot de données comme une collection de lignes et de colonnes, vous pouvez opérer sur une collection d'objets fortement typés, appelés** *entités*. **==Ces entités sont stockées dans des classes de collections spécialisées compatibles LINQ, permettant ainsi des opérations d'accès aux données à l'aide de code C#.==** Les classes de collection permettent d'interroger le magasin de données en utilisant la même grammaire LINQ que celle présentée au [[Chapitre 13#Comprendre le rôle de LINQ|Chapitre 13]]

**Outre la possibilité d'utiliser vos données comme modèle de domaine applicatif** (au lieu d'un modèle de base de données normalisé), **EF offre des gains d'efficacité tels que le suivi d'état, les opérations par unité de travail et la prise en charge native des transactions.**

**Entity Framework Core est une réécriture complète d'Entity Framework 6. Il est basé sur le framework .NET 6, ce qui permet à EF Core de fonctionner sur plusieurs plateformes.** **==La réécriture d'EF Core a permis à l'équipe d'ajouter de nouvelles fonctionnalités et d'améliorer les performances, fonctionnalités qui n'auraient pas pu être raisonnablement implémentées dans EF 6.==**

Recréer un framework entier à partir de zéro exige une analyse approfondie des fonctionnalités qui seront prises en charge et de celles qui seront abandonnées. *==Parmi les fonctionnalités d'EF 6 absentes d'EF Core==* (et qui ne seront probablement jamais ajoutée), *==figure la prise en charge de l'Entity Designer==*. **EF Core ne prend en charge que le développement** *"code-first"*. Le nom est vraiment mal choisi, car il laisse entendre qu'EF Core est inutilisable avec une base de données existante. **Il signifie en réalité « sans concepteur », mais ce n'est pas le nom qui a été retenu.** ***==EF Core peut être utilisé avec des bases de données existantes qui peuvent être générées à partir de classes d'entités et d'un `DbContext` dérivé, ou bien vous pouvez utiliser EF Core pour créer/mettre à jour votre base de données à partir de vos classes d'entités et de votre `DbContext` dérivé==***. J'aborderai ces deux cas de figure prochainement.

**À chaque nouvelle version, EF Core a ajouté des fonctionnalités présentes dans EF 6, ainsi que de nouvelles fonctionnalités inédites**. La version 3.1 a considérablement réduit la liste des fonctionnalités essentielles manquantes dans EF Core (par rapport à EF 6), et la version 5.0 a encore davantage réduit cet écart. La sortie d'EF Core 6.0 a consolidé le framework, et désormais, ***==pour la plupart des projets, EF Core offre tout le nécessaire.==***

**Ce chapitre et les trois suivants vous présenteront l'accès aux données avec EF Core.** Vous découvrirez : ==la création d'un modèle de domaine, le mappage des classes d'entités et de leurs propriétés aux tables et colonnes de la base de données, la mise en œuvre du suivi des modifications,== ***==l'utilisation de l'interface de ligne de commande (CLI) d'EF Core pour la génération de code et les migrations, ainsi que le rôle de la classe `DbContext`==***. ==Vous apprendrez également à lier les entités avec des propriétés de navigation, des transactions et la gestion de la concurrence, pour ne citer que quelques-unes des fonctionnalités explorées.== **Le quatrième et dernier chapitre consacré à EF Core met en pratique la couche d'accès aux données à l'aide d'une série de tests d'intégration. Ces tests illustrent l'utilisation d'EF Core pour les opérations CRUD (création, lecture, mise à jour et suppression).**

**==À la fin de ces chapitres, vous disposerez de la version finale de la couche d'accès aux données pour notre base de données `AutoLot`==**. **Avant d'aborder EF Core, parlons des mappeurs objet-relationnel en général.**


>[!note]
>Quatre chapitres ne suffisent pas à couvrir l'intégralité d'Entity Framework Core, car des ouvrages entiers (certains de la taille de celui-ci) sont consacrés exclusivement à EF Core. L'objectif de ces chapitres est de vous fournir les connaissances pratiques nécessaires pour vous permettre de commencer à utiliser EF Core dans vos applications métier.

# Mappeurs objet-relationnel (ORM)

==ADO.NET vous fournit une structure permettant de sélectionner, insérer, mettre à jour et supprimer des données à l'aide de connexions, de commandes et de lecteurs de données==. *==Bien que cela soit très pratique, ces aspects d'ADO.NET vous obligent à traiter les données récupérées d'une manière étroitement liée au schéma physique de la base de données.==* Par exemple, **lors de la récupération d'enregistrements de la base de données, vous ouvrez une connexion, créez et exécutez un objet commande, puis utilisez un lecteur de données pour parcourir chaque enregistrement en utilisant les noms de colonnes spécifiques à la base de données.**

**Lorsque vous utilisez ADO.NET, vous devez toujours tenir compte de la structure physique de la base de données.** *==Vous devez connaître le schéma de chaque table de données, rédiger des requêtes SQL potentiellement complexes pour interagir avec ces tables, suivre les modifications apportées aux données récupérées (ou ajoutées), etc==*. **Cela peut vous contraindre à écrire du code C# assez verbeux, car C# ne communique pas directement avec le schéma de la base de données.**

*==Pour compliquer les choses, la construction physique d'une base de données est généralement axée sur les structures de base de données telles que les clés étrangères, les vues, les procédures stockées et la normalisation des données, et non sur la programmation orientée objet.==*

**Un autre souci pour les développeurs d'applications est le suivi des modifications.** *==L'extraction des données de la base de données est une étape du processus, mais toute modification, ajout ou suppression doit être suivi par le développeur afin d'être persistée dans la base de données.==*

**==La disponibilité des frameworks de *mappage objet-relationnel* (ORM) dans .NET a considérablement simplifié l'accès aux données en prenant en charge la majeure partie des tâches CRUD pour le développeur.==** **Ce dernier crée un mappage entre les objets .NET et la base de données relationnelle, et l'ORM gère les connexions, la génération des requêtes, le suivi des modifications et la persistance des données. Le développeur peut ainsi se concentrer sur les besoins métiers de l'application.**

>[!note]
>**Il est important de se rappeler que les ORM ne sont pas des solutions miracles.** Chaque décision, implique des compromis. Les ORM réduisent la charge de travail des développeurs lors de la création de couches d'accès aux données, mais peuvent aussi, s'ils sont mal utilisés, engendrer des problèmes de performance et de mise à l'échelle. Utilisez les ORM pour les opérations CRUD et exploitez la puissance de votre base de données pour les opérations ensemblistes.

***==Bien que les différents ORM présentent de légères différences dans leur fonctionnement et leur utilisation, ils possèdent tous les mêmes composants et visent le même objectif : simplifier les opérations d'accès aux données.==*** **Les entités sont des classes mappées aux tables de la base de données. Un type de collection spécialisé contient une ou plusieurs entités. Un mécanisme de suivi des modifications enregistre l'état des entités et toutes les modifications, ajouts et/ou suppressions qui leur sont apportés, et une structure centrale contrôle les opérations.**

# Comprendre le rôle  d'Entity Framework Core

***En interne, EF Core utilise l'infrastructure ADO.NET que vous avez déjà examinée dans le chapitre précédent.*** **==Comme toute interaction ADO.NET avec un magasin de données, EF Core utilise un fournisseur de données ADO.NET pour les interactions avec ce magasin.==** ***==Avant qu'un fournisseur de données ADO.NET puisse être utilisé par EF Core, il doit être mis à jour pour une intégration complète avec EF Core. En raison de cette fonctionnalité supplémentaire, il est possible que vous disposiez de moins de fournisseurs de données EF Core que de fournisseurs de données ADO.NET.==***

**==L'avantage d'EF Core utilisant le modèle de fournisseur de base de données ADO.NET réside dans sa capacité à combiner les paradigmes d'accès aux données EF Core et ADO.NET au sein d'un même projet, augmentant ainsi vos fonctionnalités.==** Par exemple, ==l'utilisation d'EF Core pour fournir la connexion, le schéma et le nom de la table pour les opérations de copie en masse exploite les capacités de mappage d'EF Core et la fonctionnalité BCP intégrée à ADO.NET.== **Cette approche combinée fait d'EF Core un outil parmi d'autres.**

**En constatant la gestion simplifiée et efficace de la majeure partie de l'infrastructure d'accès aux données, EF Core deviendra probablement votre solution de prédilection pour l'accès aux données.**

>[!note]
>De nombreuses bases de données tierces (par exemple, Oracle et MySQL) proposent des fournisseurs de données compatibles avec EF. Si vous n'utilisez pas SQL Server, consultez votre fournisseur de base de données pour plus d'informations ou rendez-vous sur https://docs.microsoft.com/en-us/ef/core/providers pour obtenir la liste des fournisseurs de données EF Core disponibles.

**EF Core s'intègre parfaitement au processus de développement dans les contextes de formulaires sur données (ou d'API sur données). Les opérations sur un petit nombre d'entités, utilisant le modèle d'unité de travail pour garantir la cohérence, constituent le point fort d'EF Core.** ~~En revanche, il est moins adapté aux opérations sur des données à grande échelle, telles que les applications d'entrepôt de données ETL (extraction, transformation et chargement) ou les situations de reporting complexes.~~

>[!important] Note moderne (avec Gemini)
> 1. **Les opérations de masse ne nécessitent plus l'Unité de Travail**
>
>	 EF Core intègre désormais les méthodes **`ExecuteUpdate()`** et **`ExecuteDelete()`**. Elles permettent d'exécuter des requêtes de mise à jour ou de suppression de masse directement en base de données à partir d'une simple expression LINQ, **sans charger une seule ligne en mémoire** et sans utiliser le Change Tracker. Le gain de performance est massif (souvent de l'ordre de 300x à 500x plus rapide). 
>
>2. **Le traitement par lots automatique (Batching)**
>
>	Lors de l'utilisation de `AddRange()`, EF Core regroupe automatiquement les instructions `INSERT` en un nombre minimal de voyages réseau (Roundtrips). Sous PostgreSQL (avec le provider Npgsql), la taille des lots par défaut est très généreuse (jusqu'à 1000 lignes par lot), ce qui rend EF Core tout à fait capable de gérer des imports de quelques dizaines de milliers de lignes de manière très performante, sans aucun outil tiers.
>	
>## Ce qui reste vrai (Les limites d'EF Core)
>
>- **Le vrai ETL (Millions de lignes)** : Si vous devez insérer 1 million de lignes par seconde, le passage par un ORM (qui doit analyser le modèle et générer du SQL) reste moins performant qu'un flux binaire pur (comme le `NpgsqlBinaryImporter` que nous avons codé au chapitre précédent). 
>- **Le Reporting ultra-complexe** : Pour générer des rapports de Business Intelligence avec des dizaines de jointures dynamiques et des fonctions de fenêtrage SQL avancées, LINQ peut devenir difficile à écrire ou générer du SQL sous-optimal. Dans ce cas précis, les requêtes SQL brutes ou des outils comme Dapper restent préférés par les administrateurs de données. 

# Les éléments constitutifs d'Entity Framework

**Les principaux composants d'EF Core sont `DbContext`, `ChangeTracker`, le type de collection spécialisé `DbSet`, les fournisseurs de base de données et les entités de l'application.** Pour suivre ce chapitre, créez une nouvelle application console nommée *AutoLot.Samples* et ajoutez les packages `Microsoft.EntityFrameworkCore`, `Microsoft.EntityFrameworkCore.Design` et le package EF lié à votre fournisseur de données (pour moi : `Npgsql.EntityFrameworkCore.PostgreSQL`. Voir la [table des fournisseurs de données](https://learn.microsoft.com/en-us/ef/core/providers/?tabs=dotnet-core-cli) disponible ici) **N'oubliez pas de désactiver les types référence pouvant être nuls dans le fichier projet :**

```xml
<PropertyGroup>
  <OutputType>Exe</OutputType>
  <TargetFramework>net6.0</TargetFramework>
  <ImplicitUsings>enable</ImplicitUsings>
  <Nullable>disable</Nullable>
</PropertyGroup>
```

>[!note]
>Si vous préférez utiliser la console du gestionnaire de packages NuGet pour exécuter les commandes EF Core, installez le package `Microsoft.EntityFrameworkCore.Tools`. Ce texte ne traite pas des commandes de type NuGet, car l’interface de ligne de commande (CLI) fonctionne sur toutes les plateformes et ne dépend pas de Visual Studio.

>[!warning] Si on utilise un fichier *appsettings.json* et/ou un fichier *.env*
>N'oubliez pas les packages `Microsoft.Extensions.Configuration` et `Microsoft.Extensions.Configuration.EnvironmentVariables`.
>
>>[!tip] À partir de maintenant, je vais utiliser le package `DotNetEnv`  pour charger les variables d'environnements du fichier *.env*.

Ajoutez un nouveau fichier nommé *GlobalUsings.cs*, supprimez le code du modèle et mettez à jour le fichier pour qu'il corresponde à ce qui suit :

```cs
global using System.ComponentModel.DataAnnotations;
global using System.ComponentModel.DataAnnotations.Schema;
global using Microsoft.EntityFrameworkCore;
global using Microsoft.EntityFrameworkCore.ChangeTracking;
global using Microsoft.EntityFrameworkCore.Design;
global using Microsoft.EntityFrameworkCore.Metadata;
global using Microsoft.EntityFrameworkCore.Metadata.Builders;
```

## La classe `DbContext`

**La classe `DbContext` est le composant principal d'EF Core et permet d'accéder à la base de données via la propriété `Database`.** ***==`DbContext` gère l'instance `ChangeTracker`, expose la méthode virtuelle `OnModelCreating()` pour accéder à l'API Fluent, contient toutes les propriétés `DbSet<T>` et fournit la méthode `SaveChanges` pour enregistrer les données dans la base de données.==*** **Elle n'est pas utilisée directement, mais via une classe personnalisée qu'hérite de `DbContext.` C'est dans la classe dérivée que les propriétés `DbSet<T>` sont placées.** Le [[#Tableau 21-1 Membre communs de `DbContext`|Tableau 21-1]] présente certains des membres les plus fréquemment utilisés de `DbContext`.

###### Tableau 21-1: Membre communs de `DbContext`

| Membre de `DbContext`                                                            | Description                                                                                                                                                                                                                                                                                                                                                                         |
| -------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Database`                                                                       | Permet d'accéder aux informations et fonctionnalités liées à la base de données, y compris l'exécution d'instructions SQL.                                                                                                                                                                                                                                                          |
| `Model`                                                                          | Les métadonnées relatives à la structure des entités, aux relations entre elles et à leur correspondance avec la base de données. *Remarque : Cette propriété n’est généralement pas utilisée directement.*                                                                                                                                                                         |
| `ChangeTracker`                                                                  | Fournit un accès aux informations et aux opérations pour les instances d'entités suivies par ce `DbContext`.                                                                                                                                                                                                                                                                        |
| `DbSet<T>`                                                                       | Ces propriétés ne font pas partie intégrante de `DbContext`, mais sont ajoutées à la classe dérivée personnalisée `DbContext`. Elles sont de type `DbSet<T>` et servent à interroger et à enregistrer des instances d'entités applicatives. Les requêtes LINQ portant sur ces propriétés `DbSet<T>` sont traduites en requêtes SQL.                                                 |
| `Entry()`                                                                        | Permet d'accéder aux informations et opérations de suivi des modifications de l'entité, comme le chargement explicite des entités associées ou la modification de `EntityState`. Peut également être appelée sur une entité non suivie pour la rendre suivie.                                                                                                                       |
| `Set<TEntity>()`                                                                 | Crée une instance de la propriété `DbSet<T>` qui peut être utilisée pour interroger et conserver des données.                                                                                                                                                                                                                                                                       |
| `SaveChanges()`/`SaveChangesAsync()`                                             | Enregistre toutes les modifications apportées aux entités dans la base de données et renvoie le nombre d'enregistrements concernés. S'exécute dans le cadre d'une transaction (implicite ou explicite).                                                                                                                                                                             |
| `Add()`/`AddRange()`<br>`Update()`/`UpdateRange()`<br>`Remove()`/`RemoveRange()` | Méthodes permettant d'ajouter, de mettre à jour et de supprimer des instances d'entité. Les modifications ne sont enregistrées que si `SaveChanges()` est exécuté avec succès. Des versions asynchrones sont également disponibles. *Remarque : Bien que disponibles sur le `DbContext` dérivé, ces méthodes sont généralement appelées directement sur les propriétés `DbSet<T>`.* |
| `Find()`                                                                         | Recherche une entité d'un type donné avec les valeurs de clé primaire spécifiées. Des versions asynchrones sont également disponibles. Remarque : Bien que disponibles sur le `DbContext` dérivé, ces méthodes sont généralement appelées directement sur les propriétés `DbSet<T>`.                                                                                                |
| `Attach()`/`AttachRange()`                                                       | Démarre le suivi d'une entité (ou d'une liste d'entités). Des versions asynchrones sont également disponibles. *Remarque : Bien que disponibles sur le `DbContext` dérivé, ces méthodes sont généralement appelées directement sur les propriétés `DbSet<T>`.*                                                                                                                      |
| `SavingChanges()`                                                                | Événement déclenché au début d'un appel à `SaveChanges()`/`SaveChangesAsync()`.                                                                                                                                                                                                                                                                                                     |
| `SavedChanges()`                                                                 | Événement déclenché à la fin d'un appel à `SaveChanges()`/`SaveChangesAsync()`.                                                                                                                                                                                                                                                                                                     |
| `SaveChangesFailed`                                                              | Événement déclenché si un appel à `SaveChanges()`/`SaveChangesAsync()` échoue.                                                                                                                                                                                                                                                                                                      |
| `OnModelCreating()`                                                              | Appelée lorsqu'un modèle a été initialisé, mais avant sa finalisation. Les méthodes de l'API Fluent sont placées dans cette méthode pour finaliser la structure du modèle.                                                                                                                                                                                                          |
| `OnConfiguring()`                                                                | Ce générateur permet de créer ou de modifier les options de `DbContext`. Il s'exécute à chaque création d'une instance de `DbContext`. *Remarque : Il est recommandé de ne pas l'utiliser et de privilégier `DbContextOptions` pour configurer l'instance de `DbContext` à l'exécution et d'utiliser une instance de `IDesignTimeDbContextFactory` lors de la conception.           |

### Création d'un `DbContext` dérivé

**La première étape dans EF Core consiste à créer une classe personnalisée héritant de `DbContext`. Ajoutez ensuite un constructeur qui accepte une instance fortement typée de `DbContextOptions` (voir ci-après) et la transmet à la classe de base.** Créez un fichier nommé *ApplicationDbContext.cs* et mettez à jour le code comme suit :

```cs
namespace AutoLot.Samples;

public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options) { }
}
```

***==Il s'agit de la classe utilisée pour accéder à la base de données et interagir avec les entités, le suivi des modifications et tous les composants d'EF Core.==***

### Configuration du `DbContext`

**L'instance `DbContext` est configurée à l'aide d'une instance de la classe `DbContextOptions`.** ***==Cette instance est créée avec `DbContextOptionsBuilder`, car la classe `DbContextOptions` n'est pas conçue pour être directement construite dans votre code.==*** **Via l'instance `DbContextOptionsBuilder`, le fournisseur de base de données est sélectionné (ainsi que ses paramètres spécifiques), et les options générales du `DbContext` EF Core (telles que la journalisation) sont définies.** ==L'instance de `DbContextOptions` est ensuite injectée dans le `DbContext` de base lors de l'exécution.== Vous verrez cela en pratique dans la section suivante.

**Cette fonctionnalité de configuration dynamique permet de modifier les paramètres à l'exécution en sélectionnant simplement des options différentes (par exemple, Postgres au lieu de SQL Server) et en créant une nouvelle instance de votre `DbContext` dérivé.**

### La fabrique `DbContext` en mode conception

**La fabrique `DbContext`  en mode conception est une classe qui implémente l'interface `IDesignTimeDbContextFactory<T>` où `T` est la classe `DbContext` dérivée.** ***==L'interface possède une méthode, `CreateDbContext()`, que vous devez implémenter pour créer une instance de votre `DbContext` dérivé.==*** *==Cette classe n'est pas destinée à la production , mais uniquement au développement, et existe principalement pour les outils en ligne de commande d'EF Core, que vous explorerez prochainement.==* ==Dans les exemples de ce chapitre et du suivant, elle sera utilisée pour créer de nouvelles instances de `ApplicationDbContext`.==

>[!note]
>Il est considéré comme une mauvaise pratique d'utiliser la fabrique `DbContext` pour créer des instances de votre classe `DbContext` dérivée. N'oubliez pas qu'il s'agit d'un exemple de code à des fins pédagogiques, et que cette pratique permet de conserver un code plus clair. Vous découvrirez comment instancier correctement votre classe `DbContext` dérivée dans les chapitres consacrés à Windows Presentation Foundation et à ASP.NET Core.

**==La classe `ApplicationDbContextFactory` suivante utilise la méthode `CreateDbContext()` pour créer un `DbContextOptionsBuilder` fortement typé pour la classe `ApplicationDbContext`, définit le fournisseur de base de données sur le fournisseur PostgreSQL==** (en utilisant la chaîne de connexion de l'instance Docker du [[Chapitre 20#Se connecter à la base de donnée Postgres|Chapitre 20]]), **==puis crée et retourne une nouvelle instance de l'`ApplicationDbContext` :==**

```cs
using Microsoft.Extensions.Configuration;
using Npgsql;

namespace AutoLot.Samples;

public class ApplicationDbContextFactory
    : IDesignTimeDbContextFactory<ApplicationDbContext>
{
    public ApplicationDbContext CreateDbContext(string[] args)
    {
        // Charge explicitement le fichier .env
        // au mement où la commende CLI appelle la fabrique
        DotNetEnv.Env.TraversePath().Load();

        // Lit Environment et convertit __ en :
        var config = new ConfigurationBuilder()
            .AddEnvironmentVariables()
            .Build();

        var optionBuilder = new DbContextOptionsBuilder<ApplicationDbContext>();
        var conStringBuilder = new NpgsqlConnectionStringBuilder
        {
            Host = config["Postgres:Host"],
            Username = config["Postgres:Username"],
            Database = config["Postgres:Database"],
            Password = config["Postgres:Password"],
            Pooling = true, // Recommandé pour PostgreSQL
        };
        optionBuilder.UseNpgsql(conStringBuilder.ConnectionString);
        return new ApplicationDbContext(optionBuilder.Options);
    }
}
```


>[!note]
>Le nom de la base de données utilisé dans ces exemples est `AutoLotSamples`, et non `AutoLot`, qui était le nom utilisé au [[Chapitre 20#Création de la base de donnée AutoLot|Chapitre 20]]. La base de données `AutoLot` sera mise à jour vers sa version finale à partir du [[Chapitre 22]].

*==Là encore, la fabrique de contexte est conçue pour l'interface de ligne de commande d'EF Core afin de créer une instance de la classe dérivée `DbContext`, et non pour une utilisation en production.==* **L'interface de ligne de commande utilise la fabrique lors de l'exécution d'actions telles que la création ou l'application de migrations de base de données.** ***==L'une des principales raisons pour lesquelles il est déconseillé de l'utiliser en production est la chaîne de connexion codée en dur. Comme elle est destinée à la conception, l'utilisation d'une chaîne de connexion définie pointant vers la base de données de développement fonctionne parfaitement.==***

>[!success] Précisions sur le paragraphe précédent (Avec Gemini)
>
L'auteur justifie le bannissement de la Factory en production par le fait qu'elle contient une chaîne de connexion codée en dur.
>
>Puisque **mon code utilise un fichier `.env`** et des variables d'environnement à l'intérieur de la fabrique (`Environment.GetEnvironmentVariable`), **j'ai éliminé le principal défaut reproché par l'auteur.**
>
>## Pourquoi cela reste une "règle d'or" (Ground Rule) de ne pas l'utiliser en production
>
>La raison n'est plus la sécurité, mais le **cycle de vie de l'application** :
>
>- **Le rôle de la CLI (Conception)** : Lorsque l'on tape `dotnet ef`, la CLI exécute la fabrique, crée le contexte, applique la migration, puis le programme s'arrête immédiatement. C'est un processus éphémère.
>- **Le rôle du Runtime (Production)** : Une application Web ou une API tourne en continu. Elle doit gérer des centaines de requêtes par seconde. Si elle utilisait la fabrique pour créer le `DbContext` à chaque fois, elle ne pourrait pas bénéficier de l'injection de dépendances de ASP.NET Core, qui optimise la réutilisation des connexions (Pooling) et gère l'isolation des transactions par utilisateur.

**La méthode `CreateDbContext()` prend un tableau de chaînes de caractères comme argument. Bien que non utilisée dans les versions précédentes, la prise en charge du passage d'arguments depuis la ligne de commande à la méthode `CreateDbContext()` de `IDesignTimeDbContextFactory` a été ajoutée dans EF Core 5.**

### `OnModelCreating`

**La classe `DbContext` de base expose la méthode `OnModelCreating` qui permet de structurer vos entités à l'aide de l'API Fluent.** Ce point sera abordé plus en détail ultérieurement dans ce chapitre. Pour l'instant, ajoutez le code suivant à la classe `ApplicationDbContext` :

```cs
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
	// Les appels à l'API Fluent vont ici
}
```

### Enregistrement des modifications

**Pour enregistrer les modifications (ajout, mise à jour ou suppression) apportées aux entités, appelez la méthode `SaveChanges()` (ou `SaveChangesAsync()`) sur le `DbContext` dérivé.** ***==Les méthodes `SaveChanges()`/`SaveChangesAsync()` encapsulent les appels à la base de données dans une transaction implicite et les enregistrent comme une unité de travail.==*** Les transactions sont abordées ci-après, et le suivi des modifications est traité plus loin dans cette section.

Ajoutez l'instruction `using` globale suivante au fichier *GlobalUsings.cs* :

```cs
global using AutoLot.Samples;
```

Supprimez tout code du fichier *Program.cs* et mettez-le à jour pour qu'il corresponde à ce qui suit :

```cs
Console.Title = "Fun with Entity Framework Core";
Console.WriteLine("***** Fun with Entity Framework Core *****\n");

static void SampleSaveChanges()
{
    // Une fabrique n'est pas censée être utilisée comme ça,
    // mais c'est un code de démonstration :-)
    var context = new ApplicationDbContextFactory().CreateDbContext(null);

    // Effecture des changements...
    context.SaveChanges();
}
```

De nombreux exemples de sauvegarde des modifications seront présentés dans le reste de ce chapitre (et de ce livre).

#### Gestion des transactions et des points de sauvegarde

Comme indiqué précédemment, **EF Core encapsule chaque appel à `SaveChanges()`/`SaveChangesAsync()` dans une transaction implicite.** ==Par défaut, la transaction utilise le niveau d'isolation de la base de données.== ***==Pour un contrôle plus précis, vous pouvez intégrer le `DbContext` dérivé à une transaction explicite au lieu d'utiliser la transaction implicite par défaut.==*** **Pour exécuter une opération dans une transaction explicite, créez une transaction à l'aide de la propriété `Database` du `DbContext` dérivé. Effectuez vos opérations comme d'habitude, puis validez ou annulez la transaction.** Voici un extrait de code qui illustre ce comportement :

```cs
static void TransactedSaveChanges()
{
    // Une fabrique n'est pas censée être utilisée comme ça,
    // mais c'est un code de démonstration :-)
    var context = new ApplicationDbContextFactory().CreateDbContext(null);

    using var trans = context.Database.BeginTransaction();
    try
    {
        // Créer, changer, supprimer des choses
        context.SaveChanges();
        trans.Commit();
    }
    catch
    {
      trans.Rollback();
    }
```

==Les points de sauvegarde pour les transactions EF Core ont été introduits dans EF Core 5.== **Lorsqu'une méthode `SaveChanges()` ou `SaveChangesAsync()` est appelée et qu'une transaction est déjà en cours, EF Core crée un point de sauvegarde dans cette transaction.** **==Si l'appel échoue, la transaction est annulée jusqu'au point de sauvegarde et non jusqu'au début de la transaction.==** **Les points de sauvegarde peuvent également être gérés par programmation en appelant les méthodes `CreateSavePoint()` et `RollbackToSavepoint()` sur la transaction, comme ceci :**

```cs
static void UsingSavePoints()
{
    // Une fabrique n'est pas censée être utilisée comme ça,
    // mais c'est un code de démonstration :-)
    var context = new ApplicationDbContextFactory().CreateDbContext(null);

    using var trans = context.Database.BeginTransaction();
    try
    {
        //Create, change, delete stuff
        trans.CreateSavepoint("check point 1");
        context.SaveChanges();
        trans.Commit();
    }
    catch
    {
        trans.RollbackToSavepoint("check point 1");
    }
}
```

#### Transactions explicites et stratégies d'exécution

**Lorsqu'une stratégie d'exécution est active** (voir [[Chapitre 22#Résilience de la connection]]), **avant de créer une transaction explicite, vous devez obtenir une référence à la stratégie d'exécution en cours d'utilisation.** ==Appelez ensuite la méthode `Execute()` sur cette stratégie pour créer une transaction explicite.==

```cs
static void TransactionWithExecutionStrategies()
{
    // Une fabrique n'est pas censée être utilisée comme ça,
    // mais c'est un code de démonstration :-)
    var context = new ApplicationDbContextFactory().CreateDbContext(null);

    var strategy = context.Database.CreateExecutionStrategy();
    strategy.Execute(() =>
    {
        using var trans = context.Database.BeginTransaction();
        try
        {
            //actionÀExécuter();
            trans.Commit();
            Console.WriteLine("Insert succeeded");
        }
        catch (Exception ex)
        {
            trans.Rollback();
            Console.WriteLine($"Insert failed: {ex.Message}");
        }
    });
}
```

### Événements de sauvegarde/modifications enregistrées

**EF Core 5 a introduit trois nouveaux événements déclenchés par les méthodes `SaveChanges()`/`SaveChangesAsync()`.** 

- L'événement `SavingChanges` se déclenche lors de l'appel à `SaveChanges()`, avant l'exécution des instructions SQL sur la base de données. 
- L'événement `SavedChanges` se déclenche une fois l'exécution de `SaveChanges()` terminée. 
- L'événement `SaveChangesFailed` se déclenche si l'appel à `SaveChanges()` a échoué. 

Les exemples de code suivants (triviaux) **dans le constructeur de la classe `ApplicationDbContext`** illustrent ces événements et leurs gestionnaires :

```cs
public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
        SavingChanges += (sender, args) =>
        {
            Console.WriteLine(
                $"Saving changes for {((DbContext)sender).Database.GetConnectionString()}"
            );
        };

        SavedChanges += (sender, args) =>
        {
            Console.WriteLine($"Saved {args.EntitiesSavedCount} entities");
        };

        SaveChangesFailed += (sender, args) =>
        {
            Console.WriteLine(
                $"An exception occured! {args.Exception.Message} entities"
            );
        };
    }
```

## La classe `DbSet<T>`

**Pour chaque type d'entité (`T`) de votre modèle objet, vous ajoutez une propriété de type `DbSet<T>` à la classe dérivée `DbContext`.** ***==La classe `DbSet<T>` est une propriété de collection spécialisée utilisée pour interagir avec le fournisseur de base de données afin de lire, ajouter, mettre à jour ou supprimer des enregistrements dans la base de données.==*** **Chaque `DbSet<T>` fournit un certain nombre de services essentiels pour les interactions avec la base de données, notamment la traduction des requêtes LINQ exécutées sur une propriété `DbSet<T>` en requêtes de base de données par le fournisseur de base de données.** Le [[#Tableau 21-2 Membres commun et méthodes d'extensions de `DbSet<T>`|Tableau 21-2]] décrit certains des membres principaux de la classe `DbSet<T>`.

###### Tableau 21-2: Membres commun et méthodes d'extensions de `DbSet<T>`

| Member of DbSet<T>                        | Meaning in Life                                                                                                                                                                                                                                                              |
| ----------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Add()`/`AddRange()`                      | Commence le suivi de/des entités avec l'état `Added`. Les éléments seront ajoutés lors de l'appel à `SaveChanges()`. Des versions asynchrones sont également disponibles.                                                                                                    |
| `AsAsyncEnumerable()`                     | Renvoie la collection sous forme d'`IAsyncEnumerable<T>`.                                                                                                                                                                                                                    |
| `AsQueryable()`                           | Renvoie une instance de collection sous forme de `IQueryable<T>` du `DbSet<T>`.                                                                                                                                                                                              |
| `Find()`                                  | Recherche l'entité dans le `ChangeTracker` par clé primaire. Si elle n'est pas trouvée, la base de données est interrogée. Une version asynchrone est également disponible.                                                                                                  |
| `Update()`/`UpdateRange()`                | Commence le suivi de/des entités à l'état `Modified`. Les éléments seront mis à jour lors de l'appel à `SaveChanges`. Des versions asynchrones sont également disponibles.                                                                                                   |
| `Remove()`/`RemoveRange()`                | Commence le suivi de/des entités à l'état `Deleted`. Les éléments seront supprimés lors de l'appel à `SaveChanges()`. Des versions asynchrones sont également disponibles.                                                                                                   |
| `Attach()`/`AttachRange()`                | Démarre le suivi de/des entités. Les entités dont la clé primaire numérique est définie comme identité et dont la valeur est égale à zéro sont suivies comme `Added`. Toutes les autres sont suivies comme `Unchanged`. Des versions asynchrones sont également disponibles. |
| `FromSqlRaw()`<br>`FromSqlInterpolated()` | Crée une requête LINQ à partir d'une chaîne brute ou interpolée représentant une requête SQL. Peut être combinée avec d'autres instructions LINQ pour une exécution côté serveur.                                                                                            |

**Le type `DbSet<T>` implémente `IQueryable<T>`, ce qui permet d'utiliser des requêtes LINQ pour récupérer des enregistrements de la base de données.** ==Outre les méthodes d'extension ajoutées par EF Core, `DbSet<T>` prend en charge les mêmes méthodes d'extension que celles présentées au [[Chapitre 13#Méthodes d'extension|Chapitre 13]], telles que `ForEach()`, `Select()` et `All()`.==

**Vous ajouterez les propriétés de `DbSet<T>` à `ApplicationDbContext` dans la section [[#Entités]].**

>[!note]
>
De nombreuses méthodes listées dans le [[#Tableau 21-2 Membres commun et méthodes d'extensions de `DbSet<T>`|Tableau 21-2]] portent le même nom que celles du [[#Tableau 21-1 Membre communs de `DbContext`|Tableau 21-1]]. La principale différence réside dans le fait que les méthodes de `DbSet<T>` connaissent déjà le type sur lequel opérer et disposent de la liste des entités. **Les méthodes de `DbContext` doivent déterminer l'élément à traiter par réflexion. Il est beaucoup plus courant d'utiliser les méthodes sur les propriétés de `DbSet<T>` plutôt que les méthodes plus générales sur le `DbContext` dérivé.**

## Le `ChangeTracker`

**L'instance `ChangeTracker` suit l'état des objets chargés dans un `DbSet<T>` au sein d'une instance `DbContext`**. Le [[#Tableau 21-3 Valeurs de l'énumération `EntityState`|Tableau 21-3]] répertorie les valeurs possibles de l'état d'un objet.

###### Tableau 21-3: Valeurs de l'énumération `EntityState`

| Valeur      | Description                                                                      |
| ----------- | -------------------------------------------------------------------------------- |
| `Added`     | L'entité est en cours de suivi mais n'existe pas encore dans la base de données. |
| `Deleted`   | L'entité est suivie et marquée pour suppression de la base de données.           |
| `Detached`  | L'entité n'est pas suivie par le système de suivi des modifications.             |
| `Modified`  | L'entité est en cours de suivi et a été modifiée.                                |
| `Unchanged` | L'entité est suivie, existe dans la base de données et n'a pas été modifiée.     |

Pour vérifier l'état d'un objet, utilisez le code suivant :

```cs
EntityState state = context.Entry(entity).State;
```

Vous pouvez également modifier l'état d'un objet par programmation en utilisant le même mécanisme. Pour modifier l'état en `Deleted` (par exemple), utilisez le code suivant :

```cs
context.Entry(entity).State = EntityState.Deleted;
```

### Événements `ChangeTracker`

**`ChangeTracker` peut déclencher ~~deux~~ événements : `StateChanged` et `Tracked`.** ***==L’événement `StateChanged` se déclenche lorsqu’un changement d’état est survenu. Il ne se déclenche pas lors du premier suivi d’une entité. L’événement `Tracked` se déclenche lorsqu’une entité commence à être suivie, soit par ajout programmatique à une instance de `DbSet<T>`, soit lors du retour d’une requête.==***

>[!important] Note moderne sur les événement de `ChangeTracker` (Avec Gemini)
>
>Si `StateChanged` et `Tracked` restent historiquement les deux événements les plus célèbres et les plus utilisés (notamment pour injecter automatiquement des dates de création ou de modification sur les entités), **le `ChangeTracker` d'EF Core expose désormais pas moins de 8 événements distincts**.
>
>### Leurs équivalents "Avant Action" (Introduits pour plus de contrôle)
>
>- **`StateChanging`** : Identique à `StateChanged`, mais se déclenche **juste avant** que le changement d'état ne soit validé en mémoire. Cela permet d'annuler ou de modifier une valeur au tout dernier moment.
>- `Tracking`: Identique à `Tracked`, mais se déclenche **juste avant** q'une entité soit prise en charge par le contexte. Cela permet d'intercepter l'objet, inspecter ses valeurs initiales brutes, ou modifier sa trajectoire avant qu'EF Core ne commence officiellement à le surveiller.
>
> ### Les événements liés au moteur de détection (`DetectChanges`)
> 
>EF Core passe son temps à comparer l'état actuel de vos objets C# avec le cliché (snapshot) d'origine pris lors du chargement de la base. Pour analyser ce comportement, de nouveaux événements de plomberie ont été ajoutés : 
>
>- **`DetectingAllChanges`** : Émis juste avant que le moteur n'inspecte l'ensemble de votre graphe d'objets.
>- **`DetectedEntityChanges`** : Émis dès qu'une modification concrète vient d'être repérée sur une entité spécifique.
>- **`DetectingEntityChanges`** : Émis juste avant d'analyser une entité en particulier.
>- **`DetectedAllChanges`** : Émis une fois que l'analyse complète de toutes les entités du contexte est terminée. 

Mettez à jour le constructeur de la classe `ApplicationDbContext` comme suit pour spécifier les gestionnaires d’événements `StateChanged` et `Tracked` :

```cs
public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
	: base(options)
{
	...
	
	ChangeTracker.StateChanged += ChangeTracker_StateChanged;
	ChangeTracker.Tracked += ChangeTracker_Tracked;
}
```

#### L'événement `StateChanged`

Comme indiqué, **l'événement `StateChanged` se déclenche lorsque l'état d'une entité change, mais pas lors du premier suivi de l'entité.** ***==Les valeurs `OldState` et `NewState` sont accessibles via `EntityStateChangedEventArgs`.==*** L'exemple suivant écrit dans la console à chaque mise à jour d'une entité :

```cs
private void ChangeTracker_StateChanged(
	object sender,
	EntityStateChangedEventArgs e
)
{
	if (
		e.OldState == EntityState.Modified
		&& e.NewState == EntityState.Unchanged
	)
		Console.WriteLine(
			$"An entity of type {e.Entry.GetType().Name} was updated."
		);
}
```

#### L'événement `Tracked`

**L'événement `Tracked` se déclenche lorsque `ChangeTracker` commence à suivre une entité.** ***==La propriété `FromQuery` de `EntityTrackedEventArgs` indique si l'entité a été chargée via une requête de base de données ou par programmation.==*** L'exemple suivant écrit dans la console chaque fois qu'une entité est chargée depuis la base de données :

```cs
private void ChangeTracker_Tracked(object sender, EntityTrackedEventArgs e)
{
	if (e.FromQuery)
		Console.WriteLine(
			$"An entity of type {e.Entry.Entity.GetType().Name} was loaded from the database."
		);
}
```

### Réinitialisation de l'état du `DbContext`

**EF Core 5 a introduit la possibilité de réinitialiser un `DbContext` dérivé à son état d'origine.** ***==La méthode `ChangeTracker.Clear()` supprime toutes les entités des collections `DbSet<T> `en définissant leur état sur `Detached`==***. **==Le principal avantage est l'amélioration des performances.==** Comme pour tout ORM, **l'instanciation d'une classe `DbContext` dérivée engendre une certaine surcharge. Bien que cette surcharge soit généralement négligeable, la possibilité de nettoyer un contexte déjà instancié peut contribuer à améliorer les performances dans certains cas.**

## Entités

**Les classes fortement typées qui correspondent aux tables de la base de données sont officiellement appelées** *entités*. ***==L'ensemble des entités d'une application constitue un modèle conceptuel d'une base de données physique.==*** **Formellement, ce modèle est appelé** *modèle de données d'entités* **(EDM), généralement désigné simplement comme le** *modèle*. ***==Le modèle est mappé au domaine applicatif/métier.==*** Les ***==entités et leurs propriétés sont mappées aux tables et aux colonnes à l'aide des conventions, de la configuration et de l'API Fluent (code) d'Entity Framework Core.==*** ==Les entités n'ont pas besoin d'être mappées directement au schéma de la base de données.== **Vous êtes libre de structurer vos classes d'entités pour répondre aux besoins de votre application, puis de mapper vos entités uniques à votre schéma de base de données.**

**Ce faible couplage entre la base de données et vos entités signifie que vous pouvez façonner les entités pour qu'elles correspondent à votre domaine métier, indépendamment de la conception et de la structure de la base de données.** ==Par exemple, prenons la simple table `Inventory` de la base de données `AutoLot` et la classe d'entité `Car` du chapitre précédent.== **Les noms sont différents, pourtant l'entité `Car` peut être mappée à la table `Inventory`.** ***==EF Core examine la configuration de vos entités dans le modèle afin de faire correspondre la représentation côté client de la table `Inventory` (dans notre exemple, la classe `Car`) aux colonnes correspondantes de la table `Inventory`==***.

==Les sections suivantes détaillent comment les conventions EF Core, les annotations de données et le code (utilisant l'API Fluent) permettent de faire correspondre les entités, les propriétés et les relations entre les entités du modèle aux tables, colonnes et relations de clés étrangères de votre base de données. Chacun de ces sujets est traité en détail plus loin dans ce chapitre.==

>[!Tip] Rappel :
>Un schéma de base de données est l'équivalent d'un espace de noms en C#.

### Propriétés d'entité et colonnes de base de données

**Lorsqu'une base de données relationnelle est utilisée, EF Core exploite les données des colonnes d'une table pour renseigner les propriétés d'une entité lors de la lecture des données et écrit les propriétés de l'entité dans les colonnes d'une table lors de la persistance des données.** **==Si la propriété est automatique, EF Core la lit et l'écrit via ses accesseurs (getter et setter)==**. ***==Si la propriété possède un champ de stockage, EF Core lira et écrira dans ce champ plutôt que dans la propriété publique, même si le champ de stockage est privé.==*** **Bien qu'EF Core puisse lire et écrire dans des champs privés, une propriété publique en lecture-écriture encapsulant le champ de stockage est toujours nécessaire.**

**La prise en charge des champs de stockage est avantageuse dans deux cas :** ***==l'utilisation du modèle `INotifyPropertyChanged` dans les applications WPF/Avalonia et les conflits entre les valeurs par défaut de la base de données et celles de .NET.==*** **L'utilisation d'EF Core avec WPF/Avalonia est abordée au [[Chapitre 28]], et les valeurs par défaut de la base de données sont traitées plus loin dans ce chapitre.**

### Schémas de mappage de tables

**EF Core propose deux schémas de mappage classe-table :** *table par hiérarchie* **(TPH) et** *table par type* **(TPT)**. ***==Le mappage TPH, par défaut, associe une hiérarchie d’héritage à une seule table. Introduit dans EF Core 5, le mappage TPT associe chaque classe de la hiérarchie à sa propre table.==***

>[!note]
>Les classes peuvent également être associées à des vues et à des requêtes SQL brutes. On les appelle des *types de requêtes* et elles seront abordées plus loin dans ce chapitre.

#### Mappage TPH (Table-Par-Hiérarchie)

Prenons l'exemple suivant, qui illustre la classe `Car` du [[Chapitre 20#Créez les classes `Car` et `CarViewModel`.|Chapitre 20]] divisée en deux classes : une classe de base (`BaseEntity`) contenant les propriétés `Id` et `TimeStamp`, et une autre classe contenant toutes les autres propriétés de la classe `Car`.

>[!tip] Les colonnes `Id` et `TimeStamp` sont toutes les deux présentes dans toutes les table de la base de donnée `AutoLot`, donc une entité de base / table (`BaseEntity`) possèdera forcément ces deux données 

Le code correspondant aux exemples de mappage TPH se se trouve dans le projet *AutoLot.TPH*, dans le code source de ce chapitre.

```cs
//BaseEntity.cs
namespace AutoLot.TPH.Models;

public abstract class BaseEntity
{
	public int Id { get; set; }
	public DateTime TimeStamp { get; set; }
}
```

```cs
//Car.cs
namespace AutoLot.TPH.Models;

public class Car : BaseEntity
{
	public string Color { get; set; }
	public string PetName { get; set; }
	public int MakeId { get; set; }
}
```

Pour qu'EF Core reconnaisse qu'une classe d'entité fait partie du modèle objet, ajoutez une propriété `DbSet<T>` à l'entité. Créez une classe `ApplicationDbContext` et mettez-la à jour comme suit :

```cs
using AutoLot.TPH.Models;
using Microsoft.EntityFrameworkCore;

namespace AutoLot.TPH;

public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options) { }

    public DbSet<Car> Cars { get; set; }
}
```

==Notez la propriété `DbSet<T>` dans la classe `ApplicationDbContext`==. **Celle-ci indique à EF Core que la classe `Car` correspond à la table `Cars` dans la base de données** (plus d'informations à ce sujet dans la section [[#Conventions d'entités]]). ***==Notez également l'absence de propriété `DbSet<T>` pour la classe `BaseEntity`. En effet, dans le schéma TPH, toute la hiérarchie est réduite à une seule table==***. **Les propriétés des tables situées en amont de la chaîne d'héritage sont intégrées à la table possédant la propriété `DbSet<T>`.** Ceci est illustré par la requête SQL suivante :

```sql
CREATE TABLE "Cars" (
    "Id" SERIAL NOT NULL,
    "Color" TEXT NULL,
    "PetName" TEXT NULL,
    "MakeId" INTEGER NOT NULL,
    "TimeStamp" BYTEA NULL,
    "Discriminator" VARCHAR(255) NOT NULL,
    CONSTRAINT "PK_Cars" PRIMARY KEY ("Id")
);
```

#### Mappage TPT

**Pour explorer le schéma d'association TPT, les classes `BaseEntity` et `Car` peuvent être utilisées, même si la classe de base est marquée comme abstraite**. ***==Puisque TPH est le comportement par défaut, EF Core doit être configuré pour associer chaque classe à une table.==*** ==Ceci peut être réalisé à l'aide d'annotations de données (présentées plus loin dans ce chapitre) ou de l'API Fluent.== **Pour utiliser le schéma d'association TPT, utilisez le code suivant de l'API Fluent dans la méthode `OnModelCreating()` de l'`ApplicationDbContext`.** Ces exemples se trouvent dans le projet *AutoLot.TPT*, dans les exemples de code de ce chapitre.

```cs

```

EF Core créera deux tables, comme illustré ici. **Les index montrent également que les tables ont une correspondance un-à-un entre les tables `BaseEntities` et `Cars`.**

```sql
CREATE TABLE "BaseEntities" (
    "Id" SERIAL NOT NULL,
    "TimeStamp" BYTEA NULL,
    CONSTRAINT "PK_BaseEntities" PRIMARY KEY ("Id")
);

CREATE TABLE "Cars" (
    "Id" INTEGER NOT NULL,
    "Color" TEXT NULL,
    "PetName" TEXT NULL,
    "MakeId" INTEGER NOT NULL,
    "Discriminator" VARCHAR(255) NOT NULL,
    CONSTRAINT "PK_Cars" PRIMARY KEY ("Id"),
    CONSTRAINT "FK_Cars_BaseEntities_Id" FOREIGN KEY ("Id")
        REFERENCES "BaseEntities" ("Id")
        ON DELETE CASCADE
);
```

>[!note]
>Le mappage « Table-Par-Type » peut avoir des conséquences importantes sur les performances et doit être pris en compte avant d’utiliser ce schéma de mappage. Pour plus d’informations, [consultez la documentation](https://docs.microsoft.com/en-us/ef/core/performance/modeling-for-performance#inheritance-mapping.).

### Propriétés de navigation et clés étrangères

**Les** *propriétés de navigation* **représentent la manière dont les classes d'entités sont liées entre elles et permettent au code de naviguer d'une instance d'entité à une autre.** ***==Par définition, une propriété de navigation est toute propriété qui correspond à un type non scalaire, tel que défini par le fournisseur de base de données.==*** **En pratique, les propriétés de navigation correspondent à une autre entité** (appelées *propriétés de navigation de référence*) **ou à une collection d'une autre entité** (appelées *propriétés de navigation de collection*). **==Côté base de données, les propriétés de navigation sont traduites en relations de clés étrangères entre les tables.==** ***==Les relations un-à-un, un-à-plusieurs et (nouveauté d'EF Core 5) plusieurs-à-plusieurs sont directement prises en charge par EF Core.==*** ==Les classes d'entités peuvent également avoir des propriétés de navigation qui pointent vers elles-mêmes, représentant ainsi des tables autoréférentielles.==

>[!note]
>Je trouve utile de considérer les objets dotés de propriétés de navigation comme des listes chaînées, et si les propriétés de navigation sont bidirectionnelles, les objets se comportent comme des listes doublement chaînées.

Avant d'aborder en détail les propriétés de navigation et les modèles de relations entre entités, reportez-vous au [[#Tableau 21-4 Termes utilisés pour décrire les propriétés et les relations de navigation|Tableau 21-4]]. **Ces termes sont utilisés dans les trois modèles de relations.**

###### Tableau 21-4: Termes utilisés pour décrire les propriétés et les relations de navigation

| Terme                | Description                                                                                                                                                                                                    |
| -------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Entité principale    | Le parent de la relation.                                                                                                                                                                                      |
| Entité dépendante    | L'enfant de cette relation.                                                                                                                                                                                    |
| Clé principale       | La ou les propriétés utilisées pour définir l'entité principale. Il peut s'agir de la clé primaire ou d'une clé alternative. Les clés peuvent être configurées à l'aide d'une seule propriété ou de plusieurs. |
| Clé étrangère        | La ou les propriétés utilisées par l'entité enfant pour stocker la clé principale.                                                                                                                             |
| Relation requise     | Relation où la valeur de la clé étrangère est requise (non nulle).                                                                                                                                             |
| Relation facultative | Relation où la valeur de la clé étrangère n'est pas (nullable).                                                                                                                                                |

#### Propriétés de clé étrangère manquantes

**Si une entité possédant une propriété de navigation de référence ne dispose pas d'une propriété pour la valeur de la clé étrangère, EF Core créera la ou les propriétés nécessaires sur l'entité.** **==Il s'agit de *propriétés de clé étrangère fantômes*, nommées selon le format `<nom de la propriété de navigation><nom de la propriété de clé principale>` ou `<nom de l'entité principale><nom de la propriété de clé principale>`==**. **Ceci est valable pour tous les types de relations (un-à-plusieurs, un-à-un, plusieurs-à-plusieurs).** ***==Il est beaucoup plus propre de construire vos entités avec des propriétés de clé étrangère explicites plutôt que de laisser EF Core les créer.==***

#### Relations un-à-plusieurs

**Pour créer une relation un-à-plusieurs, la classe d'entité du côté principal ajoute une propriété de type collection à la classe d'entité du côté dépendant.** ==L'entité dépendante doit également posséder des propriétés pour la clé étrangère pointant vers le principal.== **==Dans le cas contraire, EF Core créera des propriétés de clé étrangère fantômes, comme expliqué précédemment.==** Par exemple, dans la base de données créée au [[Chapitre 20#Création de la table `Makes`|Chapitre 20]], la table `Makes` (représentée par la classe d'entité `Make`) et la table `Inventory` (représentée par la classe d'entité `Car`) ont une relation un-à-plusieurs.

>[!note]
>Dans ces premiers exemples, la classe `Car` sera associée à une table nommée `Cars`. **Plus loin dans ce chapitre, la classe `Car` sera associée à la table `Inventory`.**

 **De retours dans le projet** *AutoLot.Samples*, créez un nouveau dossier nommé `Models`. Ajoutez les fichiers *BaseEntity.cs*, *Make.cs* et *Car.cs* suivants, comme indiqué dans l'extrait de code. 

```cs
// Models/BaseEntity.cs
namespace AutoLot.Samples.Models;

public abstract class BaseEntity
{
    public int Id { get; set; }
    public DateTime TimeStamp { get; set; }
}
```

```cs
namespace AutoLot.Samples.Models;

public class Make : BaseEntity
{
    public string Name { get; set; }
    public IEnumerable<Car> Cars { get; set; } = new List<Car>();
}
```

```cs
// Models/Car.cs
namespace AutoLot.Samples.Models;

public class Car : BaseEntity
{
    public string Color { get; set; }
    public string PetName { get; set; }
    public int MakeId { get; set; }
    public Make MakeNavigation { get; set; }
}
```

>[!note] Les lignes suivantes représentent la navigation bidirectionnelle de la relation un-à-plusieurs :
>```cs
>// Make.cs
>public IEnumerable<Car> Cars { get; set; } = new List<Car>();
>// Car.cs
>public int MakeId { get; set; }
>public Make MakeNavigation { get; set; }
>```

>[!note]
>Lors de la génération d'une base de données existante (comme vous le ferez dans le chapitre suivant), EF Core nomme les propriétés de navigation de la même manière que leur type (par exemple, `public Make Make {get; set;}`). Cela peut entraîner des problèmes de navigation et d'IntelliSense, et rendre le code difficile à manipuler. Je préfère ajouter le suffixe `Navigation` pour référencer les propriétés de navigation, par souci de clarté, comme illustré dans l'exemple précédent.
>>[!tip] Ce qui a changé depuis .NET 6 (Avec Claude)
>>Le scaffolding peut maintenant être personnalisé via un fichier de configuration `T4` template, ce qui te permet de contrôler le nommage des propriétés générées sans les renommer manuellement après chaque scaffolding. C'est une amélioration notable par rapport à l'époque du livre.

**Dans l'exemple `Car`/`Make`, l'entité `Car` est l'entité dépendante** (le *plusieurs* de la relation un-à-plusieurs), **et l'entité `Make` est l'entité principale** (le *un* de la relation un-à-plusieurs).

Mettez à jour le fichier *GlobalUsings.cs* pour inclure le nouvel espace de noms des modèles :

```cs
global using AutoLot.Samples.Models;
```

Ensuite, ajoutez les propriétés `DbSet<Car>` et `DbSet<Make>` à `ApplicationDbContext`, comme indiqué ici :

```cs
public DbSet<Car> Cars { get; set; }
public DbSet<Make> Makes { get; set; }
```

Lors de la mise à jour de la base de données à l'aide des migrations EF Core, les tables suivantes sont créées :

```sql
CREATE TABLE "Makes" (
    "Id" SERIAL NOT NULL,
    "TimeStamp" TIMESTAMP WITH TIME ZONE NOT NULL,
    "Name" TEXT NULL,
    CONSTRAINT "PK_Makes" PRIMARY KEY ("Id")
);

CREATE TABLE "Cars" (
    "Id" SERIAL NOT NULL,
    "TimeStamp" TIMESTAMP WITH TIME ZONE NOT NULL,
    "Color" TEXT NULL,
    "PetName" TEXT NULL,
    "MakeId" INTEGER NOT NULL,
    CONSTRAINT "PK_Cars" PRIMARY KEY ("Id"),
    CONSTRAINT "FK_Cars_Makes_MakeId" FOREIGN KEY ("MakeId")
        REFERENCES "Makes" ("Id")
        ON DELETE CASCADE
);

CREATE INDEX "IX_Cars_MakeId" ON "Cars" ("MakeId");
```

Notez la clé étrangère et les contraintes de vérification créées sur la table dépendante `Car`

>[!note]
La mise à jour de la base de données à l'aide des migrations EF Core est abordée plus loin dans ce chapitre. Si vous êtes déjà familiarisé avec les migrations EF Core ou si vous souhaitez passer directement à la section sur les commandes CLI EF Core pour en savoir plus sur les migrations avant de poursuivre, voici les commandes spécifiques permettant de créer ces tables : 
>```bash
>dotnet ef migrations add Initial -o Migrations -c AutoLot.Samples.ApplicationDbContext 
>
>dotnet ef database update Initial -c AutoLot.Samples.ApplicationDbContext
>```

#### Relations un-à-un

**Dans les relations un-à-un, chaque entité possède une propriété de navigation de référence vers l'autre.** ***==Alors que les relations un-à-plusieurs désignent clairement l'entité principale et l'entité dépendante, lors de la création de relations un-à-un, EF Core doit être informé de l'entité principale.==*** **Ceci peut se faire soit en définissant clairement une clé étrangère vers l'entité principale, soit en l'indiquant via l'API Fluent.** ==Si EF Core n'est pas informé par l'une de ces deux méthodes, il en choisira une en fonction de sa capacité à détecter une clé étrangère.== **==En pratique, il est recommandé de définir clairement l'entité dépendante en ajoutant des propriétés de clé étrangère. Cela élimine toute ambiguïté et garantit la bonne configuration de vos tables.==**

Ajoutez une nouvelle classe nommée *Radio.cs* et mettez à jour le code comme suit :

```cs
namespace AutoLot.Samples.Models;

public class Radio : BaseEntity
{
    public bool HasTweeters { get; set; }
    public bool HasSubWoofers { get; set; }
    public string RadioId { get; set; }
    public int CarId { get; set; }
    public Car CarNavigation { get; set; }
}
```

Ajoutez la propriété de navigation `Radio` à la classe `Car` :

```cs
namespace AutoLot.Samples.Models;

public class Car : BaseEntity
{
	...
	
    public Radio RadioNavigation { get; set; }
}
```

**Puisque `Radio` possède une clé étrangère vers la classe `Car` (conformément à la convention, expliquée plus loin), `Radio` est l'entité dépendante et `Car` l'entité principale.** ***==EF Core crée implicitement l'index unique requis sur la propriété de clé étrangère de l'entité dépendante. Si vous souhaitez modifier le nom de l'index, vous pouvez le faire à l'aide d'annotations de données ou de l'API Fluent.==***

Ajoutez la propriété `DbSet<Radio>` à la classe `ApplicationDbContext` :

```cs
public class ApplicationDbContex : DbContext
{
	...
	
	public DbSet<Radio> Radios { get; set; }
	
	...
}
```

Lorsque la base de données est mise à jour à l'aide des migrations EF Core suivantes, la table `Cars` reste inchangée et la table `Radios` suivante est créée :

```bash
dotnet ef migrations add Radio -o Migrations -c AutoLot.Samples.ApplicationDbContext

dotnet ef database update Radio -c AutoLot.Samples.ApplicationDbContext
```

Le tableau des radios ajoutées est présenté ci-dessous :

```sql
CREATE TABLE "Radios" (
    "Id" SERIAL NOT NULL,
    "HasTweeters" BOOLEAN NOT NULL,
    "HasSubWoofers" BOOLEAN NOT NULL,
    "RadioId" TEXT NULL,
    "TimeStamp" TIMESTAMP WITH TIME ZONE NOT NULL,
    "CarId" INTEGER NOT NULL,
    CONSTRAINT "PK_Radios" PRIMARY KEY ("Id"),
    CONSTRAINT "FK_Radios_Cars_CarId" FOREIGN KEY ("CarId")
        REFERENCES "Cars" ("Id")
        ON DELETE CASCADE
);

CREATE UNIQUE INDEX "IX_Radios_CarId" ON "Radios" ("CarId");
```

Notez la clé étrangère et les contraintes de vérification créées sur la table dépendante (`Radios`).

#### Relations plusieurs-à-plusieurs

**Dans les relations plusieurs-à-plusieurs, chaque entité possède une propriété de navigation vers l'autre.** ***==Ceci est implémenté dans la base de données à l'aide d'une table de jointure entre les deux tables d'entités.==*** **==Par défaut, cette table de jointure, est nommée d'après les deux tables sous la forme `<Entity1Entity2>`, mais peut être modifiée par programmation via l'API Fluent.==** ***L'entité de jointure possède des relations un-à-plusieurs avec chacune des tables d'entités***

>[!important] Ce n'est plus tout à fait exact depuis EF Core 5 :
>
>* Si `Driver` contient `public List<Car> Cars { get; set; }`
>* Si `Car` contient `public List<Driver> Drivers { get; set; }`
>
>Le nom par défaut de la table de jointure sera `CarDriver` ou `DriverCar`
>selon l'ordre de découverte des entités par EF Core, pas nécessairement
>l'ordre alphabétique. Cet ordre dépend de l'ordre de déclaration des
>DbSet dans le DbContext.
>
>Pour éviter toute surprise, il est toujours recommandé de nommer
>explicitement la table de jointure via Fluent API :
>
>```cs
>modelBuilder.Entity<Car>()
>	.HasMany(c => c.Drivers)
>	.WithMany(d => d.Cars)
>	.UsingEntity(j => j.ToTable("CarDrivers"));
>```

>[!tip] Rappel SQL
>1. On crée une table intermédiaire (avec le nom qui mixent les deux tables principales)
>2. Relation un-à-plusieurs avec la première table
>3. Relation un-à-plusieurs avec la seconde table
>
>**Le résultat final est une relation plusieurs-à-plusieurs.**

Commencez par créer une classe `Driver`, qui aura une relation plusieurs-à-plusieurs avec la classe `Car`. Côté `Driver`, cela est implémenté à l'aide d'une propriété de navigation vers la classe `Car`.

```cs
namespace AutoLot.Samples.Models;

public class Driver : BaseEntity
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public IEnumerable<Car> Cars { get; set; } = new List<Car>();
}
```

Ajoutez la propriété `DbSet<Driver>` à la classe `ApplicationDbContext` :

```cs
public class ApplicationDbContext : DbContext
{
    public DbSet<Car> Cars { get; set; }
    public DbSet<Make> Makes { get; set; }
    public DbSet<Radio> Radios { get; set; }
    public DbSet<Driver> Drivers { get; set; }

	...
}
```

Ensuite, mettez à jour la classe `Car` pour qu'elle possède une propriété de navigation de collection pointant vers la nouvelle classe `Driver` :

```cs
namespace AutoLot.Samples.Models;

public class Car : BaseEntity
{
    public string Color { get; set; }
    public string PetName { get; set; }
    public int MakeId { get; set; }
    public Make MakeNavigation { get; set; }
    public Radio RadioNavigation { get; set; }
    public IEnumerable<Driver> Drivers { get; set; } = new List<Driver>();
}
```

Pour mettre à jour la base de données, utilisez les commandes de migration suivantes (***==les migrations seront expliquées en détail plus loin dans ce chapitre==***) :

```bash
dotnet ef migrations add Drivers -o Migrations -c AutoLot.Samples.ApplicationDbContext

dotnet ef database update Drivers -c AutoLot.Samples.ApplicationDbContext
```

Lors de la mise à jour de la base de données, la table `Cars` reste inchangée, et les tables `Drivers` et `CarDriver` sont créées, Voici les définitions des nouvelles tables :

```sql
CREATE TABLE "Drivers" (
    "Id" SERIAL NOT NULL,
    "FirstName" TEXT NULL,
    "LastName" TEXT NULL,
    "TimeStamp" TIMESTAMP WITH TIME ZONE NOT NULL,
    CONSTRAINT "PK_Drivers" PRIMARY KEY ("Id")
);

CREATE TABLE "CarDriver" (
    "CarsId" INTEGER NOT NULL,
    "DriversId" INTEGER NOT NULL,
    CONSTRAINT "PK_CarDriver" PRIMARY KEY ("CarsId", "DriversId"),
    CONSTRAINT "FK_CarDriver_Cars_CarsId" FOREIGN KEY ("CarsId")
        REFERENCES "Cars" ("Id")
        ON DELETE CASCADE,
    CONSTRAINT "FK_CarDriver_Drivers_DriversId" FOREIGN KEY ("DriversId")
        REFERENCES "Drivers" ("Id")
        ON DELETE CASCADE
);

CREATE INDEX "IX_CarDriver_DriversId" ON "CarDriver" ("DriversId");
```

Notez que la clé primaire composée, les contraintes de vérification (clés étrangères) et le comportement en cascade sont tous créés par EF Core pour garantir que la table `CarDriver` est configurée comme une table de jointure appropriée.

##### Relation plusieurs-à-plusieurs avant EF Core 5

La relation plusieurs-à-plusieurs équivalente entre une voiture et un conducteur peut être implémentée en créant explicitement les trois tables. C'est la méthode à suivre dans les versions d'EF Core antérieures à EF Core 5. Voici un exemple simplifié :

```cs
public class Driver
{
	...
	public IEnumerable<CarDriver> CarDrivers { get; set; }
}

public class Car
{
	...
	public IEnumerable<CarDriver> CarDrivers { get; set; }
}

public class CarDriver
{
	public int CarId {get;set;}
	public Car CarNavigation {get;set;}
	public int DriverId {get;set;}
	public Driver DriverNavigation {get;set;}
}
```

#### Comportement en cascade

**La plupart des bases de données (comme SQL Server et PostgresSQL) possèdent des règles qui régissent le comportement lors de la suppression d'une ligne.** ***==Si les enregistrements associés (dépendants) doivent également être supprimés, on parle de suppression en cascade.==*** ==Dans EF Core, trois actions peuvent se produire lors de la suppression d'une entité principale (dont les entités dépendantes sont chargées en mémoire) :==

- Les enregistrements dépendants sont supprimés.
- Les clés étrangères dépendantes sont définies sur null.
- L'entité dépendante reste inchangée.

**Le comportement par défaut diffère selon que les relations sont facultatives ou obligatoires.** **==Ce comportement peut être configuré sur sept valeurs, bien que seules cinq soient recommandées.==** **La configuration s'effectue à l'aide de l'énumération `DeleteBehavior` via l'API Fluent.** Les options disponibles sont les suivantes :

- `Cascade`
- `ClientCascade`
- `ClientNoAction` (non recommandé)
- `ClientSetNull`
- `NoAction` (non recommandé)
- `SetNull`
- `Restrict`

Dans EF Core, le comportement spécifié est déclenché uniquement après la suppression d'une entité et l'appel de `SaveChanges()` sur le `DbContext` dérivé. Consultez la section [[#Exécution des requêtes]] pour plus de détails sur les interactions d'EF Core avec la base de données.

##### Relations facultatives

==Rappelons qu'au [[#Tableau 21-4 Termes utilisés pour décrire les propriétés et les relations de navigation|Tableau 21-4]] les relations facultatives sont celles où l'entité dépendante peut définir la ou les valeurs de la clé étrangère sur `null`.== **Pour les relations facultatives, le comportement par défaut est `ClientSetNull`.** Le [[#Tableau 21-5 Comportement de la cascade avec des relations optionnelles|Tableau 21-5]] illustre le comportement en cascade avec les entités dépendantes et son impact sur les enregistrements de la base de données lors de l'utilisation de SQL Server.

###### Tableau 21-5: Comportement de la cascade avec des relations optionnelles

| Comportement de la Suppression  | Effet sur les Dépendances (Dans la Mémoire)          | Effet sur les Dépendances (Dans la Base de Données)                                                              |
| ------------------------------- | ---------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| `Cascade`                       | Les entités sont supprimées.                         | Les entités sont supprimées par la base de données.                                                              |
| `ClientCascade`                 | Les entités sont supprimées.                         | Pour les bases de données qui ne prennent pas en charge la suppression en cascade, EF Core supprime les entités. |
| `ClientSetNull`<br>(par defaut) | Propriété(s) de clé étrangère définie(s) sur `null`. | Aucun.                                                                                                           |
| `SetNull`                       | Propriété(s) de clé étrangère définie(s) sur `null`. | Propriété(s) de clé étrangère définie(s) sur `null`.                                                             |
| `Restrict`                      | Aucun.                                               | Aucun.                                                                                                           |

##### Relations obligatoires

**Les relations obligatoires concernent les entités dépendantes pour lesquelles la ou les valeurs de clé étrangère ne peuvent pas être `null`.** ***==Le comportement par défaut des relations obligatoires est `Cascade`.==*** Le [[#Tableau 21-6 Comportement de la cascade avec des relations requises|Tableau 21-6]] illustre ce comportement en cascade avec les entités dépendantes et son impact sur les enregistrements de la base de données lors de l'utilisation de SQL Server.

###### Tableau 21-6: Comportement de la cascade avec des relations requises

| Comportement de la Suppression | Effet sur les Dépendance (Dans la Mémoire)<br> | Effet sur les Dépendances (Dans la Base de Données)                                                              |
| ------------------------------ | ---------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| `Cascade`<br>(Par defaut)      | Les entités sont supprimées.                   | Les entités sont supprimées.                                                                                     |
| `ClientCascade`                | Les entités sont supprimées.                   | Pour les bases de données qui ne prennent pas en charge la suppression en cascade, EF Core supprime les entités. |
| `ClientSetNull`                | `SaveChanges` lève une exception.              | Aucun.                                                                                                           |
| `SetNull`                      | `SaveChanges` lève une exception.              | `SaveChanges` lève une exception.                                                                                |
| `Restrict`                     | Aucun.                                         | Aucun.                                                                                                           |

>[!tip] Les tableaux [[#Tableau 21-5 Comportement de la cascade avec des relations optionnelles|21-5]] et [[#Tableau 21-6 Comportement de la cascade avec des relations requises|21-6]] sont valides pour PostgreSQL. Les seules nuances sont :
> 
> - `ClientCascade` est pratiquement inutile avec `Npgsql` car PostgreSQL supporte le cascade delete nativement
> - Pour `SetNull` sur une relation required, l'exception peut provenir de PostgreSQL directement (violation `NOT NULL`) plutôt que d'EF Core, selon la configuration du change tracker

### Conventions d'entités

**EF Core utilise de nombreuses conventions pour définir une entité et sa relation avec la base de données.** ***==Ces conventions sont toujours activées, sauf si elles sont modifiées par des annotations de données ou du code dans l'API Fluent.==*** Le [[#Tableau 21-7 Certaines conventions fondamentales d'EF|Tableau 21-7]] répertorie certaines des conventions EF Core les plus importantes.

###### Tableau 21-7: Certaines conventions fondamentales d'EF

| Convention         | Description                                                                                                                                                                                                                                                                                                                                    |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Included tables    | Toutes les classes possédant une propriété `DbSet` et toutes les classes accessibles (via les propriétés de navigation) par une classe `DbSet` sont créées dans la base de données.                                                                                                                                                            |
| Included columns   | Toutes les propriétés publiques dotées d'un accesseur et d'un mutateur (y compris les propriétés automatiques) sont associées à des colonnes.                                                                                                                                                                                                  |
| Table name         | Correspond au nom de la propriété `DbSet` dans le `DbContext` dérivé. Si aucune propriété `DbSet` n'existe, le nom de la classe est utilisé.                                                                                                                                                                                                   |
| Schema             | Les tables sont créées dans le schéma par défaut du magasin de données (`dbo` sur SQL Server, `public` sur PostgreSQL).                                                                                                                                                                                                                        |
| Column name        | Les noms de colonnes sont associés aux noms de propriétés de la classe.                                                                                                                                                                                                                                                                        |
| Column data type   | Les types de données sont sélectionnés en fonction du type de données .NET et traduits par le fournisseur de base de données (Npgsql). Les types `DateTime` sont convertis en `TIMESTAMP WITH TIME ZONE`, et les chaînes de caractères en `TEXT`. Les chaînes de caractères utilisées dans une clé primaire sont converties en `VARCHAR(255)`. |
| Column nullability | Les types de données non nullables sont créés sous forme de colonnes persistantes `Not Null`. EF Core prend en charge la nullité en C# 8.                                                                                                                                                                                                      |
| Primary key        | Les propriétés nommées `Id` ou `<EntityTypeName>Id` seront configurées comme clé primaire. Les clés de type `short`, `int`, `long` ou `Guid` ont des valeurs gérées par la base de données. Les valeurs numériques sont créées en tant que colonnes `Identity` (SQL Server) ou `SERIAL` (PostgreSQL).                                          |
| Relationships      | Les relations entre les tables sont créées lorsqu'il existe des propriétés de navigation entre deux classes d'entités.                                                                                                                                                                                                                         |
| Foreign key        | Les propriétés nommées `<OtherClassName>Id` sont des clés étrangères pour les propriétés de navigation de type `<OtherClassName>`.                                                                                                                                                                                                             |

**Les exemples précédents de propriétés de navigation exploitent tous les conventions d'EF Core pour établir les relations entre les tables.**

###### Tableau 21-8: Tables complète des mappings .NET -> PostgreSQL

| .NET Type  | SQL Server         | PostgreSQL (Npgsql)        |
| ---------- | ------------------ | -------------------------- |
| `string`   | `nvarchar(max)`    | `text`                     |
| `string`   | `nvarchar(450)`    | `varchar(255)`             |
| `DateTime` | `datetime2(7)`     | `timestamp with time zone` |
| `int`      | `int`              | `integer`                  |
| `long`     | `bigint`           | `bigint`                   |
| `bool`     | `bit`              | `boolean`                  |
| `byte[]`   | `varbinary(max)`   | `bytea`                    |
| `decimal`  | `decimal(18,2)`    | `numeric`                  |
| `float`    | `real`             | `real`                     |
| `double`   | `float`            | `double precision`         |
| `Guid`     | `uniqueidentifier` | `uuid`                     |

## Mapper des propriétés à des colonnes

**Par convention, les propriétés publiques en lecture-écriture sont associées à des colonnes portant le même nom.** **==Le type de données correspond à l'équivalent, dans le magasin de données, du type de données CLR de la propriété.==** **==Les propriétés non nullables sont définies comme non nulles dans le magasin de données, et les propriétés nullables (y compris les types référence nullables introduits dans C# 8) sont définies pour autoriser les valeurs nulles.==**

**EF Core prend également en charge la lecture et l'écriture dans les champs sous-jacents des propriétés. EF Core s'attend à ce que le champ sous-jacent soit nommé selon l'une des conventions suivantes (par ordre de priorité) :**

• `_<nom de la propriété en camelCase>`
• `_<nom de la propriété>`
• `m_<nom de la propriété en camelCase>`
• `m_<nom de la propriété>`

Si la propriété `Color` de la classe `Car` est mise à jour pour utiliser un champ de stockage, elle devra (par convention) être nommée `_color`, `_Color`, `m_color` ou `m_Color`, comme ceci :

```cs
private string _color;
public string Color
{
	get => _color;
	set => _color = value;
}
```

#### Surcharger les conventions EF Core

Nouveauté d'EF Core 6 : les conventions peuvent être surchargées à l'aide de la méthode `ConfigureConventions()`. Par exemple, si vous souhaitez que les propriétés de type chaîne aient une taille par défaut (au lieu de `nvarchar(max)`), vous pouvez ajouter
le code suivant à la classe `ApplicationDbContext` :

```cs
protected override void ConfigureConventions(
	ModelConfigurationBuilder configurationBuilder
)
{
	configurationBuilder.Properties<string>().HaveMaxLength(50);
}
```

Lorsqu'une nouvelle migration est créée et exécutée, vous constaterez que toutes les propriétés de type chaîne sont mises à jour à `nvarchar(50)`(SQL Server) ou `VARCHAR(50)` (PostgreSQL).

### Annotations de données Entity Framework

**Les annotations de données sont des attributs C# permettant de mieux définir vos entités.** Le [[#Tableau 21-9 Certaines annotations de données prises en charge par Entity Framework Core (*Nouveaux attributs dans EF Core 6 en italique*)|Tableau 21-9]] répertorie certaines des annotations de données les plus utilisées pour définir la correspondance entre vos classes d'entités et leurs propriétés, et les tables et champs de la base de données. ***==Les annotations de données prévalent sur toute convention conflictuelle.==*** **De nombreuses autres annotations sont disponibles pour affiner les entités du modèle, comme vous le découvrirez dans la suite de ce chapitre et de cet ouvrage.**

###### Tableau 21-9: Certaines annotations de données prises en charge par Entity Framework Core (*Nouveaux attributs dans EF Core 6 en italique*)

| Annotation de donnée        | Descritiption                                                                                                                                                                                               |
| --------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Table`                     | Définit le schéma et le nom de la table pour l'entité.                                                                                                                                                      |
| *`EntityTypeConfiguration`* | Associé à l'interface `IEntityTypeConfiguration`, cet attribut permet de configurer une entité dans sa propre classe via l'API Fluent. Son utilisation est décrite dans la section relative à l'API Fluent. |
| `Keyless`                   | Indique qu'une entité ne possède pas de clé (par exemple, représentant une vue de base de données).                                                                                                         |
| `Column`                    | Définit le nom de la colonne pour la propriété de l'entité.                                                                                                                                                 |
| `BackingField`              | Spécifie le champ de stockage C# d'une propriété.                                                                                                                                                           |
| `Key`                       | Définit la clé primaire de l'entité. Les champs clés sont implicitement également `[Obligatoire]`.                                                                                                          |
| `Index`                     | Utilisée sur une classe pour spécifier un index à une ou plusieurs colonnes. Permet de garantir l'unicité de l'index.                                                                                       |
| `Owned`                     | Déclare que la classe sera détenue par une autre classe d'entité.                                                                                                                                           |
| `Required`                  | Déclare la propriété comme non nullable dans la base de données.                                                                                                                                            |
| `ForeignKey`                | Déclare une propriété utilisée comme clé étrangère pour une propriété de navigation.                                                                                                                        |
| `InverseProperty`           | Déclare la propriété de navigation à l'autre extrémité d'une relation.                                                                                                                                      |
| `StringLength`              | Spécifie la longueur maximale d'une propriété de type chaîne de caractères.                                                                                                                                 |
| `TimeStamp`                 | Déclare un type comme un `rowversion` dans SQL Server et ajoute des contrôles de concurrence aux opérations de base de données impliquant l'entité. (*Voir callout en dessous du tableau*)                  |
| `ConcurrencyCheck`          | Champ d'indicateurs à utiliser pour la vérification de la concurrence lors de l'exécution des mises à jour et des suppressions. (*Voir callout en dessous du tableau*)                                      |
| `DatabaseGenerated`         | Indique si le champ est généré par la base de données ou non. Accepte la valeur `DatabaseGeneratedOption` : `Computed`, `Identity` ou `None`.                                                               |
| `DataType`                  | Fournit une définition plus précise d'un champ que le type de données intrinsèque.                                                                                                                          |
| *`Unicode`*                 | Associe une propriété de type chaîne à une colonne de base de données Unicode/non-Unicode sans spécifier le type de données natif.                                                                          |
| *`Precision`*               | Spécifie la précision et l'échelle de la colonne de base de données sans spécifier le type de données natif.                                                                                                |
| `NotMapped`                 | Exclut la propriété ou la classe en ce qui concerne les champs et les tables de la base de données.                                                                                                         |
|                             |                                                                                                                                                                                                             |

>[!warning]- Comment Postgres gère la vérification de la concurrence ? (Avec Claude)
>## Pour les version récentes de `Npgsql`
>
>Depuis *Npgsql 7.0*, `UseXminAsConcurrencyToken()` a été remplacé par l'approche standard EF Core : utiliser `IsRowVersion()` via Fluent API ou `[Timestamp]` via Data Annotation sur n'importe quelle propriété `uint`.
>
>Le générateur de migrations Npgsql supprime automatiquement les colonnes système PostgreSQL comme `xmin` des opérations de migration. **Cela signifie que la colonne restera cachée de l'utilisateur**.
>
>## Pour `Npgsql` <= 6.0
>
.`[TimeStamp]` / `rowversion` n'existe pas. L'équivalent est la colonne système `xmin` présente automatiquement dans chaque table PostgreSQL, configurée via `UseXminAsConcurrencyToken()` dans `OnModelCreating`. Il n'existe pas d'attribut équivalent pour `xmin` avec Npgsql. **La propriété mappée doit être de type `uint` car c'est le type natif de `xmin` dans PostgreSQL.**
>
>#### `[ConcurrencyCheck]`
>
>Lors d'un `UPDATE`, EF Core ajoute la valeur de la propriété dans le `WHERE` :
>
>```sql
>UPDATE "Cars" SET "Color" = 'Red'
>WHERE "Id" = 1 AND "Color" = 'Blue' -- Vérifie que la valeur n'a pas changé
>```
>
>Si aucune ligne n'est affectée (parce que quelqu'un d'autre a modifié `Color` entre temps) → `DbUpdateConcurrencyException`.
>
>C'est bien une **comparaison de valeur**, tu avais raison sur ce point.
>
>#### `UseXminAsConcurrencyToken()` / `xmin`
>
>`xmin` n'est pas un ID de ligne, c'est un **ID de transaction**. À chaque modification de la ligne, PostgreSQL met à jour `xmin` avec l'ID de la transaction courante. EF Core l'utilise ainsi :
>
>```sql
>UPDATE "Cars" SET "Color" = 'Red'
>WHERE "Id" = 1 AND xmin = 42 -- Vérifie que personne d'autre n'a modifié la ligne
>```
>
>Si `xmin` a changé depuis la lecture → quelqu'un d'autre a modifié la ligne → `DbUpdateConcurrencyException`.
>

**Le code suivant présente la classe `BaseEntity` avec des annotations déclarant le champ `Id` comme clé primaire. La seconde annotation de données sur la propriété `Id` indique qu'il s'agit d'une colonne d'identité dans SQL Server/PosgreSQL**. ~~La propriété `TimeStamp` sera une propriété d'horodatage/version de ligne SQL Server~~ **(la vérification de la concurrence est traitée plus loin dans ce chapitre).**

```cs
public abstract class BaseEntity
{
    [Key, DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    public int Id { get; set; }

    public DateTime TimeStamp { get; set; } // Horodatage ordinaire, géré soi-même

    [Timestamp]
    public uint xmin { get; set; }          // Concurrency token, géré par PostgreSQL
}
```

Voici la classe `Car` et les annotations de données qui la structurent dans la base de données, expliquées après l'exemple de code :

```cs
namespace AutoLot.Samples.Models;

[Table("Inventory", Schema = "public")]
[Index(nameof(MakeId), Name = "IX_Inventory_MakeId")]
public class Car : BaseEntity
{
    private string _color;

    [Required, StringLength(50)]
    public string Color
    {
        get => _color;
        set => _color = value;
    }

    [Required, StringLength(50)]
    public string PetName { get; set; }
    public int MakeId { get; set; }

    [ForeignKey(nameof(MakeId))]
    public Make MakeNavigation { get; set; }
    public Radio RadioNavigation { get; set; }

    [InverseProperty(nameof(Driver.Cars))]
    public IEnumerable<Driver> Drivers { get; set; } = new List<Driver>();
}
```

**L'attribut `Table` associe la classe `Car` à la table `Inventory` du schéma `public`.** ==L'attribut `Column` (non illustré dans cet exemple) fonctionne de manière similaire et permet de modifier le nom ou le type de données d'une colonne.== **L'attribut Index crée un index sur la clé étrangère `MakeId`.** ***==Les deux champs texte (`Color` et `PetName`) sont obligatoires et leur longueur maximale est de 50 caractères.==*** **Les attributs `InverseProperty` et `ForeignKey` sont expliqués dans la section suivante.** **==Les modifications apportées par rapport aux conventions EF Core sont les suivantes :==**

- Renommage de la table `Cars` en `Inventory`.
- Modification de la propriété `xmin`, actuellement une colonne `uint` standard, pour utiliser la colonne système `xmin` de PostgreSQL (type `xid`), servant de jeton de concurrence.
- Modification du comportement des colonnes `Color` et `PetName` (autorisation de valeurs nulles désactivée).
- Définition explicite de la taille des colonnes `Color` et `PetName` à `VARCHAR(50)`. ==Ceci était déjà géré lors de la redéfinition des conventions EF Core pour les propriétés de type chaîne, mais est inclus ici par souci de clarté.==
- • Renommage de l'index sur la colonne `MakeId`.

Les autres annotations utilisées correspondent à la configuration définie par les conventions EF Core. Pour confirmer les modifications, nous examinons la table créée par EF Core :

```sql
CREATE TABLE "Inventory" (
    "Id" SERIAL NOT NULL,
    "TimeStamp" TIMESTAMP WITH TIME ZONE NOT NULL,
    "Color" VARCHAR(50) NOT NULL,
    "PetName" VARCHAR(50) NOT NULL,
    "MakeId" INTEGER NOT NULL,
    CONSTRAINT "PK_Inventory" PRIMARY KEY ("Id"),
    CONSTRAINT "FK_Inventory_Makes_MakeId" FOREIGN KEY ("MakeId")
        REFERENCES "Makes" ("Id")
        ON DELETE CASCADE
);

CREATE INDEX "IX_Inventory_MakeId" ON "Inventory" ("MakeId");
```

>**`xmin`** : colonne système PostgreSQL, pas générée dans le SQL de migration comme expliqué plus tôt.

>[!note]
Contrairement à SQL Server où l'ajout du type `timestamp`/`rowversion` nécessite une suppression et recréation de la colonne (car `ALTER COLUMN` est impossible vers ce type), PostgreSQL n'a pas ce problème. `xmin` étant une colonne système déjà présente dans chaque table dès sa création, aucune migration ne touche à cette colonne. Ajouter ou retirer `[Timestamp]` sur la propriété `xmin` ne génère aucune opération DDL dans les migrations.

==Notez les valeurs par défaut ajoutées aux colonnes `Color` et `PetName`.== **Si des données contenaient des valeurs nulles, la migration échouerait.** **==Cette modification garantit que la migration (qui n'accepte plus les valeurs nulles) réussira en insérant une chaîne vide dans ces colonnes si elles étaient nulles au moment de l'application de la migration.==**

**Pour que la propriété `CarId` de l'élément `Radio` corresponde à un champ nommé `InventoryId`, et pour rendre `RadioId` obligatoire et définir explicitement sa taille à $50$**, mettez à jour l'entité `Radio` comme suit :

```cs
namespace AutoLot.Samples.Models;

[Table("Radios", Schema = "public")]
public class Radio : BaseEntity
{
    public bool HasTweeters { get; set; }
    public bool HasSubWoofers { get; set; }

    [Required, StringLength(50)]
    public string RadioId { get; set; }

    [Column("InventoryId")]
    public int CarId { get; set; }

    [ForeignKey(nameof(CarId))]
    public Car CarNavigation { get; set; }
}
```

Les commandes de migration et le code SQL résultant sont présentés ici :

```bash
dotnet ef migrations add UpdateRadio -o Migrations -c AutoLot.Samples.ApplicationDbContext

dotnet ef database update UpdateRadio -c AutoLot.Samples.ApplicationDbContext
```

```sql
CREATE TABLE "Radios" (
    "Id" SERIAL NOT NULL,
    "TimeStamp" TIMESTAMP WITH TIME ZONE NOT NULL,
    "HasTweeters" BOOLEAN NOT NULL,
    "HasSubWoofers" BOOLEAN NOT NULL,
    "RadioId" VARCHAR(50) NOT NULL,
    "InventoryId" INTEGER NOT NULL,
    CONSTRAINT "PK_Radios" PRIMARY KEY ("Id"),
    CONSTRAINT "FK_Radios_Inventory_InventoryId" FOREIGN KEY ("InventoryId")
        REFERENCES "Inventory" ("Id")
        ON DELETE CASCADE
);

CREATE UNIQUE INDEX "IX_Radios_InventoryId" ON "Radios" ("InventoryId");
```

**Pour terminer la mise à jour de nos modèles, rendez la propriété `Name` de l'entité `Make` obligatoire, et définissez sa longueur maximale à $50$ caractères.** Procédez de même pour les propriétés `FirstNam`e et `LastName` de l'entité `Driver`.

```cs
// Make.cs
namespace AutoLot.Samples.Models;

[Table("Makes", Schema = "public")]
public class Make : BaseEntity
{
    [Required, StringLength(50)]
    public string Name { get; set; }

    [InverseProperty(nameof(Car.MakeNavigation))]
    public IEnumerable<Car> Cars { get; set; } = new List<Car>();
}
```

```cs
// Driver.cs
namespace AutoLot.Samples.Models;

[Table("Drivers", Schema = "public")]
public class Driver : BaseEntity
{
    [Required, StringLength(50)]
    public string FirstName { get; set; }

    [Required, StringLength(50)]
    public string LastName { get; set; }

    [InverseProperty(nameof(Car.Drivers))]
    public IEnumerable<Car> Cars { get; set; } = new List<Car>();
}
```

####  Annotations et propriétés de navigation

**L'annotation `ForeignKey` indique à EF Core quelle propriété sert de champ sous-jacent à la propriété de navigation.** **==Par convention, `<TypeName>Id` est automatiquement défini comme propriété de clé étrangère.==** ==Dans les exemples précédents, il est explicitement défini pour plus de clarté. Cela permet d'utiliser différents styles de nommage et d'avoir plusieurs clés étrangères vers la même table.== **Notez que dans une relation un-à-un, seule l'entité dépendante possède une clé étrangère.**

**`InverseProperty` indique à EF Core la relation entre les entités en précisant la propriété de navigation de l'autre entité qui renvoie vers cette entité.** **==`InverseProperty` est requis lorsqu'une entité est liée à une autre entité plus d'une fois et, à mon avis, améliore également la lisibilité du code.==**

### L'API Fluent

>[!tip]
>"Fluent API" est un abus de langage marketing autant que technique. C'est surtout un **pattern de design** (Builder pattern + method chaining) qu'on a baptisé "API" pour le distinguer des autres approches de configuration.

**L'API Fluent configure les entités de l'application via du code C#.** ***==Les méthodes sont exposées par l'instance `ModelBuilder` disponible dans la méthode `OnModelCreating()` de `DbContext`.==*** ***L'API Fluent est la plus puissante des méthodes de configuration et remplace toute convention ou annotation de données en conflit.*** **Certaines options de configuration sont uniquement disponibles via l'API Fluent, comme la définition de valeurs par défaut et le comportement en cascade des propriétés de navigation.**


#### Méthodes de classe et de propriété

**L'API Fluent est un sur-ensemble des annotations de données pour la structuration de vos entités.** **==Elle prend en charge toutes les fonctionnalités des annotations de données, et offre des capacités supplémentaires, telles que la spécification de clés et d'index composites, ainsi que la définition de colonnes calculées.==**

##### Correspondance entre classes et propriétés

**Le code suivant illustre l'exemple précédent de la classe `Car` avec l'API Fluent équivalent aux annotations de données utilisées** (*==en omettant les propriétés de navigation, qui seront abordées ensuite==*).

```cs
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
	// Les appels à l'API Fluent vont ici
	modelBuilder.Entity<Car>(entity =>
	{
		entity.ToTable("Inventory", "public");
	});
}
```

La table suivante associe la propriété `CarId` de la classe `Radio` à la colonne `InventoryId` de la table `Makes` :

```cs
modelBuilder.Entity<Radio>(entity =>
{
	entity.Property(e => e.CarId).HasColumnName("InventoryId");
});
```

##### Clés et index

**Pour définir la clé primaire d'une entité, utilisez la méthode `HasKey()`, comme indiqué ici :**

```cs
modelBuilder.Entity<Car>(entity =>
{
	entity.ToTable("Inventory", "public");
	entity.HasKey(e => e.Id);
});
```

**Pour définir une clé composite, sélectionnez les propriétés qui composent la clé dans l'expression de la méthode `HasKey()`.** Par exemple, si la clé primaire de l'entité `Car` doit être constituée des colonnes `Id` et d'une propriété `OrganizationId`, vous la définirez comme suit :

```cs
modelBuilder.Entity<Car>(entity =>
{
	entity.ToTable("Inventory", "public");
	entity.HasKey(e => e.Id = new {e.Id, e.OrganizationId});
});
```

**La procédure est identique pour la création d'index, à ceci près qu'elle utilise la méthode `HasIndex()` de l'API Fluent.** Par exemple, pour créer un index nommé `IX_Inventory_MakeId`, utilisez le code suivant :

```cs
modelBuilder.Entity<Car>(entity =>
{
	entity.ToTable("Inventory", "public");
	entity.HasKey(e => e.Id);
	entity.HasIndex(e => e.MakeId, "IX_Inventory_MakeId");
});
```

**Pour rendre l'index unique, utilisez la méthode `IsUnique()`. La méthode `IsUnique()` accepte un `bool` optionnel qui vaut `true` par défaut :**

```cs
entity.HasIndex(e => e.MakeId, "IX_Inventory_MakeId").IsUnique();
```

##### Taille des champs et tolérance aux valeurs nulles

**Les propriétés sont configurées en les sélectionnant à l'aide de la méthode `Property()`, puis en utilisant des méthodes supplémentaires pour les configurer. Vous avez déjà vu un exemple avec le mappage de la propriété `CarId` à la colonne `InventoryId`.**

==La méthode `IsRequired()` accepte un `bool` optionnel, défini par défaut à `true`, et détermine si la colonne de la base de données peut contenir des valeurs nulles. La méthode `HasMaxLength()` définit la taille de la colonne.== Voici le code de l'API Fluent qui définit les propriétés `Color` et `PetName` comme obligatoires avec une longueur maximale de $50$ caractères :

```cs
modelBuilder.Entity<Car>(entity =>
{
	...
	
	entity.Property(e => e.Color).IsRequired().HasMaxLength(50);
	entity.Property(e => e.PetName).IsRequired().HasMaxLength(50);
}
```

##### Valeurs par défaut

**L'API Fluent fournit des méthodes pour définir des valeurs par défaut pour les colonnes. La valeur par défaut peut être un type valeur ou une chaîne SQL.** Par exemple, pour définir la valeur de `Color` par défaut d'une nouvelle entité `Car` sur `Black`, utilisez ce qui suit :

```cs
modelBuilder.Entity<Car>(entity =>
{
	...
	entity
		.Property(e => e.Color)
		.IsRequired()
		.HasMaxLength(50)
		.HasDefaultValue("Black");
	...
});
```

Pour définir la valeur d'une fonction de base de données (comme `getdate()`), utilisez la méthode `HasDefaultValueSql()`. Supposons qu'une nouvelle propriété `DateTime` nommée `DateBuilt` ait été ajoutée à la classe `Car` :

```cs
public class Car : BaseEntity
{
	...
	
    public DateTime? DateBuilt { get; set; }
}
```

La valeur par défaut doit être la date actuelle, calculée à l'aide de la méthode `getdate()` de SQL Server (`NOW()` ou `CURRENT_TIMESTAMP` pour PostgreSQL). Pour configurer cette propriété avec cette valeur par défaut, utilisez le code d'API Fluent suivant :

```cs
modelBuilder.Entity<Car>(entity =>
{
	...
	
	entity.Property(e => e.DateBuilt).HasDefaultValueSql("NOW()");
}
```

 **Postgres utilisera le résultat de la fonction `NOW()` si la propriété `DateBuilt` de l'entité n'a pas de valeur lors de son enregistrement dans la base de données.**

*==Un problème survient lorsqu'une propriété booléenne ou numérique possède une valeur par défaut dans la base de données qui contredit la valeur par défaut du CLR.==* Par exemple, ***==si une propriété booléenne (telle que `IsDrivable`) a la valeur par défaut `true` dans la base de données, celle-ci lui attribuera la valeur `true` lors de l'insertion d'un enregistrement si aucune valeur n'est spécifiée pour cette colonne.==*** **==Il s'agit, bien entendu, du comportement attendu côté base de données.==** *==Cependant, la valeur par défaut du CLR pour les propriétés booléennes est `false`, ce qui pose problème en raison de la manière dont EF Core gère les valeurs par défaut.==* 

Par exemple, ajoutez une propriété `bool` nommée `IsDrivable` à la classe `Car`. Si vous suivez ce tutoriel, assurez-vous de créer et d'appliquer une nouvelle migration pour mettre à jour la base de données.

```cs
public class Car : BaseEntity
{
	...
	
    public bool IsDrivable { get; set; }
}
```

Avant d'aborder les valeurs par défaut, examinons le comportement d'EF Core pour le type de données `bool`. Prenons l'exemple de code suivant qui crée un enregistrement `Car` avec la propriété `IsDrivable` définie sur `false` :

```sql
INSERT INTO "Inventory" ("Color", "IsDrivable", "MakeId", "PetName", "Price")
VALUES ('Rust', false, 1, 'Lemon', NULL)
RETURNING "Id", "DateBuilt", "Display", "IsDeleted", "TimeStamp", "ValidFrom", "ValidTo";
```

>[!tip]
>Au lieu de faire un `INSERT` puis un `SELECT` séparé comme SQL Server, PostgreSQL retourne directement les valeurs générées en une seule opération `RETURNING`. C'est plus performant et atomique.

Définissez maintenant la valeur par défaut du mappage de colonne de la propriété sur `true` dans la méthode `OnModelCreating()` de `ApplicationDbContext` (encore une fois : ce qui crée et applique une migration de base de données) :

```cs
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
	// Les appels à l'API Fluent vont ici
	modelBuilder.Entity<Car>(entity =>
	{
		...

		entity.Property(e => e.IsDrivable).HasDefaultValue(true);
	}
}
```

L'exécution du même code pour insérer l'enregistrement de voiture précédent génère une requête SQL différente :

```sql
INSERT INTO "Inventory" ("Color", "MakeId", "PetName", "Price")
VALUES ('Rust', 1, 'Lemon', NULL)
RETURNING "Id", "DateBuilt", "Display", "IsDeleted", "IsDrivable", "TimeStamp", "ValidFrom", "ValidTo";
```

**Notez que la colonne `IsDrivable` n'est pas incluse dans l'instruction `INSERT`. EF Core sait que la valeur de la propriété `IsDrivable` est la valeur par défaut du CLR et que cette colonne possède une valeur par défaut Postgres**. ==Par conséquent, la colonne n'est pas incluse dans l'instruction.== **==Ainsi, lorsque vous enregistrez un nouvel enregistrement avec `IsDrivable = false`, la valeur est ignorée et la valeur par défaut de la base de données est utilisée.==** **Cela signifie que la valeur de `IsDrivable` sera toujours `true` !**

***==EF Core vous signale ce problème lors de la création d'une migration. Dans cet exemple précis, l'avertissement suivant s'affiche :==***

```
The 'bool' property 'IsDrivable' on entity type 'Car' is configured with a database-generated default. This default will always be used for inserts when the property has the value 'false', since this is the CLR default for the 'bool' type. Consider using the nullable 'bool?' type instead, so that the default will only be used for inserts when the property value is 'null'.
```

>[!success] Note moderne (Avec Claude)
>Le warning n'apparaît plus avec EF Core 8+. Ce breaking change corrige le comportement sous-jacent : EF Core 8+ détecte correctement si `false` est une valeur explicite ou la valeur par défaut CLR, rendant le `bool?` inutile dans ce contexte. Utiliser `bool` avec `HasDefaultValue(true)` est maintenant sûr sans nullable.

>La solution de l'auteur sera gardée à des fins pédagogique

Une solution consiste à rendre votre propriété publique (et donc la colonne) nullable, car la valeur par défaut d'un type nullable est `null`. Ainsi, définir une propriété `bool` sur `false` fonctionne comme prévu. Cependant, modifier la nullabilité de la propriété peut ne pas répondre aux besoins métier.

**==Une autre solution est fournie par EF Core et sa prise en charge des champs de stockage. Rappelons que si un champ de stockage existe==** (et est identifié comme tel pour la propriété par convention, annotation de données ou API Fluent), **==EF Core utilisera ce champ pour les opérations de lecture/écriture et non la propriété publique.==**

Si vous configurez `IsDrivable` pour utiliser un champ de stockage nullable (tout en conservant la propriété non nullable), EF Core effectuera les opérations de lecture/écriture à partir du champ de stockage et non de la propriété. La valeur par défaut d'un `bool` nullable est `null` et non `false`. Cette modification permet désormais à la propriété de fonctionner comme prévu.

```cs
public class Car : BaseEntity
{
	...
	
    private bool? _isDrivable;
    public bool IsDrivable
    {
        get => _isDrivable ?? true;
        set => _isDrivable = value;
    }
}
```

L'API Fluent est utilisée pour informer EF Core du champ de stockage.

```cs
entity
	.Property(e => e.IsDrivable)
	.HasField("_isDriveable")
	.HasDefaultValue(true);
```

>[!note]
>La méthode `HasField()` n'est pas nécessaire dans cet exemple, car le nom du champ sous-jacent respecte les conventions d'appellation. Je l'ai incluse pour montrer comment utiliser l'API Fluent pour le définir et pour faciliter la lecture du code.

**Bien que non illustrées dans les exemples précédents, les propriétés numériques fonctionnent de la même manière. Si vous définissez une valeur par défaut non nulle, le champ sous-jacent (ou la propriété elle-même si aucun champ sous-jacent n'est utilisé) doit autoriser les valeurs nulles.**

*==Enfin, l'avertissement s'affichera même si les champs sont correctement configurés avec des champs sous-jacents autorisant les valeurs nulles.==* ***==Vous pouvez désactiver cet avertissement ; toutefois, il est recommandé de le laisser affiché afin de vous rappeler de vérifier que le champ/la propriété est correctement configuré(e).==*** Pour le désactiver, définissez l'option suivante dans `DbContextOptions`:

```cs
options.ConfigureWarnings(wc => wc.Ignore(RelationalEventId.BoolWithDefaultWarning));
```

##### version de ligne/jeton de concurrence

**Pour définir une propriété comme type de données de version de ligne, utilisez la méthode `IsRowVersion()`. Pour également définir la propriété comme jeton de concurrence, utilisez la méthode `IsConcurrencyToken()`. La combinaison de ces deux méthodes a le même effet que l'annotation de données `[Timestamp]` :**

```cs
modelBuilder.Entity<Car>(entity =>
{
	...
	entity.Property(e => e.xmin).IsRowVersion().IsConcurrencyToken();
});
```


>[!tip]
>Avec Npgsql moderne, `IsRowVersion().IsConcurrencyToken()` s'applique sur la propriété `xmin` de type `uint`, pas sur `TimeStamp` de type `DateTime`. `IsRowVersion()` implique déjà `IsConcurrencyToken()`, donc les deux appels combinés sont redondants mais valides.

La vérification de la concurrence sera abordée dans le chapitre suivant.

##### Colonnes creuses SQL Server

Les colonnes creuses SQL Server sont optimisées pour stocker les valeurs nulles. EF Core 6 a ajouté la prise en charge des colonnes creuses avec la méthode `IsSparse()` de l'API Fluent. Le code suivant illustre la configuration de la propriété fictive `IsRaceCar` pour utiliser les colonnes creuses SQL Server :

>[!Attention] 
>`IsSparse()` est une méthode **SQL Server uniquement** dans EF Core. Elle n'est pas supportée par Npgsql/PostgreSQL. PostgreSQL gère les valeurs nulles efficacement nativement sans nécessiter de configuration spéciale. Pour les scénarios TPH avec beaucoup de nulls, préférer TPT ou des colonnes JSONB sur PostgreSQL

```cs
modelBuilder.Entity<Car>(entity =>
{
	...
	entity.Property(p => p.IsRaceCare).IsSparse();
});
```

##### Colonnes calculées

**Les colonnes peuvent également être définies comme calculées en fonction des capacités de la base de données.** Pour SQL Server, deux options sont possibles : calculer la valeur à partir de la valeur d'autres champs du même enregistrement ou utiliser une fonction scalaire. Par exemple, pour créer une colonne calculée dans la table Inventory qui combine les valeurs PetName et Color afin de créer une propriété nommée Display, utilisez la fonction HasComputedColumnSql().

>[!info] Pour Postgres
>PostgreSQL supporte `HasComputedColumnSql()` mais **uniquement en mode persisté** (`stored: true`). Les colonnes calculées à la volée (`stored: false`) ne sont pas supportées par PostgreSQL. 
>>[!warning] La syntaxe de concaténation utilise `||` au lieu de `+`.
>

Commencez par ajouter la nouvelle colonne à la classe `Car` :

```cs
public class Car : BaseEntity
{
	...
    public string Display { get; set; }
}
```

Ajoutez ensuite l'appel d'API Fluent à `HasComputedColumnSql()` :

```cs
modelBuilder.Entity<Car>(entity =>
{
	...
	
	entity
		.Property(e => e.Display)
		.HasComputedColumnSql(
			@"""PetName"" || ' (' || ""Color"" || ')'",
		);
});
```

***==Introduite dans EF Core 5, la fonctionnalité de persistance des valeurs calculées permet d'effectuer le calcul uniquement lors de la création ou de la mise à jour d'une ligne. Bien que SQL Server prenne en charge cette fonctionnalité, ce n'est pas le cas de tous les systèmes de stockage de données. Consultez donc la documentation de votre fournisseur de base de données.==***

```cs
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
	...
	
	entity
		.Property(e => e.Display)
		.HasComputedColumnSql(
			@"""PetName"" || ' (' || ""Color"" || ')'",
			stored: true
		);
});
```

**L'annotation de données `DatabaseGenerated` est souvent utilisée avec l'API Fluent pour améliorer la lisibilité du code.** Voici la version mise à jour de la propriété `Display` avec l'annotation :

```cs
public class Car : BaseEntity
{
	...
	
    [DatabaseGenerated(DatabaseGeneratedOption.Computed)]
    public string Display { get; set; }
}
```

##### Contraintes de vérification

>[!tip]
>Les contraintes de vérification `HasCheckConstraint()` sont une fonctionnalité **SQL standard** disponible sur tous les moteurs de base de données, pas uniquement SQL Server. La syntaxe Fluent API est identique pour PostgreSQL, seuls les guillemets des noms de colonnes changent (`"Name"` au lieu de `[Name]`).

>[!warning] Depuis EF Core 7, `HasCheckConstraint()` directement sur l'entity builder est obsolète. Il faut désormais l'appeler via `ToTable()` avec un builder action. La syntaxe PostgreSQL utilise `"NomColonne"` (guillemets doubles) pour les colonnes et `'valeur'` (guillemets simples) pour les littéraux string.

Les contraintes de vérification sont une fonctionnalité de ~~SQL Server~~ qui définit une condition sur une ligne qui doit être vraie. Par exemple, **dans un système de commerce électronique, une contrainte de vérification peut être ajoutée pour s'assurer que la quantité est supérieure à zéro ou que le prix est supérieur au prix remisé.** ==Comme notre système ne contient aucune valeur numérique, nous allons créer une contrainte artificielle qui empêche l'utilisation du nom `"Lemon"` dans la table `Makes`.==

Ajoutez ce qui suit à la méthode `OnModelCreating()` de la classe `ApplicationDbContext`, qui crée la contrainte de vérification empêchant la présence du nom `"Lemon"` dans la table `Makes` :

```cs
modelBuilder
	.Entity<Make>()
	.ToTable(
		"Makes",
		"public",
		t =>
			t.HasCheckConstraint("CK_Check_Name", "\"Name\" <> 'Lemon'")
	);
```

```sql
ALTER TABLE "Makes"
ADD CONSTRAINT "CK_Check_Name" CHECK ("Name" <> 'Lemon');
```

Désormais, lorsqu'un enregistrement nommé « Lemon » est ajouté à la table, une exception SQL est levée. Exécutez le code suivant pour observer cette exception (dans *Program.cs*) :

```cs
var context = new ApplicationDbContextFactory().CreateDbContext(null);
context.Makes.Add(new Make { Name = "Lemon" });
context.SaveChanges();
```

Cela génère l'exception suivante :

```
...

 Severity: ERROR
    SqlState: 23514
    MessageText: new row for relation "Makes" violates check constraint "CK_Check_Name"
    Detail: Detail redacted as it may contain sensitive data. Specify 'Include Error Detail' in the connection string to include this information.
    SchemaName: public
    TableName: Makes
    ConstraintName: CK_Check_Name
    
...
```

>**Le texte montré ici est caché en base de toute la stacktrace de Npgsql (car on n'a pas géré l'exception)**

**Vous pouvez annuler la migration relative à la contrainte de vérification et la supprimer, car le reste du livre n'utilise pas cette contrainte. Elle a été ajoutée dans cette section à titre d'exemple.**

>[!Danger] 
>Comme les tables que l'on vient de crée en C# sont assez différentes des originaux, des erreurs vont survenir et les données présentes dans les tables seront supprimés.
>
> il est préférable de `DROP` la base de donnée et la recréer, exécuter la migration (`dotnet ef update ...`) et enfin peupler les tables (avec les scripts vu dans le [[Chapitre 20#Ajout d'entrées de test|Chapitre 20]])

>[!attention]
Si la migration a déjà été appliquée à la base de donnée (`dotnet ef database update`), il faut d'abord revenir en arrière :
>```bash
># Revenir à la migration précédente
>dotnet ef database update NomMigrationPrécédente -c AutoLot.Samples.ApplicationDbContext
>
># Puis supprimer
>dotnet ef migrations remove -c AutoLot.Samples.ApplicationDbContext
>```

#### Relations un-à-plusieurs

**Pour définir des relations un-à-plusieurs avec l'API Fluent, sélectionnez l'*une* des entités à mettre à jour. Les deux côtés de la chaîne de navigation sont définis dans un seul bloc de code.**

```cs
modelBuilder.Entity<Car>(entity =>
{
	...
	entity.HasOne(d => d.MakeNavigation)
		.WithMany(p => p.Cars)
		.HasForeignKey(d => d.MakeId)
		.OnDelete(DeleteBehavior.ClientSetNull)
		.HasConstraintName("FK_Inventory_Makes_MakeId");
});
```

**==Si vous sélectionnez l'entité principale comme base pour la configuration des propriétés de navigation,==** le code ressemble à ceci :

```cs
 modelBuilder.Entity<Make>(entity =>
 {
     entity.HasMany(e => e.Cars)
         .WithOne(c => c.MakeNavigation)
         .HasForeignKey(c => c.MakeId)
         .OnDelete(DeleteBehavior.ClientSetNull)
         .HasConstraintName("FK_Inventory_Makes_MakeId");
 });
```

#### Relations un-à-un

**Les relations un-à-un se configurent de la même manière, à ceci près que la méthode `WithOne()` de l'API Fluent est utilisée au lieu de `WithMany()`.** ***==De plus, un index unique est requis sur l'entité dépendante et sera créé automatiquement s'il n'est pas défini.==*** L'exemple suivant crée explicitement l'index unique pour spécifier le nom. Voici le code de la relation entre les entités Voiture et Radio utilisant l'entité dépendante (`Radio`) :

```cs
modelBuilder.Entity<Radio>(entity =>
{
	...

	entity.HasIndex(e => e.CarId, "IX_Radios_CarId").IsUnique();
	entity
		.HasOne(d => d.CarNavigation)
		.WithOne(p => p.RadioNavigation)
		.HasForeignKey<Radio>(d => d.CarId);
});
```

**Si la relation est définie sur une entité principale, un index unique sera tout de même ajouté à l'entité dépendante.** Voici le code de la relation entre les entités `Car` et `Radio`, utilisant l'entité principale pour la relation et spécifiant le nom de l'index sur l'entité dépendante :

```cs
modelBuilder.Entity<Radio>(entity =>
{
	entity.HasIndex(e => e.CarId, "IX_Radios_CarId")
		.IsUnique();
});

modelBuilder.Entity<Car>(entity =>
{
	entity.HasOne(d => d.RadioNavigation)
		.WithOne(p => p.CarNavigation)
		.HasForeignKey<Radio>(d => d.CarId);
});
```

#### Relations plusieurs-à-plusieurs

**Les relations plusieurs-à-plusieurs sont bien plus personnalisables grâce à l'API Fluent.** **==Les noms des champs de clé étrangère, les noms des index et le comportement en cascade peuvent tous être définis dans les instructions qui définissent la relation.==** ***==L'API permet également de spécifier directement la table pivot, ce qui permet d'ajouter des champs supplémentaires et de simplifier les requêtes.==***

Commencez par ajouté l'entité `CarDriver` :

```cs
namespace AutoLot.Samples.Models;

[Table("InventoryToDrivers", Schema = "public")]
public class CarDriver : BaseEntity
{
    public int DriverId { get; set; }

    [ForeignKey(nameof(DriverId))]
    public Driver DriverNavigation { get; set; }

    [Column("InventoryId")]
    public int CarId { get; set; }

    [ForeignKey(nameof(CarId))]
    public Car CarNavigation { get; set; }
}
```

Ajoutez un `DbSet<T>` pour la nouvelle entité dans `ApplicationDbContext` :

```cs
public DbSet<CarDriver> CarsToDrivers { get; set; }
```

Ensuite, mettez à jour l'entité `Car` pour ajouter une propriété de navigation pour la nouvelle entité `CarDriver` :

```cs
public class Car : BaseEntity
{
	...

    [InverseProperty(nameof(CarDriver.CarNavigation))]
    public IEnumerable<CarDriver> CarDrivers { get; set; } =
        new List<CarDriver>();
}
```

Maintenant, mettez à jour l'entité `Driver` pour la propriété navigation vers l'entité `CarDriver` :

```cs
public class Driver : BaseEntity
{
    [InverseProperty(nameof(CarDriver.DriverNavigation))]
    public IEnumerable<CarDriver> CarDrivers { get; set; } =
        new List<CarDriver>();
}
```

Enfin, ajoutez le code de l'API Fluent pour la relation plusieurs-à-plusieurs :

```cs
modelBuilder
	.Entity<Car>()
	.HasMany(p => p.Drivers)
	.WithMany(p => p.Cars)
	.UsingEntity<CarDriver>(
		j =>
			j.HasOne(cd => cd.DriverNavigation)
				.WithMany(d => d.CarDrivers)
				.HasForeignKey(nameof(CarDriver.DriverId))
				.HasConstraintName("FK_Inventory_Drivers_DriverId")
				.OnDelete(DeleteBehavior.Cascade),
		j =>
			j.HasOne(cd => cd.CarNavigation)
				.WithMany(c => c.CarDrivers)
				.HasForeignKey(nameof(CarDriver.CarId))
				.HasConstraintName(
					"FK_InventoryDriver_Inventory_InventoryId"
				)
				.OnDelete(DeleteBehavior.Cascade),
		j =>
		{
			j.HasKey(cd => new { cd.CarId, cd.DriverId });
		}
	);
```

#### Exclusion d'entités des migrations

***Si une entité est partagée entre plusieurs `DbContexts`, chaque `DbContext` créera du code dans les fichiers de migration pour la création ou la modification de cette entité.*** *==Cela pose problème car le second script de migration échouera si les modifications sont déjà présentes dans la base de données.==* **Avant EF Core 5, la seule solution consistait à modifier manuellement l'un des fichiers de migration pour supprimer ces modifications.**

**Dans EF Core 5, un `DbContext` peut marquer une entité comme exclue des migrations, permettant ainsi à l'autre `DbContext` de devenir le système d'enregistrement pour cette entité.** Le code suivant montre une entité exclue des migrations :

```cs
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
	modelBuilder.Entity<LogEntry>().ToTable("Logs", t => t.ExcludeFromMigrations());
}
```

#### Utilisation des classes `IEntityTypeConfiguration`

**Comme vous l'avez peut-être constaté à ce stade de l'utilisation de l'API Fluent, la méthode `OnModelCreating()` peut devenir assez longue (et difficile à manipuler) à mesure que votre modèle se complexifie.** ***==Introduites dans EF Core 6, l'interface `IEntityTypeConfiguration` et l'attribut `EntityTypeConfiguration` permettent de déplacer la configuration de l'API Fluent d'une entité dans sa propre classe.==*** **==Cela permet d'obtenir un `ApplicationDbContext` plus clair et soutient le principe de séparation des préoccupations.==**

Commencez par créer un nouveau répertoire nommé `Configuration` dans le répertoire `Models`. Dans ce nouveau répertoire, ajoutez un nouveau fichier nommé *CarConfiguration.cs*, déclarez-le public et implémentez l'interface `IEntityTypeConfiguration<Car>`, comme ceci :

```cs
namespace AutoLot.Samples.Models.Configuration;

public class CarConfiguration : IEntityTypeConfiguration<Car>
{
    public void Configure(EntityTypeBuilder<Car> builder) { }
}
```

**Ensuite, déplacez le contenu de la configuration de l'entité `Car` depuis la méthode `OnModelCreating()` de `ApplicationDbContext` vers la méthode `Configure()` de la classe `CarConfiguration`. Remplacez la variable `entity` par la variable `builder` afin que la méthode `Configure()` ressemble à ceci :**

```cs
public void Configure(EntityTypeBuilder<Car> builder)
{
	builder.ToTable("Inventory", "public");
	builder.HasKey(e => e.Id);
	builder.HasIndex(e => e.MakeId, "IX_Inventory_MakeId").IsUnique();
	builder
		.Property(e => e.Color)
		.IsRequired()
		.HasMaxLength(50)
		.HasDefaultValue("Black");
	builder.Property(e => e.PetName).IsRequired().HasMaxLength(50);
	builder.Property(e => e.DateBuilt).HasDefaultValueSql("NOW()");
	builder
		.Property(e => e.IsDrivable)
		.HasField("_isDrivable")
		.HasDefaultValue(true);
	builder.Property(e => e.xmin).IsRowVersion().IsConcurrencyToken();
	builder
		.Property(e => e.Display)
		.HasComputedColumnSql(
			@"""PetName"" || ' (' || ""Color"" || ')'",
			stored: true
		);
	builder
		.HasOne(d => d.MakeNavigation)
		.WithMany(p => p.Cars)
		.HasForeignKey(d => d.MakeId)
		.OnDelete(DeleteBehavior.ClientSetNull)
		.HasConstraintName("FK_Inventory_Makes_MakeId");
}
```

**Cette configuration fonctionne également avec la configuration fluide de type plusieurs-à-plusieurs entre `Car` et  `Driver`.** **==Vous pouvez choisir d'ajouter la configuration à la classe `CarConfiguration` ou de créer une classe `DriverConfiguration`.==** Dans cet exemple, déplacez-la dans la classe `CarConfiguration` à la fin de la méthode `Configure()`.

```cs
public void Configure (EntityTypeBuilder<Car> builder)
{
	...
	
	builder
		.HasMany(p => p.Drivers)
		.WithMany(p => p.Cars)
		.UsingEntity<CarDriver>(
			j =>
				j.HasOne(cd => cd.DriverNavigation)
					.WithMany(d => d.CarDrivers)
					.HasForeignKey(nameof(CarDriver.DriverId))
					.HasConstraintName("FK_Inventory_Drivers_DriverId")
					.OnDelete(DeleteBehavior.Cascade),
			j =>
				j.HasOne(cd => cd.CarNavigation)
					.WithMany(c => c.CarDrivers)
					.HasForeignKey(nameof(CarDriver.CarId))
					.HasConstraintName(
						"FK_InventoryDriver_Inventory_InventoryId"
					)
					.OnDelete(DeleteBehavior.Cascade),
			j =>
			{
				j.HasKey(cd => new { cd.CarId, cd.DriverId });
			}
		);
}
```

Mettez à jour le fichier *GlobalUsings.cs* pour inclure le nouvel espace de noms pour les classes de configuration :

```cs
global using AutoLot.Samples.Models.Configuration;
```

Remplacez tout le code de la méthode `OnModelBuilding()` (dans la classe *ApplicationDbContext.cs*) qui configure la classe `Car` et la relation plusieurs-à-plusieurs `Car`-`Driver` par la ligne de code suivante :

```cs
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
	new CarConfiguration().Configure(modelBuilder.Entity<Car>());
	
	...
}
```

**La dernière étape pour la classe `Car` consiste à ajouter l'attribut `EntityTypeConfiguration` :**

```cs
[Table("Inventory", Schema = "public")]
[Index(nameof(MakeId), Name = "IX_Inventory_MakeId")]
[EntityTypeConfiguration(typeof(CarConfiguration))]
public class Car : BaseEntity
{
	...
}
```

***==Ensuite, répétez les mêmes étapes pour le code de l'API Radio Fluent.==*** Créez une nouvelle classe nommée `RadioConfiguration`, implémentez l'interface `IEntityTypeConfiguration<Radio>` et ajoutez le code de la méthode `OnModelBuilding()` de `ApplicationDbContext` :

```cs
namespace AutoLot.Samples.Models.Configuration;

public class RadioConfiguration : IEntityTypeConfiguration<Radio>
{
    public void Configure(EntityTypeBuilder<Radio> builder)
    {
        builder.Property(e => e.CarId).HasColumnName("InventoryId");
        builder.HasIndex(e => e.CarId, "IX_Radios_CarId").IsUnique();
        builder
            .HasOne(d => d.CarNavigation)
            .WithOne(p => p.RadioNavigation)
            .HasForeignKey<Radio>(d => d.CarId);
    }
}

```

Mettez à jour la méthode `OnModelCreating()` dans `ApplicationDbContext` :

```cs
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
	new CarConfiguration().Configure(modelBuilder.Entity<Car>());
	new RadioConfiguration().Configure(modelBuilder.Entity<Radio>());
}
```

Enfin, ajoutez l'attribut `EntityTypeConfiguration` à la classe `Radio` :

```cs
[Table("Radios", Schema = "public")]
[EntityTypeConfiguration(typeof(RadioConfiguration))]
public class Radio : BaseEntity
{
	...
}
```

### Conventions, annotations et l'API Fluent : un vrai casse-tête !

Résumé du paragraphe de l'auteur:

**On peut garder les trois type de déclaration dans le code sans que cela ne fasse planter/bugger les migrations.** Seulement, elles ont une hiérarchie de priorité fixe définie par EF Core:
 
 1. **L'API Fluent** _(Priorité maximale)_
 2. **Les Data Annotations** _(Priorité moyenne)_
 3. **Les Conventions par défaut** _(Priorité minimale)_

**Les conventions par défaut seront écrasés par les annotation de données, qui seront à leurs tours écrasés par l'API Fluent.**


>[!tip] Bonne pratique moderne - Conventions, Data Annotations et Fluent API (Avec Claude)
>
>Utiliser les **Conventions par défaut** pour tout ce qu'EF Core peut inférer  automatiquement (noms de tables, clés primaires simples, relations simples...).  Ne pas les surcharger inutilement.
>
Utiliser les **Data Annotations** uniquement pour les attributs qui ont un **double rôle** : validation côté application ET contrainte en base :
>- `[Required]` → `NOT NULL` en base + validation
>- `[StringLength]` → `VARCHAR(n)` en base + validation
>- `[Range]` → validation uniquement (pas de contrainte SQL générée)
>
>>[!Attention] Certaines Data Annotations n'ont **aucun effet en base** :
>>- `[EmailAddress]`, `[Phone]`, `[Url]`... → validation côté application uniquement
>
>Utiliser la **Fluent API** pour tout ce qui concerne l'architecture de la base :
>- Clés primaires composites
>- Relations complexes (plusieurs-à-plusieurs, comportement de cascade)
>- Index et contraintes de vérification
>- Nommage spécifique des tables, colonnes et schémas
>- Valeurs par défaut et colonnes calculées

## Types d'entités propriétaires

**Il peut arriver que deux entités ou plus contiennent le même ensemble de propriétés.** **==Utiliser une classe C# comme propriété d'une entité pour regrouper des colonnes liées est une fonctionnalité disponible depuis EF Core 2.0.==** ***==Lorsque des types marqués avec l'attribut `[Owned]` (ou configurés avec l'API Fluent) sont ajoutés comme propriété d'une entité, EF Core ajoute toutes les propriétés de la classe d'entité `[Owned]` à l'entité propriétaire.==*** **Cela augmente les possibilités de réutilisation du code C#.**

**En interne, EF Core considère cette relation comme une relation un-à-un.** ***==La classe détenue est l'entité dépendante et la classe propriétaire est l'entité principale.==*** *==La classe propriétaire, bien qu'elle soit considérée comme une entité, ne peut exister sans l'entité propriétaire.==* **Les noms de colonnes par défaut du type propriétaire sont formatés comme suit : `NomPropriétéNavigation_NomPropriétéEntitéDétenu` (par exemple, `PersonalNavigation_FirstName`). Les noms par défaut peuvent être modifiés à l'aide de l'API Fluent.**

Prenons l'exemple de la classe `Person` (notez l'attribut `[Owned]`) :

```cs
namespace AutoLot.Samples.Models;

[Owned]
public class Person
{
    [Required, StringLength(50)]
    public string FirstName { get; set; }

    [Required, StringLength(50)]
    public string LastName { get; set; }
}
```

Une fois cela en place, nous pouvons remplacer les propriétés `FirstName` et `LastName` de la classe `Driver` par la nouvelle classe `Person` :

```cs
namespace AutoLot.Samples.Models;

[Table("Drivers", Schema = "public")]
public class Driver : BaseEntity
{
    public Person PersonInfo { get; set; } = new Person();

    [InverseProperty(nameof(Car.Drivers))]
    public IEnumerable<Car> Cars { get; set; } = new List<Car>();

    [InverseProperty(nameof(CarDriver.DriverNavigation))]
    public IEnumerable<CarDriver> CarDrivers { get; set; } =
        new List<CarDriver>();
}
```

**Par défaut, les deux propriétés `Person` sont associées à des colonnes nommées `PersonInfo_FirstName` et `PersonInfo_LastName`.** Pour modifier ce comportement, ajoutez d'abord un nouveau fichier nommé *DriverConfiguration.cs* dans le dossier `Configuration`, puis mettez à jour le code comme suit :

```cs
namespace AutoLot.Samples.Models.Configuration;

public class DriverConfiguration : IEntityTypeConfiguration<Driver>
{
    public void Configure(EntityTypeBuilder<Driver> builder)
    {
        builder.OwnsOne(
            o => o.PersonInfo,
            pd =>
            {
                pd.Property<string>(nameof(Person.FirstName))
                    .HasColumnName(nameof(Person.FirstName))
                    .HasColumnType("VARCHAR(50)");
                pd.Property<string>(nameof(Person.LastName))
                    .HasColumnName(nameof(Person.LastName))
                    .HasColumnType("VARCHAR(50)");
            }
        );
    }
}
```

Mettez à jour la méthode `OnConfiguring()` de `ApplicationDbContext` :

```cs
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
	// Les appels à l'API Fluent vont ici
	new CarConfiguration().Configure(modelBuilder.Entity<Car>());
	new RadioConfiguration().Configure(modelBuilder.Entity<Radio>());
	new DriverConfiguration().Configure(modelBuilder.Entity<Driver>());
}
```

Enfin, mettez à jour la classe `Driver` :

```cs
[Table("Drivers", Schema = "public")]
[EntityTypeConfiguration(typeof(DriverConfiguration))]
public class Driver : BaseEntity
{
	...
}
```

La table `Drivers` est mise à jour comme suit (notez que la possibilité de valeurs nulles pour les colonnes `FirstName` et `LastName` ne correspond pas aux annotations de données obligatoires de l'entité `Person`) :

```sql
CREATE TABLE "Drivers" (
    "Id" SERIAL NOT NULL,
    "TimeStamp" TIMESTAMP WITH TIME ZONE NOT NULL,
    "FirstName" VARCHAR(50) NULL,
    "LastName" VARCHAR(50) NULL,
    CONSTRAINT "PK_Drivers" PRIMARY KEY ("Id")
);
```

**Bien que la classe `Person` possède l'annotation de données obligatoires sur ses deux propriétés, les colonnes SQL Server sont toutes deux définies sur `NULL`**. ==Ceci est dû au fait que dans le fichier *.csproj*, nous ayons désactivé les type de donnée anullables.==

Pour corriger ce problème, plusieurs solutions sont possibles. La première consiste à activer les types de référence nuls en C# (au niveau du projet ou dans les classes). Cela rend la propriété de navigation `PersonInfo` non nullable, ce qu'EF Core prend en compte, et EF Core configure alors correctement les colonnes de l'entité possédée. L'autre solution consiste à ajouter une instruction Fluent API pour rendre la propriété de navigation obligatoire.

```cs
public class DriverConfiguration : IEntityTypeConfiguration<Driver>
{
	public void Configure(EntityTypeBuilder<Driver> builder)
	{
		...
		builder.Navigation(d=>d.PersonInfo).IsRequired(true);
	}
}
```

Cela met à jour les propriétés du type `Person` détenue afin qu'elles soient définies comme une colonne non nulle dans la base de données :

```sql
CREATE TABLE "Drivers" (
    "Id" SERIAL NOT NULL,
    "TimeStamp" TIMESTAMP WITH TIME ZONE NOT NULL,
    "FirstName" VARCHAR(50) NOT NULL,
    "LastName" VARCHAR(50) NOT NULL,
    CONSTRAINT "PK_Drivers" PRIMARY KEY ("Id")
);
```

**L'utilisation des types détenues présente quatre limitations :**

- Il est impossible de créer un `DbSet<T>` pour un type propriétaire.
- Il est impossible d'appeler `Entity<T>()` avec un type propriétaire dans `ModelBuilder`.
- Les instances d'un type d'entité propriétaire ne peuvent pas être partagées entre plusieurs propriétaires.
- Les types d'entités propriétaires ne peuvent pas avoir de hiérarchie d'héritage.

>[!important] Note moderne (Avec Claude)
>Les 4 limitations des Owned Entity Types restent valides en EF Core 10. Cependant, depuis EF Core 8, les **Complex Types** (`ComplexProperty()`) sont introduits comme alternative recommandée. Contrairement aux Owned Types, les Complex Types ont des sémantiques de valeur (pas d'identité), peuvent être partagés entre plusieurs propriétés, et sont mappés de la même façon (colonnes aplaties dans la table parente). En EF Core 10, `ComplexProperty()` est l'approche privilégiée pour les types sans identité comme `Address`, `Money` ou `Person`.

Il existe d'autres options à explorer avec les entités possédées, notamment les collections, le fractionnement des tables et l'imbrication. Ces fonctionnalités dépassent le cadre de cet ouvrage. Pour en savoir plus, consultez la [documentation EF Core](https://docs.microsoft.com/en-us/ef/core/modeling/owned-entities).

## Types de requêtes

**Les types de requêtes sont des collections `DbSet<T>` utilisées pour représenter des vues, une instruction SQL ou des tables sans clé primaire.** Les versions précédentes d'EF Core utilisaient `DbQuery<T>` à cet effet, mais **à partir d'EF Core 3.x, le type `DbQuery` a été abandonné.** **==Les types de requêtes sont ajoutés aux `DbContext` dérivés à l'aide de propriétés `DbSet<T>` et sont configurés comme sans clé.==**

***==Les types de requêtes sont généralement utilisés pour représenter des combinaisons de tables, comme par exemple combiner les détails des tables `Make` et `Inventory`. Prenons cet exemple de requête :==***

```sql
SELECT 
    m."Id" AS "MakeId", 
    m."Name" AS "Make",
    i."Id" AS "CarId", 
    i."IsDrivable", 
    i."Display", 
    i."DateBuilt", 
    i."Color", 
    i."PetName"
FROM "Makes" m
INNER JOIN "Inventory" i ON i."MakeId" = m."Id";
```

Pour stocker les résultats de cette requête, créez un nouveau dossier nommé `ViewModels`, et dans ce dossier, créez une nouvelle classe nommée `CarMakeViewModel`:

```cs
namespace AutoLot.Samples.ViewModels;

[Keyless]
public class CarMakeViewModel
{
    public int MakeId { get; set; }
    public string Make { get; set; }
    public int CarId { get; set; }
    public bool IsDriveable { get; set; }
    public string Display { get; set; }
    public DateTime DateBuilt { get; set; }
    public string Color { get; set; }
    public string PetName { get; set; }

    [NotMapped]
    public string FullDetail => $"The {Color} {Make} is named {PetName}";

    public override string ToString() => FullDetail;
}
```

**L'attribut `Keyless` indique à EF Core que cette entité est un type de requête et ne sera jamais utilisée pour les mises à jour.** ==Elle doit être exclue du suivi des modifications lors des requêtes.== ***==Notez l'utilisation de l'attribut `NotMapped` pour créer une chaîne d'affichage combinant plusieurs propriétés en une seule chaîne lisible.==*** Mettez à jour le contexte de base de données de l'application (`ApplicationDbContext`) pour inclure un `DbSet<T>` pour le modèle de vue :

```cs
public class ApplicationDbContext : DbContext
{
    public DbSet<Car> Cars { get; set; }
    public DbSet<Make> Makes { get; set; }
    public DbSet<Radio> Radios { get; set; }
    public DbSet<Driver> Drivers { get; set; }
    public DbSet<CarDriver> CarsToDrivers { get; set; }
    public DbSet<CarMakeViewModel> CarMakeViewModels { get; set; }

	...
}
```

Mettez à jour le fichier *GlobalUsings.cs* pour inclure le nouvel espace de noms du modèle de vue et la configuration (qui sera créée ensuite) :

```cs
global using AutoLot.Samples.ViewModels;
global using AutoLot.Samples.ViewModels.Configuration;
```

**Le reste de la configuration s'effectue dans l'API Fluent.** Créez un nouveau dossier nommé `Configuration` sous le dossier `ViewModels`, puis créez dans ce dossier une nouvelle classe nommée `CarMakeViewModelConfiguration` et mettez à jour le code comme suit :

```cs
namespace AutoLot.Samples.ViewModels.Configuration;

public class CarMakeViewModelConfiguration
    : IEntityTypeConfiguration<CarMakeViewModel>
{
    public void Configure(EntityTypeBuilder<CarMakeViewModel> builder) { }
}
```

Mettez à jour la classe `CarMakeViewModel` pour ajouter l'attribut `EntityTypeConfiguration` :

```cs
[Keyless]
[EntityTypeConfiguration(typeof(CarMakeViewModelConfiguration))]
public class CarMakeViewModel
{
	...
}
```

Mettez à jour la méthode `OnModelCreating()` comme suit :

```cs
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
	// Les appels à l'API Fluent vont ici
	new CarConfiguration().Configure(modelBuilder.Entity<Car>());
	new RadioConfiguration().Configure(modelBuilder.Entity<Radio>());
	new DriverConfiguration().Configure(modelBuilder.Entity<Driver>());
	new CarMakeViewModelConfiguration().Configure(
		modelBuilder.Entity<CarMakeViewModel>()
	);
}
```

**L'exemple suivant configure l'entité comme étant sans clé et associe le type de requête à une requête SQL brute.** **==La méthode de l'API Fluent `HasNoKey()` n'est pas nécessaire si l'annotation de données `Keyless` est présente sur le modèle, et inversement.==** Elle est toutefois présentée dans cet exemple par souci d'exhaustivité.

```cs
public void Configure(EntityTypeBuilder<CarMakeViewModel> builder)
{
	builder
		.HasNoKey()
		.ToSqlQuery(
			"""
			SELECT m."Id" AS "MakeId", 
				   m."Name" AS "Make",
				   i."Id" AS "CarId", 
				   i."IsDrivable", 
				   i."Display", 
				   i."DateBuilt", 
				   i."Color", 
				   i."PetName"
			FROM "Makes" m
			INNER JOIN "Inventory" i ON i."MakeId" = m."Id"
			"""
		);
}
```

**==Les types de requêtes peuvent également être associés à une vue de base de données. Supposons qu'il existe une vue nommée `public.CarMakeView`, la configuration serait la suivante :==**

```cs
builder.HasNoKey().ToView("CarMakeView", "public");
```

>[!note]
>Lors de l'utilisation des migrations EF Core pour mettre à jour votre base de données, les types de requêtes mappés à une vue ne sont pas créés en tant que tables. Les types de requêtes non mappés à des vues sont créés en tant que tables sans clé.

**Si vous ne souhaitez pas que le modèle de vue soit associé à une table de votre base de données et que vous n'avez pas de vue à associer, utilisez la surcharge suivante de la méthode `ToTable()` pour exclure l'élément des migrations :**

```cs
builder.ToTable( x => x.ExcludeFromMigrations());
```

**Les derniers mécanismes permettant d'utiliser les types de requêtes sont les méthodes `FromSqlRaw()` et `FromSqlInterpolated()`.** Ces méthodes seront abordées en détail dans le chapitre suivant, mais voici un aperçu :

```cs
var records = context.CarMakeViewModel.FromSqlRaw("""
    SELECT m."Id" AS "MakeId", 
           m."Name" AS "Make", 
           i."Id" AS "CarId", 
           i."IsDrivable",
           i."Display", 
           i."DateBuilt", 
           i."Color", 
           i."PetName"
    FROM "Makes" m
    INNER JOIN "Inventory" i ON i."MakeId" = m."Id"
    """);
```

### Mappage flexible des requêtes et des tables

**==EF Core 5 a introduit la possibilité d'associer une même classe à plusieurs objets de base de données.==** Ces objets peuvent être des tables, des vues ou des fonctions. Par exemple, `CarViewModel` (voir [[Chapitre 20#Créez les classes `Car` et `CarViewModel`|Chapitre 20]]) peut être associé à une vue qui renvoie la marque du véhicule ainsi que les données de la table Inventory. EF Core interrogera ensuite cette vue et enverra les mises à jour à la table.

```cs
modelBuilder.Entity<CarViewModel>()
	.ToTable("Inventory")
	.ToView("InventoryWithMakesView");
```

# Exécution des requêtes

**Les requêtes d'extraction de données sont créées à l'aide de requêtes LINQ écrites sur les propriétés `DbSet<T>`.** ***==La requête LINQ est convertie dans le langage spécifique à la base de données==*** (par exemple, T-SQL) ***==par le moteur de traduction LINQ du fournisseur de base de données et exécutée côté serveur.==*** ==Les requêtes LINQ multi-enregistrements== (ou potentiellement multi-enregistrements) ==ne sont exécutées que lorsqu'elles sont parcourues== (par exemple, à l'aide d'une boucle `foreach`) ==ou liées à un contrôle pour affichage== (comme une grille de données). **Cette exécution différée permet de construire des requêtes dans le code sans subir de problèmes de performance liés à des échanges excessifs avec la base de données ou à la récupération d'un nombre d'enregistrements supérieur à celui prévu.**

**Par exemple, pour obtenir tous les enregistrements de `Car` jaunes dans la base de données, exécutez la requête suivante** (Dans le fichier *Program.cs*) :

```cs
static void QueryExecution()
{
    //Cette fabrique n'est pas censée être utilisée ainsi,
    //mais il s'agit d'un exemple de code :-)
    var context = new ApplicationDbContextFactory().CreateDbContext(null);
    var cars = context.Cars.Where(x => x.Color == "Yellow");
}
```

**En mode d'exécution différée, la base de données n'est interrogée qu'après le parcours des résultats.** **==Pour que la requête s'exécute immédiatement, utilisez la méthode `ToList()`.==**

```cs
var listOfCars = context.Cars.Where(x => x.Color == "Yellow").ToList();
```

**Comme les requêtes ne sont exécutées que lorsqu'elles sont déclenchées, elles peuvent être construites sur plusieurs lignes de code.** L'exemple de code suivant s'exécute de la même manière que l'exemple précédent :

```cs
var query = context.Cars.AsQueryable();
query = query.Where(x => x.Color == "Yellow");
var moreCars = query.ToList();
```

***==Les requêtes portant sur un seul enregistrement==*** (comme lors de l'utilisation de `First()`/`FirstOrDefault()`) ***==s'exécutent immédiatement lors de l'appel de l'action==*** (telle que `FirstOrDefault()`), ***==et les instructions de création, de mise à jour et de suppression sont exécutées immédiatement lorsque la méthode `DbContext.SaveChanges()` est exécutée.==***

>[!note]
>Les chapitres suivants traitent en détail de l'exécution des opérations CRUD.

## Évaluation mixte client-serveur

Les versions précédentes d'EF Core permettaient de combiner l'exécution côté serveur et côté client. Cela signifiait qu'une fonction C# pouvait être utilisée au milieu d'une requête LINQ, ce qui annulait en réalité le comportement décrit précédemment. La partie de la requête précédant la fonction C# s'exécutait côté serveur, mais les résultats (à ce stade de la requête) étaient ensuite renvoyés côté client, et le reste de la requête s'exécutait en tant que requête LINQ to Objects. Cette approche a finalement engendré plus de problèmes qu'elle n'en a résolu, et cette fonctionnalité a été modifiée avec la sortie d'EF Core 3.1. Désormais, seul le dernier nœud d'une requête LINQ peut s'exécuter côté client.

# Requêtes avec et sans suivi des modifications

**Lorsque des données sont lues depuis la base de données dans une instance de `DbSet<T>` avec une clé primaire, les entités (par défaut) sont suivies par le gestionnaire de modifications.** **==C'est généralement le comportement souhaité dans votre application.==** ==Toute modification apportée à l'élément peut alors être enregistrée dans la base de données simplement en appelant `SaveChanges()` sur votre instance de `DbContext` dérivée, sans aucune intervention supplémentaire de votre part.== De plus, **une fois qu'une instance est suivie par le gestionnaire de modifications, tout appel ultérieur à la base de données pour ce même élément (basé sur la clé primaire) entraînera une mise à jour de l'élément, et non une duplication.**

***==Cependant, il peut arriver que vous ayez besoin d'extraire des données de la base de données sans souhaiter qu'elles soient suivies par le gestionnaire de modifications.==*** ==Cela peut être dû à des problèmes de performance (le suivi des valeurs d'origine et actuelles pour un grand nombre d'enregistrements peut engendrer une forte consommation de mémoire), ou bien vous savez que ces enregistrements ne seront jamais modifiés par la partie de l'application qui a besoin de ces données.==

**Pour charger des données dans une instance de `DbSet<T>` sans les ajouter au ChangeTracker, ajoutez `AsNoTracking()` à l'instruction LINQ**. **==Cela indique à EF Core de récupérer les données sans les ajouter au `ChangeTracker`.==** Par exemple, pour charger un enregistrement de type `Car` sans l'ajouter au `ChangeTracker`, exécutez le code suivant :

```cs
static void TrackingNoTracking()
{
    //Cette fabrique n'est pas censée être utilisée ainsi,
    // mais c'est un code de démonstration :-)
    var context = new ApplicationDbContextFactory().CreateDbContext(null);
    var untrackedCar = context.Cars.Where(x => x.Id == 1).AsNoTracking();
}
```

*==Cela présente l'avantage d'éviter une potentielle surcharge mémoire, mais aussi un inconvénient : des appels supplémentaires pour récupérer la même voiture créeront des copies supplémentaires de l'enregistrement.==* ***==Au prix d'une consommation de mémoire accrue et d'un temps d'exécution légèrement plus long, la requête peut être modifiée afin de garantir qu'il n'existe qu'une seule instance de la voiture non mappée.==***

```cs
static void TrackingNoTracking()
{
	...
    var untrackedWithIdResolution = context
        .Cars.Where(x => x.Id == 1)
        .AsNoTrackingWithIdentityResolution();
}
```

**Les types de requêtes ne sont jamais suivis car ils ne peuvent pas être mis à jour.** *==L'exception concerne le mappage flexible requête/table.==* ==Dans ce cas, les instances sont suivies par défaut afin d'être enregistrées dans la table cible.==

# Code First vs. Database First

***==Que vous développiez une nouvelle application ou que vous ajoutiez EF Core à une application existante, vous vous retrouverez dans l'une des deux situations suivantes :==*** ***soit vous disposez d'une base de données existante avec laquelle vous devez travailler, soit vous n'en avez pas encore et vous devez en créer une.***

**==L'approche *Code First* consiste à créer et configurer vos classes d'entités et le `DbContext` dérivé directement dans le code, puis à utiliser des migrations pour mettre à jour la base de données.==** **C'est ainsi que la plupart des nouveaux projets sont développés.** ==L'avantage est que, à mesure que vous développez votre application, vos entités évoluent en fonction de ses besoins. Les migrations maintiennent la base de données synchronisée, de sorte que sa conception évolue avec votre application.== **Ce processus de conception émergent est populaire auprès des équipes de développement agiles, car il permet de construire les bonnes parties au bon moment.**

**==Si vous disposez déjà d'une base de données ou si vous préférez que la conception de votre base de données guide votre application, on parle alors d'approche *Database First*.==** **Au lieu de créer manuellement le `DbContext` dérivé et toutes les entités, vous générez les classes à partir de la base de données.** ***==Lorsque la base de données est modifiée, vous devez recréer la structure de vos classes pour maintenir votre code synchronisé avec la base de données.==*** **Tout code personnalisé dans les entités ou le `DbContext` dérivé doit être placé dans des classes partielles afin qu'il ne soit pas écrasé lors de la recréation de la structure.** **==Heureusement, le processus de génération de code crée des classes partielles précisément pour cette raison.==**

***==Quelle que soit la méthode choisie, "code first" ou "database first", sachez qu'il s'agit d'un engagement.==*** 

- **Si vous utilisez « code first », toutes les modifications sont apportées aux classes d'entités et de contexte, et la base de données est mise à jour à l'aide de migrations.**

- **Si vous travaillez avec « database first », toutes les modifications doivent être apportées à la base de données, puis les classes sont recréées.** 

***==Avec un peu d'effort et de planification, vous pouvez passer de « database first » à « code first » (et inversement), mais vous ne devez pas effectuer de modifications manuelles simultanément dans le code et la base de données.==***

# Commandes CLI de l'outil global EF Core

**L'outil global `dotnet-ef` (outil EF Core) contient les commandes nécessaires pour générer du code à partir de bases de données existantes, pour créer/supprimer des migrations de bases de données et pour effectuer des opérations sur une base de données (mise à jour, suppression, etc.).** Avant de pouvoir utiliser l'outil global `dotnet-ef`, vous devez l'installer à l'aide de la commande suivante :

```bash
dotnet tool install --global dotnet-ef
```

>[!note]
>Si vous avez installé une version antérieure des outils en ligne de commande EF Core, vous devrez la désinstaller avant d'installer la dernière version. Pour désinstaller l'outil global, utilisez :
>```bash
>dotnet tool uninstall --global dotnet-ef.
>```

Pour tester l'installation, ouvrez une invite de commandes et saisissez la commande suivante :

```bash
dotnet ef
```

***==Si l'outil est installé avec succès, vous obtiendrez la licorne d'EF Core (la mascotte de l'équipe) et la liste des commandes disponibles, comme ceci :==***

```
                     _/\__
               ---==/    \\
         ___  ___   |.    \|\
        | __|| __|  |  )   \\\
        | _| | _|   \_/ |  //|\\
        |___||_|       /   \\\/\\

Entity Framework Core .NET Command-line Tools 10.0.10

Usage: dotnet ef [options] [command]

Options:
  --version        Show version information
  -h|--help        Show help information
  -v|--verbose     Show verbose output.
  --no-color       Don't colorize output.
  --prefix-output  Prefix output with level.

Commands:
  database    Commands to manage the database.
  dbcontext   Commands to manage DbContext types.
  migrations  Commands to manage migrations.

Use "dotnet ef [command] --help" for more information about a command.
```

Le [[#Tableau 21-10 Commandes d'outillage EF Core|Tableau 21-10]] décrit les trois commandes principales de l'outil global EF Core. **Chaque commande principale possède des sous-commandes.** **==Comme pour toutes les commandes .NET, chaque commande dispose d'un système d'aide complet accessible en saisissant l'option `-h` avec la commande.==**

###### Tableau 21-10: Commandes d'outillage EF Core

| Commande     | Description                                                                                                    |
| ------------ | -------------------------------------------------------------------------------------------------------------- |
| `Database`   | Commandes permettant de gérer la base de données. Les sous-commandes incluent `drop` et `update`               |
| `DbContext`  | Commandes permettant de gérer les types `DbContext`. Les sous-commandes incluent `scaffold`, `list` et `info`. |
| `Migrations` | Commandes de gestion des migrations. Les sous-commandes incluent `add`, `list`, `remove` et `script`.          |

***Les commandes EF Core s'exécutent sur les fichiers de projet .NET.*** **Le projet cible doit référencer le package NuGet d'outils EF Core :** `Microsoft.EntityFrameworkCore.Design`. ***==Les commandes agissent sur le fichier projet situé dans le même répertoire que celui où elles sont exécutées, ou sur un fichier projet situé dans un autre répertoire s'il est référencé via les options de ligne de commande.==***

**Pour les commandes CLI EF Core nécessitant une instance d'une classe `DbContext` dérivée** (`Database` et `migrations`), **si une seule instance est présente dans le projet, elle sera utilisée.** **==S'il y en a plusieurs, le `DbContext` doit être spécifié dans les options de ligne de commande.==** ==La classe `DbContext` dérivée sera instanciée à l'aide d'une instance d'une classe implémentant l'interface `IDesignTimeDbContextFactory<TContext>`, si elle est disponible.== **Si l'outil ne trouve pas d'instance, le `DbContext` dérivé sera instancié à l'aide du constructeur sans paramètre. Si aucune de ces instances n'existe, la commande échouera.** ***==Notez que l'utilisation du constructeur sans paramètre (et non du constructeur prenant en paramètre `DbContextOptions<T>`) requiert l'existence d'une surcharge de la méthode `OnConfiguring`, ce qui est déconseillé.==*** **==La meilleure (et en réalité la seule) solution consiste à toujours créer une instance de `IDesignTimeDbContextFactory<TContext>` pour chaque `DbContext` dérivé de votre application.==**

**Des options communes sont disponibles pour les commandes EF Core, comme indiqué dans le [[#Tableau 21-11 Options de commande EF Core|Tableau 21-11]].** De nombreuses commandes possèdent des options ou arguments supplémentaires.

###### Tableau 21-11: Options de commande EF Core

| Option (Version courte) | Option (Version longue) | Type de valeur attendu | Description                                                                                                                                                       |
| ----------------------- | ----------------------- | ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `-c`                    | `--context`             | `<DBCONTEXT>`          | Classe `DbContext` dérivée entièrement qualifiée à utiliser. **Si plusieurs classes `DbContext` dérivées existent dans le projet, cette option est obligatoire.** |
| `-p`                    | `--project`             | `<PROJECT>`            | Le projet à utiliser (emplacement des fichiers). Par défaut, le répertoire de travail courant.                                                                    |
| `-s`                    | `--startup-project`     | `<PROJECT>`            | Le projet de démarrage à utiliser (contient le `DbContext` dérivé). Par défaut, il s'agit du répertoire de travail courant.                                       |
| `-h`                    | `--help`                |                        | Affiche l'aide et toutes les options.                                                                                                                             |
| `-v`                    | `--verbose`             |                        | Affiche tout les détails.                                                                                                                                         |

Pour afficher tous les arguments et options d'une commande, saisissez `dotnet ef <commande> -h` dans une fenêtre de commande, comme ceci :

```bash
dotnet ef migrations add -h
```

>[!note]
>Il est important de noter que les commandes CLI ne sont pas des commandes C#, donc les règles d'échappement des barres obliques et des guillemets ne s'appliquent pas.

## Les commandes de `migrations`

**Les commandes de migration permettent d'ajouter, de supprimer, de lister et de générer des scripts de migration.** ***==Lorsqu'une migration est appliquée à une base, un enregistrement est créé dans la table `__EFMigrationsHistory`.==*** Le [[#Tableau 21-12 Commandes de `migrations` EF Core|Tableau 21-12]] décrit ces commandes. Les sections suivantes expliquent les commandes en détail.

>[!success] Meilleure analogie pour les migrations (Avec Claude)
>
`dotnet ef migrations` est l'équivalent de *Git* mais pour le schéma de base de données.
>
>**Comme Git**, les migrations sont linéaires et chronologiques, on ne peut pas supprimer une migration du milieu sans défaire celles qui suivent. 
>
**Contrairement à Git**, il n'y a pas de branches ni de merge, et l'historique est stocké à la fois dans les fichiers C# du projet ET dans la table `__EFMigrationsHistory` de la base de données.

###### Tableau 21-12: Commandes de `migrations` EF Core

| Commande | Description                                                                                                                                                                                                                      |
| -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Add`    | Crée une nouvelle migration basée sur les modifications apportées par la migration précédente.                                                                                                                                   |
| `Remove` | Vérifie si la dernière migration du projet a été appliquée à la base de données et, si ce n'est pas le cas, supprime le fichier de migration (et son concepteur) puis rétablit la classe d'instantané à la migration précédente. |
| `list`   | Liste toutes les migrations pour un `DbContext` dérivé et leur statut (appliquées ou en attente).                                                                                                                                |
| `bundle` | Crée un fichier exécutable pour mettre à jour la base de données.                                                                                                                                                                |
| `script` | Crée un script SQL pour toutes les migrations, une seule ou une plage de migrations.                                                                                                                                             |

### La commande `add` de `migrations`

**La commande `add` crée une nouvelle migration de base de données basée sur le modèle objet actuel.** **==Le processus examine chaque entité possédant une propriété `DbSet<T>` sur le `DbContext` dérivé (et chaque entité accessible depuis ces entités via les propriétés de navigation) et détermine si des modifications doivent être appliquées à la base de données.==** **Le cas échéant, le code approprié est généré pour mettre à jour la base de données. Vous en apprendrez davantage à ce sujet prochainement.**

**La commande `add` requiert un argument `name`, utilisé pour nommer la classe et les fichiers de la migration.** ==Outre les options communes, l'option `-o <CHEMIN>` ou `--output-dir <CHEMIN>` indique l'emplacement où les fichiers de migration doivent être enregistrés. Le répertoire par défaut est nommé `Migrations`, relatif au chemin actuel.==

**Chaque migration ajoutée crée deux fichiers partiels de la même classe.** ==Le nom de ces deux fichiers commence par un horodatage et le nom de la migration utilisé comme argument de la commande `add`== **Le premier fichier se nomme** *`<YYYYMMDDHHMMSS>_<NomMIgration>`.cs*, **et le second** *`<YYYYMMDDHHMMSS>_<NomMigration>`.Designer.cs*. ***==L'horodatage correspond à la date de création du fichier et est identique pour les deux.==*** **Le premier fichier contient le code généré pour les modifications de la base de données lors de *cette* migration, tandis que le fichier "designer" contient le code permettant de créer et de mettre à jour la base de données en fonction de toutes les migrations effectuées jusqu'à celle-ci incluse.**

**Le fichier principal contient deux méthodes, `Up()` et `Down()`.** ***==La méthode `Up()` contient le code permettant de mettre à jour la base de données avec les modifications de cette migration, et la méthode `Down()` contient le code permettant d'annuler ces modifications.==*** Un extrait de la migration initiale présentée précédemment dans ce chapitre est fourni ci-dessous (toutes les migrations utilisées dans les exemples précédents se trouvent dans le projet *AutoLot.Samples*, dans le code associé) :

> Les exemple de code sont ceux de l'auteur car, comme cette explication vient seulement à la fin du chapitre, je n'ai pas su exécuter les migrations.

```cs
using System;
using Microsoft.EntityFrameworkCore.Migrations;

namespace AutoLot.Samples.Migrations
{
    public partial class Initial : Migration
    {
        protected override void Up(MigrationBuilder migrationBuilder)
        {
            migrationBuilder.CreateTable(
                name: "Makes",
                columns: table => new
                {
                    Id = table.Column<int>(type: "int", nullable: false)
                        .Annotation("SqlServer:Identity", "1, 1"),
                    Name = table.Column<string>(type: "nvarchar(max)", nullable: true),
                    TimeStamp = table.Column<byte[]>(type: "varbinary(max)", nullable: true)
                },
                constraints: table =>
                {
                    table.PrimaryKey("PK_Makes", x => x.Id);
                });

            migrationBuilder.CreateTable(
                name: "Cars",
                columns: table => new
                {
                    Id = table.Column<int>(type: "int", nullable: false)
                        .Annotation("SqlServer:Identity", "1, 1"),
                    Color = table.Column<string>(type: "nvarchar(max)", nullable: true),
                    PetName = table.Column<string>(type: "nvarchar(max)", nullable: true),
                    MakeId = table.Column<int>(type: "int", nullable: false),
                    TimeStamp = table.Column<byte[]>(type: "varbinary(max)", nullable: true)
                },
                constraints: table =>
                {
                    table.PrimaryKey("PK_Cars", x => x.Id);
                    table.ForeignKey(
                        name: "FK_Cars_Makes_MakeId",
                        column: x => x.MakeId,
                        principalTable: "Makes",
                        principalColumn: "Id",
                        onDelete: ReferentialAction.Cascade);
                });

            migrationBuilder.CreateIndex(
                name: "IX_Cars_MakeId",
                table: "Cars",
                column: "MakeId");
        }

        protected override void Down(MigrationBuilder migrationBuilder)
        {
            migrationBuilder.DropTable(
                name: "Cars");

            migrationBuilder.DropTable(
                name: "Makes");
        }
    }
}
```

==Comme vous pouvez le constater, la méthode `Up()` crée des tables, des colonnes, des index, etc. La méthode `Down()` supprime les éléments créés. Le moteur de migrations exécutera les instructions `ALTER`, `ADD` et `DROP` nécessaires pour que la base de données corresponde à votre modèle.==

**Le fichier de conception contient deux attributs qui lient ces modèles partiels au nom de fichier et au `DbContext` dérivé. Ces attributs sont présentés ici, accompagnés d'une liste partielle de la classe de conception :**

```cs
[DbContext(typeof(ApplicationDbContext))]
[Migration("20210801173031_Initial")]
partial class Initial
{
	protected override void BuildTargetModel(ModelBuilder modelBuilder)
	{
		// Omis pour brièvete'
	}
}
```

***==La première migration crée un fichier supplémentaire dans le répertoire cible==***, nommé d'après le `DbContext` dérivé, au format *`<NomDuDbContextDérivé>`ModelSnapshot.cs*. **Le format de ce fichier est identique à celui de l'aperçu du concepteur et contient le code résultant de toutes les migrations. Lorsque des migrations sont ajoutées ou supprimées, ce fichier est automatiquement mis à jour pour refléter les modifications.**

>[!note]
Il est extrêmement important de ne pas supprimer manuellement les fichiers de migration. Cela entraînerait une désynchronisation du fichier *`<NomDbContextDérivé>`ModelSnapshot.cs* avec vos migrations, les rendant ainsi inutilisables. Si vous décidez malgré tout de les supprimer manuellement, supprimez-les tous et recommencez. Pour supprimer une migration, utilisez la commande `remove`, que nous aborderons prochainement.

### La commande `remove` de `migrations`

**La commande `remove` permet de supprimer des migrations du projet et agit toujours sur la dernière migration appliquée (en fonction de leur horodatage).** ***==Lors de la suppression d'une migration, EF Core vérifie qu'elle n'a pas été appliquée en consultant la table `__EFMigrationsHistory` de la base de données.==*** *==Si la migration a été appliquée, le processus échoue.==* **==Si la migration n'a pas encore été appliquée ou a été annulée, elle est supprimée et le fichier d'instantané du modèle est mis à jour.==**

**La commande `remove` ne prend aucun argument (puisqu'elle agit toujours sur la dernière migration) et utilise les mêmes options que la commande `add`.** ***==Une option supplémentaire est disponible : `force` (`-f` ou `--force`). Cette option annule la dernière migration puis la supprime en une seule étape.==***

### La commande `list` de `migrations`

**La commande `list` permet d'afficher toutes les migrations d'un `DbContext` dérivé.** ***==Par défaut, elle liste toutes les migrations et interroge la base de données pour déterminer si elles ont été appliquées.==*** ==Si elles n'ont pas été appliquées, elles sont indiquées comme étant en attente.== **Une option pour spécifier une chaîne de connexion et une autre pour ne pas se connecter à la base de données, mais simplement de lister les migrations.** Le [[#Tableau 21-13 Options supplémentaires pour la commande `list` de `migrations` EF Core|Tableau 21-13]] présente ces options.

###### Tableau 21-13: Options supplémentaires pour la commande `list` de `migrations` EF Core

| Option (Seulement version longue) | Argument       | Description                                                                                                                                                               |
| --------------------------------- | -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `--connection`                    | `<CONNECTION>` | Chaîne de connexion à la base de données. Par défaut, celle spécifiée dans l’instance de `IDesignTimeDbContextFactory` ou dans la méthode `OnConfiguring` du `DbContext`. |
| `--no-connect`                    |                | Indique à la commande d'ignorer la vérification de la base de données.                                                                                                    |

### La commande `bundle` de `migrations`

**La commande `bundle` crée un exécutable pour mettre à jour la base de données.** ***==Cet exécutable, conçu pour un environnement d'exécution cible (par exemple, Windows ou Linux), appliquera toutes les migrations qu'il contient à la base de données.==*** Le [[#Tableau 21-14 Arguments courants de la commande `bundle` de `migrations` EF Core|Tableau 21-14]] décrit les arguments les plus fréquemment utilisés avec la commande bundle.

###### Tableau 21-14: Arguments courants de la commande `bundle` de `migrations` EF Core

| Option (Courte) | Option (Longue)    | Argument       | Description                                                                                                                                                                                           |
| --------------- | ------------------ | -------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `-o`            | `--output`         | `<FILE>`       | Le chemin d'accès pour l'exécutable à créer                                                                                                                                                           |
| `-f`            | `--force`          |                | Écrase les fichiers existants.                                                                                                                                                                        |
|                 | `--self-contained` |                | Inclue également le runtime .NET avec l'exécutable.                                                                                                                                                   |
| `-r`            | `--target-runtime` | `<RUNTIME ID>` | Environnement d'exécution cible pour lequel l'exécutable sera généré. Si aucun environnement d'exécution n'est spécifié, le fichier utilisera celui du système d'exploitation de la machine actuelle. |

**L'exécutable utilisera la chaîne de connexion fournie par `IDesignTimeDbContextFactory` ; toutefois, une autre chaîne de connexion peut être transmise à l'exécutable à l'aide de l'option `--connection`. Si les migrations ont déjà été appliquées à la base de données cible, elles ne seront pas réappliquées.**

==Lorsque l'option `--self-contained` est utilisée, la taille de l'exécutable augmente considérablement.== Sur ma machine, avec l'exemple de projet de ce chapitre, le fichier d'installation standard pèse 11 Mo, tandis que le fichier autonome pèse 74 Mo.

### La commande `script` de `migrations`

**La commande `script` crée un script SQL basé sur une ou plusieurs migrations.** ***==Elle accepte deux arguments optionnels : la migration de départ et la migration d'arrivée.==*** **Si aucun argument n'est spécifié, toutes les migrations sont générées.** Le [[#Tableau 21-15 Arguments de la commande `script` de `migrations` EF Core|Tableau 21-15]] décrit les arguments.

###### Tableau 21-15: Arguments  de la commande `script` de `migrations` EF Core

| Argument | Description                                         |
| -------- | --------------------------------------------------- |
| `<FROM>` | Migration initiale. Par défaut : 0 (zéro).          |
| `<TO>`   | Migration cible. Par défaut, la dernière migration. |

**Si aucune migration n'est spécifiée, le script créé correspondra au total cumulatif de toutes les migrations. Si des migrations sont spécifiées, le script contiendra les modifications entre les deux migrations (incluses).** ***==Chaque migration est encapsulée dans une transaction.==*** **==Si la table `__EFMigrationsHistory` n'existe pas dans la base de données où le script est exécuté, elle sera créée. La table sera également mise à jour pour refléter les migrations exécutées.==** Quelques exemples sont présentés ici :

```bash
# Script de toutes les migrations
dotnet ef migrations script
# Script du début jusqu'aux migrations Many2Many
dotnet ef migrations script 0 Many2Many
```

Des options supplémentaires sont disponibles, comme indiqué dans le [[#Tableau 21-16 Options supplémentaires pour la commande `script` de `migrations` EF Core|Tableau 21-16]]. L'option `-o` permet de spécifier un fichier pour le script (le répertoire est relatif à l'emplacement d'exécution de la commande), et l'option `-i` crée un script idempotent. Cela signifie qu'il contient des vérifications pour déterminer si une migration a déjà été appliquée et l'ignore le cas échéant. L'option `--no-transaction` désactive les transactions normales ajoutées au script.

###### Tableau 21-16: Options supplémentaires pour la commande `script` de `migrations` EF Core

| Option (Court) | Option (Longue)    | Argument | Description                                                                              |
| -------------- | ------------------ | -------- | ---------------------------------------------------------------------------------------- |
| `-o`           | `--output`         | `<FILE>` | Le fichier dans lequel écrire le script résultant                                        |
| `-i`           | `--idempotent`     |          | Génère un script qui vérifie si une migration a déjà été appliquée avant de l'appliquer. |
|                | `--no-transaction` |          | N'encapsule pas chaque migration dans une transaction                                    |



## Les commandes de `database`

**Il existe deux commandes de base de données : `drop` et `update`**. **==La commande `drop` supprime la base de données si elle existe. La commande `update` met à jour la base de données à l'aide des migrations.==**

### La commande `drop` de `database`

**La commande `drop` supprime la base de données spécifiée par la chaîne de connexion dans la fabrique de contexte de la méthode `OnConfiguring` de `DbContext`.** ==L'option `force` force la fermeture de toutes les connexions sans confirmation.== Voir le [[#Tableau 21-17 Options de la commande `drop` de `database` EF Core|Tableau 21-17]].

###### Tableau 21-17: Options de la commande `drop` de `database` EF Core
| Option (Courte) | Option (Longue) | Description                                                                  |
| --------------- | --------------- | ---------------------------------------------------------------------------- |
| `-f`            | `--force`       | Ne confirme pas la déconnexion. Force la fermeture de toutes les connexions. |
|                 | `--dry-run`     | Indique quelle base de données sera supprimée, mais ne la supprime pas.      |

### La commande `udpate` de `database`

**La commande `update` prend un argument (le nom de la migration) et les options habituelles.** ***==Elle possède une option supplémentaire : `--connection <CONNEXION>`.==*** **==Celle-ci permet d’utiliser une chaîne de connexion non configurée dans la fabrique de conception ou le `DbContext`==**.

***==Si la commande est exécutée sans nom de migration, elle met à jour la base de données avec la migration la plus récente, en créant la base de données si nécessaire.==*** ==Si une migration est spécifiée, la base de données sera mise à jour avec cette migration.== **Toutes les migrations précédentes non encore appliquées seront également appliquées.** ==Au fur et à mesure de leur application, les noms des migrations sont stockés dans la table `__EFMigrationsHistory`.==

**Si l’horodatage de la migration spécifiée est antérieur à celui des autres migrations appliquées, toutes les migrations ultérieures sont annulées.** ***==Si la valeur $0$ (zéro) est fournie comme nom de migration, toutes les migrations sont annulées, laissant une base de données vide (à l’exception de la table `__EFMigrationsHistory`).==***

>[!success] Note moderne
>**Dans EF Core 11** (==pas encore sorti au moment de la rédaction==), la commande suivante permet de créer **ET** appliquer une migration **en une seule étape** via *Rolsyn*
>
>```bash
dotnet ef database update AddCheckConstraint --add -c AutoLot.Samples.ApplicationDbContext
>```

## Les commandes de `dbcontext`

**Il existe quatre commandes `dbcontext`. Trois d'entre elles (`list`, `info`, `script`) agissent sur les classes `DbContext` dérivées de votre projet.** **==La commande `scaffold` crée un `DbContext` dérivé et des entités à partir d'une base de données existante.==** Le [[#Tableau 21-18 les commandes de `dbcontext`|Tableau 21-18]] présente les commandes disponibles.

###### Tableau 21-18: les commandes de `dbcontext`

| Commande   | Description                                                                                               |
| ---------- | --------------------------------------------------------------------------------------------------------- |
| `info`     | Obtient des informations sur un type `DbContext`                                                          |
| `list`     | Liste les types `DbContext` disponibles                                                                   |
| `optimize` | Génère une version compilée du modèle utilisé par le `DbContext`                                          |
| `scaffold` | Génère un `DbContext` et des types d'entités pour une base de données.                                    |
| `script`   | Génère un script SQL à partir du `DbContext` en fonction du modèle objet, en contournant toute migration. |

**Les commandes `list` et `info` offrent les options habituelles.** ==La commande `list` liste les classes `DbContext` dérivées du projet cible.== **==La commande `info` fournit des détails sur la classe `DbContext` dérivée spécifiée, notamment la chaîne de connexion, le nom du fournisseur, le nom de la base de données et la source de données.==** **La commande `script` crée un script SQL qui crée votre base de données à partir du modèle objet, en ignorant les migrations éventuelles.** ==La commande `scaffold` permet de reconstituer une base de données existante et est abordée dans la section suivante.==

### La commande `scaffold` de `dbcontext`

**La commande `scaffold` crée les classes C# (`DbContext` dérivé et entités) avec les annotations de données (si demandées) et les commandes de l'API Fluent à partir d'une base de données existante.** ***==Deux arguments sont requis : la chaîne de connexion à la base de données et le fournisseur complet (par exemple, `Microsoft.EntityFrameworkCore.SqlServer`)==***. Le [[#Tableau 21-19 Les arguments de la commande `scaffhold` de `dbcontext`|Tableau 21-19]] décrit ces arguments.

###### Tableau 21-19: Les arguments de la commande `scaffhold` de `dbcontext`

| Argument     | Description                                                                                                   |
| ------------ | ------------------------------------------------------------------------------------------------------------- |
| `Connection` | La chaîne de connexion à la base de données                                                                   |
| `Provider`   | Le fournisseur de base de données EF Core à utiliser (par exemple, `Microsoft.EntityFrameworkCore.SqlServer`) |

==Les options disponibles comprennent la sélection de schémas et de tables spécifiques, le nom et l'espace de noms de la classe de contexte créée, le répertoire et l'espace de noms de sortie des classes d'entités générées, et bien d'autres.== **Les options standard sont également disponibles.** Les options étendues sont répertoriées dans le [[#Tableau 21-20 et seront détaillées ultérieurement.

###### Tableau 21-20: Les options de la commande `scaffhold` de `dbcontext`

| Option (Courte)<br><br> | Option (Longue)        | Argument           | Description                                                                                                                         |
| ----------------------- | ---------------------- | ------------------ | ----------------------------------------------------------------------------------------------------------------------------------- |
| `-d`                    | `--data-annotations`   |                    | Utilisez les attributs pour configurer le modèle (lorsque cela est possible). À défaut, seule l'API Fluent sera utilisée.           |
| `-c`                    | `--context`            | `<NAME>`           | Le nom du `DbContext` dérivé à créer.                                                                                               |
|                         | `--context-dir`        | `<PATH>`           | Répertoire dans lequel placer le `DbContext` dérivé, par rapport au répertoire du projet. Par défaut, le nom de la base de données. |
| `-f`                    | `--force`              |                    | Remplace tous les fichiers existants dans le répertoire cible.                                                                      |
| `-o`                    | `--output-dir`         | `<PATH>`           | Répertoire dans lequel placer les classes d'entités générées. Relatif au répertoire du projet.                                      |
|                         | `--schema`             | `<SCHEMA_NAME>`... | Les schémas des tables pour lesquelles générer des types d'entités.                                                                 |
| `-t`                    | `--table`              | `<TABLE_NAME>`...  | Les tables pour lesquelles générer des types d'entités.                                                                             |
|                         | `--use-database-names` |                    | Utilisez directement les noms de tables et de colonnes issus de la base de données.                                                 |
| `-n`                    | `--namespaces`         | `<NAMESPACE>`      | Espace de noms des classes d'entités générées. Correspond par défaut au répertoire.                                                 |
| <br><br>                | `--context-namespace`  | `<NAMESPACE>`      | Espace de noms de la classe dérivée `DbContext` générée. Correspond par défaut au répertoire.                                       |
|                         | `--no-onconfiguring`   |                    | Ne génère pas la méthode `OnConfiguring`.                                                                                           |
|                         | `--no-pluralize`       |                    | désactive la pluralisation automatique des nom                                                                                      |

**La commande `scaffold` est devenue beaucoup plus robuste avec EF Core 6.0.** Comme vous pouvez le constater, ==de nombreuses options sont disponibles.== **==Si l'option annotations de données (`-d`) est sélectionnée, EF Core utilisera les annotations de données lorsque possible et comblera les différences avec l'API Fluent==**. ***==Si cette option n'est pas sélectionnée, toute la configuration (lorsqu'elle diffère des conventions) est codée dans l'API Fluent.==*** ==Vous pouvez spécifier l'espace de noms, le schéma et l'emplacement des entités générées et des fichiers `DbContext` dérivés.== **Si vous ne souhaitez pas générer l'intégralité de la base de données, vous pouvez sélectionner certains schémas et tables.** ***==L'option `--no-onconfiguring` supprime la méthode `OnConfiguring()` de la classe générée, et l'option `--no-pluralize` désactivele pluraliseur, qui transforme les entités singulières (`Car`) en tables plurielles (`Cars`) lors de la création de migrations et transforme les tables plurielles en entités uniques lors de la génération de code.==***

**Nouveauté d'EF Core 6 : les commentaires de base de données sur les tables et colonnes SQL sont également intégrés aux classes d'entités et à leurs propriétés.**

### La commande `optimize` de `dbcontext`

**La commande `optimize` optimise le `DbContext` dérivé en exécutant la plupart des étapes qui se produisent normalement lors de sa première utilisation.** ==Les options disponibles incluent la spécification du répertoire de destination des résultats compilés ainsi que de l'espace de noms à utiliser.== **Les options standard sont également disponibles.** Les options étendues sont listées dans le [[#Tableau 21-21 Les options de la commande `optimize` de `dbcontext`|Tableau 21-21]] et seront commentées ci-après.

###### Tableau 21-21: Les options de la commande `optimize` de `dbcontext`

| Option (Courte) | Option (Longue) | Argument      | Description                                                                                       |
| --------------- | --------------- | ------------- | ------------------------------------------------------------------------------------------------- |
| `-o`            | `--output-dir`  |               | Le répertoire dans lequel placer les fichiers. Les chemins sont relatifs au répertoire du projet. |
| `-n`            | `--namespace`   | `<NAMESPACE>` | L'espace de noms à utiliser. Correspond par défaut au répertoire.                                 |

**Lors de la compilation du `DbContext` dérivé, les résultats incluent une classe pour chaque entité de votre modèle, le `DbContext` dérivé compilé et le `ModelBuilder` du `DbContext` dérivé compilé.** ***==Par exemple, vous pouvez compiler `AutoLot.Samples.ApplicationDbContext` à l'aide de la commande suivante :==***

```bash
dotnet ef dbcontextptimize --output-dir CompiledModels
```

Les fichiers compilés sont placés dans un répertoire nommé `CompiledModels`. La liste des fichiers est disponible ici :

- *AppllicationDbContextAssemblyAttributes.cs*
- *ApplicationDbContextModel.cs*
- *ApplicationDbContextModelBuilder.cs*
- *CarDriverEntityType.cs*
- *CarEntityType.cs*
- *CarMakeViewModelEntityType.cs*
- *DriverEntityType.cs*
- *MakeEntityType.cs*
- *PersonEntityType.cs*
- *RadioEntityType.cs*

Pour utiliser le modèle compilé, appelez la méthode `UseModel()` dans `DbContextOptions`, comme ceci :

```cs
static void UseCompiledDbContext()
{
	
    DotNetEnv.Env.TraversePath().Load();

    var config = new ConfigurationBuilder().AddEnvironmentVariables().Build();

    var optionBuilder = new DbContextOptionsBuilder<ApplicationDbContext>();
    var conStringBuilder = new NpgsqlConnectionStringBuilder
    {
        Host = config["Postgres:Host"],
        Username = config["Postgres:Username"],
        Database = config["Postgres:Database"],
        Password = config["Postgres:Password"],
        Pooling = true, // Recommandé pour PostgreSQL
    };

    optionBuilder
        .UseNpgsql(conStringBuilder.ConnectionString)
        .UseModel(ApplicationDbContextModel.Instance);
    var context = new ApplicationDbContext(optionBuilder.Options);
}
```

**La compilation du `DbContext` dérivé peut améliorer considérablement les performances dans certains cas, mais il existe certaines restrictions :**

- Les filtres de requêtes globaux ne sont pas pris en charge.
- Les proxys de chargement différé ne sont pas pris en charge.
- Les proxys de suivi des modifications ne sont pas pris en charge.
- Le modèle doit être recompilé à chaque modification.

**==Si ces restrictions ne vous posent pas de problème, l’optimisation du `DbContext` peut améliorer considérablement les performances de vos applications.==**

# Résumé du chapitre

Ce chapitre a marqué le début de votre découverte d’Entity Framework Core. **Il a examiné les fondamentaux d’EF Core, le fonctionnement des requêtes et le suivi des modifications.** ***==Vous avez appris à structurer votre modèle à l’aide de conventions, d’annotations de données et de l’API Fluent. La dernière section a présenté la puissance de l’interface de ligne de commande et des outils globaux d’EF Core.==***

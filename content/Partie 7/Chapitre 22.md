---
publish: true
---

# <big><big><big><b><font color =green>Exploration d'Entity Framework Core</font></b></big></big></big>

Le chapitre précédent a présenté les composants d'EF Core. Ce chapitre explore les fonctionnalités d'EF Core, en commençant par les opérations CRUD (création, lecture, mise à jour et suppression). Après avoir abordé les opérations CRUD, nous examinerons les fonctionnalités spécifiques d'EF Core, notamment les filtres de requêtes globales, la combinaison de requêtes SQL et LINQ, les projections, et bien plus encore.

>[!note]
>Le code de ce chapitre fait suite à celui du chapitre précédent. Vous pouvez donc continuer à utiliser le code du [[Chapitre 21]] si vous avez suivi le texte. Si vous commencez par ce chapitre et souhaitez suivre le texte, utilisez le code du chapitre précédent disponible dans le dépôt.

# Création d'enregistrements

**Les enregistrements sont ajoutés à la base de données en les créant par programmation, en les ajoutant à leur `DbSet<T>` et en appelant `SaveChanges()`/`SaveChangesAsync()` sur le contexte.** ***==Lors de l'exécution de `SaveChanges()`, le `ChangeTracker` rapporte toutes les entités ajoutées et EF Core==*** (avec le fournisseur de base de données) ***==crée les instructions SQL appropriées pour insérer le ou les enregistrements.==*** ==Vous avez vu des exemples de ce processus plus tôt dans ce chapitre, lors de l'ajout des enregistrements d'exemple à la base de données.==

Pour rappel ([[Chapitre 21#Événements de sauvegarde/modifications enregistrées|Chapitre 21]]), **`SaveChanges()` s'exécute dans une transaction implicite, sauf si une transaction explicite est utilisée.** **==Si l'enregistrement a réussi, les valeurs générées par le serveur sont ensuite interrogées pour définir les valeurs des entités.==** ==La gestion des valeurs générées par le serveur est abordée en détail plus loin dans ce chapitre.==

Toutes les instructions SQL présentées dans cette section ont été collectées à l'aide de la méthode `LogTo()` d'EF Core.

>[!note]
>Il est également possible d'ajouter des enregistrements à l'aide du `DbContext` dérivé. Ces exemples utilisent tous les propriétés de la collection `DbSet<T>` pour ajouter les enregistrements. Les méthodes `Add()` et `AddRange()` de `DbSet<T>` et `DbContext` sont disponibles en versions asynchrones.

Pour commencer, ajoutez le code suivant dans le fichier *Program.cs* :

```cs
Console.Title = "More Fun with Entity Framework Core";
Console.WriteLine("*****  More Fun with Entity Framework Core *****\n");
```

>[!tip] Si vous garder les exemples de code provenant du chapitre précédent, il est préférable de créer une méthode regroupant tous les appels de méthodes par chapitre. Dans mes exemples de livre, j'utilise `TopLevelChapter21()` et `TopLevelChapter22()`

## État de l'entité

>[!Attention] 
>**Contrairement au livre, toutes les méthodes utilisé dans les chapitres suivant seront asynchrone ! (`Task`, `async`, `await`)**
>
>La méthode `CreateDbContext()` n'est pas asynchrone car elle à été au chapitre précédent et sera gardée telle quelle.

Lorsqu'une entité est créée par programmation mais n'est pas encore ajoutée à un `DbSet<T>`, son `EntityState` est `Detached`. Une fois l'entité ajoutée au `DbSet<T>`, son `EntityState` passe à `Added`. Après l'exécution réussie de `SaveChanges()`, l'`EntityState` passe à `Unchanged`.

Le code suivant illustre un enregistrement `Make` nouvellement créé et son `EntityState` :

```cs
static async Task AddRecordsAsync()
{
    //Cette fabrique n'est pas censée être utilisée ainsi,
    // mais c'est un code de démonstration :-)
    var context = new ApplicationDbContextFactory().CreateDbContext(null);
    var newMake = new Make { Name = "BMW" };
    Console.WriteLine(
        $"State of the {newMake.Name} is {context.Entry(newMake).State}"
    );
}
```

Après avoir appelé la méthode `AddRecordsAsync()` depuis les instructions de niveau supérieur, vous devriez voir le résultat suivant :

```
*****  More Fun with Entity Framework Core *****

State of the BMW is Detached
```

## Ajouter un enregistrement en utilisant `Add`

**Pour ajouter un nouvel enregistrement `Make` à la base de données, créez une nouvelle instance d'entité et appelez la méthode `Add()` du `DbSet<T>` approprié.** ***==Pour déclencher la persistance des données, la méthode `SaveChanges()` de la classe `DbContext` dérivée doit également être appelée.==*** Le code suivant ajoute le nouvel enregistrement `Make` à la base de données :

```cs
static async Task AddRecordAsync()
{
	...
    await context.Makes.AddAsync(newMake);
    Console.WriteLine(
        $"State of the {newMake.Name} is {context.Entry(newMake).State}"
    );
    await context.SaveChangesAsync();
    Console.WriteLine(
        $"State of the {newMake.Name} is {context.Entry(newMake).State}"
    );
}
```

**En exécutant à nouveau le programme, vous verrez la sortie suivante dans la console.** ==Après l'ajout de l'entité au Suivi des modifications== (à l'aide de la méthode `AddAsync()`), ==son état est passé à `Added`==. **Le message concernant l'enregistrement des modifications provient du gestionnaire d'événements `SavingChanges`, et le message « Saved 1 entities» provient du gestionnaire d'événements `SavedChanges`.** **==Après l'appel à `SaveChanges()` sur le contexte, l'état de l'entité est passé à `Unchanged`.

```
State of the BMW is Detached
State of the BMW is Added
Saving changes for ...
Saved 1 entities
State of the BMW is Unchanged
```

**==L'instruction SQL exécutée pour l'insertion est affichée ici-bas==**. **Le format de la requête est dû au traitement par lots utilisé par EF Core pour améliorer les performances des opérations de base de données.** ***==Le traitement par lots est abordé plus loin dans ce chapitre.==*** **Toutes les valeurs transmises à l'instruction SQL sont paramétrées afin de réduire le risque d'attaques par script.** **==Notez également que l'entité récemment ajoutée est interrogée==** (avec l'instruction `RETURNING` pour PostgreSQL et un `SELECT` complet sur SQL Server) **==sur les propriétés générées par la base de données.==** **La gestion des valeurs gérées par le serveur par EF Core est également abordée plus loin dans ce chapitre.**

```sql
INSERT INTO public."Makes" ("Name", "TimeStamp")
	VALUES (@p0, @p1) 
	RETURNING "Id", xmin
```

## Ajouter un enregistrement unique avec `Attach`

**Lorsqu'une clé primaire d'une entité est associée à une colonne d'identité, EF Core considère cette instance d'entité comme ajoutée lors de son ajout au `ChangeTracker` si la valeur de la propriété de clé primaire est `null`**. Le code suivant ajoute le nouvel enregistrement `Car` à l'aide de la méthode `AttachAsync()` au lieu de la méthode `AddAsync()`. ==Notez que la méthode `SaveChangesAsync()` doit toujours être appelée pour que les données soient enregistrées.==

```cs
static async Task AddRecords()
{
	...
	
    var newCar = new Car()
    {
        Color = "Blue",
        // DateTimeKind est nécessaire dans ce cas ci
        // (TimeStamp with Time Zone comme type dans la table)!
        DateBuilt = new DateTime(2016, 12, 01, 0, 0, 0, kind: DateTimeKind.Utc),
        IsDrivable = true,
        PetName = "Bluesmobile",
        MakeId = newMake.Id,
    };
    Console.WriteLine(
        $"State of the {newCar.PetName} is {context.Entry(newCar).State}"
    );
    context.Cars.Attach(newCar);
    Console.WriteLine(
        $"State of the {newCar.PetName} is {context.Entry(newCar).State}"
    );
    await context.SaveChangesAsync();
    Console.WriteLine(
        $"State of the {newCar.PetName} is {context.Entry(newCar).State}"
    );
}
```

En exécutant à nouveau le programme, vous verrez la même progression d'états que celle observée avec l'entité `Make` :

```
...

State of the Bluesmobile is Detached
State of the Bluesmobile is Added
Saving changes for ...
Saved 1 entities
State of the Bluesmobile is Unchanged
```

La requête SQL exécutée pour l'insertion est affichée ici :

```sql
INSERT INTO public."Inventory" ("Color", "DateBuilt", "IsDrivable", "MakeId", "PetName", "TimeStamp")
      VALUES (@p0, @p1, @p2, @p3, @p4, @p5)
      RETURNING "Id", "Display", xmin;
```

## Ajouter plusieurs enregistrements simultanément

**Pour insérer plusieurs enregistrements dans une seule transaction, utilisez la méthode `AddRangeAsync()` d'une propriété `DbSet<T>`,** comme illustré dans l'exemple suivant :

```cs
static async Task AddRecordsAsync()
{
	...
	
    List<Car> cars =
    [
        new()
        {
            Color = "Yellow",
            MakeId = newMake.Id,
            PetName = "Herbie",
        },
        new()
        {
            Color = "White",
            MakeId = newMake.Id,
            PetName = "Mach 5",
        },
        new()
        {
            Color = "Pink",
            MakeId = newMake.Id,
            PetName = "Avon",
        },
        new()
        {
            Color = "Blue",
            MakeId = newMake.Id,
            PetName = "Blueberry",
        },
    ];
    await context.Cars.AddRangeAsync(cars);
    await context.SaveChangesAsync();
}
```

~~Bien que quatre enregistrements aient été ajoutés, EF Core n'a généré qu'une seule instruction SQL pour les insertions~~. Voici cette instruction :

```sql
INSERT INTO public."Inventory" ("Color", "MakeId", "PetName", "TimeStamp")
	VALUES (@p0, @p1, @p2, @p3)
	RETURNING "Id", "DateBuilt", "Display", "IsDrivable", xmin;
INSERT INTO public."Inventory" ("Color", "MakeId", "PetName", "TimeStamp")
	VALUES (@p4, @p5, @p6, @p7)
	RETURNING "Id", "DateBuilt", "Display", "IsDrivable", xmin;                 INSERT INTO public."Inventory" ("Color", "MakeId", "PetName", "TimeStamp")
	VALUES (@p8, @p9, @p10, @p11)
	RETURNING "Id", "DateBuilt", "Display", "IsDrivable", xmin;
INSERT INTO public."Inventory" ("Color", "MakeId", "PetName", "TimeStamp")
	VALUES (@p12, @p13, @p14, @p15)
	RETURNING "Id", "DateBuilt", "Display", "IsDrivable", xmin;
```

>[!important] Différences entre Postgres et SQL Server (Avec Gemini)
>Le fournisseur SQL Server d'EF Core regroupe les insertions de données statiques en utilisant une syntaxe SQL spécifique qui permet d'insérer plusieurs lignes en une seule commande (Multi-row `VALUES`)
>
>Le fournisseur PostgreSQL (`Npgsql`) utilise une approche différente et plus moderne appelée le **Pipelining** (ou Batching natif).
>
>Dans les logs d'EF Core, chaque bloc de texte correspond à un seul appel réseau (un _Round-trip_). Ici, les quatre instructions `INSERT` sont écrites à la suite, séparées par de simples points-virgules (ou fins de ligne) au sein de la **même et unique exécution de commande**.
>
>>[!info] Est-ce moins performant ? 
>>**Non, au contraire.** L'approche par _Pipelining_ de PostgreSQL est souvent plus performante que la réécriture de requêtes de SQL Server, car :
>>
>>- Elle permet à PostgreSQL de réutiliser le plan d'exécution de la requête de manière optimale.
>>- Elle évite de devoir reconstruire dynamiquement une chaîne SQL gigantesque en mémoire C# si vous aviez 100 voitures à insérer d'un coup.

## Considérations relatives aux colonnes d'identité lors de l'ajout d'enregistrements

**Lorsqu'une entité possède une propriété numérique définie comme clé primaire, cette propriété est (par défaut) associée à une colonne d'identité. EF Core considère toute entité dont la valeur par défaut est zéro comme nouvelle, et toute entité dont la valeur est différente de zéro comme déjà existante dans la base de données.** *==Si vous créez une nouvelle entité, définissez sa clé primaire sur un nombre différent de zéro et tentez de l'ajouter à la base de données, EF Core ne pourra pas ajouter l'enregistrement car l'insertion d'identité n'est pas activée.==*

==Pour SQL Server, l'insertion d'identité est activée en exécutant la commande `SET IDENTITY_INSERT` dans une transaction explicite.== Cette commande requiert le schéma de base de données et le nom de la table, et non l'espace de noms C# ni le nom de l'entité. **Pour obtenir les informations de base de données d'une entité, utilisez la méthode `FindEntityType()` de la propriété `Model` du `DbContext` dérivé.** **==Une fois le type d'entité obtenu, utilisez les méthodes `GetSchema()` et `GetTableName()`.==**

>[!warning] PostgreSQL n'a pas d'équivalent à la commande globale `SET IDENTITY_INSERT ON/OFF` (Avec Gemini)
> Il n'y a aucun état de session ou de table à activer ou désactiver dans une transaction. Vous ne pouvez pas exécuter de commande textuelle pour "autoriser" temporairement l'insertion d'un ID.
> 
> Si votre table utilise une colonne d'identité native (générée via `GENERATED BY DEFAULT AS IDENTITY` ou le type historique `SERIAL`), PostgreSQL possède un comportement très flexible :
>
>- Si la requête `INSERT` **ne contient pas** la colonne `Id`, PostgreSQL génère la valeur suivante de la séquence.
>- Si la requête `INSERT` **contient explicitement** une valeur pour `Id` (par exemple `125`), PostgreSQL insère simplement cette valeur sans broncher.
>
>Comme l'ID n'est pas égal à `0`, EF Core va croire que cette entité existe déjà en base de données et **refusera de générer un `INSERT` lors du `SaveChanges()`.**
>

```cs
static async Task AddRecordsAsync()
{
	...
	
    IEntityType metadata = context.Model.FindEntityType(typeof(Car).FullName);
    var schema = metadata.GetSchema();
    var tablename = metadata.GetTableName();
}
```

**Rappelons, comme indiqué dans le chapitre précédent, que lors de l'utilisation d'une stratégie d'exécution, les transactions explicites doivent s'exécuter dans le cadre de cette stratégie.** À titre de référence, voici un exemple tiré du chapitre précédent :

```cs
var strategy = context.Database.CreateExecutionStrategy();
strategy.Execute(() =>
{
	using var trans = context.Database.BeginTransaction();
	try
	{
		//actionToExecute();
		trans.Commit();
		Console.WriteLine($"Insert succeeded");
	}
	catch (Exception ex)
	{
		trans.Rollback();
		Console.WriteLine($"Insert failed: {ex.Message}");
	}
});
```

**Lors de l'ajout d'un enregistrement via l'insertion d'identité, l'espace réservé `actionToExecute()` du bloc de code précédent est remplacé par du code activant l'insertion d'identité, ajoutant le ou les enregistrements, puis enregistrant les modifications.** **==Si tout se déroule correctement, la transaction est validée.==** *==En cas d'échec, la transaction est annulée.==* **Dans le bloc `finally`, l'insertion d'identité est désactivée.**

**==EF Core propose deux méthodes pour exécuter des commandes directement sur la base de données. La méthode `ExecuteSqlRaw()` exécute la chaîne telle quelle, tandis que `ExecuteSqlInterpolated()` utilise l'interpolation de chaînes C# pour créer une requête paramétrée.==** Pour cet exemple, utilisez la version `ExecuteSqlInterpolatedRawAsync()`. Voici le code mis à jour, les nouvelles lignes étant en gras :

```cs
static async Task AddRecords()
{
	...
	
    IEntityType metadata = context.Model.FindEntityType(typeof(Car).FullName);
    var schema = metadata.GetSchema() ?? "public";
    var tableName = metadata.GetTableName();

    var strategy = context.Database.CreateExecutionStrategy();
    await strategy.ExecuteAsync(async () =>
    {
        using var trans = await context.Database.BeginTransactionAsync();
        try
        {
            var anotherNewCar = new Car()
            {
                Id = 27, // ID explicite non nul
                Color = "Blue",
                DateBuilt = new DateTime(
                    2016,
                    12,
                    01,
                    0,
                    0,
                    0,
                    DateTimeKind.Utc
                ),
                IsDrivable = true,
                PetName = "Bluesmobile",
                MakeId = newMake.Id,
            };

            // 1. On l'entité normalement
            await context.Cars.AddAsync(anotherNewCar);

            // 2. On force manuellement son état à "Added".
            // Cela force EF Core à inclure l'Id dans
            // la requête d'insertion PostgreSQL.
            context.Entry(anotherNewCar).State = EntityState.Added;

            await context.SaveChangesAsync();

            // 3. On recale la séquence PostgreSQL
            // pour les futurs inserts automatiques
            await context.Database.ExecuteSqlInterpolatedAsync(
                $"SELECT setval(pg_get_serial_sequence('\"{schema}\".\"{tableName}\"', 'Id'), COALESCE(max(\"Id\"), 1)) FROM \"{schema}\".\"{tableName}\";"
            );

            await trans.CommitAsync();
            Console.WriteLine($"Insert succeeded");
        }
        catch (Exception ex)
        {
            await trans.RollbackAsync();
            Console.WriteLine($"Insert failed : {ex.Message}");
        }
    });
}
```

>[!note]
>Lorsque vous utilisez des valeurs connues, comme dans cet exemple, la méthode `ExecuteSqlRaw()` est sûre. Cependant, si vous collectez des données saisies par des utilisateurs, il est recommandé d'utiliser la version `ExecuteSqlInterpolated()` pour une protection accrue.

Le code précédent a exécuté les commandes suivantes sur la base de données :

```sql
NSERT INTO public."Inventory" ("Id", "Color", "DateBuilt", "IsDrivable", "MakeId", "PetName", "TimeStamp")
	VALUES (@p0, @p1, @p2, @p3, @p4, @p5, @p6)
	RETURNING "Display", xmin;
	
SELECT setval(pg_get_serial_sequence('"public"."Inventory"', 'Id'),
	COALESCE(max("Id"), 1)) FROM "public"."Inventory"
```

## Ajout d'un graphe d'objets

**Lors de l'ajout d'une entité à la base de données, les enregistrements enfants peuvent être ajoutés en une seule opération, sans avoir à les ajouter explicitement à leur propre `DbSet<T>`.** Pour ce faire, il suffit de les ajouter à la propriété de navigation de collection de l'enregistrement parent. Par exemple, une nouvelle entité `Make` est créée, et un enregistrement enfant `Car` est ajouté à la propriété `Cars` de l'entité `Make`. Lorsque l'entité `Make` est ajoutée à la propriété `DbSet<Make>`, EF Core commence automatiquement à suivre l'enregistrement enfant `Car`, sans qu'il soit nécessaire de l'ajouter explicitement à la propriété `DbSet<Car>`. L'exécution de `SaveChanges()` enregistre les entités `Make` et `Car` ensemble. Le test suivant illustre ce comportement :

# Résilience de la connection



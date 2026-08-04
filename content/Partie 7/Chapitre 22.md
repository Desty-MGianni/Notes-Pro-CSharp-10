---
publish: true
---

# <big><big><big><b><font color =green>Exploration d'Entity Framework Core</font></b></big></big></big>

Le chapitre précédent a présenté les composants d'EF Core. **Ce chapitre explore les fonctionnalités d'EF Core, en commençant par les opérations CRUD** (création, lecture, mise à jour et suppression). Après avoir abordé les opérations CRUD, **nous examinerons les fonctionnalités spécifiques d'EF Core,** notamment les filtres de requêtes globales, la combinaison de requêtes SQL et LINQ, les projections, et bien plus encore.

>[!note]
>Le code de ce chapitre fait suite à celui du chapitre précédent. Vous pouvez donc continuer à utiliser le code du [[Chapitre 21]] si vous avez suivi le texte. Si vous commencez par ce chapitre et souhaitez suivre le texte, utilisez le code du chapitre précédent disponible dans le dépôt.

# Création d'enregistrements

**Les enregistrements sont ajoutés à la base de données en les créant par programmation, en les ajoutant à leur `DbSet<T>` et en appelant `SaveChanges()`/`SaveChangesAsync()` sur le contexte.** Lors de l'exécution de `SaveChanges()`, le `ChangeTracker` rapporte toutes les entités ajoutées et EF Core (avec le fournisseur de base de données) rée les instructions SQL appropriées pour insérer le ou les enregistrements. ==Vous avez vu des exemples de ce processus plus tôt dans ce chapitre, lors de l'ajout des enregistrements d'exemple à la base de données.==

Pour rappel ([[Chapitre 21#Événements de sauvegarde/modifications enregistrées|Chapitre 21]]), **`SaveChanges()` s'exécute dans une transaction implicite, sauf si une transaction explicite est utilisée.** **==Si l'enregistrement a réussi, les valeurs générées par le serveur sont ensuite interrogées pour définir les valeurs des entités.==** La gestion des valeurs générées par le serveur est abordée en détail plus loin dans ce chapitre.

Toutes les instructions SQL présentées dans cette section ont été collectées à l'aide de la méthode `LogTo()` d'EF Core.

>[!note]
>Il est également possible d'ajouter des enregistrements à l'aide du `DbContext` dérivé. Ces exemples utilisent tous les propriétés de la collection `DbSet<T>` pour ajouter les enregistrements. Les méthodes `Add()` et `AddRange()` de `DbSet<T>` et `DbContext` sont disponibles en versions asynchrones.

Pour commencer, ajoutez le code suivant dans le fichier *Program.cs* :

```cs
Console.Title = "More Fun with Entity Framework Core";
Console.WriteLine("*****  More Fun with Entity Framework Core *****\n");
```

>[!Important] 
>Si vous garder les exemples de code provenant du chapitre précédent, il est préférable de créer une méthode regroupant tous les appels de méthodes par chapitre. Dans mes exemples de livre, j'utilise `TopLevelChapter21()` et `TopLevelChapter22()`

## État de l'entité

>[!Attention] 
>**Contrairement au livre, toutes les méthodes utilisé dans les chapitres suivant seront asynchrone ! (`Task`, `async`, `await`)**
>
>La méthode `CreateDbContext()` n'est pas asynchrone car elle à été au chapitre précédent et sera gardée telle quelle.

**Lorsqu'une entité est créée par programmation mais n'est pas encore ajoutée à un `DbSet<T>`, son `EntityState` est `Detached`. Une fois l'entité ajoutée au `DbSet<T>`, son `EntityState` passe à `Added`. Après l'exécution réussie de `SaveChanges()`, l'`EntityState` passe à `Unchanged`.**

Le code suivant illustre un enregistrement `Make` nouvellement créé et son `EntityState` :

```CS
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

**Pour ajouter un nouvel enregistrement `Make` à la base de données, créez une nouvelle instance d'entité et appelez la méthode `Add()` du `DbSet<T>` approprié.** **Pour déclencher la persistance des données, la méthode `SaveChanges()` de la classe `DbContext` dérivée doit également être appelée.** Le code suivant ajoute le nouvel enregistrement `Make` à la base de données :

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

En exécutant à nouveau le programme, vous verrez la sortie suivante dans la console. ==Après l'ajout de l'entité au Suivi des modifications== (à l'aide de la méthode `AddAsync()`), ==son état est passé à `Added`==. **Le message concernant l'enregistrement des modifications provient du gestionnaire d'événements `SavingChanges`, et le message « Saved 1 entities» provient du gestionnaire d'événements `SavedChanges`.** Après l'appel à `SaveChanges()` sur le contexte, l'état de l'entité est passé à `Unchanged`.

```
State of the BMW is Detached
State of the BMW is Added
Saving changes for ...
Saved 1 entities
State of the BMW is Unchanged
```

L'instruction SQL exécutée pour l'insertion est affichée ici-bas. **==Le format de la requête est dû au traitement par lots utilisé par EF Core pour améliorer les performances des opérations de base de données.==** Le traitement par lots est abordé plus loin dans ce chapitre. **Toutes les valeurs transmises à l'instruction SQL sont paramétrées afin de réduire le risque d'attaques par script.** Notez également que l'entité récemment ajoutée est interrogée (avec l'instruction `RETURNING` pour PostgreSQL et un `SELECT` complet sur SQL Server) sur les propriétés générées par la base de données. **La gestion des valeurs gérées par le serveur par EF Core est également abordée plus loin dans ce chapitre.**

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

Lorsqu'une entité possède une propriété numérique définie comme clé primaire, cette propriété est (par défaut) associée à une colonne d'identité. **EF Core considère toute entité dont la valeur par défaut est zéro comme nouvelle, et toute entité dont la valeur est différente de zéro comme déjà existante dans la base de données.** *==Si vous créez une nouvelle entité, définissez sa clé primaire sur un nombre différent de zéro et tentez de l'ajouter à la base de données, EF Core ne pourra pas ajouter l'enregistrement car l'insertion d'identité n'est pas activée.==*

Pour SQL Server, l'insertion d'identité est activée en exécutant la commande `SET IDENTITY_INSERT` dans une transaction explicite. Cette commande requiert le schéma de base de données et le nom de la table, et non l'espace de noms C# ni le nom de l'entité. **Pour obtenir les informations de base de données d'une entité, utilisez la méthode `FindEntityType()` de la propriété `Model` du `DbContext` dérivé.** **==Une fois le type d'entité obtenu, utilisez les méthodes `GetSchema()` et `GetTableName()`.==**

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

Rappelons, comme indiqué dans le chapitre précédent, que **lors de l'utilisation d'une stratégie d'exécution, les transactions explicites doivent s'exécuter dans le cadre de cette stratégie.** À titre de référence, voici un exemple tiré du chapitre précédent :

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

**Lors de l'ajout d'un enregistrement via l'insertion d'identité, l'espace réservé `actionToExecute()` du bloc de code précédent est remplacé par du code activant l'insertion d'identité, ajoutant le ou les enregistrements, puis enregistrant les modifications.** Si tout se déroule correctement, la transaction est validée.*==En cas d'échec, la transaction est annulée.==* **Dans le bloc `finally`, l'insertion d'identité est désactivée.**

**EF Core propose deux méthodes pour exécuter des commandes directement sur la base de données.** La méthode `ExecuteSqlRaw()` exécute la chaîne telle quelle, tandis que `ExecuteSqlInterpolated()` utilise l'interpolation de chaînes C# pour créer une requête paramétrée. ==Pour cet exemple, utilisez la version `ExecuteSqlInterpolatedRawAsync()`.== Voici le code mis à jour :

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

**Lors de l'ajout d'une entité à la base de données, les enregistrements enfants peuvent être ajoutés en une seule opération, sans avoir à les ajouter explicitement à leur propre `DbSet<T>`.** Pour ce faire, **==il suffit de les ajouter à la propriété de navigation de collection de l'enregistrement parent.==** Par exemple, une nouvelle entité `Make` est créée, et un enregistrement enfant `Car` est ajouté à la propriété `Cars` de l'entité `Make`. Lorsque l'entité `Make` est ajoutée à la propriété `DbSet<Make>`, EF Core commence automatiquement à suivre l'enregistrement enfant `Car`, sans qu'il soit nécessaire de l'ajouter explicitement à la propriété `DbSet<Car>`. L'exécution de `SaveChanges()` enregistre les entités `Make` et `Car` ensemble.** Le test suivant illustre ce comportement :

```cs
static async Task AddRecordsAsync()
{
    var anotherMake = new Make { Name = "Honda" };
    var car = new Car { Color = "Yellow", PetName = "Herbie" };
    //Casting de la propriété Cars de IEnumerable<Car> vers List<Car>
    ((List<Car>)anotherMake.Cars).Add(car);
    await context.Makes.AddAsync(anotherMake);
    await context.SaveChangesAsync();
}
```

Les instructions SQL exécutées sont affichées ici :

```sql
INSERT INTO public."Makes" ("Name", "TimeStamp")
	VALUES (@p0, @p1)
	RETURNING "Id", xmin;
	
INSERT INTO public."Inventory" ("Color", "MakeId", "PetName", "TimeStamp")
	VALUES (@p2, @p3, @p4, @p5)
	RETURNING "Id", "DateBuilt", "Display", "IsDrivable", xmin;
```

**Remarquez comment EF Core a récupéré l'identifiant du nouvel enregistrement `Make` et l'a automatiquement inclus dans l'instruction d'insertion pour l'enregistrement `Car`.**

> [!IMPORTANT] SQL Server vs PostgreSQL : Insertion d'IDs manuels (Avec Gemini)
> Bien que le code C# final produise le même résultat (ID 28), le comportement interne des deux moteurs de base de données est radicalement différent après une insertion manuelle :
> 
> - **SQL Server (`SET IDENTITY_INSERT`)** : Le compteur d'identité est *intelligent*. Lors de l'insertion manuelle de l'ID `27`, le moteur détecte que cette valeur dépasse le compteur actuel (`5`) et **recale automatiquement** sa séquence interne à 27. La voiture suivante reçoit l'ID `28`.
> - **PostgreSQL (`IDENTITY` / `SEQUENCE`)** : Le compteur est *aveugle*. Il est stocké dans un objet autonome (`SEQUENCE`) qui ignore totalement vos insertions manuelles. Si vous insérez l'ID `27`, le compteur reste bloqué à `5`.
> 
> ⚠️ **Le piège Postgres** : La voiture suivante aurait reçu l'ID `6` (risquant un crash futur par clé dupliquée arrivant à 27). Elle n'a reçu l'ID `28` que parce que **votre code a forcé** le saut du compteur en exécutant manuellement la fonction `setval(..., max("Id"))`.

## Ajout d'enregistrements de type plusieurs-à-plusieurs

**Grâce à la nouvelle prise en charge des tables plusieurs-à-plusieurs par EF Core, il est possible d'ajouter directement des enregistrements d'une entité à l'autre, *sans passer par la table pivot.*** Vous pouvez désormais utiliser le code suivant pour ajouter directement des enregistrements de conducteur aux enregistrements de `Car` :

```cs
static async Task AddRecordsAsync()
{
	...
	
    var carsForM2M = await context.Cars.Take(2).ToListAsync();
    /*
     * Cast l'IEnumerable en List pour accéder à la méthode Add.
     *
     * La prise en charge des plages fonctionne avec LINQ to Objects
     * mais n'est pas traduisible en requêtes SQL.
     *
     * AddRangeAsync n'existe que pour le type DbSet<T>,
     * pas pour une List<T>.
     */
    ((List<Driver>)carsForM2M[0].Drivers).AddRange(drivers.Take(..3));
    ((List<Driver>)carsForM2M[1].Drivers).AddRange(drivers.Take(3..));
    await context.SaveChangesAsync();
}
```

**Lors de l'exécution de la méthode `SaveChanges()`, deux instructions `INSERT` (SQL Server) ou deux blocs d'instruction `INSERT` (PostgreSQL) sont exécutées.** La première insère les six enregistrements `Driver` dans la table `Drivers`, et la seconde insère les six enregistrements dans la table `InventoryDriver` (table pivot). Voici l'instruction `INSERT` pour la table pivot :

```sql
INSERT INTO public."InventoryToDrivers" ("InventoryId", "DriverId", "TimeStamp")
	VALUES (@p18, @p19, @p20)
	RETURNING "Id", xmin;
INSERT INTO public."InventoryToDrivers" ("InventoryId", "DriverId", "TimeStamp")
	VALUES (@p21, @p22, @p23)
	RETURNING "Id", xmin;
INSERT INTO public."InventoryToDrivers" ("InventoryId", "DriverId", "TimeStamp")
	VALUES (@p24, @p25, @p26)
	RETURNING "Id", xmin;
INSERT INTO public."InventoryToDrivers" ("InventoryId", "DriverId", "TimeStamp")
	VALUES (@p27, @p28, @p29)
	RETURNING "Id", xmin;
INSERT INTO public."InventoryToDrivers" ("InventoryId", "DriverId", "TimeStamp")
	VALUES (@p30, @p31, @p32)
	RETURNING "Id", xmin;
INSERT INTO public."InventoryToDrivers" ("InventoryId", "DriverId", "TimeStamp")
	VALUES (@p33, @p34, @p35)
	RETURNING "Id", xmin;
```

**==L'expérience est bien meilleure qu'avec les versions précédentes d'EF Core lors de l'utilisation de relations plusieurs-à-plusieurs, où il fallait gérer soi-même la table pivot.==**

## Ajouter des enregistrements d'exemple

La dernière étape consiste à ajouter une série d'enregistrements `Make` et `Make` pour les exemples de requêtes présentés dans la section suivante. Cette méthode crée plusieurs entités `Make` et `Car` et les ajoute à la base de données.

Commencez par créer les entités `Make` et ajoutez-les à la propriété `Makes` `DbSet<Make>` de l'objet dérivé `ApplicationDbContext`, puis appelez la méthode `SaveChanges()`. Répétez ce processus pour les enregistrements `Car`, en utilisant la propriété Cars `DbSet<Car>` :

```cs
static async Task LoadMakesAndCarData()
{
    // Cette fabrique n'est pas censée être utilisée ainsi,
    // mais c'est un code de démonstration :-)
    var context = new ApplicationDbContextFactory().CreateDbContext(null);
    List<Make> makes =
    [
        new() { Name = "VW" },
        new() { Name = "Ford" },
        new() { Name = "Saab" },
        new() { Name = "Yugo" },
        new() { Name = "BMW" },
        new() { Name = "Pinto" },
    ];
    await context.Makes.AddRangeAsync(makes);
    await context.SaveChangesAsync();

    List<Car> inventory =
    [
        new()
        {
            MakeId = 1,
            Color = "Black",
            PetName = "Zippy",
        },
        new()
        {
            MakeId = 2,
            Color = "Rust",
            PetName = "Rusty",
        },
        new()
        {
            MakeId = 3,
            Color = "Black",
            PetName = "Mel",
        },
        new()
        {
            MakeId = 4,
            Color = "Yellow",
            PetName = "Clunker",
        },
        new()
        {
            MakeId = 5,
            Color = "Black",
            PetName = "Bimmer",
        },
        new()
        {
            MakeId = 5,
            Color = "Green",
            PetName = "Hank",
        },
        new()
        {
            MakeId = 5,
            Color = "Pink",
            PetName = "Pinky",
        },
        new()
        {
            MakeId = 6,
            Color = "Black",
            PetName = "Pete",
        },
        new()
        {
            MakeId = 4,
            Color = "Brown",
            PetName = "Brownie",
        },
        new()
        {
            MakeId = 1,
            Color = "Rust",
            PetName = "Lemon",
            IsDrivable = false,
        },
    ];
    await context.Cars.AddRangeAsync(inventory);
    await context.SaveChangesAsync();
}
```

# Effacer les données d'exemple

**La suppression des enregistrements sera abordée plus en détail ultérieurement dans ce chapitre.** Pour l'instant, nous allons créer une méthode qui efface les données d'exemple afin que, lors de l'exécution répétée des exemples, les exécutions précédentes n'interfèrent pas avec les exemples.

Créez une nouvelle méthode appelée `ClearSampleData()`. Cette méthode utilise **la méthode `FindEntityType()` sur la propriété `Model` de l'`ApplicationDbContext` pour obtenir le nom de la table et du schéma,** puis supprime les enregistrements. Une fois les enregistrements supprimés, le code utilise la commande `TRUNCATE ... RESTART IDENTITY CASCADE` pour réinitialiser l'identité de chaque table.

```cs
static async Task ClearSampleData()
{
    // Cette fabrique n'est pas censée être utilisée ainsi,
    // mais c'est un code de démonstration :-)
    var context = new ApplicationDbContextFactory().CreateDbContext(null);
    var entities = new[]
    {
        typeof(Driver).FullName,
        typeof(Car).FullName,
        typeof(Make).FullName,
    };

    foreach (var entityName in entities)
    {
        var entity = context.Model.FindEntityType(entityName);
        var tableName = entity.GetTableName();
        var schemaName = entity.GetSchema() ?? "public";
        await context.Database.ExecuteSqlRawAsync(
            $"TRUNCATE TABLE \"{schemaName}\".\"{tableName}\" RESTART IDENTITY CASCADE;"
        );
    }
}
```

Ajoutez un appel à cette méthode au début des instructions principales pour réinitialiser la base de données à chaque exécution du programme. Ajoutez également un appel après la méthode `AddRecords()` pour nettoyer les exemples qui ajoutent des enregistrements individuels.

```cs
Console.Title = "More Fun with Entity Framework Core";
Console.WriteLine("*****  More Fun with Entity Framework Core *****\n");

await ClearSampleData();
await AddRecordsAsync();
await ClearSampleData();
await LoadMakesAndCarData();
Console.ReadLine();
```

>[!tip] Si on utilise les versions asynchrone des méthodes, il faut absolument un moyen pour attendre que les opérations soient exécutée. Ici, le plus simples est d'utilisé un appel à `Console.ReadLine()`.

## Requête de données

**La requête de données avec EF Core s'effectue généralement à l'aide de requêtes LINQ.** Pour rappel, lorsqu'on utilise LINQ pour interroger la base de données afin d'obtenir une liste d'entités, la requête n'est exécutée qu'après itération, conversion en `List<T>` (ou en tableau), ou liaison à un contrôle de liste (comme une grille de données). **Pour les requêtes portant sur un seul enregistrement, l'instruction est exécutée immédiatement lors de l'utilisation de la méthode `First()`, `Single()`, etc.**

>[!note]
>Ce livre ne constitue pas une référence complète sur LINQ, mais présente quelques exemples. Pour plus d'exemples de requêtes LINQ, voir [Les 101 exemples de requêtes LINQ de Microsoft](https://github.com/dotnet/try-samples).

Nouveauté d'EF Core 5 : **vous pouvez appeler la méthode `ToQueryString()` dans la plupart des requêtes LINQ pour examiner la requête exécutée sur la base de données.** La principale exception concerne les requêtes à exécution immédiate, telles que `First()`/`FirstOrDefault()`. *==Pour les requêtes fractionnées, la méthode `ToQueryString()` ne renvoie que la première requête exécutée.==*

```cs
static async Task QueryData()
{
    // Cette fabrique n'est pas censée être utilisée ainsi,
    // mais c'est un code de démonstration :-)
    var context = new ApplicationDbContextFactory().CreateDbContext(null);

    // Retourne toutes les voitures
    IQueryable<Car> cars = context.Cars;
    foreach (Car c in cars)
        Console.WriteLine($"{c.PetName} is {c.Color}");

    // Nettoie le context
    context.ChangeTracker.Clear();
    Console.WriteLine();

    var cars2 = await context.Cars.ToListAsync();
    foreach (Car c in cars2)
        Console.WriteLine($"{c.PetName} is {c.Color}");
}
```

**Notez que le type renvoyé est un `IQueryable<Car>` lors de l'utilisation de `DbSet<Car>`, et le type de retour est `List<Car>` lors de l'utilisation de la méthode `ToList()`.** La différence d'exécution de la requêtes, comme expliqué au dessus, est visible ici :

```
...

An entity of type Car was loaded from the database.
Zippy is Black
An entity of type Car was loaded from the database.
Rusty is Rust
An entity of type Car was loaded from the database.
Mel is Black
An entity of type Car was loaded from the database.
Clunker is Yellow
An entity of type Car was loaded from the database.
Bimmer is Black
An entity of type Car was loaded from the database.
Hank is Green
An entity of type Car was loaded from the database.
Pinky is Pink
An entity of type Car was loaded from the database.
Pete is Black
An entity of type Car was loaded from the database.
Brownie is Brown
An entity of type Car was loaded from the database.
Lemon is Rust

An entity of type Car was loaded from the database.
An entity of type Car was loaded from the database.
An entity of type Car was loaded from the database.
An entity of type Car was loaded from the database.
An entity of type Car was loaded from the database.
An entity of type Car was loaded from the database.
An entity of type Car was loaded from the database.
An entity of type Car was loaded from the database.
An entity of type Car was loaded from the database.
An entity of type Car was loaded from the database.
Zippy is Black
Rusty is Rust
Mel is Black
Clunker is Yellow
Bimmer is Black
Hank is Green
Pinky is Pink
Pete is Black
Brownie is Brown
Lemon is Rust
```

## Filtrer les enregistrements

**La méthode `Where()` permet de filtrer les enregistrements d'un `DbSet<T>`.** **==Plusieurs méthodes `Where()` peuvent être chaînées de manière fluide pour construire dynamiquement la requête.==** Les méthodes `Where()` chaînées sont toujours combinées sous forme de clauses `AND` dans la requête créée. Dans l'exemple suivant, les requêtes générées pour `cars2` et `cars3` sont identiques. Pour créer une instruction `OR`, vous devez utiliser la même clause `Where()`.

```cs
static void FilterData()
{
    // Cette fabrique n'est pas censée être utilisée ainsi,
    // mais c'est un code de démonstration :-)
    var context = new ApplicationDbContextFactory().CreateDbContext(null);

    // Retourne toutes les voitures jaunes
    IQueryable<Car> cars = context.Cars.Where(c => c.Color == "Yellow");
    Console.WriteLine("**** Yellow cars ****");
    foreach (Car c in cars)
        Console.WriteLine($"{c.PetName} is {c.Color}");
    context.ChangeTracker.Clear();

    // Retourne toutes les voitures jaunes avec un PetName = Clunker
    IQueryable<Car> cars2 = context.Cars.Where(c =>
        c.Color == "Yellow" && c.PetName == "Clunker"
    );
    Console.WriteLine("**** Yellow cars AND Clunkers ****");
    foreach (Car c in cars2)
        Console.WriteLine($"{c.PetName} is {c.Color}");
    context.ChangeTracker.Clear();

    // Retourne toutes les voitures jaunes avec un PetName = Clunker
    IQueryable<Car> cars3 = context
        .Cars.Where(c => c.Color == "Yellow")
        .Where(c => c.PetName == "Clunker");
    Console.WriteLine("**** Yellow cars AND Clunkers ****");
    foreach (Car c in cars3)
        Console.WriteLine($"{c.PetName} is {c.Color}");
    context.ChangeTracker.Clear();

    // Retourne toutes les voitures jaunes ou avec un PetName = Clunker
    IQueryable<Car> cars4 = context.Cars.Where(c =>
        c.Color == "Yellow" || c.PetName == "Clunker"
    );
    Console.WriteLine("**** Yellow cars OR Clunkers ****");
    foreach (Car c in cars4)
        Console.WriteLine($"{c.PetName} is {c.Color}");
    context.ChangeTracker.Clear();
}
```

**Notez que le type renvoyé est également un `IQueryable<Car>` lors de l'utilisation d'une clause `Where`.**

***==Une des améliorations d'EF Core 6 concerne la conversion de `string.IsNullOrWhiteSpace()` en SQL.==*** Examinez le code ajouté à la fin de la méthode `FilterData()` :

```cs
static void FilterData()
{
	...
	
    // Retourne toutes les voitures possédant une couleur
    IQueryable<Car> cars5 = context.Cars.Where(c =>
        !string.IsNullOrWhiteSpace(c.Color)
    );
    Console.WriteLine("**** Cars with colors ****");
    foreach (Car c in cars5)
        Console.WriteLine($"{c.PetName} is {c.Color}");
    context.ChangeTracker.Clear();
}
```

Avant EF Core 6, la requête résultante (==**Sur SQL Server**==) était un mélange de commandes `LTRIM`/`RTRIM`. Grâce aux améliorations apportées par EF Core 6, la requête exécutée est beaucoup plus propre :

```sql
SELECT i."Id", i."Color", i."DateBuilt", i."Display", i."IsDrivable", i."MakeId", i."PetName", i."TimeStamp", i.xmin
      FROM public."Inventory" AS i
      WHERE btrim(i."Color", E' \t\n\r') <> ''
```

> [!Warning] Performance : Le piège de `IsNullOrWhiteSpace` sur PostgreSQL
> 
> - **SQL Server** : Traduit l'expression par un simple `[Color] <> N''` (grâce au comportement natif de trim de SQL Server). Elle reste optimisée.
> - **PostgreSQL** : Force l'utilisation de la fonction `btrim(i."Color", E' \t\n\r')`. 
> 
>**Problème d'index** : Appliquer une fonction (`btrim`) sur une colonne dans un `WHERE` désactive les index standards de PostgreSQL et force un scan complet de la table (*Seq Scan*).
> 
>**Correction recommandée** : Remplacer par `Where(c => c.Color != null && c.Color != "" && c.Color != " ")` pour générer du SQL brut indexable.

## Trier les enregistrements

**Les méthodes `OrderBy()` et `OrderByDescending()` définissent le ou les tris de la requête, respectivement par ordre croissant ou décroissant. Si des tris supplémentaires sont nécessaires, utilisez les méthodes `ThenBy()` et/ou `ThenByDescending()`.**

```cs
static void SortData()
{
    // Cette fabrique n'est pas censée être utilisée ainsi,
    // mais c'est un code de démonstration :-)
    var context = new ApplicationDbContextFactory().CreateDbContext(null);

    // Retourne toutes les voitures triées par couleurs
    IOrderedQueryable<Car> cars1 = context.Cars.OrderBy(c => c.Color);
    Console.WriteLine("**** Cars ordered by Color ****");
    foreach (Car c in cars1)
        Console.WriteLine($"{c.PetName} is {c.Color}");
    context.ChangeTracker.Clear();

    // Retourne toutes les voitures triées par couleurs puis par nom familier.
    IOrderedQueryable<Car> cars2 = context
        .Cars.OrderBy(c => c.Color)
        .ThenBy(c => c.PetName);
    Console.WriteLine("***** Cars ordered by Color then by PetName");
    foreach (Car c in cars2)
        Console.WriteLine($"{c.PetName} is {c.Color}");
    context.ChangeTracker.Clear();

    // Retourne toutes les voitures triées par couleurs descendantes.
    IOrderedQueryable<Car> cars3 = context.Cars.OrderByDescending(c => c.Color);
    Console.WriteLine("**** Cars ordered by Color descending ****");
    foreach (Car c in cars3)
        Console.WriteLine($"{c.PetName} is {c.Color}");
    context.ChangeTracker.Clear();
}
```

==Notez que le type de données renvoyé par une requête LINQ avec `OrderBy()`/`OrderByDescending()` est `IOrderedQueryable<Car>`.==

**Le tri ascendant et descendant peut être combiné**, comme illustré ici :

```cs
static void SortData()
{
	...
	
    // Retourne toutes les voitures triées par couleurs
    // puis par nom familier descendant.
    IOrderedQueryable<Car> cars4 = context
        .Cars.OrderBy(c => c.Color)
        .ThenByDescending(c => c.PetName);
    Console.WriteLine(
        "**** Cars ordered by Color then by PetName descending ****"
    );
    foreach (Car c in cars4)
        Console.WriteLine($"{c.PetName} is {c.Color}");
    context.ChangeTracker.Clear();

```

### Tri inversé des enregistrements

**La méthode `Reverse()` inverse l'ordre de tri complet,** comme illustré ici :

```cs
static void SortData()
{
	...
	
    // Retourne toutes les voitures triées par couleurs
    // puis par nom familier inversé.
    IQueryable<Car> cars5 = context
        .Cars.OrderBy(c => c.Color)
        .ThenBy(c => c.PetName)
        .Reverse();
    Console.WriteLine("**** Cars ordered by Color ****");
    foreach (Car c in cars4)
        Console.WriteLine($"{c.PetName} is {c.Color}");
    context.ChangeTracker.Clear();
}
```

==Notez que le type de données renvoyé par une requête LINQ avec une clause `Reverse()` est `IQueryable<Car>`, et non `IOrderedQueryable<Car>`==.

La requête LINQ précédente est traduite comme suit :

```sql
SELECT i."Id", i."Color", i."DateBuilt", i."Display", i."IsDrivable", i."MakeId", i."PetName", i."TimeStamp", i.xmin
FROM public."Inventory" AS i
ORDER BY i."Color", i."PetName" DESC
```

>[!warning] Différence entre SQL Server et PosgreSQL (Avec Claude)
>
>`Reverse()` avec Npgsql/PostgreSQL peut avoir un comportement différent de SQL Server. Pour garantir un tri descendant sur toutes les colonnes, utiliser explicitement `OrderByDescending()` et `ThenByDescending()` au lieu de `Reverse()` :
>
>```csharp
>// Au lieu de
>context.Cars.OrderBy(c => c.Color).ThenBy(c => c.PetName).Reverse();
>
>// Utiliser
>context.Cars.OrderByDescending(c => c.Color).ThenByDescending(c => c.PetName);
>```

### Pagination

EF Core offre des fonctionnalités de pagination grâce aux méthodes `Skip()` et `Take()`. `Skip()` ignore le nombre spécifié d'enregistrements, tandis que `Take()` récupère le nombre spécifié d'enregistrements.

L'utilisation de la méthode `Skip()` avec SQL Server exécute une requête avec une commande `OFFSET`. La commande `OFFSET` est l'équivalent, pour SQL Server, de l'omission d'enregistrements qui seraient normalement renvoyés par la requête. Ajoutez la méthode suivante au fichier *Program.cs* :

## Résilience de la connection


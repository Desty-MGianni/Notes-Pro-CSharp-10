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

**EF Core offre des fonctionnalités de pagination grâce aux méthodes `Skip()` et `Take()`. `Skip()` ignore le nombre spécifié d'enregistrements, tandis que `Take()` récupère le nombre spécifié d'enregistrements.**

**L'utilisation de la méthode `Skip()` avec SQL Server et PostgresSQL exécute une requête avec une commande `OFFSET`. La commande `OFFSET` est l'équivalent de l'omission d'enregistrements qui seraient normalement renvoyés par la requête.*** Ajoutez la méthode suivante au fichier *Program.cs* :

```cs
static async Task Paging()
{
    // Cette fabrique n'est pas censée être utilisée ainsi,
    // mais c'est un code de démonstration :-)
    var context = new ApplicationDbContextFactory().CreateDbContext(null);
    Console.WriteLine("**** Paging ****");
    // Passe les deux premiers enregistrements
    var cars = await context.Cars.Skip(2).ToListAsync();
}
```

L'exemple de code ignore les deux premiers enregistrements et renvoie les suivants. La requête, légèrement modifiée pour plus de clarté, est présentée ici :

```sql
SELECT i."Id", i."Color", i."DateBuilt", i."Display", i."IsDrivable", i."MakeId", i."PetName", i."TimeStamp", i.xmin
      FROM public."Inventory" AS i
      OFFSET @p
```

***==Sur SQL Server, la requête généré ajoute une clause `ORDER BY`, même quand on ne l'a pas ajouté dans la déclaration LINQ. C'est parque que la commande `OFFSET` dans SQL Server ne peux pas être exécutée sans la clause `ORDER BY`.==***

PostgreSQL prend en charge une syntaxe historique et simplifiée où `OFFSET` et `LIMIT` se suffisent à eux-mêmes. Bien qu'il accepte d'exécuter cette requête, **omettre le tri est une mauvaise pratique en production**.

**En base de données, sans clause `ORDER BY` explicite, l'ordre de retour des lignes est dit indéterminé**. PostgreSQL renvoie les lignes selon leur disposition physique sur le disque (le "relation scan").

>[!tip] 
>
>Pour garantir une pagination fiable et identique sur n'importe quel système de gestion de base de données (SQL Server, PostgreSQL, SQLite, etc.), forcez toujours un tri dans votre expression LINQ avant d'utiliser `.Skip()` ou `.Take()`.

**La méthode `Take()` génère une requête utilisant la commande `TOP` pour SQL Server et `LIMIT` pour PostgreSQL.** L'ajout suivant à la méthode `Paging()` utilise la méthode `Clear()` sur `ChangeTracker` pour réinitialiser l'`ApplicationDbContext`, puis la méthode `Take()` pour renvoyer deux enregistrements.

```cs
static async Task Paging()
{
	...
	
	context.ChangeTracker.Clear();
	// Prend les deux premiers enregistrements
	cars = await context.Cars.Take(2).ToListAsync();
}
```

```sql
SELECT i."Id", i."Color", i."DateBuilt", i."Display", i."IsDrivable", i."MakeId", i."PetName", i."TimeStamp", i.xmin
      FROM public."Inventory" AS i
      LIMIT @p
```

>[!note]
>
>Rappelons ([[Chapitre 13#Pagination des données avec des plages (Nouveauté C 10.0)|Chapitre 13]]) qu'avec .NET 6/C# 10, la méthode `Take()` pouvait accepter une plage de valeurs. Cette fonctionnalité n'est pas prise en charge par EF Core.

**La combinaison des méthodes `Skip()` et `Take()` permet la pagination des données.** Par exemple, si vous avez une page de deux éléments (en raison de la petite taille de notre base de données) et que vous devez obtenir la deuxième page, exécutez la requête LINQ suivante :

```cs
static async Task Paging()
{
	...
    // Passe les deux premiers enregistrements et prend les deux suivantes.
    cars = await context.Cars.Skip(2).Take(2).ToListAsync();
}
```

==Lorsqu'il combine `Skip()` et `Take()`, SQL Server n'utilise pas la commande `TOP`, mais une autre version de la commande `OFFSET`==, comme illustré ici :

```sql
SELECT [i].[Id], [i].[Color], [i].[DateBuilt], [i].[Display], [i].[IsDrivable], [i].[MakeId], [i].[PetName], [i].[TimeStamp]
	FROM [dbo].[Inventory] AS [i]
	ORDER BY (SELECT 1)
	OFFSET 2 ROWS FETCH NEXT 2 ROWS ONLY
```

**==Pour PostgreSQL, les commandes `LIMIT` et `OFFSET` sont combinées :==**

```sql
SELECT i."Id", i."Color", i."DateBuilt", i."Display", i."IsDrivable", i."MakeId", i."PetName", i."TimeStamp", i.xmin
      FROM public."Inventory" AS i
      LIMIT @p OFFSET @p
```

## Récupérer un seul enregistrement

**Il existe trois méthodes principales (avec leurs variantes `OrDefault` ) pour récupérer un seul enregistrement lors d'une requête : `First()`/`FirstOrDefault()`, `Last()`/`LastOrDefault()` et `Single()`/`SingleOrDefault()`.** Bien que toutes trois, récupèrent un seul enregistrement, leur fonctionnement diffère. Ces trois méthodes et leurs variantes sont détaillées ici :

- La méthode `First()` renvoie le premier enregistrement correspondant à la condition de la requête et à ses éventuelles clauses de tri. Si aucun tri n'est spécifié, l'enregistrement renvoyé est celui qui figure dans la base de données. Si aucun enregistrement n'est renvoyé, une exception est levée.
- La méthode `FirstOrDefault()` se comporte de la même manière que `First()`, sauf que si aucun enregistrement ne correspond à la requête, elle renvoie la valeur par défaut du type (`null`).
- La méthode `Single()` renvoie le premier enregistrement correspondant à la condition de la requête et à ses éventuelles clauses de tri. Si aucun tri n'est spécifié, l'enregistrement renvoyé est celui qui figure dans la base de données. Si aucun enregistrement ou plusieurs enregistrements correspondent à la requête, une exception est levée.
- La méthode `SingleOrDefault()` se comporte de la même manière que `Single()`, sauf que si aucun enregistrement ne correspond à la requête, elle renvoie la valeur par défaut du type (`null`).
- La méthode `Last()` renvoie le dernier enregistrement correspondant à la condition de la requête et à toute clause de tri. Si aucun tri n'est spécifié, une exception est levée. Si aucun enregistrement n'est renvoyé, une exception est levée.
- Le comportement de `LastOrDefault()` est identique à celui de `Last()`, sauf que si aucun enregistrement ne correspond à la requête, la méthode renvoie la valeur par défaut du type (`null`).

**Toutes les méthodes peuvent également accepter une `Expression<Func<T, bool>>` pour filtrer l'ensemble de résultats. Cela signifie que vous pouvez placer l'expression `Where()` dans l'appel des méthodes `First()`/`Single()` tant qu'il n'y a qu'une seule clause `Where()`. Les instructions suivantes sont équivalentes :**

```cs
Context.Cars.Where(c=>c.Id < 5).First();
Context.Cars.First(c=>c.Id < 5);
```

>[!info] 
>
>Les six méthodes présenté ici **ont aussi leurs variante asynchrone**, illustré par les examples suivants. 

### Utilisation de `First`

Lorsqu'on utilise la forme sans paramètre de `First()` et `FirstOrDefault()`, le premier enregistrement (selon l'ordre de la base de données ou toute clause de tri précédente) est renvoyé. L'exemple suivant récupère le premier enregistrement en fonction de l'ordre de la base de données :

```cs
static async Task SingleRecordQuery()
{
    // Cette fabrique n'est pas censée être utilisée ainsi,
    // mais c'est un code de démonstration :-)
    var context = new ApplicationDbContextFactory().CreateDbContext(null);
    Console.WriteLine("**** Single Record with database sort ****");
    var firstCar = await context.Cars.FirstAsync();
    Console.WriteLine($"{firstCar.PetName} is {firstCar.Color}");
    context.ChangeTracker.Clear();
}
```

La requête LINQ précédente est traduite comme suit :

```sql
SELECT i."Id", i."Color", i."DateBuilt", i."Display", i."IsDrivable", i."MakeId", i."PetName", i."TimeStamp", i.xmin
	FROM public."Inventory" AS i
	LIMIT 1
```

Le code suivant récupère le premier enregistrement en fonction de l'ordre de `Color` :

```cs
static async Task SingleRecordQuery()
{
	...
	
    Console.WriteLine("**** Single Record with OrderBy sort ****");
    var firstCarByColor = await context.Cars.OrderBy(c => c.Color).FirstAsync();
    Console.WriteLine($"{firstCarByColor.PetName} is {firstCarByColor.Color}");
    context.ChangeTracker.Clear();
}
```

La requête LINQ précédente est traduite comme suit :

```sql
SELECT i."Id", i."Color", i."DateBuilt", i."Display", i."IsDrivable", i."MakeId", i."PetName", i."TimeStamp", i.xmin
      FROM public."Inventory" AS i
      ORDER BY i."Color"
      LIMIT 1
```

**Le code suivant montre comment utiliser `First()` avec une clause `Where()` puis comment utiliser `First()` comme clause `Where()`** :

```cs
static async Task SingleRecordQuery()
{
	...
    Console.WriteLine("**** Single Record with Where clause ****");
    var firstCarIdThree = await context.Cars.Where(c => c.Id == 3).FirstAsync();
    Console.WriteLine(
	    $"{firstCarIdThree.PetName} is {firstCarIdThree.Color}"
	);
    context.ChangeTracker.Clear();

    Console.WriteLine("**** Single Record Using First as Where clause ****");
    var firstCarIdThree1 = await context.Cars.FirstAsync(c => c.Id == 3);
    Console.WriteLine(
        $"{firstCarIdThree1.PetName} is {firstCarIdThree1.Color}"
    );
    context.ChangeTracker.Clear();
}

```

Les deux instructions précédentes sont traduites en SQL comme suit :

```sql
SELECT i."Id", i."Color", i."DateBuilt", i."Display", i."IsDrivable", i."MakeId", i."PetName", i."TimeStamp", i.xmin
      FROM public."Inventory" AS i
      WHERE i."Id" = 3
      LIMIT 1
```

**L'exemple suivant montre qu'une exception est levée en l'absence de correspondance lors de l'utilisation de `First()`** :

```cs
static async Task SingleRecordQuery()
{
	...
	
    Console.WriteLine("**** Exception when no Record is found ****");
    try
    {
        var firstCarNotFound = await context.Cars.FirstAsync(c => c.Id == 27);
    }
    catch (InvalidOperationException ex)
    {
        Console.WriteLine(ex.Message);
    }
    context.ChangeTracker.Clear();
}
```

La requête LINQ précédente est traduite comme suit :

```sql
SELECT i."Id", i."Color", i."DateBuilt", i."Display", i."IsDrivable", i."MakeId", i."PetName", i."TimeStamp", i.xmin
      FROM public."Inventory" AS i
      WHERE i."Id" = 27
      LIMIT 1
```

**Lorsqu'on utilise `FirstOrDefault()`, au lieu d'une exception, le résultat est `null` lorsqu'aucune donnée n'est renvoyée.**

```cs
static async Task SingleRecordQuery()
{
	...
	
    Console.WriteLine(
        "**** Return Default (null) when no Record is found ****"
    );
    var firstCarNotFound1 = await context.Cars.FirstOrDefaultAsync(c =>
        c.Id == 27
    );
    Console.WriteLine(firstCarNotFound1 == null);
    context.ChangeTracker.Clear();
}
```

**==La requête LINQ précédente est traduite dans le même SQL que l'exemple précédent.==**

>[!note]
>
>Rappelons ([[Chapitre 13#Définition de la valeur par défaut pour les méthodes `[First/Last/Single]OrDefault` (Nouveauté C 10)|Chapitre 13]]), qu'avec .NET 6/C# 10, les méthodes `OrDefault()` peuvent spécifier une valeur par défaut lorsqu'aucune valeur n'est renvoyée par la requête. Malheureusement, cette fonctionnalité n'est pas prise en charge par EF Core.

### Utilisation de `Last`
Lors de l'utilisation de la forme sans paramètre de `Last()` et `LastOrDefault()`, le dernier enregistrement (basé sur n'importe quel clauses de commande précédentes) seront retournées. **Lors de l'utilisation de `Last()` ou `LastOrDefault()`, la requête LINQ doit avoir une clause  `OrderBy()`/`OrderByDescending()` ou une `InvalidOperationException` sera levée** :

```cs
static async Task SingleRecordQuery()
{
	...
	
    Console.WriteLine("**** Exception with Last and no OrderBy ****");
    try
    {
        await context.Cars.LastAsync();
    }
    catch (InvalidOperationException ex)
    {
        Console.WriteLine(ex.Message);
    }
}
```

Le test suivant obtient le dernier enregistrement en fonction de l'ordre de `Color` :

```cs
static async Task SingleRecordQuery()
{
	...
	
     Console.WriteLine("**** Get Last Record sorted by Color ****");
    var lastCar = await context.Cars.OrderBy(c => c.Color).LastAsync();
    Console.WriteLine($"{lastCar.PetName} is {lastCar.Color}");
    context.ChangeTracker.Clear();
}
```

**EF Core inverse les instructions `ORDER BY` puis prend `TOP(1)` (SQL Server) ou `LIMIT 1` (PostgreSQL) pour obtenir le résultat.** Voici la requête exécutée

```sql
SELECT i."Id", i."Color", i."DateBuilt", i."Display", i."IsDrivable", i."MakeId", i."PetName", i."TimeStamp", i.xmin
      FROM public."Inventory" AS i
      ORDER BY i."Color" DESC
      LIMIT 1
```

### Utilisation de `Single`

Conceptuellement, `Single()`/`SingleOrDefault()` fonctionne de la même manière que `First()`/`FirstOrDefault()`. **Le principal La différence est que `Single()`/`SingleOrDefault()` renvoie `TOP(2)`/`LIMIT 2` au lieu de `TOP(1)`/`LIMIT 1` et lève une exception si deux enregistrements sont renvoyés de la base de données.**

Les tests suivants récupèrent l'enregistrement unique où `Id == 1` :

```cs
static async Task SingleRecordQuery()
{
	...
	
    Console.WriteLine("**** Get Single Record ****");
    var singleCar = await context.Cars.SingleAsync(c => c.Id == 3);
    Console.WriteLine($"{singleCar.PetName} is {singleCar.Color}");
    context.ChangeTracker.Clear();
}
```

La requête LINQ précédente est traduite comme suit :

```sql
SELECT i."Id", i."Color", i."DateBuilt", i."Display", i."IsDrivable", i."MakeId", i."PetName", i."TimeStamp", i.xmin
      FROM public."Inventory" AS i
      WHERE i."Id" = 3
      LIMIT 2
```

*==La fonction `Single()` lève une exception si aucun enregistrement n'est renvoyé ou si plusieurs enregistrements sont renvoyés :==*

```cs
static async Task SingleRecordQuery()
{
	...
	
    Console.WriteLine("**** Exception when more than one Record is found ****");
    try
    {
        await context.Cars.SingleAsync(c => c.Id > 1);
    }
    catch (InvalidOperationException ex)
    {
        Console.WriteLine(ex.Message);
    }
    context.ChangeTracker.Clear();

    Console.WriteLine("**** Exception when no Records are found ****");
    try
    {
        await context.Cars.SingleAsync(c => c.Id == 27);
    }
    catch (InvalidOperationException ex)
    {
        Console.WriteLine(ex.Message);
    }
    context.ChangeTracker.Clear();
}
```

*==Lors de l'utilisation de `SingleOrDefault()`, une exception est également levée si plusieurs enregistrements sont renvoyés :==*

```cs
static async Task SingleRecordQuery()
{
	...
	
    Console.WriteLine("**** Exception when more than one Record is found ****");
    try
    {
        await context.Cars.SingleOrDefaultAsync(c => c.Id > 1);
    }
    catch (InvalidOperationException ex)
    {
        Console.WriteLine(ex.Message);
    }
    context.ChangeTracker.Clear();
}

```

Lors de l'utilisation de `SingleOrDefault()`, au lieu d'une exception, le résultat est `null` lorsqu'aucune donnée n'est renvoyée.

```cs
static async Task SingleRecordQuery()
{
	...
	
    Console.WriteLine(
        "**** Return default (null) when Single car not found ****"
    );
    var defaultSingleNotFoundCar = await context.Cars.SingleOrDefaultAsync(c =>
        c.Id == 27
    );
    Console.WriteLine(defaultSingleNotFoundCar == null);
    context.ChangeTracker.Clear();
}

```

### Utilisation de `Find`

==La méthode `Find()` renvoie également un seul enregistrement, mais son comportement diffère légèrement des autres méthodes de recherche d'enregistrements uniques.== **Les paramètres de la méthode `Find()` représentent la ou les clés primaires de l'entité. Elle recherche ensuite dans le `ChangeTracker` une instance de l'entité correspondant à la clé primaire et la renvoie si elle est trouvée. Sinon, elle effectue un appel à la base de données pour récupérer l'enregistrement.**

```cs
static async Task SingleRecordQuery()
{
	...
	
    Console.WriteLine("**** Search car using Find ****");
    var foundCar = await context.Cars.FindAsync(27);
    Console.WriteLine(foundCar == null);
    context.ChangeTracker.Clear();
}
```

**Si l'entité possède une clé primaire composée, transmettez les valeurs représentant cette clé composée :**

```cs
var item = context.MaClasseAvecCléComposite.Find(27, 3);
```

## Méthodes d'agrégation

**EF Core prend également en charge les méthodes d'agrégation côté serveur (`Max()`, `Min()`, `Count()` et `Average()`). Toutes les méthodes d'agrégation peuvent être utilisées conjointement avec les méthodes `Where()` et renvoient une seule valeur. Les requêtes d'agrégation s'exécutent côté serveur. Les filtres de requête globaux affectent également les méthodes d'agrégation et peuvent être désactivés avec `IgnoreQueryFilters()`. Les filtres de requête globaux sont abordés plus loin dans ce chapitre.**

**Notez que chaque méthode d'agrégation est une fonction terminale. Autrement dit, elles mettent fin à l'instruction LINQ lors de leur exécution, car chaque méthode renvoie une seule valeur numérique. L'exécution de la requête est également immédiate, comme pour les méthodes d'enregistrement unique présentées précédemment.**

==Toutes les instructions SQL présentées dans cette section ont été collectées à l'aide de== :

- SQL Server Profiler (Uniquement SQL Server)
- **La méthode `LogTo` dans les options de `ApplicationDbContext` (Le reste)**

==Nous utiliserons ces outils car la méthode `ToQueryString()` ne fonctionne pas avec l'agrégation.==

Ce premier exemple compte tous les enregistrements de voitures dans la base de données.

>[!info] 
>
>Tous les exemples affiché ici utiliseront leurs variantes `Async` si disponible

```cs
static async Task Aggregation()
{
    // Cette fabrique n'est pas censée être utilisée ainsi,
    // mais c'est un code de démonstration :-)
    var context = new ApplicationDbContextFactory().CreateDbContext(null);
    var count = await context.Cars.CountAsync();
}
```

La requête SQL exécutée est affichée ici :

```sql
SELECT count(*)::int
      FROM public."Inventory" AS i
```

**La méthode `Count()` peut contenir l'expression de filtre, tout comme `First()` et `Single()`.** Les exemples suivants illustrent la méthode `Count()` avec une condition `WHERE`. Le premier ajoute l'expression directement dans la méthode `Count()`, et le second ajoute la méthode `Count()` à la fin de l'instruction LINQ, après la méthode `Where()`.

```cs
static async Task Aggregation()
{
	...
	
    var countByMake = await context.Cars.CountAsync(x => x.MakeId == 1);
    Console.WriteLine($"Count: {countByMake}");
    var countByMake2 = await context
        .Cars.Where(x => x.MakeId == 1)
        .CountAsync();
    Console.WriteLine($"Count: {countByMake2}");
}
```

Les deux lignes de code génèrent les mêmes appels SQL au serveur, comme illustré ici :

```sql
SELECT count(*)::int
      FROM public."Inventory" AS i
      WHERE i."MakeId" = 1
```

**Les exemples suivants présentent les fonctions `Min()`, `Max()` et `Average()`. Chaque méthode prend une expression permettant de spécifier la propriété sur laquelle porte l'opération :**

```cs
static async Task Aggregation()
{
	...
	
    var max = await context.Cars.MaxAsync(x => x.Id);
    var min = await context.Cars.MinAsync(x => x.Id);
    var avg = await context.Cars.AverageAsync(x => x.Id);
    Console.WriteLine($"Max ID: {max} Min ID: {min} Avg ID: {avg}");
}
```

```sql
SELECT max(i."Id")
      FROM public."Inventory" AS i
      
SELECT min(i."Id")
      FROM public."Inventory" AS i
      
SELECT avg(i."Id"::double precision)
      FROM public."Inventory" AS i
```

## Les méthodes `Any()` et `All()`

**Les méthodes `Any()` et `All()` vérifient si un ensemble d'enregistrements correspond aux critères (`Any()`) ou si tous les enregistrements correspondent aux critères (`All()`).** Tout comme les méthodes d'agrégation, la méthode `Any()` (mais pas la méthode `All()`) peut être ajoutée à la fin d'une requête LINQ avec les méthodes `Where()`, ou l'expression de filtre peut être contenue dans la méthode elle-même. Les méthodes `Any()` et `All()` s'exécutent côté serveur et renvoient une valeur `bool`. ***==Ce sont deux fonctions de terminaison.==*** Les filtres de requête globaux affectent également les fonctions `Any()` et `All()` et peuvent être désactivés avec `IgnoreQueryFilters()`.

==La méthode `ToQueryString()` ne fonctionne pas non plus avec les fonctions Any()/All(), c'est pourquoi toutes les instructions SQL présentées dans cette section ont été avec les mêmes moyens que les sections précédentes.==

Ce premier exemple vérifie si des enregistrements de `Car` ont un `MakeId` égal à $1$.

```cs
static async Task AnyAndAll()
{
    // Cette fabrique n'est pas censée être utilisée ainsi,
    // mais c'est un code de démonstration :-)
    var context = new ApplicationDbContextFactory().CreateDbContext(null);

    var resultAny = await context.Cars.AnyAsync(x => x.MakeId == 1);
    // Ceci exécute la même requête que la ligne du dessus
    var resultAnyWithWhere = await context
        .Cars.IgnoreQueryFilters()
        .Where(x => x.MakeId == 1)
        .AnyAsync();

    Console.WriteLine($"Exist? {resultAny}");
    Console.WriteLine($"Exist? {resultAnyWithWhere}");
}
```

Le code SQL exécuté pour le premier exemple est présenté ici :

```sql
SELECT EXISTS (
	  SELECT 1
	  FROM public."Inventory" AS i
	  WHERE i."MakeId" = 1)
```

Ce deuxième exemple vérifie si *tous* les enregistrements de `Car` possèdent un `MakeId` spécifique.

```cs
static async Task AnyAndAll()
{
	...
	
    var resultAll = await context.Cars.AllAsync(x => x.MakeId == 1);
    Console.WriteLine($"All? {resultAll}");
}
```

Le code SQL exécuté pour le premier exemple théorique est présenté ici :

```sql
SELECT NOT EXISTS (
	  SELECT 1
	  FROM public."Inventory" AS i
	  WHERE i."MakeId" <> 1)
```

## Récupération de données à partir de procédures stockées

>[!Attention] Différence importantes entre SQL Server et PostgreSQL
>
>- SQL Server -> `PROCEDURE`
>- PosgreSQL :
>	- Avant PosgresSQL 11 -> `FUNCTION`
>	- Après PostgreSQL 11 -> `PROCEDURE` / `FUNCTION`
>
>Pour plus d'information, voir [[Chapitre 20#Création de la procédure stockée / fonction `GetPetName()`|Chapitre 20]]
>
> **Dans cette exemple, j'utiliserai une `PRODEDURE`, contrairement aux chapitres précédents.**

Le dernier modèle de récupération de données à examiner concerne les procédures stockées. ~~Bien qu'EF Core présente certaines lacunes concernant les procédures stockées (comparé à EF 6),~~ n'oubliez pas qu'EF Core repose sur ADO.NET. Il suffit de revenir à la notion de procédure stockée telle qu'on la concevait avant l'introduction des ORM.

La première étape consiste à créer la procédure stockée dans notre base de données :

```sql
CREATE OR REPLACE PROCEDURE "GetPetName"(
    IN carId INT,
    OUT petName VARCHAR(50)
)
LANGUAGE plpgsql
AS $$
BEGIN
    SELECT "PetName"
    INTO  petName
    FROM "Inventory"
    WHERE "Id" = carId;
END;
$$;
```

La méthode suivante crée les paramètres requis (entrée et sortie), exploite la propriété `Database` de l'objet `ApplicationDbContext` et appelle la fonction `ExecuteSqlRaw()` :

```cs
static async Task<string> GetPetName(ApplicationDbContext context, int id)
{
    var parameterId = new NpgsqlParameter
    {
        ParameterName = "@carId",
        NpgsqlDbType = NpgsqlDbType.Integer,
        Value = id,
    };

    var parameterName = new NpgsqlParameter
    {
        ParameterName = "@petName",
        NpgsqlDbType = NpgsqlDbType.Varchar,
        Size = 50,
        Direction = ParameterDirection.Output,
    };

    var result = await context.Database.ExecuteSqlRawAsync(
        "CALL \"GetPetName\"(@carId, NULL)",
        parameterId,
        parameterName
    );
    return (string)parameterName.Value;
}
```

>[!info] 
>
> Même si le deuxième paramètre dans la chaîne brute SQL de `GetPetName` est `NULL`, Npgsql renvois quand même la valeur de la procédure et utilisera `parameterName` pour cela.

L'étape suivante consiste à récupérer les enregistrements de `Car` (pour obtenir les valeurs d'identification de chaque enregistrement de `Car`), à ​​les parcourir et à utiliser la procédure stockée pour obtenir `PetName` :

```cs
static async Task GetDataFromStoredProc()
{
    // Cette fabrique n'est pas censée être utilisée ainsi,
    // mais c'est un code de démonstration :-)
    var context = new ApplicationDbContextFactory().CreateDbContext(null);
    var cars = context.Cars.IgnoreQueryFilters().ToList();
    foreach (var c in cars)
    {
        Console.WriteLine($"PetName: {await GetPetName(context, c.Id)}");
    }
}
```

>[!note]
>
>Cet exemple est quelque peu artificiel, car récupérer tous les enregistrements de voitures permet également de récupérer les propriétés PetName de ces enregistrements. Récupérer tous les enregistrements est un moyen pratique de démontrer l'appel répété de la procédure stockée.

Lors de l'exécution du code, EF Core exécute la requête SQL suivante pour chaque voiture de la liste (une seule est affichée ici) :

```sql
-- Exécuté qu'une seule fois : on "charge" la table
SELECT i."Id", i."Color", i."DateBuilt", i."Display", i."IsDrivable", i."MakeId", i."PetName", i."TimeStamp", i.xmin
      FROM public."Inventory" AS i
      
...

CALL "GetPetName"(@carId, NULL)
```

### Mise à jour moderne (avec Gemini)

Bien qu'EF Core repose toujours sur [ADO.NET](https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/ado-net-overview) en arrière-plan, il n'est plus aussi nécessaire qu'avant de "revenir à l'ancien temps" et d'écrire du code ADO.NET pur (comme `SqlCommand` ou `SqlDataReader`) pour gérer les procédures stockées

Pour comprendre la situation actuelle, il faut comparer la situation passée à celle d'aujourd'hui :

| Fonctionnalité                  | Époque EF 6                                           | Premières versions EF Core                                      | EF Core 7 / 8 / 9 (Aujourd'hui)                                                                            |
| ------------------------------- | ----------------------------------------------------- | --------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| **Requêtes de sélection**       | Nativement supportées via l'importation de fonctions. | Limitées aux types d'entités connus du modèle (`FromSql`).      | Totalement supportées pour n'importe quel type de données (`FromSql` et `SqlQuery`).                       |
| **Opérations d'écriture (CUD)** | Mappage automatique et transparent via le designer.   | Non supporté. Obligation d'utiliser ADO.NET ou du SQL brut.     | **Support complet** via le mappage fluide `InsertUsingStoredProcedure`, `UpdateUsingStoredProcedure`, etc. |
| **Dépendance ADO.NET**          | Cachée.                                               | Très visible (souvent obligatoire pour contourner les manques). | Optionnelle (réservée aux cas extrêmes comme les paramètres `OUTPUT` multiples).                           |

#### Ce que EF Core fait très bien aujourd'hui (Plus besoin d'ADO.NET)

Depuis les dernières mises à jour de Microsoft, vous pouvez gérer la majorité des scénarios directement dans votre `DbContext` :

##### Mappage automatique du CRUD (Nouveauté EF Core 7+)

Vous pouvez configurer EF Core pour qu'il utilise automatiquement des procédures stockées à la place des requêtes générées lors de l'appel à `SaveChangesAsync()`

```cs
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
modelBuilder.Entity<Produit>()
	.InsertUsingStoredProcedure("InsertProduit", sp => sp.HasParameter(p => p.Nom))
	.UpdateUsingStoredProcedure("UpdateProduit", sp => sp.HasParameter(p => p.Id))
	.DeleteUsingStoredProcedure("DeleteProduit", sp => sp.HasParameter(p => p.Id));
}
```

##### Retour de types non-entités (Nouveauté EF Core 8+)

Auparavant, les procédures de sélection devaient obligatoirement retourner une table correspondant à une classe de votre base de données. Désormais, la méthode `SqlQuery` permet de récupérer des types simples ou des DTO personnalisés sans qu'ils soient enregistrés comme entités : 

```cs
var rapports = context.Database
    .SqlQuery<RapportVentesDto>($"EXECUTE dbo.ObtenirRapportVentes {anne}")
    .ToList();
```

#### Quand reste-t-il vrai qu'il faut "revenir à ADO.NET" ?

La part de vérité de votre affirmation réside dans les scénarios complexes où EF Core atteint encore ses limites :

- **Plusieurs jeux de résultats (Multiple Result Sets) :** Si votre procédure stockée renvoie deux ou trois tableaux de données différents en une seule exécution, EF Core ne sait pas les lire nativement. Vous devez récupérer la connexion ADO.NET sous-jacente via `context.Database.GetDbConnection()` et utiliser un `DbDataReader`.
- **Paramètres de sortie (`OUTPUT`) complexes :** La gestion des paramètres `OUTPUT` directionnels combinés à des jeux de résultats reste souvent plus verbeuse et moins intuitive en EF Core qu'en ADO.NET pur ou avec un micro-ORM comme [Dapper](https://github.com/DapperLib/Dapper).

Avec une procédure SQL Server, vous êtes obligé d'exécuter la procédure telle quelle. Vous ne pouvez pas la filtrer en C# avant l'exécution. 

Avec une fonction PostgreSQL qui retourne une table, **vous pouvez composer votre requête LINQ par-dessus la fonction**. EF Core va fusionner le tout en une seule requête SQL envoyée à PostgreSQL

```cs
// EF Core va générer : SELECT * FROM obtenir_ventes(2026) WHERE montant > 1000 LIMIT 10;
var topVentes = context.Database
    .SqlQuery<VenteDto>($"SELECT * FROM obtenir_ventes({annee})")
    .Where(v => v.Montant > 1000)
    .Take(10)
    .ToList();
```

Npgsql vous permet d'enregistrer votre fonction PostgreSQL directement dans le `DbContext`. Vous pouvez ensuite l'utiliser de manière totalement transparente à l'intérieur de vos requêtes LINQ classiques, comme s'il s'agissait d'une méthode C# native : 

```cs
// 1. Déclaration dans le DbContext
public class MonDbContext : DbContext {
    public double CalculerRemise(int clientId, decimal total) => throw new NotSupportedException();

    protected override void OnModelCreating(ModelBuilder modelBuilder) {        
		modelBuilder.HasDbFunction(typeof(MonDbContext).GetMethod(nameof(CalculerRemise)))
.HasName("calculer_remise_pg"); // Nom de la fonction dans PostgreSQL
    }
}

// 2. Utilisation transparente dans LINQ
var clientsPrivilegues = context.Commandes
    .Where(c => context.CalculerRemise(c.ClientId, c.Total) > 15.0)
    .ToList();
```

Ici, ADO.NET est totalement invisible. EF Core traduit directement `context.CalculerRemise` en appel de fonction PostgreSQL lors de la génération du SQL.

#### Qu'en est-il du CRUD (CUD) automatique ?

Dans la sous-section précédente, j'évoquais `InsertUsingStoredProcedure` (introduit dans EF Core 7).

- **Attention :** Cette API spécifique a été conçue pour les _procédures_ (qui utilisent `CALL` ou `EXEC`).
- Si vos opérations d'écriture reposent sur des _fonctions_ PostgreSQL personnalisées (qui nécessitent un `SELECT * FROM modifier_produit()`), vous ne pourrez pas utiliser directement ces méthodes de configuration fluide du CRUD automatique. Vous devrez exécuter ces fonctions d'écriture manuellement via `context.Database.ExecuteSqlAsync(...)`

# Interrogation des données connexes

**Les propriétés de navigation d'entités permettent de charger les données associées d'une entité. Ces données peuvent être chargées de manière immédiate (une instruction LINQ, une requête SQL), de manière immédiate avec des requêtes fractionnées (une instruction LINQ, plusieurs requêtes SQL), explicitement (plusieurs appels LINQ, plusieurs requêtes SQL) ou différée (une instruction LINQ, plusieurs requêtes SQL à la demande).**

***==Outre la possibilité de charger les données associées via les propriétés de navigation, EF Core corrige automatiquement les entités lors de leur chargement dans le `ChangeTracker`==***. Par exemple, supposons que tous les enregistrements `Make` soient chargés dans la propriété de collection `DbSet<Make>`. Ensuite, tous les enregistrements `Car` sont chargés dans `DbSet<Car>`. **Bien que les enregistrements aient été chargés séparément, ils resteront accessibles les uns aux autres via les propriétés de navigation.**

## Chargement anticipé

**Le *chargement anticipé* désigne le chargement d'enregistrements liés provenant de plusieurs tables en une seule requête de base de données. Ceci est analogue à la création d'une requête T-SQL reliant deux tables ou plus par des jointures.** Lorsque les entités possèdent des propriétés de navigation et que ces propriétés sont utilisées dans les requêtes LINQ, le moteur de traduction utilise des jointures pour obtenir les données des tables liées et charge les entités correspondantes. Cette méthode est généralement beaucoup plus efficace que d'exécuter une requête pour obtenir les données d'une table, puis d'exécuter des requêtes supplémentaires pour chacune des tables liées. **==Pour les cas où l'utilisation d'une seule requête est moins efficace, EF Core 5 a introduit le fractionnement des requêtes, présenté ci-après.==**

**Les méthodes `Include()` et `ThenInclude()` (pour les propriétés de navigation suivantes) permettent de parcourir les propriétés de navigation dans les requêtes LINQ. Si la relation est obligatoire, le moteur de traduction LINQ créera un `INNER JOIN`. Si la relation est facultative, il créera un `LEFT JOIN`.**

Par exemple, pour charger tous les enregistrements de voiture avec leurs informations de marque associées, exécutez la requête LINQ suivante :

```cs
static async Task RelatedData()
{
    // Cette fabrique n'est pas censée être utilisée ainsi,
    // mais c'est un code de démonstration :-)
    var context = new ApplicationDbContextFactory().CreateDbContext(null);

    var carsWithMakes = await context
        .Cars.Include(c => c.MakeNavigation)
        .ToListAsync();
    context.ChangeTracker.Clear();
}
```

La requête LINQ précédente exécute la requête suivante sur la base de données :

```sql
SELECT i."Id", i."Color", i."DateBuilt", i."Display", i."IsDrivable", i."MakeId", i."PetName", i."TimeStamp", i.xmin, m."Id", m."Name", m."TimeStamp", m.xmin
      FROM public."Inventory" AS i
      INNER JOIN public."Makes" AS m ON i."MakeId" = m."Id"
```

>[!note]
>L'instruction `SELECT` renvoie tous les champs des tables `Inventory` et `Makes`. EF Core établit ensuite correctement les liens entre les données, renvoyant le graphe d'objets approprié.

**La propriété `MakeNavigation` est une relation obligatoire car la propriété `MakeId` de l'entité `Car` est non nulle. Étant donné qu'elle est obligatoire, la table `Make` est jointe à la table `Inventory` par une jointure interne (`INNER JOIN`). Si
la propriété de navigation était facultative (`MakeId` était défini comme un `int?`), la jointure serait une jointure externe (`OUTER JOIN`).** L'exemple suivant illustre les relations facultatives dans les requêtes générées.

**Plusieurs instructions `Include()` peuvent être utilisées dans la même requête pour joindre plusieurs entités à l'entité d'origine.** ***==Pour parcourir l'arborescence des propriétés de navigation, utilisez `ThenInclude()` après une instruction `Include()`==***. Par exemple, pour obtenir tous les enregistrements `Make` avec leurs enregistrements `Car` associés et les enregistrements `Driver` provenant de `Cars`, utilisez l'instruction suivante :

```cs
static async Task RelatedData()
{
	...
	
    var makesWithCarsAndDrivers = await context
        .Makes.Include(c => c.Cars)
            .ThenInclude(d => d.Drivers)
        .ToListAsync();
}
```

>[!note]
>
>L’appel à `Clear()` sur `ChangeTracker` est ajouté pour garantir que les exemples de code précédents n’interfèrent pas avec les résultats du code en cours d’analyse.

La requête LINQ précédente exécute la requête suivante sur la base de données :

```sql
SELECT m."Id", m."Name", m."TimeStamp", m.xmin, s0."Id", s0."Color", 
	s0."DateBuilt", s0."Display", s0."IsDrivable", s0."MakeId", s0."PetName", 
	s0."TimeStamp", s0.xmin, s0."InventoryId", s0."DriverId", s0."Id0", 
	s0."TimeStamp0", s0.xmin0, s0."Id00", s0."TimeStamp00", s0.xmin00, 
	s0."FirstName", s0."LastName"
FROM public."Makes" AS m
LEFT JOIN (
	  SELECT i."Id", i."Color", i."DateBuilt", i."Display", i."IsDrivable", 
		  i."MakeId", i."PetName", i."TimeStamp", i.xmin, s."InventoryId", 
		  s."DriverId", s."Id" AS "Id0", s."TimeStamp" AS "TimeStamp0", s.xmin AS 
		  xmin0, s."Id0" AS "Id00", s."TimeStamp0" AS "TimeStamp00", s.xmin0 AS 
		  xmin00, s."FirstName", s."LastName"
	  FROM public."Inventory" AS i
	  LEFT JOIN (
			  SELECT i0."InventoryId", i0."DriverId", i0."Id", i0."TimeStamp", 
				  i0.xmin, d."Id" AS "Id0", d."TimeStamp" AS "TimeStamp0", d.xmin 
				  AS xmin0, d."FirstName", d."LastName"
              FROM public."InventoryToDrivers" AS i0
              INNER JOIN public."Drivers" AS d ON i0."DriverId" = d."Id"
	  ) AS s ON i."Id" = s."InventoryId"
) AS s0 ON m."Id" = s0."MakeId"
ORDER BY m."Id", s0."Id", s0."InventoryId", s0."DriverId"
```

**La présence de la clause `ORDER BY` peut paraître étrange, car la requête LINQ ne comportait aucun ordre. Lors de l'utilisation d'inclusions chaînées (avec les instructions `Include()`/`ThenInclude()`), le moteur de traduction LINQ ajoute une clause `ORDER BY` en fonction de l'ordre des tables incluses et de leurs clés primaires et étrangères. Cet ordre s'ajoute à tout autre ordre spécifié dans la requête LINQ.**

Voici un exemple mis à jour :

```cs
static async Task RelatedData()
{
	...
	
    var orderedMakes = await context
        .Makes.Include(c => c.Cars)
            .ThenInclude(d => d.Drivers)
        .OrderBy(d => d.Name)
        .ToListAsync();
	context.ChangeTracker.Clear();
}
```

Le code SQL généré sera trié selon toutes les clauses de tri de la requête LINQ, suivies des clauses `ORDER BY` générées automatiquement :

```sql
SELECT m."Id", m."Name", m."TimeStamp", m.xmin, s0."Id", s0."Color", 
	s0."DateBuilt", s0."Display", s0."IsDrivable", s0."MakeId", s0."PetName", 
	s0."TimeStamp", s0.xmin, s0."InventoryId", s0."DriverId", s0."Id0", 
	s0."TimeStamp0", s0.xmin0, s0."Id00", s0."TimeStamp00", s0.xmin00, 
	s0."FirstName", s0."LastName"
FROM public."Makes" AS m
LEFT JOIN (
	  SELECT i."Id", i."Color", i."DateBuilt", i."Display", i."IsDrivable", 
			  i."MakeId", i."PetName", i."TimeStamp", i.xmin, s."InventoryId", 
			  s."DriverId", s."Id" AS "Id0", s."TimeStamp" AS "TimeStamp0", 
			  s.xmin AS xmin0, s."Id0" AS "Id00", s."TimeStamp0" AS 
			  "TimeStamp00", s.xmin0 AS xmin00, s."FirstName", s."LastName"
	  FROM public."Inventory" AS i
	  LEFT JOIN (
		    SELECT i0."InventoryId", i0."DriverId", i0."Id", i0."TimeStamp", 
			      i0.xmin, d."Id" AS "Id0", d."TimeStamp" AS "TimeStamp0", d.xmin 
			      AS xmin0, d."FirstName", d."LastName"
			FROM public."InventoryToDrivers" AS i0
            INNER JOIN public."Drivers" AS d ON i0."DriverId" = d."Id"
      ) AS s ON i."Id" = s."InventoryId"
) AS s0 ON m."Id" = s0."MakeId"
ORDER BY m."Name", m."Id", s0."Id", s0."InventoryId", s0."DriverId"
```

### Méthode `Include()` filtrée

==Introduite dans EF Core 5, la fonctionnalité d'inclusion permet de filtrer et de trier les données incluses.== Les opérations autorisées sur la navigation de collection sont : `Where()`, `OrderBy()`, `OrderByDescending()`, `ThenBy()`, `ThenByDescending()`, `Skip()` et `Take()`. Par exemple, pour obtenir tous les enregistrements de marque, mais uniquement les enregistrements de voiture associés dont la couleur est jaune, il suffit de filtrer la propriété de navigation dans l'expression lambda, comme ceci :

```cs
static async Task RelatedData()
{
    var makesWithYellowCars = await context
        .Makes.Include(x => x.Cars.Where(x => x.Color == "Yellow"))
        .ToListAsync();
    context.ChangeTracker.Clear();
}
```

La requête exécutée est la suivante :

```sql
SELECT m."Id", m."Name", m."TimeStamp", m.xmin, i0."Id", i0."Color", 
	   i0."DateBuilt", i0."Display", i0."IsDrivable", i0."MakeId", i0."PetName", 
	   i0."TimeStamp", i0.xmin
FROM public."Makes" AS m
LEFT JOIN (
	  SELECT i."Id", i."Color", i."DateBuilt", i."Display", i."IsDrivable", 
		    i."MakeId", i."PetName", i."TimeStamp", i.xmin
	  FROM public."Inventory" AS i
	  WHERE i."Color" = 'Yellow'
) AS i0 ON m."Id" = i0."MakeId"
ORDER BY m."Id"
```

### Chargement anticipé avec requêtes fractionnées

**Lorsqu'une requête LINQ contient de nombreuses inclusions, cela peut impacter négativement les performances. Pour résoudre ce problème, EF Core 5 a introduit les requêtes fractionnées. Au lieu d'exécuter une seule requête, EF Core divise la requête LINQ en plusieurs requêtes SQL, puis connecte toutes les données associées.** Par exemple, la requête précédente peut être exécutée sous forme de plusieurs requêtes SQL en ajoutant `AsSplitQuery()` à la requête LINQ, comme ceci :

```cs
static async Task RelatedData()
{
	...
 
   var splitMakes = await context
        .Makes.AsSplitQuery()
        .Include(x => x.Cars.Where(x => x.Color == "Yellow"))
        .ToListAsync();
    context.ChangeTracker.Clear();
}
```

Les requêtes exécutées sont affichées ici :

```sql
SELECT i0."Id", i0."Color", i0."DateBuilt", i0."Display", i0."IsDrivable", 
	  i0."MakeId", i0."PetName", i0."TimeStamp", i0.xmin, m."Id"
FROM public."Makes" AS m
INNER JOIN (
	  SELECT i."Id", i."Color", i."DateBuilt", i."Display", i."IsDrivable", 
		  i."MakeId", i."PetName", i."TimeStamp", i.xmin
	  FROM public."Inventory" AS i
	  WHERE i."Color" = 'Yellow'
) AS i0 ON m."Id" = i0."MakeId"
ORDER BY m."Id"
```

*==L'utilisation de requêtes fractionnées présente un inconvénient : si les données changent entre l'exécution des requêtes, les données renvoyées seront incohérentes.==*

### Requêtes plusieurs-à-plusieurs

**La nouvelle prise en charge par EF Core de la conception de tables plusieurs-à-plusieurs s'étend aux requêtes de données avec LINQ. Avant cette prise en charge, les requêtes devaient passer par la table pivot. Désormais, vous pouvez écrire l'instruction LINQ suivante pour obtenir les enregistrements de la `Car` et du conducteur associé :**

```cs
static async Task RelatedData()
{
	...
	
    var carsAndDrivers = await context
        .Cars.Include(x => x.Drivers)
        .Where(x => x.Drivers.Any())
        .ToListAsync();
    context.ChangeTracker.Clear();
}
```

Comme vous pouvez le constater dans la requête SQL `SELECT` générée, EF Core se charge de parcourir le tableau croisé dynamique afin d'associer correctement les enregistrements de `Car` et `Driver` :

```sql
SELECT i."Id", i."Color", i."DateBuilt", i."Display", i."IsDrivable", i."MakeId", 
	  i."PetName", i."TimeStamp", i.xmin, s."InventoryId", s."DriverId", s."Id", 
	  s."TimeStamp", s.xmin, s."Id0", s."TimeStamp0", s.xmin0, s."FirstName", 
	  s."LastName"
FROM public."Inventory" AS i
LEFT JOIN (
	  SELECT i1."InventoryId", i1."DriverId", i1."Id", i1."TimeStamp", i1.xmin, 
		    d0."Id" AS "Id0", d0."TimeStamp" AS "TimeStamp0", d0.xmin AS xmin0, 
		    d0."FirstName", d0."LastName"
	  FROM public."InventoryToDrivers" AS i1
	  INNER JOIN public."Drivers" AS d0 ON i1."DriverId" = d0."Id"
) AS s ON i."Id" = s."InventoryId"
WHERE EXISTS (
	  SELECT 1
	  FROM public."InventoryToDrivers" AS i0
	  INNER JOIN public."Drivers" AS d ON i0."DriverId" = d."Id"
	  WHERE i."Id" = i0."InventoryId"
)
ORDER BY i."Id", s."InventoryId", s."DriverId"
```

## Chargement explicite

**Le chargement explicite consiste à charger des données le long d'une propriété de navigation après le chargement de l'objet principal. Ce processus implique l'exécution d'un appel de base de données supplémentaire pour obtenir les données associées.** ==Cela peut s'avérer utile si votre application a besoin d'obtenir sélectivement les enregistrements associés au lieu de les récupérer systématiquement.==

**Le processus commence avec une entité déjà chargée et utilise la méthode `Entry()` sur le `DbContext` dérivé.** Lors d'une requête sur une propriété de navigation de référence (par exemple, pour obtenir les informations du `Make` d'une voiture), utilisez la méthode `Reference()`. Lors d'une requête sur une propriété de navigation de collection, utilisez la méthode `Collection()`. **La ​​requête est différée jusqu'à l'exécution de `Load()`, `ToList()` ou d'une fonction d'agrégation (par exemple, `Count()`, `Max()`).**

Les exemples suivants montrent comment obtenir la donnée `Make` associées ainsi que les `Driver` associés à un enregistrement de  `Car` :

```cs
static async Task RelatedData()
{
	...
	
    // Récupère l'enregistrment Car
    var car = await context.Cars.FirstAsync(x => x.Id == 1);
    // Récupère l'information de Make
    await context.Entry(car).Reference(c => c.MakeNavigation).LoadAsync();
    // Récupère n'importe quel Driver lié à l'enregistrment Car
    await context.Entry(car).Collection(c => c.Drivers).Query().LoadAsync();
    context.ChangeTracker.Clear();
}
```

Les instructions précédentes génèrent les requêtes suivantes :

```sql
-- Récupère l'enregistrement Car
SELECT i."Id", i."Color", i."DateBuilt", i."Display", i."IsDrivable", i."MakeId", 
	  i."PetName", i."TimeStamp", i.xmin
FROM public."Inventory" AS i
WHERE i."Id" = 1
LIMIT 1
      
-- Récupère les informations de Make      
SELECT m."Id", m."Name", m."TimeStamp", m.xmin
FROM public."Makes" AS m
WHERE m."Id" = @p
LIMIT 1
     
-- Récupère n'importe quel Driver lié à l'enregistrement Car 
SELECT s."Id", s."TimeStamp", s.xmin, s."FirstName", s."LastName", i."Id", 
	  s."InventoryId", s."DriverId", s0."InventoryId", s0."DriverId", s0."Id", 
	  s0."TimeStamp", s0.xmin, s0."Id0", s0."Color", s0."DateBuilt", 
	  s0."Display", s0."IsDrivable", s0."MakeId", s0."PetName", s0."TimeStamp0", 
	  s0.xmin0
FROM public."Inventory" AS i
INNER JOIN (
	  SELECT d."Id", d."TimeStamp", d.xmin, i0."InventoryId", i0."DriverId", 
		    d."FirstName", d."LastName"
	  FROM public."InventoryToDrivers" AS i0
	  INNER JOIN public."Drivers" AS d ON i0."DriverId" = d."Id"
) AS s ON i."Id" = s."InventoryId"
LEFT JOIN (
	  SELECT i1."InventoryId", i1."DriverId", i1."Id", i1."TimeStamp", i1.xmin, 
		    i2."Id" AS "Id0", i2."Color", i2."DateBuilt", i2."Display", 
		    i2."IsDrivable", i2."MakeId", i2."PetName", i2."TimeStamp" AS 
		    "TimeStamp0", i2.xmin AS xmin0
	  FROM public."InventoryToDrivers" AS i1
	  INNER JOIN public."Inventory" AS i2 ON i1."InventoryId" = i2."Id"
	  WHERE i2."Id" = @p
) AS s0 ON s."Id" = s0."DriverId"
WHERE i."Id" = @p
ORDER BY i."Id", s."InventoryId", s."DriverId", s."Id", s0."InventoryId", s0."DriverId"
```

***==Comme vous pouvez le constater, cette troisième et dernière requête effectue un travail considérable pour simplement obtenir les enregistrements de conducteur associés à l'enregistrement de voiture sélectionné.==*** Cela met en évidence deux points importants : 

1) Si vous pouvez tout écrire dans une seule requête en utilisant le chargement anticipé, il est généralement préférable de le faire, ce qui évite d'avoir à retourner à la base de données pour obtenir les enregistrements associés 
2) EF Core ne génère pas toujours les requêtes les plus performantes. Je vous ai déjà montré comment utiliser le chargement anticipé dans la section précédente. 

**Plus loin dans ce chapitre, je vous montrerai comment utiliser des instructions SQL avec ou sans instructions LINQ supplémentaires pour extraire des données de la base de données. Ceci est utile lorsque EF Core génère des requêtes sous-optimales.**

## Chargement différé

>[!tip]- Une alternative à la méthode utilisé par l'auteur existe (Avec Gemini)
>
> Microsoft a introduit une alternative majeure qui vous permet d'utiliser le Lazy Loading (chargement différé) **sans aucun proxy** (c'est-à-dire sans modifier la nature de vos classes d'entités avec du code généré dynamiquement) :
> 
> **Le Lazy Loading sans Proxies (Via Injection de Dépendances)**
>
>Cette méthode utilise le type `ILazyLoader` injecté directement dans le constructeur de votre entité. Vos classes restent des classes C# standards (sans avoir besoin de marquer toutes les propriétés en `virtual`).
>
> Pour plus de détail, voir la [documentation Microsoft](https://learn.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.infrastructure.ilazyloader?view=efcore-10.0)
>>[!attention] 
>>
>> Cette atlernative est extrêmement plus verbeuse (utilisation de champs privés avec des propriétés possédant des accesseurs manuels) et est donc largement moins utilisé.
>>
>> Cette manière de faire existe principalement pour résoudres les trois problèmes suivant :
>> 
>> 1. **Le cauchemar de la Sérialisation (JSON)**
>> 
>>	C'est la raison numéro un. Lorsque vous utilisez des proxies, EF Core ne vous renvoie pas un objet `Car`. Il crée dynamiquement en mémoire une classe masquée qui hérite de votre classe, du style `Castle.Proxies.CarProxy`.
>>
>>	- Si vous essayez de sérialiser cet objet en JSON pour l'envoyer à une API Web (avec `System.Text.Json`), le sérialiseur plante souvent ou lève une exception de boucle infinie.
>>	- Avec `ILazyLoader`, l'objet reste une vraie instance de `Car`. La sérialisation se passe sans aucun problème.
>>
>>2. **La pureté du Domain-Driven Design (DDD)**
>>
>>	Dans les architectures logicielles strictes (comme l'architecture hexagonale ou le DDD), le modèle métier ne doit pas être pollué par des exigences techniques de la base de données.
>>
>> 	- Forcer un développeur à mettre `virtual` partout uniquement "pour faire plaisir à l'ORM" est considéré par certains architectes comme une mauvaise pratique.
>> 	- L'alternative `ILazyLoader` permet de garder des classes scellées (`sealed`) ou d'éviter le polymorphisme forcé par `virtual`.
>> 
>> 3. **Les performances et les contraintes du compilateur**
>> 
>> 	La création de proxies dynamiques au runtime a un coût en termes de mémoire et de CPU, car le framework doit générer du code IL (Intermediate Language) à la volée. De plus, certaines plateformes ou configurations de compilation modernes (comme le mode **AOT - Ahead-Of-Time**, très mis en avant dans .NET 8/9 pour le cloud-native) **interdisent** la génération de code dynamique au runtime. Les proxies y sont donc tout simplement inutilisables.

**Le chargement différé consiste à charger un enregistrement à la demande lorsqu'une propriété de navigation est utilisée pour accéder à un enregistrement associé qui n'est pas encore chargé en mémoire.** Le chargement différé est une fonctionnalité d'EF 6 qui a été réintégrée à EF Core avec la version 2.1. ==Bien qu'il puisse sembler judicieux de l'activer, son activation peut entraîner des problèmes de performance dans votre application en effectuant des allers-retours potentiellement inutiles vers votre base de données.== Le chargement différé peut être utile dans les applications clientes intelligentes (WPF, WinForms), mais **son utilisation est déconseillée dans les applications web ou de service. C'est pourquoi le chargement différé est désactivé par défaut dans EF Core (il est maintenant activé par défaut depuis EF 6).**

**Pour utiliser le chargement différé, les propriétés de navigation à charger de manière différée doivent être marquées comme `virtual`. En effet, les propriétés de navigation seront encapsulées dans un proxy. Ce proxy permettra alors à EF Core d'effectuer un appel à la base de données si la propriété de navigation n'a pas été chargée lorsqu'elle est référencée dans votre application.**

**Pour utiliser le chargement différé avec des proxys, le `DbContext` dérivé doit être correctement configuré. Commencez par ajouter le package `Microsoft.EntityFrameworkCore.Proxies` à votre projet. Vous devez ensuite activer l'utilisation des proxys de chargement différé dans les options du `DbContext` dérivé.** ==Bien que cela soit normalement configuré dans votre code d'application lors de la configuration de votre `DbContext` dérivé, nous allons activer les proxys à l'aide de la classe `ApplicationDbContextFactory` que nous avons créée précédemment. N'oubliez pas que cette classe est conçue pour la conception et ne doit pas être utilisée dans votre code d'application. Cependant, pour l'apprentissage et l'exploration, elle fonctionnera parfaitement.==

Ouvrez le fichier *ApplicationDbContextFactory.cs* et accédez à la méthode `CreateDbContext()`. Nous utiliserons le paramètre `args` pour indiquer que nous souhaitons que la méthode renvoie un `DbContext` dérivé configuré pour utiliser les proxys de chargement différé. Mettez à jour la méthode `CreateDbContext()` comme suit :

```cs
public ApplicationDbContext CreateDbContext(string[] args)
{
	...
	
   if (
		args != null
		&& args.Length == 1
		&& args[0].Equals("lazy", StringComparison.OrdinalIgnoreCase)
	)
	{
		optionBuilder = optionBuilder.UseLazyLoadingProxies();
	}
	optionBuilder = optionBuilder.UseNpgsql(
		conStringBuilder.ConnectionString
	);
	return new ApplicationDbContext(optionBuilder.Options);
}
```

Ensuite, mettez à jour la classe `Car` comme suit :

```cs
namespace AutoLot.Samples.Models;

[Table("Inventory", Schema = "public")]
[Index(nameof(MakeId), Name = "IX_Inventory_MakeId")]
[EntityTypeConfiguration(typeof(CarConfiguration))]
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

    [DatabaseGenerated(DatabaseGeneratedOption.Computed)]
    public string Display { get; set; }

    [ForeignKey(nameof(MakeId))]
    public virtual Make MakeNavigation { get; set; }
    public virtual Radio RadioNavigation { get; set; }


    public DateTime? DateBuilt { get; set; }
    private bool? _isDrivable;
    public bool IsDrivable
    {
        get => _isDrivable ?? true;
        set => _isDrivable = value;
    }

    [InverseProperty(nameof(Driver.Cars))]
    public virtual IEnumerable<Driver> Drivers { get; set; } =
        new List<Driver>();
        
    [InverseProperty(nameof(CarDriver.CarNavigation))]
    public virtual IEnumerable<CarDriver> CarDrivers { get; set; } =
        new List<CarDriver>();
}
```

**Maintenant que les propriétés sont déclarées `virtual`, elles peuvent être utilisées avec le chargement différé.** Ajoutez la méthode suivante à votre fichier *Program.cs* (notez que nous n'utilisons pas encore le paramètre `args` de la méthode `CreateDbContext()`) et appelez-la depuis vos instructions de niveau supérieur :

```cs
static async Task LazyLoadCar()
{
    // Cette fabrique n'est pas censée être utilisée ainsi,
    // mais c'est un code de démonstration :-)
    var context = new ApplicationDbContextFactory().CreateDbContext(null);

    var query = context.Cars.AsQueryable();
    var cars = await query.ToListAsync();
    var make = cars[0].MakeNavigation;
    Console.WriteLine(make.Name);
}
```

*==Lorsque vous exécuterez cet exemple, vous recevrez une exception de référence nulle en tentant d'accéder à la propriété `Name` de l'instance `Make`.==* Cela est dû au fait que l'enregistrement `Make` n'a pas été chargé et que nous n'utilisons pas la version compatible avec le proxy de la classe `DbContext()` dérivée. Mettez à jour la méthode pour transmettre l'argument `"lazy"` à `CreateDbContext()`, ce qui active la prise en charge du chargement différé par proxy :

```cs
// Collection expression (C# 12)
var context = new ApplicationDbContextFactory().CreateDbContext(["lazy"]);
```

Lorsque vous exécuterez à nouveau le code, vous pourriez être surpris de recevoir une exception `InvalidOperationException`. **Lorsque vous utilisez des proxys chargés en différés, *toutes* les propriétés de navigation des modèles doivent être marquées comme virtuelles, même celles qui ne sont pas directement impliquées dans le bloc de code en cours d'exécution.** Jusqu'à présent, nous n'avons mis à jour que l'entité Car. Mettez à jour les autres modèles comme suit :

```cs
// CarDriver.cs
[Table("InventoryToDrivers", Schema = "public")]
public class CarDriver : BaseEntity
{
    public int DriverId { get; set; }

    [ForeignKey(nameof(DriverId))]
    public virtual Driver DriverNavigation { get; set; }

    [Column("InventoryId")]
    public int CarId { get; set; }

    [ForeignKey(nameof(CarId))]
    public virtual Car CarNavigation { get; set; }
}
```

```cs
// Driver.cs
[Table("Drivers", Schema = "public")]
[EntityTypeConfiguration(typeof(DriverConfiguration))]
public class Driver : BaseEntity
{
    public Person PersonInfo { get; set; } = new Person();

    [InverseProperty(nameof(Car.Drivers))]
    public virtual IEnumerable<Car> Cars { get; set; } = new List<Car>();

    [InverseProperty(nameof(CarDriver.DriverNavigation))]
    public virtual IEnumerable<CarDriver> CarDrivers { get; set; } =
        new List<CarDriver>();
}
```

```cs
// Make.cs
[Table("Makes", Schema = "public")]
public class Make : BaseEntity
{
    [Required, StringLength(50)]
    public string Name { get; set; }

    [InverseProperty(nameof(Car.MakeNavigation))]
    public virtual IEnumerable<Car> Cars { get; set; } = new List<Car>();
}
```

```cs
[Table("Radios", Schema = "public")]
[EntityTypeConfiguration(typeof(RadioConfiguration))]
public class Radio : BaseEntity
{
    public bool HasTweeters { get; set; }
    public bool HasSubWoofers { get; set; }

    [Required, StringLength(50)]
    public string RadioId { get; set; }

    [Column("InventoryId")]
    public int CarId { get; set; }

    [ForeignKey(nameof(CarId))]
    public virtual Car CarNavigation { get; set; }
}
```

>[!note]
>
Bien que la classe `Owned Person` représente une relation, il ne s'agit pas d'une propriété de navigation, et les propriétés de type `Owned` n'ont pas besoin d'être marquées comme `virtual`.

Maintenant, lorsque vous exécuterez à nouveau le programme, la marque de la voiture s'affichera dans la console. En observant l'activité du serveur SQL, vous pourrez clairement constater que deux requêtes ont été exécutées :

```sql

```

Si vous souhaitez en savoir plus sur le chargement différé et son utilisation avec EF Core, consultez la [documentation](https://docs.microsoft.com/en-us/ef/core/querying/related-data/lazy).

## Résilience de la connection


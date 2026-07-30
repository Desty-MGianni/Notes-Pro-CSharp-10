---
publish: false
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

En exécutant à nouveau le programme, vous verrez la sortie suivante dans la console. Après l'ajout de l'entité au Suivi des modifications (à l'aide de la méthode Add()), son état est passé à Ajouté. Le message concernant l'enregistrement des modifications provient du gestionnaire d'événements SavingChanges, et le message « 1 entité enregistrée » provient du gestionnaire d'événements SavedChanges. Après l'appel à SaveChanges() sur le contexte, l'état de l'entité est passé à Inchangé.

```
State of the BMW is Detached
State of the BMW is Added
Saving changes for ...
Saved 1 entities
State of the BMW is Unchanged
```

# Résilience de la connection



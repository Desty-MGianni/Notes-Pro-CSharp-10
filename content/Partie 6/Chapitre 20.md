---
title: "Chapitre 20: Accès aux données avec ADO.NET"
publish: true
---

# <big><big><big><b><font color =green>Accès aux Données avec ADO.NET</font></b></big></big></big>

>[!Attention]- ADO.NET est une fondation invisible
>
>Dans le monde professionnel, plus personne n'écrit du code ADO.NET brut pour créer des applications (comme ouvrir manuellement une connexion, écrire une chaîne SQL en dur et boucler sur les lignes). [[1](https://www.c-sharpcorner.com/blogs/overview-of-entity-framework)]
>
>Cependant, **ADO.NET n'est pas obsolète** : c'est la brique de base fondamentale sur laquelle sont construits les outils modernes que vous utiliserez tous les jours, comme **Entity Framework Core (EF Core)** ou **Dapper**. [[1](https://www.sciencedirect.com/science/chapter/edited-volume/pii/B9781928994503500135), [2](https://www.devart.com/blog/ado-net-vs-dapper.html), [3](https://www.devart.com/blog/ado-net-vs-entity-framework.html), [4](https://www.devart.com/dotconnect/getting-started-with-ef-core.html)]
>
>## Ce qui est toujours d'actualité (Le Mode Connecté)
>
>La première moitié du chapitre 20 (généralement axée sur le **mode connecté**) est indispensable à comprendre pour votre culture de développeur :
>
>- **Les Fournisseurs de données (_Data Providers_) :** Des classes comme `SqlConnection` ou `SqlCommand` sont toujours utilisées par les ORM pour dialoguer avec SQL Server, MySQL ou PostgreSQL. [[1](https://dev.to/maurizio8788/the-comprehensive-guide-to-entity-framework-core-l5a), [2](https://www.codemag.com/Article/2207021/Simplifying-ADO.NET-Code-in-.NET-6-Part-1)]
>- **Le `DbDataReader` :** C'est l'outil le plus rapide de .NET pour lire de gros volumes de données de manière asynchrone et linéaire. Si vous devez optimiser une importation de données ultra-rapide, vous utiliserez un `DataReader`. [[1](https://stackoverflow.com/questions/59265580/c-sharp-acces-first-recorset-item-from-adodb), [2](https://www.scribd.com/document/854562842/Unit-V-Introduction)]
>- **La gestion des transactions :** Comprendre comment sécuriser des écritures avec `DbTransaction` reste une compétence clé.
>
>## Ce qui n'est plus d'actualité (Le Mode Déconnecté)
>
>La seconde moitié du chapitre (axée sur le **mode déconnecté**) a très mal vieilli et ne doit pas retenir toute votre attention :
>
>- **La `DataTable` et le `DataSet` :** Ces objets (que vous avez croisés brièvement au chapitre 11 avec les indexeurs en cascade) servent à recréer une mini base de données en mémoire RAM. Ils sont aujourd'hui considérés comme des monstres de verbosité, très lourds pour la mémoire RAM.
>- **Le `SqlDataAdapter` :** Cet outil de synchronisation automatique entre la RAM et la base de données est totalement remplacé par la puissance d'EF Core. [[1](https://www.youtube.com/watch?v=ty-UGMqbYxA), [2](https://www.simplilearn.com/ado-net-interview-questions-answers-article)]
>
>## Le verdict pour le chapitre
>
>1. **Faites les exercices sur le Mode Connecté** (`SqlConnection`, `SqlCommand`, `SqlDataReader`) : cela vous permettra de comprendre la plomberie réseau et la sécurité (comme éviter les injections SQL avec les paramètres).
>2. **Survolez le Mode Déconnecté** (`DataSet`, `DataTable`) sans vous y attarder : considérez-le comme de l'histoire ancienne nécessaire pour maintenir du vieux code en entreprise. [[1](https://www.slideshare.net/slideshow/ado-net-81972565/81972565)]
>3. **Gardez en tête la suite logique** : Tout ce que vous apprenez ici servira à comprendre pourquoi **EF Core** (que le livre aborde généralement juste après) est une bénédiction qui vous évitera d'écrire cette plomberie manuelle. [[1](https://www.simplilearn.com/tutorials/asp-dot-net-tutorial/ado-dot-net)]

>[!important] Modification des technologie utilisés avec ADO.NET
>
>Le livre utilise *Docker* ainsi que *Micorosft SQL Server* et *Azure Data Studio*
>
> **Comme je suis sur macOS / Linux, je vais toujours utilisé** *Docker* **mais changé la base de donnée par** *PostgreSQL* **car c'est beaucoup plus simple.**
>
> En plus, **je vais utilise une deuxième machine (sous Arch Linux) pour évité de consommer la RAM de mon mac et ainsi avoir une architecture plus proche de l'industrie.**
>
> Aux sections du livre décrivant les marches à suivre pour installer les différents programmes, un lien vers [[Note annexe au chapitre 20|la note annexe]] sera affiché.
>
>>[!success] *Sql Server* et *PostgreSQL* s'utilisent de la même manière (avec Gemini)
>>
>>Comme on verra plus tard dans ce chapitre, **tout fournisseur de base de données DOIVENT implémentent exactement les mêmes interfaces, partager la même logique et posséder les mêmes membres**. Le mot `Connection` (tout comme `Command` ou `DataReader`) est imposé par les contrats de Microsoft.
>>
>>- `NpgsqlConnection` ➔ hérite de `DbConnection` 
>>	- ➔ qui implémente `IDbConnection`, `IDisposable` et `IAsyncDisposable`
>>- `SqlConnection` ➔ hérite de `DbConnection` 
>>	- ➔ qui implémente `IDbConnection`, `IDisposable` et `IAsyncDisposable`
>>
>>Puisqu'elles respectent le même contrat, vous retrouverez rigoureusement les mêmes méthodes et propriétés avec la même logique de fonctionnement : 
>>
>>- **Pour ouvrir / fermer :** `.Open()`, `.Close()`, `.OpenAsync()`, `.CloseAsync()`
>>- **Pour l'état :** La propriété `.State` (qui renvoie l'énumération `ConnectionState.Open` ou `Closed`).
>>- **Pour créer un ordre :** `.CreateCommand()` (qui renvoie un `DbCommand`).
>>- **Pour la sécurité :** `.BeginTransaction()`

==La plateforme .NET définit plusieurs espaces de noms permettant d'interagir avec les systèmes de bases de données relationnelles==. ***==Ces espaces de noms sont collectivement appelés ADO.NET==***. Dans ce chapitre, **vous découvrirez le rôle global d'ADO.NET, ses principaux types et espaces de noms, puis vous aborderez les fournisseurs de données ADO.NET**. **==La plateforme .NET prend en charge de nombreux fournisseurs de données (intégrés au framework .NET et disponibles auprès de fournisseurs tiers), chacun étant optimisé pour communiquer avec un système de gestion de base de données spécifique==** (par exemple, Microsoft SQL Server, PostgreSQL, Oracle et MySQL).

Après avoir compris les fonctionnalités communes offertes par les différents fournisseurs de données, **vous étudierez le modèle de conception « fabrication de fournisseurs de données »**. Comme vous le verrez, **==en utilisant les types des espaces de noms `System.Data`,==** (notamment `System.Data.Common` et les espaces de noms spécifiques aux fournisseurs de bases de données comme `Npgsql` pour PostgreSQL,   `Microsoft.Data.SqlClient`, `System.Data.Odbc` et l'espace de noms `System.Data.Oledb`, propre à Windows), **==vous pouvez créer une base de code unique capable de sélectionner dynamiquement le fournisseur de données sous-jacent sans avoir besoin de recompiler ni de redéployer le code de l'application.==**

Vous apprendrez ensuite à ==interagir directement avec le fournisseur de base de données, à créer et ouvrir des connexions pour récupérer des données, puis à insérer, mettre à jour et supprimer des données, avant d'aborder les transactions de base de données==. Enfin, **vous exécuterez la fonctionnalité de copie en bloc de SQL Server / PostgreSQL à l'aide d'ADO.NET pour charger une liste d'enregistrements dans la base de données.**

> [!warning] Grille de Lecture Moderne (Postgres + .NET 10/11)
> 
> * **Technologie Clé :** Remplacer spirituellement `Microsoft.Data.SqlClient` par `Npgsql`.
> * **À Ignorer :** Les sections sur `Oledb` et `Odbc` (obsolètes et/ou spécifiques à Windows).
> * **Modernisation obligatoire :** Toutes les opérations réseau (Connexions, Commandes, Lecteurs) doivent être implémentées avec leurs variantes asynchrones `Async / await`.
> * **Équivalence Bulk Copy :** Le `SqlBulkCopy` du livre se traduira par l'utilisation de `NpgsqlBinaryImporter` (Commande SQL `COPY` de PostgreSQL).

>[!note]
>Ce chapitre est consacré à ADO.NET en détail. À partir du [[chapitre 21#|Chapitre 21 A FAIREEEEE]], j'aborde Entity Framework (EF) Core, le framework de mappage objet-relationnel (ORM) de Microsoft. EF Core utilisant ADO.NET en interne pour l'accès aux données, une solide compréhension de son fonctionnement est essentielle pour le dépannage de ces accès. Certains scénarios ne sont pas pris en charge par EF Core (comme l'exécution d'une copie SQL en bloc), et vous devrez connaître ADO.NET pour les résoudre.

# ADO.NET vs. ADO

Si vous avez une expérience du modèle d'accès aux données COM précédent de Microsoft (Active Data Objects [ADO]) et que vous débutez avec la plateforme .NET, ==il est important de comprendre qu'ADO.NET n'a que peu de points communs avec ADO, mis à part les lettres *A*, *D* et *O*==. Bien qu'il existe une certaine relation entre les deux systèmes (par exemple, les concepts d'objets de connexion et de commande), certains types ADO familiers (comme l'objet `Recordset`) n'existent plus. De plus, vous trouverez de nombreux nouveaux types sans équivalent direct dans ADO classique (comme l'adaptateur de données).

# Comprendre les fournisseurs de données ADO.NET

**ADO.NET ne fournit pas un ensemble unique d'objets permettant de communiquer avec plusieurs systèmes de gestion de bases de données** ( *SGDB* ou *DBMS* en anglais). ==ADO.NET prend plutôt en charge plusieurs *fournisseurs de données*, chacun optimisé pour interagir avec un DBMS spécifique==. Le premier avantage de cette approche est la **possibilité de programmer un fournisseur de données spécifique pour accéder aux fonctionnalités uniques d'un SGBD particulier**. Le second avantage est qu'**un fournisseur de données spécifique peut se connecter directement au moteur sous-jacent du SGBD en question, sans couche de mappage intermédiaire.**

***==En résumé, un fournisseur de données est un ensemble de types définis dans un espace de noms donné, capables de communiquer avec un type spécifique de source de données==***. **Quel que soit le fournisseur de données utilisé, chacun définit un ensemble de classes fournissant les fonctionnalités de base**. Le [[#Tableau 20-1 Les objets clé d'un fournisseur de donnée ADO.NET|Tableau 20-1]] présente certaines des classes de base et les interfaces clés qu'elles implémentent.

###### Tableau 20-1: Les objets clé d'un fournisseur de donnée ADO.NET

| Classe de base  | Interfaces pertinente                   | Description                                                                                                                                                                                                                                                               |
| --------------- | --------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `DbConnection`  | `IDbConnection`                         | Permet de se connecter à la base de données et de s'en déconnecter. Les objets de connexion donnent également accès à un objet de transaction associé.                                                                                                                    |
| `DbCommand`     | `IDbCommand`                            | Représente une requête SQL ou une procédure stockée. Les objets de commande permettent également d'accéder à l'objet lecteur de données du fournisseur.                                                                                                                   |
| `DbDataReader`  | `IDataReader`,<br>`IDataRecord`         | Permet un accès en lecture seule et en avant seule aux données à l'aide d'un curseur côté serveur.                                                                                                                                                                        |
| `DbDataAdapter` | `IDataAdapter`,<br>`IDbDataAdapter`     | Transfère des objets `DataSet` entre l'appelant et la base de données. Les adaptateurs de données comprennent une connexion et quatre objets de commande internes permettant de sélectionner, insérer, mettre à jour et supprimer des informations de la base de données. |
| `DbParameter`   | `IDataParameter`,<br>`IDbDataParameter` | Représente un paramètre nommé dans une requête paramétrée.                                                                                                                                                                                                                |
| `DbTransaction` | `IDbTransaction`                        | Encapsule une transaction de base de données.                                                                                                                                                                                                                             |

>[!success] Le paragraphe suivant est le plus important à comprendre

***Bien que les noms spécifiques de ces classes principales diffèrent selon les fournisseurs de données (par exemple, `SqlConnection` et `OdbcConnection`), chaque classe hérite de la même classe de base (`DbConnection`, dans le cas des objets de connexion) qui implémente des interfaces identiques (par exemple, `IDbConnection`). De ce fait, vous pouvez raisonnablement supposer qu'une fois que vous aurez appris à utiliser un fournisseur de données, les autres seront relativement simples à appréhender.***

>[!note]
>Lorsque vous faites référence à un objet de connexion sous ADO.NET, vous faites en réalité référence à un type spécifique dérivé de `DbConnection` ; **il n’existe pas de classe nommée** *Connection*. **Il en va de même pour un** *objet de commande*, **un** *objet adaptateur de données*, etc. **==Par convention, les objets d’un fournisseur de données donné sont préfixés par le nom du SGBD associé (par exemple, `SqlConnection`, `SqlCommand` et `SqlDataReader`).

**L'image suivante présente le fonctionnement global des fournisseurs de données ADO.NET. L'assembly client peut être de tout type application .NET : programme console, Windows Forms, WPF/Avalonia, ASP.NET Core, bibliothèque de code .NET, etc.**

![[Figure 20.1.png|Les fournisseurs de données ADO.NET permettent d'accéder à un SGBD donné.]]

**Un fournisseur de données vous fournira d'autres types d'objets que ceux présentés dans l'image précédente ; cependant, ces objets de base définissent un socle commun à tous les fournisseurs de données.**

## Fournisseurs de données ADO.NET

**Comme pour l'ensemble des composants .NET, les fournisseurs de données sont distribués sous forme de packages NuGet**. Plusieurs sont pris en charge par Microsoft, ainsi qu'une multitude de fournisseurs tiers. Le tableau 20-2 répertorie certains des fournisseurs de données pris en charge par Microsoft.

###### Tableau 20-2: Certains fournisseurs de données pris en charge par Microsoft


| Fournisseur de données     | Espace de noms/Nom du package NuGet |
| -------------------------- | ----------------------------------- |
| Microsoft SQL Server       | `Microsoft.Data.SqlClient`          |
| ODBC                       | `System.Data.Odbc`                  |
| OLE DB (Windows seulement) | `System.Data.OleDb`                 |
| **PostgresSQL**            | `Npgsql`                            |

Le fournisseur de données Microsoft SQL Server offre un accès direct aux magasins de données Microsoft SQL Server, et uniquement aux magasins de données SQL Server (y compris SQL Azure). L'espace de noms `Microsoft.Data.SqlClient` contient les types utilisés par le fournisseur SQL Server.

>[!note]
>Bien que System.`Data.SqlClient` soit toujours pris en charge, tous les efforts de développement pour l'interaction avec SQL Server (et SQL Azure) sont concentrés sur la nouvelle bibliothèque fournisseur `Microsoft.Data.SqlClient`.

Le fournisseur ODBC (`System.Data.Odbc`) permet d'accéder aux connexions ODBC. **Les types ODBC définis dans l'espace de noms `System.Data.Odbc` ne sont généralement utiles que si vous devez communiquer avec un SGBD donné pour lequel il n'existe pas de fournisseur de données .NET personnalisé**. Ceci s'explique par le fait qu'ODBC est un modèle répandu qui permet d'accéder à plusieurs sources de données.

Le fournisseur de données OLE DB, composé des types définis dans l'espace de noms `System.Data.OleDb`, vous permet d'accéder aux données situées dans n'importe quelle source de données prenant en charge le protocole OLE DB classique basé sur COM. *==Du fait de sa dépendance à COM, ce fournisseur ne fonctionne que sous Windows et est obsolète dans l'environnement multiplateforme .NET.==*

Le fournisseur de données PostgreSQL, composé des types définis dans l'espace de noms tiers `Npgsql`, vous permet d'accéder aux données situées dans n'importe quel système de gestion de base de données PostgreSQL. **==Contrairement aux modèles génériques ou dépendants de Windows, `Npgsql` est un pilote managé natif, entièrement écrit en C# et optimisé spécifiquement pour le protocole réseau de PostgreSQL. Du fait de son architecture moderne, il offre des performances exceptionnelles, prend nativement en charge l'asynchronisme de bout en bout requis par l'environnement multiplateforme .NET, et s'impose comme le choix incontournable pour les développements modernes sous macOS, Linux et Windows.==**

# Types de l'espace de noms `System.Data`

**Parmi tous les espaces de noms ADO.NET, `System.Data` est le plus commun**. ***==Cet espace de noms contient des types partagés par tous les fournisseurs de données ADO.NET, quel que soit le système de stockage de données sous-jacent==***. Outre un certain nombre d'exceptions spécifiques aux bases de données (par exemple, `NoNullAllowedException`, `RowNotInTableException` et `MissingPrimaryKeyException`), **`System.Data` contient des types représentant diverses primitives de base de données** (par exemple, tables, lignes, colonnes et contraintes), **ainsi que les interfaces communes implémentées par les objets fournisseurs de données**. Le [[#Tableau 20-3 Membres clé de l'espace de noms `System.Data`|Tableau 20-3]] répertorie certains des types principaux à connaître.


###### Tableau 20-3: Membres clé de l'espace de noms `System.Data`

| Type              | Description                                                                                                                         |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| `Constraint`      | Représente une contrainte pour un objet `DataColumn` donné                                                                          |
| `DataColumn`      | Représente une seule colonne au sein d'un objet `DataTable`                                                                         |
| `DataRelation`    | Représente une relation parent-enfant entre deux objets `DataTable`                                                                 |
| `DataRow`         | Représente une seule ligne au sein d'un objet `DataTable`                                                                           |
| `DataSet`         | Représente un cache de données en mémoire constitué d'un nombre quelconque d'objets `DataTable` interreliés                         |
| `DataTable`       | Représente un bloc tabulaire de données en mémoire                                                                                  |
| `DataTableReader` | Permet de traiter un `DataTable` comme un curseur de type « firehose » (accès aux données en lecture seule et en avant uniquement). |
| `DataView`        | Représente une vue personnalisée d'un `DataTable` permettant le tri, le filtrage, la recherche, la modification et la navigation.   |
| `IDataAdapter`    | Définit le comportement de base d'un objet adaptateur de données                                                                    |
| `IDataParameter`  | Définit le comportement de base d'un objet paramètre                                                                                |
| `IDataReader`     | Définit le comportement de base d'un objet lecteur de données                                                                       |
| `IDbCommand`      | Définit le comportement de base d'un objet de commande                                                                              |
| `IDbDataAdapter`  | Étend `IDataAdapter` pour fournir des fonctionnalités supplémentaires d'un objet adaptateur de données                              |
| `IDbTransaction`  | Définit le comportement de base d'un objet de transaction                                                                           |

**Votre prochaine tâche consiste à examiner les interfaces principales de `System.Data` de manière générale ; cela vous permettra de comprendre les fonctionnalités communes offertes par tout fournisseur de données**. Vous découvrirez également des détails spécifiques tout au long de ce chapitre ; toutefois, pour l’instant, **==il est préférable de se concentrer sur le comportement global de chaque type d’interface.==**

## Rôle de l'interface `IDbConnection`

Le type `IDbConnection` est implémenté par l'*objet de connexion* d'un fournisseur de données. **Cette interface définit un ensemble de membres permettant de configurer une connexion à une base de données spécifique**. Elle permet également d'**obtenir l'objet transactionnel du fournisseur de données**. Voici la définition formelle de `IDbConnection` :

```cs
public interface IDbConnection : IDisposable
{
	string ConnectionString { get; set; }
	int ConnectionTimeout { get; }
	string Database { get; }
	ConnectionState State { get; }
	
	IDbTransaction BeginTransaction();
	IDbTransaction BeginTransaction(IsolationLevel il);
	void ChangeDatabase(string databaseName);
	void Close();
	IDbCommand CreateCommand();
	void Open();
	void Dispose();
}
```

## Rôle de l’interface `IDbTransaction`

La méthode `BeginTransaction()` surchargée, définie par `IDbConnection`, permet d’accéder à l’*objet transaction* du fournisseur. **Vous pouvez utiliser les membres définis par `IDbTransaction` pour interagir par programmation avec une session transactionnelle et la base de données sous-jacente.**

```cs
public interface IDbTransaction : IDisposable
{
	IDbConnection Connection { get; }
	IsolationLevel IsolationLevel { get; }
	
	void Commit();
	void Rollback();
	void Dispose();
}
```

## Le rôle de l'interface `IDbCommand`

**L'interface `IDbCommand`**, implémentée par l'*objet de commande* du fournisseur de données, **à l'instar des autres modèles d'objets d'accès aux données, permettent la manipulation programmatique des instructions SQL, des procédures stockées et des requêtes paramétrées**. Ils offrent également un accès au type de lecteur de données du fournisseur via la méthode surchargée `ExecuteReader()`.

```cs
public interface IDbCommand : IDisposable
{
	string CommandText { get; set; }
	int CommandTimeout { get; set; }
	CommandType CommandType { get; set; }
	IDbConnection Connection { get; set; }
	IDbTransaction Transaction { get; set; }
	IDataParameterCollection Parameters { get; }
	UpdateRowSource UpdatedRowSource { get; set; }
	
	void Prepare();
	void Cancel();
	IDbDataParameter CreateParameter();
	int ExecuteNonQuery();
	IDataReader ExecuteReader();
	IDataReader ExecuteReader(CommandBehavior behavior);
	object ExecuteScalar();
	void Dispose();
}
```

## Rôle des interfaces `IDbDataParameter` et `IDataParameter`

**Notez que la propriété `Parameters` de `IDbCommand` renvoie une collection fortement typée qui implémente `IDataParameterCollection`**. ==Cette interface donne accès à un ensemble de classes compatibles avec `IDbDataParameter`== (par exemple, des objets paramètres).

```cs
public interface IDbDataParameter : IDataParameter
{
	// Plus les membres situé dans l'interface IDataParameter
	byte Precision { get; set; }
	byte Scale { get; set; }
	int Size { get; set; }
}
```

**`IDbDataParameter` étend l'interface `IDataParameter` pour obtenir les comportements supplémentaires suivants :**

```cs
public interface IDataParameter
{
	DbType DbType { get; set; }
	ParameterDirection Direction { get; set; }
	bool IsNullable { get; }
	string ParameterName { get; set; }
	string SourceColumn { get; set; }
	DataRowVersion SourceVersion { get; set; }
	object Value { get; set; }
}
```

**Comme vous le verrez, les interfaces `IDbDataParameter` et `IDataParameter` vous permettent de représenter les paramètres d'une commande SQL** (y compris les procédures stockées) **via des objets de paramètres ADO.NET spécifiques, plutôt que via des chaînes littérales codées en dur.**

## Rôle des interfaces `IDbDataAdapter` et `IDataAdapter`

==Vous utilisez des *adaptateurs de données* pour transférer des jeux de données vers et depuis un magasin de données donné==. L'interface `IDbDataAdapter` définit l'ensemble de propriétés suivant, que vous pouvez utiliser pour gérer les instructions SQL des opérations de sélection, d'insertion, de mise à jour et de suppression associées :

```cs
public interface IDbDataAdapter : IDataAdapter
{
	// Plus les membre de IDataAdapter
	IDbCommand SelectCommand { get; set; }
	IDbCommand InsertCommand { get; set; }
	IDbCommand UpdateCommand { get; set; }
	IDbCommand DeleteCommand { get; set; }
}
```

**Outre ces quatre propriétés, un adaptateur de données ADO.NET hérite du comportement défini dans l'interface de base `IDataAdapter`**. **==Cette interface définit la fonction principale d'un adaptateur de données : la possibilité de transférer des `DataSets` entre l'appelant et la base de données sous-jacente à l'aide des méthodes `Fill()` et `Update()`==**. **L'interface `IDataAdapter` permet également d'associer les noms de colonnes de la base de données à des noms d'affichage plus conviviaux grâce à la propriété `TableMappings`.**

```cs
public interface IDataAdapter
{
	MissingMappingAction MissingMappingAction { get; set; }
	MissingSchemaAction MissingSchemaAction { get; set; }
	ITableMappingCollection TableMappings { get; }
	
	DataTable[] FillSchema(DataSet dataSet, SchemaType schemaType);
	int Fill(DataSet dataSet);
	IDataParameter[] GetFillParameters();
	int Update(DataSet dataSet);
}
```

## Rôle des interfaces `IDataReader` et `IDataRecord`

**L'interface clé suivante à connaître est `IDataReader`, qui représente les comportements courants pris en charge par un objet lecteur de données donné. Lorsque vous obtenez un type compatible `IDataReader` à partir d'un fournisseur de données ADO.NET, vous pouvez parcourir l'ensemble de résultats en lecture seule et en mode séquentiel.**

```cs
public interface IDataReader : IDisposable, IDataRecord
{
	// Plus les membre de IDataRecord
	int Depth { get; }
	bool IsClosed { get; }
	int RecordsAffected { get; }
	
	void Close();
	DataTable GetSchemaTable();
	bool NextResult();
	bool Read();
	Dispose();
}
```

**Enfin, `IDataReader` étend `IDataRecord`, qui définit de nombreux membres permettant d'extraire une valeur fortement typée du flux, plutôt que de convertir l'objet générique `System.Object` récupéré de la méthode indexer surchargée du lecteur de données**. Voici la définition de l'interface `IDataRecord` :

```cs
public interface IDataRecord
{
	int FieldCount { get; }
	object this[ int i ] { get; }
	object this[ string name ] { get; }
	bool GetBoolean(int i);
	byte GetByte(int i);
	long GetBytes(int i, long fieldOffset, byte[] buffer,
		int bufferoffset, int length);
	char GetChar(int i);
	long GetChars(int i, long fieldoffset, char[] buffer,
		int bufferoffset, int length);
	IDataReader GetData(int i);
	string GetDataTypeName(int i);
	DateTime GetDateTime(int i);
	Decimal GetDecimal(int i);
	double GetDouble(int i);
	Type GetFieldType(int i);
	float GetFloat(int i);
	Guid GetGuid(int i);
	short GetInt16(int i);
	int GetInt32(int i);
	long GetInt64(int i);
	string GetName(int i);
	int GetOrdinal(string name);
	string GetString(int i);
	object GetValue(int i);
	int GetValues(object[] values);
	bool IsDBNull(int i);
}
```

>[!note]
>Vous pouvez utiliser la méthode `IDataReader.IsDBNull()` pour déterminer par programmation si un champ spécifié est `null` avant de tenter d'obtenir une valeur du lecteur de données (afin d'éviter de déclencher une exception d'exécution). Rappelons également que C# prend en charge les types de données pouvant être nullable (voir le [[Chapitre 4#Comprendre les types nullable en C|Chapitre.4]]), qui sont idéaux pour interagir avec les colonnes de données susceptibles d'être `null` dans la table de base de données.

## Synthèse Architecturale d'ADO.NET

L'architecture d'ADO.NET repose sur un ensemble d'interfaces standardisées définies dans `System.Data`. Ces interfaces imposent des contrats stricts à tous les fournisseurs de bases de données, garantissant une logique de programmation identique, que vous utilisiez `Npgsql` (PostgreSQL) ou `Microsoft.Data.SqlClient` (SQL Server).

### 1. Le Core : Mode Connecté

#### `IDbConnection` — Le Canal Réseau

Représente la connexion physique ou logique au serveur. C'est le "tuyau" à travers lequel transitent toutes les requêtes.

* **Membres clés :** 
	* `.Open()` 
	* `.Close()` 
	* `.OpenAsync()` 
	* `.CloseAsync()` 
	* `.State` 
	* `.ConnectionString`

* **Implémentations :** `NpgsqlConnection` (Postgres) / `SqlConnection` (SQL Server).

>[!note] *Note moderne :* 
>Implémente également `IAsyncDisposable` pour la libération des ressources via `await using`.

#### `IDbCommand` — L'Émetteur d'Ordres

Encapsule la requête SQL textuelle ou la procédure stockée à exécuter. Nécessite obligatoirement une `IDbConnection` active.

* **Membres clés :** 
	* `.CommandText` (Le script SQL).
	* `.ExecuteNonQuery()` / `.ExecuteNonQueryAsync()` (Pour `INSERT`, `UPDATE`, `DELETE`).
	* `.ExecuteReader()` / `.ExecuteReaderAsync()` (Retourne un flux de lignes).
	* `.ExecuteScalar()` / `.ExecuteScalarAsync()` (Retourne une valeur unique, ex: `COUNT`).

* **Implémentations :** `NpgsqlCommand` / `SqlCommand`.

####  `IDataParameter` & `IDbDataParameter` — La Gestion des Paramètres

Représente un argument injecté dynamiquement dans la requête. C'est le bouclier indispensable contre les **injections SQL** et pour le transtypage automatique.

* **`IDataParameter` (Base) :** Gère l'identité fondamentale.
	* *Membres :* 
		* `.ParameterName` (ex: `@id`)
		* `.Value`

* **`IDbDataParameter` (Relationnel) :** Hérite de la base et ajoute les contraintes strictes des SGBDR.
	* *Membres :* 
		* `.Size` (Taille max)
		* `.Precision`
		* `.DbType`

* **Implémentations :** `NpgsqlParameter` / `SqlParameter` (Chaque classe implémente les deux interfaces).

#### `IDataReader` & `IDataRecord` — Le Flux de Lecture

Permet un parcours ultra-rapide, séquentiel (en avant uniquement) et en lecture seule du résultat d'une commande. N'alloue que la ligne en cours en mémoire RAM.

* **`IDataReader` (Le Contrôleur de Flux) :** Gère le déplacement dans le curseur de données.
	* *Membres :* 
		* `.Read()` / `.ReadAsync()` (Avance à la ligne suivante, renvoie `false` si vide)
		* `.Close()`

* **`IDataRecord` (Le Conteneur de Ligne) :** Abstraits l'accès aux cellules de la ligne active. `IDataReader` en hérite directement.
	* *Membres :* 
		* `.GetString()`
		* `.GetInt32()`
		* `.IsDBNull()`

* **Règle d'or :** Changement de ligne ➔ membres de `IDataReader`. Lecture d'une case ➔ membres de `IDataRecord`.

* **Implémentations :** `NpgsqlDataReader` / `SqlDataReader`.

#### `IDbTransaction` — Le Gardien de l'Atomicité

Regroupe plusieurs commandes au sein d'une même unité de travail. Assure le respect des propriétés ACID.
* **Membres clés :** 
	* `.Commit()`
	* `.Rollback()`

* **Implémentations :** `NpgsqlTransaction` / `SqlTransaction`.

### 2. L'Infrastructure : Mode Déconnecté

> [!warning] Statut Technologique
> Cette plomberie (`DataTable`, `DataSet`) sert à manipuler des données hors ligne en RAM. Elle est aujourd'hui considérée comme verbeuse et lourde. Elle est remplacée en production par **EF Core**, mais reste essentielle pour maintenir du code existant.

#### `IDataAdapter` & `IDbDataAdapter` — Le Pont d'Échange

Sert de traducteur et de synchronisateur automatique entre la base de données physique et les structures en mémoire vive.

* **`IDataAdapter` (Base) :** Définit les méthodes d'échange globales.
	* *Membres :* 
		* `.Fill()` (Alimente la RAM depuis la DB)
		* `.Update()` (Soumet les changements de la RAM à la DB)

* **`IDbDataAdapter` (Relationnel) :** Ajoute les commandes SQL spécifiques requises pour chaque action CRUD.
	* *Membres :* 
		* `.SelectCommand`, 
		* `.InsertCommand`, 
		- `.UpdateCommand`, 
		- `.DeleteCommand` 
	(Toutes de type `IDbCommand`).

* **Implémentations :** `NpgsqlDataAdapter` / `SqlDataAdapter`.

### 3. Schéma Global du Flux de Données

```text
================================== MODE CONNECTÉ ==================================

[IDbConnection] (Ouvre le canal réseau)
       │
       ▼
[IDbCommand] (Contient le SQL, utilise des IDbDataParameter pour la sécurité)
       │
       ├─► ExecuteNonQuery() ──► Modifie la base (INSERT/UPDATE/DELETE)
       │
       └─► ExecuteReader() ────► Génère un [IDataReader] (Gère les lignes)
                                        │
                                        ▼ [IDataRecord] (Lit la ligne actuelle)
                                        Ex: .GetString(0), .GetInt32(1)


================================= MODE DÉCONNECTÉ =================================

[IDbDataAdapter] (Le pont bidirectionnel)
       │
       ├─► Configuration : .SelectCommand, .InsertCommand, etc. (Utilisent des IDbCommand)
       │
       ├─► Méthode .Fill() ───► Extrait via SelectCommand ──► Insère dans [DataTable / DataSet] (RAM)
       │
       └─► Méthode .Update() ─► Détecte les changements RAM ─► Applique via Insert/UpdateCommand (DB)
```

# Abstraction des fournisseurs de données à l'aide d'interfaces

À ce stade, vous devriez avoir une meilleure idée des fonctionnalités communes à tous les fournisseurs de données .NET. **Rappelez-vous que même si les noms exacts des types d'implémentation diffèrent d'un fournisseur de données à l'autre, vous pouvez programmer avec ces types de manière similaire : c'est là toute la beauté du polymorphisme** basé sur les interfaces. Par exemple, si vous définissez une méthode qui prend un paramètre `IDbConnection`, vous pouvez lui passer n'importe quel objet de connexion ADO.NET, comme ceci :

```cs
public static void OpenConnection (IDbconnection)
{
	// Ouvrir la connexion entrante pour l'appelant.
	connection.Open();
}
```

> [!warning] Règle de Design Moderne (.NET 10 / 11)
> **Contrairement à la note faite par l'auteur dans le livre, il faut préférer systématiquement l'utilisation des classes de base abstraites (`DbConnection`, `DbCommand`) aux interfaces (`IDbConnection`, `IDbCommand`)**. C'est le seul moyen de garantir l'accès aux fonctionnalités modernes comme l'asynchronisme (`Async`) et les optimisations de performance récentes de .NET.

**Il en va de même pour les valeurs de retour des membres**. Créez une nouvelle application console .NET nommée *MyConnectionFactory*. Ajoutez les packages NuGet suivants au projet (notez que le package `OleDb` est valide uniquement sous Windows) :
	
- `Microsoft.Data.SqlClient`  (Sql Server)
- `Npgsql` (Postgres)
- `System.Data.Odbc`
- ~~`System.Data.Common`~~ (**Déjà inclus dans les .NET moderne**)
- ~~`System.Data.OleDb`~~ (**NE PAS INSTALLER**)

Ensuite, ajoutez un fichier nommé *DataProviderEnum.cs* et mettez à jour le code comme suit :

```cs
namespace MyConnectionFactory;

// OleDB est UNIQUEMENT pour Windows et n'est pas supporté dans .NET 5+
enum DataProviderEnum
{
    SqlServer,
    Postgres,
    Odbc,
    None,
}
```

**L'exemple de code suivant permet d'obtenir un objet de connexion spécifique en fonction de la valeur d'une énumération personnalisée**. **==À des fins de diagnostic, il suffit d'afficher l'objet de connexion sous-jacent à l'aide des services de réflexion.==**

```cs
using System.Data;
using System.Data.Common;
using System.Data.Odbc;
using Microsoft.Data.SqlClient;
using MyConnectionFactory;
using Npgsql;

Console.Title = "Very Simple Connection Factory";
Console.WriteLine("***** Very Simple Connection Factory *****\n");

Setup(DataProviderEnum.Postgres);
Setup(DataProviderEnum.SqlServer);
Setup(DataProviderEnum.Odbc);
Setup(DataProviderEnum.None);

Console.ReadLine();

static void Setup(DataProviderEnum provider)
{
    /*
     * Le code suivera les bonnes pratique moderne,
     * c'est à dire utiliser les classes de base
     * au lieu des interfaces.
     */

    // Récupère une connection spécifique
    DbConnection? myConection = GetConnection(provider);
    Console.WriteLine(
        $"Your connection is a {myConection?.GetType().Name ?? "unrecognized type"}"
    );
    // Ouvre, utilise et ferme la connexion...
}

static DbConnection? GetConnection(DataProviderEnum provider) =>
    provider switch
    {
        DataProviderEnum.SqlServer => new SqlConnection(),
        DataProviderEnum.Postgres => new NpgsqlConnection(),
        DataProviderEnum.Odbc => new OdbcConnection(),
        _ => null,
    };

```

**L'avantage de travailler avec les interfaces générales de `System.Data` (ou, d'ailleurs, les classes de base abstraites de `System.Data.Common`) est que vous avez beaucoup plus de chances de construire une base de code flexible, capable d'évoluer au fil du temps.** Par exemple, ==vous développez peut-être aujourd'hui une application ciblant Microsoft SQL Server ; cependant, il est possible que votre entreprise passe à une autre base de données==. *==Si vous créez une solution qui intègre en dur les types spécifiques à Microsoft SQL Server de `Microsoft.Data.SqlClient`, vous devrez modifier, recompiler et redéployer le code pour le nouveau fournisseur de base de données.==*

À ce stade, vous avez écrit du code ADO.NET (assez simple) qui vous permet de créer différents types d'objets de connexion spécifiques au fournisseur. Cependant, **l'obtention d'un objet de connexion n'est qu'un aspect de l'utilisation d'ADO.NET. Pour créer une bibliothèque de fabrique de fournisseurs de données performante, vous devrez également prendre en compte les objets de commande, les lecteurs de données, les objets de transaction et d'autres types orientés données**. ***==La création d'une telle bibliothèque de code ne serait pas nécessairement difficile, mais elle nécessiterait une quantité considérable de code et de temps.==***

**Depuis la sortie de .NET 2.0, les équipes de Redmond ont intégré cette fonctionnalité directement dans les bibliothèques de classes de base .NET. Cette fonctionnalité a été considérablement améliorée pour .NET Core et .NET 5 et versions ultérieures.**

Vous examinerez cette API formelle dans un instant ; cependant, ***==vous devez d’abord créer une base de données personnalisée qui sera utilisée tout au long de ce chapitre (et dans de nombreux chapitres à venir).==***

# Configuration de Docker et PostgreSQL

## Le crash-course Docker (Lancer Postgres en 1 minute)

>[!important] Pour des questions d'économie de RAM, je vais opter pour du 100% CLI et m'écarter le *Docker Desktop* **ET utiliser un autre ordinateur sous Linux (Arch).**

- Un conteneur Docker est un **vrai processus serveur isolé** qui tourne sur le noyau de l'hôte.
- On lui parle à distance via ses **ports réseau** (`-p 5432:5432`) pour les applications.
- On entre dedans via `docker exec` au lieu de SSH pour la maintenance. 


Puisque PostgreSQL est déjà installé nativement sur mes machine, l'utilisation de Docker va permettre de créer un conteneur "bac à sable" isolé, sans risquer de corrompre les base de données de mon mac. 

1. Sur ma machine sous Linux, il faut installer docker :

	```bash
	# pour Arch
	sudo pacman -S docker
	sudo systemctl enable --now docker
	```

2. Une fois installé, on peut lancer le conteneur comme ceci :

	```bash
	docker run --name AutoLot -e POSTGRES_PASSWORD=monMotDePasse -p 5432:5432 -d postgres
	```
	
	- `--name` : Donne un nom convivial au conteneur.
	- `-e POSTGRES_PASSWORD=...` : Définit le mot de passe du super-utilisateur `postgres`.
	- `-p 5432:5432` : Redirige le port 5432 du conteneur vers le port 5433 de la machine hôte (le PC Arch Linux), le rendant accessible sur votre réseau local pour votre Mac.
	- `-d` : (*Detached*) Lance le conteneur en arrière-plan pour ne pas bloquer le terminal.
	- `postgres` : Indique à Docker d'utiliser l'image officielle de PostgreSQL.

3. Pour vérifier que le container se soit lancé :

	```bash
	docker ps
	# pour voir tout les conteneurs de la machine, même ceux pas lancé:
	docker ps -a
	```
	
4. Pour stopper le conteneur :

	```bash
	docker stop 12345
	```

	Où `12345` représente les 5 premiers caractère de l'identifiant du container.

5. Pour supprimer un conteneur : 

	```bash
	docker rm -f "Nom conteneur"
	```

Une fois créer, le conteneur peut être lancé en utilisant le nom donné : 

```bash
docker start AutoLot
```

## Se connecter à la base de donnée Postgres

Puisque le conteneur PostgreSQL est actif sur ma deuxième machine, je dois utiliser `pgcli` sur le Mac pour valider la connexion réseau. Il suffit de récupérer l'adresse IP de la machine Arch. (Avec le mot de passe demandé)

```bash
psql -h XXX -p 5432 -U postgres -d postgres
```

>[!tip] Si le port du conteneur utilise le port par défaut, alors il n'est pas nécessaire d'entrer l'argument `-p 5432` dans la chaîne de connexion

**Si tout fonctionne parfaitement, nous voilà connecté dans la base de donnée `postgres` via l'utilisateur `postgres` sur la deuxième machine.**

# Création de la base de donnée AutoLot
Cette section entière est consacrée à la création de la base de données `AutoLot` à l'aide de PostgresSQL. Cette étape est nécessaire car les scripts du livre sont dans une syntaxe que Postgres va échouer de comprendre.

Plutôt que d'exécuter chaque commande SQL les unes à la suite des autres dans `psql` ou `pgcli`, j'ai décidé de créer un fichier *main.sql* qui permettra d'exécuter la création des tables ainsi que leurs peuplements de manière "automatique" et plus rapide. Voici le code du fichier.

```sql
-- Fichier main.sql 


-- Nettoyage complet de la base à chaque exécution du script maître (très utilse si on a fait des bétises)
DROP PROCEDURE IF EXISTS GetPetName;
DROP TABLE IF EXISTS CreditRisks CASCADE;
DROP TABLE IF EXISTS Orders CASCADE;
DROP TABLE IF EXISTS Inventory CASCADE;
DROP TABLE IF EXISTS Customers CASCADE;
DROP TABLE IF EXISTS Makes CASCADE;

-- Étape 1 : Création des tables 
\i CreateScripts/CreateCreditRisksTable.sql
\i CreateScripts/CreateCustomersTable.sql
\i CreateScripts/CreateGetPetnameSproc.sql
\i CreateScripts/CreateInventoryTable.sql
\i CreateScripts/CreateMakesTable.sql
\i CreateScripts/CreateOrdersTable.sql

-- Étape 2 : Création des relations entres les tables (foreign keys)
\i CreateScripts/CreateForeignKeys.sql


-- Étape 3 : Insertion des données (ordre crutial)

-- 1. Les parents absolus (Ne dépendent de personne)
\i DataScripts/AddMakeRecords.sql
\i DataScripts/AddCustomerRecords.sql

-- 2. L'enfant intermédiaire (A besoin de "makes")
\i DataScripts/AddInventoryRecords.sql

-- 3. Les enfants terminaux (Ont besoin de "customers" et "inventory")
\i DataScripts/AddCreditRisks.sql
\i DataScripts/AddOrderRecords.sql
```

>[!attention] Il faut déjà avoir crée la la base de donnée avant d'exécuter ce script !

## Créer la base de donnée

```sql
-- CreateDatabase.sql

-- Création de la base de données AutoLot en syntaxe PostgreSQL
CREATE DATABASE autolot;
```

## Création des tables 
La base de données `AutoLot` contient cinq tables : `Inventory`, `Makes`, `Customers`, `Orders` et `CreditRisks`.

### Création de la table `Inventory`
Une fois la base de données créée, il est temps de créer les tables. La première est la table `Inventory`:

```sql
-- CreateInventoryTable.sql
CREATE TABLE Inventory (
    Id SERIAL NOT NULL,
    MakeId INT NOT NULL,
    Color VARCHAR(50) NOT NULL,
    PetName VARCHAR(50) NOT NULL,
    CONSTRAINT PK_Inventory PRIMARY KEY (Id)
);
```

>[!warning] Différence de comportement entre Sql Server et Postgres (avec Gemini)
>
Dans SQL Server, un type `TIMESTAMP` (ou `ROWVERSION`) est un compteur binaire incrémenté automatiquement à chaque modification de la ligne, garantissant une valeur unique à chaque micro-changement. Dans PostgreSQL, le type `TIMESTAMP` est une simple date/heure. Si deux mises à jour ont lieu à la même milliseconde, elles auront exactement la même valeur, ce qui rend le contrôle de concurrence inefficace
>
>De plus, le mot-clé `DEFAULT CURRENT_TIMESTAMP` n'applique une valeur **qu'au moment de l'insertion (`INSERT`)**. Lors d'une mise à jour (`UPDATE`), PostgreSQL ne modifiera pas automatiquement cette colonne. La date restera celle de la création de la ligne, sauf si vous écrivez manuellement un déclencheur (Trigger) en SQL pour forcer sa mise à jour.
>
>Pour Postgres, Cette colonne ne posera aucun problème pour plus tard, mais ne servira pas pour vérifier la concurrence entre deux entrées.
>
**Pour travailler efficacement avec EF Core et PostgreSQL**, j'ai choisi la solution suivante : la colonne système `xmin`
>
>L'énorme avantage de cette approche est que **PostgreSQL gère tout en arrière-plan de manière totalement transparente**. Vous n'avez pas besoin d'écrire des scripts SQL complexes ou des déclencheurs pour forcer la mise à jour d'une date à chaque modification de ligne.
>
>---
>>[!important]- Pour les classes C#
>>Dans votre projet C#, créez votre classe `Inventory`. Pour que Entity Framework Core comprenne qu'il doit utiliser la colonne `xmin` pour détecter les conflits de modification, ajoutez une propriété de type `uint` (un entier non signé) :
>>
>>```cs
>>using System.ComponentModel.DataAnnotations;
>>
>>namespace SimpleClassExample;
>>
>>public class Inventory
>>{
>>    public int Id { get; set; }
>>    public int MakeId { get; set; }
>>    public string Color { get; set; } = string.Empty;
>>    public string PetName { get; set; } = string.Empty;
>>
>>    // Cette propriété mappe la colonne système masquée de PostgreSQL
>>    [Timestamp]
>>    public uint Version { get; set; }
>>}
>>```
>>
>>Dans le fichier de votre contexte de base de données (votre classe qui hérite de `DbContext`), vous devez indiquer explicitement à EF Core que cette propriété `Version` correspond à la colonne système `xmin` de PostgreSQL.
>>
>>Modifiez ou ajoutez la méthode `OnModelCreating` de cette façon :
>>
>>```cs
>>using Microsoft.EntityFrameworkCore;
>>
>>namespace SimpleClassExample;
>>
>>public class ApplicationDbContext : DbContext
>>{
>>    public DbSet<Inventory> Inventories { get; set; }
>>
>>    protected override void OnModelCreating(ModelBuilder modelBuilder)
>>    {
>>        // Indique à EF Core d'utiliser la colonne système 'xmin' pour la concurrence
>>        modelBuilder.Entity<Inventory>()
>>            .Property(i => i.Version)
>>            .HasColumnName("xmin")
>>            .HasColumnType("xid")
>>            .ValueGeneratedOnAddOrUpdate()
>>            .IsConcurrencyToken();
>>    }
>>}
>>```
>
> ---
> *Lorsque l'auteur abordera les captures d'exceptions de concurrence (les fameuses `DbUpdateConcurrencyException`), votre code se comportera **exactement** de la même manière que le sien :*

###  Création de la table `Makes` 
La table `Inventory` stocke une clé étrangère dans la table `Makes` (pas encore créée). Créez une nouvelle requête / script et saisissez le SQL suivant pour créer la table `Makes` :

```sql
-- CreateMakesTable.sql
CREATE TABLE Makes (
    Id SERIAL NOT NULL,
    Name VARCHAR(50) NOT NULL,
    TimeStamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT PK_Makes PRIMARY KEY (Id)
);
```

### Création de la table `Customers`

La table `Customers` (comme son nom l'indique) contiendra une liste de clients. Créez une nouvelle requête / script et entrez les commandes SQL suivantes :

```sql
-- CreateCustomersTable.sql
CREATE TABLE Customers (
    Id SERIAL NOT NULL,
    FirstName VARCHAR(50) NOT NULL,
    LastName VARCHAR(50) NOT NULL,
    TimeStamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT PK_Customers PRIMARY KEY (Id)
);
```

### Création de la table `Orders`

Vous utiliserez le tableau suivant, `Orders`, pour représenter l'automobile qu'un client donné a commandée. Créer une nouvelle requête / script, entrez le code suivant.

```sql
-- CreateOrdersTable.sql
CREATE TABLE Orders (
    Id SERIAL NOT NULL,
    CustomerId INT NOT NULL,
    CarId INT NOT NULL,
    TimeStamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT PK_Orders PRIMARY KEY (Id)
);
```

### Création de la table `CreditRisks`

Vous utiliserez votre tableau final, `CreditRisks`, pour représenter les clients considérés comme présentant un risque de crédit. Créez une nouvelle requête / script et entrez le code suivant :

```sql
-- CreateCreditRisks.sql
CREATE TABLE CreditRisks (
    Id SERIAL NOT NULL,
    FirstName VARCHAR(50) NOT NULL,
    LastName VARCHAR(50) NOT NULL,
    CustomerId INT NOT NULL,
    TimeStamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT PK_CreditRisks PRIMARY KEY (Id)
);
```

## Création des relations entre les tables

Cette section ajoutera les relations de clé étrangère entre les tables interdépendantes. **Toutes les relations seront créés dans un seul fichier** *CreateForeignKeys.sql*

### Création de la relation `Inventory` vers `Makes`

```sql
-- BLOCK 1 : Index et Clé Étrangère pour la table Inventory (liée à Makes)
CREATE INDEX IX_Inventory_MakeId ON Inventory (MakeId ASC);

ALTER TABLE Inventory 
ADD CONSTRAINT FK_Make_Inventory FOREIGN KEY (MakeId) REFERENCES Makes (Id);
```

### Création de la relation `Orders` vers `Inventory`

```sql
-- BLOCK 2 : Index et Clé Étrangère pour la table Orders (liée à Inventory)
CREATE INDEX IX_Orders_CarId ON Orders (CarId ASC);

ALTER TABLE Orders 
ADD CONSTRAINT FK_Orders_Inventory FOREIGN KEY (CarId) REFERENCES Inventory (Id);
```

### Création de la relation `Orders` vers `Customers`

```sql
-- BLOCK 3 : Index Unique et Clé Étrangère pour la table Orders (liée à Customers)
CREATE UNIQUE INDEX IX_Orders_CustomerId_CarId ON Orders (CustomerId ASC, CarId ASC);

ALTER TABLE Orders 
ADD CONSTRAINT FK_Orders_Customers FOREIGN KEY (CustomerId) REFERENCES Customers (Id) ON DELETE CASCADE;
```

### Création de la relation `CreditRisks` vers `Customers`

```sql
-- BLOCK 4 : Index et Clé Étrangère pour la table CreditRisks (liée à Customers)
CREATE INDEX IX_CreditRisks_CustomerId ON CreditRisks (CustomerId ASC);

ALTER TABLE CreditRisks 
ADD CONSTRAINT FK_CreditRisks_Customers FOREIGN KEY (CustomerId) REFERENCES Customers (Id) ON DELETE CASCADE;
```

>[!note]
>Si vous vous demandez pourquoi il y a des colonnes pour `FirstName` et `LastName` *et* une relation avec le table client, c'est simplement à des fins de démonstration. Je pourrais trouver une raison créative à cela, mais en fin de compte, ils sont utilisés dans des exemples ultérieurs du livre.

## Création de la procédure stockée `GetPetName()`

Plus loin dans ce chapitre, vous apprendrez à utiliser ADO.NET pour appeler des procédures stockées. Comme vous savez peut-être déjà, **les procédures stockées sont des routines de code stockées dans une base de données qui font quelque chose. Comme les méthodes C#, les procédures stockées peuvent renvoyer des données ou simplement opérer sur des données sans rien renvoyer**. ***==Vous ajouterez une seule procédure stockée qui renverra le nom familier d’une automobile, en fonction du `carId` fourni==***. Pour ce faire, créez un nouvelle requête / script et entrez la commande SQL suivante :

```sql
-- Très pratique en phase de test. Si vous lancez le script plusieurs fois, il écrasera l'ancienne version au lieu de planter en disant "la procédure existe déjà"
CREATE OR REPLACE PROCEDURE GetPetName(
    carID INT,
    OUT petName VARCHAR(50)
)
-- Le mot-clé OUT se place avant le nom du paramètre en PostgreSQL, alors que SQL Server le place après (output).
LANGUAGE plpgsql
AS $$
-- Le bloc BEGIN ... END; : Obligatoire en PostgreSQL pour délimiter le corps exécutable de la procédure.
BEGIN
    SELECT Inventory.PetName 
    INTO petName 
    FROM Inventory 
    WHERE Inventory.Id = carID;
END;
$$;
-- Les symboles $$ servent de délimiteurs de texte de niveau supérieur pour éviter que PostgreSQL ne se mélange les pinceaux avec les points-virgules ; qui se trouvent à l'intérieur du bloc de code
```

## Ajout d'entrées de test
Les bases de données sont plutôt ennuyeuses sans données, et c'est une bonne idée d'avoir des scripts capables de charger rapidement les tests.

### Entrées pour la table `Makes`

Créez une nouvelle requête / script et exécutez les instructions SQL suivantes pour ajouter des enregistrements dans la table `Makes` :

```sql
-- Insertion des données des constructeurs (Makes)
INSERT INTO Makes (Id, Name) VALUES (1, 'VW');
INSERT INTO Makes (Id, Name) VALUES (2, 'Ford');
INSERT INTO Makes (Id, Name) VALUES (3, 'Saab');
INSERT INTO Makes (Id, Name) VALUES (4, 'Yugo');
INSERT INTO Makes (Id, Name) VALUES (5, 'BMW');
INSERT INTO Makes (Id, Name) VALUES (6, 'Pinto');

-- Recalage obligatoire de la séquence d'auto-incrémentation (SERIAL) pour la table makes
SELECT setval(pg_get_serial_sequence('makes', 'id'), COALESCE(max(id), 1)) FROM makes;
```

### Entrées pour la table `Inventory`

Pour ajouter des enregistrements à votre première table, créez une nouvelle requête / script et exécutez les instructions SQL suivantes pour ajouter enregistrements dans la table `Inventory` :

```sql
-- Insertion des données de l'inventaire des véhicules
INSERT INTO Inventory (Id, MakeId, Color, PetName) VALUES (1, 1, 'Black', 'Zippy');
INSERT INTO Inventory (Id, MakeId, Color, PetName) VALUES (2, 2, 'Rust', 'Rusty');
INSERT INTO Inventory (Id, MakeId, Color, PetName) VALUES (3, 3, 'Black', 'Mel');
INSERT INTO Inventory (Id, MakeId, Color, PetName) VALUES (4, 4, 'Yellow', 'Clunker');
INSERT INTO Inventory (Id, MakeId, Color, PetName) VALUES (5, 5, 'Black', 'Bimmer');
INSERT INTO Inventory (Id, MakeId, Color, PetName) VALUES (6, 5, 'Green', 'Hank');
INSERT INTO Inventory (Id, MakeId, Color, PetName) VALUES (7, 5, 'Pink', 'Pinky');
INSERT INTO Inventory (Id, MakeId, Color, PetName) VALUES (8, 6, 'Black', 'Pete');
INSERT INTO Inventory (Id, MakeId, Color, PetName) VALUES (9, 4, 'Brown', 'Brownie');

-- Recalage obligatoire de la séquence d'auto-incrémentation (SERIAL) pour la table inventory
SELECT setval(pg_get_serial_sequence('inventory', 'id'), COALESCE(max(id), 1)) FROM inventory;
```

### Entrées pour la table `Customers`

Pour ajouter des entrées à la table `Customers`, créez une nouvelle requête / script et exécutez les instructions SQL suivantes :

```sql
-- Insertions des données clients
INSERT INTO Customers (Id, FirstName, LastName) VALUES (1, 'Dave', 'Brenner');
INSERT INTO Customers (Id, FirstName, LastName) VALUES (2, 'Matt', 'Walton');
INSERT INTO Customers (Id, FirstName, LastName) VALUES (3, 'Steve', 'Hagen');
INSERT INTO Customers (Id, FirstName, LastName) VALUES (4, 'Pat', 'Walton');
INSERT INTO Customers (Id, FirstName, LastName) VALUES (5, 'Bad', 'Customer');

-- Recalage obligatoire de la séquence d'auto-incrémentation (SERIAL) à la fin des inserts
SELECT setval(pg_get_serial_sequence('customers', 'id'), COALESCE(max(id), 1)) FROM customers;
```

### Entrées pour la table `Orders`

Ajoutez maintenant des données à votre table `Orders`. Créez une nouvelle requête / script et entrez le SQL suivant :

```sql
-- Insertion des données des commandes (Orders)
INSERT INTO Orders (Id, CustomerId, CarId) VALUES (1, 1, 5);
INSERT INTO Orders (Id, CustomerId, CarId) VALUES (2, 2, 1);
INSERT INTO Orders (Id, CustomerId, CarId) VALUES (3, 3, 4);
INSERT INTO Orders (Id, CustomerId, CarId) VALUES (4, 4, 7);

-- Recalage obligatoire de la séquence d'auto-incrémentation (SERIAL) pour la table orders
SELECT setval(pg_get_serial_sequence('orders', 'id'), COALESCE(max(id), 1)) FROM orders;
```

### Entrées pour la table `CreditRisks`

Ajouter des données à la table `CreditRisks`. Créez une nouvelle requête / scripts, entrez le SQL suivant

```sql
INSERT INTO CreditRisks (Id, FirstName, LastName, CustomerId) 
VALUES (1, 'Bad', 'Customer', 5);
SELECT setval(pg_get_serial_sequence('creditrisks', 'id'), COALESCE(max(id), 1)) FROM creditrisks;
```

## Exécution de tous les scripts SQL

Cette section explique la méthode pour exécuter les scripts SQL crée précédemment. **Si vous avez utilisé des requêtes directement, vous pouvez passer à la section [[#Le modèle de fabrique de fournisseurs de données ADO.NET]]**

On peut lancer la commande suivante (depuis le mac où le pc Linux, cela n'à pas d'importance) pour créer la nouvelle base de donnée `autoloc` 

>[!important]  on ne peut pas créer une base de données tout en étant déjà connecté à celle-ci.
>
>
>Pour que tout fonctionne parfaitement, on doit sortir la création de la base de donnée  du fichier `main.sql`

```bash
cat CreateDatabase.sql | pgcli -h IP_PC_ARCH -U postgres -d postgres
```

Enfin, on peut exécuter le fichier *main.sql* pour créer les tables et les peupler : 

```bash
psql -h IP_PC_ARCH -U postgres -d autolot -f main.sql
```

**Voilà, la base de données `AutoLot` est terminée !** ***==Bien sûr, elle est loin d'être une base de données d'application réelle, mais elle répondra à vos besoins pour ce chapitre et sera complétée dans les chapitres consacrés à Entity Framework Core==***. Maintenant que vous disposez d'une base de données pour vos tests, vous pouvez explorer en détail le modèle de fabrique de fournisseurs de données ADO.NET.

# Le modèle de fabrique de fournisseurs de données ADO.NET

**Le modèle de fabrique de fournisseurs de données .NET permet de créer une base de code unique utilisant des types d'accès aux données généralisés**. Pour comprendre l'implémentation de la fabrique de fournisseurs de données, **rappelez-vous du [[#Tableau 20-1 Les objets clé d'un fournisseur de donnée ADO.NET|Tableau 20-1]] que les classes d'un fournisseur de données dérivent toutes des mêmes classes de base définies dans l'espace de noms `System.Data.Common`.**

- `DbCommand` : Classe de base abstraite pour toutes les classes de commandes
- `DbConnection` : Classe de base abstraite pour toutes les classes de connexion
- `DbDataAdapter` : Classe de base abstraite pour toutes les classes d’adaptateurs de données
- `DbDataReader` : Classe de base abstraite pour toutes les classes de lecteurs de données
- `DbParameter` : Classe de base abstraite pour toutes les classes de paramètres
- `DbTransaction` : Classe de base abstraite pour toutes les classes de transactions

**Chaque fournisseur de données compatible .NET contient un type de classe dérivé de `System.Data.Common.DbProviderFactory`.** ***==Cette classe de base définit plusieurs méthodes permettant de récupérer des objets de données spécifiques au fournisseur. Voici les membres de `DbProviderFactory` :==***

```cs
public abstract class DbProviderFactory
{
	..public virtual bool CanCreateDataAdapter { get;};
	..public virtual bool CanCreateCommandBuilder { get;};
	public virtual DbCommand CreateCommand();
	public virtual DbCommandBuilder CreateCommandBuilder();
	public virtual DbConnection CreateConnection();
	public virtual DbConnectionStringBuilder
		CreateConnectionStringBuilder();
	public virtual DbDataAdapter CreateDataAdapter();
	public virtual DbParameter CreateParameter();
	public virtual DbDataSourceEnumerator
		CreateDataSourceEnumerator();
}
```

==Pour obtenir le type dérivé de `DbProviderFactory` pour votre fournisseur de données, chaque fournisseur fournit une propriété statique utilisée pour renvoyer le type correct==. Pour renvoyer la version SQL Server de `DbProviderFactory`, utilisez le code suivant :

```cs
// Obtenir la fabrique pour le fournisseur de données SQL.
DbProviderFactory sqlFactory =
	Microsoft.Data.SqlClient.SqlClientFactory.Instance;
```

**Pour rendre le programme plus polyvalent, vous pouvez créer une fabrique `DbProviderFactory` qui renvoie une version spécifique de `DbProviderFactory` en fonction d'un paramètre du fichier** *appsettings.json* de l'application. Vous apprendrez bientôt comment procéder ; **==pour l'instant, vous pouvez obtenir les objets de données spécifiques au fournisseur (par exemple, les connexions, les commandes et les lecteurs de données) une fois que vous aurez obtenu la fabrique correspondant à votre fournisseur de données.==**

## Exemple complet de fabrique de fournisseur de données

Pour un exemple complet, créez un nouveau projet d'application console C# (nommé *DataProviderFactory*) qui affiche l'inventaire automobile de la base de données AutoLot. Pour cet exemple initial, vous intégrerez directement la logique d'accès aux données dans l'application console (par souci de simplicité). Au fil de ce chapitre,
vous découvrirez des méthodes plus efficaces.

Ajoutez les packages `Microsoft.Extensions.Configuration.Json` , `System.Data.Odbc`, `Npgsql` et `Microsoft.Data.SqlClient` au projet. Ensuite, ajoutez un nouveau fichier nommé *DataProviderEnum.cs* et mettez à jour le code comme suit :

>[!important] Rappel
> - `System.Data.Common` -> Déjà présent de base dans les versions moderne de .NET
> - `System.Data.OleDb` -> Uniquement pour Windows -> retiré

```cs
namespace DataProviderFactory;

public enum DataProviderEnum
{
    Postgres,
    SqlServer,
    Odbc,
}
```

Ajoutez un nouveau fichier JSON nommé *appsettings.json* au projet et mettez à jour son contenu comme suit :

>[!warning] Gross différence avec le livre
>Comme le livre est un livre d'apprentisage, les information sensible sont écris en durs dans le fichier *appsettings.json*.
>
> Comme ces notes ansi que les projet seront publié sur mes repo GitHub, je dois **Absolument évité**  cela.

```json
{
  "ProviderName": "Postgres",
  "Postgres": {
    "ConnectionString": "Secret"
  },
  "SqlServer": {
    "ConnectoinString": "Secret"
  },
  "Odbc": {
    "ConectionString": "Secret"
  }
}
```

Ensuite, il faut trouvez l'IP du PC Arch (ex: `192.168.1.50`) en tapant ceci sur le PC

```bash
ip -a
# ou
fastfetch
```

**Pour des questions de sécurité**, je vais isoler les chaîne de connexion aux différents fournisseurs de données dans un fichier *.env*. Ce fichier sera lus par les différents application crée dans ce chapitre et dans les chapitres suivants pour extraire la bonne chaîne de connexion selon la base de donnée à laquelle on veut se connecter. **En harmonisant les noms dans les deux fichier, le code sera plus simple ensuite.

>[!warning] Il faut veillé à ce qu'un fichier *.gitignore* existe dans un des dossier plus élevé et que le fichier *.env* soit listé dedans pour éviter de le deployé sur *GitHub*. (Pour les autres système de cloud, à éffectuer au cas par cas)

Le framework .NET utilise un système de configuration modulaire. Pour lire les variables d'environnement de votre Mac ou de votre fichier de configuration local, vous devez ajouter ce package NuGet officiel

```bash
dotnet add package Microsoft.Extensions.Configuration.EnvironmentVariables
```

>[!attention] Dans le fichier *appsettings.json*, l'arborescence est séparée par des `:`. dans le fichier *.env*, il faut **ABSOLUMENT** séparer en utilisant `__` sinon .NET ne pourra pas lier la chaîne de connexion. 

Configurez MSBuild pour qu'il copie le fichier de paramètres JSON dans le répertoire de sortie à chaque compilation. Mettez à jour le fichier projet en ajoutant les lignes suivantes :

```xml
<ItemGroup>
  <None Update="appsettings.json">
    <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
  </None>
</ItemGroup>
```

>[!note] La commande `CopyToOutputDirectory` est sensible aux espaces. Assurez-vous qu'elle figure sur une seule ligne, sans espaces autour du mot *PreserveNewest*

**Maintenant que vous disposez d'un fichier** *appsettings.json* **correct, vous pouvez lire les valeurs `ProviderName` et `ConnectionString` à l'aide de la configuration .NET**. Commencez par supprimer tout le code du fichier et ajoutez les instructions `using` suivantes en haut du fichier *Program.cs* :

```cs
using System.Data.Common;
using System.Data.Odbc;
using DataProviderFactory;
using Microsoft.Data.SqlClient;
using Microsoft.Extensions.Configuration;
using Npgsql;
```

Puisque le fichier *.env* va rester sagement à la racine du projet, la détection de celui ci par l'application sera plus ardue.

Pour résoudre ce problème de manière universelle (que l'on lance l'app depuis le   terminal ou depuis un IDE), on cherche d'abord dans le dossier d'exécution, puis dans le dossier parent si nécessaire :

```cs
Console.Title = "Fun with Data Provider Factories";
Console.WriteLine("***** Fun with Data Provider Factories *****\n");

// Récupèration du fournisseur et de la chaîne de connexion.
var (provider, connectionString) = GetProviderFromConfiguration();

// Initialisation de la fabrique
DbProviderFactory factory = GetDbProviderFactory(provider);

// Récupération de l'objet de connexion
using (DbConnection connection = factory.CreateConnection())
{
    Console.WriteLine(
        $"Your connection object is a: {connection.GetType().Name}"
    );
    connection.ConnectionString = connectionString;
    connection.Open();

    // Création de l'objet de commande
    DbCommand command = factory.CreateCommand();
    Console.WriteLine($"Your command object is a: {command.GetType().Name}");
    command.Connection = connection;
    command.CommandText =
        "SELECT i.Id, m.Name FROM Inventory i INNER JOIN Makes m ON m.Id = i.MakeId;";

    // Affiche les donnée avec DbDatareader
    // Pas besoin de rajouter une nouvelle portée pour lire le fichier
    using DbDataReader dataReader = command.ExecuteReader();
    Console.WriteLine(
        $"Your data reader object is a: {dataReader.GetType().Name}"
    );
    Console.WriteLine("\n**** Current Inventory ****");
    while (dataReader.Read())
    {
        Console.WriteLine(
            $"-> Car #{dataReader["Id"]} is a {dataReader["Name"]}."
        );
    }
}
Console.ReadLine();

```

Ensuite, ajoutez le code suivant à la fin du fichier *Program.cs*. **Ces méthodes lisent la configuration et le fichier** *.env*, **définissent l'énumération `DataProviderEnum` sur la valeur correcte, obtiennent la chaîne de connexion et renvoient une instance de `DbProviderFactory` :**

```cs
static (
    DataProviderEnum provider,
    string connectionString
) GetProviderFromConfiguration()
{
    // 1. RECHERCHE INTELLIGENTE ET CHARGEMENT DU .ENV EN PREMIER
    string currentDir = Directory.GetCurrentDirectory();
    string envFilePath = Path.Combine(currentDir, ".env");

    while (!File.Exists(envFilePath) && Directory.GetParent(currentDir) != null)
    {
        currentDir = Directory.GetParent(currentDir)!.FullName;
        envFilePath = Path.Combine(currentDir, ".env");
    }

    if (File.Exists(envFilePath))
    {
        foreach (var line in File.ReadAllLines(envFilePath))
        {
            var parts = line.Split('=', 2);
            if (parts.Length == 2)
            {
                Environment.SetEnvironmentVariable(
                    parts[0].Trim(),
                    parts[1].Trim().Trim('"')
                );
            }
        }
    }
    else
    {
        throw new FileNotFoundException("the .env file couldn't be found");
    }

    // 2. CONSTRUCTION DU CONFIGURATIONBUILDER
    // .AddEnvironmentVariables() va maintenant capter TOUTES nos variables chargées juste avant
    var configuration = new ConfigurationBuilder()
        .SetBasePath(Directory.GetCurrentDirectory())
        .AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
        .AddEnvironmentVariables()
        .Build();

    // 3. LECTURE DU PROVIDER ET CONVERSION EN ENUM
    string providerName = configuration["ProviderName"];

    if (!Enum.TryParse(providerName, true, out DataProviderEnum provider))
        throw new Exception("Invalid data provider value supplie;");

    // 4. RÉCUPÉRATION DE LA CHAÎNE DE CONNEXION FUSIONNÉE
    // On utilise GetSection() pour éviter d'avoir des problèmes de casse
    string connectionString =
        configuration
            .GetSection(providerName)
            .GetSection("ConnectionString")
            .Value
        ?? string.Empty;
    if (string.IsNullOrEmpty(connectionString))
        throw new InvalidOperationException(
            $"the connection string for {providerName} is not found"
        );

    return (provider, connectionString);
}

// Sélection de la fabrique
static DbProviderFactory GetDbProviderFactory(DataProviderEnum provider) =>
    provider switch
    {
        DataProviderEnum.Postgres => NpgsqlFactory.Instance,
        DataProviderEnum.SqlServer => SqlClientFactory.Instance,
        DataProviderEnum.Odbc => OdbcFactory.Instance,
        _ => null,
    };
```

**Notez que, pour des raisons de diagnostic, vous utilisez les services de réflexion pour afficher le nom de la connexion, de la commande et du lecteur de données sous-jacents**. ***==Si vous exécutez cette application, vous trouverez les données actuelles suivantes dans la table `Inventory` de la base de données `AutoLot`, affichées dans la console :==***

```
***** Fun with Data Provider Factories *****

Your connection object is a: NpgsqlConnection
Your command object is a: NpgsqlCommand
Your data reader object is a: NpgsqlDataReader

**** Current Inventory ****
-> Car #1 is a VW.
-> Car #2 is a Ford.
-> Car #3 is a Saab.
-> Car #4 is a Yugo.
-> Car #5 is a BMW.
-> Car #6 is a BMW.
-> Car #7 is a BMW.
-> Car #8 is a Pinto.
-> Car #9 is a Yugo.
```

***==Modifiez maintenant le fichier de configuration pour spécifier un autre fournisseur. Le code détectera la chaîne de connexion correspondante et produira le même résultat qu'auparavant, à l'exception des informations spécifiques au type.==**

**Bien sûr, compte tenu de votre expérience avec ADO.NET, vous n'êtes peut-être pas certain du rôle exact des objets de connexion, de commande et de lecteur de données.** **==Ne vous préoccupez pas des détails pour le moment (il reste encore de nombreuses pages dans ce chapitre !).==** **À ce stade, il suffit de savoir que vous pouvez utiliser le modèle de fabrique de fournisseurs de données ADO.NET pour créer une base de code unique capable d'utiliser différents fournisseurs de données de manière déclarative.**

## Un inconvénient potentiel du modèle de fabrique de fournisseurs de données

Bien que ce modèle soit puissant, *==vous devez vous assurer que le code utilise uniquement des types et des méthodes communs à tous les fournisseurs via les membres des classes de base abstraites==*. Par conséquent, **lors de la création de votre code, vous êtes limité aux membres exposés par `DbConnection`, `DbCommand` et les autres types de l'espace de noms `System.Data.Common`.**

De ce fait, ***==cette approche généralisée peut vous empêcher d'accéder directement à certaines fonctionnalités avancées d'un SGBD particulier.==*** **Si vous devez pouvoir appeler des membres spécifiques du fournisseur sous-jacent (par exemple, `SqlConnection`), vous pouvez le faire en utilisant une conversion explicite, comme dans cet exemple :**

```cs
if (connection is SqlConnection sqlConnection)
{ 
	// Afficher la version de SQL Server utilisée.
	WriteLine(sqlConnection.ServerVersion);
}
```

Toutefois, *==cette approche complexifie la maintenance du code (et réduit sa flexibilité) car il faut ajouter plusieurs vérifications d'exécution==*. **Néanmoins, si vous souhaitez créer des bibliothèques d'accès aux données ADO.NET avec une flexibilité maximale, le modèle de fabrique de fournisseurs de données offre une solution idéale.**

>[!note]
>Entity Framework Core et sa prise en charge de l'injection de dépendances simplifient considérablement la création de bibliothèques d'accès aux données qui doivent accéder à des sources de données disparates.

Ce premier exemple étant derrière vous, vous pouvez maintenant vous plonger dans les détails de l'utilisation d'ADO.NET.

# Approfondissement des `Connections`, `Commands` et `DataReader`







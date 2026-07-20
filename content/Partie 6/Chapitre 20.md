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
> **Comme je suis sur macOS, je vais toujours utilisé** *Docker* **mais changé la base de donnée par** *PostgreSQL* **car c'est déjà installé sur ma machine.**
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





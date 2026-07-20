
# Installation des technologie nécessaire pour le chapitre 20

## Le crash-course Docker (Lancer Postgres en 1 minute)

>[!important] Pour des questions d'économie de RAM, je vais opter pour du 100% CLI et m'écarter le *Docker Desktop*

Puisque PostgreSQL est déjà installé nativement sur ma machine, l'utilisation de Docker va  permet de créer un conteneur "bac à sable" isolé, sans risquer de corrompre les base de données de mon mac. 

1. Pour avoir Docker en pure ligne de commande sans l'interface graphique Docker Desktop, la solution standard et open-source sur Mac s'appelle **Colima**. Vous pouvez l'installer en deux lignes avec *Homebrew* :

	```bash
	# 1. Installez le moteur CLI de Docker et Colima
	brew install docker colima
	
	# 2. Démarrez la mini-VM Linux en arrière-plan
	colima start --cpu 1 --memory 1.5
	```

	Dès que Colima est lancé, votre commande `docker run` fonctionnera directement dans votre terminal classique, de manière 100 % textuelle.

	- Pour arrêter la VM une fois que l'on à plus besoin d'utiliser docker : 
	
		```bash
		colima stop
		```

>[!note]- Explication de `colima`
>
>**Colima (Docker)** crée ce qu'on appelle une **Headless VM** (une machine virtuelle "sans tête"). Elle n'a aucune interface graphique, aucun écran, aucun bureau. Elle existe uniquement en arrière-plan comme un moteur invisible. Son seul but est de prêter son noyau Linux à Docker. [[1](https://superuser.com/questions/1384950/how-to-run-hyper-v-virtual-machine-headless), [2](https://www.reddit.com/r/Mechwarrior5/comments/1av6bun/yaml_newbie_question/)]
>
>Vous ne vous connectez jamais _à Colima_, vous utilisez simplement les commandes `docker` dans votre terminal Mac, et Colima gère la plomberie de façon transparente.

2. Ouvrez votre terminal et tapez cette unique commande pour lancer votre base de données d'exercice :

	```bash
	docker run --name pg-ado-tests -e POSTGRES_PASSWORD=monMotDePasse -p 5432:5432 -d postgres
	```
	
	- `--name pg-ado-tests` : Donne un nom convivial à votre conteneur.
	- `-e POSTGRES_PASSWORD=...` : Définit le mot de passe du super-utilisateur `postgres`.
	- `-p 5432:5432` : Redirige le port réseau du conteneur vers votre Mac (vous permet d'utiliser `pgcli` localement).
	- `-d postgres` : Télécharge et lance l'image officielle de PostgreSQL en arrière-plan. [[1](https://contic.co.uk/blogs/how-to-setup-your-local-database-management-with-docker)]

Vous pouvez vérifier que tout tourne en vous connectant via votre outil préféré :

```bash
pgcli -h localhost -p 5432 -U postgres
```

##  Adapter le code du livre (La traduction ADO.NET)

Dans le projet C#, on n'utilisera pas le package de Microsoft pour SQL Server, mais celui pour PostgreSQL. 

1. Ajoutez le pilote officiel dans votre projet .NET :
    
    ```bash
    dotnet add package Npgsql
    ```

2. Pour faire les exercices du chapitre 20, il suffit d'appliquer cette **grille de traduction** mentale lorsqu'on lis le livre : 

| Le livre (SQL Server)             | Votre code (PostgreSQL) |
| --------------------------------- | ----------------------- |
| `using Microsoft.Data.SqlClient;` | `using Npgsql;`         |
| `SqlConnection`                   | `NpgsqlConnection`      |
| `SqlCommand`                      | `NpgsqlCommand`         |
| `SqlDataReader`                   | `NpgsqlDataReader`      |
| `SqlParameter`                    | `NpgsqlParameter`       |

Exemple de code : 

```cs
using Npgsql;

// La chaîne de connexion pointe vers votre Docker local
string connectionString = "Host=localhost;Username=postgres;Password=monMotDePasse;Database=postgres";

using var connection = new NpgsqlConnection(connectionString);
await connection.OpenAsync();

Console.WriteLine("Connexion réussie au PostgreSQL de Docker depuis mon Mac !");

// Exemple de commande
using var command = new NpgsqlCommand("SELECT version();", connection);
using var reader = await command.ExecuteReaderAsync();

if (await reader.ReadAsync())
{
    Console.WriteLine($"Version de l'OS : {reader.GetString(0)}");
}
```

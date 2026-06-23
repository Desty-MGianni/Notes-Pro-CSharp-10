---
title: "Création d'un Launcher Personnalisé"
publish: true
---

# Script lançant les projets venant d'une solution

Ce "script" permet d'extraire, grace aux nouveaux format des solution (*.slnx* qui est du *xml*) le chemin d'accès de chaque projet présent dans la solution.

Pour exécuter le script:

```
./Launcher.cs <chiffre_chapitre> [args]
```

## Code

```cs
#!/usr/bin/env dotnet
using System.Diagnostics;
using System.Xml.Linq;

string SLNX_FILE = $"Chapter{args[0]}_AllProjects.slnx";
const string LOG_FILE = "errors.log";

// Initialisation du log
File.WriteAllText(LOG_FILE, string.Empty);

if (!File.Exists(SLNX_FILE))
{
    Console.Error.WriteLine($"Fichier {SLNX_FILE} introuvable.");
    return;
}

// Extraction des chemins depuis le SLNX
var projects = XDocument
    .Load(SLNX_FILE)
    .Descendants("Project")
    .Select(p => p.Attribute("Path")?.Value)
    .Where(p => !string.IsNullOrEmpty(p));

// Récupération des arguments passés au script (@args est fourni par dotnet-script)
var appArgs = string.Join(" ", args[1..]);

foreach (var proj in projects)
{
    if (!File.Exists(proj))
    {
        var error = $"Project not found: {proj}";
        Console.Error.WriteLine(error);
        File.AppendAllText(LOG_FILE, $"{error}{Environment.NewLine}");
        continue;
    }

    Console.WriteLine($"=== Running {proj} ===");

    var startInfo = new ProcessStartInfo
    {
        FileName = "dotnet",
        Arguments = $"run --project \"{proj}\" -- {appArgs}",
        UseShellExecute = false,
    };

    using var process = Process.Start(startInfo);
    process?.WaitForExit();

    if (process?.ExitCode != 0)
    {
        var timestamp = DateTime.Now.ToString("HH:mm:ss");
        File.AppendAllText(LOG_FILE, $"[{timestamp}] FAILED: {proj}{Environment.NewLine}");
    }

    Console.WriteLine($"=== {proj} finished ===\n");
}

Console.WriteLine($"Exécution terminée. Consultez {LOG_FILE} pour le résumé des erreurs.");

```

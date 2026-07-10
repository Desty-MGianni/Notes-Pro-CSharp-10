---
publish: true
---
>[!info] Informations générés par Claude

Sur Linux et macOS, les assemblies .NET sont parfois distribuées avec l'extension `.so` ou sans extension du tout pour les exécutables. Et `.nupkg` (NuGet) est un format spécifique à l'écosystème .NET.

# Créer un `.so` (Shared Object)

Pour créer une bibliothèque `.so` (shared object Linux/macOS), il faut publier ton assembly en **native AOT** :

- Configuration en ligne de commande

```bash
dotnet publish -r osx-arm64 -p:PublishAot=true
```

- Configuration dans le `.csproj`

```xml
<PropertyGroup>
    <OutputType>Library</OutputType>
    <PublishAot>true</PublishAot>
    <AllowUnsafeBlocks>true</AllowUnsafeBlocks>
</PropertyGroup>
```

Et les méthodes que tu veux exposer doivent être annotées :

```csharp
using System.Runtime.InteropServices;

public static class MyLib
{
    [UnmanagedCallersOnly(EntryPoints = ["my_function"])]
    public static int MyFunction(int x) => x * 2;
}
```


# Ce qu'est un `.so`

Un `.so` (Shared Object) est l'équivalent Linux/macOS d'une DLL Win32 : du **code machine natif** directement exécutable par le CPU, sans runtime intermédiaire. C'est ce que C/C++/Rust produisent nativement.

## Pourquoi exposer les méthodes avec `[UnmanagedCallersOnly]`

Quand tu compiles en AOT, ton code .NET devient du code machine natif. Mais le problème c'est que par défaut, **aucune méthode n'est visible de l'extérieur** : le compilateur AOT peut les supprimer ou les renommer par optimisation.

`[UnmanagedCallersOnly]` dit explicitement au compilateur :

> "Cette méthode doit être accessible depuis l'extérieur avec ce nom précis, ne la supprime pas"

C'est l'équivalent de `extern "C"` en C/C++.

## Pourquoi `unsafe`

Le passage de données entre un `.so` et un appelant externe (Python, C, Rust...) ne passe **pas** par le système de types de C#. Tu manipules directement des pointeurs et de la mémoire brute, ce qui nécessite `unsafe`.











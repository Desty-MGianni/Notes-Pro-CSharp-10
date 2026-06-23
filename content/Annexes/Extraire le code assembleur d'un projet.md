---
title: "Extraire le Code Assembleur d'un Projet"
publish: true
---

# Guide de l'Expert : Analyse de la Compilation JIT sous .NET (macOS M4)

Ce document explique comment intercepter et analyser le code machine (ASM) généré par le compilateur **RyuJIT** sur une architecture Apple Silicon (ARM64).

---

## Pourquoi utiliser `export` ?

Sous macOS/Linux, une variable d'environnement placée juste avant une commande (ex: `VAR=1 dotnet...`) n'est visible que par le processus immédiat. Comme `dotnet` lance souvent des sous-processus pour gérer l'exécution réelle de la DLL, la variable est parfois "perdue" en chemin.

- **`export`** rend la variable globale pour toute la session de votre terminal.
- Tous les sous-processus créés par `dotnet` en héritent alors obligatoirement.

---

## Variables de contrôle du JIT

Voici les leviers les plus puissants pour manipuler le comportement du runtime.

`DOTNET_JitDisasm` (Le projecteur)

C'est la variable principale. Elle demande au JIT d'afficher l'assembleur des méthodes compilées.

- `*` : Affiche absolument tout (très volumineux).
- `NomDeLaMethode` : Affiche uniquement la méthode demandée.
- `*PartieDuNom*` : Utilise des jokers pour filtrer.
- _Note :_ Si votre code utilise les "Top-level statements" (C# 9/10+), votre code principal se trouve souvent dans une classe générée nommée `<Program>$` ou une méthode `<Main>$`.

`DOTNET_TieredCompilation` (Le sélecteur de vitesse)

**Par défaut, .NET compile en deux étapes : le "Tier 0" (rapide mais peu optimisé) puis le "Tier 1" (lent mais ultra-optimisé).**

- `0` : Désactive la compilation par paliers. Le JIT produit immédiatement le code le plus optimisé possible. **Indispensable pour l'analyse de performance.**
- `1` : (Par défaut) Active les paliers.

`DOTNET_PrintNmols` (L'explorateur)

Si vous ne savez pas quel nom donner à `JitDisasm`, activez ceci.

- `1` : Affiche une liste simple de chaque méthode au moment où elle est compilée nativement, sans l'assembleur. C'est parfait pour trouver le nom exact (ex: `TestJIT.Program:Add`) à copier-coller.

---

## Procédure de capture (Workflow recommandé)

Pour éviter de polluer votre sortie avec les logs du système de build, suivez toujours ces trois étapes :

### Étape 1 : Nettoyage et Build

```bash
dotnet build
```

### Étape 2 : Configuration de l'environnement


```bash
export DOTNET_JitDisasm="VotreMethode"
export DOTNET_TieredCompilation=0
```


### Étape 3 : Exécution et redirection

On pointe directement vers la DLL pour éviter le "bruit" de l'outil `dotnet run`.

``` bash
dotnet bin/Debug/VersionDotnet/VotreProjet.dll > analyse.asm 2>&1
```

La fin de la commande: `2>&1` sert à rediriger les erreurs possible de l'opération vers le `stdout` (le terminal)

## Comprendre l'assembleur ARM64 (M4)

Contrairement au x64 (Intel), l'ARM64 utilise des registres nommés `W` (32 bits) ou `X` (64 bits).

|Instruction|Signification|
|---|---|
|`stp x29, x30, [sp, #-32]!`|Sauvegarde les registres (Prologue).|
|`mov w0, #10`|Place la valeur 10 dans le registre `w0`.|
|`add w0, w0, w1`|Additionne `w0` et `w1`, stocke le résultat dans `w0`.|
|`ldr` / `str`|Charge (Load) ou Stocke (Store) une valeur depuis/vers la mémoire.|
|`ret`|Quitte la fonction et retourne la valeur présente dans `x0/w0`.|

## Nettoyage

Une fois votre analyse terminée, n'oubliez pas de supprimer la variable pour éviter que vos futurs projets ne ralentissent votre terminal en affichant de l'assembleur partout :

```bash
unset DOTNET_JitDisasm
```


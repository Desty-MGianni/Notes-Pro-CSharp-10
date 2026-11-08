---
title: La capture de variables et les fonctions locales statiques
publish: true
---
>[!note] Cette note à été généré par Gemini

>[!tldr] **Le terme closure décrit une réalité technique très précise : *la conversion par le compilateur d'un état éphémère sur la pile en un objet persistant sur le tas***

# La capture de variables (*closures*) et les fonctions locales `static`

Lorsqu'une ==expression lambda== ou une ==fonction locale== accède à une variable déclarée dans la méthode englobante, le compilateur C# doit faire en sorte que cette variable reste accessible, même si la fonction est exécutée plus tard ou transmise ailleurs.

Pour réussir cela, le compilateur C# réécrit votre code sous le capo :

- Il génère une classe cachée (souvent nommée `&lt;&gt;c__DisplayClass`).
- Il transforme la variable locale en un **champ** de cette classe.
- Il instancie cette classe sur le **tas** (_heap allocation_).

##  Le problème des allocations masquées

Dans un langage de script dynamique (comme JavaScript), la mémoire est gérée de manière très souple sur le tas (_heap_), ce qui fait que l'impact d'une closure passe souvent inaperçu.

En C#, la mécanique sous-jacente est très stricte :

- **Le problème de la pile ( *stack* )** : Les variables locales d'une méthode sont allouées sur la pile d'exécution et sont détruites dès que la méthode se termine.
- **Le conflit :** Si vous écrivez une expression lambda qui utilise une variable locale de la méthode englobante, et que cette lambda est exécutée plus tard (par exemple transmise à une requête LINQ ou à un thread secondaire), la variable sur la pile risque de ne plus exister au moment de l'appel[4][6].

```cs
public void TraiterDonnees(List<int> nombres, int facteur)
{
	// 'facteur' est une variable locale de TraiterDonnees. 
	// La fonction locale 'Multiplier' accède à 'facteur'. 
	int Multiplier(int x) => x * facteur;
	
	var resultats = nombres.Select(Multiplier);
}
```

Sous le capot, le compilateur crée une instance de la classe de fermeture sur le tas pour stocker `facteur`. Si cette méthode est appelée des milliers de fois par seconde, cela crée une pression inutile sur le Garbage Collector.

## La solution : Le modificateur `static` (C# 8+)

En ajoutant le mot-clé `static` à une fonction locale, vous demandez au compilateur de **vérifier strictement** que la fonction n'accède à aucune variable locale englobante ni au pointeur `this`.

```cs
public void TraiterDonnees(List<int> nombres, int facteur)
{
	// ERREUR DE COMPILATION si on tente d'accéder directement à 'facteur' ! 
	// On doit passer 'facteur' explicitement en argument. 
	static int Multiplier(int x, int f) => x * f; 
	
	var resultats = nombres.Select(n => Multiplier(n, facteur));
}
```

## Bénéfices :
- **Garantie anti-allocation :** Le compilateur ne générera jamais de classe de fermeture sur le tas pour cette fonction locale.
- **Absence d'effets de bord :** La fonction locale ne peut pas modifier par inadvertance l'état de la méthode parente.

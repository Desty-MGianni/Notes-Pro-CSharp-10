---
title: Tableau Dentelés vs Tableau Multidimensionnels
publish: true
---

# Comparatif Détaillé : Tableaux Dentelés (`int[][]`) vs Tableaux Rectangulaires (`int[,]`) en C# et .NET

Ce document résume les différences architecturales, la disposition en mémoire, le fonctionnement au niveau du CLR et les caractéristiques de performance qui distinguent les tableaux dentelés (*jagged arrays*) des tableaux rectangulaires (*rectangular / multidimensional arrays*).

---

## 1. Modèle Conceptuel et Structuration Mémoire

### A. Le Tableau Dentelé (`int[][]`)
Un tableau dentelé est littéralement **un tableau de tableaux**.
* **Structure :** C'est un tableau unidimensionnel principal (`SZArray`) dont chaque élément contient une référence (un pointeur) vers un autre tableau unidimensionnel indépendant.
* **Allocations sur le tas (*Heap*) :** Pour une grille de 4 lignes × 4 colonnes (`int[4][]`), le runtime instancie **5 objets distincts** sur le tas managé :
  1. 1 tableau conteneur principal de 4 références.
  2. 4 tableaux sous-jacents de 4 entiers chacun.
* **Disposition mémoire :** Les sous-tableaux sont dispersés sur le tas managé. Les éléments de chaque sous-tableau sont contigus en mémoire, mais les sous-tableaux entre eux ne le sont pas obligatoirement.

```
Tableau principal (Pointeurs)
[0] ───> [ int, int, int, int ] (Objet 1 sur le tas)
[1] ───> [ int, int, int, int ] (Objet 2 sur le tas)
[2] ───> [ int, int, int, int ] (Objet 3 sur le tas)
[3] ───> [ int, int, int, int ] (Objet 4 sur le tas)
```

---

### B. Le Tableau Rectangulaire (`int[,]`)
Un tableau rectangulaire est **un seul et unique bloc de mémoire contigu**.
* **Structure :** Il ne s'agit pas de 4 tableaux accolés, mais d'une seule instance d'objet de type `MDArray` (Rank = 2).
* **Allocations sur le tas (*Heap*) :** Pour `new int[4, 4]`, le CLR n'effectue qu'**une seule allocation** sur le tas managé.
* **Disposition mémoire :** Les 16 entiers sont rangés de façon strictement contiguë, en ordre *row-major* (ligne par ligne) :

```
En-tête de l'objet | Dim 0 = 4 | Dim 1 = 4 | [0,0][0,1][0,2][0,3][1,0][1,1]...[3,3]
<────────────────────── Single Continuous Memory Block ──────────────────────>
```

---

## 2. Pourquoi un tableau rectangulaire n'est-il pas traité comme "4 tableaux classiques" ?

1. **Absence d'en-têtes d'objets individuels pour les lignes :**
   En .NET, tout tableau classique (`int[]`) possède un en-tête d'objet complet (SyncBlockIndex, MethodTable pointer, longueur). Dans `int[4,4]`, les lignes n'ont pas d'en-tête propre. La ligne n° 2 n'existe pas en tant qu'instance d'objet indépendante. Vous ne pouvez donc pas extraire `grid[1]` sous forme d'un `int[]`.

2. **Typage CIL et instructions Bytecode distinctes :**
   * Le type `int[]` est un **`SZArray`** (*Single-dimensional Zero-based array* / Vector). Le CIL dispose d'opcodes d'accès natifs et dédiés (`newarr`, `ldelem`, `stelem`, `ldlen`).
   * Le type `int[,]` est un **`MDArray`** (*Multidimensional Array*). Il n'a pas d'opcodes bytecode d'accès direct ; le CLR génère des appels de méthodes virtuelles/internes (`Get`, `Set`, `Address`).

3. **Formule de calcul d'indexation :**
   Pour accéder à `grid[i, j]`, le moteur d'exécution doit calculer dynamiquement l'offset mémoire :
   $$\text{Offset} = (i \times \text{NombreDeColonnes} + j) \times \text{TailleElement}$$
   Alors que pour un `SZArray`, l'indexation est directe avec suppression du contrôle des bornes dans les boucles (*Bounds Checking Elimination*).

---

## 3. Tableau Comparatif des Performances

| Critère | Tableau Dentelé (`int[][]`) | Tableau Rectangulaire (`int[,]`) |
| :--- | :--- | :--- |
| **Type CLR/CIL** | `SZArray` de `SZArray` | `MDArray` (Rank 2) |
| **Nombre d'allocations** | $N + 1$ allocations sur le tas | **1 seule allocation** sur le tas |
| **Contiguïté mémoire** | Fragmentée (chaque ligne est isolée) | **Parfaite** (optimale pour le cache CPU L1/L2) |
| **Accès aux éléments** | 2 déréférencements de pointeurs | 1 calcul d'offset ($i \times \text{cols} + j$) |
| **Vérification des bornes (*Bounds Check*)** | **Éliminée par le JIT** dans les boucles `for` | Réalisée à **chaque itération** (2 vérifications) |
| **Vitesse de parcours** | **Très rapide** (grâce à l'optimisation JIT) | Plus lente en code managé standard |

---

## 4. Synthèse

* Si vous cherchez la **vitesse d'itération pure** en code managé C#, préférez **`int[][]`** (ou `Span<T>` / `Memory<T>`).
* Si vous cherchez la **contiguïté mémoire** (pour des opérations d'interopérabilité native C/C++ ou pour éviter la fragmentation du tas), préférez **`int[,]`**.

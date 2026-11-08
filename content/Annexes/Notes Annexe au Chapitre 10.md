---
title: Note Annexe au Chapitre 10
publish: true
---
# Chapitre 10: Collections et Génériques — Sous le Capot

>[!note] Cette note à été généré par Gemini

Ce document détaille les mécanismes internes du CLR, de la compilation JIT et des métadonnées CIL pour les collections et les types génériques en C# (.NET 6 / C# 10).

---

## 1. Spécialisation JIT vs Partage de Code (Code Sharing)

Lorsque le compilateur JIT traite des types génériques (`List<T>`) :

* **Types Références (`List<string>`, `List<Personne>`)** :
  * Toutes les instantiations avec un type référence partagent le **même code natif assembleur**.
  * Comme tout pointeur de référence occupe une taille fixe (8 octets sur architecture 64 bits), le même jeu d'instructions processeur peut manipuler les adresses sans générer de duplications de code binaire.

* **Types Valeurs (`List<int>`, `List<double>`)** :
  * Le CLR génère du **code natif spécialisé et dédié** pour chaque type valeur distinct.
  * Un `int` (4 octets) et un `double` (8 octets) nécessitent des décalages d'adresses (*offsets*) et des registres processeur différents. Cette spécialisation garantit des performances identiques à du code C/C++ écrit sur mesure.

---

## 2. Types Ouverts vs Types Fermés et Champs Statiques

* **Arité et Métadonnées CIL** : En CIL, un type générique est identifié par son nom et son arité (ex: ``List`1<T>``).
* **Type Ouvert (*Open Type*)** : Un type contenant au moins un paramètre générique non résolu (ex: `Dictionary<string, T>`). Le CLR interdit l'instanciation directe d'un type ouvert.
* **Type Fermé (*Closed Type*)** : Un type dont tous les arguments génériques sont fournis (ex: `List<int>`).
* **Isolation des champs statiques** : Chaque type fermé possède sa propre zone mémoire statique et son propre constructeur statique (`.cctor`).
  * `CompteurGénérique<int>.Compte` et `CompteurGénérique<string>.Compte` sont deux variables statiques totalement distinctes sur le tas.

---

## 3. Suppression du Boxing dans les Collections Génériques

| Aspect                          | `ArrayList` (Non-générique)                | `List<T>` (Générique)                     |
| :------------------------------ | :----------------------------------------- | :---------------------------------------- |
| **Stockage interne**            | Tableau `object[]`                         | Tableau `T[]` fortement typé              |
| **Ajout d'un `struct` (`int`)** | **Boxing** (allocation sur le tas managé)  | Écriture mémoire directe (pas de boxing)  |
| **Lecture d'un élément**        | Unboxing + cast explicite à l'exécution    | Lecture directe depuis le tableau binaire |
| **Sécurité de typage**          | Erreur au Runtime (`InvalidCastException`) | Erreur de compilation immédiate           |

---

## 4. La Contrainte `where T : new()` sous le Capot

Lorsque vous appliquez la contrainte `where T : new()`, la création `new T()` ne génère pas une instruction CIL directe `newobj`.
Au niveau du binaire CIL, le compilateur génère un appel à `System.Activator.CreateInstance<T>()` (ou une instruction `initobj` si `T` est un type valeur), ce qui implique une légère introspection pour s'assurer que le constructeur sans paramètre existe et est accessible.

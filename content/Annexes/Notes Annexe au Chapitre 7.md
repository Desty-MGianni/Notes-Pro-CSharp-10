---
title: Note Annexe au Chapitre 7
publish: true
---
# Chapitre 7 : Gestion Structurée des Exceptions (SEH) –– Sous le Capot

>[!note] Cette note à été généré par Gemini

Ce document détaille l'architecture interne, la spécification ISO CIL et les mécanismes du Common Language Runtime (CLR) relatifs à la gestion structurée des exceptions (*Structured Exception Handling*) présentés dans le Chapitre 7.

---

## 1. L'Architecture CIL des Exceptions & La Table des Gestionnaires

Contrairement aux mécanismes basés sur des codes d'erreur ou `HRESULT`, le CLR intègre nativement le modèle de gestion structurée des exceptions de Windows (SEH).

Au niveau des métadonnées CIL, chaque méthode dispose d'une table d'exception (*Exception Handler Table*) stockée dans la section d'en-tête de la méthode. Chaque entrée (*Exception Entry*) définit trois plages d'instructions :
1. **Protected Region (Bloc `try`)** : Plage d'instructions surveillée.
2. **Filter Region (optionnelle)** : Plage d'instructions évaluant une condition de capture.
3. **Handler Region (`catch`, `finally` ou `fault`)** : Code exécuté lors d'une correspondance ou du nettoyage.

>[!tip] Le mot clé `fault` n'est pas disponible en C#. [voir ici](https://stackoverflow.com/questions/22065106/fault-keyword-in-try-block)

---

## 2. Le Déroulement à Deux Phases (*Two-Pass Stack Unwinding*)

Lorsque la CLI ou le code utilisateur lève une exception (via l'opcode `throw`), le CLR exécute le traitement en deux phases distinctes :

* **Phase 1 : La Recherche (*Search Pass*)**
  * Le CLR parcourt la table des gestionnaires d'exceptions de la méthode courante, puis remonte la pile d'appels (*call stack*) frame par frame.
  * Il recherche un bloc `catch` dont le type correspond à l'exception levée, ou évalue la clause `when` (bloc `filter`).
  * **Important** : Pendant cette première phase, **aucun bloc `finally` n'est exécuté**. La pile d'appels et l'état des variables locales restent strictement intacts à l'endroit exact de l'erreur.

* **Phase 2 : Le Dépilage (*Unwind Pass*)**
  * Une fois le gestionnaire correspondant localisé, le CLR effectue le dépilage de la pile depuis le point d'origine de l'exception jusqu'au bloc `catch` retenu.
  * Pendant ce trajet de remontée, le CLR exécute tous les blocs `finally` et `fault` intermédiaires rencontrés sur la pile.
  * Une fois le dépilage terminé, le contrôle est transféré au bloc `catch` sélectionné.

---

## 3. Opcodes CIL Majeurs pour SEH

| Opcode CIL      | Mnémotique                | Description & Rôle dans le CLR                                                                                                                             |
| :-------------- | :------------------------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `0x7A`          | `throw`                   | Dépile l'objet exception de la pile d'évaluation et déclenche la phase 1 de recherche de gestionnaire.                                                     |
| `0xFE 0x1A`     | `rethrow`                 | Uniquement valide dans un bloc `catch`. Relance l'exception attrapée sans modifier le point d'origine de la `StackTrace` dans le CLR.                      |
| `0xDD` / `0xDE` | `leave` / `leave.s`       | Quitte un bloc protégé (`try` ou `catch`). Vide la pile d'évaluation et garantit l'exécution séquentielle des blocs `finally` englobants.                  |
| `0xDC`          | `endfinally` / `endfault` | Signale la fin d'un bloc `finally` ou `fault`. Redonne le contrôle au moteur d'exception du CLR pour poursuivre le dépilage.                               |
| `0xFE 0x11`     | `endfilter`               | Termine un bloc de filtre (`when`). Retourne `1` (`exception_execute_handler`) ou `0` (`exception_continue_search`) au CLR pour valider ou non la capture. |

---

## 4. `throw ex` vs `throw` (`rethrow`) et `StackTrace`

* **`throw ex;` (`throw`)** :
  * Réassigne une exception et force le CLR à réinitialiser le point de départ de la trace de pile (*stack trace origin*) à la ligne actuelle.
  * La trace des cadres de pile (*stack frames*) antérieurs au bloc `catch` courant est écrasée dans la propriété `StackTrace`.

* **`throw;` (`rethrow`)** :
  * Génère l'instruction CIL `rethrow`.
  * Le CLR préserve la trace d'appels initiale depuis le point d'origine où l'exception a été instanciée et levée.

---

## 5. Restrictions de Contrôle de Flux en CIL

Pour garantir la sûreté du système de types et la gestion de la mémoire, la spécification ISO CIL impose des règles de branchement très strictes :
1. **Interdiction de `ret` dans un bloc protégé** : L'instruction `ret` ne peut pas apparaître à l'intérieur d'un bloc `try`, `catch`, `filter` ou `finally`. Pour sortir, le compilateur émet un `leave` vers une étiquette hors du bloc SEH, suivie d'un `ret`.
2. **Entrée contrôlée** : Aucun saut (`br`, `beq`, etc.) ne peut pénétrer dans un bloc `catch` ou `finally` depuis l'extérieur. L'entrée ne se fait que via le moteur d'exception du CLR.
3. **Piles vides aux frontières** : À la sortie d'un bloc `try` via `leave`, la pile d'évaluation CIL doit être vide.

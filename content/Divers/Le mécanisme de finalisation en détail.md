---
title: Le mécanisme de finalisation en détail
publish: true
---
>[!note] Cette note à été généré par Gemini

Le [[Chapitre 9#Création de types finalisables et jetables|Chapitre 9]] aborde la méthode `Finalize()` et l'interface `IDisposable`, mais il résume la plomberie interne du CLR. Voici ce qui se passe réellement au niveau du moteur d'exécution (_runtime_).

#### **1. Pourquoi une file d'attente dédiée ?**

Le Garbage Collector (GC) est conçu pour être rapide et bloquer les threads de l'application le moins longtemps possible. Exécuter du code personnalisé écrit par un développeur (dans un destructeur `~MaClasse()`) directement au milieu d'un cycle de nettoyage serait très dangereux : si le destructeur contient une boucle infinie ou un blocage, tout le GC serait totalement gelé. Le CLR sépare donc la **détection des objets morts** de **l'exécution de leurs finaliseurs**.

#### **2. Le parcours d'un objet finalisable en 5 étapes**

```
[ Allocation (new) ] ──&gt; Inscrit sur la Finalization List
                                 │
                     (L'objet devient inaccessible)
                                 ▼
                     Déplacé vers la Freachable Queue
                                 │
                     (Résurrection temporaire + Promotion GC)
                                 ▼
                     Thread de finalisation exécute Finalize()
                                 │
                     (Objet retiré de la Freachable Queue)
                                 ▼
                     Libération réelle lors du GC suivant

```

1. **Inscription à l'allocation :** Lorsqu'un objet possédant un finaliseur personnalisé est instancié avec `new`, le CLR enregistre immédiatement un pointeur vers cet objet dans une structure interne appelée la **Finalization List**.
2. **Détection d'inaccessibilité :** Quand le GC s'exécute et constate que l'objet n'est plus référencé par aucune racine applicative (_unreachable_), il ne peut pas détruire sa mémoire immédiatement.
3. **Mise en file** **freachable** **:** Le pointeur est retiré de la `Finalization List` et placé dans la **freachable** **queue** (prononcée _F-reachable_ pour _Finalization Reachable_).
4. **Résurrection temporaire et promotion :** La file `freachable` est elle-même traitée par le CLR comme une racine (_root_). L'objet redevient donc temporairement accessible (_reachable_) ! Il survit à ce cycle du GC et est promu dans la génération supérieure, ce qui rallonge artificiellement sa durée de vie en mémoire.
5. **Exécution asynchrone et nettoyage :** Un thread système dédié à haute priorité dépile les éléments de la `freachable` queue un par un et exécute leur méthode `Finalize()`. Une fois la méthode exécutée, l'objet est retiré de la file. Il n'a plus aucune racine et sera définitivement supprimé lors du **prochain passage du GC** sur cette génération.

>[!tip] **Pourquoi `IDisposable` est crucial :** 
>Lorsque vous implémentez le pattern `Dispose` et appelez `GC.SuppressFinalize(this)`, vous demandez au CLR de retirer immédiatement l'objet de la `Finalization List`. L'objet est alors détruit dès le **premier** passage du GC, sans passer par la file `freachable` ni subir de promotion de génération inutile.
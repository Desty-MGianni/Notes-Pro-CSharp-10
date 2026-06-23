---
title: "Chapitre 9: Comprendre la Durée de Vie d'un Objet"
publish: true
---

# <big><big><big><b><font color =green>Comprendre la Durée de Vie d'un Objet</font></b></big></big></big>

À ce stade du livre, vous avez acquis de solides connaissances sur la création de types de classes personnalisés en C#. Vous allez maintenant découvrir ==comment le runtime gère les instances de classes allouées== (ou objets) -==via le *ramasse-miettes*== (*Garbage Collector* ou *GC*). **Les programmeurs C# ne libèrent jamais directement un objet managé de la mémoire** (rappelons qu'il n'existe pas de mot-clé `delete` en C#). ==Les objets .NET Core sont plutôt alloués à une zone de mémoire appelée *tas managé*, où ils seront automatiquement détruits par le ramasse-miettes « ultérieurement »==.

Après avoir examiné **les principaux aspects du processus de collecte, vous apprendrez à interagir par programmation avec le ramasse-miettes à l'aide de la classe `System.GC`** (==une opération que vous n'aurez généralement pas à effectuer dans la plupart de vos projets==). Vous étudierez ensuite **comment la méthode virtuelle `System.Object.Finalize()` et l'interface `IDisposable` peuvent être utilisées pour créer des classes qui libèrent les ressources internes *non managées* de manière prévisible et opportune**. 

Vous explorerez également certaines **fonctionnalités du ramasse-miettes introduit dans .NET 4.0, notamment le ramassage des ordures en arrière-plan et l'instanciation paresseuse à l'aide de la classe générique `System.Lazy<>`**. À la fin de ce chapitre, vous aurez acquis une solide compréhension de la manière dont les objets .NET Core sont gérés par le runtime.

# Classes, objets et références

Pour bien comprendre les sujets abordés dans ce chapitre, il est important de clarifier la distinction entre les classes, les objets et les variables de référence. **Rappelons qu'une classe n'est rien d'autre qu'un modèle décrivant l'apparence et le comportement d'une instance de ce type en mémoire**. ==Les classes sont définies dans un fichier de code== (qui, en C#, porte par convention l'extension *.cs*). Prenons l'exemple de la classe `Car` suivante, définie dans un nouveau projet d'application console C# nommé *SimpleGC* :

```cs
namespace SimpleGC;

public class Car
{
    public int CurrentSpeed { get; set; }
    public string PetName { get; set; }

    public Car() { }

    public Car(string name, int speed)
    {
        PetName = name;
        CurrentSpeed = speed;
    }

    public override string ToString() =>
        $"{PetName} is going {CurrentSpeed} km/h";
}
```

**Une fois une classe définie, vous pouvez allouer autant d'objets que nécessaire à l'aide du mot-clé `new` en C#**. ==Il est important de comprendre que `new` renvoie une *référence* à l'objet sur le tas, et non l'objet lui-même==. **Si vous déclarez cette référence comme variable locale dans la portée d'une méthode, elle est stockée sur la pile pour être utilisée ultérieurement dans votre application**. Pour accéder aux membres de l'objet, appliquez l'opérateur point (`.`) de C# à la référence stockée, comme ceci :

```cs
using SimpleGC;

Console.Title = "GC Basics";
Console.WriteLine("***** GC Basics *****");

// Créer un nouvel objet Car sur le tas géré.
// Une référence à l'objet nous est renvoyée.
// ("refToMyCar").
Car refToMyCar = new Car("Zippy", 50);

// L'opérateur point (.) de C# est utilisé pour appeler les membres
// de l'objet en utilisant notre variable de référence.
Console.WriteLine(refToMyCar.ToString());
Console.ReadLine();
```

L'image suivante illustre la relation entre la classe, l'objet et la référence.

![[Figure 9.1.png|Références aux objets dans le tas géré]]

>[!note]
>Rappelons-nous du [[Chapitre 4#Comprendre les structures (`struct`)|Chapitre 4]] que les structures sont des *types valeur* qui sont toujours alloués directement sur la pile et ne sont jamais placés sur le tas managé de .NET Core. L'allocation sur le tas n'a lieu que lors de la création d'instances de *types référence* ou lors du *boxing* de types valeur.

# Principes de base de la durée de vie des objets

Lorsque vous développez vos applications C#, vous pouvez supposer à juste titre que l'environnement d'exécution .NET Core gérera le tas managé sans votre intervention directe. En effet, la règle d'or de la gestion de la mémoire .NET Core est simple.

>[!warning] Règle
>***Allouez une instance de classe sur le tas géré à l'aide du mot-clé `new` et n'y pensez plus.***

**Une fois instancié, le ramasse-miettes détruit un objet lorsqu'il n'est plus nécessaire**. La question suivante est, bien sûr : « ==Comment le ramasse-miettes détermine-t-il quand un objet n'est plus nécessaire ?== » **La réponse courte (c'est-à-dire incomplète) est que le ramasse-miettes supprime un objet du tas uniquement s'il est inaccessible par toute partie de votre code**. Supposons que vous ayez une méthode dans votre fichier *Program.cs* qui alloue un objet `Car` local comme suit :

```cs
static void MakeCar()
{
    // Si myCar est la seule référence à l'objet Car,
    // il *peut* être détruit lorsque cette méthode se termine.
    Car myCar = new Car();
}
```

**Notez que cette référence `Car` (`myCar`) a été créée directement dans la méthode `MakeACar()` et n'a pas été passée en dehors de la portée de définition** (via une valeur de retour ou des paramètres `ref`/`out`). Ainsi, **une fois cet appel de méthode terminé, la référence `myCar` n'est plus accessible et l'objet `Car` associé est désormais un candidat pour le ramasse-miettes**. Sachez toutefois que ==vous ne pouvez pas garantir que cet objet sera récupéré de la mémoire immédiatement après la fin de `MakeACar()`. Tout ce que vous pouvez supposer à ce stade, c'est que lorsque le runtime effectuera le prochain ramassage de miettes, l'objet `myCar` pourra être détruit sans risque==.

Comme vous le découvrirez certainement, ==programmer dans un environnement avec ramasse-miettes simplifie grandement le développement de votre application==. À l'inverse, les programmeurs C++ savent pertinemment que s'ils oublient de supprimer manuellement les objets alloués sur le tas, les fuites de mémoire ne sont jamais loin. **En fait, la recherche de fuites de mémoire est l'un des aspects les plus chronophages (et fastidieux) de la programmation dans des environnements non managés**. En laissant le ramasse-miettes se charger de la destruction des objets, la gestion de la mémoire vous est déchargée et confiée à l'environnement d'exécution.

## Le CIL de  `new`

**Lorsque le compilateur C# rencontre le mot-clé `new`, il insère une instruction CIL `newobj` dans l’implémentation de la méthode**. Si vous compilez l’exemple de code actuel et examinez le code assembleur résultant à l’aide de *ildasm.exe*, vous trouverez les instructions CIL suivantes dans la méthode `MakeACar()` :

```CIL
.method assembly hidebysig static void 
	    '<<Main>$>g__MakeCar|0_0'() cil managed
{
	.custom instance void [System.Runtime]System.Runtime.CompilerServices.CompilerGeneratedAttribute::.ctor() = ( 01 00 00 00 ) 
    // Code size       8 (0x8)
    .maxstack  1
    .locals init (class SimpleGC.Car V_0)
    IL_0000:  nop
    IL_0001:  newobj     instance void SimpleGC.Car::.ctor()
    IL_0006:  stloc.0
	IL_0007:  ret
} // end of method Program::'<<Main>$>g__MakeCar|0_0'
```

>[!tip]- Explication ligne par ligne pour préparer le [[Chapitre 18]]
>
>```CIL
>.locals init (class SimpleGC.Car V_0)
>```
> Réserve un emplacement mémoire dans la pile (*stack*) avec comme nom temporaire `V_0` pour stocker un pointeur vers un objet `Car`
>
>```CIL
>IL_0001: newobj insance void SimpleGC.Car::.ctor()
>```
> Expliqué dans le paragraphe suivant.
>
>```CIl
>IL_0006: stloc.0
>```
>C'est l'instruction d'affectation (*Store Local*). Prend l'adresse mémoire qui flottant en haut de la pile d'exécution (*Evaluation Stack*) généré par `newobj`. On vient l'enfoncé enfoncé à l'index `0` des variable locales (`V_0`)

Avant d'examiner les règles exactes qui déterminent quand un objet est supprimé du tas managé, **examinons plus en détail le rôle de l'instruction CIL `newobj`**. Il faut d'abord comprendre que ==le tas managé est plus qu'un simple bloc de mémoire auquel accède le runtime==. **Le ramasse-miettes de .NET Core gère efficacement le tas, en compactant les blocs de mémoire vides (lorsque nécessaire) à des fins d'optimisation**.

**Pour ce faire, le tas managé conserve un pointeur** (communément appelé *pointeur vers l'objet suivant* ou *pointeur vers le nouvel objet*) **qui indique précisément l'emplacement du prochain objet. L'instruction `newobj` indique au runtime d'effectuer les opérations principales suivantes** :

1. **Calculer la quantité totale de mémoire requise pour l'objet à allouer** (y compris la mémoire requise par les membres de données et les classes de base).
2. **Vérifier que le tas managé dispose de suffisamment d'espace pour accueillir l'objet à allouer**. Si c'est le cas, ==le constructeur spécifié est appelé et l'appelant reçoit finalement une référence au nouvel objet en mémoire, dont l'adresse correspond à la dernière position du pointeur d'objet suivant==.
3. Enfin, avant de renvoyer la référence à l'appelant, **incrémenter le pointeur d'objet suivant pour qu'il pointe vers le prochain emplacement disponible dans le tas managé**.

L'image suivante illustre les bases du processus.

![[Figure 9.2.png| Les détails de l'allocation d'objets dans le tas géré]]

**Pendant que votre application alloue des objets, l'espace du tas géré peut finir par être *saturé***. ***==Lors du traitement de l'instruction `newobj`, si l'environnement d'exécution détermine que le tas géré ne dispose pas de suffisamment de mémoire pour allouer le type demandé, il effectuera un nettoyage de la mémoire afin de libérer de la mémoire.==*** La règle suivante du nettoyage de la mémoire est donc également assez simple.

>[!warning] Règle
***Si le tas géré ne dispose pas de suffisamment de mémoire pour allouer un objet demandé, une collecte aura lieu.***

*Comment* exactement ce ramassage survient, cependant, dépend du type de nettoyage utilisé par votre application. Vous examinerez les différences un peu plus loin dans ce chapitre.

## Définition des références d'objets à `null`

Les programmeurs C/C++ initialisent souvent les variables pointeur à `null` pour s'assurer qu'elles ne référencent plus de mémoire non managée. De ce fait, **vous pourriez vous demander quel est le résultat de l'affectation de références d'objets à `null` en C#. Par exemple, supposons que la sous-routine `MakeACar()` ait été mise à jour comme suit :**

```cs
static void MakeCar()
{
    Car myCar = new Car();
    myCar = null;
}
```

**Lorsque vous assignez la valeur `null` à des références d'objet, le compilateur génère du code CIL qui garantit que la référence** (`myCar`, dans cet exemple) **ne pointe plus vers aucun objet**. Si vous utilisiez à nouveau *ildasm.exe* pour visualiser le code CIL de la fonction `MakeACar()` modifiée, **vous trouveriez l'opcode `ldnull`** (==qui empile une valeur `null` sur la pile d'exécution virtuelle==) **suivi de l'opcode `stloc.0`** (==qui affecte la valeur `null` à la référence à la variable==).

```CIL
  .method assembly hidebysig static void 
          '<<Main>$>g__MakeCar|0_0'() cil managed
  {
    .custom instance void [System.Runtime]System.Runtime.CompilerServices.CompilerGeneratedAttribute::.ctor() = ( 01 00 00 00 ) 
    // Code size       10 (0xa)
    .maxstack  1
    .locals init (class SimpleGC.Car V_0)
    IL_0000:  nop
    IL_0001:  newobj     instance void SimpleGC.Car::.ctor()
    IL_0006:  stloc.0
    IL_0007:  ldnull
    IL_0008:  stloc.0
    IL_0009:  ret
  } // end of method Program::'<<Main>$>g__MakeCar|0_0'
```

Ce qu'il faut bien comprendre, c'est que **l'affectation d'une référence à `null` n'oblige en aucun cas le ramasse-miettes à se déclencher immédiatement et à supprimer l'objet du tas**. ==La seule chose que vous avez faite est de rompre explicitement le lien entre la référence et l'objet qu'elle pointait auparavant==. **De ce fait, affecter la valeur `null` à une référence en C# a beaucoup moins de conséquences que dans d'autres langages dérivés du C; toutefois, cela ne causera certainement aucun dommage**.

# Déterminer si un objet est actif

Revenons à la façon dont le ramasse-miettes détermine si un objet n'est plus nécessaire. **Le ramasse-miettes utilise les informations suivantes pour déterminer si un objet est actif** :

- *Racines de la pile* : Variables de pile fournies par le compilateur et le parcoureur de pile
- *Descripteurs de ramasse-miettes* : Descripteurs pointant vers des objets managés pouvant être référencés depuis le code ou l'environnement d'exécution
- *Données statiques* : Objets statiques des domaines d'application pouvant référencer d'autres objets

**Lors d'un processus de ramassage de déchets, l'environnement d'exécution examine les objets du tas managé pour déterminer s'ils sont encore accessibles par l'application**. ==Pour ce faire, il construit un *graphe d'objets*, représentant chaque objet accessible du tas==. Les graphes d'objets sont expliqués en détail lors de la discussion sur la sérialisation des objets au [[Chapitre 19|Chapitre 19]]. **Pour l'instant, retenez simplement que les graphes d'objets servent à répertorier tous les objets accessibles**. Sachez également que ==le ramasse-miettes ne représente jamais deux fois le même objet dans le graphe, évitant ainsi le problème des *références circulaires* rencontrées en programmation COM==.

Supposons que le tas managé contienne un ensemble d'objets nommés A, B, C, D, E, F et G. **Lors du ramassage de déchets, ces objets** (ainsi que toutes les références internes qu'ils peuvent contenir) **sont examinés**. ==Une fois le graphe construit, les objets inaccessibles== (que l'on peut supposer être les objets C et F) ==sont marqués comme déchets==. L'image suivante illustre un graphe d'objets possible pour le scénario qui vient d'être décrit (vous pouvez lire les flèches directionnelles en utilisant l'expression dépend de ou nécessite ; par exemple, E dépend de G et B, A ne dépend de rien, etc.).

![[Figure 9.3.png|Les graphe d'objets sont construits pour déterminer quelles objets sont accessible par les racines de l'application]]

**Une fois les objets marqués pour suppression** (C et F dans ce cas, car ils ne sont pas pris en compte dans le graphe d'objets), **ils sont supprimés de la mémoire**. ***==À ce stade, l'espace restant sur le tas est compacté, ce qui entraîne la modification par l'environnement d'exécution de l'ensemble des pointeurs sous-jacents afin qu'ils pointent vers l'emplacement mémoire correct==*** (cette opération est effectuée automatiquement et de manière transparente). **Enfin, le pointeur d'objet suivant est réajusté pour pointer vers le prochain emplacement disponible**. L'image suivante illustre ce réajustement.

![[Figure 9.4.png|Un tas nettoyé et compacté]]

>[!note] 
>À proprement parler, le ramasse-miettes utilise deux tas distincts, dont l'un est spécifiquement utilisé pour stocker les objets volumineux. Ce tas est moins fréquemment consulté pendant le cycle de collecte, compte tenu des pertes de performance potentielles liées au déplacement des objets volumineux. Dans .NET Core, le tas volumineux peut être compacté à la demande ou lorsque des limites matérielles optionnelles d'utilisation de la mémoire (absolue ou en pourcentage) sont atteintes.

>[!example] Faire le lien avec la gestion de la mémoire en C (Boot.dev et Gemini)
>
>le GC de .NET est un **"Mark-and-Sweep"** à la base, mais sous stéroïdes. On l'appelle plus précisément un **"Generational Tracing Garbage Collector"**.
>
>Voici les trois couches de complexité qui s'ajoutent au simple Mark-and-Sweep :
>
>1. **Il est Générationnel (Expliquée dans la section suivante)**
>
>2. **Mark-and-Compact** (pas seulement Sweep)
>
>	En C, le `malloc` laisse des "trous" dans la mémoire (fragmentation).
>
>	- Le Mark-and-Sweep classique se contente de marquer les trous comme "libres".
>	- **Le GC de .NET fait un "Compact" :** Après avoir supprimé les objets morts, il **déplace** les objets survivants dans la RAM pour qu'ils soient tous collés les uns aux autres.
>	- **Conséquence :** Cela permet de créer de nouveaux objets ultra-rapidement (il suffit de déplacer le curseur, comme sur la **Pile**), mais cela oblige le GC à mettre à jour toutes les références (adresses) dans votre code.
>
>3. **Le cas particulier : LOH** (Large Object Heap)
>
>	Pour les très gros objets (plus de 85 000 octets, comme les grands tableaux), le GC ne fait **pas** de compactage (trop coûteux de déplacer des Mo de données). Pour eux, il utilise un **Mark-and-Sweep** classique, comme ce que vous avez vu en C.
>
>Quand l'auteur parle du **Graphe d'objets**, il décrit l'étape de **Mark** (le marquage). Le GC parcourt les racines, construit le graphe, et tout ce qui n'est pas dans le graphe est "balayé" (Sweep) puis le reste est compacté.
>
>---
>*C'est précisément parce que le GC **déplace** les objets en mémoire que les pointeurs directs en C sont interdits ou considérés comme `unsafe` en C#. Si vous aviez l'adresse mémoire d'un objet et que le GC passe, l'objet pourrait avoir déménagé sans vous prévenir !*

# Comprendre les générations d'objets

**Lorsque l'environnement d'exécution tente de localiser des objets inaccessibles, il n'examine pas littéralement chaque objet placé sur le tas géré**. ==Cela prendrait évidemment un temps considérable, surtout dans les applications de grande taille== (c'est-à-dire en conditions réelles).

==Pour optimiser ce processus, chaque objet du tas est affecté à une "génération" spécifique==. **Le principe des générations est simple : plus un objet existe longtemps sur le tas, plus il a de chances d'y rester**. Par exemple, la classe qui définit la fenêtre principale d'une application de bureau restera en mémoire jusqu'à la fin du programme. À l'inverse, les objets récemment placés sur le tas (comme un objet alloué dans la portée d'une méthode) deviendront probablement inaccessibles assez rapidement. ==Compte tenu de ces hypothèses, chaque objet du tas appartient à une collection de l'une des générations suivantes :==

- *Génération 0* : Identifie un objet nouvellement alloué qui n’a jamais été marqué pour la collecte (à l’exception des objets volumineux, qui sont initialement placés dans une collecte de génération 2). La plupart des objets sont récupérés pour le ramasse-miettes en génération 0 et survivent donc jusqu’à la génération 1.
- *Génération 1* : Identifie un objet qui a survécu à une collecte de déchets. Cette génération sert également de tampon entre les objets de courte durée de vie et les objets de longue durée de vie.
- *Génération 2* : Identifie un objet qui a survécu à plus d’un passage du ramasse-miettes ou un objet particulièrement volumineux qui a commencé dans une collecte de génération 2.

>[!note] 
>Les générations 0 et 1 sont appelées *générations éphémères*. Comme expliqué dans la section suivante, vous verrez que le processus de nettoyage de la mémoire traite différemment les générations éphémères.

**Le ramasse-miettes examinera d'abord tous les objets de génération 0**. ==Si le marquage et le nettoyage (ou, plus simplement, la suppression) de ces objets libèrent la quantité de mémoire requise, les objets restants sont promus en génération 1==. Pour comprendre comment la génération d'un objet influence le processus de collecte, reportez-vous à l'image suivante, qui illustre la promotion d'un ensemble d'objets de génération 0 (A, B et E) une fois la mémoire requise libérée.

![[Figure 9.5.png|Les objets de génération 0 qui survivent au ramasse miettes sont promus à la génération 1]]

***==Si tous les objets de génération 0 ont été évalués mais que de la mémoire supplémentaire est toujours nécessaire, les objets de génération 1 sont alors examinés afin de déterminer leur accessibilité et sont collectés en conséquence==***. Les objets de génération 1 survivants sont ensuite promus en génération 2. Si le ramasse-miettes a encore besoin de mémoire supplémentaire, les objets de génération 2 sont évalués. À ce stade, **si un objet de génération 2 survit à un passage de ramasse-miettes, il reste un objet de génération 2, compte tenu de la limite supérieure prédéfinie des générations d'objets**.

**En résumé, en attribuant une valeur de génération aux objets sur le tas, les objets les plus récents (==tels que les variables locales==) seront supprimés rapidement, tandis que les objets plus anciens (==tels que la fenêtre principale d'un programme==) ne seront pas « dérangés » aussi souvent**.

Le système déclenche le GC selon **4 critères principaux** :

1. **La saturation des budgets de génération (Le cas n'°1)**

	Chaque génération (0, 1, 2) dispose d'un "budget" en mégaoctets (géré dynamiquement par .NET). Lorsque vous faites un `newobj` en CIL, .NET tente d'allouer l'objet en Gen 0. Si le budget de la Gen 0 est dépassé, **l'allocation bloque une microseconde et déclenche immédiatement un GC de Génération 0** pour faire de la place.

2. **La mémoire globale du système est basse**

	Le système d'exploitation (Windows, Linux, macOS) peut envoyer une notification d'alerte à la machine virtuelle .NET (le CLR) pour lui dire : _"Attention, la mémoire physique (RAM) de la machine est saturée"_. Le CLR ordonne alors immédiatement au GC de lancer un nettoyage complet (Full GC : Gen 0, 1 et 2) pour restituer de la mémoire à l'OS.

3. **Le processus dépasse la limite fixée**

	Si votre programme tourne dans un conteneur (comme Docker) ou un serveur IIS avec une limite stricte de mémoire (ex: maximum 500 Mo), le CLR déclenche le GC dès que l'application s'approche de ce plafond pour éviter que le système d'exploitation ne tue brutalement le processus.

4. **L'appel manuel (À éviter)**

	Si le développeur écrit explicitement la ligne `GC.Collect();` dans son code C#. C'est généralement une mauvaise pratique, car le moteur de .NET s'auto-calibre beaucoup mieux tout seul au runtime. 

**Si tout cela vous semble idéal et préférable à la gestion manuelle de la mémoire, n'oubliez pas que le processus de ramasse-miettes a un coût**. ==Le moment du ramassage des ordures et les éléments collectés sont généralement hors du contrôle des développeurs, bien que le ramassage des ordures puisse certainement être influencé, positivement ou négativement==. **Lors de son exécution, le ramassage des ordures utilise des cycles CPU et peut affecter les performances de l'application**. Les sections suivantes examinent les différents types de ramassage des ordures.

## Générations et segments éphémères

Comme mentionné précédemment, **les générations 0 et 1 sont de courte durée et sont appelées *générations éphémères***. ==Ces générations sont allouées dans un segment mémoire appelé *segment éphémère*==. Lors du passage du ramasse-miettes, les nouveaux segments acquis deviennent de nouveaux segments éphémères, et le segment contenant les objets ayant survécu à la génération 1 devient le nouveau segment de génération 2. 

**La taille du segment éphémère varie en fonction de plusieurs facteurs, tels que le type de ramasse-miettes** (abordé ci-après) **et l'architecture du système**. Le [[#Tableau 9-1 La taille des segments éphémères|Tableau 9-1]] présente les différentes tailles des segments éphémères.

##### Tableau 9-1: La taille des segments éphémères

| Type de ramasse miette             | 32-bit  | 64-bit   |
| ---------------------------------- | ------- | -------- |
| Poste de travail                   | $16$ MB | $256$ MB |
| Serveur                            | $64$ MB | $4$ GB   |
| Serveur avec 4+ processeur logique | $32$ MB | $2$ GB   |
| Serveur avec 8+ processeur logique | $16$ MB | $1$ GB   |

>[!warning]- Ces valeurs ne sont plus à l'order du jour
>Les valeurs indiquées dans le livre (généralement **16 Mo** pour un OS 32 bits et **128 Mo** ou plus pour un OS 64 bits) sont des **valeurs par défaut de base**, mais le moteur .NET moderne est devenu beaucoup plus dynamique :
>
>- **Adaptation au CPU :** Sur les architectures modernes (y compris big.LITTLE), le GC ajuste la taille des segments en fonction du **nombre de cœurs logiques** et de la **quantité de RAM totale**.
>- **Mode Serveur vs Station de travail :** Si votre application tourne en mode "Server GC", les segments sont beaucoup plus grands et multipliés par le nombre de cœurs. Sur un CPU récent à 16 cœurs, vous aurez 16 segments distincts, ce qui n'était pas le cas sur les vieux CPU mono-cœur.
>- **Conteneurs (Docker/Kubernetes) :** .NET est maintenant capable de réduire ces tailles si vous limitez la mémoire du conteneur, pour éviter que le GC ne soit trop gourmand. 

# Types de récupération de mémoire

Le runtime propose deux types de récupération de mémoire :

- *Récupération de mémoire poste de travail* : Conçue pour les applications clientes, ==elle est le comportement par défaut pour les applications autonomes==. La récupération de mémoire côté poste de travail peut s'effectuer en arrière-plan (voir ci-après) ou de manière non concurrente.
- *Récupération de mémoire serveur* : Conçue pour les applications serveur exigeant un débit et une évolutivité élevés, la récupération de mémoire côté serveur peut s'effectuer en arrière-plan ou de manière non concurrente, tout comme la récupération de mémoire côté poste de travail.

>[!note]
>Ces noms correspondent aux paramètres par défaut pour les applications de poste de travail et de serveur, mais **la méthode de récupération de mémoire est configurable via le fichier** *runtimeconfig.json* **de la machine ou les variables d'environnement système**. À moins que l'ordinateur ne possède qu'un seul processeur, auquel cas il utilisera toujours la récupération de mémoire de poste de travail.

**Le GC du poste de travail s'exécute sur le même thread qui l'a déclenché et conserve la même priorité qu'au moment de son déclenchement**. ==Cela peut engendrer des conflits avec d'autres threads de l'application==.

**Le GC du serveur s'exécute sur plusieurs threads dédiés, configurés au niveau de priorité `THREAD_PRIORITY_HIGHEST`** (le fonctionnement des threads est abordé au [[Chapitre 15#La relation Processus/Domaine d'application/Contexte/Thread|Chapitre 15]]). ==Chaque processeur dispose d'un tas et d'un thread dédiés pour effectuer le GC. Cela peut rendre le GC du serveur très gourmand en ressources==.

## Collecte des déchets en arrière-plan

==À partir de .NET 4.0 (et dans .NET Core), le ramasse-miettes est capable de gérer la suspension des threads lorsqu'il nettoie les objets du tas managé, grâce au *ramasse-miettes en arrière-plan*==. **Malgré son nom, cela ne signifie pas que toute la collection s'effectue désormais sur des threads d'arrière-plan supplémentaires**. En effet, ==si un ramasse-miettes en arrière-plan est en cours pour des objets appartenant à une génération non éphémère, le runtime .NET Core est désormais capable de collecter les objets des générations éphémères à l'aide d'un thread d'arrière-plan dédié==.

Par ailleurs, **le ramasse-miettes de .NET 4.0 et versions ultérieures a été amélioré afin de réduire davantage la durée de suspension d'un thread donné impliqué dans les détails du ramasse-miettes**. Grâce à ces modifications, ==le processus de nettoyage des objets inutilisés appartenant aux générations 0 ou 1 est optimisé, ce qui peut améliorer les performances d'exécution de vos programmes== (un point crucial pour les systèmes temps réel qui exigent un temps d'arrêt du ramasse-miettes court et prévisible). 

**Sachez toutefois que l'introduction de ce nouveau modèle de ramasse-miettes n'a aucune incidence sur la façon dont vous développez vos applications .NET Core**. En pratique, vous pouvez simplement laisser le ramasse-miettes effectuer son travail sans votre intervention directe (et vous réjouir que les équipes de Microsoft améliorent le processus de ramasse-miettes de manière transparente).

# Le type `System.GC`

L'assembly *mscorlib.dll* fournit un type de classe nommé `System.GC` qui permet d'interagir par programmation avec le ramasse-miettes à l'aide d'un ensemble de membres statiques. **Notez cependant que vous aurez rarement** (voire jamais) **besoin d'utiliser directement cette classe dans votre code**. ==En général, vous n'utiliserez les membres de `System.GC` que lors de la création de classes utilisant des ressources *non managées en interne*==. 

Ce pourrait être le cas si vous développez une classe effectuant des appels à l'API Windows basée sur C via le protocole d'invocation de la plateforme .NET Core, ou encore en raison d'une logique d'interopérabilité COM complexe et de bas niveau. Le [[#Tableau 9-2 Sélection de membres du type `System.GC`|Tableau 9-2]] présente un aperçu de certains des membres les plus importants (consultez la documentation du SDK .NET Framework pour plus de détails).

##### Tableau 9-2: Sélection de membres du type `System.GC`

| Membre `System.GC`                                | Description                                                                                                                                                                                                                                                                                                                 |
| ------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `AddMemoryPressure()`<br>`RemoveMemoryPressure()` | **Permet de spécifier une valeur numérique représentant le "niveau d'urgence" de l'objet appelant concernant le processus de nettoyage de la mémoire**. Sachez que ces méthodes doivent modifier la pression de manière concomitante et, par conséquent, ne jamais retirer plus de pression que la quantité totale ajoutée. |
| `Collect()`                                       | **Force le GC à effectuer un nettoyage de la mémoire**. Cette méthode a été surchargée pour spécifier la génération à collecter, ainsi que le mode de collecte (via l'énumération `GCCollectionMode`).                                                                                                                      |
| `CollectionCount()`                               | **Renvoie une valeur numérique représentant le nombre de fois où une génération donnée a été balayée.**                                                                                                                                                                                                                     |
| `GetGeneration()`                                 | **Renvoie la génération à laquelle appartient actuellement un objet.**                                                                                                                                                                                                                                                      |
| `GetTotalMemory()`                                | **Renvoie la quantité estimée de mémoire** (en octets) **actuellement allouée sur le tas géré**. ==Un paramètre booléen indique si l'appel doit attendre la fin du processus de nettoyage de la mémoire avant de retourner==.                                                                                               |
| `MaxGeneration`                                   | **Renvoie le nombre maximal de générations prises en charge sur le système cible**. Sous Microsoft .NET 4.0, trois générations sont possibles : 0, 1 et 2.                                                                                                                                                                  |
| `SuppressFinalize()`                              | **Définit un indicateur signalant que la méthode `Finalize()` de l'objet spécifié ne doit pas être appelée**.                                                                                                                                                                                                               |
| `WaitForPendingFinalizers()`                      | **Suspend le thread courant jusqu'à ce que tous les objets finalisables aient été finalisés**. Cette méthode est généralement appelée directement après l'invocation du `GC.Collect()`.                                                                                                                                     |

>[!info] Petite précision sur les générations 
>l'implémentation physique sépare désormais les objets non seulement par leur **âge** (Générations), mais aussi par leur **taille** (LOH), leur **statut** (POH pour le pinning) et leur **mutabilité** (Frozen Segments).

Pour illustrer comment le type `System.GC` peut être utilisé pour obtenir diverses informations relatives au ramasse-miettes, mettez à jour les instructions de niveau supérieur de votre projet *SimpleGC* comme suit, qui utilise plusieurs membres de GC :

```cs
using SimpleGC;

Console.Title = "Fun with System.GC";
Console.WriteLine("**** Fun with System.GC ****\n");

// Affiche le nombre estimé d'octets sur le tas.
Console.WriteLine($"Estimated bytes on heap: {GC.GetTotalMemory(false)}");

// MaxGeneration est indexé à partir de zéro,
// donc ajouter 1 pour l'affichage
Console.WriteLine($"This OS has {GC.MaxGeneration + 1} object generations.\n");

Car refToMyCar = new Car("Zippy", 100);
Console.WriteLine(refToMyCar.ToString());

// Affiche la génération de l'objet refToMyCar.
Console.WriteLine(
    $"Generation of refToMyCar is: {GC.GetGeneration(refToMyCar)}"
);

Console.ReadLine();
```

Après avoir exécuté cette commande, vous devriez obtenir un résultat similaire à celui-ci :

```
**** Fun with System.GC ****

Estimated bytes on heap: 80680
This OS has 3 object generations.

Zippy is going 100 km/h
Generation of refToMyCar is: 0
```

Vous explorerez plus en détail les méthodes présentées dans le [[#Tableau 9-2 Sélection de membres du type `System.GC`|Tableau 9-2]] dans la section suivante.

## Forcer le nettoyage de la mémoire

==Le rôle principal du ramasse-miettes est de gérer la mémoire automatiquement==. Cependant, **dans certains cas rares, il peut être utile de forcer le nettoyage de la mémoire par programmation à l'aide de `GC.Collect()`**. ==Voici deux situations courantes où il peut être judicieux d'interagir avec le processus de nettoyage== :

- Votre application est **sur le point d'entrer dans un bloc de code que vous ne souhaitez pas interrompre par une éventuelle opération de nettoyage de la mémoire**.
- Votre application **vient de terminer l'allocation d'un très grand nombre d'objets, et vous souhaitez libérer le maximum de mémoire allouée le plus rapidement possible**.

==Si vous estimez qu'il pourrait être utile que le ramasse-miettes vérifie la présence d'objets inaccessibles, vous pouvez déclencher explicitement un nettoyage de la mémoire==, comme suit :

```cs
...
// Force un nettoyage de la mémoire et
// attend que chaque objet soit finalisé.
GC.Collect();
GC.WaitForPendingFinalizers();
...
```

**Lorsque vous forcez manuellement le ramasse-miettes, vous devez toujours appeler `GC. WaitForPendingFinalizers()`**. ==Cette approche vous garantit que tous les objets finalisables== (décrits dans la section suivante) ==auront eu la possibilité d'effectuer le nettoyage nécessaire avant que votre programme ne poursuive son exécution==. **En interne, `GC.WaitForPendingFinalizers()` suspend le thread appelant pendant le processus de ramasse-miettes**. C'est une bonne chose, car ==cela empêche votre code d'appeler des méthodes sur un objet en cours de destruction !==

**La méthode `GC.Collect()` peut également recevoir une valeur numérique identifiant la génération la plus ancienne sur laquelle le ramasse-miettes sera effectué**. Par exemple, pour indiquer au runtime d'examiner uniquement les objets de la génération 0, vous écrirez le code suivant :

```cs
...
// N'examiner que les objets de génération 0.
GC.Collect(0);
GC.WaitForPendingFinalizers();
...
```

**De plus, la méthode `Collect()` peut recevoir une valeur de l'énumération `GCCollectionMode` comme second paramètre, afin de préciser comment le runtime doit forcer le ramasse-miettes**. Cette énumération définit les valeurs suivantes :

```cs
public enum GCCollectionMode

{ 
	Default,  // Forced est le mode par défaut actuel.
	Forced,   // Indique au runtime de collecter immédiatement !
	Optimized // Permet au runtime de déterminer si le moment actuel est optimal pour récupérer les objets.
}
```

Comme pour tout ramasse-miettes, **l'appel à `GC.Collect()` favorise les générations survivantes**. À titre d'exemple, supposons que vos instructions de niveau supérieur aient été mises à jour comme suit :

```cs
using SimpleGC;

Console.Title = "Fun with System.GC";
Console.WriteLine("**** Fun with System.GC ****\n");

// Affiche le nombre estimé d'octets sur le tas.
Console.WriteLine($"Estimated bytes on heap: {GC.GetTotalMemory(false)}");

// MaxGeneration est indexé à partir de zéro,
// donc ajouter 1 pour l'affichage
Console.WriteLine($"This OS has {GC.MaxGeneration + 1} object generations.\n");

Car refToMyCar = new Car("Zippy", 100);
Console.WriteLine(refToMyCar.ToString());

// Affiche la génération de l'objet refToMyCar.
Console.WriteLine(
    $"Generation of refToMyCar is: {GC.GetGeneration(refToMyCar)}"
);

// Créez une multitude d'objets à des fins de test.
object[] tonsOfObjects = new object[50000];
for (int i = 0; i < 50000; i++)
{
    tonsOfObjects[i] = new object();
}

// Nettoye uniquement les objets de la génération 0.
Console.WriteLine("Force Garbage Collection");
GC.Collect(0, GCCollectionMode.Forced);
GC.WaitForPendingFinalizers();

// Affiche la génération de refToMyCar
Console.WriteLine(
    $"Generation of refToMyCar is: {GC.GetGeneration(refToMyCar)}"
);

// Vérifier si tonsOfObjects[9000] est toujours actif.
if (tonsOfObjects[9000] != null)
{
    Console.WriteLine(
        $"Generation of tonsOfObjects[9000] is: {GC.GetGeneration(tonsOfObjects[9000])}"
    );
}
else
{
    Console.WriteLine("tonsOfObjects[9000] is no longer alive.");
}

// Afficher le nombre de fois où une génération a été balayée.
Console.WriteLine($"\nGen 0 has been swept {GC.CollectionCount(0)} times.");
Console.WriteLine($"Gen 1 has been swept {GC.CollectionCount(1)} times.");
Console.WriteLine($"Gen 2 has been swept {GC.CollectionCount(2)} times.");

Console.ReadLine();
```

Ici, j'ai volontairement créé un grand tableau de types d'objets (50 000 exactement) à des fins de test. Voici le résultat du programme :

```
**** Fun with System.GC ****

Estimated bytes on heap: 80680
This OS has 3 object generations.

Zippy is going 100 km/h
Generation of refToMyCar is: 0
Force Garbage Collection
Generation of refToMyCar is: 1
Generation of tonsOfObjects[9000] is: 1

Gen 0 has been swept 1 times.
Gen 1 has been swept 0 times.
Gen 2 has been swept 0 times.
```

>[!note] On peut distinguer deux **intentions** ou "types logiques" pour `null` dans votre code
>1. Le `null` "Effaceur" (Action)
>	- **L'objectif :** Rompre un lien.
>	- **Ce qui se passe :** Vous écrasez une adresse mémoire existante par `0`.
>	- **L'effet :** Cela libère l'objet sur le **Tas** pour le Garbage Collector. C'est un acte de gestion de mémoire.
>
>2. Le `null` "Valeur" (État)
>	- **L'objectif :** Dire "je n'ai pas encore de données" ou "aucune sélection n'a été faite".
>	- **Exemple :** `Car myCar = null;` au début d'un programme.
>	- **L'effet :** La place est réservée sur la **Pile**, mais elle pointe vers le néant dès le départ.

À ce stade, j'espère que vous comprenez mieux les détails de la durée de vie des objets. ==Dans la section suivante, vous examinerez plus en détail le processus de récupération de mémoire en apprenant à créer des *objets finalisables*, ainsi que des *objets jetables*==. Sachez que les techniques suivantes ne sont généralement nécessaires que si vous créez des classes C# qui gèrent des ressources internes non managées.

>[!tip] Avec le code précédent, on comprend aussi pourquoi gérer manuellement la collecte n'est pas une bonne pratique car on fait passer des éléments de génération 0 à génération 1 pour "rien". 

# Création d'objets finalisables

Au [[Chapitre 6#Tableau 6-1 Membres principaux de `System.Object`|Chapitre 6]], vous avez appris que la classe de base suprême de .NET Core, `System.Object`, définit une méthode virtuelle nommée `Finalize()`. L'implémentation par défaut de cette méthode ne fait absolument rien.

```cs
// System.Object
public class Object
{
	...
	protected virtual void Finalize() {}
}
```

**Lorsque vous redéfinissez la méthode `Finalize()` pour vos classes personnalisées, vous définissez un emplacement spécifique pour exécuter la logique de nettoyage nécessaire à votre type**. Comme ce membre est protégé, ==il est impossible d'appeler directement la méthode `Finalize()` d'un objet depuis une instance de classe via l'opérateur point==. **Le *ramasse-miettes* appellera plutôt la méthode `Finalize()` de l'objet** (si elle est prise en charge) **avant de supprimer l'objet de la mémoire**.

>[!note]
>**Il est interdit de redéfinir la méthode `Finalize()` sur les types structure**. Cela est parfaitement logique étant donné que ==les structures sont des types valeur, qui ne sont jamais alloués sur le tas et ne sont donc pas collectés par le ramasse-miettes== ! Cependant, **si vous créez une structure contenant des ressources non gérées qui doivent être libérées, vous pouvez implémenter l’interface `IDisposable`** (décrite prochainement). Rappelez-vous du [[Chapitre 4#Utilisation des `ref struct` (Nouveauté C 7.2)|Chapitre 4]] que les structures de référence et les structures de référence en lecture seule ne peuvent pas implémenter d’interface, mais peuvent implémenter une méthode `Dispose()`.

Bien sûr, **un appel à `Finalize()` se produira** (eventuellement) **lors d'un ramassage de déchets «naturel» ou éventuellement lorsque vous forcez un ramassage par programmation via `GC.Collect()`**. Dans les versions précédentes de .NET (à l'exception de .NET Core), le finaliseur de chaque objet était appelé à l'arrêt de l'application. ==Dans .NET Core, il n'existe aucun moyen de forcer l'exécution du finaliseur, même à l'arrêt de l'application==.

**Or, malgré ce que votre intuition de développeur pourrait vous suggérer, la grande majorité de vos classes C# n'auront pas nécessité de logique de nettoyage explicite ni de finaliseur personnalisé**. La raison est simple : si vos classes utilisent simplement d'autres objets managés, tout sera finalement collecté par le ramasse-miettes. ==Le seul cas où vous auriez besoin de concevoir une classe capable de se nettoyer elle-même est lorsque vous utilisez des *ressources non managées*== (telles que des descripteurs de fichiers système bruts, des connexions à des bases de données non managées brutes, des blocs de mémoire non managée ou d'autres ressources non managées). ==Sous la plateforme .NET Core, les ressources non managées sont obtenues soit directement en appelant l'API du système d'exploitation via les services d'invocation de plateforme (*PinVoke*), soit par le biais de scénarios d'interopérabilité COM complexes==. **Compte tenu de cela, considérons la règle suivante du ramasse-miettes.**

>[!warning] Règle
>***La seule raison valable de surcharger `Finalize()` est si votre classe C# utilise des ressources non managées via PInvoke ou des tâches d'interopérabilité COM complexes (généralement via différents membres définis par le type `System.Runtime.InteropServices.Marshal`). En effet, dans ces cas, vous manipulez de la mémoire que le runtime ne peut pas gérer.***
>>[!tip] Encore plus stricte pour les versions modernes de C#
>>**Pour 99,9 % des cas, on utilise l'interface `IDisposable` et la structure `SafeHandle`**. Microsoft recommande aujourd'hui d'utiliser `SafeHandle` (une classe du framework) pour encapsuler des ressources non managées plutôt que d'écrire son propre finaliseur.
>>- **La règle d'or :** Si vous n'utilisez pas de pointeurs `IntPtr` vers de la mémoire C/C++ ou des handles Windows bruts, n'écrivez **jamais** de finaliseur.

## Rappel sur le Code Non Managé (*Unmanaged Code*)

Le **code non managé** désigne tout code qui s'exécute **en dehors du contrôle du CLR** (.NET Runtime). Contrairement au code C#, il n'est pas compilé en CIL et n'est pas surveillé par le Garbage Collector.

### Les 3 sources principales :

1. **Le Système d'Exploitation :** Les API Windows (Win32), macOS (Quartz/Cocoa) ou Linux écrites en C/C++.
2. **L'Interopérabilité COM :** Les vieux composants logiciels (ex: moteur d'Excel, DirectShow).
3. **La Mémoire Manuelle :** Les blocs de RAM alloués avec `malloc` ou `HGlobal`, où vous devez gérer vous-même le cycle de vie.

### Pourquoi est-ce dangereux / spécifique ?

- **Pas de Garbage Collection :** Si vous oubliez de libérer une ressource, elle reste en RAM jusqu'à la fermeture du PC (**Memory Leak**).
- **Gestion des "Handles" :** Ces ressources sont identifiées par des `IntPtr` (un simple nombre représentant une adresse mémoire ou un identifiant système).
- **Incompatibilité de mouvement :** Le GC déplace les objets en RAM pour compacter le tas, mais le code non managé s'attend à ce que les données ne bougent **jamais**.

Dans le framework .NET, on utilise `IDisposable` (pour un nettoyage propre et immédiat) et, en dernier recours, `Finalize()` (pour éviter une fuite si le développeur oublie d'appeler `Dispose`).

## Redéfinir `System.Object.Finalize()`

==**Dans le cas rare où vous développez une classe C# utilisant des ressources non managées**, vous voudrez évidemment vous assurer que la mémoire sous-jacente est libérée de manière prévisible==. Supposons que vous ayez créé un nouveau projet d'application console C# nommé *SimpleFinalize* et inséré une classe nommée `MyResourceWrapper` qui utilise une ressource non managée (quelle qu'elle soit) et que vous souhaitiez surcharger la méthode `Finalize()`. **La particularité en C#, il est impossible d'effectuer cette opération avec le mot-clé `override` attendu**.

```cs
namespace SimpleFinalize;

class MyResourceWrapper
{
    // Erreur lors de la compilation!
    protected override void Finalize() { }
}
```

En revanche, **lorsque vous souhaitez configurer vos types de classes C# personnalisés pour redéfinir la méthode `Finalize()`, vous utilisez la syntaxe des destructeurs** (semblable à celle de C++) **pour obtenir le même résultat**. ==Cette forme alternative de redéfinition d'une méthode virtuelle est justifiée par le fait que, lors du traitement de la syntaxe du finaliseur par le compilateur C#, celui-ci ajoute automatiquement une grande partie de l'infrastructure nécessaire à la méthode `Finalize()` implicitement redéfinie== (comme illustré ci-après).

**Les finaliseurs C# ressemblent aux constructeurs, car ils portent le même nom que la classe dans laquelle ils sont définis**. ==De plus, les finaliseurs sont préfixés par un tilde (`~`)==. **Contrairement à un constructeur, cependant, un finaliseur ne prend jamais de modificateur d'accès** (il est implicitement `protected`), **ne prend jamais de paramètres et ne peut pas être surchargé** (==un seul finaliseur par classe==).

**Voici un finaliseur personnalisé pour `MyResourceWrapper` qui émet un bip système lors de son appel**. Cet exemple est fourni à titre purement pédagogique. ==Un finaliseur réel ne ferait rien de plus que libérer les ressources non gérées et n'interagirait pas avec les autres objets gérés, même ceux référencés par l'objet courant, car on ne peut pas supposer qu'ils soient encore actifs au moment où le ramasse-miettes appelle la méthode `Finalize()`==.

```cs
namespace SimpleFinalize;

// Surcharge de System.Object.Finalize() via la syntaxe du finaliseur.
class MyResourceWrapper
{
    // Nettoyage des ressources non gérées ici.
    // Émettre un bip lors de la destruction
    // (à des fins de test uniquement !)
    ~MyResourceWrapper() => Console.Beep();
}
```

Si vous examiniez ce destructeur C# avec *ildasm.exe*, vous constateriez que ==le compilateur insère le code de vérification d'erreurs nécessaire==. Tout d'abord, **les instructions de code de votre méthode `Finalize()` sont placées dans un bloc `try`** (voir [[Chapitre 7#L'exemple le plus simple possible|Chapitre 7]]). **Le bloc finally associé garantit que la méthode `Finalize()` de vos classes de base sera toujours exécutée, quelles que soient les exceptions rencontrées dans le bloc `try`**.

```CIL
.method family hidebysig virtual instance void 
        Finalize() cil managed
{
  .override [System.Runtime]System.Object::Finalize
  // Code size       17 (0x11)
  .maxstack  1
  .try
  {
    IL_0000:  call       void [System.Console]System.Console::Beep()
    IL_0005:  nop
    IL_0006:  leave.s    IL_0010
  }  // end .try
  finally
  {
    IL_0008:  ldarg.0
    IL_0009:  call       instance void [System.Runtime]System.Object::Finalize()
    IL_000e:  nop
    IL_000f:  endfinally
  }  // end handler
  IL_0010:  ret
} // end of method MyResourceWrapper::Finalize
```

Si vous testiez ensuite le type `MyResourceWrapper`, vous constateriez qu'un bip système se produit lors de l'exécution du finaliseur.

>L'exemple de code est différent par rapport au livre! L'explication sera donnée dans une légende un peu plus loin.

```cs
using SimpleFinalize;

Console.Title = "Fun with Finalizers";
Console.WriteLine("**** Fun with Finalizers ****\n");

Console.WriteLine("Hit return to create the objects ");
Console.WriteLine("then force the GC to invoke Finalize()");

//Selon la puissance de votre système,
//vous devrez peut-être augmenter ces valeurs
CreateObjects(1_000_000);

//Augmenter artificiellement la pression sur la mémoire
GC.AddMemoryPressure(2147483647);
GC.Collect(0, GCCollectionMode.Forced);
GC.WaitForPendingFinalizers();

Console.ReadLine();

static void CreateObjects(int count)
{
    MyResourceWrapper[] tonsOfObjects = new MyResourceWrapper[count];
    for (int i = 0; i < count; i++)
    {
        tonsOfObjects[i] = new MyResourceWrapper();
    }
    tonsOfObjects = null;
}
```

>[!note] 
>La seule façon de garantir que cette petite application console forcera le ramasse-miettes dans .NET Core est de créer une grande quantité d'objets en mémoire, puis de les initialiser à null. Si vous exécutez cet exemple d'application, assurez-vous d'appuyer sur la combinaison de touches Ctrl+C pour arrêter l'exécution du programme et tous les bips !

>[!warning] Il y a une grosse disparité entre le livre et le comportement moderne de .NET sut cet exemple.
>Aujourd'hui, avec .NET 6/8/9 :
>
>- Le GC est devenu extrêmement intelligent. Créer un million d'objets vides peut être optimisé par le compilateur ou le JIT.
>- **La pression mémoire :** Le GC moderne ne se laisse plus "forcer" aussi facilement. Il analyse si l'ordinateur a encore de la RAM disponible. Si vous avez 16 Go de RAM, créer 1 million de petits objets ne représente presque rien pour lui, donc il ne se presse pas pour les finaliser.
>
>La note mentionne d'appuyer sur `Ctrl+C` pour arrêter les bips.
>
>- **Sur Windows (historiquement) :** `Console.Beep()` envoyait une instruction directe au haut-parleur interne. C'était synchrone et bruyant.
>- **Sur macOS et Linux :** `Console.Beep()` est souvent ignoré ou envoyé au système de notification qui le met en sourdine s'il y en a trop. Vous n'aurez jamais cette "cacophonie" de bips sur un Mac moderne.
>
>Pour garantir qu'une collection a lieu, on utilise simplement `GC.Collect()`.
>- L'auteur suggère de créer "énormément d'objets" pour simuler une situation réelle où le GC n'a plus le choix. Mais pour un exercice d'apprentissage, c'est une méthode "brute" qui ne fonctionne pas de la même manière sur toutes les machines (un Mac M1/M2 gère cela bien mieux qu'un vieux PC).
>
>L'auteur veut marquer l'esprit. Il veut que faire comprendre que :
>
>1. Le GC se déclenche quand la mémoire manque (**pression mémoire**).
>2. Le processus de finalisation est **asynchrone** (il continue de biper même après que le code principal a avancé).
>
>Ce que vous devriez faire pour votre test :
>Ne suivez pas le conseil du "1 million d'objets" pour entendre des bips. Faites ceci à la place :
>
>1. Utilisez **100 objets**.
>2. Remplacez le bip par un `Console.WriteLine("Nettoyage de l'objet n°" + id)`.
>3. Utilisez `GC.Collect()` explicitement.
>
>**C'est la seule façon "propre" de voir le mécanisme sans saturer votre système.**

## Explication détaillée du processus de finalisation

**Il est important de toujours garder à l'esprit que le rôle de la méthode `Finalize()` est de garantir qu'un objet .NET Core puisse libérer les ressources non managées lors de son passage dans le garbage collector**. Ainsi, ==si vous créez une classe qui n'utilise pas de mémoire non managée== (cas de loin le plus courant), ==la finalisation est peu utile==. En fait, **dans la mesure du possible, vous devriez concevoir vos types de manière à éviter de prendre en charge une méthode `Finalize()` pour la simple raison que la finalisation prend du temps**.

**Lorsque vous allouez un objet sur le tas managé, le runtime détermine automatiquement si votre objet prend en charge une méthode `Finalize()` personnalisée**. ==Si c'est le cas, l'objet est marqué comme *finalisable* et un pointeur vers cet objet est stocké dans une file d'attente interne appelée *file d'attente de finalisation*==. **La file d'attente de finalisation est une table gérée par le garbage collector qui pointe vers chaque objet qui doit être finalisé avant d'être supprimé du tas**. Lorsque le ramasse-miettes détermine qu'il est temps de libérer un objet de la mémoire, **il examine chaque entrée de la file d'attente de finalisation et copie l'objet du tas vers une autre structure gérée appelée** *table des ressources accessibles pour la finalisation* (souvent abrégée en *freachable* et prononcée « eff-reachable »). **À ce stade, un thread distinct est créé pour appeler la méthode `Finalize()` pour chaque objet de la table des ressources accessibles lors du prochain passage du ramasse-miettes**. De ce fait, ==il faudra au moins deux passages du ramasse-miettes pour finaliser un objet==.

**En résumé, bien que la finalisation d'un objet garantisse la libération des ressources non gérées, elle reste non déterministe par nature et, en raison du traitement supplémentaire en arrière-plan, considérablement plus lente**.

# Création d'objets jetables

Comme vous l'avez vu, **les finaliseurs permettent de libérer les ressources non managées lorsque le ramasse-miettes entre en action**. Cependant, ==étant donné que de nombreux objets non managés sont des «éléments précieux»== (tels que des descripteurs de base de données ou de fichiers bruts), ==il peut être judicieux de les libérer dès que possible plutôt que de compter sur le ramasse-miettes==. **Au lieu de surcharger la méthode `Finalize()`, votre classe peut implémenter l'interface `IDisposable`, qui définit une unique méthode nommée `Dispose()` comme suit** :

```cs
public interface IDisposable
{
	void Dispose();
}
```

**Lors de l'implémentation de l'interface `IDisposable`, on suppose que lorsque l'*utilisateur de l'objet* a terminé de l'utiliser, il appelle manuellement la méthode `Dispose()` avant que la référence à l'objet ne soit libérée**. Ainsi, ==un objet peut effectuer le nettoyage nécessaire des ressources non managées sans être placé dans la file d'attente de finalisation et sans attendre que le ramasse-miettes déclenche la logique de finalisation de la classe==.

>[!note]
>**Les structures normales et les types de classe peuvent tous deux implémenter l'interface `IDisposable`** (contrairement à la redéfinition de la méthode `Finalize()`, réservée aux types de classe), **car c'est l'utilisateur de l'objet** (et non le ramasse-miettes) **qui appelle la méthode `Dispose()`**. Les structures de références jetables ont été abordées au [[Chapitre 4#Utilisation des `ref struct` jetables (Nouveauté C 8.0)|Chapitre 4]].

Pour illustrer l'utilisation de cette interface, créez un nouveau projet d'application console C# nommé *SimpleDispose*. Voici une classe `MyResourceWrapper` mise à jour qui implémente désormais `IDisposable`, au lieu de redéfinir `System.Object.Finalize()` :

```cs
namespace SimpleDispose;

// Implémente IDisposable
class MyResourceWrapper : IDisposable
{
    // L'utilisateur de l'objet doit appeler cette méthode
    // lorsqu'il a terminé avec l'objet.
    public void Dispose()
    {
        // Nettoyage des ressources non gérées...
        // Suppression des autres objets jetables contenus...
        // Juste pour un test.
        Console.WriteLine("***** In Dispose! *****");
    }
}
```

**Notez qu'une méthode `Dispose()` est non seulement responsable de la libération des ressources non managées du type, mais peut également appeler `Dispose()` sur toute autre méthode jetable qu'elle contient**. Contrairement à `Finalize()`, **il est parfaitement sûr de communiquer avec d'autres objets managés au sein d'une méthode `Dispose()`**. La raison est simple : ==le ramasse-miettes ignore l'interface `IDisposable` et n'appellera jamais `Dispose()`==. Par conséquent, ==lorsque l'utilisateur de l'objet appelle cette méthode, l'objet est toujours actif sur le tas managé et a accès à tous les autres objets alloués sur le tas==. La logique d'appel, illustrée ici, est simple :

```cs
using SimpleDispose;

Console.Title = "Fun with Dispose";
Console.WriteLine("**** Fun with Dispose ****\n");

// Créer un objet jetable et appeler Dispose()
// pour libérer les ressources internes.
MyResourceWrapper rw = new MyResourceWrapper();
rw.Dispose();

Console.ReadLine();
```

Bien entendu, **avant d'appeler `Dispose()` sur un objet, vous devez vous assurer que son type implémente l'interface `IDisposable`**. Bien que ==la documentation vous permette généralement de savoir quels types de la bibliothèque de classes de base implémentent `IDisposable`, une vérification par programmation peut être effectuée à l'aide des mots-clés `is` ou `as`==, présentés au [[Chapitre 6#Utilisation du mot-clé C `is` (MaJ C 7.0 C 9.0)|Chapitre 6]].

```cs
using SimpleDispose;

Console.Title = "Fun with Dispose";
Console.WriteLine("**** Fun with Dispose ****\n");

MyResourceWrapper rw = new MyResourceWrapper();
if (rw is IDisposable)
{
    rw.Dispose();
}

Console.ReadLine();
```

Cet exemple met en lumière une autre règle concernant la gestion de la mémoire.

>[!warning] Règle
***Il est conseillé d'appeler la méthode `Dispose()` sur tout objet que vous créez directement si celui-ci implémente l'interface `IDisposable`. Partez du principe que si le concepteur de la classe a choisi d'implémenter cette interface, c'est que le type gère des*** **ressources externes** ***nécessitant un nettoyage explicite***.
>
>Si vous oubliez cet appel, la mémoire RAM finira par être récupérée par le Garbage Collector (pas de panique !), **cependant, les ressources système** (verrous de fichiers, connexions réseau, handles) pourraient rester bloquées inutilement, provoquant des erreurs ou des baisses de performance

**Il existe une exception à la règle précédente**. ==Plusieurs types des bibliothèques de classes de base qui implémentent l'interface `IDisposable` fournissent un alias== (parfois déroutant) ==à la méthode `Dispose()`, afin de rendre cette méthode de suppression plus naturelle pour le type qui la définit==. Par exemple, **bien que la classe `System.IO.FileStream` implémente `IDisposable`** (et prenne donc en charge une méthode `Dispose()`), **elle définit également la méthode `Close()` suivante, utilisée dans le même but** :

```cs
static void DisposeFileStream()
{
    FileStream fs = new FileStream("myFile.txt", FileMode.OpenOrCreate);

    // C'est pour le moins déroutant !
    // Ces appels de méthode font la même chose !
    fs.Close();
    fs.Dispose();
}
```

==Bien qu'il semble plus naturel de «fermer» un fichier plutôt que de le «supprimer», ce double emploi des méthodes de nettoyage peut prêter à confusion==. **Pour les rares types qui fournissent un alias, retenez simplement que si un type implémente `IDisposable`, appeler `Dispose()` est toujours une solution sûre**.

## Réutilisation du mot-clé `using` en C#

**Lorsque vous manipulez un objet managé implémentant `IDisposable`, il est fréquent d'utiliser une gestion structurée des exceptions pour garantir que la méthode `Dispose()` du type est appelée en cas d'exception d'exécution, comme ceci** :

```cs
using SimpleDispose;

Console.Title = "Fun with Dispose";
Console.WriteLine("**** Fun with Dispose ****\n");

MyResourceWrapper rw = new MyResourceWrapper();
try
{
    // Utilisation des membres de rw
}
finally
{
    // Appelle toujours Dispose(), erreur ou non!
    rw.Dispose();
}
```

Bien que ce soit un bon exemple de programmation défensive, la vérité est que peu de développeurs sont enthousiasmés par la perspective d'encapsuler chaque type jetable dans un bloc `try`/`finally` juste pour s'assurer que la méthode `Dispose()` soit appelée. **Pour obtenir le même résultat de manière beaucoup moins intrusive, C# prend en charge une syntaxe spéciale qui ressemble à ceci :**

```cs
// Dispose() est appelé automatiquement à la fin de la portée d'utilisation.
using (MyResourceWrapper rw = new MyResourceWrapper())
{
    // utilisation de l'objet rw
}
```

Si vous examiniez le code CIL suivant des instructions de niveau supérieur à l'aide d'*ildasm.exe*, ==vous constateriez que la syntaxe `using` se développe bien en une logique `try/finally`, avec l'appel attendu à `Dispose()`== :

```CIL
  .method private hidebysig static void  '<Main>$'(string[] args) cil managed
  {
	...
    .try
    {
	  ...
    }  // end .try
    finally
    {
	  ...
      IL_0024:  callvirt   instance void [System.Runtime]System.IDisposable::Dispose()
      ...
    }  // end handler
    ...
  } // end of method Program::'<Main>$'
```

>[!warning] Si vous essayer d'"utiliser" un objet qui n'implémente pas `IDisposable`, vous recevrez une erreur de compilation.

==Bien que cette syntaxe évite d'avoir à encapsuler manuellement les objets jetables dans une boucle `try`/`finally`==, **le mot-clé `using` en C# a malheureusement désormais une double signification** (importation d'espaces de noms et appel de la méthode `Dispose()`). **Néanmoins, lorsque vous travaillez avec des types qui implémentent l'interface `IDisposable`, cette construction syntaxique garantit que la méthode `Dispose()` de l'objet utilisé sera automatiquement appelée une fois le bloc `using` terminé**.

Sachez également qu'==il est possible de déclarer plusieurs objets *du même type* dans une portée `using`==. Comme prévu, le compilateur injectera du code pour appeler `Dispose()` sur chaque objet déclaré**.

```cs
// Utilise une liste séparée par des virgules
// pour déclarer plusieurs objets à supprimer.
using (
    MyResourceWrapper rw = new MyResourceWrapper(),
        rw2 = new MyResourceWrapper()
)
{
    // Utilisation des objets rw et rw2
}
```

>[!tip] Utiliser différents types dans un bloc `using` (Ancien style "propre")
>```cs
>using (MyResourceWrapper rw = new MyResourceWrapper())
>using (FileStream fs = new FileStream("test.txt", FileMode.Open))
>{
    >// Les deux sont accessibles ici
>}
>```

## Les déclarations `using` (Nouveauté C# 8.0)

**La nouveauté de C# 8.0 réside dans l'ajout des déclarations `using`**. Une déclaration `using` est ==une déclaration de variable précédée du mot-clé `using`==. Cela est fonctionnellement identique à la syntaxe abordée dans la question précédente, à l'exception du bloc de code explicite délimité par des accolades (`{}`).

Ajoutez la méthode suivante à votre classe :

```cs
static void UsingDeclaration()
{
    // Cette variable restera dans la portée jusqu'à la fin de la méthode.
    using var rw = new MyResourceWrapper();

    // Effectuer une action ici
    Console.WriteLine("About to dispose.");

    // La variable est maintenant supprimée.
}
```

Ensuite, ajoutez l'appel suivant aux instructions de niveau supérieur :

```cs
using SimpleDispose;

Console.Title = "Fun with Dispose";
Console.WriteLine("**** Fun with Dispose ****\n");

...

Console.WriteLine("Demonstrate using declarations");
UsingDeclaration();

Console.ReadLine();
```

Si vous examinez la nouvelle méthode avec *ildasm*, vous trouverez (comme vous pouvez vous y attendre) le même code qu'auparavant.

```CIL
.method assembly hidebysig static void 
        '<<Main>$>g__UsingDeclaration|0_1'() cil managed
{
  ...
  .try
  {
    ...
  }  // end .try
  finally
  {
    ...
    IL_0018:  callvirt   instance void [System.Runtime]System.IDisposable::Dispose()
    ...
  }  // end handler
  IL_001f:  ret
} // end of method Program::'<<Main>$>g__UsingDeclaration|0_1'
```

**Cette nouvelle fonctionnalité est en quelque sorte une astuce du compilateur, permettant d'économiser quelques frappes au clavier**. Soyez prudent lors de son utilisation, car la nouvelle syntaxe n'est pas aussi explicite que la précédente.

>[!tip] Comme cette déclaration est "rattachée" à la méthode ou elle se trouve, on peut effectuer des déclarations `using` avec des type différents:
>```cs
>void MaMethode()
>{
>	using var rw = new MyResourceWrapper();
>	using var fs = new FileStream("test.txt", FileMode.Create);
>	using var sw = new StreamWriter(fs);
>
>	// Utilisation des objets...
>	// Ils seront disposés dans l'ordre INVERSE de leur déclaration
>	// à la fin de l'accolade de la méthode.
>}
>```

# Création de types finalisables et jetables

Vous avez maintenant vu deux approches différentes pour construire une classe qui libère les ressources internes non managées. D'une part, vous pouvez utiliser un finaliseur. Cette technique vous offre la tranquillité d'esprit de savoir que l'objet se libère automatiquement lors du passage du ramasse-miettes (peu importe le moment) sans intervention de l'utilisateur. D'autre part, vous pouvez implémenter l'interface `IDisposable` pour permettre à l'utilisateur de l'objet de le libérer dès qu'il a terminé son utilisation. Cependant, si l'appelant oublie d'appeler `Dispose()`, les ressources non managées peuvent rester en mémoire indéfiniment.

Comme vous vous en doutez, **il est possible de combiner les deux techniques dans une seule définition de classe**. Ce faisant, ==vous bénéficiez des avantages des deux modèles==. Si l'utilisateur de l'objet se souvient d'appeler `Dispose()`, **vous pouvez indiquer au ramasse-miettes de contourner le processus de finalisation en appelant `GC.SuppressFinalize()`**. ==Si l'utilisateur de l'objet oublie d'appeler `Dispose()`, l'objet sera finalement finalisé et ses ressources internes seront libérées==. **La bonne nouvelle est que les ressources internes non gérées de l'objet seront libérées d'une manière ou d'une autre**.

Voici la nouvelle version de `MyResourceWrapper`, désormais finalisable et jetable, définie dans un projet d'application console C# nommé *FinalizableDisposableClass* :

```cs
namespace FinalizableDisposableClass;

// Un wrapper de ressources sophistiqué.
public class MyResourceWrapper : IDisposable
{
    // Le ramasse-miettes appellera cette méthode si
    // l'utilisateur de l'objet oublie d'appeler Dispose().
    ~MyResourceWrapper()
    {
        //Nettoye les ressources internes non gérées.
        // N'appelez **pas** Dispose() sur des objets gérés.
    }

    // L'utilisateur de l'objet appellera cette méthode
    // pour libérer les ressources dès que possible.
    public void Dispose()
    {
        // Nettoye ici les ressources non gérées.
        // Appelle Dispose() sur les autres objets jetables contenus.
        // Inutile de finaliser si l'utilisateur a appelé Dispose(),
        // donc supprimez la finalisation.
        GC.SuppressFinalize(this);
    }
}
```

**Notez que cette méthode `Dispose()` a été mise à jour pour appeler `GC.SuppressFinalize()`, ce qui indique au runtime qu'il n'est plus nécessaire d'appeler le destructeur lorsque cet objet est collecté par le ramasse-miettes, étant donné que les ressources non gérées ont déjà été libérées via la logique `Dispose()`**.

## Un modèle de suppression formalisé

L'implémentation actuelle de `MyResourceWrapper` fonctionne plutôt bien; cependant, ==elle présente quelques inconvénients mineurs==. Tout d'abord, **les méthodes `Finalize()` et `Dispose()` doivent nettoyer les mêmes ressources non managées**. ==Cela peut entraîner une duplication de code, ce qui peut rapidement devenir un cauchemar à maintenir==. Idéalement, il faudrait définir une fonction d'assistance privée appelée par l'une ou l'autre méthode.

**Ensuite, il faudrait s'assurer que la méthode `Finalize()` ne tente pas de supprimer d'objets managés, contrairement à la méthode `Dispose()`**. **Enfin, il faudrait également être certain que l'utilisateur de l'objet puisse appeler `Dispose()` plusieurs fois sans erreur**. ==Actuellement, la méthode `Dispose()` ne dispose d'aucune protection de ce type==.

**Pour résoudre ces problèmes de conception, Microsoft a défini un modèle de suppression formel et rigoureux qui offre un bon compromis entre robustesse, maintenabilité et performance**. Voici la version finale (annotée) de `MyResourceWrapper`, qui utilise ce modèle officiel :

```cs
namespace FinalizableDisposableClass;

// Un wrapper de ressources sophistiqué.
class MyResourceWrapper : IDisposable
{
    // Utilisé pour déterminer si Dispose() a déjà été appelé.
    private bool disposed = false;

    public void Dispose()
    {
        // Appelle notre méthode d'assistance.
        // Spécifier « true » indique que l'utilisateur de l'objet
        // a déclenché le nettoyage.
        CleanUp(true);
        // Supprime maintenant la finalisation.
        GC.SuppressFinalize(this);
    }

    private void CleanUp(bool disposing)
    {
        // Assurez-vous que nous n'avons pas déjà été éliminés !
        if (!this.disposed)
        {
            // Si la valeur de disposing est vraie, supprimez toutes les ressources gérées.
            if (disposing)
            {
                // Libére ici les ressources gérées.
            }
            // Nettoye ici les ressources non gérées.
        }
        disposed = true;
    }

    ~MyResourceWrapper()
    {
        // Appeler notre méthode auxiliaire.
        // Spécifier « false » indique que le GC a déclenché le nettoyage.
        CleanUp(false);
    }
}
```

**Notez que `MyResourceWrapper` définit désormais une méthode d'assistance privée nommée `CleanUp()`**. ==En spécifiant `true` comme argument, vous indiquez que l'utilisateur de l'objet a initié le nettoyage== ; **vous devez donc nettoyer toutes les ressources managées *et* non managées**. Cependant, ==lorsque le ramasse-miettes initie le nettoyage, vous spécifiez `false` lors de l'appel à `CleanUp()` afin de garantir que les objets jetables internes *ne soient pas supprimés*== (car vous ne pouvez pas supposer qu'ils soient encore en mémoire !). Enfin, **la variable membre booléenne (`disposed`) est initialisée à `true` avant de quitter `CleanUp()` afin de garantir que `Dispose()` puisse être appelée plusieurs fois sans erreur**.

>[!note] 
>Une fois qu'un objet a été « supprimé », le client peut toujours appeler ses membres, car il est toujours en mémoire. Par conséquent, une classe d'encapsulation de ressources robuste devrait également mettre à jour chaque membre de la classe avec une logique de code supplémentaire indiquant, en substance : « Si je suis supprimé, ne rien faire et quitter le membre. »

Pour tester la version finale de `MyResourceWrapper`, mettez à jour votre fichier *Program.cs* comme suit :

```cs
using FinalizableDisposableClass;

Console.Title = "Dispose() / Destructor Combo Platter";
Console.WriteLine("**** Dispose() / Destructor Combo Platter ****\n");

// Appelle Dispose() manuellement. Cela n'appellera pas le finaliseur.
MyResourceWrapper rw = new MyResourceWrapper();
rw.Dispose();

// N'appelle pas Dispose(). Cela déclenchera le finaliseur lorsque l'objet sera collecté par le GC.
MyResourceWrapper rw2 = new MyResourceWrapper();
```

**Notez que vous appelez explicitement `Dispose()` sur l'objet `rw`, ce qui empêche l'appel au destructeur. Cependant, vous avez "oublié" d'appeler `Dispose()` sur l'objet `rw2` ; pas d'inquiétude, le finaliseur s'exécutera tout de même lorsque l'objet sera collecté par le ramasse-miettes**.

>[!tip] Petite nuance moderne
>Toute cette section est complètement correcte, même avec des versions de .NET plus moderne.
>Toutefois, aujourd'hui, Microsoft conseille une alternative plus simple : `SafeHandle`
>```cs
>class MyResourceWrapper : IDisposable 
>{
>    // SafeHandle contient déjà toute la logique de Finalize + Dispose en interne
>    private SafeFileHandle _handle = ...; 
>
>    public void Dispose() => _handle.Dispose();
>}
>```

Ceci conclut votre étude de la manière dont l'environnement d'exécution gère vos objets via le ramasse-miettes. ==Bien qu'il existe d'autres détails (parfois un peu ésotériques) concernant le processus de collecte que je n'ai pas abordés ici (tels que les références faibles et la résurrection d'objets), vous êtes maintenant parfaitement préparé pour approfondir le sujet par vous-même==. Pour terminer ce chapitre, vous examinerez une fonctionnalité de programmation appelée *instanciation paresseuse* des objets.

# Comprendre l'instanciation paresseuse d'objets

**Lors de la création de classes, il peut arriver que vous ayez besoin de gérer une variable membre particulière dans votre code, alors qu'elle ne sera peut-être jamais utilisée, car l'utilisateur de l'objet n'appellera probablement jamais la méthode (ou la propriété) qui l'utilise**. C'est tout à fait compréhensible. Cependant, ==cela peut poser problème si la variable membre en question nécessite une grande quantité de mémoire pour être instanciée==.

Par exemple, supposons que vous écriviez une classe qui encapsule les opérations d'un lecteur de musique numérique. Outre les méthodes attendues, telles que `Play()`, `Pause()` et `Stop()`, vous souhaitez également permettre de retourner une collection d'objets `Song` (via une classe nommée `AllTracks`), représentant chaque fichier musical numérique présent sur l'appareil.

Pour pouvoir suivre, créez un nouveau projet d'application console nommé *LazyObjectInstantiation*, et définissez les types de classes suivants :

```cs
// Song.cs
namespace LazyObjectInstantiation;

// Représente une seule chanson.
class Song
{
    public string Artist { get; set; }
    public string TrackName { get; set; }
    public double TrackLength { get; set; }
}
```

```cs
//AllTracks.cs
namespace LazyObjectInstantiation;

// Représente toutes les chansons d'un lecteur.
class AllTracks
{
    // Notre lecteur multimédia peut contenir un maximum de
    // 10 000 chansons.

    private Song[] _allSongs = new Song[10000];

    public AllTracks()
    {
        // Supposons que nous remplissions le tableau
        // d'objets Song ici.

        Console.WriteLine("Filling up the songs!");
    }
}
```

```cs
//MediaPlayer.cs
namespace LazyObjectInstantiation;

// La classe MediaPlayer possède un objet AllTracks.
class MediaPlayer
{
    // Supposons que ces méthodes soient utiles.
    public void Play() { /* Lire un morceau */ }

    public void Pause() { /* Mettre le morceau en pause */ }

    public void Stop() { /* Arrêter la lecture */ }

    private AllTracks _allSongs = new AllTracks();

    public AllTracks GetAllTracks()
    {
        // Retourner tous les morceaux.
        return _allSongs;
    }
}
```

L'implémentation actuelle de `MediaPlayer` suppose que l'utilisateur souhaite obtenir une liste de morceaux via la méthode `GetAllTracks()`. **Que se passe-t-il si l'utilisateur n'a pas besoin de cette liste ? Dans l'implémentation actuelle, la variable membre `AllTracks` est tout de même allouée, créant ainsi $10\;000$ objets `Song` en mémoire, comme suit** :

```cs
using LazyObjectInstantiation;

Console.Title = "Fun with Lazy Instantiation";
Console.WriteLine("**** Fun with Lazy Instantiation ****\n");

// Cet appelant ne cherche pas à obtenir toutes les chansons,
// mais a indirectement créé 10 000 objets !
MediaPlayer myPlayer = new MediaPlayer();
myPlayer.Play();

Console.ReadLine();
```

**Il est clair que vous préféreriez éviter de créer 10 000 objets inutilisés, car cela surchargerait considérablement le ramasse-miettes de .NET Core**. Bien qu'il soit possible d'ajouter manuellement du code pour garantir que l'objet `_allSongs` ne soit créé que s'il est utilisé (par exemple, en utilisant le modèle de conception *Factory Method*), il existe une solution plus simple.

**Les bibliothèques de classes de base fournissent une classe générique utile nommée `Lazy<>`, définie dans l'espace de noms `System` de** *mscorlib.dll* (la version moderne étant *System.runtime.dll* ou `System.Private.CoreLib` dans [le code source .NET](https://source.dot.net/#System.Private.CoreLib/src/libraries/System.Private.CoreLib/src/System/Lazy.cs,8b99c1f377873554)). ==Cette classe vous permet de définir des données qui ne seront créées que si votre code les utilise==. **Comme il s'agit d'une classe générique, vous devez spécifier le type d'élément à créer lors de sa première utilisation**. ==Il peut s'agir de n'importe quel type des bibliothèques de classes de base .NET Core ou d'un type personnalisé que vous avez créé==. Pour activer l'instanciation paresseuse de la variable membre `AllTracks`, il vous suffit de modifier le code de `MediaPlayer` comme suit :

```cs
// La classe MediaPlayer possède un objet Lazy<AllTracks>.
class MediaPlayer
{
	...
	
    private Lazy<AllTracks> _allSongs = new Lazy<AllTracks>();

    public AllTracks GetAllTracks()
    {
        // Retourner tous les morceaux.
        return _allSongs.Value;
    }
}
```

**Outre le fait que la variable membre `AllTracks` est désormais représentée par un type `Lazy<>`, notez que l'implémentation de la méthode `GetAllTracks()` précédente a également été mise à jour**. Plus précisément, ==vous devez utiliser la propriété `Value`== (en lecture seule) ==de la classe `Lazy<>` pour obtenir les données stockées== (ici, l'objet `AllTracks` qui gère les $10\;000$ objets `Song`).

**Grâce à cette simple mise à jour, le code suivant allouera indirectement les objets `Song` uniquement si `GetAllTracks()` est effectivement appelée** :

```cs
using LazyObjectInstantiation;

Console.Title = "Fun with Lazy Instantiation";
Console.WriteLine("**** Fun with Lazy Instantiation ****\n");

// Aucune allocation d'objet AllTracks ici !
MediaPlayer myPlayer = new MediaPlayer();
myPlayer.Play();

// L'allocation de AllTracks a lieu lorsque vous appelez GetAllTracks().
MediaPlayer yourPlayer = new MediaPlayer();
AllTracks yourMusic = yourPlayer.GetAllTracks();

Console.ReadLine();
```

>[!note]
>L'instanciation paresseuse d'objets est utile non seulement pour réduire l'allocation d'objets inutiles, mais aussi pour réduire le nombre d'objets alloués. Cette technique peut également être utilisée lorsqu'un membre donné possède un code de création coûteux, comme l'appel d'une méthode distante, la communication avec une base de données relationnelle, etc.

## Personnalisation de la création des données paresseuses

Lorsque vous déclarez une variable `Lazy<>`, son type de données interne est créé à l'aide du constructeur par défaut, comme ceci :
```cs
// Le constructeur par défaut de AllTracks est appelé lorsque la variable Lazy<>
// est utilisée.
private Lazy<AllTracks> _allSongs = new Lazy<AllTracks>();
```

Bien que cela puisse convenir dans certains cas, **que se passe-t-il si la classe `AllTracks` possède des constructeurs supplémentaires et que vous souhaitez vous assurer que le bon est appelé ?** De plus, **que se passe-t-il si vous avez des opérations supplémentaires à effectuer (au-delà de la simple création de l'objet `AllTracks`) lors de la création de la variable Lazy<> ?** Heureusement, ==la classe `Lazy<>` vous permet de spécifier un délégué générique comme paramètre optionnel, qui définira une méthode à appeler lors de la création du type encapsulé==.

**Le délégué générique en question est de type `System.Func<>`, qui peut pointer vers une méthode retournant le même type de données que celui créé par la variable `Lazy<>` associée et peut accepter jusqu'à 16 arguments** (qui sont typés à l'aide de paramètres de type générique). ==Dans la plupart des cas, vous n'aurez pas besoin de spécifier de paramètres à passer à la méthode pointée par `Func<>`==. De plus, **pour simplifier considérablement l'utilisation du `Func<>` requis, je recommande d'utiliser une *expression lambda*** (voir le [[Chapitre 12#Comprendre les expressions lambda|Chapitre 12]] pour en savoir plus sur la relation délégué/lambda). 

Dans cette optique, voici la version finale de `MediaPlayer` qui ajoute un peu de code personnalisé lors de la création de l'objet `AllTracks` encapsulé. **N'oubliez pas que cette méthode doit retourner une nouvelle instance du type encapsulé par `Lazy<>` avant de se terminer, et vous pouvez utiliser le constructeur de votre choix** (ici, vous appelez toujours le constructeur par défaut de `AllTracks`).

```cs
class MediaPlayer
{
	...
    
    // Utilisation d'une expression lambda pour ajouter
    // du code supplémentaire lors de la création de l'objet AllTracks.
    private Lazy<AllTracks> _allSongs = new Lazy<AllTracks>(() =>
    {
        Console.WriteLine("Creating Alltrack object!");
        return new AllTracks();
    });

    public AllTracks GetAllTracks()
    {
        // Retourner tous les morceaux.
        return _allSongs.Value;
    }
```

Génial ! J'espère que vous percevez l'utilité de la classe `Lazy<>`. En résumé, **cette classe générique vous permet de garantir que les objets coûteux ne sont alloués que lorsque l'utilisateur en a besoin**.

>[!CAUTION] Quand NE PAS utiliser `Lazy<T>` :
>
>- Si l'objet a **90% de chances** d'être utilisé dès le départ.
>- Si l'objet est **léger** (quelques octets).
>- Si vous avez besoin d'un contrôle total sur le moment où l'exception de création pourrait survenir.
>- Si vous êtes dans une boucle ultra-rapide où le coût de vérification du `Lazy` (quelques nanosecondes) devient significatif.

# Résumé du chapitre

Ce chapitre avait pour but de démystifier le processus de ramasse-miettes. Comme vous l'avez vu, **le ramasse-miettes s'exécute uniquement lorsqu'il ne peut pas obtenir la mémoire nécessaire à partir du tas managé** (==ou lorsque le développeur appelle `GC.Collect()`==). **Lorsqu'un ramassage a lieu, vous pouvez être assuré que l'algorithme de ramassage de Microsoft est optimisé grâce à l'utilisation de générations d'objets, de threads secondaires pour la finalisation des objets et d'un tas managé dédié à l'hébergement des objets volumineux**.

Ce chapitre a également illustré **comment interagir par programmation avec le ramasse-miettes à l'aide de la classe `System.GC`**. Comme mentionné précédemment, ==vous n'aurez réellement besoin de le faire que lorsque vous créez des classes finalisables ou jetables qui opèrent sur des ressources non managées==.

Rappelons que **les types finalisables sont des classes qui fournissent un destructeur** (qui redéfinit la méthode `Finalize()`) **pour libérer les ressources non managées lors du ramassage des miettes**. **Les objets jetables, en revanche, sont des classes== (ou des structures non référencées) qui implémentent l'interface `IDisposable`, que l'utilisateur de l'objet doit appeler lorsqu'il a fini de l'utiliser**. Enfin, vous avez découvert **un modèle de « suppression » officiel qui combine les deux approches**.

Ce chapitre s'est conclu par **la présentation d'une classe générique nommée `Lazy<>`**. Comme vous l'avez vu, ==vous pouvez utiliser cette classe pour retarder la création d'un objet gourmand en mémoire jusqu'à ce que l'appelant en ait réellement besoin==. Ce faisant, **vous contribuez à réduire le nombre d'objets stockés sur le tas managé et vous vous assurez également que les objets gourmands en mémoire ne sont créés que lorsque l'appelant les requiert**.

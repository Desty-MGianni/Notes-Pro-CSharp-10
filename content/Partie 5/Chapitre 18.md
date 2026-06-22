---
title: "Chapitre 18: Comprendre le CIL et le Rôle des Assemblages Dynamiques"
publish: true
---

# <big><big><big><b><font color =green>Comprendre le CIL et le Rôle des Assemblages Dynamiques</font></b></big></big></big>

>[!warning]- Les point à étudier et à laisser dans le chapitre 
🟢 Ce qui reste INDISPENSABLE (La théorie du CIL)
>
>Toute la première partie sur la compréhension du **CIL (Common Intermediate Language)** et de la grammaire des opcodes reste 100 % valide et essentielle pour votre culture technique.
>
>- **Comprendre l'architecture binaire :** Apprendre comment vos structures C# sont traduites sous forme de pile d'évaluation (Stack-based architecture) est ce qui sépare les développeurs juniors des experts.
>- **Le débogage de bas niveau :** Savoir lire le CIL vous permet de comprendre exactement ce que fait le compilateur en arrière-plan (par exemple, comment il traduit un bloc `using` ou un enregistrement `record`).
>- **Vos outils :** C'est ici que votre maîtrise d'outils comme `ilspycmd` (ou votre configuration corrigée d'ildasm) prend tout son sens pour inspecter la réalité du binaire. [[1](https://www.reddit.com/r/csharp/comments/1cjbyii/is_this_book_too_old/)]
>
>🔴 Ce qui est OBSOLÈTE (Dynamic Assemblies & `Reflection.Emit`)
>
>La seconde moitié du chapitre traite de la génération de code à la volée dans la mémoire RAM via l'espace de noms `System.Reflection.Emit`. **C'est cette partie qui a perdu presque toute sa pertinence pratique en .NET 10.**
>
>- **Le conflit avec le Native AOT :** Générer un assembly dynamique dans la RAM requiert un moteur JIT (Just-In-Time) actif. À l'ère de .NET 10, où la priorité absolue est la compilation native en amont (**Native AOT**) et le **Trimming**, l'utilisation de `Reflection.Emit` classique brise complètement la chaîne de publication et lève des exceptions fatales au runtime.
>- **Remplacé par les Source Generators :** À l'époque de l'écriture du livre, pour créer du code dynamique rapide (comme le ferait un outil de mapping de base de données ou d'injection de dépendances), on utilisait `Reflection.Emit`. Aujourd'hui, 99 % de ces scénarios sont gérés au moment de la compilation par des **Source Generators (C# 9+)**, éliminant le besoin de manipuler du CIL manuellement à l'exécution.

**Lors du développement d'une application .NET complète, vous utiliserez très certainement C#** (ou un langage managé similaire tel que Visual Basic), **compte tenu de sa productivité et de sa facilité d'utilisation intrinsèques**. Cependant, comme vous l'avez appris au début de cet ouvrage, **==le rôle d'un compilateur managé est de traduire les fichiers de code *.cs* en code CIL, métadonnées de type et manifeste d'assembly==**. ==En réalité, CIL est un langage de programmation .NET à part entière, avec sa propre syntaxe, sa sémantique et son compilateur (*ilasm.exe*).==

**Dans ce chapitre, vous découvrirez le langage natif de .NET. Vous comprendrez la distinction entre une directive CIL, un attribut CIL et un opcode CIL**. **==Vous découvrirez ensuite le rôle de l'ingénierie aller-retour d'un assembly .NET Core et de divers outils de programmation CIL==**. ==Le reste du chapitre vous guidera à travers les bases de la définition des espaces de noms, des types et des membres à l'aide de la grammaire CIL==. Ce chapitre se conclura par une analyse du rôle de l'espace de noms `System.Reflection.Emit` et expliquera comment il est possible de construire un assembly (avec des instructions CIL) dynamiquement à l'exécution.

***==Bien sûr, rares sont les programmeurs qui auront besoin de manipuler du code CIL brut au quotidien==***. Par conséquent, ==je commencerai ce chapitre en examinant quelques raisons pour lesquelles il peut être utile de se familiariser avec la syntaxe et la sémantique de ce langage .NET de bas niveau.==

>[!danger] Il n'est pas nécessaire d'installer `ilasm` pour la suite de votre apprentissage 
>## Personne n'écrit de CIL à la main en production
>
>L'auteur vous montre `ilasm` pour vous prouver qu'on _peut_ techniquement se passer du C# et écrire directement en langage intermédiaire. En réalité, plus aucun développeur moderne ne fait cela. Si un framework a besoin de générer du CIL à la volée aujourd'hui, il utilise des API de bas niveau en mémoire, pas des fichiers texte compilés avec `ilasm`.
>
>## Les mêmes galères de compatibilité sur macOS
>
>Tout comme vous l'avez expérimenté avec `ildasm`, l'outil `ilasm` officiel est un vieux composant hérité du monde Windows. Le faire tourner de manière stable sur **macOS (Apple Silicon)** après chaque mise à jour du SDK .NET 10 va à nouveau vous demander de la maintenance de scripts et de `$PATH` pour un gain pédagogique quasi nul.

# Motivations pour l'apprentissage de la grammaire du CIL

**Lorsque vous créez un assembly .NET à l'aide du langage managé de votre choix** (C#, VB, F#, etc.), **le compilateur associé traduit votre code source en CIL**. Comme tout langage de programmation, ==le CIL fournit de nombreux jetons structurels et d'implémentation==. Étant donné que le CIL est un langage de programmation .NET comme un autre, il n'est pas surprenant qu'il soit possible de créer vos assemblies .NET directement en utilisant le CIL et le compilateur CIL (*ilasm.exe*).

>[!note]
>Comme indiqué au [[Chapitre 1#Exploration d'un assembly à l'aide de *ildasm.exe*|Chapitre 1]], ni *ildasm.exe* ni *ilasm.exe* ne sont fournis avec le .NET Runtime. Deux options s'offrent à vous pour obtenir ces outils. La première consiste à compiler le .NET Runtime à partir du code source disponible à l'adresse : https://github.com/dotnet/runtime. La seconde, plus simple, consiste à télécharger la version souhaitée depuis www.nuget.org. L'URL d'ILDasm sur NuGet est : https://www.nuget.org/packages/Microsoft.NETCore.ILDAsm/, et celle d'ILAsm.exe est : https://www.nuget.org/packages/Microsoft.NETCore.ILAsm/. Assurez-vous de sélectionner la version appropriée (pour ce livre, la version 6.0.0 ou supérieure est requise). Ajoutez les packages NuGet ILDasm et ILAsm à votre projet à l'aide de la commande suivante : 
>```bash
>dotnet add package Microsoft.NETCore.ILDAsm --version 10.0.0
>dotnet add package Microsoft.NETCore.ILAsm --version 10.0.0
>```

**Bien que peu de programmeurs (voire aucun !) choisissent de développer une application .NET complète directement avec CIL, l'étude du CIL reste un domaine d'apprentissage intellectuellement passionnant**. En clair, ***==plus vous maîtrisez la syntaxe du CIL, plus vous serez à même d'aborder le développement .NET avancé==***. Concrètement, une personne maîtrisant le CIL est capable de :

- Désassembler un assembly .NET existant, modifier le code CIL et recompiler le code source mis à jour en un binaire .NET modifié. Par exemple, il peut être nécessaire, dans certains cas, de modifier le CIL pour assurer l'interopérabilité avec certaines fonctionnalités COM avancées.

- Créer des assemblies dynamiques à l'aide de l'espace de noms `System.Reflection.Emit`. Cette API permet de générer un assembly .NET en mémoire, qui peut être, en option, enregistré sur disque. C'est une technique utile pour les développeurs d'outils qui ont besoin de générer des assemblies à la volée. 

- **Comprendre les aspects du Common Type System (CTS) qui ne sont pas pris en charge par les langages managés de haut niveau, mais qui existent au niveau du CIL**. En effet, ==le CIL est le seul langage .NET qui permet d'accéder à tous les aspects du CTS==. Par exemple, en utilisant du CIL pur, vous pouvez définir des membres et des champs globaux (ce qui est impossible en C#).

**Encore une fois, pour être parfaitement clair, si vous choisissez de ne pas vous préoccuper des détails du code CIL, vous pouvez tout de même maîtriser le C# et les bibliothèques de classes de base .NET**. À bien des égards, la connaissance du CIL est analogue à la compréhension du langage assembleur par un programmeur C (et C++). **==Ceux qui connaissent les rudiments du « matériel » de bas niveau peuvent créer des solutions assez avancées pour la tâche à accomplir et acquérir une compréhension plus approfondie de l'environnement de programmation (et d'exécution) sous-jacent==**. Alors, si vous êtes prêt à relever le défi, commençons par examiner les détails du CIL.

>[!note]
>Ce chapitre ne prétend pas être un exposé exhaustif de la syntaxe et de la sémantique du CIL. Nous n'en effleurons que la surface. Si vous souhaitez (ou devez) approfondir vos connaissances en CIL, veuillez consulter la documentation complète accessible via le site [Microsoft](https://learn.microsoft.com/en-us/dotnet/fundamentals/standards) ou [Wikipédia](https://en.wikipedia.org/wiki/List_of_CIL_instructions) Pour la liste des instructions.

# Examen des directives, attributs et opcodes CIL

==Lorsque vous commencez à explorer les langages de bas niveau comme le CIL, vous découvrirez forcément des noms nouveaux== (et souvent intimidants) ==pour des concepts familiers==. Par exemple, les chapitres précédents ont présenté des exemples CIL contenant les éléments suivants :

```
{new, public, this, base, get, set, explicit, unsafe, enum, operator, partial}
```

**Après avoir lu les chapitres précédents, vous savez qu’il s’agit de mots-clés du langage C#.** Cependant, ***==en examinant de plus près les éléments de cet ensemble, vous constaterez peut-être que si chaque élément est bien un mot-clé C#, sa sémantique est radicalement différente.==*** ==Par exemple, le mot-clé `enum` définit un type dérivé de `System.Enum`, tandis que les mots-clés `this` et `base` permettent de référencer respectivement l’objet courant et la classe parente de l’objet. Le mot-clé `unsafe` sert à définir un bloc de code qui ne peut pas être directement surveillé par le CLR, tandis que le mot-clé `operator` permet de créer une méthode cachée== (nommée spécialement) ==qui sera appelée lors de l'application d'un opérateur C# spécifique== (tel que le signe plus).

**Contrairement à un langage de plus haut niveau comme C#, CIL ne se contente pas de définir un ensemble général de mots-clés. L'ensemble des jetons compris par le compilateur CIL est plutôt subdivisé en trois grandes catégories sémantiques.**

- Directives CIL
- Attributs CIL
- Codes d'opération CIL (opcodes)

**Chaque catégorie de jeton CIL est exprimée à l'aide d'une syntaxe particulière, et les jetons sont combinés pour construire un assembly .NET valide.**

## Le rôle des directives CIL

**Tout d'abord, il existe un ensemble de jetons CIL bien connus utilisés pour décrire la structure globale d'un assembly .NET**. Ces jetons sont appelés *directives*. ***==Les directives CIL servent à indiquer au compilateur CIL comment définir les espaces de noms, les types et les membres qui composeront un assembly.==***

**==Les directives sont représentées syntaxiquement à l'aide d'un point (`.`) en préfixe==** (par exemple : `.namespace`, `.class`, `.publickeytoken`, `.method`, `.assembly`, etc.). **Ainsi, si votre fichier** *.il* (l'extension conventionnelle pour un fichier contenant du code CIL) **comporte une directive `.namespace` et trois directives `.class`, le compilateur CIL générera un assembly définissant un espace de noms .NET Core unique contenant trois types de classes .NET. **

## Rôle des attributs CIL

*==Dans de nombreux cas, les directives CIL ne suffisent pas à elles seules pour exprimer pleinement la définition d'un type .NET ou d'un membre de type donné==*. C'est pourquoi **de nombreuses directives CIL peuvent être précisées à l'aide de divers attributs CIL afin de qualifier leur traitement**. Par exemple, **==la directive `.class` peut être annotée avec l'attribut `public`==** (pour définir la visibilité du type), **==l'attribut `extends`==** (pour spécifier explicitement la classe de base du type) **==et l'attribut `implements`==** (pour lister l'ensemble des interfaces prises en charge par le type).

>[!note]
>Ne confondez pas un attribut .NET (voir [[Chapitre 17#Comprendre le rôle des attributs .NET|Chapitre 17]]) avec un attribut CIL, qui sont deux choses très différentes.

## Le rôle des opcodes CIL

**Une fois qu'un assembly .NET, un espace de noms et un jeu de types ont été définis en CIL à l'aide de diverses directives, et attributs associés, la dernière étape consiste à fournir la logique d'implémentation du type**. ==C'est le rôle des *codes d'opération*, ou *opcodes*==. ==Dans la tradition des autres langages de bas niveau, de nombreux opcodes CIL ont tendance à être cryptiques et totalement imprononçables pour le commun des mortels.== Par exemple, pour charger une variable de type chaîne de caractères en mémoire, on n'utilise pas l'opcode convivial `LoadString`, mais plutôt `ldstr`.

**==Cependant, il faut reconnaître que certains opcodes CIL correspondent assez naturellement à leurs équivalents C# (par exemple, `box`, `unbox`, `throw` et `sizeof`==**)). **Comme vous le verrez, les opcodes CIL sont toujours utilisés dans la portée de l'implémentation d'un membre, et contrairement aux directives CIL, ils ne sont jamais précédés d'un point.**

## Distinction entre les opcodes et les mnémoniques CIL

Comme expliqué précédemment, **les opcodes tels que `ldstr` servent à implémenter les membres d'un type donné**. Cependant, les jetons tels que `ldstr` sont des *mnémoniques CIL* correspondant aux *opcodes binaires CIL*. Pour clarifier cette distinction, supposons que vous ayez écrit la méthode suivante en C# dans une application console .NET nommée *FirstSamples* :

```cs
int Add(int x, int y)
{
    return x + y;
}
```

==L'addition de deux nombres est exprimée par l'opcode CIL $0X58$. De même, la soustraction de deux nombres est exprimée par l'opcode $0X59$, et l'allocation d'un nouvel objet sur le tas managé est réalisée par l'opcode $0X73$==. De ce fait, **il faut comprendre que le « code CIL » traité par un compilateur JIT n'est rien d'autre que des blocs de données binaires.**

**Heureusement, à chaque opcode binaire du CIL correspond un mnémonique**. Par exemple, on peut utiliser `add` au lieu de $0X58$, `sub` au lieu de $0X59$ et `newobj` au lieu de $0X73$. **Compte tenu de cette distinction opcode/mnémonique, les décompilateurs CIL tels que *ildasm.exe* traduisent les opcodes binaires d'un assembly en leurs mnémoniques CIL correspondants**. Par exemple, voici le code CIL présenté par *ildasm.exe* pour la méthode `Add()` de C# précédente (==votre résultat exact peut varier selon votre version de .NET Core==) :

```cil
.method assembly hidebysig static int32
          '<<Main>$>g__Add|0_0'(int32 x,
                                int32 y) cil managed
  {
    .custom instance void [System.Runtime]System.Runtime.CompilerServices.CompilerGeneratedAttribute::.ctor() = ( 01 00 00 00 )
    // Code size       9 (0x9)
    .maxstack  2
    .locals init (int32 V_0)
    IL_0000:  nop
    IL_0001:  ldarg.0
    IL_0002:  ldarg.1
    IL_0003:  add
    IL_0004:  stloc.0
    IL_0005:  br.s       IL_0007

    IL_0007:  ldloc.0
    IL_0008:  ret
  } // end of method Program::'<<Main>$>g__Add|0_0'
```

À moins de développer un logiciel .NET de très bas niveau (comme un compilateur managé personnalisé), vous n'aurez jamais à vous soucier des opcodes binaires numériques littéraux du CIL. En pratique, lorsque les programmeurs .NET parlent d'« opcodes CIL », ils font référence à l'ensemble des mnémoniques de chaînes de caractères conviviales (comme je l'ai fait dans ce texte et comme je le ferai pour le reste de ce chapitre) plutôt qu'aux valeurs numériques sous-jacentes.

## Empilement et dépilement : la nature basée sur la pile du CIL

**Les langages .NET de plus haut niveau** (comme C#) **tentent de masquer autant que possible les aspects complexes du CIL de bas niveau**. **==Un aspect du développement .NET particulièrement bien caché est que le CIL est un langage de programmation basé sur la pile==**. Rappelez-vous, d'après l'examen des espaces de noms de collections (voir [[Chapitre 10#Utilisation de la classe `Stack<T>`|Chapitre 10]]), que **la classe `Stack<T>` peut être utilisée pour empiler une valeur sur une pile ainsi que pour dépiler la valeur située au sommet de la pile**. Bien sûr, **==les développeurs CIL n'utilisent pas un objet de type `Stack<T>` pour charger et décharger les valeurs à évaluer ; cependant, le principe d'empilement et de dépilement reste le même.==**

**Formellement, l'entité utilisée pour stocker un ensemble de valeurs à évaluer est appelée la** *pile d'exécution virtuelle*. Comme vous le verrez, **==CIL fournit plusieurs opcodes permettant d'empiler une valeur==**; ==ce processus est appelé *chargement*==. De même, **CIL définit des opcodes supplémentaires permettant de transférer la valeur située au sommet de la pile en mémoire** (par exemple, dans une variable locale) **grâce à un processus appelé** *stockage*.

*==En CIL, il est impossible d'accéder directement à une donnée, qu'il s'agisse de variables locales, d'arguments de méthode ou de données de champ==*. **Il est nécessaire de charger explicitement l'élément sur la pile, puis de le dépiler pour une utilisation ultérieure** (***==gardez ce point à l'esprit, car il explique pourquoi un bloc de code CIL donné peut paraître redondant==***).

>[!note]
>Rappelons que le CIL n'est pas exécuté directement, mais compilé à la demande. Lors de la compilation du code CIL, de nombreuses redondances d'implémentation sont optimisées. De plus, si vous activez l'option d'optimisation du code pour votre projet actuel (via l'onglet Générer de la fenêtre Propriétés du projet de Visual Studio ou en ajoutant `<Optimize>true</Optimize>` au groupe de propriétés principal du fichier projet), le compilateur supprimera également
diverses redondances du CIL.

==Pour comprendre comment CIL exploite un modèle de traitement basé sur la pile, prenons l'exemple d'une méthode C# simple, `PrintMessage()`, qui ne prend aucun argument et ne renvoie aucune valeur==. Dans l'implémentation de cette méthode, vous afficherez simplement la valeur d'une variable de type chaîne locale sur le flux de sortie standard, comme ceci :

```cs
void PrintMessage()
{
    string myMessage = "Hello.";
    Console.WriteLine(myMessage);
}
```

**La valeur** (à l'index 0) **est ensuite chargée en mémoire à l'aide de l'opcode `ldloc.0`** ("==charger l'argument local à l'index 0==") **pour être utilisée par l'appel de la méthode `System.Console.WriteLine()` (spécifié par l'opcode `call`).** Enfin, la fonction retourne via l'opcode `ret`. **Voici le code CIL (annoté) de la méthode `PrintMessage()`** (==notez que les opcodes `nop` ont été supprimés par souci de concision==) :

```cil
  .method assembly hidebysig static void 
          '<<Main>$>g__PrintMessage|0_1'() cil managed
  {
    .custom instance void [System.Runtime]System.Runtime.CompilerServices.CompilerGeneratedAttribute::.ctor() = ( 01 00 00 00 ) 
    // Code size       15 (0xf)
    .maxstack  1
// Défini une variable locale de type string (à l'index 0)
    .locals init (string V_0)
// Charge un string sur la pile avec la valeur "Hello."
    IL_0001:  ldstr      "Hello."
// Stocke la valeur string sur la pile dans la variable locale.
    IL_0006:  stloc.0
// Charge la valeur à l'index 0.
    IL_0007:  ldloc.0
// Appelle la méthode avec la valeur actuelle.
    IL_0008:  call       void [System.Console]System.Console::WriteLine(string)
    IL_000e:  ret
  } // end of method Program::'<<Main>$>g__PrintMessage|0_1'

} // end of class Program
```

>[!note]
>Comme vous pouvez le constater, CIL prend en charge les commentaires de code utilisant la syntaxe à double barre oblique (ainsi que la syntaxe `/*...*/`). Comme en C#, les commentaires de code sont complètement ignorés par le compilateur CIL.

Maintenant que vous maîtrisez les bases des directives, attributs et opcodes CIL, voyons une application pratique de la programmation CIL, en commençant par le sujet de l'ingénierie aller-retour.

>[!tldr] 
>À son niveau le plus fondamental, la machine virtuelle de .NET (appelée le _VES - Virtual Execution System_) est ce qu'on appelle une **architecture basée sur la pile (Stack-based architecture)**.
>
> ## Les deux piles du CIL
>
>Le livre va faire une distinction cruciale entre deux notions de "pile" :
>
>1. **La Call Stack (Pile d'appels) :** Elle gère la structure de votre programme (quelle méthode a appelé quelle méthode, et où retourner quand une méthode se termine). C'est le standard de presque tous les langages.
>2. **L'Evaluation Stack (Pile d'évaluation) :** C'est **le cœur du CIL**. Contrairement au processeur de votre Mac (ARM64) qui utilise des _Registres_ (`X0`, `X1`...) pour faire des calculs, le processeur virtuel .NET utilise cette pile pour exécuter la moindre ligne de code.
>
> ## Le flux universel du CIL : Charger / Traiter / Stocker
>
>Chaque langage .NET (C#, VB.NET, F#) traduit ses opérations selon un rituel immuable en CIL :
>
>3. **`ld` (Load) :** On empile les données (on les pousse sur l'Evaluation Stack).
>4. **L'Opération :** On appelle une instruction (ex: `add` ou `call`). Cette instruction **consomme** automatiquement les éléments situés au sommet de la pile et y dépose le résultat .
>5. **`st` (Store) :** On dépile le résultat pour le stocker dans une variable locale ou un champ.
>
>## Pour aller plus loin
>
>Oui, tous les langages .NET ne sont au final que des générateurs d'instructions pour manipuler cette pile d'évaluation. L'avantage pour Microsoft est immense : peu importe que le langage d'origine soit orienté objet (C#) ou fonctionnel (F#), tout s'égalise sous forme d'instructions de pile standardisées (ECMA-335).
>
C'est pour cela que votre inspecteur de types ou votre décompilateur `ilspycmd` n'a aucun mal à lire une DLL, car le CIL a effacé toutes les différences syntaxiques des langages d'origine.

# Comprendre l'ingénierie aller-retour

**Vous savez comment utiliser *ildasm.exe* pour visualiser le code CIL généré par le compilateur C#** (voir [[Chapitre 1#Exploration d'un assembly à l'aide de *ildasm.exe*|Chapitre 1]]). **==Une fois le code CIL à votre disposition, vous pouvez le modifier et le recompiler à l'aide du compilateur CIL==**, *ilasm.exe*. ==Formellement, cette technique est appelée *ingénierie aller-retour* et peut s'avérer utile dans certains cas, par exemple :

- Vous devez modifier un assembly dont vous ne possédez plus le code source.
-  Vous utilisez un compilateur .NET imparfait qui a généré un code CIL inefficace (voire incorrect) et vous souhaitez modifier le code source.
- Vous développez une bibliothèque d'interopérabilité COM et souhaitez prendre en compte certains attributs IDL COM perdus lors de la conversion (comme l'attribut COM `[helpstring]`).

Pour illustrer le processus d'aller-retour, commencez par créer une nouvelle application console C# .NET Core nommée *RoundTrip*. Mettez à jour le fichier *Program.cs* comme suit :

```cs
Console.WriteLine("Hello CIL code");
Console.ReadLine();
```

Compilez votre programme à l'aide de l'interface de ligne de commande .NET Core (`dotnet build`).

>[!note]
>Rappelons-nous du [[Chapitre 1#Aperçu des assembly .NET|Chapitre 1]] que tous les assemblys .NET Core (bibliothèques de classes ou applications console) sont par défaut compilés en assemblys ayant l'extension *.dll* et exécutés à l'aide de *dotnet.exe*. Nouveauté de .NET Core 3.0 (et versions ultérieures) : le fichier *dotnet.exe* est copié dans le répertoire de sortie et renommé pour correspondre au nom de l'assembly. Ainsi, ==même si votre projet semble avoir été compilé en *RoundTrip.exe*, il a en réalité été compilé en *RoundTrip.dll*, avec *dotnet.exe* copié dans *RoundTrip.exe* ainsi que les arguments de ligne de commande nécessaires à son exécution==. Si vous publiez votre projet en un seul fichier (voir [[Chapitre 16#Publication d'applications autonomes dans un seul fichier|Chapitre 16]]), *RoundTrip.exe* contiendra alors davantage que votre code. 

Ensuite, exécutez *ildasm.exe* sur *RoundTrip.dll* en utilisant la commande suivante (exécutée depuis le dossier de la solution) :

```bash
ildasm -all -metadata -out=RoundTrip/RoundTrip.il RoundTrip/bin/Debug/net10.0/RoundTrip.dll
```

**La commande précédente a affiché la quasi-totalité du contenu de l'assembly, y compris les en-têtes de fichier, les commandes hexadécimales en tant que commentaires, toutes les métadonnées et bien plus encore**. Si vous souhaitez un fichier plus concis pour l'analyse du code IL, vous pouvez supprimer les options `-all` et `-METADATA`. **Cependant, pour ces exemples, vous avez besoin de toutes ces informations supplémentaires.**

>[!note] Note pour les utilisateur Windows
>*ildasm.exe* générera également un fichier *.res* lors de l'exportation du contenu d'un assembly vers un fichier. Ces fichiers de ressources peuvent être ignorés (et supprimés) tout au long de ce chapitre, car vous ne les utiliserez pas. Ce fichier contient notamment des informations de sécurité CLR de bas niveau.

Vous pouvez maintenant consulter *RoundTrip.il* avec Visual Studio, Visual Studio Code ou l'éditeur de texte de votre choix. **Remarquez d'abord que le fichier *.il* s'ouvre en déclarant chaque assembly externe référencé par rapport auquel l'assembly courant est compilé**. ==Si votre bibliothèque de classes utilise des types supplémentaires dans d'autres assemblies référencés== (outre `System.Runtime` et `System.Console`), ==vous trouverez des directives `.assembly extern` supplémentaires.==

```cil
.assembly extern System.Runtime
{
  .publickeytoken = (B0 3F 5F 7F 11 D5 0A 3A )                         // .?_....:
  .ver 10:0:0:0
}
.assembly extern System.Console
{
  .publickeytoken = (B0 3F 5F 7F 11 D5 0A 3A )                         // .?_....:
  .ver 10:0:0:0
}
```

Ensuite, vous trouverez la définition formelle de votre assembly *RoundTrip.dll*, décrite à l'aide de diverses directives CIL (telles que `.module`, `.imagebase`, etc.).

```cil
.assembly RoundTrip
{

...
	
  .hash algorithm 0x00008004
  .ver 1:0:0:0
}
.module RoundTrip.dll
// MVID: {5bd4d8b1-4acb-4618-860c-2335f2027498}
.custom instance void [System.Runtime]System.Runtime.CompilerServices.RefSafetyRulesAttribute::.ctor(int32) = ( 01 00 0B 00 00 00 00 00 ) 
.imagebase 0x00400000
.file alignment 0x00000200
.stackreserve 0x00100000
.subsystem 0x0003       // WINDOWS_CUI
.corflags 0x00000001    //  ILONLY

```

**Après avoir documenté les assemblies référencés en externe et défini l'assembly courant, vous trouverez une définition du type `Program`, créée à partir des instructions de niveau supérieur**. ==Notez que la directive `.class` possède divers attributs== (dont beaucoup sont facultatifs), ==tels que `extends`, illustré ici, qui indique la classe de base du type :

```cil
.class private auto ansi beforefieldinit Program
       extends [System.Runtime]System.Object
{ }
```

**L'essentiel du code CIL représente l'implémentation du constructeur par défaut de la classe et de la méthode `Main()` générée automatiquement, toutes deux définies (en partie) avec la directive `.method`**. **==Une fois les membres définis à l'aide des directives et attributs appropriés, ils sont implémentés à l'aide de divers opcodes.==**

```cil
.method private hidebysig static void  '<Main>$'(string[] args) cil managed
{
  .entrypoint
  // Code size       18 (0x12)
  .maxstack  8
  IL_0000:  ldstr      "Hello CIL code"
  IL_0005:  call       void [System.Console]System.Console::WriteLine(string)
  IL_000a:  nop
  IL_000b:  call       string [System.Console]System.Console::ReadLine()
  IL_0010:  pop
  IL_0011:  ret
} // end of method Program::'<Main>$'
```

**Il est essentiel de comprendre que lors de l'interaction avec des types .NET Core** (tels que `System.Console`) **en CIL, vous devrez *toujours* utiliser le nom complet du type**. De plus, **==ce nom complet doit toujours être préfixé par le nom convivial de l'assembly qui le définit (entre crochets), comme dans les deux lignes suivantes de la méthode `Main()` générée :==**

```cil
IL_0005: call void [System.Console]System.Console::WriteLine(string)
IL_000b: call string [System.Console]System.Console::ReadLine()
```

## Le rôle des étiquettes de code CIL

Vous avez certainement remarqué que **==chaque ligne de code d'implémentation est préfixée par un jeton de la forme `IL_XXX:`==** (par exemple, `IL_0000:`, `IL_0001:`, etc.). **Ces jetons sont appelés étiquettes de code et peuvent être nommés comme vous le souhaitez** (*==à condition qu'ils ne soient pas dupliqués dans la même portée de membre==*). **Lorsque vous exportez un assembly vers un fichier à l'aide** d'*ildasm.exe*, **celui-ci génère automatiquement des étiquettes de code suivant la convention de nommage `IL_XXX:`**. Vous pouvez toutefois les modifier pour utiliser un marqueur plus descriptif. Voici un exemple :

```cil
.method private hidebysig static void Main(string[] args) cil managed
{
	.entrypoint
	.maxstack 8
	Load_String: ldstr "Hello CIL code!"
	PrintToConsole: call void [System.Console]System.Console::WriteLine(string)
	Nothing_2: nop
	WaitFor_KeyPress: call string [System.Console]System.Console::ReadLine()
	RemoveValueFromStack: pop
	Leave_Function: ret
}
```

***==En réalité, la plupart des étiquettes de code sont facultatives==***. *==Elles ne sont obligatoires que lors de la création de code CIL utilisant des structures de branchement ou de boucle, car elles permettent de spécifier le flux d'exécution==*. **Dans cet exemple, vous pouvez supprimer ces étiquettes générées automatiquement sans problème**, comme ceci :

```cil
.method private hidebysig static void Main(string[] args) cil managed
{
	.entrypoint
	.maxstack 8
	ldstr "Hello CIL code!"
	call void [System.Console]System.Console::WriteLine(string)
	nop
	call string [System.Console]System.Console::ReadLine()
	pop
	ret
}
```

## Interaction avec CIL : Modification d’un fichier *.il*

Maintenant que vous comprenez mieux la structure d’un fichier CIL de base, ==terminons l’exercice d’aller-retour==. L’objectif est simple: **modifier le message affiché dans la console**. **==Vous pouvez faire plus, comme ajouter des références d’assembly ou créer de nouvelles classes et méthodes, mais nous allons rester simples.==**

**Pour effectuer la modification, vous devez altérer l’implémentation actuelle des instructions de niveau supérieur, créées par la méthode `<Main>$`.** Localisez cette méthode dans le fichier *.il* et remplacez le message par `"Hello from altered CIL code!`.

Vous venez ainsi de mettre à jour le code CIL pour qu’il corresponde à la définition de classe C# suivante :

```cs
static void Main(string[] args)
{ 
	Console.WriteLine("Bonjour depuis le code CIL modifié !");
	Console.ReadLine();
}
```

**Il existe deux façons de créer des assemblies .NET compilés à partir d'un fichier** *.il*. ==L'utilisation du type de projet IL offre plus de flexibilité, mais est un peu plus complexe==. **La seconde méthode utilise simplement** *ILAsm.exe* **pour créer un fichier** *.dll* **à partir du fichier IL**. Nous allons commencer par explorer l'utilisation d'*ILAsm.exe*.

## Compiler du code CIL avec *ILAsm.exe*

**Commencez par créer un nouveau répertoire sur votre machine** (dans les exemples sur GitHub, je l'ai nommé `RoundTrip2`). ==Dans ce répertoire, copiez le fichier *RoundTrip.il* mis à jour==. **Copiez également le fichier** *RoundTrip.runtimeconfig.json* **situé dans le dossier `RoundTrip\bin\Debug\.net10.0`**. ==Ce fichier est nécessaire aux exécutables créés avec *ILAsm.exe* pour configurer le nom et le framework cible==. À titre de référence, le contenu du fichier est indiqué ici :

```json
{
  "runtimeOptions": {
    "tfm": "net10.0",
    "framework": {
      "name": "Microsoft.NETCore.App",
      "version": "10.0.0"
    },
    "configProperties": {
      "System.Runtime.Serialization.EnableUnsafeBinaryFormatterSerialization": false
    }
  }
}
```

Enfin, compilez l'assembly avec la commande suivante depuis le répertoire `RoundTrip2` (mettez à jour le chemin vers *ILAsm.exe* si nécessaire) :

```bash
ilasm -dll RoundTrip2/RoundTrip.il -arm64
```

Pour exécuter le programme, utilisez l'interface de ligne de commande (CLI), comme ceci :

```bash
dotnet RoundTrip.dll
```

Effectivement, le message mis à jour s'affichera dans la fenêtre de la console.

## Compilation de code CIL avec les projets `Microsoft.NET.Sdk.il`

==Comme vous venez de le constater, la compilation de code IL avec *ILAsm.exe* présente certaines limitations==. **Une méthode bien plus performante consiste à créer un projet utilisant le type de projet `Microsoft.NET.Sdk.IL`. Malheureusement, à l'heure où nous écrivons ces lignes, ce type de projet n'est pas inclus dans les modèles de projet standard ; une intervention manuelle est donc nécessaire**. Commencez par créer un nouveau répertoire nommé `RoundTrip3` et copiez-y le fichier *RoundTrip.il* modifié.

>[!note]
>Au moment de la rédaction de ce document, Visual Studio prend directement en charge le type de projet *.ilproj*. Bien qu'il existe des extensions sur le Marketplace, je ne peux ni recommander ni déconseiller leur utilisation. Visual Studio Code prend en charge l'intégralité du code présenté dans cette section.

>[!success] Le type de projet `Microsoft.NET.Sdk.IL` est devenu la méthode moderne officielle pour compiler du code CIL pur, remplaçant totalement les appels directs et instables à `ilasm.exe`.
>### Le package est désormais disponible sur le flux officiel NuGet
>
>- **À l'époque du livre :** Il fallait configurer un flux de développement "Myget" complexe de Microsoft pour récupérer l'extension Microsoft.NET.Sdk.IL.
>- **Aujourd'hui (.NET 10) :** L'extension fait partie du catalogue standard [NuGet](https://www.nuget.org/packages/Microsoft.NET.Sdk.IL). Vous pouvez l'utiliser directement sans configuration de flux externe.
>
>### Le fichier projet utilise l'extension `.ilproj`
>
>La CLI `dotnet` gère ce projet comme un projet C# ou F# classique. Au lieu d'un fichier `.csproj`, on utilise l'extension **`.ilproj`**

Dans ce répertoire, créez un fichier *global.json*. Ce fichier s'applique au répertoire courant et à tous ses sous-répertoires. **Il sert à définir la version du SDK à utiliser lors de l'exécution des commandes .NET Core**. Mettez à jour les fichiers comme suit :

```json
{
	"msbuild-sdks": {
		"Microsoft.NET.Sdk.IL": "10.0.8"
	}
}
```

L'étape suivante consiste à créer le fichier de projet. Créez un fichier nommé *RoundTrip.ilproj* et mettez-le à jour comme suit :

```xml
<Project Sdk="Microsoft.NET.Sdk.IL">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net10.0</TargetFramework>
    <MicrosoftNetCoreIlasmPackageVersion>10.0.7</MicrosoftNetCoreIlasmPackageVersion>
    <ProduceReferenceAssembly>false</ProduceReferenceAssembly>
  </PropertyGroup>
</Project>
```

Enfin, copiez votre fichier *RoundTrip.il* mis à jour dans le répertoire. Compilez l'assembly à l'aide de l'interface de ligne de commande .NET Core :

```bash
dotnet build
```

**Vous trouverez les fichiers résultants dans le dossier habituel `bin\debug\net10.0`**. Vous pouvez alors exécuter votre nouvelle application en lançant *RoundTrip.exe*, comme si vous l'aviez créée à partir d'un modèle d'application console C# standard.

**Outre une expérience utilisateur améliorée, le projet IL peut tirer parti de la génération d'assemblies mono-fichiers**, comme expliqué au [[Chapitre 16#Publication d'applications autonomes dans un seul fichier|Chapitre 16]]. Mettez à jour le fichier projet comme suit :

```xml
<Project Sdk="Microsoft.NET.Sdk.IL/10.0.7">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net10.0</TargetFramework>
    <MicrosoftNetCoreIlasmPackageVersion>10.0.7</MicrosoftNetCoreIlasmPackageVersion>
    <ProduceReferenceAssembly>false</ProduceReferenceAssembly>
    <RuntimeIdentifier>osx-arm64</RuntimeIdentifier>
    <SelfContained>true</SelfContained>
    <PublishSingleFile>true</PublishSingleFile>
    <PublishReadyToRun>true</PublishReadyToRun>
    <PublishTrimmed>true</PublishTrimmed>
  </PropertyGroup>
</Project>

```

**Vous pouvez désormais publier votre projet comme un fichier autonome, à l'instar d'un projet C#.** Utilisez la commande de publication pour voir cela en pratique :

```bash
dotnet publish -o singlefile
```

**Bien que le résultat de cet exemple simple ne soit pas particulièrement spectaculaire, il illustre une utilisation pratique de la programmation en CIL : les allers-retours.**

# Comprendre les directives et attributs CIL

Maintenant que vous avez vu comment convertir des assemblies .NET Core en IL et compiler du code IL en assemblies, **vous pouvez maintenant vous intéresser à la syntaxe et à la sémantique du CIL**. **==Les sections suivantes vous guideront dans la création d'un espace de noms personnalisé contenant un ensemble de types==**. Cependant, ==pour simplifier, ces types ne contiendront pas encore de logique d'implémentation pour leurs membres==. **Une fois que vous aurez compris comment créer des types vides, vous pourrez vous concentrer sur la définition de membres « réels » à l'aide des opcodes CIL.**

## Spécification des assemblies référencés en externe dans CIL

Dans un nouveau répertoire nommé `CILTypes`, copiez le fichier *global.json* de l'exemple précédent. Créez un nouveau fichier de projet nommé *CILTypes.ilproj* et mettez-le à jour comme suit :

```xml
<Project Sdk="Microsoft.NET.Sdk.Il">
  <PropertyGroup>
    <TargetFramework>net10.0</TargetFramework>
    <MicrosoftNetCoreIlasmPackageVersion>10.0.7</MicrosoftNetCoreIlasmPackageVersion>
    <ProduceReferenceAssembly>false</ProduceReferenceAssembly>
  </PropertyGroup>
</Project>
```

Ensuite, créez un nouveau fichier nommé *CILTypes.il* à l'aide de l'éditeur de votre choix. La première tâche d'un projet CIL consiste à lister l'ensemble des assemblies externes utilisés par l'assembly courant. Dans cet exemple, vous utiliserez uniquement les types présents dans *System.Runtime.dll*. Pour ce faire, la directive `.assembly` sera qualifiée à l'aide de l'attribut `external`. Lorsque vous référencez un assembly à nom fort, tel que *System.Runtime.dll*, vous devrez également spécifier les directives `.publickeytoken` et `.ver`, comme ceci :

```cil
.assembly extern System.Runtime
{
  .publickeytoken = (B0 3F 5F 7F 11 D5 0A 3A )
  .ver 10.0,8.0
}
{
  .assembly extern System.Runtime.Extensions
  .publickeytoken = (B0 3F 5F 7F 11 D5 0A 3A )
  .ver 10.0.8.0
}
.assembly extern mscorlib
{
  .publickeytoken = (B0 3F 5F 7F 11 D5 0A 3A )
  .ver = 10.0.8.0
}
```

## Définition de l'assembly courant en CIL

**L'étape suivante consiste à définir l'assembly que vous souhaitez compiler à l'aide de la directive `.assembly`**. **==Au niveau le plus simple, un assembly peut être défini en spécifiant le nom convivial du binaire==**, comme ceci :

```cil
// Notre assembly
.assembly CILTypes
{ }
```

**Bien que cela définisse effectivement un nouvel assembly .NET Core, vous placerez généralement des directives supplémentaires dans la portée de la déclaration d'assembly**. Pour cet exemple, ==mettez à jour votre définition d'assembly pour inclure un numéro de version `1.0.0.0` à l'aide de la directive `.ver`== (**==notez que chaque identifiant numérique est séparé par des deux-points, et non par la notation pointée propre au C#==**), comme suit :

```cil
// Notre assembly
.assembly CILTypes
{
  .ver 1:0:0:0
}
```

Étant donné que l'assembly *CILTypes* est un assembly mono-fichier, vous terminerez sa définition
à l'aide de la directive `.module` suivante, qui indique le nom officiel de votre binaire .NET *CILTypes.dll* :

```cil
.assembly CILTypes
{
  .ver 1:0:0:0
}
// Le module de notre assembly mono-fichier
.module CILTypes.dll
```

Outres `.assembly` et `.module`, **il existe des directives CIL qui précisent davantage la structure globale du binaire .NET que vous composez**. Le [[#Tableau 18-1 Directives supplémentaires axées sur l'assembly|Tableau 18-1]] répertorie quelques-unes des directives de niveau assembleur les plus courantes.

##### Tableau 18-1: Directives supplémentaires axées sur l'assembly

| Directive     | Description                                                                                                                                                                                                                                                                 |
| ------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `.mresources` | **Si votre assemblage utilise des ressources internes** (telles que des bitmaps ou des tables de chaînes), **cette directive est utilisée pour identifier le nom du fichier contenant les ressources à incorporer**.                                                        |
| `.subsystem`  | **Cette directive CIL permet de définir l'interface utilisateur dans laquelle l'assembly doit s'exécuter**. Par exemple, ==la valeur `2` indique que l'assembly doit s'exécuter dans une *application graphique*, tandis que la valeur `3` désigne un *exécutable console*. |

## Définition des espaces de noms en CIL

Maintenant que vous avez défini l'apparence de votre assembly (et les références externes requises), vous pouvez créer un espace de noms .NET Core (`MyNamespace`) à l'aide de la directive `.namespace`, comme ceci :

```cil
// Notre assembly possède un seul espace de noms
.namespace MyNamespace {}
```

**Comme en C#, les définitions d'espaces de noms CIL peuvent être imbriquées dans d'autres espaces de noms**. ==Il n'est pas nécessaire de définir un espace de noms racine ici ; toutefois, à titre d'exemple, supposons que vous souhaitiez créer l'espace de noms racine suivant, nommé `MyCompany` :

```cil
.namespace MyCompany
{
  .namespace MyNamespace {}
}
```

Comme en C#, CIL permet de définir un espace de noms imbriqué comme suit :

```cil
// Définition d'un espace de noms imbriqué.
.namespace MyCompany.MyNamespace {}
```

## Définition des types de classes en CIL

*==Les espaces de noms vides ne présentent pas un grand intérêt==*. Voyons donc comment définir un type de classe en utilisant CIL. Sans surprise, **la directive `.class` est utilisée pour définir une nouvelle classe. Cependant, cette simple directive peut être enrichie de nombreux attributs supplémentaires, afin de préciser la nature du type**. Pour illustrer cela, ==ajoutez une classe publique à votre espace de noms, nommée `MyBaseClass`==. Comme en C#, si vous ne spécifiez pas de classe de base explicite, votre type héritera automatiquement de `System.Object`.

```cil
.namespace MyNamespace
{
  // Classe de base System.Object supposée.
  .class public MyBaseClass {}
}
```

**Lorsque vous créez un type de classe dérivé d'une classe autre que `System.Object`, vous utilisez l'attribut `extends`**. **==Lorsque vous devez référencer un type défini dans le même assembly, CIL exige l'utilisation du nom complet==** (cependant, ==si le type de base se trouve dans le même assembly, vous pouvez omettre le préfixe convivial de l'assembly==). Par conséquent, la tentative suivante d'étendre `MyBaseClass` provoque une erreur de compilation :

```cil
// Ceci ne compilera pas !
.namespace MyNamespace
{
  .class public MyBaseClass {}

  .class public MyDerivedClass
    extends MyBaseClass {}
}
```

Pour définir correctement la classe parente de MyDerivedClass, vous devez spécifier le nom complet de MyBaseClass comme suit :

```cil
// Mieux !
.namespace MyNamespace
{
  .class public MyBaseClass {}

  .class public MyDerivedClass
    extends MyNamespace.MyBaseClass {}
}
```

**Outre les attributs `public` et `extends`, une définition de classe CIL peut comporter de nombreux qualificateurs supplémentaires qui contrôlent la visibilité du type, la disposition des champs, etc**. Le [[#Tableau 18-2 Divers attributs utilisés conjointement avec la directive `.class`|Tableau 18-2]] illustre certains (mais pas tous) des attributs pouvant être utilisés conjointement avec la directive `.class`.

##### Tableau 18-2: Divers attributs utilisés conjointement avec la directive `.class`


| Attributs                                                                                                                             | Description                                                                                                                                                                                                                                                                                                 |
| ------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `public`, `private`, `nested assembly`, `nested famandassem`, `nested family`, `nested famorassem`, `nested public`, `nested private` | CIL définit divers attributs permettant de spécifier la visibilité d'un type donné. Comme vous pouvez le constater, le CIL brut offre de nombreuses possibilités différentes de celles offertes par C#. Consultez la documentation ECMA 335 pour plus de détails.                                           |
| `abstract`, `sealed`                                                                                                                  | Ces deux attributs peuvent être ajoutés à une directive `.class` pour définir respectivement une classe abstraite ou une classe scellée.                                                                                                                                                                    |
| `auto`, `sequential`, `explicit`                                                                                                      | Ces attributs servent à indiquer au CLR comment organiser les données des champs en mémoire. Pour les types de classe, l'indicateur d'organisation par défaut (auto) est approprié. Modifier cette valeur par défaut peut s'avérer utile si vous devez utiliser P/Invoke pour appeler du code C non managé. |
| `extends`, `implements`                                                                                                               | Ces attributs vous permettent de définir la classe de base d'un type (via `extends`) ou d'implémenter une interface sur un type (via `implements`).                                                                                                                                                         |

##### Tableau 18-2.5: Les correspondances réelles (Mots-clés C# vs Attributs CIL)
| Modificateur C#      | Attribut CIL équivalent               | Portée réelle dans le Runtime           |
| -------------------- | ------------------------------------- | --------------------------------------- |
| `public`             | `public`                              | Accessible de partout.                  |
| `internal`           | `assembly`                            | Limité à l'assembly en cours.           |
| `protected`          | `family`                              | Limité aux classes dérivées uniquement. |
| `private`            | `private`                             | Limité à la classe contenant le membre. |
| `protected internal` | `famorassem` _(Family OR Assembly)_   | Dérivé **OU** dans le même assembly.    |
| `private protected`  | `famandassem` _(Family AND Assembly)_ | Dérivé **ET** dans le même assembly.    |

### Les attributs CIL que C# n'utilise pas (ou différemment)

Le CIL possède d'autres attributs de visibilité natifs qui n'ont pas d'équivalent mot-clé direct pour les types de premier niveau en C# :

1. **`compilercontrolled` (ou `privatescope`) :** Indique qu'un membre n'est accessible que par des outils spécifiques ou via une signature brute d'adresse, et non par son nom. Le compilateur C# l'utilise en arrière-plan pour des variables générées automatiquement (comme les champs d'un backing-field de propriété), mais vous ne pouvez pas l'écrire en C#.
2. **`nested private` / `nested assembly` :** Le CIL fait une distinction stricte dans ses tables de métadonnées entre la visibilité d'une classe de premier niveau (Top-level) et une classe imbriquée (Nested). En C#, vous utilisez le même mot-clé `private`, mais le compilateur va traduire cela en `nested private` si la classe est imbriquée

## Définition et implémentation d'interfaces en CIL

**Aussi étrange que cela puisse paraître, les types d'interface sont définis en CIL à l'aide de la directive `.class`**. Cependant, **==lorsque la directive `.class` est munie de l'attribut `interface`, le type est réalisé comme un type d'interface CTS==**. ==Une fois l'interface définie, elle peut être liée à une classe ou à une structure à l'aide de l'attribut `implements` de CIL==, comme ceci :

```cil
.namespace MyNamespace
{
  // Définition d'une interface.
  .class public interface IMyInterface {}
  
  // Classe de base simple.
  .class public MyBaseClass {}
  
  // MyDerivedClass implémente désormais IMyInterface,
  // et étend MyBaseClass.
  .class public MyDerivedClass
    extends MyNamespace.MyBaseClass
    implements MyNamespace.IMyInterface {}
}
```

>[!note]
>La clause `extends` doit précéder la clause `implements`. De plus, la clause `implements` peut incorporer une liste d'interfaces séparées par des virgules.

Comme vous vous en souvenez peut-être du [[Chapitre 8#Conception de hiérarchies d'interfaces|Chapitre 8]], **les interfaces peuvent servir d'interface de base à d'autres types d'interfaces pour construire des hiérarchies d'interfaces**. Cependant, ==contrairement à ce que vous pourriez penser, l'attribut `extends` ne peut pas dériver l'interface A de l'interface B==. ***==L'attribut `extends` sert uniquement à qualifier la classe de base d'un type==***. **Pour étendre une interface, vous utiliserez à nouveau l'attribut `implements`**. Voici un exemple :

```cil
// Extension des interfaces en CIL.
.class public interface IMyInterface {}

.class public interface IMyOtherInterface
implements MyNamespace.IMyInterface {} 
```

## Définition de structures en CIL

**La directive `.class` permet de définir une structure CTS si le type étend `System.ValueType`**. De plus, **==la directive `.class` doit être qualifiée avec l'attribut `sealed`==** (***==étant donné qu'une structure ne peut jamais être une structure de base pour d'autres types valeur==***). ==Si vous tentez de faire autrement, *ilasm.exe* générera une erreur de compilation.==

```cil
// Une définition de structure est toujours scellée.
.class public sealed MyStruct
  extends [System.Runtime]System.ValueType{}
```

**Notez que CIL fournit une notation abrégée pour définir un type de structure. Si vous utilisez l'attribut `value`, le nouveau type héritera automatiquement du type de `[System.Runtime]System.ValueType`**. Par conséquent, vous pouvez définir `MyStruct` comme suit :

```cil
// Notation abrégée pour déclarer une structure.
.class public sealed value MyStruct{}
```

## Définition d'énumérations en CIL

**Les énumérations .NET Core** (comme vous vous en souvenez) **dérivent de `System.Enum`, qui est un `System.ValueType`** (***==et doit donc également être scellé==***). Lorsque vous souhaitez définir une énumération en CIL, il suffit d'étendre `[System.Runtime]System.Enum`, comme ceci :

```cil
// Une Énumération.
.class public sealed MyEnum
  extends [System.Runtime]System.Enum{}
```

**À l'instar des définitions de structures, les énumérations peuvent être définies à l'aide d'une notation abrégée grâce à l'attribut `enum`.** Voici un exemple :

```cil
// Notation abrégée d'une énumération.
.class public sealed enum MyEnum{}
```

==Vous verrez dans un instant comment spécifier les paires nom-valeur d'une énumération.==

## Définition des génériques en CIL

Les types génériques ont également une représentation spécifique dans la syntaxe CIL. **Rappelons** ([[Chapitre 10#Le rôle des paramètres de type générique|Chapitre 10]]) **qu'un type générique donné, ou un membre générique, peut avoir un ou plusieurs paramètres de type**. ==Par exemple, le type `List<T>` possède un seul paramètre de type, tandis que `Dictionary<TKey, TValue>` en possède deux==. ***==En CIL, le nombre de paramètres de type est spécifié à l'aide d'une apostrophe inclinée vers l'arrière, suivie d'une valeur numérique représentant le nombre de paramètres de type==***. Comme en C#, la valeur des paramètres de type est placée entre chevrons.

>[!note] Sur les claviers américains, le caractère ``` ` ``` se trouve généralement sur la touche située au-dessus de la touche Tab (et à gauche de la touche 1).

Par exemple, supposons que vous souhaitiez créer une variable `List<T>`, où `T` est de type `System.Int32`. En C#, vous saisiriez le code suivant :

```cs
void SomeMethod()
{
	List<int> myInts = new List<int>();
}
```

En CIL, vous écririez le code suivant (qui pourrait apparaître dans n'importe quelle portée de méthode CIL) :

```cil
// En C# : List<int> myInts = new List<int>();
newobj instance void class [System.Collections]
  System.Collections.Generic.List`1<int32>::.ctor()
```

**Notez que cette classe générique est définie comme ```List'1<int32>```, car `List<T>` possède un seul paramètre de type**. Toutefois, si vous deviez définir un type `Dictionary<string, int>`, vous procéderiez comme suit :

```cil
// En C# : Dictionary<string, int> d = new Dictionary<string, int>();
newobj instance void class [System.Collections]
  System.Collections.Generic.Dictionary`2<string,int32>
  ::.ctor()
```

==À titre d'exemple, si vous avez un type générique qui utilise un autre type générique comme paramètre de type==, vous écririez du code CIL comme suit :

```cil
// En C# : List<List<int>> myInts = new List<List<int>>();
newobj instance void class [mscorlib]
  System.Collections.Generic.List`1<class
	[System.Collections]
	System.Collections.Generic.List`1<int32>>
	::.ctor()
```

## Compilation du fichier *CILTypes.il*

Bien que vous n'ayez pas encore ajouté de membres ni de code d'implémentation aux types que vous avez définis, ==vous pouvez compiler ce fichier *.il* en un assembly DLL .NET Core== (ce qui est nécessaire, car vous n'avez pas spécifié de méthode `Main()`). Ouvrez une invite de commandes et saisissez la commande suivante :

```bash
dotnet build
```

Une fois cette opération effectuée, vous pouvez ouvrir votre assembly compilé dans *ildasm.exe* pour vérifier la création de chaque type. Pour comprendre comment initialiser un type, vous devez d'abord examiner les types de données fondamentaux du CIL.

# Correspondances entre la bibliothèque de classes de base .NET, C# et les types de données CIL

**==Le [[#Tableau 18-3 Correspondance entre les types de classes de base .NET et les mots clés C , et entre les mots clés C et CIL.|Tableau 18-3]] illustre la correspondance entre un type de classe de base .NET et le mot-clé C# correspondant, ainsi que la correspondance de chaque mot-clé C# avec le code CIL brut==**. **Ce tableau documente également les notations de constantes abrégées utilisées pour chaque type CIL. Comme vous le verrez, ces constantes sont fréquemment référencées par de nombreux opcodes CIL.**

##### Tableau 18-3: Correspondance entre les types de classes de base .NET et les mots clés C#, et entre les mots clés C# et CIL.

| Type de classe de base .NET Core | Mot clé C# | Représentation CIL | Notation constante CIL |
| -------------------------------- | ---------- | ------------------ | ---------------------- |
| `System.SByte`                   | `sbyte`    | `int8`             | `I1`                   |
| `System.Byte`                    | `byte`     | `unsigned int8`    | `U1`                   |
| `System.Int16`                   | `short`    | `int16`            | `I2`                   |
| `System.UInt16`                  | `ushort`   | `unsigned int16`   | `U2`                   |
| `System.Int32`                   | `int`      | `int32`            | `I4`                   |
| `System.UInt32`                  | `uint`     | `unsigned int32`   | `U4`                   |
| `System.Int64`                   | `long`     | `int64`            | `I8`                   |
| `System.UInt64`                  | `ulong`    | `unsigned int64`   | `U8`                   |
| `System.Char`                    | `char`     | `char`             | `CHAR`                 |
| `System.Single`                  | `float`    | `float32`          | `R4`                   |
| `System.Double`                  | `double`   | `float64`          | `R8`                   |
| `System.Boolean`                 | `bool`     | `bool`             | `BOOLEAN`              |
| `System.String`                  | `string`   | `string`           | N/A                    |
| `System.Object`                  | `object`   | `object`           | N/A                    |
| `System.Void`                    | `void`     | `void`             | `VOID`                 |

>[!note]
>Les types `System.IntPtr` et `System.UIntPtr` correspondent respectivement aux types natifs `int` et `unsigned int`. (De nombreux scénarios d'interopérabilité COM et P/Invoke les utilisent largement).

# Définition des membres de type en CIL

Comme vous le savez déjà, **les types .NET peuvent prendre en charge différents membres**. ==Les énumérations possèdent un ensemble de paires nom-valeur==. **==Les structures et les classes peuvent avoir des constructeurs, des champs, des méthodes, des propriétés, des membres statiques, et ainsi de suite==**. **Au cours des 18 premiers chapitres de ce livre, vous avez déjà vu des définitions CIL partielles pour les éléments mentionnés précédemment, mais voici néanmoins un bref récapitulatif de la façon dont les différents membres correspondent aux primitives CIL.**

## Définition des données de champ en CIL

**Les énumérations, les structures et les classes peuvent toutes prendre en charge des données de champ**. **==Dans chaque cas, la directive `.field` sera utilisée==**. Par exemple, donnons vie à l'énumération `MyEnum` et définissons les trois paires nom-valeur suivantes (==notez que les valeurs sont spécifiées entre parenthèses==) :

```cil
.class public sealed enum MyEnum
{
	.field public static literal valuetype
	MyNamespace.MyEnum A = int32(0)
	
	.field public static literal valuetype
	MyNamespace.MyEnum B = int32(1)
	
	.field public static literal valuetype
	MyNamespace.MyEnum C = int32(2)
}
```

**Les champs appartenant à un type dérivé de `System.Enum` de .NET Core sont qualifiés à l'aide des attributs `static` et `literal`**. Comme vous pouvez l'imaginer, **==ces attributs définissent les données du champ comme une valeur fixe accessible depuis le type lui-même==** (par exemple, `MyEnum.A`).

>[!note]
>Les valeurs attribuées à une valeur d'énumération peuvent également être en hexadécimal avec un préfixe `0x`.

**Bien entendu, lorsque vous souhaitez définir un point de données de champ au sein d'une classe ou d'une structure, vous n'êtes pas limité à un point de données littérales statiques publiques**. Par exemple, vous pouvez modifier `MyBaseClass` pour prendre en charge deux points de données de champ privées, au niveau de l'instance, initialisées avec des valeurs par défaut.

```cil
.class public MyBaseClass
{
	.field private string stringField = "hello!"
	.field private int32 intField = int32(42)
}
```

**Comme en C#, les données des champs de classe seront automatiquement initialisées avec une valeur par défaut appropriée.** **==Si vous souhaitez permettre à l'utilisateur de l'objet de fournir des valeurs personnalisées lors de la création de chacun de ces champs privés, vous devrez==** (bien entendu) **==créer des constructeurs personnalisés.==**

## Définition des constructeurs de type en CIL

**Le CTS prend en charge les constructeurs d'instance et les constructeurs statiques** (au niveau de la classe). ***==En CIL, les constructeurs d'instance sont représentés par le jeton `.ctor`, tandis qu'un constructeur statique est exprimé par `.cctor`==*** (constructeur de classe). **Ces deux jetons CIL doivent être qualifiés à l'aide des attributs `rtspecialname`** (==nom spécial du type de retour==) **et `specialname`**. En d'autres termes, **==ces attributs permettent d'identifier un jeton CIL spécifique qui peut être traité de manière unique par un langage .NET donné==**. Par exemple, ==en C#, les constructeurs ne définissent pas de type de retour ; cependant, en CIL, la valeur de retour d'un constructeur est bien `void`.==

```cil
.class public MyBaseClass 
{
	// Quelque donnée de champs
	.field private string stringField 
	.field private int32 intField	
	
	// Un constructeur personnalisé
	.method public hidebysig specialname rtspecialname 
	  instance void .ctor(string s, int32 i) cil managed
	{
		// TODO: Ajouter l'implémentation
	}
```

**Notez que la directive `.ctor` a été qualifiée avec l'attribut `instance`** (car il ne s'agit pas d'un constructeur statique). ==Les attributs `cil managed` indiquent que la portée de cette méthode contient du code CIL, et non du code non managé, qui peut être utilisé lors des requêtes d'invocation de la plateforme.==

## Définition des propriétés en CIL

**Les propriétés et les méthodes ont également des représentations CIL spécifiques.** Par exemple, ==si la classe `MyBaseClass` était mise à jour pour prendre en charge une propriété publique nommée `TheString`, vous écririez le code CIL suivant== (notez à nouveau l'utilisation de l'attribut `specialname`) :

```cil
.class public MyBaseClass 
{
	...
	// Syntaxe de propriété
	.method public hidebysig specialname 
	instance string get_TheString() cil managed
	{
		// TODO: Ajouter l'implémentation
	}  

	.method public hidebysig specialname 
	instance void set_TheString(string 'value') cil managed
	{
		// TODO: Ajouter l'implémentation
	}  

	.property instance string TheString()
	{
	  .get instance string 
		MyNamespace.MyBaseClass::get_TheString()
	  .set instance void 
		MyNamespace. MyBaseClass::set_TheString(string)
	}
}
```

**En CIL, une propriété correspond à une paire de méthodes prenant les préfixes `get_` et `set_`. La directive `.property` utilise les directives `.get` et `.set` associées pour faire correspondre la syntaxe des propriétés aux méthodes « nommées spécifiquement ».**

>[!note]
>Notez que le paramètre entrant de la méthode `set` d'une propriété est placé entre guillemets simples qui représentent le nom du jeton à utiliser à droite de l'opérateur d'affectation dans la portée de la méthode.

## Définition des paramètres membres

En résumé, **la spécification des arguments en CIL est (plus ou moins) identique à celle en C#**. Par exemple, **==chaque argument est défini en spécifiant son type de données, suivi du nom du paramètre==**. De plus, **comme en C#, le CIL offre la possibilité de définir des paramètres d'entrée, de sortie et de passage par référence**. **==Le CIL permet également de définir un tableau de paramètres (équivalent du mot-clé `params` en C#), ainsi que des paramètres optionnels.==**

Pour illustrer la définition des paramètres en CIL pur, supposons que vous souhaitiez créer une méthode qui accepte un `int32` (par valeur), un `int32` (par référence), un `System.Collection.ArrayList` et un unique paramètre de sortie (de type `int32`). En C#, cette méthode ressemblerait à ce qui suit :

```cs
public static void MyMethod(int inputInt, ref int refInt, ArrayList ar, out int outputInt)
{
	outputInt = 0; // Juste pour satisfaire le compilateur C#
}
```

**Si vous deviez traduire cette méthode en termes CIL, vous constateriez que les paramètres de référence C# sont marqués par une esperluette (`&`) suivie du type de données sous-jacent du paramètre (`int32&`).**

**Les paramètres de sortie utilisent également le suffixe `&`, mais ils sont qualifiés en outre par le jeton CIL `[out]`**. ***==Notez également que si le paramètre est un type référence (dans ce cas, le type `[mscorlib]System.Collections.ArrayList`), le jeton de classe est préfixé au type de données (à ne pas confondre avec la directive `.class` !).==***

```cil
.method public hidebysig static void MyMethod(int32 inputInt, int32& refInt,
	class [System.Runtime.Extensions]System.Collections.ArrayList ar,
	[out] int32& outputInt) cil managed
{
	...	
}
```

# Examen des opcodes CIL

**Le dernier aspect du code CIL que nous aborderons dans ce chapitre concerne le rôle des différents codes opérationnels** (==opcodes==). ***==Rappelons qu'un opcode est simplement un jeton CIL utilisé pour construire la logique d'implémentation d'un membre donné==***. L'ensemble complet des opcodes CIL (qui est vaste) peut être regroupé dans les grandes catégories suivantes :

- Opcodes contrôlant le flux d'exécution du programme
- Opcodes évaluant des expressions
- Opcodes accédant aux valeurs en mémoire (via des paramètres, des variables locales, etc.)

Pour mieux comprendre l'implémentation des membres en CIL, le [[#Tableau 18-4 Divers codes d'opération CIL spécifiques à l'implémentation|Tableau 18-4]] présente certains des opcodes les plus utiles liés à la logique d'implémentation des membres, regroupés par fonctionnalité.

##### Tableau 18-4: Divers codes d'opération CIL spécifiques à l'implémentation

| Opcodes                              | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| ------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `add`, `sub`, `mul`, `div`, `rem`    | Ces codes d'opération CIL vous permettent d'additionner, de soustraire, de multiplier et de diviser deux valeurs (`rem` renvoie le reste d'une opération de division (modulo)).                                                                                                                                                                                                                                                                                                                                                                                                |
| `and`, `or`, `not`, `xor`            | Ces codes d'opération CIL vous permettent d'effectuer des opérations bit à bit sur deux valeurs.                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| `ceq`, `cgt`, `clt`                  | Ces opcodes CIL permettent de comparer deux valeurs sur la pile de différentes manières. Voici quelques exemples :<br>`ceq` : Comparaison d’égalité<br>`cgt` : Comparaison de valeur supérieure à<br>`clt` : Comparaison de valeur inférieure à                                                                                                                                                                                                                                                                                                                                |
| `box`, `unbox`                       | Ces codes d'opération CIL sont utilisés pour convertir entre les types de référence et les types de valeur.                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `Ret`                                | Ce code d'opération CIL est utilisé pour quitter une méthode et renvoyer une valeur à l'appelant (si nécessaire).                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| `beq`, `bgt`, `ble`, `blt`, `switch` | Ces opcodes CIL (ainsi que de nombreux autres opcodes apparentés) servent à contrôler la logique de branchement au sein d'une méthode. Voici quelques exemples :<br><br>`beq` : Sortie vers l'étiquette de code si égal<br>`bgt` : Sortie vers l'étiquette de code si supérieur à<br>`ble` : Sortie vers l'étiquette de code si inférieur ou égal à<br>`blt` : Sortie vers l'étiquette de code si inférieur à<br><br>Tous les opcodes de branchement nécessitent la spécification d'une étiquette de code CIL vers laquelle effectuer le saut si le résultat du test est vrai. |
| `Call`                               | Ce code d'opération CIL est utilisé pour appeler un membre d'un type donné.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `newarr`, `newobj`                   | Ces opcodes CIL vous permettent d'allouer un nouveau tableau ou un nouveau type d'objet en mémoire (respectivement).                                                                                                                                                                                                                                                                                                                                                                                                                                                           |

**La catégorie suivante d'opcodes CIL** (==dont un sous-ensemble est présenté dans le [[#Tableau 18-5 Les opcodes principaux centrés sur la pile du CIL|Tableau 18-5]]==) **sert à charger (empiler) les arguments sur la pile d'exécution virtuelle**. Notez que ces opcodes spécifiques au chargement prennent le préfixe `ld` (chargement).

##### Tableau 18-5: Les opcodes principaux centrés sur la pile du CIL

| Opcode                                  | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| --------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ldarg` (avec de nombreuses variations) | Charge l'argument d'une méthode sur la pile. Outre l'opcode `ldarg` général (qui fonctionne avec un index identifiant l'argument), il existe de nombreuses autres variantes. Par exemple, les opcodes `ldarg` avec un suffixe numérique (`ldarg.0`) spécifient en dur l'argument à charger. De même, certaines variantes de l'opcode `ldarg` permettent de spécifier en dur le type de données à l'aide de la notation des constantes CIL présentée dans le [[#Tableau 18-3 Correspondance entre les types de classes de base .NET et les mots clés C , et entre les mots clés C et CIL.\|Tableau 18-3]] (`ldarg_I4`, pour un `int32`), ainsi que le type et la valeur (`ldarg_I4_5`, pour charger un `int32` de valeur $5$). |
| `ldc` (avec de nombreuses variations)   | Charge une valeur constante sur la pile.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| `ldfld` (avec de nombreuses variations) | Charge la valeur d'un champ au niveau de l'instance sur la pile.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| `ldloc` (avec de nombreuses variations) | Charge la valeur d'une variable locale sur la pile.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| `ldobj`                                 | Récupère toutes les valeurs collectées par un objet basé sur le tas et les place sur la pile.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| `ldstr`                                 | Charge une valeur de type chaîne de caractères sur la pile.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |

**Outre l'ensemble des opcodes spécifiques au chargement, CIL fournit de nombreux opcodes qui dépilent *explicitement* la valeur la plus haute de la pile**. Comme l'illustrent les premiers exemples de ce chapitre, **==dépiler une valeur implique généralement de la stocker dans une zone de stockage locale temporaire pour une utilisation ultérieure==** (par exemple, comme paramètre pour un appel de méthode ultérieur). À ce propos, notez combien d'opcodes qui dépilent la valeur courante de la pile d'exécution virtuelle prennent le préfixe `st` (stockage). Le tableau 18-6 en présente les principaux.

##### Tableau 18-6: Divers opcodes axés sur le dépilage

| Opcode                                  | Description                                                                                                                      |
| --------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `pop`                                   | Supprime la valeur actuellement en haut de la pile d'évaluation, mais ne prend pas la peine de la stocker.                       |
| `starg`                                 | Stocke la valeur en haut de la pile dans l'argument de la méthode à un index spécifié.                                           |
| `stloc` (avec de nombreuses variations) | Extrait la valeur actuelle du haut de la pile d'évaluation et la stocke dans une liste de variables locales à un index spécifié. |
| `stobj`                                 | Copie une valeur d'un type spécifié depuis la pile d'évaluation vers une adresse mémoire fournie.                                |
| `stsfld`                                | Remplace la valeur d'un champ statique par une valeur issue de la pile d'évaluation                                              |

**Sachez que divers opcodes CIL dépilent implicitement des valeurs pour effectuer la tâche en cours**. Par exemple, ==si vous tentez de soustraire deux nombres à l'aide de l'opcode `sub`, il est clair que `sub` devra dépiler les deux prochaines valeurs disponibles avant de pouvoir effectuer le calcul==. **==Une fois le calcul terminé, le résultat (sans surprise) est à nouveau empilé.==**

## La directive `.maxstack`

Lorsque vous écrivez des implémentations de méthodes en CIL pur, vous devez tenir compte d'une directive spéciale nommée `.maxstack`. ***==Comme son nom l'indique, `.maxstack` définit le nombre maximal de variables pouvant être empilées simultanément pendant l'exécution de la méthode==***. ==La bonne nouvelle est que la directive `.maxstack` a une valeur par défaut (`8`), ce qui devrait convenir à la grande majorité des méthodes que vous pourriez écrire==. Cependant, **==si vous souhaitez être explicite, vous pouvez calculer manuellement le nombre de variables locales sur la pile et définir cette valeur explicitement==**, comme ceci :

```cil
.method public hidebysig instance void Speak() cil managed
{
  // Pendant l'exécution de cette méthode, exactement
  // 1 valeur (la chaîne littérale) est sur la pile.
  .maxstack 1
  ldstr "Bonjour..."
  call void [mscorlib]System.Console::WriteLine(string)
  ret
}
```

## Déclaration de variables locales en CIL

Voyons d'abord comment déclarer une variable locale. ==Supposons que vous souhaitiez créer une méthode en CIL nommée `MyLocalVariables()` qui ne prend aucun argument et ne retourne aucune valeur==. Dans cette méthode, vous souhaitez définir trois variables locales de types `System.String`, `System.Int32` et `System.Object`. En C#, ce membre apparaîtrait comme suit (rappelons que les variables locales ne reçoivent pas de valeur par défaut et doivent être initialisées avant toute utilisation) :

```cs
public static void MyLocalVariables()
{
	string myStr = "CIL code is fun!";
	int. myInt = 33;
	object myObj = new object();
}
```

Si vous deviez construire `MyLocalVariables()` directement en CIL, vous pourriez écrire le code suivant :

```cil
.method public hidebysig static void MyLocalVariables() cil managed
{
  .maxstack 8
  // Définir trois variables locales.
  .locals init (string myStr, int32 myInt, object myObj)
  // Charger une chaîne de caractères sur la pile d'exécution virtuelle.
  ldstr "CIL code is fun!"
  // Dépiler la valeur actuelle et la stocker dans la variable locale [0].
  stloc.0

  // Charger une constante de type "i4"
  // (abréviation de int32) initialisée à la valeur 33.
  ldc.i4.s 33
  // Récupérer la valeur actuelle et la stocker dans 
  // la variable locale [1].
  stloc.1

  // Créer un nouvel objet et l'empiler.
  newobj instance void [mscorlib]System.Object::.ctor()
  // Dépiler la valeur actuelle et la stocker dans la variable locale [2].
  stloc.2
  ret		
}
```

**La première étape pour allouer des variables locales en CIL brut consiste à utiliser la directive `.locals`, associée à l'attribut `init`**. **==Chaque variable est identifiée par son type de données et un nom de variable facultatif. Une fois les variables locales définies, vous chargez une valeur sur la pile (à l'aide des opcodes de chargement) et vous stockez cette valeur dans la variable locale (à l'aide des opcodes de stockage).==**

## Association des paramètres aux variables locales en CIL

Vous avez déjà vu comment déclarer des variables locales en CIL brut à l'aide de la directive `.locals` ; cependant, vous n'avez pas encore vu comment associer les paramètres entrants aux méthodes locales. Prenons l'exemple de la méthode statique C# suivante :

```cs
public static int Add(int a, int b)
{
	return a + b;
}
```

**Cette méthode d'apparence anodine cache en réalité beaucoup d'opérations sous-jacente en termes de CIL**. Tout d'abord, ==les arguments entrants (`a` et `b`) doivent être empilés sur la pile d'exécution virtuelle à l'aide de l'opcode `ldarg`== (chargement d'argument). ==Ensuite, l'opcode `add` est utilisé pour dépiler les deux valeurs suivantes, calculer leur somme et la stocker à nouveau sur la pile==. Enfin, ==cette somme est dépilée et renvoyée à l'appelant via l'opcode `ret`==. **Si vous désassembliez cette méthode C# à l'aide** d' *ildasm.exe*, **vous trouveriez de nombreux jetons supplémentaires injectés lors de la compilation, mais le cœur du code CIL reste assez simple.**

```cil
.method public hidebysig static int32 Add(int32 a, int32 b) cil managed
{
	.maxstack 2
	ldarg.0 // Load "a" onto the stack.
	ldarg.1 // Load "b" onto the stack.
	add     // Add both values.
	ret
}
```

## Référence cachée à `this`

**Notez que les deux arguments entrants (`a` et `b`) sont référencés dans le code CIL par leur position indexée (index 0 et index 1), étant donné que la pile d'exécution virtuelle commence à l'indexation à la position 0.**

==Lors de l'examen ou de la création de code CIL, il est important de se rappeler que toute méthode non statique qui prend des arguments entrants reçoit automatiquement un paramètre supplémentaire implicite, à savoir une référence à l'objet courant== (comme le mot-clé `this` en C#). Dans ce cas, si la méthode `Add()` était définie comme non statique, comme ceci :

```cs
// Plus statique !
public int Add(int a, int b)
{
	return a + b;
}
```

**Les arguments entrants `a` et `b` sont ensuite chargés à l'aide de `ldarg.1` et `ldarg.2`** (==au lieu des opcodes attendus `ldarg.0` et `ldarg.1`==). **==Là encore, la raison est que l'emplacement 0 contient la référence implicite `this`==**. Considérez le pseudocode suivant :

```CIL
// Ceci n'est qu'un pseudo-code !
.method public hidebysig static int32 AddTwoIntParams(
  MyClass_HiddenThisPointer this, int32 a, int32 b) cil managed
{ 
  ldarg.0 // Charger MyClass_HiddenThisPointer sur la pile.
  ldarg.1 // Charger "a" sur la pile.
  ldarg.2 // Charger "b" sur la pile.
  ...
}
```

## Représentation des structures d'itération en CIL

**Les structures d'itération du langage de programmation C# sont représentées par les mots-clés `for`, `foreach`, `while` et `do`, chacun ayant une représentation spécifique en CIL. Prenons l'exemple de la boucle `for` classique suivante :**

```cs
public static void CountToTen()
{
	for(int i = 0; i < 10; i++)
	{
	}
}
```

Comme vous vous en souvenez peut-être, **les opcodes `br` (`br`, `blt`, etc.) servent à interrompre l'exécution lorsqu'une condition est remplie**. **==Dans cet exemple, vous avez défini une condition selon laquelle la boucle `for` doit s'interrompre lorsque la variable locale `i` est supérieure ou égale à $10$==**. **À chaque itération, la valeur $1$ est ajoutée à `i`, puis la condition est réévaluée.**

**N'oubliez pas non plus que lorsque vous utilisez un opcode de branchement CIL, vous devez définir une ou deux étiquettes de code spécifiques indiquant l'emplacement où effectuer le saut lorsque la condition est vraie**. Compte tenu de ces points, examinez le code CIL suivant (modifié) généré par *ildasm.exe* (étiquettes de code générées automatiquement incluses) :

```cil
.method public hidebysig static void CountToTen() cil managed
{
  .maxstack 2
  .locals init (int32 V_0)
  IL_0000: ldc.i4.0     // Charger cette valeur sur la pile.
  IL_0001: stloc.0      // Stockez cette valeur à l'index "0".
  IL_0002: br.s IL_0007 // Passer à IL_0007.
  IL_0003: ldloc.0      // Charger la valeur de la variable à l'index 0.
  IL_0004: ldc.i4.1     // Charger la valeur « 1 » sur la pile.
  IL_0005: add    // Ajouter la valeur actuelle à l'index 0 de la pile.
  IL_0006: stloc.0
		IL_0007: ldloc.0      // Charger la valeur à l'index "0".
  IL_0008: ldc.i4.s 10  // Charger la valeur « 10 » sur la pile.
  IL_0009: clt    // Vérifier si la valeur sur la pile est inférieure à la valeur indiquée
  IL_000a: stloc.1      // Stocker le résultat à l'index "1"
  IL_000b: ldloc.1      // Charger la valeur à l'index "1"
  IL_000c: brtrue.s IL_0002 // Si vrai, retour à IL_0002
  IL_000d: ret
}
```

**En résumé, ce code CIL commence par définir la variable locale `int32` et la charger sur la pile**. ==À ce stade, vous effectuez des allers-retours entre les étiquettes de code `IL_0008` et `IL_0004`, en incrémentant à chaque fois la valeur de `i` de `1` et en vérifiant si `i` est toujours inférieur à $10$==. Si c'est le cas, vous quittez la méthode.

## Le mot de la fin sur le CIL

Maintenant que vous comprenez le processus de création d'un exécutable à partir d'un fichier *.IL*, vous vous dites probablement : «C'est un travail considérable» et vous vous demandez ensuite : « Quel est l'intérêt ? » **Dans la grande majorité des cas, vous ne créerez jamais un exécutable .NET Core à partir d'IL. Cependant, la maîtrise de l'IL peut s'avérer utile si vous cherchez à explorer un assembly dont vous ne possédez pas le code source.**

==Il existe également des projets commerciaux capables de rétroconcevoir un assembly .NET pour en extraire le code source==. Si vous avez déjà utilisé l'un de ces outils, vous savez maintenant comment ils fonctionnent !

# Comprendre les assemblies dynamiques

>[!failure] Cette section du livre est obsolètre
>
>- **Le conflit avec le Native AOT :** Générer un assembly dynamique dans la RAM requiert un moteur JIT (Just-In-Time) actif. À l'ère de .NET 10, où la priorité absolue est la compilation native en amont (**Native AOT**) et le **Trimming**, l'utilisation de `Reflection.Emit` classique brise complètement la chaîne de publication et lève des exceptions fatales au runtime.
>- **Remplacé par les Source Generators :** À l'époque de l'écriture du livre, pour créer du code dynamique rapide (comme le ferait un outil de mapping de base de données ou d'injection de dépendances), on utilisait `Reflection.Emit`. Aujourd'hui, 99 % de ces scénarios sont gérés au moment de la compilation par des **Source Generators (C# 9+)**, éliminant le besoin de manipuler du CIL manuellement à l'exécution.

Certes, développer une application .NET complexe en CIL serait un véritable travail de longue haleine. D'une part, le CIL est un langage de programmation extrêmement expressif qui permet d'interagir avec toutes les constructions de programmation autorisées par le CTS. D'autre part, écrire du CIL brut est fastidieux, source d'erreurs et pénible. S'il est vrai que le savoir est un pouvoir, ==vous pourriez vous demander à quel point il est important de mémoriser les règles de la syntaxe CIL==. **La réponse est : "cela dépend"**. Certes, **==la plupart de vos projets de programmation .NET ne nécessiteront pas de consulter, de modifier ou d'écrire du code CIL. Cependant, maintenant que vous maîtrisez les bases du CIL, vous êtes prêt à explorer le monde des assemblies dynamiques==** (==par opposition aux assemblies statiques==) **==et le rôle de l'espace de noms `System.Reflection.Emit`==**.

La première question que vous pourriez vous poser est : =="Quelle est la différence exacte entre les assemblies statiques et dynamiques ?"== **Par définition, les *assemblies statiques* sont des binaires .NET chargés directement depuis le disque, c'est-à-dire qu'ils se trouvent dans un fichier physique (ou éventuellement un ensemble de fichiers dans le cas d'un assembly multi-fichier) au moment où le CLR les demande**. Comme vous pouvez l'imaginer, ***==chaque fois que vous compilez votre code source C#, vous obtenez un assembly statique.==***

**Un *assembly dynamique*, en revanche, est créé en mémoire, à la volée, à l'aide des types fournis par l'espace de noms `System.Reflection.Emit`**. ==Cet espace de noms permet de créer un assembly et ses modules, définitions de types et logique d'implémentation CIL à l'*exécution*==. **Une fois cela fait, vous pouvez enregistrer votre binaire en mémoire sur le disque. Cela génère, bien sûr, un nouvel assembly statique**. ***==Certes, la création d'un assemblage dynamique à l'aide de l'espace de noms `System.Reflection.Emit` requiert une certaine compréhension de la nature des opcodes CIL.==***

Bien que la création d'assemblages dynamiques soit une tâche de programmation avancée (***==et peu courante==***), elle peut s'avérer utile dans diverses circonstances. Voici un exemple :

- Vous développez un outil de programmation .NET qui doit générer des assemblages à la demande, en fonction des entrées utilisateur.
- Vous développez un programme qui doit générer à la volée des proxys vers des types distants, à partir des métadonnées obtenues.
- Vous souhaitez charger un assemblage statique et insérer dynamiquement de nouveaux types dans l'image binaire.

Examinons les types disponibles dans `System.Reflection.Emit`.

## Exploration de l'espace de noms `System.Reflection.Emit`

**La création d'un assembly dynamique nécessite une certaine familiarité avec les opcodes CIL, mais les types de l'espace de noms `System.Reflection.Emit` masquent autant que possible la complexité du CIL**. Par exemple, ==au lieu de spécifier les directives et attributs CIL nécessaires pour définir un type de classe, vous pouvez simplement utiliser la classe `TypeBuilder`==. De même, ==si vous souhaitez définir un nouveau constructeur d'instance, il n'est pas nécessaire d'émettre le jeton `specialname`, `rtspecialname` ou `.ctor` ; vous pouvez utiliser `ConstructorBuilder`==. Le [[#Tableau 18-7 Sélection de membres de l'espace de noms `System.Reflection.Emit`|Tableau 18-7]] répertorie les membres clés de l'espace de noms `System.Reflection.Emit`.

##### Tableau 18-7: Sélection de membres de l'espace de noms `System.Reflection.Emit`

| Membres                                                                                                                                                 | Description                                                                                                                                                                                                                                                             |
| ------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `AssemblyBuilder`                                                                                                                                       | Utilisé pour créer un assembly (*.dll* ou *.exe*) à l'exécution. Les fichiers *.exe* doivent appeler la méthode `ModuleBuilder.SetEntryPoint()` pour définir le point d'entrée du module. Si aucun point d'entrée n'est spécifié, une bibliothèque *.dll* sera générée. |
| `ModuleBuilder`                                                                                                                                         | Utilisé pour définir l'ensemble des modules au sein de l'assemblage actuel.                                                                                                                                                                                             |
| `EnumBuilder`                                                                                                                                           | Utilisé pour créer un type d'énumération .NET.                                                                                                                                                                                                                          |
| `TypeBuilder`                                                                                                                                           | Peut être utilisé pour créer des classes, des interfaces, des structures et des délégués au sein d'un module lors de l'exécution.                                                                                                                                       |
| `MethodBuilder`, `LocalBuilder`, `PropertyBuilder`, `FieldBuilder`,  `ConstructorBuilder`, `CustomAttributeBuilder`, `ParameterBuilder`, `EventBuilder` | Utilisé pour créer des membres de type (tels que des méthodes, des variables locales, des propriétés, des constructeurs et des attributs) lors de l'exécution.                                                                                                          |
| `ILGenerator`                                                                                                                                           | Émet des opcodes CIL dans un membre de type donné.                                                                                                                                                                                                                      |
| `OpCodes`                                                                                                                                               | Fournit de nombreux champs correspondant à des opcodes CIL. Ce type est utilisé en conjonction avec les différents membres de `System.Reflection.Emit.ILGenerator`.                                                                                                     |

**En général, les types de l'espace de noms `System.Reflection.Emit` permettent de représenter des jetons CIL bruts par programmation lors de la construction de votre assembly dynamique**. Vous verrez plusieurs de ces membres dans l'exemple suivant ; toutefois, le type `ILGenerator` mérite d'être examiné d'emblée.

## Rôle de `System.Reflection.Emit.ILGenerator`

**Comme son nom l'indique, le rôle du type `ILGenerator` est d'injecter des opcodes CIL dans un membre de type donné**. Cependant, *==vous ne pouvez pas créer directement d'objets `ILGenerator`, car ce type ne possède pas de constructeurs publics==*; **==vous obtenez plutôt un type `ILGenerator` en appelant des méthodes spécifiques des types centrés sur le constructeur==** (tels que les types `MethodBuilder` et `ConstructorBuilder`). Voici un exemple :

```cs
// Obtenir un ILGenerator à partir d'un ConstructorBuilder
// objet nommé « myCtorBuilder ».
ConstructorBuilder myCtorBuilder = helloWorldClass.DefineConstructor( 
	MethodAttributes.Public, 
	CallingConventions.Standard, 
	constructorArgs
);

ILGenerator myCILGen = myCtorBuilder.GetILGenerator();
```

**Une fois que vous disposez d'un `ILGenerator`, vous pouvez générer les opcodes CIL bruts à l'aide de différentes méthodes**. Le [[#Tableau 18-8 Diverses méthodes de `ILGenerator`|Tableau 18-8]] présente certaines (mais pas toutes) les méthodes de `ILGenerator`.

##### Tableau 18-8: Diverses méthodes de `ILGenerator`

| Method                  | Meaning in Life                                                                                                          |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| `BeginCatchBlock()`     | Débute un bloc `catch`                                                                                                   |
| `BeginExceptionBlock()` | Démarre la portée d'une exception pour une exception                                                                     |
| `BeginFinallyBlock()`   | Débute d'un bloc `finally`                                                                                               |
| `BeginScope()`          | Débute une étendue lexicale                                                                                              |
| `DeclareLocal()`        | Déclare une variable locale                                                                                              |
| `DefineLabel()`         | Déclare une nouvelle étiquette                                                                                           |
| `Emit()`                | Est surchargé à de nombreuses reprises pour vous permettre d'émettre des codes d'opération CIL.                          |
| `EmitCall()`            | Injecte un opcode `call` ou `callvirt` dans le flux CIL                                                                  |
| `EmitWriteLine()`       | Émet un appel à `Console.WriteLine()` avec différents types de valeurs                                                   |
| `EndExceptionBlock()`   | Termine un bloc `Exception`                                                                                              |
| `EndScope()`            | Termine une étendue lexicale                                                                                             |
| `ThrowException()`      | Émet une instruction pour lever une exception                                                                            |
| `UsingNamespace()`      | Spécifie l'espace de noms à utiliser pour évaluer les variables locales et surveille l'étendue lexicale active actuelle. |

**La méthode principale d'`ILGenerator` est `Emit()`, qui fonctionne conjointement avec le type de classe `System.Reflection.Emit.OpCodes`**. Comme mentionné précédemment dans ce chapitre, **==ce type expose un grand nombre de champs en lecture seule qui correspondent à des opcodes CIL bruts==**. L'ensemble de ces membres est documenté dans l'aide en ligne, et vous voir divers exemples dans les pages suivantes.

## Émission d'un assembly dynamique

**Pour illustrer le processus de définition d'un assembly .NET Core à l'exécution, examinons la création d'un assembly dynamique mono-fichier**. Cet assembly contient une classe nommée `HelloWorld`. La classe `HelloWorld` possède un constructeur par défaut et un constructeur personnalisé permettant d'assigner la valeur d'une variable membre privée (`theMessage`) de type `string`. De plus, `HelloWorld` possède une méthode d'instance publique `SayHello()` qui affiche un message de bienvenue sur le flux d'entrée/sortie standard, ainsi qu'une autre méthode d'instance `GetMsg()` qui renvoie la chaîne de caractères privée. En pratique, vous allez générer par programmation le type de classe suivant :

```cs
// Cette classe sera créée à l'exécution.
// en utilisant  System.Reflection.Emit.
public class HelloWorld
{ 
	private string theMessage;

	HelloWorld() {}
	HelloWorld(string s) {theMessage = s;}
	public string GetMsg() {return theMessage;}
	public void SayHello()
	{
	 System.Console.WriteLine("Hello from the HelloWorld class!");
	}
}
```

Supposons que vous ayez créé un nouveau projet d'application console nommé *DynamicAsmBuilder* ~~et que vous ayez ajouté le package NuGet `System.Reflection.Emit`~~. Ensuite, importez les espaces de noms `System.Reflection` et `System.Reflection.Emit`. Définissez une méthode statique nommée `CreateMyAsm()` dans le fichier *Program.cs*. Cette méthode unique gère les opérations suivantes :

- Définition des caractéristiques de l'assembly dynamique (nom, version, etc.)
- Implémentation du type `HelloClass`
- Retour de l'`AssemblyBuilder` à la méthode appelante

Voici le code complet, suivi de son analyse :

>[!tip] Dans les version modernes de .NET, `System.Reflection.Emit` est déjà présent dans le SDK, il ne faut pas importer de package supplémentaire.

```cs
static AssemblyBuilder CreateMyAsm()
{
    // Établit les caractéristiques générales de l'assembly
    AssemblyName assemblyName = new AssemblyName
    {
        Name = "MyAssembly",
        Version = new Version("1.0.0.0"),
    };

    // Crée la nouvelle assembly
    var builder = AssemblyBuilder.DefineDynamicAssembly(
        assemblyName,
        AssemblyBuilderAccess.Run
    );

    // Défini le nom du module
    ModuleBuilder module = builder.DefineDynamicModule("MyAssembly");

    // Défini une classe publique nommée "HelloWorld"
    TypeBuilder helloWorldClass = module.DefineType(
        "MyAssembly.HelloWorld",
        TypeAttributes.Public
    );

    // Défini une variable privée de type string nommée "theMessage".
    FieldBuilder msgField = helloWorldClass.DefineField(
        "theMessage",
        Type.GetType("System.String"),
        attributes: FieldAttributes.Private
    );

    // Crée le constructeur personnalisé
    Type[] constructorArgs = [typeof(string)];
    ConstructorBuilder constructor = helloWorldClass.DefineConstructor(
        MethodAttributes.Public,
        CallingConventions.Standard,
        constructorArgs
    );
    ILGenerator constructorIl = constructor.GetILGenerator();
    constructorIl.Emit(OpCodes.Ldarg_0);
    Type objectClass = typeof(object);
    ConstructorInfo superConstructor = objectClass.GetConstructor(new Type[0]);
    constructorIl.Emit(OpCodes.Call, superConstructor);
    constructorIl.Emit(OpCodes.Ldarg_0);
    constructorIl.Emit(OpCodes.Ldarg_1);
    constructorIl.Emit(OpCodes.Stfld, msgField);
    constructorIl.Emit(OpCodes.Ret);

    // Crée le constructeur par défaut
    helloWorldClass.DefineDefaultConstructor(MethodAttributes.Public);

    // Maintenant, crée la méthode GetMsg()
    MethodBuilder getMsgMethod = helloWorldClass.DefineMethod(
        "GetMsg",
        MethodAttributes.Public,
        typeof(string),
        null
    );

    ILGenerator methodIl = getMsgMethod.GetILGenerator();
    methodIl.Emit(OpCodes.Ldarg_0);
    methodIl.Emit(OpCodes.Ldfld, msgField);
    methodIl.Emit(OpCodes.Ret);

    // Crée la méthode SayHello
    MethodBuilder sayHiMethod = helloWorldClass.DefineMethod(
        "SayHello",
        MethodAttributes.Public,
        null,
        null
    );

    methodIl = sayHiMethod.GetILGenerator();
    methodIl.EmitWriteLine("Hello from the HelloWorld class!");
    methodIl.Emit(OpCodes.Ret);

    // "Finaliser" la classe HelloWorld.
    // (La finalisation est le terme formel pour émettre le type.)
    helloWorldClass.CreateType();

    return builder;
}
```

## Émission de l'ensemble assembly et module

**Le corps de la méthode commence par définir l'ensemble minimal de caractéristiques de votre assembly, à l'aide des types `AssemblyName` et `Version`** (définis dans l'espace de noms `System.Reflection`). Ensuite, vous obtenez un objet de type `AssemblyBuilder` via la méthode statique `AssemblyBuilder.DefineDynamicAssembly()`. 

Lors de l'appel à `DefineDynamicAssembly()`, vous devez spécifier le mode d'accès de l'assembly à définir; les valeurs les plus courantes sont présentées dans le [[#Tableau 18-9 Valeurs communes de l'énumération `AssemblyBuilderAccess`|Tableau 18-9]].

##### Tableau 18-9: Valeurs communes de l'énumération `AssemblyBuilderAccess`

| Valeur          | Description                                                                                           |
| --------------- | ----------------------------------------------------------------------------------------------------- |
| `RunAndCollect` | L'assembly sera immédiatement déchargé et sa mémoire sera libérée dès qu'il ne sera plus accessible.  |
| `Run`           | Cela signifie qu'un assemblage dynamique peut être exécuté en mémoire mais pas enregistré sur disque. |

L'étape suivante consiste à définir l'ensemble de modules (et son nom) pour votre nouvel assembly. Une fois que la méthode `DefineDynamicModule()` a renvoyé une valeur, vous obtenez une référence à un type `ModuleBuilder` valide.

```cs
// Créer un nouvel assembly.
var builder = AssemblyBuilder.DefineDynamicAssembly(
	assemblyName,
	AssemblyBuilderAccess.Run
);
```

## Rôle du type `ModuleBuilder`

**`ModuleBuilder` est le type clé utilisé lors du développement d'assemblages dynamiques**. Comme on peut s'y attendre, ==`ModuleBuilder` prend en charge plusieurs membres permettant de définir l'ensemble des types contenus dans un module donné== (classes, interfaces, structures, etc.) ==ainsi que l'ensemble des ressources intégrées== (tables de chaînes, images, etc.) ==qu'il contient==. Le [[#Tableau 18-10 Sélection de membre du type `ModuleBuilder`|Tableau 18-10]] décrit deux méthodes de création. (Notez que chaque méthode renvoie un type associé représentant le type que vous souhaitez construire.)

##### Tableau 18-10: Sélection de membre du type `ModuleBuilder`

| Méthode        | Description                                                                                                                         |
| -------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| `DefineEnum()` | Permet de générer une définition d’énumération .NET                                                                                 |
| `DefineType()` | Construit un `TypeBuilder`, qui permet de définir des types valeur, des interfaces et des types de classes (y compris les délégués) |

**La méthode `DefineType()` de la classe `ModuleBuilder` est essentielle**. Outre la spécification du nom du type (sous forme de chaîne de caractères), **==vous utiliserez également l'énumération `System.Reflection.TypeAttributes` pour décrire le format du type lui-même==**. Le [[#Tableau 18-11 Sélection de membre de l'énumération `TypeAttributes`|Tableau 18-11]] répertorie certains (mais pas tous) des principaux membres de l'énumération `TypeAttributes`.

##### Tableau 18-11: Sélection de membre de l'énumération `TypeAttributes`

| Membre              | Description                                                                                                                                                                                        |
| ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Abstract`          | Indique que le type est abstrait                                                                                                                                                                   |
| `Class`             | Spécifie que le type est une classe                                                                                                                                                                |
| `Interface`         | Spécifie que le type est une interface                                                                                                                                                             |
| `NestedAssembly`    | Indique que la classe est imbriquée avec une visibilité d'assemblage et n'est donc accessible que par les méthodes de son assemblage.                                                              |
| `NestedFamANDAssem` | Indique que la classe est imbriquée au niveau de l'assemblage et de la famille, et qu'elle n'est donc accessible que par les méthodes situées à l'intersection de sa famille et de son assemblage. |
| `NestedFamily`      | Indique que la classe est imbriquée avec une visibilité familiale et n'est donc accessible que par les méthodes de son propre type et de ses sous-types.                                           |
| `NestedFamORAssem`  | Indique que la classe est imbriquée au niveau de la famille ou de l'assemblage et n'est donc accessible que par les méthodes appartenant à l'union de sa famille et de son assemblage.             |
| `NestedPrivate`     | Indique que la classe est imbriquée avec une visibilité privée                                                                                                                                     |
| `NestedPublic`      | Indique que la classe est imbriquée avec une visibilité publique.                                                                                                                                  |
| `NotPublic`         | Indique que la classe n'est pas publique                                                                                                                                                           |
| `Public`            | Indique que la classe est publique                                                                                                                                                                 |
| `Sealed`            | Spécifie que la classe est concrète et ne peut pas être étendue.                                                                                                                                   |
| `Serializable`      | Indique que la classe peut être sérialisée.                                                                                                                                                        |

## Émission du type `HelloClass` et de la variable membre `string`

Maintenant que vous comprenez mieux le rôle de la méthode `ModuleBuilder.CreateType()`, examinons
comment émettre le type de classe public `HelloWorld` et la variable `string` privée.

```cs
// Définition d'une classe publique nommée « HelloWorld ».
TypeBuilder helloWorldClass = module.DefineType(
	"MyAssembly.HelloWorld",
	TypeAttributes.Public
);

// Définition d'une variable String privée nommée « theMessage ».
FieldBuilder msgField = helloWorldClass.DefineField(
	"theMessage",
	Type.GetType("System.String"),
	attributes: FieldAttributes.Private
);
```

**Notez comment la méthode `TypeBuilder.DefineField()` donne accès à un type `FieldBuilder`**. ==La classe `TypeBuilder` définit également d'autres méthodes qui donnent accès à d'autres types « constructeurs »==. Par exemple, `DefineConstructor()` renvoie un `ConstructorBuilder`, `DefineProperty()` renvoie un `PropertyBuilder`, et ainsi de suite.

## Émission des constructeurs

Comme mentionné précédemment, la méthode `TypeBuilder.DefineConstructor()` peut être utilisée pour définir un constructeur pour le type courant. Cependant, lors de l'implémentation du constructeur de `HelloClass`, vous devez injecter du code CIL brut dans le corps du constructeur, qui est chargé d'assigner le paramètre entrant à la chaîne privée interne. Pour obtenir un type `ILGenerator`, vous appelez la méthode `GetILGenerator()` à partir du type « constructeur » correspondant auquel vous avez une référence (dans ce cas, le type `ConstructorBuilder`). 

**La méthode `Emit()` de la classe `ILGenerator` est l'entité chargée d'insérer du code CIL dans une implémentation**. ***==`Emit()` utilise fréquemment le type de classe `OpCodes`, qui expose l'ensemble des codes d'opération du CIL à l'aide de champs en lecture seule==***. Par exemple, `OpCodes.Ret` signale le retour d'un appel de méthode, `OpCodes.Stfld` affecte une valeur à une variable membre et `OpCodes.Call` sert à appeler une méthode donnée (ici, le constructeur de la classe de base). Ceci étant dit, examinons la logique du constructeur suivante :

```cs
// Crée le constructeur personnalisé prennant un seul argument string.
Type[] constructorArgs = [typeof(string)];
ConstructorBuilder constructor = helloWorldClass.DefineConstructor(
	MethodAttributes.Public,
	CallingConventions.Standard,
	constructorArgs
);
// Émet le code CIL nécessaire dans le constructeur.
ILGenerator constructorIl = constructor.GetILGenerator();
constructorIl.Emit(OpCodes.Ldarg_0);
Type objectClass = typeof(object);
ConstructorInfo superConstructor = objectClass.GetConstructor(new Type[0]);
constructorIl.Emit(OpCodes.Call, superConstructor);
// Charge ce pointeur dans la pile
constructorIl.Emit(OpCodes.Ldarg_0);
constructorIl.Emit(OpCodes.Ldarg_1);
// Charge l'argiment dans une pile virtuelle and stock dedans msgField.
constructorIl.Emit(OpCodes.Stfld, msgField);
constructorIl.Emit(OpCodes.Ret);
```

Comme vous le savez, **dès que vous définissez un constructeur personnalisé pour un type, le constructeur par défaut est supprimé silencieusement. Pour redéfinir le constructeur sans argument, il suffit d'appeler la méthode `DefineDefaultConstructor()` du type `TypeBuilder` comme suit:**

```cs
// Crée le constructeur par défaut
helloWorldClass.DefineDefaultConstructor(MethodAttributes.Public);
```

## Émission de la méthode `SayHello()`

Enfin, examinons le processus d'émission de la méthode `SayHello()`. La première étape consiste à obtenir un objet de type `MethodBuilder` à partir de la variable `helloWorldClass`. Ensuite, définissez la méthode et obtenez le générateur `ILGenerator` sous-jacent pour injecter les instructions CIL, comme ceci :

```cs
// Crée la méthode SayHello
MethodBuilder sayHiMethod = helloWorldClass.DefineMethod(
	"SayHello",
	MethodAttributes.Public,
	null,
	null
);
methodIl = sayHiMethod.GetILGenerator();

// Écrit à la console.
methodIl.EmitWriteLine("Hello from the HelloWorld class!");
methodIl.Emit(OpCodes.Ret);
```

Vous avez ici défini une méthode publique (`MethodAttributes.Public`) qui ne prend aucun paramètre et ne renvoie rien (indiqué par les valeurs `null` dans l'appel à `DefineMethod()`). Notez également l'appel à `EmitWriteLine()`. Cette fonction auxiliaire de la classe `ILGenerator` écrit automatiquement une ligne sur la sortie standard, en toute simplicité.

## Utilisation de l'assembly généré dynamiquement

Maintenant que la logique de création de votre assembly est en place, il suffit d'exécuter le code généré. Le code appelant appelle la méthode `CreateMyAsm()` et obtient une référence à l'objet `AssemblyBuilder` créé.

Vous allez ensuite utiliser la liaison tardive (voir [[Chapitre 17#Comprendre la liaison tardive|Chapitre 17]]) pour créer une instance de la classe HelloWorld et interagir avec ses membres. Mettez à jour vos instructions de niveau supérieur comme suit :

```cs
using System.Reflection;
using System.Reflection.Emit;

Console.Title = "The Amazing Dynamic Assembly Builder App";
Console.WriteLine("***** The Amazing Dynamic Assembly Builder App *****\n");

// Créez le générateur d'assembly en utilisant notre assistant f(x).
AssemblyBuilder builder = CreateMyAsm();

// Récupère le type HelloWorld.
Type hello = builder.GetType("MyAssembly.HelloWorld");

// Crée une instance de HelloWorld et appelle le bon constructeur.
Console.Write("-> Enter mesage to pass HelloWorld class: ");
string msg = Console.ReadLine();
object[] ctorArgs = [msg];
object obj = Activator.CreateInstance(hello, ctorArgs);

// Appelle SayHello() et affiche le string renvoyé
Console.WriteLine("-> Calling SayHello() via late binding.");
MethodInfo mi = hello.GetMethod("SayHello");
mi.Invoke(obj, null);

// Appelle la méthode GetMsg().
mi = hello.GetMethod("GetMsg");
Console.WriteLine(mi.Invoke(obj, null));
```

**En effet, vous venez de créer un assembly .NET Core capable de créer et d'exécuter des assemblies .NET Core à l'exécution !** Ceci conclut l'étude du CIL et du rôle des assemblies dynamiques. J'espère que ce chapitre a approfondi votre compréhension du système de types .NET Core, de la syntaxe et de la sémantique du CIL, et de la façon dont le compilateur C# traite votre code lors de la compilation.

# Résumé du chapitre

**Ce chapitre a présenté la syntaxe et la sémantique du CIL.** **==Contrairement aux langages managés de haut niveau, tels que C#, le CIL ne se contente pas de définir un ensemble de mots-clés, mais fournit des directives==** (==utilisées pour définir la structure d'un assembly et ses types==), **==des attributs==** (==qui précisent une directive donnée==) **==et des opcodes==** (==utilisés pour implémenter les membres de type==).

**Vous avez découvert quelques outils de programmation spécifiques au CIL et appris à modifier le contenu d'un assembly .NET à l'aide de nouvelles instructions CIL grâce à l'ingénierie aller-retour**. ==Vous avez ensuite appris à définir l'assembly courant== (et référencé), ==les espaces de noms, les types et les membres==. ***==Nous avons conclu par un exemple simple de création d'une bibliothèque de code .NET et d'un exécutable, en utilisant principalement le CIL, des outils en ligne de commande et un peu d'huile de coude.==***

**Enfin, vous avez abordé le processus de création d'un *assembly dynamique***. **==Avec l'espace de noms `System.Reflection.Emit`, il est possible de définir un assembly .NET Core en mémoire lors de l'exécution==**. *==Comme vous l'avez constaté, l'utilisation de cette API exige une connaissance approfondie de la sémantique du code CIL==*. ***==Bien que la création d'assemblies dynamiques ne soit pas une tâche courante pour la plupart des applications .NET Core, elle peut s'avérer utile pour ceux qui doivent développer des outils de support et autres utilitaires de programmation.==***

>[!warning] Comme expliqué au début de la section, on préfère maintenant utiliser les *sources generators* car ils sont compatible avec le Native AoT.

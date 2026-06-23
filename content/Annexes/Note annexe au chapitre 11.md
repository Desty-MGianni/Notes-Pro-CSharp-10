---
title: "Note Annexe au Chapitre 11"
publish: true
---

# L'interface `INumber<T>`

`INumber<T>` fait partie du système de **Generic Math** introduit avec .NET 7 (namespace `System.Numerics`). C'est une interface "composite" qui hérite elle-même de plusieurs interfaces plus petites et spécialisées. Voici sa décomposition :

## Déclaration simplifiée

```cs
public interface INumber<TSelf> : 
    IComparable, 
    IComparable<TSelf>, 
    IEquatable<TSelf>, 
    IComparisonOperators<TSelf, TSelf, bool>, 
    IModulusOperators<TSelf, TSelf, TSelf>, 
    INumberBase<TSelf>
    where TSelf : INumber<TSelf>
```

### 1. Interfaces directement héritées par `INumber<T>`

| Interface                                  | Rôle                                                  |
| ------------------------------------------ | ----------------------------------------------------- |
| `IComparable`                              | Comparaison non générique (`CompareTo(object)`)       |
| `IComparable<TSelf>`                       | Comparaison typée (`CompareTo(TSelf)`)                |
| `IEquatable<TSelf>`                        | Égalité typée (`Equals(TSelf)`)                       |
| `IComparisonOperators<TSelf, TSelf, bool>` | Opérateurs `<`, `>`, `<=`, `>=`                       |
| `IModulusOperators<TSelf, TSelf, TSelf>`   | Opérateur `%` (modulo)                                |
| `INumberBase<TSelf>`                       | Le "cœur" des opérations numériques (voir ci-dessous) |

### 2. Ce qu'apporte `INumberBase<T>` (hérité indirectement)

`INumberBase<TSelf>` regroupe lui-même un grand nombre d'interfaces d'opérateurs :

| Interface                                    | Rôle                                                          |
| -------------------------------------------- | ------------------------------------------------------------- |
| `IEqualityOperators<TSelf, TSelf, bool>`     | Opérateurs `==`, `!=`                                         |
| `IAdditionOperators<TSelf, TSelf, TSelf>`    | Opérateur `+`                                                 |
| `ISubtractionOperators<TSelf, TSelf, TSelf>` | Opérateur `-` (binaire)                                       |
| `IMultiplyOperators<TSelf, TSelf, TSelf>`    | Opérateur `*`                                                 |
| `IDivisionOperators<TSelf, TSelf, TSelf>`    | Opérateur `/`                                                 |
| `IUnaryPlusOperators<TSelf, TSelf>`          | Opérateur unaire `+x`                                         |
| `IUnaryNegationOperators<TSelf, TSelf>`      | Opérateur unaire `-x`                                         |
| `IIncrementOperators<TSelf>`                 | Opérateur `++`                                                |
| `IDecrementOperators<TSelf>`                 | Opérateur `--`                                                |
| `IAdditiveIdentity<TSelf, TSelf>`            | Définit l'élément neutre de l'addition (0)                    |
| `IMultiplicativeIdentity<TSelf, TSelf>`      | Définit l'élément neutre de la multiplication (1)             |
| `IParsable<TSelf>`                           | Méthodes `Parse`/`TryParse` depuis une `string`               |
| `ISpanParsable<TSelf>`                       | Idem mais depuis un `ReadOnlySpan<char>` (perf)               |
| `IFormattable`                               | Méthode `ToString(format, provider)`                          |
| `ISpanFormattable`                           | Formatage direct vers un `Span<char>` (perf, sans allocation) |

## Pourquoi cette architecture ?

L'objectif de .NET 7 était de permettre d'écrire du **code générique mathématique** sans dupliquer de logique pour `int`, `double`, `decimal`, etc. En découpant les capacités en petites interfaces granulaires (addition, comparaison, parsing...), on peut :

- Écrire des méthodes génériques très précises : par exemple `static T Sum<T>(T a, T b) where T : IAdditionOperators<T, T, T>` n'exige que l'addition, pas tout `INumber<T>`.
- Composer `INumber<T>` comme une "façade" pratique qui regroupe tout ce qu'on attend d'un type numérique "classique" (entier ou flottant).

# Hiérarchie complète au-dessus de `INumber<T>`

`INumber<T>` n'est qu'un point intermédiaire dans l'arbre des interfaces de Generic Math. Voici comment ça s'organise plus largement.

## Vue d'ensemble (arbre simplifié)

```
INumberBase<T>
   └── INumber<T>
        ├── IBinaryNumber<T>
        │     ├── IBinaryInteger<T>   (entiers : int, long, BigInteger...)
        │     └── IFloatingPoint<T>   (flottants : float, double, decimal...)
        │           └── IFloatingPointIeee754<T>   (float, double — conformes IEEE 754)
        │
        ├── ISignedNumber<T>     (types signés)
        └── IUnsignedNumber<T>   (types non signés)
```

>[!warning] Précision importante 
>`IBinaryInteger<T>`, `ISignedNumber<T>`, `IUnsignedNumber<T>`, `IFloatingPoint<T>` héritent tous de `INumber<T>`, mais un type donné (ex: `int`) implémente **plusieurs** de ces interfaces en parallèle (pas une seule branche exclusive).

## Détail de chaque interface

| Interface                      | Hérite de                                                                                                                                                                                          | Apporte                                                                                                                          | Implémentée par                                           |
| ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------- |
| **`IBinaryNumber<T>`**         | `INumber<T>`, `IBitwiseOperators<T,T,T>`                                                                                                                                                           | Opérations bit à bit (`&`, `\|`, `^`, `~`), méthode `IsPow2`, `Log2`                                                             | `int`, `double`, etc.                                     |
| **`IBinaryInteger<T>`**        | `IBinaryNumber<T>`, `IShiftOperators<T,int,T>`                                                                                                                                                     | Décalages de bits (`<<`, `>>`, `>>>`), conversions en tableaux d'octets, `DivRem`, `PopCount`, `LeadingZeroCount`                | `int`, `long`, `byte`, `BigInteger`...                    |
| **`IFloatingPoint<T>`**        | `INumber<T>`, `ISignedNumber<T>`                                                                                                                                                                   | Arrondi (`Round`, `Ceiling`, `Floor`, `Truncate`), conversions binaires IEEE                                                     | `float`, `double`, `decimal`, `Half`                      |
| **`IFloatingPointIeee754<T>`** | `IFloatingPoint<T>`, `IExponentialFunctions<T>`, `ITrigonometricFunctions<T>`, `IHyperbolicFunctions<T>`, `ILogarithmicFunctions<T>`, `IPowerFunctions<T>`, `IRootFunctions<T>`, `IMinMaxValue<T>` | Toutes les fonctions mathématiques avancées : `Sin`, `Cos`, `Exp`, `Log`, `Sqrt`, `Pow`, `Epsilon`, `NaN`, `PositiveInfinity`... | `float`, `double` (pas `decimal`, qui n'est pas IEEE 754) |
| **`ISignedNumber<T>`**         | `INumber<T>`                                                                                                                                                                                       | Propriété statique `NegativeOne`                                                                                                 | `int`, `double`, `decimal`...                             |
| **`IUnsignedNumber<T>`**       | `INumber<T>`                                                                                                                                                                                       | Marqueur signalant l'absence de valeurs négatives                                                                                | `uint`, `byte`, `ulong`...                                |

## Exemple concret : à quoi ça sert ?

Le découpage permet d'écrire des fonctions génériques **aussi restrictives ou permissives que nécessaire** :

```cs
// Fonctionne avec n'importe quel nombre (entier OU flottant)
static T Doubler<T>(T x) where T : INumber<T> => x + x;

// Nécessite spécifiquement des fonctions trigonométriques → flottants IEEE754 uniquement
static T SinusDouble<T>(T x) where T : IFloatingPointIeee754<T> => T.Sin(x) * T(2);

// Nécessite des opérations binaires → entiers uniquement
static T DecalerAGauche<T>(T x, int n) where T : IBinaryInteger<T> => x << n;

// Exemple d'usage des fonctions statiques abstraites (membre clé de tout ce système)
static T Maximum<T>(T a, T b) where T : INumber<T> => T.Max(a, b);
```

## Le mécanisme sous-jacent : les "static abstract members"

Tout ce système repose sur une fonctionnalité C# 11 / .NET 7 appelée **static abstract interface members** : une interface peut définir des opérateurs (`+`, `-`, `==`...) ou des méthodes statiques (`T.Sin(x)`, `T.Parse(...)`) que chaque type implémentant doit fournir. C'est ce qui rend possible l'écriture de `a + b` dans une méthode générique `where T : INumber<T>` — chose impossible avant .NET 7.

# l'implémenter `INumber<T>` pour un type de donnée

Pour cette implémentation, j'ai pris la même application console que pour le [[Chapitre 11#Comprendre la surcharge de l'opérateur|Chapitre 11]] avec cette classe `Point`.

```cs
using System.Diagnostics.CodeAnalysis;
using System.Globalization;
using System.Numerics;

namespace OverloadedOps;

public class Point(int xPos, int yPos) : INumber<Point>
{
    // --- L'identitée et les métadonnées ---

    // détermine l'origine (un point à X = 0, Y = 1)
    public static Point Zero => new(0, 0);

    // détermine la valeur de 1 pour ce type (un point à X = 1, Y = 1)
    public static Point One => new(1, 1);

    // Détermine la base mathématique du type. Comme les Propriétés
    // X et Y utilisent des int32, qui sont basé sur le système binaire,
    // on assigne la valeur 2.
    public static int Radix => 2;

    // Synonyme de Zero
    public static Point AdditiveIdentity => Zero;

    // Synonyme de One
    public static Point MultiplicativeIdentity => One;

    // Les deux propriété DOIVENT être déclaré en lecture seule
    // sinon il serait possible de faire ceci :
    // Point.Zero.X = 5;
    public int X { get; } = xPos;
    public int Y { get; } = yPos;

    // --- La plomberie pour la classification ---
    // (Quelle type de nombre c'est ?)

    /*
     * En informatique mathématique, certains nombres peuvent avoir plusieurs
     * représentations en mémoire pour la même valeur.
     *
     * Une représentation est dite "canonique" si elle est
     * la forme standard, unique et optimisée de cette valeur.
     *
     * coordonnées sont des entiers (int). Un entier en mémoire
     * n'a qu'une seule façon d'exister
     * (le chiffre 5 n'a pas deux encodages binaires différents).
     * Vos points sont donc toujours sous leur forme standard.
    */
    public static bool IsCanonical(Point value) => true;

    /*
     * Détermine si le nombre appartient à l'ensemble des nombres complexes
     * (avec une partie réelle et une partie imaginaire \(i\), où \(i^2 = -1\)).
     *
     * On conçoit ici un point de coordonnées réelles (entières).
     * Ce n'est pas un type mathématique complexe au sens algébrique.
     */
    public static bool IsComplexNumber(Point value) => false;

    // Est-ce qu'un Point est un nombre pair ?
    // On peut décider que oui si ses deux coordonnées sont paires.
    public static bool IsEvenInteger(Point value) =>
        value.X % 2 == 0 && value.Y % 2 == 0;

    /*
     * Cette méthode est cruciale pour les calculs.
     * Elle permet de savoir si le nombre est une valeur standard
     * ou s'il a dépassé les bornes au point de devenir une "infinité"
     *
     *  Vos propriétés utilisent des int. En C#, un int qui dépasse sa valeur maximale
     *  (int.MaxValue + 1) ne devient pas "infini", il boucle de façon cyclique vers
     *  les nombres négatifs (overflow).
     *  Vos coordonnées sont donc toujours techniquement "finies"
     */
    public static bool IsFinite(Point value) => true;

    // L'inverse de IsFinite
    public static bool IsInfinity(Point value) => false;

    /*
     * Détermine si la valeur est le résultat
     * d'une opération mathématique impossible ou indéterminée
     * (comme une division de zéro par zéro 0.0 / 0.0 avec des double).
     * Un état "NaN" brise généralement tous les calculs suivants.
     *
     * Un point est toujours composé de deux entiers valides.
     * Il ne peut jamais être dans un état "non-numérique".
     */
    public static bool IsNaN(Point value) => false;

    // On déclare q'un Point(-3, 5) est considéré comme négatif.
    // Cela facilite l'implémetation car c'est l'opposé de IsPositive
    public static bool IsNegative(Point value) => !IsPositive(value);

    // L'inverse de IsEvenInteger
    public static bool IsOddInteger(Point value) => !IsEvenInteger(value);

    // Un Point est-il positif ? Décidons que oui si les deux coordonnées
    // sont supérieurs ou égale à 0.
    public static bool IsPositive(Point value) =>
        value.X >= 0 && value.Y >= 0;

    // Les coordonnée décrivent des positions mesurables et concrètes dans
    // un espace standard, on déclare que le point se comporte comme un
    // nombre réel.
    public static bool IsRealNumber(Point value) => true;

    // Un Point est-il considéré comme "Zéro" ? Oui, s'il est à l'origine.
    public static bool IsZero(Point value) => value.X == 0 && value.Y == 0;

    // Vérifie si le nombre est purement imaginaire
    // Ici, les coordonnées sont des entiers réels.
    public static bool IsImaginaryNumber(Point value) => false;

    /*
     * Détermine si la valeur représente un nombre entier comme 4 ou -12),
     * par opposition à un nombre à virgule (comme 4.5).
     *
     * Les propriétés X et Y utilisent des int.
     * Conceptuellement, votre point est entièrement basé sur des coordonnées
     * entières discrètes (une grille de pixels ou de coordonnées entières).
     * On répond donc true.
     */
    public static bool IsInteger(Point value) => true;

    // L'infini (positif on négatif) est impossible avec le type
    // Int32 utilisé pour les coordonnées.
    public static bool IsNegativeInfinity(Point value) => false;

    // L'infini (positif on négatif) est impossible avec le type
    // Int32 utilisé pour les coordonnées.
    public static bool IsPositiveInfinity(Point value) => false;

    /*
     * Vient du monde de la physique et de la représentation des
     * nombres à virgule flottante (norme IEEE 754).
     *
     * IsNormal signifie que le nombre est stocké sous une forme standard,
     * avec une précision maximale
     * (la immense majorité des nombres comme 1.5, 100.0, etc.).
     *
     * Ce concept n'existent pas pour les entiers.
     * Cependant, pour ne pas bloquer le framework, on considère qu'un point
     * valide est dans un état "normal"
     * (sauf s'il est égal à zéro, car par définition mathématique,
     * zéro n'est pas un nombre normalisé).
     */
    public static bool IsNormal(Point value) => !IsZero(value);

    /*
     * Vient du monde de la physique et de la représentation des
     * nombres à virgule flottante (norme IEEE 754).
     *
     * IsSubnormal (ou dénormalisé) signifie que le nombre est
     * si proche de zéro que l'ordinateur a dû sacrifier
     * de la précision pour réussir à le représenter.
     *
     * Ce concept n'existent pas pour les entiers.
     * Cependant, pour ne pas bloquer le framework, on considère qu'un point
     * valide est dans un état "normal"
     * (sauf s'il est égal à zéro, car par définition mathématique,
     * zéro n'est pas un nombre normalisé).
     */
    public static bool IsSubnormal(Point value) => false;

    // --- Les opération de Magnitude ---
    //
    // Renvois la valeur absolue de la donnée.
    // Si on passe un Point(-5, 10), on retourne un Point(5, 10)
    public static Point Abs(Point value) =>
        new(Math.Abs(value.X), Math.Abs(value.Y));

    /*
     * Méthode utilitaire interne pour déterminer la plus grande ou
     * la plus petite magnitudes.
     *
     * C'est-à-dire la valeur absolue,
     * la distance par rapport à zéro, indépendamment du signe.
     *
     * (Théroème de Pythagore sans racine carrés)
     *
     * Note : On renvois un long pour évitér l'overflow.
     */
    private static long GetMagnitudeSquared(Point value) =>
        (value.X * value.X) + (value.Y * value.Y);

    /*
     * Comparent la magnitude de X et Y et retournent le point gagnant.
     * Si les deux points ont exactement la même magnitude,
     * les spécifications de .NET imposent de retourner X (le premier argument).
     
     * MaxMagniture Retourne le point le plus éloigné de l'origine.
     */
    public static Point MaxMagnitude(Point x, Point y)
    {
        int magX = (int)GetMagnitudeSquared(x);
        int magY = (int)GetMagnitudeSquared(y);

        // En cas d'égalité, .NET exige de retourner x
        return (magX >= magY) ? x : y;
    }

    /* Ces méthodes ont été introduites dans .NET pour gérer le cas
     * spécifique des nombres "NaN" (Not a Number)
     *
     * Dans une comparaison standard (MaxMagnitude),
     * si l'un des paramètres est un NaN,
     * la méthode doit retourner NaN.
     *
     * Dans la variante Number (MaxMagnitudeNumber),
     * la méthode doit ignorer le NaN et retourner
     * l'autre paramètre s'il est valide.
     *
     * Comme nous avons défini précédemment que notre Point
     * ne peut jamais être un NaN (IsNaN retourne toujours false),
     * le comportement de MaxMagnitudeNumber sera strictement identique à MaxMagnitude.
     *
     * Vous pouvez donc simplement faire appel à MagMagnitude
     */
    public static Point MaxMagnitudeNumber(Point x, Point y) =>
        MaxMagnitude(x, y);

    // Même explication que pour la méthode MaxMagnitude.
    // MinMagnitude retourne le point le plus proche de l'origine.
    public static Point MinMagnitude(Point x, Point y)
    {
        int magX = (int)GetMagnitudeSquared(x);
        int magY = (int)GetMagnitudeSquared(y);

        // En cas d'égalité, .NET exige de retourner x
        return (magX <= magY) ? x : y;
    }

    // Même explication que pour la méthode MaxMagnitudeNumber
    public static Point MinMagnitudeNumber(Point x, Point y) =>
        MinMagnitude(x, y);

    // --- Plomberie de conversions textuelle (Parse et TryParse)

    /* Le format textuel pour un Point est (X,Y)
     *
     * En C# moderne (depuis C# 11 et .NET 7),
     * on implémente la logique lourde sur ReadOnlySpan<char>
     * pour éviter les allocations de mémoire inutiles,
     * puis les autres méthodes réutilisent celle-ci.
     *
     * Le but est de nettoyer les parenthèses, de découper au niveau de la virgule, et de convertir les deux morceaux en int.
     */
    public static Point Parse(
        ReadOnlySpan<char> s,
        NumberStyles style,
        IFormatProvider? provider
    )
    {
        // 1. Nettoyer les espaces au début et à la fin
        ReadOnlySpan<char> trimmed = s.Trim();

        // 2. Vérifier que la chaîne commence par '(' et termine par ')'
        if (trimmed.Length < 5 || trimmed[0] != '(' || trimmed[^1] != ')')
        {
            throw new FormatException(
                "The string format of a Point has to be \"(X,Y)\""
            );
        }

        // 3. Extraire le contenu sans les parenthèses.
        ReadOnlySpan<char> content = trimmed[1..^1];

        // 4. Trouver la position de la virgule séparatrice
        int commaIndex = content.IndexOf(',');

        if (commaIndex == -1)
        {
            throw new FormatException(
                "The string format of a Point has to have a comma separator : \"(X,Y)\""
            );
        }

        // 5. Découper X et Y
        ReadOnlySpan<char> xSpan = content[..commaIndex].Trim();
        ReadOnlySpan<char> ySpan = content[(commaIndex + 1)..].Trim();

        // 6. Parser ls deux coodonnées en utilisant le style et le provider fournis
        if (
            !int.TryParse(xSpan, style, provider, out int x)
            || !int.TryParse(ySpan, style, provider, out int y)
        )
        {
            throw new FormatException(
                "Coordinate X and Y have to be valid integers"
            );
        }

        return new Point(x, y);
    }

    // La classe string se convertit implicitement en REadOnlySpan<char>.
    // On appelle donc la première méthode en lui passant .AsSpan().
    public static Point Parse(
        string s,
        NumberStyles style,
        IFormatProvider? provider
    )
    {
        ArgumentNullException.ThrowIfNull(s);
        return Parse(s.AsSpan(), style, provider);
    }

    // Cette méthode n'impose pas de NumberStyles.
    // On appelle donc la première méthode en lui passant
    // un style par défaut pour les entiers : NumberStyles.Integer
    public static Point Parse(
        ReadOnlySpan<char> s,
        IFormatProvider? provider
    ) => Parse(s, NumberStyles.Integer, provider);

    // On passe AsSpan et NumberStyles.Integer
    public static Point Parse(string s, IFormatProvider? provider)
    {
        ArgumentNullException.ThrowIfNull(s);
        return Parse(s.AsSpan(), NumberStyles.Integer, provider);
    }

    /* La logique interne est quasiment la même, avec les différences
     * de comportement vu dans le livre :
     *
     * 1) Ne peut pas lever d'exception (renvois true/false)
     * 2) Envois une valeur par défaut via un paramètre out.
     *
     * [MaybeNullWhen(false)]
     * "Si la méthode renvoie false, la variable result en sortie pourrait être nulle
     * (ou contenir la valeur par défaut du type)."
     */
    public static bool TryParse(
        ReadOnlySpan<char> s,
        NumberStyles style,
        IFormatProvider? provider,
        [MaybeNullWhen(false)] out Point result
    )
    {
        // 1. Nettoyer les espaces
        ReadOnlySpan<char> trimmed = s.Trim();

        // 2. Vérifier la longueur minimale et les parenthèses.
        if (trimmed.Length < 5 || trimmed[0] != '(' || trimmed[^1] != ')')
        {
            result = Zero; // Utilise votre propriété static Zero
            return false;
        }

        // 3. Extraire le contenu intérieur
        ReadOnlySpan<char> content = trimmed[1..^1];

        // 4. Trouver la virgule
        int commaIndex = content.IndexOf(',');
        if (commaIndex == -1)
        {
            result = Zero;
            return false;
        }

        // 5. Découper X et Y
        ReadOnlySpan<char> xSpan = content[..commaIndex].Trim();
        ReadOnlySpan<char> ySpan = content[(commaIndex + 1)..].Trim();

        // 6. Tenter de parser les deux entiers sans lever d'exception
        if (
            !int.TryParse(xSpan, style, provider, out int x)
            || !int.TryParse(ySpan, style, provider, out int y)
        )
        {
            result = Zero;
            return false;
        }

        // Tout est bon, on crée le point et on retourne true
        result = new Point(x, y);
        return true;
    }

    /*
     * La même chose que pour les méthode Parse,
     * ici, on utilise AsSpan().
     *
     * [NotNullWhen(true)] string? s
     * "Si cette méthode renvoie true, je te garantis à 100 % que la variable s
     * qui a été passée en paramètre n'était pas nulle."
     *
     * [MaybeNullkWhen(false)] aut Point result
     * "Si la méthode renvoie false, la variable result en sortie pourrait être nulle
     * (ou contenir la valeur par défaut du type)."
     */
    public static bool TryParse(
        [NotNullWhen(true)] string? s,
        NumberStyles style,
        IFormatProvider? provider,
        [MaybeNullWhen(false)] out Point result
    )
    {
        if (s == null)
        {
            result = Zero;
            return false;
        }
        return TryParse(s.AsSpan(), style, provider, out result);
    }

    /*
     * La même chose que pour les méthode Parse,
     * ici, on utilise NumberStyles.Integer
     */
    public static bool TryParse(
        ReadOnlySpan<char> s,
        IFormatProvider? provider,
        [MaybeNullWhen(false)] out Point result
    ) => TryParse(s, NumberStyles.Integer, provider, out result);

    // La même chose que pour les méhtode Parse,
    // ici, on utilise AsSpan() et NumberStyles.Integer
    public static bool TryParse(
        [NotNullWhen(true)] string? s,
        IFormatProvider? provider,
        [MaybeNullWhen(false)] out Point result
    )
    {
        if (s == null)
        {
            result = Zero;
            return false;
        }
        return TryParse(
            s.AsSpan(),
            NumberStyles.Integer,
            provider,
            out result
        );
    }

    // --- La plomberie pour le formattage ---
    /*
     *  La méthode TryFormat est l'inverse exact de TryParse.
     *  Le framework donne à votre classe un espace mémoire de caractères
     *  (Span<char> destination) et le point doit écrire sa
     *  représentation textuelle (X,Y) à l'intérieur sans
     *  allouer de nouvelle chaîne de caractères.
     */
    public bool TryFormat(
        Span<char> destination,
        out int charsWritten,
        ReadOnlySpan<char> format,
        IFormatProvider? provider
    )
    {
        // On crée la chaîne au format standard (X,Y)
        // Note : Pour être ultra-performant sans allocation,
        // .NET permet d'utiliser un interpolateur de Span,
        //return destination.TryWrite(provider, $"({X},{Y})", out charsWritten);

        // mais pour votre défi du Chapitre 11,
        // cette syntaxe simple et moderne est parfaitement acceptée :
        string text = $"({X},{Y})";

        // Vérifie si le Span de destination est assez grand
        // pour accueillir le texte.
        if (destination.Length < text.Length)
        {
            charsWritten = 0;
            return false;
        }

        // Copie les caractère directementr dans la mémoire de destination
        text.AsSpan().CopyTo(destination);
        charsWritten = text.Length;
        return true;
    }

    public override string ToString() => $"({X},{Y})";

    /*
     * L'interface INumber<T> hérite de IFormattable,
     * ce qui vous oblige à fournir une méthode ToString avancée.
     *
     * La règle absolue en .NET moderne pour implémenter cette méthode
     * proprement et sans dupliquer de code est de réutiliser la méthode TryFormat
     * que nous venons d'écrire.
     */
    public string ToString(string? format, IFormatProvider? formatProvider)
    {
        // On prépare un espace suffisant pour stocker "(X,Y)"
        // avec deux int32 (environ 30 caractères max)
        Span<char> buffer = stackalloc char[64];

        // On demande à notre méthode performante TryFormat de faire le travail
        if (
            TryFormat(buffer, out int charsWritten, format, formatProvider)
        )
            return new string(buffer[..charsWritten]);
        return $"({X},{Y})";
    }

    // --- Plomberie de conversions génériques ---

    /* Un Point possède deux composants (X et Y),
     * alors qu'un int ou un double n'est qu'un seul nombre scalaire.
     *
     * Il faut fixer une règle arbitraire :
     *
     * - De TOther vers Point :
     *      On applique la valeur de TOther sur X et sur Y
     *      en même temps (un nombre 5 devient un Point(5, 5)).
     *
     * - De Point vers TOther :
     *      On décide d'exporter uniquement la coordonnée X
     *      (ou la magnitude, mais X est le plus simple à coder et respecte
     *      les contraintes d'identités mathématiques comme Zero ou One).
     */

    /*
     * Si la valeur dépasse la capacité maximale ou minimale
     * du type de destination, la conversion doit échouer proprement
     * (renvoyer false ici, ou lever une OverflowException
     * dans les versions non-Try).
     */
    public static bool TryConvertFromChecked<TOther>(
        TOther value,
        [MaybeNullWhen(false)] out Point result
    )
        where TOther : INumberBase<TOther>
    {
        try
        {
            // Tente d'extraire la valeur sous forme d'int en mode véfirié
            // (lève une exception si overflow)
            int target = int.CreateChecked(value);
            result = new(target, target);
            return true;
        }
        catch
        {
            result = Zero;
            return false;
        }
    }

    /*
     * Si la valeur dépasse les bornes, on la "bloque"
     * au maximum ou au minimum autorisé par le type de destination
     * (comme un effet de clipping).
     * Par exemple, tenter de convertir 300 en byte (max 255) donnera 255.
     */
    public static bool TryConvertFromSaturating<TOther>(
        TOther value,
        [MaybeNullWhen(false)] out Point result
    )
        where TOther : INumberBase<TOther>
    {
        // Sature automatiquement aux limites d'int int
        // (int.MinValue / int.MaxValue)
        int target = int.CreateSaturating(value);
        result = new(target, target);
        return true;
    }

    /*
     * Si la valeur dépasse, on coupe les bits excédentaires
     * (comportement par défaut des entiers en C#)
     * ou on ignore la partie fractionnaire sans arrondir
     * si on va vers un entier.
     * Tenter de convertir 300 en byte donnera 44 (car 300 % 256 = 44).
     */
    public static bool TryConvertFromTruncating<TOther>(
        TOther value,
        [MaybeNullWhen(false)] out Point result
    )
        where TOther : INumberBase<TOther>
    {
        // Tronque les bits si la valeur est trop grande
        int target = int.CreateTruncating(value);
        result = new(target, target);
        return true;
    }

    // On tente d'exporter la coordonnée X vers
    // le type TOther de manière sécurisée
    public static bool TryConvertToChecked<TOther>(
        Point value,
        [MaybeNullWhen(false)] out TOther result
    )
        where TOther : INumberBase<TOther>
    {
        try
        {
            result = TOther.CreateChecked(value.X);
            return true;
        }
        catch
        {
            result = default;
            return false;
        }
    }

    // Sature automatiquement aux limites d'int int
    // (int.MinValue / int.MaxValue)
    public static bool TryConvertToSaturating<TOther>(
        Point value,
        [MaybeNullWhen(false)] out TOther result
    )
        where TOther : INumberBase<TOther>
    {
        result = TOther.CreateSaturating(value.X);
        return true;
    }

    // Tronque les bits si la valeur est trop grande
    public static bool TryConvertToTruncating<TOther>(
        Point value,
        [MaybeNullWhen(false)] out TOther result
    )
        where TOther : INumberBase<TOther>
    {
        result = TOther.CreateTruncating(value.X);
        return true;
    }

    // --- Plomberie pour la comparaison et l'égalité ---
    // (pas lié aux méthodes MaxMagniture et MinMagnitude)

    public int CompareTo(object? obj)
    {
        if (obj is null)
            return 1; // Guidelines Microsoft
        if (obj is Point otherPoint)
        {
            return CompareTo(otherPoint);
        }
        throw new ArgumentException(
            "Argument obj has to be of type Point"
        );
    }

    public int CompareTo(Point? other)
    {
        if (other is null)
            return 1; // Guidelines Microsoft

        int thisMag = (X * X) + (Y * Y);
        int otherMag = (other.X * other.X) + (other.Y * other.Y);

        if (thisMag > otherMag)
            return 1;
        else if (thisMag < otherMag)
            return -1;
        return 0;
    }

    public bool Equals(Point? other)
    {
        if (other is null)
            return false;
        return X == other.X && Y == other.Y;
    }

    public override bool Equals(object? obj) => Equals(obj as Point);

    // La manière standard de combiner plusieurs champs/propriété
    // en un seul code de hashage.
    public override int GetHashCode() => HashCode.Combine(X, Y);

    // --- Les surchages des opérateurs ---

    // Opérateur unaire +
    // comme un Point est représenté par deux entiers,
    // on peut juste renvoyé value.
    public static Point operator +(Point value) => value;

    public static Point operator +(Point left, Point right) =>
        new(left.X + right.X, left.Y + right.Y);

    // opérateur unaire -
    // les deux coordonnée deviennent négatives
    public static Point operator -(Point value) => new(-value.X, -value.Y);

    public static Point operator -(Point left, Point right) =>
        new(left.X - right.X, left.Y - right.Y);

    public static Point operator ++(Point value) =>
        new(value.X + 1, value.Y + 1);

    public static Point operator --(Point value) =>
        new(value.X - 1, value.Y - 1);

    public static Point operator *(Point left, Point right) =>
        new(left.X * right.X, left.Y * right.Y);

    public static Point operator /(Point left, Point right) =>
        new(left.X / right.X, left.Y / right.Y);

    public static Point operator %(Point left, Point right) =>
        new(left.X % right.X, left.Y % right.Y);

    public static bool operator ==(Point? left, Point? right)
    {
        if (ReferenceEquals(left, right))
            return true;
        // is null est important !
        // si on utilise == null, une
        // boucle infinie occurent.
        if (left is null || right is null)
            return false;
        return left.Equals(right);
    }

    public static bool operator !=(Point? left, Point? right) =>
        !(left == right);

    public static bool operator <(Point left, Point right) =>
        left.CompareTo(right) < 0;

    public static bool operator >(Point left, Point right) =>
        left.CompareTo(right) > 0;

    public static bool operator <=(Point left, Point right) =>
        left.CompareTo(right) <= 0;

    public static bool operator >=(Point left, Point right) =>
        left.CompareTo(right) >= 0;
}
```


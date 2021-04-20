---
type: "post"
title: "Découverte de Kotlin par la pratique"
date: 2016-05-08 14:58:45 +0200
description: ""
author: Nadia Humbert-Labeaumaz
image : "images/blog/kotlin/header.png"
bg_image: "images/blog/kotlin/header.png"
categories: ["Software"]
tags: ["Kotlin"]
draft: false
comments: true
---

Kotlin est un langage de programmation créé par JetBrains. Il est exécuté sur la JVM et est 100% interopérable avec Java 1.6+.
J'ai découvert récemment ce langage et j'ai réalisé un ensemble de katas pour le pratiquer.
Le code source de ces katas est disponible sur GitHub :

- [Prime factors](https://github.com/nphumbert/kata-prime-factors)
- [String calculator](https://github.com/nphumbert/kata-string-calculator)
- [Game of life](https://github.com/nphumbert/kata-game-of-life)
- [Mars rover](https://github.com/nphumbert/kata-mars-rover)

Dans cet article, je vais présenter des particularités de Kotlin que j'ai pu rencontrer.

<!-- more -->

## Généralités

En Kotlin, un fichier peut contenir plusieurs classes et des fonctions. Ces fonctions peuvent exister en dehors d'une classe.
Un exemple de classe est donné ci-dessous :

```kotlin
data class Dimension(val width: Int, val height: Int)
```

Le constructeur est défini dans la déclaration de la classe. Le mot clé `val` signifie que le champ est en lecture seule. Il faudrait utiliser `var` pour indiquer qu'il est modifiable. De plus, des getters sont implicitement générés pour tous les champs qui ne sont pas `private` ainsi que des setters pour les champs qui ne sont pas en lecture seule. Par ailleurs, le mot clé `data` permet de créer automatiquement les méthodes `toString`, `equals`, `hashCode` et `clone`.

## Gestion des null

Par défaut, les variables ne peuvent pas être nulles. Pour indiquer qu'une variable peut être nulle, il faut ajouter un `?` à la fin du type. Par exemple, `val position: Position?`. Le fait d'appeler une méthode sur une variable qui peut être nulle génère une erreur de compilation si ce n'est pas géré explicitement.

Dans le code suivant, `moves` est une `Map` dont une valeur est récupérée à partir d'une clé. Ceci peut retourner une valeur nulle si elle ne contient pas cette clé. S'il est certain que la clé existe dans la `Map`, il faut forcer l'appel avec `!!`.

```kotlin
override fun apply(position: Position): Position {
    return moves[position.orientation]!!.invoke(position)
}
```

S'il n'est pas certain que la clé existe dans la `Map`, il est possible d'utiliser `?` pour appeler la méthode `invoke` sans risque d'exception. En effet, si la valeur est nulle, la méthode ne sera pas appelée et le résultat sera `null`. Le code deviendrait donc le suivant :

```kotlin
override fun apply(position: Position): Position? {
    return moves[position.orientation]?.invoke(position)
}
```

Cette gestion des `null` permet d'éviter les contrôles de nullité dispersés dans tout le code. De plus, elle permet de rendre très explicite les endroits où une variable peut être nulle. Finalement, tout ceci est vérifié par le compilateur, ce qui permet d'éviter les oublis.

## Valeurs par défaut

Kotlin permet de donner aux paramètres des fonctions une valeur par défaut. Cette valeur est celle utilisée si le paramètre n'est pas fourni lors de l'appel de la fonction. La syntaxe pour mettre une valeur par défaut est décrite ci-après :

```kotlin
fun add(numbers: String, separator: Char = ','): Int {
    return numbers.split(separator).map { it.toInt() }.sum()
}
```

Cette fonction peut donc être appelée de deux manières différentes :

```kotlin
add(numbers.joinToString(","))
add(numbers.joinToString(","), ',')
```

Ce mécanisme permet de modifier du code existant, pour le rendre paramétrable par exemple, sans devoir changer le code client déjà existant ni devoir créer une nouvelle fonction qui se fera déléguer son traitement.

## Streams

L'équivalent des `Streams` Java 8 sont gérés nativement par les classes Kotlin (même avec un JDK 1.6 !). Il est donc directement possible d'appeler des méthodes comme `filter`, `map` ou `reduce` sur des collections. Par exemple, dans le jeu de la vie, pour compter toutes les cellules vivantes parmi les cellules voisines, il est possible de faire ceci :

```kotlin
fun numberOfLiveNeighbours(position: Position): Int =
                      position.neighbours().filter { get(it).alive }.count()
```

Par ailleurs, d'autres méthodes plus spécifiques sont disponibles selon le type de collection. Par exemple, sur une
`Map`, il est possible d'appliquer des filtres seulement sur les valeurs grâce à la méthode `filterValues` :

```kotlin
fun toString(orientation: Orientation): String =
                      orientations.filterValues { it.equals(orientation) }.keys.first().toString()
```

Le grand nombre de méthodes de ce genre facilite la programmation fonctionnelle et rend le code plus clair et expressif. De plus, le fait que Kotlin soit intéropérable avec Java permet d'introduire des morceaux de code fonctionnel dans une base de code Java 6 ou 7.

## Fonctions d'extension

Soit le code Kotlin suivant extrait du kata des facteurs premiers :

```kotlin
primes.forEach {
    while (remains % it == 0) {
        factors.add(it)
        remains /= it
    }
}
```

La boucle ci-dessus continue tant que `remains` est un multiple du nombre premier courant. Il serait possible de rendre le code plus expressif de la manière suivante :

```kotlin
primes.forEach {
    while (remains.isMultipleOf(it)) {
        factors.add(it)
        remains /= it
    }
}
```

Le problème est que `remains` est un `Int` et que cette classe ne contient pas la méthode `isMultipleOf` par défaut. Cependant, il est possible de l'ajouter grâce à une fonction d'extension. il s'agit de créer une fonction et de l'ajouter à une classe de manière externe :

```kotlin
fun Int.isMultipleOf(number: Int): Boolean = this.mod(number) == 0
```

Les fonctions d'extension sont très pratiques pour enrichir des objets, sans devoir modifier leur classe. De plus, ces extensions peuvent être restreintes à un certain contexte (une classe, un package, etc.).

## La déstructuration

Lorsque l'on souhaite extraire des objets depuis un tableau ou une liste, il est possible de s'affranchir de l'utilisation explicite des index en utilisant la déstructuration :

```kotlin
val (gridDimensionInput, initialPositionInput, pathInput) = input.split("\n")
```

Dans l'exemple ci-dessus, l'input va être découpée par ligne. La première ligne sera stockée dans `gridDimensionInput`, la seconde dans `initialPositionInput` et la troisième dans `pathInput`.

Ceci est également possible avec une `data class` de la façon suivante :

```kotlin
val position = Position(1, 2, Orientation.NORTH)
val (x, y, orientation) = position
```

La déstructuration permet de simplifier l'extraction de données depuis un objet qui la supporte sans devoir créer des variables intermédiaires qui ne serviront plus ensuite.

## Création de type implicite

Quand un champ, un paramètre ou une variable doit être une fonction, sa signature peut être spécifiée lors de sa déclaration. Dans le kata Mars rover, une `Map` est utilisée pour associer à une orientation particulière du rover, une fonction qui va le faire avancer. Cette fonction doit prendre en paramètre une position et doit retourner la nouvelle position :

```kotlin
val moves = HashMap<Orientation, (Position) -> Position>()
moves.put(Orientation.NORTH, { Position(it.x, it.y.plus(1), it.orientation) })
```

Cette manière de déclarer les types donne une très grande liberté sur la forme souhaitée pour le type. En effet, il n'est pas nécessaire de créer un type explicitement pour l'utiliser.

## Conclusion

Kotlin est un langage avec des opinions fortes. Je trouve que ceci le rend clair et très agréable à utiliser. Il force à se poser les bonnes questions au bon moment, pour la gestion des null par exemple. Ces questions étant résolues au moment approprié, il y a moins de soucis à se faire le reste du temps. De plus, ce langage est simple à écrire et à lire car il est peu verbeux et offre une grande expressivité. Par ailleurs, son intéropérabilité avec Java permet de l'introduire dans des bases de code existantes sans devoir les modifier.

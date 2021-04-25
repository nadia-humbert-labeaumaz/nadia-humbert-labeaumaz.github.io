---
type: "post"
title: "Polymorphisme avec le kata parrot refactoring"
slug: "polymorphisme-avec-le-kata-parrot-refactoring"
date: 2016-04-20 16:26:03 +0200
description: ""
author: Nadia Humbert-Labeaumaz
image : "images/blog/kata-parrot/header.jpg"
bg_image: "images/blog/kata-parrot/header.jpg"
categories: ["Software"]
tags: ["Refactoring"]
draft: false
comments: true
---

Ce kata est tiré d'un exemple du livre "Refactoring, Improving the Design of Existing Code" de Martin Fowler, et a été créé par Emilie Bache.
L'exemple contient des signes de mauvais design et permet notamment de pratiquer le polymorphisme.  
Dans cet article, une solution à ce kata sera développée. Le projet qui a servi de support se trouve sur [GitHub](https://github.com/nphumbert/kata-parrot-refactoring) avec la solution respective.

<!--more-->

## Contexte

Il s'agit d'un projet où l'on calcule la vitesse d'un perroquet en fonction de son origine. La classe `Parrot` initiale est la suivante :

```java
public class Parrot {

    private ParrotTypeEnum type;
    private int numberOfCoconuts = 0;
    private double voltage;
    private boolean isNailed;

    public Parrot(ParrotTypeEnum _type, int numberOfCoconuts, double voltage, boolean isNailed) {
        this.type = _type;
        this.numberOfCoconuts = numberOfCoconuts;
        this.voltage = voltage;
        this.isNailed = isNailed;
    }

    public double getSpeed() {
        switch (type) {
            case EUROPEAN:
                return getBaseSpeed();
            case AFRICAN:
                return Math.max(0, getBaseSpeed() - getLoadFactor() * numberOfCoconuts);
            case NORWEGIAN_BLUE:
                return (isNailed) ? 0 : getBaseSpeed(voltage);
        }
        throw new RuntimeException("Should be unreachable");
    }

    private double getBaseSpeed(double voltage) {
        return Math.min(24.0, voltage * getBaseSpeed());
    }

    private double getLoadFactor() {
        return 9.0;
    }

    private double getBaseSpeed() {
        return 12.0;
    }
}
```

Cette classe utilise une structure conditionnelle dans la méthode `getSpeed` permettant de calculer la vitesse du perroquet. Ceci n'est pas convenable car il faudrait ajouter une nouvelle structure de ce type à chaque fois qu'un comportement dépend du type de perroquet. De plus, il faudrait ajouter une clause à chacune de ces structures dès qu'un nouveau type de perroquet est ajouté. Finalement, si la manière de calculer la vitesse d'un des types de perroquet changeait, il faudrait modifier cette classe. Cela implique que cette classe a plusieurs raisons de changer : elle ne respecte donc pas le [Single Responsibility Principle](https://www.youtube.com/watch?v=-mroBlWUBBM).

## Lecture des tests et début du refactoring

Je commence par lire les tests existants. Je remarque qu'il y a des tests associés à chaque type de perroquet.
Je décide de commencer par traiter les cas concernant le perroquet européen. Il y a un seul test associé à ce perroquet :

```java
@Test
public void getSpeedOfEuropeanParrot() {
    Parrot parrot = new Parrot(ParrotTypeEnum.EUROPEAN, 0, 0, false);
    assertEquals(parrot.getSpeed(), 12.0);
}
```

Je lance le harnais de test pour m'assurer que tous les tests sont verts et que je pars donc d'une bonne base.

Je remplace dans ce test la classe `Parrot` par la classe fille `EuropeanParrot`. Le constructeur de cette nouvelle classe n'a aucun paramètre. En effet, les valeurs passées au constructeur de la classe mère sont toutes à des valeurs par défaut.  
Comme il n'y a qu'un seul test associé à ce perroquet, il est déduit que la vitesse de ce perroquet n'est affectée par aucun paramètre externe. L'implémentation vient confirmer cette affirmation et qu'il ne s'agit pas d'un oubli de test.

Dans la nouvelle classe, j'appelle le contructeur de la classe mère et je surcharge la méthode `getSpeed` :
```java
public class EuropeanParrot extends Parrot {

    public EuropeanParrot() {
        super(ParrotTypeEnum.EUROPEAN, 0, 0, false);
    }

    @Override
    public double getSpeed() {
        return getBaseSpeed();
    }

}
```
La méthode `getBaseSpeed()` est toujours celle de la classe mère avec une nouvelle visibilité : `protected`. En effet, cette méthode est utilisée lors du calcul de la vitesse de tous les types de perroquet.

De plus, je remplace l'implémentation dans la classe mère par un déclenchement de l'exception `IllegalStateException`. Ainsi, je peux être certaine qu'il s'agit de la nouvelle implémentation qui est utilisée.

Je lance le harnais de test pour vérifier que mon refactoring n'a pas engendré de régression.

Finalement, je déplace mon test dans une nouvelle classe de test, associée à la nouvelle classe `EuropeanParrot`. J'en profite pour renommer le test avec un nom qui explicite le métier. Le nouveau test est le suivant :

```java
@Test
public void should_have_speed_equal_to_base_speed() {
    // given
    Parrot parrot = new EuropeanParrot();

    // when
    double speed = parrot.getSpeed();

    // then
    assertEquals(speed, 12.0);
}
```

Je relance les tests pour m'assurer qu'ils sont toujours verts.

## Extraction des perroquets dans des sous-classes

Comme pour le perroquet européen, j'extrait les deux autres perroquets (africain et norvégien) dans des sous-classes. J'itère les mêmes opérations que précédemment.

J'ai pu constater que le facteur externe affectant la vitesse du perroquet africain est le nombre de noix de coco qu'il porte. Ce facteur n'affectant que ce type de perroquet, il est supprimé de la classe mère et déplacé dans cette classe spécifique.
La classe `AfricanParrot` est la suivante :

```java
public class AfricanParrot extends Parrot {

    private final int numberOfCoconuts;

    public AfricanParrot(int numberOfCoconuts) {
        super(ParrotTypeEnum.AFRICAN, 0, false);
        this.numberOfCoconuts = numberOfCoconuts;
    }

    @Override
    public double getSpeed() {
        return Math.max(0, getBaseSpeed() - getLoadFactor() * numberOfCoconuts);
    }

    private double getLoadFactor() {
        return 9.0;
    }
}
```

De même, les facteurs affectant la vitesse du perroquet norvégien sont le voltage et le fait d'avoir été abbatu. Ces facteurs n'affectant que ce type de perroquet, ils sont supprimés de la classe mère et utilisés dans cette classe fille. La classe `NorwegianParrot` est la suivante :

```java
public class NorwegianParrot extends Parrot {

    private final boolean isNailed;
    private final double voltage;

    public NorwegianParrot(double voltage, boolean isNailed) {
        super(ParrotTypeEnum.NORWEGIAN_BLUE);
        this.voltage = voltage;
        this.isNailed = isNailed;
    }

    @Override
    public double getSpeed() {
        return (isNailed) ? 0 : getBaseSpeed(voltage);
    }

    private double getBaseSpeed(double voltage) {
        return Math.min(24.0, voltage * getBaseSpeed());
    }
}
```

## Abstraction de la classe mère

Après ces étapes d'extraction, la classe `Parrot` est comme suit :

```java
public class Parrot {

    private ParrotTypeEnum type;

    public Parrot(ParrotTypeEnum _type) {
        this.type = _type;
    }

    public double getSpeed() {
        switch (type) {
            case EUROPEAN:
                throw new IllegalStateException("Should be overridden");
            case AFRICAN:
                throw new IllegalStateException("Should be overridden");
            case NORWEGIAN_BLUE:
                throw new IllegalStateException("Should be overridden");
        }
        throw new RuntimeException("Should be unreachable");
    }

    protected double getBaseSpeed() {
        return 12.0;
    }

}
```

La méthode `getSpeed` n'étant plus utilisée, je la rends abstraite, ainsi que la classe.  
Je relance le harnais de test pour vérifier qu'il n'y a pas eu de régression.

De plus, je supprime l'énumération `ParrotTypeEnum` portant sur le type de perroquet car elle n'a plus lieu d'être. Par conséquent, les appels du constructeur de la super classe dans les classes filles disparaissent également. Je relance les tests pour vérifier qu'ils passent encore.  
La classe `Parrot` devient :

```java
public abstract class Parrot {

    public abstract double computeSpeed();

    protected double getBaseSpeed() {
        return 12.0;
    }

}
```

## Conclusion

Ce kata permet de remplacer une structure conditionnelle par du polymorphisme. Chacune des classes n'a plus qu'une seule raison de changer. De plus, pour ajouter un nouveau type de perroquet, il suffit de créer une nouvelle classe qui hérite de la classe `Parrot`, sans avoir à modifier quoi que ce soit par ailleurs. Le code est donc plus SOLID.
Un autre exemple de ce type de refactoring a été développé sur une des vidéos de ma chaîne YouTube [Crafties](https://www.youtube.com/watch?v=sxQAULX96P0).

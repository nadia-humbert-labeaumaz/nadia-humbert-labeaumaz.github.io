---
type: "post"
title: "Gérer les dépendances circulaires"
date: 2016-12-17 18:31:18 +0100
description: ""
author: Nadia Humbert-Labeaumaz
image : "images/blog/circular-dependencies/header.jpg"
bg_image: "images/blog/circular-dependencies/header.jpg"
categories: ["Software"]
tags: ["Design", "Java"]
draft: false
comments: true
---

Durant mon travail, j'ai rencontré des dépendances circulaires dans une application sur laquelle je suis intervenue. Dans sa plus simple forme, il s'agit de deux classes qui dépendent l'une de l'autre.
Ceci est, selon moi, un problème pour plusieurs raisons. L'objectif de cet article est de montrer ce qu'est une dépendance circulaire, en quoi cela peut poser problème, et comment les éliminer.

<!-- more -->

## Cas d'étude

Le code ci-dessous présente un cas simple de dépendance circulaire.

```java
class A {
  private final B b;

  A(B b) {
    this.b = b;
  }

  void doSomething() {
    b.doSomethingGreat();
  }

  void doSomethingAwesome() {
    System.out.println("Doing something awesome!");
  }
}

class B {
  private A a;

  void setA(A a) {
    this.a = a;
  }

  void doSomething() {
      a.doSomethingAwesome();
  }

  void doSomethingGreat() {
    System.out.println("Doing something great!");
  }
}
```

## Problèmes associés

### Instanciation de b

Le code suivant illustre l'utilisation de `B` :

```java
B b = new B(); // 1
A a = new A(b);
b.setA(a); // 2
b.doSomething(); // 3
```

Nous pouvons observer que la classe `B` ne peut pas être utilisée directement après son instanciation (`1`). Elle est dans un état incohérent car il est impératif de setter l'instance de `A` (`2`) afin de pouvoir l'utiliser. Sinon, une exception de type `NullPointerException` sera remontée lors de l'appel de la méthode `doSomething` (`3`). Cette opération (`2`) peut facilement être oubliée. De plus, si l'appel de la méthode (`3`) intervient bien plus tard, cet oubli peut ne pas être détecté immédiatemment.

### Immuabilité de B

Un autre inconvénient est que `B` n'est pas immuable. En effet, une fois instancié, il est possible de modifier son état en appelant autant de fois que l'on veut la méthode `setA`.
Pour rendre `B` immuable, il faudrait supprimer le setter, rendre l'attribut `a` `final` et le passer en paramètre du constructeur. Ceci est impossible actuellement à cause de la dépendance circulaire.

### Fort couplage

Le fort couplage introduit entre les deux classes peut être un signe qu'il y a un problème de séparation des responsabilités. Il est, par exemple, possible que les deux classes partagent la même responsabilité et qu'elles puissent donc être fusionnées. Une autre possibilité est qu'une troisième responsabilité soit présente et qu'elle doive être extraite dans une classe séparée.

## Solutions envisageables

Dans le contexte de mon travail, les solutions que j'ai dû adopter étaient relativement simples. Ces solutions seront décrites par la suite.

### Déplacer le comportement dans une des deux classes

Il est possible que le comportement de `doSomethingAwesome` soit lié uniquement à la classe `B`. Dans ce cas, il est possible de déplacer cette méthode dans `B` :

```java
class A {
  private final B b;

  A(B b) {
    this.b = b;
  }

  void doSomething() {
    b.doSomethingGreat();
  }

  void doSomethingAwesome() {
    b.doSomethingAwesome();
  }
}

class B {
  void doSomething() {
      doSomethingAwesome();
  }

  void doSomethingGreat() {
    System.out.println("Doing something great!");
  }

  void doSomethingAwesome() {
    System.out.println("Doing something awesome!");
  }
}
```

Il n'y a donc plus de dépendance circulaire car `B` ne dépend plus de `A`. Le code est ainsi plus SOLID car la classe `A` n'a plus de responsabilités qui ne lui appartiennent pas.

Le code ci-dessus a été écrit de manière à conserver l'API de `A`. La méthode `doSomethingAwesome` de `A` pourrait donc être supprimée si elle n'est désormais plus appelée.

### Créer une nouvelle classe

Si la méthode `doSomethingAwesome` n'est une responsabilité ni de `A` ni de `B`, elle doit être extraite dans une classe séparée :

```java
class Awesome {
  void doSomethingAwesome() {
    System.out.println("Doing something awesome!");
  }
}

class A {
  private final B b;
  private final Awesome awesome;

  A(B b, Awesome awesome) {
    this.b = b;
    this.awesome = awesome;
  }

  void doSomething() {
    b.doSomethingGreat();
  }

  void doSomethingAwesome() {
    awesome.doSomethingAwesome();
  }
}

class B {
  private final Awesome awesome;

  B(Awesome awesome) {
    this.awesome = awesome;
  }

  void doSomething() {
    awesome.doSomethingAwesome();
  }

  void doSomethingGreat() {
    System.out.println("Doing something great!");
  }
}
```

## Conclusion

Les dépendances circulaires peuvent être un signe de mauvais design. En effet, elles introduisent un fort couplage, provoquent l'instanciation d'objets incohérents et empêchent l'immuabilité de ceux-ci. Il est donc nécessaire de les analyser afin de bien comprendre leur origine et de les corriger si besoin. Dans cet article, deux manières de faire ont été présentées.

---
type: "post"
title: "ATDD et TDD double boucle"
date: 2016-05-22 22:27:01 +0200
description: ""
author: Nadia Humbert-Labeaumaz
image : "images/blog/atdd/header.jpg"
bg_image: "images/blog/atdd/header.jpg"
categories: ["Software"]
tags: ["Test"]
draft: false
comments: true
---

L'_Acceptance Test Driven Development_ est une pratique qui consiste à écrire un test d'acceptation dès la définition de la fonctionnalité à implémenter. Ce test permet ensuite de valider que l'implémentation de la fonctionnalité est terminée. En général, plusieurs composants unitaires sont nécessaires pour implémenter une fonctionnalité. Ces composants peuvent être développés en TDD dans une deuxième boucle de _feedback_.

L'objectif de cet article est de présenter l'ATDD et comment le mettre en pratique grâce à du TDD double boucle.

<!-- more -->

## ATDD

L'ATDD encourage fortement la collaboration entre les développeurs et le métier afin d'écrire des tests d'acceptation précis et pertinents. Pour optimiser l'échange, ces tests peuvent être écrits avec un vocabulaire partagé et bien défini. Ils peuvent également suivre le pattern _Given_ / _When_ / _Then_ comme dans l'exemple suivant :

```
# Given
Un client dépose 1000 euros sur son compte le 17/12/2015
Et il dépose 500 euros sur son compte le 18/12/2015
Et il retire 1500 euros de son compte le 19/12/2015

# When
Le client demande à imprimer son relevé de compte

# Then
Le relevé imprimé doit être égal à
   date | credit | debit | balance
   19/12/2015 | | 1500.00 | 0.00
   18/12/2015 | 500.00 | | 1500.00
   17/12/2015 | 1000.00 | | 1000.00
```

Pour implémenter la fonctionnalité, le développeur se sert de ce test d'acceptation et le traduit sous forme de code pour le rendre exécutable. Le test d'acceptation précédent peut être écrit en Java de la manière qui suit :

```java
@Test
public void should_print_statement() {
    // given
    Account account = new Account();
    account.deposit(new BigDecimal("1000"), LocalDate.of(2015, Month.DECEMBER, 17));
    account.deposit(new BigDecimal("500"), LocalDate.of(2015, Month.DECEMBER, 18));
    account.withdraw(new BigDecimal("1500"), LocalDate.of(2015, Month.DECEMBER, 19));

    // when
    String statement = account.printStatement();

    // then
    assertThat(statement).isEqualTo(
            "date | credit | debit | balance\n" +
            "19/12/2015 | | 1500.00 | 0.00\n" +
            "18/12/2015 | 500.00 | | 1500.00\n" +
            "17/12/2015 | 1000.00 | | 1000.00"
    );
}
```

Ce test sera rouge tant que la fonctionnalité ne sera pas implémentée entièrement. Il servira de fil conducteur lors des développements.

## TDD double boucle

La classe `Account` sert de point d'entrée à la fonctionnalité. Cependant, elle ne sera pas suffisante pour l'implémenter complètement. Le TDD double boucle consiste à développer la classe `Account` ainsi que ses dépendances en TDD, formant ainsi une boucle de _feedback_ à l'intérieur de celle déjà formée par le test d'acceptation.

![Double loop TDD](/images/drafties/double_loop_tdd.png)

Dans l'exemple présenté ci-dessus, il est possible de déléguer l'impression d'un relevé à une classe `Statement` qui est une dépendance de `Account`. Lors de l'écriture du test de `Account`, cette classe sera mockée.

```java
@Test
public void should_print_statement() {
    // given
    Statement statement = mock(Statement.class);
    when(statement.print()).thenReturn("printed statement");
    Account account = new Account(statement);

    // when
    String printedStatement = account.printStatement();

    // then
    assertThat(printedStatement).isEqualTo("printed statement");
}
```

De la même manière, la classe `Statement` sera développée en TDD en mockant sa dépendance vers `Transaction`, qui sera chargée d'imprimer une ligne du relevé. Finalement, cette classe `Transaction` sera implémentée en TDD.
Ainsi, le test d'acceptation passera au vert, ce qui indique que la fonctionnalité correspondante est terminée.

## Conclusion

L'ATDD encourage la collaboration métier / développeur à travers la définition précise de tests d'acceptation qui permettent de valider les fonctionnalités. Ces dernières peuvent être implémentées en utilisant du TDD double boucle.

Vous trouverez une mise en pratique du TDD double boucle dans la vidéo [Crafties #7](https://www.youtube.com/watch?v=Qe84jbwyZ3U).

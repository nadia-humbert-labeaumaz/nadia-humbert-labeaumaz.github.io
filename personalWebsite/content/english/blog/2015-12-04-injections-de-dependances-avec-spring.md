---
type: "post"
title: "Injections de dépendances avec Spring"
description: ""
date: 2015-06-14 08:35:20 +0200
author: Nadia Humbert-Labeaumaz
image : "images/blog/dependency-injection/header.jpg"
bg_image: "images/blog/dependency-injection/header.jpg"
categories: ["Software"]
tags: ["Spring Framework", "Java"]
draft: false
comments: true
---

Il existe plusieurs méthodes pour injecter une dépendance dans un objet Java :

- Injection sur un setter
- Injection sur le constructeur
- Injection directe sur la déclaration de l'attribut

Cet article a pour objectif de définir ces différentes méthodes, de décrire leurs avantages et inconvénients et indiquer leurs cas d'utilisation.

<!-- more -->

Dans la plupart des applications Java d'entreprise, il y a des services ayant besoin de DAO pour accéder à la base de données. Le code suivant est une illustration de ce cas :

```java
public class UserServiceImpl implements UserService {

   private UserDao userDao;

   @Override
   public User save(String name) {

      User user = new User(name);

      // NullPointerException car userDao n'est pas injecté.
      return userDao.save(user);
   }
}
```

Dans le cadre de cet article, l'annotation `@Autowired` de Spring sera utilisée pour injecter `userDao` dans le service.

## Injection sur un setter

Il est possible de créer un setter et de l'annoter avec `@Autowired`. Spring va alors utiliser ce setter pour injecter `userDao` dans le service.

```java
public class UserServiceImpl implements UserService {

   private UserDao userDao;

   @Autowired
   public void setUserDao(UserDao userDao) {
      this.userDao = userDao;
   }

   @Override
   public User save(String name) {

      User user = new User(name);

      // userDao est injecté via le setter annoté.
      return userDao.save(user);
   }
}
```

Cette méthode a pour avantage de rendre le `userDao` facilement injectable dans un test unitaire sans avoir à utiliser de framework particulier. Comme montré dans l'exemple ci-après :

```java
public class UserServiceTest {

   @Test
   public void test_save() {

      // given
      String name = "Martin";
      UserService userService = new UserServiceImpl();
      userService.setUserDao(new FakeUserDaoImpl());

      // when
      User user = userService.save(name);

      // then
      assertThat(user.getName()).isEqualTo(name);

   }

   private class FakeUserDaoImpl implements UserDao {

      @Override
      public User save(User user) {
         return user;
      }
   }

}
```

Cependant, elle a pour inconvénient de rendre l'attribut `userDao` du service modifiable par tous les objets qui disposent d'une instance du service (ils peuvent donc même le rendre `null`).

## Injection sur le constructeur

Dans cette méthode d'injection, le `userDao` est injecté dans le service via son constructeur annoté avec `@Autowired`.

```java
public class UserServiceImpl implements UserService {

   private final UserDao userDao;

   @Autowired
   public UserServiceImpl(UserDao userDao) {
      this.userDao = userDao;
   }

   @Override
   public User save(String name) {

      User user = new User(name);

      // userDao est injecté via le constructeur annoté.
      return userDao.save(user);
   }
}
```

Comme la méthode d'injection à l'aide du setter, celle-ci permet de rendre le `userDao` facilement injectable dans un test unitaire.

```java
public class UserServiceTest {

   @Test
   public void test_save() {

      // given
      String name = "Martin";
      UserService userService = new UserServiceImpl(new FakeUserDaoImpl());

      // when
      User user = userService.save(name);

      // then
      assertThat(user.getName()).isEqualTo(name);

   }

   private class FakeUserDaoImpl implements UserDao {

      @Override
      public User save(User user) {
         return user;
      }
   }

}
```

De plus, elle permet également d'assurer que le `userDao` ne sera jamais modifié. Il suffirait donc de mettre un contrôle de nullité dans le constructeur pour certifier qu'il ne sera jamais `null`.

Cependant, elle a pour inconvénient d'imposer la création de la dépendance dès l'instanciation du service même si elle n'est pas nécessaire.

## Injection sur la déclaration de l'attribut

Cette méthode consiste à ajouter l'annotation `@Autowired` directement sur la déclaration de l'attribut.

```java
public class UserServiceImpl implements UserService {

   @Autowired
   private UserDao userDao;

   @Override
   public User save(String name) {

      User user = new User(name);

      // userDao est injecté via l'annotation @Autowired.
      return userDao.save(user);
   }
}
```

Cette manière d'injecter a l'avantage d'être simple à utiliser.

Néanmoins, elle n'est pas conseillée car elle force à employer la réfléxion, ce qui la rend notamment plus compliquée à tester (utilisation obligatoire d'un framework de test).

```java
public class UserServiceTest {

   @Test
   public void test_save() {

      // given
      String name = "Martin";
      UserService userService = new UserServiceImpl();
      UserDao fakeUserDao = new FakeUserDaoImpl();

      // Utilisation de la classe ReflectionTestUtils de Spring.
      ReflectionTestUtils.setField(userService, "userDao", fakeUserDao);

      // when
      User user = userService.save(name);

      // then
      assertThat(user.getName()).isEqualTo(name);

   }

   private class FakeUserDaoImpl implements UserDao {

      @Override
      public User save(User user) {
         return user;
      }
   }

}
```

De plus, elle rompt le principe de la programmation orientée objet qui stipule que les objets sont responsables de leurs attributs privés. En effet, l'attribut privé est ici manipulé directement par Spring ou le framework de test choisi.

## Conclusion

L'injection d'un attribut d'un objet peut se faire de différentes manières.
L'injection par setter a pour avantage de rendre le code facilement testable. L'injection par constructeur a, en plus, l'avantage de pouvoir contrôler la nullité de l'attribut. Elle est donc conseilée pour les attributs obligatoires. Finalement, l'injection sur la déclaration de l'attribut est déconseilée car elle rend le code moins facilement testable et crée des dépendances cachées.

La testabilité du code est importante. En effet, un code non testable est signe d'un code qui sera difficile à comprendre et à maintenir.

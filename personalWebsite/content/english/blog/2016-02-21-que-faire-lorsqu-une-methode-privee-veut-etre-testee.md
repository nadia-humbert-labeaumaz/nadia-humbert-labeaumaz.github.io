---
type: "post"
title: "Que faire lorsqu'une méthode privée veut être testée ?"
date: 2016-02-21 11:09:38 +0100
description: ""
author: Nadia Humbert-Labeaumaz
image : "images/blog/testing-private-method/header.jpg"
bg_image: "images/blog/testing-private-method/header.jpg"
categories: ["Software"]
tags: ["software-testing", "software-design"]
draft: false
comments: true
---

Les tests automatisés servent à vérifier le bon comportement d'un objet (ou d'un ensemble d'objets), indépendamment de la manière dont ce comportement est implémenté. Le comportement d'un objet est décrit par son API publique (constructeurs, constantes et méthodes publiques). Les tests ne devraient donc utiliser que cette API.

Les méthodes privées (et _protected_) ne faisant pas partie de l'API publique d'un objet, elles ne devraient pas être appelées directement par le code de test.

Cet article a pour objectif de montrer comment réagir lorsqu'il paraît nécessaire de tester une méthode privée.

<!-- more -->

## Contexte

Un projet d'illustration a été créé afin de servir de support pour cet article. Il est disponible sur [GitHub](https://github.com/nphumbert/demo-private-method-test/).

Il s'agit de hasher le mot de passe lors de la création d'un utilisateur. Il faut imaginer que la méthode de hash est plus complexe (utilisation de librairies externes, de nombreuses conditions, etc.) pour donner tout son sens à la suite de l'article.

```java
public class User {

    private final String login;
    private final String email;
    private final String password;

    public User(String login, String email, String password) {
        this.login = login;
        this.email = email;
        this.password = hash(password);
    }

    // ...

    private String hash(String password) {
        try {
            MessageDigest md = MessageDigest.getInstance("SHA-256");
            return Arrays.toString(md.digest(password.getBytes("UTF-8")));
        } catch (NoSuchAlgorithmException | IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```

La méthode de hash étant considérée comme complexe, le besoin se fait sentir de tester son code en isolation. En effet, en partant de cette hypothèse, il n'est pas facilement possible de le tester complètement en utilisant uniquement des appels à l'API publique.

## Eviter la réflexion

Parfois, la réflexion est utilisée pour augmenter la visibilité de la méthode privée durant la durée d'un test. Cette méthode peut alors être appelée directement.

```java
@Test
public void should_hash_password() throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
    // given
    User user = new User("jdoe", "john.doe@gmail.com", "secret");
    Method method = user.getClass().getDeclaredMethod("hash", String.class);
    method.setAccessible(true);

    // when
    String hashedPassword = (String) method.invoke(user, "secret");

    // then
    assertThat(hashedPassword, is("[43, -72, 13, 83, 123, 29, -93, -29, -117, -45, 3, 97, -86, -123, 86, -122, -67, -32, -22, -51, 113, 98, -2, -10, -94, 95, -23, 123, -11, 39, -94, 91]"));
}
```

Utiliser la réflexion dans ce cadre est dangereux !

En effet, toutes les actions ci-dessous auront pour effet de faire échouer le test, sans empêcher celui-ci de compiler :

- Renommer la méthode `hash`.
- Changer l'ordre ou le type des attributs de la méthode `hash`.
- Changer le type de retour de la méthode `hash`.
- Supprimer la méthode `hash` en intégrant son code dans une autre méthode (ou dans le constructeur).

Toutes ces actions sont des décisions d'implémentation qui n'ont aucun lien avec le comportement de la classe. Il n'est donc pas normal qu'un test échoue lorsque l'une de ces actions est effectuée.

Par ailleurs, l'API de réflexion Java manipulant des `String` et des `Object`, l'IDE n'est pas capable d'aider à corriger automatiquement le code de test correspondant.

## Déplacer le code dans une dépendance externe

Selon moi, la meilleure manière de réagir est d'extraire le code, qui doit être testé, dans sa propre classe.

```java
public interface HashProvider {
    String hash(String value);
}
```

```java
public class Sha256HashProvider implements HashProvider {
    @Override
    public String hash(String value) {
        try {
            MessageDigest md = MessageDigest.getInstance("SHA-256");
            return Arrays.toString(md.digest(value.getBytes("UTF-8")));
        } catch (NoSuchAlgorithmException | IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```

La classe `User` de l'exemple est alors transformée comme ci-dessous.

```java
public class User {

    private final String login;
    private final String email;
    private final String password;

    public User(String login, String email, String password, HashProvider hashProvider) {
        this.login = login;
        this.email = email;
        this.password = hashProvider.hash(password);
    }
	// ...
}
```

Par ailleurs, le test se trouve également modifié.

```java
@Test
public void should_hash_password_when_create_user() {
    // given
    HashProvider hashProvider = mock(HashProvider.class);
    when(hashProvider.hash("secret")).thenReturn("hash");

    // when
    User user = new User("jdoe", "john.doe@gmail.com", "secret", hashProvider);

    // then
    assertThat(user.password(), is("hash"));
}
```

Le code respecte désormais le [_Single Responsibility Principle_ (SRP)](https://en.wikipedia.org/wiki/Single_responsibility_principle). En effet, chaque classe ne fait qu'une seule chose. Il n'est donc, par exemple, plus nécessaire de modifier la classe `User` pour changer la logique de hashage.

Le [_Open/Closed Principle_ (OCP)](https://en.wikipedia.org/wiki/Open/closed_principle) est aussi mis en application car la classe `User` ne se préoccupe pas de l'implémentation de `HashProvider` qu'elle utilise. Il est donc possible de la changer sans que le code de `User` en soit affecté.

Tout le code est complètement testé en utilisant seulement les API publiques. Cela signifie que seul le comportement est testé, et non plus l'implémentation.

Finalement, le code de production ainsi que celui de test est simplifié.

## Conclusion

Lorsqu'une méthode privée devient si complexe que le besoin de la tester en isolation se fait sentir, cela signifie que la classe fait trop de choses et que le SRP n'est sûrement pas respecté. Extraire cette méthode dans une classe externe permet de tester complètement ce code tout en améliorant le design en rendant le code plus SOLID.

Dès qu'une méthode privée est créée afin d'effectuer un traitement, il peut être utile de se demander si elle ne devrait pas être extraite, tout en restant pragmatique.

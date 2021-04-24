---
type: "post"
title: "Pourquoi ne pas utiliser l'héritage pour éviter la duplication de code ?"
slug: "heritage-duplication-code"
date: 2016-01-24 11:38:41 +0100
description: ""
author: Nadia Humbert-Labeaumaz
image : "images/blog/inheritance-duplication/header.jpg"
bg_image: "images/blog/inheritance-duplication/header.jpg"
categories: ["Software"]
tags: ["Java", "Software Design"]
draft: false
comments: true
---

L'héritage est une composante très importante des langages orientés objet tels que Java. Cependant, il doit être utilisé à bon escient afin de respecter les bonnes pratiques de programmation.

Il m'est arrivé de rencontrer des cas où l'héritage était utilisé afin de ne pas dupliquer du code. Le but de cet article est d'illustrer une telle utilisation, d'analyser ses inconvénients et de montrer une manière possible de la corriger.

<!-- more -->

## Contexte

Un projet d'illustration a été créé afin de servir de support pour cet article. Il est disponible sur [GitHub](https://github.com/nphumbert/demo-inappropriate-inheritance). Ce projet contient deux branches :

- `inappropriate-inheritance` : contient l'utilisation inappropriée de l'héritage.
- `dependency` : contient une proposition de correction en utilisant une dépendance.

Il s'agit de deux _controllers_ Spring MVC qui doivent hasher un texte. Dans le cas de l'utilisation inappropriée de l'héritage, le code d'un des _controllers_ est le suivant :

```java
@RestController
public class ProfileController extends BaseController {

    @RequestMapping(value = "/profile", method = RequestMethod.GET)
    public String profile() {
        return "profile: " + hash("profile");
    }
}
```

Ce _controller_ hérite d'une classe commune qui contient la méthode `hash` :

```java
public abstract class BaseController {
    protected String hash(String value) {
        // ...
    }
}
```

## Pourquoi n'est-il pas correct d'utiliser l'héritage ici ?

### Non respect de la programmation orientée objet (POO)

Dans ce code, la mécanique de la POO est respectée mais pas sa sémantique. En effet, la classe `BaseController` n'a pas de raison d'être. Elle ne correspond à aucun concept et ne sert qu'à contenir du code partagé entre les _controllers_. Ce genre de classe possède souvent un nom flou et générique, ce qui est un signe que le concept associé est mal défini, voire inexistant.

### Difficulté à tester

La capacité du code à être testé n'est pas une fin en soi. Cependant, un code difficile à tester est un _smell_ qui indique le plus souvent un problème de conception.

Ici, il est obligatoire de passer par un _set up_ assez lourd pour tester en isolation le _controller_. Une classe privée héritant du _controller_ est créée afin de surcharger la méthode `hash` et de fixer la valeur de retour. Les tests portent donc sur cette classe au lieu de porter sur le _controller_ initial.

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = InappropriateInheritanceApplication.class)
@WebAppConfiguration
public class ProfileControllerTest {

    public static final String HASH = "hash";

    private class TestableProfileController extends ProfileController {

        @Override
        protected String hash(String value) {
            return HASH;
        }
    }

    private MockMvc mockMvc;

    @Before
    public void setUp() {
        ProfileController controller = new TestableProfileController();
        this.mockMvc = MockMvcBuilders.standaloneSetup(controller).build();
    }

    @Test
    public void should_get_hash_when_get_profile() throws Exception {
        // when
        String contentAsString = mockMvc.perform(get("/profile"))
                .andExpect(status().isOk())
                .andReturn().getResponse().getContentAsString();

        // then
        assertThat(contentAsString, is("profile: " + HASH));
    }
}
```

La complexité du _set up_ est bien trop importante par rapport au code à tester qui est relativement simple.

### Fort couplage

L'héritage introduit un fort couplage entre les classes. En effet, tous les _controllers_ doivent hériter de `BaseController` pour bénéficier de la méthode `hash`. Ceci implique qu'ils doivent avoir accès à cette classe (soit être dans le même projet, soit avoir une dépendance vers sa librairie).

De plus, l'héritage multiple étant interdit en Java, les _controllers_ ne peuvent pas hériter d'une autre classe qui serait appropriée.

Finalement, si une autre classe avait besoin de la méthode `hash`, elle devrait forcément hériter de `BaseController` (ce qui n'a pas de sens s'il ne s'agit pas d'un _controller_) ou alors dupliquer le code. Ce problème met en évidence le fait que la notion de hashage n'a aucun rapport avec la notion de _controller_.

### Difficulté à maintenir

Le couplage fort décrit précédemment rend le code difficile à maintenir. Un changement du besoin entraînerait une modification du code à un endroit où on ne s'y attend pas. Par ailleurs, il pourrait y avoir des effets de bord inattendus sur le reste du code.

## Comment peut-on corriger ce code ?

La solution proposée pour corriger ce code est d'extraire la méthode de `hash` dans une dépendance qui sera injectée dans les _controllers_.

Tout d'abord, une interface `HashProvider` est créée et implémentée.

```java
public interface HashProvider {
    String hash(String text);
}
```

```java
@Component
public class Sha256HashProvider implements HashProvider {

    @Override
    public String hash(String value) {
		// ...
    }
}
```

Cette implémentation peut alors être injectée dans le _controller_ afin d'être utilisée.

```java
@RestController
public class ProfileController {

    private final HashProvider hashProvider;

    @Autowired
    public ProfileController(HashProvider hashProvider) {
        this.hashProvider = hashProvider;
    }

    @RequestMapping(value = "/profile", method = RequestMethod.GET)
    public String profile() {
        return "profile: " + hashProvider.hash("profile");
    }
}
```

Le _set up_ de test de ce _controller_ s'en trouve simplifié car il n'y a plus qu'à _mocker_ l'interface.

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = InappropriateInheritanceApplication.class)
@WebAppConfiguration
public class ProfileControllerTest {

    private MockMvc mockMvc;
    private HashProvider hashProvider;

    @Before
    public void setUp() {
        hashProvider = mock(HashProvider.class);
        ProfileController controller = new ProfileController(hashProvider);
        this.mockMvc = MockMvcBuilders.standaloneSetup(controller).build();
    }

    @Test
    public void should_get_hash_when_get_profile() throws Exception {
        // given
        when(hashProvider.hash("profile")).thenReturn("hash");

        // when
        String contentAsString = mockMvc.perform(get("/profile"))
                .andExpect(status().isOk())
                .andReturn().getResponse().getContentAsString();

        // then
        assertThat(contentAsString, is("profile: hash"));
    }
}
```

Finalement, la logique de hashage est totalement découplée de celle du _controller_. Ceci rend le code plus simple à maintenir car si le besoin change, il suffit de modifier l'implémentation de `HashProvider` ou d'en ajouter une nouvelle. Ainsi, le comportement du _controller_ est modifié sans que son code ne change.

## Conclusion

Il est nécessaire de faire attention à ne pas abuser de l'héritage. Dans cet article, il a été montré qu'il n'est pas approprié pour éviter la duplication de code. Dans ce cas, il est préférable d'utiliser une interface dont une implémentation sera injectée. Ceci a pour principal avantage de rendre le code plus simple et facile à maintenir.

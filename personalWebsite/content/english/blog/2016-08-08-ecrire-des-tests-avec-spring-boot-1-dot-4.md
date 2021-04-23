---
type: "post"
title: "Ecrire des tests avec Spring Boot 1.4"
date: 2016-08-08 07:11:29 +0200
description: ""
author: Nadia Humbert-Labeaumaz
image : "images/blog/spring-boot-test/header.jpg"
bg_image: "images/blog/spring-boot-test/header.jpg"
categories: ["Software"]
tags: ["Java", "Software Testing"]
draft: false
comments: true
---

La version 1.4 de Spring Boot est sortie le 28 juillet 2016. Elle contient notamment des évolutions importantes au niveau de l'écriture des tests.  
L'objectif de cet article est de voir comment migrer les tests d'un _controller_ Spring MVC en utilisant les nouvelles fonctionnalités apportées par cette version.

<!-- more -->

## Modification du pom.xml

Le starter de test de Spring Boot 1.4 embarque désormais les dépendances vers [AssertJ](http://joel-costigliola.github.io/assertj), [JSONassert](https://github.com/skyscreamer/JSONassert) et [JsonPath](https://github.com/jayway/JsonPath). Il est donc possible de supprimer l'appel explicite vers ces dépendances dans notre pom, il suffit d'importer le starter :

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-test</artifactId>
	<scope>test</scope>
</dependency>
```

## Configuration du test

Dans Spring Boot 1.3, il est possible d'écrire un test d'intégration d'un _controller_ avec Spring de la manière suivante :

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = DemoApplication.class)
@WebAppConfiguration
public class DemoControllerTest {
    // ...
}
```
Le test ci-dessus va charger la classe de configuration Spring Boot `DemoApplication` à l'intérieur d'un contexte d'application de type `WebApplicationContext`. Il sera donc possible de [tester le _controller_ avec MockMvc](https://nphumbert.github.io/blog/2015/10/31/testing-spring-mvc-controllers).

Depuis Spring Boot 1.4, l'annotation `@SpringBootTest`remplace toutes les annotations existantes pour faire des tests d'intégration avec Spring. De plus, le runner JUnit peut être remplacé par `SpringRunner` qui a été introduit dans la version 4.3 de Spring. Finalement, il n'est plus nécessaire de fournir explicitement la classe de configuration Spring Boot. En effet, la classe annotée avec `@SpringBootApplication` sera automatiquement utilisée. Le test devient donc :

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class DemoControllerTest {
    // ...
}
```

## Gestion des mocks

Pour tester un _controller_ en isolation, il faut l'instancier en mockant ses dépendances (avec [Mockito](http://mockito.org) par exemple) et le fournir au builder de `MockMvc`.

Avec Spring Boot 1.3, il est possible d'écrire ceci :

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = DemoApplication.class)
@WebAppConfiguration
public class DemoControllerTest {

    private MockMvc mockMvc;

    @Mock
    private DemoService demoService;

    @InjectMocks
    private DemoController demoController;

    @Before
    public void setUp() {
        Mockito.initMocks(this);
        this.mockMvc = MockMvcBuilders.standaloneSetup(demoController).build();
        Mockito.when(demoService.call()).thenReturn(42);
    }
}
```

Spring Boot 1.4 permet de mocker les beans Spring avec Mockito grâce à une annotation spécifique. Ainsi, le bean de l'`ApplicationContext` est remplacé par un _mock_ et Spring se charge donc de l'injecter dans le _controller_. Le test devient donc le suivant :

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class DemoControllerTest {

    private MockMvc mockMvc;

    @MockBean
    private DemoService demoService;

    @Autowired
    private DemoController demoController;

    @Before
    public void setUp() {
        this.mockMvc = MockMvcBuilders.standaloneSetup(demoController).build();
        Mockito.when(demoService.call()).thenReturn(42);
    }
}
```

Dans ce test, le contexte d'application est chargé dans son intégralité avec le bean `DemoService` remplacé par un _mock_. Le `DemoController` injecté utilisera donc ce _mock_.

## Limitation de la configuration chargée

Dans ce contexte, il n'est pas forcément nécessaire de charger tout le contexte d'application pour tester uniquement le _controller_.

Spring Boot 1.4 introduit l'annotation `@WebMvcTest` qui permet de tester spécifiquement des _controllers_ Spring MVC avec `MockMvc`. Ainsi, seule la configuration Spring MVC sera chargée. Le test précédent peut donc s'écrire de la manière suivante :


```java
@RunWith(SpringRunner.class)
@WebMvcTest(DemoController.class)
public class DemoControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private DemoService demoService;

    @Before
    public void setUp() {
        Mockito.when(demoService.call()).thenReturn(42);
    }
}
```

`MockMvc` n'a plus besoin d'être instancié explicitement dans le _setup_. Grâce à l'annotation `@WebMvcTest`, l'instance peut être directement injectée dans le test.

## Conclusion

Les évolutions apportées par Spring Boot 1.4 au niveau des tests permettent de simplifier l'écriture des tests d'intégration. En effet, il n'y a plus qu'une seule annotation à utiliser (`@SpringBootTest`). De plus, l'intégration de Mockito dans Spring Boot permet de remplacer directement des beans du contexte d'application par des _mocks_. Finalement, dans le cadre du test d'un _controller_ Spring MVC avec `MockMvc`, l'annotation `@WebMvcTest` permet de ne charger que les beans nécessaires à l'exécution des tests.

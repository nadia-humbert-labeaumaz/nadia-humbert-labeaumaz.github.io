---
type: "post"
title: "Exposer des ressources statiques avec Spring MVC"
slug: "exposer-des-ressources-statiques-avec-spring-mvc"
description: ""
date: 2015-09-18 12:59:06 +0100
author: Nadia Humbert-Labeaumaz
image : "images/blog/exposing-static-resources/header.jpg"
bg_image: "images/blog/exposing-static-resources/header.jpg"
categories: ["Software"]
tags: ["Java"]
draft: false
comments: true
---

Il est souvent nécessaire d'exposer des ressources statiques telles que des images, des fichiers pdf, des vidéos, etc. dans une application web.
Ces fichiers sont stockés sur le serveur et ne sont donc pas accessibles à l'utilisateur via une URL.


L'objectif de cet article est d'exposer des fichiers stockés sur le serveur via une URL dans une application Spring MVC.

<!--more-->

## Surcharge de la configuration web

Pour exposer des ressources statiques, il faut surcharger la configuration web de Spring MVC.
Pour ce faire, une classe de configuration qui hérite de `WebMvcConfigurerAdapter` doit être implémentée.
Plus précisément, il est nécessaire de surcharger la méthode `addResourceHandlers()`.

```java
@Configuration
public class WebConfig extends WebMvcConfigurerAdapter {

    private static final String DIRECTORY = "/path/to/server/disk/resources";

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/resources/**") // 1
                .addResourceLocations("file:///" + DIRECTORY + "/"); // 2
    }
}
```


La méthode de configuration fournit un objet de type `ResourceHandlerRegistry` qui permet d'associer une URL (`1`) à un dossier (`2`). Ainsi, l'URL `http://example.com/resources/file.pdf` sera associée au fichier `/path/to/server/disk/resources/file.pdf`.

## Conclusion

Pour conclure, Spring permet d'exposer des ressources statiques via des URL de manière simple et rapide.

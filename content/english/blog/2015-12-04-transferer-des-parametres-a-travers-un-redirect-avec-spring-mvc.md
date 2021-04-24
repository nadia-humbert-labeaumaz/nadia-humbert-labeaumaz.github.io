---
type: "post"
title: "Transférer des paramètres à travers un redirect avec Spring MVC"
slug: "transferer-des-parametres-a-travers-un-redirect-avec-spring-mvc"
date: 2015-07-25 12:06:04 +0200
description: ""
author: Nadia Humbert-Labeaumaz
image : "images/blog/transfer-redirect-parameters/header.jpg"
bg_image: "images/blog/transfer-redirect-parameters/header.jpg"
categories: ["Software"]
tags: ["Java"]
draft: false
comments: true
---

Dans une application web, il est courant d'effectuer une redirection après la soumission d'un formulaire. Cette redirection permet notamment de rendre l'url _bookmarkable_ et d'empêcher l'utilisateur de resoumettre le formulaire par erreur (suite à un _refresh_, par exemple).

Il peut être intéressant de passer des paramètres à travers ce redirect. Par exemple, pour indiquer à l'utilisateur que son formulaire a bien été soumis. Pour ce faire, il est possible d'ajouter des paramètres directement dans l'url : `redirect:/path/to/endpoint?formSubmitted=true`.
Cette solution n'est pas toujours adpatée pour des raisons de confidentialité, de taille d'url ou simplement d'esthétique.

L'objectif de cet article est de démontrer comment transférer des paramètres à travers une redirection avec Spring MVC en utilisant des _flash attributes_. L'exemple utilisé consistera en un formulaire qui permet d'envoyer un nom. Ce nom s'affichera ensuite une fois le formulaire soumis.  

<!-- more -->

## Traitement du formulaire

Le code suivant décrit la gestion de la soumission du formulaire par un _controller_ Spring MVC :

```java
@RequestMapping(value = "/", method = RequestMethod.POST)
public ModelAndView post(@RequestParam("name") String name, RedirectAttributes redirectAttributes) {
   redirectAttributes.addFlashAttribute("name", name); // 1
   return new ModelAndView("redirect:/"); // 2
}
```

Afin d'ajouter un _flash attribute_ à la redirection, il faut utiliser une instance de la classe `RedirectAttributes` (`1`). Cette instance est automatiquement injectée par Spring lorsqu'elle est déclarée dans la méthode du _controller_.

Une fois le _flash attribute_ ajouté, il est possible de faire la redirection (`2`).


## Traitement de l'affichage

Le code suivant décrit le traitement de la requête `GET` par un _controller_ Spring MVC :

```java
@RequestMapping(value = "/", method = RequestMethod.GET)
public ModelAndView get(HttpServletRequest request) {

   ModelAndView modelAndView = new ModelAndView("default");

   Map<String, ?> flashMap = RequestContextUtils.getInputFlashMap(request); // 1
   if (!CollectionUtils.isEmpty(flashMap)) {
      modelAndView.addObject("name", flashMap.get("name")); // 2
   }

   return modelAndView;
}
```

Pour récupérer la _map_ contenant les _flash attributes_, il faut utiliser la méthode `RequestContextUtils.getInputFlashMap` sur la requête `HttpServletRequest` (`1`). Cette requête est également injectée par Spring lorsqu'elle est déclarée dans la méthode du _controller_.


La valeur de l'attribut peut alors être récupérée depuis la _map_ pour être utilisée, ici dans le `ModelAndView` (`2`).

Ci-dessous un exemple est donné pour afficher conditionnellement cet attribut dans un template [Thymeleaf](http://www.thymeleaf.org/) :


```java
<div class="alert alert-success" role="alert" th:if="${not #strings.isEmpty(name)}">
   Hello, <span th:text="${name}">John Doe</span>!
</div>
```

## Conclusion

Les _flash attributes_ de Spring MVC peuvent être utilisés pour transférer des paramètres à travers une redirection. Ce mécanisme permet notamment d'implémenter très facilement une confirmation de soumission de formulaire à l'utilisateur.

Le code source du projet de démonstration peut être trouvé sur [GitHub](https://github.com/nphumbert/demo-flash-attributes).

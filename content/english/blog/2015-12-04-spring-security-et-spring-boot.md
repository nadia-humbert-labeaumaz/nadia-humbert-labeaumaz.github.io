---
type: "post"
title: "Spring Security et Spring Boot"
slug: "spring-security-et-spring-boot"
description: ""
date: 2015-06-21 11:51:37 +0200
author: Nadia Humbert-Labeaumaz
image : "images/blog/spring-security/header.jpg"
bg_image: "images/blog/spring-security/header.jpg"
categories: ["Software"]
tags: ["Java"]
draft: false
comments: true
---

Depuis Spring 3.1, il est possible de configurer Spring en Java.
La configuration Java de Spring Security est supportée depuis sa version 3.2 ([source](http://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#jc)).

L'objectif de cet article est de montrer comment configurer Spring Security en Java config, dans une application Spring Boot, avec une base de données qui contient les utilisateurs et leur mot de passe hashé. De plus, une authentification HTTP basic sera mise en place.

<!-- more -->

## Créer la classe de configuration

La classe Java qui doit gérer la configuration doit être munie de l'annotation `@Configuration` définie par Spring. De plus, elle doit étendre la classe `WebSecurityConfigurerAdapter`. Spring Boot mettant en place une configuration par défaut, il faut donc finalement utiliser l'annotation `@Order` afin d'indiquer que la nouvelle configuration doit être utilisée.

```java
@Configuration
@Order(SecurityProperties.ACCESS_OVERRIDE_ORDER)
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {

}
```
## Configurer Spring Security pour utiliser une base de données

Pour indiquer à Spring la méthode d'authentification, il faut redéfinir la méthode `configure(AuthenticationManagerBuilder)`. Spring Security offre une API _fluent_ pour configurer une authentification à partir de l'objet `AuthenticationManagerBuilder`.

Le code ci-dessous permet de configurer l'authentification en utilisant une base de données.

```java
@Configuration
@Order(SecurityProperties.ACCESS_OVERRIDE_ORDER)
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {

    @Autowired
    private DataSource dataSource; // 3

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.jdbcAuthentication() // 1
                .passwordEncoder(new ShaPasswordEncoder(256)) // 6
                .dataSource(dataSource) // 2
                .usersByUsernameQuery("SELECT login, password, active FROM user WHERE login=?") // 4
                .authoritiesByUsernameQuery("SELECT login, role FROM user WHERE login=?"); // 5
    }
}
```

Dans le cadre de cet article, il s'agit d'une authentification JDBC (`1`). La base de données à laquelle Spring doit accéder est accessible via une `dataSource` (`2`). Elle a été configurée dans l'application (typiquement dans `application.properties`) et peut donc être directement injectée par Spring grâce à l'annotation `@Autowired` (`3`).

La requête pour récupérer un utilisateur à partir de son login est fournie via la méthode `usersByUsernameQuery` (`4`) et celle pour récupérer les rôles d'un utilisateur à partir de son login via la méthode `authoritiesByUsernameQuery` (`5`). Un utilisateur est caractérisé par un login, un mot de passe et un booléen indiquant s'il est actif. Un rôle est une chaîne de caractères qui correspondra à un profil ayant un ensemble de droits.

Finalement, il est indiqué que les mots de passe dans la base sont hashés à l'aide de l'agorithme SHA-256 (`6`).

## Configuration de l'authentification HTTP basic

La méthode d'authentification [HTTP basic](https://fr.wikipedia.org/wiki/Authentification_HTTP#M.C3.A9thode_.C2.AB_Basic_.C2.BB) est une méthode d'authentification simple en HTTP et sera utilisée dans cet article.
Cette méthode est indiquée en surchargeant la méthode `configure(HTTPSecurity)`. De même que dans la section précédente, Spring Security offre une API _fluent_ pour configurer une authentification HTTP à partir de l'objet `HTTPSecurity`.

Le code ci-dessous permet de configurer l'authentification HTTP basic.

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.httpBasic() // 1
            .and()
                .authorizeRequests()
                    .antMatchers("/", "/css/**", "/js/**").permitAll() // 2
                    .anyRequest().authenticated() // 3
            .and()
                .formLogin()
                   .loginPage("/login").permitAll() // 4
            .and()
               .logout()
                   .logoutUrl("/logout") // 5
                   .logoutSuccessUrl("/"); // 6
}
```

La méthode d'authentification HTTP est précisée avec la méthode `httpBasic` (`1`). La racine, les fichiers CSS et javascript sont accessibles à tout le monde (`2`) tandis que toutes les autres requêtes nécessitent d'être authentifié (`3`).  

L'URL de login est spécifiée via la méthode `loginPage` (`4`) et celle de logout via la méthode `logoutUrl` (`5`). Après la déconnexion, l'utilisateur est redirigé vers la racine (`6`).

Le code suivant illustre un formulaire de login et un formulaire de logout.

```html
<!-- Formulaire de login -->
<form action="/login" method="post">
	<input type="text" name="username" />
	<input type="password" name="password" />
	<button type="submit">Login</button>
</form>

<!-- Formulaire de logout -->
<form action="/logout" method="post">
	<button type="submit">Logout</button>
</form>
```

## Conclusion

La configuration de Spring Security en Java config est relativement simple et directe. Elle permet de contrôler et de centraliser les accès à toutes les URL de l'application. De plus, il est possible de définir très simplement les URL de login et de logout, Spring Security se chargeant des traitements des requêtes.

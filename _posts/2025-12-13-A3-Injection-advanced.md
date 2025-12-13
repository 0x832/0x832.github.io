---
layout: single
title: Resolución de A4 SQL Injection Advanced-Webgoat
excerpt: "En este artículo veremos cómo resolver los ejercicios SQL Injection Advanced del apartado A4 de Webgoat de forma rápida y fácil"
date: 2025-12-13
classes: wide
header:
  teaser: /assets/images/2025-12-13-A3-Injection-advanced/sqlintrro.png
  teaser_home_page: true
categories:
  - SQL Injection
  - WebGoat
tags:
  - SQL Injection
  - Advanced
---

![](/assets/images/2025-12-13-A3-Injection-advanced/sqlintrro.png)

En este artículo veremos cómo resolver los ejercicios SQL Injection Advanced del apartados A4 de Webgoat de forma rápida y fácil

---

## Ejercicio 3
En este ejercicio tendremos que conseguir los datos de una tabla sin permisos usando otra tabla con permisos

Empezaremos con un **'or 1=1 --**

![](/assets/images/2025-12-13-A3-Injection-advanced/sql5.8.png)

![](/assets/images/2025-12-13-A3-Injection-advanced/sql3.1.png)

```sql
;SELECT * FROM users_data where last_name= 'or 1=1; select * from user_system_data;--
```
![](/assets/images/2025-12-13-A3-Injection-advanced/sql5.9.png)


## Ejercicio 5
En este ejercicio tendremos que detectar el campo vulnerable, en este caso es el apartado de **username**. 

![](/assets/images/2025-12-13-A3-Injection-advanced/sql5.1.png)


Si miramos las pistas vemos que nos dice que podemos utilizar el 
![](/assets/images/2025-12-13-A3-Injection-advanced/sql5.2.png)

```sql
  tom' AND substring(password,1,1)='a 
```
![](/assets/images/2025-12-13-A3-Injection-advanced/sql5.3.png)

Si se investiga un poco vemos que es SQL Blind así que tendremos que aplicar la sentencia SQL probando letras ya que una letra correcta podría dar una respuesta diferente

![](/assets/images/2025-12-13-A3-Injection-advanced/sql5.4.png)

Para poder agilizar este proceso usaremos **Burp** 
Asumiré que ya saben cómo se utiliza Burp ya que es algo esencial y básico

Lo que haremos será interceptar la petición y mandarlo al Intruder creando una variable a la letra que hemos introducido en la consulta **tom' AND substring(password,1,1)='a** y lanzar un ataque de fuerza bruta fijándonos en el **length** de la respuesta (Esto solo aplicará al primer carácter de la contraseña, para cambiar al carácter 2 tendremos que cambiar el número que apunta la flecha azul)
![](/assets/images/2025-12-13-A3-Injection-advanced/sql5.5.png)

![](/assets/images/2025-12-13-A3-Injection-advanced/sql5.6.png)

Una vez tenemos la contraseña debería verse algo así 

![](/assets/images/2025-12-13-A3-Injection-advanced/sql5.7.png)

## Finalización

Espero que hayan aprendido mucho y que esta explicación les haya servido para entender mejor cómo funcionan las SQL injection. Muchas gracias por leer el artículo, y no olviden seguirme en Github <a href="https://github.com/0x832">0x832</a> además iré subiendo resoluciones de máquinas de HTB, TryHackMe y más herramientas que vaya desarrollando.

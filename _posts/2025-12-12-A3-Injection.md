---
layout: single
title: Resolución de A3 SQL injection introduction Webgoat
excerpt: "En este artículo veremos cómo resolver la introducción de sql injection del apartado A3 de Webgoat de forma rápida y fácil"
date: 2025-12-12
classes: wide
header:
  teaser: /assets/images/2025-12-12-Webgoat-A4/introduccion.png
  teaser_home_page: true
categories:
  - SQL Injection
  - WebGoat
tags:
  - SQL Injection
  - Easy
---

![](/assets/images/2025-12-12-Webgoat-A4/introduccion.png)

En este artículo veremos cómo resolver la introducción de sql injection del apartado A3 de Webgoat de forma rápida y fácil
---

# SQL Injection(Intro)

SQL es un lenguaje de programación estandarizado (ANSI en 1986, ISO en 1987) que se utiliza para gestionar bases de datos relacionales y realizar diversas operaciones sobre los datos que contienen.

Una base de datos es una colección de datos. Estos se organizan en filas, columnas y tablas, y se indexan para facilitar la búsqueda de información relevante.

## Ejercicio 2
Para resolver este Ejercicio simplemente tenemos que hacer una consulta a la tabla **Employees** para seleccionar el departamento 

![](/assets/images/2025-12-12-Webgoat-A4/sql2.1.png)

```sql
SELECT department FROM employees WHERE first_name = 'Bob'
```
![](/assets/images/2025-12-12-Webgoat-A4/sql2.png)

## Ejercicio 3
Para resolver este Ejercicio simplemente tenemos que cambiar el departamento de Tobi Barnett a Ventas

![](/assets/images/2025-12-12-Webgoat-A4/sql3.1.png)

```sql
UPDATE employees SET department = 'Sales' WHERE first_name = 'Tobi';

```
![](/assets/images/2025-12-12-Webgoat-A4/sql3.png)

## Ejercicio 4

Para resolver este Ejercicio simplemente tenemos que añadir el campo phone en la tabla employees. Como podemos ver, los ejercicios consisten en consultas SQL para acostumbrarnos a la dinámica

![](/assets/images/2025-12-12-Webgoat-A4/sql4.1.png)


```sql
ALTER TABLE employees ADD COLUMN phone VARCHAR(20);
```
![](/assets/images/2025-12-12-Webgoat-A4/sql4.png)


## Ejercicio 5 
Para resolver este Ejercicio simplemente tenemos que dar permisos al usuario **authorized_user** a la tabla **grant_rights**
![](/assets/images/2025-12-12-Webgoat-A4/sql5.1.png)
```sql 
GRANT SELECT ON grant_rights TO authorized_user;

```
![](/assets/images/2025-12-12-Webgoat-A4/sql5.png)


## Ejercicio  9 
Una vez hemos realizado las primeras sentencias SQL tocará practicar cómo hacer SQL Injection
![](/assets/images/2025-12-12-Webgoat-A4/sql9.1.png)

Para ver cuál tiene una respuesta que llame la atención
![](/assets/images/2025-12-12-Webgoat-A4/sql9.png)

Cuando en el formulario escribimos  **smith' OR '1'='1** lo que hacemos es romper la consulta SQL original y agregar una condición que siempre es verdadera.

Lo que quiere decir esto es que la consulta normal sería algo cómo 

```sql
SELECT * FROM user_data 
WHERE first_name='John' AND last_name='smith';
```
Pero con la inyección sería algo cómo 

```sql 
SELECT * FROM user_data 
WHERE first_name='John' AND last_name='smith' OR '1'='1';
```
La parte '1'='1' siempre es verdadera, por lo tanto, la condición del WHERE se vuelve verdadera para todos los registros y la base de datos devuelve todas las filas, aunque el apellido no sea **smith**


## Ejercicio  10

En este punto tendrémos que detectar qué entrada es vulnerable a un SQL Injection

![](/assets/images/2025-12-12-Webgoat-A4/sql10.1.png)


Para descubrir el campo vulnerable simplemente tendrémos que poner **1='1'** en uno de los dos campos para ver cual tiene una respuesta que llame la atención 

Una vez detectamos el campo vulnerable lanzaremos el ataque 

```sql
0 OR 1234=1234
```
El **OR 1 = 1** es una condición que siempre será verdadera, ya que 1 siempre es igual a 1. Esto significa que, sin importar lo que esté en el resto de la consulta, la consulta siempre devolverá todos los registros.

![](/assets/images/2025-12-12-Webgoat-A4/sql10.png)


## Ejercicio  11

Tendremos que descubrir el salario de los trabajadores
![](/assets/images/2025-12-12-Webgoat-A4/sql11.1.png)

Para poder realizar este apartado tendremos que hacer como antes, encontrar el campo vulnerable y atacarlo

```sql
3SL99A' OR '1' = '1
```
![](/assets/images/2025-12-12-Webgoat-A4/sql11.png)


## Ejercicio  12
Aquí tendremos que concatenar consultas para modificar el salario

![](/assets/images/2025-12-12-Webgoat-A4/sql12.1.png)


El punto y coma ; separa dos consultas SQL. Lo que pasa es que en muchas aplicaciones vulnerables a inyección SQL, no se valida correctamente la entrada del usuario. Esto permite que el atacante ejecute múltiples consultas SQL en una sola ejecución

```sql
' OR 1=1; UPDATE employees SET SALARY = 99999 WHERE FIRST_NAME = 'John'; --
```
![](/assets/images/2025-12-12-Webgoat-A4/sql12.png)


## Ejercicio  13

En esta práctica aplicaremos una consulta SQL para borrar una tabla; si leemos la práctica, nos pide que borremos la tabla access_logs
![](/assets/images/2025-12-12-Webgoat-A4/sql13.1.png)

```sql
'; DROP TABLE access_log; --

```

![](/assets/images/2025-12-12-Webgoat-A4/sql13.png)



## Finalización

Espero que hayan aprendido mucho y que esta explicación les haya servido para entender mejor cómo funcionan las SQL injection. Muchas gracias por leer el artículo, y no olviden seguirme en Github <a href="https://github.com/0x832">0x832</a> además iré subiendo resoluciones de máquinas de HTB, TryHackMe y más herramientas que vaya desarrollando.

---
layout: single
title: Editorial - Easy
excerpt: "
Hola a todos, hoy les presentaré la resolución de una máquina **Hack the box de dificultad Easy**. En esta máquina aprenderemos a detectar un **SSRF** para luego conectarnos por **SSH** y llegar a ser **root abusando de privilegios a nivel de Sudoers**"

date: 2024-08-7
classes: wide
header:
  teaser: /assets/images/2024-08-07-editorial-easy/Editorial.png
  teaser_home_page: true

categories:
  - Hack the box 
  - Pentesting
  - CTF
tags:  
  - SSRF
  - SSH 
  - ABUSO DE SUDOERS
  - Hack the box 

---

![](/assets/images/2024-08-07-editorial-easy/Editorial.png)

Hola a todos, hoy les presentaré la resolución de una máquina **Hack the box de dificultad Easy**. En esta máquina aprenderemos a detectar un **SSRF** para luego conectarnos por **SSH** y llegar a ser **root abusando de privilegios a nivel de Sudoers**"

<br>

# Reconocimiento
- Enumeración con nmap
    
    Hacemos un `nmap` simple para ver los puertos que corren en la máquina
    
    ```python
    # nmap -p- -sS --min-rate 5000 -T5 -Pn -vvv -oN nmap.txt 10.10.11.20
    Increasing send delay for 10.10.11.20 from 0 to 5 due to 235 out of 587 dropped probes since last increase.
    Warning: 10.10.11.20 giving up on port because retransmission cap hit (2).
    Nmap scan report for 10.10.11.20
    Host is up, received user-set (0.61s latency).
    Scanned at 2024-08-06 00:05:27 CEST for 39s
    Not shown: 55285 closed tcp ports (reset), 10248 filtered tcp ports (no-response)
    PORT   STATE SERVICE REASON
    22/tcp open  ssh     syn-ack ttl 63
    80/tcp open  http    syn-ack ttl 63
    
    Read data files from: /usr/bin/../share/nmap
    ```
    
  Una vez acaba el escaneo vemos los puertos abiertos, que en este caso como podemos ver es el `22(SSH)` y el `80(HTTP)` 

  Ahora  le aplicamos un `nmap` más exhaustivo a dichos puertos

    ```python
    #nmap -p22,80 -sVC -vvv -oN port.txt 10.10.11.19
    Nmap scan report for 10.10.11.19
    Host is up, received echo-reply ttl 63 (1.2s latency).
    Scanned at 2024-08-06 00:07:56 CEST for 59s
    
    PORT   STATE SERVICE    REASON         VERSION
    22/tcp open  tcpwrapped syn-ack ttl 63
    | ssh-hostkey:
    |   3072 3e:21:d5:dc:2e:61:eb:8f:a6:3b:24:2a:b7:1c:05:d3 (RSA)
    | ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC0B2izYdzgANpvBJW4Ym5zGRggYqa8smNlnRrVK6IuBtHzdlKgcFf+Gw0kSgJEouRe8eyVV9iAyD9HXM2L0N/17+rIZkSmdZPQi8chG/PyZ+H1FqcFB2LyxrynHCBLPTWyuN/tXkaVoDH/aZd1gn9QrbUjSVo9mfEEnUduO5Abf1mnBnkt3gLfBWKq1P1uBRZoAR3EYDiYCHbuYz30rhWR8SgE7CaNlwwZxDxYzJGFsKpKbR+t7ScsviVnbfEwPDWZVEmVEd0XYp1wb5usqWz2k7AMuzDpCyI8klc84aWVqllmLml443PDMIh1Ud2vUnze3FfYcBOo7DiJg7JkEWpcLa6iTModTaeA1tLSUJi3OYJoglW0xbx71di3141pDyROjnIpk/K45zR6CbdRSSqImPPXyo3UrkwFTPrSQbSZfeKzAKVDZxrVKq+rYtd+DWESp4nUdat0TXCgefpSkGfdGLxPZzFg0cQ/IF1cIyfzo1gicwVcLm4iRD9umBFaM2E=
    |   256 39:11:42:3f:0c:25:00:08:d7:2f:1b:51:e0:43:9d:85 (ECDSA)
    | ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBFMB/Pupk38CIbFpK4/RYPqDnnx8F2SGfhzlD32riRsRQwdf19KpqW9Cfpp2xDYZDhA3OeLV36bV5cdnl07bSsw=
    |   256 b0:6f:a0:0a:9e:df:b1:7a:49:78:86:b2:35:40:ec:95 (ED25519)
    |_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOjcxHOO/Vs6yPUw6ibE6gvOuakAnmR7gTk/yE2yJA/3
    80/tcp open  tcpwrapped syn-ack ttl 63
    |_http-title: Did not follow redirect to http://app.blurry.htb/
    | http-methods:
    |_  Supported Methods: GET HEAD POST OPTIONS
    |_http-server-header: nginx/1.18.0
    
    Read data files from: /usr/bin/../share/nmap
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    ```
    
    Como podemos observar vemos que el puerto `80` aplica un `redirect` a [`http://app.blurry.htb/`](http://app.blurry.htb/) lo cual nos hace pensar que tendremos que ponerlo en el `/etc/hosts`
    
    ![Untitled](/assets/images/2024-08-07-editorial-easy/Untitled.png)
    
  ## Ahora accedemos a la web para poder determinar que clase de vulnerabilidades podemos encontrar

  Vamos a acceder a la web, ya que como hemos visto anteriormente, el `puerto 80` está abierto
    
    Podemos observar varias cosas, pero si  accedemos a esta sección de la web podemos observar esto de aquí

    ![Untitled](/assets/images/2024-08-07-editorial-easy/Untitled%202.png)
        
    Podríamos pensar en una subida de archivos, pero no es el caso, pero otra cosa que nos llama la atención es que podemos introducir una `URL` lo cual nos hace pensar  en un `SSRF`(**Server-Side Request Forgery**)    

    ![Untitled](/assets/images/2024-08-07-editorial-easy/Untitled%203.png)

        
  # Explotación del `SSRF`
        
    Empecemos rellenando el formulario de esta manera , pero ponemos http://127.0.0.1 para intentar apuntar a la máquina interna
    
    ![Untitled](/assets/images/2024-08-07-editorial-easy/Untitled%204.png)
    
    Mandamos la petición y la interceptamos con `burp`. Una vez hecho eso lo enviamos al `repeter` , al enviar la petición podemos observar que nos sale una ruta de una `imagen.jpg`
    
    ![Untitled](/assets/images/2024-08-07-editorial-easy/Untitled%205.png)
    
    Una vez sabemos que es vulnerable a `SSRF` podemos intentar listar puertos abiertos de la máquina local, para poder listar dichos puertos mandaremos la petición al `intruder`
    
    Añadiremos `:puerto_probar`y le daremos a `Add §`
    
    ![Untitled](/assets/images/2024-08-07-editorial-easy/Untitled%206.png)
    
    Nos crearemos un documento con `top 200 puertos`para probar
    
    Le daremos a `Payload` y añadiremos el fichero con los puertos
    
    ![Untitled](/assets/images/2024-08-07-editorial-easy/Untitled%207.png)
    
    Una vez configurado esto , iremos a `setting` bajaremos hasta la opción `Grep-Extract` le daremos a `Add` y seleccionaremos la ruta de la imagen para que muestre todas las respuestas que tengan esa ruta
    
    ![Untitled](/assets/images/2024-08-07-editorial-easy/Untitled%208.png)
    
    Y le daremos a `StartAttack`
    
    ![Untitled](/assets/images/2024-08-07-editorial-easy/Untitled%209.png)
    
    Una vez finaliza el ataque  podemos observar que el puerto `5000` tiene otro `Lenght` diferente
    
    ![Untitled](/assets/images/2024-08-07-editorial-easy/Untitled%2010.png)
    
    Si lo mandamos al `Repeter` y mandamos la solicitud, vemos una respuesta totalmente diferente a las demás  
    
    ![Untitled](/assets/images/2024-08-07-editorial-easy/Untitled%2011.png)
    
    Vamos a ver que es esto de aquí 
    
    Aplicamos un `curl` para descargar lo que hemos encontrado 
    
    ![Untitled](/assets/images/2024-08-07-editorial-easy/Untitled%2012.png)
    
     Una vez descargado el archivo, hacemos esta instalación 
        
    `apt install jq` para poder visualizar el archivo de mejor forma
    
    Podemos observar los puntos finales de una `API` lo cual nos puede ser útil para hacer peticiones e intentar encontrar alguna  información 
    interesante. En este caso nos llama mucho la atención de `new_authors`
        
    ![Untitled](/assets/images/2024-08-07-editorial-easy/Untitled%2013.png)
    
    Nos copiamos la ruta de `new_authors` interceptamos la petición como hicimos al principio y en vez de poner solo `5000` le añadimos la ruta.
    
    Vemos que nos da otra archivo 
    
    ![Untitled](/assets/images/2024-08-07-editorial-easy/Untitled%2014.png)
    
    Nos lo descargamos 
    
    ```python
    curl -O [http://editorial.htb/static/uploads/c51085e1-b565-4d03-8c2a-c8ed92decbea](http://editorial.htb/static/uploads/c51085e1-b565-4d03-8c2a-c8ed92decbea)
    ```
    
    ![Untitled](/assets/images/2024-08-07-editorial-easy/Untitled%2015.png)
    
    Y vemos que hay un `user` y `password`
    
  # Acceso a la máquina con ssh

    Con la contraseña y password encontrada podemos intentar acceder al ssh.

    Y efectivamente estamos dentro
    
    ![Untitled](/assets/images/2024-08-07-editorial-easy/Untitled%2016.png)
    
    Aplicamos un `ls -la` y encontramos el `user.txt`, ya tenemos la primera **flag**
        
    ![Untitled](/assets/images/2024-08-07-editorial-easy/Untitled%2017.png)
        
  # Escalada a `root`
        
    Vemos una carpeta llamada `apps` lo cual entramos por curiosidad

    ![Untitled](/assets/images/2024-08-07-editorial-easy/Untitled%2019.png)

    Y encontramos una carpeta llamada `.git` lo cual podría haber información interesante

    ![Untitled](/assets/images/2024-08-07-editorial-easy/Untitled%2020.png)

    ![Untitled](/assets/images/2024-08-07-editorial-easy/Untitled%2021.png)

    Si nos ponemos a mirar los logs del `.git` aplicando el comando `git log` encontramos mucha información, pero si miramos un poco vemos algo que nos llama la atención, que nos dice que puede haber **información de acceso a la editorial**

    ![Untitled](/assets/images/2024-08-07-editorial-easy/Untitled%2022.png)

    
    `git show 1e84a036b2f33c59e2390730699a488c65643d28` vemos que nos da bastante información, pero si vamos bajando acabamos encontrando esto de aquí un `user` y una `password`
    
    ![Untitled](/assets/images/2024-08-07-editorial-easy/Untitled%2023.png)
    
    ![Untitled](/assets/images/2024-08-07-editorial-easy/Untitled%2024.png)
    
    Probamos si esas credenciales funcionan en el ssh `usuario y password` y efectivamente funcionan
    
    ![Untitled](/assets/images/2024-08-07-editorial-easy/Untitled%2024.5.png)
    
    Si aplicamos un `sudo -l` nos pedirá `password` , pero pondremos la de usuario
    
    Podemos observar que hay posibilidad de `escalar a root abusando de privilegios a nivel de Sudoers`

    ![Untitled](/assets/images/2024-08-07-editorial-easy/Untitled%2025.png)
    
    Nos dirigimos a la ruta del programa Python
    
    ![Untitled](/assets/images/2024-08-07-editorial-easy/Untitled%2026.png)
    
    Vemos que el script utiliza `bibliotecas git` para realizar la operación de clonación
    
    ![Untitled](/assets/images/2024-08-07-editorial-easy/Untitled%2027.png)
    
    Si miramos la `versión de gitpython` que usa el programa vemos esto 
    
    ![Untitled](/assets/images/2024-08-07-editorial-easy/Untitled%2028.png)
    
    Para poder explotar la vulnerabilidad aplicaremos este comando, que lo que hace es ejecutar el programa a nivel de sudoers y ejecuta una Shell que copia el ``/root/root.txt`` en ``/tmp/root``
    
    ```python
    sudo /usr/bin/python3 /opt/internal_apps/clone_changes/clone_prod_change.py 'ext::sh -c cat% /root/root.txt% >% /tmp/root'    
    ```
        
    ![Untitled](/assets/images/2024-08-07-editorial-easy/Untitled%2029.png)
    
    Y si hacemos un `ls /tmp/root` dentro del archivo `root` tenemos `flag`
    
    ![Untitled](/assets/images/2024-08-07-editorial-easy/Untitled%2030.png)
    
  # Finalización 
    Espero que hayan aprendido mucho haciendo esta máquina y les haya servido de ayuda mi explicación para poder entender esta **CTF**. Muchas gracias por leer el artículo y no olviden seguirme en <a href="https://github.com/0x832/">GitHub</a>, ya que también iré subiendo herramientas de hacking.

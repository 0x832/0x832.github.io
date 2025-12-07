---
layout: single
title: Evil IntrudeR - Bash Script para Auditoría Wi-Fi
excerpt: "Hoy presentaré el análisis de una herramienta que desarrollé en Bash para automatizar tareas de auditoría Wi-Fi con airmon-ng, macchanger, john y cowpatty. Explicaré de forma breve cómo funciona esta herramienta y sus funciones."
date: 2025-11-11
classes: wide
header:
  teaser: /assets/images/2025-11-11-Evil-Intruder-python-scripting/evil-intruder.png
  teaser_home_page: true

categories:
  - Pentesting
  - Bash
  - Ciberseguridad
tags:
  - Auditoría Wi-Fi
  - Bash
  - Automatización
---


![](/assets/images/2025-11-11-Evil-Intruder-python-scripting/evil-intruder.png)

# Introducción

Hoy presentaré el análisis de una herramienta que desarrollé en Bash para automatizar tareas de auditoría Wi-Fi con airmon-ng, macchanger, john y cowpatty. Explicaré de forma breve cómo funciona esta herramienta y sus funciones

Este script reúne y automatiza herramientas como:

- **airmon-ng** → Activar modo monitor.  
- **airodump-ng** → Captura de tráfico.  
- **aireplay-ng** → Pruebas de desautenticación.  
- **macchanger** → Cambio de MAC.  
- **tshark** → Análisis de tráfico DNS/HTTP.  
- **airdecap-ng** → Descifrado de capturas WPA/WPA2.  
- **cowpatty / genpmk / john / aircrack-ng** → Ataques de diccionario sobre handshakes.

**Aviso ético:** Todo lo que se explica debe realizarse exclusivamente en entornos controlados (laboratorios, CTFs) y con autorización. El objetivo es aprender cómo funcionan estas técnicas, no aplicarlas de forma indebida.

---

## Reconocimiento del Script

Estructura general del programa:

- **Menú interactivo** con opciones numeradas.  
- **Comprobación de privilegios** (`root()`).  
- **Funciones modulares** para cada tarea (activar modo monitor, escaneo, análisis, etc.).  
- **Automatización** de comandos complejos para reducir errores.

El flujo principal está en la función `main()`, que coordina la ejecución según la opción elegida.

---

## Funciones Clave y Herramientas Automatizadas

### 1. **root()**
Comprueba si el usuario tiene permisos de superusuario (`EUID`).  
- Herramientas como `airmon-ng` necesitan acceso al hardware.  
- Por eso, si no somos root, la herramienta lo solicita.

---

### 2. **inicio()**
Presenta un menú con opciones como:
- Activar modo monitor.  
- Escanear redes.  
- Ejecutar ataques simulados.  
- Analizar capturas.

**Automatización:**  
Gracias a este menú evitamos escribir comandos manualmente y reducimos errores al trabajar.

---

## 3. **modo_monitor1() y activar_modo_monitor()**
Activa el **modo monitor** en la tarjeta de red mediante `airmon-ng`.  
Opcionalmente permite cambiar la MAC con `macchanger`.

**Concepto de ciberseguridad:**  
- El modo monitor sirve para capturar tráfico inalámbrico.  
- Cambiar la MAC ayuda a mantener anonimato durante pruebas.

---

## 4. **scan_wifis3() y scan_wifi_en_concreto4()**
Automatiza `airodump-ng` para:
- Escanear todas las redes cercanas.  
- Capturar tráfico de una red concreta (BSSID/ESSID + canal).

---

## 5. **Ataque_Beacon_Flood5()**
Genera múltiples AP falsos con `mdk3`.

**Objetivo:**  
- Analizar ataques de denegación de servicio en Wi-Fi.  
- Sobrecargar el canal hasta provocar saturación en las redes del entorno.

---

## 6. **expultar_a_la_gente6()**
Simula desautenticación mediante `aireplay-ng`.  

Esta función permite expulsar temporalmente a usuarios de una red para capturar el handshake.

---

## 7. **brutal_force7()**
Integra varias herramientas:
- `aircrack-ng`, `john`, `cowpatty`, `genpmk`.

**Propósito:**  
- Auditar contraseñas débiles en entornos controlados.

---

## 8. **analisi_De_captura()**
Descifra tráfico con `airdecap-ng` y analiza DNS/HTTP con `tshark`.

Permite evaluar riesgos en redes con cifrado débil o mal configurado.

---

## Herramientas Automatizadas y su Rol

| Herramienta      | Función principal                                  |
|-------------------|---------------------------------------------------|
| **airmon-ng**     | Activar/desactivar modo monitor                   |
| **airodump-ng**   | Captura de tráfico Wi-Fi                          |
| **aireplay-ng**   | Inyección de paquetes (desautenticación)          |
| **macchanger**    | Cambio de dirección MAC                           |
| **aircrack-ng**   | Ataque de diccionario sobre handshakes WPA/WPA2   |
| **cowpatty**      | Fuerza bruta con PMK                              |
| **genpmk**        | Generación de diccionarios precomputados          |
| **john**          | Cracking avanzado de hashes WPA                   |
| **tshark**        | Análisis de tráfico DNS/HTTP                      |
| **airdecap-ng**   | Descifrado de capturas                            |

---

## Buenas prácticas y defensa

- **WPA3** y contraseñas robustas.  
- Deshabilitar **WPS**.  
- Implementar **IDS/IPS** para redes inalámbricas.  
- Cifrado DNS y uso estricto de **HTTPS**.

---

## Finalización

Espero que hayan aprendido mucho y que esta explicación les haya servido para entender mejor cómo se puede comprometer una red Wi-Fi en un entorno controlado. Muchas gracias por leer el artículo, y no olviden seguirme en Github. no obstante si quieren descargar mi tool les dejó aquí el acceso directo 
<a href="https://github.com/0x832/Evil-Intruder">evil-intruder</a>: además iré subiendo resoluciones de máquinas de HTB, TryHackMe y más herramientas que vaya desarrollando.


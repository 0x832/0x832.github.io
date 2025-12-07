---
layout: single
title: "ARP & IP Spoofing: Ataques, Conceptos y Seguridad"
excerpt: "En este artículo veremos cómo funcionan los ataques basados en ARP Spoofing e IP Spoofing, técnicas de explotación y cómo protegernos. Además, abalremos sobre los protocolos Wi-Fi como TKIP vs AES y herramientas como aircrack-ng."
date: 2025-11-23
classes: wide
header:
  teaser: /assets/images/2025-11-23-arp-ip-spoofing/teaser.png
  teaser_home_page: true

categories:
  - Redes
  - Pentesting

tags:
  - ARP
  - IP Spoofing
  - MiTM
  - Wi-Fi
  - Cracking
  - Aircrack-ng2025-11-23-arp-ip-spoofing
---


![](/assets/images/2025-11-23-arp-ip-spoofing/teaser.png)

# Introducción

En este artículo veremos cómo funcionan los ataques basados en ARP Spoofing e IP Spoofing, técnicas de explotación y cómo protegernos. Además, abalremos sobre los protocolos Wi-Fi como TKIP vs AES y herramientas como aircrack-ng

- **Qué es ARP Spoofing**
- **Qué es IP Spoofing y cómo se usa en DoS/DDos**
- **Diferencias entre TKIP y AES en seguridad Wi-Fi**
- **Herramientas prácticas para auditoría inalámbrica**
<br>

---

# ARP Spoofing

El ARP Spoofing consiste en **engañar a la red asociando direcciones IP con MAC falsas**.

> *Objetivo: Modificar la IP ↔ MAC para interceptar tráfico en una LAN.*

### Características importantes

- Afecta a **Capa 2 (MAC)** y **Capa 3 (IP)**
- ARP **no autentica respuestas**
- Cualquier *ARP Reply* puede almacenarse en caché
- Permite ataques **Man-in-the-Middle**, DoS o sniffing

Ejemplo de escenario:

Víctima → (Cree que la MAC del atacante es la puerta de enlace)

## Herramientas

| Herramienta | Uso |
|------------|-----|
| **Ettercap** | Ataques MITM mediante ARP Spoofing |
| **arpspoof (dsniff)** | Manipulación ARP directa |
| **Wireshark** | Inspección de tráfico interceptado |

---

# IP Spoofing

El IP Spoofing consiste en **falsificar la dirección IP de origen** en los paquetes enviados, simulando que provienen de una máquina diferente.

## Objetivos

- Ocultar identidad del atacante
- Realizar ataques DoS y DDoS
- Facilitar ataques MITM en entornos sin control estricto

Se aplica en:

- **Capa 3 – Protocolo IP**
- No se valida origen

---

## Cómo protegernos

**Rate Limiting**

Limitar paquetes por IP para evitar saturación:


**Prevenir DDoS y análisis de tráfico**

- Configurar el firewall o dispositivo de la red para limitar la cantidad de paquetes entrrantes por segundo de una IP 

---

# Seguridad Wi-Fi: TKIP vs AES

## TKIP (Temporal Key Integrity Protocol)

- Introducido como sustitución temporal de WEP
- Hoy en día es  **inseguro**

## AES (Advanced Encryption Standard)

- Algoritmo moderno de cifrado 
- Base de **WPA2 y WPA3**
- Mucho más seguro que el TKIP

---

# Herramientas Wi-Fi (Aircrack-ng)

| Herramienta | Función principal | Ejemplo |
|-------------|------------------|---------|
| **airmon-ng** | Activar modo monitor | `airmon-ng start wlan0` |
| **airodump-ng** | Captura tráfico Wi-Fi | `airodump-ng -w captura wlan0mon` |
| **aireplay-ng** | Inyección / deauth | `aireplay-ng -0 200 -e ESSID -c MAC wlan0mon` |
| **aircrack-ng** | Ataque por diccionario | `aircrack-ng -w wordlist captura.cap` |

---

# Hasta aquí llega nuestro artículo explicativo sobre cómo crear Command&control.

 Espero que les haya servido la explicación y que haya sido clara. No olviden seguirme en <a href="https://github.com/0x832/">GitHub</a>, ya que iré subiendo más artículos sobre ciberseguridad.

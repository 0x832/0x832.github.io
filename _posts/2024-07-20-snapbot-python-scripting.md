---
layout: single
title: SnapBot - Python scripting
excerpt: "
Hola a todos, hoy les presentaré un pequeño script en Python que desarrollé hace unos días, llamado SnapBot . Este programa está diseñado para capturar imágenes utilizando la cámara web de tu ordenador y enviarlas directamente a un chat de Telegram que hayas configurado previamente. "

date: 2024-07-12
classes: wide
header:
  teaser: /assets/images/2024-07-20-script-python-SnapBot/SnapBot1.jpg
  teaser_home_page: true

categories:
  - Programación
tags:  
  - Python
  - Programación
---

![](/assets/images/2024-07-20-script-python-SnapBot/SnapBot1.jpg)

Hola a todos, hoy enseñaré un pequeño script en Python que desarrollé hace unos días, llamado SnapBot . Este programa está echo para capturar una imágen utilizando la cámara web de tu ordenador y enviarlas a un chat de Telegram que hayas configurado anteriormente. 
<br>

# Requisitos
- Primero, necesitaremos crear tres archivos: **SnapBot.pyw, config.py y webcam.py.**

  - SnapBot.pyw: Con este script se iniciará el programa
  - config.py: Con este otro sé configurará el TOKEN y el ID para que nos llegue la foto al Telegram
  - webcam.py: Y en este script escribiremos la mayoría del código
<br>

# Desarrollo del Script 
### **1. Configuración del config.py :** ###

  **Descripción:**
  Este archivo contiene el token de autenticación y el ID del chat.
<br>

  Primero, vamos a configurar el archivo **config.py** para definir el **TOKEN** y el **CHAT_ID** necesarios para enviar la foto a Telegram.

  ```python 
      def get_telegram_config():
            return {
            'TOKEN': '', # Token de autenticación de la API de Telegram
            'CHAT_ID': '' # Identificador del chat al que se enviará la foto
            }
  ```
  <br>

### **2 Configuración de webcam.py** ###

  **Descripción:**
  Este script se encarga de obtener una imagen desde la cámara web y enviarla a Telegram. Aquí es donde está la mayor parte del código.
  <br>

  - ### **1- Entramos al webcam.py para importación de Librerías :**
      En esta sección se importan las librerias necesarias para que el script funcione.
          
    ```python 
      import cv2 
      import io 
      import requests 
      from config import get_telegram_config 
    ```
    - **Import cv2:** Nos sirve para capturar y procesar imágenes desde la cámara web. 

    - **Import io:** Sirve para manejar datos de imagen en memoria. 

    - **Import requests:** Para enviar la imagen a Telegram a través de solicitudes HTTP

    - **Import from config import get_telegram_config:** Para obtener la configuración del ``config.py``
<br>

  - ### **2- Llamada al config.py:**
    En esta sección, explicaremos cómo se utiliza la función ``get_telegram_config()`` para obtener la configuración necesaria para enviar la imagen a Telegram.

    ``` python 
        def Captur_webcam():
            telegram_config = get_telegram_config()
            TOKEN = telegram_config['TOKEN']
            CHAT_ID = telegram_config['CHAT_ID']
    ```
    - **get_telegram_config():** Esta función, que se encuentra en ``config.py``, devuelve los datos con el ``TOKEN`` y el ``CHAT_ID`` necesarios para interactuar con la API de Telegram.

    - **telegram_config:** Cuando llamas a `get_telegram_config()`, obtienes la configuración de Telegram.

    -  **TOKEN:** Se obtiene la configuración ``telegram_config`` para autenticar el bot en la API de Telegram.

    -  **CHAT_ID:** Se obtiene la configuración ``telegram_config`` para especificar el chat al que se enviará la imagen.
<br>

  - ### **3- Captura de imagen con la webcam:**
    A continuación, explicaremos cómo usar ``OpenCV`` para tomar una foto con la webcam.

    ```python 
        cap = cv2.VideoCapture(0)
        for i in range(30):
            cap.read()
        ret, frame = cap.read()
    ```
    - **cv2.VideoCapture(0):** Inicia la webcam del PC.

    - **for i in range(30): cap.read():** Realiza 30 lecturas iniciales para iniciar la cámara, ajustando parámetros automáticos como la exposición y el balance de blancos.

    - **ret, frame = cap.read():** Captura una imagen de la webcam, donde ``ret`` indica si la captura fue exitosa y ``frame`` contiene la imagen obtenida.
<br>

  - ### **4- Codificación y almacenamiento de la imagen:**
    En esta sección, detallaremos el proceso de codificación y almacenamiento de la imagen capturada desde la webcam utilizando OpenCV y la biblioteca io.

    ```python  
        buffer = io.BytesIO()
        _, encoded_image = cv2.imencode('.jpg', frame)
        buffer.write(encoded_image.tobytes())
        cap.release()
    ```

    - **buffer = io.BytesIO():** Crea un ``buffer en memoria`` que actuará como un archivo para almacenar la imagen.

    - **cv2.imencode('.jpg', frame):** Codifica la imagen capturada ``(frame)`` en formato JPEG.

    - **buffer.write(encoded_image.tobytes()):** Escribe los datos de la imagen codificada en el ``buffer``, convirtiendo primero los datos a ``bytes`` con ``tobytes()``.

    - **cap.release():** Cierra la conexión con la webcam.
<br>

  - ### **5- Envío de la imagen a Telegram:**
    En este apartado, explicaré cómo envía la imagen capturada a Telegram utilizando la API de Telegram .

    ```python  
        archivos = {'photo': ('webcam.jpg', buffer.getvalue())}
        datos = {'chat_id': CHAT_ID}
        url = f'https://api.telegram.org/bot{TOKEN}/sendPhoto'

        respuesta = requests.post(url, data=datos, files=archivos))
    ```
    - **archivos = {'photo': ('webcam.jpg', buffer.getvalue())}:** Prepara la imagen para enviar. 'photo' es el nombre del campo esperado por Telegram. 'webcam.jpg' es el nombre del archivo, y ``buffer.getvalue()`` obtiene los datos de la imagen.

    - **datos = {'chat_id': CHAT_ID}:** Especifica el identificador del chat ``(CHAT_ID)`` al que se enviará la imagen.
    
    - **url = f'https://api.telegram.org/bot{TOKEN}/sendPhoto':** Crea la ``URL`` para enviar la foto usando el token del bot ``(TOKEN).``
<br>

  - ### **6- Verificación del envío:**
    ```python  
        if respuesta.status_code == 200:
            print("Foto de la webcam enviada a Telegram")
        else:
            print("Error al enviar la foto a Telegram.")
    ```

    - **if respuesta.status_code == 200:** Verifica si la solicitud fue exitosa. El código de estado 200 indica que la imagen se envió correctamente.

    - **print("Foto de la webcam enviada a Telegram"):** Muestra un mensaje de éxito si la solicitud fue exitosa.

    - **else:** Si el código de estado no es 200, indica que hubo un error.

    - **print("Error al enviar la foto a Telegram."):** Muestra un mensaje de error si la solicitud falló.
<br>

 ¡Perfecto! Una vez finalizada está parte, vamos a configurar el script que ejecutará todo el proceso.

### **8. Configuración en SnapBot.py:** ##

  **Descripción:**
  Este es el script que ejecuta el programa. Su extensión ``.pyw`` sirve para qué se ejecutará sin abrir una ventana de consola en Windows.
  <br>

  Ahora, configuraremos el archivo **SnapBot.pyw** para gestionar la captura de imágenes desde la cámara web.

  ```python 
    import webcam 

    def main():
        try:
            webcam.Captur_webcam() 
        except Exception as e:
            pass
    if __name__ == "__main__":
        main()
  ```

  **Import webcam:** Se importa el módulo webcam para acceder a las funciones definidas en ``webcam.py``.

  **main():** Dentro de esta función, se llama a 
  **webcam.Captur_webcam():** ``webcam:`` Hace referencia al archivo ``webcam.py``
     
  **Captur_webcam():** Es una función definida dentro del archivo ``webcam.py`` que se encarga de capturar la imagen desde la cámara web.

<br>

# Hasta aquí llega nuestro artículo explicativo sobre cómo crear SnapBot.

 Espero que les haya servido la explicación y que haya sido clara. No olviden seguirme en <a href="https://github.com/0x832/">GitHub</a>, ya que iré subiendo nuevos repositorios. Si tienen alguna duda o no saben cómo hacer el script, pueden descargarlo desde <a href="https://github.com/0x832/SnapBot">mi repositorio</a>. 


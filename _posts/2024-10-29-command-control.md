---
layout: single
title: Command & Control - Python scripting
excerpt: "
Hola a todos, hoy les presentaré un pequeño script en Python que desarrollé hace unos días, llamado Command and Control. Su función principal es permitir la ejecución remota de diferentes funciones del sistema Windows a través de un bot de Telegram."

date: 2024-10-29
classes: wide
header:
  teaser: /assets/images/2024-10-29-command-control/telegram.jpg
  teaser_home_page: true

categories:
  - Programación
tags:  
  - Python
  - Programación
  - Telegram bot
---

![](/assets/images/2024-10-29-command-control/telegram.jpg)

Hola a todos, hoy les presentaré un pequeño script en Python que desarrollé hace unos días, llamado Command and Control. Su función principal es permitir la ejecución remota de diferentes funciones del sistema Windows a través de un bot de Telegram
<br>

# Requisitos
- Necesitaremos tener BotFeather, Get My ID

  - **BotFeather**: Utilizaremos BotFeather para crear y tener el token del bot
  - **Get MY ID**: Será necesario para saber cuál es nuestro ID y poder configurar el bot de forma correcta.

  - Para poder encontrarlo es necesario entrar a Telegram y buscar por **BotFeather** **Get MY ID** en el buscador

    ![](/assets/images/2024-10-29-command-control/imagen.png)

<br>

# Desarrollo del Script 
### **1. Configuración del Command and Control :** ###

  **Descripción:**
  Esta sección del script contiene todo el código necesario para configurar y establecer el sistema de Command and Control
  <br>

  Primero, vamos a importar las librerías necesarias para que el script funcione correctamente.

  ```python 
    import telegram 
    from telegram.ext import Application, CommandHandler 
    import os 
    from functools import wraps 
    import pyautogui 

  ```
  <br>
  
  - **import telegram:** Para la comunicación con la API de Telegram

  - **from telegram.ext import Application, CommandHandler:** Para la comunicación directa al bot de 

      telegram tanto respuesta como comandos directos que se le enviaran 

  - **import os:** Para que el programa pueda comunicarse directamente al sistema operativo

  - **from functools import wraps:** Para guardar metadatos de funciones al crear los decoradores

  - **import pyautogui:** Para la comunicación con la interfaz gráfica


### **2- Configuración del Token ,ID del bot :**
  En esta sección escribiremos tanto nuestro TOKEN como nuestro ID de Telegram.

  ```python 
    TOKEN = 'TOKEN_BOT'
    AUTHORIZED_USERS = [tu_id]  

  ```
  -  **TOKEN:** Se obtiene la configuración ``telegram_config`` para autenticar el bot en la API de Telegram.

  -  **CHAT_ID:** Se obtiene la configuración ``telegram_config`` para especificar el chat al que se enviará la imagen.
  <br>


### **3- Validador de ID telegram:**
  En esta parte del script se creará el decorador, el cual sirve para modificar o extender el funcionamiento de una función o clase sin modificar directamente el código. Lo que hará este decorador será validar si quien intenta entrar en tu ChatBot está autorizado o no, para mayor seguridad 
```python 

    def authorized_only(func):
        @wraps(func)
        async def wrapped(update, context, *args, **kwargs):
            user_id = update.effective_user.id  
            if user_id not in AUTHORIZED_USERS: 
                await update.message.reply_text('❌No tienes permiso para usar este comando❌')
                return  
            return await func(update, context, *args, **kwargs)  
        return wrapped 
  ```

  <br>

### **4- Funciones de Control del Sistema Operativo  :**
  En esta sección están los comandos que ejecutará el sistema a través de las órdenes que reciba del bot.

  ```python  

    @authorized_only
    async def lock_screen(update, context):
        os.system('rundll32.exe user32.dll,LockWorkStation')
        await update.message.reply_text('Pantalla bloqueada.')

    @authorized_only
    async def shutdown(update, context):
        await update.message.reply_text('Apagando el PC...')
        os.system('shutdown /s /t 0')

    @authorized_only
    async def reboot(update, context):
        await update.message.reply_text('Reiniciando el PC...')
        os.system('shutdown /r /t 0')

  ```

  Básicamente, estas son 3 funciones asíncronas: la primera bloquea la pantalla del ordenador, la segunda apaga el sistema Windows y la tercera reinicia el sistema. Para poder usar estas funciones, se aplica el decorador **(@authorized_only)** para no repetir código y hacerlo más eficiente.<br>




### **5- Función de Información del Bot:**
Esta sección del código hace que el bot creado anteriormente te mande los comandos de cómo usar sus funciones predeterminadas.

  ```python   

    @authorized_only
    async def info(update, context):
        await update.message.reply_text("""
        📡 Command & Control 📡
                                        
        🖥️ Suspender pantalla: /lock
        📴 Apagar PC: /shutdown
        🔄 Reiniciar PC: /reboot
        ⏭️ info: /start

        Usa estos comandos para 
        controlar tu PC de manera remota.
        """)

  ```
Esta parte del código simplemente hace que, cuando tú inicias el bot **/info**, automáticamente te enviará las funciones preprogramadas que tiene para poder empezar a utilizarlo  <br>



### **6- Función de Información del Bot:**
  Esta es la última parte del código, la cual permite la interacción con el bot a través de comandos, asegurando un buen funcionamiento.

  ```python 
      def main():
          application = Application.builder().token(TOKEN).build()

          application.add_handler(CommandHandler('lock', lock_screen))
          application.add_handler(CommandHandler('shutdown', shutdown))
          application.add_handler(CommandHandler('reboot', reboot))    
          application.add_handler(CommandHandler('start', info)) 
          

          application.run_polling()

      if __name__ == '__main__':
          main()
  ```
  Se crea la función main, la cual en el primer paso establece la conexión necesaria con la API de Telegram utilizando el token. Las demás opciones son los comandos que tendrás que usar para que responda el bot. Por ejemplo, para reiniciar el PC tendrás que escribir en el **chatbot** **/reboot**, y el bot cogerá la función reboot y reiniciará el PC. 
  
  En el caso de que no hayas sabido hacer el script, te dejo el link a mi <a href="https://github.com/0x832/Command-Control">GitHub</a>, para que puedas descargarlo y utilizar el programa sin problemas. 



# Hasta aquí llega nuestro artículo explicativo sobre cómo crear Command&control.

 Espero que les haya servido la explicación y que haya sido clara. No olviden seguirme en <a href="https://github.com/0x832/">GitHub</a>, ya que iré subiendo nuevos repositorios.


---
Title: Hello World! - Tutorial CBM prg Studio
Template: blog-post
Date: 2016/12/31
Description: Este es un tutorial que estoy realizando con la idea de aprender a programar lenguaje ensamblador para C64...
---

# %meta.title%

Este es un tutorial que estoy realizando con la idea de aprender a programar lenguaje ensamblador para C64, utilizando modernas herramientas de desarrollo. Para ello utilizaremos el IDE CBMPRG Studio, que es una herramienta bastante reciente, en la que se puede desarrollar software para practicamente cualquier modelo de commodore, como las PET, VIC, C64 y C128.

No quiero poner como descargar e instalar el entorno de desarrollo, para descargarlo ir a la siguiente direccion: [www.ajordison.co.uk/download.html](http://www.ajordison.co.uk/download.html).

una guia introductoria: [www.programmermind.com/CBMPrgStudioTutorial.html](http://www.programmermind.com/CBMPrgStudioTutorial.html)

y un video (a partir de 0:55): [CBM Prg Studio Demo | Arthur Jordison](https://www.youtube.com/watch?v=2mifvS6qDQ0)

Si encuentro alguna otra referencia la agregaré aqui. 

Asi que sin mas, vamos por el primer ejemplo: Configuracion - Creación de un proyecto y el clasico **Hello World**

## Configuracion y Creacion de un proyecto

Antes que nada tenemos que configurar el emulador sobre el cual correremos nuestro codigo. Para ello vamos a `Tools - Options` y seleccionamos `Emulator control`. 

Alli configuramos:
* el target (en nuestro caso C64)
* el path del emulador (Yo uso el WinVICE, le damos el path completo, incluido el ejecutable)
* los parametros del emulador (para WinVICE: %prg)
* le ponemos `2` donde dice `Wait seconds before launching` (si ponemos valores bajos puede pasar que cuando querramos compilar y ejecutar el proyecto no llegue a cargarse en la commodore)
* y tildamos los dos checks.

una imagen vale mas que palabras: 
![Configuracion IDE](%base_url%/assets/images/blog_hello_world/configuracion_ide.png)

Y ahora creamos un nuevo proyecto: 
* File - New Proyect
* elejimos target (C64 por configuracion) - Next
* damos un nombre al project (HelloWorld, no pongo espacios, porque me los pone en el nombre... creo que me hace despelote...) - Next
* Click en `Create`

y nos queda algo asi:

![Pantalla inicial](%base_url%/assets/images/blog_hello_world/primer_pantalla_proyecto.png)

Ahora vamos a crear un archivo .asm: vamos a la ventana `Project Explorer` y clickeamos con el boton derecho sobre `Assembly files`, seleccionamos `New`, y le damos el nombre `helloworld.asm`.  
Luego vamos a File - Close Project, y cerramos el proyecto. Volvemos a abrirlo...
¿Para que hacemos todo esto? bueno, parece que esta version tiene algun que otro errorcillo, y haciendo esto salteamos un pequeño bug que tiene esta version (3.9.0). Si creamos un proyecto vamos a ver que en la configuracion del emulador va a generar el archivo `sample.prg` (y no se puede cambiar), pero al momento de compilar y ejecutar el archivo que va a buscar el emulador es helloworld.prg, y nunca lo va a encontrar. Salvamos, cerramos, volvemos a abrir, y si vamos nuevamente a la configuracion vemos que el archivo a generar es el correcto.

>**NOTA:** A pesar de ser la tercera version, el programa puede que tenga algun que otro error... no es la quinta esencia de estabilidad, pero es gratis, y a caballo regalado...  
>**NOTA 2:** El error viene por el lado que el programa parece estar, al momento de hacer `build and run` (![Boton Build&Run](%base_url%/assets/images/blog_hello_world/build_run_button.png)) generando el .prg con el nombre del proyecto abierto anteriormente... no lo encuentra (entonces aparece la pantalla original de la commodore, sin muestra de haber cargado algo), o si previamente habiamos creado un archivo .bas lo muestra, pero el .asm no lo carga... y bueno... a probar. Funcionar funciona, pero tiene sus detalles, el autor continuamente esta corrigiendo errores, asi que supongo que en una proxima version estara mejorado.

## Ahora EL CODIGO!!

~~~~~~~~
*=$C000                 ; este es el origen donde se cargara el codigo en RAM. 
                        ; Para ejecutarlo desde BASIC escribimos SYS 49152.
       LDX #$0          ; L1
cycle  LDA hworld,X     ; L2
       CMP #0           ; L3
       BEQ exit         ; L4 
       STA $0400,X      ; L5
       INX              ; L6
       JMP cycle        ; L7
exit   RTS              ; L8

hworld text 'hello world!'  ; L9
       byte 0
~~~~~~~~

El codigo es muy simple (si tenes un minimo conocimiento de ASM):

* en L1 (ver comentario) inicializa el registro X con 0. El '#' indica que es `direccionamiento inmediato` y no hace otra cosa que cargar el valor que le indicamos (#$0) al registro X (uno de los 3 registros del 6502). Existen varios modos de direccionamiento, algun otro lo vemos aca, el resto en siguientes tutoriales.
* en L2 tenemos una etiqueta, que utilizaremos para hacer un ciclo. A continuacion hay una instruccion que carga en el registro A el contenido de la direccion de memoria `hworld + X`. Esto se conoce como `direccionamiento absoluto indexado`,  llamado asi porque usamos X como indice para acceder a las direcciones de memoria a partir de 'hworld'. Esto es util para manejar tablas, strings, copiar bloques de memoria, etc.
* en L3 comparamos el contenido de A con 0. En la siguiente linea saltamos a la etiqueta `exit` si es igual (Branch on EQual). Esto es asi porque vamos a ver un poco mas adelante que donde definimos `hworld` tenemos una cadena, e inmediatamente en la siguiente linea tenemos un valor `byte` con valor 0. Este valor es el que se lee e indica que el string finalizo.
* a L5 llegamos si no se cumple la condicion de las 2 anteriores lineas. Esta linea almacena el contenido cargado previamente en A en la direccion `$0400 + X`, que corresponde con el primer caracter en la pantalla. En la siguiente linea se incrementa X, y en la siguiente se hace un salto a la etiqueta `cycle`.
* La ultima linea de codigo tiene la instruccion RTS, que indica que vuelva de la llamada realizada... ¿cual llamada? Bueno, para ejecutar este codigo nosotros tipeamos `SYS 49152` desde BASIC, y si no ponemos esta instruccion no retorna mas, o falla estrepitosamente.

~~~~~~~~
**NOTA de etiqueta hworld**: de esta manera podemos definir bloques de codigo, tablas, texto, graficos, sprites... en este ejemplo, la etiqueta queda definida como la cadena de texto 'hello world!' y a continuacion un byte con valor 0. Se pueden combinar de diferentes formas, para armar las estructuras de datos que necesitemos acceder.
~~~~~~~~

Bueno, solo queda hacer build and run, clickeando en el boton ![Boton Build&Run](%base_url%/assets/images/blog_hello_world/build_run_button.png). Si todo esta bien inicia el VICE, y carga el .prg generado. Solo queda escribir `SYS 49152` y tiene que aparecer el texto en la primera linea de la pantalla.

![Resultado final](%base_url%/assets/images/blog_hello_world/result.png)

En el [proximo tutorial](%base_url%/blog/____) veremos ____________________






























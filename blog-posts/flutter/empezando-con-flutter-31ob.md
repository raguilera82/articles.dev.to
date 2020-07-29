---
published: false
title: "Empezando con Flutter"
cover_image: 
description: 
tags: flutter
series: flutter
canonical_url:
---

# Resumen

En este artículo vamos a ver cómo hacer la instalación inicial de Flutter en un sistema operativo Ubuntu 20.04, preparar el entorno de desarrollo necesario y crear nuestro primer proyecto ejecutándolo en un dispositivo móvil.

# Entorno

* Slimbook Prox15 32 Gb RAM i7 
* SlimbookOS (Ubuntu 20.04)
* Android Studio 4.0.1
* Flutter 1.17.5
* Visual Studio Code

# Introducción

Siempre me gusta empezar las series de artículos dejando muy claro donde veo el potencial de la tecnología en cuestión. En el caso de Flutter está claro que su rango de acción es el de la construcción de aplicaciones móviles nativas en IOs y Android con el mantenimiento de un único proyecto.

Existen otras alternativas en este rango como las aplicaciones híbridas con Ionic pero realmente no son nativas ya que se ejecutan en un WebView y con la aparición de nuevos elementos nativos en los dispositivos móviles como la autenticación por huella o por reconocimiento facial, cada vez son más costosas de implementar y la experiencia de usuario deja mucho que desear.

En definitiva, mi opinión con respecto a híbridas vs nativas es que para aplicaciones de consumo rápido que estén muy acotadas en el tiempo, por ejemplo, un Madrid Fashion Week o cualquier otro evento, la solución híbridas a través de la creación de PWAs es la mejor solución; pero para aplicaciones empresariales, por ejemplo, de bancos o de cualquier otro sector, la llegada de Flutter ha hecho que sea posible y económicamente viable, que puedan ser aplicaciones nativas que mejoran la experiencia de usuario.

Flutter es una tecnología apoyada por Google que poco a poco vamos a ir viendo mucho más en el entorno empresarial por lo que es de preveer que aprenderla te asegure un futuro profesional.... así que vamos al lío!

# Instalación y configuración inicial

Si una cosa buena tiene Flutter es su documentación, en este tutorial nos vamos a centrar en su [Get Started](https://flutter.dev/docs/get-started/install)

Como vemos en la imagen cuando accedemos al enlace, Flutter está disponible para todos los sistemas operativos más comunes, incluido mi amado Linux, por tanto la guía la voy a seguir para Linux, pero el resto de plataformas son muy similares.

Obviamente en Linux solo vamos a poder trabajar con Android, por lo que tenemos que descargar el Android Studio desde la [página oficial](https://developer.android.com/studio) ,o instalarlo como snap de Ubuntu desde el Store, y descargar los últimos SDK disponibles. También tenemos que tener instalado Java compatible con la versión de Android que vayamos a instalar (suele venir ya instalado uno por defecto en Ubuntu).

En función de donde hayamos decidido instalar Android en nuestra máquina a través del asistente de Android Studio, tenemos que editar el fichero .bashrc para añadir la siguiente variable de entorno al sistema:

```
export ANDROID_SDK_ROOT=/home/tu-usuario/developement/Android/Sdk
```

Una vez ya podemos correr aplicaciones Android en nuestro equipo es el momento de instalar Flutter. Para ello descargamos la última versión para nuestro sistema operativo, es el caso de Linux, y a día de hoy es esta: https://storage.googleapis.com/flutter_infra/releases/stable/linux/flutter_linux_1.17.5-stable.tar.xz

Una vez descargado el fichero .tar.xz, lo descomprimos en la carpeta de nuestra elección, por ejemplo "development", con el comando:

```bash
$> cd ~/development
$> tar xf ~/Downloads/flutter_linux_1.17.5-stable.tar.xz
```

Y actualizamos la variable de entorno PATH dentro del fichero .bashrc para añadir el directorio bin de la instalación de Flutter.

```
export PATH=$PATH:/home/tu-usuario/development/flutter/bin
```

Para que los cambios surtan efecto debemos cerrar el terminal o bien ejecutar el siguiente comando:

```bash
$> source ~/.bashrc
```

Y probamos que se ha establecido correctamente y que podemos ejecutar Flutter con el comando:

```bash
$> flutter --version
Flutter 1.17.5 • channel stable • https://github.com/flutter/flutter.git
Framework • revision 8af6b2f038 (4 weeks ago) • 2020-06-30 12:53:55 -0700
Engine • revision ee76268252
Tools • Dart 2.8.4
```

# Preparando el entorno de desarrollo (VSCode)

Para poder desarrollar nuestras primeras aplicaciones con Flutter necesitamos un IDE, fundamentalmente manejamos dos opciones desarrollar, con Android Studio o con Visual Studio Code, a través de distintos plugins.

Nosotros vamos a hacerlo con Visual Studio, así que si ya lo tienes instalado abrelo y si no descargarlo de la [página oficial](https://code.visualstudio.com/) . Una vez abierto vamos a la sección de "Extensions" e instalamos los siguientes plugins:

* **Flutter (dart-code.flutter):** es el plugin oficial de VSCode desarrollado por el equipo de Flutter.
* **Dart (dart-code.dart-code):** es el plugin oficial de VSCode desarrollado por el equipo de Dart, que es el lenguaje en el que se apoya Flutter.

Otro a elemento a tener en cuenta es donde vamos a ejecutar las aplicaciones, dependiendo del sistema operativo podrás tener un emulador u otro, pero mi consejo es que en la medida de lo posible uses un dispositivo físico para realizar el desarrollo.

Supongamos que tienes un móvil Android que quieres utilizar para desarrollar con Flutter, en este caso tienes que habilitar el modo de "Developer" del dispositivo, esto se se suele hacer desde la pantalla Settings -- About phone -- Software information presionando repetidas veces sobre "Kernel version" hasta que te diga que ya está habilitado el modo Developer. De esta forma desde el menú principal de "Settings" ahora existirá la entrada "Developer Options" donde tenemos que habilitar la opción "USB Debugging".

Ahora conectamos el teléfono al ordenador a través de un puerto USB y tendremos que ver varios mensajes de aceptación de permisos que le permitirán al ordenador reconocer el dispositivo y poder ser utilizado para desarrollar las aplicaciones con Flutter.

# Creación y ejecución del primer proyecto

En este punto ya lo tenemos todo preparado para crear nuestro primer proyecto con Flutter. Abrimos el VSCode y desde la paleta de acciones seleccionamos la opción "Flutter: New Project", el asistente nos va a pedir que introduzcamos el nombre de proyecto y seguidamente le demos una ubicación para la creación de los archivos.

Por defecto, el asistente ya nos crea una aplicación de ejemplo con Flutter que podemos ejecutar en nuestro dispositivo con F5 a fin de comprobar que todo es correcto.

Pasados unos segundos vemos que la consola de VSCode muestra que la construcción ha sido un éxito y en el dispositivo podremos ver la aplicación de ejemplo corriendo.

Otro de los puntos fuertes de este entorno de desarrollo es que ya viene configurado para hacer Debug, de hecho si pones un punto de ruptura en la línea 62 del fichero lib/main.dart y pulsas en el botón con + de la aplicación, verás que la ejecución se detiene y puedes inspeccionar el valor de la variable "_counter".

Además cualquier cambio que hagas en el código, por ejemplo, un cambio en el literal de la línea 29 del fichero lib/main.dart hace que automáticamente se refleje en el dispositivo. Magic!









 
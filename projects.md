# Interfaz gráfica de usuario: Proyectos

Chloros te permite crear proyectos que se pueden volver a abrir en el futuro. Un proyecto es una carpeta sencilla (dentro de tu carpeta de proyectos) que contiene:

* `project.json`: configuración del proyecto, lista de archivos y preferencias de visualización
* `cameras.json`: cámaras y matrices conectadas mientras el proyecto estaba abierto, con su configuración
* `sensors.json`: sensores de luz DAQ conectados mientras el proyecto estaba abierto, además de las asociaciones cámara↔sensor
* tus capturas, grabaciones `.daq` y carpetas de resultados procesados

No existe un formato de archivo de proyecto propietario: la carpeta y sus archivos JSON constituyen el proyecto, lo que también facilita copiar, archivar y acceder a los proyectos desde [CLI](CLI.md) o [Python SDK](api-python-sdk.md).

## Nuevo proyecto

<figure><img src=".gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>Selecciona «Nuevo proyecto» en el menú principal e introduce un nombre único para tu proyecto.

Si has guardado alguna plantilla de proyecto, aparecerá un menú desplegable **Seleccionar plantilla** debajo del campo de nombre; al elegir una, el nuevo proyecto se iniciará con la configuración de esa plantilla. Las plantillas se guardan desde [Configuración del proyecto](project-settings/project-settings.md): introduce un nombre en el campo «Nombre de la plantilla del proyecto» y haz clic en el icono de guardar.

## Abrir proyecto

<figure><img src=".gitbook/assets/v120-open-project.jpg" alt=""><figcaption><p>La opción «Abrir proyecto» muestra todos los proyectos de tu carpeta de proyectos, con la opción <strong>«Abrir carpeta de proyectos»</strong> en la parte inferior</p></figcaption></figure>Selecciona «Abrir proyecto» para ver una lista de los proyectos existentes en la carpeta de proyectos. Si no hay ningún proyecto, el menú lateral secundario no se abrirá. En la imagen anterior se pueden ver algunos proyectos creados mediante la interfaz gráfica de usuario (t1, t2, t3). Los proyectos DATE\_TIME fueron creados por CLI utilizando el esquema de nomenclatura predeterminado para proyectos. Al hacer clic en el nombre de cualquier proyecto, este se abrirá.

Al hacer clic en el botón «Abrir carpeta de proyectos», se abre el explorador de archivos de tu ordenador en la ruta del proyecto. Puedes ajustar la ruta del proyecto en [Configuración del proyecto](project-settings/project-settings.md).

Si alguno de los archivos de imagen de origen del proyecto se ha movido o eliminado desde la última vez que se abrió, Chloros muestra un cuadro de diálogo en el que se enumeran exactamente los archivos que faltan, en lugar de abrir una cuadrícula en blanco.

## Duplicar proyecto

Disponible una vez abierto el proyecto. Selecciona «Duplicar proyecto» para copiar el proyecto actual con un nuevo nombre —Chloros sugiere el siguiente nombre disponible (por ejemplo, «MiProyecto (2)»)— y el duplicado se abre inmediatamente.

## Añadir archivos

Una vez abierto un proyecto, selecciona «Añadir archivos» en el menú principal para añadir archivos de imagen individuales al proyecto actual. Esta función es idéntica a la de «Añadir» del explorador de archivos, pero se puede acceder a ella directamente desde el menú principal para mayor comodidad.

## Añadir carpeta

Una vez abierto el proyecto, selecciona «Añadir carpeta» en el menú principal para añadir carpetas de imágenes al proyecto actual. Puedes seleccionar varias carpetas de una sola vez. Los archivos duplicados se ignoran.

## Iniciar/Detener el procesamiento

Una vez añadidos los archivos a un proyecto, la opción «Iniciar procesamiento» estará disponible en el menú principal. Esta acción equivale a hacer clic en el botón Reproducir/Iniciar de la barra superior. Durante el procesamiento, la opción del menú cambia a «Detener procesamiento» para permitirte interrumpir el proceso.

## Conectar a la cámara / Conectar al sensor de luz

En la parte inferior del menú principal hay dos accesos directos de hardware, disponibles tanto si hay un proyecto abierto como si no:

* **Conectar a la cámara**: abre la [pestaña Cámaras](lattice/) para conectar una cámara o una matriz LATTICE.
* **Conectar a sensor de luz**: abre la [pestaña Sensores de luz](daq/) para conectar un sensor de luz DAQ.

Si se conecta el hardware mientras hay un proyecto abierto, la conexión se guarda en dicho proyecto (véase más abajo). Sin un proyecto, las conexiones son válidas únicamente para la sesión.

{% hint style="info" %}
Las opciones de menú «Añadir archivos», «Añadir carpeta» y «Iniciar/Detener procesamiento» solo están visibles o activadas cuando hay un proyecto abierto y se han añadido archivos. Proporcionan un acceso rápido a acciones que también están disponibles a través de la barra lateral del explorador de archivos y los botones del encabezado.
{% endhint %}

## Los proyectos recuerdan tu hardware

Novedad en la versión 1.2.0: un proyecto conserva el hardware que conectes mientras esté abierto. Las cámaras y las matrices (con su configuración por cámara, nombres, colores y disposición en cuadrícula) se guardan en `cameras.json`, y los sensores de luz (con nombres, colores y asignaciones a cámaras) en `sensors.json` — de forma automática, mientras trabajas.

Cuando **vuelves a abrir** un proyecto, Chloros no interactúa inmediatamente con ningún dispositivo. Cada parte se vuelve a conectar la primera vez que accedes a la pestaña a la que pertenece:

* Al abrir la pestaña **Cámaras**, se vuelven a conectar las cámaras y matrices guardadas y se vuelven a aplicar sus ajustes guardados.
* Al abrir la pestaña **Sensores de luz**, se vuelven a conectar los sensores DAQ guardados.

De esta forma, abrir un proyecto solo para explorar o exportar imágenes nunca activa la transmisión de las cámaras. Si no se encuentra un dispositivo guardado al abrir su pestaña, un cuadro de diálogo te indica qué dispositivos no están disponibles para que puedas volver a conectarlos o eliminarlos.

## Grabaciones DAQ y archivos .daq en un proyecto

* Las grabaciones `.daq` realizadas mientras el proyecto está abierto (desde la pestaña **Sensores de luz**o durante las capturas) se**añaden automáticamente al proyecto**.
* Los archivos `.daq` importados, así como todas las grabaciones del proyecto, aparecen en la sección **DAQ Light Sensor** de [Configuración del proyecto](project-settings/project-settings.md), cada uno con su perfil de corrección de saturación.
* Durante el procesamiento, los archivos `.daq` del proyecto proporcionan la iluminación descendente para los productos de reflectancia; véase [Formatos de imagen de salida](output-image-formats.md).

## Ejecución de un proyecto guardado sin interfaz gráfica

Un proyecto guardado se puede ejecutar sin la interfaz gráfica de usuario:

* **CLI**: `chloros-cli project open / connect / capture / sensor / align / run` opera con una ruta de carpeta de proyecto; véase la [Referencia de CLI](reference/cli-reference.md).
* **SDK**: `chloros_sdk.open_project(path)` devuelve un identificador de proyecto; `connect_all()` activa todas las cámaras y sensores guardados con su configuración guardada; véase la [Referencia de SDK](reference/sdk-reference.md).

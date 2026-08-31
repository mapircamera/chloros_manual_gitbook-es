# Interfaz gráfica de usuario: Navegación

Al iniciar Chloros por primera vez, se pone en marcha su motor de procesamiento. Una vez que el motor está listo, aparece el icono del menú principal en la esquina superior izquierda <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> y se activan las pestañas «Cámaras» y «Sensores de luz» en la barra lateral izquierda (hasta ese momento aparecen atenuadas).

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

De izquierda a derecha, el encabezado superior contiene:

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> Menú principal

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Desde el menú principal puedes:

* **Nuevo proyecto**: crear un nuevo proyecto. Si tienes plantillas de proyecto guardadas, aparecerá un menú desplegable**Seleccionar plantilla** para que el nuevo proyecto comience con la configuración de una plantilla.
* **Abrir proyecto**: abrir un proyecto existente. La lista incluye un botón**Abrir carpeta del proyecto** que abre la carpeta de proyectos en tu explorador de archivos.
* **Duplicar proyecto**: copia el proyecto actualmente abierto con un nuevo nombre (se sugiere un nombre libre como «MiProyecto (2)») y abre la copia. _(visible una vez abierto el proyecto)_
* **Añadir archivos**: añade archivos de imagen individuales al proyecto actual _(visible una vez abierto el proyecto)_
* **Añadir carpeta**: añade una o más carpetas de imágenes al proyecto actual _(visible una vez abierto el proyecto)_
* **Iniciar procesamiento / Detener procesamiento**: inicia o detiene el flujo de procesamiento de imágenes _(disponible una vez añadidos los archivos)_
* **Conectar a la cámara**: pasa a la [pestaña Cámaras](lattice/) para conectar una cámara o matriz LATTICE. Funciona sin necesidad de tener un proyecto abierto.
* **Conectar a sensor de luz**: pasa a la [pestaña «Sensores de luz»](daq/) para conectar un sensor de luz DAQ. Funciona sin necesidad de tener un proyecto abierto.

{% hint style="info" %}
**Solo Windows**: La interfaz gráfica de usuario de escritorio Chloros está disponible en Windows. Los usuarios de [Linux](CLI.md) y [Python SDK](api-python-sdk.md) para el procesamiento sin interfaz gráfica.
{% endhint %}

### Botón «Reproducir/Iniciar» de<img src=".gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">



Cuando está activado, el botón de inicio del procesamiento pone en marcha el proceso de procesamiento de imágenes.

### Barra de progreso de<img src=".gitbook/assets/image (4).png" alt="" data-size="line">

<img src=".gitbook/assets/image (5).png" alt="" data-size="line">

En el modo gratuito de Chloros, que procesa todos los archivos de forma secuencial, la barra de progreso mostrará dos fases: «Detección de objetivos» y «Procesamiento».

En el modo de licencia de pago Chloros+, que procesa todos los archivos simultáneamente, la barra de progreso muestra cuatro etapas: Detección, Análisis, Calibración y Exportación. Si pasas el cursor del ratón por encima de la barra de progreso Chloros+, se desplegará el panel ampliado con las cuatro fases de la barra de progreso para que puedas seguir el proceso. Al hacer clic en la parte superior de la barra de progreso, se congelará el panel desplegable; al volver a hacer clic, se descongelará.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Menú lateral

El menú de la barra lateral izquierda contiene varios iconos con los que interactuar, en este orden de arriba abajo:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Configuración del proyecto](project-settings/project-settings.md)

La pestaña «Configuración del proyecto» te permite ajustar la configuración global del proyecto y los parámetros de procesamiento. Ajústalas antes de empezar a procesar tus archivos.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Explorador de archivos

Añade archivos o carpetas y elimina archivos del proyecto. Los archivos duplicados se ignoran. Marca la casilla de la columna «Destino» de cualquier imagen de destino, y el procesamiento solo tendrá en cuenta las imágenes marcadas como destino, lo que acelerará considerablemente el tiempo de procesamiento. Utilice el botón de alternancia «Imagen/Metadatos» para cambiar entre la visualización de la cuadrícula de miniaturas de la imagen seleccionada y una tabla detallada de metadatos.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Visor de imágenes](image-viewer-gui/opening-an-image-full-screen.md)

Al hacer clic en una imagen en el visor de imágenes principal, esta se abre a pantalla completa en la pestaña «Visor de imágenes».

#### <img src=".gitbook/assets/image (3) (1).png" alt="" data-size="line"> [Visor de mapas](image-viewer-gui/map-markers.md)

Visualiza tus imágenes en un mapa 2D interactivo basado en sus coordenadas GPS. Es compatible con los proveedores de mosaicos de Google Maps y ESRI, y selecciona automáticamente el mejor servicio para tu ubicación. Pasa el cursor por encima de los marcadores para ver vistas previas en miniatura de las imágenes.

#### <img src=".gitbook/assets/image (17).png" alt="" data-size="line"> [Cámaras](lattice/)

Conecta y controla cámaras LATTICE en directo, una por una o como conjuntos sincronizados de varias cámaras. La pestaña muestra mosaicos de vista previa en directo con superposiciones e histogramas, ajustes por cámara y por conjunto, así como ajustes de captura que permiten elegir qué cámaras y tipos de exportación genera la opción «Capturar todo». Disponible una vez que el backend esté listo; consulta la [sección de LATTICE](lattice/) para ver la guía completa.

#### <img src=".gitbook/assets/image (23).png" alt="" data-size="line"> [Sensores de luz](daq/)

Conecta sensores de luz DAQ —DAQ-U (USB), DAQ-M (Bluetooth) y DAQ-E (Ethernet)— y visualiza sus gráficos de espectro calibrados en tiempo real en W/m²/nm. Desde aquí puedes grabar archivos `.daq` en el proyecto abierto, renombrar sensores, seleccionar perfiles de corrección de tapa y actualizar el firmware del DAQ-E. Disponible una vez que el backend esté listo; consulta la [sección DAQ](daq/) para ver la guía completa.

#### Registro de depuración de <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line">

Revisa el registro en busca de mensajes de depuración cuando surjan problemas. Copia o descarga el registro y envíalo al [Servicio de asistencia de MAPIR](https://www.mapir.camera/community/contact) para obtener ayuda.

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Inicio de sesión de usuario](chloros+-login.md)

La barra lateral de inicio de sesión de usuario te permite iniciar sesión en tu cuenta de Chloros+ para desbloquear funciones avanzadas. También puede consultar la versión actual de la aplicación, así como ajustar el idioma del texto que se muestra en la interfaz gráfica de usuario de Chloros y en CLI.

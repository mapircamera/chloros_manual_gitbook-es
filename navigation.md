# Interfaz gráfica de usuario: Navegación

Al iniciar Chloros y Chloros (navegador) por primera vez, se iniciará su backend. Una vez que esté listo, aparecerá el icono del menú principal en la esquina superior izquierda <img src=".gitbook/assets/image (1) (1) (1).png" alt="" data-size="line"> .

<figure><img src=".gitbook/assets/header.JPG" alt=""><figcaption></figcaption></figure>

De izquierda a derecha, el encabezado superior contiene:

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> Menú principal

<figure><img src=".gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

Desde el menú principal puedes:

* **Nuevo proyecto** — crear un nuevo proyecto
* **Abrir proyecto** — abrir un proyecto existente
* **Abrir carpeta del proyecto** — abrir la carpeta del proyecto en el explorador de archivos
* **Añadir archivos** — añadir archivos de imagen individuales al proyecto actual _(visible una vez abierto el proyecto)_
* **Añadir carpeta**: añade una carpeta de imágenes al proyecto actual _(visible una vez abierto el proyecto)_
* **Iniciar procesamiento / Detener procesamiento**: inicia o detiene el proceso de procesamiento de imágenes _(disponible una vez añadidos los archivos)_

{% hint style="info" %}
**Solo Windows**: La interfaz gráfica de usuario de escritorio Chloros está disponible en Windows. Los usuarios de Linux deben consultar [CLI](CLI.md) y [Python SDK](api-python-sdk.md) para el procesamiento sin interfaz gráfica.
{% endhint %}

### <img src=".gitbook/assets/image (2) (1).png" alt="" data-size="line"> Botón Reproducir/Iniciar

Cuando está habilitado, el botón de inicio del procesamiento inicia el proceso de procesamiento de imágenes.

### <img src=".gitbook/assets/image (4).png" alt="" data-size="line"> Barra de progreso <img src=".gitbook/assets/image (5).png" alt="" data-size="line">En el modo gratuito Chloros, que procesa todos los archivos de forma secuencial, la barra de progreso mostrará 2 etapas: Detección de objetivos y Procesamiento.

En el modo de licencia de pago Chloros+, que procesa todos los archivos simultáneamente, la barra de progreso muestra 4 etapas: Detección, Análisis, Calibración y Exportación. Si pasas el cursor del ratón por encima de la barra de progreso de Chloros+, se desplegará el panel ampliado de la barra de progreso de 4 fases para que puedas seguir el proceso. Al hacer clic en la barra de progreso superior se congelará el panel desplegable; al volver a hacer clic, se descongelará.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Menú lateral

El menú de la barra lateral izquierda contiene varios iconos con los que interactuar:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Configuración del proyecto](project-settings/project-settings.md)

La pestaña Configuración del proyecto te permite ajustar la configuración global y de procesamiento del proyecto. Ajusta estos parámetros antes de comenzar a procesar tus archivos.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Explorador de archivos

Añade archivos o carpetas y elimina archivos del proyecto. Los archivos duplicados se ignoran. Marca la casilla de la columna «Destino» de cualquier imagen de destino y el procesamiento solo buscará imágenes marcadas como destinos, lo que acelerará considerablemente el tiempo de procesamiento. Utiliza el botón de alternancia «Imagen/Metadatos» para cambiar entre la vista de la cuadrícula de miniaturas de la imagen seleccionada y una tabla detallada de metadatos.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Visor de imágenes](image-viewer-gui/opening-an-image-full-screen.md)

Al hacer clic en una imagen en el visor de imágenes principal, esta se abre a pantalla completa en la pestaña «Visor de imágenes».

#### <img src=".gitbook/assets/image (7).png" alt="" data-size="line"> [Mapa](image-viewer-gui/map-markers.md)

Visualice sus imágenes en un mapa 2D interactivo basado en sus coordenadas GPS. Es compatible con los proveedores de mosaicos de Google Maps y ESRI, seleccionando automáticamente el mejor servicio para su ubicación. Pase el cursor por encima de los marcadores para ver vistas previas en miniatura de las imágenes.

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> Registro de depuración

Revisa el registro en busca de mensajes de depuración cuando surjan problemas. Copia o descarga el registro y envíalo al [Soporte de MAPIR](https://www.mapir.camera/community/contact) para obtener ayuda.

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Inicio de sesión de usuario](chloros+-login.md)

La barra lateral de inicio de sesión de usuario le permite iniciar sesión en su cuenta Chloros+ para desbloquear funciones avanzadas. También puede ver la versión actual de la aplicación, así como ajustar el idioma del texto que se muestra en la interfaz gráfica de usuario (GUI) de Chloros y CLI.

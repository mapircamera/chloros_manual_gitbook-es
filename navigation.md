# GUI : Navegación

Cuando lance Chloros y Chloros (Navegador) por primera vez, iniciará su backend. Una vez que esté listo, se revelará el ícono del menú principal en la esquina superior izquierda <img src=".gitbook/assets/image (1).png" alt="" data-size="line"> .

<figure><img src=".gitbook/assets/header.JPG" alt=""><figcaption></figcaption></figure>

De izquierda a derecha, el encabezado superior contiene:

### <img src=".gitbook/assets/image (1) (1).png" alt="" data-size="line"> Menú Principal

Desde el menú principal puede iniciar un nuevo proyecto, abrir un proyecto existente o abrir la carpeta del proyecto.

### <img src=".gitbook/assets/image (2).png" alt="" data-size="line"> Botón Reproducir/Iniciar

Cuando está habilitado, el botón de iniciar procesamiento inicia el pipeline de procesamiento de imágenes.

### <img src=".gitbook/assets/image (4).png" alt="" data-size="line"> Barra de Progreso <img src=".gitbook/assets/image (5).png" alt="" data-size="line">

En el modo Chloros gratuito, que procesa todos los archivos secuencialmente, la barra de progreso mostrará 2 etapas: Detección de Objetivo y Procesamiento.

En el modo con licencia Chloros+ pagada, que procesa todos los archivos simultáneamente, la barra de progreso muestra 4 etapas: Detectando, Analizando, Calibrando, Exportando. Si pasa el cursor del mouse sobre la barra de progreso de Chloros+, se desplegará el panel extendido de 4 barras de progreso para que pueda seguir el proceso. Hacer clic en la barra de progreso superior congelará el panel desplegable, hacer clic nuevamente lo descongelará.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Menú Lateral

La barra lateral izquierda contiene varios íconos con los que interactuar:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Configuración del Proyecto](project-settings/page-2.md)

La pestaña de Configuración del Proyecto le permite ajustar la configuración global y de procesamiento del proyecto. Ajuste estos antes de comenzar a procesar sus archivos.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Explorador de Archivos

Agregue archivos/carpetas y elimine archivos del proyecto. Los archivos duplicados se ignoran. Marque la casilla de la columna objetivo para cualquier imagen objetivo, y el procesamiento solo buscará targets en las imágenes marcadas, acelerando enormemente su tiempo de procesamiento.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Visor de Imágenes](image-viewer-gui/page-3.md)

Cuando se hace clic en una imagen en el visor de imágenes principal, se abre en pantalla completa en la pestaña del Visor de Imágenes.

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> Registro de Depuración

Revise el registro para impresiones de depuración cuando ocurran problemas. Copie/descargue el registro y envíelo al [Soporte de MAPIR](https://www.mapir.camera/community/contact) para obtener asistencia.

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Inicio de Sesión de Usuario](chloros+-login.md)

La barra lateral de inicio de sesión de usuario le permite iniciar sesión en su cuenta Chloros+ para desbloquear funciones avanzadas. También puede ver la versión actual de la aplicación, así como ajustar el idioma del texto mostrado en la GUI y CLI de Chloros.

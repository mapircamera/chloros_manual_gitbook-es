---
metaLinks: {}
---

# Introducción

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>

Chloros es una aplicación de software de [MAPIR](https://www.mapir.camera) que permite procesar imágenes multiespectrales, controlar en tiempo real el hardware de MAPIR y registrar datos de sensores. Chloros 1.3.0 es compatible con toda la gama de productos MAPIR:

* **Cámaras Survey3**: procesan capturas RAW+JPG para generar mapas calibrados de reflectancia e índices de vegetación. Consulta [Cámaras compatibles](supported-cameras.md).
* **Cámaras LATTICE**: conecta los módulos de cámara multiespectral GigE en tiempo real, de forma individual o como conjuntos sincronizados de varias cámaras; permite previsualizar, capturar y procesar los datos para obtener productos calibrados de radiancia y reflectancia. Consulta la [sección de LATTICE](lattice/README.md).
* **Sensores de luz DAQ** — Sensores espectrales DAQ-U (USB), DAQ-M (Bluetooth) y DAQ-E y DAQ-E-S (Ethernet): espectros calibrados en tiempo real, grabaciones `.daq` e iluminación descendente para el procesamiento de la reflectancia. Consulte la [sección DAQ](daq/README.md).

{% hint style="success" %}
**Novedades de Chloros 1.3.0**: los asistentes de IA pueden controlar Chloros a través de `chloros-mcp`, y la CLI, el SDK y `chloros-mcp` ofrecen las mismas funciones. Se han mejorado los productos LATTICE Index; las matrices de sensores completas son más rápidas y funcionan en cualquier red, y el backend sigue ejecutándose entre comandos. Consulta la lista de novedades a continuación y [descarga](download.md) los instaladores.
{% endhint %}

{% hint style="info" %}
**¿Utilizas Chloros con un asistente de IA?** Este manual está pensado para ello. Indícale a tu asistente que acceda a:

* `https://mapir.gitbook.io/chloros/llms.txt`: índice legible por máquina de todas las páginas.
* Cualquier página en formato Markdown sin formato: añade `.md` a su URL (p. ej., `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* La [Referencia de CLI](reference/cli-reference.md) y la [Referencia de SDK](reference/sdk-reference.md): páginas de referencia completas con valores exactos, escritas para su uso con modelos de lenguaje grande (LLM).

Ejemplo de prompt: *«Lee https://mapir.gitbook.io/chloros/reference/cli-reference.md, y, a continuación, escribe un script que inicie sesión y procese la carpeta ~/flights/flight_001 para convertirla en archivos GeoTIFF de reflectancia y NDVI».*

O bien, deja que ejecute Chloros directamente: `chloros-mcp`, un servidor MCP instalado con el SDK, funciona con Claude Desktop, Claude Code, Cursor y otros clientes MCP.

Guía completa: [Cómo utilizar Chloros con asistentes de IA](ai-assistants.md).
{% endhint %}

***

## Novedades de Chloros 1.3.0

* **Los asistentes de IA pueden controlar Chloros.** `chloros-mcp`, un servidor MCP instalado con el SDK de Python, permite que Claude Desktop, Claude Code, Cursor y otros clientes MCP procesen imágenes y controlen cámaras LATTICE, matrices y sensores de luz DAQ. Consulta [Cómo utilizar Chloros con asistentes de IA](ai-assistants.md).
* **La CLI, el SDK y `chloros-mcp` ofrecen las mismas funciones.** Por motivos de seguridad, solo el inicio de sesión, el aprovisionamiento de DAQ-E y los servidores DAQ directos permanecen en la CLI. Algunos scripts de la versión 1.2.0 requieren modificaciones: véase **Cambios en la versión 1.3.0** al principio de la [Referencia de la CLI](reference/cli-reference.md) y la [Referencia del SDK](reference/sdk-reference.md).
* **Productos de índices LATTICE mejorados.** Se han mejorado el NDVI y los demás productos de índices exportados desde las capturas de LATTICE, en todos los modos de procesamiento.
* **Opciones del instalador de Windows.**En**Configuración del sistema**, el instalador ofrece las opciones**Habilitar rutas largas de Windows**y**Excluir el backend de Chloros del análisis de Microsoft Defender**, ambas marcadas por defecto. La primera permite que las carpetas de proyectos tengan una ruta de más de 260 caracteres; la segunda evita que el análisis en tiempo real ralentice la captura de la cámara (aproximadamente un 30 % de la velocidad de captura a velocidades elevadas). El instalador ahora también añade la carpeta CLI a `PATH` solo una vez y elimina las entradas duplicadas que dejaban los instaladores anteriores. Véase [Descargar](download.md).
* **Las carpetas de proyecto con rutas demasiado profundas se rechazan de entrada.** Con las rutas largas de Windows desactivadas (la configuración predeterminada de Windows), una carpeta de proyecto demasiado profunda para el límite de ruta de 260 caracteres se rechaza antes de que se ejecute nada, indicando la carpeta de proyecto más larga que cabe y cómo activar las rutas largas. Antes, una ejecución de este tipo no generaba ningún producto. Con las rutas largas activadas, los productos en carpetas muy anchas conservan ahora sus metadatos (etiquetas GPS y XMP).
* **Máscaras de recorte.** Cuando la separación de bandas de un producto de radiancia o reflectancia de LATTICE produce un valor de banda negativo, dicho valor se recorta a cero, y una máscara por píxel en una carpeta `clamp_masks/` situada junto a las carpetas de productos registra qué píxeles y bandas se han visto afectados. El producto nombra su máscara en la etiqueta `Chloros:UnmixClampMask`. Las vistas en directo ahora se recortan exactamente igual que en la exportación. Véase [Formatos de imagen de salida](output-image-formats.md).
* **Los productos se describen a sí mismos.** Los productos de índice llevan sus propias etiquetas de procesamiento (`Chloros:ProcessingLevel=index`, la expresión del índice y si se han calculado a partir de la radiancia o la reflectancia) en lugar de las del archivo RAW original. Los productos de Survey3 llevan etiquetas de procesamiento de Chloros en todos los modos de procesamiento. Los archivos sin procesar y sin debayering de LATTICE conservan la orientación nativa del sensor y registran dicha orientación en XMP; el visor de imágenes los gira únicamente para su visualización.
* **Los paquetes para Linux se distribuyen sin descomprimir.** El backend y la CLI se instalan como carpetas dentro de `/usr/lib/chloros/`, por lo que ya no se descomprimen automáticamente cada vez que se inician. En x86_64, el backend ya no tarda unos 100 segundos en iniciarse, y el `.deb` ocupa aproximadamente la mitad de su tamaño anterior. La herramienta ExifTool incluida ahora funciona en una instalación limpia de Ubuntu. El `.deb` ahora también instala el Python SDK para el sistema `python3` (instálalo con `sudo apt install ./chloros_1.3.0_<arch>.deb`), aumenta los búferes de recepción de sockets del kernel que necesitan las transmisiones de la cámara y añade un servicio de backend que se ejecuta como tu usuario. El paquete para Jetson, `chloros_1.3.0_arm64_jp6.deb`, está compilado para JetPack 6 y procesa en la GPU del Jetson con la versión de PyTorch para CUDA de JetPack. En Linux, cada imagen exportada llega ahora al disco una sola vez (se escribe y se etiqueta en memoria, y luego se copia a su ubicación): en un Jetson, donde el disco marca el ritmo, una ejecución de LATTICE con 288 imágenes tarda 14 minutos en lugar de 24. Véase [Instalación en Linux](linux/linux-installation.md).
* **Las cámaras LATTICE OCN ofrecen una lectura más precisa a la luz del sol.** La radiancia, la reflectancia y los índices reciben una corrección rápida tras la separación de bandas, por lo que un objetivo gris se lee con un NDVI cercano al neutro bajo el sol. Las cámaras RGN no presentan cambios.
* **Informes de errores desde la aplicación.** Un icono de error situado en la parte superior de la pestaña «Registro de depuración» envía a MAPIR una descripción, hasta 10 imágenes, los registros y los detalles del sistema. Véase [GUI: Navegación](navigation.md).
* **Matrices LATTICE.**Las matrices conservan el sensor completo por defecto, y el transporte comprimido de 12 bits (`--packed`, por defecto `auto`) aumenta la velocidad de fotogramas del sensor completo en aproximadamente un tercio; una matriz funciona en cualquier red a la velocidad de fotogramas que esta permita (un enlace lento reduce la velocidad de fotogramas en lugar de rechazar la matriz o reducir el tamaño de sus fotogramas); una matriz cuyas cámaras se reinician se recupera por sí sola, con el PTP de nuevo activo; la exposición automática ya no reduce esa velocidad con poca luz (llena el periodo y, a continuación, aumenta la ganancia); La captura continua sigue el ritmo de las matrices rápidas; un control de**Highlights** para cámaras RGB; y el SDK incorpora `ArraySession`, `grab()`, `frame_stream()` y `continuous()`. Consulta la [sección LATTICE](lattice/README.md).
* **Exposición automática más estable.** La exposición se mantiene constante con luz constante y se estabiliza a los pocos segundos de un cambio; las pequeñas zonas de alta luminosidad ya no oscurecen la escena; una exposición manual deja la ganancia en «Auto»; y una cámara que deja de responder bajo un sol intenso se mantiene en el nivel adecuado para esa luz, medido por el sensor de luz o, si no hay ninguno conectado, por los propios fotogramas de la cámara. Consulta [Ajustes de la cámara](lattice/camera-settings.md).
* **Sensores de luz.** **Compatibilidad con DAQ-E-S:** su difusor fijo no requiere corrección de tapa, y cada lectura incluye la inclinación, el balanceo y el cabeceo de la unidad. Una grabación indica si se ha seleccionado o se ha supuesto el valor máximo; el procesamiento avisa en caso de que se haya supuesto: los modelos DAQ-U, DAQ-M y DAQ-E se suministran sin valor máximo, por lo que hay que especificar el valor instalado. Los registros en tiempo real `.daq` se conservan tras un corte de corriente (conserva un archivo `.daq-journal` junto con su `.daq`), y un DAQ-E que deja de transmitir se vuelve a conectar con el intervalo marcado en su grabación. Un proyecto que solo contenga grabaciones de sensores de luz las muestra y las procesa desde la aplicación y la CLI, y una grabación realizada con el `chloros_scripts` aparece en la lista como el DAQ que la realizó, con un límite que puedes configurar. Cada captura guarda la lectura del DAQ que se ha utilizado. Consulta la [sección DAQ](daq/README.md).
* **Procesamiento.**Las tramas LATTICE comienzan a procesarse durante la detección de objetivos. Las referencias de objetivos y DAQ se determinan por banda. Al mover un proyecto, este vuelve a vincular sus archivos. Las exportaciones de una captura de 8 bits son de 8 bits, con un nuevo formato**TIFF (8 bits)**. Una ejecución de Survey3 sin objetivo de calibración se archiva como su producto alternativo (`Vignette_Corrected_Images` o `Sensor_Response_Images`), no como reflectancia. Las etiquetas DLS por banda de una imagen de reflectancia son siempre las de su propio fotograma. La opción «Texture Aware» no se admite para las capturas LATTICE, que siempre se procesan con el debayer estándar.
* **CLI y SDK.** Las rutas de salida relativas ahora se guardan en la carpeta desde la que se ejecuta el comando o el script (antes, un backend instalado las resolvía en su propia carpeta de instalación y daba error). El backend sigue ejecutándose entre comandos y scripts y se detiene automáticamente tras 30 minutos de inactividad; `chloros-cli backend status` y `backend stop` se encargan de gestionarlo. Los comandos de captura enumeran todos los archivos que no han podido escribir y salen con el código 1; además, las capturas repetidas se detienen cuando el disco está lleno. Una ejecución correcta de `process` genera el informe `Image products written: N (plus M clamp masks)`. En Windows, el SDK de `ChlorosLocal` / `process_folder` del SDK localizan el backend instalado (y utilizan uno que ya esté en ejecución), y `read_image_audit_tags` funciona desde un `pip install` sin opciones.

***

## Novedades de Chloros 1.2.0

* **Control de cámaras en directo: nueva pestaña «Cámaras».** Conecta cámaras LATTICE de una en una o como conjuntos sincronizados de varias cámaras (sincronización de tiempo PTP, captura activada por hardware), con superposiciones de vista previa en directo, histogramas por banda, exposición automática inteligente, un calculador de índice en directo y actualizaciones del firmware de la cámara desde la propia aplicación.
* **Sensores de luz — nueva pestaña «Sensores de luz».** Conecta sensores DAQ-U (USB), DAQ-M (Bluetooth) y DAQ-E (Ethernet); visualiza espectros calibrados en tiempo real (W/m²/nm), graba archivos `.daq` en tu proyecto, selecciona perfiles de corrección de capacidad y actualiza el firmware del DAQ-E a través de la red.
* **Modos de captura y grabadores.** Captura única / continua / por intervalos, además de un modo de captura más rápida solo en formato sin procesar; selección por proyecto de las cámaras y los tipos de exportación que genera la opción «Capturar todo»; grabadores matriciales para vídeo indexado de calidad de monitorización y ráfagas sin procesar de calidad de análisis con compilaciones de vídeo sin conexión.
* **Flujo de procesamiento LATTICE.** Importa carpetas de captura de LATTICE y divide cada fotograma sin procesar en productos de debayering, vista previa, radiancia en float32 (W/m²/sr/nm) y reflectancia, con opciones de activación o desactivación por producto. La reflectancia puede proceder de un objetivo de calibración dentro del fotograma o de la radiación descendente de DAQ; se aplica la alineación de la matriz a las exportaciones; la calibración de fábrica que falte se descarga automáticamente mediante el número de serie de la cámara.
* **Los proyectos recuerdan la configuración del hardware.** Las cámaras y los sensores de luz conectados se guardan con el proyecto (`cameras.json` / `sensors.json`) y se vuelven a conectar con sus ajustes guardados al volver a abrir el proyecto. Véase [Interfaz gráfica de usuario: Proyectos](projects.md).
* **Mejoras en el visor de imágenes.** Lectura de píxeles/índices del cursor con escalado correcto de la reflectancia por archivo, histogramas de capas, un control deslizante de agrupación de GSD, modos de cuadrícula «Por disparador» y «Por cámara», vistas de productos LATTICE y exportaciones a disco del índice y la tabla de conversión (LUT) en el entorno de pruebas.
* **CLI y SDK, ampliamente ampliados.** Nuevas familias de comandos `lattice`, `daq pool-*`, `project` y `time-sync`; nuevas opciones `process` (`--input-level`, conmutadores por producto; `--reflectance-source`, indicadores de alineación de matrices); manejadores de conexión inteligente del SDK (`connect_camera` / `connect_array` / `connect_daq_sensor`) que inician automáticamente el backend; Automatización de `open_project()`; el paquete «wheel» del SDK se incluye con los instaladores y se publica en PyPI como `chloros-sdk`.
* **Semántica de fallo transparente.** Una ejecución de `chloros-cli process` que solicitara productos pero no generara ninguno ahora falla de forma evidente y termina con un código de salida distinto de cero; las ejecuciones correctas informan del número de productos de imagen generados.
* **Nuevo diseño de salida.** Los productos se guardan en carpetas `<project>/<camera>/<format>/<Product>_Images/` y conservan el nombre del archivo de origen; es la carpeta, y no un sufijo del nombre de archivo, la que identifica el producto. Véase [Formatos de imagen de salida](output-image-formats.md).
* **Más entradas, planes e idiomas.** Compatibilidad con entradas `.dng`; los 38 idiomas de la interfaz están completamente disponibles; límites de hardware por plan con uso gratuito (sin necesidad de iniciar sesión) de hasta 4 cámaras y 2 sensores de luz.
* **Fiabilidad.** La opción «Detener procesamiento» finaliza correctamente con un resumen preciso de la ejecución; los proyectos con varias cámaras exportan todas las cámaras; y las actualizaciones del instalador ya no te desconectan de la sesión.

***

Chloros está disponible en tres entornos de aplicación:

## Chloros: aplicación de interfaz gráfica de usuario (GUI) de escritorio

Ventana independiente con todas las funciones, incluidas las pestañas «Cámaras en directo» y «Sensores de luz». _Solo para Windows._

## [Chloros CLI: interfaz de línea de comandos](CLI.md)

Procesamiento por lotes desde la línea de comandos, además de los comandos en tiempo real `lattice`, `daq pool-*`, `project` y `time-sync`. Perfecto para la automatización, la creación de scripts y el funcionamiento sin interfaz gráfica. Disponible en **Windows, Linux amd64 y Linux arm64 (NVIDIA Jetson)**. _Para acceder a la CLI es necesario disponer de un plan de pago Chloros+._

## [Chloros API: Python SDK](api-python-sdk.md)

Interfaz programática en Python para la automatización y flujos de trabajo personalizados: procesamiento de canalización completa, sesiones en directo con cámaras o matrices, sesiones con sensores DAQ y automatización de proyectos guardados. Se instala con el paquete de escritorio/CLI y también se publica como `pip install chloros-sdk`. _Para acceder a la API es necesario disponer de un plan de pago de Chloros+._

***

## Plataformas compatibles

| Plataforma | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11 (x64)** | Sí | Sí | Sí |
| **Linux amd64 (x86_64)** | No | Sí | Sí |
| **Linux arm64 (NVIDIA Jetson)** | No | Sí | Sí |

Para obtener instrucciones de instalación en Linux, consulta la sección [Linux y computación periférica](linux/linux-overview.md).

***

## Empieza en tres pasos

1. **Instalar**: descarga y ejecuta el instalador correspondiente a tu plataforma. Consulta [Descargar](download.md).
2. **Inicia sesión (opcional para la interfaz gráfica)**: la interfaz gráfica procesa imágenes de forma gratuita sin necesidad de una cuenta. Un [inicio de sesión en Chloros+](chloros+-login.md) desbloquea el procesamiento paralelo, la aceleración por GPU, límites de dispositivos más altos y acceso a la CLI y al SDK.
3. **Crea tu primer proyecto**: abre Chloros, crea un [nuevo proyecto](projects.md), [añade tus imágenes](processing-images-gui/adding-files-to-a-project.md) y [inicia el procesamiento](processing-images-gui/starting-the-processing.md). Si prefieres controlar hardware en tiempo real, abre la pestaña «Cámaras» o «Sensores de luz»; consulta [GUI: Navegación](navigation.md).

***

## Chloros+

Aunque Chloros es de uso gratuito para la mayoría de las tareas, es posible que quieras más. Ahí es donde una licencia de pago para Chloros+ puede resultarte útil. Con una licencia de Chloros+ puedes desbloquear nuevas funciones como:

* **Procesamiento multihilo**: acelera considerablemente el procesamiento de imágenes en proyectos de mayor envergadura al procesar imágenes simultáneamente a través del flujo de trabajo.
* **Aceleración por GPU (CUDA)**: aprovecha las opciones actuales de mayor memoria de GPU para acelerar aún más el flujo de trabajo de procesamiento de imágenes. Recomendamos 4 GB o más de VRAM para obtener los mejores resultados.
* **Chloros+**[**CLI**](CLI.md) **Acceso**: ejecuta Chloros+ desde la línea de comandos para automatizar e integrar el proceso en tu propio software. Disponible en cualquier plan de pago; se aplica del lado del servidor.
* **Chloros+**[**API**](api-python-sdk.md) **Acceso:** ejecuta Chloros+ desde Python para un control programático, lo que permite una integración perfecta con tus procesos de investigación, flujos de trabajo de análisis de datos y aplicaciones personalizadas. Disponible en cualquier plan de pago; se aplica del lado del servidor.
* **Límites de hardware más altos**: conecta más cámaras y sensores de luz a la vez. Sin necesidad de iniciar sesión, la interfaz gráfica de usuario (GUI) conecta hasta 4 cámaras y 2 sensores de luz DAQ; los planes de pago aumentan ambos límites:

| Plan | Cámaras | Sensores de luz DAQ |
| --- | --- | --- |
| Iron (gratuito, sin iniciar sesión) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

* **Uso en varios dispositivos**: cada licencia de Chloros+ permite registrar 2 o más dispositivos. Utiliza tu cuenta de MAPIR Cloud para gestionar los dispositivos registrados. Amplía la compatibilidad con más dispositivos actualizando tu licencia de Chloros+.
* **Método avanzado de despixelarización sensible a la textura:** un método de despixelarización de alta calidad sensible a los bordes, combinado con un modelo de eliminación de ruido basado en IA/ML que elimina casi todo el ruido de la despixelarización.
* **Fórmulas personalizadas de índices multiespectrales:** introduce índices multiespectrales personalizados en las calculadoras de ráster de Chloros, tanto para el procesamiento como para el entorno de pruebas de visualización de imágenes.
* **Linux y computación en el borde:** ejecuta Chloros en plataformas Linux x86_64 y ARM64, incluida NVIDIA Jetson, para el procesamiento sobre el terreno y en el borde. Consulta la [Descripción general de Linux](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Precios y registro en Chloros+</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: cli.JPG shows the 1.1.0 CLI banner. Re-shoot a terminal running `chloros-cli --version` + `chloros-cli status` on the 1.3.0 build so the banner prints "Chloros CLI 1.3.0". -->

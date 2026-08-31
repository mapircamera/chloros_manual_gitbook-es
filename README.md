---
metaLinks: {}
---

# Introducción

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>

Chloros

es una aplicación de software de [MAPIR

](https://www.mapir.camera) para procesar imágenes multiespectrales, controlar en tiempo real el hardwareMAPIR

y registrar datos de sensores.Chloros

1.2.0 es compatible con toda la familia de productosMAPIR

:

* **CámarasSurvey3**: procesan capturas RAW+JPG para obtener mapas calibrados de reflectancia e índices de vegetación. Véase [Cámaras compatibles](supported-cameras.md).
* **Cámaras LATTICE**: conecta los módulos de cámara multiespectral GigE en tiempo real, de forma individual o como conjuntos sincronizados de varias cámaras; previsualiza, captura y procesa los datos para obtener productos calibrados de radiancia y reflectancia. Consulta la [sección de LATTICE](lattice/README.md).
* **Sensores de luz DAQ** — Sensores espectrales DAQ-U (USB), DAQ-M (Bluetooth) y DAQ-E (Ethernet): espectros calibrados en tiempo real, grabaciones `.daq` e iluminación descendente para el procesamiento de la reflectancia. Consulte la [sección DAQ](daq/README.md).

{% hint style="success" %}
**Novedades deChloros

1.2.0**: control en tiempo real de cámaras y matrices LATTICE, integración de sensores de luz DAQ, modos de captura y grabadoras, un flujo de trabajo completo de procesamiento radiométrico de LATTICE, automatización de proyectos desdeCLI

/SDK

y mucho más. Consulta la lista de novedades a continuación y [descarga](download.md) el registro de cambios.
{% endhint %}

{% hint style="info" %}
**¿UtilizasChloros

con un asistente de IA?** Este manual está diseñado para ello. Indica a tu asistente que acceda a:

* `https://mapir.gitbook.io/chloros/llms.txt`: índice legible por máquina de todas las páginas.
* Cualquier página en formato Markdown sin formato: añade `.md` a suURL

(p. ej., `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* La [Referencia deCLI

](reference/cli-reference.md) y la [Referencia deSDK

](reference/sdk-reference.md): páginas de referencia completas con valores exactos, escritas para su uso por parte de los modelos de lenguaje grande (LLM).

Ejemplo de prompt: *«Lee https://mapir.gitbook.io/chloros/reference/cli-reference.md, y, a continuación, escribe un script que inicie sesión y procese la carpeta ~/flights/flight_001 para convertirla en archivos GeoTIFF de reflectancia +NDVI

».*

Guía completa: [Uso deChloros

con asistentes de IA](ai-assistants.md).
{% endhint %}

***

## Novedades deChloros

1.2.0

* **Control de la cámara en directo: nueva pestaña «Cámaras».** Conecta cámaras LATTICE de una en una o como conjuntos sincronizados de varias cámaras (sincronización de tiempo PTP, captura activada por hardware), con superposiciones de vista previa en directo, histogramas por banda, exposición automática inteligente, un calculador de índice en directo y actualizaciones del firmware de la cámara desde la propia aplicación.
* **Sensores de luz — nueva pestaña «Sensores de luz».** Conecta sensores DAQ-U (USB), DAQ-M (Bluetooth) y DAQ-E (Ethernet); visualiza espectros calibrados en tiempo real (W/m²/nm), graba archivos `.daq` en tu proyecto, selecciona perfiles de corrección de cap y actualiza el firmware del DAQ-E a través de la red.
* **Modos de captura y grabadores.** Captura única / continua / por intervalos, además de un modo de captura más rápida solo en formato sin procesar; selección por proyecto de las cámaras y los tipos de exportación que genera la opción «Capturar todo»; grabadores matriciales para vídeo indexado de calidad de monitorización y ráfagas sin procesar de calidad de análisis con compilaciones de vídeo sin conexión.
* **Flujo de procesamiento LATTICE.** Importa carpetas de captura de LATTICE y procesa cada fotograma sin procesar para obtener productos de desbayering, vista previa, radiancia en float32 (W/m²/sr/nm) y reflectancia, con opciones de activación o desactivación por producto. La reflectancia puede proceder de un objetivo de calibración dentro del fotograma o de la luz descendente del DAQ; se aplica la alineación de la matriz a las exportaciones; la calibración de fábrica que falte se descarga automáticamente mediante el número de serie de la cámara.
* **Los proyectos recuerdan la configuración del hardware.** Las cámaras y los sensores de luz conectados se guardan con el proyecto (`cameras.json` / `sensors.json`) y se vuelven a conectar con sus ajustes guardados al volver a abrir el proyecto. Véase [Interfaz gráfica de usuario: Proyectos](projects.md).
* **Mejoras en el visor de imágenes.** Lectura de píxeles/índice del cursor con escalado correcto de la reflectancia por archivo, histogramas de capas, un control deslizante de agrupación de GSD, modos de cuadrícula «Por disparador» y «Por cámara», vistas de productos LATTICE y exportaciones a disco del índice y la tabla de conversión (LUT) en modo de prueba.
* **Ampliación significativa deCLI

ySDK

.** Nuevas familias de comandos `lattice`, `daq pool-*`, `project` y `time-sync`; nuevas opciones `process` (`--input-level`, opciones de activación/desactivación por producto; `--reflectance-source`, indicadores de alineación de matrices);SDK

manejadores «smart-connect» (`connect_camera` / `connect_array` / `connect_daq_sensor`) que inician automáticamente el backend; automatización de `open_project()`; el paquete «SDK

» se incluye con los instaladores y se publica en PyPI como `chloros-sdk`.
* **Semántica de fallo transparente.** Una ejecución de `chloros-cli process` que solicitara productos pero no escribiera ninguno ahora falla de forma evidente y sale con un código de salida distinto de cero; las ejecuciones exitosas informan de cuántos productos de imagen se han escrito.
* **Nuevo diseño de salida.** Los productos se guardan en carpetas `<project>/<camera>/<format>/<Product>_Images/` y conservan el nombre del archivo de origen; es la carpeta, y no un sufijo del nombre de archivo, la que identifica el producto. Véase [Formatos de imagen de salida](output-image-formats.md).
* **Más entradas, planes e idiomas.** Compatibilidad con entradas `.dng`; los 38 idiomas de la interfaz están completamente disponibles; límites de hardware por plan con uso gratuito (sin necesidad de iniciar sesión) de hasta 4 cámaras y 2 sensores de luz.
* **Fiabilidad.** La opción «Detener procesamiento» finaliza correctamente con un resumen detallado de la ejecución; los proyectos con varias cámaras exportan todas las cámaras; y las actualizaciones del instalador ya no te desconectan.***

Chloros

está disponible en tres entornos de aplicación:

##Chloros

: Aplicación de interfaz gráfica de usuario (GUI) de escritorio

Ventana independiente con todas las funciones, incluidas las pestañas «Cámaras en directo» y «Sensores de luz». _Solo para Windows._

## [Chloros

CLI

: Interfaz de línea de comandos](CLI.md)

Procesamiento por lotes desde la línea de comandos, además de los comandos en tiempo real `lattice`, `daq pool-*`, `project` y `time-sync`. Perfecto para la automatización, la creación de scripts y el funcionamiento sin interfaz gráfica. Disponible en **Windows

,Linux

amd64 yLinux

arm64 (NVIDIA Jetson)**. _Para acceder a la CLI es necesario disponer de un plan de pagoChloros

+._

## [Chloros

API

:Python

SDK

](api-python-sdk.md)

Interfaz programáticaPython

para la automatización y los flujos de trabajo personalizados: procesamiento de canalización completa, sesiones en directo con cámaras o matrices, sesiones con sensores DAQ y automatización de proyectos guardados. Se instala con el paquete de escritorio/CLI

y también se publica como `pip install chloros-sdk`. _Para acceder a la API es necesario disponer de un plan de pagoChloros

+._

***

## Plataformas compatibles

| Plataforma | Interfaz gráfica |CLI

|Python

SDK

|
| --- | --- | --- | --- |
| **Windows

10/11 (x64)** | Sí | Sí | Sí |
| **Linux

amd64 (x86_64)** | No | Sí | Sí |
| **Linux

arm64 (NVIDIA Jetson)** | No | Sí | Sí |

Para obtener instrucciones de instalación enLinux

, consulta la sección [Linux

y computación en el borde](linux/linux-overview.md).

***

## Empieza en tres pasos

1. **Instalar**: descarga y ejecuta el instalador correspondiente a tu plataforma. Consulta [Descargar](download.md).
2. **Inicia sesión (opcional para la interfaz gráfica)**: la interfaz gráfica procesa imágenes de forma gratuita sin necesidad de una cuenta. Un [Chloros

+ inicio de sesión](chloros+-login.md) desbloquea el procesamiento paralelo, la aceleración por GPU, límites más altos para los dispositivos y acceso aCLI

/SDK

.
3. **Crea tu primer proyecto**: abreChloros

, crea un [nuevo proyecto](projects.md), [añade tus imágenes](processing-images-gui/adding-files-to-a-project.md) y [inicia el procesamiento](processing-images-gui/starting-the-processing.md). Si prefieres controlar hardware en tiempo real, abre la pestaña «Cámaras» o «Sensores de luz»; consulta [GUI: Navegación](navigation.md).

***

##Chloros

+

AunqueChloros

es de uso gratuito para la mayoría de las tareas, es posible que necesites más funciones. Ahí es donde una licencia de pago paraChloros

+ puede resultarte útil. Con una licencia deChloros

+ podrás acceder a nuevas funciones, tales como:

* **Procesamiento multihilo**: acelera considerablemente el procesamiento de imágenes en proyectos de mayor envergadura al procesar imágenes simultáneamente a lo largo del flujo de trabajo.
* **Aceleración por GPU (CUDA)**: aprovecha las opciones actuales de mayor memoria de GPU para acelerar aún más el proceso de procesamiento de imágenes. Recomendamos 4 GB o más de VRAM para obtener los mejores resultados.
* **Acceso aChloros

+**[**CLI**](CLI.md): ejecutaChloros

+ desde la línea de comandos para automatizar e integrar la herramienta en tu propio software. Disponible en cualquier plan de pago; se aplica del lado del servidor.
* **Chloros

+**[**API**](api-python-sdk.md) **Acceso:** ejecutaChloros

+ desdePython

para el control mediante programación, lo que permite una integración perfecta con tus procesos de investigación, flujos de trabajo de análisis de datos y aplicaciones personalizadas. Disponible en cualquier plan de pago; se aplica del lado del servidor.
* **Límites de hardware más altos**: conecta más cámaras y sensores de luz a la vez. Sin necesidad de iniciar sesión, la interfaz gráfica de usuario (GUI) permite conectar hasta 4 cámaras y 2 sensores de luz DAQ; los planes de pago aumentan ambos límites:

| Plan | Cámaras | Sensores de luz DAQ |
| --- | --- | --- |
| Iron (gratuito, sin iniciar sesión) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

* **Uso en varios dispositivos**: cada licenciaChloros

+ permite registrar 2 o más dispositivos. Utiliza tu cuenta deMAPIR

Cloud para gestionar los dispositivos registrados. Amplía la compatibilidad con más dispositivos actualizando tu licenciaChloros

+.
* **Método avanzado de eliminación de mosaicos con reconocimiento de texturas:** un método de eliminación de mosaicos de alta calidad que tiene en cuenta los bordes, combinado con un modelo de eliminación de ruido basado en IA/ML que elimina casi todo el ruido de la eliminación de mosaicos.
* **Fórmulas personalizadas de índices multiespectrales:** introduce índices multiespectrales personalizados en las calculadoras de ráster deChloros

, tanto para el procesamiento como para el entorno de pruebas de visualización de imágenes.
* **Computación en el borde (Edge Computing) yLinux

:** ejecutaChloros

enLinux

plataformas x86_64 y ARM64, incluida NVIDIA Jetson, para el procesamiento sobre el terreno y en el borde. Consulta la [Descripción general deLinux

](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Chloros+ Precios y registro</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: cli.JPG shows the 1.1.0 CLI banner. Re-shoot a terminal running `chloros-cli --version` + `chloros-cli status` on the 1.2.0 build so the banner prints "Chloros CLI 1.2.0". -->

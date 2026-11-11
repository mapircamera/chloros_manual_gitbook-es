---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Preguntas frecuentes

<details>

<summary>¿Puedo procesar con Chloros imágenes de cámaras que no sean de la marca MAPIR?</summary>

No, Chloros solo admite el procesamiento de imágenes de cámaras MAPIR: las familias Survey3 y LATTICE. Consulta la lista de [modelos de cámaras compatibles](supported-cameras.md) para obtener más información. Ofrecemos el procesamiento de otras cámaras en MAPIR Cloud; consulta la lista completa [aquí](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>¿Chloros es compatible con las cámaras LATTICE?</summary>

Sí. Chloros 1.3.0 es compatible con los módulos de cámara LATTICE M3C y M3M de principio a fin: **control en directo**—detectar, conectar, previsualizar y capturar desde la pestaña «Cámaras» de la interfaz gráfica de usuario, `chloros-cli lattice`, o desde el SDK de Python, incluidas las matrices multicámara sincronizadas con sincronización temporal PTP— y**procesamiento radiométrico completo** de las capturas (sin procesar → sin bayering → radiancia → reflectancia → índice). Consulta [Cámaras compatibles](supported-cameras.md) y la [guía de LATTICE](lattice/README.md).

</details>

<details>

<summary>¿Puedo calibrar mis imágenes para la reflectancia sin un objetivo de calibración?</summary>

**Survey3

:** No. Sin una imagen del objetivo de calibración capturada al mismo tiempo que las imágenes que no son del objetivo, no podrás relacionar los valores de píxel de la imagen con un porcentaje de reflectancia conocido. Si además no incluyes el registro de un sensor de luz MAPIR

, no se medirá el espectro de la luz ambiental y los resultados de reflectancia no serán precisos.

**LATTICE:** Sí. La reflectancia puede referenciarse a la irradiancia descendente medida por un sensor de luz DAQ en lugar de por un panel (ρ = π·L/E). Cuando *sí* hay un objetivo dentro del encuadre que ha superado el control de calidad, este se convierte en la referencia absoluta por defecto (`--reflectance-source auto`), a menos que se apruebe con un indicador, en cuyo caso el objetivo y las cadenas del DAQ compiten por la puntuación medida. Una excepción: «La reflectancia F988 se calibra utilizando un panel de reflectancia en la escena: la banda se encuentra fuera del rango calibrado del sensor de luz del DAQ, por lo que Chloros

aplica tu captura de panel más reciente y la mantiene entre observaciones del panel». Véase [Objetivos de calibración](calibration-targets.md).

</details>

<details>

<summary>¿Necesito un sensor de luz DAQ?</summary>

No para la radiancia: las exportaciones de radiancia de LATTICE proceden de la calibración radiométrica de fábrica de cada cámara y no necesitan ni un sensor DAQ ni un objetivo. Para la **reflectancia**, necesitas una referencia de la luz ambiental, ya sea una medición descendente de un sensor de luz DAQ o un objetivo de calibración dentro del encuadre. Un sensor DAQ te permite obtener valores de reflectancia calibrados**sin necesidad de colocar ningún panel en la escena**. Los archivos `.daq` grabados se asocian automáticamente a tus imágenes mediante la marca de tiempo. Consulta [Objetivos de calibración](calibration-targets.md) y la [Referencia de CLI

](reference/cli-reference.md).

</details>

<details>

<summary>¿Puedo utilizar Chloros con un asistente de IA (Claude, ChatGPT, etc.)?</summary>

Sí, este manual y la CLI/SDK están diseñados para ello:

* El índice completo del manual está disponible en `https://mapir.gitbook.io/chloros/llms.txt` para que los asistentes de IA puedan encontrar todas las páginas.
* El código Markdown sin formato de cada página está disponible en la URL de la página en minúsculas, con `.md` añadido al final (por ejemplo, `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* La [Referencia de CLI](reference/cli-reference.md) y la [Referencia de SDK](reference/sdk-reference.md) están redactadas para su uso por parte de modelos de lenguaje grande (LLM): indicadores exactos, valores predeterminados, semántica de salida y comandos que se pueden copiar y pegar.

Consulta [Asistentes de IA](ai-assistants.md) para saber cómo configurar tu asistente para que utilice Chloros.

</details>

<details>

<summary>¿Dónde se guardan mis archivos de salida procesados?</summary>

Los productos se guardan en la carpeta del proyecto, agrupados por cámara y, a continuación, por formato de archivo:

```
<project>/<camera-folder>/<format-folder>/<Product>_Images/
```

* **carpeta-de-la-cámara** — `LATT-<sensor>-<lens>-F<filter>` para LATTICE, `<model>_<filter>` (p. ej., `Survey3N_RGN`) para Survey3
* **carpeta-formato** — `tiff16`, `tiff8`, `png8`, `jpg8` o `tiff32`
* **carpetas de productos** — `Reflectance_Calibrated_Images/`, `Debayered_Images/`, `Preview_Images/`, `Radiance_Images/` (siempre dentro de `tiff32`), `<INDEX>_Index_Images/`
* **`clamp_masks/`** — Solo multiespectral de LATTICE: máscaras por píxel junto a las carpetas de productos; véase más abajo**Los archivos exportados conservan el nombre del archivo de origen; la carpeta identifica el producto, no un sufijo del nombre de archivo.**Con la CLI, la carpeta del proyecto se crea dentro de tu carpeta de proyectos predeterminada (`Documents/MAPIR/Chloros Projects`, a menos que la hayas cambiado), salvo que especifiques `-o`; una ruta relativa como `-o` se interpreta como relativa a la carpeta desde la que se ejecuta el comando. Ten en cuenta que una ejecución de `chloros-cli process` que haya solicitado productos pero no haya escrito ninguno muestra `Processing finished but wrote no image products.` y**termina con un valor distinto de cero**, por lo que los scripts pueden detectarlo. Consulta [Formatos de imagen de salida](output-image-formats.md) y la [Referencia de la CLI](reference/cli-reference.md).

</details>

<details>

<summary>¿Qué son las carpetas «clamp_masks» y los archivos *_clampmask.tif?</summary>

Novedad en la versión 1.3.0. Los productos de radiancia y reflectancia de una cámara multiespectral LATTICE M3C se someten a un proceso de desmezcla de bandas 3×3, y cuando se produce un valor negativo en una banda, Chlorosa recorta ese valor a cero. Un producto con al menos un píxel recortado recibe una máscara `uint8` de una sola banda que indica qué píxeles y bandas se han recortado (bit 0 = canal R del producto, bit 1 = G, bit 2 = B). Las máscaras se guardan en una carpeta `clamp_masks/` **junto a** las carpetas de los productos —por ejemplo, `tiff16/clamp_masks/Reflectance_Calibrated_Images/<name>_clampmask.tif` — y la etiqueta XMP del producto `Chloros:UnmixClampMask` indica el nombre de su máscara (o indica `none`).

No son imágenes de la escena: la propia función de importación de Chloros las omite y, dado que nunca se encuentran dentro de una carpeta de producto, puedes pasar una carpeta de producto a Pix4D o Metashape tal cual. Véase [Máscaras de sujeción](output-image-formats.md#clamp-masks).

</details>

<details>

<summary>¿Por qué Chloros indica «La carpeta del proyecto es demasiado profunda para este ordenador»?</summary>

Esto solo ocurre en Windows. Windows viene configurado de fábrica con las rutas largas desactivadas, por lo que ninguna ruta de archivo puede superar los 260 caracteres. Los productos de Chloros se encuentran a unos 130 caracteres por debajo de la carpeta del proyecto, por lo que Chloros rechaza una carpeta de proyecto más profunda antes de procesarla —en lugar de ejecutarse y no escribir nada, como podía ocurrir antes de la versión 1.3.0—. El mensaje indica la carpeta de proyecto más larga que cabe.

Soluciona el problema de cualquiera de estas dos formas:

* Utiliza una carpeta de proyecto más corta (el **Directorio de trabajo** en la configuración del proyecto, o `-o` / `chloros-cli set-project-folder` para la CLI).
* Activa las rutas largas de Windows y reinicia Chloros. La opción **Habilitar rutas largas de Windows** del instalador de Chloros (marcada por defecto) se encarga de ello; vuelve a ejecutar el instalador si la desmarcaste.

Consulta [Iniciar el procesamiento](processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).

</details>

<details>

<summary>¿Puedo editar mis imágenes antes de procesarlas en Chloros?</summary>

No. Chloros da por hecho que los datos de entrada no se han modificado. No cambies los nombres de los archivos.

</details>

<details>

<summary>¿Puedo configurar mis cámaras MAPIR Survey3 en exposición automática y procesar las imágenes en Chloros?</summary>

No. Los conjuntos de datos de imágenes de Survey3 deben tener una exposición fija o bloqueada, por lo que no se permite la velocidad de obturación automática ni el ISO automático. Todas las imágenes del mismo modelo de cámara deben tener la misma velocidad de obturación y el mismo ISO (exposición).

Las cámaras LATTICE no tienen esta restricción: Chloros controla su exposición en tiempo real (Smart AE), y cada captura registra la exposición y la ganancia realmente utilizadas, lo que tiene en cuenta el proceso radiométrico.

</details>

<details>

<summary>¿Puede Chloros procesar o analizar imágenes ortomosaicas?</summary>

No. Solo se admiten imágenes individuales de cámaras MAPIR, no imágenes unidas como un mapa ortomosaico.

</details>

<details>

<summary>¿Cómo puedo acelerar la etapa de detección de objetivos de Chloros?</summary>

En la tabla del explorador de archivos, al preseleccionar las imágenes objetivo en la columna de la derecha, se indica a Chloros que busque los objetivos de calibración únicamente en esas imágenes, lo que acelera considerablemente el procesamiento.

</details>

<details>

<summary>Si voy a subir mis imágenes a <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud,</a> ¿debo procesarlas en Chloros antes de subirlas?</summary>

Si tiene previsto subirlas a nuestra plataforma de procesamiento en línea [MAPIR

Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), no edite las imágenes antes de subirlas. Cloud realizará todo el mismo procesamiento y mucho más.

</details>

<details>

<summary>¿MAPIR ofrecerá alguna vez la función X? Me gustaría mucho que MAPIR ofreciera X.</summary>

Siempre nos interesa recibir comentarios sobre nuestros productos. Si detectas algún problema con nuestros productos o tienes alguna sugerencia sobre cómo podemos mejorarlos, por favor, [CONTÁCTANOS](https://www.mapir.camera/community/contact) para compartir tus opiniones. La mayor parte de nuestra I+D se basa en escuchar las principales necesidades de nuestros clientes.

</details>

<details>

<summary>¿Chloros está disponible para Linux?</summary>

¡Sí! Chloros 1.3.0 es compatible con Linux amd64 (x86_64) y arm64 (NVIDIA Jetson JetPack 6) a través de los paquetes `.deb`. La interfaz de línea de comandos (CLI) y el SDK de Python son totalmente compatibles con Linux, incluido el control en tiempo real de cámaras LATTICE y sensores DAQ. No hay interfaz gráfica de usuario (GUI) para Linux: toda la interacción se realiza a través de la [CLI](CLI.md) o del [SDK de Python](api-python-sdk.md). Consulta la [Descripción general de Linux](linux/linux-overview.md) para obtener más detalles.

</details>

<details>

<summary>¿Puedo ejecutar Chloros en NVIDIA Jetson?</summary>

¡Sí! Chloros es compatible con la **familia Orin** de NVIDIA Jetson —Orin Nano, Orin NX y AGX Orin— que ejecutan JetPack 6. Los modelos más antiguos (Jetson Nano, TX2, Xavier NX) no pueden ejecutar JetPack 6 y no son compatibles con el paquete actual. Chloros detecta automáticamente tu modelo de Jetson y optimiza su estrategia de procesamiento. El paquete para Jetson utiliza la versión de PyTorch para CUDA de JetPack 6, por lo que el procesamiento se ejecuta en la GPU del Jetson. Consulta la [Guía de NVIDIA Jetson](linux/nvidia-jetson-guide.md) para obtener instrucciones de configuración e implementación.

</details>

<details>

<summary>¿Chloros se optimiza automáticamente para mi hardware?</summary>

¡Sí! Chloros incluye la [Adaptación dinámica del cálculo](processing-architecture/dynamic-compute-adaptation.md), que detecta automáticamente tu CPU, GPU, RAM y (en Jetson) los sensores térmicos. A continuación, selecciona la estrategia de procesamiento óptima — desde `GPU_PARALLEL` en GPUs con gran capacidad de memoria hasta `GPU_SINGLE` en las más pequeñas, pasando por `CPU_PARALLEL` en sistemas sin una GPU CUDA que PyTorch pueda utilizar. No es necesaria ninguna configuración manual.

</details>

<details>

<summary>¿En qué consiste el proceso de procesamiento de 4 subprocesos?</summary>

Chloros
utiliza una arquitectura en pipeline de 4 subprocesos para los usuarios de Chloros
+: El hilo 1 (Detección) carga imágenes y detecta los objetivos de calibración; el hilo 2 (Calibración) calcula la calibración de reflectancia; el hilo 3 (Procesamiento) realiza el «debayering» acelerado por GPU y el cálculo de índices; y el hilo 4 (Exportación) escribe los archivos de salida. Se pueden procesar varias imágenes en diferentes subprocesos simultáneamente para obtener el máximo rendimiento. Consulta [Cadena de procesamiento](processing-architecture/processing-pipeline.md) para obtener más detalles.

</details>

<details>

<summary>¿Cómo puedo ejecutar pruebas de diagnóstico en mi instalación de Chloros?</summary>

Utiliza el comando `selftest` para ejecutar una prueba básica de 7 pasos: versión, disponibilidad de puertos, inicio del backend, conectividad de la API (`/api/test`), información del sistema (`/api/system-info` — GPU/CUDA/PyTorch), presencia del modelo de eliminación de ruido y disponibilidad de CUDA y del modelo de eliminación de ruido:

```bash
chloros-cli selftest
```

Esto resulta especialmente útil en sistemas de Linux para verificar la configuración de la GPU y CUDA. En un Jetson, `CUDA: False` suele indicar que faltan las bibliotecas CUDA y cuDNN de JetPack 6: instálalas con `sudo apt install nvidia-jetpack`.

</details>

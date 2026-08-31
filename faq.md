---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Preguntas frecuentes

<details>

<summary>¿Puedo procesar imágenes de cámaras que no sean de la marca MAPIR con el Chloros?</summary>

No, Chloros solo admite el procesamiento de imágenes de cámaras MAPIR, es decir, las familias Survey3 y LATTICE. Consulta la lista de [modelos de cámaras compatibles](supported-cameras.md) para obtener más información. Ofrecemos el procesamiento de otras cámaras en MAPIR Cloud; consulta la lista completa [aquí](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>¿Es compatible Chloros con las cámaras LATTICE?</summary>

Sí. Chloros 1.2.0 es compatible con los módulos de cámara LATTICE M3C y M3M de principio a fin: **control en directo**— detectar, conectar, previsualizar y capturar desde la pestaña «Cámaras» de la interfaz gráfica de usuario, `chloros-cli lattice` o en Python SDK, incluidas las matrices multicámara sincronizadas con sincronización temporal PTP — y**procesamiento radiométrico completo** de las capturas (sin procesar → sin bayering → radiancia → reflectancia → índice). Consulte [Cámaras compatibles](supported-cameras.md) y la [guía de LATTICE](lattice/README.md).

</details>

<details>

<summary>¿Puedo calibrar mis imágenes para la reflectancia sin un objetivo de calibración?</summary>

**Survey3:** No. Sin una imagen del objetivo de calibración capturada al mismo tiempo que las imágenes que no son del objetivo, no podrás relacionar los valores de píxel de la imagen con un porcentaje de reflectancia conocido. Si además no incluye el registro de un sensor de luz MAPIR, no se medirá el espectro de la luz ambiental y los resultados de reflectancia no serán precisos.**LATTICE:** Sí. La reflectancia puede referirse a la irradiancia descendente medida por un sensor de luz DAQ en lugar de por un panel (ρ = π·L/E). Cuando *sí* hay un objetivo en el encuadre que ha superado el control de calidad, este se convierte en la referencia absoluta por defecto (`--reflectance-source auto`). Una excepción: «La reflectancia F988 se calibra utilizando un panel de reflectancia en la escena: la banda se encuentra fuera del rango calibrado del sensor de luz DAQ, por lo que Chloros aplica tu captura de panel más reciente y la mantiene entre observaciones del panel». Véase [Objetivos de calibración](calibration-targets.md).

</details>

<details>

<summary>¿Necesito un sensor de luz DAQ?</summary>

No para la radiancia: las exportaciones de radiancia de LATTICE proceden de la calibración radiométrica de fábrica de cada cámara y no necesitan ni un sensor DAQ ni un objetivo. Para la **reflectancia**, necesitas una referencia de la luz ambiental, ya sea una medición descendente de un sensor de luz DAQ o un objetivo de calibración dentro del encuadre. Un sensor DAQ te permite obtener datos de reflectancia calibrados**sin necesidad de colocar ningún panel en la escena**. Los archivos `.daq` grabados se asocian automáticamente a tus imágenes mediante la marca de tiempo. Consulta [Objetivos de calibración](calibration-targets.md) y la [Referencia CLI](reference/cli-reference.md).

</details>

<details>

<summary>¿Puedo utilizar Chloros con un asistente de IA (Claude, ChatGPT, etc.)?</summary>

Sí; este manual y los archivos CLI/SDK están diseñados para ello:

* El índice completo del manual está disponible en `https://mapir.gitbook.io/chloros/llms.txt` para que los asistentes de IA puedan encontrar todas las páginas.
* El código Markdown sin formato de cada página está disponible en su página en minúsculas URL con `.md` añadido al final (por ejemplo, `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* La [Referencia de CLI](reference/cli-reference.md) y [Referencia de SDK](reference/sdk-reference.md) están redactadas para su uso con modelos de lenguaje grande (LLM): indicadores exactos, valores por defecto, semántica de salida y comandos que se pueden copiar y pegar.

Consulta [Asistentes de IA](ai-assistants.md) para saber cómo configurar tu asistente para que utilice Chloros.

</details>

<details>

<summary>¿Dónde se guardan mis archivos de salida procesados?</summary>

Los productos se guardan en la carpeta del proyecto, agrupados por cámara y, a continuación, por formato de archivo:

```
<project>/<camera-folder>/<format-folder>/<Product>_Images/
```

* **carpeta-cámara** — `LATT-<sensor>-<lens>-F<filter>` para LATTICE, `<model>_<filter>` (p. ej., `Survey3N_RGN`) para Survey3
* **carpeta-formato** — `tiff16`, `tiff8`, `png8`, `jpg8` o `tiff32`
* **carpetas de productos** — `Reflectance_Calibrated_Images/`, `Debayered_Images/`, `Preview_Images/`, `Radiance_Images/` (siempre bajo `tiff32`), `<INDEX>_Index_Images/`**Los archivos exportados conservan el nombre del archivo de origen: la carpeta identifica el producto, no un sufijo del nombre de archivo.**Con el CLI, la carpeta del proyecto se crea junto a la carpeta de entrada, a menos que se pase `-o`. Ten en cuenta que una ejecución de `chloros-cli process` que haya solicitado productos pero no haya generado ninguno muestra el mensaje `Processing finished but wrote no image products.` y**termina con un valor distinto de cero**, por lo que los scripts pueden detectarlo. Consulta [Formatos de imagen de salida](output-image-formats.md) y la [Referencia de CLI](reference/cli-reference.md).

</details>

<details>

<summary>¿Puedo editar mis imágenes antes de procesarlas en Chloros?</summary>

No. Chloros da por hecho que los datos de entrada no se han modificado. No cambies los nombres de los archivos.

</details>

<details>

<summary>¿Puedo configurar mis cámaras MAPIR y Survey3 en exposición automática y procesar las imágenes en Chloros?</summary>

No. Los conjuntos de datos de imágenes de Survey3 deben tener una exposición fija o bloqueada, por lo que no se permite la velocidad de obturación automática ni el ISO automático. Todas las imágenes del mismo modelo de cámara deben tener la misma velocidad de obturación y el mismo ISO (exposición).

Las cámaras LATTICE no tienen esta restricción: Chloros controla su exposición en tiempo real (Smart AE), y cada captura registra la exposición y la ganancia realmente utilizadas, lo cual tiene en cuenta el proceso radiométrico.

</details>

<details>

<summary>¿Puede Chloros procesar o analizar imágenes ortomosaicas?</summary>

No. Solo se admiten imágenes individuales de la cámara MAPIR, no imágenes unidas como un mapa ortomosaico.

</details>

<details>

<summary>¿Cómo puedo acelerar el paso de detección de objetivos de Chloros?</summary>

En la tabla del explorador de archivos, al preseleccionar las imágenes objetivo en la columna de la derecha, se indica a Chloros que busque los objetivos de calibración únicamente en esas imágenes, lo que acelera considerablemente el procesamiento.

</details>

<details>

<summary>Si voy a subir mis imágenes a <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud,</a> ¿debo procesarlas en Chloros antes de subirlas?</summary>

Si tienes pensado subirlas a nuestra plataforma de procesamiento en línea [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), no edites las imágenes antes de subirlas. Cloud llevará a cabo todo el procesamiento habitual y mucho más.

</details>

<details>

<summary>¿Llegará MAPIR a ser compatible con la función X? Me gustaría mucho que MAPIR ofreciera X.</summary>

Siempre nos interesa recibir comentarios sobre nuestros productos. Si detectas algún problema con nuestros productos o tienes alguna sugerencia sobre cómo podemos mejorarlos, por favor, [CONTÁCTANOS](https://www.mapir.camera/community/contact) para compartir tus opiniones. La mayor parte de nuestra I+D se basa en escuchar las principales necesidades de nuestros clientes.

</details>

<details>

<summary>¿Está disponible Chloros para Linux?</summary>

¡Sí! Chloros 1.2.0 es compatible con Linux amd64 (x86_64) y arm64 (NVIDIA Jetson JetPack 6) a través de los paquetes `.deb`. CLI y Python SDK son totalmente compatibles con Linux, incluido el control en tiempo real de cámaras LATTICE y sensores DAQ. No hay interfaz gráfica de usuario (GUI) para Linux ; toda la interacción se realiza a través de [CLI](CLI.md) o [Python SDK](api-python-sdk.md). Consulte la [Descripción general de Linux](linux/linux-overview.md) para obtener más detalles.

</details>

<details>

<summary>¿Puedo ejecutar Chloros en NVIDIA Jetson?</summary>

¡Sí! Chloros es compatible con las plataformas NVIDIA Jetson, incluidas Jetson Nano, Orin Nano, Orin NX y AGX Orin que ejecutan JetPack 6. Chloros detecta automáticamente tu modelo de Jetson y optimiza su estrategia de procesamiento. Consulta la [Guía de NVIDIA Jetson](linux/nvidia-jetson-guide.md) para obtener instrucciones de configuración e implementación.

</details>

<details>

<summary>¿Se optimiza Chloros automáticamente para mi hardware?</summary>

¡Sí! Chloros incluye la [Adaptación dinámica de cálculo](processing-architecture/dynamic-compute-adaptation.md), que detecta automáticamente la CPU, la GPU, la RAM y (en Jetson) los sensores térmicos. A continuación, selecciona la estrategia de procesamiento óptima: desde `GPU_PARALLEL` en sistemas con mucha memoria hasta `GPU_SINGLE` en dispositivos con recursos limitados, pasando por `CPU_PARALLEL` en sistemas sin una GPU de NVIDIA. No es necesaria ninguna configuración manual.

</details>

<details>

<summary>¿Qué es la canalización de procesamiento de 4 subprocesos?</summary>

Chloros utiliza una arquitectura en pipeline de 4 subprocesos para los usuarios de Chloros+: El hilo 1 (Detección) carga imágenes y detecta objetivos de calibración; el hilo 2 (Calibración) calcula la calibración de reflectancia; el hilo 3 (Procesamiento) realiza el debayering acelerado por GPU y el cálculo del índice; y el hilo 4 (Exportación) escribe los archivos de salida. Se pueden procesar varias imágenes en diferentes subprocesos simultáneamente para obtener el máximo rendimiento. Consulta [Cadena de procesamiento](processing-architecture/processing-pipeline.md) para obtener más detalles.

</details>

<details>

<summary>¿Cómo puedo ejecutar pruebas de diagnóstico en mi instalación de Chloros?</summary>

Utilice el comando `selftest` para ejecutar una prueba de funcionamiento de 7 pasos: versión, disponibilidad de puertos, inicio del backend, conectividad de API (`/api/test`), información del sistema (`/api/system-info` — GPU/CUDA/PyTorch), presencia del modelo de eliminación de ruido y disponibilidad de CUDA + el eliminador de ruido:

```bash
chloros-cli selftest
```

Esto resulta especialmente útil en sistemas Linux/Jetson para verificar la configuración de la GPU y CUDA.

</details>

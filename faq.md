---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Preguntas frecuentes

<details>

<summary>¿Puedo procesar imágenes de cámaras que no sean de la marca MAPIR con Chloros?</summary>

No, Chloros solo admite el procesamiento de imágenes de cámaras MAPIR. Consulte la lista de [modelos de cámaras compatibles](supported-cameras.md) para obtener más información. Ofrecemos el procesamiento de otras cámaras en MAPIR Cloud; consulte la lista completa [aquí](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>¿Puedo calibrar mis imágenes para la reflectancia sin un objetivo de calibración?</summary>

No. Sin una imagen del objetivo de calibración capturada al mismo tiempo que las imágenes sin objetivo, no podrá relacionar los valores de píxeles de la imagen con un porcentaje de reflectancia conocido. Si además no incluye el registro de un sensor de luz MAPIR, no se medirá el espectro de luz ambiental y los resultados de reflectancia no serán precisos.

</details>

<details>

<summary>¿Puedo editar mis imágenes antes de procesarlas en Chloros?</summary>

No. Chloros asume que los datos de entrada no han sido modificados. No cambie los nombres de los archivos.

</details>

<details>

<summary>¿Puedo configurar mis cámaras MAPIR y Survey3 en exposición automática y procesar las imágenes en Chloros?</summary>

No. Los conjuntos de datos de imágenes Survey3 deben tener una exposición fija/bloqueada, por lo que no se permite la velocidad de obturación automática ni el ISO automático. Todas las imágenes del mismo modelo de cámara deben tener una velocidad de obturación y un ISO (exposición) idénticos.

</details>

<details>

<summary>¿Puede Chloros procesar o analizar imágenes ortomosaicas?</summary>

No. Solo se admiten imágenes individuales de cámara MAPIR, no imágenes unidas como un mapa ortomosaico.

</details>

<details>

<summary>¿Cómo puedo acelerar el paso de detección de objetivos de Chloros?</summary>

En la tabla del explorador de archivos, al preseleccionar las imágenes objetivo en la columna de la derecha, se indicará a Chloros que busque objetivos de calibración únicamente en esas imágenes, lo que acelerará considerablemente el procesamiento.

</details>

<details>

<summary>Si voy a subir mis imágenes a <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud,</a> ¿debo procesarlas en Chloros antes de subirlas?</summary>

Si tiene previsto subirlas a nuestra plataforma de procesamiento en línea [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), no edite las imágenes antes de subirlas. Cloud realizará todo el mismo procesamiento y mucho más.

</details>

<details>

<summary>¿MAPIR llegará a ser compatible con la función X? Me gustaría mucho que MAPIR ofreciera X.</summary>

Siempre nos interesa recibir comentarios sobre nuestros productos. Si detecta algún problema con nuestros productos o tiene alguna sugerencia sobre cómo podemos mejorarlos, póngase en [CONTACTO CON NOSOTROS](https://www.mapir.camera/community/contact) para compartir sus opiniones. La mayor parte de nuestra I+D se basa en escuchar las principales necesidades de nuestros clientes.

</details>

<details>

<summary>¿Está Chloros disponible para Linux?</summary>

¡Sí! Chloros 1.1.0 es compatible con Linux amd64 (x86_64) y arm64 (NVIDIA Jetson JetPack 6) a través de los paquetes `.deb`. CLI y Python SDK son totalmente compatibles con Linux. No hay interfaz gráfica de usuario para Linux: toda la interacción se realiza a través de [CLI](CLI.md) o [Python SDK](api-python-sdk.md). Consulte [Descripción general de Linux](linux/linux-overview.md) para obtener más detalles.

</details>

<details>

<summary>¿Puedo ejecutar Chloros en NVIDIA Jetson?</summary>

¡Sí! Chloros 1.1.0 es compatible con las plataformas NVIDIA Jetson, incluidas Jetson Nano, Orin Nano, Orin NX y AGX Orin con JetPack 6. Chloros detecta automáticamente su modelo de Jetson y optimiza su estrategia de procesamiento. Consulte la [Guía de NVIDIA Jetson](linux/nvidia-jetson-guide.md) para obtener instrucciones de configuración e implementación.

</details>

<details>

<summary>¿Chloros se optimiza automáticamente para mi hardware?</summary>

¡Sí! Chloros 1.1.0 incluye [Adaptación dinámica de computación](processing-architecture/dynamic-compute-adaptation.md) que detecta automáticamente tu CPU, GPU, RAM y (en Jetson) sensores térmicos. A continuación, selecciona la estrategia de procesamiento óptima: desde `GPU_PARALLEL` en sistemas con mucha memoria hasta `GPU_SINGLE` en dispositivos con recursos limitados y `CPU_PARALLEL` en sistemas sin una GPU NVIDIA. No es necesaria ninguna configuración manual.

</details>

<details>

<summary>¿Qué es la canalización de procesamiento de 4 subprocesos?</summary>

Chloros 1.1.0 utiliza una arquitectura en pipeline de 4 subprocesos para los usuarios de Chloros+: El subproceso 1 (Detección) carga imágenes y detecta objetivos de calibración; el subproceso 2 (Calibración) calcula la calibración de reflectancia; el subproceso 3 (Procesamiento) realiza el debayering acelerado por GPU y el cálculo de índices; y el subproceso 4 (Exportación) escribe los archivos de salida. Se pueden tener varias imágenes en diferentes subprocesos simultáneamente para obtener el máximo rendimiento. Consulte [Cadena de procesamiento](processing-architecture/processing-pipeline.md) para obtener más detalles.

</details>

<details>

<summary>¿Cómo ejecuto diagnósticos en mi instalación de Chloros?</summary>

Utilice el comando `selftest` para ejecutar 7 diagnósticos del sistema, incluyendo comprobación de versión, disponibilidad de puertos, inicio del backend, conectividad de API, información del sistema, modelos de denoiser y disponibilidad de CUDA:

```bash
chloros-cli selftest
```

Esto resulta especialmente útil en sistemas Linux/Jetson para verificar la configuración de la GPU y CUDA.

</details>

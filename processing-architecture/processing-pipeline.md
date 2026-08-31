# Canal de procesamiento

Chloros1.2.0 utiliza un canal de procesamiento de 4 subprocesos que funciona como una cadena de montaje por etapas. Cada subproceso se encarga de una fase distinta del flujo de trabajo, por lo que pueden haber varias imágenes en proceso en diferentes etapas al mismo tiempo.

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

***

## Arquitectura del proceso

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Cada imagen recorre los cuatro subprocesos en orden. Con el procesamiento multihilo de Chloros+, varias imágenes ocupan diferentes subprocesos simultáneamente: mientras el subproceso 3 procesa una imagen, el subproceso 1 puede estar detectando la siguiente, el subproceso 2 calibrando otra y el subproceso 4 guardando en el disco una imagen ya terminada.

El progreso se comunica por hilo y se transmite a través de Server-Sent Events (el backend los publica en `/api/events`). En la pantalla de progreso en tiempo real de la «CLI», las cuatro etapas se denominan **Detección, Análisis, Procesamiento, Exportación**.***

## Detalles de los subprocesos

### Subproceso 1: Detección

**Objetivo**: Cargar imágenes y detectar objetivos de calibración.

* Lee archivos de imagen del disco — pares Survey3 `.raw`+`.jpg`, capturas de LATTICE `.tif`/`.tiff` y `.dng`
* Extrae metadatos EXIF (GPS, modelo de cámara, marcas de tiempo, exposición)
* Detecta objetivos de calibración: geometrías de objetivos marcadas con ArUco para las capturas de LATTICE, y el detector de panel clásico para las fotos de objetivos de calibración de Survey3
* Resultados: datos de imagen + metadatos + resultados de la detección de objetivos

Hilo limitado principalmente por E/S y CPU.

### Hilo 2: Calibración

**Objetivo**: Calcular los parámetros de calibración a partir de los objetivos detectados.

* Calcula los coeficientes de calibración de reflectancia a partir de las imágenes de los objetivos
* Calcula los parámetros de corrección de viñeteado
* Determina las curvas de calibración por banda
* Resultados: parámetros de calibración para cada imagen

Un hilo de cálculo limitado por la CPU. El hilo 3 espera a que finalice cuando la calibración de reflectancia está habilitada, de modo que sus coeficientes estén listos antes de que se procese ninguna imagen.

### Hilo 3: Procesamiento (GPU)

**Objetivo**: Aplicar correcciones y calcular índices de vegetación.**Este es el hilo que requiere mayor potencia de cálculo.*** **Debayering**: convierte los datos RAW de Bayer en imágenes multicanal
  * Estándar (rápido, calidad media): predeterminado, `--debayer standard`
  * Sensible a la textura (lento, máxima calidad): solo «Chloros», `--debayer texture-aware`; utiliza un modelo de eliminación de ruido basado en IA/ML
  * Las capturas LATTICE mono (M3M) son de una sola banda: en ellas se omiten los pasos de demosaico y balance de blancos (con un mensaje de registro de una línea), mientras que las imágenes M3C/Bayer de la misma ejecución sí se someten a dichos pasos
* **Corrección de viñeteado**: aplica la corrección del viñeteado del objetivo a toda la imagen
* **Calibración de reflectancia**: aplica coeficientes de calibración para convertir los valores a reflectancia
* **Cálculo de índices**: calcula índices de vegetación (NDVI, NDRE, GNDVI, …)
* Resultados: datos de imagen procesados listos para su exportación

Este hilo es el que más se beneficia de la aceleración por GPU, y es el hilo que ajusta la [Adaptación dinámica de cálculo](dynamic-compute-adaptation.md).

### Hilo 4: Exportación

**Objetivo**: escribir las imágenes procesadas en el disco.

* Escribe los archivos de salida en el formato seleccionado: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`
* Incorpora metadatos en los archivos de salida (GPS, marcas de tiempo, parámetros de procesamiento)
* Organiza los archivos de salida en la carpeta del proyecto como `<camera>/<format>/<Product>_Images/` — por ejemplo, `LATT-M3M-L41-F550/tiff16/Reflectance_Calibrated_Images/`. **Los archivos exportados conservan el nombre del archivo de origen; la carpeta identifica el producto.**
* En el caso de las capturas de LATTICE, un fotograma de origen puede dar lugar a varios productos (sin bayering, vista previa, radiancia, reflectancia, índice), cada uno en su propia carpeta de productos
* Resultados: archivos finales en el disco

Se trata principalmente de un hilo limitado por la E/S; el almacenamiento en SSD mejora notablemente su rendimiento.

***

## Detrás de las cámaras: los ejecutores

Dentro del hilo 3, el trabajo por imagen se paraleliza con el `concurrent.futures` estándar de Python:

* **Las estrategias de GPU**(`GPU_SINGLE`, `GPU_PARALLEL`) utilizan un método**spawn** de `ProcessPoolExecutor` — cada trabajador es un proceso independiente con su propio contexto CUDA (`fork` heredaría el estado CUDA inicializado del proceso padre y corrompería los procesos hijos)
* **`CPU_PARALLEL`** utiliza un `ThreadPoolExecutor`: NumPy y OpenCV liberan el GIL, por lo que los subprocesos son suficientes
* Los dispositivos Jetson con 8 GB o menos de RAM compartida omiten por completo el ejecutor y procesan de forma secuencial dentro del mismo proceso
* Texture Aware en una GPU con menos de 7 GB de VRAM también se ejecuta secuencialmente: el modelo de eliminación de ruido no cabe más de una vez

Chlorosno utiliza ningún marco de trabajo distribuido de terceros (como Ray). Consulta [Adaptación dinámica del cálculo](dynamic-compute-adaptation.md) para saber cómo se eligen la estrategia y el número de trabajadores.

***

## Procesamiento secuencial frente a procesamiento en pipeline

### Modo libre (secuencial)

En la versión gratuita de Chloros, las imágenes se procesan **una a una**, de forma secuencial a lo largo de las cuatro etapas:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

La interfaz gráfica de usuario muestra una barra de progreso simplificada en el modo gratuito; sus fases secuenciales se indican como **Detección de objetivos**y, a continuación,**Procesamiento**.

### Modo «Chloros» (en pipeline)

Con una licencia «Chloros», los cuatro subprocesos operan **simultáneamente** sobre imágenes diferentes:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

La barra de progreso de la interfaz gráfica muestra las cuatro etapas; pasa el cursor por encima para ver el progreso de cada hilo. En el CLI, esas mismas cuatro etapas se muestran en tiempo real como **Detectando, Analizando, Procesando, Exportando**.

{% hint style="info" %}
**Una etiqueta, dos nombres.** El CLI denomina a la etapa 3 _Procesamiento_. La transmisión de progreso del modo premium del backend —la que muestra la barra de progreso de la interfaz gráfica— etiqueta la misma etapa como _Calibración_. Se trata del mismo hilo realizando el mismo trabajo (Hilo 3: debayer, correcciones, índices).
{% endhint %}

{% hint style="success" %}
**El procesamiento en pipeline con Chloros+** puede ser entre 3 y 5 veces más rápido que el procesamiento secuencial, dependiendo del hardware y del tamaño del conjunto de datos. La aceleración es mayor en sistemas con GPU y SSD rápidas.
{% endhint %}

***

## Progreso de la exportación del hilo 4

El hilo de exportación tiene su propio seguimiento de progreso, que puedes consultar por separado:**CLI:**

```bash
chloros-cli export-status
```

**SDK:**

```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

El procesamiento habrá finalizado cuando el hilo 4 alcance el 100 %.

{% hint style="info" %}
**Una ejecución que no graba ninguna imagen se considera un error.**Si se realiza con éxito, `chloros-cli process` informa de cuántos productos de imagen se han grabado (`Image products written: N`). Si se solicitaron productos y**ninguno**se escribió —solo `project.json` y `calibration_data.json`—, el CLI imprime `Processing finished but wrote no image products.` y**sale con un valor distinto de cero**, indicando el nombre de la carpeta del proyecto y las causas habituales (la carpeta de entrada no se reconoció como una captura —comprueba la disposición y `--input-level`— o todos los productos solicitados eran inaplicables a esas cámaras). Los scripts pueden basarse en el código de salida.
{% endhint %}

***

## Relación con la adaptación dinámica de cálculo

[La adaptación dinámica de cálculo](dynamic-compute-adaptation.md) afecta principalmente al **hilo 3 (procesamiento)**:

* **`GPU_PARALLEL`**: el hilo 3 procesa varias imágenes a través de la GPU simultáneamente utilizando el canal de procesamiento `fused_gpu`
* **`GPU_SINGLE`**: El hilo 3 serializa el acceso a la GPU con un semáforo mientras los procesos de trabajo superponen las operaciones de E/S, utilizando `fused_gpu` o el canal `tiled_gpu`, que hace un uso eficiente de la memoria
* **`CPU_PARALLEL`**: El hilo 3 utiliza un procesamiento basado en la CPU con paralelismo multihilo

La asignación de memoria de la GPU del hilo 3 también aumenta a medida que los hilos 1 y 2 finalizan; véase [Asignación dinámica de memoria de la GPU](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Próximos pasos

* [Adaptación dinámica del cálculo](dynamic-compute-adaptation.md): cómo «Chloros» selecciona la estrategia óptima para tu hardware
* [Guía de NVIDIA Jetson](../linux/nvidia-jetson-guide.md): comportamiento del pipeline específico de la plataforma en Jetson
* [Supervisión del procesamiento](../processing-images-gui/monitoring-the-processing.md): supervisión del progreso a través de la interfaz gráfica de usuario
* [Referencia de CLI](../reference/cli-reference.md) — `process`, `export-status`, códigos de salida y estructura de salida

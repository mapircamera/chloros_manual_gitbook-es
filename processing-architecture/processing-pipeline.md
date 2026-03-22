# Canalización de procesamiento

Chloros 1.1.0 utiliza una canalización de procesamiento de 4 subprocesos que funciona como una cadena de montaje por etapas. Cada subproceso se encarga de una fase distinta del flujo de trabajo de procesamiento, lo que permite procesar varias imágenes simultáneamente en diferentes etapas.

***

## Arquitectura de la canalización

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Cada imagen recorre los cuatro subprocesos en orden. Con el procesamiento multihilo de Chloros+, varias imágenes pueden encontrarse en diferentes subprocesos simultáneamente: mientras el subproceso 3 procesa una imagen, el subproceso 1 puede estar detectando la siguiente, el subproceso 2 puede estar calibrando otra y el subproceso 4 puede estar escribiendo en el disco una imagen procesada anteriormente.

***

## Detalles de los subprocesos

### Hilo 1: Detección

**Objetivo**: Cargar imágenes y detectar objetivos de calibración.

* Lee archivos de imagen del disco (RAW, JPG)
* Extrae metadatos EXIF (GPS, modelo de cámara, marcas de tiempo, exposición)
* Detecta objetivos de calibración ArUco en imágenes de objetivos marcadas
* Resultados: datos de imagen + metadatos + resultados de detección de objetivos

Este es principalmente un subproceso limitado por E/S y CPU.

### Subproceso 2: Calibración

**Objetivo**: Calcular los parámetros de calibración a partir de los objetivos detectados.

* Calcula los coeficientes de calibración de reflectancia a partir de las imágenes de objetivos
* Calcula los parámetros de corrección de viñeteado
* Determina las curvas de calibración por banda
* Resultados: parámetros de calibración para cada imagen

Este es un hilo de cálculo dependiente de la CPU.

### Hilo 3: Procesamiento (GPU)

**Objetivo**: Aplicar correcciones y calcular índices de vegetación.**Este es el hilo que requiere mayor potencia de cálculo.*** **Debayering**: Convierte datos RAW con patrón Bayer en imágenes multicanal
  * Estándar (rápido, calidad media) — predeterminado
  * Sensible a la textura (lento, máxima calidad) — solo Chloros+, utiliza eliminación de ruido mediante IA/ML
* **Corrección de viñeteado**: Aplica la corrección de viñeteado de la lente a toda la imagen
* **Calibración de reflectancia**: aplica coeficientes de calibración para convertir a valores de reflectancia
* **Cálculo de índices**: calcula índices de vegetación (NDVI, NDRE, GNDVI, etc.)
* Resultados: datos de imagen procesados listos para exportar

Este subproceso es el que más se beneficia de la aceleración por GPU. El sistema de [Adaptación dinámica de cálculo](dynamic-compute-adaptation.md) optimiza principalmente el comportamiento de este subproceso.

### Hilo 4: Exportación

**Objetivo**: Grabar las imágenes procesadas en el disco.

* Graba los archivos de salida en el formato seleccionado (TIFF 16 bits, TIFF 32 bits %, PNG, JPG)
* Incorpora metadatos EXIF en los archivos de salida (GPS, marcas de tiempo, parámetros de procesamiento)
* Organiza los resultados en subcarpetas por modelo de cámara
* Resultados: archivos finales en el disco

Este es principalmente un hilo limitado por E/S. El almacenamiento en SSD mejora significativamente el rendimiento del Hilo 4.

***

## Procesamiento secuencial frente a procesamiento en pipeline

### Modo gratuito (secuencial)

En la versión gratuita de Chloros, las imágenes se procesan **una a una**, de forma secuencial a lo largo de las cuatro etapas:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

La barra de progreso de la interfaz gráfica de usuario muestra dos etapas: Detección de objetivos y Procesamiento.

### Modo Chloros+ (en pipeline)

Con una licencia Chloros+, los cuatro subprocesos operan **simultáneamente** en diferentes imágenes:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

La barra de progreso de la interfaz gráfica de usuario muestra 4 etapas: Detección, Análisis, Calibración y Exportación. Pase el cursor por encima de la barra de progreso para ver el progreso de cada subproceso.

{% hint style="success" %}
**El procesamiento en pipeline con Chloros+** puede ser entre 3 y 5 veces más rápido que el procesamiento secuencial, dependiendo de su hardware y del tamaño del conjunto de datos. La aceleración es mayor en sistemas con GPU y SSD rápidas.
{% endhint %}

***

## Progreso de la exportación del hilo 4

En Chloros 1.1.0, el hilo de exportación (hilo 4) cuenta con su propio seguimiento de progreso dedicado. Puede supervisar el progreso de la exportación por separado:**CLI:**
```bash
chloros-cli export-status
```

**SDK:**
```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

El procesamiento se completa cuando el Hilo 4 alcanza el 100 %.

***

## Relación con la adaptación dinámica de computación

El sistema de [adaptación dinámica de computación](dynamic-compute-adaptation.md) afecta principalmente al **hilo 3 (procesamiento)**:

* **Estrategia `GPU_PARALLEL`**: el hilo 3 procesa varias imágenes a través de la GPU simultáneamente utilizando el canal `fused_gpu`
* Estrategia **`GPU_SINGLE`**: el hilo 3 procesa una imagen cada vez utilizando el canal `tiled_gpu`, eficiente en memoria
* Estrategia **`CPU_PARALLEL`**: el hilo 3 utiliza procesamiento basado en la CPU con paralelismo multihilo

La asignación de memoria de la GPU del subproceso 3 también cambia dinámicamente a medida que los subprocesos 1 y 2 completan su ejecución; consulte [Asignación dinámica de memoria de la GPU](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Próximos pasos

* [Adaptación dinámica de la computación](dynamic-compute-adaptation.md) — Cómo Chloros selecciona la estrategia óptima para su hardware
* [Guía de NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Comportamiento del pipeline específico de la plataforma en Jetson
* [Supervisión del procesamiento](../processing-images-gui/monitoring-the-processing.md) — Supervisión del progreso mediante la interfaz gráfica de usuario

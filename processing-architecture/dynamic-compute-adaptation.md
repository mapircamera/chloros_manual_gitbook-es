# Adaptación dinámica del procesamiento

Chloros 1.1.0 introduce la detección inteligente de hardware y la selección automática de estrategias de procesamiento. El motor de procesamiento se adapta a tu hardware —desde un Jetson Nano hasta una estación de trabajo con varias GPU— sin necesidad de configuración manual.

***

## Cómo funciona

Cuando se inicia Chloros, analiza automáticamente su sistema:

1. **Detecta el sistema operativo**: Windows o Linux
2. **Identifica los núcleos de la CPU y la RAM total**

3.**Detecta la presencia de GPU** — Capacidad NVIDIA CUDA, VRAM, modelo
4. **Identifica el modelo Jetson** (si procede) — a través de `/proc/device-tree/model`
5. **Comprueba los sensores térmicos** (Jetson) — para un procesamiento sensible a la temperatura
6. **Selecciona la estrategia de cálculo óptima** — basada en todo el hardware detectado
7. **Configura el número de trabajadores, el tipo de canalización y la asignación de memoria** automáticamente

El resultado se almacena en caché para que las ejecuciones posteriores se inicien más rápido. Si el hardware cambia (por ejemplo, se añade una GPU), Chloros vuelve a perfilar en el siguiente inicio.

***

## Estrategias de cálculo

Chloros selecciona una de las tres estrategias de cálculo en función de su hardware:

| Estrategia | GPU requerida | Trabajadores | Canalización | Ideal para |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`** | Sí (12 GB+ de VRAM o 16 GB+ compartidos) | 3-4 | `fused_gpu` | GPU de escritorio con 12 GB+, Jetson Orin NX 16 GB, AGX Orin |
| **`GPU_SINGLE`** | Sí (&lt; 12 GB de VRAM) | 1-3 | `tiled_gpu` | GPU de gama básica, Jetson Nano, Orin Nano |
| **`CPU_PARALLEL`** | No | núcleos - 1 | `cpu_fallback` | Sistemas sin GPU NVIDIA |

### Tipos de canalización

* **`fused_gpu`** — Ruta de procesamiento completa de la GPU. Todas las operaciones de debayer, corrección e indexación se ejecutan en la GPU en una única pasada fusionada. Ofrece el mayor rendimiento, pero requiere más VRAM.
* **`tiled_gpu`** — Ruta de GPU eficiente en memoria. Procesa las imágenes en mosaicos para que quepan dentro de la memoria limitada de la GPU. Ofrece un rendimiento menor, pero funciona en dispositivos con limitaciones de memoria.
* **`cpu_fallback`** — Procesamiento solo en CPU mediante paralelismo multihilo. Se utiliza cuando no hay ninguna GPU NVIDIA disponible.***

## Comportamiento específico de la plataforma

| Plataforma | Estrategia | Trabajadores | Canalización | Notas |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (en serie) | Modo eficiente en memoria, procesa una imagen a la vez |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 3 | `fused_gpu` (concurrente) | Dispositivo periférico recomendado: procesamiento paralelo real de la GPU |
| **Jetson AGX Orin 64 GB** | `GPU_PARALLEL` | 4 | `fused_gpu` (concurrente) | Máximo rendimiento en el borde |
| **Ordenador de sobremesa con GPU de 8 GB** | `GPU_SINGLE` | 3 | `tiled_gpu` | Buen rendimiento de sobremesa con mosaicos eficientes en memoria |
| **Ordenador de sobremesa con GPU de 12 GB o más** | `GPU_PARALLEL` | 3-4 | `fused_gpu` | Rendimiento óptimo en el escritorio |
| **Sistema solo con CPU** | `CPU_PARALLEL` | núcleos - 1 | `cpu_fallback` | No requiere GPU, utiliza ThreadPool |

{% hint style="info" %}
**Memoria unificada de Jetson**: los dispositivos Jetson comparten la memoria de la GPU y la CPU. Un Jetson Orin NX de 16 GB muestra unos 15,3 GB de VRAM, pero se trata de la misma RAM física que utilizan el sistema operativo y los procesos de la CPU. Chloros tiene esto en cuenta al establecer los umbrales de asignación de memoria.
{% endhint %}

***

## Asignación dinámica de memoria de la GPU

Chloros utiliza un [flujo de procesamiento de 4 subprocesos](processing-pipeline.md):

* **Subproceso 1** (Detección): carga de imágenes, análisis de EXIF, detección de objetivos
* **Subproceso 2** (Calibración): cálculo de la calibración de reflectancia
* **Hilo 3** (Procesamiento): desentrelazado por GPU, corrección de viñeteado, cálculo de índices
* **Hilo 4** (Exportación): escritura de archivos, incrustación de metadatos

A medida que los hilos anteriores del canal completan su trabajo (por ejemplo, cuando se han detectado todas las imágenes), su asignación de memoria de la GPU se libera y **se redistribuye a los hilos activos restantes**. Esto significa que el hilo 3 (la etapa que requiere un uso intensivo de la GPU) obtiene progresivamente más memoria a medida que avanza el proceso, lo que mejora el rendimiento para el trabajo que requiere mayor capacidad de cálculo.

### Etapas de asignación

| Etapa | Hilos activos | Distribución de la memoria de la GPU |
| --- | --- | --- |
| **Inicial** | 1, 2, 3, 4 | Repartida entre todos los hilos |
| **Inicio-intermedio** | 2, 3, 4 | Memoria del hilo 1 redistribuida |
| **Intermedio-final** | 3, 4 | La memoria de los hilos 1 y 2 se destina a los hilos 3 y 4 |
| **Final** | 3 o 4 | Memoria máxima para el hilo restante |***

## Procesamiento con reconocimiento de texturas

El método de debayer con reconocimiento de texturas (solo Chloros+) utiliza una cantidad de memoria de GPU significativamente mayor que el método estándar debido al modelo de eliminación de ruido de IA/ML:

* Los sistemas con **&lt; 7 GB de VRAM** se ven obligados a utilizar un bucle de procesamiento síncrono para el modo con reconocimiento de texturas (una imagen a la vez)
* Los sistemas con **más de 7 GB de VRAM** pueden procesar con reconocimiento de texturas de forma simultánea, aunque con un número reducido de trabajadores en comparación con el modo estándar***

## Gestión térmica (Jetson)

Los dispositivos Jetson tienen limitaciones térmicas, especialmente en implementaciones en espacios cerrados o aéreos. Chloros supervisa las temperaturas de la GPU y la CPU y ajusta automáticamente el procesamiento:

| Temperatura | Respuesta |
| --- | --- |
| **&lt; 70 °C** | Funcionamiento normal — velocidad máxima |
| **70 °C** (Advertencia) | Reducir el tamaño del lote |
| **80 °C** (Crítico) | Limitación agresiva — menor concurrencia y número de trabajadores |
| **90 °C** (Apagado) | Detener por completo el procesamiento de la GPU |

La supervisión de la temperatura utiliza `tegrastats` en plataformas Jetson. En sistemas de escritorio con refrigeración adecuada, la limitación térmica rara vez se activa.

***

## Gestión de la presión de memoria

Chloros supervisa la presión de memoria del sistema durante el procesamiento:

* **Umbral de memoria**: una utilización del 85 % activa un comportamiento conservador
* **Reducción por falta de memoria**: si se produce un evento de falta de memoria, la asignación se reduce en un 25 % (multiplicador de 0,75x)
* **Recurso de reserva del pipeline**: En caso de presión de memoria grave, el pipeline recurre automáticamente de `fused_gpu` a `tiled_gpu`
* **Recomendaciones de intercambio**: En Jetson, Chloros le avisa si el espacio de intercambio es insuficiente para el tamaño de su conjunto de datos***

## Supervisión de la adaptación de computación

### Salida de estado de CLI

Cuando se inicia el procesamiento, CLI muestra el perfil de hardware detectado:

```

Chloros CLI 1.1.0
Platform: Linux aarch64 (Jetson Orin NX 16GB)
Strategy: GPU_PARALLEL | Workers: 3 | Pipeline: fused_gpu
CUDA: Available | GPU Memory: 15.3 GB (shared)
```

### Diagnóstico del sistema

Ejecute `chloros-cli selftest` para ver un perfil de hardware completo y verificar las capacidades de computación:

```bash
chloros-cli selftest
```

Esto comprueba la disponibilidad de CUDA, la memoria de la GPU, los modelos de eliminación de ruido y la conectividad del backend.

***

## Próximos pasos

* [Canal de procesamiento](processing-pipeline.md) — Comprensión de la arquitectura del canal de 4 subprocesos
* [Guía de NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Implementación y optimización específicas para Jetson
* [CLI : Línea de comandos](../CLI.md) — Referencia completa de CLI

# Adaptación dinámica del procesamiento

Chloros 1.2.0 utiliza la detección de hardware y la selección automática de estrategias de procesamiento. El motor de procesamiento se adapta a tu hardware —desde un Jetson Orin Nano hasta una estación de trabajo con varias GPU— sin necesidad de ninguna configuración manual.

***

## Cómo funciona

Cuando se inicia Chloros, realiza un perfil de tu sistema:

1. **Detecta el sistema operativo**: Windows o Linux
2. **Identifica los núcleos de la CPU y la RAM total**

3.**Detecta la presencia de una GPU** — Capacidad NVIDIA CUDA, VRAM, modelo
4. **Identifica el modelo de Jetson** (si procede) — a través de `/proc/device-tree/model`
5. **Comprueba los sensores térmicos** (Jetson) — para un procesamiento que tenga en cuenta la temperatura
6. **Selecciona la estrategia de cálculo** — en función de todo el hardware detectado
7. **Configura automáticamente el número de trabajadores, el tipo de canalización y la asignación de memoria**

El perfil detectado se almacena en caché para la sesión, tanto en memoria como en disco, de modo que las ejecuciones posteriores se inician más rápido:

| Plataforma | Perfil almacenado en caché |
| --- | --- |
| **Linux / Jetson** | `~/.config/chloros/system_config.json` (da prioridad a `XDG_CONFIG_HOME`) |
| **Windows** | `%LOCALAPPDATA%\Chloros\config\system_config.json` |

Elimina ese archivo para forzar una nueva detección; esto resulta útil tras añadir una GPU o más RAM. Chloros también vuelve a detectar automáticamente cuando la caché ha sido escrita por una versión anterior incompatible.

***

## Estrategias de cálculo

Chloros selecciona una de las tres estrategias de cálculo en función de tu hardware:

| Estrategia | Se selecciona cuando | Trabajadores | Ejecutor | Canalización |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`**| GPU CUDA que indica**más de 12 GB de VRAM**(en memoria unificada de Jetson; también requiere más de 12 GB de RAM compartida en total) | `min(4, VRAM ÷ 4GB)`, mínimo 2 —**limitado a 2 en Jetson** | `ProcessPoolExecutor` (generación) | `fused_gpu` |
| **`GPU_SINGLE`**| GPU CUDA con**entre 2 y 12 GB de VRAM**| 3 (solapamiento de E/S; acceso a la GPU serializado mediante un semáforo).**1 (secuencial) en Jetson con menos de 12 GB de RAM** | `ProcessPoolExecutor` (generación); secuencial en el mismo proceso en Jetson con poca RAM | `fused_gpu` / `tiled_gpu` |
| **`CPU_PARALLEL`** | Sin GPU CUDA o con menos de 2 GB de VRAM | `max(2, physical cores − 1)` | `ThreadPoolExecutor` | `cpu_fallback` |

Ejemplos prácticos de la fórmula de trabajadores de `GPU_PARALLEL`: 12 GB de VRAM → 3 trabajadores, 16 GB o más → 4 trabajadores, cualquier Jetson → 2 trabajadores.

El paralelismo se implementa con el `concurrent.futures` estándar de Python: Las estrategias de GPU utilizan un `ProcessPoolExecutor` con el método de inicio **spawn** (cada trabajador es un proceso independiente con su propio contexto CUDA; `fork` copiaría un estado CUDA ya inicializado y corrompería los procesos hijos), mientras que la estrategia de CPU utiliza un `ThreadPoolExecutor`. Chloros no utiliza ningún marco de trabajo distribuido de terceros (como Ray).

### Tipos de canalización

* **`fused_gpu`** — Ruta de procesamiento íntegramente en GPU. Las operaciones de debayering, corrección e indexación se ejecutan en la GPU en una única pasada fusionada. Ofrece el mayor rendimiento, pero requiere la mayor cantidad de VRAM.
* **`tiled_gpu`** — Ruta de GPU con eficiencia de memoria. Procesa las imágenes en mosaicos para que quepan dentro de la memoria limitada de la GPU. Ofrece un rendimiento menor, pero funciona en dispositivos con limitaciones de memoria.
* **`cpu_fallback`** — Procesamiento exclusivamente en la CPU mediante paralelismo multihilo. Se utiliza cuando no hay ninguna GPU NVIDIA disponible y como opción de reserva de último recurso cuando fallan ambas rutas de la GPU.

La cadena de alternativas en tiempo de ejecución es siempre `fused_gpu` → `tiled_gpu` → `cpu_fallback`.

***

## Anulación manual de la estrategia

Establece la variable de entorno `CHLOROS_STRATEGY` para forzar una estrategia específica: una opción de escape avanzada para cuando la detección automática elija algo inadecuado para tu situación (por ejemplo, mantener la GPU libre para otras tareas):

```bash
# Valid values: CPU_PARALLEL, GPU_SINGLE, GPU_PARALLEL
CHLOROS_STRATEGY=CPU_PARALLEL chloros-cli process ~/datasets/flight001
```

La variable se compara sin distinguir entre mayúsculas y minúsculas; cualquier valor que no sea uno de los tres nombres se ignora y la detección automática continúa con normalidad. En caso de anulación, Chloros sigue seleccionando el número de trabajadores por ti:

| Anulación | Número de trabajadores utilizado |
| --- | --- |
| `CPU_PARALLEL` | `max(2, physical cores − 1)` |
| `GPU_SINGLE` | 3 |
| `GPU_PARALLEL` | `min(4, physical cores)` |

Es preferible configurarlo por comando en lugar de de forma permanente, para que las ejecuciones normales sigan adaptándose automáticamente.

***

## Comportamiento específico de la plataforma

| Plataforma | Estrategia | Trabajadores | Canalización | Notas |
| --- | --- | --- | --- | --- |
| **Jetson Orin Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (secuencial) | Modo de eficiencia de memoria, una imagen cada vez |
| **Jetson Orin NX 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (secuencial) | La RAM compartida inferior a 12 GB obliga a un procesamiento secuencial |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (concurrente) | Dispositivo periférico recomendado: Jetson limitado a 2 trabajadores |
| **Jetson AGX Orin 32-64 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (concurrente) | Máximo rendimiento en el borde (Jetson también limitado a 2 trabajadores) |
| **Ordenador de sobremesa con GPU de 8 GB** | `GPU_SINGLE` | 3 | `fused_gpu` / `tiled_gpu` | 3 trabajadores superponen las E/S mientras un semáforo serializa el acceso a la GPU |
| **Ordenador de sobremesa con GPU de 12 GB o más** | `GPU_PARALLEL` | 3-4 | `fused_gpu` (concurrente) | Rendimiento óptimo en ordenador de sobremesa: 12 GB → 3 trabajadores, 16 GB o más → 4 |
| **Sistema solo con CPU** | `CPU_PARALLEL` | núcleos físicos − 1 (mín. 2) | `cpu_fallback` | No se requiere GPU, utiliza un grupo de subprocesos |

{% hint style="info" %}
**Memoria unificada de Jetson**: los dispositivos Jetson comparten la memoria de la GPU y la CPU. Un Jetson Orin NX de 16 GB muestra unos 15,3 GB de VRAM, pero se trata de la misma memoria RAM física que utilizan el sistema operativo y los procesos de la CPU. Por eso, los Jetson de 16 GB+ cumplen los requisitos para `GPU_PARALLEL` al igual que una GPU de sobremesa de 12 GB+, aunque tienen un límite de 2 trabajadores: la GPU, los procesos de trabajo y sus contextos CUDA por trabajador recurren todos al mismo fondo compartido.
{% endhint %}

### Presupuesto de la GPU según la VRAM (GPU discretas)

En hosts x86_64 con una GPU NVIDIA discreta, la VRAM detectada también determina la cantidad de recursos de procesamiento que puede reclamar la tarjeta y el tamaño máximo que pueden alcanzar los lotes:

| VRAM detectada | Límite máximo del presupuesto de la GPU | Multiplicador del tamaño del lote |
| --- | --- | --- |
| **8 GB+** | 90 % | ×2,0 |
| **6-8 GB** | 85 % | ×1,75 |
| **3,5-6 GB** | 80 % | ×1,5 |
| **2-3,5 GB** | 75 % | ×1,25 |
| **Menos de 2 GB** | 70 % | ×1,0 |

Las GPU discretas reservan solo 0,5 GB para el sistema, ya que no comparten la RAM del sistema. Los perfiles de Jetson reservan una cantidad mucho mayor y tienen un límite inferior; consulta la [Guía de NVIDIA Jetson](../linux/nvidia-jetson-guide.md#per-model-gpu-budget).

***

## Asignación dinámica de memoria de la GPU

Chloros utiliza un [flujo de procesamiento de 4 subprocesos](processing-pipeline.md):

* **Hilo 1** (Detección): carga de imágenes, análisis de EXIF, detección de objetivos
* **Hilo 2** (Calibración): cálculo de la calibración de reflectancia
* **Hilo 3** (Procesamiento): debayer de la GPU, corrección de viñeteado, cálculo del índice
* **Hilo 4** (Exportación): escritura de archivos e incrustación de metadatos

Los hilos 1, 2 y 4 consumen pocos recursos de la GPU; el hilo 3 es el que más recursos consume. A medida que finalizan los hilos anteriores del proceso, su presupuesto de GPU se **redistribuye entre los hilos activos restantes**, por lo que el hilo 3 obtiene progresivamente más memoria a medida que avanza la ejecución.

### Etapas de asignación

| Etapa | Hilos activos | Distribución de memoria de la GPU |
| --- | --- | --- |
| **Inicial** | 1, 2, 3, 4 | Repartida entre todos los hilos, la mayor parte para el hilo 3 |
| **Inicial-intermedia** | 2, 3, 4 | Se redistribuye la parte del hilo 1 |
| **Intermedia-Final** | 3, 4 | Las partes de los hilos 1 y 2 se asignan a los hilos 3 y 4 |
| **Final** | 3 o 4 | El último hilo activo recibe su asignación máxima |

Hay dos reglas que rigen estas cifras:

* A un hilo que sea el **único** activo se le concede la asignación máxima de su perfil.
* Cuando hay más de una tarea *pesada* de la GPU activa, la asignación base de cada tarea pesada se divide entre ellas (sin bajar nunca del mínimo configurado).

El valor que se utiliza realmente en tiempo de ejecución es el **más bajo** entre la asignación del perfil de la plataforma y la recomendación en tiempo real del monitor de memoria de la GPU, por lo que una tarjeta ocupada siempre prevalece sobre un perfil optimista.***

## Procesamiento sensible a las texturas

El debayer con reconocimiento de texturas (**solo Chloros+** — `--debayer texture-aware`) ejecuta un modelo de eliminación de ruido basado en IA/ML que necesita aproximadamente 1,75 GB de VRAM en FP16 por copia, por lo que utiliza mucha más memoria de la GPU que el método estándar:

* Los sistemas con **menos de 7 GB de VRAM**procesan «Texture Aware» en un**bucle sincrónico, una imagen cada vez** — no caben varias copias del modelo, y un grupo de trabajadores solo añadiría conflictos
* Los sistemas con **más de 7 GB de VRAM** pueden procesar Texture Aware de forma concurrente, aunque con un número reducido de trabajadores en comparación con el método estándar
* En **Jetson**, Texture Aware siempre se asigna a un único trabajador y, en los modelos de bajo consumo (Nano, Orin Nano), también aplica automáticamente un límite de frecuencia de la GPU; consulta la [Guía de NVIDIA Jetson](../linux/nvidia-jetson-guide.md#gpu-frequency-cap-for-texture-aware-on-nano-and-orin-nano)***

## Gestión térmica (Jetson)

Los dispositivos Jetson tienen limitaciones térmicas, especialmente en implementaciones en espacios cerrados o a bordo de aeronaves. Chloros supervisa los sensores de temperatura integrados en el Jetson y ajusta automáticamente el tamaño de los lotes:

| Temperatura | Respuesta |
| --- | --- |
| **&lt; 70 °C** | Funcionamiento normal — velocidad máxima |
| **70 °C** (Advertencia) | El tamaño de los lotes se reduce progresivamente (del 100 % al 50 % entre 70 °C y 80 °C) |
| **80 °C** (Crítico) | Limitación agresiva (del 50 % al 0 % entre 80 °C y 90 °C) |
| **90 °C** (Apagado) | Se detiene por completo el procesamiento de la GPU |

En sistemas de sobremesa con una refrigeración adecuada, la limitación térmica rara vez se activa.

***

## Gestión de la presión de memoria

Chloros supervisa continuamente la memoria de la GPU durante el procesamiento y reacciona en tres niveles.

**Tamaño del lote.** Un lote comienza con 8 imágenes multiplicadas por el multiplicador de la plataforma que figura en las tablas anteriores. Chloros comprueba a continuación la VRAM libre, reserva el 20 % de la misma para la sobrecarga propia de PyTorch y asume aproximadamente 100 MB de memoria de la GPU por cada imagen de 12 MP; el lote será el menor de los dos, el límite derivado de la memoria o la base de la plataforma. Nunca desciende por debajo de 1.**Reducción preventiva.**Por encima del**85 % de utilización de VRAM**, los tamaños de los lotes se reducen antes de que se produzca cualquier fallo.**Recorte de la asignación por hilo.** A medida que aumenta la utilización en tiempo real, se reduce el presupuesto de GPU de cada hilo: ×0,75 por encima del 80 % de utilización, ×0,5 por encima del 90 %. Los umbrales del monitor son el 70 % (conservador), el 85 % (límite de funcionamiento normal) y el 95 % (riesgo de OOM).**Retroceso y recuperación ante OOM.** Si, a pesar de todo, se produce un evento de falta de memoria:

* el tamaño del lote se **reduce a la mitad**, y se vuelve a reducir a la mitad con cada «OOM» consecutivo; cada lote posterior que se ejecute con éxito retrocede esa penalización un paso
* las asignaciones de los hilos activos se reducen al 70 % de su valor actual y el asignador cambia a su estrategia conservadora, relajándose de nuevo tras una serie de asignaciones exitosas
* bajo una presión extrema, la canalización recurre a `fused_gpu` en lugar de `tiled_gpu`, y a `cpu_fallback` como último recurso

**RAM del host (Jetson).** Antes del procesamiento, CLI calcula el pico de memoria del host a partir del recuento de imágenes y del modo de debayer, y avisa si es probable que la RAM más el espacio de intercambio respaldado por archivos sea insuficiente, mostrando los comandos exactos para añadir espacio de intercambio; consulta la [Guía de NVIDIA Jetson](../linux/nvidia-jetson-guide.md#swap-warning-and-recommendations).***

## Supervisión de la adaptación computacional

### Diagnósticos del sistema

`chloros-cli selftest` es la forma más rápida de confirmar lo que detecta la capa computacional:

```bash
chloros-cli selftest
```

Sus 7 comprobaciones abarcan la versión, la disponibilidad de puertos, el inicio del backend, `/api/test`, la información del sistema, la presencia del modelo de eliminación de ruido y la disponibilidad de CUDA y del eliminador de ruido. La comprobación 5 muestra directamente la línea de hardware:

```
      GPU: NVIDIA RTX A4000, CUDA: True, PyTorch: 2.7.0
```

La comprobación 7 muestra `CUDA: <bool>, Denoiser: <bool>`; ambas deben ser verdaderas para que Texture Aware sea utilizable.

### Registros del backend

La estrategia y el número de trabajadores se eligen dentro del backend al inicio de cada ejecución; no hay ningún mensaje de CLI que los anuncie. Cuando algo se comporta de forma inesperada (una ruta de GPU que recurre a un plan alternativo, un error OOM, un denoiser que no se carga), el registro del backend de esa sesión es donde aparece:

| Plataforma | Ubicación del registro |
| --- | --- |
| **Linux / Jetson** | `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` (un archivo por ejecución) |
| **Linux, CLI-backend iniciado** | también `~/.chloros/backend.log` |
| **Windows** | `%LOCALAPPDATA%\Chloros\logs\` |

### Progreso en tiempo real

Durante una ejecución, el CLI muestra el progreso en tiempo real por hilo (Detectando, Analizando, Procesando, Exportando) transmitido a través de Server-Sent Events: la forma más práctica de saber si el hilo 3 es el cuello de botella. Véase [Cadena de procesamiento](processing-pipeline.md).

***

## Próximos pasos

* [Canal de procesamiento](processing-pipeline.md): comprensión de la arquitectura del canal de 4 subprocesos
* [Guía de NVIDIA Jetson](../linux/nvidia-jetson-guide.md): implementación y optimización específicas para Jetson
* [CLI: Línea de comandos](../CLI.md) — La guía de CLI
* [Referencia de CLI](../reference/cli-reference.md) — Lista exhaustiva de comandos para la versión 1.2.0

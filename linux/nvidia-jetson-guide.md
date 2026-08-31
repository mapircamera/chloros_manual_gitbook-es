# Guía de NVIDIA Jetson

Chloros en NVIDIA Jetson permite el procesamiento de imágenes multiespectrales en el borde: sobre el terreno, en vehículos aéreos no tripulados (UAV) y en instalaciones remotas. Chloros 1.2.0 detecta tu modelo de Jetson al iniciarse y optimiza su estrategia de procesamiento en función del hardware que encuentra. **No es necesario realizar ningún ajuste manual.**

***

## Modelos de Jetson compatibles

| Modelo                | RAM            | Estrategia de procesamiento                                     | Uso recomendado                                          |
| -------------------- | -------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32-64 GB compartidos | `GPU_PARALLEL` (2 trabajadores)                              | Máximo rendimiento, grandes conjuntos de datos                      |
| **Jetson Orin NX**   | 8-16 GB compartidos  | `GPU_PARALLEL` (2 trabajadores, 16 GB) / `GPU_SINGLE` (8 GB)   | Recomendación principal para despliegues aéreos y sobre el terreno |
| **Jetson Orin Nano** | 8 GB compartidos     | `GPU_SINGLE` (1 trabajador, secuencial)                     | Computación en el borde de nivel básico                                 |

{% hint style="info" %}
El paquete Linux para arm64 requiere **JetPack 6**, que está disponible en la familia Jetson Orin. Los modelos más antiguos (Nano, TX2, Xavier NX) no pueden ejecutar JetPack 6 y no son compatibles con el paquete actual.
{% endhint %}

***

## Requisitos

* **JetPack 6.x** (se recomienda la última versión)
* **NVIDIA CUDA** (incluido con JetPack)
* **Plan de pago Chloros+** — Nivel «Copper» o superior (requerido para todo el acceso a CLI/SDK; se aplica desde el servidor)

## Instalación

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f

# Verify installation
chloros-cli --version    # prints "Chloros CLI 1.2.0"

# Install Python SDK (optional) — the bundled wheel always matches this build
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl

# Run system diagnostics
chloros-cli selftest
```

Para obtener información general sobre la instalación de Linux, la ubicación de los archivos y la resolución de problemas, consulta [Instalación de Linux](linux-installation.md).

{% hint style="info" %}
**Coloca el directorio de extracción en un almacenamiento rápido.** Los binarios compilados se descomprimen automáticamente en un directorio temporal cada vez que se inicia el programa, lo que resulta extremadamente lento si se hace desde una tarjeta SD. Chloros utiliza `/mnt/ssd/tmp` automáticamente si existe; de lo contrario, configura `TMPDIR` con una ruta de tu NVMe (`export TMPDIR=/mnt/nvme/tmp`).
{% endhint %}

***

## Adaptación dinámica de la computación en Jetson

### Cómo funciona

Al iniciarse, Chloros analiza su sistema:

1. **Detecta el modelo de Jetson** a través de `/proc/device-tree/model`
2. **Lee la memoria compartida disponible de la GPU/CPU** (Jetson utiliza memoria unificada)
3. **Selecciona una estrategia de procesamiento** (`GPU_PARALLEL`, `GPU_SINGLE` o `CPU_PARALLEL`)
4. **Establece el número de trabajadores, el tipo de canalización y la asignación de memoria** automáticamente

La decisión viene determinada por la **memoria RAM compartida total**, no por el nombre del modelo:

* **Menos de 12 GB de RAM total**(todos los Jetson de 8 GB): `GPU_SINGLE` con**1 trabajador — procesamiento secuencial deliberado**. La memoria es insuficiente para trabajadores concurrentes, por lo que las imágenes se procesan de una en una. En los Jetson con**8 GB o menos**, el hilo 3 omite por completo el grupo de trabajadores y ejecuta su trabajo por imagen dentro del propio proceso.
* **12 GB o más**(Orin NX de 16 GB, AGX Orin): la memoria unificada cumple los requisitos para `GPU_PARALLEL`, pero el número de trabajadores**está limitado a 2 en Jetson** — la GPU, la RAM de los procesos de trabajo y sus contextos CUDA por trabajador recurren todos al mismo grupo compartido, por lo que un mayor número de trabajadores conlleva el riesgo de fallos por falta de memoria.

Puedes anular la elección automática con la variable de entorno `CHLOROS_STRATEGY`; consulta [Adaptación dinámica de la computación](../processing-architecture/dynamic-compute-adaptation.md#manual-strategy-override).

### Comportamiento por modelo

| Modelo de Jetson                | Estrategia       | Trabajadores | Ejecución                                      |
| --------------------------- | -------------- | ------- | ---------------------------------------------- |
| **Jetson Orin Nano 8 GB**    | `GPU_SINGLE`   | 1       | Bucle secuencial dentro del proceso (`tiled_gpu` en caso de presión de memoria) |
| **Jetson Orin NX 8 GB**      | `GPU_SINGLE`   | 1       | Bucle secuencial dentro del proceso                     |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 2       | Procesos de trabajo concurrentes, ruta `fused_gpu`  |
| **Jetson AGX Orin 32-64 GB** | `GPU_PARALLEL` | 2       | Procesos de trabajo concurrentes, ruta `fused_gpu`  |

La diferencia clave entre las plataformas es la **memoria**. Un Jetson de 8 GB debe procesar las imágenes de una en una utilizando un enfoque por mosaicos que optimiza el uso de la memoria cuando la carga es elevada, mientras que un Orin de 16 GB o más puede procesar dos imágenes simultáneamente a través de la GPU utilizando el canal de procesamiento fusionado de mayor rendimiento.

### Presupuesto de GPU por modelo

Cada modelo de Jetson también cuenta con un perfil de hardware que limita la cantidad de recursos que el procesamiento del pool compartido puede reclamar y escala los tamaños de los lotes:

| Modelo | Límite máximo del presupuesto de GPU | Multiplicador del tamaño del lote | Reservado para el sistema/pantalla |
| --- | --- | --- | --- |
| **Jetson Orin Nano** | 70 % | ×0,8 | 2,0 GB |
| **Jetson Orin NX** | 75 % | ×1,0 | 3,0 GB |
| **Jetson AGX Orin** | 80 % | ×1,5 | 4,0 GB |

La RAM detectada ajusta el perfil: si un Jetson indica **16 GB o más**, su multiplicador de lotes se incrementa en ×1,2. El tamaño de lote base antes de aplicar los multiplicadores es de 8 imágenes.

Para consultar la referencia completa sobre la adaptación de cálculo, véase [Adaptación dinámica del cálculo](../processing-architecture/dynamic-compute-adaptation.md).

***

## Límite de frecuencia de la GPU para Texture Aware en Nano y Orin Nano

El debayer de Texture Aware ejecuta inferencia de redes neuronales en la GPU, lo que puede provocar **avisos de sobrecorriente**en modelos Jetson de bajo consumo (clase de 10-15 W) a la velocidad máxima del reloj de la GPU. Antes del procesamiento de Texture Aware en un**Jetson Nano u Orin Nano**, Chloros comprueba la frecuencia máxima de la GPU y la limita a**510 MHz** (510000000) si en ese momento es superior:

* Si el comando CLI puede escribir en el nodo sysfs de la frecuencia de la GPU, el límite se **aplica automáticamente** y se muestra un mensaje de confirmación.
* Si no es así (se necesita acceso de root), el comando CLI muestra el comando exacto `sudo` para aplicar el límite manualmente, espera un momento para que puedas leerlo y, a continuación, continúa; el procesamiento sigue en marcha, pero pueden aparecer advertencias de sobrecorriente.

Para aplicar el límite tú mismo antes del procesamiento:

```bash
echo 510000000 | sudo tee /sys/devices/platform/bus@0/17000000.gpu/devfreq/17000000.gpu/max_freq
```

Los modelos de mayor potencia (Orin NX 25 W, AGX Orin 60 W) funcionan a plena velocidad de la GPU; no se aplica ningún límite. El «Standard debayer» nunca activa el límite en ningún modelo.

{% hint style="info" %}
**La opción «Texture Aware» en Jetson siempre procesa una imagen a la vez.** Cada trabajador necesitaría su propio contexto CUDA (~1 GB) además de su propia copia del modelo de eliminación de ruido, algo que la memoria unificada no puede permitirse; por lo tanto, en Jetson la ruta «Texture Aware» se asigna a un único trabajador con acceso a la GPU en serie. Es de esperar que «Texture Aware» sea notablemente más lento que «Standard» en cualquier dispositivo Jetson.
{% endhint %}

***

## Gestión térmica

Los dispositivos Jetson tienen un margen térmico limitado, especialmente en implementaciones en espacios cerrados o a bordo de aeronaves. Chloros supervisa la temperatura del SoC y limita automáticamente el tamaño de los lotes:

| Temperatura         | Acción                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**          | Funcionamiento normal: velocidad de procesamiento máxima          |
| **70 °C** (Advertencia) | El tamaño de los lotes se reduce progresivamente (del 100 % al 50 % entre 70 °C y 80 °C) |
| **80 °C** (Crítico) | Reducción agresiva (del 50 % al 0 % entre 80 °C y 90 °C) |
| **90 °C** (Apagado) | Se detiene por completo el procesamiento de la GPU — se requiere enfriamiento |

{% hint style="warning" %}
**Asegúrese de que haya una ventilación y una disipación térmica adecuadas** para un procesamiento continuado, especialmente en recintos de campo cerrados o en sistemas aéreos. La limitación térmica reducirá el rendimiento del procesamiento para proteger el hardware.
{% endhint %}

***

## Gestión de la memoria

Los dispositivos Jetson utilizan **memoria unificada**: la GPU y la CPU comparten la misma RAM física. La VRAM indicada (por ejemplo, ~15,3 GB en un Orin NX de 16 GB) no es memoria dedicada a la GPU, sino la misma RAM que utilizan el sistema operativo y el resto de procesos.

### Advertencia y recomendaciones sobre el intercambio de memoria

Antes de iniciar el procesamiento en Jetson, el CLI cuenta las imágenes RAW de la carpeta de entrada (`.tif`, `.tiff`, `.raw`, `.dng` — las vistas previas en JPG no se cuentan), calcula la memoria máxima que necesitará la ejecución y **avisa antes de comenzar** si es probable que la RAM y el espacio de intercambio resulten insuficientes. La advertencia lleva por título `LOW MEMORY WARNING - Jetson Detected`, muestra el recuento de imágenes, la RAM, el espacio de intercambio actual y el pico estimado, y a continuación proporciona los comandos exactos `fallocate` / `chmod` / `mkswap` / `swapon` adaptados al tamaño de tu proyecto (nunca inferiores a 8 GB). Se detiene unos segundos para que el mensaje no se pierda al desplazarse por el historial, y a continuación continúa el procesamiento.**Estimaciones de memoria utilizadas por la advertencia:**

| Modo de despixelado | Base | Por imagen |
| --- | --- | --- |
| Estándar | ~1,5 GB | ~10 MB |
| Con reconocimiento de texturas | ~2,5 GB (modelo + tiempo de ejecución de Python) | ~15 MB |

La advertencia se activa cuando el pico estimado supera la RAM más el espacio de intercambio menos un margen de seguridad de 1 GB, y solo cuenta el espacio de intercambio **respaldado por archivos**; una configuración basada únicamente en zram seguirá activando la advertencia.

Para añadir espacio de intercambio manualmente (ejemplo: 8 GB):



<!-- SCREENSHOT-NEEDED: Terminal on a Jetson Orin (SSH session) showing the full "LOW MEMORY WARNING - Jetson Detected" block printed by `chloros-cli process` on a large folder: the image count and debayer mode line, RAM / current swap / estimated peak figures, and the fallocate/chmod/mkswap/swapon command block it recommends -->

```bash
# Check current memory and swap
free -h

# Create a swap file
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```### Gestión de OOM (sin memoria)

Durante el procesamiento, Chloros supervisa la memoria de la GPU y reduce el rendimiento de forma controlada en lugar de bloquearse:

1. Cuando la utilización de la memoria de la GPU supera el **85 %**, los tamaños de los lotes se reducen de forma preventiva
2. Si aún así se produce un evento de falta de memoria, el tamaño del lote se **reduce a la mitad**, y se vuelve a reducir a la mitad con cada OOM consecutivo; cada lote posterior que se ejecute con éxito retrocede esa penalización un paso
3. Bajo una presión sostenida, la canalización recurre de `fused_gpu` a la ruta `tiled_gpu`, más eficiente en cuanto a memoria, y, como último recurso, al procesamiento por CPU

***

## Implementación en campo

### Consideraciones sobre el consumo energético

| Modelo Jetson     | Consumo típico | Notas                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Orin Nano | 7-15 W              | Conector cilíndrico de CC          |
| Jetson Orin NX   | 10-25 W             | Conector cilíndrico de CC          |
| Jetson AGX Orin  | 15-60 W             | USB-C PD o conector cilíndrico |

Planifica tu presupuesto energético para un procesamiento sostenido: el consumo máximo de potencia se produce durante el Hilo 3 (Procesamiento), que requiere un uso intensivo de la GPU.

### Recomendaciones de almacenamiento

* Se recomienda encarecidamente el uso de **SSD NVMe** para implementaciones en arm64
* Las tarjetas SD son demasiado lentas para el procesamiento; utilízalas únicamente como soporte de arranque
* Calcula entre 2 y 3 veces el tamaño de los datos de imagen sin procesar para los resultados procesados

### Funcionamiento sin pantalla a través de SSH

Chloros y CLI son ideales para implementaciones de Jetson sin pantalla:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format "TIFF (32-bit, Percent)"

# Monitor export progress
chloros-cli export-status
```

### Servidor de fondo siempre activo para la sincronización horaria de LATTICE / DAQ-E

Si tu Jetson controla cámaras LATTICE o sensores de luz DAQ-E en modo sin interfaz gráfica, activa el servicio systemd del backend para que el «grandmaster» PTP funcione de forma continua (la unidad está instalada, pero no está activada por defecto):

```bash
sudo systemctl enable --now chloros-backend.service
chloros-cli time-sync status
```

Consulte [Instalación de Linux](linux-installation.md#always-on-ptp-for-headless-hosts) para obtener más detalles, incluyendo cómo el paquete permite que los puertos PTP 319/320 sean asignables sin privilegios de root.

### Procesamiento automatizado con systemd

Crea un servicio de systemd para el procesamiento automatizado:

```ini
# /etc/systemd/system/chloros-process.service
[Unit]
Description=Chloros Automated Processing
After=network.target

[Service]
Type=oneshot
User=chloros
ExecStart=/usr/bin/chloros-cli process /data/incoming --output /data/processed
StandardOutput=append:/var/log/chloros-process.log
StandardError=append:/var/log/chloros-process.log

[Install]
WantedBy=multi-user.target
```

`chloros-cli process` devuelve un valor distinto de cero cuando una ejecución que ha solicitado productos no escribe ninguna imagen, por lo que el estado de fallo de systemd resulta útil para la supervisión.

Combínalo con un temporizador de systemd para el procesamiento programado:

```ini
# /etc/systemd/system/chloros-process.timer
[Unit]
Description=Run Chloros Processing Every Hour

[Timer]
OnCalendar=hourly
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
sudo systemctl enable chloros-process.timer
sudo systemctl start chloros-process.timer
```

***

## Ejemplos de flujos de trabajo

### Procesamiento básico en Jetson

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI NDRE GNDVI
```

### Python y SDK en Jetson

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (32-bit, Percent)",
        reflectance_calibration=True
    )
    chloros.process(mode="parallel")

print("Processing complete!")
```

### Procesamiento por lotes de varios vuelos

```bash
#!/bin/bash
# Process all flight datasets in a directory
for flight in /data/flights/*/; do
    name=$(basename "$flight")
    echo "Processing $name..."
    chloros-cli process "$flight" \
        --output "/data/processed/$name" \
        --format "TIFF (32-bit, Percent)" \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## Sistemas Jetson recomendados para uso sobre el terreno

Para implementaciones sobre el terreno y aéreas, ten en cuenta estas opciones de placas base Jetson Orin NX de 16 GB:

* **Aéreo/drones**: Sistemas con resistencia a las vibraciones (MIL-STD), ligeros (menos de 300 g) y con refrigeración pasiva
* **Campo resistente**: Carcasas impermeables IP67/IP69K con conectividad para cámaras GigE PoE
* **Mínimo/económico**: kits para desarrolladores con carcasas adicionales

Ponte en contacto con [el servicio de asistencia de MAPIR](https://www.mapir.camera/community/contact) para obtener recomendaciones específicas de hardware para tu escenario de despliegue.

***

## Próximos pasos

* [Instalación de Linux](linux-installation.md) — Detalles generales de la instalación de Linux
* [Adaptación dinámica de la capacidad de cálculo](../processing-architecture/dynamic-compute-adaptation.md) — Referencia completa sobre estrategias de cálculo
* [Cadena de procesamiento](../processing-architecture/processing-pipeline.md) — Descripción de la cadena de 4 subprocesos
* [CLI: Línea de comandos](../CLI.md) — La guía de CLI
* [API : Python SDK](../api-python-sdk.md) — La guía de SDK
* [Referencia de CLI](../reference/cli-reference.md) y [Referencia de SDK](../reference/sdk-reference.md) — Listados exhaustivos de comandos/API para la versión 1.2.0

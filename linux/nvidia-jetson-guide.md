# Guía de NVIDIA Jetson:

Chloros en NVIDIA Jetson permite el procesamiento de imágenes multiespectrales en el borde: sobre el terreno, en vehículos aéreos no tripulados (UAV) y en instalaciones remotas. Chloros 1.3.0 detecta tu modelo de Jetson al iniciarse y optimiza su estrategia de procesamiento en función del hardware que encuentra. **No es necesario realizar ningún ajuste manual.**
{% hint style="info" %}
**El paquete Jetson 1.3.0 realiza el procesamiento en la GPU del Jetson.** `chloros_1.3.0_arm64_jp6.deb` está diseñado para JetPack 6 y utiliza la versión de PyTorch para CUDA de JetPack 6, por lo que el procesamiento —incluido el algoritmo de despixelarización «Texture Aware»— se ejecuta en la GPU del Orin. El paquete no incluye CUDA en sí mismo: utiliza las bibliotecas CUDA y cuDNN que instala JetPack 6 (`sudo apt install nvidia-jetpack` en una imagen que carece de ellas). Cuando no se puede utilizar una GPU con CUDA, el procesamiento se ejecuta en la CPU y Texture Aware utiliza el «debayer» estándar.
{% endhint %}

***

## Modelos Jetson compatibles

| Modelo                | RAM            | Estrategia de procesamiento                                     | Uso recomendado                                          |
| -------------------- | -------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32-64 GB compartidos | `GPU_PARALLEL` (2 trabajadores)                              | Máximo rendimiento, conjuntos de datos grandes                      |
| **Jetson Orin NX**   | 8-16 GB compartidos  | `GPU_PARALLEL` (2 trabajadores, 16 GB) / `GPU_SINGLE` (8 GB)   | Recomendación principal para el despliegue sobre el terreno              |
| **Jetson Orin Nano** | 8 GB compartidos     | `GPU_SINGLE` (1 trabajador, secuencial)                     | Computación periférica de nivel básico                                 |

{% hint style="info" %}
El paquete de Linux para arm64 requiere **JetPack 6**, que está disponible en la familia Jetson Orin. Los modelos más antiguos (Nano, TX2, Xavier NX) no pueden ejecutar JetPack 6 y no son compatibles con el paquete actual.
{% endhint %}

***

## Requisitos

* **JetPack 6.x** (se recomienda la última versión)
* **NVIDIA CUDA y cuDNN** de JetPack 6 (`sudo apt install nvidia-jetpack` si tu imagen no los incluye)
* **Plan de pago Chloros+** — Nivel Copper o superior (requerido para todo acceso a CLI/SDK; se aplica del lado del servidor)

## Instalación

```bash
# Install the arm64 .deb package (JetPack 6) -- apt, not dpkg -i, so the
# Python SDK's requirements (python3-numpy, python3-requests) come too
sudo apt install ./chloros_1.3.0_arm64_jp6.deb

# Verify installation
chloros-cli --version    # prints "Chloros CLI 1.3.0"
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # the .deb installs the SDK

# Run system diagnostics
chloros-cli selftest
```

El `.deb` instala el SDK de Python para el sistema `python3`, que coincide con esta compilación. Solo es necesario instalar el archivo «wheel» incluido en un entorno virtual (virtualenv): `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` (totalmente sin conexión). Para obtener detalles generales sobre la instalación en Linux, la ubicación de los archivos y la resolución de problemas, consulta [Instalación en Linux](linux-installation.md).

{% hint style="info" %}
**Ya no es necesario descomprimir al iniciar.** A partir de la versión 1.3.0, el backend y la CLI se instalan sin descomprimir en `/usr/lib/chloros/`, por lo que ya no se extraen a un directorio temporal cada vez que se inician. Los archivos temporales siguen guardándose automáticamente en `/mnt/ssd/tmp` cuando existe esa carpeta; de lo contrario, puede configurar `TMPDIR` con una ruta en su NVMe (`export TMPDIR=/mnt/nvme/tmp`).
{% endhint %}

***

## Adaptación dinámica de la potencia de cálculo en Jetson

### Cómo funciona

Al iniciarse, Chloros crea un perfil de tu sistema:

1. **Detecta el modelo de Jetson** a través de `/proc/device-tree/model`
2. **Lee la memoria compartida disponible de la GPU y la CPU** (Jetson utiliza memoria unificada)
3. **Pregunta a PyTorch si puede utilizar CUDA**: si se tienen instalados CUDA y cuDNN de JetPack 6, puede hacerlo
4. **Selecciona automáticamente una estrategia de procesamiento** (`GPU_PARALLEL`, `GPU_SINGLE` o `CPU_PARALLEL`) y el número de trabajadores

La decisión viene determinada por la **memoria RAM compartida total**, no por el nombre del modelo:

* **Con menos de 12 GB de RAM total**(todos los Jetson de 8 GB):**1 trabajador — procesamiento secuencial deliberado**. La memoria es demasiado limitada para trabajadores concurrentes, por lo que las imágenes se procesan de una en una. En los Jetson con**8 GB o menos**, el hilo 3 omite por completo el grupo de trabajadores y ejecuta su trabajo por imagen dentro del propio proceso. El registro del backend denomina a esta estrategia `GPU_SINGLE`.
* **12 GB o más**(Orin NX de 16 GB, AGX Orin): la memoria unificada cumple los requisitos para `GPU_PARALLEL`, pero el número de trabajadores**está limitado a 2 en Jetson** — la GPU, la RAM de los procesos de trabajo y los contextos CUDA de cada uno de ellos recurren al mismo grupo compartido, por lo que un mayor número de procesos de trabajo conlleva el riesgo de fallos por falta de memoria.
* **Sin CUDA utilizable** (faltan las bibliotecas CUDA de JetPack): `CPU_PARALLEL` con 12 GB o más; una imagen cada vez a continuación.

Puedes anular la elección automática con la variable de entorno `CHLOROS_STRATEGY`; consulta [Adaptación dinámica de computación](../processing-architecture/dynamic-compute-adaptation.md#manual-strategy-override).

### Comportamiento por modelo

| Modelo Jetson                | Estrategia       | Trabajadores | Ejecución                                      |
| --------------------------- | -------------- | ------- | ---------------------------------------------- |
| **Jetson Orin Nano 8 GB**    | `GPU_SINGLE`   | 1       | Bucle secuencial dentro del proceso (`tiled_gpu` en caso de presión de memoria) |
| **Jetson Orin NX 8 GB**      | `GPU_SINGLE`   | 1       | Bucle secuencial dentro del proceso                     |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 2       | Procesos de trabajo concurrentes, ruta `fused_gpu`  |
| **Jetson AGX Orin 32-64 GB** | `GPU_PARALLEL` | 2       | Procesos de trabajo concurrentes, ruta `fused_gpu`  |

La diferencia clave entre las plataformas es la **memoria**. Un Jetson de 8 GB debe procesar las imágenes de una en una utilizando un enfoque por mosaicos que optimiza el uso de la memoria cuando la carga es elevada, mientras que un Orin de 16 GB o más puede procesar dos imágenes simultáneamente a través de la GPU utilizando el canal de procesamiento fusionado de mayor rendimiento.

### Presupuesto de GPU por modelo

Cada modelo de Jetson también cuenta con un perfil de hardware que limita la cantidad de memoria que el procesamiento del pool compartido puede reclamar y escala los tamaños de los lotes:

| Modelo | Límite máximo del presupuesto de GPU | Multiplicador del tamaño del lote | Reservado para el sistema/pantalla |
| --- | --- | --- | --- |
| **Jetson Orin Nano** | 70 % | ×0,8 | 2,0 GB |
| **Jetson Orin NX** | 75 % | ×1,0 | 3,0 GB |
| **Jetson AGX Orin** | 80 % | ×1,5 | 4,0 GB |

La RAM detectada ajusta el perfil: si un Jetson indica **16 GB o más**, su multiplicador de lotes se incrementa en ×1,2. El tamaño de lote base antes de aplicar los multiplicadores es de 8 imágenes.

Para consultar la referencia completa sobre la adaptación de cálculo, véase [Adaptación dinámica del cálculo](../processing-architecture/dynamic-compute-adaptation.md).

***

## Límite de frecuencia de la GPU para Texture Aware en Nano y Orin Nano

El algoritmo de debayer «Texture Aware» ejecuta inferencia de redes neuronales en la GPU, lo que puede desencadenar **avisos de sobrecorriente**en modelos Jetson de bajo consumo (clase de 10-15 W) a la velocidad máxima del reloj de la GPU. Al pasar `--debayer texture-aware` en un**Jetson Nano u Orin Nano**, la CLI comprueba la frecuencia máxima de la GPU y la limita a **510 MHz** (510000000) si en ese momento es superior:

* Si la CLI puede escribir en el nodo sysfs de la frecuencia de la GPU, el límite se **aplica automáticamente** y se muestra un mensaje de confirmación.
* Si no es así (se necesita acceso de root), la CLI muestra el comando exacto `sudo` para aplicar el límite manualmente, espera un momento para que puedas leerlo y, a continuación, continúa; el procesamiento sigue ejecutándose, pero puede mostrar advertencias de sobrecorriente.

Para aplicar el límite tú mismo antes del procesamiento:

```bash
echo 510000000 | sudo tee /sys/devices/platform/bus@0/17000000.gpu/devfreq/17000000.gpu/max_freq
```

Los modelos de mayor potencia (Orin NX 25 W, AGX Orin 60 W) funcionan a la velocidad máxima de la GPU; no se aplica ningún límite. El «Standard debayer» nunca activa el límite en ningún modelo.

{% hint style="info" %}
**La opción «Texture Aware» en Jetson siempre procesa una imagen cada vez.** Cada trabajador necesitaría su propio contexto CUDA (~1 GB) además de su propia copia del modelo de eliminación de ruido, algo que la memoria unificada no puede permitirse; por lo tanto, en Jetson la ruta de «Texture Aware» se asigna a un único trabajador con acceso a la GPU en serie. Es de esperar que «Texture Aware» sea notablemente más lento que «Standard» en cualquier Jetson. «Texture Aware» se aplica a las imágenes de Survey3: las capturas de «LATTICE» siempre utilizan el «debayer» estándar. Sin una GPU CUDA utilizable, la ejecución utiliza el modo «Standard», y el backend registra una línea `[DEBAYER] … needs a CUDA GPU …` explicando el motivo (mostrada junto con `chloros-cli --verbose`).
{% endhint %}

***

## Gestión térmica

Los dispositivos Jetson tienen un margen térmico limitado, especialmente en implementaciones en espacios cerrados o aéreos. Chloros supervisa la temperatura del SoC y limita automáticamente el tamaño de los lotes:

| Temperatura         | Acción                                            |
| ------------------- | ------------------------------------------------- |
| **< 70 °C**          | Funcionamiento normal: velocidad de procesamiento máxima          |
| **70 °C** (Advertencia) | El tamaño de los lotes se reduce progresivamente (del 100 % al 50 % entre 70 °C y 80 °C) |
| **80 °C** (crítico) | Reducción agresiva (del 50 % al 0 % entre 80 °C y 90 °C) |
| **90 °C** (apagado) | Detiene por completo el procesamiento de la GPU — se requiere enfriamiento |

{% hint style="warning" %}
**Asegúrese de que haya una ventilación y una disipación del calor adecuadas** para un procesamiento continuado, especialmente en recintos cerrados o en sistemas aéreos. La limitación térmica reducirá el rendimiento del procesamiento para proteger el hardware.
{% endhint %}

***

## Gestión de la memoria

Los dispositivos Jetson utilizan **memoria unificada**: la GPU y la CPU comparten la misma RAM física. La VRAM indicada (por ejemplo, ~15,3 GB en un Orin NX de 16 GB) no es memoria dedicada a la GPU, sino la misma RAM que utilizan el sistema operativo y el resto de procesos.

### Advertencia y recomendaciones sobre el intercambio

Antes de procesar en Jetson, la CLI cuenta las imágenes RAW de tu carpeta de entrada (`.tif`, `.tiff`, `.raw`, `.dng` — no se cuentan las vistas previas en JPG), calcula la memoria máxima que necesitará la ejecución y **avisa antes de comenzar** si es probable que la RAM más el espacio de intercambio sean insuficientes. La advertencia lleva por título `LOW MEMORY WARNING - Jetson Detected`, muestra el número de imágenes, la RAM, el espacio de intercambio actual y el pico estimado, y a continuación proporciona los comandos exactos `fallocate` / `chmod` / `mkswap` / `swapon` adaptados al tamaño de tu proyecto (nunca inferior a 8 GB). Se detiene unos segundos para que el mensaje no se pierda al desplazarse por el historial, y luego continúa el procesamiento.**Estimaciones de memoria utilizadas por la advertencia:**

| Modo de despixelado | Base | Por imagen |
| --- | --- | --- |
| Estándar | ~1,5 GB | ~10 MB |
| Con reconocimiento de texturas | ~2,5 GB (modelo + tiempo de ejecución de Python) | ~15 MB |

La advertencia se activa cuando el pico estimado supera la RAM + el espacio de intercambio menos un margen de seguridad de 1 GB, y solo cuenta el espacio de intercambio **respaldado por archivos**; una configuración basada únicamente en zram seguirá marcándose como error.

Para añadir espacio de intercambio manualmente (ejemplo: 8 GB):

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
```

<!-- SCREENSHOT-NEEDED: Terminal on a Jetson Orin (SSH session) showing the full "LOW MEMORY WARNING - Jetson Detected" block printed by `chloros-cli process` on a large folder: the image count and debayer mode line, RAM / current swap / estimated peak figures, and the fallocate/chmod/mkswap/swapon command block it recommends -->


### Gestión de OOM (falta de memoria)

Durante el procesamiento, Chloros supervisa la memoria de la GPU y reduce el rendimiento de forma controlada en lugar de bloquearse:

1. Cuando la utilización de la memoria de la GPU supera el **85 %**, los tamaños de los lotes se reducen de forma preventiva
2. Si aún así se produce un evento de falta de memoria, el tamaño del lote se **reduce a la mitad**, y se vuelve a reducir a la mitad con cada OOM consecutivo; cada lote posterior que se ejecute con éxito retrocede un paso en esa penalización
3. Bajo una presión sostenida, la cadena de procesamiento pasa de `fused_gpu` a la ruta `tiled_gpu`, más eficiente en cuanto a memoria, y, como último recurso, al procesamiento por CPU

Dado que la GPU comparte la RAM del Jetson, la RAM más el espacio de intercambio siguen limitando la ejecución; la advertencia sobre el espacio de intercambio mencionada anteriormente es la indicación a tener en cuenta.

***

## Implementación sobre el terreno

### Consideraciones sobre el consumo energético

| Modelo de Jetson     | Consumo energético típico | Notas                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Orin Nano | 7-15 W              | Conector cilíndrico de CC          |
| Jetson Orin NX   | 10-25 W             | Conector cilíndrico de CC          |
| Jetson AGX Orin  | 15-60 W             | USB-C PD o conector cilíndrico |

Planifica tu presupuesto energético para un procesamiento sostenido: el consumo máximo de energía se produce durante el Hilo 3 (Procesamiento), que requiere un uso intensivo de la GPU.

### Recomendaciones de almacenamiento

* Se recomienda encarecidamente el uso de **SSD NVMe** para implementaciones arm64
* Las tarjetas SD son demasiado lentas para el procesamiento; utilízalas únicamente como soporte de arranque
* Prevé entre 2 y 3 veces el tamaño de tus datos de imagen sin procesar para el resultado procesado

### Funcionamiento sin interfaz gráfica a través de SSH
La Chloros de

Chloros es ideal para implementaciones de Jetson sin interfaz gráfica:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format "TIFF (32-bit, Percent)"

# Monitor export progress
chloros-cli export-status
```

### Backend siempre activo para la sincronización horaria de LATTICE / DAQ-E

Si tu Jetson controla cámaras LATTICE o sensores de luz DAQ-E sin monitor, activa el servicio systemd del backend **como tu usuario** para que el gran maestro PTP funcione de forma continua y los scripts del SDK encuentren un backend en ejecución (la unidad está instalada, pero no está habilitada por defecto):

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
chloros-cli time-sync status
```

De este modo, el backend se ejecuta como si fueras tú: las capturas que escribe son tuyas y utiliza tu `chloros-cli login`. Consulta [Instalación en Linux](linux-installation.md#always-on-ptp-for-headless-hosts) para obtener más detalles, incluyendo el `chloros-backend.service` de root y cómo el paquete permite que los puertos PTP 319/320 sean asignables sin necesidad de root.

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
    --indices NDVI GNDVI OSAVI
```

### SDK Python para Jetson

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "GNDVI", "OSAVI"],
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
        --indices NDVI OSAVI
    echo "Completed $name"
done
```

***

## Sistemas Jetson recomendados para uso sobre el terreno

Para implementaciones sobre el terreno y aéreas, ten en cuenta estas opciones de placas base Jetson Orin NX de 16 GB:

* **Aéreas/drones**: Sistemas con resistencia a las vibraciones (MIL-STD), ligeros (menos de 300 g) y refrigeración pasiva
* **Entornos de campo exigentes**: carcasas impermeables IP67/IP69K con conectividad para cámaras GigE PoE
* **Mínimo/económico**: kits para desarrolladores con carcasas complementarias

Póngase en contacto con [el servicio de asistencia de MAPIR](https://www.mapir.camera/community/contact) para obtener recomendaciones específicas de hardware para su escenario de despliegue.

***

## Próximos pasos

* [Instalación de Linux](linux-installation.md) — Detalles generales sobre la instalación de Linux

* [Adaptación dinámica de la capacidad de cálculo](../processing-architecture/dynamic-compute-adaptation.md) — Referencia completa sobre estrategias de cálculo
* [Canal de procesamiento](../processing-architecture/processing-pipeline.md) — Descripción del canal de 4 subprocesos
* [CLI: línea de comandos](../CLI.md) — Guía de la CLI

* [API: Python SDK](../api-python-sdk.md) — Guía del SDK

* [Referencia de la CLI](../reference/cli-reference.md) y [Referencia del SDK](../reference/sdk-reference.md) — Listados exhaustivos de comandos y API para la versión 1.3.0

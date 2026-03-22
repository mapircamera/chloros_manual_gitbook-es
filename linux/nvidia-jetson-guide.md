# Guía de NVIDIA Jetson

Chloros en NVIDIA Jetson permite el procesamiento de imágenes multiespectrales en el borde: sobre el terreno, en vehículos aéreos no tripulados (UAV) y en instalaciones remotas. Chloros detecta automáticamente tu modelo de Jetson y optimiza su estrategia de procesamiento para tu hardware.

***

## Modelos de Jetson compatibles

| Modelo                | RAM            | Estrategia de procesamiento                                   | Uso recomendado                                          |
| -------------------- | -------------- | ----------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32-64 GB compartidos | `GPU_PARALLEL` (4 trabajadores)                            | Máximo rendimiento, conjuntos de datos grandes                      |
| **Jetson Orin NX**   | 8-16 GB compartidos  | `GPU_PARALLEL` (3 trabajadores, 16 GB) / `GPU_SINGLE` (8 GB) | Recomendación principal para despliegues aéreos y sobre el terreno |
| **Jetson Orin Nano** | 8 GB compartidos     | `GPU_SINGLE` (1 trabajador)                               | Computación periférica de nivel básico                                 |
| **Jetson Nano**      | 4-8 GB compartidos   | `GPU_SINGLE` (1 trabajador)                               | Nivel básico, con memoria limitada                          |

{% hint style="info" %}
**Es posible que los modelos Jetson heredados** (TX2, TX1, Xavier NX) no sean compatibles. El rendimiento variará en función de la memoria de GPU disponible y las capacidades de CUDA.
{% endhint %}

***

## Requisitos

* **JetPack 6.x** (se recomienda la última versión)
* **NVIDIA CUDA** (incluido con JetPack)
* **Licencia Chloros+** (necesaria para acceder a CLI/SDK)

## Instalación

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros-arm64-jp6.deb

# Verify installation
chloros-cli --version

# Install Python SDK (optional)
pip install chloros-sdk

# Run system diagnostics
chloros-cli selftest
```

Para obtener información general sobre la instalación de Linux, consulte [Instalación de Linux](linux-installation.md).

***

## Adaptación dinámica de computación en Jetson

Chloros detecta automáticamente su modelo Jetson y selecciona la estrategia de procesamiento óptima. **No se requiere ningún ajuste manual.**

### Cómo funciona

Al iniciarse, Chloros perfila su sistema:

1. **Detecta el modelo Jetson** a través de `/proc/device-tree/model`
2. **Lee la memoria GPU/compartida disponible**

3.**Selecciona una estrategia de procesamiento** (`GPU_PARALLEL`, `GPU_SINGLE` o `CPU_PARALLEL`)
4. **Establece el número de trabajadores, el tipo de canalización y la asignación de memoria** automáticamente

### Comportamiento por modelo

| Modelo Jetson                | Estrategia       | Trabajadores | Canalización                       | Concurrencia |
| --------------------------- | -------------- | ------- | ------------------------------ | ----------- |
| **Jetson Nano 8 GB**         | `GPU_SINGLE`   | 1       | `tiled_gpu` (eficiente en memoria) | Serializado  |
| **Jetson Orin Nano 8 GB**    | `GPU_SINGLE`   | 1       | `tiled_gpu`                    | Serializado  |
| **Jetson Orin NX 8 GB**      | `GPU_SINGLE`   | 2       | `tiled_gpu`                    | Serializado  |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 3       | `fused_gpu` (ruta completa de la GPU)    | Concurrente  |
| **Jetson AGX Orin 32-64 GB** | `GPU_PARALLEL` | 4       | `fused_gpu`                    | Concurrente  |

{% hint style="success" %}
**Jetson Orin NX 16 GB** es la opción ideal para la implementación en el borde: adopta la estrategia `GPU_PARALLEL` con 3 trabajadores simultáneos, lo que ofrece un procesamiento paralelo real de la GPU en un formato compacto.
{% endhint %}

La diferencia clave entre las plataformas es la **memoria**. Un Jetson Nano con 8 GB de memoria compartida debe procesar las imágenes de una en una utilizando un enfoque en mosaico eficiente en memoria, mientras que un Orin NX con 16 GB puede procesar 3 imágenes a través de la GPU simultáneamente utilizando el canal fusionado de mayor rendimiento.

Para consultar la referencia completa sobre la adaptación computacional, véase [Adaptación computacional dinámica](../processing-architecture/dynamic-compute-adaptation.md).

***

## Gestión térmica

Los dispositivos Jetson tienen un margen térmico limitado, especialmente en implementaciones en espacios cerrados o a bordo de aeronaves. Chloros incluye supervisión térmica automática y limitación de rendimiento:

| Temperatura         | Acción                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**          | Funcionamiento normal — velocidad de procesamiento completa          |
| **70 °C** (Advertencia)  | Reducir el tamaño del lote automáticamente                   |
| **80 °C** (Crítico) | Limitación agresiva — menor concurrencia         |
| **90 °C** (Apagado) | Detener por completo el procesamiento de la GPU — se requiere enfriamiento |

{% hint style="warning" %}
**Asegúrese de que haya una ventilación y disipación de calor adecuadas** para un procesamiento sostenido, especialmente en recintos de campo cerrados o sistemas aéreos. La limitación térmica reducirá el rendimiento del procesamiento para proteger el hardware.
{% endhint %}

***

## Gestión de la memoria

Los dispositivos Jetson utilizan **memoria unificada**: la GPU y la CPU comparten la misma RAM física. Esto significa que la VRAM indicada (por ejemplo, 15,3 GB en Orin NX 16 GB) no es memoria dedicada a la GPU, sino que se comparte con el sistema operativo y otros procesos.

### Recomendaciones sobre el espacio de intercambio

Para conjuntos de datos de gran tamaño o el procesamiento de desentrelazado con reconocimiento de texturas, Chloros puede recomendar crear espacio de intercambio:

```bash
# Check current memory and swap
free -h

# Create a swap file (example: 8GB)
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

**Estimaciones de memoria por imagen:**

* Desentrelazado estándar: ~10 MB por imagen
* Desentrelazado con reconocimiento de texturas: ~15 MB por imagen

Chloros calcula automáticamente la memoria necesaria en función del tamaño de su conjunto de datos y le avisa si se recomienda el espacio de intercambio.

### Plan de contingencia por falta de memoria (OOM)

Si se detecta una condición de falta de memoria durante el procesamiento:

1. Chloros reduce automáticamente el número de trabajadores de la GPU
2. Cambia del canal `fused_gpu` al canal `tiled_gpu` (más eficiente en cuanto a memoria)
3. Continúa el procesamiento con un rendimiento reducido en lugar de bloquearse

***

## Implementación en campo

### Consideraciones sobre la alimentación

| Modelo Jetson     | Consumo típico | Notas                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Nano      | 5-10 W              | USB-C o conector cilíndrico    |
| Jetson Orin Nano | 7-15 W              | Conector cilíndrico de CC          |
| Jetson Orin NX   | 10-25 W             | Conector cilíndrico de CC          |
| Jetson AGX Orin  | 15-60 W             | USB-C PD o conector cilíndrico |

Planifique su presupuesto de energía para un procesamiento sostenido: el consumo máximo de energía se produce durante el Hilo 3 (Procesamiento), que requiere un uso intensivo de la GPU.

### Recomendaciones de almacenamiento

* Se recomienda encarecidamente el uso de **SSD NVMe** para implementaciones arm64
* Las tarjetas SD son demasiado lentas para el procesamiento; utilícelas solo como medio de arranque
* Prevea un espacio de 2 a 3 veces el tamaño de los datos de imagen sin procesar para los resultados procesados

### Funcionamiento sin periféricos a través de SSH

Chloros CLI es ideal para implementaciones de Jetson sin periféricos:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format tiff-32

# Monitor export progress
chloros-cli export-status
```

### Procesamiento automatizado con systemd

Cree un servicio systemd para el procesamiento automatizado:

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

Combínelo con un temporizador de systemd para el procesamiento programado:

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

## Flujos de trabajo de ejemplo

### Procesamiento básico con Jetson

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI
```

### Python SDK en Jetson

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

### Procesamiento por lotes de múltiples vuelos

```bash
#!/bin/bash
# Process all flight datasets in a directory
for flight in /data/flights/*/; do
    name=$(basename "$flight")
    echo "Processing $name..."
    chloros-cli process "$flight" \
        --output "/data/processed/$name" \
        --format tiff-32 \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## Sistemas Jetson recomendados para uso sobre el terreno

Para implementaciones sobre el terreno y aéreas, considere estas opciones de placas base Jetson Orin NX de 16 GB:

* **Aéreo/drones**: Sistemas con resistencia a las vibraciones (MIL-STD), ligeros (menos de 300 g) y refrigeración pasiva
* **Campo resistente**: Carcasas impermeables IP67/IP69K con conectividad para cámaras GigE PoE
* **Mínimo/económico**: Kits de desarrollo con carcasas adicionales

Póngase en contacto con [el servicio de asistencia de MAPIR](https://www.mapir.camera/community/contact) para obtener recomendaciones específicas de hardware para su escenario de implementación.

***

## Próximos pasos

* [Linux Instalación](linux-installation.md) — Detalles generales de la instalación de Linux
* [Adaptación dinámica de la computación](../processing-architecture/dynamic-compute-adaptation.md) — Referencia completa de la estrategia de computación
* [Canal de procesamiento](../processing-architecture/processing-pipeline.md) — Descripción del canal de 4 subprocesos
* [CLI : Línea de comandos](../CLI.md) — Referencia completa de CLI
* [API : Python SDK](../api-python-sdk.md) — Referencia completa de SDK

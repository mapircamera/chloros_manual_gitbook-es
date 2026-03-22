# Instalación de Linux

Chloros se distribuye para Linux en forma de paquetes `.deb` que instalan CLI y el backend. Python y SDK se instalan por separado mediante pip.

***

## Linux amd64 (x86_64)

### Requisitos del sistema

| Requisito | Mínimo | Recomendado |
| --- | --- | --- |
| **Distribución** | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+ |
| **Procesador** | x86_64 (Intel/AMD) | Intel Core i7 o superior |
| **Memoria (RAM)** | 8 GB | 16 GB o más |
| **Tarjeta gráfica** | Ninguna (procesamiento por CPU) | GPU NVIDIA con 4 GB+ de VRAM |
| **Almacenamiento** | 2 GB de espacio libre | SSD con 10 GB+ de espacio libre |
| **Python** | Python 3.7+ (para SDK) | Python 3.10+ |

### Instalación

Descargue el paquete `.deb` e instálelo:

```bash
sudo dpkg -i chloros-amd64.deb
```

Verifique la instalación:

```bash
chloros-cli --version
```

***

## Linux arm64 (NVIDIA Jetson)

### Requisitos del sistema

| Requisito | Mínimo | Recomendado |
| --- | --- | --- |
| **Plataforma** | NVIDIA Jetson con JetPack 6 | Jetson Orin NX de 16 GB o AGX Orin |
| **JetPack** | JetPack 6.x | Última versión de JetPack 6 |
| **Memoria (RAM)** | 8 GB (GPU/CPU compartida) | 16 GB+ compartida |
| **Almacenamiento** | 2 GB de espacio libre | SSD NVMe con 10 GB+ libres |
| **Python** | Python 3.7+ (para SDK) | Python 3.10+ |

### Instalación

Descargue el paquete JetPack 6 `.deb` e instálelo:

```bash
sudo dpkg -i chloros-arm64-jp6.deb
```

Compruebe la instalación:

```bash
chloros-cli --version
```

Para obtener información detallada sobre la configuración de Jetson, incluida la gestión térmica y la implementación en campo, consulte la [Guía de NVIDIA Jetson](nvidia-jetson-guide.md).

***

## Python SDK Instalación (Todo Linux)

El Python SDK se instala por separado mediante pip y funciona tanto en amd64 como en arm64:

```bash
pip install chloros-sdk
```

Para incluir la compatibilidad opcional con la transmisión de progreso:

```bash
pip install chloros-sdk[progress]
```

Verifique el SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
El paquete `.deb` instala el backend Chloros y CLI. El Python SDK es un paquete pip independiente que se comunica con el backend a través de un HTTP API local.
{% endhint %}

***

## Directorios de configuración

Chloros en Linux sigue la [Especificación de directorios base de XDG](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html):

| Propósito | Linux Ruta | Windows Equivalente |
| --- | --- | --- |
| **Configuración** | `~/.config/chloros/` | `%APPDATA%\Chloros\` |
| **Datos / Proyectos** | `~/.local/share/chloros/` | `%LOCALAPPDATA%\Chloros\` |
| **Caché / Credenciales** | `~/.cache/chloros/` | `%APPDATA%\Chloros\cache\` |

## Ubicaciones del ejecutable del backend

El paquete `.deb` instala el backend en una ubicación estándar. Los paquetes CLI y SDK detectan automáticamente la ruta del backend:

| Método de instalación | Ruta del backend |
| --- | --- |
| Paquete `.deb` | `/usr/lib/chloros/chloros-backend` |
| Manual / personalizado | `/opt/mapir/chloros/backend/chloros-backend` |

Puede anular la ruta del backend con el indicador `--backend-exe` CLI o el parámetro del constructor `backend_exe` SDK.

***

## Configuración inicial

### 1. Active su licencia

Se requiere una licencia Chloros+ para acceder a CLI y SDK:

```bash
chloros-cli login your@email.com 'your-password'
```

### 2. Compruebe el estado de su licencia

```bash
chloros-cli status
```

### 3. Procese su primer conjunto de datos

```bash
chloros-cli process ~/datasets/flight001
```

### 4. Ejecute el diagnóstico del sistema

Compruebe que su sistema está configurado correctamente:

```bash
chloros-cli selftest
```

Esto ejecuta 7 comprobaciones de diagnóstico, incluyendo la versión, el inicio del backend, la conectividad API y la disponibilidad de CUDA/GPU.

***

## Ejemplos de scripts de Bash

### Procesar varios conjuntos de datos

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    chloros-cli process "$dataset" --format tiff-32
    echo "Done: $(basename "$dataset")"
done
```

### Procesar con ajustes personalizados

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

### Procesamiento automatizado con Cron

Añádalo a su crontab (`crontab -e`) para procesar nuevos conjuntos de datos automáticamente:

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Python SDK Ejemplo

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "NDRE"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Solución de problemas

### CLI no encontrado tras la instalación

Si no se encuentra `chloros-cli` tras instalar el paquete `.deb`:

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# If not in PATH, check the installation
dpkg -L chloros-amd64  # or chloros-arm64-jp6

# Reload your shell
source ~/.bashrc
```

### Permiso denegado

```bash
# Ensure the binary is executable
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### Error al iniciar el backend

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

### No se ha detectado CUDA

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

### Faltan bibliotecas compartidas

```bash
# Install common dependencies
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

***

## Actualización de Chloros en Linux

Utilice el comando de actualización integrado para buscar e instalar actualizaciones:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

***

## Próximos pasos

* [Guía de NVIDIA Jetson](nvidia-jetson-guide.md) — Optimización y despliegue específicos para Jetson
* [CLI : Línea de comandos](../CLI.md) — Referencia completa de comandos de CLI
* [API : Python SDK](../api-python-sdk.md) — Referencia completa de SDK
* [Adaptación dinámica de computación](../processing-architecture/dynamic-compute-adaptation.md) — Cómo se adapta Chloros a su hardware

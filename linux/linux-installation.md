# Instalación de Linux

Chloros se distribuye para Linux como paquetes `.deb` que instalan CLI y el servidor backend. El Python SDK es un paquete pip independiente (también incluido en el `.deb` como un archivo wheel con la misma versión).

Los nombres de los archivos de los paquetes incluyen la versión y la arquitectura: `chloros_1.2.0_amd64.deb` para x86_64 y `chloros_1.2.0_arm64_jp6.deb` para las compilaciones de JetPack 6 para Jetson. Sustituye el archivo que hayas descargado realmente en los comandos que aparecen a continuación.

***

## Linux amd64 (x86_64)

### Requisitos del sistema

| Requisito | Mínimo | Recomendado |
| --- | --- | --- |
| **Distribución** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **Procesador** | x86_64 (Intel/AMD) | Intel Core i7 o superior |
| **Memoria (RAM)** | 8 GB | 16 GB o más |
| **Tarjeta gráfica** | Ninguna (procesamiento por CPU) | GPU NVIDIA con 4 GB o más de VRAM (12 GB o más desbloquean `GPU_PARALLEL`; 7 GB o más mantienen «Texture Aware» desactivado en la ruta de imagen única) |
| **Almacenamiento** | 2 GB de espacio libre | SSD con 10 GB o más de espacio libre |
| **Python** | Python 3.7+ (para el SDK) | Python 3.10+ |

> **Ubuntu 20.04 y Debian 11 no son compatibles.** La lista de dependencias de `.deb` se
> deriva de aquello con lo que el backend Chloros se vincula realmente, y eso incluye
> `libc6 (>= 2.34)`. Tanto Focal como Bullseye incluyen glibc 2.31, por lo que `apt` rechaza la
> instalación directamente en lugar de permitir que falle más tarde en tiempo de ejecución.

### Instalación

```bash
sudo dpkg -i chloros_1.2.0_amd64.deb
sudo apt-get install -f    # pulls the declared dependencies (libibverbs1, libcap2-bin)
```

{% hint style="info" %}
`dpkg -i` no resuelve las dependencias. Si indica que faltan paquetes, `sudo apt-get install -f` (o `sudo apt --fix-broken install`) completa la instalación; este es el proceso normal, no un error.
{% endhint %}

Comprueba la instalación:



<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo dpkg -i chloros_1.2.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block naming the bundled wheel path under /usr/lib/chloros/sdk/, any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend.service" hint -->

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```***

## Linux arm64 (NVIDIA Jetson)

### Requisitos del sistema

| Requisito | Mínimo | Recomendado |
| --- | --- | --- |
| **Plataforma** | NVIDIA Jetson con JetPack 6 | Jetson Orin NX de 16 GB o AGX Orin |
| **JetPack** | JetPack 6.x | Última versión de JetPack 6 |
| **Memoria (RAM)** | 8 GB (compartida entre GPU y CPU) | 16 GB o más compartidos (12 GB o más es el umbral para los trabajadores de GPU en paralelo) |
| **Almacenamiento** | 2 GB de espacio libre | SSD NVMe con 10 GB o más de espacio libre |
| **Python** | Python 3.7+ (para el SDK) | Python 3.10+ |

### Instalación

```bash
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Mismo diseño que el `.deb` para amd64, con una compilación CUDA optimizada para Jetson Orin / Orin NX / Orin Nano. Para obtener información sobre el comportamiento de la memoria, la temperatura y la implementación en campo de Jetson, consulta la [Guía de NVIDIA Jetson](nvidia-jetson-guide.md).

***

## Instalación de Python y SDK (todos los Linux)

El SDK es un cliente puro Python HTTP para el backend, por lo que el mismo paquete funciona tanto en amd64 como en arm64. Dos fuentes:**Desde PyPI** — la versión estable publicada:

```bash
pip install chloros-sdk
```

**Desde el archivo wheel incluido** — se garantiza que es compatible con el backend que acabas de instalar (utilízalo cuando tu versión sea más reciente que la de PyPI):

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

{% hint style="warning" %}
**Las distribuciones PEP 668** (Ubuntu 23.10+, Debian 12+) no admiten instalaciones de pip en todo el sistema. Utiliza `pip install --user …`, un entorno virtual o `sudo pip install --break-system-packages …`. El instalador del paquete nunca instala automáticamente el SDK en el Python de tu sistema; esa decisión te corresponde a ti.
{% endhint %}

Extras opcionales:

| Extra | Comando | Añade |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` para la transmisión en directo del progreso |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` para transporte BLE (DAQ-M) |

Comprueba el SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
El `.deb` instala el Chloros, el CLI y el backend. El Python y el SDK se comunican con ese backend a través de una dirección IPv4 local HTTP y API (`http://127.0.0.1:5000`) y lo inicia automáticamente cuando es necesario. Utiliza siempre la dirección IPv4 literal en lugar de `localhost` — ya que `localhost` puede resolverse como `::1` y costar aproximadamente dos segundos por solicitud.
{% endhint %}

***

## Configuración inicial

### 1. Iniciar sesión

El acceso a CLI y SDK requiere un nivel de pago Chloros+ (**Copper** o superior), que se aplica del lado del servidor: un usuario que haya cerrado sesión obtiene `401 AUTH_REQUIRED`, y un usuario del nivel gratuito (Iron) obtiene `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

Las credenciales se almacenan en caché en `~/.chloros/user_session.json`.

{% hint style="warning" %}
**Debes volver a iniciar sesión después de cada instalación o actualización.** El script `prerm` del paquete borra deliberadamente `~/.chloros/user_session.json` y la licencia almacenada en caché para todos los usuarios del equipo, de modo que una nueva compilación siempre revalida la licencia en lugar de confiar en una caché obsoleta.
{% endhint %}

### 2. Comprueba el estado de tu licencia

```bash
chloros-cli status
```

`chloros-cli status` funciona en cualquier nivel (incluido el gratuito), por lo que siempre podrás ver por qué el acceso está disponible o no.

### 3. Ejecuta los diagnósticos del sistema

```bash
chloros-cli selftest
```

Se ejecutan siete comprobaciones en orden, y el comando devuelve un valor distinto de cero si alguna de ellas falla:

| # | Comprobación | Qué comprueba |
| --- | --- | --- |
| 1 | **Versión** | El CLI informa de su versión (`v1.2.0`). |
| 2 | **Puerto disponible** | El puerto 5000 está libre, *o* ya ha respondido un backend Chloros en buen estado (lo que cuenta como una prueba superada). |
| 3 | **Inicio del backend** | Se inicia el binario del backend. |
| 4 | **Prueba de API (`/api/test`)** | El backend responde con `status: ok`. |
| 5 | **Información del sistema** | Muestra `GPU: <name>, CUDA: <bool>, PyTorch: <version>` desde `/api/system-info`. |
| 6 | **Modelos de denoiser** | Encuentra los modelos `*.pth.enc` (en Linux: `/usr/lib/chloros/models`). |
| 7 | **CUDA + eliminador de ruido**| La función «Texture Aware» es realmente utilizable: requiere CUDA**y** al menos un archivo de modelo. |

La ejecución finaliza con `N/7 checks passed`, enumerando los posibles errores por nombre.

### 4. Procesa tu primer conjunto de datos

```bash
chloros-cli process ~/datasets/flight001
```

***

## Archivos y directorios

### Por usuario

Chloros guarda sus credenciales y la configuración de CLI en un único directorio multiplataforma, **`~/.chloros/`** (en Windows, `%USERPROFILE%\.chloros\`). En cambio, dos cachés específicos de Linux siguen las convenciones XDG; estos respetan `XDG_CONFIG_HOME` / `XDG_CACHE_HOME` cuando se establecen.

| Ruta | Finalidad |
| --- | --- |
| `~/.chloros/user_session.json` | Caché de sesión de inicio de sesión creada por `chloros-cli login` (se borra con cada instalación o actualización de un paquete) |
| `~/.chloros/working_directory.txt` | Anulación de la carpeta de proyecto predeterminada (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | Preferencia de idioma de CLI (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Configuración de idioma compartida con la interfaz gráfica de usuario de Windows: un valor de `language` tiene aquí prioridad sobre `cli_language.json` |
| `~/.chloros/update_cache.json` | Caché de una hora para la comprobación de actualizaciones al inicio de Linux/Jetson |
| `~/.chloros/backend.log` | Registro del backend cuando este fue iniciado por el CLI |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Paquetes de calibración LATTICE almacenados en caché por cámara, indexados por número de serie y hash del paquete |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Modificaciones opcionales del usuario para los perfiles de corrección de techo de DAQ |
| `~/.config/chloros/system_config.json` | Perfil de hardware almacenado en caché procedente de la Adaptación Dinámica de Cálculo (DCA): elimínalo para forzar una nueva detección de hardware |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Registros del servidor backend, un archivo por ejecución |
| `~/Chloros Projects/` | Carpeta de proyecto predeterminada cuando no se ha establecido ninguna configuración personalizada |

### A nivel de sistema

| Ruta | Finalidad |
| --- | --- |
| `/usr/bin/chloros-cli` | Script envolvente: configura `LD_LIBRARY_PATH` para las bibliotecas nativas incluidas y, a continuación, ejecuta el binario real |
| `/usr/bin/chloros-backend` | Script envolvente: lo mismo, más `CHLOROS_PRODUCTION=1` para que la puerta de autenticación del backend nunca pueda desactivarse de forma silenciosa |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | Los binarios compilados |
| `/usr/lib/chloros/arena_runtime/` | Entorno de ejecución Arena SDK requerido por las cámaras LATTICE |
| `/usr/lib/chloros/models/*.pth.enc` | Modelos de eliminación de ruido cifrados utilizados por el «debayer» sensible a texturas |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Python SDK: rueda correspondiente exactamente a esta compilación |
| `/usr/lib/chloros/exiftool` | exiftool incluido (enlazado simbólicamente a `/usr/local/bin/exiftool` solo si no existe un exiftool del sistema) |
| `/etc/chloros/update.conf` | Configuración del canal de actualización leída por `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Configura `net.ipv4.ip_unprivileged_port_start = 319` para que el backend pueda vincular los puertos PTP sin derechos de root |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Dirige el cargador dinámico hacia `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Concede al usuario que ha iniciado sesión acceso al puente serie USB DAQ-U (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend.service` | Activa el servicio de backend siempre activo (instalado, **no habilitado**) |
| `/usr/share/applications/chloros-cli.desktop` | Entrada del menú de la aplicación «Chloros CLI» que abre un terminal |

## Ubicación del ejecutable del backend

CLI y SDK detectan automáticamente el backend:

| Componente | Ruta |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Backend | `/usr/lib/chloros/chloros-backend` |

Sobrescribe la ruta del backend con la bandera `--backend-exe` CLI o el parámetro del constructor `backend_exe` SDK, y el puerto con `--port` (por defecto, `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` apunta a las familias de comandos **`lattice`**,**`project`**y**`daq pool-*`** a un backend remoto. Los comandos principales (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) lo ignoran deliberadamente y siempre se dirigen a `http://127.0.0.1:<port>`.
{% endhint %}

***

## Cámaras LATTICE y sensores de luz DAQ en Linux

Todas las familias de comandos de hardware en tiempo real funcionan en Linux (amd64 y Jetson):

* **`chloros-cli lattice`** — detecta, conecta, configura y captura datos de cámaras LATTICE y matrices sincronizadas. El `.deb` incluye el entorno de ejecución Arena SDK que necesitan y lo registra en el cargador dinámico.
* **`chloros-cli daq pool-*`** — conecta sensores de luz DAQ-U/M/E a través del grupo de backend, transmite espectros calibrados y graba archivos `.daq`. El CLI compilado incluye únicamente la familia `pool-*`: `pool-connect`, `pool-disconnect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`.
* **`chloros-cli project`**: ejecuta un proyecto guardado (sus cámaras, sensores y ajustes de procesamiento) sin interfaz gráfica.
* **`chloros-cli time-sync`** — inspecciona el «grandmaster» PTP en el que se ejecuta el backend Chloros para cámaras LATTICE y sensores DAQ-E.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

`--sensor-id` es un requisito previo para `pool-latest`, `pool-stream`, `pool-record` y `pool-set-cap`; `pool-list` muestra los ID que hay actualmente en el conjunto.

{% hint style="info" %}
**Se recomienda utilizar `--eth-host` para la primera conexión de DAQ-E en un equipo con múltiples direcciones.** La detección automática explora mDNS y puede pasar por alto la interfaz del sensor debido a una caché ARP vacía, por lo que la primera conexión `pool-connect --eth` tras el arranque puede fallar incluso cuando el sensor funciona perfectamente. Al indicar la IP o el nombre de host del sensor, se omite por completo el proceso de detección.
{% endhint %}

**Los permisos de serie de DAQ-U** se gestionan mediante la regla udev instalada (`uaccess` + grupo `dialout`). Si un sensor que ya estaba conectado sigue sin estar accesible, recarga las reglas o vuelve a conectarlo:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Consulte la [referencia de CLI](../CLI.md) para ver el conjunto completo de comandos.

### PTP siempre activo para hosts sin interfaz gráfica

En la primera instalación, se genera la unidad de systemd `chloros-backend.service`, pero **no está habilitada**. En un Jetson sin pantalla o en un servidor que deba mantener la sincronización de tiempo PTP en funcionamiento de forma continua para los sensores DAQ-E y las cámaras LATTICE, habilítala:

```bash
sudo systemctl enable --now chloros-backend.service
sudo systemctl status chloros-backend.service
```

Sin ella, el PTP solo se ejecuta mientras el backend Chloros está en funcionamiento, es decir, durante una sesión activa de CLI/SDK.

La unidad vincula el backend a los ajustes de entorno de `127.0.0.1:5000` (configuración del entorno `CHLOROS_HOST` / `CHLOROS_PORT` dentro de la unidad; se puede anular con `sudo systemctl edit chloros-backend.service`) y lo reinicia en caso de fallo tras 5 segundos.

**Cómo obtiene PTP sus puertos.** PTP utiliza los puertos UDP 319/320, ambos por debajo del límite inferior habitual de 1024 para los puertos privilegiados. El paquete `postinst` escribe `/etc/sysctl.d/60-chloros-ptp.conf` con `net.ipv4.ip_unprivileged_port_start = 319`, lo que permite al backend vincularlos mientras se ejecuta como tu usuario. También aplica `setcap cap_net_bind_service,cap_net_raw=+ep` al binario del backend como medida de seguridad adicional; por eso `libcap2-bin` es una dependencia declarada del paquete.***

## Ejemplos de scripts en Bash

{% hint style="info" %}
**Códigos de salida compatibles con scripts.**`chloros-cli process` devuelve el código `0` en caso de éxito y**un valor distinto de cero en caso de fallo —incluida una ejecución que solicitó productos de imagen pero no generó ninguno** (muestra el código `Processing finished but wrote no image products.` e indica el nombre de la carpeta del proyecto y las causas habituales). Las ejecuciones correctas informan del número de productos de imagen generados (`Image products written: N`). Códigos de salida: `0` (éxito), `1` (fallo), `2` (error de argumento), `130` (interrumpido).
{% endhint %}

### Procesar varios conjuntos de datos

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    if chloros-cli process "$dataset" --format "TIFF (32-bit, Percent)"; then
        echo "Done: $(basename "$dataset")"
    else
        echo "FAILED: $(basename "$dataset")" >&2
    fi
done
```

### Procesar con ajustes personalizados

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

Los valores válidos de `--format` son exactamente cuatro y contienen espacios; hay que ponerlos siempre entre comillas:

| Valor de `--format` | Carpeta de salida |
| --- | --- |
| `TIFF (16-bit)` *(por defecto)* | `tiff16` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` admite `standard` (por defecto) o `texture-aware` (Chloros+).

### Procesamiento automatizado con Cron

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

### CLI no se encuentra tras la instalación

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# List everything the package installed
dpkg -L chloros

# Reload your shell
source ~/.bashrc
```

### Permiso denegado

```bash
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### «setcap failed» durante la instalación

El `.deb` aplica el `cap_net_bind_service` al `/usr/lib/chloros/chloros-backend` para que pueda vincular los puertos PTP 319/320 sin derechos de root. Si faltaba `libcap2-bin` en el momento de la instalación, la llamada se omite. Instálalo y vuelve a instalar el paquete:

```bash
sudo apt install libcap2-bin
sudo apt reinstall chloros
```

### PTP no se inicia / No se puede vincular el puerto 319

Comprueba que se haya reducido el límite de puertos sin privilegios y, si no es así, vuelve a aplicarlo para el arranque actual:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

A continuación, comprueba el «grandmaster»:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### «No se han encontrado los controladores de la cámara LATTICE»

No se está resolviendo el tiempo de ejecución de Arena SDK. Comprueba que la configuración del cargador que escribe el paquete esté presente y actualizada:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
sudo ldconfig
ls /usr/lib/chloros/arena_runtime | head
```

### Fallo al iniciar el backend

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

Los registros del backend correspondientes al fallo de inicio se encuentran en `~/.cache/chloros/logs/`.

### No se ha detectado CUDA

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

`chloros-cli selftest` informa de lo mismo en una sola línea: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Faltan bibliotecas compartidas

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

### Arranque lento en sistemas con tarjeta SD

Los binarios compilados se extraen a un directorio temporal cada vez que se inician. Si existe `/mnt/ssd/tmp`, Chloros lo utiliza automáticamente; de lo contrario, configura `TMPDIR` en un sistema de archivos rápido:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Actualización de Chloros en Linux

El comando `update` solo está disponible en Linux/Jetson. Comprueba la versión publicada en el canal de actualización configurado en `/etc/chloros/update.conf` y ofrece descargar e instalar la versión correspondiente `.deb`:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

En Linux/Jetson, CLI también realiza una comprobación de actualizaciones sin bloqueo en cada inicio (el resultado se almacena en caché durante una hora en `~/.chloros/update_cache.json`) y muestra el mensaje `Update available: vX.Y.Z` cuando existe una versión más reciente. Tu configuración y tus proyectos se conservan tras la actualización; tendrás que volver a iniciar sesión después.

## Desinstalación

```bash
sudo apt remove chloros
```

La desinstalación detiene `chloros-backend.service`, restablece el límite mínimo predeterminado de puertos sin privilegios (1024), elimina el enlace simbólico de exiftool incluido y la configuración del cargador de Arena, y borra las credenciales almacenadas en caché. Tus proyectos y los archivos de datos de `~/.chloros/` no se ven afectados.

***

## Próximos pasos

* [Guía de NVIDIA Jetson](nvidia-jetson-guide.md): optimización e implementación específicas para Jetson
* [CLI : Línea de comandos](../CLI.md) — la guía de CLI
* [API : Python SDK](../api-python-sdk.md) — la guía SDK
* [Referencia de CLI](../reference/cli-reference.md) y [Referencia de SDK](../reference/sdk-reference.md) — Listas exhaustivas de comandos/API para la versión 1.2.0
* [Adaptación dinámica de la computación](../processing-architecture/dynamic-compute-adaptation.md): cómo se adapta Chloros a tu hardware

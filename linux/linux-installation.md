# Instalación en Linux

Chloros se distribuye para Linux en forma de paquetes `.deb` que instalan la CLI, el servidor backend y el SDK de Python (para el sistema `python3`, además de un archivo «wheel» compatible con la versión para entornos virtuales).

Los nombres de los archivos de los paquetes incluyen la versión y la arquitectura: `chloros_1.3.0_amd64.deb` para x86_64, y `chloros_1.3.0_arm64_jp6.deb` para arm64 (NVIDIA Jetson, JetPack 6). Sustituye el archivo que hayas descargado realmente en los comandos que aparecen a continuación.

{% hint style="info" %}
**Novedad en la versión 1.3.0: los programas se distribuyen sin empaquetar.** El backend y la CLI se instalan como carpetas, `/usr/lib/chloros/backend/` y `/usr/lib/chloros/cli/`, en lugar de como archivos autoextraíbles individuales. Ya no se descomprimen automáticamente en una carpeta temporal cada vez que se inician, por lo que se inician mucho más rápido (en x86_64, el backend solía tardar unos 100 segundos en descomprimirse cada vez que se iniciaba) y ya no necesitan varios GB de espacio libre en `/tmp`. La versión amd64, `.deb`, también ocupa aproximadamente la mitad de su tamaño anterior.
{% endhint %}

***

## Linux amd64 (x86_64)

### Requisitos del sistema

| Requisito | Mínimo | Recomendado |
| --- | --- | --- |
| **Distribución** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **Procesador** | x86_64 (Intel/AMD) | Intel Core i7 o superior |
| **Memoria (RAM)** | 8 GB | 16 GB o más |
| **Tarjeta gráfica** | Ninguna (procesamiento por CPU) | GPU NVIDIA con 4 GB o más de VRAM (12 GB o más desbloquean `GPU_PARALLEL`; 7 GB o más mantienen «Texture Aware» fuera de la ruta de imagen única) |
| **Almacenamiento** | 2 GB de espacio libre | SSD con 10 GB o más de espacio libre |
| **Python** | Python 3.7 o superior (para el SDK) | Python 3.10 o superior |

> **Ubuntu 20.04 y Debian 11 no son compatibles.** La lista de dependencias de `.deb` se
> deriva de aquello con lo que el backend de Chloros se vincula realmente, y eso incluye una
> `libc6` más reciente que la glibc 2.31 que incluyen Focal y Bullseye, por lo que `apt` rechaza la
> instalación directamente en lugar de permitir que falle más tarde en tiempo de ejecución.

### Instalación

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
```

{% hint style="info" %}
**Utiliza `apt install ./…`, no `dpkg -i`.** `apt` resuelve las dependencias del paquete y también instala lo que recomienda: los requisitos del Python SDK, `python3-numpy` y `python3-requests`. `dpkg -i` no instala ninguno de los dos. La CLI y el backend son autónomos y funcionan sin ellos, pero `import chloros_sdk` los necesita; la salida de la instalación indica cuáles faltan. Mantén el `./`, para que `apt` instale el archivo en lugar de buscar un paquete con ese nombre. El paquete también declara `libgl1`, que carga el OpenCV incluido y que puede faltar en una imagen de servidor mínima.
{% endhint %}

Comprueba la instalación:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo apt install ./chloros_1.3.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block ("chloros_sdk 1.3.0 for python3" and the "In a virtualenv: pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl" line), any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend@<user>.service" hint -->

***

## Linux arm64 (NVIDIA Jetson)

### Requisitos del sistema

| Requisito | Mínimo | Recomendado |
| --- | --- | --- |
| **Plataforma** | NVIDIA Jetson con JetPack 6 | Jetson Orin NX de 16 GB o AGX Orin |
| **JetPack** | JetPack 6.x | Última versión de JetPack 6 |
| **Memoria (RAM)** | 8 GB (compartida entre GPU y CPU) | 16 GB o más compartidos (12 GB o más es el umbral para el procesamiento paralelo) |
| **Almacenamiento** | 2 GB de espacio libre | SSD NVMe con 10 GB+ de espacio libre |
| **Python** | Python 3.7+ (para el SDK) | Python 3.10+ |

### Instalación

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Mismo diseño que el amd64 `.deb` (Python SDK, búferes de recepción y servicios de backend incluidos), para Jetson Orin / Orin NX / Orin Nano en **JetPack 6** (L4T R36). Su PyTorch es la compilación CUDA de JetPack 6 para Orin, por lo que el procesamiento por GPU, incluido el «Texture Aware debayer», se ejecuta en la GPU del Jetson. El paquete no incluye CUDA en sí mismo: utiliza las bibliotecas CUDA y cuDNN que instala JetPack 6 (`sudo apt install nvidia-jetpack` en una imagen que carece de ellas). Cuando no se puede utilizar una GPU con CUDA, el procesamiento se ejecuta en la CPU y `--debayer texture-aware` utiliza el «debayer» estándar (el backend registra una línea `[DEBAYER] … needs a CUDA GPU …` explicando el motivo). Para conocer el comportamiento de Jetson en cuanto a memoria, temperatura y despliegue en campo, consulta la [Guía de NVIDIA Jetson](nvidia-jetson-guide.md).

***

## Instalación del Python SDK (todos los sistemas Linux)**El `.deb` instala el SDK automáticamente.** Ambos paquetes (amd64 y Jetson) lo instalan para el sistema `python3`, por lo que `import chloros_sdk` funciona inmediatamente después de la instalación, sin necesidad de pip ni de conexión a red. Sus paquetes (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` y los cargadores de proyectos) se incluyen en `/usr/lib/python3/dist-packages`, que pertenece al paquete: una actualización los sustituye, y al eliminar el paquete se eliminan también. La versión siempre coincide con la CLI y el backend que hayas instalado.

Los requisitos propios del SDK, `python3-numpy` y `python3-requests`, son **recomendados, no obligatorios** por el paquete: `sudo apt install ./chloros_<version>_<arch>.deb` los instala, mientras que `dpkg -i` no lo hace. El resultado de la instalación indica la versión del SDK que importa `python3`, cualquier requisito que falte, y cualquier `chloros_sdk` en cualquier otra ubicación de la ruta de `python3` que sobrescriba la copia del paquete (como uno instalado anteriormente con `pip install --user`).**En un virtualenv**, que no tiene acceso a los paquetes del sistema, instala el archivo «wheel» incluido en él (totalmente sin conexión):

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

**Desde PyPI** — solo para un equipo en el que no haya ningún paquete Chloros instalado y cuyo SDK controle un backend en otro equipo (véase [Modo de backend remoto](../reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

Extras opcionales, para una instalación con pip (en un virtualenv o desde PyPI):

| Extra | Comando | Añade |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` para la transmisión en directo del progreso |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` para transporte BLE (DAQ-M) |

Comprueba el SDK:

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
El SDK de Python se comunica con el backend que `.deb` instala a través de una HTTP API local (`http://127.0.0.1:5000`). `ChlorosLocal` y las utilidades de conexión lo inician automáticamente cuando es necesario; el `connect_all()` de un proyecto guardado nunca lo hace, por lo que, en una máquina sin interfaz gráfica, ejecuta el backend como un [servicio](#always-on-ptp-for-headless-hosts). Utiliza siempre la dirección IPv4 literal en lugar de `localhost`: `localhost` puede resolverse a `::1` y costar aproximadamente dos segundos por solicitud.
{% endhint %}

***

## Configuración inicial

### 1. Iniciar sesión

El acceso a la CLI y al SDK requiere un plan de pago de Chloros+ (**Copper** o superior), lo cual se aplica del lado del servidor: un usuario que haya cerrado sesión obtiene `401 AUTH_REQUIRED`, y un usuario del plan gratuito (Iron) obtiene `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

Las credenciales se almacenan en caché en `~/.chloros/user_session.json`.

{% hint style="info" %}
**Las actualizaciones conservan tu sesión; la desinstalación la borra.** Al instalar una versión más reciente de `.deb` sobre una instalación existente (incluso a través de `chloros-cli update`), se conservan `~/.chloros/user_session.json` y la licencia almacenada en caché, por lo que los trabajos de CLI o SDK sin interfaz gráfica seguirán funcionando. Si se desinstala el paquete (`sudo apt remove chloros`), se borran los datos de inicio de sesión guardados y la caché de la licencia de todos los usuarios del equipo, por lo que una nueva instalación requerirá volver a instalar `chloros-cli login`.
{% endhint %}

### 2. Comprueba el estado de tu licencia

```bash
chloros-cli status
```

`chloros-cli status` funciona en cualquier nivel (incluido el gratuito), por lo que siempre podrás ver por qué el acceso está o no disponible.

### 3. Ejecuta el diagnóstico del sistema

```bash
chloros-cli selftest
```

Se ejecutan siete comprobaciones en orden, y el comando devuelve un valor distinto de cero si alguna de ellas falla:

| # | Comprobación | Qué comprueba |
| --- | --- | --- |
| 1 | **Versión** | La CLI indica su versión (`v1.3.0`). |
| 2 | **Puerto disponible** | El puerto 5000 está libre, *o* ya ha sido respondido por un backend de Chloros en buen estado (lo que cuenta como una superación). |
| 3 | **Inicio del backend** | Se inicia el binario del backend. |
| 4 | **Prueba de API (`/api/test`)** | El backend responde con `status: ok`. |
| 5 | **Información del sistema** | Muestra `GPU: <name>, CUDA: <bool>, PyTorch: <version>` desde `/api/system-info`. |
| 6 | **Modelos del denoiser** | Encuentra modelos `*.pth.enc` (en Linux: `/usr/lib/chloros/models`). |
| 7 | **CUDA + eliminador de ruido**| La función «Texture Aware» es realmente utilizable: requiere CUDA**y** al menos un archivo de modelo. |

La ejecución finaliza con `N/7 checks passed`, enumerando los posibles errores por su nombre.

### 4. Procesa tu primer conjunto de datos

```bash
chloros-cli process ~/datasets/flight001
```

***

## Archivos y directorios

### Por usuario

Chloros guarda sus credenciales y la configuración de la CLI en un único directorio multiplataforma, **`~/.chloros/`** (en Windows, `%USERPROFILE%\.chloros\`). Por su parte, dos cachés específicos de Linux siguen las convenciones XDG; estos respetan las rutas `XDG_CONFIG_HOME` / `XDG_CACHE_HOME` cuando se configuran.

| Ruta | Finalidad |
| --- | --- |
| `~/.chloros/user_session.json` | Caché de sesión de inicio de sesión creada por `chloros-cli login` (se conserva tras las actualizaciones del paquete; se borra al eliminar el paquete) |
| `~/.chloros/working_directory.txt` | Anulación predeterminada de la carpeta del proyecto (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | Preferencia de idioma de la CLI (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Configuración de idioma compartida con la interfaz gráfica de usuario de Windows: un valor `language` aquí tiene prioridad sobre `cli_language.json` |
| `~/.chloros/update_cache.json` | Caché de una hora para la comprobación de actualizaciones al inicio de Linux/Jetson |
| `~/.chloros/backend.log` | Registro del backend cuando este fue iniciado por la CLI |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Paquetes de calibración LATTICE almacenados en caché por cámara, indexados por número de serie y hash del paquete |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Modificaciones opcionales del usuario para los perfiles de corrección de la capacidad de adquisición de datos (DAQ) |
| `~/.config/chloros/system_config.json` | Perfil de hardware almacenado en caché procedente de Dynamic Compute Adaptation: elimínalo para forzar una nueva detección de hardware |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Registros del servidor backend, un archivo por inicio |
| `~/Documents/MAPIR/Chloros Projects/` | Carpeta de proyecto predeterminada cuando no se ha establecido ninguna configuración |

### A nivel del sistema

| Ruta | Finalidad |
| --- | --- |
| `/usr/bin/chloros-cli` | Script envolvente: establece `LD_LIBRARY_PATH` para las bibliotecas nativas incluidas (y de `TMPDIR` a `/mnt/ssd/tmp` cuando existe dicha carpeta), y a continuación ejecuta el binario real |
| `/usr/bin/chloros-backend` | Script de envoltura: lo mismo para el backend, al que marca como una compilación de producción instalada |
| `/usr/lib/chloros/cli/`, `/usr/lib/chloros/backend/` | La CLI y el backend compilados, instalados sin descomprimir (novedad en la versión 1.3.0); los programas son `cli/chloros-cli` y `backend/chloros-backend` |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | Scripts de inicio de dos líneas que ejecutan los programas reales mencionados anteriormente — se mantienen en estas rutas porque el SDK, la CLI y los scripts más antiguos los buscan aquí |
| `/usr/lib/chloros/arena_runtime/` | Entorno de ejecución del SDK de Arena requerido por las cámaras LATTICE |
| `/usr/lib/chloros/models/*.pth.enc` | Modelos de eliminación de ruido cifrados utilizados por el «debayer» sensible a texturas |
| `/usr/lib/python3/dist-packages/chloros_sdk/` (y `lattice_sdk`, `lattice_cam`, `arena_api`, los cargadores del proyecto) | El SDK de Python, instalado para el sistema `python3` y propiedad del paquete |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Archivo «wheel» del SDK de Python correspondiente exactamente a esta compilación, para un entorno virtual |
| `/usr/lib/chloros/exiftool` | El lanzador de ExifTool incluido (enlazado simbólicamente a `/usr/local/bin/exiftool` solo si no existe un exiftool del sistema). A partir de la versión 1.3.0, el script ExifTool se incluye con su propia biblioteca Perl en `/usr/lib/chloros/exiftool-dist/`, por lo que se ejecuta sin el paquete del sistema `libimage-exiftool-perl` |
| `/etc/chloros/update.conf` | Configuración del canal de actualización leída por `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Establece `net.ipv4.ip_unprivileged_port_start = 319` para que el backend pueda vincular los puertos PTP sin privilegios de root |
| `/etc/sysctl.d/60-chloros-gige.conf` | Aumenta los búferes de recepción de sockets del kernel (`net.core.rmem_max`, `net.core.rmem_default`) a 32 MB para las transmisiones de las cámaras LATTICE, y los aplica durante la instalación (si ya se ha establecido un valor más alto, no se modifica). Con la configuración predeterminada del núcleo, los paquetes de la cámara se descartan en el núcleo y una matriz pierde fotogramas y frecuencia de fotogramas; `chloros-cli lattice network` informa de los búferes cuando están bajos. Véase [Hosts de Linux: búferes de recepción de sockets](../reference/cli-reference.md#linux-hosts--socket-receive-buffers) |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Dirige el cargador dinámico a `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Concede al usuario que ha iniciado sesión acceso al puente serie USB DAQ-U (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend@.service` | Servicio de fondo «siempre activo» opcional que ejecuta el backend como el usuario que se indique (instalado, **no habilitado**); para habilitarlo, véase [PTP siempre activo](#always-on-ptp-for-headless-hosts) |
| `/lib/systemd/system/chloros-backend.service` | El mismo servicio que ejecuta el backend como **root**(instalado,**no habilitado**), que se mantiene para que un equipo que lo haya habilitado siga funcionando |
| `/var/log/chloros/` | El registro detallado del servicio root, al que puede acceder el grupo `adm` (se elimina al purgar el paquete) |
| `/usr/share/applications/chloros-cli.desktop` | Entrada del menú de aplicaciones «Chloros CLI» que abre un terminal |

**Cortafuegos (novedad en la versión 1.3.0).** Si `ufw` o `firewalld` están activos en el momento de la instalación, el paquete abre los puertos UDP `5002` (datos DAQ-E), `5353` (detección mDNS), `319` y `320` (PTP), y elimina esas reglas cuando se desinstala el paquete. Si hay un cortafuegos instalado pero no está activo, no añade nada y muestra los puertos que se deben permitir en caso de que se active más adelante. No abre ningún puerto TCP: una actualización del firmware de DAQ-E, en la que el sensor se vuelve a conectar a este ordenador a través de TCP para recopilar la imagen, falla con el código de error `… device never connected back on TCP <port> …` si un cortafuegos activo bloquea dicha conexión.

## Ubicación del ejecutable del backend

La CLI y el SDK detectan automáticamente el backend:

| Componente | Ruta |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Backend | `/usr/lib/chloros/chloros-backend` (un lanzador para `/usr/lib/chloros/backend/chloros-backend`) |

Se puede anular la ruta del backend mediante el indicador de la CLI `--backend-exe` o el parámetro del constructor del SDK `backend_exe`, y el puerto con `--port` (por defecto, `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` apunta a las familias de comandos **`lattice`**,**`project`**y**`daq pool-*`** en un backend remoto. Los comandos principales (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) lo ignoran deliberadamente y siempre se dirigen a `http://127.0.0.1:<port>`.
{% endhint %}

***

## Cámaras LATTICE y sensores de luz DAQ en Linux

Todas las familias de comandos «live-hardware» funcionan en Linux (amd64 y Jetson):

* **`chloros-cli lattice`**: detecta, conecta, configura y captura desde cámaras LATTICE y matrices sincronizadas. El comando `.deb` incluye el entorno de ejecución del SDK de Arena que necesitan y lo registra en el cargador dinámico.
* **`chloros-cli daq pool-*`** — conecta sensores de luz DAQ-U/M/E a través del grupo de backends, transmite espectros calibrados, graba archivos `.daq` e intercambia perfiles de tapa; además, en el caso de un DAQ-E, cambia su configuración de red, lo reinicia y actualiza su firmware. `chloros-cli daq discover` busca sensores que se puedan conectar, y `chloros-cli daq --help` muestra una lista de los subcomandos `pool-*`.
* **`chloros-cli project`** — ejecuta un proyecto guardado (sus cámaras, sensores y ajustes de procesamiento) en modo sin interfaz gráfica.
* **`chloros-cli time-sync`** — inspecciona el «grandmaster» PTP que ejecuta el backend Chloros para las cámaras LATTICE y los sensores DAQ-E.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

`--sensor-id` es un requisito previo para `pool-latest`, `pool-stream`, `pool-record` y `pool-set-cap`; `pool-list` muestra los ID que hay actualmente en el grupo.

{% hint style="info" %}
**`--eth-host` omite la detección.** `pool-connect --eth` encuentra un DAQ-E mediante mDNS, con un recurso de ARP que funciona a partir de una caché ARP vacía; al pasar la IP o el nombre de host del sensor con `--eth-host`, se conecta directamente a él.
{% endhint %}

**Los permisos serie de DAQ-U** se gestionan mediante la regla udev instalada (`uaccess` + grupo `dialout`). Si un sensor que ya estaba conectado sigue sin estar accesible, recarga las reglas o vuelve a conectarlo:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Consulta la [referencia de la CLI](../CLI.md) para ver el conjunto completo de comandos.

### PTP siempre activo para hosts sin pantalla

El paquete instala dos servicios de systemd para el backend, ninguno de ellos habilitado. En un Jetson sin pantalla o en un servidor que deba mantener la sincronización de tiempo PTP en funcionamiento de forma continua para sensores DAQ-E y cámaras LATTICE, o en el que los scripts del SDK conecten proyectos guardados (`connect_all()` nunca inicia un backend), habilita el servicio **como tu usuario**:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service` ejecuta el backend como ese usuario: las capturas que escribe son suyas, su registro detallado se encuentra en su `~/.cache/chloros/logs` y utiliza su `chloros-cli login` (`~/.chloros`), así que primero inicia sesión una vez con ese usuario. Al instalar el paquete se muestra este comando con tu nombre de usuario ya rellenado.

Sin un servicio, el backend (y con él el «grandmaster» PTP) solo se ejecuta mientras algo lo haya iniciado: un comando `chloros-cli` o un script del SDK.

La unidad vincula el backend a los ajustes de entorno `127.0.0.1:5000` (configuración de entorno de `CHLOROS_HOST` / `CHLOROS_PORT` dentro de la unidad; se puede anular con `sudo systemctl edit chloros-backend@.service`) y lo reinicia en caso de fallo tras 5 segundos.

**El servicio root.**`chloros-backend.service` ejecuta el backend como**root**. Sigue instalado para que una máquina que lo haya habilitado siga funcionando, pero todo lo que escribe un backend como root pertenece a root, por lo que un usuario no puede eliminar sus propias capturas; su registro se encuentra en `/var/log/chloros`, legible por el grupo `adm`. Para cambiar a tu usuario:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

**El backend de un servicio es compartido y nunca lo detiene un cliente.** Los scripts `chloros-cli` y SDK lo utilizan tal y como lo encuentran: `process` se ejecuta en él, cargando su propio proyecto en lugar del último. En los casos en los que, de otro modo, la CLI. reiniciaría el backend (`--restart`, o un `process` cuya importación no haya encontrado nada), indica el nombre de la unidad, muestra `sudo systemctl restart <unit>` y sale de `1`, ya que un backend detenido de esa forma permanecería inactivo. Reinícialo tú mismo con `sudo systemctl restart chloros-backend@$USER.service`. El backend de un servicio nunca se detiene por sí mismo cuando está inactivo, y `chloros-cli backend stop` lo rechaza, indicando el comando `systemctl stop`.**Cómo obtiene PTP sus puertos.** PTP utiliza los puertos UDP 319 y 320, ambos por debajo del límite habitual de 1024 para los puertos privilegiados. El paquete `postinst` escribe `/etc/sysctl.d/60-chloros-ptp.conf` con `net.ipv4.ip_unprivileged_port_start = 319`, lo que permite al backend vincularlos mientras se ejecuta como tu usuario. A partir de la versión 1.3.0, el paquete no establece ninguna capacidad de archivo (`setcap`) en el backend, y `libcap2-bin` ya no es una dependencia: en el backend descomprimido, una capacidad de archivo impediría que se iniciara, así que no añadas ninguna por tu cuenta.

***

## Ejemplos de scripts en Bash

{% hint style="info" %}
**Códigos de salida compatibles con scripts.**`chloros-cli process` sale de `0` si se ejecuta correctamente y**con un valor distinto de cero si falla —incluida una ejecución que solicitó productos de imagen pero no generó ninguno** (muestra `Processing finished but wrote no image products.` e indica el nombre de la carpeta del proyecto y las causas habituales). Las ejecuciones correctas informan de cuántos productos de imagen se han generado (`Image products written: N`, más `(plus M clamp masks)` cuando se han generado máscaras). Códigos de salida: `0` (éxito), `1` (fallo), `2` (error de argumento), `130` (interrumpido).
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
    --indices NDVI GNDVI OSAVI \
    --debayer texture-aware \
    --no-vignette
```

Los valores válidos de `--format` son exactamente cuatro y contienen espacios; hay que ponerlos siempre entre comillas:

| Valor de `--format` | Carpeta de salida |
| --- | --- |
| `TIFF (16-bit)` *(por defecto)* | `tiff16` |
| `TIFF (8-bit)` | `tiff8` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` admite `standard` (predeterminado) o `texture-aware` (Chloros+).

### Procesamiento automatizado con Cron

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Ejemplo del SDK de Python

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "OSAVI"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Solución de problemas

### No se encuentra la CLI tras la instalación

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
sudo chmod +x /usr/lib/chloros/chloros-backend /usr/lib/chloros/backend/chloros-backend
```

### El backend no se inicia tras añadir capacidades de archivo

No ejecutes `setcap` en el backend. A partir de la versión 1.3.0, el paquete no establece capacidades de archivo, y los puertos 319/320 de PTP provienen de la configuración `sysctl` descrita en [Cómo obtiene PTP sus puertos](#always-on-ptp-for-headless-hosts). Una capacidad de archivo coloca el backend descomprimido en un modo de ejecución segura en el que no puede encontrar sus propias bibliotecas, por lo que no se inicia. Si ha añadido una, elimínela:

```bash
sudo setcap -r /usr/lib/chloros/backend/chloros-backend
```

### PTP no se inicia / No puede vincular el puerto 319

Comprueba que se haya reducido el límite mínimo de puertos sin privilegios y, si no es así, vuelve a aplicarlo para el arranque actual:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

A continuación, comprueba el «grandmaster»:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### Los comandos `lattice` fallan con el mensaje «No se han encontrado los controladores de la cámara LATTICE»

El backend no ha podido cargar el entorno de ejecución de la cámara LATTICE con el que está instalado. Todos los comandos `lattice` se ejecutan en el backend; la propia CLI no incluye controladores de cámara. Reinstala Chloros (`sudo apt install --reinstall ./chloros_1.3.0_<arch>.deb`): el `.deb` instala el entorno de ejecución junto al backend, en `/usr/lib/chloros/arena_runtime`, y lo registra en el cargador dinámico. Para comprobar ese registro:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
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

Los registros del backend correspondientes al fallo de inicio se encuentran en `~/.cache/chloros/logs/`. Si el puerto 5000 pertenece a un servicio del backend (`systemctl status chloros-backend@$USER.service`), reinicia el servicio en lugar de terminar su proceso.

### No se ha detectado CUDA

```bash
# Check NVIDIA driver installation (x86_64)
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check the L4T release (JetPack 6 is R36) and that JetPack's CUDA libraries are installed
cat /etc/nv_tegra_release
sudo apt install nvidia-jetpack
```

`chloros-cli selftest` informa de lo mismo en una sola línea: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Faltan bibliotecas compartidas

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries (the program itself, not the launcher script)
ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"
```

### Arranque lento en sistemas con tarjeta SD

A partir de la versión 1.3.0, el backend y la CLI se instalan sin descomprimir, por lo que ya no se extraen a un directorio temporal cada vez que se inician. Los archivos temporales se guardan automáticamente en `/mnt/ssd/tmp` si existe esa carpeta; de lo contrario, puedes indicarles un sistema de archivos rápido:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Actualización de Chloros en Linux

El comando `update` solo está disponible en Linux/Jetson. Comprueba la versión publicada en el canal de actualizaciones configurado en `/etc/chloros/update.conf` y ofrece la posibilidad de descargar e instalar la versión correspondiente `.deb`:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

En Linux y Jetson, la CLI también realiza una comprobación de actualizaciones sin bloqueo en cada inicio (el resultado se almacena en caché durante una hora en `~/.chloros/update_cache.json`) y muestra el mensaje `Update available: vX.Y.Z` cuando existe una versión más reciente. Tu configuración, tus proyectos y tus datos de inicio de sesión se conservan tras la actualización.

La actualización del paquete detiene todos los servicios de fondo mientras se instala, por lo que un script en ejecución pierde sus cámaras y sensores, y luego vuelve a iniciar cada uno de los que estén habilitados (uno iniciado manualmente pero nunca habilitado permanece detenido, igual que tras un reinicio).

## Desinstalación

```bash
sudo apt remove chloros
```

La desinstalación detiene y desactiva todos los servicios de fondo, restablece el límite predeterminado de puertos sin privilegios (1024), elimina el archivo de configuración del búfer de recepción (los valores actuales se mantienen hasta el reinicio), el enlace simbólico de exiftool incluido, la configuración del cargador de Arena, las reglas de cortafuegos añadidas por el paquete y el SDK Python instalado, y borra las credenciales almacenadas en caché. Tus proyectos y los archivos de datos de `~/.chloros/` no se ven afectados. `sudo apt purge chloros` también elimina `/var/log/chloros`.

***

## Próximos pasos

* [Guía de NVIDIA Jetson](nvidia-jetson-guide.md) — Optimización e implementación específicas para Jetson
* [CLI: línea de comandos](../CLI.md) — la guía de CLI
* [API: Python SDK](../api-python-sdk.md) — la guía del SDK
* [Referencia de CLI](../reference/cli-reference.md) y [Referencia de SDK](../reference/sdk-reference.md): listados exhaustivos de comandos y API para la versión 1.3.0
* [Adaptación dinámica de recursos de cálculo](../processing-architecture/dynamic-compute-adaptation.md): cómo se adapta Chloros a tu hardware

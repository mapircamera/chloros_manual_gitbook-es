# Chloros Python SDK Referencia

**Versión:**

1.2.0**Generado:**29/07/2026 19:19 ·**Revisado:** 30/08/2026**Paquete:** `chloros-sdk` (PyPI)**Público objetivo:** Optimizado para su uso con modelos de lenguaje grandes (LLM); legible para humanos.**Ámbito:** Todas las clases, funciones y utilidades públicas expuestas por `import chloros_sdk`, con ejemplos que se pueden copiar y pegar y que abarcan el procesamiento de imágenes, el control de una sola cámara, las matrices sincronizadas, los sensores DAQ y la automatización de proyectos.

Si solo necesitas lo más destacado, ve directamente a:
- [Instalación y guía de inicio rápido](#installation)
- [Smart-Connect para matrices LATTICE](#smart-connect-for-lattice-cameras)
- [Sesiones de sensores DAQ](#daq-sensor-sessions)
- [Automatización de proyectos](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)

---

## La arquitectura en 60 segundos

SDK es una capa de «thin-Python» sobre el backend Chloros (el mismo servidor Flask que utilizan la interfaz gráfica de usuario de escritorio y CLI). Para la automatización, se importa `chloros_sdk` y se invocan métodos de alto nivel; en segundo plano, cada llamada se convierte en una solicitud HTTP al backend local en el puerto 5000 — `http://127.0.0.1:5000/api/...` (deliberadamente no `localhost`, que se resuelve primero como `::1` en Windows y cuesta unos 2 s por solicitud frente a un backend solo IPv4). El backend es propietario del conjunto de hardware —cámaras, sensores DAQ, perfiles de alineación, búferes de fotogramas—, por lo que los scripts de SDK pueden coexistir con la interfaz gráfica de usuario sin competir por los puertos serie ni por el ancho de banda de la tarjeta de red.

Hay tres interfaces que utilizarás:

1. **`ChlorosLocal` + funciones libres** (`process_folder`, `process_lattice_capture`) — Canal de procesamiento de imágenes. Procesar una carpeta completa mediante calibración, debayer y exportación de índices con una sola llamada a Python.
2. **Manejo de conexiones inteligentes** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — Abre una sesión de backend persistente para hardware en tiempo real. El mismo flujo de «preparación inteligente» que en la interfaz gráfica de usuario: sonda de red, selección automática de nivel, PTP, inicialización de AE, configuración del disparador GPIO.
3. **`ChlorosProject` / `open_project`** — Carga un proyecto guardado (carpeta con `cameras.json` + `sensors.json` + `project.json`), conecta todo a la vez y realiza capturas de unidad mediante identificadores con nombre.

Las superficies 1 y 2 **inician automáticamente un backend local** si aún no hay ninguno a la escucha (el mismo binario incluido que lanzan la GUI y CLI), por lo que un script básico funciona desde un shell nuevo sin que tengas que iniciar primero un backend. Pasa `auto_start_backend=False` para desactivar esta opción (p.p. ej., al apuntar a un backend remoto, que nunca se inicia). Véase [Inicio automático del backend](#backend-auto-start). La superficie 3 se comporta de forma diferente: `open_project()` no admite el parámetro `auto_start_backend`, y `connect_all()` nunca inicia un backend: comprueba `http://127.0.0.1:5000` una vez y, si no obtiene respuesta, recurre silenciosamente al control directo (sin backendsin backend). Solo `proj.process()` y `stream(..., overlays=True)` crean de forma diferida un `ChlorosLocal()` (que sí se inicia automáticamente).

Los tres requieren autenticación: ejecuta `chloros-cli login` una vez en el equipo o inicia sesión a través de la interfaz gráfica de usuario del escritorio. Las llamadas a SDK sin una sesión válida generan el error `ChlorosAuthenticationError`.

Requisitos:
- Python 3.7+ (según lo declarado por el paquete; desarrollado y probado en la versión 3.10)
- Chloros Desktop instalado localmente (el binario del backend se incluye en el instalador)
- Cuenta activa en Chloros+. El nivel mínimo para SDK / CLI es **Copper**o superior (Copper / Bronze / Silver / Gold); el nivel gratuito**Iron**no tiene acceso a SDK / CLI. Esto se aplica**del lado del servidor**: toda solicitud marcada con SDK / CLI debe incluir tanto una sesión activa como un plan de pago, o el backend devolverá `403` con `error_code: PLAN_UPGRADE_REQUIRED` (que aparece como `ChlorosLicenseError` por `ChlorosLocal`, y como `ChlorosConnectError` por los ayudantes `connect_*`). Un solicitante registradorecibe en su lugar `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`); los dos son distintos porque al volver a ejecutar `chloros-cli login` se corrige el primero, pero no puede solucionar el segundo.
- Se admite el uso sin conexión durante el periodo de gracia del plan: el nivel se lee de la(5 min) o de la caché de licencias firmadas y vinculadas al equipo (30 días para los planes mensuales; hasta el vencimiento de la suscripción para los anuales). Cuando ese periodo de gracia expira, el plan pasa a ser gratuito y el acceso a SDK / CLI se interrumpe hasta que el equipo pueda conectarse al servidor al menos una vez. `chloros-cli status` (`GET /api/license-status`) sigue siendo accesible en el nivel gratuito, por lo que el motivo es evidente: es la única ruta SDK / CLI exenta de la restricción de nivel.
- Windows 10/11 de 64 bits, **Ubuntu 22.04 LTS o posterior**, o Jetson (JetPack 6). Ubuntu 20.04**no** es compatible: las dependencias de `.deb` se derivan de aquello con lo que se vincula el backend, incluido `libc6 (>= 2.34)`, y Focal incluye glibc 2.31.

---

## Instalación

El Python SDK es una capa ligera de Python sobre el backend Chloros. Para cualquier cosa que vaya más allá de unos pocos flujos de trabajo dedicados exclusivamente a la adquisición de datos (DAQ), necesitas tener instalado localmente el **paquete de escritorio Chloros** (instalador Windows o Linux `.deb`); este es el que proporciona el binario del backend, el entorno de ejecución Arena SDK para cámaras LATTICE, y los paquetes de calibración.

Últimas descargas: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### Paso 1 — Instalar el paquete de la plataforma «Chloros»

#### Windows (.exe)

1. Descarga `Chloros-Setup-x.y.z.exe` desde la página de descargas.
2. Ejecuta el instalador y sigue las instrucciones del asistente. La ruta de instalación predeterminada es `C:\Program Files\MAPIR\Chloros\`.
3. Inicia Chloros al menos una vez e inicia sesión con tu cuenta de Chloros+.

#### Linux amd64 (.deb)

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

### Paso 2 — Instalar el paquete «Python» SDK

**El instalador de Chloros incluye un paquete «SDK» correspondiente.** Cada instalador de Windows y cada archivo .deb de Linux coloca en el disco un `chloros_sdk-X.Y.Z-py3-none-any.whl` que coincide exactamente con la versión de la interfaz gráfica de usuario (GUI), del backend (CLI) y del backend. No es necesario estar pendiente de PyPI para mantener la sincronización.

#### Windows

El instalador ejecuta automáticamente `pip install` con el archivo «wheel» incluido utilizando el lanzador de tu sistema Python (se prefiere el lanzador `py.exe`; si no está disponible, se recurre a `python -m pip`). No es necesario realizar ninguna acción — `import chloros_sdk` funciona en tu entorno Python tras una instalación correcta. Si no hay ningún Python en el sistema, el instalador omite este paso de forma silenciosa y la interfaz gráfica de usuario (GUI) y CLI siguen funcionando.

#### Linux (.deb)

El archivo .deb coloca el wheel en `/usr/lib/chloros/sdk/`. El `postinst` muestra el comando exacto: las distribuciones que cumplen con PEP 668 rechazan por defecto las escrituras globales en pip, por lo que no realizamos la instalación automáticainstalar automáticamente:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Para implementaciones de Jetson en modo «air-gapped», esto se realiza totalmente sin conexión: el archivo «wheel» ya se encuentra en el disco.

#### PyPI público

Para hosts que solo utilizan pip (sin el paquete de escritorio «Chloros» instalado; flujos de trabajo con backend remoto o solo DAQ):

```bash
pip install chloros-sdk
```

PyPI se actualiza en las compilaciones del instalador de la versión de lanzamiento, por lo que el archivo «wheel» publicado coincide con la última versión estable. Las compilaciones de desarrollo (p. ej., `1.1.4.dev1`) solo se distribuyen a través del archivo «wheel» incluido en el instalador.

#### Verificar

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Se requiere una suscripción a Chloros+.** Todas las llamadas a SDK requieren un inicio de sesión activo en Chloros+. Ejecuta `chloros-cli login user@example.com 'YourPassword'` una vez por equipo; las credenciales se almacenan en caché en `~/.chloros/`.

### ¿Necesito el paquete de escritorio?

El paquete pip por sí solo **no** es suficiente para la mayoría de los flujos de trabajo. Esto es lo que necesita cada superficie de SDK:

| Superficie de SDK | ¿Necesita el paquete de escritorio? | ¿Por qué? |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Sí** | Inicia automáticamente el binario del backend en `/usr/lib/chloros/chloros-backend` (Linux) o `C:\Program Files\MAPIR\Chloros\…` (Windows). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Sí**(local)**/ No**(remoto) | Clientes «HTTP» puros a través del backend. Backend local → se requiere el paquete de escritorio. Backend remoto → `backend_url=`**a través de un túnel** (véase Modo de backend remoto: los backends incluidos solo se vinculan a la interfaz de bucle cerrado). |
| `ChlorosProject` / `open_project` | **Sí** | Gestiona los proyectos guardados a través del backend. |
| Clases LATTICE directas (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Sí** | Requiere el tiempo de ejecución nativo de Arena SDK que se incluye en el paquete de escritorio. De lo contrario, `CAMERA_AVAILABLE` es `False` en la importación. |
| Clases Direct DAQ (`DAQUSensor`, `DAQMSensor`, `DAQESensor`, `SensorFleet`, `discover_all`) | **No** | Acceso directo a DAQ (Python) a través de pyserial/bleak/zeroconf. Un entorno solo con pip puede gestionar los DAQ de principio a fin. |

### Modo de backend remoto (host solo con pip, a través de túnel)

> **No se puede acceder al backend incluido a través de la LAN.** Las
> compilaciones de producción solo se vinculan a loopback (ambas familias de loopback) y rechazan rotundamente el
> único modo que no es de loopback (`CHLOROS_CLOUD_MODE`), por lo que
> `backend_url="http://<lan-ip>:5000"` **no puede funcionar con un
> Chloros instalado** — ese patrón solo funcionaba con un backend de origen/desarrollo
> (source/dev). Para controlar un backend en otra máquina, reenvía tú mismo su puerto de bucle cerrado
> y dirige el SDK hacia el túnel:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"   # the tunnel endpoint

chloros_sdk.connect_camera("213800234", backend_url=BACKEND)
chloros_sdk.connect_array(serials, backend_url=BACKEND)
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local", backend_url=BACKEND)
```

Los hosts sin interfaz gráfica (headless), de integración continua (CI) o robóticos pueden mantener una máquina con la instalación completa del escritorio como «servidor Chloros» y `pip install chloros-sdk` en todas las demás; sin embargo, el transporte entre ellos es el túnel configurado por el usuario mencionado anteriormente, no una conexión directa por LAN URL.

> **Limitación conocida — `ChlorosLocal` no es compatible exclusivamente con pip.** `ChlorosLocal(backend_url=BACKEND)` actualmente resuelve un binario de backend local en su constructor *antes* de sondear el URL, y genera el error `ChlorosBackendError` («No se ha encontrado el backend Chloros…») cuando no hay ningún paquete de escritorio instalado, incluso si hay un backend remoto accesible. Solo la interfaz de conexión inteligente anterior (`connect_camera` / `connect_array` / `connect_daq_sensor`, además de `analyze_array_network` y los auxiliares `list_*` / `discover_*`) funciona desde un host que solo utiliza pip.

### Flujo de trabajo solo DAQ (host que solo utiliza pip)

Si solo necesitas sensores DAQ y notener que utilizar cámaras LATTICE ni procesamiento de imágenes, el paquete pip es autónomo:

```bash
pip install chloros-sdk
```

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

sensor = DAQUSensor(port="/dev/ttyUSB0")
sensor.connect()
sensor.start_streaming()
```

Sin backend, sin .deb, sin necesidad de iniciar sesión en Chloros+ para trabajar directamente con el hardware de DAQ.

---

## Inicio rápido

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## Índice de primer nivel de API

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()

# === Project lifecycle ===
chloros_sdk.open_project(path)                    # → ChlorosProject
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk / daq_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics
chloros_sdk.DAQUSensor, DAQMSensor, DAQESensor, SensorFleet, discover_all

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.DAQ_AVAILABLE        # True iff daq_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True iff ChlorosProject deps available
```

---

## Procesamiento de imágenes — `ChlorosLocal`

La clase principal del flujo de trabajo. Inicia el backend al usarla por primera vez, crea y configura proyectos, supervisa el progreso y devuelve resúmenes tras la ejecución.

### Constructor

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

### Métodos

| Método | Descripción |
| --- | --- |
| `create_project(project_name, camera=None)` | Crea un nuevo proyecto (opcionalmente con una plantilla de cámara como `"Survey3N_RGN"`). |
| `import_images(folder_path, recursive=False)` | Importa imágenes RAW/TIF/JPG/DNG **y grabaciones de sensor de luz `.daq`****. Devuelve `count` (imágenes) y `scan_count` (grabaciones). Solo muestra un aviso si la carpeta no contiene ninguno de los dos. |
| `export_light_sensor(daq=True, csv=True)` | Escribe los archivos calibrados `.daq` + `.csv` para cadadel proyecto en `<project>/Light Sensor/`. Véase [Grabaciones de sensores de luz](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Configurar los parámetros de procesamiento. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Ejecutar el proceso. Devuelve `{"status": "complete", "async": False}`, además de una clave `summary` cuando el backend la proporcione; véase [Resumen y sugerencias tras la ejecución](#post-run-summary--hints). |
| `get_config()` / `get_status()` / `status()` | Inspecciona el estado del backend. |
| `logout()` | Borra las . |
| `shutdown_backend()` | Cerrar el backend (si se ha iniciado con `SDK`). |
| `discover_cameras()` | Detectar cámaras LATTICE **a través del backend de esta instancia** (`/api/camera/discover`). Devuelve una lista de diccionarios (`serial`, `model`, `ip`, …) — con la misma estructura que la que ven la interfaz gráfica de usuario (GUI) y CLI. Lista vacía si no se encuentra ninguna o si no se puede acceder al backend. |
| `camera_capture(output_dir, format="tiff", **settings)` | Captura un fotograma único**a través del backend**(iniciado automáticamente por este identificador) para que reciba la misma preparación que la GUI/ CLI (por defecto 12 bits, reutilización del pool, metadatos de calibración integrados). Resuelve el objetivo con `serial=` o `device_index=`; pasa `exposure`/`gain`/`pixel_format`/`preset` como `**settings`. Devuelve el diccionario de metadatos heredado (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Genera fotogramas de vista previa con superposición compuesta a partir de una cámara agrupada — cliente MJPEG ligero a través de la ruta `/api/camera/<serial>/stream-annotated` del backend (zebra / cuadrícula / retículo / histograma / peaking / punto dibujado en el lado del servidor). `decode=True` genera matrices BGR; `False` genera bytes sin procesar JPEG. También accesible por proyecto como `ChlorosProject.stream(overlays=True)`. |

Úsalo como gestor de contexto para garantizar la limpieza:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

### Registros del sensor de luz — calibrados `.daq` + `.csv`

Se puede registrar un DAQ-U / DAQ-M / DAQ-E **sin** su paquete de calibración. Eso es
lo que los [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`): escriben los recuentos sin procesar del sensor y marcan el
archivo para que Chloros recupere la calibración de fábrica de ese sensor **por número de serie** — primero en la caché local
y luego en la nube de MAPIR — y la aplica al importarla.

Chloros vuelve a guardar el resultado como dos productos por grabación, bajo
`<project>/Light Sensor/`:

| Producto | Qué es |
| --- | --- |
| `<name>_calibrated.daq` | El archivo reprocesable —mismo esquema que un registro en tiempo real, pero ahora indicando el paquete que lo generó—. Al volver a importarlo, **no** se vuelve a calibrar. |
| `<name>_calibrated.csv` | Irradiancia espectral en W/m²/nm en la propia rejilla de longitudes de onda del sensor, una fila por lectura, más columnas fotométricas (potencia total, lux fotópicos/escotópicos, PPFD y su desglose en azul/verde/rojo, longitud de onda máxima). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Solo sensores sin paquete (DAQ-A).** Recuentos espectrales brutos del sensor — *no* irradiancia. Véase más abajo. |

`process()` realiza esta exportación como una de sus etapas. **No** requiere imágenes:
un sensor de luz que vuela por sí solo constituye un flujo de trabajo de primera clase, y un proyecto de este tipo tiene cero
imágenes por su propia naturaleza.

**Las grabaciones del DAQ-A se exportan como recuentos sin procesar.** La familia DAQ-A es anterior al sistema de paquetes por número de serie
y no tiene ningún paquete que recuperar; en su lugar, se calibra sobre el terreno frente a un
objetivo de reflectancia, por lo que nunca ha necesitado uno. Esas grabaciones se exportan
con el prefijo `_raw` en lugar de `_calibrated`: un nombre de archivo diferente en lugar de un indicador
dentro del archivo, ya que la información debe conservarse al reenviarse por correo electrónico como un simple nombre. El
encabezado `.csv` indica `raw spectral sensor counts (NOT irradiance)` y advierte de que los
valores son comparables **dentro** del archivo —exactamente para lo que los utiliza la calibración basada en objetivos—
y no entre sensores. Las columnas fotométricas dependientes de la potencia (potencia total,
lux fotópico/escotópico, PPFD) devuelven **NULL** en lugar de integrarse a partir de los recuentos.

Un DAQ-U / DAQ-M / DAQ-E cuyo paquete simplemente no se haya podido recuperar sigue siendo **omitido**,
no se escribe en formato sin procesar: en ese caso, el paquete existe y «volver a conectar y volver a procesar» es un consejo válido.

Las grabaciones heredadas **v1.01 / v1.02** (un DAQ-A-SD las escribe) no incluyen una época por lectura,
sino solo la hora de escritura del archivo. El comparador de imágenes↔luz descendente sigue rechazándolas —comparar un
fotograma con una hora de escritura daría un error imperceptible—, pero el exportador las lee, y el
«CSV» imprime `clock=daq_created_on`, de modo que el producto indica en qué reloj se encuentra.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

Una grabación cuyo paquete de calibración no se puede recuperar (fuera de línea o un sensor sin
calibración en el archivo) se notifica con el código `skipped` **indicando el motivo**. Nunca
se guarda como un archivo «calibrado» que contenga recuentos sin procesar: conéctate a Internet y
vuelve a ejecutar la tarea, y la exportación se completará.

### Llamadas de progreso

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Resumen y consejos tras la ejecución

Una vez finalizada la ejecución, `process()` recupera `GET /api/processing-summary` y adjunta el cuerpo como `result["summary"]`. La recuperación se realiza en la medida de lo posible y nunca bloquea un retorno satisfactorio — si el resumen no está disponible, `process()` recurre a la forma básica de `{"status": "complete", "async": False}`. Cada entrada de `summary["hints"]` —frases completas con la solución sugerida, p. ej., por qué una ejecución no ha generado ningún resultado —, se vuelve a emitir como un `UserWarning` «Python», de modo que las ejecuciones sin resultados se autodiagnostican incluso si nunca se consulta el diccionario:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]` es la parte legible por máquina:

| Clave | Lo que cuenta |
| --- | --- |
| `models` | Grupos de cámaras en la ejecución. |
| `images_in_groups` | Imágenes originales de esos grupos. |
| `targets_found` | Objetivos de reflectancia detectados. |
| `images_calibrated` | Imágenes con las que se calibró la ejecución. |
| `exported_files` | **Archivos de productos de imagen generados por la ejecución.** |
| `daq_recordings_exported` / `daq_recordings_skipped` | Registros del sensor de luz, contabilizados por separado a propósito: proceden de una fase diferente y existen para ejecuciones sin imágenes en absoluto, por lo que incluirlos haría que una ejecución solo de DAQ pareciera hubiera exportado imágenes. |

Junto a ellos: `summary["output_dirs"]` (cada directorio en el que se ha escrito),
`summary["light_sensor_export"]`, `summary["stopped"]` (se marca cuando el usuario ha interrumpido la
ejecución, de modo que los recuentos parciales no se interpreten como una ejecución completada con un rendimiento inferior al esperado), y
`summary["groups"]` (el desglose por grupo).

`exported_files` lo registra la cadena de procesamiento **a medida que escribe**, no se extrae posteriormente de los
objetos de imagen del proyecto. Las estrategias paralelas y de GPU crean sus propios objetos de imagen
(en subprocesos de trabajo para las rutas de GPU), por lo que el escaneo anterior informaba de
`0 file(s) written` por cada ejecución de este tipo y, a continuación, emitía la indicación de «cero exportaciones» —en ejecuciones
en las que todo había funcionado correctamente. Si se crea un script basado en este número, una ejecución paralela correcta ahora
muestra un recuento distinto de cero.

Los saltos del sensor de luz informan del motivo que el lector estableció realmente para cada archivo —un
esquema ilegible, un paquete que falta, un error de escritura— **deduplicado**, de modo que veinte archivos
omitidos por una misma causa se interpretan como una sola causa en lugar de veinte repeticiones de la misma.

> **`process()` no se activa cuando una ejecución no genera imágenes.** Este es el único aspecto en el que SDK y
> CLI difieren deliberadamente: `chloros-cli process` trata «se solicitaron productos, pero no se
> escribieron» como un error y sale con un valor distinto de cero, mientras que SDK finaliza normalmente e informa de la
> condición a través de `summary` / indicaciones. Si tu canalización debe detenerse en una ejecución vacía, compruébalo
> tú mismo: inspecciona `summary` (o cuenta los archivos en la carpeta del proyecto) en lugar de basarte en
> la ausencia de una excepción. Las causas habituales son una carpeta de entrada que no se ha reconocido como
> captura y productos omitidos por no ser aplicables a las cámaras presentes (p. ej., radiancia de cámaras que solo
> capturan RGB).

### Funciones de conveniencia

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

### Valores admitidos

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as already-debayered BGR
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### Salida radiométrica (flujo de trabajo multiespectral LATTICE)

El nivel de exportación multiespectral LATTICE (M3C/M3M) del flujo de trabajo `process`: `reflectance` (por defecto), `radiance`, `sensor-response` o `all` (cada modo aplicable por imagen)— se corresponde con el **«ajuste de procesamiento de salida radiométrica»** del proyecto. `configure()` tiene una palabra clave específica para ello:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

La salida de emergencia avanzada —escribir la clave `"Radiometric output"` del proyecto a través de `custom_settings` — sigue funcionando, pero recuerda que sustituye todo el bloque de configuración (consulta la advertencia a continuación):

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Radiometric output": "radiance"},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (el valor por defecto) divide la radiancia de la cámara por el **flujo descendente del DAQ con marca de tiempo coincidente**, resuelta automáticamente a partir de un `.daq` (DAQ-U/M/E) registrado**o un `.csv` nativo de DAQ-M**que se encuentre junto a las imágenes; cualquier paquete de calibración por cámara o DAQ que falte localmente se**obtiene automáticamente de AWS** en el primer uso. El CLI muestra esto como opciones de activación por tipo de producto en `chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.

> `custom_settings` **sustituye** todo el bloque de ajustes calculados (ignora lasdemás palabras clave y validaciones de `configure()` por diseño). Cuando lo utilices, incluye todas las claves `Project Settings` que te interesen, tal y como se muestra en el ejemplo anterior.

---

## Smart-Connect para cámaras LATTICE

Sesiones de backend persistentes para hardware en tiempo real. Se utilizan los mismos puntos finales que la interfaz gráfica de usuario, por lo que el comportamiento es idéntico en SDK / CLI / la interfaz gráfica de usuario.

### Cámara única — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

#### Firma de `connect_camera()`

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

#### Métodos de `CameraSession`

| Método | Descripción |
| --- | --- |
| `read_nodes(names, enum_names=(), timeout=30.0)` | Lee nodos GenICam; devuelve `{nodes, errors, enums, device}`. |
| `set_settings(**kwargs)` | Escribe nodos por nombre descriptivo (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, settings=None, timeout=None)` | Captura un **único** fotograma. Devuelve una lista de un elemento con diccionarios de metadatos del fotograma. (Se ha eliminado lase ha eliminado; llama a `capture()` en un bucle si necesitas una serie.) |
| `disconnect()` | Liberar del grupo. No realiza ninguna operación si nos hemos conectado a una sesión ya abierta. |

Controles de exportación de `capture()` (mismo modelo que la matriz + GUI):

- `processing` / `levels` — `processing="all"` guarda todos los tipos de exportación aplicables; `levels=["raw","radiance"]` guarda solo esos (anula `processing`). Omite ambos para el valor predeterminado del backend.
- `force_daq=True` — guarda la lectura asignada de DAQ/DLS como un archivo complementario `.daq` incluso en una captura solo en formato sin procesar, de modo que la trama se pueda volver a procesar posteriormente en reflectancia/índice. No realiza ninguna operación si no hay ningún DAQ vinculado.

### Matriz sincronizada — `ArraySession` (Smart-Prep)

`connect_array` es **el punto de entrada recomendado** para configuraciones multicámara. Ejecuta en segundo plano todo el flujo de Smart-Prep de la interfaz gráfica de usuario:

1. **Análisis de red** (`/api/camera/array/recommend`): determina el tamaño de fotograma máximo que cabe en el nivel «sim-emit» sin perder fotogramas.
2. **Selección automática de nivel**: `sim-capture-sim-emit` si la conexión lo admite; de lo contrario, `sim-capture-ftd-stagger` o `slip-emit-and-capture`.
3. **Reducción automática**— reduce silenciosamente el tamaño de fotograma o aumenta el binning cuando el cable no puede mantener la resolución solicitada.**Esta medida de seguridad no cubre la sobresuscripción agregada**: un exceso de cámaras para el cable no se puede solucionar reduciendo los fotogramas — véase [Sobresuscripción](#over-subscription-the-per-cam-floor).
4. **PTP habilitado** por defecto: las marcas de tiempo entre cámaras son comparables con una precisión de microsegundos.
5. **Selección automática del formato de píxeles por cámara** — cámaras «RGB» → `BayerRG8`, multiespectrales → `BayerRG12`.
6. **Inicialización de AE** — captura el estado actual de AE de cada cámara para que la conexión no reinicie la exposición a mitadel vuelo.
7. **Configuración del disparador GPIO** — `connect_array` activa todas las cámaras (`TriggerMode=On`, `TriggerSource=Line2`) para que el pulso del maestro active a los esclavos a través del cable M8. Este paso es exclusivo para matrices: una sola cámara abierta con `LatticeCamera` funciona en modo libreen su lugar.

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

#### Firma de `connect_array()`

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # master trigger fire rate (auto if None)
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=180.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

Valores de `force_tier`:
- `"sim-capture-sim-emit"` — verdaderamente simultáneo (todas las cámaras disparan en el mismo flanco del reloj).
- `"sim-capture-ftd-stagger"` — escalonamiento flexible en el dominio del tiempo (las levas emiten en momentos ligeramente desplazados, de modo que los paquetes se serializan en la línea).
- `"slip-emit-and-capture"` — captura secuencial por cámara (sin sincronización temporal; única opción cuando ningún tamaño de trama se ajusta a la sincronización simultánea).

`wire_ceiling_mbps` anula el **presupuesto de ancho de banda sostenido del host** en MB/s — el único
cifra del que depende toda la asignación de la matriz. Déjalo en `None` para utilizar el valor detectado automáticamente
. Redúcelo cuando la matriz informe de tramas corruptas por GVSP: el valor automático se deriva
de la velocidad de enlace anunciada por la tarjeta de red, que sobreestima los adaptadores USB, los carriles PCIe de ancho reducido y
las estructuras compartidas muy ocupadas; y esa sobreestimación se manifiesta en forma de tramas corruptas en lugar de como un
enlace visiblemente lento. El valor se conserva en el bloque de captura de la matriz del proyecto, por lo que una
al reabrirlo o al establecer posteriormente `connect_array` se restablece como cualquier otro ajuste de la matriz.
Véase [Estado de la matriz](#array-health--which-subsystem-is-losing-frames).

#### Sobresuscripción (el límite mínimo por cámara)

El control de ritmo de emisión simulada asigna a cada cámara una parte del presupuesto de ancho de banda a prueba de colisiones, con un mínimo de **8 MB/s por cámara**(`per_cam_floor_bps`). Una vez que `N × floor` supera el límite máximo de seguridad contra colisiones, la matriz**sobresuscribe el canal**—el modo de fallo es la pérdida de paquetes GVSP, no una frecuencia de fotogramas más baja— y no existe ninguna solución relacionada con el tamaño de los fotogramas:**el agrupamiento y la región de interés (ROI) reducen los bytes por fotograma, no los bytes regulados por segundo**que compara la comprobación agregada. Límites máximos prácticos a resolución completa en un host de 1 GbE:**6 cámaras a 1500 MTU, 9 con tramas jumbo** (`max_cams_collision_safe` en la respuesta del análisis indica el límite máximo para tu conexión). Soluciones: menos cámaras, tramas jumbo de extremo a extremo o una tarjeta de red más rápida.

- Las respuestas de `analyze_array_network()` y `/api/camera/array/connect` incluyen `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` y `per_cam_floor_bps`. Cuando `oversubscribed` es verdadero, la proyección **pone a cero los campos de fps** (`achievable_fps_max` / `fps_bright` / `fps_dark`) en lugar de indicar una tasa engañosa que, aunque lenta, funciona.
- `POST /api/camera/array/connect` acepta un parámetro de cuerpo `pin_resolution` (**solo HTTP — no es un argumento kwarg de SDK**; `connect_array` no lo expone). La fijación elimina la red de seguridad del descenso por binning, por lo que una conexión sobresuscrita con `pin_resolution` establecido es**rechazada de forma tajante** con un error en el que se indican todas las soluciones posibles. Sin el «pinning», la conexión continúa con el «walk-down», pero advierte de que la reducción no puede liberar el agregado.
- Vía de escape para pruebas de banco: configura `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` en el entorno del backend para rebajar el rechazo a una advertencia clara: te conectas de todos modos y aceptas la pérdida de paquetes.

#### Estado de la matriz: qué subsistema está perdiendo tramas

`GET /api/camera/array/<array_id>/capability` contiene un bloque activo `health` en una
matriz conectada, reevaluado en una ventana **de 10 segundos** continua. Desglosa la pérdida de tramas
en dos causas que requieren soluciones opuestas, en lugar de una tasa de «incompletos» que
no especifica ninguna de ellas:

| Campo | Qué significa | Qué subsistema |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (por serie) | El fotograma **llegó y presentaba un error estructural**— pérdida de paquetes GVSP. |**Red**: ancho de banda, ritmo de transmisión, anillo de recepción de la NIC, MTU |
| `never_arrived_rate_pct` (por número de serie) | La trama **nunca llegó**— la cámara no se activó o no salió nada de ella. |**Disparador/sincronización**: cable M8, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | La peor tasa de la cámara en cada caso. | — |
| `per_cam_rate_pct` | Tasa combinada de fallos por cámara (ambas causas juntas). | — |
| `stable_for_seconds` | Cuánto tiempo ha permanecido cada cámara por debajo del 0,01 %. | — |

Junto con `health`, el mismo registro indica la cantidad de la asignación total que queda pendiente:

| Campo | Qué significa |
| --- | --- |
| `wire_ceiling_mbps` | El presupuesto de ancho de banda sostenido del host actualmente en vigor, en MB/s. |
| `wire_ceiling_source` | De dónde procede ese número, en palabras — p. ej., `USB-capped 200 MB/s (was theoretical 1062; …)` o `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` cuando `wire_ceiling_mbps=` lo establece. |
| `nic_is_usb` | `true` para un adaptador USB-Ethernet. |

No hay ningún contenedor «SDK» para este punto final; léelo directamente:

```python
import requests, chloros_sdk

arr = chloros_sdk.attach_array(["213800234", "214000533"])
h = requests.get(
    f"http://127.0.0.1:5000/api/camera/array/{arr.array_id}/capability",
    timeout=10).json()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    arr.disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Al leerlo:** un valor distinto de cero en `gvsp_corrupt_rate_pct` con `never_arrived_rate_pct` en 0 significa
que el disparo y la sincronización del cable son perfectos y que el 100 % de la pérdida se produce en la ruta de red — reduce
el valor de `wire_ceiling_mbps` y vuelve a conectarte. El patrón inverso apunta, en cambio, al cable de sincronización o a la
línea de disparo.

> **`target_fps` no es el factor determinante para las tramas corruptas.** El ritmo de GevSCPD se establece una vez al
> conectarse, por lo que reducir la frecuencia de disparo cambia el ciclo de trabajo y no la
> tasa de ráfagas de emisión simultánea. Una reducción medida de la demanda de 5× no produjo ninguna mejora, mientras que
> bajar el límite máximo del cable de 240 a 200 MB/s redujo el porcentaje de tramas corruptas de ese mismo equipo del 10,4 % al
> 0,00 %.

> **La reducción automática a mitad de flujo no está disponible en el firmware del TRI032S.** Una matriz en funcionamiento no puede
> solucionar esto por sí misma; hay que desconectar y volver a conectar para que el selector de tiempo de conexión vuelva a planificar en función
> del nuevo límite máximo.

Un **adaptador Ethernet USB tiene un límite de 200 MB/s** impuesto por la sonda, independientemente de su
especificación técnica: la tabla de eficiencia que convierte una velocidad de enlace en una cifra sostenida se
deriva de PCIe, y una tarjeta de red USB anuncia su velocidad de enlace Ethernet, aunque está limitada por el
bus USB y su controlador. El límite es absoluto, no una fracción: un adaptador USB de 1 GbE
alcanza unos 80 MB/s y no se ve afectado.

#### Métodos `ArraySession`

| Método | Descripción |
| --- | --- |
| `status(timeout=10.0)` | `{fps, ptp, frame_count, last_error, …}` en tiempo real. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, smart=False, timeout=300.0)` | Un grupo de captura sincronizado. Devuelve una `CaptureResult` (lista de diccionarios de fotogramas + `.skipped`). Controles de exportación a continuación. |
| `capture(..., smart=True)` | **Captura inteligente** : espera a que el AE se estabilice en todas las cámaras y, a continuación, se activa. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, timeout=120.0)` | Captura más rápida: solo datos sin procesar + la lectura DAQ asignada (+ el índice combinado libre). Refleja el botón «Captura más rápida». |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Única / Continua / Por intervalos en un bucle delimitado. Devuelve `list[CaptureResult]`.**Requiere `count` y/o `duration_s`** para que finalice (el «SDK» no admite Ctrl+C). |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, timeout=30.0)` | Inicia la grabación de la vista de índice combinado en directo a vídeo/GIF → `RecorderHandle`. Una grabadora compuesta por cada matriz. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, timeout=30.0)` | Iniciar una ráfaga de imágenes Bayer sin procesar a alta velocidad de fotogramas → `RecorderHandle`. Reprocesar fuera de línea con `build_video()`. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Reprocesar fuera de línea una ráfaga RAW guardada en uno o varios vídeos calibrados. Queda en espera hasta que finalice (`wait=True`) y devuelve `{outputs, errors, combined}`. |
| `build_video_status(job_id, timeout=15.0)` | Consulta un trabajo de compilación fuera de línea: `{running, result, error, burst_dir}`. |
| `disconnect()` | Libera toda la matriz. |

`capture()` controles de exportación (el mismo punto final que utilizan la interfaz gráfica de usuario y CLI):

- `processing` / `levels` — `processing="all"` (o `levels=["raw","radiance",…]`) guarda todos los tipos de exportación aplicables por cámara; un único valor `processing` guarda solo ese nivel.
- `aligned=True` — transforma la exportación no sin procesar de cada miembro según el [perfil de alineación](#array-alignment) (co-registrado); los datos sin procesar permanecen sin deformar, pero llevan la transformación en los metadatos. Recurre a la opción «sin alinear» (con una advertencia que aparece en el `alignment` del resultado) si la matriz no tiene perfil.
- `render_index=False`: omite la superposición del índice de vegetación por cámara; por defecto, se muestra donde se haya configurado.
- `force_daq=True` — guarda la lectura DAQ/DLS asignada como un archivo adjunto `.daq`, incluso cuando ningún nivel seleccionado lo requiera.

**Compresión TIFF (control HTTP -only):**`ArraySession.capture()` no envía ninguna clave `compression`, por lo que se aplica el valor predeterminado del backend: `POST /api/camera/array/capture` lee un parámetro de cuerpo `compression`, `"deflate"` por defecto (zlib L1 sin pérdida + predictor horizontal, ~4,1 MB por fotograma a resolución completa). `"none"` escribe sin comprimir (~6,3 MB/fotograma) con una**escritura ~5 veces más rápida** — ambos son sin pérdidas y se leen de forma idéntica al importarlas. El `SDK` no expone ningún `kwarg` para ello; la vía de escape es ``chloros-cli lattice array-capture --compression none`` o el formato sin procesar `HTTP`. DEFLATE también retiene el GIL de `Python`, por lo que las escrituras comprimidas nose pueden paralelizar entre los hilos de escritura de cada cámara; la captura sostenida a resolución completa con 8 cámaras a la velocidad del sensor requiere `compression: "none"`. Detalles: [Referencia de CLI → array-capture](cli-reference.md).**Anulaciones de exportación por miembro (solo HTTP):**el mismo punto final también acepta `exclude_serials` (lista — eliminar miembros del conjunto guardado; la matriz sigue activándose como un único grupo sincronizado y los miembros excluidos se devuelven en `excluded`), `serial_levels` (anulaciones a nivel de cámara de `{serial: [level tokens]}`) y `serial_index` (anulaciones de superposición de índices por cámara en `{serial: bool}`). Se trata de parámetros de paridad de la interfaz gráfica de usuario y**todavía no son argumentos clave-valor deSDK**; los miembros ausentes de los mapas recurren a los valores globales de la matriz `levels` / `render_index`.

##### Inspección de cámaras omitidas — `CaptureResult.skipped`

`ArraySession.capture()` devuelve un `CaptureResult`, que es una subclase de `list`: itera sobre él, indexa sus elementos, aplica `len()` a él — todos los patrones existentes siguen funcionando. El nuevo código puede inspeccionar el atributo `.skipped` para ver qué cámaras se han excluido y por qué. El caso más habitual es el de las cámaras «RGB» en una matriz dematriz de filtros mixtos cuando se solicita `processing="radiance"` o `"reflectance"`: la radiancia por píxel Bayer carece de sentido para un sensor de banda ancha, por lo que el backend omite esas cámaras en lugar de generar datos sin sentido.

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

Los tokens de motivo siguen el patrón `<level>-not-applicable-to-rgb-cam` (una entrada por cada nivel omitido, cada una con `level`). Las omisiones específicas de reflectancia son `reflectance-skipped-no-fresh-dls` (no hay disponible ninguna lectura reciente de la radiación descendente), `reflectance-skipped-bound-daq-unavailable (…)` (no se pudo acceder al DAQ vinculado) y `dls-uncalibrated-band-<nm>` —la banda se encuentra en su mayor parte fuera del rango radiométricamente calibrado del sensor de luz del DAQ (~374–974 nm), por lo que se rechaza la división de reflectancia absoluta basada en el DAQ y el fotograma se rebaja de forma evidente a la respuesta del sensor. Entre las referencias disponibles, solo la F988 lo activa; la ruta compatible con esa cámara es el flujo de trabajo del panel de reflectancia.

`processing` niveles:

| Nivel | Salida |
| --- | --- |
| `"raw"` | Bayer monocanal (cámaras monocromáticas: la banda única) directamente desde el sensor. |
| `"debayered"` *(valor predeterminado de SDK)* | BGR de 3 canales mediante demosaico bilineal (cámaras monocromáticas: escala de grises de 1 canal). |
| `"radiance"` | float32 W/m²/sr/nm a través de la cadena radiométrica completa. Solo multiespectral: se omiten las cámaras «RGB». |
| `"reflectance"` | uint16 0..32768 (compatible con Pix4D); requiere un emparejamiento DAQ en tiempo real para obtener una referencia absoluta. Solo multiespectral. |
| `"display"` | Cadena completa que coincide con la vista previa de la interfaz gráfica de usuario (CCM + WB + gamma según el perfil de la cámara). |
| `"all"` | **Un archivo por cada nivel aplicable** para cada cámara (que coincide con el valor predeterminado de la interfaz gráfica de usuario «Capturar todo» / CLI). El archivo `CaptureResult` devuelto contiene entonces un diccionario de fotogramas por cada `(cam, level)`, con el nivel en cada diccionario; los niveles no aplicables aparecen en `.skipped`. La lectura del DAQ utilizada para cualquier fotograma de reflectancia se guarda como un archivo complementario `.daq`. |

> **Nota: el valor por defecto difiere del de CLI.** `ArraySession.capture()` toma por defecto el valor de `processing="debayered"`; el comando `chloros-cli lattice array-capture` toma por defecto el valor de `processing="all"`. Pasa `processing="all"` explícitamente desde SDK para reflejar el guardado multinivel de CLI /GUI.

### Modos de captura y grabadoras

La superficie de la matriz refleja el panel de captura de la interfaz gráfica de usuario: modos de obturación Único / Continuo / Intervalo / Más rápido, además de dos grabadoras (vídeo compuesto en directo y ráfaga sin procesar → reprocesamiento fuera de línea).

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Combined-index video/GIF recorder (needs the combined live view streaming)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**es el bucle «Continuo/Intervalo» de «SDK». Dado que no existe un `Ctrl+C` para interrumpirlo desde un script,**debes** pasar `count` y/o `duration_s` (se detiene cuando se alcanza cualquiera de ellos). `interval_s` se mide desde el inicio de cada pasada (al igual que en la interfaz gráfica de usuario). El resto de kwargs se pasan directamente a `capture()`.
- **`record`** es de *nivel de monitorización*: captura el compuesto de índice combinado en tiempo real tal y como se muestra, por lo que el flujo combinado debe estar abierto para que lleguen los fotogramas. Un grabador compuesto por matriz (genera una excepción si ya hay uno en ejecución).
- **`burst` → `build_video`** es de *nivel de análisis*: `burst` escribe fotogramas sin procesar + un+ un `.daq` por cada lectura DLS distinta bajo `<output>/bursts/<base>/` a la velocidad máxima del bucle de captura (sin cadena, sin exiftool, sin vista en directo). `build_video` sincroniza cada fotograma con el `.daq` más cercano y vuelve a ejecutar la cadena de radiancia/reflectancia/índice del proceso de importación. `products` es una lista de `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}` (por defecto: el índice combinado). `burst().stop()` también inicia automáticamente una construcción del índice combinado con el mejor resultado posible, que se devuelve como `build_job` en el resultado de la detención.

#### `RecorderHandle`

Devuelto por `ArraySession.record()` y `ArraySession.burst()`. Úsalo como gestor de contexto para detener automáticamente al salir del ámbito, o contrólalo manualmente.

| Miembro | Descripción |
| --- | --- |
| `job_id` | ID del trabajo de backend (cadena). |
| `kind` | `"composite"` (de `record`) o `"raw"` (de `burst`). |
| `start_stats` | El diccionario devuelto por la llamada a `start`. |
| `result` | `None` mientras se está ejecutando; el diccionario con el resultado final de la parada una vez detenido. |
| `stats(timeout=10.0)` | Estadísticas del trabajo en tiempo real (fotogramas grabados, fps reales, transcurridos). |
| `stop(timeout=60.0)` | Detiene el grabador; devuelve y almacena en caché el resultado final. Idempotente (una segunda llamada devuelve el resultado almacenado en caché). |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

### Conectarse a una matriz ya conectada — `attach_array`

Si la matriz ya está activa (la interfaz gráfica la ha abierto o una sesión anterior de SDK ha llamado a `connect_array`), utiliza `attach_array` para obtener un identificador de la misma en lugar de volver a conectarte. `connect_array` siempre devuelve el error «La cámara  <sn>ya </sn>está <sn>en el conjunto<id>», en esa situación, ya que el envío de un POST con `/array/connect` para un miembrodel grupo no es idempotente; `attach_array` lee `/api/camera/array/list` y realiza la coincidencia bien por array_id o por números de serie.

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

Patrón: SDK. Los scripts que comparten entorno con la interfaz gráfica de usuario de escritorio deberían intentar primero `attach_array` y recurrir a `connect_array` si aún no hay ninguna matriz en el grupo.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Importante: la salida de-manager SÍ provoca la desconexión.**`ArraySession.disconnect()` siempre envía un POST a `/array/disconnect`; no existe una protección de «conectado pero no propio» como la que hay para `CameraSession` / `DAQSensorSession`. Si compartes espacio con la interfaz gráfica de usuario y no quieres desmontar la matriz al salir del ámbito,**no utilices el bloque `with`**: guarda el identificador en una variable normal y omite el `disconnect()` explícito:
>
> ```python
> arr = chloros_sdk.attach_array(serials)
> arr.capture("output/", processing="reflectance")
> # … script ends; array stays up for the GUI
> ```

### Ayuda para el análisis de redes

Útil antes de abrir la matriz: permite prever si la configuración propuesta encajará:

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    print(f"Shrink to {r['out_width']}x{r['out_height']} binning={r['binning']}")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

`status` es uno de los siguientes: `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (en caso contrario, `error`). `auto_capped_fps` significa que la resolución solicitada solo se adapta al anillo RX a una frecuencia de disparo limitada; mantén la resolución y pasa `target_fps=result["recommended"]["recommended_target_fps"]` a `connect_array` (véase [Ejemplo 6](#6-capability-probe-before-connecting-a-4-cam-array)).

**Cómo interpretar la proyección** (mismo modelo que el panel «Configuración de la matriz» de la interfaz gráfica de usuario):

- **La ráfaga (`frame_bytes_total`) se suma por cámara según el formato de píxeles real de cada una de ellas.**Las cámaras mono**M3M**transmiten Mono12 (2 B/px) independientemente del valor de `pixel_format` que se pase, por lo que un fotograma a resolución completa de 4 cámaras tiene un tamaño de**~25 MB** con tres cámaras mono, y no los ~12,6 MB que daría la suposición de 8 bits para todas.. El backend determina el formato de cada cámara a partir de su modelo.
- **Admisión (`burst_fits_nic_ring`) tiene en cuenta el drenaje**, no la diferencia entre ráfaga completa y anillo: la emisión simulada encaja cuando el host vacía el anillo de recepción más rápido de lo que las cámaras lo llenan. Un host de 10 G + cámaras de 1 GbE**admite** resolución completa incluso cuando la ráfaga excede el anillo; un host de 1 GbE bloquea (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` es un límite máximo conservador de recuperación en serie** — `max(readout+emit, N×emit)` con la emisión por cámara limitada al enlace de cámara de 1 GbE, independientemente de la exposición. Por ejemplo, ~2,8 fps para una matriz de 4 cámaras a resolución máxima de 12 bits (coincide con los ~2,7–3,0 medidos en tiempo de ejecución). Modelo completo: [CLI Referencia → fps de la matriz y modelo de ráfagas](cli-reference.md#array-fps--burst-model).
- **La sobresuscripción (`oversubscribed: true`) significa que el mínimo de N × por cámara supera el máximo seguro frente a colisiones** — los campos de fps (`achievable_fps_max` / `fps_bright` / `fps_dark`) muestran un valor de 0, y la reducción automática o elagrupación automática no pueden solucionarlo (reducen los bytes por trama, no los bytes por segundo según el ritmo establecido). Las soluciones son reducir el número de cámaras, utilizar tramas jumbo o una tarjeta de red más rápida; `max_cams_collision_safe` indica el límite máximo (6 cámaras a resolución completa en 1 GbE con MTU de 1500, 9 con tramas jumbo). La respuesta también incluye `aggregate_demand_bps`, `collision_safe_ceiling_bps` y `per_cam_floor_bps` (8 MB/s). Véase [Sobresuscripción](#over-subscription-the-per-cam-floor).

### Detección y listado

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

---

## Smart-AE / Smart-Capture

Las matrices LATTICE ejecutan el AE de forma continua en segundo plano tan pronto como se conectan, pero una escena recién enfocada tarda un momento en converger. **Smart-Capture** es la solución más cómoda: comprueba la exposición de cada cámara, espera hasta que la matriz se estabilice en toda la ventana y, a continuación, activa la captura. Es equivalente a la interfaz gráfica de usuario: el botón de captura «inteligente» de la aplicación de escritorio llama al mismo punto final del backend.

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

Al utilizar `ChlorosProject` (siguiente sección), dispones de más opciones de configuración:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

La política de AE inteligente es conservadora por defecto. Ajusta `exposure_tolerance_pct` para trabajos radiométricos exigentes; amplíala para escenas que cambian rápidamente en las que solo se busca una precisión «suficientemente cercana».

---

## Sesiones de sensores DAQ

Grupo de fondo persistente para sensores espectrales (DAQ-U por USB, DAQ-M por BLE, DAQ-E por Ethernet). Refleja la superficie de la cámara: detección inteligente, reutilización del grupo, conexión idempotente.

### Detección inteligente (configuración cero)

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

Prioridad: Ethernet → BLE → USB. Pasa cualquier indicación explícita para fijar el transporte.

### Transporte fijado

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

### Métodos de `DAQSensorSession`

| Método | Descripción |
| --- | --- |
| `status(timeout=10.0)` | Resumen de la entrada del grupo (estado de transmisión/grabación, rango de longitudes de onda, SHA de calibración, tiempo de integración, frame_avg, estado de AE). |
| `latest(n=1, timeout=10.0)` | Devuelve hasta N tramas de espectro más recientes. |
| `stream_start()` / `stream_stop()` | Reanudar / pausar la transmisión (el identificador permanece abierto). |
| `record_start(output_dir=None, device_name=None)` | Inicia la grabación de un archivo .daq. Devuelve la ruta del archivo. No se admite en DAQ-U/M sin un paquete de calibración AWS (el DAQ-E está exento). |
| `record_stop()` | Detiene la grabación. Devuelve `{path, rows}`. |
| `disconnect()` | Liberar del grupo. No realiza ninguna operación para manejadores conectados pero no propios. |

> **Los perfiles de corrección de nivel máximo (`cap_id`) no son un controlador de «SDK».** `connect_daq_sensor()` / `DAQSensorSession` no exponen ningún parámetro `cap_id` ni método `set_cap`. Selecciona un perfil de corrección de límite de flotaa través de CLI (`chloros-cli daq pool-connect --cap-id …` / `chloros-cli daq pool-set-cap …`) o de las rutas `/api/daq` HTTP del backend (`/api/daq/connect` y `/api/daq/<id>/cap-id` aceptan `cap_id`).

### Descubrimiento: búsqueda de una dirección a la que conectarse

`discover_daq_sensors()` escanea USB / BLE / ETH en busca de sensores que *podrías* abrir. Es el equivalente en DAQ a `discover_lattice_cameras()`, y la única forma de obtener la **dirección MAC BLE de un DAQ-M**; un DAQ-E tiene un nombre de host y un DAQ-U un puerto COM, pero la dirección MAC no aparece impresa en el dispositivo ni figura en el sistema operativo.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               None   {'manufacturer': 'Intel'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Campo | Descripción |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | Puerto COM / MAC BLE / nombre de host — pasar a `connect_daq_sensor` como `port=` / `mac=` / `eth_host=`. |
| `display` | Etiqueta legible para humanos. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E`, o `None` para un puerto que el escáner no puede identificar (los adaptadores serie USB son indistinguibles sin una sonda, por lo que los desconocidos se muestran en lugar de ocultarse). |
| `extra` | Detalles por transporte (nombre anunciado por BLE, fabricante USB, IP/firmware/… de DAQ-E). Se omiten los valores vacíos. |

| Parámetro | Valor por defecto | Descripción |
| --- | --- | --- |
| `transports` | los tres | Secuencia (o cadena CSV) que limita el escaneo. Vale la pena indicarla cuando se sabe lo que se quiere; el BLE es el eslabón más lento. |
| `scan_timeout` | 5 | Ventana de escaneo por transporte en segundos; el backend la limita a un rango de 1 a 20. |
| `timeout` | 60,0 | Límite máximo de «HTTP» para toda la llamada (al igual que en el resto del «SDK»). |
| `auto_start_backend` | `True` | Inicia un backend local si no hay ninguno en ejecución. Nunca se inicia para un `backend_url` remoto. |

> **Los sensores que ya están abiertos en el grupo no aparecen.** Un periférico BLE conectado deja de anunciarse y no se puede sondear un puerto COM abierto, por lo que la detección muestra lo que está *disponible para conectarse*. Es normal obtener un resultado vacío justo después de haber conectado algo; utiliza `list_daq_sensors()` para lo que ya tienes. Los transportes cuyo escaneo no se puede ejecutar (sin bleak ni zeroconf instalados) se omiten en lugar de generar un error, por lo que un equipo sin Bluetooth sigue obteniendo sus respuestas USB y ETH.

### Listado

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Coexistencia con la GUI / CLI

Si la GUI ya tiene un sensor abierto, al llamar a `connect_daq_sensor(port="COM3")` desde Python se devuelve un identificador marcado como `already_connected=True`. La sesión`disconnect()` de la sesión es entonces una operación nula, por lo que tu script SDK no retira el sensor de la GUI al salir del osciloscopio.

### Clases de hardware directo (sin backend)

`daq_sdk` es reexportado por `chloros_sdk`, por lo que también puedes controlar los sensores de extremo a extremo en el proceso sin necesidad del backend:

> **Disponibilidad:**`daq_sdk` se incluye con la instalación de escritorio de Chloros,**pero no** con el paquete de PyPI — `pip install chloros-sdk` te proporciona `lattice_sdk`, pero omite `chloros_sdk.DAQ_AVAILABLE == False`. Comprueba ese indicador antes de utilizar estas clases; en un host que solo utilice pip, controla el sensor mediante [`connect_daq_sensor()`](#daq-sensor-sessions), que no requiere bibliotecas de transporte locales.

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

# Discovery
for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

# Direct DAQ-U
sensor = DAQUSensor(port="COM3")
sensor.connect()
sensor.start_streaming()
# ... use sensor.add_spectrum_callback(...) ...
sensor.stop()
```

Da prioridad a la ruta de conexión inteligente (`connect_daq_sensor`) cuando desees compartir la propiedad con la interfaz gráfica de usuario; utiliza las clases directas para scripts sin interfaz gráfica que tengan la propiedad exclusiva del sensor.

---

## Automatización de proyectos — `ChlorosProject`

Un proyecto guardado de «Chloros» es una carpeta que contiene `cameras.json` + `sensors.json` + `project.json`. `open_project` carga el manifiesto, y `connect_all` conecta todos los dispositivos guardados con su configuración guardada —el mismo estado del hardware que produciría la interfaz gráfica de usuario—.

### Ejemplo mínimo

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Read a DAQ
spectrum = proj.sensors["Sky"].read()

# Trigger every device simultaneously
proj.capture_all("./out")

proj.disconnect_all()
```

O como gestor de contexto:

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### Métodos de `ChlorosProject`

| Método | Descripción |
| --- | --- |
| `connect_all(cameras=True, arrays=True, sensors=True, verbose=False, align=None)` | Detecta y conecta todos los dispositivos guardados. Devuelve un informe de conexión por clase. Utiliza un backend en ejecución cuando hay uno a la escucha en `127.0.0.1:5000`; de lo contrario, vuelve al control directo (sin backend) de dispositivos `lattice_sdk`; nunca crea un backend. |
| `disconnect_all()` | Desconecta todo. |
| `capture_all(output_dir=".")` | Un fotograma de cada cámara + matriz + espectro de cada sensor. |
| `stream(camera, overlays=False, fps=10.0)` | Generador que produce fotogramas BGR `numpy` a partir de una cámara (o matriz) especificada. `overlays=False` es un bucle de captura directa `lattice_sdk` (las matrices generan diccionarios `{serial: frame}`). `overlays=True` se enruta a través de `ChlorosLocal.camera_stream()` → la transmisión MJPEG del backend, con el bloque guardado de la cámara `ui.overlay` pasado como parámetros de consulta. Requiere el modo backend y una **cámara independiente**: una cámara en modo directo genera el error `RuntimeError` (el backend no puede acceder a una cámara propiedad de este proceso) y una matriz genera el error `NotImplementedError` (superpone el compuesto por cámara — transmite un miembro por su nombre). Equivalente de una sola vez: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Ejecuta la alineación en todas las matrices conectadas actualmente. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Ejecuta el proceso de calibración e indexación en las imágenes del proyecto (envuelve `ChlorosLocal.process`; estos cuatro son los **únicos** argumentos clave aceptados — `indices=`, etc. genera un error `TypeError`; establece los índices mediante `ChlorosLocal.configure()`). Construye de forma diferida un `ChlorosLocal()`, que inicia automáticamente un backend. |

Atributos:
- `proj.cameras` — `Dict[str, CameraHandle]` indexado por nombre Y número de serie.
- `proj.arrays` — `Dict[str, ArrayHandle]` indexado por nombre Y array_id.
- `proj.sensors` — `Dict[str, SensorHandle]` con clave de nombre Y slot_id.
- `proj.config` — `project.json["config"]` diccionario.

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**Niveles de procesamiento.** `capture()`, `grab()` y `frame_stream()` utilizan todos el mismo token `processing`
, y la cadena es acumulativa: cada nivel ejecuta todo lo que hay por encima de él:

| Nivel | Salida | Notas |
| --- | --- | --- |
| `raw` | Bayer de 1 canal, nativo del sensor | Sin demosaico. Las superposiciones no están disponibles en este nivel. |
| `debayered` | BGR de 3 canales (**predeterminado**) | Desmosaico bilineal. El único nivel que funciona sin el modo backend. |
| `radiance` | float32, W/m²/sr/nm | Cadena radiométrica completa: desmosaico + desmezclado 3×3 (multiespectral) + DSNU + campo plano + escala NIST, con la exposición × ganancia eliminadas para que los valores sean absolutos. |
| `reflectance` | uint16, 32768 = 1,0 | Radiancia dividida por la irradiancia descendente (ρ = π·L/E). Requiere una lectura DLS/DAQ; véase la nota más abajo. |
| `display` | sRGB de 8 bitsaproximadamente | Representación equivalente a la de la interfaz gráfica: CCM + balance de blancos + gamma a través del perfil de color activo de la cámara. |

Cualquier valor distinto de `debayered` requiere el modo backend; una cámara en modo directo genera
`NotImplementedError`. `reflectance` necesita una lectura descendente válida: el punto final del fotograma introduce
el DAQ agrupado en la ranura DLS de la cámara automáticamente, pero si no hay ningún DAQ vinculado, la cadena rechaza la
salida de reflectancia y marca claramente la degradación en los metadatos devueltos, en lugar de devolver en silencio
un producto de menor calidad.

> **Escala DN de reflectancia — no la codifiques de forma fija.** La reflectancia LATTICE utiliza `32768` = ρ 1,0 y marca
> XMP `Chloros:PixelScale=32768`; la reflectancia «Survey3» utiliza `65535` = ρ 1,0 y no lleva
> etiquetas `Chloros:*`. Lee la etiqueta y divide por ella. Está definida en el dominio uint16, por lo que se mantiene
> `32768` para todos los formatos que se reescalan (TIFF de 16 bits, PNG de 8 bits /JPG, porcentaje de 32 bits): normaliza
> primero el tipo de datos almacenado a uint16 (×257 desde 8bits, ×65535 desde float). La única excepción:
> una captura de origen de 8 bits escrita como TIFF de 8 bits se *recorta*, no se reescala, por lo que ninguna escala la describe
> — Chloros omite por completo `PixelScale` y la tupla de MicaSense en ese caso. Tratar la ausencia de una
> etiqueta en un archivo de reflectancia LATTICE como «sin escala válida», no como un valor por defecto.

> **EXIF se transfiere a la exportación.** `process()` copia el bloque GPS de la captura original
> **y su ExifIFD** en cada producto, por lo que las exportaciones incluyen `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` y `CameraSerialNumber`, así como la
> georreferenciación. `FocalLength` es lo que Pix4D utiliza para calcular la distancia de muestreo al suelo; sin él,
> la reconstrucción recurre a una escala totalmente errónea (en un caso concreto, un emplazamiento de 411 m
> se convirtió en uno de 47,8 km). Se ha evitado deliberadamente utilizar `-all:all`: las etiquetas estructurales de IFD0 alteran
> la salida de LATTICE, y `ExifImageWidth`/`Height` se excluyen porque describen la captura
> original en lugar de la trama exportada.

Subindicadores de la fase de captura (se aplican a los niveles radiométricos — `radiance`, `reflectance`, `display`):

| Indicador | Predeterminado | Significado |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + campo plano + desmezclado 3x3 + escala radiométrica NIST. |
| `apply_white_balance` | `True` | LUT de balance de blancos. Compatible con DLS cuando hay un DAQ vinculado a la cámara. |
| `apply_index` | `False` | Evaluación del índice de vegetación. |
| `index_expression` | `None` | Fórmula de anulación. Si no está vacía, se activa automáticamente el índice. |
| `annotated` | `False` | Superposición de decoraciones de la interfaz gráfica de usuario (zebra/cuadrícula/picos). No disponible para `raw`. |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **El tipo de retorno es `CapturePathMap`, no `Dict[str, str]`.**
> `chloros_sdk.CapturePathMap` es `Dict[str, Union[str, List[str]]]`: un único nivel
> `processing` asigna una ruta a cada número de serie, mientras que una de varios niveles (`"all"`, o una
> lista explícita `levels`) le proporciona la **lista ordenada** de todos los productos guardados para esa
> cámara. Una composición combinada en directo, si se estuviera transmitiendo, llega bajo la clave adicional
> `"combined"` en lugar de bajo un número de serie. El código que asume `str` falla con el
> formato de lista sin que ningún verificador de tipos lo detecte: la anotación indicaba `Dict[str, str]`
> durante un tiempo después de que se lanzara el formato de lista, por lo que existe el alias. Normaliza
> cuando quieras la forma plana:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

### Alineación de matrices

`ArrayHandle` expone toda la superficie de alineación. Los perfiles son solo de sesión por defecto; llama explícitamente a `export_alignment()` para que se mantengan.

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Alineación en el momento de la conexión

`connect_all(align=...)` puede alinear automáticamente cada matriz en el momento de la conexión:

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

Recurre a `project.json["config"]["auto_align_on_connect"]` cuando no se especifica.

### `SensorHandle`

```python
spectrum = proj.sensors["Sky"].read()
# (spectrum_list, is_saturated, integration_time, x, y, z) — matches the
# daq_sdk add_spectrum_callback signature.
```

---

## Hardware directo (sin backend)

Cuando se desee una dependencia nula del backend (CI, robots sin interfaz gráfica, sistemas embebidos), importa directamente `lattice_sdk` y `daq_sdk`; ambos son reexportados por `chloros_sdk`. Precaución con `CAMERA_AVAILABLE` / `DAQ_AVAILABLE`: `lattice_sdk` se encuentra en el paquete PyPI (pero requiere que esté presente el entorno de ejecución Arena SDK), mientras que `daq_sdk` solo se incluye con la instalación de escritorio.

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### Preajustes y el disparador

Tres de los cuatro preajustes funcionan en **modo libre**: la cámara expone de forma continua y, cuando llega un
`capture()`, devuelve el siguiente fotograma. `triggered` es la excepción: activa la
cámara para un flanco de hardware en la línea 2, por lo que no captura nada hasta que se produzca uno.

| Preajuste | Disparador | Úselo cuando |
| --- | --- | --- |
| `default` | funcionamiento libre | uso general |
| `high_speed` | funcionamiento libre | 8 bits, límite de 60 fps, exposición corta |
| `high_quality` | funcionamiento libre | 12 bits, sin límite de fps — la opción habitual para fotografías |
| `triggered` | **activada, línea 2** | la cámara está conectada a un y otro dispositivo la dispara |

Si eliges `triggered` (o configuras tú mismo `trigger_mode="On"`) sin que nada
active la línea 2, todos los `capture()` agotarán el tiempo de espera —correctamente, ya que le has pedido
a la cámara que espere. El «SDK» lo explica cuando ocurre; consulta
[SC_ERR_TIMEOUT durante la captura](#direct-hardware-backend-free).

> **Nota: los mensajes «GVSP probe» / `SC_ERR_TIMEOUT -1011` al conectarse no son errores.**&gt; Al conectarse, el SDK intenta negociar**tramas jumbo** (paquetes GVSP de 9000 bytes) para obtener un mayor rendimiento. En una conexión directa-a-punto (por ejemplo, una dirección `169.254.x.x` local del enlace), la red normalmente no puede transportar tramas jumbo, por lo que esta sonda agota el tiempo de espera y registra líneas como:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> Esta es la **solución alternativa prevista**: el SDK vuelve automáticamente a los paquetes estándar de 1500 bytes y la cámara sigue conectándose con normalidad (las líneas `[chunk-enable …]` que siguen forman parte de la secuencia de conexión habitual). La captura sigue funcionando.
>
> Puedes omitir esta prueba, pero **no es solo un silenciador de registros, sino que desactiva las tramas jumbo**. La cámara responde a los pings «Don&#x27;t-Fragment» solo hasta 1500 bytes, independientemente de lo buena que sea tu red, por lo que la prueba de ping por sí sola nunca puede detectar tramas jumbo; esta sonda es la única que puede hacerlo. Si la desactivas, la cámara seguirá enviando paquetes estándar de 1500 bytes indefinidamente, en cualquier red:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> Solo merece la pena en una red en la que *sabes* que no admite tramas jumbo, donde ahorra aproximadamente un segundo de tiempo de conexión por cámara. Dado que se trata de un cambio real y no meramente estético, el archivo «SDK» ahora lo indica cuando lo utilizas:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **No lo toques a menos que tengas una razón para hacerlo.** Si se deja activado, cada conexión vuelve a medir la red de la que dispones realmente: conéctate a un conmutador compatible con paquetes jumbo y la siguiente conexión detectará los paquetes jumbo por sí sola, sin necesidad de configurar nada ni de reiniciar.
>
> Si *quieres* el rendimiento de los paquetes jumbo, habilita el modo jumbo de extremo a extremo (MTU de la tarjeta de red de 9000 + un conmutador que los admita), o fíjalo con el comando `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` cuando sepas que el enlace lo admite —aunque es preferible utilizar el comando por`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` por comando individual en lugar de configurarlo de forma permanente, ya que un tamaño fijado omite la prueba y deja de adaptarse a la red que tiene delante. **Todos** los dispositivos de la ruta deben admitir paquetes jumbo, incluidos los divisores o inyectores PoE, que suelen ser la razón por la que una configuración que, por lo demás, es compatible con paquetes jumbo, no puede transmitirlos.

> **`SC_ERR_TIMEOUT -1011` durante `capture()` / `grab*()` es un problema diferente: ese sí es un error real.**&gt; La nota anterior solo se refiere a `-1011` registrado por la**sonda de tiempo de conexión**. Si se produce el mismo error a partir de una**captura**, significa que la cámara se ha conectado correctamente, pero no está enviando ninguna imagen:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> La clave está en una cámara cuyo canal de *control* funciona correctamente —la detección funciona, la configuración y las escrituras de `[chunk-enable …]` se realizan con éxito— mientras que *todos* los fotogramas agotan el tiempo de espera.
>
> **La causa habitual es que la cámara está configurada para un disparador de hardware.** Con `trigger_mode="On"` y `trigger_source="Line2"`, la cámara no emite nada en absoluto hasta que llega un flanco eléctrico al cable de sincronización M8. Si no hay ningún cable que active esa línea, cada captura se queda esperando indefinidamente. La cámara no está averiada y la red funciona correctamente: está haciendo exactamente lo que se le ha indicado.
>
> `CameraSettings()` y los presets `default` / `high_speed` / `high_quality` están preconfigurados para funcionamiento autónomo, y una captura que agota el tiempo de espera mientras está activada muestra un mensaje explicativo en lugar de mostrar simplemente «`-1011`». `PRESETS["triggered"]` activa la línea 2, tal y como está diseñado.
>
> Para forzar a cualquier cámara a funcionar en modo libre:
>
> ```python
> settings = PRESETS["high_quality"]
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Si sigue agotando el tiempo de espera con `trigger_mode="Off"`, la cámara realmente no está enviando datos: envíanos el registro y `ip link show`.

#### Perfiles de color (vista previa en directo de RGB) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` selecciona el perfil de color de pantalla para la **vista previa en directo** en las cámaras RGB (las cámaras multiespectrales ignoran este ajuste):

| Perfil | Significado |
| --- | --- |
| `raw` | Omite por completo la cadena radiométrica. |
| `linear` | DSNU + flat + WB, sin CCM, sin gamma. |
| `natural` | Lineal + CCM medido + gamma sRGB, solo con el acabado «barato» (suavizado de crominancia + desaturación de luces) — el ajuste predeterminado realista. |
| `enhanced` | `natural` más el ajuste completo(eliminación de franjas, vibrancia, contraste local CLAHE). Aspecto más rico con aproximadamente **el doble del coste del acabado por fotograma**, por lo que la frecuencia de fotogramas en directo es menor. |
| `custom_temp` | `natural`, pero con el balance de blancos fijado en `custom_cct_k` Kelvin (se ignora el DLS; limitado a 2000–10 000 K en el backend). |

El perfil es un **control de velocidad/aspecto exclusivo de la vista previa en directo**: las capturas guardadas siempre obtienen el acabado completo e intenso independientemente del perfil seleccionado, por lo que elegir `natural` para recuperar tiempo de fotograma no reduce la calidad de lo que se guarda en el disco. Un perfil desconocido aumenta `ValueError`; cuando se puede acceder a un backend de Chloros, el cambio también se envía mediante POST a este para que el siguiente fotograma de vista previa lo refleje (los usuarios de SDK sin backend siguen obteniendo la modificación de la configuración).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Cámaras mono (M3M) y `Calibration`

Una cámara mono **M3M** (`M3M-<lens>-F<wavelength>`) es de banda única: un plano en escala de grises, sin mosaico de Bayer, sin matriz espectral 3×3. `Calibration` la reconoce y expone un indicador `is_mono`. La reflectancia sigue aplicándose como un mapa radiométrico por banda (la desmezcla es la matriz identidad), pero los cálculos multibanda en una sola cámara dan resultados válidos en lugar de devolver valores sin sentido:

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

Para crear un índice de vegetación a partir de hardware monocromo, combina varias cámaras M3M a diferentes longitudes de onda en una pila multibanda alineada (véase [Alineación de matrices](#array-alignment)) y calcula el índice sobre esa pila en lugar de sobre una sola cámara.

Modo directo de DAQ:

```python
from chloros_sdk import (
    DAQUSensor, DAQMSensor, DAQESensor,
    SensorFleet, discover_all, DiscoveredSensor,
    apply_sensor_settings, SensorSettings,
)

for d in discover_all(timeout=3.0):
    print(d)

sensor = DAQUSensor(port="COM3")
sensor.connect()
apply_sensor_settings(sensor, settings={"integration_time_ms": 64, "frame_avg": 20})
sensor.start_streaming()
# ... sensor.add_spectrum_callback(your_callback) ...
sensor.stop()
```

> **Claves aceptadas por `apply_sensor_settings`**— exactamente `integration_time_ms`, `frame_avg`, `ae_enabled`, `sunshine_diffuser_installed` (DAQ-E; obsoleto, sustituido por `cap_id`), `filter_model` (DAQ-M) y `cap_id` (todos los tipos de DAQ; `None`/`""`/`"none"` = sensor sin carcasa, sin corrección de mayúsculas). Las claves desconocidas se**ignoran silenciosamente**; por ejemplo, `{"integration_time": 64}` no hace nada (debe ser `integration_time_ms`). Devuelve `{"applied": [...], "errors": {...}}` y nunca genera un error.

`chloros_sdk` reexporta únicamente la superficie principal utilizada anteriormente. El API público completo de `daq_sdk` (22 nombres) añade lo siguiente; impórtalos directamente desde `daq_sdk`:

```python
from daq_sdk import (
    DAQULogger, DAQMLogger, DAQELogger,     # rotating-file recorders (the ones the GUI uses)
    ConnectResult, FleetRecordResult,       # SensorFleet result types
    discover_all_detailed, build_sensor,    # detailed discovery + build-by-descriptor
    scan_eth_devices, DaqEControl,          # DAQ-E Ethernet scan + control channel
    scan_ble_devices, detect_ble_device, list_ble_devices,   # DAQ-M BLE discovery
    detect_port, list_serial_ports,         # DAQ-U serial-port discovery
    TcpSerial,                              # serial-over-TCP transport shim
)
```

---

## Excepciones

Captura la clase base para gestionar «cualquier cosa que haya salido mal Chloros»:

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

> `ChlorosAuthenticationError` y `ChlorosConfigurationError` se exportan en el nivel superior junto con el resto; también se pueden importar desde `chloros_sdk.exceptions`, tal y como se muestra.

Jerarquía:

```

ChlorosError
├── ChlorosBackendError           (backend failed to start / unreachable)
├── ChlorosConnectionError        (HTTP transport failure)
├── ChlorosLicenseError           (subscription / tier gate)
├── ChlorosAuthenticationError    (login required)
├── ChlorosConfigurationError     (bad configure() / open_project() inputs)
└── ChlorosProcessingError        (pipeline failed)

ChlorosConnectError                (raised by connect_camera / connect_array /
                                    connect_daq_sensor only — derives from
                                    plain Exception, NOT from ChlorosError,
                                    so `except ChlorosError` will not catch it)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## Ejemplos completos

### 1. Procesar una carpeta con una barra de progreso personalizada

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. Matriz LATTICE en tiempo real → Reflectancia + Referencia DAQ

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. Campaña de captura basada en proyectos

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. Flujo de fotogramas multicámara → Canalización NumPy

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. Script de captura directa en hardware sin interfaz gráfica (sin backend)

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. Prueba de capacidades antes de conectar una matriz de 4 cámaras

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["out_width"], height=r["out_height"],
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Receta de captura equivalente (Python puro)

El lenguaje DSL de recetas de CLI tiene un equivalente directo en Python:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply
    # (CameraHandle has no direct apply method; use the underlying lattice_sdk
    #  helper or the backend's /api/camera/<sn>/apply-settings via requests)
    # For most cases just use cam.cam.set_exposure(...) in direct mode or
    # the GUI's saved settings via project.connect_all().

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## Inicio automático del backend

Los puntos de entrada de Smart-Connect — `connect_camera`, `connect_array`, `connect_daq_sensor` y `discover_lattice_cameras` — son clientes «HTTP» ligeros que dan por hecho que un backend está a la escucha en `127.0.0.1:5000` (el «URL» predeterminado de la interfaz de Smart-Connect). Cuando la interfaz gráfica de usuario (GUI) o CLI ya se están ejecutando, uno de ellos ya está en marcha. Sin embargo, si se ejecuta desde un script sin más, es posible que no haya ninguno, por lo que estas funciones **inician automáticamente el binario del backend incluido** (sin ventana, del mismo modo que lo hace `ChlorosLocal`) antes de su primera llamada y, a continuación, esperan hasta `backend_startup_timeout` a que se inicie.

Reglas:

- **Solo se inicia un URL local.** Un `backend_url` que apunte a `localhost` / `127.0.0.1` / `[::1]` es válido; cualquier otro host se considera el equipo de otra persona y nunca se inicia.
- **El backend se deja en ejecución para su reutilización** (igual que en CLI) — nono se produce ningún apagado implícito cuando el script finaliza. Al volver a ejecutar el script, se reutiliza el backend activo.
- **Desactívalo con `auto_start_backend=False`** en cualquiera de esas llamadas (por ejemplo, cuando hayas apuntado a un backend remoto o gestiones tú mismo el ciclo de vida del backend).

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

Si no se puede localizar o iniciar el binario incluido, la llamada posterior a HTTP genera un error procesable y **adaptado a la plataforma** `ChlorosConnectError` en lugar de un simple rastreo de conexión rechazada: en Windows te remite a la aplicación de escritorio o a un comando `chloros-cli`; en Linux (sin interfaz gráfica) te remite a un comando `chloros-cli` o al `.deb`.

---

## Entorno y encabezados

El SDK marca cada llamada al backend HTTP con `X-Chloros-Client: sdk`. El backend aplica las reglas de licencia de SDK / CLI (se requiere iniciar sesión **y** un plan de pago Chloros+), en lugar de la vía gratuita de la interfaz gráfica de usuario. Esto se configura automáticamente en el momento de la importación; no es necesario que hagas nada.

`http://localhost` y `http://127.0.0.1` se detectan como el backend local. Las llamadas a otros hosts (por ejemplo, tu propio servicio de análisis) no se modifican.

Anula el backend URL pasando `backend_url=` (o `api_url=` en `ChlorosLocal`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(Un `backend_url` que no sea de bucle cerrado solo llega a un backend source/dev; los backends incluidos se vinculan únicamente al bucle cerrado; consulta el modo de backend remoto para conocer el patrón del túnel).

---

## Versiones y compatibilidad

- La versión «SDK» se presenta como `chloros_sdk.__version__`.
- El SDK vincula el comportamiento a la versión del backend incluido. La combinación de un backend antiguo SDK con uno más reciente suele funcionar (puntos finales compatibles con versiones posteriores), pero la combinación de un backend más reciente SDK con uno antiguo puede provocar errores `404` en los nuevos puntos finales; actualiza la aplicación de escritorio para que coincida.
- La superficie de conexión «smart-(`connect_camera` / `connect_array` / `connect_daq_sensor`) y el punto final de análisis de red devuelven esquemas estables de JSON; los nuevos campos son adicionales.

---

## Consejos para la resolución de problemas

- **`ChlorosAuthenticationError: Login required`** → Ejecuta `chloros-cli login EMAIL PASSWORD` una vez en este equipo o inicia sesión a través de la aplicación de escritorio Chloros.
- **`ChlorosConnectError: No Chloros backend is running …`** → Las llamadas de Smart-Connect inician automáticamente un backend local, por lo que este mensaje solo aparece cuando no se encuentra o no se puede iniciar el binario incluido (por ejemplo, en un host que solo utiliza pip y no tiene el paquete de escritorio). El mensaje varía según la plataforma: en Windows, abre la aplicación de escritorio o ejecuta cualquier comando `chloros-cli`; en Linux, ejecuta un comando `chloros-cli` (no existe interfaz gráfica) o instala `.deb`. Para un backend remoto, pasa `backend_url=` (y `auto_start_backend=False`).
- **`CAMERA_AVAILABLE == False`** al importar → `lattice_sdk` no se ha podido cargar (normalmente, las DLL de tiempo de ejecución de Arena SDK no están instaladas). La superficie de la cámara nosuperficie sin cámara sigue funcionando.
- **Array Connect devuelve una resolución inferior a la nativa**→ La función «smart-prep» del backend reduce automáticamente el tamaño del fotograma para que se ajuste al cable. Utiliza `analyze_array_network()` para ver por qué y, a continuación, actualiza el enlace, acepta la reducción o pasa `force_tier="slip-emit-and-capture"` para la captura secuencial. La red de seguridad de la reducción**no** cubre la sobresuscripción agregada (`oversubscribed: true`, campos de fps a 0): el exceso de cámaras para el canal no se puede solucionar mediante el agrupamiento en binning o el ROI; reduce el número de cámaras, habilita los tramas jumbo o cambia a una tarjeta de red más rápida (véase [Sobresuscripción](#over-subscription-the-per-cam-floor)).
- **`analyze_array_network()` indica que el anillo de recepción (RX) de la tarjeta de red es muy pequeño (~0,26 MB) / conecta las puertas con «FRAMES WILL DROP»** → El anillo de recepción de la tarjeta de red del host está en su valor predeterminado (a menudo se restablece a 32 tras una actualización del controlador de la tarjeta de red). En un adaptador Realtek USB de 10 GbE, configura `ReceiveBufferLen=256` y `PendingReceives=64` (elevado) y, a continuación, reinicia el backend para que vuelva a leer el anillo. Procedimiento completo: [CLI Referencia → Configuración y ajuste de la NIC del host](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **El host se cuelga al reiniciarse o apagarse; posteriormente, se producen errores WMI `Invalid class` / la tarjeta de red no se activa** → Un controlador USB 10GbE obsoleto provoca `DRIVER_POWER_STATE_FAILURE` (pantalla azul `0x9F`). Actualiza el controlador del adaptador a una versión reciente (≥ 2026) y vuelve aaplique la configuración del anillo de recepción. Consulte [Referencia de CLI → Configuración y ajuste de la NIC del host](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Reflectancia rechazada** → Para obtener la reflectancia en escala absoluta, debe vincularse un DAQ activo a la cámara (o matriz). Vincúlelo a través de la interfaz gráfica de usuario o utilice `processing="radiance"` (W/m²/sr/nm), que no requiere un sensor emparejado.
- **La captura de `smart=True` tarda más de lo esperado** → La convergencia AE depende de la dinámica de la escena; ajuste `exposure_tolerance_pct` o acorte `stability_window_s` si desea un disparador más rápido (menos estable).

---

## Véase también

- [Referencia de CLI](cli-reference.md) — cada subcomando CLI se corresponde con una llamada a SDK.
- [Guía de sensores DAQ](../daq/README.md) — cableado específico de cada sensor, calibración y normas de registro.
- Documentación en línea: `https://mapir.gitbook.io/chloros/api-python-sdk`</id></sn>

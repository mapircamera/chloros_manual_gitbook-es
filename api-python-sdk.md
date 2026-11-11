# API: Python SDK

{% hint style="info" %}
**¿Buscas la API completa?** Esta página es un tutorial práctico. Todas las clases públicas, métodos, firmas exactas y ejemplos que se pueden copiar y pegar se encuentran en la [Referencia del SDK](reference/sdk-reference.md), que está optimizada para asistentes de IA.**¿Trabajas con un asistente de IA?** Pega esta URL en el chat para que disponga de la API completa y actualizada de Chloros 1.3.0:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Cada página de este manual está disponible en formato Markdown sin formato en su slug en minúsculas + `.md`, y el manual completo está indexado en `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

El **Chloros Python SDK** (`chloros-sdk` en PyPI) controla todo lo que la aplicación de escritorio puede hacer desde Python: procesamiento de imágenes por lotes, control en directo de la cámara LATTICE y de la matriz, sesiones de DAQ con sensores de luz y automatización de proyectos guardados. Se trata de una capa ligera sobre el mismo backend local que utilizan la interfaz gráfica de usuario (GUI) y la interfaz de línea de comandos (CLI) (HTTP en `127.0.0.1:5000`), por lo que el comportamiento es idéntico en las tres interfaces.

## Instalación

El SDK viene incluido en el paquete de escritorio de Chloros, que también proporciona el backend de procesamiento y los entornos de ejecución de hardware, y siempre coincide con su versión.

**Paso 1 — Instalar Chloros.** Windows: ejecuta el instalador de escritorio (ruta predeterminada `C:\Program Files\Chloros\`) desde la página [Descargas](download.md). Linux: instala el paquete `.deb` con `sudo apt install ./chloros_1.3.0_<arch>.deb` ([Instalación en Linux](linux/linux-installation.md)).**Paso 2 — Comprueba el SDK** (Python 3.7+). Normalmente no hay que instalar nada:

* **Windows:** el instalador instala el paquete «wheel» del SDK incluido en el Python del sistema (si no hay ningún Python instalado, se salta este paso).
* **Linux:** el `.deb` instala el SDK para el sistema `python3` en `/usr/lib/python3/dist-packages`. El paquete recomienda sus requisitos, `python3-numpy` y `python3-requests`: `apt install` los incluye, mientras que `dpkg -i` no.
* **Un virtualenv** no detecta los paquetes del sistema; instala el archivo «wheel» incluido en él: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` en Linux, o el `chloros_sdk-*.whl` de la carpeta `sdk` de la carpeta de instalación de Windows (`C:\Program Files\Chloros\sdk\` por defecto).

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # Windows: py -c "…"
```

En un equipo que no tenga el paquete Chloros, cuyo SDK controla un backend en otro equipo (véase [Qué se obtiene con una instalación solo con pip](#what-a-pip-only-install-gets)), instálalo desde PyPI: `pip install chloros-sdk`. PyPI se actualiza con cada nueva versión, por lo que siempre coincide con la última versión estable.

**Paso 3 — Inicia sesión una vez en cada máquina:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

Las credenciales se almacenan en caché en `~/.chloros/` (en ambas plataformas). En Windows, también puedes iniciar sesión a través de la pestaña «Usuario» <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> de la aplicación de escritorio. El SDK requiere un plan de pago Chloros+; consulta [Requisitos de licencia](#license-requirement) más abajo.

| Requisito | Detalles |
| --- | --- |
| **Chloros instalado** | Windows: instalador de escritorio; Linux: paquete `.deb` (proporciona el binario del backend y el SDK) |
| **Python** | 3.7 o superior (desarrollado y probado en 3.10) |
| **Sistema operativo** | Windows 10/11 de 64 bits, Ubuntu 22.04 LTS o posterior, o NVIDIA Jetson (JetPack 6) |
| **Licencia** | Cuenta activa de Chloros+, cualquier plan de pago (Copper o superior) |

## El éxito en 60 segundos

Una sola llamada crea un proyecto, importa una carpeta, configura el procesamiento y ejecuta el flujo de trabajo, iniciando automáticamente el backend si aún no se está ejecutando:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)
```

(En Linux, utiliza las rutas de Linux: `/home/user/drone_images/flight001`. El SDK funciona de forma idéntica en ambas plataformas.)

¿Estás procesando una carpeta de capturas de LATTICE? Utiliza el envoltorio compatible con LATTICE: aplica los valores predeterminados adecuados (sin detección del panel de destino, debayer estándar):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — control total del flujo de trabajo

Para cualquier cosa que vaya más allá de una sola línea de comando, utiliza `ChlorosLocal`. Este utiliza un backend que ya está en ejecución (el de la aplicación de escritorio) o inicia el instalado la primera vez que se utiliza (`auto_start_backend=True`; en Windows, desde la carpeta `backend` de la carpeta de instalación de Chloros; `C:\Program Files\Chloros\backend\` por defecto), crea y configura proyectos, supervisa el progreso y devuelve un resumen tras la ejecución.

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

{% hint style="info" %}
Mantén el valor predeterminado `http://127.0.0.1:5000` en lugar de sustituirlo por `localhost` — en Windows: `localhost` se resuelve primero como `::1` y supone un coste de unos 2 segundos por solicitud frente al backend solo IPv4.
{% endhint %}

Úsalo como gestor de contexto. Al salir del bloque, se pone fin al uso del backend por parte de esta instancia, no al propio backend: este sigue en funcionamiento para el siguiente script y se detiene automáticamente tras 30 minutos sin uso. `shutdown_backend()` detiene una instancia iniciada.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

`configure()` acepta estas palabras clave: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` y `custom_settings`. Los valores principales:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (8-bit)"            # 8-bit TIFF; what all-8-bit images get for a deeper format
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

Los parámetros específicos de LATTICE (`input_level`, `radiometric_output` y la familia `array_alignment*`) se documentan con sus tablas de valores completas en la [Referencia del SDK](reference/sdk-reference.md#supported-values).

### Seguimiento del progreso

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Lectura del resumen posterior a la ejecución — y detección de ejecuciones vacías

Al finalizar, `process()` adjunta el resumen de procesamiento del backend como `result["summary"]`. Cada entrada de `summary["hints"]` es una frase completa que explica cualquier aspecto destacable —por ejemplo, por qué una ejecución no ha generado ningún resultado— y cada indicación se vuelve a emitir como un `UserWarning` de Python, de modo que las ejecuciones vacías se autodiagnostican incluso si nunca se revisa el diccionario:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()` no genera un error cuando una ejecución no produce imágenes.** Este es el único aspecto en el que el SDK y la CLI difieren deliberadamente: `chloros-cli process` considera que «se solicitaron productos, pero no se escribió ninguno» como un fallo y sale con un valor distinto de cero, mientras que el SDK vuelve a la normalidad e informa de la situación a través de `summary` / hints. Si tu canalización debe detenerse ante una ejecución vacía, compruébalo tú mismo: inspecciona `summary` (o cuenta los archivos que hay en la carpeta del proyecto) en lugar de confiar en una excepción.
{% endhint %}

## Smart Connect — hardware en tiempo real

Tres ayudantes abren sesiones persistentes en el grupo de hardware del backend —el mismo grupo que utilizan la CLI, otros scripts y las pestañas de cámara de la GUI—, de modo que los scripts del SDK coexisten con ellos sin competir por los puertos serie ni por el ancho de banda de la red. Los tres inician automáticamente un backend local si no hay ninguno en ejecución. El panel DAQ de la interfaz gráfica de usuario no forma parte del grupo: cierra un sensor de luz en la interfaz gráfica de usuario antes de abrirlo desde un script.

### Una sola cámara LATTICE — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

{% hint style="info" %}
**Las rutas relativas se refieren a la ubicación donde se ejecuta el script (modificado en la versión 1.3.0).** Con un backend local, una ruta relativa `output_dir` (incluida la ruta predeterminada `"output"`) — y las demás rutas que abre el backend (`burst_dir`, `profile_path`, `save_path`, `mask_path`) — se resuelven en función del directorio de trabajo de tu script, por lo que `cam.capture("output/")` escribe en `<current folder>/output/`. En la versión 1.2.0, un backend instalado los resolvía en su propia carpeta de instalación y daba error con el mensaje «Acceso denegado». En el caso de un backend remoto, las rutas se envían tal y como se escriben.
{% endhint %}

### Matriz sincronizada — `connect_array`

`connect_array` es el punto de entrada recomendado para montajes multicámara. Ejecuta el mismo flujo de preparación inteligente que la interfaz gráfica de usuario: análisis de red, selección automática del nivel de sincronización, sincronización de tiempo PTP, selección del formato de píxel por cámara, inicialización de AE y activación del disparador GPIO. La **primera serie es la maestra** (la que dispara el pulso de activación del hardware); el resto son esclavas.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Añade `smart=True` a cualquier captura en matriz para esperar a que la exposición automática se estabilice en todas las cámaras antes de disparar.

Como novedad en la versión 1.3.0, `ArraySession` también te proporciona fotogramas sincronizados directamente, sin escribir archivos:

* `arr.grab()` devuelve un fotograma por cámara, todos a partir de un único disparo, como matrices numpy (`{serial: array}`); `.group_id` es el disparador compartido y `.errors` identifica cualquier cámara que se haya quedado fuera.
* `arr.frame_stream(fps=5.0)` genera un conjunto de este tipo por tick, para su procesamiento en tiempo real en tu propio flujo de trabajo.
* `arr.continuous("output/")` ejecuta la captura continua propia del backend (el modo «Continuo» de la interfaz gráfica de usuario) y devuelve un `ContinuousHandle` con `status()`, `wait()` y `stop()`; utilízalo en lugar de un bucle de captura cuando necesites que se guarden todos los disparadores a una alta frecuencia.

Para los modos de captura (Único / Continuo / Intervalo / Más rápido), los grabadores, la captura en ráfaga a vídeo y la alineación de matrices, consulta la [Referencia del SDK](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### Sensor de luz DAQ — `connect_daq_sensor`

Sin argumentos, `connect_daq_sensor()` detecta automáticamente el protocolo de transporte (orden de prioridad: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Cada trama contiene el valor de 135 puntos `spectrum` (W/m²/nm tras la calibración), un indicador `is_saturated` y los valores CIE `x`, `y` y `z`. Para fijar un sensor o un protocolo de transporte específico, se debe pasar una indicación explícita:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Ten en cuenta que los perfiles de corrección de saturación (`cap_id`) **no** son un control del SDK; selecciónalos mediante `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap` en su lugar.

### Proyectos guardados — `open_project`

Un proyecto guardado de Chloros conserva el hardware conectado (`cameras.json` + `sensors.json` junto con `project.json`), y `chloros_sdk.open_project(path)` puede volver a conectar sus cámaras, matrices y sensores de luz de una sola vez y controlar las capturas por nombre de dispositivo. Los sensores de luz se vuelven a conectar a través de un backend de Chloros en ejecución, con su configuración guardada y su capacidad. Consulta [Automatización de proyectos](reference/sdk-reference.md#project-automation--chlorosproject) en la referencia.

## Qué ofrece una instalación «solo pip»

Comprueba los indicadores de disponibilidad a nivel de módulo antes de utilizar superficies de hardware:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

En un host con **solo** `pip install chloros-sdk` y sin el paquete de escritorio de Chloros:

* `ChlorosLocal`, `process_folder` y `process_lattice_capture` **no** funcionan: necesitan el binario de backend que se incluye en el instalador de escritorio.
* Los auxiliares de conexión inteligente (`connect_camera`, `connect_array`, `connect_daq_sensor`) son clientes HTTP puros, por lo que funcionan con un backend en otra máquina — pero los backends incluidos solo se vinculan al bucle de retorno, por lo que debes reenviar el puerto tú mismo (p. ej., `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) y pasar `backend_url="http://127.0.0.1:5000"` junto con `auto_start_backend=False`. Véase [Modo de backend remoto](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* Las clases LATTICE de hardware directo (`LatticeCamera`, `CameraPool`, …) se importan, pero necesitan el tiempo de ejecución del SDK de Arena del paquete de escritorio; sin él, `CAMERA_AVAILABLE` es `False`.
* Los sensores DAQ se controlan únicamente a través de un backend (tunelizado) — `connect_daq_sensor()` — o del `proj.sensors` de un proyecto; el backend abre el sensor en su propia máquina.

## Asistentes de IA — `chloros-mcp`

El SDK incluye `chloros-mcp`, un servidor MCP que permite a Claude Desktop, Claude Code, Cursor y otros clientes MCP controlar Chloros: procesar imágenes y manejar cámaras LATTICE, matrices y sensores de luz DAQ. Sus herramientas abarcan las funciones de este SDK. Se instala junto con el SDK, por lo que la configuración en Claude Code se realiza con una sola línea:

```bash
claude mcp add chloros -- chloros-mcp
```

Otros clientes: [Uso de Chloros con asistentes de IA](ai-assistants.md). Opciones y todas las herramientas: [Servidor MCP (asistentes de IA)](reference/sdk-reference.md#mcp-server-ai-assistants).

## Requisitos de licencia

El acceso al SDK requiere una sesión activa de Chloros+ en cualquier plan de pago — **Copper o superior**(Copper / Bronze / Silver / Gold); el nivel gratuito «Iron» no tiene acceso a SDK ni a CLI. La aplicación de esta norma se realiza**del lado del servidor**: cada solicitud de SDK debe incluir tanto una sesión activa como un plan de pago; de lo contrario, el backend devuelve los códigos `403` / `PLAN_UPGRADE_REQUIRED` (generados como `ChlorosLicenseError` por `ChlorosLocal`, y como `ChlorosConnectError` por los ayudantes `connect_*`). Un usuario que ha cerrado sesión recibe los códigos `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`); volver a ejecutar `chloros-cli login` soluciona el primer caso, pero no el segundo.

El uso sin conexión funciona dentro del periodo de gracia del plan: el nivel se lee desde la caché de validación del servidor (5 minutos) o desde la caché de licencias firmadas y vinculadas al equipo (30 días para los planes mensuales; hasta el vencimiento de la suscripción para los anuales). Cuando vence el periodo de gracia, el plan pasa a ser gratuito y el acceso al SDK se interrumpe hasta que el equipo se conecte al servidor al menos una vez. `chloros-cli status` sigue estando disponible en el nivel gratuito, por lo que el motivo siempre es visible. Consulte [Inicio de sesión en Chloros+](chloros+-login.md).

## Excepciones

Capture la clase base para gestionar «cualquier error de Chloros»:

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

Todas las excepciones del proceso (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) derivan de `ChlorosError`. Una salvedad: `ChlorosConnectError` — solo lo genera `connect_camera` / `connect_array` / `connect_daq_sensor` — se deriva directamente de `Exception`, **no** de `ChlorosError`, por lo que `except ChlorosError` no lo detectará. La jerarquía completa se encuentra en la [Referencia del SDK](reference/sdk-reference.md#exceptions).

## Véase también

* [Referencia del SDK](reference/sdk-reference.md): la superficie completa de la API, optimizada para asistentes de IA.
* [Referencia de la CLI](reference/cli-reference.md): cada subcomando de la CLI se corresponde con una llamada al SDK.
* [Uso de Chloros con asistentes de IA](ai-assistants.md): configuraciones y plantillas de indicaciones de `chloros-mcp`.
* [Descargar](download.md): instaladores para Windows y Linux.

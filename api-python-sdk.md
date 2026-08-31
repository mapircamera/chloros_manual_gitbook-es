# API : Python SDK

{% hint style="info" %}
**¿Buscas la guía completa de API?** Esta página es un tutorial práctico. Todas las clases públicas, métodos, firmas exactas y ejemplos que se pueden copiar y pegar se encuentran en la [Referencia de SDK](reference/sdk-reference.md), que está optimizada para asistentes de IA.**¿Trabajas con un asistente de IA?** Pega este URL en el chat para que disponga del Chloros 1.2.0 completo y actualizado:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Cada página de este manual está disponible en formato Markdown sin formato en su slug en minúsculas + `.md`, y el manual completo está indexado en `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

El **Chloros Python SDK** (`chloros-sdk` en PyPI) controla todas las funciones de la aplicación de escritorio a partir de Python: procesamiento de imágenes por lotes, control en tiempo real de la cámara LATTICE y de la matriz, sesiones de DAQ con sensores de luz y automatización de proyectos guardados. Se trata de una capa ligera sobre el mismo backend local que utilizan la interfaz gráfica de usuario y CLI (HTTP en `127.0.0.1:5000`), por lo que el comportamiento es idéntico en las tres plataformas.

## Instalación

La instalación consta de dos pasos: primero, el paquete de escritorio Chloros (que proporciona el backend de procesamiento y los entornos de ejecución de hardware) y, a continuación, el paquete Python.

**Paso 1: instalar Chloros.** Windows: ejecuta el instalador de escritorio (ruta predeterminada `C:\Program Files\MAPIR\Chloros\`) desde la página [Descargas](download.md). Linux: instala el paquete `.deb` ([Instalación de Linux](linux/linux-installation.md)).**Paso 2 — Instalar SDK** (Python 3.7+):

```bash
pip install chloros-sdk
```

Es posible que ni siquiera necesites pip: cada instalador incluye un wheel SDK correspondiente. El instalador Windows lo instala automáticamente en tu sistema Python; el Linux `.deb` lo coloca en `/usr/lib/chloros/sdk/` y muestra el comando exacto `pip install --user`. PyPI se actualiza en las compilaciones de lanzamiento, por lo que `pip install chloros-sdk` coincide con la última versión estable.

**Paso 3 — Iniciar sesión una vez por equipo:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

Las credenciales se almacenan en caché en `~/.chloros/` (en ambas plataformas). En Windows puedes iniciar sesión de forma equivalente a través de la pestaña «<img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line">» de la aplicación de escritorio. SDK requiere un plan de pago Chloros+; consulta los [Requisitos de licencia](#license-requirement) más abajo.

| Requisito | Detalles |
| --- | --- |
| **Chloros instalado** | Windows: instalador de escritorio; Linux: paquete `.deb` (proporciona el binario de fondo) |
| **Python** | 3.7 o superior (desarrollado/probado en 3.10) |
| **Sistema operativo** | Windows 10/11 de 64 bits, Ubuntu 22.04 LTS o posterior, o NVIDIA Jetson (JetPack 6) |
| **Licencia** | Inicio de sesión activo en Chloros+, cualquier plan de pago (Copper o superior) |

## La victoria en 60 segundos

Una sola llamada crea un proyecto, importa una carpeta, configura el procesamiento y ejecuta el flujo de trabajo, iniciando automáticamente el backend si aún no se está ejecutando:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)
```

(En Linux, utiliza las rutas de Linux: `/home/user/drone_images/flight001`. El SDK funciona de forma idéntica en ambas plataformas.)

¿Estás procesando una carpeta de capturas de LATTICE? Utiliza el envoltorio compatible con LATTICE: aplica los valores predeterminados adecuados (sin detección de panel de destino, debayer estándar):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — control total del flujo de trabajo

Para cualquier tarea que vaya más allá de una sola línea de código, utiliza `ChlorosLocal`. Este ejecuta el backend la primera vez que se utiliza (`auto_start_backend=True`), crea y configura proyectos, supervisa el progreso y devuelve un resumen tras la ejecución.

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
Mantén el valor por defecto `http://127.0.0.1:5000` en lugar de sustituirlo por `localhost`: en Windows, `localhost` se resuelve primero como `::1` y tarda unos 2 segundos por solicitud en el backend solo para IPv4.
{% endhint %}

Úsalo como gestor de contexto para garantizar la limpieza:

```python
import chloros_sdk

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

`configure()` acepta estas palabras clave: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` y `custom_settings`. Los valores principales:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

Los controles específicos de LATTICE (`input_level`, `radiometric_output` y la familia `array_alignment*`) se documentan con sus tablas de valores completas en la [Referencia de SDK](reference/sdk-reference.md#supported-values).

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

Al finalizar, `process()` adjunta el resumen de procesamiento del backend como `result["summary"]`. Cada entrada de `summary["hints"]` es una frase completa que explica cualquier aspecto destacable —por ejemplo, por qué una ejecución no ha generado ningún resultado— y cada indicación se vuelve a emitir como un Python `UserWarning`, de modo que las ejecuciones vacías se autodiagnostican incluso si nunca se revisa el diccionario:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()` no se genera cuando una ejecución no produce imágenes.** Este es el único aspecto en el que SDK y CLI difieren deliberadamente: `chloros-cli process` considera que «se solicitaron productos, pero no se escribió ninguno» como un fallo y sale con un valor distinto de cero, mientras que SDK finaliza normalmente e informa de la situación a través de `summary` / hints. Si tu canalización debe detenerse ante una ejecución vacía, compruébalo tú mismo: inspecciona `summary` (o cuenta los archivos que hay en la carpeta del proyecto) en lugar de confiar en una excepción.
{% endhint %}

## Smart Connect — hardware en tiempo real

Tres ayudantes abren sesiones persistentes en el grupo de hardware del backend —el mismo grupo que utiliza la interfaz gráfica de usuario—, por lo que los scripts SDK coexisten con la aplicación de escritorio sin competir por los puertos serie ni por el ancho de banda de la red. Los tres inician automáticamente un backend local si no hay ninguno en ejecución.

### Cámara LATTICE única — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

### Matriz sincronizada — `connect_array`

`connect_array` es el punto de entrada recomendado para configuraciones multicámara. Ejecuta el mismo flujo de preparación inteligente que la interfaz gráfica de usuario: análisis de red, selección automática del nivel de sincronización, sincronización temporal PTP, selección del formato de píxel por cámara, inicialización de AE y activación del disparador GPIO. La **primera serie es la maestra** (es la que emite el pulso de disparo del hardware); el resto son esclavas.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Añade `smart=True` a cualquier captura en matriz para esperar a que la exposición automática se estabilice en todas las cámaras antes de activar el disparador. Para los modos de captura (Única / Continua / Intervalo / Más rápida), grabadoras, ráfaga a vídeo y alineación de matrices, consulta la [Referencia de SDK](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### Sensor de luz DAQ — `connect_daq_sensor`

Sin argumentos, `connect_daq_sensor()` detecta automáticamente el protocolo de transporte (orden de prioridad: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Cada trama contiene el valor de 135 puntos `spectrum` (W/m²/nm tras la calibración), un indicador `is_saturated` y CIE `x`, `y`, `z`. Para asignar un sensor o un protocolo de transporte específico —la opción más fiable en hosts con múltiples interfaces de red, donde el autodescubrimiento de Ethernet puede pasar por alto un DAQ-E en buen estado en el primer intento—, se debe pasar una indicación explícita:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Ten en cuenta que los perfiles de corrección de mayúsculas (`cap_id`) **no** son un control de tipo SDK; selecciónalos mediante `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap`.

### Proyectos guardados — `open_project`

Un proyecto guardado (Chloros) conserva el hardware conectado (`cameras.json` + `sensors.json` junto con `project.json`), y `chloros_sdk.open_project(path)` puede volver a conectar todo de una vez y controlar las capturas por nombre de dispositivo. Consulta [Automatización de proyectos](reference/sdk-reference.md#project-automation--chlorosproject) en la documentación de referencia.

## Qué se obtiene con una instalación solo con pip

Comprueba los indicadores de disponibilidad a nivel de módulo antes de utilizar superficies de hardware:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)       # True iff daq_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

En un host que tenga **solo** `pip install chloros-sdk` y no disponga del paquete de escritorio Chloros:

* `ChlorosLocal`, `process_folder` y `process_lattice_capture` **no** funcionan: necesitan el binario de backend que se incluye en el instalador de escritorio.
* Los auxiliares de conexión inteligente (`connect_camera`, `connect_array`, `connect_daq_sensor`) son clientes puros de HTTP, por lo que funcionan con un backend en otra máquina; sin embargo, los backends incluidos solo se vinculan al bucle de retorno, por lo que debes reenviar el puerto tú mismo (p. ej., `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) y pasar `backend_url="http://127.0.0.1:5000"` junto con `auto_start_backend=False`. Véase [Modo de backend remoto](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* Las clases LATTICE de hardware directo (`LatticeCamera`, `CameraPool`, …) se pueden importar, pero necesitan el entorno de ejecución Arena SDK del paquete de escritorio; sin él, `CAMERA_AVAILABLE` es `False`.
* `daq_sdk` (las clases DAQ directas) se incluye con la instalación de escritorio, no con el paquete de PyPI, por lo que `DAQ_AVAILABLE` es `False` en un host que solo utilice pip; en su lugar, controla los sensores DAQ mediante `connect_daq_sensor()` contra un backend (tunelizado).

## Requisitos de licencia

El acceso a SDK requiere una cuenta activa de Chloros+ en cualquier plan de pago — **Copper o superior**(Copper / Bronze / Silver / Gold); el nivel gratuito «Iron» no tiene acceso a SDK ni a CLI. La aplicación de esta norma se realiza**del lado del servidor**: cada solicitud SDK debe incluir tanto una sesión activa como un plan de pago; de lo contrario, el backend devuelve `403` / `PLAN_UPGRADE_REQUIRED` (generado como `ChlorosLicenseError` por `ChlorosLocal`, y como `ChlorosConnectError` por los ayudantes `connect_*`). Un usuario que ha cerrado sesión recibe los códigos de error `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`); volver a ejecutar `chloros-cli login` soluciona el primer caso, pero no el segundo.

El uso sin conexión funciona dentro del periodo de gracia del plan: el nivel se lee desde la caché de validación del servidor (5 minutos) o desde la caché de licencias firmadas y vinculadas al equipo (30 días para los planes mensuales; hasta el vencimiento de la suscripción para los anuales). Cuando vence el periodo de gracia, el plan pasa a ser gratuito y el acceso a SDK se interrumpe hasta que el equipo se conecte al servidor al menos una vez. `chloros-cli status` sigue estando disponible en el nivel gratuito, por lo que el motivo siempre es visible. Consulta [Chloros+ Inicio de sesión](chloros+-login.md).

## Excepciones

Captura la clase base para gestionar «cualquier error Chloros»:

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

Todas las excepciones del proceso (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) se derivan de `ChlorosError`. Una salvedad: `ChlorosConnectError` —solo lo genera `connect_camera` / `connect_array` / `connect_daq_sensor` — se deriva del `Exception` simple, **no** del `ChlorosError`, por lo que el `except ChlorosError` no lo detectará. La jerarquía completa se encuentra en la [Referencia de SDK](reference/sdk-reference.md#exceptions).

## Véase también

* [Referencia de SDK](reference/sdk-reference.md): la superficie completa de API, optimizada para asistentes de IA.
* [Referencia de CLI](reference/cli-reference.md): cada subcomando de CLI se corresponde con una llamada a SDK.
* [Descargar](download.md) — instaladores para Windows y Linux.

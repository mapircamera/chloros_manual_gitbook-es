# CLI : Línea de comandos

> **Referencia completa:**[CLI Referencia](reference/cli-reference.md) documenta**cada opción de cada subcomando** y está optimizada para asistentes de IA: pega su URL en tu asistente y pide un comando que funcione: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Consejo para herramientas de IA:** cualquier página de este manual está disponible en formato Markdown sin formato añadiendo `.md` a su URL (p. ej., `https://mapir.gitbook.io/chloros/reference/cli-reference.md`), y `https://mapir.gitbook.io/chloros/llms.txt` indexa todo el manual para su uso con modelos de lenguaje grande (LLM).

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.2.0 build so the version line reads "Chloros CLI 1.2.0" -->


## ¿Qué es «CLI
»?

`chloros-cli` es la interfaz de línea de comandos del mismo motor de procesamiento que utiliza la aplicación de escritorioChloros
. Se trata de un cliente «HTTP
» ligero que se ejecuta sobre el backendChloros
(un servidor local en `127.0.0.1:5000`); la mayoría de los comandos inician el backend automáticamente, por lo que una sola llamada a `chloros-cli process …` es todo lo que necesita un script.

Funciona en **Windows
10/11 (x64)**y**Linux
(x86_64 y NVIDIA Jetson arm64 en JetPack 6)**, en cualquier terminal, sin necesidad de interfaz gráfica de usuario. Comprueba tu instalación con:

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```

Resumen de las familias de comandos:

* **Procesamiento y cuenta** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 idiomas — véase [Idiomas compatibles](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (soloLinux
/Jetson)
* **Hardware en tiempo real** — `lattice` (control de cámara LATTICE, más de 45 subcomandos), `daq pool-*` (sensores de luz DAQ), `time-sync` (PTP)
* **Automatización** — `project` (ejecutar un proyecto guardado deChloros
sin interfaz gráfica, incluyendo recetas de captura en YAML)

Opciones globales que conviene conocer: `--port N` (puerto del backend, por defecto `5000`), `-v/--verbose`, `--restart` (reinicio forzado del backend), `--backend-exe PATH`. Consulta la [Referencia deCLI
](reference/cli-reference.md) para ver la lista completa.

***

## Instalación

CLI
**se incluye en el instalador deChloros** en todas las plataformas; no hay una descarga independiente deCLI
. Descarga el instalador desde la página [Descargas](download.md).

###Windows


El instalador coloca el archivoCLI
en:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

y añade esa carpeta a tu sistema `PATH` — **abre un nuevo terminal**tras la instalación para que se detecte el `PATH` actualizado. El instalador también coloca scripts de inicio (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) en la raíz de instalación, además de un**Chloros
CLI
** acceso directo en el menú de Inicio, cada uno de los cuales abre un terminal con `chloros-cli` listo para usar.

###Linux


Instala el `.deb` correspondiente a tu arquitectura:

```bash
# Linux x86_64
sudo dpkg -i chloros-amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

Esto instala `chloros-cli` hasta `/usr/bin/chloros-cli` (ya en `PATH`) y el backend a `/usr/lib/chloros/chloros-backend`, junto con el tiempo de ejecución de ArenaSDK
necesario para las cámaras LATTICE. Consulta [Instalación deLinux
](linux/linux-installation.md) para obtener más detalles.

### Verificar

```bash
chloros-cli --version    # "Chloros CLI 1.2.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Inicio de sesión y licencias

CLI
(yPython
SDK
) el acceso requiere un **plan de pagoChloros
+**; cualquier nivel de pago lo incluye, pero el nivel gratuito no. El límite mínimo se aplica**del lado del servidor** por el backend, no por el binarioCLI
: una llamada sin haber iniciado sesión se rechaza con el código `401 AUTH_REQUIRED`, y una llamada con sesión iniciada en el plan gratuito con el código `403 PLAN_UPGRADE_REQUIRED`, independientemente de si procede de `chloros-cli`, deSDK
o de un clienteHTTP
creado manualmente. Actualiza en [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).

Inicia sesión **una vez por equipo**:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.2.0 build showing the license tier line -->


{% hint style="warning" %}
**Contraseñas con caracteres especiales**(`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` se altera en el shell; elCLI
lo detecta con un error 401 y vuelve a intentarlo automáticamente, pero las comillas simples evitan el problema por completo).
{% endhint %}

La sesión se almacena en caché en `~/.chloros/user_session.json` y sigue funcionando sin conexión durante el periodo de gracia del plan (30 días para los planes mensuales, hasta su vencimiento para los anuales). `chloros-cli status` funciona incluso sin un plan de pago, por lo que el motivo del rechazo siempre es visible.

{% hint style="danger" %}
**¿Quieres programar tareas sin interfaz gráfica? Inicia sesión primero.**Un comando de generación de procesos en segundo plano (`process`, `status`, `export-status`, …) que se ejecute**sin sesión almacenada en caché**no falla rápidamente, sino que pasa a un indicador interactivo `Email:` / `Password:` en stdin. Por lo tanto, una tarea cron o un paso de CI sin supervisión**se quedará bloqueada a la espera de entrada**. Ejecuta `chloros-cli login EMAIL 'PASSWORD'` una vez en la máquina antes de programar nada.
{% endhint %}

***

## Tu primera ejecución de procesamiento

Dirige `process` a una carpeta de capturas: detecta automáticamenteSurvey3
(`.raw` + `.jpg`), LATTICE (`.tif`/`.tiff`), `.dng` o una combinación de ambos:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

Los flujos de progreso se muestran en tiempo real por cada hilo de la canalización (Detección, Análisis, Procesamiento, Exportación), y una ejecución satisfactoria finaliza indicando cuántos productos de imagen se han guardado (`Image products written: N`).



<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N" summary line -->
### Dónde se guardan los resultados

`process` guarda los archivos en una **carpeta de proyecto**, no en tu carpeta de entrada:

* Sin `-o`: el proyecto se crea en tu carpeta de proyectos predeterminada (compartida con la interfaz gráfica de usuario; gestiónala con `get-project-folder` / `set-project-folder`, opción alternativa `~/Chloros Projects`), y se nombra según `-n/--project-name` o una marca de tiempo (`YYYYMMDD_HHMMSS`) cuando se omite.
* Con `-o PATH`: esa carpeta **es** la carpeta del proyecto. Si ya contiene un `project.json`, se crea una carpeta hermana con el sufijo `_1`/`_2`… en lugar de sobrescribirla.

Dentro del proyecto, los productos se agrupan **por cámara y, a continuación, por formato de archivo**:

```
<project>/
├── project.json
├── calibration_data.json
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

La carpeta de la cámara es `LATT-<sensor>-<lens>-F<filter>` para LATTICE (que coincide con el EXIF de la captura `Model`) y `<model>_<filter>` (p. ej., `Survey3N_RGN`) paraSurvey3
. La carpeta de formatos sigue a `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` o `tiff32` para `TIFF (32-bit, Percent)`.

{% hint style="info" %}
**Cada producto exportado conserva el nombre del archivo ORIGINAL.**Una exportación de Radiance de `capture_..._raw.tif` sigue llamándose `capture_..._raw.tif`; simplemente se encuentra en `tiff32/Radiance_Images/`.**La carpeta identifica el producto, no el nombre del archivo**, así que utiliza un patrón global para el directorio, no para el sufijo `*radiance*`.
{% endhint %}

### Las opciones que realmente utilizarás

| Opción | Valor por defecto | Qué hace |
| --- | --- | --- |
| `-o, --output PATH` | carpeta de proyecto por defecto | Ubicación de la carpeta del proyecto (véase más arriba). |
| `-n, --project-name NAME` | marca de tiempo | Nombre del proyecto. |
| `--format FMT` | `TIFF (16-bit)` | Uno de los siguientes: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)` o `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | ninguno | Índices de vegetación que se van a exportar (véase [Índices de vegetación](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = despixelarización neuronal, más lenta, máxima calidad (Chloros
+, GPU NVIDIA). |
| `--vignette / --no-vignette` | activado | Corrección de viñeteado. |
| `--reflectance / --no-reflectance` | activado | Calibración de reflectancia; para LATTICE, esto también activa o desactiva el producto de reflectancia. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Forzar el punto de entrada del proceso para los archivos TIFF de LATTICE. |

Para todo lo demás —ajuste de la detección de objetivos, PPK, marcadores de exposición, indicadores de alineación de matrices—, consulta la [sección `process` de la Referencia deCLI
](reference/cli-reference.md).

***

## Elección de qué exportar (productos LATTICE)

El procesamiento LATTICE se distribuye a **todos los productos aplicables en una sola pasada**. Los cuatro conmutadores por producto están**activados por defecto**; utiliza el formulario `--no-` para desactivar uno:

| Opción | Producto |
| --- | --- |
| `--debayered` | Desmosaico lineal → `Debayered_Images/` |
| `--preview` | Vista previa en pantalla (balance de blancos + gamma; expansión de colores falsos para multiespectral) → `Preview_Images/` |
| `--radiance` | Radiancia float32, W/m²/sr/nm → `Radiance_Images/` (siempre `tiff32/`) |
| `--reflectance` | reflectancia uint16, compatible con Pix4D → `Reflectance_Calibrated_Images/` |

RGB
Las cámaras principales solo emiten datos sin debayering y una vista previa; la radiancia o reflectancia por banda no tiene sentido para un sensor de banda ancha, por lo que esos conmutadores no tienen ningún efecto en ellas.Survey3
`.raw` ignora los conmutadores y sigue la ruta estándar de reflectancia/objetivo.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (por defecto `auto`) selecciona la referencia de reflectancia: `auto` crea un [objetivo de calibración](calibration-targets.md) dentro del fotograma que supera el control de calidad como referencia absoluta y recurre a la división de luz descendente del sensor de luz del DAQ (ρ = π·L/E) cuando no hay ningún objetivo presente; `target` es estricto (sin sustitución del DAQ); `daq` se basa en los datos del DAQ. Se pueden proporcionar escaneos de objetivos medidos por unidad con `--target-reflectance-dir`.

{% hint style="info" %}
**Lectura de píxeles de reflectancia:**el DN que significa ρ = 1,0 es**por fuente** — Los archivos LATTICE incluyen la etiqueta `Chloros:PixelScale=32768` en el XMP; los archivos «Survey3
» utilizan 65535 (y no incluyen etiquetas `Chloros:*`). Lee la etiqueta y divide por ella en lugar de suponer una constante. Los detalles y el único caso extremo deliberado sin escala se encuentran en la [Referencia deCLI
](reference/cli-reference.md).
{% endhint %}

**El procesamiento siempre comienza desde `raw`.** Los productos derivados (exportaciones de debayering, radiancia o reflectancia) nunca se reintroducen en el proceso; volver a importarlos y procesarlos aplicaría dos veces los cálculos de calibración, por lo queChloros
los omite y lo indica. `--input-level` es la vía de escape prevista para cuando realmente se necesita forzar un punto de entrada.

***

## Cuando falla una ejecución

A partir de la versión 1.2.0, `process` falla de forma evidente en lugar de «tener éxito» sin mostrar ningún resultado:

* Una ejecución que **solicitó productos pero no escribió ninguno**—solo `project.json` y `calibration_data.json`— muestra el mensaje `Processing finished but wrote no image products.` y**termina con un valor distinto de cero**, por lo que los scripts pueden detectarlo. Las causas habituales: la carpeta de entrada no se reconoció como una captura (comprueba la estructura y `--input-level`), o todos los productos solicitados eran inaplicables para esas cámaras (por ejemplo, solicitar radiancia/reflectancia de cámaras que solo capturRGB
).
* Una **ejecución deliberada solo con metadatos** (todos los productos desactivados, sin `--indices`) sigue considerándose un éxito: en este caso, el resultado correcto es una salida de imagen vacía.
* Vuelve a ejecutar la operación con `--verbose` y comprueba el registro del backend en busca de las líneas `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`, que explican los omisiones por cámara.

Códigos de salida: `0`: éxito · `1`: error genérico · `2`: error de argumento · `130`: interrumpido con Ctrl+C.

***

## Índices de vegetación

Ejecuta `--indices` con uno o más nombres de preajustes; cada índice se guarda en su propia carpeta `<INDEX>_Index_Images/`:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI NDRE GNDVI
```

Los 22 nombres predefinidos que acepta `process --indices`:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

{% hint style="warning" %}
**Existen tres listas de índices: no las confundas.**El menú desplegable «Configuración del proyecto» de la interfaz gráfica de usuario (GUI) contiene 27 fórmulas (añade `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI`; estas cinco son exclusivas de la interfaz gráfica y**no** son válidas para `--indices`). El comando `lattice index --preset` (en tiempo real/fuera de línea) utiliza su propia lista independiente de 22 preajustes. Las fórmulas y los cálculos de bandas se describen en [Fórmulas de índices multiespectrales](project-settings/multispectral-index-formulas.md).
{% endhint %}

***

## Sensores de luz DAQ: una breve introducción

La familia `daq pool-*` controla los sensores espectrales DAQ deMAPIR
(DAQ-U a través de USB, DAQ-M a través de BLE, DAQ-E a través de Ethernet) a través del grupo persistente del backend: la interfaz gráfica de usuario (GUI),CLI
ySDK
comparten todos un mismo identificador activo. **`pool-*` es la ruta DAQ compatible en el archivo «CLI
» incluido**; otros subcomandos `daq` a los que se pueda hacer referencia son una superficie interna de «MAPIR
» (solo de origen) y se cierran con un error explícito que remite a `pool-*`.

```bash
# 1. Open a pooled session (pick the line matching your sensor)
chloros-cli daq pool-connect                              # smart-detect
chloros-cli daq pool-connect --port COM3                  # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF      # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local   # DAQ-E by hostname (reliable)

# 2. List pooled sensors and their ids
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 3. Read the latest calibrated spectrum (W/m²/nm)
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 4. Record a calibrated .daq file for 60 s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 5. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

`pool-record` sin `--duration` se ejecuta hasta `pool-record --stop`; el directorio de salida predeterminado es `~/Documents/DAQ Live View/` **en la máquina del backend**. El perfil de corrección de recorte se selecciona en el momento de la conexión (`--cap-id`, valor predeterminado del backend `sunshine_cosine`) y se puede cambiar en tiempo real con `pool-set-cap` — los perfiles de límite y el rango calibrado del sensor se tratan en los capítulos sobre DAQ de este manual.

{% hint style="warning" %}
**DAQ-E en un host con varias tarjetas de red:** la primera detección automática de `pool-connect --eth` tras el arranque puede fallar incluso con un sensor en buen estado. `--eth-host <ip-or-hostname>` es la opción fiable: utilízala siempre que la detección no dé resultados.
{% endhint %}

***

## Cámaras LATTICE, PTP y automatización de proyectos

La familia `lattice` (más de 45 subcomandos) cubre todo el proceso de trabajo con cámaras LATTICE de principio a fin: detección, capturas individuales, matrices sincronizadas persistentes con el flujo de conexión «smart-prep» de la interfaz gráfica de usuario, vista previa en directo en el navegador, alineación, cálculos de índice y diagnóstico de la tarjeta de red del host. Un ejemplo:

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

Además: `chloros-cli time-sync` genera informes sobre el «grandmaster» PTP que ejecuta el hostChloros
(las cámaras LATTICE y los sensores DAQ-E se sincronizan con él para obtener marcas de tiempo entre dispositivos), y `chloros-cli project` abre un proyecto guardado deChloros
y controla sus cámaras, matrices y sensores de forma automatizada —incluidas las recetas de captura YAML programadas mediante scripts—.

Estas tres familias (`lattice`, `project`, `daq pool-*`) son también las únicas que admiten `CHLOROS_BACKEND_URL` para controlar un backend **remoto**; los comandos principales siempre se dirigen a la máquina local.

En los capítulos dedicados a LATTICE de este manual se incluyen tutoriales completos; todas las opciones se encuentran en la [Referencia deCLI
](reference/cli-reference.md).

***

## Solución de problemas: las 5 principales

| Síntoma | Solución |
| --- | --- |
| `Login required`, o una tarea programada se queda bloqueada en un indicador de `Email:` | Ejecuta `chloros-cli login EMAIL 'PASSWORD'` una vez en esta máquina: los comandos sin una sesión almacenada en caché se ejecutan de forma interactiva en lugar de fallar rápidamente. |
| `backend unreachable` | Inicia la aplicación de escritorioChloros
o ejecuta directamente el binario del backend (`chloros-backend`). Si diriges `lattice`/`project`/`daq pool-*` a un backend remoto, comprueba `CHLOROS_BACKEND_URL`. |
| Conexión de la matriz bloqueada: `FRAMES WILL DROP` / `Reduce ROI to enable` | El anillo de recepción de la tarjeta de red del host se ha restablecido a los valores predeterminados: la causa número uno por la que un equipo que antes funcionaba deja de conectarse, normalmente tras una actualización del controlador de la tarjeta de red. Ejecute `chloros-cli lattice network --fix` desde un terminal **con privilegios elevados** (o configure `ReceiveBufferLen=256`, `PendingReceives=64`); consulte la sección *Configuración y ajuste de la tarjeta de red del host* de la guía de referencia. |
| El subcomando `daq` se cierra con el mensaje: «requiere el paquete DAQ completo…» | Es lo esperado en las versiones comercializadas: el paquete «CLI
» compilado solo incluye la familia `daq pool-*`, que abarca la conexión, la transmisión, la grabación y la selección de capturas. Utilice `pool-*` (o `chloros_sdk.connect_daq_sensor()` dePython
). |
| Jetson muestra una advertencia sobre el intercambio antes de carpetas de gran tamaño | Añadir intercambio respaldado por archivos:CLI
muestra los comandos exactos `fallocate`/`swapon` que hay que ejecutar. |

***

## Obtener ayuda

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Todas las opciones, todos los subcomandos:** [CLI
Referencia](reference/cli-reference.md)
* **Equivalente enPython
:** [Python
SDK
](api-python-sdk.md) y la [SDK
Referencia](reference/sdk-reference.md)
* **Soporte:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)

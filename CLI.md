# CLI: Línea de comandos

> **Referencia completa:**[Referencia de CLI](reference/cli-reference.md) documenta**cada opción de cada subcomando** y está optimizada para asistentes de IA: pega su URL en tu asistente y solicita un comando que funcione: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Consejo para herramientas de IA:** cualquier página de este manual está disponible en formato Markdown sin formato añadiendo `.md` a su URL (p. ej., `https://mapir.gitbook.io/chloros/reference/cli-reference.md`), y `https://mapir.gitbook.io/chloros/llms.txt` indexa todo el manual para su uso con modelos de lenguaje grande (LLM).

> **¿Quieres que tu asistente ejecute Chloros por sí mismo?** `chloros-mcp`, un servidor MCP instalado con el Python SDK, funciona con Claude Desktop, Claude Code, Cursor y otros clientes MCP: consulta [Cómo usar Chloros con asistentes de IA](ai-assistants.md).

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.3.0 build so the version line reads "Chloros CLI 1.3.0" -->


## ¿Qué es la CLI?

`chloros-cli` es la interfaz de línea de comandos del mismo motor de procesamiento que utiliza la aplicación de escritorio de Chloros. Se trata de un cliente HTTP ligero que se conecta al backend de Chloros (un servidor local en `127.0.0.1:5000`); la mayoría de los comandos inician el backend automáticamente, por lo que una sola llamada a `chloros-cli process …` es todo lo que necesita un script. A continuación, el backend permanece en ejecución para el siguiente comando: se detiene en `chloros-cli logout` (a menos que la aplicación esté abierta o haya una ejecución o una compilación en curso), en `chloros-cli backend stop`, o por sí solo tras 30 minutos sin ningún cliente, ejecución o dispositivo conectado. `chloros-cli backend status` lo muestra.

Funciona en **Windows 10/11 (x64)**y**Linux (x86_64 y NVIDIA Jetson arm64 en JetPack 6)**, en cualquier terminal, sin necesidad de interfaz gráfica de usuario. Comprueba tu instalación con:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

Resumen de las familias de comandos:

* **Procesamiento y cuenta** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 idiomas — véase [Idiomas compatibles](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (solo Linux/Jetson)
* **Hardware en funcionamiento** — `lattice` (control de cámara LATTICE, más de 45 subcomandos), `daq pool-*` (sensores de luz DAQ), `time-sync` (PTP)
* **Automatización** — `project` (ejecutar un proyecto Chloros guardado sin interfaz gráfica, incluyendo recetas de captura YAML)

Opciones globales que conviene conocer: `--port N` (puerto del backend, por defecto `5000`), `-v/--verbose`, `--restart` (reiniciar el backend de forma forzada), `--backend-exe PATH`. Consulta la [Referencia de la CLI](reference/cli-reference.md) para ver la lista completa.

***

## Instalación

La CLI **se incluye en el instalador de Chloros** en todas las plataformas; no hay una descarga independiente de la CLI. Descarga el instalador desde la página [Descargas](download.md).

### Windows


El instalador coloca la CLI en:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

y añade esa carpeta a tu sistema `PATH` una sola vez (una actualización también elimina las entradas duplicadas que hayan dejado instaladores anteriores); **abre un nuevo terminal**tras la instalación para que se detecte el `PATH` actualizado. El instalador también coloca scripts de inicio (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) en la raíz de instalación, además de un**acceso directo a Chloros CLI** en el menú de Inicio, cada uno de los cuales abre un terminal con `chloros-cli` listo para usar.

### Linux


Instala el `.deb` correspondiente a tu arquitectura con `apt` (no `dpkg -i`, que omite los requisitos recomendados del SDK Python):

```bash
# Linux x86_64
sudo apt install ./chloros_1.3.0_amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
```

Esto instala `chloros-cli` hasta `/usr/bin/chloros-cli` (que ya se encuentra en `PATH`), el backend y los programas de la CLI descomprimidos en `/usr/lib/chloros/backend/` y `/usr/lib/chloros/cli/` (`/usr/lib/chloros/chloros-backend` es un lanzador que inicia el backend), el entorno de ejecución del SDK de Arena que necesita el backend para las cámaras LATTICE y el Python SDK para el sistema `python3`. Consulta [Instalación en Linux](linux/linux-installation.md) para obtener más detalles.

### Verificar

```bash
chloros-cli --version    # "Chloros CLI 1.3.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Inicio de sesión y licencias

El acceso a la CLI (y al SDK Python) requiere un **plan de pago Chloros+**; cualquier nivel de pago lo incluye, pero el nivel gratuito no. El límite mínimo se aplica**del lado del servidor** por el backend, no por el binario de la CLI: una llamada sin haber iniciado sesión se rechaza con el código `401 AUTH_REQUIRED`, y una llamada con sesión iniciada en el nivel gratuito con el código `403 PLAN_UPGRADE_REQUIRED`, independientemente de si procede de `chloros-cli`, del SDK o de un cliente HTTP creado manualmente. Actualiza en [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).

Inicia sesión **una vez por equipo**:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.3.0 build showing the license tier line -->


{% hint style="warning" %}
**Contraseñas con caracteres especiales**(`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` se altera en el shell; la CLI lo detecta con un error 401 y vuelve a intentarlo automáticamente, pero las comillas simples evitan el problema por completo).
{% endhint %}

La sesión se almacena en caché en `~/.chloros/user_session.json` y sigue funcionando sin conexión durante el periodo de gracia del plan (30 días para los planes mensuales, hasta su vencimiento para los anuales). `chloros-cli status` funciona incluso sin un plan de pago, por lo que el motivo del rechazo siempre es visible.

{% hint style="danger" %}
**¿Quieres programar tareas sin interfaz gráfica? Inicia sesión primero.**Un comando de generación de procesos en segundo plano (`process`, `status`, `export-status`, …) que se ejecute**sin sesión almacenada en caché**no falla rápidamente, sino que pasa a un indicador interactivo `Email:` / `Password:` en stdin. Por lo tanto, una tarea cron o un paso de CI sin supervisión**se quedará bloqueada a la espera de entrada**. Ejecuta `chloros-cli login EMAIL 'PASSWORD'` una vez en el equipo antes de programar nada.
{% endhint %}

***

## Tu primera ejecución de procesamiento

Dirige `process` a una carpeta de capturas: detecta automáticamente Survey3 (`.raw` + `.jpg`), LATTICE (`.tif`/`.tiff`), `.dng` o una combinación de ambos:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

Los flujos de progreso se transmiten en directo por cada hilo de la canalización (Detección, Análisis, Procesamiento, Exportación), y una ejecución satisfactoria finaliza informando del número de productos de imagen que se han generado: `Image products written: N`, seguido de `(plus M clamp masks)` cuando la ejecución también ha escrito [máscaras de recorte](output-image-formats.md#clamp-masks) (estas no se cuentan como productos).

<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N (plus M clamp masks)" summary line -->


Si ya se está ejecutando un backend de Chloros (por ejemplo, el de la aplicación de escritorio), `process` lo utiliza y abre su proyecto en él. Se rechaza mientras la aplicación esté abierta en un proyecto que ella misma haya abierto: procesa desde la aplicación o cierra ese proyecto. Si una ejecución anterior aún no ha finalizado, espera hasta 15 segundos antes de rechazar la solicitud. `--restart` fuerza el inicio de un backend nuevo. En Linux, un backend ejecutado como servicio de systemd nunca se reinicia: `process` lo utiliza tal cual, y en los casos en los que, de otro modo, lo reiniciaría (`--restart`, o una importación que no haya encontrado nada), muestra el mensaje `sudo systemctl restart <unit>` y sale de `1`.

### Dónde se guardan los resultados

`process` escribe en una **carpeta de proyecto**, no en tu carpeta de entrada:

* Si no se especifica `-o`: el proyecto se crea en tu carpeta de proyectos predeterminada (compartida con la interfaz gráfica de usuario; gestiónala con `get-project-folder` / `set-project-folder`; si no se ha configurado ninguna, `Documents/MAPIR/Chloros Projects` en tu carpeta de usuario), con el nombre `-n/--project-name` o una marca de tiempo (`YYYYMMDD_HHMMSS`) si se omite.
* Con `-o PATH`: esa carpeta **es** la carpeta del proyecto. Si ya contiene un `project.json`, se creará un archivo hermano con el sufijo `_1`/`_2`… en lugar de sobrescribirlo. Un `PATH` relativo se toma en relación con la carpeta desde la que se ejecuta `chloros-cli`.

{% hint style="info" %}
**Rutas relativas (modificadas en la versión 1.3.0).** Cada ruta que abre el backend — `-o`/`--output` en cualquier comando (incluido el `output` predeterminado de los comandos de captura), `--input`, `--burst-dir`, `--profile`, `--save` y `--mask` — se resuelve en función de la carpeta desde la que se ejecuta `chloros-cli`, por lo que `-o captures/` escribe en `<current folder>/captures/`. En la versión 1.2.0, el backend los resolvía en su propia carpeta de instalación, lo que provocaba un error de «Acceso denegado». En el caso de un backend remoto (`CHLOROS_BACKEND_URL`), las rutas se envían tal y como se han introducido y se resuelven en esa máquina.
{% endhint %}

{% hint style="warning" %}
**Windows: mantén la carpeta del proyecto corta, a menos que las rutas largas estén activadas.** Con las rutas largas de Windows desactivadas (la configuración predeterminada de Windows), ninguna ruta de archivo puede superar los 260 caracteres, y los productos se sitúan hasta unos 130 caracteres por debajo de la carpeta del proyecto. `process` comprueba esto antes de iniciarse: se rechaza una carpeta de proyecto demasiado anida con el mensaje **«La carpeta de proyecto es demasiado anida para este ordenador»**, se indica la carpeta de proyecto más larga que cabe y se explica cómo activar las rutas largas; no se procesa nada. Utiliza una versión más corta de `-o` (o `set-project-folder`), o activa las rutas largas de Windows (la opción del instalador **Habilitar rutas largas de Windows**, marcada por defecto) y reinicia Chloros.
{% endhint %}

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

La carpeta de la cámara es `LATT-<sensor>-<lens>-F<filter>` para LATTICE (que coincide con el EXIF de la captura `Model`) y `<model>_<filter>` (p. ej., `Survey3N_RGN`) para Survey3. La carpeta de formato sigue a `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` o `tiff32` para `TIFF (32-bit, Percent)`.

En el caso de una cámara multiespectral M3C, también puede aparecer una carpeta `clamp_masks/` en una carpeta de formato, junto a las carpetas de productos (por ejemplo, `tiff16/clamp_masks/Reflectance_Calibrated_Images/` y `tiff32/clamp_masks/Radiance_Images/`). Contiene las máscaras por píxel de los productos de radiancia y reflectancia en los que la desmezcla de bandas ha limitado un valor negativo a cero; véase [Máscaras de limitación](output-image-formats.md#clamp-masks). Ninguna carpeta de productos contiene nunca una máscara.

{% hint style="info" %}
**Todos los productos exportados conservan el nombre del archivo ORIGINAL.**Una exportación de radiancia de `capture_..._raw.tif` sigue llamándose `capture_..._raw.tif`; simplemente se encuentra en `tiff32/Radiance_Images/`.**La carpeta identifica el producto, no el nombre del archivo**, así que utiliza un patrón de búsqueda para el directorio, no para el sufijo `*radiance*`.
{% endhint %}

### Las opciones que realmente utilizarás

| Opción | Valor por defecto | Qué hace |
| --- | --- | --- |
| `-o, --output PATH` | carpeta de proyecto por defecto | Ubicación de la carpeta del proyecto (véase más arriba). |
| `-n, --project-name NAME` | marca de tiempo | Nombre del proyecto. |
| `--format FMT` | `TIFF (16-bit)` | Uno de los siguientes: `TIFF (16-bit)`, `TIFF (8-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)` o `JPG (8-bit)`. Chloros no realiza escalado: en todas las imágenes de 8 bits, un formato de mayor profundidad se escribe como `TIFF (8-bit)`, y así se indica una vez. |
| `--indices NAME [NAME ...]` | ninguno | Índices de vegetación que se van a exportar (véase [Índices de vegetación](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = despixelarización neuronal, más lenta, máxima calidad (Chloros+, requiere una GPU con CUDA: sin ella, la ejecución utiliza `standard`; no se admite para capturas LATTICE, que siempre utilizan `standard`). |
| `--vignette / --no-vignette` | activado | Corrección de viñeteado. |
| `--reflectance / --no-reflectance` | activado | Calibración de reflectancia; para LATTICE, esto también sirve para activar o desactivar el producto de reflectancia. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Forzar el punto de entrada del proceso para los archivos TIFF de LATTICE. |

Para todo lo demás —ajuste de la detección de objetivos, PPK, pines de exposición, indicadores de alineación de matrices—, consulta la [sección `process` de la Referencia de la CLI](reference/cli-reference.md).

***

## Elección de qué exportar (productos LATTICE)

El procesamiento de LATTICE se distribuye a **todos los productos aplicables en una sola pasada**. Los cuatro conmutadores por producto están**activados por defecto**; utiliza el formulario `--no-` para desactivar uno:

| Opción | Producto |
| --- | --- |
| `--debayered` | Desmosaico lineal → `Debayered_Images/`: se elimina el desplazamiento oscuro del paquete de calibración y, a continuación, se aplica un demosaico sensible a los bordes, sin corrección de linealidad, campo plano ni desmezcla —el mismo archivo que guarda una captura en su nivel `debayered`—. Se mantiene la orientación nativa del sensor (registrada en XMP). |
| `--preview` | Vista previa en pantalla (balance de blancos + gamma; estiramiento de colores falsos para multiespectral) → `Preview_Images/` |
| `--radiance` | radiancia float32, W/m²/sr/nm → `Radiance_Images/` (siempre `tiff32/`) |
| `--reflectance` | uint16 reflectancia, compatible con Pix4D → `Reflectance_Calibrated_Images/` |

Las cámaras maestras RGB solo emiten datos sin debayering + vista previa; la radiancia o reflectancia por banda no tiene sentido para un sensor de banda ancha, por lo que esos controles no tienen ningún efecto en ellas. Survey3 `.raw` ignora los controles y sigue la ruta estándar de reflectancia/objetivo.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (por defecto `auto`) selecciona la referencia de reflectancia: `auto` establece como referencia absoluta un [objetivo de calibración](calibration-targets.md) dentro del encuadre que haya superado el control de calidad (QA) y, en su defecto, recurre a la división descendente del sensor de luz del DAQ (ρ = π·L/E) cuando no hay ningún objetivo presente (un objetivo que supera el control de calidad con un indicador compite con el DAQ en la puntuación medida); `target` es estricto (sin sustitución por el DAQ); `daq` se rige por el DAQ. Se pueden proporcionar escaneos de objetivos medidos por unidad con `--target-reflectance-dir`.

{% hint style="info" %}
**Lectura de píxeles de reflectancia:**el DN que significa ρ = 1,0 es**por fuente** — Los archivos LATTICE incluyen la etiqueta `Chloros:PixelScale=32768` en el XMP; los archivos Survey3 utilizan 65535 y no incluyen la etiqueta `Chloros:PixelScale` (aunque sí incluyen otras etiquetas de procesamiento `Chloros:*`). Lee la etiqueta y divide por ella en lugar de suponer una constante. Los detalles y el único caso extremo deliberado sin escala se encuentran en la [Referencia de la CLI](reference/cli-reference.md).
{% endhint %}

**El procesamiento siempre comienza desde `raw`.** Los productos derivados (exportaciones de debayering, radiancia o reflectancia) nunca se reintroducen en el proceso; volver a importarlos y procesarlos aplicaría dos veces los cálculos de calibración, por lo que Chloros los omite y lo indica. `--input-level` es la vía de escape prevista para cuando realmente se necesita forzar un punto de entrada.

***

## Cuando falla una ejecución

A partir de la versión 1.2.0, `process` falla de forma evidente en lugar de «tener éxito» sin mostrar ningún resultado:

* Una ejecución que **solicitó productos pero no escribió ninguno**—solo `project.json` y `calibration_data.json`— muestra `Processing finished but wrote no image products.` y**termina con un valor distinto de cero**, por lo que los scripts pueden detectarlo. Las causas habituales: la carpeta de entrada no se reconoció como una captura (comprueba la estructura y `--input-level`), o todos los productos solicitados eran inaplicables para esas cámaras (por ejemplo, solicitar radiancia/reflectancia de cámaras que solo captan RGB).
* Una **ejecución deliberada solo con metadatos** (todos los productos desactivados, sin `--indices`) sigue considerándose un éxito: en ese caso, el resultado correcto es una salida de imagen vacía.
* Vuelve a ejecutar la tarea con `--verbose` y comprueba el registro del backend en busca de las líneas `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`, que explican los omisiones por cámara.
* En Windows, con las rutas largas desactivadas, una carpeta de proyecto demasiado profunda para el límite de 260 caracteres en la ruta se rechaza **antes** de que comience el procesamiento, utilizándose la carpeta de proyecto más larga que quepa (véase [Dónde se guardan los resultados](#where-the-outputs-land)).
* Los comandos de captura (`lattice capture`, `lattice array-capture` y el resto de familias de captura) enumeran todos los archivos que no han podido escribir y salen con el código `1`; las capturas repetidas se detienen cuando la unidad está llena.

Códigos de salida: `0` éxito · `1` fallo genérico · `2` error de argumento · `130` interrumpido por Ctrl+C.

***

## Índices de vegetación

Ejecuta `--indices` con uno o más nombres predefinidos; cada índice se guarda en su propia carpeta `<INDEX>_Index_Images/`:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI GNDVI OSAVI
```

Los 22 nombres predefinidos que acepta `process --indices`:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

Se añade al proyecto un nombre que aún no figura en la lista. Si un nombre no es un valor predefinido o las bandas de las cámaras no pueden generarlo, se omite y se muestra una advertencia indicando dicho nombre. Si la CLI no puede leer la lista de índices actual del proyecto, deja la lista sin cambios y muestra una advertencia en lugar de sustituirla.

{% hint style="warning" %}
**Existen tres listas de índices: no las confundas.**El menú desplegable «Configuración del proyecto» de la interfaz gráfica de usuario (GUI) contiene 27 fórmulas (añade `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI`; estas cinco son exclusivas de la interfaz gráfica y**no** son válidas para `--indices`). El comando `lattice index --preset` (en tiempo real/fuera de línea) utiliza su propia lista independiente de 22 preajustes. Las fórmulas y los cálculos de bandas se describen en [Fórmulas de índices multiespectrales](project-settings/multispectral-index-formulas.md).
{% endhint %}

***

## Sensores de luz DAQ: una breve introducción

La familia `daq pool-*` controla los sensores espectrales DAQ de MAPIR (DAQ-U a través de USB, DAQ-M a través de BLE, DAQ-E a través de Ethernet) mediante el grupo persistente del backend: la GUI, la CLI y el SDK comparten un único identificador activo. **`pool-*` es la ruta DAQ compatible en la CLI incluida**; otros subcomandos `daq` a los que se pueda hacer referencia son una superficie interna de MAPIR solo de origen y salen con un error explícito que te remite a `pool-*`.

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

`pool-record` sin `--duration` se ejecuta hasta `pool-record --stop`; el directorio de salida predeterminado es `~/Documents/DAQ Live View/` **en la máquina del backend**. El perfil de corrección de límite se selecciona en el momento de la conexión (`--cap-id`, valor por defecto del backend `sunshine_cosine`) y se puede cambiar en tiempo real con `pool-set-cap` — los perfiles de límite y el rango calibrado del sensor se tratan en los capítulos sobre DAQ de este manual.

{% hint style="warning" %}
**DAQ-E en un host con varias tarjetas de red:** la primera detección automática de `pool-connect --eth` tras el arranque puede fallar incluso con un sensor en buen estado. `--eth-host <ip-or-hostname>` es la opción fiable: utilízala siempre que la detección no dé resultados.
{% endhint %}

***

## Cámaras LATTICE, PTP y automatización de proyectos

La familia `lattice` (más de 45 subcomandos) cubre el funcionamiento de las cámaras LATTICE de principio a fin: detección, capturas individuales, matrices sincronizadas persistentes con el flujo de conexión «smart-prep» de la interfaz gráfica de usuario, vista previa en directo en el navegador, alineación, cálculos de índice y diagnóstico de la tarjeta de red del host. Un ejemplo:

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

Además: `chloros-cli time-sync` genera informes sobre el «grandmaster» PTP que ejecuta el host Chloros (las cámaras LATTICE y los sensores DAQ-E se sincronizan con él para obtener marcas de tiempo entre dispositivos), y `chloros-cli project` abre un proyecto de Chloros guardado y controla sus cámaras, matrices y sensores de forma autónoma —incluidas las recetas de captura YAML programadas mediante scripts—.

Estas tres familias (`lattice`, `project`, `daq pool-*`) son también las únicas que admiten `CHLOROS_BACKEND_URL` para controlar un backend **remoto**; los comandos principales siempre se dirigen a la máquina local.

En los capítulos dedicados a LATTICE de este manual se incluyen tutoriales completos; todas las opciones se encuentran en la [Referencia de la CLI](reference/cli-reference.md).

***

## Resolución de problemas: los 5 más comunes

| Síntoma | Solución |
| --- | --- |
| `Login required`, o una tarea programada se queda bloqueada en un indicador `Email:` | Ejecuta `chloros-cli login EMAIL 'PASSWORD'` una vez en esta máquina: los comandos sin una sesión almacenada en caché se ejecutan de forma interactiva en lugar de fallar rápidamente. |
| `backend unreachable` | Inicia la aplicación de escritorio de Chloros o inicia tú mismo el backend: `"C:\Program Files\Chloros\backend\chloros-backend.exe"` en Windows (la carpeta `backend` de tu carpeta de instalación; no se encuentra en `PATH`), `chloros-backend` en Linux. `chloros-cli backend status` muestra si hay alguno en ejecución; `lattice`/`project`/`daq pool-*` nunca inician uno. Si los configuras para que apunten a un backend remoto, comprueba `CHLOROS_BACKEND_URL`. |
| Conexión de matriz bloqueada: `FRAMES WILL DROP` / `Reduce ROI to enable` | El anillo de recepción de la tarjeta de red del host se ha restablecido a los valores predeterminados: la causa número uno por la que un equipo que antes funcionaba se niega a conectarse, normalmente tras una actualización del controlador de la tarjeta de red. Ejecuta `chloros-cli lattice network --fix` desde un terminal **con privilegios elevados** (o configura `ReceiveBufferLen=256`, `PendingReceives=64`); consulta la sección *Configuración y ajuste de la tarjeta de red del host* de la referencia. |
| El subcomando `daq` se cierra: «requiere el paquete DAQ completo…» | Utilice los comandos `daq pool-*`, que controlan el mismo sensor a través del backend: conexión, transmisión, grabación, selección de capturas y, en el caso de un DAQ-E, su configuración de red, reinicio y actualizaciones de firmware (o `chloros_sdk.connect_daq_sensor()` desde Python). |
| Jetson muestra una advertencia sobre el espacio de intercambio antes de procesar carpetas grandes | Añade espacio de intercambio respaldado por archivos: la CLI muestra los comandos exactos `fallocate`/`swapon` que hay que ejecutar. |

***

## Obtener ayuda

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Todas las opciones, todos los subcomandos:** [Referencia de la CLI](reference/cli-reference.md)
* **Equivalente en Python:** [Python SDK](api-python-sdk.md) y la [Referencia del SDK](reference/sdk-reference.md)
* **Asistencia técnica:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)

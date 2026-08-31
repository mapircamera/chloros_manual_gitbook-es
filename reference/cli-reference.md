# Chloros CLI Referencia

**Versión:**

1.2.0**Generado:**29/07/2026 19:19 ·**Revisado:** 30/08/2026**Destinatarios:** Optimizado para su uso con modelos de lenguaje grande (LLM); legible para humanos.**Ámbito:** Todos los subcomandos de `chloros-cli` destinados al usuario, con opciones y ejemplos que se pueden copiar y pegar.

Este documento es la referencia completa de la herramienta de línea de comandos `chloros-cli` que se incluye con MAPIR Chloros. Se ha redactado de forma intencionadamente exhaustiva para que un LLM (o una persona) pueda crear cualquier flujo de trabajo compatible a partir de los ejemplos que se muestran a continuación sin necesidad de examinar el código fuente.

Si solo necesitas lo más destacado, ve directamente a:
- [Guía de inicio rápido en cinco minutos](#five-minute-quickstart)
- [Flujo de trabajo de primera conexión de la cámara LATTICE](#lattice-camera-first-connect-workflow)
- [Flujo de trabajo de primera conexión del sensor DAQ](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Modos de captura, grabadoras y reprocesamiento fuera de línea](#capture-modes-recorders--offline-reprocess)

---

## Convenciones

- Todos los comandos llevan el prefijo `chloros-cli`. En Windows, el binario es `chloros-cli.exe`; en Linux /Jetson es `chloros-cli`.
- Los argumentos opcionales se muestran como `--flag`. Los argumentos posicionales obligatorios se muestran sin corchetes.
- Cuando se proporciona un valor por defecto, si se omite el indicador, se utiliza ese valor.
- El CLI es un cliente ligero HTTP que utiliza el backend Chloros (servidor Flask en `127.0.0.1:5000`). La mayoría de los comandos inician automáticamente el backend. `CHLOROS_BACKEND_URL=<url>` redirige las familias de comandos **`lattice`**,**`project`**y**`daq pool-*`** a un backend remoto: los comandos principales (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) fijan deliberadamente `http://127.0.0.1:<port>` y lo ignoran (el literal IPv4 evita la penalización de ~2 s por solicitud de «Windows» en `localhost`→`::1`). Véase [Variables de entorno](#environment-variables).
- Se requiere iniciar sesión con una cuenta Chloros+ para todas las llamadas a SDK / CLI (ejecuta `chloros-cli login` una vez por máquina; almacenado en caché en `~/.chloros/`).
- Los ejemplos utilizan rutas Linux; en Windows, sustituya `/home/user/...` por `C:/Users/.../...`.

---

## Sinopsis de nivel superior

```
chloros-cli [global options] COMMAND [command options]
```

### Opciones globales

| Opción | Descripción |
| --- | --- |
| `--backend-exe PATH` | Anular el ejecutable del backend detectado automáticamente. |
| `--port N` | Puerto del backend HTTP (por defecto: `5000`). |
| `-v, --verbose` | Habilitar salida detallada. |
| `--restart` | Forzar el reinicio del backend (termina cualquier proceso `backend_server.py`). |
| `--version` | Mostrar la versión (`Chloros CLI 1.2.0`). |
| `--help` | Mostrar la ayuda de nivel superior. |

### Índice de comandos

| Comando | Finalidad |
| --- | --- |
| [`process`](#chloros-cli-process) | Procesa de principio a fin una carpeta de capturas de Survey3 o LATTICE. |
| [`login`](#chloros-cli-login) | Autenticar este equipo con una cuenta de Chloros+. |
| [`logout`](#chloros-cli-logout) | Borrar las credenciales almacenadas en caché. |
| [`status`](#chloros-cli-status) | Mostrar el estado actual de la licencia y la autenticación. |
| [`export-status`](#chloros-cli-export-status) | Progreso en tiempo real de la exportación de Thread-4 durante una ejecución de `process`. |
| [`language`](#chloros-cli-language) | Establecer o mostrar el idioma de visualización de CLI (38 idiomas compatibles). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#comandos-de-carpeta-de-proyecto) / [`reset-project-folder`](#comandos-de-carpeta-de-proyecto) | Carpeta de proyecto predeterminada (compartida con la interfaz gráfica de usuario). |
| [`update`](#chloros-cli-update) | Busca e instala actualizaciones de CLI (Linux /Jetson). |
| [`selftest`](#chloros-cli-selftest) | Diagnósticos del sistema + pruebas de funcionamiento básico. |
| [`time-sync`](#chloros-cli-time-sync) | Estado y control del «grandmaster» PTP. |
| [`lattice`](#chloros-cli-lattice) | Control y captura de la cámara LATTICE (más de 45 subcomandos). |
| [`daq`](#chloros-cli-daq) | Control de sensores espectrales DAQ (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Abrir y ejecutar un proyecto guardado de «Chloros» (cámaras + DAQ). |

---

## Instalación

`chloros-cli` se incluye en el instalador de escritorio de Chloros para todas las plataformas compatibles; no hay una descarga independiente de CLI. Al instalar el paquete de la plataforma, se añade `chloros-cli` a tu `PATH` junto con la aplicación de escritorio y el binario de backend que controla.

Últimas descargas: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> El instalador también incluye scripts de inicio (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`) que abren unCLI; se describen en la [Guía del usuario de CLI](../CLI.md) y no se repiten aquí.

### Windows (.exe)

1. Descarga el instalador de Windows desde la página de descargas.
2. Ejecuta `Chloros-Setup-x.y.z.exe` y sigue las instrucciones del asistente. La ruta de instalación predeterminada es `C:\Program Files\Chloros\` (el directorio «CLI» se encuentra en `C:\Program Files\Chloros\cli\`, que el instalador añade a la variable PATH).
3. Abre un nuevo terminal (`cmd.exe`, PowerShell o una terminal deWindows) para que se detecte el `PATH` actualizado.

```powershell
chloros-cli --version
```

El instalador añade automáticamente `chloros-cli.exe` al sistema `PATH` e incluye el entorno de ejecución de Arena SDK necesario para las cámaras LATTICE.

### Linux amd64 (.deb)

Para estaciones de trabajo x86_64 basadas en Ubuntu 22.04 LTS o posterior / Debian.

> **Ubuntu 20.04 no es compatible.** La lista de dependencias del paquete se deriva de
> aquello con lo que el backend se vincula realmente, y eso incluye `libc6 (>= 2.34)`;
> Focal incluye glibc 2.31. `apt` rechaza la instalación en lugar de permitir que falle en
> tiempo de ejecución.

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
```

El archivo .deb instala:
- `chloros-cli` a `/usr/bin/chloros-cli`
- El backend compilado a `/usr/lib/chloros/chloros-backend`
- El tiempo de ejecución de Arena SDK (para cámaras LATTICE)
- Modelos de eliminación de ruido, paquetes de calibración y configuración del canal de actualización

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Misma estructura que el archivo .deb para amd64, con una compilación CUDA optimizada para Jetson Orin / Orin NX / Orin Nano.

### Autenticación única por máquina

Cada plataforma requiere un inicio de sesión único en Chloros+ para que funcionen las llamadas a SDK / CLI:

```bash
chloros-cli login user@example.com 'YourPassword'
```

Las credenciales se almacenan en caché en `~/.chloros/user_session.json`.

### Verificar la instalación

```bash
chloros-cli --version           # prints "Chloros CLI 1.2.0"
chloros-cli selftest            # full 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

> **Chloros+ se requiere una suscripción.**El CLI requiere un plan activo Chloros+.**Copper**es el nivel básico Chloros+; todos los niveles de pago Chloros+ tienen acceso a CLI / SDK; solo el nivel gratuito**Iron** no lo tiene. (Mapa de identificadores de planes: `0`=Iron/gratis, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) Actualiza en [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing).
>
> Este límite mínimo lo impone el backend, no solo el CLI: una solicitud con los indicadores SDK / CLI sin un plan de pago se rechaza con el código `403 PLAN_UPGRADE_REQUIRED`, independientemente de si procede de `chloros-cli`, Python SDK o un cliente HTTP creado manualmente. Un usuario que haya cerrado sesión recibe el código de error `401 AUTH_REQUIRED`. El acceso funciona sin conexión durante el periodo de gracia del plan (30 días para el plan mensual, hasta su vencimiento para el anual) y se interrumpe cuando este vence; `chloros-cli status` sigue funcionando para que el motivo sea visible (es la única ruta SDK / CLI exenta del control de niveles — `GET /api/license-status`).

---

## Guía de inicio rápido en cinco minutos

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI NDRE GNDVI

# 3. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 4. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 5. Read a spectrum from a connected DAQ-U
chloros-cli daq pool-connect --port COM3
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

---

## `chloros-cli process`

Procesa una carpeta de imágenes a través de todo el flujo de trabajo de Chloros (detección de objetivos → calibración → viñeta → reflectancia → exportación del índice).

### Sinopsis

```
chloros-cli process INPUT [OPTIONS]
```

### Argumentos posicionales

| Argumento | Descripción |
| --- | --- |
| `INPUT` | Ruta a la carpeta de entrada que contiene `.raw + .jpg` (Survey3), `.tif/.tiff` (LATTICE) o `.dng`. |

### Opciones comunes

| Opción | Valor predeterminado | Descripción |
| --- | --- | --- |
| `-o, --output PATH` | una nueva carpeta con marca de tiempo en la ruta predeterminada del proyecto (`~/Chloros Projects` a menos que se configure lo contrario) | Carpeta del proyecto que se va a crear o reutilizar. Si la carpeta ya contiene un `project.json`, se creará un `_1`/`_2` en lugar de sobrescribirla. |
| `-n, --project-name NAME` | automático (marca de tiempo) | Nombre del proyecto. |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` utiliza un «Chloros» + «neural debayer»; más lento pero de mayor calidad. |
| `--vignette / --no-vignette` | `--vignette` | Corrección de viñeteado. |
| `--reflectance / --no-reflectance` | `--reflectance` | Calibración de reflectancia (utiliza el panel de referencia si se encuentra; calibración NIST por número de serie para LATTICE). Para LATTICE multiespectral, esto también sirve como opción de activación/desactivación del **producto** de reflectancia; véase [Opciones de exportación por producto](#per-product-export-toggles-lattice-multispectral). |
| `--ppk` | desactivado | Aplica correcciones GNSS PPK a partir de archivos sidecar. |
| `--exposure-pin-1 MODEL` | desactivado | Fija el «pin-1» de un equipo de doble cámara «Survey3». |
| `--exposure-pin-2 MODEL` | desactivado | Fijar el modelo «pin-2». |
| `--recal-interval SECONDS` | 0 | Forzar la ejecución de los cálculos de calibración cada N segundos de tiempo de captura. |
| `--timezone-offset HOURS` | local | Anular la diferencia horaria incorporada en los metadatos de salida. |
| `--format FORMAT` | `TIFF (16-bit)` | Uno de los siguientes: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | ninguno | Índices de vegetación (`NDVI`, `NDRE`, `GNDVI`, `EVI`, `SAVI`, `OSAVI`, `CIG`, …). |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Forzar el punto de entrada del proceso para los archivos LATTICE TIFF (el archivo Survey3.raw no se ve afectado). También la vía de escape que permite que una captura **sin raw** se procese por completo; véase [Cómo es una carpeta de capturas](#how-a-captures-folder-looks-like). |
| `--debayered / --no-debayered` | activado | Emite el producto con debayering lineal (`Debayered_Images`). Véase [Opciones de exportación por producto](#per-product-export-toggles-lattice-multispectral). |
| `--preview / --no-preview` | activado | Emite la vista previa en pantalla (`Preview_Images`): RGB = balance de blancos (iluminante DAQ si está disponible; en su defecto, «gray-world») + gamma; multispec = estiramiento de colores falsos. |
| `--radiance / --no-radiance` | activado | Emite la radiancia en formato float32 (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | Referencia para el producto de reflectancia LATTICE: `auto` = el objetivo dentro del encuadre que ha superado el control de calidad (QA) es la referencia absoluta; se recurre a la radiación descendente de DAQ (ρ = π·L/E) como alternativa; `target` = estricto (sin sustitución por DAQ); `daq` = DAQ como autoridad. Véase [Opciones de exportación por producto](#per-product-export-toggles-lattice-multispectral). |
| `--target-reflectance-dir DIR` | ninguna | Directorio de escaneos de reflectancia del objetivo **medida** por unidad (`<serial>.csv`); en caso de fallo, se recurre a los espectros T3/T4P nominales. |
| `--array-alignment / --no-array-alignment` | activado | Matrices LATTICE: aplicar la alineación del módulo conmódulo a módulo, tal y como figura en el archivo XMP `Chloros:Alignment*` de cada captura, a todos los productos procesados (desbayering / vista previa / radiancia / reflectancia / índice). No realiza ninguna operación en imágenes sin estas etiquetas. |
| `--array-alignment-crop / --no-array-alignment-crop` | recorte | Recorta las exportaciones alineadas a la región de solapamiento común de la matriz para que todos los módulos compartan una misma huella; `--no-…` conserva el lienzo completo del sensor (relleno negro fuera de la fuente). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Remuestreo para la deformación de alineación. `nearest` conserva los DN exactos de la fuente (sin mezcla entre píxeles de los valores radiométricos). |

### Opciones de detección de objetivos

| Indicador | Descripción |
| --- | --- |
| `--min-target-size PIXELS` | Tamaño mínimo del objetivo del panel (px) para el detector. |
| `--target-clustering 0-100` | Sensibilidad de agrupación. |
| `--target / --targets` | Tratar la carpeta de entrada como si contuviera únicamente paneles objetivo (omitir la detección de estudios). |

### Ejemplos

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI NDRE GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a custom output location with a project name
chloros-cli process "C:/input" -o "C:/output" -n "Field_A_2026-05-26"
```

### Opciones de exportación por producto (LATTICE multiespectral)

El procesamiento de LATTICE se distribuye en **todos los productos aplicables en una sola pasada**. Los cuatro controles de cada tipo — `--debayered`, `--preview`, `--radiance`, `--reflectance` — están todos**activados por defecto**; utiliza el formulario `--no-<type>` para desactivar uno. Las cámaras principales de RGB solo emiten imágenes sin debayering + vista previa (sin radiancia/reflectancia por banda), por lo que `--radiance`/`--reflectance` no tienen ningún efecto en ellas. Los conmutadores se ignoran para Survey3 `.raw` (que sigue la ruta estándar de reflectancia/objetivo). *(El antiguo indicador `--radiometric-output {reflectance,radiance,sensor-response}` fue **eliminadoy sustituido por estos conmutadores; ya no existe el nivel `sensor-response`.)*

| Producto | Salida | ¿Se necesita DAQ descendente? |
| --- | --- | --- |
| `--debayered` | Desmosaico lineal (`Debayered_Images`). | No. |
| `--preview` | Vista previa de pantalla (`Preview_Images`): RGB = WB + gamma; multiespectral = estiramiento a falso color. | No. |
| `--radiance` | float32 W/m²/sr/nm procedente de la cadena radiométrica completa (`Radiance_Images`). | N.º |
| `--reflectance` | uint16 reflectancia ρ (`32768` = 1,0), compatible con Pix4D. | **Sí**, a menos que lo ancle un objetivo dentro del fotograma que haya superado el control de calidad (véase más abajo). |

`--reflectance-source` selecciona la referencia de reflectancia:**`auto`**(por defecto) convierte un objetivo dentro del fotograma que haya superado el control de calidad en la**referencia absoluta**—las cadenas de líneas empíricas ancladas al objetivo seen los paneles no utilizados y se aplica el ganador medido — recurriendo a la división descendente del DAQ (ρ = π·L/E) cuando no hay ningún objetivo presente o el control de calidad falla;**`target`**es estricto (sin sustitución del DAQ);**`daq`**opta por el comportamiento en el que prevalece el DAQ. La geometría del objetivo (ArUco / ROI fija / franja) procede de la configuración del objetivo del proyecto; `--target-reflectance-dir DIR` contiene escaneos**medidos** por unidad (`<serial>.csv`) consultados mediante el número de serie o el código QR de la unidad objetivo, con los espectros T3/T4P como alternativa.

La ruta de reflectancia del DAQ resuelve automáticamente el **flujo descendente con marca de tiempo coincidente**a partir de un**`.daq`**(DAQ-U/M/E)**o un `.csv` nativo de DAQ-M**que se encuentre junto a las imágenes. Si no hay un paquete de calibración por cámara o por DAQ almacenado en la caché local, el proceso**lo recupera automáticamentelo recupera de AWS** la primera vez que se utiliza (requiere conexión a Internet una vez; se almacena en caché como `~/.chloros/`).

#### Lectura de píxeles de reflectancia (Pix4D / Metashape / tus propios scripts)

La reflectancia se almacena como un DN entero, y **el valor de DN que corresponde a ρ = 1,0 depende de la cámara de origen**:

| Origen | ρ = 1,0 es | Cómo identificarlo |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (margen hasta ρ 2,0) | El archivo lleva el sello XMP `Chloros:PixelScale=32768`. |
| Survey3 | `65535` (recortado en ρ 1,0) | No hay etiquetas XMP «`Chloros:*`»; esa ausencia *es* la señal. |

**Lee `Chloros:PixelScale` y divide por ese valor** en lugar de suponer una constante. La etiqueta está definida en el dominio uint16, por lo que se mantiene como `32768` en todos los formatos de salida que se reescalan — `TIFF (16-bit)`, `PNG (8-bit)`, `JPG (8-bit)` y `TIFF (32-bit, Percent)` son todos autodescriptivos (normaliza primero el tipo de datos almacenado a uint16: ×257 desde 8 bits, ×65535 desde flotante).

> **Un caso no lleva escala, por diseño.** Cuando una captura de origen de 8 bits (BayerRG8) se escribe como «TIFF» de 8 bits, el proceso *recorta* el rango a 0..255 en lugar de reescalarlo, por lo que todos los valores superiores a ρ≈0,008 se aplana a 255 y el archivo no incluye ninguna escala. Chloros omite deliberadamente tanto `Chloros:PixelScale` como la tupla `MicaSense:RadiometricCalibration` en ese caso, y explica el motivo. **Si la etiqueta no está presente en un archivo de reflectancia LATTICE, no se debe suponer una escala: volver a exportar a 16 bits o 32**** en lugar de dividir píxeles que nunca fueron divisibles.

#### EXIF transferido a la exportación

`process` copia el **bloque GPS y su ExifIFD** de la captura original en cada producto, por lo que una
exportación incluye `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` y
`CameraSerialNumber` junto con la georreferenciación.

**`FocalLength` no es opcional para la fotogrametría.** Pix4D calcula la distancia de muestreo al suelo a partir de
la distancia focal más la altitud; si falta la etiqueta, recurre a una escala totalmente errónea. En un
vuelo de 49, la falta de la etiqueta convirtió un terreno de 411 m × 160 m en uno reconstruido de
47,8 km × 13 km: una ortofoto de 455 MP compuesta en su mayor parte por «nodata», lo que se interpretó como un problema de mosaico y
un problema de BigTIFF antes de que nadie comprobara el GSD. Si tu ortofoto sale con una escala inverosímil,
ejecuta primero `exiftool -FocalLength` sobre el producto exportado.

La copia **no** `-all:all`: las etiquetas estructurales de IFD0 alteran la salida de LATTICE al
copiarlas, y `ExifImageWidth` / `ExifImageHeight` se excluyen porque describen la
*captura original*; de lo contrario, una exportación cuyo tamaño se hubiera modificado en algún momento contendría dimensiones
contradictorias con respecto a su propia trama. El XMP se escribe directamente en lugar de copiarse, ya que ExifTool
descarta las etiquetas XMP de la misma invocación cuando se copia el bloque XMP (lo que eliminaría las etiquetas de calibración de MAPIR
).

### Dónde se guardan los archivos de salida

Los productos se guardan **en la carpeta del proyecto, agrupados por cámara y, a continuación, por formato de archivo**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

La carpeta de la cámara es `LATT-<sensor>-<lens>-F<filter>` para LATTICE (que coincide con el EXIF de la captura
`Model`) y `<model>_<filter>` para Survey3 — dos cámaras que comparten sensor y filtro, pero que difieren
en el objetivo, por lo que se mantienen en directorios separados, ya que el viñeteado, el campo de visión y la distorsión difieren. El formato
de la carpeta sigue el patrón `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` o `tiff32` para
`TIFF (32-bit, Percent)`.

> **Cada producto exportado conserva el nombre del archivo ORIGINAL.** Una exportación de radiancia de
> `capture_…_raw.tif` sigue llamándose `capture_…_raw.tif`; simplemente se encuentra en
> `tiff32/Radiance_Images/`. **La carpeta identifica el producto, no el nombre del archivo**, por lo que la búsqueda global
> de `*radiance*.tif` no encuentra nada; en su lugar, busca la coincidencia en el directorio.

### Grabaciones del sensor de luz — calibradas `.daq` + `.csv`

`process` también gestiona las grabaciones `.daq` de tu carpeta de entrada, y **no**
necesita ninguna imagen para hacerlo: un DAQ-U / DAQ-M / DAQ-E que vuele por sí solo constituye una
captura completa, y una carpeta que contenga únicamente archivos `.daq` es una entrada válida.

Se puede grabar un DAQ **sin** su calibración ; eso es lo que hacen por defecto los grabadores públicos
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`) hacen por defecto: escriben los recuentos sin procesar del sensor y marcan el archivo para que
Chloros recupere la calibración de fábrica de ese sensor **por serie** (primero en la caché local,
luego en la nube de MAPIR) y la aplique. `process` vuelve a escribir el resultado:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

El `.csv` contiene una fila por lectura: marca de tiempo UTC, tiempo de integración, potencia total,
lux escotópico/escotópico en lux, PPFD (y su desglose en azul/verde/rojo), longitud de onda máxima y, a continuación, el
espectro completo en la propia rejilla de longitudes de onda del sensor. El `.daq` se vuelve a importar sin necesidad de
calibrarlo por segunda vez.

Si se realiza correctamente, la ejecución genera el informe `Light-sensor products written: N (calibrated .daq + .csv)`.
Lo que figura entre paréntesis describe lo que se ha escrito realmente, por lo que se lee
`(RAW COUNTS — this sensor has no calibration bundle)` para un sensor sin paquete y
`(N calibrated, M raw counts)` para una carpeta que contenga ambos. Los propios encabezados del backend,
`[DAQ-EXPORT]` y `[RUN-SUMMARY]`, derivan su redacción de la misma manera: ninguno de
los tres puede considerar una exportación sin procesar calibrada.

Una grabación DAQ-U / DAQ-M / DAQ-E cuyo paquete de calibración no se pueda recuperar —ya sea porque estás
desconectado o porque ese sensor no tiene ninguna calibración archivada— se **omite con un motivo** en una
línea `[DAQ-EXPORT]`, y nunca se guarda como un archivo «calibrado» que contenga recuentos sin procesar.
Conéctate a Internet y vuelve a ejecutar el proceso. El motivo es el que el lector ha
establecido realmente para ese archivo (esquema ilegible, ausencia de paquete, un error de escritura), y el resumen de la ejecución
enumera motivos **distintos**: veinte archivos omitidos por una misma causa se registran como una
única causa, no como veinte repeticiones de la misma.

#### Las grabaciones DAQ-A se exportan como recuentos sin procesar

La familia **DAQ-A** es anterior al sistema de paquetes por número de serie y no tiene ningún paquete de calibración
que recuperar; en su lugar, se calibra sobre el terreno frente a un objetivo de reflectancia, por lo que
razón por la que nunca lo ha necesitado. Rechazar esos registros les dejaba sin ninguna forma de
obtener sus datos, por lo que se exportan con un **nombre diferente**:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

Un nombre de archivo diferente en lugar de un indicador dentro del archivo, ya que la información debe conservarse
al reenviarse por correo electrónico como un simple nombre. El encabezado `.csv` indica
`raw spectral sensor counts (NOT irradiance)` y advierte de que los valores son comparables
**dentro de** el archivo —que es exactamente para lo que los utiliza la calibración basada en objetivos— y
no entre sensores. Las columnas fotométricas dependientes de la potencia (potencia total, lux fotópicos y
escotópicos, PPFD) se escriben como **NULL** en lugar de integrarse a partir de recuentos, y el resumen
indica `RAW COUNTS`, por lo que los datos «exportados» en un registro no pueden interpretarse como irradiancia.

Las grabaciones heredadas **v1.01 / v1.02** (un DAQ-A-SD las escribe) no incluyen una época por lectura,
sino solo la hora de escritura del archivo. El comparador de imágenes↔luz descendente sigue rechazándolas —comparar un
fotograma con una hora de escritura daría un error imperceptible—, pero el exportador las lee, y
el CSV muestra `clock=daq_created_on`, de modo que el producto indica en qué reloj se encuentra.

### Notas

- `process` detecta automáticamente si tu carpeta es de tipo «Survey3», «LATTICE» o mixta.
- El progreso se transmite a través de Server-Sent Events; el CLI muestra el progreso en tiempo real por hilo (Detectando, Analizando, Procesando, Exportando).
- Para Linux /Jetson, el CLI comprueba el espacio de intercambio y puede mostrar una advertencia antes de procesar carpetas de gran tamaño. El «debayer» sensible a las texturas también aplica automáticamente un límite de frecuencia de la GPU en los Jetson de bajo consumo (Nano, Orin Nano).
- Si se realiza correctamente, la ejecución informa de cuántos productos de imagen se han guardado (`Image products written: N`).

#### Una ejecución que no guarda imágenes falla

Si has solicitado productos y la ejecución no ha escrito **ninguno** —solo `project.json` y
`calibration_data.json`—, `process` lo interpreta como un error: muestra
`Processing finished but wrote no image products.` y **sale con un valor distinto de cero**, por lo que un script puede
detectarlo. El mensaje indica la carpeta del proyecto y las causas habituales:

- la carpeta de entrada no se reconoció como una captura (comprueba la estructura y `--input-level`), o
- se omitieron todos los productos solicitados por no ser aplicables a esas cámaras (por ejemplo, solicitar
  radiancia/reflectancia de cámaras que solo tienen el canal «RGB»).

Vuelve aejecuta de nuevo con `--verbose` y comprueba el registro del backend en busca de las líneas `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`,
que explican los omisiones por cámara que, de otro modo, no llegarían a la salida de CLI.

Una ejecución deliberada «solo con metadatos» —con todos los productos desactivados y sin `--indices`— sigue siendo un
**éxito**, ya que una salida de imágenes vacía es el resultado correcto en este caso.

Lo mismo ocurre con una **ejecución solo con el sensor de luz**: una carpeta de grabaciones `.daq` no contiene imágenes que exportar
por definición, y la ejecución se evalúa en función del `.daq` calibrado / `.csv` que ha generado en su lugar.

---

## `chloros-cli login`

Autentifica este equipo con una cuenta en la nube de Chloros+. Las credenciales se almacenan de forma segura en `~/.chloros/user_session.json`.

```
chloros-cli login EMAIL PASSWORD
```

### Ejemplos

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (eliminando, o duplicando partes de la contraseña). Ante un error 401, el CLI vuelve a intentarlo automáticamente añadiendo `$$`, y luego con la mitad de la contraseña sin duplicados; si el nuevo intento tiene éxito, te inicia sesión y muestra la sintaxis correcta con comillas simples que debes utilizar la próxima vez.

> **Uso sin interfaz gráfica/mediante scripts: la ausencia de sesión almacenada en caché implica un indicador interactivo, no un fallo rápido.** Cualquier comando que genere un backend (`process`, `status`, `export-status`, `time-sync`, …) que se ejecute sin una licencia o sesión almacenada en caché, mostrará un indicador interactivo de `Email:` / `Password:` en la entrada estándar (stdin) antes de continuar. Por lo tanto, un trabajo desatendido sin sesión almacenada en caché se bloqueará, por lo tanto, a la espera de entrada; ejecute `chloros-cli login EMAIL PASSWORD` una vez por máquina antes de programar trabajos sin interfaz gráfica.

---

## `chloros-cli logout`

Borra la sesión almacenada en caché y fuerza un nuevo inicio de sesión en la siguiente llamada.

```bash
chloros-cli logout
```

---

## `chloros-cli status`

Muestra el nivel de licencia actual (Iron/Copper/Bronze/Silver/Gold), el usuario autenticado y el recuento de dispositivos vinculados.

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

Consulta el progreso en tiempo real de la exportación de Thread-4. Se puede llamar con total seguridad **durante** la ejecución de `process` desde otro shell.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

Establece el idioma de visualización de «CLI» (admite 38 idiomas, incluidos CJK, RTL e indios). Recurre de forma fluida al inglés en consolas antiguas que no pueden mostrar el script.

```
chloros-cli language [LANG_CODE] [--list]
```

### Ejemplos

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## Comandos de la carpeta del proyecto

Estos comandos gestionan la ubicación predeterminada de la carpeta del proyecto (compartida con la interfaz gráfica de usuario).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

---

## `chloros-cli update`

Linux/ Solo para Jetson. Comprueba `version_url` a partir de `/etc/chloros/update.conf` y ofrece descargar e instalar el `.deb` correspondiente.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

En Linux /Jetson, el CLI también realiza una **comprobación automática de actualizaciones en cada inicio** (sin bloqueo, nunca retrasa el comando): lee `/etc/chloros/update.conf`, almacena el resultado en caché durante 1 hora en `~/.chloros/update_cache.json` y muestra `Update available: vX.Y.Z / Run: chloros-cli update` cuando existe una versión más reciente. Se omite silenciosamente ante cualquier error y en Windows.

---

## `chloros-cli selftest`

Ejecuta una prueba básica de 7 pasos: versión, disponibilidad de puertos, inicio del backend, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), presencia del modelo de eliminación de ruido y disponibilidad de CUDA + eliminador de ruido.

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

Estado y control del «grandmaster» de PTP. El host Chloros ejecuta el «grandmaster» de PTP; las cámaras LATTICE y las unidades DAQ-E se sincronizan con él para obtener marcas de tiempo entre dispositivos.

| Subcomando | Descripción |
| --- | --- |
| `status` | Mostrar el estado del «grandmaster», las prioridades BMCA y la identidad del reloj. |
| `peers` | Mostrar una lista de los esclavos detectados a través de Delay_Req (cámaras + sensores DAQ-E). |
| `cameras` | Estado de PTP por cámara (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | Reiniciar el proceso «grandmaster». |
| `set-priority --priority1 N --priority2 N` | Anular las prioridades de BMCA. |

### Ejemplos

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

Control de cámaras LATTICE. Cada subcomando se canaliza a través del backend Chloros; el backend es propietario del grupo de cámaras, por lo que las llamadas posteriores a CLI reutilizan el mismo identificador abierto.

### Opciones comunes (compartidas por la mayoría de los subcomandos)

| Indicador | Descripción |
| --- | --- |
| `-d, --device N` | Índice de la cámara (por defecto: 0). |
| `-s, --serial SN` | Número de serie específico; anula `--device`. |
| `--serials SN1,SN2,…` | Números de serie separados por comas para el funcionamiento con varias cámaras. |
| `--all` | Actuar sobre todas las cámaras detectadas. |
| `--exposure US` | Tiempo de exposición en microsegundos. |
| `--gain DB` | Ganancia en dB. |
| `--pixel-format FMT` | p. ej., `BayerRG8`, `BayerRG12`. |
| `--width N` / `--height N` | Dimensiones de la imagen. |
| `--preset {default,high_quality,high_speed,triggered}` | Aplica un preajuste de configuración. Todos funcionan en modo libre excepto `triggered`, que activa la cámara mediante un evento de hardware en la línea 2; si no hay nada que active esa línea, esperará indefinidamente en lugar de capturar. |
| `-o, --output DIR` | Directorio de salida (por defecto: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | Tamaño del paquete GVSP. `auto` ejecuta sondas ICMP+GVSP; `jumbo` = 9000; `standard` = 1500. |

### Flujo de trabajo de primera conexión de cámaras LATTICE

```bash
# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Auto-shrink frame size to fit the wire
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### Referencia de subcomandos

#### Detección e información

| Subcomando | Finalidad |
| --- | --- |
| `lattice info` | Mostrar una lista de las cámaras conectadas (fabricante, modelo, número de serie, IP, MAC). |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | Analiza el sistema anfitrión para obtener una configuración óptima de la cámara. `--no-discover` omite el descubrimiento de la cámara (más rápido, análisis solo de la tarjeta de red). |
| `lattice network [--fix] [--estimate] [--cameras N]` | Comprueba/corregir la configuración de la tarjeta de red; estimar el ancho de banda y los FPS. |
| `lattice network-analysis --master SN --slaves SN1,SN2,… [--width N] [--height N] [--pixel-format FMT] [--binning N] [--force-tier TIER] [--backend-url URL] [--json]` | Capacidad de red del backend con esquema estable + recomendación de matriz (devuelve `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). `auto_capped_fps` mantiene la resolución solicitada pero limita los fps — lee `recommended.recommended_target_fps` y lo pasa como destino de conexión; lo trata como un éxito, no como un error. |
| `lattice analyze-array [--models M1,M2,…] [--binning N] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--json]` | Análisis hipotético sin abrir cámaras. **`--n-active` es el número total de cámaras en la red, no solo las de este conjunto**— auméntalo cuando las cámaras independientes transmitan simultáneamente, o cuando el presupuesto de la red se calcule en función de una demanda que las subestime (por defecto: `len(--models)`). Siempre muestra el valor agregado `Wire budget:` (MB/s demandados frente al límite máximo seguro contra colisiones) y las líneas `Max cameras:`, y marca `** OVER-SUBSCRIBED**` cuando la matriz sobreasigna el ancho de banda — véase [Fps de la matriz y modelo de ráfagas](#array-fps--burst-model). |
| `lattice gpu` | Mostrar el estado de la GPU. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | Comprueba o actualiza el firmware de la cámara. La selección local de `.fwa` está fijada: el archivo en `firmware/<MODEL_PREFIX>/` que coincide con el `MIN_FIRMWARE_VERSION` de la compilación se instala cuando está presente (la versión más alta solo como alternativa), por lo que una imagen del proveedor más reciente almacenada en el disco permanece inactiva hasta que se actualice ese pin; las versiones más recientes llegan a las unidades a través del manifiesto firmado de AWS, lo cual es preferible cuando son más recientes. |
| `lattice presets [--apply NAME]` | Mostrar o aplicar ajustes preestablecidos de la cámara. |
| `lattice status` | Estado de la cámara en directo. |

#### Captura

| Subcomando | Finalidad |
| --- | --- |
| `lattice capture [--format tiff\|png\|jpg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--force-daq]` | Fotograma único. **Guarda todos los tipos de exportación de forma predeterminada** (`--processing all`); véase [Niveles de exportación de captura](#capture-export-levels-the-all-default). `--levels` guarda un subconjunto explícito (anula a `--processing`); `--force-daq` escribe la lectura DAQ asignada como un `.daq`, incluso en una captura solo en formato sin procesar. `--jpeg-quality` = JPEG calidad 1–100 (por defecto 95). |
| `lattice continuous [--format tiff\|png\|jpg] [--jpeg-quality N] [--queue-depth N]` | Transmite al disco hasta pulsar Ctrl+C. |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Vista previa MJPEG en directo basada en navegador. `--ae-damping` establece la amortiguación de la exposición automática (0,4–100). |

#### Ajuste del sensor

| Subcomando | Finalidad |
| --- | --- |
| `lattice configure [--get N1 N2…] [--set N=V N=V…] [--dump] [--json]` | Leer/escribir en cualquier nodo GenICam. |
| `lattice exposure [--auto] [--auto-once] [--off] [--set US] [--brightness N] [--damping F] [--upper-limit US]` | Exposición y AE. |
| `lattice gain [--auto] [--off] [--set DB]` | Ganancia y ganancia automática. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N] [--binning-mode Sum\|Average]` | Área de interés (ROI) del sensor y agrupación de píxeles. |
| `lattice format [--set FMT] [--list]` | Formato de píxeles. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Disparador de hardware/software. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (sin indicadores = balance de blancos de un solo disparo) | Operaciones de balance de blancos. Solo para cámarasRGB/Bayer; sin efecto (se omite) en la M3M monocromática. |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct K] [--get]` | Canal de color de visualización «RGB». `natural` (por defecto) es el acabado en directo más económico; `enhanced` añade eliminación de franjas + vibrancia + contraste local CLAHE para obtener el aspecto completo de «hub-parity» a un coste de acabado por fotograma de aproximadamente el doble, por lo que la velocidad de fotogramas **en directo** es menor— las capturas guardadas siempre obtienen el acabado completo en cualquier caso. Solo para cámaras RGB/Bayer; se omite en la M3M monocromática. |
| `lattice color [--saturation N] [--contrast N] [--reset] [--get]` | Saturación/contraste de visualización (cámaras con filtro RGB). Se omite en la M3M monocromática. |
| `lattice filter [--set NAME] [--list]` | Establece el modelo de filtro de la cámara (`RGN-IMX265`, `OCN`, `NGB`, …). |
| `lattice power [--sleep]` | Sondear nodos de potencia/térmicos; activar o desactivar el modo de reposo de bajo consumo. |

#### Calibración y sensores

| Subcomando | Finalidad |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Calibrar a partir de un objetivo de reflectancia. |
| `lattice dls [--connect] [--spectrum] [--irradiance] [--mac MAC] [--filter NAME] [--json]` | Comandos integrados para el sensor de luz descendente. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Aplicar corrección de viñeteado a imágenes existentes. |

#### Multicámara (sesiones transitorias)

| Subcomando | Finalidad |
| --- | --- |
| `lattice multi-info` | Mostrar todas las cámaras con funciones de sincronización. |
| `lattice multi-capture [--format FMT] [--jpeg-quality N] [--processing LEVEL]` | Un fotograma sincronizado de cada cámara. Guarda **todos los tipos de exportación por defecto**cuando hay una matriz persistente conectada; la opción transitoria sin matriz**solo se somete a debayering** (ejecuta primero `array-connect` para el resto). |
| `lattice multi-stream [--fps F] [--count N] [--format FMT] [--jpeg-quality N]` | Transmite fotogramas sincronizados (transitoria). |
| `lattice multi-test [--count N]` | Prueba de sincronización de GPIO. |
| `lattice multi-detect [--line LINE] [--json]` | Detección automática del cableado maestro/esclavo de GPIO. |

#### Alineación

| Subcomando | Finalidad |
| --- | --- |
| `lattice align-calibrate [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N]` — además de los controles de detección/coincidencia `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, los controles RANSAC `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, la combinación decombinación de fotogramas `[--averaging mean\|median\|inlier_weighted]`, restricciones geométricas `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, restricción espacial `[--roi X0,Y0,X1,Y1] [--mask PATH]` y anulaciones por esclavo `[--per-cam-override SN:KEY=VALUE]` (repetible) | Calcula el perfil de alineación a partir de cámaras en tiempo real. `--prefilter` se establece por defecto en `gradient` (mapa de bordes; coincide con el alineador de la interfaz gráfica de usuario/matriz — los bordes se mantienen en todas las bandas espectrales). `--matcher flann` ofrece buenos resultados a partir de ~5000 características; `--averaging median` es robusto ante una captura defectuosa; `inlier_weighted` pondera según el recuento de coincidencias; `--lock-scale` proyecta a la rotación más cercana (sin escala); `--lock-axis` pone a cero un componente de traslación; `--mask` se aplica a todas las cámaras (utiliza `--per-cam-override` para ajustes por cámara, p. ej., `--per-cam-override 214701292:method=phase`). `--rms-threshold-px` rechaza guardar una calibración cuyo RMS de reproyección supere el límite. |
| `lattice align-apply --profile PATH [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N]` | Captura un fotograma multibanda alineado. `--bit-depth` se ajusta por defecto a la cámara; `--no-crop` conserva el fotograma completo (rellenando con negro); `--interpolation` (por defecto `linear`) y `--border-mode`/`--border-value` (por defecto `constant`/0) controlan la deformación de la CPU; la ruta de la GPU es bilineal en cualquier caso. |
| `lattice align-stream --profile PATH [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N]` | Fotogramas multibanda alineados con el flujo (mismos controles de warp que `align-apply`). |
| `lattice align-info --profile PATH [--json]` | Mostrar detalles del perfil. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | Cambiar el orden de las capas. |

#### Índice / Cálculos de vegetación

```bash
# Offline: compute NDVI from an aligned multi-band TIFF
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: discover array, calibrate alignment, capture, compute index, in one go
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --save-multiband -o output/
```

Conjunto completo de indicadores: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI / NDRE / EVI / SAVI / GNDVI /…), `--formula EXPR`, `--channel SYM=BAND` (repetible), `--capture-level raw|debayered|radiance|reflectance|unknown` (anula el nivel de captura registrado en el TIFF de origen; por defecto: se lee de los metadatos de TIFF), `--output PATH`, `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. Con `--live`, los controles de deformación de alineación también se aplican a: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`.

> **`--channel`: los símbolos distinguen entre mayúsculas y minúsculas.** El lado del símbolo debe coincidir exactamente con los(los presets utilizan minúsculas; p. ej., NDVI = `red`, `nir` — comprueba `--list-presets`), y la parte de la banda debe coincidir con un nombre de banda de la pila alineada (o ser un índice de banda basado en 0en modo offline). `--channel red=Red_660 --channel nir=NIR_850` funciona; `--channel RED=660` falla con un error `channel_map missing entries`.

#### Conexiones persistentes (Smart-Prep, flujo equivalente a la interfaz gráfica de usuario)

Estos comandos mantienen las cámaras abiertas en el grupo de fondo a lo largo de las invocaciones de CLI.

| Subcomando | Finalidad |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Añade una cámara al grupo (una sola, sin matriz). |
| `lattice cam-disconnect [--serial SN] [--all]` | Liberar. |
| `lattice cam-list` | Mostrar las cámaras del grupo. |
| **`lattice array-connect`**|**Conectar una matriz sincronizada persistente (EL punto de entrada recomendado ).** Ejecuta el flujo completo de preparación inteligente de la interfaz gráfica de usuario. |
| `lattice array-disconnect [--array-id ID] [--all]` | Liberar una matriz. |
| `lattice array-list` | Mostrar una lista de matrices conectadas. |
| `lattice array-status [--array-id ID]` | FPS en tiempo real, PTP y último error. |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--index\|--no-index] [--force-daq] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | Una captura sincronizada de la matriz en directo: única / continua / a intervalos / más rápida. **Por defecto, se utiliza `all`** (un archivo por cada tipo de exportación aplicable por cámara). Las cámaras omitidas (p. ej., RGB excluidas de la radiancia/reflectancia) se indican con `Skipped: SN:<serial> (<reason>)`; la lectura del DAQ utilizada para la reflectancia se guarda junto a ella y se indica con `DAQ: <path>`. Véase [Modos de captura, grabadoras y reprocesamiento fuera de línea](#capture-modes-recorders--reprocesamiento-fuera-de-línea). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only]` | Graba la vista en directo del índice combinado en vídeo/GIF (calidad de monitorización; requiere que la transmisión combinada esté abierta). |
| `lattice array-burst [--duration S] [--max-frames N] [--build] [--products …]` | Ráfaga Bayer sin procesar a alta velocidad de fotogramas (calidad de análisis; reprocesar sin conexión). |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | Reprocesar una ráfaga sin procesar guardada en uno o varios vídeos calibrados. |

##### Opciones de `array-connect`

| Indicador | Predeterminado | Descripción |
| --- | --- | --- |
| `--serials SN1,SN2,…` | Detección automática de todas las cámaras LATTICE (se necesitan ≥2) | La primera serie es la MAESTRA. Si se omite, la detección se limita a los modelos LATTICE (`TRI032*`) y conecta todas ellas. |
| `--line {Line0,Line2,Line3}` | `Line2` | Línea de sincronización GPIO. |
| `--target-fps F` | auto | Frecuencia de disparo del disparador maestro. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Anular el selector de niveles. |
| `--wire-ceiling-mbps MB_PER_S` | detección automática | **El presupuesto de ancho de banda sostenido del host, en MB/s: el valor del que depende toda la asignación de la matriz.** Redúcelo cuando la matriz notifique tramas GVSP corruptas: el valor automático se deriva de la velocidad de enlace anunciada por la tarjeta de red, que sobreestima los adaptadores USB, los carriles PCIe de ancho reducido y las estructuras compartidas con mucho tráfico. Se almacena de forma persistente en el bloque de captura de la matriz del proyecto, por lo que al volver a abrirlo (reopen) o al realizar una reconexión con `CLI` o `SDK`, se restaura. Véase [Estado de la matriz](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | auto | Agrupación por hardware. |
| `--no-recommend` | off | Omite el paso de análisis de red. |
| `--no-ptp` | desactivado | Desactivar PTP (las marcas de tiempo entre cámaras **no** son comparables). |

### Smart-AE / Smart-Capture

Las matrices LATTICE ejecutan AE de forma continua en segundo plano tan pronto como se conectan, pero una escena reciéntarda un momento en converger. `array-capture --smart` es la **opción más cómoda**: espera a que el AE se estabilice en todas las cámaras de la matriz y, a continuación, activa la captura. Úsala cuando cambies de escena a mitad de la sesión.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

La política de estabilización es conservadora por defecto: tiempo de espera de 5 s, ventana de estabilidad de 1,5 s, tolerancia de variación de exposición de ±5 %. Ajústala a través de la configuración de la interfaz de línea de comandos de LATTICE (SDK, `ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`) si necesitas un comportamiento diferente al de la automatización.

### Niveles de exportación de capturas (el valor por defecto de `all`)

A partir de esta versión, `lattice capture`, `lattice multi-capture` y `lattice array-capture` **se establecen por defecto en `--processing all`** — un archivo guardado por tipo de exportación que se aplica a cada cámara, coincidiendo con el comportamiento de «Capturar todo» de la interfaz gráfica de usuario. Los niveles son:

| Nivel | Salida | Se aplica a |
| --- | --- | --- |
| `raw` | Bayer de un solo canal (cámaras monocromáticas: la banda única) directamente desde el sensor. | Todas las cámaras. |
| `debayered` | Desmosaico BGR de 3 canales (cámaras monocromáticas: 1 canal en escala de grises). | Todas las cámaras. |
| `radiance` | float32 W/m²/sr/nm a través de la cadena radiométrica completa. | Solo multiespectral (M3C/M3M) — **se omite para las cámaras con filtro «RGB»**. |
| `reflectance` | uint16 ρ (`32768` = 1,0), compatible con Pix4D. | Solo multiespectral, y **solo cuando hay un DAQ vinculado y la cámara está calibrada**; de lo contrario, se omite. |
| `preview` / `display` | Cadena completa de vista previa en la interfaz gráfica de usuario (CCM + WB + gamma según el perfil de la cámara). `lattice capture` denomina a este `preview`; `array-capture`/`multi-capture` utilizan `display`. | Todas las cámaras. |

Pasa un único nivel para guardar solo ese (`--processing debayered`). Cuando se solicita `all`, los niveles que no se aplican a una leva determinada se omiten (y se indican), sin que se produzca un error: una leva no conectada o sin calibrar sigue recibiendo `raw` / `debayered` / `preview`.

Para cualquier fotograma de reflectancia, la lectura descendente del DAQ que se utiliza realmente se escribe en un **`.daq`** junto a las imágenes (para que la captura pueda volver a procesarse más adelante) y se indica en una línea `DAQ:`.

### Cómo es una carpeta de capturas

Cada tipo de exportación se almacena en su **propia subcarpeta** dentro de `-o`, de modo que una captura de varios niveles nunca mezcla tipos:

```
output/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
└── *.daq          the downwelling reading matched to the capture
```

`<ts>` es la marca de tiempo de la captura y `<serial>` es el número de serie de la cámara, por lo que un grupo sincronizado comparte una
marca de tiempo entre todas las cámaras. **Fíjate en la única asimetría:** el nivel `display` se almacena en una carpeta
denominada `preview/`, mientras que los propios archivos conservan `_display` en el nombre; la carpeta y el sufijo difieren
solo en ese nivel. Los niveles desconocidos recurren a una carpeta con su propio nombre y, si no
no se puede crear, el archivo se guarda en la raíz de salida en lugar de perderse.

**Volver a procesar una carpeta de capturas:**apunta `chloros-cli process` a la**raíz de capturas**
(`output/`). `process` normalmente solo importa la carpeta que se le indique, pero cuando dicha carpeta no contiene
imágenes y sí tiene subcarpetas, desciende automáticamente —de modo que lassubcarpetas de nivel y la
carpeta raíz `.daq` se recogen de una sola vez. Cada nivel de una captura se importa como una única imagen, con
los demás niveles disponibles como modos, en lugar de como una imagen por nivel.

También funciona nombrar directamente una **subcarpeta de nivel** (p.ej. `output/raw/`) también funciona. Al hacerlo, se omite la raíz
`.daq`, por lo que debes copiar o indicar que la lectura del DAQ se incluya al volver a derivar un producto radiométrico
a partir de `raw/`; de lo contrario, la coincidencia de la marca de tiempo no tendrá nada con lo que resolverse.

**El procesamiento siempre comienza a partir de `raw`.** Dentro de cada captura, el fotograma sin procesar es la fuente del proceso;
`debayered`, `radiance`, `reflectance` y `preview` aparecen como modos visualizables, pero nunca se devuelven
a través del canal de procesamiento. Volver a procesar un producto derivado volvería a aplicar el viñeteado, el CCM y
los cálculos de radiancia que ya están integrados en sus píxeles, por lo que Chloros lo rechaza en lugar de
realizar un doble procesamiento. Hay dos consecuencias que conviene conocer:

- Los renderizados `index/` y `composite/` **nunca** se procesan. Son salidas, no capturas;
  un renderizado de LUT «NDVI» no tiene una interpretación significativa de la radiancia.
- Una carpeta de capturas exportada **sin** `raw` (p. ej., `array-capture --processing reflectance`) no tiene
  ninguna fuente válida en el flujo de trabajo. Esas capturas se importan y se muestran con normalidad, pero `process` las omite
  y lo indica así:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Si realmente necesitas pasar un producto derivado —una sesión de concentrador capturada con
  `demosaic` activado, o una carpeta heredada —, `--input-level {raw,debayered,processed}` fuerza el punto de entrada
  y anula la omisión. Esa opción es la vía de escape deliberada; `auto` (el valor por defecto)
  nunca procesa una captura que no tenga datos sin procesar.

### Capturas omitidas en matrices de filtros mixtos

Cuando se mezclan cámaras «RGB» y multiespectrales en una misma matriz, `array-capture --processing radiance` (o `reflectance`) guarda los fotogramas multiespectrales y **omite** las cámaras «RGB» —la radiancia por píxel Bayer no tiene sentido para un sensor de banda ancha. El archivo «CLI» muestra explícitamente cada archivo guardado (con su nivel de exportación), cada «`.daq`» escrito y cada omisión, por lo que el recuento de archivos no resulta sorprendente:

```
  Saved: output/sync_…_SN213800234.tif [reflectance] (SN:213800234, fid:1)
  Saved: output/sync_…_SN214000533.tif [reflectance] (SN:214000533, fid:1)
  Saved: output/sync_…_SN214701288.tif [reflectance] (SN:214701288, fid:1)
  DAQ:   output/sync_…_daq-e-54b5e0.daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

Los tokens de omisión; los tokens de motivo de omisión siguen el patrón `<level>-not-applicable-to-rgb-cam`. La reflectancia también puede omitirse con `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, y con `dls-uncalibrated-band-<nm>` cuando la banda se encuentra en su mayor parte fuera del rango radiométricamente calibrado del sensor de luz del DAQ (~374–974 nm); entre las referencias disponibles, solo la F988, cuya ruta compatible es el flujo de trabajo con panel de reflectancia.

Utiliza `--processing debayered` (o `display`) para incluir todas las cámaras independientemente del tipo de filtro, o el valor predeterminado `all` para obtener todos los niveles aplicables por cámara de una sola vez.

---

## Modos de captura, grabadoras y reprocesamiento fuera de línea

Todos ellos operan en una **matriz persistente** (ejecuta primero `array-connect`). Reflejan el panel de captura de la interfaz gráfica de usuario.

### Modos `array-capture`

`array-capture` es un único comando con cuatro modos de obturación, además de un conjunto de opciones de exportación:

| Modo | Indicador | Comportamiento |
| --- | --- | --- |
| **Único** *(por defecto)* | (ninguno) | Un grupo de captura sincronizado y, a continuación, salida. |
| **Continuo** | `--continuous` | Pasadas consecutivas hasta `Ctrl+C`, `--count N` o `--duration S`. |
| **Intervalo** | `--interval S` | Una pasada cada `S` segundos (medidos desde el inicio de cada pasada), con los mismos límites. |
| **Más rápido** | `--fastest` | Solo datos sin procesar + la lectura DAQ asignada + el compuesto de índice combinado; omite los cálculos de radiancia/reflectancia/visualización para que el fotograma se procese rápidamente. Implica `--processing raw --force-daq`. Volver a procesar posteriormente el `.daq` guardado para obtener productos calibrados. |

Opciones de exportación (se pueden combinar con cualquier modo; todas comparten la interfaz gráfica de usuario y el punto final SDK):

| Indicador | Efecto |
| --- | --- |
| `--processing LEVEL` | Nivel de exportación único, o `all` (por defecto). |
| `--levels L1,L2,…` | Subconjunto explícito de tipos de exportación (p. ej., `raw,radiance,reflectance`); **anula `--processing`**. |
| `--aligned` / `--no-aligned` | Ajusta la exportación no sin procesar de cada miembro al [perfil de alineación](#alignment) de la matriz (co-registrada). Los datos sin procesar permanecen sin ajustar, pero llevan la transformación en los metadatos. Recurre a la opción no alineada (con una advertencia) si la matriz no tiene perfil. |
| `--index` / `--no-index` | Guardar u omitir la superposición del índice de vegetación por cámara cuando esté configurada. Por defecto: renderizarla. |
| `--force-daq` | Guardar la lectura DAQ/DLS asignada como un archivo sidecar `.daq`, incluso cuando ningún nivel seleccionado lo requiera (p. ej., una captura solo de datos sin procesar), para que los fotogramas puedan volver a procesarse en reflectancia/índice fuera de línea. |
| `--smart` | Esperar a que el AE se estabilice en todas las cámaras antes de disparar (véase [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | Compresión de píxeles «TIFF». `deflate` (por defecto) = zlib L1 sin pérdidas + predictor horizontal, ~4,1 MB por fotograma a resolución completa; `none` = sin comprimir, escritura ~5 veces más rápida a ~6,3 MB por fotograma — utilízalo para obtener la máxima velocidad sostenida cuando el disco lo permita. Ambos son sin pérdidas y se leen de forma idéntica al importarlos. |

> **Escritura única TIFF + el modelo de velocidad sostenida.**Las capturas se escriben en**una sola**pasada en un archivo TIFF que contiene píxeles + XMP + IFD0 Marca/Modelo (medido en Mono12 a resolución completa: 36 ms comprimido / 6,5 ms sin comprimir, frente a unos 148 ms del antiguo método de «escribir primero y reescribir con ExifTool»); la única tarea pendiente de ExifTool (pulido del sub-IFD EXIF) se ejecuta en un proceso de fondo asíncrono, y un fotograma queda completo y listo para importarse aunque ese proceso nunca se ejecute. Ten en cuenta que la compresión DEFLATE retiene el GIL de Python, por lo que las escrituras comprimidas**no**se paralelizan entre los hilos de escritura de cada cámara: la captura sostenida a resolución completa con 8 cámaras a la velocidad del sensor (~10,4 fps) requiere `--compression none`**y** un disco de clase NVMe (~500 MB/s de escritura sostenida). El mismo parámetro se expone como `compression` en `POST /api/camera/array/capture`.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --aligned --no-index -o out/
```

### `array-record` — índice combinado vídeo/GIF (calidad de monitorización)

Graba lo que se muestre en la **vista combinada en directo** se muestre en un `.avi` (y, opcionalmente, en un `.gif`). Dado que extrae la señal compuesta en directo, la transmisión combinada debe estar abierta (por ejemplo, la matriz se está previsualizando en la interfaz gráfica de usuario) para que se reciban los fotogramas. Comprueba el progreso cada 2 s y se detiene en `--duration`, `Ctrl+C` o cuando la grabadora finaliza automáticamente.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/
```

| Indicador | Valor predeterminado | Descripción |
| --- | --- | --- |
| `--array-id ID` | solo matriz | Matriz de destino (omitir si solo hay una conectada). |
| `-o, --output DIR` | `output` | Directorio de salida (backendlocal). |
| `--fps F` | `10` | Frecuencia de fotogramas de grabación. |
| `--duration S` | hasta Ctrl+C | Parada automática tras `S` segundos. |
| `--gif` | desactivado | Grabar también un GIF animado. |
| `--gif-only` | desactivado | Guardar solo un GIF (sin `.avi`). |

### `array-burst` — ráfaga de alta velocidad de fotogramas en formato Bayer sin procesar (calidad de análisis)

Lee directamente el búfer de grupo sincronizado del bucle de captura — **sin cadena de calibración, sin exiftool, sin necesidad de vista en directo** —, por lo que funciona a la velocidad máxima de captura de la cámara. Escribe fotogramas RAW + un manifiesto por fotograma + un `.daq` por cada lectura DLS distinta bajo `<output>/bursts/<base>/`. Reprocesar fuera de línea (siguiente comando), o pasa `--build` para hacerlo inmediatamente al detenerse.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| Indicador | Predeterminado | Descripción |
| --- | --- | --- |
| `--array-id ID` | solo matriz | Matriz de destino. |
| `-o, --output DIR` | `output` | Directorio de salida (los resultados se guardan en `<DIR>/bursts/<base>/`). |
| `--duration S` | hasta Ctrl+C | Parada automática tras `S` segundos. |
| `--max-frames N` | sin límite | Parada automática tras `N` fotogramas sin procesar. |
| `--build` | desactivado | Tras la detención, volver a procesar inmediatamente la ráfaga (igual que `array-build-video`). |
| `--products …` | `combined:index` | Con `--build`: qué vídeo(s) generar (véase más abajo). |
| `--fps F` | `10` | Con `--build`: fps del vídeo de salida. |
| `--save-tiffs` | desactivado | Con `--build`: guarda también archivos TIFF calibrados por fotograma. |
| `--gif` | desactivado | Con `--build`: también se escriben GIF animados. |

### `array-build-video` — Reprocesar sin conexión una ráfaga guardada

Sincroniza temporalmente cada fotograma sin procesar con la lectura `.daq` guardada más cercana y lo pasa por la **misma cadena de radiancia / reflectancia / índice que el proceso de importación**, generando uno o más vídeos.

`--products` es una lista separada por comas de elementos `kind:level`, donde `kind` ∈ `per_cam` | `combined` y `level` ∈ `radiance` | `reflectance` | `index`. Un `level` (sin `kind:`) toma por defecto el valor de `per_cam`. El valor por defecto es `combined:index`.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| Indicador | Valor predeterminado | Descripción |
| --- | --- | --- |
| `--burst-dir DIR` | (obligatorio) | Ruta a la carpeta de ráfagas (`…/bursts/<base>/`). |
| `--products …` | `combined:index` | Lista `kind:level`, como se ha indicado anteriormente. |
| `--fps F` | `10` | Fotogramas por segundo (fps) del vídeo de salida. |
| `--save-tiffs` | desactivado | Guardar también archivos TIFF calibrados por fotograma junto con el. |
| `--gif` | desactivado | También graba GIF animados. |

> **Elige la grabadora adecuada.** `array-record` es *de nivel de monitorización*: captura la imagen compuesta en directo tal y como se muestra y necesita que el flujo esté abierto. `array-burst` → `array-build-video` es *de nivel de análisis*: guarda los datos brutos del sensor a velocidad máxima y reconstruye posteriormente vídeos calibrados de radiancia, reflectancia e índice, sin necesidad de visión en directo.

### Cámaras monocromáticas (M3M) de banda única

La gama **M3M**es la variante monocromática de la**M3C**de Bayer: un filtro de interferencias de banda estrecha por cámara (`M3M-<lens>-F<wavelength>`, p. ej., `M3M-L87-F685`), por lo que el sensor ofrece una**única banda en escala de grises** sin mosaico Bayer. No hay nada que demosaicar, ni diafonía entre canales que separar, ni balance de blancos que ajustar: todo el proceso de color de la pantalla «RGB» simplemente no se aplica.

¿Qué significa esto para las CLI?:

- **`lattice white-balance`, `lattice color-profile`, `lattice color`**detectan una cámara monocromática y**la omiten con un mensaje de una línea** en lugar de aplicar ajustes sin sentido. Siguen funcionando con normalidad con una cámara RGB /Bayer M3C en la misma sesión.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** siguen funcionando: la radiancia y la reflectancia son mapas radiométricos *por banda* y están perfectamente bien definidos para una sola banda. Los fotogramas monocromáticos contienen una matriz de respuesta del sensor **identitaria** (sin desmezcla 3×3), por lo que la plano pasa por el proceso de calibración sin sufrir modificaciones.
- **Una sola cámara monocromática no puede generar un índice de vegetación.**NDVI / NDRE /etc. necesitan al menos dos bandas (p. ej., Red + NIR). Para obtener un índice a partir de hardware monocromático, apunta**varias** cámaras M3M a diferentes longitudes de onda, alinéalas en una pila multibanda e indexa *esa*:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

Los símbolos `--channel` deben coincidir **exactamente** (se distingue entre mayúsculas y minúsculas; los de NDVI son en minúsculas: `red`, `nir` — véase `--list-presets`), y el nombre del lado de la banda debe corresponder a una banda de la pila alineada (el modo sin conexión también acepta índices de banda basados en 0, p. ej., `--channel red=0 --channel nir=1`).

El discriminador en toda la pila es el token `M3M` de la cadena del modelo (nunca aparece en una cadena `M3C`), que se muestra en la interfaz gráfica de usuario/ SDK como `is_mono`.

---

## Configuración y ajuste de la tarjeta de red del host (matrices LATTICE)

Las cámaras LATTICE transmiten GVSP a través del adaptador Ethernet del host, por lo que, en el caso de matrices multicámara, el **controlador**y el**tamaño del anillo de recepción** del adaptador son tan importantes como la velocidad del enlace. Una configuración incorrecta se manifiesta como una puerta `FRAMES WILL DROP` / `Reduce ROI to enable` en el panel «Configuración de la matriz» (y en `lattice network-analysis` / el «SDK»`analyze_array_network()`), incluso cuando las propias cámaras funcionan correctamente.

### Adaptadores USB 10 GbE — Realtek RTL8157 («Realtek USB 10GbE Family Controller»)

| Elemento | Valor requerido | Por qué es importante |
| --- | --- | --- |
| **Versión del controlador**|**≥ v10.67 (enero de 2026)**, INF `rtump64x64sta.inf` | El antiguo**controlador de 2016**(v10.65, `rtump64x64.inf`) gestiona incorrectamente el apagado y provoca errores de sistema con**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**al apagar, reiniciar o poner el sistema en suspensión. La transición se cuelga (tiempo de espera de unos 5 minutos), el usuario apaga el equipo a la fuerza, y los apagados incorrectos repetidos**corrompen el repositorio WMI**(PowerShell y otras herramientas empiezan a fallar con el error `Invalid class`) y**bloquean la pila USB** en el siguiente arranque (la tarjeta de red no se activa; las unidades USB dejan de enumerarse). Actualiza desde realtek.com (o del proveedor del dongle) antes de confiar en reinicios limpios. |
| **Búferes de recepción**— palabra clave `ReceiveBufferLen` |**256**(máximo del controlador) | El anillo de recepción (RX) de la tarjeta de red. El valor predeterminado del controlador,**32**, deja solo unos ~0,26 MB de anillo utilizable —demasiado pequeño para una ráfaga de varias cámaras—, por lo que el panel de la matriz muestra el código `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` y bloquea la conexión. A**256**el anillo es grande (**unos 13,5 MB medidos en el host de 10 GbE del laboratorio**), lo que proporciona al canal de recepción un margen real para ráfagas GVSP multicámara. (Que una configuración determinada se *conecte* realmente viene determinado por dos comprobaciones: la comprobación de admisión **sensible al drenaje**y la comprobación de**sobresuscripción agregada** — y no de una simple comparación entre la ráfaga y el anillo; véase [Modelo de fps y ráfagas del array](#array-fps--burst-model).) |
| **URB de recepción**— palabra clave `PendingReceives` |**64** (máx.) | Bloques de solicitud USB en tránsito; se aumenta junto con los búferes de recepción para la absorción de ráfagas. |
| **Trama jumbo** — palabra clave `*JumboPacket` | **9014** | Necesario para paquetes GVSP de 9000 bytes (6 veces menos paquetes por trama que con 1500). |

> ⚠️ **Una actualización del controlador de la tarjeta de red REINICIA estas propiedades avanzadas a sus valores predeterminados.**Tras actualizar o sustituir el controlador del adaptador,**vuelve a aplicar** `ReceiveBufferLen=256` y `PendingReceives=64`, o el panel de la matriz volverá a bloquearse aunque «no haya cambiado nada en el hardware». Esta es la causa principal de que un equipo que antes funcionaba deje de conectarse de repente.

Aplícalo desde un PowerShell **con privilegios elevados** (sustituye el nombre de tu adaptador, p. ej., `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` es válido para adaptadores USB de 10 GbE.** Ahora detecta el tipo de adaptador y ajusta la palabra clave correcta del anillo de recepción: `*ReceiveBuffers`→2048 para tarjetas de red PCIe (Intel I219, etc.), o `ReceiveBufferLen`→256 + `PendingReceives`→64 para el controlador Realtek **USB** de 10 GbE de Realtek (que no expone `*ReceiveBuffers`). Los valores objetivo se limitan al máximo indicado por cada controlador (`NumericParameterMaxValue`), por lo que nunca se escribe un valor fuera de. Ejecútalo desde un terminal **con privilegios elevados**; al igual que cualquier ajuste basado en el Registro, el cambio surtirá efecto tras reiniciar el adaptador o reiniciar el sistema. Los comandos manuales `Set-NetAdapterAdvancedProperty` mencionados anteriormente siguen siendo una buena alternativa, ya que se aplican en tiempo real (vuelven a vincular el adaptador) sin necesidad de reiniciar.

### Conceptos básicos de red (todos los enlaces LATTICE)

- **Direccionamiento:** `169.254.0.0/16` local de enlace (GigE Vision LLA). El host adopta una dirección estática `169.254.x.x/16`; las cámaras y el DAQ-E se autoasignan en el mismo rango. No se requiere DHCP ni puerta de enlace.
- **Tamaño de paquete:**es preferible el jumbo (9000), pero deja que la sonda automática lo detecte; esta vuelve a medir en cada conexión y ya ignora el límite de 1500-bytes de la cámara mediante una sonda GVSP, por lo que se establece en jumbo siempre que el cable realmente lo admita. Establece un valor fijo con `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` solo cuando sepas más que la sonda, y da preferencia a la configuración por comando frente a la permanente: un valor fijo omite la sonda, por lo que si la ruta no puedetransportar realmente 9000,**todas** las capturas agotan el tiempo de espera con `SC_ERR_TIMEOUT -1011` (véase [Variables de entorno](#environment-variables)).
- **El anillo RX se escala con `ReceiveBufferLen`:**con el valor predeterminado `32`, el anillo utilizable es de ~0,26 MB (demasiado pequeño para cualquier ráfaga multicámara); con el valor máximo `256` es grande (~13,5 MB medidos en el host de 10 GbE del laboratorio), lo que proporciona un margen real. La conexión de una configuración viene determinada entonces por la comprobación de admisión sensible al consumo**y** la comprobación de sobresuscripción agregada que se describe a continuación, y no por una simple comparación entre la ráfaga y el anillo.

### fps de la matriz y modelo de ráfagas

Cómo interpretar el panel de configuración de la matriz (y `lattice analyze-array` / el `analyze_array_network` del «SDK»):

- **El ráfaga se suma por cámara según el formato de píxeles real de cada una.**Las cámaras monocromáticas**M3M**transmiten**Mono12 (2 B/px)**;**M3C**transmiten en 8 o 12 bits (la TRI032S emite silenciosamente BayerRG12 incluso cuando se solicita BayerRG8). Así pues, un fotograma a resolución completa de 4 cámaras tiene un tamaño de**~12,6 MB si todas son de 8 bits, pero de ~25 MB con tres cámaras mono de 12 bits****. La proyección determina el formato de cada cámara a partir de su modelo (caché de identidad), por lo que la ráfaga coincide con lo que realmente transporta el cable, y no con una suposición genérica de BayerRG8.
- **Un adaptador Ethernet USB tiene un límite de 200 MB/s, independientemente de lo que indique su placa de características.** La tabla de eficiencia que convierte una velocidad de enlace en una cifra sostenida se deriva del PCIe; una tarjeta de red USB anuncia su velocidad de enlace *Ethernet*, pero está limitada por el bus USB y su controlador. Un adaptador USB 10GbEsolía alcanzar unos ~1063 MB/s «sostenidos» —una cifra que nunca se comprobó— y la regulación resultante corrompía el 6–18 % de las tramas, aunque seguía indicando un número de fotogramas por segundo (fps) adecuado. Las tarjetas de red conectadas por USB tienen ahora un límite máximo de **200 MB/s** como valor absoluto (el límite lo marca el bus, por lo que no varía en función de las especificaciones nominales; un adaptador USB de 1 GbE alcanza unos ~80 MB/s y no se ve afectado). `wire_ceiling_source` en el registro de capacidades lo indica con palabras, y `nic_is_usb` lo señala. Se puede anular en ambos casos con `--wire-ceiling-mbps`.
- **La admitancia tiene en cuenta el drenaje, no se basa en la diferencia entre ráfaga completa y anillo.** Una ráfaga simultánea solo tiene que caber en el *atraso transitorio* = `max(0, Σ per-cam arrival − host drain) × emit_window`, no a la ráfaga completa. En una estructura de host rápido / cams lentas (un host **PCIe**de 10 G + 4 cams de 1 GbE: llegada ≈ 320 MB/s, drenaje ≈ 1063 MB/ss), el host vacía más rápido de lo que las cámaras se llenan, el backlog es ≈ 0, por lo que la emisión simulada a resolución completa**se admite**aunque la ráfaga de 25 MB supere el anillo de 13,5 MB. Si se conectan las mismas cuatro cámaras a un adaptador**USB**de 10 GbE, la salida es de 200 MB/s, no de 1063: la llegada la supera, y la pérdida se manifiesta como tramas corruptas en lugar de como una frecuencia de tramas más baja. En un host de 1 GbE, el límite inferior de DLThr de 31,25 MB/s hace que la llegada supere la capacidad de descarga →**bloquea** correctamente (para *esta* clase de bloqueo, reduce el ROI o utiliza un binning ≥ 2). La admisión es una de las **dos** puertas de conexión; la otra es la comprobación de sobresuscripción agregada que se describe a continuación.
- **Los fps proyectados son un límite máximo conservador de recuperación en serie.**El bucle de captura del host actualmente extrae el búfer de cada cámara**en serie**(aproximadamente una ventana de emisión por cámara cada una), por lo que el ciclo está limitado por `max(readout+emit, N × emit)`, con la emisión por cámara restringida al**enlace de acceso**de la cámara (1 GbE ≈ 80 MB/s), no al enlace ascendente del host. Para una matriz de 4 cámaras a resolución completa de 12, que es de**~2,8 fps**, lo que coincide con los ~2,7–3,0 medidos. Los fps son deliberadamente**independientes de la exposición**, por lo que en escenas con poca luz el valor real puede descender ligeramente por debajo del límite máximo a medida que se alarga la exposición. La recuperación en serie es el verdadero limitador de los fps; paralelizarla elevaría el límite máximo hacia la velocidad de emisión única.
- **La sobresuscripción agregada es un obstáculo insalvable para la conexión.**El límite mínimo de asignación de ancho de banda por cámara es de**8 MB/s**(`ARRAY_PER_CAM_FLOOR_BPS`), por lo que, una vez que se alcanza ese límite mínimo, la demanda agregada (`per_cam × N`) puede superar el**límite máximo de línea libre de colisiones**(`sustained × sim_emit_factor`). Límites máximos prácticos a resolución completa en 1 GbE:**6 cámaras a 1500 MTU, 9 con jumbo**. Este límite máximo depende únicamente del cable y del umbral mínimo; es**independiente del tamaño de la trama**, por lo que**el agrupamiento y las ROI más pequeñas NO ayudan** (reducen los bytes por *trama*, no los bytes por *segundo* regulados por GevSCPD); las únicas soluciones son reducir el número de cámaras, utilizar tramas jumbo de extremo a extremo o una tarjeta de red más rápida. El síntoma sería la pérdida de paquetes GVSP, no una reducción gradual de los fps, por lo que `analyze-array` pone a cero las cifras de fps alcanzables y muestra `**OVER-SUBSCRIBED**`, y `array-connect`, con una resolución fija, **se niega a conectarse** (de lo contrario, el «walk-down» agrupa los fotogramas en bloques más pequeños, lo que tampoco resuelve este tipo de bloqueo). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` rebaja el rechazo a una advertencia sonora para el trabajo en banco; véase [Variables de entorno](#environment-variables).

### Estado de la matriz: qué subsistema está perdiendo fotogramas

El `GET /api/camera/array/<array_id>/capability` de una matriz conectada contiene un
`health`, reevaluado en una ventana móvil de **10 segundos**. Desglosa la pérdida de tramas
en las dos causas que requieren soluciones opuestas, en lugar de informar de una tasa «incompleta»
que no especifica ninguna de ellas:

| Campo | Qué significa | Qué subsistema |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (por serie) | La trama **llegó pero presentaba errores estructurales**— pérdida de paquetes GVSP. |**Red**: ancho de banda, sincronización, anillo de recepción de la NIC, MTU |
| `never_arrived_rate_pct` (por serie) | La trama **nunca llegó**: la cámara no se activó o no se envió nada. |**Disparador / sincronización**: cable M8, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | La peor tasa de cada cámara. | — |
| `per_cam_rate_pct` | Tasa combinada de fallos por cámara (ambas causas juntas). | — |
| `stable_for_seconds` | Tiempo que cada cámara ha permanecido por debajo del 0,01 %. | — |

Por encima del 5 %, el backend registra una línea `[array-health <id>] WARN` indicando la división: en la
primera infracción, en un cambio de banda de gravedad, una vez por minuto mientras persista, y una vez cuando
se resuelva. La mitad dañada registra `[gvsp-corrupt <SN>]` en la primera aparición por cámara y
motivo, y luego un resumen cada 60 s. Todas las evaluaciones siguen registrándose en el archivo de registro del backend;
los contadores avanzan en cada búfer independientemente de lo que se imprima.

El mismo registro indica el número del que depende toda la asignación:

| Campo | Qué significa |
| --- | --- |
| `wire_ceiling_mbps` | El presupuesto de ancho de banda sostenido del host actualmente en vigor, MB/s. |
| `wire_ceiling_source` | De dónde procede esa cifra, en palabras — p. ej., `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` o `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` cuando `--wire-ceiling-mbps` (o el campo **Presupuesto de conexión** de la interfaz gráfica de usuario) lo haya establecido. |
| `nic_is_usb` | `true` para un adaptador Ethernet USB; véase el límite de 200 MB/s más arriba. |

**Interpretación:** un valor distinto de cero en `gvsp_corrupt_rate_pct` con `never_arrived_rate_pct` en 0
significa que la activación y la sincronización del cable son perfectas y que el 100 % de la pérdida se produce en la ruta de red
— reduce el valor de `--wire-ceiling-mbps` y vuelve a conectar. El patrón inverso apunta, en cambio, al
cable de sincronización o a la línea de activación.

> **`--target-fps` no es el indicador de tramas corruptas.** GevSCPD se establece
> una vez al conectarse, por lo que reducir la frecuencia de disparo cambia el ciclo de trabajo y no la
> tasa de ráfagas de emisión simultánea. Una reducción medida de 5× en la demanda no produjo ninguna mejora;
> bajar el límite máximo del cable de 240 a 200 MB/s redujo el porcentaje de tramas corruptas de ese mismo equipo del 10,4 %
> al el 0,00 %.

> **La reducción automática a mitad de flujo no está disponible en el firmware TRI032S.** Una matriz en funcionamiento
> no puede solucionar esto por sí misma; desconéctala y vuelve a conectarla para que el selector de tiempo de conexión pueda
> replanificar con el nuevo límite máximo.

### Síntoma → solución

| Síntoma (Configuración de la matriz / conexión / `analyze_array_network`) | Causa | Solución |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` se restablece a 32 (normalmente tras una actualización del controlador) | Establezca `ReceiveBufferLen`→256, `PendingReceives`→64; vuelve a abrir el panel (reinicia el backend si ha almacenado en caché el tamaño de anillo anterior) |
| El reinicio o el apagado se cuelgan; posteriormente, `Invalid class`: errores WMI, la tarjeta de red no se activa, faltan unidades USB | Antiguo controlador Realtek USB 10 GbE de 2016 → pantalla azul de la muerte (BSOD) `0x9F` → apagado forzado- | Actualiza el controlador del adaptador a la versión ≥ v10.67 (2026) y, a continuación, vuelve a aplicar la configuración del anillo de recepción indicada anteriormente |
| La conexión se establece correctamente, pero devuelve una resolución inferior a la nativa | Smart-prep ha reducido automáticamente el marco para adaptarlo al cable | Actualiza el enlace / acepta la reducción / `--force-tier slip-emit-and-capture` |
| La matriz indica un fps objetivo correcto, pero solo ofrece una fracción del mismo; `health.gvsp_corrupt_rate_pct` distinto de cero, `never_arrived_rate_pct` 0 | El presupuesto de ancho de banda del host, calculado de forma inferida, sobreestima lo que realmente soporta (algo habitual en un adaptador USB-Ethernet, un carril PCIe de poco ancho o una estructura compartida) | Vuelve a conectarte con un valor `--wire-ceiling-mbps` más bajo y vuelve acomprueba de nuevo el bloque de estado. **No** `--target-fps` — La cadencia de GevSCPD se fija al conectarse |
| Faltan cámaras en los grupos publicados; `health.never_arrived_rate_pct` distinto de cero, `gvsp_corrupt_rate_pct` 0 | Ruta de activación/sincronización — las cámaras no se activan, no es un problema de red | Comprueba el cable de sincronización M8 y `--line`; confirma que todos los miembros estén armados (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget` superados en `analyze-array`, o rechazo de conexión con resolución fijada (`array over-subscribes the wire`) | LaLa demanda agregada por cámara (8 MB/s como mínimo × N cámaras) supera el límite máximo del cable sin riesgo de colisión: 6 cámaras a resolución completa en 1 GbE con MTU de 1500, 9 con tramas jumbo | Menos cámaras, tramas jumbo de extremo a extremo o una tarjeta de red más rápida. **El ROI o el binning NO servirán de nada** (el límite máximo es independiente del tamaño de la trama). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` se anula en el banco de pruebas (acepta la pérdida de paquetes) |

---

## `chloros-cli daq`

Comandos del sensor Spectral. Dos clases:
- **`pool-*`**— clientes «HTTP» ligeros que controlan el sensor a través del grupo persistente del backend.**Esta es la ruta compatible y la única presente en el CLI que se suministra.** El backend es el responsable del transporte, por lo que la interfaz gráfica de usuario y los scripts CLI y SDK comparten un único identificador activo en lugar de disputarse el puerto serie.
- **Todo lo demás**(`test`, `record`, `live`, `stream`, `connect`, `info`, `net`, `ota`, `sample-rate`, `calibrate`, `serve`, `ws`, `udp`, `mqtt`, `reflectance`, `login`, `logout`, `status`) — acceso directo al hardware, documentado a continuación para completar la información. Estos requieren el paquete `daq` Python, que**no se incluye en ningún artefacto distribuido**: el CLI compilado lo excluye (`scripts/Build-CLI.ps1` establece `--nofollow-import-to=daq`, y los transportes `pyserial` / `bleak` / `zeroconf` lo incluyen), y el paquete de PyPI SDK tampoco lo contiene. Solo se ejecutan a partir de una copia del código fuente, así que considéralos como una vía de desarrollo interna de MAPIR en lugar de algo a lo que recurrir.
- **`discover` / `list`** se sitúan a medio camino entre ambos: son comandos directos al hardware procedentes de una copia del código fuente, pero en una compilación distribuida recurren a `pool-discover` y el backend realiza el escaneo. Así pues, el escaneo funciona en todas partes, lo cual es importante porque es la única forma de conocer la MAC BLE de un DAQ-M.

> **`chloros-cli daq --help`** (y `-h` / `help`) enumera los subcomandos de `pool-*`; la ayuda se redirige deliberadamente al cliente del pool para que refleje los comandos que realmente se ejecutan. Si invocas un subcomando de acceso directo al hardware en una versión comercial, el programa se cierra con un error explícito que indica el paquete que falta y te remite a `pool-*`; nada falla en silencio. (`discover` / `list` son la excepción: redirigen a `pool-discover` y simplemente funcionan.)
>
> **Todo lo que un cliente necesita es accesible a través de `pool-*`** — conectarse, transmitir, grabar archivos `.daq` calibrados e intercambiar perfiles de condensadores. El DAQ también se puede controlar desde Python con `chloros_sdk.connect_daq_sensor()`, que utiliza la misma ruta compartida.

### Flujo de trabajo de primera conexión del sensor DAQ

```bash
# 1. Smart-detect any DAQ on this machine (Ethernet → BLE → USB precedence)
chloros-cli daq connect

# 2. Detailed scan: every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U COM port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq discover                      # or: daq pool-discover
chloros-cli daq discover --only ble           # BLE only
chloros-cli daq discover --json               # machine-readable

# 3. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect           # smart-detect
chloros-cli daq pool-connect --port COM3                       # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 4. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 5. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 6. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 7. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### Referencia de `pool-*`

| Subcomando | Finalidad |
| --- | --- |
| `daq pool-connect` (smart-detect) | Abrir un sensor en el grupo de fondo. |
| `daq pool-connect --port PORT` | DAQ-U en un puerto serie específico. |
| `daq pool-connect --ble` | DAQ-M a través de BLE, escaneo automático de MAC. |
| `daq pool-connect --mac MAC` | DAQ-M a través de BLE con una dirección MAC conocida (implica `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E a través de Ethernet en un host conocido. |
| `daq pool-connect --eth` | DAQ-E a través de Ethernet, host detectado automáticamente (mDNS + ARP como alternativa; funciona con una caché ARP vacía en Windows y Linux). |
| `daq pool-connect --integration-time MS --frame-avg N --no-ae` | Ajustar la ventana de integración / estado AE. |
| `daq pool-connect --no-stream` | Conectarse, pero sin iniciar aún la transmisión (reanudar con `pool-stream --start`). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | Perfil de corrección de capacidad . El valor predeterminado en el backend es `sunshine_cosine`. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Escanea cada transporte en busca de sensores a los que puedas conectarte, sin establecer conexión. **Así es como se encuentra la dirección MAC BLE de un DAQ-M.** `daq discover` / `daq list` se redirigen aquí automáticamente en las versiones comercializadas. Los sensores que ya están abiertos en el grupo no aparecen en la lista —un DAQ-M conectado deja de anunciar —, así que utiliza `pool-list` para esos casos. |
| `daq pool-list` | Mostrar todos los sensores del grupo del backend. |
| `daq pool-disconnect --sensor-id ID [--all]` | Liberar. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Las N tramas de espectro más recientes. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Reanudar/pausar la transmisión. |
| `daq pool-record --sensor-id ID [--duration SEC] [--output DIR] [--device-name NAME] [--stop]` | Iniciar/ detener una grabación .daq. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Cambiar el perfil de corrección de capacidad en tiempo de ejecución. |

### Subcomandos directos de hardware (solo en el código fuente; no están incluidos en las versiones distribuidas)

> Se incluyen para completar la lista. Requieren el paquete `daq` Python, además de `pyserial` / `bleak` / `zeroconf`, ninguno de los cuales se incluye en la versión compilada CLI ni en PyPI SDK — solo se ejecutan desde una copia del código fuente MAPIR. **Si estás utilizando una versión publicada Chloros, utiliza en su lugar los comandos `pool-*` anteriores**; estos abarcan la conexión, la transmisión, la grabación y la selección de cápsulas.

```bash
chloros-cli daq test --port COM3                           # Verify connection
chloros-cli daq connect --eth                              # Smart-detect over ETH
chloros-cli daq info --eth-host daq-e-xxx.local            # Device summary as JSON
chloros-cli daq discover --only usb,ble --timeout 5        # Scan local interfaces
chloros-cli daq list                                       # Alias of discover
# ^ discover/list are the exception in this section: in a shipped build they
#   fall back to `pool-discover` (the backend does the scan), so they work
#   without a source checkout. The only difference is that the fallback needs
#   the Chloros backend running, as all pool-* commands do.

# Streaming JSON Lines to stdout (pipeable)
chloros-cli daq stream --port COM3 --format jsonl --photometrics

# Record to .daq for 60 seconds
chloros-cli daq record --port COM3 --duration 60 -o ~/Documents/spectra/

# Live spectrum visualization in a window
chloros-cli daq live --port COM3 --record

# Dual-sensor reflectance (ambient + object) → JSON Lines
chloros-cli daq reflectance \
  --ambient-eth-host daq-e-field.local \
  --object-eth-host daq-e-canopy.local \
  --record -o ~/Documents/reflectance/

# Convenience: pick integration_time + frame_avg for a target rate
chloros-cli daq sample-rate --port COM3 --target-hz 5

# Calibration profile management
chloros-cli daq calibrate --port COM3 --list
chloros-cli daq calibrate --port COM3 --set field_calibration_2026_05

# DAQ-E network config (mDNS auto-discovers the host)
chloros-cli daq net --eth-host daq-e-xxx.local set-ip --mode static --ip 192.168.2.20
chloros-cli daq net --eth-host daq-e-xxx.local set-name "sky-sensor"
chloros-cli daq net --eth-host daq-e-xxx.local set-ptp --enabled true --domain 0
chloros-cli daq net --eth-host daq-e-xxx.local set-auto-stream true          # auto-stream on boot
chloros-cli daq net --eth-host daq-e-xxx.local set-require-signature         # require factory-signed cal (fw v1.6.0+; refused while the held cal is unsigned)
chloros-cli daq net --eth-host daq-e-xxx.local set-time                      # push host clock (refused when PTP SLAVE)
chloros-cli daq net --eth-host daq-e-xxx.local set-auth-token --current "" --new "s3cret"   # control-channel auth ("" new = disable)
chloros-cli daq net --eth-host daq-e-xxx.local set-ota-password "newpass"    # change OTA password (min 4 chars)
chloros-cli daq net --eth-host daq-e-xxx.local factory-reset                 # clear all NVS settings and reboot
chloros-cli daq net --eth-host daq-e-xxx.local reboot

# OTA firmware update
chloros-cli daq ota --eth-host daq-e-xxx.local \
  --firmware daq_e_1.21.bin --password mapir-daq-e

# Bridge spectra to other protocols
chloros-cli daq serve --port COM3 --tcp-port 9000           # TCP JSON-lines
chloros-cli daq ws    --port COM3 --ws-port 9001            # WebSocket
chloros-cli daq udp   --port COM3 --udp-port 9002           # UDP broadcast
chloros-cli daq mqtt  --port COM3 --broker mqtt.example.com --topic daq/spectrum
```

---

## `chloros-cli project`

Abrir, conectarse y controlar un proyecto guardado de Chloros (una carpeta con `cameras.json` + `sensors.json` + `project.json`). Todo se canaliza a través del backend, por lo que la interfaz gráfica de usuario y CLI muestran un estado del hardware idéntico.

### Referencia de subcomandos

| Subcomando | Finalidad |
| --- | --- |
| `project open PATH` | Imprimir el manifiesto de dispositivos del proyecto (cámaras, matrices, sensores). |
| `project devices PATH [--reconnect]` | Mostrar una lista o volver a ejecutar la detección. |
| `project connect PATH [--cameras-only] [--sensors-only]` | Conectar todas las cámaras, matrices y sensores guardados. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | Captura única desde una cámara o matriz especificada. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | Ráfaga de N fotogramas de una cámara o matriz especificada (`-n/--count`: valor predeterminado 5; `-i/--interval`: intervalo en segundos entre fotogramas, valor predeterminado 0). Las ráfagas de matrices eliminan la duplicación de grupos sincronizados repetidos (control de caducidad), de modo que una matriz de ciclo parcial no puede devolver N copias de un fotograma; muestra los resultados porresultado de cada iteración. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--poll-interval S]` | Transmisión a disco mediante un trabajo de fondo. `--poll-interval` = segundos entre sondeos `/stats` (por defecto 2,0). |
| `project sensor read PATH NAME [--json]` | Última trama del espectro. |
| `project sensor log PATH NAME --seconds SEC [-o DIR] [--device-name NAME]` | Grabar .daq. |
| `project run PATH RECIPE.yaml` | Ejecutar una receta de captura YAML/JSON. `--dry-run` se valida sin ejecutarse. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Calcula la alineación de una matriz; consulta [la tabla de indicadores más abajo](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | Muestra el perfil de alineación actual. |
| `project align clear PATH NAME` | Eliminar el perfil almacenado en caché. |
| `project align tweak PATH NAME --serial SN --dx N --dy N --rotation-deg N --scale N` | Desplaza la transformación de un esclavo. |
| `project align export PATH NAME --to FILE` | Guarda el perfil en JSON. |
| `project align import PATH NAME --from FILE [--no-validate]` | Carga un perfil guardado. |

#### Opciones de `project align calibrate`

| Indicador | Predeterminado | Descripción |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Método de alineación. **Estas denominaciones difieren de `lattice align-calibrate`**, que admite las formas abreviadas `orb` / `akaze` / `phase`; los dos comandos no son intercambiables en esta opción. |
| `--model {translation, rigid, affine, homography}` | `affine` | Transformar el modelo para ajustarlo. |
| `--frames N` | `1` | Instantáneas de fotogramas sincronizadas para calcular la media. |
| `--reference SN` | la cámara maestra | Número de serie de la cámara de referencia; el resto de miembros se deforman para ajustarse a ella. |
| `--max-features N` | `5000` | Límite de recuento de características ORB. |
| `--ratio-threshold F` | `0.75` | Prueba de la razón de Lowe. |
| `--ransac-threshold-px F` | `3.0` | Umbral de puntos válidos de RANSAC. |
| `--min-matches N` | `15` | **Control de calidad**: rechaza la solución si no alcanza este número de coincidencias de puntos válidos. |
| `--max-reproj-err-px F` | `4.0` | **Umbral de calidad**: rechaza la solución si el error RMS de reproyección supera este valor. |
| `--checkerboard RxC` | — | Geometría de la placa para `--method checkerboard`, p. ej., `9x6`. |
| `--name PROFILE` | vacío | Nombre del perfil incrustado en el archivo «JSON» guardado. **No es el nombre de la matriz** — ese es el `NAME` posicional. |

Las dos puertas de calidad son la razón por la que una calibración puede resolver con éxito y, aun así,
rechazar el guardado: un perfil que falle en cualquiera de ellas registraría incorrectamente de forma silenciosa cada
captura posterior, por lo que se rechaza en lugar de guardarse.

### Ejemplos

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### DSL de recetas

`project run RECIPE.yaml` acepta un archivo YAML o JSON que describa una secuencia de acciones:

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        exposure_auto: "Off"
        gain: 6.0
        gain_auto: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

Acciones admitidas: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. La acción `burst` requiere `name` (obligatorio), `count` (valor predeterminado 5), `interval` (segundos, valor predeterminado 0), `output`, `format` y `settings` (misma configuración por cámara que `apply`); las ráfagas de matriz utilizan el mismo watchdog de grupo recién sincronizado que `project burst`.

Ejecútalo:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

---

## Variables de entorno

| Variable | Efecto |
| --- | --- |
| `CHLOROS_BACKEND_URL` | Anula el backend URL (por defecto `http://127.0.0.1:5000`) — **solo se tiene en cuenta en las familias de comandos `lattice`, `project` y `daq pool-*`.** Los comandos principales (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) fijan `http://127.0.0.1:<port>` e ignoran esta variable (el literal IPv4 elude el Windows `localhost`→`::1` ~2 s-penalización por solicitud), por lo que siempre se dirigen a la máquina local. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | `1` rebaja el rechazo de conexión por sobresuscripción de la matriz (agregado pordemanda por cámara &gt; límite de seguridad contra colisiones del cable con `pin_resolution`) a una advertencia sonora y continuación, aceptando la pérdida de paquetes GVSP. Solo para uso en banco de pruebas — véase [Modelo de fps y ráfagas de la matriz](#array-fps--burst-model). |
| `CHLOROS_CLI_MODE` | Lo establece el propio CLI; indica al backend que habilite el procesamiento paralelo. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` omite la sonda de respaldo de GVSP (solo resultados ICMP). **Esto desactiva los paquetes jumbo, no solo reduce el volumen de los registros**: la cámara solo responde a pings DF de hasta 1500 en cada ruta, por lo que esta prueba es lo único que puede detectar paquetes jumbo. Ahorra ~1 s por cámara y por conexión; cuesta ~1,45× el límite de ancho de banda si la red *pudiera* haber transportado paquetes jumbo. El SDK avisa al configurarlo. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | Fija el tamaño de los paquetes GVSP en N bytes; omite por completo la comprobación. Es preferible utilizarlo por comando (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) en lugar de configurarlo de forma permanente: un tamaño fijo impide la adaptación a la red que hay delante, y fijar 9000 en una ruta que no admite paquetes jumbo hace que **todas** las capturas agoten el tiempo de espera con el error `SC_ERR_TIMEOUT -1011`. |
| `TMPDIR` (Linux) | Anular el directorio de extracción de un solo archivo de Nuitka. El CLI utiliza automáticamente `/mnt/ssd/tmp` si está presente. |

---

## Códigos de salida

| Código | Significado |
| --- | --- |
| `0` | Éxito. |
| `1` | Fallo genérico (la mayoría de los errores de subcomandos). |
| `2` | Error de argumento. |
| `130` | Interrumpido por Ctrl+C. |

---

## Consejos para la resolución de problemas

- **«Se requiere inicio de sesión»** → Ejecuta `chloros-cli login EMAIL PASSWORD` una vez en este equipo.
- **«Backend inaccesible»** → Inicia la aplicación de escritorio Chloros o ejecuta directamente el binario del backend (`chloros-backend`), o comprueba `CHLOROS_BACKEND_URL` si se trata de un acceso remoto.
- **Los comandos de `lattice` fallan con el mensaje «No se han encontrado los controladores de cámara LATTICE»** → No está instalado el tiempo de ejecución de Arena SDK; el CLI se distribuye con `win32api` incluido en Windows, pero el tiempo de ejecución en C forma parte del instalador de la interfaz gráfica de usuario.
- **«Array connect» / «Array Settings» muestra «FRAMES WILL DROP» o «Reduce ROI to enable»** → El anillo de recepción de la tarjeta de red del host es demasiado pequeño (suele restablecerse a 32 tras una actualización del controlador de la tarjeta de red). Véase [Configuración y ajuste de la tarjeta de red del host](#host-nic-setup--tuning-lattice-arrays): configura `ReceiveBufferLen=256` y `PendingReceives=64`.
- **El equipo se cuelga al reiniciarse o apagarse; a continuación, WMI `Invalid class` / la tarjeta de red no se activa / faltan unidades USB** → Un controlador obsoleto del adaptador USB de 10 GbE provoca el error `DRIVER_POWER_STATE_FAILURE` (pantalla azul `0x9F`). Actualice el controlador del adaptador — consulte [Configuración y ajuste de la NIC del host](#host-nic-setup--tuning-lattice-arrays).
- **Advertencia de swap de Jetson** → Añade un swap respaldado por un archivo; el comando «CLI» muestra los comandos exactos `fallocate` / `swapon`.
- **Faltan los comandos directos de DAQ** → Lo esperado: el `chloros-cli` incluido excluye deliberadamente el paquete `daq`, por lo que solo está presente el `pool-*` (el SDK de PyPI tampoco lo incluye). Utilice `pool-*`, que controla el mismo sensor a través del backend, o `chloros_sdk.connect_daq_sensor()` de Python.

---

## Véase también

- [Python Referencia de SDK](sdk-reference.md): equivalente programático de todos los comandos de CLI.
- [Guía de sensores DAQ](../daq/README.md): cableado y calibración específicos para cada sensor.
- Documentación en línea: `https://mapir.gitbook.io/chloros/cli`

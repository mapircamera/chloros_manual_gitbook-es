# Referencia de la CLI Chloros

**Versión:**

1.3.0**Generada:**29/07/2026 19:19 ·**Revisada:** 06/11/2026**Destinatarios:** Optimizado para su uso por modelos de lenguaje grande (LLM); legible para humanos.**Ámbito:** Todos los subcomandos de `chloros-cli` destinados a los usuarios, con opciones y ejemplos que se pueden copiar y pegar.

Este documento es la referencia completa de la herramienta de línea de comandos `chloros-cli` que se incluye con MAPIR Chloros. Se ha redactado de forma intencionadamente exhaustiva para que un LLM (o una persona) pueda crear cualquier flujo de trabajo compatible a partir de los listados que figuran a continuación sin necesidad de examinar el código fuente.

Si solo necesitas lo más destacado, ve directamente a:
- [Guía de inicio rápido en cinco minutos](#five-minute-quickstart)
- [Flujo de trabajo de primera conexión de la cámara LATTICE](#lattice-camera-first-connect-workflow)
- [Flujo de trabajo de primera conexión del sensor DAQ](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Modos de captura, grabadoras y reprocesamiento fuera de línea](#capture-modes-recorders--offline-reprocess)

> **¿Controlas Chloros desde un asistente de IA?** Claude Desktop, Claude Code, Cursor y otros clientes de MCP pueden ejecutar Processing, cámaras y matrices LATTICE, así como sensores de luz DAQ a través de `chloros-mcp`, el servidor MCP que se instala con el Python SDK (y con el .deb de Linux, en `/usr/bin`): `claude mcp add chloros -- chloros-mcp`. La configuración, las opciones y las herramientas se encuentran en la sección [Servidor MCP (asistentes de IA)](sdk-reference.md#mcp-server-ai-assistants). Un asistente con un shell también puede ejecutar directamente todos los comandos de esta página.

> **Modificaciones en la versión 1.3.0** (lo que detectará un script o flujo de trabajo existente de la versión 1.2.0).
>
> **Cambios que un script de la versión 1.2.0 podría tener que gestionar:**
>
> - **Se aplica `--preset`.** `lattice capture`, `continuous`, `calibrate`, `multi-capture`, `multi-stream`, `align-calibrate`, `align-apply` y `align-stream` lo eliminaron en la versión 1.2.0 y funcionaba con los ajustes propios de la cámara. Ahora configura el formato, la exposición, la ganancia y la velocidad de fotogramas del preajuste: `--preset high_speed` captura a 8 bits a 5 ms, 6 dB y 60 fps. `lattice cam-connect --preset` también mantiene el formato del preajuste (la versión 1.2.0 configuraba la cámara a 12 bits tras aplicar el preajuste). Véase [Opciones comunes](#common-options-shared-by-most-subcommands).
> - **El formato de píxeles Bayer en una cámara M3M (mono) es su profundidad.** Los preajustes denominan formatos Bayer, que un sensor mono no puede capturar: en la versión 1.2.0, la cámara rechazaba el formato y mantenía el que tenía, por lo que `--preset high_speed` no era de 8 bits en ese caso. Ahora es el formato Mono de la misma profundidad: `high_speed` y `--pixel-format BayerRG8` dan como resultado `Mono8`, `high_quality`, `triggered` y `BayerRG12` dan como resultado `Mono12`. En una matriz mixta de M3C + M3M, cada cámara adopta el formato compartido dentro de su propia familia, y el proyecto `network-analysis` / `analyze-array --pixel-format` proyectan cada cámara a esa profundidad. Lo mismo ocurre a la inversa: una M3C a la que se le solicita un formato Mono adopta el formato Bayer de esa profundidad, por lo que una matriz cuyo maestro sea una M3M ya no vuelve a abrir sus cámaras M3C en Mono. Véase [Cámaras monocromáticas (M3M) de banda única](#mono-m3m-single-band-cameras).
> - **Las exportaciones de `lattice array-capture` están alineadas por defecto, al igual que las de la aplicación.** Los archivos `debayered`, `display`, `radiance`, `reflectance` y los archivos de índice se ajustan al fotograma de la cámara maestra y se recortan a la zona de solapamiento de los miembros (más pequeño que el sensor debido a los desplazamientos de las cámaras, `XMP-mapir:AlignmentJSON` `"applied": true`), por lo que una captura de CLI corresponde a los mismos archivos que la captura de la misma escena realizada por la aplicación Chloros; `raw` se mantiene en formato nativo. Anteriormente, una captura sin `--aligned` guardaba todos los archivos en su tamaño nativo, sin ajustar. `--no-aligned` mantiene esa configuración. El mismo valor predeterminado se aplica a `--smart`, `--continuous` y `--interval`, así como al SDK y a `chloros-mcp`. Véase [Modos de `array-capture`](#módulos-de-captura-de-matriz).
> - **`lattice array-capture --compression` se establece por defecto en `none`.** Los archivos se graban sin comprimir (unos 6,3 MB por fotograma a resolución completa) para obtener la velocidad sostenida más rápida; la versión 1.2.0 grababa en `deflate` (unos 4,1 MB). Utiliza `--compression deflate` para obtener archivos más pequeños. Ambos formatos son sin pérdidas.
> - **Los formatos PNG y JPEG solo admiten los productos `display` / `preview` y los productos indexados.** Una captura `--format png` o `jpg` con un nivel que un archivo de 8 bits no puede contener se rechaza antes de que se capture nada (código HTTP400, salida `1`), al igual que el `--processing all` predeterminado con `--format png`/`jpg`. Véase [Niveles de exportación de capturas](#capture-export-levels-the-all-default).
> - **`lattice dls` requiere `--mac`.**Ya no hay una MAC predeterminada (era la dirección de una unidad); `daq pool-discover --only ble` encuentra la de un sensor.**`lattice dls --camera SERIAL`** es nuevo.
> - **Más errores al salir de `1`.** `process` cuando no se han podido verificar las exportaciones de una ejecución o se ha perdido una imagen, aunque se hayan escrito los demás productos; un comando de captura con cualquier archivo que no se haya podido escribir (los demás archivos se guardan igualmente); `lattice continuous` y `multi-stream` cuando la unidad está llena; `array-burst` / `array-record` cuando la grabadora informa de un error; `array-build-video` cuando se ha omitido un producto solicitado o no se ha creado según lo solicitado; `project burst` / `stream` cuando una captura falla a mitad del proceso; `project sensor log` cuando la grabación no contiene ninguna lectura; `lattice exposure --set`, `gain --set` y `configure --set` con un valor que la cámara rechaza. `lattice network-analysis --json` se cierra al igual que el resumen (`1` para `needs_force_slip`). Al revés: `daq pool-disconnect` de un sensor que ya no está en el grupo sale de `0`. Véase [Códigos de salida](#exit-codes).
> - **Un `lattice array-connect` lento nunca falla con la matriz conectada.** Espera hasta 600 s (180 s en la versión 1.2.0; `project connect` 90 s), mostrando el paso del backend a medida que avanza, y una conexión que no se haya completado para entonces es deshecha por el backend — cada cámara vuelve a su estado inicial —, por lo que un `Error:` (salida `1`) significa que no hay ninguna matriz conectada. En la versión 1.2.0, una conexión realizada justo después de que las cámaras se encendieran podía prolongarse más allá del tiempo de espera y conectar la matriz después de que el comando hubiera fallado. Ahora, Ctrl+C cancela la conexión. Véase [Conexiones persistentes](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **`lattice array-connect` mantiene el sensor completo.** Sin `--width` / `--height` / `--binning`, todas las cámaras se configuran con el sensor completo y la velocidad de fotogramas desciende hasta el nivel que permiten este ordenador y la red; la salida de conexión muestra opciones más rápidas (agrupación de píxeles o una región de interés más pequeña) con sus velocidades previstas. La versión 1.2.0 reducía o agrupaba los fotogramas para mantener la velocidad. Véase [Opciones de `array-connect`](#array-connect-options).
> - **Una red lenta ya no rechaza una matriz.** Cada cámara se adapta a su parte de ancho de banda y la velocidad de fotogramas disminuye en consecuencia. Cuando no es posible adaptar las cámaras a un ritmo tan lento, la matriz funciona en modo de emisión en serie (`slip-emit-and-capture`): llegan todos los fotogramas, pero las cámaras no se exponen al mismo tiempo. `--force-tier sim-capture-sim-emit` exige una exposición simultánea. Véase [Fps de la matriz y modelo de ráfaga](#array-fps--burst-model).
> - **`lattice network-analysis` / `analyze-array` definen qué «connect» se aplica**: sin `--pixel-format`, los formatos que establece «connect» (Mono12 / BayerRG12, antes era BayerRG8) y la decisión sobre [transporte empaquetado de 12 bits](#packed-12-bit-transport---packed). Por lo tanto, el mismo comando puede indicar una frecuencia de fotogramas diferente a la de la versión 1.2.0.
> - **Las rutas relativas se refieren a la ubicación desde donde se ejecuta el comando.** `-o` y las demás rutas que abre el backend (`--input`, `--burst-dir`, `--profile`, `--save`, `--mask`), el `output` de un paso de receta, y el `output/` predeterminado de `lattice index --live` se resuelven en función de la carpeta desde la que se ejecuta `chloros-cli`. En la versión 1.2.0, el backend los resolvía en su propia carpeta de instalación, lo que, en una compilación instalada, provocaba un error con el mensaje «Acceso denegado». Véase [Convenciones](#conventions).
> - **`lattice index` se ejecuta en el backend en todas sus variantes**, al igual que los demás comandos `lattice`, por lo que debes iniciar primero el backend. El índice sin conexión (`--input`) lo calcula el backend, que lee el archivo TIFF y escribe los productos junto a él o en `-o`, mientras que `--list-presets` y `--list-gradients` imprimen las tablas del backend. En Windows, ahora funcionan: antes, la Windows de Windows respondía a cualquier forma de `lattice index`, incluida `--live`, con `Error: lattice_sdk not available`. En Linux, el índice offline solía ejecutarse dentro de la CLI sin backend y sin iniciar sesión; ahora necesita ambos, como cualquier otro comando de la CLI. Véase [Índice / Cálculos de vegetación](#index--vegetation-maths).
> - **Nuevos archivos junto a los productos.** Cada producto de radiancia, reflectancia e índice en el que la desmezcla de bandas ha limitado un píxel obtiene una máscara en una carpeta `clamp_masks/` junto a la carpeta del producto, y la captura imprime una línea `clamp mask:`. Véase [Máscara de recorte de la separación de bandas](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on).
> - **Orientación de la cámara.** Una captura con una sola cámara guarda los archivos `raw` y `debayered` tal y como los leyó el sensor y registra el giro o la rotación de la cámara en el archivo XMP; `process` lo aplica a los productos que genera. Los archivos miembros de la matriz se mantienen en su formato nativo a menos que se pase el nuevo `--orient` (`array-capture`, `array-burst`, `project capture` / `burst` / `stream`). Véase [Orientación de la cámara](#cómo-es-una-carpeta-de-capturas).
> - **Se rechaza cualquier nombre de nivel de exportación que no sea uno.** `lattice capture --processing` y todas las entradas de la lista `--levels` admiten `raw`, `debayered`, `radiance`, `reflectance` o `display` (o un alias como `preview`; `--processing` y también `all`). Se rechaza cualquier otro nombre antes de acceder a la cámara (400 HTTP `UNKNOWN_LEVEL`, salida `1`); 1.2.0 guardó `raw` para ello y salió de `0`.
> - **`daq pool-set-cap` necesita `--cap-id`.** Sin ello, el comando sale de `2`; 1.2.0 eliminó la corrección de la tapa, dejando el sensor al descubierto. `--cap-id none` solicita el sensor sin correcciones.
> - **`lattice resolution --binning-mode` ha desaparecido.** El binning es siempre «Average»; la bandera sigue aceptándose, muestra una advertencia y no realiza ninguna acción. Véase [Ajuste del sensor](#sensor-tuning).
> - **`process --debayer texture-aware` no se admite para capturas LATTICE.** En el caso de una carpeta que contenga capturas LATTICE, `process` sale de `1` tras la importación, indicando cuántas hay, antes de procesar nada. Las capturas de LATTICE siempre se someten al demosaico estándar, por lo que la versión 1.2.0 aceptaba la opción y la ignoraba; la aplicación Chloros tampoco la ofrece para ellas. Véase [`chloros-cli process`](#chloros-cli-process).
> - **Las cámaras vuelven a su estado original — a menos que se utilice `--keep-settings`.** `lattice array-disconnect`, todos los comandos transitorios (`multi-capture` sin matriz, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, `index --live`) y `lattice continuous` devuelven cada cámara al estado en que la encontraron: el cableado del disparador y de la línea de sincronización, su PTP y todos los ajustes de la cámara que se modificaron durante la sesión (exposición y ganancia, el objetivo de AE, límites y amortiguación, balance de blancos, nivel de negro, formato de píxeles, binning y ROI, volteos, frecuencia de fotogramas). `--keep-settings`, en cambio, guarda los ajustes de cada cámaraen la propia memoria de la cámara, de modo que se enciende con ellos; `--clear-settings` restablece esa memoria a los ajustes de fábrica. En cualquier caso, las cámaras se dejan funcionando de forma autónoma. Un proyecto configura sus cámaras a partir de sus propios ajustes guardados cuando las conecta. Véase [Memoria de la cámara](#camera-memory----keep-settings----clear-settings).
> - **Todos los modos guardan un proyecto de la misma manera.** Cualquier cambio en una cámara, una matriz o un sensor de luz conectado a un proyecto —ya sea desde la aplicación, la CLI, el SDK o el servidor MCP— se guarda en el archivo `cameras.json` / `sensors.json` de ese proyecto, y al abrir el proyecto en cualquier modo se configuran los dispositivos a partir de él. Antes, solo la aplicación guardaba el proyecto, y su siguiente guardado sobrescribía cualquier otra cosa. Véase [Un único registro de proyecto, todos los modos](#one-project-record-every-mode).
> - **Los archivos creados a partir de varias cámaras incluyen el nombre de cada una de ellas.** Una pila multibanda (`align-apply`, `align-stream`, `index --live --save-multiband`), un índice en tiempo real creado a partir de las bandas de varias cámaras y una imagen `composite/` de una captura en matriz contienen el número de serie, el modelo y el paquete de calibración de cada cámara de origen en forma de listas (`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceCalBundleSha256s`, …), y EXIF `SerialNumber` enumera los números de serie separados por comas. Antes no incluían ninguna cámara, o solo el número de serie de la primera cámara. Véase [Archivos creados a partir de varias cámaras](#files-made-from-several-cameras).
> - **Solo un archivo `raw` contiene las etiquetas de calibración de la cámara.** El conjunto compatible con MicaSense (`BlackLevel`, `RadiometricCalibration`, el `Band{R,G,B}RadiometricCalibration` por canal, los conjuntos `MicaSenseVignetting*`) y la carga útil `CalibrationJSON` describen los valores sin procesar del sensor, por lo que se incluyen únicamente en el archivo de captura `raw`. Todos los demás archivos —los archivos de captura `debayered`, `display`, `radiance`, `reflectance` y los archivos de índice, las tramas `lattice continuous` y `multi-stream`, y cada producto `process` — contiene la identidad de la cámara (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) y, en cada captura, el registro propio del fotograma (`XMP-mapir:FrameJSON`: exposición, ganancia, temperatura, formato de píxeles, hora de captura). En la versión 1.2.0 también incluían el conjunto de datos sin procesar, que una herramienta que lo aplicara (Pix4D, Metashape) se aplicaba a píxeles que no eran valores sin procesar. `process` encuentra la calibración de dicho archivo mediante su número de serie y `CalBundleSha256`.
>
> **Otros cambios:**
>
> - **La frecuencia de fotogramas automática de una matriz se ajusta a lo que esta proporciona.** Una frecuencia que pierde más del 1 % de sus disparos se reduce, tras evaluar varias observaciones prolongadas en lugar de una sola, y la frecuencia desde la que se redujo se vuelve a intentar de vez en cuando. La frecuencia que el usuario establezca por sí mismo no se modifica.
> - **La exposición automática es más estable.** Se mantiene constante con luz constante, se estabiliza en cuestión de segundos tras un cambio, ignora las pequeñas zonas sobreexpuestas y restablece el funcionamiento de una cámara que deja de responder bajo el sol intenso. Consulta [Cómo se comporta la exposición automática](#how-auto-exposure-behaves).
> - **`process` solo cuenta tus imágenes y grabaciones.** Si se vuelve a ejecutar en una carpeta que contiene el `camera_cal/` de un proyecto, ya no se cuentan esos archivos de calibración («Se han encontrado 96 imágenes» en un conjunto de 90 imágenes), y un proyecto que ya los haya incluido los descarta.
> - **`lattice multi-test` parte del estado de encendido de las cámaras.** Activa el PTP de las cámaras para la ejecución, espera a que sus relojes se sincronicen y se estabilicen antes de su primer disparo —cada cámara permanece sincronizada durante 120 s, unos 2 minutos desde el encendido (`--ptp-settle SECONDS` lo configura), ya que un reloj recién sincronizado presenta un desfase de hasta ~200 µs durante unos 2 minutos, y restablece el PTP de cada cámara tal y como lo encontró. No se vuelve a esperar a una cámara que siga sincronizada de una prueba anterior. Anteriormente, dejaba el PTP desactivado en una red con un único conmutador en la que ningún DAQ-E respondía, e interpretaba el reloj de funcionamiento libre de cada cámara como un reloj PTP que no se sincronizaría (`PTP is not locked`, en cada ejecución). Véase [Multicámara (sesiones transitorias)](#multi-camera-transient-sessions).
> - **Una grabación `.daq` indica si su tapa fue seleccionada o se asumió que estaba presente.** Los modelos DAQ-U, DAQ-M y DAQ-E se suministran sin tapa, y Chloros no puede detectar si se ha instalado una, por lo que un sensor conectado sin una tapa especificada recibe el valor `sunshine_cosine` por defecto, lo cual es una *suposición*. Ahora, cada registro de Chloros (esquema 1.30) almacena `cap_id_source` junto a `cap_id`: `operator` si has asignado un nombre a la tapa (incluido `none`), y `auto_default` si se ha utilizado el valor por defecto. Al procesar un registro cuya tapa se ha supuesto, con la tapa DAQ del proyecto en «Auto», muestra un mensaje `[daq-dls] WARNING` con el nombre del archivo: un DAQ-U sin corrección de luz solar da una lectura aproximadamente 30 veces mayor. **Asigna un nombre al límite que hayas establecido** al conectarte, o configura el límite de DAQ del proyecto, y la advertencia desaparecerá.
> - **Se acabaron las ventanas de consola «pip» en Windows.** En un PC con Python instalado, el backend y cada uno de sus procesadores de imágenes ejecutaban `pip` una vez al iniciarse, y cada uno abría una ventana de consola. Ya no lo hacen.
> - **Las grabaciones en directo de `.daq` sobreviven a un corte de corriente.**Una grabación se escribe con un diario y una sincronización completa del disco en cada confirmación, detrás del sensor, por lo que un disco lento retrasa el archivo, pero nunca las lecturas. Una grabación interrumpida a mitad de una confirmación (corte de corriente, cierre forzado) puede dejar un archivo `<name>.daq-journal` junto a ella:**copia los dos juntos.**Chloros retrocede el diario la primera vez que lee el archivo, por lo que el registro se abre con todo lo confirmado antes de la interrupción (se pierden aproximadamente los últimos 3 s); en la versión 1.2.0, un corte de corriente en el momento inadecuado podía dejar todo el archivo ilegible («la imagen del disco de la base de datos está mal formada»).
> - **Una grabación en un disco ocupado ya no bloquea el sensor.** `daq pool-record` pone en cola las lecturas y las escribe en un hilo propio; una lectura que no se ha podido escribir se contabiliza (`daq pool-list` muestra `N write error(s)` con el último error) en lugar de bloquear el sensor (una escritura tardó 8,4 s junto a cuatro cámaras que guardaban archivos TIFF).
> - **`lattice multi-capture` sin matriz ya no pierde los fotogramas que ha guardado cuando se agota el tiempo de espera de un disparador.** El disparador de una cámara que agota el tiempo de espera (`SC_ERR_TIMEOUT -1011`, observado con cuatro cámaras en un conmutador de 1 GbE) finaliza la ejecución en ese grupo de fotogramas: los grupos ya guardados se enumeran con `Saved:`, seguido del error, salida `1`. En la versión 1.2.0 se producía un error HTTP500 sin lista, aunque los archivos se encontraban en el disco.
> - **Las cámaras activadas por software ya no omiten cada dos activaciones.** Una cámara en modo de activación por software ahora pone en cola una activación que llega durante su lectura (o Chloros espera a que esté lista), tal y como ya hacían las cámaras sincronizadas por cable.
> - **`daq pool-discover` nombra un DAQ-E-S `DAQ-E-S`** en su columna `MODEL` (y la de `--json`, en `model`). Indicaba `DAQ-E` para cada sensor Ethernet; uno que no responde a la comprobación de identidad en unos 2 s sigue apareciendo en la lista, como `DAQ-E`.
> - **Productos de índices LATTICE mejorados** (NDVI y otros índices) a partir de `process`, en todos los modos de procesamiento.
> - **Cámaras OCN: productos de radiancia, reflectancia e índices mejorados bajo luz solar.** Se aplica una corrección de flota tras la separación de bandas; las cámaras RGN no sufren cambios, y los valores de OCN difieren de los de la versión 1.2.0. Esto también se aplica a una captura de OCN realizada con una versión anterior, cuando se procesa en un ordenador que contiene la calibración con la que se realizó la captura (Chloros conserva la calibración de una cámara una vez que esta se ha conectado).
> - **`lattice continuous` graba el fotograma sin procesar.** Una ejecución de `tiff` generaba fotogramas desmosaicados de 3, a partir de los cuales `process` no puede trabajar. Sus fotogramas son ahora el archivo `raw` que genera una captura de matriz: un plano mosaico con los valores propios del sensor, sin comprimir, junto con el paquete de calibración. Un script que lee tres planos de estos archivos solo lee uno.
>
> - **Cada captura guarda la lectura del DAQ que ha utilizado, al igual que lo hace la interfaz gráfica de usuario.** `lattice capture`, `lattice continuous` (un `.daq` por fotograma), `array-capture` y `multi-capture` con una matriz conectada (y el SDK) escriben el `.daq` y su `.match.json` cada vez que se vincula o se agrupa un DAQ, incluida una captura solo en formato sin procesar; en la versión 1.2.0, una captura sin procesar o sin debayering se guardaba únicamente con `--force-daq`. `--no-daq` lo desactiva, `--daq-tolerance-ms MS` establece la tolerancia de coincidencia para esa captura, y el nuevo `daq pool-bind` vincula una cámara o una matriz a un DAQ desde la CLI. Con `.daq`, una captura guardada se empareja en `process` únicamente con las cámaras que se especifiquen. Ahora, una captura espera a la siguiente lectura del DAQ para poder combinar las lecturas de ambos lados de la exposición. Véase [La lectura del DAQ que guarda una captura](#the-daq-reading-a-capture-saves) y [Una lectura del DAQ alejada de la exposición](#a-daq-reading-far-from-the-exposure).
> - **`process --indices NAME`** añade el índice al proyecto para las bandas de cada cámara, incluyendo un nuevo proyecto. En la versión 1.2.0, con `Unknown index name(s) ignored` se ignoraba cualquier nombre que aún no figurara en el proyecto.
> - **`process` junto a una matriz en tiempo real:**ya no reinicia un backend que contenga cámaras o DAQ, y espera hasta 15 s a que finalice una ejecución que esté llegando a su fin.**Ctrl+C** detiene la ejecución en el backend, no solo en la CLI. Véanse las [Notas sobre `process`](#notes).
> - **Una carpeta de proyecto demasiado profunda para el límite de 260 caracteres de las rutas de Windows** se rechaza directamente, utilizando la carpeta más larga que quepa, en lugar de realizar una ejecución que no escriba nada. El instalador de Windows ofrece ahora la opción «Habilitar rutas largas de Windows» y una exclusión de Microsoft Defender para el backend, ambas marcadas por defecto. Véase [Windows (.exe)](#windows-exe).
> - **`lattice array-capture --continuous`** ejecuta la captura continua propia del backend: una línea de progreso cada 2 s, un resumen en lugar de un listado por archivo y un código de salida distinto de cero si hay algún archivo sin escribir o si la unidad está llena. Véase [Modos de `array-capture`](#array-capture-modes).
> - **El primer `array-capture` tras `array-connect`** espera hasta 20 s por el primer grupo de fotogramas de la matriz, en lugar de fallar con el código HTTP 503 «La matriz aún se está calentando».
> - **Exposición de la matriz en una zona de interés (ROI):** una ROI pequeña ya no limita la exposición de un miembro de la matriz al periodo de funcionamiento libre del sensor (4,9 ms a 768 × 304). Véase [Exposición en un miembro de la matriz](#exposure-on-an-array-member).
> - **La exposición automática ya no reduce la frecuencia de fotogramas automática de una matriz.** Una matriz sincronizada por cable funciona a la frecuencia que permiten su red y la lectura de las cámaras; la exposición automática ocupa el periodo que dicha frecuencia proporciona y, a continuación, aumenta la ganancia. Una exposición manual más larga que el periodo (`lattice exposure --set`, `configure --set`) ralentiza el disparador para adaptarse a ella en lugar de rechazarla, y la frecuencia se restablece cuando el miembro vuelve al modo automático. Véase [Exposición en un miembro de la matriz](#exposure-on-an-array-member).
> - **Las matrices producen menos grupos de fotogramas incompletos** y menos errores 503 de captura («No hay ningún grupo de fotogramas sincronizado reciente»): la entrada de disparo de cada cámara esclava ahora está libre de interferencias.
> - **Transporte empaquetado de 12 bits** (`--packed`, por defecto `auto`) aumenta las velocidades de fotogramas de las matrices de sensor completo en aproximadamente un tercio. Ahora se aplica `array-connect --packet-size` (la versión 1.2.0 lo aceptaba pero lo ignoraba). Véase [Transporte empaquetado de 12 bits](#packed-12-bit-transport---packed).
> - **Una matriz cuyas cámaras se recuperan por sí solas tras un reinicio de alimentación.** Tras un reinicio de alimentación del conmutador, la matriz de `array-connect`las cámaras cuando estas responden y vuelve a transmitir con PTP activado; en la versión 1.2.0 se detenía definitivamente y se necesitaba un nuevo `array-connect`. `array-status` informa de `ptp_enabled: False` (y `ptp_off:`) mientras una cámara detecta que el PTP está desactivado, cuando antes informaba de la configuración con la que estaba conectado el conjunto. Véase [Conexiones persistentes](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **`lattice array-record` graba sin la interfaz gráfica abierta.** Con la vista combinada de la interfaz gráfica cerrada, el backend genera la vista por sí mismo: elNDVI, o el índice que se pase con el nuevo `--expr`. Una grabación que no escriba ningún fotograma ahora falla (`no-frames: …`, salida `1`). Antes, solo se grababa mientras la vista combinada de la interfaz gráfica de usuario estuviera abierta, y una grabación que no escribía nada seguía saliendo de `0`. Véase [`array-record`](#array-record--combined-index-videogif-monitoring-grade).
> - **Una cámara que Chloros ya tiene en su poder no se abre por segunda vez.** Un comando que intentara abrirla por sí mismo devuelve en su lugar un error HTTP 409 `CAMERA_HELD`, indicando quién la tiene en su poder y el comando que la libera. Esto incluye una cámara que otro comando está utilizando en ese momento (un `multi-test` o un `multi-capture` que esté abriendo sus cámaras, un `lattice capture`): el segundo comando se rechaza hasta que el primero finalice. Volver a abrirla podría detener la transmisión del titular o provocar un fallo en el backend. Véase [Conexiones persistentes](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **Los comandos transitorios de múltiples cámaras entregan todas las cámaras o fallan.** `lattice multi-capture` sin ningún conjunto conectado, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream` y `index --live` salen de `1` indicando una cámara que no ha enviado ningún fotograma. Las transmisiones se saltan un grupo de fotogramas en el que falta una cámara y se detienen tras tres seguidos; `multi-test` imprime una fila por activación, `(no frame)` donde una cámara no ha enviado el fotograma; `align-stream` también sale de `1` ante un error en el trabajo. A continuación, cada ejecución restablece cada cámara tal y como la encontró: su cableado de disparo y de línea de sincronización, su PTP (`PtpEnable`) y todos los ajustes de la cámara que la ejecución haya modificado —exposición y ganancia, objetivo AE, límites y amortiguación, balance de blancos, nivel de negro, formato de píxeles, binning y ROI, volteos, frecuencia de fotogramas—, a menos que `--keep-settings` / `--clear-settings` (véase [Memoria de la cámara](#camera-memory----keep-settings----clear-settings)). Antes, estos comandos conservaban los fotogramas que llegaban y salían de `0` (`align-stream` mostraba un error de trabajo como `Note:`), y dejaban todas las cámaras en `TriggerMode` `On`. También ajustaban la exposición de cada cámara a partir de la escena anterior a guardar un fotograma (unos segundos como máximo), en el valor en el que una matriz conectada inicia cada cámara: 40 multiespectral, 60 RGB (`--brightness N` lo configura), y la mantiene durante la sesión; lo mismo ocurre con un único `lattice capture` de una cámara que Chloros no mantiene. Antes, el primer fotograma de cada cámara se capturaba al límite de exposición propio de la cámara, unos 100 ms, independientemente de la luz, y a menudo se recortaba. Véase [Multicámara (sesiones transitorias)](#multi-camera-transient-sessions).
> - **Los comandos transitorios de multicámara comparten la red del mismo modo que lo hace `array-connect`, y solo abren `--serials`.** Cada cámara se adapta a su parte del enlace del host, por lo que cuatro cámaras funcionan en un enlace de 1 Gbps; antes, las cuatro solicitaban 125 MB/s entre todas, el total de 1 GbE, y cada fotograma perdía paquetes. Una matriz que la red no puede soportar se rechaza antes de que ninguna cámara transmita. Ahora `--serials` elige las cámaras, siendo la primera la que activa el disparador; antes, cada comando abría todas las cámaras de la red independientemente de lo que indicara `--serials`. Un fallo que consiste en la pérdida de paquetes GVSP ahora se indica como tal, en lugar de apuntar al cable de sincronización M8. Véase [Multicámara (sesiones transitorias)](#multi-camera-transient-sessions).
> - **`lattice array-disconnect` deja las cámaras funcionando libremente, tal y como se encontraron.** `TriggerMode`, `Off` y todo lo demás tal y como lo encontró la conexión: la configuración del disparador y de la línea de sincronización (`TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, `LineMode` / `LineSource`), PTP y los ajustes de la cámara que se modificaron durante la sesión — de modo que un únicocomando de cámara dirigido a un antiguo miembro permite obtener todos los fotogramas de una sola vez, y el maestro ya no se queda en `TriggerSource` `Software`. `--keep-settings` guarda la configuración de la sesión en la memoria de cada cámara, mientras que `--clear-settings` restablece dicha memoria (véase [Memoria de la cámara](#camera-memory----keep-settings----clear-settings)). `lattice continuous` también configura una cámara que encuentre activada en modo libre para la carrera (y la vuelve a poner en su estado anterior después), y una carrera sin salidas de fotograma sale de `1`.
> - **`lattice configure --get` / `lattice exposure`** imprimen un `note:` con laexposición real o ganancia de la cámara mientras se ejecuta su modo automático. Un `lattice exposure --set` / `configure --set` rechazado indica el rango que acepta la cámara y restablece el modo automático en lugar de desactivar la exposición automática. Véase [Ajuste del sensor](#sensor-tuning).
> - **Nuevos controles de la cámara:**`lattice highlights` (atenúa las superficies blancas brillantes en la representación de la pantalla RGB) y `lattice exposure --brightness-auto`.**`lattice multi-detect`** detecta las cámaras conectadas por cable en una ronda de sondeo y establece por defecto la cámara RGB como maestra.
> - **Nuevos comandos DAQ:** `daq pool-bind`; `pool-enable-ptp`; y para la configuración propia de un DAQ-E: `pool-info`, `pool-set-network`, `pool-set-auto-stream`, `pool-reboot` y `pool-update-firmware`. Véase [Configuración del dispositivo DAQ-E](#daq-e-device-settings).
> - **Un DAQ-E que deja de transmitir es reconectado por el backend.** Cuando el flujo de un DAQ-E del grupo deja de transmitir (por un reinicio del conmutador que lo alimenta, una pérdida de conexión de la cámara), el backend lo vuelve a conectar con la configuración que tenía, reintentando la conexión a intervalos cada vez mayores mientras permanece inactivo. La grabación que se esté realizando en él continúa en el mismo `.daq` con la interrupción marcada en el archivo, y `pool-record --stop` enumera cada interrupción. En la versión 1.2.0, el sensor permanecía inactivo hasta `pool-disconnect` + `pool-connect`, lo que también ponía fin a la grabación. Véase [Un DAQ-E que deja de enviar datos](#a-daq-e-that-stops-delivering).
> - **`time-sync status`** indica si los paquetes del «grandmaster» PTP llegan a la red (`health`); si no es así, se muestra un `WARNING:` en stderr. Véase [`time-sync`](#chloros-cli-time-sync).
> - **Paquete .deb para Linux:** aumenta los búferes de recepción de sockets del kernel que necesitan las transmisiones de las cámaras (con los valores predeterminados del kernel, se pierden tramas y se reduce la velocidad de fotogramas), añade un servicio de fondo que se ejecuta como tu usuario y `lattice network` comprueba los búferes. La CLI nunca detiene un backend que se ejecuta como servicio, y una actualización reinicia los servicios de backend habilitados. Véase [Ejecutar el backend como servicio](#running-the-backend-as-a-service-headless) y [Hosts Linux: búferes de recepción de sockets](#linux-hosts--socket-receive-buffers).
> - **`lattice array-capture` indica cuándo no se ha podido alinear:**cada motivo por el que se ha guardado una captura sin alinear (ausencia de perfil de alineación, miembros con resoluciones diferentes, una matriz no sincronizada con el cable, un fotograma al que no se puede asignar el perfil) se muestra como una línea `Alignment:`.**`chloros-cli daq help`** es `daq --help` (salió de `2`), y en Linux la ayuda enumera los comandos `daq` de acceso directo al hardware que incluye la compilación.
> - **Jetson:** el paquete es `chloros_1.3.0_arm64_jp6.deb`, para JetPack 6, y su procesamiento por GPU se ejecuta en el Orin con CUDA PyTorch de JetPack. Véase [Linux arm64 — Jetson](#linux-arm64--jetson-jetpack-6).

---

## Convenciones

- Todos los comandos llevan el prefijo `chloros-cli`. En Windows, el binario es `chloros-cli.exe`; en Linux/Jetson es `chloros-cli`.
- Los argumentos opcionales se muestran como `--flag`. Los argumentos posicionales obligatorios se muestran sin corchetes.
- Cuando se proporciona un valor por defecto, si se omite el indicador, se utiliza ese valor.
- La CLI es un cliente HTTP ligero sobre el backend de Chloros (el servidor en `127.0.0.1:5000` que también utiliza la aplicación de escritorio). **¿Qué comandos la inician?**
  - Los comandos principales — `process`, `login`, `logout`, `status`, `export-status`, `time-sync` y `selftest` — utilizan un backend en ejecución, o inician uno cuando no hay ninguno en marcha y lo detienen de nuevo cuando terminan.
  - **Los comandos `lattice`, `project` y `daq` nunca inician uno.** Necesitan un backend que ya esté en ejecución: el de la aplicación de escritorio de Windows, el [servicio de backend](#running-the-backend-as-a-service-headless) de Linux, o uno que inicies tú mismo (consulta [Iniciar el backend](#start-the-backend)). Sin un backend, muestran el mensaje `Error: Chloros backend not reachable at http://127.0.0.1:5000`, junto con el comando que lo inicia, y se cierran con el mensaje `1`.
- `CHLOROS_BACKEND_URL=<url>` redirige las familias de comandos **`lattice`**,**`project`**y**`daq pool-*`** hacia otro backend (`lattice network-analysis`, por su parte, las redirige a `--backend-url`). Los comandos principales fijan deliberadamente `http://127.0.0.1:<port>` y lo ignoran (el literal IPv4 evita la penalización de Windows de `localhost`→`::1` de unos 2 s por solicitud). Véase [Variables de entorno](#environment-variables).
- **Las rutas relativas se resuelven en función de la carpeta desde la que se ejecuta `chloros-cli`.** El backend es el proceso que abre las carpetas de salida y entrada, por lo que, antes de enviar una ruta, la CLI la convierte en absoluta: `-o` / `--output` (incluido su valor por defecto, `output`), `--input`, `--burst-dir`, `--profile`, `--save` y `--mask`, un paso [receta](#receta-dsl) `output` (por defecto, `output`), y la carpeta predeterminada `output` de `lattice index --live`. Por lo tanto, `-o captures/` escribe en `<current folder>/captures/`, y las rutas impresas son absolutas. `~` se expande. En el caso de un backend remoto (`CHLOROS_BACKEND_URL` que hace referencia a otra máquina), las rutas se envían tal y como se han escrito y se resuelven en esa máquina.
- Se requiere iniciar sesión con una cuenta de Chloros+ para todas las llamadas SDK/CLI (ejecuta `chloros-cli login` una vez por máquina; se almacena en caché en `~/.chloros/`).
- Los ejemplos utilizan rutas de Linux; en Windows, sustituya `/home/user/...` por `C:/Users/.../...`. El puerto serie de un DAQ-U&#x27;es del tipo `COM3` en Windows y `/dev/ttyUSB0` / `/dev/ttyACM0` en Linux.
- Las sinopsis escriben los indicadores mutuamente excluyentes como `{--a | --b}` y las opcionales como `[...]`.

---

## Sinopsis de nivel superior

```
chloros-cli [global options] COMMAND [command options]
```

### Opciones globales

| Indicador | Descripción |
| --- | --- |
| `--backend-exe PATH` | Anula el ejecutable del backend detectado automáticamente. |
| `--port N` | Puerto HTTP del backend (por defecto: `5000`). Solo lo utilizan los comandos principales (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`); `lattice`, `project` y `daq` utilizan `CHLOROS_BACKEND_URL` (`http://127.0.0.1:5000` por defecto). |
| `-v, --verbose` | Habilita la salida detallada. |
| `--restart` | Forzar el reinicio del backend (termina cualquier proceso `backend_server.py` en ejecución). Un backend ejecutado por un servicio de systemd (.deb de Linux) nunca se detiene: en su lugar, el comando muestra el mensaje `sudo systemctl restart <unit>` y sale `1`. |
| `--version` | Muestra la versión (`Chloros CLI 1.3.0`). |
| `--help` | Muestra la ayuda de primer nivel. |

### Índice de comandos

| Comando | Finalidad |
| --- | --- |
| [`process`](#chloros-cli-process) | Procesar de principio a fin una carpeta de capturas de Survey3 o LATTICE. |
| [`login`](#chloros-cli-login) | Autenticar este equipo con una cuenta de Chloros+. |
| [`logout`](#chloros-cli-logout) | Borrar las credenciales almacenadas en la caché. |
| [`status`](#chloros-cli-status) | Mostrar el estado actual de la licencia y la autenticación. |
| [`export-status`](#chloros-cli-export-status) | Progreso en tiempo real de la exportación de Thread-4 durante una ejecución de `process`. |
| [`language`](#chloros-cli-language) | Establecer o mostrar el idioma de visualización de la CLI (38 idiomas compatibles). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | Carpeta de proyecto predeterminada (compartida con la interfaz gráfica de usuario). |
| [`update`](#chloros-cli-update) | Buscar e instalar actualizaciones de Chloros (Linux/Jetson). |
| [`selftest`](#chloros-cli-selftest) | Diagnóstico del sistema + pruebas de funcionamiento básico. |
| [`time-sync`](#chloros-cli-time-sync) | Estado y control del «grandmaster» PTP. |
| [`lattice`](#chloros-cli-lattice) | Control y captura de la cámara LATTICE (más de 45 subcomandos). |
| [`daq`](#chloros-cli-daq) | Control de sensores espectrales DAQ (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Abrir y ejecutar un proyecto de Chloros guardado (cámaras + DAQ). |

---

## Instalación

`chloros-cli` se incluye en el instalador de escritorio de Chloros en todas las plataformas compatibles — no hay una descarga independiente de la CLI. Al instalar el paquete de la plataforma, se añade `chloros-cli` a tu `PATH` junto con la aplicación de escritorio y el binario de fondo que controla.

Últimas descargas: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> El instalador también incluye scripts de inicio prácticos (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`) que abren un shell de CLI listo para usar.

### Windows (.exe)

1. Descarga el instalador para Windows desde la página de descargas.
2. Ejecuta `Chloros Installer 1.3.0.exe` y sigue las instrucciones del asistente. La ruta de instalación predeterminada es `C:\Program Files\Chloros\` (la CLI se encuentra en `C:\Program Files\Chloros\cli\`, que el instalador añade a la variable PATH).
3. Abre un nuevo terminal (`cmd.exe`, PowerShell o Terminal de Windows) para que se detecte el `PATH` actualizado.

```powershell
chloros-cli --version
```

El instalador añade automáticamente `chloros-cli.exe` a tu sistema `PATH` e incluye los controladores de la cámara LATTICE. El backend es `C:\Program Files\Chloros\backend\chloros-backend.exe` (la carpeta `backend` de tu carpeta de instalación), que no se encuentra en `PATH`.

**Configuración del sistema.** La página *Seleccionar tareas adicionales* del instalador tiene dos opciones en **Configuración del sistema**, ambas**marcadas por defecto**:

| Opción (tal y como aparece en el instalador) | Qué hace | Desactivar la instalación silenciosa |
| --- | --- | --- |
| **Habilitar rutas largas de Windows** (permite que las carpetas de proyecto tengan más de 260 caracteres; recomendado) | Establece el valor de `LongPathsEnabled` de Windows para todo el equipo en 1. Windows viene de fábrica con esta opción desactivada, por lo que ninguna ruta puede superar los 260 caracteres; los productos de Chloros se ubican hasta unos 130 caracteres por debajo de la carpeta del proyecto, por lo que se rechaza una carpeta de proyecto más profunda (véase [Solución de problemas](#consejos-de-resolución-de-problemas)). Surte efecto en los programas iniciados tras la instalación. Se mantiene activado tras la desinstalación, ya que otros programas pueden depender de ello. | `/MERGETASKS="!longpaths"` |
| **Excluir el backend de Chloros del análisis de Microsoft Defender**(captura de cámara mucho más rápida; recomendado) | Añade una exclusión de**proceso** de Defender para `chloros-backend.exe` en la carpeta de instalación, de modo que la protección en tiempo real no analice cada archivo que escriba el backend, independientemente de dónde lo escriba. No se excluye ningún otro programa; la exclusión se elimina al desinstalar. Si Defender no está presente o está bloqueado por una política, el paso falla de forma silenciosa y la instalación continúa. | `/MERGETASKS="!defenderexclusion"` |**Por qué es importante la opción de Defender:** a altas tasas de captura, el análisis en tiempo real de los archivos de captura supone aproximadamente un 30 % de la tasa de captura (medido en una captura continua con 9 cámaras: 77,7 capturas/s excluidas, 52,7 analizadas). `lattice array-capture --continuous` muestra un mensaje cuando detecta que Defender utiliza un núcleo de CPU o más durante la captura; vuelve a ejecutar el instalador con la opción marcada.

Una instalación silenciosa (`/SILENT` o `/VERYSILENT`) aplica ambas opciones a menos que se desactive, p. ej., `"Chloros Installer 1.3.0.exe" /VERYSILENT /MERGETASKS="!defenderexclusion"`.

**Firewall de Windows.** El instalador siempre añade dos reglas de entrada para `chloros-backend.exe`, en cada perfil de red, y el desinstalador las elimina: `Chloros Backend (UDP-In)`, para flujos de cámara, PTP, mDNS y el flujo de datos del DAQ-E, y `Chloros Backend (TCP-In)`, limitada a la subred local, para una actualización del firmware del DAQ-E, en la que el sensor se vuelve a conectar a este ordenador para recopilar la imagen (véase [Configuración del dispositivo DAQ-E](#daq-e-device-settings)). La API HTTP del backend permanece vinculada a `127.0.0.1`.

### Linux amd64 (.deb)

Para estaciones de trabajo con Ubuntu 22.04 LTS o posterior, o basadas en Debian, con arquitectura x86_64.

> **Ubuntu 20.04 no es compatible.** La lista de dependencias del paquete se deriva de
> aquello con lo que el backend se vincula realmente, y eso incluye `libc6 (>= 2.35)`;
> Focal incluye glibc 2.31. `apt` rechaza la instalación en lugar de permitir que falle en
> tiempo de ejecución. La lista también incluye `libgl1`, que carga el OpenCV incluido y del que puede carecer una
> imagen de servidor mínima; `apt install` lo aporta.

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
```

El archivo .deb instala:
- `chloros-cli` a `/usr/bin/chloros-cli`, y `chloros-backend` (inicia el backend) hasta `/usr/bin/chloros-backend`
- El backend y la CLI compilados, descomprimidos, en `/usr/lib/chloros/backend/` y `/usr/lib/chloros/cli/`
- Los controladores de la cámara LATTICE
- Modelos de eliminación de ruido, paquetes de calibración y configuración del canal de actualización
- El SDK Python para el sistema `python3` (`import chloros_sdk`). `apt install` también incluye sus propios requisitos, `python3-numpy` y `python3-requests` (recomendados, no obligatorios: los programas anteriores son autónomos; `dpkg -i` los omite, y el resultado de la instalación indica qué falta). En un virtualenv: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`
- `/etc/sysctl.d/60-chloros-gige.conf`, que aumenta los búferes de recepción de sockets del kernel para los flujos de la cámara a 32 MB y los aplica de una sola vez (si ya se ha establecido un valor superior, no se modifica). Al desinstalar el paquete se elimina el archivo. Véase [Hosts Linux: búferes de recepción de sockets](#linux-hosts--socket-receive-buffers).
- Dos servicios opcionales de systemd para el backend, ninguno de ellos habilitado (siguiente sección)

#### Ejecución del backend como servicio (sin interfaz gráfica)

Sin un servicio, el backend (y con él el «grandmaster» PTP) solo se ejecuta mientras haya algo que lo haya iniciado: `chloros-backend` en un terminal, un script de SDK o un comando principal `chloros-cli` durante el tiempo que dure su ejecución. Para disponer de un backend siempre activo en una máquina sin interfaz gráfica, activa el servicio **como tu usuario**:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service` ejecuta el backend como ese usuario. Las capturas que escribe son de ese usuario, su registro detallado se encuentra en su `~/.cache/chloros/logs`, y utiliza su `chloros-cli login` (`~/.chloros`), así que primero inicia sesión una vez como ese usuario. PTP no necesita privilegios de root. Al igual que cualquier backend instalado, escucha únicamente en el puerto `127.0.0.1:5000`.

`chloros-backend.service` ejecuta el backend como **root**. Sigue estando instalado para que un equipo que lo haya habilitado siga funcionando, pero todo lo que escribe un backend como root pertenece a root, por lo que un usuario no puede eliminar sus propias capturas. Para cambiar a tu usuario:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

El registro detallado del servicio de root se encuentra en `/var/log/chloros`, al que pueden acceder los miembros del grupo `adm` (el servicio establece `CHLOROS_LOG_DIR`; véase [Variables de entorno](#environment-variables)). Al instalar el paquete se muestra el comando `enable` con tu nombre de usuario ya introducido, así como el modificador cuando el servicio root está habilitado.

**Habilita una unidad de backend.** Solo se ejecuta un backend a la vez. Una segunda unidad habilitada (`chloros-backend.service` junto a `chloros-backend@<user>.service`, o dos unidades `@` de distintos usuarios) se niega a iniciarse y permanece como `failed` con un estado de salida 98; systemd no la reinicia. Su `systemctl status` explica el motivo: `Another instance is already running. Exiting.`, es decir, lo que retiene el backend (el puerto 5000, o el archivo de bloqueo y el usuario al que pertenece), y, a continuación, `systemd will not restart <unit> (exit status 98)`. Tras un reinicio, cualquiera de las dos unidades puede ser la que se ejecute. Para cambiar, desactiva la que no utilices, tal y como se ha indicado anteriormente.**La CLI nunca detiene el backend de un servicio.** `chloros-cli` lo utiliza tal cual, al igual que la GUI y el SDK: `process` se ejecuta en él, cargando su propio proyecto en lugar del último. En los casos en los que, de otro modo, la CLI. reiniciaría el backend (`--restart`, o un `process` cuya importación no haya encontrado nada), nombra la unidad, muestra «`sudo systemctl restart <unit>`» y sale de «`1`», ya que un backend detenido de esa forma permanecería inactivo. `GET /api/status` nombra la unidad como `service` (`null` en el caso de un backend en el que no se ejecute ningún servicio).**Actualización y eliminación.** La actualización del paquete detiene todos los servicios de backend y, a continuación, reinicia cada uno de los que están habilitados, empezando por las unidades `@<user>` (uno iniciado manualmente pero nunca habilitado permanece detenido, como tras un reinicio). Si hay más de uno habilitado, solo se ejecuta uno y los demás permanecen en estado de fallo como se ha indicado anteriormente; la actualización muestra los mensajes `Chloros: N backend units are enabled: …` y `Keep one enabled: sudo systemctl disable --now <unit>`. Al desinstalarlo, se detienen y se deshabilitan. Al purgarlo, también se elimina `/var/log/chloros`.

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Mismo diseño que el .deb para amd64, incluye búferes de recepción y [servicios de backend](#running-the-backend-as-a-service-headless), para Jetson Orin / Orin NX / Orin Nano en **JetPack 6** (L4T R36). Su PyTorch es la compilación CUDA de JetPack 6 para Orin, por lo que el procesamiento por GPU —incluido el debayer con reconocimiento de texturas— se ejecuta en la GPU del Jetson. El paquete no incluye CUDA en sí mismo: utiliza las bibliotecas CUDA y cuDNN que instala JetPack 6 (`sudo apt install nvidia-jetpack` en una imagen que carece de ellas). Cuando no se puede utilizar una GPU con CUDA, el procesamiento se ejecuta en la CPU y la función «texture-aware» utiliza el «debayer» estándar (véase [`process`](#chloros-cli-process)).

### Autenticación única por máquina

Cada plataforma requiere un inicio de sesión único en Chloros+ antes de que funcionen las llamadas al SDK o a la CLI:

```bash
chloros-cli login user@example.com 'YourPassword'
```

Las credenciales se almacenan en caché en `~/.chloros/user_session.json`.

### Iniciar el backend

Los comandos `lattice`, `project` y `daq` se comunican con un backend que ya está en ejecución; nunca inician uno (véase [Convenciones](#convenciones)). Mantén uno en ejecución mientras los utilizas:

| Plataforma | Cómo |
| --- | --- |
| Windows | Abre la aplicación de escritorio de Chloros (su backend también da servicio a la CLI), o ejecuta `"C:\Program Files\Chloros\backend\chloros-backend.exe"` en una terminal propia (la carpeta `backend` de tu carpeta de instalación) y déjala abierta. |
| Linux/Jetson | Ejecuta `chloros-backend` en un terminal independiente, o mantén uno siempre en ejecución con el [servicio de fondo](#running-the-backend-as-a-service-headless). |

Solo se ejecuta un backend a la vez; todos los clientes (la interfaz gráfica, la CLI y los scripts del SDK) lo comparten. Los comandos principales (`process`, `login`, `status`, …) también lo utilizan, e inician uno cuando no hay ninguno en ejecución.

**Un backend sigue ejecutándose entre comandos**, por lo que el siguiente no tiene que volver a iniciarlo. Se detiene cuando cierras sesión (`chloros-cli logout`, a menos que la aplicación Chloros esté abierta o se esté realizando una ejecución o una compilación sin conexión), cuando ejecutas `chloros-cli backend stop`, o de forma automática tras 30 minutos sin que nada lo utilice: sin comandos, scripts ni aplicaciones, sin ejecuciones ni compilaciones sin conexión, sin cámaras conectadas ni sensores de luz. `backend stop` se niega mientras se está realizando una ejecución o una compilación sin conexión, indicando el motivo; `--force` lo detiene inmediatamente y esa ejecución o compilación se pierde (se conserva una ráfaga, y `array-build-video` la vuelve a compilar). Configura `CHLOROS_BACKEND_IDLE_MIN` antes de que se inicie el backend para modificar los 30 minutos (`0`: nunca); el [servicio del backend](#running-the-backend-as-a-service-headless) nunca se detiene por sí solo. `chloros-cli backend status` muestra si hay alguno en ejecución, qué lo está utilizando y si hay alguna compilación sin conexión en marcha. `process` abre su proyecto en el backend en ejecución, a menos que la aplicación Chloros esté abierta con un proyecto que : en ese caso, rechaza la operación, y debes procesarlo desde la aplicación o cerrar primero ese proyecto. Un proyecto que un `process` anterior o un script de SDK haya dejado abierto simplemente se sustituye.

### Verificar la instalación

```bash
chloros-cli --version           # prints "Chloros CLI 1.3.0"
chloros-cli selftest            # 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

`selftest` sale de `0` solo cuando se superan las siete comprobaciones. La última requiere una GPU CUDA (una GPU NVIDIA, o de un Jetson Orin) y los modelos de eliminación de ruido; por lo tanto, en un ordenador que no disponga de ella, muestra el mensaje `Failed: CUDA + Denoiser` y sale de `1`; en ese caso, todo sigue funcionando excepto el debayer sensible a texturas, que solo funciona con GPU.

> **Se requiere suscripción a Chloros+.**La CLI requiere un plan Chloros+ activo.**Copper**es el nivel básico de Chloros+; todos los niveles de pago de Chloros+ tienen acceso a la CLI y al SDK; solo el nivel gratuito**Iron** no lo tiene. (Correspondencia de ID de planes: `0`=Iron/gratuito, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) Actualización en [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing).
>
> Este límite mínimo lo aplica el backend, no solo la CLI: una solicitud con indicadores de SDK/CLI que no cuente con un plan de pago se rechaza con el código `403 PLAN_UPGRADE_REQUIRED`, independientemente de si procede de `chloros-cli`, del Python SDK o de un cliente HTTP creado manualmente. En cambio, un usuario que haya cerrado sesión recibe el código `401 AUTH_REQUIRED`. El acceso funciona sin conexión durante el periodo de gracia del plan (30 días para los planes mensuales y hasta su vencimiento para los anuales) y se interrumpe cuando este vence; el código `chloros-cli status` sigue funcionando para que el motivo sea visible (es la única ruta de SDK/CLI exenta del control de niveles — `GET /api/license-status`).

---

## Guía de inicio rápido en cinco minutos

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI GNDVI OSAVI

# 3. The lattice / daq commands need a running backend: on Windows open the
#    Chloros desktop app; on Linux start one (or see "Start the Backend")
chloros-backend &

# 4. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 5. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 6. Read a spectrum from a connected DAQ-U (Windows: --port COM3)
chloros-cli daq pool-connect --port /dev/ttyUSB0
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

`process` (paso 2) inicia un backend para su propia ejecución cuando no hay ninguno en marcha y lo detiene cuando termina, así que inicia uno a partir del paso 4. Consulta [Iniciar el backend](#start-the-backend).

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
| `INPUT` | Ruta a la carpeta de entrada que contiene los archivos `.raw + .jpg` (Survey3), `.tif/.tiff` (LATTICE) o `.dng`. |

### Opciones comunes

| Opción | Valor predeterminado | Descripción |
| --- | --- | --- |
| `-o, --output PATH` | una nueva carpeta con el nombre de `-n` en la carpeta predeterminada del proyecto | La carpeta del proyecto. El proyecto recibe el mismo nombre, y una carpeta que ya contenga un `project.json` obtendrá un `_1` / `_2` en lugar de sobrescribirse. La carpeta predeterminada del proyecto es la elegida por `set-project-folder` (o la interfaz gráfica de usuario); de lo contrario, es `Documents/MAPIR/Chloros Projects`. |
| `-n, --project-name NAME` | automático (marca de tiempo) | Nombre del proyecto, utilizado únicamente **sin** `-o` (con `-o`, el proyecto toma el nombre de la carpeta). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` utiliza un desmosaico neuronal de Chloros+ para imágenes de Survey3; más lento, pero de mayor calidad. Es solo para Survey3: Las capturas de LATTICE siempre utilizan el demosaico estándar, por lo que, en el caso de una carpeta que contenga capturas de LATTICE, `process` rechaza `texture-aware` tras la importación, mostrando un error en el que se indica cuántas hay, y sale de `1` antes de procesar nada (la aplicación Chloros tampoco ofrece la función «Texture Aware» para ellas). Requiere una GPU CUDA (una GPU NVIDIA en Windows / Linux x64, la GPU del Orin en un Jetson): sin ella, la ejecución utiliza `standard`, y el backend registra una línea `[DEBAYER] … needs a CUDA GPU …` explicando el motivo (mostrada con `--verbose`). |
| `--vignette / --no-vignette` | `--vignette` | Corrección de viñeteado. |
| `--reflectance / --no-reflectance` | `--reflectance` | Calibración de reflectancia (utiliza el panel de referencia si se encuentra; calibración NIST por serie para LATTICE). Para LATTICE multiespectral, esto también actúa como el **producto** de reflectancia — véase [Opciones de exportación por producto](#per-product-export-toggles-lattice-multispectral). |
| `--ppk` | desactivado | Aplica correcciones GNSS PPK a partir de archivos sidecar. |
| `--exposure-pin-1 CAMERA_MODEL` | ninguno | El modelo de cámara conectado al pin de exposición 1, para la sincronización temporal PPK. |
| `--exposure-pin-2 CAMERA_MODEL` | ninguno | El modelo de cámara conectado al pin de exposición 2, para la sincronización temporal PPK. |
| `--recal-interval SECONDS` | `0` | Intervalo mínimo de recalibración, en segundos (el *intervalo mínimo de recalibración* del proyecto). |
| `--timezone-offset HOURS` | `0` | Diferencia horaria del sensor de luz respecto al UTC, en horas (la *Diferencia horaria del sensor de luz* del proyecto). |
| `--format FORMAT` | `TIFF (16-bit)` | Uno de los siguientes: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)`. Chloros nunca aumenta la profundidad de bits de una imagen: en imágenes de 8 bits, un formato de mayor profundidad se guarda como `TIFF (8-bit)` y el programa lo indica una vez. En un proyecto que combine capturas de 8 bits y de 12/16 bits, cada captura se exporta con su propia profundidad o una inferior. La radiancia es siempre de 32 bits en coma flotante. |
| `--indices NAME [NAME ...]` | ninguna | Índices de vegetación por nombre de preajuste (`NDVI`, `GNDVI`, `OSAVI`, `SAVI`, `EVI`, `GCI`, …), sin distinción entre mayúsculas y minúsculas. Cada uno se añade al proyecto para las bandas de su cámara, incluyéndose un nuevo proyecto. Un nombre que no sea un preajuste, o que las bandas de la cámara no puedan generar (`NDRE` necesita RE y NIR, que ninguna cámara proporciona por sí sola), aparece en una advertencia y no recibe ninguna imagen índice. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Forzar el punto de entrada del proceso para los archivos TIFF de LATTICE (los archivos .raw de Survey3 no se ven afectados). También la vía de escape que permite que una captura **sin raw** se procese de todos modos; véase [Cómo es una carpeta de capturas](#how-a-captures-folder-looks-like). |
| `--debayered / --no-debayered` | activado | Emite el producto con debayering lineal (`Debayered_Images`). Véase [Opciones de exportación por producto](#per-product-export-toggles-lattice-multispectral). |
| `--preview / --no-preview` | activado | Emite la vista previa en pantalla (`Preview_Images`): RGB = balance de blancos (iluminante DAQ cuando esté disponible; en su defecto, «gray-world») + gamma; multiespectral = estiramiento a colores falsos. |
| `--radiance / --no-radiance` | activado | Emite la radiancia en float32 (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | Referencia para el producto de reflectancia LATTICE: `auto` = el objetivo dentro del marco que ha superado el control de calidad (QA) es la referencia absoluta (un objetivo con un indicador de QA compite con el DAQ en la puntuación medida), con el valor de reserva del DAQ descendente (ρ = π·L/E); `target` = estricto (sin sustitución del DAQ); `daq` = con autoridad del DAQ. Véase [Opciones de exportación por producto](#per-product-export-toggles-lattice-multispectral). |
| `--target-reflectance-dir DIR` | ninguno | Directorio de escaneos de reflectancia objetivo **medida** por unidad (`<serial>.csv`); recurre a los espectros T3/T4P nominales en caso de fallo. |
| `--array-alignment / --no-array-alignment` | activado | Matrices LATTICE: aplica la alineación entre módulos indicada en el archivo XMP `Chloros:Alignment*` de cada captura a todos los productos procesados (desbayering / vista previa / radiancia / reflectancia / índice). No realiza ninguna operación en imágenes sin estas etiquetas. |
| `--array-alignment-crop / --no-array-alignment-crop` | recortar | Recorta las exportaciones alineadas a la región de solapamiento común de la matriz para que todos los módulos compartan una misma huella; `--no-…` conserva el lienzo completo del sensor (relleno negro fuera de la fuente). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Remuestreo para la deformación de alineación. `nearest` conserva los valores de DN exactos de la fuente (sin mezcla entre píxeles de los valores radiométricos). |

### Opciones de detección de objetivos

| Indicador | Descripción |
| --- | --- |
| `--min-target-size PIXELS` | Tamaño mínimo en píxeles de los objetivos de calibración para el detector. |
| `--target-clustering 0-100` | Porcentaje de agrupamiento de objetivos (por defecto `60`). |
| `--target / --targets` | Buscar objetivos de calibración únicamente en una subcarpeta `target` o `targets` de la entrada (más rápido). |

### Ejemplos

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI GNDVI OSAVI

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

# Save to a project folder of your choice (the project is named after it)
chloros-cli process "C:/input" -o "C:/projects/Field_A_2026-05-26"

# Name the project and let it go to the default project folder
chloros-cli process "C:/input" -n "Field_A_2026-05-26"
```

### Opciones de exportación por producto (LATTICE multiespectral)

El procesamiento de LATTICE se distribuye en **todos los productos aplicables en una sola pasada**. Las cuatro opciones por tipo — `--debayered`, `--preview`, `--radiance` y `--reflectance` — están**activados por defecto**; utiliza el formulario `--no-<type>` para desactivar uno. Las cámaras maestras RGB solo emiten imágenes sin debayering + vista previa (sin radiancia/reflectancia por banda), por lo que `--radiance`/`--reflectance` no tienen ningún efecto en ellas. Los conmutadores se ignoran para `.raw` de Survey3 (que sigue la ruta estándar de reflectancia/objetivo). *(El antiguo indicador `--radiometric-output {reflectance,radiance,sensor-response}` **se ha eliminado** y sustituido por estos conmutadores; ya no existe el nivel `sensor-response`.)*

| Producto | Salida | ¿Se necesita DAQ descendente? |
| --- | --- | --- |
| `--debayered` | Desmosaico lineal (`Debayered_Images`): el mismo archivo que guarda una captura en su nivel `debayered` — el desplazamiento oscuro (DSNU) del paquete de calibración aplicado a los datos sin procesar, seguido del demosaico sensible a los bordes de OpenCV; sin LUT de linealidad, campo plano ni desmezclado. | No. |
| `--preview` | Vista previa en pantalla (`Preview_Images`): RGB = balance de blancos + gamma; multiespectral = estiramiento a colores falsos. | No. |
| `--radiance` | float32 W/m²/sr/nm procedente de la cadena radiométrica completa (`Radiance_Images`). | N.º |
| `--reflectance` | reflectancia ρ en uint16 (`32768` = 1,0), Compatible con Pix4D. | **Sí**, a menos que lo ancle un objetivo dentro del fotograma que haya superado el control de calidad (véase más abajo). |

`--reflectance-source` selecciona la referencia de reflectancia:**`auto`**(por defecto) convierte un objetivo dentro del fotograma que haya superado el control de calidad en la**referencia absoluta**— las cadenas de líneas empíricas ancladas al objetivo se comparan con los paneles no utilizados y se aplica el resultado ganador, banda por banda: una banda que el DAQ no pueda calibrar, o cuyo ajuste de pendiente del DAQ falle, toma la línea del propio objetivo (`ReflectanceElBandMethods` indica cuál) — un objetivo que supera el control de calidad con un indicador de error (un panel descartado, una banda fuera de servicio) también compite con la división del DAQ en esa puntuación medida, y el fotograma al que pierde se marca como `ReflectanceChain=daq` — se recurre a la división descendente del DAQ (ρ = π·L/E) cuando no hay ningún objetivo presente o falla el control de calidad;**`target`**es estricto (sin sustitución del DAQ);**`daq`**opta por el comportamiento autoritativo del DAQ. La geometría del objetivo (ArUco / ROI fijo / franja) proviene de la configuración del objetivo del proyecto; `--target-reflectance-dir DIR` mantiene los**escaneos medidos** por unidad (`<serial>.csv`), consultados mediante el número de serie o el código QR de la unidad de destino, con los espectros T3/T4P nominales como alternativa.

La ruta de reflectancia del DAQ resuelve automáticamente la **radiación descendente con marca de tiempo coincidente**a partir de un**`.daq`**(DAQ-U/M/E)**o un `.csv` nativo de DAQ-M**que se encuentre junto a las imágenes. Si no hay un paquete de calibración por cámara o por DAQ almacenado en la caché local, el proceso**lo recupera automáticamente de AWS** la primera vez que se utiliza (requiere conexión a Internet una vez; se almacena en caché en `~/.chloros/`).**¿De quién es la lectura?**Un `.daq` que guarda una captura es la lectura de la(s) cámara(s) que describe —nombrada en un archivo sidecar `<file>.daq.cameras.json` junto a ella (`lattice capture`, `multi-capture`, `array-capture` y la interfaz gráfica de usuario; una ráfaga sin procesar indica la lectura de cada miembro en su manifiesto) — y `process` la asocia**únicamente a imágenes de esas cámaras**. Por lo tanto, una cámara independiente que no tenga lecturas propias ya no utiliza el DAQ de otra cámara; hay que asignarle un DAQ o grabar una. La propia grabación de un DAQ (`daq pool-record`, o una grabación iniciada en la interfaz gráfica de usuario), o cualquier `.daq` sin ese archivo complementario (incluidas las capturas más antiguas), sigue sirviendo para todas las cámaras.

#### Lectura de píxeles de reflectancia (Pix4D / Metashape / tus propios scripts)

La reflectancia se almacena como un valor DN entero, y **el valor DN que corresponde a ρ = 1,0 depende de la cámara de origen**:

| Origen | ρ = 1,0 es | Cómo identificarlo |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (margen hasta ρ 2,0) | El archivo lleva estampado el XMP `Chloros:PixelScale=32768`. |
| Survey3 | `65535` (recortado en ρ 1,0) | No hay etiquetas XMP `Chloros:*`: esa ausencia *es* la señal. |

**Lee `Chloros:PixelScale` y divide por ese valor** en lugar de suponer una constante. La etiqueta está definida en el dominio uint16, por lo que se mantiene como `32768` en todos los formatos de salida que se reescalan — `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)` y `TIFF (32-bit, Percent)` son todos autodescriptivos (normaliza primero el tipo de datos almacenado a uint16: ×257 desde 8 bits, ×65535 desde float).

> **Los archivos de versiones anteriores pueden no tener escala.** Un archivo TIFF de reflectancia de 8 bits se reescala y se le asigna el nombre `Chloros:PixelScale`, al igual que los demás. Las versiones anteriores a la 1.3.0, en cambio, lo *recortaban* a 0..255, por lo que todos los valores superiores a ρ≈0,008 se aplanaban a 255, y omitían tanto `Chloros:PixelScale` como la tupla `MicaSense:RadiometricCalibration`. **Si la etiqueta no está presente en un archivo de reflectancia LATTICE, no se debe suponer una escala: hay que volver a procesarlo** en lugar de dividir píxeles que nunca fueron divisibles.

#### Lectura de píxeles de índice (NDVIy los demás índices de vegetación)

**Una trama de índice entera es un estiramiento de contraste por imagen, no un mapa de −1…1.** `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)` y `JPG (8-bit)` —incluido el archivo TIFF de 8 bits en el que se guarda una captura de 8 bits— almacenan

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

donde `IndexMin` / `IndexMax` son **los valores de índice finitos más bajo y más alto de ese fotograma**. DN 0 no es −1, y la escala completa no es +1.

Chloros marca ambos extremos para que la correspondencia siga siendo invertible:

| Etiqueta | Significado |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` — la convención que se indica a continuación |
| `Chloros:IndexMin` | el valor de índice almacenado como DN 0 |
| `Chloros:IndexMax` | el valor de índice almacenado como escala completa |

Léelas e invierte los valores, normalizando primero el DN almacenado a un rango de 0…1 según el máximo del tipo de datos almacenado, de modo que una sola expresión cubra todos los formatos de enteros:

```python
import json, subprocess
import numpy as np, tifffile

tags = json.loads(subprocess.run(
    ["exiftool", "-j", "-XMP:all", path],
    capture_output=True, text=True).stdout)[0]

# float(): exiftool hands back a negative real as a quoted string, so
# IndexMin arrives as str on most real NDVI data while IndexMax is a number.
lo, hi = float(tags["IndexMin"]), float(tags["IndexMax"])

raw = tifffile.imread(path)                    # uint16 for TIFF (16-bit)
ndvi = lo + raw.astype(np.float64) / np.iinfo(raw.dtype).max * (hi - lo)
```

La lectura no necesita `-config pix4d.config`: exiftool lee estas propiedades XMP por sí solo. La configuración solo es necesaria para *escribirlas*.

> **`IndexMin == IndexMax`** significa que cada píxel de ese fotograma tenía el mismo valor de índice. La trama se rellena a media escala y no hay nada por lo que dividir; toma el valor extremo.

> **La ausencia de etiquetas `Chloros:Index*` significa que no hay una correspondencia especificada.** Una exportación LUT es un renderizado coloreado cuyos píxeles son colores degradados vinculados a la ventana de umbral de tu LUT, y un fotograma sin píxeles de índice finito no tiene ningún rango que especificar. No des por sentado que existe una asignación en ninguno de los dos casos: se aplica la misma regla que cuando falta `PixelScale`.**Dado que el estiramiento es por imagen, dos fotogramas de un mismo vuelo no son comparables en términos de DN**: cada uno tiene sus propios extremos, por lo que el mismo NDVI se sitúa en diferentes valores de DN en distintas imágenes. Para trabajos cuantitativos a lo largo de un vuelo (estadísticas zonales, umbrales fijos, series temporales), exporta**`TIFF (32-bit, Percent)`**: las mallas de índice se escriben allí como float32**los propios valores de índice**, sin necesidad de reescalado ni etiquetas. A pesar del nombre del formato, los productos de índice no son ×100; esa escala solo se aplica a la reflectancia de tipo de datos entero.

#### Máscara de limitación de la separación espectral (en qué píxeles se ha limitado una banda)

La separación espectral 3×3 puede hacer que una banda resulte negativa —cuando la luz apenas contiene esa banda, o cuando la escena queda fuera de lo que describe la matriz de separación de la cámara—. Como no se trata de un valor real, la exportación lo fija en 0, y la vista en directo muestra ese mismo 0. Una banda puede estar limitada en unos pocos píxeles o en casi todos ellos; la máscara registra cualquiera de las dos situaciones. `Chloros:Band{R,G,B}UnmixClampFraction` indica en qué parte del fotograma se produjo; la máscara de recorte indica **qué píxeles**:

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— en una carpeta `clamp_masks/`**junto a** la carpeta del producto, nunca dentro de ella (`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`; una exportación de `process`: `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif`) — para cada producto de radiancia, reflectancia e índice en la que al menos un píxel haya sido recortado: tanto capturas como exportaciones `process` (y los TIFF por fotograma de `array-build-video --save-tiffs`). `uint8` de banda única, en la propia cuadrícula de píxeles del producto (incluidas la orientación, la región de interés, el binning y la alineación de la matriz). Cada máscara indica la cámara de su producto (`Make` / `Model` / número de serie), la hora de captura y las etiquetas del objetivo, y enumera las bandas a las que corresponden sus bits (`XMP-Camera:BandName`, primero el bit 0); sus `ImageDescription` y `Software` la identifican como una máscara de recorte, y no contiene calibración.
- **Bits**: bit 0 = la banda del canal R del producto (`Chloros:BandRName`), bit 1 = G (`BandGName`), bit 2 = B (`BandBName`). Un bit activado significa que el valor de ese píxel en esa banda se ha limitado a 0; considéralo, así como cualquier índice construido a partir de él, como de baja fiabilidad.
- **`Chloros:UnmixClampMask`** aparece en todos los productos en los que se ha ejecutado la separación de canales: la ruta de la máscara relativa a la propia carpeta del producto (`../clamp_masks/<product folder>/<product name>_clampmask.tif`, siempre separado por `/` — únelo a la carpeta de la imagen para abrir la máscara), o `none` cuando no se ha limitado ningún píxel. La ausencia de etiqueta significa que el producto nunca se sometió al proceso de «unmix» (cámaras RGB o monocromáticas, niveles sin procesar o sin debayering).
- `lattice capture` / `array-capture` imprimen la ruta de la máscara debajo del archivo guardado (`clamp mask: …`), y la lista de resultados de la captura la incluye como `clamp_mask` del fotograma.
- `lattice index --live --save-multiband` escribe una máscara por cada cámara de origen para la pila multibanda; véase [`lattice index` — máscaras de pila multibanda](#lattice-index--multiband-stack-masks). Un vídeo no tiene máscara: en su lugar, `array-build-video` informa de las fracciones de recorte de cada fotograma (véase [`array-build-video`](#array-build-video--offline-reprocess-a-saved-burst)).

Dado que ninguna carpeta de producto contiene una máscara, a cualquier nivel de profundidad, una carpeta de producto (`radiance/`, `reflectance/`, la carpeta `*_Images/` de una exportación `process`) puede entregarse a Pix4D, Metashape u otro software de fotogrametría tal y como está —subcarpetas incluidas— y solo admite imágenes. Pásale la carpeta del producto, no la carpeta que está por encima (esa también contiene `clamp_masks/` y todos los demás tipos de exportación). La propia función de importación de Chloros omite las máscaras. Lectura de una máscara: `(mask & 1) != 0` significa «el canal R se ha limitado aquí», `& 2` G, `& 4` B.

#### Datos EXIF transferidos a la exportación

`process` copia el **bloque GPS y su ExifIFD** de la captura original a cada producto, por lo que una
exportación incluye `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` y
`CameraSerialNumber` junto con la georreferenciación.

**`FocalLength` no es opcional para la fotogrametría.** Pix4D calcula la distancia de muestreo al suelo a partir de
la distancia focal más la altitud; si falta la etiqueta, recurre a una escala totalmente errónea. En un
vuelo sobre un naranjal de 49 capturas, la falta de la etiqueta convirtió un terreno de 411 m × 160 m en uno reconstruido
de 47,8 km × 13 km — una ortofoto de 455 MP compuesta en su mayor parte por «sin datos», lo que se interpretó como un problema de mosaico y
un problema con BigTIFF antes de que nadie comprobara la GSD. Si tu ortofoto sale a una escala inverosímil, ejecuta primero `exiftool -FocalLength` sobre el producto exportado.

La copia **no** es deliberadamente `-all:all`: las etiquetas estructurales de IFD0&#x27;, ya que sus etiquetas estructurales alteran la salida de LATTICE al
copiarse, y `ExifImageWidth` / `ExifImageHeight` se excluyen porque describen la
captura *original* —una exportación a la que se le haya cambiado el tamaño en algún momento presentaría, de lo contrario, unas dimensiones
contradictorias con respecto a su propia trama. El XMP se escribe directamente en lugar de copiarse, ya que ExifTool
descarta las etiquetas XMP de la misma invocación cuando se copia el bloque XMP (lo que eliminaría las etiquetas de calibración de MAPIR
).

### Dónde se guardan los resultados

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
`Model`) y `<model>_<filter>` para Survey3; las dos cámaras que comparten sensor y filtro, pero difieren
en el objetivo, mantienen árboles de archivos separados, ya que el viñeteado, el campo de visión y la distorsión difieren. La carpeta de formato
de la carpeta sigue este patrón: `--format`: `tiff16`, `tiff8`, `png8`, `jpg8`, o `tiff32` para
`TIFF (32-bit, Percent)`.

> **Cada producto exportado conserva el nombre del archivo ORIGINAL.** Una exportación de radiancia de
> `capture_…_raw.tif` sigue llamándose `capture_…_raw.tif`; simplemente se encuentra en
> `tiff32/Radiance_Images/`. **La carpeta identifica el producto, no el nombre del archivo**, por lo que una búsqueda global
> de «`*radiance*.tif`» no encuentra nada; en su lugar, busca la coincidencia en el directorio.

### Grabaciones del sensor de luz — calibradas `.daq` + `.csv`

`process` también gestiona las grabaciones `.daq` de tu carpeta de entrada, y **no**
necesita ninguna imagen para hacerlo: un DAQ-U / DAQ-M / DAQ-E que haya volado por sí solo constituye una
captura completa, y una carpeta que solo contenga archivos `.daq` es una entrada válida.

Se puede grabar un DAQ **sin** su calibración; eso es lo que hacen por defecto los grabadores públicos
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`) hacen de forma predeterminada: escriben los recuentos sin procesar del sensor y marcan el archivo para que
Chloros recupere la calibración de fábrica de ese sensor **por serie** (primero en la caché local,
luego en la nube de MAPIR) y la aplica. `process` vuelve a escribir el resultado:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

El `.csv` contiene una fila por lectura: marca de tiempo UTC, tiempo de integración, potencia total,
lux fotópico/escotópico, PPFD (y su desglose en azul/verde/rojo), longitud de onda máxima y, a continuación, el
espectro completo en la propia rejilla de longitudes de onda del sensor. El `.daq` se vuelve a importar sin necesidad de
calibrarlo por segunda vez.

`<name>` es el nombre del propio archivo de la grabación. Dos grabaciones en carpetas diferentes pueden compartir
una —las lecturas por captura (`<capture>/daq/daq_data_<sensor id>_<ts>.daq`) guardadas en el
mismo milisegundo por dos cámaras— y cada una sigue obteniendo sus propios productos: la primera conserva
`<name>`, la siguiente añade las carpetas que la distinguen **después** del nombre
(`<name>__<capture>_daq_calibrated.daq`), de modo que la fecha y la hora de la grabación siguen apareciendo al principio.

Si la operación se realiza con éxito, la ejecución muestra `Light-sensor products written: N (calibrated .daq + .csv)`.
El texto entre paréntesis describe lo que se ha escrito realmente, por lo que se lee
`(RAW COUNTS — this sensor has no calibration bundle)` para un sensor sin paquete y
`(N calibrated, M raw counts)` para una carpeta que contiene ambos. Los propios encabezados del backend
`[DAQ-EXPORT]` y `[RUN-SUMMARY]` derivan su redacción de la misma manera: ninguno de
los tres puede calificar una exportación sin procesar como calibrada.

Una grabación DAQ-U / DAQ-M / DAQ-E cuyo paquete de calibración no se pueda recuperar —ya sea porque estás
desconectado o porque ese sensor no tiene ninguna calibración archivada— se **se omite indicando el motivo** en una
línea `[DAQ-EXPORT]`, y nunca se guarda como un archivo «calibrado» que contenga recuentos sin procesar.
Conéctate a Internet y vuelve a ejecutar la tarea. El motivo es el que el lector ha
establecido realmente para ese archivo (esquema ilegible, falta de paquete, error de escritura), y el resumen de la
ejecución enumera motivos **distintos**: veinte archivos omitidos por una misma causa se registran como una
única causa, no como veinte repeticiones de la misma. La CLI nombra cada grabación omitida al final de
la ejecución, junto con su motivo:

```
⚠ Light-sensor recording not exported: <file>.daq: no calibration bundle available for …
```

Una carpeta de grabaciones que no exporta **ninguna** de ellas falla: véase
[Una ejecución que no escribe imágenes falla](#a-run-that-writes-no-images-fails).

Una grabación cuyo nombre termine en `_calibrated` o `_raw` (`field_raw.daq`) **no
importada**: esos son los nombres que `process` asigna a sus propios productos, y un producto leído
como una grabación se contaría dos veces. La CLI lo indica y nombra el archivo —
`Not imported as a light-sensor recording: field_raw.daq … Rename the file to import it.`
— en lugar de informar de una carpeta vacía. Cambia el nombre de dicha grabación antes de procesarla.

#### Las grabaciones DAQ-A se exportan como recuentos sin procesar

La familia **DAQ-A** es anterior al sistema de paquetes por número de serie y no tiene ningún paquete de calibración
que recuperar; en su lugar, se calibra sobre el terreno frente a un objetivo de reflectancia, razón por la cual
nunca lo ha necesitado. Rechazar esas grabaciones les dejaba sin ninguna forma de
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
**dentro** del archivo —que es precisamente para lo que los utiliza la calibración basada en objetivos— y
no entre sensores. Las columnas fotométricas dependientes de la potencia (potencia total, lux fotópicos y
escotópicos, PPFD) se registran como **NULL** en lugar de integrarse a partir de los recuentos, y el resumen
de la ejecución indica `RAW COUNTS`, por lo que «exportado» en un registro no puede interpretarse como irradiancia.

Los registros heredados **v1.01 / v1.02** (un DAQ-A-SD los escribe) no incluyen una época por lectura,
sino solo la hora de escritura del archivo. El comparador de imágenes↔radiación descendente sigue rechazándolas —comparar un
fotograma con una hora de escritura daría un error imperceptible—, pero el exportador las lee, y
el archivo CSV muestra `clock=daq_created_on`, por lo que el producto indica en qué reloj se encuentra.

### Notas

- `process` detecta automáticamente si tu carpeta es de tipo Survey3, «LATTICE» o mixta.
- El progreso se transmite a través de Server-Sent Events; la CLI muestra el progreso en tiempo real por hilo (Detectando, Analizando, Procesando, Exportando).
- En Linux/Jetson, la CLI comprueba el espacio de intercambio y puede mostrar una advertencia antes de procesar carpetas de gran tamaño. El «debayer» sensible a las texturas también aplica automáticamente un límite de frecuencia de la GPU en los Jetson de bajo consumo (Nano, Orin Nano).
- Si se completa con éxito, la ejecución informa de cuántos productos de imagen se han generado (`Image products written: N`; `(plus M clamp masks)` cuando el proceso de desmezclado ha limitado algún píxel).
- **`process` comparte un backend en ejecución de forma segura.** Reinicia automáticamente un backend que ya está en ejecución solo cuando dicho backend está inactivo con un proyecto anterior cargado y no tiene cámaras, matrices ni DAQ. Un backend que contenga cualquiera de estos elementos (la interfaz gráfica de usuario transmitiendo una matriz, un script de SDK, una grabación de DAQ) se utiliza tal cual, sin reiniciarse nunca, por lo que `process` puede ejecutarse junto a una matriz en tiempo real. Si el backend informa de una ejecución en curso, `process` espera hasta 15 s a que finalice (una ejecución que acaba de completarse tarda aproximadamente un segundo en borrarse), tras lo cual devuelve un error con el código `Backend is mid-run (processing). Wait for it to finish, or pass --restart to force a fresh backend (this kills the in-flight run).`. El indicador global `--restart` sigue forzando la creación de un backend nuevo y finaliza cualquier sesión de cámara, matriz o DAQ en él. Un backend ejecutado como servicio «systemd» de Linux nunca se reinicia: `process` lo utiliza, y `--restart` muestra `sudo systemctl restart <unit>` en su lugar (véase [Ejecutar el backend como servicio](#running-the-backend-as-a-service-headless)). Tampoco lo hace el propio backend de la aplicación Chloros: con la aplicación abierta, `process` lo utiliza, cargando su proyecto en lugar del de la aplicación (un backend iniciado en su lugar bloquearía el acceso a la aplicación abierta).
- **Una ejecución en curso conserva su proyecto.** Mientras se está ejecutando una sesión de procesamiento —iniciada por la aplicación, `process`, un script del SDK o `chloros-mcp`—, el backend se niega a crear, abrir, importar, añadir o eliminar archivos, mover, eliminar o modificar la configuración de procesamiento de un proyecto, con un código de error HTTP 409 `PROCESSING_IN_PROGRESS` y `A processing run is in progress on project '<name>', so ... now is refused`; un `process` que lo consiga sale de `1`. La configuración de visualización (tamaño de las miniaturas) sí se guarda. El modo de procesamiento y la configuración de reflectancia objetivo del proyecto se rechazan de la misma manera (una ejecución mantiene el modo en el que se inició). Una captura guardada en el proyecto a mitad de una ejecución (desde la aplicación, la CLI, el SDK o `chloros-mcp`) se guarda como de costumbre, pero se añade a la lista de archivos del proyecto solo cuando finaliza la ejecución; así lo indica su respuesta (`project_add`), al igual que ocurre con un registro del sensor de luz detenido en el proyecto. Antes, un segundo cliente podía cambiar el proyecto abierto durante la ejecución, y la ejecución en curso se exportaba al otro proyecto.
- **Ctrl+C detiene la ejecución en el backend, no solo en la eCLI**. La CLI solicita al backend que detenga el procesamiento, muestra el mensaje `Waiting for the backend to stop (Ctrl+C again to stop waiting)...` y espera hasta 120 s a que aparezca `Processing stopped.`, de modo que `chloros-cli process … && next` no inicie `next` en un backend que aún esté ocupado con la ejecución. Un segundo Ctrl+C pone fin a la espera. A continuación, `process` sale de `130`, como lo hace cualquier comando interrumpido.

#### Una ejecución que no escribe imágenes falla

Si has solicitado productos y la ejecución no ha escrito **ninguno** —solo `project.json` y
`calibration_data.json`—, `process` lo interpreta como un fallo: muestra
`Processing finished but wrote no image products.` y **sale con un valor distinto de cero**, por lo que un script puede
detectarlo. El mensaje indica la carpeta del proyecto y las causas habituales:

- la carpeta de entrada no se ha reconocido como una captura (comprueba la configuración y `--input-level`), o
- todos los productos solicitados se han omitido por no ser aplicables a esas cámaras (por ejemplo, al solicitar
  radiancia/reflectancia de cámaras que solo tienen RGB).

Vuelve a ejecutarlo con `--verbose` y comprueba el registro del backend en busca de las líneas `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`,
que explican las omisiones por cámara que, de otro modo, no aparecerían en la salida de la CLI.

Una ejecución deliberada solo con metadatos —con todos los productos desactivados y sin `--indices`— sigue siendo un
**éxito**, ya que una salida de imagen vacía es el resultado correcto en este caso.

#### Una ejecución cuyas exportaciones no se hayan podido verificar falla

Tras una ejecución, `process` comprueba las etiquetas de los archivos exportados con ExifTool. Si ExifTool no se ha podido ejecutar,
muestra el mensaje «Verificación de la exportación FALLIDA: ExifTool no se ha podido ejecutar, por lo que las etiquetas de N de M archivos exportados
no se han comprobado (véanse las líneas [EXIFTOOL] en el registro del backend)` and `Los productos se guardan y permanecen
en su lugar; vuelve a ejecutar el proceso para verificarlos.`, does not print `¡Procesamiento completado!`, y **sale con un valor distinto de cero**. Los
productos se conservan.

#### Las capturas que realiza una ejecución

Se incluyen las capturas finalizadas antes de que se ejecute `process`. No se incluyen las capturas guardadas durante la ejecución; `process`
avisa de cuántas se han omitido, y al volver a ejecutarlo se incluyen.

#### Una ejecución en la que se ha perdido una imagen falla

Si una imagen falla durante una ejecución, `process` muestra el mensaje `N image(s) failed and were not exported` con la
causa más habitual, no muestra el mensaje `Processing complete!` y **termina con un valor distinto de cero**. Los demás productos se
escriben y permanecen en su lugar, y el registro del backend recoge cada fallo.

Lo mismo ocurre con una **ejecución solo con el sensor de luz**: una carpeta de grabaciones `.daq` no contiene imágenes que exportar
por definición, y la ejecución se evalúa en función de los archivos calibrados `.daq` / `.csv` que ha escrito en su lugar.
Si no ha escrito **ninguna** —se han omitido todas las grabaciones—, muestra
`Processing finished but wrote no light-sensor products.`, nombra cada grabación con el
motivo por el que se ha omitido y **sale con un valor distinto de cero**.

---

## `chloros-cli login`

Autentica esta máquina con una cuenta en la nube de Chloros+. Las credenciales se almacenan de forma segura en `~/.chloros/user_session.json`.

```
chloros-cli login EMAIL PASSWORD
```

### Ejemplos

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (eliminando partes de la contraseña o duplicándolas). Ante un error 401, la CLI vuelve a intentarlo automáticamente añadiendo `$$`, y después con la mitad de la contraseña sin duplicados; si el nuevo intento tiene éxito, te inicia sesión y muestra la sintaxis correcta con comillas simples que debes utilizar la próxima vez.

> **Uso sin interfaz gráfica/mediante script: la ausencia de sesión almacenada en caché implica que aparecerá un mensaje de inicio de sesión.** Un comando principal (`process`, `status`, `export-status`, `time-sync`, …) ejecutado sin una sesión almacenada en caché solicita `Email:` / `Password:` en la entrada estándar antes de continuar. Si la entrada estándar está cerrada (cron, una unidad de systemd), falla inmediatamente con el código de error `login cancelled`; si la entrada estándar es un terminal abierto o una tubería, espera a recibir entrada. Ejecuta `chloros-cli login EMAIL PASSWORD` una vez por máquina antes de programar tareas sin interfaz gráfica.

---

## `chloros-cli logout`

Borra la sesión almacenada en caché y obliga a iniciar sesión de nuevo en la siguiente llamada.

Cerrar sesión mientras se está ejecutando un proceso no lo detiene: la ejecución finaliza en el modo de procesamiento en el que se inició, y la siguiente ejecución utiliza el procesamiento estándar. `logout` así lo indica (`The processing run in progress finishes in <mode> mode; the next run uses standard`), y también lo hace la aplicación al cerrar allí.

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

Consulta el progreso en tiempo real de la exportación de Thread-4. Se puede llamar de forma segura **durante** la ejecución de `process` desde otro shell.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

Configura el idioma de visualización de la CLI (admite 38 idiomas, incluidos CJK, RTL e indios). Recurre de forma fluida al inglés en consolas antiguas que no pueden mostrar el script.

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

## Comandos de la carpeta de proyectos

Estos comandos gestionan la ubicación predeterminada de la carpeta de proyectos (compartida con la interfaz gráfica de usuario).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

`get-project-folder` muestra la carpeta a la que se guardan los nuevos proyectos. Si la carpeta configurada ya no existe (se ha eliminado, se le ha cambiado el nombre o se encuentra en una unidad que no está conectada), la CLI, al igual que la interfaz gráfica de usuario, recurre a la carpeta predeterminada: la carpeta predeterminada se muestra en stdout, se envía a stderr una advertencia indicando el nombre de la carpeta que falta y el código de salida es `0`.

---

## `chloros-cli update`

Solo para Linux/Jetson. Comprueba `version_url` desde `/etc/chloros/update.conf` y, cuando existe una versión más reciente, descarga la versión correspondiente `.deb` y la instala inmediatamente con `sudo dpkg -i` (sin solicitar confirmación; `sudo` puede pedirte la contraseña). En un Jetson, la actualización solo se ofrece a los sistemas con JetPack 6. En Windows, el comando no realiza ninguna acción y muestra el mensaje `up to date`: actualiza con un nuevo instalador.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

En Linux/Jetson, la CLI también realiza una **comprobación automática de actualizaciones en cada inicio** (sin bloqueo, nunca retrasa el comando): lee `/etc/chloros/update.conf`, almacena el resultado en caché durante 1 hora en `~/.chloros/update_cache.json` y muestra `Update available: vX.Y.Z / Run: chloros-cli update` cuando existe una versión más reciente. Se omite silenciosamente ante cualquier error y en Windows; `CHLOROS_DISABLE_UPDATE_CHECK=1` lo desactiva. Debido a la caché de una hora, una versión publicada en el plazo de una hora puede seguir apareciendo como actualizada.

---

## `chloros-cli selftest`

Ejecuta una prueba básica de 7 pasos: versión, disponibilidad de puertos, inicio del backend, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), presencia del modelo de eliminación de ruido, disponibilidad de CUDA + denoiser. Utiliza un backend en ejecución o inicia uno para la prueba y lo detiene después. Muestra `N/7 checks passed` y las comprobaciones fallidas, y sale de `0` solo cuando las siete superan la prueba; la comprobación de CUDA + filtro de ruido falla en un ordenador sin una GPU con CUDA (véase [Verificar la instalación](#verify-the-install)).

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

Estado y control del «grandmaster» PTP. El host Chloros ejecuta el «grandmaster» PTP; las cámaras LATTICE y las unidades DAQ-E se sincronizan con él para obtener marcas de tiempo entre dispositivos (aproximadamente 1 ms entre dispositivos). El PTP sincroniza las **marcas de tiempo**: las cámaras de una matriz exponen simultáneamente porque el disparador de hardware M8 las activa (en menos de 100 µs entre módulos), no por el PTP. `chloros-cli time-sync` sin subcomando ejecuta `status`.

| Subcomando | Descripción |
| --- | --- |
| `status` | Muestra el estado del «grandmaster», las prioridades BMCA, la identidad del reloj y si sus paquetes llegan a la red: `health` es `OK` (`STOPPED` cuando el «grandmaster» no está en funcionamiento), o `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT` junto con un `health_detail`, y, a continuación, se imprime una línea `WARNING:` en stderr (stdout sigue siendo JSON). `sync_sent` / `announce_sent` cuentan solo los mensajes que se han enviado; `*_send_errors` y `last_send_error` cuentan los que no se han enviado. Después de que se interrumpa y se restablezca la conexión de una cámara, el «grandmaster» reconstruye sus sockets por sí mismo (`rebinds`, `last_rebind`). |
| `peers` | Lista de esclavos detectados a través de Delay_Req (cámaras + sensores DAQ-E). |
| `cameras` | Estado de PTP por cámara (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | Reiniciar el proceso del «grandmaster». |
| `set-priority --priority1 N --priority2 N` | Anular las prioridades de BMCA. |

El «grandmaster» se anuncia en las tarjetas de red de las cámaras (cualquier adaptador con una dirección `169.254.x.x` incluida). Si no hay ninguno activo, recurre al adaptador de la ruta predeterminada solo cuando está conectado por cable, nunca a una conexión Wi-Fi ni a un adaptador virtual (Tailscale, WSL), ya que ninguna cámara ni DAQ-E se encuentre detrás de uno. Al no haber nada que anunciar, funciona sin sockets (`health` `NO_SOCKETS`), y la siguiente cámara o DAQ que se conecte lo reconstruye en la NIC que esté activa en ese momento; `time-sync restart` lo hace de inmediato.

El «grandmaster» se ejecuta en un proceso propio, junto al backend (`own_process` / `process_pid` en `time-sync status`), con alta prioridad en Windows (`process_priority`), de modo que un backend ocupado (una ejecución de procesamiento, una exportación, varios comandos a la vez) no pueda retrasar las marcas de tiempo; antes, un retraso de este tipo podía desplazar el reloj de todas las cámaras a la vez. Ese proceso finaliza con el backend; si no se puede iniciar, el «grandmaster» se ejecuta dentro del backend como antes (`own_process` false, y así lo indica el registro del backend).

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

Control de cámaras LATTICE. Cada subcomando pasa por el backend de Chloros; el backend gestiona el grupo de cámaras, por lo que las llamadas posteriores desde la CLI reutilizan el mismo identificador abierto.

### Opciones comunes (compartidas por la mayoría de los subcomandos)

| Indicador | Descripción |
| --- | --- |
| `-d, --device N` | Índice de la cámara (por defecto: 0). |
| `-s, --serial SN` | Número de serie específico; anula `--device`. |
| `--serials SN1,SN2,…` | Números de serie separados por comas para el funcionamiento con varias cámaras. Los [comandos transitorios para múltiples cámaras](#multi-camera-transient-sessions) abren únicamente estas cámaras, y la primera de ellas activa el disparador. |
| `--all` | Actúa sobre todas las cámaras detectadas. |
| `--exposure US` | Tiempo de exposición en microsegundos. |
| `--gain DB` | Ganancia en dB. |
| `--pixel-format FMT` | p. ej., `BayerRG8`, `BayerRG12`; `Mono8`, `Mono12` en un M3M. El nombre establece su profundidad de bits dentro de la propia familia de la cámara: en un M3M, un nombre «Bayer» establece el formato «Mono» de su profundidad (`BayerRG8` establece `Mono8`); en un M3C, un nombre «Mono» establece el «Bayer» (`Mono12` establece `BayerRG12`). |
| `--width N` / `--height N` | Dimensiones de la imagen. |
| `--preset {default,high_quality,high_speed,triggered}` | Aplicar un preajuste de configuración: su formato de píxeles, exposición, ganancia y velocidad de fotogramas (`high_speed`: `BayerRG8`, 5 ms, 6 dB, 60 fps; `high_quality`: `BayerRG12`, 20 ms, 0 dB; `lattice presets` los enumera). En un M3M, el formato es el Mono de la misma profundidad: `high_speed` captura `Mono8`, `high_quality` y `triggered` `Mono12`. Un indicador que se proporciona con él (`--exposure`, `--gain`, `--pixel-format`) prevalece sobre el preajuste, y un ajuste que no figure en el preajuste mantiene el valor propio de la cámara. Todas las ejecuciones libresexcepto `triggered`, que activa la cámara para un flanco de hardware en la línea 2; si nada activa esa línea, esperará indefinidamente en lugar de capturar. |
| `-o, --output DIR` | Directorio de salida (por defecto: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | Tamaño del paquete GVSP. `auto` ejecuta sondas ICMP+GVSP; `jumbo` = 9000; `standard` = 1500. |

### Flujo de trabajo de primera conexión de cámaras LATTICE

```bash
# 0. A backend must be running: the Chloros desktop app (Windows),
#    `chloros-backend`, or the Linux service (see "Start the Backend")

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
#      - Full sensor (no binning, no ROI); the frame rate falls to
#        what the host and wire deliver, and the faster options are
#        printed
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
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | Analiza el sistema anfitrión para obtener la configuración óptima de la cámara. `--no-discover` omite la detección de cámaras (más rápido, análisis solo de la tarjeta de red). |
| `lattice network [--fix] [--estimate] [--cameras N] [--pixel-format FMT]` | Comprueba/corrige la configuración de la tarjeta de red (en Linux, también los búferes de recepción de sockets del kernel —véase [hosts Linux](#linux-hosts--socket-receive-buffers)); estima el ancho de banda y los FPS. |
| `lattice network-analysis --master SN [--slaves SN1,SN2,…] [--width N] [--height N] [--pixel-format FMT] [--binning {1,2,4}] [--force-tier TIER] [--packed {auto,on,off}] [--backend-url URL] [--json]` | Recomendación de red del backend con esquema estable+ recomendación de matriz (devuelve `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). `--width` / `--height` se establecen por defecto en el sensor completo, `2048` × `1536`. Se sale de `0` para `ok`, `auto_capped_fps` y `auto_shrunk`, y de `1` a `needs_force_slip` y `error`, con o sin `--json`. Llega al backend en `--backend-url` (por defecto, `http://127.0.0.1:5000`), no en `CHLOROS_BACKEND_URL`. `auto_capped_fps` mantiene la resolución solicitada pero limita los fps de destino: lee `recommended.recommended_target_fps` y pásalo como destino de la conexión; considéralo un éxito, no como un error. `auto_shrunk` solo aparece cuando el fotograma solicitado no se puede entregar a ninguna frecuencia de fotogramas; la conexión nunca reduce los fotogramas por sí misma. |
| `lattice analyze-array [--models M1,M2,…] [--binning {1,2,4}] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--packed {auto,on,off}] [--json]` | Análisis hipotético sin abrir cámaras (`--width` / `--height` por defecto `2048` × `1536`). Tanto este como `network-analysis` proyectan lo que «connect» transmitirá: sin `--pixel-format`, se aplican los formatos por cámara que utiliza Connect (Mono12 / BayerRG12), y con `--packed auto` (por defecto), la decisión que tomaría Connect en este ordenador respecto al [transporte empaquetado de 12 bits](#transport-12-bits-comprimido---comprimido) que «connect» tomaría en este ordenador — por lo que los fps indicados son la velocidad a la que funciona la matriz. Ambos muestran la decisión de compresión. **`--n-active` es el número total de cámaras conectadas, no solo las de esta matriz**— auméntalo cuando las cámaras independientes transmitan simultáneamente, o cuando el presupuesto de la red se calcule en función de una demanda que las subestime (por defecto: `len(--models)`). Siempre muestra las líneas agregadas `Wire budget:` (MB/s demandados frente al límite máximo a prueba de colisiones) y `Max cameras:`, y marca `** OVER-SUBSCRIBED**` cuando la matriz sobreasigna el canal — véase [Modelo de fps y ráfagas de la matriz](#array-fps--burst-model). |
| `lattice gpu` | Mostrar el estado de la GPU. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | Comprobar o actualizar el firmware de la cámara. La selección local de `.fwa` está fijada: el archivo de `firmware/<MODEL_PREFIX>/` que coincide con el `MIN_FIRMWARE_VERSION` de la compilación se flashea cuando está presente (solo la versión más alta como alternativa), por lo que una imagen del fabricante más reciente almacenada en el disco permanece inactiva hasta que se actualice ese pin; las versiones más recientes llegan a las unidades a través del manifiesto firmado de AWS, que es la opción preferida cuando hay una versión más reciente. |
| `lattice presets [--apply NAME]` | Mostrar o aplicar preajustes de la cámara. `--apply` establece los parámetros de los preajustes (formato de píxeles, exposición, ganancia, frecuencia de fotogramas, disparador) y muestra esos ajustes tal y como los lee la cámara a continuación: en un M3M, el formato Mono de la profundidad del preajuste; la velocidad de fotogramas a la que funciona, que puede ser inferior a la que solicita el preajuste (`high_speed` solicita 60 fps; una cámara de fotograma completo en un enlace de 1 Gbps funciona a unos 35). El resto de ajustes que su comando «open» ha escrito se restablecen tal y como se encontraron. `--apply default` restablece la cámara a sus valores predeterminados de fábrica: su configuración de fábrica con exposición automática en modo libre en la línea 2, el límite de ancho de banda del enlace y la corrección de defectos desactivados, y el balance de blancos calibrado de una cámara RGB conservado (unidad en cualquier otra). No se guarda nada en la memoria de la cámara, por lo que, al encenderse de nuevo, arranca desde su propia memoria como antes. Al ejecutar `--preset` durante una captura o conexión, `default` no es un ajuste preestablecido. |
| `lattice status` | Estado de la cámara en directo. |

#### Captura

| Subcomando | Finalidad |
| --- | --- |
| `lattice capture [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--no-daq] [--daq-tolerance-ms MS]` | Fotograma único. **Guarda todos los tipos de exportación por defecto**(`--processing all`); véase [Niveles de exportación de captura](#capture-export-levels-the-all-default). `--processing` admite `all`, `raw`, `debayered`, `radiance`, `reflectance` o `display` (se acepta `preview` para `display`); cualquier otro nombre se rechaza antes de tocar la cámara (salida `1`). `--levels` guarda un subconjunto explícito (anula `--processing`). Se abre para la captura una cámara que Chloros no tiene, y su exposición se ajusta primero a partir de la escena (unos segundos como máximo), tal y como hacen los [comandos transitorios](#multi-camera-transient-sessions); cuando finaliza la captura, independientemente de cómo haya finalizado, esa cámara se restablece tal y como se encontró (formato, exposición, canalización de color, todos los ajustes que la captura haya modificado —la lista que aparece en [Multicámara (sesiones transitorias)](#multi-camera-transient-sessions)). La lectura del DAQ utilizada para la captura se guarda como un archivo sidecar `.daq`, que incluye una captura solo en formato sin procesar; `--no-daq` / `--daq-tolerance-ms` lo ajustan ([El DAQ que lee una captura guarda](#the-daq-reading-a-capture-saves)). `--force-daq` sigue siendo válido. `--jpeg-quality` = calidad JPEG 1–100 (por defecto 95).**Los archivos guardados en el proyecto abierto se incorporan a él.** Al igual que en la aplicación, una captura cuyos archivos se guarden en la carpeta del proyecto abierto en Chloros los añade a la lista de archivos de ese proyecto; los archivos guardados en otra ubicación no se modifican. El comando muestra una línea indicando esto (`Project: ...`) o, si hay un proceso en curso, que se retienen y se incorporan cuando finaliza dicho proceso. Lo mismo ocurre con `multi-capture`, `array-capture` (también `--smart`) y `project capture`. |
| `lattice continuous [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--queue-depth N] [--no-daq] [--daq-tolerance-ms MS]` | Transmite al disco hasta pulsar Ctrl+C. Una ejecución de `tiff`son el archivo `raw`, una matriz que captura los datos de esa cámara: el mosaico del sensor, un plano con sus propios valores, sin comprimir, con el paquete de calibración (un fotograma de 2048 x 1536 y 12 bits ocupa 6,0 MiB más el paquete, sea cual sea la escena), por lo que `process` parte de ellos. `png` y `jpg` contienen 8 bits, por lo que esas series son fotogramas desmosaicados para su visualización, de los que `process` no parte. Con un DAQ vinculado (`daq pool-bind`) o agrupado, cada fotograma obtiene sus propios `.daq` y `.match.json` en la carpeta `daq/` de la serie, ya que `lattice capture` guarda uno; `--no-daq` desactiva esa función, y al detener el proceso se muestra el número de fotogramas. Cada fotograma se guarda una vez, con todos sus metadatos; al pulsar Ctrl+C, los fotogramas ya capturados se escriben antes de que se suelte la cámara. Se detiene automáticamente cuando se llena la unidad, imprime `Failed:` / `Disk full:` y sale con `1`; cualquier fotograma que no haya podido guardar también sale con `1`. Una cámara que se haya dejado preparada para un disparador de hardware (`TriggerMode` `On`) se configura en modo de funcionamiento libre para la sesión. Cuando finaliza la sesión, la cámara se restablece tal y como se encontró: se recuperan los ajustes de disparo y todos los parámetros que la sesión haya modificado (exposición, ganancia, formato, …)—, salvo en el caso de `--keep-settings`, que guarda los ajustes de la sesión en la propia memoria de la cámara (modo libre), o `--clear-settings`, que restablece dicha memoria a los ajustes de fábrica. Una sesión que no graba ningún fotograma se considera un fallo: si no hay ningún fotograma durante 10 s, se detiene automáticamente, imprime `No frames:` con el motivo y sale con `1`. Los fotogramas que los escritores no han podido procesar a tiempo se cuentan como perdidos, y al detenerse se muestra el mensaje `Dropped:` con el motivo (la frecuencia de fotogramas de la cámara, la de los escritores y la de la cola); una frecuencia de fotogramas más baja o un formato de píxeles de 8 bits permite conservar todos ellos. El comando se comunica con el backend cada 2 s; si se bloquea sin detener la grabación (`kill -9`, falta de memoria), el backend la detiene 30 s después de la última comunicación y vuelve a poner la cámara (`stop_reason` `client_gone`, y una línea en el registro del backend). `multi-stream` y `align-stream` hacen lo mismo. Un inicio en el que la cámara tarda más en abrirse de lo que espera el comando (60 s; 180 s para `multi-stream` / `align-stream`) —una cámara que aún se está iniciando— es anulado por el backend: la cámara se vuelve a colocar en su sitio y se cierra, y no se deja ninguna grabación en marcha (504 HTTP `CLIENT_DEADLINE_PASSED`). |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Vista previa MJPEG en directo basada en navegador: conecta la cámara, muestra la URL de la transmisión y la abre en tu navegador. `--ae-damping` configura la amortiguación de la exposición automática (0,4–100). |

##### Transmisiones en directo (MJPEG; sin RTSP)

Chloros ofrece vídeo en directo como **MJPEG sobre HTTP**(`multipart/x-mixed-replace`, límite `frame`) desde el backend.**No es compatible con RTSP.** Abre una URL en un navegador o léela con cualquier cliente MJPEG (`cv2.VideoCapture(url)` de OpenCV, ffmpeg, VLC). La cámara debe conectarse primero en el backend (`lattice cam-connect`, `lattice array-connect`, `lattice viewer` o la interfaz gráfica de usuario).

| URL (`http://127.0.0.1:5000` + …) | Flujo |
| --- | --- |
| `/api/camera/stream/<serial>` | La vista previa de la pantalla, hasta unos 30 fps. Para un elemento de la matriz, `?max_h=` (por defecto `720`) y `?jpeg_q=` (por defecto `80`) establecen la altura y la calidad JPEG. Esta es la URL que abre `lattice viewer`. |
| `/api/camera/<serial>/stream-annotated` | La misma vista previa con las superposiciones de la interfaz gráfica de usuario dibujadas en: `zebra=1`, `crosshair=1`, `grid=3x3` (o `9x9`), `histogram=1`, `peaking=1`, `spot=1`; sin necesidad de consulta, la configuración de superposición guardada en el proyecto para esa cámara. |
| `/api/camera/<serial>/stream-reflectance` | Reflectancia de 8 bits por banda: ρ = píxel ÷ la parte`X-Chloros-Pixel-Scale` (255 a menos que se active `refl_max`). `fps` (por defecto `10`), `jpeg_q` (por defecto `95`), `max_h` (por defecto `0` = nativo), `band=<name>`, `align=1`. Formato JPEG, por lo que presenta pérdida de información. Se rechaza con un código de estado HTTP 409 cuando la cámara no tiene reflectancia (una cámara RGB, sin paquete de calibración, sin lectura DAQ). |

- **Coincidencia de fotogramas entre cámaras.**Cada parte que envía un**miembro de la matriz**, en las tres secuencias, lleva los encabezados `X-Chloros-Group-Id` (el contador de grupo sincronizado de la matriz), `X-Chloros-Frame-Id` y `X-Chloros-Timestamp-Ns`. Las partes procedentes de diferentes cámaras con el mismo `X-Chloros-Group-Id` provienen del mismo disparador. Una cámara independiente no tiene grupo y no envía ningún identificador de grupo. Los navegadores ignoran los encabezados de cada parte; es el propio cliente que analiza la transmisión multiparte quien los lee.
- **En pausa mientras la vista de matriz está abierta.** El `/api/camera/stream/<serial>` de un miembro de la matriz envía un fotograma estático de «pausa» mientras el flujo en directo combinado de la matriz está abierto (por ejemplo, la vista de matriz de la interfaz gráfica de usuario), y se reanuda cuando se cierra.

#### Ajuste de sensores

| Subcomando | Finalidad |
| --- | --- |
| `lattice configure [{--get N1 N2… \| --set N=V N=V… \| --dump}] [--json]` | Leer/escribir en cualquier nodo GenICam. `--set ExposureTime=…` (`Gain=…`) sin sus conmutadores `ExposureAuto` (`GainAuto`) que desactivan primero el modo automático, tal y como hace `lattice exposure --set`; si la cámara rechaza un valor, se muestra `Failed to set <node>: …` en stderr y se sale de `1`. `--get` y la vista agrupada por defecto muestran un `note:` bajo `ExposureTime` / `Gain` mientras se ejecuta ese modo automático — véase [Lectura de la exposición y la ganancia en modo automático](#reading-exposure-and-gain-under-auto-mode). |
| `lattice exposure [{--auto \| --auto-once \| --off \| --set US}] [--brightness 0-255] [--brightness-auto] [--damping 0.4-100] [--upper-limit US]` | Exposición y AE. Se mantiene una exposición `--set` o un objetivo `--brightness` como el tuyo (el control automático de exposición de Chloros no lo modifica); la ganancia se mantiene tal y como estaba, por lo que, en modo automático, sigue ajustándose hacia el objetivo hasta que establezcas `lattice gain`. `--auto` devuelve el control de la exposición, `--brightness-auto` devuelve el objetivo al sistema automático de Chloros. Sin indicadores, lee los nodos de exposición, con el mismo `note:` que `configure --get`. Si la cámara rechaza un valor `--set`, imprime `Failed to set ExposureTime: ExposureTime=<value> refused (<camera error>); the camera accepts <min>..<max> now (ExposureAuto put back to <mode>)` en stderr — el modo automático, cuya escritura se había desactivado, se restablece en lugar de permanecer desactivado — no muestra ninguna línea `Exposure: … us (auto OFF)` y sale de `1`. |
| `lattice gain [{--auto \| --off \| --set DB}]` | Ganancia y ganancia automática. Un valor `--set` que la cámara rechaza imprime `Failed to set Gain: …` en stderr, no imprime la línea `Gain: … dB (auto OFF)` y sale de `1`. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N]` | Área de interés (ROI) del sensor y agrupación de píxeles. La agrupación de píxeles es siempre **promedio** — véase más abajo. |
| `lattice format [--set FMT] [--list]` | Formato de píxeles. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Disparador de hardware/software. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (sin indicadores = balance de blancos de un solo disparo) | Operaciones de balance de blancos. Solo para cámaras RGB/Bayer; una operación nula (omitida) en M3M monocromáticas. El modo de un solo disparo conserva el balance de blancos que calcula (`BalanceWhiteAuto` y las ganancias); el resto de ajustes que se escriben en memoria abierta se devuelven tal y como se encontraron, y todo ello cuando el modo de un solo disparo no puede ejecutarse. |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct 2000-10000] [--get]` | Canal de color de visualización RGB. `natural` (por defecto) es el acabado en directo más sencillo; `enhanced` añade eliminación de franjas + vibrante + contraste local CLAHE para obtener el aspecto completo de paridad de hub a un coste de procesamiento por fotograma de aproximadamente el doble, lo que se traduce en una menor velocidad de fotogramas **en directo**; las capturas guardadas siempre obtienen el procesamiento completo en cualquier caso. Solo para cámaras RGB/Bayer; se omite en M3M monocromo. |
| `lattice color [--saturation 0-200] [--contrast 0-200] [--reset] [--get]` | Saturación/contraste de la vista previa en directo (cámaras con filtro RGB); `100` es neutro, `--reset` establece ambos en 100. Las capturas guardadas no se ven afectadas. Se omite en la M3M monocromática. |
| `lattice highlights [{--set -100..0 \| --reset \| --get}]` (sin indicadores = lectura) | Mostrar **altas luces** (cámaras con filtro RGB): atenúa los píxeles más brillantes y blancos —por ejemplo, una superficie blanca brillante que se representa como una mancha blanca plana—. `0` = desactivado, `-100` = máximo. La vista previa en directo y las capturas a nivel de pantalla lo siguen; la radiancia/reflectancia nunca lo hacen, y no puede recuperar datos del sensor que se hayan recortado (para ello, reduce la exposición). Se omite en el M3M monocromo. |
| `lattice filter [--list]` | Muestra la lista de modelos de filtro que conoce Chloros (`RGN-IMX265`, `OCN`, `NGB`, …). El filtro de una cámara se lee a partir de la cadena de su modelo y no se puede configurar: `--set NAME` solo busca un modelo y, a continuación, sale de `1` con `setting the filter model is not supported`. |
| `lattice power [--sleep]` | Analiza los nodos de alimentación y térmicos; activa o desactiva el modo de inactividad de bajo consumo. `--sleep` deja la cámara en modo de suspensión (con el límite de ancho de banda del enlace activado (`DeviceLinkThroughputLimitMode` `On`) en su valor mínimo, y el modo de ahorro de energía activado si la cámara lo tiene) y el resto de ajustes se restablecen tal y como se encontraron inicialmente; un modo de reposo que falla se restablece por completo. El despertar devuelve el enlace tal y como lo encontró el modo de reposo (el límite, su modo y el retraso de paquetes `GevSCPD`, que la cámara mantiene en 0 mientras el límite está activado; el registro se conserva en este ordenador, de modo que una reactivación tras el reinicio del servidor también lo encuentre así), o, en el caso de una cámara que este ordenador no haya puesto en modo de suspensión pero que parezca estarlo (con el límite activado en el mínimo), con el límite desactivado y un `GevSCPD` dejado en 0 a 80 (la configuración de fábrica); un límite establecido por encima del mínimo se deja tal cual. La sonda y la activación restablecen lo que su apertura escribió. |

> **El binning es siempre «Average».** `lattice resolution --binning N` establece el binning «Average» y
> no ofrece cambio de modo. El binning por suma multiplica cada DN por N², lo que supone un coste aproximado de dos pasos de
> margen de luces altas por paso de binning; Chloros divide ese factor antes de la
> cadena radiométrica, pero los píxeles que la suma ha llevado al techo del sensor se recortan y no pueden
> recuperarse. No aporta nada que el modo «Promedio» no aporte: un índice es invariante a la escala y los
> escalares de radiancia se calibran a la escala del modo «Promedio». Si una cámara viene ya configurada en «Suma» (dejada
> así por otra herramienta o por un restablecimiento de fábrica), Chloros lo detecta a partir de una lectura del nodo y
> lo compensa; al ejecutar `lattice resolution --binning N` se vuelve a establecer en «Average». La versión 1.2.0
> `--binning-mode` sigue siendo válida, por lo que los scripts antiguos se ejecutan, pero solo muestra una advertencia:
> el binning permanece en «Promedio».

> **Cambiar el binning, el formato de píxeles y el disparador en una cámara no tiene ningún efecto.** En una cámara en la que el
> backend permanece abierto y transmite datos (`cam-connect`, la interfaz gráfica), `lattice resolution --binning`
> no surte efecto (muestra que el binning sigue vigente, `Binning: 1x1`), y
> `lattice format --set` y `lattice trigger --mode` son rechazados por la cámara
> (`SC_ERR_ERROR -1001`). Libéralos primero (`lattice cam-disconnect -s <SN>`) o configúralos
> al conectarte (`cam-connect --pixel-format …`, `array-connect --binning …`).

##### Lectura de la exposición y la ganancia en modo automático

Mientras se está ejecutando el `ExposureAuto` (`GainAuto`) de una cámara, su nodo `ExposureTime` (`Gain`) conserva el **último valor escrito**, no lo que la cámara está haciendo en ese momento. En el caso de una cámara cuyo backend permanece abierto (`cam-connect`, `array-connect`, la interfaz gráfica de usuario), `lattice configure --get`, la vista predeterminada `lattice configure` y `lattice exposure` (sin indicadores) muestran, por lo tanto, el valor del nodo y, debajo, una nota con el fotograma más reciente de la cámara:

```

ExposureTime = 2000.448
  note: ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448); the camera's newest frame was at 4900 us.
```

Lee el valor del fotograma de la nota; el valor del nodo es solo la última escritura. A través de HTTP, la misma respuesta `POST /api/camera/<serial>/read-nodes` mantiene `nodes` sin cambios y añade `frame_values` (`{"ExposureTime": <us>, "Gain": <dB>}`, el del fotograma más reciente) y `notes` (la frase anterior). Ambos están presentes solo mientras el modo automático no sea `Off` y se conozca un fotograma; una cámara abierta solo para la lectura no tiene fotograma y no obtiene ninguno de los dos. `lattice gain` (sin indicadores) solo imprime el nodo; utiliza `lattice configure --get Gain` para la nota.

##### Exposición en un miembro de la matriz

Mientras una matriz transmite, cada miembro funciona con **`AcquisitionFrameRateEnable` activado**, a una frecuencia de fotogramas ligeramente superior a la frecuencia de disparo de la matriz: su período es el período de disparo menos un 15 % (como mínimo 3 ms), o la frecuencia máxima propia de la cámara cuando el disparo ya se encuentra cerca de ella. Por lo tanto,**`configure --get` en un miembro**lee**`AcquisitionFrameRateEnable`**como**`True`**y**`AcquisitionFrameRate`**con ese ajuste; la activación se desactiva cuando la matriz se detiene.**No escribas estos dos nodos en un miembro de la matriz**: la matriz los mantiene sincronizados con su disparador y los reescribe.

Esa configuración es también el límite máximo de exposición del miembro: unos 85 ms con un disparador de 10 fps (período de 100 ms), en **cualquier ROI o binning**. Antes de la versión 1.3.0, una ROI pequeña limitaba la exposición de un miembro al período de funcionamiento libre del sensor (4,9 ms a 768 × 304), independientemente de lo lento que se disparara la matriz.**Un `ExposureTime` manual más largo que el límite máximo ralentiza el disparador para ajustarse a él** cuando se escribe con `lattice exposure --set` o `configure --set` (o desde la interfaz gráfica de usuario). La matriz abre primero el límite máximo de ese miembro para el valor y deja que la velocidad más lenta funcione durante unos cuantos períodos de disparo, por lo que la escritura puede tardar unos segundos más; a continuación, el disparo se activa a una velocidad no superior a la que permite la exposición. En una matriz de velocidad automática, la velocidad a la que funcionaba anteriormente se restablece una vez que todos los miembros cuya exposición manual la ralentizó vuelven a la exposición automática. Un valor que la propia cámara no puede aceptar sigue siendo rechazado, indicándose en el mensaje de error el rango que admite, y se restablece la exposición automática del miembro si se había desactivado únicamente para esa escritura. Una exposición de la receta `apply` no ralentiza el disparador: se mantiene en el límite máximo. Chloros nunca acorta una exposición manual para adaptarse a un disparador más rápido; una frecuencia fijada (`array-connect --target-fps F`) más rápida de lo que permite una exposición deja a ese miembro un límite máximo lo suficientemente largo para ello, y ese miembro no dispara a una velocidad superior a la que permite su exposición.

#### Calibración y sensores

| Subcomando | Finalidad |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Calibrar a partir de un objetivo de reflectancia. Se buscan hasta `--attempts` fotogramas (por defecto, `30`); un fotograma incompleto (pérdida de paquetes GVSP) cuenta como un intento, no como una ejecución. Fallará con `no target detected in N attempts`, o en `no complete frame in N attempts: <last error>` cuando no ha llegado ninguna trama completa. |
| `lattice dls {--connect \| --spectrum \| --irradiance} --mac MAC [--filter NAME] [--camera SERIAL] [--json]` | El espectro y las magnitudes de banda de un DAQ-M, leídos a través de Bluetooth (BLE) mediante el backend. `--mac` es la dirección MAC BLE propia del sensor y es necesaria para cualquiera de las tres acciones: no hay un valor por defecto, ya que cada unidad tiene la suya propia (`daq pool-discover --only ble` las enumera); sin él, el comando sale de `1`. Si no se especifica ninguna acción, muestra su ayuda. `--camera SERIAL` integra las magnitudes de banda mostradas mediante la respuesta espectral medida por la propia cámara, en lugar de la curva de flotilla del modelo de filtro — cámaras RGB, con el paquete de calibración de la cámara cargado (conectada) o almacenado en la caché de este ordenador. El encabezado indica qué curva se ha utilizado: `Band magnitudes (camera <SERIAL>):`, o `Band magnitudes (model <filter>):` cuando la cámara no tiene ninguna (no es una cámara RGB, o no tiene ningún paquete cargado ni almacenado en caché) y, en su lugar, se ha utilizado la curva predeterminada. A través de HTTP, `POST /api/dls/cli-read` toma `camera_serial` y devuelve `band_magnitudes_source`. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Aplica la corrección de viñeteado a las imágenes existentes, cada una con el campo plano de su propio objetivo (según su modelo EXIF). `--lens-model` se aplica a imágenes cuyo modelo no especifique ninguna cámara; se omite cualquier imagen cuyo modelo entre en contradicción con ello. |

#### Multicámara (sesiones transitorias)

Un comando transitorio abre las cámaras para una ejecución y las cierra cuando esta finaliza: `multi-capture` sin ninguna matriz persistente conectada, `multi-stream` y `multi-test` en este caso, y `align-calibrate`, `align-apply`, `align-stream` y `index --live` a continuación. Estas reglas se aplican a todas ellas:

- **`--serials SN1,SN2,…` selecciona las cámaras.** Solo se abren esas, y la primera activa el disparador (en el caso de `align-calibrate` y `--reference`, cuando se especifiquen). Una cámara en serie que no esté en la red provoca el fallo de la ejecución antes de que se abra ninguna cámara. Sin `--serials`, se abren todas las cámaras de la red. Una cámara que Chloros ya tiene abierta (la interfaz gráfica de usuario, `cam-connect`, `array-connect`) rechaza la ejecución, independientemente de si `--serials` la nombra o no: comparte la red para la que está configurada la ejecución. (`index --live` sin `--serials` abre en su lugar las cámaras habilitadas de su perfil: véase [Índice / Matemáticas de la vegetación](#index--vegetation-maths).)
- **Las cámaras comparten la red del mismo modo que lo hace `array-connect`.** Cada cámara se adapta a su parte de lo que transporta el enlace principal (un retardo entre paquetes, no el límite de rendimiento de la cámara, cuyo mínimo de 31,25 MB/s ya superan cuatro cámaras en 1 GbE), por lo que cuatro cámaras funcionan en un enlace de 1 Gbps. Cuando las cámaras no pueden enviar todas a la vez al ritmo más lento que alcanzan, se activan una tras otra, cada una con el enlace para sí misma, y el comando muestra `Serial emit: the wire cannot carry these cameras at once, so each is triggered <ms> ms after the one before; their frames are not exposed together`. Cuando ni siquiera cabe una sola cámara a la vez, la ejecución se rechaza antes de que ninguna cámara transmita: `array over-subscribes the wire: <n> cameras demand <x> MB/s but the collision-safe ceiling is <y> MB/s. …`, HTTP 409, salida `1`.
- **Todas las cámaras deben transmitir, o la ejecución falla.** Una cámara que no transmite ningún fotograma es un error: el comando muestra `Error: … no frame from <n> of <total> cameras: SN:<serial> (<why>), …`, donde `<why>` es el error de recuperación de esa cámara, seguido de la causa indicada, y sale con `1`. Una trama que llega con pérdida de paquetes GVSP (`IncompleteBufferError: GVSP packet loss …`) significa que la red no soporta el ancho de banda combinado de las cámaras (`Every missing frame arrived with GVSP packet loss: …`): comprueba la velocidad de enlace del host y del enlace ascendente del conmutador (`lattice network`), utiliza tramas jumbo, o abre menos cámaras con `--serials`. Una cámara que no ha enviado nada normalmente no está recibiendo la señal de activación del maestro: comprueba su cable de sincronización M8 y, a continuación, ejecuta `lattice multi-detect`. Lo que hace cada comando cuando falta una cámara en un grupo de tramas se indica en su fila.
- **Cada cámara se restablece tal y como se encontró.** Cuando finaliza la ejecución, cada cámara recupera el estado en el que se encontraba al iniciarse la ejecución: `TriggerMode`, `TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, y `LineMode` / `LineSource` en `Line2` y `Line3` (y la entrada de disparador elimina el ruido de una matriz que proporciona la línea de un esclavo, `LineFilterSelector` / `LineFilterWidth`); su `PtpEnable` (y el conjunto `PtpSlaveOnly` / `PtpDomainNumber` asociado a él); `ExposureAuto` / `GainAuto` (y `ExposureTime` / `Gain`, donde se había establecido un valor manual); `TargetBrightness`, `ExposureAutoLimitAuto` / `ExposureAutoUpperLimit` / `ExposureAutoLowerLimit`, `GainAutoUpperLimit`, `ExposureAutoDamping`; `BalanceWhiteAuto` (y cada `BalanceRatio` en los que el balance de blancos era manual); `BlackLevelAuto` / `BlackLevel`; `PixelFormat`, agrupación de píxeles, `Width` / `Height` / `OffsetX` / `OffsetY`; `GammaEnable` (y `Gamma` en los que el gamma estaba activado), `LUTEnable`, `ColorTransformationEnable`; `ReverseX` / `ReverseY`; `UserSetSelector`; el tamaño del paquete (`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`); la cadencia, `DeviceLinkThroughputLimitMode` (y `DeviceLinkThroughputLimit` cuando el modo era `On`), elpaquete `GevSCPD` y el desfase de inicio `GevSCFTD` que proporciona una matriz para cada cámara; los datos de fragmento que cada flujo activa (`ChunkModeActive`, cada `ChunkEnable` bajo su `ChunkSelector`); `AcquisitionFrameRateEnable` (y `AcquisitionFrameRate` donde la activación estaba activada; cuando está desactivada, la velocidad es de solo lectura y se lee el máximo que permite el ritmo, por lo que no es un ajuste). Solo se escribe lo que difiere: la cámara se detuvo primero, la cadencia y la frecuencia de fotogramas antes de una exposición que requiere su periodo de fotograma (una cámara limita `ExposureTime` a un periodo de fotograma). Con `--keep-settings`, el cableado del disparador y de la línea de sincronización se desconecta, la cámara funciona de forma autónoma y el resto se guarda en la propia memoria de la cámara tal y como lo ha configurado la sesión; `--clear-settings` restablece allí los ajustes de fábrica.
- **La exposición se ajusta a partir de la escena anterior a la captura del primer fotograma.** La serie toma grupos de fotogramas sincronizados y los descarta mientras ajusta la exposición y la ganancia de cada cámara a partir de esos fotogramas. El objetivo es alcanzar un brillo medio en el objetivo de la cámara sin que se recorte más del 1 % del fotograma; una escena con zonas muy brillantes no alcanza el objetivo. Esto suele requerir entre 2 y 6 grupos de fotogramas, y un máximo de 5 s.
  - Cada cámara enfoca el objetivo desde el que la inicia una matriz conectada, según su filtro: TargetBrightness **40**en una cámara multiespectral (RGN, OCN, NGB) y**60** en una RGB. Esto supone la mitad del objetivo de exposición que proporciona el valor 80 (tres cuartas partes en RGB), por lo que es menos probable que se produzca recorte en un panel de reflectancia brillante. `--brightness N` (0-255) establece en su lugar el objetivo de cada cámara (`multi-test` no tiene `--brightness`: su exposición automática está desactivada). Los nombres de los registros del backend indican el objetivo de cada cámara: `[transient-ae] AE target per camera (filter default): SN:<serial> <target>, …`.
  - La exposición es lo primero, hasta el límite de exposición automática de la propia cámara. La ganancia solo cubre lo que queda.
  - La exposición y la ganancia calculadas se mantienen durante toda la sesión, por lo que cada fotograma de una `multi-stream` se captura con la misma exposición. Una captura que deba adaptarse a cambios de luz debe realizarse en una matriz conectada (`array-connect`), cuya exposición automática se ajusta continuamente.
  - `--exposure` y `--gain` se mantienen exactamente tal y como se han especificado. Si se especifican ambos, no se descarta ningún fotograma.
  - Cuando finaliza la sesión, se vuelve a activar la exposición automática de cada cámara.
  - El registro del backend informa del resultado: `[array-ae] exposure solved in <s> s (<n> warm-up groups): SN:<serial> <ms> ms <dB> dB mean <m>/<target>, …`, o `exposure NOT solved after …` con el motivo, y `(not settled)` en cada cámara que no lo haya hecho. En ese caso, los fotogramas se siguen capturando.
  - `CHLOROS_TRANSIENT_AE_SETTLE_S` modifica el límite de 5 s; `0` lo desactiva (véase [Variables de entorno](#environment-variables)).

**Mientras Chloros tenga seleccionada cualquier cámara** (la interfaz gráfica de usuario, `cam-connect` o una matriz conectada), `multi-stream`, `multi-test`, `multi-detect`, `align-apply`, `align-stream` y `index --live` se niegan a ejecutarse, ya que abren todas las cámaras de la red y una segunda apertura de una cámara retenida puede provocar un fallo del backend. (`index --live` se rechaza cuando una cámara que iba a abrir está retenida: ya sea `--serials` o las cámaras de su perfil, y cualquier cámara de un perfil que no especifique ninguna.) El backend responde con un código HTTP 409 a `CAMERA_HELD`, indicando cada cámara bloqueada y los comandos de liberación (`chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all`), y el comando se cierra con `1`. Si hay una matriz conectada, `array-capture --continuous` la graba en lugar de `multi-stream`, y `array-capture` (alineado por defecto) guarda un conjunto alineado en lugar de `align-apply`; `multi-capture` y `align-calibrate` utilizan una matriz conectada que contiene sus cámaras en lugar de abrirlas.

| Subcomando | Finalidad |
| --- | --- |
| `lattice multi-info [--serials SN1,SN2,…]` | Muestra todas las cámaras con funciones de sincronización; con `--serials`, solo aquellas; la primera como `MASTER`. |
| `lattice multi-capture [--serials SN1,SN2,…] [--format FMT] [--jpeg-quality N] [--processing LEVEL] [--brightness N] [--no-daq] [--daq-tolerance-ms MS]` | Un fotograma sincronizado de cada cámara. Guarda **todos los tipos de exportación de forma predeterminada**cuando hay una matriz persistente conectada, y la lectura del DAQ que se ha utilizado ([Lo que guarda la lectura del DAQ al realizar una captura](#the-daq-reading-a-capture-saves)); la solución alternativa transitoria sin matriz**solo se somete a debayering** (ejecuta primero `array-connect` para el resto). `--brightness` se aplica únicamente al modo de reserva: una matriz conectada mantiene su propia exposición automática. El modo de reserva no guarda `.daq` (`--no-daq` / `--daq-tolerance-ms` no tienen ningún efecto allí), y `multi-capture` no acepta `--force-daq`. En el modo de reserva, una cámara que no envía ningún fotograma falla la captura (`Error: multi-capture failed: no frame from …`, salir de `1`) y no se guarda ningún archivo de ese grupo de fotogramas. |
| `lattice multi-stream [--serials SN1,SN2,…] [--fps F] [--count N] [--format FMT] [--jpeg-quality N] [--brightness N]` | Fotogramas sincronizados de la transmisión (transitorios), `--fps` por defecto `3`, hasta que se pulse Ctrl+C o `--count` grupos. No se escribe ningún grupo de fotogramas para el que una cámara no haya enviado nada, y tres seguidos dan por finalizado el flujo. Si se ha omitido algún grupo, la ejecución finaliza imprimiendo `Incomplete: <n> frame group(s) not written, a camera delivered nothing (SN:<serial> x<count>, …)`, además de `Error: the stream stopped: …` indicando el nombre de la cámara cuando finalizó el flujo, y sale de `1`X. Se detiene automáticamente cuando se llena la unidad, muestra `Failed:` / `Disk full:` y sale con `1`; cualquier archivo que no haya podido escribir también sale de `1`. |
| `lattice multi-test [--serials SN1,SN2,…] [--count N] [--ptp-settle SECONDS]` | Prueba de sincronización de GPIO (transitoria), `--count` disparadores predeterminados `10`. Muestra cómo se comparte la red (`Wire: each camera paced to <x> MB/s (<n> on a <y> MB/s ceiling, <link> Mbps link)`, y la línea `Serial emit:` cuando las cámaras se activan una tras otra, cuyo desfase se refleja entonces en las marcas de tiempo), una línea `Sync group: master SN:<serial> -> SN:<serial>, …` por grupo de sincronización (`(no slaves: not synced)` para una cámara sin cable de sincronización, que se activa por sí sola), luego una fila por disparo: `[  0] SN:<serial>=<timestamp> \| …`, con `SN:<serial>=(no frame)` para una cámara que no ha enviado nada para ese disparo. A continuación, una línea `Offsets from master SN:<serial> (<PTP / device (PTP off)> clock, <n> trigger(s)):` y una línea `SN:<serial>: mean <±x> us, jitter <y> us` por cámara, cada cámara respecto al maestro en un ciclo de reloj: se omite un disparador cuyas marcas mezclan los relojes de los dispositivos de las cámaras y el PTP (PTP desactivado o aún en proceso de sincronización) se omite, y una línea `Note:` las cuenta. Cualquier fotograma perdido hace que la prueba falle (`Error: multi-test failed: <k> of <n> frame groups incomplete (SN:<serial> missed <k>); …`, salida `1`); tres disparadores incompletos seguidos la dan por terminada antes de tiempo. También falla (salida `1`) cuando menos de dos disparadores tienen todas las cámaras sincronizadas en un mismo reloj (`… the clocks cannot be compared`), o cuando los relojes nunca coinciden: los relojes PTP se desvían más de 1 ms (más cualquier desfase del nivel serie) respecto al maestro (`… PTP is not locked`), o los relojes de los dispositivos cuyo desfase varía más que eso entre disparos. Inicia el gran maestro PTP si no está en funcionamiento, activa el PTP de todas las cámaras (tal y como hace `array-connect`, independientemente de la configuración de la red) y, antes del primer disparador, espera a que el reloj PTP de todas las cámaras se sincronice con el gran maestro y se estabilice: `PtpStatus` Esclavo en un gran maestro, con una desviación de menos de 100 µs respecto a este y estable durante 3 s, y cada cámara sincronizada de este modo, sin interrupción, durante 120 s —aproximadamente 2 minutos desde el encendido. Un reloj que acaba de sincronizarse presenta un desfase de hasta ~200 µs durante unos 2 minutos después de que se le identifique como «Slave», y el propio `PtpOffsetFromMaster` de la cámara no lo muestra; tras 120 s, las cámaras se sitúan a ~10 µs de un reloj estabilizado. Una cámara sigue sincronizada desde una prueba anterior en la que se dejó el PTP activado, tal y como lo detectó este backend, cuenta a partir de ese sincronizado, por lo que una prueba realizada justo después de otra no vuelve a esperar; una cámara que se enciende durante la prueba, o una que se encuentra con PTP activado y que este backend no ha visto sincronizada (encendida en otro lugar o antes de un reinicio del backend), espera sus 120 s, y en una matriz en la que ambas cámaras acaban de sincronizarse, estas siguen esperando su turno. Una cámara que sale del modo Esclavo, supera los 100 µs o sigue a otro gran maestro durante 3 lecturas consecutivas (1,5 s) durante la espera, reinicia su cuenta; una o dos de estas lecturas se registran (`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`) y el recuento continúa; además, un ID de «grandmaster» que no se pueda leer se considera desconocido, no un cambio. Que el recuento de todas las cámaras se reinicie a la vez (en menos de un segundo) se debe a que es el tiempo del «grandmaster» el que avanza, no el de las cámaras: el registro del backend indica `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …`, `ptp_lock` lo incluye en `grandmaster_steps`, y una espera que luego se agota inicia su error con `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`. El límite no se amplía para ello. `--ptp-settle SECONDS` establece el tiempo de estabilización para la ejecución (0-600; `0` para que no haya espera de estabilización). La espera está limitada al tiempo de estabilización más 90 s (210 s por defecto, con un mínimo de 90 s), y el comando espera al backend 240 s más el tiempo de estabilización (360 s por defecto). Muestra el mensaje `PTP: locked after <s> s (each camera locked <settle> s, already locked: SN:<serial>, …; from the grandmaster: SN:<serial> <±x> us, …)`, y el registro del backend contiene una línea cada 5 s con el estado y el desplazamiento de cada cámara, cuánto tiempo lleva bloqueada y cuánto tiempo le queda: `[array-sync] PTP at <s> s: SN:<serial> Slave <±x> us for <s> s (<s> s left), … -- <s> s left, the wait gives up at <s> s`. Todas las cámaras se encienden con el PTP desactivado; a partir de ahí, los relojes se sincronizan en unos 10 s. Posteriormente, cada cámara se restablece tal y como la encontró la prueba: su PTP, la exposición automática y la ganancia, que la prueba mantiene desactivadas durante su ejecución, y el resto (a menos que sea `--keep-settings`) — de modo que una cámara que se haya quedado con el PTP activado (`lattice configure -s <serial> --set PtpEnable=True`, o una ejecución con `--keep-settings`) permanece sincronizada, y una vez que este backend ha comprobado que ha permanecido sincronizada durante el tiempo de estabilización, una prueba posterior no espera a que lo haga. Si los relojes no se estabilizan durante la espera, falla antes de activarse (`PTP: NOT locked …`, salida `1`) y indica qué hacer: una cámara que nunca se ha bloqueado (`PtpStatus Listening`) significa que el «grandmaster» no la está alcanzando (`time-sync status`); una que se ha ido desconectando continuamente se identifica con el tiempo que ha estado bloqueada (`… had not been locked for the 120 s settle time … (SN:<serial> locked <s> s, <s> s to go)`); relojes que siguen convergiendo: comprueba `time-sync status`, y para que estén sincronizadas antes de la prueba, activa el PTP y déjalo activado (`lattice configure -s <serial> --set PtpEnable=True` para cada cámara, o ejecuta la prueba con `--keep-settings`; la prueba restablece lo que ha detectado, por lo que permanece activado), y luego ejecútala. |
| `lattice multi-detect [--line LINE] [--json]` | Detecta las cámaras en la línea de sincronización M8 y una cámara maestra por defecto: la cámara RGB (modelo que termina en `-FRGB`) si está conectada por cable; de lo contrario, la primera cámara conectada por cable según el número de serie. Los cables M8 se unen en un punto de conexión, por lo que **cualquier cámara conectada puede ser la maestra** — el `MASTER` impreso es un valor por defecto, no un dato de cableado. Cada cámara, por turnos, activa la línea y se comprueba si las demás emiten su pulso; una cámara ya detectada en el grupo cableado no se vuelve a sondear, por lo que una matriz totalmente cableada tarda una ronda. Si no hay `--line`, comprueba `Line2`, luego `Line3`. Imprime `MASTER: SN:…`, un `SLAVE: SN:…` por cada otra cámara conectada y `(probed on <line>)`; `--json` añade el resultado completo (`master`, `slaves`, `no_cable` —cámaras a las que no llegó el pulso— `line`, y por cámara `results`). Cuando termina, cada cámara recupera los ajustes de disparo y línea de sincronización que tenía (los mismos nodos que los anteriores). |

#### Alineación

| Subcomando | Finalidad |
| --- | --- |
| `lattice align-calibrate [--serials SN1,SN2,…] [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N] [--brightness N]` — más controles de detector/comparador `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, parámetros de RANSAC `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, combinación de fotogramas múltiples `[--averaging mean\|median\|inlier_weighted]`, restricciones geométricas `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, restricción espacial `[--roi X0,Y0,X1,Y1] [--mask PATH]` yesclavo `[--per-cam-override SN:KEY=VALUE]` (repetible) | Calcula el perfil de alineación a partir de cámaras en tiempo real. Valores por defecto: `--method orb`, `--model rigid`, `--frames 3`, `--save alignment_profile.json` (en la carpeta desde la que se ejecuta), `--max-features 2000`, `--ratio-threshold 0.75`, `--matcher bf`, `--knn-k 2`, `--ransac-threshold-px 3.0`, `--ransac-iters 2000`, `--ransac-confidence 0.99`, `--averaging mean`. (`project align calibrate` tiene sus propios valores predeterminados; consulta [su tabla](#project-align-calibrate-options).) `--prefilter` utiliza por defecto `gradient` (mapa de bordes; coincide con el alineador de la interfaz gráfica de usuario/matriz: los bordes se conservan entre bandas espectrales). `--matcher flann` da buenos resultados a partir de unas 5000 características; `--averaging median` es robusto ante una captura defectuosa; `inlier_weighted` pondera según el recuento de coincidencias; `--lock-scale` proyecta hacia la rotación más cercana (sin escala), `--lock-axis` pone a cero un componente de traslación; `--mask` se aplica a todas las cámaras (utiliza `--per-cam-override` para ajustes por cámara, p. ej., `--per-cam-override 214701292:method=phase`). `--rms-threshold-px` rechaza guardar una calibración cuyo RMS de reproyección supere el umbral. `--method manual` toma sus puntos de `--points PATH`: un objeto JSON, `{"<serial>": [[x, y], ...], ...}`, que proporciona las coordenadas de píxeles de cada cámara para las mismas características, incluida la cámara de referencia (al menos 2 por cámara); cada uno de los `--frames` se resuelve a partir de esos puntos. Cuando un array conectado contiene las cámaras, la calibración utiliza los fotogramas en directo de ese array, y el perfil se convierte también en la alineación del array para sus capturas. De lo contrario, abre un [array transitorio](#multi-camera-transient-sessions): si una cámara no envía ningún fotograma en ninguno de los `--frames`, la calibración falla (salida `1`, asignándole un nombre) y no se guarda ningún perfil. |
| `lattice align-apply --profile PATH [--serials SN1,SN2,…] [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N] [--brightness N]` | Captura un fotograma multibanda alineado. `--bit-depth` se ajusta por defecto a la cámara; `--no-crop` conserva el fotograma completo (rellenándolo con negro); `--interpolation` (por defecto `linear`) y `--border-mode`/`--border-value` (por defecto `constant`/0) controlan la deformación de la CPU; la ruta de la GPU es bilineal en cualquier caso. Abre una [matriz transitoria](#multi-camera-transient-sessions): una cámara que no envía ningún fotograma falla la captura (sale de `1`, nombrándola), y no se escribe nada. |
| `lattice align-stream --profile PATH [--serials SN1,SN2,…] [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N] [--brightness N]` | Transmite fotogramas multibanda alineados (mismos controles de warp que `align-apply`), `--fps` por defecto `3`. Abre una [matriz transitoria](#multi-camera-transient-sessions): no se guarda un grupo de fotogramas para el que una cámara no ha enviado nada, y tres seguidos dan por finalizado el flujo. Si se ha omitido algún grupo, la ejecución finaliza mostrando `Incomplete: <n> frame group(s) not saved, a camera delivered nothing (SN:<serial> x<count>, …)` y `Error: …`, y sale con el mensaje `1`. Una transmisión que finaliza por cualquier otro error también muestra el mensaje `Error: …` y sale con el mensaje `1`. |
| `lattice align-info --profile PATH [--json]` | Mostrar detalles del perfil. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | Cambiar el orden de las capas. |

#### Índice / Cálculos de vegetación

```bash
# Offline: compute NDVI from an aligned multi-band TIFF (an RGN camera's bands)
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: open the cameras, capture one aligned set with a profile made by
# `lattice align-calibrate`, and compute the index from it
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --save-multiband -o output/
```

**`--channel` asigna los símbolos de un preajuste a las bandas** (NDVI: `red`, `nir`), uno `--channel SYMBOL=BAND` cada uno.

- **`--live`: solo para un símbolo al que la pila no puede dar respuesta.** Al igual que con `--input`, un símbolo de un preajuste que se omita tomará la única banda de la pila (las bandas de las cámaras habilitadas del perfil) cuyo nombre comience por él. Esto se comprueba antes de que se abra ninguna cámara: un símbolo al que no responda ninguna banda, o varias bandas, se rechaza entonces, nombrándolas, y se sale de `1`. Un `--formula` personalizado necesita un `--channel` para cada símbolo.
- **Los productos `--live` indican a quién pertenecen.** Los `index.tif` y `index_LUT.png` contienen los datos de la cámara de la pila (marca, modelo, número de serie), su hora de captura (la exposición de la cámara de referencia, UTC) y el objetivo, y ambos designan el índice como su única banda (`XMP-Camera:BandName` `NDVI`; el archivo PNG en color designa el índice, no sus canales de color), tal y como hace un producto `--input`. En cambio, un índice formado por bandas de varias cámaras nombra cada una de ellas, y su `Model` es `LATTICE` — véase [Archivos creados a partir de varias cámaras](#files-made-from-several-cameras).
- **`--input`: solo para un símbolo que el archivo no puede resolver.** Un símbolo que se omita tomará la única banda del archivo cuyo nombre comience por él: `red` toma `Red_660` o `Red_660_RGN`, y `nir` toma `NIR_850`. Los nombres proceden del propio archivo: la lista de bandas de la pila o, en una captura de radiancia/reflectancia de Chloros o en un producto `process`, sus etiquetas `Chloros:Band{R,G,B}Name` / `XMP-Camera:BandName`. Por lo tanto, `lattice index --input <reflectance capture> --preset NDVI` no necesita `--channel`.
- **No se distinguen las dos bandas que corresponden a un mismo símbolo** (una pila de 9 cámaras tiene dos bandas NIR): asigna un nombre a esa. Un símbolo al que no se le asigna nada se rechaza con `channel_map missing entries`; sal de `1`.**Los nombres de las bandas en una pila** indican el filtro de la cámara: las bandas de una cámara M3C son `<band>_<filter>` — `NIR_850_RGN`, `Green_550_RGN`, `Red_660_RGN` en un RGN; `NIR_808_OCN`, `Cyan_490_OCN`, `Orange_615_OCN` en un OCN; `Blue_475_RGB`, `Green_550_RGB`, `Red_625_RGB` en un RGB; `Blue_475_NGB`, `Green_550_NGB`, `NIR_850_NGB` en un NGB — y labanda única de la M3M es `Mono_<filter>` (`Mono_660`, `Mono_850`). `lattice align-info --profile PATH` enumera las bandas de un perfil.

Conjunto completo de indicadores: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI/NDRE/EVI/SAVI/GNDVI/…), `--formula EXPR`, `--channel SYM=BAND` (repetible), `--capture-level raw|debayered|radiance|reflectance|unknown` (anula el nivel de captura registrado en el archivo TIFF de origen; por defecto: se lee de los metadatos del TIFF, recurriendo en su defecto al  de un archivo Chloros001565, por lo que una captura de reflectancia indica `reflectance`), `--output PATH` / `-o` (fuera de línea: una ruta; si se incluye una extensión de imagen, los archivos se nombran a partir de su raíz — `-o ndvi_FRGN.png` escribe `ndvi_FRGN.tif` y `ndvi_FRGN_LUT.png` — y, si no la hay, se utiliza un prefijo, `-o ndvi` -&gt; `ndvi.tif`, `ndvi_LUT.png`; por defecto, junto a la entrada, como `<input name>_<preset>`; con `--live` y sin `-o`, una carpeta `output/` dentro de la carpeta desde la que se ejecuta el comando), `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. Con `--live`, los controles de deformación de alineación también se aplican a: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`, `--brightness N` (el objetivo de exposición automática de las cámaras; véanse las reglas de transitorios), `--serials SN1,SN2,…`. `--live` abre un [matriz de transitorios](#sesiones-transitorias-multicámara): una cámara que no envía ningún fotograma hace que la ejecución falle (salida de `1`, asignándole un nombre), y no se escribe ninguna pila ni índice. Abre `--serials` (la primera activa el disparador); de lo contrario, las cámaras del perfil cuya entrada `layer_order` esté habilitada, empezando por la de referencia —la de referencia incluso con sus bandas desactivadas, ya que las demás están alineadas con ella y activa el disparador. Una cámara cuyas bandas estén desactivadas nunca se calibra, por lo que una cámara RGB desactivada en el perfil ya no falla en un índice de reflectancia (`no-absolute-radiance-scale`). Un perfil que no nombre ninguna cámara abre todas las cámaras de la red, como antes.

Todos los formularios se ejecutan en el backend, por lo que este debe estar en marcha: inícialo desde la interfaz gráfica de usuario de Chloros, o con el comando que muestra la CLI cuando no puede acceder a él. Sin conexión, el backend lee `--input` y escribe el archivo TIFF de tipo float y el PNG coloreado él mismo, por lo que un `--input` relativo o un `-o` se resuelve en función de la carpeta desde la que se ejecuta el comando (véase [Convenciones](#conventions)). Ambos productos sin conexión incluyen el registro de la cámara de la entrada, al igual que lo hace una exportación de índice `process`: marca/modelo/número de serie, GPS, las etiquetas del objetivo, la hora de captura (la propia de la entrada; cuando sus datos EXIF no indican ninguna, el instante de exposición que registró o la marca de tiempo del nombre del archivo, en UTC) y la identidad de calibración de MAPIR —nunca la calibración raw-DN, que no se aplica a un índice—, además de `Chloros:ProcessingLevel=index` y la expresión del índice; el TIFF de tipo float designa una banda, el preajuste. `--list-presets` y `--list-gradients` imprimen las tablas de preajustes y gradientes del backend. Una fórmula que el backend rechaza, o una banda `--channel` que el TIFF no tiene, sale de `1` con el motivo del backend.

> **Los símbolos `--channel` distinguen entre mayúsculas y minúsculas.** La parte del símbolo debe coincidir exactamente con los nombres de los canales del preajuste (los preajustes utilizan minúsculas; p. ej., NDVI = `red`, `nir` — comprueba `--list-presets`), y la parte de la banda debe coincidir con un nombre de banda de la pila alineada (o ser un índice de banda basado en 0 en modo offline). `--channel red=Red_660_RGN --channel nir=NIR_850_RGN` funciona; `--channel RED=660` falla con un error `channel_map missing entries` .

#### `lattice index` — máscaras de pila multibanda

En `--capture-level radiance` o `reflectance`, la desmezcla 3×3 de cada cámara puede limitar una banda a 0 (véase [Máscara de recorte de desmezclado](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on)). Una pila multibanda contiene bandas de varias cámaras, y cada cámara limita la suya propia, por lo que `--live --save-multiband` escribe **una máscara por cada cámara de origen** junto a la pila:

```
output/
├── index_multiband.tif
└── clamp_masks/
    ├── index_multiband_SN<serial A>-<filter A>_clampmask.tif
    └── index_multiband_SN<serial B>-<filter B>_clampmask.tif
```

- Cada máscara es `uint8` de una sola banda en la cuadrícula de píxeles de la **pila** (mediante la misma deformación de alineación y recorte), con los bits propios de esa cámara: bit 0 = la banda de su canal R, bit 1 = G, bit 2 = B. Una banda de la pila tomada del canal *c* de una cámara se limita donde está activado el bit *c* de esa cámara (R = 0, G = 1, B = 2).
- La procedencia incrustada de cada banda indica el nombre de la máscara de su cámara (`unmix_clamp_mask`, en relación con la carpeta de la pila, o `none`) y el bit que se debe leer (`unmix_clamp_mask_bit`).
- Una cámara cuyo proceso de separación de canales no haya limitado ningún valor no obtiene ningún archivo; una cámara monocromática o RGB, al no ejecutar la separación de canales, tampoco obtiene ninguno. La CLI muestra cada máscara debajo de la pila (`clamp mask SN…: …`); la respuesta las enumera como `clamp_masks`.
- El TIFF/PNG de índice de la misma ejecución se calcula a partir de la misma pila, en la misma cuadrícula, por lo que las máscaras también lo describen. `--capture-level raw` no ejecuta ningún proceso de separación de canales y no escribe ninguna máscara.
- La pila y el índice se encuentran directamente en `--output`, por lo que las máscaras van en el propio `clamp_masks/` de `--output` —nunca por encima de la carpeta que has nombrado—. Esa carpeta contiene distintos tipos de productos (pila, índice, renderizado coloreado): selecciona sus archivos, no pases la carpeta completa al software de fotogrametría.

#### Conexiones persistentes (Smart-Prep, flujo equivalente a la interfaz gráfica de usuario)

Estos comandos mantienen las cámaras abiertas en el grupo de fondo entre invocaciones de lCLI.

| Subcomando | Finalidad |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Añade una cámara al grupo (una sola cámara, sin matriz). Las [opciones comunes](#common-options-shared-by-most-subcommands) `--exposure`, `--gain`, `--preset`, `--pixel-format`, `--width` / `--height` y `--packet-size` (`standard` = 1500, `jumbo` = 9000, o bytes; sondas `auto`) se aplican al conectar (un `--exposure` o un `--gain` desactiva el modo automático). Devuelve un resultado una vez que ha comenzado la transmisión de la cámara (esperando unos 15 s para ello y, a continuación, 10 s para su primer búfer completo) e imprime `stream: streaming` o `stream: no_frames -- connected, no frames yet: …` cuando la transmisión no ha proporcionado nada — en el caso de una cámara que utilice paquetes jumbo, con la recomendación de volver a conectarse con `--packet-size standard`. Mientras una cámara se conecta (la primera conexión descarga su calibración, unos 70 s), otro comando que requiera una cámara espera por ella un máximo de 5 s, luego falla con el código HTTP 503 `CAMERA_POOL_BUSY` indicando la cámara que se está conectando; vuelve a ejecutarlo al cabo de un momento. `lattice status` y la lista de cámaras nunca esperan: una cámara en proceso de conexión aparece como «conectándose», con su fase. |
| `lattice cam-disconnect [--serial SN] [--all] [--keep-settings \| --clear-settings]` | Liberar. La cámara se restablece tal y como se encontró al conectarse (cableado del disparador, PTP, ajustes); `--keep-settings` guarda sus ajustes en su propia memoria, mientras que `--clear-settings` restablece esa memoria a los ajustes de fábrica (véase [Memoria de la cámara](#camera-memory----keep-settings----clear-settings)). Una cámara que forma parte de una matriz libera toda la matriz. |
| `lattice cam-list` | Enumera las cámaras del grupo. |
| **`lattice array-connect`**|**Conectar una matriz sincronizada persistente (el punto de entrada recomendado).** Ejecuta el flujo completo de preparación inteligente de la interfaz gráfica de usuario. Tarda aproximadamente un minuto para 9 cámaras; justo después de que las cámaras se enciendan puede tardar varios. Mientras espera, muestra cada paso que alcanza el backend, como `    <s> s: <step>` (por ejemplo, `    41 s: open SN:214000535 (4 of 9)`), y el registro del backend contiene una línea de tiempo por paso (`[array-connect <id>] <step>: <s> s …`) y una última línea con los pasos más lentos. Espera hasta 600 s. Si la conexión no se ha establecido para entonces, el backend la anula: cada cámara que haya abierto se devuelve al estado en que se encontró y se cierra; el comando muestra `Error:` con el paso al que había llegado y sale en `1`, sin que haya ninguna matriz conectada (504 HTTP `CLIENT_DEADLINE_PASSED`). Ctrl+C cancela la conexión de la misma manera (`Interrupted: …`); si la matriz ya se había conectado, la línea lo indica y menciona el `array-disconnect` que la libera. Un comando interrumpido mientras espera (`kill -9`) hace que la conexión quede sin completar cuando el backend no ha recibido respuesta durante 30 s. Una matriz que está activa pero que aún no tiene un grupo de tramas sincronizado sigue apareciendo como conectada, con su flujo indicado como `no_frames`. |
| `lattice array-disconnect [--array-id ID] [--all] [--keep-settings \| --clear-settings]` | Liberar una matriz. Todas las cámaras quedan en funcionamiento libre (`TriggerMode` `Off`), y todo lo demás se mantiene tal y como se encontró al establecer la conexión: sus ajustes de disparador y línea de sincronización (el`TriggerSource` vuelve a su estado anterior, y su línea de sincronización vuelve a ser una entrada), su PTP y los ajustes de la cámara que la sesión modificó; de este modo, un comando para una sola cámara en un miembro funciona de inmediato. `--keep-settings` guarda los ajustes de cada cámaratal y como quedaron al finalizar la sesión en la propia memoria de la cámara; `--clear-settings` restablece esa memoria a los ajustes de fábrica. Véase [Memoria de la cámara](#camera-memory----keep-settings----clear-settings). |
| `lattice array-list` | Mostrar una lista de matrices conectadas. |
| `lattice array-status [--array-id ID]` | fps en directo, PTP, último error. `ptp_enabled` es igual a `True` solo cuando todos los miembros tienen PTP activado; `ptp_off:` indica un miembro que no lo tiene. La línea `state:` indica `restarting (applying settings)` durante un reinicio de la configuración, `recovering (…)` mientras se vuelve a iniciar un miembro que ha perdido alimentación (véase la nota más abajo), `stalled (…)` mientras el bucle de captura se ejecuta pero no publica ningún grupo de tramas, y `stopped (…)` una vez que el bucle de captura ha desistido (vuelve a conectar la matriz). |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--orient] [--index\|--no-index] [--no-daq] [--daq-tolerance-ms MS] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | Una captura sincronizada desde la matriz en directo: Única / Continua / A intervalos / Más rápido. **Por defecto, `all`**(un archivo por tipo de exportación aplicable por cámara),**alineado** según la captura de la aplicación (`--no-aligned` para archivos nativos). Las cámaras omitidas (p. ej., RGB, excluidas de la radiancia/reflectancia) se registran con `Skipped: SN:<serial> (<reason>)`; la lectura del DAQ utilizada para la captura se guarda junto con ella, se incluye una captura solo en formato sin procesar y se registra con `DAQ: <path>` ([El DAQ que lee una captura guarda](#el-daq-que-lee-una-captura-guarda)). Véase [Modos de captura, grabadoras y reprocesamiento fuera de línea](#modos-de-captura-grabadoras--reprocesamiento-fuera-de-línea). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only] [--expr EXPR]` | Graba la vista combinada en tiempo real del índice en vídeo o GIF (calidad de monitorización; sin ninguna vista de la interfaz gráfica abierta, el backend renderiza el NDVI de la matriz, o `--expr`). |
| `lattice array-burst [--duration S] [--max-frames N] [--orient] [--build] [--products …]` | Ráfaga de datos Bayer sin procesar a alta velocidad de fotogramas (calidad de análisis; reprocesar sin conexión). Al igual que `array-record` y `array-capture --continuous`, se comunica con el backend en cada sondeo: se cancela sin detenerse (`kill -9`); el backend lo detiene 30 s después del último registro (`stop_reason` `client_gone`; se finaliza el manifiesto de la ráfaga y se inicia la creación de su índice combinado, tal y como ocurre con una detención) y la matriz permanece conectada. |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | Reprocesar una ráfaga sin procesar guardada en uno o varios vídeos calibrados. |

> **Una cámara que pierde alimentación o su conexión se recupera por sí sola.** Cuando un miembro deja de responder (un ciclo de encendido y apagado del conmutador reinicia todas las cámaras a las que alimenta), la matriz espera a que vuelva, la vuelve a conectar cuando responde de nuevo, y le devuelve lo que `array-connect` había configurado: PTP, la línea de disparo y sincronización, el ritmo de red, el formato de píxeles y el tamaño de fotograma, y la exposición automática. A continuación, vuelve a transmitir sin ningún nuevo `array-connect`. Mientras tanto, `array-status` muestra `state: recovering (...)` y `last_error` indica a qué cámara está esperando; a continuación, imprime `re-opened:`. Una cámara que se ha reiniciado vuelve con el PTP desactivado, por lo que, hasta que se vuelve a abrir, `ptp_enabled` es `False` y `ptp_off:` le asigna un nombre.

> **Una cámara retenida nunca se abre por segunda vez.** Una segunda apertura detiene el flujo del retentor y sobrescribe su configuración, lo que puede provocar un fallo del backend. Así pues, en una cámara que mantiene el grupo (la interfaz gráfica de usuario o `cam-connect`), un miembro de una matriz o una cámara que mantiene una tarea en segundo plano (un `lattice continuous` en ejecución, `multi-stream` o `align-stream`, ya sea propio o de otro cliente), `lattice continuous`, `lattice capture`, `cam-connect`, `power --sleep`, `calibrate`, un-shot `white-balance` y `presets --apply` rechazan con un código HTTP 409 `CAMERA_HELD`, indicando el titular y el comando de liberación (`chloros-cli lattice cam-disconnect -s <SN>`, `chloros-cli lattice array-disconnect --array-id <ID>`, o, en el caso de un trabajo, su ID y quién lo inició), y salir `1`. Lo mismo ocurre con una cámara que otro comando está utilizando en ese momento (un `multi-test` o un `multi-capture` que está abriendo sus cámaras, otro comando de un solo uso); para cada comando de esta lista, el error indica que hay que esperar a que ese comando termine. `lattice power` sin `--sleep` lee a través del identificador propio del titular, cuando existe uno (el grupo, una matriz, una cámara abierta de un trabajo), y se rechaza si hay una cámara que esté utilizando otro comando. Cada respuesta de `CAMERA_HELD` lleva el código tanto en `code` como en `error_code`. Para los comandos que abren todas las cámaras, véase [Multicámara (sesiones transitorias)](#multi-camera-transient-sessions).

##### Opciones de `array-connect`

| Indicador | Predeterminado | Descripción |
| --- | --- | --- |
| `--serials SN1,SN2,…` | Detección automática de todas las cámaras LATTICE (se necesitan ≥2) | La primera serie es la MAESTRA. Si se omite, la detección se filtra a los modelos LATTICE (M3C / M3M) y se conectan todos ellos. |
| `--line {Line0,Line2,Line3}` | `Line2` | Línea de sincronización GPIO. |
| `--target-fps F` | automático | Frecuencia de disparo del maestro. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | automático | Anular la selección de nivel. |
| `--wire-ceiling-mbps MB_PER_S` | detección automática | **El presupuesto de ancho de banda sostenido del host, en MB/s: el valor del que depende toda la asignación de la matriz.** Redúcelo cuando la matriz informe de tramas GVSP dañadas: el valor automático se deriva de la velocidad de enlace anunciada por la tarjeta de red, que sobreestima los adaptadores USB, los carriles PCIe de ancho reducido y las estructuras compartidas con mucho tráfico. Se almacena de forma persistente en el bloque de captura de la matriz del proyecto, por lo que se restaura al reabrir o al volver a conectarse mediante la CLI o el SDK. Véase [Estado de la matriz](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | ninguno (sensor completo) | Agrupación por hardware (siempre «Promedio»). |
| `--width N` / `--height N` / `--pixel-format FMT` | sensor completo; formato por cámara | Un tamaño de fotograma o formato de píxeles explícito para cada cámara, en lugar del sensor completo y la selección del formato por cámara (Mono12 / BayerRG12). |
| `--no-recommend` | desactivado | Omitir el paso de análisis de red (cada cámara mantiene entonces su tamaño actual en lugar de configurarse con el sensor completo). |
| `--no-ptp` | desactivado | Desactivar PTP (las marcas de tiempo entre cámaras **no** serán comparables). |
| `--packed {auto,on,off}` | `auto` | Transmite fotogramas de 12 bits empaquetados y desempaquétalos en este ordenador: más fotogramas por segundo con el sensor completo, datos idénticos. `auto` solo empaqueta cuando sea útil y este ordenador pueda soportarlo; `off` nunca empaqueta. Véase [Transporte empaquetado de 12 bits](#packed-12-bit-transport---packed). |
| `--packet-size {auto,jumbo,standard,N}` | `auto` | Tamaño de paquete GVSP para cada cámara de la matriz. `auto` permite que cada cámara sondee la red; `jumbo` = 9000, `standard` = 1500 o un tamaño en bytes lo bloquea. Antes de la versión 1.3.0, `array-connect` aceptaba este indicador y lo ignoraba. |

> **Sensor completo por defecto.** Sin `--width` / `--height` / `--binning` /
> `--pixel-format`, `array-connect` configura todas las cámaras en el sensor completo —sin
> binning, sin recorte de ROI — y permite que la velocidad de fotogramas descienda hasta lo que este ordenador y
> la red puedan ofrecer, en lugar de reducir los fotogramas para mantener la velocidad. La
> línea de análisis de red muestra la velocidad prevista, y la siguiente línea enumera
> opciones más rápidas (agrupación de píxeles o un ROI más pequeño) con sus velocidades previstas; pasa
> `--binning` o `--width`/`--height` para seleccionar una. «Connect» nunca realiza binning ni recortes
> por sí mismo.

##### Transporte empaquetado de 12 bits (`--packed`)

Con el sensor a pleno rendimiento, la velocidad de fotogramas de una matriz viene determinada por la rapidez con la que cada cámara puede enviar
sus fotogramas. Los formatos de 12 bits (`BayerRG12`, `Mono12`) suelen transmitirse como palabras de 16 bits;
el transporte **compactado** envía los mismos 12 bits en 1,5 bytes por píxel y
los descomprime en este ordenador, por lo que cada fotograma tarda un 25 % menos en enviarse —
aproximadamente un **33 % más de fotogramas por segundo a resolución máxima del sensor** (medido: 10,7 → 14,0 fps
en una matriz de 9 cámaras). **Los datos guardados son idénticos**: mismos valores, mismos
archivos de 16 bits, mismos metadatos.

- `auto` (predeterminado) comprime solo cuando resulta útil **y** este ordenador puede permitirse
  la descompresión. Al conectarse, el backend mide su propia velocidad de descompresión, el número de núcleos de CPU,
  la carga actual de la CPU y el ancho de banda de red disponible, de modo que un ordenador de sobremesa con núcleos de sobra comprime
  a plena capacidad del sensor, mientras que un ordenador ocupado o de bajo consumo transmite sin comprimir. Las regiones de interés (ROI) pequeñas y los fotogramas agrupados se benefician
  poco.
- `off` nunca comprime: utilízalo para desactivar la compresión durante el diagnóstico, o en un
  ordenador que deba reservar toda su CPU para otras tareas.
- `on` siempre comprime formatos de 12 bits.

La salida de «connect» muestra la decisión y el motivo («comprimido a 12 bits: en (automático:
proyección de 11,0 -&gt; 14,4 fps, …)`); `/api/camera/array/estado` informa de
`packed_transport` y `packed_decision`. Para establecer un valor predeterminado para todo el sistema
en lugar de pasar el indicador cada vez, configura `CHLOROS_ARRAY_PACKED12=off` (o
`on`) en el entorno del backend; no obstante, un `--packed on|off` explícito sigue teniendo prioridad.
Los formatos de 8 y 16 bits nunca se empaquetan.

### Un único registro de proyecto para todos los modos

Una carpeta de proyecto contiene la configuración de todos los dispositivos: `cameras.json` —la de cada cámara (`cameras[].settings`, para una sola cámara y para cada miembro de la matriz) y las de toda la matriz (`arrays[].capture`: formato, ROI, binning, frecuencia de disparo, presupuesto de cableado), así como `sensors.json`, los de cada sensor de luz (tiempo de integración, promedio de fotogramas, límite). La aplicación, la CLI, el SDK y el servidor MCP leen y escriben los mismos archivos, de la misma manera:

- **Un dispositivo conectado a un proyecto se guarda en él y registra en él cada cambio.** «En un proyecto» significa con un proyecto abierto en el backend (el de la aplicación) o —desde el SDK— a través de un `ChlorosProject`, que designa su propia carpeta. Una cámara, una matriz o un sensor de luz que el proyecto aún no contenga se añade a sus archivos cuando se conecta (`lattice cam-connect`, `array-connect`, `daq pool-connect`, la aplicación, el SDK, `chloros-mcp`), junto con su configuración. Cualquier cambio en cualquier modo (`lattice exposure`, `gain`, `configure --set`, `resolution`, la configuración de una matrizy la frecuencia de disparo, el límite del sensor de luz, los controles de la aplicación) se guarda en los archivos de ese proyecto un instante después. Una exposición, ganancia o objetivo de exposición automática (AE) configurados en una cámara se guardan como valores memorizados, por lo que al volver a abrirla se mantienen.
- **Al abrir un proyecto, se configuran sus dispositivos según los datos del mismo**, en todos los modos: una cámara, una matriz o un sensor de luz conectados que no tengan ajustes propios adoptan los ajustes guardados del proyecto (`lattice cam-connect` / `array-connect` con un proyecto abierto, `chloros-cli project connect`, `open_project().connect_all()`, al volver a abrir la aplicación). Una matriz conectada con un nuevo ID actualiza la matriz que el proyecto guardó con las mismas cámaras; una matriz con otras cámaras se añade junto a ella.
- **El guardado propio de la aplicación nunca sobrescribe los ajustes en tiempo real de un dispositivo** con su copia anterior de los mismos, y conserva las modificaciones por miembro que haya guardado quien haya llamado desde la CLI o el SDK.
- **El formato de píxeles que se configure es el que conserva el proyecto.** Un formato establecido en la configuración de la matriz, mediante `--pixel-format` o mediante `connect_array(pixel_format=)`, incluido el de 8 bits, se guarda y se vuelve a conectar exactamente igual. Si no se establece ninguno, cada cámara se conecta con su valor predeterminado de 12 bits (`BayerRG12`, `Mono12` en un sensor monocromo). El nombre de un formato indica su profundidad de bits; la familia corresponde al sensor, por lo que una cámara en color nunca se configura en un formato monocromo ni una cámara monocroma en uno de tipo Bayer, y la línea `[PixelFormat <serial>] ... written` lo indica cuando se ha asignado un formato. Las exportaciones de una captura de 8las exportaciones de una captura de 8 bits son de 8 bits, incluida la reflectancia: reescaladas y marcadas con su `PixelScale`.
- Un dispositivo conectado sin estar asociado a ningún proyecto no graba nada.
- **Una grabación sigue el proyecto creado después de ella; al cerrarse, conserva su proyecto.**Cuando se crea un proyecto —un SDK `create_project()`, el `chloros-mcp` de `create_project` o el**Nuevo proyecto**de la aplicación — o se duplica (la opción**Duplicar**de la aplicación, que abre la copia) mientras una grabación sigue escribiendo en el proyecto que estaba abierto anteriormente, la grabación continúa y lo que escribe acaba en el nuevo proyecto, en el mismo lugar dentro de este (más abajo). Al abrir un proyecto existente — `chloros-cli process`, `project devices --reconnect`, `ChlorosProject.process()`, mediante la opción**Abrir proyecto** de la aplicación — hace que la grabación siga en curso en el proyecto en el que se está grabando: finaliza allí y sus archivos se añaden a la lista de archivos de ese proyecto (la aplicación detiene todas las grabaciones antes de abrir un proyecto).
  - `array-record` y `array-burst`: el vídeo, o la carpeta de ráfagas con todos los fotogramas, se traslada íntegramente allí cuando se detiene la grabación (su `--duration`, Ctrl+C). El `Burst dir` que se genera es el nuevo, y `--build` y `array-build-video` operan a partir de él. Si la unidad rechaza el traslado, la grabación permanece donde se escribió; eldel registro indica el motivo, al igual que la respuesta HTTP de la parada (`relocation_error`). Los fotogramas sin procesar de una ráfaga detenida y el `.daq` se añaden a la lista de archivos del proyecto en el que terminan; un vídeo (un `array-record`, el vídeo generado a partir de una ráfaga) no se añade a la lista de archivos de un proyecto.
  - `array-capture --continuous`: todas las capturas posteriores al cambio se guardan en el nuevo proyecto; las guardadas anteriormente permanecen donde estaban.
  - `daq pool-record` y `project sensor log` (y la grabación del sensor de luz de la propia aplicación): el `.daq` se traslada a la nueva carpeta del proyectocarpeta `light_sensor` del nuevo proyecto cuando se detiene la grabación, y se añade a la lista de archivos de ese proyecto.

  Una grabación en una carpeta fuera del proyecto abierto (un `-o` en otra ubicación) permanece donde está, y `lattice continuous` y `multi-stream` conservan la carpeta que se les asignó.
- **En la aplicación, las capturas realizadas tras crear un nuevo proyecto se guardan en este.** Al crear o duplicar un proyecto, todas las sesiones de captura y grabaciones en curso continúan, y cada captura realizada a partir de ese momento («Capturar todo», «Iniciar captura») se guarda en el nuevo proyecto y se añade a su lista de archivos. Al abrir un proyecto existente en la aplicación, primero se detienen todas ellas.
- **Un proyecto no se mueve mientras se está grabando.**Al mover el proyecto abierto a otro directorio de trabajo (en la aplicación, en**Configuración → Directorio de trabajo**; ningún comando de la CLI mueve un proyecto, pero sí lo hace un cliente HTTP que envía datos a `/api/move-project`) se rechaza con el código HTTP 409 `PROJECT_RECORDING` mientras una grabación en matriz o en ráfaga, una captura continua en matriz, un trabajo `lattice continuous` / `multi-stream` o una grabación del sensor de luz esté escribiendo en él: `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.` La aplicación rechaza la operación de la misma manera, antes de solicitar una carpeta, mientras se ejecuta su propia función «Capturar todo», «Iniciar captura» o una grabación del sensor de luz.

Esto es independiente de cómo quedan las cámaras al desconectarse (véase [Memoria de la cámara](#camera-memory----keep-settings----clear-settings)): un proyecto configura sus propias cámaras al abrirse.

### Memoria de la cámara — `--keep-settings` / `--clear-settings`

Cuando se conecta una matriz, sus cámaras adoptan la configuración de la matriz, y a continuación puedes modificarla. Cuando Chloros libera las cámaras — `array-disconnect`, el final de un comando transitorio (`multi-capture` sin matriz, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, `index --live`), la parada de `lattice continuous`, el final de un `lattice capture` de una cámara que no tiene conectada; devuelve cada una tal y como estaba antes de la conexión: su cableado de disparador y línea de sincronización, así como la configuración de PTP y de la cámara (la lista completa se encuentra en [Multicámara (sesiones transitorias)](#multi-camera-transient-sessions)).

Para utilizar las cámaras fuera de un proyecto, dos indicadores en cualquiera de esos comandos escriben en la **propia memoria** de la cámara —la configuración con la que se enciende, independientemente del software—:

- **`--keep-settings`** guarda en la memoria de cada cámara los ajustes tal y como quedaron al finalizar la sesión (UserSet2, que se convierte en el conjunto de arranque). Las cámaras siguen funcionando de forma autónoma (`TriggerMode` `Off`), con el cableado de la línea de sincronización tal y como se encuentra. Una cámara sin UserSet2 guarda los datos en UserSet1; este ordenador lo recuerda, por lo que la autorreparación de la matriz de Chloros —lo que restablece un UserSet1 que se ha desviado de la línea de referencia de Chloros— no la altera.
- **`--clear-settings`** restablece la memoria de cada cámara a la configuración de fábrica: la línea de referencia de Chloros, la que la autorreparación de la matriz fija — valores predeterminados de fábrica con `TriggerMode`, `Off`, `TriggerSource` y `Line2`, exposición automática y el balance de blancos calibrado de una cámara RGB — se mantienen en UserSet1, que se convierte en el conjunto de arranque.
- No es posible combinar ambos. Cada cámara recibe una línea: `SN:…: settings saved to the camera's memory (UserSet2); it powers up with them.` o `SN:…: camera memory reset to the factory settings.`; la respuesta `camera_memory` es la misma para cada cámara.

La aplicación no ofrece estos indicadores.

**Cada salida restablece las cámaras a su estado original.**El mismo restablecimiento tal y como se encontró se ejecuta en cada ruta de salida, no solo en los comandos anteriores: `cam-disconnect` (y la cámara**X**de la aplicación); el**X** de la aplicación en una matriz o en la fila de cualquier miembro de la matriz (una matriz no puede seguir ejecutándose sin un miembro, por lo que se libera toda la matriz); al cerrar la aplicación (le da al backend hasta 20 s para restablecer todas las cámaras, matriz y el `lattice continuous` en ejecución antes de detenerlo; un comando que siga ejecutándose en ese momento —un comando transitorio que abra sus cámaras o espere a que sus relojes PTP se sincronicen, un `lattice capture` de una sola ejecución, `power`, `white-balance` o `presets --apply` — se detiene en su siguiente punto seguro, restablece primero sus cámaras, durante un máximo de 8 s, y responde con un 503 de HTTP: `SHUTTING_DOWN`; una conexión en serie que sigue en ejecución —`lattice array-connect`, la conexión de la aplicación— se detiene antes de su siguiente cámara y vuelve a cerrar las cámaras que ha abierto, durante un máximo de 10 s, y responde con un HTTP 503 `SHUTTING_DOWN`; y una que llega durante el apagado no abre nada: HTTP 503 `SHUTTING_DOWN`); y un comando de CLI que se interrumpe o cuya terminal se cierra (SIGTERM / SIGHUP detienen su tarea — y vuelve a cerrar sus cámaras — antes de salir (`143` / `129`, como Ctrl+C). Un `lattice continuous`, `multi-stream` o `align-stream` eliminado de forma inmediata (SIGKILL), que no puede detener nada, ve cómo el backend detiene su tarea y devuelve sus cámaras a su estado anterior 30 s después de su última actualización. Un backend que se cuelga o se cierra de forma forzada no puede restablecer nada; un reinicio devuelve cada cámara a su memoria, y la siguiente conexión de Chloros reconfigura cualquier cámara que encuentre activada.**Cada proyecto configura sus propias cámaras.** Cuando una matriz se conecta con un proyecto abierto, cada miembro obtiene los ajustes que el proyecto ha guardado para él —un valor de exposición, ganancia o objetivo de exposición automática (AE) fijado en él (considerado como propio, por lo que la exposición automática no lo modifica) y su modo de balance de blancos (corrección de Chloros; el balance de blancos propio de la cámara permanece desactivado)—, sea cual sea el contenido de la memoria de la cámara.

### Smart-AE / Smart-Capture

Las matrices LATTICE ejecutan el AE continuo en segundo plano tan pronto como se conectan, pero una escena recién enfocada tarda un momento en converger. `array-capture --smart` es la **opción más cómoda**: espera a que la exposición automática se estabilice en todas las cámaras de la matriz y, a continuación, activa la captura. Úsala cuando cambies de escena a mitad de una sesión.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

La política de estabilización es conservadora por defecto: tiempo de espera de 5 s, ventana de estabilidad de 1,5 s y tolerancia de variación de la exposición de ±5 %. Ajústala a través del SDK (`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`) si necesitas un comportamiento diferente de la automatización.

#### Cómo funciona la exposición automática

- **Primero la exposición, luego la ganancia.** Con la ganancia en «Auto», esta se mantiene en 0 dB hasta que la exposición haya alcanzado el valor máximo permitido.
- **Una exposición manual deja la ganancia en «Auto»** hasta que se modifique la ganancia o se configure en modo manual. Al volver a «Auto», el límite de la ganancia es el mismo que antes.
- **Las pequeñas zonas de alta luminosidad no oscurecen la escena.** Se ignora un reflejo especular de hasta el 1 % del encuadre y, en los paneles de un objetivo de calibración conocido, hasta el 0,5 % de un panel.
- **El límite máximo de exposición del DAQ solo se aplica mientras hay un objetivo de calibración a la vista.** Evita que un panel blanco se recorte; si no se ve ningún panel, la cámara expone según la escena.
- **La luz constante se mantiene estable, y los cambios de luz se estabilizan en segundos.** Con luz constante, la exposición y su límite permanecen fijos, y tras un cambio importante de luz, la imagen se estabiliza en pocos segundos. Un recorte ya no limita la exposición de forma permanente, y un cambio en la zona de interés (ROI) vuelve a ajustar la exposición.
- **El objetivo de brillo se mantiene al alcance.** No aumenta mientras la cámara se encuentra en su límite de exposición y su ganancia no puede subir, por lo que una luz que vuelve a aparecer no provoca un destello en el fotograma.
- **Una cámara cuya exposición automática deja de responder se mantiene en el nivel correspondiente a la luz en la que se encuentra.** Bajo un sol intenso, una cámara podría caer a su exposición más corta y quedarse ahí con un fotograma oscuro, aunque todos sus ajustes indiquen valores normales. Chloros supervisa el fotograma, mantiene esa cámara en el nivel que conservaba para esa luz y la libera cuando la luz cambia. Un sensor de luz oscuro u oculto no altera este proceso.

### Niveles de captura y exportación (el valor predeterminado de `all`)

A partir de esta versión, `lattice capture`, `lattice multi-capture` y `lattice array-capture` **vienen por defecto configurados en `--processing all`** — un archivo guardado por tipo de exportación que se aplica a cada cámara, coincidiendo con el comportamiento de «Capturar todo» de la interfaz gráfica de usuario. Los niveles son:

| Nivel | Salida | Se aplica a |
| --- | --- | --- |
| `raw` | Bayer monocanal (cámaras monocromáticas: la banda única) directamente desde el sensor. | Todas las cámaras. |
| `debayered` | Color de 3 canales, guardado en orden de canales **RGB**(un archivo TIFF con interpretación fotométrica RGB; los archivos PNG y JPEG también se abren como RGB): se elimina el desplazamiento oscuro (DSNU) del paquete de calibración del archivo RAW y, a continuación, se aplica el demosaico de OpenCV—sin LUT de linealidad, campo plano ni desmezcla (cámaras monocromáticas: escala de grises de 1 canal). Los fotogramas sin bayering en memoria que devuelve el SDK de Python (`grab()`, `frame_stream()`) están en formato**BGR**, el orden de OpenCV; solo el archivo guardado está en formato RGB. | Todas las cámaras. |
| `radiance` | float32 W/m²/sr/nm a través de la cadena radiométrica completa. | Solo multiespectral (M3C/M3M) — **se omite para cámaras con filtro RGB**. |
| `reflectance` | uint16 ρ (`32768` = 1,0), compatible con Pix4D. | Solo multiespectral, y **solo cuando hay un DAQ vinculado y la cámara está calibrada**; en caso contrario, se omite. |
| `display` | Cadena completa de vista previa en la interfaz gráfica de usuario (CCM + WB + gamma según el perfil de la cámara). Se guarda en una carpeta `preview/`. `lattice capture` también acepta el nombre `preview` para ella; `array-capture` / `multi-capture` solo admiten `display`. | Todas las cámaras. |

Indica un único nivel para guardar solo ese (`--processing debayered`). Cuando se solicita `all`, los niveles que no se aplican a una leva determinada se omiten (y se notifican), pero no se consideran errores: una cámara desconectada o sin calibrar sigue recibiendo `raw` / `debayered` / `preview`.

#### El DAQ que lee una captura guarda los datos

Cada vez que un DAQ se vincula a la cámara o al conjunto de cámaras, o se agrupa, la lectura descendente utilizada en una captura se escribe en un archivo **`.daq`** en la subcarpeta `daq/` de la carpeta de capturas, junto con su `.match.json` (la distancia a la que se encontraba la lectura respecto a la exposición) y su `.cameras.json` (de quién es la lectura), y se indica en una línea `DAQ:`. Esto se aplica a todos los niveles, incluidas las capturas sin procesar o sin debayering, por lo que cualquier captura puede volver a procesarse posteriormente en radiancia, reflectancia o un índice. Se trata del mismo archivo, con el mismo nombre y archivos complementarios, independientemente de si la captura procede de la interfaz gráfica de usuario (GUI), de la interfaz de línea de comandos (CLI) o del kit de desarrollo de aplicaciones (SDK). Si no hay ninguna lectura de DAQ, no existe el `.daq`, y un `multi-capture` sin matriz conectada (la solución alternativa transitoria) no guarda nada.

| Indicador | Efecto |
| --- | --- |
| `--no-daq` | No guardar la lectura en una captura cuyos niveles no la utilicen (`raw`, `debayered`). Un nivel `radiance`, `reflectance` o `display` sigue escribiéndola, ya que la reimportación necesita la lectura con la que se creó el producto. |
| `--daq-tolerance-ms MS` | Distancia máxima a la que puede estar la lectura respecto a la exposición y seguir considerándose dentro de la tolerancia (por defecto, 500). Solo cambia el veredicto (el `.match.json`, el `in_tolerance` y el `tolerance_ms`, así como el `DlsInTolerance` de la reflectancia), nunca la lectura utilizada. Véase [Una lectura DAQ muy alejada de la exposición](#a-daq-reading-far-from-the-exposure). |
| `--force-daq` | La forma antigua de solicitar el `.daq` en una captura solo en formato RAW. Ahora es la opción predeterminada, y `lattice capture` y `array-capture` siguen aceptando el indicador (`multi-capture` no lo hace). `--no-daq` tiene prioridad sobre él. |

¿Qué DAQ? El vinculado a la cámara o a la matriz (la selección del sensor de luz de la interfaz gráfica de usuario, un enlace guardado de un proyecto o `chloros-cli daq pool-bind`), o bien el más reciente que tenga el backend. Una ráfaga en formato RAW (`array-burst`) conserva su propio `.daq` por cada lectura distinta en la carpeta de ráfagas, como antes.

**Los formatos PNG y JPEG contienen 8 bits por canal, por lo que solo admiten los productos `display` / `preview` e indexados.** Los archivos `--format png` o `jpg` con un nivel que no pueden admitir se rechazan antes de que se capture nada, indicando el nivel y el formato TIFF que lo admite:

- `radiance` (float32) y `reflectance` (uint16, `32768` = 1,0) — siempre, por lo que `--processing all` (el valor por defecto) junto con `--format png`/`jpg` también se rechaza: utiliza `--format tiff` o `--levels display`.
- `raw` y `debayered` procedentes de una cámara que transmite más de 8 bits (los fotogramas M3C de 12 bits / Mono12 M3M): la CLI no puede detectar la profundidad de bits de la cámara, por lo que el backend los rechaza (HTTP 400). El fotograma único`lattice capture --processing debayered` de un solo fotograma captura un fotograma de 8 bits para PNG/JPEG y no es rechazado, al igual que un `multi-capture` sin matriz persistente (desbayering de 8 bits).

Todas las rutas de captura responden de la misma manera (`error_code` `FORMAT_CANNOT_HOLD_LEVEL`): `lattice capture`, `multi-capture`, `array-capture`, `project capture` / `burst`, y las rutas HTTP que hay detrás de ellas.

### Cómo es una carpeta de capturas

Cada tipo de exportación se guarda en su **propia subcarpeta**dentro de `-o`, por lo que en una captura de varios niveles nunca se mezclan los tipos. El esquema muestra la**captura predeterminada `all` de una cámara por parte de `lattice capture`**: varios niveles por cámara, por lo que cada archivo lleva el sufijo `_<level>`:

```
output/
├── raw/           capture_<ts>_SN<serial>-<filter>_raw.tiff
├── debayered/     capture_<ts>_SN<serial>-<filter>_debayered.tiff
├── radiance/      capture_<ts>_SN<serial>-<filter>_radiance.tiff
├── reflectance/   capture_<ts>_SN<serial>-<filter>_reflectance.tiff
├── preview/       capture_<ts>_SN<serial>-<filter>_display.tiff
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
├── clamp_masks/   radiance/…_radiance_clampmask.tif, reflectance/…, index/… — the unmix clamp
│                  masks, one subfolder per product folder, only when a band clamped
└── daq/           daq_data_<sensor id>_<ts>.daq: the downwelling reading matched to the capture
                   (+ .daq.cameras.json: whose)
```

Cómo varían los nombres de los archivos:

- **Campo de la cámara — número de serie, seguido del filtro:** cada archivo de captura incluye el nombre de la cámara que lo tomó como `SN<serial>-<filter>`, donde `<filter>` es la parte correspondiente al filtro del modelo de la cámara: `SN261902142-F450` para una `M3M-L87-F450`, `SN213800234-FRGN` para una `M3C-L87-FRGN`, `SN214701292-FRGB` para una cámara RGB. El filtro se une al número de serie con `-`, no con `_`, por lo que el nombre tiene los mismos campos separados por `_` que antes (prefijo, fecha, hora, milisegundos, cámara, nivel). Una cámara cuyo modelo no se pueda leer (un DeviceUserID borrado) recibe únicamente el nombre `SN<serial>`. Las capturas de versiones anteriores no tienen filtro en el nombre; Chloros lee ambas.
- **Prefijo:** `lattice capture` nombra los archivos `capture_…`; `array-capture`, `multi-capture` con una matriz conectada, y `project capture` / `burst` / `stream` de una matriz los nombra `sync_<ts>_SN<serial>-<filter>…`.
- **Sufijo de nivel solo con varios niveles.**Una cámara que guarda**varios**niveles en una captura (`all` o `--levels a,b`) añade `_<level>`. Una cámara que guarda**un** nivel mantiene el nombre sin sufijo —`--processing reflectance` se escribe como `reflectance/sync_<ts>_SN<serial>-<filter>.tiff`—, de modo que los scripts escritos para capturas de un solo nivel siguen funcionando. El nivel sigue indicándose mediante la carpeta.
- **Las máscaras de recorte siguen el nombre del producto:** `<product name>_clampmask.tif`. Una captura de reflectancia de un solo nivel genera `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif`; una captura `all` genera `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif`.
- **La extensión** sigue a `--format`: el valor por defecto `tiff` da como resultado `.tiff`, `--format tif` da como resultado `.tif` (las máscaras de recorte son siempre `.tif`).
- **Una excepción:** `lattice capture --processing debayered` (o `--levels debayered`) toma una ruta rápida de un soloy escribe `capture_<ts>_SN<serial>-<filter>.tiff` directamente en `-o`, sin subcarpeta.
- **Los flujos y las ráfagas** contienen el mismo campo: `lattice continuous` escribe `frame_<seq>_SN<serial>-<filter>.tiff`, `lattice multi-stream`, `stream_<seq>_SN<serial>-<filter>_<frame id>.tiff` y una ráfaga sin procesar `<burst>_f<frame>_SN<serial>-<filter>_raw.tiff`.

`<ts>` es la marca de tiempo de captura y `<serial>` el número de serie de la cámara, por lo que un grupo sincronizado comparte una
marca de tiempo entre todas las cámaras. **Ten en cuenta la única asimetría:** el nivel `display` se almacena en una carpeta
denominada `preview/`, mientras que los archivos de varios niveles conservan `_display` en el nombre — la carpeta y el sufijo difieren
solo para ese nivel. Los niveles desconocidos recurren a una carpeta con su propio nombre y, si no se puede crear la subcarpeta,
el archivo se guarda en la raíz de salida en lugar de perderse.

**Reprocesamiento de una carpeta de capturas:**dirige `chloros-cli process` a la**raíz de capturas**
(`output/`). `process` normalmente solo importa la carpeta que le indiques, pero cuando esa carpeta no contiene
imágenes y tiene subcarpetas, desciende automáticamente, de modo que las subcarpetas de nivel y la
subcarpeta `daq/` se recogen todas de una sola vez. Cada nivel de una captura se importa como una única imagen, y
los demás niveles están disponibles como modos, en lugar de como una imagen por nivel.

También funciona nombrar directamente una **subcarpeta de nivel** (p. ej., `output/raw/`). Al hacerlo, se omite `daq/`,
por lo que debes copiar o indicar la lectura del DAQ junto con ella cuandoderivar un producto radiométrico
a partir de `raw/`; de lo contrario, la coincidencia de la marca de tiempo no tendrá nada con lo que resolverse.

**El procesamiento siempre comienza a partir de `raw`.** Dentro de cada captura, el fotograma sin procesar es la fuente del proceso de procesamiento;
`debayered`, `radiance`, `reflectance` y `preview` aparecen como modos visualizables, pero nunca se reenvían
de vuelta a la cadena de procesamiento. Volver a procesar un producto derivado supondría volver a aplicar el viñeteado, el CCM y
los cálculos de radiancia que ya están integrados en sus píxeles, por lo que Chloros, en lugar de
realizar un doble procesamiento, opta por descartarlo. Dos consecuencias que conviene conocer:

- Las representaciones `index/` y `composite/` **nunca** se procesan. Son salidas, no capturas —
  una representación LUT de NDVI no tiene una interpretación significativa de la radiancia.
- Una carpeta de capturas exportada **sin** `raw` (p. ej., `array-capture --processing reflectance`) no tiene
  ninguna fuente válida en el flujo de trabajo. Esas capturas se importan y se muestran con normalidad, pero `process` las omite
  y lo indica así:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Si realmente necesitas pasar un producto derivado —una sesión de hub capturada con
  `demosaic` activado, o una carpeta heredada—, `--input-level {raw,debayered,processed}` fuerza el punto de entrada
  y anula el salto. Esa bandera es la vía de escape deliberada; `auto` (el valor por defecto)
  nunca procesa una captura que no contenga datos sin procesar.

**Etiquetas de calibración: solo en `raw`.** El archivo `raw` de una captura contiene el paquete de calibración de la cámara: la carga útil `CalibrationJSON` y las etiquetas compatibles con MicaSense (`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`), que describen los valores sin procesar del sensor. Todos los demás archivos de nivel (`debayered`, `display`, `radiance`, `reflectance`, el renderizado `index/`) y cada trama `lattice continuous` / `multi-stream`, contiene la identidad de la cámara (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) y el propio registro del fotograma, `XMP-mapir:FrameJSON` (exposición, ganancia, temperatura, formato de píxeles, hora de captura, coincidencia DAQ), ya que cada producto `process` lleva su propia identidad. `process` localiza la calibración de dicho archivo mediante su número de serie y `CalBundleSha256` en la caché de calibración de Chloros, descargando el paquete una sola vez si el ordenador no lo tiene.

#### Archivos creados a partir de varias cámaras

Un archivo creado a partir de las bandas de más de una cámara —una pila multibanda (`align-apply`, `align-stream`, `index --live --save-multiband`), un índice en tiempo real creado a partir de varias cámaras (`index.tif`, `index_LUT.png`), una imagen `composite/` de una captura en matriz — incluye el nombre de todas las cámaras que se han utilizado:

- **Listas por cámara, en orden de bandas** (la imagen compuesta: capa de fondo, luego primer plano; una capa de índice, las cámaras de las que proceden sus bandas): `XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceSensorProfiles`, `SourceProductLines`, `SourceFilterTypes`, `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s` (el paquete de calibración por el que pasaron las bandas de cada cámara; de lo contrario, — una pila `align-apply` / `align-stream`, cuyas bandas no pasan por ninguna — el paquete de la cámara que Chloros ha almacenado en caché, el que da nombre a sus archivos de captura), `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers` — listas paralelas, en las que el elemento *n* de cada una describe la misma cámara; `-` cuando una cámara no tiene ningún valor (sin paquete de calibración, con un objetivo desconocido). Cada cámara es **un** elemento: cuando su paquete tiene varios valores para un campo —los certificados de lámparas halógenas de un paquete—, se unen con `;` (`F-1479-1;F-1480-1;F-1481-1`), nunca con una coma, por lo que una lista muestra un elemento por cámara, independientemente de cómo se imprima.
- **EXIF `SerialNumber`** contiene los mismos números de serie, separados por comas (`214000535,214000551`).
- **No** incluye ninguna identidad de cámara individual (`XMP-mapir:CameraSerial`, `CalBundleSha256`, …): esos identifican una cámara y su paquete, y `process` determinaría la calibración del archivo a partir de ellos. Un índice calculado a partir de las bandas de varias cámaras (`index.tif`, `index_LUT.png`) tiene EXIF `Model` `LATTICE` (el equipo), y no el modelo de una sola cámara `LATT-…`.

Un archivo con las bandas de **una** sola cámara —ya sea un TIFF por banda (`--per-band`), un fotograma `align-apply --per-camera`, una pila o un índice en tiempo real de una sola cámara — lleva las etiquetas de identidad propias de esa cámara, al igual que cualquier otro archivo de captura. La pila de `align-apply`, los archivos por banda y por cámara, así como cada pila de `align-stream` y sus archivos por banda, contienen la hora de captura de su grupo (la exposición de la cámara de referencia, UTC), y un fotograma `--per-camera` de una cámara monocromática es un plano, en `--bit-depth` al igual que la pila.**Orientación de la cámara (volteo / rotación).** La configuración de orientación de una cámara (las opciones «Mirror H» /
Mirror V / rotación) de la interfaz gráfica de usuario determina lo que se guarda en una captura:

- **`raw` y `debayered` se guardan exactamente tal y como los lee el sensor**, sea cual sea la configuración. Un
  mosaico de Bayer girado o volteado presentaría un patrón de filtros de color diferente, y los mapas de calibración
  (desviación oscura, campo plano) se sitúan en la propia cuadrícula del sensor.
- **`preview`/`display`, `radiance`, `reflectance` y el renderizado `index/` están orientados** —girados
  una vez, tras todos los cálculos de calibración—, de modo que se abren tal y como se encuadró la cámara. Sus
  archivos `clamp_masks/…_clampmask.tif` se encuentran en la misma cuadrícula.
- **Cada archivo de captura registra la configuración** en XMP: `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`), `ChlorosRotation` (0 / 90 / 180 / 270, en el sentido de las agujas del reloj, tras
  los giros) y `ChlorosOrientationApplied` — `false` en los productos sin procesar y sin bayering (píxeles nativos),
  `true` en los productos orientados. `ChlorosProcessing` enumera las operaciones aplicadas a los píxeles de ese archivo
  (`none` cuando son nativos). Esto incluye cada fotograma que transmite `lattice continuous` (la
  propia configuración de la cámara, sin aplicar) y cada fotograma de `lattice multi-stream` (la identidad: sus
  fotogramas miembros son nativos, como los de una captura en matriz). Las pilas `lattice align-stream` / `align-apply`
  registran la configuración de la cámara maestra, sin aplicar: sus bandas se deforman para ajustarse al fotograma de la cámara maestra,
  pero no se giran.
- **`process` aplica la orientación registrada** al procesar un archivo sin procesar: primero el desplazamiento oscuro, el campo plano
, el demosaico y la separación de canales en coordenadas del sensor; a continuación, la deformación de alineación de la matriz; y, por último, la
  orientación en último lugar — de modo que los productos de vista previa, radiancia, reflectancia e índice salen exactamente igual que
  los de la captura en directo, y se graba `ChlorosOrientationApplied=true`. Los productos de reserva sin calibrar
  (`Sensor_Response_Images`, `Vignette_Corrected_Images`) se orientan de la misma manera, tras
  el campo plano. El producto sin «debayering» se mantiene en su formato nativo.
- **El visor de imágenes Chloros muestra un archivo sin procesar o sin «debayering» girado según su registro** (solo en pantalla —
  el archivo no se modifica), de modo que se ve igual que sus productos; la miniatura, la vista a tamaño completo, la
  lectura del cursor y el entorno de pruebas de Índice/LUT utilizan todos la misma rotación. Un archivo sin registro y
  un producto (ya orientado) se muestran tal y como están almacenados. Una exportación al entorno de pruebas registra la orientación que
  tienen sus píxeles (`ChlorosOrientationApplied=true`; la identidad para una capa sin registro).
- **Las capturas sin registro se procesan como nativas** (comportamiento sin cambios). Antes de la versión 1.3.0, una
  captura con una sola cámara en la que se había establecido un volteo o una rotación giraba por sí misma sus archivos RAW y sin debayering;
  Chloros no puede distinguir un archivo RAW ya-girado de uno nativo y no hace suposiciones, por lo que
  lo procesa tal y como está almacenado. Los mapas de calibración se encuentran en la propia cuadrícula del sensor, por lo que, para una
  captura de este tipo, utiliza una tomada con la versión 1.3.0 cuando la calibración sea relevante.
- **Las matrices no giran ningún archivo miembro por defecto** (la deformación de alineación y el XMP `Alignment*` se encuentran en
  los propios fotogramas de las cámaras, sin girar) y registran la identidad. `array-capture --orient` gira los
  productos de cada miembro, tras la transformación de alineación: según la orientación del **maestro** cuando fueron
  transformados a su sistema de referencia (para que los miembros permanezcan co-registrados), o bien según la de su propia cámara. Cada
  archivo registra la orientación del marco en el que se encuentran sus propios píxeles —un archivo sin procesar, el de su propia cámara—
  y un archivo sin procesar que contiene la transformación de alineación también registra `ChlorosAlignedFlipHorizontal` /
  `ChlorosAlignedFlipVertical` / `ChlorosAlignedRotation`: la del maestro, para un producto deformado
  al fotograma de la cámara principal. `process` gira cada producto según el que corresponde a sus píxeles:
  el alineado cuando deforma el archivo RAW (`--array-alignment`, el valor por defecto —los productos de la captura alineada en tiempo real
  ), el propio de la cámara cuando no lo hace (`--no-array-alignment` — los de la
  captura en directo no alineada). Existe la misma opción para `array-burst`, `project capture` / `burst` /
  `stream`, las recetas y el SDK.

### Capturas omitidas en matrices de filtros mixtos

Cuando se mezclan cámaras RGB y multiespectrales en una matriz, `array-capture --processing radiance` (o `reflectance`) guarda los fotogramas multiespectrales y **omite** las cámaras RGB, ya que la radiancia por píxel Bayer no tiene sentido para un sensor de banda ancha. La CLI muestra explícitamente cada archivo guardado (con su nivel de exportación), cada `.daq` escrito y cada omisión, por lo que el recuento de archivos no resulta sorprendente:

```
  Saved: …/output/reflectance/sync_…_SN213800234-FRGN.tiff [reflectance] (SN:213800234, fid:1)
  Saved: …/output/reflectance/sync_…_SN214000533-FOCN.tiff [reflectance] (SN:214000533, fid:1)
  Saved: …/output/reflectance/sync_…_SN214701288-FNGB.tiff [reflectance] (SN:214701288, fid:1)
  DAQ:   …/output/daq/daq_data_daq-e-54b5e0_….daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

Los tokens de motivo de omisión siguen el patrón `<level>-not-applicable-to-rgb-cam`. Una cámara cuyo paquete de calibración no esté cargado omite `radiance` / `reflectance` con `<level>-needs-calibration (connect the camera so its cal pack loads)`, y si no hay una lectura de reflectancia del DAQ utilizable, omite los valores `reflectance-skipped-no-dls`. La reflectancia también puede omitirse con `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, y con `dls-uncalibrated-band-<nm>` cuando la banda se encuentra en su mayor parte fuera del rango radiométricamente calibrado del sensor de luz del DAQ (~374–974 nm); entre las referencias disponibles, solo la F988, cuya ruta compatible es el flujo de trabajo del panel de reflectancia.

Utiliza `--processing debayered` (o `display`) para incluir todas las cámaras independientemente del tipo de filtro, o el valor predeterminado `all` para obtener todos los niveles aplicables por cámara de una sola vez.

### Una lectura DAQ alejada de la exposición

Cada `.daq` que guarda una captura es la lectura DAQ correspondiente a esa exposición, y el `.match.json` que aparece junto a ella registra la distancia entre ambas (`gap_ms`) y si está dentro de la tolerancia (`in_tolerance`, 500 ms a menos que se haya configurado la captura `--daq-tolerance-ms`). Una lectura fuera de ese margen se sigue utilizando, por lo que no se pierde ningún fotograma, y se etiqueta: el archivo XMP de reflectancia contiene `DlsInTolerance=False` y `DlsMatchGapS`. `lattice capture`, `multi-capture` y `array-capture` también lo imprimen bajo el `.daq` correspondiente, con elcuando se explica:

**Cómo se empareja una lectura con una exposición.** Una captura espera a la primera lectura del DAQ tras la exposición, para poder combinar las lecturas a ambos lados de la misma (`method` `interpolated`). Un DAQ-E envía una lectura cada 0,4 s a la luz del día y cada 1-2 s en interiores, por lo que la espera suma aproximadamente un intervalo y medio de esos tiempos al comando, no a la exposición. No se espera a un DAQ que haya dejado de enviar datos. Una lectura del DAQ-E es la media de la luz durante el intervalo transcurrido desde su lectura anterior, por lo que se sitúa en el centro de dicho intervalo: `gap_ms` se mide desde la exposición hasta ese punto medio, y `bracket_stamps_ns` proporciona las marcas de tiempo propias de las lecturas. `array-capture --continuous` espera sin ralentizar la captura: sus archivos sin procesar se escriben inmediatamente y sus `.daq` una vez que llega la lectura. Con `--interval`, cada pasada espera como si fuera una única captura.

```
  DAQ:   …/output/daq/daq_data_daq-e-54b774_….daq
         DAQ reading 114.9 s from the exposure (tolerance 500 ms): reflectance uses it, labelled DlsInTolerance=False
         DAQ stream fault: no frame accepted from daq-e-54b774.local in 25 s, after 1932 good frame(s). …
```

Cuando no se ha escrito ningún archivo de reflectancia a partir de esa lectura, la primera línea termina en `saved anyway; its .match.json says in_tolerance: false`.

La captura sigue terminando con `0`: se han escrito todos los archivos y se indica su contenido. Un DAQ cuyo flujo se ha estancado sigue suministrando su última lectura, por lo que conviene comprobarlo con `chloros-cli daq pool-list` antes de confiar en la reflectancia de esa captura. El backend vuelve a conectar un DAQ-E agrupado que ha dejado de enviar datos (véase [Un DAQ-E que deja de enviar datos](#a-daq-e-that-stops-delivering)). Una lectura fuera de tolerancia solo muestra su línea `DAQ:`, a menos que se trate de una lectura conservada (véase más abajo).

**Una lectura conservada tras la desaparición de su DAQ.** Cuando ningún DAQ puede proporcionar una lectura para una exposición (el DAQ vinculado no está disponible o ningún DAQ del grupo dispone de una, p. ej., tras `daq pool-disconnect`), la captura reutiliza la última lectura que tenía Chloros, y su `.match.json` describe dicha lectura en relación con esta exposición: `method`, `retained`, `gap_ms` de esta exposición a las muestras de las que se obtuvo la lectura, `in_tolerance` de ese intervalo, y `retained_from_epoch_ns`, la exposición con la que se emparejó inicialmente la lectura. Por lo tanto, una captura realizada mucho después de que el DAQ dejara de funcionar imprime las líneas anteriores y, debajo de ellas aparece una más, que se imprime incluso cuando la lectura aún se encuentra dentro de los límites de tolerancia:

```
         This is the DAQ's last reading: the DAQ is no longer reporting, so each capture reuses it until a new one arrives
```

Su `.daq` sigue indicando el DAQ que tomó la lectura, junto con la calibración y la tapa bajo las que se tomó dicha lectura, de modo que, al volver a importarla, se sabe qué DAQ, calibración y tapa la generaron.

### Archivos que no se pueden escribir

Una cámara cuyo archivo no se pueda escribir —por ejemplo, una unidad llena o un error de permisos— no impide que se guarden los archivos de las demás cámaras. Cada comando de captura (`lattice capture`, `multi-capture`, `array-capture`; `project capture`, `burst`, `stream`; y los de una receta: `capture`, `burst` y `stream`) imprimen cada archivo que ha fallado en stderr, y luego **sale de `1`** aunque el resto se haya guardado:

```
  Failed: SN:214701292 [reflectance] OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

Se incluyen todos los archivos que debe una captura, incluido el único`lattice capture --levels debayered` con un solo fotograma sin debayering. Una lectura `.daq` cuyo archivo sidecar de procedencia `.match.json` no se pudo escribir aparece como `[match]` (el propio `.daq` sí se guardó).

En una unidad llena, una captura repetida (`array-capture --continuous` / `--interval`, `project burst` / `stream`, una receta `burst` / `stream`) **se detiene en esa pasada** en lugar de no escribir nada en la misma unidad durante el resto de la ejecución. Cualquier otro fallo se notifica en cada pasada en la que se produce, y la ejecución continúa hasta el final, tras lo cual se sale de `1`. `array-capture --continuous` (la captura continua del backend) no muestra ningún archivo, incluidos los que han fallado: los cuenta en sus líneas de progreso y resumen (`N failed`, `N file(s) failed to write.`) y sale de `1` — véase [Modo continuo](#continuous-mode---continuous).

Los grabadores — `array-burst` y `array-record` — informan de sus propios fallos. `array-burst` se detiene por sí mismo cuando la unidad se llena, o justo antes de que esto ocurra. Ambos continúan funcionando a pesar de cualquier otro error de escritura. Cuando cualquiera de los dos notifica un error, este se muestra después de los contadores, y el comando **sale de `1`**:

```
  Recorder error: disk-full: OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

`Disk low:` en lugar de `Disk full:` significa que la ráfaga se detuvo antes de llenar la unidad. Se conservan las tramas grabadas antes de la parada. `array-burst --build` omite la compilación cuando se agota el espacio, ya que los vídeos se guardarían en la misma unidad. Libera espacio primero y, a continuación, ejecuta `array-build-video --burst-dir …`. Tras cualquier otro error, la compilación sigue ejecutándose.

Un `array-record` que no haya escrito ningún fotograma también supone un fallo en la grabación (`Recorder error: no-frames: the combined view published no frame to record …`, salir de `1`); véase [`array-record`](#array-record--combined-index-videogif-monitoring-grade).

---

## Modos de captura, grabadoras y reprocesamiento fuera de línea

Todos ellos operan sobre una **matriz persistente** (ejecuta primero `array-connect`). Reflejan el panel de captura de la interfaz gráfica de usuario.

### Modos de `array-capture`

`array-capture` es un único comando con cuatro modos de obturación, además de un conjunto de opciones de exportación:

| Modo | Indicador | Comportamiento |
| --- | --- | --- |
| **Único** *(por defecto)* | (ninguno) | Un grupo de capturas sincronizadas y, a continuación, salida. |
| **Continuo** | `--continuous` | Capturas consecutivas hasta que `Ctrl+C`, `--count N` capture, `--duration S` o hasta que se llene la unidad. Ejecuta la captura continua propia del backend (el «Modo continuo» de la interfaz gráfica de usuario); véase [Modo continuo](#continuous-mode---continuous) más abajo. |
| **Intervalo** | `--interval S` | Una pasada cada `S` segundos (medidos desde el inicio de cada pasada), con los mismos límites. |
| **Más rápido** | `--fastest` | Solo datos sin procesar+ la lectura DAQ asignada + el compuesto de índice combinado; omite los cálculos de radiancia/reflectancia/visualización para que el fotograma se genere rápidamente. Implica `--processing raw --force-daq`. Reprocesar posteriormente el `.daq` guardado en productos calibrados. |

Opciones de exportación (se pueden combinar con cualquier modo; todas comparten el punto final de la GUI/el SDK):

| Indicador | Efecto |
| --- | --- |
| `--array-id ID` | La matriz desde la que se va a capturar (omitir cuando solo haya una conectada). |
| `-o, --output DIR` | Carpeta de salida (por defecto `output`, dentro de la carpeta desde la que se ejecuta el comando). |
| `--format {tiff,tif,png,jpg,jpeg}` | Formato de archivo (por defecto, `tiff`). PNG/JPEG: solo admiten `display` y el renderizado del índice; consulta [Niveles de exportación de captura](#capture-export-levels-the-all-default). |
| `--processing LEVEL` | Nivel de exportación único (`raw`, `debayered`, `radiance`, `reflectance`, `display`), o `all` (por defecto). |
| `--levels L1,L2,…` | Subconjunto explícito de tipos de exportación (p. ej., `raw,radiance,reflectance`); **anula `--processing`**. |
| `--aligned` / `--no-aligned` | **Por defecto: alineado, tal y como lo captura la aplicación Chloros**(su configuración de captura**Alineada** por matriz, activada a menos que la desactives). La exportación no sin procesar de cada miembro (`debayered`, `display`, `radiance`, `reflectance`, el renderizado del índice) se adapta al fotograma de la cámara principal y se recorta a la zona de solapamiento común de los miembros, de modo que los archivos están alineados por píxeles y presentan el mismo tamaño, recorte y etiquetas `Alignment*` que los de la aplicación —más pequeños que el sensor (o la zona de interés) en función de los desplazamientos de los miembros; en el caso de una zona de interés, la transformación se reajusta a la ventana. `XMP-mapir:AlignmentJSON` indica `"applied": true` y proporciona el `crop_roi`. El archivo sin procesar se mantiene sin deformar en su tamaño nativo, pero lleva la transformación en los metadatos (`"applied": false`). `--no-aligned` guarda cada archivo en su tamaño nativo, sin deformar. `--aligned` es la configuración predeterminada. Independientemente de si se aplica o no la distorsión, cada archivo miembro registra la transformación `Alignment*` de su cámara una vez que la matriz tiene un perfil de alineación — salvo una cámara que el perfil no alinee (omitida o forzada a la identidad por un fallo en la resolución) y la cámara de referencia cuando no haya ninguna otra cámara alineada: estas no llevan etiquetas `Alignment*`. Si la matriz carece de perfil, la captura se guarda sin alinear y la CLI muestra el motivo en una línea `Alignment:`, tal y como hace en cualquier otro caso en el que no se haya podido aplicar la deformación (miembros con resoluciones diferentes, una matriz no sincronizada por cable, un fotograma al que no se pueda asignar el perfil). La alineación automática de la matriz crea el perfil una vez que se estabiliza la exposición; `align-calibrate` crea uno bajo demanda. |
| `--orient` | Genera los **productos**de cada miembro — `display`, `radiance`, `reflectance`, el renderizado `index/` y el `composite/` — según la orientación de la cámara, tras la deformación de alineación: un producto deformado al marco del maestro (por defecto) según la**orientación del**marco maestro, de modo que los elementos permanezcan co-registrados; uno no alineado (`--no-aligned`, o una captura a la que no se pudo aplicar la transformación) según la orientación de su propia cámara.**Por defecto: desactivado — cada archivo de elementos permanece en su formato nativo**, como siempre ocurre con las capturas de matriz. `raw` y `debayered` se mantienen en formato nativo en cualquier caso. Cada archivo registra la orientación que tienen sus productos (véase [Orientación de la cámara](#cómo-es-una-carpeta-de-capturas)); el archivo XMP de `Alignment*` conserva la transformación del fotograma nativo, aplicada antes de la orientación, de modo que `process` (deformación y, a continuación, orientación) reproduce el producto en tiempo real. |
| `--index` / `--no-index` | Guardar / omitir la superposición del índice de vegetación por cámara, cuando esté configurada. Por defecto: renderizarla. |
| `--no-daq` | Omite el archivo «sidecar» `.daq` cuando ningún nivel seleccionado utilice el DAQ (una captura solo en formato sin procesar). Por defecto, la lectura se guarda, de modo que los fotogramas pueden volver a procesarse en reflectancia/índice fuera de línea. `--force-daq` sigue siendo válido (valor por defecto). |
| `--daq-tolerance-ms MS` | La tolerancia de coincidencia del DAQ para estas capturas (por defecto, 500 ms). |
| `--smart` | Esperar a que el AE se estabilice en todas las cámaras antes de disparar (véase [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | Compresión de píxeles TIFF. `none` (por defecto desde la versión 1.3.0; la versión 1.2.0 escribía `deflate`) = sin comprimir — los píxeles del sensor tal y como se leen, ~6,3 MB por fotograma a resolución completa y la velocidad sostenida más rápida; `deflate` = zlib L1 sin pérdidas + predictor horizontal, ~4,1 MB por fotograma, más lento. Ambos son sin pérdidas y se leen de forma idéntica al importarlos. |

> **TIFF de escritura única + el modelo de velocidad sostenida.**Cada archivo se escribe**una sola vez**: píxeles, el paquete XMP, el IFD0 (marca/modelo/número de serie de la cámara) y las etiquetas EXIF `ExposureTime` / `SerialNumber` en una sola pasada, sin que nada lo reescriba posteriormente. `--continuous`X captura grupos sincronizados consecutivos —cada captura toma el grupo siguiente al anterior, incluido uno publicado mientras esa captura aún se estaba guardando—, por lo que funciona a la velocidad de la matriz. DEFLATE retiene el GIL de Python, por lo que las escrituras comprimidas**no**se paralelizan entre los hilos de escritura de cada cámara; la captura sostenida a resolución completa a la velocidad del sensor requiere escrituras sin comprimir (el valor por defecto)**y** un disco de clase NVMe (~600 MB/s sostenidos para una matriz de 9 cámaras a ~10 fps). El mismo control se muestra como `compression` en `POST /api/camera/array/capture`.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (aligned is the default; drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --no-index -o out/

# Each member at its native size, unwarped
chloros-cli lattice array-capture --processing reflectance --no-aligned -o native/

# Continuous raw capture for 60 s, as fast as the array publishes
chloros-cli lattice array-capture --continuous --duration 60 --fastest -o flightline/
```

#### Modo continuo (`--continuous`)

`array-capture --continuous` (sin `--interval` ni `--smart`) transfiere la ejecución a la **captura continua propia del backend** —la sesión en la que se ejecuta el modo continuo de la interfaz gráfica de usuario—. El backend guarda cada grupo sincronizado por sí mismo, con dos capturas en curso y sin solicitudes desde la CLI entre grupos, por lo que mantiene el ritmo de una matriz rápida en la que una solicitud por captura se queda rezagada (se han medido entre 73 y 77 capturas/s en una matriz de 9 cámaras con un ROI de 512 píxeles). Cada opción de exportación anterior se aplica a cada captura.

- **Límites:** `--count N` se detiene tras N capturas (una captura = un grupo sincronizado: los archivos de cada miembro); `--duration S` tras S segundos. Sin ninguno de los dos, se ejecuta hasta `Ctrl+C`, hasta que se llene la unidad, se produzca un error o la matriz deje de estar disponible. `Ctrl+C` detiene la sesión del backend y y a continuación muestra el resumen.
- **Progreso** cada 2 s, con `, N failed` y `, N groups skipped` añadidos cuando el valor es distinto de cero (un grupo omitido es aquel que la matriz publicó y del que no se realizó ninguna captura):

  ```
     12.0s  880 captures (73.3/s), 7920 files
  ```

- **No se muestra un listado por archivo.** La ejecución finaliza con una línea de resumen —capturas, velocidad, archivos y la carpeta de salida— seguida de `N file(s) failed to write.` / `N group(s) skipped.` cuando el valor es distinto de cero:

  ```
  Done — 880 captures (73.3/s), 7920 files in /home/user/flightline.
  ```

- **Se detiene** cuando la unidad está llena con `Continuous capture stopped: the drive is full.`, y ante un error o si la matriz desaparece con `Continuous capture stopped: <reason>.` (ambos en stderr).
- **Estado de salida:** `0` cuando se han guardado las capturas y se han escrito todos los archivos, incluido `Ctrl+C`; `1` si falla la escritura de cualquier archivo, una unidad llena, un error, la desaparición de la matriz o la ausencia total de captura.
- **Microsoft Defender:** cuando Defender utiliza un núcleo de CPU o más para analizar los archivos que escribe la captura, la ejecución muestra un mensaje una vez, en stderr: `Note: Microsoft Defender is using N.N CPU cores scanning the capture files, CPU the capture needs. Re-run the Chloros installer with "Exclude the Chloros backend from Microsoft Defender scanning" checked.`. Consulta la sección [Configuración del sistema](#windows-exe).
- Una captura continua por matriz a la vez; se rechaza una segunda con el código `a continuous capture is already running for this array`. A través de HTTP, la sesión es `POST /api/camera/array/continuous/start` (el cuerpo `/api/camera/array/capture` más `max_captures` / `duration_s`), `GET /api/camera/array/continuous/status?array_id=ID` y `POST /api/camera/array/continuous/stop`.

**Intervalo**(`--interval S`, con o sin `--continuous`) y**`--continuous --smart`** mantienen el bucle por solicitud: una solicitud de captura por pasada —en cada momento programado para Intervalo, una vez que el AE se estabiliza para `--smart` —, con los archivos de cada pasada enumerados como en el modo «Single», una línea `Done — N capture pass(es).` al final y `--count N` contando las pasadas.

### `array-record` — vídeo/GIF con índice combinado (calidad de monitorización)

Graba la **vista en directo del índice combinado** de la matriz en un `.avi` (y, opcionalmente, en un `.gif`). Con la vista combinada de la interfaz gráfica abierta, graba dicha vista tal y como se muestra. Si no hay ninguna abierta, el backend genera la vista por sí mismo: el índice que se pase con `--expr` o, por defecto, el NDVI de la matriz, una vez que esta esté alineada. El NDVI por defecto utiliza las bandas roja y NIR más cercanas a 660 nm y 850 nm, al igual que la vista en directo. Una vista de la interfaz gráfica de usuario abierta durante la grabación se graba mientras permanece abierta. Comprueba el progreso cada 2 s y se detiene en `--duration`, `Ctrl+C`, o cuando la grabadora finaliza automáticamente.

Una grabación que no escribe ningún fotograma falla: la grabadora informa de `no-frames: …` (es posible que la matriz aún se esté alineando o ajustando su exposición) y el comando **sale de `1`**.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF (no GUI needed)
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# Record a chosen index instead of the default NDVI
chloros-cli lattice array-record --duration 30 \
  --expr "(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)" -o monitoring/
```

| Indicador | Valor predeterminado | Descripción |
| --- | --- | --- |
| `--array-id ID` | solo matriz | Matriz de destino (omitir si solo hay una conectada). |
| `-o, --output DIR` | `output` | Directorio de salida. Una ruta relativa se encuentra en la carpeta desde la que ejecutas el comando. |
| `--fps F` | `10` | Frecuencia de fotogramas de grabación. |
| `--duration S` | hasta Ctrl+C | Parada automática tras `S` segundos. |
| `--gif` | desactivado | Escribir también un GIF animado. |
| `--gif-only` | desactivado | Guardar solo un GIF (sin `.avi`). |
| `--expr EXPR` | el NDVI de la matriz | Expresión de índice que se registrará cuando no haya ninguna vista de la interfaz gráfica abierta, en los nombres de las bandas de la matriz (p. ej., `'(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)'`; véase [nombres de bandas](#index--vegetation-maths)). Un nombre de color corto (`NIR`, `Red`) solo funciona cuando una banda de la matriz lo tiene. En cambio, si hay una vista de la interfaz gráfica de usuario abierta, se registra tal y como se muestra. |

### `array-burst` — ráfaga raw-Bayer de alta velocidad de fotogramas (calidad de análisis)

Lee directamente el búfer del grupo sincronizado del bucle de captura — **sin necesidad de cadena de calibración, exiftool ni vista en directo** —, por lo que funciona a la velocidad máxima de captura de la cámara. Escribe fotogramas RAW + un manifiesto por fotograma + un `.daq` por cada lectura DLS distinta bajo `<output>/bursts/<base>/`. Vuelve a procesar sin conexión (siguiente comando) o pasa `--build` para hacerlo inmediatamente al detenerse.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| Indicador | Predeterminado | Descripción |
| --- | --- | --- |
| `--array-id ID` | solo matriz | Matriz de destino. |
| `-o, --output DIR` | `output` | Directorio de salida (los datos se guardan en `<DIR>/bursts/<base>/`). |
| `--duration S` | hasta Ctrl+C | Parada automática tras `S` segundos. |
| `--max-frames N` | sin límite | Parada automáticaparada automática tras `N` fotogramas sin procesar. |
| `--orient` | desactivado | Los productos que genera una compilación a partir de esta ráfaga se giran según la orientación de la cámara: los productos de cada cámara se giran según la orientación de su propia cámara, el índice combinado por la de la cámara maestra—, tal y como hace `array-capture --orient`. Los fotogramas sin procesar se mantienen en su formato nativo; el `manifest.json` de la ráfaga registra la orientación y la opción de cada miembro. |
| `--build` | desactivado | Tras detenerse, volver a procesar inmediatamente la ráfaga (igual que `array-build-video`). |
| `--products …` | `combined:index` | Con `--build`: qué vídeo(s) generar (véase más abajo). |
| `--fps F` | `10` | Con `--build`: fps del vídeo de salida. |
| `--save-tiffs` | desactivado | Con `--build`: guarda también archivos TIFF calibrados por fotograma (cada uno con su [máscara de recorte de desmezclado](#unmix-clamp-mask-which-pixels-en-la-banda-en la que se ha aplicado el recorte) cuando se recorta un píxel; el resultado de la compilación los enumera como `clamp_masks`). |
| `--gif` | desactivado | Con `--build`: también se escriben archivos GIF animados. |

### `array-build-video` — reprocesa fuera de línea una ráfaga guardada

Sincroniza temporalmente cada fotograma sin procesar con la lectura `.daq` guardada más cercana y lo pasa por la **misma cadena de radiancia / reflectancia / índice que el proceso de importación**, generando uno o más vídeos.

`--products` es una lista separada por comas de elementos `kind:level`, donde `kind` ∈ `per_cam` | `combined` y `level` ∈ `radiance` | `reflectance` | `index`. Un `level` sin especificar (sin `kind:`) toma por defecto el valor de `per_cam`. El valor por defecto es `combined:index`.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| Indicador | Valor por defecto | Descripción |
| --- | --- | --- |
| `--burst-dir DIR` | (obligatorio) | Ruta a la carpeta de ráfagas (`…/bursts/<base>/`). |
| `--products …` | `combined:index` | Lista `kind:level`, como se ha indicado anteriormente. |
| `--fps F` | `10` | Fotogramas por segundo (fps) del vídeo de salida. |
| `--save-tiffs` | desactivado | Guardar también archivos TIFF calibrados por fotograma junto con los vídeos. |
| `--gif` | desactivado | También escribe GIF animados. |

**Un vídeo no contiene máscara de recorte** (una secuencia de imágenes, 8 bits). En su lugar, el resultado de la compilación incluye `clamp_fractions`: por cada producto de vídeo, la proporción de píxeles de cada fotograma que el proceso de desmezclado ha limitado en la banda que contiene su canal R / G / B — el `Chloros:Band{R,G,B}UnmixClampFraction` con el que se marcaría el producto de radiancia/reflectancia de ese fotograma (para el índice combinado, de cada cámara participante). La ejecución imprime el peor fotograma por producto en una línea `Clamp:`. `--save-tiffs` añade las máscaras por píxel para los archivos TIFF, en `clamp_masks/<level>/`, junto a la carpeta `<level>/` de los TIFF.**Una compilación interrumpida no deja ningún vídeo incompleto.** Cada vídeo se guarda como `<name>.partial.avi` y solo adquiere su nombre definitivo cuando está terminado, por lo que un vídeo anterior con ese nombre permanece hasta que su sustituto esté completo. Vuelve a ejecutar la compilación para sustituir un `.partial.avi` que se haya quedado atrás.

La ejecución muestra cada vídeo que ha grabado, seguido de cualquier archivo `Skipped:` (una cámara o nivel solicitado —o el índice combinado— que no ha generado ningún vídeo, indicando el motivo —por ejemplo, falta de paquete de calibración, ausencia de lectura DLS para la reflectancia, o ausencia de un par de bandas roja/NIR para el índice combinado) y cualquier `Note:`. Una cámara RGB a la que se le ha solicitado radiancia o reflectancia obtiene en su lugar su vídeo de visualización, con balance de blancos cuando el paquete de calibración de la cámara está disponible; una cámara que se procesa sin él se indica en un `Note:`. **Devuelve un valor distinto de cero** cuando se omite un producto solicitado o no se ha generado según lo solicitado (una expresión de índice configurada no se evalúa fuera de línea); los vídeos que se han generado se siguen escribiendo y enumerando.

> **Elige el grabador adecuado.** `array-record` es de *nivel de monitorización*: graba la composición en directo renderizada: la vista combinada de la interfaz gráfica de usuario o, si no hay ninguna interfaz abierta, la vista NDVI (o `--expr`). `array-burst` → `array-build-video` es de *nivel de análisis* — guarda los datos brutos del sensor a velocidad máxima y reconstruye posteriormente vídeos calibrados de radiancia, reflectancia e índice, sin necesidad de una vista en directo.

### Cámaras monocromáticas (M3M) de banda única

La gama **M3M**es la versión monocromática de la**M3C**de Bayer: un filtro de interferencia de banda estrecha por cámara (`M3M-<lens>-F<wavelength>`, p. ej., `M3M-L87-F685`), por lo que el sensor ofrece una**única banda en escala de grises** sin mosaico Bayer. No hay nada que demosaicar, ni diafonía entre canales que separar, ni balance de blancos que ajustar: todo el proceso de color de la visualización en RGB simplemente no se aplica.

Lo que esto significa en la CLI:

- **`lattice white-balance`, `lattice color-profile`, `lattice color`**detectan una cámara monocromática y**pasan de largo con un mensaje de una línea** en lugar de aplicar ajustes sin sentido. Siguen funcionando con normalidad con una cámara M3C de RGB/Bayer en la misma sesión.
- **Un formato de píxeles Bayer implica su profundidad.** Un M3M solo ofrece formatos mono, por lo que un formato Bayer de `--pixel-format` o `--preset` establece el formato mono de la misma profundidad: `BayerRG8` y `--preset high_speed` dan como resultado `Mono8`; `BayerRG12`, `high_quality` y `triggered` dan como resultado `Mono12`; un nombre empaquetado permanece empaquetado (`BayerRG12p` da como resultado `Mono12p`). Los archivos y el nombre `lattice format` se denominan formato Mono. Una matriz que combina cámaras M3C y M3M adopta un único formato de la mismo modo: Bayer en cada M3C, su profundidad Mono en cada M3M. Un nombre es su profundidad de bits: lo mismo se aplica a la inversa, por lo que una M3C nunca se asigna a un formato Mono (`--pixel-format Mono12` es `BayerRG12` en una M3C, independientemente de cuál sea la cámara maestra de la matriz), y `lattice format --set` se asigna de la misma manera e imprime el formato que se le ha establecido.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** siguen funcionando: la radiancia y la reflectancia son mapas radiométricos *por banda* y están perfectamente bien definidos para una sola banda. Los fotogramas monocromáticos contienen una matriz de respuesta del sensor **identitaria** (sin desmezclado 3×3), por lo que el plano pasa por el cálculo de calibración sin modificaciones.
- **Una sola cámara monocromática no puede generar un índice de vegetación.**El NDVI, el NDRE y demás índices requieren al menos dos bandas (por ejemplo, Red + NIR). Para obtener un índice a partir de hardware monocromático, hay que apuntar**varias** cámaras M3M a diferentes longitudes de onda, alinearlas en una pila multibanda y calcular el índice a partir de *esa* pila:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Mono_660 --channel nir=Mono_850 \
  --save-multiband -o output/
```

`index --live` abre las cámaras por sí mismo (una [sesión transitoria](#multi-camera-transient-sessions)), así que primero no conectes nada: `--serials`; de lo contrario, las cámaras habilitadas en el perfil (véase [Índice / Cálculos de vegetación](#index--vegetation-maths)). Mientras Chloros mantenga controlada cualquier cámara (la interfaz gráfica de usuario, `cam-connect`, `array-connect`) se rechaza con un código HTTP 409 `CAMERA_HELD` y se cierra `1`; libéralas con `chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all`.

Los símbolos `--channel` deben coincidir **exactamente** con los nombres de los canales del preajuste (se distingue entre mayúsculas y minúsculas; los de NDVI están en minúsculas `red`, `nir` — véase `--list-presets`), y el nombre de la banda debe corresponder a una banda de la pila alineada (el modo sin conexión también acepta índices de banda basados en 0, p. ej., `--channel red=0 --channel nir=1`).

El discriminador en toda la pila es el token `M3M` de la cadena del modelo (nunca aparece en una cadena `M3C`), que se muestra en la interfaz gráfica de usuario (GUI) y el SDK como `is_mono`.

---

## Configuración y ajuste de la tarjeta de red del host (matrices LATTICE)

Las cámaras LATTICE transmiten GVSP a través del adaptador Ethernet del host, por lo que, en el caso de matrices multicámara, el **controlador**y el**tamaño del anillo de recepción** son tan importantes como la velocidad del enlace. Una configuración incorrecta se manifiesta como un error `FRAMES WILL DROP` / `Reduce ROI to enable` en el panel de configuración de la matriz (y en `lattice network-analysis` / el SDK `analyze_array_network()`), incluso cuando las propias cámaras funcionan correctamente.

### Adaptadores USB 10 GbE — Realtek RTL8157 («Realtek USB 10GbE Family Controller»)

Los ajustes que se indican a continuación corresponden a las propiedades del controlador de Windows. En Linux, los adaptadores USB también tienen un límite (400 MB/s en x86_64, 200 MB/s en NVIDIA Jetson) (véase [Fps de matriz y modelo de ráfaga](#array-fps--burst-model)), y los búferes de recepción de sockets del kernel son importantes para todos los adaptadores; véase [Hosts de Linux](#linux-hosts--socket-receive-buffers).

| Elemento | Valor requerido | Por qué es importante |
| --- | --- | --- |
| **Versión del controlador**|**≥ v10.67 (enero de 2026)**, INF `rtump64x64sta.inf` | El controlador heredado de**2016**(v10.65, `rtump64x64.inf`) gestiona incorrectamente el apagado y produce errores de sistema con**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**al apagar, reiniciar o poner el equipo en suspensión. La transición se cuelga (tiempo de espera de unos 5 minutos), el usuario apaga el equipo a la fuerza y los apagados incorrectos repetidos**corrompen el repositorio WMI**(PowerShell y otras herramientas empiezan a fallar con el error `Invalid class`) y**bloquean la pila USB** en el siguiente arranque (la tarjeta de red no se activa; las unidades USB dejan de enumerarse). Actualiza desde realtek.com (o el proveedor del dongle) antes de confiar en reinicios limpios. |
| **Búferes de recepción**— palabra clave `ReceiveBufferLen` |**256**(máximo del controlador) | El anillo de recepción (RX) de la tarjeta de red. El valor predeterminado del controlador,**32**, deja solo unos 0,26 MB de anillo utilizable —demasiado pequeño para una ráfaga multicámara—, por lo que el panel de la matriz muestra el código `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` y bloquea la conexión. Con**256**el anillo es grande (**~13,5 MB medidos en el host de 10 GbE del laboratorio**), lo que proporciona al canal de recepción un margen real para ráfagas GVSP multicámara. (Que una configuración determinada se *conecte* realmente viene determinado por dos comprobaciones: la comprobación de **admisión con consideración del drenaje**y la de**sobresuscripción agregada** —, no una simple comparación entre la ráfaga y el anillo; véase [Modelo de fps y ráfagas de la matriz](#array-fps--burst-model).) |
| **URB de recepción**— palabra clave `PendingReceives` |**64** (máx.) | Bloques de solicitud USB en tránsito; aumentarlos junto con los búferes de recepción para absorber ráfagas. |
| **Trama jumbo** — palabra clave `*JumboPacket` | **9014** | Necesario para paquetes GVSP de 9000 bytes (6 veces menos paquetes por trama que con 1500). |

> ⚠️ **Una actualización del controlador de la tarjeta de red REINICIA estas propiedades avanzadas a sus valores predeterminados.**Tras actualizar o sustituir el controlador del adaptador,**vuelve a aplicar** `ReceiveBufferLen=256` y `PendingReceives=64`, o el panel de la matriz volverá a bloquearse aunque «no haya cambiado nada en el hardware». Esta es la causa principal de que un equipo que antes funcionaba deje de conectarse de repente.

Aplícalas desde un PowerShell **con privilegios elevados** (sustituye el nombre de tu adaptador, p. ej., `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` es válido para adaptadores USB de 10 GbE.** Ahora detecta el tipo de adaptador y ajusta la palabra clave correcta del anillo de recepción: `*ReceiveBuffers`→2048 para tarjetas de red PCIe (Intel I219, etc.), o `ReceiveBufferLen`→256 + `PendingReceives`→64 para el controlador **USB** 10 GbE de Realtek (que no expone `*ReceiveBuffers`). Los valores objetivo se limitan al máximo indicado por cada controlador (`NumericParameterMaxValue`), por lo que nunca se escribe un valor fuera de rango. Ejecútalo desde un terminal **con privilegios elevados**; al igual que cualquier ajuste basado en el Registro, el cambio surtirá efecto tras reiniciar el adaptador o reiniciar el sistema. Los comandos manuales `Set-NetAdapterAdvancedProperty` mencionados anteriormente siguen siendo una buena alternativa, ya que se aplican en tiempo real (vuelven a vincular el adaptador) sin necesidad de reiniciar.

### Hosts Linux
: búferes de recepción de sockets

En Linux
, incluido NVIDIA Jetson, los búferes de recepción de sockets del núcleo son tan importantes como el anillo de recepción de la tarjeta de red. Con la configuración predeterminada del núcleo (`net.core.rmem_max` / `net.core.rmem_default` = 212 992 bytes), una ráfaga de paquetes GVSP de una cámara desborda el socket de flujo antes de que Chloros
lo lea. Los paquetes se descartan en el núcleo, mientras que la propia tarjeta de red no informa de ninguna pérdida, y una matriz pierde grupos de fotogramas y reduce su velocidad de fotogramas. En una prueba realizada con una matriz de 4 cámaras conectadas a un adaptador USB de 10 GbE, al aumentar ambos valores a 32 MB, la matriz pasó de 4,96 a 7,52 fps tras la conexión, y la captura continua en formato sin procesar pasó de 4,7 a 7,6 capturas/s, sin que se perdieran paquetes en el núcleo.

- **El archivo .deb los configura.** Al instalar el paquete, se escribe `/etc/sysctl.d/60-chloros-gige.conf` (ambos en `33554432`) y se aplica de inmediato. Si ya hay un valor establecido más alto, no se modifica.
- **`lattice network` los comprueba.** Cuando cualquiera de los dos valores es inferior, muestra `socket receive buffers too small for GigE Vision (…)` en *Problemas detectados*, y el registro del backend incluye la misma advertencia al conectarse la primera cámara o matriz.
- **Para aumentarlos manualmente** (hasta el siguiente reinicio; el archivo del paquete los aplica en cada arranque):

  ```bash
  sudo sysctl -w net.core.rmem_max=33554432 net.core.rmem_default=33554432
  ```

  `lattice network --fix` también los aumenta y los mantiene, pero solo cuando el backend se ejecuta como root. Un backend que se ejecute como tu usuario (uno iniciado con `chloros-backend`, o [`chloros-backend@<user>`](#ejecutar-el-backend-como-un-servicio-sin-interfaz-gráfica)) no puede modificar la configuración del kernel y, en su lugar, muestra la línea `sudo sysctl`.

**El ajuste automático de adaptadores solo afecta a los adaptadores conectados.** La primera vez que se conecta una cámara, un backend que se ejecuta como root ajusta los adaptadores con cable cuyo anillo de recepción es pequeño. Ahora solo ajusta los adaptadores con conexión, por lo que un puerto integrado desconectado mantiene su MTU y no modifica los búferes de recepción. `lattice network` ya no indica que el MTU de un adaptador desconectadocomo un problema. `lattice network`, `--fix`, `lattice probe` y el ajuste de adaptadores solo tienen en cuenta los puertos Ethernet por cable, no el bus CAN de un Jetson (`can0`) ni los enlaces en modo dispositivo USB (`usb0`, `usb1`) que ofrece a un PC anfitrión; un adaptador USB Ethernet para cámaras sigue contando.

### Conceptos básicos de red (todos los enlaces LATTICE)

- **Direccionamiento:** `169.254.0.0/16` local de enlace (GigE Vision LLA). El host adopta una dirección estática `169.254.x.x/16`; las cámaras y el DAQ-E se autoasignan en el mismo rango. No se requiere DHCP ni puerta de enlace.
- **Tamaño de paquete:**es preferible el tamaño jumbo (9000), pero deja que la detección automática lo determine; esta función ignora el límite ICMP de 1500 bytes de la cámara mediante una sonda GVSP, por lo que se establece en jumbo siempre que el cable lo admita realmente.**Cada cámara se mide una vez por sesión de backend y de nuevo cuando cambia su enlace (otro adaptador, otra velocidad de enlace o una conexión que se ha interrumpido, como ocurre al cambiar de conmutador); las reconexiones reutilizan el resultado. Tras un cambio detrás de un conmutador que se ha mantenido activo, reinicia el backend para volver a realizar la medición.**La primera conexión puede tardar ocasionalmente unos minutos en esta comprobación, y el registro lo indica cuando ocurre. Utiliza «pin» con `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` solo cuando sepas más que la sonda, y da preferencia a la configuración por comando frente a la permanente: un «pin» omite la sonda, por lo que, si la ruta no puede transportar realmente 9000,**todas** las capturas agotan el tiempo de espera con `SC_ERR_TIMEOUT -1011` (véase [Variables de entorno](#environment-variables)).
- **El anillo RX se escala con `ReceiveBufferLen`:**con el valor predeterminado `32`, el anillo utilizable es de ~0,26 MB (demasiado pequeño para cualquier ráfaga multicámara); con el valor máximo `256` es grande (~13,5 MB medidos en el host de 10 GbE del laboratorio), lo que proporciona un margen real. La conexión de una configuración viene determinada entonces por la comprobación de admisión sensible al consumo**y** la comprobación de sobresuscripción agregada que se describe a continuación, y no por una simple comparación entre la ráfaga bruta y el anillo.

### fps del Array y modelo de ráfagas

Cómo interpretar el panel de configuración del Array (y `lattice analyze-array` / `analyze_array_network` del SDK
):

- **El ráfaga se suma por cámara según el formato de píxeles real de cada una.**Las cámaras mono**M3M**transmiten en formato**Mono12 (2 B/px)**, o con `--pixel-format`, el formato mono de su profundidad (`BayerRG8` es `Mono8`, 1 B/px); las cámaras**M3C**Bayer transmiten en 8 o 12 bits (el firmware M3C emite de forma silenciosa BayerRG12 incluso cuando se solicita BayerRG8). Así pues, un fotograma a resolución completa de 4 cámaras ocupa**~12,6 MB si todas son de 8 bits, pero ~25 MB con tres cámaras mono de 12 bits**. La proyección determina el formato de cada cámara a partir de su modelo (caché de identidad), de modo que la ráfaga coincide con lo que realmente transporta el cable, y no con una suposición genérica de BayerRG8.
- **Un adaptador USB-Ethernet tiene un límite de 200 MB/s, independientemente de lo que indique su placa de características.** La tabla de eficiencia que convierte una velocidad de enlace en una cifra sostenida se deriva del PCIe; una tarjeta de red USB anuncia su velocidad de enlace *Ethernet*, pero está limitada por el bus USB y su controlador. Un adaptador USB de 10 GbE solía alcanzar unos ~1063 MB/s «sostenida» —una cifra que nunca se comprobó— y la regulación resultante corrompía entre el 6 % y el 18 % de las tramas, aunque seguía indicando un número de fotogramas por segundo (fps) objetivo correcto. Las tarjetas de red conectadas por USB tienen ahora un límite absoluto: **200 MB/s** en Windows
y en NVIDIA Jetson, **400 MB/s** en Linux
x86_64 (en Linux
, a un adaptador conectado a un puerto USB 2.0 se le asignan **30 MB/s**, y las especificaciones de capacidad indican que se conecte a un puerto USB 3) (el límite lo marcan el bus USB y el controlador del host, por lo que no se ajusta a las especificaciones nominales; un adaptador USB de 1 GbE alcanza unos 80 MB/s y no se ve afectado). En Linux
, el adaptador se identifica a través de sysfs (recurriendo a `ethtool -i` de `bus-info`). Cada cifra se ha medido: en Windows
, la misma matriz de 4 cámaras corrompió entre el 6 % y el 10 % de los fotogramas por encima de los 200 MB/s; en Linux
x86_64, el mismo adaptador funcionó sin problemas en todos los límites máximos hasta los 480 MB/s (273 MB/s entregados, el límite propio de las cámaras). En NVIDIA Jetson, el mismo adaptador funcionó sin errores solo a 200 MB/s y perdió fotogramas de forma intermitente por encima de esa velocidad. `wire_ceiling_source` en el registro de capacidades lo indica con palabras, y `nic_is_usb` lo señala. Se puede anular en ambos casos con `--wire-ceiling-mbps`.
- **La admisión tiene en cuenta el consumo de datos, no la diferencia entre ráfaga completa y anillo.** Una ráfaga simultánea solo tiene que caber en el *atraso transitorio* = `max(0, Σ per-cam arrival − host drain) × emit_window`, no en toda la ráfaga. En una estructura de host rápido / cámara lenta (un **PCIe**de 10 G + 4 cámaras de 1 GbE: llegada ≈ 320 MB/s, drenaje ≈ 1063 MB/s), el host vacía más rápido de lo que se llenan las cámaras, el backlog es ≈ 0, por lo que la emisión simulada a resolución completa**se admite**aunque la ráfaga de 25 MB supere el anillo de 13,5 MB. Si se conectan las mismas cuatro cámaras a un adaptador**USB** de 10 GbE en Windows
, el drenaje es de 200 MB/s, no de 1063: la llegada lo supera, y la pérdida se manifiesta como tramas corruptas en lugar de como una frecuencia de tramas más baja. En un host de 1 GbE, el umbral mínimo de DLThr de las cámaras (31,25 MB/s) hace que la llegada supere la capacidad de descarga → se **bloquea** correctamente (para *esta* clase de bloqueo, reduce el ROI o utiliza un binning ≥ 2). La admisión es una de las **dos** puertas de conexión; la otra es la comprobación de sobresuscripción agregada que se describe a continuación.
- **Los fps previstos corresponden al ciclo de la cámara más lenta.**En la emisión simultánea, todas las cámaras envían su fotograma al mismo tiempo, por lo que la matriz funciona al ritmo del tiempo más largo entre la lectura del sensor y el tiempo de emisión de su miembro más lento (bytes del fotograma ÷ su tasa de emisión: el**enlace de acceso** de la cámara, 1 GbE ≈ 80 MB/s, o su parte del límite máximo de banda del host cuando este sea menor). Cada exposición se solapa con la lectura del fotograma anterior, por lo que la exposición no entra en el ciclo: la exposición automática ocupa el periodo que marca la tasa, luego aumenta la ganancia, y una escena oscura ya no reduce la tasa (en la versión 1.2.0 sí lo hacía). Solo una exposición manual más larga que el periodo ralentiza el disparo (véase [Exposición en un miembro de la matriz](#exposure-on-an-array-member)). Añadir cámaras no ralentiza la matriz hasta que se reducen sus tiempos de transmisión compartidos. En la emisión en serie (`slip-emit-and-capture`), en cambio, los tiempos de emisión de las cámaras se suman, y la exposición de cada miembro se tiene en cuenta en el escalonamiento. Medido con el sensor a pleno rendimiento, a 12 bits, en una matriz de 9 cámaras con un host PCIe de 10 GbE: 10,7 fps, o 14,0 fps con [transporte empaquetado](#packed-12-bit-transport---packed).
- **Tras una captura continua, la proyección indica el valor que se ha mantenido.** Una captura de 30 s o más a una velocidad automática registra la velocidad en la que se estabilizó con ese tamaño de matriz en este ordenador; se omiten los breves intentos del controlador de velocidad de alcanzar una velocidad mayor. A partir de ese momento, la línea de `array-connect` y el panel «Configuración de la matriz» indican esa velocidad cuando es inferior a la del modelo, junto con lo que la limitó: `clock-bound` (la matriz funcionó a la velocidad en la que se estabilizó, el bucle de captura a la espera de su reloj de activación), `loop-bound` (al bucle de captura se le acabó el tiempo), `capture-bound` (la captura se retrasó y se saltó grupos) o `measured` (no se especifica cuál). La cifra del modelo se encuentra en `wire_fps_max`. La matriz sigue alcanzando la velocidad que soporta la conexión.
- **Una matriz funciona en cualquier red; lo que limita el rendimiento es la frecuencia de fotogramas.**Cada cámara obtiene una parte equitativa del**límite máximo del cable a prueba de colisiones**(`sustained × sim_emit_factor`), sincronizada con GevSCPD, y todas ellas siguen exponiendo con el mismo disparador. En un cable lento, la cuota es pequeña y la velocidad de fotogramas desciende en consecuencia: 8 cámaras de 12 bits y resolución completa en una conexión de 1 GbE sin tramas jumbo obtienen 52 / 8 = 6,5 MB/s cada una, aproximadamente**1 fps**(aproximadamente 1,3 fps comprimidos), mientras que un host de 10G las ejecuta a una velocidad de entre 11 y 14. Si las cámaras no pueden sincronizarse a un ritmo tan lento —compruébalo leyendo el rango GevSCPD de cada cámara—, la matriz cambia a emisión**en serie**(`slip-emit-and-capture`): las cámaras se turnan en el cable a la velocidad máxima del enlace, activándose una tras otra, de modo que todas las tramas siguen llegando, pero los miembros**no quedan expuestos simultáneamente**. `analyze-array` y la salida de «connect» indican el modo; fija `--force-tier sim-capture-sim-emit` para rechazar el modo de reserva. El tamaño de fotograma no influye en este reparto, por lo que**el binning y una zona de interés (ROI) más pequeña aumentan la frecuencia de fotogramas, pero no alteran si una matriz cabe**. Solo un cable que no pueda transportar la matriz ni siquiera en serie está**sobrecargado**—por ejemplo, si hay muchas más cámaras de las que un cable puede gestionar, o si hay muchas otras cámaras transmitiendo por él— y, en ese caso, el síntoma es la pérdida de paquetes GVSP, no una velocidad inferior: `analyze-array` pone a cero las cifras de fps alcanzables y muestra `** OVER-SUBSCRIBED**`; la conexión se lleva a cabo con una advertencia sonora, y una conexión HTTP
con `pin_resolution: true` (modo estricto) **se rechaza**. Las soluciones son reducir el número de cámaras, utilizar tramas jumbo de extremo a extremo o una tarjeta de red más rápida. `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` rebaja el rechazo a una advertencia sonora para trabajos de banco de pruebas; véase [Variables de entorno](#environment-variables).

### Estado de la matriz: qué subsistema está perdiendo tramas

El `GET /api/camera/array/<array_id>/capability` de una matriz conectada contiene un bloque
`health` en tiempo real, que se reevalúa en una ventana móvil de **10 segundos**. Desglosa la pérdida de tramas
en las dos causas que requieren soluciones opuestas, en lugar de informar de una tasa «incompleta»
que no especifica ninguna de ellas:

| Campo | Qué significa | Qué subsistema |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (por serie) | La trama **llegó y presentaba un error estructural**— Pérdida de paquetes GVSP. |**Red**: ancho de banda, ritmo de transmisión, anillo de recepción de la NIC, MTU |
| `never_arrived_rate_pct` (por número de serie) | La trama **nunca llegó**— la cámara no se activó o no se envió nada desde ella. |**Disparador / sincronización**: cable M8, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | La peor tasa de cada cámara. | — |
| `per_cam_rate_pct` | Tasa combinada de fallos por cámara (ambas causas juntas). | — |
| `stable_for_seconds` | Tiempo que cada cámara ha permanecido por debajo del 0,01 %. | — |

Por encima del 5 %, el backend registra una línea `[array-health <id>] WARN` indicando la división: en la
primera infracción, al cambiar de nivel de gravedad, una vez por minuto mientras persista y una vez cuando
se resuelva. La mitad dañada escribe `[gvsp-corrupt <SN>]` en el archivo de registro del backend en el
primer impacto por cámara y motivo, y luego un resumen cada 60 s; el panel de registro de la aplicación muestra la
línea `[array-health <id>]`. Todas las evaluaciones siguen registrándose en el archivo de registro del backend;
los contadores avanzan con cada búfer, independientemente de lo que se imprima.

El mismo registro indica el número del que depende toda la asignación:

| Campo | Qué significa |
| --- | --- |
| `wire_ceiling_mbps` | El presupuesto de ancho de banda sostenido del host actualmente en vigor, en MB/s. |
| `wire_ceiling_source` | De dónde procede esa cifra, en palabras —p. ej., `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` (en Linux
x86_64, `USB-capped 400 MB/s (…; sysfs: bus=usb (usb2/2-1/2-1:1.0))`) o `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` cuando lo establece `--wire-ceiling-mbps` (o el campo **Presupuesto de conexión** de la interfaz gráfica de usuario). |
| `nic_is_usb` | `true` para un adaptador Ethernet USB; véase el límite de USB anterior. |

**Interpretación:** un valor distinto de cero en `gvsp_corrupt_rate_pct` con `never_arrived_rate_pct` en 0
significa que el disparo y la sincronización del cable son perfectos y que el 100 % de la pérdida se produce en la ruta de red
; reduce el valor de `--wire-ceiling-mbps` y vuelve a conectar. El patrón inverso apunta, en cambio, al
cable de sincronización o a la línea de disparo.

> **`--target-fps` no es el factor determinante de las tramas corruptas.** El ritmo de GevSCPD se establece
> una sola vez al conectarse, por lo que reducir la frecuencia de disparo cambia el ciclo de trabajo y no la
> tasa de ráfagas de emisión simultánea. Una reducción medida de 5× en la demanda no produjo ninguna mejora;
> reducir el límite máximo del cable de 240 a 200 MB/s hizo que ese mismo equipo pasara del 10,4 %
> de tramas corruptas al 0,00 %.

> **Una matriz en funcionamiento no puede replanificarse por sí misma con el firmware LATTICE (M3C / M3M).** No
> puede solucionar esto por sí misma; desconéctala y vuelve a conectarla para que el selector de tiempo de conexión pueda
> replanificar con el nuevo límite máximo.

### Síntoma → solución

| Síntoma (Configuración de la matriz / conexión / `analyze_array_network`) | Causa | Solución |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` se restablece a 32 (normalmente tras una actualización del controlador) | Establece `ReceiveBufferLen`→256, `PendingReceives`→64; vuelve a abrir el panel (reinicia el backend si ha almacenado en caché el tamaño de anillo anterior) |
| El reinicio o el apagado se cuelgan; posteriormente, `Invalid class`: errores de WMI, la tarjeta de red no se activa, faltan las unidades USB | Antiguo controlador Realtek USB 10GbE de 2016 → pantalla azul de la muerte (BSOD) `0x9F` → apagados forzados | Actualiza el controlador del adaptador a la versión ≥ v10.67 (2026) y, a continuación, vuelve a aplicar la configuración del anillo de recepción indicada anteriormente |
| El Array se conecta a la máxima capacidad del sensor, pero con una baja tasa de fotogramas | El modo «Auto» mantiene la máxima capacidad del sensor y deja que la tasa caiga hasta lo que admite el cable | Elige una de las opciones más rápidas que aparecen en la lista de salida de conexión (`--binning 2` / `4`, o un `--width`/`--height`), habilita los tramas jumbo o actualiza el enlace |
| El Array indica una frecuencia de fotogramas objetivo correcta, pero solo entrega una fracción de la misma; `health.gvsp_corrupt_rate_pct` distinto de cero, `never_arrived_rate_pct` 0 | El presupuesto de ancho de banda estimado del host sobreestima lo que realmente soporta (algo habitual en un adaptador USB Ethernet, un carril PCIe de ancho reducido o una estructura compartida) | Vuelve a conectarte con un valor de `--wire-ceiling-mbps` más bajo y vuelve a comprobar el bloque de estado. **No** `--target-fps` — El control de ritmo de GevSCPD se fija al conectarse |
| Linux
: una matriz pierde grupos de tramas y reduce su velocidad de tramas, mientras que la tarjeta de red no informa de pérdidas; `lattice network` muestra `socket receive buffers too small` | Los búferes de recepción de sockets del kernel están en su valor por defecto, por lo que los paquetes de la cámara se pierden en el kernel | Aumenta `net.core.rmem_max` / `rmem_default` a 32 MB — véase [hosts Linux
](#linux-hosts--socket-receive-buffers). El paquete .deb lo hace durante la instalación |
| Faltan cámaras en los grupos publicados; `health.never_arrived_rate_pct` distinto de cero, `gvsp_corrupt_rate_pct` 0 | Ruta de activación/sincronización: las cámaras no se activan, no es un problema de red | Comprueba el cable de sincronización M8 y `--line`; confirma que todos los miembros estén armados (`TriggerMode=On`) |
| Se ha superado `**OVER-SUBSCRIBED**` / `Wire budget` en `analyze-array`, o se ha producido un rechazo de conexión en modo estricto (`pin_resolution`; `array over-subscribes the wire`) | El cable no puede transportar la matriz a ninguna velocidad que alcancen las cámaras, ya sea simultáneamente o por turnos: hay muchas más cámaras de las que el cable puede compartir, o hay otras cámaras transmitiendo por él | Menos cámaras, tramas jumbo de extremo a extremo o una tarjeta de red más rápida. **El ROI o el binning NO servirán de nada** (el uso compartido es independiente del tamaño de la trama). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` se anula en el banco de pruebas (acepta la pérdida de paquetes) |
| La matriz se conecta en `slip-emit-and-capture` (serial) en una red lenta | Las cámaras no pueden sincronizarse lo suficientemente lento como para compartir el cable simultáneamente, por lo que se turnan; las tramas no se exponen en el mismo instante | Las tramas jumbo o una tarjeta de red más rápida restablecen la captura simultánea; `--force-tier sim-capture-sim-emit` insiste en ello (y, en su lugar, pierde paquetes) |

---

## `chloros-cli daq`

Comandos del sensor espectral. Los comandos `pool-*` son clientes HTTP
ligeros que controlan el sensor a través del grupo persistente del backend, por lo que necesitan un backend en ejecución (véase [Iniciar el backend](#start-the-backend)). El backend es el propietario del transporte, por lo que los scripts de la CLI
y el SDK
comparten un único identificador activo en lugar de disputarse el puerto serie. `pool-discover` es la única forma de conocer la dirección MAC BLE de un DAQ-M.

> **`chloros-cli daq --help`** (o `-h`, o `daq help`) muestra la lista de subcomandos de `pool-*` (en Linux
, junto con los comandos directos de hardware que incluye la compilación), y `chloros-cli daq <subcommand> --help` (o `daq help <subcommand>`) muestra las opciones de cada uno. Estas abarcan la conexión, transmisión, grabación de archivos `.daq` calibrados, intercambio de perfiles de capacidad y vinculación de un DAQ a una cámara; y, en el caso de un DAQ-E, su configuración de red, PTP, reinicio y actualizaciones de firmware. El DAQ también se puede controlar desde Python
con `chloros_sdk.connect_daq_sensor()`, que utiliza la misma ruta compartida.
>
> **Utiliza los comandos `pool-*` en todas las plataformas.** Funcionan igual en Windows
, Linux
y Jetson. En Windows
, `daq discover` y `daq list` son alias de `pool-discover`, y cualquier otro comando `daq` que no sea `pool-*` da como resultado `1` con `DAQ direct-hardware commands require the full daq package`. En Linux
y Jetson, `daq discover`, `daq list` y los demás comandos que no sean `pool-*` ni `daq` son los comandos directos de la herramienta DAQ: abren el sensor en el propio proceso de la CLI
, fuera del grupo del backend (por lo que no se trata de un sensor que mantenga el backend, la interfaz gráfica de usuario o un script), y allí se aceptan `daq discover`, `--only` y `--timeout`, pero no `--json`. Esta referencia documenta únicamente los comandos `pool-*`.

### Flujo de trabajo de la primera conexión del sensor DAQ

```bash
# 1. Scan every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U serial port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq pool-discover
chloros-cli daq pool-discover --only ble           # BLE only
chloros-cli daq pool-discover --json               # machine-readable

# 2. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect                                   # smart-detect
chloros-cli daq pool-connect --port /dev/ttyUSB0               # DAQ-U on a serial port (Windows: COM3)
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 3. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 4. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

#    Bind it to a camera, so that camera's captures use and save its readings
chloros-cli daq pool-bind --camera 213800234 CB-7C-A8-2E-5F

# 5. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 6. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### Referencia de `pool-*`

| Subcomando | Finalidad |
| --- | --- |
| `daq pool-connect` (smart-detect) | Abre un sensor en el grupo de backend. |
| `daq pool-connect --port PORT` (`-p`) | DAQ-U en un puerto serie específico. |
| `daq pool-connect --ble` | DAQ-M a través de BLE, MAC escaneado automáticamente. |
| `daq pool-connect --mac MAC` (`-m`) | DAQ-M a través de BLE con una dirección MAC conocida (implica `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E a través de Ethernet con un nombre de host o una dirección IP conocidos (implica `--eth`). |
| `daq pool-connect --eth` | DAQ-E a través de Ethernet, host detectado automáticamente (mDNS + ARP como alternativa; funciona con una caché ARP vacía en Windows
y Linux
). |
| `daq pool-connect --integration-time MS (-t) --frame-avg N (-f) --no-ae` | Ajustar la ventana de integración / estado AE. |
| `daq pool-connect --no-stream` | Conectarse, pero sin iniciar aún la transmisión (reanudar con `pool-stream --start`). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | Perfil de corrección de capacidad. El valor predeterminado en el backend es `sunshine_cosine`; `none` es un sensor sin cubierta. Un [DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor) lo ignora (difusor fijo). **Los sensores se suministran sin tapa, así que asigna un nombre a la tapa que hayas instalado:** el valor por defecto se registra como (`cap_id_source=auto_default`) y el procesamiento avisa al respecto, ya que un DAQ-U sin tapa bajo la corrección de luz solar da una lectura aproximadamente 30 veces superior. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Escanea cada transporte en busca de sensores a los que puedas conectarte, sin conectarte. **Así es como se encuentra la dirección MAC BLE de un DAQ-M.** `--timeout` se aplica por protocolo de transporte (por defecto 5 s, 1-20). Los sensores que ya están abiertos en el grupo no aparecen en la lista —un DAQ-M conectado deja de anunciarse—, así que utiliza `pool-list` para esos. `usb` solo muestra dispositivos serie USB: los puertos serie propios del ordenador (`COM1`, `/dev/ttyS*` en Linux
) no pueden ser sensores y se omiten. |
| `daq pool-list` | Muestra todos los sensores del grupo de fondo. |
| `daq pool-bind (--camera SERIAL \| --array ARRAY_ID) {SENSOR \| --clear}` | Vincula una cámara o una matriz a un DAQ, tal y como lo hace la selección de sensores de luz de la interfaz gráfica de usuario: sus capturas utilizan, comparan y guardan las lecturas de ese DAQ. `SENSOR` es un identificador de sensor de `pool-list` o la dirección en la que se accede al DAQ. `--clear` (no `SENSOR`) elimina la vinculación, y las capturas vuelven al DAQ más reciente. La vinculación dura hasta que se reinicia el backend, o hasta que esa cámara, matriz o DAQ se desconecta: una desconexión (aquí, desde el SDK
o en la aplicación) rompe todas las vinculaciones del hardware que se desconecta, independientemente de quién las haya establecido, y las de los miembros de una matriz con la propia matriz. La cámara debe estar conectada, en el proyecto abierto, o haberse abierto previamente en este ordenador, y la matriz debe estar conectada o en el proyecto abierto; cualquier otro número de serie o identificador es rechazado por su nombre y no se establece ningún enlace. |
| `daq pool-disconnect {--sensor-id ID \| --all}` | Liberación: quienquiera que haya abierto el sensor; por lo tanto, también libera un sensor que un script que se ha bloqueado haya dejado en el pool (previamente se completará cualquier grabación que haya en él). Un sensor que no se encuentre en el pool no tiene nada que liberar: muestra `ID: already released (not in the pool)` y sale con `0`; una liberación que el backend ha rechazado o no ha confirmado sale con `1`. SDK
: `DAQSensorSession.force_disconnect()`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Las N tramas de espectro más recientes (por defecto, 1). |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Reanudar/pausar la transmisión. |
| `daq pool-record --sensor-id ID [--duration SEC] [-o DIR] [--device-name NAME] [--confirm-after S] [--heartbeat S]` / `--stop` | Iniciar/detener una grabación .daq. `-o` toma por defecto el valor de `~/Documents/DAQ Live View` en el ordenador del backend (una ruta relativa se encuentra en la carpeta desde la que se ejecuta el comando); `--duration` (`-d`, por defecto `0` = hasta `--stop`). `--confirm-after S` avisa cuando no ha llegado ninguna lectura S segundos después del inicio (por defecto 20; 0 lo desactiva); con `--duration`, `--heartbeat S` muestra el recuento de lecturas cada S segundos y avisa cuando deja de aumentar (por defecto 30; 0 lo desactiva). Una grabación en un DAQ-E que se queda en silencio continúa en el mismo archivo una vez que el backend se ha vuelto a conectar, y `--stop` enumera los huecos ([Un DAQ-E que deja de enviar datos](#a-daq-e-that-stops-delivering)). Ctrl+C finaliza prematuramente una grabación de `--duration` y lo notifica; sin `--duration`, la grabación sigue ejecutándose tras pulsar Ctrl+C (salir de `130`) hasta `--stop`. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Cambia el perfil de corrección de la tapa en tiempo de ejecución. Se requiere `--cap-id`; `--cap-id none` elimina la corrección de la tapa (un sensor sin tapa). Un DAQ-E-S mantiene `as_recorded` sea cual sea el valor establecido. |
| `daq pool-enable-ptp --sensor-id ID` | Vuelve a activar el PTP en un DAQ-E cuyo PTP esté desactivado. La configuración se guarda en la unidad, por lo que una unidad que se haya desactivado permanece desactivada —sus marcas de tiempo no están sincronizadas con PTP— hasta que se ejecute este comando. `pool-list` marca dicha unidad (`!! PTP is switched OFF`) y muestra este comando para ella. Chloros
nunca vuelve a activar una unidad por sí mismo, ya que el estado desactivado puede ser intencionado. |
| `daq pool-info --sensor-id ID [--json]` | Ajustes propios de un DAQ-E: su firmware (y si esta versión de Chloros
incluye uno más reciente), su dirección y cómo la obtuvo (DHCP / link-local / estática), la configuración de red guardada hasta que el sensor la ejecute (una guardada con `--no-reboot` incluida), el flujo al encenderse y el PTP. En un sensor cuyo flujo no llega a este ordenador, lee el sensor en una conexión nueva y lo indica. |
| `daq pool-set-network --sensor-id ID {--auto \| --static IP [--mask M] [--gateway G]} [--no-reboot] [--force] [--timeout S]` | Configurar un DAQ-E con DHCP (`--auto`) o una dirección estática (`--mask` por defecto `255.255.255.0`). El sensor se reinicia para aplicar la configuración y se vuelve a conectar con los ajustes que tenía, esperando hasta `--timeout` segundos (por defecto 45) para ello; `--no-reboot` solo la guarda. Véase [Configuración del dispositivo DAQ-E](#daq-e-device-settings). |
| `daq pool-set-auto-stream --sensor-id ID {--on \| --off}` | Indica si un DAQ-E inicia la transmisión al encenderse sin ningún software conectado (guardado en el sensor). Chloros
inicia la transmisión por sí mismo en cualquier caso. |
| `daq pool-reboot --sensor-id ID [--timeout S]` | Reiniciar un DAQ-E y volver a conectarlo con su configuración anterior (espera hasta `--timeout`, por defecto 45 s). |
| `daq pool-update-firmware --sensor-id ID [--timeout S]` | Actualiza el firmware del DAQ-E incluido en esta versión de Chloros
en un sensor que ejecute una versión anterior y, a continuación, vuelve a conectarlo (aproximadamente un minuto; no lo apagues; `--timeout` por defecto 45 s para que vuelva a estar operativo). Un sensor que ya esté actualizado no se toca. |

### Un DAQ-E que deja de transmitir

El backend vuelve a conectar un DAQ-E del grupo que no transmite ningún espectro durante 20 segundos. Esto se consigue apagando y encendiendo el conmutador que alimenta el sensor: el sensor se reinicia con ello y no vuelve a transmitir hasta que se le indique que lo haga (a menos que se active `pool-set-auto-stream --on`).

- **Primero comprueba que se trata del mismo sensor.** Solicita, en la dirección que tenía el sensor y sin cambiar nada, el nombre, la dirección MAC y el número de serie del sensor. También realiza la consulta bajo el nombre `daq-e-<id>.local` del sensor, pero solo confía en esa respuesta cuando procede del propio adaptador de red (MAC) del sensor. No se vuelve a conectar ni se escribe en ningún otro dispositivo que responda.
- **Se vuelve a conectar con la configuración de la sesión:** tiempo de integración, promedio de fotogramas, exposición automática y perfil de captura tal y como estaban, no con los valores predeterminados de encendido del sensor. Si un sensor tenía PTP activado y ha vuelto con PTP desactivado, se vuelve a activar. A continuación, reinicia la transmisión y el receptor del ordenador destinado a ella.
- **Vuelve a intentarlo a intervalos cada vez mayores:** de inmediato, luego tras 5, 10, 20 y 40 segundos, y después cada 60 segundos mientras el sensor siga sin estar disponible. En cada intento solo se le pregunta al sensor cuál es su identificación. Un sensor que responde pero que sigue sin enviar nada tras tres intentos de reconexión se deja tal cual y se notifica (`auto-reconnect GAVE UP`). `pool-stream --start` vuelve a intentarlo.
- **`pool-list` muestra su estado.** Mientras el sensor permanece en silencio, su línea `STREAM FAULT` indica lo que está haciendo el proceso de reconexión. La línea se borra con el primer espectro y, a continuación, el sensor muestra `reconnected automatically N time(s)`.
- **El registro continúa en el mismo `.daq`.** El archivo marca cada interrupción con dos filas: `event_type` 5 en el último espectro antes de la interrupción y 6 en el primero después de ella, cada una con solo una marca de tiempo. `pool-record --duration` muestra `!! STREAM LOST …` mientras el sensor está ausente y `stream resumed after …` cuando regresa. `pool-record --stop` enumera cada interrupción y emite una advertencia, al igual que lo hace con un archivo vacío (una grabación que no contiene ninguna lectura sale de `1`).
- **Una grabación conserva una calibración.** Si el sensor vuelve con una calibración diferente mientras se está realizando una grabación, el resto de la grabación conserva la que tenía al inicio, y la advertencia de parada así lo indica. `pool-disconnect` + `pool-connect` adopta la nueva.
- **Tus comandos tienen prioridad.** Un sensor al que se le aplique `pool-disconnect`, o se ponga en pausa con `pool-stream --stop`, nunca se vuelve a conectar, incluso cuando ya se esté llevando a cabo una reconexión.
- **`pool-info` en un sensor silencioso** lee el sensor en una conexión nueva y añade `!! stream NOT reaching this computer`. Un sensor que no responde se notifica de inmediato. `pool-list` ya no muestra un estado PTP para un sensor que ha dejado de responder.
- No se incluyen el DAQ-U ni el DAQ-M. Un DAQ-M se vuelve a conectar por sí solo a través de Bluetooth, y un DAQ-U desconectado del USB necesita volver a introducir el código `pool-connect`.

### DAQ-E-S — difusor fijo y sensor de inclinación

El **DAQ-E-S** es un DAQ-E con dos diferencias relevantes para el software. Todo lo demás —detección, conexión, PTP, grabación y la configuración del dispositivo que se indica a continuación— funciona exactamente igual que en un DAQ-E.**Su difusor es permanente, por lo que no requiere corrección de tapa.** El difusor tipo Sunshine está integrado y ya se encontraba en la unidad cuando se realizó su calibración de fábrica, por lo que dicha calibración ya indica la irradiancia descendente real. Chloros
establece su tapa en `as_recorded` (sin perfil) independientemente de lo que se solicite: un límite con su nombre (`sunshine_cosine`, `fov_*`, `none`) se ignora con un mensaje, y sus registros almacenan `cap_id_source=model`, por lo que la advertencia de límite supuesto nunca se activa en su caso. Tampoco apliques un límite a sus datos manualmente: un `sunshine_cosine` en la parte superior daría una lectura aproximadamente 11 veces superior a la real.

**Incorpora un acelerómetro de 3 ejes.** La respuesta coseno del difusor de luz solar solo es válida mientras está nivelado, por lo que un sensor inclinado indica una irradiancia errónea —y la reflectancia hereda el error— sin que se observe ningún otro síntoma. Por lo tanto, cada lectura del DAQ-E-S incluye su actitud:

| Campo | Significado |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | Ángulos en grados. `None` cuando el firmware marca el ángulo como indefinido (caída libre, un eje inactivo). |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | Aceleración en miligravs. Un valor de `mag_mg` muy alejado de ~1000 indica que la unidad se estaba moviendo y que los ángulos no son significativos para esa muestra. |
| `cal_applied` | Activado: los ejes y los ángulos están en el sistema de referencia del difusor (se aplica la calibración de montaje de la unidad). Desactivado: sistema de referencia sin procesar de la placa, que puede mostrar hasta ~2,3° de inclinación que no es real. |
| `sample_age_ms` | Antigüedad de la muestra del acelerómetro en el momento en que se envió el espectro. |
| `temp_raw` | **Tendencia** de la temperatura del chip sin calibrar: solo diferencias, nunca una lectura del termómetro. |

`daq pool-latest --json` los devuelve bajo la clave `imu` de cada fotograma (`null` en un DAQ-U, DAQ-M o un DAQ-E sin más). Un `.daq` registrado desde un DAQ-E-S almacena los mismos valores en sus columnas `imu_*` (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, `imu_mag_mg`, `imu_cal_applied`, …) junto a cada espectro, de modo que el lector pueda distinguir una nube de un trípode que se ha movido. En todos los demás modelos, esas columnas están vacías.

El flujo de actitud se registra por separado: el `daq pool-list` muestra el `!! ATTITUDE STREAM FAULT (spectra unaffected)` mientras está inactivo, y de cuántos fallos se recuperó posteriormente. Un fallo en este caso nunca provoca una interrupción en el registro espectral. `daq pool-discover` identifica la unidad como `DAQ-E-S` (como `DAQ-E` si no responde a la comprobación de identidad en unos 2 s).

### Ajustes del dispositivo DAQ-E

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330

# A fixed address on the network this computer is on (192.168.1.x)
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1

# Back to the factory setting: DHCP, falling back to a 169.254.x.x
# link-local address when no DHCP server answers
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto

chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

- **`--static` sin `--gateway` no guarda ninguna puerta de enlace** (se descarta cualquier puerta de enlace configurada anteriormente), lo cual es adecuado para una red de cámaras sin router.
- **Debe haber una dirección estática en la red de este ordenador.** Chloros
rechaza aquellas subredes que no incluyan la dirección de este ordenador en la red del sensor: tras el reinicio, nada podría acceder al sensor para revertir el cambio. Aplica `--force` solo cuando este ordenador vaya a ser redirigido o enrutado a la nueva subred.
- **No se permite reiniciar mientras el sensor está grabando.** Detén primero la grabación (`pool-record --stop`). Una grabación iniciada justo antes de un reinicio se cierra con este, y los códigos `pool-set-network`, `pool-reboot` y `pool-update-firmware` terminan con una línea `Warning:` que indica el nombre del archivo de grabación (incluso cuando el sensor no vuelve a conectarse).
- **Tras un reinicio, Chloros
solo vuelve a conectar el propio sensor.** Espera en la ubicación donde se encontraba el sensor —y, tras `--auto`, también bajo su nombre `daq-e-<id>.local`— una respuesta con el nombre del sensor, nombre de host y modelo, desde el adaptador de red (MAC) que tenía, y lo vuelve a conectar en la dirección que respondió. Solo se acepta una respuesta con ese nombre si procede de esa dirección MAC concreta; por lo tanto, desde un ordenador que accede al sensor a través de un router, o desde WSL2 en su configuración de red NAT predeterminada, un sensor que el DHCP haya trasladado a una nueva dirección no se vuelve a conectar: el comando lo indica y muestra el comando `pool-connect --eth-host <its address> …`, que lo recupera con los ajustes de límite y exposición que tenía.
- **`pool-update-firmware` rechaza un sensor cuya versión de firmware no puede leer.** Un sensor que tarda más del tiempo de espera en responder tras el flash no ha fallado: una vez que responde, se vuelve a conectar con los ajustes del nuevo firmware ya configurados, y el comando advierte de que ha tardado en responder.
- **Una actualización de firmware requiere una conexión TCP entrante.** El sensor se vuelve a conectar a este ordenador para recopilar la imagen. Chloros
escucha en el adaptador que redirige al sensor (en todos los adaptadores solo cuando no puede determinar cuál es) y atiende únicamente a la propia dirección del sensor; cualquier otra conexión se cierra sin ser atendida. En Windows
, la regla de cortafuegos `Chloros Backend (TCP-In)` del instalador permite el acceso (véase [Windows
(.exe)](#windows-exe)); un cortafuegos que la bloquee provocará un error en la actualización con el código `… device never connected back on TCP <port> …`.
- **En WSL2, utiliza la red duplicada** (`networkingMode=mirrored` en `%UserProfile%\.wslconfig`). En el modo NAT predeterminado, la dirección de este ordenador en la red del sensor es la propia de WSL, por lo que se rechaza cualquier dirección estática (salvo `--force`), la transmisión en directo no llega a WSL y la actualización del firmware no puede completarse, ya que el sensor se reconecta al ordenador para recibir la imagen.
- **El ID del sensor y el nombre de host no cambian.** Sea cual sea su dirección, un DAQ-E sigue siendo `daq-e-<id>` y `daq-e-<id>.local`, por lo que `--eth-host daq-e-<id>.local` sigue encontrándolo.
- **La interfaz gráfica de usuario (GUI) tiene la misma configuración**: pestaña «Sensores de luz», configuración del sensor (icono de engranaje), «Red». La GUI mantiene su propia conexión con un sensor, por lo que si se modifica un sensor que la GUI tiene abierto desde la propia GUI: un reinicio desde la CLI
termina la sesión de la interfaz gráfica con él.
- Los equivalentes del SDK
son `DAQSensorSession.device_info()`, `set_network()`, `set_auto_stream()`, `reboot()` y `update_firmware()`.

---

## `chloros-cli project`

Abrir, conectarse y controlar un proyecto guardado de Chloros
(una carpeta con `cameras.json` + `sensors.json` + `project.json`). Todo se gestiona a través del backend, por lo que la interfaz gráfica de usuario (GUI) y la línea de comandos (CLI
) producen un estado de hardware idéntico.

### Referencia de subcomandos

| Subcomando | Finalidad |
| --- | --- |
| `project open PATH [--relink FOLDER]` | Imprime el manifiesto de dispositivos del proyecto (cámaras, matrices, sensores), lo lee de sus archivos y comprueba sus archivos fuente (véase [Un proyecto movido](#a-moved-project---relink)). No abre el proyecto en el backend: el proyecto, ya abierto allí, así como sus dispositivos y grabaciones, permanecen tal y como están. Sale de `1` cuando no se encuentra ninguno de los archivos fuente del proyecto. |
| `project devices PATH [--reconnect]` | Mostrar o volver a ejecutar el descubrimiento. |
| `project connect PATH [--cameras-only] [--sensors-only] [--packed {auto,on,off}]` | Conectar todas las cámaras, matrices y sensores guardados. Cada matriz se conecta como lo hace `lattice array-connect`: hasta 600 s, imprimiéndose sus pasos en stderr (`[array <id>]    <s> s: <step>`; stdout es el informe en JSON
), y una matriz que no se haya conectado en ese tiempo se deshace —sus cámaras vuelven a su estado inicial— y se informa de ello en `errors`, sin dejarla nunca conectada. `--packed` configura el transporte empaquetado de 12 bits de cada matriz (por defecto, `auto`; un proyecto no guarda la configuración por matriz; véase [Transporte empaquetado de 12 bits](#packed-12-bit-transport---packed)). Cada sensor de luz se abre en el grupo DAQ del backend en su dirección guardada con su tiempo de integración, promedio de fotogramas, exposición automática y límite guardados: un sensor sin límite de exposición seleccionado recibe `sunshine_cosine`, uno guardado sin configuración (`none`) permanece sin configuración, y un DAQ-E-S nunca admite un límite de exposición. El panel DAQ de la interfaz gráfica de usuario no forma parte del grupo: cierra primero un sensor allí. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | Captura única desde una cámara o matriz con nombre (`--prefix` por defecto, `capture`). `--orient` (matrices): la matriz guarda los datos sin procesar, que permanecen en formato nativo; esto indica que los productos que `process` genera a partir de ellos deben orientarse como `array-capture --orient`. Por defecto: nativo. `project burst` / `stream` y los `capture` / `burst` / `stream` (`orient: true`) deben tener la misma opción. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | Ráfaga de N fotogramas de una cámara o matriz especificada (`-n/--count` por defecto 5; `-i/--interval` segundos entre fotogramas, por defecto 0). Las ráfagas de matrices eliminan los duplicados de grupos sincronizados repetidos (control de caducidad), de modo que una matriz de ciclo parcial no puede devolver N copias de un fotograma; imprime los resultados por iteración. Una captura que falla a mitad de camino detiene la ráfaga y sale de `1`, indicando cuántas se guardaron antes de que fallara. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--orient]` | Captura en disco en bucle: `--fps F` captura un segundo (por defecto 3), un fotograma por captura desde una cámara, un grupo de fotogramas sincronizados desde una matriz (con el mismo controlador de caducidad que `project burst`), hasta que `-n N` capture (por defecto, `0` = hasta que se pulse Ctrl+C, lo que muestra cuántos se han guardado y sale de `0`, o `1` cuando no se ha guardado ninguno o ha fallado un archivo; si se pulsa Ctrl+C antes de que `-n N` capture, se sale de `130`). Una captura que falla a mitad de camino detiene el flujo y sale de `1`, indicando cuántos fotogramas se guardaron antes de que fallara. `--poll-interval` está obsoleto y se ignora: aún se acepta para que se ejecuten los scripts más antiguos, con una nota en stderr. |
| `project sensor read PATH NAME [--json]` | Último fotograma del espectro. |
| `project sensor log PATH NAME [--seconds SEC] [-o DIR] [--device-name NAME]` | Graba un archivo .daq para `--seconds` (por defecto 10). Si la grabación no contiene ninguna lectura, se informa de ello en stderr y se sale de `1`. |
| `project run PATH RECIPE.yaml` | Ejecuta una receta de captura YAML/JSON
. `--dry-run` valida sin ejecutarse. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Calcula la alineación para una matriz; véase [la tabla de opciones más abajo](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | Mostrar el perfil de alineación actual. |
| `project align clear PATH NAME` | Eliminar el perfil almacenado en caché. |
| `project align tweak PATH NAME --serial SN [--dx N] [--dy N] [--rotation-deg N] [--scale N]` | Desplazar la transformación de un esclavo (por defecto, `0`, `0`, `0`, `1.0`: sin cambios). |
| `project align export PATH NAME --to FILE` | Guardar el perfil en formato JSON
. |
| `project align import PATH NAME --from FILE [--no-validate]` | Cargar un perfil guardado. |

#### Opciones de `project align calibrate`

| Parámetro | Valor por defecto | Descripción |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Método de alineación. **Estas denominaciones difieren de `lattice align-calibrate`**, que admite las formas abreviadas `orb` / `akaze` / `phase`; los dos comandos no son intercambiables en este indicador. |
| `--model {translation, rigid, affine, homography}` | `affine` | Transformar el modelo para que encaje. |
| `--frames N` | `1` | Instalaciones de fotogramas sincronizados a la media. |
| `--reference SN` | la cámara maestra | Número de serie de la cámara de referencia; todos los demás miembros se deforman para ajustarse a ella. |
| `--max-features N` | `5000` | Límite de recuento de características ORB. |
| `--ratio-threshold F` | `0.75` | Prueba de la. |
| `--ransac-threshold-px F` | `3.0` | Umbral de puntos internos de RANSAC. |
| `--min-matches N` | `15` | **Control de calidad**: rechaza la solución si no alcanza este número de coincidencias válidas. |
| `--max-reproj-err-px F` | `4.0` | **Umbral de calidad**: rechaza la resolución si supera este error RMS de reproyección. |
| `--checkerboard RxC` | — | Geometría de la placa para `--method checkerboard`, p. ej., `9x6`. |
| `--name PROFILE` | vacío | Nombre del perfil incrustado en el JSON
guardado. **No es el nombre de la matriz** — ese es el `NAME` posicional. |

Las dos comprobaciones de calidad son la razón por la que una calibración puede resolver con éxito y, aun así,
rechazar el guardado: un perfil que falle en cualquiera de ellas provocaría un desajuste silencioso en todas las
capturas posteriores, por lo que se rechaza en lugar de guardarse.

### Un proyecto trasladado (`--relink`)

El `project.json` de un proyecto nombra cada archivo fuente —cada imagen y cada
registro del sensor de luz— por su ruta completa. Si se mueve la carpeta del proyecto, o toda la carpeta de proyectos,
a otra unidad, esas rutas seguirán apuntando a la ubicación anterior.

Al abrir un proyecto, este corrige lo que puede por sí mismo: se busca un archivo que estaba dentro de la carpeta del proyecto
en la carpeta en la que se encuentra ahora el proyecto, y cada ruta que se encuentra allí
se reorienta y se guarda (`project.json` se copia primero como
`project.json.before-relink-<date>-<time>`). Esto ocurre cada vez que se abre —la aplicación,
`chloros-cli project open`, `chloros-cli process` en un proyecto existente, los del SDK

y los de `chloros-mcp`. Solo se reasigna una ruta a un archivo que se encuentre allí.

Los archivos que se encontraban fuera de la carpeta del proyecto —una sesión de Hub, una importación mediante «Añadir carpeta»—
no se pueden encontrar de esa manera. Indica el nombre de la carpeta en la que se encuentran ahora:

```bash
chloros-cli project open "N:/Chloros Projects/Field_A" --relink "E:/backup/Hub Projects"
```

`--relink` toma la carpeta a la que se trasladaron los archivos, o cualquier carpeta situada por encima de ella: cada carpeta de los archivos que faltan
se busca en ella según los nombres de las carpetas que la siguen
(`Hub Projects/<session>/raw/<serial>`, luego `<session>/raw/<serial>`, ...), y cada
los archivos de cada carpeta se toman del lugar donde se encuentren la mayoría de ellos. `project open` muestra
lo que se ha reasignado, cuántos archivos fuente se han encontrado y las carpetas en las que se encontraban
el resto. Aún faltan algunos: una advertencia, salida `0`; el proyecto se abre con el resto. No se ha
encontrado ninguno: un error al nombrar esas carpetas y `--relink`, salida `1`; el backend, del mismo modo,
se niega a abrir dicho proyecto para `process` o `project devices --reconnect` (código HTTP
409 `PROJECT_SOURCES_MISSING`).

### Ejemplos

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Its hub captures moved to another drive: point it at them
chloros-cli project open "/home/user/Chloros Projects/Field_A" --relink "/media/backup/Hub Projects"

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

### Lenguaje DSL de recetas

`project run RECIPE.yaml` admite un archivo YAML o JSON que describa una secuencia de acciones:

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
        auto_exposure: "Off"
        gain_db: 6.0
        auto_gain: "Off"
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

Acciones admitidas: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. `apply` toma las claves `cameras.json`, `settings`, `exposure_us`, `auto_exposure`, `gain_db`, `auto_gain`, `target_brightness`, … (los nombres antiguos `exposure_time`, `exposure_auto`, `gain`, `gain_auto` se aceptan como alias). Una exposición, ganancia o valor objetivo configurado de esta manera se mantiene como propio: el control automático de exposición de Chloros (incluida la exposición medida por el DAQ) no lo modifica. Una exposición manual deja la ganancia tal y como estaba: en modo automático, la cámara sigue ajustando la ganancia (en todo su rango) hacia el brillo objetivo hasta que se establezca `gain_db` o `auto_gain: "Off"`. Un comando posterior `auto_exposure: "Continuous"` devuelve la exposición al modo automático, y `target_brightness_auto: true` devuelve un objetivo fijado al objetivo automático de Chloros. La acción `burst` toma como parámetros `name` (obligatorio), `count` (por defecto 5), `interval` (segundos, valor por defecto 0), `output`, `format`, y `settings` (misma configuración por cámara que `apply`); las ráfagas en matriz utilizan el mismo watchdog de grupo recién sincronizado que `project burst`.

En el paso `capture`, `burst` o `stream`, el valor por defecto de `output` es `output`. Un `output` relativo —el valor por defecto incluido— se resuelve en función de la carpeta desde la que se ejecuta `chloros-cli project run`, no de la carpeta del archivo de receta ni de la carpeta del proyecto (en la versión 1.2.0 se ubicó en la carpeta de instalación del backend).

Ejecútalo:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

**Cada ejecución se comprueba primero con respecto al proyecto.**Cada acción debe tener como clave exactamente uno de los verbos anteriores: `- capture: {name: FrontLeft}`, no `- action: capture` ni `- type: capture` — con un `name`, el proyecto tiene `cameras.json` / `sensors.json`, una cámara o matriz para `apply` / `capture` / `burst` / `stream` y un sensor de luz para `sensor`; un `stream` necesita que `count` sea mayor que 0 y un `wait` necesita una duración. Una receta que contenga cualquiera de estos errores**no** ejecuta ninguna acción (`[recipe] error: …`, `[recipe] not run: …`, salir de `1`), ni siquiera con `stop_on_error: false`. Las teclas que un paso no utiliza son un `[recipe] warning:`, al igual que cualquier matriz o sensor de luz que controle la receta y que no esté conectado en ese momento. Una receta nunca conecta dispositivos: ejecuta primero `chloros-cli project connect PATH`. `--dry-run` imprime la receta, luego los mismos errores y advertencias, y `[recipe] valid: …` / `NOT valid: …`, no ejecuta nada y sale de `1` cuando la receta presenta un error.

---

## Variables de entorno

| Variable | Efecto |
| --- | --- |
| `CHLOROS_BACKEND_URL` | Anula la URL del backend (por defecto `http://127.0.0.1:5000`) — **solo la tienen en cuenta las familias de comandos `lattice`, `project` y `daq pool-*`** (`lattice network-analysis` utiliza `--backend-url` en su lugar). Los comandos principales (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) fijan `http://127.0.0.1:<port>` e ignoran esta variable (el literal IPv4 evita la penalización de Windows de `localhost`→`::1` de ~2 s por solicitud), por lo que siempre se dirigen a la máquina local. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | Se configura en el entorno del backend. `1` reduce la severidad del rechazo por sobresuscripción —una conexión de matriz con `pin_resolution`, o un [comando transitorio de multicámara](#multi-camera-transient-sessions) (`error_code` `ARRAY_OVERSUBSCRIBED`) — a una advertencia sonora, aceptando la pérdida de paquetes GVSP. Solo con fines de prueba: véase [Modelo de fps y ráfagas de la matriz](#array-fps--burst-model). |
| `CHLOROS_ARRAY_PACKED12` | Se configura en el entorno del backend: `on` o `off` establecen el valor predeterminado de [transporte empaquetado de 12 bits](#packed-12-bit-transport---packed) para cada matriz (por defecto, `auto`). Una configuración explícita de `--packed` sigue teniendo prioridad. |
| `CHLOROS_DISABLE_UPDATE_CHECK` | `1` (o `true` / `yes`) desactiva la [comprobación automática de actualizaciones](#chloros-cli-update) de la CLI de Linux al inicio. |
| `CHLOROS_CLI_MODE` | Lo establece la propia CLI; indica al backend que habilite el procesamiento paralelo. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` omite la prueba de respaldo de GVSP (solo resultados ICMP). **Esto desactiva los paquetes jumbo, no solo silencia el registro**: la cámara responde a pings DF solo hasta 1500 en cada ruta, por lo que esta prueba es lo único que puede detectar paquetes jumbo. Ahorra ~1 s por cámara por conexión; cuesta ~1,45× el límite del cable si la red *pudiera* haber transportado paquetes jumbo. El SDK avisa al configurarlo. |
| `CHLOROS_GVSP_SESSION_REUSE` | Configurar en el entorno del backend: `0` mide el tamaño de los paquetes en cada conexión, en lugar de una vez por cámara por sesión de backend. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | Fija el tamaño de los paquetes GVSP en N bytes; omite por completo la detección. Es preferible utilizar la opción por comando (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) en lugar de configurarlo de forma permanente: un tamaño fijo deja de adaptarse a la red que tiene delante, y fijar 9000 en una ruta que no admite paquetes jumbo hace que **todas** las capturas agoten el tiempo de espera con `SC_ERR_TIMEOUT -1011`. |
| `CHLOROS_TRANSIENT_AE_SETTLE_S` | Se configura en el entorno del backend: el número máximo de segundos que dura un [comando transitorio de multicámara](#multi-camera-transient-sessions), o un `lattice capture` de una cámara que no retiene nada, dedica a ajustar la exposición de cada cámara a partir de la escena anterior antes de conservar un fotograma (por defecto, `5`). `0` lo desactiva: la exposición automática de cada cámara se ejecuta entonces desde el primer fotograma, comenzando por su límite de exposición. |
| `CHLOROS_LOG_DIR` | Se configura en el entorno del backend: la carpeta a la que se guarda su registro detallado. Por defecto, `%LOCALAPPDATA%\Chloros\logs` en Windows y `~/.cache/chloros/logs` en Linux (`$XDG_CACHE_HOME/chloros/logs` cuando está configurado). El servicio raíz del .deb configura `/var/log/chloros`, al que tiene acceso de lectura el grupo `adm`; véase [Ejecutar el backend como servicio](#running-the-backend-as-a-service-headless). |
| `TMPDIR` (Linux) | Ubicación de los archivos temporales. La CLI utiliza automáticamente `/mnt/ssd/tmp` si está presente. |

---

## Códigos de salida

| Código | Significado |
| --- | --- |
| `0` | Éxito. |
| `1` | Fallo genérico (la mayoría de los errores de subcomandos). Un comando de captura también sale con el código `1` cuando no se ha podido escribir en algún archivo que se le había solicitado, aunque los demás se hayan guardado, y con el código `array-burst` / `array-record` y `1` cuando el grabador informa de un error; véase [Archivos que no se pueden escribir](#files-that-fail-to-write). `process` también sale de `1` cuando no se han podido verificar las exportaciones de una ejecución, o cuando se ha perdido una imagen, aunque se hayan escrito los demás productos. |
| `2` | Error de argumento (un indicador desconocido o que falta; también `time-sync set-priority` sin ninguna de las dos prioridades). |
| `130` | Interrumpido por Ctrl+C: cualquier comando que se interrumpa — `lattice`, `project`, `daq` y `process` por igual (`process` después de que el backend haya detenido la ejecución; véase [Notas sobre `process`](#notes)). Un comando cuya forma habitual de detenerse es con Ctrl+C — `lattice array-capture --continuous` / `--interval`, `lattice viewer`, `lattice continuous`, `multi-stream`, `align-stream`, `array-record`, `array-burst` y `project stream` con `-n 0` — se cierra en función de lo que haya hecho: `0` cuando se ha escrito todo, `1` cuando ha fallado algo. |
| `143` / `129` | Finalizado por SIGTERM / SIGHUP (128 + la señal), tras ejecutarse la limpieza del comando: se detuvo un trabajo en ejecución y se restablecieron sus cámaras; se detuvo una ejecución de procesamiento en el backend. Una grabación `lattice continuous` u otra cuya detención normal sea Ctrl+C también sale de `143`, no de `0`. |

`lattice network-analysis` sale de `1` para los veredictos `needs_force_slip` y `error`, y `selftest` sale de `1` a menos que se superen las siete comprobaciones.

---

## Consejos para la resolución de problemas

- **«Se requiere inicio de sesión»** → Ejecuta `chloros-cli login EMAIL PASSWORD` una vez en este equipo.
- **«La carpeta del proyecto está demasiado anidada para este ordenador»** → Las rutas largas de Windows están desactivadas en este equipo (configuración predeterminada de Windows), por lo que ninguna ruta puede superar los 260 caracteres, y los productos se encuentran hasta unos 130 caracteres por debajo de la carpeta del proyecto. No se ha procesado nada. El mensaje indica la carpeta de proyecto más larga que cabe: utiliza una más corta (`-o` o `chloros-cli set-project-folder`), o activa las rutas largas de Windows y reinicia Chloros. El instalador de Windows ofrece esta opción como «Habilitar rutas largas de Windows», marcada por defecto; de lo contrario, utiliza la Política de grupo: *Configuración del equipo &gt; Plantillas administrativas &gt; Sistema &gt; Sistema de archivos &gt; Habilitar rutas largas de Win32*.
- **«backend unreachable»** → Inicia la aplicación de escritorio de Chloros (Windows) o inicia tú mismo el backend: `"C:\Program Files\Chloros\backend\chloros-backend.exe"` en Windows (la carpeta `backend` de tu carpeta de instalación; no se encuentra en `PATH`), `chloros-backend` en Linux. O comprueba `CHLOROS_BACKEND_URL` si es remoto.
- **Los comandos `lattice` fallan con el mensaje «No se han encontrado los controladores de cámara LATTICE»** → El backend no ha podido cargar el entorno de ejecución de la cámara LATTICE con el que está instalado. Todos los comandos `lattice` se ejecutan en el backend; la propia CLI no incluye controladores de cámara. Reinstala Chloros: el instalador de Windows y el archivo .deb instalan el entorno de ejecución junto al backend.
- **«Array connect» / «Array Settings» muestra «FRAMES WILL DROP» o «Reduce ROI to enable»** → El anillo de recepción de la tarjeta de red del host es demasiado pequeño (suele restablecerse a 32 tras una actualización del controlador de la tarjeta de red). Consulte [Configuración y ajuste de la NIC del host](#host-nic-setup--tuning-lattice-arrays): configure `ReceiveBufferLen=256` y `PendingReceives=64`.
- **El equipo se bloquea al reiniciarse o apagarse; a continuación, aparece el error WMI `Invalid class`, la NIC no se activa y faltan las unidades USB** → Un controlador obsoleto del adaptador USB de 10 GbE provoca el error `DRIVER_POWER_STATE_FAILURE` (pantalla azul `0x9F`). Actualiza el controlador del adaptador; consulta [Configuración y ajuste de la NIC del host](#host-nic-setup--tuning-lattice-arrays).
- **En Linux: las capturas pertenecen a root, por lo que no puedes eliminarlas, y el registro del backend no se encuentra en tu `~/.cache`** → El backend se ejecuta como root a través de `chloros-backend.service`. Cambia a `chloros-backend@<user>.service`, que lo ejecuta como tú; consulta [Ejecución del backend como servicio](#running-the-backend-as-a-service-headless). Hasta entonces, el registro del servicio como root se encuentra en `/var/log/chloros` (legible por el grupo `adm`).
- **En Linux: una unidad de backend es `failed` con el mensaje «Ya hay otra instancia en ejecución» (estado de salida 98)** → Otro backend ya ocupa el puerto: solo se puede ejecutar uno a la vez. Si hay dos unidades de backend habilitadas, mantén una: `sudo systemctl disable --now <unit>` para la que no utilices; consulta [Ejecutar el backend como servicio](#running-the-backend-as-a-service-headless). Si solo está habilitada esta unidad, un backend iniciado fuera de systemd (mediante `chloros-backend`, `chloros-cli` o un script del SDK) lo ocupa: detén ese backend y, a continuación, ejecuta `sudo systemctl start <unit>`.
- **Advertencia de swap de Jetson** → Añade swap basado en archivos; la CLI muestra los comandos exactos `fallocate` / `swapon`.
- **«Backend no accesible» desde un comando `lattice`, `project` o `daq`** → No hay ningún backend en ejecución: esos comandos nunca inician uno. Ejecuta `chloros-cli backend status` y consulta [Iniciar el backend](#start-the-backend).
- **«`daq net ...` no está disponible en Chloros»** (Linux y Jetson) → Algunos de los ajustes propios de un DAQ-E están reservados para el servicio MAPIR. La configuración de red, los reinicios y las actualizaciones de firmware se realizan mediante `pool-set-network`, `pool-reboot` y `pool-update-firmware`; para el resto, ponte en contacto con el servicio de asistencia de MAPIR.
- **«Los comandos directos de hardware del DAQ requieren el paquete daq completo»** (Windows) → Utiliza `pool-*`, que controla el mismo sensor a través del backend, o `chloros_sdk.connect_daq_sensor()` desde Python.

---

## Véase también

- [Referencia del SDK de Python](sdk-reference.md): equivalente programático de todos los comandos de la CLI.
- [Sensores de luz DAQ](https://mapir.gitbook.io/chloros/daq-light-sensors) en el manual de Chloros: los sensores, sus límites y rango calibrado, grabación y conexión en red DAQ-E.
- [Cámaras LATTICE](https://mapir.gitbook.io/chloros/lattice-cameras/lattice) en el manual de Chloros: conexión, ajustes de la cámara, modos de captura y matrices.
- Documentación en línea: `https://mapir.gitbook.io/chloros/cli`

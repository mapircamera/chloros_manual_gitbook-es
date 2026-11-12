# Referencia del SDK de Chloros Python

**Versión:**

1.3.0**Generado:**29/07/2026 a las 19:19 ·**Revisado:** 06/11/2026**Paquete:** `chloros-sdk` (PyPI)**Destinatarios:** Optimizado para su uso con modelos de lenguaje grande (LLM); legible para humanos.**Ámbito:** Todas las clases, funciones y ayudantes públicos expuestos por `import chloros_sdk`, con ejemplos que se pueden copiar y pegar y que abarcan el procesamiento de imágenes, el control de una sola cámara, las matrices sincronizadas, sensores DAQ y la automatización de proyectos, además del servidor `chloros-mcp` para asistentes de IA.

Si solo necesitas lo más destacado, ve directamente a:
- [Instalación y guía de inicio rápido](#instalación)
- [Smart-Connect para matrices LATTICE](#smart-connect-for-lattice-cameras)
- [Sesiones de sensores DAQ](#daq-sensor-sessions)
- [Utilidades de cámara](#utilidades-de-cámara)
- [Matrices transitorias, perfiles de alineación e índices](#matrices-transitorias-perfiles-de-alineación--índices)
- [Carpeta de proyectos y recetas](#carpeta-de-proyectos--recetas)
- [Automatización de proyectos](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Fotogramas en directo en tu propio flujo de trabajo](#live-frames-in-your-own-pipeline)
- [Servidor MCP (asistentes de IA)](#mcp-server-ai-assistants)

> **Cambios en la versión 1.3.0** (lo que detecta un script existente de la versión 1.2.0).
>
> **Cambios que un script de la versión 1.2.0 podría tener que gestionar:**
>
> - **Ahora se genera un error si el backend no confirma una publicación.** `CameraSession.disconnect()`, `ArraySession.disconnect()` / `force_disconnect()` en una matriz abierta por la sesión, `DAQSensorSession.disconnect()` / `force_disconnect()` en un sensor que haya abierto, y `ChlorosProject.disconnect_all()` genera el error `ChlorosConnectError` cuando el backend rechaza la liberación o no la confirma a tiempo, ya que el dispositivo puede seguir ocupado. En la versión 1.2.0 se devolvían de forma silenciosa. La salida de un bloque `with` sigue sin generar nunca un error.
> - **Un dispositivo prestado nunca se libera.** `with attach_array(...)` y un `disconnect()` explícito sobre él dejan el conjunto en funcionamiento (en la versión 1.2.0, al salir del bloque se liberaba, incluso cuando la interfaz gráfica de usuario lo tenía en su poder); llama a `force_disconnect()` cuando quieras darlo por terminado. Del mismo modo, un `connect_daq_sensor()` que encuentre el sensor ya en el grupo, incluida la detección inteligente con indicador cero, lo deja abierto en `disconnect()` y al salir de su bloque `with`.
> - **Los formatos PNG y JPEG solo admiten los productos `display` e index.** `capture()` con `format="png"` / `"jpg"` (`ext=".png"` / `".jpg"`) y un nivel `radiance` o `reflectance`, o `processing="all"`, genera un `ValueError` antes de cualquier solicitud; un `raw` / `debayered` de 12 bits es rechazado por el backend. Véase [PNG / JPEG](#png--jpeg-take-the-display-and-index-products-only).
> - **`connect_daq_sensor(sensor_id=...)` abre ese sensor o genera** `ChlorosConnectError` indicando lo que ha encontrado; nunca recurre a otro sensor. Véase [Transporte fijado](#pinned-transport).
> - **`frame_stream()` genera un error en lugar de permanecer en silencio:** `FrameFetchError`, ante un rechazo, repetiría en el siguiente tick (una lectura de reflectancia sin incluir ningún DAQ), tras 60 s sin trama y tras 60 s sin un nuevo disparador. Véase [Un disparador por lectura](#one-trigger-per-read--group_id).
> - **Un tiempo de espera de lectura agotado o una conexión interrumpida genera el error `ChlorosConnectError`** en cada llamada a smart-connect y el error `analyze_array_network()`, donde la versión 1.2.0 generaba el error de tiempo de espera o de conexión subyacente.
> - **`connect_array()` espera 600 s por defecto (`timeout`, 180 s en la versión 1.2.0), y una conexión que se da por perdida se deshace.** La llamada envía su `timeout` al backend, que devuelve cada cámara al estado en el que se encontró y no establece ninguna conexión cuando esta no se completa a tiempo; la llamada genera el error `ChlorosConnectError` (504 de HTTP `CLIENT_DEADLINE_PASSED`). En la versión 1.2.0, una conexión realizada justo después de que las cámaras se encendieran podía prolongarse más allá de la llamada y conectar la matriz después de que esta hubiera generado el error. `connect_all()` espera por cada matriz guardada de la misma manera (120 s en la versión 1.2.0). Véase [Firma de `connect_array()`](#connect_array-signature).
> - **`connect_array()` conserva el sensor completo.** Sin `width` / `height` / `binning`, todas las cámaras se configuran con el sensor completo y la velocidad de fotogramas desciende a la que ofrecen el host y la red; en la versión 1.2.0 ha reducido o agrupado los fotogramas para mantener la velocidad. Véase [Matriz sincronizada](#synchronized-array--arraysession-smart-prep).
> - **Un formato de píxeles Bayer en una cámara M3M (mono) es su profundidad.** Una M3M solo ofrece formatos mono, y los preajustes denominan «Bayer» a los siguientes: en la versión 1.2.0, la cámara rechazaba el formato y mantenía el suyo propio, mientras que `settings.pixel_format` denominaba al formato Bayer. `LatticeCamera(preset=...)`, `apply_preset()`, `set_format()`, un `CameraSettings(pixel_format=...)`, un `connect_camera(preset=...)` y una matrizformato compartido establecen ahora el formato Mono de la misma profundidad en un M3M: `"BayerRG8"` y `high_speed` dan como resultado `Mono8`; `"BayerRG12"`, `high_quality` y `triggered` dan como resultado `Mono12`; `cam.settings.pixel_format` le asigna un nombre. Lo mismo ocurre a la inversa: un M3C nunca se guarda en formato Mono, por lo que `"Mono12"` —procedente de cualquiera de estos, `connect_array(pixel_format=...)`, `set_settings(pixel_format=...)` o el formato guardado de una matriz — da como resultado `BayerRG12` en un M3C, sea cual sea la cámara maestra de la matriz. Un `CameraSettings` que se pase conserva su nombre, por lo que el mismo objeto sigue solicitando a cada cámara esa profundidad dentro de su propia familia. `connect_camera(preset=...)` conserva el formato del preajuste (la versión 1.2.0 configuraba la cámara a 12 bits tras aplicar el preajuste), y una cámara abierta en un preajuste ya no escribe lo que ha capturado en `PRESETS`. Cualquier otro formato que una cámara rechace (uno que no figure en su lista `PixelFormat`, como `RGB8` en un M3M o `BayerRG16` en un firmware que no lo incluya) tampoco recibe un nombre propio: tras la conexión de un `LatticeCamera`, `apply_settings()` o `set_format()`, `cam.settings.pixel_format` y `packed_transport` indican el formato en el que la cámara transmite, y una línea impresa indica el formato que ha rechazado.
> - **`analyze_array_network()` indica qué conexión se aplica:** `pixel_format=None` (antes `"BayerRG8"`) significa los formatos que establece la conexión (Mono12 / BayerRG12), y `packed="auto"` indica la decisión sobre el transporte empaquetado, por lo que los fps que indica pueden diferir de los de la versión 1.2.0'. Un `pixel_format` que pases indica un M3M a esa profundidad. Consulta [Ayuda para el análisis de redes](#network-analysis-helper).
> - **`CameraSession.capture()` devuelve un `CaptureResult`** (sigue siendo una lista de diccionarios de tramas) con `.skipped`, `.failed`, `.daq_files` y `.daq_matches`. Un resultado `ArraySession.capture()` también incluye `.alignment` (que indica si se ha aplicado la distorsión de alineación y, en caso contrario, el motivo) y `.oriented`.
> - **Las capturas de matriz se alinean por defecto, al igual que las de la aplicación.** `ArraySession.capture()`, `continuous()`, `capture_repeated()` y `ArrayHandle.capture()` / `capture_smart()` distorsionan los archivos `debayered`, `display`, `radiance`, `reflectance` y los archivos de índice en el fotograma de la cámara principal y los recorta al área de solapamiento de los miembros (más pequeña que el sensor debido a los desplazamientos de las cámaras, `XMP-mapir:AlignmentJSON` `"applied": true`), de modo que son los mismos archivos que los capturados por la aplicación Chloros de la misma escena; `raw` se mantiene en su formato nativo. Antes, una captura sin `aligned=True` guardaba todos los archivos en su tamaño nativo, sin deformar. `aligned=False` mantiene esa configuración. Los archivos`capture_array` y `start_capture_run` adoptan el mismo valor por defecto. Véase [Métodos de `ArraySession`](#arraysession-methods).
> - **`connect_all()` identifica un backend que rechaza el SDK.** Un backend en ejecución que ha cerrado sesión o que tiene un plan inferior a Copper se notifica en `errors` con su propio motivo (HTTP 401 / 403), y no como «no hay ningún backend de Chloros en ejecución»; además, cada llamada a `connect_all()` vuelve a comprobar el backend, por lo que llamarla tras iniciar sesión funciona en el mismo objeto de proyecto.
> - **Un `output_dir` relativo funciona con un backend local.** Las rutas relativas se convierten en absolutas con respecto al directorio de trabajo de tu script antes de enviarse. En la versión 1.2.0, el backend las resolvía en su propia carpeta de instalación, lo que provocaba un error de «Acceso denegado».
> - **`numpy` es ahora un requisito** del SDK (las llamadas a «live-frame» devuelven matrices de NumPy).
> - **Se rechaza cualquier nombre de nivel de exportación que no sea «uno».** `processing=` y `levels=` aceptan `raw`, `debayered`, `radiance`, `reflectance` o `display` (o un alias como `preview`; `processing=` y también `"all"`). Se rechaza cualquier otro nombre antes de acceder a la cámara (400 HTTP `UNKNOWN_LEVEL`: `capture()` genera `ChlorosConnectError`, `grab()` / `frame_stream()` `FrameFetchError`); la versión 1.2.0 guardó o devolvió `raw` para ello.
> - **Se rechaza «Texture Aware» para capturas LATTICE.** En un proyecto que contenga capturas LATTICE, `configure(debayer="Texture Aware (Slow, Highest Quality)")` genera un `ChlorosConfigurationError`, al igual que `process()` tras un `configure()` que lo solicitó antes de la importación. Un valor de «Texture Aware» que el proyecto había guardado anteriormente se establece en «Standard», con un `UserWarning`. Las capturas LATTICE siempre reciben el demosaico estándar, por lo que la versión 1.2.0 aceptó la configuración y la ignoró; la aplicación Chloros tampoco ofrece esta opción para ellas. Véase [Valores admitidos](#supported-values).
> - **Las cámaras vuelven a su estado original, salvo que se produzca un `keep_settings=True`.** `ArraySession.disconnect()` / `force_disconnect()`, al final de cada función transitoria (`capture_cameras_transient()`, `start_multi_camera_stream()`, `test_array_sync()`, `calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `capture_live_index()`) y la detención de `start_camera_continuous()` devuelven cada cámara al estado en que se encontró: su cableado de disparador y línea de sincronización, su PTP y todos los ajustes de la cámara que se modificaron durante la sesión (exposición y ganancia, el objetivo de AE, los límites y la amortiguación, el balance de blancos, el nivel de negro, el formato de píxeles, el binning y la zona de interés (ROI), los giros y la frecuencia de fotogramas). `keep_settings=True`, en cambio, guarda los ajustes de cada cámara en la propia memoria de la cámara, de modo que se enciende con ellos; `clear_settings=True` restablece esa memoria a los ajustes de fábrica. En cualquier caso, las cámaras quedan en funcionamiento autónomo. Un proyecto configura sus cámaras a partir de sus propios ajustes guardados cuando se conecta a ellas. Véase [Memoria de la cámara](#camera-memory--keep_settings--clear_settings).
> - **Todos los modos guardan un proyecto de la misma manera.** Cualquier cambio en una cámara, una matriz o un sensor de luz conectado a un proyecto —ya sea desde el SDK, la aplicación, la CLI o el servidor MCP— se guarda en el archivo `cameras.json` / `sensors.json`, y al abrir el proyecto en cualquier modo se configuran los dispositivos a partir de él; un `ChlorosProject` asigna su propia carpeta al backend. Antes, solo la aplicación guardaba el proyecto. Véase [Un registro de proyecto, todos los modos](#one-project-record-every-mode).
> - **Los archivos creados a partir de varias cámaras incluyen el nombre de cada una de ellas.** Una pila multibanda, un índice en tiempo real creado a partir de las bandas de varias cámaras y una imagen de captura en matriz `composite/` contienen el número de serie, el modelo y el paquete de calibración de cada cámara de origen en forma de listas (`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceCalBundleSha256s`, …), y el EXIF `SerialNumber` enumera los números de serie separados por comas. Antes no incluían ninguna cámara, o solo el número de serie de la primera cámara. Véase [Archivos creados a partir de varias cámaras](#files-made-from-several-cameras).
> - **Solo un archivo `raw` contiene las etiquetas de calibración de la cámara.** El conjunto compatible con MicaSense (`BlackLevel`, `RadiometricCalibration`, los conjuntos`Band{R,G,B}RadiometricCalibration` por canal, los conjuntos `MicaSenseVignetting*`) y la carga útil `CalibrationJSON` describen los valores sin procesar del sensor, por lo que se incluyen únicamente en el archivo de captura `raw`. Todos los demás archivos —una captura`debayered`, `display`, `radiance`, `reflectance` y los archivos de índice, así como los fotogramas de grabación continua y de— y todos los productos `process()` — lleva la identidad de la cámara (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) y, en una captura, el registro propio del fotograma (`XMP-mapir:FrameJSON`: exposición, ganancia, temperatura, formato de píxeles, hora de captura). En la versión 1.2.0 también incluían el conjunto de datos sin procesar, que una herramienta de aplicación (Pix4D, Metashape) aplicaba a los píxeles que no eran valores sin procesar. El procesamiento determina la calibración de dicho archivo mediante su número de serie y `CalBundleSha256`.
>
> **Otros cambios:**
>
> - **La frecuencia de fotogramas automática de una matriz se ajusta a lo que esta proporciona.** Una frecuencia que pierde más del 1 % de sus disparos se reduce, tras evaluar varias observaciones prolongadas en lugar de una sola, y la frecuencia desde la que se redujo se vuelve a intentar de vez en cuando. Una frecuencia que configures tú mismo no se modifica.
> - **La exposición automática es más estable.** Se mantiene constante con luz constante, se estabiliza en cuestión de segundos tras un cambio, ignora las pequeñas zonas sobreexpuestas y restablece el funcionamiento de una cámara que deja de responder bajo un sol intenso. Véase [Cómo funciona la exposición automática](#how-auto-exposure-behaves).
> - **`import_images(recursive=True)` solo cuenta tus imágenes y grabaciones.** En una carpeta que contenga el archivo `camera_cal/` de un proyecto, esos archivos de calibración ya no se contabilizan como imágenes ni grabaciones, y un proyecto que ya los haya incluido los elimina.
> - **Se aplica `configure(radiometric_output=...)`.** Ahora establece los productos `Export radiance` / `Export reflectance` del proyecto; antes, la configuración de productos propia de cada proyecto prevalecía sobre esta, por lo que `radiance`, `sensor-response` y `all` exportaban tanto la radiancia como la reflectancia.
> - **`test_array_sync()` se ejecuta desde el estado de encendido de las cámaras.** Activa el PTP de las cámaras para la ejecución, espera a que sus relojes se sincronicen y se estabilicen antes de su primer disparo (`ptp_lock`); cada cámara tarda 120 s en sincronizarse, unos 2 minutos desde el encendido (`ptp_settle_s=` lo configura), ya que un reloj recién sincronizado presenta un desfase de hasta ~200 us durante unos 2 minutos — y restablece el PTP de cada cámara tal y como lo encontró. No se vuelve a esperar a una cámara que siga sincronizada de una prueba anterior. Antes dejaba el PTP desactivado en una red de un solo conmutador en la que ningún DAQ-E respondía, y leía el reloj de funcionamiento libre de cada cámara como un reloj PTP que no se sincronizaría («PTP no sincronizado», en cada ejecución).
> - **Una grabación `.daq` indica si su tapa se ha seleccionado o se ha asumido.** DAQ-U, DAQ-M y DAQ-E se suministran sin tapa, y Chloros no puede detectar si lleva una colocada, por lo que un sensor conectado sin una tapa especificada recibe el nombre `sunshine_cosine` por defecto, una *suposición*. Cada registro de Chloros (esquema 1.30) almacena ahora `cap_id_source` junto a `cap_id`: `operator` cuando se ha asignado un nombre a una tapa (incluido `none`), `auto_default` cuando era el valor por defecto. Al procesar un registro cuyo condensador se ha supuesto, con el valor del condensador DAQ del proyecto en «Auto», se muestra un `[daq-dls] WARNING` que nombra el archivo: un DAQ-U sin nada más, bajo la corrección de luz solar, da una lectura aproximadamente 30 veces mayor. **Asigna un nombre al condensador que hayas instalado** al conectarlo (`connect_daq_sensor(cap_id=...)` o `DAQSensorSession.set_cap()`, `chloros-cli daq pool-connect --cap-id` / `daq pool-set-cap`, o la lista desplegable de condensadores de la interfaz gráfica), o configura el condensador DAQ del proyecto, y la advertencia desaparecerá.
> - **Ya no aparecen ventanas de consola pip en Windows.** En un PC con Python instalado, el backend y cada uno de sus procesadores de imágenes ejecutaban `pip` una vez al iniciarse, y cada uno abría una ventana de consola. Ya no lo hacen.
> - **Las grabaciones en directo de `.daq` sobreviven a un corte de corriente.**Una grabación se escribe con un registro y una sincronización completa del disco en cada confirmación, detrás del sensor, por lo que un disco lento retrasa el archivo, pero nunca las lecturas. Una grabación interrumpida a mitad de una confirmación (corte de corriente, cierre forzado) puede dejar un archivo `<name>.daq-journal` junto a ella:**copia los dos juntos.**Chloros revierte el diario la primera vez que lee el archivo, por lo que el registro se abre con todo lo confirmado antes de la interrupción (se pierden aproximadamente los últimos 3 s); en la versión 1.2.0, un corte de corriente en el momento inadecuado podía dejar todo el archivo ilegible («la imagen del disco de la base de datos está malformada»).
> - **`DAQSensorSession.record_start()` en un disco ocupado ya no bloquea el sensor**: las lecturas se ponen en cola y se escriben en un hilo propio, y aquellas que no se han podido escribir se contabilizan en el `status()` del `record_errors` (junto con el `record_last_error`).
> - **`LatticeCamera` directo:**un fotograma sin exposición/ganancia por fragmentos ahora informa de la exposición y ganancia**reales** de la cámara (la alternativa leía un valor almacenado en caché que podía permanecer congelado durante toda una secuencia de exposición automática), y `apply_settings(trigger_source='LineN')` / `set_trigger_source('LineN')` aplican el mismo filtrado de interferencias de entrada de 5 µs que reciben los esclavos por cable de Chloros (sin filtrar, un esclavo disparaba fotogramas sin que se hubiera enviado ningún disparador).
> - **Las cámaras con disparo por software ya no omiten cada segundo disparo.** Una cámara en modo de disparo por software ahora pone en cola un disparo que llega durante su lectura (o Chloros espera a que esté lista), tal y como ya hacían las cámaras sincronizadas por cable.
> - **`discover_daq_sensors()` asigna un nombre a un DAQ-E-S `DAQ-E-S`.** Su `model` era `DAQ-E` para cada sensor Ethernet; aquel que no responda a la comprobación de identidad en unos 2 s sigue apareciendo en la lista, como `DAQ-E`.
> - **Novedad: `chloros-mcp`, un servidor MCP para asistentes de IA.** Claude Desktop, Claude Code, Cursor y otros clientes MCP pueden procesar imágenes (crear un proyecto, importar capturas, elegir ajustes, procesar, generar informes sobre los resultados exportados) y controlar cámaras LATTICE, matrices y sensores de luz DAQ: conectarse, previsualizar, ajustar la exposición y el color, capturar y grabar. Sus herramientas cubren las funciones de este SDK. Se instala junto con el SDK, y el archivo .deb para Linux lo coloca en `/usr/bin`. Véase [Servidor MCP (asistentes de IA)](#mcp-server-ai-assistants).
> - **Novedad: utilidades para cámaras**, los comandos `chloros-cli lattice` tal y como los invoca el SDK: perfil de color, saturación/contraste, luces altas, balance de blancos (de un solo paso, automático, relaciones fijas), preajustes, modelos de filtro, estado y actualización del firmware, alimentación (sondeo, suspensión, activación), calibración del objetivo, adaptadores de red (comprobación, estimación, corrección), corrección de viñeteado en archivos, lectura DLS y sondeo del host. Véase [Utilidades de la cámara](#camera-utilities).
> - **Novedad: el resto de la CLI como llamadas SDK**, de modo que la CLI, el SDK y `chloros-mcp` ofrecen las mismas funciones (la CLI conserva únicamente el inicio de sesión, el aprovisionamiento de un DAQ-E y los servidores DAQ directos, por motivos de seguridad): cámaras abiertas con un solo comando (`start_camera_continuous()`, `start_multi_camera_stream()`, `capture_cameras_transient()`, `test_array_sync()`, `detect_array_master()`, `analyze_array_models()`), archivos de perfiles de alineación (`calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `alignment_profile_info()`, `reorder_alignment_profile()`), índices (`index_catalog()`, `compute_index()`, `capture_live_index()`), el «grandmaster» de PTP (`time_sync_status()` y demás), la carpeta del proyecto (`project_folder()`, `set_project_folder()`, `reset_project_folder()`) y las recetas (`run_project_recipe()`, el propio ejecutor de la CLI). Para los sensores de luz: `connect_daq_sensor(cap_id=...)`, `DAQSensorSession.set_cap()`, `daq_bindings()` / `set_daq_bindings()`, `daq_band_presets()` y `light_sensor_reflectance()` (reflectancia ambiental y de objetos, e índices de dos sensores); para una cámara: `CameraSession.status()` y `capture_repeated()`; para una matriz: `ArraySession.capability()` y `set_member_settings()` (exposición, ganancia, etc., de un miembro). Véase [Matrices transitorias, perfiles de alineación e índices](#transient-arrays-alignment-profiles--indices), [Carpeta de proyectos y recetas](#project-folder--recipes) y [Sesiones de sensores DAQ](#daq-sensor-sessions).
> - **Productos de índices LATTICE mejorados** (NDVIy otros índices) de `process()`, en todos los modos de procesamiento.
> - **Cámaras OCN de LATTICE: productos de radiancia, reflectancia e índices mejorados bajo la luz solar.** Se aplica una corrección de flota tras la separación de bandas —en tiempo real, en las capturas y en `process()`—; las cámaras RGN no sufren cambios. Véase [Salida radiométrica](#radiometric-output-lattice-multispectral-pipeline).
> - **Cada captura guarda la lectura del DAQ que ha utilizado, al igual que lo hace la interfaz gráfica de usuario.** `CameraSession.capture()` y `ArraySession.capture()` / `continuous()` escriben el `.daq` y su `.match.json` cada vez que se vincula o se agrupa un DAQ, incluida una captura solo en formato sin procesar; en la versión 1.2.0, una captura sin procesar o sin debayering se guardaba únicamente con `force_daq=True`. `daq=False` lo desactiva y `daq_tolerance_ms=` establece la tolerancia de coincidencia; ambos resultados indican lo que se guardó en `.daq_files` y `.daq_matches`. Ahora, una captura espera a la siguiente lectura del DAQ para poder combinar las lecturas de ambos lados de la exposición. Véase [La lectura del DAQ que guarda una captura](#the-daq-reading-una-captura-guarda--archivos-daq-y-coincidencias-daq).
> - **`ArraySession` tiene fotogramas en tiempo real y una captura continua del lado del servidor.** `grab()` y `frame_stream()` devuelven fotogramas numpy de un disparador (la misma lectura que `ArrayHandle.grab()` / `frame_stream()` de la API del proyecto), y `continuous()` guarda cada grupo sincronizado en el backend y devuelve un [`ContinuousHandle`](#continuoushandle). Véase [Fotogramas en directo en tu propio flujo de trabajo](#live-frames-in-your-own-pipeline).
> - **Un array `grab()` está vinculado a un disparador** (`ArrayHandle.grab()` y el nuevo `ArraySession.grab()`). En la versión 1.2.0, 12 de las 20cámara `ArrayHandle.grab()` se repartían entre dos disparadores. Ahora, cada fotograma lleva `group_id`, y una matriz `frame_stream()` genera un resultado por cada disparador. Véase [Un disparador por lectura](#one-trigger-per-read--group_id).
> - **Fotogramas por banda y alineados:** `grab()` / `frame_stream()` toman `band=`, `align=`, `bit_depth=` y `refl_max=` en los identificadores de cámara y matriz. Véase [Exportación de una sola banda](#exporting-a-single-band--band).
> - **Nuevos métodos y argumentos de `ArraySession`:** `set_target_fps()`, `force_disconnect()`, `capture(after_seq=..., orient=...)` (junto con `CaptureResult.group_seq` y `.failed`), `continuous()`, `connect_array(packed=...)`; `capture_repeated()` encadena grupos consecutivos y se detiene cuando se completa la unidad.
> - **`ArraySession.record()` registra sin la interfaz gráfica abierta.** Con la vista combinada de la interfaz gráfica cerrada, el backend representa la vista por sí mismo: elNDVI o el índice que se pasa como `expr=`. `RecorderHandle.stop()` genera un error `ChlorosConnectError` para una grabación que ha fallado, en la que no se ha escrito ningún fotograma, y mantiene el resultado de la detención en `.result` (dejar un bloque `with` no genera una excepción: comprueba `rec.result`). Véase [`RecorderHandle`](#recorderhandle).
> - **Nuevos métodos `DAQSensorSession`:** `enable_ptp()`, `force_disconnect()` y, para la configuración propia de un DAQ-E, `device_info()`, `set_network()`, `set_auto_stream()`, `reboot()` y `update_firmware()`. `record_stop()` devuelve `warning` y `gaps`. Un reinicio que haya cerrado una grabación la nombra, en su respuesta o, cuando el reinicio falla, en el `ChlorosConnectError` (`recording_closed`; véase [Configuración del dispositivo DAQ-E](#daq-e-device-settings)). Véase [Métodos de `DAQSensorSession`](#daqsensorsession-methods).
> - **Un DAQ-E agrupado que deja de enviar datos es reconectado por el backend**, con la configuración que tenía, y una grabación en él continúa en el mismo `.daq` con el intervalo marcado en el archivo; `record_stop()` devuelve las interrupciones como `gaps`. En la versión 1.2.0, el sensor permanecía inactivo hasta que se desconectaba y se volvía a conectar, lo que ponía fin a la grabación. Consulte la referencia de la CLI [Un DAQ-E que deja de funcionar](cli-reference.md#a-daq-e-that-stops-delivering).
> - **`open_project()` funciona a partir de un SDK instalado.** El instalador (ely PyPI) ahora incluye los cargadores de proyectos que necesita; antes, `open_project()` generaba `ImportError` al nombrar `project_cameras` / `project_sensors` en cualquier SDK instalado con pip. `connect_all()` conecta las cámaras, matrices y sensores de luz guardados a través de un backend en ejecución (los sensores de luz a través de su grupo DAQ), y un dispositivo guardado que no tiene forma de localizar aparece en la lista de `errors` indicando el motivo, en lugar de omitirse sin explicación alguna. Una matriz guardada que ya está activa en el backend (la de la interfaz gráfica de usuario, o las de `chloros-cli project connect`) se conecta y se deja en funcionamiento, mientras que antes se notificaba como un error (`Camera <sn> is already in array <id>`). Véase [Métodos de `ChlorosProject`](#chlorosproject-methods).
> - **`ChlorosLocal()`, `process_folder()` y `process_lattice_capture()` localizan Chloros en Windows.** Buscan dónde coloca el instalador el backend (`<install folder>\backend\`, `C:\Program Files\Chloros\backend\` por defecto, una carpeta personalizada según el registro del propio instalador), y se conectan a un backend que ya se está ejecutando sin buscarlo en absoluto. En la versión 1.2.0 mostraban el mensaje «No se ha encontrado el backend de Chloros» en un equipo instalado, a menos que se pasara `backend_exe=`.
> - **Se notifica inmediatamente si un backend se bloquea al iniciarse.** Cuando `ChlorosLocal()` o una llamada de conexión inteligente inicia un backend que se cierra antes de responder, la llamada genera un error en cuestión de segundos: `ChlorosBackendError` desde `ChlorosLocal` y `ChlorosConnectError` desde las llamadas smart-connect, con el código de salida, el comando que ejecutó y las últimas líneas que imprimió el backend. La versión 1.2.0 esperaba a que transcurrieran los 60 s del tiempo de espera de inicio y no indicaba el motivo. Véase [Inicio automático del backend](#backend-auto-start).
> - **El backend sigue ejecutándose entre scripts.** Al salir de un bloque `with ChlorosLocal(...)`, `process_folder()` / `process_lattice_capture()` al volver y `chloros-mcp` al salir ya no detienen un backend iniciado por el SDK; el siguiente script lo reutiliza. Se detiene en `logout()` (a menos que la aplicación Chloros esté abierta o haya una ejecución en curso), en `shutdown_backend()`, o por sí solo tras 30 minutos sin que nada lo esté utilizando (`CHLOROS_BACKEND_IDLE_MIN` en el entorno del backend cambia esto; `0`: nunca).
> - **Se solicita al backend iniciado por el SDK que se apague, no que se elimine.** `shutdown_backend()` permite que el backend libere sus cámaras y sensores de luz y finalice cualquier grabación; solo se cierra si sigue en ejecución 10 s después. La versión 1.2.0 lo cerraba de inmediato (en Windows, un cierre forzado), lo que podía dejar sin respuesta un sensor de luz DAQ conectado a ese backend. Un backend que el SDK encontrara en ejecución se deja seguir funcionando.
> - **En Linux, el inicio automático nunca ejecuta un `backend_server.py` desde la carpeta actual.** Si no había ningún backend instalado, un script iniciado en una carpeta que contenía uno lo ejecutaba; ahora solo se inicia el backend instalado (o, en el caso de `ChlorosLocal`, el que `backend_exe=` especifique).
> - **`configure(indices=...)`**añade cada preajuste al proyecto para las bandas de su cámara, incluye un nuevo proyecto y muestra una advertencia (`UserWarning`) si hay un nombre que no puede añadir.**`import_images()`** informa de las grabaciones de `.daq` como `scan_count` y ya no muestra avisos sobre una carpeta de captura de LATTICE.
> - **`read_image_audit_tags()` funciona tras una instalación con pip.** Lee los metadatos de la imagen por sí mismo; en la versión 1.2.0 devolvía `{}` sin los componentes internos de la instalación de escritorio en la ruta de Python.
> - **`capture()`, justo después de `connect_array()`**, espera hasta 20 s por el primer grupo de fotogramas de la matriz, en lugar de fallar con el código HTTP503 «La matriz aún se está calentando».
> - **`status()` informa de un bloqueo.** El nuevo campo `stalled` es «True» mientras se ejecuta el bucle de captura, pero no publica nada, y `actual_fps` pasa entonces a ser 0. En la versión 1.2.0, `actual_fps` mantenía su último valor durante un bloqueo, por lo que una matriz bloqueada mostraba `actual_fps > 0` mientras que las capturas fallaban con el error 503 «No hay ningún grupo de fotogramas sincronizados recientes».
> - **Menos conjuntos de fotogramas incompletos y errores 503 en la captura:** la entrada de disparo de cada cámara esclava ahora está libre de interferencias.
> - **Transporte empaquetado de 12 bits** (`connect_array(packed="auto")`) aumenta la velocidad de fotogramas de la matriz de sensor completo en aproximadamente un tercio. Véase [Transporte empaquetado de 12 bits](#packed-12-bit-transport-packed).
> - **Exposición de la matriz en una región de interés (ROI):** una ROI pequeña ya no limita la exposición de un miembro al periodo de funcionamiento libre del sensor (4,9 ms a 768 × 304). Véase [Exposición en los miembros de la matriz](#exposure-on-array-members).
> - **La exposición automática ya no reduce la frecuencia de fotogramas automática de una matriz.** Un conjunto sincronizado por cable funciona a la velocidad que permiten su red y la lectura de las cámaras; la exposición automática ocupa el periodo que proporciona esa velocidad y, a continuación, aumenta la ganancia. Una exposición manual más larga que el periodo ralentiza el disparador para adaptarse a ella, en lugar de rechazarla. Véase [Exposición en los miembros del conjunto](#exposure-on-array-members).
> - **`read_nodes()`**añade `frame_values` y `notes` mientras se ejecuta la exposición automática (véase [`read_nodes()` bajo exposición automática](#read_nodes-under-auto-exposure--frame_values-and-notes)). Cuando la cámara rechaza una exposición o una ganancia,**`set_settings()`** indica el rango que acepta y restablece el modo automático que se había desactivado para ese valor; `set_settings(target_brightness_auto=True)` es nuevo.
> - **`orient=`** en `ArraySession.capture()` / `capture_fastest()` / `continuous()` / `burst()` y `ArrayHandle.capture()` / `capture_smart()` gira los productos según la orientación de la cámara. Consulta [Métodos de `ArraySession`](#arraysession-methods).
> - **`DLS()` requiere el `mac_address` propio del sensor.** Ya no hay una MAC predeterminada (era la dirección de una unidad); `DLS()` sin una genera `DLSError`. Consulta [magnitudes de banda DAQ-M](#daq-m-band-magnitudes--dlsband_magnitudes).
> - **El instalador de Windows** ofrece las opciones «Habilitar rutas largas de Windows» y «Excluir el backend de Chloros del análisis de Microsoft Defender», ambas marcadas por defecto. Véase [Paso 1](#step-1--install-the-chloros-platform-package).
> - **El paquete .deb para Linux** instala el Python SDK para el sistema `python3` (antes, solo colocaba el archivo wheel en el disco), aumenta los búferes de recepción de sockets del kernel que necesitan las transmisiones de las cámaras y añade un servicio de backend que se ejecuta como tu usuario. El paquete para Jetson es `chloros_1.3.0_arm64_jp6.deb`, para JetPack 6, con PyTorch para CUDA de JetPack. Consulta el [Paso 1](#step-1--install-the-chloros-platform-package) y el [Paso 2](#linux-deb).

---

## La arquitectura en 60 segundos

El SDK es una capa ligera de Python sobre el backend de Chloros (el mismo servidor Flask que utilizan la interfaz gráfica de usuario de escritorio y la CLI). Para la automatización, se importa `chloros_sdk` y se invocan métodos de alto nivel; en segundo plano, cada llamada se convierte en una solicitud HTTP al backend local en el puerto 5000 — `http://127.0.0.1:5000/api/...` (deliberadamente no `localhost`, que en Windows se resuelve primero como `::1` y cuesta unos 2 s por solicitud frente a un backend solo IPv4). El backend gestiona el conjunto de hardware —cámaras, sensores DAQ, perfiles de alineación, búferes de fotogramas—, de modo que los scripts del SDK pueden coexistir con la interfaz gráfica de usuario sin competir por los puertos serie ni por el ancho de banda de la tarjeta de red.

Hay tres interfaces que utilizarás:

1. **`ChlorosLocal` + funciones libres** (`process_folder`, `process_lattice_capture`): canal de procesamiento de imágenes. Ejecuta una carpeta completa a través de los procesos de calibración, eliminación de mosaico y exportación de índices con una sola llamada a Python.
2. **Controladores Smart-connect** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — Abre una sesión de backend persistente para hardware en tiempo real. El mismo flujo «smart-prep» que en la interfaz gráfica de usuario: sonda de red, selección automática de nivel, PTP, inicialización de AE y configuración del disparador GPIO.
3. **`ChlorosProject` / `open_project`** — Carga un proyecto guardado (carpeta con `cameras.json` + `sensors.json` + `project.json`), conecta sus cámaras, matrices y sensores de luz de una sola vez, y controla las capturas mediante identificadores con nombre.

Las superficies 1 y 2 **inician automáticamente un backend local** si aún no hay ninguno en escucha (el mismo binario incluido que ejecutan la GUI y la CLI), de modo que un script básico funciona desde un shell nuevo sin que tengas que iniciar primero un backend: `ChlorosLocal` (y `process_folder()` / `process_lattice_capture()`), `connect_camera()`, `connect_array()`, `connect_daq_sensor()`, `discover_lattice_cameras()` y `discover_daq_sensors()`. `list_cameras()`, `list_arrays()`, `list_daq_sensors()`, `attach_array()` y `analyze_array_network()` nunca se inician por sí mismos: necesitan un backend en ejecución. Indica `auto_start_backend=False` para desactivar esta opción (por ejemplo, cuando se apunta a un backend remoto, que nunca se inicia). Véase [Inicio automático del backend](#backend-auto-start). Surface 3 se comporta de forma diferente: `open_project()` no admite el parámetro `auto_start_backend`, y `connect_all()` nunca inicia un backend : comprueba `http://127.0.0.1:5000` una vez y, si no obtiene respuesta, recurre al control directo del dispositivo (sin backend) `lattice_sdk`. Para ello se necesita el tiempo de ejecución del SDK de Arena y OpenCV (`cv2`) importable en tu Python, algo que el paquete «wheel» del SDK no instala; sin ellos, cada cámara y matriz guardadas se notifican como un error, indicando el motivo. Los sensores de luz de un proyecto no tienen una ruta sin backend: si no hay ningún backend en ejecución, cada sensor guardado se notifica como un error, indicando el motivo (véase [`SensorHandle`](#sensorhandle)). Solo `proj.process()` y `stream(..., overlays=True)` construyen de forma diferida un `ChlorosLocal()` (que se inicia automáticamente).

Los tres son auth: ejecuta `chloros-cli login` una vez en el equipo o inicia sesión a través de la interfaz gráfica de usuario del escritorio. Lo que genera una llamada sin una sesión válida depende de la interfaz (véase [Excepciones](#exceptions)): `ChlorosAuthenticationError` desde `ChlorosLocal` (y `process_folder()` / `process_lattice_capture()`), `ChlorosConnectError` de las llamadas-connect y sus sesiones, `status: "error"` con un `error_code` de `analyze_array_network()`, y una entrada bajo `errors` en el informe `connect_all()`.

Requisitos:
- Python 3.7+ (según indica el paquete; desarrollado y probado en la versión 3.10)
- Chloros Desktop instalado localmente (el binario del backend se incluye en el instalador)
- Inicio de sesión activo en Chloros+ en **cualquier plan de pago de Chloros+ (Copper o superior)**; el plan gratuito**Iron**no tiene acceso a SDK ni a CLI. Esto se aplica**por parte del servidor**:** toda solicitud marcada con SDK o CLI debe incluir tanto una sesión activa como un plan de pago. Un plan inferior a Copper recibe el código de error `403` junto con `error_code: PLAN_UPGRADE_REQUIRED` (`ChlorosLicenseError` a partir de `ChlorosLocal`, `ChlorosConnectError` en las llamadas de Smart Connect). Un usuario que haya cerrado sesión recibe `401` / `AUTH_REQUIRED` en su lugar (`ChlorosAuthenticationError` de `ChlorosLocal`, `ChlorosConnectError` de las llamadas de Smart-Connect): ambos son distintos porque al volver a ejecutar `chloros-cli login` corrige el error `401`, pero no puede corregir el `403`, que requiere una actualización del plan.
- Se admite el uso sin conexión durante el periodo de gracia del plan: el nivel se lee desde la caché de validación del servidor (5 min) o desde la caché de licencias firmadas y vinculadas al equipo (30 días para los planes mensuales, hasta el vencimiento de la suscripción para los anuales). Cuando finaliza ese periodo de gracia, el plan pasa a ser gratuito y el acceso al SDK y a la CLI se interrumpe hasta que el equipo pueda conectarse al servidor al menos una vez. El código de error `chloros-cli status` (`GET /api/license-status`) sigue siendo accesible en el nivel gratuito, por lo que el motivo es visible.
- Windows 10/11 de 64 bits, **Ubuntu 22.04 LTS o posterior**, o Jetson (JetPack 6). Ubuntu 20.04**no** es compatible: las dependencias de `.deb` se derivan de aquello con lo que se vincula el backend, incluido `libc6 (>= 2.35)`, y Focal incluye glibc 2.31.

---

## Instalación

El Python SDK es una capa ligera de Python sobre el backend de Chloros. A menos que utilices un backend en otra máquina (véase [Modo de backend remoto](#remote-backend-mode-pip-only-host-via-tunnel)), necesitas tener **el paquete de escritorio de Chloros instalado localmente** (instalador para Windows o Linux `.deb`), que incluye los sensores de luz DAQ. El paquete proporciona el binario del backend, el entorno de ejecución del SDK de Arena para cámaras LATTICE y los paquetes de calibración.

Últimas descargas: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### Paso 1 — Instalar el paquete de la plataforma Chloros

#### Windows (.exe)

1. Descarga `Chloros Installer 1.3.0.exe` desde la página de descargas.
2. Ejecuta el instalador y sigue las instrucciones del asistente. La ruta de instalación predeterminada es `C:\Program Files\Chloros\`.
3. Inicia Chloros al menos una vez e inicia sesión con tu cuenta de Chloros+.

El instalador permite que el backend (`chloros-backend.exe`, nada más) a través del cortafuegos de Windows en todos los perfiles de red: entrada UDP, para las transmisiones de las cámaras, PTP, mDNS y la transmisión del DAQ-E, y entrada TCP solo desde la subred local, para la conexión que establece el DAQ-E con el ordenador durante una actualización de firmware ([`update_firmware()`](#daq-e-device-settings)). El desinstalador elimina ambas reglas. La API HTTP de HTTP del backend permanece en `127.0.0.1` en cualquier caso.

El grupo **Configuración del sistema** del asistente tiene dos opciones, ambas marcadas por defecto:

| Opción | Qué hace | Desactivar la instalación silenciosa |
| --- | --- | --- |
| **Habilitar rutas largas de Windows** | Permite que las carpetas de proyectos se encuentren en un nivel más profundo que el límite de 260 caracteres de Windows. Si las rutas largas están desactivadas (la configuración predeterminada de Windows), una carpeta de proyecto que se encuentre en un nivel demasiado profundo para sus productos se rechaza antes de que se procese nada; consulta [Solución de problemas](#troubleshooting-pointers). | `/MERGETASKS="!longpaths"` |
| **Excluir el backend de Chloros del análisis de Microsoft Defender** | Impide que Defender realice un análisis en tiempo realde cada archivo que escribe una captura de cámara. A altas velocidades de captura, el análisis consume aproximadamente el 30 % de la velocidad: medido en ROI 512 en una matriz de 9 cámaras, la captura RAW continua alcanzó 52,7 capturas/s con el análisis y 77,7 capturas/s con el backend excluido. | `/MERGETASKS="!defenderexclusion"` |

Una captura continua informa de la carga de Defender como `defender_cores` (véase [`ContinuousHandle`](#continuoushandle)).

#### Linux amd64 (.deb)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

Instálalo con `apt install ./…`, no con `dpkg -i`: apt también instala los requisitos del Python SDK, que el paquete recomienda en lugar de exigir (véase [Paso 2](#linux-deb)).

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

Para Jetson Orin / Orin NX / Orin Nano con JetPack 6 (L4T R36). Su PyTorch es la compilación CUDA de JetPack 6, por lo que el procesamiento de la GPU se ejecuta en la GPU del Orin; utiliza las bibliotecas CUDA y cuDNN que instala JetPack 6 (`sudo apt install nvidia-jetpack` en una imagen que carece de ellas).

#### Linux: búferes de recepción y el servicio de fondo

Ambos paquetes .deb aumentan a 32 MB los búferes de recepción de sockets del kernel que necesitan las transmisiones de las cámaras LATTICE (`/etc/sysctl.d/60-chloros-gige.conf`). Con la configuración predeterminada del kernel, los paquetes de la cámara se descartan en el núcleo y el array pierde fotogramas y frecuencia de fotogramas; `chloros-cli lattice network` avisa cuando los búferes están bajos. Consulta la referencia de la CLI, en [Hosts de Linux — búferes de recepción de sockets](cli-reference.md#linux-hosts--socket-receive-buffers).

En una máquina sin interfaz gráfica, ejecuta el backend como servicio **con tu cuenta de usuario**, de modo que un script detecte que hay uno en ejecución (`connect_all()` nunca inicia un backend):

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
```

El backend se ejecuta entonces como tú: las capturas que escribe son tuyas y utiliza tu `chloros-cli login`. El `chloros-backend.service` de root sigue instalado para las máquinas que lo tienen habilitado, pero todos los archivos que escribe un backend de root pertenecen a root. Consulta la sección [Ejecución del backend como servicio](cli-reference.md#running-the-backend-as-a-service-headless) de la referencia de la CLI.

Un backend ejecutado por el servicio es compartido y nunca lo detiene un cliente: los scripts del SDK y `chloros-cli` lo utilizan tal y como lo encuentran (`ChlorosLocal.shutdown_backend()` solo detiene un backend que `ChlorosLocal` haya iniciado él mismo), y `GET /api/status` denomina la unidad como `service`. Reinícialo con `sudo systemctl restart chloros-backend@$USER.service`. La actualización del paquete detiene los servicios de backend durante la instalación, por lo que un script en ejecución pierde sus cámaras y sensores, y luego vuelve a iniciar todos los que están habilitados; al desinstalar el paquete, estos se detienen y se deshabilitan los que estaban habilitados. Solo se ejecuta un backend a la vez, así que habilita una unidad: una segunda unidad se niega a iniciarse y permanece como `failed` (estado de salida 98, no se reinicia), y su `systemctl status` explica el motivo. Para cambiar de unidad, desactiva la que no utilices.

### Paso 2 — Instalar el Python SDK

**El instalador de Chloros incluye un archivo «wheel» del SDK correspondiente.** Cada instalador de Windows y cada paquete .deb de Linux coloca en el disco un archivo `chloros_sdk-X.Y.Z-py3-none-any.whl` que coincide exactamente con la versión de la interfaz gráfica de usuario (GUI), la interfaz de línea de comandos (CLI) y el backend. No es necesario estar pendiente de PyPI para mantener la sincronización.

#### Windows

El instalador ejecuta automáticamente `pip install` con el archivo «wheel» incluido utilizando el Python del sistema (se prefiere el lanzador `py.exe`; si no está disponible, se recurre a `python -m pip`). No es necesario realizar ninguna acción: `import chloros_sdk` funciona en tu entorno de Python tras una instalación correcta. Si no hay Python instalado en el equipo, el instalador omite este paso de forma silenciosa y la GUI y la CLI siguen funcionando.

#### Linux (.deb)

Ambos paquetes .deb (x86_64 y Jetson) instalan el SDK para el sistema `python3`, por lo que `import chloros_sdk` funciona tras la instalación, sin necesidad de pip ni de conexión a red. Los paquetes (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` y los cargadores de proyectos) se instalan en `/usr/lib/python3/dist-packages`, y el comando `chloros-mcp` del SDK (el [servidor MCP](#mcp-server-ai-assistants)) en `/usr/bin`; todos ellos pertenecen al paquete: una actualización los sustituye y, al eliminar el paquete, se eliminan también.

Los requisitos propios del SDK, `python3-numpy` y `python3-requests`, son **recomendados, no obligatorios** (la interfaz gráfica de usuario, la CLI y el backend son autónomos): `sudo apt install ./chloros_<version>_amd64.deb` (`…_arm64_jp6.deb` en un Jetson) los instala, mientras que `dpkg -i` no lo hace. El resultado de la instalación indica la versión del SDK que importa `python3`, cualquier requisito que falte, y cualquier `chloros_sdk` que se encuentre en otra ubicación de la ruta de `python3` y que sobrescriba la copia del paquete (como uno instalado anteriormente con `pip install --user`).

En un entorno virtualenv, que no tiene acceso a los paquetes del sistema, instala el archivo «wheel» incluido en él:

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Para implementaciones de Jetson aisladas (air-gapped), este proceso se realiza totalmente sin conexión: el SDK y el archivo «wheel» ya se encuentran en el disco.

#### PyPI público

Para hosts que solo utilizan pip (sin el paquete de escritorio de Chloros instalado; en este caso, el SDK controla un backend en otra máquina; véase [Modo de backend remoto](#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

PyPI contiene la última versión estable. Mantén el SDK en la misma versión que el backend de Chloros que controla (consulta [Versiones y compatibilidad](#versioning--compatibility)). El programa auxiliar DAQ-M `DLS` también necesita `bleak`: `pip install "chloros-sdk[camera]"` (o `pip install bleak`).

#### Verificar

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Se requiere una suscripción a Chloros+.** Todas las llamadas al SDK requieren un inicio de sesión activo en Chloros+. Ejecuta `chloros-cli login user@example.com 'YourPassword'` una vez por máquina; las credenciales se almacenan en caché en `~/.chloros/`.

### ¿Necesito el paquete de escritorio?

El paquete pip por sí solo **no** es suficiente para la mayoría de los flujos de trabajo. Esto es lo que necesita cada interfaz del SDK:

| Interfaz del SDK | ¿Necesita el paquete de escritorio? | ¿Por qué? |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Sí**(para iniciar uno)**/ No** (un backend al que ya se pueda acceder) | Utilizar un backend en ejecución (el de la aplicación de escritorio o otro script iniciado); de lo contrario, iniciar automáticamente el instalado: `/usr/lib/chloros/chloros-backend` (Linux) o `<install folder>\backend\chloros-backend.exe` (en Windows; `C:\Program Files\Chloros\backend\` por defecto; se lee una carpeta de instalación personalizada a partir del registro del instalador). Pasa `backend_exe=` para asignarle un nombre tú mismo. |
| `chloros-mcp` ([servidor MCP](#mcp-server-ai-assistants)) | **Sí**(para iniciar uno)**/ No** (ya hay un backend accesible) | Basado en `ChlorosLocal`, por lo que es igual que la fila anterior. |
| `read_image_audit_tags` | **No** | Lee los metadatos propios de la imagen procesada (solo biblioteca estándar de Python). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Sí**(local)**/ No**(remoto) | Clientes HTTP puros a través del backend. Backend local → se requiere un paquete de escritorio. Backend remoto → `backend_url=`**a través de un túnel** (véase Modo de backend remoto — los backends incluidos solo se vinculan al bucle de retorno). |
| `ChlorosProject` / `open_project` | **Sí** | Gestiona los proyectos guardados a través del backend. |
| Clases LATTICE directas (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Sí** | Requiere el entorno de ejecución del SDK de Arena, que se incluye en el paquete de escritorio, y OpenCV (`cv2`) en tu Python, que ni el paquete de escritorio ni el SDK instalan (`pip install opencv-python`). De lo contrario, `CAMERA_AVAILABLE` es `False` al importarlo. `DLS` también necesita `bleak`. |

### Modo de backend remoto (host solo con pip, a través de un túnel)

> **No se puede acceder al backend instalado a través de la LAN.** Escucha
> solo en el bucle de retorno (`127.0.0.1` / `::1`), por lo que `backend_url="http://<lan-ip>:5000"`
> **no puede acceder a un Chloros instalado**. Para controlar un backend en otra
> máquina, reenvía tú mismo su puerto de bucle cerrado y configura el SDK para que apunte al
> túnel:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"     # the tunnel endpoint
serials = ["213800234", "214000533"]   # master first

# auto_start_backend=False: the tunnel URL looks local, so without it a tunnel
# that is down would start this machine's own backend, if one is installed.
arr = chloros_sdk.connect_array(serials, backend_url=BACKEND,
                                auto_start_backend=False)
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                     backend_url=BACKEND,
                                     auto_start_backend=False)

# The backend opens every path on ITS machine: pass absolute paths valid there.
# (A relative path would be made absolute against this machine's working folder.)
arr.capture("/home/chloros/captures/run1", processing="raw")
```

Los hosts sin interfaz gráfica / de CI / robóticos pueden mantener una máquina con la instalación completa del escritorio como «servidor Chloros» y `pip install chloros-sdk` en todas partes; sin embargo, el transporte entre ellos es el túnel configurado por el usuario mencionado anteriormente, no una URL directa de la LAN.

> **`ChlorosLocal` en un host que solo dispone de pip.** `ChlorosLocal(backend_url=BACKEND)` se conecta a un backend al que puede acceder sin necesidad de una instalación de escritorio: solo busca un binario de backend local cuando tiene que INICIAR uno (si no hay respuesta y aparece `auto_start_backend=True`). Hasta la versión 1.2.0, buscaba primero el binario y generaba el error `ChlorosBackendError` («No se ha encontrado el backend de Chloros…») en un host que solo tiene pip, incluso si había un backend accesible. Las carpetas que se indican (`process_folder`, `import_images`) se comprueban en esta máquina y son leídas por el backend, por lo que, con un backend remoto, deben tener el mismo nombre en ambas, como por ejemplo una unidad compartida.

### Flujo de trabajo solo DAQ (host solo pip)

`pip install chloros-sdk` por sí solo no puede controlar un sensor DAQ: un sensor DAQ siempre se controla a través de un backend de Chloros con `connect_daq_sensor()`:

- **Sensor en esta máquina:** instala aquí el paquete de escritorio de Chloros. `connect_daq_sensor()` inicia su backend si no hay ninguno en ejecución.
- **Sensor en otra máquina:** esa máquina necesita el paquete de escritorio; acceda a su backend a través del túnel en [Modo de backend remoto](#remote-backend-mode-pip-only-host-via-tunnel). El backend abre el sensor en su propia máquina, por lo que un DAQ-U debe estar conectado a esa máquina, un DAQ-M debe estar dentro del alcance de Bluetooth de la misma y un DAQ-E debe estar en una red a la que pueda acceder.

```python
import chloros_sdk

# http://127.0.0.1:5000 is the local desktop install's backend, or the tunnel.
with chloros_sdk.connect_daq_sensor() as daq:   # smart-detect USB / BLE / ETH
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

En cualquier caso, el backend aplica el plan «Chloros+»: inicia sesión en la máquina que ejecuta el backend (`chloros-cli login` o la aplicación de escritorio) con cualquier nivel de suscripción de pago (Copper o superior). Tanto un backend sin sesión como un plan inferior a Copper generan el código de error `ChlorosConnectError`, que incluye el motivo del backend (`AUTH_REQUIRED` / `PLAN_UPGRADE_REQUIRED`).

---

## Guía de inicio rápido

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
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

## Índice de API de nivel superior

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
chloros_sdk.CaptureResult                         # list subclass returned by CameraSession.capture / ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job
chloros_sdk.ContinuousHandle                      # handle for an array continuous() capture
chloros_sdk.GrabResult                            # dict returned by grab() / yielded by frame_stream()
chloros_sdk.FrameFetchError                       # raised by grab()/frame_stream() on a refusal

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()
chloros_sdk.daq_bindings() / set_daq_bindings(cameras=, arrays=)   # which DAQ feeds which camera
chloros_sdk.daq_band_presets(filter_code=None)    # band sets for Ambient + Object reflectance
chloros_sdk.light_sensor_reflectance(ambient_id, object_id, ...)   # two DAQs -> reflectance + indices
chloros_sdk.time_sync_status() / time_sync_peers() / time_sync_cameras()   # PTP grandmaster
chloros_sdk.restart_time_sync() / set_time_sync_priority(priority1=, priority2=)

# === Camera utilities (the CLI's lattice commands; by serial) ===
chloros_sdk.camera_color_profile(serial) / set_camera_color_profile(serial, ...)
chloros_sdk.camera_color_adjust(serial) / set_camera_color_adjust(serial, ...)
chloros_sdk.camera_highlights(serial) / set_camera_highlights(serial, value)
chloros_sdk.camera_white_balance_once(serial) / set_camera_white_balance(serial, ...)
chloros_sdk.camera_presets() / apply_camera_preset(serial, preset)
chloros_sdk.camera_filter_models() / camera_filter_model_info(name)
chloros_sdk.camera_firmware_status() / update_camera_firmware(serial)
chloros_sdk.camera_power_probe(serial) / sleep_camera(serial) / wake_camera(serial)
chloros_sdk.calibrate_camera_with_target(serial, ...)
chloros_sdk.network_adapters(...) / fix_network_settings()
chloros_sdk.apply_vignette_correction(input_dir, output_dir, ...)
chloros_sdk.read_dls(mac, ...) / probe_system(...) / system_info()

# === Transient cameras, alignment profiles, indices (cameras opened for one command) ===
chloros_sdk.start_camera_continuous(serial, ..., daq=None, daq_tolerance_ms=None)  # → CliJob; a DAQ bound to the camera records one .daq per frame (daq=False: none)
chloros_sdk.start_multi_camera_stream(serials, ...)   # → CliJob
chloros_sdk.capture_cameras_transient(serials, ...)
chloros_sdk.test_array_sync(serials, count, ptp_settle_s=None) / detect_array_master(line) / analyze_array_models(models)
chloros_sdk.calibrate_alignment_profile(serials, save_path, ...)
chloros_sdk.capture_aligned_multiband(profile_path, ...) / start_aligned_stream(profile_path, ...)  # → dict / CliJob
chloros_sdk.alignment_profile_info(path) / reorder_alignment_profile(path, ...)
chloros_sdk.index_catalog() / compute_index(input_path, ...) / capture_live_index(profile_path, ...)
chloros_sdk.CliJob                                # stats() / stop() / wait()

# === Project lifecycle ===
chloros_sdk.open_project(path, relink=None)       # → ChlorosProject
chloros_sdk.project_folder() / set_project_folder(path) / reset_project_folder()
chloros_sdk.run_project_recipe(recipe, project_path)  # `chloros-cli project run`
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics               # DLS: DAQ-M over Bluetooth; needs bleak

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosProjectFilesMissingError      # a ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True when chloros_sdk.project imported; open_project()
                                 # still raises ImportError without the project loaders
```

---

## Procesamiento de imágenes — `ChlorosLocal`

La clase principal del flujo de trabajo. Inicia el backend al crearse (si ninguno responde y `auto_start_backend=True`), crea y configura proyectos, supervisa el progreso y devuelve resúmenes tras la ejecución.

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
| `open_project(project_path, relink=None)` | Abre un proyecto guardado (su carpeta o su nombre en la carpeta de proyectos) y lo convierte en el proyecto abierto. Las rutas de un proyecto movido se redirigen a la carpeta en la que se encuentra ahora; `relink` indica el nombre de la carpeta en la que se encuentran ahora los demás archivos que faltan. Devuelve la respuesta del backend con `source_files`; genera un error `ChlorosProjectFilesMissingError` cuando no se encuentra ninguno. Véase **Un proyecto movido** más abajo. |
| `import_images(folder_path, recursive=False)` | Importa imágenes RAW/TIF/JPG/DNG **y `.daq` grabaciones del sensor de luz**. Devuelve `count` (imágenes) y `scan_count` (registros). Solo muestra una advertencia si la carpeta no contiene ninguno de los dos. |
| `export_light_sensor(daq=True, csv=True)` | Escribe los archivos calibrados `.daq` + `.csv` para cada registro del sensor de luz del proyecto, en `<project>/Light Sensor/`. Véase [Grabaciones del sensor de luz](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Configurar los controles de procesamiento. `indices` son nombres de preajustes (`NDVI`, `GNDVI`, `OSAVI`, …), cada uno de los cuales se añade al proyecto para las bandas de su cámara; un nombre que no sea un preajuste, o que las bandas no puedan formar (`NDRE` necesita RE y NIR, que ninguna cámara proporciona por sí sola), recibe un nombre del tipo `UserWarning` y no tiene imagen índice. `daq_log_path` designa una grabación del sensor de luz (un archivo `.daq` o `.csv`, o una carpeta con ellos) con el que se compara cada fotograma, tanto en LATTICE como en Survey3, en lugar de las grabaciones importadas con las imágenes; si se indica una ruta que no existe o que no contiene ninguna grabación, se utilizarán las importadas. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Ejecuta el proceso. Devuelve `{"status": "complete", "async": False}`, además de una clave `summary` cuando el backend la proporcione — véase [Resumen y sugerencias posteriores a la ejecución](#post-run-summary--hints) — `export_check_failure` cuando no se han podido verificar las exportaciones de la ejecución, y `image_failure` cuando se ha perdido una imagen. `wait=False` devuelve `{"status": "started", "async": True}` a la vez. |
| `get_config()` / `get_status()` / `status()` | Inspecciona el estado del backend. |
| `logout()` | Cerrar sesión: borra las credenciales almacenadas en caché y libera la ranura de dispositivo de esta máquina (`device_deregistration` en el resultado). No se detiene una ejecución de procesamiento en curso: finaliza en el modo en el que se inició (`run_in_progress: true` y `run_mode` en el resultado), y la siguiente ejecución utiliza el procesamiento estándar. |
| `shutdown_backend()` | Detiene el backend si esta instancia lo ha iniciado; si encuentra un backend en ejecución, no interviene. Dejar un bloque `with` o que la instancia sea objeto de la recolección de basura no lo detiene: el backend sigue ejecutándose para el siguiente script y se detiene por sí mismo tras 30 minutos sin que nada lo utilice. Se le solicita al backend que se apague, por lo que libera sus cámaras y sensores de luz y finaliza primero cualquier grabación; solo se cierra si sigue en ejecución 10 s después. |
| `discover_cameras()` | Detecta cámaras LATTICE **a través del backend de esta instancia** (`/api/camera/discover`). Devuelve una lista de diccionarios (`serial`, `model`, `ip`, …) — con la misma estructura que ven la interfaz gráfica de usuario (GUI) y la CLI. Lista vacía si no se encuentra ninguna o si no se puede acceder al backend. |
| `camera_capture(output_dir, format="tiff", **settings)` | Captura un único fotograma**a través del backend**(iniciado automáticamente por este identificador) para que reciba la misma preparación que la GUI o la CLI (12 bits por defecto, reutilización del pool, metadatos de calibración incrustados). Resuelve el destino con `serial=` o `device_index=`; pasa `exposure`/`gain`/`pixel_format`/`preset` como `**settings`. Devuelve el diccionario de metadatos heredado (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Genera fotogramas de vista previa compuestos mediante superposición a partir de una cámara agrupada: cliente MJPEG ligero a través de la ruta `/api/camera/<serial>/stream-annotated` del backend (zebra / cuadrícula / retículo / histograma / peaking / punto dibujado en el lado del servidor). `decode=True` genera matrices BGR y requiere OpenCV (`cv2`), que el SDK no instala; `False` genera bytes JPEG sin procesar. También accesible por proyecto como `ChlorosProject.stream(overlays=True)`. |

**Una ejecución en curso conserva su proyecto.** Mientras se está ejecutando un proceso —ya sea este script, la aplicación, `chloros-cli process` o `chloros-mcp`—, el backend se niega a crear, abrir, importar o modificar la configuración de procesamiento de un proyecto (409 HTTP `PROCESSING_IN_PROGRESS`): `create_project()` y `import_images()` generan el error `ChlorosProcessingError`, y `configure()` genera `ChlorosConfigurationError`, y `process()` con un modo distinto al actual genera `ChlorosProcessingError` (una ejecución mantiene el modo en el que se inició), cada una con el mensaje del backend. Una captura guardada en el proyecto se une a este cuando finaliza la ejecución (así lo indica su respuesta `project_add`). Espera a que finalice la ejecución (`get_status()`) y, a continuación, continúa.**Un proyecto movido.** El `project.json` de un proyecto nombra cada archivo fuente (cada imagen y cada grabación del sensor de luz) por su ruta completa, por lo que al mover la carpeta del proyecto —o toda la carpeta de proyectos a otra unidad— estos siguen haciendo referencia a la ubicación anterior. Cada vez que se abre, el programa corrige lo que puede por sí mismo: busca un archivo que estaba dentro de la carpeta del proyecto en la carpeta en la que se encuentra ahora el proyecto, y cada ruta que encuentra allí se reasigna y se guarda (`project.json` se guarda primero como copia de seguridad con el nombre `project.json.before-relink-<date>-<time>`). Solo se reasigna la ruta de un archivo que se encuentre realmente en ese lugar. Los archivos que se encontraban fuera de la carpeta del proyecto —una sesión de Hub, una importación de «Añadir carpeta»— solo se encuentran con `relink=<the folder they are in now>` (o cualquier carpeta por encima de ella). `open_project()` devuelve `source_files`: `total`, `found`, `missing`, `missing_roots` (las carpetas en las que se encontraban los archivos que faltan), `missing_outside_project` y `relink` (lo que se ha reasignado, de dónde a dónde). Un proyecto al que le faltan algunos archivos se abre con el resto; uno en el que no se encuentra ninguno no se abre, y `open_project()` genera el error `ChlorosProjectFilesMissingError` (`.missing_roots`, `.audit`; HTTP 409 `PROJECT_SOURCES_MISSING`). Los códigos `project open --relink` y `chloros-mcp` de la CLI`open_project` (`relink_folder`) de la CLI hacen lo mismo.

Úsalo como gestor de contexto para garantizar la limpieza:

```python
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

### Registros del sensor de luz — calibrados `.daq` + `.csv`

Un DAQ-U / DAQ-M / DAQ-E se puede grabar **sin** su paquete de calibración. Eso es
lo que hacen por defecto los grabadores públicos [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`): escriben los recuentos sin procesar de los sensores y marcan el
archivo para que Chloros recupere la calibración de fábrica de ese sensor **por número de serie** — primero en la caché local
primero, luego en la nube de MAPIR — y la aplica al importarlo.

Chloros vuelve a guardar el resultado como dos productos por grabación, bajo
`<project>/Light Sensor/`:

| Producto | Qué es |
| --- | --- |
| `<name>_calibrated.daq` | El archivo reprocesable —con el mismo esquema que una grabación en tiempo real—, que ahora indica el paquete que lo generó. Al volver a importarlo, **no** se vuelve a calibrar. |
| `<name>_calibrated.csv` | Irradiancia espectral en W/m²/nm en la propia rejilla de longitudes de onda del sensor, una fila por lectura, más columnas fotométricas (potencia total, lux fotópicos/escotópicos, PPFD y su desglose en azul/verde/rojo, longitud de onda máxima). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Solo sensores sin paquete (DAQ-A).** Recuentos espectrales sin procesar del sensor — *no* irradiancia. Véase más abajo. |

`<name>` es el nombre del propio archivo de la grabación. Dos grabaciones en carpetas diferentes pueden compartir
una —lecturas por captura (`<capture>/daq/daq_data_<sensor id>_<ts>.daq`) guardadas en el
mismo milisegundo por dos cámaras— y cada una sigue obteniendo sus propios productos: la primera conserva
`<name>`; la siguiente añade las carpetas que la distinguen **después** del nombre
(`<name>__<capture>_daq_calibrated.daq`), de modo que la fecha y la hora de la grabación siguen apareciendo al principio.

`process()` lleva a cabo esta exportación como una de sus etapas. **No** requiere imágenes:
un sensor de luz que vuela por sí solo constituye un flujo de trabajo de primera clase, y un proyecto de este tipo tiene cero
imágenes por su propia naturaleza.

**Las grabaciones de DAQ-A se exportan como recuentos sin procesar.** La familia DAQ-A es anterior al sistema de paquetes por número de serie
y no tiene ningún paquete que recuperar; en su lugar, se calibra sobre el terreno frente a un
blanco de reflectancia, por lo que nunca lo ha necesitado. Esos registros se exportan
con el prefijo `_raw` en lugar de `_calibrated`: un nombre de archivo diferente en lugar de un indicador
dentro del archivo, ya que la información debe conservarse al reenviarse por correo electrónico como un simple nombre. El
encabezado `.csv` indica `raw spectral sensor counts (NOT irradiance)` y advierte de que los
valores son comparables **dentro** del archivo —exactamente para lo que sirve la calibración basada en objetivos—
para ello—, y no entre sensores. Las columnas fotométricas dependientes de la potencia (potencia total,
lux fotópico/escotópico, PPFD) devuelven **NULL** en lugar de integrarse a partir de los recuentos.

Un DAQ-U / DAQ-M / DAQ-E cuyo paquete simplemente no se haya podido recuperar sigue siendo **omitido**,
en lugar de grabarse en formato sin procesar: en ese caso, el paquete existe y «volver a conectar y volver a procesar» es un consejo válido.

Las grabaciones heredadas **v1.01 / v1.02** (un DAQ-A-SD las escribe) no incluyen una época por lectura,
sino solo la hora de escritura del archivo. El comparador de imágenes↔luz descendente sigue rechazándolas —comparar un
fotograma con una hora de escritura daría un resultado erróneo de forma imperceptible—, pero el exportador las lee, y el
archivo CSV muestra `clock=daq_created_on`, de modo que el producto indica en qué reloj se encuentra.

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
calibración en el archivo) se indica en `skipped` **con el motivo**. Nunca se
guarda como un archivo «calibrado» que contenga recuentos sin procesar: conéctate a Internet y
vuelve a ejecutar la tarea, y la exportación se completará.

### Llamadas de retorno de progreso

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Resumen y sugerencias tras la ejecución

Al finalizar, `process()` recupera `GET /api/processing-summary` y adjunta el cuerpo como `result["summary"]`. La recuperación se realiza en la medida de lo posible y nunca bloquea un retorno satisfactorio — si el resumen no está disponible, `process()` recurre a la forma básica de `{"status": "complete", "async": False}`. Cada entrada de `summary["hints"]` —frases completas con la solución sugerida, p. ej., por qué una ejecución no ha generado ningún resultado —, se vuelve a emitir como un `UserWarning` de Python, de modo que las ejecuciones sin resultados se autodiagnostican incluso si nunca se revisa el diccionario:

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
| `images_in_groups` | Imágenes de origen de esos grupos. |
| `targets_found` | Objetivos de reflectancia detectados. |
| `images_calibrated` | Imágenes para las que la ejecución generó un producto de reflectancia. |
| `images_failed` | Imágenes que fallaron y no se exportaron. |
| `exported_files` | **Archivos de productos de imagen que generó la ejecución**: todos los productos (desbayering, vista previa, radiancia, reflectancia, índice, renderizados de objetivos), en todos los formatos. |
| `reflectance_files` | Los productos de reflectancia entre ellos (`Reflectance_Calibrated_Images`), tanto si han sido calibrados por un objetivo como por la irradiancia del sensor de luz (DAQ). Una ejecución sin objetivo que aún así haya generado datos de reflectancia lo indica en una nota; solo a una ejecución que no haya generado ninguno se le indica que faltan sus resultados de reflectancia. |
| `clamp_masks_written` | Las [máscaras de recorte de desmezclado](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) que aparecen junto a ellas —son complementarias, no productos, por lo que no figuran en `exported_files` (la CLI, las cuenta por separado de la misma manera). |
| `captures_left_out` | Fotogramas sin procesar en la carpeta `captures` del proyecto que esta ejecución no ha capturado. Una ejecución espera al iniciarse a lo que se capturó anteriormente, por lo que este valor es 0 a menos que se guardara una captura durante la ejecución o se agotara el tiempo de espera. `summary["captures_left_out_note"]` indica qué hacer con ellas. |
| `daq_recordings_exported` / `daq_recordings_skipped` | Registros del sensor de luz, contados por separado a propósito — proceden de una etapa diferente y existen para ejecuciones sin imágenes en absoluto, por lo que incluirlas haría que una ejecución solo de DAQ pareciera que había exportado imágenes. |

Junto a ellos: `summary["output_dirs"]` (todos los directorios en los que se ha escrito),
`summary["light_sensor_export"]`, `summary["stopped"]` (verdadero cuando el usuario interrumpió la
ejecución, para que los recuentos parciales no se interpreten como una ejecución completada con una producción inferior a la esperada), y
`summary["groups"]` (el desglose por grupos).

`exported_files` lo registra el canal **a medida que escribe**, no se extrae posteriormente de los
objetos de imagen del proyecto. Las estrategias paralelas y de GPU crean sus propios objetos de imagen
(en subprocesos de trabajo para las rutas de GPU), por lo que el antiguo escaneo informaba de
`0 file(s) written` para cada una de esas ejecuciones y, a continuación, emitía la indicación de «cero exportaciones» —en ejecuciones
en las que todo había funcionado correctamente—. Si se crea un script basado en este número, una ejecución paralela correcta ahora
muestra un recuento distinto de cero.

Los saltos del sensor de luz informan del motivo que el lector estableció realmente para cada archivo —un
esquema ilegible, un paquete que falta, un error de escritura— **deduplicado**, de modo que veinte archivos
omitidos por una misma causa se interpretan como una sola causa en lugar de veinte repeticiones de la misma.

> **`process()` no se activa cuando una ejecución no genera imágenes.** Este es uno de los dos puntos en los que el SDK y
> la CLI difieren deliberadamente: `chloros-cli process` trata «se solicitaron productos, pero no se
> escribió ninguno» como un fallo y sale con un valor distinto de cero, mientras que el SDK devuelve el resultado normalmente e informa de la
> condición a través de `summary` / hints. Si tu canalización debe detenerse ante una ejecución vacía, compruébalo
> tú mismo: inspecciona `summary` (o cuenta los archivos que hay en la carpeta del proyecto) en lugar de basarte en
> la ausencia de una excepción. Las causas habituales son una carpeta de entrada que no se ha reconocido como
> captura y productos omitidos por no ser aplicables a las cámaras presentes (por ejemplo, la radiancia de cámaras que solo captan RGB
>).
>
> `process()` tampoco se activa cuando no se han podido verificar las exportaciones de una ejecución: muestra un aviso y establece
> `result["export_check_failure"]`. Los productos se escriben. Para detener la ejecución en ese caso, comprueba esa clave. Lo mismo ocurre con una ejecución que
> ha perdido una imagen: muestra un aviso y establece `result["image_failure"]`, lo que hace que `chloros-cli process`
> termine con un valor distinto de cero.

### Funciones de conveniencia

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "GNDVI", "OSAVI"],
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

# process_folder() / process_lattice_capture() also take input_level=,
# array_alignment=, array_alignment_crop=, array_alignment_interpolation=,
# progress_callback=, poll_interval=, timeout=, processing_stuck_timeout=,
# backend_exe= and api_url= (NOT backend_url=). indices defaults to ["NDVI"].

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

Cada una se ejecuta dentro de su propio `with ChlorosLocal(...)`; un backend que tuvo que iniciar sigue ejecutándose cuando devuelve el resultado, para la siguiente llamada, y se detiene automáticamente tras 30 minutos si no hay nada utilizándolo. Ninguna de ellas admite `auto_start_backend`, `radiometric_output` ni `daq_log_path`; utiliza `ChlorosLocal` y `configure()` para esos casos. `process_lattice_capture()` no admite `camera`, `debayer` ni `ppk`: omite la detección de paneles y utiliza el «debayer» estándar.

### Valores admitidos

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"TIFF (8-bit)"            # all-8-bit images are written as this; never raised
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer for Survey3 imagery, Chloros+ only.
                                         # Survey3 only: on a project holding LATTICE captures
                                         # (which always use the standard demosaic) configure()
                                         # or process() raises ChlorosConfigurationError; a
                                         # Texture Aware value the project saved earlier is set
                                         # to Standard with a UserWarning, as the app does. Needs a
                                         # CUDA GPU the backend's PyTorch can use (an NVIDIA
                                         # GPU, or a Jetson Orin's). Without one, process()
                                         # uses the standard debayer, and the backend prints
                                         # one "[DEBAYER] ... needs a CUDA GPU" line saying why.
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as Chloros's saved debayered (3-channel) product
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

#### Salida radiométrica (cadena de procesamiento multiespectral LATTICE)

El nivel de exportación multiespectral (M3C/M3M) de LATTICE del proceso `process` — `reflectance` (por defecto), `radiance`, `sensor-response` o `all` (cada modo aplicable por imagen)— establece los productos **«Exportar radiancia»** y **«Exportar reflectancia»**: `radiance` exporta solo radiancia, `reflectance` solo reflectancia, `sensor-response` ninguna de las dos (solo los productos sin bayering y de vista previa), `all` ambos. Los productos sin bayering y de vista previa no se pueden modificar en este nivel. `configure()` tiene una palabra clave específica para ello:

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

La vía de escape avanzada —escribir los productos mediante `custom_settings`— sigue funcionando, pero recuerda que sustituye todo el bloque de configuración (véase la advertencia más abajo). Escribe las claves `"Export <product>"`: la configuración de `"Export radiance"` / `"Export reflectance"` de un proyecto prevalecen sobre la clave `"Radiometric output"` más antigua, por lo que esa clave por sí sola no cambia nada.

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Export radiance": True, "Export reflectance": False},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (la predeterminada) divide la radiancia de la cámara por el **flujo descendente del DAQ con marca de tiempo coincidente**, resuelto automáticamente a partir de un `.daq` (DAQ-U/M/E)**o un `.csv` nativo de DAQ-M**que se encuentre junto a las imágenes; cualquier paquete de calibración por cámara o por DAQ que falte localmente se**obtiene automáticamente de AWS** en el primer uso. La CLI lo expone segúnconmutadores de tipo de producto en `chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.**¿De quién es esa lectura?**Una lectura que guarda una captura `.daq` es la lectura de la(s) cámara(s) que describe —nombrada en un archivo sidecar `<file>.daq.cameras.json` junto a ella (capturas únicas, sincronizadas y en matriz; una ráfaga sin procesar nombra la lectura de cada miembro en su manifiesto) — y se empareja**únicamente con imágenes de esas cámaras**. Por lo tanto, una cámara independiente que no tenga una lectura propia ya no toma prestado el DAQ de otra cámara; hay que vincularle un DAQ o grabar uno. La propia grabación de un DAQ (`chloros-cli daq pool-record`, o una grabación iniciada en la interfaz gráfica de usuario), o cualquier `.daq` sin ese archivo complementario (incluidas las capturas más antiguas), sigue sirviendo para todas las cámaras.**Cámaras OCN bajo la luz solar.** Todos los paquetes de calibración de OCN se crean bajo luz de banda estrecha, con un LED por banda, mientras que la luz solar mezcla las bandas. Chloros 1.3.0 aplica una corrección global basada en la luz solar tras la separación de bandas, lo que mejora los productos de radiancia, reflectancia e índice de OCN bajo la luz solar. Se ejecuta dondequiera que se realice la separación de bandas —`process()`, capturas, `grab()` / `frame_stream()` y la vista en directo— para los paquetes OCN que lo admiten; las cámaras RGN no sufren cambios. Las imágenes OCN capturadas con una versión anterior también se benefician de ello cuando se procesan en un ordenador que conserve la calibración con la que se tomaron (Chloros conserva la calibración de una cámara una vez que esta se ha conectado); de lo contrario, se procesan como antes. Por lo tanto, los valores de radiancia, reflectancia e índice de OCN difieren de los de la versión 1.2.0.

> `custom_settings` **sustituye** todo el bloque de ajustes calculados (por diseño, omite las demás palabras clave y la validación de `configure()`). Cuando lo utilices, incluye todas las claves `Project Settings` que te interesen, tal y como se muestra en el ejemplo anterior.

#### Lectura de píxeles de índice (el NDVI y los demás índices de vegetación)

**Una ráster de índices entera es un estiramiento de contraste por imagen, no un mapa de −1…1.** `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)` y `JPG (8-bit)` —incluido el archivo TIFF de 8 bits en el que se guarda una captura de 8 bits— almacenan

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

> **`IndexMin == IndexMax`** significa que cada píxel de ese fotograma tenía el mismo valor de índice. La trama se rellena a escala media y no hay nada por lo que dividir; toma el punto final.

> **La ausencia de etiquetas `Chloros:Index*` significa que no hay una asignación especificada.** Una exportación LUT es un renderizado coloreado cuyos píxeles son colores degradados ajustados a la ventana de umbral de tu LUT, y un fotograma sin píxeles de índice finito no tiene rango que indicar. No des por sentado que existe una asignación en ninguno de los dos casos —se aplica la misma regla que cuando falta una etiqueta `PixelScale`.**Dado que el estiramiento es por imagen, dos fotogramas de un mismo vuelo no son comparables en términos de DN**: cada uno tiene sus propios extremos, por lo que el mismo NDVI se sitúa en diferentes valores de DN en distintas imágenes. Para trabajos cuantitativos a lo largo de un vuelo (estadísticas zonales, umbrales fijos, series temporales), exporta**`TIFF (32-bit, Percent)`**: las mallas de índice se escriben allí como float32**los propios valores de índice**, sin necesidad de reescalado ni etiquetas. A pesar del nombre del formato, los productos de índice no son ×100; esa escala solo se aplica a la reflectancia de tipo de datos entero.

#### Máscara de recorte de desmezclado: en qué píxeles se ha recortado una banda

El desmezclado espectral 3×3 puede hacer que una banda resulte negativa —cuando la luz apenas contiene esa banda, o cuando la escena queda fuera de lo que describe la matriz de desmezclado de la cámara—. Como no se trata de un valor real, la exportación lo fija en 0 — y la vista en directo muestra ese mismo 0. Una banda puede estar limitada en unos pocos píxeles o en casi todos ellos; la máscara registra cualquiera de las dos situaciones. `Chloros:Band{R,G,B}UnmixClampFraction` indica en qué parte del fotograma ha ocurrido; la máscara de limitación indica **qué píxeles**:

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— en una carpeta `clamp_masks/`**junto a** la carpeta del producto, nunca dentro de ella (`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`) — para cada producto de radiancia, reflectancia e índice en el que al menos un píxel haya sido recortado. `uint8` de banda única, en la propia cuadrícula de píxeles del producto (orientación, ROI, agrupación de píxeles y la distorsión de alineación de la matriz incluidas). Cada máscara indica la cámara de su producto (`Make` / `Model` / número de serie), la hora de captura y las etiquetas del objetivo, y enumera las bandas a las que corresponden sus bits (`XMP-Camera:BandName`, primero el bit 0); sus marcas `ImageDescription` y `Software` la identifican como una máscara de recorte, y no contiene calibración.
- **Bits**: bit 0 = la banda del canal R del producto (`Chloros:BandRName`), bit 1 = G (`BandGName`), bit 2 = B (`BandBName`). Un bit activado significa que el valor de ese píxel en esa banda se ha limitado a 0; considéralo, así como cualquier índice construido a partir de él, como de baja fiabilidad.
- **`Chloros:UnmixClampMask`** aparece en todos los productos en los que se ha ejecutado la separación de canales: la ruta de la máscara relativa a la propia carpeta del producto (`../clamp_masks/<product folder>/<product name>_clampmask.tif`, siempre `/`separado por un espacio — únelo a la carpeta de la imagen para abrir la máscara), o `none` cuando no se ha limitado ningún píxel. La ausencia de etiqueta significa que el producto nunca se sometió al proceso de separación de canales (cámaras RGB o monocromáticas, niveles sin procesar/sin debayering).
- Los resultados de la captura lo incluyen como **`clamp_mask`** del fotograma (la ruta de la máscara, o `None` cuando no hay nada recortado) — en los resultados `CameraSession.capture()`, los fotogramas `ArraySession.capture()`en los fotogramas `CaptureResult` y las respuestas de captura HTTP. La compilación fuera de línea enumera sus máscaras como `clamp_masks`.
- La pila multibanda de un índice en directo (`/api/camera/cli-array-index` con `save_multiband`) obtiene una máscara por cada cámara de origen, en el propio `clamp_masks/` de la carpeta de salida —véase la referencia de la CLI`lattice index`. Un vídeo no tiene máscara: en su lugar, el resultado de la compilación fuera de línea indica la fracción de recorte de cada fotograma.

Dado que ninguna carpeta de producto contiene una máscara, a cualquier nivel de jerarquía, una carpeta de producto puede entregarse tal cual al software de fotogrametría —subcarpetas incluidas— y este solo procesa las imágenes. Entrégale la carpeta de producto, no la carpeta que está por encima. La propia función de importación de Chloros omite las máscaras.

```python
import os, numpy as np, tifffile

mask = tifffile.imread(
    "clamp_masks/radiance/capture_<ts>_SN<serial>-<filter>_radiance_clampmask.tif")
# or, from the image: os.path.join(os.path.dirname(image_path), tag), with
# tag = the image's Chloros:UnmixClampMask
r_clamped = (mask & 1) != 0      # the band Chloros:BandRName names
g_clamped = (mask & 2) != 0
b_clamped = (mask & 4) != 0
```

---

## Smart-Connect para cámaras LATTICE

Sesiones de backend persistentes para hardware en tiempo real. Se utilizan los mismos puntos finales que la interfaz gráfica de usuario, por lo que el comportamiento es idéntico en SDK, CLI y GUI.

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

La exposición, la ganancia y el objetivo configurados aquí se mantienen como los tuyos: el control automático de exposición de Chloros (incluida la exposición medida por DAQ) no los modifica. `cam.set_settings(exposure_auto="Continuous")` (o `gain_auto=`) devuelve la exposición (ganancia), y `cam.set_settings(target_brightness_auto=True)` devuelve el objetivo al sistema automático de Chloros.

#### `connect_camera()` Firma

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

- **Vuelve con la transmisión iniciada.** El backend espera, con un límite (unos 15 s para que se inicie el flujo y, a continuación, 10 s para su primer búfer completo —no para el primer fotograma de vista previa, que llega aproximadamente 1,5 s después—), por lo que aparece un `status()` justo después de que diga `streaming`. El `connect_stream` de la sesión es el bloque `stream` de la respuesta: `state` `streaming`, `no_frames` (el flujo se inició y no se transmitió nada — así lo indica el `note` y, en el caso de una cámara con paquetes jumbo, para volver a conectarse con un tamaño de paquete de 1500 bytes), `starting` o `start_failed`, con `waited_s`. `None` para una cámara que ya estaba conectada. Se solicitó una vista previa (`get_camera_preview`, `/api/camera/jpeg/<serial>`) solicitada justo después de una espera, limitada a 4 s, del primer fotograma de vista previa de la cámara en lugar de responder «Aún no hay fotograma disponible»; una cámara cuyo flujo no haya enviado nada (`no_frames`, o activada por un disparador de hardware) sigue respondiendo de inmediato.
- **El tamaño del paquete lo decides tú.** `settings={"gev_packet_size": 1500, "gev_packet_size_auto": False}` abre la cámara con paquetes de 1500 bytes, sin sondear (`chloros-cli lattice cam-connect --packet-size standard` envía lo mismo).
- **Solo se permite una conexión a la vez en el grupo de cámaras**: la apertura, la comprobación del tamaño de paquete, una primera descarga de calibración (unos 70 s para una cámara nueva en este ordenador) y el inicio de la transmisión. Una solicitud que necesite una cámara mientras tanto (`read_nodes`, `set_settings`, una captura, una segunda conexión) espera como máximo 5 s y, a continuación, falla con un error HTTP 503 `CAMERA_POOL_BUSY`, indicando el nombre de la cámara que se está conectando (`ChlorosConnectError`); vuelve a inténtalo de nuevo en un momento. Las listas nunca esperan: `list_cameras()` (`GET /api/camera/pool`), `list_arrays()` y `CameraSession.status()` responden de inmediato. Una cámara en proceso de conexión aparece en la lista bajo la respuesta del grupo `connecting` (`serial`, `phase` — `opening the camera`, `rebalancing the pool bandwidth`, `loading its calibration`, `starting its stream`, `waiting for its stream` — y `elapsed_s`), y su `status()` indica `connecting: True` con su `phase`; las demás cámaras se leen en tiempo real cuando el grupo está libre; en caso contrario, se responde a partir de su última lectura (`live: False`).

#### Métodos de `CameraSession`

| Método | Descripción |
| --- | --- |
| `status(timeout=15.0)` | El estado en directo de la cámara, tal y como lo lee `chloros-cli lattice status`: `streaming`, `stream_state` (`starting`, `no_frames`, `streaming`, `start_failed` o `stopped`), `device` (modelo, número de serie, firmware, temperatura, `is_mono`) y su configuración actual. |
| `read_nodes(names, enum_names=(), timeout=30.0)` | Lee los nodos GenICam; devuelve `{nodes, errors, enums, device, was_in_pool, in_array_id}`, además de `frame_values` y `notes` mientras se ejecuta la exposición automática (véase más abajo). |
| `set_settings(**kwargs)` | Escribe en los nodos por su nombre descriptivo (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `target_brightness_auto`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). Un `exposure_time` (`gain`) es una exposición (ganancia) manual: al escribirse sin `exposure_auto` (`gain_auto`), primero se desactiva el modo automático, para que el valor no sea rechazado por un modo automático en marcha. El resultado, `applied`, conserva lo que captó la cámara; una escritura rechazada por la cámara se encuentra en `errors` y no se conserva. El error indica el rango que la cámara acepta en ese momento (un valor máximo de `ExposureTime` que depende de la velocidad de fotogramas de la cámara), y se restablece un modo automático que se había desactivado únicamente para el valor rechazado; `restored` lo enumera (p. ej., `{"ExposureAuto": "Continuous"}`). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, daq=None, daq_tolerance_ms=None, settings=None, timeout=None)` | Captura un **único** fotograma. Devuelve un `CaptureResult`: una lista de diccionarios de metadatos de fotogramas (uno por nivel guardado), con `.skipped`, `.failed`, `.daq_files` y `.daq_matches`. (Se ha eliminado la captura en ráfaga o de varios fotogramas; llama a `capture()` en un bucle si necesitas una serie.) |
| `capture_repeated(output_dir="output", *, count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Capturas consecutivas (`interval_s=0`) o a intervalos desde esta cámara, para capturas `count` y/o `duration_s` (al menos una, para que finalice), tal y como hace `ArraySession.capture_repeated()` con una matriz. Devuelve la lista de `CaptureResult`; se detiene tras una captura que indique que la unidad está llena. `on_capture(i, result)` se ejecuta después de cada una; el resto de palabras clave pasan a `capture()`. |
| `disconnect(keep_settings=False, clear_settings=False)` | Liberación del grupo; la cámara se restablece tal y como estaba al conectarse (cableado del disparador, PTP, ajustes); `keep_settings=True` guarda su configuración en su propia memoria; `clear_settings=True` restablece dicha memoria (véase [Memoria de la cámara](#camera-memory--keep_settings--clear_settings)); devuelve `camera_memory` cuando se solicita cualquiera de las dos opciones. Una cámara que forma parte de una matriz libera toda la matriz. No realiza ninguna operación si nos hemos conectado a una sesión ya abierta. Devuelve un resultado una vez que el backend confirma la liberación, o cuando no hay nada que liberar (la cámara ya no existe o no hay ningún backend en ejecución); una liberación que haya agotado el tiempo de espera o que el backend haya rechazado genera el error `ChlorosConnectError`, ya que la cámara podría seguir ocupada y la siguiente conexión entraría en conflicto con ella. La salida del bloque `with` nunca genera ningún error. |

Atributos: `serial`, `device` (el registro de detección de la cámara) y `backend_url`.

Controles de exportación de `capture()` (mismo modelo que la matriz + interfaz gráfica de usuario):

- `processing` / `levels` — `processing="all"` guarda todos los tipos de exportación aplicables; `levels=["raw","radiance"]` guarda solo esos (anula a `processing`). Omite ambos para utilizar el valor predeterminado del backend.
- `daq`: el DAQ que lee la captura utilizada se guarda de forma predeterminada como un archivo complementario `.daq`, que incluye una captura solo en formato sin procesar, de modo que la imagen se pueda volver a procesar posteriormente en reflectancia/índice. `daq=False` lo omite cuando ningún nivel lo utiliza. `force_daq=True` es la grafía anterior del valor por defecto. Sin DAQ, ni `.daq`.
- `daq_tolerance_ms`: distancia máxima a la que puede estar la lectura respecto a la exposición y seguir estando dentro de la tolerancia (por defecto, 500). Solo cambia el veredicto.

#### `read_nodes()` con exposición automática — `frame_values` y `notes`

Mientras se ejecuta `ExposureAuto` (`GainAuto`), el nodo `ExposureTime` (`Gain`) lee el **último valor escrito**, no lo que está haciendo la cámara en ese momento. `nodes` sigue conservando ese valor del nodo. Cuando se solicita `ExposureTime` o `Gain` en una cámara conectada cuyo modo automático no sea `Off`, la respuesta añade:

- `frame_values`: el valor que contenía el fotograma más reciente de la cámara, bajo el nombre del nodo (`ExposureTime` en µs, `Gain` en dB);
- `notes`: una frase por cada nodo de este tipo en la que se indican ambos valores.

```python
r = cam.read_nodes(["ExposureTime", "Gain"])
print(r["nodes"]["ExposureTime"])                  # last value written, e.g. 2000.448
print(r.get("frame_values", {}).get("ExposureTime"))  # what the newest frame used, e.g. 19600.0
for name, note in r.get("notes", {}).items():
    print(note)
# ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448);
# the camera's newest frame was at 19600 us.
```

Ambas claves están ausentes cuando el modo automático es `Off` (en ese caso, el nodo es el) y para una cámara que no está conectada al backend. En el caso de una matriz, `ArraySession.status()["member_ae"]` proporciona los valores de fotograma de cada elemento a la vez.

#### Límite máximo de exposición medido por DAQ (HTTP)

Con un sensor de luz DAQ conectado, Chloros limita la exposición × ganancia de una cámara en función de la luz diurna medida, de modo que un panel de referencia blanco no se recorte. El límite solo se aplica mientras haya un objetivo de calibración en el campo de visión de esa cámara (detectado en su fotograma en los últimos 10 s); si no hay ningún objetivo a la vista, se activa en su lugar la exposición automática habitual de la cámara, con su propia protección contra las luces altas, para que el resto de la escena no quede más oscura de lo necesario. No hay ningún envoltorio de SDK; se puede consultar o activar por cámara con `GET` / `POST /api/camera/<serial>/daq-exposure-cap` (el cuerpo del POST `{"enabled": false}` lo desactiva para esa cámara). La respuesta incluye `enabled`, `cap_us` (el límite máximo en tiempo real; `None` cuando está inactivo, incluso cuando no hay ningún objetivo de calibración a la vista), `k_exp_g` (la constante con la que se calcula el techo) y:

| Campo | Significado |
| --- | --- |
| `k_exp_g_source` | Qué constante se está utilizando: `camera` — la propia de esta cámara, calculada a partir del punto de laboratorio de su paquete de calibración (cámaras RGB); `fleet` — la constante de la flota, utilizada cuando el paquete no tiene punto de laboratorio; `override` — una anulación de campo establecida en el backend. |
| `k_exp_g_fleet` | La constante de la flota, a efectos comparativos. |

### Matriz sincronizada — `ArraySession` (Smart-Prep)

`connect_array` es **el punto de partida recomendado** para configuraciones con varias cámaras. Ejecuta en segundo plano todo el flujo de Smart-Prep a través de la interfaz gráfica de usuario:

1. **Análisis de red**(`/api/camera/array/recommend`): configura el**sensor completo** (sin binning ni recorte de ROI) y permite que la velocidad de fotogramas se reduzca a la que ofrezcan este host y la red, calculada en función de los formatos de píxeles y el empaquetado que utilizará Connect. Los fotogramas nunca se reducen para mantener la velocidad: se debe especificar un valor menor para `width` / `height` o `binning=2` / `4` para sacrificar resolución a cambio de velocidad de fotogramas (la lista del punto final`alternatives` enumera esas opciones con sus fps previstos).
2. **Selección automática de nivel** — `sim-capture-sim-emit` (todas las cámaras exponen con el mismo disparador, la emisión se sincroniza para compartir el cable, y la velocidad de fotogramas disminuye a medida que se reduce el ancho de banda compartido) en cualquier red en la que las cámaras puedan sincronizarse para compartir el cable; `slip-emit-and-capture` (las cámaras se turnan en el cable, sin exponerse simultáneamente) solo cuando no sea posible.
3. **PTP habilitado**por defecto: las marcas de tiempo entre cámaras se sincronizan con un reloj compartido con una precisión de**~1 ms**, el [gran maestro PTP](#ptp-grandmaster--time_sync_status) del backend. La exposición simultánea se debe al disparador por hardware del M8 (**< 100 µs** entre módulos), no al PTP: el PTP sincroniza las *marcas de tiempo*, no las exposiciones.
4. **Selección automática del formato de píxel por cámara**: monocromo → `Mono12`, color → `BayerRG12`.
5. **Inicialización de la exposición automática (AE)**: captura el estado actual de la exposición automática de cada cámara para que la conexión no reinicie la exposición en pleno funcionamiento.
6. **Configuración del disparador GPIO**: `connect_array` activa todas las cámaras (`TriggerMode=On`, `TriggerSource=Line2`) para que el pulso de la cámara maestraimpulso de la cámara maestra accione a las esclavas a través del cable M8. Este paso es exclusivo de la matriz: una sola cámara activada con `LatticeCamera` funciona en modo libre. Al liberar la matriz (`disconnect()` / `force_disconnect()`, `chloros-cli lattice array-disconnect`) hace que todas las cámaras vuelvan al modo de funcionamiento libre (`TriggerMode=Off`), manteniendo el resto de ajustes tal y como los encontró la conexión: los ajustes de disparo y de la línea de sincronización (`TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, `LineMode` / `LineSource`; la línea de sincronización de la cámara maestra vuelve a una entrada), PTP y los ajustes de la cámara que se modificaron durante la sesión —a menos que `keep_settings=True` / `clear_settings=True` (véase [Memoria de la cámara](#camera-memory--keep_settings--clear_settings)).

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

Atributos: `array_id`, `master_serial`, `slave_serials`, `ptp_enabled`, `sync_mode`, `packed_transport` y `packed_reason` (los cuatro últimos son `None` en un identificador de `attach_array`), `connect_stream` y `backend_url`. `connect_stream` es el bloque `stream` de la respuesta de conexión, la forma de la conexión única más `members` (`{serial: {state, why?}}`): `streaming` una vez que ha llegado el primer grupo sincronizado de la matriz (el backend lo espera, con un límite de unos 10 s), o `no_frames` con el último fallo de recuperación de cada miembro como su `why` y, para los paquetes jumbo, el valor de reserva de 1500 bytes en el `note`. `None` en un identificador `attach_array`.

#### Firma de `connect_array()`

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # pin the master trigger rate (auto if None); see set_target_fps
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    packed="auto",                        # packed 12-bit transport: "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=600.0,                        # s; also the backend's deadline (see below)
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

`timeout` indica cuánto tiempo espera la llamada a que se establezca la conexión, y se envía al
backend como su plazo límite. Si la conexión no se ha establecido para entonces, se deshace: todas las cámaras que
se hayan abierto se devuelven al estado en que se encontraron y se cierran, no se conecta ningún array y la
llamada genera el error `ChlorosConnectError` (HTTP 504 `CLIENT_DEADLINE_PASSED`, indicando el
paso en el que se encontraba la conexión). Por lo tanto, una llamada que genera un error nunca deja una matriz
conectada tras de sí. Una matriz de 9 cámaras se conecta en aproximadamente un minuto; justo después de
que las cámaras se enciendan, puede tardar varios. Se devuelve una matriz que está activa pero que aún no tiene
un grupo de fotogramas sincronizado, con el código de error `connect_stream["state"]`
`no_frames`. El registro del backend contiene una línea de tiempo por cada paso de conexión
(`[array-connect <id>] <step>: <s> s …`) y una última línea que indica cuál ha sido el más lento.

Valores de `force_tier`:
- `"sim-capture-sim-emit"`: verdaderamente simultáneo (todas las cámaras disparan en el mismo flanco del reloj).
- `"sim-capture-ftd-stagger"` — escalonamiento flexible en el dominio del tiempo (las cámaras emiten en momentos ligeramente desfasados, de modo que los paquetes se serializan en el cable).
- `"slip-emit-and-capture"` — captura secuencial por cámara (sin sincronización temporal). Se selecciona automáticamente solo cuando no es posible ralentizar el ritmo de las cámaras lo suficiente como para que compartan el cable de forma simultánea.

`wire_ceiling_mbps` anula el **presupuesto de ancho de banda sostenido del host** en MB/s — la única
cifra de la que depende toda la asignación de la matriz. Déjalo en `None` para utilizar el valor detectado automáticamente. Redúcelo cuando la matriz informe de tramas GVSP corruptas: el valor automático se deriva
de la velocidad de enlace anunciada por la tarjeta de red, que sobreestima los adaptadores USB, los carriles PCIe de ancho reducido y
las estructuras compartidas muy ocupadas; y esa sobreestimación se manifiesta en forma de tramas corruptas en lugar de como un
enlace visiblemente lento. El valor se conserva en el bloque de captura de la matriz del proyecto, por lo que al
reabrirlo o al establecer posteriormente `connect_array` se restaura como cualquier otro ajuste de la matriz.
Consulte [Estado de la matriz](#array-health--which-subsystem-is-losing-frames).

#### Transporte empaquetado de 12 bits (`packed=`)

Con todos los sensores activos, la frecuencia de fotogramas de una matriz viene determinada por la velocidad a la que cada cámara puede enviar
sus fotogramas. Los formatos de 12 bits (`BayerRG12`, `Mono12`) suelen transmitirse como palabras de 16 bits;
el transporte comprimido envía los mismos 12 bits en 1,5 bytes por píxel y
los descomprime en el host, por lo que cada fotograma tarda un 25 % menos en enviarse —aproximadamente
**un 33 % más de fotogramas por segundo a plena capacidad del sensor** (medido: de 10,7 a 14,0 fps en una
matriz de 9 cámaras). **Los datos guardados son idénticos**: mismos valores, mismos archivos de 16 bits, mismos metadatos.

- `packed="auto"` (por defecto) comprime solo cuando resulta útil **y** el host puede
  permitirse la descompresión: al conectarse, el backend mide su propia velocidad de descompresión,
  el número de núcleos de CPU, la carga actual y el ancho de banda de red disponible. Un ordenador de sobremesa con núcleos de sobra
  comprime a plena capacidad del sensor; un host ocupado o de baja potencia (ordenadores de placa única,
  controladores integrados) transmite sin comprimir. Las regiones de interés (ROI) pequeñas y los fotogramas agrupados apenas obtienen beneficio.
- `packed="off"` (o `False`) nunca comprime, para descartar esta opción durante el diagnóstico,
  o en un host que debe reservar su CPU para otras tareas.
- `packed="on"` (o `True`) siempre empaqueta los formatos de 12 bits.
- Cualquier otro valor genera `ValueError`. Los formatos de 8 y 16 bits nunca
  se empaquetan.

`analyze_array_network(..., packed="auto")` se aplica con la misma decisión,
por lo que sus fps son la velocidad a la que se ejecutará la matriz (la respuesta contiene
`packed_transport` y `packed_reason`); su valor predeterminado `pixel_format` es el que
aplica «connect» (Mono12 / BayerRG12) en lugar de 8 bits.

La decisión y su motivo se encuentran en la sesión:

```python
arr = chloros_sdk.connect_array(serials)          # packed="auto"
print(arr.packed_transport, arr.packed_reason)
# True  auto: projected 11.0 -> 14.4 fps, unpack 0.16 of 1.50 budgeted cores

arr = chloros_sdk.connect_array(serials, packed="off")
```

`ChlorosProject.connect_all(packed=...)` aplica la misma sustitución a todas las
matrices de un proyecto (por defecto `None`, lo que significa `"auto"`: un proyecto no guarda
ninguna configuración de empaquetado por matriz).
Para establecer un valor por defecto para todo el equipo, configura `CHLOROS_ARRAY_PACKED12=off` (o `on`) en el
entorno del backend; aunque una configuración explícita de `packed="on"` / `"off"` seguirá teniendo prioridad.
`/api/camera/array/status` informa de `packed_transport` y `packed_decision`.

#### Frecuencias de fotogramas esperadas

Frecuencias de fotogramas medidas de la matriz (sincronizadas por hardware, 12 bits; sensor completo = 2048 × 1536). Úsalas para comprobar la validez de `status()["actual_fps"]`; `analyze_array_network()` proyecta la tasa para tu propio host y red.

| Configuración | Frecuencia de fotogramas de la matriz |
| --- | --- |
| Sensor completo, tarjeta de red PCIe de 10 GbE en el host, transporte empaquetado (`packed="auto"` en un ordenador de sobremesa) | ~14 fps |
| Sensor completo, tarjeta de red PCIe de 10 GbE, sin empaquetar (`packed="off"`) | ~10,5 fps |
| Sensor completo, 8 cámaras en un adaptador USB de 10 GbE | ~3,8 fps |
| Sensor completo, enlace de host de 1 GbE | 1,2–1,7 fps |
| Agrupación 2 × 2 (`binning=2`), tarjeta de red de host PCIe 10 GbE | ~30–33 fps |

- En un enlace PCIe de 10 GbE, el límite con el sensor completo lo marcan las **cámaras**, no la red: 4 y 8 cámaras funcionan a la misma velocidad.
- **`target_fps` es la velocidad de disparo, y la matriz nunca funciona más rápido.** `target_fps=1` hace funcionar el conjunto a 1 fps; `None` (el valor por defecto) lo hace funcionar a la velocidad máxima que admiten el host y la red. Cualquier valor superior a esa velocidad se reduce a ella (véase [`set_target_fps`](#interval-capture-for-drone-surveys)).
- **`link_speed_mbps` de `4294` en `status()` no es una velocidad de enlace.** Esto significa que Windows no ha podido leer la velocidad del adaptador (el controlador ha informado del valor máximo de 32 bits), lo cual es habitual en los adaptadores Ethernet USB. La velocidad útil de un adaptador USB en Windows tiene un límite máximo de 200 MB/s en cualquier caso (véase [Estado de la matriz](#array-health--which-subsystem-is-losing-frames)); para ajustar uno, consulta la sección «Adaptadores USB 10 GbE» de la referencia de la CLI en [Configuración y ajuste de la NIC del host](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **En Linux también se aplica un límite para USB: 400 MB/s en x86_64 y 200 MB/s en NVIDIA Jetson.** El adaptador se identifica a través de sysfs (recurriendo a `ethtool -i`), por lo que un adaptador USB de 5 GbE o 10 GbE se limita a este límite máximo en lugar de a su velocidad nominal de 500 / ~1060 MB/s. La cifra de x86_64 se midió en Linux: el mismo adaptador que corrompe tramas por encima de los 200 MB/s en Windows funcionó sin problemas con una matriz de 4 cámaras en todos los límites máximos hasta los 480 MB/s. En NVIDIA Jetson, el mismo adaptador funcionó sin problemas solo a 200 MB/s y perdió tramas de forma intermitente por encima de esa velocidad. Si tu host absorbe más sin que se pierdan tramas con `gvsp_corrupt`, aumenta el límite con `wire_ceiling_mbps`. En Linux, a un adaptador conectado a un puerto USB 2.0 se le asignan 30 MB/s; conéctalo a un puerto USB 3.

#### Sobresuscripción (el límite mínimo por cámara)

Una matriz funciona en cualquier red; la velocidad de fotogramas es la que marca la diferencia. El control de ritmo «Sim-emit» asigna a cada cámara una parte equitativa del ancho de banda disponible sin riesgo de colisiones, y todas ellas siguen disparando con el mismo disparador; por lo tanto, en una conexión lenta, la parte asignada —y la velocidad de fotogramas— es reducida: 8 cámarasde 12 bits en 1 GbE sin tramas jumbo obtienen 6,5 MB/s cada una, aproximadamente **1 fps**(unos 1,3 fps comprimidos). La cuota tiene un límite mínimo en la velocidad más lenta a la que se pueden sincronizar las cámaras (`per_cam_floor_bps`: se lee del rango GevSCPD de cada cámara al conectarse, y nunca por debajo de 1 MB/s). Si la conexión requiere una velocidad inferior a esa, la conexión recurre a la emisión**en serie**(`slip-emit-and-capture`): las cámaras se turnan a la velocidad máxima del enlace, por lo que llegan todas las tramas, pero los miembros**no se exponen simultáneamente**—el nivel de respuesta y los bloqueadores (`SERIAL: ...`) así lo indican; pasa `force_tier="sim-capture-sim-emit"` para rechazar el recurso alternativo. El tamaño de fotograma no influye en el reparto, por lo que**el binning y la ROI aumentan la frecuencia de fotogramas, pero no alteran si una matriz cabe**. Solo un cable que no pueda transportar la matriz a ningún ritmo, ya sea simultáneamente o por turnos,**se sobresuscribe**: el modo de fallo es la pérdida de paquetes GVSP, no una frecuencia de fotogramas más baja. Soluciones: menos cámaras, fotogramas jumbo de extremo a extremo o una tarjeta de red más rápida.

- Las respuestas `analyze_array_network()` y `/api/camera/array/connect` contienen `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` y `per_cam_floor_bps`. Cuando `oversubscribed` es verdadero, la proyección **pone a cero los campos de fps** (`achievable_fps_max` / `fps_bright` / `fps_dark`) en lugar de indicar una tasa engañosa que, aunque lenta, funciona.
- `POST /api/camera/array/connect` acepta un parámetro de cuerpo `pin_resolution` (**solo HTTP — no es un argumento clave del SDK**; `connect_array` no lo expone). Se trata de un modo estricto: una conexión con exceso de suscripciones en la que se haya establecido `pin_resolution` se**rechaza rotundamente** con un error en el que se indican todas las soluciones posibles. Sin él, la conexión prosigue con una advertencia de que la matriz perderá paquetes. En cualquier caso, la conexión mantiene el tamaño de trama solicitado; nunca agrupa ni recorta por su cuenta.
- Solo para pruebas: configura `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` en el entorno del backend para rebajar el rechazo a una advertencia clara: te conectas de todos modos y aceptas la pérdida de paquetes.

#### Estado de la matriz: qué subsistema está perdiendo tramas

`ArraySession.capability()` (`GET /api/camera/array/<array_id>/capability`) lleva un bloque `health` activo en una
matriz conectada, reevaluado en una ventana móvil de **10 segundos** . Desglosa la pérdida de tramas
en las dos causas que requieren soluciones opuestas, en lugar de una tasa «incompleta» que
no especifica ninguna de ellas:

| Campo | Qué significa | Qué subsistema |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (por número de serie) | El fotograma **llegó y presentaba un error estructural**— pérdida de paquetes GVSP. |**Red**: ancho de banda, ritmo de transmisión, anillo de recepción de la NIC, MTU |
| `never_arrived_rate_pct` (por número de serie) | La trama **nunca llegó**— la cámara no se disparó o no salió nada de ella. |**Disparador / sincronización**: cable M8, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Peor tasa de la cámara para cada una. | — |
| `per_cam_rate_pct` | Tasa combinada de imágenes incompletas por cámara (ambas causas juntas). | — |
| `stable_for_seconds` | Tiempo que cada cámara ha permanecido por debajo del 0,01 %. | — |

Junto con `health`, el mismo registro indica la cantidad de la que depende toda la asignación:

| Campo | Qué significa |
| --- | --- |
| `wire_ceiling_mbps` | El presupuesto de ancho de banda sostenido del host actualmente en vigor, en MB/s. |
| `wire_ceiling_source` | De dónde procede esa cifra, en palabras — p. ej., `USB-capped 200 MB/s (was theoretical 1062; …)` o `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` cuando `wire_ceiling_mbps=` lo estableció. |
| `nic_is_usb` | `true` para un adaptador USB-Ethernet. |

```python
import chloros_sdk

serials = ["213800234", "214000533"]           # master first
arr = chloros_sdk.attach_array(serials)
h = arr.capability()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    # force_disconnect(), not disconnect(): this handle came from attach_array,
    # and disconnect() never tears down an array the session only attached to.
    # It returns only once the backend confirms the release; a teardown that
    # timed out or was refused raises ChlorosConnectError instead, so the
    # reconnect below never races a half-released array.
    arr.force_disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Interpretación:** un valor distinto de cero en `gvsp_corrupt_rate_pct` con `never_arrived_rate_pct` en 0 significa
que el disparo y la sincronización del cable son perfectos y que el 100 % de la pérdida se produce en la ruta de red; reduce
`wire_ceiling_mbps` y vuelve a conectarte. El patrón inverso apunta, en cambio, al cable de sincronización o a la
línea de disparo.

> **`target_fps` no es el factor determinante para las tramas corruptas.** El ritmo de GevSCPD se establece una vez al
> conectarse, por lo que reducir la frecuencia de disparo cambia el ciclo de trabajo y no la
> tasa de ráfagas de emisión simultánea. Una reducción medida de la demanda en un 5× no produjo ninguna mejora, mientras que
> reducir el límite máximo del cable de 240 a 200 MB/s hizo que el mismo equipo pasara de un 10,4 % de tramas corruptas a
> un 0,00 %.

> **Una matriz en funcionamiento no puede replanificarse por sí misma en el firmware LATTICE (M3C / M3M).** No puede
> solucionar esto por sí misma; desconéctala y vuelve a conectarla para que el selector de tiempo de conexión vuelva a planificar en función
> del nuevo límite máximo.

Un **adaptador Ethernet USB tiene un límite de 200 MB/s** impuesto por la sonda, independientemente de su
marca: la tabla de eficiencia que convierte una velocidad de enlace en una cifra sostenida se
deriva de PCIe, y una tarjeta de red USB anuncia su velocidad de enlace Ethernet, aunque está limitada por el
bus USB y su controlador. El límite es absoluto, no una fracción: un adaptador USB de 1 GbE
alcanza unos 80 MB/s y no se ve afectado. Se aplica en Windows y Linux (incluido Jetson);
consulta [Velocidades de fotogramas esperadas](#expected-frame-rates) para ver dónde se midió.

#### Métodos `ArraySession`

| Método | Descripción |
| --- | --- |
| `status(timeout=10.0)` | Estado en tiempo real de la matriz en ejecución: `target_fps` / `target_fps_auto` (la frecuencia de activación y si el controlador de frecuencia la gestiona), `target_fps_clamped_reason` (por qué se mantiene más baja una frecuencia fijada; en caso contrario, `None`), `actual_fps` (medido; 0 una vez que el bucle de captura ha desistido o se ha bloqueado), `recommended_fps`, `streaming` (Falso una vez que el bucle de captura se ha dado por vencido — comprueba esto, no solo `last_error`; permanece «True» tras un reinicio de la configuración, lo que señala `restarting`), `stalled` (True mientras el bucle de captura está en ejecución pero no ha publicado ningún grupo de fotogramas durante unos diez períodos de fotogramas, de 2 a 30 s; `last_error` solo lo indica tras 60 s), `last_frame_age_s`, `last_error`, `serials`, `ptp_enabled` (`True` solo cuando la matriz estaba conectada con PTP **y** cada miembro haya leído por última vez `PtpEnable` en; no es lo mismo que «bloqueado», que es lo que mide `ptp_epoch`), `ptp_configured` (lo que solicitó la conexión), `ptp_off_members` / `ptp_unread_members` (miembros que leyeron por última vez `PtpEnable` desactivado / no respondieron), `member_recovery` (`None`, o un miembro que ha perdido alimentación o su conexión: `state`, `waiting`, aunque no responda; `reopening`, luego `recovered`, junto con `members` y `why` — la matriz lo vuelve a incluir cuando vuelve a responder y le devuelve su PTP de tiempo de conexión, el cableado de disparador y línea de sincronización, el ritmo y los ajustes de la cámara, y luego vuelve a transmitir sin nuevos `connect_array`) y `ptp_epoch` (medido), `sync_mode` / `sync_tier`, `packed_transport` / `packed_decision`, `per_cam_emit_mbps`, `link_speed_mbps` (`4294` significa que Windows no ha podido leer la velocidad del adaptador —véase [Frecuencias de fotogramas esperadas](#expected-frame-rates)), `trigger_pipeline` / `pipeline_tune`, `radiometric_chain` (si la cadena de radiancia por banda es utilizable y qué miembro la ha interrumpido), y `band_clamp` —, mientras una vista en directo muestra la matriz: la proporción de píxeles iluminados que el desmezclador ha limitado a 0, por elemento (`members`: `{serial: [band0, band1, band2]}` en el orden de bandas R, G, B de la cámara) y, por banda, lo que muestra la imagen compuesta (`composite`: `{band name: share}`), junto con `warn`, la proporción que la interfaz gráfica de usuario marca como «Datos de banda de baja fiabilidad» (medidos aproximadamente una vez por segundo; `None` cuando no se está ejecutando la visualización en directo), y `member_ae` —por miembro—, la exposición (`exposure_us`) y la ganancia (`gain_db`) que contenía su último fotograma publicado, extraídos del propio fotograma (`read_nodes` de `ExposureTime` / `Gain` devuelve el último valor escrito mientras se ejecuta la exposición automática, no lo que está haciendo). |
| `set_member_settings(serial, **kwargs)` | Escribe los ajustes de una cámara miembro, tal y como `CameraSession.set_settings()` los recoge (exposición, ganancia, brillo objetivo, ...); la cámara miembro permanece en la matriz. Genera un error `ChlorosConnectError` si se trata de una serie que no es miembro. Véase [Exposición en miembros de la matriz](#exposure-on-array-members). |
| `capability(timeout=10.0)` | Lo que ha establecido la preparación inteligente (el nivel de captura, el tamaño de fotograma, `pixel_format`), el bloque `health` en tiempo real y el presupuesto de ancho de banda del host. Véase [Estado de la matriz](#array-health--which-subsystem-is-losing-frames). |
| `set_target_fps(fps, timeout=10.0)` | Cambia la frecuencia de activación mientras la matriz está en funcionamiento, sin necesidad de volver a conectarse. Un número la fija (el controlador de frecuencia no la modifica); `None` lo vuelve a poner en modo automático. Devuelve `{requested, target_fps, target_fps_auto, clamped}` — `clamped` es True cuando la frecuencia se ha reducido a lo que el host y la red pueden soportar. Véase [Captura por intervalos para levantamientos con drones](#interval-capture-for-drone-surveys). |
| `grab(processing="debayered", with_metadata=False, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, bit_depth=None, band=None, align=False, refl_max=None)` | Un fotograma de cada cámara, **todos a partir de un único disparador**, como matrices numpy (sin escribir archivos). Devuelve un `GrabResult` (un `dict`): `{serial: array}`, o `{serial: (array, metadata)}` con `with_metadata=True`. `.group_id` es el disparador que comparten los fotogramas; `.errors` asigna cada cámara omitida a la razón correspondiente (una cámara que no puede atenderse en ese disparador se omite, nunca se rellena con otra). `processing` es `raw` / `debayered` / `radiance` / `reflectance` / `display`. La misma lectura que la [`ArrayHandle.grab()`](#one-trigger-per-read--group_id) de la API del proyecto — una implementación compartida —, por lo que todas las palabras clave que aparecen allí (`band`, `align`, `bit_depth`, `refl_max`) funciona aquí. Requiere `numpy`. |
| `frame_stream(*, processing="debayered", fps=5.0, duration=None, count=None, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, with_metadata=False, bit_depth=None, band=None, align=False, refl_max=None)` | Iterador que devuelve un resultado `grab()` por tick (solo argumentos de palabra clave), con un ritmo de `fps`, limitado por `duration` (segundos) y/o `count` (`None` = sin límite). **Cada conjunto devuelto es un nuevo disparador:** un tick cuyo desencadenador más reciente sea el conjunto que ya ha generado espera al siguiente desencadenador en lugar de repetirlo, por lo que el flujo se ejecuta a la velocidad menor entre `fps` y la propia frecuencia de fotogramas de la matriz. Genera un `FrameFetchError` en lugar de permanecer en silencio: de inmediato cuando todas las cámaras rechazan, por cualquier motivo, el siguiente tick que se repetiría; tras 60 s de ticks sin fotograma; y tras 60 s sin ningún nuevo disparador («sin nuevos disparadores durante 60 s»). Misma implementación que `ArrayHandle.frame_stream()`. Véase [Fotogramas en directo en tu propio flujo de trabajo](#live-frames-in-your-own-pipeline). |
| `continuous(output_dir="output", *, format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, max_captures=None, duration_s=None, daq=None, daq_tolerance_ms=None, timeout=30.0)` | Inicia la captura continua propia del backend (el modo «Continuo» de la interfaz gráfica) → [`ContinuousHandle`](#continuoushandle). Guarda **todos** los grupos sincronizados que publica la matriz, tal y como lo haría `capture()` (con los mismos controles de exportación), hasta que se detenga o hasta que `max_captures` / `duration_s` (`None` = hasta que se detenga). Genera un error `ChlorosConnectError` si ya hay uno en ejecución para esta matriz. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, smart=False, after_seq=None, daq=None, daq_tolerance_ms=None, timeout=300.0)` | Un grupo de captura sincronizado. Devuelve un `CaptureResult` (lista de diccionarios de fotogramas + `.skipped` + `.failed` + `.group_seq` + `.daq_files` + `.daq_matches` + `.alignment` + `.oriented`). Pasa un resultado anterior`group_seq` como `after_seq` para obtener el siguiente grupo sincronizado después de él, en lugar de esperar a uno nuevo. Controles de exportación a continuación. |
| `capture(..., smart=True)` | **Captura inteligente**: espera a que el AE se estabilice en todas las cámaras y, a continuación, se activa. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, orient=None, timeout=120.0)` | Captura más rápida: solo datos sin procesar + la lectura DAQ asignada (+ el índice combinado libre). Equivale al botón «Captura más rápida» de la interfaz gráfica de usuario. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Única / Continua / Por intervalos en un bucle delimitado. Devuelve `list[CaptureResult]`.**Requiere `count` y/o `duration_s`** para que finalice (el SDK no dispone de Ctrl+C). También finaliza antes de tiempo, tras la pasada, cuando una pasada informa de que la unidad está llena (una entrada `.failed` con `disk_full`); esa pasada es el último resultado. |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, expr=None, timeout=30.0)` | Iniciar la grabación de la vista combinada en directo del índice en vídeo/GIF → `RecorderHandle`. El vídeo es un Motion-JPEG `.avi` (no se necesita ffmpeg). Si no hay ninguna vista de la interfaz gráfica abierta, el backend lo renderiza: `expr` (una expresión de índice en los nombres de las bandas de la matriz, p. ej., `"(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)"`), o el NDVI de la matriz por defecto. Una grabadora compuesta por cada matriz. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, orient=None, timeout=30.0)` | Iniciar una ráfaga de imágenes Bayer sin procesar a alta velocidad de fotogramas → `RecorderHandle`. Reprocesar fuera de línea con `build_video()`. `orient=True`: los productos que genera `build_video()` se giran según la orientación de la cámara (los productos de cada cámara según su propia orientación, el índice combinado según el de la cámara maestra); los fotogramas sin procesar se mantienen en su formato nativo y el manifiesto de la ráfaga registra la orientación de cada miembro. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Reprocesa fuera de línea una ráfaga RAW guardada para convertirla en uno o varios vídeos calibrados. Queda en espera hasta que finalice (`wait=True`) y devuelve el resultado de la compilación: `{success, outputs, errors, skipped, substitutions, notes, degraded, combined}` — véase [el resultado de la compilación](#the-build_video-result) a continuación. |
| `build_video_status(job_id, timeout=15.0)` | Consultar un trabajo de compilación fuera de línea: `{running, result, error, burst_dir}`. |
| `disconnect(keep_settings=False, clear_settings=False)` | Liberar toda la matriz — **si esta sesión la abrió** (`connect_array`): entonces pasa a ser `force_disconnect()`, y lanza una excepción `ChlorosConnectError` cuando no se confirma la liberación (la salida del bloque `with` nunca genera un error). En un identificador procedente de `attach_array`, deja la matriz en funcionamiento e imprime una línea en stderr indicándolo: la matriz puede pertenecer a la interfaz gráfica de usuario (GUI) o a otro script. Cada cámara se deja funcionando libremente y, por lo demás, tal y como la encontró la conexión; `keep_settings=True` guarda su configuración en la propia memoria de la cámara, mientras que `clear_settings=True` restablece esa memoria a los ajustes de fábrica (véase [Memoria de la cámara](#memoria-de-la-cámara--mantener-configuración--borrar-configuración)). |
| `force_disconnect(timeout=60.0, keep_settings=False, clear_settings=False)` | Libera todo el conjunto independientemente de quién lo haya abierto. Úsalo en un identificador asociado cuando lo que pretendas sea finalizar el conjunto (por ejemplo, para volver a conectarlo con una configuración diferente). Devuelve un resultado una vez que el backend confirma la liberación, o cuando no hay nada que liberar (el conjunto ya no existe o no hay ningún backend en ejecución); una liberación que ha agotado el tiempo de espera o que el backend ha rechazado genera el error `ChlorosConnectError`. Con cualquiera de los dos indicadores, devuelve lo que la memoria de cada cámara ha recibido (`camera_memory`); en caso contrario, devuelve `None`. |

Controles de exportación de `capture()` (el mismo punto final que utilizan la GUI y la CLI):

- `processing` / `levels` — `processing="all"` (o `levels=["raw","radiance",…]`) guarda todos los tipos de exportación aplicables por cámara; un único valor `processing` guarda solo ese nivel.
- `aligned` — **activado por defecto (`None`), ya que la captura de la aplicación Chloros del mismo conjunto es**(su configuración de captura**Alineada** por matriz). La exportación no sin procesar de cada miembro se adapta al fotograma de la cámara principal mediante el [perfil de alineación](#array-alignment) y recortada a la superposición común de los miembros, de modo que los archivos están alineados por píxeles y presentan el mismo tamaño, recorte y etiquetas `Alignment*` que los de la aplicación (dentro de una región de interés, la transformación se reajusta a la ventana); `XMP-mapir:AlignmentJSON` indica `"applied": true` con su `crop_roi`. El formato RAW permanece sin distorsionar en su tamaño nativo, pero lleva la transformación en los metadatos. `aligned=False` guarda cada archivo en su tamaño nativo, sin distorsionar; `aligned=True` es la configuración predeterminada. `continuous()`, `capture(smart=True)`, `capture_repeated()` y los de nivel de proyecto `ArrayHandle.capture()` / `capture_smart()` adoptan el mismo valor por defecto. Independientemente de si se aplica o no la deformación, cada archivo miembro registra la transformación `Alignment*` de su cámara una vez que la matriz tiene un perfil de alineación —excepto en el caso de una cámara cuyo perfil no se alinea (ya sea porque se ha omitido o porque se ha forzado a la identidad debido a un fallo en la resolución), así como la cámara de referencia cuando no hay ninguna otra cámara alineada: estas no llevan etiquetas `Alignment*`. Si la matriz no tiene perfil, la captura se guarda sin alinear, y el resultado lo indica: `result.alignment` es `{"requested": True, "applied": False, "warnings": [...]}`, una advertencia por cada motivo por el que no se pudo aplicar la deformación (ausencia de perfil, miembros con resoluciones diferentes, una matriz no sincronizable, un fotograma al que no se puede asignar el perfil). `applied` es `True` solo cuando todas las exportaciones que solicitaron la deformación la obtuvieron. La alineación automática de la matriz crea el perfil una vez que se estabiliza la exposición; `ArrayHandle.calibrate_alignment()` / el `align-calibrate` de la CLI crean uno bajo demanda. `result.oriented` indica si `orient=True` ha procesado los productos.
- `render_index=False`: omite la superposición del índice de vegetación por cámara; por defecto, se renderiza donde esté configurada.
- `daq` / `daq_tolerance_ms`: la lectura del DAQ se guarda como un archivo «sidecar» `.daq` de forma predeterminada, incluso cuando ningún nivel seleccionado lo requiera; `daq=False` lo omite en ese caso, y `daq_tolerance_ms` establece la tolerancia de coincidencia (por defecto, 500). `force_daq=True` es la grafía anterior del valor por defecto. Véase [La lectura del DAQ que guarda una captura](#el-daq-que-lee-una-captura-guarda--archivos-daq-y-coincidencias-daq).
- `orient=True` — activa los **productos**de cada miembro (`display`, `radiance`, `reflectance`, el índice por cámara y la composición) por la orientación de la cámara, tras la deformación de alineación: por la orientación de la**cámara principal**cuando se transformaron a su sistema de referencia (el valor por defecto de `aligned` —los miembros permanecen co-registrados—), o bien por la de sus propias cámaras.**Por defecto: cada archivo de miembro en formato nativo**, como siempre ocurre con las capturas en matriz; `raw` y `debayered` permanecen en formato nativo en cualquier caso. Cada archivo registra la orientación del fotograma en el que se encuentran sus píxeles (`XMP-Camera:ChlorosRotation` / `ChlorosFlip*` / `ChlorosOrientationApplied` — un archivo RAW, el de su propia cámara), y el XMP `Alignment*` conserva la transformación del fotograma nativo, aplicada antes de la orientación; un archivo RAW que contiene esa transformación también registra `ChlorosAligned*` (la del marco maestro), es decir, la orientación de un producto transformado al marco maestro. El procesamiento gira cada producto según la transformación que corresponde a sus píxeles: la alineada cuando transforma el archivo RAW (opción «Alineación de matriz» activada), y la propia de la cámara cuando no lo hace. `capture_repeated()` y `capture_fastest()` la dejan pasar; las de nivel de proyecto `ArrayHandle.capture()` / `capture_smart()` también la aceptan.

**Compresión TIFF (parámetro exclusivo de HTTP):** `ArraySession.capture()` no envía ninguna clave `compression`, por lo que se aplica el valor predeterminado del backend: `POST /api/camera/array/capture` lee un parámetro del cuerpo `compression`; por defecto, `"none"`: sin comprimir, los píxeles del sensor tal y como se leen (~6,3 MB por fotograma a resolución completa). `"deflate"` (zlib L1 sin pérdidas + predictor horizontal, ~4,1 MB/fotograma) escribe archivos más pequeños pero más lentamente; ambos son sin pérdidas y se leen de forma idéntica al importarlos. El SDK no expone ningún kwarg para ello; la vía de escape es `chloros-cli lattice array-capture --compression deflate` o HTTP sin procesar. DEFLATE retiene el GIL de Python, por lo que las escrituras comprimidas no se paralelizan entre los hilos de escritura de cada cámara; la captura sostenida a resolución completa a la velocidad del sensor requiere el valor predeterminado sin comprimir. Detalles: [Referencia de la CLI → array-capture](cli-reference.md).**Grupos consecutivos (protocolo HTTP):** el punto final devuelve `group_seq`; un bucle que lo reenvía como `after_seq` en su siguiente solicitud obtiene el siguiente grupo sincronizado —incluido uno publicado mientras se guardaba la captura anterior— —, en lugar de esperar a un grupo publicado después de la solicitud, y `skipped_groups` informa de cualquier grupo que el bucle haya omitido. `capture_repeated`, tanto el `--continuous` de la CLI como el modo continuo de la GUI hacen esto; los intervalos que han transcurrido se mantienen actualizados en cada momento programado.**Anulaciones de exportación por miembro (solo HTTP):**el mismo punto final también acepta `exclude_serials` (lista: excluir miembros del conjunto guardado; la matriz sigue activándose como un único grupo sincronizado y los miembros excluidos se devuelven en `excluded`), `serial_levels` (anulaciones a nivel de cámara `{serial: [level tokens]}`) y `serial_index` (anulaciones de superposición de índices por cámara `{serial: bool}`). Se trata de parámetros de paridad de la interfaz gráfica de usuario y**todavía no son argumentos clave del SDK**; los miembros que no aparecen en los mapas recurren a los valores globales de la matriz `levels` / `render_index`.

##### Inspección de cámaras omitidas — `CaptureResult.skipped`

`ArraySession.capture()` devuelve un `CaptureResult`, que es una subclase de `list`: recórrelo, indexa `len()` — todos los patrones existentes siguen funcionando. El nuevo código puede inspeccionar el atributo `.skipped` para ver qué cámaras se han excluido y por qué. El caso más habitual es el de las cámaras RGB en una matriz de filtros mixtos cuando se solicita `processing="radiance"` o `"reflectance"` — la radiancia por píxel Bayer carece de sentido para un sensor de banda ancha, por lo que el backend omite esas cámaras en lugar de generar datos sin sentido.

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

Los tokens de motivo siguen el patrón `<level>-not-applicable-to-rgb-cam` (una entrada por cada nivel omitido, cada una con `level`). Una cámara cuyo paquete de calibración no esté cargado omite `radiance` / `reflectance` con `<level>-needs-calibration (connect the camera so its cal pack loads)`, y, si no hay ninguna lectura de reflectancia del DAQ utilizable, omite con `reflectance-skipped-no-dls`. Las demás omisiones específicas de reflectancia son `reflectance-skipped-no-fresh-dls` (no hay disponible ninguna lectura reciente de la radiación descendente), `reflectance-skipped-bound-daq-unavailable (…)` (no se pudo acceder al DAQ vinculado) y `dls-uncalibrated-band-<nm>` —la banda se encuentra en su mayor parte fuera del rango radiométricamente calibrado del sensor de luz del DAQ (~374–974 nm), por lo que se rechaza la división de reflectancia absoluta basada en el DAQ y el fotograma recurre abiertamente a la respuesta del sensor. Entre las referencias de producto disponibles, solo el modelo F988 lo activa; la ruta compatible con esa cámara es el flujo de trabajo con panel de reflectancia.

##### Archivos que no se pudieron escribir — `CaptureResult.failed`

Una captura que solo pudo escribir algunos de sus archivos sigue devolviendo: la lista contiene todos los archivos que **se** escribieron, y `.failed` enumera el resto, un diccionario por archivo con `serial` (`None` para el compuesto combinado y el archivo auxiliar `.daq` de una matriz), `level` (`raw`, `radiance`, …, `index`, `composite`, `daq`, o `match` —la procedencia de `.daq` en `.match.json`, cuando se escribió `.daq` pero no su sidecar— y `error`. En una unidad llena, la entrada también incluye `disk_full: True`; cualquier captura posterior en esa unidad fallaría de la misma manera, por lo que `capture_repeated()` se detiene ahí. No se genera ninguna excepción en caso de captura parcial, y no quedó ningún archivo en `.failed` a medio escribir en el disco. `.failed` está vacío en una captura limpia.

```python
result = arr.capture("output/", processing="all")
for f in result.failed:
    print(f"SN:{f['serial']} {f['level']} not written: {f['error']}")
if any(f.get("disk_full") for f in result.failed):
    raise SystemExit("drive full -- free space and capture again")
```

##### El DAQ que lee una captura guarda: `.daq_files` y `.daq_matches`

Siempre que un DAQ está vinculado a la cámara o a la matriz, o bien está agrupado, cada captura guarda la lectura descendente que ha utilizado: un `.daq` en la subcarpeta `daq/` de la carpeta de la captura, junto con su `.match.json` y su `.cameras.json` (de la que procede la lectura). Esto se aplica también a las capturas sin procesar o sin debayering, y el archivo es el mismo independientemente de si la captura se ha realizado desde la interfaz gráfica de usuario (GUI), la interfaz de línea de comandos (CLI) o el kit de desarrollo de aplicaciones (SDK). `daq=False` omite este paso en el caso de una captura cuyos niveles no utilicen el DAQ (un nivel `radiance`, `reflectance` o `display` sí lo escribe); `daq_tolerance_ms` desplaza la línea de tolerancia para esa captura y nunca la lectura utilizada. ¿Qué DAQ? El vinculado a la cámara o a la matriz (la selección del sensor de luz de la interfaz gráfica de usuario, el enlace de un proyecto o `chloros-cli daq pool-bind`), o bien el más reciente que tenga el backend. La CLI especifica los controles `--no-daq` y `--daq-tolerance-ms` ([Referencia de la CLI](cli-reference.md#the-daq-reading-a-capture-saves)).

```python
r = cam.capture("out", levels=["raw"])      # a CameraSession; ArraySession results match
for m in r.daq_matches:                      # one per .daq in r.daq_files
    print(m["daq_file"], m["gap_ms"], m["in_tolerance"])
```

Una captura guarda la lectura del sensor de luz que ha utilizado como `.daq` junto con un `<file>.daq.match.json` que indica cómo se vinculó la lectura a la exposición: `method`, `gap_ms` (desde la exposición hasta la lectura DAQ más cercana) y `in_tolerance` (en un plazo de 500 ms, o el `daq_tolerance_ms` de la captura). El XMP del archivo de reflectancia recoge el mismo veredicto que `Chloros:DlsMatchMethod`, `Chloros:DlsMatchGapS` y `Chloros:DlsInTolerance`.

Una captura espera a la primera lectura del DAQ tras la exposición, para poder combinar las lecturas de ambos lados de la misma (`method` `interpolated`). Un DAQ-E envía una lectura cada 0,4 s a la luz del día y cada 1-2 s en interiores, por lo que `capture()` se devuelve hasta aproximadamente un intervalo y medio después; la exposición en sí no se retrasa. No se espera a un DAQ que haya dejado de enviar datos. Una lectura del DAQ-E es la media de la luz durante el intervalo transcurrido desde su lectura anterior, por lo que se sitúa en el centro de dicho intervalo: `gap_ms` se mide en ese punto medio, y el `.match.json`'`bracket_stamps_ns` proporciona las marcas de tiempo propias de las lecturas. `ArraySession.continuous()` espera sin ralentizar la captura. Las pasadas de idano esperan, por lo que mantienen su frecuencia y se emparejan con la lectura más cercana anterior a la exposición.

Una lectura nunca caduca, por lo que no se pierde ningún fotograma por una lectura tardía; en su lugar, se etiqueta como:

- **El flujo del DAQ se ha estancado**: se utiliza su lectura más reciente, y `gap_ms` indica su antigüedad (`in_tolerance: false`: más de 500 ms).
- **No hay DAQ que leer**— desconectado (`disconnect()`, `chloros-cli daq pool-disconnect`), o no se puede acceder al DAQ vinculado — se reutiliza la última lectura de la cámara y se describe en relación con**esta** exposición: `method`, `retained` y `gap_ms` de esta exposición, `in_tolerance` se vuelve a calcular, y `retained_from_epoch_ns` indica la exposición con la que se emparejó por primera vez. Una cámara que nunca haya tenido una lectura sigue sin tener reflectancia, y el resultado explica por qué (`reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, o el rechazo de la captura).

Los archivos que guarda una captura dentro de la carpeta del proyecto abierto en Chloros se añaden a la lista de archivos de ese proyecto, al igual que ocurre con una captura realizada en la aplicación (los archivos guardados en otros lugares no se ven afectados): `CameraSession.capture()`, `ArraySession.capture()` (también `smart=True`) y `capture_cameras_transient()`. El resultado así lo indica en `.project_add` (`capture_cameras_transient()`: el `project_add` del diccionario) -- `held`, `files` y el `message` del backend; `held` es `True` mientras se está ejecutando un proceso, y los archivos se incorporan una vez finalizado este. `None` cuando ningún archivo se ha incorporado al proyecto.

`CameraSession.capture()` y `ArraySession.capture()` devuelven como resultado `.daq_files` y `.daq_matches`, mientras que las respuestas de captura HTTP (`/api/camera/<serial>/cli-capture`, `/api/camera/capture`, `/api/camera/capture-all-synced`, `/api/camera/array/capture`) contienen `daq_files` y `daq_matches`, uno por cada `.daq`: `daq_file`, `cameras`, `method`, `gap_ms`, `in_tolerance`, `tolerance_ms` y `stream_fault` — el error de flujo propio del DAQ, que solo se consulta cuando una lectura está fuera de tolerancia. Antes de basarse en la reflectancia de una captura, compruebe `in_tolerance` en su `.match.json` o XMP, o en los [`status()`](#daqsensorsession-methods).

Niveles de `processing`:

| Nivel | Salida |
| --- | --- |
| `"raw"` | Bayer monocanal (cámaras monocromáticas: la banda única) directamente desde el sensor. |
| `"debayered"` *(valor predeterminado del SDK)* | Color de 3 canales: el desplazamiento oscuro (DSNU) del paquete de calibración se elimina del dato sin procesar y, a continuación, se aplica el demosaico con detección de bordes de OpenCV — sin LUT de linealidad, campo plano ni desmezclado (cámaras monocromáticas: escala de grises de un canal). El archivo TIFF guardado tiene el orden de canales **RGB**(RGB fotométrico, por lo que cualquier visor de imágenes muestra el color real); los matrices en memoria que devuelven `grab()` / `frame_stream()` tienen el orden**BGR** de OpenCV de OpenCV: invierte los canales (`frame[..., ::-1]`) para compararlos con un archivo guardado. `chloros-cli process` genera el mismo resultado que `Debayered_Images`. |
| `"radiance"` | float32 W/m²/sr/nm a través de la cadena radiométrica completa. Solo multiespectral: se omiten las cámaras RGB. |
| `"reflectance"` | uint16 0..32768 (compatible con Pix4D); requiere un emparejamiento DAQ en tiempo real para obtener una referencia absoluta. Solo multiespectral. |
| `"display"` | Cadena completa que coincide con la vista previa de la interfaz gráfica de usuario (CCM + WB + gamma según el perfil de la cámara). |
| `"all"` | **Un archivo por nivel aplicable** para cada cámara (que coincide con la opción «Capturar todo» de la interfaz gráfica de usuario / valor predeterminado de la CLI). El archivo `CaptureResult` devuelto contiene entonces un diccionario de fotogramas por cada `(cam, level)`, con el nivel en cada diccionario; los niveles no aplicables aparecen en `.skipped`. La lectura del DAQ utilizada para la captura se guarda como un archivo complementario `.daq`. |

> **Nota: el valor por defecto difiere del de la CLI.** `ArraySession.capture()` toma por defecto el valor de `processing="debayered"`; el comando `chloros-cli lattice array-capture` toma por defecto el valor de `processing="all"`. Pasa `processing="all"` explícitamente desde el SDK para reflejar el guardado multinivel de la CLI o la GUI.

##### Nombres de los archivos de captura de matrices

Cada nivel se guarda en su propia subcarpeta de `output_dir` (`raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/` para `display`, `index/`, `composite/`). Cada archivo lleva como nombre su cámara: `SN<serial>-<filter>`: el número de serie, seguido de la parte del filtro del modelo de la cámara — `SN214000533-FOCN` para una `M3C-L87-FOCN`, `SN261902142-F450` para una `M3M-L87-F450` — junto con `-`, por lo que el nombre tiene los mismos campos separados por «`_`» que antes. Una cámara cuyo modelo no se puede leer es simplemente «`SN<serial>`», y las capturas de versiones anteriores no incluyen ningún filtro en el nombre. El nombre del archivo depende del número de niveles que se le haya solicitado al miembro:

| Nivel solicitado | Nombre del archivo | Ejemplo (`reflectance`, por defecto `format="tiff"`) |
| --- | --- | --- |
| **Un nivel** (`processing="reflectance"`, o un `levels` de un solo elemento) | `sync_<timestamp>_SN<serial>-<filter>.<format>` — sin sufijo de nivel, como en versiones anteriores, por lo que los scripts existentes siguen funcionando | `reflectance/sync_<ts>_SN214000533-FOCN.tiff` |
| **Varios niveles** (`processing="all"`, o `levels` con más de un elemento) | `sync_<timestamp>_SN<serial>-<filter>_<level>.<format>` | `reflectance/sync_<ts>_SN214000533-FOCN_reflectance.tiff` |

La [máscara de recorte](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) va a continuación del nombre del producto, `clamp_masks/<product folder>/<product name>_clampmask.tif`: unacaptura de reflectancia de un solo nivel genera `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif`, mientras que `processing="all"` genera `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif`. La regla del sufijo se determina para cada miembro a partir de los niveles solicitados, por lo que un nivel omitido (por ejemplo, la reflectancia en una cámara RGB) no la modifica. Lee las rutas a partir de los fotogramas devueltos en lugar de generarlas. Una captura con una sola cámara (`CameraSession.capture()`) siempre añade el nivel: `capture_<ts>_SN<serial>-<filter>_<level>.tiff`.

##### Los formatos PNG y JPEG solo admiten la visualización y los productos de índice

Un archivo PNG o JPEG tiene 8 bits por canal. Si se solicita uno con un nivel que no puede admitir, la solicitud se rechaza antes de que se capture nada, con un mensaje en el que se indica el nivel y TIFF como el formato que lo admite:

- **`radiance`**(float32) y**`reflectance`**(uint16, `32768` = 1,0) — y, por tanto, `processing="all"`, que los incluye. El SDK comprueba esto en los argumentos: `capture()` genera un**`ValueError`** sin una solicitud (`CameraSession`, `ArraySession`, `CameraHandle`, `ArrayHandle`, `capture_smart`).
- **`raw` / `debayered` procedentes de una cámara que transmite más de 8 bits** (M3C de 12 bits, Mono12 M3M): el backend los rechaza, ya que conoce la profundidad de bits de cada cámara: Código 400 de HTTP con `error_code` y `FORMAT_CANNOT_HOLD_LEVEL`, generado como `ChlorosConnectError` (`CameraSession` / `ArraySession`) o `RuntimeError` (`CameraHandle` / `ArrayHandle`). La captura de un solo nivel `debayered` de una cámara obtiene un fotograma de 8 bits para PNG/JPEG y no se rechaza.

Guarda los datos radiométricos como `format="tiff"` (`ext=".tiff"`); utiliza PNG o JPEG para `display` y las representaciones de índice.

#### Cómo funciona la exposición automática

- **Primero la exposición, luego la ganancia.** Con la ganancia en «Auto», esta se mantiene en 0 dB hasta que la exposición alcanza el valor máximo permitido.
- **Una exposición manual deja la ganancia en «Auto»** hasta que se modifique la ganancia o se configure en modo manual. Al volver a «Auto», el límite de la ganancia es el mismo que antes.
- **Las pequeñas zonas de alta luminosidad no oscurecen la escena.** Se ignora un reflejo especular de hasta el 1 % del fotograma y, en los paneles de un objetivo de calibración conocido, hasta el 0,5 % de un panel.
- **El límite máximo de exposición del DAQ solo se aplica mientras el objetivo de calibración está a la vista.** Esto evita que un panel blanco se recorte; si no se ve ningún panel, la cámara expone la escena.
- **La luz constante se mantiene estable, y los cambios de luz se estabilizan en segundos.** Con luz constante, la exposición y su límite permanecen fijos, y tras un cambio importante en la luz, la imagen se estabiliza en pocos segundos. Un recorte ya no limita la exposición de forma permanente, y un cambio en la zona de interés (ROI) vuelve a ajustar la exposición.
- **El objetivo de brillo se mantiene al alcance.** No aumenta mientras la cámara se encuentra en su límite de exposición y su ganancia no puede subir, por lo que una luz que vuelve a aparecer no provoca un destello en el fotograma.
- **Una cámara cuya exposición automática deja de responder se mantiene en el nivel correspondiente a la luz en la que se encuentra.** Bajo un sol intenso, una cámara podría caer a su exposición más corta y permanecer ahí con un fotograma oscuro, aunque todos sus ajustes indiquen valores normales. Chloros supervisa el fotograma, mantiene esa cámara en el nivel que conservaba para esa luz y la libera cuando la luz cambia. Un sensor de luz oscuro u oculto no interfiere en este proceso.

#### Exposición en los miembros de la matriz

Una cámara LATTICE limita `ExposureTime` a un periodo de su frecuencia de fotogramas. Mientras una matriz transmite, Chloros ejecuta cada miembro con `AcquisitionFrameRateEnable` activado, a una frecuencia ligeramente superior a la frecuencia de disparo, de modo que el límite máximo de exposición de un miembro se sitúe un poco por debajo del periodo de disparo, **en cualquier ROI**:

- El periodo de fotograma de un miembro es el periodo de disparo menos un margen del 15 % del periodo (como mínimo 3 ms, como máximo la mitad del periodo), y su límite máximo de exposición es ese periodo de fotograma menos unos 0,1 ms. Ejemplo: un disparador de 10 fps (100 ms) da una frecuencia de miembro de unos 11,8 fps y un límite máximo de exposición de unos 85 ms.
- **Un `ExposureTime` manual más largo que ese límite máximo ralentiza el disparador para adaptarse a él.** Chloros primero abre el límite máximo de ese miembro para ese valor y deja que la frecuencia más lenta se ejecute durante unos cuantos períodos de disparo (la escritura puede tardar unos segundos más), y luego lo escribe, y el disparador no se activa más rápido de lo que permite la exposición. En una matriz de frecuencia automática, la frecuencia con la que funcionaba anteriormente se restablece una vez que todos los miembros cuya exposición manual la ralentizó vuelven a la exposición automática. Un valor que la propia cámara no puede aceptar sigue siendo rechazado, con el rango que acepta ahora en el error, y se restablece la exposición automática del miembro si se había desactivado únicamente para esa escritura. Antes de la versión 1.3.0, una pequeña región de interés (ROI) limitaba a cada miembro al periodo de funcionamiento libre del propio sensor, independientemente de la frecuencia de disparo: 4,9 ms a 768 × 304, por lo que no se podía aumentar el brillo de ningún miembro.
- Chloros nunca acorta una exposición manual para adaptarse a un disparo más rápido. Una frecuencia automática se mantiene dentro de lo que permiten las exposiciones manuales; una frecuencia fijada más rápida de lo que permite una exposición deja a ese miembro un límite máximo lo suficientemente largo para ello, y no admite disparos más rápidos de lo que permite su exposición. Reduce la exposición o la frecuencia fijada cuando todos los miembros deban realizar todos los disparos.
- **Lectura de retorno:** al leer los nodos de un miembro, se muestra `AcquisitionFrameRateEnable` True y `AcquisitionFrameRate` a la velocidad del miembro; la habilitación se desactiva cuando la matriz se detiene.
- **La matriz es propietaria de esos dos nodos. No escribas `AcquisitionFrameRateEnable` ni `AcquisitionFrameRate` en un miembro de la matriz**: ajusta lacon `target_fps` / `set_target_fps()`.

Escriba la exposición de un miembro con `arr.set_member_settings(serial, exposure_time=20000)` (las palabras clave de `CameraSession.set_settings()`; `chloros-cli lattice exposure --serial` hace lo mismo). El miembro permanece en la matriz. No abras un miembro de la matriz con `connect_camera()`: connect rechaza una cámara que ya se encuentra en una matriz, y el valor que devolvería `CameraSession` elimina la cámara de su matriz al desconectarse. `ArraySession.status()["member_ae"]` informa de la exposición de cada miembroa partir de su fotograma más reciente.

### Un registro de proyecto, para todos los modos

Una carpeta de proyecto contiene la configuración de cada dispositivo: `cameras.json` — las de cada cámara (`cameras[].settings`, para una sola cámara y para cada miembro de la matriz) y las de toda la matriz (`arrays[].capture`: formato, ROI, binning, frecuencia de disparo, presupuesto de cableado), así como `sensors.json`, los de cada sensor de luz (tiempo de integración, promedio de fotogramas, límite). La aplicación, la CLI, el SDK y el servidor MCP leen y escriben los mismos archivos, de la misma manera:

- **Un dispositivo conectado a un proyecto se guarda en él y registra en él cada cambio.** «En un proyecto» significa con un proyecto abierto en el backend (el de la aplicación) o —a través de un `ChlorosProject` (`open_project()`), que indica su propia carpeta al backend (`project_path`). Una cámara, una matriz o un sensor de luz que el proyecto aún no contenga se añade a sus archivos al conectarse (`connect_camera()`, `connect_array()`, la conexión del sensor de luz, la CLI, la aplicación, `chloros-mcp`), junto con su configuración. Un cambio desde cualquier modo (`set_settings()`, `ArraySession.set_member_settings()`, `set_target_fps()`, la configuración de la matriz, `set_cap()`, el CLI y los controles de la aplicación) se guarda en los archivos de ese proyecto un instante después. Una exposición, ganancia o objetivo de AE establecido en una cámara se guarda como «retenido», por lo que al volver a abrirla se mantiene de nuevo.
- **Al abrir un proyecto, se configuran sus dispositivos a partir de él**, en cualquier modo: una cámara, una matriz o un sensor de luz conectados que no tengan ajustes propios adoptan losguardados del proyecto (`lattice cam-connect` / `array-connect` con un proyecto abierto, `chloros-cli project connect`, `open_project().connect_all()`, al volver a abrir la aplicación). Una matriz conectada con un nuevo ID actualiza la matriz que el proyecto guardó con las mismas cámaras; una matriz con otras cámaras se añade junto a ella.
- **El propio guardado de la aplicación nunca sobrescribe los ajustes en tiempo real de un dispositivo** con su copia anterior de los mismos, y conserva las modificaciones por miembro que haya guardado una llamada desde la CLI o el SDK.
- **El formato de píxeles que se configure es el que conserva el proyecto.** Un formato establecido en la configuración de la matriz, mediante `--pixel-format` o `connect_array(pixel_format=)`, incluido el de 8 bits, se guarda y se vuelve a conectar exactamente igual. Si no se establece ninguno, cada cámara se conecta con su valor predeterminado de 12 bits (`BayerRG12`, `Mono12` en un sensor monocromo). El nombre de un formato indica su profundidad de bits; la familia es la del sensor, por lo que una cámara en color nunca se configura en un formato monocromo ni una cámara monocroma en uno de tipo Bayer, y la línea `[PixelFormat <serial>] ... written` lo indica cuando se ha asignado un formato. Las exportaciones de una captura de 8 bits son de 8 bits, incluida la reflectancia: reescaladas y marcadas con su `PixelScale`.
- Un dispositivo conectado sin ningún proyecto registrado nada.
- **Una grabación se adjudica a un proyecto creado después de ella; al abrirla, se mantiene en su proyecto.**Cuando se crea un proyecto — `create_project()`, `chloros-mcp` de `create_project`, o la opción**Nuevo proyecto**de la aplicación — o se duplica (la opción**Duplicar**de la aplicación, que abre la copia) mientras una grabación sigue escribiendo en el proyecto que estaba abierto anteriormente, la grabación continúa y lo que escribe acaba en el nuevo proyecto, en el mismo lugar dentro de este (abajo). Al abrir un proyecto existente — `ChlorosProject.process()` (que abre su proyecto en segundo plano), `chloros-cli process`, la opción**Abrir proyecto** de la aplicación — deja una grabación aún en curso con el proyecto en el que se graba: esta finaliza allí, y sus archivos se incorporan a la lista de archivos de ese proyecto (la aplicación detiene todas las grabaciones antes de abrir un proyecto).
  - `ArraySession.record()` y `ArraySession.burst()`: el vídeo, o la carpeta de ráfaga con todos los fotogramas, se traslada allí íntegramente cuando se detiene la grabación. El resultado de la detención indica la nueva ruta (`out_dir`, `video_path`), y a partir de ella se genera una secuencia a partir de la ráfaga. Si la unidad rechaza el traslado, la grabación permanece donde se escribió, y el resultado así lo indica (`relocation_error`). Los fotogramas sin procesar de una ráfaga detenida y `.daq` se añaden a la lista de archivos del proyecto en el que acaba; no se añade ningún vídeo a la lista de archivos del proyecto.
  - `ArraySession.continuous()`: todas las capturas posteriores al cambio se guardan en el nuevo proyecto; las guardadas anteriormente permanecen donde están.
  - `DAQSensorSession.record_start()` (y la grabación del sensor de luz de la propia aplicación): el `.daq` se traslada a la carpeta `light_sensor` del nuevo proyecto cuando se detiene la grabación; `record_stop()` devuelve la nueva ruta, y el archivo se incorpora a la lista de archivos de ese proyecto.

  Una grabación en una carpeta ajena al proyecto abierto permanece donde está, y los trabajos `start_camera_continuous()` / multihilo conservan la carpeta que se les asignó.
- **En la aplicación, las capturas realizadas tras crear un nuevo proyecto se guardan en él.** Al crear o duplicar un proyecto, todas las capturas en curso y las grabaciones continúan, y cada captura realizada a partir de ese momento («Capturar todo», «Iniciar captura») se guarda en el nuevo proyecto y se añade a su lista de archivos. Al abrir un proyecto existente en la aplicación, primero se detienen todas ellas.
- **Un proyecto no se mueve mientras se está grabando.**Al mover el proyecto abierto a otro directorio de trabajo (en la aplicación, en**Configuración → Directorio de trabajo**; ninguna llamada al SDK mueve un proyecto, un cliente HTTP que envía datos a `/api/move-project` sí lo hace) se rechaza con el código HTTP 409 `PROJECT_RECORDING` mientras una grabación en matriz o en ráfaga, una captura continua en matriz, un trabajo continuo o multitransmisión de una cámara o una grabación del sensor de luz esté escribiendo en él: `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.`

Esto es independiente de cómo quedan las cámaras al desconectarse (véase [Memoria de la cámara](#camera-memory--keep_settings--clear_settings)): un proyecto configura sus propias cámaras al abrirse.

### Memoria de la cámara — `keep_settings` / `clear_settings`

Cuando se conecta una matriz, sus cámaras adoptan la configuración de la matriz, y a continuación puedes modificarla. Cuando Chloros libera las cámaras — `ArraySession.disconnect()` / `force_disconnect()`, al final de una función transitoria, al detenerse `start_camera_continuous()` —, devuelve cada una al estado en el que se encontraba antes de la conexión: su cableado de disparador y línea de sincronización, así como los ajustes de PTP y de la cámara — XPTX001665 / XPTX001666 (y un modo manual `ExposureTime` / `Gain`), `TargetBrightness` y los límites y amortiguación del AE, el balance de blancos, el nivel de negro, `PixelFormat`, agrupación de píxeles y la zona de interés (ROI), el proceso de color (`GammaEnable` / `Gamma`, `LUTEnable`, `ColorTransformationEnable`), `ReverseX` / `ReverseY`, `UserSetSelector`, los PTP `PtpSlaveOnly` / `PtpDomainNumber`, el tamaño del paquete (`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`), la cadencia (`DeviceLinkThroughputLimitMode` / `DeviceLinkThroughputLimit`, `GevSCPD`, `GevSCFTD`), los datos del fragmento (`ChunkModeActive`, cada `ChunkEnable` bajo su `ChunkSelector`), la eliminación de interferencias de la entrada de disparo; una matriz proporciona la línea de sincronización de un esclavo (`LineFilterSelector` / `LineFilterWidth`) y la velocidad de fotogramas de adquisición (`AcquisitionFrameRate` solo cuando su habilitación estaba activada: con ella desactivada, la frecuencia se ajusta al máximo que permite la sincronización, no a un valor configurado). Solo se registra lo que difiere: la cámara se detuvo primero, la sincronización y la frecuencia de fotogramas antes de una exposición que requiere su periodo de fotograma.

Para utilizar las cámaras fuera de un proyecto, dos argumentos escriben en la **propia memoria** de la cámara —con lo que se inicia— en cualquier software (en `disconnect()` / `force_disconnect()`, todas las funciones transitorias, `start_camera_continuous()` y `ChlorosProject.disconnect_all()`; MCP: los mismos argumentos que `disconnect_device` y las herramientas transitorias; CLI: `--keep-settings` / `--clear-settings`):

- **`keep_settings=True`** guarda en su memoria la configuración de cada cámara tal y como quedó al finalizar la sesión (UserSet2, que se convierte en la configuración de arranque). Las cámaras siguen funcionando de forma autónoma (`TriggerMode` `Off`), con el cableado de la línea de sincronización tal y como se encontró. Una cámara sin UserSet2 guarda la configuración en UserSet1; este ordenador lo recuerda, por lo que la autorreparación de la matriz de Chloros —que restablece un UserSet1 que se ha desviado de la línea de base de Chloros— no la afecta.
- **`clear_settings=True`** restablece la memoria de cada cámara a los ajustes de fábrica: la línea de base de Chloros, aquella que la autocorrección de su matriz integra — valores predeterminados de fábrica con `TriggerMode`, `Off`, `TriggerSource`, `Line2`, exposición automática y el balance de blancos calibrado de una cámara RGB se mantiene — en UserSet1, que se convierte en el conjunto de arranque.
- No ambos (`ValueError`; el backend responde con un 400). La respuesta incluye `camera_memory`, por cada cámara `{"saved_to": "UserSet2" | "UserSet1" | None, "errors": [...]}` o `{"cleared": bool, "errors": [...]}` — `force_disconnect()` lo devuelve; para un trabajo, viene con `stop()`.

La aplicación no ofrece estos valores.

**Cada salida restaura las cámaras a su estado original.**La restauración tal y como se encontró se ejecuta en cada ruta de salida: `CameraSession.disconnect()` (la cámara**X**de la aplicación), el**X** de la aplicación en una matriz o en cualquier elemento de la matriz (se libera toda la matriz), el cierre de la aplicación (el backend dispone de hasta 20 s para restablecer todas las cámaras, matrices y grabaciones continuas en ejecución; una función transitoria o una llamada de un solo uso que siga ejecutándose se detiene en su siguiente punto seguro, restablece primero sus cámaras, durante un máximo de 8 s, y responde con un HTTP 503 `SHUTTING_DOWN`; un `connect_array()` que siga en ejecución se detiene antes de su siguiente cámara y restaura las cámaras que ha abierto, durante un máximo de 10 s, y genera un `ChlorosConnectError` (HTTP 503 `SHUTTING_DOWN`); y una que llega durante el apagado no abre nada: HTTP 503 `SHUTTING_DOWN`), y un comando `chloros-cli` que se interrumpe o cuyo terminal se cierra (su tarea se detiene y sus cámaras se devuelven en primer lugar). Un backend que se bloquea o se cierra de forma forzada no puede devolver nada; un reinicio devuelve cada cámara a su memoria.**Un proyecto configura sus propias cámaras.** Cuando una matriz se conecta a través de un proyecto —el proyecto abierto de la aplicación o un `ChlorosProject`, que se identifica ante el backend (`project_path`)—, cada miembro obtiene los ajustes que el proyecto ha guardado para él: un valor de exposición, ganancia o objetivo de exposición automática (AE) fijado para él (considerado como propio, por lo que la exposición automática no lo modifica) y su modo de balance de blancos (corrección de Chloros; el balance de blancos propio de la cámara permanece desactivado), sea cual sea el contenido de la memoria de la cámara.

### Modos de captura y grabadoras

La superficie de la matriz refleja el panel de captura de la interfaz gráfica de usuario: modos de obturación Única / Continua / Intervalo / Más rápida, además de dos grabadoras (vídeo compuesto en directo y ráfaga en formato RAW → reprocesamiento fuera de línea).

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

    # Continuous — every synced group for 60 s, saved by the backend itself
    with arr.continuous("run1/", processing="raw", duration_s=60) as run:
        run.wait()
    print(run.result["captures"], run.result["captures_per_s"],
          run.result["stop_reason"])

    # Combined-index video/GIF recorder: the GUI's combined view, or with no
    # GUI open the array's NDVI (pass expr="..." for another index)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    if rec.result and rec.result.get("error"):   # leaving the block does not raise
        print("recording failed:", rec.result["error"])
    else:
        print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**es el bucle Continuo/Intervalo del SDK. Como no existe `Ctrl+C` para interrumpirlo desde un script,**debes** pasar `count` y/o `duration_s` (se detiene cuando se alcanza cualquiera de ellos). `interval_s` se mide desde el inicio de cada pasada (al igual que en la interfaz gráfica). Las pasadas continuas (`interval_s=0`) encadenan `group_seq` → `after_seq`, por lo que son grupos sincronizados consecutivos a la velocidad de la matriz. Los demás kwargs se pasan directamente a `capture()`.
- **`continuous`**es la captura continua propia del backend: el modo «Continuo» de la interfaz gráfica de usuario.**Úsala en lugar de `capture_repeated()` cuando quieras que cada activación se guarde en disco a una alta frecuencia de fotogramas:** `capture_repeated()` realiza una solicitud HTTP por grupo desde tu script, mientras que `continuous()` mantiene dos capturas en curso dentro del backend sin ida y vuelta entre grupos. Medido en ROI 512 en una matriz de 9 cámaras: 73-77 capturas/s, frente a una matriz que publica unas 78. Mantén `capture_repeated()` para la captura por intervalos (`interval_s`) y para las pasadas que se revisan una por una. En Windows, excluye el backend de Microsoft Defender para obtener altas velocidades (véase [Paso 1](#step-1--install-the-chloros-platform-package)).
- **`record`** es de *nivel de monitorización*: registra la composición del índice combinado renderizada en tiempo real. Con la vista combinada de la interfaz gráfica de usuario abierta, graba esa vista tal y como se muestra. Si no hay ninguna abierta, el backend renderiza la vista por sí mismo: `expr` si se le pasa una; de lo contrario, el NDVI de la matriz (las bandas roja y NIR más cercanas a 660 nm y 850 nm, tal y como las capta la vista en directo), una vez alineada la matriz. Nombra las bandas en `expr` tal y como las nombra la matriz: `<band>_<filter>` para una cámara M3C (`NIR_850_RGN`, `Red_660_RGN`, `Orange_615_OCN`), `Mono_<filter>` para una M3M (`Mono_850`); un nombre de color corto (`NIR`, `Red`) solo funciona cuando una banda de la matriz lo tiene. Una vista de la interfaz gráfica de usuario (GUI) abierta durante la grabación se graba mientras permanece abierta. Una grabación que no escribe ningún fotograma falla con los errores `error` y `"no-frames: …"` (es posible que la matriz aún esté alineándose o estabilizando su exposición), y se genera el error `stop()` (véase [`RecorderHandle`](#recorderhandle)). Un grabador compuesto por matriz (genera un error si ya hay uno en ejecución).
- **`burst` → `build_video`** es de *calidad analítica*: `burst` escribe fotogramas sin procesar + un manifiesto por fotograma + un `.daq` por cada lectura DLS distinta bajo `<output>/bursts/<base>/` a la velocidad máxima del bucle de captura (sin cadena, sin exiftool, sin vista en directo). `build_video` sincroniza temporalmente cada fotograma con el `.daq` más cercano y vuelve a ejecutar la cadena de radiancia/reflectancia/índice del proceso de importación. `products` es una lista de `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}` (por defecto: el índice combinado). `burst().stop()` también inicia automáticamente la creación de un, que se devuelve como `build_job` en el resultado de la parada.

##### El resultado `build_video`

| Clave | Significado |
| --- | --- |
| `success` | `True` solo cuando todos los productos solicitados se han generado según lo solicitado: al menos una salida, nada en `skipped`, y no en `degraded`. |
| `outputs` | Rutas de los vídeos (y GIF) grabados. |
| `errors` | Problemas, en forma de texto. |
| `skipped` | Un `{"serial", "level", "reason"}` por cada producto solicitado que **no** haya generado ningún vídeo —ya sea por (sin paquete de calibración para esa cámara, sin lectura DLS de reflectancia, fallo en la cadena de procesamiento) o del índice combinado, que incluye `"kind": "combined"`, `serial` y `None` (no hay ningún par de bandas de rojo/NIR entre las cámaras, ni ningún grupo con radiancia para ambas bandas). Cualquier entrada genera `success` y `False`; los demás vídeos se siguen grabando y mostrando en la lista. |
| `substitutions` | Cámaras renderizadas a un nivel diferente al solicitado, y por qué: una cámara RGB para la que se ha solicitado radiancia/reflectancia obtiene su vídeo de visualización, con balance de blancos cuando está disponible el paquete de calibración de la cámara. No se trata de un error. |
| `notes` | Texto informativo, no se trata de problemas: por ejemplo, una cámara sin expresión de índice específica no tiene vídeo de índice, o el vídeo de visualización de una cámara RGB no tiene el balance de blancos ajustado (no hay paquete de calibración o este carece de balance de blancos de fábrica), indicando el nombre de la cámara. No es un error. |
| `degraded` | `True` cuando se ha escrito un producto pero no es el solicitado: el índice combinado fuera de línea es la diferencia normalizada integrada entre rojo y NIR, por lo que no se evalúa una expresión de índice configurada. Da lugar a `success` `False`. |
| `combined` | Resumen del índice combinado: las bandas utilizadas, el rango de la LUT, los fotogramas generados. |
| `clamp_masks` | Rutas de las [máscaras de recorte de desmezclado](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) de losTIFF calibrados fotograma a fotograma (`save_tiffs=True`), en `clamp_masks/<level>/` junto a la carpeta `<level>/` de los TIFF, uno por cada TIFF en el que se ha limitado un píxel. Un vídeo no contiene ninguno. |
| `clamp_fractions` | Lo que indica un vídeo en lugar de una máscara: un `{"kind", "serial", "level", "frames"}` por producto de vídeo (`kind` `"combined"` para las cámaras que forman parte del índice combinado), `frames` una lista de `[frame, R, G, B]` —la proporción de píxeles de ese fotograma que el proceso de desmezclado ha limitado en la banda que ocupa su canal R / G / B (`Chloros:Band{R,G,B}UnmixClampFraction`). |

#### `RecorderHandle`

Devuelto por `ArraySession.record()` y `ArraySession.burst()`. Úsalo como gestor de contexto para detenerse automáticamente al salir del ámbito, o contrólalo manualmente.

| Miembro | Descripción |
| --- | --- |
| `job_id` | ID del trabajo de backend (cadena). |
| `kind` | `"composite"` (de `record`) o `"raw"` (de `burst`). |
| `start_stats` | El diccionario devuelto por la llamada a `start`. |
| `result` | `None` durante la ejecución; el diccionario con el resultado final de la parada una vez detenida. |
| `stats(timeout=10.0)` | Estadísticas en tiempo real del trabajo (fotogramas grabados, fps alcanzados, tiempo transcurrido). |
| `stop(timeout=60.0)` | Detiene el grabador; devuelve y almacena en caché el resultado final (`.result` permanece como `None` si la propia solicitud de detención ha fallado). En el caso de una grabación `record()` que haya fallado —una que no haya escrito ningún fotograma (`error` `"no-frames: …"`) —, almacena en caché el resultado en `.result` y, a continuación, genera un error en `ChlorosConnectError`. Idempotente (una segunda llamada devuelve el resultado almacenado en caché, sin generar un error). Salir de un bloque `with` detiene la grabadora sin generar un error, así que comprueba `rec.result.get("error")` después de ello. |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

#### `ContinuousHandle`

Devuelto por `ArraySession.continuous()`. Úsalo como gestor de contexto — salir del bloque `with` detiene la captura si aún se está ejecutando — o contrólalo con `status()` / `wait()` / `stop()`. Solo se ejecuta una captura continua por matriz a la vez.

| Miembro | Descripción |
| --- | --- |
| `start_status` | El diccionario de estado devuelto por la llamada de inicio. |
| `result` | `None` durante la ejecución; el diccionario de estado final una vez finalizada la captura (establecido por `stop()`, o por un `status()` / `wait()` que haya detectado su finalización). |
| `status(timeout=10.0)` | Diccionario de estado en tiempo real (campos a continuación). |
| `wait(timeout=None, poll_s=0.5)` | Bloquear hasta que la captura finalice por sí sola —se haya alcanzado `max_captures` o `duration_s`, se haya agotado la matriz, se haya llenado la unidad o se haya producido un error— y devolver el estado final. Con `timeout`, devuelve el estado más reciente tras ese número de segundos, incluso si la operación sigue en curso: comprueba `running`. |
| `stop(timeout=30.0)` | Detiene la captura y devuelve su estado final. Idempotente (una segunda llamada devuelve el resultado almacenado en caché). |

Campos de estado:

| Campo | Significado |
| --- | --- |
| `running` | Verdadero mientras se ejecuta la captura. |
| `captures` | Grupos sincronizados guardados. |
| `files` | Archivos escritos. |
| `failed_files` | Archivos que no se han podido escribir (como en [`CaptureResult.failed`](#archivos-que-no-se-han-podido-escribir--captureresultfailed)). |
| `failed_captures` | Intentos de captura que fallaron en su totalidad, incluidos aquellos en los que se volvió a intentar la captura (un error HTTP503 mientras la matriz no tenía ningún grupo nuevo a tiempo). |
| `skipped_groups` | Grupos publicados por la matriz que no se guardaron (la captura se retrasó). |
| `captures_per_s` | Tasa media de guardado desde el inicio. |
| `elapsed_s` / `started_at` | Segundos transcurridos hasta ahora; hora de inicio (segundos de la época Unix). |
| `last_group_seq` | El último grupo sincronizado guardado. |
| `last_error` | El último error, o `None`. |
| `stop_reason` | Una vez finalizado: `count` (se ha alcanzado `max_captures`), `duration` (se ha alcanzado el `duration_s`), `stopped` (se ha llamado al `stop()`), `array_gone` (la matriz se ha desconectado), `error` (20 fallos consecutivos o una solicitud rechazada por el backend), `disk_full` (un archivo para el que la unidad no tenía espacio). `None` durante la ejecución. |
| `max_captures` / `duration_s` | Los límites con los que se inició (`0` = ninguno). |
| `defender_cores` | Núcleos de CPU utilizados por Microsoft Defender durante los últimos 10 s; `None` hasta que se mida uno, y siempre `None` en un host sin Defender. |
| `defender_warned` | Se establece en «True» una vez que la captura ha registrado que Defender la está ralentizando (en 1 núcleo o más). Excluye el backend del análisis de Defender; consulta [Paso 1](#step-1--install-the-chloros-platform-package). |

```python
run = arr.continuous("captures/run1", processing="raw", max_captures=5000)
try:
    while True:
        st = run.wait(timeout=5)              # returns every 5 s while running
        print(f"{st['captures']} captures, {st['captures_per_s']:.1f}/s, "
              f"{st['skipped_groups']} skipped")
        if not st["running"]:
            break
finally:
    final = run.stop()                        # no-op once it has ended
if final["failed_files"] or final["stop_reason"] in ("disk_full", "error"):
    raise SystemExit(f"continuous capture: {final['stop_reason']}: "
                     f"{final['last_error']}")
```

### Tramas en tiempo real en tu propio flujo de trabajo

Para enviar fotogramas sincronizados a tu propio código —aprendizaje automático, un visor personalizado, tu propia grabadora—, utiliza `ArraySession.frame_stream()`. Cada tick genera un `GrabResult`: `{serial: numpy array}` para cada cámara, **todo a partir de un único disparador**, con el ID de dicho disparador en `.group_id`. Se trata de la misma implementación que el `ArrayHandle.frame_stream()` de la API del proyecto, por lo que todo lo que se describe en [Un disparador por lectura](#one-trigger-per-read--group_id), [`band`](#exporting-a-single-band--band), [`align`](#aligning-the-streams--align) y [`bit_depth`](#bit_depth--trading-precision-for-bandwidth) se aplica a ambos.

```python
import chloros_sdk
from chloros_sdk import FrameFetchError

serials = ["213800234", "214000533", "214701288", "214701292"]   # master first

with chloros_sdk.connect_array(serials) as arr:
    try:
        for frames in arr.frame_stream(processing="raw", fps=10, duration=300,
                                       with_metadata=True):
            if not frames:              # no camera answered this tick
                continue
            if frames.errors:           # cameras missing from this trigger
                print("left out:", {s: str(e) for s, e in frames.errors.items()})
            for serial, (img, hdr) in frames.items():
                # img: numpy array (raw: one Bayer or mono plane)
                # hdr: group_id, frame_id, timestamp_ns, exposure_us, gain_db, ...
                my_pipeline(frames.group_id, serial, img, hdr["timestamp_ns"])
    except FrameFetchError as exc:      # the stream cannot continue
        print("stream ended:", exc.status, exc)
```

- **Cada conjunto generado constituye un nuevo disparador.** `frames.group_id` nunca se repite dentro de un flujo; cuando se solicita más `fps` del que admite la matriz, el flujo espera al siguiente disparador, por lo que se ejecuta a la velocidad más lenta de las dos.
- **Las cámaras que faltan se nombran, no se rellenan.** Una cámara a la que no se ha podido dar servicio en ese disparador está ausente del diccionario y aparece en la lista de `frames.errors`.
- **Finaliza de forma llamativa.** `FrameFetchError`, en caso de rechazo, se repetiría en el siguiente tick, tras 60 s sin fotogramas o tras 60 s sin un nuevo disparador.**Qué ofrece.**`frame_stream()` transfiere fotogramas numpy completos a través de la conexión HTTP local y alcanza un máximo cercano a los**170 MB/s**. Esto es suficiente para el procesamiento en tiempo real de fotogramas sin procesar, o de una matriz agrupada o de una región de interés (ROI), pero no para cada disparo a resolución máxima; para conservar**cada** disparo, guárdalo en el disco con [`continuous()`](#continuoushandle). Medido en una matriz de cuatro cámaras (FOCN, FRGN, FRGB, F450) con transporte empaquetado; cada tick era un conjunto completo de un disparador:

| | Sensor completo (matriz 14,2–14,4 fps) | Agrupación 2 × 2 (matriz ~33 fps) |
| --- | --- | --- |
| `frame_stream(processing="raw")` | 6,7 conjuntos/s | 22,2 conjuntos/s |
| `frame_stream(processing="debayered")` | 2,3 conjuntos/s | 9,3 conjuntos/s |
| `frame_stream(processing="display")` | 0,8 conjuntos/s | 3,0 conjuntos/s |
| MJPEG por cámara, 4 flujos a 720p | 13,5 fps cada uno; cada fotograma sincronizado con un disparador | ~25 fps cada uno; ~45 % de los disparadores presentes en los 4 |

Utiliza `processing="raw"` para el rendimiento y el demosaico por tu parte; el nivel `debayered` también elimina el desplazamiento oscuro del paquete de calibración, algo que no hace el demosaico de `raw`.

#### Flujos MJPEG por cámara (sin RTSP)

**No se admite RTSP.** La interfaz de transmisión es MJPEG sobre HTTP (`multipart/x-mixed-replace`, un JPEG por parte, cada parte con un `Content-Length`), una URL por cámara:

| URL | Qué ofrece |
| --- | --- |
| `GET /api/camera/stream/<serial>` | Vista previa de 8 bits, hasta ~30 fps. Para un miembro de la matriz, `max_h` (por defecto 720, 64–2160) y `jpeg_q` (por defecto 80) definen el tamaño y la calidad. |
| `GET /api/camera/<serial>/stream-annotated` | La misma vista previa con las superposiciones de la interfaz gráfica de usuario dibujadas sobre ella (zebra, cuadrícula, retículo, histograma, picos, medidor puntual). `ChlorosLocal.camera_stream()` lo lee por ti. |
| `GET /api/camera/<serial>/stream-reflectance` | Reflectancia de 8 bits por banda — véase [Flujo de reflectancia en directo](#live-reflectance-stream-mjpeg). |

- **Encabezados de disparo.** Para un miembro de la matriz, cada parte de las tres transmisiones lleva `X-Chloros-Group-Id`, `X-Chloros-Frame-Id` y `X-Chloros-Timestamp-Ns`. Las partes procedentes de cámaras diferentes con el mismo `X-Chloros-Group-Id` pertenecen al mismo disparador: recoge las partes y agrúpalas según ese encabezado para volver a montar un conjunto. Una cámara independiente no tiene grupo: sus partes de vista previa no llevan ninguno de estos encabezados (sus piezas de reflectancia llevan un `X-Chloros-Group-Id` vacío).
- **La transmisión de vista previa omite fotogramas bajo carga.** Muestra el fotograma más reciente y omite cualquier fotograma que no haya podido enviar a tiempo, por lo que, a altas velocidades de matriz, no todos los disparadores llegan a la transmisión de todas las cámaras (véase la tabla anterior). Para cada activador, utiliza `frame_stream()` o `continuous()`.
- **`/api/camera/stream/<serial>` se detiene mientras la transmisión en directo combinada de la matriz está abierta** (`/api/camera/array/stream/<array_id>`: la vista de matriz de la interfaz gráfica de usuario la abre). A continuación, la transmisión de cada miembro envía un fotograma marcador de posición «Pausado (transmisión de la matriz activa)», sin encabezados `X-Chloros-*`, y nada más hasta que se cierre la transmisión combinada; entonces se reanuda por sí sola. `stream-annotated` y `stream-reflectance` no se detienen. Cierra la vista de matriz de la interfaz gráfica de usuario cuando un script lea las transmisiones de vista previa de cada cámara.
- **Para números, en lugar de imágenes, utiliza `frame_stream()`.**El formato JPEG tiene pérdida y es de 8 bits; `frame_stream()` devuelve los mismos productos sin ningún códec en la ruta.

### Captura a intervalos para levantamientos con drones

Un vuelo de levantamiento necesita un grupo de imágenes sincronizadas cada pocos metros de recorrido, no todos los fotogramas que la matriz pueda producir. Hay dos ajustes que controlan esto:**`target_fps`**establece la frecuencia con la que las cámaras capturan un fotograma, y**`capture_repeated(interval_s=…)`** establece la frecuencia con la que se guarda un fotograma.**Las cámaras siguen transmitiendo entre guardados.** En cada intervalo se guarda el grupo sincronizado más reciente; los fotogramas intermedios no se graban. La exposición automática se ajusta en cada fotograma, por lo que la exposición ya se ha estabilizado cuando se produce el guardado. No existe ningún modo en el que las cámaras se detengan entre disparos y se activen para cada uno de ellos: en ese caso, cada disparo tendría que esperar a que comenzara la transmisión y se estabilizara la exposición.**Fija la frecuencia de fotogramas a la que requiera el intervalo.**Si se deja en modo automático, una matriz funciona a la frecuencia más alta que admiten el host y la red. Fijarla a un valor inferior reduce el calentamiento de la cámara, la carga de la red y la CPU del host sin alterar lo que se guarda. Elige una frecuencia que ofrezca**entre 2 y 4 fotogramas por intervalo**:

- Cada guardado espera al siguiente fotograma tras su solicitud, por lo que un grupo guardado llega hasta un periodo de fotograma después del momento programado. Una frecuencia más alta mantiene ese intervalo reducido.
- La exposición automática se ajusta una vez por fotograma, por lo que una frecuencia muy baja ralentiza la rapidez con la que la exposición sigue el paso de una nube.

**Calcula el intervalo a partir del plan de vuelo:**

```
spacing    = along-track footprint × (1 − forward overlap)
interval_s = spacing / ground speed
```

Por ejemplo, una huella de 60 m con un solapamiento hacia delante del 80 % supone una separación de 12 m; a 10 m/s, eso da `interval_s=1.2`, y un `target_fps=3` fijado da entre 3 y 4 fotogramas por intervalo. La captura se basa en el tiempo: el SDK no dispone de modo activado por distancia ni por GPS, por lo que hay que mantener una velocidad respecto al suelo constante en las líneas de levantamiento.

**Guarda los datos sin procesar en la aeronave y calíbralos más tarde.** `interval_s` se mide desde el inicio de cada guardado, por lo que un guardado que tarde más que el intervalo hace que el siguiente comience tarde y que la separación se desvíe. Al guardar los datos sin procesar junto con la lectura del sensor de luz (`processing="raw"`; la lectura se guarda por defecto), cada guardado es breve y todos los fotogramas se pueden convertir posteriormente a reflectancia.

```python
import chloros_sdk

INTERVAL_S = 1.2      # from the flight plan
FLIGHT_S = 20 * 60    # bounds the loop; it also stops at count= if given

with chloros_sdk.connect_array(serials, target_fps=3.0) as arr:
    groups = arr.capture_repeated(
        "survey/", interval_s=INTERVAL_S, duration_s=FLIGHT_S,
        processing="raw", render_index=False,
        on_capture=lambda i, r: print(f"group {i}: {len(r)} frames"))
```

- **`connect_array(target_fps=…)`**fija la frecuencia desde el inicio;**`arr.set_target_fps(fps)`**la cambia durante el vuelo sin necesidad de volver a conectarse, y**`arr.set_target_fps(None)`** la vuelve a poner en modo automático. Una velocidad superior a la que pueden soportar el host y la red se reduce a ese límite máximo —comprueba `clamped` en el resultado— y la velocidad mínima es de 0,1 fps.
- **No utilices `burst()` para reducir el número de fotogramas.** Escribe todos los fotogramas que produce la matriz; sí que sigue a un `target_fps` fijado, por lo que solo es la herramienta adecuada cuando se desean todos los fotogramas.
- **`target_fps` no corrige los fotogramas dañados.** Reducir este valor disminuye el número de fotogramas producidos, no la velocidad a la que se envía cada uno; para los fotogramas corruptos de GVSP, reduce en su lugar `wire_ceiling_mbps` (véase [Estado de la matriz](#array-health--which-subsystem-is-losing-frames)).

```python
r = arr.set_target_fps(5.0)
if r["clamped"]:
    print(f"asked for {r['requested']} fps, running at {r['target_fps']}")
```

### Conectarse a una matriz ya conectada — `attach_array`

Si la matriz ya está activa (ya sea porque la interfaz gráfica de usuario la ha abierto o porque una sesión anterior del SDK ha llamado a `connect_array`), utiliza `attach_array` para obtener un identificador de la misma en lugar de volver a conectarte. `connect_array` siempre devuelve el error `Camera <sn> is already in array <id>`, ya que el envío de una solicitud POST con `/array/connect` para un miembro del grupo no es idempotente; `attach_array` lee `/api/camera/array/list` y realiza la coincidencia mediante array_id o serials.

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

Patrón: los scripts del SDK que comparten tenencia con la interfaz gráfica de usuario de escritorio deberían intentar primero `attach_array` y recurrir a `connect_array` si aún no hay ninguna matriz en el grupo.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Una matriz conectada se toma prestada, no se posee.** Un identificador de `attach_array` nunca desmonta la matriz: ni la salida del bloque `with` ni una llamada explícita a `disconnect()` la liberan (la llamada explícita imprime una línea en stderr indicando que ha dejado el array en ejecución). La misma regla que para `CameraSession` / `DAQSensorSession`. Por lo tanto, el uso compartido con la interfaz gráfica de usuario es seguro tanto con como sin `with`:
>
> ```python
> with chloros_sdk.attach_array(serials) as arr:
>     arr.capture("output/", processing="reflectance")
> # … the array stays up for the GUI
> ```
>
> Cuando realmente quieras finalizarlo —por ejemplo, para volver a conectarte con un `wire_ceiling_mbps` diferente—, llama a `arr.force_disconnect()`. Volver a conectarse tras un simple `disconnect()` en un identificador adjunto falla con el mensaje `Camera <sn> is already in array <id>`, porque no se ha liberado nada.

### Asistente de análisis de red

Útil antes de abrir la matriz: predice si la configuración propuesta encajará:

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
    # width / height are sensor pixels BEFORE binning (what connect_array
    # takes with binning=); out_width / out_height is what is streamed.
    print(f"Connect with width={r['width']} height={r['height']} "
          f"binning={r['binning']} ({r['out_width']}x{r['out_height']} streamed)")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

```python
analyze_array_network(
    master_serial,
    slave_serials=None,
    width=2048, height=1536,              # sensor pixels before binning
    pixel_format=None,                    # None = what connect applies (Mono12 / BayerRG12)
    binning=1,                            # 1 | 2 | 4
    force_tier=None,
    packed="auto",                        # "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",
    timeout=30.0,
) -> dict
```

Nunca inicia un backend: si uno no responde, se genera el error `ChlorosConnectError`, mientras que una solicitud rechazada se devuelve como resultado. Un `packed` incorrecto genera el error `ValueError`.

`status` es uno de los siguientes: `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (en caso contrario, `error` —incluidas las solicitudes rechazadas: usuario no registrado, `error_code: AUTH_REQUIRED`, o un plan por debajo del mínimo del SDK, `PLAN_UPGRADE_REQUIRED`). `auto_capped_fps` significa que la resolución solicitada solo se ajusta al anillo RX a una frecuencia de activación limitada: mantén la resolución y pasa de `target_fps=result["recommended"]["recommended_target_fps"]` a `connect_array` (véase [Ejemplo 6](#6-capability-probe-before-connecting-a-4-cam-array)). `auto_shrunk` solo aparece cuando el fotograma solicitado no se puede entregar a ninguna frecuencia de fotogramas; «connect» nunca reduce los fotogramas por sí mismo, así que aplica `recommended` tú mismo.

**Cómo interpretar la proyección** (mismo modelo que el panel «Configuración de la matriz» de la interfaz gráfica de usuario):

- **Ráfaga (`frame_bytes_total`) se suma por cámara en el formato de píxeles real de cada una.**Las cámaras**M3M**transmiten en formato Mono12 (2 B/px) o, con un `pixel_format`, el formato Mono de su profundidad (`"BayerRG8"` es `Mono8`, 1 B/px), por lo que, sin él, un fotograma a resolución completa de 4 cámaras ocupa**~25 MB** con tres cámaras mono, y no los ~12,6 MB que daría de sí una suposición de que todo fuera de 8 bits. El backend determina el formato de cada cámara a partir de su modelo.
- **La admisión (`burst_fits_nic_ring`) tiene en cuenta el drenaje**, no se trata de una comparación entre ráfaga completa y anillo: la emisión simulada encaja cuando el host vacía el anillo de recepción más rápido de lo que las cámaras lo llenan. Un host de 10 G + cámaras de 1 GbE**admite** la resolución completa incluso cuando la ráfaga excede el anillo; un host de 1 GbE lo bloquea (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` es un límite máximo conservador de recuperación en serie** — `max(readout+emit, N×emit)` con la emisión por cámara limitada al enlace de cámara de 1 GbE, independientemente de la exposición. Por ejemplo, ~2,8 fps para una matriz de 4 cámaras a resolución máxima de 12 bits (coincide con los ~2,7–3,0 medidos en tiempo de ejecución). Modelo completo: [Referencia de CLI → Modelo de fps y ráfagas del conjunto](cli-reference.md#array-fps--burst-model).
- **Tras una captura continua, la proyección indica la velocidad que mantuvo.** Una captura de 30 s o más a una velocidad automática registra la velocidad en la que se estabilizó con ese tamaño de matriz en este ordenador; se omiten los breves intentos del controlador de velocidad de alcanzar una velocidad mayor. A partir de ahí, la línea de `array-connect`, el análisis de `connect_array()` y el panel «Configuración de la matriz» indican esa velocidad cuando es inferior a la del modelo, junto con lo que la limitó en `cycle_limiter`: `"clock"` (la matriz funcionó a la frecuencia en la que se estabilizó, mientras el bucle de captura esperaba su reloj de activación), `"loop"` (al bucle de captura se le acabó el tiempo), `"capture"` (la captura se retrasó y se saltó grupos) o `"measured"` (no se especifica cuál). La cifra del modelo se encuentra en `wire_fps_max`. La matriz sigue aumentando hasta alcanzar lo que transmite el cable.
- **La sobresuscripción (`oversubscribed: true`) significa que el cable no puede transmitir la matriz a ninguna frecuencia que alcancen las cámaras** — los campos de fps (`achievable_fps_max` / `fps_bright` / `fps_dark`) muestran un valor de 0, y el binning no puede solucionarlo (ya que reduce los bytes por fotograma, no los bytes sincronizados por segundo). Un cable lento por sí solo no provoca esto: en su lugar, la frecuencia de fotogramas disminuye y, si las cámaras no pueden sincronizarse a un ritmo lo suficientemente lento, el nivel pasa a ser `slip-emit-and-capture` (serial). Las soluciones son reducir el número de cámaras, utilizar tramas jumbo o una tarjeta de red más rápida; `max_cams_collision_safe` indica el límite máximo de tu conexión y tus cámaras. La respuesta también incluye `aggregate_demand_bps`, `collision_safe_ceiling_bps` y `per_cam_floor_bps`. Véase [Sobresuscripción](#over-subscription-the-per-cam-floor).

### Detección y listado

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

Cada una de ellas acepta solo palabras clave: `backend_url=` y `timeout=30.0`, y devuelve una lista de diccionarios. `discover_lattice_cameras()` (`serial`, `ip`, `mac`, `model`, `lattice_model`, `vendor`, `connected`, `array_id`, …) también admite `auto_start_backend=True`; las llamadas a `list_*` nunca inician un backend.

---

## Smart-AE / Smart-Capture

Las matrices LATTICE ejecutan AE continuo en segundo plano tan pronto como se conectan, pero una escena recién enfocada tarda un momento en converger. **Smart-capture** es la solución integrada: comprueba la exposición de cada cámara, espera hasta que la matriz se estabilice en toda la ventana y, a continuación, activa la captura. Es equivalente a la interfaz gráfica de usuario: el botón de captura «inteligente» de la aplicación de escritorio llama al mismo punto final del backend.

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

Grupo de backend persistente para sensores espectrales (DAQ-U por USB, DAQ-M por BLE, DAQ-E por Ethernet). Refleja la superficie de la cámara: detección inteligente, reutilización del grupo, conexión idempotente.

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

Prioridad: Ethernet → BLE → USB. Se puede pasar cualquier indicación explícita para fijar el transporte.

### Transporte fijado

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# A specific sensor by id (as list_daq_sensors() shows it; ':' or '-' alike)
daq = chloros_sdk.connect_daq_sensor(sensor_id="CB-7C-A7-5A-73")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

`sensor_id=` nombra un sensor, y solo ese sensor se abre. Se compara con los sensores que ya están abiertos; a continuación, se solicita al nombre de host o a la IP de un DAQ-E (a un DAQ-E que el escaneo de red solo detecta por su IP se le pide, en modo de solo lectura, su nombre, MAC y número de serie, por lo que cualquiera de ellos también lo detecta), la dirección MAC BLE de un DAQ-M y el identificador que un DAQ-U o DAQ-M comunica una vez abierto (cualquier otro que se abra para comprobarlo se vuelve a cerrar). Si no se encuentra, `connect_daq_sensor` genera una excepción `ChlorosConnectError` en la que se detalla lo que se ha encontrado; incluyendo también una dirección (`port=` / `mac=` / `eth_host=`) que contenga un sensor diferente, también se genera un error. Nunca recurre al sensor que esté visible; para eso sirve la llamada con el indicador de cero.

### `connect_daq_sensor()` Firma

```python
connect_daq_sensor(
    *,                                    # every argument is keyword-only
    sensor_id=None,                       # one named sensor (see above)
    transport=None,                       # "usb" | "ble" | "eth"; None = smart-detect
    port=None,                            # DAQ-U serial port ("COM3", "/dev/ttyUSB0")
    mac=None,                             # DAQ-M BLE MAC (implies transport="ble")
    eth_host=None,                        # DAQ-E hostname or IP (implies transport="eth")
    integration_time=None,                # ms; None = the sensor's own
    frame_avg=None,
    enable_ae=None,
    cap_id=None,                          # cap correction: "sunshine_cosine", "fov_30", ..., "none" (bare)
    start_streaming=True,
    backend_url="http://127.0.0.1:5000",
    timeout=60.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> DAQSensorSession
```

Un sensor que ya se encuentra en el conjunto se devuelve tal cual (tomado prestado): no se le aplican los códigos `integration_time`, `frame_avg` ni `enable_ae`. Desconéctalo primero para abrirlo con tu configuración. `cap_id` es la excepción: en un sensor tomado en préstamo se configura con `set_cap()`. Si se omite, se mantiene el valor por defecto del backend (véase la nota debajo de la tabla de métodos).

### Métodos `DAQSensorSession`

| Método | Descripción |
| --- | --- |
| `status(timeout=10.0)` | Resumen de la entrada del grupo (estado de transmisión/grabación, rango de longitudes de onda, sha de calibración, tiempo de integración, frame_avg, estado de AE). Para un DAQ-E cuya transmisión se haya interrumpido: `reconnect` (en los casos en que se activa la reconexión automática del backend: `state`, `attempts`, `next_attempt_in_s`, `last_error`; `None` mientras transmite), `stream_lost_s`, `reconnects` y `last_reconnect`, y `record_gaps` para la grabación en curso; `rx_stream_fault` indica el estado de reconexión hasta que llega el primer espectro. |
| `latest(*, n=1, timeout=10.0)` | Devuelve hasta N tramas de espectro más recientes (cada una de ellas `ts_ns`, `spectrum`, `is_saturated`, `integration_time`, `x`, `y`, `z`). |
| `stream_start()` / `stream_stop()` | Reanudar / pausar la transmisión (el identificador permanece abierto). |
| `record_start(*, output_dir=None, device_name=None, timeout=15.0)` | Iniciar la grabación de un archivo .daq. Devuelve la ruta del archivo. Se rechaza para DAQ-U/M sin un paquete de calibración AWS (DAQ-E queda exento). |
| `record_stop()` | Detener la grabación. Devuelve `{path, rows, warning, gaps}`; `warning` es `None` en caso de una grabación correcta, en caso contrario, el diagnóstico del backend (por ejemplo, un archivo vacío o una interrupción en el flujo), que también se muestra en stderr. `gaps` enumera cada tramo en el que el flujo no ha proporcionado nada: `start_utc`, `end_utc` (`None`: sigue perdido al detenerse), `duration_s`, `reconnected`, `marked_in_file`. Queda vacío en el caso de una grabación continua. |
| `set_cap(cap_id, timeout=15.0)` | Establece el perfil de corrección de recorte que se aplicará a las lecturas a partir de la siguiente: `"sunshine_cosine"`, `"fov_15"` ... `"fov_90"`, o `"none"` (o `None`) para un sensor sin cubierta. Devuelve elresumen del sensor, que se lee de nuevo con `cap_id`. Un DAQ-E-S mantiene `as_recorded` independientemente de lo que se le solicite (su difusor está integrado). Genera `ChlorosConnectError` para un perfil que se carga. CLI: `chloros-cli daq pool-set-cap`. |
| `enable_ptp(timeout=15.0)` | Solo DAQ-E: vuelve a activar el PTP en una unidad cuyo PTP esté desactivado (`status()["rx_ptp_enabled"]` es `False`; la configuración se guarda en la unidad, por lo que permanece desactivado hasta que se vuelve a activar). Devuelve el estado actualizado. Genera el código de error `ChlorosConnectError` para un DAQ-U / DAQ-M, o cuando la unidad no confirma. Chloros nunca vuelve a activar una unidad por sí mismo, ya que el estado desactivado puede ser intencionado. CLI: `chloros-cli daq pool-enable-ptp`. |
| `device_info(timeout=15.0)` | Solo DAQ-E: la configuración propia de la unidad. `firmware` (`running`, `bundled`, `update_available`), `ip` y `ip_mode` (`dhcp` / `linklocal` / `static`: lo que se está ejecutando ahora), `network` (la configuración guardada: `mode`, más `ip` / `mask` / `gateway` en modo estático), `network_change_pending` (válido hasta que la unidad ejecute la configuración guardada, una que incluye `set_network(..., reboot=False)`), `auto_stream`, `ptp`. `firmware["update_available"]` es `None` cuandono se ha podido leer su versión. Mientras el flujo de datos de la unidad no llega a este ordenador, se lee en una nueva conexión y el diccionario añade `stream_silent` (`True`), `stream_fault` y `reconnect`; se genera un aviso si una unidad no responde. CLI: `pool-info`. |
| `set_network(mode="auto", *, ip=None, mask=None, gateway=None, reboot=True, force=False, comeback_timeout=45.0)` | DAQ-E únicamente: DHCP (`"auto"`) o una dirección estática. Reinicia la unidad para aplicarla y la vuelve a conectar; esta sesión sigue funcionando y `address` sigue a la unidad. `reboot=False` solo la guarda. Véase [Configuración del dispositivo DAQ-E](#daq-e-device-settings). CLI: `pool-set-network`. |
| `set_auto_stream(enabled, timeout=15.0)` | Solo DAQ-E: transmisión al encender el dispositivo sin ningún software conectado (guardado en el dispositivo). Devuelve la configuración confirmada. CLI: `pool-set-auto-stream`. |
| `reboot(*, comeback_timeout=45.0)` | Solo DAQ-E: reinicia la unidad y vuelve a conectarla con la configuración que tenía. CLI: `pool-reboot`. |
| `update_firmware(*, comeback_timeout=45.0)` | Solo DAQ-E: actualiza el firmware incluido en esta versión de Chloros en una unidad que ejecute una versión anterior y, a continuación, vuelve a conectarla (tarda aproximadamente un minuto; no la apagues). Devuelve `updated`, `fw_before`, `fw_after`, `bundled` y un `warning` cuando la unidad tardó en responder o volvió a conectarse con otra versión; `updated` es `False` para una unidad que ya está actualizada. Genera el error `ChlorosConnectError` para una unidad cuya versión no se puede leer. CLI: `pool-update-firmware`. |
| `disconnect()` | Liberación del grupo. Nooperación para manejadores adjuntos que no son de su propiedad. En un sensor abierto por esta sesión es `force_disconnect()`, y genera el error `ChlorosConnectError` cuando la liberación no se confirma (la salida del bloque `with` nunca genera un error). |
| `force_disconnect(timeout=30.0)` | Liberar el sensor independientemente de quién lo haya abierto —por ejemplo, un sensor que un script que se ha bloqueado dejó en el pool, al que un nuevo script solo accede como un identificador adjunto cuyo `disconnect()` no hace nada—. Al cerrar el sensor de otro cliente, se finaliza cualquier registro que esté realizando. Devuelve un resultado una vez que el backend confirma la liberación, o cuando no hay nada que liberar (el sensor no está en el grupo o no hay ningún backend en ejecución); una liberación que haya agotado el tiempo de espera o que el backend haya rechazado genera el error `ChlorosConnectError`. CLI: `chloros-cli daq pool-disconnect --sensor-id ID`. |

Atributos: `sensor_id`, `model`, `transport`, `address` (sigue a un DAQ-E que `set_network()` ha redirigido), `info` y `backend_url`.

> **Indica el nombre de la tapa.** Los modelos DAQ-U, DAQ-M y DAQ-E se suministran sin condensador, y Chloros no puede detectar si hay alguno instalado: un sensor conectado sin el `cap_id` obtiene el `sunshine_cosine`, una suposición que, según sus registros, se ha realizado (`cap_id_source=auto_default`). Un DAQ-U sin tapa, con esa corrección, da una lectura aproximadamente 30 veces superior, así que pasa `cap_id="none"` para un sensor sin tapa (o `set_cap("none")`), y el perfil de la tapa que hayas instalado en caso contrario.

### DAQ-E-S — difusor fijo y sensor de inclinación

El **DAQ-E-S** es un DAQ-E con dos diferencias importantes para el software. Todo lo demás —detección, conexión, PTP, grabación y los ajustes del dispositivo que se indican a continuación— funciona exactamente igual que en un DAQ-E.**Su difusor es fijo, por lo que no requiere corrección de la tapa.** El difusor tipo Sunshine está integrado y ya se encontraba en la unidad cuando se realizó su calibración de fábrica, por lo que dicha calibración ya indica la irradiancia descendente real. Chloros establece su límite en `as_recorded` (sin perfil) independientemente de lo que se solicite: se ignora cualquier límite con su nombre (`sunshine_cosine`, `fov_*`, `none`) se ignora con un mensaje, y sus registros almacenan `cap_id_source=model`, por lo que la advertencia de límite supuesto nunca se activa en su caso. Tampoco apliques un límite a sus datos manualmente: `sunshine_cosine` en la parte superior daría una lectura aproximadamente 11 veces superior.

**Lleva incorporado un acelerómetro de 3 ejes.** La respuesta coseno del difusor de luz solar solo es válida mientras está nivelado, por lo que un sensor inclinado indica una irradiancia errónea —y la reflectancia hereda el error— sin que se observe ningún otro síntoma. Por lo tanto, cada lectura del DAQ-E-S incluye su actitud:

| Campo | Significado |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | Ángulos en grados. `None` cuando el firmware marca el ángulo como indefinido (caída libre, un eje inactivo). |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | Aceleración en miligravs. `mag_mg`: un valor alejado de ~1000 indica que la unidad se estaba moviendo y que los ángulos no son significativos para esa muestra. |
| `cal_applied` | Activado: los ejes y los ángulos están en el sistema de referencia del difusor (se aplica la calibración de montaje de la unidad). Desactivado: sistema de referencia sin procesar de la placa, que puede mostrar hasta ~2,3° de inclinación que no es real. |
| `sample_age_ms` | Antigüedad de la muestra del acelerómetro en el momento en que se envió el espectro. |
| `temp_raw` | **Tendencia** de la temperatura del chip sin calibrar: solo diferencias, nunca una lectura del termómetro. |

`DAQSensorSession.latest()` los devuelve bajo la clave `imu` de cada fotograma (`None` en un DAQ-U, DAQ-M o DAQ-E básico). Un `.daq` registrado desde un DAQ-E-S almacena los mismos valores en sus columnas `imu_*` (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, `imu_mag_mg`, `imu_cal_applied`, …) junto a cada espectro, de modo que el lector pueda distinguir una nube de un trípode que se ha movido. En todos los demás modelos, esas columnas están vacías.

El flujo de actitud informa sobre sus propios campos en `status()` (`rx_imu_stream_fault`, `rx_imu_stream_faults`, `rx_last_imu_stream_fault`), separados de los espectrales para que un fallo del acelerómetro nunca se interprete como una interrupción en el registro. `discover_daq_sensors()` y el `model` de la sesión indican `DAQ-E-S`.

### Configuración del dispositivo DAQ-E

```python
with chloros_sdk.connect_daq_sensor(eth_host="daq-e-def330.local") as daq:
    info = daq.device_info()
    print(info["ip"], info["ip_mode"], info["firmware"])

    # A fixed address on the network this computer is on
    daq.set_network("static", ip="192.168.1.50", gateway="192.168.1.1")
    print(daq.address)              # 192.168.1.50: the session follows it

    if info["firmware"]["update_available"]:
        daq.update_firmware()
```

- **Debe existir una dirección estática en la red de este ordenador.** `set_network` genera el error `ChlorosConnectError` si la subred no incluye la dirección de este ordenadoren la red del sensor: tras el reinicio, ningún dispositivo podría acceder al sensor para revertir el cambio. Aplique `force=True` únicamente cuando este ordenador vaya a ser reasignado o redirigido a la nueva subred.
- **`set_network`, `reboot` y `update_firmware` reinician el sensor** y generan el error `ChlorosConnectError` mientras este registra datos, y cuando no se restablece en un plazo de `comeback_timeout` segundos. El ID del sensor y el nombre de host `daq-e-<id>.local` nunca cambian.
- **Una grabación iniciada justo antes del reinicio es cerrada por este y se le asigna un nombre.** Cuando el sensor vuelve a estar operativo, el diccionario devuelto contiene `recording_closed` (la ruta del archivo) y un `warning` que lo indica. Cuando no vuelve, o la transferencia de firmware falla, el `ChlorosConnectError` lo indica en su lugar: su mensaje termina con la advertencia, y se establecen `e.recording_closed` / `e.warning` se activan (ambos `None` cuando no se ha cerrado ninguna grabación). Reinicia la grabación una vez que el sensor haya vuelto a conectarse.
- **Solo se vuelve a conectar el propio sensor.** Tras el reinicio, la unidad debe responder indicando dónde se encontraba —y, tras `set_network("auto")`, también bajo su nombre `.local`— con su nombre, nombre de host y modelo, a partir de la dirección MAC del adaptador de red que tenía; `address` se convierte en la IP que respondió. Una respuesta bajo ese nombre solo cuenta con esa dirección MAC, por lo que desde un ordenador que se conecta al sensor a través de un router, o desde la red NAT predeterminada de WSL2, un sensor que haya sido reubicado por DHCP no se vuelve a conectar (`ChlorosConnectError`; vuelve a conectarlo en su nueva dirección).
- **Para actualizar el firmware, es necesario que el sensor vuelva a conectarse.** `update_firmware()` invita a la unidad, y esta abre una conexión TCP de vuelta a este ordenador para la imagen, en un puerto que elige el sistema operativo. El backend escucha en el adaptador que se comunica con la unidad (en todos los adaptadores solo cuando no puede determinar cuál es) y atiende únicamente a esa unidad: cualquier otra conexión es rechazada (`[DAQ-FW] … refused a firmware connection from <ip>` en la salida del backend) mientras permanece a la espera. Un cortafuegos que bloquee la conexión provoca el fallo de la actualización con el código `device never connected back on TCP <port>`. El instalador de Windows añade la regla correspondiente (véase [Paso 1](#windows-exe)); en otro cortafuegos, permita el tráfico TCP entrante hacia el backend desde la subred del sensor.
- **En WSL2, utilice redes duplicadas** (`networkingMode=mirrored` en `.wslconfig`): en el modo NAT predeterminado, se rechaza cualquier dirección estática (la propia dirección de WSL no está en la red del sensor), el flujo no llega a WSL y la actualización del firmware no se puede completar.
- La interfaz gráfica de usuario (GUI) mantiene su propia conexión con un sensor: un reinicio desde un script finaliza la sesión de la GUI con dicho sensor. La GUI tiene la misma configuración (pestaña «Sensores de luz», configuración del sensor, «Red»).

### Detección: encontrar una dirección a la que conectarse

`discover_daq_sensors()` escanea los puertos USB, BLE y ETH en busca de sensores que *se podrían* abrir. Es el equivalente DAQ de `discover_lattice_cameras()`, y la única forma de obtener la **dirección MAC BLE de un DAQ-M** — un DAQ-E tiene un nombre de host y un DAQ-U un puerto COM, pero la dirección MAC no aparece impresa en el dispositivo ni figura en la lista del sistema operativo.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               DAQ-U  {'vid': '0403', 'pid': '6001', 'manufacturer': 'FTDI'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Campo | Descripción |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | Puerto COM / MAC BLE / nombre de host — pasar a `connect_daq_sensor` como `port=` / `mac=` / `eth_host=`. |
| `display` | Etiqueta legible para el ser humano. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E` \| `DAQ-E-S`, o `None`. Un sensor Ethernet es `DAQ-E-S` o `DAQ-E`, tal y como se identifica cuando se le solicita, en modo de solo lectura, durante el escaneo (`DAQ-E` cuando no responde a tiempo). Un puerto serie USB se lee como `DAQ-U` cuando su chip puente es uno de los que utiliza un DAQ-U (FTDI, CH340, CP210x, Prolific) —una pista que se desprende de la descripción del puerto, no una sonda — y `None` para cualquier otro dispositivo USB, que sigue apareciendo en la lista, por lo que puedes pasarlo como `port=`. Un puerto serie sin identificador de fabricante USB y sin dicho chip puente (un UART de la placa base, como el `/dev/ttyS*` de Linux, un enlace serie Bluetooth o Intel AMT) no puede ser un DAQ-U y no aparece en la lista. |
| `extra` | Detalles por transporte (nombre anunciado por BLE; USB `vid`, `pid`, `manufacturer`, `serial_number`; DAQ-E ip/fw/…). Se omiten los valores vacíos. |

| Parámetro | Valor por defecto | Descripción |
| --- | --- | --- |
| `transports` | los tres | Secuencia (o cadena CSV) que limita el escaneo. Vale la pena indicarla cuando se sabe lo que se quiere — el BLE es el eslabón más lento. |
| `scan_timeout` | 5 | Ventana de exploración por transporte en segundos; el backend la limita a un rango de 1 a 20. |
| `timeout` | 60,0 | Límite de HTTP para toda la llamada (al igual que en el resto del SDK). |
| `auto_start_backend` | `True` | Inicia un backend local si no hay ninguno en ejecución. Nunca se inicia para un `backend_url` remoto. |

> **Los sensores que ya están abiertos en el grupo no aparecen.** Un periférico BLE conectado deja de anunciarse y no se puede sondear un puerto COM abierto, por lo que la búsqueda muestra lo que está *disponible para conectarse*. Es normal obtener un resultado vacío justo después de conectar algo; utiliza `list_daq_sensors()` para lo que ya tienes. Los transportes cuyo escaneo no se puede ejecutar (sin bleak ni zeroconf instalados) se omiten en lugar de generar un error, por lo que un equipo sin Bluetooth sigue obteniendo sus respuestas USB y ETH.

### Listado

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Qué DAQ alimenta a qué cámara — `daq_bindings()`

La reflectancia de una captura, y el `.daq` que guarda, utilizan el DAQ vinculado a esa cámara o matriz; de lo contrario, se utiliza el DAQ más reciente que tenga el backend. La selección del sensor de luz en la interfaz gráfica de usuario y el `chloros-cli daq pool-bind` lo configuran; al igual que:

```python
chloros_sdk.set_daq_bindings(cameras={"214701281": "COM7"},          # a sensor id or an address
                             arrays={"array-1779862544497": "daq-e-def330"})
chloros_sdk.set_daq_bindings(cameras={"214701281": None})            # None clears it
for serial, b in chloros_sdk.daq_bindings()["cameras"].items():
    print(serial, b["binding"], b["source"], b["resolved"]["in_pool"])
```

`daq_bindings()` devuelve `{"cameras": {serial: b}, "arrays": {array_id: b}}`: cada `b` contiene `binding` (el valor establecido), `source` (`project`, del `sensors.json` del proyecto, o `runtime`), `known` / `known_as` (si la clave de la cámara o de la matriz hace referencia a algo que Chloros conoce, y cómo: `connected`, `project` — el `cameras.json` del proyecto abierto — o, en el caso de una cámara, `seen`: ya se ha abierto antes en este ordenador; `known: False` para una clave que no designa nada) y `resolved` (el sensor del grupo al que está asignado ahora: `in_pool`, `sensor_id`, `address`, `model`, `streaming`). Un enlace en tiempo de ejecución prevalece sobre el del proyectodurante la sesión; el archivo de proyecto no se modifica. Al desconectar una cámara, una matriz o un DAQ, se eliminan todos los enlaces de ese hardware, independientemente de quién los haya establecido (los miembros de una matriz se enlazan con la propia matriz).

`set_daq_bindings()` solo vincula una cámara que esté conectada, que se encuentre en el proyecto abierto o que se haya abierto anteriormente en este ordenador, y una matriz que esté conectada o que se encuentre en el proyecto abierto (o una tecla ya vinculada). Cualquier otra tecla genera el error `ChlorosConnectError` con el nombre (`400`, `code`, `UNKNOWN_BINDING_TARGET`, con las teclas `unknown`), y **nada** de esa llamada queda vinculado. `None` borra cualquier clave.

### Reflectancia ambiental + del objeto — `light_sensor_reflectance()`

Dos DAQ agrupados, uno mirando hacia el cielo (ambiente) y otro hacia el objetivo (objeto), proporcionan la reflectancia del objetivo y los índices de vegetación a partir de sus últimas lecturas: el par «Ambiente + Objeto» de la interfaz gráfica de usuario y `chloros-cli daq reflectance` sin necesidad de abrir los sensores por segunda vez. Ambos deben estar conectados y transmitiendo datos.

```python
r = chloros_sdk.light_sensor_reflectance("daq-e-def330", "daq-e-54b774")
print(r["reflectance_type"])                       # "Apparent Reflectance" when both are calibrated
for name, band in r["bands"].items():
    print(name, band["rho"] if band["valid"] else band["reason"])
for idx in r["indices"]:
    print(idx["name"], idx["value"])
```

```python
light_sensor_reflectance(
    ambient_sensor_id, object_sensor_id, *,
    bands=None,          # [{"name", "center_nm", "fwhm_nm", "role"}]; role blue / green / red / nir
    preset=None,         # a preset id from daq_band_presets(); default: the suggested one, else "GENERIC"
    expressions=None,    # custom indices: [{"name": "MyNDVI", "expr": "(NIR_850 - Red_660) / (NIR_850 + Red_660)"}]
    builtins=True,       # include the built-in indices
    backend_url="http://127.0.0.1:5000", timeout=30.0,
) -> dict
```

- **Bandas.** La reflectancia de cada banda es la irradiancia de la banda del objetosobre la del ambiente. `daq_band_presets(filter_code=None)` enumera los conjuntos de bandas: uno por cada filtro LATTICE M3C (`RGN`, `OCN`, `NGB`, el modelo de bandas propio de la cámara, por lo que un NDVI de DAQ y un NDVI de la cámara son la misma magnitud) y `GENERIC` de banda ancha. Su `suggested` es el preajuste que coincide con `filter_code`; de lo contrario, es el filtro de la primera cámara conectada (`suggested_from` indica cuál).
- **Devuelve** `bands` (por banda: `rho`, `valid`, `reason`, `e_ambient`, `e_object`), `indices` (los integrados), `custom` (cada uno con `value`, o `valid: False` y el `error`), `invalid_bands`, `reflectance_type`, `preset` y las dos lecturas de `timestamps` (`ts_ns`). Un intervalo que el sensor ambiental no haya podido medir (fuera de su rango o en su umbral de ruido) se considera inválido, indicando el motivo, y se omite de todos los índices que lo requieran, sin que se calcule nunca con valor cero.
- **Aparente o relativo.** Los índices dependientes de la escala (SAVI, OSAVI, EVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) necesitan una relación absoluta, por lo que solo se obtienen cuando ambos sensores están calibrados en potencia (cada uno tiene un `calibration_bundle_sha`): `"Apparent Reflectance"`. De lo contrario, el resultado es `"Relative Reflectance"`, que incluye únicamente los índices invariantes a la escala.
- **Se comparan dos rejillas de longitudes de onda diferentes** en función de su solapamiento: los puntos del sensor ambiental dentro del alcance del objeto y el espectro del objeto interpolado sobre ellos.
- **Dos sensores, nunca uno solo.** Asignar el mismo nombre a un sensor (el mismo identificador, en cualquier forma ortográfica que acepte el pool) genera el error `ChlorosConnectError`: la lectura de un sensor sobre sí mismo es 1,0 en todas las bandas, lo que no mide nada. El backend también lo rechaza (`POST /api/daq/indices` responde a `400`, `code` a `SAME_SENSOR`, cuando los `ambient_sensor_id` y `object_sensor_id` que envía el SDK corresponden a un único sensor agrupado, p. ej., su ID y su puerto).

### Uso compartido con la CLI y otros scripts

El grupo de sensores DAQ del backend es compartido por todos los clientes de la CLI y el SDK de la máquina. Si un sensor ya se encuentra en el grupo —abierto por `chloros-cli daq pool-connect` o por otro script—, al llamar a `connect_daq_sensor(port="COM3")` desde Python se devuelve un identificador prestado: su `disconnect()` no realiza ninguna operación, por lo que tu script no le quita el sensor al cliente que lo abrió al salir del ámbito. Lo mismo ocurre con `sensor_id=` (escrito con `:` o `-`, cualquiera de las dos formas es válida), para un DAQ-E identificado por su IP cuando se abrió mediante el nombre de host (o al revés), y para la detección inteligente con indicador cero cuando el backend responde con un sensor que ya tenía: un bloque `with` solo libera un sensor que haya abierto él mismo.

La interfaz gráfica de usuario de Chloros **no** forma parte de este grupo: su panel DAQ establece su propia conexión con el sensor, por lo que `connect_daq_sensor` nunca toma prestado un sensor que la interfaz gráfica tenga abierto. No utilices el mismo sensor desde la interfaz gráfica y desde un script al mismo tiempo. Un DAQ-U o DAQ-M que la interfaz gráfica tenga abierto normalmente no se abre, pero un DAQ-E admite una segunda sesión: la conexión del script modifica los ajustes de adquisición de la unidad en la interfaz gráfica, y su `disconnect()` detiene el flujo de datos de la unidad, lo que pone fin a la visualización en directo de la interfaz gráfica y a cualquier grabación que esté realizando. Cierra primero el sensor en la interfaz gráfica.

### PTP Grandmaster — `time_sync_status()`

El backend ejecuta el PTP Grandmaster del que las cámaras LATTICE y las unidades DAQ-E obtienen sus marcas de tiempo. Se inicia bajo demanda: `connect_camera()`, `connect_array()`, `connect_daq_sensor()` y `connect_all()` lo inician si no está en ejecución —incluida una conexión que luego falla, como cuando `connect_all()` intenta conectarse a un DAQ-E que no existe—, y lo mismo ocurre con `test_array_sync()` y todas las llamadas que abren cámaras para un comando (`capture_cameras_transient()`, `start_multi_camera_stream()`, las llamadas de alineación, `capture_live_index()`).

- **Dónde se anuncia.** En los adaptadores de red por cable conectados a las cámaras (incluida una dirección local de enlace `169.254.x.x`); en caso contrario, en el adaptador de la ruta predeterminada si este es por cable. Nunca en Wi-Fi ni en un adaptador virtual (Tailscale, WSL): no hay ninguna cámara ni DAQ-E detrás de ellos. Si no hay ningún adaptador de este tipo activo, se ejecuta anunciando en ningún sitio (`health` `NO_SOCKETS`) y se vincula al adaptador que se active la próxima vez que se conecte una cámara o un DAQ.
- **Tras una pérdida de conexión.** Cuando el adaptador de la cámara pierde la conexión y la recupera (por ejemplo, al desconectar un cable, sustituir un conmutador o reenumerar un adaptador USB), el «grandmaster» reconstruye sus sockets por sí mismo, sin necesidad de reiniciar, y vuelve a comprobar todos los DAQ-E conectados, reactivando aquel que haya quedado atascado. También se reconstruye cuando todas las cámaras y DAQ-E a los que prestaba servicio permanecen inactivos durante 15 s.
- **¿Llega la señal?** Lee `time_sync_status()` (`GET /api/time-sync/status`). `state` solo indica que el hilo del «grandmaster» se está ejecutando; `health` indica si sus paquetes se envían: `OK` (no se detecta ningún fallo), `STOPPED`, `NO_SOCKETS`, `SENDS_FAILING`, `DEGRADED` (algunos adaptadores fallan) o `PEERS_SILENT` (las cámaras y los DAQ-E a los que prestaba servicio dejaron de responder), con una frase en `health_detail`. `announce_sent` / `sync_sent` / `delay_resps_sent` cuentan solo los mensajes que se enviaron; `announce_send_errors`, `sync_send_errors`, `follow_up_send_errors`, `delay_resp_send_errors`, `last_send_error`, `failing_interfaces` y `last_delay_req_age_s` describen los que no lo hicieron, y `rebinds` / `last_rebind` las reconstrucciones. `sync_sends_slow` cuenta las sincronizaciones cuyo envío superó los 200 µs y `sync_send_max_us` es la más larga; el tiempo de origen de cada «Follow_Up» cuenta como máximo 200 µs de su envío. El «grandmaster» se ejecuta en un proceso propio (`own_process`, `process_pid`), con alta prioridad en Windows (`process_priority`), por lo que un backend ocupado no puede retrasar las marcas de tiempo ni sincronizar los relojes de todas las cámaras a la vez; si ese proceso no se puede iniciar, se ejecuta dentro del backend como antes (`own_process` false).

```python
st = chloros_sdk.time_sync_status()
if st.get("health") not in (None, "OK", "STOPPED"):
    print("PTP:", st["health"], "-", st.get("health_detail"))
    chloros_sdk.restart_time_sync()
```

| Función | Qué hace |
| --- | --- |
| `time_sync_status()` | El estado del «grandmaster», su salud, la identidad del reloj, las prioridades y los contadores de mensajes (arriba). `chloros-cli time-sync status`. |
| `time_sync_peers()` | Los esclavos PTP que detecta (cámaras, DAQ-E): `peers`, cada uno de los `port_identity`, `last_seen_ip`, `last_delay_req_age_ms`, `delay_req_count`. `time-sync peers`. |
| `time_sync_cameras()` | Cada cámara matricial conectada: `PtpStatus`, `PtpOffsetFromMaster` y `PtpMeanPathDelay` (una cámara cuyo firmware no tenga nodos PTP solo tiene el número de serie). `time-sync cameras`. |
| `restart_time_sync()` | Detiene y reinicia el «grandmaster»; a continuación, se vuelve a comprobar el PTP de cada DAQ-E agrupado (`daq_ptp_rechecked`). Genera `ChlorosConnectError` cuando no se restablece. `time-sync restart`. |
| `set_time_sync_priority(priority1=None, priority2=None)` | Sus prioridades PTP (0-255; el valor más bajo gana la elección de mejor maestro), para una red con otro maestro PTP. Requiere que esté en ejecución. `time-sync set-priority`. |

Cada uno toma `backend_url=` y `timeout=` e inicia un backend local que no se está ejecutando.

---

## Utilidades de la cámara

Lo que hace `chloros-cli lattice`, además de conectarse y capturar, son funciones básicas en `chloros_sdk` (módulo `chloros_sdk.camera_utils`). Cada una llama a la ruta del backend que utiliza el comando de la CLI, con las reglas de la CLI, por lo que ambas se comportan de forma similar. Aceptan un **número de serie** de cámara, no una sesión, y las mismas palabras clave `backend_url=` / `timeout=` que las llamadas de conexión inteligente; si el backend no está en ejecución, se inicia, tal y como hace `connect_camera()`. En caso de rechazo, se genera `ChlorosConnectError` con el motivo indicado por el backend.

```python
import chloros_sdk as c

c.set_camera_color_profile("214701281", profile="natural")
c.set_camera_highlights("214701281", -40)
print(c.camera_firmware_status()["cameras"])
st = c.network_adapters(estimate=True, cameras=4, pixel_format="BayerRG12")
```

| Función | Qué hace |
| --- | --- |
| `camera_color_profile(serial)` / `set_camera_color_profile(serial, profile=, custom_cct_k=)` | El perfil de color de la pantalla: `raw`, `linear`, `natural`, `enhanced` o `custom_temp`, y su temperatura de color (limitada a 2000-10 000 K; se lee cuando el perfil es `custom_temp`). |
| `camera_color_adjust(serial)` / `set_camera_color_adjust(serial, saturation=, contrast=, reset=False)` | Saturación y contraste de la pantalla, de 0 a 200 % cada uno (100 = sin cambios); `reset=True` restablece ambos valores a 100. |
| `camera_highlights(serial)` / `set_camera_highlights(serial, value)` | Reduce el brillo de los píxeles más brillantes y más blancos de la imagen de la pantalla: -100..0 (truncado hacia cero y luego limitado; 0 = desactivado). |
| `camera_white_balance_once(serial, settle_s=None)` | Balance de blancos de una sola toma: la cámara transmite brevemente, converge y devuelve `red_ratio` / `blue_ratio` (`ratios_read` Falso cuando no puede comunicarlos). Mantiene el balance de blancos que ha calculado; el resto de ajustes que haya escrito su interfaz de programación se devuelven tal y como se encontraron. |
| `set_camera_white_balance(serial, mode=None, red=None, blue=None)` | `mode="auto"` (continuo) o `"off"`; o relaciones de balance fijas `red` / `blue`, que desactivan primero el modo automático. Una relación que la cámara rechaza se denomina `errors`. |
| `camera_presets()` / `apply_camera_preset(serial, preset)` | Los preajustes de la cámara; al aplicar uno de ellos, la cámara mantiene los ajustes que indican los nombres de los preajustes (la respuesta es `settings`, tal y como los lee la cámara a continuación; por ejemplo, la frecuencia de fotogramas a la que funciona; `None` para un campo que el preajuste no nombra), y todos los demás ajustes que su archivo de configuración contenía vuelven a su estado original. `"default"` restablece la cámara a sus valores predeterminados de fábrica (su configuración de fábrica con exposición automática en modo libre, el límite de rendimiento del enlace y la corrección de defectos desactivados, y el balance de blancos calibrado de una cámara RGB conservado), sin guardarlos en su memoria: la respuesta contiene `factory_defaults: True` y todos los campos de `settings`. En `camera_presets()`, `default` es `{factory_defaults: True, description}`. |
| `camera_filter_models()` / `camera_filter_model_info(name)` | Los modelos de filtro y los detalles de un modelo (bandas, número de condición de respuesta espectral). El modelo de filtro de una cámara se lee a partir de su cadena de modelo; no se configura manualmente. |
| `camera_firmware_status(serial=None)` | El firmware de cada cámara: `current_version`, `available_version`, `update_offered`, `needs_minimum_update` y `minimum_firmware`. |
| `update_camera_firmware(serial, force=False)` | Instala el firmware de esta versión de Chloros en una cámara: restablecimiento de fábrica, instalación, reinicio; se reescriben los ajustes básicos de Chloros; se mantiene la calibración de la cámara. Varios minutos; la cámara debe permanecer encendida. `force=True` vuelve a flashear una cámara actual. |
| `camera_power_probe(serial)` | Los nodos de alimentación y térmicos de la cámara y su temperatura. |
| `sleep_camera(serial)` / `wake_camera(serial)` | El modo de inactividad de bajo consumo de la cámara (permanece en reposo hasta que se activa: su límite de rendimiento del enlace activado (`DeviceLinkThroughputLimitMode` `On`) en su valor mínimo, el modo de ahorro de energía activado si la cámara dispone de él), con la temperatura antes y después; al activarse, el enlace vuelve a quedar tal y como estaba al entrar en modo de reposo (el límite, su modo y el retraso de paquetes `GevSCPD`, que la cámara mantiene en 0 mientras el límite está activado; el registro se guarda en este ordenador, por lo que un backend posterior que la active también lo encontrará), o, en el caso de una cámara que este ordenador no haya puesto en modo de suspensión pero que parezca estar en él (con el límite activado en el valor mínimo), con el límite desactivado y un `GevSCPD` dejado en 0 a 80 (los ajustes de fábrica); un límite establecido por encima del mínimo se deja tal cual. Solo se conserva el estado de alimentación; el resto de ajustes que se hayan modificado se restablecen a su valor original. |
| `calibrate_camera_with_target(serial, filter=None, attempts=30, settings=None)` | Captura fotogramas hasta que un objetivo de calibración MAPIR esté a la vista, y ajusta los coeficientes por canal a partir de sus paneles: `target_id`, `panel_dns`, `coefficients`. |
| `network_adapters(estimate=False, cameras=1, pixel_format="BayerRG8")` | Los adaptadores Ethernet de este ordenador para GigE Vision (MTU, velocidad de enlace, tramas jumbo) y cualquier `issues`; `estimate=True` añade la velocidad de fotogramas y el ancho de banda que obtendrían las cámaras `cameras`. |
| `fix_network_settings()` | Aplica la configuración del adaptador GigE Vision al. Modifica la configuración del adaptador y solo funciona cuando Chloros se ejecuta como administrador; de lo contrario, muestra un mensaje de error con el registro de lo que ha detectado. |
| `apply_vignette_correction(input_dir, output_dir, lens_model=None)` | Corrige el viñeteado de las imágenes de una carpeta, cada una con el mapa de campo plano de su propio objetivo: `processed`, `lens_models` (el objetivo para el que se ha corregido cada archivo), `skipped` (con los motivos). Sin `lens_model`, el objetivo de cada archivo se toma de su modelo EXIF; un `lens_model` se aplica a los archivos cuyo «Modelo» no especifica ninguna cámara, y se omite cualquier archivo cuyo «Modelo» lo contradiga. Solo archivos. |
| `read_dls(mac, filter=None, camera_serial=None)` | Una lectura de un DLS (sensor de luz descendente por Bluetooth): espectro, irradiancia por banda y magnitudes de banda a través de la propia respuesta de la cámara cuando se proporciona `camera_serial`. `discover_daq_sensors(transports=["ble"])` determina la dirección MAC. |
| `probe_system(pixel_format="BayerRG12", discover=True)` / `system_info()` | El hardware del host para cámaras y la configuración que recomienda (`report_text`), así como la CPU, la GPU y el entorno de ejecución de PyTorch del sistema. |

- **Algunos necesitan la cámara para uso exclusivo.** `camera_white_balance_once()`, `apply_camera_preset()`, `sleep_camera()` / `wake_camera()`, `calibrate_camera_with_target()` y `update_camera_firmware()` abren la cámara. Mientras Chloros la mantiene ocupada (conectada en la aplicación, con `connect_camera()`, como miembro de una matriz, o en uso por otro comando en ese momento, como una prueba de matriz o una captura que abre sus cámaras), el backend devuelve un error HTTP 409 `CAMERA_HELD` (tanto en `code` como en `error_code`), indicando cómo liberarlo. Los ajustes de color, las luces altas, `camera_power_probe()` y las lecturas funcionan en una cámara conectada.
- **El color, las luces altas y el balance de blancos son para cámaras RGB.** En una cámara monocroma de banda única (M3M), los ajustes no modifican nada y devuelven `{"skipped": True, "reason": ...}`, al igual que lo hace la CLI. Se trata de ajustes de visualización: la vista en directo y las capturas a nivel de pantalla, nunca la radiancia ni la reflectancia.

---

## Matrices transitorias, perfiles de alineación e índices

Lo que hace `chloros-cli lattice` sin una conexión persistente, al igual que las funciones de `chloros_sdk` (módulo `chloros_sdk.array_utils`): `continuous`, `multi-capture`, `multi-stream`, `multi-test`, `multi-detect`, `analyze-array`, `align-calibrate`, `align-apply`, `align-stream`, `align-info`, `align-reorder` y `index`. Cada uno envía la solicitud que envía el comando de la CLI, con los valores predeterminados de la CLI.

```python
import chloros_sdk as c

prof = c.calibrate_alignment_profile(["214000535", "214000551", "214000556"],
                                     "D:/rig/alignment_profile.json", method="orb", model="rigid")
print({sn: t["reprojection_error"] for sn, t in prof["transforms"].items()})

c.capture_aligned_multiband("D:/rig/alignment_profile.json", "D:/rig/out")
idx = c.capture_live_index("D:/rig/alignment_profile.json", preset="NDVI",
                           output_dir="D:/rig/ndvi", capture_level="reflectance")

with c.start_multi_camera_stream(["214000535", "214000551"], "D:/rig/stream", fps=5, count=50) as job:
    print(job.wait()["final_stats"])          # waits for the 50 groups, then closes the cameras
```

- **Abren las cámaras por sí mismos.** Cada comando abre sus cámaras, realiza su tarea y las cierra, por lo que necesita cámaras que Chloros no tenga en su poder (conectadas en la aplicación, con `connect_camera()`, o como miembro de una matriz); el backend devuelve un error HTTP 409 `CAMERA_HELD` y indica cómo liberar una. Una apertura transitoria tarda un rato (unos 40 s para unas cuantas cámaras). Para capturas repetidas, `connect_array()` una sola vez es la vía más rápida, y funciona con todos los tipos de exportación, mientras que una captura transitoria guarda una imagen sin debayering. `calibrate_alignment_profile()` es la excepción: en una matriz conectada de esas cámaras, lo resuelve a partir de los propios fotogramas de la matriz (`used_live_array`).
- **`serials`** son las cámaras que hay que abrir, siendo la primera la que activa el disparador; `None` abre todas las cámaras de la red.
- **`settings`** es un diccionario de ajustes de cámara que anulan los valores predeterminados; los campos que establecen los indicadores de la CLI: `exposure_time` (µs), `gain` (dB), `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `frame_rate`, `gev_packet_size` y `preset` (un preajuste de cámara con ese nombre). Al establecer un valor de exposición o ganancia, se desactiva ese control automático, al igual que ocurre con los indicadores.
- **`gpu`**: `True` fuerza el warp de la GPU, `False` el de la CPU (`--gpu` / `--no-gpu`); `None` deja que el backend elija.
- **`keep_settings`**/**`clear_settings`** (todas las funciones que abren cámaras; por defecto, `False`): al final, cada cámara se restablece tal y como se encontró; `keep_settings=True` guarda la configuración de la sesión en la propia memoria de la cámara (funcionamiento libre), `clear_settings=True` restablece esa memoria a los ajustes de fábrica — véase [Memoria de la cámara](#camera-memory--keep_settings--clear_settings). La respuesta (para un trabajo, la de `stop()`) incluye `camera_memory`.
- **Las rutas son las del backend**: las relativas se convierten en absolutas con respecto a tu directorio de trabajo cuando el backend se encuentra en esta máquina. Las funciones del archivo de perfil de alineación (`alignment_profile_info()`, `reorder_alignment_profile()`) leen el archivo en este ordenador, al igual que lo hace la CLI.

| Función | Qué hace |
| --- | --- |
| `start_camera_continuous(serial, output_dir="output", *, format="tiff", jpeg_quality=95, queue_depth=64, settings=None)` | Graba con una cámara de forma continua hasta que se detenga. Devuelve un `CliJob`. Los fotogramas de una ejecución de `tiff` son el archivo `raw`, una matriz que captura los datos de esa cámara (un plano de mosaico con los, sin comprimir); `png` y `jpg` contienen 8 bits, por lo que esas series son fotogramas desmosaicados para su visualización. Un DAQ vinculado o agrupado escribe un `.daq` por fotograma en lacarpeta `daq/` de la serie (`daq_files` en sus estadísticas). Cada fotograma se escribe una vez con todos sus metadatos (modelo, hora de captura, objetivo, bandas, calibración en un fotograma sin procesar); `stop()` devuelve un resultado una vez que se han escrito todos los fotogramas capturados (hasta 25 s), y solo entonces se cierra la cámara. Una cámara que se deja preparada para un disparador de hardware (`TriggerMode` `On`) se configura en modo libre para la sesión (la respuesta de inicio `trigger` indica lo que se ha encontrado). Al detenerse, la cámara se restablece tal y como se encontró —con su configuración de disparo y todos los ajustes modificados durante la sesión—, a menos que se utilice `keep_settings=True`, que guarda los ajustes de la sesión en la propia memoria de la cámara (funcionamiento libre), o `clear_settings=True`, que restablece dicha memoria. Una sesión que no graba ningún fotograma finaliza automáticamente tras 10 s con `stop_reason` `no_frames` y el motivo en `last_error`, y su `stop()` devuelve ese motivo como `error`. Se contabilizan los fotogramas perdidos porque los escritores no pudieron seguir el ritmo, y `drop_reason` indica el motivo. Para una cámara que Chloros mantiene, utiliza `CameraSession.capture_repeated()`. `lattice continuous`. |
| `start_multi_camera_stream(serials=None, output_dir="output", *, fps=3.0, count=None, format="tiff", jpeg_quality=95, settings=None)` | Transmite grupos de fotogramas sincronizados en `fps`, para grupos `count` o hasta que se detenga. Devuelve un `CliJob` (su `start_reply` tiene `n_cams` y el cable `pacing`); sus estadísticas finales contabilizan los grupos capturados, escritos y descartados, los archivos que el escritor no pudo escribir y los grupos para los que una cámara no entregó nada. `lattice multi-stream`. |
| `capture_cameras_transient(serials=None, output_dir="output", *, format="tiff", jpeg_quality=95, settings=None)` | Una captura sincronizada, una imagen sin bayering por cámara: `results` (`filepath`, `serial`, `frame_id`) y `pacing`. Aparece un grupo al que le falta una cámara, indicando primero cuántos archivos se han guardado. `lattice multi-capture` sin matriz conectada. |
| `test_array_sync(serials=None, count=10, ptp_settle_s=None, timeout=None)` | `count` capturas sincronizadas con una exposición fija: `sync_groups`, el `master`, una fila `group_timestamps` por disparo (una cámara que se saltó uno está ausente de su fila) y `sync`: el desfase de cada dos cámaras respecto a la principal, disparo a disparo (`offsets_us`: por cámara `mean_us`, `min_us`, `max_us`, `jitter_us`, `groups`), en UN ciclo de reloj (`clock_domain`: `ptp`, o `device` — PTP desactivado, el reloj de tiempo de actividad propio de cada cámara, por lo que su desfase se produce al arrancar y solo la fluctuación mide la sincronización). Se omite y se cuenta un disparador cuyas marcas combinen el reloj del dispositivo y el PTP (PTP desactivado o aún en proceso de sincronización) (`groups_mixed_domain`), al igual que aquel en el que un reloj PTP sigue estando a más de `agree_within_us` del maestro (`groups_clocks_disagree`); `notes` así lo indican. `clock_offset_mean_us` / `clock_offset_jitter_us` son la media y la fluctuación máximas por cámara. `success` es falso, al igual que `error`, cuando se ha perdido un disparador, cuando hay menos de dos disparadores en los que todas las cámaras estén sincronizadas con un mismo reloj, o cuando los relojes nunca coinciden (`sync.problems`); las filas se devuelven en cualquier caso. Inicia el «grandmaster» PTP si no está en funcionamiento (`time_sync` indica `grandmaster_running` y `grandmaster_started_by_this_test`), activa el PTP de todas las cámaras (tal y como hace `connect_array()`) y, antes del primer disparador, espera a que el reloj PTP de todas las cámaras se sincronice con el «grandmaster» y se estabilice: `PtpStatus`: «Slave» en un «grandmaster», con una desviación de menos de 100 µs respecto a este y estable durante 3 s, y cada cámara sincronizada de esa forma, sin interrupción, durante 120 s —unos 2 minutos desde el encendido—. Un reloj que acaba de sincronizarse presenta un desfase de hasta ~200 us durante unos 2 minutos después de que se le identifique como «Slave», y el propio `PtpOffsetFromMaster` de la cámara no lo muestra; 120 s hacen que las cámaras se sitúen a unos ~10 us de un reloj estabilizado. Una cámara que siga sincronizada desde una prueba anterior en la que se dejó PTP activado, tal y como lo ve este backend, cuenta a partir de esa sincronización, por lo que una prueba realizada inmediatamente después de otra no vuelve a esperar; una cámara que se enciende durante la prueba, o una que se encuentra con PTP activado y que este backend no ha visto sincronizada (encendida en otro lugar o antes de un reinicio del backend), espera sus 120 s, y en una matriz que incluye ambas, las cámaras que acaban de sincronizarse siguen esperando su turno. Una cámara que sale del modo esclavo, supera los 100 µs o sigue a otro gran maestro durante tres lecturas consecutivas (1,5 s) durante la espera, reinicia su cuenta; se registran una o dos de esas lecturas (`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`) y la cuenta continúa, y un ID de «grandmaster» que no se pueda leer se considera desconocido, no un cambio. Que el recuento de todas las cámaras se reinicie a la vez (en menos de un segundo) se debe a que es el tiempo del «grandmaster» el que avanza, no el de las cámaras: el registro del backendregistro indica `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …`, `ptp_lock` lo recoge en `grandmaster_steps`, y una espera que se agota a continuación inicia su error con `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`. El límite no se amplía para ello. `ptp_settle_s` (0-600; `0` si no hay espera de estabilización) establece el tiempo de estabilización para la ejecución; se rechaza cualquier otro valor (código HTTP 400). La espera está limitada al tiempo de estabilización más 90 s (210 s por defecto, con un mínimo de 90 s), y la llamada espera al backend `timeout` s: por defecto, 240 más el tiempo de estabilización (360 s), al igual que lo hace `test_array_sync` de `chloros-mcp`. `ptp_lock` tiene `converged`, `waited_s`, `timeout_s`, `settle_s`, `locked_s` (el tiempo que cada cámara ha permanecido bloqueada y estable), `found_settled` (las cámaras que ya estaban bloqueadas durante el tiempo de estabilización cuando se detectaron) y, por cámara, `status`, `offsets_ns`, `slave_s` (cuánto tiempo lleva sincronizada; «Ninguna»: no está sincronizada) y `remaining_s` («Ninguna» cuando el PTP de ninguna cámara se activa). El registro del backend contiene una línea cada 5 s con el estado de cada cámara, cuánto tiempo lleva sincronizada y cuánto tiempo le queda. Todas las cámaras se encienden con el PTP desactivado; a partir de ahí, los relojes se sincronizan en unos 10 s. Posteriormente, cada cámara se restablece tal y como la encontró la prueba: su PTP, la exposición automática y la ganancia, que la prueba mantiene desactivadas durante su ejecución (`ExposureAuto` / `GainAuto`, y una prueba manual `ExposureTime` / `Gain`), y el resto, salvo en el caso de `keep_settings=True` —es decir, una cámara que queda con el PTP activado (`PtpEnable=True`, o una ejecución con `keep_settings=True`) permanece sincronizada, y una vez que este backend la ha detectado sincronizada durante el tiempo de estabilización, una prueba posterior no espera a que lo esté. Si los relojes no se estabilizan, `success` es falso antes de cualquier activación, y `error` indica qué hacer: una cámara que nunca se ha sincronizado (`not_slave`) significa que el «grandmaster» no la está alcanzando (`time_sync_status()`); a una que se ha ido desconectando continuamente se le asigna un nombre que indica cuánto tiempo ha estado sincronizada (`remaining_s`); relojes que aún están convergiendo (`far`): comprueba `time_sync_status()`, y para que estén sincronizados antes de la prueba, activa el PTP y déjalo activado (`PtpEnable=True` en cada cámara, o `keep_settings=True`; la prueba restablece los valores que ha encontrado, por lo que permanece activado), y, a continuación, ejecútala. `lattice multi-test`. |
| `detect_array_master(line=None)` | Las cámaras de la línea de sincronización y un maestro predeterminado: `master_serial`, `slave_serials`, las `line` detectadas. `lattice multi-detect`. |
| `analyze_array_models(camera_models=None, *, width=2048, height=1536, n_active=None, force_tier=None, binning=1, packed="auto", pixel_format=None)` | Menuda variedad de modelos de cámaras (nombres de producto, `"LATT-M3C-L87-FOCN"`) que se conectarían a la red de este ordenador: `tier`, `achievable_fps_max`, el presupuesto de cableado (`max_cams_collision_safe`, `oversubscribed`), `cam_caps` y `fix_suggestions`. Por defecto, los modelos de las cámaras detectadas; no hay ninguna cámara abierta. Para ver las cámaras de las que dispones, con la configuración que utilizaría una conexión, consulta [`analyze_array_network()`](#network-analysis-helper). `lattice analyze-array`. |
| `calibrate_alignment_profile(serials=None, save_path="alignment_profile.json", *, frames=3, method="orb", model="rigid", reference_serial=None, checkerboard=None, control_points=None, preview=False, vignette=False, roi=None, mask_path=None, rms_threshold_px=None, per_cam_overrides=None, settings=None, ...)` | Resuelve un perfil de alineación a partir de fotogramas en directo y guárdalo. `method`: `orb`, `akaze`, `phase`, `checkerboard` (junto con `checkerboard="9x6"`, las esquinas interiores) o `manual` (junto con `control_points={serial: [[x, y], ...]}`, las de la cámara de referencia también). `model`: `translation`, `rigid`, `affine` o `homography`. Las opciones más detalladas son los parámetros de la CLI: `max_features`, `ratio_threshold`, `matcher` (`bf` / `flann`), `knn_k`, `prefilter` (`none` / `gradient` / `clahe` / `blur` / `hist_match`), `ransac_threshold_px`, `ransac_max_iters`, `ransac_confidence`, `averaging`, `lock_rotation`, `lock_scale`, `lock_axis`; `per_cam_overrides` es `{serial: {option: value}}`. `rms_threshold_px` se niega a guardar una solución peor. Devuelve `save_path`, `output_width` / `output_height`, `crop_roi`, `reference_serial`, `transforms` (por cámara `reprojection_error`, `num_matches`) y `preview_path`. `lattice align-calibrate`. |
| `capture_aligned_multiband(profile_path, output_dir="output", *, serials=None, format="tiff", bit_depth=None, gpu=None, no_crop=False, bands=None, order=None, vignette=False, per_band=False, per_camera=False, interpolation="linear", border_mode="constant", border_value=0, settings=None)` | Una imagen multibanda alineada a lo largo del perfil: `files` (el archivo TIFF multibanda con su `bands` y tamaño, más una por banda con `per_band` y una por cámara con `per_camera`) y `vignette_corrected`. `bands` selecciona y `order` ordena las bandas por nombre (una lista o una cadena separada por comas). `lattice align-apply`. |
| `start_aligned_stream(profile_path, output_dir="output", *, serials=None, fps=3.0, count=None, ...)` | Imágenes multibanda alineadas por flujo (las mismas opciones de deformación que `capture_aligned_multiband()`), para `count` o hasta que se detenga. Devuelve un `CliJob`; sus estadísticas finales incluyen el recuento de `frames_written` y cualquier `frame_groups_incomplete`. `lattice align-stream`. |
| `alignment_profile_info(path)` | Un archivo de perfil resumido como `align-info` lo imprime así: `name`, `created`, `method`, `reference_serial`, `sensor_resolution`, `output_width` / `output_height`, `crop_roi`, `camera_serials`, `transforms` (por cámara: `model`, `reprojection_error`, `num_matches`, `reference`), `camera_models`, `layer_order` (por cámara: `band_names`, `wavelengths_nm`, `enabled`); `profile` es el archivo completo. |
| `reorder_alignment_profile(path, order=None, enable=None, disable=None)` | Modifica el archivo de perfil in situ: `order` son nombres de bandas (cada una desplaza hacia arriba la capa de su cámara, en ese orden), `enable` / `disable` son números de serie. Devuelve el nuevo `layer_order`. `lattice align-reorder`. |
| `index_catalog()` | Los preajustes de índice (`presets`: cada uno de `name`, `channels`, `formula`) y de color `gradients`. `lattice index --list-presets` / `--list-gradients`. |
| `compute_index(input_path, *, preset=None, formula=None, channel_map=None, output=None, capture_level="unknown", output_format="all", colorize=False, gradient="RdYlGn", vmin=None, vmax=None, percentile=None, bg_mode="clip")` | Un índice sobre un archivo TIFF multibanda alineado: un `preset` (`NDVI`, `NDRE`, ...) o un `formula` sobre los símbolos `channel_map` que se asignan a bandas (`{"NIR": "NIR_850", "Red": 0}`, por nombre o índice). Un «preset»símbolo `channel_map` omite y selecciona la única banda del archivo cuyo nombre comience por él (`red` -> `Red_660`, `nir` → `NIR_850_RGN`), cuyo nombre viene determinado por el propio archivo (su lista de bandas o el `Chloros:Band{R,G,B}Name` de una captura o producto de Chloros), por lo que el NDVI en una captura de reflectancia de Chloros no necesita `channel_map`; se rechazan las dos bandas que corresponden a un símbolo, no se adivinan. `capture_level` indica qué contienen las bandas: `reflectance` o `radiance` convierten los valores en cuantitativos; una pila sin calibrar (la de `align-apply`) proporciona valores cualitativos; `"unknown"` lo lee del archivo, incluido el `Chloros:ProcessingLevel` de un archivo Chloros. Devuelve `stats` (forma, el rango del percentil 1-99, el rango de la LUT, el `capture_level` utilizado) y el `files` escrito junto a la entrada (`<input stem>_<preset>.tif`, `..._LUT.png`) o en `output`, una ruta: si tienen extensión de imagen, los archivos se nombran a partir de su raíz (`output="D:/out/ndvi_FRGN.png"` genera `ndvi_FRGN.tif` y `ndvi_FRGN_LUT.png`); si no la tienen, se trata de un prefijo (`D:/out/ndvi` → `ndvi.tif`, `ndvi_LUT.png`); lee las rutas a partir de `files`. Contienen los datos de registro de la cámara de la entrada (marca / modelo / número de serie, GPS, objetivo, hora de captura en UTC, la identidad de calibración, pero nunca su calibración raw-DN), `Chloros:ProcessingLevel=index`, y el índice como su única banda (`XMP-Camera:BandName`, tanto el TIFF como el PNG de la LUT). Solo archivos; sin cámara. `lattice index --input`. |
| `capture_live_index(profile_path, *, preset=None, formula=None, channel_map=None, serials=None, output_dir="output", output_prefix="index", capture_level="reflectance", save_multiband=False, ...)` | Captura, calibra (`reflectance` o `radiance`; `raw` se salta este paso), alinear y calcular un índice en un solo paso; `save_multiband=True` conserva la pila calibrada, que `compute_index()` puede utilizar más tarde. Devuelve `stats` (junto con el `serials` cuyas bandas ha utilizado) y el `files`. Abre `serials` (la primera activa el disparador); de lo contrario, las cámaras del perfil cuya entrada `layer_order` esté habilitada, la de referencia en primer lugar (se abre incluso con sus bandas desactivadas); una cámara cuyas bandas estén desactivadas nunca se calibra. `channel_map` solo es necesario para un símbolo al que la pila no puede responder, como en `compute_index()`: el símbolo de un preajuste toma la única banda habilitada cuyo nombre comience por él, lo cual se comprueba antes de que se abra ninguna cámara (si ninguna o varias responden, se rechaza, nombrándolas); un `formula` personalizado necesita cada símbolo. `index.tif` y la LUT contienen la cámara de la pila, la hora de captura (la exposición de la cámara de referencia, UTC) y el objetivo, y ambos designan el índice como su única banda (`XMP-Camera:BandName`, p. ej. `NDVI`: el archivo PNG en color indica el índice, no sus canales de color). `lattice index --live`. |

### `CliJob`

El identificador que devuelven las tres secuencias. El trabajo se ejecuta en el backend, que mantiene el control de sus cámaras hasta que se detiene.

| Miembro | Descripción |
| --- | --- |
| `stats(timeout=10.0)` | `running`, `state`, `stats` (fotogramas guardados / escritos, descartados, `fps`, `elapsed`, errores de escritura, `stop_reason`; en un trabajo continuo también `frames_grabbed` y `frames_queued` —cada fotograma capturado se escribe, se pone en cola, se descarta o falla— y `drop_reason`, `writers` y `write_ms`, el tiempo medio que tardó en escribirse un fotograma) y `output_dir`. `state` es `running`, `ended` (se ha detenido por sí mismo y aún no se ha detenido), `stopping` (una parada está vaciando su cola: estadísticas en tiempo real, y `stopped_by`) o `finished` (sus estadísticas finales, las de la parada `error` y `stopped_by`). El registro de un trabajo detenido se conserva durante 10 minutos (los últimos 32 como máximo); para cada cliente: otro script o `chloros-mcp` lee el mismo estado y las estadísticas finales. Un trabajo `chloros-cli` iniciado (`lattice continuous`, `multi-stream`, `align-stream`) es detenido por el backend cuando la CLI deja de enviar confirmaciones durante 30 s —se ha cerrado, su máquina está en suspensión— con `stop_reason`, `client_gone` y `stopped_by` en el backend; leer sus estadísticas desde aquí no lo mantiene activo. Un trabajo iniciado desde el SDK se ejecuta hasta que se detiene. Cada inicio envía su `timeout` como fecha límite del backend (`client_deadline_s`): una cámara que se abre después de que esta (una que aún se está iniciando) se restablezca y se cierre, y no se inicie ningún trabajo, y la llamada genera `ChlorosConnectError` (`CLIENT_DEADLINE_PASSED`). Una cámara que mantiene un trabajo es rechazada para cualquier otra apertura (`connect_camera()`, otro inicio, una captura transitoria) con el código HTTP 409 `CAMERA_HELD`, indicando el trabajo y quién lo inició. |
| `stop(timeout=30.0)` | Detenerlo y cerrar sus cámaras: `final_stats`, `output_dir` y `error` cuando la detención no se ha realizado correctamente. Una segunda llamada devuelve la misma respuesta, al igual que una orden de parada desde otro cliente: una enviada mientras la primera aún está escribiendo su cola espera a que termine (hasta 25 s; pasado ese tiempo, se genera un error y las estadísticas finales aparecen en `stats()`). |
| `wait(timeout=None, poll_s=0.5)` | Esperar a que finalice por sí solo (al alcanzar su `count`, una unidad llena o un error) y, a continuación, `stop()` para recopilar los recuentos finales. `timeout` genera un error `TimeoutError` mientras sigue ejecutándose; un trabajo sin `count` solo finaliza cuando se detiene. |
| `job_id`, `kind`, `output_dir`, `start_reply`, `stopped` | Su ID y tipo (`continuous`, `multi-stream`, `align-stream`), dónde escribe, la respuesta de la ruta de inicio y si se ha ejecutado `stop()`. |

Úsalo como gestor de contexto para detenerlo al salir del ámbito.

---

## Carpeta del proyecto y recetas

**La carpeta del proyecto** es la carpeta en la que se crean los nuevos proyectos, compartida por la aplicación Chloros, la CLI y el SDK (módulo `chloros_sdk.system_utils`; `chloros-cli project-folder`):

```python
print(chloros_sdk.project_folder())                # the configured folder, else the default
chloros_sdk.set_project_folder("D:/Flights")       # created if missing
chloros_sdk.reset_project_folder()                 # -> {"path": <default>, "was_set": True}
```

Si una carpeta configurada ya no existe, se recurre a la carpeta predeterminada (`Documents/MAPIR/Chloros Projects`), al igual que hace la aplicación. Una ruta relativa indicada para `set_project_folder()` se toma de tu directorio de trabajo cuando el backend se encuentra en esta máquina.

**Una receta** es la lista de acciones en YAML/JSON que `chloros-cli project run PATH RECIPE` ejecuta sobre un proyecto guardado. `run_project_recipe()` la ejecuta con el propio ejecutor de la CLIpropio ejecutor (instalado con el SDK), por lo que una receta se comporta de la misma manera desde cualquiera de los siguientes:

```python
r = chloros_sdk.run_project_recipe("D:/campaign/morning.yaml", "D:/Projects/Field_A")
print(r["ok"], r["actions"])
print(r["log"])                                    # what the CLI would have printed

chloros_sdk.run_project_recipe({"actions": [{"apply": {"name": "FrontLeft", "settings": {"exposure_us": 5000}}},
                                            {"wait": "2s"},
                                            {"capture": {"name": "FrontLeft", "output": "pose_a/", "format": "tiff"}}]},
                               "D:/Projects/Field_A")
```

```python
run_project_recipe(recipe, project_path, *, dry_run=False, echo=False,
                   backend_url="http://127.0.0.1:5000") -> dict
```

- **`recipe`** es un archivo `.yaml` / `.yml` / `.json` o la propia receta: un diccionario con `actions` (`apply`, `capture`, `burst`, `stream`, `sensor`, `wait`), y opcionalmente `overrides` y `stop_on_error` (por defecto, «true»: la ejecución se detiene en la primera acción que falla). Las acciones son las descritas en la referencia de la CLI [Recipe DSL](cli-reference.md#recipe-dsl); una ruta relativa `output` se resuelve en relación con el directorio de trabajo de tu script.
- **Devuelve** `ok` (todas las acciones se han completado con éxito), `actions` (cuántas), `log` (la salida del ejecutor; `echo=True` también la muestra mientras se ejecuta) y `validation`. Si falta un archivo de receta o no se puede analizar, se genera el error `ChlorosConnectError`.
- **Se comprueba antes de su ejecución.**`validation` es `{"ok", "errors", "warnings", "steps"}`: cada acción debe tener exactamente un verbo conocido como clave (`{"capture": {"name": ...}}`, no `{"action": "capture"}` ni `{"type": "capture"}`) que designe una cámara, una matriz o un sensor del que disponga el proyecto, del tipo adecuado. Una receta con un error**no** ejecuta ninguna acción (`ok` false, los errores en `log`), independientemente de lo que indique `stop_on_error`. Las claves no utilizadas, así como cualquier matriz o sensor de luz que la receta controle y que no esté conectado en ese momento, son `warnings`. Una receta no conecta dispositivos: conecta primero el proyecto (`ChlorosProject.connect_all()`, `chloros-cli project connect`, o la herramienta MCP `connect_project_devices`). `dry_run=True` no ejecuta nada y devuelve `{"recipe": ..., "validation": ...}`.
- En un proceso solo se ejecuta una receta a la vez. Una receta YAML necesita PyYAML.

---

## Automatización de proyectos — `ChlorosProject`

Un proyecto de Chloros guardado es una carpeta que contiene `cameras.json` + `sensors.json` + `project.json`. `open_project(path, relink=None)` carga el manifiesto (`relink`: la carpeta en la que se encuentran ahora los archivos fuente que faltan de un proyecto movido, utilizada cuando el backend lo carga — `process()` — véase **Un proyecto movido** en `ChlorosLocal`), y `connect_all` pone en línea todas las cámaras, matrices y sensores de luz guardados con su configuración guardada —el mismo estado del hardware que produciría la interfaz gráfica de usuario—. Los sensores de luz se conectan a través del [grupo de sensores DAQ](#daq-sensor-sessions), por lo que necesitan un backend en ejecución (véase [`SensorHandle`](#sensorhandle)).

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

# Light sensors are addressable by name OR slot_id
reading = proj.sensors["Sky"].read()   # (spectrum, is_saturated, ...) or None

# One frame from every camera and array, one spectrum from every sensor
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
| `connect_all(*, cameras=True, arrays=True, sensors=True, verbose=False, align=None, packed=None)` | Detecta y conecta todas las cámaras, matrices y sensores de luz guardados. Devuelve un informe de conexión por clase (`{"cameras": {...}, "arrays": {...}, "sensors": {...}}`, cada uno con `connected`, `missing` y `errors`; `arrays` también incluye `alignment`). Si hay un backend a la escucha en `127.0.0.1:5000`, las cámaras y matrices guardadas se localizan y se conectan a través de él. Si no hay ninguno, este proceso las localiza y las controla por sí mismo a través de `lattice_sdk`, que requiere el entorno de ejecución del SDK de Arena y OpenCV (`cv2`); el paquete SDK no instala ninguno de ellos; cuando no puede hacerlo, todas las cámaras y matrices guardadas aparecen en la lista de `errors` junto con el motivo, y `missing` permanece vacío. Un backend que se está ejecutando pero rechaza el SDK (conexión cancelada: HTTP 401; un plan por debajo de Copper: 403) no se considera «sin backend»: cada cámara, matriz y sensor de luz guardados aparecen en la lista de `errors` con el motivo del backend, y no hay nada abierto en este proceso. Cada llamada vuelve a comprobar el backend, por lo que `connect_all()`, tras iniciar sesión o tras iniciar Chloros, se conecta al mismo objeto de proyecto. Una matriz guardada que ya esté activa en el backend con las mismas cámaras (las de la interfaz gráfica de usuario o las de `chloros-cli project connect`) se adjunta tal cual, sin volver a aplicar su configuración guardada, y `disconnect_all()` lo deja en ejecución. Los sensores de luz siempre pasan por el backend (véase [`SensorHandle`](#sensorhandle)). Nunca genera un backend. |
| `disconnect_all()` | Desmontar todo. Primero se libera cada dispositivo; a continuación, si el backend no ha confirmado la liberación de una cámara, una matriz o un sensor de luz (el desmontaje ha agotado el tiempo de espera o ha sido rechazado), se asigna un nombre `ChlorosConnectError` a cada uno. Un dispositivo que ya no existe, o si no hay ningún backend en ejecución, permanece en silencio. Una matriz o un sensor de luz que otro cliente tenía abiertos se deja en ejecución. La salida del proyecto `with` nunca se activa. |
| `capture_all(output_dir=".")` | Un fotograma de cada cámara y matriz conectadas, y un espectro (`SensorHandle.read()`) de cada sensor de luz conectado, capturados por turnos (primero las cámaras, luego las matrices y, por último, los sensores), no en un mismo instante. |
| `stream(camera, overlays=False, fps=10.0)` | Generador que produce fotogramas BGR `numpy` a partir de una cámara (o matriz) con nombre. `overlays=False` lee fotogramas con el identificador`grab()` en el nivel `debayered`: a través del backend cuando el proyecto se conecta a través de uno; de lo contrario, un bucle de captura `lattice_sdk` directo (las matrices generan diccionarios `{serial: frame}`). `overlays=True` se enruta a través de `ChlorosLocal.camera_stream()` → la señal MJPEG del backend, con el bloque `ui.overlay` guardado de la cámara pasado como parámetros de consulta. Requiere el modo backend, OpenCV (`cv2`) y una **cámara independiente**: una cámara en modo directo genera el error `RuntimeError` (el backend no puede acceder a una cámara que pertenece a este proceso) y una matriz genera el error `NotImplementedError` (superpone la composición por cámara: transmite un miembro por su nombre). Equivalente de un solo paso: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Ejecuta la alineación en cada matriz conectada actualmente. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Ejecuta el proceso de calibración e indexación en las imágenes del proyecto (envuelve `ChlorosLocal.process`; estos cuatro son los **únicos** argumentos clave aceptados — `indices=`, etc. genera un error `TypeError`; establece los índices mediante `ChlorosLocal.configure()`). Construye de forma diferida un `ChlorosLocal()`, que inicia automáticamente un backend. |

Atributos:
- `proj.cameras` — `Dict[str, CameraHandle]` con clave de nombre Y número de serie.
- `proj.arrays` — `Dict[str, ArrayHandle]` con clave de nombre Y array_id.
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

**Niveles de procesamiento.** `capture()`, `grab()` y `frame_stream()` utilizan todos el mismo token `processing`, y la cadena es acumulativa: cada nivel ejecuta todo lo que hay por encima de él:

| Nivel | Salida | Notas |
| --- | --- | --- |
| `raw` | Bayer de 1 canal, nativo del sensor | Sin demosaico. Las superposiciones no están disponibles en este nivel. |
| `debayered` | Color de 3 canales (**predeterminado** para `CameraHandle` y todos los `grab()` / `frame_stream()`; `ArrayHandle.capture()`, `capture_smart()` y `stream()` se establecen por defecto en `raw`): BGR en memoria (`grab()` / `frame_stream()`), guardado como RGB por `capture()` | En modo backend, como captura: la compensación de oscuro (DSNU) del paquete de calibración aplicada a la imagen sin procesar, seguida del demosaico sensible a los bordes de OpenCV —sin LUT de linealidad, campo plano ni desmezclado—. El único nivel que funciona sin el modo backend es el demosaico bilineal simple de la biblioteca de la cámara (sin desplazamiento oscuro). El archivo TIFF guardado es RGB fotométrico, así que invierte los canales de un fotograma en memoria (`frame[..., ::-1]`) para compararlo con un archivo. |
| `radiance` | float32, W/m²/sr/nm, **un plano por banda** | Cadena radiométrica completa: demosaico + desmezclado 3×3 (multiespectral) + DSNU + campo plano + escala NIST, con la exposición × ganancia eliminadas para que los valores sean absolutos. |
| `reflectance` | uint16, 32768 = 1,0 (o uint8, 255 = 1,0 — véase `bit_depth`), **un plano por banda** | Radiancia dividida por la irradiancia descendente (ρ = π·L/E). Requiere una lectura DLS/DAQ — véase la nota más abajo. |
| `display` | Similar a sRGB de 8 bits | Representación equivalente a la de la interfaz gráfica de usuario: CCM + balance de blancos + gamma a través del perfil de color activo de la cámara. |

En modo directo (sin backend) solo está disponible `debayered`: `grab()`, en otro nivel, anula
`NotImplementedError`, `capture()` ignora `processing`, y `frame_stream()` eleva
`RuntimeError` en todos los niveles. `reflectance` necesita una lectura descendente válida: el punto final del fotograma introduce
automáticamente los datos de adquisición agrupados en la ranura DLS de la cámara, pero si no hay ningún DAQ vinculado, la cadena rechaza la
salida de reflectancia y marca claramente la degradación en los metadatos devueltos, en lugar de
devolver un producto de menor calidad.

> **Escala DN de reflectancia: no la codifiques de forma fija.** La reflectancia de LATTICE utiliza `32768` = ρ 1,0 y marca
> XMP `Chloros:PixelScale=32768`; la reflectancia de Survey3 utiliza `65535` = ρ 1,0 y no lleva ninguna
> etiquetas `Chloros:*`. Lee la etiqueta y divide por ella. Está definida en el dominio uint16, por lo que se mantiene como
> `32768` para todos los formatos que cambian de escala (TIFF de 16 bits, TIFF de 8 bits, PNG/JPG de 8 bits, porcentaje de 32 bits)
> — normaliza primero el tipo de datos almacenado a uint16 (×257 desde 8 bits, ×65535 desde flotante). La única
> excepción es un archivo de una versión anterior a la 1.3.0: en aquel entonces, un TIFF de 8 bits se *recortaba*, no se reescalaba, por lo que
> ninguna escala lo describe, y no contiene ninguna tupla `PixelScale` ni MicaSense. Trata la ausencia de una etiqueta en un
> archivo de reflectancia LATTICE como «escala no válida», no como un valor por defecto.

**Etiquetas de calibración: solo en `raw`.** El archivo `raw` de una captura contiene el paquete de calibración de la cámara: la carga útil `CalibrationJSON` y las etiquetas compatibles con MicaSense (`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`), que describen los valores sin procesar del sensor. Los archivos de todos los demás niveles (`debayered`, `display`, `radiance`, `reflectance`, el renderizado `index/`) y cada fotograma de grabación continua o de transmisión multicámara incluye la identificación de la cámara (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...), así como el registro propio del fotograma, `XMP-mapir:FrameJSON` (exposición, ganancia, temperatura, formato de píxel, hora de captura, coincidencia DAQ), ya que cada producto `process()` lleva su propia identidad. El procesamiento localiza la calibración de dicho archivo mediante su número de serie y `CalBundleSha256` en la caché de calibración de Chloros, descargando el paquete una vez si este ordenador no lo tiene.

#### Archivos creados a partir de varias cámaras

Un archivo creado a partir de las bandas de más de una cámara —una pila multibanda (`capture_aligned_multiband()`, `start_aligned_stream()`, `capture_live_index(save_multiband=True)`, `Registration.save_multiband()`), un índice en directo creado a partir de varias cámaras (`index.tif`, `index_LUT.png`) o una imagen `composite/` de una captura en matriz — incluye el nombre de todas las cámaras que se han utilizado:

- **Por-listas por cámara, en orden de banda** (la composición: capa de fondo, luego primer plano; una capa de índice, las cámaras de las que proceden sus bandas): `XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceSensorProfiles`, `SourceProductLines`, `SourceFilterTypes`, `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s` (el paquete de calibración por el que han pasado las bandas de cada cámara; en caso contrario, una pila `capture_aligned_multiband()` / `start_aligned_stream()`, cuyas bandas no pasan por ninguna: el paquete que Chloros ha almacenado en caché, el que da nombre a sus archivos de captura), `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers` — listas paralelas, en las que el elemento *n* de cada una describe la misma cámara; `-`, donde una cámara no tiene ningún valor (sin paquete de calibración, un objetivo desconocido). Cada cámara es **un** elemento: cuando su paquete tiene varios valores para un campo —los certificados de lámparas halógenas de un paquete —, se unen con `;` (`F-1479-1;F-1480-1;F-1481-1`), nunca con una coma, por lo que una lista muestra un elemento por cámara independientemente de cómo se imprima.
- **EXIF `SerialNumber`** contiene los mismos números de serie, separados por comas (`214000535,214000551`).
- **No** contiene ninguna identidad de cámara individual (`XMP-mapir:CameraSerial`, `CalBundleSha256`, …): estos identifican una cámara y su paquete, y el procesamiento determinaría la calibración del archivo a partir de ellos. Un índice calculado a partir de las bandas de varias cámaras (`index.tif`, `index_LUT.png`) tiene EXIF `Model` `LATTICE` (el equipo), y no el modelo `LATT-…` de una sola cámara.

Un archivo con las bandas de **una** sola cámara —un TIFF por banda, un fotograma `capture_aligned_multiband(per_camera=True)`, una pila o un índice en tiempo real de una sola cámara— contiene las etiquetas de identidad propias de esa cámara, al igual que cualquier otro archivo de captura. La pila de `capture_aligned_multiband()`, los archivos por banda y por cámara, así como cada pila de `start_aligned_stream()` y sus archivos por banda, contienen la hora de captura de su grupo (la exposición de la cámara de referencia, UTC), y un fotograma `per_camera` de una cámara monocromática es un plano, en `bit_depth` al igual que la pila.**Orientación de la cámara (volteo/rotación).** Los fotogramas de `grab()` / `frame_stream()` siempre se encuentran en la
orientación nativa de la cámara, en todos los niveles. Las capturas guardadas siguen la configuración de orientación
de la cámara (las opciones «Mirror H» / «Mirror V» / rotación de la interfaz gráfica de usuario):

- **Los archivos `raw` y `debayered` son exactamente lo que lee el sensor**, sea cual sea la configuración: un mosaico de Bayer
  girado o volteado sería un patrón de filtro de color diferente, y los mapas de calibración se basan
  en la propia cuadrícula del sensor.
- **Los archivos `display`, `radiance`, `reflectance` y los archivos de índice están orientados** —girados una vez, tras todos los
  cálculos de calibración. Sus máscaras de recorte se encuentran en la misma cuadrícula.
- **Cada archivo de captura registra la configuración** en XMP: `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`), `ChlorosRotation` (0 / 90 / 180 / 270, en el sentido de las agujas del reloj, tras
  los giros) y `ChlorosOrientationApplied` — `false` en los productos sin procesar y sin debayering, `true` en los productos orientados. `ChlorosProcessing` enumera las operaciones aplicadas a los píxeles de ese archivo (`none` cuando
  es nativo). Esto incluye todos los fotogramas de una grabación continua (la configuración propia de la cámara,
  sin aplicar) y de una secuencia multicámara (la identidad: sus fotogramas son nativos, como los de una
  captura en matriz).
- **El procesamiento de un proyecto aplica la orientación registrada** a los productos que genera a partir de un
  (primero la calibración en coordenadas del sensor, luego la distorsión de alineación de la matriz y, por último, la orientación),
  para que coincidan con los productos de la captura en directo —incluidos los productos de reserva sin calibrar de respuesta del sensor y con corrección de viñeteado
  —; su producto sin debayering permanece nativo.
- **El visor de imágenes de Chloros muestra un archivo sin procesar o sin «bayering» girado según su registro** —solo a efectos de visualización,
  el archivo no se modifica; un archivo sin el registro se muestra tal y como está almacenado.
- **Las capturas sin el registro se procesan como nativas.** Antes de la versión 1.3.0, una captura con una sola cámara
  convertía por sí misma sus archivos sin procesar y sin bayering; Chloros no puede distinguir un archivo sin procesar de uno nativo y
  no hace suposiciones.
- **Las capturas en matriz mantienen todos los archivos miembros en formato nativo por defecto** y registran la identidad;
  `ArraySession.capture(orient=True)` procesa sus productos (véanse los controles de captura de matrices).

#### Salida por banda y matrices mixtas

En los niveles `radiance` y `reflectance`, una cámara devuelve **un plano por banda de dicha
cámara** — tres para una M3C (sus tres bandas sin mezclar, orden de canales BGR), uno para una M3M (su
único filtro de banda estrecha). Los planos replicados de la cámara monocromática se agrupan, por lo que una **matriz mixta
M3C/M3M produce diferentes formas de matriz por serie**. Lee la clave `bands` en los metadatos del fotograma
para saber qué es cada plano; no lo deduzcas a partir del recuento de canales:

```python
for serial, (arr, hdr) in arr_handle.grab(processing="reflectance",
                                          with_metadata=True).items():
    print(serial, arr.shape, hdr["bands"])   # ('…', (1536, 2048, 3), ['nir','green','red'])
                                             # ('…', (1536, 2048, 1), ['F685'])
    rho = arr.astype("float32") / hdr["pixel_scale"]
```

Comprueba siempre `hdr["processing_level"] == "reflectance"` antes de utilizar los valores. Una degradación
(sin lectura DAQ, sin paquete de calibración, disposición de bandas desconocida) se indica en `processing_level` y
`radiometric_refused` en lugar de señalarse como elevada; la matriz que se obtiene sigue siendo una trama válida, simplemente
no es el producto que has solicitado. Una captura guardada que ha sido degradada lleva el mismo código en su XMP
que `Chloros:RadiometricRefused` (p. ej., `no-band-layout`, `no-dls-snapshot`).

#### Un disparador por lectura — `group_id`

**Los miembros de la matriz se sirven desde el grupo de fotogramas sincronizados de la matriz, no desde el propio
búfer de captura de cada cámara.** Las cámaras de una matriz sincronizada por hardware se exponen conjuntamente, pero las N lecturas por cámara
no se *muestrean* juntas: la lectura del último fotograma almacenado en el búfer de cada cámara traspasa los límites del disparador, y un solo fotograma perdido desincroniza el conjunto sin que nadie se dé cuenta.

El suministro desde el grupo es necesario, pero no suficiente por sí solo: cada solicitud sigue
captura *el último* grupo en su propio instante, y el bucle de captura puede publicar entre medias.
Según mediciones realizadas en una matriz en directo de 4 cámaras, **12 de 20 lecturas paralelas abarcaban dos grupos** —siempre
exactamente adyacentes—. Por lo tanto, la lectura queda **fijada**:

1. Cada fotograma lleva `group_id`, el contador monótono de publicaciones del conjunto.
2. `grab()` lee la cámara maestra (o, si no se puede atender a la cámara maestra, la siguiente cámara que pueda),
   y luego fija las cámaras restantes al identificador de grupo en el que ha aterrizado (`?group_id=N` en el punto final HTTP).
3. El backend conserva los grupos recientes durante **al menos medio segundo**, por lo que un grupo homólogo ya
   superado por publicaciones posteriores sigue respondiendo a ese disparador —tiempo suficiente para que la
   de la exportación de reflectancia por cámara, incluso en una matriz de 9 cámaras con sensor completo. Una asignación que el backend ya no
   mantiene se rechaza con `409` y `group_aged_out: true`, en lugar de responderse silenciosamente
   con un disparador diferente; `grab()` vuelve a leer el conjunto.

Remedido con fijación: **20 de 20 lecturas devolvieron un único disparador.** `grab()` se encarga de esto por ti;
solo necesitas `group_id=` cuando accedes directamente al punto final HTTP.

- Una cámara a la que no se puede dar servicio en el grupo fijado —ya sea porque no figura en él o porque rechaza la solicitud
  (un `band` que no tiene, un nivel que no puede generar)— **se omite en lugar de rellenarse retroactivamente**
  a partir de un disparador diferente, y las demás cámaras siguen apareciendo en la respuesta.
- El diccionario que devuelve `grab()` es un `GrabResult` (una subclase simple de `dict`;
  `from chloros_sdk import GrabResult`): `.errors` asigna cada
  número de serie omitido a la excepción que provocó su lectura, y `.group_id` es el grupo compartido. Un
  `.errors` vacío significa que todas las cámaras forman parte de este grupo.
- Una cámara independiente no pertenece a ningún grupo, informa de `group_id: None` y **rechaza** un `group_id=`
  en lugar de ignorarlo.

```python
frames = arr_handle.grab(processing="reflectance")
for serial, err in frames.errors.items():
    print(f"SN:{serial} omitted: {err}")       # e.g. status=400 ... band 'F685' not in [...]

frames = arr_handle.grab(processing="reflectance", with_metadata=True)
ids = {hdr["group_id"] for _a, hdr in frames.values()}
assert len(ids) == 1, f"straddled triggers: {ids}"     # belt and braces
```

El mismo contador se publica en **cada flujo MJPEG por cámara** de un miembro de la matriz: la
vista previa (`/api/camera/stream/<serial>`), la vista previa anotada (`/stream-annotated`) y el
flujo de reflectancia (`/stream-reflectance`)— como `X-Chloros-Group-Id` en cada parte, además de
`X-Chloros-Frame-Id` y `X-Chloros-Timestamp-Ns`. Las transmisiones son continuas en lugar de
puntuales, por lo que se emparejan recopilando partes y agrupándolas en función de ese encabezado; no hay ningún
pin que configurar. Una cámara independiente no tiene grupo. Véase
[Flujos MJPEG por cámara](#per-camera-mjpeg-streams-no-rtsp).

**`frame_stream()` nunca se queda en silencio.** Se omite un tick que falla por un motivo que puede resolverse por sí solo
: un flujo de cámara no devuelve nada para él, un flujo de matriz devuelve las cámaras que sí
respondieron (posiblemente ninguna). Esas razones son: aún no hay fotograma; el hilo de captura de la cámara se ha detenido durante un
segundo o dos (`404 not streaming`, a lo que el backend responde cada vez que otra cámara se conecta
o se desconecta, o cuando un cambio de configuración reequilibra el grupo); un reinicio del backend o `5xx`; la alineación
aún se está estabilizando (`409 … retry shortly`); y un rechazo de reflectancia porque el DAQ aún no tiene ninguna lectura
(`no-dls-snapshot`, una lectura obsoleta o vacía). Un rechazo en el siguiente tick solo se repetiría: un
`band` desconocido, `bit_depth=8` en radiancia, un paquete de calibración sin escala de radiancia, un rechazo de reflectancia
**sin ningún DAQ en absoluto** (ninguno vinculado a la cámara ni a su matriz, ninguno conectado al
backend: el `dls_source` del rechazo es `"none"`), cualquier otro `4xx` —
genera un `chloros_sdk.FrameFetchError` (un `RuntimeError`, que lleva consigo `.status` y `.body`) de
una sola vez. Lo mismo ocurre con **60 segundos** de ticks omitidos sin ningún fotograma entre ellos, sea cual sea el `fps` — lo suficientemente
largo como para superar la estabilización de la alineación y un reinicio del backend, lo suficientemente corto como para que una cámara que
realmente ha dejado de funcionar termine la transmisión. Ese error de abandono también es un `FrameFetchError`, tanto para una cámara como para
una matriz, con la propia excepción de la última lectura en `.errors` (y, para una cámara, como `__cause__`). Para
una matriz, «rechazado» significa que todas las cámaras han sido rechazadas; la excepción `.errors` contiene el motivo de cada cámara.

**Un conjunto `frame_stream()` genera cada desencadenante una vez** (tanto `ArrayHandle` como `ArraySession`). Un
tick cuyo conjunto más reciente sea el disparador que ya ha generado espera al siguiente disparador en lugar de
generar el mismo conjunto de nuevo, por lo que el flujo se ejecuta a la velocidad más lenta entre `fps` y la propia frecuencia de fotogramas, y los conjuntos consecutivos nunca comparten un `group_id`. Un conjunto que deja de publicar genera
`FrameFetchError` («sin nuevo disparador durante 60 s») en lugar de repetir su último conjunto. Sin esto, una
transmisión que solicitara más de lo que publicaba la matriz —o que se iniciara justo después de la conexión— generaría
repeticiones: en una ejecución medida, 20 conjuntos contenían 11 identificadores de grupo distintos.

```python
from chloros_sdk import FrameFetchError

try:
    for arr in cam.frame_stream(processing="radiance", bit_depth=8):
        my_analysis(arr)
except FrameFetchError as exc:
    print(exc.status, exc.body)        # the HTTP status and the backend's reason
```

#### Exportación de una sola banda — `band`

`grab()` / `frame_stream()` y `band="<name>"` se utilizan para exportar solo una de las bandas de una cámara como un
matriz `(H, W, 1)` de un solo canal — la misma forma que produce una cámara monocromática, por lo que la salida de una sola banda se
gestiona de forma idéntica, tanto si procede de una M3M como de la división de una M3C. La banda roja de una FRGN:

```python
red = cam.grab(processing="reflectance", band="red")        # (1536, 2048, 1)
```

Las formas de escritura aceptadas son el nombre completo del diseño (`Red_660`, `NIR_850`, `Green_550`) o su abreviatura
( (`red`, `nir`, `green`), sin distinción entre mayúsculas y minúsculas. Un nombre desconocido se rechaza y se muestran los
nombres disponibles.

> **La selección se realiza por nombre, no por número de canal —de forma deliberada—.** El «Canal 1» es `Green_550` en
> el orden de canales BGR del array, pero *rojo* en el orden de las letras del código del filtro(`RGN`), y ambos
> no coinciden. Si se elige el incorrecto, se obtiene una trama perfectamente plausible de la banda equivocada, que
> nada más adelante en la cadena puede detectar. Por ese motivo no hay ningún índice numérico.

#### Alineación de las secuencias — `align`

`align=True` transforma el fotograma al fotograma de referencia común de su matriz. Cada cámara se transforma
mediante la **misma** resolución de perfil y la misma zona de interés (ROI) recortada, por lo que las secuencias de cada cámara, aunque se abran de forma independiente,
resultan **registradas entre sí**; no es necesario solicitarlas juntas:

```python
frames = arr_handle.grab(processing="reflectance", align=True)   # {serial: array}
# or one MJPEG stream per camera, each co-registered on its own:
#   GET /api/camera/<serial>/stream-reflectance?align=1
```

Requisitos y comportamiento:

- La cámara debe estar en una matriz **sincronizada por hardware** (cable de sincronización M8) que cuente con un perfil de alineación
  — el registro conjunto supone que los miembros se exponen simultáneamente, algo que una matriz de funcionamiento libre por software o
  matriz de funcionamiento libre no garantiza. De lo contrario, se rechaza con el código `409`.
- **Rechazado; nunca se deja sin alinear de forma silenciosa.** La ruta de captura puede guardarse sin alinear con una advertencia
  porque un archivo conserva sus etiquetas de procedencia y puede volver a registrarse más tarde; una transmisión en directo no tiene
  esa segunda oportunidad, por lo que la falta de un perfil es un error y no una solución alternativa.
- **Solicitar la alineación la inicia.** Solicitar `align` en una matriz sin perfil pone en marcha la
  calibración y devuelve `409` con lo que está esperando (p. ej., *«esperando a que la exposición
  se estabilice en 214000551 (1/3)»*) — si se vuelve a intentar, se consigue, normalmente en unas pocas decenas de segundos.
  Esto es importante en modo sin interfaz gráfica: la alineación automática en el momento de la conexión se da por perdida tras 30 s, y cada intento
  dentro de ese intervalo se omite mientras el AE inteligente sigue estabilizándose, por lo que una matriz cuya exposición
  converge lentamente solía quedarse sin perfil para toda la sesión y sin forma de obtener
  uno sin abrir la vista combinada del escritorio. `alignment_status` también informa
  `ae_align_holdouts`, indicando las cámaras que aún se están estabilizando y su progreso.
- Asigna a cada flujo de una matriz el **mismo `max_h`**, o sus recortes se situarán en diferentes
  rejillas de píxeles.
- La alineación se aplica *antes* de la codificación de profundidad de bits, por lo que la deformación interpola valores de precisión completa.

#### `bit_depth` — sacrificar precisión a cambio de ancho de banda

`grab()` y `frame_stream()` admiten `bit_depth=16` (por defecto) o `bit_depth=8`:

| `bit_depth` | Codificación de reflectancia | Coste |
| --- | --- | --- |
| `16` | uint16, 32768 = ρ 1,0, margen hasta ρ 2,0 | Precisión total. |
| `8` | uint8 que abarca `0..refl_max` (por defecto ρ 1,0) | La mitad de bytes; cuantificación de ~0,004 ρ en el límite superior por defecto. Los valores por encima del límite superior se recortan — véase más abajo. |

##### El límite de 8 bits — `refl_max`

`refl_max` establece el ρ que abarca el rango de 8 bits, por lo que `pixel_scale = 255 / refl_max` y ρ sigue siendo
`pixel / pixel_scale`. `grab()` y `frame_stream()` lo aceptan como palabra clave tanto en la cámara como en
manejadores de matriz (`?refl_max=` en el punto final HTTP). Aumenta este valor para ganar margen para los reflejos especulares,
en pasos proporcionalmente más gruesos:

```python
frame = cam.grab(processing="reflectance", bit_depth=8, refl_max=1.5)   # 0..1.5 in 0..255
```

| `refl_max` | `pixel_scale` | paso |
| --- | --- | --- |
| `1.0` (por defecto) | 255 | 0,0039 ρ |
| `1.5` | 170 | 0,0059 ρ |
| `2.0` | 127,5 | 0,0078 ρ |

> **La exposición automática no puede corregir un recorte de 8 bits, y utilizarla para intentarlo empeora los datos.**
> ρ = π·L/E una vez divididos la exposición y la ganancia, por lo que ρ es una propiedad de la escena, no de la
> configuración de la cámara —medido en una matriz en tiempo real, reducir a la mitad la exposición desplazó la media de ρ en un 1,3 %. Aumentar
> la exposición para «ajustarla» al rango, en cambio, lleva a los píxeles a la saturación del **sensor**, lo que
> subestima L y sesga ρ **a la baja** sin que se genere ningún error (medido: ρ máximo 1,203 → 0,985 con
> el doble de exposición, media −23 %). Elige el rango con `refl_max`; mantener el sensor fuera de la
> saturación es precisamente para lo que ya sirve el AE, y eso protege la *validez* de ρ más que su rango.

**Lee `reflectance_clipped_fraction`, no el indicador.** `reflectance_clipped_at_1_0` se activa con un
solo píxel: en una escena real informó de `True` mientras que la fracción era `0.000001`, un píxel
de cada millón. La fracción es lo que indica si hay que elevar el límite máximo.

**`bit_depth=8` se refiere únicamente a la reflectancia.** Si se solicita para `radiance`, la solicitud se rechaza en lugar de
atenderse: la reflectancia tiene un rango fijo de 0 a 1 que se mantiene en una codificación de 8 bits, mientras que la radiancia no tiene
un rango fijo, por lo que ajustarla a 8 bits requiere un estiramiento percentil por fotograma cuya correspondencia
cambia en cada fotograma —una imagen, no una medición—.

#### Flujo de reflectancia en directo (MJPEG)

`GET /api/camera/<serial>/stream-reflectance` ofrece reflectancia de 8 bits por banda como
`multipart/x-mixed-replace`: un M3C como JPEG de 3 canales y un M3M en escala de grises. Parámetros de consulta:
`fps` (por defecto 10), `jpeg_q` (por defecto 95), `max_h` (0 = nativo), `band` (una banda con nombre, como
se ha indicado anteriormente), `align` (`1` para el co-registro — mismas reglas que el `align` del SDK), `refl_max` (el límite máximo de
codificación de 8 bits, como arriba). Cada parte lleva
`X-Chloros-Group-Id`, `X-Chloros-Frame-Id`, `X-Chloros-Timestamp-Ns`, `X-Chloros-Bands`,
`X-Chloros-Pixel-Scale`, `X-Chloros-Aligned` y `X-Chloros-Reflectance-Clipped`.
**Las partes de diferentes cámaras que contienen el mismo `X-Chloros-Group-Id` corresponden al mismo disparador** —
así es como se reensamblan varias secuencias por cámara de una matriz, y esto está garantizado
del lado del servidor, ya que los miembros de la matriz se sirven desde el grupo de fotogramas sincronizado (véase
[Un disparador por lectura](#un-disparador-por-lectura--group_id)). Devuelve `409` cuando la reflectancia no está
disponible en el momento de la conexión y finaliza el flujo si deja de estar disponible durante la ejecución, por lo que un flujo
detenido significa «ya no hay reflectancia» en lugar de «sigue bien, solo hay estática».

> **El formato JPEG tiene pérdida de información: estos píxeles son una medición.** Los errores de compresión afectan directamente a los
> valores de reflectancia y son más graves en los bordes, que es donde se encuentran los límites de los materiales. Por ese motivo, la calidad
> se establece por defecto en 95. Si necesitas que los números se mantengan intactos, utiliza
> `grab(processing="reflectance")` / `frame_stream(...)`, que devuelven el mismo resultado sin
> códec en la ruta (y `bit_depth=8` si la limitación es el coste de la conexión, no el códec).

> **Los datos EXIF se transfieren a la exportación.** `process()` copia el bloque GPS de la captura original
> **y su ExifIFD** en cada producto, por lo que las exportaciones incluyen `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` y `CameraSerialNumber`, así como la
> georreferenciación. `FocalLength` es a partir de lo que Pix4D calcula la distancia entre muestras en el suelo; sin él,
> la reconstrucción recurre a una escala totalmente errónea (en un caso concreto, un emplazamiento de 411 m
> se convirtió en uno de 47,8 km). Se ha omitido deliberadamente el archivo `-all:all`: las etiquetas estructurales de IFD0 rompen
> la salida de LATTICE, y `ExifImageWidth`/`Height` se excluyen porque describen la
> captura original en lugar de la trama exportada.

Subindicadores de la fase de captura (se aplican a los niveles radiométricos: `radiance`, `reflectance`, `display`):

| Indicador | Predeterminado | Significado |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + campo plano + desmezclado 3x3 + escala radiométrica NIST. |
| `apply_white_balance` | `True` | LUT de balance de blancos. Compatible con DLS cuando hay un DAQ vinculado a la cámara. |
| `apply_index` | `False` | Evaluación del índice de vegetación. |
| `index_expression` | `None` | Anular fórmula. Si novacíos → activa automáticamente el índice. |
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

> **El valor de retorno es un `chloros_sdk.CapturePaths`, de tipo `CapturePathMap`, no `Dict[str, str]`.**
> `chloros_sdk.CapturePathMap` es `Dict[str, Union[str, List[str]]]`: una ruta de un solo nivel
> `processing` proporciona a cada número de serie una ruta, mientras que `processing="all"` le proporciona la
> **lista ordenada** de todos los productos guardados para esa cámara. `ArrayHandle.capture()` no admite
> ningún `levels=`, e ignora los argumentos clave que no reconoce (`levels`, `daq`,
> `render_index`, `force_daq`), así que pasa solo los que se muestran aquí (`aligned=False`
> guarda archivos nativos, al igual que en `ArraySession.capture()`); utiliza
> `ArraySession.capture()` para esos controles. Un compuesto combinado en directo, si se estuviera transmitiendo, llega bajo la clave adicional
> `"combined"` en lugar de bajo un número de serie. El código que asume `str` falla en el
> formato de lista sin que ningún verificador de tipos lo detecte —la anotación indicaba `Dict[str, str]`
> durante un tiempo después de que se lanzara la forma de lista, razón por la cual existe el alias. Normaliza
> cuando quieras la forma plana:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

> **Lo que no se podía escribir: `.failed`.** El mapa que devuelve `capture()` es una
> `chloros_sdk.CapturePaths` — una subclase simple de `dict`, por lo que todo lo anterior funciona
> sin cambios — que también incluye `.failed`: los archivos que el backend no pudo escribir,
> las mismas entradas que [`CaptureResult.failed`](#archivos-que-no-se-pudieron-escribir--captureresultfailed)
> (`serial`, `level`, `error` y `disk_full: True` en una unidad llena). Una
> captura parcial sigue devolviendo todas las rutas en las que se ha escrito; en un disco lleno, el mapa está
> vacío y `.failed` explica el motivo. `.skipped` enumera los saltos por cámara y
> `.group_seq` el grupo sincronizado, y `.alignment` / `.oriented` son los
> del backend, al igual que en `CaptureResult`. `capture_smart()` los devuelve como
> `result["frames"].failed` y `result["failed"]`.
>
> ```python
> paths = arr.capture("./out", processing="all")
> if any(f.get("disk_full") for f in paths.failed):
>     raise SystemExit("drive full -- free space and capture again")
> ```

### Alineación de matrices

`ArrayHandle` expone toda la superficie de alineación. Los perfiles son, por defecto, exclusivos de la sesión; llama explícitamente a `export_alignment()` para que se mantengan.

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

# Manual control points: each camera's pixel coordinates of the same
# features, the reference camera's included (>= 2 per camera). Every one
# of num_frames is solved from these points.
arr.calibrate_alignment(method="manual", model="rigid", control_points={
    "214701173": [[412, 300], [1630, 310], [1020, 1180]],   # reference (master)
    "214701292": [[418, 297], [1637, 306], [1027, 1177]],
})

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Alineación en el momento de la conexión

`connect_all(align=...)` puede alinear automáticamente todas las matrices en el momento de la conexión:

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

Si no se especifica, se recurre a `project.json["config"]["auto_align_on_connect"]`.

### `SensorHandle`

Un sensor de luz guardado (DAQ-U, DAQ-M o DAQ-E), abierto por `connect_all()` en el [pool de sensores DAQ](#daq-sensor-sessions) —el mismo pool que utilizan `connect_daq_sensor()` y `chloros-cli daq pool-connect`—.

```python
reading = proj.sensors["Sky"].read()
if reading is not None:
    spectrum, is_saturated, integration_time, x, y, z = reading
```

- `read()` espera el **siguiente** espectro del sensor —uno que ya esté almacenado en el búfer en el momento de la llamada no cuenta — y devuelve `(spectrum, is_saturated, integration_time, x, y, z)`, o `None` si no llega ninguno en un plazo de 5 s. Se reanuda una transmisión en pausa. `spectrum` se expresa en W/m²/nm cuando el sensor está calibrado.
- `.sensor` es el [`DAQSensorSession`](#sesiones-del-sensor-de-adquisición de datos); para todo lo demás: `latest(n=...)`, `status()`, `record_start()` / `record_stop()`.
- `.slot_id`, `.model`, `.serial`, `.name` y `.settings` son la entrada guardada de `sensors.json`.

**Lo que restaura `connect_all`.** Cada sensor se abre en su dirección guardada —su puerto COM, dirección Bluetooth o host DAQ-E— y, cuando el proyecto ha registrado el identificador del sensor, debe tratarse de ese sensor: se rechaza cualquier unidad diferente en esa dirección, que no esté conectada. Se aplican su tiempo de integración guardado, el promedio de fotogramas y la exposición automática, así como su corrección de la tapa:

| Límite guardado | Conectado con |
| --- | --- |
| Un perfil de límite (p. ej., `fov_30`) | Ese perfil. |
| Ninguna seleccionada | `sunshine_cosine`, la conexión por defecto, registrada como se supone (`cap_id_source=auto_default`), por lo que el procesamiento emite una advertencia. Los sensores se suministran sin tapa: elija la tapa que esté instalada. |
| `none` — seleccionado como **Ninguno (sensor sin tapa)** en la interfaz gráfica de usuario | Sin corrección de tapa. |

Un DAQ-E-S no admite ningún perfil de tapa, independientemente de lo que se haya guardado: su difusor forma parte de su calibración (véase [DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor)). Un proyecto guardado por un Chloros anterior registró un sensor sin tapa como opción predeterminada, por lo que se vuelve a conectar con el `sunshine_cosine`; selecciona **Ninguno (sensor sin tapa)** en la interfaz gráfica de usuario una vez más y guarda el proyecto.

Un sensor que otro cliente ya tiene abierto en el grupo (la CLI u otro script) se conecta tal cual, sin volver a aplicar la configuración, y `disconnect_all()` lo deja en funcionamiento; se libera un sensor abierto por `connect_all()`. Al igual que con `connect_daq_sensor()`, la interfaz gráfica de usuario de Chloros **no** forma parte del grupo: cierra un sensor en la interfaz gráfica antes de conectar el proyecto desde un script.**Requisitos y el informe.** Los sensores de luz necesitan un backend de Chloros en ejecución e iniciado sesión en un plan de pago de Chloros+ (Copper o superior), y `connect_all()` nunca inicia uno. Si no hay ninguno en ejecución, todos los sensores guardados aparecen en la lista de `report["sensors"]["errors"]`, indicando esta circunstancia. Los sensores que el backend no puede abrir también aparecen en esa lista, junto con el motivo indicado por el backend; `report["sensors"]["missing"]` permanece vacío para los sensores de luz.

---

## Hardware directo (sin backend)

Si deseas que las cámaras LATTICE (CI, robots sin pantalla, integradas) no dependan en absoluto del backend, importa directamente `lattice_sdk`; este es reexportado por `chloros_sdk`. Advertencia sobre `CAMERA_AVAILABLE`: `lattice_sdk` se encuentra en el paquete SDK, pero requiere que estén presentes el entorno de ejecución de Arena SDK y OpenCV (`cv2`), y el paquete no instala ninguno de los dos. Los sensores de luz pasan por el backend — [`connect_daq_sensor()`](#daq-sensor-sessions) o el `proj.sensors` de un proyecto—, con una excepción: el ayudante de magnitud de banda de DAQ-M [`DLS`](#daq-m-band-magnitudes--dlsband_magnitudes), que también necesita `bleak`.

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

Tres de los cuatro preajustes funcionan en **modo libre**: la cámara expone de forma continua y un
`capture()` devuelve el siguiente fotograma. `triggered` es la excepción : activa la
cámara ante un flanco de hardware en la línea 2, por lo que no captura nada hasta que se produzca uno.

| Preajuste | Disparador | Úsalo cuando |
| --- | --- | --- |
| `default` | funcionamiento libre | uso general |
| `high_speed` | funcionamiento libre | 8 bits, límite de 60 fps, exposición corta |
| `high_quality` | funcionamiento libre | 12 bits, sin límite de fps — la opción habitual para fotografías |
| `triggered` | **activada, línea 2** | la cámara está conectada a un cable de sincronización M8 y otro dispositivo la dispara |

Si eliges `triggered` (o configuras tú mismo `trigger_mode="On"`) sin que nada
active la línea 2, todos los `capture()` agotarán el tiempo de espera — correctamente, ya que le has pedido
a la cámara que espere. El SDK lo explica cuando ocurre; consulta
[SC_ERR_TIMEOUT durante la captura](#direct-hardware-backend-free).

> **Nota: los mensajes «GVSP probe» / `SC_ERR_TIMEOUT -1011` al conectarse no son errores.**
> Al conectarse, el SDK intenta negociar**tramas jumbo** (paquetes GVSP de 9000 bytes) para obtener un mayor rendimiento. En un enlace directo punto a punto de la tarjeta de red (por ejemplo, una dirección `169.254.x.x` local del enlace), la red normalmente no puede transportar tramas jumbo, por lo que esta sonda agota el tiempo de espera y registra líneas como:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> Esta es la **solución alternativa prevista**: el SDK vuelve automáticamente a los paquetes estándar de 1500 bytes y la cámara sigue conectándose con normalidad (las líneas `[chunk-enable …]` que siguen forman parte de la secuencia de conexión habitual). La captura sigue funcionando.
>
> Puedes omitir esta prueba, pero **no se trata solo de silenciar el registro: desactiva las tramas jumbo.** La cámara responde a los pings «Don't-Fragment» solo hasta 1500 bytes, independientemente de la calidad de tu red, por lo que la prueba de ping por sí sola nunca puede detectar tramas jumbo; esta prueba es la única que puede hacerlo. Si la desactivas, la cámara utilizará paquetes estándar de 1500 bytes para siempre, en cualquier red:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> Solo merece la pena en una red de la que *sabes* que no admite tramas jumbo, donde ahorra aproximadamente un segundo de tiempo de conexión por cámara. Dado que se trata de una diferencia real y no meramente estética, el SDK ahora lo indica cuando se utiliza:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **No lo modifiques a menos que tengas una razón de peso.**Si se deja activado, cada cámara se mide una vez por sesión de backend y de nuevo cuando cambia su enlace (otro adaptador, otra velocidad de enlace o un enlace que se ha caído, como ocurre al cambiar de conmutador); las reconexiones reutilizan el resultado.**Tras un cambio detrás de un conmutador que se ha mantenido activo, reinicia el backend para volver a realizar la medición.** La primera conexión puede tardar ocasionalmente unos minutos en esta comprobación, y el registro lo indica cuando ocurre.
>
> Si *quieres* el rendimiento de paquetes jumbo, habilita los paquetes jumbo de extremo a extremo (MTU de la tarjeta de red de 9000 + un conmutador que los transmita), o fíjalo con `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` cuando sepas que el enlace lo admite —aunque es preferible utilizar `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` por comando en lugar de configurarlo de forma permanente, ya que un tamaño fijo omite la prueba y deja de adaptarse a la red que tiene delante. **Todos** los dispositivos de la ruta deben admitir paquetes jumbo, incluidos los divisores o inyectores PoE, que suelen ser la razón por la que una configuración que, por lo demás, es compatible con paquetes jumbo, no puede transmitirlos.

> **`SC_ERR_TIMEOUT -1011` durante `capture()` / `grab*()` es un problema diferente — ese es un error real.**
> La nota anterior solo se refiere al `-1011` registrado por la**sonda de tiempo de conexión**. Si se produce el mismo error a partir de una**captura**, significa que la cámara se ha conectado correctamente, pero no está enviando ninguna imagen:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> La clave está en una cámara cuyo canal de *control* funciona correctamente —la detección funciona, la configuración y las escrituras de `[chunk-enable …]` se realizan con éxito— mientras que *todos* los fotogramas agotan el tiempo de espera.
>
> **La causa habitual es que la cámara está activada para un disparador de hardware.** Con los códigos `trigger_mode="On"` y `trigger_source="Line2"`, la cámara no emite nada en absoluto hasta que llega un flanco eléctrico al cable de sincronización M8. Si no hay ningún cable que transmita esa señal, cada captura espera indefinidamente. La cámara no está averiada y la red funciona correctamente : está haciendo exactamente lo que se le ha indicado.
>
> `CameraSettings()` y los presets `default` / `high_speed` / `high_quality` activan el modo de funcionamiento libre, y una captura que agota el tiempo de espera mientras está activada muestra un mensaje explicativo en lugar de mostrar simplemente «`-1011`». `PRESETS["triggered"]` activa la Línea 2, tal y como está diseñado.
>
> Para forzar a cualquier cámara a funcionar en modo libre:
>
> ```python
> import copy
> settings = copy.deepcopy(PRESETS["high_quality"])   # PRESETS entries are shared
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Si sigue agotándose el tiempo de espera con `trigger_mode="Off"`, la cámara realmente no está enviando datos: envíanos el registro y `ip link show`.

#### Perfiles de color (vista previa en directo RGB) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` selecciona el perfil de color de la pantalla para la **vista previa en directo** en cámaras RGB (las cámaras multiespectrales ignoran este ajuste):

| Perfil | Significado |
| --- | --- |
| `raw` | Omite por completo la cadena radiométrica. |
| `linear` | DSNU + flat + WB, sin CCM, sin gamma. |
| `natural` | Lineal + CCM medido + gamma sRGB, solo con el acabado básico (suavizado de crominancia + desaturación de luces) — el valor predeterminado realista. |
| `enhanced` | `natural` más el acabado completo «hub-parity» (eliminación de franjas, vibrancia, contraste local CLAHE). Aspecto más rico con aproximadamente **el doble del coste de acabado por fotograma**, por lo que la velocidad de fotogramas en directo es menor. |
| `custom_temp` | `natural`, pero con el balance de blancos fijado en `custom_cct_k` Kelvin (se ignora DLS; limitado a 2000–10 000 K en el backend). |

El perfil es un control de velocidad/aspecto **solo para la vista previa en directo**: las capturas guardadas siempre obtienen el acabado completo y rico independientemente del perfil seleccionado, por lo que elegir `natural` para recuperar tiempo de fotograma no reduce la calidad de lo que se guarda en el disco. Un perfil desconocido activa `ValueError`; cuando se puede acceder a un backend de Chloros, el cambio también se envía mediante POST a este para que el siguiente fotograma de vista previa lo refleje (los usuarios de SDK directo sin backend siguen recibiendo la modificación de la configuración).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Aspectos destacados (pantalla RGB) — `set_highlights`

`LatticeCamera.set_highlights(value)` reduce los píxeles más brillantes y más blancos de la **pantalla** en las cámaras RGB: la solución para que una superficie blanca brillante no se muestre como una mancha blanca plana. `value` es `-100`..`0` (limitado); `0` está desactivado, y el resultado es exactamente el mismo que sin este ajuste. Los ajustes más intensos reducen el blanco de la pantalla (hasta aproximadamente el 60 % en `-100`) y amplían la curva de tonos, por lo que una superficie que la vista previa automática había convertido en blanco puro recupera sus detalles.

Se trata de un control de visualización, pero a diferencia de la saturación y el contraste (que solo modifican la vista previa en directo), también cambia lo que se guarda: tanto la vista previa en directo como las capturas a **nivel de pantalla** lo respetan, por lo que esos archivos también salen más oscuros (llevan el valor `Chloros:DisplayHighlights`); las exportaciones de radiancia y reflectancia nunca lo hacen, y no puede recuperar los datos del sensor que se han recortado; para ello, hay que reducir la exposición. Las cámaras multiespectrales y monocromáticas lo ignoran. El valor se almacena en `settings.highlights` y se envía mediante POST a un backend de Chloros accesible, exactamente igual que `set_color_profile`; el equivalente en la CLI es `lattice highlights --set N`.

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_highlights(-60)                      # 0 = off, -100 = strongest
```

#### Cámaras monocromáticas (M3M) y `Calibration`

Una cámara monocromática **M3M** (`M3M-<lens>-F<wavelength>`) es de banda única: un plano en escala de grises, sin mosaico de Bayer ni matriz de diafonía espectral 3×3. `Calibration` la reconoce y muestra un indicador `is_mono`. La reflectancia sigue aplicándose como un mapa radiométrico por banda (la desmezcla es la matriz identidad), pero los cálculos multibanda en una sola cámara dan resultados válidos en lugar de devolver valores sin sentido:

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

Para crear un índice de vegetación a partir de hardware monocromático, combina varias cámaras M3M a diferentes longitudes de onda en una pila multibanda alineada (véase [Alineación de matrices](#array-alignment)) y calcula el índice sobre esa pila en lugar de hacerlo en una sola cámara.

#### Magnitudes de banda del DAQ-M — `DLS.band_magnitudes`

`DLS(mac_address, filter_model="RGN-IMX265")` se conecta a un DAQ-M a través de Bluetooth mediante su propia dirección MAC BLE. Es la única clase de sensor de luz que se comunica directamente con el sensor en lugar de hacerlo a través del backend, por lo que necesita lo mismo que las clases de cámara directa (`CAMERA_AVAILABLE`: el entorno de ejecución del SDK de Arena y OpenCV), además de `bleak` para Bluetooth (`pip install "chloros-sdk[camera]"` o `pip install bleak`; sin él, `DLS` genera el error `DLSError`). Un DAQ-M conectado en otro lugar (el pool del backend, un proyecto, la interfaz gráfica de usuario) deja de anunciarse, por lo que hay que liberarlo allí primero. **Se requiere `mac_address`**: no hay un valor por defecto, ya que cada unidad tiene el suyo propio, y `DLS()`, si carece de él, genera `DLSError` antes de que se intente cualquier conexión. Busca la dirección MAC con `discover_daq_sensors(transports=["ble"])` (véase [Detección](#discovery--finding-an-address-to-connect-with)) o `chloros-cli daq pool-discover --only ble`.

`DLS.band_magnitudes(spectrum=None, *, spectral_response=None)` integra un espectro DAQ-M (la última lectura cuando `spectrum` es `None`; `DLSError` si aún no hay ninguno) a través de las respuestas de los tres canales de una cámara y devuelve `{"red": …, "green": …, "nir": …}` — las claves nombran los canales en ese orden; con las curvas de una cámara RGB, el tercero (`nir`) es su canal azul.

- Sin `spectral_response`, utiliza la curva de flota del modelo de filtro (la `filter_model` con la que se construyó el `DLS`), que se midió en una unidad.
- `spectral_response` utiliza en su lugar las curvas medidas **propias** de la cámara: un diccionario `{"wavelengths": [...], "R": [...], "G": [...], "B": [...]}`, tal y como se almacena en el paquete de calibración de dicha cámara (`color_correction.spectral_response_smoothed` — cámaras RGB; los paquetes multiespectrales no incluyen dicha curva). Si no puede utilizar dicho diccionario, recurre a la curva de la flota.

```python
from chloros_sdk import DLS

# camera_curves: {"wavelengths": [...], "R": [...], "G": [...], "B": [...]}
with DLS(mac_address="AA:BB:CC:DD:EE:FF") as dls:
    reading = dls.acquire()
    fleet = dls.band_magnitudes(reading)                         # fleet curve
    own = dls.band_magnitudes(reading,
                              spectral_response=camera_curves)   # the camera's own
```

La función `chloros-cli lattice dls --camera SERIAL` de la CLI hace lo mismo a través del backend, buscando la curva de la cámara en su paquete de calibración cargado o almacenado en caché, e indica qué curva se ha utilizado. A través de HTTP, esto es `POST /api/dls/cli-read` con `mac` (obligatorio: sin él, la respuesta es un error HTTP400, `code` `MAC_REQUIRED`) y `camera_serial` en el cuerpo; el `band_magnitudes_source` de la respuesta es `camera <serial>` cuando se ha utilizado la curva propia de esa cámara, o `model <filter>` para la curva de la flota (una cámara sin curva: no es RGB o no se ha cargado ningún paquete ni almacenado en caché).

---

## Excepciones

Atrapa la clase base para gestionar «cualquier cosa que haya salido mal en Chloros»:

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

> `ChlorosAuthenticationError` y `ChlorosConfigurationError` son atributos de `chloros_sdk` (y de `chloros_sdk.exceptions`), pero no en `__all__`, por lo que `from chloros_sdk import *` no los incluye: impórtalos por su nombre. La familia `ChlorosError` es heredada de `ChlorosLocal` y de las funciones basadas en ella (`process_folder()`, `process_lattice_capture()`); la superficie Smart-Connect hereda de `ChlorosConnectError`.

Jerarquía:

```

ChlorosError
├── ChlorosBackendError           (backend binary not found / failed to start)
├── ChlorosConnectionError        (HTTP transport failure; no backend running
│                                  with auto_start_backend=False)
├── ChlorosLicenseError           (403 PLAN_UPGRADE_REQUIRED: plan below Copper)
├── ChlorosAuthenticationError    (401 AUTH_REQUIRED: not logged in)
├── ChlorosConfigurationError     (bad configure() inputs)
└── ChlorosProcessingError        (pipeline failed; create/import refused)
    └── ChlorosProjectFilesMissingError  (open_project(): none of the
                                  project's source files found, even after
                                  the relink; .missing_roots, .audit)

ChlorosConnectError                (the smart-connect surface: connect_*, attach_array,
                                    list_*, discover_*, every CameraSession /
                                    ArraySession / DAQSensorSession method,
                                    RecorderHandle, ContinuousHandle,
                                    disconnect_all(), and analyze_array_network()
                                    when no backend answers; 401 and 403 included.
                                    Derives from plain Exception, NOT from
                                    ChlorosError, so `except ChlorosError` will
                                    not catch it. .warning: a warning the
                                    backend sent beside its error (also at the
                                    end of the message); .recording_closed: the
                                    recording a failed DAQ-E restart closed;
                                    both None otherwise)

FrameFetchError (RuntimeError)     (grab() / frame_stream() refusals; .status,
                                    .body, .errors)
ValueError                         (capture() arguments: PNG/JPEG with a level they
                                    cannot hold; a bad packed=)
FileNotFoundError / ImportError    (open_project(): no such folder / no project
                                    loaders)
RuntimeError                       (CameraHandle / ArrayHandle: a backend refusal)

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

## Ejemplos de extremo a extremo

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
        indices=["NDVI", "GNDVI", "OSAVI", "SAVI"],
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

### 3. Campaña de captura orientada a proyectos

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

### 4. Flujo de fotogramas multicámara → canalización NumPy

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
        width=r["width"], height=r["height"],     # before binning
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Equivalente a una receta de captura (Python puro)

`run_project_recipe()` ejecuta una receta de CLI tal cual (véase [Carpeta del proyecto y recetas](#project-folder--recipes)). Escritos en Python, los mismos pasos son:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply: connect_all() applied each camera's saved settings. CameraHandle
    # has no settings method; a CameraSession on the same pooled camera does
    # (it is borrowed, so it never releases the camera)
    chloros_sdk.connect_camera(cam.serial).set_settings(
        exposure_time=5000, gain=6.0)

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

## Servidor MCP (asistentes de IA)

`chloros-mcp` permite que un asistente de IA que utilice el [Protocolo de contexto del modelo](https://modelcontextprotocol.io) (MCP), como Claude Desktop, Claude Code o Cursor, controle Chloros: procesar imágenes (crear un proyecto, importar una carpeta de capturas, seleccionar ajustes, procesar y generar informes sobre los resultados exportados) y manejar cámaras LATTICE, matrices y sensores de luz DAQ (conectar, previsualizar, ajustar la exposición y el color, capturar, grabar y actualizar el firmware). Sus herramientas abarcan las funciones de este SDK. Tú pides en lenguaje natural y el asistente invoca las herramientas del servidor:

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

The assistant calls:
  create_project          {"project_name": "Flight_2026-10-17"}
  import_images           {"folder_path": "D:\\Flights\\2026-10-17"}
  configure_processing    {"indices": ["NDVI"]}
  start_processing        {}
  get_processing_status   {}      (every 30-60 s until its state is "finished")

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.

The assistant calls:
  list_devices            {}
  get_camera_preview      {"array_id": "array-1779862544497"}
  start_capture_run       {"array_id": "array-1779862544497", "mode": "interval",
                           "interval_s": 10, "duration_s": 300,
                           "processing": "reflectance"}
  get_job_status          {"job_id": "..."}   (until its state is "finished")
```

El servidor se basa en este SDK: las herramientas de procesamiento invocan [`ChlorosLocal`](#image-processing--chloroslocal) y las herramientas de hardware las [llamadas de smart-connect](#smart-connect-for-lattice-cameras), [sesiones DAQ](#daq-sensor-sessions), `ChlorosProject` y las [utilidades de cámara](#camera-utilities), todas ellas comunicándose con el mismo backend que utiliza la aplicación Chloros. Sus solicitudes llevan el identificador `X-Chloros-Client: sdk`, por lo que el backend aplica las comprobaciones de inicio de sesión y de plan del SDK, y las reglas del SDK se aplican automáticamente (por ejemplo, se rechaza «Texture Aware» para las capturas de LATTICE).

### Instalación

El servidor se incluye en el paquete `chloros-sdk`; no hay que instalar nada más. El SDK instala un comando `chloros-mcp`, que es el mismo que `python -m chloros_sdk.mcp_server`:

```bash
pip install chloros-sdk        # not needed where the Chloros installer set up the SDK (Step 2)
chloros-mcp --help
```

- **Windows:** el instalador de Chloros instala el SDK mediante pip en tu Python, lo que coloca `chloros-mcp.exe` en la carpeta `Scripts` de ese Python.
- **Linux (.deb):** el paquete instala el SDK para el sistema `python3` y coloca el comando `chloros-mcp` en `/usr/bin`, de modo que está en el `PATH`. En un virtualenv, `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` añade `chloros-mcp` al `bin` del virtualenv.

El servidor utiliza únicamente la biblioteca estándar sobre el SDK (Python 3.7+). Un cliente MCP lo inicia y se comunica con él a través de stdin/stdout; no es necesario ejecutarlo manualmente, salvo con `--help`.

### Configuración del cliente

**Claude Desktop:**añade el servidor a `claude_desktop_config.json` (en Windows, `%APPDATA%\Claude\claude_desktop_config.json`; se abre desde**Configuración > Desarrollador > Editar configuración**) y, a continuación, reinicia Claude Desktop:

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

**Claude Code:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Cursor:** la misma entrada `mcpServers` que en Claude Desktop, en `~/.cursor/mcp.json` (en todos los proyectos; `%USERPROFILE%\.cursor\mcp.json` en Windows) o `.cursor/mcp.json` en un proyecto.

Las opciones se introducen en `args`, o después del comando para Claude Code (`claude mcp add chloros -- chloros-mcp --no-auto-start`). Para utilizar un SDK de Python concreto en lugar del `chloros-mcp` en el `PATH`, ejecuta el módulo con ese Python (su ruta completa como `command` si no es el primer `python3` en el `PATH`):

```json
{
  "mcpServers": {
    "chloros": {"command": "python3", "args": ["-m", "chloros_sdk.mcp_server"]}
  }
}
```

Un cliente inicia el servidor sin tu shell, por lo que, si informa de que el servidor no se ha podido iniciar, es probable que `chloros-mcp` no se encuentre en el `PATH` que ve el cliente. Indica la ruta completa en su lugar (`where chloros-mcp` en Windows, `which chloros-mcp` en Linux, donde se encuentra el archivo .deb es `/usr/bin/chloros-mcp`), o ejecuta el módulo con la versión de Python que incluya el SDK.

### Opciones

| Opción | Valor predeterminado | Qué hace |
| --- | --- | --- |
| `--backend-url URL` | `http://127.0.0.1:5000` | El backend de Chloros que se va a utilizar. |
| `--no-auto-start` | desactivado | Nunca iniciar un backend: utilizar uno que ya esté en ejecución, como el de la aplicación Chloros, el servicio de backend de Linux o un túnel hacia la (véase [Modo de backend remoto](#remote-backend-mode-pip-only-host-via-tunnel); en ese caso, las rutas de las carpetas deben hacer referencia a la misma carpeta en ambas máquinas). |
| `--backend-exe PATH` | donde Chloros lo instala | El backend que se iniciará cuando no haya ninguno en ejecución, para un Chloros instalado en una ubicación que el SDK no detecta. |
| `--log-level LEVEL` | `WARNING` | `DEBUG`, `INFO`, `WARNING` o `ERROR`. Los registros se envían a stderr, que la mayoría de los clientes muestran en su registro MCP; stdout solo transporta mensajes de protocolo. |

### Herramientas

Los argumentos de cada herramienta son los de la llamada al SDK que envuelve; el cliente enumera cada uno de ellos con su esquema completo. Las rutas son absolutas.

**Procesamiento**

| Herramienta | Llamada al SDK | Qué hace |
| --- | --- | --- |
| `create_project` | `ChlorosLocal.create_project` | Crea un proyecto y lo establece como el proyecto abierto, sobre el que actúan las herramientas de procesamiento. Un nombre sin más crea la carpeta en la carpeta de proyectos predeterminada; una ruta absoluta la crea allí. |
| `open_project` | `ChlorosLocal.open_project` | Abre un proyecto guardado y lo convierte en el proyecto abierto. Las rutas de un proyecto movido se redirigen a la carpeta en la que se encuentra ahora; `relink_folder` (absoluto) indica la carpeta en la que se encuentran ahora los demás archivos que faltan. Devuelve `source_files` y `image_count`. No se ha encontrado nada: un mensaje de error que indica dónde estaban y que se denomina `relink_folder`; no hay nada abierto. |
| `import_images` | `import_images` | Importa los pares `.RAW`+`.JPG` de Survey3, y las capturas de LATTICE `.tif` capturas y DAQ `.daq` grabaciones; devuelve `count` (imágenes) y `scan_count` (grabaciones). |
| `configure_processing` | `configure` | Modifica los parámetros pasados y ningún otro (valores: [Valores admitidos](#supported-values)); `custom_settings` envía toda la configuración tal y como se ha especificado. |
| `get_project_settings` | `get_config` | Lee los ajustes guardados de procesamiento, exportación e índice del proyecto abierto. |
| `start_processing` | `process(wait=False)` | Inicia el procesamiento del proyecto abierto y vuelve inmediatamente. |
| `get_processing_status` | `get_status` | Informa de `state` (`idle`, `starting`, `processing` o `finished`) y el progreso; una vez finalizada, se registran los códigos `summary` y `export_check_failure` cuando no se han podido verificar las exportaciones, y `image_failure` cuando se han perdido imágenes; el resto de productos se registran; se informa de que la ejecución ha fallado. |
| `process_folder` | `process_folder` / `process_lattice_capture` | Crea un proyecto para una carpeta, lo importa, lo configura e inicia la ejecución en una sola llamada (`lattice: true` para capturas de LATTICE). Mientras una ejecución está en curso, rechaza cualquier acción antes de crear nada, y el backend también rechaza entonces cualquier cambio de proyecto desde cualquier herramienta. |
| `export_light_sensor` | `export_light_sensor` | Escribe los archivos calibrados `.daq` y `.csv` para las grabaciones del sensor de luz del proyecto sin procesar las imágenes, obteniendo la calibración de fábrica de MAPIR Cloud cuando no está almacenada en caché (véase [Registros del sensor de luz](#light-sensor-recordings--calibrated-daq--csv)). |
| `read_image_audit_tags` | `read_image_audit_tags` | Lee los metadatos de auditoría de Chloros de una imagen procesada (fuente, calibración, etapas aplicadas). |

**Cámaras y matrices**

| Herramienta | Llamada al SDK | Qué hace |
| --- | --- | --- |
| `list_devices` | `list_cameras`, `list_arrays`, `list_daq_sensors` | Lo que está conectado actualmente, en cualquier cliente, cada uno con `connected_here`; además de los trabajos en ejecución del servidor. |
| `discover_devices` | `discover_lattice_cameras`, `discover_daq_sensors` | Cámaras y sensores de luz que podrían estar conectados. |
| `get_device_status` | `CameraSession.status`, `ArraySession.status` / `capability`, `DAQSensorSession.status` | Estado en tiempo real de una cámara, una matriz (con su capacidad y estado de funcionamiento) o un sensor de luz. |
| `analyze_array_network` | `analyze_array_network` | Lo que este host y esta red pueden soportar para una matriz propuesta. |
| `connect_camera` / `connect_array` | `connect_camera` / `connect_array` | Conectar una cámara o una matriz sincronizada (primero el maestro). |
| `connect_project_devices` | `open_project(...).connect_all` | Conecta todos los dispositivos que recuerda un proyecto guardado, con su configuración. |
| `disconnect_device` | `disconnect` / `force_disconnect` | Desconectar una cámara, una matriz o un sensor de luz (véase «Comportamiento»). `keep_settings: true` en una matriz guarda la configuración de cada cámara en su propia memoria; `clear_settings: true` restablece esa memoria (su resultado lleva `camera_memory`). |
| `disconnect_project_devices`, `align_project_arrays`, `capture_project_devices` | `disconnect_all`, `align_arrays`, `capture_all` | Los dispositivos del proyecto, en conjunto. |
| `read_camera_settings` / `set_camera_settings` | `CameraSession.read_nodes` / `set_settings` | Leer o escribir la configuración de la cámara, por nombre descriptivo o nombre de nodo. |
| `set_array_frame_rate` | `ArraySession.set_target_fps` | Fijar la velocidad de una matriz o restablecerla a la velocidad automática. |
| `capture_camera` / `capture_array` | `CameraSession.capture` / `ArraySession.capture` | Una captura (`fastest` y `wait_for_exposure` son `capture_fastest` y la captura con exposición ajustada). Si no hay `output_dir`, se guarda en la carpeta abierta del proyectocarpeta `captures` del proyecto abierto, y los archivos se incorporan al proyecto (se retienen hasta que finalice una ejecución de procesamiento); así lo indica la respuesta `project_add`. |
| `start_capture_run` | `ArraySession.continuous`, `capture_repeated` | Capturas continuas o a intervalos, como un trabajo. |
| `start_array_recording` | `ArraySession.record` / `burst` | Un vídeo con índice combinado o una ráfaga sin procesar, como tarea. |
| `build_burst_video` | `ArraySession.build_video` | Reprocesar una ráfaga para convertirla en vídeo calibrado, como tarea. |
| `get_job_status` / `stop_job` | | Seguir o detener un trabajo. |
| `get_camera_preview` | | Vista previa en directo de cada cámara, devuelta como una imagen. |
| `grab_frames` | `ArraySession.grab`, `CameraHandle.grab` | Un fotograma (o un conjunto sincronizado) en un nivel de procesamiento, sin guardar: estadísticas por plano y un archivo PNG de cada uno. |
| `calibrate_array_alignment`, `get_array_alignment`, `clear_array_alignment`, `nudge_array_alignment`, `export_array_alignment`, `import_array_alignment` | `ArrayHandle.calibrate_alignment`, `alignment_status`, `clear_alignment`, `tweak_alignment`, `export_alignment`, `import_alignment` | Alineación de un array. |

**Sensores de luz**

| Herramienta | Llamada al SDK | Qué hace |
| --- | --- | --- |
| `connect_light_sensor` | `connect_daq_sensor` | Conecta un DAQ-U, DAQ-M o DAQ-E (`cap_id` establece su corrección de límite). |
| `get_light_sensor_reading` | `DAQSensorSession.latest` | Las últimas lecturas: marca de tiempo, saturación, rango y pico del espectro, y el espectro promediado en intervalos de longitud de onda (`full_spectrum` para cada punto). |
| `set_light_sensor_streaming` | `stream_start` / `stream_stop` | Iniciar o detener la transmisión. |
| `start_light_sensor_recording` / `stop_light_sensor_recording` | `record_start` / `record_stop` | Graba un archivo `.daq` como un trabajo; `duration_s` lo detiene automáticamente. |
| `set_light_sensor_cap` | `DAQSensorSession.set_cap` | La corrección de la tapa (`none` para un sensor sin cubierta). |
| `get_light_sensor_bindings` / `set_light_sensor_bindings` | `daq_bindings` / `set_daq_bindings` | Qué sensor proporciona la reflectancia de cada cámara y matriz. |
| `get_light_sensor_band_presets` / `get_light_sensor_reflectance` | `daq_band_presets` / `light_sensor_reflectance` | Los conjuntos de bandas, así como la reflectancia y los índices ambientales y del objeto de dos sensores de transmisión continua. |
| `get_light_sensor_device_info`, `set_light_sensor_network`, `set_light_sensor_auto_stream`, `enable_light_sensor_ptp`, `reboot_light_sensor`, `update_light_sensor_firmware` | `device_info`, `set_network`, `set_auto_stream`, `enable_ptp`, `reboot`, `update_firmware` | Ajustes propios de un DAQ-E, guardados en la unidad. |

**Utilidades de la cámara** ([Utilidades de la cámara](#camera-utilities))

| Herramienta | Llamada al SDK | Qué hace |
| --- | --- | --- |
| `get_camera_color_settings` / `set_camera_color_settings` | `camera_color_profile`, `camera_color_adjust`, `camera_highlights` y sus funciones de configuración | Color de visualización de una cámara RGB: perfil y temperatura de color, saturación/contraste, luces altas. |
| `run_camera_white_balance` / `set_camera_white_balance` | `camera_white_balance_once` / `set_camera_white_balance` | Balance de blancos de un solo disparo, o automático / desactivado / relaciones fijas. |
| `list_camera_presets` / `apply_camera_preset` | `camera_presets` / `apply_camera_preset` | Preajustes de la cámara. |
| `list_filter_models` | `camera_filter_models`, `camera_filter_model_info` | Los modelos de filtro, o uno por su nombre. |
| `get_camera_firmware_status` / `update_camera_firmware` | `camera_firmware_status` / `update_camera_firmware` | Versiones de firmware y un flash (una tarea por defecto). |
| `get_camera_power_info` / `set_camera_power` | `camera_power_probe`, `sleep_camera`, `wake_camera` | Estado de alimentación y térmico; en reposo o activo. |
| `calibrate_camera_with_target` | `calibrate_camera_with_target` | Coeficientes de un objetivo de calibración en la vista. |
| `get_network_adapters` / `fix_network_settings` | `network_adapters` / `fix_network_settings` | Los adaptadores de red de la cámara, una estimación y la solución (solo para administradores). |
| `apply_vignette_correction` | `apply_vignette_correction` | Corregir el viñeteado de una carpeta de imágenes. |
| `read_dls` | `read_dls` | Una lectura DLS. |
| `probe_system` / `get_system_info` | `probe_system` / `system_info` | El hardware y el tiempo de ejecución del host. |

**Cámaras transitorias, perfiles de alineación e índices** ([Matrices transitorias, perfiles de alineación e índices](#matrices-transitorias-perfiles-de-alineación--índices)). Todas las herramientas de esta sección que abren cámaras utilizan `keep_settings` / `clear_settings` (por defecto, false): véase [Memoria de la cámara](#camera-memory--keep_settings--clear_settings).

| Herramienta | Llamada al SDK | Qué hace |
| --- | --- | --- |
| `start_camera_continuous` / `start_multi_camera_stream` | `start_camera_continuous` / `start_multi_camera_stream` | Graba una cámara o transmite varias de forma sincronizada, como una tarea (`duration_s` lo detiene automáticamente). |
| `capture_cameras_transient` | `capture_cameras_transient` | Una captura sincronizada sin conectar una matriz. |
| `test_array_sync` / `detect_array_master` | `test_array_sync` / `detect_array_master` | La comprobación de la sincronización del disparador y qué cámaras están en la línea de sincronización. |
| `analyze_array_models` | `analyze_array_models` | Qué rendimiento obtendría una matriz de estos modelos en esta red. |
| `calibrate_alignment_profile` | `calibrate_alignment_profile` | Resolver y guardar un archivo de perfil de alineación (`options` contiene las opciones más avanzadas del solucionador). |
| `capture_aligned_multiband` / `start_aligned_stream` | `capture_aligned_multiband` / `start_aligned_stream` | Una imagen multibanda alineada, o una secuencia de ellas como tarea. |
| `read_alignment_profile` / `reorder_alignment_profile` | `alignment_profile_info` / `reorder_alignment_profile` | El contenido de un archivo de perfil: el orden de las bandas y las cámaras activadas. |
| `list_index_presets`, `compute_index`, `capture_live_index` | `index_catalog`, `compute_index`, `capture_live_index` | Preajustes de índice y degradados; un índice sobre un archivo TIFF multibanda; captura e índice en un solo paso. |

**Sincronización horaria, carpeta de proyectos y recetas** ([PTP Grandmaster](#ptp-grandmaster--time_sync_status), [Carpeta de proyectos y recetas](#project-folder--recipes))

| Herramienta | Llamada al SDK | Qué hace |
| --- | --- | --- |
| `get_time_sync_status` | `time_sync_status`, `time_sync_peers`, `time_sync_cameras` | El estado del «grandmaster» PTP, sus pares y el estado PTP de cada cámara de matriz. |
| `restart_time_sync` / `set_time_sync_priority` | `restart_time_sync` / `set_time_sync_priority` | Reinícialo; configura sus prioridades. |
| `get_project_folder` / `set_project_folder` / `reset_project_folder` | `project_folder` / `set_project_folder` / `reset_project_folder` | La carpeta en la que se crean los nuevos proyectos. |
| `run_project_recipe` | `run_project_recipe` | Ejecuta la receta de un proyecto guardado como una tarea con su registro. Se comprueba primero: se rechaza una receta que contenga un error y no se ejecuta nada; se emite una advertencia sobre matrices o sensores no conectados (no los conecta: primero se ejecuta `connect_project_devices`). `dry_run` devuelve el resultado de la comprobación y no ejecuta nada. |

No hay ninguna herramienta para abrir un proyecto guardado para su procesamiento: cada trabajo de procesamiento comienza con `create_project` (o `process_folder`).

### Requisitos

- **Chloros instalado en el mismo ordenador.** El backend de Chloros se encarga del procesamiento. El servidor también puede utilizar un backend al que se acceda a través de un túnel (`--backend-url` con `--no-auto-start`; véase [Opciones](#options)).
- **Haber iniciado sesión en ese ordenador**, en la aplicación de Chloros o mediante `chloros-cli login`. El servidor no puede iniciar sesión por ti: si se ejecuta una herramienta mientras no has iniciado sesión, se muestra el mensaje «No has iniciado sesión en Chloros» junto con el motivo indicado por el backend, y el asistente te lo indica.
- **Cualquier plan de pago de Chloros+ (Copper o superior).** El servidor requiere acceso al SDK, y el plan gratuito Iron no lo tiene: una herramienta que se ejecute en el plan gratuito responderá «No disponible en este plan de Chloros» indicando el motivo del backend.
- Python 3.7 o posterior con el SDK, y un cliente MCP que inicie servidores locales (stdio).

### Funcionamiento

- **El backend.** Al iniciar el servidor o mostrar la lista de sus herramientas no se inicia nada; la primera llamada a una herramienta establece la conexión. Si la aplicación Chloros (o el servicio de backend de Linux) está en ejecución, el servidor utiliza su backend y lo deja en ejecución. Si no hay ningún backend en ejecución, el servidor inicia el que está instalado, sin ventana. Cuando el servidor se cierra (al salir del cliente o al reiniciar el servidor), detiene los trabajos de captura y las grabaciones del sensor de luz que había iniciado y libera los dispositivos a los que se había conectado; el backend sigue funcionando, de modo que cualquier proceso de procesamiento inconcluso continúa hasta su final, y se detiene automáticamente tras 30 minutos sin que nada lo esté utilizando. Con `--no-auto-start`, el servidor nunca inicia un backend: si se invoca una herramienta sin que haya ninguno en ejecución, se te pedirá que abras la aplicación Chloros, y la siguiente llamada, una vez que lo hayas hecho, establecerá la conexión.
- **Un proyecto abierto.** El backend tiene un proyecto abierto, compartido con la aplicación Chloros cuando esta está abierta, por lo que no debes trabajar en otro proyecto en la aplicación mientras el asistente está en funcionamiento.
- **Solo cambian los ajustes que se le pasan.** `configure_processing` deja cada parámetro que no se le haya proporcionado en su valor guardado (el SDK `configure()` restablece el debayer, la corrección de viñeteado, la calibración de reflectancia, el PPK y el formato de exportación a sus valores predeterminados cuando no se pasan; el servidor los rellena a partir del proyecto). Si no puede leer los ajustes guardados del proyecto, no cambia nada y lo indica, a menos que se especifiquen los cinco. `indices` sustituye la lista de índices del proyecto. El resultado enumera lo que se envió como `settings_sent`, y denomina en `warnings` cualquier índice que las bandas de la cámara no puedan generar.
- **El procesamiento se realiza según el método «iniciar y sondear».** `start_processing` devuelve un resultado tan pronto como comienza la ejecución, que luego tarda entre minutos y horas, y el asistente lo sigue con `get_processing_status`. `state` es `idle` (sin procesamiento), `starting` (el backend aún no ha informado de la ejecución, durante un máximo de 20 s), `processing` o `finished`, con el [resumen](#post-run-summary--hints): lo que se ha exportado, y indicaciones cuando se ha exportado menos de lo esperado (sin imágenes, no se ha encontrado el objetivo de calibración). Los productos se guardan en la carpeta del proyecto, tal y como se hace desde la aplicación. `start_processing` se rechaza mientras una ejecución está en curso, y un backend que deja de responder se notifica como un error, no como una ejecución finalizada.
- **Lo que ve el asistente.** Las advertencias del SDK aparecen en el resultado de la herramienta bajo `warnings`, y los fallos (cese de sesión, plan, ruta incorrecta) como texto de error que puede leer y transmitirte. Una lista con más de 20 elementos se recorta a los primeros 20, y su longitud completa aparece como `<key>_total`. Las rutas deben ser absolutas.
- **Los dispositivos se comparten.** Las cámaras, matrices y sensores de luz residen en el backend, por lo que el asistente puede controlar uno de ellos a través de la aplicación Chloros, y cualquier cambio que realice se muestra en la aplicación. `disconnect_device` muestra lo que el servidor ha conectado; un dispositivo al que se ha conectado otro cliente (la aplicación, `chloros-cli`, un script u otro `chloros-mcp`) solo se libera mediante `force: true`, que es el `force_disconnect` del SDK, y el nombre de quien lo conectó se indica cuando el backend lo sabe; un dispositivo que el backend no tiene registrado en absoluto se indica como no conectado (`released: false`, `connected: false`, nada que liberar). Una grabación o transmisión que ejecuta el backend lleva su vista `get_job_status` con `counts` — `frames_written`, `frames_dropped`, `frames_failed`, `drop_reason` (grupos de fotogramas para una transmisión multicámara) — y una línea de `summary` («220 fotogramas grabados, 55 descartados, 0 fallidos: ...»), mientras se ejecuta y, una vez finalizada, sus cifras finales; la lista de otro cliente sobre ello (`other_clients_jobs`) muestra lo mismo (`list_cameras()`, `list_arrays()` y `list_daq_sensors()` muestran los datos de cada dispositivo`connected_by`: `client`, `label`, `name`, `at`). Cada `chloros-mcp` se identifica ante el backend como `chloros-mcp (pid N)` (`chloros_sdk.connect.CLIENT_LABEL`, enviado como `X-Chloros-Client-Label`; cualquier script de SDK puede configurarlo). Detener la grabación del sensor de luz de otro cliente también requiere `force`.
- **El trabajo prolongado es una tarea.** `start_capture_run`, `start_array_recording`, `build_burst_video`, `start_light_sensor_recording`, `start_camera_continuous`, `start_multi_camera_stream`, `start_aligned_stream`, `run_project_recipe`, y cualquier captura, resolución o índice solicitado (`background`) devuelve un `job_id` de inmediato; a continuación viene el `get_job_status` y el `stop_job` pone fin . Una ejecución continúa hasta que se detiene, a menos que se le asigne `max_captures`, `duration_s` (o, para una ráfaga, `max_frames`); `min_free_gb` se niega a iniciarse cuando la unidad tiene poco espacio libre. No se aplica ningún límite a menos que se establezca uno. Una grabación o transmisión que ejecute el backend (`start_camera_continuous`, `start_multi_camera_stream`, `start_aligned_stream`) conserva el ID de tarea del backend, por lo que otro `chloros-mcp` —iniciado posteriormente— lo localiza mediante ese ID: `get_job_status` lo sigue (y enumera trabajos como `other_clients_jobs`, a partir de `GET /api/camera/cli-jobs`, cada uno con `started_by`), y `stop_job` solo lo detiene con `force: true`. Cualquier otro trabajo (una ejecución de captura, una grabación en matriz o ráfaga, una captura en segundo plano) se ejecuta en el `chloros-mcp` que lo inició, y solo ese servidor lo sabe: otro que lo haya solicitado así lo indica. Uno detenido lee `stopping` mientras vacía su cola, luego `finished` con sus recuentos finales, durante 10 minutos y desde cualquier `chloros-mcp`: el que lo inició detecta la parada forzada de otro de la misma manera. `get_job_status` nunca espera a que se produzca una parada: durante una (la de su propio `stop_job`, la de otro clienteo la parada que libera las cámaras de un trabajo que ha finalizado por sí mismo), responde de inmediato con `stopping` y los recuentos en tiempo real, y `stop_job` de un trabajo que ya se está deteniendo lo indica. Un trabajo que ha finalizado indica el motivo en `stop_reason`: su propio motivo (`no_frames`, `disk_full`, `member_missing`, o, en el caso de una ejecución continua en matriz, `count`, `duration`, `array_gone`), `limit reached` solo cuando sus recuentos indican su `count` (o el `max_frames` de una ráfaga, el de una grabación`duration_s` de una grabación) se alcanza, `error`, `stopped` (con `stopped_by`) cuando un cliente lo detiene; en caso contrario, `finished`.
- **Dónde se guardan las capturas.** A la carpeta `captures` del proyecto abierto, donde la aplicación las guarda, a menos que se especifique `output_dir`.
- **Imágenes.** `get_camera_preview` y `grab_frames` devuelven imágenes que el asistente puede visualizar, además de su JSON. Las imágenes de `grab_frames` tienen el contraste ajustado para su visualización; sus estadísticas son los valores reales.
- **Cuando el servidor se cierra**, los trabajos que había iniciado se detienen y se cierra cualquier grabación del sensor de luz que hubiera iniciado. Si el servidor había iniciado el backend, también libera los dispositivos a los que se había conectado antes de detener el backend; en el backend de la aplicación, estos permanecen conectados.
- **Protocolo.** Solo stdio (sin punto final HTTP), solo herramientas (sin recursos ni mensajes de solicitud), resultados de texto e imagen, versiones del protocolo MCP del 05-11-2024 al 25-11-2025.**Lo que no se incluye.** Iniciar o cerrar sesión (la cuenta es tuya; el servidor no puede iniciar sesión) y detener un backend que el servidor no haya iniciado. Para todo lo demás que este SDK hace con el procesamiento y el hardware existe una herramienta. (El aprovisionamiento de un DAQ-E —su nombre, token de autenticación, contraseña OTA, requisito de firma, restablecimiento de fábrica— y los servidores DAQ directos son exclusivos de `chloros-cli`, también incluidos en el SDK: un nombre o token incorrectos bloquean todas las instalaciones de Chloros en la unidad.)

---

## Inicio automático del backend

Los puntos de entrada de Smart-Connect —`connect_camera`, `connect_array`, `connect_daq_sensor`, `discover_lattice_cameras` y `discover_daq_sensors` — son clientes HTTP ligeros que dan por hecho que un backend está a la escucha en `127.0.0.1:5000` (la URL predeterminada de la interfaz Smart-Connect). Cuando la interfaz gráfica de usuario (GUI) o la interfaz de línea de comandos (CLI) ya están en ejecución, uno de ellos ya lo está. Desde un script sin más, puede que no haya ninguno, por lo que estas funciones **inician automáticamente el binario del backend incluido** (sin ventana, del mismo modo que lo hace `ChlorosLocal`) antes de su primera llamada y, a continuación, esperan hasta 60 s a que se inicie. `list_cameras()`, `list_arrays()`, `list_daq_sensors()`, `attach_array()` y `analyze_array_network()` nunca inician ninguno.

Reglas:

- **Solo se genera una URL local.** Un `backend_url` que apunte a `localhost` / `127.0.0.1` / `[::1]` es válida; cualquier otro host se considera que pertenece a otra persona y nunca se inicia.
- **Si un backend se cierra antes de responder, se pone fin a la espera.** La llamada devuelve inmediatamente el código de salida, el comando que se ejecutó y las últimas líneas que imprimió el backend, en lugar de esperar los 60 s. El código de salida 98 es la excepción: otro backend ya ocupa el puerto o el archivo de bloqueo (el de la aplicación Chloros, el servicio backend u otro script que aún se esté iniciando), por lo que la llamada sigue esperando a que ese responda. `ChlorosLocal` hace lo mismo.
- **El backend se deja en ejecución para su reutilización** (igual que en la CLI): no se produce un apagado implícito cuando el script finaliza. Al volver a ejecutar el script, se reutiliza el backend activo.
- **Desactiva esta opción con `auto_start_backend=False`** en cualquiera de esas llamadas (por ejemplo, cuando hayas apuntado a un backend remoto o gestiones tú mismo el ciclo de vida del backend).

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

Si no se puede localizar o iniciar el binario incluido, la llamada HTTP posterior genera un `ChlorosConnectError` procesable y **adaptado a la plataforma**, en lugar de un simple rastro de conexión rechazada : en Windows, te remite a la aplicación de escritorio o a un comando `chloros-cli`; en Linux (sin interfaz gráfica), te remite a un comando `chloros-cli` o al `.deb`.

---

## Entorno y encabezados

El SDK marca cada llamada HTTP al backend con `X-Chloros-Client: sdk`. El backend aplica las reglas de licencia del SDK/CLI (se requiere iniciar sesión **y** un plan de pago de Chloros+) en lugar de la vía gratuita de la interfaz gráfica de usuario. Esto se configura automáticamente en el momento de la importación; no es necesario que hagas nada.

El encabezado se añade a cada llamada `requests` / `urllib` cuya URL contenga `localhost` o `127.0.0.1`, en cualquier puerto —incluidos tus propios servicios locales— — y sustituye un valor que hayas establecido tú mismo. Las llamadas a otros hosts no se ven afectadas.

Para anular la URL del backend, pasa `backend_url=` (`ChlorosLocal` admite `backend_url=` o `api_url=`; `process_folder()` y `process_lattice_capture()` solo admiten `api_url=`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(Un backend instalado solo escucha en el bucle de retorno, por lo que un `backend_url` de la LAN no puede acceder a él; véase [Modo de backend remoto](#remote-backend-mode-pip-only-host-via-tunnel) para conocer el patrón de túnel.)

---

## Versiones y compatibilidad

- La versión del SDK se indica como `chloros_sdk.__version__`.
- El SDK vincula el comportamiento a la versión del backend incluido. Combinar un SDK más antiguo con un backend más reciente suele funcionar (puntos finales compatibles con versiones posteriores), pero combinar un SDK más reciente con un backend más antiguo puede provocar errores `404` en los nuevos puntos finales; actualiza la aplicación de escritorio para que coincida.
- La interfaz de Smart Connect (`connect_camera` / `connect_array` / `connect_daq_sensor`) y el punto final de análisis de red devuelven esquemas JSON estables; los nuevos campos son acumulativos.

---

## Consejos para la resolución de problemas

- **`ChlorosAuthenticationError: Login required`** (procedente de `ChlorosLocal`; las llamadas de Smart Connect generan el error `ChlorosConnectError: Login required. …` por la misma causa) → Ejecuta `chloros-cli login EMAIL PASSWORD` una vez en este equipo o inicia sesión a través de la aplicación de escritorio de Chloros.
- **`ChlorosProcessingError: Failed to start processing: … The project folder is too deep for this computer …`** (o `Failed to create project: …` derivado de `create_project()`) → Las rutas largas de Windows están desactivadas en este equipo (configuración predeterminada de Windows), por lo que ninguna ruta puede superar los 260 caracteres, y los productos se sitúan hasta unos 130 caracteres por debajo de la carpeta del proyecto. No se ha procesado nada. El mensaje indica la carpeta de proyecto más larga que cabe: utiliza una más corta o activa las rutas largas de Windows y reinicia Chloros. El instalador de Windows ofrece esta opción como «Habilitar rutas largas de Windows», marcada por defecto; de lo contrario, utiliza la Política de grupo: *Configuración del equipo > Plantillas administrativas > Sistema > Sistema de archivos > Habilitar rutas largas de Win32*.
- **`ChlorosConnectError: No Chloros backend is running …`** → La función Smart-Connect activa automáticamente un backend local, por lo que este error solo aparece cuando no se encuentra o no se puede iniciar el binario incluido (por ejemplo, un host que solo ejecuta pipsin paquete de escritorio). El mensaje varía según la plataforma: en Windows, abre la aplicación de escritorio o ejecuta cualquier comando `chloros-cli`; en Linux, ejecuta un comando `chloros-cli` (no existe interfaz gráfica) o instala el `.deb`. Para un backend remoto, pasa `backend_url=` (y `auto_start_backend=False`).
- **`CAMERA_AVAILABLE == False`** al importar → `lattice_sdk` no se ha podido cargar: el tiempo de ejecución del SDK de Arena, o OpenCV (`cv2`), no se puede importar en este Python. La superficie controlada por el backend (`connect_*`, que se proyecta a través de un backend, `ChlorosLocal`) sigue funcionando.
- **Array se conecta con el sensor completo pero a una baja frecuencia de fotogramas**→ Por diseño: el modo automático mantiene el sensor completo y deja que la frecuencia baje hasta lo que admite la conexión. Pasa `binning=2` / `4` o un `width` / `height` más pequeño para obtener más fotogramas por segundo, o habilita los fotogramas jumbo / actualiza el enlace. El binning y el ROI**no** cubren la sobresuscripción agregada (`oversubscribed: true`, campos fps 0): el exceso de cámaras para el cableado no se puede solucionar mediante el binning o el ROI: reduce el número de cámaras, activa las tramas jumbo o cambia a una tarjeta de red más rápida (véase [Sobresuscripción](#over-subscription-the-per-cam-floor)).
- **`analyze_array_network()` indica que el anillo de recepción (RX ring) de la tarjeta de red es muy pequeño (~0,26 MB) / las puertas de conexión muestran «FRAMES WILL DROP»** → El anillo de recepción de la tarjeta de red del host está en su valor predeterminado (a menudo se restablece a 32 tras una actualización del controlador de la tarjeta de red). En un adaptador Realtek USB de 10 GbE, configura `ReceiveBufferLen=256` y `PendingReceives=64` (con privilegios elevados) y, a continuación, reinicia el backend para que vuelva a leer el anillo. Procedimiento completo: [Referencia de la CLI → Configuración y ajuste de la tarjeta de red del host](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **El host se cuelga al reiniciarse o apagarse; posteriormente, aparecen errores WMI `Invalid class` / la tarjeta de red no se activa** → Un controlador USB de 10 GbE obsoleto provoca el error `DRIVER_POWER_STATE_FAILURE` (pantalla azul de la muerte [BSOD] `0x9F`). Actualiza el controlador del adaptador a una versión reciente (≥ 2026) y vuelve a aplicar la configuración del anillo de recepción. Consulte [Referencia de la CLI → Configuración y ajuste de la NIC del host](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **La captura de la matriz falla con el código HTTP503 «No hay ningún grupo de tramas sincronizado reciente»** → La matriz no ha publicado ningún grupo de tramas nuevo durante el tiempo de espera. Comprueba `arr.status()`: `stalled: True` (o `streaming: False`) junto con `actual_fps`; un valor de 0 en `actual_fps` significa que el bucle de captura ha dejado de publicar — desconecta y vuelve a conectar la matriz. Si `stalled` es False y `actual_fps` es superior a 0, vuelve a intentarlo. Justo después de `connect_array()`, una captura ya lleva esperando hasta 20 s por el primer grupo.
- **`status()` muestra `link_speed_mbps: 4294`** → Windows no ha podido leer la velocidad del adaptador; esto es habitual en los adaptadores Ethernet USB y no constituye un error. Consulta [Velocidades de tramas esperadas](#expected-frame-rates).
- **`update_firmware()` falla con `device never connected back on TCP <port>`** → Un cortafuegos ha bloqueado la conexión del DAQ-E con este ordenador. El instalador de Windows añade la regla (véase [Configuración del dispositivo DAQ-E](#daq-e-device-settings)); si se utiliza otro cortafuegos, permita el tráfico TCP entrante hacia el servidor desde la subred del sensor.
- **Reflectancia rechazada** → Para obtener la reflectancia en escala absoluta, es necesario vincular un DAQ activo a la cámara (o matriz). Vincúlalo a través de la interfaz gráfica de usuario o utiliza `processing="radiance"` (W/m²/sr/nm), que no requiere un sensor emparejado.
- **La captura con `smart=True` tarda más de lo esperado** → La convergencia de AE depende de la dinámica de la escena. Con `ArrayHandle.capture_smart()`, amplíe `exposure_tolerance_pct`, acorta el `stability_window_s` o reduce el `settle_timeout_s` para obtener un disparo más rápido y menos estabilizado.

---

## Véase también

- [Referencia de la CLI](cli-reference.md): cada subcomando de la CLI se corresponde con una llamada al SDK.
- [Sensores de luz DAQ](https://mapir.gitbook.io/chloros/daq-light-sensors) en el manual de Chloros: los sensores, sus límites y rango calibrado, el registro de datos y la conexión en red de DAQ-E.
- [Cámaras LATTICE](https://mapir.gitbook.io/chloros/lattice-cameras/lattice) en el manual de Chloros: conexión, ajustes de la cámara, modos de captura y matrices.
- Documentación en línea: `https://mapir.gitbook.io/chloros/api-python-sdk`

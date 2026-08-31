# Matrices multicámara

Una **matriz**LATTICE consiste en dos o más cámaras LATTICE conectadas como una única unidad sincronizada. Una de las cámaras actúa como**maestra**: envía un pulso de disparo GPIO por hardware a una línea de sincronización compartida (por defecto,**Línea 2**), de modo que todas las cámaras capturan el mismo instante. Chloros añade sincronización temporal PTP, una vista previa en directo (mosaicos por cámara o una única composición multibanda alineada) y captura sincronizada: cada pasada de captura produce un**grupo de fotogramas** en el que todas las cámaras comparten la misma marca de tiempo y el mismo ID de fotograma (indicado como `fid:N` en la salida de captura).

Las matrices son la forma en que las cámaras monocromáticas (M3M) generan índices de vegetación: una cámara aporta una banda y la matriz las alinea en una pila multibanda. Véase [Cámaras monocromáticas e índices de vegetación](mono-indices.md).

Existen tres formas equivalentes de conectar una matriz, y todas ellas ejecutan el mismo flujo de «preparación inteligente»:

| Superficie | Punto de entrada |
| --- | --- |
| Interfaz gráfica de usuario | Pestaña «Cámaras» → **Conectar matriz** (botón azul) |
| CLI | `chloros-cli lattice array-connect --serials SN1,SN2,…` (primer número de serie = maestro) |
| Python SDK | `connect_array(serials=[…])` → `ArraySession` (el primer número de serie es el maestro) |

Smart-prep realiza, en este orden: una prueba de capacidad de red (ping ICMP DF + prueba GVSP), selección del nivel de sincronización, reducción automática del tamaño de trama para adaptarse al cable, activación de PTP, selección automática del formato de píxeles por cámara, el ajuste inicial de la exposición automática a partir del estado guardado de cada cámara y la configuración del disparador GPIO en la Línea 2.

{% hint style="info" %}
Las cámaras deben estar accesibles en el enlace para que todo esto funcione; consulta [Conexión de cámaras](connecting.md) para obtener información sobre la detección, el direccionamiento y la descarga de la calibración de la primera conexión. En el caso de configuraciones con múltiples, la configuración del anillo de recepción de la tarjeta de red del host es tan importante como la velocidad del enlace; la tabla completa de síntomas y soluciones se encuentra en [CLI Referencia § Configuración y ajuste de la tarjeta de red del host](../reference/cli-reference.md#host-nic-setup--tuning-lattice-arrays).
{% endhint %}

## El cuadro de diálogo «Conectar matriz»

La pestaña «Cámaras» → **Conectar matriz**abre un asistente de tres pasos:**Seleccionar → Modo de visualización → Ajustes**.

### Paso 1 — Seleccionar la cámara maestra y las esclavas

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Select scene, with 3-4 LATTICE cameras discovered. Table showing Camera / Serial / IP / Master radio / Slave checkbox columns, with the green "GPIO master detected — selections pre-populated" probe banner visible above the table. -->

El cuadro de diálogo escanea la red tan pronto como se abre («Escaneando la red...») y, a continuación, comprueba el cableado del disparador GPIO («Comprobando el cableado GPIO...»). Se necesitan al menos **2 cámaras** para crear una matriz.

La comprobación del cableado rellena automáticamente la selección de roles cuando es posible y muestra uno de estos tres mensajes:

| Mensaje | Significado |
| --- | --- |
| «Maestro GPIO detectado — selecciones rellenadas automáticamente» (verde) | La comprobación ha encontrado la topología de activación; las casillas de selección de «maestro» y «esclavo» ya están marcadas. |
| «No se ha detectado ningún maestro: comprueba el cable GPIO» (naranja) | Ninguna cámara ha detectado un pulso de activación; comprueba el cableado de sincronización. Aún así, puedes seleccionar las funciones manualmente. |
| «Sin cable de sincronización: {seriales}» (naranja) | Las cámaras de la lista no tienen conectado ningún cable de sincronización. |

La tabla de cámaras tiene las columnas **Cámara / Serie / IP / Maestro (radio) / Esclavo (casilla de selección)**:

* Selecciona exactamente **un maestro**y**uno o más esclavos**. Al volver a hacer clic en la radio del maestro actual, se desmarca.
* Una cámara marcada como **«Sin cable de sincronización»** nunca puede seleccionarse como esclava: una esclava sin cableado de disparo esperaría eternamente en la línea de sincronización y proporcionaría una señal muerta. Conecta esa cámara como cámara independiente en su lugar.
* Las cámaras que ya están conectadas de forma independiente *no* se desactivan: la conexión a la matriz libera la sesión independiente y vuelve a abrir la cámara dentro de la matriz.

**Siguiente: Modo de visualización →**se habilita una vez que se han elegido un maestro y al menos un esclavo.****Volver a escanear** vuelve a ejecutar la detección y la prueba de cableado.

{% hint style="warning" %}
**Cancelar** está desactivada mientras se está realizando un escaneo o una comprobación; cancelar a mitad de la comprobación puede provocar un fallo de la cámara SDK con el firmware de la cámara LATTICE. Espera a que termine el indicador de carga.
{% endhint %}

### Paso 2 — Modo

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Display Mode scene, showing the two selectable cards ("Separate Cameras" and "Combined Cameras") with Combined selected/highlighted as the default. -->

de visualización | Modo | Qué se obtiene |
| --- | --- |
| **Cámaras independientes** | Un mosaico en directo por cámara, todos activados a la vez para que los fotogramas se mantengan sincronizados. Cada cámara conserva su propio color y sus propios ajustes. |
| **Cámaras combinadas** *(predeterminado)* | Un único mosaico que muestra la composición multibanda alineada NDVI/index. Las cámaras comparten el color de la matriz. |

El modo de visualización solo cambia la presentación de la vista previa en directo; el comportamiento de captura es el mismo en ambos.

### Paso 3 — Ajustes de la matriz y resultado

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, healthy state: left column with ROI / Binning / Pin resolution / Trigger Rate controls, right "Projected Outcome" column showing green "Simultaneous capture" tier, an fps range, the NIC line, the "Sim-emit burst" line, and the "Wire budget" line with a checkmark. -->

previsto Al entrar en esta escena, Chloros solicita al backend una **recomendación**y aplica automáticamente una combinación de ROI y binning que se adapta al anillo de recepción de tu NIC (prefiere el binning al recorte de ROI, ya que el binning conserva todo el campo de visión). Cada cambio que realices vuelve a ejecutar el análisis en tiempo real y actualiza el panel de la derecha**Resultado previsto**.

Columna izquierda — ajustes:

| Control | Opciones | Predeterminado | Notas |
| --- | --- | --- | --- |
| **ROI (campo de visión)** | Completo (2048×1536) / Mitad (1024×768) / Cuarto (512×384) | Completo | Recorte del sensor: recorte a la mitad o a un cuarto en una región más pequeña con el paso de píxeles nativo. |
| **Binning** | 1× / 2× (suma 2×2) / 4× (suma 4×4) | 1× | Binning por hardware: 2×2 = campo de visión completo a una cuarta parte del coste de transmisión; 4×4 = campo de visión completo a 1/16. Oculto si las cámaras no admiten el binning. |
| **Imagen en el cable** (lectura) | — | — | El ancho × altura tras el binning que se envía realmente por el cable, redondeado a múltiplos de 16 (mínimo 64). |
| **Resolución de pines**| casilla de selección | desactivada | Chloros normalmente activa el binning automáticamente al conectarse cuando la velocidad prevista cae por debajo de**1,5 fps**. La fijación de pines mantiene el tamaño de fotograma elegido y acepta la tasa más baja, lo que convierte una configuración con exceso de demanda en un rechazo rotundo de la conexión en lugar de una reducción automática de la tasa. |
| **Tasa de disparo** | 0,5–60 fps, paso de 0,1 | vacío = automático | La tasa de disparo del maestro. Déjalo en blanco para que Chloros lo calcule. |
| **Presupuesto de ancho de banda**| 20–2000 MB/s, paso de 10 | vacío = automático | La cantidad que el host puede absorber realmente, en MB/s —**la única cifra de la que depende toda la asignación de la matriz.** Detectado automáticamente a partir del adaptador de red. Redúcelo si la matriz informa de tramas dañadas: el valor detectado sobreestima la capacidad de los adaptadores USB y los conmutadores compartidos. Al modificarlo, se vuelve a ejecutar la proyección en tiempo real. |

Columna de la derecha — **Resultado previsto**:

* **Nivel de sincronización** — «Captura simultánea» (verde), «Captura simultánea (emisión escalonada FTD)» (verde), «Captura escalonada (desviación de 100 ms)» (ámbar) o «Configuración demasiado grande» (rojo).
* **Proyección de fps** — se muestra como un rango («atenuado → brillante»), ya que la velocidad de una matriz sincronizada viene determinada por la exposición de la cámara más lenta.
* **Línea de la NIC**: velocidad de enlace y rendimiento sostenido («NIC {mbps} Mbps · sostenido {N} MB/s»).
* **Comprobación de ráfagas de emisión simultánea**: ¿puede el anillo de recepción de la NIC del host absorber una ráfaga simultánea procedente de todas las cámaras? («Ráfaga de emisión simultánea: X MB · Anillo de la NIC utilizable: Y MB ✓/✗»).
* **Comprobación del presupuesto de cable** — demanda agregada en estado estacionario frente al límite máximo del cable a prueba de colisiones («Presupuesto de cable: {demanda} MB/s solicitados por {n} cámaras · límite máximo {límite} MB/s ✓/✗ sobresuscrito»).
* **«Número máximo de cámaras en este cable: {n} — establecido por el límite mínimo de ancho de banda por cámara, por lo que el agrupamiento no lo aumenta.»** — se muestra cuando te acercas al límite máximo de cámaras (o lo superas).
* **«SE PERDERÁN FOTOGRAMAS con esta configuración».**— Advertencia en rojo con el motivo indicado por el backend, además de una lista de obstáculos y**sugerencias de solución** en azul («Para que esta matriz quepa en la red» / «Para habilitar la captura simultánea»).**Aplicar y conectar** permanece desactivado hasta que exista una proyección, y su etiqueta indica por qué se rechaza:

| Etiqueta del botón | Significado | Qué ayuda realmente |
| --- | --- | --- |
| «Analizando...» | El análisis aún está en curso. | Espera. |
| **«Demasiadas cámaras para esta red»**| La matriz sobrecarga el canal (fallo en la comprobación de agregación). | Menos cámaras, tramas jumbo de extremo a extremo o una tarjeta de red más rápida.**Reducir el ROI NO servirá de nada** — véase más abajo. |
| **«Reducir el ROI para habilitar»** | Se perderían tramas con esta configuración (fallo en la comprobación de ráfagas/anillo). | Reduce el ROI, aumenta el binning o repara el anillo de recepción de la tarjeta de red. |

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, over-subscribed state: red "Wire budget ... over-subscribed" line, the "Max cameras on this wire" hint, and the Apply button reading "Too many cameras for this network". Reproduce by configuring more cameras than the 1 GbE ceiling (e.g. 7+ cams at 1500 MTU) or with CHLOROS-simulated models via `lattice analyze-array`. -->

Durante la conexión, puede aparecer un **panel verde de descarga de calibración** con una barra de progreso por puerto serie: la primera vez que se conecta una cámara a un equipo, Chloros descarga su paquete de calibración de fábrica de ~3,8 MB desde la cámara a través de GigE (aproximadamente 70 segundos por cámara). Las cámaras almacenadas en caché nunca muestran este panel. Véase [Conexión de cámaras](connecting.md).

## Ancho de banda: cuántas cámaras caben

La capacidad de una matriz depende del cable, no de Chloros, por lo que las cifras de planificación se encuentran en el manual del hardware: **[Planificación del ancho de banda de la matriz](https://mapir.gitbook.io/lattice-camera/setup/array-bandwidth-planning)**.

Lo que hace Chloros con estos datos: el cuadro de diálogo de conexión ejecuta una prueba de red, calcula la frecuencia de fotogramas alcanzable y selecciona un nivel adecuado. Si la matriz sobrecarga el cable, rechaza la conexión en lugar de descartar paquetes de forma silenciosa; véase el panel de resultados previstos descrito anteriormente.

## Cuando faltan tramas

Una cámara puede estar ausente de un grupo publicado por dos razones completamente diferentes,
y cada una requiere una solución opuesta. Chloros las contabiliza por separado en lugar de informar de una
cifra «incompleta» que no especifique ninguna de las dos:

| Qué ha ocurrido | Qué significa | Dónde buscar |
| --- | --- | --- |
| **Corrupto**— la trama llegó y presentaba errores estructurales | Pérdida de paquetes GVSP en la ruta de red | El**presupuesto de cableado**, el anillo de recepción de la NIC, las tramas jumbo, el conmutador |
| **Nunca llegó**— no llegó ninguna trama | La cámara no se activó, o no salió nada de ella | El**cable de sincronización M8**, la línea de sincronización, si todos los miembros están activados |

La distribución se reevalúa cada 10 segundos mientras el conjunto de cámaras transmite. Por encima del 5 %, se
registra indicando ambos valores, y cada búfer dañado se notifica la primera vez que
ocurre por cámara; después, se agrupa una vez por minuto para que una sesión larga siga siendo legible.

**Los fotogramas dañados con cero «nunca llegados» significan que el disparo y la sincronización del cable son perfectos**y que todos los fotogramas perdidos se deben a la ruta de red. La solución es reducir el**presupuesto de cable** y
volver a conectar.

{% hint style="warning" %}
**Reducir la frecuencia de disparo no ayuda con los fotogramas corruptos.** El ritmo de paquetes
de la cámara se establece una sola vez, al conectarse. Reducir la frecuencia de disparo cambia la frecuencia con la que se produce una ráfaga,
no la velocidad a la que la propia ráfaga se transmite por el cable. En un equipo medido de 4 cámaras, una
reducción de 5 veces en la frecuencia de activación no cambió nada, mientras que reducir el presupuesto de banda de 240 a
200 MB/s hizo que el mismo equipo pasara de un 10,4 % de tramas corruptas a cero.
{% endhint %}

Una matriz en funcionamiento no puede replanificarse por sí misma: hay que desconectarla y volver a conectarla para que el selector de tiempo de conexión
pueda funcionar con el nuevo presupuesto.

### Los adaptadores de red USB tienen un límite de 200 MB/s

Un adaptador Ethernet USB anuncia su velocidad de enlace *Ethernet*, pero lo que realmente puede
mantener está limitado por el bus USB y su controlador. A un adaptador USB de 10 GbE se le solía atribuir
un rendimiento de aproximadamente 1000 MB/s —una cifra que nunca se había medido— y ajustar
cuatro cámaras a ese margen fantasmal corrompía entre el 6 % y el 18 % de los fotogramas, mientras que la matriz
seguía indicando una frecuencia de fotogramas objetivo correcta. Los adaptadores conectados por USB tienen ahora un límite de
**200 MB/s**. El límite es un valor absoluto y no un porcentaje, ya que el limitador es el
bus: un adaptador USB de 1 GbE alcanza unos 80 MB/s y no se ve afectado.

Si tu host es realmente más rápido que el límite, aumenta el **Wire Budget** para indicarlo.

## Sincronización temporal PTP

La *sincronización* de fotogramas proviene del disparador de hardware; **PTP** (IEEE 1588 PTPv2) proporciona *marcas de tiempo* comparables en todos los dispositivos. Se activa por defecto al conectar la matriz:

* El **backend del host Chloros ejecuta el «grandmaster» PTP**. Las cámaras LATTICE y los sensores de luz DAQ-E se sincronizan con él en el dominio 0, por lo que las marcas de tiempo de las imágenes y los espectros DAQ se ajustan a un mismo reloj (~1 ms).
* `--no-ptp` (CLI) lo desactiva para el trabajo en banco; en ese caso, las marcas de tiempo entre cámaras **no** son comparables.
* Comprueba el estado de la sincronización con el CLI:

```bash
chloros-cli time-sync status     # grandmaster state, clock identity
chloros-cli time-sync peers      # slaves seen (cameras + DAQ-E sensors)
chloros-cli time-sync cameras    # per-camera PtpStatus / PtpOffsetFromMaster / PtpMeanPathDelay
```

La pestaña «Cámaras» en sí misma no tiene ningún indicador PTP; allí se muestran los datos de sincronización por cámara: el **Rol**(Maestro/Esclavo), de solo lectura, la**Línea de sincronización**y el nivel de**Capacidades** de la matriz. El estado PTP de DAQ-E se muestra en los detalles del sensor de la pestaña «Sensores de luz».

## La vista

<!-- SCREENSHOT-NEEDED: Cameras tab with a connected combined array: sidebar showing the ARRAY row (color badge, array name, "DAQ · on" pill) with indented member camera rows, and the main area showing the combined index composite tile with the LUT-colored NDVI render, top-left array name pill, and top-right fps readout. -->

en directo de la matriz El área principal de visualización ofrece dos diseños (se alternan en la barra superior): **vista de cuadrícula**(cada mosaico es una celda; se pueden arrastrar para reordenarlos cuando el candado de la cuadrícula está desbloqueado) y**vista de lista**(matrices a ancho completo en la parte superior, una cámara activa debajo). El control deslizante**Zoom de la transmisión** ajusta el tamaño de los mosaicos; por debajo de los 200 píxeles de ancho de celda, las superposiciones de nombre y fps se ocultan automáticamente.

El **modo separado** muestra un mosaico por cámara. Cada mosaico muestra:

* el nombre de la cámara (arriba a la izquierda),
* una **lectura de fps** (arriba a la derecha): se trata de la *frecuencia real de captura* de la cámara comunicada por el servidor, no de la frecuencia de sondeo de la vista previa (la vista previa en directo tiene un límite de 30 fps independientemente de la frecuencia de captura),
* un punto de estado: verde (transmisión) / ámbar (cargando) / rojo (error),
* un **indicador de fotograma obsoleto** cuando no ha llegado ningún fotograma nuevo durante 2 s — algo normal durante unos 5 s tras cualquier conexión o desconexión, mientras el backend reequilibra el presupuesto de banda entre las cámaras.

El **modo combinado**muestra un único mosaico compuesto: el backend realiza el debayering, escala, alinea, elimina el ruido, convierte a radiancia por banda (más la reflectancia DLS cuando hay un sensor de luz vinculado), evalúa la expresión del índice de la matriz, aplica la LUT y transmite el resultado como MJPEG. Hasta que se renderiza el primer fotograma alineado, el mosaico indica su estado: «Preparando matriz…», «Calibrando alineación…», «Esperando el primer fotograma…» o —si se ha agotado el tiempo de reintentos de alineación automática (unos 30 s)— «Se requiere alineación», junto con un botón**Calibrar alineación**.

Datos útiles sobre el modo combinado:

* La composición se registra en el fotograma de la cámara **maestra**. El enfoque AE-ROI y la medición puntual en la composición son exactos para la cámara maestra y aproximados para las esclavas; utiliza**Vista dividida** (ajustes de la matriz → «Mostrar cámaras miembros») para obtener mosaicos con precisión de píxel por cámara sin abrir conexiones de cámara adicionales.
* ****Capas de visualización**(ajustes de matriz; desactivadas por defecto) te permiten elegir una capa de primer plano y otra de fondo: cualquier cámara miembro o**Índice**. Con el primer plano = Índice, los píxeles fuera del rango mínimo/máximo de la LUT muestran la capa de fondo.
* ****Resolución de renderizado** (por defecto 720p) establece la altura de la transmisión en directo *y* el tamaño de exportación de la composición guardada. Las imágenes de cada cámara siempre se exportan a resolución completa.
* La alineación se calcula por sesión y nunca se guarda; consulta la sección de alineación del panel de configuración de la matriz para ver los residuos RMS y el botón «Recalibrar».

## Captura: monitorización frente a análisis

Las superficies de captura de la matriz se dividen claramente en **grado de monitorización**(grabar lo que se ve) y**grado de análisis** (grabar datos sin procesar, calibrar más tarde):

| Flujo de trabajo | Nivel | Qué se guarda | Interfaz gráfica | CLI |
| --- | --- | --- | --- | --- |
| **Captura** (imágenes fijas) | Análisis | Un grupo de fotogramas sincronizados por pasada; archivos por cámara en cada nivel de exportación seleccionado (sin procesar/despixelar/radiancia/reflectancia/vista previa/índice) + archivo sidecar `.daq` | *Botón *Capturar todo** + Ajustes de captura | `lattice array-capture` |
| **Grabar vídeo índice** | Supervisión | La composición del índice combinado en directo tal y como se muestra: 8 bits, resolución de vista previa, LUT integrada; requiere que la transmisión en directo esté abierta | ● Grabar vídeo índice (matrices combinadas) | `lattice array-record` |
| **Ráfaga sin procesar → crear vídeo**| Análisis | Fotogramas sin procesar del sensor a la velocidad máxima de captura + manifiesto + `.daq`; a continuación, reconstrucción fuera de línea en vídeo calibrado de radiancia, reflectancia e índice, sincronizado temporalmente con las lecturas del DAQ | ⦿ Grabar ráfaga sin procesar →**Crear vídeo** | `lattice array-burst` → `lattice array-build-video` |

Regla general: si los píxeles van a proporcionar *mediciones*, utiliza la captura o la ráfaga (nivel de análisis); si solo necesitas *ver o mostrar* lo que ha captado la matriz, graba el vídeo indexado (nivel de monitorización).

### Ajustes de captura (GUI)

<!-- SCREENSHOT-NEEDED: Capture Settings pane (gear next to Capture All) with a connected array: capture-mode buttons (Single/Continuous/Interval), the bulk export-type toggle row, the Fastest Capture toggle, and the per-array group card showing the Aligned checkbox and the "Record index video" / "Record raw burst" buttons. -->

El icono con forma de engranaje junto a **Capturar todo** abre el panel de ajustes de captura (requiere tener un proyecto abierto, ya que las capturas se guardan en él):

* **Modo de captura**:**Único**(una pasada) /**Continuo**(consecutivo; limitado por un número de capturas, por defecto 1, o una duración, por defecto 10 s) /**Intervalo** (timelapse: N capturas cada X intervalos hasta un total de Y; por defecto, 1 cada 5 s durante 1 minuto).
* **Tipos de exportación por cámara**: Raw, sin bayering, radiancia, reflectancia, vista previa, índice; todas las opciones aplicables están activadas por defecto. Radiance/Reflectance están ocultas para las cámaras RGB-filter;**Reflectance solo aparece cuando la cámara tiene un sensor de luz DAQ** (propio o heredado de la matriz); Index requiere una expresión de índice configurada.
* **Alineado**(por matriz,**activado** por defecto): deforma las exportaciones de los miembros según el perfil de alineación de la matriz, de modo que las exportaciones quedan registradas píxel a píxel. El formato Raw siempre permanece sin deformar, pero incluye la transformación en los metadatos.
* **Captura más rápida** (conmutador): solo datos sin procesar + la lectura DAQ asignada + el compuesto de índice combinado libre, omitiendo los cálculos de calibración en el momento de la captura para obtener la máxima velocidad; la radiancia, la reflectancia y el índice se reconstruyen posteriormente a partir del `.daq` guardado.
* Las selecciones se conservan con el proyecto. Se omiten las cámaras ocultas o en pausa.

El equivalente CLI (mismo punto final de backend, misma semántica):

```bash
# One synced group, every applicable export level per camera (the default)
chloros-cli lattice array-capture -o output/

# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# 30-second monitoring clip of the combined index view, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# 5-second analysis-grade raw burst, then build the combined index video
chloros-cli lattice array-burst --duration 5 --build --products combined:index --fps 10 -o capture/
```

La compresión de capturas de TIFF es `deflate` (sin pérdida, por defecto) o `none` — las tablas completas de indicadores, la estructura de la carpeta de capturas y las reglas de reprocesamiento se encuentran en la [Referencia de CLI](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

## Emparejamiento de un sensor de luz DAQ

Las vistas previas con corrección de reflectancia e iluminación necesitan datos de luz descendente procedentes de un sensor DAQ (conectado en la pestaña **Sensores de luz**):

* La **fila de la matriz**de la barra lateral muestra un**botón «DAQ · activado/desactivado»**: aparece *activado* cuando se ha configurado un sensor de luz a nivel de matriz **o** cuando alguna de las cámaras de la matriz tiene el suyo propio; su información sobre herramientas indica exactamente qué sensor alimenta a cada cámara.
* Asigna la configuración para toda la matriz en los ajustes de la matriz → **Sensor de luz ambiental**→ menú desplegable**Sensor de luz**. La selección se mantiene con el proyecto, se aplica a todas las cámaras del conjunto y las cámaras individuales pueden seguir anulándola con su propio sensor.
* La línea de estado situada debajo muestra el estado en tiempo real: **Desactivado**→ «Esperando el primer espectro…» →**«Activo: todas las cámaras del conjunto tienen la iluminación corregida»** → o, si no ha llegado ningún espectro nuevo en los últimos 3 s, un aviso de datos obsoletos: se sigue utilizando la última lectura (las lecturas nunca caducan en la ruta de captura).

Con un sensor asignado: el tipo de exportación «Reflectancia» queda disponible, las vistas previas en directo se corrigen en cuanto a la iluminación, la exposición automática predictiva puede utilizar el espectro y cada captura de reflectancia escribe la lectura del DAQ realmente utilizada como un **archivo complementario `.daq`** junto a la imagen, de modo que la captura pueda volver a procesarse más adelante.

## Opciones `array-connect` CLI

| Indicador | Predeterminado | Descripción |
| --- | --- | --- |
| `--serials SN1,SN2,…` | Detección automática de todas las cámaras LATTICE (se necesitan ≥2) | **El primer número de serie es el MASTER.** |
| `--line {Line0,Line2,Line3}` | `Line2` | Línea de sincronización GPIO. |
| `--target-fps F` | auto | Frecuencia de disparo del disparador maestro. |
| `--binning {1,2,4}` | automático | Agrupación por bin de hardware. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | automático | Anulación por parte del experto del selector de nivel de sincronización. |
| `--wire-ceiling-mbps MB_PER_S` | detección automática | Presupuesto de ancho de banda del host en MB/s — el campo **Presupuesto de cable**. Redúcelo si la matriz informa de tramas dañadas. Se guarda con el proyecto, por lo que al volver a conectarse más adelante se restablecerá. |
| `--no-recommend` | desactivado | Omite el paso de análisis de red. |
| `--no-ptp` | desactivado | Desactiva PTP (las marcas de tiempo entre cámaras no serán comparables). |

`lattice array-list`, `array-status` y `array-disconnect` gestionan la sesión persistente. La referencia completa de subcomandos, incluida la alineación (`align-calibrate` / `align-apply`) y las herramientas de red, se encuentra en la [Referencia de CLI § chloros-cli lattice](../reference/cli-reference.md#chloros-cli-lattice); los equivalentes de SDK (`connect_array`, `ArraySession`, `attach_array`, `analyze_array_network`) se encuentran en la [Referencia de SDK](../reference/sdk-reference.md). A partir de Python, el presupuesto de cableado es `connect_array(..., wire_ceiling_mbps=120)`, y la división entre «corrupto en producción» y «nunca llegó» se encuentra en [`/api/camera/array/<id>/capability`](../reference/sdk-reference.md#array-health--which-subsystem-is-losing-frames).

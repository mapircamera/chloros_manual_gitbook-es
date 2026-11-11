# Matrices multicámara

Una **matriz**LATTICE consiste en dos o más cámaras LATTICE conectadas como una única unidad sincronizada. Una de las cámaras actúa como**maestra**: envía un pulso de disparo GPIO por hardware a una línea de sincronización compartida (por defecto,**Línea 2**), de modo que todas las cámaras capturan el mismo instante. Chloros

añade sincronización temporal PTP, una vista previa en directo (mosaicos por cámara o una única imagen compuesta multibanda alineada) y captura sincronizada: cada pasada de captura produce un **grupo de fotogramas** en el que todas las cámaras comparten la misma marca de tiempo y el mismo ID de fotograma (indicado como `fid:N` en la salida de captura).

Las matrices son la forma en que las cámaras monocromáticas (M3M) generan índices de vegetación: cada cámara aporta una banda y la matriz las alinea en una pila multibanda. Véase [Cámaras mono e índices de vegetación](mono-indices.md).

Hay tres formas equivalentes de conectar una matriz, y todas ellas ejecutan el mismo flujo «smart-prep»:

| Superficie | Punto de entrada |
| --- | --- |
| GUI | Pestaña «Cámaras» → **Conectar matriz** (botón azul) |
| CLI

| `chloros-cli lattice array-connect --serials SN1,SN2,…` (primer número de serie = maestro) |
| Python

SDK

| `connect_array(serials=[…])` → `ArraySession` (primer número de serie = maestro) |

«Smart-prep» realiza lo siguiente: una prueba de capacidad de red (ping ICMP DF + prueba GVSP), selección del nivel de sincronización, una proyección de la frecuencia de fotogramas con el tamaño de fotograma solicitado, la decisión de transporte empaquetado de 12 bits (véase [Transporte empaquetado de 12 bits](#packed-12-bit-transport)), la activación de PTP, la selección automática del formato de píxel por cámara, el ajuste inicial de la exposición automática a partir del estado guardado de cada cámara y la configuración del disparador GPIO en la línea 2.

**Sensor completo por defecto (novedad en la versión 1.3.0).** Connect mantiene el tamaño de fotograma que has solicitado —el sensor completo de 2048×1536, sin binning ni recorte, a menos que elijas lo contrario — y permite que la velocidad de fotogramas se adapte a lo que ofrecen el host y la red. Nunca realiza binning ni recortes por sí mismo (la versión 1.2.0 activaba automáticamente el binning por debajo de 1,5 fps). El binning o un ROI más pequeño es decisión tuya: en el cuadro de diálogo de conexión, elige una opción y el «Resultado previsto» muestra la velocidad de fotogramas resultante; `array-connect` muestra las opciones más rápidas con sus velocidades previstas.

{% hint style="info" %}
Las cámaras deben estar accesibles a través del enlace para que todo esto funcione; consulta [Conexión de cámaras](connecting.md) para la detección, el direccionamiento y la descarga de la calibración de la primera conexión. En los montajes con varias cámaras, la configuración del anillo de recepción de la tarjeta de red del host es tan importante como la velocidad del enlace; la tabla completa de síntomas y soluciones se encuentra en [Referencia de CLI

, § Configuración y ajuste de la tarjeta de red del host](../reference/cli-reference.md#host-nic-setup--tuning-lattice-arrays).
{% endhint %}

## El cuadro de diálogo «Conectar matriz»

La pestaña «Cámaras» → **Conectar matriz**abre un asistente de tres pasos:**Seleccionar → Modo de visualización → Ajustes**.

### Paso 1: seleccionar la cámara maestra y las esclavas

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Select scene, with 3-4 LATTICE cameras discovered (an FRGB among them). Table showing Camera / Serial / IP / Master radio / Slave checkbox columns in camera order (M3C first, FRGB at the top), with the FRGB pre-selected as master and the green "Sync cable found on N cameras. Any of them can be master; the RGB camera (else the first) is pre-selected." probe banner visible above the table. -->



El cuadro de diálogo escanea la red nada más abrirse («Escaneando la red...») y, a continuación, comprueba los cables de sincronización («Comprobando los cables de sincronización...») para detectar qué cámaras están conectadas a la línea de sincronización M8. Se necesitan al menos **2 cámaras** para crear una matriz.

Los cables de sincronización M8 convergen en un punto de unión, por lo que **cualquier cámara conectada puede actuar como maestra**. La prueba preasigna los roles —la cámara RGB

(`-FRGB`) como maestra cuando está conectada, o de lo contrario la primera cámara conectada, y las demás cámaras conectadas como esclavas— y muestra uno de estos tres mensajes:

| Mensaje | Significado |
| --- | --- |
| «Se ha detectado cable de sincronización en {n} cámaras. Cualquiera de ellas puede ser maestra; la cámara RGB

(o, en su defecto, la primera) está preseleccionada.» (verde) | La sonda ha detectado las cámaras en la línea de sincronización; las casillas de selección de «maestra» y «esclava» ya están marcadas. |
| «No se ha detectado ninguna maestra: comprueba el cable GPIO» (naranja) | Ninguna cámara ha detectado un pulso de disparo; comprueba el cableado de sincronización. Los roles siguen preasignados (la cámara RGB

, o en su defecto la primera cámara de la lista, como maestra) y puedes modificarlos. |
| «Sin cable de sincronización: {seriales}» (naranja) | Las cámaras de la lista no tienen conectado ningún cable de sincronización. |

La tabla de cámaras tiene las columnas **Cámara / Serial / IP / Maestra (radio) / Esclava (casilla de selección)**. Las filas se muestran en**orden de cámara**: las cámaras M3C (Bayer) antes que las M3M (mono); entre las M3C, FRGB, FRGN, FOCN y, a continuación, FNGB; las M3M por longitud de onda del filtro, de menor a mayor (F450 antes que F808). La lista de cámaras de la barra lateral y la disposición predeterminada de la cuadrícula siguen el mismo orden.

* Selecciona exactamente **una maestra**y**una o más esclavas**. Al volver a hacer clic en el botón de opción de la maestra actual, esta se desmarca. Para seleccionar una maestra diferente basta con un clic: la nueva maestra abandona las esclavas y la maestra anterior pasa a ser una esclava (a menos que no tenga cable de sincronización).
* Una cámara marcada como **«Sin cable de sincronización»** nunca puede seleccionarse como esclava: una esclava sin cableado de disparo esperaría indefinidamente en la línea de sincronización y proporcionaría una señal muerta. Conecta esa cámara como cámara independiente en su lugar.
* Las cámaras que ya estén conectadas de forma independiente *no* se desactivan: la conexión en matriz libera la sesión independiente y vuelve a abrir la cámara dentro de la matriz.

**Siguiente: Modo de visualización →**se habilita una vez que se han elegido un maestro y al menos un esclavo.**Volver a escanear** vuelve a ejecutar la detección y la prueba de cableado.

{% hint style="warning" %}
**Cancelar** está desactivado mientras se está realizando un escaneo o una prueba de cableado; cancelar a mitad de la prueba puede provocar un fallo del SDK

de la cámara en el firmware de las cámaras LATTICE. Espera a que termine el indicador de carga.
{% endhint %}

### Paso 2 — Modo de visualización



<!-- SCREENSHOT-NEEDED: Array Connect dialog, Display Mode scene, showing the two selectable cards ("Separate Cameras" and "Combined Cameras") with Combined selected/highlighted as the default. -->

| Modo | Qué se obtiene |
| --- | --- |
| **Cámaras separadas** | Un mosaico en directo por cámara, todos activados a la vez para que los fotogramas permanezcan sincronizados. Cada cámara mantiene su propio color y sus propios ajustes. |
| **Cámaras combinadas** *(predeterminado)* | Un único mosaico que muestra la composición multibanda alineada de NDVI

/índice. Las cámaras comparten el color de la matriz. |

El modo de visualización solo cambia la presentación de la vista previa en directo; el comportamiento de captura es el mismo en ambos.

### Paso 3 — Ajustes de la matriz y resultado

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, healthy state: left column with ROI / Binning / Trigger Rate / Wire Budget controls (1.3.0 has no "Pin resolution" checkbox), right "Projected Outcome" column showing green "Simultaneous capture" tier, an fps range, the NIC line, the "Sim-emit burst" line, and the "Wire budget" line with a checkmark. -->

previsto

Al entrar en esta escena, Chloros

solicita al backend una **recomendación**y la aplica: el**sensor completo —sin binning ni recorte de ROI— siempre que la frecuencia de fotogramas lo permita**, sea cual sea la velocidad que proporcionen el host y la conexión. Solo cuando ninguna frecuencia permite utilizar el sensor completo, la recomendación reduce el fotograma, y, en ese caso, da prioridad al binning frente al recorte de la zona de interés (ROI), ya que el binning conserva el campo de visión completo. A partir de ahí, se sacrifica resolución a cambio de velocidad de fotogramas. Cada cambio que realices vuelve a ejecutar el análisis en tiempo real y actualiza el panel de la derecha**Resultado previsto**.

Columna izquierda — ajustes:

| Control | Opciones | Predeterminado | Notas |
| --- | --- | --- | --- |
| **ROI (campo de visión)** | Completo (2048×1536) / Mitad (1024×768) / Cuarto (512×384) | Completo | Recorte del sensor: recorte a la mitad o a un cuarto en una región más pequeña con el paso de píxeles nativo. |
| **Binning** | 1× / 2× (suma 2×2) / 4× (suma 4×4) | 1× | Binning por hardware: 2×2 = campo de visión completo a una cuarta parte del coste de transmisión; 4×4 = campo de visión completo a 1/16. Oculto si las cámaras no admiten binning. |
| **Imagen por cable** (lectura) | — | — | El ancho × altura tras el binning que realmente se envía por cable, redondeado a múltiplos de 16 (mínimo 64). |
| **Frecuencia de disparo** | 0,5–60 fps, paso de 0,1 | vacío = automático | La frecuencia de disparo del maestro. Déjalo en blanco para que Chloros

lo calcule. |
| **Capacidad del cable**| 20–2000 MB/s, en incrementos de 10 | vacío = automático | La cantidad que el host puede absorber realmente, en MB/s —**el único valor del que depende toda la asignación de la matriz.** Se detecta automáticamente a partir del adaptador de red. Redúcelo si la matriz informa de tramas dañadas: el valor detectado sobreestima la capacidad de los adaptadores USB y los conmutadores compartidos. Al modificarlo, se vuelve a ejecutar la proyección en tiempo real. El campo se abre con el presupuesto con el que se conectó la última matriz (guardado con el proyecto), por lo que se mantiene cualquier presupuesto que hayas corregido anteriormente. |

La casilla de selección **Resolución de pines** de la versión 1.2.0 ha desaparecido: la conexión mantiene siempre el tamaño de trama que elijas y, en su lugar, reduce la frecuencia de tramas.

Columna derecha — **Resultado previsto**:

* **Nivel de sincronización** — «Captura simultánea» (verde), «Captura simultánea (emisión escalonada FTD)» (verde), «Captura escalonada (desviación de 100 ms)» (ámbar) o «Configuración demasiado grande» (rojo). Chloros

elige el nivel: captura simultánea siempre que el cable se pueda compartir a una velocidad a la que las cámaras puedan adaptarse (la frecuencia de fotogramas disminuye a medida que se reduce la parte que corresponde a cada cámara), y captura escalonada solo cuando no sea posible; de este modo, una matriz se conecta a cualquier red, a la frecuencia de fotogramas que permita dicha red.
* **Proyección de fps**: la velocidad de fotogramas a la que se espera que funcione la matriz. Con la captura simultánea es un único valor: primero se establece la velocidad y la exposición automática ocupa el tiempo que esta permite; a continuación, aumenta la ganancia para que una escena con poca luz no la reduzca (véase [Cómo gestiona la exposición automática la exposición y la ganancia](camera-settings.md)). Con la captura escalonada se muestra como un rango («oscuro → claro»), ya que la exposición de cada cámara alarga el escalonamiento en una escena oscura.
* **Línea NIC** — velocidad de enlace y rendimiento sostenido («NIC {mbps} Mbps · sostenido {N} MB/s»).
* **Comprobación de ráfagas de emisión simultánea**: ¿puede el anillo de recepción de la NIC del host absorber una ráfaga simultánea procedente de todas las cámaras («Ráfaga de emisión simultánea: X MB · Anillo de la NIC utilizable: Y MB ✓/✗»)?
* **Comprobación del presupuesto de ancho de banda** — demanda agregada en estado estacionario frente al límite máximo de ancho de banda sin colisiones («Presupuesto de ancho de banda: {demanda} MB/s solicitados por {n} cámaras · límite máximo {límite} MB/s ✓/✗ sobresuscrito»).
* **«Número máximo de cámaras en este cable: {n} — establecido por el límite mínimo de ancho de banda por cámara, por lo que la agrupación no lo aumenta.»** — se muestra cuando te acercas al límite máximo de cámaras (o lo superas).
* **«SE PERDERÁN TRAMAS con esta configuración.»**— advertencia en rojo con el motivo del backend, además de una lista de obstáculos y**sugerencias de solución** en azul («Para que esta matriz quepa en la red» / «Para habilitar la captura simultánea»).**Aplicar y conectar** permanece bloqueado hasta que exista una proyección, y su etiqueta indica por qué se rechaza:

| Etiqueta del botón | Significado | Qué ayuda realmente |
| --- | --- | --- |
| «Analizando...» | El análisis aún está en curso. | Espera. |
| **«Demasiadas cámaras para esta red»**| La conexión no puede soportar la matriz ni siquiera con una cámara a la vez (fallo en la comprobación de agregación): hay muchas más cámaras de las que puede gestionar, o hay otras cámaras transmitiendo por la misma conexión. | Menos cámaras, tramas jumbo de extremo a extremo o una tarjeta de red más rápida.**Un ROI más pequeño NO servirá de nada** — véase más abajo. |
| **«Reducir el ROI para habilitar»** | Se perderían fotogramas con esta configuración (fallo en la comprobación de ráfagas/anillo). | Reducir el ROI, aumentar el binning o reparar el anillo de recepción de la tarjeta de red. |

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, over-subscribed state: red "Wire budget ... over-subscribed" line, the "Max cameras on this wire" hint, and the Apply button reading "Too many cameras for this network". Reproduce by configuring more cameras than the 1 GbE ceiling (e.g. 7+ cams at 1500 MTU) or with CHLOROS-simulated models via `lattice analyze-array`. -->



Durante la conexión, puede aparecer un **panel de descarga de calibración** verde con una barra de progreso por puerto serie: la primera vez que se conecta una cámara a un equipo, Chloros

extrae su paquete de calibración de fábrica de ~3,8 MB de la cámara a través de GigE (aproximadamente 70 segundos por cámara). Las cámaras almacenadas en caché nunca muestran este panel; con más de seis cámaras, las filas se desplazan. Véase [Conexión de cámaras](connecting.md).

### Transporte empaquetado de 12 bits

Los formatos de 12 bits (`BayerRG12`, `Mono12`) suelen transmitirse como palabras de 16 bits. **El transporte**comprimido**envía los mismos 12 bits en 1,5 bytes por píxel y los descomprime en el host, por lo que cada fotograma tarda un 25 % menos en enviarse; con el sensor a pleno rendimiento, esto supone aproximadamente un tercio más de fotogramas por segundo.**Los datos guardados son idénticos.** Array Connect lo decide automáticamente (`auto`): solo comprime cuando resulta útil y el host dispone de CPU suficiente para la descompresión. La interfaz gráfica de usuario (GUI) no tiene ninguna opción para ello; desde la CLI

, utiliza `array-connect --packed {auto,on,off}` (por defecto, `auto`). Los formatos de 8 y 16 bits nunca se comprimen.

## Ancho de banda: cuántas cámaras caben

La capacidad de una matriz depende de la conexión, no de Chloros

, por lo que las cifras de planificación se encuentran en el manual del hardware: **[Planificación del ancho de banda de la matriz](https://mapir.gitbook.io/lattice-camera/setup/array-bandwidth-planning)**.

Lo que hace Chloros

con ellos: el cuadro de diálogo de conexión ejecuta una prueba de red, calcula la frecuencia de fotogramas alcanzable y elige un nivel que se ajuste a ella. Una matriz se conecta a cualquier red: en un enlace lento, cada cámara se adapta a su parte del ancho de banda y la frecuencia de fotogramas disminuye, en lugar de que se rechace la matriz o se reduzcan sus fotogramas. Solo se rechaza una matriz que la conexión no pueda soportar ni siquiera una cámara a la vez (**Demasiadas cámaras para esta red**), en lugar de descartar paquetes de forma silenciosa; véase el panel de resultados previstos descrito anteriormente.

## Cuando faltan fotogramas

Una cámara puede estar ausente de un grupo publicado por dos razones completamente diferentes,
y cada una requiere una solución opuesta. Chloros

las contabiliza por separado en lugar de informar de una
cifra «incompleta» que no especifique ninguna de ellas:

| Qué ha ocurrido | Qué significa | Dónde buscar |
| --- | --- | --- |
| **Corrupto**— el fotograma llegó y presentaba errores estructurales | Pérdida de paquetes GVSP en la ruta de red | El**presupuesto de ancho de banda**, el anillo de recepción de la NIC, los fotogramas jumbo, el conmutador |
| **Nunca llegó**— no llegó ninguna trama | La cámara no se activó, o no salió nada de ella | El**cable de sincronización M8**, la línea de sincronización, si todos los miembros están armados |

La distribución se reevalúa cada 10 segundos mientras el conjunto de cámaras transmite. Por encima del 5 %, se
registra indicando ambos valores, y cada búfer dañado se notifica la primera vez que
ocurre por cámara; a continuación, se agrupa una vez por minuto para que una sesión larga siga siendo legible.

**Los fotogramas dañados con cero «nunca llegados» significan que el disparo y la sincronización del cable son perfectos**y que todos los fotogramas perdidos se deben a la ruta de red. La solución es reducir el**presupuesto de cable** y
volver a conectar.

{% hint style="warning" %}
**Reducir la frecuencia de disparo no ayuda con los fotogramas corruptos.** El ritmo de paquetes
de la cámara se establece una sola vez, al conectarse. Reducir la frecuencia de disparo cambia la frecuencia con la que se produce una ráfaga,
no la velocidad a la que la propia ráfaga se transmite por el cable. En un equipo de 4 cámaras medido, una
reducción de 5× en la frecuencia de disparo no cambió nada, mientras que reducir el presupuesto de banda de 240 a
200 MB/s redujo el porcentaje de fotogramas dañados de ese mismo sistema del 10,4 % a cero.
{% endhint %}

Una matriz en funcionamiento no puede replanificarse por sí misma: hay que desconectarla y volver a conectarla para que el selector de tiempo de conexión
pueda ajustarse al nuevo presupuesto.

### Los adaptadores de red USB tienen un límite

Un adaptador Ethernet USB anuncia su velocidad de enlace *Ethernet*, pero lo que realmente
puede mantener está limitado por el bus USB y su controlador. A un adaptador USB de 10 GbE se le solía atribuir
un rendimiento de aproximadamente 1000 MB/s —una cifra que nunca se había medido— y ajustar
cuatro cámaras a ese margen fantasmal corrompía entre el 6 % y el 18 % de los fotogramas, mientras que la matriz
seguía indicando una frecuencia de fotogramas objetivo correcta. Los adaptadores conectados por USB tienen ahora un límite de
**200 MB/s** en Windows

y NVIDIA Jetson, y **400 MB/s** en Linux

x86_64. El límite es un valor absoluto y no un porcentaje, ya que el limitador es el
bus: un adaptador USB de 1 GbE alcanza unos 80 MB/s y no se ve afectado.

Si tu host es realmente más rápido que el límite, aumenta el **Wire Budget** para indicarlo.

## Sincronización temporal PTP

La *sincronización* de fotogramas proviene del disparador de hardware; **PTP** (IEEE 1588 PTPv2) proporciona *marcas de tiempo* comparables en todos los dispositivos. Está habilitado por defecto al conectar la matriz:

* El **backend del host Chloros

ejecuta el «grandmaster» PTP**. Las cámaras LATTICE y los sensores de luz DAQ-E se sincronizan con él en el dominio 0, de modo que las marcas de tiempo de las imágenes y los espectros DAQ se ajustan a un mismo reloj (~1 ms).
* `--no-ptp` (CLI

) lo desactiva para el trabajo en banco; en ese caso, las marcas de tiempo entre cámaras **no** son comparables.
* Comprueba el estado de la sincronización con la CLI

:

```bash
chloros-cli time-sync status     # grandmaster state, clock identity, health (OK, or why its packets are not reaching the network)
chloros-cli time-sync peers      # slaves seen (cameras + DAQ-E sensors)
chloros-cli time-sync cameras    # per-camera PtpStatus / PtpOffsetFromMaster / PtpMeanPathDelay
```

La pestaña «Cámaras» en sí misma no tiene ningún indicador PTP; los datos de sincronización por cámara que aparecen allí son el **Rol**(maestro/esclavo), de solo lectura, la**Línea de sincronización**y el nivel de**Capacidades** de la matriz. El estado PTP de DAQ-E se muestra en los detalles del sensor de la pestaña «Sensores de luz».

## La vista

<!-- SCREENSHOT-NEEDED: Cameras tab with a connected combined array: sidebar showing the ARRAY row (color badge, array name, "DAQ · on" pill) with indented member camera rows in camera order (M3C before M3M, monos by wavelength), and the main area showing the combined index composite tile with the LUT-colored NDVI render, top-left array name pill, and top-right fps readout. -->

en directo de la matriz El área principal de visualización ofrece dos diseños (se alternan en la barra superior): **vista en cuadrícula**(cada mosaico es una celda; se pueden arrastrar para reordenarlas cuando el candado de la cuadrícula está desbloqueado) y**vista de lista**(filas a ancho completo: una matriz**combinada**ocupa una fila —su mosaico compuesto—, mientras que una matriz**separada**asigna a**cada miembro su propia fila**, con su propia transmisión en directo y su propia tabla de objetivos de calibración; debajo de las matrices, una cámara independiente —la activa—). El control deslizante**Zoom de la transmisión** ajusta el tamaño de los mosaicos; por debajo de los 200 píxeles de ancho de celda, las superposiciones de nombre/fps se ocultan automáticamente. Las cámaras que forman parte de la matriz aparecen en la barra lateral y en el diseño predeterminado de la cuadrícula en el orden de las cámaras (M3C antes que M3M, FRGB primero, monocromáticas por longitud de onda); al desbloquear la cuadrícula y arrastrar, se anula el orden de la cuadrícula.

<!-- SCREENSHOT-NEEDED: list view of a connected SEPARATE array (1.3.0): one full-width row per member camera, each with its live feed and its own Calibration Target table below it. -->



El **modo separado** muestra un mosaico por cámara. Cada mosaico muestra:

* el nombre de la cámara (arriba a la izquierda),
* una **lectura de fps** (arriba a la derecha): se trata de la *frecuencia real de captura* de la cámara comunicada por el backend, no de la frecuencia de sondeo de la vista previa (la vista previa en directo tiene un límite de 30 fps independientemente de la frecuencia de captura),
* un punto de estado: verde (transmisión) / ámbar (cargando) / rojo (error),
* un **indicador giratorio de fotograma obsoleto** cuando no ha llegado ningún fotograma nuevo durante 2 s — algo normal durante unos 5 s tras cualquier conexión o desconexión, mientras el backend reequilibra el presupuesto de ancho de banda entre las cámaras.

El **modo combinado**muestra un único mosaico compuesto: el backend realiza el debayering, escala, alinea, elimina el ruido, convierte a radiancia por banda (más la reflectancia DLS cuando hay un sensor de luz vinculado), evalúa la expresión del índice de la matriz, aplica la LUT y transmite el resultado como MJPEG. Hasta que se renderiza el primer fotograma alineado, el mosaico indica su estado: «Preparando matriz…», «Calibrando la alineación…», «Esperando el primer fotograma…» o —si se agotan los intentos de alineación automática del mosaico (aproximadamente un minuto)— «Se requiere alineación» con un botón**Calibrar alineación**.

La lectura de fps del mosaico de la matriz también indica el estado de la transmisión:**«Aplicando ajustes…»**mientras un cambio de configuración (formato, agrupación de píxeles, ROI) reinicia las cámaras,**«Atascado»**cuando la matriz está activa pero ha dejado de enviar grupos de fotogramas, y**«Detenido»**cuando ha dejado de transmitir; en ese caso, vuelve a conectar el conjunto. Cuando la separación espectral ha tenido que limitar una banda mostrada en la imagen compuesta en una gran parte de sus píxeles (un 25 % o más), el mosaico muestra**«Datos de banda de baja confianza · N %»**: los valores de índice calculados a partir de esa banda son dudosos (normalmente, una banda ha recibido muy poca luz).

Datos útiles sobre el modo combinado:

* La imagen compuesta está alineada con el fotograma de la cámara **maestra**. El enfoque AE-ROI y la medición puntual en la imagen compuesta son exactos para la cámara maestra y aproximados para las esclavas; utiliza**Vista dividida** (ajustes del conjunto → «Mostrar cámaras del conjunto») para obtener mosaicos por cámara con precisión de píxel sin abrir conexiones de cámara adicionales.
* ****Capas de visualización**(ajustes de la matriz; desactivadas por defecto) te permiten elegir una capa de primer plano y otra de fondo: cualquier cámara miembro o**Índice**. Con primer plano = Índice, los píxeles fuera de los valores mínimo/máximo de la LUT muestran la capa de fondo.
* **Resolución de renderizado** (por defecto 720p) establece la altura de la transmisión en directo *y* el tamaño de exportación de la composición guardada. Las imágenes de cada cámara siempre se exportan a resolución completa.
* La alineación se calcula por sesión y nunca se guarda. Chloros

alinea **todas las matrices automáticamente tras la conexión, en ambos modos de visualización**, una vez que se ha estabilizado la exposición automática de los miembros (sigue intentándolo hasta 5 minutos). La sección**Alineación**del panel de configuración de la matriz muestra los residuos RMS y el botón**Recalibrar alineación**, también para matrices independientes que no tienen mosaico compuesto.

## Captura: monitorización frente a análisis

Las superficies de captura de la matriz se dividen claramente en **nivel de monitorización**(grabar lo que se ve) y**nivel de análisis** (grabar datos sin procesar, calibrar más tarde):

| Flujo de trabajo | Nivel | Qué guarda | GUI | CLI

|
| --- | --- | --- | --- | --- |
| **Captura**(fotogramas) | Análisis | Un grupo de fotogramas sincronizados por pasada; archivos por cámara en cada nivel de exportación seleccionado (sin procesar/sin debayering/radiancia/reflectancia/vista previa/índice) + archivo sidecar `.daq` |**Botón **Capturar todo** + Ajustes de captura | `lattice array-capture` |
| **Grabar vídeo de índice** | Supervisión | La composición del índice combinado en directo tal y como se muestra: 8 bits, resolución de vista previa, LUT integrada. Desde la CLI

, sin ninguna vista de la interfaz gráfica abierta, el backend renderiza el NDVI

de la matriz (o `--expr`) una vez alineada la matriz; una grabación que no escribe ningún fotograma falla | ● Grabar vídeo de índice (matrices combinadas) | `lattice array-record` |
| **Ráfaga sin procesar → crear vídeo**| Análisis | Fotogramas sin procesar del sensor a la velocidad máxima de captura + manifiesto + `.daq`; a continuación, reconstrucción fuera de línea en vídeo calibrado de radiancia, reflectancia o índice, sincronizado temporalmente con las lecturas del DAQ | ⦿ Grabar ráfaga sin procesar →**Crear vídeo** | `lattice array-burst` → `lattice array-build-video` |

Regla general: si los píxeles van a proporcionar *mediciones*, utiliza la captura o la ráfaga (calidad de análisis); si solo necesitas *ver o mostrar* lo que ha captado la matriz, graba el vídeo de índice (calidad de monitorización).

### Ajustes de captura (GUI)

<!-- SCREENSHOT-NEEDED: Capture Settings pane (gear next to Capture All) with a connected array: capture-mode buttons (Single/Continuous/Interval), the bulk export-type toggle row, the Fastest Capture toggle, and the per-array group card showing the Aligned checkbox and the "Record index video" / "Record raw burst" buttons. -->



El icono de engranaje junto a **Capturar todo** abre el panel de ajustes de captura (requiere un proyecto abierto, ya que las capturas se guardan en él):

* **Modo de captura**:**Único**(una pasada) /**Continuo**(consecutivo; limitado por un número de capturas, por defecto 1, o por una duración, por defecto 10 s) /**Intervalo** (timelapse: N capturas cada X intervalos hasta un total de Y; por defecto, 1 cada 5 s durante 1 minuto).
* **Tipos de exportación por cámara**: Raw, sin bayering, radiancia, reflectancia, vista previa, índice; todas las opciones aplicables están activadas por defecto. Las opciones de radiancia y reflectancia están ocultas para las cámaras con filtro RGB

; **«Reflectancia» solo aparece cuando la cámara tiene un sensor de luz DAQ** (propio o heredado de la matriz); «Índice» requiere una expresión de índice configurada.
* **Alineado**(por matriz, por defecto**activado**): adapta las exportaciones de los miembros al perfil de alineación de la matriz para que las exportaciones estén registradas por píxeles. El formato «Raw» siempre permanece sin deformar, pero lleva la transformación en los metadatos. Los archivos de los miembros de la matriz se guardan en la orientación nativa de cada cámara (cualquier ajuste de «Espejo» o «Rotación» se registra en el archivo, pero no se aplica).
* **Captura más rápida** (conmutable): solo «raw» + la lectura DAQ asignada + la composición de índice combinado libre, omitiendo los cálculos de calibración en el momento de la captura para obtener la máxima velocidad; reconstruye la radiancia, la reflectancia y el índice más tarde a partir del `.daq` guardado.
* Las selecciones se conservan con el proyecto. Se omiten las cámaras ocultas o en pausa.
* **Continuo** en matrices se ejecuta dentro del backend de Chloros

(novedad en la versión 1.3.0): cuando todas las cámaras de la lista pertenecen a una matriz, el backend ejecuta él mismo el bucle de captura, escribiendo un grupo mientras selecciona el siguiente, sin necesidad de una solicitud por cada captura. Una unidad llena detiene la ejecución («Detenido: unidad llena. Guardados N, F falló»). Consulte [Configuración y modos de captura](capture.md#capture-modes) para ver la nota de Microsoft Defender sobre las altas tasas de captura.

La CLI

equivalente (mismo punto final del backend, misma semántica):

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

La compresión TIFF

para las capturas es `deflate` (sin pérdidas, por defecto) o `none`; las tablas completas de indicadores, la estructura de la carpeta de capturas y las reglas de reprocesamiento se encuentran en la [Referencia de CLI

](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

## Emparejamiento de un sensor de luz DAQ

Las vistas previas con corrección de reflectancia e iluminación necesitan datos de luz descendente procedentes de un sensor DAQ (conectado en la pestaña **Sensores de luz**):

* La **fila de la matriz**de la barra lateral muestra un**botón «DAQ · activado/desactivado»**: aparece *activado* cuando se ha configurado un sensor de luz a nivel de matriz **o** cuando alguna de las cámaras de la matriz tiene el suyo propio; su información sobre herramientas indica exactamente qué sensor alimenta a cada cámara.
* Asigna la configuración para toda la matriz en los ajustes de la matriz → **Sensor de luz ambiental**→ menú desplegable**Sensor de luz**. La selección se mantiene con el proyecto, se aplica a todas las cámaras del conjunto y las cámaras individuales pueden seguir anulándola con su propio sensor.
* La línea de estado situada debajo muestra el estado en tiempo real: **Desactivado**→ «Esperando el primer espectro…» →**«Activo: todas las cámaras de la matriz tienen la iluminación corregida»** → o, si no ha llegado ningún espectro nuevo en los últimos 3 s, un aviso de datos caducados: se sigue utilizando la última lectura (las lecturas nunca caducan en la ruta de captura).

Con un sensor asignado: el tipo de exportación «Reflectancia» queda disponible, las vistas previas en directo se corrigen en cuanto a la iluminación, la exposición automática predictiva puede utilizar el espectro y cada captura de reflectancia escribe la lectura del DAQ realmente utilizada como un **archivo `.daq`** en la carpeta `daq/`, de modo que la captura se pueda volver a procesar más tarde. Junto a él, un archivo `.daq.cameras.json` identifica las cámaras cuyos fotogramas describe dicha lectura; al volver a procesar, Chloros

hace coincidir la lectura únicamente con imágenes de esas cámaras.

## Opciones de la CLI

** de `array-connect`

| Opción | Predeterminado | Descripción |
| --- | --- | --- |
| `--serials SN1,SN2,…` | Detección automática de todas las cámaras LATTICE (se necesitan ≥2) | **El primer número de serie es el del MAESTRO.** |
| `--line {Line0,Line2,Line3}` | `Line2` | Línea de sincronización GPIO. |
| `--target-fps F` | auto | Frecuencia de disparo del disparador maestro. |
| `--binning {1,2,4}` | auto | Agrupación de píxeles por hardware. «Auto» corresponde al sensor completo (sin agrupación) siempre que la frecuencia de fotogramas lo permita; «connect» nunca agrupa píxeles, solo para mantener la frecuencia. |
| `--packed {auto,on,off}` | `auto` | [Transporte empaquetado de 12 bits](#packed-12-bit-transport): `auto` solo comprime cuando resulta útil y el host puede permitirse la descompresión. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | automático | Anulación por parte de un experto del selector de nivel de sincronización. |
| `--wire-ceiling-mbps MB_PER_S` | detección automática | Presupuesto de ancho de banda del host en MB/s: el formato de la CLI

del campo **Presupuesto de ancho de banda**. Redúcelo si la matriz informa de tramas dañadas. Se guarda con el proyecto, por lo que al volver a conectarse más adelante se restablecerá. |
| `--no-recommend` | desactivado | Omite el paso de análisis de red. |
| `--no-ptp` | desactivado | Desactiva PTP (las marcas de tiempo entre cámaras ya no serán comparables). |

`lattice array-list`, `array-status` y `array-disconnect` gestionan la sesión persistente. La referencia completa de subcomandos, incluida la alineación (`align-calibrate` / `align-apply`) y las herramientas de red, se encuentra en la [Referencia de la CLI

§ chloros-cli lattice](../reference/cli-reference.md#chloros-cli-lattice); los equivalentes del SDK

(`connect_array`, `ArraySession`, `attach_array`, `analyze_array_network`) se encuentran en la [Referencia del SDK

](../reference/sdk-reference.md). Desde Python

, el presupuesto de transmisión es `connect_array(..., wire_ceiling_mbps=120)`, y la distinción entre datos corruptos en tiempo real y datos que nunca llegaron se encuentra en [`/api/camera/array/<id>/capability`](../reference/sdk-reference.md#array-health--which-subsystem-is-losing-frames).

# Ajustes de la cámara

La pestaña **Cámaras**es la superficie de control en directo de Chloros para las cámaras LATTICE: un área de visualización principal que muestra cada cámara conectada como un mosaico en directo, y una barra lateral que se desplaza entre tres páginas: la**lista de cámaras**, un**panel de ajustes**(ajustes por cámara, de matriz o de captura; uno a la vez) y la**Calculadora de índice**. Esta página documenta todos los controles de la lista de cámaras, el panel de ajustes por cámara y el panel de ajustes de matriz. Los modos de captura, la selección del tipo de exportación y el flujo «Capturar todo» se encuentran en la página complementaria [Ajustes y modos de captura](capture.md).

La pestaña «Cámaras» aparece en la barra lateral una vez que el backend Chloros está listo. Todos los controles que se muestran a continuación se comunican con el backend local a través de `127.0.0.1:5000`; los cambios se aplican a la cámara en directo de forma inmediata, salvo que se indique lo contrario.

## Tipos de cámara utilizados en esta página

Los controles se muestran u ocultan en función del tipo de cámara seleccionado. El manual utiliza estos términos a lo largo de todo el documento:

| Término | Significado | Canales de filtro |
| --- | --- | --- |
| **Cámara RGB** | LATTICE M3C con filtro FRGB (el modelo incluye `-FRGB`) | Red / Green / Blue |
| **multiespectral Bayer** | LATTICE M3C con FRGN, FOCN o FNGB | FRGN: Red / Green / NIR · FOCN: Orange / Cyan / NIR · FNGB: NIR / Green / Blue |
| **Mono (M3M)** | LATTICE M3M: un filtro de banda estrecha, una banda calibrada | Banda única |
| **Miembro de la matriz** | Una cámara conectada como parte de una matriz sincronizada (visualización combinada o por separado) | Según su filtro |

Las cámaras RGB se someten a un procesamiento fotométrico (balance de blancos, perfiles de color, gamma); las cámaras multiespectrales y monocromáticas pasan por la cadena radiométrica y se saltan los controles fotométricos. Los miembros de la matriz transfieren los ajustes a nivel de flujo (formato de píxeles, resolución, binning, activación, frecuencia de fotogramas) a la matriz; esas filas pasan a ser de solo lectura en el panel de cada cámara y se trasladan al panel de ajustes de la matriz.

## El área

<!-- SCREENSHOT-NEEDED: Cameras tab with 2+ cameras connected in grid view — live tiles visible with name and fps overlays, sidebar camera list open on the right. -->

principal de visualización Sin cámaras conectadas, el área de visualización muestra un mensaje de bienvenida **«Conecta una cámara para empezar»**con dos botones:**Conectar cámara**(verde, abre el cuadro de diálogo de conexión de una sola cámara) y**Conectar matriz** (azul, abre el cuadro de diálogo de conexión de la matriz). Los propios cuadros de diálogo de conexión se describen en [Conexión de cámaras](connecting.md); los conceptos relacionados con las matrices (sincronización, niveles, ancho de banda) se tratan en [Matrices multicámara](arrays.md). Al abrir un proyecto guardado que contenga cámaras, la pantalla de inicio muestra en su lugar un indicador de carga con el mensaje «Reabriendo N cámaras guardadas…» mientras Chloros restaura las transmisiones de la última sesión.

<!-- SCREENSHOT-NEEDED: Cameras tab empty state — the "Connect a camera to get started" splash with the green Connect Camera and blue Connect Array buttons. -->

### Barra superior

| Control | Qué hace |
| --- | --- |
| **Alternar modo de visualización**| Cambia entre la**vista en cuadrícula**(todos los mosaicos como celdas) y la**vista en lista** (matrices a ancho completo en la parte superior, UNA cámara activa debajo). Información sobre herramientas: «Cambiar a vista en cuadrícula» / «Cambiar a vista en lista». |
| **Bloqueo de cuadrícula**(candado) | Por defecto**bloqueado**: los mosaicos quedan fijos en su posición. Desbloquéalo para arrastrar y reordenar los mosaicos en cualquier espacio (se conservan los huecos). La cuadrícula se vuelve a bloquear automáticamente cada vez que se conecta una nueva cámara. Información sobre herramientas: «Desbloquear cuadrícula (habilitar arrastre de mosaicos)» / «Bloquear cuadrícula (fijar mosaicos en su sitio)». |
| Control deslizante **Zoom de la transmisión** | Tamaño de los mosaicos, desde 60 píxeles hasta el ancho total del contenedor. Las celdas mantienen una relación de aspecto de 4:3. Por debajo de los 200 píxeles de ancho de celda, el nombre y las superposiciones de fps se ocultan para mantener el mosaico limpio. |

### Mosaicos de la transmisión

Cada cámara genera un mosaico en directo compuesto; además, una cámara puede mostrar tres mosaicos en escala de grises **divididos por canal** (véase [Divisiones de canales](#display-overlays-drawn-over-the-live-feed)), y las matrices generan un mosaico combinado. El mosaico activo muestra un anillo de selección del color de la cámara (o de la matriz).

Al pasar el cursor por encima de un mosaico, aparece un botón de cierre **X**:

* Cerrar un mosaico **compuesto** mientras sus divisiones de canal permanecen visibles solo oculta el mosaico compuesto.
* Cerrar el **último mosaico visible de una cámara independiente** desconecta esa cámara.
* **Los mosaicos divididos que forman parte de un conjunto combinado nunca desconectan** la cámara; solo la ocultan.

Con la cuadrícula desbloqueada, arrastra cualquier mosaico a cualquier ranura; el diseño se guarda con el proyecto.

## Barra lateral — lista

<!-- SCREENSHOT-NEEDED: sidebar camera list pane showing a standalone camera row and an ARRAY group with indented member rows, the DAQ on/off pill visible on the array row, plus the Connect Camera / Connect Array / Capture All buttons at the top. -->

de cámaras La primera página de la barra lateral muestra todas las cámaras y matrices conectadas:

* **Conectar cámara**(verde) /**Conectar matriz** (azul, muestra «Detectando...» durante el escaneo). Ambas opciones están desactivadas mientras haya un cuadro de diálogo de conexión abierto.
* **Capturar todo** (rojo): captura todas las cámaras de la lista con los tipos de exportación seleccionados en «Configuración de captura». Requiere un proyecto abierto. Documentado al detalle en [Configuración y modos de captura](capture.md).
* **Icono de configuración de captura** (junto a «Capturar todo»): abre el [panel de configuración de captura](capture.md#the-capture-settings-pane). Desactivado si no hay ningún proyecto o durante la captura.

### Filas de cámaras

Cada fila de cámara muestra un borde codificado por colores (el color personalizado de la cámara), una etiqueta «CAM» —con una **M**azul (maestra) o una**S** verde (esclava) para los miembros de la matriz — y el nombre que se muestra. El nombre predeterminado es `LATTICE-MODEL (serial)`; se puede renombrar desde el panel de configuración de cada cámara. Botones de la fila:

| Botón | Efecto |
| --- | --- |
| **Ojo**| Alternar visibilidad. Las cámaras ocultas desaparecen de la cuadrícula y quedan**excluidas de «Capturar todo»**. |
| **Engranaje** | Abrir el panel de configuración de cada cámara (siguiente sección). |
| **Pausa / Reproducir**| Congela la vista previa en directo**solo en la pantalla**; la captura en segundo plano sigue ejecutándose. Las cámaras en pausa no pueden capturar. |
| **X** | Desconectar. La interfaz de usuario se actualiza inmediatamente (en el mejor de los casos); la desconexión en segundo plano puede tardar entre 10 y 30 s en completarse. |

### Filas de la matriz

Una fila de la matriz muestra una insignia «ARRAY» en el color de la matriz, el nombre de la matriz (que se puede renombrar en la configuración de la matriz) y una **pila**DAQ · activado/desactivado**:**activado** cuando el sensor de luz a nivel de matriz está configurado *o* cualquier miembro tiene un sensor por cámara; su información sobre herramientas indica exactamente qué sensor alimenta qué. Las cámaras miembros aparecen enumeradas, sangradas, debajo, en sus propias filas. Botones de la fila de la matriz: **ojo**(oculta/muestra TODOS los miembros a la vez),**engranaje**(panel de configuración de la matriz),**X**(desconecta toda la matriz).

El estado del sensor de luz (DLS) que se utiliza en las filas de la matriz y en el panel de configuración de la matriz tiene cuatro estados:**apagado**,**en espera**(aún no hay espectro),**activo**(ha llegado un espectro en los últimos 3 s) y**caducado**: no hay espectro nuevo en 3 s, pero la última lectura *sigue utilizándose* (las lecturas de DAQ nunca caducan en la ruta de captura).

Puedes arrastrar cámaras independientes y grupos de matrices completos unos sobre otros en la barra lateral para reordenar la lista; los miembros de la matriz no se pueden arrastrar de forma independiente.

## Panel de configuración por cámara

Se abre con el **engranaje** de la fila de una cámara. El panel se desliza sobre la lista de cámaras.

<!-- SCREENSHOT-NEEDED: per-camera settings pane, top portion — header with color swatch, camera name, rename pencil and close X; live histogram with the orange dashed AE-target line and green mean-luma line; the RGB per-band toggle button visible top-right of the histogram. -->

**Encabezado**: la**muestra de color**de la cámara (haz clic para abrir un selector de color nativo; establece el color del borde de la barra lateral y del anillo de selección de mosaicos), el**nombre**con un botón de**Renombrar**en forma de lápiz (si se guarda un nombre vacío, se restablece el valor predeterminado `MODEL (serial)`) y**×** para cerrar.

### Histograma en tiempo real

En la parte superior del panel hay un histograma de luminancia en tiempo real calculado a partir de la vista previa JPEG a ~8 Hz. La media está ponderada según el método Bayer —(R+2G+B)/4— para que coincida con la medición de exposición automática (AE) de la propia cámara.

* **Línea discontinua Orange**= el objetivo de exposición automática (AE).**Arrástrala horizontalmente para reajustar el objetivo**: al soltar se envía un comando, y al arrastrar se cambia el modo de objetivo de exposición automática a «Manual».
* **Línea continua Green** = la luminancia media real (lo que el AE está proporcionando actualmente).
* **Botón RGB** (arriba a la derecha): activa o desactiva los histogramas superpuestos por banda, coloreados según el filtro de la cámara (p. ej., en FRGN: gris NIR, verde, rojo). En las cámaras monocromáticas (M3M), el botón muestra «MONO» y está desactivado; el modo mono siempre muestra el histograma de luminancia de una sola banda.
* Las etiquetas del eje X siguen la profundidad de bits del sensor del formato de píxel actual: 0..255, 0..1023, 0..4095 o 0..65535.

### Filas

<!-- SCREENSHOT-NEEDED: per-camera settings info rows — Model, Radiometric Calibration "Active" badge with the tier/sha/date caption, Calibration Report Download button, Serial, Firmware row showing the "Up to date" state, IP, Temperature readout, Calibration Target checkbox, Light Sensor dropdown. -->

de información de la cámara | Fila | Comportamiento |
| --- | --- |
| **Modelo** | Solo lectura (p. ej., `LATT-M3C-L87-FRGN`). |
| **Calibración radiométrica**| Insignia**«Activa»**de Green con una leyenda que muestra el nivel de calibración, el hash, la fecha de calibración y la lista de bandas, cargados desde el paquete de calibración de la cámara (véase [Calibración radiométrica de fábrica](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration)).**Oculto para las cámaras RGB**: estas cuentan con una calibración fotométrica del balance de blancos, no con la radiancia por banda. |
| **Informe de calibración**| Botón**Descargar** — abre el certificado de calibración NIST específico para cada número de serie de la cámara en formato PDF en el visor de tu sistema operativo. Si el certificado aún no está almacenado en la caché, Chloros muestra un mensaje indicativo en su lugar. |
| **Número de serie** | Solo lectura. |
| **Firmware**| Muestra la versión actual y, a continuación, busca la versión disponible para este modelo (almacenada en caché por modelo; un conjunto de N cámaras comprueba el servidor una sola vez). Estados: «Comprobando…» → botón**«Actualizar a X»**→ «Actualizando…» → «Actualizado de A a B» / «Error: …» / «Omitido: …» /**«Al día»** en verde. Descripción emergente del botón de actualización: «Restablecimiento de fábrica + actualización de firmware + reprogramación de UserSet1. ~2–3 minutos; no desconectar». |
| **IP** | Solo lectura. |
| **Temperatura** | Solo lectura, se actualiza cada 3 s. Se vuelve naranja a partir de 65 °C y roja con un ⚠ a partir de 75 °C. |
| **Objetivo de calibración** | Habilita la detección de objetivos de reflectancia ArUco con una tabla de validación NDVI por panel situada debajo de la vista en directo (vista de lista). Solo para la sesión: siempre se abre desactivado. |
| Menú desplegable **Sensor de luz** | Asigna un sensor de luz DAQ (DAQ-E/M/U, de la lista de la pestaña «Sensores de luz») a esta cámara para la corrección de la iluminación por luz descendente (DLS) y la exposición automática predictiva. «Ninguno» borra la vinculación. Si no hay sensores conectados, el menú desplegable muestra «(no hay sensores conectados — abrir la pestaña DAQ)». La vinculación se guarda con el proyecto. |

### Exposición y ganancia

<!-- SCREENSHOT-NEEDED: per-camera Exposure & Gain section — Exposure (us) and Gain (dB) rows with Auto/Manual toggles, AE Target Brightness, AE Smoothing slider, AE Region of Interest row with the Aim button, and (on an array camera) AE Tune Speed and Highlight Protection rows. -->

Todas las entradas numéricas aquí utilizan controles deslizantes que se aceleran al mantenerlos pulsados: pulsar = ±1, mantener pulsado &gt;1,5 s = ±10, mantener pulsado &gt;3 s = ±100. El valor se envía a la cámara al soltar el botón.

| Control | Rango / opciones | Predeterminado | Se aplica a | Qué hace |
| --- | --- | --- | --- | --- |
| **Exposición (us)**| Mín./máx. en tiempo real de la cámara | Automático | Todo | Tiempo de exposición en microsegundos, con un conmutador**Automático/Manual**. Automático = exposición automática continua por parte de la cámara. |
| **Ganancia (dB)**| Mín. y máx. en tiempo real de la cámara (p. ej., hasta 48 dB) | Manual (desactivada) | Todos | Ganancia analógica/digital con su propio conmutador**Automático/Manual**. |
| **Brillo objetivo de AE**| 0–255 | 80, modo**Automático**| Todos (editable cuando AE o la ganancia automática están activadas) | El brillo al que apunta la AE. En**Automático**(el valor predeterminado), un controlador interno basado en el histograma selecciona el objetivo por sí mismo, manteniendo la exposición entre el 60 % y el 75 % del máximo del sensor. Al introducir un valor o arrastrar la línea naranja del histograma, se cambia a**Manual**. |
| **Suavizado de AE** | 0,5–40, paso de 0,1 | 8,0 | Todos | Amortiguación de AE. Descripción emergente: «Cuanto menor sea el valor, más rápido reacciona la AE (puede producir pulsaciones a altas velocidades de fotogramas). Cuanto mayor sea el valor, más suave y lenta será la respuesta». Los valores muy por debajo del predeterminado pueden provocar pulsaciones en la AE y desestabilizar la transmisión a altas velocidades de fotogramas; 8,0 es el valor predeterminado estable. |
| **Región de interés de AE**| Casilla de selección «Habilitar» + botón**Apuntar**| Desactivado | Todo | Cuando está activado, AE mide solo la región marcada con una línea discontinua verde en lugar de todo el fotograma. El botón**Apuntar** activa la función de «hacer clic para colocar» en la transmisión en directo: un clic centra una región en el 30 % del fotograma; al hacer clic y arrastrar se traza un rectángulo personalizado (mínimo 5 % × 5 %). «Apuntar» se desactiva automáticamente tras una colocación. La región se mapea de nuevo a las coordenadas nativas de la cámara según cualquier rotación o reflejo que hayas configurado, y se guarda con el proyecto. |
| **Velocidad de ajuste de AE** | 0,1–5, paso de 0,1 | 1,0 | Solo para miembros de la matriz | Velocidad a la que el objetivo de AE automático sigue los cambios de brillo de la escena; 1,0× vuelve a comprobar cada 2,5 s. |
| **Protección de altas luces** | Estricta (1 %) / Normal (5 %) / Flexible (15 %) | Estricta | Cámaras que permiten ajustar esta configuración | La cantidad de la imagen que puede recortarse a blanco antes de que el AE oscurezca la imagen. |

{% hint style="info" %}
**Requisitos de iluminación para cámaras multiespectrales Bayer (RGN / OCN / NGB):** la escena debe tener suficiente luz en los tres canales; de lo contrario, la calibración no funcionará correctamente, ya que una sola exposición del sensor cubre los tres espectros. Utiliza un sensor de luz DAQ para medir la iluminación, o bien utiliza el modo totalmente monocromo (M3M) para que cada banda tenga su propia exposición. Si una captura incumple este requisito, Chloros lo detecta y te avisa (notificación «unmix-clamp»).
{% endhint %}

### Formato de píxeles y resolución**Los miembros de la

<!-- SCREENSHOT-NEEDED: per-camera Pixel Format & Resolution section on a STANDALONE camera — Pixel Format, Resolution, and Binning dropdowns plus the Current WxH readout. A second capture on an array member showing the read-only "Set in array settings" state would also be useful. -->

matriz** muestran filas de solo lectura «Current» (formato + Ancho x Alto) y «Binning» con la nota «Configurado en los ajustes de la matriz»: reiniciar la transmisión en un miembro rompería la sincronización, por lo que se gestionan en el [panel de ajustes de la matriz](#array-settings-pane).**Las cámaras independientes** tienen:

| Control | Opciones | Qué hace |
| --- | --- | --- |
| **Formato de píxeles** | BayerRG8 / BayerRG10 / BayerRG12 / BayerRG16 / Mono8 | Formato de píxeles del sensor (profundidad de bits). |
| **Resolución** | Completa / Media / Un cuarto | En relación con el binning actual: Completa = 2048/N × 1536/N para un binning N×N. |
| **Agrupación** | 1x1 (ninguna) / 2x2 / 4x4 | Agrupación N×N por hardware: los valores más altos reducen la resolución, pero aumentan la relación señal-ruido (SNR) y la velocidad de fotogramas. Al cambiarlo, se reinicia la transmisión y se restablecen las regiones de interés (ROI) al nuevo campo de visión completo. |
| **Actual** | solo lectura | El ancho × alto real y el desplazamiento (x, y) que están activos. |

### Vista previa en directo

Todo lo que aparece en esta sección es **solo para la visualización**—cambia lo que ves en la transmisión en directo, mientras que las capturas guardadas se mantienen lineales y sin modificar— con una excepción:**La viñeta** es radiométrica y también afecta a las exportaciones (como se indica más abajo).

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on an RGB (FRGB) camera — Render resolution, White Balance mode, Gamma, Denoise, Sharpness, Vignette, Color Profile dropdown open showing Raw/Linear/Natural/Enhanced/Custom Temperature, Saturation, Contrast, Mirror H/V and Rotation. -->

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on a Bayer multispectral (e.g. FRGN) camera — showing the Index row with its gear button (and the absence of the RGB-only White Balance / Gamma / Color Profile / Saturation / Contrast rows). -->

| Control | Rango / opciones | Predeterminado | Se aplica a | Qué hace |
| --- | --- | --- | --- | --- |
| **Resolución de renderizado** | 360p (más rápida) / 480p / 720p / 1080p / Resolución nativa del sensor (más lenta) | 720p | Todo | La altura a la que el backend ejecuta la cadena de vista previa radiométrica. Un valor más bajo aumenta la frecuencia de fotogramas sin modificar el campo de visión. |
| **Índice**| Casilla de selección «Habilitar» + icono de engranaje | Desactivado | Solo multiespectral Bayer,**no** para miembros de matriz combinada | Vista previa en directo del índice de vegetación. El icono de engranaje abre la [Calculadora de índices](#index-calculator-pane) compartida, precargada con las bandas naturales del filtro de la cámara (p. ej., `Red_660_RGN`, `Green_550_RGN`, `NIR_850_RGN`). La expresión personalizada más la LUT (activada/desactivada, nivel por defecto 3, mínimo por defecto 0,2, máximo por defecto 1) se calcula en cada fotograma de vista previa. Los elementos de la matriz combinada ocultan esta fila; la matriz posee un índice compartido. |
| **Balance de blancos** | Desactivado / Una vez / Continuo + un botón de recaptura | Continuo | Solo RGB | Balance de blancos en tiempo real. El botón de actualización vuelve a capturar el balance de blancos a partir del espectro DLS actual (desactivado cuando el modo está en «Desactivado»). |
| **Gamma** | Activado / Desactivado | Activado | Solo RGB | Muestra la gamma (LUT γ = 2,2) en la vista previa en directo. Las capturas guardadas se mantienen lineales. |
| **Reducción de ruido** | Casilla de selección + intensidad 0–100 | Desactivado / 50 | Todos (por cámara, incluso dentro de matrices) | Filtro bilateral en la vista previa en directo. Cuanto mayor sea el valor, más suave será la imagen, pero con menos detalle. |
| **Nitidez** | Casilla de selección + intensidad 0–100 | Desactivado / 30 | Todo | Máscara de desenfoque en la vista previa en directo, aplicada en último lugar. Puede amplificar el ruido. Solo en la vista previa. |
| **Viñeta**| Casilla de selección + intensidad 0–100 | Desactivado / 0 | Todo | Eliminación manual de la viñeta residual (aclara las esquinas), superpuesta a la estimación de viñeta inteligente de la matriz.**Radiométrico: afecta a la vista en directo Y a las exportaciones**, a diferencia de «Reducción de ruido» y «Nitidez». |
| **Perfil de color** | Raw / Lineal / Natural / Mejorado / Temperatura personalizada | Natural | Solo RGB | Véase más abajo. |
| **Temperatura de color** | 2000–10 000 K, en pasos de 100 | 5500 K | Solo RGB, perfil de temperatura personalizada | Fija el balance de blancos en una temperatura de color correlacionada fija (se ignora la entrada DLS). El último valor en kelvin seleccionado se recuerda al cambiar de perfil. |
| **Saturación** | 0–200 (100 = neutro) | 100 | Solo RGB | Saturación HSV en la vista previa en directo. |
| **Contraste** | 0–200 (100 = neutro) | 100 | Solo RGB | Contraste lineal en torno alen la vista previa en directo. |
| **Invertir H / Invertir V** | Casillas de selección | Desactivado | Todos | Invertir la vista previa horizontalmente / verticalmente. |
| **Rotación**| 0° / 90° / 180° / 270° | 0° | Todas | Gira la vista previa. La orientación se aplica al final de la cadena de vista previa del backend:**las capturas guardadas conservan la orientación nativa de la cámara**, y las vistas compuestas en matriz la ignoran. |**Semántica de los perfiles de color** (cámaras RGB):

* **Raw** — omite por completo la cadena de procesamiento.
* **Lineal** — señal oscura + campo plano + balance de blancos; sin matriz de color ni gamma.
* **Natural** *(predeterminado)* — lineal más la matriz de corrección de color medida y una curva de tono adaptativa a la escena.
* **Mejorado**: «Natural» más intensidad cromática y contraste local CLAHE. El coste adicional se aplica**únicamente a la vista previa en directo**; las capturas guardadas siempre obtienen el acabado completo, independientemente del perfil.
* **Temperatura personalizada** — «Natural» con el balance de blancos fijado en el valor en Kelvin que elijas.

{% hint style="warning" %}
Para los perfiles «Natural», «Mejorado» y «Temperatura personalizada», el panel muestra una nota sobre el tono: los fotogramas se aclaran según su propia escena, por lo que las imágenes guardadas *en pantalla* no son comparables entre sí fotograma a fotograma. **Exporta la radiancia o la reflectancia para realizar mediciones.**
{% endhint %}

### Superposiciones en pantalla (dibujadas sobre la transmisión en directo)

Son exclusivas de la interfaz de usuario: se superponen al vídeo, sin afectar nunca a la transmisión ni a las capturas.

<!-- SCREENSHOT-NEEDED: a live feed tile with overlays active — zebra stripes on clipped sky, 3x3 grid, focus peaking in the default orange, and the on-feed histogram strip; the overlays section of the settings pane visible alongside. -->

| Superposición | Controles | Predeterminado | Qué hace |
| --- | --- | --- | --- |
| **Zebra** | Casilla de selección + umbral 200–255 | Desactivado / 250 | Rayas diagonales magenta en los píxeles recortados. |
| **Punto de mira** | Casilla de selección | Desactivado | Marca en el centro del fotograma. |
| **Cuadrícula** | Desactivado / 3 × 3 / 9 × 9 | Desactivado | Cuadrícula de composición. |
| **Histograma** | Casilla de selección + ancho 0,10–0,90 del fotograma | Desactivado / 0,25 | Una franja de histograma sobre la transmisión. |
| **Pico de enfoque** | Casilla de selección + umbral 20–200 + muestra de color | Desactivado / 80 / `#ff5722` | Resaltado de bordes Sobel para enfocar. |
| **Divisiones de canales** | «Mostrar divisiones (Red / Green / NIR)» / Botón «Ocultar divisiones» | Oculto | Añade tres mosaicos independientes en escala de grises por canal junto a la imagen compuesta (la etiqueta del botón sigue los canales de filtro de la cámara). Cada mosaico dividido se puede arrastrar y comparte el color del borde de la cámara. No disponible en cámaras monocromáticas. Se guarda con el proyecto. |

### Medidor puntual

* Casilla de selección **Hacer clic para muestrear**: haz clic en la imagen en directo para muestrear un único píxel (se marca con una retícula en forma de cruz), o haz clic y arrastra una región para obtener una media de píxeles.**Borrar**elimina la muestra y la retícula. Se excluye mutuamente con el modo**Apuntar** de AE-ROI.
* Menú desplegable **Mostrar**:**Raw (profundidad de bits)**—valores digitales nativos con la profundidad de bits del sensor (p. ej., 12 bits → 0..4095)— o**Pantalla (8 bits)** (por defecto). Cuando hay un índice en directo activo, la pantalla muestra en su lugar el valor del índice calculado (p. ej., NDVI).
* El panel de lectura muestra las coordenadas de los píxeles, el tamaño del fotograma, el formato de píxeles, la profundidad de bits y una tabla de canales (Canal / Valor / %) con etiquetas de banda y longitudes de onda; se promedian los pares verdes de Bayer; las muestras de región muestran «N px avg».

El estado del medidor puntual es exclusivo de la sesión.

<!-- SCREENSHOT-NEEDED: Spot Meter in use — reticle placed on the live feed, readout panel showing the per-channel value table with band wavelength labels. -->

### Exposición automática predictiva (basada en DLS)

Esta sección solo aparece cuando **hay al menos un sensor de luz DAQ conectado**: el solucionador necesita un espectro descendente en tiempo real para funcionar.

<!-- SCREENSHOT-NEEDED: Predictive Auto-Exposure (DLS-driven) section with a DAQ connected — Enable checkbox, Smoothing (α) slider at 0.30, and the "Recalibrate ρ" button. -->

| Control | Rango | Predeterminado | Qué hace |
| --- | --- | --- | --- |
| **Habilitar** | Casilla de selección | Activado (cámaras independientes) | Un solucionador de forma cerrada utiliza el espectro del DLS junto con los escalares del paquete de calibración de la cámara para situar la banda más brillante cerca de la saturación, al tiempo que mantiene la banda más tenue por encima del umbral de SNR — una sola escritura de exposición por resolución, sin bucle de estabilización. Diseñado para time-lapse con energía solar, donde cada toma debe estar correctamente expuesta. El backend recurre de forma silenciosa a la exposición automática reactiva (AE) siempre que la lectura del DLS esté obsoleta o falte, o cuando no se haya cargado el paquete de calibración. |
| **Suavizado (α)** | 0,05–1,0, paso de 0,05 | 0,3 | Suavizado de soluciones predictivas sucesivas (cuanto menor sea el valor, mayor será el suavizado). |
| **Reflectancia de la escena**| Botón**Recalibrar ρ** | — | Reestima el factor de reflectancia de la escena que utiliza el solucionador. |

{% hint style="info" %}
**La conexión en matriz desactiva la AE predictiva por defecto** — en el caso de las matrices, la AE inteligente de Chloros, junto con la exposición automática del lado de la cámara, se encarga de la exposición (con protección contra la saturación), y la estimación única de la reflectancia de la escena de la AE predictiva no es segura en escenas mixtas. Puede volver a habilitarla por cámara aquí si desea específicamente una exposición radiométrica controlada por DLS.
{% endhint %}

**Límite máximo de exposición controlado por DAQ y AE fijado a la luz incidente.**Independientemente de la casilla de selección anterior, cuando se asigna un sensor de luz DAQ a una cámara RGB, Chloros calcula —a partir de la irradiancia descendente absoluta medida — la exposición máxima × ganancia a la que una superficie con un 100 % de reflectancia permanece por debajo del recorte, y la aplica como**límite máximo**en la exposición automática. Mientras el límite máximo está activo, la cámara se encuentra en**modo fijado a la luz incidente**: funciona en bucle abierto con la exposición medida en la luz incidente y una ganancia de 0 dB; la exposición sigue la luz medida, no el contenido de la escena. Dado que el límite máximo solo puede acortar la exposición, no puede provocar por sí mismo el recorte. El límite máximo se desactiva automáticamente —y se reanuda la exposición automática (AE) normal de la escena— siempre que la lectura del DAQ falte, esté desactualizada (&gt;30 s) o sea oscura, o si ≥15 % del fotograma se recorta con la exposición fijada (lo que significa que el sensor y la cámara perciben una iluminación diferente). No hay ningún conmutador en la interfaz gráfica de usuario; este es el comportamiento estándar siempre que una cámara RGB tenga un DAQ vinculado.

### Los miembros de la matriz de adquisición y disparo

<!-- SCREENSHOT-NEEDED: Acquisition & Trigger section on a standalone camera — Trigger Mode, Trigger Source, and the Frame Rate row in Auto mode showing live fps; ideally a second capture on an array member showing the read-only Role/Sync Line/Peers rows. -->

muestran además un **Rol**de solo lectura (Maestro en azul / Esclavo en verde),**Línea de sincronización**y**Pares**.

| Control | Opciones | Predeterminado | Notas |
| --- | --- | --- | --- |
| **Modo de disparo** | Desactivado / Activado | Activado | Desactivado para los miembros de la matriz (la matriz gestiona el disparo). |
| **Fuente de disparo** | Software / Línea 0 (M8) / Línea 1 / Línea 2 | Línea 0 | Oculta mientras el modo de disparo está desactivado; desactivada para los miembros del conjunto. La Línea 0 es la entrada de disparo externa optoaislada M8. |
| **Velocidad de fotogramas**| Automática / Manual + valor | Automática |**Automática**: el límite de velocidad de fotogramas de la cámara está desactivado; la exposición determina los fps, y el cuadro muestra la velocidad real en directo.**Manual**: se limita el número de fps con un control deslizante (de 1 hasta el máximo limitado por el ancho de banda), partiendo de la frecuencia real actual. Los miembros de la matriz ven un valor de solo lectura «N fps (en directo)» con la indicación «Configurado en los ajustes de la matriz». |

### Red / Transporte

| Fila | Comportamiento |
| --- | --- |
| **Tamaño del paquete**| 1500 (estándar) / 9000 (jumbo) — por defecto**jumbo**. |
| **Ancho de banda** | Límite de ancho de banda del enlace de solo lectura en MB/s. El backend reequilibra este valor entre todas las cámaras conectadas cada vez que se establece o se interrumpe la conexión. |
| **Gestión del búfer** | Modo de gestión del búfer de solo lectura. |

### Captura

El panel termina con un botón **«Abrir configuración de captura…»** que lleva al [panel de configuración de captura](capture.md#the-capture-settings-pane) (desactivado hasta que se abra un proyecto — «Crea o abre un proyecto para guardar capturas»). Si la cámara está oculta o en pausa, una indicación le recuerda que debe mostrarla o reanudarla antes de realizar la captura.

## Panel de configuración de la matriz

Se abre con el **engranaje**de una fila de la MATRIZ. Encabezado: nombre de la matriz con un lápiz para cambiar el nombre y**×** para cerrar. Las secciones siguientes marcadas como *solo combinadas* solo aparecen para matrices conectadas en modo de visualización combinada.

<!-- SCREENSHOT-NEEDED: array settings pane, top portion — array name header, Sync section (Master/Slaves/Sync Line), and Ambient Light Sensor section with the Light Sensor dropdown and the green "Active — all cameras in the array are illumination-corrected" status line. -->

### Sincronización

Filas **Maestro**,**Esclavos**y**Línea de sincronización** de solo lectura.

### Sensor de luz ambiental

Aparece tanto para matrices combinadas como separadas:

* Casilla de selección **Objetivo de calibración** — «Detectar el objetivo ArUco MAPIR y validar NDVI frente a la LUT de reflectancia del panel»; controla la superposición del objetivo y la tabla de validación del mosaico combinado.
* Menú desplegable **Sensor de luz**: vincula un DAQ a toda la matriz. La selección se aplica de forma inmediata, se propaga al menú desplegable «Sensor de luz» de cada cámara de la matriz (aunque se puede anular la configuración por cámara) y comienza a enviar espectros a la matriz.
* Línea de **Estado** en tiempo real: Desactivado · «Esperando el primer espectro…» · «Activo: todas las cámaras de la matriz tienen la iluminación corregida» · «No hay ningún espectro nuevo en los últimos 3 s: se sigue utilizando la última lectura (sin tiempo de espera por datos caducados)…».
* Nota en el panel: «Corrección radiométrica para toda la matriz. Los ajustes por cámara anulan esta configuración».

### Captura — ajustes uniformes del sensor *(solo combinados)*

Estos ajustes se aplican de forma uniforme a todos los miembros (los cambios por miembro romperían la sincronización). Las modificaciones se preparan y se aplican juntas.

<!-- SCREENSHOT-NEEDED: array settings Capture section — Pixel Format, Binning, Resolution preset, the ROI crop W/H/X/Y fields with the "max WxH" hint and Reset button, Trigger Rate row in Auto showing the derived fps, and the Apply/Cancel buttons; ideally with the live orange crop-preview box visible on the array tile. -->

| Control | Opciones / rango | Qué hace |
| --- | --- | --- |
| **Formato de píxel** | BayerRG8 / BayerRG10 / BayerRG12 / BayerRG16 / Mono8 | Formato de sensor uniforme para todos los miembros. |
| **Agrupación de píxeles** | 1x1 / 2x2 / 4x4 | Agrupación de píxeles por hardware: mantiene el campo de visión completo al tiempo que mejora la relación señal-ruido (SNR) y la velocidad de fotogramas. Al cambiarlo, los campos de la región de interés (ROI) se restablecen al nuevo campo de visión completo. |
| Preajuste de **resolución** | Completa / Mitad / Cuarto | Relativo al binning; rellena los campos de la región de interés (ROI) con un recorte centrado. |
| **Recorte de la región de interés (px)**| Campos numéricos W / H / X / Y | Recorte del sensor. La anchura y la altura se ajustan a múltiplos de 16 (mínimo 64); los desplazamientos se ajustan a múltiplos de 4. Una indicación de «máx. WxH» muestra el límite máximo y**Restablecer** vuelve al campo de visión completo. Durante la edición, se dibuja un recuadro naranja de vista previa del recorte en tiempo real sobre el mosaico de la matriz (incluido un esquema del sensor completo cuando se amplía el recorte hacia fuera). |
| **Frecuencia de disparo**| Alternancia entre Automático / Manual + fps 0,5–10, paso de 0,5 |**Automático**(por defecto): el backend calcula la frecuencia de disparo a partir de la resolución y el ancho de banda; la entrada está desactivada y muestra el valor calculado.**Manual**: fija el valor al pulsar Aplicar. |

Nota en el panel: «Los cambios de formato o resolución reinician brevemente todas las cámaras. La frecuencia de disparo se aplica en tiempo real». Los botones **Aplicar / Cancelar** se encuentran en la parte inferior del panel.

### Alineación (co-registro) *(solo combinada)*

<!-- SCREENSHOT-NEEDED: array settings Alignment section after a successful calibration — green "RMS x.xx px" residual pill, "✓ All cameras aligned (N)" summary, the per-camera table with px error / match count / NCC columns, the Recalibrate alignment button and the "Auto-expose cameras for alignment" checkbox. -->



* Casilla **Residual**: «RMS x,xx px» — verde por debajo de 1 px, ámbar por debajo de 3 px, rojo en los demás casos o si alguna cámara ha fallado; «sin perfil» antes de la primera resolución.
* Línea de resumen: «✓ Todas las cámaras alineadas (N)» / «⚠ p/N cámaras alineadas —  <serial (filter)="">fallo» / «Recorte activo — Recalibrar para alinear (utiliza el sensor completo)» / «Esperando a que se estabilice la exposición…».
* Tabla por cámara: cámara (últimos 4 dígitos del número de serie + filtro), error de reproyección en píxeles con recuento de coincidencias («ref» para la cámara maestra) y la puntuación de correlación cruzada normalizada de solapamiento frente al umbral de aprobación de 0,35.
* Botón **Recalibrar alineación** (muestra «Calibrar alineación» antes del primer perfil): vuelve a ejecutar el registro conjunto con fotogramas nuevos.
* Casilla de selección **«Exposición automática de las cámaras para la alineación»** (marcada por defecto): aumenta temporalmente el brillo de las cámaras oscuras o sin contraste (primero la exposición y luego la ganancia) para que tengan textura que coincida, y luego restaura la exposición automática.

La vista previa combinada se alinea automáticamente al abrirla; recalibrar si ha cambiado el enfoque o la profundidad de la escena. La alineación está **diseñada para ser válida solo durante la sesión** — nunca se guarda en un perfil, ya que depende de la distancia de la escena en ese momento. Las capturas se pueden seguir exportando con registro de píxeles (véase [Exportaciones alineadas](capture.md#per-array-controls)).

### Viñeta inteligente

* Casilla de selección **Habilitar corrección**: aplica la estimación de viñeta por cámara a la cadena radiométrica (en tiempo real**y** en las exportaciones).
* **Calibrar desde la vista actual**: apunta primero la matriz hacia un objetivo uniforme (pantalla plana, pared o cielo); cada cámara se aplana individualmente y el estado indica una ganancia de planitud de «n/N cámaras · −x,x %».**Borrar** elimina la estimación.
* Ajusta con precisión cada cámara mediante el control deslizante **Vignette** de cada cámara en [Vista previa en directo](#live-preview).

### Vista previa en directo *(solo combinada)** **Índice**: marca la casilla + rueda dentada — abre la [Calculadora de índice](#index-calculator-pane) compartida con bandas dibujadas a partir de**todas** las cámaras del grupo. Una línea de vista previa de la expresión situada debajo muestra la expresión actual («No hay expresión definida — abre la calculadora para crear una»), que se actualiza cada segundo.
* **Resolución de renderizado**(los mismos ajustes preestablecidos que por cámara, por defecto 720p): la altura de la transmisión en directo**y** el tamaño de exportación del compuesto guardado. Nota en el panel: «Vista previa + tamaño del compuesto guardado. Las imágenes por cámara siempre se exportan a resolución completa».

### Capas de visualización *(solo combinadas)** Casilla de selección **Habilitar** (desactivada por defecto — la cámara principal se muestra directamente; activada = composición en capas).
* Menús desplegables **Primer plano**/**Fondo**: cada cámara miembro (por nombre) o**Índice**. Cuando el primer plano es «Índice», los píxeles fuera de los valores mínimo/máximo de la LUT muestran la capa de fondo.

### Vista dividida *(solo combinada)*

**«Mostrar cámaras de los miembros»**: un botón**Dividir / Ocultar cámaras de los miembros** que añade la señal en directo de cada miembro como mosaicos independientes en la cuadrícula junto a la composición. Los mosaicos leen el búfer de fotogramas existente de la matriz (sin conexión adicional de cámara). Solo en vista de cuadrícula; se guarda por matriz con el proyecto.

### Funcionalidades

Un panel de solo lectura que se actualiza cada 5 s:

* **Etiqueta de nivel**: «Captura simultánea» (verde) · «Captura simultánea (emisión escalonada FTD)» (verde) · «Captura escalonada (desviación de 100 ms)» (ámbar) · «Configuración demasiado grande» (rojo).
* **Estado del fotograma**: «x,xx % incompleto» — verde por debajo del 1 %, ámbar por debajo del 5 %, rojo a partir del 5 %.
* **Línea de enlace**: «NIC {mbps} Mbps - sostenido {MB/s} MB/s».

Este es el presupuesto de ancho de banda en tiempo real de la matriz. Para conocer los fps subyacentes y el modelo de red —y qué cambios hay que realizar cuando el nivel pasa a ámbar o rojo—, consulta [Matrices multicámara](arrays.md) y la [Referencia CLI](../reference/cli-reference.md).



<!-- SCREENSHOT-NEEDED: array settings Capabilities panel showing a green "Simultaneous capture" tier, the frame-health percentage, and the NIC/sustained-throughput line. -->## Panel «Calculadora de índice»

La tercera página de la barra lateral, compartida por el panel de configuración de índice por cámara y el de la matriz combinada (uno a la vez; el encabezado dice «Calculadora de índice — <camera name="">» o «Calculadora de índice —<array name="">

»). Recibe la lista de bandas (las bandas naturales del filtro de la cámara o todas las bandas de los miembros de la matriz), la expresión actual y la configuración de la LUT (activada/desactivada, nivel —por defecto 3—, mínimo —por defecto 0,2— y máximo —por defecto 1—), además de un histograma de índice en tiempo real. **«Aplicar»** confirma la expresión; los cambios en la LUT se aplican en tiempo real a la vista previa.

<!-- SCREENSHOT-NEEDED: Index Calculator pane open for a combined array — band buttons for all member cameras, an NDVI-style expression in the editor, LUT controls, and the live index histogram. -->

## Ajustes por cámara frente a ajustes gestionados por la matriz

Referencia rápida de qué se encuentra dónde cuando una cámara forma parte de una matriz:

| Gestionado por la matriz (solo lectura en el panel de la cámara) | Sigue siendo por cámara dentro de una matriz |
| --- | --- |
| Formato de píxeles, resolución, agrupación de píxeles | Exposición automática (exposición, ganancia, objetivo, suavizado, ROI) |
| Modo/fuente de disparo, frecuencia de fotogramas | Reducción de ruido, nitidez, viñeteado |
| | Orientación (espejo/rotación), superposiciones de pantalla, medidor puntual |
| | Índice (matrices con visualización independiente), vinculación del sensor de luz |

Otros comportamientos transversales:

* **Visualización combinada frente a separada**: se elige al conectar la matriz; combinada = un mosaico compuesto alineado (las cámaras de la matriz solo transmiten a través de «Split View»); separada = cada cámara renderiza su propio mosaico sincronizado. Una cámara nunca muestra a la vez una transmisión independiente y un mosaico de la matriz.
* **Reconexión automática**: al abrir un proyecto guardado, se restauran sus cámaras y matrices y se vuelven a aplicar todos los ajustes guardados al backend antes de que se reanuden las transmisiones.
* **Control de captura**: las cámaras ocultas o en pausa quedan excluidas de «Capturar todo»; una matriz solo queda totalmente bloqueada cuando TODOS los miembros están ocultos o en pausa. Véase [Ajustes y modos de captura](capture.md).

## Cómo se conservan los ajustes

El estado de la pestaña de la cámara se guarda **con el proyecto**, no en el navegador:

* Cada cambio reactivo captura una instantánea de las cámaras y las matrices en el `cameras.json` del proyecto (con un retardo de 500 ms). Esto incluye los nombres y colores de las cámaras, ajustes de exposición/ganancia/AE, formato de píxeles/resolución/agrupación de píxeles, frecuencia de disparo, ajustes de vista previa (resolución de renderizado, eliminación de ruido, nitidez, viñeta, perfil de color, saturación/contraste), orientación, superposiciones, divisiones de canales, configuración del índice, ajustes de AE predictiva, ROI de AE, nombres de matrices, modo de visualización, ajustes de captura de matrices (incluida la posición de recorte del ROI) y el bloque de cuadrícula (zoom de la señal, modo de visualización, bloqueo de cuadrícula, orden manual de mosaicos, cámaras ocultas, mosaicos cerrados, cámara activa).
* Las asignaciones de sensores de luz se guardan en el archivo `sensors.json` del proyecto.
* Al volver a abrir el proyecto, se vuelve a conectar el hardware y se vuelven a aplicar todos estos ajustes.
* **Si no hay ningún proyecto abierto = solo sesión**: sin proyecto, nada se conserva al cerrar Chloros.
* Solo sesión, independientemente del proyecto: estado de pausa, muestras del medidor puntual, la casilla de verificación «Objetivo de calibración» por cámara (que siempre se abre desactivada) y el perfil de alineación de la matriz (recalculado por sesión según el diseño).
* Una excepción: las selecciones de exportación de **Ajustes de captura** y el modo de captura se conservan por proyecto en el almacenamiento local de la aplicación, en lugar de en `cameras.json`; véase [Ajustes y modos de captura](capture.md).</array></camera></serial>

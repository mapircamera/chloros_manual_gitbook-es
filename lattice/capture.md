# Ajustes y modos de captura

La captura en la pestaña «Cámaras» se controla mediante un botón rojo **Capturar todo**y un panel de**Ajustes de captura** que determina el resultado de dicho botón: qué cámaras participan, qué tipos de exportación guarda cada cámara y si el obturador se dispara una vez, de forma continua o a intervalos. Esta página documenta todo el proceso: la configuración, la captura en sí, dónde se guardan los archivos en el disco y cómo volver a procesarlos posteriormente para obtener productos calibrados. Los controles de las cámaras y las matrices se encuentran en [Ajustes de cámara](camera-settings.md).

{% hint style="info" %}
**Las capturas requieren un proyecto abierto.** Las opciones «Capturar todo» y el icono de configuración de captura permanecen desactivadas hasta que se abra un proyecto («Crea o abre un proyecto para guardar las capturas»). Cada captura se guarda en la carpeta del proyecto en `captures/`.
{% endhint %}

## El panel «Configuración de captura»

Ábrelo con el **icono de engranaje situado junto a «Capturar todo»**en la lista de cámaras de la barra lateral, o con el botón**«Abrir configuración de captura…»** situado en la parte inferior de cualquier panel de configuración específico de cada cámara. El encabezado indica «Configuración de captura» y cuenta con un botón ← para volver atrás.

<!-- SCREENSHOT-NEEDED: the full Capture Settings pane — Single/Continuous/Interval mode buttons at top, the bulk export-type toggle rows (All Raw … All Index), the orange Fastest Capture toggle, an array group card with the Aligned checkbox and Record buttons, and an expanded per-camera row showing per-type checkboxes. -->



Las selecciones que realices aquí —cámaras incluidas, casillas de selección por tipo y el modo de captura— se guardan **por proyecto** y se restauran al volver a abrirlo.

### Modos de captura

Tres botones de modo en la parte superior del panel:

| Modo | Qué hace | Subajustes (por defecto) |
| --- | --- | --- |
| **Única** *(por defecto)* | Una captura en todas las cámaras seleccionadas. | — |
| **Continuo**| Capturas consecutivas hasta que se cumpla una condición de parada. | Parada por**número de capturas** (por defecto 1) *o* **duración de la captura** (por defecto 10 s; unidades: segundos / minutos / horas / días). |
| **Intervalo**(timelapse) | Ráfagas programadas por temporizador. |**Capturas / intervalo**(por defecto 1) ·**Cada**N unidades (por defecto 5 s) ·**Durante** N unidades (por defecto 1 m). |

En los modos Continuo o Intervalo, el botón «Capturar todo» se convierte en un botón «**Detener (N)**» mientras se está ejecutando, contando las capturas a medida que se realizan. Una unidad llena detiene la ejecución (véase [El flujo de «Capturar todo»](#the-capture-all-flow)).**El modo «Continuo» en matrices se ejecuta en el backend (novedad en la versión 1.3.0).** Cuando todas las cámaras de la lista pertenecen a una matriz (cámaras independientes ocultas o excluidas), el modo «Continuo» ya no envía una solicitud por cada captura: el backend de Chloros ejecuta él mismo el bucle de captura, escribiendo un grupo sincronizado mientras selecciona el siguiente, y el botón consulta su recuento. Si hay cámaras independientes en la lista, Chloros captura paso a paso como antes.

{% hint style="warning" %}
**Captura rápida en Windows: mantén el backend de Chloros excluido de Microsoft Defender.** El análisis en tiempo real de cada archivo que escribe una captura de alta velocidad supone aproximadamente un 30 % de la velocidad de captura en las pruebas de MAPIR (cientos de archivos por segundo en una zona de interés pequeña). El instalador de Windows ofrece la opción **«Excluir el backend de Chloros del análisis de Microsoft Defender (captura de cámara mucho más rápida; recomendado)»**, marcada por defecto, y elimina la exclusión al desinstalar el programa. Si la has desmarcado, vuelve a ejecutar el instalador con la opción marcada, o añade una exclusión de procesos de Defender para `chloros-backend.exe` (en la carpeta `backend` de la carpeta de instalación de Chloros). Cuando Defender utiliza un núcleo de CPU o más durante la captura continua de una matriz, el backend de Chloros lo anota una vez en su registro, con estas mismas dos soluciones.
{% endhint %}

<!-- SCREENSHOT-NEEDED: the capture-mode area of Capture Settings with Interval selected — showing the "Captures / interval", "Every N (unit)" and "For N (unit)" rows with their defaults (1, 5 s, 1 m). -->




### Selección de cámaras y tipos de exportación

El texto de ayuda del panel lo resume así: elige qué cámaras y tipos de exportación genera la opción «Capturar todo»; todo está activado por defecto y las opciones se guardan con este proyecto.

* Los botones **Seleccionar todo / No seleccionar nada** activan o desactivan a la vez las casillas de selección de todas las cámaras.
* **Botones para cambiar el tipo de exportación de forma masiva**(dos filas de botones):**Todo en formato RAW / Todo sin bayering / Toda vista previa / Toda radiancia / Toda reflectancia / Todo índice**. Cada uno tiene tres estados de color: verde ✓ = activado para todas las cámaras que lo admiten, ámbar – = activado para algunas, gris = ninguna. Un conmutador queda desactivado cuando ninguna cámara conectada admite ese tipo. Todos ellos aparecen en gris mientras la opción «Captura más rápida» está activada.
* **Filas por cámara**: una casilla de selección, además de una lista expandible (▸/▾) de los tipos de exportación aplicables a esa cámara con casillas de selección individuales. La fila muestra un recuento de activaciones como «4/6».

### Tipos de exportación y qué cámaras los admiten

Existen seis tipos de exportación: **Raw, sin Bayer, radiancia, reflectancia, vista previa, índice**. Solo aparecen los aplicables en la fila de cada cámara:

| Tipo de exportación | Contenido | RGB (FRGB) | Multiespectral Bayer (FRGN/FOCN/FNGB) | Mono (M3M) |
| --- | --- | --- | --- | --- |
| **Raw** | Mosaico de Bayer (mono: la banda única) directamente del sensor | ✓ | ✓ | ✓ |
| **Sin demosaico** | Demosaico lineal: se elimina el desplazamiento oscuro (DSNU) del paquete de calibración del archivo RAW y, a continuación, se aplica un demosaico sensible a los bordes —sin corrección de linealidad, campo plano ni desmezclado—. Se guarda como un archivo RGB de 3 canales en formato TIFF (mono: escala de grises de 1 canal) | ✓ | ✓ | ✓ |
| **Vista previa** | Cadena de visualización completa (balance de blancos + gamma según el perfil de la cámara, más el ajuste de «Highlights» de la cámara RGB; multiespectral: estiramiento de colores falsos) | ✓ | ✓ | ✓ |
| **Radiancia** | float32 W/m²/sr/nm a través de la cadena radiométrica completa | — (no disponible) | ✓ | ✓ |
| **Reflectancia** | uint16 ρ (32768 = 1,0) | — (no disponible) | ✓ — solo se muestra cuando la cámara dispone de un sensor de luz DAQ (propio o heredado de su matriz) | igual que en multiespectral |
| **Índice** | Representación del índice de vegetación (LUT) | — | ✓ — requiere una expresión de índice habilitada y no vacía en la cámara, y no está disponible para los miembros de una matriz combinada (la matriz posee un índice compartido) | — (un índice necesita ≥2 bandas; véase [Cámaras mono y índices de vegetación](mono-indices.md)) |

La radiancia y la reflectancia nunca están disponibles para cámaras RGB: la radiancia por píxel Bayer no tiene sentido para un sensor fotométrico de banda ancha.

### Captura más rápida

El conmutador **⚡ Captura más rápida — solo RAW**(de color naranja cuando está activado) anula todas las selecciones de exportación y las establece en**solo RAW** —además de una composición gratuita con índice combinado para las matrices—, de modo que el fotograma se guarda lo más rápido posible: los cálculos de radiancia, reflectancia y visualización se omiten por completo en el momento de la captura.

{% hint style="info" %}
**Se sigue guardando un `.daq`.** Cuando se asigna un sensor de luz, «Captura más rápida» sigue registrando la lectura descendente del DAQ (en la carpeta `daq/` de las capturas), de modo que los productos de radiancia, reflectancia e índice se pueden generar posteriormente mediante un nuevo procesamiento (véase [Reprocesamiento de capturas](#re-processing-captures-into-calibrated-products)). Además, Fastest Capture no afecta a las selecciones de las casillas de verificación: si lo desactivas, estas vuelven a aparecer.
{% endhint %}

### Controles por matriz

Cada matriz conectada tiene su propia tarjeta de grupo en el panel:

* **Casilla de verificación «Incluir»** (con tres estados para los miembros) y el nombre de la matriz con su modo de visualización: «(combinado | separado)».
* Casilla de verificación **Alineado** (activada por defecto): distorsiona las exportaciones de los miembros según el perfil de alineación de la matriz, de modo que las exportaciones quedan registradas píxel a píxel entre las cámaras. Los datos sin procesar permanecen sin distorsionar, pero llevan la transformación en sus metadatos. (El perfil en sí se calcula en el [panel de configuración de la matriz](camera-settings.md#alignment-co-registration).) Los archivos de los miembros de la matriz se guardan en la orientación nativa de cada cámara, y cualquier ajuste de espejo o rotación se registra en el archivo en lugar de aplicarse.
* Las filas de las cámaras que forman parte del conjunto están anidadas dentro de la tarjeta.

La tarjeta del conjunto también alberga dos grabadores. Piensa en ellos como **«supervisión frente a análisis»**:

| Grabador | Nivel | Qué graba |
| --- | --- | --- |
| **● Grabar vídeo de índice combinado / ■ Detener la grabación** *(solo matrices combinadas)* | **Supervisión** | La composición en directo del índice combinado en vídeo a 10 fps: 8 bits, resolución de vista previa, LUT integrada. Requiere un proyecto abierto y una vista en directo en streaming. Muestra los fotogramas y el tiempo transcurrido durante la grabación. |
| **⦿ Grabar ráfaga sin procesar / ■ Detener ráfaga sin procesar** *(cualquier matriz)* | **Análisis**| Fotogramas Bayer sin procesar a la velocidad de captura en directo (sin procesamiento), además de un manifiesto por fotograma y lecturas `.daq`, en formato `captures/bursts/`. Tras una ráfaga, aparece el botón**Crear vídeo**: vuelve a procesar la ráfaga sin conexión para convertirla en vídeo calibrado —índice combinado y/o radiancia/reflectancia/índice por cámara— más archivos TIFF opcionales. La creación del índice combinado se inicia automáticamente al detener la ráfaga. |

<!-- SCREENSHOT-NEEDED: an array group card in Capture Settings while a raw burst is recording — the ⦿/■ burst button in its recording state with frame count, and (in a second capture) the Build video button that appears after stopping. -->

## El flujo

<!-- SCREENSHOT-NEEDED: the sidebar during a capture — Capture All showing live "Capturing… 3/6" progress text, and (second capture) the result flash "Saved N files". -->

«Capturar todo»

Pulsa **Capturar todo** en la lista de cámaras de la barra lateral:

1. Todas las cámaras incluidas, visibles y que no estén en pausa capturan con los tipos de exportación seleccionados. **Las matrices se activan como un único disparador sincronizado** (un único grupo sincronizado entre todos los miembros —véase [Matrices multicámara](arrays.md)); las cámaras independientes capturan de forma individual.
2. Se omiten las cámaras ocultas (con el icono del ojo) o en pausa. Una matriz solo queda bloqueada por completo cuando *todos* sus miembros están ocultos o en pausa.
3. Siempre que se asigne un sensor de luz, la lectura correspondiente de DAQ de luz descendente se guarda como un archivo `.daq` en la carpeta `daq/` de las capturas —incluso para capturas solo en formato sin procesar— para que los productos radiométricos siempre puedan obtenerse posteriormente.
4. El botón muestra el progreso en tiempo real —«Capturando… hecho/total»— y, en el modo Continuo/Intervalo, se convierte en **Detener (N)**. Cada elemento de captura tiene un tiempo de espera de 300 s.
5. Cuando finaliza la pasada, un mensaje de resultado muestra **«Archivos guardados: N»**o**«Archivos guardados: N, F fallidos»**, además de «(S ocultos/en pausa/omitidos)» cuando se han omitido cámaras. Si la unidad se llena, la ejecución se detiene y muestra el mensaje**«Detenido: unidad llena. Guardados N, F fallaron»**; cada archivo que no se haya podido escribir se cuenta como fallido.

## Dónde se guardan las capturas

Las capturas se guardan en el proyecto abierto en `<project>/captures/`. Cada tipo de exportación se guarda en su **propia subcarpeta**, por lo que una captura de varios niveles nunca mezcla tipos:

```
<project>/captures/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when Index is selected
├── composite/     array foreground/background live-view composite, when produced
├── clamp_masks/   radiance/…_clampmask.tif, reflectance/…, index/… — unmix clamp masks,
│                  one subfolder per product folder, only when a band was clamped
├── bursts/        raw-burst recordings (frames + manifest + .daq per burst)
└── daq/           the downwelling reading(s) matched to each capture (.daq), each with a
                   .daq.cameras.json naming the cameras it describes
```

* `<ts>` es la marca de tiempo de la captura y `<serial>` el número de serie de la cámara. Las capturas independientes se denominan `capture_<ts>_SN<serial>_<level>`; las capturas en serie procedentes de un disparador sincronizado se denominan `sync_<ts>_SN<serial>_<level>` y **comparten una misma marca de tiempo entre todas las cámaras del grupo** (el sufijo de nivel se omite cuando una cámara guarda un único nivel).
* **Una asimetría que hay que tener en cuenta:** el nivel de visualización se almacena en una carpeta denominada `preview/`, mientras que los archivos conservan `_display` en el nombre; la carpeta y el sufijo difieren únicamente para ese nivel.
* Los niveles desconocidos se guardan en una carpeta con su propio nombre; si no se puede crear una subcarpeta, el archivo se guarda en la raíz de «captures» en lugar de perderse.
* Los archivos TIFF de captura se comprimen sin pérdida (DEFLATE) de forma predeterminada e incluyen todos sus metadatos de calibración y procesamiento **dentro del XMP del archivo** — las capturas son autodescriptivas, sin archivos complementarios aparte de la lectura `.daq` (y su `.daq.cameras.json`).
* **Máscaras de recorte (novedad en la versión 1.3.0).**Cuando la separación espectral 3×3 tenía que recortar una banda a 0 en al menos un píxel, los productos de radiancia, reflectancia e índice obtienen una máscara por píxel, `clamp_masks/<product folder>/<product name>_clampmask.tif` (bit 0 = la banda en el canal R del producto, bit 1 = G, bit 2 = B); la etiqueta XMP del producto, `Chloros:UnmixClampMask`, apunta a ella. Las máscaras se encuentran**junto a** las carpetas de los productos, nunca dentro de ellas, por lo que una carpeta de productos como `reflectance/` puede entregarse tal cual al software de fotogrametría. El propio importador de Chloros omite las máscaras.
* **Orientación (1.3.0).** Los productos «Preview», «Radiance», «Reflectance» e «Index» de una cámara independiente se giran según su configuración de «Mirror/Rotation»; los archivos «Raw» y «Debayered» permanecen tal y como los leyó el sensor. Todos los archivos miembros de la matriz se mantienen en su formato nativo. Cada archivo registra la configuración en su XMP, por lo que al volver a procesarlo se aplica de la misma manera.
* **A qué cámara pertenece un `.daq` (1.3.0).** El `.daq` de una captura se asocia, en el momento del procesamiento, únicamente a las imágenes de las cámaras que indica su `.daq.cameras.json`. Por lo tanto, una cámara independiente que no disponga de sensor de luz propio ya no toma prestada la lectura de otra cámara: hay que asignarle un sensor o registrar uno en la pestaña «Sensores de luz». Un registro de sensor de luz (y cualquier `.daq` que carezca de ese archivo, incluidas las capturas anteriores) sigue siendo válido para todas las cámaras.

Este es el mismo esquema que `chloros-cli lattice capture` / `array-capture` escriben en su directorio `-o` —documentado en la [Referencia de la CLI, § Cómo es una carpeta de capturas](../reference/cli-reference.md#what-a-captures-folder-looks-like).

<!-- SCREENSHOT-NEEDED: OS file explorer showing a real <project>/captures/ folder after a multi-level array capture — the raw/debayered/radiance/reflectance/preview subfolders, the daq/ and clamp_masks/ folders, and sync_<ts>_SN<serial>_<level>.tif filenames visible inside one subfolder. -->

## Reprocesamiento de capturas para obtener productos calibrados

Los fotogramas sin procesar capturados, junto con el archivo `.daq` guardado, son todo lo que necesita el proceso de procesamiento; por eso «Fastest Capture» es una opción segura para el trabajo real.

* ****GUI**: añade la carpeta de capturas a un proyecto ([Añadir archivos a un proyecto](../processing-images-gui/adding-files-to-a-project.md)) y procesa como de costumbre.
* ****CLI**: indica a `process` la **raíz de las capturas**:

```bash
chloros-cli process "C:/ChlorosProjects/MyField/captures"
```

`process` normalmente solo importa la carpeta que se especifique, pero cuando dicha carpeta no contiene imágenes y tiene subcarpetas, recorre automáticamente los niveles inferiores, de modo que las subcarpetas de cada nivel y la carpeta `daq/` se recogen de una sola vez. Cada captura se importa como una **única imagen** con los demás niveles adjuntos como modos de visualización, no como una imagen por nivel.

Nombrar directamente una subcarpeta de nivel (p. ej., `…/captures/raw/`) también funciona, pero deja fuera la carpeta `daq/`; cópiala junto con ella cuandoderivar un producto radiométrico a partir de `raw/`; de lo contrario, la coincidencia de la marca de tiempo no tendrá nada con lo que resolverse.

{% hint style="warning" %}
**El procesamiento siempre comienza desde `raw`.** Dentro de cada captura, el fotograma sin procesar es la fuente del proceso; `debayered`, `radiance`, `reflectance`, y `preview` aparecen como modos visualizables, pero nunca se devuelven al canal de procesamiento; volver a procesar un producto derivado volvería a aplicar el viñeteado, el color y los cálculos de radiancia ya integrados en sus píxeles, por lo que Chloros, en lugar de realizar un doble procesamiento, opta por descartarlos. Los renderizados `index/` y `composite/` no se procesan en absoluto (son salidas, no capturas). Una carpeta de capturas guardada **sin** importaciones en bruto se muestra con normalidad, pero `process` la omite y lo indica; `--input-level {raw,debayered,processed}` es la vía de escape deliberada que fuerza un punto de entrada. Consulta la [Referencia de la CLI](../reference/cli-reference.md#what-a-captures-folder-looks-like) para ver los mensajes exactos de omisión.
{% endhint %}

Otros dos comportamientos que conviene conocer al crear scripts de reprocesamiento:

* Una ejecución de `chloros-cli process` que haya solicitado productos pero **no haya generado ningún producto de imagen**fallará de forma evidente y terminará con un valor distinto de cero**; nunca obtendrás una ejecución vacía silenciosa. Las ejecuciones correctas informan del recuento de productos. (Una ejecución deliberada que solo genere metadatos sigue contándose como un éxito).
* Las exportaciones procesadas y reimportadas nunca ocupan la ranura de datos sin procesar de una captura; los datos sin procesar originales siempre siguen siendo la fuente del proceso.

## Equivalentes en la CLI



Todo lo que aparece en esta página se puede ejecutar en modo sin interfaz gráfica. Los modos de captura de la interfaz gráfica se corresponden directamente con `chloros-cli lattice array-capture`:

| GUI | CLI |
| --- | --- |
| Única | `chloros-cli lattice array-capture` |
| Continua | `array-capture --continuous [--count N] [--duration S]` |
| Por intervalos | `array-capture --interval S [--duration S]` |
| Captura más rápida | `array-capture --fastest` |
| Casilla «Alineado» | `--aligned / --no-aligned` |
| Casillas de selección de tipo de exportación | `--processing LEVEL` o `--levels L1,L2,…` (por defecto `all`) |
| Grabar vídeo de índice | `chloros-cli lattice array-record` |
| Grabar ráfaga sin procesar / Crear vídeo | `chloros-cli lattice array-burst` / `array-build-video` |

Las tablas completas de indicadores, la opción de captura estabilizada con AE inteligente (`--smart`) y el modelo de velocidad sostenida se describen en la [Referencia de CLI, § Modos de captura, grabadoras y reprocesamiento offline](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

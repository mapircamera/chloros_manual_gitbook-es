# Cámaras monocromáticas e índices de vegetación

## Una cámara = una banda

Una cámara **M3M**es la versión monocromática de la**M3C**de Bayer: un sensor IMX265 monocromático detrás de un único filtro de interferencia de banda estrecha. La cadena del modelo indica el nombre de la banda: `M3M-<lens>-F<wavelength>`, por ejemplo, `M3M-L87-F685` (que en Chloros se muestra como `LATT-M3M-L87-F685`). El sensor ofrece una**única banda en escala de grises** sin mosaico de Bayer: no hay nada que desmosaicar, ni diafonía entre canales que separar, ni balance de blancos que ajustar.

Consecuencias que conviene conocer antes de planificar un sistema monocromo:

* **La radiancia y la reflectancia están totalmente definidas por banda.**Se trata de mapas radiométricos por banda, por lo que una cámara M3M produce radiancia calibrada en float32 (W/m²/sr/nm) y reflectancia en uint16 (`32768` = ρ 1,0) exactamente igual que lo hace una banda M3C. Los fotogramas monocromáticos contienen una matriz de respuesta del sensor**identitaria**; no es necesario aplicar ninguna desmezcla 3×3.
* **Una sola cámara monocromática no puede generar un índice de vegetación.** NDVI, NDRE y similares necesitan al menos dos bandas. Para calcular índices a partir de hardware monocromático, hay que combinar varias cámaras M3M; véase más abajo.
* Las cámaras M3M transmiten **Mono12** (12 bits, 2 bytes/píxel en la conexión), lo cual es importante para la [gestión del ancho de banda de la matriz](arrays.md#bandwidth-the-rules-of-thumb).

## Lo que Chloros omite en el modo mono — y cómo te lo indica

Las etapas del proceso de procesamiento del color simplemente no se aplican a un sensor de una sola banda. Chloros **las omite con un mensaje de una línea** en lugar de generar un error, y sigue ejecutándolas con normalidad para cualquier cámara M3C (Bayer) en la misma sesión:

| Etapa | Comportamiento en modo mono (M3M) | Comportamiento en M3C |
| --- | --- | --- |
| Desmosaico / debayer | Se omite: el nivel de exportación de `debayered` es una imagen en escala de grises de 1 canal. | Desmosaico de 3 canales. |
| Balance de blancos (`lattice white-balance`) | Se omite con un mensaje de una línea. | Se ejecuta con normalidad. |
| Perfil de color (`lattice color-profile`) | Se omite con un mensaje de una línea. | Se ejecuta con normalidad. |
| Saturación/contraste (`lattice color`) | Se omite con un mensaje de una línea. | Se ejecuta con normalidad. |
| Separación de interferencias espectrales | Identidad (sin matriz 3×3). | Se aplica una matriz 3×3 por cámara. |
| Radiancia/reflectancia | **Se ejecuta** — por banda, totalmente calibrado. | Se ejecuta por banda. |

La interfaz gráfica de usuario aplica la misma filtración: para una cámara monocromática, el panel de ajustes por cámara oculta las filas exclusivas de RGB (balance de blancos, gamma, perfil de color, saturación, Contraste, divisiones de canales), y el histograma en tiempo real se bloquea en una única traza **MONO**. El discriminador en todo el apilado es el token `M3M` en la cadena del modelo, que aparece en la interfaz gráfica de usuario/SDK como `is_mono`.

## Los índices requieren ≥ 2 bandas: alinear → apilar → indexar

El flujo de trabajo de indexación monocromática sigue siempre los mismos tres pasos:

1. **Alineación**: apuntar varias cámaras M3M a diferentes longitudes de onda (por ejemplo, una F650 «Red» y una F850 «NIR»), conéctalas como una [matriz multicámara](arrays.md) y deja que Chloros calcule la transformación de co-registro entre las cámaras.
2. **Pila**: los fotogramas alineados se convierten en una imagen multibanda (cada cámara aporta una banda con nombre).
3. **Índice**: evalúa una fórmula de índice sobre las bandas de la pila, renderizándola opcionalmente a través de una LUT.

En la interfaz gráfica de usuario, toda esta cadena constituye el modo de visualización de matriz **Cámaras combinadas**: la composición en directo ya está alineada, y la Calculadora de índice de la matriz (más abajo) define la fórmula que se representa. Las exportaciones capturadas pueden deformarse para obtener la misma alineación con la opción de captura**Alineado**.

## La calculadora de índices

La calculadora de índices crea la expresión de índice utilizada por la vista en directo y las exportaciones de índices por cámara. Se trata de una superficie compartida, a la que se accede desde dos lugares de la barra lateral de la pestaña «Cámaras»:

* **Por cámara**— Vista previa en directo → icono de engranaje**Índice** (solo cámaras Bayer RGN/OCN/NGB; una cámara monocromática aislada no tiene control de índice porque una sola banda no puede generar un índice).
* **Por matriz**— Configuración de la matriz → Vista en directo → icono de engranaje**Índice**. Esta es la ruta monocromática: la lista de bandas abarca**todas las cámaras que forman parte de la matriz**, por lo que un par monocromático aporta aquí sus dos bandas.

<!-- SCREENSHOT-NEEDED: Index Calculator pane opened for a combined array of two mono cameras (e.g. F650 + F850): band chips row showing the two bands with wavelength labels, the operator buttons, the expression textarea containing "(NIR - Red) / (NIR + Red)", the green "Valid expression" banner, the LUT controls (Apply LUT checked, Level 7-stop, Min 0.2 / Max 1), and the live histogram with p2/p98 percentile lines. -->

Sus controles, de arriba abajo:

* **Fichas de banda** («Bandas — haz clic para añadir a la expresión») — un botón por cada banda disponible, etiquetado con el nombre del color + longitud de onda en nm (los nombres de colores duplicados se distinguen, por ejemplo, como «Color 850»). Al hacer clic se inserta el token de la banda en la posición del cursor. Se filtran las bandas de las cámaras que no pueden producir radiancia por banda (RGB/FRGB).
* **Botones de operadores y funciones**: `+ - * / ( ) ^ ,` más `abs() sqrt() log() log10() exp() min() max() pow()`.
* **Área de texto de expresiones** — fórmula de entrada libre; el marcador de posición muestra la forma clásica de NDVI: `(NIR - Red) / (NIR + Red)`. Una vista previa tokenizada de solo lectura situada encima muestra los chips de banda, los números y las banderas como tokens desconocidos.
* **Banner de validez**— gris «Vacío — no se aplicará ningún índice»; verde «Expresión válida»; rojo con el error de análisis específico (banda desconocida, banda ambigua captada por varias cámaras, paréntesis que falta, …); o ámbar cuando la expresión es válida pero**constante** (p. ej., `X/X`, o un denominador NDVI escrito con `−` en lugar de `+`): una constante asigna un único color a todo el fotograma.
* Aparece una advertencia ámbar independiente si la expresión aplicada es correcta, pero el **fotograma en directo es uniforme** (escena plana o saturada): se detecta automáticamente el colapso del histograma.
* **Aplicar LUT**(activado por defecto; desactivado = estiramiento en escala de grises),**Nivel**de 2/3/5/7 pasos (por defecto, 7 pasos) y entradas**Mín. / Máx.**a ambos lados de la barra de degradado. El valor predeterminado de Mín. es**0,2**: amplía la rampa de color al rango relevante para la vegetación, mientras que los valores inferiores se muestran en escala de grises; establece Mín. en −1 para obtener el rango completo del índice (el botón**Restablecer** restablece el rango de −1 a +1). El valor predeterminado de Máx. es 1.
* **Histograma en tiempo real** de la distribución del índice: barras escaladas por la raíz cuadrada, líneas de percentiles p2 y p98 en ámbar, una línea mediana blanca y lecturas de los extremos fuera de rango («◀ N % &lt; lo» / «hi &lt; N % ▶») que se vuelven de color ámbar por encima del 1 % como indicación para ampliar la ventana de Mín./Máx.
* **Aplicar**aplica la expresión a la transmisión en directo; los ajustes de LUT se aplican en tiempo real sin necesidad de pulsar «Aplicar». Las expresiones son deliberadamente**exclusivas de la sesión**: no se conservan entre sesiones.

<!-- SCREENSHOT-NEEDED: Combined-array live tile rendering NDVI from a mono pair through the default 7-stop LUT, with the array name pill and fps readout visible — the result of applying the expression from the previous screenshot. -->

## La ruta CLI

La misma cadena de alineación → pila → índice, programable de principio a fin:

```bash
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`--channel` asigna los símbolos de un preajuste a los nombres de las bandas de la pila. Dos reglas te evitarán una ejecución fallida:

* **Los símbolos distinguen entre mayúsculas y minúsculas** y deben coincidir exactamente con los nombres de los canales del preset — los presets utilizan minúsculas (los de NDVI son `red`,`nir`; comprueba `--list-presets`). `--channel red=Red_660` funciona; `--channel RED=660` falla con un error de `channel_map missing entries`.
* El lado de la banda debe especificar el nombre de una banda de la pila alineada (`lattice align-info --profile align.json` las enumera). El modo sin conexión también acepta índices de banda con base 0, p. ej., `--channel red=0 --channel nir=1`.

`lattice index` también se ejecuta totalmente sin conexión con un TIFF multibanda alineado y guardado:

```bash
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn
```

### Preajustes de índice

`lattice index --preset` (y la opción [Index/LUT sandbox](../image-viewer-gui/index-lut-sandbox.md) de la pestaña Imagen, que utiliza el mismo motor) incluye estos **22 preajustes**:

`NDVI, GNDVI, BNDVI, NDRE, ENDVI, SAVI, OSAVI, MSAVI, EVI, EVI2, CVI, MSR, TDVI, LAI, GLI, NGRDI, VARI, TGI, EXG, CIRE, CIGREEN, NDWI`

Ejecuta `chloros-cli lattice index --list-presets` para ver la fórmula y los símbolos de canal de cada preajuste, y `--list-gradients` para ver los degradados de color disponibles. Las fórmulas personalizadas utilizan `--formula EXPR` con la misma sintaxis que la Calculadora de índices. Ten en cuenta que esta lista de preajustes es específica del motor de índices LATTICE; el menú desplegable «Procesamiento» de la configuración del proyecto para las imágenes importadas muestra una lista diferente (véase [Fórmulas de índices multiespectrales](../project-settings/multispectral-index-formulas.md)).

El conjunto completo de indicadores (`--output-format`, `--vmin/--vmax/--percentile`, `--bg-mode`, mandos de alineación y deformación para `--live`, y más) se documenta en la [Referencia de CLI, sección «Índices / Matemáticas de la vegetación»](../reference/cli-reference.md#index--vegetation-maths); los equivalentes de SDK se encuentran en la [Referencia de SDK](../reference/sdk-reference.md).

## Captura de productos de índice a partir de una matriz mono

Con una matriz conectada y una expresión de índice aplicada, `array-capture` (o la opción **Capturar todo** de la interfaz gráfica de usuario) guarda los niveles de exportación por cámara *y* el renderizado del índice — `--index`/`--no-index` lo activa en CLI, y la captura incluye por defecto todos los niveles aplicables. La contribución de una cámara mono a cada grupo de captura es su única banda en los niveles sin procesar/despixelar (escala de grises)/radiancia/reflectancia, más la composición de índice combinado compartida cuando la matriz funciona en modo combinado. Véase [Matrices multicámara § Captura](arrays.md#capturing-monitoring-vs-analysis).

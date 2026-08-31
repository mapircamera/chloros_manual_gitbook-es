# Configuración del proyecto

La barra lateral «Configuración del proyecto»<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

de Chloros te permite configurar todos los aspectos relacionados con el procesamiento de imágenes, la detección de objetivos de calibración, el cálculo de índices multiespectrales y las opciones de exportación de tu proyecto. Estos ajustes se guardan junto con tu proyecto y pueden guardarse como plantillas para reutilizarlos en varios proyectos.

## Cómo acceder a la configuración del proyecto

Para acceder a la configuración del proyecto:

1. Abre un proyecto en Chloros
2. Haz clic en la pestaña **Configuración del proyecto**<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

de la barra lateral izquierda
3. El panel de configuración mostrará todas las opciones de configuración disponibles organizadas por categorías

<!-- SCREENSHOT-NEEDED: Full Project Settings sidebar of a LATTICE project, scrolled so the Processing category is visible showing the per-product export checkboxes (Export sensor response, Export vignette corrected, Export debayered, Export preview, Export radiance, Export reflectance) and the Debayer method row. -->

{% hint style="info" %}
**Los ajustes que dependen de otros ajustes aparecen en gris.** Cuando un control principal impide que se aplique una configuración (por ejemplo, al desmarcar *Calibración de reflectancia / balance de blancos* se imposibilita la opción *Exportar reflectancia*), el control dependiente se desactiva y su información sobre herramientas indica el control que debe modificarse.
{% endhint %}

***

## Visualización

### Resolución de la miniatura de la imagen

* **Tipo**: Menú desplegable
* **Opciones**: `Default (512 px)`, `1024 px`, `2048 px`, `Full resolution`
* **Predeterminado**: Predeterminado (512 píxeles)
* **Descripción**: Resolución (en el lado más largo, en píxeles) a la que se generan las miniaturas de la cuadrícula de imágenes. Los valores más altos se ven más nítidos al ampliar la imagen, pero se cargan más lentamente y consumen más memoria. La resolución completa corresponde al tamaño original de la imagen.
* **Nota**: Solo para visualización; esto nunca afecta al procesamiento ni a los archivos exportados.***

## Detección de objetivos

Estos ajustes controlan cómo Chloros detecta y procesa los objetivos de calibración en tus imágenes. Ambos solo están activos mientras está habilitada la **calibración de reflectancia / balance de blancos** (en caso contrario, aparecen en gris, ya que la detección de objetivos se omite por completo).

### Área mínima de muestra de calibración (px)

* **Tipo**: Número
* **Rango**: De 0 a 10 000 píxeles
* **Valor predeterminado**: 25 píxeles
* **Descripción**: Establece el área mínima (en píxeles) necesaria para que una región detectada se considere una muestra válida de objetivo de calibración. Los valores más bajos detectarán objetivos más pequeños, pero pueden aumentar los falsos positivos. Los valores más altos requieren regiones de objetivos más grandes y definidas para su detección.
* **Cuándo ajustarlo**:
  * Aumenta este valor si obtienes detecciones erróneas en pequeños artefactos de la imagen
  * Disminúyelo si tus objetivos de calibración aparecen pequeños en las imágenes y no se detectan

### Agrupación mínima de objetivos (0-100)

* **Tipo**: Número
* **Rango**: de 0 a 100
* **Valor predeterminado**: 60
* **Descripción**: Controla el umbral de agrupación para agrupar regiones de colores similares al detectar objetivos de calibración. Los valores más altos requieren que se agrupen colores más similares, lo que da lugar a una detección de objetivos más conservadora. Los valores más bajos permiten una mayor variación de color dentro de un grupo de objetivos.
* **Cuándo ajustarlo**:
  * Aumenta el valor si los objetivos de calibración se dividen en varias detecciones
  * Disminuye el valor si los objetivos de calibración con variación de color no se detectan por completo

***

## Procesamiento

Estos ajustes controlan cómo Chloros procesa y calibra tus imágenes.

### Corrección de viñeteado

* **Tipo**: Casilla de selección
* **Predeterminado**: Activado (marcado)
* **Descripción**: Aplica la corrección de viñeteado para compensar el oscurecimiento de la lente en los bordes de las imágenes. El viñeteado es un fenómeno óptico común en el que las esquinas y los bordes de una imagen aparecen más oscuros que el centro debido a las características de la lente.
* **Efecto secundario**: Esta opción también selecciona qué *producto de reserva no calibrado* genera una ejecución (véase más abajo).

### Calibración de reflectancia / balance de blancos

* **Tipo**: Casilla de selección
* **Predeterminado**: Activado (marcado)
* **Descripción**: Habilita la calibración de reflectancia —a partir de los objetivos de calibración detectados en el encuadre y/o de los datos de luz descendente del sensor de luz del DAQ, dependiendo de la cámara y de lo que esté disponible—. Esto normaliza los valores de reflectancia en todo el conjunto de datos y garantiza mediciones consistentes independientemente de las condiciones de iluminación.
* **Cuando está desactivada**: Se omite por completo la detección de objetivos y**ninguna cámara puede generar un producto de reflectancia**, tanto en el sistema Survey3 basado en objetivos como en el LATTICE basado en DAQ. Los ajustes dependientes (*Exportar reflectancia*, *Intervalo mínimo de recalibración* y los umbrales de detección de objetivos) aparecen desactivados.

### Productos de reserva sin calibrar: Exportar respuesta del sensor / Exportar con corrección de viñeteado

* **Tipo**: Dos casillas de selección
* **Valores predeterminados**: Ambas activadas (marcadas)
* **Descripción**: Cuando no es posible calibrar la reflectancia de un fotograma (no se ha encontrado ningún objetivo de calibración o la calibración de reflectancia está desactivada), se guarda como un *producto de reserva sin calibrar*. **Existe exactamente uno de los dos productos de reserva por serie de pruebas, para cada modelo de cámara**, elegido mediante el selector *Corrección de viñeteado*:
  * Corrección de viñeteado **activada**→ `Vignette_Corrected_Images/` (regulada por**Exportar con corrección de viñeteado**)
  * Corrección de viñeteado **desactivada**→ `Sensor_Response_Images/` (regulada por**Exportar respuesta del sensor**)
* El producto de reserva que no está activo aparece en gris. Al desmarcar el que está activo, se impide por completo que se guarde ese archivo.

### Productos de exportación de LATTICE

En los proyectos que contienen capturas de LATTICE, cada fotograma de LATTICE importado se distribuye a todos los productos habilitados **y aplicables**en una sola pasada de procesamiento. Cuatro casillas de selección controlan esta distribución (todas**activadas** por defecto):

| Configuración | Carpeta de salida | Qué exporta |
| --- | --- | --- |
| **Exportar sin bayering** | `Debayered_Images/` | La imagen lineal sin bayering. Se aplica a RGB y a las cámaras multiespectrales. |
| **Exportar vista previa** | `Preview_Images/` | La vista previa en pantalla. RGB = balance de blancos (iluminante DAQ cuando esté disponible; en caso contrario, mundo gris) + gamma; multiespectral = expansión a colores falsos. |
| **Radiancia de exportación** | `Radiance_Images/` | Radiancia espectral de tipo Float32 en W/m²/sr/nm. Solo multiespectral (M3C/M3M); no aplicable a los originales RGB. Siempre se escribe como TIFF de 32 bits, independientemente de la configuración de *Formato de imagen calibrada*. |
| **Reflectancia de exportación**| `Reflectance_Calibrated_Images/` | Reflectancia Uint16, escalada de modo que**32768 = reflectancia 1,0** (marcada como XMP `Chloros:PixelScale`). Solo multiespectral; se escribe cuando un registro descendente `.daq` coincidente (o un objetivo dentro del fotograma que haya superado el control de calidad) cubre el fotograma. |

* Las cámaras maestras RGB emiten datos sin bayering + vista previa; en ellas se omiten la radiancia y la reflectancia, ya que no son aplicables.
* La profundidad de bits de la imagen sin bayering y la vista previa sigue la configuración del *formato de imagen calibrado*; la radiancia es siempre float32.
* El procesamiento de Survey3 no se ve afectado por estos cuatro conmutadores.

Los mismos cuatro conmutadores existen sin encabezado como `chloros-cli process --debayered / --preview / --radiance / --reflectance` y como los parámetros correspondientes de SDK. Sustituyeron al antiguo indicador `--radiometric-output`, que ya no existe.

{% hint style="warning" %}
**Si se desactivan todos los productos aplicables, la ejecución falla.** A partir de la versión 1.2.0, una ejecución de procesamiento en la que se solicitaron productos pero no se generó ningún producto de imagen informa de un fallo y el CLI finaliza con un valor distinto de cero, en lugar de informar de un éxito silencioso. El registro indica el producto que no se pudo generar y el motivo. Una ejecución deliberadamente limitada a metadatos (sin nada solicitado) sigue considerándose un éxito.
{% endhint %}

### Fuente de reflectancia (configuración del proyecto, establecida mediante CLI/SDK)

El proyecto también almacena qué **referencia de reflectancia** utiliza el producto de reflectancia LATTICE. No hay ningún control específico en el panel de configuración; el valor se almacena en la configuración del proyecto como `Processing → "Target reflectance source"` y se establece mediante `chloros-cli process --reflectance-source {auto,target,daq}` o el parámetro `reflectance_source` de SDK:

* **`auto`** (por defecto): un objetivo de calibración dentro del encuadre que haya superado el control de calidad (QA) se convierte en la referencia absoluta; si no hay ningún objetivo presente o el control de calidad falla, se recurre a la división de la radiación descendente del DAQ (ρ = πL/E).
* **`target`**: reflectancia estrictamente basada en el objetivo; sin sustitución por parte del DAQ.
* **`daq`**: reflectancia determinada por el DAQ; las marcas dentro del fotograma no se utilizan como referencia.

El valor almacenado se compara sin distinguir entre mayúsculas y minúsculas, y se aceptan algunas variantes ortográficas como alias: `target`, `target_image`, `empirical` y `empirical_line` significan todos **objetivo**; `daq`, `dls`, `light_sensor` y `sensor` significan todos**daq**. Cualquier otra cosa —incluida una clave ausente— se resuelve como**auto**.

Las lecturas de **medidas** por unidad se buscan mediante el número de serie o el código QR de la unidad de destino, como `<serial>.csv`, en tres lugares: el directorio indicado con `--target-reflectance-dir` (almacenado como `Processing → "Target reflectance dir"`), la propia carpeta `target_reflectance/` del proyecto y la ruta de la variable de entorno `CHLOROS_TARGET_REFLECTANCE_DIR`. Cuando no existe ningún escaneo medido para esa unidad, se utiliza en su lugar la curva nominal publicada para el modelo de destino.

### Método de despixelar

* **Tipo**: Selección en el menú desplegable
* **Opciones**:
  * Estándar (rápido, calidad media)
  * Sensible a la textura (lento, máxima calidad) \[Chloros+]
* **Predeterminado**: Estándar (rápido, calidad media)
* **Descripción**: Selecciona el algoritmo de demosaico utilizado para convertir los datos sin procesar del sensor con patrón de Bayer en imágenes a todo color. El método «Estándar (rápido, calidad media)» ofrece un equilibrio óptimo entre velocidad de procesamiento y calidad de imagen. El método «Con reconocimiento de texturas (lento, máxima calidad)» \[Chloros+] utiliza un algoritmo de demosaico de alta calidad sensible a los bordes, combinado con un modelo de eliminación de ruido basado en IA/ML que elimina casi todo el ruido del demosaico. El modelo «Texture Aware» requiere memoria de la GPU (VRAM) para ejecutarse. Recomendamos utilizarlo cuando se disponga de más de 4 GB de VRAM para un procesamiento más rápido.
* **Siempre que la fila sea un menú desplegable**: el menú desplegable de dos opciones solo aparece cuando**ambas**condiciones se cumplen — has iniciado sesión con una suscripción válida a Chloros+,**y** el proyecto no contiene capturas de LATTICE. De lo contrario, la fila se muestra como texto sin formato que dice «`Standard (Fast, Medium Quality)`» sin ninguna opción para seleccionar.
* **Nota sobre LATTICE**: No existe ningún modelo «Texture Aware» entrenado para LATTICE, y el proceso obliga a aplicar el demosaico estándar a los fotogramas LATTICE, independientemente del valor almacenado. Si se añade una carpeta LATTICE a un proyecto en el que ya se había seleccionado «Texture Aware», Chloros restablece la configuración a «Estándar» en lugar de dejar un valor obsoleto en `project.json`.

### Intervalo mínimo de recalibración

* **Tipo**: Número
* **Rango**: De 0 a 3.600 segundos
* **Predeterminado**: 0 segundos
* **Descripción**: Establece el intervalo de tiempo mínimo (en segundos) entre el uso de objetivos de calibración. Si se establece en 0, Chloros utilizará todos los objetivos de calibración detectados. Si se establece en un valor superior, Chloros solo utilizará los objetivos de calibración que estén separados por al menos ese número de segundos, lo que reduce el tiempo de procesamiento para conjuntos de datos con capturas frecuentes de objetivos de calibración.
* **Cuándo ajustarlo**:
  * Establece el valor en 0 para obtener la máxima precisión de calibración cuando las condiciones de iluminación varían
  * Aumenta el valor (por ejemplo, a entre 60 y 300 segundos) para acelerar el procesamiento cuando la iluminación es constante y se dispone de imágenes frecuentes de objetivos de calibración

### Desfase horario del sensor de luz

* **Tipo**: Número
* **Rango**: de -12 a +12 horas
* **Valor predeterminado**: 0 horas
* **Descripción**: Especifica la diferencia horaria (en horas respecto al UTC) para las marcas de tiempo de los datos del sensor de luz, que se utiliza al hacer coincidir los registros del sensor de luz con las horas de captura de las imágenes. Las grabaciones más recientes de `.daq` incluyen su propia información de zona horaria, por lo que esto es necesario principalmente para los registros más antiguos grabados en hora local.

### Aplicar correcciones PPK

* **Tipo**: Casilla de selección
* **Predeterminado**: Desactivado (sin marcar)
* **Descripción**: Habilita el uso de correcciones cinemáticas posprocesadas (PPK) de los registradores DAQ MAPIR que contengan un GPS (GNSS). Cuando está activada, Chloros utilizará cualquier archivo de registro .daq que contenga datos de pines de exposición en el directorio de su proyecto y aplicará correcciones de geolocalización precisas a sus imágenes.
* **Requisito**: Debe haber un archivo de registro .daq con entradas de pines de exposición en el directorio de su proyecto
* **Cuándo activarla**: Se recomienda activar siempre la corrección PPK si dispone de entradas de retroalimentación de exposición en su archivo de registro .daq.

### Pin de exposición 1

* **Tipo**: Selección desplegable
* **Visibilidad**: Solo visible cuando está activada la opción «Aplicar correcciones PPK» Y hay datos de exposición disponibles para el pin 1
* **Opciones**:
  * Nombres de modelos de cámara detectados en el proyecto
  * «No utilizar»: ignora este pin de exposición
* **Predeterminado**: Se selecciona automáticamente en función de la configuración del proyecto
* **Descripción**: Asigna una cámara específica al pin de exposición 1 para la sincronización temporal PPK. El pin de exposición registra el momento exacto en que se dispara el obturador de la cámara, lo cual es fundamental para una geolocalización PPK precisa.
* **Comportamiento de la selección automática**:
  * Una sola cámara + un solo pin: selecciona automáticamente la cámara
  * Una sola cámara + dos pines: el pin 1 se asigna automáticamente a la cámara
  * Varias cámaras: se requiere selección manual

### Pin de exposición 2

* **Tipo**: Selección en el menú desplegable
* **Visibilidad**: Solo visible cuando está activada la opción «Aplicar correcciones PPK» Y hay datos de exposición disponibles para el pin 2
* **Opciones**:
  * Nombres de modelos de cámara detectados en el proyecto
  * «No utilizar»: ignora este pin de exposición
* **Predeterminado**: Se selecciona automáticamente en función de la configuración del proyecto
* **Descripción**: Asigna una cámara específica al pin de exposición 2 para la sincronización temporal PPK cuando se utiliza una configuración de doble cámara.
* **Comportamiento de la selección automática**:
  * Una cámara + un pin: el pin 2 se configura automáticamente en «No utilizar»
  * Una cámara + dos pines: el pin 2 se configura automáticamente en «No utilizar»
  * Varias cámaras: se requiere selección manual
* **Nota**: No se puede asignar la misma cámara al pin 1 y al pin 2 simultáneamente.***

## Sensor de luz DAQ

Esta sección aparece en «Configuración del proyecto» y enumera todos los archivos de DAQ de luz descendente del proyecto: grabaciones `.daq` y registros de DAQ-M `.csv` de luz descendente. Las grabaciones realizadas en la pestaña «Sensores de luz» se añaden automáticamente al proyecto abierto.

<!-- SCREENSHOT-NEEDED: Project Settings "DAQ Light Sensor" section of a project containing at least one .daq file, showing the "Cap override (all files)" dropdown and a per-file row with its resolved cap. -->

Cada fila muestra el archivo, el modelo de sensor y la corrección de la tapa difusora que se aplica realmente a ese archivo. Por encima de las filas hay un único control para todo el proyecto:

### Anulación de la corrección de la tapa (todos los archivos)

* **Tipo**: Selección desplegable
* **Opciones**: `Auto` más los perfiles de corrección de la tapa válidos para los tipos de sensores presentes en el proyecto
* **Predeterminado**: Automático
* **Almacenado como**: `Processing → "DAQ cap id"` (por defecto, `auto`)
* **Descripción**: `Auto` utiliza el límite máximo registrado en cada archivo (se asume el límite máximo de luz solar cuando no se ha registrado nada; todos los dispositivos de adquisición de datos MAPIR incluyen de serie el corrector de luz solar). Al seleccionar un límite máximo específico, se anula**todos** los archivos de radiación descendente del proyecto: las grabaciones sin procesar se corrigen con él, y las grabaciones que ya llevan un cap se re-referencian (se deshace la corrección registrada y se aplica la seleccionada).
* **Importante**: El cap seleccionado debe coincidir con el cap que se instaló físicamente durante la grabación. Ni el sensor ni el software pueden detectar la tapa física; un ID de tapa que no coincida corregirá erróneamente los espectros.

Se ha optado deliberadamente por **un único** control para todo el proyecto en lugar de menús desplegables por archivo: la configuración se aplica a todas las fuentes de luz descendente del proyecto.***

## Alineación de matrices

Esta sección aparece **solo** cuando al menos una imagen del proyecto contiene la transformación de alineación entre módulos que las matrices LATTICE estampan en el momento de la captura (etiquetas XMP `Chloros:Alignment*`). Muestra cuántas imágenes contienen etiquetas de alineación, qué cámara es la de referencia (identificador `REF`) y una tabla con el recuento de imágenes por cámara.

<!-- SCREENSHOT-NEEDED: Project Settings "Array Alignment" section for an imported LATTICE array capture set, showing the tagged-image count, the per-camera rows with the REF badge, and the three controls (Apply array alignment, Crop to common overlap, Resampling). -->

### Aplicar alineación de matrices

* **Tipo**: Casilla de selección
* **Predeterminado**: Activado (marcado)
* **Almacenado como**: `Processing → "Array alignment"`
* **Descripción**: Deforma cada producto procesado (desbayering / vista previa / radiancia / reflectancia / índice) a la geometría de referencia compartida de la matriz utilizando la transformación aplicada en el momento de la captura. Desactivado = exportación en la geometría nativa de cada sensor.

### Recortar a la superposición común

* **Tipo**: Casilla de selección (solo está activa cuando «**Aplicar alineación de matriz**» está activada)
* **Predeterminado**: Habilitado (marcado)
* **Almacenado como**: `Processing → "Array alignment crop"`
* **Descripción**: Recorta las exportaciones alineadas a la región que comparten todos los módulos de cámara, de modo que cada banda tenga la misma huella. Desactivado mantiene el lienzo completo del sensor (relleno negro fuera de la fuente).

### Remuestreo

* **Tipo**: Menú desplegable (solo activo mientras «Aplicar alineación de matriz» esté activado)
* **Opciones**: `Bilinear (smooth, default)`, `Nearest (preserve exact values)`, `Cubic (sharpest)`
* **Predeterminado**: Bilineal
* **Almacenado como**: `Processing → "Array alignment interpolation"`
* **Descripción**: Interpolación utilizada por la transformación de alineación. «Más cercano» conserva los valores exactos de la fuente (sin mezcla entre píxeles) para un análisis radiométrico riguroso; «Bilineal» es la mejor opción para la cartografía y el uso visual.

Las mismas tres opciones existen sin encabezado como `chloros-cli process --array-alignment`, `--array-alignment-crop` y `--array-alignment-interp {bilinear,nearest,cubic}`.

***

## Índice

Estos ajustes te permiten configurar índices multiespectrales para su análisis y visualización.

### Añadir índice

* **Tipo**: Panel de configuración de índices especiales
* **Descripción**: Abre un panel interactivo en el que puedes seleccionar y configurar índices multiespectrales de vegetación (NDVI, NDRE, EVI, etc.) que se calcularán durante el procesamiento de la imagen. Puedes añadir varios índices, cada uno con su propia configuración de visualización.
* **Índices disponibles**: El menú desplegable de la interfaz gráfica de usuario incluye**27** fórmulas de índices multiespectrales predefinidas (véase [Fórmulas de índices multiespectrales](multispectral-index-formulas.md) para ver la lista completa, incluyendo los nombres que también son aceptados por la opción CLI/SDK `--indices`).
* **Características**:
  * Selecciona entre fórmulas de índices predefinidas
  * Arrastra los canales de filtro de tu cámara a las ranuras de banda de la fórmula
  * Configura los degradados de color de visualización (LUT: tablas de consulta)
  * Establece valores de umbral y modos de recorte
  * Crea fórmulas de índices personalizadas
* **Nota**: Los índices no se calculan para las cámaras monocromáticas LATTICE M3M de una sola banda; los índices multibanda no están definidos en una sola banda. Esto no afecta a Survey3 ni a LATTICE M3C.

<!-- SCREENSHOT-NEEDED: Project Settings > Index section with one index added and expanded: the filter dropdown, the formula dropdown open showing preset names, the coloured channel circles above the rendered formula, and the "+ Add LUT" button below it. -->

Cada índice que añadas se representa matemáticamente, con un círculo de color por cada ranura de banda: rojo = Red, verde = Green, azul = Blue, naranja = Orange, cian = Cyan, morado = NIR, magenta = RE. Arrastra un círculo desde la fila situada encima de la fórmula hasta una ranura para vincularlo; haz doble clic en una ranura vinculada para borrarla. El índice solo se calcula una vez que todas las ranuras que utiliza la fórmula tengan un canal.

### Fórmulas personalizadas (función de Chloros+)

* **Tipo**: Matriz de definiciones de fórmulas personalizadas
* **Disponibilidad**: Requiere iniciar sesión con una suscripción válida a Chloros+.
* **Descripción**: Te permite crear y guardar fórmulas de índices multiespectrales personalizadas utilizando operaciones matemáticas con bandas. Las fórmulas personalizadas se guardan con la configuración de tu proyecto y se pueden utilizar igual que los índices integrados.
* **Cómo crearlas**:
  1. En el panel de configuración de índices, abre la calculadora de fórmulas personalizadas
  2. Escribe la fórmula utilizando los **símbolos de ranura de banda**, no los nombres de las bandas
  3. Guarda la fórmula con un nombre descriptivo; a continuación, aparecerá en la parte inferior del menú desplegable de fórmulas, y podrás arrastrar los círculos de canal de tu cámara a sus ranuras exactamente igual que con un preajuste integrado
* **Sintaxis de la fórmula**:
  * Ranuras de banda: `x`, `y`, `z`, `a`, `b`, `c`: seis posiciones que puedes asignar a canales reales arrastrándolas
  * Operadores: `+`, `-`, `*`, `/`, `^` y `()` para agrupar
  * Funciones: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
* **¿Por qué símbolos y no nombres de bandas?**: una fórmula escrita como `(y-x)/(y+x)` funciona con cualquier cámara, ya que la asignación mediante «arrastrar y soltar» determina si `y` es el filtro NIR de 850 nm de un filtro RGN o el NIR de 808 nm de un filtro OCN. Los ajustes predefinidos integrados se almacenan de la misma manera; véase [Fórmulas del índice multiespectral](multispectral-index-formulas.md) para ver la forma exacta de los símbolos de las 27 fórmulas.
* **Dónde funcionan**: las fórmulas personalizadas se guardan con la configuración del proyecto y pueden utilizarse tanto en el [Área de pruebas de índices/LUT](../image-viewer-gui/index-lut-sandbox.md) como en el procesamiento.**no** son compatibles con la lista de nombres CLI/SDK `--indices`, que solo amplía los 22 nombres de preajustes integrados.***

## Exportación

Estos ajustes controlan el formato y la calidad de las imágenes procesadas que se exportan.

### Formato de imagen calibrada

* **Tipo**: Selección desplegable
* **Opciones**:
  * **TIFF (16 bits)**: formato TIFF sin comprimir de 16 bits
  * **TIFF (32 bits, porcentaje)**: TIFF de 32 bits en coma flotante con valores de reflectancia expresados en porcentajes
  * **PNG (8 bits)** - Formato PNG comprimido de 8 bits
  * **JPG (8 bits)** - Formato JPEG comprimido de 8 bits
* **Predeterminado**: TIFF (16 bits)
* **Descripción**: Selecciona el formato de archivo para guardar las imágenes procesadas y calibradas. Las imágenes exportadas se guardan en una subcarpeta específica para cada formato dentro de la carpeta de cada cámara (`tiff16`, `tiff32`, `png8`, `jpg8`), con una carpeta `<Product>_Images/` por producto. Los archivos exportados conservan el nombre del archivo de origen; la carpeta, y no un sufijo del nombre de archivo, es lo que identifica el producto.
* **Recomendaciones de formato**:
  * **TIFF (16 bits)**: Recomendado para análisis científicos y flujos de trabajo profesionales. Conserva la máxima calidad de los datos sin artefactos de compresión. Ideal para el análisis multiespectral y el procesamiento posterior en software SIG.
  * **TIFF (32 bits, porcentaje)**: Ideal para flujos de trabajo que requieren valores de reflectancia expresados en porcentajes (0-100 %). Ofrece la máxima precisión para mediciones radiométricas.
  * **PNG (8 bits)**: Adecuado para la visualización en la web y la visualización general. Archivos de menor tamaño con compresión sin pérdidas, pero con un rango dinámico reducido.
  * **JPG (8 bits)**: Tamaños de archivo más pequeños, ideal únicamente para vistas previas y visualización en la web. Utiliza compresión con pérdida, lo que no es adecuado para el análisis científico.
* **Nota**: La radiancia de LATTICE siempre se exporta como TIFF de 32 bits en formato flotante, independientemente de esta configuración.***

## Guardar plantilla de proyecto

Esta función te permite guardar la configuración actual de tu proyecto como una plantilla reutilizable.

* **Tipo**: Campo de texto + botón «Guardar»
* **Descripción**: Introduce un nombre descriptivo para tu plantilla de configuración y haz clic en el icono de guardar. La plantilla almacenará toda la configuración actual de tu proyecto (detección de objetivos, opciones de procesamiento, índices, y formato de exportación) para poder reutilizarlas fácilmente en proyectos futuros. Las plantillas se guardan en la carpeta `Project Templates/`, dentro de la carpeta de guardado de tu proyecto, y también se pueden seleccionar o exportar desde el menú principal (*Seleccionar plantilla* / *Guardar plantilla* / *Exportar plantilla*).
* **Casos de uso**:
  * Crear plantillas para diferentes sistemas de cámaras (RGB, multiespectral, NIR)
  * Guardar configuraciones estándar para tipos de cultivos específicos o flujos de trabajo de análisis
  * Compartir ajustes coherentes entre todo el equipo
* **Cómo utilizarla**:
  1. Configura todos los ajustes que desees para el proyecto
  2. Introduce un nombre para la plantilla (por ejemplo, «RedEdge Survey3 NDVI Estándar»)
  3. Haz clic en el icono de guardar
  4. Ahora la plantilla se puede cargar al crear nuevos proyectos

***

## Carpeta de guardado de proyectos

Esta configuración especifica dónde se guardan por defecto los nuevos proyectos.

* **Tipo**: Visualización de la ruta del directorio + botón «Editar»
* **Por defecto (Windows)**: `C:\Users\[Username]\Chloros Projects`
* **Predeterminado (Linux)**: `~/Chloros Projects`
* **Descripción**: Muestra el directorio predeterminado actual donde se crean los nuevos proyectos Chloros. Haz clic en el icono de edición para seleccionar otro directorio. La modificación se almacena como una sola línea de texto en `~/.chloros/working_directory.txt` —en Windows, es decir, `C:\Users\<Username>\.chloros\working_directory.txt`—. Si ese archivo no existe o apunta a una ruta que ya no existe, Chloros recurre a la configuración predeterminada anterior. El CLI lee y escribe el mismo archivo, por lo que `chloros-cli` y la interfaz gráfica de usuario siempre coinciden en la ubicación de los proyectos se encuentran.
* **Las plantillas de proyecto** se encuentran en una subcarpeta `Project Templates/` de este directorio.
* **Cuándo cambiarlo**:
  * Configúralo en una unidad de red para la colaboración en equipo
  * Cámbialo a una unidad con más espacio de almacenamiento para conjuntos de datos de gran tamaño
  * Organiza los proyectos por año, cliente o tipo de proyecto en diferentes carpetas
* **Nota**: Cambiar esta configuración solo afecta a los proyectos NUEVOS. Los proyectos existentes permanecen en sus ubicaciones originales.***

## Persistencia de la configuración

Un proyecto Chloros es una **carpeta**. Toda la configuración del proyecto se guarda en `project.json` dentro de ella; el hardware conectado se memoriza junto con ella en `cameras.json` y `sensors.json`, por lo que al volver a abrir un proyecto también se vuelven a conectar sus cámaras y sensores de luz. Al volver a abrir un proyecto, todos los ajustes se restauran exactamente tal y como los dejaste. Los proyectos guardados también se pueden controlar sin interfaz gráfica mediante `chloros-cli project` o el archivo `open_project` de SDK.

### Jerarquía de ajustes

Los ajustes se aplican en el siguiente orden:

1. **Valores predeterminados del sistema**: valores predeterminados integrados definidos por Chloros
2. **Configuración de la plantilla**: si se carga una plantilla al crear un proyecto
3. **Configuración del proyecto guardado**: configuración guardada con el archivo del proyecto
4. **Ajustes manuales**: cualquier cambio que se realice durante la sesión actual

### Configuración y procesamiento de imágenes

Los ajustes de procesamiento se leen al iniciar una ejecución de procesamiento. Cambiar un ajuste no altera de forma retroactiva los productos que ya se encuentran en el disco; es necesario volver a ejecutar el procesamiento para aplicar los nuevos ajustes. Hay algunos ajustes que nunca afectan al procesamiento:

* Resolución de las miniaturas de las imágenes (solo visualización)
* Guardar plantilla del proyecto
* Guardar carpeta del proyecto

***

## Referencia de claves de configuración

Para la automatización (CLI `--config`, SDK `configure`, o al leer `project.json` directamente), estas son las claves exactas que se encuentran en `Project Settings`:

| Ruta de la clave | Tipo | Valor por defecto |
| --- | --- | --- |
| `Display → Image Thumbnail Resolution` | `"512" \| "1024" \| "2048" \| "full"` | `"512"` |
| `Target Detection → Minimum calibration sample area (px)` | número 0-10000 | `25` |
| `Target Detection → Minimum Target Clustering (0-100)` | número 0-100 | `60` |
| `Processing → Vignette correction` | bool | `true` |
| `Processing → Reflectance calibration / white balance` | booleano | `true` |
| `Processing → Export sensor response` | booleano | `true` |
| `Processing → Export vignette corrected` | bool | `true` |
| `Processing → Export debayered` | bool | `true` |
| `Processing → Export preview` | bool | `true` |
| `Processing → Export radiance` | bool | `true` |
| `Processing → Export reflectance` | bool | `true` |
| `Processing → Array alignment` | bool | `true` |
| `Processing → Array alignment crop` | bool | `true` |
| `Processing → Array alignment interpolation` | `"Bilinear" \| "Nearest" \| "Cubic"` | `"Bilinear"` |
| `Processing → Debayer method` | `"Standard (Fast, Medium Quality)" \| "Texture Aware (Slow, Highest Quality)"` | Estándar |
| `Processing → Minimum recalibration interval` | número 0-3600 | `0` |
| `Processing → Light sensor timezone offset` | número -12..12 | `0` |
| `Processing → Apply PPK corrections` | booleano | `false` |
| `Processing → DAQ cap id` | ID del perfil de límite o `"auto"` | `"auto"` |
| `Processing → Target reflectance source` | `"auto" \| "target" \| "daq"` | `"auto"` |
| `Index → Add index` | lista de configuraciones de índice | `[]` |
| `Export → Calibrated image format` | `"TIFF (16-bit)" \| "TIFF (32-bit, Percent)" \| "PNG (8-bit)" \| "JPG (8-bit)"` | `"TIFF (16-bit)"` |

Las claves `Array alignment` se escriben la primera vez que se renderiza la sección «Array Alignment» o cuando una llamada de automatización las establece. Mientras estén ausentes, el proceso utiliza los mismos valores que se muestran arriba (`true`, `true`, bilineal), por lo que un proyecto .json sin ellas se comporta de forma idéntica a uno que las incluye.

### Claves almacenadas en `project.json` sin control en el panel de configuración

Estas se encuentran en el mismo árbol `Project Settings` y son leídas por el procesamiento, pero no encontrarás ningún widget para ellas en la barra lateral:

| Ruta de la clave | Tipo | Por defecto | Establecida por |
| --- | --- | --- | --- |
| `Processing → LATTICE input level` | `"auto" \| "raw" \| "debayered" \| "processed"` | `"auto"` | `chloros-cli process --input-level`, SDK `input_level=`. Anula la forma en que se interpretan los archivos TIFF de entrada de LATTICE; `auto` lo deduce a partir de la etiqueta XMP de cada archivo `Chloros:ProcessingLevel` y del número de canales. Se ignora para las capturas Survey3 `.raw`. No es un ajuste de la interfaz gráfica de usuario a propósito: la opción «auto» es la correcta en todos los casos normales. |
| `Processing → Target reflectance dir` | cadena de ruta | `""` | `chloros-cli process --target-reflectance-dir`, o el destino del proyecto API |
| `Processing → Target reflectance config` | diccionario con clave el número de serie de la cámara | `{}` | Registro de un objetivo dentro del fotograma (modo `fixed_block` / `fixed_strip` / `aruco`) |
| `Processing → DAQ-U log path` | cadena de ruta | `""` | SDK `process_folder(daq_log_path=…)`. Apunta a una grabación `.daq` o a una carpeta que los contenga |
| `Target Detection → Minimum calibration target squares` | número | `4` | Valor predeterminado heredado; sin control ni indicador CLI |
| `UI → Grid thumbnail size` | número | `160` | El control deslizante de zoom de las miniaturas propio de la cuadrícula de imágenes |

Dos preferencias del visor se almacenan **en el nivel superior de `project.json`**, fuera por completo de `Project Settings`, ya que se trata de el estado de visualización y no de ajustes de procesamiento:

| Ruta clave | Tipo | Predeterminado | Establecido por |
| --- | --- | --- | --- |
| `viewer_display → gsd_bin` | entero 1–256 | `1` | El control GSD (px) de la pestaña de imagen — véase [Abrir una imagen a pantalla completa](../image-viewer-gui/opening-an-image-full-screen.md) |

***

## Prácticas recomendadas

1. **Empieza con los valores predeterminados**: Los ajustes predeterminados funcionan bien para la mayoría de los sistemas de cámaras MAPIR y los flujos de trabajo habituales.
2. **Crea plantillas**: Una vez que hayas optimizado los ajustes para un flujo de trabajo o una cámara específicos, guárdalos como plantilla para garantizar la coherencia entre proyectos.
3. **Prueba antes de procesar todo**: Cuando experimentes con nuevos ajustes, pruébalos en un pequeño subconjunto de imágenes antes de procesar todo el conjunto de datos.
4. **Documenta tus ajustes**: Utiliza nombres descriptivos para las plantillas que indiquen el sistema de cámara, el tipo de procesamiento y el uso previsto (por ejemplo, «Survey3\_RGB\_NDVI\_Agricultura»).
5. **Selección del formato de exportación**: Elige el formato de exportación en función de tu uso final:
   * Análisis científico → TIFF (16 bits o 32 bits)
   * Procesamiento SIG → TIFF (16 bits)
   * Visualización rápida → PNG (8 bits)
   * Compartir en la web → JPG (8 bits)

***

Para obtener más información sobre los índices multiespectrales en Chloros, consulte la página [Fórmulas de índices multiespectrales](multispectral-index-formulas.md).

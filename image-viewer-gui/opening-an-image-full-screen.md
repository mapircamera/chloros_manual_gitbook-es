# Abrir una imagen a pantalla completa

<figure><img src="../.gitbook/assets/image (34).png" alt=""><figcaption><p>Una imagen abierta a pantalla completa, con el selector de capas en la esquina superior derecha</p></figcaption></figure>

El visor de imágenes Chloros es la interfaz a pantalla completa para visualizar, inspeccionar y medir tus imágenes. Es donde se leen los **valores reales de los píxeles** — DN por canal, porcentaje de reflectancia o radiancia en W/m²/sr/nm — en lugar de la vista previa estirada que muestra la pantalla.

## Acceder al visor de imágenes

### Desde el explorador de archivos

1. Abre la pestaña **Explorador de archivos** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Haz clic en cualquier **miniatura** de la [cuadrícula de imágenes](image-grid.md)
3. La imagen se abre a pantalla completa en la pestaña **Visor de imágenes**

La imagen se abre en el producto que mostraba la cuadrícula. Si la cuadrícula está configurada en `RAW (Reflectance)`, esa será la capa en la que se acceda.

### Abrir la barra lateral del Visor de imágenes

Haz clic en el icono del **Visor de imágenes** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> en la barra lateral izquierda para desplegar el panel de análisis. Contiene, de arriba abajo:

* el nombre de la imagen y el modelo de su cámara
* el botón **Exportar/Guardar imagen(es)** (solo cuando hay un índice o una LUT activos)
* las casillas de selección **Índice**y**LUT** y el panel de configuración del índice; véase [Área de pruebas de índice/LUT](index-lut-sandbox.md)
* el panel **Valores del cursor**: lectura por canal, histograma de la capa y el control GSD***

## Navegación y zoom

### Navegar por las imágenes

* **Imagen siguiente**: el botón → o la tecla**→** (flecha derecha)
* **Imagen anterior**: el botón ← o la tecla**←** (flecha izquierda)
* **Ir a una imagen concreta**: vuelve a la cuadrícula y haz clic en su miniatura

El zoom y el desplazamiento se mantienen al pasar de una imagen a otra, por lo que puedes recorrer un conjunto de imágenes sin salir de la misma parte del fotograma.

### Zoom

El zoom se controla con la **rueda del ratón**, en incrementos del 15 %, y se fija al cursor: el punto situado debajo del puntero permanece debajo del mismo. El rango está limitado por el tamaño de la imagen y de la ventana: no se puede alejar el zoom más allá del ajuste a la ventana, y el límite superior viene determinado por la resolución nativa de la imagen.

No hay teclas específicas para el zoom en el visor a pantalla completa. (En la cuadrícula, **Ctrl + `+` / `−`** cambia el tamaño de las miniaturas, lo cual es un control distinto).

### Desplazamiento al ampliar

Haz clic con el botón izquierdo del ratón sobre la imagen, mantén pulsado y arrastra. El desplazamiento está limitado, por lo que la imagen no se puede arrastrar fuera de la pantalla.

### Inspección píxel a píxel con un zoom elevado

Una vez que el aumento efectivo supera **60×**, Chloros dibuja un recuadro resaltado alrededor del píxel individual mostrado bajo el cursor y muestra un valor flotante junto a él.

El aumento «efectivo» tiene en cuenta el tamaño del bloque GSD: con un tamaño de bloque de 8, el recuadro aparece con un aumento de 7,5× en lugar de 60×, ya que un píxel mostrado equivale ya a 8 × 8 píxeles de origen. Si se reduce el zoom por debajo del umbral, el recuadro desaparece.

### Atajos de teclado

| Tecla                             | Dónde       | Acción                              |
| ------------------------------- | ----------- | ----------------------------------- |
| **→**                           | Pantalla completa | Imagen siguiente                          |
| **←**                           | Pantalla completa | Imagen anterior                      |
| **Ctrl + R**                    | Pantalla completa | Restablecer el índice/el entorno de pruebas de LUT         |
| **Ctrl + `+`**/**Ctrl + `=`** | Cuadrícula        | Miniaturas más grandes (4 px por pulsación)  |
| **Ctrl + `−`**                  | Cuadrícula        | Miniaturas más pequeñas (4 px por pulsación) |***

## Valores del cursor

Al pasar el cursor por encima de la imagen, el panel **Valores del cursor** muestra el valor de cada canal situado debajo de él.

{% hint style="success" %}
**Estos son los valores reales del archivo.** El lienzo que se ve en pantalla es una vista previa ampliada de 8 bits y no puede proporcionar esos valores, por lo que Chloros toma muestras del archivo real del producto para mostrar la lectura. Por eso, un fotograma sin procesar de 12 bits muestra valores superiores a 255, y una capa de radiancia float32 muestra unidades físicas.
{% endhint %}

### Qué significan las columnas

El panel se adapta a la capa que se está visualizando:

| Capa que se está visualizando              | Columnas mostradas    | Notas                                                                                           |
| ---------------------------------- | ---------------- | ----------------------------------------------------------------------------------------------- |
| Reflectancia                        | **DN**y**%** | El porcentaje se calcula con la escala propia de ese archivo — véase más abajo                                      |
| Radiancia                           | **W/m²/sr/nm**   | Valores físicos de tipo flotante; no hay columna de DN, ya que un DN carece de sentido en este caso                           |
| Sin procesar / Sin debayering / vista previa / JPG    | **DN**           | Números digitales enteros                                                                         |
| Exportaciones de reflectancia porcentual de 32 bits | Solo **%**       | El valor flotante almacenado no es un DN, por lo que redondearlo a un entero daría como resultado un valor sin sentido como `0` o `1` |

Cada fila está etiquetada con el nombre del canal correspondiente al filtro de tu cámara — `Red / Green / NIR` para RGN, `Orange / Cyan / NIR` para OCN, `NIR / Green / Blue` para NGB, `Red / Green / Blue` para RGB, y el nombre de la banda única para las cámaras RE, NIR y las cámaras mono M3M. Cada etiqueta lleva un punto de color que coincide con los círculos de canal utilizados en el editor de fórmulas de índice.

Las imágenes **de índice y LUT** guardadas son un caso especial: contienen componentes de mapa de colores en lugar de bandas espectrales, por lo que sus filas se etiquetan como `Red / Green / Blue` (o `Index` para un archivo de índice de un solo canal) en lugar de con los nombres de los filtros de la cámara.

Cuando un índice está activo en el entorno de pruebas, aparece una fila adicional debajo de los canales que muestra el **valor del índice** en la posición del cursor, junto con el nombre del índice y un punto blanco que coincide con su marcador en el histograma.

### El porcentaje de reflectancia utiliza la escala propia de cada archivo

{% hint style="warning" %}
**No des por sentado que 65535 = 100 %.** Chloros almacena la reflectancia en diferentes escalas dependiendo de la cámara que la haya generado, y el visor determina cuál es la correcta para cada archivo.
{% endhint %}

| Fuente                  | DN equivalente a una reflectancia de 1,0 | Cómo se identifica                                                                                                                               |
| ----------------------- | ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **LATTICE**(M3C / M3M) |**32768**                      | Etiqueta XMP `Chloros:PixelScale=32768` incluida en cada exportación de reflectancia de LATTICE. El margen de 2× permite que el archivo contenga valores de ρ superiores a 1,0 sin recorte |
| **Survey3**|**65535**                      | No hay etiqueta de escala XMP Chloros: la calibración Survey3 escribe ρ × dtype-max y recorta en 1,0                                                               |

El visor, el entorno de pruebas de índice/LUT y la exportación de índices resuelven la escala mediante una única implementación, por lo que el valor que se lee en el cursor es el mismo que utiliza el cálculo del índice.

Dos consecuencias que conviene conocer:

* Un **porcentaje de 32 bits**TIFF almacena DN/65535 como un número flotante, y un**8 bits** PNG/JPG almacena DN × 255/65535; el visor convierte ambos valores de nuevo antes de mostrar el porcentaje.
* Hay un caso que no se puede recuperar: una **exportación TIFF de 8 bits de una captura con origen de 8 bits** se recorta al rango 0–255 en lugar de reescalarse, y deliberadamente no lleva ninguna etiqueta de escala. Para esos archivos, el panel muestra únicamente el DN, sin columna de porcentajes. Esta es la respuesta sincera, no se trata de un error.***

## El histograma de la capa

Debajo de las filas del cursor hay un histograma en tiempo real de la capa que estás viendo, en **256 intervalos**. Por defecto, dibuja una curva combinada, ponderada según `(R + 2G + B) / 4` —el mismo espacio de medición que utilizan los histogramas de la cámara LATTICE—. Al activar**RGB**, se sustituye por curvas por canal en los colores de los canales, mezcladas de forma aditiva para que las superposiciones sigan siendo legibles. Las capas monocromáticas siempre trazan la curva única.

El eje horizontal está en la unidad propia de la capa:

| Capa       | Unidad del eje  | Máximo del eje                                               |
| ----------- | ---------- | ---------------------------------------------------------- |
| Reflectancia | porcentaje    | 125 % — el margen del producto permite un ρ superior a 1,0           |
| Radiancia    | W/m²/sr/nm | El pico propio del fotograma, redondeado al alza a dos cifras significativas |
| Datos de 8 bits  | DN         | 255                                                        |
| Datos de 12 bits | DN         | 4095                                                       |
| Datos de 16 bits | DN         | 65535                                                      |

Cuando el eje está en DN y se sitúa en uno de esos tres límites máximos, Chloros también conoce la profundidad de bits de lo que estás viendo.

Hay tres botones situados encima del histograma:

| Botón     | Predeterminado | Efecto                                                                                                                                                                                                                                                                                   |
| ---------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **CURSOR** | Activado | Dibuja líneas de referencia en el histograma en los valores exactos que se muestran en las filas superiores, para que puedas ver dónde se sitúa el píxel bajo el cursor en la distribución del fotograma. En el modo RGB hay un marcador por canal en su propio color; en caso contrario, aparece un único marcador blanco en el valor combinado |
| **ÍNDICE**| Activado      | Solo aparece mientras hay un índice activo. Cambia el histograma de las bandas de origen a la**distribución de valores del índice**, con los dos umbrales de recorte representados como líneas discontinuas naranjas y el valor del índice del cursor como una línea blanca                                                          |
| **RGB**| Desactivado     | Cambia de la curva combinada a las curvas por canal. En un sensor monocromo, este botón muestra**MONO** y está desactivado, ya que solo hay un canal que mostrar                                                                                                                                  |

El histograma se calcula a partir de los **bloques que se ven**, no en los píxeles originales que hay detrás de ellos: al cambiar el tamaño de bloque GSD, la distribución se vuelve a calcular, de modo que el histograma, el marcador del cursor y la imagen mostrada siempre coinciden.***

## Tamaño de bloque GSD

En la parte inferior del panel se encuentra el control **GSD (px)**: un cuadro de texto, un control deslizante de**1 a 256**y un botón**RESET**.

Este control reduce la resolución de la imagen _mostrada_ promediando un bloque de N × N píxeles de origen en un solo píxel mostrado. `1` es la resolución nativa.

* Afecta a **la vista a pantalla completa, las miniaturas de la cuadrícula, la lectura del cursor y ambos histogramas**: todo lo que muestra la imagen se ajusta a la misma resolución de fondo.
* Es **solo para visualización**. El procesamiento y la exportación no se ven afectados. La única excepción es deliberada: una exportación [Index/LUT Sandbox](index-lut-sandbox.md) guarda lo que estás viendo, por lo que conserva el tamaño de bloque actual, y el panel de exportación te avisa cuando el tamaño del bloque es superior a 1.
* El valor se almacena **por proyecto** como `viewer_display.gsd_bin` en `project.json`, por lo que se conserva al cerrar y volver a abrir el programa.
* La lectura del cursor indica el bloque, no el píxel de origen, siempre que el tamaño del bloque sea superior a 1; el valor mostrado es la media del bloque situado bajo tu cursor.

{% hint style="info" %}
**¿Por qué «tamaño de bloque» y no centímetros por píxel?** Una cifra en cm/px requiere una altura sobre el suelo. Los datos EXIF de un fotograma individual contienen la altitud GPS sobre el nivel medio del mar, no sobre el terreno al que apuntaba, por lo que Chloros no imprimirá una distancia al suelo que no pueda calcular con precisión. El tamaño del bloque en píxeles de origen es la misma solución alternativa que utilizan las herramientas de nubes de MAPIR cuando se desconoce la distancia de muestreo al suelo.
{% endhint %}

***

## Tipos de imagen que se pueden visualizar

El menú desplegable de capas situado en la esquina superior derecha del visor muestra todas las versiones de la imagen actual. Las entradas que aparecen dependen de la cámara y de lo que se haya procesado; consulta [Capas de imagen](image-layers.md) para ver la lista completa y cómo funciona el menú desplegable.

### Survey3

* **JPG**: el archivo de vista previa propio de la cámara
* **RAW (Original)**: el `.RAW` original, sin correcciones y con el efecto «debayering» aplicado para su visualización
* **RAW (Target)**: un fotograma identificado como contenedor de un objetivo de calibración
* **RAW (reflectancia)**: el producto de reflectancia calibrado (65535 = ρ 1,0)
* **Con corrección de viñeteado**/**Respuesta del sensor**: el producto de reserva sin calibrar
* **Con balance de blancos**: el producto con balance de blancos
* **RAW (índice `<INDEX>`)**y**LUT `<INDEX>`**: imágenes de índice calculadas

### LATTICE

Las capturas de LATTICE utilizan el mismo menú desplegable, con los nombres de los niveles del proceso:

| Capa                 | Qué contiene                                                        |
| --------------------- | -------------------------------------------------------------------- |
| **RAW (Original)**    | El fotograma RAW de origen tal y como se ha capturado                                     |
| **RAW (sin bayering)**   | La imagen lineal sin bayering                                           |
| **RAW (vista previa)**     | La vista previa en pantalla: estiramiento en colores falsos para cámaras multiespectrales |
| **Con balance de blancos**    | La vista previa en pantalla para las cámaras maestras RGB (balance de blancos + gamma)   |
| **RAW (Radiancia)**    | Radiancia espectral en Float32, en W/m²/sr/nm                              |
| **RAW (Reflectancia)** | Reflectancia en uint16, 32768 = ρ 1,0                                    |

La radiancia y la reflectancia son exclusivas del modo multiespectral: una cámara maestra RGB no dispone de radiometría por banda, por lo que no se generan esas capas para ella.

***

## Índice y aplicación de LUT

Aplica índices multiespectrales y tablas de consulta (LUT) de color desde la barra lateral:

1. Abre el **Visor de imágenes** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> en la barra lateral
2. Marca **Índice**

3. Elige el filtro de tu cámara y una fórmula de índice; a continuación, arrastra los círculos de los canales a las ranuras de la fórmula
4. Añade una LUT y selecciona un degradado, umbrales y un modo de recorte
5. Lee los valores en la posición del cursor y guarda el resultado con **Exportar/Guardar imagen(es)**Consulta [Entorno de pruebas de índices/LUT](index-lut-sandbox.md) para ver el tutorial completo.***

## Solución de problemas

### La imagen no se abre

**Posibles causas**: el archivo se ha movido o eliminado tras la importación; el producto nunca se guardó; no hay memoria suficiente para una imagen muy grande.**Qué hacer**:

1. Comprueba que el archivo de la capa siga existiendo en el árbol de salida del proyecto
2. Abre el archivo en un visor externo para confirmar que está intacto
3. Cierra otras aplicaciones para liberar memoria

### La imagen aparece en negro, en blanco o con colores muy distorsionados

**Posibles causas**: el estiramiento de la pantalla no tiene con qué trabajar (un fotograma casi constante); una capa float32 con valores inusuales; un índice que no ha generado datos válidos.**Qué hacer**:

1. Lee los valores del cursor: si todos los canales están en cero o cerca de cero, el problema está en los datos, no en la visualización
2. Comprueba el histograma: un único pico en un extremo indica que el fotograma está recortado o está vacío
3. Consulta el registro de procesamiento de la ejecución que generó la capa

### Los valores parecen incorrectos

**Posibles causas**: estás en una capa diferente a la que crees; estás comparando un porcentaje con un DN sin procesar; estás comparando un archivo LATTICE con un archivo Survey3 utilizando el mismo divisor.**Qué hacer**:

1. Confirma la capa seleccionada en el menú desplegable: las unidades del panel se ajustan a la capa.
2. Para la reflectancia, utiliza la columna **%** en lugar de dividir tú mismo el valor DN; si debes dividir, utiliza el valor `Chloros:PixelScale` de ese archivo (32768 para LATTICE; si no aparece, significa 65535 para Survey3)
3. Vuelve a establecer el tamaño de bloque GSD en 1; por encima de 1, se lee un promedio de bloque, no un píxel
4. Comprueba que la calibración de reflectancia se haya ejecutado realmente para ese fotograma; un producto de reserva no calibrado (Respuesta del sensor / Corregido por viñeteado) no es reflectancia

***

## Próximos pasos

* [**Capas de imagen**](image-layers.md): el nombre de cada capa, cuando existe, y el significado de sus valores
* [**Área de pruebas de índices/LUT**](index-lut-sandbox.md): crea, ajusta y exporta visualizaciones de índices
* [**Marcadores de mapa**](map-markers.md): el mismo conjunto de imágenes en un mapa
* [**Fórmulas de índices multiespectrales**](../project-settings/multispectral-index-formulas.md): la referencia de los índices

Para conocer el flujo de trabajo de procesamiento, consulta [Procesamiento de imágenes (GUI)](../processing-images-gui/adding-files-to-a-project.md).

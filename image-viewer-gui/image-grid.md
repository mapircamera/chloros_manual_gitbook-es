# Cuadrícula de imágenes

Tras importar imágenes a un proyecto, las verás dispuestas en una cuadrícula en el área principal. La cuadrícula es donde eliges **qué versión de cada imagen estás viendo**: los botones situados encima permiten cambiar todas las miniaturas a la vez entre los archivos originales y cada producto procesado.

## Tamaño de las miniaturas

Utiliza el control deslizante de zoom situado en la esquina superior derecha para ajustar el tamaño de las miniaturas de las imágenes. El control deslizante va de **64 px a 1200 px**.

* **Ctrl + rueda del ratón** también permite cambiar la escala de las miniaturas.
* **Ctrl + `+`**/**Ctrl + `=`**y**Ctrl + `−`** aumentan el tamaño en 4 píxeles por pulsación. El rango de ajuste del teclado se detiene en 64 píxeles en el extremo más pequeño y, en el extremo más grande, en el tamaño que permita encajar exactamente dos miniaturas por fila en la ventana actual.
* El tamaño que elijas se guarda con el proyecto (`UI → Grid thumbnail size` en `project.json`, por defecto `160`), por lo que al volver a abrir el proyecto se restablecerá.

<figure><img src="../.gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>La *resolución* de las miniaturas es un ajuste independiente del *tamaño* de las mismas: consulta **Visualización → Resolución de miniaturas de imagen** en [Configuración del proyecto](../project-settings/project-settings.md) (por defecto, 512 px en el lado más largo). El tamaño es la dimensión con la que se dibuja el mosaico; la resolución es la cantidad de detalle que se obtiene para rellenarlo.***

## La barra de herramientas de la cuadrícula

La fila de botones situada encima de la cuadrícula tiene hasta tres grupos, de izquierda a derecha:

1. **Por disparador / Por cámara**: modo de agrupación. Aparece únicamente en proyectos que contengan capturas de LATTICE.
2. **Botones de filtro de cámara** — uno por cada cámara de LATTICE. Aparecen únicamente en el modo «Por cámara».
3. **Botones de modo de exportación/visualización** — qué producto muestra cada miniatura.

Cuando la ventana es demasiado estrecha para mostrarlos todos, los grupos se contraen de derecha a izquierda en menús desplegables al pasar el cursor por encima: primero se pliegan los botones de exportación/vista y, a continuación, los botones de cámara. El grupo contraído deja a la vista un único botón etiquetado con la opción activa en ese momento, y al pasar el cursor por encima se desplaza hacia abajo el conjunto completo. **Los botones «Por disparador» y «Por cámara» nunca se pliegan.

<!-- SCREENSHOT-NEEDED: Image grid toolbar of a LATTICE array project at full width, showing all three button groups inline: Per Trigger / Per Camera, three camera filter buttons labelled "LATT-M3M (serial)", and the export/view buttons including TIFF, RAW (Original), RAW (Radiance), RAW (Reflectance). -->

*****

## Botones de vista de exportación

Estos botones cambian las miniaturas de la cuadrícula entre distintos tipos de imagen. **Aparece un botón tan pronto como existe el producto al que hace referencia** —lo que, en el caso de los archivos de origen, significa inmediatamente tras la importación, no después del procesamiento—. Chloros vuelve a escanear los productos del proyecto mientras se está ejecutando una sesión, por lo que los botones aparecen durante el procesamiento a medida que cada producto empieza a guardarse en el disco.

### El botón principal

El botón de exportación situado más a la izquierda muestra **lo que has importado realmente**:

| Lo que has importado | Etiqueta del botón |
| --- | --- |
| Survey3 RAW+JPG | `JPG` |
| Capturas de LATTICE con una vista previa en pantalla junto al fotograma RAW | `PNG` o `TIFF`, según cuáles sean las vistas previas |
| Capturas de LATTICE en las que el archivo base **es** el fotograma RAW | *sin botón* — `RAW (Original)` ya muestra ese archivo |

En un proyecto mixto, la etiqueta sigue la extensión que utilicen la mayoría de las imágenes.

### Botones de producto

| Botón | Muestra | Cuándo aparece |
| --- | --- | --- |
| **Objetivos** | Imágenes con un objetivo de calibración detectado | Tras una ejecución en la que se hayan detectado objetivos |
| **Reflectancia** | Imágenes de reflectancia calibradas | Solo en proyectos Survey3 — Los proyectos LATTICE utilizan `RAW (Reflectance)` en su lugar, por lo que la cuadrícula nunca muestra dos botones de reflectancia |
| **Equilibrio de blancos** | El producto con equilibrio de blancos (cámaras RGB) | Tras el procesamiento |
| **Corrección de viñeteado** | La opción de reserva sin calibrar con corrección de viñeteado | Tras una sesión en la que no se pudo aplicar la calibración de reflectancia y la *corrección de viñeteado* estaba activada |
| **Respuesta del sensor** | La opción alternativa sin calibrar basada en la respuesta del sensor | Lo mismo, pero con la *corrección de viñeteado* desactivada |
| **`RAW (<INDEX> Index)`** | Un botón por cada índice calculado | Tras una ejecución con los índices configurados |
| **`<INDEX> LUT`** | Un botón por índice con mapa de colores | Tras una ejecución con una LUT configurada |
| **`<Index> <Index\|LUT> <NNN>`** | Un botón por cada ejecución de exportación del [Index/LUT Sandbox](index-lut-sandbox.md) | En el momento en que finaliza una exportación del sandbox |

### Botones de nivel de LATTICE

Los proyectos que contienen capturas de LATTICE añaden estos botones, etiquetados con el nombre del nivel en lugar del nombre del producto:

| Botón | Nivel |
| --- | --- |
| **RAW (Original)** | El fotograma RAW de origen, tal y como se importó |
| **RAW (Radiancia)** | Radiancia espectral en Float32, W/m²/sr/nm |
| **RAW (Reflectancia)** | Reflectancia en uint16, 32768 = ρ 1,0 |

`RAW (Original)` está disponible desde el momento de la importación; no requiere ningún procesamiento. Cuando una importación de LATTICE no tiene ningún botón de base (el archivo base de cada captura es su fotograma sin procesar), la cuadrícula se desplaza automáticamente al primer botón de nivel disponible para que el resaltado de la barra de herramientas coincida con lo que se está viendo.

Las exportaciones de dos niveles Chloros **no tienen botón de cuadrícula propio**:

* **Sin bayering**: la vista `RAW (Original)` ya se representa sin bayering, por lo que un segundo botón en una imagen visualmente idéntica resultaría superfluo. El producto `RAW (Debayered)` sigue guardándose en el disco y sigue pudiéndose seleccionar en el menú desplegable de capas a pantalla completa.
* **Vista previa**: en las cámaras RGB, la vista previa se registra como la capa `White Balanced`, que sí tiene un botón. En las cámaras multiespectrales se registra como `RAW (Preview)` y se puede acceder a ella desde el menú desplegable de capas a pantalla completa.

{% hint style="info" %}
Estos botones de nivel solo se muestran en proyectos que contengan realmente fotogramas LATTICE. Los proyectos Survey3 registran algunos de los mismos nombres de capas internas, y los botones se filtran para ellos, de modo que una cuadrícula Survey3 conserva su conjunto habitual `JPG / Targets / Reflectance`.
{% endhint %}

Al hacer clic en la miniatura de una cuadrícula, se abre el [Visor de imágenes](opening-an-image-full-screen.md) a pantalla completa en **el mismo producto que muestra la cuadrícula** — si la cuadrícula está configurada en `Targets`, la miniatura abre la imagen de destino exportada.

<figure><img src="../.gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: This GIF predates the LATTICE level buttons and the toolbar group separators. Reshoot on a LATTICE project cycling base -> RAW (Original) -> RAW (Radiance) -> RAW (Reflectance) -> an index button, so the new button set and the level names are visible. -->

***

## Agrupación de un proyecto LATTICE: por disparador frente a por cámara

Las capturas en matriz producen varias imágenes del mismo instante procedentes de diferentes módulos de cámara. La agrupación determina cómo las apila la cuadrícula. Ambos modos muestran barras de encabezado plegables a ancho completo; **cada grupo se abre por defecto**, y Chloros recuerda las que se cierran. El estado de plegado se gestiona por separado en cada modo, por lo que cerrar un grupo en el modo «Por cámara» no cierra nada en el modo «Por disparador».

### Por cámara (predeterminado)

Un grupo por módulo de cámara. El encabezado muestra el modelo y el número de serie de la cámara (`LATT-M3M — <serial>`), así como el recuento de fotos. Los mosaicos dentro de un grupo se ordenan cronológicamente por evento de captura.

En este modo, la barra de herramientas también cuenta con un **botón de filtro de cámara por cada cámara**, etiquetado como `MODEL (SERIAL)`. Todas las cámaras aparecen seleccionadas inicialmente; al hacer clic en un botón, se deselecciona esa cámara y se elimina su grupo de la cuadrícula. Esta es la forma más rápida de revisar una banda a lo largo de todo un vuelo.

### Por disparo

Un grupo por evento de captura: el conjunto de fotogramas que todos los módulos capturaron en el mismo disparo. El encabezado muestra la hora de captura, el número de cámaras que han contribuido y una insignia por cada modelo de cámara del grupo. Los mosaicos dentro de un grupo se ordenan por número de serie de la cámara, por lo que la misma banda se encuentra en la misma columna para cada disparo.

<!-- SCREENSHOT-NEEDED: Image grid in Per Trigger mode for a 3-camera LATTICE array, showing two consecutive trigger groups with their header bars (chevron, capture timestamp, "3 cameras", and the three model badges) and one group collapsed to show the closed state. -->
Las imágenes que no son de LATTICE en un proyecto mixto no se agrupan; se muestran como mosaicos simples después de los grupos.

***

## Las miniaturas de la cuadrícula se ajustan al tamaño de bloque GSD

Si has establecido un tamaño de bloque **GSD (px)** en la barra lateral de la pestaña de imagen, las miniaturas en cuadrícula se muestran con esa misma resolución al suelo, no solo en la vista de pantalla completa. Un tamaño de bloque de 8 significa que cada píxel mostrado es la media de un bloque de 8 × 8 píxeles de origen, en cualquier parte de la aplicación donde se muestre la imagen.

Dado que un mosaico solo tiene unos doscientos píxeles de ancho para empezar, los tamaños de bloque gruesos dejan de marcar una diferencia visible en la cuadrícula mucho antes de que lo hagan en la vista a pantalla completa: un fotograma de 4000 px dibujado en un mosaico de 160 px ya tiene aproximadamente 25 píxeles de origen por cada píxel mostrado. Consulta [Abrir una imagen a pantalla completa](opening-an-image-full-screen.md#gsd-block-size) para ver el control en sí.

***

## Páginas relacionadas

* [**Abrir una imagen a pantalla completa**](opening-an-image-full-screen.md): el visor a pantalla completa, los valores del cursor y el histograma
* [**Capas de imagen**](image-layers.md): el menú desplegable de capas dentro del visor a pantalla completa
* [**Área de pruebas de índices/LUT**](index-lut-sandbox.md): creación y exportación de visualizaciones de índices
* [**Configuración del proyecto**](../project-settings/project-settings.md): los controles de exportación que determinan qué productos están disponibles

---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# Formatos de imagen de salida

Chloros exporta los productos procesados en cuatro formatos de archivo. Selecciona el formato en la configuración del proyecto (GUI), con `--format` (CLI) o con `export_format` (SDK). La CLI y el SDK aceptan las cadenas exactas que se indican a continuación.

| Cadena de formato | Extensión | Tipo de píxel | Rango de píxeles | Notas |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(predeterminado)* | `.tif` | número digital uint16 | 0 – 65535 | Recomendado para fotogrametría / SIG. |
| `TIFF (8-bit)` | `.tif` | número digital uint8 | 0 – 255 | Para capturas totalmente de 8 bits, que solo ofrecen formatos de 8 bits: Chloros no aumenta la profundidad de bits. |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0,0 – 1,0 | 1,0 = 100 % de reflectancia. Algunas aplicaciones no pueden leer archivos TIFF de punto flotante; los archivos son más grandes. |
| `PNG (8-bit)` | `.png` | número digital uint8 | 0 – 255 | Compresión sin pérdidas, adecuada para la visualización en la web. |
| `JPG (8-bit)` | `.jpg` | número digital uint8 | 0 – 255 | Compresión con pérdida, archivos más pequeños. |

## Dónde se guardan los archivos de salida

Los archivos se guardan en la carpeta del proyecto, agrupados por cámara y, a continuación, por formato de archivo:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera (model+lens+filter)
    ├── tiff16/                          # follows --format: tiff16, tiff8, png8, jpg8, or tiff32
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one <INDEX>_Index_Images/ folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

La carpeta de la cámara es `LATT-<sensor>-<lens>-F<filter>` para LATTICE y `<model>_<filter>` (p. ej., `Survey3N_RGN`) para Survey3. **Cada producto exportado conserva el nombre del archivo de origen: la carpeta identifica el producto, no el sufijo del nombre de archivo.** Consulta [Dónde se guardan los archivos de salida](reference/cli-reference.md) en la Referencia de la CLI para conocer todas las reglas.

## Productos LATTICE (niveles de captura y exportación)

Un fotograma sin procesar de LATTICE se divide en todos los productos solicitados en una sola pasada. Cada tipo de producto tiene su propio conmutador (casillas de selección de la interfaz gráfica de usuario o comandos de la CLI: `--debayered` / `--preview` / `--radiance` / `--reflectance`, todos activados por defecto):

| Nivel | Contenido | Tipo de datos |
| --- | --- | --- |
| `raw` | Datos Bayer directamente del sensor (cámaras monocromáticas: la banda única). El procesamiento siempre parte de los datos sin procesar. | Tal y como se capturan |
| `debayered` | Desmosaico lineal: 3 canales para M3C, 1 canal en escala de grises para M3M: se elimina el desplazamiento oscuro del paquete de calibración y, a continuación, se aplica un desmosaico sensible a los bordes, sin corrección de linealidad, campo plano ni desmezcla. Una exportación `process` genera el mismo archivo que se guarda al capturar en este nivel, en cualquier modo de procesamiento. | DN lineal |
| `radiance` | Radiancia espectral absoluta procedente de la cadena radiométrica completa, en **W/m²/sr/nm**. Siempre se guarda como TIFF de 32 bits (`tiff32/Radiance_Images/`), independientemente del formato de exportación seleccionado. | float32 |
| `reflectance` | Reflectancia ρ, donde **DN 32768 = ρ 1,0 (100 %)** con margen hasta ρ 2,0. Compatible con Pix4D. | uint16 |
| `preview` | Renderizado listo para pantalla: RGB = balance de blancos + gamma; multiespectral = estiramiento de colores falsos. | pantalla de 8 bits |

### Orientación

Los archivos `raw` y `debayered` siempre conservan la orientación **nativa** del sensor; la configuración de volteo/rotación de la cámara solo se aplica a los demás productos (vista previa, radiancia, reflectancia, índice). Las capturas en matriz mantienen la orientación nativa de los archivos de cada miembro, a menos que se solicite un cambio de orientación (`--orient` en la CLI, `orient=True` en el SDK). Cada archivo registra la orientación correspondiente a sus píxeles en XMP: `ChlorosRotation`, `ChlorosFlipHorizontal`, `ChlorosFlipVertical` — y `ChlorosOrientationApplied` indica si sus propios píxeles la contienen (`false` en archivos sin procesar y sin debayering). El visor de imágenes Chloros aplica ese registro a los archivos RAW y sin bayering únicamente con fines de visualización; los archivos en sí nunca se reescriben. Un archivo sin ese registro (creado antes de la versión 1.3.0 o con otra herramienta) se muestra tal y como está almacenado.

## Máscaras de recorte

Los productos de radiancia y reflectancia de una cámara multiespectral M3C realizan una separación de bandas de 3×3. Cuando la separación produce un valor negativo en una banda, dicho valor se recorta a cero (las vistas en directo se recortan exactamente de la misma manera). Como novedad en la versión 1.3.0, un producto en el que al menos un píxel haya sido recortado incluye una **máscara de recorte** asociada que indica exactamente qué píxeles y bandas se ven afectados:

* **Dónde:**en una carpeta `clamp_masks/`**junto a** las carpetas de productos, nunca dentro de una de ellas: `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif`, y `tiff32/Radiance_Images/x.tif` → `tiff32/clamp_masks/Radiance_Images/x_clampmask.tif`.
* **Qué:** un archivo TIFF de una sola banda `uint8` en la propia cuadrícula de píxeles del producto (incluidas la orientación, la región de interés, el binning y la alineación de la matriz). El bit 0 (`mask & 1`) se establece donde se recortó la banda en el canal R del producto, el bit 1 (`& 2`) el canal G, y el bit 2 (`& 4`) el canal B. La proporción de píxeles recortados por banda también se indica en el producto como `Chloros:BandRUnmixClampFraction`, `Chloros:BandGUnmixClampFraction` y `Chloros:BandBUnmixClampFraction`.
* **Cómo lo indica el producto:** la etiqueta XMP `Chloros:UnmixClampMask` contiene la ruta de la máscara relativa a la carpeta del producto (`../clamp_masks/<product folder>/<name>_clampmask.tif`, siempre separada por `/`), `none` cuando no se ha recortado ningún píxel (no se escribe ninguna máscara) o `unavailable` cuando no se ha podido escribir una máscara. Un producto sin etiqueta nunca ha ejecutado la separación de canales (cámaras RGB y monocromáticas, niveles sin procesar y sin debayering).

Dado que ninguna carpeta de producto contiene una máscara, puedes entregar una carpeta de producto como `Reflectance_Calibrated_Images/` a Pix4D, Metashape u otro software de fotogrametría tal cual, subcarpetas incluidas; entrégale la carpeta de producto, no la carpeta de formato que hay por encima. La propia importación de Chloros omite las máscaras de recorte. La CLI las cuenta por separado: `Image products written: N (plus M clamp masks)`.

## Productos de índice

Cada índice se almacena en su propia carpeta `<INDEX>_Index_Images/` (p. ej., `NDVI_Index_Images/`).

* **Los formatos enteros**(`TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)`) almacenan un**estiramiento lineal por imagen**, no una correspondencia −1…1: DN 0 es el valor de índice más bajo de ese fotograma y la escala completa, el más alto. El archivo contiene `Chloros:IndexScaling=linear_minmax`, `Chloros:IndexMin` y `Chloros:IndexMax`, así como `value = IndexMin + DN / full_scale × (IndexMax − IndexMin)`. Dado que cada fotograma tiene sus propios valores extremos, se deben comparar los fotogramas por sus valores, no por su DN.
* **`TIFF (32-bit, Percent)`** almacena los propios valores de índice como float32, sin reescalado: la mejor opción para el trabajo cuantitativo a lo largo de un vuelo.

Como novedad en la versión 1.3.0, un producto de índices se describe a **sí mismo** en lugar de repetir las etiquetas de los datos brutos de origen. Sus etiquetas de GPS y de cámara se copian del producto a partir del cual se calculó (el producto de reflectancia, si existe) y, a continuación:

| Etiqueta | Valor |
| --- | --- |
| `Chloros:ProcessingLevel` | `index` (por lo que el archivo nunca se vuelve a importar como captura) |
| `Chloros:IndexApplied` / `Chloros:IndexExpression` | `True` / la fórmula utilizada |
| `Chloros:IndexLevel` | `reflectance` o `radiance` — a partir de qué se calculó el índice (ausente cuando no se da ninguno de los dos) |
| `Chloros:IlluminationInvariant` | `True` para un índice calculado a partir de la reflectancia; `False`, a partir de la radiancia o de los valores del sensor |
| `Chloros:ChlorosPipelineVersion` | la versión de Chloros que generó el índice |

`Chloros:PixelScale` se ha eliminado de los productos de índice; utilice `IndexMin` / `IndexMax` más arriba. La receta completa de lectura se encuentra en «Una trama de índice entero es un estiramiento de contraste por imagen» en la [Referencia de CLI](reference/cli-reference.md).

## Lectura de los valores de reflectancia de los píxeles

La reflectancia se almacena como un número digital entero, y **el DN que corresponde a ρ = 1,0 (100 % de reflectancia) depende de la cámara de origen**:

| Cámara de origen | ρ = 1,0 es DN | Cómo determinarlo |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (margen de hasta ρ 2,0) | La etiqueta XMP `Chloros:PixelScale=32768` aparece en el archivo. |
| Survey3 | `65535` (recortado en ρ 1,0) | No hay etiqueta `Chloros:PixelScale`; esa ausencia es la señal. (Los productos Survey3 sí incluyen otras etiquetas de procesamiento `Chloros:*`, como `Chloros:CalibrationApplied`, en todos los modos de procesamiento a partir de la versión 1.3.0.) |

**Lee la etiqueta XMP `Chloros:PixelScale` y divide por ella** en lugar de suponer un valor constante. La etiqueta está definida en el dominio uint16, por lo que se mantiene como `32768` en todos los formatos de salida que cambian la escala; primero hay que normalizar el tipo de datos almacenado de nuevo a uint16 (×257 desde 8 bits, ×65535 desde float32).

{% hint style="warning" %}
**Capturas de 8 bits.** Las exportaciones de una captura de 8 bits son de 8 bits. Su reflectancia se reescala y se marca como `Chloros:PixelScale=32768`, por lo que se mantiene la escala; los productos DN de 8 bits y los archivos de reflectancia creados antes de la versión 1.3.0 se recortan al rango 0–255 y no incluyen ninguna etiqueta de escala. Si la etiqueta no aparece en un archivo de reflectancia de LATTICE, no se debe suponer que existe una escala; en su lugar, se debe volver a exportar a 16 bits o 32 bits.
{% endhint %}

Para consultar las reglas completas (incluidas las etiquetas compatibles con MicaSense), véase **«Lectura de píxeles de reflectancia»** en la [Referencia de la CLI](reference/cli-reference.md).

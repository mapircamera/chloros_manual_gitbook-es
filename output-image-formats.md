---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# Formatos de imagen de salida

Chloros exporta los productos procesados en cuatro formatos de archivo. Selecciona el formato en la configuración del proyecto (GUI), con `--format` (CLI) o con `export_format` (SDK). Los parámetros CLI y SDK aceptan las cadenas exactas que se indican a continuación.

| Cadena de formato | Extensión | Tipo de píxel | Rango de píxeles | Notas |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(por defecto)* | `.tif` | número digital uint16 | 0 – 65535 | Recomendado para fotogrametría / SIG. |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0,0 – 1,0 | 1,0 = 100 % de reflectancia. Algunas aplicaciones no pueden leer archivos TIFF con números de coma flotante; los archivos son más grandes. |
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

La carpeta de la cámara es `LATT-<sensor>-<lens>-F<filter>` para LATTICE y `<model>_<filter>` (p. ej., `Survey3N_RGN`) para Survey3. **Cada producto exportado conserva el nombre del archivo de origen; la carpeta identifica el producto, no un sufijo del nombre de archivo.** Consulta [Dónde se guardan los resultados](reference/cli-reference.md) en la Referencia de CLI para conocer todas las reglas.

## Productos de LATTICE (niveles de captura y exportación)

Un fotograma sin procesar de LATTICE se distribuye a todos los productos solicitados en una sola pasada. Cada tipo de producto tiene su propio control de activación (casillas de selección de la interfaz gráfica de usuario, o CLI `--debayered` / `--preview` / `--radiance` / `--reflectance`, todos activados por defecto):

| Nivel | Contenido | Tipo de datos |
| --- | --- | --- |
| `raw` | Datos Bayer directamente del sensor (cámaras monocromáticas: la banda única). El procesamiento siempre parte de los datos sin procesar. | Tal y como se capturan |
| `debayered` | Desmosaico lineal: 3 canales para M3C, 1 canal en escala de grises para M3M. | DN lineal |
| `radiance` | Radiancia espectral absoluta procedente de la cadena radiométrica completa, en **W/m²/sr/nm**. Siempre se escribe como TIFF de 32 bits (`tiff32/Radiance_Images/`), independientemente del formato de exportación seleccionado. | float32 |
| `reflectance` | Reflectancia ρ, donde **DN 32768 = ρ 1,0 (100 %)** con margen hasta ρ 2,0. Listo para Pix4D. | uint16 |
| `preview` | Renderizado listo para visualización: RGB = balance de blancos + gamma; multiespectral = estiramiento de colores falsos. | visualización de 8 bits |

## Lectura de los valores de píxeles de reflectancia

La reflectancia se almacena como un número digital entero, y **el DN que corresponde a ρ = 1,0 (100 % de reflectancia) depende de la cámara de origen**:

| Cámara de origen | ρ = 1,0 es DN | Cómo determinarlo |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (margen de hasta ρ 2,0) | La etiqueta XMP `Chloros:PixelScale=32768` aparece en el archivo. |
| Survey3 | `65535` (recortado en ρ 1,0) | No hay etiquetas XMP `Chloros:*`; esa ausencia es la señal. |

**Lee la etiqueta XMP `Chloros:PixelScale` y divide por ella** en lugar de suponer un valor constante. La etiqueta está definida en el dominio uint16, por lo que se mantiene como `32768` en los formatos de salida que reescalan; normaliza primero el tipo de datos almacenado de vuelta a uint16 (×257 desde 8 bits, ×65535 desde float32).

{% hint style="warning" %}
**Hay un caso que, por diseño, no lleva escala alguna.** Cuando una captura de origen de 8 bits (BayerRG8) se escribe como TIFF de 8 bits, el proceso recorta el valor a 0–255 en lugar de reescalarlo, por lo que el archivo no tiene escala; Chloros omite deliberadamente `Chloros:PixelScale` en ese caso. Si la etiqueta no aparece en un archivo de reflectancia de LATTICE, no des por sentado que hay una escala; en su lugar, vuelve a exportarlo a 16 o 32 bits.
{% endhint %}

Para consultar las reglas completas (incluidas las etiquetas compatibles con MicaSense), véase **«Lectura de píxeles de reflectancia»** en la [Referencia de CLI](reference/cli-reference.md).

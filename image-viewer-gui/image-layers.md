# Capas de imagen

El **menú desplegable de capas** situado en la esquina superior derecha del visor de imágenes permite alternar entre todas las versiones de la imagen que estás viendo —desde la captura original, pasando por cada producto procesado, hasta las imágenes de índice calculadas— sin salir del visor.

## ¿Qué son las capas de imagen?

Una «capa» en Chloros es un **archivo de producto**registrado en relación con una imagen original. La importación te proporciona los archivos originales; el procesamiento añade una capa por cada producto generado en la ejecución. Los archivos exportados conservan el nombre del archivo original: es la**carpeta** la que identifica el producto, y el nombre de la capa es la etiqueta que Chloros asigna a dicha carpeta.

<!-- SCREENSHOT-NEEDED: Image Viewer full screen with the layer dropdown open on a processed LATTICE multispectral image, showing the full list: TIFF base, RAW (Original), RAW (Debayered), RAW (Preview), RAW (Radiance), RAW (Reflectance), and one RAW (NDVI Index) entry. -->

***

## La lista de capas

### Siempre presentes

| Capa | Qué es |
| --- | --- |
| **JPG**(o**PNG**/**TIFF**) | El archivo base que se incluyó con la captura. Survey3 importa un `.JPG` junto a cada `.RAW`; Las capturas de LATTICE incluyen una vista previa de pantalla de tipo PNG o TIFF. Etiquetada según lo que se importó realmente |
| **RAW (Original)** | El fotograma RAW de origen, sin correcciones y sin aplicar ningún procesamiento. Disponible desde el momento de la importación; no necesita procesamiento |

Una captura de LATTICE cuyo archivo base **sea** su fotograma RAW no tiene una entrada base separada: `RAW (Original)` ya la cubre.

### Productos de procesamiento de Survey3

| Capa | Escrita en | Existe cuando |
| --- | --- | --- |
| **RAW (Objetivo)** | — | Se identificó que el fotograma contenía un objetivo de calibración |
| **RAW (Reflectancia)** | `Reflectance_Calibrated_Images/` | La calibración de reflectancia se ha realizado con éxito en este fotograma |
| **Corregida la viñeta**| `Vignette_Corrected_Images/` | No se pudo realizar la calibración de reflectancia del fotograma**y** la *corrección de viñeta* estaba activada |
| **Respuesta del sensor**| `Sensor_Response_Images/` | No se pudo calibrar el fotograma en reflectancia**y** la *corrección de viñeteado* estaba desactivada |
| **Equilibrio de blancos** | `White_Balanced_Images/` | Se ha generado un producto con equilibrio de blancos |

{% hint style="info" %}
**La corrección de viñeteado y la respuesta del sensor son alternativas, nunca ambas a la vez.** Existe exactamente un producto de reserva sin calibrar por ejecución, para cada modelo de cámara, y el conmutador de *corrección de viñeteado* elige cuál. Véase [Configuración del proyecto](../project-settings/project-settings.md).
{% endhint %}

### Niveles de LATTICE

LATTICE captura el fan-out en estos niveles en una única pasada de procesamiento. Los que existen dependen de los controles de exportación por producto en la configuración del proyecto y de lo que se aplique a la cámara.

| Capa | Se guarda en | Se aplica a |
| --- | --- | --- |
| **RAW (desbayero)** | `Debayered_Images/` | RGB y multiespectral |
| **RAW (vista previa)** | `Preview_Images/` | Multiespectral (expansión de colores falsos) |
| **Con balance de blancos** | `Preview_Images/` | Cámaras maestras RGB — la vista previa RGB se registra con este nombre para que coincida con la capa Survey3 del mismo nombre |
| **RAW (radiancia)** | `Radiance_Images/` | Solo multiespectral |
| **RAW (radiancia)** | `Reflectance_Calibrated_Images/` | Solo multiespectral, y únicamente cuando un registro descendente `.daq` coincidente o un objetivo dentro del fotograma que haya superado el control de calidad cubra el fotograma |

Las cámaras principales RGB no disponen de radiometría por banda, por lo que la radiancia y la reflectancia se omiten en su caso como **no aplicables**; el registro lo indica en lugar de generar un error silencioso.

### Capas de índice, LUT y sandbox

| Patrón de capa | Ejemplo | De dónde procede |
| --- | --- | --- |
| **RAW (`<INDEX>` Índice)** | `RAW (NDVI Index)` | Uno por cada índice configurado en los ajustes del proyecto, calculado durante el procesamiento |
| **`<INDEX>` LUT** | `NDVI LUT` | La versión con mapa de colores de un índice |
| **Entorno de pruebas (`<Name>` `<Index\|LUT>` `<NNN>`)** | `Sandbox (NDVI LUT 003)` | Uno por cada ejecución de exportación de [Índice/LUT Sandbox](index-lut-sandbox.md) |

Si se configura el mismo nombre de índice más de una vez con ajustes diferentes, al segundo y a los siguientes se les añade un número en el nombre (`RAW (NDVI2 Index)`) para que las capas se puedan distinguir.

***

## Uso del selector de capas

1. Abre una imagen a pantalla completa haciendo clic en una miniatura de la cuadrícula
2. Haz clic en el **menú desplegable de capas** situado en la esquina superior derecha del visor
3. Selecciona una capa; la imagen se actualiza inmediatamente

El menú desplegable muestra primero **JPG, RAW (Original), RAW (Destino), RAW (Reflectancia)**, en ese orden, y a continuación enumera el resto según el orden en que se registraron los productos.

### Preferencia de capa al navegar

Al pulsar **←**/**→** se pasa a la siguiente imagen y se intenta mantenerte en la misma capa:

1. **Coincidencia exacta primero**: si la siguiente imagen tiene una capa con el mismo nombre, se muestra esa. Esto es lo que te mantiene en `RAW (NDVI Index)` mientras recorres un conjunto completo
2. **A continuación, coincidencia por tipo**: una capa de índice busca cualquier capa de índice, una LUT busca cualquier LUT, una de reflectancia busca una de reflectancia, una de destino busca una de destino, una original busca una original y una base busca una base
3. **A continuación, solo para capas de exportación**: se mantiene el nombre aunque la lista de capas aún no se haya actualizado, ya que el archivo ya existe en el disco. Esto es lo que te permite revisar los productos mientras la ejecución aún los está creando.
4. **En caso contrario**: la primera capa disponible, que normalmente es la imagen base.

Los archivos sidecar `.daq` y `.csv` del proyecto se omiten al navegar con las teclas de flecha, por lo que al recorrer las imágenes nunca se llega a un registro del sensor de luz.

El zoom y el desplazamiento también se mantienen al pasar de una imagen a otra, lo que facilita la comparación «antes y después» de la misma posición en el campo.

***

## Comprender los valores de píxel por capa

El [panel «Valores del cursor»](opening-an-image-full-screen.md#cursor-values) muestra el valor real por canal situado bajo el cursor, en la unidad en la que está almacenada la capa. Sus columnas varían según la capa:

| Capa | Unidad mostrada | Notas |
| --- | --- | --- |
| Base (JPG / PNG / Vista previa de TIFF) | DN, 0–255 | Valores de visualización, con corrección de gamma en RGB. Solo para inspección visual |
| RAW (Original) | DN | Valores digitales sin procesar del sensor. El eje del histograma indica la profundidad: 255 (8 bits), 4095 (12 bits) o 65535 (16 bits) |
| RAW (sin bayering) | DN | Lineal, sin ampliación en pantalla |
| RAW (Vista previa) / Balance de blancos | DN | Producto de visualización: ampliado o con corrección de gamma. No apto para mediciones |
| RAW (Radiancia) | **W/m²/sr/nm** | Radiancia física en Float32. Sin columna DN |
| RAW (reflectancia) | DN **y %** | Porcentaje calculado con la escala propia de ese archivo — véase más abajo |
| Exportaciones de índice / LUT / sandbox | Valor de índice, o componentes RGB | Un archivo de índice de un solo canal indica el valor de índice; un archivo LUT con mapeo de colores indica los componentes Red/Green/Blue |

### Reflectancia: la escala es por archivo

{% hint style="warning" %}
**«Dividir por 65 535» solo es correcto para Survey3.** La reflectancia de LATTICE se almacena a una escala diferente, y mezclar los dos divisores es la forma más habitual de obtener valores de reflectancia que son exactamente la mitad de lo que deberían ser.
{% endhint %}

| Fuente | DN que equivale a una reflectancia de 1,0 | Identificado por |
| --- | --- | --- |
| **LATTICE**(M3C / M3M) |**32768** | La etiqueta XMP `Chloros:PixelScale=32768` incluida en cada exportación de reflectancia de LATTICE. El margen de 2× significa que ρ por encima de 1,0 es representable en lugar de recortarse |
| **Survey3**|**65535** | Si no hay etiqueta de escala XMP Chloros, la calibración Survey3 escribe ρ × dtype-max y recorta el valor en 1,0 |

Para SIG y scripts: lee el valor `Chloros:PixelScale` del archivo y divide por él. Si la etiqueta no está presente, el archivo tiene una escala Survey3 (65535). El visor, el entorno de pruebas de índices/LUT y la exportación de índices resuelven la escala de esta misma forma, por lo que el número que se lee en el cursor es el número utilizado en los cálculos del índice.

Almacenamiento específico del formato además de esa escala:

* **TIFF (32 bits, porcentaje)** almacena DN / 65535 como un número de coma flotante
* **PNG (8 bits)**y**JPG (8 bits)** almacenan DN × 255 / 65535
* Una **exportación TIFF de 8 bits de una captura con origen de 8 bits** se recorta a 0–255 en lugar de reescalarla, y deliberadamente no lleva ninguna etiqueta de escala. El panel muestra el DN solo para esos archivos, sin columna de porcentaje

### Rangos de valores de índice

| Familia de índices | Rango típico | Lectura |
| --- | --- | --- |
| Diferencia normalizada (NDVI, GNDVI, NDRE, ENDVI…) | −1 a +1 | La vegetación sana suele estar entre 0,4 y 0,9; el suelo desnudo cerca de 0; el agua, en valores negativos |
| Ajustada al suelo (SAVI, OSAVI, MSAVI2…) | aproximadamente de −1 a +1,5 | Lectura similar a la de NDVI con el fondo del suelo suprimido |
| Relación (GRVI, GCI, MSR, CIRE…) | sin límite por encima | Las relaciones crecen sin límite a medida que la banda del denominador tiende a cero |
| EVI / LAI | de 0 a ~1, de 0 a ~3,5 | Las nubes y otros píxeles saturados hacen que ambos valores se salgan del rango; enmascáralos primero |

Consulta [Fórmulas de índices multiespectrales](../project-settings/multispectral-index-formulas.md) para ver la fórmula exacta que hay detrás de cada preajuste.

***

## Flujos de trabajo habituales

### Comparación antes/después

1. Selecciona **RAW (Original)** y fíjate en el viñeteado y en los valores sin calibrar
2. Cambia a **RAW (Reflectancia)**

3. Compara: se ha eliminado el viñeteado y se han calibrado los valores. Mantén el zoom y el desplazamiento fijos, para que estés viendo la misma zona del terreno

### Revisar un índice en todo un conjunto

1. Abre la primera imagen procesada y selecciona la capa de índice
2. Pulsa **→** repetidamente: la capa de índice te seguirá de imagen en imagen
3. Observa el histograma en la barra lateral a medida que avanzas: un fotograma cuya distribución presente saltos merece una revisión más detallada

### Verificar los objetivos de calibración

1. Selecciona **RAW (Objetivo)** en un fotograma objetivo
2. Confirma que el objetivo sea claramente visible y se haya detectado
3. Pasa al siguiente fotograma objetivo: la capa de objetivos te seguirá

### Comprueba la precisión de los valores de reflectancia

1. Selecciona **RAW (Reflectancia)**

2. Lee la columna**%** en el panel «Valores del cursor»: ya está escalada correctamente para ese archivo
3. Comprueba la coherencia con materiales conocidos en el fotograma: la vegetación sana presenta valores altos en NIR y bajos en rojo; un objetivo de calibración debería mostrar valores cercanos a su reflectancia publicada

***

## Resolución de problemas

### Una capa que esperaba no aparece en el menú desplegable

**Posibles causas**

* La imagen nunca se procesó: solo existen la capa base y `RAW (Original)`
* La opción de exportación del producto no está marcada en la configuración del proyecto
* El producto no es aplicable a esa cámara (radiancia y reflectancia en una cámara maestra RGB; cualquier índice en una cámara monocromática M3M de banda única)
* La calibración de reflectancia no tenía datos con los que trabajar —no había cobertura descendente de `.daq` ni ningún objetivo dentro del fotograma que hubiera superado el control de calidad—, por lo que el fotograma recayó en «Vignette Corrected» o «Sensor Response»

**Qué hacer**

1. Comprueba el registro de la ejecución: Chloros indica cuándo no fue posible generar un producto de exportación solicitado y por qué
2. Comprueba los controles de exportación por producto en [Configuración del proyecto](../project-settings/project-settings.md)
3. Confirma que la carpeta del producto existe en el árbol de resultados del proyecto.
4. Vuelve a procesar con el producto habilitado.

### La lista de capas parece desactualizada

Chloros vuelve a escanear las carpetas de productos del proyecto mientras se está ejecutando una tarea y corrige los registros de capas que faltan a partir de lo que realmente hay en el disco, por lo que una capa cuya exportación haya finalizado normalmente aparece por sí sola en una consulta. Salir de la imagen y volver a ella fuerza una nueva resolución.

### Los valores de reflectancia parecen ser la mitad de lo que deberían ser

Es casi seguro que estás dividiendo un archivo LATTICE por 65535. Utiliza `Chloros:PixelScale` (32768) o consulta la columna **%**, en la que ya se ha aplicado dicho valor.

### La capa de índice existe, pero la imagen está en blanco

El índice necesita bandas que tu capa no tiene; por ejemplo, un índice que lee un tercer canal aplicado a un archivo de uno o dos canales. Cambia a una capa multibanda (reflectancia o sin bayering), o elige un índice que se ajuste al filtro de la cámara.

***

## Próximos pasos

* [**Abrir una imagen a pantalla completa**](opening-an-image-full-screen.md): lectura del cursor, histograma y control del GSD
* [**Área de pruebas de índices/LUT**](index-lut-sandbox.md): visualización interactiva de índices y exportación
* [**Fórmulas de índices multiespectrales**](../project-settings/multispectral-index-formulas.md) — la referencia de índices
* [**Finalización del procesamiento**](../processing-images-gui/finishing-the-processing.md) — el árbol de carpetas de salida al que apuntan estas capas

# Finalización del procesamiento

Una vez que Chloros haya completado el procesamiento, es el momento de revisar los resultados, comprobar la calidad de la salida y preparar las imágenes procesadas para su uso en tu flujo de trabajo. Esta página te guía a través de los pasos finales y las acciones siguientes.

## Indicación de que el procesamiento ha finalizado

Cuando el procesamiento finalice con éxito, verás varios indicadores:

* ✅ **Barra de progreso**: Alcanza el 100 % de finalización
* ✅ **Registro de depuración**: Muestra las últimas líneas `[RUN-SUMMARY]` con los recuentos (imágenes, grupos de cámaras, objetivos, imágenes calibradas, archivos guardados)
* ✅ **Botón de inicio**: vuelve a estar habilitado (listo para la siguiente ejecución del procesamiento)
* ✅ **Archivos de salida**: todas las imágenes procesadas se guardan en el árbol de salida del proyecto (a continuación)

{% hint style="warning" %}
**Una ejecución que no genera imágenes se considera un error.** Si has solicitado productos de imagen y la ejecución no ha escrito ninguno, Chloros lo notifica como un fallo; el `[RUN-SUMMARY]` del nombre del registro sugiere la causa probable (no se ha importado nada, no se ha detectado ningún objetivo o se han omitido todos los productos solicitados por no ser aplicables). El equivalente en la CLI sale con un valor distinto de cero. Una ejecución deliberada solo con metadatos (todos los productos de exportación desactivados, sin índices) sigue considerándose un éxito. Consulta [la Referencia de la CLI](../reference/cli-reference.md#a-run-that-writes-no-images-fails).
{% endhint %}

***

## Localización de tus imágenes procesadas

### Abrir la carpeta de salida

1. Haz clic en el icono del **Menú principal** <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line">
(arriba a la izquierda)
2. Selecciona **«Abrir carpeta del proyecto»**

3. Se abrirá el explorador de archivos en el directorio del proyecto
4. Localiza tu proyecto por su nombre

### El árbol de salida

Los productos se guardan **en la carpeta del proyecto, agrupados por cámara y, a continuación, por formato de archivo**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per selected index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

* **Carpeta de la cámara**: `LATT-<sensor>-<lens>-F<filter>` para LATTICE (que coincide con el EXIF de la captura: `Model`), `<model>_<filter>` para Survey3 (p. ej., `Survey3N_RGN`). Dos cámaras que comparten sensor y filtro, pero que tienen objetivos diferentes, mantienen árboles de archivos separados: el viñeteado, el campo de visión y la distorsión difieren.
* **Carpeta de formato**: sigue la configuración de formato de exportación: `tiff16`, `tiff8`, `png8`, `jpg8`, o `tiff32` para TIFF (32 bits, porcentaje). La radiancia es siempre de tipo float32 y siempre se almacena en `tiff32`.
* **Carpetas de productos**:
  * `Reflectance_Calibrated_Images/` — reflectancia calibrada
  * `Debayered_Images/` — sin bayering lineal (LATTICE): se ha eliminado el desplazamiento oscuro y se ha aplicado el demosaico, sin campo plano ni desmezcla, en la orientación nativa del sensor
  * `Preview_Images/` — vista previa en pantalla (LATTICE)
  * `Radiance_Images/` — radiancia espectral en float32, W/m²/sr/nm (multiespectral LATTICE)
  * `Vignette_Corrected_Images/` **o** `Sensor_Response_Images/` — la opción de reserva sin calibrar para fotogramas sin referencia de reflectancia; existe exactamente uno de los dos por cada ejecución, elegido según el ajuste de corrección de viñeteado
  * `<INDEX>_Index_Images/` — una carpeta por cada índice seleccionado (p. ej., `NDVI_Index_Images`)
* **`clamp_masks/`** (novedad en la versión 1.3.0, solo para cámaras multiespectrales M3C de LATTICE): aparece en una carpeta de formato, junto a las carpetas de productos, cuando la desmezcla de bandas recorta los valores negativos a cero en un producto de radiancia o reflectancia. Contiene una máscara por píxel por cada producto afectado, p. ej., `tiff16/clamp_masks/Reflectance_Calibrated_Images/<name>_clampmask.tif`. Las máscaras nunca se encuentran dentro de una carpeta de producto, por lo que una carpeta de producto puede entregarse tal cual al software de fotogrametría. Véase [Máscaras de recorte](../output-image-formats.md#clamp-masks).

{% hint style="info" %}
**Cada producto exportado conserva el nombre del archivo ORIGINAL.**Una exportación de radiancia de `capture_..._raw.tif` sigue llamándose `capture_..._raw.tif`; simplemente se encuentra en `tiff32/Radiance_Images/`.**La carpeta identifica el producto, no el nombre del archivo**, por lo que si buscas `*radiance*.tif` no encontrarás nada; busca en el directorio en su lugar.
{% endhint %}

<!-- SCREENSHOT-NEEDED: Windows Explorer open on a processed project folder showing the tree: a LATT-M3C multispectral camera folder expanded with tiff16 (Reflectance_Calibrated_Images, Debayered_Images, Preview_Images, NDVI_Index_Images, clamp_masks) and tiff32 (Radiance_Images, clamp_masks) subfolders visible -->


### ¿Cuántos archivos debería haber?

No los cuentes según una fórmula: el número de resultados depende de qué productos se hayan habilitado y de cuáles se apliquen a cada cámara (por ejemplo, las cámaras RGB no tienen radiancia ni reflectancia). El recuento definitivo se encuentra en el registro: la última línea `[RUN-SUMMARY]` indica exactamente cuántos archivos se han escrito, y las líneas de ayuda explican todo lo que se ha omitido.

***

## Revisión de las imágenes procesadas

### Vista previa rápida en el Explorador de archivos

**Vista previa integrada de Windows:**

1. Accede a una carpeta de productos (p. ej., `tiff16/Reflectance_Calibrated_Images/`)
2. Selecciona un archivo de imagen
3. La vista previa aparece en el panel de vista previa del Explorador de Windows

4. Utiliza las teclas de flecha para desplazarte por las imágenes

### Vista previa en visores de imágenes externos

**Visores recomendados:**

* **QGIS**: software SIG gratuito (ideal para el análisis multiespectral georreferenciado)
* **IrfanView**: visor de imágenes rápido y ligero (compatible con TIFF)
* **Adobe Photoshop**: edición profesional (compatible con TIFF)
* **GIMP**: alternativa gratuita a Photoshop
* **Fotos de Windows**: visualización básica (puede que no sea compatible con TIFF de 16 bits)

### Vista previa en el visor de imágenes de Chloros


Utiliza el visor de imágenes integrado de Chloros para una visualización avanzada:

1. Haz clic en la miniatura de una imagen en el explorador de archivos
2. La imagen se abre en el área principal de vista previa
3. Haz clic en la pestaña **Visor de imágenes** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
de la barra lateral izquierda
4. Utiliza [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) para realizar un análisis interactivo

Consulta [Visor de imágenes](../image-viewer-gui/opening-an-image-full-screen.md) para obtener instrucciones detalladas.

***

## Lectura de los valores de reflectancia de los píxeles (SIG / Pix4D / Scripts)

La reflectancia se almacena como un valor DN entero, y **el valor DN que corresponde a ρ = 1,0 depende de la cámara de origen**:

| Fuente          | ρ = 1,0 es | Cómo determinarlo                                        |
| --------------- | ---------- | -------------------------------------------------- |
| LATTICE (M3C/M3M) | **32768** (margen de hasta ρ 2,0) | La etiqueta XMP `Chloros:PixelScale=32768` aparece en el archivo |
| Survey3          | **65535** (recortado en ρ 1,0)     | No hay etiqueta `Chloros:PixelScale` — esa ausencia es la señal (los productos de Survey3 sí llevan otras etiquetas de procesamiento `Chloros:*`) |

**Lee la etiqueta `Chloros:PixelScale` y divide por ese valor** en lugar de asumir un valor genérico de 65535; dividir la reflectancia de LATTICE por 65535 reduce a la mitad todos los valores sin avisar. Un archivo de reflectancia anterior a la versión 1.3.0 escrito en 8 bits se recortó, no se reescaló, y no tiene etiqueta de escala: vuelve a exportarlo en 16 bits o 32 bits en lugar de dividirlo. Consulta [Formatos de imagen de salida](../output-image-formats.md) para obtener más información.

***

## Metadatos que se incluyen en las exportaciones

Cada producto conserva el **bloque GPS**de la captura original y su**sub-IFD EXIF**, por lo que una
exportación incluye `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` y
`CameraSerialNumber`, además de la georreferenciación.

{% hint style="warning" %}
**Si un ortomosaico se genera a una escala absurda, comprueba primero `FocalLength`.**
Pix4D calcula la distancia entre muestras en el suelo a partir de la distancia focal más la altitud. Sin la etiqueta,
recurre a una escala totalmente errónea: en un vuelo medido de 49 capturas, un
se reconstruyó como de 47,8 km × 13 km, lo que generó una ortofoto de 455 megapíxeles compuesta en su mayor parte
por espacio vacío. La lentitud en la división en mosaicos y un archivo inesperadamente enorme son síntomas de esto, no
problemas independientes.

```bash
exiftool -FocalLength -GPSLatitude "YourProject/.../some_export.tif"
```
{% endhint %}

No se copian *todas* las etiquetas. Las etiquetas estructurales de IFD0 se omiten deliberadamente (copiarlas
corrompe la salida de LATTICE), y se excluyen `ExifImageWidth` / `ExifImageHeight` se excluyen
porque describen la captura original; de lo contrario, una exportación cuyo tamaño se hubiera modificado
reivindicaría unas dimensiones que su propia trama contradice.

Chloros también escribe sus propias etiquetas XMP `Chloros:*` que describen lo que se ha hecho con cada producto
(por ejemplo, `Chloros:ProcessingLevel`, `Chloros:CalibrationApplied`,
`Chloros:VignetteCorrected`). A partir de la versión 1.3.0, los productos de Survey3 las incluyen en todos los modos de procesamiento; los productos de índice describen el propio índice (`Chloros:ProcessingLevel=index`, la
fórmula y si se ha calculado a partir de la reflectancia o la radiancia) en lugar de repetir las
etiquetas del archivo sin procesar de origen, y los productos en carpetas con nombres de más de 260 caracteres también conservan sus metadatos. Véase [Formatos de imagen de salida](../output-image-formats.md#index-products).

***

## Revisión del registro de depuración

### Comprobación de advertencias o errores

1. Abre la pestaña **Registro de depuración** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">

2. Desplázate por los mensajes
3. Busca advertencias amarillas o errores rojos
4. Lee las líneas `[RUN-SUMMARY]` y cualquier sugerencia
5. Ponte en contacto con el servicio de asistencia de MAPIR para obtener ayuda

### Guardar el registro

Para mantener un registro del procesamiento o para enviarlo al servicio de asistencia de MAPIR:

1. Haz clic en el botón **«Copiar»**o**«Descargar»**

2. Guárdalo como archivo de texto en la carpeta del proyecto
3. Inclúyelo en la documentación del proyecto
4. Envíalo al servicio de asistencia de MAPIR si surgen problemas.

Para enviar un problema directamente a MAPIR, utiliza el icono de error (**Informar de un error**) situado en la parte superior de la pestaña «Registro de depuración»; consulta [Cómo informar de un error desde la pestaña «Registro de depuración»](monitoring-the-processing.md#reporting-a-bug-from-the-log-tab).

***

## Problemas comunes en los resultados y sus soluciones

### Problema: Faltan archivos de salida

**Posibles causas:**

* El producto no es aplicable a esa cámara (p. ej., radiancia/reflectancia para cámaras RGB; así lo indica el registro)
* Faltaba una referencia obligatoria (p. ej., reflectancia sin objetivo y sin `.daq` de radiación descendente)
* La casilla de verificación de exportación del producto estaba desactivada en la configuración del proyecto
* Se agotó el espacio en disco durante la exportación

**Soluciones:**

1. Comprueba las sugerencias `[RUN-SUMMARY]` y las líneas `[EXPORT-CHECK]` en el registro de depuración; en ellas se explican los omisiones por cámara
2. Comprueba las casillas de selección de productos de exportación en [Configuración del proyecto](adjusting-project-settings.md).
3. Comprueba que haya espacio suficiente en el disco.
4. Vuelve a procesar tras solucionar la causa.

### Problema: Bordes oscuros o brillantes (el viñeteado sigue siendo visible)

**Posibles causas:**

* Corrección de viñeteado desactivada
* La cámara o el objetivo no figuran en la base de datos de perfiles de Chloros

* Viñeteado extremo que supera la capacidad de corrección

**Soluciones:**

1. Comprueba que la corrección de viñeteado esté activada en la configuración del proyecto
2. Comprueba que se haya detectado correctamente el modelo de cámara
3. Ponte en contacto con el servicio de asistencia de MAPIR si el viñeteado persiste

### Problema: Colores o valores incorrectos

**Posibles causas:**

* No se han detectado objetivos de calibración
* Se ha seleccionado un modelo de objetivo de calibración incorrecto
* La calibración de reflectancia está desactivada
* Imágenes de los objetivos de baja calidad

**Soluciones:**

1. Comprueba que la calibración de reflectancia esté activada
2. Comprueba los mensajes de «Objetivo encontrado» en el registro de depuración.
3. Revisa la calidad de las imágenes de los objetivos.
4. Vuelve a procesar marcando los objetivos correctos.

### Problema: Los valores de NDVI parecen incorrectos

**Rangos esperados de NDVI:**

* **Agua, rocas, suelo**: de -0,1 a 0,2
* **Vegetación escasa o en mal estado**: de 0,2 a 0,4
* **Vegetación moderada**: de 0,4 a 0,6
* **Vegetación sana y densa**: de 0,6 a 0,9**Si los valores se encuentran fuera de estos rangos:**

1. Comprueba que se haya aplicado la calibración de reflectancia
2. Comprueba que se haya incluido el registro del sensor de luz
3. Comprueba que se hayan detectado los objetivos de calibración
4. Asegúrate de que se haya detectado el modelo correcto de cámara
5. Revisa el momento y las condiciones de captura de las imágenes de referencia
6. Si calculas tú mismo los índices a partir de los archivos de reflectancia, confirma que has dividido por el valor `Chloros:PixelScale` del archivo (véase más arriba)

***

## Uso de las imágenes procesadas

### Para fotogrametría / creación de ortomosaicos

**Flujo de trabajo recomendado:**

1.**Importa las imágenes de reflectancia calibradas** al software de fotogrametría:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Conserva los metadatos EXIF**: asegúrate de que se conservan los datos GPS para el geoetiquetado
3. **Flujos de trabajo calibrados**: utiliza imágenes de reflectancia para obtener precisión científica; la reflectancia de LATTICE incluye las etiquetas de calibración XMP que lee Pix4D
4. **Procesa mosaicos de índices**: crea ortomosaicos NDVI a partir de imágenes de índices individuales
5. **Exportar GeoTIFF georreferenciado**: para su uso en aplicaciones SIG

### Para el análisis SIG

**Flujo de trabajo recomendado:**

1.**Cargar en QGIS, ArcGIS o similar**

2.**Utiliza imágenes de reflectancia TIFF de 16 bits** para el análisis multibanda (divide por el valor `Chloros:PixelScale` del archivo)
3. **Utiliza imágenes de índices** (NDVI, NDRE) como capas de vegetación listas para usar
4. **Calculadora ráster**: Combina bandas para realizar análisis personalizados
5. **Exportar**: Crea mapas de clasificación, detección de cambios y mapas de estado de la vegetación

### Para análisis directo / elaboración de informes

**Flujo de trabajo recomendado:**

1.**Utiliza imágenes de índices con colores LUT** para informes visuales
2. **Extraer estadísticas**: NDVI medio por campo/parcela
3. **Series temporales**: comparar índices entre varias sesiones
4. **Generar informes**: incluir mapas, estadísticas y visualizaciones***

## Archivado y copias de seguridad

### Estrategia de copias de seguridad recomendada

**Qué guardar:**

* ✅ **Imágenes RAW/JPG originales o capturas RAW de LATTICE**: archivar en una unidad independiente o en la nube; los archivos RAW son la fuente del proceso y todo lo demás se puede regenerar a partir de ellos
* ✅ **Archivos del sensor de luz `.daq` / `.csv`**: necesarios para volver a calcular la reflectancia más adelante
* ✅ **Resultados procesados**: conservar las imágenes calibradas y los índices
* ✅ **Carpeta del proyecto** (`project.json` y archivos relacionados): contiene todos los ajustes para volver a procesar los datos si fuera necesario
* ✅ **Registro de depuración**: documenta los detalles del procesamiento
* ✅ **Imágenes de referencia de calibración**: para verificación y reprocesamiento

**Recomendaciones de almacenamiento:**

* **Copia de seguridad inmediata**: disco duro externo
* **Archivo a largo plazo**: almacenamiento en la nube (Google Drive, Dropbox, etc.)
* **Datos críticos**: conservar 2-3 copias en ubicaciones diferentes***

## Próximas ejecuciones de procesamiento

### Reutilización de la configuración del proyecto

Si vas a procesar conjuntos de datos similares en el futuro:

1. **Guarda la plantilla del proyecto** (si aún no lo has hecho)
2. **Crea un nuevo proyecto** utilizando la plantilla guardada
3. **Importa nuevas imágenes**

4.**Procesa**con la misma configuración para garantizar la coherencia

### Procesamiento por lotes de varias sesiones

Para varias sesiones o conjuntos de datos:**Opción 1: Interfaz gráfica de usuario (GUI) - Varios proyectos**

* Crea un proyecto independiente para cada sesión
* Utiliza una configuración de plantilla coherente
* Procesa uno por uno

**Opción 2: Chloros CLI (solo Chloros+)**

* Automatizar el procesamiento por lotes
* Procesar varias carpetas con scripts
* Consulta la [Documentación de la CLI](../CLI.md) y la [Referencia de la CLI](../reference/cli-reference.md)

**Opción 3: Python SDK (solo Chloros+)**

* Control mediante programación
* Integración con flujos de análisis
* Consulta la [Documentación de la API](../api-python-sdk.md) y la [Referencia del SDK](../reference/sdk-reference.md)

***

## Solución de problemas en el posprocesamiento

### Reprocesamiento con diferentes ajustes

Si los resultados no son satisfactorios:

1. Conserva las imágenes originales (no las elimines nunca)
2. Abre el mismo proyecto en Chloros
3. Ajusta la configuración en el panel «Ajustes del proyecto»
4. Vuelve a procesar: los resultados se guardan en las mismas carpetas de productos, por lo que los archivos con el mismo nombre de la ejecución anterior se sustituyen

### Procesamiento de un subconjunto de imágenes

Para volver a procesar solo imágenes específicas:

1. Crea un nuevo proyecto
2. Importa solo las imágenes que necesiten volver a procesarse
3. Utiliza la misma plantilla de configuración
4. Procesa un conjunto de datos más pequeño

### Cómo obtener ayuda

Si tienes algún problema:

* 📧 **Correo electrónico**: info@mapir.camera (incluye el registro de depuración)
* 🌐 **Asistencia técnica**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Preguntas frecuentes**: [Preguntas frecuentes](../faq.md)
* 📖 **Documentación**: [Manual de Chloros](../)

***

## Resumen: Flujo de trabajo completo

Ya has completado todo el flujo de trabajo de procesamiento de Chloros:

1. ✅ **Proyecto creado** - Consulta [Proyectos](../projects.md)
2. ✅ **Añadido archivos**: consulta [Añadir archivos](adding-files-to-a-project.md)
3. ✅ **Ajustado la configuración** - Véase [Ajustar la configuración del proyecto](adjusting-project-settings.md)
4. ✅ **Marcado los objetivos** - Véase [Seleccionar imágenes objetivo](choosing-target-images.md)
5. ✅ **Inicio del procesamiento** - Véase [Inicio del procesamiento](starting-the-processing.md)
6. ✅ **Supervisión del progreso** - Véase [Supervisión del procesamiento](monitoring-the-processing.md)
7. ✅ **Resultados revisados**: esta página**¡Tus imágenes multiespectrales calibradas y con corrección de reflectancia están listas para su análisis!**

***

## Recursos adicionales

### Funciones avanzadas

* [**Visor de imágenes**](../image-viewer-gui/opening-an-image-full-screen.md) - Visualización y análisis interactivos
* [**Entorno de pruebas de índices/LUT**](../image-viewer-gui/index-lut-sandbox.md) - Pruebas de índices personalizados
* [**Fórmulas de índices multiespectrales**](../project-settings/multispectral-index-formulas.md) - Referencia completa de índices

### Automatización e integración

* [**Documentación de CLI**](../CLI.md) - Procesamiento por lotes desde la línea de comandos
* [**Python SDK**](../api-python-sdk.md) - Automatización mediante programación
* [**Funcionalidades de Chloros+**](../#chloros) - Capacidades avanzadas de procesamiento

### Asistencia y formación

* [**Preguntas frecuentes**](../faq.md) - Respuestas a preguntas habituales
* [**Objetivos de calibración**](../calibration-targets.md) - Explicación de la calibración de la reflectancia
* [**Cámaras compatibles**](../supported-cameras.md) - Hardware compatible

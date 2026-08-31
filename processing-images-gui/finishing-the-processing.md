# Finalización del procesamiento

Una vez queChloros
haya completado el procesamiento, es el momento de revisar los resultados, comprobar la calidad de la salida y preparar las imágenes procesadas para utilizarlas en tu flujo de trabajo. Esta página te guía a través de los pasos finales y las acciones siguientes.

## Indicación de que el procesamiento ha finalizado

Cuando el procesamiento finalice con éxito, verás varios indicadores:

* ✅ **Barra de progreso**: Alcanza el 100 % de finalización
* ✅ **Registro de depuración**: Muestra las últimas líneas de `[RUN-SUMMARY]` con los recuentos (imágenes, grupos de cámaras, objetivos, imágenes calibradas, archivos guardados)
* ✅ **Botón de inicio**: vuelve a estar habilitado (listo para la siguiente ejecución de procesamiento)
* ✅ **Archivos de salida**: todas las imágenes procesadas se guardan en el árbol de salida del proyecto (a continuación)

{% hint style="warning" %}
**Una ejecución que no genera imágenes se considera un fallo.** Si has solicitado productos de imagen y la ejecución no ha generado ninguno,Chloros
lo notifica como un fallo; las entradas `[RUN-SUMMARY]` del registro indican la causa probable (no se ha importado nada, no se ha detectado ningún objetivo o se han omitido todos los productos solicitados por no ser aplicables). El equivalente enCLI
finaliza con un valor distinto de cero. Una ejecución deliberada solo con metadatos (todos los productos de exportación desactivados, sin índices) sigue considerándose un éxito. Consulta [la Referencia deCLI
](../reference/cli-reference.md#a-run-that-writes-no-images-fails).
{% endhint %}

***

## Localización de las imágenes procesadas

### Apertura de la carpeta de salida

1. Haz clic en el icono **Menú principal**<img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line">
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

* **Carpeta de la cámara**: `LATT-<sensor>-<lens>-F<filter>` para LATTICE (que coincide con el EXIF de la captura `Model`), `<model>_<filter>` para «Survey3
» (por ejemplo, `Survey3N_RGN`). Dos cámaras que comparten sensor y filtro, pero que difieren en el objetivo, mantienen árboles separados, ya que el viñeteado, el campo de visión y la distorsión varían.
* **Carpeta de formato**: sigue la configuración de formato de exportación: `tiff16`, `tiff8`, `png8`, `jpg8` o `tiff32` para «TIFF
» (32 bits, porcentaje). «Radiance» siempre es de tipo float32 y siempre se guarda en `tiff32`.
* **Carpetas de productos**:
  * `Reflectance_Calibrated_Images/` — reflectancia calibrada
  * `Debayered_Images/` — sin bayering lineal (LATTICE)
  * `Preview_Images/` — vista previa en pantalla (LATTICE)
  * `Radiance_Images/` — radiancia espectral en float32, W/m²/sr/nm (LATTICE multiespectral)
  * `Vignette_Corrected_Images/` **o** `Sensor_Response_Images/` — el valor de reserva no calibrado para fotogramas sin referencia de reflectancia; existe exactamente uno de los dos por ejecución, elegido por el ajuste de corrección de viñeteado
  * `<INDEX>_Index_Images/` — una carpeta por cada índice seleccionado (p. ej., `NDVI_Index_Images`)

{% hint style="info" %}
**Cada producto exportado conserva el nombre del archivo ORIGINAL.**Una exportación de radiancia de `capture_..._raw.tif` sigue llamándose `capture_..._raw.tif`, pero se encuentra en `tiff32/Radiance_Images/`.**La carpeta identifica el producto, no el nombre del archivo**, por lo que si buscas `*radiance*.tif` no encontrarás nada; busca en el directorio en su lugar.
{% endhint %}



<!-- SCREENSHOT-NEEDED: Windows Explorer open on a processed project folder showing the tree: a LATT-… camera folder expanded with tiff16 (Reflectance_Calibrated_Images, Debayered_Images, Preview_Images, NDVI_Index_Images) and tiff32 (Radiance_Images) subfolders visible -->
### ¿Cuántos archivos debería haber?

No los cuentes según una fórmula: el número de archivos de salida depende de qué productos se hayan habilitado y de cuáles se apliquen a cada cámara (por ejemplo, las cámarasRGB
no obtienen radiancia ni reflectancia). El recuento definitivo se encuentra en el registro: la última línea `[RUN-SUMMARY]` indica exactamente cuántos archivos se han escrito, y las líneas de ayuda explican todo lo que se ha omitido.

***

## Revisión de las imágenes procesadas

### Vista previa rápida en el Explorador de archivos

**Vista previa integrada enWindows
:**

1. Accede a una carpeta de productos (p. ej., `tiff16/Reflectance_Calibrated_Images/`)
2. Selecciona un archivo de imagen
3. La vista previa aparece en el panel de vista previa del Explorador de lWindows

4. Utiliza las teclas de flecha para desplazarte por las imágenes

### Vista previa en visores de imágenes externos

**Visores recomendados:*** **QGIS**: software SIG gratuito (ideal para el análisis multiespectral georreferenciado)
* **IrfanView**: visor de imágenes rápido y ligero (compatible con el formato «TIFF
»)
* **Adobe Photoshop**: edición profesional (compatible con el formato «TIFF
»)
* **GIMP**: alternativa gratuita a Photoshop
* **Fotos deWindows**: visualización básica (puede que no sea compatible con el formato «TIFF
» de 16 bits)

### Vista previa en el visor de imágenes deChloros


Utiliza el visor de imágenes integrado enChloros
para una visualización avanzada:

1. Haz clic en la miniatura de una imagen en el explorador de archivos
2. La imagen se abre en el área principal de vista previa
3. Haz clic en la pestaña **Visor de imágenes**<img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
en la barra lateral izquierda
4. Utiliza [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) para realizar análisis interactivos

Consulta [Visor de imágenes](../image-viewer-gui/opening-an-image-full-screen.md) para obtener instrucciones detalladas.

***

## Lectura de los valores de reflectancia de los píxeles (GIS / Pix4D / Scripts)

La reflectancia se almacena como un DN entero, y **el DN que corresponde a ρ = 1,0 depende de la cámara de origen**:

| Origen          | ρ = 1,0 es | Cómo determinarlo                                        |
| --------------- | ---------- | -------------------------------------------------- |
| LATTICE (M3C/M3M) | **32768** (margen de hasta ρ 2,0) | La etiqueta XMP `Chloros:PixelScale=32768` aparece en el archivo |
|Survey3
         | **65535** (recortado en ρ 1,0)     | No hay etiquetas XMP `Chloros:*` — esa ausencia es la señal |

**Lea la etiqueta `Chloros:PixelScale` y divida por ese valor** en lugar de suponer un valor general de 65535; dividir la reflectancia de LATTICE por 65535 reduce a la mitad todos los valores sin avisar. Hay un caso extremo que, por diseño, no lleva escala: una captura con fuente de 8 bits guardada como salida de 8 bits se recorta, no se reescala, y deliberadamente no recibe ninguna etiqueta de escala; reexporta a 16 o 32 bits en lugar de dividirla. Consulta [Formatos de imagen de salida](../output-image-formats.md) para obtener toda la información.***

## Metadatos que se incluyen en las exportaciones

Cada producto conserva el **bloque GPS**de la captura original y su**sub-IFD EXIF**, por lo que una
exportación incluye `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` y
`CameraSerialNumber`, además de la georreferenciación.

{% hint style="warning" %}
**Si un ortomosaico se genera a una escala absurda, comprueba primero `FocalLength`.**
Pix4D calcula la distancia de muestreo al suelo a partir de la distancia focal más la altitud. Sin la etiqueta,
recurre a una escala totalmente errónea: en un vuelo medido de 49 capturas, un
naranjal de 49 capturas se reconstruyó como 47,8 km × 13 km, lo que generó una ortofoto de 455 megapíxeles compuesta en su mayor parte
por espacio vacío. La lentitud en la división en mosaicos y un archivo inesperadamente enorme son síntomas de esto, no problemas
independientes.

```bash
exiftool -FocalLength -GPSLatitude "YourProject/.../some_export.tif"
```
{% endhint %}

No se copian *todas* las etiquetas. Las etiquetas estructurales de IFD0 se omiten deliberadamente (copiarlas
corrompe la salida de LATTICE), y se excluyen `ExifImageWidth` / `ExifImageHeight`
porque describen la captura original; de lo contrario, una exportación cuyo tamaño se hubiera modificado
reivindica unas dimensiones que su propia trama contradice.

***

## Revisión del registro de depuración

### Comprobar si hay advertencias o errores

1. Abre la pestaña **Registro de depuración**<img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">

2. Desplázate por los mensajes
3. Busca advertencias amarillas o errores rojos
4. Lee las líneas `[RUN-SUMMARY]` y cualquier sugerencia
5. Ponte en contacto con el servicio de asistencia deMAPIR
para obtener ayuda

### Guardar el registro

Para mantener un registro del procesamiento o para enviarlo al servicio de asistencia deMAPIR
:

1. Haz clic en el botón **«Copiar»**o**«Descargar»**

2. Guárdalo como archivo de texto en la carpeta del proyecto
3. Inclúyelo en la documentación del proyecto
4. Envíalo al servicio de asistencia deMAPIR
si surgen problemas

***

## Problemas comunes con los resultados y sus soluciones

### Problema: faltan archivos de salida

**Posibles causas:**

* El producto no es compatible con esa cámara (p. ej., radiancia/reflectancia para cámarasRGB
— así lo indica el registro)
* Faltaba una referencia necesaria (p. ej., reflectancia sin objetivo y sin `.daq` descendente)
* La casilla de exportación del producto estaba desactivada en la configuración del proyecto
* Se agotó el espacio en disco durante la exportación

**Soluciones:**

1. Comprueba las indicaciones `[RUN-SUMMARY]` y las líneas `[EXPORT-CHECK]` en el registro de depuración: explican los omisiones por cámara
2. Verifica las casillas de selección de productos de exportación en [Configuración del proyecto](adjusting-project-settings.md)
3. Comprueba que había espacio suficiente en el disco
4. Vuelve a procesar tras solucionar la causa

### Problema: Bordes oscuros o brillantes (el viñeteado sigue siendo visible)

**Posibles causas:**

* Corrección de viñeteado desactivada
* La cámara o el objetivo no figuran en la base de datos de perfiles deChloros

* Viñeteado extremo que supera la capacidad de corrección

**Soluciones:**

1. Comprueba que la corrección de viñeteado esté activada en la configuración del proyecto
2. Comprueba que se haya detectado correctamente el modelo de cámara
3. Ponte en contacto con el servicio de asistencia deMAPIR
si el viñeteado persiste

### Problema: Colores o valores incorrectos

**Posibles causas:**

* No se han detectado objetivos de calibración
* Se ha seleccionado un modelo de objetivo de calibración incorrecto
* La calibración de reflectancia está desactivada
* Imágenes de los objetivos de baja calidad

**Soluciones:**

1. Comprueba que la calibración de reflectancia esté activada
2. Comprueba los mensajes «Target found» en el registro de depuración
3. Revisa la calidad de las imágenes de los objetivos
4. Vuelve a procesar marcando los objetivos correctos

### Problema: los valores de «NDVI
» parecen incorrectos

**Rangos esperados de «NDVI
»:*** **Agua, rocas, suelo**: de -0,1 a 0,2
* **Vegetación escasa o en mal estado**: de 0,2 a 0,4
* **Vegetación moderada**: de 0,4 a 0,6
* **Vegetación sana y densa**: de 0,6 a 0,9**Si los valores se encuentran fuera de estos rangos:**

1. Comprueba que se haya aplicado la calibración de reflectancia
2. Comprueba que se haya incluido el registro del sensor de luz
3. Comprueba que se hayan detectado los objetivos de calibración
4. Asegúrate de que se haya detectado el modelo correcto de cámara
5. Revisa el momento y las condiciones de captura de la imagen objetivo
6. Si calcula usted mismo los índices a partir de archivos de reflectancia, confirme que ha dividido por el valor `Chloros:PixelScale` del archivo (véase más arriba)

***

## Uso de las imágenes procesadas

### Para fotogrametría / creación de ortomosaicos

**Flujo de trabajo recomendado:**

1.**Importa las imágenes de reflectancia calibradas** al software de fotogrametría:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Conserva los metadatos EXIF**: asegúrate de que se conserven los datos GPS para el geoetiquetado
3. **Flujos de trabajo calibrados**: utiliza imágenes de reflectancia para obtener precisión científica; la reflectancia de LATTICE incluye las etiquetas de calibración XMP que lee Pix4D
4. **Procesa mosaicos de índice**: crea ortomosaicos deNDVI
a partir de imágenes de índice individuales
5. **Exportar «GeoTIFF
» georreferenciados**: para su uso en aplicaciones SIG

### Para análisis SIG

**Flujo de trabajo recomendado:**

1.**Cargar en QGIS, ArcGIS o similar**

2.**Utilizar imágenes de reflectancia de 16 bitsTIFF
** para el análisis multibanda (dividir por el valor `Chloros:PixelScale` del archivo)
3. **Utilizar imágenes de índice** (NDVI
,NDRE
) como capas de vegetación listas para usar
4. **Calculadora ráster**: Combina bandas para realizar análisis personalizados
5. **Exportar**: Crea mapas de clasificación, detección de cambios y mapas de estado de la vegetación

### Para análisis directos y elaboración de informes

**Flujo de trabajo recomendado:**

1.**Utiliza imágenes de índices con colores LUT** para informes visuales
2. **Extraer estadísticas**: mediaNDVI
o por campo/parcela
3. **Series temporales**: comparar índices entre varias sesiones
4. **Generar informes**: incluir mapas, estadísticas y visualizaciones***

## Archivado y copias de seguridad

### Estrategia de copia de seguridad recomendada

**Qué guardar:*** ✅ **Imágenes RAW/JPG originales o capturas RAW de LATTICE**: archivarlas en una unidad independiente o en la nube; los archivos RAW son la fuente del proceso de tratamiento y todo lo demás se puede regenerar a partir de ellos
* ✅ **Archivos del sensor de luz `.daq` / `.csv`**: necesarios para volver a calcular la reflectancia más adelante
* ✅ **Resultados procesados**: conservar las imágenes calibradas y los índices
* ✅ **Carpeta del proyecto** (`project.json` y archivos relacionados): contiene todos los ajustes para volver a procesar los datos si fuera necesario
* ✅ **Registro de depuración**: documenta los detalles del procesamiento
* ✅ **Imágenes de referencia de calibración**: para verificación y reprocesamiento**Recomendaciones de almacenamiento:*** **Copia de seguridad inmediata**: disco duro externo
* **Archivo a largo plazo**: almacenamiento en la nube (Google Drive, Dropbox, etc.)
* **Datos críticos**: Conserva entre 2 y 3 copias en ubicaciones diferentes***

## Próximas ejecuciones de procesamiento

### Reutilización de la configuración del proyecto

Si vas a procesar conjuntos de datos similares en el futuro:

1. **Guarda la plantilla del proyecto** (si aún no lo has hecho)
2. **Crea un nuevo proyecto** utilizando la plantilla guardada
3. **Importar nuevas imágenes**

4.**Procesar**con la misma configuración para garantizar la coherencia

### Procesamiento por lotes de varias sesiones

Para varias sesiones o conjuntos de datos:**Opción 1: Interfaz gráfica de usuario (GUI) - Varios proyectos**

* Crear un proyecto independiente para cada sesión
* Utilizar una configuración de plantilla coherente
* Procesar de uno en uno

**Opción 2:Chloros
CLI
(soloChloros
+)**

* Automatizar el procesamiento por lotes
* Procesar varias carpetas con scripts
* Consulta la [Documentación deCLI
](../CLI.md) y la [Referencia deCLI
](../reference/cli-reference.md)

**Opción 3:Python
SDK
(soloChloros
+)**

* Control mediante programación
* Integración con flujos de trabajo de análisis
* Consulte la [Documentación deAPI
](../api-python-sdk.md) y la [Referencia deSDK
](../reference/sdk-reference.md)

***

## Resolución de problemas en el posprocesamiento

### Volver a procesar con diferentes ajustes

Si los resultados no son satisfactorios:

1. Conserva las imágenes originales (no las elimines nunca)
2. Abre el mismo proyecto enChloros
3. Ajusta la configuración en el panel «Configuración del proyecto»
4. Vuelve a procesar: los resultados se guardan en las mismas carpetas de productos, por lo que los archivos con el mismo nombre de la ejecución anterior se sustituyen

### Procesamiento de un subconjunto de imágenes

Para volver a procesar solo imágenes específicas:

1. Crea un nuevo proyecto
2. Importa solo las imágenes que necesiten volver a procesarse
3. Utiliza la misma plantilla de configuración
4. Procesa un conjunto de datos más pequeño

### Obtener ayuda

Si tienes algún problema:

* 📧 **Correo electrónico**: info@mapir.camera (incluye el registro de depuración)
* 🌐 **Asistencia técnica**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Preguntas frecuentes**: [Preguntas frecuentes](../faq.md)
* 📖 **Documentación**: [Manual deChloros
](../)

***

## Resumen: Flujo de trabajo completo

Ya has completado todo el flujo de trabajo de procesamiento deChloros
:

1. ✅ **Proyecto creado** - Consulta [Proyectos](../projects.md)
2. ✅ **Archivos añadidos** - Consulta [Añadir archivos](adding-files-to-a-project.md)
3. ✅ **Ajustado la configuración**: consulta [Ajustar la configuración del proyecto](adjusting-project-settings.md)
4. ✅ **Se han marcado los objetivos** - Véase [Selección de imágenes objetivo](choosing-target-images.md)
5. ✅ **Se ha iniciado el procesamiento** - Véase [Inicio del procesamiento](starting-the-processing.md)
6. ✅ **Progreso supervisado** - Véase [Supervisión del procesamiento](monitoring-the-processing.md)
7. ✅ **Resultados revisados** - Esta página**¡Tus imágenes multiespectrales calibradas y con corrección de reflectancia están listas para su análisis!**

***

## Recursos adicionales

### Funciones avanzadas

* [**Visor de imágenes**](../image-viewer-gui/opening-an-image-full-screen.md) - Visualización y análisis interactivos
* [**Entorno de pruebas de índices/LUT**](../image-viewer-gui/index-lut-sandbox.md) - Pruebas de índices personalizados
* [**Fórmulas de índices multiespectrales**](../project-settings/multispectral-index-formulas.md) - Referencia completa de índices

### Automatización e integración

* [**Documentación deCLI**](../CLI.md) - Procesamiento por lotes desde la línea de comandos
* [**SDK
dePython
**](../api-python-sdk.md) - Automatización mediante programación
* [**Chloros
+ Características**](../#chloros) - Funciones avanzadas de procesamiento

### Asistencia y formación

* [**Preguntas frecuentes**](../faq.md) - Respuestas a preguntas habituales
* [**Objetivos de calibración**](../calibration-targets.md) - Explicación de la calibración de reflectancia
* [**Cámaras compatibles**](../supported-cameras.md) - Hardware compatible

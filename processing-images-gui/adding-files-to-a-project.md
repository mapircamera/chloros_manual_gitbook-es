# Añadir archivos a un proyecto

Una vez que hayas creado o abierto un proyecto en Chloros, el siguiente paso es añadir tus imágenes multiespectrales para comenzar el procesamiento. La pestaña «Explorador de archivos» (<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">) facilita la importación de imágenes y la gestión de tu conjunto de datos.

## Acceder al explorador de archivos

1. Abre o crea un proyecto en Chloros
2. Haz clic en el icono **Explorador de archivos** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> en la barra lateral izquierda
3. El panel del Explorador de archivos mostrará la lista de archivos de tu proyecto

{% hint style="info" %}
**Tipos de archivo compatibles**:

* **Survey3W / Survey3N**: pares RAW+JPG e imágenes JPG (se recomienda RAW+JPG)
* **LATTICE**: capturas `.tif` / `.tiff` — grabadas mediante el control de cámara Chloros o mediante un concentrador LATTICE
* **Datos del sensor de luz**: Grabaciones `.daq` (DAQ-U/M/E) y registros de luz descendente DAQ-M `.csv` — importados junto con las imágenes para realizar la calibración de la reflectancia
{% endhint %}

***

## Añadir imágenes a tu proyecto

Hay dos formas principales de añadir imágenes a tu proyecto:

### Método 1: Añadir archivos

Utiliza esta opción para importar archivos de imagen individuales o una pequeña selección de archivos.

1. Haz clic en el botón **«Añadir archivos»** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> situado en la parte superior del panel del explorador de archivos
2. Navega hasta la carpeta que contiene tus imágenes
3. Selecciona uno o varios archivos de imagen (mantén pulsada la tecla **Ctrl** para seleccionar varios archivos)
4. Haz clic en **«Abrir»** para importar los archivos seleccionados

### Método 2: Añadir carpeta

Utiliza esta opción para importar todas las imágenes de una carpeta de una sola vez. Puedes seleccionar **varias carpetas** en un mismo cuadro de diálogo.

1. Haz clic en el botón **«Añadir carpeta»** <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> situado en la parte superior del panel «Explorador de archivos»
2. Navega hasta la carpeta o carpetas que contengan las imágenes de tu sesión de captura y selecciónalas
3. Haz clic en **«Seleccionar carpeta»** para importar todas las imágenes compatibles

{% hint style="info" %}
**Se notifican los archivos que no se cargan.** Si una carpeta contiene archivos que Chloros reconoce pero no puede cargar, aparecerá una advertencia; las imágenes no desaparecerán de la cuadrícula sin previo aviso.
{% endhint %}

***

## Importación de carpetas de capturas de LATTICE

Las capturas de LATTICE se guardan con **una subcarpeta por cada nivel de exportación**; por ejemplo, `raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/` — junto con el archivo correspondiente de flujo descendente `.daq` en la raíz:

```
output/
├── raw/           capture_<timestamp>_SN<serial>_raw.tif
├── debayered/     capture_<timestamp>_SN<serial>_debayered.tif
├── preview/       capture_<timestamp>_SN<serial>_display.tif
└── *.daq          the downwelling reading matched to the capture
```

**Selecciona «Añadir carpeta» en la raíz de las capturas** (`output/` arriba). Cuando la carpeta seleccionada no contiene imágenes en sí misma, pero tiene subcarpetas, Chloros se adentra en ellas automáticamente; las subcarpetas de ese nivel y la carpeta raíz `.daq` se recogen todas de una sola vez.**Cómo se importan las capturas:*** Cada captura se importa como una **única imagen**, agrupada por captura (no una entrada por nivel). Los demás niveles de la misma captura aparecen como modos de visualización de esa única imagen.
* **El procesamiento siempre comienza a partir del fotograma sin procesar.** Los demás niveles se pueden visualizar, pero solo `raw` pasa por el proceso; volver a procesar un producto ya procesado aplicaría las correcciones por duplicado, por lo que Chloros se descarta. Una exportación reimportada nunca puede ocupar la ranura de datos sin procesar de una captura.
* Una carpeta de capturas guardada **sin** datos sin procesar se muestra con normalidad, pero el procesamiento la omite y así se indica en el registro. (El indicador CLI `--input-level` puede forzar un punto de entrada para este caso; véase [la Referencia de CLI](../reference/cli-reference.md#what-a-captures-folder-looks-like).)**Las sesiones del concentrador LATTICE** se importan de la misma manera: selecciona «Añadir carpeta» en la carpeta de la sesión copiada del hub (contiene `raw/` y `previews/`), junto con cualquier registro de flujo descendente de DAQ-M `.csv`. Si la calibración de la cámara o del DAQ aún no está almacenada en la caché de tu equipo, Chloros la obtiene automáticamente por número de serie al importarla (requiere conexión a Internet una sola vez).***

## Cómo interpretar la tabla del explorador de archivos

Una vez importadas las imágenes, aparecen en una tabla con las siguientes columnas:

### Nombre del archivo

* Nombre de archivo original de la cámara
* Mantiene la convención de nomenclatura de la cámara (p. ej., IMG\_0001.RAW o capture\_20260816\_101500\_SN213800234\_raw.tif)

### Marca de tiempo

* Fecha y hora en que se capturó la imagen
* Extraída de los metadatos EXIF de la imagen
* Se utiliza para la correspondencia de sensores de luz, la sincronización PPK y la programación de objetivos de calibración

### Modelo de cámara

* Configuración de la cámara y del filtro detectada automáticamente
* Ejemplos de Survey3: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* Ejemplos de LATTICE: LATT-M3M-L41-F550, LATT-M3C-L87-FRGN
* Se utiliza para aplicar los perfiles de procesamiento correctos

### Columna de objetivos (casilla de selección)

* Marque esta casilla para las imágenes que contengan objetivos de calibración
* Cuando se marque al menos una imagen, **solo se escanearán las imágenes marcadas** en busca de objetivos
* Consulte [Selección de imágenes objetivo](choosing-target-images.md) para obtener más detalles

### Visualización de los metadatos de las imágenes

Al hacer clic en el botón de alternancia situado en la esquina superior derecha, encima de la tabla, se muestran los metadatos de la imagen seleccionada en el área de la cuadrícula de imágenes.

<figure><img src="../.gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

***

## Archivos del sensor de luz en tu proyecto

* Los archivos `.daq` y `.csv` aparecen en la lista del explorador de archivos, pero no son imágenes en las que se pueda hacer clic: proporcionan la irradiancia descendente para la calibración de la reflectancia.
* Cada archivo `.daq`/`.csv` importado aparece en **Configuración del proyecto → Sensor de luz DAQ**, donde puedes revisar la corrección de la tapa difusora aplicada a cada archivo. Consulte [Ajustar la configuración del proyecto](adjusting-project-settings.md).
* Los registros que realice en la pestaña **Sensores de luz** se añaden automáticamente al proyecto abierto; no es necesario importarlos manualmente.***

## Gestión de archivos en su proyecto

### Eliminación de archivos

Para eliminar imágenes no deseadas de tu proyecto:

1. Selecciona una o varias imágenes en la tabla del explorador de archivos
2. Haz clic en el botón **«Eliminar seleccionados»** <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">
3. Confirma la eliminación (los archivos no se borran del disco, solo se eliminan del proyecto)

### Ordenar y filtrar

* **Ordenar por columna**: haz clic en cualquier encabezado de columna para ordenar las imágenes
* **Ordenar por fecha y hora**: útil para organizar secuencias de capturas cronológicas
* **Filtro por modelo de cámara**: agrupa las imágenes por tipo de cámara si utilizas varias cámaras***

## Vista previa de la imagen

### Visualización de la imagen completa

Haz clic en cualquier miniatura de imagen del explorador de archivos para mostrarla en el área principal de vista previa:

1. La imagen aparece en el panel central de vista previa
2. Utiliza los controles de zoom para examinar los detalles de la imagen
3. Navega entre las imágenes utilizando las teclas de flecha

### Navegación rápida

* **Imagen anterior**: haz clic en la flecha izquierda o pulsa la tecla ←
* **Imagen siguiente**: haz clic en la flecha derecha o pulsa la tecla →
* **Acercar/alejar**: utiliza la rueda del ratón o los botones de zoom
* **Desplazamiento**: haz clic y arrastra sobre la imagen cuando estés acercado***

## Gestión de archivos duplicados

Chloros detecta automáticamente e ignora los archivos duplicados:

* Se omiten los archivos con nombres idénticos
* Evita el procesamiento doble accidental
* Se muestra un mensaje de advertencia cuando se detectan duplicados

{% hint style="warning" %}
**Importante**: No cambies el nombre ni modifiques los archivos de imagen originales antes de importarlos. Chloros se basa en los nombres de archivo y los metadatos originales para un procesamiento correcto.
{% endhint %}

***

## Conjuntos de datos de cámaras mixtas

Si su proyecto contiene imágenes de varias cámaras MAPIR:

1. Chloros detecta automáticamente cada modelo de cámara: Survey3, LATTICE o una combinación de ambos
2. Cada tipo de cámara se procesa con su perfil de calibración adecuado
3. El explorador de archivos muestra el modelo de cámara en la columna «Modelo de cámara»
4. Cada cámara obtiene su propio árbol de carpetas de salida al procesarse

**Ejemplos de escenarios**: configuración de doble cámara Survey3W RGN + Survey3N OCN, o una matriz LATTICE con un módulo maestro RGB y varios módulos de banda estrecha***

## Prácticas recomendadas

### Organizar antes de importar

* Mantener las imágenes de los objetivos de calibración en la misma carpeta que las imágenes del levantamiento
* Guarda los archivos de los sensores de luz `.daq` / `.csv` de cada sesión de captura junto con las imágenes de dicha sesión
* Mantén la estructura de carpetas original de tu cámara, tarjeta SD o concentrador
* No mezcles conjuntos de datos de diferentes sesiones en un mismo proyecto

### Denominación de archivos

* Conserva los nombres de archivo originales de la cámara (IMG\_0001.RAW, capture\_..., etc.)
* No cambies el nombre de los archivos antes de importarlos
* Los nombres originales contienen metadatos importantes

### Imágenes de referencia para la calibración

* Incluye siempre entre 1 y 2 imágenes de referencia para la calibración por sesión (Survey3; en el caso de LATTICE, puede sustituirse por un registro DAQ —véase [Elección de imágenes de referencia](choosing-target-images.md))
* Captura las imágenes de referencia antes y después de la sesión de captura
* Coloca las imágenes de referencia en las mismas condiciones de iluminación que el área de captura
* Marca las imágenes de referencia utilizando la casilla de selección «Target»

***

## Problemas habituales y soluciones

### Las imágenes no aparecen tras la importación

**Posibles causas:**

* Formato de archivo no compatible (consulte la lista de tipos compatibles en la parte superior de esta página)
* Las imágenes proceden de cámaras que no son del modelo MAPIR (véase [Cámaras compatibles](../supported-cameras.md))
* Archivo dañado o transferencia incompleta desde la tarjeta SD

**Solución**: Comprueba la compatibilidad del formato de archivo y del modelo de cámara, y revisa la advertencia de carga de archivos para identificar exactamente cuáles han fallado

### No se ha detectado el modelo de cámara

**Posibles causas:**

* Metadatos EXIF modificados
* Imágenes editadas en un software externo
* Transferencia de archivos incompleta

**Solución**: Vuelve a importar los archivos originales sin modificar desde la cámara o la tarjeta SD

### Faltan las marcas de tiempo

**Posibles causas:**

* El reloj de la cámara no está configurado correctamente
* Los datos EXIF han sido eliminados por un software externo

**Solución**: Comprueba que la configuración de la hora de la cámara fuera correcta durante la captura

### El proyecto reabierto informa de que faltan archivos

Si los archivos de origen se han movido o eliminado desde la última vez que se abrió el proyecto, el código Chloros te indica **qué** archivos faltan, en lugar de abrirse en una cuadrícula en blanco. Restaura los archivos en sus rutas originales o elimina las entradas que faltan y vuelve a importarlos.***

## Próximos pasos

Una vez importados los archivos:

1. **Revisa la lista de archivos**: asegúrate de que todas las imágenes se hayan cargado correctamente
2. **Comprueba los modelos de cámara**: verifica que la detección de la cámara sea correcta
3. **Marca las imágenes de destino**: consulta [Selección de imágenes de destino](choosing-target-images.md)
4. **Ajusta la configuración**: configura las opciones de procesamiento en [Configuración del proyecto](adjusting-project-settings.md)
5. **Inicia el procesamiento**: consulta [Iniciar el procesamiento](starting-the-processing.md)

Para obtener información detallada sobre la configuración del proyecto, consulta [Ajustar la configuración del proyecto](adjusting-project-settings.md).

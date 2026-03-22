# Añadir archivos a un proyecto

Una vez que hayas creado o abierto un proyecto en Chloros, el siguiente paso es añadir tus imágenes multiespectrales para comenzar el procesamiento. La pestaña «Explorador de archivos»<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> facilita la importación de imágenes y la gestión de su conjunto de datos.

## Acceder al explorador de archivos

1. Abra o cree un proyecto en Chloros
2. Haga clic en el icono **Explorador de archivos** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> en la barra lateral izquierda
3. El panel del explorador de archivos mostrará la lista de archivos de su proyecto

{% hint style="info" %}
**Tipos de archivo compatibles**: Chloros admite archivos de imagen RAW+JPG y JPG de las cámaras MAPIR, Survey3W y Survey3N. Se recomienda utilizar únicamente RAW+JPG.
{% endhint %}

***

## Añadir imágenes a tu proyecto

Hay dos formas principales de añadir imágenes a tu proyecto:

### Método 1: Añadir archivos

Utiliza esta opción para importar archivos de imagen individuales o una pequeña selección de archivos.

1. Haz clic en el botón **«Añadir archivos»** <img src="../.gitbook/assets/image.png" alt="" data-size="line"> situado en la parte superior del panel del explorador de archivos
2. Navega hasta la carpeta que contiene tus imágenes
3. Selecciona uno o varios archivos de imagen (mantén pulsada la tecla **Ctrl** para seleccionar varios archivos)
4. Haz clic en **«Abrir»** para importar los archivos seleccionados

### Método 2: Añadir carpeta

Utiliza esta opción para importar todas las imágenes de una carpeta de una sola vez.

1. Haz clic en el botón **«Añadir carpeta»** <img src="../.gitbook/assets/image (1).png" alt="" data-size="line"> situado en la parte superior del panel del explorador de archivos
2. Navega hasta la carpeta que contiene las imágenes de tu sesión de captura y selecciónala
3. Haz clic en **«Seleccionar carpeta»** para importar todas las imágenes compatibles de esa carpeta***

## Explicación de la tabla del explorador de archivos

Una vez importadas las imágenes, aparecen en una tabla con las siguientes columnas:

### Nombre del archivo

* Nombre de archivo original de la cámara
* Mantiene la convención de nomenclatura de la cámara (p. ej., IMG\_0001.RAW)

### Marca de tiempo

* Fecha y hora en que se capturó la imagen
* Extraída de los metadatos EXIF de la imagen
* Se utiliza para la sincronización PPK y la detección de objetivos de calibración

### Modelo de cámara

* Configuración de la cámara y del filtro detectada automáticamente
* Ejemplos: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* Se utiliza para aplicar los perfiles de procesamiento correctos

### Columna de objetivos (casilla de verificación)

* Marque esta casilla para las imágenes que contengan objetivos de calibración
* Acelera considerablemente la detección de objetivos durante el procesamiento
* Consulte [Selección de imágenes objetivo](choosing-target-images.md) para obtener más detalles

### Visualización de metadatos de la imagen

Al hacer clic en el botón de alternancia situado en la esquina superior derecha, encima de la tabla, se muestran los metadatos de la imagen seleccionada en el área de la cuadrícula de imágenes.

<figure><img src="../.gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

***

## Gestión de archivos en su proyecto

### Eliminación de archivos

Para eliminar imágenes no deseadas de su proyecto:

1. Seleccione una o varias imágenes en la tabla del explorador de archivos
2. Haga clic en el botón **«Eliminar seleccionados»** <img src="../.gitbook/assets/image (2).png" alt="" data-size="line"> .
3. Confirme la eliminación (los archivos no se borran del disco, solo se eliminan del proyecto).

### Ordenar y filtrar

* **Ordenar por columna**: haga clic en cualquier encabezado de columna para ordenar las imágenes.
* **Ordenar por fecha y hora**: útil para organizar secuencias de captura cronológicas.
* **Filtro por modelo de cámara**: agrupe las imágenes por tipo de cámara si utiliza varias cámaras.***

## Vista previa de la imagen

### Visualización de la imagen completa

Haga clic en cualquier miniatura de imagen en el Explorador de archivos para mostrarla en el área de vista previa principal:

1. La imagen aparece en el panel de vista previa central
2. Utilice los controles de zoom para examinar los detalles de la imagen
3. Navegue entre las imágenes utilizando las teclas de flecha

### Navegación rápida

* **Imagen anterior**: Haga clic en la flecha izquierda o pulse la tecla ←
* **Imagen siguiente**: Haga clic en la flecha derecha o pulse la tecla →
* **Acercar/alejar**: Utilice la rueda del ratón o los botones de zoom
* **Desplazamiento**: Haga clic y arrastre sobre la imagen cuando esté ampliada***

## Gestión de archivos duplicados

Chloros detecta e ignora automáticamente los archivos duplicados:

* Se omiten los archivos con nombres idénticos
* Evita el procesamiento doble accidental
* Se muestra un mensaje de advertencia cuando se detectan duplicados

{% hint style="warning" %}
**Importante**: No renombre ni modifique sus archivos de imagen originales antes de importarlos. Chloros se basa en los nombres de archivo y metadatos originales para un procesamiento adecuado.
{% endhint %}

***

## Conjuntos de datos de cámaras mixtas

Si su proyecto contiene imágenes de varias cámaras MAPIR:

1. Chloros detecta automáticamente cada modelo de cámara
2. Cada tipo de cámara se procesa con su perfil de calibración adecuado
3. El explorador de archivos muestra el modelo de cámara en la columna «Modelo de cámara»
4. El procesamiento aplica los ajustes correctos para cada tipo de cámara

**Ejemplo**: Survey3W RGN + Survey3N OCN configuración de doble cámara***

## Prácticas recomendadas

### Organizar antes de importar

* Mantenga las imágenes de referencia de calibración en la misma carpeta que las imágenes del levantamiento
* Mantenga la estructura de carpetas original de su cámara o tarjeta SD
* No mezcle conjuntos de datos de diferentes sesiones en un mismo proyecto

### Nombres de archivos

* Conserve los nombres de archivo originales de la cámara (IMG\_0001.RAW, etc.)
* No renombre los archivos antes de importarlos
* Los nombres originales contienen metadatos importantes

### Imágenes de referencia de calibración

* Incluya siempre 1-2 imágenes de referencia de calibración por sesión
* Capture las referencias antes y después de la sesión de captura
* Coloque las referencias en las mismas condiciones de iluminación que el área de captura
* Marque las imágenes de referencia utilizando la casilla de verificación «Target» para acelerar el procesamiento

***

## Problemas comunes y soluciones

### Las imágenes no aparecen tras la importación

**Posibles causas:**

* Formato de archivo no compatible (solo RAW+JPG y JPG de cámaras MAPIR)
* Las imágenes proceden de cámaras que no son MAPIR (véase [Cámaras compatibles](../supported-cameras.md))
* Archivo dañado o transferencia incompleta desde la tarjeta SD

**Solución**: Comprueba la compatibilidad del formato de archivo y del modelo de cámara

### No se detecta el modelo de cámara

**Posibles causas:**

* Metadatos EXIF modificados
* Imágenes editadas en software externo
* Transferencia de archivos incompleta

**Solución**: Vuelve a importar los archivos originales sin modificar desde la cámara o la tarjeta SD

### Faltan marcas de tiempo

**Posibles causas:**

* El reloj de la cámara no está configurado correctamente
* Datos EXIF eliminados por software externo

**Solución**: Comprueba que la configuración de la hora de la cámara fuera correcta durante la captura***

## Próximos pasos

Una vez importados los archivos:

1. **Revisa la lista de archivos**: asegúrate de que todas las imágenes se hayan cargado correctamente
2. **Comprueba los modelos de cámara**: verifica que la detección de la cámara sea correcta
3. **Marca las imágenes de destino**: consulta [Selección de imágenes de destino](choosing-target-images.md)
4. **Ajusta la configuración**: configura las opciones de procesamiento en [Configuración del proyecto](adjusting-project-settings.md)
5. **Inicie el procesamiento**: consulte [Inicio del procesamiento](starting-the-processing.md)

Para obtener información detallada sobre la configuración del proyecto, consulte [Ajuste de la configuración del proyecto](adjusting-project-settings.md).

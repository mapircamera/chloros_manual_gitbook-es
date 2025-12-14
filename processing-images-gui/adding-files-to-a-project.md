# Añadir archivos a un proyecto

Una vez que haya creado o abierto un proyecto en Chloros, el siguiente paso es añadir sus imágenes multiespectrales para comenzar el procesamiento. El explorador de archivos<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> facilita la importación de imágenes y la gestión de su conjunto de datos.

## Acceso al explorador de archivos

1. Abra o cree un proyecto en Chloros
2. Haga clic en el icono **Explorador de archivos** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> en la barra lateral izquierda
3. El panel Explorador de archivos mostrará la lista de archivos de su proyecto

{% hint style=&quot;info&quot; %}
**Tipos de archivos compatibles**: Chloros admite archivos de imagen RAW+JPG y JPG de las cámaras MAPIR Survey3W y Survey3N. Solo se recomiendan los archivos RAW+JPG.
{% endhint %}

***

## Añadir imágenes a su proyecto

Hay dos formas principales de añadir imágenes a su proyecto:

### Método 1: Añadir archivos

Utilice esta opción para importar archivos de imagen individuales o una pequeña selección de archivos.

1. Haga clic en el botón **«Añadir archivos»** situado en la parte superior del panel del explorador de archivos.
2. Navegue hasta la carpeta que contiene sus imágenes.
3. Seleccione uno o varios archivos de imagen (mantenga pulsada la tecla **Ctrl** para seleccionar varios archivos).
4. Haga clic en **«Abrir»** para importar los archivos seleccionados.

### Método 2: Añadir carpeta

Utilice esta opción para importar todas las imágenes de una carpeta a la vez.

1. Haga clic en el botón **«Añadir carpeta»** situado en la parte superior del panel del explorador de archivos.
2. Navegue hasta la carpeta que contiene las imágenes de su sesión de captura y selecciónela.
3. Haga clic en **«Seleccionar carpeta»** para importar todas las imágenes compatibles de esa carpeta.

***

## Descripción de la tabla del explorador de archivos

Una vez importadas las imágenes, estas aparecen en una tabla con las siguientes columnas:

### Miniatura

* Pequeña vista previa de cada imagen.
* Haga clic en la miniatura para ver la imagen completa en el área de vista previa principal.

### Nombre de archivo

* Nombre de archivo original de la cámara.
* Mantiene la convención de nomenclatura de la cámara (por ejemplo, IMG\_0001.RAW).

### Marca de tiempo

* Fecha y hora en que se capturó la imagen.
* Extraída de los metadatos EXIF de la imagen.
* Se utiliza para la sincronización PPK y la detección de objetivos de calibración

### Modelo de cámara

* Configuración de la cámara y el filtro detectada automáticamente
* Ejemplos: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* Se utiliza para aplicar los perfiles de procesamiento correctos

### Columna Objetivo (casilla de verificación)

* Marque esta casilla para las imágenes que contienen objetivos de calibración
* Acelera considerablemente la detección de objetivos durante el procesamiento
* Consulte [Selección de imágenes objetivo](choosing-target-images.md) para obtener más detalles

***

## Gestión de archivos en su proyecto

### Eliminación de archivos

Para eliminar imágenes no deseadas de su proyecto:

1. Seleccione una o varias imágenes en la tabla del explorador de archivos
2. Haga clic en el botón **«Eliminar seleccionados»**
3. Confirme la eliminación (los archivos no se eliminan del disco, solo se eliminan del proyecto)

### Ordenar y filtrar

* **Ordenar por columna**: haga clic en cualquier encabezado de columna para ordenar las imágenes
* **Ordenar por marca de tiempo**: útil para organizar secuencias de captura cronológicas.
* **Filtro de modelo de cámara**: agrupa las imágenes por tipo de cámara si utilizas varias cámaras.

***

## Vista previa de imágenes

### Ver la imagen completa

Haz clic en cualquier miniatura de imagen en el explorador de archivos para mostrarla en el área de vista previa principal:

1. La imagen aparece en el panel de vista previa central.
2. Utiliza los controles de zoom para inspeccionar los detalles de la imagen.
3. Navegue entre las imágenes utilizando las teclas de flecha.

### Navegación rápida

* **Imagen anterior**: haga clic en la flecha izquierda o pulse la tecla ←.
* **Imagen siguiente**: haga clic en la flecha derecha o pulse la tecla →.
* **Acercar/alejar**: utilice la rueda del ratón o los botones de zoom.
* **Panorámica**: haga clic y arrastre sobre la imagen cuando esté ampliada.

***

## Gestión de archivos duplicados

Chloros detecta e ignora automáticamente los archivos duplicados:

* Se omiten los archivos con nombres idénticos.
* Evita el doble procesamiento accidental.
* Se muestra un mensaje de advertencia cuando se detectan duplicados.

{% hint style=&quot;warning&quot; %}
**Importante**: No renombre ni modifique los archivos de imagen originales antes de importarlos. Chloros se basa en los nombres de archivo y metadatos originales para realizar un procesamiento adecuado.
{% endhint %}

***

## Conjuntos de datos de cámaras mixtas

Si su proyecto contiene imágenes de varias cámaras MAPIR:

1. Chloros detecta automáticamente cada modelo de cámara
2. Cada tipo de cámara se procesa con su perfil de calibración adecuado
3. El explorador de archivos muestra el modelo de cámara en la columna Modelo de cámara
4. El procesamiento aplica la configuración correcta para cada tipo de cámara

**Ejemplo de escenario**: Survey3W RGN + Survey3N OCN configuración de doble cámara.

***

## Prácticas recomendadas

### Organizar antes de importar

* Mantenga las imágenes de calibración en la misma carpeta que las imágenes de la encuesta.
* Mantenga la estructura original de carpetas de su cámara/tarjeta SD.
* No mezcle conjuntos de datos de diferentes sesiones en un mismo proyecto.

### Nombres de archivos

* Conserve los nombres de archivo originales de la cámara (IMG\_0001.RAW, etc.).
* No renombre los archivos antes de importarlos.
* Los nombres originales contienen metadatos importantes.

### Imágenes de calibración

* Incluya siempre 1-2 imágenes de calibración por sesión.
* Capture los objetivos antes y después de la sesión de captura.
* Coloque los objetivos en las mismas condiciones de iluminación que el área de captura.
* Marque las imágenes de calibración con la casilla Objetivo para acelerar el procesamiento.

***

## Problemas comunes y soluciones

### Las imágenes no aparecen después de la importación

**Posibles causas:**

* Formato de archivo no compatible (solo RAW+JPG y JPG de cámaras MAPIR).
* Las imágenes proceden de cámaras que no son MAPIR (consulte [Cámaras compatibles](../supported-cameras.md)).
* Archivo dañado o transferencia incompleta desde la tarjeta SD.

**Solución**: Verifique la compatibilidad del formato de archivo y el modelo de cámara.

### No se detecta el modelo de cámara

**Posibles causas:**

* Metadatos EXIF modificados.
* Imágenes editadas en software externo.
* Transferencia de archivos incompleta.

**Solución**: Vuelva a importar los archivos originales sin modificar desde la cámara o la tarjeta SD.

### Faltan marcas de tiempo

**Posibles causas:**

* Reloj de la cámara mal ajustado
* Datos EXIF eliminados por software externo

**Solución**: Compruebe que la configuración de la hora de la cámara era correcta durante la captura.

***

## Siguiente paso

Una vez importados los archivos:

1. **Revise la lista de archivos**: asegúrese de que todas las imágenes se han cargado correctamente.
2. **Compruebe los modelos de cámara**: verifique que la detección de la cámara es correcta.
3. **Marque las imágenes de destino**: consulte [Selección de imágenes de destino](choosing-target-images.md).
4. **Ajuste la configuración**: configure las opciones de procesamiento en [Configuración del proyecto](adjusting-project-settings.md).
5. **Inicie el procesamiento**: consulte [Inicio del procesamiento](starting-the-processing.md).

Para obtener información detallada sobre la configuración del proyecto, consulte [Ajuste de la configuración del proyecto](adjusting-project-settings.md).

# Marcadores del mapa

La pestaña Mapa muestra sus imágenes en un mapa 2D interactivo basado en sus coordenadas GPS. Esto proporciona una visión general geográfica de su sesión de captura y le ayuda a visualizar la cobertura espacial. También es útil cuando se importan las imágenes por primera vez para eliminar rápidamente aquellas que no necesita procesar.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Acceso a la pestaña Mapa

1. Abra o cree un proyecto en Chloros.
2. Importe imágenes que contengan metadatos GPS.
3. Haga clic en la pestaña **Mapa** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> en la barra lateral izquierda.
4. El mapa mostrará marcadores en la ubicación GPS de cada imagen.

{% hint style="info" %}
**Se requiere GPS**: Solo aparecerán en el mapa las imágenes con coordenadas GPS incrustadas en sus metadatos EXIF. Asegúrese de que su cámara tiene el GPS activado durante la captura.
{% endhint %}

***

## Ajustar imágenes desde la pestaña Mapa

La pestaña **Mapa**<img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> tiene las mismas funciones de añadir  <img src="../.gitbook/assets/image.png" alt="" data-size="line">   <img src="../.gitbook/assets/image (1).png" alt="" data-size="line">  y eliminar  <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">  que la pestaña [**Explorador de archivos**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> . También muestra la misma lista de archivos de proyecto, pero con diferentes encabezados de columna:

### Nombre de archivo

* Nombre de archivo original de la cámara.
* Mantiene la convención de nomenclatura de la cámara (por ejemplo, IMG\_0001.RAW).

### Latitud

* La latitud de la imagen.

### Longitud

* La longitud de la imagen.

### Altitud

* Altitud de la imagen

{% hint style="info" %}
Al hacer clic en los encabezados de las columnas de la tabla, también se ordenan los datos de las filas.
{% endhint %}

***

## Marcadores de imágenes

Cada imagen con datos GPS se representa con un marcador en el mapa:

### Visualización de marcadores

* Los marcadores indican las coordenadas GPS exactas donde se capturó cada imagen.
* Los marcadores agrupados pueden agruparse al alejar el zoom.
* Acerque el zoom para ver las ubicaciones de las imágenes individuales.

{% hint style="success" %}
SUPERZOOM: cuando se alcanza el nivel máximo de zoom del proveedor de mosaicos del mapa, el mosaico se amplía al acercar aún más el zoom, lo que permite ver los marcadores que están muy juntos.
{% endhint %}

### Vista previa al pasar el cursor

* **Pase el cursor** sobre cualquier marcador para ver una vista previa en miniatura de esa imagen.
* Esto permite una rápida identificación visual sin salir de la vista del mapa.
* Útil para localizar imágenes específicas dentro de una sesión de captura grande.

***

## Proveedores de mosaicos de mapas

{% hint style="success" %}
**Selección automática**: Chloros elige automáticamente el servicio de mosaicos que proporciona el mejor nivel de zoom para su ubicación actual en el mapa. Si lo desea, puede cambiar manualmente entre proveedores.
{% endhint %}

La pestaña Mapa admite dos proveedores de mosaicos para las imágenes de fondo del mapa:

### Google Maps

* Imágenes estándar de satélite y mapas de Google.
* Ideal para una cobertura general en todo el mundo.

### ESRI

* Imágenes de satélite y aéreas de ESRI ArcGIS.
* A menudo proporciona imágenes de mayor resolución en determinadas regiones.

***

## Tipos de mosaicos de mapas

Puede elegir el tipo de capa del mapa (de izquierda a derecha):

 <img src="../.gitbook/assets/image (23).png" alt="" data-size="original">### Terreno

Muestra perfiles de elevación y mosaicos de mapa con detalles (carreteras, etc.).

### Mapa

Muestra mosaicos de mapa estándar (menor ancho de banda) con detalles (carreteras, etc.).

### Satélite

Muestra mosaicos de mapa satelital detallados (mayor ancho de banda).

### Híbrido

Muestra mosaicos de mapa satelital con detalles añadidos (carreteras, etc.).

***

## Navegación por el mapa

### Controles de zoom

* **Acercar/alejar**: utilice la rueda del ratón o los botones de zoom.
* **Pantalla completa**: muestra el mapa en pantalla completa.

### Controles de desplazamiento

* **Desplazamiento**: haga clic y arrastre para desplazarse por el mapa.***

## Casos de uso

### Visualización de la trayectoria de vuelo

* Ver el área de cobertura de las sesiones de captura con drones.
* Identificar lagunas en la cobertura de las imágenes.
* Verificar la ejecución de la trayectoria de vuelo.

### Revisión del estudio del terreno

* Ver la distribución espacial de las capturas terrestres.
* Localizar las imágenes de calibración en relación con el área de estudio.
* Planificar ubicaciones de captura adicionales.

### Control de calidad

* Identifique rápidamente las imágenes capturadas en ubicaciones inesperadas.
* Verifique la precisión del GPS en todo el conjunto de datos.
* Cruce las ubicaciones de las imágenes con las notas de campo.

***

## Solución de problemas

### No aparecen marcadores

**Posibles causas:**

* Las imágenes no contienen metadatos GPS.
* El GPS estaba desactivado en la cámara durante la captura.
* Los datos EXIF fueron eliminados por un software externo.

**Solución**: Verifique que el GPS esté habilitado en su cámara y vuelva a importar los archivos originales.

### Marcadores en una ubicación incorrecta

**Posibles causas:**

* El GPS de la cámara tenía una mala fijación satelital.
* Desviación del GPS durante la captura.

**Solución**: Por lo general, se trata de un problema relacionado con el momento de la captura; considere el uso de GPS PPK/RTK para aplicaciones de precisión.

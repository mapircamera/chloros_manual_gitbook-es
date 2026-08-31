# Marcadores del mapa

La pestaña «Mapa» traza tus imágenes en un mapa 2D interactivo a partir de sus coordenadas GPS. Te ofrece una visión general geográfica de una sesión de captura y es la forma más rápida, justo después de la importación, de descartar las imágenes que no deseas procesar.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Cómo acceder a la pestaña «Mapa»

1. Abre o crea un proyecto en Chloros
2. Importa imágenes que contengan metadatos GPS
3. Haz clic en la pestaña **Mapa** <img src="../.gitbook/assets/image (3) (1).png" alt="" data-size="line"> de la barra lateral izquierda
4. El mapa muestra un marcador en la ubicación GPS de cada imagen

{% hint style="info" %}
**Se requiere GPS**: solo aparecen en el mapa las imágenes con coordenadas GPS en sus metadatos EXIF. Una imagen sin coordenadas sigue estando en el proyecto y se procesa con normalidad; simplemente no tiene marcador.
{% endhint %}

***

## Ajuste de imágenes desde la pestaña «Mapa»

La pestaña **Mapa**<img src="../.gitbook/assets/image (3) (1).png" alt="" data-size="line"> cuenta con los mismos botones para añadir <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> y eliminar <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line"> archivos que la pestaña [**Explorador de archivos**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">. Muestra la misma lista de archivos del proyecto, con columnas geográficas:

| Columna        | Contenido                                                           |
| ------------- | ------------------------------------------------------------------ |
| **Nombre**      | El nombre del archivo tal y como salió de la cámara                             |
| **Latitud**  | Grados decimales, seis decimales                                |
| **Longitud** | Grados decimales, seis decimales                                |
| **Altitud**  | Metros, un decimal — `-` cuando la imagen no incluye altitud |

{% hint style="info" %}
Haz clic en el encabezado de cualquier columna para ordenar por ella; vuelve a hacer clic para invertir el orden.
{% endhint %}

{% hint style="warning" %}
**La altitud es la altura sobre el nivel del mar, no la altura sobre el suelo.** El valor procede de la etiqueta EXIF `GPSAltitude` de la imagen, que se refiere al nivel medio del mar. No se trata de la altura de vuelo sobre el terreno, y Chloros no calculará la distancia de muestreo al suelo a partir de ella: sobre un campo situado a 300 m sobre el nivel del mar, un dron a 100 m AGL registrará aquí aproximadamente 400 m. Utiliza esta columna para detectar valores atípicos y confirmar que la altitud de vuelo es constante, no como una medida AGL.
{% endhint %}

***

## Marcadores de imagen

Cada imagen con datos GPS recibe un marcador en sus coordenadas.

### Visualización de marcadores

* Los marcadores se sitúan en las coordenadas exactas registradas para cada captura
* Los marcadores muy próximos entre sí pueden superponerse visualmente al alejar el zoom; acércate para separarlos
* Los marcadores seleccionados y resaltados se muestran por encima del resto

### Vista previa al pasar el cursor

* **Pasa el cursor** por cualquier marcador para que aparezca una miniatura de esa imagen con su nombre de archivo
* **Haz clic**en un marcador para seleccionar la imagen y**fijar** la ventana emergente; permanecerá abierta hasta que hagas clic en otro lugar. Mientras la ventana esté fijada, al pasar el cursor por encima de otros marcadores no desaparecerá
* Esta es la forma más rápida de encontrar un fotograma concreto en una sesión extensa sin salir del mapa

<figure><img src="../.gitbook/assets/image (36).png" alt=""><figcaption><p>La pestaña «Mapa» muestra todas las imágenes geolocalizadas del proyecto</p></figcaption></figure>### Superzoom

{% hint style="success" %}
**SUPERZOOM**: cuando alcanzas el zoom máximo para el que el proveedor de mosaicos dispone de imágenes, al seguir ampliando se agrandan los mosaicos en lugar de detenerse, lo que te permite distinguir marcadores que se superponen casi por completo.
{% endhint %}

* El superzoom solo se activa cuando te encuentras **en** el zoom máximo del proveedor para esa ubicación y los mosaicos han terminado de cargarse. Por debajo de ese nivel, el zoom funciona con normalidad
* El rango es de **1× a 32×** por encima del máximo del propio proveedor
* Un indicador en la esquina muestra el superzoom actual como porcentaje, y un botón **×** situado junto a él te devuelve al zoom normal con un solo clic
* Al alejar el zoom, la acción siempre se transmite al mapa propiamente dicho, por lo que nunca te quedarás atascado en el superzoom
* Al ampliar y desplazar la vista mientras se está en superzoom, el desplazamiento resultante se transfiere al mapa, de modo que la zona descentrada a la que te has desplazado sigue solicitando mosaicos en lugar de quedar en blanco
* Los marcadores se dibujan como elementos vectoriales en lugar de rasterizados, por lo que se mantienen nítidos en cualquier nivel de superzoom

***

## Proveedores de mosaicos de mapa

{% hint style="success" %}
**Selección automática**: Chloros elige el servicio de mosaicos que ofrece el mejor nivel de zoom para la ubicación de tus imágenes. Puedes cambiar manualmente en cualquier momento.
{% endhint %}

| Proveedor        | Notas                                                                                                                                                             |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Google Maps** | Amplia cobertura mundial; admite los cuatro tipos de mosaicos                                                                                                            |
| **Esri ArcGIS**| Imágenes aéreas a menudo de mayor resolución en determinadas regiones. El tipo de mosaico**Terrain** no está disponible para Esri y su botón aparece desactivado mientras Esri está seleccionado |***

## Tipos de mosaicos de mapa

Elige el tipo de capa de mapa con los botones (de izquierda a derecha):

![](&lt;../.gitbook/assets/image (14).png&gt;)

| Tipo                 | Muestra                                                                |
| -------------------- | -------------------------------------------------------------------- |
| **Relieve**          | Sombreado de elevación con detalles del mapa (carreteras, etiquetas). Solo Google       |
| **Mapa**              | Mosaicos de mapa de calles estándar: la opción que requiere menos ancho de banda              |
| **Satélite**        | Imágenes detalladas de satélite, sin etiquetas — la opción que requiere mayor ancho de banda |
| **Híbrido** (predeterminado) | Imágenes de satélite con carreteras y etiquetas superpuestas                |

La pestaña «Mapa» se abre en modo **Híbrido**. La opción elegida se aplica al cambiar de proveedor, siempre que este lo admita.***

## Navegación por el mapa

* **Zoom**: rueda de desplazamiento del ratón o botones de zoom en el mapa
* **Desplazamiento**: haz clic y arrastra
* **Pantalla completa**: el control de pantalla completa amplía el mapa a toda la ventana***

## Casos de uso

### Revisión de la trayectoria de vuelo

* Ver de un vistazo el área de cobertura de una sesión con dron
* Detectar huecos donde se haya omitido una pasada
* Confirmar que el vuelo siguió el patrón planificado

### Revisión de levantamientos terrestres

* Ver cómo se distribuyen las capturas terrestres
* Localizar los marcos de referencia de calibración en relación con el área de levantamiento
* Decidir dónde se necesitan capturas adicionales

### Control de calidad

* Encontrar imágenes capturadas en lugares inesperados y eliminarlas antes del procesamiento
* Ordenar por altitud para detectar un fotograma capturado a una altura incorrecta, o uno en el que la posición GPS fuera deficiente
* Comparar las ubicaciones de las imágenes con las notas de campo

***

## Solución de problemas

### No aparecen marcadores

**Posibles causas**

* Las imágenes no contienen metadatos GPS
* El GPS estaba desactivado en la cámara durante la captura
* Los datos EXIF fueron eliminados por otro software antes de la importación

**Qué hacer**: comprueba que el GPS esté activado en la cámara y vuelve a importar los archivos originales. Puedes comprobar si un archivo concreto tiene coordenadas buscándolo en la tabla de archivos de la pestaña «Mapa»: una imagen sin coordenadas no aparecerá en ninguna fila de esa tabla.

### Los marcadores están en el lugar equivocado

**Posibles causas**: una mala recepción de la señal satelital en el momento de la captura o una deriva del GPS durante la sesión.**Qué hacer**: se trata de un problema relacionado con el momento de la captura, no es algo que Chloros pueda corregir a posteriori. Para un trabajo de precisión, utiliza un flujo de trabajo con GPS PPK/RTK; consulta la configuración**Aplicar correcciones PPK** en [Configuración del proyecto](../project-settings/project-settings.md).

### El mapa está en blanco o los mosaicos dejan de cargarse

Los proveedores de mosaicos son servicios en línea. Si dejan de cargarse los mosaicos, comprueba la conexión de red del dispositivo y, a continuación, prueba a cambiar de proveedor. Si tenías el zoom muy ampliado, pulsa el botón de reinicio **×** para volver a un nivel de zoom normal y deja que el mapa vuelva a solicitar los mosaicos.***

## Páginas relacionadas

* [**Cuadrícula de imágenes**](image-grid.md): el mismo conjunto de imágenes que las miniaturas
* [**Abrir una imagen a pantalla completa**](opening-an-image-full-screen.md): examinar una imagen en detalle
* [**Añadir archivos a un proyecto**](../processing-images-gui/adding-files-to-a-project.md): los botones para añadir o eliminar archivos que comparte esta pestaña

# Abrir una imagen a pantalla completa

El visor de imágenes Chloros ofrece una interfaz específica a pantalla completa para visualizar, analizar y manipular sus imágenes multiespectrales. Tanto si se trata de imágenes originales como de resultados procesados, el visor de imágenes ofrece potentes herramientas de inspección y análisis.

## Acceso al visor de imágenes

### Desde el explorador de archivos

La forma más habitual de abrir una imagen en el visor de imágenes:

1. Asegúrese de que se encuentra en la pestaña **Explorador de archivos** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Haga clic en cualquier **miniatura de imagen** de la cuadrícula de imágenes
3. La imagen se abre en el **área de vista previa principal** (centro de la pantalla)
4. La imagen ya está cargada y lista para su visualización a pantalla completa

### Abrir la pestaña del visor de imágenes

Una vez que la imagen se ha cargado en el área de vista previa:

1. Haga clic en el icono **Visor de imágenes** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> en la barra lateral izquierda
2. Se abre la pestaña del visor de imágenes, mostrando la imagen seleccionada a pantalla completa
3. Las herramientas avanzadas de visualización y análisis estarán disponibles en la barra lateral izquierda

***

## Descripción general de la interfaz del visor de imágenes

### Área de visualización principal

La mayor parte de la pantalla muestra tu imagen:

* **Resolución completa**: Imágenes mostradas a resolución nativa
* **Zoomable**: utiliza los controles o la rueda del ratón para hacer zoom
* **Pannable**: haz clic y arrastra para desplazarte cuando estés haciendo zoom
* **Relación de aspecto mantenida**: las imágenes se escalan proporcionalmente***

## Opciones de visualización

### Navegación básica por las imágenes

#### Navegar por las imágenes

Navega por tu conjunto de imágenes utilizando atajos de teclado o botones:

* **Imagen siguiente**: haz clic en el botón → o pulsa la tecla**→** (flecha derecha)
* **Imagen anterior**: haz clic en el botón ← o pulsa la tecla**←** (flecha izquierda)
* **Ir a una imagen específica**: vuelve al explorador de archivos y haz clic en la miniatura deseada

#### Controles de zoom

Ajusta el aumento para examinar los detalles de la imagen:

**Acercar:*** Haz clic en el botón **+** (más)
* Pulsa la tecla **+**o**=*** Gira la rueda del ratón **hacia arriba**

**Alejar:*** Haz clic en el botón **−** (menos)
* Pulsa la tecla **−** (menos)
* Gira la rueda del ratón **hacia abajo**

#### Desplazamiento al ampliar

Cuando se amplía más allá del tamaño de la pantalla:

1. Mueve el cursor del ratón sobre la imagen
2. Haz clic y **mantén pulsado el botón izquierdo del ratón**

3.**Arrastra** para mover la imagen
4. Suelta para detener el desplazamiento

**Alternativa**: Utiliza las teclas de flecha para desplazarte en pequeños incrementos***

## Inspección del valor de los píxeles

### Visualización de los valores de píxeles en el cursor

A medida que mueves el cursor del ratón sobre la imagen, los valores de los píxeles se muestran en tiempo real:**Ubicación de la visualización de valores:*** **Número flotante y línea roja en la leyenda del gradiente LUT del índice del lado derecho*** **Cuando se amplía aún más, valor flotante cerca del cursor y del píxel resaltado*** Muestra los valores del píxel **bajo el cursor o resaltado*** Se actualiza a medida que se mueve el ratón

***

## Tipos de imagen que se pueden visualizar

### JPG

**Imágenes JPG de la cámara:**

* Muestra los datos JPG tal y como se previsualizan
* Muestra los valores originales, sin corregir
* Útil para comprobar la calidad de la imagen antes del procesamiento

### RAW (Original)

### RAW (Reflectancia)

**Tras el procesamiento:**

* Viñete corregida
* Reflectancia calibrada
* Multibanda TIFF (Red, Green, NIR, etc.)
* Datos científicos listos para su análisis

### RAW (Índice)

**NDVI, NDRE, GNDVI, etc. (archivos \_NDVI.tif):**

* Imágenes en escala de grises de una sola banda
* Los valores de los píxeles representan los resultados del cálculo del índice
* Rango típico de -1 a +1 para índices normalizados
* Se pueden aplicar tablas de conversión de color (LUT) para la visualización

***

## Aplicación de índices y LUT

Aplicar índices multiespectrales y tablas de conversión de color:

1. Localice **Index/LUT Sandbox**en**Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> barra lateral
2. Seleccione el índice de vegetación (NDVI, NDRE, etc.)
3. Seleccione una fórmula multiespectral o cree una personalizada (solo Chloros+)
4. Aplique un degradado de LUT de color para la visualización
5. Ajuste los rangos de valores y los umbrales

Consulte [Index/LUT Sandbox](index-lut-sandbox.md) para obtener instrucciones detalladas.

***

## Atajos de teclado

### Navegación

* **→** (Flecha derecha): Imagen siguiente
* **←** (Flecha izquierda): Imagen anterior
* **Inicio**: Primera imagen de la lista
* **Fin**: Última imagen de la lista

### Zoom

* **+**o**=**: Acercar
* **−**: Alejar
* **Rueda del ratón**: Acercar/alejar***

### Verificación de los cálculos de los índices

Comprueba que los índices se hayan calculado correctamente:

1. Abra NDVI u otra imagen de índice
2. Compruebe las áreas de vegetación:
   * **NDVI**: Debería mostrar un valor de 0,4-0,9 para plantas sanas
   * **NDRE**: Valores más altos para un crecimiento vigoroso
   * **GNDVI**: Similar a NDVI, pero sensible a la clorofila
3. Compruebe las zonas sin vegetación:
   * **Suelo**: Cerca de 0 o ligeramente negativo
   * **Agua**: Valores negativos (de -0,5 a 0)***

## Solución de problemas de visualización

### La imagen no se abre

**Posibles causas:**

* Archivo dañado durante el procesamiento
* Formato de archivo no compatible
* Memoria insuficiente para imágenes de gran tamaño

**Soluciones:**

1. Intente abrirla en un visor externo para verificar la integridad del archivo
2. Compruebe que el formato del archivo coincide con el tipo esperado
3. Cierre otras aplicaciones para liberar memoria
4. Pruebe con una imagen más pequeña o diferente

### Visualización de la imagen en blanco o negro

**Posibles causas:**

* Rango de valores fuera de la capacidad de visualización
* Imagen de 32 bits en formato flotante con valores inusuales
* Error en el cálculo del índice

**Soluciones:**

1. Comprueba los valores de los píxeles: si todos son muy bajos o muy altos, ajusta el rango de visualización
2. Intenta abrirla en QGIS o similar con ajuste automático de rango
3. Comprueba el registro de depuración del procesamiento en busca de errores

### Los valores de los píxeles parecen incorrectos

**Posibles causas:**

* Se está viendo la imagen incorrecta (original frente a procesada)
* La calibración no se ha aplicado correctamente
* Los datos del sensor de luz no se han incluido en la entrada
* El modo porcentual se ha activado incorrectamente

**Soluciones:**

1. Comprueba que estás viendo el resultado procesado (comprueba el sufijo del nombre del archivo)
2. Comprueba el estado del botón del modo porcentual
3. Compara con imágenes que se sabe que son correctas del mismo conjunto de datos

***

## Próximos pasos

Ahora que puede ver las imágenes a pantalla completa:

* [**Capas de imagen**](image-layers.md) - Más información sobre la visualización multibanda
* [**Área de pruebas de índices/LUT**](index-lut-sandbox.md) - Aplique índices personalizados y mapeo de colores
* [**Fórmulas de índices multiespectrales**](../project-settings/multispectral-index-formulas.md) - Conoce los índices disponibles

Para el flujo de trabajo de procesamiento, consulta:

* [**Procesamiento de imágenes (GUI)**](../processing-images-gui/adding-files-to-a-project.md) - Guía completa de procesamiento

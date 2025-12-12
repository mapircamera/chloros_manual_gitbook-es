# Abrir una imagen a pantalla completa

El visor de imágenes Chloros proporciona una interfaz dedicada a pantalla completa para ver, analizar y manipular sus imágenes multiespectrales. Tanto si se trata de imágenes originales como de resultados procesados, el visor de imágenes ofrece potentes herramientas para la inspección y el análisis.

## Acceso al visor de imágenes

### Desde el explorador de archivos

La forma más habitual de abrir una imagen en el visor de imágenes:

1. Asegúrese de que se encuentra en la pestaña **Explorador de archivos**. <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Haga clic en cualquier **miniatura de imagen** de la cuadrícula de imágenes
3. La imagen se abre en el **área de vista previa principal** (centro de la pantalla)
4. La imagen ya está cargada y lista para su visualización a pantalla completa

### Abrir la pestaña del visor de imágenes

Una vez que la imagen se ha cargado en el área de vista previa:

1. Haga clic en el icono **Visor de imágenes** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> en la barra lateral izquierda.
2. Se abrirá la pestaña Visor de imágenes, que mostrará la imagen seleccionada a pantalla completa.
3. Las herramientas avanzadas de visualización y análisis estarán disponibles en la barra lateral izquierda.

***

## Descripción general de la interfaz del visor de imágenes

### Área de visualización principal

La mayor parte de la pantalla muestra la imagen:

* **Resolución completa**: las imágenes se muestran con su resolución nativa.
* **Zoomable**: utilice los controles o la rueda del ratón para ampliar
* **Desplazable**: haga clic y arrastre para desplazarse cuando esté ampliado
* **Relación de aspecto mantenida**: las imágenes se escalan proporcionalmente

***

## Opciones de visualización

### Navegación básica por las imágenes

#### Navegar por las imágenes

Navegue por su conjunto de imágenes utilizando atajos de teclado o botones:

* **Imagen siguiente**: haga clic en el botón → o pulse la tecla **→** (flecha derecha)
* **Imagen anterior**: haga clic en el botón ← o pulse la tecla **←** (flecha izquierda)
* **Ir a una imagen específica**: vuelva al explorador de archivos y haga clic en la miniatura deseada

#### Controles de zoom

Ajuste el aumento para inspeccionar los detalles de la imagen:

**Acercar:**

* Haga clic en el botón **+** (más).
* Pulse la tecla **+** o **=**.
* Desplácese con la rueda del ratón **hacia arriba**.

**Alejar:**

* Haga clic en el botón **−** (menos).
* Pulse la tecla **−** (menos).
* Desplácese con la rueda del ratón **hacia abajo**.

**Ajustar a la pantalla:**

* Haga clic en el botón **↔** (Ajustar).
* Pulse la tecla **0** (cero).
* Haga doble clic en la imagen.

#### Desplazamiento cuando se amplía

Cuando se amplía más allá del tamaño de la pantalla:

1. Mueva el cursor del ratón sobre la imagen.
2. Haga clic y **mantenga pulsado el botón izquierdo del ratón**.
3. **Arrastre** para mover la imagen.
4. Suelte para detener el desplazamiento.

**Alternativa**: utilice las teclas de flecha para desplazarse en pequeños incrementos.

***

## Inspección del valor de los píxeles

### Visualización de los valores de los píxeles en el cursor

A medida que mueve el cursor del ratón sobre la imagen, los valores de los píxeles se muestran en tiempo real:

**Ubicación de la visualización del valor:**

* **Número flotante y línea roja en la leyenda del gradiente LUT del índice del lado derecho**
* **Al ampliar aún más, valor flotante cerca del cursor y píxel resaltado**
* Muestra los valores del píxel **debajo del cursor o resaltado**
* Se actualiza al mover el ratón

***

## Tipos de imágenes que se pueden ver

### Imágenes originales (preprocesamiento)

**Imágenes RAW + JPG de la cámara:**

* Muestra los datos RAW tal y como se previsualizan
* Muestra los valores originales sin corregir
* Útil para comprobar la calidad de la imagen antes del procesamiento

### Imágenes de reflectancia calibradas

**Después del procesamiento:**

* Viñeta corregida.
* Reflectancia calibrada.
* Multibanda TIFF (Red, Green, NIR, etc.).
* Datos científicos listos para su análisis.

### Imágenes de índice

**NDVI, NDRE, GNDVI, etc. (archivos \_NDVI.tif):**

* Imágenes en escala de grises de banda única
* Los valores de píxeles representan los resultados del cálculo del índice
* Rango típico de -1 a +1 para índices normalizados
* Se pueden aplicar LUT de color para la visualización

***

## Aplicación de índices y LUT

Aplicar índices multiespectrales y tablas de consulta de colores:

1. Localizar **Index/LUT Sandbox** en la <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> barra lateral
2. Seleccione el índice de vegetación (NDVI, NDRE, etc.)
3. Seleccione la fórmula multiespectral o cree una personalizada (solo Chloros+)
4. Aplique un gradiente de tabla de consulta de colores (LUT) para la visualización
5. Ajuste los rangos de valores y los umbrales

Consulte [Índice/LUT Sandbox](index-lut-sandbox.md) para obtener instrucciones detalladas.

***

## Atajos de teclado

### Navegación

* **→** (flecha derecha): imagen siguiente
* **←** (flecha izquierda): imagen anterior
* **Inicio**: primera imagen de la lista
* **Fin**: Última imagen de la lista

### Zoom

* **+** o **=**: Acercar
* **−**: Alejar
* **0** (cero): Ajustar a la pantalla
* **Rueda del ratón**: Acercar/alejar

### Controles de visualización

* **P**: Alternar modo de porcentaje de píxeles
* **L**: Alternar panel de capas
* **Esc**: Cerrar pantalla completa o volver al explorador de archivos

### Otros

* **Ctrl+S**: Guardar imagen actual
* **F**: Modo de pantalla completa (si está disponible)

***

### Verificación de los cálculos del índice

Compruebe que los índices se han calculado correctamente:

1. Abra NDVI u otra imagen del índice.
2. Compruebe las áreas de vegetación:
   * **NDVI**: Debe mostrar entre 0,4 y 0,9 para plantas sanas.
   * **NDRE**: valores más altos para un crecimiento vigoroso
   * **GNDVI**: similar a NDVI, pero sensible a la clorofila
3. Compruebe la ausencia de vegetación:
   * **Suelo**: Cerca de 0 o ligeramente negativo.
   * **Agua**: Valores negativos (de -0,5 a 0).

***

## Solución de problemas de visualización

### La imagen no se abre

**Posibles causas:**

* Archivo dañado durante el procesamiento.
* Formato de archivo no compatible.
* Memoria insuficiente para imágenes grandes.

**Soluciones:**

1. Intente abrirla en un visor externo para verificar la integridad del archivo.
2. Compruebe que el formato del archivo coincide con el tipo esperado.
3. Cierre otras aplicaciones para liberar memoria.
4. Pruebe con una imagen más pequeña o diferente.

### Imagen en blanco y negro

**Posibles causas:**

* Rango de valores fuera de la capacidad de visualización.
* Imagen flotante de 32 bits con valores inusuales.
* Error en el cálculo del índice.

**Soluciones:**

1. Compruebe los valores de los píxeles: si todos son muy bajos o muy altos, ajuste el rango de visualización.
2. Intente abrirlo en QGIS o similar con ajuste automático del rango.
3. Compruebe el registro de depuración del procesamiento en busca de errores.

### Los valores de los píxeles parecen incorrectos

**Posibles causas:**

* Visualización de una imagen incorrecta (original frente a procesada).
* La calibración no se aplicó correctamente.
* Los datos del sensor de luz no se incluyeron en la entrada.
* El modo porcentual se activó incorrectamente.

**Soluciones:**

1. Verifique que está viendo el resultado procesado (compruebe la extensión del nombre del archivo).
2. Compruebe el estado del botón del modo porcentual.
3. Compare con imágenes correctas conocidas del mismo conjunto de datos.

***

## Próximos pasos

Ahora que puede ver las imágenes a pantalla completa:

* [**Capas de imagen**](image-layers.md): aprenda sobre la visualización multibanda.
* [**Sandbox de índices/LUT**](index-lut-sandbox.md): aplique índices personalizados y mapeo de colores.
* [**Fórmulas de índice multiespectral**](../project-settings/multispectral-index-formulas.md): comprenda los índices disponibles.

Para el flujo de trabajo de procesamiento, consulte:

* [**Procesamiento de imágenes (GUI)**](../processing-images-gui/adding-files-to-a-project.md): guía completa de procesamiento.

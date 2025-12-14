# Capas de imagen

El menú desplegable Capas de imagen del visor de imágenes Chloros le permite cambiar rápidamente entre diferentes versiones de la misma imagen, desde las capturas originales hasta los resultados de reflectancia procesados y las imágenes de índice calculadas.

## ¿Qué son las capas de imagen?

En Chloros, las **capas** se refieren a los diferentes resultados de imagen disponibles para una sola imagen de origen. Cuando procesa imágenes, Chloros crea varias versiones:

* **Imágenes originales** (archivos JPG y RAW de su cámara)
* Resultados **calibrados por reflectancia** (si se habilitó la calibración de reflectancia)
* **Imágenes objetivo** (si la imagen contiene objetivos de calibración)
* **Imágenes de índice** (NDVI, NDRE, GNDVI, etc. si se configuraron índices)

El **menú desplegable Selector de capas** situado en la parte superior derecha del visor de imágenes le permite cambiar instantáneamente entre estas versiones sin salir del visor.

***

## Tipos de capas disponibles

### JPG

* La imagen de vista previa JPG original de su cámara
* Siempre disponible para todas las imágenes
* Sin procesar, tal y como la capturó la cámara
* La más rápida de cargar y mostrar

**Cuándo verla:**

* Vista previa rápida de la captura original.
* Comprobación de la composición y el encuadre de la imagen.
* Verificación de la calidad de la captura antes del procesamiento.

### RAW (original)

* Los datos originales del sensor RAW de su cámara.
* Desbayered sin aplicar ningún posprocesamiento.
* Mayor profundidad de bits que JPG (normalmente datos del sensor de 12 o 14 bits).

**Cuándo ver:**

* Inspección de la calidad de los datos originales del sensor.
* Comprobación de problemas del sensor o artefactos.
* Comparación de los resultados antes y después del procesamiento.

### RAW (objetivo)

* Solo aparece para imágenes identificadas como contenedoras de objetivos de calibración.
* Muestra la imagen RAW original con el objetivo detectado.
* Se utiliza para verificar que la detección del objetivo se ha realizado correctamente.

**Cuándo ver:**

* Confirmar que los objetivos de calibración se han detectado correctamente.
* Comprobar la calidad de la imagen del objetivo.
* Solucionar problemas de calibración.

{% hint style=&quot;info&quot; %}
**Capa de objetivo**: esta capa solo aparece en el menú desplegable de las imágenes que contienen objetivos de calibración. Las imágenes de captura normales no tendrán esta opción.
{% endhint %}

### RAW (reflectancia)

* La imagen de salida de reflectancia calibrada.
* Corrección de viñeta (si está habilitada en el procesamiento).
* Reflectancia calibrada utilizando datos de objetivo (si está habilitado)
* Multibanda TIFF con todos los canales de la cámara
* Los valores de píxeles representan el porcentaje de reflectancia (cuando se utiliza el modo porcentaje)
* Listo para manipular con el [Índice/LUT Sandbox](index-lut-sandbox.md)

**Cuándo ver:**

* Inspeccionar los resultados calibrados
* Verificar la calidad de la calibración
* Comprobar los valores de píxeles para garantizar la precisión científica
* Comparar con el original para ver los efectos de la calibración

{% hint style=&quot;success&quot; %}
**Recomendado**: Utilice la capa RAW (reflectancia) al comprobar los valores de píxeles para mediciones y análisis científicos.
{% endhint %}

### RAW (NDVI Índice)... y similares

* Imagen del índice de vegetación calculado (NDVI en este ejemplo)
* El nombre del índice cambia en función del índice que se haya configurado durante el procesamiento
* Ejemplos: RAW (Índice NDVI), RAW (Índice NDRE), RAW (Índice GNDVI), etc.
* Imagen en escala de grises de banda única que muestra los resultados del cálculo del índice
* Aparece una capa por cada índice configurado en la configuración del proyecto

**Posibles nombres de índices:**

* RAW (índice NDVI)
* RAW (índice NDRE)
* RAW (índice GNDVI)
* RAW (Índice OSAVI)
* RAW (Índice EVI)
* RAW (Índice SAVI)
* Y muchos más... (véase [Fórmulas de índices multiespectrales](../project-settings/multispectral-index-formulas.md))

**Cuándo ver:**

* Examinar los resultados del cálculo del índice.
* Comprobar los rangos de valores del índice.
* Identificar áreas de interés.
* Verificar las imágenes del índice antes de utilizarlas en SIG o análisis.

***

## Uso del selector de capas

### Abrir el menú desplegable

1. Abra una imagen en modo de pantalla completa (haga clic en cualquier miniatura del visor de imágenes).
2. Localice el **menú desplegable de capas** en la esquina superior derecha del visor.
3. El menú desplegable muestra la capa seleccionada actualmente (por ejemplo, «JPG»).
4. Haga clic en el menú desplegable para ver todas las capas disponibles.

### Cambiar de capa

1. Haga clic en el menú desplegable de capas para abrir la lista.
2. Se muestran todas las capas disponibles para la imagen actual.
3. Haga clic en el nombre de cualquier capa para cambiar a esa versión.
4. La imagen se actualiza inmediatamente para mostrar la capa seleccionada.

**Cambio rápido:**

* El menú desplegable recuerda su última selección.
* Al navegar a la siguiente imagen, Chloros intenta mostrar el mismo tipo de capa.
* Si esa capa no existe en la siguiente imagen, se establece de forma predeterminada en JPG.

### Disponibilidad de capas

No todas las capas están disponibles para todas las imágenes:

**Siempre disponibles:**

* ✅ JPG (todas las imágenes tienen una vista previa en JPG).

**Disponibles de forma condicional:**

* ⚠️ RAW (original): solo si la imagen se ha capturado en modo RAW o RAW+JPG.
* ⚠️ RAW (objetivo): solo si la imagen contiene objetivos de calibración detectados.
* ⚠️ RAW (Reflectancia): solo después de procesar con la calibración de reflectancia habilitada.
* ⚠️ RAW (\[Índice] Índice): solo después de procesar con índices configurados.

***

## Persistencia de capas

### Navegación entre imágenes

Cuando navegas a una imagen diferente (utilizando las teclas de flecha o haciendo clic en las miniaturas):

**Se conserva la preferencia de capa:**

* Si se visualiza «RAW (reflectancia)», la siguiente imagen muestra «RAW (reflectancia)» (si está disponible).
* Si se visualiza «RAW (NDVI índice)», la siguiente imagen muestra «RAW (NDVI índice)». (si está disponible).
* Si no existe la misma capa, se establece de forma predeterminada JPG.

**Ejemplo de flujo de trabajo:**

1. Abra la imagen 1, cambie a RAW (NDVI Index).
2. Pulse → para ver la imagen 2.
3. La imagen 2 muestra automáticamente la capa RAW (NDVI Index).
4. Continúe navegando: todas las imágenes muestran la capa NDVI.
5. Muy eficaz para revisar los resultados del índice en muchas imágenes.

***

## Flujos de trabajo comunes

### Flujo de trabajo 1: Comparación antes/después

**Objetivo**: Comparar la imagen original con la imagen calibrada.

1. Abra la imagen procesada en el visor de imágenes.
2. Seleccione **RAW (Original)** en el menú desplegable.
3. Observe el viñeteado y los valores sin calibrar.
4. Cambie a **RAW (Reflectancia)** en el menú desplegable.
5. Compare: se ha eliminado el viñeteado y se han calibrado los valores.

### Flujo de trabajo 2: revisión del índice

**Objetivo**: revisar rápidamente los resultados de NDVI en todo el conjunto de datos.

1. Abra la primera imagen procesada.
2. Seleccione **RAW (Índice NDVI)** en el menú desplegable.
3. Utilice la tecla de flecha → para navegar a la siguiente imagen.
4. La capa NDVI persiste automáticamente.
5. Continúe con todas las imágenes, comprobando los patrones NDVI.
6. Cambie a **RAW (Índice NDRE)** para comparar.

### Flujo de trabajo 3: Verificación del objetivo

**Objetivo**: Verificar que todas las imágenes objetivo se hayan detectado correctamente.

1. Navegue hasta una imagen objetivo.
2. Seleccione **RAW (Objetivo)** en el menú desplegable.
3. Verifique que los objetivos de calibración sean claramente visibles y se hayan detectado.
4. Navegue hasta la siguiente imagen objetivo.
5. Repita la verificación para todos los objetivos.

### Flujo de trabajo 4: Inspección del valor de los píxeles

**Objetivo**: Comprobar los valores de reflectancia para garantizar la precisión científica.

1. Abra la imagen procesada.
2. Seleccione la capa **RAW (Reflectancia)**.
3. Active el modo **Porcentaje de píxeles** (botón en la barra de herramientas superior derecha).
4. Mueva el cursor sobre las áreas de vegetación.
5. Verifique que los valores de píxeles estén dentro de los rangos esperados (30-70 % para NIR, 5-15 % para Red).
6. Compruebe que las áreas de suelo y agua tengan valores adecuados.

***

## Comprensión de los valores de píxeles por capa

Las diferentes capas muestran diferentes rangos de valores de píxeles:

### Capa JPG

* **Rango**: 0-255 (8 bits)
* **Significado**: Valores de visualización, con corrección gamma
* **Uso**: Solo inspección visual, no para mediciones científicas

### RAW (Original)

* **Rango**: 0-65535 (16 bits)
* **Significado**: Números digitales del sensor sin procesar.
* **Uso**: Comprobación del rendimiento del sensor, sin calibrar.

### RAW (reflectancia)

* **Rango**: 0-65 535 (16 bits TIFF) o 0,0-1,0 (32 bits porcentaje)
* **Significado**: Porcentaje de reflectancia calibrado
* **Uso**: Mediciones y análisis científicos

**Para 16 bits TIFF:** Dividir por 65 535 para obtener el porcentaje de reflectancia **Para 32 bits Porcentaje:** Los valores representan directamente el porcentaje (0,5 = 50 % de reflectancia)

### RAW (imágenes de índice)

* **Rango**: varía según el índice (normalmente de -1,0 a +1,0 para índices normalizados)
* **Significado**: resultado del cálculo del índice
* **Ejemplos**:
  * NDVI: de -1 a +1 (vegetación normalmente de 0,4 a 0,9)
  * NDRE: de -1 a +1 (detección de estrés)
  * EVI: de 0 a 1 (vegetación mejorada)

***

## Consejos y prácticas recomendadas

### Cambio eficiente de capas

* **Conocimiento de los atajos de teclado**: Aunque no hay atajos de teclado para las capas, las flechas de navegación (←/→) funcionan en todas las capas
* **Flujos de trabajo coherentes**: elija una capa (por ejemplo, NDVI) y revise todo el conjunto de datos antes de cambiar a otra
* **Comparaciones rápidas**: alterne entre Original y Reflectancia para verificar la calidad del procesamiento

### Consideraciones sobre el rendimiento

* **JPG se carga más rápido**: utilícelo para navegar rápidamente por muchas imágenes.
* **Las capas RAW se cargan más lentamente**: mayor resolución y profundidad de bits.
* **Capas de índice**: velocidad similar a las capas de reflectancia.
* **La primera carga es la más lenta**: las vistas posteriores de la misma capa se almacenan en caché y son más rápidas.

### Verificación de la calidad

* **Compruebe siempre RAW (Original)**: verifique la calidad de los datos de origen antes de confiar en los resultados procesados.
* **Compare capas**: utilice el cambio de capas para validar que el procesamiento ha funcionado correctamente.
* **Compruebe los rangos de índice**: utilice el modo Porcentaje de píxeles con capas de índice para verificar que los valores son razonables.

***

## Solución de problemas

### Capa no disponible

**Problema**: La capa esperada no aparece en el menú desplegable.

**Posibles causas:**

* La imagen no se ha procesado (solo están disponibles JPG y RAW (original)).
* La calibración de reflectancia se desactivó durante el procesamiento.
* No se configuró un índice específico en la configuración del proyecto.
* La imagen es una imagen solo de destino (no se generaron índices para los destinos).

**Soluciones:**

1. Verifique que la imagen se haya procesado (compruebe la carpeta de salida para ver los archivos procesados).
2. Compruebe la configuración del proyecto para confirmar que se hayan configurado los índices.
3. Vuelva a procesar con los índices deseados habilitados.

### Se muestra una capa incorrecta

**Problema**: La imagen se abre en una capa inesperada.

**Causa**: Se ha transferido la preferencia de capa de la imagen anterior, pero esa capa no existe en la imagen actual.

**Solución**: Chloros vuelve automáticamente a JPG cuando la capa preferida no está disponible; este es un comportamiento normal.

### No se pueden ver los objetivos de calibración.

**Problema**: La capa RAW (objetivo) no muestra la detección de objetivos.

**Posibles causas:**

* Los objetivos no se detectaron durante el procesamiento.
* La imagen no contiene realmente objetivos.
* La configuración de detección de objetivos es demasiado estricta.

**Soluciones:**

1. Compruebe el registro de depuración en busca de mensajes de «Objetivo encontrado».
2. Verifique que la imagen contiene realmente objetivos de calibración visibles.
3. Ajuste la configuración de detección de objetivos en la configuración del proyecto.
4. Consulte [Selección de imágenes objetivo](../processing-images-gui/choosing-target-images.md).

***

## Funciones relacionadas

### Herramientas del visor de imágenes

Al visualizar cualquier capa, puede utilizar:

* **Controles de zoom**: amplíe para inspeccionar los detalles.
* **Panorámica**: haga clic y arrastre para desplazarse por la imagen ampliada.
* **Inspección del valor de los píxeles**: vea los valores en la ubicación del cursor.
* **Flechas de navegación**: muévase entre imágenes manteniendo la capa.
* **Modo Porcentaje de píxeles**: alterne entre la visualización DN y porcentual.

Consulte [Abrir una imagen a pantalla completa](opening-an-image-full-screen.md) para obtener la documentación completa del visor de imágenes.

### Índice/LUT Sandbox

Para pruebas y visualización interactivas del índice:

* **Cálculo del índice en tiempo real**: pruebe diferentes fórmulas de índice.
* **Asignación de colores LUT**: aplique degradados de color a los índices en escala de grises.
* **Exportar visualizaciones**: guarde imágenes de índice en color.

Consulte [Área de pruebas de índice/LUT](index-lut-sandbox.md) para obtener más detalles.

***

## Próximos pasos

Ahora que ya conoce las capas de imagen:

* [**Abrir una imagen a pantalla completa**](opening-an-image-full-screen.md): guía completa del visor de imágenes.
* [**Index/LUT Sandbox**](index-lut-sandbox.md): visualización interactiva de índices.
* [**Fórmulas de índices multiespectrales**](../project-settings/multispectral-index-formulas.md): referencia de índices disponibles
* [**Finalización del procesamiento**](../processing-images-gui/finishing-the-processing.md): comprensión de los resultados procesados

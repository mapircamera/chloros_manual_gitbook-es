# Capas de imagen

El menú desplegable «Capas de imagen» del visor de imágenes Chloros te permite cambiar rápidamente entre diferentes versiones de la misma imagen, desde las capturas originales hasta los resultados de reflectancia procesados y las imágenes de índice calculadas.

## ¿Qué son las capas de imagen?

En Chloros, las **capas** se refieren a los diferentes resultados de imagen disponibles para una única imagen de origen. Al procesar imágenes, Chloros crea varias versiones:

* **Imágenes originales** (archivos JPG y RAW de su cámara)
* Resultados **con reflectancia calibrada** (si se ha habilitado la calibración de reflectancia)
* **Imágenes de referencia** (si la imagen contiene marcas de calibración)
* **Imágenes de índice** (NDVI, NDRE, GNDVI, etc., si se han configurado índices)

El **menú desplegable Selector de capas** situado en la parte superior derecha del visor de imágenes le permite cambiar al instante entre estas versiones sin salir del visor.***

## Tipos de capas disponibles

### JPG

* La imagen de vista previa JPG original de su cámara
* Siempre disponible para todas las imágenes
* Sin procesar, tal y como la capturó la cámara
* La más rápida de cargar y mostrar

**Cuándo verla:**

* Vista previa rápida de la captura original
* Comprobación de la composición y el encuadre de la imagen
* Verificación de la calidad de la captura antes del procesamiento

### RAW (Original)

* Los datos RAW originales del sensor de su cámara
* Desbayereados sin aplicar ningún procesamiento posterior
* Mayor profundidad de bits que el JPG (normalmente datos del sensor de 12 o 14 bits)

**Cuándo verlo:**

* Inspeccionar la calidad de los datos originales del sensor
* Comprobar si hay problemas en el sensor o artefactos
* Comparar los resultados antes y después del procesamiento

### RAW (Objetivo)

* Solo aparece para imágenes identificadas como que contienen objetivos de calibración
* Muestra la imagen RAW original con el objetivo detectado
* Se utiliza para verificar que la detección del objetivo se ha realizado correctamente

**Cuándo visualizar:**

* Para confirmar que los objetivos de calibración se han detectado correctamente
* Para comprobar la calidad de la imagen del objetivo
* Para solucionar problemas de calibración

{% hint style="info" %}
**Capa de objetivo**: Esta capa solo aparece en el menú desplegable para imágenes que contienen objetivos de calibración. Las imágenes de captura normales no tendrán esta opción.
{% endhint %}

### RAW (Reflectancia)

* La imagen de salida de reflectancia calibrada
* Con corrección de viñeteado (si se ha habilitado en el procesamiento)
* Reflectancia calibrada utilizando datos de objetivos (si está habilitado)
* Multibanda TIFF con todos los canales de la cámara
* Los valores de píxel representan el porcentaje de reflectancia (cuando se utiliza el modo porcentual)
* Listo para manipular con el [Index/LUT Sandbox](index-lut-sandbox.md)

**Cuándo visualizar:**

* Inspeccionar los resultados calibrados
* Verificar la calidad de la calibración
* Comprobar los valores de los píxeles para garantizar la precisión científica
* Comparar con el original para ver los efectos de la calibración

{% hint style="success" %}
**Recomendado**: Utilice la capa RAW (Reflectancia) al comprobar los valores de los píxeles para mediciones y análisis científicos.
{% endhint %}

### RAW (Índice NDVI)... y similares

* Imagen del índice de vegetación calculado (NDVI en este ejemplo)
* El nombre del índice cambia en función del índice que se haya configurado durante el procesamiento
* Ejemplos: RAW (Índice NDVI), RAW (Índice NDRE), RAW (Índice GNDVI), etc.
* Imagen en escala de grises de una sola banda que muestra los resultados del cálculo del índice
* Aparece una capa por cada índice configurado en los Ajustes del proyecto

**Posibles nombres de índices:**

* RAW (Índice NDVI)
* RAW (Índice NDRE)
* RAW (Índice GNDVI)
* RAW (Índice OSAVI)
* RAW (Índice EVI)
* RAW (Índice SAVI)
* Y muchos más... (véase [Fórmulas de índices multiespectrales](../project-settings/multispectral-index-formulas.md))

**Cuándo visualizarlas:**

* Examinar los resultados del cálculo de índices
* Comprobar los rangos de valores de los índices
* Identificar áreas de interés
* Verificar las imágenes de índices antes de utilizarlas en SIG o en análisis

***

## Uso del selector de capas

### Abrir el menú desplegable

1. Abre una imagen en modo de pantalla completa (haz clic en cualquier miniatura del visor de imágenes)
2. Localice el **menú desplegable de capas** en la esquina superior derecha del visor
3. El menú desplegable muestra la capa seleccionada actualmente (por ejemplo, «JPG»)
4. Haga clic en el menú desplegable para ver todas las capas disponibles

### Cambiar de capa

1. Haga clic en el menú desplegable de capas para abrir la lista
2. Se muestran todas las capas disponibles para la imagen actual
3. Haga clic en el nombre de cualquier capa para cambiar a esa versión
4. La imagen se actualiza inmediatamente para mostrar la capa seleccionada

**Cambio rápido:**

* El menú desplegable recuerda tu última selección
* Al pasar a la siguiente imagen, Chloros intenta mostrar el mismo tipo de capa
* Si esa capa no existe en la siguiente imagen, se establece por defecto en JPG

### Disponibilidad de capas

No todas las capas están disponibles para todas las imágenes:

**Siempre disponibles:*** ✅ JPG (todas las imágenes tienen una vista previa en JPG)

**Disponibles de forma condicional:**

* ⚠️ RAW (Original): solo si la imagen se capturó en modo RAW o RAW+JPG
* ⚠️ RAW (Objetivo): solo si la imagen contiene objetivos de calibración detectados
* ⚠️ RAW (Reflectancia): solo tras el procesamiento con la calibración de reflectancia activada
* ⚠️ RAW (\[Índice] Índice): solo tras el procesamiento con los índices configurados

***

## Persistencia de capas

### Navegación entre imágenes

Al pasar a una imagen diferente (utilizando las teclas de flecha o haciendo clic en las miniaturas):**Se conserva la preferencia de capa:**

* Si se está viendo «RAW (Reflectancia)», la siguiente imagen muestra «RAW (Reflectancia)» (si está disponible)
* Si se está viendo «RAW (NDVI Índice)», la siguiente imagen muestra «RAW (NDVI Índice)» (si está disponible)
* Si no existe la misma capa, se establece por defecto en JPG

**Ejemplo de flujo de trabajo:**

1. Abra la Imagen 1, cambie a RAW (Índice NDVI)
2. Pulse → para ver la Imagen 2
3. La Imagen 2 muestra automáticamente la capa RAW (NDVI Index)
4. Continúe navegando: todas las imágenes muestran la capa NDVI
5. Muy eficiente para revisar los resultados del índice en muchas imágenes

***

## Flujos de trabajo habituales

### Flujo de trabajo 1: Comparación antes/después

**Objetivo**: Comparar la imagen original con la calibrada

1. Abra la imagen procesada en el visor de imágenes
2. Seleccione **RAW (Original)** en el menú desplegable
3. Observe el viñeteado y los valores sin calibrar
4. Cambie a **RAW (Reflectancia)** en el menú desplegable
5. Compare: viñeteado eliminado, valores calibrados

### Flujo de trabajo 2: Revisión del índice

**Objetivo**: Revisar rápidamente los resultados de NDVI en todo el conjunto de datos

1. Abre la primera imagen procesada
2. Selecciona **RAW (Índice NDVI)** en el menú desplegable
3. Utilice la tecla de flecha → para pasar a la siguiente imagen
4. La capa NDVI se mantiene automáticamente
5. Continúe con todas las imágenes, comprobando los patrones de NDVI
6. Cambie a **RAW (Índice NDRE)** para comparar

### Flujo de trabajo 3: Verificación de objetivos

**Objetivo**: Verificar que todas las imágenes de objetivos se hayan detectado correctamente

1. Navegue hasta una imagen de objetivo
2. Seleccione **RAW (Objetivo)** en el menú desplegable
3. Verifique que los objetivos de calibración sean claramente visibles y se hayan detectado
4. Navegue hasta la siguiente imagen de objetivo
5. Repita la verificación para todos los objetivos

### Flujo de trabajo 4: Inspección del valor de los píxeles

**Objetivo**: Comprobar los valores de reflectancia para garantizar la precisión científica

1. Abra la imagen procesada
2. Seleccione la capa **RAW (Reflectancia)**

3. Active el modo**Porcentaje de píxeles** (botón en la barra de herramientas superior derecha)
4. Mueva el cursor sobre las zonas de vegetación
5. Compruebe que los valores de píxel se encuentran dentro de los rangos esperados (30-70 % para NIR, 5-15 % para Red)
6. Compruebe que las áreas de suelo y agua presentan valores adecuados

***

## Comprensión de los valores de píxel por capa

Las diferentes capas muestran distintos rangos de valores de píxel:

### Capa JPG

* **Rango**: 0-255 (8 bits)
* **Significado**: Valores de visualización, con corrección gamma
* **Uso**: Solo para inspección visual, no para mediciones científicas

### RAW (Original)

* **Rango**: 0-65535 (16 bits)
* **Significado**: Números digitales sin procesar del sensor
* **Uso**: Comprobación del rendimiento del sensor, sin calibrar

### RAW (Reflectancia)

* **Rango**: 0-65 535 (16 bits TIFF) o 0,0-1,0 (32 bits Porcentaje)
* **Significado**: Porcentaje de reflectancia calibrado
* **Uso**: Mediciones y análisis científicos**Para 16 bits TIFF:**Dividir entre 65 535 para obtener el porcentaje de reflectancia**Para 32 bits Porcentaje:** Los valores representan directamente el porcentaje (0,5 = 50 % de reflectancia)

### RAW (Imágenes de índice)

* **Rango**: Varía según el índice (normalmente de -1,0 a +1,0 para índices normalizados)
* **Significado**: Resultado del cálculo del índice
* **Ejemplos**:
  * NDVI: de -1 a +1 (la vegetación suele estar entre 0,4 y 0,9)
  * NDRE: de -1 a +1 (detección de estrés)
  * EVI: de 0 a 1 (vegetación mejorada)

***

## Consejos y buenas prácticas

### Cambio eficiente de capas

* **Atención a los atajos de teclado**: Aunque no hay atajos de teclado para las capas, las flechas de navegación (←/→) funcionan en todas las capas
* **Flujos de trabajo coherentes**: Elija una capa (p. ej., NDVI) y revise todo el conjunto de datos antes de cambiar a otra
* **Comparaciones rápidas**: Alterne entre Original y Reflectancia para verificar la calidad del procesamiento

### Consideraciones de rendimiento

* **El formato JPG se carga más rápido**: utilícelo para navegar rápidamente por muchas imágenes
* **Las capas RAW se cargan más lentamente**: mayor resolución y profundidad de bits
* **Capas de índice**: velocidad similar a la de las capas de reflectancia
* **La primera carga es la más lenta**: las visualizaciones posteriores de la misma capa se almacenan en caché y son más rápidas

### Verificación de la calidad

* **Compruebe siempre el formato RAW (Original)**: Verifique la calidad de los datos de origen antes de confiar en los resultados procesados
* **Compare capas**: utilice el cambio de capa para validar que el procesamiento ha funcionado correctamente
* **Compruebe los rangos de índice**: utilice el modo Porcentaje de píxeles con capas de índice para verificar que los valores son razonables***

## Solución de problemas

### Capa no disponible

**Problema**: la capa esperada no aparece en el menú desplegable**Posibles causas:**

* La imagen no se ha procesado (solo están disponibles JPG y RAW (Original))
* La calibración de reflectancia estaba desactivada durante el procesamiento
* No se ha configurado un índice específico en los ajustes del proyecto
* La imagen es una imagen solo de objetivos (no se generan índices para los objetivos)

**Soluciones:**

1. Comprueba que la imagen se haya procesado (busca los archivos procesados en la carpeta de salida)
2. Comprueba los ajustes del proyecto para confirmar que se hayan configurado los índices
3. Vuelva a procesar con los índices deseados habilitados

### Se muestra una capa incorrecta

**Problema**: La imagen se abre en una capa inesperada**Causa**: Se ha mantenido la preferencia de capa de la imagen anterior, pero esa capa no existe en la imagen actual**Solución**: Chloros recurre automáticamente al formato JPG cuando la capa preferida no está disponible; este es un comportamiento normal

### No se ven los objetivos de calibración

**Problema**: La capa RAW (Objetivo) no muestra la detección de objetivos**Posibles causas:**

* No se detectaron objetivos durante el procesamiento
* La imagen no contiene realmente objetivos
* La configuración de detección de objetivos es demasiado estricta

**Soluciones:**

1. Compruebe el registro de depuración en busca de mensajes de «Objetivo encontrado»
2. Verifique que la imagen contiene realmente objetivos de calibración visibles
3. Ajuste la configuración de detección de objetivos en la configuración del proyecto
4. Consulte [Selección de imágenes objetivo](../processing-images-gui/choosing-target-images.md)

***

## Funciones relacionadas

### Herramientas del visor de imágenes

Al visualizar cualquier capa, puede utilizar:

* **Controles de zoom**: amplíe la imagen para inspeccionar los detalles
* **Desplazamiento**: haga clic y arrastre para desplazarse por la imagen ampliada
* **Inspección del valor de píxeles**: vea los valores en la ubicación del cursor
* **Flechas de navegación**: desplácese entre imágenes sin cambiar de capa
* **Modo Porcentaje de píxeles**: alterne entre la visualización de DN y porcentaje

Consulte [Abrir una imagen a pantalla completa](opening-an-image-full-screen.md) para obtener la documentación completa del visor de imágenes.

### Entorno de pruebas de índices/LUT

Para pruebas interactivas de índices y visualización:

* **Cálculo de índices en tiempo real**: Prueba diferentes fórmulas de índices
* **Asignación de colores LUT**: Aplica degradados de color a índices en escala de grises
* **Exportar visualizaciones**: Guarda imágenes de índices en color

Consulta [Entorno de pruebas de índices/LUT](index-lut-sandbox.md) para obtener más detalles.

***

## Próximos pasos

Ahora que ya conoce las capas de imagen:

* [**Abrir una imagen a pantalla completa**](opening-an-image-full-screen.md) - Guía completa del visor de imágenes
* [**Index/LUT Sandbox**](index-lut-sandbox.md) - Visualización interactiva de índices
* [**Fórmulas de índices multiespectrales**](../project-settings/multispectral-index-formulas.md) - Referencia de índices disponibles
* [**Finalización del procesamiento**](../processing-images-gui/finishing-the-processing.md) - Comprensión de los resultados procesados

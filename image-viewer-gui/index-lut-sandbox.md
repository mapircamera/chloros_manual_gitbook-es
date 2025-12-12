# Sandbox de índices/LUT

El Sandbox de índices/LUT es un espacio de trabajo interactivo dentro del visor de imágenes Chloros que le permite experimentar con cálculos de índices multiespectrales y visualizaciones de color en tiempo real. Esta potente herramienta le ayuda a probar diferentes índices, refinar rangos de valores y crear visualizaciones listas para su publicación sin necesidad de volver a procesar todo su conjunto de datos.

## ¿Qué es el Sandbox de índices/LUT?

### Finalidad

El Sandbox ofrece:

* **Cálculo de índices en tiempo real**: aplique cualquier índice de vegetación al instante.
* **Ajuste interactivo de LUT**: ajuste con precisión los degradados y rangos de color.
* **Optimización del flujo de trabajo**: determine la mejor configuración antes del procesamiento por lotes.

### Sandbox frente a procesamiento de proyectos

**Sandbox de índices/LUT (interactivo):**

* Una sola imagen a la vez.
* Respuesta instantánea.
* Experimental e iterativo.
* Sin cambios permanentes en los archivos.
* Perfecto para explorar y probar.

**Procesamiento de proyectos (por lotes):**

* Todo el conjunto de datos a la vez
* Ajustes preconfigurados
* Archivos de salida permanentes
* Requiere mucho tiempo
* Ideal cuando los ajustes están finalizados

{% hint style=&quot;success&quot; %}
**Mejor flujo de trabajo**: utilice el entorno de pruebas para experimentar y encontrar la configuración óptima del índice y la LUT, y luego aplique esa configuración durante el procesamiento del proyecto para todo el conjunto de datos.
{% endhint %}

***

## Trabajar con el entorno de pruebas del índice/LUT

### Comprensión de los índices precalculados

En Chloros, los índices se pueden aplicar durante el procesamiento del proyecto. Para determinar qué ajustes de índice y LUT desea aplicar a las exportaciones, lo más fácil es utilizar el entorno de pruebas del visor de imágenes.

El entorno de pruebas le permite:

* **Aplicar nuevos índices y gradientes de color (LUT)** para visualizar los datos.
* **Ajustar la configuración de visualización** de forma interactiva.
* **Ver** imágenes de índices ya calculadas.
* **Inspeccionar** los valores de píxeles en todos los niveles de zoom.

### Abrir el entorno de pruebas

Se accede al entorno de pruebas de índices/LUT en la pestaña **Visor de imágenes** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. Haga clic en una imagen en la cuadrícula de imágenes del explorador de archivos y se abrirá en la pestaña **Visor de imágenes** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> .
2. Haga clic en la pestaña **Visor de imágenes** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> para abrir la barra lateral emergente de la izquierda si aún no está abierta

### Seleccionar una imagen para aplicar un índice/LUT

Para trabajar con un índice en el Sandbox del visor de imágenes <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. **Abra una imagen** de la cuadrícula de imágenes principal haciendo clic en ella
2. Se abrirá la pestaña **Visor de imágenes** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> .
3. Haga clic en el **menú desplegable Capa** (parte superior derecha del visor).
4. Seleccione la capa en el menú desplegable:
   * RAW (reflectancia)

### Aplicar un índice a una imagen

Una vez que la imagen esté en pantalla completa y la barra lateral de la pestaña **Visor de imágenes** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> esté abierta:

1. Marque la casilla Índice en la parte superior de la barra lateral.
2. Elija el filtro de su cámara en el menú desplegable de la izquierda.
3. Elija la fórmula de índice deseada en el menú desplegable de la derecha.
4. Arrastre los círculos de color del canal del filtro a las ubicaciones de la fórmula de índice que aparece a continuación.
5. Una vez que la fórmula sea válida, la imagen se actualizará y mostrará los valores del índice.
6. Mueva el cursor del ratón para ver los valores en la ubicación del cursor.
7. Amplíe la imagen para ver los píxeles individuales y sus valores asociados.

Cada índice tiene un rango de valores y un significado específicos:

#### NDVI Ejemplo

```
Formula: (NIR - Red) / (NIR + Red)

For Survey3W RGN camera:
NIR = 850nm band
Red = 661nm band

Result range: -1.0 to +1.0
Typical vegetation: 0.4 to 0.9
Stressed vegetation: 0.2 to 0.4
Bare soil: 0.0 to 0.2
Water: -0.1 to 0.1
```

Para obtener la documentación completa de la fórmula del índice, consulte [Fórmulas de índices multiespectrales](../project-settings/multispectral-index-formulas.md).

***

## Trabajar con LUT (tablas de consulta)

### ¿Qué es una LUT?

Una **tabla de consulta (LUT)** asigna valores numéricos de índice a colores para su visualización:

* **Entrada**: valor de píxel del índice (por ejemplo, NDVI 0,65)
* **Salida**: color RGB (por ejemplo, verde brillante)
* **Finalidad**: facilitar la visualización e interpretación de los patrones

**LUT en escala de grises frente a LUT en color:**

* Escala de grises: científica y neutra, muestra los datos sin procesar
* LUT en color: intuitiva e impactante, resalta los patrones y las diferencias

{% hint style=&quot;success&quot; %}
**Capacidad de visualización**: la aplicación de una LUT de color a una imagen índice en escala de grises facilita enormemente la identificación de patrones, anomalías y áreas de interés de un solo vistazo.
{% endhint %}

### Aplicación de una LUT a una imagen de índice

Una vez que tenga una imagen de índice que muestre

1. Haga clic en el <img src="../.gitbook/assets/image.png" alt="" data-size="line"> botón «+Añadir LUT»
2. Seleccione el degradado de color
3. Ajuste los puntos finales mínimo/máximo del recorte
4. Ajuste el modo de recorte
5. Marque la casilla Índice en la barra lateral del **Visor de imágenes** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> para aplicar la LUT.

### Elección de un degradado de color

**Selección de un degradado:**

1. En el panel LUT, localice la **barra de degradado de color**.
2. Pase el ratón por encima para ver los ajustes preestablecidos de degradado disponibles.
3. Seleccione el degradado deseado.
4. La imagen **se actualiza inmediatamente** con los nuevos colores cuando se marca la casilla Índice.

{% hint style=&quot;success&quot; %}
**Práctica recomendada**: Para índices de vegetación como NDVI, el degradado Red-Amarillo-Green es el más intuitivo, ya que se ajusta a las asociaciones de colores naturales (verde = saludable, amarillo = moderado, rojo = estresado).
{% endhint %}

### Ajuste de las clases de color

El **control Clases** determina cuántos pasos de color discretos aparecen en el degradado:

**Opciones de recuento de clases:**

* **2-5 clases**: categorías muy amplias, zonas distintas
* **6-10 clases**: equilibradas, buenas para la clasificación
* **11-20 clases**: degradados suaves, apariencia continua
* **Más de 20 clases**: casi continuas, máxima suavidad

**Cómo ajustar:**

1. En el panel LUT, localice los **cuadrados de muestras de color debajo de la barra de degradado**
2. Ajuste el número de clases añadiendo con el botón +
3. Elimine el número de clases haciendo doble clic en una muestra de color.
4. El degradado se actualiza **en tiempo real** en la imagen.

**Efecto en la visualización:**

* **Menos clases** (3-5): crea zonas diferenciadas, clasificación simplificada, categorías más fáciles de distinguir.
* **Clases medias** (6-10): enfoque equilibrado, adecuado para la mayoría de las aplicaciones.
* **Más clases** (15-20): transiciones suaves, variación detallada, aspecto fotográfico.

**Cuándo utilizarlo:**

* **Pocas clases (3-5)**: diapositivas de presentación, mapas de clasificación, informes sencillos.
* **Clases medias (6-10)**: análisis general, detalle equilibrado, informes estándar.
* **Muchas clases (15-20)**: análisis científico, inspección detallada, resultados con calidad de publicación.

### Ajuste de los rangos de valores

Los **controles de rango de valores** determinan qué valores de índice se asignan a qué colores en el degradado:

**Controles de rango en el panel LUT:**

* **Valor mínimo**: límite inferior de la escala de colores.
* **Valor máximo**: límite superior de la escala de colores
* **Valores intermedios**: distribuidos automáticamente entre el mínimo y el máximo (en función del recuento de clases)

#### Ajuste de los valores mínimo y máximo

**Para ajustar los rangos de valores:**

1. En el panel LUT, localice los campos de entrada **Valor mínimo** y **Valor máximo**
2. Haga clic en el campo **Valor mínimo**
3. Escriba el valor mínimo deseado (por ejemplo, `0.2`)
4. Pulse **Intro** o haga clic fuera del campo
5. Repita el proceso para el campo **Valor máximo** (por ejemplo, `0.9`)
6. La visualización **se actualiza inmediatamente**

{% hint style=&quot;info&quot; %}
**Escalado automático**: cuando se aplica una LUT por primera vez, Chloros establece automáticamente el mínimo/máximo en el rango de datos real de la imagen. A continuación, puede reducir este rango para centrarse en rangos de valores específicos de interés.
{% endhint %}

**Ejemplo de ajustes de rango NDVI:**

* **Rango completo**: `-1.0` a `1.0` (mostrar todos los valores posibles)
* **Centrado en la vegetación**: `0.2` a `0.9` (excluye el suelo desnudo y el agua)
* **Solo vegetación sana**: `0.5` a `0.9` (resalta solo las plantas vigorosas)
* **Detección de estrés**: `0.2` a `0.5` (enfatizar las áreas problemáticas)
* **Rango personalizado**: ajustar en función de los valores de píxeles observados

**¿Por qué ajustar los rangos?**

* **Aumentar el contraste** en su área de interés
* **Excluir valores irrelevantes** (por ejemplo, masas de agua, suelo desnudo)
* **Estandarizar la visualización** en varias imágenes o fechas
* **Destacar diferencias sutiles** dentro de un rango de valores estrecho

### Recortar valores fuera de rango

Cuando los valores de píxeles quedan fuera del rango mínimo/máximo definido, puede controlar cómo se muestran utilizando los **modos de recorte**.

#### **Opciones de modo de recorte disponibles:**

#### 1. Mínimo y máximo

* Píxeles **por debajo del mínimo** → se muestran utilizando el **primer color** del degradado (por ejemplo, rojo)
* Píxeles **por encima del máximo** → se muestran utilizando el **último color** del degradado (por ejemplo, verde)
* **Caso de uso**: enfatizar los extremos, mostrar el rango completo de datos con colores saturados en los límites
* **Ejemplo**: los valores NDVI por debajo de 0,2 aparecen todos en rojo, los valores por encima de 0,9 aparecen todos en verde

#### 2. Fondo transparente

* Los píxeles **fuera del rango** se vuelven **totalmente transparentes**
* Solo los píxeles **dentro del rango** muestran el degradado de color
* **Caso de uso**: superposición GIS, aislamiento de rangos de valores específicos, resaltado solo de áreas de interés
* **Ejemplo**: mostrar solo NDVI 0,4-0,7 en color, todo lo demás transparente

{% hint style=&quot;warning&quot; %}
**Limitación de transparencia**: los píxeles transparentes aparecerán como el color de fondo en el visor. Cuando se exportan durante el procesamiento, la transparencia se conserva en el formato PNG, pero no en JPG.
{% endhint %}

#### 3. Fondo del índice

* Los píxeles **fuera del rango** se muestran en **escala de grises** (mostrando los valores del índice sin procesar).
* Los píxeles **dentro del rango** muestran un **gradiente de color**
* **Caso de uso**: resaltado sutil, mantiene el contexto mientras enfatiza las áreas de interés
* **Ejemplo**: resalta con color la vegetación estresada (NDVI 0,3-0,5) mientras muestra las áreas sanas en gris

#### 4. Fondo original

* Los píxeles **fuera del rango** muestran la **imagen multiespectral original**.
* Los píxeles **dentro del rango** muestran un **gradiente de color**.
* **Caso de uso**: el más intuitivo, combina el contexto natural de la imagen con una superposición analítica de color.
* **Ejemplo**: ver el aspecto real del campo/cultivo con las áreas estresadas superpuestas y codificadas por colores.

### Elegir el modo de recorte adecuado

| Modo de recorte              | Ideal para                                   | Estilo de visualización          |
| -------------------------- | ------------------------------------------ | ---------------------------- |
| **Mínimo y máximo**    | Visualización completa de datos, análisis científico     | Todos los píxeles coloreados           |
| **Fondo transparente** | Superposiciones GIS, aislamiento de rangos específicos    | Color en el rango, en blanco fuera de él |
| **Fondo de índice**       | Énfasis sutil, mantenimiento del contexto de los datos  | Color en el rango, gris fuera de él  |
| **Fondo original**    | Informes, presentaciones, análisis intuitivo | Color en el rango, foto fuera de él |

### Creación de colores LUT personalizados

Para tener un control total sobre la visualización, puede crear **gradientes de color personalizados** editando los paradas de color individuales.

**Para crear un gradiente personalizado:**

1. En el panel LUT, localice la **barra de vista previa del gradiente**.
2. Busque los **cuadrados de muestras de color** debajo del gradiente.
3. **Haga clic en una parada de color** para seleccionarla.
4. Se abrirá un **selector de color**.
5. Elija un nuevo color utilizando:
   * **Rueda de colores**: selección visual de colores.
   * **Controles deslizantes RGB/HSV**: control preciso del color.
   * **Entrada de código hexadecimal**: especificación exacta del color (por ejemplo, `#FF0000` para el rojo).
6. Haga clic fuera del selector de color **para aplicar el nuevo color**.
7. El degradado **se actualiza inmediatamente** en la imagen.

**Añadir o eliminar paradas de color:**

* **Añadir una parada**: haga clic en el icono + para añadir una nueva muestra al final.
* **Eliminar una parada**: haga doble clic en el cuadrado de color para eliminar la muestra.

**Estrategias de personalización:**

* **Invertir degradado**: invierta el orden de los colores para invertir el significado (por ejemplo, verde = bajo, rojo = alto).
* **Colores de marca**: combine la paleta de colores de su organización para los informes.
* **Apto para daltónicos**: utilice combinaciones de naranja-azul o morado-amarillo.
* **Optimización de la impresión**: elija colores que funcionen tanto en la impresión en color como en escala de grises.
* **Múltiples umbrales**: utilice colores distintos en umbrales de valores específicos para la clasificación.

{% hint style=&quot;info&quot; %}
**Guardar degradados personalizados**: los degradados personalizados se pueden guardar y reutilizar. Haga clic en el icono Guardar del panel LUT para conservar sus combinaciones de colores personalizadas para su uso futuro.
{% endhint %}

***

## Flujo de trabajo interactivo

### Actualizaciones en tiempo real

Todos los ajustes de LUT en el entorno de pruebas actualizan la imagen **de forma instantánea e interactiva**:

* **Cambiar capa** → La imagen cambia inmediatamente
* **Seleccionar degradado** → Los colores se actualizan al instante
* **Ajustar rango de valores** → El contraste cambia en tiempo real
* **Cambiar clases** → La suavidad del degradado se actualiza inmediatamente
* **Modificar recorte** → La visualización del fondo cambia al instante
* **Editar colores** → El degradado personalizado se aplica inmediatamente

**No es necesario el botón «Aplicar»**: ¡todos los cambios son en directo e interactivos!

{% hint style=&quot;success&quot; %}
**Comentarios en directo**: los comentarios visuales instantáneos le permiten experimentar rápidamente con diferentes ajustes hasta encontrar la visualización óptima para sus necesidades de análisis.
{% endhint %}

### Flujo de trabajo de refinamiento iterativo

**Flujo de trabajo típico de optimización de LUT:**

1. **Seleccionar capa de índice** (por ejemplo, RAW (reflectancia))
2. **Aplicar índice**: elija el filtro de la cámara y la fórmula del índice, arrastre los círculos de color a la ubicación adecuada en la fórmula del índice
3. **Aplicar gradiente LUT**: comience con el ajuste preestablecido Red-Yellow-Green
4. **Inspeccionar los valores de los píxeles**: mueva el cursor y observe los rangos de valores
5. **Ajustar mín./máx.**: reducir para centrarse en la vegetación (por ejemplo, de 0,2 a 0,9).
6. **Elegir recorte**: probar «Fondo original» para el contexto.
7. **Refinar colores**: personalizar el degradado si es necesario para enfatizar algo específico.
8. **Finalizar la configuración**: documentar la configuración y copiarla en la configuración del proyecto para el procesamiento de exportación.

### Inspección del valor de los píxeles

Comprender los valores reales de los píxeles es fundamental para establecer rangos LUT eficaces:

**Cómo inspeccionar los valores:**

1. Los valores de píxeles se muestran cuando la imagen tiene **marcadas** las casillas Índice o Índice y LUT.
2. **Mueva el cursor** sobre diferentes áreas de la imagen.
3. **Observe los valores de píxeles** que se muestran en la leyenda al pasar el cursor por encima.
4. Amplíe la imagen para ver los píxeles individuales resaltados con un valor flotante.
5. **Tome nota** de los rangos de valores para las diferentes características:
   * **Vegetación sana**: por ejemplo, NDVI 0,55-0,85.
   * **Vegetación estresada**: por ejemplo, NDVI 0,30-0,50.
   * **Suelo desnudo**: por ejemplo, NDVI 0,05-0,25
   * **Agua** (si está presente): por ejemplo, NDVI -0,05 a 0,10

**Uso de valores de píxeles para establecer rangos LUT:**

Después de inspeccionar los valores de píxeles, ajuste el mínimo y el máximo de LUT según corresponda:

**Ejemplo de escenario:**

* **Observación**: Valores del suelo = 0,05-0,25, estresado = 0,25-0,50, sano = 0,50-0,85
* **Objetivo**: Visualizar solo la salud de las plantas (excluir el suelo)
* **Ajustes LUT**: Mín. = `0.25`, Máx. = `0.85`
* **Recorte**: «Fondo original» para ver el suelo en su color natural
* **Resultado**: El degradado de color solo se aplica a la vegetación, el suelo se muestra como en la imagen original

{% hint style=&quot;info&quot; %}
**Rango dinámico**: los diferentes cultivos, estaciones y etapas de crecimiento tendrán diferentes rangos de valores. Compruebe siempre los valores de píxeles en su conjunto de datos específico antes de establecer los rangos LUT.
{% endhint %}

***

## Índices personalizados (Chloros+)

### Creación de fórmulas de índices personalizados

{% hint style=&quot;info&quot; %}
**Dónde crear**: Los índices personalizados se pueden configurar en **Configuración del proyecto** antes del procesamiento, así como en la barra lateral del visor de imágenes.
{% endhint %}

**Para crear un índice personalizado:**

1. **Abra la configuración del proyecto** (antes del procesamiento) o la barra lateral del visor de imágenes.
2. Vaya al **menú desplegable Fórmula del índice**.
3. Busque la opción **«Personalizado»** (debe iniciar sesión con la licencia Chloros+).
4. **Defina su fórmula** utilizando variables de banda:
   * Nombres de banda: `NIR`, `Red`, `Green`, `Blue`, `RedEdge`, etc.
   * Operadores: `+`, `-`, `*`, `/`, `^` (exponente)
   * Funciones: `sqrt()`, `abs()`, etc. (si son compatibles)
   * Paréntesis: `()` para el orden de las operaciones
5. **Asigne un nombre a su índice** (por ejemplo, «MyIndex» o «CustomNDVI»).
6. **Guarde la configuración**.

**Ejemplos de fórmulas personalizadas:**

```
Modified NDVI with offset:
(NIR - Red) / (NIR + Red + 0.5)

Simple ratio:
NIR / Red

Complex multi-band:
(NIR - Red) / (NIR + Red - Blue)

Exponential index:
(NIR / Red) ^ 2
```

{% hint style=&quot;warning&quot; %}
**Validación de fórmulas**: asegúrese de que su fórmula utiliza bandas disponibles en su cámara. Por ejemplo, RedEdge solo está disponible en cámaras con un filtro RedEdge.
{% endhint %}

***

## Próximos pasos

Ahora que ya conoce el índice/LUT Sandbox:

* **Aplicar al procesamiento**: utilice la configuración descubierta en [Configuración del proyecto](../project-settings/project-settings.md)
* **Procesamiento por lotes**: aplique los índices optimizados a conjuntos de datos completos
* **Más información**: lea [Fórmulas de índices multiespectrales](../project-settings/multispectral-index-formulas.md)

Documentación relacionada:

* [**Capas de imagen**](image-layers.md) - Gestión y visualización de capas
* [**Abrir una imagen a pantalla completa**](opening-an-image-full-screen.md): conceptos básicos del visor de imágenes
* [**Procesamiento de imágenes (GUI)**](../processing-images-gui/adding-files-to-a-project.md): flujo de trabajo completo de procesamiento

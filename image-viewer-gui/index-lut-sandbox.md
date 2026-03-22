# Entorno de pruebas de índices/LUT

El entorno de pruebas de índices/LUT es un espacio de trabajo interactivo integrado en el visor de imágenes Chloros que permite experimentar con cálculos de índices multiespectrales y visualizaciones en color en tiempo real. Esta potente herramienta ayuda a probar diferentes índices, ajustar los rangos de valores y crear visualizaciones listas para su publicación sin necesidad de volver a procesar todo el conjunto de datos.

## ¿Qué es el Index/LUT Sandbox?

### Objetivo

El Sandbox ofrece:

* **Cálculo de índices en tiempo real**: aplica cualquier índice de vegetación al instante
* **Ajuste interactivo de LUT**: ajusta con precisión los gradientes y rangos de color
* **Optimización del flujo de trabajo**: determina la mejor configuración antes del procesamiento por lotes

### Sandbox frente al procesamiento de proyectos

**Sandbox de índices/LUT (interactivo):**

* Una sola imagen cada vez
* Respuesta instantánea
* Experimental e iterativo
* Sin cambios permanentes en los archivos
* Perfecto para explorar y probar

**Procesamiento de proyectos (por lotes):**

* Conjunto de datos completo de una sola vez
* Ajustes preconfigurados
* Archivos de salida permanentes
* Requiere mucho tiempo
* Ideal cuando los ajustes están finalizados

{% hint style="success" %}
**Mejor flujo de trabajo**: Utilice el entorno de pruebas para experimentar y encontrar los ajustes óptimos de índice y LUT, y luego aplique esos ajustes durante el procesamiento del proyecto a todo su conjunto de datos.
{% endhint %}

***

## Trabajar con el entorno de pruebas de índices/LUT

### Comprender los índices precalculados

En Chloros, los índices se pueden aplicar durante el procesamiento del proyecto. Para determinar qué ajustes de índice y LUT desea aplicar a las exportaciones, lo más sencillo es utilizar el entorno de pruebas del visor de imágenes.

El entorno de pruebas le permite:

* **Aplicar nuevos índices y gradientes de color (LUT)** para visualizar los datos
* **Ajustar la configuración de visualización** de forma interactiva
* **Ver** imágenes de índice ya calculadas
* **Inspeccionar** los valores de píxeles en todos los niveles de zoom

### Abrir el entorno de pruebas

Se accede al entorno de pruebas de índices/LUT en la pestaña <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. Haga clic en una imagen de la cuadrícula de imágenes del explorador de archivos; se abrirá en la pestaña **Visor de imágenes** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> .
2. Haz clic en la pestaña ****Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> para abrir la barra lateral emergente de la izquierda si aún no está abierta

### Selección de una imagen a la que aplicar un índice/LUT

Para trabajar con un índice en el <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. **Abre una imagen** de la cuadrícula de imágenes principal haciendo clic en ella
2. Se abrirá entonces la pestaña **Visor de imágenes** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> se abrirá entonces
3. Haz clic en el **menú desplegable de capas** (arriba a la derecha del visor)
4. Selecciona la capa en el menú desplegable:
   * RAW (Reflectancia)

### Aplicar un índice a una imagen

Una vez que la imagen esté a pantalla completa y la barra lateral de la pestaña **Visor de imágenes** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. Marque la casilla Índice en la parte superior de la barra lateral
2. Elija el filtro de su cámara en el menú desplegable de la izquierda
3. Elija la fórmula de índice deseada en el menú desplegable de la derecha
4. Arrastre los círculos de color del canal del filtro a las ubicaciones correspondientes en la fórmula de índice que aparece a continuación
5. Una vez que la fórmula sea válida, la imagen se actualizará y mostrará los valores del índice
6. Mueva el cursor del ratón para ver los valores en la ubicación del cursor
7. Amplíe la imagen para ver píxeles individuales y sus valores asociados

Cada índice tiene un rango de valores y un significado específicos:

#### Ejemplo de NDVI

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

Para consultar la documentación completa sobre fórmulas de índices, véase [Fórmulas de índices multiespectrales](../project-settings/multispectral-index-formulas.md).

***

## Trabajar con LUT (tablas de consulta)

### ¿Qué es una LUT?

Una **tabla de consulta (LUT)** asigna valores numéricos de índice a colores para su visualización:

* **Entrada**: Valor de píxel del índice (p. ej., NDVI 0,65)
* **Salida**: color RGB (p. ej., verde brillante)
* **Finalidad**: facilitar la visualización e interpretación de los patrones**LUT en escala de grises frente a LUT en color:**

* Escala de grises: científica y neutra, muestra los datos sin procesar
* LUT en color: intuitiva y llamativa, resalta los patrones y las diferencias

{% hint style="success" %}
**Potencia de visualización**: aplicar una LUT de color a una imagen índice en escala de grises facilita enormemente la identificación de patrones, anomalías y áreas de interés de un solo vistazo.
{% endhint %}

### Aplicar una LUT a una imagen índice

Una vez que tenga una imagen índice que muestre

1. Haga clic en el <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> botón «+Añadir LUT»
2. Seleccione el degradado de color
3. Ajuste los puntos extremos mínimo y máximo del recorte
4. Ajuste el modo de recorte
5. Marque la casilla Índice en la barra lateral de la pestaña **Visor de imágenes** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> para aplicar la LUT

### Elección de un degradado de color

**Selección de un degradado:**

1. En el panel LUT, localice la**barra de degradado de color**

2. Pase el ratón por encima para ver los preajustes de degradado disponibles
3. Seleccione el degradado deseado
4. La imagen **se actualiza inmediatamente** con los nuevos colores cuando se marca la casilla Índice

{% hint style="success" %}
**Práctica recomendada**: Para índices de vegetación como NDVI, el degradado Red-Amarillo-Green es el más intuitivo, ya que se ajusta a las asociaciones de colores naturales (verde = sano, amarillo = moderado, rojo = estresado).
{% endhint %}

### Ajuste de clases de color

El **control Clases**determina cuántos pasos de color discretos aparecen en el degradado:**Opciones de número de clases:*** **2-5 clases**: Categorías muy amplias, zonas diferenciadas
* **6-10 clases**: Equilibrado, adecuado para la clasificación
* **11-20 clases**: Degradados suaves, aspecto continuo
* **Más de 20 clases**: Casi continuo, máxima suavidad**Cómo ajustar:**

1. En el panel LUT, localice los**cuadrados de muestras de color debajo de la barra de degradado**

2. Ajuste el número de clases añadiendo con el botón +
3. Elimine el número de clases haciendo doble clic en una muestra de color
4. El degradado se actualiza **en tiempo real** en la imagen**Efecto en la visualización:*** **Menos clases** (3-5): Crea zonas diferenciadas, clasificación simplificada, categorías más fáciles de distinguir
* **Número medio de clases** (6-10): Enfoque equilibrado, adecuado para la mayoría de aplicaciones
* **Más clases** (15-20): Transiciones suaves, variación detallada, aspecto fotográfico**Cuándo utilizarlo:*** **Pocas clases (3-5)**: diapositivas de presentación, mapas de clasificación, informes sencillos
* **Clases medias (6-10)**: análisis general, detalle equilibrado, informes estándar
* **Muchas clases (15-20)**: análisis científico, inspección detallada, resultados con calidad de publicación

### Ajuste fino de los rangos de valores

Los **controles de rango de valores**determinan qué valores de índice se asignan a qué colores en el degradado:**Controles de rango en el panel LUT:*** **Valor mínimo**: Límite inferior de la escala de colores
* **Valor máximo**: Límite superior de la escala de colores
* **Valores intermedios**: Distribuidos automáticamente entre el mínimo y el máximo (en función del número de clases)

#### Ajuste de los valores mínimo y máximo

**Para ajustar los rangos de valores:**

1. En el panel LUT, localice los campos de entrada**Valor mínimo**y**Valor máximo**

2. Haga clic en el campo**Valor mínimo**

3. Escriba el valor mínimo deseado (p. ej., `0.2`)
4. Pulse **Intro** o haga clic fuera del campo
5. Repita el proceso para el campo **Valor máximo** (p. ej., `0.9`)
6. La visualización **se actualiza inmediatamente**{% hint style="info" %}**Escalado automático**: Cuando se aplica una LUT por primera vez, Chloros establece automáticamente el mínimo y el máximo según el rango de datos real de la imagen. A continuación, puede reducir este rango para centrarse en rangos de valores específicos de interés.
{% endhint %}

**Ejemplo de ajustes de rango en NDVI:*** **Rango completo**: de `-1.0` a `1.0` (muestra todos los valores posibles)
* **Centrado en la vegetación**: de `0.2` a `0.9` (excluir suelo desnudo y agua)
* **Solo vegetación sana**: de `0.5` a `0.9` (resaltar solo las plantas vigorosas)
* **Detección de estrés**: de `0.2` a `0.5` (resaltar las zonas problemáticas)
* **Rango personalizado**: Ajústelo en función de los valores de píxeles observados**¿Por qué ajustar los rangos?*** **Aumentar el contraste** en su área de interés
* **Excluir valores irrelevantes** (p. ej., masas de agua, suelo desnudo)
* **Estandarizar la visualización** en múltiples imágenes o fechas
* **Resaltar diferencias sutiles** dentro de un rango de valores estrecho

### Recorte de valores fuera de rango

Cuando los valores de píxeles se encuentran fuera del rango mínimo/máximo definido, puede controlar cómo se muestran utilizando **modos de recorte**.

#### **Opciones de modos de recorte disponibles:**

#### 1. Mínimo y máximo

* Píxeles **por debajo del mínimo**→ se muestran utilizando el**primer color** del degradado (p. ej., rojo)
* Píxeles **por encima del máximo**→ se muestran utilizando el**último color** del degradado (p. ej., verde)
* **Caso de uso**: resaltar extremos, mostrar el rango completo de datos con colores saturados en los límites
* **Ejemplo**: los valores de NDVI por debajo de 0,2 aparecen todos en rojo, los valores por encima de 0,9 aparecen todos en verde

#### 2. Fondo transparente

* Los píxeles **fuera del rango**se vuelven**totalmente transparentes*** Solo los píxeles **dentro del rango** muestran el degradado de color
* **Caso de uso**: superposición GIS, aislar rangos de valores específicos, resaltar solo las áreas de interés
* **Ejemplo**: Mostrar solo NDVI 0,4-0,7 en color, todo lo demás transparente

{% hint style="warning" %}
**Limitación de transparencia**: Los píxeles transparentes aparecerán con el color de fondo en el visor. Al exportar durante el procesamiento, la transparencia se conserva en formato PNG, pero no en JPG.
{% endhint %}

#### 3. Fondo de índice

* Los píxeles **fuera del rango**se muestran en**escala de grises** (mostrando los valores de índice sin procesar)
* Los píxeles **dentro del rango**muestran un**gradiente de color*** **Caso de uso**: resaltado sutil, mantener el contexto mientras se enfatizan las áreas de interés
* **Ejemplo**: Resaltar en color la vegetación estresada (NDVI 0,3-0,5) mientras se muestran las áreas sanas en gris

#### 4. Fondo original

* Los píxeles **fuera del rango**muestran la**imagen multiespectral original*** Los píxeles **dentro del rango**muestran un**gradiente de color*** **Caso de uso**: El más intuitivo: combina el contexto natural de la imagen con una superposición de colores analítica
* **Ejemplo**: Vea el aspecto real del campo o cultivo con las áreas de estrés superpuestas y codificadas por colores

### Elección del modo de recorte adecuado

| Modo de recorte              | Ideal para                                   | Estilo de visualización          |
| -------------------------- | ------------------------------------------ | ---------------------------- |
| **Mínimo y máximo**    | Visualización completa de datos, análisis científico     | Todos los píxeles coloreados           |
| **Fondo transparente** | Superposiciones SIG, aislamiento de rangos específicos    | Color en el rango, en blanco fuera de él |
| **Fondo índice**       | Énfasis sutil, manteniendo el contexto de los datos  | Color en el rango, gris fuera de él  |
| **Fondo original**    | Informes, presentaciones, análisis intuitivo | Color en el rango, foto fuera de él |

### Creación de colores LUT personalizados

Para tener un control total sobre tu visualización, puedes crear **gradientes de color personalizados** editando los puntos de color individuales.**Para crear un degradado personalizado:**

1. En el panel LUT, localiza la**barra de vista previa del degradado**

2. Busca los**cuadrados de muestras de color** debajo del degradado
3. **Haz clic en un punto de color** para seleccionarlo
4. Se abrirá un **selector de color**

5. Elige un nuevo color utilizando:
   * **Rueda de colores**: selección visual del color
   * **Deslizadores RGB/HSV**: control preciso del color
   * **Introducción de código hexadecimal**: especificación exacta del color (p. ej., `#FF0000` para el rojo)
6. Haz clic fuera del selector de color **para aplicar el nuevo color**

7. El degradado**se actualiza inmediatamente** en la imagen**Añadir o eliminar paradas de color:*** **Añadir una parada**: Haz clic en el icono + para añadir una nueva muestra al final
* **Eliminar una parada**: Haz doble clic en el cuadrado de color para eliminar la muestra**Estrategias de personalización:*** **Invertir degradado**: invierte el orden de los colores para cambiar el significado (p. ej., verde = bajo, rojo = alto)
* **Colores de marca**: adapta la paleta de colores de tu organización para los informes
* **Apto para daltónicos**: utiliza combinaciones de naranja-azul o morado-amarillo
* **Optimización de la impresión**: elija colores que funcionen tanto en impresión a color como en escala de grises
* **Umbrales múltiples**: utilice colores distintos en umbrales de valores específicos para la clasificación

{% hint style="info" %}
**Guardar degradados personalizados**: los degradados personalizados se pueden guardar y reutilizar. Haga clic en el icono de guardar del panel LUT para conservar sus esquemas de color personalizados para su uso futuro.
{% endhint %}

***

## Flujo de trabajo interactivo

### Actualizaciones en tiempo real

Todos los ajustes de LUT en el entorno de pruebas actualizan la imagen **al instante y de forma interactiva**:

* **Cambiar capa** → La imagen cambia inmediatamente
* **Seleccionar degradado** → Los colores se actualizan al instante
* **Ajustar rango de valores** → El contraste cambia en tiempo real
* **Cambiar clases** → La suavidad del degradado se actualiza inmediatamente
* **Modificar recorte** → La visualización del fondo cambia al instante
* **Editar colores** → El degradado personalizado se aplica inmediatamente**No se necesita el botón «Aplicar»**: ¡todos los cambios son en directo e interactivos!

{% hint style="success" %}
**Respuesta en tiempo real**: La respuesta visual instantánea le permite experimentar rápidamente con diferentes ajustes hasta encontrar la visualización óptima para sus necesidades de análisis.
{% endhint %}

### Flujo de trabajo de refinamiento iterativo

**Flujo de trabajo típico de optimización de LUT:**

1.**Seleccionar capa de índice** (p. ej., RAW (Reflectancia))
2. **Aplicar índice**: elija el filtro de la cámara y la fórmula del índice; arrastre los círculos de color a la ubicación adecuada en la fórmula del índice
3. **Aplicar gradiente de LUT**: comience con el preajuste Red-Amarillo-Green
4. **Inspeccionar los valores de píxel**: mover el cursor por la imagen y tomar nota de los rangos de valores
5. **Ajustar mínimo/máximo**: reducir el rango para centrarse en la vegetación (p. ej., de 0,2 a 0,9)
6. **Elegir recorte**: probar «Fondo original» para mantener el contexto
7. **Perfeccionar los colores**: personalizar el degradado si es necesario para dar énfasis a elementos específicos
8. **Finalizar la configuración**: documenta la configuración y cópiala en la configuración del proyecto para el procesamiento de la exportación

### Inspección de los valores de píxel

Comprender los valores reales de los píxeles es crucial para establecer rangos de LUT eficaces:

**Cómo inspeccionar los valores:**

1. Los valores de píxel se muestran cuando la imagen tiene**marcadas** las casillas de Índice, o tanto de Índice como de LUT.
2. **Mueva el cursor** sobre diferentes áreas de la imagen
3. **Observe los valores de píxeles** que se muestran en la leyenda al pasar el cursor
4. Amplíe la imagen para ver píxeles individuales resaltados con un valor flotante
5. **Tome nota** de los rangos de valores para las diferentes características:
   * **Vegetación sana**: p. ej., NDVI 0,55-0,85
   * **Vegetación estresada**: p. ej., NDVI 0,30-0,50
   * **Suelo desnudo**: p. ej., NDVI 0,05-0,25
   * **Agua** (si está presente): p. ej., NDVI -0,05 a 0,10**Uso de valores de píxeles para establecer los rangos de la LUT:**Tras examinar los valores de los píxeles, ajuste los valores mínimo y máximo de la LUT en consecuencia:**Ejemplo de escenario:*** **Observación**: Valores del suelo = 0,05-0,25, Estrés = 0,25-0,50, Saludable = 0,50-0,85
* **Objetivo**: Visualizar solo la salud de las plantas (excluir el suelo)
* **Configuración de la LUT**: Mín. = `0.25`, Máx. = `0.85`
* **Recorte**: «Fondo original» para ver el suelo en su color natural
* **Resultado**: El degradado de color solo se aplica a la vegetación; el suelo se muestra como en la imagen original

{% hint style="info" %}
**Rango dinámico**: Los diferentes cultivos, estaciones y etapas de crecimiento tendrán rangos de valores distintos. Compruebe siempre los valores de píxeles en su conjunto de datos específico antes de establecer los rangos de LUT.
{% endhint %}

***

## Índices personalizados (Chloros+)

### Creación de fórmulas de índices personalizados

{% hint style="info" %}
**Dónde crearlos**: Los índices personalizados se pueden configurar en**Configuración del proyecto** antes del procesamiento, así como en la barra lateral del entorno de pruebas del visor de imágenes.
{% endhint %}

**Para crear un índice personalizado:**

1.**Abre la configuración del proyecto** (antes del procesamiento) o la barra lateral del entorno de pruebas del visor de imágenes
2. Ve al **menú desplegable de fórmulas de índice**

3. Busca la opción**«Personalizado»** (debes haber iniciado sesión con una licencia Chloros+)
4. **Define tu fórmula** utilizando variables de banda:
   * Nombres de banda: `NIR`, `Red`, `Green`, `Blue`, `RedEdge`, etc.
   * Operadores: `+`, `-`, `*`, `/`, `^` (exponente)
   * Funciones: `sqrt()`, `abs()`, etc. (si son compatibles)
   * Paréntesis: `()` para el orden de las operaciones
5. **Asigne un nombre a su índice** (por ejemplo, «MyIndex» o «CustomNDVI»)
6. **Guarde la configuración**

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

{% hint style="warning" %}
**Validación de fórmulas**: Asegúrese de que su fórmula utilice bandas disponibles en su cámara. Por ejemplo, RedEdge solo está disponible en cámaras con un filtro RedEdge.
{% endhint %}

***

## Próximos pasos

Ahora que ya conoce el entorno de pruebas de índices/LUT:

* **Aplicar al procesamiento**: Utilice los ajustes descubiertos en [Configuración del proyecto](../project-settings/project-settings.md)
* **Procesamiento por lotes**: Aplique índices optimizados a conjuntos de datos completos
* **Más información**: Lea [Fórmulas de índices multiespectrales](../project-settings/multispectral-index-formulas.md)

Documentación relacionada:

* [**Capas de imagen**](image-layers.md) - Gestión y visualización de capas
* [**Abrir una imagen a pantalla completa**](opening-an-image-full-screen.md) - Conceptos básicos del visor de imágenes
* [**Procesamiento de imágenes (GUI)**](../processing-images-gui/adding-files-to-a-project.md) - Flujo de trabajo de procesamiento completo

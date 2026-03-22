# Selección de imágenes de referencia

Marcar las imágenes que contienen objetivos de calibración es un paso crucial que acelera considerablemente el proceso de Chloros. Al preseleccionar las imágenes de referencia, se evita que Chloros tenga que analizar todas las imágenes del conjunto de datos en busca de objetivos de calibración.

## ¿Por qué marcar las imágenes de referencia?

### Velocidad de procesamiento

Sin marcar las imágenes objetivo, Chloros debe:

* Analizar cada una de las imágenes de su proyecto
* Ejecutar algoritmos de detección de objetivos en cada imagen
* Comprobar cientos o miles de imágenes innecesariamente

**Resultado**: El procesamiento puede tardar mucho más tiempo, especialmente en el caso de conjuntos de datos grandes.

### Con imágenes objetivo marcadas

Cuando marca la columna «Objetivo» para imágenes específicas:

* Chloros solo escanea las imágenes marcadas en busca de objetivos
* La detección de objetivos se completa mucho más rápido
* El tiempo total de procesamiento se reduce considerablemente

{% hint style="success" %}
**Mejora de la velocidad**: Marcar 2-3 imágenes objetivo en un conjunto de datos de 500 imágenes puede reducir el tiempo de detección de objetivos de más de 30 minutos a menos de 1 minuto.
{% endhint %}

***

## Cómo marcar imágenes de objetivos

### Paso 1: Identifique sus imágenes de objetivos

Revise las imágenes importadas en el explorador de archivos e identifique cuáles contienen objetivos de calibración.

**Situaciones habituales:*** **Objetivo previo a la captura**: Capturado antes de iniciar la sesión
* **Objetivo posterior a la captura**: Capturado tras completar la sesión
* **Objetivos en el campo**: Objetivos colocados dentro del área de captura
* **Objetivos múltiples**: 2-3 imágenes objetivo por sesión (recomendado)

### Paso 2: Compruebe la columna «Target»

Para cada imagen que contenga un objetivo de calibración:

1. Localice la imagen en la tabla del explorador de archivos
2. Busque la columna **Target** (la columna situada más a la derecha)
3. Haga clic en la casilla de verificación de la columna «Target» correspondiente a esa imagen
4. Repita el proceso para todas las imágenes que contengan objetivos

### Paso 3: Verifique su selección

Antes de procesar, compruebe lo siguiente:

* [ ] Todas las imágenes con objetivos de calibración están marcadas
* [ ] No hay imágenes que no sean de objetivos marcadas accidentalmente
* [ ] Los objetivos son claramente visibles en las imágenes marcadas

***

## Prácticas recomendadas para imágenes de objetivos

### Directrices para la captura de objetivos

**Momento:**

* Captura las imágenes de objetivos inmediatamente antes y durante toda la sesión de captura
* En las mismas condiciones de iluminación que el sensor de luz DAQ
* Lo ideal es capturar imágenes de objetivos con la mayor frecuencia posible para obtener los mejores resultados. De lo contrario, se utilizarán los datos del sensor de luz para ajustar la calibración a lo largo del tiempo.

**Posición de la cámara:**

* Sostenga la cámara por encima del objetivo de manera que quede centrado y ocupe entre el 40 % y el 60 % del centro de la imagen.
* Mantenga la cámara paralela o en posición nadir con respecto a la superficie del objetivo

**Iluminación:**

* La misma iluminación ambiental que la de su sensor de luz DAQ
* Evite las sombras en las superficies de los objetivos
* No bloquee la fuente de luz con su cuerpo, vehículo o vegetación
* Las condiciones de cielo nublado proporcionan los resultados más consistentes

**Condición del objetivo:**

* Mantenga los paneles de los objetivos limpios y secos
* Los 4 paneles deben ser claramente visibles y estar libres de obstrucciones
* Coloque los objetivos perpendiculares o en el nadir respecto a la fuente de luz, si es posible

### ¿Cuántas imágenes de los objetivos?

**Mínimo:**1 imagen del objetivo por sesión.**Recomendado:** 3-5 imágenes del objetivo por sesión.**Programa de buenas prácticas:**

* Captura de 3-5 imágenes poco después de que el sensor de luz comience a grabar
* Gire la cámara entre capturas para obtener los mejores resultados
* Opcional: periódicamente a mitad de la sesión si las condiciones de iluminación cambian constantemente

***

## Trabajar con varias cámaras

### Configuraciones con dos cámaras

Si se utilizan dos cámaras MAPIR simultáneamente (por ejemplo, Survey3W RGN + Survey3N OCN):

1. Capture las imágenes de referencia con **ambas cámaras** al mismo tiempo
2. Utilice el **mismo objetivo físico** para ambas cámaras
3. Marque las imágenes de referencia para **ambos tipos de cámara** en el explorador de archivos
4. Chloros utilizará los objetivos adecuados para la calibración de cada cámara

### Columna «Modelo de cámara»

La columna **«Modelo de cámara»** ayuda a identificar qué imágenes proceden de cada cámara:

* Survey3W\_RGN
* Survey3N\_OCN
* Survey3W\_RGB
* etc.

Utilice esta columna para verificar que ha marcado objetivos para cada tipo de cámara en su proyecto.

***

## Ajustes de detección de objetivos

### Ajuste de la sensibilidad de detección

Si Chloros no detecta sus objetivos correctamente, ajuste estos parámetros en [Ajustes del proyecto](adjusting-project-settings.md):**Área mínima de muestra de calibración:*** **Predeterminado**: 25 píxeles
* **Aumente** si se producen detecciones erróneas en artefactos pequeños
* **Disminuya** si no se detectan los objetivos**Agrupación mínima de objetivos:*** **Predeterminado**: 60
* **Aumente** si los objetivos se dividen en múltiples detecciones
* **Reduzca** si los objetivos con variaciones de color no se detectan por completo***

## Problemas comunes con las imágenes de los objetivos

### Problema: No se detectan objetivos

**Posibles causas:**

* Las imágenes de los objetivos no están marcadas en el explorador de archivos
* El objetivo es demasiado pequeño en el fotograma (&lt; 30 % de la imagen)
* Iluminación deficiente (sombras, reflejos)
* Ajustes de detección de objetivos demasiado estrictos

**Soluciones:**

1. Compruebe que la columna «Objetivo» esté marcada para las imágenes correctas
2. Revise la calidad de la imagen del objetivo en la vista previa
3. Vuelva a capturar los objetivos si la calidad es deficiente
4. Ajuste la configuración de detección de objetivos si es necesario

### Problema: Detecciones de objetivos falsas

**Posibles causas:**

* Edificios, vehículos o cobertura del suelo de color blanco confundidos con objetivos
* Parches brillantes en la vegetación
* Sensibilidad de detección demasiado baja

**Soluciones:**

1. Marque solo las imágenes de objetivos reales para limitar el alcance de la detección
2. Aumente el área mínima de la muestra de calibración
3. Aumente el valor mínimo de agrupamiento de objetivos
4. Asegúrese de que las imágenes de objetivos muestren solo el objetivo (mínimo ruido de fondo)

***

## Lista de verificación

Antes de iniciar el procesamiento, verifique su selección de imágenes de objetivos:

* [ ] Al menos 1 imagen de objetivo marcada por sesión
* [ ] Las casillas de la columna «Objetivo» están marcadas para todas las imágenes de objetivos
* [ ] Imágenes de objetivos capturadas en el mismo intervalo de tiempo que el estudio
* [ ] Objetivos claramente visibles en la vista previa al hacer clic
* [ ] Los 4 paneles de calibración visibles en cada imagen de objetivo
* [ ] Sin sombras ni obstrucciones en los objetivos
* [ ] Para cámaras duales: Objetivos marcados para ambos tipos de cámara

***

## Procesamiento sin objetivos

### Procesamiento sin objetivos de calibración

Aunque no se recomienda para trabajos científicos, puede procesar sin objetivos:

1. Deje sin marcar todas las casillas de la columna «Objetivo»
2. **Desactive** «Calibración de reflectancia» en la configuración del proyecto
3. Se seguirá aplicando la corrección de viñeteado
4. El resultado no se calibrará para la reflectancia absoluta

{% hint style="warning" %}
**No recomendado**: Sin calibración de reflectancia, los valores de píxeles representan únicamente el brillo relativo, no mediciones científicas de reflectancia. Utilice objetivos de calibración para obtener resultados precisos y repetibles.
{% endhint %}

***

## Próximos pasos

Una vez que haya marcado sus imágenes objetivo:

1. **Revise su configuración** - Consulte [Ajustar la configuración del proyecto](adjusting-project-settings.md)
2. **Inicie el procesamiento** - Consulte [Iniciar el procesamiento](starting-the-processing.md)
3. **Supervise el progreso**: consulte [Supervisar el procesamiento](monitoring-the-processing.md)

Para obtener más información sobre los objetivos de calibración, consulte [Objetivos de calibración](../calibration-targets.md).

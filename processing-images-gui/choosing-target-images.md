# Selección de imágenes objetivo

Marcar qué imágenes contienen objetivos de calibración es un paso crucial que acelera significativamente el proceso de Chloros. Al preseleccionar las imágenes objetivo, se elimina la necesidad de que Chloros escanee todas las imágenes del conjunto de datos en busca de objetivos de calibración.

## ¿Por qué marcar las imágenes objetivo?

### Velocidad de procesamiento

Sin marcar las imágenes objetivo, Chloros debe:

* Escanear todas y cada una de las imágenes de su proyecto.
* Ejecutar algoritmos de detección de objetivos en cada imagen.
* Comprobar cientos o miles de imágenes innecesariamente.

**Resultado**: el procesamiento puede tardar mucho más tiempo, especialmente en el caso de conjuntos de datos grandes.

### Con imágenes objetivo marcadas

Cuando marca la columna Objetivo para imágenes específicas:

* Chloros solo escanea las imágenes marcadas en busca de objetivos.
* La detección de objetivos se completa mucho más rápido.
* El tiempo total de procesamiento se reduce considerablemente.

{% hint style=&quot;success&quot; %}
**Mejora de la velocidad**: marcar 2-3 imágenes objetivo en un conjunto de datos de 500 imágenes puede reducir el tiempo de detección de objetivos de más de 30 minutos a menos de 1 minuto.
{% endhint %}

***

## Cómo marcar imágenes objetivo

### Paso 1: Identifique sus imágenes objetivo

Revise las imágenes importadas en el explorador de archivos e identifique cuáles contienen objetivos de calibración.

**Escenarios comunes:**

* **Objetivo previo a la captura**: capturado antes de iniciar la sesión.
* **Objetivo posterior a la captura**: capturado después de completar la sesión.
* **Objetivos en el campo**: objetivos colocados dentro del área de captura.
* **Objetivos múltiples**: 2-3 imágenes objetivo por sesión (recomendado).

### Paso 2: Compruebe la columna Objetivo.

Para cada imagen que contenga un objetivo de calibración:

1. Localice la imagen en la tabla del explorador de archivos.
2. Busque la columna **Objetivo** (la columna situada más a la derecha).
3. Haga clic en la casilla de verificación de la columna Objetivo para esa imagen.
4. Repita el proceso para todas las imágenes que contengan objetivos.

### Paso 3: Verifique su selección.

Antes de procesar, compruebe lo siguiente:

* [ ] Todas las imágenes con objetivos de calibración están marcadas.
* [ ] No hay imágenes que no sean objetivos marcadas accidentalmente.
* [ ] Los objetivos son claramente visibles en las imágenes marcadas.

***

## Mejores prácticas para imágenes objetivo

### Directrices para la captura de objetivos

**Momento:**

* Capture imágenes objetivo inmediatamente antes y durante la sesión de captura.
* En las mismas condiciones de iluminación que el sensor de luz DAQ.
* Lo ideal es capturar imágenes de objetivos con la mayor frecuencia posible para obtener los mejores resultados. De lo contrario, se utilizarán los datos del sensor de luz para ajustar la calibración a lo largo del tiempo.

**Posición de la cámara:**

* Sostenga la cámara por encima del objetivo de manera que quede centrada y ocupe entre el 40 y el 60 % del centro de la imagen.
* Mantenga la cámara paralela/nadir a la superficie del objetivo.

**Iluminación:**

* La misma iluminación ambiental que su sensor de luz DAQ.
* Evite las sombras en las superficies objetivo.
* No bloquee la fuente de luz con su cuerpo, vehículo o vegetación.
* Las condiciones nubladas proporcionan los resultados más consistentes.

**Condición del objetivo:**

* Mantenga los paneles objetivo limpios y secos.
* Los 4 paneles deben ser claramente visibles y estar sin obstrucciones.
* Los objetivos deben estar perpendiculares/nadir a la fuente de luz, si es posible.

### ¿Cuántas imágenes objetivo?

**Mínimo:** 1 imagen objetivo por sesión. **Recomendado:** 3-5 imágenes objetivo por sesión.

**Calendario de buenas prácticas:**

* Capture entre 3 y 5 imágenes poco después de que el sensor de luz comience a grabar.
* Gire la cámara entre capturas para obtener los mejores resultados.
* Opcional: periódicamente a mitad de la sesión si las condiciones de iluminación cambian constantemente.

***

## Trabajar con varias cámaras

### Configuraciones de doble cámara

Si utiliza dos cámaras MAPIR simultáneamente (por ejemplo, Survey3W RGN + Survey3N OCN):

1. Capture imágenes objetivo con **ambas cámaras** al mismo tiempo.
2. Utilice el **mismo objetivo físico** para ambas cámaras.
3. Marque las imágenes objetivo para **ambos tipos de cámara** en el explorador de archivos.
4. Chloros utilizará los objetivos adecuados para la calibración de cada cámara.

### Columna Modelo de cámara

La columna **Modelo de cámara** ayuda a identificar qué imágenes provienen de qué cámara:

* Survey3W\_RGN
* Survey3N\_OCN
* Survey3W\_RGB
* etc.

Utilice esta columna para verificar que ha marcado los objetivos para cada tipo de cámara en su proyecto.

***

## Ajustes de detección de objetivos

### Ajustar la sensibilidad de detección

Si Chloros no detecta correctamente sus objetivos, ajuste estos ajustes en [Ajustes del proyecto](adjusting-project-settings.md):

**Área mínima de muestra de calibración:**

* **Predeterminado**: 25 píxeles
* **Aumente** si obtiene detecciones falsas en artefactos pequeños
* **Disminuya** si no se detectan los objetivos

**Agrupación mínima de objetivos:**

* **Predeterminado**: 60
* **Aumente** si los objetivos se dividen en múltiples detecciones.
* **Disminuya** si los objetivos con variación de color no se detectan completamente.

***

## Problemas comunes con las imágenes de los objetivos

### Problema: No se detectan objetivos.

**Posibles causas:**

* Las imágenes de los objetivos no están marcadas en el explorador de archivos.
* El objetivo es demasiado pequeño en el marco (&lt; 30 % de la imagen).
* Iluminación deficiente (sombras, reflejos)
* Ajustes de detección de objetivos demasiado estrictos

**Soluciones:**

1. Compruebe que la columna Objetivo está marcada para las imágenes correctas
2. Revise la calidad de la imagen del objetivo en la vista previa
3. Vuelva a capturar los objetivos si la calidad es deficiente
4. Ajuste la configuración de detección de objetivos si es necesario

### Problema: Detecciones de objetivos falsos

**Posibles causas:**

* Edificios blancos, vehículos o cubierta vegetal confundidos con objetivos.
* Manchas brillantes en la vegetación.
* Sensibilidad de detección demasiado baja.

**Soluciones:**

1. Marque solo las imágenes de objetivos reales para limitar el alcance de la detección.
2. Aumente el área mínima de muestra de calibración.
3. Aumente el valor mínimo de agrupación de objetivos.
4. Asegúrese de que las imágenes de objetivos solo muestren el objetivo (mínimo ruido de fondo).

***

## Lista de verificación

Antes de comenzar el procesamiento, verifique la selección de imágenes objetivo:

* [ ] Al menos 1 imagen objetivo marcada por sesión.
* [ ] Las casillas de verificación de la columna Objetivo están marcadas para todas las imágenes objetivo.
* [ ] Imágenes objetivo capturadas en el mismo intervalo de tiempo que el estudio.
* [ ] Objetivos claramente visibles en la vista previa al hacer clic.
* [ ] Los 4 paneles de calibración visibles en cada imagen objetivo.
* [ ] No hay sombras ni obstrucciones en los objetivos.
* [ ] Para cámaras duales: objetivos marcados para ambos tipos de cámara.

***

## Procesamiento sin objetivos

### Procesamiento sin objetivos de calibración

Aunque no se recomienda para trabajos científicos, puede procesar sin objetivos:

1. Deje todas las casillas de verificación de la columna Objetivo sin marcar.
2. **Desactive** la «Calibración de reflectancia» en la configuración del proyecto.
3. La corrección de viñeta se seguirá aplicando.
4. La salida no se calibrará para la reflectancia absoluta.

{% hint style=&quot;warning&quot; %}
**No recomendado**: sin la calibración de reflectancia, los valores de píxeles representan solo el brillo relativo, no mediciones científicas de reflectancia. Utilice objetivos de calibración para obtener resultados precisos y repetibles.
{% endhint %}

***

## Próximos pasos

Una vez que haya marcado las imágenes objetivo:

1. **Revise la configuración**: consulte [Ajustar la configuración del proyecto](adjusting-project-settings.md)
2. **Inicie el procesamiento** - Consulte [Inicio del procesamiento](starting-the-processing.md)
3. **Supervise el progreso** - Consulte [Supervisión del procesamiento](monitoring-the-processing.md)

Para obtener más información sobre los objetivos de calibración, consulte [Objetivos de calibración](../calibration-targets.md).

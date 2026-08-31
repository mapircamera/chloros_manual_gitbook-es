# Selección de imágenes de referencia

Al marcar qué imágenes contienen objetivos de calibración, se indica a Chloros exactamente dónde buscarlos. Cuando hay al menos una imagen marcada en la columna «Target», Chloros analiza **solo las imágenes marcadas**; por lo tanto, marcar los objetivos sirve tanto para acelerar el procesamiento como para evitar que las imágenes de levantamiento topográfico se confundan con un objetivo.

<figure><img src="../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

## ¿Por qué marcar las imágenes objetivo?

### El marcado controla el escaneo

Cuando marcas la columna «Objetivo» para imágenes específicas:

* Chloros escanea solo las imágenes marcadas en busca de objetivos
* La detección de objetivos se completa mucho más rápido
* Las imágenes de levantamiento topográfico no pueden generar detecciones erróneas de objetivos

Si **no** hay ninguna imagen marcada, Chloros recurre a escanear todas las imágenes del proyecto:

* Los algoritmos de detección de objetivos se ejecutan en cada imagen
* Se revisan innecesariamente cientos o miles de imágenes
* El procesamiento tarda mucho más, especialmente en conjuntos de datos grandes

{% hint style="success" %}
**Mejora de la velocidad**: Marcar 2-3 imágenes de objetivos en un conjunto de datos de 500 imágenes puede reducir el tiempo de detección de objetivos de más de 30 minutos a menos de 1 minuto.
{% endhint %}

***

## Cómo marcar las imágenes objetivo

### Paso 1: Identifica tus imágenes objetivo

Revisa las imágenes importadas en el explorador de archivos e identifica cuáles contienen objetivos de calibración.

**Situaciones habituales:*** **Objetivo previo a la captura**: capturado antes de iniciar la sesión
* **Objetivo posterior a la captura**: capturado tras completar la sesión
* **Objetivos en el terreno**: objetivos colocados dentro del área de captura
* **Múltiples objetivos**: 2-3 imágenes de objetivos por sesión (recomendado)

### Paso 2: Marca la columna «Target» <img src="../.gitbook/assets/image (33).png" alt="" data-size="original">

Para cada imagen que contenga un objetivo de calibración:

1. Localiza la imagen en la tabla del explorador de archivos
2. Busca la columna **Target** (la columna más a la derecha)
3. Marca la casilla de la columna «Target» correspondiente a esa imagen
4. Repite el proceso con todas las imágenes que contengan objetivos

### Paso 3: Verifica tu selección

Antes de procesar, comprueba lo siguiente:

* [ ] Todas las imágenes con objetivos de calibración están marcadas
* [ ] No hay ninguna imagen que no sea un objetivo marcada por error
* [ ] Los objetivos son claramente visibles en las imágenes marcadas

***

## LATTICE: Las marcas de calibración son opcionales cuando un DAQ está grabando

En el caso de las cámaras multiespectrales LATTICE, una marca de calibración dentro del fotograma es **una de las dos** referencias de reflectancia posibles:

* **Objetivo dentro del encuadre**: cuando una imagen de objetivo marcada supera los controles de calidad (QA) de Chloros, el objetivo se convierte en la**referencia de reflectancia absoluta** para las imágenes que lo rodean.
* **Irradiancia descendente del DAQ**: cuando no hay ningún objetivo presente (o falla el control de calidad), Chloros calcula la reflectancia a partir de la irradiancia descendente del sensor de luz del DAQ (ρ = π·L/E). Si una grabación con `.daq` o DAQ-M `.csv` cubre tus capturas, obtendrás una reflectancia calibrada**sin necesidad de imágenes de referencia alguna**.

Este comportamiento automático es el predeterminado. En el archivo CLI / SDK, corresponde a `--reflectance-source auto`; también puedes forzar `target` (estricto — sin sustitución del DAQ) o `daq` (con prioridad del DAQ). Consulte la [Referencia de CLI](../reference/cli-reference.md#per-product-export-toggles-lattice-multispectral).

**Geometrías de objetivos LATTICE**: además de la detección clásica de paneles utilizada para Survey3, el procesamiento LATTICE admite**objetivos marcados con ArUco**,**objetivos con ROI fija**y**objetivos en forma de tira**, configurados por proyecto. Se pueden proporcionar escaneos de reflectancia**medida** de los objetivos por unidad mediante un número de serie (CLI: `--target-reflectance-dir`, uno `<serial>.csv` por unidad de objetivo), con los espectros nominales T3/T4P como alternativa.

{% hint style="info" %}
**Módulo F988**: la reflectancia del F988 se calibra mediante un panel de reflectancia en la escena: la banda se encuentra fuera del rango calibrado del sensor de luz del DAQ, por lo que Chloros aplica su captura de panel más reciente y la mantiene entre las lecturas del panel. Si un módulo F988 se procesa únicamente con DAQ, Chloros rechaza la reflectancia basada en DAQ para esa banda (motivo de omisión `dls-uncalibrated-band-988`); el flujo de trabajo con el panel es la vía admitida.
{% endhint %}

***

## Prácticas recomendadas para las imágenes del objetivo

### Directrices para la captura del objetivo

**Momento:**

* Captura las imágenes del objetivo inmediatamente antes y durante toda la sesión de captura
* En las mismas condiciones de iluminación que el sensor de luz de DAQ
* Lo ideal es capturar imágenes del objetivo con la mayor frecuencia posible para obtener los mejores resultados. De lo contrario, se utilizarán los datos del sensor de luz para ajustar la calibración a lo largo del tiempo.

**Posición de la cámara:**

* Mantén la cámara por encima del objetivo de forma que este quede centrado y ocupe entre el 40 % y el 60 % del centro de la imagen.
* Mantén la cámara paralela o en posición nadir respecto a la superficie del objetivo

**Iluminación:**

* La misma iluminación ambiental que la de tu sensor de luz DAQ
* Evita las sombras en las superficies del objetivo
* No bloquees la fuente de luz con tu cuerpo, un vehículo o la vegetación
* Las condiciones de cielo nublado proporcionan los resultados más uniformes

**Estado del objetivo:**

* Mantén los paneles del objetivo limpios y secos.
* Todos los paneles del objetivo (por ejemplo, los 4 de un T4) deben estar claramente visibles y sin obstrucciones.
* Si es posible, coloca los objetivos en posición perpendicular o nadir respecto a la fuente de luz.

### ¿Cuántas imágenes del objetivo?

**Mínimo:**1 imagen del objetivo por sesión.**Recomendado:** 3-5 imágenes del objetivo por sesión.**Programación recomendada:**

* Captura de 3-5 imágenes poco después de que el sensor de luz comience a grabar
* Gira la cámara entre capturas para obtener los mejores resultados
* Opcional: periódicamente a mitad de la sesión si las condiciones de iluminación cambian constantemente

***

## Trabajar con varias cámaras

### Configuraciones con dos cámaras

Si se utilizan dos cámaras «MAPIR» simultáneamente (por ejemplo, Survey3W RGN + Survey3N OCN):

1. Captura imágenes del objetivo con **ambas cámaras** al mismo tiempo
2. Utiliza el **mismo objetivo físico** para ambas cámaras
3. Marca las imágenes de los objetivos para **ambos tipos de cámara** en el explorador de archivos
4. Chloros utilizará los objetivos adecuados para la calibración de cada cámara

### Columna «Modelo de cámara»

La columna **«Modelo de cámara»** ayuda a identificar qué imágenes proceden de cada cámara:

* Survey3W\_RGN
* Survey3N\_OCN
* LATT-M3M-L41-F550
* LATT-M3C-L87-FRGN
* etc.

Utiliza esta columna para comprobar que has marcado los objetivos para cada tipo de cámara en tu proyecto.

***

## Ajustes de detección de objetivos

### Ajuste de la sensibilidad de detección

Si Chloros no detecta correctamente tus objetivos, ajusta estos parámetros en [Configuración del proyecto](adjusting-project-settings.md):**Área mínima de muestra de calibración (px):*** **Por defecto**: 25 píxeles
* **Aumenta** este valor si se producen detecciones erróneas en artefactos pequeños
* **Disminuye** este valor si no se detectan los objetivos**Agrupación mínima de objetivos (0-100):*** **Por defecto**: 60
* **Aumenta** si los objetivos se dividen en varias detecciones
* **Disminuye** si los objetivos con variaciones de color no se detectan por completo

{% hint style="info" %}
**Consejo de CLI**: `chloros-cli process` admite los mismos parámetros (`--min-target-size`, `--target-clustering`), y su indicador `--target`/`--targets` marca toda una carpeta de entrada como «solo para el panel de objetivos». Consulte la [Referencia de CLI](../reference/cli-reference.md).
{% endhint %}

***

## Problemas comunes con las imágenes objetivo

### Problema: No se detectan objetivos

**Posibles causas:**

* Las imágenes de los objetivos no están marcadas en el explorador de archivos
* El objetivo es demasiado pequeño en el encuadre (&lt; 30 % de la imagen)
* Iluminación deficiente (sombras, reflejos)
* Ajustes de detección de objetivos demasiado estrictos

**Soluciones:**

1. Comprueba que la columna «Objetivo» esté marcada en las imágenes correctas
2. Revisa la calidad de la imagen del objetivo en la vista previa
3. Vuelve a capturar los objetivos si la calidad es deficiente
4. Ajusta la configuración de detección de objetivos si es necesario

### Problema: Detecciones erróneas de objetivos

**Posibles causas:**

* Edificios, vehículos o cobertura del suelo de color blanco confundidos con objetivos
* Zonas brillantes en la vegetación
* Sensibilidad de detección demasiado baja

**Soluciones:**

1. Marca solo las imágenes de objetivos reales: solo se escanean las imágenes marcadas
2. Aumenta el área mínima de la muestra de calibración
3. Aumenta el valor mínimo de agrupamiento de objetivos
4. Asegúrate de que las imágenes de objetivos muestren únicamente el objetivo (con el mínimo de interferencias de fondo)

***

## Lista de comprobación

Antes de iniciar el procesamiento, comprueba tu selección de imágenes de objetivo:

* [ ] Al menos una imagen de objetivo marcada por sesión (o, en el caso de LATTICE, una grabación `.daq`/`.csv` que cubra la sesión)
* [ ] Las casillas de la columna «Objetivo» están marcadas para todas las imágenes de objetivo
* [ ] Las imágenes de objetivo se han capturado en el mismo intervalo de tiempo que el estudio
* [ ] Los objetivos son claramente visibles en la vista previa al hacer clic sobre ellos
* [ ] Todos los paneles de calibración son visibles en cada imagen de objetivo
* [ ] No hay sombras ni obstrucciones sobre los objetivos
* [ ] Para el sistema de doble cámara: los objetivos están marcados en ambos tipos de cámara

***

## Procesamiento sin objetivos

### LATTICE: con una grabación DAQ

Si un sensor de luz DAQ ha registrado la irradiancia descendente durante las capturas de LATTICE, no se necesita ningún objetivo:

1. Importe el archivo `.daq` (o DAQ-M `.csv`) con las imágenes
2. Deje sin marcar la columna «Objetivo»
3. La reflectancia se calcula automáticamente a partir de la referencia de irradiación descendente del DAQ
4. La radiancia nunca necesita un objetivo ni un DAQ: se obtiene únicamente a partir de la calibración radiométrica de fábrica de la cámara

### Procesamiento sin ninguna referencia

También puede procesar sin objetivos y sin un DAQ:

1. Deje desmarcadas todas las casillas de la columna «Objetivo»
2. **Desactiva** «Calibración de reflectancia / balance de blancos» en la configuración del proyecto; de este modo, se omite por completo la detección de objetivos
3. La corrección de viñeteado seguirá aplicándose
4. El resultado no se calibrará para la reflectancia absoluta (LATTICE multiespectral sigue exportando productos sin debayering, de vista previa y de radiancia)

{% hint style="warning" %}
**No recomendado para trabajo científico Survey3**: sin calibración de reflectancia, los valores de píxeles de Survey3 representan únicamente el brillo relativo, no mediciones científicas de reflectancia. Utiliza objetivos de calibración (o, en el caso de LATTICE, un sensor de luz DAQ) para obtener resultados precisos y repetibles.
{% endhint %}

***

## Próximos pasos

Una vez que haya marcado sus imágenes objetivo:

1. **Revise su configuración**: consulte [Ajustar la configuración del proyecto](adjusting-project-settings.md)
2. **Inicia el procesamiento**: consulta [Inicio del procesamiento](starting-the-processing.md)
3. **Supervisa el progreso**: consulta [Supervisar el procesamiento](monitoring-the-processing.md)

Para obtener más información sobre los propios objetivos de calibración, consulta [Objetivos de calibración](../calibration-targets.md).

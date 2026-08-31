# Ajuste de la configuración del proyecto

Antes de procesar tus imágenes, es importante configurar los ajustes del proyecto para que se adapten a los requisitos de tu flujo de trabajo. El panel «Configuración del proyecto» (<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">) ofrece un control exhaustivo sobre la calibración, las opciones de procesamiento, los índices multiespectrales y los formatos de exportación.

## Cómo acceder a la configuración del proyecto

1. Abre tu proyecto en Chloros
2. Haz clic en el icono **Configuración del proyecto** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> en la barra lateral izquierda
3. El panel «Configuración del proyecto» muestra todas las opciones de configuración

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption><p>El panel «Configuración del proyecto»: visualización, detección de objetivos y procesamiento</p></figcaption></figure>{% hint style="info" %}
**La configuración se guarda automáticamente** junto con el proyecto. Al volver a abrir un proyecto, se restauran todos los ajustes.
{% endhint %}

***

## Configuración rápida para flujos de trabajo habituales

### Ajustes predeterminados (recomendados para la mayoría de los usuarios)

Los ajustes predeterminados funcionan bien para los flujos de trabajo típicos de Survey3 y LATTICE:

* ✅ **Corrección de viñeteado**: Activada
* ✅ **Calibración de reflectancia / balance de blancos**: Activada (utiliza objetivos MAPIR y/o datos del sensor de luz DAQ)
* ✅ **Método de debayering**: Estándar (rápido, calidad media)
* ✅ **Formato de exportación**: TIFF (16 bits)
* ✅ **Todos los productos de exportación**: Activado (LATTICE captura automáticamente la distribución en imágenes sin baya, vista previa, radiancia y reflectancia)

Solo tienes que importar tus imágenes y empezar a procesarlas con estos valores predeterminados.

***

## Descripción general de la configuración del proyecto

El panel «Configuración del proyecto» está organizado en las secciones que se indican a continuación. Dos secciones adicionales —**Sensor de luz DAQ**y**Alineación de la matriz**— aparecen automáticamente cuando el proyecto contiene los archivos pertinentes. Para consultar la documentación completa, véase [Configuración del proyecto](../project-settings/project-settings.md).

### Visualización

* **Resolución de las miniaturas de las imágenes**: Resolución de las miniaturas de la cuadrícula de imágenes. Opciones:**Predeterminada (512 px)**,**1024 px**,**2048 px**,**Resolución completa**. Solo para visualización: nunca afecta al procesamiento. Los valores más altos se ven más nítidos al ampliar la imagen, pero tardan más en cargarse.

### Detección de objetivos

Controla cómo Chloros identifica los objetivos de calibración en tus imágenes.

**Ajustes clave:*** **Área mínima de muestra de calibración (px)**: Umbral de tamaño para la detección de objetivos (por defecto:**25**, rango 0–10 000)
* **Agrupación mínima de objetivos (0-100)**: Umbral de similitud para agrupar regiones objetivo (por defecto:**60**)**Cuándo ajustarlo:**

* Aumenta el área de muestra si se producen detecciones erróneas
* Disminúela si no se detectan los objetivos
* Ajusta la agrupación si los objetivos se dividen en múltiples detecciones

{% hint style="info" %}
Estos ajustes aparecen desactivados cuando la **calibración de reflectancia / balance de blancos** está desactivada; si está desactivada, la detección de objetivos no se ejecuta en absoluto.
{% endhint %}

### Procesamiento

Opciones principales de procesamiento de imágenes y calibración.

**Ajustes clave:*** **Corrección de viñeteado**: Compensa el oscurecimiento de los bordes causado por el objetivo ✅ Recomendado
* **Calibración de reflectancia / balance de blancos**: Calibra las imágenes utilizando los objetivos detectados (Survey3) y/o los datos del sensor de luz DAQ (LATTICE) ✅ Recomendado
* **Método de despixelado**: algoritmo para convertir RAW a multiespectral de 3 canales
* **Intervalo mínimo de recalibración**: tiempo mínimo en segundos entre el uso de un objetivo de calibración (por defecto:**0** = usar todos, rango 0–3600)**Productos alternativos sin calibrar:**Cuando no es posible calibrar la reflectancia de un fotograma (no hay objetivo disponible o la calibración está desactivada), se exporta como uno de los dos productos alternativos —**existe exactamente uno de los dos por ejecución**—, elegido mediante el selector de corrección de viñeteado:

* **Exportar respuesta del sensor**: escribe `Sensor_Response_Images`; se utiliza cuando la corrección de viñeteado está**desactivada*** **Exportar con corrección de viñeteado**: guarda `Vignette_Corrected_Images`; se utiliza cuando la corrección de viñeteado está**activada**La casilla que no está activa aparece en gris. Al desmarcar la casilla activa, se impide por completo que se guarde ese archivo.**Productos de exportación de LATTICE** (se muestran para cada proyecto; se aplican a las capturas de LATTICE):

* **Exportar sin bayering**: la imagen lineal sin bayering (`Debayered_Images`). Se aplica a RGB y a los módulos multiespectrales.
* **Exportar vista previa**: la vista previa en pantalla (`Preview_Images`). RGB = balance de blancos (iluminante DAQ cuando esté disponible; en su defecto, escala de grises) + gamma; multiespectral = expansión a colores falsos.
* **Exportación de radiancia**: radiancia espectral de tipo float32 (`Radiance_Images`, W/m²/sr/nm). Solo módulos multiespectrales; no aplicable a los maestros RGB.
* ****Exportar reflectancia**: reflectancia uint16 (`Reflectance_Calibrated_Images`, DN 32768 = ρ 1,0) cuando una lectura descendente de `.daq` o un objetivo dentro del fotograma cubre dicho fotograma. Solo módulos multiespectrales.

Las cuatro opciones están **activadas por defecto**: un fotograma sin procesar de LATTICE importado se distribuye a todos los productos habilitados y aplicables en una única pasada de procesamiento. La casilla de selección**Exportar reflectancia** aparece desactivada cuando la calibración de reflectancia está desactivada. Los ajustes cuya opción principal los deshabilita siempre aparecen desactivados con una información sobre herramientas que indica qué opción hay que cambiar.**Ajustes avanzados:*** **Desfase horario del sensor de luz**: Horas respecto al UTC para la sincronización horaria del sensor de luz (por defecto: 0, rango de −12 a +12)
* **Aplicar correcciones PPK**: utiliza datos de GPS/pines de exposición de los archivos `.daq` (por defecto: desactivado)
* **Pines de exposición 1/2**: asigna cámaras a los pines de exposición para configuraciones de doble cámara

{% hint style="info" %}
**El nivel de entrada de LATTICE es automático.** Las capturas de LATTICE incluyen su nivel de procesamiento en los metadatos XMP, y el procesamiento siempre entra en el flujo de trabajo a partir del fotograma sin procesar; no hay nada que configurar en la interfaz gráfica de usuario. (El indicador CLI `--input-level` existe como una opción avanzada para usuarios avanzados en el caso de capturas con metadatos perdidos; véase la [Referencia de CLI](../reference/cli-reference.md).)
{% endhint %}

### Método de eliminación de mosaico

Actualmente ofrecemos dos métodos de eliminación de mosaico en Chloros:

#### Estándar (rápido, calidad media)

El método estándar procesa rápidamente, pero presenta ruido de color debido a la eliminación de mosaico, lo que da como resultado imágenes menos precisas y con más ruido.

#### Sensible a texturas (lento, máxima calidad) \[Solo Chloros+]

El método «Sensible a texturas» utiliza un algoritmo de eliminación de mosaicos de alta calidad que tiene en cuenta los bordes, combinado con un modelo de eliminación de ruido basado en IA/ML que elimina casi todo el ruido propio de este proceso. El modelo requiere memoria de la GPU (VRAM) para funcionar: con **7 GB o más de VRAM** puede procesar varias imágenes a la vez; por debajo de 7 GB, procesa una imagen cada vez (lo que resulta notablemente más lento). Véase [Adaptación dinámica de computación](../processing-architecture/dynamic-compute-adaptation.md).

{% hint style="info" %}
**Las capturas de LATTICE siempre utilizan el demosaico estándar.**No existe ningún modelo «Texture Aware» entrenado para LATTICE, por lo que esta opción no está disponible para las imágenes de LATTICE; no obstante, las imágenes**Survey3** del mismo proyecto sí pueden utilizarla.
{% endhint %}

### Índice (índices multiespectrales)

Configura qué índices de vegetación se van a calcular y exportar. El menú desplegable de la interfaz gráfica de usuario ofrece **27 fórmulas de índices predefinidas**.**Cómo añadir índices:**

1. Haga clic en el botón**«Añadir índice»**

2. Seleccione un índice del menú desplegable (NDVI, NDRE, GNDVI, etc.)
3. Configura los ajustes de visualización (colores de la tabla de conversión [LUT], rangos de valores).
4. Añade varios índices según sea necesario.

**Índices más utilizados:*** **NDVI**: Estado general de la vegetación (el más habitual)
* **NDRE**: Detección precoz de estrés junto con RedEdge
* **GNDVI**: Sensible a la concentración de clorofila
* **OSAVI**: Funciona bien con suelo visible
* **EVI**: Regiones con alto índice de área foliar (LAI)**Fórmulas personalizadas:**

* Crea fórmulas de índices multiespectrales personalizadas mediante operaciones matemáticas entre bandas en todos los canales de la imagen
* Guarda las fórmulas personalizadas para reutilizarlas
* Las fórmulas personalizadas son una función de Chloros+; su disponibilidad depende del nivel de tu plan

Para consultar todos los índices y fórmulas disponibles —incluidos aquellos nombres que solo están disponibles en la interfaz gráfica de usuario y los que también funcionan en CLI/SDK—, véase [Fórmulas de índices multiespectrales](../project-settings/multispectral-index-formulas.md).

### Exportar

Controla el formato del archivo de salida.

**Formatos disponibles**(configuración:**Formato de imagen calibrada**, por defecto**TIFF (16 bits)**):

* **TIFF (16 bits)**: Recomendado para SIG y análisis científicos
* **TIFF (32 bits, porcentaje)**: Valores en coma flotante
* **PNG (8 bits)**: Compresión sin pérdidas para visualización
* **JPG (8 bits)**: Archivos más pequeños, compresión con pérdidas

Los archivos de salida se guardan en la carpeta del proyecto, agrupados por cámara y formato: `<project>/<camera>/<format>/<Product>_Images/`. La radiación se guarda **siempre** como float32 en la carpeta `tiff32`, independientemente de esta configuración. Los archivos exportados conservan el nombre del archivo de origen; la carpeta identifica el producto. Consulta [Finalización del procesamiento](finishing-the-processing.md) para ver el árbol completo de resultados.

{% hint style="warning" %}
**Lectura de valores de reflectancia**: el valor de DN que corresponde a ρ = 1,0 depende de la cámara de origen: LATTICE utiliza 32768 (marcado como XMP `Chloros:PixelScale`), mientras que Survey3 utiliza 65535. Lee la etiqueta en lugar de suponer un valor constante. Consulta [Formatos de imagen de salida](../output-image-formats.md).
{% endhint %}

### Sensor de luz DAQ

Esta sección enumera todos los archivos de DAQ de luz descendente (`.daq` / `.csv`) de tu proyecto, con una fila por archivo, mostrando el modelo de sensor, el nombre del archivo y la corrección de la **tapa** del difusor aplicada a ese archivo.

* **Anulación de la corrección (todos los archivos)**: un único menú desplegable para todo el proyecto. La opción**Auto** (predeterminada) utiliza la corrección registrada en cada archivo; se supone que hay sol cuando no se ha registrado nada, ya que todos los DAQ MAPIR incluyen de serie el corrector de sol. Al seleccionar un límite máximo, se anula la configuración de todos los archivos: las grabaciones sin procesar se corrigen con él, y las grabaciones que ya incluyen un límite máximo se reajustan (se deshace la corrección registrada y se aplica el límite máximo seleccionado).
* Las filas advierten cuando un límite registrado era el valor predeterminado asumido por el concentrador en lugar de haber sido confirmado por el operador, y cuando el límite seleccionado no tiene perfil para ese modelo de dispositivo (la anulación se rechaza para ese archivo).

Las grabaciones de los DAQ realizadas en la pestaña «Sensores de luz» se añaden automáticamente al proyecto abierto, y los archivos `.daq` / `.csv` importados aparecen aquí tan pronto como se añaden.

<figure><img src="../.gitbook/assets/image (32).png" alt=""><figcaption><p>Configuración inferior del proyecto: índice, formato de exportación, la sección «Sensores de luz» de DAQ y los controles de plantillas y carpetas del proyecto</p></figcaption></figure>### Alineación de matrices

Esta sección aparece **solo** cuando al menos una imagen del proyecto contiene la transformación de alineación de módulo a módulo que las matrices LATTICE aplican en el momento de la captura (`Chloros:Alignment*` XMP). Muestra cuántas imágenes contienen etiquetas y qué cámara es la de referencia, con los siguientes controles:

* **Aplicar alineación de matriz** (por defecto: activada): deforma cada producto procesado (desbayering / vista previa / radiancia / reflectancia / índice) para adaptarlo a la geometría de referencia compartida de la matriz. Desactivada = exportación en la geometría nativa del sensor.
* **Recortar a la superposición común** (por defecto: activado): recorta las exportaciones alineadas a la región que comparten todos los módulos, de modo que cada banda tenga la misma huella. Si se desactiva, se mantiene el lienzo completo del sensor (relleno negro fuera de la fuente).
* **Remuestreo**:**Bilineal (suave, predeterminado)**,**Más cercano (conserva los valores exactos)**—sin mezcla entre píxeles, para un análisis radiométrico riguroso— o**Cúbico (más nítido)**.***

## Guardar y cargar ajustes

### Guardar plantilla de proyecto

Crea plantillas reutilizables para flujos de trabajo coherentes:

1. Configura todos los ajustes deseados en el panel «Ajustes del proyecto»
2. Desplázate hasta la sección **«Guardar plantilla de proyecto»** en la parte inferior
3. Introduce un nombre descriptivo para la plantilla (por ejemplo, «Survey3N\_RGN\_Agriculture»)
4. Haz clic en el icono de guardar

**Ventajas:**

* Aplica la misma configuración a varios proyectos
* Compartir configuraciones con los miembros del equipo
* Mantener la coherencia en encuestas repetidas

### Cargar una plantilla en un nuevo proyecto

Al crear un nuevo proyecto:

1. Selecciona **«Nuevo proyecto»** en el menú principal
2. Elige una plantilla de proyecto en el selector de plantillas opcional
3. Se aplican automáticamente todos los ajustes de la plantilla

### Directorio de trabajo

La configuración **«Directorio de trabajo»** especifica dónde se crean los nuevos proyectos de forma predeterminada:

* **Ubicación predeterminada**: `C:\Users\[Username]\Chloros Projects`
* **Cambiar ubicación**: haz clic en el icono de edición y selecciona una nueva carpeta
* **Compartido con CLI**: `chloros-cli` utiliza la misma configuración predeterminada de carpeta de proyectos
* **Cuándo cambiarlo**:
  * Unidad de red para la colaboración en equipo
  * Unidad diferente con más espacio de almacenamiento
  * Estructura de carpetas organizada por año/cliente

***

## Configuración de PPK (cinemática posprocesada)

Si se utilizan registradores DAQ MAPIR con GPS para una geolocalización precisa:

### Requisitos previos

* DAQ MAPIR con módulo GPS (GNSS)
* Archivo de registro .daq con entradas de pines de exposición
* Cámara conectada a los pines de exposición del DAQ durante la sesión de captura

### Pasos de configuración

1. Coloca el archivo de registro .daq en la carpeta de tu proyecto
2. En «Configuración del proyecto», marca la casilla **«Aplicar correcciones PPK»**

3. Configura el**«Desfase horario del sensor de luz»** si es necesario (por defecto: 0 para UTC)
4. Asigna las cámaras a los pines de exposición:
   * **Cámara única**: se asigna automáticamente al pin 1
   * **Dos cámaras**: asigna manualmente cada cámara al pin correcto**Asignación de pines de exposición:*** **Pin de exposición 1**: Selecciona el modelo de cámara en el menú desplegable
* **Pin de exposición 2**: Selecciona la segunda cámara o «No usar»
* No se puede asignar la misma cámara a ambos pines

{% hint style="warning" %}
**Importante**: Los pines de exposición deben asignarse correctamente a sus respectivas cámaras. Una asignación incorrecta dará lugar a datos de geolocalización erróneos.
{% endhint %}

***

## Escenarios avanzados

### Proyectos con varias cámaras

Al procesar imágenes de varias cámaras MAPIR en un mismo proyecto:

1. Chloros detecta automáticamente el modelo de cada cámara (tanto Survey3 como LATTICE)
2. A cada cámara se le asignan los perfiles de procesamiento adecuados y su propio árbol de carpetas de salida
3. PPK: Asigna manualmente cada cámara Survey3 al pin de exposición correcto
4. Todas las cámaras utilizan el mismo formato de exportación y los mismos índices

**Ejemplos**: Survey3W, RGN + Survey3N, OCN (sistema de doble cámara), o una matriz LATTICE que combina una cámara maestra RGB con módulos de banda estrecha

### Estudios con lapso de tiempo o en varias fechas

Para estudios repetidos de la misma zona a lo largo del tiempo:

1. Crea una plantilla con tu configuración estándar
2. Utiliza una configuración de objetivos de calibración coherente en cada sesión
3. Procesa cada fecha como un proyecto independiente
4. Utiliza una configuración idéntica para obtener resultados comparables
5. Exporta en el mismo formato para el análisis temporal

### Conjuntos de datos de gran tamaño

Para proyectos con muchas imágenes (más de 500):

* Considera la posibilidad de dividir el proyecto en proyectos más pequeños por fecha o zona
* Utiliza el procesamiento paralelo Chloros+ para obtener resultados más rápidos
* Considera el uso de CLI o API para la automatización por lotes
* Ajusta el intervalo mínimo de recalibración para reducir el tiempo de detección de objetivos

***

## Verificación de la configuración

Antes de iniciar el procesamiento, revisa estos ajustes clave:

* [ ] Modelo de cámara detectado correctamente en el explorador de archivos
* [ ] Corrección de viñeteado activada
* [ ] Calibración de reflectancia activada
* [ ] Para Survey3: se ha importado y comprobado al menos una imagen de objetivo de calibración; para LATTICE: hay un objetivo y/o un registro descendente de `.daq`
* [ ] Se han añadido los índices multiespectrales deseados
* [ ] Formato de exportación adecuado para su flujo de trabajo
* [ ] Ajustes PPK configurados (si se utiliza .daq con eventos de exposición)

***

## Próximos pasos

Una vez configurados los ajustes:

1. **Selecciona las imágenes de referencia para la calibración**: consulta [Selección de imágenes de referencia](choosing-target-images.md)
2. **Inicia el procesamiento**: consulta [Inicio del procesamiento](starting-the-processing.md)
3. **Supervisa el progreso**: consulta [Supervisar el procesamiento](monitoring-the-processing.md)

Para obtener información detallada sobre todos los ajustes disponibles, consulta la documentación de referencia [Ajustes del proyecto](../project-settings/project-settings.md).

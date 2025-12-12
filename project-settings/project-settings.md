# Configuración del proyecto

La barra lateral de configuración del proyecto <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> de Chloros le permite configurar todos los aspectos del procesamiento de imágenes, la detección de objetivos de calibración, los cálculos de índices multiespectrales y las opciones de exportación de su proyecto. Estos ajustes se guardan con su proyecto y pueden guardarse como plantillas para reutilizarlos en varios proyectos.

## Acceso a la configuración del proyecto

Para acceder a la configuración del proyecto:

1. Abra un proyecto en Chloros
2. Haga clic en la pestaña **Configuración del proyecto**  <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> en la barra lateral izquierda.
3. El panel de configuración mostrará todas las opciones de configuración disponibles organizadas por categoría.

***

## Detección de objetivos

Estos ajustes controlan cómo Chloros detecta y procesa los objetivos de calibración en sus imágenes.

### Área mínima de muestra de calibración (px)

* **Tipo**: Número
* **Rango**: De 0 a 10 000 píxeles
* **Predeterminado**: 25 píxeles
* **Descripción**: Establece el área mínima (en píxeles) necesaria para que una región detectada se considere una muestra válida del objetivo de calibración. Los valores más pequeños detectarán objetivos más pequeños, pero pueden aumentar los falsos positivos. Los valores más grandes requieren regiones objetivo más grandes y claras para su detección.
* **Cuándo ajustar**:
  * Aumente el valor si obtiene detecciones falsas en pequeños artefactos de imagen.
  * Disminuya el valor si los objetivos de calibración aparecen pequeños en las imágenes y no se detectan.

### Agrupación mínima de objetivos (0-100)

* **Tipo**: Número.
* **Rango**: De 0 a 100.
* **Valor predeterminado**: 60.
* **Descripción**: Controla el umbral de agrupamiento para agrupar regiones de colores similares al detectar objetivos de calibración. Los valores más altos requieren que se agrupen colores más similares, lo que da como resultado una detección de objetivos más conservadora. Los valores más bajos permiten una mayor variación de color dentro de un grupo objetivo.
* **Cuándo ajustar**:
  * Aumente si los objetivos de calibración se dividen en múltiples detecciones.
  * Disminuya si los objetivos de calibración con variación de color no se detectan completamente.

***

## Procesamiento

Estos ajustes controlan cómo Chloros procesa y calibra sus imágenes.

### Corrección de viñeteado

* **Tipo**: Casilla de verificación
* **Predeterminado**: Activado (marcado)
* **Descripción**: Aplica la corrección de viñeteado para compensar el oscurecimiento de la lente en los bordes de las imágenes. El viñeteado es un fenómeno óptico común en el que las esquinas y los bordes de una imagen aparecen más oscuros que el centro debido a las características de la lente.
* **Cuándo desactivar**: Desactive esta opción solo si la combinación de cámara y lente ya ha aplicado la corrección de viñeteado o si desea corregir manualmente el viñeteado en el posprocesamiento.

### Calibración de reflectancia/balance de blancos

* **Tipo**: Casilla de verificación
* **Predeterminado**: Activado (marcado)
* **Descripción**: Habilita la calibración automática de la reflectancia utilizando los objetivos de calibración detectados en sus imágenes. Esto normaliza los valores de reflectancia en todo su conjunto de datos y garantiza mediciones consistentes independientemente de las condiciones de iluminación.
* **Cuándo desactivar**: Desactive solo si desea procesar imágenes sin calibrar o si está utilizando un flujo de trabajo de calibración diferente.

### Método de desmosaico

* **Tipo**: Selección desplegable
* **Opciones**:
  * Alta calidad (más rápido): actualmente es la única opción disponible
* **Predeterminado**: Alta calidad (más rápido)
* **Descripción**: selecciona el algoritmo de demosaicing utilizado para convertir los datos sin procesar del sensor con patrón Bayer en imágenes a todo color. El método «Alta calidad (más rápido)» proporciona un equilibrio óptimo entre la velocidad de procesamiento y la calidad de la imagen.
* **Nota**: Es posible que se añadan métodos de debayer adicionales en futuras versiones de Chloros.

### Intervalo mínimo de recalibración

* **Tipo**: Número
* **Rango**: De 0 a 3600 segundos
* **Predeterminado**: 0 segundos
* **Descripción**: Establece el intervalo de tiempo mínimo (en segundos) entre el uso de los objetivos de calibración. Cuando se establece en 0, Chloros utilizará todos los objetivos de calibración detectados. Cuando se establece en un valor más alto, Chloros solo utilizará los objetivos de calibración que estén separados por al menos este número de segundos, lo que reduce el tiempo de procesamiento de los conjuntos de datos con capturas frecuentes de objetivos de calibración.
* **Cuándo ajustar**:
  * Establezca en 0 para obtener la máxima precisión de calibración cuando las condiciones de iluminación varían.
  * Aumente (por ejemplo, a 60-300 segundos) para un procesamiento más rápido cuando la iluminación es constante y tiene imágenes de objetivos de calibración frecuentes.

### Desviación de la zona horaria del sensor de luz

* **Tipo**: Número
* **Rango**: -12 a +12 horas
* **Predeterminado**: 0 horas
* **Descripción**: Especifica la diferencia horaria (en horas con respecto a UTC) para las marcas de tiempo de los datos del sensor de luz. Se utiliza al procesar archivos de datos PPK (cinemática posprocesada) para garantizar la sincronización correcta entre las capturas de imágenes y los datos GPS.
* **Cuándo ajustar**: Establezca este valor en la diferencia horaria local si sus datos PPK utilizan la hora local en lugar de UTC. Por ejemplo:
  * Hora del Pacífico: -8 o -7 (dependiendo del horario de verano)
  * Hora del Este: -5 o -4 (dependiendo del horario de verano)
  * Hora central europea: +1 o +2 (dependiendo del horario de verano)

### Aplicar correcciones PPK

* **Tipo**: Casilla de verificación
* **Predeterminado**: Desactivado (sin marcar)
* **Descripción**: Habilita el uso de correcciones cinemáticas posprocesadas (PPK) de los registradores DAQ MAPIR que contienen un GPS (GNSS). Cuando está habilitado, Chloros utilizará cualquier archivo de registro .daq que contenga datos de pines de exposición en el directorio de su proyecto y aplicará correcciones de geolocalización precisas a sus imágenes.
* **Requisito**: el archivo de registro .daq con entradas de pines de exposición debe estar presente en el directorio del proyecto
* **Cuándo habilitarlo**: se recomienda habilitar siempre la corrección PPK si hay entradas de retroalimentación de exposición en el archivo de registro .daq.

### Pin de exposición 1

* **Tipo**: Selección desplegable.
* **Visibilidad**: Solo visible cuando «Aplicar correcciones PPK» está habilitado Y hay datos de exposición disponibles para el pin 1.
* **Opciones**:
  * Nombres de modelos de cámara detectados en el proyecto.
  * «No usar»: ignora este pin de exposición.
* **Predeterminado**: seleccionado automáticamente en función de la configuración del proyecto.
* **Descripción**: asigna una cámara específica al pin de exposición 1 para la sincronización horaria PPK. El pin de exposición registra el momento exacto en que se dispara el obturador de la cámara, lo cual es fundamental para una geolocalización PPK precisa.
* **Comportamiento de selección automática**:
  * Una sola cámara + un solo pin: selecciona automáticamente la cámara
  * Una sola cámara + dos pines: el pin 1 se asigna automáticamente a la cámara
  * Varias cámaras: se requiere selección manual

### Pin de exposición 2

* **Tipo**: selección desplegable
* **Visibilidad**: solo visible cuando «Aplicar correcciones PPK» está habilitado Y hay datos de exposición disponibles para el pin 2
* **Opciones**:
  * Nombres de modelos de cámara detectados en el proyecto
  * «No usar»: ignora este pin de exposición.
* **Predeterminado**: seleccionado automáticamente en función de la configuración del proyecto.
* **Descripción**: asigna una cámara específica al pin de exposición 2 para la sincronización de tiempo PPK cuando se utiliza una configuración de doble cámara.
* **Comportamiento de selección automática**:
  * Cámara única + pin único: el pin 2 se establece automáticamente en «No usar».
  * Cámara única + dos pines: el pin 2 se establece automáticamente en «No utilizar»
  * Varias cámaras: se requiere selección manual
* **Nota**: no se puede asignar la misma cámara al pin 1 y al pin 2 simultáneamente.

***

## Índice

Estos ajustes le permiten configurar índices multiespectrales para su análisis y visualización.

### Añadir índice

* **Tipo**: Panel de configuración de índices especiales
* **Descripción**: Abre un panel interactivo en el que puede seleccionar y configurar índices de vegetación multiespectrales (NDVI, NDRE, EVI, etc.) para calcular durante el procesamiento de imágenes. Se pueden añadir varios índices, cada uno con su propia configuración de visualización.
* **Índices disponibles**: El sistema incluye más de 30 índices multiespectrales predefinidos, entre los que se incluyen:
* NDVI (Índice de vegetación de diferencia normalizada)
  * NDRE (diferencia normalizada RedEdge)
  * EVI (índice de vegetación mejorado)
  * GNDVI, SAVI, OSAVI, MSAVI2
  * Y muchos más (consulte [Fórmulas de índices multiespectrales](multispectral-index-formulas.md) para ver la lista completa)
* **Características**:
  * Seleccione entre fórmulas de índice predefinidas.
  * Configure gradientes de color de visualización (LUT, tablas de consulta).
  * Establezca valores umbral para el análisis.
  * Cree fórmulas de índice personalizadas.

### Fórmulas personalizadas (característica Chloros+)

* **Tipo**: Matriz de definiciones de fórmulas personalizadas
* **Descripción**: Permite crear y guardar fórmulas de índice multiespectral personalizadas utilizando matemáticas de banda. Las fórmulas personalizadas se guardan con la configuración del proyecto y se pueden utilizar igual que los índices integrados.
* **Cómo crear**:
  1. En el panel de configuración del índice, busque la opción de fórmula personalizada
  2. Defina su fórmula utilizando identificadores de banda (por ejemplo, NIR, Red, Green, Blue).
  3. Guarde la fórmula con un nombre descriptivo.
* **Sintaxis de la fórmula**: se admiten operaciones matemáticas estándar, entre las que se incluyen:
  * Aritmética: `+`, `-`, `*`, `/`
  * Paréntesis para el orden de las operaciones
  * Referencias de banda: NIR, Red, Green, Blue, RedEdge, Cyan, Orange, NIR1, NIR2

***

## Exportar

Estos ajustes controlan el formato y la calidad de las imágenes procesadas exportadas.

### Formato de imagen calibrado

* **Tipo**: Selección desplegable
* **Opciones**:
  * **TIFF (16 bits)**: formato TIFF sin comprimir de 16 bits
  * **TIFF (32 bits, porcentaje)**: TIFF de 32 bits con coma flotante con valores de reflectancia en porcentajes
  * **PNG (8 bits)** - Formato PNG comprimido de 8 bits.
  * **JPG (8 bits)**: formato JPEG comprimido de 8 bits.
* **Predeterminado**: TIFF (16 bits).
* **Descripción**: Selecciona el formato de archivo para guardar imágenes procesadas y calibradas.
* **Recomendaciones de formato**:
  * **TIFF (16 bits)**: Recomendado para análisis científicos y flujos de trabajo profesionales. Conserva la máxima calidad de datos sin artefactos de compresión. Ideal para análisis multiespectrales y procesamiento posterior en software GIS.
  * **TIFF (32 bits, porcentaje)**: Ideal para flujos de trabajo que requieren valores de reflectancia en forma de porcentajes (0-100 %). Ofrece la máxima precisión para mediciones radiométricas.
  * **PNG (8 bits)**: adecuado para visualización web y visualización general. Archivos de menor tamaño con compresión sin pérdidas, pero con un rango dinámico reducido.
  * **JPG (8 bits)**: archivos de menor tamaño, ideales solo para vistas previas y visualización web. Utiliza compresión con pérdidas, lo que no es adecuado para análisis científicos.

***

## Guardar plantilla de proyecto

Esta función le permite guardar la configuración actual de su proyecto como una plantilla reutilizable.

* **Tipo**: Entrada de texto + botón Guardar
* **Descripción**: Introduzca un nombre descriptivo para su plantilla de configuración y haga clic en el icono Guardar. La plantilla almacenará toda la configuración actual de su proyecto (detección de objetivos, opciones de procesamiento, índices y formato de exportación) para facilitar su reutilización en proyectos futuros.
* **Casos de uso**:
  * Crear plantillas para diferentes sistemas de cámaras (RGB, multiespectral, NIR)
  * Guardar configuraciones estándar para tipos de cultivos específicos o flujos de trabajo de análisis
  * Compartir ajustes coherentes entre todo el equipo
* **Cómo utilizarlo**:
  1. Configure todos los ajustes del proyecto que desee
  2. Introduzca un nombre para la plantilla (por ejemplo, «RedEdge Survey3 NDVI Estándar»).
  3. Haga clic en el icono Guardar.
  4. Ahora la plantilla se puede cargar al crear nuevos proyectos.

***

## Guardar carpeta de proyectos

Esta configuración especifica dónde se guardan los nuevos proyectos de forma predeterminada.

* **Tipo**: Visualización de la ruta del directorio + botón Editar
* **Predeterminado**: `C:\Users\[Username]\Chloros Projects`
* **Descripción**: Muestra el directorio predeterminado actual donde se crean los nuevos proyectos Chloros. Haga clic en el icono de edición para seleccionar un directorio diferente.
* **Cuándo cambiarlo**:
  * Configure una unidad de red para la colaboración en equipo.
  * Cambie a una unidad con más espacio de almacenamiento para conjuntos de datos grandes.
  * Organice los proyectos por año, cliente o tipo de proyecto en diferentes carpetas.
* **Nota**: Cambiar esta configuración solo afecta a los proyectos NUEVOS. Los proyectos existentes permanecen en sus ubicaciones originales.

***

## Persistencia de la configuración

Todos los ajustes del proyecto se guardan automáticamente con el archivo del proyecto (formato de proyecto `.mapir`). Cuando vuelva a abrir un proyecto, todos los ajustes se restaurarán exactamente como los dejó.

### Jerarquía de ajustes

Los ajustes se aplican en el siguiente orden:

1. **Valores predeterminados del sistema**: valores predeterminados integrados definidos por Chloros
2. **Configuración de la plantilla**: si carga una plantilla al crear un proyecto
3. **Configuración del proyecto guardada**: configuración guardada con el archivo del proyecto
4. **Ajustes manuales**: cualquier cambio que realice durante la sesión actual

### Configuración y procesamiento de imágenes

La mayoría de los cambios en la configuración (especialmente en las categorías Procesamiento y Exportación) provocarán un nuevo procesamiento de las imágenes para reflejar la nueva configuración. Sin embargo, algunos ajustes son «solo para exportación» y no requieren un nuevo procesamiento inmediato:

* Guardar plantilla de proyecto
* Directorio de trabajo
* Formato de imagen calibrado (se aplica al exportar)

***

## Prácticas recomendadas

1. **Comience con los valores predeterminados**: la configuración predeterminada funciona bien para la mayoría de los sistemas de cámaras MAPIR y los flujos de trabajo típicos.
2. **Cree plantillas**: una vez que haya optimizado la configuración para un flujo de trabajo o una cámara específicos, guárdelos como plantilla para garantizar la coherencia entre proyectos.
3. **Pruebe antes del procesamiento completo**: cuando experimente con nuevos ajustes, pruébelos en un pequeño subconjunto de imágenes antes de procesar todo el conjunto de datos.
4. **Documente sus ajustes**: utilice nombres de plantillas descriptivos que indiquen el sistema de cámara, el tipo de procesamiento y el uso previsto (por ejemplo, «Survey3\_RGB\_NDVI\_Agricultura»).
5. **Selección del formato de exportación**: elija el formato de exportación en función del uso final:
   * Análisis científico → TIFF (16 bits o 32 bits)
   * Procesamiento GIS → TIFF (16 bits)
   * Visualización rápida → PNG (8 bits)
   * Compartir en la web → JPG (8 bits)

***

Para obtener más información sobre los índices multiespectrales en Chloros, consulte la página [Fórmulas de índices multiespectrales](multispectral-index-formulas.md).

# Configuración de los ajustes del proyecto

Antes de procesar las imágenes, es importante configurar los ajustes del proyecto para que se adapten a los requisitos de tu flujo de trabajo. El panel «Ajustes del proyecto» <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> ofrece un control exhaustivo sobre la calibración, las opciones de procesamiento, los índices multiespectrales y los formatos de exportación.

## Acceso a la configuración del proyecto

1. Abre tu proyecto en Chloros
2. Haz clic en el icono **Configuración del proyecto** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> en la barra lateral izquierda
3. El panel de configuración del proyecto muestra todas las opciones de configuración

{% hint style="info" %}
**La configuración se guarda automáticamente** con el proyecto. Al volver a abrir un proyecto, se restauran todos los ajustes.
{% endhint %}

***

## Configuración rápida para flujos de trabajo habituales

### Ajustes predeterminados (recomendados para la mayoría de los usuarios)

Para los flujos de trabajo típicos con cámaras MAPIR y Survey3, los ajustes predeterminados funcionan bien:

* ✅ **Corrección de viñeteado**: Activada
* ✅ **Calibración de reflectancia**: Activada (requiere imágenes de objetivos MAPIR)
* ✅ **Método de debayerización**: Estándar (Rápido, Calidad media)
* ✅ **Formato de exportación**: TIFF (16 bits)

Simplemente importe sus imágenes y comience el procesamiento con estos valores predeterminados.

***

## Descripción general de la configuración del proyecto

El panel de configuración del proyecto está organizado en varias categorías. A continuación se ofrece un resumen de cada sección. Para consultar la documentación completa, véase [Configuración del proyecto](../project-settings/project-settings.md).

### Detección de objetivos

Controla cómo Chloros identifica los objetivos de calibración en sus imágenes.

**Configuración clave:*** **Área mínima de muestra de calibración**: umbral de tamaño para la detección de objetivos (por defecto: 25 píxeles)
* **Agrupación mínima de objetivos**: umbral de similitud para agrupar regiones objetivo (por defecto: 60)**Cuándo ajustar:**

* Aumente el área de muestra si se producen detecciones falsas
* Redúzcala si no se detectan objetivos
* Ajuste la agrupación si los objetivos se dividen en múltiples detecciones

### Procesamiento

Opciones principales de procesamiento de imágenes y calibración.

**Ajustes clave:*** **Corrección de viñeteado**: Compensa el oscurecimiento de los bordes de la lente ✅ Recomendado
* **Calibración de reflectancia**: Normaliza los valores utilizando objetivos de calibración ✅ Recomendado
* **Método de debayerización**: algoritmo para convertir RAW a multiespectral de 3 canales
* **Intervalo mínimo de recalibración**: tiempo entre el uso de objetivos de calibración (0 = usar todos)**Ajustes avanzados:*** **Desfase de zona horaria del sensor de luz**: para la sincronización horaria PPK (por defecto: 0)
* **Aplicar correcciones PPK**: utiliza datos de GPS/pines de exposición de archivos .daq
* **Pines de exposición 1/2**: asigna cámaras a los pines de exposición para configuraciones de doble cámara

### Método de debayering

Actualmente ofrecemos 2 métodos de debayering en Chloros:

#### Estándar (rápido, calidad media)

El método estándar procesa rápidamente, pero muestra ruido de color en el debayering, lo que da como resultado imágenes menos precisas y con más ruido.

#### Sensible a la textura (lento, máxima calidad) \[Solo Chloros+]

El método sensible a la textura utiliza un debayering de alta calidad sensible a los bordes, combinado con un modelo de eliminación de ruido basado en IA/ML que elimina casi todo el ruido del debayering. El modelo «Sensible a la textura» requiere memoria de la GPU (VRAM) para funcionar. Recomendamos utilizarlo cuando se disponga de más de 4 GB de VRAM para un procesamiento más rápido.

### Índice (Índices multiespectrales)

Configura qué índices de vegetación se van a calcular y exportar.

**Cómo añadir índices:**

1. Haz clic en el botón**«Añadir índice»**

2. Seleccione un índice del menú desplegable (NDVI, NDRE, GNDVI, etc.)
3. Configure los ajustes de visualización (colores LUT, rangos de valores)
4. Añada varios índices según sea necesario

**Índices populares:*** **NDVI**: Estado general de la vegetación (el más común)
* **NDRE**: Detección temprana de estrés junto con RedEdge
* **GNDVI**: Sensible a la concentración de clorofila
* **OSAVI**: Funciona bien con suelo visible
* **EVI**: Regiones con alto índice de área foliar (LAI)**Fórmulas personalizadas (solo Chloros+):**

* Crea fórmulas de índices multiespectrales personalizadas
* Utiliza operaciones matemáticas entre bandas con todos los canales de la imagen
* Guarda fórmulas personalizadas para reutilizarlas

Para ver todos los índices y fórmulas disponibles, consulta [Fórmulas de índices multiespectrales](../project-settings/multispectral-index-formulas.md).

### Exportar

Controla el formato y la calidad del archivo de salida.

**Formatos disponibles:*** **TIFF (16 bits)**: Recomendado para SIG y análisis científicos (rango 0-65 535)
* **TIFF (32 bits, porcentaje)**: Valores de reflectancia en coma flotante (rango 0,0-1,0)
* **PNG (8 bits)**: Compresión sin pérdidas para visualización (rango de 0 a 255)
* **JPG (8 bits)**: Archivos más pequeños, compresión con pérdidas (rango de 0 a 255)***

## Guardar y cargar configuraciones

### Guardar plantilla de proyecto

Cree plantillas reutilizables para flujos de trabajo coherentes:

1. Configure todos los ajustes deseados en el panel Ajustes del proyecto
2. Desplácese hasta la sección **«Guardar plantilla de proyecto»** en la parte inferior
3. Introduzca un nombre descriptivo para la plantilla (por ejemplo, «Survey3N\_RGN\_Agricultura»)
4. Haga clic en el icono de guardar

**Ventajas:**

* Aplica la misma configuración a varios proyectos
* Comparte configuraciones con los miembros del equipo
* Mantén la coherencia en encuestas repetidas

### Cargar plantilla en un nuevo proyecto

Al crear un nuevo proyecto:

1. Selecciona **«Nuevo proyecto»** en el menú principal
2. Elige la opción **«Cargar desde plantilla»**

3. Selecciona la plantilla guardada
4. Toda la configuración se aplica automáticamente

### Directorio de trabajo

La configuración **«Carpeta de guardado del proyecto»** especifica dónde se crean los nuevos proyectos de forma predeterminada:

* **Ubicación predeterminada**: `C:\Users\[Username]\Chloros Projects`
* **Cambiar ubicación**: Haga clic en el icono de edición y seleccione una nueva carpeta
* **Cuándo cambiar**:
  * Unidad de red para la colaboración en equipo
  * Otra unidad con más espacio de almacenamiento
  * Estructura de carpetas organizada por año/cliente

***

## Configuración de PPK (cinemática posprocesada)

Si utiliza registradores DAQ MAPIR con GPS para una geolocalización precisa:

### Requisitos previos

* DAQ MAPIR con módulo GPS (GNSS)
* Archivo de registro .daq con entradas de pines de exposición
* Cámara conectada a los pines de exposición del DAQ durante la sesión de captura

### Pasos de configuración

1. Coloque el archivo de registro .daq en la carpeta de su proyecto
2. En la configuración del proyecto, active la casilla **«Aplicar correcciones PPK»**

3. Configure**«Desfase de zona horaria del sensor de luz»** si es necesario (por defecto: 0 para UTC)
4. Asigne las cámaras a los pines de exposición:
   * **Cámara única**: se asigna automáticamente al pin 1
   * **Cámaras duales**: asigne manualmente cada cámara al pin correcto**Asignación de pines de exposición:*** **Pin de exposición 1**: Seleccione el modelo de cámara en el menú desplegable
* **Pin de exposición 2**: Seleccione la segunda cámara o «No usar»
* No se puede asignar la misma cámara a ambos pines

{% hint style="warning" %}
**Importante**: Los pines de exposición deben asignarse correctamente a sus respectivas cámaras. Una asignación incorrecta dará lugar a datos de geolocalización erróneos.
{% endhint %}

***

## Escenarios avanzados

### Proyectos con varias cámaras

Al procesar imágenes de varias cámaras MAPIR en un mismo proyecto:

1. Chloros detecta automáticamente el modelo de cada cámara
2. Cada cámara obtiene el perfil de procesamiento adecuado
3. PPK: Asigne manualmente cada cámara al pin de exposición correcto
4. Todas las cámaras utilizan el mismo formato de exportación e índices

**Ejemplo**: Survey3W RGN + Survey3N OCN equipo de dos cámaras

### Estudios con lapso de tiempo o de múltiples fechas

Para estudios repetidos de la misma zona a lo largo del tiempo:

1. Crea una plantilla con tu configuración estándar
2. Utiliza una configuración de objetivos de calibración coherente en cada sesión
3. Procesa cada fecha como un proyecto independiente
4. Utiliza una configuración idéntica para obtener resultados comparables
5. Exporta en el mismo formato para el análisis temporal

### Conjuntos de datos de gran tamaño

Para proyectos con muchas imágenes (más de 500):

* Considere dividir el proyecto en proyectos más pequeños por fecha o zona
* Utilice el procesamiento paralelo Chloros+ para obtener resultados más rápidos
* Considere CLI o API para la automatización por lotes
* Ajuste el intervalo mínimo de recalibración para reducir el tiempo de detección de objetivos

***

## Verificación de la configuración

Antes de comenzar el procesamiento, revise estos ajustes clave:

* [ ] Modelo de cámara detectado correctamente en el explorador de archivos
* [ ] Corrección de viñeteado habilitada
* [ ] Calibración de reflectancia habilitada
* [ ] Al menos una imagen de objetivo de calibración importada
* [ ] Índices multiespectrales deseados añadidos
* [ ] Formato de exportación adecuado para su flujo de trabajo
* [ ] Ajustes PPK configurados (si utiliza .daq con eventos de exposición)

***

## Próximos pasos

Una vez configurados los ajustes:

1. **Marque las imágenes de referencia de calibración** - Consulte [Selección de imágenes de referencia](choosing-target-images.md)
2. **Inicie el procesamiento** - Consulte [Inicio del procesamiento](starting-the-processing.md)
3. **Supervise el progreso**: consulte [Supervisar el procesamiento](monitoring-the-processing.md)

Para obtener información detallada sobre todos los ajustes disponibles, consulte la documentación de referencia [Ajustes del proyecto](../project-settings/project-settings.md).

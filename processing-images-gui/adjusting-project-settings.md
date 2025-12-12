# Ajuste de la configuración del proyecto

Antes de procesar las imágenes, es importante configurar los ajustes del proyecto para que se adapten a los requisitos de su flujo de trabajo. El panel Configuración del proyecto <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> ofrece un control completo sobre la calibración, las opciones de procesamiento, los índices multiespectrales y los formatos de exportación.

## Acceso a la configuración del proyecto

1. Abra su proyecto en Chloros
2. Haga clic en el icono **Configuración del proyecto** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> en la barra lateral izquierda
3. El panel Configuración del proyecto muestra todas las opciones de configuración

{% hint style=&quot;info&quot; %}
**La configuración se guarda automáticamente** con el proyecto. Al volver a abrir un proyecto, se restauran todos los ajustes.
{% endhint %}

***

## Configuración rápida para flujos de trabajo comunes

### Ajustes predeterminados (recomendados para la mayoría de los usuarios)

Para los flujos de trabajo típicos de la cámara MAPIR Survey3, la configuración predeterminada funciona bien:

* ✅ **Corrección de viñeta**: habilitada
* ✅ **Calibración de reflectancia**: habilitada (requiere imágenes de objetivos MAPIR)
* ✅ **Método Debayer**: Alta calidad (más rápido)
* ✅ **Formato de exportación**: TIFF (16 bits)

Simplemente importe sus imágenes y comience a procesarlas con estos valores predeterminados.

***

## Descripción general de la configuración del proyecto

El panel Configuración del proyecto está organizado en varias categorías. A continuación se ofrece un resumen de cada sección. Para obtener la documentación completa, consulte [Configuración del proyecto](../project-settings/project-settings.md).

### Detección de objetivos

Controla cómo Chloros identifica los objetivos de calibración en sus imágenes.

**Configuración clave:**

* **Área mínima de muestra de calibración**: umbral de tamaño para la detección de objetivos (predeterminado: 25 píxeles).
* **Agrupación mínima de objetivos**: umbral de similitud para agrupar regiones objetivo (predeterminado: 60).

**Cuándo ajustar:**

* Aumente el área de muestra si obtiene detecciones falsas.
* Disminúyala si no se detectan los objetivos.
* Ajuste la agrupación si los objetivos se dividen en múltiples detecciones.

### Procesamiento

Opciones principales de procesamiento y calibración de imágenes.

**Ajustes clave:**

* **Corrección de viñeta**: compensa el oscurecimiento de la lente en los bordes ✅ Recomendado
* **Calibración de reflectancia**: normaliza los valores utilizando objetivos de calibración ✅ Recomendado
* **Método Debayer**: algoritmo para convertir RAW a multiespectral de 3 canales
* **Intervalo mínimo de recalibración**: Tiempo entre el uso de objetivos de calibración (0 = usar todos)

**Ajustes avanzados:**

* **Desfase de zona horaria del sensor de luz**: Para la sincronización horaria PPK (por defecto: 0)
* **Aplicar correcciones PPK**: Utiliza datos GPS/pin de exposición de archivos .daq
* **Pin de exposición 1/2**: asigna cámaras a pines de exposición para configuraciones de doble cámara

### Índice (índices multiespectrales)

Configure qué índices de vegetación calcular y exportar.

**Cómo añadir índices:**

1. Haga clic en el botón **«Añadir índice»**
2. Seleccione un índice en el menú desplegable (NDVI, NDRE, GNDVI, etc.).
3. Configure los ajustes de visualización (colores LUT, rangos de valores).
4. Añada varios índices según sea necesario.

**Índices populares:**

* **NDVI**: Salud general de la vegetación (el más común).
* **NDRE**: Detección temprana de estrés con RedEdge.
* **GNDVI**: Sensible a la concentración de clorofila.
* **OSAVI**: Funciona bien con suelo visible
* **EVI**: Regiones con alto índice de área foliar (LAI)

**Fórmulas personalizadas (solo Chloros+):**

* Cree fórmulas de índice multiespectral personalizadas
* Utilice matemáticas de banda con todos los canales de imagen
* Guarde fórmulas personalizadas para reutilizarlas

Para ver todos los índices y fórmulas disponibles, consulte [Fórmulas de índice multiespectral](../project-settings/multispectral-index-formulas.md).

### Exportar

Controla el formato y la calidad del archivo de salida.

**Formatos disponibles:**

* **TIFF (16 bits)**: recomendado para SIG y análisis científico (rango de 0 a 65 535).
* **TIFF (32 bits, porcentaje)**: valores de reflectancia en coma flotante (rango de 0,0 a 1,0).
* **PNG (8 bits)**: compresión sin pérdidas para visualización (rango de 0 a 255).
* **JPG (8 bits)**: archivos más pequeños, compresión con pérdidas (rango de 0 a 255).

***

## Guardar y cargar ajustes

### Guardar plantilla de proyecto

Cree plantillas reutilizables para flujos de trabajo coherentes:

1. Configure todos los ajustes deseados en el panel Ajustes del proyecto.
2. Desplácese hasta la sección **«Guardar plantilla de proyecto»** en la parte inferior.
3. Introduzca un nombre descriptivo para la plantilla (por ejemplo, «Survey3N\_RGN\_Agricultura»).
4. Haga clic en el icono Guardar.

**Ventajas:**

* Aplique la misma configuración en varios proyectos.
* Comparta las configuraciones con los miembros del equipo.
* Mantenga la coherencia en las encuestas repetidas.

### Cargar plantilla en un nuevo proyecto

Al crear un nuevo proyecto:

1. Seleccione **«Nuevo proyecto»** en el menú principal.
2. Elija la opción **«Cargar desde plantilla»**.
3. Seleccione la plantilla guardada.
4. Se aplicarán automáticamente todos los ajustes.

### Directorio de trabajo

La configuración **«Guardar carpeta del proyecto»** especifica dónde se crean los nuevos proyectos de forma predeterminada:

* **Ubicación predeterminada**: `C:\Users\[Username]\Chloros Projects`.
* **Cambiar ubicación**: haga clic en el icono de edición y seleccione una nueva carpeta.
* **Cuándo cambiarla**:
  * Unidad de red para la colaboración en equipo.
  * Unidad diferente con más espacio de almacenamiento.
  * Estructura de carpetas organizada por año/cliente.

***

## Configuración de PPK (cinemática posprocesada)

Si utiliza registradores DAQ MAPIR con GPS para una geolocalización precisa:

### Requisitos previos

* DAQ MAPIR con módulo GPS (GNSS)
* Archivo de registro .daq con entradas de pines de exposición
* Cámara conectada a los pines de exposición DAQ durante la sesión de captura

### Pasos de configuración

1. Coloque el archivo de registro .daq en la carpeta de su proyecto
2. En la configuración del proyecto, active la casilla **«Aplicar correcciones PPK»**
3. Configure **«Desfase horario del sensor de luz»** si es necesario (por defecto: 0 para UTC)
4. Asigne las cámaras a los pines de exposición:
   * **Cámara única**: Asignada automáticamente al pin 1.
   * **Dos cámaras**: Asigne manualmente cada cámara al pin correcto.

**Asignación de pines de exposición:**

* **Pin de exposición 1**: Seleccione el modelo de cámara en el menú desplegable.
* **Pin de exposición 2**: Seleccione la segunda cámara o «No usar».
* No se puede asignar la misma cámara a ambos pines.

{% hint style=&quot;warning&quot; %}
**Importante**: Los pines de exposición deben asignarse correctamente a sus respectivas cámaras. Una asignación incorrecta dará lugar a datos de geolocalización erróneos.
{% endhint %}

***

## Escenarios avanzados

### Proyectos con varias cámaras

Al procesar imágenes de varias cámaras MAPIR en un proyecto:

1. Chloros detecta automáticamente cada modelo de cámara
2. Cada cámara obtiene el perfil de procesamiento adecuado
3. PPK: Asigne manualmente cada cámara al pin de exposición correcto.
4. Todas las cámaras utilizan el mismo formato de exportación e índices.

**Ejemplo**: Survey3W RGN + Survey3N OCN equipo de doble cámara

### Estudios con lapso de tiempo o en varias fechas

Para estudios repetidos de la misma zona a lo largo del tiempo:

1. Cree una plantilla con su configuración estándar.
2. Utilice una configuración de objetivo de calibración coherente en cada sesión.
3. Procese cada fecha como un proyecto independiente.
4. Utilice una configuración idéntica para obtener resultados comparables.
5. Exporte en el mismo formato para el análisis temporal.

### Conjuntos de datos grandes

Para proyectos con muchas imágenes (más de 500):

* Considere dividirlos en proyectos más pequeños por fecha o área.
* Utilice el procesamiento paralelo Chloros+ para obtener resultados más rápidos.
* Considere CLI o API para la automatización por lotes.
* Ajuste el intervalo mínimo de recalibración para reducir el tiempo de detección del objetivo.

***

## Verificación de la configuración

Antes de comenzar el procesamiento, revise estos ajustes clave:

* [ ] Modelo de cámara detectado correctamente en el explorador de archivos.
* [ ] Corrección de viñeta habilitada.
* [ ] Calibración de reflectancia habilitada.
* [ ] Al menos una imagen de objetivo de calibración importada.
* [ ] Índices multiespectrales deseados añadidos.
* [ ] Formato de exportación adecuado para su flujo de trabajo.
* [ ] Ajustes PPK configurados (si utiliza .daq con eventos de exposición).

***

## Próximos pasos

Una vez configurados los ajustes:

1. **Marque las imágenes objetivo de calibración**: consulte [Selección de imágenes objetivo](choosing-target-images.md)
2. **Inicie el procesamiento**: consulte [Inicio del procesamiento](starting-the-processing.md)
3. **Supervise el progreso**: consulte [Supervisar el procesamiento](monitoring-the-processing.md).

Para obtener información detallada sobre todos los ajustes disponibles, consulte la documentación de referencia [Ajustes del proyecto](../project-settings/project-settings.md).

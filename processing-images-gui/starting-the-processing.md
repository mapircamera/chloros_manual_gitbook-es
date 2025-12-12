# Inicio del procesamiento

Una vez que haya importado sus imágenes, marcado sus objetivos de calibración y configurado los ajustes de su proyecto, estará listo para comenzar el procesamiento. Esta página le guiará a través del inicio del proceso de procesamiento Chloros.

## Lista de verificación previa al procesamiento

Antes de hacer clic en el botón Iniciar, compruebe que todo está listo:

* [ ] **Archivos importados**: todas las imágenes aparecen en el explorador de archivos.
* [ ] **Imágenes objetivo marcadas**: columna Objetivo marcada para las imágenes de calibración.
* [ ] **Modelos de cámara detectados**: la columna Modelo de cámara muestra las cámaras correctas
* [ ] **Configuración configurada**: ajustes del proyecto revisados y ajustados
* [ ] **Índices seleccionados**: índices multiespectrales deseados añadidos (si es necesario)
* [ ] **Formato de exportación elegido**: formato de salida adecuado para su flujo de trabajo

{% hint style=&quot;info&quot; %}
**Consejo**: Haga clic en algunas imágenes del explorador de archivos para verificar que se hayan cargado correctamente antes de procesarlas.
{% endhint %}

***

## Inicio del procesamiento

### Localice el botón de inicio

El botón de inicio/reproducción se encuentra en la barra superior de Chloros:

* Posición: parte superior central de la ventana
* Icono: **botón de reproducción/inicio** <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">
* Estado: el botón está habilitado (brillante) cuando está listo para procesar

### Haga clic para iniciar

1. Haga clic en el **botón Reproducir/Iniciar** en la barra superior
2. El procesamiento comienza inmediatamente
3. El botón se deshabilita (aparece en gris) durante el procesamiento
4. La barra de progreso se actualiza y muestra el estado del procesamiento

{% hint style=&quot;success&quot; %}
**Procesamiento iniciado**: Una vez hecho clic, Chloros gestiona automáticamente todos los pasos del procesamiento: detección de objetivos, debayering, calibración, cálculo de índices y exportación.
{% endhint %}

***

## Comprensión de los modos de procesamiento

Chloros funciona en dos modos de procesamiento diferentes, dependiendo de su licencia:

### Modo gratuito (procesamiento secuencial)

**Disponible para todos los usuarios**

**Cómo funciona:**

* Procesa las imágenes de una en una, de forma secuencial.
* Funcionamiento de un solo subproceso.
* Menor uso de memoria.

**La barra de progreso muestra dos etapas:**

1. **Detección de objetivos**: búsqueda de objetivos de calibración.
2. **Procesamiento**: aplicación de la calibración y exportación de imágenes.

**Tiempo de procesamiento:**

* Mucho más lento que el modo paralelo Chloros+.
* Adecuado para conjuntos de datos pequeños y medianos (&lt; 200 imágenes).

### Modo Chloros+ (procesamiento paralelo)

**Requiere la licencia Chloros+.**

**Cómo funciona:**

* Procesa varias imágenes simultáneamente
* Funcionamiento multihilo (hasta 16 trabajadores paralelos)
* Utiliza varios núcleos de CPU
* Aceleración GPU (CUDA) opcional con tarjetas gráficas NVIDIA

**La barra de progreso muestra 4 etapas:**

1. **Detección**: búsqueda de objetivos de calibración
2. **Análisis**: examen de los metadatos de la imagen y preparación del proceso
3. **Calibración**: aplicación de correcciones y calibraciones
4. **Exportación**: guardado de imágenes procesadas e índices

**Interacción con la barra de progreso:**

* **Pase el ratón** por encima de la barra para ver el panel desplegable detallado de 4 etapas
* **Haga clic** en la barra de progreso para congelar el panel desplegable en su lugar
* **Haga clic de nuevo** para desbloquear y ocultar el panel.

**Tiempo de procesamiento:**

* Significativamente más rápido que el modo libre.
* Se adapta al número de núcleos de la CPU.
* La aceleración de la GPU mejora aún más la velocidad.

{% hint style=&quot;info&quot; %}
**Chloros+ Velocidad**: El procesamiento paralelo puede ser entre 5 y 10 veces más rápido que el modo secuencial para conjuntos de datos grandes. Un proyecto de 500 imágenes que tarda 2 horas en el modo gratuito puede completarse en 15-20 minutos con Chloros+.
{% endhint %}

***

## Qué ocurre durante el procesamiento

### Etapa 1: Detección de objetivos

**Qué hace Chloros:**

* Escanea las imágenes de objetivos marcadas (o todas las imágenes si no hay ninguna marcada).
* Identifica los 4 paneles de calibración en cada objetivo
* Extrae los valores de reflectancia de los paneles objetivo
* Registra las marcas de tiempo de los objetivos para programar la calibración

**Duración:** 1-30 segundos (con objetivos marcados), 5-30+ minutos (sin marcar)

### Etapa 2: Desbayering (conversión RAW)

**Qué hace Chloros:**

* Convierte los datos del patrón Bayer RAW en imágenes RGB completas.
* Aplica un algoritmo de demosaico de alta calidad.
* Conserva la máxima calidad y detalle de la imagen.

**Duración:** varía según el número de imágenes y la velocidad de la CPU.

### Etapa 3: Calibración

**Qué hace Chloros:**

* **Corrección de viñetas**: elimina el oscurecimiento de los bordes de la lente.
* **Calibración de la reflectancia**: normaliza utilizando valores de reflectancia objetivo.
* Aplica correcciones en todas las bandas/canales.
* Utiliza el objetivo de calibración adecuado para cada imagen en función de la marca de tiempo.

**Duración:** La mayor parte del tiempo de procesamiento

### Etapa 4: Cálculo del índice

**Qué hace Chloros:**

* Calcula los índices multiespectrales configurados (NDVI, NDRE, etc.)
* Aplica matemáticas de banda a las imágenes calibradas
* Genera imágenes de índice para cada índice seleccionado

**Duración:** unos segundos por imagen

### Etapa 5: Exportación

**Qué hace Chloros:**

* Guarda las imágenes calibradas en el formato seleccionado
* Exporta imágenes de índice con colores LUT configurados
* Escribe archivos en subcarpetas del modelo de cámara
* Conserva los nombres de archivo originales con sufijos.

**Duración:** varía según el formato de exportación y el tamaño del archivo.

***

## Comportamiento del procesamiento

### Canalización de procesamiento automático

Una vez iniciada, toda la canalización se ejecuta automáticamente:

* No se necesita la interacción del usuario.
* Todos los pasos configurados se ejecutan en secuencia.
* Las actualizaciones del progreso se muestran en tiempo real.

### Uso del ordenador durante el procesamiento

**Modo libre:**

* Uso relativamente bajo de la CPU (un solo subproceso).
* El ordenador sigue respondiendo a otras tareas.
* Es seguro minimizar Chloros y trabajar en otras aplicaciones.

**Chloros+ Modo paralelo:**

* Alto uso de la CPU (multisubprocesos, hasta 16 núcleos).
* Con aceleración de GPU: alto uso de la GPU.
* El ordenador puede responder menos durante el procesamiento.
* Evite iniciar otras tareas que requieran un uso intensivo de la CPU.

{% hint style=&quot;warning&quot; %}
**Consejo de rendimiento**: Para obtener el mejor rendimiento de Chloros+, cierre otras aplicaciones y deje que Chloros utilice todos los recursos del sistema.
{% endhint %}

### El procesamiento no se puede pausar

**Limitaciones importantes:**

* Una vez iniciado, el procesamiento no se puede pausar.
* Puede cancelar el procesamiento, pero se perderá el progreso.
* Los resultados parciales no se guardan.
* Si se cancela, se debe reiniciar desde el principio.

**Consejo de planificación:** Para proyectos muy grandes, considere procesar por lotes o utilizar CLI para un mejor control.

***

## Supervisión del procesamiento

Mientras se ejecuta el procesamiento, puede:

* **Ver la barra de progreso**: vea el porcentaje de finalización general.
* **Ver la etapa actual**: detectar, analizar, calibrar o exportar.
* **Comprobar la pestaña de registro**: vea mensajes y advertencias detallados del procesamiento.
* **Previsualizar imágenes completadas**: algunos archivos de exportación pueden aparecer durante el procesamiento.

Para obtener información detallada sobre la supervisión, consulte [Supervisión del procesamiento](monitoring-the-processing.md).

***

## Cancelación del procesamiento

Si necesita detener el procesamiento:

### Cómo cancelar

1. Localice el **botón Detener/Cancelar** (sustituye al botón Iniciar durante el procesamiento).
2. Haga clic en el botón Detener.
3. El procesamiento se detiene inmediatamente.
4. Los resultados parciales se descartan.

### Cuándo cancelar

**Motivos válidos para cancelar:**

* Se ha detectado que se han utilizado ajustes incorrectos.
* Se ha olvidado marcar las imágenes de destino.
* Se han importado imágenes incorrectas.
* El sistema funciona demasiado lento o no responde.

**Después de cancelar:**

* Revise y corrija cualquier problema.
* Ajuste la configuración según sea necesario.
* Reinicie el procesamiento desde el principio.
* Para obtener la mejor experiencia, cierre completamente Chloros y reinicie.

{% hint style=&quot;warning&quot; %}
**Sin resultados parciales**: la cancelación descarta todo el progreso. Chloros no guarda las imágenes procesadas parcialmente.
{% endhint %}

***

## Estimaciones del tiempo de procesamiento

El tiempo de procesamiento real varía mucho en función de:

* Número de imágenes
* Resolución de las imágenes
* Formato de entrada RAW frente a JPG
* Modo de procesamiento (Free frente a Chloros+)
* La velocidad de la CPU y el número de núcleos
* La disponibilidad de la GPU (solo Chloros+)
* El número de índices que se deben calcular
* La complejidad del formato de exportación

### Estimaciones aproximadas (Chloros+, imágenes de 12 MP, CPU moderna)

| Número de imágenes | Modo gratuito | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 imágenes   | 15-20 min | 5-8 min        | 3-5 min        |
| 100 imágenes  | 30-40 min | 10-15 min      | 5-8 min        |
| 200 imágenes  | 1-1,5 horas | 20-30 min      | 10-15 min      |
| 500 imágenes  | 2-3 horas   | 45-60 min      | 20-30 min      |
| 1000 imágenes | 4-6 horas   | 1,5-2 horas      | 40-60 min      |

{% hint style=&quot;info&quot; %}
**Primera ejecución**: el procesamiento inicial puede tardar más tiempo, ya que Chloros crea cachés y perfiles. El procesamiento posterior de conjuntos de datos similares será más rápido.
{% endhint %}

***

## Problemas comunes al inicio

### Botón de inicio desactivado (en gris)

**Posibles causas:**

* No se han importado imágenes
* El backend no se ha iniciado completamente
* El procesamiento anterior sigue en ejecución
* El proyecto no se ha cargado completamente

**Soluciones:**

1. Espere a que el backend se inicialice por completo (compruebe el icono del menú principal).
2. Compruebe que las imágenes se han importado en el explorador de archivos.
3. Reinicie Chloros si el botón sigue desactivado.
4. Compruebe si hay mensajes de error en el registro de depuración.

### El procesamiento se inicia y falla inmediatamente

**Posibles causas:**

* No hay imágenes válidas en el proyecto
* Archivos de imagen dañados
* Espacio en disco insuficiente
* Memoria insuficiente (RAM)

**Soluciones:**

1. Compruebe el registro de depuración <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> en busca de mensajes de error.
2. Compruebe el espacio disponible en disco.
3. Intente procesar un subconjunto más pequeño de imágenes.
4. Compruebe que las imágenes no estén dañadas.

### Advertencia «No se han detectado objetivos»

**Posibles causas:**

* Se ha olvidado marcar las imágenes objetivo.
* Las imágenes objetivo no contienen objetivos visibles.
* La configuración de detección de objetivos es demasiado estricta.

**Soluciones:**

1. Revise [Selección de imágenes objetivo](choosing-target-images.md).
2. Marque las imágenes adecuadas en la columna Objetivo.
3. Compruebe que los objetivos sean visibles en las imágenes marcadas.
4. Ajuste la configuración de detección de objetivos si es necesario.

***

## Consejos para un procesamiento satisfactorio

### Antes de comenzar

1. **Pruebe primero con un pequeño subconjunto**: procese entre 10 y 20 imágenes para verificar la configuración.
2. **Compruebe el espacio disponible en el disco**: asegúrese de tener libre entre 2 y 3 veces el tamaño del conjunto de datos.
3. **Cierre las aplicaciones innecesarias**: libere recursos del sistema.
4. **Verifique las imágenes objetivo**: obtenga una vista previa de los objetivos marcados para garantizar la calidad.
5. **Guarde el proyecto**: el proyecto se guarda automáticamente, pero es recomendable guardarlo manualmente.

### Durante el procesamiento

1. **Evite que el sistema entre en modo de suspensión**: desactive los modos de ahorro de energía.
2. **Mantenga Chloros en primer plano**: o, al menos, visible en la barra de tareas.
3. **Compruebe el progreso de vez en cuando**: compruebe si hay advertencias o errores.
4. **No cargue otras aplicaciones pesadas**: especialmente con el modo paralelo Chloros+.

### Aceleración de GPU Chloros+

Si utiliza la aceleración de GPU NVIDIA:

1. Actualice los controladores NVIDIA a la última versión.
2. Asegúrese de que la GPU tenga más de 4 GB de VRAM.
3. Cierre las aplicaciones que consumen muchos recursos de la GPU (juegos, edición de vídeo).
4. Supervise la temperatura de la GPU (asegúrese de que la refrigeración sea adecuada).

***

## Siguiente paso

Una vez iniciado el procesamiento:

1. **Supervise el progreso**: consulte [Supervisión del procesamiento](monitoring-the-processing.md).
2. **Espere a que finalice**: el procesamiento se ejecuta automáticamente.
3. **Revise los resultados**: consulte [Finalización del procesamiento](finishing-the-processing.md).

Para obtener información sobre qué hacer durante el procesamiento, consulte [Supervisión del procesamiento](monitoring-the-processing.md).

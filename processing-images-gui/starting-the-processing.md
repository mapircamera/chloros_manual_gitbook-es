# Inicio del procesamiento

Una vez que hayas importado tus imágenes, marcado los objetivos de calibración y configurado los ajustes del proyecto, ya estás listo para comenzar el procesamiento. Esta página te guía a lo largo del proceso de inicio del flujo de procesamiento Chloros.

## Lista de comprobación previa al procesamiento

Antes de hacer clic en el botón «Iniciar», comprueba que todo esté listo:

* [ ] **Archivos importados**: todas las imágenes aparecen en el explorador de archivos
* [ ] **Imágenes de referencia marcadas**: se ha marcado la columna «Referencia» para las imágenes de calibración (o se ha importado una grabación `.daq` para LATTICE)
* [ ] **Modelos de cámara detectados**: la columna «Modelo de cámara» muestra las cámaras correctas
* [ ] **Ajustes configurados**: se han revisado y ajustado los ajustes del proyecto
* [ ] **Índices seleccionados**: se han añadido los índices multiespectrales deseados (si es necesario)
* [ ] **Formato de exportación elegido**: formato de salida adecuado para su flujo de trabajo

{% hint style="info" %}
**Consejo**: Haz clic en algunas imágenes del explorador de archivos para comprobar que se han cargado correctamente antes de procesarlas.
{% endhint %}

***

## Inicio del procesamiento

### Localiza el botón «Iniciar»

El botón «Iniciar/Reproducir» se encuentra en la barra de encabezado superior de Chloros:

* Posición: parte superior central de la ventana
* Icono: **botón «Reproducir/Iniciar»** <img src="../.gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">
* Estado: el botón está activado (iluminado) cuando está listo para procesar

### Haz clic para iniciar

1. Haz clic en el **botón Reproducir/Iniciar** situado en la barra superior
2. El procesamiento comienza inmediatamente
3. El botón se convierte en un botón de **Detener** durante el procesamiento
4. La barra de progreso se actualiza, mostrando el estado del procesamiento

{% hint style="success" %}
**Procesamiento iniciado**: Una vez pulsado, Chloros gestiona automáticamente todos los pasos del procesamiento: detección de objetivos, debayering, calibración, cálculo del índice y exportación. Detecta automáticamente si tu proyecto es Survey3, LATTICE o una combinación de ambos, y aplica el flujo de trabajo adecuado a cada cámara.
{% endhint %}

***

## Explicación de los modos de procesamiento

Chloros funciona en dos modos de procesamiento diferentes en función de tu licencia:

### Modo gratuito (procesamiento secuencial)

**Disponible para todos los usuarios**

**Cómo funciona:**

* Procesa las imágenes de una en una, de forma secuencial
* Funcionamiento en un solo hilo
* Menor consumo de memoria

**La barra de progreso muestra dos etapas:**

1.**Detección de objetivos**: búsqueda de objetivos de calibración
2. **Procesamiento**: aplicación de la calibración y exportación de imágenes**Tiempo de procesamiento:**

* Mucho más lento que el modo paralelo de Chloros+
* Adecuado para conjuntos de datos pequeños y medianos (&lt; 200 imágenes)

### Modo Chloros+ (procesamiento paralelo)

**Requiere la licencia Chloros+**

**Cómo funciona:**

* Procesa varias imágenes simultáneamente mediante un [flujo de procesamiento de 4 subprocesos](../processing-architecture/processing-pipeline.md)
* La [adaptación dinámica de cálculo](../processing-architecture/dynamic-compute-adaptation.md) selecciona automáticamente la estrategia óptima para tu hardware al inicio de la ejecución
* Aceleración por GPU (CUDA) con tarjetas gráficas NVIDIA (de sobremesa y Jetson)
* **El número de trabajadores se adapta al hardware**: las estrategias de GPU ejecutan**entre 1 y 4 trabajadores simultáneos** (escalados según la VRAM; un Jetson con poca memoria ejecuta 1, mientras que una GPU de ordenador de sobremesa de 12 GB o más ejecuta hasta 4); los sistemas que solo utilizan la CPU ejecutan un trabajador por núcleo físico, menos uno**La barra de progreso muestra 4 etapas** (que se corresponden con los 4 subprocesos del canal):

1. **Detección** (Subproceso 1): búsqueda de objetivos de calibración
2. **Análisis** (hilo 2): examen de los metadatos de la imagen y cálculo de la calibración
3. **Calibración** (hilo 3): eliminación del efecto «bayering», corrección del viñeteado, calibración y cálculo del índice
4. **Exportación** (hilo 4): guardado de las imágenes procesadas y los índices**Interacción con la barra de progreso:*** **Pasa el ratón** por encima de la barra para ver el panel desplegable detallado de las 4 etapas
* **Haz clic** en la barra de progreso para fijar el panel desplegable en su posición
* **Vuelve a hacer clic** para desbloquearlo y ocultar el panel**Tiempo de procesamiento:**

* Significativamente más rápido que el modo gratuito
* La aceleración por GPU mejora aún más la velocidad

{% hint style="info" %}
**Chloros+ Velocidad**: El procesamiento paralelo puede ser entre 5 y 10 veces más rápido que el modo secuencial para conjuntos de datos de gran tamaño. Un proyecto de 500 imágenes que tarda 2 horas en el modo gratuito puede completarse en 15-20 minutos con Chloros+.
{% endhint %}

***

## ¿Qué ocurre durante el procesamiento?

### Etapa 1: Detección de objetivos

**Qué hace Chloros:**

* Escanea las imágenes que has marcado en la columna «Objetivo» (todas las imágenes si no hay ninguna marcada)
* Identifica los paneles de calibración en cada objetivo
* Extrae los valores de reflectancia de los paneles de objetivos
* Registra las marcas de tiempo de los objetivos para la programación de la calibración

**Duración:** 1-30 segundos (con objetivos marcados), 5-30+ minutos (sin marcar)

### Etapa 2: Desbayering (conversión RAW)

**Qué hace Chloros:**

* Convierte los datos RAW con patrón Bayer en imágenes completas de 3 canales (los módulos LATTICE mono permanecen en una sola banda; en su caso, se omite el proceso de «debayering» y se incluye una nota en el registro)
* Aplica el algoritmo de demosaico seleccionado
* Conserva la máxima calidad y detalle de la imagen

**Duración:** Varía en función del número de imágenes y de la velocidad de la CPU/GPU

### Etapa 3: Calibración

**Qué hace Chloros:*** **Corrección de viñeteado**: elimina el oscurecimiento de los bordes causado por la lente
* **Calibración de reflectancia**: normaliza utilizando valores de reflectancia de referencia y/o datos de radiación descendente del DAQ
* Aplica correcciones en todas las bandas/canales
* Utiliza la referencia de calibración adecuada para cada imagen en función de la marca de tiempo

**Duración:** La mayor parte del tiempo de procesamiento

### Etapa 4: Cálculo de índices

**Qué hace Chloros:**

* Calcula los índices multiespectrales configurados (NDVI, NDRE, etc.)
* Aplica operaciones matemáticas a las bandas de las imágenes calibradas
* Genera imágenes de índice para cada índice seleccionado

**Duración:** Unos segundos por imagen

### Etapa 5: Exportación

**Qué hace Chloros:**

* Guarda las imágenes procesadas en el formato seleccionado
* **Desglosado de LATTICE**: cada fotograma LATTICE sin procesar se exporta como todos los productos habilitados en una sola pasada: sin bayering, vista previa, radiancia (siempre float32) y reflectancia
* Escribe los archivos en el árbol de salida del proyecto: `<project>/<camera>/<format>/<Product>_Images/`
* **Mantiene el nombre del archivo de origen**: la carpeta identifica el producto, no se añade ningún sufijo**Duración:** Varía según el formato de exportación y el tamaño del archivo***

## Comportamiento del procesamiento

### Flujo de trabajo de procesamiento automático

Una vez iniciado, todo el flujo de trabajo se ejecuta automáticamente:

* No se necesita interacción del usuario
* Todos los pasos configurados se ejecutan en secuencia
* Las actualizaciones de progreso se muestran en tiempo real
* Los archivos exportados se guardan en el disco a medida que se completan; puedes abrir los resultados finales mientras continúa la ejecución

### Uso del ordenador durante el procesamiento

**Modo libre:**

* Uso de la CPU relativamente bajo (un solo subproceso)
* El ordenador sigue respondiendo para otras tareas
* Es seguro minimizar Chloros y trabajar en otras aplicaciones

**Chloros+ Modo paralelo:**

* Alto uso de la CPU en todo el grupo de trabajadores de la estrategia
* Con aceleración por GPU: alto uso de la GPU
* El ordenador puede responder con menos rapidez durante el procesamiento
* Evita iniciar otras tareas que requieran un uso intensivo de la CPU

{% hint style="warning" %}
**Consejo de rendimiento**: Para obtener el mejor rendimiento de Chloros+, cierra otras aplicaciones y deja que Chloros utilice todos los recursos del sistema.
{% endhint %}

### El procesamiento no se puede pausar (pero la detención es definitiva)

* Una vez iniciado, el procesamiento no se puede pausar para reanudarlo más tarde
* Al hacer clic en **Detener**, la ejecución se detiene de forma definitiva con el primer clic
* Los productos ya exportados antes de la detención permanecen en el disco
* Una ejecución detenida informa con precisión de lo que se ha completado (véanse las líneas `[RUN-SUMMARY]` en el registro)
* Una nueva ejecución inicia el proceso desde el principio

**Consejo de planificación:** Para proyectos muy grandes, plantéate procesar por lotes o utilizar el CLI para un mejor control.***

## Supervisión del procesamiento

Mientras se ejecutan las ejecuciones de procesamiento, puedes:

* **Observar la barra de progreso**: ver el porcentaje global de finalización
* **Ver la etapa actual**: detección, análisis, calibración o exportación
* **Consultar la pestaña de registro**: ver mensajes y advertencias detallados del procesamiento
* **Previsualizar las imágenes completadas**: los archivos de exportación aparecen en el disco durante el procesamiento

Para obtener información detallada sobre la supervisión, consulta [Supervisión del procesamiento](monitoring-the-processing.md).

***

## Detener el procesamiento

Si necesitas detener el procesamiento:

### Cómo detenerlo

1. Localiza el **botón Detener** (sustituye al botón Iniciar durante el procesamiento)
2. Haz clic una vez en él; la barra mostrará **«Deteniendo...»** mientras se termina la imagen en curso
3. La ejecución finaliza en un estado de parada definitiva y el registro muestra un informe detallado `[RUN-SUMMARY]` de lo que se ha completado

### Cuándo detener

**Motivos válidos para detener:**

* Se ha detectado que se han utilizado ajustes incorrectos
* Se han olvidado de marcar las imágenes de destino
* Se han importado imágenes erróneas
* El sistema funciona demasiado lento o no responde

**Tras detener el proceso:**

* Los productos exportados antes de la detención permanecen en el disco
* Revisa y corrige cualquier problema; ajusta la configuración según sea necesario
* Reinicia el procesamiento: la ejecución comienza desde el principio

***

## Estimaciones del tiempo de procesamiento

El tiempo real de procesamiento varía considerablemente en función de:

* El número de imágenes
* La resolución de las imágenes
* El formato de entrada (RAW o JPG)
* El modo de procesamiento (Free o Chloros+)
* La velocidad de la CPU y el número de núcleos
* Disponibilidad de la GPU (solo Chloros+)
* Número de índices que calcular
* Número de productos de exportación habilitados (LATTICE)

### Estimaciones aproximadas (Chloros+, imágenes de 12 MP, CPU moderna)

| Número de imágenes | Modo gratuito | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 imágenes   | 15-20 min | 5-8 min        | 3-5 min        |
| 100 imágenes  | 30-40 min | 10-15 min      | 5-8 min        |
| 200 imágenes  | 1-1,5 h | 20-30 min      | 10-15 min      |
| 500 imágenes  | 2-3 horas   | 45-60 min      | 20-30 min      |
| 1.000 imágenes | 4-6 horas   | 1,5-2 horas      | 40-60 min      |

{% hint style="info" %}
**Primera ejecución**: El procesamiento inicial puede tardar más tiempo, ya que Chloros crea cachés y perfiles. El procesamiento posterior de conjuntos de datos similares será más rápido.
{% endhint %}

***

## Problemas habituales al iniciar

### Botón de inicio desactivado (en gris)

**Posibles causas:**

* No se han importado imágenes
* El backend no se ha iniciado por completo
* El procesamiento anterior sigue en curso
* El proyecto no se ha cargado por completo

**Soluciones:**

1. Espera a que el backend se inicialice por completo (comprueba el icono del menú principal)
2. Comprueba que las imágenes se hayan importado en el explorador de archivos
3. Reinicia Chloros si el botón sigue desactivado
4. Comprueba el registro de depuración en busca de mensajes de error

### El procesamiento se inicia y falla inmediatamente

**Posibles causas:**

* No hay imágenes válidas en el proyecto
* Archivos de imagen dañados
* Espacio en disco insuficiente
* Memoria insuficiente (RAM)

**Soluciones:**

1. Comprueba el registro de depuración <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> en busca de mensajes de error
2. Comprueba el espacio disponible en disco
3. Intenta procesar un subconjunto más pequeño de imágenes
4. Comprueba que las imágenes no estén dañadas

### La ejecución finaliza, pero no se guardan imágenes

Una ejecución que haya solicitado productos de imagen pero no haya guardado ninguno se considera un **error, no un éxito**; el código Chloros lo indica claramente:

* El registro de la interfaz gráfica muestra mensajes `[RUN-SUMMARY]` que indican la causa probable: no se han importado imágenes, no se ha detectado ningún destino o todos los productos solicitados se han omitido por no ser aplicables (por ejemplo, solicitar radiancia/reflectancia de cámaras que solo admiten RGB)
* El equivalente a CLI (`chloros-cli process`) muestra `Processing finished but wrote no image products.` y **termina con un valor distinto de cero**, por lo que los scripts pueden detectarlo
* Una ejecución deliberada solo con metadatos (todos los productos de exportación desactivados, sin índices) sigue considerándose un éxito

Consulta [la Referencia de CLI](../reference/cli-reference.md#a-run-that-writes-no-images-fails) para conocer la semántica completa.

### Advertencia «No se han detectado objetivos»

**Posibles causas:**

* Se ha olvidado marcar las imágenes objetivo
* Las imágenes objetivo no contienen objetivos visibles
* La configuración de detección de objetivos es demasiado estricta

**Soluciones:**

1. Revisa [Cómo elegir imágenes objetivo](choosing-target-images.md)
2. Marca las imágenes adecuadas en la columna «Objetivo»
3. Comprueba que los objetivos sean visibles en las imágenes marcadas
4. Ajusta la configuración de detección de objetivos si es necesario

***

## Consejos para un procesamiento satisfactorio

### Antes de empezar

1. **Prueba primero con un pequeño subconjunto**: procesa entre 10 y 20 imágenes para verificar la configuración
2. **Comprueba el espacio disponible en disco**: asegúrate de que haya espacio libre equivalente a 2 o 3 veces el tamaño del conjunto de datos (más si están habilitados todos los productos LATTICE)
3. **Cierra las aplicaciones innecesarias**: libera recursos del sistema
4. **Comprueba las imágenes de los objetivos**: previsualiza los objetivos marcados para garantizar la calidad
5. **Guarda el proyecto**: el proyecto se guarda automáticamente, pero es recomendable guardarlo manualmente

### Durante el procesamiento

1. **Evita que el sistema entre en modo de suspensión**: desactiva los modos de ahorro de energía
2. **Mantén Chloros en primer plano**: o, al menos, visible en la barra de tareas
3. **Supervisa el progreso de vez en cuando**: comprueba si hay advertencias o errores
4. **No ejecutes otras aplicaciones que consuman muchos recursos**: especialmente con el modo paralelo de Chloros+

### Aceleración por GPU de Chloros+

Si utilizas la aceleración por GPU de NVIDIA:

1. Actualiza los controladores de NVIDIA a la última versión
2. Asegúrate de que la GPU tenga 4 GB o más de VRAM (7 GB o más para el «debayering» simultáneo con reconocimiento de texturas)
3. Cierra las aplicaciones que consuman muchos recursos de la GPU (juegos, edición de vídeo)
4. Supervisa la temperatura de la GPU (asegúrate de que la refrigeración sea adecuada)

***

## Próximos pasos

Una vez que haya comenzado el procesamiento:

1. **Supervisa el progreso**: consulta [Supervisión del procesamiento](monitoring-the-processing.md)
2. **Espera a que finalice**: el procesamiento se ejecuta automáticamente
3. **Revisa los resultados**: consulta [Finalización del procesamiento](finishing-the-processing.md)

Para obtener información sobre qué hacer durante el procesamiento, consulta [Supervisión del procesamiento](monitoring-the-processing.md).

# Inicio del procesamiento

Una vez que hayas importado tus imágenes, marcado tus objetivos de calibración y configurado los ajustes del proyecto, estarás listo para comenzar el procesamiento. Esta página te guía a través del inicio del proceso de procesamiento de Chloros.

## Lista de comprobación previa al procesamiento

Antes de hacer clic en el botón «Iniciar», comprueba que todo esté listo:

* [ ] **Archivos importados**: todas las imágenes aparecen en el explorador de archivos
* [ ] **Imágenes de referencia marcadas**: se ha marcado la columna «Referencia» para las imágenes de calibración (o se ha importado un archivo `.daq` para LATTICE)
* [ ] **Modelos de cámara detectados**: la columna «Modelo de cámara» muestra las cámaras correctas
* [ ] **Configuración establecida**: se han revisado y ajustado los ajustes del proyecto
* [ ] **Índices seleccionados**: se han añadido los índices multiespectrales deseados (si es necesario)
* [ ] **Formato de exportación elegido**: formato de salida adecuado para su flujo de trabajo

{% hint style="info" %}
**Consejo**: Haga clic en algunas imágenes del explorador de archivos para comprobar que se han cargado correctamente antes de procesarlas.
{% endhint %}

***

## Inicio del procesamiento

### Localiza el botón «Iniciar»

El botón «Iniciar/Reproducir» se encuentra en la barra superior de Chloros:

* Posición: parte superior central de la ventana
* Icono: **botón «Reproducir/Iniciar»** <img src="../.gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">
* Estado: el botón está activado (iluminado) cuando está listo para procesar

### Haz clic para iniciar

1. Haz clic en el **botón Reproducir/Iniciar** de la barra superior
2. El procesamiento comienza inmediatamente
3. El botón se convierte en un botón **Detener** durante el procesamiento
4. La barra de progreso se actualiza, mostrando el estado del procesamiento

{% hint style="success" %}
**Procesamiento iniciado**: Una vez pulsado, Chloros se encarga automáticamente de todos los pasos del procesamiento: detección de objetivos, debayering, calibración, cálculo de índices y exportación. Detecta automáticamente si tu proyecto es Survey3, LATTICE o una combinación de ambos, y aplica el flujo de trabajo adecuado a cada cámara.
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

**Requiere una licencia de Chloros+**

**Cómo funciona:**

* Procesa varias imágenes simultáneamente mediante un [flujo de procesamiento de 4 subprocesos](../processing-architecture/processing-pipeline.md)
* La [adaptación dinámica de cálculo](../processing-architecture/dynamic-compute-adaptation.md) selecciona automáticamente la estrategia óptima para tu hardware al inicio de la ejecución
* Aceleración por GPU (CUDA) con tarjetas gráficas NVIDIA
* **El número de trabajadores se adapta al hardware**: las estrategias de GPU ejecutan**entre 2 y 4 trabajadores simultáneos** (escalados según la VRAM: 3 en una GPU de 2-12 GB, y de 2 a 4 en una GPU de 12 GB o más); los sistemas que solo utilizan CPU ejecutan un trabajador por núcleo físico, menos uno**La barra de progreso muestra 4 etapas** (que se corresponden con los 4 subprocesos de la canalización):

1. **Detección** (Subproceso 1): búsqueda de objetivos de calibración
2. **Análisis** (hilo 2): examen de los metadatos de la imagen y cálculo de la calibración
3. **Calibración** (hilo 3): eliminación del efecto «bayering», corrección del viñeteado, calibración y cálculo del índice
4. **Exportación** (hilo 4): guardado de las imágenes procesadas y los índices**Interacción con la barra de progreso:*** **Pasa el ratón** por encima de la barra para ver el panel desplegable detallado de las 4 etapas
* **Haz clic** en la barra de progreso para fijar el panel desplegable en su posición
* **Vuelve a hacer clic** para desbloquearlo y ocultar el panel**Tiempo de procesamiento:**

* Significativamente más rápido que el modo gratuito
* La aceleración por GPU mejora aún más la velocidad

{% hint style="info" %}
**Velocidad de Chloros+**: El procesamiento paralelo puede ser entre 5 y 10 veces más rápido que el modo secuencial para conjuntos de datos grandes. Un proyecto de 500 imágenes que tarda 2 horas en el modo gratuito puede completarse en 15-20 minutos con Chloros+.
{% endhint %}

***

## ¿Qué ocurre durante el procesamiento?

### Fase 1: Detección de objetivos

**Qué hace Chloros:**

* Analiza las imágenes que has marcado en la columna «Objetivo» (todas las imágenes si no hay ninguna marcada)
* Identifica los paneles de calibración en cada objetivo
* Extrae los valores de reflectancia de los paneles objetivo
* Registra las marcas de tiempo de los objetivos para la programación de la calibración

**Duración:** de 1 a 30 segundos (con objetivos marcados), de 5 a 30 minutos o más (sin marcar)

### Etapa 2: Desbayering (conversión RAW)

**Qué hace Chloros:**

* Convierte los datos RAW con patrón Bayer en imágenes completas de 3 canales (los módulos mono LATTICE permanecen en una sola banda; en su caso, se omite el proceso de «debayering» y se incluye una nota en el registro)
* Aplica el algoritmo de demosaico seleccionado
* Conserva la máxima calidad de imagen y el máximo detalle

**Duración:** Varía en función del número de imágenes y de la velocidad de la CPU/GPU

### Etapa 3: Calibración

**Qué hace Chloros:*** **Corrección de viñeteado**: Elimina el oscurecimiento de los bordes causado por la lente
* **Calibración de la reflectancia**: Normaliza utilizando valores de reflectancia de referencia y/o datos de radiación descendente del sistema de adquisición de datos (DAQ)
* Aplica correcciones en todas las bandas/canales
* Utiliza la referencia de calibración adecuada para cada imagen en función de la marca de tiempo

**Duración:** La mayor parte del tiempo de procesamiento

### Etapa 4: Cálculo de índices

**Qué hace Chloros:**

* Calcula los índices multiespectrales configurados (NDVI, NDRE, etc.)
* Aplica operaciones matemáticas entre bandas a las imágenes calibradas
* Genera imágenes de índice para cada índice seleccionado

**Duración:** Unos segundos por imagen

### Etapa 5: Exportación

**Qué hace Chloros:**

* Guarda las imágenes procesadas en el formato seleccionado
* **LATTICE fan-out**: cada fotograma LATTICE sin procesar se exporta como todos los productos habilitados en una sola pasada: sin bayering, vista previa, radiancia (siempre float32) y reflectancia
* Guarda los archivos en el árbol de salida del proyecto: `<project>/<camera>/<format>/<Product>_Images/`
* **Mantiene el nombre del archivo original**: la carpeta identifica el producto, no se añade ningún sufijo**Duración:** Varía según el formato de exportación y el tamaño del archivo***

## Comportamiento del procesamiento

### Flujo de procesamiento automático

Una vez iniciado, todo el flujo se ejecuta automáticamente:

* No se necesita interacción por parte del usuario
* Todos los pasos configurados se ejecutan en secuencia
* Las actualizaciones de progreso se muestran en tiempo real
* Los archivos exportados se guardan en el disco a medida que se completan; puedes abrir los resultados finales mientras la ejecución continúa

### Uso del ordenador durante el procesamiento

**Modo libre:**

* Uso de la CPU relativamente bajo (un solo subproceso)
* El ordenador sigue respondiendo para otras tareas
* Es seguro minimizar Chloros y trabajar en otras aplicaciones

**Modo paralelo de Chloros+:**

* Alto uso de la CPU en todo el grupo de trabajadores de la estrategia
* Con aceleración por GPU: alto uso de la GPU
* El ordenador puede responder con menos rapidez durante el procesamiento
* Evita iniciar otras tareas que requieran un uso intensivo de la CPU

{% hint style="warning" %}
**Consejo de rendimiento**: Para obtener el mejor rendimiento de Chloros+, cierra otras aplicaciones y deja que Chloros utilice todos los recursos del sistema.
{% endhint %}

### El procesamiento no se puede pausar (pero la detención es limpia)

* Una vez iniciado, el procesamiento no se puede pausar para reanudarlo más tarde
* Al hacer clic en **Detener**, la ejecución se detiene de forma definitiva con el primer clic
* Los productos ya exportados antes de la detención permanecen en el disco
* Una ejecución detenida informa con precisión de lo que ha completado (consulta las líneas `[RUN-SUMMARY]` en el registro)
* Una nueva ejecución inicia el proceso desde el principio

**Consejo de planificación:** Para proyectos muy grandes, plantéate procesar por lotes o utilizar la CLI para un mayor control.***

## Supervisión del procesamiento

Mientras se ejecuta el procesamiento, puedes:

* **Observar la barra de progreso**: ver el porcentaje global de finalización
* **Ver la etapa actual**: detección, análisis, calibración o exportación
* **Consultar la pestaña de registro**: ver mensajes y advertencias detallados del procesamiento
* **Previsualizar las imágenes completadas**: los archivos de exportación aparecen en el disco durante el procesamiento

Para obtener información detallada sobre la supervisión, consulta [Supervisión del procesamiento](monitoring-the-processing.md).

***

## Detener el procesamiento

Si necesitas detener el procesamiento:

### Cómo detenerlo

1. Localiza el **botón Detener** (sustituye al botón «Iniciar» durante el procesamiento)
2. Haz clic en él una vez; la barra mostrará **«Deteniéndose...»** mientras se completa la imagen en curso
3. La ejecución finaliza en un estado de parada definitiva y el registro muestra un informe detallado `[RUN-SUMMARY]` de lo que se ha completado

### Cuándo detener el procesamiento

**Motivos válidos para detener el procesamiento:**

* Te has dado cuenta de que se han utilizado ajustes incorrectos
* Te has olvidado de marcar las imágenes de destino
* Se han importado imágenes equivocadas
* El sistema funciona demasiado lento o no responde

**Tras detener el procesamiento:**

* Los productos exportados antes de la detención permanecen en el disco
* Revisa y corrige cualquier problema; ajusta la configuración según sea necesario
* Reinicia el procesamiento: la ejecución comienza desde el principio

***

## Estimaciones del tiempo de procesamiento

El tiempo real de procesamiento varía considerablemente en función de:

* El número de imágenes
* La resolución de las imágenes
* El formato de entrada (RAW o JPG)
* El modo de procesamiento (Gratuito frente a Chloros+)
* Velocidad de la CPU y número de núcleos
* Disponibilidad de la GPU (solo Chloros+)
* Número de índices que calcular
* Número de productos de exportación habilitados (LATTICE)

### Estimaciones aproximadas (Chloros+, imágenes de 12 MP, CPU moderna)

| Número de imágenes | Modo gratuito | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 imágenes   | 15-20 min | 5-8 min        | 3-5 min        |
| 100 imágenes  | 30-40 min | 10-15 min      | 5-8 min        |
| 200 imágenes  | 1-1,5 horas | 20-30 min      | 10-15 min      |
| 500 imágenes  | 2-3 horas   | 45-60 min      | 20-30 min      |
| 1 000 imágenes | 4-6 horas   | 1,5-2 horas      | 40-60 min      |

{% hint style="info" %}
**Primera ejecución**: El procesamiento inicial puede tardar más tiempo, ya que Chloros crea cachés y perfiles. El procesamiento posterior de conjuntos de datos similares será más rápido.
{% endhint %}

***

## Problemas habituales al inicio

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
4. Consulta el registro de depuración para ver si hay mensajes de error

### El procesamiento se inicia pero falla inmediatamente

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

### La ejecución finaliza, pero no se escriben imágenes

Una ejecución que haya solicitado productos de imagen pero no haya escrito ninguno se considera un **fallo, no un éxito**; Chloros lo indica claramente:

* El registro de la interfaz gráfica de usuario muestra mensajes `[RUN-SUMMARY]` que indican la causa probable: no se han importado imágenes, no se ha detectado ningún objetivo o se han omitido todos los productos solicitados por no ser aplicables (por ejemplo, solicitar radiancia/reflectancia de cámaras que solo captan RGB)
* El equivalente en la CLI (`chloros-cli process`) muestra el mensaje `Processing finished but wrote no image products.` y **sale con un valor distinto de cero**, por lo que los scripts pueden detectarlo
* Una ejecución deliberada solo con metadatos (todos los productos de exportación desactivados, sin índices) sigue considerándose un éxito

Consulte [la Referencia de la CLI](../reference/cli-reference.md#a-run-that-writes-no-images-fails) para conocer la semántica completa.

### La carpeta del proyecto está demasiado anidada para este ordenador

**Solo Windows, novedad en la versión 1.3.0.** Windows viene con las rutas largas desactivadas, por lo que ninguna ruta de archivo puede superar los 260 caracteres. Los productos de Chloros se ubican hasta unos 130 caracteres por debajo de la carpeta del proyecto, por lo que una carpeta de proyecto más profunda que eso no podría recibir sus productos. En lugar de ejecutarse y no escribir nada, Chloros ahora comprueba antes de que comience el procesamiento:

* Al hacer clic en «Iniciar» no se inicia la ejecución: la barra de progreso muestra el motivo y el registro de depuración muestra `[PROCESS] refused: The project folder is too deep for this computer…`. El mensaje indica la longitud de la carpeta del proyecto y la carpeta de proyecto más larga que cabe.
* También se rechaza la creación o apertura de un proyecto en una carpeta demasiado profunda, incluso para los propios archivos del proyecto. Un cuadro de mensaje explica el motivo, indicando la carpeta de proyecto más larga que cabe y cómo habilitar las rutas largas, y no se crea ni se abre nada; el registro de depuración lo registra como `[PROJECT] refused: …`. Duplicar un proyecto con un nombre más largo también puede superar el límite: la copia se realiza, pero no se abre, y el mismo mensaje explica el motivo.

**Soluciones:**

1. Mantén los proyectos en una carpeta más corta: cambia el**directorio de trabajo** en [Configuración del proyecto](../project-settings/project-settings.md) y vuelve a crear el proyecto, o
2. Activa las rutas largas de Windows y reinicia Chloros: vuelve a ejecutar el instalador de Chloros con la opción **Habilitar rutas largas de Windows** marcada (está marcada por defecto), o utiliza la Política de grupo: *Configuración del equipo &gt; Plantillas administrativas &gt; Sistema &gt; Sistema de archivos &gt; Habilitar rutas largas de Win32*.

### Advertencia «No se han detectado objetivos»

**Posibles causas:**

* Se ha olvidado de marcar las imágenes de destino
* Las imágenes de destino no contienen objetivos visibles
* La configuración de detección de objetivos es demasiado estricta

**Soluciones:**

1. Revise [Selección de imágenes de destino](choosing-target-images.md)
2. Marca las imágenes adecuadas en la columna «Objetivo»
3. Comprueba que los objetivos sean visibles en las imágenes marcadas
4. Ajusta la configuración de detección de objetivos si es necesario

***

## Consejos para un procesamiento satisfactorio

### Antes de empezar

1. **Realiza primero una prueba con un pequeño subconjunto**: procesa entre 10 y 20 imágenes para verificar la configuración
2. **Comprueba el espacio disponible en disco**: asegúrate de que haya espacio libre equivalente a 2-3 veces el tamaño del conjunto de datos (más si están habilitados todos los productos LATTICE)
3. **Cierra las aplicaciones innecesarias**: libera recursos del sistema.
4. **Comprueba las imágenes de los objetivos**: previsualiza los objetivos marcados para garantizar la calidad.
5. **Guarda el proyecto**: el proyecto se guarda automáticamente, pero es recomendable guardarlo manualmente.

### Durante el procesamiento

1. **Evita que el sistema entre en suspensión**: desactiva los modos de ahorro de energía
2. **Mantén Chloros en primer plano**: o, al menos, visible en la barra de tareas
3. **Supervisa el progreso de vez en cuando**: comprueba si hay advertencias o errores
4. **No cargues otras aplicaciones pesadas**: especialmente con el modo paralelo de Chloros+

### Aceleración por GPU de Chloros+

Si utilizas la aceleración por GPU de NVIDIA:

1. Actualiza los controladores de NVIDIA a la última versión
2. Asegúrate de que la GPU tenga 4 GB o más de VRAM (7 GB o más para el «debayering» simultáneo con reconocimiento de texturas)
3. Cierra las aplicaciones que consuman muchos recursos de la GPU (juegos, edición de vídeo)
4. Supervisa la temperatura de la GPU (asegúrate de que la refrigeración sea adecuada)

***

## Próximos pasos

Una vez iniciado el procesamiento:

1. **Supervisa el progreso** - Consulta [Supervisión del procesamiento](monitoring-the-processing.md)
2. **Espera a que finalice** - El procesamiento se ejecuta automáticamente
3. **Revisa los resultados**: consulta [Finalización del procesamiento](finishing-the-processing.md)

Para obtener información sobre qué hacer durante el procesamiento, consulta [Supervisión del procesamiento](monitoring-the-processing.md).

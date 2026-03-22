# Supervisión del procesamiento

Una vez iniciado el procesamiento, Chloros ofrece varias formas de supervisar el progreso, detectar posibles problemas y comprender qué está sucediendo con su conjunto de datos. En esta página se explica cómo realizar un seguimiento del procesamiento e interpretar la información que proporciona Chloros.

## Descripción general de la barra de progreso

La barra de progreso situada en la parte superior de la página muestra el estado del procesamiento en tiempo real y el porcentaje de finalización.

### Barra de progreso del modo gratuito

Para usuarios sin licencia de Chloros+:

**Visualización del progreso en dos etapas:**

1.**Detección de objetivos**: búsqueda de objetivos de calibración en las imágenes
2. **Procesamiento**: aplicación de correcciones y exportación**La barra de progreso muestra:**

* Porcentaje de finalización total (0-100 %)
* Nombre de la etapa actual
* Visualización sencilla mediante una barra horizontal

### Barra de progreso de Chloros+

Para usuarios con licencia de Chloros+:

**Visualización del progreso en 4 etapas:**

1.**Detección**: búsqueda de objetivos de calibración
2. **Análisis**: examen de imágenes y preparación del proceso
3. **Calibración**: aplicación de correcciones de viñeteado y reflectancia
4. **Exportación**: guardado de archivos procesados**Funciones interactivas:*** **Pase el cursor por encima** de la barra de progreso para ver el panel ampliado de 4 etapas
* **Haga clic** en la barra de progreso para congelar/fijar el panel ampliado
* **Vuelva a hacer clic** para descongelarlo y que se oculte automáticamente al alejar el ratón
* Cada etapa muestra el progreso individual (0-100 %)

***

## Explicación de cada etapa del procesamiento

{% hint style="info" %}
**Arquitectura de canalización**: Estas 4 etapas de la interfaz gráfica de usuario corresponden a la [canalización de procesamiento de 4 subprocesos](../processing-architecture/processing-pipeline.md). En sistemas con aceleración por GPU, el subproceso 3 (Calibración) se beneficia de la [Adaptación dinámica de cálculo](../processing-architecture/dynamic-compute-adaptation.md), que optimiza el procesamiento para su hardware específico.
{% endhint %}

### Etapa 1: Detección (Detección de objetivos)

**Qué ocurre:**

* Chloros escanea las imágenes marcadas con la casilla de verificación «Objetivo»
* Los algoritmos de visión artificial identifican los 4 paneles de calibración
* Se extraen los valores de reflectancia de cada panel
* Se registran las marcas de tiempo de los objetivos para programar correctamente la calibración

**Duración:**

* Con objetivos marcados: 10-60 segundos
* Sin objetivos marcados: 5-30+ minutos (escanea todas las imágenes)

**Indicador de progreso:**

* Detectando: 0 % → 100 %
* Número de imágenes escaneadas
* Recuento de objetivos encontrados

**A tener en cuenta:**

* Debería completarse rápidamente si los objetivos están correctamente marcados
* Si tarda demasiado, es posible que los objetivos no estén marcados
* Compruebe el registro de depuración en busca de mensajes de «Objetivo encontrado»

### Etapa 2: Análisis

**Qué está sucediendo:**

* Lectura de los metadatos EXIF de la imagen (marcas de tiempo, ajustes de exposición)
* Determinación de la estrategia de calibración basada en las marcas de tiempo de los objetivos
* Organización de la cola de procesamiento de imágenes
* Preparación de los trabajadores de procesamiento paralelo (solo Chloros+)

**Duración:** 5-30 segundos**Indicador de progreso:**

* Analizando: 0 % → 100 %
* Etapa rápida, suele completarse rápidamente

**A tener en cuenta:**

* Debe avanzar de forma constante sin pausas
* Las advertencias sobre metadatos que faltan aparecerán en el registro de depuración

### Etapa 3: Calibración

**Qué está sucediendo:*** **Debayering**: Conversión del patrón RAW de Bayer a 3 canales
* **Corrección de viñeteado**: eliminación del oscurecimiento de los bordes de la lente
* **Calibración de reflectancia**: normalización con valores objetivo
* **Cálculo de índices**: cálculo de índices multiespectrales
* Procesamiento de cada imagen a través de todo el flujo de trabajo

**Duración:** la mayor parte del tiempo total de procesamiento (60-80 %)**Indicador de progreso:**

* Calibrando: 0 % → 100 %
* Imagen actual en proceso
* Imágenes completadas / Total de imágenes

**Comportamiento del procesamiento:*** **Modo libre**: procesa una imagen cada vez de forma secuencial
* **Modo Chloros+**: procesa hasta 16 imágenes simultáneamente
* **Aceleración por GPU**: Acelera significativamente esta etapa**A tener en cuenta:**

* Progreso constante en el recuento de imágenes
* Compruebe el registro de depuración para ver los mensajes de finalización de cada imagen
* Advertencias sobre problemas de calidad de imagen o calibración

### Etapa 4: Exportación

**Qué ocurre:**

* Escritura de imágenes calibradas en el disco en el formato seleccionado
* Exportación de imágenes de índice multiespectral con colores LUT
* Creación de subcarpetas de modelos de cámara
* Conservación de los nombres de archivo originales con los sufijos adecuados

**Duración:** 10-20 % del tiempo total de procesamiento**Indicador de progreso:**

* Exportación: 0 % → 100 %
* Archivos en proceso de escritura
* Formato de exportación y destino

**A qué prestar atención:**

* Advertencias de espacio en disco
* Errores de escritura de archivos
* Finalización de todas las salidas configuradas

***

## Pestaña «Registro de depuración»

El registro de depuración proporciona información detallada sobre el progreso del procesamiento y cualquier problema encontrado.

### Acceso al registro de depuración

1. Haga clic en el icono **Registro de depuración** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> en la barra lateral izquierda
2. Se abre el panel de registro mostrando mensajes de procesamiento en tiempo real
3. Se desplaza automáticamente para mostrar los mensajes más recientes

### Interpretación de los mensajes de registro

#### Mensajes informativos (blanco/gris)

Actualizaciones normales del procesamiento:

```
[INFO] Processing started
[INFO] Target detected in IMG_0015.RAW - 4 panels found
[INFO] Calibrating IMG_0234.RAW
[INFO] Exported NDVI image: IMG_0234_NDVI.tif
[INFO] Processing complete
```

#### Mensajes de advertencia (amarillo)

Problemas no críticos que no detienen el procesamiento:

```
[WARN] No GPS data found in IMG_0145.RAW
[WARN] Target image timestamp gap > 30 minutes
[WARN] Low contrast in calibration panel - results may vary
```

**Acción:** Revise las advertencias tras el procesamiento, pero no lo interrumpa

#### Mensajes de error (Red)

Problemas críticos que pueden provocar que el procesamiento falle:

```
[ERROR] Cannot write file - disk full
[ERROR] Corrupted image file: IMG_0299.RAW
[ERROR] No targets detected - enable reflectance calibration or mark target images
```

**Acción:** Detener el procesamiento, resolver el error y reiniciar

### Mensajes comunes del registro

| Mensaje                          | Significado                                | Acción necesaria                                         |
| -------------------------------- | -------------------------------------- | ----------------------------------------------------- |
| «Objetivo detectado en \[nombre de archivo]» | Objetivo de calibración encontrado correctamente  | Ninguna - normal                                         |
| «Procesando imagen X de Y»        | Actualización del progreso actual                | Ninguna - normal                                         |
| «No se han encontrado objetivos»               | No se han detectado objetivos de calibración        | Marcar imágenes objetivo o desactivar la calibración de reflectancia |
| «Espacio en disco insuficiente»        | No hay suficiente almacenamiento para la salida          | Libere espacio en disco                                    |
| «Omitiendo archivo dañado»        | El archivo de imagen está dañado                  | Vuelva a copiar el archivo desde la tarjeta SD                             |
| «Datos PPK aplicados»               | Correcciones GPS del archivo .daq aplicadas | Ninguna - normal                                         |

### Copia de datos de registro

Para copiar el registro con fines de resolución de problemas o asistencia técnica:

1. Abra el panel de registro de depuración
2. Haga clic en el botón **«Copiar registro»** (o haga clic con el botón derecho → Seleccionar todo)
3. Péguelo en un archivo de texto o en un correo electrónico
4. Envíelo al servicio de asistencia de MAPIR si es necesario

***

## Supervisión de recursos del sistema

### Uso de la CPU

**Modo libre:**

* 1 núcleo de CPU al ~100 %
* Otros núcleos inactivos o disponibles
* El sistema sigue respondiendo

**Modo paralelo Chloros+:**

* Varios núcleos al 80-100 % (hasta 16 núcleos)
* Alta utilización general de la CPU
* El sistema puede parecer menos receptivo

**Para supervisar:**

* Administrador de tareas (Ctrl+Mayús+Esc)
* Pestaña Rendimiento → sección CPU
* Busque los procesos «Chloros» o «chloros-backend»

### Uso de memoria (RAM)

**Uso habitual:**

* Proyectos pequeños (&lt; 100 imágenes): 2-4 GB
* Proyectos medianos (100-500 imágenes): 4-8 GB
* Proyectos grandes (más de 500 imágenes): 8-16 GB
* El modo paralelo de Chloros+ consume más RAM

**Si la memoria es escasa:**

* Procese lotes más pequeños
* Cierre otras aplicaciones
* Amplíe la RAM si procesa habitualmente conjuntos de datos grandes

### Uso de la GPU (Chloros+ con CUDA)

Cuando la aceleración por GPU está habilitada:

* La GPU NVIDIA muestra una alta utilización (60-90 %)
* El uso de VRAM aumenta (requiere 4 GB o más de VRAM)
* La fase de calibración es significativamente más rápida

**Para supervisar:**

* Icono de NVIDIA en la bandeja del sistema
* Administrador de tareas → Rendimiento → GPU
* GPU-Z o una herramienta de supervisión similar

### E/S de disco

**Qué esperar:**

* Alta lectura de disco durante la fase de análisis
* Alta escritura de disco durante la fase de exportación
* El SSD es significativamente más rápido que el HDD

**Consejo de rendimiento:**

* Utilice un SSD para la carpeta del proyecto siempre que sea posible
* Evite las unidades de red para conjuntos de datos de gran tamaño
* Asegúrese de que el disco no esté casi lleno (afecta a la velocidad de escritura)

***

## Detección de problemas durante el procesamiento

### Señales de advertencia

**El progreso se detiene (sin cambios durante más de 5 minutos):**

* Compruebe el registro de depuración en busca de errores
* Verifique el espacio disponible en disco
* Compruebe el Administrador de tareas para asegurarse de que Chloros se está ejecutando

**Aparecen mensajes de error con frecuencia:**

* Detenga el procesamiento y revise los errores
* Causas comunes: espacio en disco, archivos dañados, problemas de memoria
* Consulte la sección de resolución de problemas más abajo

**El sistema deja de responder:**

* El modo paralelo de Chloros+ consume demasiados recursos
* Considere reducir las tareas simultáneas o actualizar el hardware
* El modo libre consume menos recursos

### Cuándo detener el procesamiento

Detenga el procesamiento si observa:

* ❌ Errores de «Disco lleno» o «No se puede escribir el archivo»
* ❌ Errores repetidos de corrupción de archivos de imagen
* ❌ Sistema completamente bloqueado (no responde)
* ❌ Se ha detectado que se han configurado ajustes incorrectos
* ❌ Se han importado imágenes incorrectas

**Cómo detenerlo:**

1. Haga clic en el**botón Detener/Cancelar** (sustituye al botón Iniciar)
2. El procesamiento se detiene y se pierde el progreso
3. Solucione los problemas y reinicie desde el principio

***

## Solución de problemas durante el procesamiento

### El procesamiento es muy lento

**Posibles causas:**

* Imágenes de destino sin marcar (se están escaneando todas las imágenes)
* Almacenamiento en HDD en lugar de SSD
* Recursos del sistema insuficientes
* Se han configurado muchos índices
* Acceso a una unidad de red

**Soluciones:**

1. Si acaba de iniciarse y se encuentra en la fase de detección: Cancelar, marcar los objetivos, reiniciar
2. Para el futuro: Utilizar SSD, reducir los índices, actualizar el hardware
3. Considerar CLI para el procesamiento por lotes de grandes conjuntos de datos

### Advertencias de «Espacio en disco»

**Soluciones:**

1. Liberar espacio en disco inmediatamente
2. Mover el proyecto a una unidad con más espacio
3. Reducir el número de índices a exportar
4. Utilizar el formato JPG en lugar de TIFF (archivos más pequeños)

### Mensajes frecuentes de «archivo dañado»

**Soluciones:**

1. Volver a copiar las imágenes desde la tarjeta SD para garantizar su integridad
2. Comprobar si la tarjeta SD tiene errores
3. Eliminar los archivos dañados del proyecto
4. Continuar procesando las imágenes restantes

### Sobrecalentamiento del sistema / Limitación de rendimiento

**Soluciones:**

1. Asegurarse de que haya una ventilación adecuada
2. Limpiar el polvo de las rejillas de ventilación del ordenador
3. Reducir la carga de procesamiento (utilizar el modo Free en lugar de Chloros+)
4. Procesar durante las horas más frescas del día

***

## Notificación de procesamiento completado

Cuando finaliza el procesamiento:

* La barra de progreso alcanza el 100 %
* Aparece el mensaje **«Procesamiento completado»** en el registro de depuración
* El botón de inicio vuelve a estar habilitado
* Todos los archivos de salida se encuentran en la subcarpeta del modelo de cámara

***

## Próximos pasos

Una vez completado el procesamiento:

1. **Revisar los resultados** - Consulte [Finalización del procesamiento](finishing-the-processing.md)
2. **Compruebe la carpeta de salida**: verifique que todos los archivos se hayan exportado correctamente
3. **Revise el registro de depuración**: compruebe si hay advertencias o errores
4. **Previsualice las imágenes procesadas**: utilice el visor de imágenes o un software externo

Para obtener información sobre cómo revisar y utilizar los resultados procesados, consulte [Finalización del procesamiento](finishing-the-processing.md).

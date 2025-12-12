# Supervisión del procesamiento

Una vez iniciado el procesamiento, Chloros ofrece varias formas de supervisar el progreso, comprobar si hay problemas y comprender qué está sucediendo con su conjunto de datos. En esta página se explica cómo realizar un seguimiento del procesamiento e interpretar la información que proporciona Chloros.

## Descripción general de la barra de progreso

La barra de progreso situada en la parte superior del encabezado muestra el estado del procesamiento en tiempo real y el porcentaje completado.

### Barra de progreso en modo gratuito

Para usuarios sin licencia Chloros+:

**Visualización del progreso en dos etapas:**

1. **Detección de objetivos**: búsqueda de objetivos de calibración en las imágenes.
2. **Procesamiento**: aplicación de correcciones y exportación.

**La barra de progreso muestra:**

* Porcentaje de finalización total (0-100 %)
* Nombre de la etapa actual
* Visualización sencilla con barra horizontal

### Barra de progreso de Chloros+

Para usuarios con licencia Chloros+:

**Visualización del progreso en 4 etapas:**

1. **Detección**: búsqueda de objetivos de calibración
2. **Análisis**: examen de imágenes y preparación del proceso
3. **Calibración**: aplicación de correcciones de viñeta y reflectancia
4. **Exportación**: guardado de archivos procesados

**Funciones interactivas:**

* **Pase el cursor por encima** de la barra de progreso para ver el panel ampliado de 4 etapas
* **Haga clic** en la barra de progreso para congelar/fijar el panel ampliado
* **Vuelva a hacer clic** para descongelarlo y ocultarlo automáticamente al retirar el ratón
* Cada etapa muestra el progreso individual (0-100 %)

***

## Comprensión de cada etapa del procesamiento

### Etapa 1: Detección (detección de objetivos)

**Qué ocurre:**

* Chloros escanea las imágenes marcadas con la casilla Objetivo
* Los algoritmos de visión artificial identifican los 4 paneles de calibración
* Se extraen los valores de reflectancia de cada panel
* Se registran las marcas de tiempo de los objetivos para programar correctamente la calibración

**Duración:**

* Con objetivos marcados: 10-60 segundos
* Sin objetivos marcados: 5-30+ minutos (escanea todas las imágenes)

**Indicador de progreso:**

* Detección: 0 % → 100 %.
* Número de imágenes escaneadas.
* Recuento de objetivos encontrados.

**A qué hay que prestar atención:**

* Debería completarse rápidamente si los objetivos están correctamente marcados.
* Si tarda demasiado, es posible que los objetivos no estén marcados.
* Compruebe el registro de depuración en busca de mensajes de «Objetivo encontrado».

### Etapa 2: Análisis

**Qué está sucediendo:**

* Lectura de metadatos EXIF de la imagen (marcas de tiempo, ajustes de exposición).
* Determinación de la estrategia de calibración basada en las marcas de tiempo de los objetivos.
* Organización de la cola de procesamiento de imágenes.
* Preparación de los trabajadores de procesamiento paralelo (solo Chloros+).

**Duración:** 5-30 segundos.

**Indicador de progreso:**

* Análisis: 0 % → 100 %
* Etapa rápida, suele completarse rápidamente

**A qué hay que prestar atención:**

* Debe progresar de forma constante sin pausas
* Las advertencias sobre metadatos faltantes aparecerán en el registro de depuración

### Etapa 3: Calibración

**Qué está sucediendo:**

* **Desbayerización**: conversión del patrón Bayer RAW a 3 canales
* **Corrección de viñeta**: eliminación del oscurecimiento del borde de la lente.
* **Calibración de reflectancia**: normalización con valores objetivo.
* **Cálculo de índices**: cálculo de índices multiespectrales.
* Procesamiento de cada imagen a través de todo el proceso.

**Duración:** la mayor parte del tiempo total de procesamiento (60-80 %).

**Indicador de progreso:**

* Calibración: 0 % → 100 %
* Imagen actual en proceso
* Imágenes completadas / Imágenes totales

**Comportamiento del procesamiento:**

* **Modo libre**: procesa una imagen a la vez de forma secuencial
* **Modo Chloros+**: procesa hasta 16 imágenes simultáneamente
* **Aceleración GPU**: Acelera significativamente esta etapa.

**A qué hay que prestar atención:**

* Progreso constante a través del recuento de imágenes.
* Compruebe el registro de depuración para ver los mensajes de finalización por imagen.
* Advertencias sobre la calidad de la imagen o problemas de calibración.

### Etapa 4: Exportación

**Qué está sucediendo:**

* Escribir imágenes calibradas en el disco en el formato seleccionado
* Exportar imágenes de índice multiespectral con colores LUT
* Crear subcarpetas de modelos de cámara
* Conservar los nombres de archivo originales con los sufijos adecuados

**Duración:** 10-20 % del tiempo total de procesamiento

**Indicador de progreso:**

* Exportación: 0 % → 100 %
* Archivos en proceso de escritura
* Formato y destino de exportación

**A qué hay que prestar atención:**

* Advertencias de espacio en disco
* Errores de escritura de archivos
* Finalización de todas las salidas configuradas

***

## Pestaña Registro de depuración

El Registro de depuración proporciona información detallada sobre el progreso del procesamiento y cualquier problema que se haya encontrado.

### Acceso al Registro de depuración

1. Haga clic en el icono **Registro de depuración** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> en la barra lateral izquierda
2. Se abre el panel de registro, que muestra mensajes de procesamiento en tiempo real
3. Se desplaza automáticamente para mostrar los mensajes más recientes

### Comprensión de los mensajes de registro

#### Mensajes de información (blancos/grises)

Actualizaciones de procesamiento normales:

```
[INFO] Processing started
[INFO] Target detected in IMG_0015.RAW - 4 panels found
[INFO] Calibrating IMG_0234.RAW
[INFO] Exported NDVI image: IMG_0234_NDVI.tif
[INFO] Processing complete
```

#### Mensajes de advertencia (amarillos)

Problemas no críticos que no detienen el procesamiento:

```
[WARN] No GPS data found in IMG_0145.RAW
[WARN] Target image timestamp gap > 30 minutes
[WARN] Low contrast in calibration panel - results may vary
```

**Acción:** Revise las advertencias después del procesamiento, pero no lo interrumpa.

#### Mensajes de error (Red)

Problemas críticos que pueden provocar un fallo en el procesamiento:

```
[ERROR] Cannot write file - disk full
[ERROR] Corrupted image file: IMG_0299.RAW
[ERROR] No targets detected - enable reflectance calibration or mark target images
```

**Acción:** Detener el procesamiento, resolver el error y reiniciar.

### Mensajes de registro comunes

| Mensaje                          | Significado                                | Acción necesaria                                         |
| -------------------------------- | -------------------------------------- | ----------------------------------------------------- |
| «Objetivo detectado en \[nombre de archivo]» | Objetivo de calibración encontrado correctamente  | Ninguna - normal                                         |
| «Procesando imagen X de Y»        | Actualización del progreso actual                | Ninguna - normal                                         |
| «No se han encontrado objetivos»               | No se han detectado objetivos de calibración        | Marcar las imágenes objetivo o desactivar la calibración de reflectancia |
| «Espacio en disco insuficiente»        | Almacenamiento insuficiente para la salida          | Liberar espacio en disco                                    |
| «Omitiendo archivo dañado»        | El archivo de imagen está dañado                  | Volver a copiar el archivo desde la tarjeta SD                             |
| «Datos PPK aplicados»               | Correcciones GPS del archivo .daq aplicadas | Ninguna - normal                                         |

### Copiar datos de registro

Para copiar el registro con fines de resolución de problemas o asistencia técnica:

1. Abra el panel Registro de depuración.
2. Haga clic en el botón **«Copiar registro»** (o haga clic con el botón derecho → Seleccionar todo).
3. Péguelo en un archivo de texto o en un correo electrónico.
4. Envíelo a la asistencia técnica de MAPIR si es necesario.

***

## Supervisión de los recursos del sistema

### Uso de la CPU

**Modo libre:**

* 1 núcleo de CPU al ~100 %
* Otros núcleos inactivos o disponibles
* El sistema sigue respondiendo

**Chloros+ Modo paralelo:**

* Varios núcleos al 80-100 % (hasta 16 núcleos)
* Alta utilización general de la CPU
* El sistema puede parecer menos receptivo.

**Para supervisar:**

* Windows Administrador de tareas (Ctrl+Mayús+Esc)
* Pestaña Rendimiento → Sección CPU
* Busque los procesos «Chloros» o «chloros-backend».

### Uso de la memoria (RAM)

**Uso típico:**

* Proyectos pequeños (&lt; 100 imágenes): 2-4 GB
* Proyectos medianos (100-500 imágenes): 4-8 GB
* Proyectos grandes (más de 500 imágenes): 8-16 GB
* El modo paralelo Chloros+ utiliza más RAM

**Si la memoria es insuficiente:**

* Procese lotes más pequeños.
* Cierre otras aplicaciones.
* Actualice la RAM si procesa regularmente grandes conjuntos de datos.

### Uso de la GPU (Chloros+ con CUDA)

Cuando la aceleración de la GPU está habilitada:

* La GPU NVIDIA muestra una alta utilización (60-90 %).
* El uso de VRAM aumenta (requiere 4 GB+ de VRAM).
* La etapa de calibración es significativamente más rápida.

**Para supervisar:**

* Icono de la bandeja del sistema NVIDIA.
* Administrador de tareas → Rendimiento → GPU.
* GPU-Z o herramienta de supervisión similar.

### E/S de disco

**Qué esperar:**

* Alta lectura del disco durante la fase de análisis.
* Alta escritura del disco durante la fase de exportación.
* SSD significativamente más rápido que HDD.

**Consejo de rendimiento:**

* Utilice SSD para la carpeta del proyecto siempre que sea posible.
* Evite las unidades de red para conjuntos de datos grandes.
* Asegúrese de que el disco no esté cerca de su capacidad máxima (afecta a la velocidad de escritura).

***

## Detección de problemas durante el procesamiento

### Señales de advertencia

**El progreso se detiene (sin cambios durante más de 5 minutos):**

* Compruebe si hay errores en el registro de depuración.
* Verifique el espacio disponible en disco.
* Compruebe el Administrador de tareas para asegurarse de que Chloros se está ejecutando.

**Aparecen mensajes de error con frecuencia:**

* Detenga el procesamiento y revise los errores.
* Causas comunes: espacio en disco, archivos dañados, problemas de memoria.
* Consulte la sección Solución de problemas más abajo.

**El sistema deja de responder:**

* Chloros+ en modo paralelo utiliza demasiados recursos.
* Considere la posibilidad de reducir las tareas simultáneas o actualizar el hardware.
* El modo libre consume menos recursos.

### Cuándo detener el procesamiento

Detenga el procesamiento si ve:

* ❌ Errores «Disco lleno» o «No se puede escribir el archivo».
* ❌ Errores repetidos de corrupción de archivos de imagen.
* ❌ El sistema se ha bloqueado por completo (no responde).
* ❌ Se ha detectado que se han configurado ajustes incorrectos.
* ❌ Se han importado imágenes incorrectas.

**Cómo detenerlo:**

1. Haga clic en el **botón Detener/Cancelar** (sustituye al botón Iniciar).
2. El procesamiento se detiene y se pierde el progreso
3. Solucione los problemas y vuelva a empezar desde el principio

***

## Solución de problemas durante el procesamiento

### El procesamiento es muy lento

**Posibles causas:**

* Imágenes de destino sin marcar (escaneo de todas las imágenes)
* Almacenamiento en HDD en lugar de SSD
* Recursos del sistema insuficientes
* Muchos índices configurados
* Acceso a la unidad de red

**Soluciones:**

1. Si acaba de empezar y se encuentra en la fase de detección: cancele, marque los objetivos y reinicie.
2. Para el futuro: utilice SSD, reduzca los índices y actualice el hardware.
3. Considere la posibilidad de utilizar CLI para el procesamiento por lotes de grandes conjuntos de datos.

### Advertencias de «espacio en disco»

**Soluciones:**

1. Libere espacio en disco inmediatamente.
2. Mueva el proyecto a una unidad con más espacio.
3. Reduzca el número de índices para exportar.
4. Utilice el formato JPG en lugar de TIFF (archivos más pequeños).

### Mensajes frecuentes de «archivo dañado»

**Soluciones:**

1. Vuelva a copiar las imágenes desde la tarjeta SD para garantizar su integridad.
2. Compruebe si la tarjeta SD tiene errores.
3. Elimine los archivos dañados del proyecto.
4. Continúe procesando las imágenes restantes.

### Sobrecalentamiento/ralentización del sistema

**Soluciones:**

1. Asegúrese de que haya una ventilación adecuada.
2. Limpie el polvo de las rejillas de ventilación del ordenador.
3. Reduzca la carga de procesamiento (utilice el modo Free en lugar de Chloros+).
4. Procese durante las horas más frescas del día.

***

## Notificación de procesamiento completado

Cuando finalice el procesamiento:

* La barra de progreso alcanza el 100 %.
* Aparece el mensaje **«Procesamiento completado»** en el registro de depuración.
* El botón de inicio vuelve a estar habilitado.
* Todos los archivos de salida se encuentran en la subcarpeta del modelo de cámara.

***

## Pasos siguientes

Una vez completado el procesamiento:

1. **Revise los resultados**: consulte [Finalización del procesamiento](finishing-the-processing.md).
2. **Compruebe la carpeta de salida**: verifique que todos los archivos se hayan exportado correctamente.
3. **Revise el registro de depuración**: compruebe si hay advertencias o errores.
4. **Previsualice las imágenes procesadas**: utilice el visor de imágenes o un software externo.

Para obtener información sobre cómo revisar y utilizar los resultados procesados, consulte [Finalización del procesamiento](finishing-the-processing.md).

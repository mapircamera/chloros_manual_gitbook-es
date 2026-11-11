# Seguimiento del procesamiento

Una vez iniciado el procesamiento, Chloros ofrece varias formas de supervisar el progreso, detectar posibles problemas y comprender qué está ocurriendo con tu conjunto de datos. En esta página se explica cómo realizar un seguimiento del procesamiento e interpretar la información que proporciona Chloros.

## Descripción general de la barra de progreso

La barra de progreso situada en la parte superior de la página muestra el estado del procesamiento en tiempo real y el porcentaje de finalización. El progreso se transmite en directo desde el backend a través de Server-Sent Events (SSE), por lo que la barra refleja lo que el proceso está haciendo realmente.

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
2. **Análisis**: examen de las imágenes y preparación del proceso
3. **Calibración**: aplicación de correcciones de viñeteado y reflectancia
4. **Exportación**: guardado de los archivos procesados**Funciones interactivas:*** **Pasa el cursor por encima** de la barra de progreso para ver el panel ampliado de 4 etapas
* **Haz clic** en la barra de progreso para congelar o fijar el panel ampliado
* **Haz clic de nuevo** para desbloquearlo; se ocultará automáticamente al alejar el ratón
* Cada etapa muestra su progreso individual (0-100 %)

{% hint style="info" %}
**Paridad con la CLI**: durante una ejecución de `chloros-cli process`, los mismos cuatro subprocesos muestran los estados «Detectando», «Analizando», «Procesando» y «Exportando», y `chloros-cli export-status` muestra en tiempo real el progreso de la exportación del subproceso 4 desde otro terminal. Consulte la [Referencia de la CLI](../reference/cli-reference.md).
{% endhint %}

***

## Explicación de cada etapa de procesamiento

{% hint style="info" %}
**Arquitectura de canalización**: Estas 4 etapas de la interfaz gráfica de usuario se corresponden con la [canalización de procesamiento de 4 subprocesos](../processing-architecture/processing-pipeline.md). En sistemas con aceleración por GPU, el hilo 3 (Calibración) se beneficia de la [adaptación dinámica de cálculo](../processing-architecture/dynamic-compute-adaptation.md), que optimiza el procesamiento para tu hardware específico.
{% endhint %}

### Etapa 1: Detección (detección de objetivos)

**Qué ocurre:**

* Chloros escanea las imágenes que has marcado con la casilla «Objetivo» (todas las imágenes si no se ha marcado ninguna)
* Los algoritmos de visión artificial identifican los paneles de calibración
* Se extraen los valores de reflectancia de cada panel
* Se registran las marcas de tiempo de los objetivos para programar correctamente la calibración

**Duración:**

* Con objetivos marcados: 10-60 segundos
* Sin objetivos marcados: 5-30+ minutos (escanea todas las imágenes)

**Indicador de progreso:**

* Detectando: 0 % → 100 %
* Número de imágenes escaneadas (solo cuenta las imágenes que se están escaneando realmente)
* Recuento de objetivos encontrados

**A tener en cuenta:**

* Debería completarse rápidamente si los objetivos están correctamente marcados
* Si tarda demasiado, es posible que los objetivos no estén marcados
* Comprueba el registro de depuración en busca de mensajes de «Objetivo encontrado»

### Fase 2: Análisis

**Qué ocurre:**

* Lectura de los metadatos EXIF de las imágenes (marcas de tiempo, ajustes de exposición)
* Determinación de la estrategia de calibración basada en las marcas de tiempo de los objetivos y los datos de radiación descendente de DAQ disponibles
* Organización de la cola de procesamiento de imágenes
* Preparación de los procesos de procesamiento en paralelo (solo Chloros+)

**Duración:** 5-30 segundos**Indicador de progreso:**

* Analizando: 0 % → 100 %
* Etapa rápida, suele completarse rápidamente

**A qué hay que prestar atención:**

* El progreso debe ser constante, sin pausas
* Las advertencias sobre metadatos que faltan aparecerán en el registro de depuración

### Etapa 3: Calibración

**Qué ocurre:*** **Debayering**: Conversión del patrón RAW de Bayer a 3 canales (se omite en los módulos monocromáticos LATTICE, con una nota)
* **Corrección de viñeteado**: eliminación del oscurecimiento en los bordes del objetivo
* **Calibración de reflectancia**: normalización con valores objetivo y/o downwelling de DAQ
* **Cálculo de índices**: cálculo de índices multiespectrales
* Procesamiento de cada imagen a lo largo de todo el flujo de trabajo

**Duración:** La mayor parte del tiempo total de procesamiento (60-80 %)**Indicador de progreso:**

* Calibrando: 0 % → 100 %
* Imagen actual en proceso
* Imágenes completadas / Total de imágenes

**Comportamiento del procesamiento:*** **Modo libre**: procesa una imagen cada vez de forma secuencial
* **Modo Chloros+**: ejecuta un grupo de trabajadores adaptado al hardware: de 1 a 4 trabajadores simultáneos en sistemas con GPU (según la VRAM); un trabajador por núcleo físico (menos uno) en sistemas solo con CPU. Véase [Adaptación dinámica de cálculo](../processing-architecture/dynamic-compute-adaptation.md)
* **Aceleración por GPU**: Acelera significativamente esta etapa**A tener en cuenta:**

* Progreso constante en el recuento de imágenes
* Comprueba el registro de depuración para ver los mensajes de finalización de cada imagen
* Advertencias sobre problemas de calidad de imagen o de calibración

### Etapa 4: Exportación

**Qué ocurre:**

* Se guardan en disco las imágenes procesadas en el formato seleccionado, a medida que se completan
* **LATTICE**: cada fotograma se distribuye a todos los productos habilitados (debayering / vista previa / radiancia / reflectancia)
* Exportación de imágenes de índices multiespectrales con colores LUT
* Creación del árbol de salida `<project>/<camera>/<format>/<Product>_Images/`: los archivos exportados conservan el nombre del archivo de origen; la carpeta identifica el producto

**Duración:** entre el 10 % y el 20 % del tiempo total de procesamiento**Indicador de progreso:**

* Exportando: 0 % → 100 %
* Archivos en proceso de escritura
* Formato de exportación y destino

**A tener en cuenta:**

* Advertencias sobre el espacio en disco
* Errores al escribir archivos
* Finalización de todas las salidas configuradas

***

## Pestaña «Registro de depuración»

El «Registro de depuración» proporciona información detallada sobre el progreso del procesamiento y cualquier problema que se haya producido. Los mensajes de inicio del backend también se reproducen en la consola de registro, por lo que el registro ofrece una visión completa de lo ocurrido, incluso si se abre más tarde.

### Cómo acceder al registro de depuración

1. Haz clic en el icono **Registro de depuración** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">

de la barra lateral izquierda
2. Se abre el panel de registro, que muestra mensajes de procesamiento en tiempo real
3. Se desplaza automáticamente para mostrar los mensajes más recientes

<!-- SCREENSHOT-NEEDED: Debug Log tab open at the end of a completed run, showing real backend log lines including the [RUN-SUMMARY] lines (images / camera groups / targets / calibrated / files written), with the 1.3.0 bug icon (Report a bug) visible at the top of the tab -->

### Cómo interpretar los mensajes del registro Las líneas del registro de

Chloros van precedidas de etiquetas entre corchetes que indican el nombre del subsistema; por ejemplo, `[PROCESSING]`, `[RUN-SUMMARY]`, `[LATTICE-EXPORT]`, `[EXPORT-CHECK]`, `[IMPORT-LEVEL]`. Lo más importante que hay que saber es el **resumen de la ejecución**, que aparece al final de cada ejecución (incluidas las que se han detenido):

```
[RUN-SUMMARY] 49 image(s) in 2 camera group(s); 4 target(s) detected; 45 image(s) calibrated; 180 file(s) written.
```

Siempre que haya algo que requiera una explicación, aparecen líneas adicionales de ayuda `[RUN-SUMMARY]`; por ejemplo, una ejecución que no haya generado ningún resultado, o una cámara cuyo producto solicitado se omitió por no ser aplicable. Las líneas `[EXPORT-CHECK]` explican las omisiones por cámara (por ejemplo, por qué una cámara RGB no obtuvo ningún producto de radiancia).

Los niveles de gravedad generales de los mensajes (los ejemplos que figuran a continuación son ilustrativos, no literales):

#### Mensajes informativos (blanco/gris)

Actualizaciones normales del procesamiento: inicio del procesamiento, objetivos detectados (con recuentos de paneles), progreso de la calibración por imagen, archivos exportados, procesamiento completado.

#### Mensajes de advertencia (amarillo)

Problemas no críticos que no detienen el procesamiento; por ejemplo, datos GPS que faltan en un fotograma, una gran diferencia en la marca de tiempo entre imágenes objetivo o bajo contraste en un panel de calibración.

**Acción:** Revisar las advertencias tras el procesamiento, pero no interrumpirlo

#### Mensajes de error (color Red

e)

Problemas críticos que pueden provocar que el procesamiento falle —por ejemplo, disco lleno, un archivo de imagen dañado o que no se hayan detectado objetivos cuando se solicitó la calibración de reflectancia.

**Acción:** Detener el procesamiento, resolver el error y reiniciar

### Situaciones habituales en el registro

| Situación                             | Significado                                       | Acción necesaria                                         |
| ------------------------------------- | --------------------------------------------- | ----------------------------------------------------- |
| Objetivo detectado en \[nombre de archivo] | Objetivo de calibración encontrado con éxito | Ninguna — normal                                         |
| Líneas de progreso por imagen              | Actualización del progreso actual                       | Ninguna — normal                                         |
| No se han encontrado objetivos                      | No se han detectado objetivos de calibración               | Marcar las imágenes de los objetivos o desactivar la calibración de reflectancia |
| Espacio en disco insuficiente               | No hay suficiente almacenamiento para la salida                 | Liberar espacio en disco                                    |
| Se omite el archivo dañado               | El archivo de imagen está dañado                         | Vuelve a copiar el archivo desde la tarjeta SD                             |
| `[IMPORT-LEVEL] Skipping ... no raw source` | No se puede procesar una captura sin un fotograma sin procesar | Vuelve a capturar con el fotograma sin procesar, o utilizar la CLI `--input-level`  |
| `[RUN-SUMMARY] ... 0 file(s) written` | La ejecución no ha generado productos de imagen — se ha notificado como un fallo con indicaciones | Lee las líneas de indicación; comprueba qué se ha omitido y por qué |
| `[PROCESS] refused: The project folder is too deep for this computer…` | Las rutas largas de Windows están desactivadas y la carpeta del proyecto tiene demasiados niveles de subcarpetas para el límite de 260 caracteres de la ruta; no se ha procesado nada | Utiliza una carpeta de proyecto más corta (el mensaje indica la ruta más larga que cabe), o activa las rutas largas de Windows y reinicia Chloros; consulta [Inicio del procesamiento](starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer) |
| `[unmix-clamp-mask] …` | No se ha podido escribir la máscara de sujeción de un producto LATTICE; el producto en sí está bien y muestra el mensaje `Chloros:UnmixClampMask=unavailable` | Comprueba el espacio en disco y los permisos de la carpeta; a continuación, vuelve a procesar si necesitas la máscara |

### Copiar datos del registro

Para copiar el registro con fines de resolución de problemas o asistencia técnica:

1. Abre el panel «Registro de depuración»
2. Haz clic en el botón **Copiar**(copia el registro al portapapeles) o en**Descargar** para guardarlo como un archivo de texto
3. Pégalo en un archivo de texto o en un correo electrónico
4. Envíalo al servicio de asistencia de MAPIR si es necesario

### Notificar un error desde la pestaña «Registro de depuración»

Novedad en la versión 1.3.0: el icono de error (**Notificar un error**) situado en la parte superior de la pestaña «Registro de depuración» envía un informe directamente a MAPIR.

1. Haz clic en el icono de error situado en la parte superior de la pestaña «Registro de depuración»
2. Describe el problema: qué estabas haciendo, qué esperabas y qué ocurrió en realidad (obligatorio).
3. Si lo deseas, marca **Añadir algunas imágenes (hasta 10)**y selecciona imágenes del proyecto abierto, o utiliza**Examinar...** para cualquier otro archivo de imagen.
4. Haz clic en **Enviar**

El informe incluye tu descripción, las imágenes que adjuntes, los registros de Chloros y los detalles del sistema (versión de Chloros, sistema operativo, GPU y configuración del proyecto). Debes haber iniciado sesión en Chloros (pestaña «Usuario») para enviarlo. Puedes cerrar la ventana mientras se envía; cuando termine, se registrará en el registro una referencia como `Bug report sent. Reference: BR-…`; indícala cuando te pongas en contacto con MAPIR. Si no se puede cargar el informe (por ejemplo, por falta de conexión a Internet), Chloros lo guarda en tu ordenador y te muestra la ruta para que puedas enviarlo por correo electrónico a support@mapir.camera.

***

## Supervisión de recursos del sistema

### Uso de la CPU

**Modo libre:**

* 1 núcleo de CPU al 100 % aproximadamente
* Los demás núcleos están inactivos o disponibles
* El sistema sigue respondiendo

**Modo paralelo de Chloros+:**

* Varios núcleos con una alta utilización; el número depende de la estrategia elegida por [Adaptación dinámica de cálculo](../processing-architecture/dynamic-compute-adaptation.md)
* El sistema puede parecer menos receptivo

**Para supervisarlo:**

* Administrador de tareas de Windows (Ctrl+Mayús+Esc)
* Pestaña «Rendimiento» → sección «CPU»
* Busca los procesos «Chloros» o «chloros-backend»

### Uso de la memoria (RAM)

**Uso habitual:**

* Proyectos pequeños (< 100 imágenes): 2-4 GB
* Proyectos medianos (100-500 imágenes): 4-8 GB
* Proyectos grandes (más de 500 imágenes): 8-16 GB
* El modo paralelo de Chloros+ consume más RAM

**Si la memoria es escasa:**

* Procesa lotes más pequeños
* Cierra otras aplicaciones
* Amplía la RAM si procesas habitualmente conjuntos de datos grandes

### Uso de la GPU (Chloros+ con CUDA)

Cuando la aceleración por GPU está activada:

* La GPU NVIDIA muestra una alta utilización (60-90 %)
* Aumenta el uso de VRAM (se requieren más de 4 GB de VRAM; más de 7 GB para el «debayering» simultáneo con reconocimiento de texturas)
* La fase de calibración es significativamente más rápida

**Para supervisar:**

* Icono de NVIDIA en la bandeja del sistema
* Administrador de tareas → Rendimiento → GPU
* GPU-Z o una herramienta de supervisión similar

### E/S de disco

**Qué cabe esperar:**

* Elevada lectura de disco durante la fase de análisis
* Elevada escritura de disco durante la fase de exportación
* Los SSD son significativamente más rápidos que los HDD

**Consejo de rendimiento:**

* Utiliza un SSD para la carpeta del proyecto siempre que sea posible
* Evita las unidades de red para conjuntos de datos de gran tamaño
* Asegúrate de que el disco no esté casi lleno (afecta a la velocidad de escritura)

***

## Detección de problemas durante el procesamiento

### Señales de advertencia

**El progreso se detiene (sin cambios durante más de 5 minutos):**

* Comprueba el registro de depuración en busca de errores
* Verifica el espacio disponible en disco
* Comprueba el Administrador de tareas para asegurarte de que Chloros se está ejecutando

**Aparecen mensajes de error con frecuencia:**

* Detén el procesamiento y revisa los errores
* Causas comunes: espacio en disco, archivos dañados, problemas de memoria
* Consulta la sección «Solución de problemas» más abajo

**El sistema deja de responder:**

* El modo paralelo de Chloros consume demasiados recursos
* Considera la posibilidad de reducir el número de tareas simultáneas o actualizar el hardware
* El modo libre consume menos recursos

### Cuándo detener el procesamiento

Detén el procesamiento si observas:

* ❌ Errores del tipo «Disco lleno» o «No se puede escribir en el archivo»
* ❌ Errores repetidos de corrupción de archivos de imagen
* ❌ El sistema se ha bloqueado por completo (no responde)
* ❌ Te has dado cuenta de que se han configurado ajustes incorrectos
* ❌ Se han importado imágenes incorrectas

**Cómo detenerlo:**

1. Haz clic en el**botón Detener** (que sustituye al botón Iniciar); basta con una vez
2. La barra muestra «Deteniéndose...» mientras se completa la imagen en curso; a continuación, la ejecución finaliza en estado detenido
3. Los productos ya exportados permanecen en el disco; el registro muestra un informe detallado (`[RUN-SUMMARY]`) de lo que se ha completado
4. Soluciona los problemas y reinicia: la ejecución comienza desde el principio

***

## Resolución de problemas durante el procesamiento

### El procesamiento es muy lento

**Posibles causas:**

* Imágenes de destino sin marcar (se escanean todas las imágenes)
* Almacenamiento en disco duro (HDD) en lugar de SSD
* Recursos del sistema insuficientes
* Se han configurado muchos índices
* Acceso a una unidad de red

**Soluciones:**

1. Si acaba de iniciarse y se encuentra en la fase de detección: deténgalo, marque los objetivos y reinicie
2. Para el futuro: utilice un SSD, reduzca el número de índices y actualice el hardware
3. Considere el uso de la CLI para el procesamiento por lotes de grandes conjuntos de datos

### Advertencias de «espacio en disco»

**Soluciones:**

1. Libere espacio en disco inmediatamente
2. Mueve el proyecto a una unidad con más espacio
3. Reduce el número de índices que se van a exportar
4. Desactiva los productos de exportación de LATTICE que no necesites (Configuración del proyecto → Procesamiento)
5. Utiliza el formato JPG en lugar de TIFF (archivos más pequeños)

### Mensajes frecuentes de «archivo dañado»

**Soluciones:**

1. Vuelve a copiar las imágenes desde la tarjeta SD para garantizar su integridad
2. Comprueba si la tarjeta SD tiene errores
3. Elimina los archivos dañados del proyecto
4. Continúa procesando las imágenes restantes

### Sobrecalentamiento del sistema / Limitación de rendimiento

**Soluciones:**

1. Asegúrate de que haya una ventilación adecuada
2. Limpia el polvo de las rejillas de ventilación del ordenador
3. Reduce la carga de procesamiento (utiliza el modo «Free» en lugar de «Chloros+»)
4. Realiza el procesamiento durante las horas más frescas del día

***

## Notificación de finalización del procesamiento

Cuando finaliza el procesamiento:

* La barra de progreso alcanza el 100 %
* Aparecen las líneas `[RUN-SUMMARY]` en el registro de depuración con los recuentos finales
* El botón «Iniciar» vuelve a estar activo
* Todos los archivos de salida se encuentran en el árbol de salida por cámara del proyecto: `<project>/<camera>/<format>/<Product>_Images/`

***

## Próximos pasos

Una vez finalizado el procesamiento:

1. **Revisar los resultados**: consulta [Finalización del procesamiento](finishing-the-processing.md)
2. **Comprobar la carpeta de salida**: verifica que todos los archivos se hayan exportado correctamente
3. **Revisar el registro de depuración**: compruebe si hay advertencias o errores
4. **Previsualizar las imágenes procesadas**: utilice el visor de imágenes o un software externo

Para obtener información sobre cómo revisar y utilizar los resultados procesados, consulte [Finalización del procesamiento](finishing-the-processing.md).

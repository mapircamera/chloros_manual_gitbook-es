# Finalización del procesamiento

Una vez que Chloros haya finalizado el procesamiento, es el momento de revisar los resultados, comprobar la calidad de la salida y preparar las imágenes procesadas para su uso en su flujo de trabajo. Esta página le guía a través de los pasos finales y las acciones siguientes.

## Indicación de que el procesamiento ha finalizado

Cuando el procesamiento finalice correctamente, verá varios indicadores:

* ✅ **Barra de progreso**: alcanza el 100 % de finalización
* ✅ **Registro de depuración**: muestra el mensaje «Procesamiento completado»
* ✅ **Botón de inicio**: vuelve a estar habilitado (listo para la siguiente ejecución del procesamiento)
* ✅ **Archivos de salida**: Todas las imágenes procesadas se guardan en la subcarpeta del modelo de cámara***

## Localización de las imágenes procesadas

### Apertura de la carpeta de salida

1. Haz clic en el icono del **Menú principal** <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> (arriba a la izquierda)
2. Seleccione **«Abrir carpeta del proyecto»**

3. Se abrirá el explorador de archivos en el directorio del proyecto
4. Localice su proyecto por el nombre

***

## Revisión de las imágenes procesadas

### Vista previa rápida en el explorador de archivos

**Vista previa integrada de Windows:**

1. Navega hasta la subcarpeta del modelo de cámara
2. Selecciona un archivo de imagen
3. La vista previa aparece en el panel de vista previa del explorador de Windows
4. Utiliza las teclas de flecha para desplazarte por las imágenes

### Vista previa en visores de imágenes externos

**Visores recomendados:*** **QGIS**: software SIG gratuito (ideal para análisis multiespectral georreferenciado)
* **IrfanView**: visor de imágenes rápido y ligero (compatible con TIFF)
* **Adobe Photoshop**: edición profesional (compatible con TIFF)
* **GIMP**: alternativa gratuita a Photoshop
* **Windows Photos**: visualización básica (puede que no sea compatible con TIFF de 16 bits)

### Vista previa en el visor de imágenes de Chloros

Utilice el visor de imágenes integrado de Chloros para una visualización avanzada:

1. Haga clic en la miniatura de una imagen en el explorador de archivos
2. La imagen se abre en el área de vista previa principal
3. Haga clic en la pestaña **Visor de imágenes** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> en la barra lateral izquierda
4. Utilice [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) para realizar un análisis interactivo

Consulte [Visor de imágenes](../image-viewer-gui/opening-an-image-full-screen.md) para obtener instrucciones detalladas.

***

## Revisión del registro de depuración

### Compruebe si hay advertencias o errores

1. Abra la pestaña **Registro de depuración** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> 2. Desplácese por los mensajes
3. Busque advertencias amarillas o errores rojos
4. Revise cualquier problema detectado
5. Póngase en contacto con el servicio de asistencia de MAPIR para obtener ayuda

### Guardar el registro

Para mantener un registro del procesamiento o enviarlo al servicio de asistencia de MAPIR:

1. Haga clic en el botón **«Copiar»**o**«Descargar»**

2. Guárdelo como archivo de texto en la carpeta del proyecto
3. Inclúyalo en la documentación del proyecto
4. Envíelo al servicio de asistencia de MAPIR si se producen problemas

***

## Problemas comunes de salida y soluciones

### Problema: Faltan archivos de salida

**Posibles causas:**

* Los archivos no cumplían los criterios de procesamiento
* Imágenes solo de destino (excluidas de la exportación)
* Se agotó el espacio en disco durante la exportación
* Corrupción de archivos durante el procesamiento

**Soluciones:**

1. Compruebe el registro de depuración en busca de mensajes de omisión o error
2. Verifique que había suficiente espacio en disco
3. Cuente los archivos: debe coincidir con (recuento original - recuento de destino) × (índices + 1)
4. Vuelva a importar y reprocesar los archivos que falten

### Problema: Bordes oscuros o brillantes (el viñeteado sigue siendo visible)

**Posibles causas:**

* Corrección de viñeteado desactivada
* La cámara o el objetivo no están en la base de datos de perfiles de Chloros
* Viñeteado extremo que supera la capacidad de corrección

**Soluciones:**

1. Compruebe que la corrección de viñeteado esté habilitada en la configuración del proyecto
2. Compruebe que se haya detectado correctamente el modelo de cámara
3. Póngase en contacto con el servicio técnico de MAPIR si el viñeteado persiste

### Problema: Colores o valores incorrectos

**Posibles causas:**

* No se han detectado objetivos de calibración
* Se ha seleccionado un modelo de objetivo de calibración incorrecto
* Calibración de reflectancia desactivada
* Imágenes de los objetivos de baja calidad

**Soluciones:**

1. Compruebe que la calibración de reflectancia estaba habilitada
2. Compruebe los mensajes «Objetivo encontrado» en el registro de depuración
3. Revise la calidad de las imágenes de los objetivos
4. Vuelva a procesar con los objetivos adecuados marcados

### Problema: Los valores de NDVI parecen incorrectos

**Rangos esperados de NDVI:*** **Agua, rocas, suelo**: de -0,1 a 0,2
* **Vegetación escasa/en mal estado**: de 0,2 a 0,4
* **Vegetación moderada**: de 0,4 a 0,6
* **Vegetación sana y densa**: de 0,6 a 0,9**Si los valores se encuentran fuera de estos rangos:**

1. Comprueba que se haya aplicado la calibración de reflectancia
2. Comprueba que se haya incluido el registro del sensor de luz
3. Comprueba que se hayan detectado los objetivos de calibración
4. Asegúrate de que se haya detectado el modelo de cámara correcto
5. Revisa el momento y las condiciones de captura de la imagen objetivo

***

## Uso de las imágenes procesadas

### Para fotogrametría / creación de ortomosaicos

**Flujo de trabajo recomendado:**

1.**Importe las imágenes de reflectancia calibradas** al software de fotogrametría:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Conserve los metadatos EXIF**: Asegúrese de que se conservan los datos GPS para el geoetiquetado
3. **Flujos de trabajo calibrados**: Utilice imágenes de reflectancia para obtener precisión científica
4. **Procese mosaicos de índice**: Cree ortomosaicos NDVI a partir de imágenes de índice individuales
5. **Exporte GeoTIFF georreferenciados**: Para su uso en aplicaciones SIG

### Para análisis SIG

**Flujo de trabajo recomendado:**

1.**Cargue en QGIS, ArcGIS o similar**

2.**Utilizar imágenes de reflectancia de 16 bits TIFF** para el análisis multibanda
3. **Utilizar imágenes de índice** (NDVI, NDRE) como capas de vegetación listas para usar
4. **Calculadora ráster**: Combinar bandas para análisis personalizados
5. **Exportar**: Crea mapas de clasificación, detección de cambios y mapas de salud de la vegetación

### Para análisis directo / generación de informes

**Flujo de trabajo recomendado:**

1.**Utiliza imágenes de índice con colores LUT** para informes visuales
2. **Extrae estadísticas**: Media de NDVI por campo/parcela
3. **Series temporales**: compara índices entre varias sesiones
4. **Genera informes**: incluye mapas, estadísticas y visualizaciones***

## Archivo y copias de seguridad

### Estrategia de copias de seguridad recomendada

**Qué guardar:*** ✅ **Imágenes RAW/JPG originales**: archívalas en una unidad o en la nube separadas
* ✅ **Resultados procesados**: conservar las imágenes calibradas y los índices
* ✅ **Archivo del proyecto**: contiene todos los ajustes para volver a procesar si es necesario
* ✅ **Registro de depuración**: documenta los detalles del procesamiento
* ✅ **Imágenes de referencia de calibración**: para verificación y reprocesamiento**Recomendaciones de almacenamiento:*** **Copia de seguridad inmediata**: disco duro externo
* **Archivo a largo plazo**: almacenamiento en la nube (Google Drive, Dropbox, etc.)
* **Datos críticos**: guarda 2-3 copias en ubicaciones diferentes***

## Próximas ejecuciones de procesamiento

### Reutilización de la configuración del proyecto

Si vas a procesar conjuntos de datos similares en el futuro:

1. **Guarda la plantilla del proyecto** (si aún no lo ha hecho)
2. **Crear un nuevo proyecto** utilizando la plantilla guardada
3. **Importar nuevas imágenes**

4.**Procesar**con la misma configuración para garantizar la coherencia

### Procesamiento por lotes de varias sesiones

Para varias sesiones o conjuntos de datos:**Opción 1: GUI - Varios proyectos**

* Crear un proyecto independiente para cada sesión
* Utilizar una configuración de plantilla coherente
* Procesar de uno en uno

**Opción 2: Chloros CLI (solo Chloros+)**

* Automatizar el procesamiento por lotes
* Procesar varias carpetas con scripts
* Consulte la [documentación de CLI](../CLI.md)

**Opción 3: Python SDK (solo Chloros+)**

* Control programático
* Integración con flujos de análisis
* Véase [Documentación de API](../api-python-sdk.md)

***

## Solución de problemas en el posprocesamiento

### Reprocesamiento con diferentes ajustes

Si los resultados no son satisfactorios:

1. Conserve las imágenes originales (no las elimine nunca)
2. Abra el mismo proyecto en Chloros
3. Ajuste la configuración en el panel «Configuración del proyecto»
4. Vuelva a procesar: los resultados sobrescribirán los anteriores

### Procesamiento de un subconjunto de imágenes

Para volver a procesar solo imágenes específicas:

1. Cree un nuevo proyecto
2. Importe solo las imágenes que necesiten volver a procesarse
3. Utilice la misma plantilla de configuración
4. Procese un conjunto de datos más pequeño

### Obtener ayuda

Si tiene algún problema:

* 📧 **Correo electrónico**: info@mapir.camera (incluya el registro de depuración)
* 🌐 **Asistencia técnica**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Preguntas frecuentes**: [Preguntas frecuentes](../faq.md)
* 📖 **Documentación**: [Manual de Chloros](../)***

## Resumen: Flujo de trabajo completo

Ya has completado el flujo de trabajo completo de procesamiento de Chloros:

1. ✅ **Proyecto creado** - Consulta [Proyectos](../projects.md)
2. ✅ **Añadidos archivos** - Véase [Añadir archivos](adding-files-to-a-project.md)
3. ✅ **Ajustada la configuración** - Véase [Ajustar la configuración del proyecto](adjusting-project-settings.md)
4. ✅ **Marcado de objetivos** - Véase [Selección de imágenes objetivo](choosing-target-images.md)
5. ✅ **Inicio del procesamiento** - Véase [Inicio del procesamiento](starting-the-processing.md)
6. ✅ **Progreso supervisado** - Véase [Supervisar el procesamiento](monitoring-the-processing.md)
7. ✅ **Resultados revisados** - Esta página**¡Sus imágenes multiespectrales calibradas y con corrección de reflectancia están listas para su análisis!**

***

## Recursos adicionales

### Funciones avanzadas

* [**Visor de imágenes**](../image-viewer-gui/opening-an-image-full-screen.md) - Visualización y análisis interactivos
* [**Área de pruebas de índices/LUT**](../image-viewer-gui/index-lut-sandbox.md) - Pruebas de índices personalizados
* [**Fórmulas de índices multiespectrales**](../project-settings/multispectral-index-formulas.md) - Referencia completa de índices

### Automatización e integración

* [**Documentación de CLI**](../CLI.md) - Procesamiento por lotes desde la línea de comandos
* [**Python SDK**](../api-python-sdk.md) - Automatización programática
* [**Chloros+ Funciones**](../#chloros) - Capacidades de procesamiento avanzadas

### Asistencia y formación

* [**Preguntas frecuentes**](../faq.md) - Respuestas a preguntas comunes
* [**Objetivos de calibración**](../calibration-targets.md) - Explicación de la calibración de reflectancia
* [**Cámaras compatibles**](../supported-cameras.md) - Hardware compatible

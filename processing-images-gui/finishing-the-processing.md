# Finalización del procesamiento

Una vez que Chloros haya completado el procesamiento, es el momento de revisar los resultados, verificar la calidad de salida y preparar las imágenes procesadas para su uso en el flujo de trabajo. Esta página le guía a través de los pasos finales y las siguientes acciones.

## Indicación de procesamiento completado

Cuando el procesamiento finalice correctamente, verá varios indicadores:

* ✅ **Barra de progreso**: alcanza el 100 % de finalización
* ✅ **Registro de depuración**: muestra el mensaje «Procesamiento completado»
* ✅ **Botón de inicio**: se vuelve a habilitar (listo para la siguiente ejecución de procesamiento)
* ✅ **Archivos de salida**: todas las imágenes procesadas se guardan en la subcarpeta del modelo de cámara

***

## Localización de las imágenes procesadas

### Apertura de la carpeta de salida

1. Haga clic en el icono **Menú principal** <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> (arriba a la izquierda).
2. Seleccione **«Abrir carpeta del proyecto»**.
3. Se abrirá el explorador de archivos en el directorio del proyecto.
4. Localice su proyecto por su nombre.

***

## Revisar las imágenes procesadas

### Vista previa rápida en el explorador de archivos

**Vista previa integrada en Windows:**

1. Navegue hasta la subcarpeta del modelo de cámara.
2. Seleccione un archivo de imagen.
3. La vista previa aparecerá en el panel de vista previa del explorador Windows.
4. Utilice las teclas de flecha para navegar por las imágenes.

### Vista previa en visores de imágenes externos

**Visores recomendados:**

* **QGIS**: software GIS gratuito (ideal para análisis multiespectral georreferenciado).
* **IrfanView**: visor de imágenes rápido y ligero (compatible con TIFF).
* **Adobe Photoshop**: edición profesional (compatible con TIFF).
* **GIMP**: alternativa gratuita a Photoshop.
* **Windows Photos**: visualización básica (puede que no sea compatible con TIFF de 16 bits).

### Vista previa en el visor de imágenes Chloros

Utilice el visor de imágenes integrado en Chloros para una visualización avanzada:

1. Haga clic en una miniatura de imagen en el explorador de archivos.
2. La imagen se abre en el área de vista previa principal.
3. Haga clic en la pestaña **Visor de imágenes** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> en la barra lateral izquierda.
4. Utilice [Índice/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) para realizar análisis interactivos.

Consulte [Visor de imágenes](../image-viewer-gui/opening-an-image-full-screen.md) para obtener instrucciones detalladas.

***

## Revisión del registro de depuración

### Compruebe si hay advertencias o errores

1. Abra la pestaña **Registro de depuración** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> .
2. Desplácese por los mensajes.
3. Busque advertencias amarillas o errores rojos.
4. Revise cualquier problema detectado.
5. Póngase en contacto con el servicio de asistencia de MAPIR para obtener ayuda.

### Guardar el registro

Para mantener un registro del procesamiento o enviarlo al servicio de asistencia de MAPIR:

1. Haga clic en el botón **«Copiar»** o **«Descargar»**.
2. Guarde el archivo como archivo de texto en la carpeta del proyecto.
3. Incluya la documentación del proyecto.
4. Envíelo al servicio de asistencia de MAPIR si encuentra algún problema.

***

## Problemas comunes de salida y soluciones

### Problema: Faltan archivos de salida

**Posibles causas:**

* Los archivos no cumplían los criterios de procesamiento.
* Imágenes solo de destino (excluidas de la exportación).
* Se agotó el espacio en disco durante la exportación.
* Corrupción de archivos durante el procesamiento.

**Soluciones:**

1. Compruebe el registro de depuración en busca de mensajes de omisión/error.
2. Verifique que haya suficiente espacio en disco.
3. Cuente los archivos: deben coincidir (recuento original - recuento de destino) × (índices + 1)
4. Vuelva a importar y procesar los archivos que faltan.

### Problema: bordes oscuros o brillantes (el viñeteado sigue siendo visible)

**Posibles causas:**

* Corrección de viñeteado desactivada.
* Cámara/lente no incluida en la base de datos de perfiles Chloros.
* Viñeteado extremo que supera la capacidad de corrección.

**Soluciones:**

1. Compruebe que la corrección del viñeteado está habilitada en la configuración del proyecto.
2. Compruebe que el modelo de cámara se ha detectado correctamente.
3. Póngase en contacto con el servicio de asistencia de MAPIR si el viñeteado persiste.

### Problema: colores o valores incorrectos.

**Posibles causas:**

* No se han detectado objetivos de calibración.
* Se ha seleccionado un modelo de objetivo de calibración incorrecto.
* La calibración de la reflectancia está desactivada.
* Imágenes de objetivos de mala calidad.

**Soluciones:**

1. Compruebe que la calibración de la reflectancia está habilitada.
2. Compruebe los mensajes «Objetivo encontrado» en el registro de depuración.
3. Revise la calidad de la imagen del objetivo.
4. Vuelva a procesar con los objetivos adecuados marcados.

### Problema: los valores NDVI parecen incorrectos.

**Rangos NDVI esperados:**

* **Agua, rocas, suelo**: de -0,1 a 0,2.
* **Vegetación escasa/poco saludable**: de 0,2 a 0,4.
* **Vegetación moderada**: de 0,4 a 0,6.
* **Vegetación saludable y densa**: de 0,6 a 0,9.

**Si los valores están fuera de estos rangos:**

1. Compruebe que se ha aplicado la calibración de la reflectancia.
2. Compruebe que se ha incluido el registro del sensor de luz.
3. Compruebe que se han detectado los objetivos de calibración.
4. Asegúrese de que se ha detectado el modelo de cámara correcto.
5. Revise el momento y las condiciones de captura de la imagen objetivo.

***

## Uso de las imágenes procesadas

### Para fotogrametría/creación de ortomosaicos

**Flujo de trabajo recomendado:**

1. **Importe imágenes de reflectancia calibradas** al software de fotogrametría:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Conserve los metadatos EXIF**: asegúrese de que se conservan los datos GPS para el geoetiquetado.
3. **Flujos de trabajo calibrados**: utilice imágenes de reflectancia para obtener precisión científica.
4. **Procese mosaicos de índice**: Cree ortomosaicos NDVI a partir de imágenes índice individuales.
5. **Exporte GeoTIFF georreferenciado**: para su uso en aplicaciones SIG.

### Para análisis SIG

**Flujo de trabajo recomendado:**

1. **Cargue en QGIS, ArcGIS o similar**.
2. **Utilizar imágenes de reflectancia de 16 bits TIFF** para análisis multibanda.
3. **Utilizar imágenes índice** (NDVI, NDRE) como capas de vegetación listas para usar.
4. **Calculadora ráster**: combinar bandas para análisis personalizados.
5. **Exportar**: cree mapas de clasificación, detección de cambios y mapas de salud de la vegetación.

### Para análisis directo/informes

**Flujo de trabajo recomendado:**

1. **Utilice imágenes de índice con colores LUT** para informes visuales.
2. **Extraiga estadísticas**: media NDVI por campo/parcela.
3. **Series temporales**: comparar índices entre varias sesiones.
4. **Generar informes**: incluir mapas, estadísticas y visualizaciones.

***

## Archivado y copia de seguridad

### Estrategia de copia de seguridad recomendada

**Qué guardar:**

* ✅ **Imágenes RAW/JPG originales**: archivarlas en una unidad/nube independiente.
* ✅ **Resultados procesados**: conserve las imágenes calibradas y los índices
* ✅ **Archivo del proyecto**: contiene todos los ajustes para volver a procesar si es necesario
* ✅ **Registro de depuración**: documenta los detalles del procesamiento
* ✅ **Imágenes de calibración**: para verificación y reprocesamiento

**Recomendaciones de almacenamiento:**

* **Copia de seguridad inmediata**: disco duro externo
* **Archivo a largo plazo**: almacenamiento en la nube (Google Drive, Dropbox, etc.)
* **Datos críticos**: guarde 2-3 copias en diferentes ubicaciones

***

## Próximas ejecuciones de procesamiento

### Reutilización de la configuración del proyecto

Si va a procesar conjuntos de datos similares en el futuro:

1. **Guarde la plantilla del proyecto** (si aún no lo ha hecho)
2. **Cree un nuevo proyecto** utilizando la plantilla guardada
3. **Importe nuevas imágenes**
4. **Procese** con la misma configuración para mantener la coherencia

### Procesamiento por lotes de varias sesiones

Para varias sesiones/conjuntos de datos:

**Opción 1: GUI - Varios proyectos**

* Cree un proyecto independiente para cada sesión.
* Utilice una configuración de plantilla coherente.
* Procese uno por uno.

**Opción 2: Chloros CLI (solo Chloros+)**

* Automatice el procesamiento por lotes.
* Procese varias carpetas con scripts.
* Consulte la [Documentación de CLI](../CLI.md)

**Opción 3: Python SDK (solo Chloros+)**

* Control programático
* Integración con procesos de análisis
* Consulte la [Documentación de API](../api-python-sdk.md)

***

## Solución de problemas de posprocesamiento

### Reprocesamiento con diferentes ajustes

Si los resultados no son satisfactorios:

1. Conserve las imágenes originales (no las elimine nunca)
2. Abra el mismo proyecto en Chloros
3. Ajuste la configuración en el panel Configuración del proyecto
4. Vuelva a procesar: los resultados sobrescribirán los anteriores

### Procesamiento de un subconjunto de imágenes

Para volver a procesar solo imágenes específicas:

1. Cree un nuevo proyecto
2. Importe solo las imágenes que necesiten volver a procesarse
3. Utilice la misma plantilla de configuración
4. Procese un conjunto de datos más pequeño

### Obtener ayuda

Si tiene algún problema:

* 📧 **Correo electrónico**: info@mapir.camera (incluya el registro de depuración).
* 🌐 **Asistencia técnica**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact).
* 📚 **Preguntas frecuentes**: [Preguntas frecuentes](../faq.md)
* 📖 **Documentación**: [Manual de Chloros](../)

***

## Resumen: flujo de trabajo completo

Ya ha completado el flujo de trabajo completo de procesamiento de Chloros:

1. ✅ **Proyecto creado**: consulte [Proyectos](../projects.md)
2. ✅ **Añadido archivos** - Ver [Añadir archivos](adding-files-to-a-project.md)
3. ✅ **Ajustado la configuración** - Ver [Ajustar la configuración del proyecto](adjusting-project-settings.md)
4. ✅ **Objetivos marcados** - Véase [Selección de imágenes objetivo](choosing-target-images.md)
5. ✅ **Procesamiento iniciado** - Véase [Inicio del procesamiento](starting-the-processing.md)
6. ✅ **Progreso supervisado**: consulte [Supervisar el procesamiento](monitoring-the-processing.md)
7. ✅ **Resultados revisados**: esta página

**¡Sus imágenes multiespectrales calibradas y con corrección de reflectancia están listas para su análisis!**

***

## Recursos adicionales

### Funciones avanzadas

* [**Visor de imágenes**](../image-viewer-gui/opening-an-image-full-screen.md): visualización y análisis interactivos.
* [**Sandbox de índices/LUT**](../image-viewer-gui/index-lut-sandbox.md): pruebas de índices personalizados.
* [**Fórmulas de índice multiespectral**](../project-settings/multispectral-index-formulas.md): referencia completa del índice

### Automatización e integración

* [**Documentación de CLI**](../CLI.md): procesamiento por lotes desde la línea de comandos
* [**Python SDK**](../api-python-sdk.md) - Automatización programática
* [**Características de Chloros+**](../#chloros) - Capacidades de procesamiento avanzadas

### Asistencia y aprendizaje

* [**Preguntas frecuentes**](../faq.md) - Respuestas a preguntas comunes
* [**Objetivos de calibración**](../calibration-targets.md) - Comprensión de la calibración de la reflectancia
* [**Cámaras compatibles**](../supported-cameras.md) - Hardware compatible

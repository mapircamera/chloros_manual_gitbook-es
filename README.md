---
metaLinks: {}
---

# Introducción

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>Chloros es una aplicación de software de [MAPIR](https://www.mapir.camera) para procesar imágenes y otros datos de sensores.

***{% hint style="success" %}**Novedades de Chloros 1.1.0**: Compatibilidad nativa con Linux (amd64 y arm64), computación periférica NVIDIA Jetson, adaptación dinámica de computación, canal de procesamiento de 4 subprocesos, nuevos comandos y opciones de CLI. Consulte [Descargar](download.md) para ver el registro completo de cambios.
{% endhint %}

Chloros está disponible en 3 modos de aplicación:

## Chloros: Aplicación GUI de escritorio

Ventana independiente con todas las funciones. _Solo para Windows._

## [Chloros CLI: Interfaz de línea de comandos](CLI.md)

Procesamiento por lotes desde la línea de comandos. Perfecto para automatización, creación de scripts y funcionamiento sin interfaz gráfica. Disponible en **Windows, Linux amd64 y Linux arm64 (NVIDIA Jetson)**. _La CLI requiere una licencia Chloros+ para acceder._

## [Chloros API: Python SDK](api-python-sdk.md)

Interfaz programática Python para la automatización y los flujos de trabajo personalizados. Perfecta para procesos de investigación, integración con aplicaciones Python existentes y creación de herramientas personalizadas. Disponible en **todas las plataformas** a través de `pip install chloros-sdk`. _La API requiere una licencia Chloros+ para acceder._***

## Plataformas compatibles

| Plataforma | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11** | Sí | Sí | Sí |
| **Linux amd64 (x86_64)** | No | Sí | Sí |
| **Linux arm64 (NVIDIA Jetson)** | No | Sí | Sí |

Para obtener instrucciones de instalación de Linux, consulte la sección [Linux y computación en el borde](linux/linux-overview.md).

***

## Chloros+

Aunque Chloros es de uso gratuito para la mayoría de las tareas, es posible que desee más. Ahí es donde una licencia de pago para Chloros+ puede resultarle útil. Con una licencia de Chloros+ puedes desbloquear nuevas funciones como:

* **Procesamiento multihilo**: acelera considerablemente el procesamiento de imágenes en proyectos de mayor envergadura al procesar imágenes simultáneamente a través del canal de procesamiento.
* **Aceleración por GPU (CUDA)**: aprovecha las opciones actuales de mayor memoria de GPU para acelerar aún más el canal de procesamiento de imágenes. Recomendamos 4 GB o más de VRAM para obtener los mejores resultados.
* **Chloros+**[**CLI**](CLI.md)**Acceso**: ejecute Chloros+ desde la línea de comandos para automatizar e integrar en su propio software.
* **Chloros+**[**API**](api-python-sdk.md)**Acceso:** ejecute Chloros+ desde Python para el control programático, lo que permite una integración perfecta con sus procesos de investigación, flujos de trabajo de análisis de datos y aplicaciones personalizadas.
* **Uso en múltiples dispositivos**: cada licencia de Chloros+ permite registrar más de 2 dispositivos. Utilice su cuenta de MAPIR Cloud para gestionar los dispositivos registrados. Añada compatibilidad con más dispositivos actualizando su licencia de Chloros+.
* **Método avanzado de debayer con reconocimiento de texturas:** un debayer de alta calidad con reconocimiento de bordes combinado con un modelo de eliminación de ruido basado en IA/ML que elimina casi todo el ruido del proceso de debayer. 
* **Fórmulas de índices multiespectrales personalizadas:** introduzca índices multiespectrales personalizados en las calculadoras de ráster de Chloros, tanto para el procesamiento como para el entorno de pruebas de visualización de imágenes.
* **Linux y computación periférica:** ejecute Chloros en plataformas Linux x86_64 y ARM64, incluyendo NVIDIA Jetson, para el procesamiento de campo y periférico. Consulte la [Descripción general de Linux](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Precios y registro de Chloros+</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>

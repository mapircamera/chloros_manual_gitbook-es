---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Descargar

Descargue la última versión de Chloros para comenzar con el procesamiento de imágenes multiespectrales.

### Requisitos del sistema

| Requisito          | Mínimo                         | Recomendado                     |
| -------------------- | ------------------------------- | ------------------------------- |
| **Sistema operativo** | Windows 10 (64 bits)             | Windows 11 (64 bits)             |
| **Procesador**        | Intel Core i5 o equivalente     | Intel Core i7 o superior         |
| **Memoria (RAM)**     | 8 GB                             | 16 GB o más                    |
| **Tarjeta gráfica**    | Compatible con DirectX 11           | GPU NVIDIA con 4 GB+ de VRAM       |
| **Almacenamiento**          | 6 GB de espacio libre                  | SSD con 10 GB+ de espacio libre       |
| **Pantalla**          | 1920 x 1080                       | 2560 x 1440 o superior             |
| **Internet**         | Necesario para la activación de la licencia | Necesario para la activación de la licencia |

{% hint style=&quot;info&quot; %}
**Aceleración GPU**: Los usuarios de Chloros+ con GPU NVIDIA (4 GB+ VRAM) pueden utilizar la aceleración CUDA para un procesamiento significativamente más rápido. Los usuarios de Chloros+ también obtienen un procesamiento multihilo para una velocidad máxima.
{% endhint %}

***

## Descargar Chloros

### <a href="https://drive.google.com/file/d/1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4/view?usp=drive_link" class="button primary">Descargar Chloros aquí</a>

### Última versión estable

**Chloros Instalador para Windows*** **Versión**: 1.0.4
* **Fecha de lanzamiento**: 5 de enero de 2026
* **Tamaño del archivo (descarga)**: 1,8 GB
* **Tamaño del archivo (instalado)**: 5,7 GB
* **Tipo de archivo**: .exe (instalador de Windows)

#### **Pasos de instalación:**

1. Descargue el archivo `CHLOROS INSTALLER - CURRENT VERSION.exe`
2. Haga doble clic en el instalador para comenzar la instalación
3. Siga las instrucciones del asistente de instalación
4. Elija el directorio de instalación (por defecto: `C:\Program Files\[USER]\Chloros\`)
5. Complete la instalación y ejecute Chloros, Chloros (navegador) o Chloros CLI.
6. Inicie sesión con su [cuenta MAPIR Cloud Chloros+](https://cloud.mapir.camera/pricing) (o continúe con la versión gratuita).

{% hint style=&quot;success&quot; %}
El instalador añade automáticamente `chloros-cli` a la ruta PATH de su sistema para acceder a la línea de comandos.
{% endhint %}

***

## Recursos adicionales

### Python SDK

Para desarrolladores y flujos de trabajo de automatización, instale Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Documentación**: [API: Python SDK](api-python-sdk.md)**Requisitos**: Chloros Desktop debe estar instalado, se requiere iniciar sesión con la licencia Chloros+.***

## Qué incluye

La instalación de Chloros incluye:

* ✅ **Chloros**: interfaz gráfica con todas las funciones
* ✅ **Chloros (navegador)**: interfaz basada en web para sistemas con especificaciones más bajas
* ✅ **Chloros CLI**: interfaz de línea de comandos (requiere la licencia Chloros+)
* ✅ **Chloros SDK** - Python API (requiere licencia Chloros+)
* ✅ **Perfiles de cámara** - Plantillas de cámara MAPIR preconfiguradas***

## Actualiza a Chloros+

Desbloquea funciones avanzadas con una suscripción a Chloros+:

* 🚀 **Procesamiento multihilo**: procesa imágenes en paralelo
* ⚡ **Aceleración GPU (CUDA)**: aprovecha la potencia de la GPU NVIDIA
* 💻 **Acceso a CLI**: automatice con herramientas de línea de comandos
* 🐍 **Python SDK**: acceso programático a API
* 📱 **Múltiples dispositivos**: utilícelo en entre 2 y más de 10 dispositivos (dependiendo del plan)
* 🧮 **Fórmulas personalizadas**: cree índices multiespectrales personalizados

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Ver planes y precios de Chloros+</a></p>***

## Ayuda para la instalación

### Solución de problemas

**La instalación falla con el mensaje de error:**

* Asegúrese de tener derechos de administrador
* Desactive temporalmente el software antivirus
* Compruebe que cumple los requisitos mínimos del sistema

**La aplicación no se inicia:**

* Pruebe la versión Chloros (navegador).
* Compruebe que Windows 10/11 (64 bits) está instalado.
* Actualice los controladores gráficos.
* Compruebe el visor de eventos Windows para obtener detalles sobre el error.
* Póngase en contacto con el servicio de asistencia con los registros de errores.

**Problemas con la activación de la licencia:**

* Asegúrese de que la conexión a Internet está activa.
* Verifique las credenciales en [https://cloud.mapir.camera](https://cloud.mapir.camera).
* Compruebe que el cortafuegos no está bloqueando Chloros.
* Consulte [Chloros+ Inicio de sesión](chloros+-login.md) para obtener instrucciones detalladas

### Obtener asistencia

¿Necesita ayuda con la instalación o la configuración?

* 📧 **Correo electrónico**: info@mapir.camera
* 🌐 **Sitio web**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Documentación**: [Introducción](./)
* ❓ **Preguntas frecuentes**: [Preguntas frecuentes](faq.md)***

## Registro de cambios

<details>

<summary>Versión 1.0.4</summary>

#### **Fecha de lanzamiento**: 5 de enero de 2026**Nuevas funciones*** **Alternar imagen/metadatos**: se ha añadido una opción en el explorador de archivos para ver los metadatos de la imagen seleccionada en una tabla en lugar de en la cuadrícula de imágenes
* **Control deslizante de zoom de la cuadrícula de imágenes**: nuevo control deslizante de la interfaz de usuario para ajustar el tamaño de las miniaturas (también admite CTRL + rueda del ratón).
* **Botones de exportación de la cuadrícula de imágenes**: botones en la fila superior para cambiar las miniaturas de JPG a exportaciones procesadas (objetivos, reflectancia, índice, LUT).
* **Pestaña Mapa**: nuevo mapa 2D interactivo que muestra los marcadores de ubicación GPS de las imágenes.
  * Compatible con Google Maps y mosaicos de mapas ESRI (selecciona automáticamente el mejor servicio de mosaicos en función de la disponibilidad del nivel de zoom).
  * Vista previa de la miniatura al pasar el ratón por los marcadores del mapa.

**Corrección de errores*** Se ha mejorado la compatibilidad para instalar Chloros en ordenadores que no estén en inglés.

</details>

<details>

<summary>Versión 1.0.3</summary>

#### **Fecha de lanzamiento**: 20 de diciembre de 2025.**Nuevas funciones*** Lanzamiento inicial.

**Mejoras*** Lanzamiento inicial.

**Corrección de errores*** Lanzamiento inicial.

**Problemas conocidos*** Lanzamiento inicial.

</details>***

## Acuerdo de licencia**Software propietario** - Copyright (c) 2025 MAPIR Inc.

Queda prohibido el uso, la distribución o la modificación no autorizados.

**Versión gratuita**: disponible para uso personal y comercial con limitaciones de funciones.**Chloros+**: licencia basada en suscripción para funciones avanzadas e implementaciones comerciales.

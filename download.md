---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Descargar

Descarga la última versión de Chloros para empezar a trabajar con el procesamiento de imágenes multiespectrales.

### Requisitos del sistema

#### Windows

| Requisito          | Mínimo                                              | Recomendado                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Sistema operativo** | Windows 10 (64 bits)                                  | Windows 11 (64 bits)                                  |
| **Procesador**        | Intel Core i5 o equivalente                          | Intel Core i7 o superior                              |
| **Memoria (RAM)**     | 8 GB                                                  | 16 GB o más                                         |
| **Tarjeta gráfica**    | Compatible con DirectX 11                                | GPU NVIDIA con 4 GB o más de VRAM                            |
| **Almacenamiento**          | 6 GB de espacio libre                                       | SSD con 10 GB o más de espacio libre                            |
| **Pantalla**          | 1920x1080                                            | 2560x1440 o superior                                  |
| **Internet**         | Requerido para la activación de la licencia [opcional] Chloros+ | Requerido para la activación de la licencia [opcional] Chloros+ |

#### Linux amd64 (x86\_64)

| Requisitos       | Mínimos                    | Recomendados               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribución**  | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+             |
| **Procesador**     | x86\_64 (Intel/AMD)        | Intel Core i7 o superior   |
| **Memoria (RAM)**  | 8 GB                        | 16 GB o más              |
| **Tarjeta gráfica** | Ninguna (procesamiento por CPU)      | GPU NVIDIA con 4 GB+ de VRAM |
| **Almacenamiento**       | 2 GB de espacio libre             | SSD con 10 GB+ libres       |
| **Python**        | Python 3.7+ (para SDK)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Requisito      | Mínimo                      | Recomendado                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Plataforma**     | NVIDIA Jetson con JetPack 6 | Jetson Orin NX 16 GB o AGX Orin |
| **Memoria (RAM)** | 8 GB (GPU/CPU compartida)         | 16 GB+ compartida                    |
| **Almacenamiento**      | 2 GB de espacio libre               | SSD NVMe con 10 GB+ libres        |
| **Python**       | Python 3.7+ (para SDK)        | Python 3.10+                    |

{% hint style="info" %}
**Aceleración por GPU**: Los usuarios de Chloros+ con GPU NVIDIA pueden utilizar la aceleración CUDA para un procesamiento significativamente más rápido. Esto funciona tanto en Windows (GPU de escritorio) como en Linux (GPU de escritorio y NVIDIA Jetson). Los usuarios de Chloros+ también obtienen procesamiento multihilo para una velocidad máxima.
{% endhint %}

***

## Descargar Chloros

### Última versión estable (23 de marzo de 2026): Versión 1.1.0

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Descargar Chloros para Windows (.exe)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Descargar Chloros para Linux amd64 (.deb)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Descargar Chloros para Linux arm64 / Jetson (.deb)</a>

#### Instalador de Windows (GUI + CLI + Backend)

* **Tipo de archivo**: .exe (Instalador de Windows)**Pasos de instalación:**

1. Descargue el archivo .exe anterior
2. Haga doble clic en el instalador para iniciar la instalación
3. Siga las instrucciones del asistente de instalación
4. Elija el directorio de instalación (por defecto: `C:\Program Files\[USER]\Chloros\`)
5. Complete la instalación y ejecute Chloros o Chloros CLI
6. Inicie sesión con su [cuenta de MAPIR Cloud Chloros+](https://cloud.mapir.camera/pricing) (o continúe con la versión gratuita)

{% hint style="success" %}
El instalador añade automáticamente `chloros-cli` a la ruta PATH de su sistema para permitir el acceso desde la línea de comandos.
{% endhint %}

#### Linux amd64 (paquete .deb — CLI + Backend)

* **Tipo de archivo**: .deb (paquete Debian/Ubuntu)
* **Arquitectura**: x86_64 (amd64)

```bash
sudo dpkg -i chloros-amd64.deb
chloros-cli --version  # Verify installation
```

#### Linux arm64 — NVIDIA Jetson (paquete .deb — CLI + Backend)

* **Tipo de archivo**: .deb (JetPack 6)
* **Arquitectura**: aarch64 (arm64)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
chloros-cli --version  # Verify installation
```

Consulte [Instalación de Linux](linux/linux-installation.md) para obtener instrucciones detalladas de configuración y la [Guía de NVIDIA Jetson](linux/nvidia-jetson-guide.md) para obtener orientación específica sobre Jetson.

#### Python SDK (Todas las plataformas)

```bash
pip install chloros-sdk
```

Consulte [API : Python SDK](api-python-sdk.md) para obtener la documentación.

{% hint style="info" %}
**Usuarios de Linux**: El paquete `.deb` instala CLI y el backend. Python y SDK se instalan por separado mediante pip. No hay interfaz gráfica de usuario (GUI) para Linux; toda la interacción se realiza a través de CLI o SDK.
{% endhint %}

***

## Recursos adicionales

### Python SDK

Para desarrolladores y flujos de trabajo de automatización, instale Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Documentación**: [API: Python SDK](api-python-sdk.md)**Requisitos**: Debe estar instalado Chloros (instalador de Windows o paquete de Linux `.deb`), se requiere iniciar sesión con la licencia de Chloros+***

## Qué incluye

### Instalador Windows

* ✅ **Chloros GUI** - Interfaz gráfica con todas las funciones
* ✅ **Chloros CLI** - Interfaz de línea de comandos (requiere licencia Chloros+)
* ✅ **Chloros Backend** - Motor de procesamiento
* ✅ **Perfiles de cámara** - Plantillas de cámara MAPIR preconfiguradas

### Paquete .deb de Linux

* ✅ **Chloros CLI** - Interfaz de línea de comandos (requiere licencia Chloros+)
* ✅ **Chloros Backend** - Motor de procesamiento
* ✅ **Perfiles de cámara** - Plantillas de cámara MAPIR preconfiguradas
* ❌ Sin interfaz gráfica de usuario — Linux es solo CLI/SDK sin interfaz gráfica

### Python SDK (pip, todas las plataformas)

* ✅ **Chloros SDK** - Python API (requiere licencia Chloros+)***

## Actualizar a Chloros+

Desbloquea funciones avanzadas con una suscripción a Chloros+:

* 🚀 **Procesamiento multihilo** - Procesa imágenes en paralelo
* ⚡ **Aceleración por GPU (CUDA)** - Aprovecha la potencia de las GPU de NVIDIA
* 💻 **Acceso a CLI**: automatice con herramientas de línea de comandos
* 🐍 **Python SDK**: acceso programático a API
* 📱 **Múltiples dispositivos**: utilícelo en entre 2 y más de 10 dispositivos (dependiendo del plan)
* **🐻 Método avanzado de debayerización con reconocimiento de texturas**: un debayerizador de alta calidad con reconocimiento de bordes combinado con un modelo de eliminación de ruido basado en IA/ML que elimina casi todo el ruido de la debayerización.
* 🧮 **Fórmulas personalizadas**: cree índices multiespectrales personalizados

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Ver planes y precios de Chloros+</a></p>***

## Ayuda para la instalación

### Solución de problemas

**La instalación falla con el siguiente mensaje de error:**

* Asegúrate de tener derechos de administrador
* Desactiva temporalmente el software antivirus
* Comprueba que cumples los requisitos mínimos del sistema

**La aplicación no se inicia (Windows):**

* Compruebe que Windows 10/11 (64 bits) está instalado
* Actualice los controladores gráficos
* Consulte el Visor de eventos de Windows para ver los detalles del error
* Póngase en contacto con el servicio de asistencia con los registros de errores

**CLI no se inicia (Linux):**

* Comprueba que el paquete `.deb` esté instalado correctamente: `dpkg -l | grep chloros`
* Comprueba los permisos: `sudo chmod +x /usr/bin/chloros-cli`
* Ejecute el diagnóstico: `chloros-cli selftest`
* Compruebe si faltan bibliotecas: `ldd /usr/lib/chloros/chloros-backend | grep "not found"`

**Problemas con la activación de la licencia:**

* Asegúrese de que la conexión a Internet está activa
* Verifique las credenciales en [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Comprueba que el cortafuegos no esté bloqueando Chloros
* Consulta [Chloros+ Inicio de sesión](chloros+-login.md) para obtener instrucciones detalladas

### Obtener asistencia

¿Necesitas ayuda con la instalación o la configuración?

* 📧 **Correo electrónico**: info@mapir.camera
* 🌐 **Sitio web**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Documentación**: [Introducción](./)
* ❓ **Preguntas frecuentes**: [Preguntas frecuentes](faq.md)***

## Registro de cambios

<details>

<summary>Versión 1.1.0 (Última)</summary>

**Fecha de lanzamiento: marzo de 2026**

**Nuevas funciones*** **Compatibilidad con Linux** — CLI y SDK nativos para Linux amd64 (x86\_64) y arm64 (NVIDIA Jetson JetPack 6). Instalación mediante paquetes `.deb`.
* **Compatibilidad con NVIDIA Jetson** — Procesamiento optimizado para dispositivos periféricos Jetson Nano, Orin Nano, Orin NX y AGX Orin.
* **Adaptación dinámica del cálculo**: detección automática del hardware y optimización de la estrategia de procesamiento. Chloros se adapta a su hardware, desde un Jetson Nano hasta una estación de trabajo con múltiples GPU.
* **Canal de procesamiento de 4 subprocesos**: subprocesos simultáneos de detección, calibración, procesamiento y exportación con asignación dinámica de memoria de la GPU.
* **Nuevos comandos CLI**: `selftest` (diagnóstico del sistema) y `update` (gestión de actualizaciones de Linux).
* **Nuevos indicadores de proceso CLI** — `--debayer` (estándar/sensible a texturas), `--indices` (especificar índices), `--target` (buscar primero en la subcarpeta de destino para una detección más rápida).
* **Nuevos elementos del menú de la interfaz gráfica de usuario** — Ahora se puede acceder a «Añadir archivos», «Añadir carpeta» y «Iniciar/Detener procesamiento» desde el menú desplegable del menú principal.**Mejoras**

* Detección automática del backend multiplataforma (rutas Windows y Linux)
* Mejora de SDK y `get_status()` con seguimiento del progreso por subproceso
* Nuevas excepciones de SDK: `ChlorosConfigurationError`, `ChlorosAuthenticationError`
* Gestión térmica y limitación adaptativa para NVIDIA Jetson
* Gestión automática de la memoria con recurso de reserva OOM para el procesamiento en mosaico de la GPU

</details>

<details>

<summary>Versión 1.0.5</summary>

**Fecha de lanzamiento: 10 de febrero de 2026**

**Nuevas funciones*** **Método de despixelado sensible a texturas \[Solo Chloros+] -** Texture Aware utiliza un despixelado sensible a los bordes de alta calidad combinado con un modelo de eliminación de ruido de IA/ML que elimina casi todo el ruido del despixelado.
* **Compatibilidad con objetivos de calibración T4P*** **Procesamiento por GPU Chloros+ más rápido, mejor gestión de la memoria**

**Correcciones de errores*** Interfaz de usuario (GUI) completamente nueva; ahora debería funcionar en todos los ordenadores Windows.

</details>

<details>

<summary>Versión 1.0.4</summary>

**Fecha de lanzamiento: 5 de enero de 2026**

**Nuevas funciones*** **Alternancia entre imagen y metadatos**: Se ha añadido un botón en el explorador de archivos para ver los metadatos de la imagen seleccionada en una tabla en lugar de en la cuadrícula de imágenes
* **Control deslizante de zoom de la cuadrícula de imágenes**: Nuevo control deslizante en la interfaz de usuario para ajustar el tamaño de las miniaturas (también es compatible con CTRL + rueda del ratón)
* **Botones de exportación de la cuadrícula de imágenes**: Botones en la fila superior para cambiar las miniaturas de JPG a exportaciones procesadas (Objetivos, Reflectancia, Índice, LUT)
* **Pestaña Mapa**: Nuevo mapa 2D interactivo que muestra marcadores de ubicación GPS de las imágenes
  * Compatible con Google Maps y mosaicos de mapas ESRI (selecciona automáticamente el mejor servicio de mosaicos según la disponibilidad del nivel de zoom)
  * Vista previa de miniaturas al pasar el ratón por los marcadores del mapa

**Correcciones de errores*** Se ha mejorado la compatibilidad para instalar Chloros en ordenadores que no estén en inglés

</details>

<details>

<summary>Versión 1.0.3</summary>

**Fecha de lanzamiento: 20 de diciembre de 2025**

**Nuevas funciones*** Lanzamiento inicial

**Mejoras*** Lanzamiento inicial

**Correcciones de errores*** Lanzamiento inicial

**Problemas conocidos*** Lanzamiento inicial

</details>***

## Acuerdo de licencia**Software propietario** - Copyright (c) 2026 MAPIR Inc.

Queda prohibido el uso, la distribución o la modificación no autorizados.

**Versión gratuita**: Disponible para uso personal y comercial con limitaciones de funciones**Chloros+**: Licencia basada en suscripción para funciones avanzadas e implementaciones comerciales

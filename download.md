---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Descargar

Descarga la última versión de Chloros
para empezar a trabajar con el procesamiento de imágenes multiespectrales.

### Requisitos del sistema

#### Windows


| Requisito          | Mínimo                                              | Recomendado                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Sistema operativo** | Windows
10 (64 bits)                                  | Windows
11 (64 bits)                                  |
| **Procesador**        | Intel Core i5 o equivalente                          | Intel Core i7 o superior                              |
| **Memoria (RAM)**     | 8 GB                                                  | 16 GB o más                                         |
| **Tarjeta gráfica**    | Compatible con DirectX 11                                | GPU NVIDIA con 4 GB o más de VRAM                            |
| **Almacenamiento**          | 6 GB de espacio libre                                       | SSD con 10 GB o más de espacio libre                            |
| **Pantalla**          | 1920x1080                                            | 2560x1440 o superior                                  |
| **Internet**         | Necesario para la activación de la licencia [opcional] de Chloros
+ | Necesario para la activación de la licencia [opcional] de Chloros
+ |

#### Linux
amd64 (x86_64)

| Requisitos       | Mínimos                    | Recomendados               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribución**  | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS      |
| **Procesador**     | x86_64 (Intel/AMD)        | Intel Core i7 o superior   |
| **Memoria (RAM)**  | 8 GB                        | 16 GB o más              |
| **Tarjeta gráfica** | Ninguna (procesamiento por CPU)      | GPU NVIDIA con 4 GB o más de VRAM |
| **Almacenamiento**       | 2 GB de espacio libre             | SSD con 10 GB o más de espacio libre       |
| **Python**        | Python
3.7+ (para el SDK
)      | Python
3.10+              |

#### Linux
arm64 (NVIDIA Jetson)

| Requisitos      | Mínimos                      | Recomendados                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Plataforma**     | NVIDIA Jetson con JetPack 6 | Jetson Orin NX de 16 GB o AGX Orin |
| **Memoria (RAM)** | 8 GB (compartida entre GPU y CPU)         | 16 GB o más compartidos                    |
| **Almacenamiento**      | 2 GB de espacio libre               | SSD NVMe con 10 GB o más de espacio libre        |
| **Python**       | Python
3.7 o superior (para el SDK
)        | Python
3.10 o superior                    |

{% hint style="info" %}
**Aceleración por GPU**: Los usuarios de Chloros
+ con GPU de NVIDIA pueden utilizar la aceleración CUDA para un procesamiento significativamente más rápido. Esto funciona con GPU de escritorio de NVIDIA en Windows
y Linux
amd64, y en NVIDIA Jetson con JetPack 6. Los usuarios de Chloros
+ también se benefician del procesamiento multihilo para obtener la máxima velocidad.
{% endhint %}

***

## Descargar Chloros


### Última versión estable: Versión 1.3.0 (11 de noviembre de 2026)

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Descargar Chloros para Windows (.exe)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Descargar Chloros para Linux amd64 (.deb)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Descargar Chloros para Linux arm64 / Jetson (.deb)</a>


#### Instalador de Windows
(GUI + CLI
+ Backend)

* **Tipo de archivo**: .exe (Instalador de Windows
)

**Pasos de instalación:**

1. Descarga el archivo .exe anterior
2. Haz doble clic en el instalador para iniciar la instalación
3. Sigue las instrucciones del asistente de instalación
4. Elige el directorio de instalación (por defecto: `C:\Program Files\Chloros\`)
5. En la página **Seleccionar tareas adicionales**, mantén marcadas ambas opciones de**Configuración del sistema** (recomendado; véase más abajo)
6. Completa la instalación e inicia Chloros
o Chloros
CLI
7. Inicia sesión con tu [cuenta de MAPIR
Cloud Chloros
+](https://cloud.mapir.camera/pricing) (o continúa con la versión gratuita)

**Opciones de configuración del sistema** (ambas marcadas por defecto):

| Opción, tal y como aparece en el instalador | Qué hace | Desactivar la instalación silenciosa |
| --- | --- | --- |
| **Habilitar rutas largas de Windows** (permite que las carpetas de proyectos tengan más de 260 caracteres; recomendado) | Establece el parámetro `LongPathsEnabled` de Windows
para todo el equipo en 1. Windows
viene de fábrica con esta opción desactivada, por lo que ninguna ruta de archivo puede superar los 260 caracteres; los productos Chloros
se ubican hasta unos 130 caracteres por debajo de la carpeta del proyecto, por lo que, si las rutas largas están desactivadas, se rechaza una carpeta de proyecto más profunda. Surte efecto para los programas iniciados tras la instalación. Permanece activado tras la desinstalación, ya que otros programas pueden depender de ello. | `/MERGETASKS="!longpaths"` |
| **Excluir el backend de Chloros
del análisis de Microsoft Defender** (captura de cámara mucho más rápida; recomendado) | Añade una exclusión del proceso de Microsoft Defender para `chloros-backend.exe` en la carpeta de instalación, de modo que la protección en tiempo real no analice todos los archivos que escribe el backend, independientemente de dónde los escriba. A altas velocidades de captura, el análisis suponía aproximadamente un 30 % de la velocidad de captura (77,7 frente a 52,7 capturas por segundo en una captura continua con 9 cámaras). No se excluye ningún otro programa, y la exclusión se elimina al desinstalar. Si Defender no está presente o está bloqueado por una política, este paso falla de forma silenciosa y la instalación continúa. | `/MERGETASKS="!defenderexclusion"` |

Una instalación silenciosa (`/SILENT` o `/VERYSILENT`) aplica ambas opciones, a menos que se desactive con el valor `/MERGETASKS` que se muestra.

{% hint style="success" %}
El instalador añade la carpeta CLI
(`C:\Program Files\Chloros\cli\`) a tu sistema `PATH` una sola vez; al reinstalar o actualizar ya no se añade una entrada duplicada, y se eliminan los duplicados que hubieran añadido instaladores anteriores. Abre un nuevo terminal tras la instalación para utilizar `chloros-cli`. El instalador también añade dos reglas del cortafuegos de Windows
para el backend, en cada perfil de red: **Backend de Chloros
(UDP-In)**, para flujos de cámaras, PTP, mDNS y el flujo de datos DAQ-E, y**Backend de Chloros
(TCP-In)**, limitado a la subred local, para una actualización del firmware de DAQ-E, en la que el sensor se conecta de nuevo a este ordenador para recopilar la imagen. Ambas se eliminan al desinstalar el programa.
{% endhint %}

#### Linux
amd64 (paquete .deb — CLI
+ backend)

* **Tipo de archivo**: .deb (paquete Debian/Ubuntu)
* **Arquitectura**: x86_64 (amd64)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

#### Linux
arm64 — NVIDIA Jetson (paquete .deb — CLI
+ backend)

* **Tipo de archivo**: .deb (JetPack 6)
* **Arquitectura**: aarch64 (arm64)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

Utiliza el nombre del archivo que hayas descargado realmente y mantén el `./` para que `apt` instale ese archivo en lugar de buscar un paquete por su nombre. Instálalo con `apt install`, no con `dpkg -i`: apt también instala los requisitos del Python
SDK
, que el paquete recomienda en lugar de exigir, y `dpkg -i` los omite. Desde la versión 1.3.0, el backend y la CLI
se instalan sin descomprimir bajo `/usr/lib/chloros/`, por lo que se inician sin necesidad de descomprimirse primero.

El paquete Jetson utiliza la versión de PyTorch para CUDA de JetPack 6, por lo que el procesamiento, incluido el «debayering» con reconocimiento de texturas, se ejecuta en la GPU del Jetson. Utiliza las bibliotecas CUDA y cuDNN que instala JetPack 6 (`sudo apt install nvidia-jetpack` en una imagen que carece de ellas).

Consulta [Instalación en Linux
](linux/linux-installation.md) para obtener instrucciones detalladas de configuración y la [Guía de NVIDIA Jetson](linux/nvidia-jetson-guide.md) para obtener orientación específica sobre Jetson.

#### Python
SDK
(todas las plataformas)

Cada instalador incluye un paquete «wheel» `chloros_sdk` correspondiente, por lo que la versión del SDK
siempre coincide con la GUI/CLI
/backend instalado. En Windows
, el instalador lo instala automáticamente en el Python
del sistema. En Linux
, el `.deb` instala el SDK
para el propio sistema `python3` (en `/usr/lib/python3/dist-packages`), por lo que `import chloros_sdk` funciona inmediatamente después de la instalación, sin necesidad de pip ni de conexión a red. Solo un virtualenv, que no tiene acceso a los paquetes del sistema, necesita que se le instale el archivo wheel incluido:

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Para los hosts que solo tienen pip (sin el paquete Chloros
instalado; en cuyo caso el SDK
controla un backend en otra máquina), el SDK
también está disponible en PyPI:

```bash
pip install chloros-sdk
```

Consulta [API
: Python
SDK
](api-python-sdk.md) y la [Referencia del SDK
](reference/sdk-reference.md) para obtener documentación.

{% hint style="info" %}
**Usuarios de Linux**: El paquete `.deb` instala la CLI
, el backend y el SDK
de Python
. No hay interfaz gráfica de usuario (GUI) para Linux
; toda la interacción se realiza a través de la CLI
o el SDK
.
{% endhint %}

***

## Recursos adicionales

### SDK
Python
de Chloros


Para desarrolladores y flujos de trabajo de automatización, el Chloros
Python
SDK
de Windows
se incluye con todos los instaladores: el instalador de Windows
lo instala en tu Python
, y el de Linux
`.deb` lo instala para el sistema `python3`. En un equipo que no tenga el paquete Chloros
(uno que controle un backend en otro equipo), instálalo desde PyPI:

```bash
pip install chloros-sdk
```

**Documentación**: [API
: Python
SDK
](api-python-sdk.md)

**Requisitos**: un backend de Chloros
que controlar (el instalador de Windows o el paquete `.deb` para Linux
instalan uno), y una cuenta de Chloros
+ en un plan de pago

***

## Qué incluye

### Instalador de Windows


* ✅ **Interfaz gráfica de usuario de Chloros**: interfaz gráfica con todas las funciones
* ✅ **CLI
de Chloros
**: interfaz de línea de comandos (requiere licencia de Chloros
+)
* ✅ **Backend de Chloros**: motor de procesamiento
* ✅ **Python
SDK
**: se instala en tu Python
si lo tienes (requiere licencia de Chloros
+)
* ✅ **Perfiles de cámara**: plantillas de cámara MAPIR
preconfiguradas

### Paquete .deb para Linux


* ✅ **Chloros
CLI
**: interfaz de línea de comandos (requiere licencia Chloros
+)
* ✅ **Chloros
Backend**: motor de procesamiento
* ✅ **Python
SDK
**: instalado para el sistema `python3` (requiere licencia Chloros
+)
* ✅ **Perfiles de cámara**: plantillas de cámara MAPIR
preconfiguradas
* ❌ Sin interfaz gráfica de usuario (GUI): Linux
es un sistema sin interfaz gráfica, solo CLI
/SDK


### Python
SDK
(PyPI, para máquinas sin paquete Chloros
)

* ✅ **Chloros
SDK
**: Python
API
(requiere licencia de Chloros
+)

***

## Actualizar a Chloros+

Accede a funciones avanzadas con una suscripción a Chloros
+:

* 🚀 **Procesamiento multihilo**: procesa imágenes en paralelo
* ⚡ **Aceleración por GPU (CUDA)** - Aprovecha la potencia de las GPU de NVIDIA
* 💻 **Acceso a la CLI**: automatiza con herramientas de línea de comandos
* 🐍 **Python
SDK
**: acceso programático a la API

* 📱 **Múltiples dispositivos**: utilízalo en entre 2 y 10 o más dispositivos (dependiendo del plan)
* **🐻 Método avanzado de eliminación de mosaico sensible a las texturas**: un método de eliminación de mosaico de alta calidad sensible a los bordes, combinado con un modelo de eliminación de ruido basado en IA/ML que elimina casi todo el ruido propio de la eliminación de mosaico.
* 🧮 **Fórmulas personalizadas**: crea índices multiespectrales personalizados

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Ver planes y precios de Chloros+</a></p>***

## Ayuda para la instalación

### Solución de problemas

**La instalación falla con el siguiente mensaje de error:**

* Asegúrate de tener derechos de administrador
* Desactiva temporalmente el software antivirus
* Comprueba que cumples los requisitos mínimos del sistema

**La aplicación no se inicia (en Windows

):**

* Comprueba que Windows

10/11 (64 bits) esté instalado
* Actualiza los controladores gráficos
* Consulta el Visor de eventos de Windows

para ver los detalles del error
* Ponte en contacto con el servicio de asistencia con los registros de errores

**La CLI

no se inicia (Linux

):**

* Comprueba que el paquete `.deb` esté instalado correctamente: `dpkg -l | grep chloros`
* Comprueba los permisos: `sudo chmod +x /usr/bin/chloros-cli`
* Ejecuta el diagnóstico: `chloros-cli selftest`
* Comprueba si faltan bibliotecas: `ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"` (a partir de la versión 1.3.0, `/usr/lib/chloros/chloros-backend` es un pequeño script de inicio; el programa en sí se encuentra en `/usr/lib/chloros/backend/`)

**Problemas con la activación de la licencia:**

* Asegúrate de que la conexión a Internet esté activa
* Verifica las credenciales en [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Comprueba que el cortafuegos no esté bloqueando Chloros

* Consulta [Chloros

+ Inicio de sesión](chloros+-login.md) para obtener instrucciones detalladas

### Obtener asistencia

¿Necesitas ayuda con la instalación o la configuración?

* 📧 **Correo electrónico**: info@mapir.camera
* 🌐 **Página web**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Documentación**: [Primeros pasos](./)
* ❓ **Preguntas frecuentes**: [Preguntas frecuentes](faq.md)***

## Actualizaciones de software

Chloros

comprueba si hay actualizaciones, avisa cuando hay una nueva versión disponible y proporciona un enlace a esta página de descarga; para actualizar, basta con ejecutar el nuevo instalador firmado. Tu configuración, tus proyectos y tus datos de inicio de sesión se conservan tras las actualizaciones. En Linux

y Jetson, `chloros-cli update` comprueba si hay una versión más reciente y ofrece la posibilidad de descargar e instalar la versión correspondiente `.deb` (este comando solo está disponible en Linux

).

***

## Registro de cambios**Versión 1.3.0 (última, 11 de noviembre de 2026)**: consulta**Novedades de Chloros

1.3.0** en la página [Primeros pasos](./) para ver la lista completa.**Versión 1.2.0**— consulta**Novedades de Chloros

1.2.0** en la página [Primeros pasos](./).

<details>

<summary>Versión 1.0.5</summary>

**Fecha de lanzamiento: 10 de febrero de 2026**

**Nuevas funciones*** **Método de despixelación sensible a texturas [solo Chloros

+] —** «Texture Aware» utiliza un método de despixelación de alta calidad sensible a los bordes, combinado con un modelo de eliminación de ruido basado en IA/ML que elimina casi todo el ruido de la despixelación.
* **Compatibilidad con los objetivos de calibración T4P*** **Procesamiento más rápido de Chloros

+ en la GPU y mejor gestión de la memoria**

**Correcciones de errores*** Interfaz de usuario (GUI) completamente nueva; ahora debería funcionar en todos los ordenadores con Windows

.

</details>

<details>

<summary>Versión 1.0.4</summary>

**Fecha de lanzamiento: 5 de enero de 2026**

**Nuevas funciones*** **Alternancia entre imagen y metadatos**: se ha añadido un botón en el explorador de archivos para ver los metadatos de la imagen seleccionada en una tabla en lugar de en la cuadrícula de imágenes
* **Control deslizante de zoom de la cuadrícula de imágenes**: nuevo control deslizante en la interfaz de usuario para ajustar el tamaño de las miniaturas (también es compatible con CTRL + rueda del ratón)
* **Botones de exportación de la cuadrícula de imágenes**: Botones en la fila superior para cambiar las miniaturas de JPG a exportaciones procesadas (Objetivos, Reflectancia, Índice, LUT)
* **Pestaña «Mapa»**: Nuevo mapa 2D interactivo que muestra marcadores de ubicación GPS de las imágenes
  * Compatible con Google Maps y mosaicos de mapas ESRI (selecciona automáticamente el mejor servicio de mosaicos en función de la disponibilidad del nivel de zoom)
  * Vista previa de las miniaturas al pasar el ratón por encima de los marcadores del mapa

**Corrección de errores*** Se ha mejorado la compatibilidad para instalar Chloros en ordenadores que no estén en inglés

</details>

<details>

<summary>Versión 1.0.3</summary>

**Fecha de lanzamiento: 20 de diciembre de 2025**

**Nuevas funciones*** Lanzamiento inicial

**Mejoras*** Lanzamiento inicial

**Corrección de errores*** Lanzamiento inicial

**Problemas conocidos*** Lanzamiento inicial

</details>

***

## Acuerdo de licencia**Software propietario** - Copyright (c) 2026 MAPIR Inc.

Queda prohibido el uso, la distribución o la modificación no autorizados.

**Versión gratuita**: Disponible para uso personal y comercial con limitaciones de funciones**Chloros+**: Licencia por suscripción para funciones avanzadas e implementaciones comerciales

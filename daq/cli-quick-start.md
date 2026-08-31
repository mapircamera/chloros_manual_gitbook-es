# CLI: Guía de inicio rápido (pool-*)

Las unidades `chloros-cli` suministradas controlan los sensores DAQ a través de la **familia de comandos `daq pool-*`** — unos clientes ligeros HTTP que controlan el sensor a través del grupo de sensores persistente del backend Chloros. El backend se encarga del transporte, por lo que la interfaz gráfica de usuario, el script CLI y los scripts SDK comparten un único identificador activo en lugar de disputarse el puerto. Todo lo que un cliente necesita está disponible a través de `pool-*`: conectarse, transmitir, grabar archivos `.daq` calibrados e intercambiar perfiles de condensadores.

`pool-*` es también la **única** interfaz de adquisición de datos (DAQ) en las versiones publicadas. `chloros-cli daq --help` muestra la lista de subcomandos de `pool-*`, y al ejecutar un subcomando de DAQ de hardware directo en una versión publicada, el programa se cierra con un error explícito en el que se indica el paquete que falta y se remite de nuevo a `pool-*`; nada falla de forma silenciosa. (Los comandos de acceso directo al hardware solo se ejecutan desde una copia del código fuente de MAPIR; `pip install chloros-sdk` tampoco los proporciona).

***

## Requisitos previos

* **El backend Chloros debe estar en ejecución**: los comandos `pool-*` son clientes de HTTP, no controladores de hardware. En Windows, inicia la aplicación de escritorio Chloros (esta inicia el backend). En un Linux/Jetson sin pantalla, activa el servicio: `sudo systemctl enable --now chloros-backend.service`.
* **Inicio de sesión en Chloros+ (nivel de pago)**: ejecuta primero `chloros-cli login`. La aplicación de las restricciones se realiza en el lado del servidor: sin iniciar sesión, los comandos fallan con `401 AUTH_REQUIRED`; en el nivel gratuito (Iron), fallan con `403 PLAN_UPGRADE_REQUIRED`.
* Los comandos se dirigen a `http://127.0.0.1:5000` por defecto; la familia `daq pool-*` respeta la variable de entorno `CHLOROS_BACKEND_URL` si tu backend se ejecuta en otro lugar.

***

## Una sesión de cinco minutos

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)

# 2. List the pool — this shows the sensor_id used by every command below
chloros-cli daq pool-list

# 3. Read the most recent calibrated spectrum frame (add --json for scripting)
chloros-cli daq pool-latest --sensor-id daq-e-def330 --json

# 4. Record a calibrated .daq file for 60 seconds
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 60 \
  --device-name "field-A"

# 5. Release the sensor when done
chloros-cli daq pool-disconnect --sensor-id daq-e-def330
```

***

## `pool-connect` — abrir un sensor en el grupo

| Variante | Significado |
| --- | --- |
| `daq pool-connect` | Detección inteligente: busca cualquier DAQ en esta máquina. |
| `daq pool-connect --port PORT` | DAQ-U en un puerto serie específico (p. ej., `COM3`, `/dev/ttyUSB0`). |
| `daq pool-connect --ble` | DAQ-M a través de BLE, MAC escaneado automáticamente. |
| `daq pool-connect --mac MAC` | DAQ-M en una dirección MAC BLE conocida (implica `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E con un nombre de host o una IP conocidos: **la ruta fiable**. |
| `daq pool-connect --eth` | DAQ-E con detección automática (mDNS, con un recurso de ARP). Véase la advertencia más abajo. |

Parámetros de ajuste, todos opcionales:

| Parámetro | Significado |
| --- | --- |
| `--integration-time MS` / `-t MS` | Tiempo de integración manual en milisegundos. |
| `--frame-avg N` / `-f N` | Promedio de fotogramas por espectro notificado. |
| `--no-ae` | Desactivar la exposición automática (la AE está activada por defecto). |
| `--no-stream` | Conectarse sin iniciar la transmisión (reanudar más tarde con `pool-stream --start`). |
| `--cap-id CAP` | Perfil de corrección de cap; el valor predeterminado del backend es `sunshine_cosine`. Véase [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap). |

{% hint style="warning" %}
**Advertencia sobre la detección automática de `--eth`.** En un host con múltiples interfaces de red (más de una interfaz de red activa), el *primer* `pool-connect --eth` tras el arranque puede no encontrar nada, incluso si el sensor está en buen estado: la búsqueda de detección puede pasar por alto la interfaz del sensor mientras la caché ARP está inactiva. Si `--eth` no encuentra nada, vuelve a intentarlo o omite por completo la detección con `--eth-host <ip-or-hostname>`, que es la ruta fiable en máquinas con múltiples interfaces de red. El nombre de host del DAQ-E es `daq-e-<id>.local` (p. ej., `daq-e-def330.local`); su dirección IP sin prefijo también funciona.
{% endhint %}

## `pool-list` — ver qué está conectado

Muestra todos los sensores del grupo de backend, incluido el `sensor_id` que necesitan todos los demás comandos:

| Modelo | Formato `sensor_id` | Ejemplo |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5 octetos con guiones | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

## `pool-latest` — leer tramas de espectro

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

Devuelve el fotograma más reciente, o los fotogramas más recientes `--recent N`; `--json` genera una salida legible por máquina para la creación de scripts. Los fotogramas son valores de irradiancia espectral (W/m²/nm) calibrados radiométricamente en una cuadrícula de 135 puntos, de 340 a 1010 nm, con el perfil de la tapa del sensor ya aplicado. Para obtener valores cuantitativos de irradiancia, se debe calcular la media de al menos 15 segundos de fotogramas; se trata de una característica del instrumento, no de un defecto.

## `pool-stream` — pausar o reanudar la transmisión

```bash
chloros-cli daq pool-stream --sensor-id daq-e-def330 --stop    # pause
chloros-cli daq pool-stream --sensor-id daq-e-def330 --start   # resume
```

## `pool-record` — grabar un archivo `.daq`

```bash
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
  --output ~/Documents/spectra --device-name "rooftop-A"
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

| Indicador | Predeterminado | Significado |
| --- | --- | --- |
| `--duration SEC` / `-d SEC` | `0` | Duración de la grabación en segundos; `0` significa que se ejecutará hasta que se introduzca `--stop`. |
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | Directorio de salida, resuelto **en la máquina que ejecuta el backend**. |
| `--device-name NAME` | — | Etiqueta almacenada con la grabación. |
| `--stop` | — | Detener una grabación en curso. |

{% hint style="info" %}
La grabación se realiza en el backend, por lo que el archivo `.daq` se guarda en el sistema de archivos de **la máquina del backend** —por defecto en `~/Documents/DAQ Live View/` allí, no necesariamente donde se ejecutó el CLI. Los nombres de los archivos incluyen el ID del sensor y una marca de tiempo.
{% endhint %}

## `pool-set-cap` — declarar la tapa instalada

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

El ID de la tapa selecciona el perfil de corrección medido en fábrica que se aplica a cada espectro, y **debe coincidir con la tapa instalada físicamente en el sensor**; ni el sensor ni el software pueden detectar la tapa por sí mismos, y la selección queda registrada en cada archivo `.daq`. El valor predeterminado en todos los casos es `sunshine_cosine` (todos los DAQ se suministran con la tapa correctora de coseno Sunshine instalada, con una atenuación de aproximadamente 12× por diseño; un cambio de tapa no declarado corrige erróneamente los espectros en más o menos ese factor).

| `--cap-id` | Disponible en |
| --- | --- |
| `sunshine_cosine` (por defecto) | DAQ-U, DAQ-M, DAQ-E |
| `fov_15`, `fov_45`, `fov_90` | DAQ-U, DAQ-E |
| `fov_30`, `fov_60` | Solo DAQ-U |
| `none` | Solo DAQ-E — véase la nota |

Si el ID de la tapa no pertenece al conjunto del sensor, se rechaza la conexión con un error claro. `none` (DAQ-E) significa que la tapa se ha retirado físicamente; sigue aplicando un perfil geométrico de fábrica para el difusor de cristal empotrado del DAQ-E, por lo que no es una operación nula, y un DAQ-E sin tapa es una configuración de banco, no una configuración de campo compatible. (Un DAQ-U sin tapa es verdaderamente «desnudo» y no necesita ningún perfil de corrección; el DAQ-M se utiliza con su tapa Sunshine.)

## `pool-disconnect` — liberar sensores

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

***

## Resumen de comandos

| Comando | Finalidad |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | Abrir un sensor en el grupo de sensores del backend. |
| `daq pool-list` | Mostrar todos los sensores del grupo con su `sensor_id`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Las N tramas de espectro calibradas más recientes. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Reanudar/pausar la transmisión. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--stop]` | Iniciar/detener una grabación `.daq` (del lado del backend). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Cambiar el perfil de corrección de recorte en tiempo de ejecución. |
| `daq pool-disconnect --sensor-id ID [--all]` | Liberar un sensor o todos ellos. |

***

## Solución de problemas en la primera conexión del DAQ-E

1. El DAQ-E no tiene LED de estado: comprueba que está alimentado mediante el indicador de PoE/enlace del conmutador o del puerto del inyector, y espera unos segundos tras el encendido para que arranque y se conecte a la red.
2. El equipo backend debe estar en el **mismo dominio de difusión** que el sensor; mDNS no atraviesa routers.
3. En Windows, acepta la solicitud del cortafuegos Defender en la primera ejecución (mDNS UDP 5353, datos del DAQ-E UDP 5002, PTP UDP 319/320).
4. ¿Sigue sin haber respuesta de `--eth`? Utiliza `--eth-host` con el nombre de host de la unidad (`daq-e-<id>.local`) o su IP; esta es la ruta más fiable, especialmente en hosts con múltiples direcciones IP.

***{% hint style="info" %}**Consejo para los asistentes de IA.** Todas las páginas de este manual se sirven en formato Markdown sin procesar: añade `.md` al slug en minúsculas de una página, URL (esta página: `https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`); el índice legible por máquina es `https://mapir.gitbook.io/chloros/llms.txt`. Para obtener la documentación completa a nivel de opciones de `chloros-cli daq` y de todas las demás familias de comandos, consulta la [Referencia de CLI](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`); la ruta de Python es `chloros_sdk.connect_daq_sensor()` en la [Referencia de SDK](../reference/sdk-reference.md).
{% endhint %}

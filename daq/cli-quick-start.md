# Guía de inicio rápido de CLI (pool-*)

El `chloros-cli` incluido controla los sensores DAQ a través de la familia de comandos **`daq pool-*`**: clientes HTTP ligeros que gestionan el sensor a través del grupo de sensores persistente del backend de Chloros. El backend se encarga del transporte, por lo que la interfaz gráfica de usuario (GUI), la CLI y los scripts del SDK comparten un único identificador activo en lugar de disputarse el puerto. Todo lo que un cliente necesita está disponible a través de `pool-*`: conectarse, transmitir, grabar archivos `.daq` calibrados e intercambiar perfiles de capacidad.

`pool-*` es la interfaz DAQ compatible en las versiones publicadas. `chloros-cli daq --help` muestra la lista de subcomandos de `pool-*`, y `daq discover` / `daq list` ejecutan el mismo escaneo que `pool-discover`. Cuando no está disponible un subcomando de DAQ de acceso directo al hardware, el programa se cierra con un error explícito en el que se indica el paquete que falta y se remite al usuario a `pool-*`; ningún error pasa desapercibido. (`pip install chloros-sdk` tampoco proporciona los comandos directos de hardware).

***

## Requisitos previos

* **El backend de Chloros debe estar en ejecución**: los comandos `pool-*` son clientes HTTP, no controladores de hardware. En Windows, inicia la aplicación de escritorio de Chloros (esta inicia el backend). En Linux/Jetson sin interfaz gráfica, habilita el servicio como tu usuario: `sudo systemctl enable --now chloros-backend@$USER.service` (consulta [Instalación en Linux](../linux/linux-installation.md#always-on-ptp-for-headless-hosts)).
* **Inicio de sesión en Chloros+ (nivel de pago)**: ejecuta primero `chloros-cli login`. La aplicación de las restricciones se realiza del lado del servidor: sin iniciar sesión, los comandos fallan con `401 AUTH_REQUIRED`; en el nivel gratuito (Iron), fallan con `403 PLAN_UPGRADE_REQUIRED`.
* Los comandos se dirigen a `http://127.0.0.1:5000` por defecto; la familia `daq pool-*` respeta la variable de entorno `CHLOROS_BACKEND_URL` si tu backend se ejecuta en otro lugar.

***

## Una sesión de cinco minutos

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)
#    (a DAQ-M's BLE MAC is not printed anywhere: find it with `chloros-cli daq pool-discover --only ble`)

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
| `daq pool-connect --eth` | DAQ-E con detección automática (mDNS, con ARP como alternativa). Véase la advertencia más abajo. |

Parámetros de ajuste, todos opcionales:

| Parámetro | Significado |
| --- | --- |
| `--integration-time MS` / `-t MS` | Tiempo de integración manual en milisegundos. |
| `--frame-avg N` / `-f N` | Promedio de fotogramas por espectro notificado. |
| `--no-ae` | Desactivar la exposición automática (la AE está activada por defecto). |
| `--no-stream` | Conectarse sin iniciar la transmisión (reanudar más tarde con `pool-stream --start`). |
| `--cap-id CAP` | Perfil de corrección de cap; el valor predeterminado del backend es `sunshine_cosine`. Véase [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap). |

Al conectar un DAQ-E que ya se encuentra en el grupo bajo otra de sus direcciones —su nombre de host `daq-e-<id>.local` o su IP—, se devuelve la entrada existente del grupo en lugar de abrir la unidad por segunda vez.

{% hint style="warning" %}
**Advertencia sobre la detección automática de `--eth`.** En un host con múltiples interfaces de red (más de una interfaz de red activa), el *primer* `pool-connect --eth` tras el arranque puede no encontrar nada, incluso si el sensor está en buen estado: la búsqueda de detección puede pasar por alto la interfaz del sensor mientras la caché ARP está inactiva. Si `--eth` no encuentra nada, vuelve a intentarlo o omite por completo el proceso de detección con `--eth-host <ip-or-hostname>`, que es la ruta fiable en máquinas con múltiples interfaces de red. El nombre de host del DAQ-E es `daq-e-<id>.local` (p. ej., `daq-e-def330.local`); su dirección IP sin prefijo también funciona.
{% endhint %}

## `pool-discover` — escanear sin conectarse

```bash
chloros-cli daq pool-discover                  # USB, BLE and Ethernet
chloros-cli daq pool-discover --only ble       # BLE only: this is how you find a DAQ-M's MAC
chloros-cli daq pool-discover --json           # machine-readable
```

Busca sensores a los que te podrías conectar y muestra la dirección a la que conectarte, sin establecer la conexión. `--only` toma un subconjunto separado por comas de `usb,ble,eth`; `--timeout SEC` establece el tiempo de escaneo por transporte (por defecto 5 s, limitado a un rango de 1 a 20 s). Los sensores que ya están abiertos en el grupo no aparecen en la lista (un DAQ-M conectado deja de anunciarse); para ellos, utiliza `pool-list`. En una versión comercial, `chloros-cli daq discover` y `daq list` ejecutan este mismo escaneo.

## `pool-list` — ver qué está conectado

Muestra todos los sensores del grupo de fondo, incluido el `sensor_id` que necesitan todos los demás comandos:

| Modelo | Formato `sensor_id` | Ejemplo |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5 octetos con guiones | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

Un DAQ-E cuyo PTP se haya desactivado (la configuración se guarda en la unidad) se marca en `pool-list` con `!! PTP is switched OFF on this unit (saved in its flash): its timestamps are not PTP-synced.` y el comando que lo vuelve a activar; véase [`pool-enable-ptp`](#activar-ptp-en-un-daq-y-reactivar-ptp).

## `pool-latest` — leer tramas de espectro

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

Devuelve la trama más reciente o las tramas más recientes `--recent N`; `--json` genera una salida legible por máquina para la creación de scripts. Los fotogramas son irradiancias espectrales (W/m²/nm) calibradas radiométricamente en una cuadrícula de 135 puntos, de 340 a 1010 nm, con el perfil de la tapa del sensor ya aplicado. Para obtener valores cuantitativos de irradiancia, se debe calcular la media de al menos 15 segundos de fotogramas; se trata de una característica del instrumento, no de un defecto.

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
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | Directorio de salida. Con el backend en este ordenador, la ruta relativa se resuelve respecto a la carpeta en la que se ha ejecutado el comando; con un backend remoto, la ruta se resuelve **en el equipo que ejecuta el backend**. |
| `--device-name NAME` | — | Etiqueta almacenada con la grabación. |
| `--stop` | — | Detener una grabación en curso. |
| `--confirm-after SEC` | `20` | Tras el inicio, espera hasta este tiempo a que se escriba la primera fila y avisa con un mensaje de aviso sonoro (`!! WARNING: no rows written after …s. This recording is producing an EMPTY file.`) si no llega ninguna. `0` desactiva la comprobación. |
| `--heartbeat SEC` | `30` | Con `--duration`, muestra el recuento de filas cada SEC segundos y avisa si deja de aumentar. `0` la desactiva. |

{% hint style="info" %}
El registro se realiza en el backend, por lo que el archivo `.daq` se guarda en el sistema de archivos de la **máquina del backend** —por defecto, en `~/Documents/DAQ Live View/` allí, no necesariamente donde se ejecutó la CLI—. Los nombres de los archivos incluyen el ID del sensor y una marca de tiempo.
{% endhint %}

## `pool-set-cap` — declarar la tapa instalada

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

El ID de la tapa selecciona el perfil de corrección medido en fábrica que se aplica a cada espectro, y **debe coincidir con la tapa instalada físicamente en el sensor**; ni el sensor ni el software pueden detectar la tapa por sí mismos, y la selección queda registrada en cada archivo `.daq`. El valor predeterminado en todos los casos es `sunshine_cosine` (todos los DAQ se envían con la tapa correctora de coseno Sunshine instalada, con una atenuación de aproximadamente 12× por diseño; un cambio de tapa no declarado corrige erróneamente los espectros en aproximadamente ese factor).

| `--cap-id` | Disponible en |
| --- | --- |
| `sunshine_cosine` (predeterminado) | DAQ-U, DAQ-M, DAQ-E |
| `fov_15`, `fov_45`, `fov_90` | DAQ-U, DAQ-E |
| `fov_30`, `fov_60` | Solo DAQ-U |
| `none` | Solo DAQ-E — véase la nota |

Si el ID de la tapa no pertenece al conjunto del sensor, se rechaza la conexión con un error claro. `none` (DAQ-E) significa que la tapa se ha retirado físicamente; sigue aplicando un perfil geométrico de fábrica para el difusor de cristal empotrado del DAQ-E, por lo que no es una operación nula, y un DAQ-E sin tapa es una configuración de banco de pruebas, no una configuración de campo compatible. (Un DAQ-U sin tapa es realmente «desnudo» y no necesita ningún perfil de corrección; el DAQ-M se utiliza con su tapa Sunshine.)

## `pool-disconnect` — liberar sensores

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

`pool-disconnect` libera un sensor independientemente de quién lo haya abierto, por lo que también libera un sensor que un script que se ha bloqueado haya dejado en el grupo (siempre que la grabación que se esté realizando en él haya finalizado primero). Un sensor que no se encuentra en el grupo no tiene nada que liberar: el comando muestra `<sensor-id>: already released (not in the pool)` y sale de `0`. Una liberación que el backend ha rechazado o no ha confirmado sale de `1`.

## `pool-enable-ptp` — volver a activar el PTP de un DAQ-E

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

Un DAQ-E guarda su configuración de PTP (activado/desactivado) en su propia memoria, por lo que una unidad cuyo PTP se haya desactivado permanece desactivada —y sus marcas de tiempo no están sincronizadas con PTP— hasta que se vuelva a activar. `pool-list` marca dicha unidad e imprime este comando para ella. Chloros nunca vuelve a activar una unidad por sí mismo, ya que el estado desactivado puede ser intencionado. Novedad en la versión 1.3.0.

## Configuración de dispositivos DAQ-E

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto
chloros-cli daq pool-set-auto-stream --sensor-id daq-e-def330 --on
chloros-cli daq pool-reboot --sensor-id daq-e-def330
chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

`pool-info` muestra el firmware del sensor (y si esta versión de Chloros incluye uno más reciente), su dirección y cómo la obtuvo, la configuración de red guardada cuando difiere de la actual, la transmisión al encenderse y el PTP. `pool-set-network` reinicia el sensor para aplicar el cambio y lo vuelve a conectar con la configuración que tenía; `--no-reboot` solo la guarda. Debe existir una dirección estática en la red de este ordenador; de lo contrario, Chloros la rechaza; `--force` anula esa restricción, pero solo para un ordenador al que se le vaya a reasignar una dirección o que se redirija a él. No se permiten reinicios mientras el sensor está registrando datos. `pool-update-firmware` es el equivalente en la interfaz gráfica de usuario (GUI) de **Actualizar a \<version\>** para la línea de comandos, y la forma de actualizar un DAQ-E en Linux, que no dispone de interfaz gráfica. Novedad en la versión 1.3.0.***

## Resumen de comandos

| Comando | Finalidad |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | Abrir un sensor en el grupo de fondo. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Busca sensores sin conectarse (localiza la dirección MAC BLE de un DAQ-M). |
| `daq pool-list` | Mostrar todos los sensores del grupo con su `sensor_id`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Los N fotogramas de espectro calibrados más recientes. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Reanudar/pausar la transmisión. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--confirm-after SEC] [--heartbeat SEC] [--stop]` | Iniciar/detener una grabación `.daq` (del lado del backend). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Cambiar el perfil de corrección de recorte en tiempo de ejecución. |
| `daq pool-disconnect --sensor-id ID [--all]` | Liberar un sensor o todos ellos. |
| `daq pool-enable-ptp --sensor-id ID` | Volver a activar el PTP en un DAQ-E cuyo PTP esté desactivado. |
| `daq pool-info --sensor-id ID [--json]` | El firmware, la dirección, la configuración de red guardada, la transmisión al encender y el PTP de un DAQ-E. |
| `daq pool-set-network --sensor-id ID (--auto \| --static IP [--mask M] [--gateway G]) [--no-reboot] [--force]` | DHCP o una dirección estática; reinicia y vuelve a conectar el sensor. |
| `daq pool-set-auto-stream --sensor-id ID (--on \| --off)` | Transmisión al encender el dispositivo. |
| `daq pool-reboot --sensor-id ID` | Reinicia un DAQ-E y vuelve a conectarlo. |
| `daq pool-update-firmware --sensor-id ID` | Actualiza el firmware del DAQ-E de esta versión en un sensor más antiguo. |

***

## Solución de problemas en la primera conexión del DAQ-E

1. El DAQ-E no tiene LED de estado: comprueba que recibe alimentación a través del indicador PoE/enlace del conmutador o del puerto del inyector, y espera unos segundos tras el encendido para que arranque y se conecte a la red.
2. El equipo backend debe estar en el **mismo dominio de difusión** que el sensor; el mDNS no atraviesa routers.
3. El cortafuegos debe permitir el paso de mDNS (UDP 5353), los datos del DAQ-E (UDP 5002) y PTP (UDP 319/320). El instalador de Windows añade una regla para ello, y el `.deb` de Linux las abre en un `ufw` / `firewalld` que estaba activo en el momento de la instalación; un cortafuegos de terceros, o uno habilitado posteriormente, debe permitir el acceso a estos puertos.
4. ¿Sigue sin haber respuesta de `--eth`? Utiliza `--eth-host` con el nombre de host del dispositivo (`daq-e-<id>.local`) o su IP, lo que omite el proceso de detección.

***
{% hint style="info" %}
**Consejo para los asistentes de IA.** Todas las páginas de este manual se sirven en formato Markdown sin formato: añade `.md` al slug de la URL en minúsculas de una página (esta página: `https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`); el índice legible por máquina es `https://mapir.gitbook.io/chloros/llms.txt`. Para obtener la documentación completa a nivel de opciones de `chloros-cli daq` y de todas las demás familias de comandos, consulta la [Referencia de la CLI](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`); la ruta de Python es `chloros_sdk.connect_daq_sensor()` en la [Referencia del SDK](../reference/sdk-reference.md).
{% endhint %}
</version\>

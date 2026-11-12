# Conexión en red y sincronización horaria de DAQ-E

> La configuración física de la red para el sensor —cableado, PoE, asignación de direcciones IP y los propios ajustes de red del dispositivo— se describe en el **[manual de usuario de DAQ](https://mapir.gitbook.io/daq/daq-e/network-setup)**. Esta página trata los aspectos relacionados con Chloros: la conexión, la sincronización horaria y qué hacer cuando la detección no da resultados.

El DAQ-E es el miembro de la familia DAQ que utiliza Ethernet: se alimenta a través de PoE, se detecta mediante mDNS (servicio `_daq-e._tcp`) y se puede acceder a él mediante un nombre de host derivado de su ID de sensor — `daq-e-<6 hex>.local`, por ejemplo, `daq-e-def330.local`. Esta página explica cómo transfiere datos por la red y cómo participa en la sincronización horaria PTP.

## Modos de transporte

| Modo | Punto final | Consumidores | Notas |
| --- | --- | --- | --- |
| **Multidifusión** (por defecto) | UDP `239.10.10.10:5002` | Cualquier dispositivo de la misma LAN recibe el mismo flujo | Cada datagrama se valida mediante CRC-16/CCITT |
| **Sin procesar** | Puerto TCP `5000` | Exactamente un cliente (exclusivo) | Compatible al pie de la letra con DAQ-U |

Chloros utiliza la multidifusión de forma predeterminada, lo que permite que la interfaz gráfica de usuario (GUI), la interfaz de línea de comandos (CLI) y el kit de desarrollo de aplicaciones (SDK) puedan monitorizar un sensor a la vez.

## Requisitos de red

* **Mismo dominio de difusión.** El equipo en el que se ejecuta Chloros debe estar en el mismo segmento de red de capa 2 que el sensor; el descubrimiento mDNS no atraviesa routers.
* **Cortafuegos.** En Windows, el instalador añade la regla de cortafuegos **Chloros Backend (UDP-In)**para el backend en todos los perfiles de red, lo que cubre los datos de DAQ-E (UDP 5002), mDNS (UDP 5353) y PTP (UDP 319/320), así como una regla**«Chloros Backend (TCP-In)»**, limitada a la subred local, para la conexión que establece un DAQ-E con el ordenador durante una actualización de firmware. En Linux, el `.deb` abre los puertos UDP en el `ufw` o el `firewalld` cuando ese cortafuegos está activo en el momento de la instalación. Un cortafuegos de terceros, o uno que se active posteriormente, necesita que se permitan los mismos puertos.
* **Alimentación PoE, sin LED de estado.** El DAQ-E no tiene LED propio: comprueba la alimentación a través del indicador de enlace/PoE del conmutador o del puerto del inyector, y espera unos segundos tras el encendido para que arranque y se conecte a la red.

## Conexión

**GUI:** Pestaña «Sensores de luz» → «Conectar sensor» → Tipo de dispositivo «DAQ-E (Ethernet)». La búsqueda solo se ejecuta mientras el cuadro de diálogo de conexión está en pantalla (búsqueda mDNS más un barrido ARP en Windows), repitiéndose cada 15 segundos; el botón «Actualizar» vuelve a realizar la búsqueda inmediatamente. Los sensores detectados aparecen en el menú desplegable; el primer sensor detectado se selecciona automáticamente.

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI** (backend en ejecución):

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

Un DAQ-E que ya se encuentra en el grupo se reconoce tanto por su nombre de host como por su IP: al conectarlo de nuevo con la otra dirección, se recupera la entrada existente del grupo en lugar de abrir la unidad por segunda vez.

### Hosts con varias tarjetas de red y la primera conexión tras el arranque

En hosts con más de una interfaz de red activa, el **primer** `pool-connect --eth` tras el arranque puede quedar vacío incluso cuando el sensor está en buen estado: la búsqueda de detección puede pasar por alto la interfaz en la que se encuentra el sensor mientras la caché ARP aún está fría. La solución fiable es omitir el proceso de detección y pasar la dirección de forma explícita:

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host` acepta el nombre de host mDNS o la dirección IP, siempre se dirige al sensor correcto y es la forma recomendada para scripts e instalaciones sin interfaz gráfica. En la interfaz gráfica de usuario, utiliza el botón «Actualizar» del cuadro de diálogo de conexión y espera a que se complete un ciclo de búsqueda.

## Configuración del dispositivo y firmware

El propio sensor almacena su configuración de red: DHCP (la configuración de fábrica, que recurre a una dirección local de enlace 169.254.x.x cuando ningún servidor DHCP responde) o una dirección estática. También almacena si comienza a transmitir al encenderse. Modifícalas en la interfaz gráfica de usuario (en el menú de configuración del sensor, **Red**:**Cambiar…**,**Transmisión al encender**,**Reiniciar sensor**; véase [La pestaña DAQ](gui.md)), o desde la CLI y el SDK con `pool-set-network`, `pool-set-auto-stream` y `pool-reboot` (`DAQSensorSession.set_network()`, `set_auto_stream()`, `reboot()`; véase la [Referencia de la CLI](../reference/cli-reference.md#daq-e-device-settings)). El sensor se reinicia para aplicar un cambio en la red, y Chloros lo vuelve a conectar. Debe haber una dirección estática en la red del ordenador en el que se ejecuta Chloros: Chloros rechaza aquellas a las que no puede acceder tras el reinicio. El ID del sensor y su nombre de host `daq-e-<id>.local` nunca cambian. Novedad en la versión 1.3.0.

**Las actualizaciones de firmware están integradas en la interfaz gráfica de usuario (GUI).** Cuando un DAQ-E conectado ejecuta una versión de firmware anterior a la imagen incluida en tu compilación de Chloros, la fila de su sensor muestra un icono ámbar con el texto **Actualización disponible**, y la ventana modal de configuración (icono de engranaje) ofrece un botón «Actualizar a \<version\>». La actualización se realiza a través de la red en unos 30 segundos; el sensor se reinicia y se vuelve a conectar automáticamente, y si la transferencia se interrumpe, el firmware actual permanece intacto. Desde la CLI o el SDK, y en Linux —que no dispone de interfaz gráfica—, ejecuta `chloros-cli daq pool-update-firmware --sensor-id <id>` (`DAQSensorSession.update_firmware()`).

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## Sincronización de tiempo PTP

El firmware v1.2.0+ del DAQ-E participa en el estándar IEEE 1588 PTPv2 como un reloj ordinario (solo esclavo). **El backend del host Chloros es el «grandmaster» PTP**: todos los DAQ-E y todas las cámaras LATTICE de la LAN se sincronizan con él en el dominio 0, manteniendo todas las marcas de tiempo de los dispositivos dentro de una tolerancia de aproximadamente 1 ms. Ese reloj compartido es lo que permite que las lecturas del DAQ coincidan en la marca de tiempo con las exposiciones de la cámara (véase [Grabación y el formato .daq](recording.md)).

Comprueba la sincronización desde la CLI:

| Comando | Muestra |
| --- | --- |
| `chloros-cli time-sync status` | Estado del «grandmaster» del host, prioridades BMCA, identidad del reloj y `health`: `OK`, o `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT` con un `health_detail` que indica por qué sus paquetes no llegan a la red |
| `chloros-cli time-sync peers` | Todos los esclavos detectados (sensores DAQ-E + cámaras LATTICE) |
| `chloros-cli time-sync cameras` | Estado de PTP por cámara (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`) |
| `chloros-cli time-sync restart` | Reiniciar el proceso del «grandmaster» |

En la interfaz gráfica de usuario, la ventana modal de configuración del DAQ-E muestra una fila de **Sincronización PTP** en tiempo real con el estado PTP actual del sensor.**Un DAQ-E con el PTP desactivado permanece desactivado.** La unidad guarda su configuración de PTP (activado/desactivado) en su propia memoria, por lo que una unidad cuyo PTP se haya desactivado permanece desactivada tras una reconexión o un reinicio, y sus marcas de tiempo no están sincronizadas con el PTP. `chloros-cli daq pool-list` marca dicha unidad (`!! PTP is switched OFF on this unit …`) e imprime el comando que lo soluciona:

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

Chloros nunca vuelve a activar una unidad por sí mismo, ya que el estado desactivado puede ser intencionado.

Detalles para usuarios que requieren una alineación estricta:

* Cada datagrama transmitido lleva un campo de indicadores; **el bit 2 se activa en las tramas cuya marca de tiempo está sincronizada con PTP**. Los procesos que requieran una alineación estricta entre la cámara y el DAQ deben basarse en ese bit.
* Antes de una captura sincronizada, comprueba que el sensor aparezca en `chloros-cli time-sync peers` y que `chloros-cli daq pool-list` no indique que su PTP está desactivado. (Las herramientas internas de MAPIR para el hardware directo también pueden activar la grabación al alcanzar el bloqueo PTP mediante un indicador de `--wait-ptp` que espera hasta 15 s a que el sensor alcance el estado SLAVE; dichas herramientas no forman parte de la CLI suministrada.)
* Mientras el PTP está en modo esclavo activo, el sensor rechaza los envíos manuales de reloj («PTP está proporcionando el reloj»). Esto es así por diseño: confía en el PTP.

## Notas sobre Linux

* **Puertos PTP sin privilegios de root.** El PTP utiliza los puertos UDP 319/320. El `.deb` escribe en el `/etc/sysctl.d/60-chloros-ptp.conf` (`net.ipv4.ip_unprivileged_port_start = 319`), por lo que el backend se vincula a ellos ejecutándose como tu usuario. A partir de la versión 1.3.0, el paquete no establece ninguna capacidad de archivo (`setcap`) en el backend y no necesita `libcap2-bin`; no añadas ninguna capacidad por tu cuenta, ya que, de lo contrario, el backend descomprimido no se iniciará. Si PTP no puede vincular el puerto 319, consulta [Instalación en Linux](../linux/linux-installation.md#ptp-will-not-start--cannot-bind-port-319).

* **Jetson sin pantalla / host Linux:** el paquete instala los servicios del backend, pero no habilita ninguno. Para que PTP esté siempre activo (y el DAQ disponible) sin que nadie ejecute ningún comando, habilita el servicio como tu usuario:

  ```bash
  sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
  ```

  Sin ello, PTP solo se ejecuta mientras algo haya iniciado el backend: un comando `chloros-cli` o un script del SDK. Consulta [Instalación en Linux](../linux/linux-installation.md#always-on-ptp-for-headless-hosts) para obtener información sobre el usuario root `chloros-backend.service` y cómo cambiar de él.

## Solución de problemas: «No se han encontrado dispositivos DAQ-E»

| Comprobación | Detalle |
| --- | --- |
| Alimentación | El LED del sensor no se enciende: comprueba los indicadores de PoE y de enlace del conmutador o del puerto del inyector; espera unos segundos tras el encendido |
| Dominio de difusión | El host y el sensor se encuentran en el mismo segmento L2; mDNS no se enruta |
| Cortafuegos | Windows: la regla **Chloros Backend (UDP-In)** del instalador permite el paso del flujo de DAQ-E (UDP 5002), mDNS (5353) y PTP (319/320); un cortafuegos de terceros debe tener la misma configuración. Linux: el `.deb` abre estos puertos en `ufw` / `firewalld` solo si el cortafuegos estaba activo en el momento de la instalación; permítelos tú mismo si lo activas más tarde |
| Host con varias tarjetas de red | El descubrimiento explora mDNS con un recurso de ARP; `--eth-host <ip-or-hostname>` omite el descubrimiento y se conecta directamente |
| Nueva búsqueda desde la interfaz gráfica | El descubrimiento solo se ejecuta mientras el cuadro de diálogo de conexión está abierto; utiliza «Actualizar» |

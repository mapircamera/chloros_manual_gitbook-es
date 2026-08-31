# Conexión en red y sincronización horaria de DAQ-E

> La configuración física de la red para el sensor —cableado, PoE, asignación de direcciones IP y los propios ajustes de red del dispositivo— se describe en el **[manual de usuario de DAQ](https://mapir.gitbook.io/daq/daq-e/network-setup)**. Esta página aborda los aspectos relacionados con el Chloros: conexión, sincronización horaria y qué hacer cuando la detección no da resultados.

El DAQ-E es el miembro de la familia DAQ que utiliza Ethernet: se alimenta mediante PoE, se detecta a través de mDNS (servicio `_daq-e._tcp`) y se puede direccionar mediante un nombre de host derivado de su ID de sensor — `daq-e-<6 hex>.local`, por ejemplo, `daq-e-def330.local`. Esta página explica cómo transfiere datos por la red y cómo participa en la sincronización horaria PTP.

## Modos de transporte

| Modo | Punto final | Consumidores | Notas |
| --- | --- | --- | --- |
| **Multidifusión** (por defecto) | UDP `239.10.10.10:5002` | Cualquier dispositivo de la misma LAN recibe el mismo flujo | Cada datagrama se valida mediante CRC-16/CCITT |
| **Sin procesar** | Puerto TCP `5000` | Exactamente un cliente (exclusivo) | Compatible a nivel de byte con DAQ-U |

Chloros utiliza multidifusión por defecto, lo que permite que la interfaz gráfica de usuario, CLI y SDK puedan supervisar un sensor a la vez.

## Requisitos de red

* **Mismo dominio de difusión.** La máquina en la que se ejecuta Chloros debe estar en el mismo segmento de red de capa 2 que el sensor; el descubrimiento mDNS no atraviesa routers.
* **Aviso del cortafuegos de Windows: acéptalo.** La primera vez que Chloros vincula los sockets de multidifusión, Windows Defender lo solicita una vez. Al permitirlo, se cubren los datos de DAQ-E (UDP 5002), mDNS (UDP 5353) y PTP (UDP 319/320). En Linux esto ocurre de forma silenciosa.
* **Alimentación PoE, sin LED de estado.** El DAQ-E no tiene LED propio; comprueba la alimentación mediante el indicador de enlace/PoE del conmutador o del puerto del inyector, y espera unos segundos tras el encendido para que arranque y se conecte a la red.

## Conexión

**Interfaz gráfica de usuario (GUI):** Pestaña «Sensores de luz» → «Conectar sensor» → Tipo de dispositivo «DAQ-E (Ethernet)». La búsqueda solo se ejecuta mientras el cuadro de diálogo de conexión está en pantalla (búsqueda mDNS más un barrido ARP en Windows), repitiéndose cada 15 segundos; el botón «Actualizar» vuelve a realizar la búsqueda inmediatamente. Los sensores detectados aparecen en el menú desplegable; se selecciona automáticamente el primer sensor detectado.

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI** (servidor en ejecución):

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

### Hosts con varias tarjetas de red y la primera conexión tras el arranque

En hosts con más de una interfaz de red activa, el **primer** `pool-connect --eth` tras el arranque puede quedar vacío incluso cuando el sensor está en buen estado: la búsqueda de detección puede pasar por alto la interfaz en la que se encuentra el sensor mientras la caché ARP aún está inactiva. La solución más fiable es omitir el proceso de detección y pasar la dirección de forma explícita:

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host` acepta el nombre de host mDNS o la IP, siempre se dirige al sensor correcto y es la forma recomendada para scripts e instalaciones sin interfaz gráfica. En la interfaz gráfica de usuario, utiliza el botón «Actualizar» del cuadro de diálogo de conexión y deja que se realice un ciclo de búsqueda de nuevo.

## Configuración del dispositivo y firmware

El propio sensor contiene la configuración de red —IP estática frente a DHCP + direccionamiento local de enlace—, el nombre del dispositivo, la transmisión automática al arrancar y la contraseña OTA. Esta configuración del dispositivo no está disponible como comandos en el CLI suministrado; se gestiona a través de la interfaz gráfica de usuario (GUI) del Chloros, donde aparece, o mediante el servicio de asistencia técnica del MAPIR.

**Las actualizaciones de firmware están integradas en la interfaz gráfica de usuario.**Cuando un DAQ-E conectado ejecuta un firmware más antiguo que la imagen incluida en tu compilación Chloros, en la fila del sensor aparece un icono ámbar con el texto**«Actualización disponible»**, y en la ventana modal de configuración con el icono de engranaje se muestra el<version>

botón</version> «Actualizar a<version>

». La actualización se realiza a través de la red en unos 30 segundos; el sensor se reinicia y se vuelve a conectar automáticamente, y si la transferencia se interrumpe, el firmware actual permanece intacto.

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## Sincronización de tiempo PTP

El firmware v1.2.0+ del DAQ-E participa en el estándar IEEE 1588 PTPv2 como un reloj ordinario (solo esclavo). **El backend del host Chloros es el «grandmaster» PTP**: todos los DAQ-E y todas las cámaras LATTICE de la LAN se sincronizan con él en el dominio 0, manteniendo todas las marcas de tiempo de los dispositivos dentro de una tolerancia de aproximadamente 1 ms. Ese reloj compartido es lo que permite que las lecturas del DAQ coincidan en la marca de tiempo con las exposiciones de la cámara (véase [Grabación y el formato .daq](recording.md)).

Comprueba la sincronización desde el CLI:

| Comando | Muestra |
| --- | --- |
| `chloros-cli time-sync status` | Estado del «grandmaster» del host, prioridades BMCA, identidad del reloj |
| `chloros-cli time-sync peers` | Todos los esclavos detectados (sensores DAQ-E + cámaras LATTICE) |
| `chloros-cli time-sync cameras` | Estado de PTP por cámara (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`) |
| `chloros-cli time-sync restart` | Reiniciar el proceso «grandmaster» |

En la interfaz gráfica de usuario, la ventana modal de configuración de DAQ-E muestra una fila en tiempo real denominada **Sincronización PTP** con el estado PTP actual del sensor.

Detalles para los consumidores que requieren una alineación estricta:

* Cada datagrama transmitido lleva un campo de indicadores; **el bit 2 se activa en las tramas cuya marca de tiempo está sincronizada con PTP**. Los procesos que requieran una alineación estricta entre la cámara y el DAQ deben basarse en ese bit.
* Antes de una captura sincronizada, comprueba que el sensor aparece en `chloros-cli time-sync peers`. (Las herramientas internas de hardware directo de MAPIR también pueden condicionar la grabación al bloqueo PTP mediante un indicador de `--wait-ptp` que espera hasta 15 s a que el sensor alcance el estado SLAVE; esa herramienta no forma parte del CLI suministrado.)
* Mientras el PTP está en modo esclavo activo, el sensor rechaza los envíos manuales de reloj («PTP está proporcionando el reloj»). Esto es así por diseño: confíe en el PTP.

## Notas sobre Linux

* **PTP necesita `libcap2-bin` en el momento de la instalación.** El script postinst de `.deb` concede permisos a `cap_net_bind_service=+ep` en `/usr/lib/chloros/chloros-backend` para que pueda vincular los puertos PTP 319/320 sin necesidad de ser root. Si falta `libcap2-bin`, se omite ese paso y PTP no se iniciará. Solución:

  ```bash
  sudo apt install libcap2-bin
  sudo apt reinstall chloros
  ```

* **Jetson / Raspberry Pi sin interfaz gráfica:** en la primera instalación, se genera la unidad systemd `chloros-backend.service`, pero no está habilitada. Para que PTP esté siempre activo (y la DAQ esté disponible) sin la interfaz gráfica:

  ```bash
  sudo systemctl enable --now chloros-backend.service
  ```

  Sin ella, el PTP solo funciona mientras la interfaz gráfica Chloros está abierta.

## Solución de problemas: «No se han encontrado dispositivos DAQ-E»

| Comprobación | Detalle |
| --- | --- |
| Alimentación | El LED del sensor no se enciende: comprueba los indicadores de PoE y de enlace del conmutador o del puerto del inyector; espera unos segundos tras el encendido |
| Dominio de difusión | El host y el sensor se encuentran en el mismo segmento de capa 2; mDNS no enruta |
| Cortafuegos Windows | Acepta el mensaje de Defender en la primera ejecución (UDP 5002, 5353, 319/320) |
| Host con varias tarjetas de red | Es posible que el sensor no se detecte en la primera búsqueda tras el arranque: conéctate mediante `--eth-host <ip-or-hostname>` |
| Nueva búsqueda desde la interfaz gráfica | La búsqueda solo se ejecuta mientras el cuadro de diálogo de conexión está abierto; utiliza «Actualizar» |</version>

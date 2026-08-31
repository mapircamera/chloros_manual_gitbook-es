# Conexión de cámaras

<figure><img src="../.gitbook/assets/image (37).png" alt=""><figcaption><p>La pestaña «Cámaras» antes de conectar nada</p></figcaption></figure>Chloros detecta automáticamente las cámaras LATTICE en el enlace, ya sea desde la pestaña «Cámaras» de la interfaz gráfica de usuario, desde `chloros-cli lattice`, o desde Python SDK. La cadena del modelo de la cámara determina todo lo que viene a continuación: Chloros determina el perfil del sensor, la configuración de bandas y la calibración de fábrica a partir de los valores `DeviceUserID` + `DeviceSerialNumber` de la cámara, por lo que **no hay que configurar nada por cámara**.

Antes de conectarte, asegúrate de que la red del host esté configurada: direccionamiento local de enlace, tramas jumbo y, en el caso de las matrices, los ajustes del búfer de recepción de la tarjeta de red. Esa es la configuración del hardware y se encuentra en el manual de LATTICE: [**Configuración de red**](https://mapir.gitbook.io/lattice-camera/setup/network-setup).

## Conexión desde la interfaz gráfica de usuario

Abre la pestaña **Cámaras**en la barra lateral de Chloros (las pestañas de hardware aparecen una vez que el backend ha terminado de iniciarse), o utiliza el menú principal →**Conectar a la cámara**. Ambas opciones abren el cuadro de diálogo**Conectar cámara(s)**.

### El cuadro de diálogo «Conectar cámaras»

El cuadro de diálogo escanea la red en el momento en que se abre («Escaneando la red...») y muestra una lista de todas las cámaras que encuentra. Cada fila muestra el **modelo**de la cámara (p. ej., `LATT-M3M-L41-F550`), el**número de serie**y la**dirección IP**.

* **Haz clic en una fila para seleccionarla**(se resaltará en verde). Puedes seleccionar**varias cámaras** y conectarlas de una sola vez; Chloros las conecta de forma secuencial.
* Las filas con la insignia **«Conectado»** ya están conectadas y no se pueden volver a seleccionar.
* Las filas con la insignia **«En matriz»** pertenecen a una matriz de cámaras conectada actualmente. Desconecta primero la matriz para utilizar esa cámara de forma independiente.
* **Conectar**: conecta las cámaras seleccionadas; el botón muestra un recuento, p. ej., «Conectar (3)», cuando se selecciona más de una.
* **Volver a escanear**: vuelve a ejecutar la búsqueda.
* **Cerrar**: cierra el cuadro de diálogo.
* Si la búsqueda finaliza sin resultados, el cuadro de diálogo muestra **«No se han encontrado cámaras en la red»**; consulta [Solución de problemas](connecting.md#troubleshooting) más abajo.

<figure><img src="../.gitbook/assets/image (38).png" alt=""><figcaption><p>El cuadro de diálogo «Conectar cámara(s)» — que se muestra aquí sin cámaras en la red</p></figcaption></figure>### Primera conexión: descarga del paquete de calibración

La **primera vez**que se conecta una cámara determinada a un equipo, Chloros obtiene el paquete de calibración de fábrica de la cámara (\~3,8 MB) directamente de la propia cámara a través de GigE. Mientras se lleva a cabo este proceso, el cuadro de diálogo muestra un panel verde con el mensaje**«Descargando datos de calibración de la cámara»**y una barra de progreso por número de serie; el proceso tarda aproximadamente**70 segundos** por cámara. El paquete se almacena en la caché del host, por lo que las conexiones posteriores de la misma cámara omiten por completo la descarga (y nunca muestran el panel).

### Analizar sistema

El botón **Analizar sistema** del cuadro de diálogo examina el host y la red (la etiqueta muestra «Analizando...» mientras se ejecuta) y genera un informe de diagnóstico:

* **Host**: núcleos de CPU y RAM; nombre y memoria de la GPU, o «GPU: No se ha detectado ninguna».
* **Interfaces de red**: nombre de cada tarjeta de red, velocidad de enlace, MTU (con la etiqueta «jumbo» cuando está activa), estado de conexión (activa/inactiva) y si se encuentra en un bus USB.
* **Cámaras**: número de serie, modelo, IP y**en qué tarjeta de red se conecta cada cámara**.
* **Rendimiento**: fps actuales frente a los ideales por cámara para el formato de píxeles, con una línea verde que indica «Potencial: posible mejora de N×» cuando el valor ideal supera al actual.
* **Advertencias y recomendaciones numeradas** — o «El sistema funciona correctamente para el número actual de cámaras» cuando no hay nada que corregir.

Ejecútalo siempre que la detección o la transmisión se comporten de forma inesperada: identifica la mayoría de los problemas relacionados con las NIC (MTU incorrecta, cámara en la interfaz equivocada, límites del adaptador USB) sin salir del cuadro de diálogo.

### Conexión de una matriz

Para conectar dos o más cámaras como una **matriz sincronizada**, utiliza en su lugar el asistente de conexión de matrices (**Conectar matriz de cámaras**): este le guía a través de la selección de maestro/esclavo (preconfigurada mediante una sonda de cableado GPIO), la elección del modo de visualización (mosaicos separados frente a combinados) y una pantalla de configuración del conjunto con una proyección en tiempo real de los fps alcanzables y el ancho de banda del cableado antes de confirmar la configuración. El asistente y los flujos de trabajo de las matrices se tratan en la sección de matrices multicámara de este manual; el equivalente en CLI es el «Flujo de trabajo de primera conexión de cámaras LATTICE» en la [Referencia de CLI](../reference/cli-reference.md).

## Conexión desde CLI y SDK

El acceso a CLI y SDK requiere un plan de pago Chloros+ y estar conectado; esto se aplica del lado del servidor (`401 AUTH_REQUIRED` si no se ha iniciado sesión, `403 PLAN_UPGRADE_REQUIRED` en el nivel gratuito).

```bash
# List cameras on the network (vendor, model, serial, IP, MAC)
chloros-cli lattice info

# Single-camera smoke test: capture one frame (saves every applicable export type)
chloros-cli lattice capture -o output/

# Connect a synchronized array — same smart-prep flow as the GUI
chloros-cli lattice array-connect --serials 213800234,214000533
```

```python
import chloros_sdk

# Persistent live-camera session through the backend
with chloros_sdk.connect_camera("213800234") as cam:
    ...

# Array session (smart-prep: network probe, tier auto-pick, PTP, AE seeding, trigger config)
with chloros_sdk.connect_array(["213800234", "214000533"]) as array:
    ...
```

Firmas completas, opciones y flujos de trabajo de captura: [Referencia CLI](../reference/cli-reference.md) § `chloros-cli lattice`, [Referencia SDK](../reference/sdk-reference.md) § `connect_camera()` / `connect_array()`.

## Cómo se resuelve la calibración al conectarse

Todas las cámaras LATTICE llevan su paquete de calibración de fábrica **integrado en la propia cámara**, y Chloros también comprueba la nube de MAPIR cuando la cámara se conecta:

| Situación   | Lo que utiliza Chloros                                                                                                                                                                                                          |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **En línea**| La**calibración más reciente publicada para ese número de serie**: la copia de la nube prevalece sobre la copia de la cámara. Por lo tanto, una cámara que haya sido recalibrada o actualizada por MAPIR se actualiza automáticamente; no es necesaria ninguna acción por parte del usuario. |
| **Sin conexión**| El**paquete de la cámara**, tal y como está. Los flujos de trabajo totalmente sin conexión siguen funcionando; simplemente no incorporan las calibraciones más recientes hasta que la cámara se conecta una vez (o se le vuelve a flashear el firmware de fábrica).                                                  |

En el momento de la captura, los coeficientes realmente aplicados quedan **congelados en los metadatos XMP de cada imagen**. Una actualización posterior de la calibración nunca modifica de forma silenciosa las imágenes que ya hayas capturado: al volver a procesar una captura antigua se utilizan los coeficientes grabados en su XMP, no los más recientes del momento.

## Solución de problemas

* **«No se han encontrado cámaras en la red»**: comprueba la configuración de enlace local en [Configuración de red](https://mapir.gitbook.io/lattice-camera/setup/network-setup): NIC del host estática `169.254.x.x/16`, cámaras en el mismo enlace, no se espera DHCP ni puerta de enlace. A continuación, utiliza**Analizar sistema**en el cuadro de diálogo de conexión para comprobar en qué tarjeta de red es (o no es) visible cada cámara.**Vuelve a escanear** tras cualquier cambio en el cableado o en la tarjeta de red.
* **Un equipo que antes funcionaba se niega a conectarse** (puertas del panel de matriz con `FRAMES WILL DROP` / `Reduce ROI to enable`): una actualización del controlador de la tarjeta de red ha restablecido de forma silenciosa la configuración del anillo de recepción. Vuelve a aplicarlas o ejecuta `chloros-cli lattice network --fix` desde un terminal con privilegios elevados; consulta [Configuración de red](https://mapir.gitbook.io/lattice-camera/setup/network-setup).
* **Una cámara muestra «In Array»**: pertenece a una sesión de matriz conectada. Desconecta la matriz para utilizar la cámara de forma independiente.

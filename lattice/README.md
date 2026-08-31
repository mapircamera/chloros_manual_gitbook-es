# Cámaras LATTICE

LATTICE es el sistema modular de cámaras multiespectrales de MAPIR destinado a la obtención de imágenes agrícolas y científicas. Todas las cámaras LATTICE se basan en el sensor de obturador global Sony IMX265 (**3,1 MP, píxeles de 3,45 µm**) y se conectan a través de Ethernet como dispositivos**GigE Vision**.

Chloros 1.2.0 controla las cámaras LATTICE en tiempo real —detección, vista previa en directo, captura y matrices sincronizadas de varias cámaras— desde tres interfaces:

| Interfaz    | Dónde                                                          | Plataformas                                                |
| ---------- | -------------------------------------------------------------- | -------------------------------------------------------- |
| Interfaz gráfica de usuario    | Pestaña **Cámaras** en la barra lateral de Chloros                         | Windows 10/11 x64                                        |
| CLI        | Familia de comandos `chloros-cli lattice`                           | Windows 10/11 x64, Linux x86_64, Linux aarch64 (Jetson) |
| Python SDK | `chloros_sdk.connect_camera()` / `chloros_sdk.connect_array()` | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |

> **¿Buscas el hardware?**Los módulos de cámara, los objetivos, los filtros y las bandas, los marcos y los soportes de montaje, los cables, el PoE y el cableado de disparo se describen en el [**manual de usuario de LATTICE**](https://mapir.gitbook.io/lattice-camera). Este capítulo trata sobre el control de las cámaras desde Chloros.

Las capturas de LATTICE son archivos estándar `.tif`/`.tiff`, y Chloros siempre las procesa a partir de la captura sin procesar. Consulte la [Referencia de CLI](../reference/cli-reference.md) y la [Referencia de SDK](../reference/sdk-reference.md) para ver el comando completo y la superficie API.

## Dos configuraciones de sensores

| Configuración | Sensor       | Filtro                                | Lo que ofrece una cámara                                          |
| ------------- | ------------ | ------------------------------------- | ----------------------------------------------------------------- |
| **M3C**| Color Bayer | filtro de paso de banda triple                |**Tres bandas calibradas a partir de una única exposición**                 |
| **M3M**| Monocromo   | filtro de interferencia de banda estrecha única |**Una banda calibrada**; combinar varias cámaras M3M para obtener índices |

Dado que una cámara M3M es monocromática y cuenta con un único filtro, cada banda obtiene su propia exposición. Una cámara M3C cubre sus tres bandas con una sola exposición del sensor.

## Cadenas de modelo y nomenclatura

Cada cámara almacena su identidad en GenICam `DeviceUserID` como una cadena de modelo:

```
<sensor>-<lens>-F<filter>       e.g.  M3C-L41-FRGN,  M3M-L87-F450
```

Chloros la muestra con el prefijo `LATT-` (por ejemplo, `LATT-M3M-L87-F450`). La misma cadena `LATT-…` se escribe en la etiqueta EXIF `Model` de cada exportación y se utiliza como nombre de la carpeta de salida de la cámara en los proyectos procesados.

| Componente | Valores                                                   | Significado                                                                                            |
| --------- | -------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Sensor    | `M3C` / `M3M`                                            | Color Bayer / monocromo                                                                          |
| Objetivo      | `L41` / `L87`                                            | El número corresponde al **campo de visión horizontal en grados**: L41 = estrecho (41°), L87 = amplio (87°)    |
| Filtro    | `FRGB` / `FRGN` / `FOCN` / `FNGB` (M3C) o `F<nm>` (M3M) | Véase [Filtros y bandas espectrales](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands) |

La cadena del modelo controla todo lo que viene a continuación: Chloros determina el perfil del sensor, la disposición de las bandas y la calibración de fábrica a partir de `DeviceUserID` + `DeviceSerialNumber`. No hay nada que configurar por cámara; consulta [Conexión de cámaras](connecting.md).

## Filtros y bandas

Los centros de banda, los bordes de FWHM y el catálogo completo de 23 referencias de M3M son especificaciones del producto, por lo que se encuentran en el manual del hardware: [**Filtros y bandas espectrales**](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands).

Lo que importa en cuanto al software: el código del filtro en la cadena del modelo determina qué productos Chloros puede generar. Las cámaras con filtro RGB (`FRGB`) emiten únicamente productos sin bayering y de vista previa; la radiancia y la reflectancia por banda no son significativas para un sensor de banda ancha, por lo que Chloros las omite y lo indica. Cualquier otro filtro genera la cadena completa de radiancia → reflectancia → índice.

## La calibración radiométrica de un vistazo

Cada cámara LATTICE se calibra individualmente en fábrica siguiendo una cadena trazable según el NIST y se envía con un certificado específico para cada cámara. Lo que abarca, cómo se mide y la precisión que se puede indicar se detalla en el manual de hardware: [**Calibración radiométrica de fábrica**](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration).

En cuanto al software, lo importante es que Chloros determina la calibración correcta cuando se conecta una cámara y fija los coeficientes aplicados en cada exportación; véase [Conexión de cámaras](connecting.md).

## En este capítulo

* [Conexión de cámaras](connecting.md): detección automática, el cuadro de diálogo de conexión de la interfaz gráfica de usuario, los equivalentes de CLI/SDK, y cómo se resuelve la calibración de fábrica (paquete integrado en la cámara frente a la nube) cuando se conecta una cámara.

Otros temas relacionados con LATTICE —ajustes de la cámara y control en directo, modos de captura, matrices multicámara y procesamiento e índices mono (M3M)— se tratan en secciones específicas de este manual, y la lista completa de comandos se encuentra en la [Referencia CLI](../reference/cli-reference.md) y en la [Referencia de SDK](../reference/sdk-reference.md).

---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/supported-cameras
---

# Cámaras compatibles

Chloros procesa imágenes de dos familias de cámaras MAPIR en **todas las plataformas** (Windows, Linux amd64 y Linux arm64/Jetson):

* **Survey3** — Cámaras Survey3W (gran angular) y Survey3N (ángulo estrecho). Entrada: `RAW+JPG`.
* **LATTICE**— Módulos de cámara multiespectral M3C y M3M. Entrada: capturas `.tif`/`.tiff`. Las cámaras LATTICE también se pueden**controlar en directo** desde Chloros — a través de la pestaña «Cámaras» de la interfaz gráfica de usuario (Windows) o `chloros-cli lattice` / Python SDK (Windows y Linux), incluidas las matrices multicámara sincronizadas. Consulte la [guía de LATTICE](lattice/).

El proceso de tratamiento también admite archivos de entrada `.dng`.

## Survey3

<table data-header-hidden><thead><tr><th width="156">Fabricante</th><th width="250">Modelo de cámara</th><th width="138">Modelo de filtro</th><th width="187">Tipo de imagen</th></tr></thead><tbody><tr><td><strong>Fabricante</strong></td><td><strong>Modelo de cámara</strong></td><td><strong>Modelo de filtro</strong></td><td><strong>Tipo de imagen</strong></td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RGB</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RGN</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>OCN</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>NGB</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RE</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>NIR</td><td>RAW+JPG, JPG</td></tr></tbody></table>## LATTICE

La línea LATTICE es un sistema modular de cámaras multiespectrales basado en el sensor de obturador global Sony IMX265 (3,1 MP, píxeles de 3,45 µm). Cada cámara almacena su identidad como una cadena de modelo:

```
<sensor>-<lens>-F<filter>        e.g.  M3C-L41-FRGN,  M3M-L87-F550
```

Chloros la muestra con el prefijo `LATT-` (por ejemplo, `LATT-M3M-L41-F550`), y la cadena de modelo controla todo lo que viene a continuación: el perfil del sensor, la disposición de las bandas y la calibración se resuelven automáticamente; no hay que configurar nada por cámara. El número del objetivo corresponde al **campo de visión horizontal en grados**: `L41` = estrecho 41°, `L87` = amplio 87°.

Existen dos configuraciones de sensor:

| Configuración | Sensor      | Tipo de filtro                           | Bandas por cámara                                                        |
| ------------- | ----------- | ------------------------------------- | ----------------------------------------------------------------------- |
| **M3C**       | Color Bayer | Triple paso de banda                       | 3 bandas espectrales en una sola exposición                                 |
| **M3M**       | Monocromo  | Filtro de interferencia de banda estrecha única | 1 banda calibrada — combinar varias cámaras M3M para índices de vegetación |

### Opciones de filtro M3C (Bayer)

| Filtro | Bandas (nombre @ centro nm / FWHM nm)       |
| ------ | ---------------------------------------- |
| `FRGB` | Blue 475/30 · Green 550/30 · Red 625/30  |
| `FRGN` | Red 660/21 · Green 550/30 · NIR 850/30   |
| `FOCN` | Orange 615/21 · Cyan 490/38 · NIR 808/14 |
| `FNGB` | Blue 475/30 · Green 550/30 · NIR 850/30  |

### Catálogo de filtros M3M (mono) — 23 referencias

El número F es la referencia del artículo; la banda medida (estampada en cada unidad calibrada destinada a la exportación) es el escaneo del filtro por lote:

| Referencia    | Centro (nm, medido) | Límites de FWHM (nm) | Ancho (nm) |
| ------ | --------------------- | --------------- | ---------- |
| F385   | 379,4                 | 367–392         | 25         |
| F405   | 403,9                 | 390–417         | 27         |
| F450   | 443,7                 | 430–458         | 28         |
| F485   | 489,7                 | 478–502         | 24         |
| F520   | 519,9                 | 504–536         | 32         |
| F550   | 548,4                 | 531–566         | 35         |
| F590   | 589,0                 | 570–608         | 38         |
| F615   | 623,8                 | 614–634         | 20         |
| F632   | 633,4                 | 616–651         | 35         |
| F650   | 651,1                 | 636–666         | 30         |
| F685   | 686,2                 | 675–698         | 23         |
| F715   | — (nominal)           | 706–724         | 18         |
| F725   | 725.2                 | 712–738         | 26         |
| F750   | 746.0                 | 729–763         | 34         |
| F780   | 775.1                 | 754–796         | 42         |
| F808   | 810.3                 | 789–832         | 43         |
| F832   | 826,1                 | 810–843         | 33         |
| F850   | 846,5                 | 828–865         | 37         |
| F880   | — (nominal)           | 867–893         | 26         |
| F905   | — (nominal)           | 892–920         | 28         |
| F940   | 940,6                 | 923–958         | 35         |
| F950   | 945,1                 | 929–961         | 32         |
| F988 † | 985,3                 | 968–1003        | 35         |

_«Los extremos de banda se miden como valores de anchura total a la mitad del máximo a partir de los escaneos de filtro por lote de MAPIR; los mismos valores que Chloros incluye en cada exportación calibrada.»_ «— (nominal)» = aún no se ha realizado ningún escaneo del lote; para esas referencias, el centro indicado es el número de referencia y la anchura es la cifra del fabricante.

† «La reflectancia de F988 se calibra utilizando un panel de reflectancia en escena: la banda se encuentra fuera del rango calibrado del sensor de luz DAQ, por lo que Chloros aplica tu captura más reciente del panel y la mantiene entre las lecturas del panel». Véase [Objetivos de calibración](calibration-targets.md).

Para el control de la cámara en directo, las matrices, la configuración de red y la cadena de procesamiento radiométrico, consulte la [guía de LATTICE](lattice/).

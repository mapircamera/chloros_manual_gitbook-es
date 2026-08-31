---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Fórmulas de índices multiespectrales

Las siguientes fórmulas de índices utilizan una combinación de los rangos de transmisión media del filtro Survey3:

<table><thead><tr><th align="center">Color del filtro Survey3</th><th width="196.199951171875" align="center">Survey3 Nombre del filtro</th><th width="159.800048828125" align="center">Rango de transmisión (FWHM)</th><th align="center">Transmisión media</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468-483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN - Cyan</td><td align="center">476-512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543-558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598-640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653-668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712-735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798-848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835-865 nm</td><td align="center">850 nm</td></tr></tbody></table>Cuando se utilizan estas fórmulas, el nombre puede terminar en «\_1» o «\_2», lo que indica qué filtro NIR se ha utilizado: o bien el NIR1 o bien el NIR2.

Para las cámaras LATTICE M3C (pasabanda triple de Bayer), el mismo motor de indexación utiliza las bandas del filtro M3C:

| Filtro M3C | Banda 1 (centro/FWHM) | Banda 2 (centro/FWHM) | Banda 3 (centro/FWHM) |
| --- | --- | --- | --- |
| FRGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | Red 625 nm / 30 nm |
| FRGN | Red 660 nm / 21 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |
| FOCN | Orange 615 nm / 21 nm | Cyan 490 nm / 38 nm | NIR 808 nm / 14 nm |
| FNGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |

Las cámaras LATTICE M3M son monobanda (un filtro de banda estrecha por cámara), por lo que no se calculan índices multibanda para una imagen M3M aislada. Para calcular índices con M3M, combina dos o más cámaras en una pila multibanda alineada y utiliza el motor de índices de LATTICE (`chloros-cli lattice index` o la calculadora de índices en tiempo real de la interfaz gráfica de usuario).

***

## Dónde funciona cada nombre de índice

Chloros tiene **tres** superficies de índice, y sus listas predefinidas no son idénticas. Utiliza esta sección para comprobar si un nombre funcionará donde tienes previsto utilizarlo.

| Dónde te encuentras | Qué lista se aplica | Recuento |
| --- | --- | --- |
| Ajustes del proyecto → Índice → Añadir índice (interfaz gráfica) | Superficie 1 | 27 |
| Visor de imágenes [Área de pruebas de índices/LUT](../image-viewer-gui/index-lut-sandbox.md) (interfaz gráfica) | Superficie 1 | 27 |
| `chloros-cli process --indices NDVI,NDRE` | Superficie 2 | 22 |
| SDK `process_folder(indices=[...])` | Superficie 2 | 22 |
| `chloros-cli lattice index --preset` | Superficie 3 | 22 (un 22 diferente) |
| Calculadora de índices en tiempo real de la pestaña «Cámaras» | Superficie 3 | 22 (un 22 diferente) |

Las superficies 1 y 2 trabajan con **una imagen cada vez de una sola cámara**, utilizando las ranuras de símbolos `x`/`y`/`z`(/`a`) vinculados a los canales de filtro de esa cámara. Surface 3 trabaja con una**pila multibanda alineada** —varias cámaras LATTICE co-registradas en un único cubo— y hace referencia a los canales mediante nombres en minúsculas.

### 1. Ajustes del proyecto en la interfaz gráfica de usuario / Menú desplegable «Sandbox» del visor de imágenes — 27 fórmulas

El menú desplegable las muestra en este orden (se trata del orden de inserción, no alfabético):

`NDVI, GNDVI, CVI, ENDVI, EVI, MSR, OSAVI, TDVI, LAI, FCI1, FCI2, GARI, GCI, GEMI, GLI, GOSAVI, GRVI, GSAVI, LCI, MNLI, MSAVI2, NDRE, NLI, RDVI, SAVI, VARI, WDRVI`

En la interfaz gráfica de usuario, se arrastran los canales de filtro de la cámara a las ranuras de banda de la fórmula, por lo que cualquier fórmula se puede utilizar con cualquier asignación de bandas que admita la cámara. Las fórmulas personalizadas que hayas guardado se añaden al final de esta lista.

Las **cinco fórmulas exclusivas de la interfaz gráfica** —aquellas que la lista CLI/SDK `--indices` no acepta— se implementan de la siguiente manera:

| Preajuste exclusivo de la interfaz gráfica | Fórmula (tal y como está implementada) | Ranuras |
| --- | --- | --- |
| FCI1 | `x*y` | x, y |
| FCI2 | `x*y` | x, y |
| GARI | `(y-(x-1.7*(z-a)))/(y+(x-1.7*(z-a)))` | x, y, z, a (cuatro ranuras) |
| GEMI | `((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5))*(1-0.25*((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5)))-((x-0.125)/(1-x))` | x, y |
| LCI | `(y-x)/(y+z)` | x, y, z |

La correspondencia prevista para cada uno de ellos se indica en su propia sección más adelante en esta página (por ejemplo, GARI espera x=Green, y = NIR, z = Blue, a = Red). GARI es la única fórmula de Chloros que utiliza una cuarta posición.

### 2. Expansión de nombres CLI / SDK `--indices` — 22 preajustes

La opción `chloros-cli process --indices` (y el parámetro SDK `indices`) admite estos nombres de preajustes:

`NDVI, GNDVI, NDRE, OSAVI, SAVI, MSAVI2, EVI, MSR, TDVI, LAI, GCI, GRVI, GSAVI, GOSAVI, NLI, MNLI, RDVI, WDRVI, CVI, ENDVI, GLI, VARI`

{% hint style="warning" %}
**Los nombres de índices desconocidos se omiten sin aviso.** Un nombre que no figure en esta lista (incluidas las cinco fórmulas exclusivas de la interfaz gráfica de usuario `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` y cualquier fórmula personalizada que hayas guardado en la interfaz gráfica de usuario) se omite con solo un aviso en el registro; la ejecución continúa sin ese índice y, aun así, se considera un éxito. El aviso se muestra de la siguiente forma:

```
[INDEX_EXPAND] skipping unknown preset 'LCI'; known: ['CVI', 'ENDVI', 'EVI', ...]
```

La coincidencia de nombres se realizatras eliminar los espacios en blanco, por lo que `ndvi`, `NDVI` y ` NDVI ` corresponden al mismo preajuste. Un preajuste también se omite si requiere una banda que el filtro de tu cámarano ofrece.
{% endhint %}

Las fórmulas exactas tal y como se han implementado (los símbolos `x`/`y`/`z` son ranuras de banda; se muestra la asignación predeterminada por preajuste):

| Preajuste | Fórmula (tal y como está implementada) | Filtro predeterminado | Ranuras (x, y, z) |
| --- | --- | --- | --- |
| NDVI | `(y-x)/(y+x)` | RGN | Red, NIR |
| GNDVI | `(y-x)/(y+x)` | RGN | Green, NIR |
| NDRE | `(y-x)/(y+x)` | RE | RE, NIR |
| OSAVI | `(y-x)/(y+x+0.16)` | RGN | Red, NIR |
| SAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Red, NIR |
| MSAVI2 | `(2*y+1-sqrt((2*y+1)*(2*y+1)-8*(y-x)))/2` | RGN | Red, NIR |
| EVI | `2.5*(y-x)/(y+6*x-7.5*z+1)` | RGN | Red, NIR, Blue |
| MSR | `((y/x)-1)/(sqrt(y/x)+1)` | RGN | Red, NIR |
| TDVI | `1.5*(y-x)/sqrt(y*y+x+0.5)` | RGN | Red, NIR |
| LAI | `3.618*(2.5*(y-x)/(y+6*x-7.5*z+1))-0.118` | RGN | Red, NIR, Blue |
| GCI | `(y/x)-1` | RGN | Green, NIR |
| GRVI | `y/x` | RGN | Green, NIR |
| GSAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Green, NIR |
| GOSAVI | `(y-x)/(y+x+0.16)` | RGN | Green, NIR |
| NLI | `((y*y)-x)/((y*y)+x)` | RGN | Red, NIR |
| MNLI | `((y*y-x)*(1+0.5))/((y*y)+x+0.5)` | RGN | Red, NIR |
| RDVI | `(y-x)/sqrt(y+x)` | RGN | Red, NIR |
| WDRVI | `(0.2*y-x)/(0.2*y+x)` | RGN | Red, NIR |
| CVI | `(z/y)/(x/y)` | RGB | Red, Green, Blue |
| ENDVI | `((x+y)-(2*z))/((x+y)+(2*z))` | RGB | Red, Green, Blue |
| GLI | `((y-x)+(y-z))/((2*y)+x+z)` | RGB | Red, Green, Blue |
| VARI | `(y-x)/(y+x-z)` | RGB | Red, Green, Blue |

#### Cómo se convierten los nombres de los preajustes en posiciones de banda

Cuando se pasa un nombre sin más, como `NDVI`, Chloros tiene que decidir qué canal de qué archivo lee cada símbolo. Para ello, utiliza esta tabla, que asigna un código de filtro a la posición en la matriz de cada canal:

| Código de filtro | Canal → índice de la matriz |
| --- | --- |
| OCN | Orange 0, Cyan 1, NIR 2 (`Red` se acepta como alias de Orange, también 0) |
| RGN | Red 0, Green 1, NIR 2 |
| NGB | NIR 0, Green 1, Blue 2 |
| RGB | Red 0, Green 1, Blue 2 |
| RE | RE 0 |
| NIR | NIR 0 |

El **filtro predeterminado** del preajuste (la columna «Filtro predeterminado» anterior) se utiliza cuando el proyecto contiene imágenes con ese filtro. Si no es así, Chloros analiza los filtros realmente presentes en el proyecto en el orden `RGN, OCN, NGB, RGB, RE, NIR` y selecciona el primero que pueda proporcionar todos los canales que necesita el preajuste. Si ninguno puede hacerlo, el preajuste se descarta para esa ejecución. Por eso, `NDVI`, solicitado en un conjunto de datos que solo contiene OCN, sigue produciendo un resultado aceptable : se vincula a las posiciones Orange y NIR de OCN.

Las cadenas del modelo LATTICE M3C llevan el filtro con el prefijo `F` (`LATT-M3C-L41-FRGN`), pero el prefijo se omite cuando se lee el código del filtro de la imagen, por lo que una cámara FRGN lo resuelve a través de la fila `RGN` situada encima y no requiere ningún tratamiento especial.

### 3. Motor de índice LATTICE (`lattice index --preset`, calculadora de índices en tiempo real) — 22 preajustes

El motor LATTICE funciona con pilas multibanda alineadas (matrices en tiempo real o archivos TIFF multibanda exportados) y utiliza nombres de canales en minúsculas (`red`, `green`, `blue`, `red_edge`, `nir`). Su lista de preajustes difiere de las dos anteriores:

| Preajuste | Fórmula | Canales |
| --- | --- | --- |
| NDVI | `(nir - red) / (nir + red)` | rojo, nir |
| GNDVI | `(nir - green) / (nir + green)` | verde, NIR |
| BNDVI | `(nir - blue) / (nir + blue)` | azul, NIR |
| NDRE | `(nir - red_edge) / (nir + red_edge)` | rojo\_borde, nir |
| ENDVI | `((nir + green) - 2*blue) / ((nir + green) + 2*blue)` | azul, verde, infrarrojo |
| SAVI | `1.5 * (nir - red) / (nir + red + 0.5)` | rojo, infrarrojo |
| OSAVI | `1.5 * (nir - red) / (nir + red + 0.16)` | rojo, infrarrojo |
| MSAVI | `(2*nir + 1 - sqrt((2*nir + 1)**2 - 8*(nir - red))) / 2` | rojo, nir |
| EVI | `2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)` | azul, rojo, nir |
| EVI2 | `2.5 * (nir - red) / (nir + 2.4*red + 1)` | rojo, infrarrojo |
| CVI | `(nir / green) - (red / green)` | rojo, verde, NIR |
| MSR | `((nir/red) - 1) / (sqrt(nir/red) + 1)` | rojo, NIR |
| TDVI | `sqrt((nir - red) / (nir + red) + 0.5)` | rojo, NIR |
| LAI | `3.618 * ((nir - red) / (nir + 6*red - 7.5*green + 1)) - 0.118` | rojo, verde, NIR |
| GLI | `(2*green - red - blue) / (2*green + red + blue)` | rojo, verde, azul |
| NGRDI | `(green - red) / (green + red)` | rojo, verde |
| VARI | `(green - red) / (green + red - blue)` | rojo, verde, azul |
| TGI | `green - 0.39*red - 0.61*blue` | rojo, verde, azul |
| EXG | `2*green - red - blue` | rojo, verde, azul |
| CIRE | `(nir / red_edge) - 1` | rojo\_borde, infrarrojo |
| CIGREEN | `(nir / green) - 1` | verde, nir |
| NDWI | `(green - nir) / (green + nir)` | verde, nir |

Ejecuta `chloros-cli lattice index --list-presets` para imprimir esta tabla desde tu versión instalada, y `--list-gradients` para ver los degradados de color disponibles. Los símbolos de los canales distinguen entre mayúsculas y minúsculas y deben coincidir con los nombres en minúsculas del preajuste (p. ej., `--channel red=Red_660 --channel nir=NIR_850`).

***

## CVI

Tal y como se ha implementado en la interfaz gráfica de usuario y en la lista de preajustes CLI/SDK, CVI es la fórmula de la «razón de razones»:

$$
CVI = {(z / y) \over (x / y)}
$$

con la asignación de canales predeterminada de RGB: x=Red, y=Green, z=Blue. En la interfaz gráfica de usuario puedes arrastrar cualquiera de los canales de tu cámara a las ranuras x/y/z. Ten en cuenta que el preajuste `CVI` del motor de índices LATTICE utiliza una fórmula diferente, `(NIR / Green) - (Red / Green)`; consulta las tablas anteriores para la superficie que estés utilizando.

***

## ENDVI - Índice de vegetación por diferencia normalizada mejorado

Este índice utiliza el canal azul además de NIR y el verde, y es muy utilizado con cámaras filtradas con NGB, en las que la banda azul sustituye a la roja.

$$
ENDVI = {(NIR + Green) - (2 * Blue) \over (NIR + Green) + (2 * Blue)}
$$

La implementación es la fórmula de símbolos `((x+y)-(2*z))/((x+y)+(2*z))` : asigna los canales NIR y Green de tu cámara a las ranuras x/y y Blue a z (para una cámara NGB: x=NIR, y=Green, z=Blue).

***

## EVI - Índice de vegetación mejorado

Este índice se desarrolló originalmente para su uso con datos MODIS como una mejora respecto al NDVI, optimizando la señal de vegetación en zonas con un alto índice de área foliar (LAI). Resulta especialmente útil en regiones con valores elevados de LAI, donde el NDVI puede saturarse. Utiliza la región de reflectancia azul para corregir las señales de fondo del suelo y reducir las influencias atmosféricas, incluida la dispersión de aerosoles.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Los valores de EVI deben oscilar entre 0 y 1 para los píxeles de vegetación. Los elementos brillantes, como las nubes y los edificios blancos, junto con los elementos oscuros, como el agua, pueden dar lugar a valores de píxel anómalos en una imagen EVI. Antes de crear una imagen EVI, debes enmascarar las nubes y los elementos brillantes de la imagen de reflectancia, y, opcionalmente, aplicar un umbral a los valores de los píxeles de 0 a 1.

_Referencia: Huete, A., et al. «Overview of the Radiometric and Biophysical Performance of the MODIS Vegetation Indices». Remote Sensing of Environment 83 (2002): 195-213._

***

## FCI1 - Índice de cobertura forestal 1

_Solo en la interfaz gráfica de usuario (GUI); no está disponible como preajuste CLI/SDK `--indices`._

Este índice distingue las copas de los árboles de otros tipos de vegetación utilizando imágenes de reflectancia multiespectral que incluyen una banda de «borde rojo».

$$
FCI1 = Red * RedEdge
$$

Las zonas boscosas presentarán valores más bajos de FCI1 debido a la menor reflectancia de los árboles y a la presencia de sombras dentro del dosel.

_Referencia: Becker, Sarah J., Craig S.T. Daughtry y Andrew L. Russ. «Índices robustos de cobertura forestal para imágenes multiespectrales». Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## FCI2 - Índice de cobertura forestal 2

_Solo en la interfaz gráfica de usuario (GUI); no está disponible como preajuste CLI/SDK `--indices`._

Este índice distingue las copas de los árboles de otros tipos de vegetación utilizando imágenes de reflectancia multiespectral que no incluyen una banda de «borde rojo».

$$
FCI2 = Red * NIR
$$

Las zonas boscosas presentarán valores más bajos de FCI2 debido a la menor reflectancia de los árboles y a la presencia de sombras dentro del dosel.

_Referencia: Becker, Sarah J., Craig S.T. Daughtry y Andrew L. Russ. «Índices robustos de cobertura forestal para imágenes multiespectrales». Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## GEMI - Índice de monitorización medioambiental global

_Solo GUI — no disponible como CLI/SDK ni `--indices`._

Este índice de vegetación no lineal se utiliza para la monitorización medioambiental global a partir de imágenes de satélite e intenta corregir los efectos atmosféricos. Es similar al NDVI, pero es menos sensible a los efectos atmosféricos. Se ve afectado por el suelo desnudo; por lo tanto, no se recomienda su uso en zonas con vegetación escasa o moderadamente densa.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Donde:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Referencia: Pinty, B. y M. Verstraete. GEMI: un índice no lineal para el seguimiento de la vegetación global a partir de satélites. Vegetation 101 (1992): 15-20._

***

## GARI - Green: índice resistente a la atmósfera

_Solo en la interfaz gráfica de usuario (GUI); no está disponible como preajuste CLI/SDK `--indices`._

Este índice es más sensible a un amplio rango de concentraciones de clorofila y menos sensible a los efectos atmosféricos que el NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

La constante gamma es una función de ponderación que depende de las condiciones de los aerosoles en la atmósfera. ENVI utiliza un valor de 1,7, que es el valor recomendado por Gitelson, Kaufman y Merzylak (1996, página 296).

_Referencia: Gitelson, A., Y. Kaufman y M. Merzylak. «Uso de un canal Green en la teledetección de la vegetación global a partir de EOS-MODIS.» Remote Sensing of Environment 58 (1996): 289-298._

***

## GCI - Green Índice de clorofila

Este índice se utiliza para estimar el contenido de clorofila en las hojas de una amplia gama de especies vegetales.

$$
GCI = {NIR \over Green} - 1
$$

Contar con un amplio rango de longitudes de onda NIR y verdes permite predecir mejor el contenido de clorofila, al tiempo que ofrece mayor sensibilidad y una relación señal-ruido más elevada.

_Referencia: Gitelson, A., Y. Gritz y M. Merzlyak. «Relaciones entre el contenido de clorofila en las hojas y la reflectancia espectral, y algoritmos para la evaluación no destructiva de la clorofila en las hojas de plantas superiores». Journal of Plant Physiology 160 (2003): 271-282._

***

## Índice foliar GLI - Green

Este índice se diseñó originalmente para su uso con una cámara digital RGB con el fin de medir la cobertura de trigo, en la que los valores digitales (DN) de rojo, verde y azul oscilan entre 0 y 255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

Los valores de GLI oscilan entre -1 y +1. Los valores negativos representan el suelo y los elementos inertes, mientras que los valores positivos representan las hojas y los tallos verdes.

_Referencia: Louhaichi, M., M. Borman y D. Johnson. «Plataforma de localización espacial y fotografía aérea para la documentación de los impactos del pastoreo en el trigo». Geocarto International 16, n.º 1 (2001): 65-70._

***

## GNDVI - Green Índice de vegetación por diferencia normalizada

Este índice es similar al NDVI, salvo que mide el espectro verde de 540 a 570 nm en lugar del espectro rojo. Este índice es más sensible a la concentración de clorofila que el NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Referencia: Gitelson, A. y M. Merzlyak. «Teledetección de la concentración de clorofila en las hojas de las plantas superiores». Advances in Space Research 22 (1998): 689-692._

***

## GOSAVI - Green Índice de vegetación optimizado y ajustado al suelo

Este índice se diseñó originalmente con fotografía en color e infrarrojo para predecir las necesidades de nitrógeno del maíz. Es similar al OSAVI, pero sustituye la banda verde por la roja.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Referencia: Sripada, R., et al. «Determinación de las necesidades de nitrógeno del maíz durante la temporada mediante fotografía aérea en infrarrojo y color». Tesis doctoral, Universidad Estatal de Carolina del Norte, 2005._

***

## Índice de vegetación basado en la relación GRVI - Green

Este índice es sensible a las tasas de fotosíntesis en las copas de los bosques, ya que las reflectancias del verde y del rojo se ven fuertemente influenciadas por los cambios en los pigmentos foliares.

$$
GRVI = {NIR \over Green }
$$

_Referencia: Sripada, R., et al. «Fotografía aérea en color e infrarrojo para determinar las necesidades de nitrógeno en las primeras etapas de la temporada del maíz». Agronomy Journal 98 (2006): 968-977._

***

## GSAVI - Green Índice de vegetación ajustado al suelo

Este índice se diseñó originalmente con fotografía en color e infrarrojo para predecir las necesidades de nitrógeno del maíz. Es similar al SAVI, pero sustituye la banda verde por la roja.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Referencia: Sripada, R., et al. «Determinación de las necesidades de nitrógeno del maíz durante la temporada mediante fotografía aérea en infrarrojo color». Tesis doctoral, Universidad Estatal de Carolina del Norte, 2005._

***

## LAI - Índice de área foliar

Este índice se utiliza para estimar la cobertura foliar y para pronosticar el crecimiento y el rendimiento de los cultivos. ENVI calcula el LAI verde utilizando la siguiente fórmula empírica de Boegh et al (2002):

$$
LAI = 3.618 * EVI - 0.118
$$

Donde EVI es:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Los valores altos de LAI suelen oscilar entre aproximadamente 0 y 3,5. Sin embargo, cuando la escena contiene nubes y otros elementos brillantes que producen píxeles saturados, los valores de LAI pueden superar el 3,5. Lo ideal es enmascarar las nubes y los elementos brillantes de la escena antes de crear una imagen LAI.

_Referencia: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde y A. Thomsen. «Datos multiespectrales aéreos para cuantificar el índice de área foliar, la concentración de nitrógeno y la eficiencia fotosintética en la agricultura». Remote Sensing of Environment 81, n.º 2-3 (2002): 179-193._

***

## LCI - Índice de clorofila foliar

_Solo en la interfaz gráfica de usuario (GUI); no está disponible como CLI/SDK ni en el preajuste `--indices`._

Este índice se utiliza para estimar el contenido de clorofila en plantas superiores y es sensible a la variación de la reflectancia causada por la absorción de la clorofila.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Referencia: Datt, B. «Remote Sensing of Water Content in Eucalyptus Leaves». Journal of Plant Physiology 154, n.º 1 (1999): 30-36._

***

## MNLI - Índice no lineal modificado

Este índice es una mejora del índice no lineal (NLI) que incorpora el índice de vegetación ajustado al suelo (SAVI) para tener en cuenta el fondo del suelo. ENVI utiliza un valor de 0,5 para el factor de ajuste del fondo del dosel (_L_).

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Referencia: Yang, Z., P. Willis y R. Mueller. «Impact of Band-Ratio Enhanced AWIFS Image to Crop Classification Accuracy». Actas del Simposio de Teledetección Pecora 17 (2008), Denver, CO._

***

## MSAVI2 - Índice de vegetación ajustado al suelo modificado 2

Este índice es una versión más sencilla del índice MSAVI propuesto por Qi et al. (1994), que mejora el Índice de Vegetación Ajustado al Suelo (SAVI). Reduce el ruido del suelo y aumenta el rango dinámico de la señal de la vegetación. El MSAVI2 se basa en un método inductivo que no utiliza un valor constante de _L_ (como en el caso del SAVI) para resaltar la vegetación sana.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Referencia: Qi, J., A. Chehbouni, A. Huete, Y. Kerr y S. Sorooshian. «A Modified Soil Adjusted Vegetation Index». Remote Sensing of Environment 48 (1994): 119-126._

***

## MSR - Razón simple modificada

Este índice es una modificación de la relación simple NIR/Red, diseñada para linealizar su relación con los parámetros biofísicos, y es más sensible que el NDVI a densidades de vegetación más elevadas.

$$
MSR = {(NIR / Red) - 1 \over \sqrt{NIR / Red} + 1}
$$

_Referencia: Chen, J. «Evaluación de índices de vegetación y de una relación simple modificada para aplicaciones boreales». Canadian Journal of Remote Sensing 22 (1996): 229-242._

***

## NDRE: diferencia normalizada de RedEdge

Este índice es similar al NDVI, pero compara el contraste entre el NIR y el RedEdge en lugar del Red, que a menudo detecta la vegetación .

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI - Índice de vegetación de diferencia normalizada

Este índice es una medida de la vegetación verde y sana. La combinación de su formulación de diferencia normalizada y el uso de las regiones de mayor absorción y reflectancia de la clorofila lo hacen robusto en una amplia gama de condiciones. Sin embargo, puede saturarse en condiciones de vegetación densa cuando el valor de LAI es elevado.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

El valor de este índice oscila entre -1 y 1. El rango habitual para la vegetación verde es de 0,2 a 0,8.

_Referencia: Rouse, J., R. Haas, J. Schell y D. Deering. «Monitoring Vegetation Systems in the Great Plains with ERTS». Tercer Simposio ERTS, NASA (1973): 309-317._

***

## NLI - Índice no lineal

Este índice parte de la base de que la relación entre muchos índices de vegetación y los parámetros biofísicos de superficie es no lineal. Linealiza las relaciones con los parámetros de superficie que tienden a ser no lineales.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Referencia: Goel, N. y W. Qin. «Influencias de la arquitectura del dosel en las relaciones entre diversos índices de vegetación y LAI y Fpar: una simulación por ordenador». Remote Sensing Reviews 10 (1994): 309-347._

***

## OSAVI - Índice de vegetación ajustado al suelo optimizado

Este índice se basa en el Índice de vegetación ajustado al suelo (SAVI). Utiliza un valor estándar de 0,16 para el factor de ajuste del fondo del dosel. Rondeaux (1996) determinó que este valor proporciona una mayor variación del suelo que el SAVI en casos de baja cobertura vegetal, al tiempo que muestra una mayor sensibilidad ante una cobertura vegetal superior al 50 %. Este índice resulta más adecuado en zonas con vegetación relativamente escasa, donde el suelo es visible a través del dosel.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Referencia: Rondeaux, G., M. Steven y F. Baret. «Optimization of Soil-Adjusted Vegetation Indices». Remote Sensing of Environment 55 (1996): 95-107._

***

## RDVI - Índice de vegetación por diferencia renormalizada

Este índice utiliza la diferencia entre las longitudes de onda del infrarrojo cercano y del rojo, junto con el NDVI, para resaltar la vegetación sana. Es insensible a los efectos del suelo y a la geometría de observación solar.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Referencia: Roujean, J. y F. Breon. «Estimación de la PAR absorbida por la vegetación a partir de mediciones de reflectancia bidireccionales». Remote Sensing of Environment 51 (1995): 375-384._

***

## SAVI - Índice de vegetación ajustado al suelo

Este índice es similar al NDVI, pero suprime los efectos de los píxeles de suelo. Utiliza un factor de ajuste del fondo del dosel, _L_, que es una función de la densidad de la vegetación y que a menudo requiere un conocimiento previo de la cantidad de vegetación. Huete (1988) sugiere un valor óptimo de _L_ = 0,5 para tener en cuenta las variaciones de primer orden del fondo del suelo. Este índice se utiliza mejor en zonas con vegetación relativamente escasa, donde el suelo es visible a través del dosel.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Referencia: Huete, A. «A Soil-Adjusted Vegetation Index (SAVI)». Remote Sensing of Environment 25 (1988): 295-309._

***

## TDVI - Índice de vegetación de diferencia transformada

Este índice resulta útil para el seguimiento de la cobertura vegetal en entornos urbanos. No se satura como el NDVI ni el SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Referencia: Bannari, A., H. Asalhi y P. Teillet. «Índice de diferencia de vegetación transformado (TDVI) para la cartografía de la cobertura vegetal». En Actas del Simposio de Geociencias y Teledetección, IGARSS &#x27;02, IEEE International, volumen 5 (2002)._

***

## VARI - Índice visible resistente a los efectos atmosféricos

Este índice se basa en el ARVI y se utiliza para estimar la fracción de vegetación en una escena con baja sensibilidad a los efectos atmosféricos.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Referencia: Gitelson, A., et al. «Líneas de vegetación y suelo en el espacio espectral visible: un concepto y una técnica para la estimación remota de la fracción de vegetación». International Journal of Remote Sensing 23 (2002): 2537-2562._

***

## WDRVI - Índice de vegetación de amplio rango dinámico

Este índice es similar al NDVI, pero utiliza un coeficiente de ponderación (_a_) para reducir la disparidad entre las contribuciones de las señales del infrarrojo cercano y del rojo al NDVI. El WDRVI resulta especialmente eficaz en escenas con una densidad de vegetación de moderada a alta cuando el NDVI supera el valor de 0,6. El NDVI tiende a estabilizarse a medida que aumentan la fracción de vegetación y el índice de área foliar (LAI) aumentan, mientras que el WDRVI es más sensible a un rango más amplio de fracciones de vegetación y a los cambios en el LAI.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

El coeficiente de ponderación (_a_) puede oscilar entre 0,1 y 0,2. Henebry, Viña y Gitelson (2004) recomiendan un valor de 0,2.

_Referencias_

_Gitelson, A. «Índice de vegetación de amplio rango dinámico para la cuantificación remota de las características biofísicas de la vegetación». Journal of Plant Physiology 161, n.º 2 (2004): 165-173._

_Henebry, G., A. Viña y A. Gitelson. «El índice de vegetación de amplio rango dinámico y su utilidad potencial para el análisis de brechas». Gap Analysis Bulletin 12: 50-56._

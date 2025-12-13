---
description: This page lists some multispectral indices that Chloros uses.
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---
# Fórmulas del índice multiespectral

Las siguientes fórmulas del índice utilizan una combinación de los rangos de transmisión media del filtro Survey3:

<table><thead><tr><th align="center">Survey3 Color del filtro</th><th width="196.199951171875" align="center">Survey3 Nombre del filtro</th><th width="159.800048828125" align="center">Rango de transmisión (FWHM)</th><th align="center">Transmisión media</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468-483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476-512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543-558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598-640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653-668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712-735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798-848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835-865 nm</td><td align="center">850 nm</td></tr></tbody></table>

Cuando se utilizan estas fórmulas, el nombre puede terminar en «\_1» o «\_2», lo que corresponde al filtro NIR, ya sea NIR1 o NIR2.

***

## EVI - Índice de vegetación mejorado

Este índice se desarrolló originalmente para su uso con datos MODIS como una mejora con respecto a NDVI, optimizando la señal de vegetación en áreas con un índice de área foliar alto (LAI). Es más útil en regiones con un LAI alto, donde el NDVI puede saturarse. Utiliza la región de reflectancia azul para corregir las señales de fondo del suelo y reducir las influencias atmosféricas, incluida la dispersión de aerosoles.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Los valores EVI deben oscilar entre 0 y 1 para los píxeles de vegetación. Las características brillantes, como las nubes y los edificios blancos, junto con las características oscuras, como el agua, pueden dar lugar a valores de píxeles anómalos en una imagen EVI. Antes de crear una imagen EVI, debe enmascarar las nubes y las características brillantes de la imagen de reflectancia y, opcionalmente, establecer un umbral para los valores de píxeles de 0 a 1.

_Referencia: Huete, A., et al. «Overview of the Radiometric and Biophysical Performance of the MODIS Vegetation Indices» (Resumen del rendimiento radiométrico y biofísico de los índices de vegetación MODIS). Remote Sensing of Environment 83 (2002): 195-213.

***

## FCI1 - Índice de cobertura forestal 1

Este índice distingue las copas de los bosques de otros tipos de vegetación utilizando imágenes de reflectancia multiespectral que incluyen una banda de borde rojo.

$$
FCI1 = Red * RedEdge
$$

Las zonas boscosas tendrán valores FCI1 más bajos debido a la menor reflectancia de los árboles y a la presencia de sombras dentro de la copa.

_Referencia: Becker, Sarah J., Craig S.T. Daughtry y Andrew L. Russ. «Índices robustos de cobertura forestal para imágenes multiespectrales». Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## FCI2 - Índice de cobertura forestal 2

Este índice distingue las copas de los árboles de otros tipos de vegetación utilizando imágenes de reflectancia multiespectral que no incluyen una banda de borde rojo.

$$
FCI2 = Red * NIR
$$

Las zonas boscosas tendrán valores FCI2 más bajos debido a la menor reflectancia de los árboles y a la presencia de sombras dentro del dosel.

_Referencia: Becker, Sarah J., Craig S.T. Daughtry y Andrew L. Russ. «Índices robustos de cobertura forestal para imágenes multiespectrales». Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## GEMI - Índice de monitorización medioambiental global

Este índice de vegetación no lineal se utiliza para la monitorización medioambiental global a partir de imágenes satelitales e intenta corregir los efectos atmosféricos. Es similar al NDVI, pero es menos sensible a los efectos atmosféricos. Se ve afectado por el suelo desnudo, por lo que no se recomienda su uso en zonas con vegetación escasa o moderadamente densa.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Donde:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Referencia: Pinty, B. y M. Verstraete. GEMI: un índice no lineal para monitorizar la vegetación global a partir de satélites. Vegetación 101 (1992): 15-20._

***

## GARI - Green Índice resistente a la atmósfera

Este índice es más sensible a una amplia gama de concentraciones de clorofila y menos sensible a los efectos atmosféricos que NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

La constante gamma es una función de ponderación que depende de las condiciones de los aerosoles en la atmósfera. ENVI utiliza un valor de 1,7, que es el valor recomendado por Gitelson, Kaufman y Merzylak (1996, página 296).

_Referencia: Gitelson, A., Y. Kaufman y M. Merzylak. «Use of a Green Channel in Remote Sensing of Global Vegetation from EOS-MODIS». Remote Sensing of Environment 58 (1996): 289-298._

***

## GCI - Green Índice de clorofila

Este índice se utiliza para estimar el contenido de clorofila de las hojas en una amplia gama de especies vegetales.

$$
GCI = {NIR \over Green} - 1
$$

Disponer de amplias longitudes de onda NIR y verdes proporciona una mejor predicción del contenido de clorofila, al tiempo que permite una mayor sensibilidad y una relación señal-ruido más alta.

_Referencia: Gitelson, A., Y. Gritz y M. Merzlyak. «Relaciones entre el contenido de clorofila en las hojas y la reflectancia espectral y algoritmos para la evaluación no destructiva de la clorofila en las hojas de plantas superiores». Journal of Plant Physiology 160 (2003): 271-282._

***

## GLI - Green Índice foliar

Este índice se diseñó originalmente para su uso con una cámara digital RGB para medir la cobertura del trigo, donde los números digitales (DN) rojos, verdes y azules oscilan entre 0 y 255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

Los valores GLI oscilan entre -1 y +1. Los valores negativos representan el suelo y los elementos no vivos, mientras que los valores positivos representan las hojas y los tallos verdes.

_Referencia: Louhaichi, M., M. Borman y D. Johnson. «Plataforma localizada espacialmente y fotografía aérea para la documentación de los impactos del pastoreo en el trigo». Geocarto International 16, n.º 1 (2001): 65-70._

***

## GNDVI - Green Índice de vegetación de diferencia normalizada

Este índice es similar al NDVI, salvo que mide el espectro verde de 540 a 570 nm en lugar del espectro rojo. Este índice es más sensible a la concentración de clorofila que el NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Referencia: Gitelson, A. y M. Merzlyak. «Remote Sensing of Chlorophyll Concentration in Higher Plant Leaves» (Teledetección de la concentración de clorofila en las hojas de las plantas superiores). Advances in Space Research 22 (1998): 689-692._

***

## GOSAVI - Green Índice de vegetación ajustado al suelo optimizado

Este índice se diseñó originalmente con fotografía infrarroja en color para predecir las necesidades de nitrógeno del maíz. Es similar al OSAVI, pero sustituye la banda verde por la roja.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Referencia: Sripada, R., et al. «Determinación de las necesidades de nitrógeno del maíz durante la temporada mediante fotografía aérea infrarroja en color». Tesis doctoral, Universidad Estatal de Carolina del Norte, 2005._

***

## GRVI - Green Índice de vegetación de relación

Este índice es sensible a las tasas de fotosíntesis en las copas de los bosques, ya que las reflectancias verde y roja se ven muy influidas por los cambios en los pigmentos de las hojas.

$$
GRVI = {NIR \over Green }
$$

_Referencia: Sripada, R., et al. «Fotografía aérea en color infrarrojo para determinar las necesidades de nitrógeno del maíz al inicio de la temporada». Agronomy Journal 98 (2006): 968-977._

***

## GSAVI - Green Índice de vegetación ajustado al suelo

Este índice se diseñó originalmente con fotografía infrarroja en color para predecir las necesidades de nitrógeno del maíz. Es similar al SAVI, pero sustituye la banda verde por la roja.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Referencia: Sripada, R., et al. «Determinación de las necesidades de nitrógeno durante la temporada para el maíz utilizando fotografía aérea en color infrarrojo». Tesis doctoral, Universidad Estatal de Carolina del Norte, 2005._

***

## LAI - Índice de área foliar

Este índice se utiliza para estimar la cobertura del follaje y pronosticar el crecimiento y el rendimiento de los cultivos. ENVI calcula el LAI verde utilizando la siguiente fórmula empírica de Boegh et al (2002):

$$
LAI = 3.618 * EVI - 0.118
$$

Donde EVI es:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Los valores altos de LAI suelen oscilar entre 0 y 3,5 aproximadamente. Sin embargo, cuando la escena contiene nubes y otros elementos brillantes que producen píxeles saturados, los valores de LAI pueden superar 3,5. Lo ideal es enmascarar las nubes y los elementos brillantes de la escena antes de crear una imagen LAI.

_Referencia: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde y A. Thomsen. «Datos multiespectrales aéreos para cuantificar el índice de área foliar, la concentración de nitrógeno y la eficiencia fotosintética en la agricultura». Remote Sensing of Environment 81, n.º 2-3 (2002): 179-193._

***

## LCI - Índice de clorofila foliar

Este índice se utiliza para estimar el contenido de clorofila en plantas superiores, sensibles a la variación de la reflectancia causada por la absorción de clorofila.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Referencia: Datt, B. «Teledetección del contenido de agua en las hojas de eucalipto». Journal of Plant Physiology 154, n.º 1 (1999): 30-36._

***

## MNLI: índice no lineal modificado

Este índice es una mejora del índice no lineal (NLI) que incorpora el índice de vegetación ajustado al suelo (SAVI) para tener en cuenta el fondo del suelo. ENVI utiliza un valor de factor de ajuste del fondo del dosel (_L_) de 0,5.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Referencia: Yang, Z., P. Willis y R. Mueller. «Impacto de la imagen AWIFS mejorada con relación de banda en la precisión de la clasificación de cultivos». Actas del Simposio de Teledetección Pecora 17 (2008), Denver, CO._

***

## MSAVI2 - Índice de vegetación ajustado al suelo modificado 2

Este índice es una versión más simple del índice MSAVI propuesto por Qi et al (1994), que mejora el índice de vegetación ajustado al suelo (SAVI). Reduce el ruido del suelo y aumenta el rango dinámico de la señal de vegetación. El MSAVI2 se basa en un método inductivo que no utiliza un valor _L_ constante (como en el caso del SAVI) para resaltar la vegetación sana.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Referencia: Qi, J., A. Chehbouni, A. Huete, Y. Kerr y S. Sorooshian. «A Modified Soil Adjusted Vegetation Index» (Índice de vegetación ajustado al suelo modificado). Remote Sensing of Environment 48 (1994): 119-126._

***

## NDRE: diferencia normalizada RedEdge

Este índice es similar al NDVI, pero compara el contraste entre NIR y RedEdge en lugar de Red, lo que a menudo detecta antes el estrés de la vegetación.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI - Índice de vegetación de diferencia normalizada

Este índice es una medida de la vegetación verde y saludable. La combinación de su formulación de diferencia normalizada y el uso de las regiones de mayor absorción y reflectancia de la clorofila lo hacen robusto en una amplia gama de condiciones. Sin embargo, puede saturarse en condiciones de vegetación densa cuando LAI se vuelve alto.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

El valor de este índice oscila entre -1 y 1. El rango común para la vegetación verde es de 0,2 a 0,8.

_Referencia: Rouse, J., R. Haas, J. Schell y D. Deering. Monitorización de los sistemas de vegetación en las Grandes Llanuras con ERTS. Tercer Simposio ERTS, NASA (1973): 309-317._

***

## NLI - Índice no lineal

Este índice asume que la relación entre muchos índices de vegetación y los parámetros biofísicos de la superficie no es lineal. Linealiza las relaciones con los parámetros de la superficie que tienden a ser no lineales.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Referencia: Goel, N. y W. Qin. «Influencias de la arquitectura del dosel en las relaciones entre diversos índices de vegetación y LAI y Fpar: una simulación por ordenador». Remote Sensing Reviews 10 (1994): 309-347._

***

## OSAVI - Índice de vegetación ajustado al suelo optimizado

Este índice se basa en el índice de vegetación ajustado al suelo (SAVI). Utiliza un valor estándar de 0,16 para el factor de ajuste del fondo del dosel. Rondeaux (1996) determinó que este valor proporciona una mayor variación del suelo que SAVI para una cobertura vegetal baja, al tiempo que demuestra una mayor sensibilidad a una cobertura vegetal superior al 50 %. Este índice se utiliza mejor en zonas con vegetación relativamente escasa, donde el suelo es visible a través del dosel.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Referencia: Rondeaux, G., M. Steven y F. Baret. «Optimization of Soil-Adjusted Vegetation Indices» (Optimización de los índices de vegetación ajustados al suelo). Remote Sensing of Environment 55 (1996): 95-107._

***

## RDVI: índice de vegetación de diferencia renormalizada

Este índice utiliza la diferencia entre las longitudes de onda del infrarrojo cercano y el rojo, junto con el NDVI, para resaltar la vegetación sana. Es insensible a los efectos del suelo y la geometría de observación solar.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Referencia: Roujean, J. y F. Breon. «Estimación de la PAR absorbida por la vegetación a partir de mediciones de reflectancia bidireccional». Remote Sensing of Environment 51 (1995): 375-384._

***

## SAVI - Índice de vegetación ajustado al suelo

Este índice es similar al NDVI, pero suprime los efectos de los píxeles del suelo. Utiliza un factor de ajuste del fondo del dosel, _L_, que es una función de la densidad de la vegetación y a menudo requiere un conocimiento previo de la cantidad de vegetación. Huete (1988) sugiere un valor óptimo de _L_=0,5 para tener en cuenta las variaciones de primer orden del fondo del suelo. Este índice se utiliza mejor en áreas con vegetación relativamente escasa, donde el suelo es visible a través del dosel.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Referencia: Huete, A. «A Soil-Adjusted Vegetation Index (SAVI)». Remote Sensing of Environment 25 (1988): 295-309._

***

## TDVI - Índice de vegetación por diferencia transformada

Este índice es útil para monitorizar la cobertura vegetal en entornos urbanos. No se satura como NDVI y SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Referencia: Bannari, A., H. Asalhi y P. Teillet. «Índice de vegetación transformado (TDVI) para la cartografía de la cobertura vegetal». En Actas del Simposio de Geociencias y Teledetección, IGARSS &#x27;02, IEEE International, volumen 5 (2002)._

***

## VARI - Índice visible resistente a la atmósfera

Este índice se basa en el ARVI y se utiliza para estimar la fracción de vegetación en una escena con baja sensibilidad a los efectos atmosféricos.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Referencia: Gitelson, A., et al. «Vegetación y líneas de suelo en el espacio espectral visible: un concepto y una técnica para la estimación remota de la fracción de vegetación». Revista Internacional de Teledetección 23 (2002): 2537-2562._

***

## WDRVI - Índice de vegetación de amplio rango dinámico

Este índice es similar al NDVI, pero utiliza un coeficiente de ponderación (_a_) para reducir la disparidad entre las contribuciones de las señales del infrarrojo cercano y el rojo al NDVI. El WDRVI es especialmente eficaz en escenas con una densidad de vegetación de moderada a alta cuando el NDVI supera 0,6. El NDVI tiende a estabilizarse cuando aumentan la fracción de vegetación y el índice de área foliar (LAI), mientras que el WDRVI es más sensible a una gama más amplia de fracciones de vegetación y a los cambios en el LAI.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

El coeficiente de ponderación (_a_) puede oscilar entre 0,1 y 0,2. Henebry, Viña y Gitelson (2004) recomiendan un valor de 0,2.

_Referencias_

_Gitelson, A. «Índice de vegetación de amplio rango dinámico para la cuantificación remota de las características biofísicas de la vegetación». Journal of Plant Physiology 161, n.º 2 (2004): 165-173._

_Henebry, G., A. Viña y A. Gitelson. «The Wide Dynamic Range Vegetation Index and its Potential Utility for Gap Analysis» (El índice de vegetación de amplio rango dinámico y su utilidad potencial para el análisis de brechas). Gap Analysis Bulletin 12: 50-56._

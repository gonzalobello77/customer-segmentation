# Segmentación de clientes de tarjetas de crédito

Proyecto de Machine Learning no supervisado orientado a identificar distintos
perfiles de clientes a partir de sus patrones de uso de tarjetas de crédito.

El análisis comprende la exploración y preparación de los datos, el estudio de
su estructura para clustering, la construcción y evaluación de una segmentación
mediante K-Means, la caracterización de los segmentos obtenidos y el uso de
técnicas complementarias de visualización y clustering.

## Problema

La segmentación de clientes permite identificar grupos con comportamientos
similares y puede servir como apoyo para diseñar estrategias diferenciadas de
marketing, comunicación y gestión de clientes.

En este proyecto se analizan variables relacionadas con saldo, compras,
adelantos de efectivo, pagos, límites de crédito y frecuencia de uso, con el
objetivo de encontrar patrones de comportamiento sin disponer previamente de
una variable objetivo o etiquetas de segmento.

## Datos

El conjunto de datos contiene información sobre aproximadamente 9.000 titulares
activos de tarjetas de crédito durante un período de seis meses.

Cada fila representa un cliente y la base original contiene **8.950
observaciones y 18 variables de comportamiento e identificación**.

**Fuente:** [Credit Card Dataset](https://www.kaggle.com/datasets/arjunbhasin2013/ccdata)

Las descripciones siguientes corresponden a una traducción y adaptación del
diccionario proporcionado por la fuente original.

### Diccionario de variables

| Variable | Descripción |
|---|---|
| `CUST_ID` | Identificador del titular de la tarjeta de crédito. |
| `BALANCE` | Saldo disponible o mantenido en la cuenta para realizar compras. |
| `BALANCE_FREQUENCY` | Frecuencia con la que se actualiza el saldo, entre 0 y 1. |
| `PURCHASES` | Monto total de compras realizadas. |
| `ONEOFF_PURCHASES` | Monto asociado a compras realizadas de una sola vez, en contraste con compras financiadas en cuotas. |
| `INSTALLMENTS_PURCHASES` | Monto de compras realizadas en cuotas. |
| `CASH_ADVANCE` | Monto obtenido mediante adelantos de efectivo. |
| `PURCHASES_FREQUENCY` | Frecuencia con la que se realizan compras, entre 0 y 1. |
| `ONEOFF_PURCHASES_FREQUENCY` | Frecuencia de compras realizadas en una sola operación. |
| `PURCHASES_INSTALLMENTS_FREQUENCY` | Frecuencia de compras realizadas en cuotas. |
| `CASH_ADVANCE_FREQUENCY` | Frecuencia de utilización de adelantos de efectivo. |
| `CASH_ADVANCE_TRX` | Número de transacciones de adelanto de efectivo. |
| `PURCHASES_TRX` | Número de transacciones de compra realizadas. |
| `CREDIT_LIMIT` | Límite de crédito de la tarjeta. |
| `PAYMENTS` | Monto total de pagos realizados por el cliente. |
| `MINIMUM_PAYMENTS` | Monto mínimo de pagos realizados. |
| `PRC_FULL_PAYMENT` | Proporción de pagos realizados por el monto completo. |
| `TENURE` | Antigüedad observada del servicio de tarjeta, expresada en meses. |

## Flujo del proyecto

El proyecto se divide en dos etapas principales:

1. **Preparación y análisis exploratorio**
   - inspección y calidad de los datos;
   - tratamiento de valores faltantes;
   - análisis de distribuciones, asimetrías y valores extremos;
   - análisis de relaciones entre variables;
   - generación de la base procesada.

2. **Modelado y segmentación**
   - transformaciones para métodos basados en distancias;
   - estandarización;
   - evaluación de tendencia al agrupamiento;
   - selección y validación del número de clusters;
   - K-Means y perfilamiento de segmentos;
   - PCA para visualización;
   - comparación exploratoria con DBSCAN.

## Preparación y análisis exploratorio

La exploración inicial confirmó **8.950 clientes y 18 variables**. La mayoría
de las características son numéricas, mientras que `CUST_ID` funciona
únicamente como identificador y no aporta información de comportamiento para
el clustering.

### Calidad de los datos

Se revisaron valores faltantes y registros duplicados.

Se detectó que dos variables tienen valores nulos:

| Variable | Valores nulos | Porcentaje |
|---|---:|---:|
| `CREDIT_LIMIT` | 1 | 0.01% |
| `MINIMUM_PAYMENTS` | 313 | 3.50% |

`CREDIT_LIMIT` representa el límite de crédito asignado al cliente, mientras que
`MINIMUM_PAYMENTS` corresponde al monto asociado a pagos mínimos.

Dado el contexto se planea imputar, y antes de hacerlo se compara la media y la mediana de
ambas variables:

| Variable | Media | Mediana |
|---|---:|---:|
| `CREDIT_LIMIT` | 4494.45 | 3000.00 |
| `MINIMUM_PAYMENTS` | 864.21 | 312.34 |

En ambos casos la media supera la mediana lo que sugiere que las
distribuciones pueden estar influenciadas por valores elevados, por ello se imputa con la **mediana** ya que resulta menos sensible a valores extremos que la media.

`CUST_ID` se eliminó antes del modelado porque funciona únicamente como
identificador del cliente.

Después de estas decisiones, la base procesada conserva **8.950 clientes y
17 variables de comportamiento**.

### Distribución de las variables

|                                  |   count |    mean |     std |   min |     25% |     50% |     75% |     max |
|:---------------------------------|--------:|--------:|--------:|------:|--------:|--------:|--------:|--------:|
| BALANCE                          |    8950 | 1564.47 | 2081.53 |  0    |  128.28 |  873.39 | 2054.14 | 19043.1 |
| BALANCE_FREQUENCY                |    8950 |    0.88 |    0.24 |  0    |    0.89 |    1    |    1    |     1   |
| PURCHASES                        |    8950 | 1003.2  | 2136.63 |  0    |   39.64 |  361.28 | 1110.13 | 49039.6 |
| ONEOFF_PURCHASES                 |    8950 |  592.44 | 1659.89 |  0    |    0    |   38    |  577.4  | 40761.2 |
| INSTALLMENTS_PURCHASES           |    8950 |  411.07 |  904.34 |  0    |    0    |   89    |  468.64 | 22500   |
| CASH_ADVANCE                     |    8950 |  978.87 | 2097.16 |  0    |    0    |    0    | 1113.82 | 47137.2 |
| PURCHASES_FREQUENCY              |    8950 |    0.49 |    0.4  |  0    |    0.08 |    0.5  |    0.92 |     1   |
| ONEOFF_PURCHASES_FREQUENCY       |    8950 |    0.2  |    0.3  |  0    |    0    |    0.08 |    0.3  |     1   |
| PURCHASES_INSTALLMENTS_FREQUENCY |    8950 |    0.36 |    0.4  |  0    |    0    |    0.17 |    0.75 |     1   |
| CASH_ADVANCE_FREQUENCY           |    8950 |    0.14 |    0.2  |  0    |    0    |    0    |    0.22 |     1.5 |
| CASH_ADVANCE_TRX                 |    8950 |    3.25 |    6.82 |  0    |    0    |    0    |    4    |   123   |
| PURCHASES_TRX                    |    8950 |   14.71 |   24.86 |  0    |    1    |    7    |   17    |   358   |
| CREDIT_LIMIT                     |    8950 | 4494.28 | 3638.65 | 50    | 1600    | 3000    | 6500    | 30000   |
| PAYMENTS                         |    8950 | 1733.14 | 2895.06 |  0    |  383.28 |  856.9  | 1901.13 | 50721.5 |
| MINIMUM_PAYMENTS                 |    8950 |  844.91 | 2332.79 |  0.02 |  170.86 |  312.34 |  788.71 | 76406.2 |
| PRC_FULL_PAYMENT                 |    8950 |    0.15 |    0.29 |  0    |    0    |    0    |    0.14 |     1   |
| TENURE                           |    8950 |   11.52 |    1.34 |  6    |   12    |   12    |   12    |    12   |

Las variables presentan comportamientos considerablemente diferentes según
su naturaleza. Los montos y conteos muestran, en varios casos, una fuerte
concentración en valores bajos junto con colas derechas pronunciadas,
especialmente en compras, adelantos de efectivo y pagos.

![Distribución de las variables](figures/feature_distributions.png)


### Análisis de asimetría

El coeficiente de asimetría confirma lo observado previamente en las
distribuciones: varias variables presentan colas pronunciadas,
especialmente aquellas asociadas a montos y actividad transaccional.

|                                  |       |
|:---------------------------------|------:|
| MINIMUM_PAYMENTS                 | 13.85 |
| ONEOFF_PURCHASES                 | 10.05 |
| PURCHASES                        |  8.14 |
| INSTALLMENTS_PURCHASES           |  7.3  |
| PAYMENTS                         |  5.91 |
| CASH_ADVANCE_TRX                 |  5.72 |
| CASH_ADVANCE                     |  5.17 |
| PURCHASES_TRX                    |  4.63 |
| BALANCE                          |  2.39 |
| PRC_FULL_PAYMENT                 |  1.94 |
| CASH_ADVANCE_FREQUENCY           |  1.83 |
| ONEOFF_PURCHASES_FREQUENCY       |  1.54 |
| CREDIT_LIMIT                     |  1.52 |
| PURCHASES_INSTALLMENTS_FREQUENCY |  0.51 |
| PURCHASES_FREQUENCY              |  0.06 |
| BALANCE_FREQUENCY                | -2.02 |
| TENURE                           | -2.94 |

Dado que el modelado posterior utiliza **K-Means**, un método basado en
distancias, estas distribuciones pueden influir de forma desproporcionada en la
formación de los grupos.

Por ello, en la etapa de modelado se evaluarán transformaciones para reducir la
asimetría de las variables pertinentes y posteriormente se aplicará
estandarización para homogeneizar sus escalas.


### Boxplot de variables

Se muestran que para la mayoría de variables se tienen numerosos valores extremos, especialmente en variables monetarias y transaccionales. En este contexto pueden representar comportamientos legítimos de clientes —por ejemplo,
clientes con montos de compra, saldo o adelantos particularmente elevados— por lo que no se asume necesariamente errores de registro.

Su influencia sobre los métodos basados en distancias se aborda posteriormente mediante transformaciones y estandarización.

![Boxplot de variables](figures/boxplots.png)
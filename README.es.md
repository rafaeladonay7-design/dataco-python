# DataCo: pronóstico de demanda y punto de reorden con Python

[English](README.md) · **Español**

**Caso de estudio con datos públicos** · Python (pandas, matplotlib) · Rafael Adonay Rocha Gutiérrez

## Escenario

Rol simulado: analista de planeación de inventario en DataCo, empresa que vende artículos deportivos, ropa y electrónica en 5 mercados. En mi [análisis con SQL](https://github.com/rafaeladonay7-design/dataco-sql) encontré que unos pocos productos generan la mayor parte de las ventas. La gerente de planeación pide:

1. ¿Qué productos concentran las ventas y necesitan control continuo de inventario?
2. ¿Qué método de pronóstico funciona mejor para ellos?
3. ¿Cuánto stock de seguridad tener y en qué punto reordenar para un nivel de servicio de 95%?

**Análisis completo:** [`analisis_inventario.ipynb`](analisis_inventario.ipynb) (código, tablas y gráficas).

## Datos

[DataCo Smart Supply Chain for Big Data Analysis](https://www.kaggle.com/datasets/shashwatwork/dataco-smart-supply-chain-for-big-data-analysis) (Kaggle): 180,519 líneas de pedido, de enero 2015 a enero 2018.

## Preparación de los datos

- El CSV viene en Latin-1. Las fechas vienen como texto `M/D/YYYY H:MM` y las convierto con `pd.to_datetime`.
- Desde octubre 2017 las unidades por mes caen de ~11,000 a ~2,000, así que uso **enero 2015 a septiembre 2017** (33 meses completos).
- Quito los pedidos cancelados y los sospechosos de fraude (7,754 líneas): no son demanda que haya que surtir.
- La demanda diaria incluye los días sin venta, para no subestimar la variación.

## Hallazgos clave

| # | Hallazgo | Resultado |
|---|---|---|
| 1 | Concentración | **7 productos (7%)** generan **82.5%** de las ventas |
| 2 | Estabilidad | La demanda mensual de la clase A varía solo **4% a 8%** (coeficiente de variación), sin tendencia ni temporada |
| 3 | Pronóstico | El **promedio histórico** tiene el menor error (**4.3%** WAPE) contra 6.1% del método ingenuo, y gana en 5 de 7 productos |
| 4 | Stock de seguridad | Con 95% de servicio y 7 días de entrega bastan **304 unidades (~$31,000)**: de 1 a 1.5 días de demanda por producto |
| 5 | Lo que más pesa | Que el proveedor tarde 30 días en vez de 7 **duplica** el stock de seguridad (+107%); subir el servicio de 95% a 99% lo sube 42% |

![Pareto ABC](figuras/01_pareto_abc.png)

![Error del pronóstico por método](figuras/03_error_pronostico.png)

## Método

- **Clasificación ABC:** A = primer 80% de las ventas, B = siguiente 15%, C = último 5%. Al quitar cancelados, fraude y meses incompletos, la clase A queda en 7 productos en vez de los 8 del proyecto SQL.
- **Prueba retrospectiva del pronóstico:** para cada mes de enero a septiembre 2017, cada método pronostica solo con los meses anteriores. Comparo: ingenuo, promedio móvil de 3 y 6 meses, estacional ingenuo y promedio histórico. Error: WAPE (suma de errores absolutos / demanda real).
- **Stock de seguridad:** SS = z × σ(demanda diaria) × √LT; punto de reorden = demanda diaria promedio × LT + SS; z = 1.645 para 95%.
- **Sensibilidad:** valor del stock de seguridad con 90/95/99% de servicio y 7/14/30 días de entrega.

## Recomendaciones

1. **Revisión continua para los 7 productos clase A** con los puntos de reorden de la sección 5 del notebook. Las clases B y C pueden ir con revisión periódica.
2. **Pronosticar con el promedio histórico** y medir el error cada mes. Un modelo más complejo no se justifica con demanda tan estable; si el error pasa de ~10%, revisar si cambió el patrón.
3. **Medir y negociar el tiempo de entrega de los proveedores** antes que subir el nivel de servicio. Es la variable que más mueve el inventario y hoy no se registra.

## Limitaciones

- El tiempo de entrega de 7 días es un supuesto: el dataset no trae datos de proveedores.
- El stock de seguridad solo considera la variación de la demanda, no la del tiempo de entrega.
- El inventario está valuado a precio de venta porque el dataset no trae el costo.
- La demanda es más estable de lo normal en datos reales, lo que sugiere que el dataset es en parte simulado.

## Cómo correrlo

```bash
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
# Descarga DataCoSupplyChainDataset.csv de Kaggle y ponlo en data/
jupyter notebook analisis_inventario.ipynb
```

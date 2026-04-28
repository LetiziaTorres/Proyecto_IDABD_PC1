## PC1 – Propuesta y EDA Inicial
## Caso: ¿Qué hace que un producto triunfe en e-commerce?

# Proyecto MarketPulse Analytics - Análisis de Satisfacción del Cliente (Olist)

## 1. Portada del equipo

| Rol | Integrante |
| :--- | :--- |
| EDA | Camila Torres |
| --- | Integrante2 |
| Hipótesis | Letizia Torres |
| ---- | Integrante4 |
| ----- | Integrante5 |

## 2. Descripción del problema de negocio

El caso se basa en **Olist**, una plataforma brasileña de e-commerce que conecta a pequeños vendedores con canales de venta en línea. Su dataset público en Kaggle contiene información real sobre pedidos, productos, reseñas, clientes y logística.

### ¿Cuál es el problema? 

No todos los productos que se venden mucho logran mantener altos niveles de satisfacción ni generar recompra. Los marketplaces y vendedores digitales desconocen qué factores influyen más en ese éxito: si el precio, la categoría, el tiempo de entrega o la calificación del vendedor. Sin esta claridad, es difícil priorizar mejoras que realmente impacten en los resultados.


**Problema:**  
> *¿Qué factores (precio, categoría, tiempo de entrega, calificación del vendedor) predicen mejor la satisfacción del cliente y la recompra?*

**Relevancia:**  
Responder esta pregunta permite a marketplaces y vendedores digitales enfocar sus esfuerzos en lo que realmente importa para escalar ventas. Es decir, pasar de intuiciones a decisiones respaldadas por datos, mejorando tanto la experiencia del cliente como la rentabilidad del negocio.

## 3. Descripción de los datos

### Fuente principal — Brazilian E-Commerce (Olist)

- **Origen:** Kaggle – olistbr/brazilian-ecommerce
- **Período:** Septiembre 2016 – Octubre 2018

### Archivos y registros

| Dataset | Registros | Descripción |
|---------|-----------|-------------|
| olist_orders_dataset.csv | 99,441 | Órdenes con timestamps y estado |
| olist_order_items_dataset.csv | 112,650 | Ítems por orden (precio, flete) |
| olist_order_reviews_dataset.csv | 104,719 | Reseñas (score 1–5, comentarios) |
| olist_customers_dataset.csv | 99,441 | Clientes (ciudad, estado) |
| olist_products_dataset.csv | 32,951 | Productos (categoría, dimensiones, fotos) |
| olist_sellers_dataset.csv | 3,095 | Vendedores (localización) |
| olist_order_payments_dataset.csv | 103,886 | Métodos y montos de pago |
| olist_geolocation_dataset.csv | ~1M | Coordenadas por código postal |
| product_category_name_translation.csv | 71 | Traducción PT→EN de categorías |

**Variables clave:** review_score, price, freight_value, order_status, order_delivered_customer_date, order_estimated_delivery_date, product_category_name, payment_type, payment_installments

### Fuente de enriquecimiento

- **Calendario de festivos Brasil:** API Nager.Date (festivos nacionales 2016–2018) → detectar picos de compra estacionales
- **Tipo de cambio BRL/USD histórico:** ExchangeRate-API o datos de Banco Central do Brasil → contextualizar el valor real de las transacciones
- **Tipo de cambio BRL/PEN:** BCRP para conversión a contexto peruano

## 4. EDA Inicial

> Las visualizaciones se generan en el notebook adjunto (`PF_Big data.ipynb`). A continuación se describe cada una con sus hallazgos clave.

### Viz 1 – Distribución de precios: 

Los precios presentan un sesgo positivo (hacia la derecha), donde la mayoría de productos se concentra en rangos bajos, mientras que existe una minoría con precios muy altos. Esto sugiere un mercado altamente competitivo con predominancia de productos accesibles.

### Viz 2 – Distribución de precio y flete (boxplot + histograma)

- **Precio:** media = 120.65 BRL | mediana = 74.99 BRL | max = 6,735 BRL
- **Flete:** media = 19.99 BRL | mediana = 16.26 BRL | max = 409.68 BRL

**Hallazgo:** Ambas variables tienen colas muy largas (outliers extremos). El 75% de los productos cuesta menos de 135 BRL. Se requerirá transformación logarítmica antes del modelado. El flete representa en promedio el 16.5% del precio del producto.

### Viz 3 – Nulos por columna (heatmap)

- review_comment_title: 87,656 nulos (83.7%)
- review_comment_message: 58,247 nulos (55.6%)
- order_delivered_customer_date: 2,965 nulos (3.0%)
- product_category_name: 610 nulos (1.9%)

**Hallazgo:** La ausencia de comentarios de texto es estructural (muchos clientes solo ponen score sin texto). Las fechas de entrega nulas corresponden a órdenes canceladas o en tránsito. **Estrategia:** imputar categorías por mediana de categorías similares; excluir órdenes sin fecha de entrega del análisis de tiempo.

### Viz 4 – review_score promedio por categoría de producto (top 15 y bottom 5)

**Hallazgo:** Categorías como `cds_dvds_musicals` y `fashion_childrens_clothes` tienen scores promedio >4.5, mientras que `security_and_services` y `office_furniture` promedian <3.5. Esto sugiere que la categoría per se es un predictor relevante de satisfacción.

### Viz 5 – Tiempo de entrega vs. review_score (días reales vs. estimados)

**Hallazgo:** Se observa correlación negativa entre días de retraso y review_score. Órdenes entregadas antes de la fecha estimada tienen score promedio de 4.3 vs. 2.9 para las que llegan tarde. El tiempo de entrega es probablemente el predictor más fuerte de insatisfacción, por encima del precio.

### Viz 6 – Método de pago vs. ticket promedio

| Método de pago | % transacciones | Ticket medio (BRL) |
|----------------|-----------------|--------------------|
| credit_card    | 73.9%           | 163                |
| boleto         | 19.1%           | 108                |
| voucher        | 5.5%            | 64                 |
| debit_card     | 1.5%            | 144                |

**Hallazgo:** El pago con tarjeta de crédito está asociado a tickets más altos. Los compradores de boleto (pago en efectivo, típico en Brasil) tienden a comprar productos más baratos. Esto puede relacionarse con el perfil socioeconómico y tolerancia al riesgo.

## 5. Hipótesis de negocio

Las siguientes hipótesis serán investigadas en la Etapa 2 mediante modelos predictivos y análisis estadísticos:

**H1 – Tiempo de entrega como predictor dominante**  
¿Es el tiempo de entrega (real vs. estimado) el factor con mayor peso predictivo sobre el review_score del cliente, por encima del precio o la categoría del producto?

**H2 – Efecto del precio relativo en la satisfacción**  
¿Los productos con una relación flete/precio alta (>30%) tienen significativamente menor review_score que los productos con flete bajo, independientemente de la categoría?

**H3 – Concentración de vendedores y calidad del servicio**  
¿Los vendedores con mayor volumen de ventas (top 10%) ofrecen tiempos de entrega más confiables y obtienen calificaciones más altas que los vendedores pequeños?

**H4 – Estacionalidad y satisfacción**  
¿Las compras realizadas durante periodos de alta demanda (Navidad, Black Friday, Día de los Niños en Brasil) presentan peores tiempos de entrega y menor review_score debido a la saturación logística?

**H5 – Recompra y calidad percibida**  
¿Los clientes que dieron 5 estrellas en su primera compra tienen mayor probabilidad de realizar una segunda compra en los siguientes 6 meses?

## 6. Enriquecimiento de datos

Además del dataset base de Olist, el proyecto propone incorporar fuentes de enriquecimiento que permitan contextualizar mejor el comportamiento de compra y la satisfacción del cliente. Este enriquecimiento forma parte de la planificación de la PC1 y servirá para profundizar el análisis en la siguiente etapa del proyecto.

### 6.1 Fuente principal

La fuente principal del proyecto es el dataset **Brazilian E-Commerce Public Dataset by Olist**, disponible en Kaggle. Este dataset contiene información de pedidos, productos, clientes, vendedores, pagos, reseñas y logística de entregas realizadas en Brasil entre 2016 y 2018.

En el análisis inicial se trabajó principalmente con las siguientes tablas:

| Dataset | Uso en el proyecto |
|---|---|
| `olist_orders_dataset.csv` | Permite analizar fechas de compra, entrega y estado del pedido |
| `olist_order_items_dataset.csv` | Permite analizar precio, flete y productos vendidos |
| `olist_order_reviews_dataset.csv` | Permite analizar la satisfacción del cliente mediante `review_score` |
| `olist_customers_dataset.csv` | Permite relacionar pedidos con clientes y ubicación |
| `olist_products_dataset.csv` | Permite analizar categoría y características del producto |

Estas tablas fueron integradas en una sola base analítica mediante operaciones de `merge`, lo que permitió relacionar precio, tiempo de entrega, categoría de producto y calificación del cliente.

### 6.2 Fuente de enriquecimiento seleccionada: calendario comercial y festivos

La fuente de enriquecimiento principal será un calendario de fechas relevantes en Brasil durante el periodo 2016–2018. Esta fuente permitirá identificar si una compra ocurrió cerca de una fecha especial, feriado o campaña comercial.

Se propone incorporar variables como:

| Variable nueva | Descripción | Utilidad para el análisis |
|---|---|---|
| `purchase_month` | Mes en que se realizó la compra | Permite detectar estacionalidad mensual |
| `purchase_day_of_week` | Día de la semana de la compra | Permite analizar patrones de compra por día |
| `is_holiday_period` | Indica si la compra ocurrió cerca de un feriado | Permite evaluar impacto de feriados en demanda y entrega |
| `is_black_friday_period` | Identifica compras cercanas a Black Friday | Permite analizar campañas de alta demanda |
| `is_christmas_period` | Identifica compras cercanas a Navidad | Permite analizar saturación logística de fin de año |

### 6.3 Justificación del enriquecimiento

El enriquecimiento con calendario es relevante porque el comportamiento del consumidor en e-commerce puede variar según la temporada. En periodos de alta demanda, como Black Friday o Navidad, es posible que aumente el volumen de pedidos y también los retrasos logísticos.

Esto puede afectar directamente el `review_score`, ya que una entrega tardía o una mala experiencia durante campañas comerciales podría reducir la satisfacción del cliente.

Por ello, esta fuente de enriquecimiento permitirá analizar no solo variables internas del pedido, como precio o categoría, sino también el contexto temporal en el que ocurrió la compra.

### 6.4 Fuente de enriquecimiento opcional: tipo de cambio BRL/PEN

Como fuente complementaria, se plantea usar el tipo de cambio histórico BRL/PEN para convertir los precios del dataset a soles peruanos.

Esta conversión no cambia el análisis original, ya que los precios están registrados en reales brasileños, pero puede facilitar la interpretación económica del proyecto desde el contexto peruano del equipo.

Variables posibles:

| Variable nueva | Descripción |
|---|---|
| `price_pen` | Precio convertido de BRL a PEN |
| `freight_value_pen` | Flete convertido de BRL a PEN |
| `total_value_pen` | Valor total aproximado de la compra en PEN |

### 6.5 Alcance del enriquecimiento en la PC1

En esta primera entrega, el enriquecimiento se presenta como una propuesta metodológica y una fuente identificada. La integración completa de estas variables se desarrollará posteriormente según el plan de trabajo.

El objetivo de la PC1 es dejar definida la fuente, justificar su utilidad y explicar cómo se conectará con las hipótesis de negocio.

## 7. Plan de trabajo – Etapa 2 (Semanas 7–14)

| Semana | Camila Torres (EDA) | Integrante2 (Datos) | Letizia Torres (Estadística) | Integrante4 (Viz) | Integrante5 (Enriquecimiento) |
|--------|------------------------|---------------------|---------------------------|--------------------|------------------------------|
| 7 | Definir pipeline de modelado | Limpieza profunda + merge de tablas | Pruebas estadísticas H1 y H2 | Dashboard interactivo v1 | Integrar festivos y tipo de cambio |
| 8 | Feature engineering (retraso, ratio flete) | Encoding de variables categóricas | Regresión logística baseline | Visualizaciones de correlación | Validar datos de enriquecimiento |
| 9 | Modelo Random Forest – review_score | Validación cruzada y métricas | Test H3 (vendedores) | Mapa geográfico de ventas | Merge datos externos al dataset |
| 10 | Tuning de hiperparámetros | Manejo de desbalance (SMOTE) | Test H4 (estacionalidad) | Gráficos de feature importance | Análisis de tipo de cambio |
| 11 | Modelo XGBoost comparativo | Documentar transformaciones | Test H5 (recompra) | Storytelling visual integrado | Reporte de calidad de enriquecimiento |
| 12 | Comparación de modelos (F1, AUC) | Preparar dataset final para entrega | Redactar conclusiones estadísticas | Pulir dashboard final | Apéndice de enriquecimiento |
| 13 | Integración de resultados | Subir datasets procesados a GitHub | Revisión de hipótesis | Preparar slides de presentación | Revisión final del .md |
| 14 | Presentación final | Soporte técnico durante expo | Exposición de resultados estadísticos | Exposición de visualizaciones | Exposición de enriquecimiento |

## 8. Apéndice de uso de IA

Durante el desarrollo de la PC1 se utilizó inteligencia artificial generativa como herramienta de apoyo para organizar el documento, mejorar la redacción y estructurar algunas secciones del entregable.

La IA no fue utilizada para reemplazar el análisis de datos ni para generar resultados inexistentes. Su uso se limitó a apoyar la claridad del informe, la formulación de hipótesis y la organización del contenido en formato Markdown.

### 8.1 Herramienta utilizada

| Herramienta | Uso principal |
|---|---|
| ChatGPT | Apoyo en redacción, estructura del entregable, hipótesis, enriquecimiento y organización del README |

### 8.2 Prompts utilizados

| Actividad | Prompt utilizado | Validación realizada |
|---|---|---|
| Estructura del entregable | “Genera un documento para el entregable PC1 que será subido al repositorio de GitHub.” | Se comparó la estructura con los requisitos de la rúbrica |
| Explicación de la Etapa 1 | “Explica qué se hace en la Etapa 1 – Definición y Exploración Inicial.” | Se verificó que el contenido se mantenga dentro de la PC1 |
| Revisión del notebook | “Repetir el proceso teniendo en cuenta los datos del notebook.” | Se revisó que las variables mencionadas existan en el notebook |
| Enriquecimiento | “Desarrollar el enriquecimiento teniendo en cuenta calendario, festivos y tipo de cambio.” | Se validó que las fuentes propuestas sean coherentes con el caso Olist |
| Plan de trabajo | “Modificar el plan de trabajo dejando claro que Etapa 2 es un entregable futuro.” | Se ajustó el texto para no presentar resultados futuros como ya realizados |
| Apéndice de IA | “Redactar un apéndice de uso de IA para un informe académico.” | Se revisó manualmente para declarar el uso real y no exagerar el alcance |

### 8.3 Validación del contenido generado

El contenido generado con apoyo de IA fue revisado manualmente por el equipo y contrastado con el notebook inicial.

Se validó que las secciones hicieran referencia a variables realmente utilizadas en el análisis, como:

| Variable | Uso en el proyecto |
|---|---|
| `price` | Análisis de precios |
| `freight_value` | Análisis del costo de envío |
| `review_score` | Medición de satisfacción del cliente |
| `delivery_time` | Tiempo real de entrega |
| `product_category_name` | Análisis por categoría |

También se verificó que las hipótesis estuvieran relacionadas con las visualizaciones del EDA inicial, como distribución de calificaciones, distribución de precios, tiempo de entrega, relación entre tiempo de entrega y satisfacción, y satisfacción promedio por categoría.

### 8.4 Uso responsable de IA

La IA fue utilizada como apoyo académico y de redacción. Las decisiones principales del proyecto, la selección del dataset, la carga de datos, la integración de tablas, la creación de variables y las visualizaciones fueron realizadas por el equipo.

El equipo revisó que el documento no incluyera conclusiones no respaldadas por el notebook ni resultados que pertenezcan a etapas posteriores.

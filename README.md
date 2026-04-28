## PC1 – Propuesta y EDA Inicial
## Caso: ¿Qué hace que un producto triunfe en e-commerce?

# Proyecto MarketPulse Analytics - Análisis de Satisfacción del Cliente (Olist)

## 1. Portada del equipo

| Rol | Integrante |
| :--- | :--- |
| EDA | Camila Torres |
| --- | Integrante2 |
| Hipótesis | Letizia Torres |
| IA- Enriquecimiento de Datos| Marcelo Villafuerte |
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

### Viz 1 – Distribución de precios
El análisis de la variable precio evidencia una distribución fuertemente sesgada hacia la derecha, donde el precio promedio (≈120 BRL) es significativamente mayor que la mediana (≈75 BRL). Esto confirma la presencia de valores extremos elevados que influyen en el promedio. Asimismo, el 75% de los productos se encuentra por debajo de aproximadamente 135 BRL, lo que indica que la mayor parte del mercado se concentra en rangos de precios bajos y medios. La existencia de productos con precios máximos cercanos a 6735 BRL evidencia la presencia de outliers que no representan el comportamiento típico, pero sí reflejan la existencia de un segmento premium minoritario. En términos de negocio, esto sugiere un mercado altamente competitivo donde el precio no es el principal diferenciador.

### Viz 2 – Alta satisfacción general
La distribución de calificaciones muestra una clara concentración en ratings de 4 y 5, lo que indica un nivel de satisfacción general elevado entre los clientes. Esto sugiere que, en términos globales, la experiencia de compra cumple con las expectativas del usuario. Sin embargo, esta alta concentración también puede ocultar variaciones importantes en la experiencia, por lo que resulta necesario analizar factores específicos que expliquen las calificaciones más bajas.

### Viz 3 – Tiempo de entrega
El tiempo de entrega presenta una distribución concentrada en rangos bajos, lo que indica que la mayoría de pedidos se entrega en pocos días. No obstante, se identifican valores extremos que alcanzan hasta 208 días, lo que evidencia la existencia de casos atípicos dentro del sistema logístico. Aunque estos outliers no representan el comportamiento general, pueden afectar significativamente la percepción del servicio en situaciones específicas.

### Viz 4 – Impacto del tiempo en la satisfacción
El análisis de la relación entre tiempo de entrega y calificación del cliente muestra una tendencia donde mayores tiempos de entrega se asocian con menores niveles de satisfacción. Aunque la relación no es perfectamente lineal, se observa que los pedidos con tiempos más prolongados tienden a concentrarse en ratings más bajos. Esto sugiere que la rapidez en la entrega es un factor crítico en la experiencia del cliente, y que mejoras en la logística podrían generar impactos directos en la satisfacción.

### Viz 5 – Categorías más vendidas vs satisfacción
Las ventas se concentran en un conjunto reducido de categorías, lo que indica una demanda focalizada en ciertos segmentos del mercado. Sin embargo, al analizar el rating promedio por categoría, se observa que las diferencias en la satisfacción entre ellas son relativamente pequeñas. Esto sugiere que el tipo de producto no es el principal determinante de la satisfacción del cliente, y que factores transversales, como la calidad del servicio o la eficiencia en la entrega, tienen un mayor peso en la percepción final del usuario.

## 5. Hipótesis de negocio

**H1 – Precio y satisfacción:**
¿Los productos en rangos de precio medio (ni los más baratos ni los más costosos) obtienen calificaciones más altas que los extremos de la distribución de precios?

**H2 – Umbral crítico de tiempo de entrega:**
¿Existe un número específico de días de entrega a partir del cual la calificación del cliente cae de forma significativa, o la relación es gradual y continua?

**H3 – Casos extremos logísticos:**
¿Los pedidos con tiempos de entrega atípicamente altos (outliers) concentran de forma desproporcionada las calificaciones de 1 y 2 estrellas, y comparten características comunes como región geográfica o categoría de producto?

**H4 – Categoría como moderadora:**
¿La categoría del producto modera el efecto del tiempo de entrega sobre la satisfacción? Es decir, ¿en algunas categorías los clientes toleran más la demora que en otras?

**H5 – Perfil del cliente insatisfecho:**
¿Es posible identificar un perfil de compra (combinación de precio, categoría y tiempo de entrega) que prediga con alta probabilidad una calificación baja (1–2 estrellas), a pesar del contexto general de alta satisfacción observado en el EDA?

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

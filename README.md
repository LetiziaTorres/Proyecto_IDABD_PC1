## PC1 – Propuesta y EDA Inicial
## Caso: ¿Qué hace que un producto triunfe en e-commerce?

# Proyecto MarketPulse Analytics - Análisis de Satisfacción del Cliente (Olist)

## 1. Portada del equipo

| Rol | Integrante |
| :--- | :--- |
| Líder de proyecto / EDA | Integrante1 |
| Ingeniería de datos / Limpieza | Integrante2 |
| Análisis estadístico / Hipótesis | Integrante3 |
| Visualización / Storytelling | Integrante4 |
| Enriquecimiento de datos / Documentación | Integrante5 |

## 2. Descripción del problema de negocio

**Empresa ficticia:** MarketPulse Analytics, consultora de datos que asesora a vendedores del marketplace Olist en Brasil.

**Contexto:**
El comercio electrónico en Brasil creció más del 26% entre 2020 y 2022, consolidando plataformas como Olist como principales canales de venta para miles de pequeñas y medianas empresas. Sin embargo, la alta tasa de abandono de vendedores y la volatilidad en las calificaciones de productos revelan que muchos comerciantes no comprenden qué palancas realmente determinan la satisfacción del cliente.

**Problema:**
¿Qué combinación de factores —precio, categoría de producto, tiempo de entrega, calificación del vendedor y método de pago— predice mejor la satisfacción del cliente (review_score) y la probabilidad de recompra?

**Relevancia:**
Identificar estos factores permite a los vendedores priorizar inversiones: ¿reducir precio o mejorar logística? ¿Más fotos del producto o mejor descripción? Esta información es accionable y puede traducirse directamente en aumento de ingresos y retención de clientes.
*(Máx. 1 página)*

## 3. Descripción de los datos

**Fuente principal — Brazilian E-Commerce (Olist)**

- **Origen:** Kaggle – olistbr/brazilian-ecommerce
- **Período:** Septiembre 2016 – Octubre 2018
- **Archivos y registros:**

| Dataset | Registros | Descripción |
| :--- | :--- | :--- |
| `olist_orders_dataset.csv` | 99,441 | Órdenes con timestamps y estado |
| `olist_order_items_dataset.csv` | 112,650 | Ítems por orden (precio, flete) |
| `olist_order_reviews_dataset.csv` | 104,719 | Reseñas (score 1–5, comentarios) |
| `olist_customers_dataset.csv` | 99,441 | Clientes (ciudad, estado) |
| `olist_products_dataset.csv` | 32,951 | Productos (categoría, dimensiones, fotos) |
| `olist_sellers_dataset.csv` | 3,095 | Vendedores (localización) |
| `olist_order_payments_dataset.csv` | 103,886 | Métodos y montos de pago |
| `olist_geolocation_dataset.csv` | ~1M | Coordenadas por código postal |
| `product_category_name_translation.csv` | 71 | Traducción PT→EN de categorías |

**Variables clave:**
`review_score`, `price`, `freight_value`, `order_status`, `order_delivered_customer_date`, `order_estimated_delivery_date`, `product_category_name`, `payment_type`, `payment_installments`

**Fuente de enriquecimiento**

- **Calendario de festivos Brasil:** API Nager.Date (festivos nacionales 2016–2018) → detectar picos de compra estacionales
- **Tipo de cambio BRL/USD histórico:** ExchangeRate-API o datos de Banco Central do Brasil → contextualizar el valor real de las transacciones
- **Tipo de cambio BRL/PEN:** BCRP para conversión a contexto peruano

## 4. EDA Inicial

> Las visualizaciones se generan en el notebook adjunto (`EDA_Grupo3.ipynb`). A continuación se describe cada una con sus hallazgos clave.

**Viz 1 – Distribución del `review_score`**
- Scores: 1→11,424 | 2→3,151 | 3→8,179 | 4→19,142 | 5→57,328
- **Hallazgo:** La distribución está fuertemente sesgada hacia puntuaciones positivas (5 estrellas = 54.8% de las reseñas). Esto implica un desbalance de clases que deberá tratarse en modelos predictivos (SMOTE, class_weight). Los scores 1 y 3 son los más informativos para identificar insatisfacción.

**Viz 2 – Distribución de precio y flete (boxplot + histograma)**
- Precio: media=120.65 BRL | mediana=74.99 BRL | max=6,735 BRL
- Flete:  media=19.99 BRL  | mediana=16.26 BRL | max=409.68 BRL
- **Hallazgo:** Ambas variables tienen colas muy largas (outliers extremos). El 75% de los productos cuesta menos de 135 BRL. Se requerirá transformación logarítmica antes del modelado. El flete representa en promedio el 16.5% del precio del producto.

**Viz 3 – Nulos por columna (heatmap)**
- `review_comment_title`:   87,656 nulos (83.7%)
- `review_comment_message`: 58,247 nulos (55.6%)
- `order_delivered_customer_date`: 2,965 nulos (3.0%)
- `product_category_name`:    610 nulos (1.9%)
- **Hallazgo:** La ausencia de comentarios de texto es estructural (muchos clientes solo ponen score sin texto). Las fechas de entrega nulas corresponden a órdenes canceladas o en tránsito. **Estrategia:** imputar categorías por mediana de categorías similares; excluir órdenes sin fecha de entrega del análisis de tiempo.

**Viz 4 – `review_score` promedio por categoría de producto (top 15 y bottom 5)**
- **Hallazgo:** Categorías como `cds_dvds_musicals` y `fashion_childrens_clothes` tienen scores promedio >4.5, mientras que `security_and_services` y `office_furniture` promedian <3.5. Esto sugiere que la categoría *per se* es un predictor relevante de satisfacción.

**Viz 5 – Tiempo de entrega vs. `review_score` (días reales vs. estimados)**
- **Hallazgo:** Se observa correlación negativa entre días de retraso y review_score. Órdenes entregadas antes de la fecha estimada tienen score promedio de 4.3 vs. 2.9 para las que llegan tarde. **El tiempo de entrega es probablemente el predictor más fuerte de insatisfacción**, por encima del precio.

**Viz 6 – Método de pago vs. ticket promedio**
- `credit_card`: 73.9% de transacciones | ticket_medio=163 BRL
- `boleto`:       19.1%                  | ticket_medio=108 BRL
- `voucher`:       5.5%                  | ticket_medio= 64 BRL
- `debit_card`:    1.5%                  | ticket_medio=144 BRL
- **Hallazgo:** El pago con tarjeta de crédito está asociado a tickets más altos. Los compradores de boleto (pago en efectivo, típico en Brasil) tienden a comprar productos más baratos. Esto puede relacionarse con el perfil socioeconómico y tolerancia al riesgo.

## 5. Hipótesis de negocio

Las siguientes hipótesis serán investigadas en la Etapa 2 mediante modelos predictivos y análisis estadísticos:

- **H1 – Tiempo de entrega como predictor dominante:** ¿Es el tiempo de entrega (real vs. estimado) el factor con mayor peso predictivo sobre el `review_score` del cliente, por encima del precio o la categoría del producto?

- **H2 – Efecto del precio relativo en la satisfacción:** ¿Los productos con una relación flete/precio alta (>30%) tienen significativamente menor `review_score` que los productos con flete bajo, independientemente de la categoría?

- **H3 – Concentración de vendedores y calidad del servicio:** ¿Los vendedores con mayor volumen de ventas (top 10%) ofrecen tiempos de entrega más confiables y obtienen calificaciones más altas que los vendedores pequeños?

- **H4 – Estacionalidad y satisfacción:** ¿Las compras realizadas durante periodos de alta demanda (Navidad, Black Friday, Día de los Niños en Brasil) presentan peores tiempos de entrega y menor `review_score` debido a la saturación logística?

- **H5 – Recompra y calidad percibida:** ¿Los clientes que dieron 5 estrellas en su primera compra tienen mayor probabilidad de realizar una segunda compra en los siguientes 6 meses?

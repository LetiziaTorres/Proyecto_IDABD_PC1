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

### Viz 1 – Distribución del review_score

**Scores:**  
1 → 11,424 | 2 → 3,151 | 3 → 8,179 | 4 → 19,142 | 5 → 57,328

**Hallazgo:** La distribución está fuertemente sesgada hacia puntuaciones positivas (5 estrellas = 54.8% de las reseñas). Esto implica un desbalance de clases que deberá tratarse en modelos predictivos (SMOTE, class_weight). Los scores 1 y 3 son los más informativos para identificar insatisfacción.

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

## 6. Plan de trabajo – Etapa 2 (Semanas 7–14)

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

## 7. Apéndice de uso de IA

El equipo utilizó IA generativa (Claude de Anthropic) exclusivamente como apoyo en la **estructura y formato del entregable**:

| Actividad | Prompt utilizado (resumen) | Validación realizada |
|-----------|---------------------------|----------------------|
| Estructura y formato del documento | "Genera un documento .md con las secciones requeridas para PC1, basado en el dataset Olist, grupo de 5 personas con roles" | Revisión manual de cada sección por el equipo; verificación de coherencia con la rúbrica y ajuste de contenido |

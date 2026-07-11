# Ecosistema de Big Data para el Análisis de Sentimiento en Opiniones de Consumidores

Proyecto del curso **Big Data y Analítica de Datos** — Universidad Privada Antenor Orrego (UPAO).

Sistema de Big Data que clasifica automáticamente el sentimiento de reseñas de productos alimenticios y detecta las causas de insatisfacción del consumidor, para apoyar decisiones que reduzcan devoluciones y desperdicio.

## Descripción del proyecto

El proyecto construye un pipeline completo de datos sobre el dataset **Amazon Fine Food Reviews** (568,454 reseñas), aplicando procesamiento distribuido, machine learning y análisis de texto. El objetivo es detectar productos con problemas de calidad a partir de la voz del consumidor, e identificar los motivos concretos de las quejas.

## Arquitectura

Se implementó una arquitectura **Data Lakehouse** con patrón Medallion (Bronce, Plata, Oro) sobre Databricks.

- **Capa Bronce:** datos crudos ingestados sin transformar.
- **Capa Plata:** datos limpios, anonimizados y con la etiqueta de sentimiento.
- **Capa Oro:** tablas agregadas listas para consumo (por producto, por año, distribución, quejas).

## Tecnologías utilizadas

- **Apache Spark (PySpark)** — procesamiento distribuido de los 568k registros.
- **Spark MLlib** — modelo de clasificación (Regresión Logística + TF-IDF).
- **Spark Structured Streaming** — clasificación de reseñas en tiempo real.
- **Databricks** — plataforma cloud donde se ejecuta todo el ecosistema.
- **Delta Lake** — almacenamiento de las capas del Lakehouse.
- **Dashboards de Databricks** — visualización ejecutiva.

## Modelo de Machine Learning

- **Tarea:** clasificación multiclase de sentimiento (Positivo / Neutro / Negativo), usando el Score de 1-5 como proxy.
- **Pipeline NLP:** Tokenizer → StopWordsRemover → HashingTF → IDF → Regresión Logística.
- **Tratamiento del desbalanceo:** ponderación de clases (class weighting), ya que el 78% de las reseñas son positivas.
- **Comparación de modelos:** se evaluó Regresión Logística vs Random Forest.

### Resultados

| Métrica | Regresión Logística (balanceada) | Random Forest |
|---|---|---|
| Accuracy | 0.7685 | 0.7817 |
| F1-Score | **0.7969** | 0.6872 |

Se eligió la **Regresión Logística balanceada** como modelo final. Aunque Random Forest tuvo mayor accuracy, su F1 fue mucho menor porque ignoraba las clases minoritarias (predecía "Positivo" a casi todo). Para el objetivo del proyecto —detectar reseñas negativas— priorizar el F1 es lo correcto.

## Hallazgos principales

- **Distribución:** 78% de reseñas positivas, 14.4% negativas, 7.5% neutras.
- **Insight de negocio:** el análisis de bigramas en reseñas negativas revela que las quejas se concentran en el sabor ("taste like", "tastes like"), la percepción de valor ("waste money") y la pérdida de cliente ("not buy"). El término "disappointed" aparece más de 7,000 veces.
- Estos hallazgos permiten a las empresas corregir problemas de calidad antes de que generen devoluciones y desperdicio de alimentos.

## Seguridad y normas

- **Anonimización (GDPR / Ley N.° 29733):** se eliminaron los campos de datos personales (UserId, ProfileName) en la Capa Plata.
- **Calidad de datos (ISO/IEC 25012):** limpieza de nulos, estandarización y validación de tipos.
- **Seguridad (ISO/IEC 27001):** control de acceso mediante Unity Catalog de Databricks.
## Cómo ejecutar

1. Subir el dataset `Reviews.csv` a un Volume de Databricks.
2. Ejecutar `notebooks/01_pipeline_completo.ipynb` en orden (crea las capas Bronce, Plata, Oro y entrena el modelo).
3. Ejecutar `notebooks/02_streaming_tiempo_real.ipynb` para la demo de clasificación en vivo.
4. Abrir el dashboard en Databricks para la visualización.

## Dataset

[Amazon Fine Food Reviews - Kaggle](https://www.kaggle.com/datasets/snap/amazon-fine-food-reviews)

*Proyecto académico — 2026*

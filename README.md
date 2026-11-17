# 🧬 Detección de Interacciones Fármaco-Fármaco (DDI) con NLP

**Proyecto de Práctica Profesional — DATOMICS · UNAB**  
*Equipo de 3 integrantes · Director: Dr. Eduardo Martínez*

Este repositorio contiene el pipeline completo, el dataset analítico y los modelos de Machine Learning desarrollados para detectar y clasificar automáticamente interacciones fármaco-fármaco (DDI) a partir de la base de datos **DrugBank** y una base clínica local.

## 📋 Contexto y Metodología

El objetivo del proyecto es entrenar un modelo de NLP que, dada una oración clínica y un par de fármacos, determine si existe una interacción real y de qué tipo. 

**Un hito metodológico clave:** Se descartó la *Closed-World Assumption* (inventar interacciones negativas con texto de relleno). En su lugar, se implementó una **extracción real y trazable** usando patrones de texto (negación/afirmación) directamente sobre las fichas clínicas, cruzadas contra un vocabulario de 4.333 fármacos. Esto garantiza que el **100% de las 2.701 filas** del dataset final estén respaldadas por una oración real de la literatura médica.

## 🗂️ Estructura del Repositorio

### 1. Preprocesamiento y Extracción
* **`01_limpiardatos.py`**: Limpieza del archivo original `drugbank.csv`, corrección de filas rotas y generación de un diccionario médico.
* **`02_cruce_y_extraccion.py`**: Cruce con la base clínica local. Extracción de pares de fármacos (E1/E2) y sus oraciones mediante expresiones regulares. Incluye la corrección de bugs detectados (desduplicación por mayúsculas/minúsculas y reasignación de destinos en listas).

### 2. Modelos Clásicos (Línea Base)
* **`03_modelo_svm.py`**: SVM lineal con TF-IDF. Entrenado descartando las filas ambiguas (Clases claras).
* **`03.1_modelo_svm_8clases.py`**: SVM lineal con TF-IDF. Entrenado subdividiendo las 333 filas de "duda" en 4 subcategorías (origen, destino, no atómica, vaga), dando un total de 8 clases.

### 3. Modelos de Deep Learning (Transformers)
* **`04_modelo_biobert.py`**: Fine-tuning de **BioBERT v1.1** con 3 épocas fijas.
* **`05_modelo_biobert_v2.py`**: Mejora del modelo anterior incorporando **Early Stopping** (patience=2) y **marcado de entidades E1/E2** en el texto.
* **`06_modelo_pubmedbert.py`**: Fine-tuning de **PubMedBERT** (entrenado desde cero con texto biomédico), aplicando Early Stopping y el marcado de entidades.

## 🏆 Resultados y Comparativa de Modelos

Tras una exhaustiva evaluación con un split estratificado (70/15/15 agrupado por fármaco para evitar fuga de datos), estos son los resultados de los modelos de Deep Learning:

| Modelo | Arquitectura | Épocas | F1 Macro | F1 Weighted | Accuracy |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Modelo 3** | BioBERT v1.1 | 3 (Fijas) | 0.8464 | 0.8762 | 0.88 |
| **Modelo 3.1** | BioBERT v1.1 + Early Stopping | 7 (Early Stop) | 0.8685 | 0.8945 | 0.90 |
| **Modelo 4 (Ganador)** | **PubMedBERT** + Early Stopping | 7 (Early Stop) | **0.8759** | **0.9000** | **0.90** |

**Factores clave del éxito:**
1. **Early Stopping (+2.21% F1 Macro):** Permitir que BioBERT entrenara hasta la época 7 capturó relaciones complejas que en 3 épocas no lograba.
2. **Dominio Puro (+0.74% F1 Macro adicional):** PubMedBERT superó a BioBERT al usar un vocabulario especializado creado desde cero con literatura médica, reconociendo entidades químicas como bloques unificados y evitando la fragmentación de términos.

### Análisis de Errores
* **Confusión `mechanism` vs `effect`:** El solapamiento lingüístico en la literatura biomédica hace que modificaciones farmacocinéticas se confundan con manifestaciones clínicas.
* **Heterogeneidad `duda_vaga`:** Al fusionar 4 subtipos de dudas lingüísticamente dispares, el modelo no logra generalizar un único patrón sintáctico.

## 🛠️ Tecnologías Utilizadas

* **Datos:** `pandas`, `numpy`, `openpyxl`, `re` (Regex)
* **Machine Learning Clásico:** `scikit-learn` (TF-IDF, LinearSVC, métricas)
* **Deep Learning:** `transformers` (Hugging Face), `torch` (PyTorch), `datasets`
* **Visualización:** `matplotlib`, `seaborn`

## 🚀 ¿Cómo ejecutar este proyecto?

Este proyecto fue desarrollado y ejecutado en **Google Colab**. Para replicarlo:
1. Clona este repositorio.
2. Sube tus archivos de datos (`drugbank.csv` y `ddiData.xlsx`) a tu Google Drive.
3. Modifica las rutas absolutas dentro de los scripts (`/content/drive/MyDrive/...`) para que apunten a tus archivos.
4. Para los modelos de Transformers, asegúrate de tener activada la GPU (`Entorno de ejecución > Cambiar tipo de entorno > GPU T4`).

# 🧬 Pipeline de Procesamiento y Clasificación de Interacciones Farmacológicas (NLP)

Este repositorio contiene un pipeline completo de Ciencia de Datos y Procesamiento de Lenguaje Natural (NLP) diseñado para procesar, limpiar, cruzar y clasificar automáticamente interacciones medicamentosas. El proyecto utiliza como fuente principal la base de datos **DrugBank** y una base clínica local, implementando desde modelos clásicos de Machine Learning hasta arquitecturas avanzadas de Deep Learning (Transformers).

## 📋 Descripción del Proyecto

El objetivo principal es automatizar la clasificación de textos clínicos que describen interacciones entre fármacos. Para ello, el pipeline transforma archivos CSV masivos y textos no estructurados en datasets limpios y modelos predictivos capaces de categorizar las interacciones en: `effect`, `advise`, `mechanism`, `no_interaction` y `duda_vaga`. 

## 🗂️ Estructura del Pipeline (Archivos)

El proyecto está dividido en fases secuenciales:

### Fase 1 y 2: Preprocesamiento y Extracción
* **`01_limpiardatos.py`**: Repara el archivo original `drugbank.csv` (con filas rotas), limpia etiquetas HTML y genera un diccionario médico en `.txt`.
* **`02_cruce_y_extraccion.py`**: Cruza los datos de DrugBank con la base local (`ddiData.xlsx`). Utiliza expresiones regulares para extraer los pares de fármacos (E1, E2) y sus oraciones. Genera hojas de Excel con formato visual y aplica reglas heurísticas para un primer etiquetado.

### Fase 3: Modelos Clásicos (Machine Learning)
* **`03_modelo_svm.py`**: Modelo baseline. Elimina las dudas, vectoriza con **TF-IDF** y entrena un **LinearSVC** (4/5 clases).
* **`03.1_modelo_svm_8clases.py`**: Experimento paralelo. Conserva las subcategorías de dudas (`duda_origen`, `duda_destino`, etc.) para evaluar si el modelo puede distinguir entre 8 clases. Incluye visualización gráfica de la matriz de confusión.

### Fase 4: Modelos Avanzados (Transformers / Deep Learning)
* **`04_modelo_biobert.py`**: Ajuste fino (fine-tuning) del modelo **BioBERT** (`dmis-lab/biobert-v1.1`) para clasificación en 5 clases.
* **`05_modelo_biobert_v2.py`**: Versión mejorada de BioBERT. Implementa **Early Stopping** (para evitar el sobreajuste) y **marcado de entidades E1/E2** en el texto (técnica avanzada de NLP para enfocar la atención del modelo).
* **`06_modelo_pubmedbert.py`**: Implementación de **PubMedBERT** (`microsoft/BiomedNLP-PubMedBERT...`), un modelo entrenado desde cero con texto completo de artículos biomédicos, aplicando las mismas mejoras de la versión 2.

## 📊 Comparativa de Modelos

A lo largo del proyecto se experimentó con diferentes arquitecturas y enfoques de etiquetado:

| Modelo | Algoritmo | Características Clave | Clases | Enfoque |
| :--- | :--- | :--- | :--- | :--- |
| **Modelo 1** | LinearSVC | TF-IDF, `class_weight='balanced'` | 5 | Clásico, sin dudas. |
| **Modelo 2** | LinearSVC | TF-IDF, visualización de matriz | 8 | Clásico, con subtipos de dudas. |
| **Modelo 3** | BioBERT v1.1 | Hugging Face Trainer | 5 | Deep Learning, dudas fusionadas. |
| **Modelo 3.1** | BioBERT v1.1 | **Early Stopping + Marcado E1/E2** | 5 | Deep Learning mejorado. |
| **Modelo 4** | PubMedBERT | Early Stopping + Marcado E1/E2 | 5 | SOTA en texto biomédico. |

## 🛠️ Tecnologías y Librerías Utilizadas

* **Manipulación de Datos:** `pandas`, `numpy`, `openpyxl`
* **Machine Learning Clásico:** `scikit-learn` (TF-IDF, LinearSVC, métricas)
* **Deep Learning / NLP:** `transformers` (Hugging Face), `datasets`, `torch` (PyTorch)
* **Visualización:** `matplotlib`, `seaborn`
* **Utilidades:** `re` (Regex), `joblib`, `json`, `os`

## 🚀 ¿Cómo ejecutar este proyecto?

Este proyecto fue desarrollado y ejecutado en **Google Colab**. Para replicarlo:

1. Clona este repositorio.
2. Sube tus archivos de datos (`drugbank.csv` y `ddiData.xlsx`) a tu Google Drive.
3. Modifica las rutas absolutas dentro de los scripts (ej. `/content/drive/MyDrive/...`) para que apunten a la ubicación de tus archivos.
4. Ejecuta los scripts en orden secuencial. Para los modelos de Transformers (BioBERT y PubMedBERT), asegúrate de tener activada la GPU en Colab (`Entorno de ejecución > Cambiar tipo de entorno > GPU T4`).

---
*Proyecto desarrollado para el análisis avanzado y la automatización en la clasificación de interacciones farmacológicas.*

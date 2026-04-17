# 🌱 Predicción de Necesidad de Riego

Proyecto de clasificación multiclase para la competencia de Kaggle **Irrigation Need Prediction**. El objetivo es predecir la necesidad de riego (`High`, `Medium`, `Low`) a partir de variables agrícolas y ambientales.

---

## 📑 Tabla de Contenidos

- [Descripción General](#descripción-general)
- [Estructura del Proyecto](#estructura-del-proyecto)
- [Datos](#datos)
- [Metodología](#metodología)
  - [Análisis Exploratorio (EDA)](#análisis-exploratorio-eda)
  - [Preprocesamiento](#preprocesamiento)
  - [Modelado](#modelado)
  - [Optimización de Hiperparámetros](#optimización-de-hiperparámetros)
- [Resultados](#resultados)
- [Tech Stack](#tech-stack)
- [Configuración y Uso](#configuración-y-uso)

---

## Descripción General

Se trata de un problema de **clasificación multiclase con 3 clases** (`High`, `Medium`, `Low`) que representan el nivel de necesidad de riego. El dataset presenta un **desbalance de clases** significativo, con aproximadamente el **58% de las muestras pertenecientes a la clase `High`**.

Se evaluaron múltiples modelos (LinearSVC, LogisticRegression, XGBClassifier) y se utilizó `balanced_accuracy` como métrica principal de evaluación durante la búsqueda de hiperparámetros, junto con F1-score macro/weighted y matrices de confusión para el reporte de resultados.

---

## Estructura del Proyecto

```
irrigation need/
├── eda.ipynb              # Análisis exploratorio de datos
├── preprocessing.ipynb    # Preprocesamiento, entrenamiento y generación de submission
├── AGENTS.md              # Instrucciones para agentes de IA
├── formater.md            # Agente personalizado: Notebook Markdown Formatter
├── duckdb/                # Directorio reservado para archivos DuckDB
├── .gitignore             # Ignora archivos CSV, .venv, checkpoints y cache
├── .github/agents/        # Directorio para agentes de GitHub
├── train.csv              # Datos de entrenamiento (git-ignored)
├── test.csv               # Datos de test (git-ignored)
└── submission.csv         # Predicciones para Kaggle (git-ignored)
```

---

## Datos

| Archivo     | Descripción                                      |
|-------------|--------------------------------------------------|
| `train.csv` | Dataset de entrenamiento con la columna objetivo |
| `test.csv`  | Dataset de test para generar predicciones        |

- **Columna objetivo:** `Irrigation_Need` (3 clases: `High`, `Low`, `Medium`)
- **Columna índice:** `id`
- **Variables:** Incluyen características numéricas y categóricas de baja cardinalidad (< 10 valores únicos)


> Los archivos CSV no se incluyen en el repositorio (están en `.gitignore`). Deben descargarse directamente desde la competencia de Kaggle.

---

## Metodología

### Análisis Exploratorio (EDA)

Realizado en `eda.ipynb`. Hallazgos principales:

1. **Estadísticas descriptivas:** La variable `Rainfall_mm` destaca por tener la media alejada de la mediana
2. **Matriz de correlación:** No se encontraron correlaciones fuertes entre las variables numéricas, lo que sugiere que cada variable aporta información única
3. **Análisis de outliers:** Se analizaron mediante el método IQR y boxplots — no se identificaron outliers significativos, incluyendo `Rainfall_mm`
4. **Desbalance de clases:** La clase `Low` representa ~58% de las muestras del target, un desbalance considerable

### Preprocesamiento

Realizado en `preprocessing.ipynb`. Pipeline de transformación con `sklearn`:

- **División de datos:** 80% entrenamiento / 20% validación (`train_test_split`, `random_state=0`)
- **Variables numéricas:** Estandarización con `StandardScaler`
- **Variables categóricas:** Codificación con `OneHotEncoder` (`handle_unknown='ignore'`)
- **Estructura:** `ColumnTransformer` dentro de un `Pipeline` de scikit-learn

### Modelado

Se compararon dos estrategias de clasificación multiclase:

| Modelo                          | Estrategia |
|---------------------------------|------------|
| `LinearSVC` + `OneVsRestClassifier` | One-vs-Rest |
| `LogisticRegression` (multinomial)  | Softmax     |

El modelo con mejor rendimiento en la comparación inicial fue **LogisticRegression (Softmax)**, que se seleccionó para la optimización de hiperparámetros. Adicionalmente, se entrenó un **XGBClassifier** con aceleración GPU.

### Optimización de Hiperparámetros

#### LogisticRegression (Softmax)
- **Método:** `GridSearchCV`
- **Validación cruzada:** 10 folds
- **Métrica de scoring:** `balanced_accuracy`
- **Parámetros explorados:**
  - `C`: 20 valores en escala logarítmica (10⁻³ a 10³)
  - `max_iter`: [500, 1000, 2000, 5000, 10000]

#### XGBClassifier
- **Método:** `RandomizedSearchCV` (50 iteraciones de 500 combinaciones posibles)
- **Validación cruzada:** 3 folds
- **Métrica de scoring:** `balanced_accuracy`
- **Configuración GPU:** `device='cuda'`
- **Parámetros explorados:**
  - `learning_rate`: 20 valores en escala logarítmica (10⁻³ a 10⁰)
  - `max_depth`: [3, 5, 7, 9, 11]
  - `n_estimators`: [100, 200, 300, 400, 500]

---

## Resultados

El modelo final seleccionado fue **XGBClassifier** con los hiperparámetros optimizados. La matriz de confusión (en porcentaje) muestra un rendimiento excelente:

| Clase Real \ Predicha | High  | Low   | Medium |
|------------------------|-------|-------|--------|
| **High**               | 92.2% | 0.0%  | 7.8%   |
| **Low**                | 0.0%  | 99.5% | 0.5%   |
| **Medium**             | 0.3%  | 2.3%  | 97.4%  |

- La clase `Low` es la mejor predicha con un **99.5%** de acierto
- La clase `Medium` alcanza un **97.4%** de acierto
- La clase `High` tiene un **92.2%** de acierto, con una ligera confusión hacia `Medium`

Las predicciones finales se generaron sobre el dataset de test y se exportaron a `submission.csv`.

---

## Tech Stack

| Librería         | Uso                                             |
|------------------|--------------------------------------------------|
| `pandas`         | Manipulación y análisis de datos                 |
| `scikit-learn`   | Pipelines, preprocesamiento, modelos, evaluación |
| `xgboost`        | Modelo XGBClassifier con soporte GPU             |
| `seaborn`        | Visualización estadística (heatmaps, boxplots)   |
| `matplotlib`     | Gráficos y visualización                         |
| `numpy`          | Operaciones numéricas                            |

---

## Configuración y Uso

### Requisitos previos

- Python 3.x
- GPU compatible con CUDA (para XGBoost con `device='cuda'`)

### Instalación de dependencias

```bash
pip install pandas scikit-learn xgboost seaborn matplotlib numpy
```

### Ejecución

1. Descargar los archivos `train.csv` y `test.csv` desde la competencia de Kaggle
2. Colocarlos en el directorio raíz del proyecto
3. Ejecutar `eda.ipynb` para el análisis exploratorio
4. Ejecutar `preprocessing.ipynb` para el entrenamiento y generación de `submission.csv`

---

## Reconocimientos

- Competencia de Kaggle: **Irrigation Need Prediction**
- Referencia para tratamiento de desbalance de clases: [Imbalanced Datasets — Google ML Crash Course](https://developers.google.com/machine-learning/crash-course/overfitting/imbalanced-datasets?hl=es-419)

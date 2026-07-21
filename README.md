# Análisis y clasificación de lípidos: Mouse vs NMR

Este repositorio contiene un análisis completo del comportamiento de lípidos en dos grupos de organismos: Mouse y NMR. El objetivo es desarrollar modelos de aprendizaje automático capaces de diferenciar entre estos dos grupos basándose en características extraídas de datos moleculares tridimensionales.

## Descripción del proyecto

El proyecto se enfoca en dos tipos de lípidos:

- **CHOL**: Colesterol. Caracterizado por un `head` (cabeza) y un `body` (cuerpo).
- **DPSM**: Esfingomielina. Caracterizado por `heads` (cabezas) y `tails` (colas).

Para cada tipo de lípido, se entrena un modelo independiente con el objetivo de clasificar correctamente a qué grupo pertenece cada molécula.

## Estructura del proyecto

El análisis se divide en cuatro etapas principales, cada una implementada en un notebook de Jupyter:

### 1. Análisis de datos exploratorio (`01_analysis.ipynb`)

En esta etapa se realiza una carga inicial de los datos y se obtienen estadísticas descriptivas fundamentales:

- Carga de arrays `.npy` desde los directorios `Mouse` y `NMR`
- Verificación de integridad de datos (valores nulos, infinitos, tipos de dato y formas)
- Estadísticas descriptivas de los datos tridimensionales
- Análisis de la distribución de los datos por grupo

**Entrada**: Archivos `.npy` con datos moleculares crudos.

**Salida**: Características estadísticas descriptivas.

### 2. Ingeniería de características (`02_feat_engineering.ipynb`)

En esta etapa se transforman los datos tridimensionales en un conjunto de características numéricas que servirán como entrada a los modelos:

Para cada molécula se extraen las siguientes características:

- **Residual**: Índice identificador de la molécula (`resid`)
- **Magnitud**: Media y desviación estándar de los valores
- **Proporciones**: Cociente entre las medias y desviaciones estándar de los componentes (head/body para CHOL; heads/tails para DPSM)
- **Correlación**: Coeficiente de Pearson entre componentes
- **Amplitud**: Rango, valor mínimo y máximo

Se generan dos conjuntos de datos independientes:

- `chol.csv`: Características de moléculas CHOL (512 muestras)
- `dpsm.csv`: Características de moléculas DPSM (512 muestras)

**Entrada**: Datos tridimensionales crudos (Mouse y NMR).

**Salida**: Dos archivos CSV preparados para los modelos/algoritmos de ls siguientes etapas (`chol.csv`, `dpsm.csv`).

### 3. Construcción y validación de modelos (`03_training.ipynb`)

En esta etapa se entrenan modelos de clasificación independientes para cada tipo de lípido:

- Carga de los conjuntos de características generados
- Preparación de los datos (normalización, división en conjuntos de entrenamiento/validación)
- Entrenamiento de múltiples algoritmos de clasificación
- Validación cruzada y evaluación de métricas (precisión, recall, F1-score)
- Análisis del desempeño comparativo

Se utilizan técnicas de validación cruzada para evaluar la capacidad de generalización de los modelos.

**Entrada**: Archivos CSV con características (`chol.csv`, `dpsm.csv`).

**Salida**: Modelos entrenados y evaluados; métricas de desempeño.

### 4. Generación de embeddings (`04_embeddings.ipynb`)

En esta etapa final se generan representaciones de menor dimensionalidad (embeddings) de los datos utilizando las características extraídas:

- Reducción de dimensionalidad mediante técnicas como PCA o UMAP
- Visualización de los embeddings en espacios de baja dimensión
- Análisis de la separabilidad entre grupos
- Caracterización de patrones y similitudes en los espacios de embedding

**Entrada**: Características de ambos tipos de lípidos.

**Salida**: Embeddings y visualizaciones de reducción de dimensionalidad.

## Flujo de trabajo

El flujo de análisis sigue este orden:

```
Datos crudos (Mouse/NMR)
        |
        v
01_analysis.ipynb
(Exploración y estadísticas)
        |
        v
02_feat_engineering.ipynb
(Extracción de características)
        |
        +----> CHOL features
        |
        +----> DPSM features
        |
        v
03_training.ipynb
(Entrenamiento de modelos)
        |
        v
04_embeddings.ipynb
(Generación de embeddings y visualización)
```

## Características por tipo de lípido

### CHOL (Colesterol)

Columnas en `chol.csv`:

- `resid`: Índice identificador
- `lipid_type`: Tipo de lípido (CHOL)
- `mean_head`, `mean_body`: Media de valores
- `std_head`, `std_body`: Desviación estándar
- `ratio_head_body`: Proporción de medias
- `ratio_std_head_body`: Proporción de desviaciones estándar
- `corr_head_body`: Correlación de Pearson
- `min_head`, `max_head`, `range_head`: Amplitud del head
- `min_body`, `max_body`, `range_body`: Amplitud del body
- `target`: Etiqueta de clasificación (0=Mouse, 1=NMR)

### DPSM (Esfingomielina)

Columnas en `dpsm.csv`:

- `resid`: Índice identificador
- `lipid_type`: Tipo de lípido (DPSM)
- `mean_heads`, `mean_tails`: Media de valores
- `std_heads`, `std_tails`: Desviación estándar
- `ratio_heads_tails`: Proporción de medias
- `ratio_std_heads_tails`: Proporción de desviaciones estándar
- `corr_heads_tails`: Correlación de Pearson
- `min_heads`, `max_heads`, `range_heads`: Amplitud de heads
- `min_tails`, `max_tails`, `range_tails`: Amplitud de tails
- `target`: Etiqueta de clasificación (0=Mouse, 1=NMR)

## Resultados principales
- Análisis exploratorio que presentó la distribución de los datos (media y desviación estándar), análisis temporal y espacial; se encontró una alta correlación entre `DPSM_heads` y `DPSM_tails` (0.923 y 0.764) y una independencia (-0.004 y 0.009) entre `CHOL_head`y `CHOL_body` para los grupos **Mouse** y **NMR** respectivamente con $r$ de Pearson.
- Los modelos entrenados durante la validación cruzada alcanzaron valores de F1-score aproximadamente iguales a 1.0, indicando una separación clara entre los dos grupos (Mouse y NMR) basándose en las características moleculares extraídas.
- Los embeddings creados con `t-SNE` y `UMAP` presentan muy buenos resultados en las gráficas y en **Silhouette** ((`CHOL`: `t-SNE`=0.501, `UMAP`=0.598), (`DPSM`: `t-SNE`=0.631, `UMAP`=0.678)) y **Davies-Bouldin** ((`CHOL`: `t-SNE`=0.808, `UMAP`=0.658), (`DPSM`: `t-SNE`=0.526, `UMAP`=0.454)) para ambos tipos de lípidos, siendo `UMAP` con el mejor puntaje. Esto demuestra cohesión y compacidad entre los clústers resultantes.

## Requisitos

- Python 3.8 o superior
- NumPy
- Pandas
- Scikit-learn
- Matplotlib/Seaborn (para visualizaciones)
- UMAP o similar (para reducción de dimensionalidad avanzada)

## Ejecución

Para ejecutar el análisis completo:

1. Asegurar que los datos crudos están disponibles en los directorios `Mouse/` y `NMR/`
2. Ejecutar los notebooks en el orden especificado
3. Las salidas intermedias (archivos CSV y modelos) se generarán automáticamente

## Estructura de directorios esperada

```
mouse-nmr/
├── data/
│   ├── chol.csv
│   └── dpsm.csv
├── Mouse/
│   ├── g3_CHOL_head.npy
│   ├── g3_CHOL_body.npy
│   ├── g3_DPSM_heads.npy
│   ├── g3_DPSM_tails.npy
│   └── *_resids.npy
├── NMR/
│   ├── g3_CHOL_head.npy
│   ├── g3_CHOL_body.npy
│   ├── g3_DPSM_heads.npy
│   ├── g3_DPSM_tails.npy
│   └── *_resids.npy
└── notebooks/
    ├── 01_analysis.ipynb
    ├── 02_feat_engineering.ipynb
    ├── 03_training.ipynb
    └── 04_embeddings.ipynb
```

## Notas

- Todos los datos provistos están normalizados por el volumen de caja (`artificial_volume`)
- Los índices de residuo (`resids`) son identificadores únicos para cada molécula
- El análisis asume que la separabilidad entre grupos es suficiente para obtener buenos resultados de clasificación

## Uso de IA

Se utilizó Claude para las siguientes tareas:

- Recomendación de metodología
- Comprensión de organización de los datos crudos
- Corrección de ortografía y redacción para los notebooks
- Dudas conceptuales del uso de librerías
- Autocompletado de código trivial (estadísticas, gráficas, carga de datos, etc.)
- Creación de `README.md`
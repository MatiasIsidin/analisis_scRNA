# Clasificación Automática de Tipos Celulares en scRNA-seq

## Descripción Técnica del Problema
La secuenciación de ARN de célula única (scRNA-seq) permite medir la expresión génica a nivel individual, generando volúmenes masivos de datos de alta dimensionalidad. La clasificación e identificación precisa de los tipos celulares en estos conjuntos de datos es un desafío crítico en la bioinformática y biología computacional. El objetivo de este proyecto es desarrollar, evaluar y validar modelos de Machine Learning capaces de clasificar automáticamente seis linajes celulares distintos basándose en sus perfiles de expresión génica, superando problemas inherentes como la alta dimensionalidad, el ruido biológico y el fuerte desbalance de clases.

## Objetivos del Proyecto
* **Objetivo Principal:** Implementar un pipeline end-to-end de Machine Learning para la clasificación multiclase de linajes celulares a partir de datos scRNA-seq.
* **Objetivos Específicos:**
  * Reducir la dimensionalidad del espacio génico conservando la varianza explicativa mediante PCA.
  * Entrenar y comparar sistemáticamente cinco algoritmos de clasificación (Logistic Regression, Linear SVM, Random Forest, Extra Trees y XGBoost).
  * Validar la robustez y capacidad de generalización de los modelos mediante validación cruzada estratificada (5-Fold Stratified CV).
  * Seleccionar y justificar el modelo óptimo maximizando métricas ponderadas frente al desbalance de clases (F1-score macro).

## Descripción del Dataset
El proyecto emplea un conjunto de datos en formato `.h5ad` gestionado mediante el ecosistema AnnData/Scanpy.
* **Volumen:** 328,170 células (observaciones) y 2,000 genes altamente variables (características).
* **Target (`lineage`):** Clasificación multiclase con 6 linajes celulares:
  * **0:** MNP (Mononuclear Phagocytes)
  * **1:** B & plasma (Células B y Plasmáticas)
  * **2:** T (Células T)
  * **3:** NK (Natural Killers)
  * **4:** pDC (Células Dendríticas Plasmocitoides)
  * **5:** mast (Mastocitos)
* **Desbalance:** Fuerte desbalanceo, con la clase MNP como mayoritaria y mast/pDC como minoritarias.

## Tecnologías y Librerías Utilizadas
El proyecto está desarrollado en **Python 3** y emplea las siguientes librerías principales:
* **Manejo de Datos y Bioinformática:** `scanpy`, `anndata`, `numpy`, `pandas`, `scipy`
* **Modelado y Machine Learning:** `scikit-learn`, `xgboost`
* **Visualización:** `matplotlib`, `seaborn`
* **Serialización y Entorno:** `joblib`, `jupyter`, `notebook`

## Estructura del Repositorio
```text
analisis_scRNA/
├── data/
│   ├── raw/                 # Datos originales (e.g., adata_final.h5ad)
│   └── processed/           # Artefactos procesados y modelos serializados
├── notebooks/
│   ├── exploracion.ipynb    # EDA, análisis de componentes y distribuciones
│   ├── preprocesamiento.ipynb # Carga, limpieza, reducción de dimensionalidad (PCA) y división de datos
│   └── Modelos.ipynb        # Entrenamiento, hyperparameter tuning, validación cruzada y evaluación
├── reports/                 # (Opcional) Informes generados, gráficos exportados
├── scRNA_env/               # Entorno virtual sugerido (ignorado en control de versiones)
├── requirements.txt         # Dependencias críticas y versiones del proyecto
└── README.md                # Documentación principal del proyecto
```

## Flujo Completo del Pipeline
1. **Carga y Exploración:** Lectura del archivo `.h5ad` usando `scanpy` y `anndata`. Inspección de variables, calidad de datos y distribución de la variable objetivo.
2. **Preprocesamiento y Reducción de Dimensionalidad:** 
   * Aprovechamiento de representaciones latentes mediante **PCA (50 componentes principales)** para mitigar la maldición de la dimensionalidad y eliminar ruido técnico.
   * Codificación de la variable categórica `lineage` y partición estratificada de datos (80% entrenamiento, 20% prueba).
3. **Entrenamiento y Validación (Cross-Validation):**
   * Configuración sistemática de algoritmos con penalizaciones ajustadas (`class_weight='balanced'`) para abordar el desbalanceo.
   * Evaluación robusta utilizando **Stratified 5-Fold Cross Validation** para evitar sesgos de partición.
4. **Evaluación Comparativa y Análisis:**
   * Medición del desempeño en el conjunto de prueba hold-out utilizando Accuracy y F1-score macro.
   * Análisis pormenorizado de Matrices de Confusión (absolutas y normalizadas) y Reportes de Clasificación.
5. **Selección y Serialización:** Guardado automático del mejor modelo (`xgboost_best_model.pkl`) mediante `joblib` en el directorio `data/processed/`.

## Resultados y Evaluación Comparativa

El desempeño de los cinco modelos sobre el conjunto de prueba (hold-out) arrojó los siguientes resultados globales:

| Modelo | Accuracy | F1-score (Macro) |
|--------|----------|------------------|
| Logistic Regression | 0.9795 | 0.9583 |
| Linear SVM | 0.9838 | 0.9680 |
| Random Forest | 0.9854 | 0.9696 |
| Extra Trees | 0.9835 | 0.9673 |
| **XGBoost** | **0.9894** | **0.9777** |

### Resultados de Cross-Validation (5-Fold Stratified)
Para asegurar la capacidad de generalización, la evaluación en validación cruzada confirmó las métricas, mostrando desviaciones estándar extremadamente bajas (≤ 0.0017), lo que ratifica la estabilidad estadística del espacio latente y la consistencia de los modelos sin sobreajuste.

### Análisis de Matrices de Confusión
Las matrices de confusión demuestran que:
* Modelos lineales (Logistic Regression y Linear SVM) son competentes pero sufren solapamientos entre fenotipos biológicamente cercanos.
* Modelos de ensamble (Random Forest, Extra Trees) logran trazar fronteras de decisión no lineales, mejorando sensiblemente la detección de las clases raras.
* **XGBoost** exhibió una diagonal principal cuasi-perfecta, minimizando drásticamente los falsos negativos en fenotipos críticos como las células dendríticas plasmocitoides (pDC) y los mastocitos.

## Modelo Seleccionado y Justificación Técnica
**XGBoost** ha sido seleccionado como el modelo definitivo del pipeline por las siguientes razones:
1. **Rendimiento Máximo:** Obtuvo las métricas más altas tanto en Accuracy (98.94%) como en F1-score Macro (97.77%), demostrando ser el algoritmo más eficaz de manera unánime.
2. **Sensibilidad a Clases Minoritarias:** Gracias a su naturaleza de boosting secuencial y una cuidadosa selección de hiperparámetros (ej. `min_child_weight`, `subsample`, y métrica de evaluación probabilística `mlogloss`), corrigió iterativamente los errores sobre las clases menos representadas, obteniendo el mejor equilibrio global (F1-macro superior).
3. **Capacidad ante Fronteras Complejas:** La biología subyacente de la diferenciación celular implica transiciones no lineales; XGBoost logró capturar estos patrones latentes en las componentes principales con una eficacia que los clasificadores simples no pudieron igualar.
4. **Eficiencia Computacional y Escalabilidad:** El uso de la implementación mediante histogramas (`tree_method="hist"`) y el aprovechamiento de múltiples núcleos (`n_jobs=-1`) permitió paralelización y tiempos de entrenamiento altamente competitivos, un aspecto esencial en el procesamiento de más de 300,000 células de alta resolución transcriptómica.

## Instrucciones de Instalación y Ejecución

**1. Clonar el Repositorio e Ingresar al Directorio**
```bash
git clone <URL_DEL_REPOSITORIO>
cd analisis_scRNA
```

**2. Crear y Activar un Entorno Virtual**
Recomendado usar Python 3.9 o superior.
```bash
python -m venv scRNA_env
# En Windows:
scRNA_env\Scripts\activate
# En Linux/macOS:
source scRNA_env/bin/activate
```

**3. Instalar Dependencias**
Se recomienda utilizar el gestor de paquetes pip para instalar la lista consolidada de librerías.
```bash
pip install --upgrade pip
pip install -r requirements.txt
```

**4. Ubicar el Dataset**
Asegúrate de colocar el archivo de datos original (`adata_final.h5ad` o equivalente) dentro de la ruta de entrada estructurada, típicamente `data/raw/`.

**5. Ejecutar el Pipeline**
Lanzar Jupyter Notebook para ejecutar la secuencia de análisis:
```bash
jupyter notebook
```
Se deben ejecutar en el siguiente orden secuencial para reproducir los resultados:
1. `notebooks/exploracion.ipynb` - Análisis de distribuciones y verificación de calidad.
2. `notebooks/preprocesamiento.ipynb` - Extracción y reducción de dimensionalidad PCA.
3. `notebooks/Modelos.ipynb` - Entrenamiento, validación cruzada y comparación de algoritmos.

El modelo final XGBoost entrenado será serializado de forma automática en la ruta `data/processed/xgboost_best_model.pkl` al finalizar el último notebook, dejándolo listo para ingesta en producción o validación posterior.

---
*Desarrollado con rigor científico y alineado a los estándares de Data Science aplicados en biología computacional.*

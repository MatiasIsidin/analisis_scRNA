# Clasificación de Tipos Celulares en scRNA-seq

## Descripción del proyecto

Este proyecto tiene como objetivo desarrollar un modelo de Machine Learning capaz de clasificar automáticamente el tipo celular (lineage) a partir de datos de expresión génica obtenidos mediante secuenciación de ARN a nivel de célula única (scRNA-seq).

En este contexto, cada célula es representada por su perfil de expresión génica, lo que permite estudiar la composición celular de un tejido y analizar procesos biológicos complejos.

---

## Problemática

En estudios de scRNA-seq:

- Se generan datasets con cientos de miles de células  
- Cada célula contiene información de miles de genes  
- La identificación manual del tipo celular es:
  - Lenta  
  - Costosa  
  - Propensa a errores  

Sin una clasificación adecuada, no es posible:

- Entender la composición celular de un tejido  
- Identificar cambios asociados a enfermedades  
- Detectar subpoblaciones celulares relevantes  

---

## Objetivo

Desarrollar un modelo de clasificación supervisado que permita:

- Predecir el tipo celular (`lineage`)  
- Automatizar el análisis de datasets masivos  
- Apoyar la interpretación biológica  

---

## Conceptos clave

- **scRNA-seq:** Técnica que mide la expresión génica a nivel de células individuales  
- **Expresión génica:** Nivel de actividad de un gen  
- **Lineage:** Tipo celular general (ej: T, B, NK, etc.)  
- **Matriz dispersa:** La mayoría de los valores son cero  
- **PCA:** Reducción de dimensionalidad  
- **UMAP:** Visualización de datos de alta dimensión  

---

## Pipeline del proyecto hasta ahora

1. **Datos crudos**
   - Dataset en formato `.h5ad`
   - 328,170 células × 2,000 genes

2. **Preprocesamiento**
   - Uso de `X_pca` (50 componentes)
   - Encoding de la variable objetivo (`lineage`)
   - División estratificada 80/20


---

## Estructura del proyecto


analisis_scRNA/

│

├── data/

│ ├── raw/ #Datos originales (.h5ad)

│ └── processed/ #datos procesados (.npy)

│

├── notebooks/

│ ├── exploración/ #Analisis del dataset original

│ └── preprocesamiento/ #Procesado y guardado de los datos 

│

├── reports/ # Reportes

├── requirements.txt # Dependencias

└── README.md # Documentación


---

Instalación
1. Clonar el repositorio
git clone https://github.com/MatiasIsidin/analisis_scRNA.git

cd analisis_scRNA

2. Crear entorno virtual
python -m venv scRNA_env

Activar entorno:

- Windows: 

scRNA_env\Scripts\activate

- Linux / Mac:

source scRNA_env/bin/activate

3. Instalar dependencias

pip install -r requirements.txt

4. Ejecutar el proyecto

jupyter notebook

Luego abrir los notebooks dentro de la carpeta:

notebooks/
- Dataset:
Formato: .h5ad (AnnData)
328,170 células
2,000 genes

Incluye:
Expresión génica (X)
Metadatos (obs)
Información de genes (var)
PCA y UMAP

- Resultados esperados:
Clasificación automática de células
Identificación de tipos celulares
Manejo del desbalance
Pipeline reproducible

- Reproducibilidad:
Uso de random_state=42
Datos preprocesados guardados
Archivo requirements.txt incluido
Encoder guardado (.pkl)
Uso de random_state=42
Datos preprocesados guardados
requirements.txt incluido
Encoder guardado (.pkl)

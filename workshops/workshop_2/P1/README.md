# Problema 1 — Detección de Fatiga Muscular con EMG

## Descripción

En este problema trabajarás con señales de **electromiografía (EMG)** capturadas en 8 músculos de la pierna. Tu objetivo es construir un clasificador que determine, a partir de una ventana de 1 segundo de señal, si el sujeto está entrando en **transición a la fatiga** o aún se encuentra en estado normal.

Este es un problema real de procesamiento de señales biomédicas: detectar la fatiga de forma temprana — antes de que se vuelva severa — tiene aplicaciones en rehabilitación, deporte de alto rendimiento y prevención de lesiones.

---

## Dataset

| Atributo | Valor |
|----------|-------|
| Archivo | `emg_data.csv` |
| Frecuencia de muestreo | 1000 Hz |
| Canales | 8 músculos (ver tabla abajo) |
| Etiqueta (`Target`) | `0` = sin fatiga · `1` = transición a fatiga · `2` = fatiga severa |
| Formato | Una fila por muestra temporal |

### Estrategia de clases

El dataset tiene 3 estados, pero trabajaremos con **clasificación binaria**:

| Target | Estado | Muestras | Rol |
|--------|--------|----------|-----|
| `0` | Sin fatiga | ~2,127,600 (70.9%) | Clase negativa |
| `1` | Transición a fatiga | ~631,200 (21.0%) | **Clase positiva** ✅ |
| `2` | Fatiga severa | ~243,337 (8.1%) | Se descarta ❌ |

La clase 2 se descarta porque tiene muy pocas muestras para entrenar un modelo confiable. El objetivo clínico más valioso es detectar la **transición** (clase 1) a tiempo, antes de que el músculo llegue a fatiga severa.

### Canales EMG

| # | Canal | Lado |
|---|-------|------|
| 1 | Right Rectus femoris | Derecho |
| 2 | Left Gluteus maximus | Izquierdo |
| 3 | Left Gastrocnemius medialis | Izquierdo |
| 4 | Left Semitendinosus | Izquierdo |
| 5 | Left Biceps femoris caput longus | Izquierdo |
| 6 | Right Vastus medialis | Derecho |
| 7 | Right Tibialis anterior | Derecho |
| 8 | Left Gastrocnemius lateralis | Izquierdo |

---

## Estructura del taller

```
Problema 1/
├── emg_data.csv                   ← datos crudos (debes colocarlo aquí)
├── P1_01_preparacion_datos.ipynb  ← Notebook 1: procesamiento de señal
├── P1_02_entrenamiento.ipynb      ← Notebook 2: modelos ML
├── data_procesada/                ← generado por el Notebook 1
│   ├── train.csv
│   ├── val.csv
│   └── test.csv
└── README_P1.md                   ← este archivo
```

> ⚠️ **Debes ejecutar los notebooks en orden.** El Notebook 2 depende de los archivos generados por el Notebook 1.

---

## Notebook 1 — Preparación de datos

**Archivo:** `P1_01_preparacion_datos.ipynb`

### Objetivo

Transformar la señal EMG continua en una tabla de features lista para entrenar modelos clásicos de ML.

### Pipeline

```
emg_data.csv  →  División temporal  →  Segmentación  →  Features  →  data_procesada/
```

### Tareas 

#### TODO 1 — `extract_features_window`

Extrae **5 features clásicas de EMG** por canal, produciendo un vector de **40 features** (8 canales × 5).

| Feature | Fórmula | Significado |
|---------|---------|-------------|
| MAV | `mean(|x|)` | Amplitud media de activación |
| RMS | `sqrt(mean(x²))` | Potencia de la señal |
| STD | `std(x)` | Variabilidad |
| WL | `sum(|x[i+1] - x[i]|)` | Longitud de forma de onda |
| ZC | `sum(x[i] * x[i+1] < 0)` | Cruces por cero |

```python
def extract_features_window(signals: np.ndarray) -> np.ndarray:
    # signals: (1000, 8)  →  retorna: (40,)
    # orden: [MAV_ch0, RMS_ch0, STD_ch0, WL_ch0, ZC_ch0, MAV_ch1, ...]
```

### Por qué dividimos temporalmente (no aleatoriamente)

Esta es la decisión de diseño más importante del notebook. Como los datos son una **serie de tiempo continua**, mezclar filas aleatoriamente antes de dividir introduciría **data leakage**: el modelo vería muestras del futuro durante el entrenamiento, produciendo una evaluación optimista falsa.

```
CORRECTO:   [──── train 70% ────][── val 15% ──][── test 15% ──]  → por bloques de tiempo
INCORRECTO: mezclar filas y luego dividir                          → el futuro contamina el pasado
```

---

## Notebook 2 — Entrenamiento de modelos

**Archivo:** `P1_02_entrenamiento.ipynb`

### Objetivo

Entrenar y comparar **Random Forest** y **Gradient Boosting** sobre las features extraídas en el Notebook 1.

### Tareas del estudiante

#### TODO 1 — Pipelines

Construye un `Pipeline` de scikit-learn para cada modelo:

```
StandardScaler  →  RandomForestClassifier
StandardScaler  →  GradientBoostingClassifier
```

El scaler es necesario porque las features tienen escalas muy distintas (WL es ~1000× mayor que RMS). Al estar dentro del pipeline, se ajusta únicamente con datos de train — sin contaminar val ni test.

#### TODO 2 — Grilla de hiperparámetros

Define `param_grid` con al menos 3 hiperparámetros para `GridSearchCV`:

```python
param_grid = {
    'classifier__n_estimators':    [...],
    'classifier__max_depth':       [...],
    'classifier__min_samples_leaf': [...],
}
```

Nota: el prefijo `classifier__` indica que el parámetro pertenece al paso `'classifier'` del pipeline.

### Métricas de evaluación

| Métrica | Por qué se usa |
|---------|---------------|
| F1-macro | Promedia el F1 de cada clase por igual — robusta ante desbalance |
| Accuracy | Referencia general |
| AUC-ROC | Mide la capacidad discriminativa independiente del umbral |

---

## Preguntas de reflexión clave

1. ¿Por qué dividimos temporalmente y no aleatoriamente?
2. **Descarte de clase 2:** se eliminaron ~243k muestras de fatiga severa. ¿En qué escenario sería importante no descartarlas? ¿Cómo cambiaría el problema?
3. ¿Cuál feature discrimina mejor fatiga vs no-fatiga? ¿Tiene sentido fisiológico?
4. ¿Por qué el `StandardScaler` debe estar dentro del pipeline?

---

## Requisitos

```
numpy
pandas
matplotlib
scikit-learn
scipy  (opcional, para features adicionales)
```

Instalar con:
```bash
pip install numpy pandas matplotlib scikit-learn scipy
```

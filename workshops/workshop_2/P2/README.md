# Problema 2 — Estimación de Edad con CNNs

## Descripción

En este problema construirás una **red neuronal convolucional (CNN)** capaz de predecir la edad de una persona a partir de una fotografía de su rostro. Es un problema de **regresión**: la salida del modelo es un número continuo (la edad estimada), no una clase.

Este tipo de modelos tiene aplicaciones en sistemas de control de acceso, análisis demográfico y experiencias interactivas.

---

## Dataset — UTKFace

| Atributo | Valor |
|----------|-------|
| Fuente | [Kaggle — UTKFace](https://www.kaggle.com/datasets/jangedoo/utkface-new) |
| Imágenes | ~23,000 fotografías de rostros |
| Resolución de trabajo | 64×64 px, 3 canales RGB |
| Etiqueta | Edad en años (entero, rango 0–116) |
| Formato del nombre | `[age]_[gender]_[race]_[datetime].jpg` |

**Ejemplo:** `25_0_2_20170116174525125.jpg` → edad = 25

### Cómo obtener el dataset

1. Crea una cuenta en [Kaggle](https://www.kaggle.com)
2. Descarga el dataset desde: https://www.kaggle.com/datasets/jangedoo/utkface-new
3. Descomprime y coloca **todas las imágenes** directamente en la carpeta `UTKFace/`:

```
UTKFace/
  1_0_0_20161219203650533.jpg
  1_0_0_20161219222832135.jpg
  ...
```

---

## Estructura del taller

```
Problema 2/
├── UTKFace/                       ← imágenes crudas (debes colocarlas aquí)
├── P2_01_dataloaders.ipynb        ← Notebook 1: pipeline de datos
├── P2_02_entrenamiento.ipynb      ← Notebook 2: arquitectura y entrenamiento
├── dataset/                       ← generado por el Notebook 1
│   ├── train/  (~16,000 imgs)
│   ├── val/    (~3,500 imgs)
│   └── test/   (~3,500 imgs)
├── dataloaders_utk.pth            ← generado por el Notebook 1
├── mejor_simple.pth               ← generado por el Notebook 2 (baseline)
├── mejor_mejor.pth                ← generado por el Notebook 2 (tu CNN)
└── README_P2.md                   ← este archivo
```

> ⚠️ **Debes ejecutar los notebooks en orden.** El Notebook 2 carga `dataloaders_utk.pth`, generado por el Notebook 1.

---

## Notebook 1 — DataLoaders

**Archivo:** `P2_01_dataloaders.ipynb`

### Objetivo

Construir el pipeline completo de datos en PyTorch: desde las imágenes en disco hasta los `DataLoader` listos para alimentar el modelo.

```
UTKFace/  →  División  →  AgeDataset  →  Transforms  →  DataLoader
```

### Conceptos clave

**¿Por qué no cargar todas las imágenes en RAM?**

Con 23,000 imágenes de 64×64×3 = ~170 MB solo para el tensor. En resoluciones mayores o datasets más grandes esto es inviable. PyTorch resuelve esto con dos abstracciones:

```
Dataset      → sabe cómo leer UNA imagen del disco (solo cuando se necesita)
DataLoader   → agrupa N imágenes en un batch, en paralelo, mientras el modelo entrena
```

### Tareas del estudiante

#### TODO 1a — `AgeDataset.__len__`

Retorna el número total de ejemplos en el dataset.

#### TODO 1b — `AgeDataset.__getitem__`

Carga una imagen del disco y retorna el par `(tensor, edad)`:

```python
def __getitem__(self, idx):
    # 1. Desempaquetar self.samples[idx]  →  (img_path, age)
    # 2. Image.open(img_path).convert("RGB")
    # 3. Aplicar self.transform si no es None
    # 4. return image_tensor, torch.tensor(age, dtype=torch.float32)
```

> `__init__` ya está implementado: construye la lista de rutas y edades parseando los nombres de archivo. **No carga ninguna imagen.**

#### TODO 2 — Transformaciones de entrenamiento

`val_transform` (sin augmentation) ya está dado. Agrega al menos **2 augmentations** a `train_transform`:

```python
train_transform = transforms.Compose([
    transforms.Resize((64, 64)),
    # ← agrega aquí: RandomHorizontalFlip, ColorJitter, RandomRotation...
    transforms.ToTensor(),
    transforms.Normalize([0.485, 0.456, 0.406], [0.229, 0.224, 0.225]),
])
```

Las augmentations se aplican en línea — el modelo ve versiones distintas de cada imagen en cada época, sin guardar copias adicionales en disco.

#### TODO 3 — DataLoaders

Crea los tres DataLoaders con los parámetros correctos:

| Parámetro | Train | Val / Test |
|-----------|-------|-----------|
| `shuffle` | `True` | `False` |
| `drop_last` | `True` | `False` |
| `pin_memory` | `True` si hay CUDA | igual |

---

## Notebook 2 — Entrenamiento

**Archivo:** `P2_02_entrenamiento.ipynb`

### Objetivo

Implementar una CNN para regresión de edad y comparar su desempeño contra un MLP baseline.

### Modelo baseline — `SimpleCNN`

Ya está implementado como referencia. Tiene **3 bloques convolucionales**:

```
[B, 3, 64, 64]
  → Conv(3→16)  + ReLU + MaxPool  →  [B, 16, 32, 32]
  → Conv(16→32) + ReLU + MaxPool  →  [B, 32, 16, 16]
  → Conv(32→64) + ReLU + MaxPool  →  [B, 64,  8,  8]
  → Flatten  →  Linear(4096→128)  →  ReLU  →  Linear(128→1)
```

### Tarea del estudiante — `MejorCNN`

#### TODO 1 — Implementa `MejorCNN`

**Requisitos mínimos:**

- Al menos **5 bloques convolucionales** (el baseline tiene 3 — agrega 2 más)
- `BatchNorm2d` después de cada `Conv2d`
- `Dropout` antes de la capa de salida del regresor
- El `forward` debe aceptar `[B, 3, 64, 64]` y retornar `[B, 1]`

#### Cómo trazar las dimensiones

`Conv2d(padding=1, kernel_size=3)` no cambia H×W. Solo `MaxPool2d(2)` divide a la mitad:

```
Input:          [B,   3, 64, 64]
→ Conv + Pool:  [B,  32, 32, 32]
→ Conv + Pool:  [B,  64, 16, 16]
→ Conv + Pool:  [B, 128,  8,  8]
→ Conv + Pool:  [B, 256,  4,  4]   ← bloque nuevo
→ Conv + Pool:  [B, 256,  2,  2]   ← bloque nuevo
→ Flatten:      [B, 1024]          ← 256 × 2 × 2
→ Linear(1024→256) → ReLU → Dropout → Linear(256→1)
```

> Con imagen 64×64 y 5 MaxPool(2): H_final = 64 / 2⁵ = **2**. No agregues un 6.° MaxPool.

#### Tabla de experimentos

Registra tus resultados:

| Cambio | Val MAE | Test MAE |
|--------|---------|---------|
| SimpleCNN (3 bloques, baseline) | | |
| + 2 bloques convolucionales | | |
| + BatchNorm2d en cada bloque | | |
| + Dropout(0.4) en el regresor | | |
| + augmentations adicionales | | |

### Métricas

| Métrica | Fórmula | Interpretación |
|---------|---------|---------------|
| MAE | `mean(|ŷ - y|)` | Error medio en años — directamente interpretable |
| RMSE | `sqrt(mean((ŷ - y)²))` | Penaliza errores grandes más que el MAE |

La función de pérdida durante el entrenamiento es **MSELoss** (para backprop). El MAE se usa solo para reportar resultados porque es más legible.

### Conceptos a investigar antes de empezar

Antes de implementar `MejorCNN`, responde en el notebook:

1. ¿Qué hace `nn.Conv2d(in_channels, out_channels, kernel_size, padding)`?
2. ¿Qué es un **feature map**?
3. ¿Para qué sirve `MaxPool2d(2)`?
4. ¿Por qué las CNN superan a los MLP en imágenes?

**Recursos:**
- [PyTorch `nn.Conv2d`](https://pytorch.org/docs/stable/generated/torch.nn.Conv2d.html)
- [CS231n — CNNs](https://cs231n.github.io/convolutional-networks/)
- [3Blue1Brown — But what is a convolution?](https://www.youtube.com/watch?v=KuXjwB4LzSA)

---

## Preguntas de reflexión clave

1. ¿Por qué `__getitem__` carga la imagen del disco y no `__init__`?
2. ¿Qué pasa si usas `shuffle=True` en `val_loader`?
3. ¿Por qué hay que denormalizar las imágenes para visualizarlas correctamente?
4. ¿En qué rangos de edad comete más error el modelo? ¿Por qué?
5. ¿Hay un punto donde más capas convolucionales ya no mejoran el MAE?

---

## Requisitos

```
torch >= 1.13
torchvision
Pillow
numpy
matplotlib
scikit-learn  (para confusion_matrix en diagnóstico)
```

Instalar con:
```bash
pip install torch torchvision Pillow numpy matplotlib scikit-learn
```

O en Colab / Lightning Studio: las dependencias ya están disponibles.

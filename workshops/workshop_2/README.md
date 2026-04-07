# 🧠 Taller — Machine Learning & Deep Learning Aplicado

Este taller está compuesto por **dos problemas complementarios**, cada uno enfocado en un tipo de dato y enfoque de modelado distinto.

> ⚠️ **IMPORTANTE:** Lee cuidadosamente todas las instrucciones antes de comenzar.  
> Cada parte tiene su propia lógica, flujo de trabajo y objetivos.

---

## 🧩 Estructura del Taller

El taller se divide en **dos partes principales**:

---

## 📊 Parte 1 — Señales EMG y Machine Learning Clásico

📄 Ver: [README_P1.md](P1/README.md)

En esta parte trabajarás con **señales biomédicas (EMG)** para detectar fatiga muscular.

### 🎯 Aprendizajes clave

- Transformar señales continuas en muestras usando **sliding window**
- Extraer **features manuales** (ingeniería de características)
- Entrenar modelos de **Machine Learning clásico**
- Evitar **data leakage** en series de tiempo

### 🧠 Idea central

señal continua → segmentación → features → tabla → modelo ML

---

## 🖼️ Parte 2 — Imágenes y Deep Learning

📄 Ver: [README_P2.md](P1/README.md)

En esta parte trabajarás con **imágenes (dataset UTKFace)** para predecir la edad usando redes neuronales convolucionales.

### 🎯 Aprendizajes clave

- Construir pipelines de datos en **PyTorch**
- Implementar y entrenar **CNNs**
- Aplicar **data augmentation**
- Comparar modelos baseline vs modelos mejorados

### 🧠 Idea central

imágenes → dataset → dataloaders → CNN → entrenamiento → evaluación

---

## ⚠️ Instrucciones Generales (Leer con atención)

### 1. Trabaja en orden

Cada problema tiene **dos notebooks**:

- Notebook 1 → preparación de datos  
- Notebook 2 → entrenamiento de modelos  

> ⚠️ No ejecutes el segundo sin haber terminado el primero.

---

### 2. No modifiques la estructura

- No cambies nombres de carpetas o archivos  
- No alteres rutas a menos que sea estrictamente necesario  

Esto puede romper el flujo del taller.

---

### 3. Entiende antes de implementar

Cada sección está diseñada para enseñar un concepto:

- Segmentación de señales  
- Extracción de features  
- Construcción de pipelines  
- Diseño de modelos  

> ❗ No se trata solo de que funcione, sino de entender por qué funciona.

---

### 4. Lee antes de programar

Antes de escribir código:

- Revisa la descripción del problema  
- Entiende los datos  
- Identifica qué se te está pidiendo  

Muchos errores vienen de no leer bien.

---

### 5. Organiza tus experimentos

Especialmente en la Parte 2:

- Compara resultados entre modelos  
- Registra métricas (MAE, accuracy, etc.)  
- No te quedes con un solo intento  

---

## 🔁 Conexión entre las dos partes

Aunque los problemas son distintos, comparten una misma lógica:

| Parte 1 (EMG) | Parte 2 (Imágenes) |
|--------------|-------------------|
| Señales | Imágenes |
| Sliding window | Muestras de imagen |
| Features manuales | Features aprendidas |
| ML clásico | Deep Learning |

👉 Diferencia clave:

- En la **Parte 1**, tú diseñas las features  
- En la **Parte 2**, el modelo aprende las features  

---

## 🚀 Resultado esperado

Al finalizar el taller deberías ser capaz de:

- Transformar datos crudos en datasets utilizables  
- Elegir el enfoque adecuado según el tipo de datos  
- Entender la diferencia entre **feature engineering vs representation learning**  
- Construir pipelines completos de ML y DL  

---

## 🧠 Recomendación final

> Este taller no es para hacerlo rápido, es para hacerlo bien.

Tómate el tiempo de entender, experimentar y analizar.

---

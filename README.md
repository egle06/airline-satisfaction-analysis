# Análisis de Satisfacción de Pasajeros de Aerolíneas

## Objetivo
Limpiar, explorar y modelar el dataset *Airline Passenger Satisfaction* para identificar qué factores se asocian con la satisfacción de los pasajeros y evaluar modelos de clasificación supervisada. Este repositorio constituye la base del proyecto final de Ciencia de Datos.

## Origen de los datos
- **Fuente:** Kaggle — [Airline Passenger Satisfaction](https://www.kaggle.com/datasets/teejmahal20/airline-passenger-satisfaction)
- **Archivo usado:** `train.csv` (103.904 filas y 24 columnas útiles, excluyendo el índice `Unnamed: 0`). Se prioriza su lectura local para garantizar reproducibilidad sin conexión a internet.
- **Variable objetivo:** `satisfaction` (*satisfied* / *neutral or dissatisfied*).

## Estructura del repositorio
| Archivo | Descripción |
|---|---|
| `train.csv` | Dataset original utilizado en ambos notebooks. |
| `Pre-Entrega: Limpieza y Análisis Exploratorio de un Dataset Real.ipynb` | **Etapa 1:** Carga, diagnóstico inicial, limpieza descriptiva y análisis exploratorio (EDA). |
| `Entrenar_y_evaluar_sin_trampas_train_test_y_sobreajuste.ipynb` | **Etapa 2:** Preprocesamiento sin fuga de datos (*data leakage*), partición estratificada, validación cruzada (`cross_val_score`), ajuste de hiperparámetros (`max_depth`) y comparación entre Árbol de Decisión y Random Forest. |

---

## Etapa 1: Limpieza y Análisis Exploratorio (EDA)
**Archivo:** `Pre-Entrega: Limpieza y Análisis Exploratorio de un Dataset Real.ipynb`

### Diagnóstico inicial
- **Nulos:** Solo en `Arrival Delay in Minutes` (310 registros, 0,30 %).
- **Duplicados:** 0 filas duplicadas y 0 `id` repetidos.
- **Categorías con mayúsculas inconsistentes:** `disloyal Customer` vs. `Loyal Customer`, y `Business travel` vs. `Personal Travel`.
- **Ceros en las escalas de servicio (1 a 5):** 15.932 ceros distribuidos en 8.200 filas (7,9 %), interpretados como *"no aplica / sin respuesta"*.
- **Asimetría:** Demoras muy sesgadas hacia la derecha, con valores máximos superiores a 1.500 minutos.
- **Fechas:** El dataset no incluye columnas temporales.

### Decisiones de limpieza
| Problema | Decisión | Justificación |
|---|---|---|
| Duplicados | `drop_duplicates(subset='id')` | No había duplicados; se incluye por robustez en el flujo. |
| Categorías inconsistentes | `.str.strip().str.title()` | Unificar el formato de texto. |
| 310 nulos en `Arrival Delay in Minutes` | Imputación con la **mediana** (0 min) | Distribución altamente asimétrica; la media (15,2 min) está inflada por valores extremos. |
| 15.932 ceros en escalas de servicio | Reemplazo por la **moda** de cada escala | Son variables ordinales enteras; eliminar filas implicaría perder el 7,9 % de los datos. |
| Outliers en demoras | **Se conservaron** | Representan casos reales (cancelaciones o demoras severas) asociados a la insatisfacción; se mitigan usando la mediana. |
| Tipos de datos y orden | `Arrival Delay` a `int`; textos a `category`; `reset_index(drop=True)` | Optimizar memoria, reflejar la naturaleza de cada variable y mantener el índice continuo. |

### Agrupaciones y principales hallazgos
1. **Clase y tipo de viaje (% de satisfechos):** Es el factor más determinante. En viajes de negocios, el 72,0 % de quienes viajan en clase *Business* está satisfecho, frente al 29,6 % en *Eco* y 39,3 % en *Eco Plus*. En viajes personales la satisfacción cae en las tres clases (8,7 % a 12,2 %).
2. **Wifi y comodidad del asiento:** Los pasajeros satisfechos califican casi 1 punto por encima ambos servicios (wifi: 3,37 vs. 2,40; asiento: 3,97 vs. 3,04).
3. **Demoras (media y mediana):** La mediana es 0 min en ambos grupos y las medias difieren en apenas ≈ 4 minutos, por lo que de forma aislada no resultan un diferenciador fuerte.
4. **Tipo de cliente:** Los clientes leales registran el doble de satisfacción que los no leales (47,7 % vs. 23,7 %).

### Limitaciones del análisis exploratorio
- Imputar ceros con la moda asume que los pasajeros sin respuesta habrían elegido el valor más frecuente (~1,1 % de las celdas de escala).
- Los cruces descriptivos reflejan asociaciones estadísticas, no relaciones causales.

---

## Etapa 2: Modelado, Validación y Control de Sobreajuste
**Archivo:** `Entrenar_y_evaluar_sin_trampas_train_test_y_sobreajuste.ipynb`

### Preparación y prevención de fuga de datos (*Data Leakage*)
- **Target (`y`):** `satisfaction` codificada como `1` (*satisfied*) y `0` (*neutral or dissatisfied*).
- **Características (`X`):** 23 columnas predictoras (se descartan `id` y `satisfaction`; las variables categóricas se transforman mediante `pd.get_dummies` con `drop_first=True`).
- **División estratificada:** `train_test_split` con `test_size=0.2`, `random_state=42` y `stratify=y`, manteniendo una proporción exacta de 43,3 % de casos positivos tanto en **Train** (83.123 filas) como en **Test** (20.781 filas).
- **Imputación post-split:** Tanto la mediana de `Arrival Delay in Minutes` como las modas de las 14 columnas de servicio se calcularon **exclusivamente sobre `X_train`** y luego se aplicaron a `X_test`, eliminando cualquier fuga de información del conjunto de prueba.

### Selección de hiperparámetros y comparación de modelos
Para elegir la profundidad máxima (`max_depth`) sin introducir sesgo de optimización sobre el conjunto de prueba, se utilizó **Validación Cruzada de 5 particiones (`cross_val_score`, `cv=5`)** únicamente dentro de `X_train`, identificando `max_depth = 15` como el punto óptimo de generalización. Posteriormente se incorporó un **Random Forest** para reducir la varianza del árbol individual.

| Modelo | Train | CV (5-Fold) | Test | Brecha (Train - Test) |
|---|:---:|:---:|:---:|:---:|
| **1. Árbol sin límite (Base)** | 100,00 % (1,0000) | 93,45 % (0,9345) | 93,62 % (0,9362) | 6,38 p.p. (0,0638) |
| **2. Árbol podado (`max_depth = 15`)** | 97,25 % (0,9725) | 94,25 % (0,9425) | 94,62 % (0,9462) | 2,62 p.p. (0,0262) |
| **3. Random Forest (100 árboles, `max_depth = 15`)** | **97,69 % (0,9769)** | **95,08 % (0,9508)** | **95,24 % (0,9524)** | **2,46 p.p. (0,0246)** |
| **Referencia (predecir siempre «no satisfecho»)** | — | — | 56,67 % (0,5667) | — |

### Conclusiones del modelado y próximos pasos
1. **Diagnóstico del árbol base:** Sin restricción de profundidad, el árbol memoriza el conjunto de entrenamiento (100,00 % de acierto con 34 niveles) y cae a 93,62 % en prueba (brecha de 6,38 p.p.), evidenciando **sobreajuste moderado**. No existe *underfitting*, ya que supera ampliamente el baseline de 56,67 %.
2. **Efecto de la regularización (`max_depth = 15`):** Podar el árbol redujo la brecha entre entrenamiento y prueba a menos de la mitad (2,62 p.p.) y mejoró la exactitud en datos no vistos en +1,00 punto porcentual (94,62 %).
3. **Salto a métodos de ensamble (Random Forest):** Al promediar 100 árboles entrenados sobre submuestras aleatorias, el modelo alcanzó el mejor rendimiento en validación cruzada (95,08 %) y en prueba (95,24 %), logrando además la menor brecha de sobreajuste (2,46 p.p.).
4. **Siguientes pasos:** Complementar el *accuracy* con métricas detalladas por clase (**Matriz de confusión, Precisión, Recall y F1-Score**) para evaluar el equilibrio entre falsos positivos y falsos negativos.

---

## Autora
**Egle Martinelli**

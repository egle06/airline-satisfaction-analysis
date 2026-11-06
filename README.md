# Análisis de satisfacción de pasajeros de aerolíneas

## Objetivo
Limpiar, explorar y modelar el dataset *Airline Passenger Satisfaction* para identificar qué factores se asocian con la satisfacción de los pasajeros y evaluar un modelo de clasificación. Es la base del proyecto final de Ciencia de Datos.

## Origen de los datos
- **Fuente:** Kaggle, [Airline Passenger Satisfaction](https://www.kaggle.com/datasets/teejmahal20/airline-passenger-satisfaction)
- **Archivo usado:** `train.csv` (103.904 filas y 24 columnas, sin contar la columna de índice `Unnamed: 0`)
- **Variable objetivo:** `satisfaction` (*satisfied* / *neutral or dissatisfied*)

## Contenido del repositorio
- `Pre-Entrega: Limpieza y Análisis Exploratorio de un Dataset Real.ipynb`: carga, diagnóstico, limpieza y agrupaciones.
- `Entrenar_y_evaluar_sin_trampas_train_test_y_sobreajuste.ipynb`: separación en train y test, entrenamiento de un árbol de decisión y análisis de sobreajuste.
- `train.csv`: dataset original.

## Diagnóstico inicial
- Nulos: solo `Arrival Delay in Minutes` (310 registros, 0,30 %).
- Duplicados: 0 filas duplicadas y 0 `id` repetidos.
- Categorías con mayúsculas inconsistentes: `disloyal Customer` vs `Loyal Customer`, y `Business travel` vs `Personal Travel`.
- Ceros en las escalas de servicio (1 a 5): 15.932 ceros en 8.200 filas (7,9 %), que se interpretan como "no aplica / sin respuesta".
- Demoras muy asimétricas, con máximos superiores a 1.500 minutos.
- El dataset no tiene columnas de fecha, por lo que no hubo conversiones de ese tipo.

## Decisiones de limpieza
| Problema | Decisión | Justificación |
|---|---|---|
| Duplicados | Se verificó con `drop_duplicates(subset='id')` | No había duplicados; se deja como control |
| Categorías inconsistentes | `.str.strip().str.title()` | Unificar el formato de escritura |
| 310 nulos en `Arrival Delay in Minutes` | Imputación con la **mediana** (0 min) | La variable es muy asimétrica; la media (15,2 min) está inflada por outliers |
| 15.932 ceros en escalas de servicio | Reemplazo por la **moda** de cada columna | Son escalas ordinales enteras; eliminar filas perdería el 7,9 % del dataset |
| Outliers en demoras | **No se eliminaron** | Son casos plausibles (cancelaciones, fallas técnicas) y pueden estar asociados a la insatisfacción. Su efecto se neutraliza usando la mediana en las agrupaciones.|
| Tipos de datos | `Arrival Delay` a entero; 5 columnas de texto a `category` | Reflejar la naturaleza de cada variable |
| Índice | `reset_index(drop=True)` | Dejar el índice ordenado tras la limpieza |

## Agrupaciones realizadas
1. % de satisfechos por clase y tipo de viaje.
2. Valoración promedio de wifi y asiento según satisfacción.
3. Demoras (media y mediana) según satisfacción.
4. Satisfacción por tipo de cliente.

## Principales hallazgos

1. **Clase y tipo de viaje:** es el factor con mayor diferencia. En viajes de negocios, el 72 % de los pasajeros de clase Business está satisfecho, frente al 29,6 % en Eco y 39,3 % en Eco Plus. En viajes personales la satisfacción es baja en las tres clases (8,7 % a 12,2 %). El grupo Business + Personal Travel es reducido (2.157 pasajeros), por lo que su porcentaje es menos estable.
2. **Wifi y asiento:** los pasajeros satisfechos puntúan casi un punto más alto en ambos servicios (wifi: 3,37 vs 2,40; asiento: 3,97 vs 3,04), lo que respalda la hipótesis de que son factores relevantes.
3. **Demoras:** las medias difieren poco (≈ 4 minutos) y la mediana es 0 en ambos grupos. Con este análisis descriptivo, las demoras no parecen un factor diferenciador fuerte.
4. **Tipo de cliente:** los clientes leales tienen una satisfacción del 47,7 %, el doble que los desleales (23,7 %).

**Conclusión preliminar:** la hipótesis se sostiene parcialmente. Wifi y asiento muestran diferencias claras, pero las demoras casi no se diferencian, y la clase combinada con el tipo de viaje pesa más de lo supuesto. Estas son asociaciones descriptivas, no relaciones causales; la importancia relativa de cada variable se evaluará con modelos en etapas posteriores.


## Limitaciones del análisis exploratorio
- Imputar con la moda asume que quien no respondió habría elegido la respuesta más común (afecta ~1,1 % de las celdas de escala).
- Las agrupaciones muestran asociaciones, no causalidad.

## Evaluación de un modelo: separación en train y test

### Archivo
`Entrenar_y_evaluar_sin_trampas_train_test_y_sobreajuste.ipynb`: reproduce la carga y la limpieza descritas en la sección anterior (`df_limpio`) y entrena un clasificador para predecir la satisfacción del pasajero.

### Preparación
- **Target (`y`):** `satisfaction` codificada como 1 (satisfecho) y 0 (neutral o insatisfecho).
- **Características (`X`):** 23 columnas. Se descartaron `id` (identificador) y el propio target, y las 4 variables de texto se convirtieron en columnas 0/1 con `get_dummies`.

### División usada
- `train_test_split` con **`test_size=0.2`** y **`random_state=42`** (división reproducible).
- **Train:** 83.123 pasajeros (80 %) | **Test:** 20.781 pasajeros (20 %).
- El balance del target es similar en ambos conjuntos (43,3 % y 43,6 % de satisfechos).
- El modelo se entrenó **solo con `X_train` e `y_train`**; el conjunto de prueba se usó únicamente para evaluar.

### Modelo y resultados
`DecisionTreeClassifier` sin límite de profundidad (no requiere escalar las variables).

| Conjunto | Score (accuracy) |
|---|---|
| Train | 1,0000 |
| Test | 0,9395 |

Referencia: predecir siempre «no satisfecho» daría 0,5636.

### Conclusión del experimento
El árbol presenta **sobreajuste moderado**: acierta el 100 % de los pasajeros de entrenamiento (los memoriza, con 34 niveles y 3.737 hojas), pero baja a 93,95 % en test, una diferencia de 6,05 puntos. Aun así supera ampliamente la referencia de 56,4 %, por lo que aprendió patrones reales; no hay *underfitting*.

Al limitar la profundidad, la brecha se reduce: con `max_depth=15` el test sube a 94,55 % y la diferencia baja a 2,7 puntos. Más allá de esa profundidad, el puntaje en train sigue subiendo mientras el de test cae.

**Limitaciones:** la profundidad se comparó usando el conjunto de prueba, lo que en un proyecto real debería hacerse con validación cruzada. El *accuracy* es una métrica simple; más adelante conviene complementarla con precisión, recall y matriz de confusión.
## Autora
Egle Martinelli

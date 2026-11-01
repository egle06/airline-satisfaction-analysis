# Análisis de satisfacción de pasajeros de aerolíneas

## Objetivo
Limpiar y explorar el dataset *Airline Passenger Satisfaction* para identificar qué factores se asocian con la satisfacción de los pasajeros. Es la base del proyecto final de Ciencia de Datos.

## Origen de los datos
- **Fuente:** Kaggle, [Airline Passenger Satisfaction](https://www.kaggle.com/datasets/teejmahal20/airline-passenger-satisfaction)
- **Archivo usado:** `train.csv` (103.904 filas y 24 columnas, sin contar la columna de índice `Unnamed: 0`)
- **Variable objetivo:** `satisfaction` (*satisfied* / *neutral or dissatisfied*)

## Contenido del repositorio
- `Pre-Entrega: Limpieza y Análisis Exploratorio de un Dataset Real.ipynb`: notebook con carga, diagnóstico, limpieza y agrupaciones.
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


## Limitaciones
- Imputar con la moda asume que quien no respondió habría elegido la respuesta más común (afecta ~1,1 % de las celdas de escala).
- Las agrupaciones muestran asociaciones, no causalidad.

## Autora
Egle Martinelli

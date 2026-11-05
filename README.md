# TP3 - Aprendizaje No Supervisado: segmentación de clientes mayoristas

## Descripción del dataset

Wholesale Customers (UCI Machine Learning Repository, también en Kaggle): 440 clientes de un distribuidor mayorista en Portugal, con su gasto anual en distintas categorías de producto. No tiene valores faltantes.

Variables usadas en el análisis (6, todas numéricas y continuas):

| Variable | Descripción |
|---|---|
| Fresh | Gasto anual en productos frescos |
| Milk | Gasto anual en lácteos |
| Grocery | Gasto anual en almacén |
| Frozen | Gasto anual en congelados |
| Detergents_Paper | Gasto anual en detergentes y papel |
| Delicassen | Gasto anual en delicatessen |

`Channel` (Horeca o Retail) y `Region` son códigos categóricos y se excluyeron del agrupamiento. `Channel` se usó solo al final como referencia externa para validar los clusters.

## Objetivo

Comparar tres representaciones del mismo conjunto de datos (variables originales estandarizadas, dos componentes principales y un subconjunto de variables sin redundancia) y evaluar cómo cambia la calidad del agrupamiento con K-means (K entre 2 y 6), con el fin de decidir qué enfoque es más apropiado para segmentar clientes.

## Contenido del repositorio

- `TP3_No_Supervisado.ipynb`: notebook completo con explicaciones, visualizaciones e interpretación.
- `Wholesale_customers_data.csv`: datos.
- `resultados_kmeans.csv`: tabla comparativa (enfoque, K, inercia, silhouette, tamaños de cluster).
- `figures/`: gráficos generados por el notebook.

Para reproducir: `pip install pandas numpy scikit-learn matplotlib seaborn jupyter` y ejecutar el notebook (semilla fija en 42).

## Metodología

1. Estandarización con `StandardScaler`.
2. PCA sobre las 6 variables: scree plot, cargas y dispersión en el plano PC1-PC2.
3. Selección de variables por correlación: se eliminan variables redundantes con |r| > 0.70.
4. K-means (`n_init=10`) con K de 2 a 6 sobre las tres versiones, evaluado con método del codo y silhouette score, más tamaños de cluster y validación externa con `Channel`.

## Principales hallazgos

- PC1 y PC2 explican 44.1% y 28.4% de la varianza (72.5% acumulado). PC1 es un eje de consumo de Grocery, Detergents_Paper y Milk (perfil minorista); PC2 está dominado por Frozen, Fresh y Delicassen.
- Grocery y Detergents_Paper tienen r = 0.92, y Milk-Grocery r = 0.73. Se conservó Milk y se retuvieron Fresh, Milk, Frozen y Delicassen.
- Los datos tienen asimetría fuerte y valores extremos: en el plano PCA se ve un núcleo denso con una cola de clientes de gasto muy alto, sin grupos separados por zonas vacías.

Silhouette por enfoque y K:

| K | Original (6 var.) | PCA (2 comp.) | Selección (4 var.) |
|---|---|---|---|
| 2 | 0.547 | 0.636 | 0.720 |
| 3 | 0.548 | 0.627 | 0.503 |
| 4 | 0.348 | 0.489 | 0.485 |
| 5 | 0.369 | 0.488 | 0.383 |
| 6 | 0.378 | 0.459 | 0.318 |

Con K=2, tamaño del cluster minoritario y su composición según `Channel`:

| Enfoque | n minoritario | % Retail dentro del minoritario |
|---|---|---|
| Original | 51 | 90.2% |
| PCA | 45 | 95.6% |
| Selección | 19 | 26.3% |

## Conclusiones

- PCA mejora el silhouette respecto de las variables originales en todos los valores de K.
- El silhouette más alto de la tabla (selección de variables, K=2) es engañoso: ese cluster agrupa solo 19 clientes extremos, y casi no coincide con el canal Retail. Un silhouette alto no equivale a una buena segmentación.
- Enfoque recomendado: PCA con K=2. Es el que combina buen silhouette, interpretabilidad (dos ejes con lectura de negocio) y coherencia externa: su cluster minoritario (45 clientes) es en un 95.6% Retail, con alto gasto en almacén, limpieza y lácteos. K=3 con PCA agrega un cluster de solo 3 clientes, por lo que no suma información útil.
- Limitaciones: la inercia y el silhouette se calculan en espacios de distinta dimensión y no son estrictamente comparables; el método del codo es poco concluyente; K-means es sensible a los atípicos, y una transformación logarítmica previa a la estandarización sería el siguiente paso natural; ningún enfoque captura a todos los clientes Retail (cerca de 30% con PCA).

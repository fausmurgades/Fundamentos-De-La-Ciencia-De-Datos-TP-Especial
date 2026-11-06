# Cambios: UMAP y regresión logística

Decisiones y reflexiones sobre la sección de hipótesis (desde "Hipótesis NUESTRAS") de `nacho.ipynb` y `entrega.ipynb`, que ahora tienen el mismo contenido. Las celdas anteriores (división de grupos e histogramas) no se modificaron.

## Hipótesis multivariada: causas y consecuencias

- **Causas (hábitos):** IMC, frutas y verduras por día, horas sentado por día, actividad física, tabaco y consumo episódico excesivo de alcohol.
- **Controles:** edad y sexo.
- **Consecuencias:** hipertensión y diabetes, analizadas por separado. Juntarlas en "HTA o diabetes" daba una variable casi igual a HTA (correlación 0,88).
- **Glucemia y colesterol quedan afuera:** son mediciones del lado de las consecuencias, no hábitos, y la glucemia es parte del diagnóstico de diabetes. Sacarlas además permite usar 23.333 casos en lugar de los ~4.000 de la submuestra bioquímica.
- **Corrección de codificación:** según el diccionario, en `consumo_tabaco_100` el código 2 es "ex fumador". Los UMAP anteriores lo contaban como "fuma". Ahora el tabaco tiene tres categorías: fumador actual, ex fumador y no fumador.
- **Alcohol:** se usa el consumo episódico excesivo en lugar de "tomó alcohol en los últimos 30 días", que es lo que plantea la hipótesis.

## UMAP

- Los dos UMAP anteriores se reemplazaron por uno solo, construido únicamente con las causas y la edad (sin sexo) y coloreado por cada consecuencia.
- **Lectura:** las islas del mapa son combinaciones de las causas categóricas (actividad física, tabaco y alcohol). La hipertensión y la diabetes aparecen en todas las islas y se concentran del lado de mayor edad.
- **El UMAP no alcanza para la hipótesis:** es exploratorio y no usa la enfermedad, así que no permite testearla. Por eso se agregó la regresión logística.
- **PCA descartado:** se probó y se eliminó. La varianza quedaba muy repartida (PC1 explicaba el 18,8% y hacían falta 6 de 8 componentes para llegar al 80%), así que no resumía las variables.

## Regresión logística

- **Formato de clase:** se usa `Logit` de statsmodels para leer el `summary` como en clase. Pseudo R² equivale a R², `LLR p-value` al test F y `P>|z|` al test t.
- **H0:** el coeficiente de cada hábito vale 0. Se rechaza cuando el intervalo de confianza del 95% no contiene al 0.
- **Holdout 70/30:** estratificado por la combinación HTA × diabetes, para mantener las prevalencias en los dos conjuntos.
- **Estandarización después de separar:** las variables continuas se estandarizan con la media y el desvío del conjunto de entrenamiento, para que el de prueba no aporte información al ajuste. Las variables 0/1 quedan sin estandarizar, así su odds ratio sigue leyéndose como "sí contra no". Los odds ratio continuos se leen por desvío estándar.
- **Sin ponderar por `wf1p`:** el `summary` de `Logit` no admite pesos, así que las conclusiones valen para la muestra, no para la población.
- **Modelo reducido:** se quitan las variables cuyo IC del 95% contiene al 0. El de HTA pasa de 10 a 5 variables con el mismo Pseudo R² (0,123) y menor AIC y BIC; el de diabetes solo pierde fumador actual. Por parsimonia se prefieren los reducidos.
- **Multicolinealidad:** todos los VIF son menores a 2,2. Normalidad y homocedasticidad no se evalúan porque son supuestos de la regresión lineal.

## Evaluación: desbalance, ponderación y Ridge

- **El problema era el desbalance, no el overfitting:** el modelo acertaba lo mismo en entrenamiento que en prueba. Con 14% de diabéticos, predecía casi siempre "No": recall 0,01, y una accuracy igual a la de predecir siempre "No".
- **Ponderación de errores** (`class_weight='balanced'`): un falso negativo pesa tanto más que un falso positivo como veces es más frecuente la clase "No" (unas 6 veces en diabetes y 1,7 en HTA). Se eligió porque a los enfermos hay que detectarlos.
- **Ridge con λ = 1:** se probaron λ = 1, 1.000 y 10.000 con 10-fold cross validation. El λ más chico fue el mejor y las métricas casi no cambiaban, lo esperable sin overfitting. Se dejó λ = 1 y se sacó la validación cruzada para simplificar.
- **Resultado:** el recall sube de 0,46 a 0,69 en HTA y de 0,01 a 0,67 en diabetes. El costo es la precisión en diabetes, que queda en 0,22 (1 de cada 5 casos marcados es diabético). Para detección temprana es un buen intercambio: un falso positivo se descarta con un análisis, y un falso negativo queda sin diagnosticar.

## Reflexiones

- **Se rechaza H0, pero no para todos los hábitos:** IMC y actividad física se asocian con las dos enfermedades. Fumar actualmente no se asocia con ninguna.
- **Causalidad inversa:** los ex fumadores tienen más chance de enfermar, y en diabetes el alcohol aparece como protector y las frutas y verduras suben la chance. La encuesta es transversal, y quien ya fue diagnosticado suele cambiar sus hábitos. Los resultados muestran asociación, no causalidad.
- **La edad pesa más que cualquier hábito:** se ve tanto en el UMAP como en la regresión.
- **Explicar no es predecir:** los hábitos explican diferencias de riesgo entre grupos (Pseudo R² de 0,12 en HTA y 0,06 en diabetes), pero no alcanzan para predecir quién está enfermo sin muchas falsas alarmas.

## Otros cambios

- **Error corregido:** la variable `ref_minmax` se usaba en la hipótesis grupal sin estar definida.
- **Dependencias:** se agregaron `umap-learn` y `statsmodels` (con `patsy` y `formulaic`) a `requirements.txt`.
- **Limpieza:**
  - Se eliminaron comentarios redundantes y la supresión global de warnings (`warnings.filterwarnings('ignore')`).
  - Se unificó la preparación de datos que comparten el UMAP y la regresión.

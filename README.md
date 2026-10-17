# Predicción del tiempo de retención cromatográfica mediante QSRR y Machine Learning

Proyecto final de **Aprendizaje de Máquina** de la Maestría en Ciencia de Datos (MSDS) de la Universidad San Francisco de Quito (USFQ). La implementación, los resultados, las gráficas y las conclusiones están contenidos en la notebook [ProyectoMLmasdatos.ipynb](./ProyectoMLmasdatos.ipynb), de acuerdo con la rúbrica del curso.

## Resumen

El proyecto desarrolla un modelo de regresión para predecir el tiempo de retención cromatográfica (rt) a partir de descriptores moleculares. Se integran datos de retención y descriptores del repositorio RepoRT para construir un flujo QSRR (*Quantitative Structure–Retention Relationship*). Se compara Random Forest con Ridge, se optimizan hiperparámetros mediante validación cruzada repetida y se analizan curvas de aprendizaje y regularización.

En la ejecución registrada se integraron **2.968 filas QSRR** con **286 predictores numéricos**, tras procesar hasta los primeros 80 estudios de retención ordenados por ruta. El Random Forest ajustado obtuvo **R² = 0.4996**, **MSE = 27.3896** y **RMSE = 5.2335** en el conjunto de prueba. Las curvas muestran que incorporar más datos mejora el desempeño de validación, aunque persiste una brecha respecto al entrenamiento.

## Problema y objetivo

En cromatografía, el tiempo de retención depende de las propiedades de la molécula y de las condiciones experimentales. El objetivo es aprender una relación entre los descriptores disponibles y rt para estimar el tiempo de retención de compuestos. Se trata de un problema de **regresión supervisada**: rt es la variable objetivo y los descriptores moleculares son los predictores.

## Datos

- **Fuente:** RepoRT, repositorio público de datos de retención cromatográfica y descriptores moleculares.
- **Consolidación:** se leen archivos TSV de retención y se enlazan con sus descriptores usando el identificador molecular.
- **Límite de estudios:** se consideran como máximo los primeros 80 archivos de retención después de ordenar las rutas. No todos necesariamente aportan filas a la tabla integrada.
- **Muestra integrada de la ejecución registrada:** 2.968 filas QSRR y 302 columnas totales, incluidas la variable objetivo y campos auxiliares; 286 predictores numéricos se usan en el modelo.
- **Variable objetivo:** rt (tiempo de retención).
- **División:** conjunto de entrenamiento y conjunto de prueba reservada. La búsqueda y las curvas de aprendizaje se ajustan con entrenamiento; el conjunto de prueba se usa para la evaluación final indicada en la notebook.

Los datos de retención consolidados contienen más registros que los que finalmente se pueden enlazar con descriptores. Por ello, el límite de 80 archivos no equivale a 80 estudios con contribución efectiva a la tabla final.

## Metodología

1. **Carga e integración:** localizar los TSV de retención y de descriptores, consolidarlos y unirlos por identificador.
2. **Análisis exploratorio (EDA):** examinar la distribución de rt, las dimensiones y los valores faltantes; visualizar la distribución de la variable objetivo y la proporción de datos ausentes.
3. **Preparación y selección de variables:** usar imputación por mediana y VarianceThreshold dentro de pipelines para procesar faltantes y descartar predictores sin variabilidad suficiente.
4. **Modelo principal:** construir un pipeline de scikit-learn que termina en RandomForestRegressor.
5. **Validación cruzada:** emplear RepeatedKFold con 5 pliegues y 2 repeticiones (10 puntuaciones por configuración), con random_state=42.
6. **Optimización:** evaluar 5 combinaciones muestreadas de n_estimators, max_depth y min_samples_split mediante RandomizedSearchCV. Se informa el R² medio de validación y se evalúa el mejor estimador en el conjunto de prueba.
7. **Curvas de aprendizaje:** medir R² y MSE de entrenamiento y validación frente al tamaño de la muestra, para analizar el efecto de disponer de más observaciones y diagnosticar una brecha de generalización.
8. **Regularización y comparación:** estudiar el hiperparámetro alpha (lambda) del modelo Ridge estandarizado con una curva de validación; comparar las puntuaciones de Random Forest y Ridge con la prueba de rangos con signo de Wilcoxon.
9. **Visualización exploratoria:** proyectar una muestra de descriptores a dos dimensiones con t-SNE y representar la importancia de variables del Random Forest.

La extracción de características mediante transfer learning no se aplica directamente: el proyecto trabaja con descriptores tabulares ya calculados, no con datos no estructurados como imágenes, texto o señales. Esta decisión se explica en la notebook.

## Resultados de la ejecución registrada

| Evaluación | Resultado |
|---|---:|
| Filas QSRR integradas | 2.968 |
| Predictores numéricos | 286 |
| Random Forest base: R² en prueba | 0.5092 |
| Random Forest base: R² medio en repeated k-fold | 0.4466 ± 0.0479 |
| Mejor configuración encontrada | n_estimators=100, max_depth=10, min_samples_split=5 |
| Random Forest ajustado: R² medio de validación cruzada | 0.4545 |
| Random Forest ajustado: R² en prueba | 0.4996 |
| Random Forest ajustado: MSE en prueba | 27.3896 |
| Random Forest ajustado: RMSE en prueba | 5.2335 |
| Ridge estandarizado: lambda seleccionado por validación | 0.3162 |
| Ridge: MSE medio de validación en lambda seleccionado | 32.4256 |
| Comparación repeated k-fold: R² medio Random Forest | 0.4466 |
| Comparación repeated k-fold: R² medio Ridge (alpha predeterminado = 1) | 0.3474 |
| Test de Wilcoxon sobre puntuaciones emparejadas | W = 0.0000; p = 0.00195 |

La búsqueda elevó ligeramente el R² medio de validación cruzada frente al modelo base, pero no mejoró su R² de prueba en esta ejecución (0.4996 frente a 0.5092). La configuración encontrada no debe presentarse como una mejora definitiva de generalización; la diferencia de prueba es pequeña y debe juzgarse con nuevas validaciones.

En la curva de aprendizaje, el R² de validación aumenta aproximadamente de 0.24 a 0.45 y el MSE de validación disminuye aproximadamente de 38 a 29 al crecer el tamaño de entrenamiento. La tendencia sugiere que más datos ayudan, pero la brecha entre las curvas de entrenamiento y validación continúa siendo visible. Por tanto, el aumento de muestra reduce el problema, pero no elimina el sobreajuste.

La curva de Ridge selecciona lambda = 0.3162 dentro del rango probado (10⁻³ a 10⁴). El MSE de validación crece al incrementar mucho la penalización, comportamiento compatible con subajuste. Este valor se seleccionó por validación y no se usó en la comparación Wilcoxon reportada: allí Ridge conserva alpha=1 (valor predeterminado). La comparación estadística, por tanto, corresponde a Random Forest frente a ese Ridge estandarizado, no frente al Ridge optimizado.

## Interpretación y limitaciones

- El modelo capta una parte útil de la variación de rt, pero los resultados muestran margen considerable de mejora.
- La mejora de las curvas con más datos respalda seguir ampliando la muestra, aunque no permite concluir que el sobreajuste haya desaparecido.
- El valor p de Wilcoxon es nominal y debe interpretarse con cautela: las puntuaciones de pliegues en validación cruzada repetida no son observaciones completamente independientes. Tampoco se debe extrapolar este resultado a estudios cromatográficos nuevos.
- En la ejecución aparecen advertencias asociadas con descriptores completamente vacíos. El análisis exploratorio muestra columnas con proporción de valores ausentes del 100%; su tratamiento debe revisarse antes de usar el modelo fuera de este ejercicio.
- Una división aleatoria por filas puede ubicar mediciones del mismo estudio en entrenamiento y prueba. Una evaluación más estricta de generalización a estudios nuevos debería separar los datos por study_id (validación agrupada).
- El hiperparámetro Ridge seleccionado y el rendimiento de la comparación estadística no corresponden al mismo modelo: la curva selecciona alpha=0.3162, mientras que la comparación usa alpha=1. Una comparación confirmatoria debería incorporar la selección de alpha dentro de un procedimiento de validación apropiado, idealmente anidado.
- t-SNE y la importancia de características son herramientas exploratorias; no prueban causalidad ni identifican por sí solas efectos químicos.

## Estructura del repositorio

- [ProyectoMLmasdatos.ipynb](./ProyectoMLmasdatos.ipynb): notebook principal con código, explicación metodológica, resultados, visualizaciones, conclusiones y referencias.
- [README.md](./README.md): resumen del problema, metodología y resultados.

## Ejecución

1. Abrir la notebook en Google Colab.
2. Ejecutar las celdas en orden para descargar o acceder al repositorio RepoRT, integrar los datos, ajustar los modelos y generar las figuras.
3. La primera celda clona RepoRT solo si la carpeta aún no existe; si ya está disponible, reutiliza esa copia.
4. Interpretar las métricas de prueba junto con las curvas de aprendizaje y las limitaciones descritas; no comparar resultados de ejecuciones distintas sin conservar la configuración y la muestra utilizada.

## Referencias

[1] RepoRT, “RepoRT: a repository of chromatographic retention data,” repositorio de código y datos. [En línea]. Disponible: https://github.com/michaelwitting/RepoRT

[2] F. Pedregosa *et al*., “Scikit-learn: Machine Learning in Python,” *Journal of Machine Learning Research*, vol. 12, pp. 2825–2830, 2011.

[3] L. van der Maaten y G. Hinton, “Visualizing Data using t-SNE,” *Journal of Machine Learning Research*, vol. 9, pp. 2579–2605, 2008.

[4] F. Wilcoxon, “Individual Comparisons by Ranking Methods,” *Biometrics Bulletin*, vol. 1, no. 6, pp. 80–83, 1945.

[5] Scikit-learn developers, “Pipeline,” “RepeatedKFold,” “RandomizedSearchCV,” “validation_curve,” “Ridge,” y “RandomForestRegressor,” documentación de scikit-learn. [En línea]. Disponible: https://scikit-learn.org/stable/

[6] USFQ, “Proyecto ML,” rúbrica e ítems del proyecto final de Machine Learning. [En línea]. Disponible: https://pointy-whistle-142.notion.site/Proyecto-ML-734cbbc3faae41c9a3f2425105baab07

# Taller 2 · Predicción de abandono de clientes (*churn*) – FinAlpes

**MINE-4101 Ciencia de Datos Aplicada · Universidad de los Andes · 2026-20**

**Integrantes:** _[Nombre 1]_ · _[Nombre 2]_

Prueba de concepto de una herramienta de predicción de abandono temprano de clientes bancarios para campañas de retención. Se entrenan y comparan una **regresión logística regularizada** y un **árbol de decisión**, y se interpretan con coeficientes, reglas del árbol, SHAP, LIME e ICE.

## Resultados principales

| | |
|---|---|
| Modelo elegido | Regresión logística regularizada (L2, C = 1, `class_weight="balanced"`) |
| Desempeño en prueba | AUC 0,84 · AP 0,66 · recall 75 % · precisión 45 % · F1 0,56 (umbral 0,5) |
| Árbol de decisión (validación) | AUC 0,83 · AP 0,60 · F1 0,55 (profundidad 5, 29 hojas) |
| Factores de abandono | Número de productos (1 o 3–4), edad (pico a los 50–60 años), inactividad, país (Alemania) y género |

## Estructura del repositorio

```
taller_2/
├── README.md
├── requirements.txt
├── data/
│   ├── churn.csv                          # dataset original (10.045 registros)
│   └── processed/                         # generado por los notebooks
│       ├── churn_clean.csv                #   01 → datos tratados
│       └── predicciones.csv               #   02 → probabilidades de ambos modelos
├── notebooks/
│   ├── 01_entendimiento_preparacion.ipynb # Entregable 1
│   └── 02_modelado_interpretacion.ipynb   # Entregables 2, 3 y 4
└── reports/
    └── figures/                           # figuras exportadas por los notebooks
```

| Entregable | Peso | Dónde |
|---|---|---|
| 1. Entendimiento y preparación de los datos | 10 % | [01_entendimiento_preparacion.ipynb](notebooks/01_entendimiento_preparacion.ipynb) |
| 2. Entrenamiento de modelos | 15 % | [02_modelado_interpretacion.ipynb](notebooks/02_modelado_interpretacion.ipynb), secciones 1–4 |
| 3. Análisis cuantitativo | 20 % | [02_modelado_interpretacion.ipynb](notebooks/02_modelado_interpretacion.ipynb), secciones 5–7 |
| 4. Análisis cualitativo (interpretación) | 20 % | [02_modelado_interpretacion.ipynb](notebooks/02_modelado_interpretacion.ipynb), secciones 8–10 |
| 5. Generación de valor | 20 % | _[pendiente]_ |
| 6. Insights | 10 % | _[pendiente]_ |
| 7. Declaración de uso de IA generativa | 5 % | [Al final de este README](#declaración-de-uso-de-ia-generativa) |

## Cómo ejecutar

Requiere Python 3.12.

```bash
python3 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

**Los notebooks deben ejecutarse en orden (01 → 02)**, porque el 02 lee los datos tratados que guarda el 01 (`data/processed/churn_clean.csv`):

```bash
cd notebooks
jupyter nbconvert --to notebook --execute --inplace 01_entendimiento_preparacion.ipynb
jupyter nbconvert --to notebook --execute --inplace 02_modelado_interpretacion.ipynb
```

También se pueden abrir en Jupyter o VS Code y ejecutar con *Run All*. Todos los procesos aleatorios usan `random_state=42`. El notebook 02 tarda unos minutos por la búsqueda de hiperparámetros y las explicaciones SHAP/LIME.

## Metodología (resumen)

1. **Datos:** se unificaron categorías mal escritas, se eliminaron 45 duplicados y los valores imposibles (edad 210, *credit score* 1.200…) se convirtieron en nulos. La imputación se hace con la mediana dentro del *pipeline*, ajustada solo con el entrenamiento. Se seleccionaron 7 variables con gráficos y pruebas estadísticas (t de Welch, U de Mann-Whitney, χ² y V de Cramér) y se creó la variable derivada `age_sq`.
2. **Modelado:** división estratificada 60/20/20. `GridSearchCV` con validación cruzada estratificada de 5 *folds* optimizando F1. Regresión logística con penalización L1, L2 o *elastic net* (`C`, `l1_ratio`, `class_weight`). Árbol de decisión (`max_depth`, `min_samples_leaf`, `criterion`, `class_weight`) seleccionado con la regla de un error estándar.
3. **Evaluación:** accuracy, precisión, recall, F1, AUC-ROC y AP; curvas PR y ROC; prueba de McNemar; intervalos de confianza por *bootstrap*; ablación de las decisiones de ingeniería de características; desempeño por segmento.
4. **Interpretación:** coeficientes y razones de momios, reglas del árbol, importancia por permutación, SHAP (global y local), ICE/PDP y LIME sobre 5 clientes (VP, FP, FN, VN y un caso frontera).

Metodología de referencia del curso: [Prácticas de Machine Learning – Uniandes](https://datascience-uniandes.github.io/PracticasMachineLearning/).
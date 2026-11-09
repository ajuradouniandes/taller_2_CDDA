# Taller 2 · Predicción de abandono de clientes (*churn*) – FinAlpes

**MINE-4101 Ciencia de Datos Aplicada · Universidad de los Andes · 2026-20**

**Integrantes:** Mateo Jurado · Iván Herrera

Prueba de concepto de una herramienta de predicción de abandono temprano de clientes bancarios para campañas de retención. Se entrenan y comparan una **regresión logística regularizada** y un **árbol de decisión**, y se interpretan con coeficientes, reglas del árbol, SHAP, LIME e ICE.

## Resultados principales

| | |
|---|---|
| Modelo elegido | Regresión logística regularizada (L2, C = 1, `class_weight="balanced"`) |
| Desempeño en prueba | AUC 0,84 · AP 0,66 · recall 75 % · precisión 45 % · F1 0,56 (umbral 0,5) |
| Árbol de decisión (validación) | AUC 0,83 · AP 0,60 · F1 0,55 (profundidad 5, 29 hojas) |
| Factores de abandono | Número de productos (1 o 3–4), edad (pico a los 50–60 años), inactividad, país (Alemania) y género |
| Valor para un banco de 20.000 clientes (15 % de abandono) | Ganancia de la campaña ≈ COP 242 M/año (contactar a todos pierde COP 155 M) · beneficio neto ≈ COP 200 M/año |
| Punto de equilibrio | Inversión de ≈ COP 102 M recuperada en 6,2 meses · rentable en el 99 % de 5.000 escenarios simulados |

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
│   ├── 02_modelado_interpretacion.ipynb   # Entregables 2, 3 y 4
│   └── 03_generacion_valor.ipynb          # Entregables 5 y 6 (sección 8)
└── reports/
    ├── informe_ejecutivo.md               # Entregable 6: informe para la gerencia
    ├── figures/                           # figuras exportadas por los notebooks
    └── resumen_valor.csv                  #   03 → indicadores clave de valor
```

| Entregable | Peso | Dónde |
|---|---|---|
| 1. Entendimiento y preparación de los datos | 10 % | [01_entendimiento_preparacion.ipynb](notebooks/01_entendimiento_preparacion.ipynb) |
| 2. Entrenamiento de modelos | 15 % | [02_modelado_interpretacion.ipynb](notebooks/02_modelado_interpretacion.ipynb), secciones 1–4 |
| 3. Análisis cuantitativo | 20 % | [02_modelado_interpretacion.ipynb](notebooks/02_modelado_interpretacion.ipynb), secciones 5–7 |
| 4. Análisis cualitativo (interpretación) | 20 % | [02_modelado_interpretacion.ipynb](notebooks/02_modelado_interpretacion.ipynb), secciones 8–10 |
| 5. Generación de valor | 20 % | [03_generacion_valor.ipynb](notebooks/03_generacion_valor.ipynb), secciones 1–7 |
| 6. Insights | 10 % | [Informe ejecutivo](reports/informe_ejecutivo.md) · [03_generacion_valor.ipynb](notebooks/03_generacion_valor.ipynb), sección 8 |
| 7. Declaración de uso de IA generativa | 5 % | [Al final de este README](#declaración-de-uso-de-ia-generativa) |

## Cómo ejecutar

Requiere Python 3.12.

```bash
python3 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

**Los notebooks deben ejecutarse en orden (01 → 02 → 03)**: el 02 lee los datos tratados que guarda el 01 (`data/processed/churn_clean.csv`) y el 03 lee las probabilidades que guarda el 02 (`data/processed/predicciones.csv`):

```bash
cd notebooks
jupyter nbconvert --to notebook --execute --inplace 01_entendimiento_preparacion.ipynb
jupyter nbconvert --to notebook --execute --inplace 02_modelado_interpretacion.ipynb
jupyter nbconvert --to notebook --execute --inplace 03_generacion_valor.ipynb
```

También se pueden abrir en Jupyter o VS Code y ejecutar con *Run All*. Todos los procesos aleatorios usan `random_state=42`. El notebook 02 tarda unos minutos por la búsqueda de hiperparámetros y las explicaciones SHAP/LIME.

## Metodología (resumen)

1. **Datos:** se unificaron categorías mal escritas, se eliminaron 45 duplicados y los valores imposibles (edad 210, *credit score* 1.200…) se convirtieron en nulos. La imputación se hace con la mediana dentro del *pipeline*, ajustada solo con el entrenamiento. Se seleccionaron 7 variables con gráficos y pruebas estadísticas (t de Welch, U de Mann-Whitney, χ² y V de Cramér) y se creó la variable derivada `age_sq`.
2. **Modelado:** división estratificada 60/20/20. `GridSearchCV` con validación cruzada estratificada de 5 *folds* optimizando F1. Regresión logística con penalización L1, L2 o *elastic net* (`C`, `l1_ratio`, `class_weight`). Árbol de decisión (`max_depth`, `min_samples_leaf`, `criterion`, `class_weight`) seleccionado con la regla de un error estándar.
3. **Evaluación:** accuracy, precisión, recall, F1, AUC-ROC y AP; curvas PR y ROC; prueba de McNemar; intervalos de confianza por *bootstrap*; ablación de las decisiones de ingeniería de características; desempeño por segmento.
4. **Interpretación:** coeficientes y razones de momios, reglas del árbol, importancia por permutación, SHAP (global y local), ICE/PDP y LIME sobre 5 clientes (VP, FP, FN, VN y un caso frontera).
5. **Generación de valor:** las tasas de verdaderos y falsos positivos del modelo se trasladan a un banco de 20.000 clientes con 15 % de abandono. Ganancia = clientes retenidos × ingreso − contactos × costo por contacto. El umbral que maximiza la ganancia se elige en validación y se mide en prueba. Las probabilidades se recalibran (corrección de *odds* por los pesos balanceados) para calcular el valor esperado de cada cliente. Se calcula el punto de equilibrio en meses, tamaño del banco y cada supuesto, con análisis de tornado, mapa ingreso × costo y Monte Carlo (5.000 escenarios con *bootstrap* del conjunto de prueba). Los supuestos y sus fuentes están en la sección 1 del notebook 03.

Metodología de referencia del curso: [Prácticas de Machine Learning – Uniandes](https://datascience-uniandes.github.io/PracticasMachineLearning/).

## Declaración de uso de IA generativa

Durante el desarrollo de este taller usamos herramientas de inteligencia artificial generativa como apoyo en todas las etapas del proyecto. La IA nos ayudó a entender y preparar los datos, a proponer y depurar código en Python (pandas, scikit-learn, SHAP y LIME), a interpretar métricas y explicaciones del modelo, a buscar fuentes para los supuestos de costos del análisis de generación de valor y a revisar la redacción del informe ejecutivo y de este README. Todas las respuestas fueron revisadas, ejecutadas y validadas por los integrantes del equipo. Las decisiones metodológicas, la selección del modelo final, la interpretación de los resultados y las conclusiones son responsabilidad nuestra.

| | |
|---|---|
| Modelo(s) usado(s) | Claude Opus 5.5 y Sonnet 5.5 mediante Claude Code |
| Porcentaje de ayuda | 65% - 70% |
| Para qué se usó | _generación de código, redacción, búsqueda de fuentes para los supuestos de costos_ |

### Prompts empleados

#### 1. Entendimiento y preparación de datos

- _Tengo un dataset de churn bancario con estas columnas:.... ¿Qué tipo de variable es cada una (numérica continua, discreta, categórica nominal u ordinal) y cuáles debería descartar, como IDs o apellidos?_
- _Escribe código en Python con pandas y seaborn para graficar la distribución de las variables numéricas, separadas por la variable objetivo Exited._
- _Interpreta estas estadísticas descriptivas y dime qué indicadores vale la pena destacar en un reporte de entendimiento de datos:..._
- _La tasa de churn de mi dataset es del 20,4 %. ¿Esto se considera desbalance? ¿Qué implica para el modelado y para las métricas?_
- _¿Cómo reviso faltantes, duplicados y outliers en este dataset? Dame el código y explícame cómo decidir qué hacer con cada caso._

#### 2. Entrenamiento de modelos

- _Propón un pipeline de scikit-learn con ColumnTransformer que haga one-hot a Geography y Gender y escale las numéricas, sin fuga de datos._
- _¿Cómo hago una división train/validation/test estratificada y la documento correctamente en un notebook?_
- _Quiero comparar una regresión logística, un Random Forest y un XGBoost. ¿Qué hiperparámetros conviene explorar en cada uno con GridSearchCV o RandomizedSearchCV, y en qué rangos?_
- _¿Debo usar class_weight, SMOTE o ajustar el umbral para el desbalance? Explícame las ventajas y desventajas de cada opción._
- _Me sale este error al ejecutar el GridSearch: ... ¿Qué está pasando?_

#### 3. Análisis cuantitativo

- _Explícame en términos de negocio qué significan precision, recall, F1 y AUC-ROC en un problema de retención de clientes._
- _Estos son mis resultados para los tres modelos: ... Ayúdame a justificar cuál elegir, considerando que un falso negativo es un cliente perdido y un falso positivo es una oferta pagada de más._
- _¿Cómo grafico la matriz de confusión, la curva ROC y la curva precision-recall de los modelos comparados?_
- _¿Cómo elijo el umbral de clasificación óptimo cuando los costos de FP y FN son distintos?_
- _Revisa mi análisis de métricas y sugiéreme oportunidades de mejora del modelo: ..._

#### 4. Análisis cualitativo / interpretabilidad

- _¿Cómo uso SHAP con un modelo XGBoost dentro de un pipeline de sklearn? Dame código para el summary plot y para waterfall plots individuales._
- _Interpreta este gráfico SHAP global: ... ¿Qué nos dice sobre Age, NumOfProducts e IsActiveMember?_
- _Elegí cuatro clientes: un verdadero positivo, un falso positivo, un falso negativo y un verdadero negativo. Ayúdame a explicar por qué el modelo predijo lo que predijo en cada caso a partir de sus valores SHAP._
- _¿Qué diferencia hay entre feature importance por impureza, permutation importance y SHAP? ¿Cuál es más confiable?_

#### 5. Generación de valor

- _Ayúdame a plantear la fórmula de ganancia esperada de una campaña de retención: base de 20.000 clientes, churn del 15 %, efectividad del 30 % sobre los verdaderos churners, costo por contacto C e ingreso anual por cliente entre COP 500.000 y 900.000._
- _¿Qué fuentes recientes existen sobre el costo por cliente contactado en campañas de retención bancaria en Colombia o Latinoamérica (SMS, call center, bonos)?_
- _¿Cómo estimo el costo de implementación de un modelo de ML en un banco mediano (desarrollo, infraestructura, mantenimiento)? Dame rangos con fuentes._
- _Escribe código que calcule la ganancia neta según el umbral del modelo y encuentre el break-even point, con un análisis de sensibilidad sobre el costo por contacto y el ingreso por cliente._
- _¿Qué limitaciones tiene aplicar un modelo entrenado con clientes europeos a un banco colombiano? Piensa en el dataset shift y en la diferencia entre el 20 % de churn del dataset y el 15 % del mercado._

#### 6. Insights / informe ejecutivo

- _Con estos resultados ..., ayúdame a estructurar un informe ejecutivo de una página para la gerencia de un banco: hallazgos, valor estimado y recomendación._
- _Corrige el estilo y la redacción de este informe sin cambiar las cifras ni las conclusiones:..._

#### 7. Entrega

- _Redáctame un README para el repositorio de GitHub que explique la estructura, el orden de ejecución de los notebooks y cómo instalar las dependencias._
- _Ayúdame a redactar la declaración de uso de IA generativa con el modelo, el porcentaje aproximado de ayuda por sección y los prompts usados._

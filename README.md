# fifa-wc-predictor-crisp-dm

**Descripcion del repositorio:**
> Modelo de machine learning para predecir resultados de partidos del FIFA World Cup usando la metodologia CRISP-DM. Incluye ingenieria de features historicas, comparacion de 4 algoritmos de clasificacion y simulacion Monte Carlo para pronosticar el TOP 4 del Mundial 2026.

> Se usaron datos desde 1872 hasta 2026
---

## Tabla de contenidos

1. [Descripcion general](#1-descripcion-general)
2. [Stack tecnologico](#2-stack-tecnologico)
3. [Estructura del repositorio](#3-estructura-del-repositorio)
4. [Proceso CRISP-DM paso a paso](#4-proceso-crisp-dm-paso-a-paso)
5. [Seleccion y justificacion de variables](#5-seleccion-y-justificacion-de-variables)
6. [Modelos evaluados](#6-modelos-evaluados)
7. [Simulacion Monte Carlo TOP 4](#7-simulacion-monte-carlo-top-4)
8. [Como ejecutar el proyecto](#8-como-ejecutar-el-proyecto)
9. [Salidas para Power BI](#9-salidas-para-power-bi)
10. [Resultados y prediccion](#10-resultados-y-prediccion)
11. [Limitaciones y mejoras futuras](#11-limitaciones-y-mejoras-futuras)

---

## 1. Descripcion general

Este proyecto implementa un pipeline completo de ciencia de datos siguiendo la metodologia **CRISP-DM** (Cross-Industry Standard Process for Data Mining) para predecir el resultado de partidos de futbol internacional, con enfasis en el **FIFA World Cup 2026**.

El objetivo central es predecir si el equipo local gana, empata o pierde un partido dado, y usar esas probabilidades para simular el torneo completo mediante **Monte Carlo** y obtener un ranking probabilistico del TOP 4.

**Variable objetivo:** `resultado` (clasificacion multiclase)
- `0` = Derrota del equipo local
- `1` = Empate
- `2` = Victoria del equipo local

**Fuente de datos primaria:**
- Dataset publico de resultados historicos internacionales: [martj42/international_results](https://github.com/martj42/international_results)
- ~47,000 partidos internacionales desde 1872 hasta la actualidad

---

## 2. Stack tecnologico

| Categoria | Tecnologia |
|-----------|-----------|
| Lenguaje | Python 3.10+ |
| Notebook | Jupyter Notebook |
| Manipulacion de datos | pandas, numpy |
| Machine Learning | scikit-learn, xgboost |
| Visualizacion | matplotlib, seaborn |
| Export | openpyxl (Excel para Power BI) |
| Simulacion | Monte Carlo (numpy) |

---

## 3. Estructura del repositorio

```
fifa-wc-predictor-crisp-dm/
|
|-- fifa_prediccion_crisp_dm.ipynb   # Notebook principal (CRISP-DM completo + TOP 4)
|-- mundial_2026_top4.ipynb          # Notebook auxiliar de simulacion TOP 4
|-- add_top4_cells.py                # Script para agregar celdas al notebook
|
|-- output_powerbi/
|   |-- fifa_predicciones_powerbi.xlsx   # Dataset final (4 hojas para Power BI)
|   |-- todos_partidos.csv
|   |-- world_cup.csv
|   |-- modelos.csv
|   |-- importancia.csv
|   |-- top4_modelo_ml.xlsx
|   |-- confusion_matrix.png
|   |-- feature_importance.png
|   |-- top4_crisp_dm.png
|
|-- README.md
```

---

## 4. Proceso CRISP-DM paso a paso

### Fase 1 — Business Understanding

Se define el problema como una clasificacion multiclase. La pregunta de negocio es:

> "Dado el historial de dos selecciones, su forma reciente y su historial de enfrentamientos, ?cual es la probabilidad de que el equipo local gane, empate o pierda?"

Las metricas de exito se establecieron en:
- Accuracy >= 55% en el conjunto de prueba
- F1-score macro >= 0.50
- Comparacion de al menos 4 algoritmos

El entregable final es un dataset enriquecido con predicciones exportable a Power BI.

---

### Fase 2 — Data Understanding

Se descarga automaticamente el dataset desde GitHub usando `requests`. Si no hay conexion a internet, el sistema usa un dataset de respaldo generado con estructura identica para garantizar que el pipeline corra end-to-end.

Se realiza un analisis exploratorio que incluye:
- Inspeccion de tipos de datos y valores nulos
- Distribucion de resultados (Victoria / Empate / Derrota) global y filtrado por torneo
- Promedio de goles por torneo
- Evolucion de goles por anno en el World Cup

**Hallazgo clave del EDA:** En toda la historia de partidos internacionales, el equipo local gana aproximadamente el 45-50% de los partidos, empata el 22% y pierde el 28-32%. En el World Cup en particular, la diferencia se reduce porque muchos partidos son en campo neutral.

---

### Fase 3 — Data Preparation

Esta fase es la mas critica del proyecto. Se construyen 20 variables predictoras mediante **feature engineering temporal**, asegurando que no haya data leakage: cada feature se calcula usando unicamente partidos **anteriores** a la fecha del partido que se predice.

El detalle completo de las variables se encuentra en la seccion 5 de este documento.

**Division temporal train/test:**
Se utiliza una division temporal (no aleatoria) donde el 80% de los registros mas antiguos forman el conjunto de entrenamiento y el 20% mas reciente forma el conjunto de prueba. Esto simula un escenario realista donde el modelo se entrena con datos pasados y predice eventos futuros.

```python
n_train = int(n_total * 0.80)
train = df[:n_train]   # datos historicos
test  = df[n_train:]   # datos mas recientes
```

---

### Fase 4 — Modeling

Se entrenan 4 algoritmos con validacion cruzada estratificada de 5 folds:

1. **Regresion Logistica** — modelo lineal de referencia (baseline)
2. **Random Forest** — ensemble de arboles de decision con bagging
3. **Gradient Boosting** — ensemble secuencial con correccion de errores
4. **XGBoost** — implementacion optimizada de Gradient Boosting

Para la Regresion Logistica se aplica estandarizacion (`StandardScaler`). Los modelos basados en arboles no requieren escalar.

---

### Fase 5 — Evaluation

Se evaluan los modelos con:
- **Cross-validation accuracy** (5-fold) en el conjunto de entrenamiento
- **Test accuracy** en el 20% de datos mas recientes
- **F1-score macro** para penalizar el desbalance de clases
- **Matriz de confusion** para analizar errores por clase
- **Importancia de variables** (para modelos tree-based)

El mejor modelo se selecciona automaticamente por test accuracy.

---

### Fase 6 — Deployment

Se generan los siguientes archivos listos para consumir en Power BI:

| Archivo / Hoja | Contenido |
|----------------|-----------|
| `Todos_Partidos` | Todos los partidos con prediccion, resultado real y probabilidades |
| `World_Cup` | Filtrado solo por FIFA World Cup |
| `Evaluacion_Modelos` | Tabla comparativa de los 4 modelos |
| `Importancia_Vars` | Ranking de importancia de features |

---

### Bonus — Simulacion TOP 4 Mundial 2026

Usando el modelo ya entrenado, se calcula la forma reciente de cada seleccion semifinalista directamente desde `df` (sin hardcodear valores), se simula el bracket del torneo 5,000 veces y se obtiene un ranking probabilistico del TOP 4.

---

## 5. Seleccion y justificacion de variables

La seleccion de variables responde a tres principios:
1. **Causalidad plausible:** solo se incluyen variables que, teoricamente, pueden influir en el resultado de un partido de futbol.
2. **Sin data leakage:** todas las features se computan con datos anteriores a la fecha del partido.
3. **Disponibilidad:** los datos deben poder calcularse desde el dataset historico publico sin fuentes externas.

Se descartaron variables como el resultado exacto del partido anterior, los nombres de jugadores o los valores de mercado porque no estan en el dataset y agregan complejidad sin garantia de mejora.

### Variables de forma reciente del equipo local (ventana = 5 partidos)

| Variable | Descripcion | Justificacion |
|----------|-------------|---------------|
| `home_pts_f` | Puntos acumulados en los ultimos 5 partidos (3=V, 1=E, 0=D) | El momentum reciente es el predictor mas directo de rendimiento futuro. Un equipo con 13-15 puntos en los ultimos 5 partidos esta en forma optima. |
| `home_gf_f` | Goles anotados en los ultimos 5 partidos | Mide la eficacia ofensiva reciente. Equipos que no anotan tienen menor probabilidad de ganar independientemente de su defensa. |
| `home_gc_f` | Goles recibidos en los ultimos 5 partidos | Mide la solidez defensiva reciente. Correlaciona negativamente con la probabilidad de victoria. |
| `home_dif_f` | Diferencia de goles en los ultimos 5 partidos (`gf - gc`) | Sintetiza ataque y defensa en una sola cifra. Es mas informativa que cada componente por separado en modelos lineales. |

### Variables de forma reciente del equipo visitante (misma logica)

| Variable | Descripcion |
|----------|-------------|
| `away_pts_f` | Puntos del visitante en ultimos 5 partidos |
| `away_gf_f` | Goles a favor del visitante |
| `away_gc_f` | Goles en contra del visitante |
| `away_dif_f` | Diferencia de goles del visitante |

### Variables diferenciales (local minus visitante)

Estas variables capturan la **brecha de rendimiento** entre los dos equipos, que es el concepto clave para predecir el ganador.

| Variable | Formula | Justificacion |
|----------|---------|---------------|
| `dif_pts` | `home_pts_f - away_pts_f` | Un diferencial positivo grande indica al local como claro favorito. Es la variable con mayor importancia en la mayoria de modelos tree-based. |
| `dif_gf` | `home_gf_f - away_gf_f` | Brecha ofensiva entre equipos. |
| `dif_gc` | `home_gc_f - away_gc_f` | Brecha defensiva. Un valor negativo indica que el local recibe mas goles que el visitante. |

### Variables de historial directo (Head-to-Head)

| Variable | Descripcion | Justificacion |
|----------|-------------|---------------|
| `h2h_v` | Victorias del local en los ultimos 10 enfrentamientos directos | El historial entre dos selecciones especificas captura factores psicologicos y tacticos que no se reflejan en la forma general. Brasil vs Argentina tiene una dinamica distinta a Brasil vs Camerun. |
| `h2h_e` | Empates en los ultimos 10 enfrentamientos | Algunos duelos son historicamente muy parejos (muchos empates). |
| `h2h_d` | Derrotas del local en los ultimos 10 enfrentamientos | Complementa `h2h_v`. Los tres sumados siempre dan el total de partidos H2H considerados. |

### Variables de contexto del partido

| Variable | Valores | Justificacion |
|----------|---------|---------------|
| `neutral` | 0 = campo del local / 1 = campo neutral | En el World Cup casi todos los partidos son en sedes neutras (el torneo se juega en paises anfitriones). La ventaja de campo desaparece en terreno neutral, lo que redistribuye las probabilidades. |
| `es_wc` | 0 = otro torneo / 1 = FIFA World Cup | Los partidos del Mundial tienen mayor presion y nivel competitivo. Los equipos tienden a ser mas conservadores, lo que aumenta la tasa de empates. |
| `anio` | Entero (ej: 2026) | Captura la evolucion del nivel competitivo global a traves del tiempo. El futbol actual es mas parejo que el de los anos 60-70. |
| `mes` | 1-12 | Proxy del calendario: los partidos amistosos (enero-marzo) difieren en intensidad de los partidos de competicion (junio-julio). |

### Variables de identidad del equipo

| Variable | Descripcion | Justificacion |
|----------|-------------|---------------|
| `home_enc` | Codigo numerico del equipo local (LabelEncoder) | Permite al modelo aprender patrones especificos por seleccion. Brasil tiene tasas de victoria historicamente distintas a Liechtenstein, aunque su forma reciente sea similar. |
| `away_enc` | Codigo numerico del equipo visitante | Idem para el equipo visitante. |

### Variables descartadas y por que

| Variable considerada | Razon de descarte |
|---------------------|-------------------|
| Ranking FIFA numerico | No disponible directamente en el dataset publico base |
| Valor de mercado del equipo | Requiere fuente externa (Transfermarkt); agrega dependencia |
| Goleador individual | Lesiones y cambios de alineacion invalidan la variable antes del partido |
| Resultado del ultimo partido | Ya capturado de forma mas robusta por `home_pts_f` con ventana de 5 |
| Estadisticas avanzadas (xG, pases) | No disponibles en datasets publicos historicos pre-2015 |

---

## 6. Modelos evaluados

### Por que estos 4 algoritmos

Se eligio una combinacion que cubre el espectro de complejidad:

**Regresion Logistica** — Modelo lineal que sirve como **baseline**. Si los modelos complejos no superan a la regresion logistica, indica que la relacion entre features y resultado es aproximadamente lineal o que las features no son suficientemente informativas.

**Random Forest** — Ensemble de arboles que maneja bien la no-linealidad y las interacciones entre variables. Robusto al sobreajuste gracias al bagging. Produce importancia de variables directamente.

**Gradient Boosting (sklearn)** — Aprende de los errores del modelo anterior de forma secuencial. Generalmente supera a Random Forest en datasets tabulares con suficientes datos. Mas lento de entrenar pero mas preciso.

**XGBoost** — Version optimizada de Gradient Boosting con regularizacion L1/L2 integrada, manejo nativo de valores nulos y paralelizacion. Es el estandar de la industria para clasificacion tabular y frecuentemente gana competencias de datos estructurados.

### Estrategia de validacion

Se usa `StratifiedKFold(n_splits=5)` para mantener la proporcion de clases en cada fold, lo cual es critico porque la clase "Empate" es la menos frecuente (~22%).

---

## 7. Simulacion Monte Carlo TOP 4

### Por que Monte Carlo y no una prediccion directa

Un torneo eliminatorio tiene dependencias entre partidos: el rival en semifinales depende de quien gane los cuartos. Propagar estas incertidumbres de forma analitica es complejo. Monte Carlo resuelve esto simulando el torneo completo N veces (5,000 en este proyecto) y contando frecuencias.

### Como se calculan las features en la simulacion

**Punto clave:** los valores de forma (`pts_f`, `gf_f`, `gc_f`, `dif_f`) y el historial H2H se extraen directamente desde `df` usando las mismas funciones `calcular_forma()` y `calcular_h2h()` definidas en la Fase 3. No se hardcodea ningun valor.

```python
FORMA = {}
for equipo in EQUIPOS_WC:
    FORMA[equipo] = calcular_forma(df, equipo, fecha_ref, ventana=5)
```

Para cada partido simulado, el H2H tambien se recalcula desde `df`:
```python
def features_partido(local, visitante):
    h2h = calcular_h2h(df, local, visitante, fecha_ref)
    ...
```

### Manejo de empates en eliminatoria

El modelo predice tres clases: Victoria, Empate, Derrota. En fase eliminatoria no hay empate en tiempo reglamentario (la UEFA/FIFA resuelve con prorroga y penales). Se modela asi:

```
P(local avanza) = P(Victoria) + P(Empate) * 0.50
```

Los penales se modelan como un evento aleatorio 50/50, lo cual es una aproximacion razonable dado que historicamente la tasa de avance en penales es aproximadamente igual para ambos equipos.

### Bracket del Mundial 2026 (cuartos de final en adelante)

```
QF1 (9-jul):  Francia 2-0 Marruecos       [CONFIRMADO]
QF2 (10-jul): Espana 2-1 Belgica           [CONFIRMADO]
QF3 (11-jul): Noruega vs Inglaterra        [SIMULADO]
QF4 (11-jul): Argentina vs Suiza           [SIMULADO]

Semi 1 (14-jul): Francia vs Espana
Semi 2 (15-jul): Ganador QF3 vs Ganador QF4

3er lugar (18-jul): Perdedor Semi 1 vs Perdedor Semi 2
Final (19-jul): Ganador Semi 1 vs Ganador Semi 2
```

---

## 8. Como ejecutar el proyecto

### Requisitos

```bash
pip install pandas numpy scikit-learn xgboost matplotlib seaborn requests openpyxl jupyter
```

### Ejecucion

```bash
jupyter notebook fifa_prediccion_crisp_dm.ipynb
```

Ejecutar las celdas en orden de arriba a abajo. El notebook descarga automaticamente el dataset desde GitHub. Si no hay conexion, usa datos de respaldo y el pipeline completo sigue funcionando.

### Resultado esperado

Al finalizar la ejecucion completa del notebook:
1. Se imprime la tabla comparativa de los 4 modelos
2. Se muestra la matriz de confusion del mejor modelo
3. Se grafica la importancia de variables
4. Se ejecuta la simulacion TOP 4
5. Se exporta `output_powerbi/fifa_predicciones_powerbi.xlsx`

---

## 9. Salidas para Power BI

Importar el archivo `output_powerbi/fifa_predicciones_powerbi.xlsx` en Power BI Desktop (Obtener datos > Excel).

### Visualizaciones recomendadas

| Visual | Tabla | Campos |
|--------|-------|--------|
| Tarjeta: Accuracy del modelo | `Evaluacion_Modelos` | `Test_Accuracy` filtrado por `Es_Mejor = True` |
| Grafico de lineas: predicciones correctas por anno | `Todos_Partidos` | Año en eje X, promedio de `Prediccion_Correcta` en Y |
| Grafico de barras: distribucion de predicciones | `Todos_Partidos` | `Prediccion` en eje X, COUNT en Y |
| Tabla con barras de datos: probabilidades por partido | `World_Cup` | `Prob_Victoria`, `Prob_Empate`, `Prob_Derrota` |
| Grafico de barras horizontal: importancia de variables | `Importancia_Vars` | `Variable` en Y, `Importancia` en X |

---

## 10. Resultados y prediccion

### Prediccion TOP 4 Mundial 2026

Basado en 5,000 simulaciones Monte Carlo con el modelo entrenado:

| Posicion | Seleccion | Prob. de campeonato |
|----------|-----------|---------------------|
| Campeon | Argentina | ~31% |
| Subcampeon | Francia | ~32% |
| 3er lugar | Espana | ~24% |
| 4to lugar | Inglaterra / Noruega | ~13% |

> Nota: los resultados exactos dependen del dataset descargado. Con el dataset real de 47,000 partidos las probabilidades seran mas precisas que con el dataset de respaldo.

---

## 11. Limitaciones y mejoras futuras

### Limitaciones actuales

- El dataset base no incluye estadisticas avanzadas (xG, posesion, tiros a puerta).
- Los rankings FIFA no se integran directamente por falta de una fuente publica estructurada.
- El modelo trata todos los partidos del mismo torneo de igual forma, sin distinguir grupos de equipos muy diferentes en nivel.
- La ventana de 5 partidos es fija; no se optimizo mediante validacion cruzada.

### Mejoras propuestas

- Integrar datos de rankings FIFA historicos como feature adicional.
- Usar una ventana de forma variable (3, 5, 10 partidos) y seleccionarla por validacion cruzada.
- Incorporar estadisticas avanzadas de fuentes como StatsBomb o Opta para partidos recientes.
- Implementar un sistema de ratings Elo dinamico en paralelo al modelo ML.
- Agregar un modulo de calibracion de probabilidades (`CalibratedClassifierCV`) para que las probabilidades del modelo sean mas confiables.
- Extender la simulacion para cubrir el torneo completo desde fase de grupos (48 equipos).

---

## Creditos

- Dataset: [martj42/international_results](https://github.com/martj42/international_results)
- Metodologia: [CRISP-DM Wikipedia](https://en.wikipedia.org/wiki/Cross-industry_standard_process_for_data_mining)
- Datos del torneo 2026: FIFA.com, ESPN, CBS Sports

---

*Proyecto desarrollado con metodologia CRISP-DM | FIFA World Cup 2026*

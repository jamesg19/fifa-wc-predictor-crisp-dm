# Presentacion del proyecto FIFA WC Predictor

## 1. Proposito del repositorio

Este repositorio contiene un proyecto de ciencia de datos para predecir resultados de partidos internacionales de futbol, con enfasis en el FIFA World Cup. El flujo principal esta implementado en el notebook `fifa_prediccion_crisp_dm.ipynb` y sigue la metodologia CRISP-DM: entendimiento del negocio, entendimiento de datos, preparacion, modelado, evaluacion y despliegue.

El problema se plantea como una clasificacion multiclase del resultado del equipo local:

| Clase | Significado |
|-------|-------------|
| `0` | Derrota del equipo local |
| `1` | Empate |
| `2` | Victoria del equipo local |

El entregable practico del proyecto es un conjunto de predicciones, metricas, graficos y archivos Excel listos para analizar en Power BI.

## 2. Estructura real del proyecto

```text
fifa-wc-predictor-crisp-dm/
├── README.md
├── requirements.txt
├── fifa_prediccion_crisp_dm.ipynb
├── docs/
│   └── presentacion.md
└── output_powerbi/
    ├── confusion_matrix.png
    ├── eda_distribucion.png
    ├── evaluacion_modelos.png
    ├── feature_importance.png
    ├── fifa_predicciones_powerbi.xlsx
    ├── top4_crisp_dm.png
    ├── top4_modelo_ml.xlsx
    └── wc_goles_anio.png
```

El notebook es el centro del repositorio. Los archivos de `output_powerbi/` son salidas generadas por el proceso: visualizaciones, evaluaciones, predicciones y resultados de simulacion.

## 3. Dependencias

Las dependencias necesarias quedaron declaradas en `requirements.txt`. El proyecto usa:

| Categoria | Librerias |
|-----------|-----------|
| Analisis y manipulacion | `pandas`, `numpy` |
| Descarga de datos | `requests` |
| Visualizacion | `matplotlib`, `seaborn` |
| Machine learning | `scikit-learn`, `xgboost` |
| Exportacion Excel | `openpyxl`, `jinja2` |
| Ejecucion interactiva | `jupyter`, `ipykernel` |

Instalacion recomendada:

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Luego se ejecuta el notebook:

```bash
jupyter notebook fifa_prediccion_crisp_dm.ipynb
```

## 4. Flujo general del notebook

### 4.1 Carga de librerias y configuracion

El notebook inicializa las librerias principales, define una semilla reproducible (`SEED = 42`) y configura el directorio `output_powerbi/` como destino de los archivos generados.

Tambien desactiva advertencias para mantener limpia la salida del notebook durante la ejecucion.

### 4.2 Obtencion de datos

La fuente principal es el dataset publico `martj42/international_results`, descargado desde GitHub con `requests`.

Si la descarga falla, el notebook genera un dataset de respaldo con la misma estructura basica. Esto permite ejecutar el pipeline completo aunque no exista conexion a internet.

Despues de cargar los datos, el proyecto filtra los partidos desde `2010-01-01` en adelante. Por eso, aunque la fuente historica contiene partidos desde el siglo XIX, el entrenamiento real del notebook actual se concentra en datos recientes.

### 4.3 Entendimiento de datos

En la fase de analisis exploratorio se revisan:

- tipos de datos;
- valores nulos;
- estadisticas descriptivas de goles;
- torneos mas frecuentes;
- distribucion de resultados;
- promedio de goles por torneo;
- comportamiento de goles por anio en FIFA World Cup.

De esta fase se generan graficos como `eda_distribucion.png` y `wc_goles_anio.png`.

## 5. Preparacion de datos

La variable objetivo `resultado` se calcula comparando goles del equipo local y visitante.

Luego se construyen variables predictoras mediante feature engineering temporal. El punto importante es que las variables se calculan usando partidos anteriores a la fecha del partido evaluado, reduciendo el riesgo de data leakage.

### 5.1 Forma reciente

La funcion `calcular_forma()` toma los ultimos 5 partidos de cada seleccion antes de una fecha de referencia y calcula:

| Variable | Descripcion |
|----------|-------------|
| `pts_f` | puntos acumulados |
| `gf_f` | goles a favor |
| `gc_f` | goles en contra |
| `dif_f` | diferencia de goles |

Estas variables se calculan para local y visitante.

### 5.2 Historial directo

La funcion `calcular_h2h()` revisa los ultimos 10 enfrentamientos directos entre los dos equipos antes de la fecha del partido. Produce:

| Variable | Descripcion |
|----------|-------------|
| `h2h_v` | victorias historicas del local contra ese rival |
| `h2h_e` | empates historicos |
| `h2h_d` | derrotas historicas del local contra ese rival |

### 5.3 Variables finales del modelo

El notebook entrena con 20 columnas predictoras:

```text
home_pts_f, home_gf_f, home_gc_f, home_dif_f,
away_pts_f, away_gf_f, away_gc_f, away_dif_f,
dif_pts, dif_gf, dif_gc,
h2h_v, h2h_e, h2h_d,
neutral, es_wc, anio, mes,
home_enc, away_enc
```

Ademas, codifica los nombres de equipos con `LabelEncoder` para convertirlos en variables numericas.

## 6. Entrenamiento y evaluacion

El dataset limpio se ordena por fecha y se divide temporalmente:

- 80% mas antiguo para entrenamiento;
- 20% mas reciente para prueba.

Esta division es adecuada para un problema deportivo temporal porque simula el escenario real: entrenar con el pasado y evaluar contra partidos posteriores.

El notebook compara cuatro modelos:

| Modelo | Rol en el proyecto |
|--------|--------------------|
| Regresion Logistica | baseline lineal |
| Random Forest | ensemble robusto de arboles |
| Gradient Boosting | modelo secuencial de boosting |
| XGBoost | boosting optimizado para datos tabulares |

La Regresion Logistica usa `StandardScaler`. Los modelos basados en arboles usan los valores originales.

La evaluacion incluye:

- validacion cruzada estratificada de 5 folds;
- accuracy en test;
- F1 macro;
- reporte de clasificacion;
- matriz de confusion;
- comparacion visual de modelos;
- importancia de variables cuando el mejor modelo la soporta.

Los graficos principales generados en esta etapa son `evaluacion_modelos.png`, `confusion_matrix.png` y `feature_importance.png`.

## 7. Despliegue para Power BI

El notebook genera `output_powerbi/fifa_predicciones_powerbi.xlsx` con cuatro hojas:

| Hoja | Contenido |
|------|-----------|
| `Todos_Partidos` | partidos con resultado real, prediccion, probabilidades, features resumidas y conjunto ML |
| `World_Cup` | subconjunto de partidos del FIFA World Cup |
| `Evaluacion_Modelos` | comparacion de modelos con accuracy, F1 macro y bandera de mejor modelo |
| `Importancia_Vars` | importancia de variables del mejor modelo |

Las columnas principales de prediccion son:

- `Resultado_Real`;
- `Prediccion`;
- `Prediccion_Correcta`;
- `Prob_Derrota`;
- `Prob_Empate`;
- `Prob_Victoria`;
- `Conjunto_ML`;
- `Modelo`.

Este archivo permite construir dashboards de desempeno del modelo, analisis de resultados por torneo y visualizaciones de importancia de variables.

## 8. Simulacion TOP 4 Mundial 2026

La ultima parte del notebook usa el mejor modelo entrenado para simular escenarios del Mundial 2026 mediante Monte Carlo.

El procedimiento es:

1. Se define una fecha de referencia (`2026-07-10`).
2. Se calcula la forma reciente de los equipos candidatos con `calcular_forma()`.
3. Para cada partido simulado se construye un vector de features con `features_partido()`.
4. El modelo estima probabilidades de derrota, empate y victoria.
5. En eliminatorias, el empate se convierte en una probabilidad 50/50 de avance por penales.
6. Se repite el torneo 5,000 veces.
7. Se agregan frecuencias de campeon, subcampeon, tercer lugar y cuarto lugar.

Las salidas de esta simulacion son:

| Archivo | Contenido |
|---------|-----------|
| `top4_modelo_ml.xlsx` | tabla de probabilidades por equipo y posicion |
| `top4_crisp_dm.png` | visualizacion del TOP 4 y distribucion de probabilidades |

El Excel `top4_modelo_ml.xlsx` contiene las columnas:

```text
Equipo, Campeon (%), Subcampeon (%), 3er Lugar (%), 4to Lugar (%),
Pts_Forma, GF_Forma, GC_Forma, Dif_Forma
```

## 9. Funcionamiento end-to-end

El flujo completo del repositorio puede resumirse asi:

```text
Dataset historico internacional
        ↓
Limpieza, filtrado desde 2010 y EDA
        ↓
Feature engineering temporal
        ↓
Division train/test por fecha
        ↓
Entrenamiento de 4 modelos
        ↓
Seleccion del mejor modelo por accuracy en test
        ↓
Predicciones y probabilidades para todos los partidos
        ↓
Exportacion a Excel y graficos para Power BI
        ↓
Simulacion Monte Carlo del TOP 4 Mundial 2026
```

## 10. Consideraciones importantes

- El notebook depende de internet para descargar el dataset real, aunque tiene respaldo local sintetico en caso de fallo.
- La version actual filtra datos desde 2010, lo cual hace que el modelo se enfoque en futbol internacional reciente.
- Las probabilidades del TOP 4 dependen del dataset descargado, del modelo ganador y de la fecha de referencia usada en la simulacion.
- El proyecto no incorpora ranking FIFA, ratings Elo, lesiones, alineaciones, xG ni informacion individual de jugadores.
- Los resultados deben interpretarse como una aproximacion probabilistica, no como una prediccion deterministica.

## 11. Conclusion

El repositorio implementa un pipeline completo y reproducible para analisis predictivo de partidos internacionales. Su valor principal esta en combinar metodologia CRISP-DM, feature engineering temporal, comparacion de modelos y exportacion orientada a Power BI.

La arquitectura actual es adecuada para un proyecto academico o demostrativo: todo el proceso esta en un notebook, las dependencias estan centralizadas en `requirements.txt` y las salidas finales quedan disponibles en `output_powerbi/` para presentacion, visualizacion y analisis.

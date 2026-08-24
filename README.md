# Clasificación de municipios de Morelos según su nivel de conectividad a internet

## Integrantes

- Estefany Alexa Delgado Heredia
- Rocio Rodríguez Torres
- Antonio Garcia Gonzalez
- Diego Ricardo Amador Casillas

## Grupo

9A IDGS

## Materia

Extracción de conocimiento de Bases de Datos

## Pregunta de investigación

¿Es posible clasificar a los municipios de Morelos como de "conectividad alta" o "conectividad baja" usando el porcentaje de viviendas con computadora, TV de paga y celular?

## Fuente de datos

Los datos provienen del **Instituto Nacional de Estadística y Geografía (INEGI)**, del tabulado de Viviendas del Censo de Población y Vivienda 2020, filtrado al estado de Morelos. Se descargaron desde **SCITEL** (Sistema de Consulta de Tabulados sobre Tecnologías de la Información en los Hogares), que permite seleccionar solo las variables e indicadores de interés en vez de descargar el tabulado completo del Censo.

- Fuente: [https://www.inegi.org.mx/app/scitel/Default?ev=9](https://www.inegi.org.mx/app/scitel/Default?ev=9)
- Detalle de la fuente y fecha de consulta: [`referencias/fuentes_datos.txt`](referencias/fuentes_datos.txt)

## Descripción breve del dataset

El CSV original (`data/raw/dataset_original.csv`) viene a **nivel localidad**: 1,678 filas y 11 variables. El análisis final trabaja a **nivel municipio** (36 filas), quedándose solo con las filas donde `LOC == 0` ("Total del Municipio") y `MUN > 0`.

| Variable     | Descripción                                                    |
|-------------|------------------------------------------------------------------|
| `ENTIDAD`   | Clave de la entidad federativa                                   |
| `NOM_ENT`   | Nombre de la entidad (Morelos)                                   |
| `MUN`       | Clave del municipio                                               |
| `NOM_MUN`   | Nombre del municipio                                              |
| `LOC`       | Clave de la localidad (`0` = total del municipio)                 |
| `NOM_LOC`   | Nombre de la localidad                                            |
| `VPH_C_ELEC`| Viviendas particulares habitadas con electricidad                 |
| `VPH_INTER` | Viviendas particulares habitadas con internet                     |
| `VPH_CEL`   | Viviendas particulares habitadas con teléfono celular              |
| `VPH_STVP`  | Viviendas particulares habitadas con servicio de TV de paga        |
| `VPH_PC`    | Viviendas particulares habitadas con computadora                   |

Las cifras confidenciales del INEGI (localidades con menos de tres viviendas) vienen marcadas con `*`.

## Técnicas usadas

1. **Limpieza de datos:** filtrado a nivel municipio (`MUN > 0` y `LOC == 0`, 36 filas), conversión de `*` a `NaN` e imputación con `0`.
2. **Feature engineering:** cálculo de porcentajes respecto a viviendas con electricidad — `pct_internet`, `pct_pc`, `pct_stvp`, `pct_cel` — en vez de conteos absolutos, para no confundir "tamaño del municipio" con "brecha digital".
3. **Variable objetivo:** etiqueta binaria `conectividad_alta`:
   - `1` (Alta): si `pct_internet >= promedio estatal de pct_internet` (~42.7%)
   - `0` (Baja/Media): si `pct_internet < promedio estatal`
   El umbral es el promedio estatal calculado sobre los 36 municipios, no un número fijo elegido a priori.
4. **Selección de predictoras:** `pct_pc`, `pct_stvp`, `pct_cel` (numéricas). `NOM_MUN` **no** se usa como predictor: a nivel municipio cada fila es un municipio distinto, así que sería un identificador (fuga de datos), no una variable generalizable.
5. **Modelo supervisado:** `DecisionTreeClassifier` de scikit-learn con `max_depth=4`, criterio Gini y `random_state=42`.
6. **Evaluación:** división entrenamiento/prueba (70/30) estratificada, matriz de confusión, accuracy, precision, recall y f1-score.

## Principales resultados

- El modelo alcanzó una **exactitud (accuracy) de 0.9091** (10 de 11 municipios) en el conjunto de prueba.
- Importancia de variables: **`pct_stvp`** (mayor peso) > `pct_pc` > `pct_cel`.
- Distribución de clases: **20 municipios "Baja/Media"** (55.6%) y **16 "Alta"** (44.4%), sobre 36 municipios totales.

Ver el detalle en [`outputs/resumen_resultados.csv`](outputs/resumen_resultados.csv) y [`outputs/resultados_modelo.csv`](outputs/resultados_modelo.csv).

## Limitaciones

- **Muestra chica:** solo 36 municipios (11 en el conjunto de prueba), por lo que la exactitud tiene varianza alta y puede cambiar de forma notoria con otro `random_state`.
- **Datos agregados a nivel municipio:** al usar el total municipal (`LOC == 0`) se pierde la variación interna entre localidades de un mismo municipio.
- **Datos de un periodo censal específico:** las cifras son del Censo 2020 y no capturan cambios posteriores en la conectividad.
- **Variable faltante:** el "promedio de ocupantes por vivienda" es un indicador socioeconómico que podría enriquecer el modelo, pero no está disponible en este tabulado del Censo 2020; se omitió en vez de aproximarlo con datos que no están en el dataset.
- **Confidencialidad de datos base:** el INEGI marca con `*` las cifras de localidades individuales con menos de tres viviendas. Ninguna de las 36 filas de nivel municipio usadas en este análisis trae valores marcados como confidenciales (el paso de imputación `* → 0` del código no llega a aplicarse aquí), pero no es posible verificar con este tabulado si los totales municipales que reporta el INEGI incorporan por completo el equipamiento de esas localidades pequeñas.
- **Umbral propio del proyecto:** la definición de "conectividad alta" (promedio estatal de `pct_internet`) es una definición propia de este ejercicio, no un estándar oficial de INEGI — una etiqueta creada con esta regla no representa una verdad absoluta.
- El modelo no demuestra causalidad; los resultados se limitan a observar asociaciones con los datos disponibles.

## Cómo ejecutar el notebook

1. Clonar el repositorio:
   ```bash
   git clone https://github.com/amadorcdr/proyecto-articulo-datos-publicos.git
   cd proyecto-articulo-datos-publicos
   ```

2. Instalar dependencias (se recomienda usar un entorno virtual):
   ```bash
   pip install -r requirements.txt
   ```

3. Abrir y ejecutar el notebook de principio a fin:
   ```bash
   jupyter notebook notebook/analisis_datos_publicos.ipynb
   ```

   El notebook crea automáticamente las carpetas de salida necesarias (`data/`, `outputs/`, `imagenes/`, `referencias/`) y parte del dataset ya incluido en `data/raw/dataset_original.csv`, así que no requiere ninguna descarga adicional.

4. Los resultados se exportan a:
   - `data/processed/dataset_limpio.csv` — dataset limpio a nivel municipio (36 filas), con porcentajes y variable objetivo.
   - `outputs/resultados_modelo.csv` — predicciones del modelo sobre el conjunto de prueba.
   - `outputs/resumen_resultados.csv` — reporte de clasificación (precision/recall/f1 por clase).
   - `imagenes/grafica_1.png` — distribución del porcentaje de viviendas con internet por municipio, con el umbral de conectividad alta marcado.
   - `imagenes/grafica_2.png` — conteo de clases de la variable objetivo.

## Enlace al artículo técnico

El artículo técnico completo se encuentra en [`articulo_tecnico.md`](articulo_tecnico.md).

## Estructura del repositorio

```
proyecto-articulo-datos-publicos/
├── README.md
├── articulo_tecnico.md
├── requirements.txt
├── notebook/
│   └── analisis_datos_publicos.ipynb
├── data/
│   ├── raw/
│   │   └── dataset_original.csv
│   └── processed/
│       └── dataset_limpio.csv
├── outputs/
│   ├── resultados_modelo.csv
│   └── resumen_resultados.csv
├── imagenes/
│   ├── grafica_1.png
│   └── grafica_2.png
└── referencias/
    └── fuentes_datos.txt
```

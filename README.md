# Clasificación de localidades de Morelos según su nivel de conectividad digital

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

¿Es posible clasificar las localidades del estado de Morelos como de alta o baja conectividad digital a partir de indicadores de equipamiento tecnológico en viviendas particulares habitadas?

## Fuente de datos

Los datos provienen del **Instituto Nacional de Estadística y Geografía (INEGI)**, específicamente de los indicadores de viviendas particulares habitadas del Censo de Población y Vivienda. La fuente es pública, oficial y verificable.

- Sitio oficial: [https://www.inegi.org.mx/](https://www.inegi.org.mx/)

## Descripción breve del dataset

El dataset original contiene **1,678 registros** y **11 variables** correspondientes a localidades del estado de Morelos. Las variables incluyen:

| Variable     | Descripción                                                    |
|-------------|----------------------------------------------------------------|
| `ENTIDAD`   | Clave de la entidad federativa                                 |
| `NOM_ENT`   | Nombre de la entidad (Morelos)                                 |
| `MUN`       | Clave del municipio                                            |
| `NOM_MUN`   | Nombre del municipio                                           |
| `LOC`       | Clave de la localidad                                          |
| `NOM_LOC`   | Nombre de la localidad                                         |
| `VPH_C_ELEC`| Viviendas particulares habitadas con electricidad              |
| `VPH_INTER` | Viviendas particulares habitadas con internet                  |
| `VPH_CEL`   | Viviendas particulares habitadas con teléfono celular          |
| `VPH_STVP`  | Viviendas particulares habitadas con servicio de TV de paga    |
| `VPH_PC`    | Viviendas particulares habitadas con computadora               |

## Técnicas usadas

1. **Limpieza de datos:** eliminación de registros agregados (totales estatales y municipales), conversión de valores confidenciales (`*`) a `NaN` e imputación con 0, y creación de variables derivadas.
2. **Feature engineering:** cálculo del porcentaje de viviendas con internet respecto a las que tienen electricidad (`pct_internet`).
3. **Variable objetivo:** etiqueta binaria `conectividad_alta` creada con regla clara:
   - `1` (Alta): si `pct_internet >= 30%`
   - `0` (Baja/Media): si `pct_internet < 30%`
4. **One-Hot Encoding:** codificación de la variable categórica `NOM_MUN` en indicadores numéricos.
5. **Modelo supervisado:** `DecisionTreeClassifier` de scikit-learn con `max_depth=4`, criterio Gini y `random_state=42`.
6. **Evaluación:** división entrenamiento/prueba (70/30) estratificada, matriz de confusión, accuracy, precision, recall y f1-score.

## Principales resultados

- El modelo alcanzó una **exactitud (accuracy) del 77.22%** en el conjunto de prueba.
- La clase 0 (Baja/Media) obtuvo una precision de 0.86 y recall de 0.72.
- La clase 1 (Alta) obtuvo una precision de 0.69 y recall de 0.84.
- Las variables con mayor peso predictivo fueron **VPH_PC** (computadoras) y **VPH_CEL** (celulares).
- La distribución de clases está relativamente equilibrada: 57.1% Baja/Media vs 42.9% Alta.

## Limitaciones

- Los datos corresponden a un periodo censal específico y no reflejan cambios posteriores en la conectividad.
- Las cifras marcadas como confidenciales (`*`) fueron imputadas con 0, lo cual puede subestimar los valores reales en localidades pequeñas.
- La etiqueta `conectividad_alta` se creó con un umbral arbitrario del 30%, el cual no representa una verdad absoluta.
- El modelo no demuestra causalidad; los resultados se limitan a observar asociaciones con los datos disponibles.
- Solo se analizó el estado de Morelos, por lo que los resultados no son generalizables al resto del país.
- KMeans o modelos más complejos podrían revelar patrones adicionales no capturados por un árbol de decisión con profundidad limitada.

## Cómo ejecutar el notebook

1. Clonar el repositorio:
   ```bash
   git clone <url-del-repositorio>
   cd analisis-municipal
   ```

2. Instalar dependencias (se recomienda usar un entorno virtual):
   ```bash
   pip install pandas numpy matplotlib seaborn scikit-learn joblib
   ```

3. Abrir y ejecutar el notebook:
   ```bash
   jupyter notebook notebook/analisis_datos_publicos.ipynb
   ```

   El notebook crea automáticamente las carpetas de salida necesarias (`data/`, `outputs/`, `imagenes/`, `referencias/`).

4. Los resultados se exportan a:
   - `outputs/resultados_modelo.csv` — predicciones del modelo sobre el conjunto de prueba.
   - `imagenes/grafica_1.png` — distribución del porcentaje de viviendas con internet.
   - `imagenes/grafica_2.png` — conteo de clases de la variable objetivo.

## Enlace al artículo técnico

El artículo técnico completo se encuentra en [`articulo_tecnico.md`](articulo_tecnico.md).

## Estructura del repositorio

```
analisis-municipal/
├── README.md
├── articulo_tecnico.md
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
# Clasificación de localidades de Morelos según su nivel de conectividad digital

**Autores:** 
- Estefany Alexa Delgado Heredia
- Rocio Rodríguez Torres
- Antonio Garcia Gonzalez
- Diego Ricardo Amador Casillas

**Materia:** Extracción de conocimiento de Bases de Datos

**Institución:** UTEZ (Universidad Tecnológica de Emiliano Zapata) - Grupo 9A IDGS

---

## Resumen

En este artículo se presenta un análisis de clasificación supervisada aplicado a localidades del estado de Morelos, México, con el objetivo de determinar si es posible identificar zonas con alta o baja conectividad digital a partir de indicadores de equipamiento tecnológico en viviendas particulares habitadas. Se utilizaron datos públicos del Instituto Nacional de Estadística y Geografía (INEGI), que incluyen variables como la disponibilidad de electricidad, internet, teléfono celular, televisión de paga y computadora. Se construyó una variable objetivo binaria basada en el porcentaje de viviendas con acceso a internet respecto a las que cuentan con electricidad, utilizando un umbral del 30% como criterio de clasificación. Se aplicó un modelo de Árbol de Decisión (`DecisionTreeClassifier`) que alcanzó una exactitud del 77.22% en el conjunto de prueba. Los resultados sugieren que la disponibilidad de computadoras y teléfonos celulares son los factores más relevantes para predecir el nivel de conectividad de una localidad. Este trabajo busca servir como evidencia de portafolio profesional y como un ejercicio reproducible de análisis de datos públicos.

---

## 1. Introducción

La brecha digital es un fenómeno que afecta de manera desigual a las localidades de México. Mientras que algunas zonas urbanas cuentan con amplia cobertura de servicios de telecomunicaciones, muchas localidades rurales y semiurbanas presentan carencias significativas en el acceso a internet y tecnologías de la información.

En el estado de Morelos, esta disparidad se manifiesta entre sus diferentes municipios y localidades. Comprender qué factores están asociados con un mayor o menor nivel de conectividad digital resulta relevante para orientar políticas públicas de inversión en infraestructura de telecomunicaciones.

La pregunta que guía este análisis es: **¿Es posible clasificar las localidades de Morelos como de alta o baja conectividad digital a partir de indicadores de equipamiento tecnológico en viviendas?**

Para responder esta pregunta se utilizó un enfoque de aprendizaje supervisado con un modelo de clasificación, tomando como base datos oficiales y públicos del INEGI.

---

## 2. Fuente de datos

Los datos utilizados en este análisis provienen del **Instituto Nacional de Estadística y Geografía (INEGI)**, específicamente de los indicadores del Censo de Población y Vivienda correspondientes al estado de Morelos.

- **Sitio oficial:** [https://www.inegi.org.mx/](https://www.inegi.org.mx/)
- **Tipo de datos:** indicadores de viviendas particulares habitadas a nivel localidad.
- **Confiabilidad:** el INEGI es el organismo público autónomo responsable de generar la información estadística oficial de México. Sus datos son de uso libre y ampliamente utilizados en investigaciones académicas y decisiones de política pública.

El dataset original contiene **1,678 registros** y **11 variables**, que incluyen claves geográficas (entidad, municipio, localidad) y cinco indicadores de equipamiento tecnológico en viviendas: electricidad (`VPH_C_ELEC`), internet (`VPH_INTER`), teléfono celular (`VPH_CEL`), TV de paga (`VPH_STVP`) y computadora (`VPH_PC`).

---

## 3. Preparación de datos

La preparación de los datos involucró los siguientes pasos:

### 3.1 Inspección inicial

Se verificó la estructura del dataset, identificando que las cinco variables de equipamiento tecnológico estaban almacenadas como tipo `object` en lugar de tipo numérico. Esto se debe a que el INEGI utiliza el carácter `*` para señalar datos confidenciales o no disponibles en localidades con menos de tres viviendas.

### 3.2 Limpieza básica

1. **Eliminación de registros agregados:** se excluyeron los registros correspondientes a totales estatales y municipales (donde `MUN == 0` o `LOC` corresponde a agregados), conservando únicamente las localidades individuales. Tras esta operación el dataset se redujo de 1,678 a **1,578 registros de localidades**.

2. **Conversión de tipos:** las columnas con valores `*` se convirtieron a tipo numérico, transformando los asteriscos en valores `NaN`.

3. **Imputación:** los valores `NaN` resultantes se imputaron con `0`, bajo la interpretación de que localidades sin datos reportados probablemente carecen del equipamiento en cuestión.

### 3.3 Feature engineering

Se calculó una nueva variable `pct_internet`, definida como el porcentaje de viviendas con internet respecto a las viviendas con electricidad en cada localidad:

```
pct_internet = (VPH_INTER / VPH_C_ELEC) × 100
```

En los casos donde `VPH_C_ELEC` es cero, se asignó el valor `0` a `pct_internet` para evitar divisiones por cero.

### 3.4 Creación de la variable objetivo

Se creó la variable binaria `conectividad_alta` siguiendo una regla clara:

- Si `pct_internet >= 30` → `conectividad_alta = 1` (Alta)
- Si `pct_internet < 30` → `conectividad_alta = 0` (Baja/Media)

La distribución resultante fue: **57.1%** localidades clasificadas como Baja/Media y **42.9%** como Alta, lo que indica un balance razonable entre clases.

---

## 4. Selección de variables

Se definieron como variables predictoras (`X`):

- **Variables numéricas:** `VPH_C_ELEC`, `VPH_CEL`, `VPH_STVP`, `VPH_PC`
- **Variable categórica:** `NOM_MUN` (nombre del municipio)

La variable `VPH_INTER` se excluyó del conjunto de predictores porque fue utilizada directamente para construir la variable objetivo, lo cual habría generado fuga de datos (data leakage).

La variable categórica `NOM_MUN` se transformó mediante **One-Hot Encoding** para generar indicadores binarios para cada municipio.

---

## 5. Análisis exploratorio

Antes de aplicar el modelo, se realizó un análisis exploratorio para comprender la distribución de los datos.

### 5.1 Distribución del porcentaje de internet

Se construyó un histograma del porcentaje de viviendas con internet (`pct_internet`). La distribución muestra una concentración de localidades con valores bajos (entre 0% y 20%), con una cola derecha que se extiende hasta el 100%. El umbral del 30% utilizado para la clasificación queda ubicado en una zona de transición entre la mayoría de localidades con baja conectividad y aquellas con conectividad moderada a alta.

### 5.2 Conteo de clases

La distribución de la variable objetivo muestra 901 localidades en la clase 0 (Baja/Media) y 677 en la clase 1 (Alta). Este balance relativo permite entrenar el modelo sin necesidad de técnicas de balanceo de clases.

Las gráficas generadas se encuentran en la carpeta `imagenes/` del repositorio.

---

## 6. Metodología

### 6.1 Modelo seleccionado

Se utilizó un modelo de **Árbol de Decisión** (`DecisionTreeClassifier` de scikit-learn) por las siguientes razones:

- Es un modelo de aprendizaje supervisado adecuado cuando se cuenta con una variable objetivo definida.
- Su estructura es interpretable, lo que permite identificar qué variables influyen más en la clasificación.
- Es un modelo visto durante la materia y apropiado para el alcance del proyecto.

### 6.2 Configuración del modelo

- **Profundidad máxima:** `max_depth = 4` para evitar sobreajuste.
- **Criterio de división:** Gini.
- **Semilla aleatoria:** `random_state = 42` para reproducibilidad.

### 6.3 Entrenamiento y evaluación

El dataset se dividió en conjunto de entrenamiento (70%) y conjunto de prueba (30%), aplicando estratificación sobre la variable objetivo para mantener la proporción de clases:

- **Entrenamiento:** 1,104 localidades.
- **Prueba:** 474 localidades.

---

## 7. Resultados

El modelo entrenado produjo los siguientes resultados sobre el conjunto de prueba:

| Métrica        | Clase 0 (Baja/Media) | Clase 1 (Alta) | Promedio ponderado |
|---------------|:--------------------:|:--------------:|:------------------:|
| Precision     | 0.86                 | 0.69           | 0.79               |
| Recall        | 0.72                 | 0.84           | 0.77               |
| F1-Score      | 0.78                 | 0.76           | 0.77               |

**Exactitud global (Accuracy):** 0.7722 (77.22%)

La matriz de confusión muestra que el modelo clasifica correctamente la mayoría de las localidades en ambas clases, aunque presenta una tendencia ligeramente mayor a clasificar localidades de baja conectividad como alta (falsos positivos de la clase 1).

Se generó además una visualización del árbol de decisión entrenado, que permite observar las reglas de clasificación utilizadas en cada nodo.

---

## 8. Interpretación

Con los datos analizados se observa lo siguiente:

1. **Variables clave:** la disponibilidad de **computadoras (`VPH_PC`)** y **teléfonos celulares (`VPH_CEL`)** son las variables con mayor peso predictivo dentro del árbol de decisión. Esto sugiere que las localidades con mayor penetración de estos dispositivos tienden a tener también mayor acceso a internet.

2. **Diferencias municipales:** la inclusión de variables indicadoras por municipio permite al modelo capturar diferencias estructurales en conectividad entre los distintos municipios de Morelos, aunque su contribución es menor comparada con las variables numéricas de equipamiento.

3. **Aplicación práctica:** los resultados sugieren que un modelo relativamente sencillo puede identificar patrones de conectividad a nivel localidad. Bajo las variables consideradas, se encontraron asociaciones consistentes entre el equipamiento tecnológico doméstico y el acceso a internet.

Es importante señalar que este resultado debe interpretarse con cuidado, ya que el modelo no demuestra causalidad. La disponibilidad de computadoras no necesariamente causa mayor conectividad a internet; ambos indicadores podrían estar asociados con variables latentes como el nivel socioeconómico de la localidad.

---

## 9. Limitaciones

Todo análisis tiene limitaciones que deben reconocerse:

1. **Datos de un periodo específico:** los datos corresponden a un corte censal específico y no capturan cambios temporales en la conectividad digital.

2. **Datos agregados:** los indicadores están agregados a nivel vivienda, no a nivel individual, lo cual limita la granularidad del análisis.

3. **Valores confidenciales:** las cifras marcadas como `*` por el INEGI fueron imputadas con 0. Esta decisión puede subestimar los valores reales en localidades muy pequeñas.

4. **Umbral arbitrario:** la etiqueta `conectividad_alta` se creó con un umbral del 30% que, aunque razonable, no representa una verdad absoluta. Diferentes umbrales producirían clasificaciones distintas.

5. **El modelo no demuestra causalidad:** las asociaciones encontradas entre equipamiento tecnológico y conectividad no implican relaciones causales.

6. **Variables faltantes:** no se incluyeron variables socioeconómicas adicionales (ingreso, escolaridad, densidad poblacional) que podrían mejorar la capacidad predictiva del modelo.

7. **Generalización:** el análisis se limita al estado de Morelos, por lo que los resultados no son generalizables al resto del país sin validación adicional.

8. **Posible sesgo de captura:** las localidades más pequeñas y remotas son precisamente aquellas con mayor probabilidad de tener datos confidenciales, lo cual introduce un sesgo hacia la subestimación de su equipamiento.

---

## 10. Conclusiones

A partir del análisis realizado se pueden establecer las siguientes conclusiones:

1. Los resultados sugieren que es posible clasificar localidades de Morelos según su nivel de conectividad digital utilizando indicadores de equipamiento tecnológico en viviendas, con una exactitud del 77.22%.

2. Con los datos disponibles se observa que la disponibilidad de computadoras y teléfonos celulares son los indicadores más fuertemente asociados con el acceso a internet a nivel localidad.

3. El análisis confirma la existencia de una brecha digital significativa dentro del estado de Morelos, donde más de la mitad de las localidades analizadas presentan un nivel de conectividad clasificado como bajo o medio.

4. Bajo las variables consideradas, se encontró que un modelo de Árbol de Decisión con profundidad limitada puede capturar patrones relevantes de conectividad, lo que sugiere su utilidad como herramienta de exploración inicial para la identificación de zonas con déficit digital.

5. Este tipo de análisis podría servir como insumo para focalizar la inversión pública en infraestructura de telecomunicaciones en las localidades que más lo necesitan, aunque se requeriría complementar con información adicional y validación en campo.

---

## 11. Referencias

1. Instituto Nacional de Estadística y Geografía (INEGI). Censo de Población y Vivienda. Disponible en: [https://www.inegi.org.mx/](https://www.inegi.org.mx/)

2. scikit-learn. `DecisionTreeClassifier`. Documentación oficial. Disponible en: [https://scikit-learn.org/stable/modules/generated/sklearn.tree.DecisionTreeClassifier.html](https://scikit-learn.org/stable/modules/generated/sklearn.tree.DecisionTreeClassifier.html)

3. Pedregosa, F. et al. (2011). Scikit-learn: Machine Learning in Python. *Journal of Machine Learning Research*, 12, 2825–2830.

4. McKinney, W. (2010). Data Structures for Statistical Computing in Python. *Proceedings of the 9th Python in Science Conference*, 51–56.

5. Hunter, J. D. (2007). Matplotlib: A 2D Graphics Environment. *Computing in Science & Engineering*, 9(3), 90–95.

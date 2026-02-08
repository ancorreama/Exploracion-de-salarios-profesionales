### DOCUMENTACIÓN DEL MODELO
Ask a manager salary survey 2021

## 1. Variables en la base de datos original

La siguiente tabla describe las principales variables utilizadas de la base de datos original.
Los nombres de las variables se mantienen exactamente como aparecen en el formulario original
para facilitar la trazabilidad con la fuente.

| Variable (base original) | Tipo de variable | Descripción |
|--------------------------|-----------------|-------------|
| Timestamp | Fecha/Hora | Momento en el que se registró la respuesta en el formulario. |
| How old are you? | Texto | Edad reportada por la persona encuestada. Puede venir como número o texto. |
| What industry do you work in? | Texto | Industria o sector económico en el que trabaja la persona encuestada. |
| Job title | Texto | Cargo o título del puesto de trabajo reportado. |
| If your job title needs additional context, please clarify here: | Texto | Campo abierto para aclarar o ampliar información sobre el cargo. |
| What is your annual salary? (You'll indicate the currency in a later question. If you are part-time or hourly, please enter an annualized equivalent -- what you would earn if you worked the job 40 hours a week for 52 weeks.) | Texto | Salario anual reportado. Puede presentarse en distintos formatos (por ejemplo: “80k”, “80000”, “$80,000” o “80”). |
| How much additional monetary compensation do you get, if any (for example, bonuses or overtime in an average year)? Please only include monetary compensation here, not the value of benefits. | Texto | Compensaciones monetarias adicionales reportadas (bonos, horas extra, etc.). |
| Please indicate the currency | Texto | Moneda en la que se reportan el salario y las compensaciones. |
| If "Other," please indicate the currency here: | Texto | Campo abierto para especificar la moneda cuando se selecciona “Other”. |
| If your income needs additional context, please provide it here: | Texto | Campo abierto para aclaraciones adicionales sobre el ingreso. |
| What country do you work in? | Texto | País donde trabaja la persona encuestada (campo libre con variaciones de escritura). |
| If you're in the U.S., what state do you work in? | Texto | Estado de EE. UU. donde trabaja la persona (si aplica). |
| What city do you work in? | Texto | Ciudad donde trabaja la persona encuestada (campo libre, puede incluir paréntesis). |
| How many years of professional work experience do you have overall? | Texto | Años de experiencia profesional total. |
| How many years of professional work experience do you have in your field? | Texto | Años de experiencia profesional en el campo específico. |
| What is your highest level of education completed? | Texto | Nivel educativo más alto completado. |
| What is your gender? | Texto | Género reportado por la persona encuestada. |
| What is your race? (Choose all that apply.) | Texto | Raza reportada (puede permitir múltiples selecciones). |

## 2. Variables luego de modeladas

Durante el modelado se decidió mantener los nombres de variables en inglés,
garantizando consistencia y facilidad de trazabilidad con la base original.

| Variable modelada | Tipo de variable | Descripción |
|-------------------|-----------------|-------------|
| country_clean | Texto | Versión normalizada del país original (minúsculas, sin puntos ni espacios dobles) para facilitar estandarización. |
| country_std | Texto | País estandarizado mediante reglas de equivalencia (por ejemplo, “us”, “usa”, “united states” → “United States”), utilizado en visualizaciones geográficas. |
| city_std | Texto | Ciudad normalizada removiendo paréntesis, reduciendo espacios repetidos y aplicando formato de título. |
| annual_salary_usd | Numérico | Salario anual convertido a valor numérico en USD a partir de distintos formatos textuales. |
| bonus_usd | Numérico | Compensación monetaria adicional convertida a valor numérico en USD usando la misma lógica que el salario anual. |
| salario_anual_cop | Numérico | Salario anual expresado en pesos colombianos (COP), calculado como `annual_salary_usd * trm`. |
| compensaciones_cop | Numérico | Compensaciones adicionales expresadas en COP. |
| total_comp_cop | Numérico | Compensación total anual en COP, calculada como la suma del salario anual y las compensaciones adicionales. |
| trm | Numérico | Tasa de cambio utilizada (COP por USD) para la conversión de valores monetarios. |
| trm_fecha | Texto / Fecha | Fecha correspondiente a la TRM utilizada, incluida para trazabilidad. |
| df_model | Dataset | Dataset final filtrado para visualización, sin valores no plausibles que puedan distorsionar los gráficos. |


## 3.	Reglas de negocio y supuestos de modelado
Durante el proceso de modelado se establecieron las siguientes reglas y supuestos:

## 1.	Normalización de texto:
Los campos de país y ciudad se limpiaron para reducir variaciones de escritura propias de campos de texto libre.
## 2.	Conversión de salarios desde texto a numérico:
Se eliminaron símbolos monetarios, comas y textos no numéricos.
Los valores expresados con la letra “k” fueron interpretados como miles (por ejemplo, “80k” → 80,000).
## 3.	Regla de interpretación de valores pequeños:
Cuando el valor numérico resultante es menor a 1000, se asume que el salario fue reportado en miles (por ejemplo, 80 → 80,000 USD).
Esta decisión se toma debido al uso común de abreviaciones informales en el formulario.
## 4.	Conversión a pesos colombianos:
Todos los valores monetarios se convirtieron a COP utilizando la TRM correspondiente a la fecha del ejercicio, la cual se registra explícitamente en el dataset.
## 5.	Tratamiento de valores extremos:
Se eliminaron valores no plausibles (outliers extremos) con el fin de evitar distorsiones en las visualizaciones y en los promedios presentados en el dashboard.
## 6.	Enfoque exploratorio:
El análisis y las visualizaciones están pensadas para exploración interna y no para inferencia estadística.

## 4.	Paso a paso para actualizar los datos y replicar el modelado
Esta guía explica cómo replicar el proceso de actualización y modelado asumiendo que la estructura del formulario/base original no cambia:

## A. Actualización de datos (extracción)
1.	Abrir el Google Sheets público de la encuesta.
2.	Descargar la versión más reciente como archivo CSV.
3.	Guardar el archivo con un nombre identificable (ej: Ask_A_Manager_2021_responses.csv).

## B. Cargar datos en Google Colab
4.	Abrir el notebook de modelado en Google Colab.
5.	Subir el archivo CSV a Colab (panel Files → Upload).
6.	Actualizar la variable file_path con el nombre/ruta del archivo cargado.
7.	Ejecutar la celda de lectura del CSV con pd.read_csv(file_path).

## C. Preparación (nombres de columnas)
8.	Normalizar nombres de columnas (strip, lower, reemplazo de espacios por guiones bajos y eliminación de signos como “?” y paréntesis) para facilitar referencia programática.
9.	Definir variables base que apunten a las columnas clave del formulario (país, ciudad, salario y compensación), para evitar errores por nombres largos.

## D. Limpieza de ubicación (Country / City)
10.	Crear country_clean aplicando limpieza de texto (minúsculas, sin puntos, sin espacios dobles).
11.	Crear country_std aplicando un diccionario de estandarización (por ejemplo, “us/usa/united states” → “United States”).
12.	Crear city_std removiendo paréntesis, reduciendo espacios repetidos y aplicando formato de título.

## E. Conversión de salarios a numérico (USD)
13.	Definir una función para convertir los campos de salario y compensación (texto) a números:
•	Remover comas y símbolos monetarios.
•	Interpretar formatos tipo “80k” como 80,000.
•	Regla: si el valor numérico es menor a 1000, interpretarlo como miles (ej: 80 → 80,000).
14.	Crear annual_salary_usd aplicando la función sobre la columna de salario anual.
15.	Crear bonus_usd aplicando la función sobre la columna de compensación adicional.

## F. Conversión a pesos colombianos (COP)
16.	Consultar la TRM del día del ejercicio (COP por USD).
17.	Registrar:
•	trm (valor numérico)
•	trm_fecha (fecha)
18.	Calcular:
•	salario_anual_cop = annual_salary_usd * trm
•	compensaciones_cop = bonus_usd * trm
•	total_comp_cop = salario_anual_cop + compensaciones_cop (tratando nulos como 0)

## G. Filtros de calidad para visualización
19.	Crear el dataset final df_model aplicando filtros mínimos para visualización:
•	Salario anual no nulo
•	Salario anual > 0
•	Eliminación de outliers para evitar distorsión en gráficos
•	País estandarizado no nulo
•	
## H. Exportación del dataset final
20.	Exportar el dataset final a CSV (ej: ask_a_manager_modelado.csv).
21.	Usar este archivo como fuente en Looker Studio para actualizar el dashboard

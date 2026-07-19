# Proyecto BI FISIMart S.A.C. — Proyecto Grupal 2026

Solución integral de Business Intelligence para **FISIMart S.A.C.**, empresa peruana ficticia de retail minorista con 20 puntos de venta en Lima Metropolitana y un programa de fidelización de 5,000 clientes. El proyecto abarca la generación de datos sintéticos, la construcción de un datamart en esquema estrella, un tablero de Power BI y cuatro modelos de minería de datos (clasificación, segmentación, asociación y regresión).

Asignatura: Inteligencia de Negocios
Docente: Mg. Juan Gamarra Moreno.

## Integrantes

- Astuhuaman Vega, Max Steven
- Calle Ramos, Guillermo Javier
- Luján Vila, Frank José
- Rouillon Haro, Favio
- Villanueva Aguirre, Cesar Alexander

## Estructura del repositorio

proyecto-bi-FISIMart/
├── README.md              # Descripción, integrantes, cómo ejecutar
├── requirements.txt       # Dependencias de Python (versiones)
├── data/
│   ├── raw/                # Datos sintéticos crudos (con problemas de calidad)
│   └── processed/          # Datos limpios listos para análisis y Power BI
├── notebooks/
│   ├── 00_generacion_datos.ipynb   # Generación reproducible de datos sintéticos
│   ├── 01_datamart_etl.ipynb       # Parte 1: ETL y modelo dimensional
│   ├── 02_visualizacion.ipynb      # Parte 2: visualizaciones en Python
│   ├── 03_clasificacion.ipynb      # Parte 3: clasificación (churn)
│   ├── 04_segmentacion.ipynb       # Parte 4: segmentación RFM
│   ├── 05_asociacion.ipynb         # Parte 5: reglas de asociación
│   └── 06_regresion.ipynb          # Parte 6: regresión / pronóstico
├── powerbi/
│   └── FISIMart.pbix       # Modelo, medidas DAX y tableros
├── prompts/
│   └── registro_prompts.md # Bitácora de prompts (Anexo A y B)
├── informe/
│   └── Informe_PG_FISIMart.pdf   # Informe consolidado
├── reports/
│   └── figures/             # Gráficos exportados para el informe
└── exposición/
    ├── enlace_video.txt
    └── orden_participacion.txt

## Cómo ejecutar el proyecto

1. **Requisitos previos**: Python 3.12 (los notebooks fueron guardados y probados con
   Python 3.12.10) y `pip`.

2. **Clonar el repositorio**

```bash
   git clone https://github.com/jose-lv/proyecto-bi-FISIMart.git
   cd proyecto-bi-FISIMart
```

3. **Crear un entorno virtual e instalar dependencias**

```bash
   python -m venv venv
   source venv/bin/activate      # En Windows: venv\Scripts\activate
   pip install -r requirements.txt
```

4. **Ejecutar los notebooks en orden**, desde `notebooks/`, con Jupyter Notebook,
   JupyterLab o VS Code:

```bash
   jupyter lab
```

   Orden de ejecución:

   1. `00_generacion_datos.ipynb` — genera los datos sintéticos en `data/raw/`
      (semilla fija = 20, reproducible).
   2. `01_datamart_etl.ipynb` — limpia y transforma los datos en el esquema
      estrella, exportando las tablas finales a `data/processed/`.
   3. `02_visualizacion.ipynb` — genera las visualizaciones analíticas (Pareto,
      estacionalidad, evolución de ventas y margen).
   4. `03_clasificacion.ipynb` — modelo de predicción de abandono (churn).
   5. `04_segmentacion.ipynb` — segmentación RFM con K-Means.
   6. `05_asociacion.ipynb` — reglas de asociación (Apriori / FP-Growth).
   7. `06_regresion.ipynb` — pronóstico de demanda diaria.

5. **Abrir `powerbi/FISIMart.pbix`** con Power BI Desktop para explorar el modelo
   dimensional, las medidas DAX y los tableros, que consumen las tablas de
   `data/processed/`.

## Datos sintéticos

Todos los datos son sintéticos y fueron generados con `pandas`, `numpy` y `faker`
fijando `SEMILLA = 20` en todos los generadores estocásticos, para garantizar la
reproducibilidad. Se incorporaron deliberadamente problemas de calidad (nulos, fechas
heterogéneas, texto sucio, duplicados, outliers y claves huérfanas) que son detectados
y corregidos en el notebook de ETL. El detalle completo de volúmenes, distribuciones y
anomalías está documentado en el informe (`informe/Informe_PG_FISIMart.pdf`, secciones
3.1 a 3.4).

## Registro de prompts

El uso de asistentes de IA generativa (Gemini, Claude y Cursor) como apoyo en el
desarrollo del proyecto está documentado en `prompts/registro_prompts.md` y en la
sección 10 del informe, en cumplimiento de los lineamientos de integridad académica
del curso.

## Informe y sustentación

- Informe consolidado: `informe/Informe_PG_FISIMart.pdf`
- Material de la sustentación: carpeta `exposición/`

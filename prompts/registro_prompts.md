

**FISIMART \- Registro de prompts**

**Anexo A — Plantilla de registro de prompts**

| N° | Parte | Objetivo del prompt | Herramienta / Modelo | Resultado / Uso | Ajustes del equipo |
| :---: | ----- | ----- | :---: | ----- | ----- |
| **B.1** | Modelo dimensional | Diseñar esquema estrella para soporte de BI y ML. | Gemini | Se obtuvo la arquitectura del datamart analítico. | Reajuste en el uso de tildes en los nombres de las columnas y atributos del modelo. |
| **B.2** | Generación de datos | Crear script de Python reproducible (SEMILLA=20) para generar datos sintéticos de retail con reglas de negocio y anomalías. | Gemini | Código que crea carpetas relativas, simula picos de ventas, e inyecta nulos, duplicados y huérfanos | Calibración de ponderados de estacionalidad y tasas de anomalía, e inclusión de nuevas imperfecciones en todas las tablas que fueron obviadas. |
| **B.3** | ETL | Diseñar un pipeline ETL para limpiar, estandarizar formatos de fecha, texto, y calcular métricas derivadas de negocio. | Gemini | Código modular que genera el esquema estrella limpio e imputado. | Ajustes en el prompt para la inclusión de reportes de auditoría explícitos. |
| **B.4** | Clasificación | Diseñar un flujo completo de clasificación supervisada para predecir el abandono (Churn) de clientes. | Claude | Se obtuvo el código y la metodología para preparar el dataset, entrenar y comparar modelos de clasificación. | Refinamiento del prompt para incorporar la comparación de múltiples modelos de clasificación, métricas de evaluación e interpretación de las variables más influyentes . |
| **B.5** | Segmentación | Diseñar un flujo completo de segmentación no supervisada para identificar arquetipos de clientes según comportamiento de compra, calcular métricas RFM, determinar el número óptimo de clusters y generar estrategias diferenciadas por segmento.  | Claude | Código que construye la tabla RFM por cliente (Recency, Frequency, Monetary con fecha de corte dinámica), aplica K-Means con k=4 seleccionado por método del codo y silhouette score (0.43), genera cuatro figuras en reports/figures/ y exporta Segmentacion\_Clientes.csv listo para Power BI.  | Refinamiento del prompt para explorar el repo antes de codear (verificar nombres de columnas reales en Fact\_Ventas y Dim\_Cliente), fijar la fecha de corte en max(fecha)+1 en lugar de la fecha del sistema, invertir correctamente la escala de Recency en los quintiles, y alinear la semilla random\_state=20 con la semilla global del proyecto.  |
| **B.6** | Asociacion | Construir un pipeline de reglas de asociación a nivel SKU, transformando las transacciones en canastas binarias y filtrando por soporte, confianza y lift, para priorizar oportunidades de venta cruzada. | Cursor (Cursor Grok 4.5) | Código que arma las canastas de compra por boleta a nivel producto, aplica Apriori con soporte ajustado a la dispersión del catálogo, y entrega las reglas de asociación (soporte, confianza y lift) junto con las propuestas de venta cruzada. | Se ajustó el nivel de análisis de categoría a producto (SKU), recalibrando los umbrales de soporte mínimo (0,001), confianza mínima (0,03) y lift (\>1) según la dispersión del catálogo, y se propusieron las recomendaciones de venta cruzada a partir de las reglas priorizadas. |
| **B.7** | Regresión (pronóstico de demanda) | Construir un pipeline de pronóstico de venta diaria con variables de calendario, estacionalidad y rezagos, comparando modelos bajo validación temporal. | Claude | Código con el pronóstico de demanda diaria, la comparación de métricas entre los tres modelos y la identificación de los factores (feriados, fin de semana, estacionalidad) con mayor efecto sobre las ventas. | Se ajustó el criterio de selección del mejor modelo para priorizarlo por RMSE ascendente y se verificó manualmente que no existiera fuga de información temporal en la construcción de rezagos y medias móviles (cálculo estrictamente sobre histórico previo). |

**Anexo B — Prompts de generación de datos sintéticos** 

**B.1 Prompt para diseñar el caso y el modelo dimensional.**   
Actúa como Arquitecto Senior de Business Intelligence y Diseñador de Modelos Dimensionales. Basándote en las mejores prácticas de Ralph Kimball, diseña la arquitectura analítica y el esquema estrella para la empresa de retail omnicanal peruana "FISIMart S.A.C.", la cual cuenta con tiendas físicas en Lima Metropolitana, un canal de Ecommerce (Online) y un programa de fidelización de clientes.

Este modelo dimensional servirá como base estructural absoluta para alimentar subsiguientes tareas de analítica avanzada y Machine Learning, específicamente:  
\- Análisis clásico de ventas y rendimiento comercial en Power BI.  
\- Predicción de abandono de clientes (Lógica de Churn).  
\- Segmentación avanzada de clientes (Modelos RFM y Clustering).  
\- Descubrimiento de afinidades de canasta (Reglas de Asociación / Market Basket Analysis).  
\- Pronóstico predictivo de ventas y demanda (Modelos de Regresión).

Por favor, estructura tu entrega detallando estrictamente los siguientes 4 puntos:

1\) PROCESO DE NEGOCIO Y DETERMINACIÓN DEL GRANO:  
Define formalmente el proceso de negocio seleccionado y establece el "Grano" (mínimo nivel de detalle) de la tabla de hechos, garantizando que sea a nivel de línea de ticket/boleta de venta para permitir el análisis de co-ocurrencia de productos.

2\) ESPECIFICACIÓN DE DIMENSIONES (CATÁLOGOS MAESTROS CLEANED):  
Detalla las siguientes 5 dimensiones especificando el nombre de la tabla, su clave primaria (Surrogate Key / Business Key) y todos los atributos necesarios en texto limpio (sin tildes ni caracteres especiales como eñes) para evitar errores de codificación (encoding) al migrar a sistemas externos:  
\- Dim\_Cliente: (id\_cliente, nombre, sexo, fecha\_nacimiento, edad, distrito \[de Lima Metropolitana\], fecha\_alta, segmento\_programa, ingresos\_mensuales\_aprox).  
\- Dim\_Producto: (id\_producto, nombre, categoria \[Abarrotes, Tecnologia, Hogar, Bebidas, Cuidado Personal\], subcategoria, marca, precio\_lista, costo).  
\- Dim\_Tienda: (id\_tienda, nombre, canal \[Fisico / Online\], region, ciudad).  
\- Dim\_Tiempo: (fecha, dia, mes, trimestre, anio, dia\_semana, es\_feriado).  
\- Dim\_Promocion: (id\_promocion, nombre, tipo, descuento\_pct, fecha\_inicio, fecha\_fin).

3\) TABLA DE HECHOS (FACT\_VENTAS):  
Define la estructura de la tabla central 'Fact\_Ventas'. Incluye todas las claves foráneas (Foreign Keys) que conectan con las dimensiones y detalla las métricas cuantitativas base y calculadas requeridas para el negocio: cantidad, precio\_unitario, descuento (porcentaje de la promocion), importe\_bruto (cantidad \* precio\_unitario), importe\_neto (aplicando el descuento), costo\_total (cantidad \* costo) y margen\_ganancia.

4\) DIAGRAMA TEXTUAL Y DICCIONARIO DE DATOS ANALÍTICO:  
\- Proporciona un diagrama textual claro (utilizando caracteres ASCII o notación Mermaid) que ilustre visualmente la topología del Esquema Estrella (relaciones 1 a muchos desde las dimensiones hacia la tabla de hechos).  
\- Construye un Diccionario de Datos exhaustivo en formato de tabla para todo el modelo, donde se indique por cada campo: Nombre de la columna, Tipo de dato analítico (Entero, Decimal, Cadena, Fecha), Descripción del negocio y si actúa como Clave Primaria (PK), Clave Foránea (FK) o Métrica.

**B.2 Prompt maestro de generación de datos sintéticos.**  
Actúa como ingeniero de datos senior y experto en Machine Learning. Genera un script de Python estructurado, robusto y limpio diseñado para ser ejecutado en un Jupyter Notebook (\`00\_generacion\_datos.ipynb\`) dentro de la carpeta \`notebooks/\`. El objetivo es crear un dataset sintético altamente REPRODUCIBLE para la empresa ficticia de retail omnicanal peruana "FISIMart S.A.C.". 

Fija estrictamente la semilla aleatoria en 20 (\`SEMILLA \= 20\`) usando numpy, random y faker para asegurar la consistencia y reproducibilidad matemática absoluta de todas las simulaciones.

REQUISITOS ADICIONALES DE INFRAESTRUCTURA Y FORMATO ELECTRÓNICO:  
1\. Instalación Segura de Librerías y Auditoría: El script debe iniciar obligatoriamente con una celda que ejecute la instalación silenciosa de dependencias usando:  
   %pip install \-q pandas numpy Faker mlxtend  
   Inmediatamente después, debe incluir código en Python que imprima en pantalla un reporte con las versiones exactas instaladas. Para evitar errores de tipo 'AttributeError' con el módulo faker, se debe extraer su versión de forma segura consultando los metadatos de pip mediante 'from importlib.metadata import version; version("Faker")' envuelto en un bloque try-except.  
2\. Estructura de Directorios con Rutas Relativas: Dado que el notebook se ejecutará desde la carpeta \`notebooks/\`, el script debe emplear la nomenclatura \`../\` para escalar a la raíz y crear de forma automatizada las siguientes carpetas usando \`os.makedirs\`:  
   os.makedirs("../data/raw", exist\_ok=True)  
   os.makedirs("../data/processed", exist\_ok=True)  
   os.makedirs("../notebooks", exist\_ok=True)  
   os.makedirs("../prompts", exist\_ok=True)  
   os.makedirs("../powerbi", exist\_ok=True)  
   os.makedirs("../informe", exist\_ok=True)  
3\. Sin Caracteres Especiales (Cleaned Text): Omite de forma estricta la creación de cualquier dato o cadena que contenga tildes, eñes o caracteres especiales en nombres, distritos, categorías o marcas (ej. usar "Tecnologia", "San Borja", "Comas", "Los Olivos", "Santiago de Surco", "Abarrotes", "Hogar", "Bebidas", "Cuidado Personal") para evitar fallas latentes de codificación (encoding) al ser procesados o exportados.

VOLÚMENES REQUERIDOS:  
\- Dim\_Cliente: 5000 registros (id\_cliente, nombre, sexo, fecha\_nacimiento, edad, distrito, fecha\_alta, segmento\_programa, ingresos\_mensuales\_aprox). Los distritos deben ser exclusivamente de Lima Metropolitana.  
\- Dim\_Producto: 600 registros (id\_producto, nombre, categoria, subcategoria, marca, precio\_lista, costo). Categorías base obligatorias: "Abarrotes", "Tecnologia", "Hogar", "Bebidas" y "Cuidado Personal". Asegura que precio\_lista \> costo se cumpla siempre.  
\- Dim\_Tienda: 20 registros (id\_tienda, nombre, canal \[Fisico / Online\], region, ciudad). Al menos 2 tiendas deben pertenecer al canal "Online" (Ecommerce).  
\- Dim\_Tiempo: 730 registros (2 años completos, cubriendo el periodo 2024 y 2025\) (fecha, dia, mes, trimestre, anio, dia\_semana, es\_feriado).  
\- Dim\_Promocion: 40 registros de campañas (id\_promocion, nombre, tipo, descuento\_pct \[valores entre 0.05 y 0.30\], fecha\_inicio, fecha\_fin). Incluye un registro técnico base 'PROMO\_NONE' (descuento 0.0) para transacciones sin promoción.  
\- Fact\_Ventas: 50000 líneas de venta base (id\_venta, fecha, id\_cliente, id\_producto, id\_tienda, id\_promocion, cantidad, precio\_unitario, descuento).

REGLAS DE REALISMO ANALÍTICO:  
1\. Estacionalidad temporal: Incrementa exponencialmente el volumen de transacciones diarias en los meses de julio (Fiestas Patrias) y diciembre (Navidad). Contrae la demanda en febrero y marzo.  
2\. Concentración de Pareto (80/20): Aplica distribuciones estadísticas de modo que un \~20% de los productos represente el \~80% de las ventas. Asimismo, define un grupo selecto de clientes "muy frecuentes" y una gran masa de clientes "esporádicos".  
3\. Precios y Descuentos: El 'precio\_unitario' en la tabla de hechos debe oscilar aleatoriamente en un \+/- 5% respecto al 'precio\_lista' de la dimensión producto. El 'descuento' (en porcentaje) solo se aplicará si la fecha de la venta está estrictamente dentro del rango de vigencia de su 'id\_promocion'; si no intersecta o es 'PROMO\_NONE', el descuento es 0\.  
3\< 4\. Afinidad de canasta (Market Basket Analysis): Inyecta reglas de co-ocurrencia lógicas bajo un mismo 'id\_venta'. Si un cliente adquiere un producto de "Abarrotes", eleva al 78% la probabilidad de que agregue un ítem de "Bebidas" en la misma boleta para forzar un lift predictivo \> 1\.  
5\. Lógica de Abandono (Churn): Simula matemáticamente que un \~20-25% de los clientes (con correlación a menores ingresos o segmento Basico) cesen por completo sus compras durante los últimos 4 meses del año 2025\.

PROBLEMAS DE CALIDAD A INYECTAR INTENCIONALMENTE (Solo en los archivos crudos de la carpeta ../data/raw):  
\- Formato de fechas mixto: En Fact\_Ventas y Dim\_Cliente, introduce un 2% de fechas mutadas a formatos heterogéneos ("dd/mm/aaaa", "aaaa-mm-dd" y cadenas de texto sucias como "2024/Dic/15").  
\- Valores faltantes (Nulos): Inyecta entre un 4% y 5% de valores nulos (NaN) distribuidos aleatoriamente en las columnas 'descuento' (Fact\_Ventas), 'distrito' (Dim\_Cliente) y 'categoria' (Dim\_Producto).  
\- Duplicados de transacciones: Clona exactamente entre el 1% y 2% de las filas de Fact\_Ventas para generar redundancia transaccional.  
\- Inconsistencia de texto y Errores de tipeo: En Dim\_Producto, altera algunas categorías para que aparezcan con espacios adicionales, combinaciones caóticas de mayúsculas/minúsculas y variaciones ortográficas sin acento (ej. "  aBaRroTes ", "Bebidass", "BEBIDAS").  
\- Outliers de negocio: Introduce un 0.3% de registros anómalos en Fact\_Ventas con cantidad \= 500 o precio\_unitario \= 0.0.  
\- Ruptura de Integridad Referencial (Claves huérfanas): Modifica un 0.3% de las líneas de Fact\_Ventas asignándoles el 'id\_producto' inexistente "PROD9999" para simular fallas de consistencia referencial.

FORMATO DE SALIDA Y VALIDACIÓN:  
\- Exporta cada una de las 6 tablas crudas generadas en formato CSV dentro del directorio relativo \`../data/raw/\`, utilizando obligatoriamente como separador el punto y coma (";") y codificación "utf-8".  
\- Al finalizar, el script debe imprimir en la consola un "REPORTE DE CALIDAD INICIAL \- OBLIGATORIO (FISIMart S.A.C.)" que evalúe volumetría, porcentaje exacto de nulos por atributo, cantidad neta de registros duplicados idénticos y conteo de registros huérfanos.

**B.3 Prompts de apoyo para el pipeline de ETL**  
Actúa como un Ingeniero de Datos Senior y Arquitecto de Business Intelligence. Genera el código completo de Python, estructurado celda por celda y listo para producción en un Jupyter Notebook denominado \`01\_datamart\_etl.ipynb\` dentro de la carpeta \`notebooks/\`. El objetivo es transformar y purificar los datos transaccionales crudos de "FISIMart S.A.C." para consolidar un modelo dimensional en Esquema Estrella optimizado.

El proceso de negocio a modelar es el de Ventas Omnicanal y el grano establecido es la LINEA DE VENTA (un producto específico dentro de una boleta/ticket).

El script debe estructurarse estrictamente en las siguientes celdas de Jupyter Notebook:

CELDA 1: CONTROL DE ENTORNO E IMPORTACIÓN  
\- Importa las librerías necesarias: \`pandas\`, \`numpy\`, \`os\` y \`datetime\`.  
\- Configura las rutas relativas de lectura (\`../data/raw/\`) y escritura (\`../data/processed/\`).

CELDA 2: EXTRACCIÓN Y DIAGNÓSTICO DE CALIDAD INICIAL (REPORTE ANTES DEL ETL)  
\- Carga los 6 archivos CSV desde \`../data/raw/\` usando el separador ";" y codificación "utf-8".  
\- Ejecuta una auditoría programática exhaustiva que calcule e imprima en pantalla un "Reporte de Calidad Inicial (Pre-ETL)" detallando obligatoriamente:  
  1\. El volumen total de filas por cada dataframe.  
  2\. El conteo exacto de valores nulos por columna en Clientes, Productos y Ventas.  
  3\. El número de registros duplicados exactos en Ventas.  
  4\. El número exacto de claves huérfanas (registros en ventas cuyo 'id\_producto' NO existe en el catálogo maestro de productos).

CELDA 3: PIPELINE DE TRANSFORMACIÓN (ETL) \- PARTE 1: SANEAMIENTO DE DIMENSIONES  
\- Aplica las siguientes técnicas de limpieza en las dimensiones en memoria:  
  \* Dim\_Producto: Elimina espacios en blanco en los extremos (.str.strip()), estandariza todo a mayúsculas/minúsculas correctas, y homologa los errores de tipeo en las categorías (ej. transformar "Bebidass", "BEBIDAS", "  bebidas " a la categoría limpia "Bebidas", y de igual forma con "Abarrotes"). Elimina duplicados si existieran.  
  \* Dim\_Cliente: Identifica y corrige las fechas mutadas en 'fecha\_alta'. Convierte los formatos mixtos ("dd/mm/aaaa", "aaaa-mm-dd", "aaaa/Dic/dd") a un formato datetime estándar homogéneo (\`YYYY-MM-DD\`). Trata los valores nulos en 'distrito' reemplazándolos de forma segura por la categoría "No Especificado".

CELDA 4: PIPELINE DE TRANSFORMACIÓN (ETL) \- PARTE 2: PURIFICACIÓN DE LA TABLA DE HECHOS  
\- Aplica las siguientes reglas rígidas sobre \`ventas\_raw\`:  
  \* Estandariza la columna 'fecha' convirtiendo todos los formatos de cadena mixtos a datetime real (\`YYYY-MM-DD\`).  
  \* Trata los valores nulos en la columna 'descuento' imputándoles el valor \`0.0\` de forma segura.  
  \* Elimina de forma definitiva el 1.5% de filas correspondientes a duplicados exactos.  
  \* Filtra y remueve los outliers de negocio (registros con cantidad \= 500 o precio\_unitario \= 0.0).  
  \* Resuelve la ruptura de integridad referencial: Filtra y elimina por completo las líneas de venta que contengan claves huérfanas (\`PROD9999\`), garantizando consistencia absoluta con la dimensión producto limpia.

CELDA 5: INGENIERÍA DE CARACTERÍSTICAS (MÉTRICAS DERIVADAS)  
\- En la tabla de hechos de ventas, calcula de forma vectorizada las métricas solicitadas por el negocio para habilitar el análisis financiero en Power BI:  
  \* Importe Neto: Calcule la fórmula: importe \= cantidad \* precio\_unitario \* (1 \- descuento).  
  \* Costo Total Lineal: Obtén el 'costo' unitario de la dimensión producto haciendo un merge/match por 'id\_producto' y calcula: costo\_total \= cantidad \* costo.  
  \* Margen de Ganancia: Calcule la fórmula: margen \= importe\_neto \- costo\_total.

CELDA 6: CARGA Y PERSISTENCIA EN DATAMART PROCESADO  
\- Estructura las tablas finales seleccionando únicamente las columnas requeridas para el modelo estrella:  
  \* Dim\_Cliente (id\_cliente, nombre, sexo, fecha\_nacimiento, edad, distrito, fecha\_alta, segmento\_programa, ingresos\_mensuales\_aprox)  
  \* Dim\_Producto (id\_producto, nombre, categoria, subcategoria, marca, precio\_lista, costo)  
  \* Fact\_Ventas (id\_venta, fecha, id\_cliente, id\_producto, id\_tienda, id\_promocion, cantidad, precio\_unitario, descuento, importe, costo\_total, margen)  
\- Exporta las 6 tablas del Datamart final en formato CSV dentro del directorio \`../data/processed/\`, usando como separador el punto y coma (";") y codificación "utf-8".

CELDA 7: REPORTE DE CALIDAD FINAL DE AUDITORÍA (POST-ETL)  
\- Imprime en la consola un "REPORTE DE CALIDAD FINAL \- CERTIFICADO (POST-ETL)". Debe demostrar de forma matemática que el número de nulos es 0.0%, los duplicados son 0, las claves huérfanas se redujeron a 0 y detallar el total de filas netas salvadas y listas para ser conectadas directamente a Power BI.

**B.4 Prompts de clasificación.**

Actúa como un Científico de Datos Senior especializado en Machine Learning para Retail y analítica predictiva de clientes. Genera el código completo de Python, estructurado celda por celda y listo para producción en un Jupyter Notebook denominado 03\_clasificacion.ipynb dentro de la carpeta notebooks/. El objetivo es construir un modelo de clasificación supervisada que permita responder la pregunta de negocio: "¿Qué clientes tienen mayor probabilidad de abandonar el programa de fidelización?" utilizando la información procesada del proyecto FISIMart S.A.C.

El script debe estructurarse estrictamente en las siguientes celdas del Jupyter Notebook:

CELDA 1: CONTROL DE ENTORNO E IMPORTACIÓN  
Instalar silenciosamente (-q) las librerías necesarias si no se encuentran disponibles.  
Importar pandas, numpy, matplotlib, seaborn y los módulos necesarios de scikit-learn.  
Configurar la semilla SEMILLA \= 20 para garantizar la reproducibilidad.  
Configurar rutas relativas para leer los archivos desde ../data/processed/.  
CELDA 2: CARGA E INTEGRACIÓN DE LOS DATOS  
Cargar Fact\_Ventas.csv, Dim\_Cliente.csv, Dim\_Tiempo.csv, Dim\_Producto.csv, Dim\_Tienda.csv y Dim\_Promocion.csv.  
Realizar los merge necesarios para obtener un único DataFrame analítico a nivel cliente.  
Mostrar un resumen del conjunto de datos:  
Número de clientes.  
Número de ventas.  
Variables disponibles.  
Valores nulos por columna.  
CELDA 3: CONSTRUCCIÓN DE LA VARIABLE OBJETIVO (CHURN)  
Construir la variable objetivo Churn considerando como clientes abandonados aquellos que no realizaron compras durante los últimos cuatro meses del período analizado.  
Codificar:  
Churn \= 1 (abandono)  
Churn \= 0 (cliente activo)  
Mostrar la distribución de clases mediante una tabla y un gráfico de barras.  
CELDA 4: INGENIERÍA DE CARACTERÍSTICAS

Construir variables predictoras utilizando únicamente la información disponible en el esquema estrella, tales como:

Recencia.  
Frecuencia de compra.  
Monto total gastado.  
Ticket promedio.  
Número de compras.  
Número de categorías adquiridas.  
Cantidad de promociones utilizadas.  
Edad.  
Ingreso mensual.  
Segmento del programa.  
Antigüedad del cliente.

Eliminar variables identificadoras que puedan generar fuga de información (id\_cliente, nombres, etc.).

Justificar en una celda Markdown la utilidad de las variables seleccionadas.

CELDA 5: PREPARACIÓN DEL DATASET  
Separar variables predictoras y variable objetivo.  
Codificar variables categóricas mediante One-Hot Encoding cuando corresponda.  
Escalar variables numéricas únicamente si el modelo lo requiere.  
Dividir los datos en entrenamiento y prueba utilizando una proporción 80/20 con random\_state \= SEMILLA.

Mostrar las dimensiones de cada conjunto generado.

CELDA 6: ENTRENAMIENTO DE MODELOS

Entrenar los siguientes modelos de clasificación:

Regresión Logística.  
Árbol de Decisión.  
Random Forest.

Mantener parámetros razonables para evitar sobreajuste y garantizar la reproducibilidad.

CELDA 7: EVALUACIÓN Y COMPARACIÓN DE MODELOS

Para cada modelo calcular:

Accuracy.  
Precision.  
Recall.  
F1 Score.  
ROC-AUC.

Mostrar:

Matriz de confusión.  
Curva ROC.  
Tabla comparativa con todas las métricas obtenidas.

Seleccionar automáticamente el modelo con mejor desempeño considerando principalmente el ROC-AUC y el equilibrio entre Precision y Recall.

CELDA 8: INTERPRETACIÓN DEL MODELO

Analizar el modelo seleccionado.

Si corresponde:

Mostrar la importancia de variables mediante un gráfico de barras.  
Interpretar las variables con mayor influencia sobre la probabilidad de abandono.  
Explicar los resultados desde la perspectiva del negocio y relacionarlos con el comportamiento de compra de los clientes.  
CELDA 9: CONCLUSIONES Y RECOMENDACIONES

Redactar en celdas Markdown:

Las principales conclusiones obtenidas.  
Responder explícitamente la pregunta de negocio:  
¿Qué clientes tienen mayor probabilidad de abandonar el programa de fidelización?  
Proponer estrategias de retención basadas en los resultados del modelo, tales como campañas personalizadas, promociones dirigidas, programas de fidelización y seguimiento de clientes de alto riesgo.

No desarrollar técnicas correspondientes a segmentación (RFM o clustering), reglas de asociación o modelos de regresión, ya que pertenecen a las Partes 4, 5 y 6 del proyecto.

CELDA 10: EXPORTACIÓN DE RESULTADOS  
Exportar la tabla final con las probabilidades de abandono por cliente a ../data/processed/Prediccion\_Churn.csv, utilizando separador ";" y codificación "utf-8".  
Este archivo deberá quedar listo para su posterior visualización en Power BI y para apoyar la toma de decisiones del área comercial.

**B.5. Prompt de Segmentación**

Actúa como un Científico de Datos Senior especializado en Machine Learning para Retail y analítica de clientes. Genera el código completo de Python, estructurado celda por celda y listo para producción en un Jupyter Notebook denominado 04\_segmentacion.ipynb dentro de la carpeta notebooks/. El objetivo es construir un modelo de segmentación no supervisada que permita responder la pregunta de negocio: "¿Qué segmentos de clientes existen y cómo debería tratarse a cada uno?" utilizando la información procesada del proyecto FISIMart S.A.C.

Antes de escribir código, explora el repositorio para entender la estructura real: revisa data/processed/Fact\_Ventas.csv y data/processed/Dim\_Cliente.csv imprimiendo .head(), .info() y .describe() de cada uno para conocer los nombres y tipos exactos de columnas; consulta también un notebook existente como notebooks/05\_asociacion.ipynb para replicar el estilo de imports, rutas relativas y guardado de figuras.

El script debe estructurarse estrictamente en las siguientes celdas del Jupyter Notebook:

CELDA 1: CONTROL DE ENTORNO E IMPORTACIÓN  
 Instalar silenciosamente las librerías necesarias si no se encuentran disponibles. Importar pandas, numpy, matplotlib, seaborn y los módulos necesarios de scikit-learn. Configurar la semilla SEMILLA \= 20 para garantizar la reproducibilidad. Configurar rutas relativas para leer los archivos desde ../data/processed/.

CELDA 2: CARGA E INTEGRACIÓN DE LOS DATOS  
 Cargar Fact\_Ventas.csv y Dim\_Cliente.csv. Realizar los merge necesarios para obtener un único DataFrame analítico a nivel cliente. Mostrar un resumen del conjunto de datos con número de clientes, número de ventas, variables disponibles y valores nulos por columna.

CELDA 3: CONSTRUCCIÓN DE LA TABLA RFM  
 Calcular una fila por id\_cliente con las siguientes métricas: Recency como número de días desde la última compra respecto a la fecha de corte max(fecha)+1 (no usar la fecha del sistema; el dataset termina en 2025-12-31), Frequency como número de tickets únicos (id\_venta distintos) por cliente, y Monetary como suma del importe por cliente. Asignar scores del 1 al 5 por quintiles, invirtiendo la escala de Recency para que un menor número de días corresponda a un score más alto. Justificar en una celda Markdown la interpretación de cada métrica.

CELDA 4: CLUSTERING CON K-MEANS  
 Estandarizar las variables R, F y M con StandardScaler. Determinar el número óptimo de clusters mediante el método del codo sobre la inercia y el coeficiente de silueta (silhouette score); justificar el k elegido en una celda Markdown. Ajustar el modelo final con random\_state=20 y asignar el cluster resultante a cada cliente.

CELDA 5: PERFILES DE SEGMENTOS  
 Para cada cluster, calcular el RFM promedio y cruzar con Dim\_Cliente (edad, ingresos\_mensuales\_aprox, segmento\_programa, distrito). Asignar un nombre de negocio a cada cluster (por ejemplo: Campeones, Leales, En Riesgo, Perdidos). Nota de diseño del dataset: se inyectó a propósito que aproximadamente el 23% de los clientes del segmento Básico con bajos ingresos dejaron de comprar durante los últimos cuatro meses de 2025; comentar si este patrón aparece reflejado en algún cluster con Recency muy alta.

CELDA 6: ESTRATEGIAS DE NEGOCIO  
 Redactar en una celda Markdown una recomendación de negocio concreta por segmento, orientada a maximizar el valor de vida del cliente y a mitigar la deserción.

CELDA 7: EXPORTACIÓN DE RESULTADOS Y FIGURAS  
 Guardar las figuras en reports/figures/ siguiendo la numeración existente del proyecto (curva del codo y silhouette, scatter RFM coloreado por cluster, distribución de clientes por segmento y heatmap comparativo de perfiles). Exportar la tabla de segmentación a ../data/processed/Segmentacion\_Clientes.csv con separador ";" y codificación "utf-8", lista para su visualización en Power BI.

Convenciones generales: código y comentarios en español, sin emojis, rutas relativas al root del repositorio, semilla fija random\_state=20 en KMeans en línea con la semilla global del proyecto. Al finalizar, mostrar en consola un resumen en texto de los perfiles y estrategias obtenidas.

**B.5 Prompts de apoyo para crear visualizaciones analíticas avanzadas**

Actúa como un Senior Data Analyst y experto en Python para Retail. Necesito que generes el código completo y estructurado celda por celda para el cuaderno "02\_visualizacion.ipynb" de mi proyecto FISIMart S.A.C. 

El objetivo es crear visualizaciones analíticas avanzadas utilizando la data procesada en los pasos previos (leyendo los archivos consolidados en la carpeta 'processed/' como Fact\_Ventas.csv, Dim\_Producto.csv, etc.). El diseño visual debe ser profesional, utilizando una paleta de colores coherente (tonos azules corporativos, grises y naranjas/rojos suaves para contrastes).

El cuaderno debe estar estructurado estrictamente en las siguientes celdas de código y celdas Markdown asociadas:

\--- CELDAS DE ENTORNO E INSTALACIÓN  \---  
\- Celda de Código 0: Instalar de forma silenciosa (usando el parámetro \-q) las librerías necesarias que no siempre vienen por defecto, específicamente 'plotly' y 'seaborn' (para asegurar su última versión). Inmediatamente después, importar y verificar la instalación exitosa imprimiendo un reporte limpio que muestre las versiones instaladas de: pandas, numpy, matplotlib, seaborn y plotly.

\--- CELDAS DE CONFIGURACIÓN INICIAL \---  
\- Celda de Código 1: Importación de librerías esenciales (pandas, numpy, matplotlib.pyplot, seaborn, plotly.express y plotly.graph\_objects). Configuración de estilos visuales globales (sns.set\_theme) para que los gráficos se vean limpios, minimalistas y corporativos.  
\- Celda de Código 2: Carga de los archivos CSV necesarios desde la carpeta 'processed/' y unificación o merges necesarios para tener las descripciones de categorías, subcategorías y distritos integradas en el DataFrame de hechos de ventas.

\--- CELDAS DE GRÁFICOS Y ANÁLISIS \---

\[GRÁFICO 1: EVOLUCIÓN TEMPORAL DE VENTAS Y MARGEN\]  
\- Celda de Código 3: Generar un gráfico de líneas doble eje Y utilizando Matplotlib. En el eje X debe mostrarse la línea de tiempo continua (Año-Mes). El eje Y principal debe mostrar la 'Suma de Importe' (Ventas) y el eje Y secundario debe mostrar la 'Suma de Margen'. Debe guardarse la figura automáticamente en la carpeta 'reports/figures/01\_evolucion\_temporal.png'.

\[GRÁFICO 2: ANÁLISIS DE PARETO DE PRODUCTOS (REGLA 80/20)\]  
\- Celda de Código 4: Calcular las ventas acumuladas por producto (nombre\_producto), ordenar de mayor a menor y calcular el porcentaje acumulado. Generar un gráfico combinado (barras para ventas individuales de productos y una línea roja para el porcentaje acumulado). Incluir una línea guía horizontal en el 80%. Guardar la figura en 'reports/figures/02\_pareto\_productos.png'.

\[GRÁFICO 3: DISTRIBUCIÓN DE VENTAS Y MARGEN POR CATEGORÍA\]  
\- Celda de Código 5: Crear un gráfico de barras agrupadas o horizontales que compare los ingresos totales (importe) frente al margen de ganancia obtenido para cada una de las categorías principales de la empresa (Abarrotes, Bebidas, Cuidado Personal, Hogar, Tecnología). Guardar en 'reports/figures/03\_distribucion\_categorias.png'.

\[GRÁFICO 4: MAPA DE CALOR (HEATMAP) DE ESTACIONALIDAD Y SURTIDO\]  
\- Celda de Código 6: Crear una tabla pivote que cruce 'Mes' (en las filas, ordenado de enero a diciembre) contra 'Categoría de Producto' (en las columnas), midiendo la suma de los importes vendidos. Graficarlo usando un Heatmap (Mapa de calor) de Seaborn con una paleta de colores intuitiva (ej. "YlGnBu"). Guardar la figura en 'reports/figures/04\_heatmap\_estacionalidad.png'.

**B.6 Prompts de apoyo para crear asociaciones**

Actúa como un Científico de Datos Senior especializado en Market Basket Analysis y sistemas de recomendación para retail. Genera el código completo de Python, estructurado celda por celda y listo para producción en un Jupyter Notebook denominado 05\_asociacion.ipynb dentro de la carpeta notebooks/. El objetivo es descubrir reglas de asociación a nivel producto (SKU) a partir de las transacciones de "FISIMart S.A.C." para diseñar promociones cruzadas, bundles y estrategias de venta cruzada en tienda física y canal Online.

El script debe estructurarse estrictamente en las siguientes celdas de Jupyter Notebook:

CELDA 1: CONTROL DE ENTORNO E IMPORTACIÓN  
Instala de forma silenciosa (-q) la librería mlxtend si no está disponible.  
Importa pandas, numpy, mlxtend.preprocessing.TransactionEncoder y mlxtend.frequent\_patterns (apriori, fpgrowth, association\_rules).  
Configura las rutas relativas de lectura desde ../data/processed/, dado que el notebook se ejecuta desde la carpeta notebooks/.  
Fija la semilla SEMILLA \= 20 para cualquier proceso estocástico (por ejemplo, si se hace muestreo de transacciones).

CELDA 2: CARGA DE DATOS PROCESADOS  
Carga Fact\_Ventas.csv y Dim\_Producto.csv desde ../data/processed/, usando separador ";" y codificación "utf-8".  
Realiza un merge entre Fact\_Ventas y Dim\_Producto por id\_producto para incorporar los campos nombre, categoria y subcategoria a cada línea de venta.  
Imprime un resumen inicial: número de líneas de venta, número de boletas únicas (id\_venta), número de categorías y productos distintos.

CELDA 3: CONSTRUCCIÓN DE TRANSACCIONES (CANASTA)  
Agrupa las líneas de venta por id\_venta y construye una lista de listas, donde cada sublista contiene los nombres de producto (nombre) comprados en esa boleta (nivel SKU).  
Filtra boletas con un solo ítem si se desea analizar solo canastas múltiples, justificando la decisión en una celda Markdown: las reglas de asociación requieren co-ocurrencia entre al menos dos productos distintos.  
Reporta el número total de transacciones resultantes, productos distintos en canastas y tamaño promedio de la canasta.

CELDA 4: CODIFICACIÓN ONE-HOT DE TRANSACCIONES  
Usa TransactionEncoder de mlxtend para transformar la lista de transacciones en una matriz booleana (DataFrame con una columna por producto).  
Imprime las dimensiones de la matriz resultante (transacciones × ítems).

CELDA 5: MINERÍA DE ITEMSETS FRECUENTES  
Aplica el algoritmo Apriori (o FP-Growth como alternativa de comparación) sobre la matriz booleana, usando un soporte mínimo adaptado al nivel producto: min\_support \= 0.001 (0,1 %) — justificar que, con \~550 SKUs y \~9 500 canastas multi-ítem, este umbral exige al menos \~10 co-compras por par.  
Convierte los itemsets internos de mlxtend (frozenset) a texto legible antes de mostrarlos (p. ej. "Producto Gaseosas 54", no frozenset({...})).  
Muestra los itemsets frecuentes más relevantes ordenados por soporte descendente.

CELDA 6: GENERACIÓN Y FILTRADO DE REGLAS DE ASOCIACIÓN  
Genera las reglas de asociación a partir de los itemsets frecuentes usando association\_rules, calculando soporte, confianza y lift.  
Filtra las reglas con lift \> 1 y confianza mínima \= 0.03 (3 %), umbrales adaptados al nivel SKU, donde los pares exactos se repiten mucho menos que a nivel categoría.  
Conserva solo reglas unívocas (1 antecedente \-\> 1 consecuente) accionables en tienda y Online.  
Convierte antecedents y consequents a texto legible (solo el nombre del producto, sin frozenset).  
Enriquece cada regla con categoria\_antecedente y categoria\_consecuente a partir del merge con Dim\_Producto.  
Ordena las reglas resultantes por lift descendente y muestra el top 10-15.

CELDA 7: INTERPRETACIÓN Y TRADUCCIÓN A ACCIONES DE NEGOCIO  
Redacta en celdas Markdown la interpretación de las 5–8 reglas más relevantes a nivel producto: qué SKUs se compran juntos, con qué fuerza (lift), qué tan confiable es la regla (confianza) y entre qué categorías cruzan.  
Propón acciones concretas de negocio: proximidad física de SKUs en tienda, paquetes promocionales, descuentos cruzados o recomendaciones en el canal Online.  
Incluye una sección de acciones transversales (layout, CRM, monitoreo mensual).

CELDA 8: EXPORTACIÓN PARA POWER BI  
Exporta la tabla final de reglas priorizadas (antecedents, consequents, categoria\_antecedente, categoria\_consecuente, support, confidence, lift) a un archivo CSV en ../data/processed/Reglas\_Asociacion.csv, usando separador ";" y codificación "utf-8", listo para importar en Power BI y construir la página "Canasta de mercado".

**B.7 Prompts de apoyo para crear regresiones**

Actúa como un Científico de Datos Senior especializado en pronóstico de series de tiempo (demand forecasting) para retail. Genera el código completo de Python, estructurado celda por celda y listo para producción en un Jupyter Notebook denominado 06\_regresion.ipynb dentro de la carpeta notebooks/. El objetivo es pronosticar la venta diaria de "FISIMart S.A.C." para apoyar la planificación de inventario y campañas.

El script debe estructurarse estrictamente en las siguientes celdas de Jupyter Notebook:

CELDA 1: LIBRERÍAS Y CONFIGURACIÓN  
Importa pandas, numpy, matplotlib.pyplot, seaborn, y desde scikit-learn: LinearRegression, RandomForestRegressor, GradientBoostingRegressor, y las métricas mean\_absolute\_error, mean\_squared\_error, r2\_score. Fija SEED \= 42 en numpy para reproducibilidad. Configura el estilo de gráficos (seaborn whitegrid) y el tamaño de figura por defecto.

CELDA 2: CARGA DE DATOS  
Carga Fact\_Ventas.csv, Dim\_Tiempo.csv, Dim\_Producto.csv y Dim\_Promocion.csv desde data/processed/, con separador ";" y parseo de columnas de fecha. Imprime las dimensiones de cada tabla y muestra las primeras filas de Fact\_Ventas.

CELDA 3: VALIDACIÓN RÁPIDA DE CALIDAD  
Verifica que los datos ya vienen limpios de la Parte 1: imprime el rango de fechas de Fact\_Ventas, los nulos por columna y el número de duplicados exactos.

CELDA 4: DEFINICIÓN DE LA VARIABLE OBJETIVO (SERIE DIARIA)  
Une Fact\_Ventas con la categoría del producto (merge con Dim\_Producto por id\_producto). Agrupa por fecha para construir la serie objetivo principal: ventas totales por día (suma de importe), unidades vendidas y número de transacciones únicas (id\_venta).

CELDA 5: SERIE OBJETIVO SECUNDARIA POR CATEGORÍA  
Agrupa las ventas por fecha y categoría para obtener una serie diaria de ventas por categoría de producto, que sirva de insumo opcional para un pronóstico segmentado.

CELDA 6: GRÁFICO DE LA SERIE DIARIA  
Grafica la evolución de las ventas diarias totales a lo largo de todo el periodo, como visualización exploratoria de la variable objetivo.

CELDA 7: VARIABLES TEMPORALES Y ESTACIONALIDAD CÍCLICA  
Une el calendario (Dim\_Tiempo) con la serie de ventas diarias, rellenando con 0 los días sin venta. Construye el día de la semana numérico, el indicador de fin de semana y el día del año. Calcula variables cíclicas seno/coseno para mes, día de la semana y día del año, de modo que el modelo capture la estacionalidad sin discontinuidades.

CELDA 8: INDICADOR DE PROMOCIÓN ACTIVA POR DÍA  
Calcula, por cada fecha, la proporción de líneas de venta que tuvieron una promoción real activa (distinta de "PROMO\_NONE") y la incorpora como variable predictora pct\_lineas\_con\_promo.

CELDA 9: REZAGOS (LAGS) Y MEDIAS MÓVILES  
Ordena la serie por fecha y genera los rezagos de ventas de 1, 7 y 14 días, además de las medias móviles de 7 y 14 días (calculadas sobre el histórico previo, sin fuga de información). Elimina las filas iniciales sin histórico suficiente y reporta cuántas filas quedan disponibles para modelar.

CELDA 10: MATRIZ DE CORRELACIÓN  
Genera un mapa de calor (heatmap) de correlación entre todas las variables predictoras numéricas y la variable objetivo, para identificar qué features tienen mayor relación lineal con las ventas.

CELDA 11: DIVISIÓN TRAIN/TEST TEMPORAL  
Define la lista de FEATURES y el TARGET. Divide el dataset en 80% entrenamiento / 20% prueba respetando estrictamente el orden temporal (sin mezclar aleatoriamente), de modo que el conjunto de prueba corresponda al periodo más reciente. Imprime el rango de fechas de cada partición.

CELDA 12: GRÁFICO DE LA DIVISIÓN TEMPORAL  
Grafica la serie completa distinguiendo con colores el tramo de entrenamiento y el tramo de prueba, marcando con una línea vertical el punto de corte.

CELDA 13: ENTRENAMIENTO DE MODELOS  
Entrena tres modelos de regresión sobre el conjunto de entrenamiento — Regresión Lineal, Random Forest Regressor (300 árboles) y Gradient Boosting Regressor — todos con random\_state \= SEED, y genera las predicciones de cada uno sobre el conjunto de prueba.

CELDA 14: CÁLCULO DE MÉTRICAS DE EVALUACIÓN  
Define una función que calcule MAE, RMSE, MAPE (%) y R² dados los valores reales y predichos. Aplícala a las predicciones de los tres modelos y construye una tabla comparativa ordenada por RMSE ascendente.

CELDA 15: GRÁFICO COMPARATIVO DE MÉTRICAS  
Genera un gráfico de barras con cuatro paneles (MAE, RMSE, MAPE, R²) comparando el desempeño de los tres modelos.

CELDA 16: SELECCIÓN DEL MEJOR MODELO Y GRÁFICO REAL VS. PRONÓSTICO  
Selecciona el modelo con menor RMSE como el mejor. Grafica la serie real contra la pronosticada por ese modelo en el periodo de prueba.

CELDA 17: IMPORTANCIA DE VARIABLES  
Si el mejor modelo es de árboles (Random Forest o Gradient Boosting), grafica la importancia de cada variable predictora. Si es Regresión Lineal, muestra sus coeficientes ordenados por magnitud absoluta.

CELDA 18: EXPORTACIÓN DE PRONÓSTICO PARA POWER BI  
Construye una tabla con fecha, ventas reales, el pronóstico de cada uno de los tres modelos, el pronóstico del mejor modelo, el modelo seleccionado, el error absoluto y el error porcentual. Exporta esa tabla a data/processed/pronostico\_ventas.csv con separador ";" y codificación "utf-8".

CELDA 19: EXPORTACIÓN DE MÉTRICAS  
Exporta la tabla comparativa de métricas (MAE, RMSE, MAPE, R² por modelo) a data/processed/metricas\_regresion.csv, para documentar el desempeño del modelo elegido.


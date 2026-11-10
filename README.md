# 📱 Análisis de uso de clientes – ConnectaTel

Proyecto de análisis de datos sobre el comportamiento de uso de servicios móviles (llamadas y mensajes) de los clientes de **ConnectaTel**, una empresa de telecomunicaciones con operaciones en México y Colombia.

## 🎯 Objetivo

Explorar, limpiar y analizar los datos de clientes y de uso real para:

- Identificar patrones de uso de llamadas y mensajes.
- Detectar comportamientos atípicos (posible fraude o errores de registro).
- Entender qué segmentos de clientes tienen necesidades diferenciadas.
- Proponer mejoras a la oferta de planes y a la experiencia del usuario.

### Preguntas de negocio

1. ¿Qué segmentos de clientes muestran mayor o menor uso de llamadas y mensajes?
2. ¿Qué usuarios presentan valores atípicos?
3. ¿Cómo varía el uso según la edad y el tipo de plan contratado?
4. ¿Qué patrones pueden ayudar a diseñar mejores planes y mejorar la satisfacción del cliente?

## 🗂️ Datasets

| Archivo | Filas | Descripción |
|---|---|---|
| `plans.csv` | 2 | Planes actuales: precio mensual, minutos, mensajes y GB incluidos, y costo por extra. |
| `users_latam.csv` | 4,000 | Clientes: edad, ciudad, fecha de registro, plan contratado y fecha de baja (`churn_date`). |
| `usage.csv` | 40,000 | Detalle de uso: cada registro es una llamada (`duration`) o un mensaje (`length`). Cubre de enero a junio de 2024. |

> ⚠️ **Nota:** los archivos CSV [no están incluidos / están incluidos] en este repositorio. *(Elige una opción y, si no los incluyes, indica dónde obtenerlos.)*

## 🔎 Etapas del análisis

1. **Carga y exploración:** revisión de estructura, tipos de datos y tamaño de cada dataset.
2. **Calidad de datos:** conteo de nulos, detección de valores centinela y revisión de fechas fuera de rango.
3. **Limpieza:** reemplazo de centinelas, conversión de fechas y decisión sobre los nulos.
4. **Resumen por usuario:** agregación de `usage` por `user_id` (mensajes, llamadas y minutos) y unión con `users`, en la tabla `user_profile`.
5. **Visualización:** histogramas por plan para edad, mensajes, llamadas y minutos.
6. **Valores atípicos:** boxplots y método IQR.
7. **Segmentación:** grupos por nivel de uso y por edad.
8. **Insight ejecutivo:** conclusiones y recomendaciones para stakeholders.

## 🧹 Problemas de calidad de datos y decisiones

| Columna | Problema | Magnitud | Decisión |
|---|---|---|---|
| `users.age` | Valor centinela `-999` | Distorsiona la media (33.7 frente a una mediana de 47) | Reemplazado por la mediana |
| `users.city` | Nulos y valor `?` | 469 nulos (11.7%) + 96 registros con `?` = 565 sin ciudad | `?` convertido a nulo; los nombres abreviados (`GDL`, `MTY`, `CDMX`) se mantienen sin estandarizar |
| `users.reg_date` | Fechas posteriores a junio de 2024 | 711 registros fuera de rango | Convertidas a nulo |
| `users.churn_date` | 88.3% nulos (3,534 usuarios) | No es un error: indica clientes activos | Conservada |
| `usage.date` | Nulos | 50 registros (0.125%) | Dejados como nulos |
| `usage.duration` / `usage.length` | 55.2% y 44.7% de nulos | No son errores: `duration` solo aplica a llamadas y `length` solo a mensajes | No se imputan; se verificó contra `type` |

## 📊 Principales hallazgos

- **Distribución de planes:** Básico concentra el 64.9% de los clientes (2,595) y Premium el 35.1% (1,405).
- **Edad:** las edades van de 18 a 79 años con una distribución casi uniforme, y la proporción de Premium se mantiene sin una tendencia clara entre grupos.
- **Uso típico (mediana):** 5 mensajes, 4 llamadas y 19 minutos de llamada por usuario. Las distribuciones tienen sesgo a la derecha.
- **Plan frente a uso:** Básico y Premium muestran distribuciones de uso con la misma forma y el mismo centro, así que el volumen de uso no distingue un plan del otro.
- **Valores atípicos (IQR, solo límite superior):**

| Variable | Límite superior | Máximo |
|---|---|---|
| Mensajes | 11.5 | 17 |
| Llamadas | 10.5 | 15 |
| Minutos de llamada | 61 | 155 |

Se mantuvieron porque son valores plausibles de clientes de uso intensivo. Los casos extremos de minutos se recomienda validarlos por posible fraude o error de registro.

## 🧩 Segmentación

**Por nivel de uso** (`grupo_uso`):

| Grupo | Condición |
|---|---|
| Bajo uso | llamadas < 5 **y** mensajes < 5 |
| Uso medio | llamadas < 10 **y** mensajes < 10 (y no es bajo uso) |
| Alto uso | el resto |

**Por edad** (`grupo_edad`):

| Grupo | Condición |
|---|---|
| Joven | edad < 30 |
| Adulto | 30 ≤ edad < 60 |
| Adulto Mayor | edad ≥ 60 |

## 💡 Recomendaciones

1. Revisar la propuesta de valor de Premium frente a Básico, comparando el uso real con lo incluido en cada plan.
2. Evaluar un plan más económico para clientes de bajo uso.
3. Diseñar paquetes o complementos para el segmento de alto uso.
4. Implementar alertas para usuarios con uso extremo (posible fraude o error de registro).
5. Agregar validaciones en el registro (edad, catálogo cerrado de ciudades, fechas en rango).
6. Como siguiente paso, comparar el uso de clientes que cancelaron con el de los activos para identificar señales de abandono.

## ▶️ Cómo ejecutar el notebook

### Requisitos

- Python 3.9 o superior
- `pandas`, `numpy`, `seaborn`, `matplotlib`

```bash
pip install pandas numpy seaborn matplotlib jupyter
```

### Pasos

1. Clona el repositorio:
   ```bash
   git clone <URL-DEL-REPOSITORIO>
   cd <NOMBRE-DEL-REPOSITORIO>
   ```
2. Coloca `plans.csv`, `users_latam.csv` y `usage.csv` en una carpeta `datasets/`.
3. En el notebook, las rutas de carga están como `/datasets/plans.csv`. Ajústalas a `datasets/plans.csv` (o a la ruta donde guardaste los archivos).
4. Abre `S7 Version-Estudiante-Project-ConnectaTel_version_final.ipynb` con Jupyter y ejecuta todas las celdas en orden.

### En Google Colab

1. Abre el notebook desde GitHub (**File → Open notebook → GitHub**).
2. Sube los tres CSV al entorno de Colab y ajusta las rutas de carga.
3. Ejecuta todas las celdas.

## 📁 Estructura del repositorio

```
├── README.md
├── S7 Version-Estudiante-Project-ConnectaTel_version_final.ipynb
└── datasets/          # (opcional) archivos CSV
```

## ⚠️ Limitaciones

- El análisis cubre el uso registrado de enero a junio de 2024.
- No se dispone de datos de ingresos por cliente, por lo que la identificación de los segmentos más valiosos es una hipótesis que debe validarse con `plans.csv` (por ejemplo, cuántos usuarios exceden lo incluido en su plan).
- Un usuario no tiene registros de uso, por lo que `user_profile` tiene 3,999 valores de uso para 4,000 clientes.

## 👤 Autor
**Edgar Arturo Acosta Arenas**
-Linkedin [https://www.linkedin.com/in/edgaracostarenas/]

**[Tu nombre]** – Proyecto del bootcamp de Análisis de Datos
[LinkedIn / GitHub]

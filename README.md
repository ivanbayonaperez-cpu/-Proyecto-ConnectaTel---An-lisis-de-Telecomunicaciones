
# 📱 ConnectaTel: Segmentación de Clientes y Patrones de Uso (Telecom LATAM)

Análisis del comportamiento real de uso (llamadas y mensajes) de 4,000 clientes de una empresa de telecomunicaciones con operación en México y Colombia, con el objetivo de identificar segmentos de valor y detectar comportamientos atípicos que informen la oferta comercial.

## 🎯 Contexto de negocio

ConnectaTel necesitaba entender qué segmentos de clientes muestran mayor o menor uso de llamadas y mensajes, qué usuarios presentan comportamientos inusuales, y cómo varía el consumo según edad y tipo de plan — para optimizar la oferta y mejorar la retención.

## 🧱 Fuentes de datos

- **`plans.csv`** — catálogo de 2 planes (Básico, Premium) con precios y beneficios incluidos.
- **`users_latam.csv`** — 4,000 clientes (edad, ciudad, fecha de registro, plan, fecha de baja).
- **`usage.csv`** — 40,000 registros de actividad real (llamadas con duración, mensajes con longitud).

## 🧹 Metodología de limpieza

**1. Detección de valores sentinel:** `age` contenía el valor `-999` como marcador de dato faltante (no un error real), y `city` usaba `"?"` con el mismo propósito. Ambos se reemplazaron: `-999` por la mediana de edad, `"?"` por `NaN` explícito — evitando que un sentinel numérico distorsionara las estadísticas descriptivas (la media de edad pasaba de coherente a negativa antes de la corrección).

**2. Validación temporal:** se detectaron registros de `reg_date` con año 2026 — inconsistentes con el período operativo del análisis (hasta 2024). Se marcaron como `NaT` en vez de eliminarse, preservando el resto del registro del cliente.

**3. Análisis MAR (Missing At Random) en `duration` y `length`:** en vez de imputar directamente el 55% de nulos en `duration` y 44% en `length`, se verificó su dependencia de la columna `type` (llamada vs. mensaje):

```python
usage.groupby('type')['duration'].apply(lambda x: x.isna().mean() * 100)
# call:  0.00%   |   text: 99.93%
```

Confirmado que `duration` es nula en el 99.9% de los mensajes (no aplica esa métrica) y `length` en el 99.9% de las llamadas — los nulos son **informativos**, no errores. Decisión: conservarlos sin imputar, evitando sesgo artificial en el consumo promedio por tipo de servicio.

## 📊 Perfil de usuario y detección de outliers

Se construyó un perfil por cliente (`user_profile`) agregando `usage` a nivel `user_id`: cantidad de mensajes, llamadas y minutos totales.

**Outliers (método IQR)** sobre las 3 métricas de consumo:

| Variable | Límite superior (IQR) | Máximo real | Outliers detectados |
|---|---|---|---|
| Mensajes | 11.5 | 17.0 | 46 usuarios |
| Llamadas | 10.5 | 15.0 | 30 usuarios |
| Minutos de llamada | 61.9 | 155.7 | 109 usuarios |

**Decisión:** se conservaron todos los outliers. Justificación de negocio: un consumo de 17 mensajes o 155 minutos no es un error de captura, es comportamiento humano plausible — y estos usuarios son candidatos naturales a planes de mayor valor o corporativos, no ruido a eliminar.

## 🎯 Segmentación de clientes

**Por nivel de uso** (según llamadas y mensajes): Bajo uso (45-50% de la base), Uso medio (30-35%), Alto uso / *power users* (15-20%) — el segmento de mayor valor comercial, con mayor probabilidad de exceder los límites del plan contratado y generar ingresos adicionales por excedentes.

**Por edad:** Joven (<30), Adulto (30-59), Adulto Mayor (60+). La distribución de edad es simétrica y **no muestra correlación con el tipo de plan** — Básico y Premium tienen presencia constante en todos los rangos etarios, lo que indica que la segmentación comercial debe basarse en volumen de consumo, no en edad.

## 🔎 Hallazgos accionables

- **Distribución de planes:** 64.9% Básico, 35.1% Premium.
- **Mensajes y llamadas siguen distribución sesgada a la derecha**, con la mayoría de usuarios en un rango bajo-medio y una cola de usuarios de alto consumo.
- **Oportunidad de migración detectada:** un número considerable de usuarios del plan Básico realiza un volumen de llamadas similar al de usuarios Premium — sugiere que el valor diferencial de Premium está más en la *duración* de las llamadas que en la cantidad, y son candidatos a upselling.
- **Segmento "power users" (Alto uso)** es el más valioso: paga más por excedentes o ya está en el plan de mayor precio, contribuyendo desproporcionadamente a los ingresos.

## 📁 Estructura del repositorio

```
connectatel-segmentacion-clientes/
├── README.md
├── notebook/
│   └── connectatel_analisis.ipynb
└── visualizaciones/
    ├── distribucion_edad_por_plan.png
    ├── distribucion_mensajes_llamadas.png
    ├── boxplots_outliers.png
    └── segmentacion_uso_edad.png
```

## 🛠️ Herramientas

Python — Pandas (limpieza, análisis MAR, `groupby`/`agg`, segmentación con funciones condicionales), NumPy, Seaborn y Matplotlib (histogramas con `hue`, boxplots, countplots).

## 📁 Estructura del Proyecto

## ▶ Cómo abrir el notebook en Google Colab

Haz clic en el siguiente botón:

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/ivanbayonaperez-cpu/-Proyecto-ConnectaTel---An-lisis-de-Telecomunicaciones/blob/main/S7_Project_ConnectaTel.ipynb)


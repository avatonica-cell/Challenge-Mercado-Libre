# 🔐 UEBA — Detección de Anomalías en Inicios de Sesión

Prueba técnica de Data Science para Mercado Libre orientada a ciberseguridad. El objetivo es detectar 
inicios de sesión sospechosos mediante análisis estadístico y machine learning.

## 📁 Estructura
- `Ejercicio-1.ipynb` — notebook principal con todo el análisis
- `user_logins.csv` — dataset original (18.941 registros, 100 usuarios, 127 días)

## 🔍 Parte A — Análisis Estadístico y Etiquetado

Se construyó un perfil de comportamiento normal (baseline) para cada usuario 
usando el 80% más antiguo de su historial. Se definieron 6 reglas de anormalidad:

- **R1** Hora inusual — login fuera del rango horario habitual del usuario (percentil 5-95)
- **R2** IP nueva — IP no vista en el período de entrenamiento
- **R3** Device inusual — dispositivo no habitual para ese usuario
- **R4** Horario nocturno — login entre 00:00 y 05:59
- **R5** Alta frecuencia — más de 5 logins en una ventana de 1 hora
- **R6** Fin de semana — login en fin de semana si el usuario nunca lo hace

Resultado: 73.6% logins normales / 26.4% anómalos (ratio 2.8:1)

Los reportes de los 3 usuarios más sospechosos fueron generados automáticamente 
usando la API de Gemini 2.5 Flash.

## 🤖 Parte B — Modelo de Machine Learning

Se entrenaron y compararon dos modelos de ensamble con búsqueda de hiperparámetros 
via **Optuna** (búsqueda bayesiana):

| Modelo | F1 | ROC-AUC | PR-AUC | Recall anomalías |
|---|---|---|---|---|
| Random Forest + class_weight | 0.92 | 0.971 | 0.959 | 86% |
| LightGBM + class_weight | 0.93 | 0.970 | 0.958 | 88% |
| LightGBM + SMOTE | 0.93 | 0.971 | 0.958 | 88% |

**Modelo seleccionado: LightGBM con class_weight='balanced'**

Con un desbalance de 2.8:1, SMOTE no aportó mejora significativa respecto a 
class_weight, que resultó más simple, más rápido e igual de efectivo.


# 🛡️ Análisis de Vulnerabilidades (CVE) con Web Scraping & Looker Studio
Este proyecto forma parte de una prueba técnica de Data Science orientada a ciberseguridad. El objetivo es recolectar, procesar y visualizar vulnerabilidades recientes (CVE) para identificar tendencias críticas de seguridad en diferentes fabricantes y tecnologías.

## 📁 Estructura
Ejercicio-2.ipynb — Notebook con el proceso de Web Scraping, limpieza y exportación.

vulnerabilidades_cve.csv — Dataset generado con los datos de los últimos 3 meses.

Dashboard Interactivo — https://lookerstudio.google.com/s/pXkvU9dY7iA

## 🕷️ Parte A — Web Scraping & Data Cleaning
Se desarrolló un scraper en Python para extraer información de CVEDetails.

- Alcance: Vulnerabilidades publicadas en los últimos 3 meses.

- Campos extraídos: CVE ID, Fecha de publicación, CVSS Score (Base), Tipo de vulnerabilidad, Vendor y Producto afectado.

Manejo de Datos:

- Normalización de Fechas: Se transformó el formato de texto de origen a ISO 8601 (YYYY-MM-DD) para permitir análisis temporales.

- Enriquecimiento: Se creó una columna de Categoría de Vulnerabilidad simplificada para mejorar la legibilidad de los reportes.

## 📊 Parte B — Dashboard de Ciberseguridad
Se construyó un tablero interactivo en Looker Studio para transformar los datos crudos en insights accionables.

Metodología de Visualización
Análisis Temporal (Línea de Tiempo): Se implementó una Media Móvil (7 períodos) para suavizar la volatilidad diaria y detectar la verdadera tendencia en el volumen de reportes.

Categorización de Severidad: Se utilizó lógica de negocio (vía Campos Calculados) para clasificar los scores según el estándar CVSS v3.1:

Crítico (9.0 - 10.0) | Alto (7.0 - 8.9) | Medio (4.0 - 6.9) | Bajo (0.1 - 3.9).

Interactividad: El dashboard incluye filtros dinámicos por Fecha y Vendor, permitiendo realizar drill-down sobre fabricantes específicos (ej. Microsoft, Linux, Google).

## 💡 Insights de Seguridad Detectados
Concentración de Riesgo: La categoría de "Ejecución remota" presenta el Score promedio más alto (8.48), lo que indica que, aunque no sea la más frecuente, es la que requiere atención inmediata.

Volumen vs. Gravedad: Se observó que las vulnerabilidades de tipo "Inyección" son las más recurrentes, representando el volumen principal de casos reportados en el trimestre analizado.

Tendencia de Publicación: A través de la media móvil, se identificó un ritmo constante de reportes sin picos anómalos significativos, lo que sugiere un flujo estable de descubrimiento de vulnerabilidades por parte de la comunidad.

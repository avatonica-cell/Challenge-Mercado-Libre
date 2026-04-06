# 🔐 UEBA — Detección de Anomalías en Inicios de Sesión

Prueba técnica de Data Science para Mercado Libre orientada a ciberseguridad. El objetivo es detectar 
inicios de sesión sospechosos mediante análisis estadístico y machine learning.

## 📁 Estructura
- `Ejercicio-1.ipynb` — notebook principal con todo el análisis
- `user_logins.csv` — dataset original (18.941 registros, 100 usuarios, 127 días)

## 🔍 Parte A — Análisis Estadístico y Etiquetado

Se construyó un perfil de comportamiento normal (baseline) para cada usuario 
usando el 80% más antiguo de su historial. Se definieron 6 reglas de normalidad:

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
| Random Forest + class_weight | 0.92 | 0.971 | 0.959 | 85.5% |
| LightGBM + class_weight | 0.93 | 0.970 | 0.958 | 87.5% |
| LightGBM + SMOTE | 0.93 | 0.971 | 0.958 | 87.5% |

**Modelo seleccionado: LightGBM con class_weight='balanced'**

Con un desbalance de 2.8:1, SMOTE no aportó mejora significativa respecto a 
class_weight, que resultó más simple, más rápido e igual de efectivo.

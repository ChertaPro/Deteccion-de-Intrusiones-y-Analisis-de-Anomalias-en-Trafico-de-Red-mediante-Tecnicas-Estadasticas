# Detección-de-Intrusiones-y-Análisis-de-Anomalías-en-Tráfico-de-Red-mediante-Técnicas-Estadísticas
La seguridad informática enfrenta grandes retos hoy. Los IDS basados en firmas fallan ante ataques nuevos o zero-day. Este enfoque estadístico analiza el comportamiento del tráfico para detectar anomalías sin depender de firmas conocidas, mejorando la identificación de amenazas en entornos cambiantes y reforzando la protección de la infraestructura

📋 ESTRUCTURA COMPLETA DEL PROYECTO

SECCIÓN 1: INTRODUCCIÓN Y PREPARACIÓN
1.1 Contexto y Motivación

✅ Justificación del enfoque estadístico en detección de intrusiones
✅ Limitaciones de IDS tradicionales basados en firmas

1.2 Objetivos del Análisis

✅ Pregunta 1 (Análisis Comparativo): ¿Existen diferencias estadísticamente significativas en el comportamiento de variables de flujo de red (src_bytes, dst_bytes, duration) entre tráfico normal y los distintos tipos de ataques (DoS, Probe, R2L, U2R)?
✅ Pregunta 2 (Reducción Dimensional): ¿Es posible reducir la dimensionalidad de las 41 características mediante PCA conservando ≥95% de varianza explicada, y cómo impacta en la visualización y separación de ataques?
✅ Pregunta 3 (Clasificación Comparativa): ¿Qué técnica (Regresión Logística o K-NN) ofrece mayor sensibilidad para detectar ataques raros (U2R) vs. comunes (DoS)?

1.3 Dataset: NSL-KDD

✅ Fuente y características principales
✅ Descripción de variables

1.4 Carga de Datos

✅ Carga de train y test
✅ Creación variable binaria is_attack

1.5 Mapeo de Categorías de Ataque

✅ Mapeo de 39 ataques específicos → 5 categorías (Normal, DoS, Probe, R2L, U2R)
✅ Verificación de mapeo
✅ Distribución de categorías

1.6 Preparación de Muestra Estratificada

✅ Muestra de 5,000 observaciones para visualizaciones pesadas
✅ Verificación de estratificación


SECCIÓN 2: ANÁLISIS EXPLORATORIO DE DATOS (EDA)
2.1 Información General del Dataset ✅ COMPLETADO

✅ Dimensiones y tipos de datos
✅ Verificación de valores nulos
✅ Uso de memoria
✅ Estadísticos descriptivos de variables clave
✅ Asimetría (Skewness) y Curtosis
✅ Hallazgos e implicaciones

2.2 Análisis de la Variable Objetivo ⏳ SIGUIENTE

✅ Distribución binaria (Normal vs. Ataque)
✅ Distribución por categorías (Normal, DoS, Probe, R2L, U2R)
✅ Análisis de ataques específicos por categoría
✅ Comparación train vs. test
✅ Hallazgos sobre desbalance de clases

2.3 Análisis Univariado - Variables Numéricas ✅ PENDIENTE

✅ Estadísticos descriptivos por categoría de ataque
✅ Histogramas (escala log) de variables clave
✅ Boxplots comparativos por tipo de ataque
✅ Violin plots (opcional)
✅ Hallazgos: diferencias visuales entre categorías

2.4 Análisis Univariado - Variables Categóricas ✅ PENDIENTE

✅ Distribución de protocol_type
✅ Distribución de service (top 15)
✅ Distribución de flag
✅ Crosstabs: categorías × tipo de ataque
✅ Heatmaps de frecuencias
✅ Hallazgos: patrones en variables categóricas

2.5 Análisis Bivariado y Correlaciones ✅ PENDIENTE

✅ Matriz de correlación (heatmap)
✅ Scatterplots clave (src_bytes vs dst_bytes, etc.)
✅ Pairplot de variables principales (muestra estratificada)
✅ Identificación de multicolinealidad
✅ Hallazgos: relaciones entre variables

2.6 Análisis de Valores Atípicos (Outliers) ✅ PENDIENTE

✅ Cuantificación mediante método IQR
✅ % de outliers por variable
✅ Decisión: mantener vs. eliminar
✅ Justificación: outliers = ataques legítimos

2.7 Análisis Específico por Categoría de Ataque ✅ PENDIENTE

✅ Perfiles de cada categoría (DoS, Probe, R2L, U2R)
✅ Comparación de medias entre categorías
✅ Variables más distintivas por categoría
✅ Gráficos comparativos

2.8 Síntesis de Hallazgos del EDA ✅ PENDIENTE

✅ Resumen ejecutivo de descubrimientos
✅ Conexión con preguntas de investigación
✅ Decisiones fundamentadas para Sección 3


SECCIÓN 3: PREPARACIÓN DE DATOS
3.1 Manejo de Valores Faltantes 🔲 PENDIENTE

🔲 Verificación (ya sabemos que no hay)
🔲 Tratamiento de attack_category faltante en test

3.2 Codificación de Variables Categóricas 🔲 PENDIENTE

🔲 One-Hot Encoding para protocol_type y flag
🔲 Estrategia para service (66 categorías): Target Encoding o agrupación
🔲 Label Encoding para attack_category

3.3 Transformación de Variables Numéricas 🔲 PENDIENTE

🔲 Transformación logarítmica (log1p) para variables asimétricas
🔲 Estandarización (StandardScaler) para PCA y K-NN
🔲 Justificación de cada transformación

3.4 División y Estratificación 🔲 PENDIENTE

🔲 Uso de train/test proporcionados (ya separados)
🔲 Creación de validation set si es necesario
🔲 Estratificación por categoría

3.5 Resumen de Preparación 🔲 PENDIENTE

🔲 Pipeline de transformaciones aplicadas
🔲 Dimensiones finales de datasets
🔲 Variables listas para modelado


SECCIÓN 4: APLICACIÓN DE TÉCNICAS ESTADÍSTICAS
4.1 Pruebas de Hipótesis (Pregunta 1) 🔲 PENDIENTE
4.1.1 Formulación de Hipótesis

🔲 H₀: No hay diferencias en src_bytes entre Normal y ataques
🔲 H₁: Sí hay diferencias significativas
🔲 (Repetir para dst_bytes y duration)

4.1.2 Verificación de Supuestos

🔲 Prueba de normalidad (Shapiro-Wilk o Kolmogorov-Smirnov)
🔲 Decisión: ANOVA vs. Kruskal-Wallis

4.1.3 Ejecución de Pruebas

🔲 Kruskal-Wallis para cada variable (src_bytes, dst_bytes, duration)
🔲 Comparaciones post-hoc (Dunn test) entre pares de categorías
🔲 Cálculo de p-values y tamaño de efecto

4.1.4 Interpretación de Resultados

🔲 Respuesta a Pregunta 1: ¿Hay diferencias significativas?
🔲 Qué variables diferencian más entre categorías
🔲 Significancia estadística vs. práctica


4.2 Análisis de Componentes Principales - PCA (Pregunta 2) 🔲 PENDIENTE
4.2.1 Preparación para PCA

🔲 Estandarización de variables (ya hecho en 3.3)
🔲 Selección de variables numéricas (excluir categóricas codificadas opcionalmente)

4.2.2 Aplicación de PCA

🔲 Fit de PCA con todas las componentes
🔲 Scree plot: varianza explicada por componente
🔲 Varianza acumulada

4.2.3 Determinación de Número de Componentes

🔲 ¿Cuántas componentes para ≥95% varianza?
🔲 Comparación: 80%, 90%, 95%, 99%

4.2.4 Visualización en Espacio Reducido

🔲 Proyección en 2D (PC1 vs PC2)
🔲 Proyección en 3D (PC1, PC2, PC3)
🔲 Colorear por categoría de ataque
🔲 ¿Se separan visualmente las categorías?

4.2.5 Interpretación de Componentes

🔲 Loadings: ¿Qué variables contribuyen más a cada PC?
🔲 Interpretación semántica de PC1, PC2, PC3

4.2.6 Respuesta a Pregunta 2

🔲 Número de componentes para 95% varianza
🔲 Reducción lograda (41 → X variables)
🔲 Impacto en separación visual de ataques


4.3 Clasificación: Regresión Logística y K-NN (Pregunta 3) 🔲 PENDIENTE
4.3.1 Preparación de Datos para Clasificación

🔲 Features (X): Variables transformadas y codificadas
🔲 Target (y): attack_category
🔲 Decisión: clasificación multiclase (5 categorías) o binaria por categoría

4.3.2 Modelo 1: Regresión Logística

🔲 Entrenamiento con regularización (L2)
🔲 Predicciones en test
🔲 Matriz de confusión
🔲 Métricas por clase: Precision, Recall, F1-Score
🔲 Especial énfasis en recall de U2R y DoS

4.3.3 Modelo 2: K-Nearest Neighbors (K-NN)

🔲 Selección de K óptimo (validación cruzada)
🔲 Entrenamiento con datos estandarizados
🔲 Predicciones en test
🔲 Matriz de confusión
🔲 Métricas por clase: Precision, Recall, F1-Score
🔲 Especial énfasis en recall de U2R y DoS

4.3.4 Comparación de Modelos

🔲 Tabla comparativa de métricas
🔲 Gráficos comparativos (barras de recall por categoría)
🔲 Análisis de errores: ¿Dónde falla cada modelo?

4.3.5 Respuesta a Pregunta 3

🔲 ¿Qué modelo tiene mayor recall en U2R?
🔲 ¿Qué modelo tiene mayor recall en DoS?
🔲 Trade-offs: sensibilidad vs. especificidad
🔲 Recomendación justificada


4.4 Técnica Adicional (Opcional): Clustering K-Means 🔲 OPCIONAL

🔲 Aplicación de K-Means (k=5)
🔲 ¿Los clusters coinciden con categorías reales?
🔲 Índice de Silhouette
🔲 Visualización de clusters vs. categorías reales


SECCIÓN 5: RESULTADOS Y CONCLUSIONES
5.1 Resumen de Hallazgos Principales 🔲 PENDIENTE

🔲 Síntesis del EDA
🔲 Síntesis de técnicas estadísticas

5.2 Respuestas a las Preguntas de Investigación 🔲 PENDIENTE
Pregunta 1:

🔲 Respuesta: Sí/No, con p-values
🔲 Variables más discriminativas
🔲 Categorías más diferentes

Pregunta 2:

🔲 Respuesta: X componentes para 95% varianza
🔲 Reducción: 41 → X variables
🔲 Separación visual: clara/moderada/pobre

Pregunta 3:

🔲 Respuesta: Modelo X tiene mejor recall en U2R
🔲 Modelo Y tiene mejor recall en DoS
🔲 Recomendación según contexto

5.3 Limitaciones del Estudio 🔲 PENDIENTE

🔲 Dataset sintético (no tráfico real actual)
🔲 Desbalance extremo de U2R (11 muestras en train)
🔲 Técnicas básicas (no ensembles, no deep learning)
🔲 Validación temporal no considerada

5.4 Propuestas de Extensión 🔲 PENDIENTE

🔲 Técnicas de balanceo (SMOTE, ADASYN)
🔲 Modelos más complejos (Random Forest, XGBoost)
🔲 Validación con datos reales
🔲 Análisis temporal (series de tiempo)
🔲 Feature engineering avanzado

5.5 Conclusión General 🔲 PENDIENTE

🔲 Síntesis de contribuciones del proyecto
🔲 Aplicabilidad práctica
🔲 Reflexión metodológica


DOCUMENTOS ADICIONALES
Presentación (12 diapositivas máximo) 🔲 PENDIENTE

🔲 Portada
🔲 Introducción y motivación
🔲 Dataset y preguntas de investigación
🔲 Hallazgos del EDA (2 slides)
🔲 Pregunta 1: Pruebas de hipótesis
🔲 Pregunta 2: PCA y visualización
🔲 Pregunta 3: Comparación de clasificadores (2 slides)
🔲 Limitaciones
🔲 Conclusiones y recomendaciones
🔲 Preguntas

Exposición Oral (10-12 minutos) 🔲 PENDIENTE

🔲 Script de presentación
🔲 Ensayo y timing
🔲 Preparación para preguntas
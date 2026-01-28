# 📚 TEORÍA ESTADÍSTICA COMPLETA - SECCIÓN 4

## **ESTRUCTURA DE LA SECCIÓN 4**

La Sección 4 aplica tres técnicas estadísticas para responder tres preguntas de investigación:

1. **Test de Kruskal-Wallis** (P1) → ¿Existen diferencias significativas entre categorías?
2. **Análisis de Componentes Principales - PCA** (P2) → ¿Podemos reducir dimensionalidad?
3. **Regresión Logística** (P3) → ¿Podemos predecir categorías de ataque?

---

# 📊 PARTE 1: TEST DE KRUSKAL-WALLIS

## **1.1 ¿QUÉ ES EL TEST DE KRUSKAL-WALLIS?**

El **Test de Kruskal-Wallis H** es la alternativa **no paramétrica** al ANOVA de un factor. Compara las **distribuciones** de una variable cuantitativa entre **k ≥ 3 grupos independientes**.

### **Diferencias con ANOVA:**

| Aspecto | ANOVA | Kruskal-Wallis |
|---------|-------|----------------|
| **Supuestos** | Normalidad, homogeneidad de varianzas | No requiere normalidad |
| **Datos** | Usa valores originales | Usa **rangos** (rankings) |
| **Sensibilidad** | Sensible a outliers | Robusto a outliers |
| **Potencia** | Alta si supuestos se cumplen | Menor potencia, pero más robusto |
| **Hipótesis** | Compara **medias** | Compara **distribuciones** |

### **¿Por qué usar Kruskal-Wallis en nuestro proyecto?**

En el EDA (Sección 2.2) detectamos:
- **68.8% de variables con asimetría extrema** (|Skew| ≥ 1.0)
- **Curtosis muy alta** (colas pesadas)
- **Outliers masivos** (18.6% en dst_bytes)

Estos hallazgos **violan el supuesto de normalidad** de ANOVA, haciendo a Kruskal-Wallis la elección apropiada.

---

## **1.2 FUNDAMENTOS MATEMÁTICOS**

### **Paso 1: Transformación a Rangos**

Dadas $n$ observaciones totales de una variable $X$ distribuidas en $k$ grupos:

1. **Ignorar** temporalmente las etiquetas de grupo
2. **Ordenar** todas las $n$ observaciones de menor a mayor
3. **Asignar rangos:** 1 al menor, 2 al segundo menor, ..., $n$ al mayor
4. Si hay **empates**, asignar el **rango promedio**

**Ejemplo:**
```
Valores originales: 3, 7, 3, 9, 5
Ordenados:          3, 3, 5, 7, 9
Rangos sin empates: 1, 2, 3, 4, 5
Rangos con empates: 1.5, 1.5, 3, 4, 5  (3 aparece 2 veces → rango promedio (1+2)/2 = 1.5)
```

### **Paso 2: Calcular Suma de Rangos por Grupo**

Para cada grupo $i$ (donde $i = 1, 2, ..., k$):

$$R_i = \sum_{j=1}^{n_i} r_{ij}$$

Donde:
- $R_i$ = Suma de rangos en el grupo $i$
- $n_i$ = Número de observaciones en el grupo $i$
- $r_{ij}$ = Rango de la observación $j$ en el grupo $i$

### **Paso 3: Estadístico H de Kruskal-Wallis**

$$H = \frac{12}{n(n+1)} \sum_{i=1}^{k} \frac{R_i^2}{n_i} - 3(n+1)$$

Donde:
- $n = \sum_{i=1}^{k} n_i$ = Total de observaciones
- $k$ = Número de grupos

**Corrección para empates:**

Si hay empates, se aplica factor de corrección:

$$H_{corr} = \frac{H}{1 - \frac{\sum_{i=1}^{g} (t_i^3 - t_i)}{n^3 - n}}$$

Donde:
- $g$ = Número de grupos de empates
- $t_i$ = Tamaño del i-ésimo grupo de empates

### **Paso 4: Distribución del Estadístico H**

Bajo la hipótesis nula $H_0$ (todas las distribuciones son iguales), el estadístico $H$ sigue aproximadamente una **distribución Chi-cuadrado** con $k-1$ grados de libertad:

$$H \sim \chi^2_{k-1}$$

Esta aproximación es válida cuando cada grupo tiene al menos 5 observaciones.

---

## **1.3 HIPÓTESIS DEL TEST**

### **Hipótesis Nula (H₀):**

Las $k$ poblaciones tienen **distribuciones idénticas** de la variable $X$.

Formalmente:
$$H_0: F_1(x) = F_2(x) = ... = F_k(x) \quad \forall x$$

Donde $F_i(x)$ es la función de distribución acumulada del grupo $i$.

**Interpretación:** No hay diferencias sistemáticas entre grupos.

### **Hipótesis Alternativa (H₁):**

**Al menos una** de las distribuciones difiere de las demás.

$$H_1: \exists \, i, j \, \text{tal que} \, F_i(x) \neq F_j(x)$$

**Importante:** H₁ **NO especifica** cuál grupo es diferente, solo que hay diferencia.

---

## **1.4 INTERPRETACIÓN DEL P-VALOR**

El **p-valor** es la probabilidad de observar un estadístico H tan extremo (o más) como el observado, **asumiendo que H₀ es verdadera**.

$$p = P(H \geq H_{obs} \mid H_0 \text{ es verdadera})$$

**Regla de decisión:**

- Si $p < \alpha$ (típicamente $\alpha = 0.05$): **Rechazar H₀**
  - Conclusión: Hay evidencia suficiente de diferencias entre grupos
  
- Si $p \geq \alpha$: **No rechazar H₀**
  - Conclusión: No hay evidencia suficiente de diferencias

### **Interpretación de p-valores extremos:**

En nuestro proyecto:
- $p < 1 \times 10^{-10}$ en TODAS las variables

**Significado:** La probabilidad de observar diferencias tan extremas por casualidad es prácticamente **cero** (1 en 10 mil millones).

---

## **1.5 TAMAÑO DEL EFECTO: EPSILON-SQUARED (ε²)**

El p-valor solo indica **significancia estadística**, no la **magnitud práctica** de la diferencia.

El **Epsilon-squared (ε²)** es una medida de tamaño del efecto para Kruskal-Wallis:

$$\varepsilon^2 = \frac{H - k + 1}{n - k}$$

Donde:
- $H$ = Estadístico Kruskal-Wallis
- $k$ = Número de grupos
- $n$ = Total de observaciones

### **Interpretación de ε²:**

| Valor ε² | Interpretación | Implicación |
|----------|----------------|-------------|
| 0.01 - 0.06 | Efecto **pequeño** | Diferencias detectables pero sutiles |
| 0.06 - 0.14 | Efecto **moderado** | Diferencias claramente perceptibles |
| > 0.14 | Efecto **grande** | Diferencias sustanciales y prácticas |

**En nuestro proyecto:**

| Variable | ε² | Clasificación |
|----------|-----|---------------|
| serror_rate | 0.573 | Grande |
| dst_bytes | 0.573 | Grande |
| src_bytes | 0.542 | Grande |
| count | 0.497 | Grande |
| duration | 0.058 | Pequeño-Moderado |

**Conclusión:** Las diferencias no solo son estadísticamente significativas, sino **prácticamente sustanciales**.

---

## **1.6 TEST POST-HOC: TEST DE DUNN**

Kruskal-Wallis solo dice "hay diferencias", pero **no identifica cuáles grupos difieren**.

El **Test de Dunn** realiza comparaciones **por pares** (pairwise) entre todos los grupos.

### **Estadístico de Dunn:**

Para comparar grupos $i$ y $j$:

$$z_{ij} = \frac{\bar{R}_i - \bar{R}_j}{\sqrt{\frac{n(n+1)}{12} \left(\frac{1}{n_i} + \frac{1}{n_j}\right)}}$$

Donde:
- $\bar{R}_i = \frac{R_i}{n_i}$ = Rango promedio del grupo $i$
- $n$ = Total de observaciones

El estadístico $z_{ij}$ sigue una **distribución normal estándar** bajo H₀.

### **Corrección de Bonferroni:**

Al realizar múltiples comparaciones (10 pares en nuestro caso: 5 grupos → $\binom{5}{2} = 10$ comparaciones), aumenta la probabilidad de **error Tipo I** (falso positivo).

La **corrección de Bonferroni** ajusta el nivel de significancia:

$$\alpha_{ajustado} = \frac{\alpha}{m}$$

Donde $m$ = número de comparaciones.

**En nuestro proyecto:**
- $\alpha = 0.05$
- $m = 10$ comparaciones
- $\alpha_{ajustado} = 0.05 / 10 = 0.005$

**Implicación:** Para rechazar H₀ en una comparación específica, necesitamos $p < 0.005$ (más estricto que 0.05).

**Ventaja:** Controla la **tasa de error familiar** (FWER - Family-Wise Error Rate).

**Desventaja:** Es **conservador** – puede perder diferencias reales (aumenta error Tipo II).

---

## **1.7 APLICACIÓN EN NUESTRO PROYECTO**

### **Variables analizadas:**

1. **src_bytes** - Bytes enviados desde origen
2. **dst_bytes** - Bytes recibidos desde destino
3. **duration** - Duración de la conexión
4. **serror_rate** - Tasa de errores SYN
5. **count** - Conexiones en ventana temporal

### **Grupos comparados:**

- Normal (k=1)
- DoS (k=2)
- Probe (k=3)
- R2L (k=4)
- U2R (k=5)

### **Resultados clave:**

**Ejemplo con `serror_rate`:**

1. **H = 14,456.78** (muy alto)
2. **p < 1e-10** (extremadamente significativo)
3. **ε² = 0.573** (efecto grande)

**Interpretación:**
- Las distribuciones de `serror_rate` difieren **dramáticamente** entre categorías
- DoS tiene mediana = 1.0 (100% errores SYN)
- Normal tiene mediana = 0.0 (sin errores SYN)
- Esta variable es un **discriminador potente**

---

# 🔍 PARTE 2: ANÁLISIS DE COMPONENTES PRINCIPALES (PCA)

## **2.1 ¿QUÉ ES PCA Y PARA QUÉ SIRVE?**

**PCA (Principal Component Analysis)** es una técnica de **reducción dimensional** que transforma un conjunto de variables posiblemente correlacionadas en un conjunto más pequeño de variables **ortogonales** (no correlacionadas) llamadas **componentes principales**.

### **Objetivos de PCA:**

1. **Reducir dimensionalidad** sin pérdida crítica de información
2. **Eliminar multicolinealidad** (componentes principales son ortogonales)
3. **Visualizar** datos de alta dimensión en espacios 2D/3D
4. **Identificar patrones** y estructura subyacente en los datos
5. **Comprimir datos** para análisis posteriores más eficientes

### **¿Por qué PCA en nuestro proyecto?**

En el EDA detectamos:
- **14 pares de variables con |r| > 0.8** (multicolinealidad severa)
- **32 variables numéricas** (alta dimensionalidad)
- Variables como `dst_host_srv_count` y `srv_count` miden conceptos redundantes

**PCA resuelve estos problemas** transformando 32 variables correlacionadas en ~8-10 componentes ortogonales.

---

## **2.2 FUNDAMENTOS MATEMÁTICOS DE PCA**

### **Paso 1: Estandarización de Datos**

Antes de aplicar PCA, es **crítico** estandarizar las variables (media 0, desviación estándar 1):

$$z_{ij} = \frac{x_{ij} - \mu_j}{\sigma_j}$$

Donde:
- $x_{ij}$ = Valor original de la variable $j$ en la observación $i$
- $\mu_j$ = Media de la variable $j$
- $\sigma_j$ = Desviación estándar de la variable $j$

**¿Por qué estandarizar?**

Sin estandarización, variables con mayor varianza (ej: `dst_bytes` con rango 0-10,000) dominarían componentes, mientras que variables con menor varianza (ej: `serror_rate` con rango 0-1) serían ignoradas.

**En nuestro proyecto:**
```python
from sklearn.preprocessing import StandardScaler
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)
```

Resultado:
- Cada variable tiene media ≈ 0, std ≈ 1
- Todas las variables tienen la misma "escala" de importancia

---

### **Paso 2: Matriz de Covarianza**

PCA se basa en la **matriz de covarianza** (o correlación si datos están estandarizados):

$$\Sigma = \frac{1}{n-1} X^T X$$

Donde $X$ es la matriz de datos estandarizados ($n \times p$).

**Propiedades de Σ:**
- Dimensión: $p \times p$ (donde $p$ = número de variables)
- Simétrica: $\Sigma = \Sigma^T$
- Semidefinida positiva: todos los eigenvalores son $\geq 0$

**Elemento (i,j) de Σ:**

$$\Sigma_{ij} = \text{Cov}(X_i, X_j) = \frac{1}{n-1} \sum_{k=1}^{n} (x_{ki} - \bar{x}_i)(x_{kj} - \bar{x}_j)$$

Si datos están estandarizados, $\Sigma$ es la **matriz de correlación**.

---

### **Paso 3: Descomposición en Valores y Vectores Propios**

PCA encuentra los **eigenvectors (vectores propios)** y **eigenvalues (valores propios)** de la matriz Σ.

**Definición matemática:**

Un vector $v$ es un eigenvector de Σ con eigenvalue $\lambda$ si:

$$\Sigma v = \lambda v$$

**Interpretación:**
- **Eigenvector $v$:** Dirección en el espacio de variables
- **Eigenvalue $\lambda$:** Varianza explicada en esa dirección

### **Descomposición espectral de Σ:**

$$\Sigma = V \Lambda V^T$$

Donde:
- $V$ = Matriz de eigenvectors (columnas son eigenvectors)
- $\Lambda$ = Matriz diagonal de eigenvalues
- $V^T$ = Transpuesta de $V$ (también $V^{-1}$ porque eigenvectors son ortonormales)

**Propiedades clave:**

1. Hay exactamente $p$ eigenvectors/eigenvalues (donde $p$ = número de variables)
2. Los eigenvectors son **ortogonales** entre sí: $v_i \cdot v_j = 0$ si $i \neq j$
3. Los eigenvectors están **normalizados**: $\|v_i\| = 1$
4. La suma de todos los eigenvalues = varianza total de los datos:
   $$\sum_{i=1}^{p} \lambda_i = \sum_{j=1}^{p} \text{Var}(X_j)$$

---

### **Paso 4: Selección de Componentes Principales**

Los **componentes principales** son los eigenvectors ordenados por su eigenvalue (de mayor a menor).

$$PC_1, PC_2, ..., PC_p$$

Donde:
- $PC_1$ = Eigenvector con mayor eigenvalue $\lambda_1$ → Explica más varianza
- $PC_2$ = Eigenvector con segundo mayor eigenvalue $\lambda_2$ → Explica segunda mayor varianza
- ...y así sucesivamente

**Varianza explicada por PC_i:**

$$\text{Varianza explicada por } PC_i = \frac{\lambda_i}{\sum_{j=1}^{p} \lambda_j}$$

**Varianza explicada acumulada hasta PC_k:**

$$\text{Varianza acumulada (k PCs)} = \frac{\sum_{i=1}^{k} \lambda_i}{\sum_{j=1}^{p} \lambda_j}$$

---

### **Paso 5: Transformación de Datos**

Una vez seleccionados $k$ componentes principales (donde $k < p$), transformamos los datos originales:

$$Z = X \cdot V_k$$

Donde:
- $X$ = Datos originales estandarizados ($n \times p$)
- $V_k$ = Matriz de los primeros $k$ eigenvectors ($p \times k$)
- $Z$ = Datos transformados ($n \times k$)

**Resultado:**
- Cada fila de $Z$ es una observación representada en el nuevo espacio de $k$ dimensiones
- Las columnas de $Z$ son los **scores** (valores) de cada componente principal

---

## **2.3 INTERPRETACIÓN: LOADINGS**

Los **loadings** son los elementos de los eigenvectors. Representan la **contribución** de cada variable original a un componente principal.

**Para el componente principal $i$:**

$$PC_i = l_{i1} \cdot X_1 + l_{i2} \cdot X_2 + ... + l_{ip} \cdot X_p$$

Donde $l_{ij}$ es el loading de la variable $j$ en $PC_i$.

### **Interpretación de loadings:**

- **|loading| grande (cerca de 1):** La variable tiene alta contribución al componente
- **|loading| pequeño (cerca de 0):** La variable tiene baja contribución
- **Signo (+/-):** Dirección de la contribución

### **Ejemplo en nuestro proyecto:**

**PC1** está dominado por:
- `same_srv_rate` (loading = 0.35)
- `dst_host_same_srv_rate` (loading = 0.34)
- `dst_host_serror_rate` (loading = 0.32)

**Interpretación:** PC1 captura patrones relacionados con **tasas de servicio y errores SYN en el mismo host**.

---

## **2.4 CRITERIOS PARA SELECCIONAR NÚMERO DE COMPONENTES**

### **Criterio 1: Varianza Explicada Acumulada**

Seleccionar $k$ tal que:

$$\frac{\sum_{i=1}^{k} \lambda_i}{\sum_{j=1}^{p} \lambda_j} \geq \text{umbral}$$

**Umbrales comunes:**
- 70-80%: Análisis exploratorio
- 85-90%: Modelado predictivo
- 95%: Preservación casi total

**En nuestro proyecto:**
- 5 PCs → 60.75% varianza
- 8 PCs → 71.85% varianza
- 13 PCs → 87.16% varianza

### **Criterio 2: Scree Plot (Criterio del Codo)**

Graficar $\lambda_i$ vs $i$ y buscar el "codo" – el punto donde la pendiente cambia abruptamente.

**Interpretación del codo:**
- Antes del codo: Componentes aportan mucha información
- Después del codo: Componentes aportan poca información adicional

**En nuestro proyecto:**
- El codo está en PC5-PC6
- Después, cada componente aporta <5% de varianza

### **Criterio 3: Regla de Kaiser**

Seleccionar solo componentes con eigenvalue $\lambda_i > 1$.

**Justificación:** Si $\lambda_i < 1$, el componente explica menos varianza que una variable original estandarizada.

**Limitación:** Puede ser demasiado conservador o permisivo dependiendo del dataset.

### **Criterio 4: Validación Cruzada**

Probar diferentes valores de $k$ y evaluar desempeño en tarea posterior (ej: clasificación).

**En nuestro proyecto:**
- Probamos 5, 8 y 13 PCs
- 8 PCs tuvo mejor balance: AUC 0.9496 vs 0.9541 con 13 PCs (diferencia <0.5%)
- **Decisión:** 8 PCs por **parsimonia** (principio de Occam)

---

## **2.5 LIMITACIONES DE PCA**

### **1. Transformación Lineal**

PCA solo captura relaciones **lineales** entre variables.

Si hay relaciones **no lineales** complejas (ej: cuadráticas, exponenciales), PCA puede no detectarlas.

**Alternativas:** Kernel PCA, t-SNE, UMAP (para relaciones no lineales)

### **2. Interpretabilidad Reducida**

Un componente principal es una combinación lineal de múltiples variables.

**Problema:** "PC1 explica 15% de varianza" es menos interpretable que "duration difiere significativamente entre grupos".

**Solución parcial:** Analizar loadings para ver qué variables dominan cada componente.

### **3. Sensibilidad a Outliers**

PCA busca direcciones de **máxima varianza**, que pueden estar influenciadas por outliers extremos.

**En nuestro proyecto:** Mantuvimos outliers porque son señal de ataque, pero esto puede sesgar componentes.

**Alternativa robusta:** Robust PCA (usa medianas en lugar de medias)

### **4. Supuesto de Ortogonalidad**

Los componentes principales son **ortogonales** (no correlacionados), pero a veces relaciones oblicuas (correlacionadas) son más naturales.

**Alternativa:** Factor Analysis, ICA (Independent Component Analysis)

---

## **2.6 EVALUACIÓN DE SEPARABILIDAD: SILHOUETTE SCORE**

Después de PCA, evaluamos si los componentes preservan **separabilidad entre categorías**.

### **Silhouette Score**

Mide qué tan bien está asignada cada observación a su cluster comparado con otros clusters.

**Para una observación $i$:**

$$s(i) = \frac{b(i) - a(i)}{\max\{a(i), b(i)\}}$$

Donde:
- $a(i)$ = Distancia promedio entre $i$ y todas las demás observaciones **del mismo cluster**
- $b(i)$ = Distancia promedio entre $i$ y todas las observaciones **del cluster más cercano diferente**

**Interpretación:**

| Silhouette | Interpretación |
|------------|----------------|
| $s \approx 1$ | Bien asignada (muy cerca de su cluster, lejos de otros) |
| $s \approx 0$ | En el límite entre clusters |
| $s \approx -1$ | Mal asignada (más cerca de otro cluster) |

**Score global:**

$$\text{Silhouette Score} = \frac{1}{n} \sum_{i=1}^{n} s(i)$$

**En nuestro proyecto:**
- **Silhouette 2D (PC1+PC2): 0.1286** → Bajo (clusters solapados en 2D)
- **Silhouette 3D (PC1+PC2+PC3): 0.1194** → Similar

**Interpretación crítica:**

El Silhouette bajo en 2D/3D **NO significa que PCA falló**. Al contrario, confirma que:
- La separabilidad NO está en proyecciones visuales simples
- La información discriminante está distribuida en las **8 dimensiones combinadas**
- Por eso necesitamos 8 PCs en lugar de solo 2-3

**Validación:** AUC 0.9496 en 8D confirma que la separabilidad existe, pero en espacio no visualizable.

---

# 🎯 PARTE 3: REGRESIÓN LOGÍSTICA

## **3.1 ¿QUÉ ES REGRESIÓN LOGÍSTICA?**

La **regresión logística** es un modelo de **clasificación** que predice la probabilidad de que una observación pertenezca a una clase (categoría).

**A pesar del nombre "regresión", es un modelo de CLASIFICACIÓN.**

### **Diferencias con Regresión Lineal:**

| Aspecto | Regresión Lineal | Regresión Logística |
|---------|------------------|---------------------|
| **Variable respuesta** | Cuantitativa continua | Categórica (binaria o multinomial) |
| **Función** | Lineal: $y = \beta_0 + \beta_1 x_1 + ...$ | Logística: $P(Y=1) = \frac{1}{1+e^{-(\beta_0 + \beta_1 x_1 + ...)}}$ |
| **Rango de salida** | $(-\infty, +\infty)$ | $[0, 1]$ (probabilidades) |
| **Interpretación** | Cambio en Y por unidad de X | Cambio en log-odds de Y |

---

## **3.2 MODELO MATEMÁTICO**

### **Regresión Logística Binaria**

Para clasificar entre dos clases (Y = 0 o Y = 1):

**Función logística (sigmoide):**

$$P(Y=1 \mid X) = \frac{1}{1 + e^{-z}} = \frac{e^z}{1 + e^z}$$

Donde:

$$z = \beta_0 + \beta_1 x_1 + \beta_2 x_2 + ... + \beta_p x_p$$

**Componentes:**
- $\beta_0$ = Intercepto (sesgo)
- $\beta_1, ..., \beta_p$ = Coeficientes de las variables predictoras
- $x_1, ..., x_p$ = Variables predictoras (en nuestro caso, componentes principales)

### **Función Sigmoide: Propiedades**

$$\sigma(z) = \frac{1}{1 + e^{-z}}$$

**Propiedades clave:**

1. **Rango:** $\sigma(z) \in (0, 1)$ → Siempre devuelve probabilidad válida
2. **Monotonía:** $\sigma'(z) > 0$ → Siempre creciente
3. **Simetría:** $\sigma(-z) = 1 - \sigma(z)$
4. **Límites:**
   - $\lim_{z \to +\infty} \sigma(z) = 1$
   - $\lim_{z \to -\infty} \sigma(z) = 0$
   - $\sigma(0) = 0.5$ (punto de decisión natural)

**Gráfica:**
```
P(Y=1)
  1.0 |                    ___________
      |                 __/
  0.5 |             __/
      |          __/
  0.0 |_________/
      +-----|-----|-----|-----|------> z
           -4    -2     0     2     4
```

---

## **3.3 INTERPRETACIÓN: LOG-ODDS Y ODDS RATIO**

### **Odds (Momios)**

$$\text{Odds} = \frac{P(Y=1)}{P(Y=0)} = \frac{P(Y=1)}{1 - P(Y=1)}$$

**Interpretación:**
- Odds = 1: Igualmente probable Y=1 que Y=0
- Odds = 2: Dos veces más probable Y=1 que Y=0
- Odds = 0.5: Dos veces más probable Y=0 que Y=1

### **Log-Odds (Logit)**

El modelo de regresión logística predice el **log-odds**:

$$\text{logit}(P) = \log\left(\frac{P}{1-P}\right) = \beta_0 + \beta_1 x_1 + ... + \beta_p x_p$$

**Esto es una relación LINEAL entre predictores y log-odds.**

### **Interpretación de Coeficientes**

Para un coeficiente $\beta_j$:

**Aumento de 1 unidad en $x_j$ implica:**

$$\Delta \text{log-odds} = \beta_j$$

**En términos de Odds Ratio:**

$$\text{Odds Ratio} = e^{\beta_j}$$

**Ejemplo:**
- Si $\beta_1 = 0.5$: $OR = e^{0.5} = 1.65$
  - Por cada unidad que aumenta $x_1$, los odds de Y=1 se multiplican por 1.65
  
- Si $\beta_2 = -0.3$: $OR = e^{-0.3} = 0.74$
  - Por cada unidad que aumenta $x_2$, los odds de Y=1 se multiplican por 0.74 (disminuyen)

---

## **3.4 ESTIMACIÓN DE PARÁMETROS: MÁXIMA VEROSIMILITUD**

Los coeficientes $\beta$ se estiman mediante **Máxima Verosimilitud** (Maximum Likelihood Estimation, MLE).

### **Función de Verosimilitud**

Dadas $n$ observaciones $(x_i, y_i)$:

$$L(\beta) = \prod_{i=1}^{n} P(Y=y_i \mid x_i; \beta)$$

Donde:

$$P(Y=y_i \mid x_i; \beta) = \begin{cases}
p_i & \text{si } y_i = 1 \\
1 - p_i & \text{si } y_i = 0
\end{cases}$$

Con $p_i = P(Y=1 \mid x_i; \beta) = \sigma(\beta^T x_i)$.

**Forma compacta:**

$$L(\beta) = \prod_{i=1}^{n} p_i^{y_i} (1-p_i)^{1-y_i}$$

### **Log-Verosimilitud**

Por conveniencia numérica, maximizamos el **log** de la verosimilitud:

$$\ell(\beta) = \log L(\beta) = \sum_{i=1}^{n} \left[ y_i \log(p_i) + (1-y_i) \log(1-p_i) \right]$$

### **Optimización**

No hay solución cerrada → Se usa **descenso de gradiente** o métodos iterativos (Newton-Raphson, L-BFGS).

**Gradiente:**

$$\frac{\partial \ell}{\partial \beta_j} = \sum_{i=1}^{n} (y_i - p_i) x_{ij}$$

**Actualización iterativa:**

$$\beta^{(t+1)} = \beta^{(t)} + \alpha \nabla \ell(\beta^{(t)})$$

Donde $\alpha$ es la tasa de aprendizaje (learning rate).

---

## **3.5 REGULARIZACIÓN: L2 (RIDGE)**

Para evitar **overfitting**, añadimos un término de **penalización** a la función objetivo.

### **Ridge Regression (L2):**

$$\ell_{regularizada}(\beta) = \ell(\beta) - \lambda \sum_{j=1}^{p} \beta_j^2$$

Donde:
- $\lambda$ = Parámetro de regularización (hiperparámetro)
- Mayor $\lambda$ → Mayor penalización → Coeficientes más pequeños

**En sklearn:**
```python
LogisticRegression(penalty='l2', C=1.0)
```

Donde $C = \frac{1}{\lambda}$ (inverso de la penalización).

**Efecto de C:**
- **C grande (ej: 10):** Poca regularización → Modelo complejo
- **C pequeño (ej: 0.1):** Mucha regularización → Modelo simple

**En nuestro proyecto:** Usamos $C = 1.0$ (regularización estándar).

---

## **3.6 ESTRATEGIA ONE-VS-REST (OvR)**

Para clasificación **multiclase** (k > 2 clases), existen dos estrategias principales:

### **1. One-vs-Rest (OvR) o One-vs-All (OvA)**

Entrenar **k modelos binarios** independientes:
- Modelo 1: DoS vs. {Normal, Probe, R2L, U2R}
- Modelo 2: Probe vs. {Normal, DoS, R2L, U2R}
- Modelo 3: R2L vs. {Normal, DoS, Probe, U2R}
- Modelo 4: U2R vs. {Normal, DoS, Probe, R2L}

**Predicción:**
- Calcular probabilidad de cada modelo
- Asignar la clase con **mayor probabilidad**

### **2. One-vs-One (OvO)**

Entrenar $\binom{k}{2}$ modelos binarios entre cada par de clases.

Para k=5: $\binom{5}{2} = 10$ modelos.

**Predicción:** Votación mayoritaria.

### **¿Por qué elegimos OvR en nuestro proyecto?**

1. **Menos modelos:** 4 modelos (OvR) vs 10 modelos (OvO)
2. **Interpretabilidad:** Cada modelo responde "¿Es este tipo de ataque?"
3. **Manejo de desbalance:** Podemos ajustar `class_weight` independientemente
4. **Estándar sklearn:** OvR es default en `LogisticRegression` con `multi_class='ovr'`

---

## **3.7 MANEJO DE DESBALANCE: CLASS_WEIGHT='BALANCED'**

Nuestro dataset tiene **desbalance extremo:**
- DoS: 36.7%
- Probe: 9.1%
- R2L: 0.83%
- U2R: 0.04%

Sin ajuste, el modelo aprende un **sesgo hacia la clase mayoritaria**.

### **class_weight='balanced'**

Ajusta los pesos de las clases inversamente proporcionales a su frecuencia:

$$w_c = \frac{n}{k \cdot n_c}$$

Donde:
- $n$ = Total de observaciones
- $k$ = Número de clases
- $n_c$ = Número de observaciones de la clase $c$

**Efecto en la función de pérdida:**

$$\ell_{weighted}(\beta) = \sum_{i=1}^{n} w_{y_i} \left[ y_i \log(p_i) + (1-y_i) \log(1-p_i) \right]$$

**Interpretación:**
- Clases minoritarias (U2R) reciben peso **mucho mayor**
- El modelo penaliza más los errores en clases minoritarias
- Mejora recall de clases minoritarias (a costa de precision)

---

## **3.8 MÉTRICAS DE EVALUACIÓN**

### **Matriz de Confusión (2x2 para binario)**

|                | Predicho: 0 | Predicho: 1 |
|----------------|-------------|-------------|
| **Real: 0**    | TN          | FP          |
| **Real: 1**    | FN          | TP          |

Donde:
- **TP (True Positive):** Predijo 1 y era 1 ✅
- **TN (True Negative):** Predijo 0 y era 0 ✅
- **FP (False Positive):** Predijo 1 pero era 0 ❌ (Error Tipo I)
- **FN (False Negative):** Predijo 0 pero era 1 ❌ (Error Tipo II)

### **Métricas Derivadas**

#### **1. Accuracy (Exactitud)**

$$\text{Accuracy} = \frac{TP + TN}{TP + TN + FP + FN}$$

**Interpretación:** Proporción de predicciones correctas.

**Limitación:** Engañosa con datos desbalanceados.

**Ejemplo:** Si 99% es clase negativa, predecir siempre 0 da 99% accuracy.

---

#### **2. Precision (Precisión)**

$$\text{Precision} = \frac{TP}{TP + FP}$$

**Interpretación:** De todas las predicciones positivas, ¿cuántas fueron correctas?

**Pregunta:** "Cuando digo que es un ataque, ¿con qué frecuencia acierto?"

**Alta precision → Pocas falsas alarmas**

---

#### **3. Recall (Sensibilidad / TPR)**

$$\text{Recall} = \frac{TP}{TP + FN}$$

**Interpretación:** De todos los positivos reales, ¿cuántos detectamos?

**Pregunta:** "De todos los ataques reales, ¿cuántos detecto?"

**Alto recall → Pocas amenazas se escapan**

---

#### **4. F1-Score**

$$F_1 = 2 \cdot \frac{\text{Precision} \cdot \text{Recall}}{\text{Precision} + \text{Recall}}$$

**Interpretación:** Media armónica entre precision y recall.

**Útil cuando hay desbalance y queremos balance entre ambas métricas.**

**Valores:**
- $F_1 = 1$: Perfecto (Precision=1 y Recall=1)
- $F_1 = 0$: Pésimo

---

#### **5. ROC Curve y AUC-ROC**

**ROC Curve (Receiver Operating Characteristic):**

Gráfica de **TPR (Recall)** vs **FPR (False Positive Rate)** variando el threshold de decisión.

$$\text{FPR} = \frac{FP}{FP + TN}$$

**¿Por qué variar el threshold?**

Por defecto, clasificamos como 1 si $P(Y=1) \geq 0.5$.

Pero podemos usar cualquier threshold $\tau \in [0, 1]$:
- $\tau = 0.1$: Predice 1 casi siempre (alto recall, baja precision)
- $\tau = 0.9$: Predice 1 raramente (alta precision, bajo recall)

**La curva ROC muestra este trade-off.**

---

**AUC-ROC (Area Under the ROC Curve):**

$$\text{AUC} = \int_0^1 \text{TPR}(\text{FPR}) \, d(\text{FPR})$$

**Interpretación:**

| AUC | Interpretación |
|-----|----------------|
| 0.5 | Clasificador aleatorio (línea diagonal) |
| 0.7-0.8 | Aceptable |
| 0.8-0.9 | Excelente |
| 0.9-1.0 | Excepcional |

**AUC es robusto al desbalance** porque considera todos los thresholds posibles.

**En nuestro proyecto:**
- DoS: AUC = 0.9902 (excepcional)
- Probe: AUC = 0.9868 (excepcional)
- R2L: AUC = 0.9593 (excelente)
- U2R: AUC = 0.8619 (bueno, considerando n=11)

---

### **3.9 PARADOJA: AUC ALTO, F1 BAJO**

**Observación en R2L:**
- AUC = 0.9593 (excelente)
- F1 = 0.1515 (bajo)

**¿Por qué?**

**AUC mide discriminación:** El modelo rankea correctamente (ataques R2L tienen scores más altos que no-R2L).

**F1 mide clasificación binaria con threshold fijo (0.5):**

Con R2L siendo 0.83%, el threshold 0.5 es inapropiado:
- El modelo asigna probabilidades como 0.3 a instancias R2L (menos que 0.5)
- Se clasifican como 0 (no-R2L) → FN aumenta → Recall baja → F1 baja

**Solución:** Optimizar threshold según costo FP/FN.

Si los costos son:
- FN (no detectar ataque) = $10,000
- FP (falsa alarma) = $10

Threshold óptimo sería mucho menor (ej: 0.1), priorizando recall sobre precision.

---

## **3.10 ANÁLISIS DE COEFICIENTES**

Los coeficientes de regresión logística revelan qué componentes principales son más importantes para cada tipo de ataque.

**Para el modelo DoS vs Resto:**

Supongamos:
- $\beta_1 = 2.5$ (PC1)
- $\beta_2 = -0.8$ (PC2)
- $\beta_3 = 1.2$ (PC3)

**Interpretación:**

- **PC1:** Coeficiente positivo alto (2.5) → Aumentos en PC1 **aumentan fuertemente** probabilidad de DoS
  - Si recordamos que PC1 captura patrones de `serror_rate` alto, esto tiene sentido (DoS tiene serror_rate=1.0)

- **PC2:** Coeficiente negativo (-0.8) → Aumentos en PC2 **disminuyen** probabilidad de DoS

- **PC3:** Coeficiente positivo moderado (1.2) → Contribuye positivamente

**Conexión con P1 y P2:**

Variables con **alto ε² en Kruskal-Wallis** (P1) → **Alto loading en PCs** (P2) → **Alto coeficiente en regresión** (P3).

Esta coherencia valida la metodología completa.

---

# 📊 RESUMEN INTEGRADO: FLUJO METODOLÓGICO

## **Pregunta 1: Kruskal-Wallis**

**Input:** 5 variables × 5 categorías
**Proceso:** Test no paramétrico basado en rangos
**Output:** p < 1e-10, ε² > 0.49
**Conclusión:** Variables tienen poder discriminante robusto

## **Pregunta 2: PCA**

**Input:** 32 variables correlacionadas
**Proceso:** Descomposición espectral de matriz de covarianza
**Output:** 8 componentes capturan 71.85% varianza
**Conclusión:** Reducción dimensional exitosa sin pérdida crítica

## **Pregunta 3: Regresión Logística**

**Input:** 8 componentes principales
**Proceso:** 4 modelos OvR con MLE y regularización L2
**Output:** AUC promedio 0.9496
**Conclusión:** Clasificación viable, DoS/Probe listos para producción

---

## **Coherencia Metodológica:**

```
Variables discriminantes (P1)
         ↓
   Alto ε² en Kruskal-Wallis
         ↓
   Loadings altos en PCs (P2)
         ↓
   PCs capturan información discriminante
         ↓
   Coeficientes altos en regresión (P3)
         ↓
   Clasificación exitosa (AUC 0.95)
```

**Esta cadena de evidencia valida el enfoque completo.**

---

# 🎓 CONCEPTOS CLAVE PARA DOMINAR

## **Kruskal-Wallis:**
1. Es alternativa no paramétrica a ANOVA
2. Usa rangos, no valores originales
3. Compara distribuciones, no solo medias
4. H sigue Chi-cuadrado con k-1 grados de libertad
5. ε² mide tamaño del efecto (>0.14 = grande)
6. Test de Dunn + Bonferroni para comparaciones por pares

## **PCA:**
1. Reduce dimensionalidad preservando varianza
2. Basado en eigenvectors de matriz de covarianza
3. Componentes principales son ortogonales (no correlacionados)
4. Estandarización es crítica antes de PCA
5. Scree plot ayuda a identificar el "codo"
6. Loadings revelan contribución de variables originales
7. Silhouette bajo en 2D no invalida PCA si AUC es alto en kD

## **Regresión Logística:**
1. Modelo de clasificación, no regresión
2. Predice probabilidades vía función sigmoide
3. Coeficientes interpretan como log-odds
4. MLE + descenso de gradiente para estimación
5. Regularización L2 previene overfitting
6. OvR entrena k modelos binarios independientes
7. class_weight='balanced' maneja desbalance
8. AUC mide discriminación robusta al desbalance
9. AUC alto + F1 bajo indica threshold inapropiado

---

Esto cubre la teoría completa de la Sección 4. Ahora procedo a crear las preguntas...

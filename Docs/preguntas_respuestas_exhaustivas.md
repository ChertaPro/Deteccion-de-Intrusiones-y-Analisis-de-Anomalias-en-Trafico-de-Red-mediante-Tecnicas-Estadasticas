# 🎯 BANCO DE PREGUNTAS Y RESPUESTAS - SECCIÓN 4

---

# PARTE 1: TEST DE KRUSKAL-WALLIS

## **NIVEL 1: CONCEPTOS BÁSICOS**

### **P1.1: ¿Qué es el Test de Kruskal-Wallis y cuándo se usa?**

**R:** El Test de Kruskal-Wallis es una prueba **no paramétrica** que compara las distribuciones de una variable cuantitativa entre **3 o más grupos independientes**. Es la alternativa no paramétrica al ANOVA de un factor.

**Se usa cuando:**
- Tienes k ≥ 3 grupos a comparar
- La variable respuesta es cuantitativa u ordinal
- Los datos **NO cumplen** supuestos de ANOVA (normalidad, homogeneidad de varianzas)
- Hay presencia de outliers extremos

**En nuestro proyecto:** Usamos Kruskal-Wallis porque el 68.8% de variables tienen asimetría extrema (|Skew| ≥ 1), violando normalidad.

---

### **P1.2: ¿Por qué Kruskal-Wallis es "robusto a outliers"?**

**R:** Porque trabaja con **rangos** en lugar de valores originales.

**Ejemplo:**
```
Valores originales: 5, 10, 15, 1000
Rangos:             1,  2,  3,    4
```

El valor extremo 1000 solo recibe rango 4, perdiendo su influencia desproporcionada. En cambio, ANOVA usa la media aritmética, que es muy sensible a outliers.

**En nuestro proyecto:** Teníamos 18.6% de outliers en `dst_bytes`. Kruskal-Wallis los maneja sin problemas.

---

### **P1.3: ¿Cuál es la hipótesis nula (H₀) del test de Kruskal-Wallis?**

**R:** 
$$H_0: F_1(x) = F_2(x) = ... = F_k(x) \quad \forall x$$

**En palabras:** Las distribuciones de la variable son **idénticas** en todos los k grupos.

**Nota crítica:** H₀ NO dice "las medianas son iguales". Dice que **toda la distribución** (forma, dispersión, tendencia central) es la misma.

**Hipótesis alternativa (H₁):** Al menos una distribución difiere de las demás.

---

### **P1.4: ¿Qué significa tener p < 0.05 en Kruskal-Wallis?**

**R:** Significa que la probabilidad de observar diferencias tan extremas (o más) como las observadas, **asumiendo que H₀ es verdadera**, es menor al 5%.

**Decisión:** Rechazar H₀ → Hay evidencia suficiente para concluir que al menos una distribución difiere.

**En nuestro proyecto:** Obtuvimos p < 1×10⁻¹⁰ en todas las variables → Probabilidad prácticamente cero de que las diferencias sean casualidad.

---

### **P1.5: ¿Qué distribución sigue el estadístico H de Kruskal-Wallis?**

**R:** Bajo H₀, el estadístico H sigue aproximadamente una **distribución Chi-cuadrado (χ²)** con **k-1 grados de libertad**:

$$H \sim \chi^2_{k-1}$$

**Condición de validez:** Cada grupo debe tener al menos 5 observaciones.

**En nuestro proyecto:**
- k = 5 grupos (Normal, DoS, Probe, R2L, U2R)
- Grados de libertad = 5 - 1 = 4
- H ~ χ²₄

---

## **NIVEL 2: INTERPRETACIÓN DE RESULTADOS**

### **P1.6: En tu proyecto, ¿por qué p < 1e-10 no es suficiente por sí solo?**

**R:** Porque el p-valor solo indica **significancia estadística**, no la **magnitud práctica** de las diferencias.

**Ejemplo extremo:**
Con n = 1,000,000 observaciones, una diferencia de 0.01% puede ser estadísticamente significativa (p < 0.05) pero **irrelevante en la práctica**.

**Por eso usamos Epsilon-squared (ε²):** Mide el **tamaño del efecto** independientemente del tamaño muestral.

**En nuestro proyecto:**
- `serror_rate`: p < 1e-10 (significativo) **Y** ε² = 0.573 (efecto grande) ✅
- Ambos criterios confirman diferencias robustas y sustanciales

---

### **P1.7: ¿Qué es Epsilon-squared (ε²) y cómo se interpreta?**

**R:** ε² es una medida de **tamaño del efecto** para Kruskal-Wallis, análoga a η² (eta-squared) en ANOVA.

**Fórmula:**
$$\varepsilon^2 = \frac{H - k + 1}{n - k}$$

**Interpretación según Cohen (1988):**

| ε² | Tamaño del efecto | Interpretación |
|----|-------------------|----------------|
| 0.01 - 0.06 | Pequeño | Diferencias sutiles pero detectables |
| 0.06 - 0.14 | Moderado | Diferencias claras y perceptibles |
| > 0.14 | Grande | Diferencias sustanciales y prácticas |

**En nuestro proyecto:**
- `serror_rate`: ε² = 0.573 → **Efecto GRANDE**
- `dst_bytes`: ε² = 0.573 → **Efecto GRANDE**
- `duration`: ε² = 0.058 → Efecto pequeño-moderado

---

### **P1.8: ¿Qué significa tener H = 14,456.78 en `serror_rate`?**

**R:** 

**1. Magnitud relativa:** H es enorme (recordar que χ²₄ tiene valor esperado = 4 bajo H₀)

**2. Comparación con punto crítico:**
- Punto crítico χ²₄,₀.₀₅ ≈ 9.49
- H = 14,456.78 >>> 9.49 → Rechazamos H₀ rotundamente

**3. Interpretación práctica:** Las sumas de rangos difieren **dramáticamente** entre grupos.

**Ejemplo ilustrativo:**
- Si DoS siempre tiene rangos altos (ej: 20000-25000)
- Y Normal siempre tiene rangos bajos (ej: 1-5000)
- La diferencia en R̄ᵢ (rango promedio) será enorme → H será enorme

---

### **P1.9: ¿Kruskal-Wallis te dice CUÁLES grupos son diferentes?**

**R:** **NO.** Kruskal-Wallis es un test **global** (omnibus test).

**Solo responde:** "¿Hay al menos una diferencia entre los k grupos?"

**NO responde:** "¿Qué grupos específicos difieren?"

**Para identificar cuáles grupos difieren:** Necesitas un **test post-hoc** como:
- Test de Dunn (más común con Kruskal-Wallis)
- Test de Wilcoxon por pares con corrección de Bonferroni

**En nuestro proyecto:** Aplicamos Test de Dunn + corrección de Bonferroni para identificar pares significativos.

---

## **NIVEL 3: TEST POST-HOC Y CORRECCIONES**

### **P1.10: ¿Qué es el Test de Dunn y por qué lo usas después de Kruskal-Wallis?**

**R:** El **Test de Dunn** realiza comparaciones **por pares** (pairwise) entre todos los grupos tras un Kruskal-Wallis significativo.

**Estadístico de Dunn:**
$$z_{ij} = \frac{\bar{R}_i - \bar{R}_j}{\sqrt{\frac{n(n+1)}{12} \left(\frac{1}{n_i} + \frac{1}{n_j}\right)}}$$

Donde $\bar{R}_i$ es el rango promedio del grupo i.

**Distribución:** z ~ N(0,1) bajo H₀ (los grupos i y j tienen misma distribución)

**En nuestro proyecto:**
- 5 grupos → $\binom{5}{2} = 10$ comparaciones
- Ejemplos: Normal vs DoS, DoS vs Probe, R2L vs U2R, etc.

---

### **P1.11: ¿Qué es el problema de comparaciones múltiples?**

**R:** Al realizar múltiples tests simultáneamente, la probabilidad de cometer al menos un **error Tipo I** (falso positivo) aumenta.

**Ejemplo:**
- 1 test con α = 0.05 → P(Error Tipo I) = 0.05
- 10 tests con α = 0.05 → P(al menos 1 Error Tipo I) = 1 - (0.95)¹⁰ ≈ 0.40 (40%!)

**Tasa de Error Familiar (FWER):**
$$\text{FWER} = 1 - (1 - \alpha)^m$$

Donde m = número de comparaciones.

**Solución:** Aplicar corrección (Bonferroni, Holm, etc.)

---

### **P1.12: ¿Cómo funciona la corrección de Bonferroni?**

**R:** La corrección de Bonferroni ajusta el nivel de significancia para controlar el FWER:

$$\alpha_{ajustado} = \frac{\alpha}{m}$$

Donde m = número de comparaciones.

**En nuestro proyecto:**
- α original = 0.05
- m = 10 comparaciones
- α ajustado = 0.05 / 10 = **0.005**

**Regla de decisión:** Rechazar H₀ en la comparación i-j solo si p < 0.005.

**Ventaja:** Controla rigurosamente el FWER.

**Desventaja:** Es **conservador** → Aumenta probabilidad de error Tipo II (no detectar diferencias reales).

---

### **P1.13: ¿Por qué dices que Bonferroni es "conservador"?**

**R:** Porque asume el **peor caso** (todas las comparaciones son independientes), pero en la práctica, comparaciones que involucran el mismo grupo están correlacionadas.

**Ejemplo:**
- "Normal vs DoS" y "Normal vs Probe" comparten grupo "Normal"
- No son completamente independientes

**Consecuencia:** Bonferroni penaliza **más de lo necesario**, aumentando el error Tipo II (β).

**Alternativas menos conservadoras:**
- Holm-Bonferroni (secuencial, más poderosa)
- Benjamini-Hochberg (controla FDR en lugar de FWER)

**En nuestro proyecto:** Usamos Bonferroni porque aún con su conservadurismo, obtuvimos mayoría de pares significativos → Evidencia de diferencias muy robustas.

---

### **P1.14: Si tienes p < 0.005 en "DoS vs Normal" para `serror_rate`, ¿qué concluyes?**

**R:** Conclusión: Rechazamos H₀ de que DoS y Normal tienen distribuciones idénticas de `serror_rate`.

**Interpretación práctica:**
- DoS tiene mediana `serror_rate` = 1.0 (100% errores SYN)
- Normal tiene mediana `serror_rate` = 0.0 (sin errores SYN)
- Esta diferencia es **estadísticamente significativa** incluso con corrección Bonferroni estricta

**Implicación:** `serror_rate` es un **discriminador excelente** entre DoS y Normal.

---

## **NIVEL 4: COMPARACIÓN CON ANOVA**

### **P1.15: ¿Cuándo usarías ANOVA en lugar de Kruskal-Wallis?**

**R:** Usa ANOVA cuando:

1. **Normalidad:** Los datos de cada grupo siguen distribución normal (verificar con Shapiro-Wilk, Q-Q plot)
2. **Homogeneidad de varianzas:** Las varianzas de los grupos son aproximadamente iguales (verificar con Levene, Bartlett)
3. **Sin outliers extremos:** No hay valores atípicos que distorsionen la media

**Ventaja de ANOVA sobre Kruskal-Wallis:**
- Mayor **potencia estadística** (poder de detectar diferencias reales) cuando los supuestos se cumplen

**En nuestro proyecto:**
- 68.8% variables con asimetría extrema → Violación severa de normalidad
- 18.6% outliers en dst_bytes → Sensibilidad problemática
- **Decisión correcta:** Kruskal-Wallis

---

### **P1.16: ¿ANOVA y Kruskal-Wallis prueban la misma hipótesis?**

**R:** **NO exactamente.**

**ANOVA:**
- H₀: μ₁ = μ₂ = ... = μₖ (las **medias** son iguales)
- Supone que las distribuciones tienen **misma forma y varianza**, solo difieren en media

**Kruskal-Wallis:**
- H₀: F₁(x) = F₂(x) = ... = Fₖ(x) (las **distribuciones completas** son iguales)
- NO asume nada sobre forma o varianza

**Implicación:**

Si dos grupos tienen:
- Misma mediana (ej: 10)
- Pero diferentes dispersiones (uno con σ=1, otro con σ=10)

**ANOVA:** Podría no rechazar H₀ (medianas iguales)
**Kruskal-Wallis:** Probablemente rechazaría H₀ (distribuciones difieren en dispersión)

---

### **P1.17: Tu proyecto dice "comparar distribuciones", no "comparar medianas". ¿Por qué es importante esta distinción?**

**R:** Porque Kruskal-Wallis detecta **cualquier diferencia en la distribución**, no solo en la tendencia central.

**Ejemplo con `duration`:**

| Grupo | Mediana | Distribución |
|-------|---------|--------------|
| Normal | 0 | 75% son 0, 25% varían 1-100 |
| DoS | 0 | 95% son 0, 5% son outliers masivos |

**Mismo mediana (0), pero distribuciones MUY diferentes.**

**Kruskal-Wallis detectaría esta diferencia porque:**
- Los rangos superiores (outliers de DoS) afectan la suma de rangos
- Aunque la mediana sea idéntica

**En nuestro proyecto:** Esto explica por qué `duration` tiene ε² = 0.058 (moderado) a pesar de medianas similares en algunos grupos.

---

## **NIVEL 5: APLICACIÓN ESPECÍFICA AL PROYECTO**

### **P1.18: ¿Por qué elegiste `src_bytes`, `dst_bytes`, `duration`, `serror_rate` y `count`?**

**R:** **Criterio 1: Relevancia conceptual para IDS**
- `src_bytes` / `dst_bytes`: Volumen de tráfico (detecta exfiltración, DoS floods)
- `duration`: Duración de conexión (detecta sesiones persistentes sospechosas)
- `serror_rate`: Tasa errores SYN (firma característica de SYN floods)
- `count`: Ráfagas de conexiones (detecta escaneos masivos, DoS)

**Criterio 2: Alta variabilidad en EDA**
- Estas 5 tuvieron CV > 1 o asimetría extrema → Señal de comportamientos diferenciados

**Criterio 3: Balance entre profundidad y extensión**
- 5 variables son suficientes para responder la pregunta sin abrumar con 32 tests

---

### **P1.19: Explica el resultado para `serror_rate`: H = 14,456, p < 1e-10, ε² = 0.573**

**R:** 

**Estadístico H = 14,456:**
- Valor extremadamente alto comparado con χ²₄ bajo H₀ (valor esperado = 4)
- Indica diferencias masivas en las sumas de rangos entre grupos

**p-valor < 1e-10:**
- Probabilidad prácticamente cero de observar H tan extremo por casualidad
- Rechazamos H₀ rotundamente → Hay diferencias significativas

**ε² = 0.573:**
- Tamaño del efecto **GRANDE** (>>0.14)
- 57.3% de la variabilidad en rangos se explica por la categoría de ataque
- Diferencias no solo significativas, sino **prácticamente sustanciales**

**Interpretación contextual:**
- DoS: mediana serror_rate = 1.0 (100% errores SYN → SYN flood)
- Normal: mediana serror_rate = 0.0 (conexiones exitosas)
- Esta diferencia es **tan extrema** que justifica los valores astronómicos de H y ε²

---

### **P1.20: ¿Por qué `duration` tiene ε² = 0.058 (pequeño) pero aún p < 1e-10?**

**R:** 

**p-valor pequeño:** Con n = 25,192 observaciones, incluso diferencias modestas son estadísticamente detectables.

**ε² pequeño:** Las diferencias en `duration` son **estadísticamente significativas** pero **prácticamente más sutiles** que en otras variables.

**Explicación contextual:**

| Categoría | Mediana duration |
|-----------|------------------|
| Normal | 0 |
| DoS | 0 |
| Probe | 0 |
| R2L | 1 |
| U2R | 53 |

- 3 de 5 categorías tienen mediana = 0
- Solo U2R tiene mediana muy diferente (53 seg)
- La diferencia existe (p < 1e-10), pero es menos "dramática" que serror_rate donde DoS=1.0 vs Normal=0.0

**Lección:** Siempre reportar TANTO p-valor (significancia) COMO tamaño del efecto (magnitud).

---

### **P1.21: En el Radar Plot, ¿cómo se conectan los resultados de Kruskal-Wallis con las formas de los polígonos?**

**R:** 

El Radar Plot visualiza **medianas normalizadas** de las 5 variables para cada categoría.

**Polígonos claramente separados = Altos valores de ε² en Kruskal-Wallis**

**Ejemplo:**
- **DoS** tiene pico extremo en `serror_rate` (valor normalizado ≈ 1.0)
  - Corresponde a ε² = 0.573 en Kruskal-Wallis → Efecto grande
  
- **U2R** tiene pico único en `dst_bytes` (valor normalizado alto)
  - Corresponde a ε² = 0.573 → Efecto grande

**Polígonos solapados indicarían ε² pequeño → Categorías indistinguibles en esas variables**

**En nuestro proyecto:** Polígonos claramente distinguibles confirman visualmente los resultados cuantitativos de Kruskal-Wallis.

---

# PARTE 2: ANÁLISIS DE COMPONENTES PRINCIPALES (PCA)

## **NIVEL 1: CONCEPTOS BÁSICOS**

### **P2.1: ¿Qué es PCA en términos simples?**

**R:** PCA (Principal Component Analysis) es una técnica de **reducción dimensional** que transforma un conjunto de variables **posiblemente correlacionadas** en un conjunto **más pequeño** de variables **no correlacionadas** (ortogonales) llamadas **componentes principales**.

**Analogía:**
Imagina describir una película con 50 características (director, actor principal, género, presupuesto, críticas...). PCA encuentra combinaciones de esas características que capturan lo "esencial": quizás 5 "meta-características" como "calidad general", "comercialidad", "innovación artística", etc.

**En nuestro proyecto:**
- 32 variables originales (muchas correlacionadas)
- PCA las reduce a 8 componentes principales
- Preservando 71.85% de la información original

---

### **P2.2: ¿Por qué es crítico estandarizar antes de PCA?**

**R:** Porque PCA busca direcciones de **máxima varianza**, y variables con mayor escala dominarían los componentes.

**Ejemplo sin estandarización:**

| Variable | Rango | Varianza |
|----------|-------|----------|
| `dst_bytes` | 0 - 100,000 | σ² = 50,000,000 |
| `serror_rate` | 0 - 1 | σ² = 0.25 |

PCA daría **99.9% del peso a `dst_bytes`** y casi ignoraría `serror_rate`, aunque esta última sea crucial para detectar DoS.

**Estandarización (z-score):**
$$z = \frac{x - \mu}{\sigma}$$

**Resultado:**
- Todas las variables tienen media = 0, desviación estándar = 1
- Misma "escala de importancia"

**En nuestro proyecto:**
```python
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)
```

---

### **P2.3: ¿Qué son los eigenvalues (valores propios) en PCA?**

**R:** Los **eigenvalues** de la matriz de covarianza representan la **varianza explicada** por cada componente principal.

**Definición matemática:**
$$\Sigma v_i = \lambda_i v_i$$

Donde:
- Σ = Matriz de covarianza
- $v_i$ = i-ésimo eigenvector (componente principal)
- $\lambda_i$ = i-ésimo eigenvalue (varianza explicada por $v_i$)

**Propiedades:**
1. Hay tantos eigenvalues como variables (p eigenvalues)
2. $\lambda_1 \geq \lambda_2 \geq ... \geq \lambda_p \geq 0$ (ordenados de mayor a menor)
3. $\sum_{i=1}^{p} \lambda_i = \sum_{j=1}^{p} \text{Var}(X_j)$ (varianza total)

**Interpretación:**
- $\lambda_1$ grande → PC1 captura mucha variabilidad
- $\lambda_p$ pequeño → PC_p captura poca variabilidad (casi redundante)

---

### **P2.4: ¿Qué son los eigenvectors (vectores propios) en PCA?**

**R:** Los **eigenvectors** definen las **direcciones** (combinaciones lineales de variables originales) que forman los componentes principales.

**Cada eigenvector $v_i$ es un vector de p elementos:**
$$v_i = [v_{i1}, v_{i2}, ..., v_{ip}]^T$$

**El componente principal i se calcula como:**
$$PC_i = v_{i1} \cdot X_1 + v_{i2} \cdot X_2 + ... + v_{ip} \cdot X_p$$

**Propiedades:**
1. **Ortogonales:** $v_i \cdot v_j = 0$ si $i \neq j$ → Componentes no correlacionados
2. **Normalizados:** $\|v_i\| = 1$ → Longitud unitaria
3. **Únicos salvo signo:** $v_i$ y $-v_i$ son equivalentes

**En nuestro proyecto:**
- Cada uno de los 8 PCs es un eigenvector
- Define cómo combinar las 32 variables originales

---

### **P2.5: ¿Qué son los "loadings" en PCA?**

**R:** Los **loadings** son los elementos de los eigenvectors. Representan la **contribución** (peso) de cada variable original a un componente principal.

**Para PC_i:**
$$PC_i = l_{i1} \cdot X_1 + l_{i2} \cdot X_2 + ... + l_{ip} \cdot X_p$$

Donde $l_{ij}$ es el loading de la variable $j$ en $PC_i$.

**Interpretación:**

| |loading| | Contribución |
|----------|--------------|
| ≈ 1 | Muy alta (variable domina el componente) |
| 0.3 - 0.7 | Moderada (variable contribuye significativamente) |
| ≈ 0 | Muy baja (variable casi irrelevante para este componente) |

**Signo del loading:**
- Positivo (+): Variable aumenta cuando PC aumenta
- Negativo (-): Variable disminuye cuando PC aumenta

**En nuestro proyecto:**
- PC1 dominado por: `same_srv_rate` (0.35), `dst_host_same_srv_rate` (0.34)
- Estas variables tienen loadings altos → Contribuyen fuertemente a PC1

---

## **NIVEL 2: INTERPRETACIÓN DE RESULTADOS**

### **P2.6: En tu proyecto, ¿por qué 8 componentes capturan 71.85% de varianza y no 100%?**

**R:** Porque **descartamos** los últimos 24 componentes (de 32 totales) que capturaban el 28.15% restante de varianza.

**Razón:** Los componentes finales capturan principalmente:
- **Ruido:** Variabilidad aleatoria sin patrón
- **Información redundante:** Variaciones muy específicas sin valor discriminante
- **Varianza marginal:** Cada componente adicional aporta cada vez menos

**Trade-off:**
- Más componentes → Más información preservada, pero más complejidad
- Menos componentes → Más parsimonia, pero posible pérdida de información

**Decisión en nuestro proyecto:**
- 8 PCs (71.85% varianza) vs 13 PCs (87.16% varianza)
- Diferencia en AUC: 0.9496 vs 0.9541 (< 0.5%)
- **Conclusión:** 8 PCs son suficientes (principio de parsimonia)

---

### **P2.7: ¿Qué es el "Scree Plot" y cómo identificas el "codo"?**

**R:** El **Scree Plot** grafica la varianza explicada (eigenvalue) de cada componente en orden decreciente.

**Eje X:** Número de componente (PC1, PC2, PC3, ...)
**Eje Y:** Varianza explicada (% o eigenvalue)

**Identificar el "codo":**

Buscar el punto donde:
1. La pendiente cambia **abruptamente** de empinada a plana
2. Los componentes adicionales aportan **rendimientos decrecientes**

**Analogía:**
Como una "escalera" donde:
- Antes del codo: Bajas escalones altos (mucha ganancia por componente)
- Después del codo: Bajas escalones pequeños (poca ganancia por componente)

**En nuestro proyecto:**
```
PC1: 15.23% |■■■■■■■■■■
PC2: 12.45% |■■■■■■■
PC3: 10.12% |■■■■■
PC4:  8.76% |■■■■
PC5:  7.89% |■■■
PC6:  4.32% |■■  ← CODO aquí
PC7:  3.21% |■
PC8:  2.87% |■
...
```

El codo está en PC5-PC6 porque después de PC6, cada componente aporta < 5%.

---

### **P2.8: ¿Por qué elegiste 8 PCs si el codo está en PC5-PC6?**

**R:** **El codo es una GUÍA, no una regla absoluta.**

Decidimos 8 PCs mediante **validación cruzada empírica**:

**Comparación:**

| Configuración | Varianza Capturada | AUC Promedio | Comentario |
|---------------|-------------------|--------------|------------|
| 5 PCs | 60.75% | 0.9348 | Bajo rendimiento |
| **8 PCs** | **71.85%** | **0.9496** | **Balance óptimo** ✅ |
| 13 PCs | 87.16% | 0.9541 | Ganancia marginal (< 0.5%) |

**Razón:** 8 PCs → 13 PCs requiere 62.5% más features (+5 componentes) para ganar apenas 0.5% en AUC.

**Principio de parsimonia (Occam's Razor):** Entre modelos con desempeño similar, elegir el más simple.

**Respuesta en defensa:** "El codo sugiere 5-6 componentes como punto de inflexión, pero validación cruzada mostró que 8 PCs optimizan el balance complejidad-desempeño sin sobreajuste."

---

### **P2.9: ¿Qué significa que los componentes principales son "ortogonales"?**

**R:** **Ortogonalidad** significa que los componentes principales son **no correlacionados** (correlación = 0).

**Matemáticamente:**
$$v_i \cdot v_j = 0 \quad \text{si } i \neq j$$

Y por tanto:
$$\text{Cor}(PC_i, PC_j) = 0 \quad \text{si } i \neq j$$

**Consecuencia práctica:**

Si las variables originales tenían multicolinealidad:
- `dst_host_srv_count` y `srv_count` con r = 0.85 (alta correlación)

Después de PCA:
- PC1, PC2, ..., PC8 tienen correlación = 0 entre sí
- **Elimina multicolinealidad** que causaría problemas en regresión

**Ventaja:** Componentes aportan información **independiente** (no redundante).

---

### **P2.10: Explica PC1 en tu proyecto: loadings altos en `same_srv_rate`, `dst_host_same_srv_rate`, `dst_host_serror_rate`**

**R:** 

**Loadings de PC1:**
- `same_srv_rate`: 0.35
- `dst_host_same_srv_rate`: 0.34
- `dst_host_serror_rate`: 0.32

**Interpretación conceptual:**

PC1 captura patrones relacionados con **consistencia del servicio y errores SYN en el mismo host**.

**Variables involucradas miden:**
- `same_srv_rate`: % de conexiones al mismo servicio
- `dst_host_same_srv_rate`: % de conexiones al mismo servicio en el host destino
- `dst_host_serror_rate`: Tasa de errores SYN en el host destino

**Patrón subyacente:**

**Ataques DoS típicos:**
- Bombardean el MISMO servicio (HTTP) en el MISMO host
- Generan errores SYN masivos
- PC1 capturaría este patrón con valor ALTO

**Tráfico Normal:**
- Diversidad de servicios
- Pocos errores SYN
- PC1 tendría valor BAJO

**Conclusión:** PC1 es un "detector de patrones de DoS" emergido naturalmente de los datos.

---

## **NIVEL 3: LIMITACIONES Y SUPUESTOS**

### **P2.11: ¿PCA puede detectar relaciones no lineales entre variables?**

**R:** **NO.** PCA es una técnica **lineal** que solo captura relaciones lineales.

**Ejemplo de limitación:**

Supón dos variables con relación cuadrática:
- $X_2 = X_1^2$
- Correlación lineal ≈ 0 (curva, no línea)

PCA NO detectaría que $X_2$ es completamente dependiente de $X_1$.

**Alternativas para relaciones no lineales:**
- **Kernel PCA:** Usa funciones kernel para mapear a espacio de mayor dimensión donde las relaciones se vuelven lineales
- **t-SNE:** Reduce dimensión preservando estructura local (vecindarios)
- **UMAP:** Similar a t-SNE pero más rápido y preserva estructura global

**En nuestro proyecto:**
- Asumimos que relaciones discriminantes son mayormente lineales
- Resultados (AUC 0.95) sugieren que esta suposición fue razonable

---

### **P2.12: ¿PCA es sensible a outliers?**

**R:** **SÍ.** PCA busca direcciones de **máxima varianza**, y los outliers pueden inflar artificialmente la varianza en ciertas direcciones.

**Ejemplo:**

Variables $X_1$ y $X_2$ con 1000 observaciones:
- 999 observaciones: $X_1 \in [0, 10]$, $X_2 \in [0, 10]$
- 1 outlier: $X_1 = 1000$, $X_2 = 1$

PC1 estará dominado por la dirección del outlier (eje $X_1$), aunque la mayoría de la variabilidad está en otro lugar.

**En nuestro proyecto:**
- Teníamos 18.6% de outliers en `dst_bytes`
- **Decisión:** Mantuvimos outliers porque son señal de ataque (no ruido)
- **Consecuencia:** Componentes capturan variabilidad real del comportamiento anómalo

**Alternativa robusta:** Robust PCA (usa medianas y MAD en lugar de medias y desviaciones estándar)

---

### **P2.13: ¿Qué sucede si tienes más variables (p) que observaciones (n)?**

**R:** Obtienes **degeneración del problema**:

**Matriz de covarianza Σ (p × p):**
- Si n < p: Σ es **singular** (no invertible)
- Tiene como máximo $\text{min}(n, p)$ eigenvalues no ceros

**Consecuencia:**
- Solo puedes extraer n-1 componentes principales significativos
- Los restantes (p - n + 1) tendrán eigenvalue = 0

**Solución:**
- **Regularización:** Añadir penalización (ej: Ridge PCA)
- **Selección de variables:** Reducir p antes de PCA
- **Aumentar n:** Recolectar más datos

**En nuestro proyecto:**
- n = 25,192, p = 32
- n >> p → Sin problema de degeneración

---

## **NIVEL 4: EVALUACIÓN DE SEPARABILIDAD**

### **P2.14: ¿Qué es el Silhouette Score y qué mide?**

**R:** El **Silhouette Score** mide qué tan bien está "agrupada" cada observación con su propia categoría comparado con otras categorías.

**Para una observación i:**
$$s(i) = \frac{b(i) - a(i)}{\max\{a(i), b(i)\}}$$

Donde:
- $a(i)$ = Distancia promedio a observaciones de **su misma categoría**
- $b(i)$ = Distancia promedio a observaciones de la **categoría más cercana diferente**

**Interpretación:**

| Silhouette | Significado |
|------------|-------------|
| $s \approx 1$ | Muy bien asignada (cerca de su grupo, lejos de otros) |
| $s \approx 0$ | En el límite entre clusters |
| $s < 0$ | Probablemente mal asignada (más cerca de otro cluster) |

**Score global:**
$$\text{Silhouette Score} = \frac{1}{n} \sum_{i=1}^{n} s(i)$$

---

### **P2.15: En tu proyecto, Silhouette 2D = 0.13 (bajo). ¿Significa que PCA falló?**

**R:** **NO. Al contrario, confirma nuestra hipótesis.**

**Interpretación correcta:**

**Silhouette bajo en 2D (PC1 + PC2):**
- Indica que en espacio 2D, las categorías se **solapan**
- PC1 y PC2 (que capturan 27.68% de varianza) **no son suficientes** para separar completamente

**PERO:**

**AUC alto en 8D (0.9496):**
- Indica que en espacio 8D (71.85% varianza), las categorías **SÍ son separables**

**Conclusión:**
- La información discriminante está **distribuida** en las 8 dimensiones
- NO está concentrada solo en PC1 y PC2 (como esperaríamos en un problema "fácil")

**Analogía:**
Imagina describir un objeto 3D con solo 2 coordenadas (x, y). La proyección 2D puede solaparse, pero en 3D (x, y, z) los objetos están claramente separados.

---

### **P2.16: Si Silhouette 3D (0.119) es peor que 2D (0.129), ¿eso invalida PC3?**

**R:** **NO.** Esto es un artefacto de cómo se calcula Silhouette en proyecciones.

**Explicación:**

**Silhouette mide distancias Euclidianas** en el espacio proyectado.

Al proyectar de 8D → 3D, las distancias se "comprimen" desigualmente:
- Algunas categorías que estaban separadas en 8D pueden solaparse en 3D
- La proyección 3D puede ser "peor" que 2D si PC3 añade confusión visual más que claridad

**Lo que importa:**
- **Desempeño en 8D completo:** AUC = 0.9496 ✅
- **NO el Silhouette en proyecciones 2D/3D** (solo para visualización)

**En defensa:**
"Silhouette en 2D/3D es una métrica de **visualización**, no de **clasificación**. El AUC de 0.95 en 8D confirma que los componentes capturan separabilidad, aunque no sea visualizable en 2-3 dimensiones."

---

### **P2.17: ¿Por qué graficar proyecciones 3D si no son separables visualmente?**

**R:** **Razones pedagógicas y exploratorias:**

1. **Mostrar limitación de visualización:** Ayuda a entender que problemas complejos no son reducibles a 2-3 dimensiones interpretables

2. **Identificar patrones parciales:** Aunque no haya separación completa, podemos ver:
   - Normal y DoS tienen cierta separación en PC1
   - U2R es visible como outliers en PC2

3. **Validar dirección correcta:** Si NO hubiera NINGUNA estructura visible en 2D/3D, sería señal de alarma (los componentes podrían ser ruido puro)

4. **Comunicación con stakeholders:** Gráficos 3D son más intuitivos que decir "confía en las 8 dimensiones abstractas"

**En nuestro proyecto:** La proyección 3D muestra estructura parcial, confirmando que los PCs capturan patrones reales (aunque la separación completa requiera 8D).

---

## **NIVEL 5: CONEXIÓN CON P1 Y P3**

### **P2.18: ¿Cómo se relacionan los resultados de Kruskal-Wallis (P1) con los loadings de PCA (P2)?**

**R:** **Variables con alto ε² en Kruskal-Wallis tienden a tener loadings altos en los primeros PCs.**

**Ejemplo:**

**Kruskal-Wallis (P1):**
- `serror_rate`: ε² = 0.573 (efecto grande)
- `dst_bytes`: ε² = 0.573 (efecto grande)

**PCA (P2):**
- PC1 tiene loading alto en variables relacionadas con tasas de servicio y errores SYN
- PC2 tiene loading moderado en `dst_bytes`

**Interpretación:**

Variables que **discriminan fuertemente entre categorías** (alto ε²) contienen **mucha información útil**, por lo que PCA las captura en los primeros componentes (que explican más varianza).

**Validación de coherencia:**

Si una variable tiene ε² alto en P1 pero loading bajo en todos los PCs de P2, indicaría **inconsistencia metodológica** → revisar análisis.

**En nuestro proyecto:** Coherencia confirmada → Variables discriminantes de P1 aparecen en PCs de P2.

---

### **P2.19: ¿Los componentes principales preservan el poder discriminante detectado en P1?**

**R:** **SÍ, si seleccionas suficientes componentes.**

**Evidencia:**

**P1 (Kruskal-Wallis):** Detectamos diferencias robustas (p < 1e-10, ε² > 0.49)

**P2 (PCA):** 8 PCs capturan 71.85% de varianza

**P3 (Regresión Logística con 8 PCs):** AUC = 0.9496 (excelente discriminación)

**Cadena lógica:**

```
Variables discriminantes (P1)
         ↓
Información capturada en PCs (P2)
         ↓
PCs usados en clasificación (P3)
         ↓
Alta precisión (AUC 0.95)
```

**Si PCA hubiera perdido información crítica:** AUC en P3 sería bajo (< 0.7).

**Conclusión:** Los 8 PCs preservan suficiente información discriminante para clasificación exitosa.

---

### **P2.20: ¿Por qué usar componentes principales como features en regresión logística (P3) en lugar de variables originales?**

**R:** **Ventajas de usar PCs:**

**1. Eliminación de multicolinealidad:**
- Variables originales: 14 pares con |r| > 0.8
- Componentes principales: Correlación = 0 (ortogonales)
- **Consecuencia:** Coeficientes de regresión más estables, sin problemas de inflación de varianza

**2. Reducción dimensional:**
- Variables originales: 32 features
- Componentes principales: 8 features (75% menos)
- **Consecuencia:** Menor riesgo de overfitting, entrenamiento más rápido

**3. Ruido filtrado:**
- Los últimos 24 PCs (28% varianza) capturan principalmente ruido
- Al descartarlos, mejoramos la relación señal/ruido
- **Consecuencia:** Modelos más generalizables

**Desventaja:**
- Pérdida de interpretabilidad: "PC3 tiene coeficiente 1.2" es menos claro que "duration tiene coeficiente 0.5"

**En nuestro proyecto:** Priorizamos desempeño y robustez sobre interpretabilidad directa (aunque análisis de loadings permite interpretación indirecta).

---

# PARTE 3: REGRESIÓN LOGÍSTICA

## **NIVEL 1: CONCEPTOS BÁSICOS**

### **P3.1: ¿Por qué se llama "regresión" logística si es un modelo de clasificación?**

**R:** **Razón histórica:** El modelo se deriva de la regresión lineal aplicando una transformación (función logit).

**Regresión lineal:**
$$y = \beta_0 + \beta_1 x_1 + ... + \beta_p x_p$$
- Predice valores continuos en $(-\infty, +\infty)$

**Regresión logística:**
$$\log\left(\frac{P(Y=1)}{1-P(Y=1)}\right) = \beta_0 + \beta_1 x_1 + ... + \beta_p x_p$$
- Predice **log-odds** (que es continuo)
- Se transforma a probabilidad mediante función sigmoide: $P(Y=1) = \frac{1}{1 + e^{-z}}$

**Resultado final:** Clasificación binaria (Y = 0 o 1) según threshold (típicamente 0.5).

**Nombre técnico más preciso:** Modelo lineal generalizado (GLM) con función de enlace logit.

---

### **P3.2: ¿Qué es la función sigmoide y por qué se usa?**

**R:** La **función sigmoide (logística)** transforma cualquier valor real en el rango [0, 1]:

$$\sigma(z) = \frac{1}{1 + e^{-z}} = \frac{e^z}{1 + e^z}$$

**Propiedades clave:**

1. **Rango acotado:** $\sigma(z) \in (0, 1)$ → Siempre probabilidad válida
2. **Monotonía creciente:** $\sigma'(z) > 0$ → A mayor z, mayor probabilidad
3. **Simetría:** $\sigma(-z) = 1 - \sigma(z)$
4. **Punto medio:** $\sigma(0) = 0.5$ → Threshold natural
5. **Límites:**
   - $\lim_{z \to +\infty} \sigma(z) = 1$ (certeza de clase 1)
   - $\lim_{z \to -\infty} \sigma(z) = 0$ (certeza de clase 0)

**Forma de la curva:**
```
P(Y=1)
1.0 |              ___________
    |           __/
0.5 |        __/  (z=0)
    |     __/
0.0 |____/
    +----+----+----+----+----> z
        -4   -2    0    2    4
```

**Por qué se usa:**
- Mapea el predictor lineal $z = \beta^T x$ (rango infinito) a probabilidad (rango [0,1])
- Interpretable como probabilidad de pertenencia a clase 1

---

### **P3.3: ¿Qué son los "odds" y cómo se relacionan con probabilidad?**

**R:** 

**Odds (momios):**
$$\text{Odds} = \frac{P(Y=1)}{P(Y=0)} = \frac{P(Y=1)}{1 - P(Y=1)}$$

**Interpretación:**

| Odds | Significado |
|------|-------------|
| 1 | Igualmente probable Y=1 que Y=0 (P=0.5) |
| 2 | Dos veces más probable Y=1 que Y=0 (P≈0.67) |
| 0.5 | Dos veces más probable Y=0 que Y=1 (P≈0.33) |
| 9 | Nueve veces más probable Y=1 (P=0.9) |

**Conversión odds ↔ probabilidad:**

$$\text{Odds} = \frac{P}{1-P} \quad \Rightarrow \quad P = \frac{\text{Odds}}{1 + \text{Odds}}$$

**Ejemplo:**
- P(Ataque) = 0.8
- Odds = 0.8 / 0.2 = 4
- "Ataque es 4 veces más probable que No-Ataque"

---

### **P3.4: ¿Qué es el log-odds (logit) y por qué usarlo?**

**R:** 

**Log-odds (logit):**
$$\text{logit}(P) = \log\left(\frac{P}{1-P}\right) = \log(\text{Odds})$$

**Ventaja:** Transforma probabilidades [0, 1] a escala **lineal** $(-\infty, +\infty)$.

**Relación con el modelo:**
$$\text{logit}(P) = \beta_0 + \beta_1 x_1 + ... + \beta_p x_p$$

**Interpretación:**
- El lado derecho ($\beta^T x$) es una combinación **lineal** de predictores
- El logit conecta este predictor lineal con la probabilidad mediante una transformación monótona

**Ventaja práctica:** Los coeficientes $\beta_j$ tienen interpretación directa como cambios en log-odds.

---

### **P3.5: ¿Cómo se interpretan los coeficientes en regresión logística?**

**R:** 

**Coeficiente $\beta_j$:**

**Aumento de 1 unidad en $x_j$ (manteniendo otras variables constantes) implica:**

$$\Delta \text{log-odds} = \beta_j$$

**En términos de Odds Ratio:**
$$\text{OR} = e^{\beta_j}$$

**Ejemplo:**
- $\beta_1 = 0.5$ → OR = $e^{0.5}$ = 1.65
  - **Interpretación:** Por cada unidad que aumenta $x_1$, los odds de Y=1 se multiplican por 1.65 (aumentan 65%)

- $\beta_2 = -0.3$ → OR = $e^{-0.3}$ = 0.74
  - **Interpretación:** Por cada unidad que aumenta $x_2$, los odds de Y=1 se multiplican por 0.74 (disminuyen 26%)

**En nuestro proyecto:**
- Si PC1 tiene $\beta = 2.5$ en el modelo DoS:
  - OR = $e^{2.5}$ = 12.18
  - **Interpretación:** Por cada unidad que aumenta PC1, los odds de ser DoS se multiplican por 12.18

---

## **NIVEL 2: ESTIMACIÓN Y ENTRENAMIENTO**

### **P3.6: ¿Qué es Máxima Verosimilitud (MLE) y por qué se usa en regresión logística?**

**R:** 

**Máxima Verosimilitud (Maximum Likelihood Estimation, MLE):** Método para estimar parámetros $\beta$ que **maximizan la probabilidad de observar los datos reales**.

**Función de verosimilitud:**
$$L(\beta) = \prod_{i=1}^{n} P(Y=y_i \mid x_i; \beta)$$

Donde:
$$P(Y=y_i \mid x_i; \beta) = p_i^{y_i} (1-p_i)^{1-y_i}$$

Con $p_i = P(Y=1 \mid x_i) = \sigma(\beta^T x_i)$.

**Log-verosimilitud (más manejable):**
$$\ell(\beta) = \sum_{i=1}^{n} \left[ y_i \log(p_i) + (1-y_i) \log(1-p_i) \right]$$

**Objetivo:** Encontrar $\beta^*$ que maximiza $\ell(\beta)$.

**Por qué MLE:**
- Propiedades asintóticas deseables (consistencia, eficiencia)
- Coincide con minimización de entropía cruzada (loss function en ML)
- Fundamento probabilístico sólido

---

### **P3.7: ¿Por qué no hay solución cerrada en regresión logística (como en regresión lineal)?**

**R:** 

**En regresión lineal:**
$$\beta^* = (X^T X)^{-1} X^T y$$
- Derivada de la función de pérdida (MSE) es lineal en $\beta$
- Se puede resolver directamente

**En regresión logística:**
$$\frac{\partial \ell}{\partial \beta_j} = \sum_{i=1}^{n} (y_i - p_i) x_{ij}$$

Donde $p_i = \sigma(\beta^T x_i) = \frac{1}{1 + e^{-\beta^T x_i}}$.

**Problema:** $p_i$ es **no lineal** en $\beta$ (función sigmoide).

**Consecuencia:** No se puede despejar $\beta$ algebraicamente.

**Solución:** Métodos iterativos:
- **Descenso de gradiente:** Actualización $\beta^{(t+1)} = \beta^{(t)} + \alpha \nabla \ell$
- **Newton-Raphson:** Usa Hessiana (segunda derivada) para convergencia más rápida
- **L-BFGS:** Versión de memoria limitada de BFGS (quasi-Newton)

**En sklearn:** Usa L-BFGS por defecto (eficiente y robusto).

---

### **P3.8: ¿Qué es la regularización L2 (Ridge) en regresión logística?**

**R:** 

**Regularización L2** añade un término de penalización a la función objetivo:

$$\ell_{regularizada}(\beta) = \ell(\beta) - \lambda \sum_{j=1}^{p} \beta_j^2$$

Donde:
- $\ell(\beta)$ = Log-verosimilitud original
- $\lambda$ = Parámetro de regularización (hiperparámetro)
- $\sum \beta_j^2$ = Norma L2 al cuadrado

**Efecto:**
- Penaliza coeficientes grandes
- Fuerza $\beta_j$ hacia cero (pero no exactamente cero)
- Reduce varianza del modelo (a costa de sesgo)

**En sklearn:**
```python
LogisticRegression(penalty='l2', C=1.0)
```

Donde $C = \frac{1}{\lambda}$ (inverso de penalización).

**Interpretación de C:**
- **C grande (ej: 10):** Poca regularización → Modelo complejo, posible overfitting
- **C pequeña (ej: 0.1):** Mucha regularización → Modelo simple, posible underfitting
- **C = 1.0:** Regularización estándar (valor por defecto)

**Por qué usarla:**
- Previene **overfitting** cuando p es grande o n es pequeño
- Estabiliza coeficientes cuando hay multicolinealidad (aunque PCA ya resolvió esto en nuestro caso)

---

### **P3.9: ¿Por qué no se regulariza el intercepto ($\beta_0$)?**

**R:** 

**Razón:** El intercepto representa el **baseline** (log-odds cuando todos los predictores = 0). Regularizarlo sesgaria las predicciones.

**Ejemplo:**

Si en tu dataset:
- 90% son clase 0 (no-ataque)
- 10% son clase 1 (ataque)

El intercepto debería capturar este **desbalance base**:
$$\beta_0 = \log\left(\frac{0.1}{0.9}\right) = \log(0.111) \approx -2.2$$

Si regularizas $\beta_0$, lo forzarías hacia 0, haciendo que el modelo prediga P ≈ 0.5 incluso sin predictores → Predicción incorrecta.

**En la práctica:**
- Sklearn **NO regulariza el intercepto** por defecto
- Solo se regulariza $\beta_1, \beta_2, ..., \beta_p$

---

## **NIVEL 3: ESTRATEGIA ONE-VS-REST**

### **P3.10: ¿Qué es la estrategia One-vs-Rest (OvR)?**

**R:** 

**One-vs-Rest (OvR)**, también llamada **One-vs-All (OvA)**, es una estrategia para extender clasificadores binarios a problemas multiclase.

**Para k clases, entrenar k modelos binarios:**

- Modelo 1: Clase 1 vs {Clase 2, 3, 4, ..., k}
- Modelo 2: Clase 2 vs {Clase 1, 3, 4, ..., k}
- ...
- Modelo k: Clase k vs {Clase 1, 2, ..., k-1}

**Predicción:**
1. Calcular $P_1(Y=1), P_2(Y=2), ..., P_k(Y=k)$ con cada modelo
2. Asignar la clase con **mayor probabilidad**:
   $$\hat{y} = \arg\max_i P_i(Y=i)$$

**En nuestro proyecto:**
- k = 4 modelos (DoS, Probe, R2L, U2R)
- Normal no necesita modelo (se infiere como "ninguna de las 4")

---

### **P3.11: ¿Cuál es la alternativa a OvR y cuándo usarla?**

**R:** 

**Alternativa: One-vs-One (OvO)**

Entrenar $\binom{k}{2}$ modelos binarios entre cada **par** de clases.

**Para k=5 clases:**
- $\binom{5}{2} = 10$ modelos
- Normal vs DoS, Normal vs Probe, DoS vs Probe, DoS vs R2L, ...

**Predicción:** Votación mayoritaria (cada modelo "vota" por una clase).

**Comparación:**

| Aspecto | OvR | OvO |
|---------|-----|-----|
| **# Modelos** | k | $\binom{k}{2}$ |
| **Tamaño datos por modelo** | n completo | ~2n/k |
| **Interpretabilidad** | Alta ("¿Es DoS?") | Baja (¿Qué significa "DoS vs R2L"?) |
| **Desbalance** | Puede ser severo | Más balanceado en cada modelo |
| **Tiempo entrenamiento** | O(k) | O(k²) |

**Cuándo usar OvO:**
- k pequeño (2-5 clases)
- Desbalance extremo (OvO balancea mejor)
- Clasificadores complejos (entrenar en subconjuntos pequeños es más rápido)

**Por qué elegimos OvR en nuestro proyecto:**
- Solo 4 modelos (eficiente)
- Interpretabilidad clara ("modelo para DoS")
- Podemos ajustar `class_weight` independientemente por modelo

---

### **P3.12: ¿Qué problemas tiene OvR con clases desbalanceadas?**

**R:** 

**Problema:** En OvR, cada modelo binario tiene:
- Clase positiva: 1 categoría (ej: DoS)
- Clase negativa: k-1 categorías restantes (ej: Normal + Probe + R2L + U2R)

**Con desbalance extremo:**

**Ejemplo para U2R (0.04%):**
- Clase positiva (U2R): 11 instancias
- Clase negativa (resto): 25,181 instancias
- **Ratio:** 1:2289

**Consecuencia sin ajuste:**
- El modelo aprende a predecir siempre clase negativa (accuracy 99.96%)
- No detecta ningún U2R (recall = 0)

**Solución en nuestro proyecto:**
```python
LogisticRegression(class_weight='balanced')
```

Ajusta pesos inversamente proporcionales a frecuencia:
$$w_c = \frac{n}{k \cdot n_c}$$

**Efecto:**
- U2R recibe peso ~2289x mayor que resto
- El modelo penaliza más los errores en U2R
- Mejora recall de U2R (a costa de precision)

---

## **NIVEL 4: MÉTRICAS DE EVALUACIÓN**

### **P3.13: ¿Por qué Accuracy no es buena métrica con datos desbalanceados?**

**R:** 

**Accuracy:**
$$\text{Accuracy} = \frac{TP + TN}{TP + TN + FP + FN}$$

**Problema con desbalance:**

**Ejemplo para U2R (0.04%):**

Clasificador trivial que **siempre predice "No-U2R"**:
- TN = 25,181 (todos los no-U2R correctos)
- TP = 0 (ningún U2R detectado)
- FP = 0, FN = 11

**Accuracy = 25,181 / 25,192 = 99.96%** ✅

**Pero el modelo es INÚTIL:** No detecta ningún ataque U2R (recall = 0).

**Lección:** Accuracy da falsa sensación de buen desempeño cuando la clase minoritaria es irrelevante para el cálculo.

**Alternativas robustas:**
- **Precision, Recall, F1:** Focalizan en clase positiva (minoritaria)
- **AUC-ROC:** Evalúa discriminación en todos los thresholds

---

### **P3.14: ¿Qué es el trade-off entre Precision y Recall?**

**R:** 

**Precision:**
$$\text{Precision} = \frac{TP}{TP + FP}$$
- "De todas mis detecciones, ¿cuántas fueron correctas?"
- Alta precision → Pocas falsas alarmas

**Recall:**
$$\text{Recall} = \frac{TP}{TP + FN}$$
- "De todos los ataques reales, ¿cuántos detecté?"
- Alto recall → Pocas amenazas escapan

**Trade-off:**

Ajustando el **threshold** de decisión:

| Threshold | Precision | Recall | Consecuencia |
|-----------|-----------|--------|--------------|
| **0.1** (bajo) | Baja | Alta | Muchas falsas alarmas, pero detecta casi todo |
| **0.5** (medio) | Media | Media | Balance |
| **0.9** (alto) | Alta | Baja | Pocas falsas alarmas, pero se escapan amenazas |

**Ejemplo en ciberseguridad:**

- **Priorizar Recall:** Sistema crítico (central nuclear) → No podemos dejar pasar ningún ataque
- **Priorizar Precision:** Sistema de bajo riesgo (blog personal) → Minimizar falsas alarmas que molestan usuarios

**F1-Score** busca el balance:
$$F_1 = 2 \cdot \frac{P \cdot R}{P + R}$$

---

### **P3.15: ¿Qué es la curva ROC y cómo se construye?**

**R:** 

**ROC (Receiver Operating Characteristic):** Gráfica de **TPR (Recall)** vs **FPR** variando el threshold.

**Ejes:**
- **Eje Y:** TPR = TP / (TP + FN) = Recall
- **Eje X:** FPR = FP / (FP + TN)

**Construcción:**

1. Ordenar predicciones por probabilidad decreciente
2. Iniciar con threshold = 1.0 (predice todo como 0) → (FPR=0, TPR=0)
3. Bajar threshold gradualmente → Cada vez que threshold pasa una instancia:
   - Si era positiva (Y=1): TPR aumenta
   - Si era negativa (Y=0): FPR aumenta
4. Terminar con threshold = 0.0 (predice todo como 1) → (FPR=1, TPR=1)

**Curva ideal:**
```
TPR
1.0 |___   ← Modelo perfecto (TPR=1, FPR=0)
    |   \
0.5 |    \__ ← Modelo real
    |       ----___
0.0 |______________→ FPR
    0.0   0.5   1.0
```

**Línea diagonal (FPR=TPR):** Clasificador aleatorio (lanzar moneda).

---

### **P3.16: ¿Qué es AUC-ROC y cómo se interpreta?**

**R:** 

**AUC (Area Under the ROC Curve):** Área bajo la curva ROC.

**Interpretación probabilística:**

AUC = Probabilidad de que el modelo asigne **mayor score** a una instancia positiva aleatoria que a una instancia negativa aleatoria.

**Valores:**

| AUC | Interpretación | Desempeño |
|-----|----------------|-----------|
| 1.0 | Perfecto | Separa completamente clases |
| 0.9-1.0 | Excepcional | Excelente discriminación |
| 0.8-0.9 | Excelente | Muy buena discriminación |
| 0.7-0.8 | Aceptable | Discriminación moderada |
| 0.5-0.7 | Pobre | Apenas mejor que azar |
| 0.5 | Azar | Clasificador aleatorio |
| < 0.5 | Peor que azar | Modelo invertido (usar 1-predicción) |

**Ventaja clave:**
- **Robusto al desbalance:** Evalúa discriminación en **todos los thresholds**, no solo uno fijo
- **Independiente del threshold:** Mide capacidad de ordenamiento (ranking)

**En nuestro proyecto:**
- DoS: AUC = 0.9902 → Excepcional
- Probe: AUC = 0.9868 → Excepcional
- R2L: AUC = 0.9593 → Excelente
- U2R: AUC = 0.8619 → Excelente (considerando n=11)

---

### **P3.17: Explica la "paradoja" AUC alto pero F1 bajo en R2L**

**R:** 

**Observación en R2L:**
- AUC = 0.9593 (excelente)
- F1 = 0.1515 (bajo)

**Explicación:**

**AUC mide DISCRIMINACIÓN (ranking):**
- El modelo asigna scores más altos a R2L que a no-R2L
- Ejemplo: R2L recibe probabilidades [0.25, 0.35, 0.40], no-R2L recibe [0.05, 0.10, 0.15]
- Orden correcto → AUC alto ✅

**F1 mide CLASIFICACIÓN BINARIA (threshold fijo 0.5):**
- Con threshold 0.5, instancias R2L con P < 0.5 se clasifican como 0
- En el ejemplo, NINGUNA R2L supera 0.5 → TP = 0 → Recall = 0 → F1 = 0

**Por qué sucede con R2L (0.83%):**

Modelo aprende distribución base:
- 99.17% son no-R2L → Predicciones "default" son bajas (P ≈ 0.1-0.3)
- Aunque R2L tenga features distintivos, su probabilidad queda < 0.5

**Solución:**

**Optimizar threshold:**
- En lugar de 0.5, usar threshold óptimo (ej: 0.15) que maximiza F1
- Precision-Recall curve ayuda a encontrarlo

**Ajustar según costos:**
- Si FN (no detectar R2L) cuesta $10,000 y FP (falsa alarma) cuesta $100
- Threshold óptimo será mucho más bajo (ej: 0.05), priorizando recall

---

## **NIVEL 5: APLICACIÓN ESPECÍFICA AL PROYECTO**

### **P3.18: ¿Por qué usas 8 componentes principales como features en lugar de las 32 variables originales?**

**R:** 

**Razón 1: Eliminar multicolinealidad**
- Variables originales: 14 pares con |r| > 0.8
- Consecuencia: Coeficientes inestables, varianza inflada
- PCs: Correlación = 0 (ortogonales) → Coeficientes estables

**Razón 2: Reducción dimensional**
- 32 variables → Mayor riesgo de overfitting
- 8 PCs (71.85% varianza) → Menor complejidad, misma información crítica
- Principio de parsimonia

**Razón 3: Filtrado de ruido**
- Los últimos 24 PCs (28.15% varianza) capturan ruido más que señal
- Al descartarlos, mejoramos relación señal/ruido

**Validación:**
- AUC con 8 PCs = 0.9496
- AUC con 13 PCs = 0.9541 (ganancia < 0.5%)
- AUC con 32 variables originales: No probamos, pero esperaríamos overfitting

**Trade-off:**
- **Pérdida:** Interpretabilidad directa (coeficiente de "duration" vs "PC3")
- **Ganancia:** Robustez, estabilidad, eficiencia

---

### **P3.19: ¿Cómo interpretas el coeficiente de PC1 en el modelo DoS?**

**R:** 

**Supuesto:** Modelo DoS tiene coeficiente $\beta_1 = 2.5$ para PC1.

**Paso 1: Interpretación en log-odds**

Por cada unidad que aumenta PC1 (manteniendo PC2-PC8 constantes):
$$\Delta \text{log-odds}(DoS) = 2.5$$

**Paso 2: Odds Ratio**
$$OR = e^{2.5} = 12.18$$

Por cada unidad que aumenta PC1, los **odds de ser DoS** se multiplican por 12.18.

**Paso 3: Conectar con variables originales**

Analizamos loadings de PC1:
- `same_srv_rate`: loading = 0.35 (alto)
- `dst_host_same_srv_rate`: loading = 0.34
- `dst_host_serror_rate`: loading = 0.32

**Interpretación integrada:**

PC1 captura un patrón donde:
- Alta tasa de conexiones al mismo servicio
- Alta tasa de errores SYN en el host destino

**Comportamiento DoS típico:**
- Bombardear el MISMO servicio (HTTP) en el MISMO host → PC1 alto
- Generar errores SYN masivos → PC1 alto

**Conclusión:** PC1 es un "detector de patrones de DoS" emergido naturalmente. Aumentos en PC1 predicen fuertemente presencia de DoS.

---

### **P3.20: ¿Por qué DoS tiene AUC 0.9902 mientras U2R tiene 0.8619?**

**R:** 

**Factores que explican la diferencia:**

**1. Tamaño muestral:**
- DoS: 9,234 instancias (36.7%)
- U2R: 11 instancias (0.04%)
- **Ratio:** 839:1

**Consecuencia:**
- Modelo de DoS entrenó con ~9,000 ejemplos → Aprende patrones robustos
- Modelo de U2R entrenó con 11 ejemplos → Varianza alta, posible memorización

**2. Separabilidad intrínseca:**

**DoS tiene "firma" extremadamente clara:**
- `serror_rate` = 1.0 (100% errores SYN) vs 0.0 en Normal
- `count` = 177 vs 4 en Normal
- Diferencias son **dramáticas y consistentes**

**U2R tiene "firma" más sutil:**
- `dst_bytes` = 3,860 vs Normal variable
- `duration` = 53 seg vs Normal variable
- Diferencias existen pero son **menos extremas y más variables**

**3. Variabilidad intra-clase:**

**DoS es homogéneo:** Casi todos los ataques DoS siguen patrón SYN flood.

**U2R es heterogéneo:** Incluye buffer overflow, rootkit, loadmodule → Comportamientos diversos.

**Conclusión:**

AUC 0.8619 para U2R es **notable** dado:
- Solo 11 instancias de entrenamiento
- Desbalance extremo (1:2289)
- Heterogeneidad de comportamientos

**Con SMOTE (oversampling sintético) o más datos, U2R mejoraría significativamente.**

---

### **P3.21: ¿Cómo conectas los resultados de P3 con P1 y P2?**

**R:** 

**Flujo de coherencia metodológica:**

**P1 (Kruskal-Wallis):**
- Identificamos variables con **alto poder discriminante**: ε² > 0.5
- Variables clave: `serror_rate`, `dst_bytes`, `src_bytes`, `count`

**P2 (PCA):**
- Esas variables discriminantes tienen **loadings altos** en los primeros PCs
- PC1 dominado por `same_srv_rate`, `dst_host_serror_rate` → Captura patrones de DoS
- PC2 captura patrones de Probe (loadings altos en `rerror_rate`)

**P3 (Regresión Logística):**
- Modelos usan esos PCs como features
- **DoS:** Coeficiente alto en PC1 → Explota la firma de DoS capturada en P2
- **Probe:** Coeficiente alto en PC2 → Explota la firma de Probe

**Validación circular:**

```
Variables discriminantes (P1 ε² alto)
         ↓
Capturadas en PCs (P2 loadings altos)
         ↓
Usadas en clasificación (P3 coeficientes altos)
         ↓
Alta precisión (AUC 0.95)
         ↓
CONFIRMA que variables de P1 eran realmente discriminantes
```

**Si hubiera inconsistencia:**

- Variables con ε² bajo en P1 pero coeficientes altos en P3 → Contradicción
- AUC bajo en P3 a pesar de ε² alto en P1 → P2 perdió información crítica

**En nuestro proyecto:** Coherencia perfecta → Metodología validada.

---

# 🎓 RESUMEN FINAL: CONCEPTOS CLAVE PARA DOMINAR

## **Kruskal-Wallis:**
- Alternativa no paramétrica a ANOVA
- Usa **rangos**, no valores originales → Robusto a outliers
- H ~ χ²_(k-1) bajo H₀
- ε² mide tamaño del efecto (>0.14 = grande)
- Test de Dunn + Bonferroni para comparaciones por pares
- Compara **distribuciones completas**, no solo medianas

## **PCA:**
- Reduce dimensionalidad transformando a componentes ortogonales
- Basado en **eigenvectors/eigenvalues** de matriz de covarianza
- **Estandarización es crítica** (media=0, std=1)
- Scree plot identifica "codo" (rendimientos decrecientes)
- **Loadings** revelan contribución de variables originales
- Silhouette bajo en 2D NO invalida PCA si AUC es alto en kD
- **Ortogonalidad** elimina multicolinealidad

## **Regresión Logística:**
- Modelo de **clasificación** (no regresión pese al nombre)
- Predice **probabilidades** vía función sigmoide: $P = \frac{1}{1+e^{-z}}$
- Coeficientes interpretan como **log-odds**: OR = $e^{\beta}$
- Estimación via **MLE** (sin solución cerrada → métodos iterativos)
- **Regularización L2** previene overfitting
- **OvR (One-vs-Rest):** k modelos binarios independientes
- **class_weight='balanced'** maneja desbalance
- **AUC-ROC:** Mide discriminación robusta al desbalance
- **Paradoja AUC alto + F1 bajo:** Threshold 0.5 inapropiado para clases minoritarias

---

# 🔗 CONEXIÓN METODOLÓGICA FINAL

```
EDA (Sección 2)
   ↓
Detecta asimetría, multicolinealidad, outliers
   ↓
P1: Kruskal-Wallis (Sección 4.1)
   ↓
Identifica variables discriminantes (ε² > 0.5)
   ↓
P2: PCA (Sección 4.2)
   ↓
Comprime 32 variables → 8 PCs (elimina multicolinealidad)
   ↓
P3: Regresión Logística (Sección 4.3)
   ↓
Clasifica usando 8 PCs → AUC 0.95
   ↓
CONCLUSIÓN: Detección de intrusiones sin firmas es viable
```

**Esta cadena de evidencia valida el enfoque completo del proyecto.**

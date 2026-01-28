# 🎤 MONÓLOGO PARA EXPOSICIÓN (10-12 MINUTOS)

**Timing total:** 11 minutos | **Tono:** Profesional-académico, seguro, didáctico

---

## **[SLIDE 1 - PORTADA]** (30 seg)

*[Contacto visual con la audiencia, respirar profundo]*

Buenos días/tardes. Mi nombre es [TU NOMBRE] y hoy presentaré mi proyecto de investigación titulado **"Detección de Intrusiones y Análisis de Anomalías en Tráfico de Red mediante Técnicas Estadísticas"**, realizado como trabajo final del curso de Estadística en MATCOM, Universidad de La Habana.

*[Pausa breve]*

La pregunta central que responderemos hoy es: **¿Podemos detectar ataques de red sin conocerlos previamente?** Es decir, ¿pueden las técnicas estadísticas identificar comportamientos maliciosos basándose únicamente en patrones anómalos del tráfico, sin depender de catálogos de firmas?

*[Avanzar slide]*

---

## **[SLIDE 2 - CONTEXTO Y MOTIVACIÓN]** (1 min 30 seg)

Para entender por qué este enfoque es relevante, primero veamos cómo funcionan los **Sistemas de Detección de Intrusiones tradicionales**.

Los IDS convencionales operan mediante **firmas conocidas** – es como tener una base de datos de "huellas digitales" de ataques. Funcionan excelente contra amenazas catalogadas, **pero tienen un problema fundamental:** son **completamente ciegos ante ataques nuevos** – los llamados "zero-day attacks".

*[Señalar segunda columna]*

Además, mantener estas bases de datos actualizadas es **costoso** y requiere actualización constante.

*[Pausa dramática]*

**Nuestra propuesta** toma un camino diferente. En lugar de buscar firmas específicas, buscamos **anomalías estadísticas** en el comportamiento del tráfico de red.

*[Señalar cuadro comparativo]*

Observen esta comparación: mientras que la detección por firmas **falla completamente** ante ataques nuevos o variantes, nuestro enfoque basado en anomalías los **detecta porque su comportamiento estadístico difiere del tráfico normal**, sin importar si los conocemos o no.

La ventaja clave es la **generalización**: aprendemos qué es "normal" y detectamos cualquier cosa que se desvíe significativamente.

*[Avanzar]*

---

## **[SLIDE 3 - DATASET NSL-KDD]** (1 min)

Para validar este enfoque, utilizamos el **dataset NSL-KDD**, que es el benchmark académico estándar en investigación de detección de intrusiones.

NSL-KDD es una **versión mejorada** del clásico KDD Cup 1999. Los investigadores del Canadian Institute for Cybersecurity eliminaron el 78% de registros redundantes que contaminaban el dataset original, haciéndolo mucho más apropiado para entrenar modelos sin sobreajuste.

Trabajamos con:
- **25,192 observaciones de entrenamiento** (el subset estratificado del 20%)
- **22,544 observaciones de prueba**
- **41 variables** que describen el tráfico de red
- **5 categorías:** Tráfico Normal (53.4%), ataques de Denegación de Servicio o DoS (36.7%), Escaneo o Probe (9.1%), Acceso No Autorizado o R2L (0.8%), y Escalada de Privilegios o U2R (0.04%)

*[Señalar gráfico de barras]*

Noten el **desbalance severo**: U2R apenas representa 11 instancias en train, lo cual será un desafío importante que veremos más adelante.

*[Avanzar]*

---

## **[SLIDE 4 - VISUALIZACIONES]** (30 seg)

*[NOTA: Si esta slide tiene visualizaciones, describir brevemente. Si solo tiene título truncado, ser breve]*

Antes de aplicar técnicas avanzadas, realizamos un **análisis exploratorio exhaustivo** para entender la estructura de los datos.

Aquí vemos algunas distribuciones y patrones de outliers que nos ayudaron a formular nuestras hipótesis.

*[Avanzar rápidamente - no detenerse mucho aquí]*

---

## **[SLIDE 5 - EDA: HALLAZGOS CLAVE]** (1 min 30 seg)

El análisis exploratorio reveló tres hallazgos críticos que guiaron nuestro enfoque metodológico:

**Primero, calidad de datos:** Excelente noticia – **cero valores faltantes**. El dataset está completo. Sin embargo, detectamos que el **68.8% de las variables presentan asimetría extrema**, lo que viola los supuestos de normalidad. Esto nos obligará a usar **pruebas no paramétricas** en lugar de ANOVA.

**Segundo, outliers:** Detectamos que el 18.6% de observaciones en la variable `dst_bytes` son outliers según el criterio IQR de Tukey. *[Pausa]* Pero aquí viene lo interesante: **NO son ruido**. Al analizar su distribución por categoría, descubrimos que están **concentrados en ataques**, no distribuidos aleatoriamente. Por ejemplo, en U2R, el 81.82% de instancias son outliers, versus solo 32.91% en tráfico Normal. Es decir, **los outliers son la señal que buscamos, no ruido a eliminar**.

**Tercero, multicolinealidad masiva:** Identificamos **14 pares de variables con correlación mayor a 0.8**. Variables como `dst_host_srv_count` y `srv_count` miden esencialmente lo mismo. Esta redundancia motiva la necesidad de **reducción dimensional** que veremos en la Pregunta 2.

*[Avanzar]*

---

## **[SLIDE 6 - PREGUNTA 1: FIRMAS ESTADÍSTICAS]** (1 min 45 seg)

Ahora sí, entramos en nuestras **tres preguntas de investigación**.

**Pregunta 1:** *¿Tienen los ataques "firmas" estadísticas distinguibles?*

Para responderla, seleccionamos 5 variables clave del tráfico – `src_bytes`, `dst_bytes`, `duration`, `serror_rate` y `count` – y aplicamos el **Test de Kruskal-Wallis**, que es la alternativa no paramétrica a ANOVA para comparar distribuciones entre múltiples grupos independientes.

Planteamos:
- **H₀:** Las distribuciones son idénticas entre las 5 categorías
- **H₁:** Al menos una categoría tiene distribución diferente

*[Pausa dramática]*

Los resultados son contundentes:

**Todas las variables** tienen **p-valor menor a 1×10⁻¹⁰** – es decir, prácticamente cero. La probabilidad de que estas diferencias sean casualidad es astronómicamente baja.

Además, el **tamaño del efecto Epsilon-squared es mayor a 0.49** en todas, clasificándose como **efecto GRANDE** según los criterios de Cohen. Esto significa que las diferencias no solo son estadísticamente significativas, sino **prácticamente sustanciales**.

*[Señalar radar plot]*

Este radar plot visualiza los **perfiles estadísticos** de cada categoría usando las medianas normalizadas. Observen:

- **DoS** (en rojo) tiene un pico extremo en `serror_rate` y `count` – esto es característico de ataques SYN flood masivos
- **U2R** (gris oscuro) destaca en `dst_bytes` y `duration` – sesiones largas descargando exploits
- **Normal** (verde) tiene un perfil balanceado sin extremos

Cada polígono es claramente distinguible. **Conclusión de P1:** Sí, cada tipo de ataque tiene una "huella digital" estadística robusta.

*[Avanzar]*

---

## **[SLIDE 7 - PATRONES ESPECÍFICOS]** (1 min)

Esta slide detalla las **firmas específicas** que identificamos:

**DoS (Denegación de Servicio):**
- `src_bytes` y `dst_bytes` = 0 → No hay transferencia de datos
- `serror_rate` = 1.0 → 100% errores SYN
- `count` = 177 conexiones → Ráfaga masiva

Esto corresponde a un **SYN flood clásico**: el atacante envía miles de solicitudes SYN incompletas para colapsar el servidor.

**Probe (Escaneo de puertos):**
- `src_bytes` ≈ 1 → Paquetes mínimos de sondeo
- `dst_bytes` = 0 → No esperan respuesta completa
- Escanean rápidamente múltiples puertos buscando vulnerabilidades

**U2R (Escalada de privilegios):**
- `dst_bytes` = 3,860 → Descarga de exploits o shells
- `duration` = 53 segundos → Sesiones persistentes

**Normal:**
- Valores intermedios con **alta variabilidad** – el uso legítimo es naturalmente diverso

Estos patrones confirman que **el comportamiento anómalo deja huellas medibles**.

*[Avanzar]*

---

## **[SLIDE 8 - PREGUNTA 2: COMPRESIÓN DIMENSIONAL]** (1 min 45 seg)

**Pregunta 2:** *¿Podemos reducir las 32 variables sin perder información discriminante?*

Dado que detectamos multicolinealidad severa en el EDA, aplicamos **Análisis de Componentes Principales (PCA)** sobre las variables numéricas estandarizadas.

*[Señalar scree plot]*

El **scree plot** muestra la varianza explicada por cada componente. Buscamos el "codo" – el punto donde los rendimientos decrecen abruptamente. Esto ocurre alrededor de los **5-6 componentes**.

Sin embargo, mediante **validación cruzada**, determinamos que **8 componentes principales** ofrecen el mejor balance:
- Capturan el **71.85% de la varianza total**
- Logran una **reducción del 75%** en dimensionalidad (de 32 a 8 variables)
- Tienen **desempeño predictivo casi idéntico** a usar 13 PCs, pero con 38% menos features

*[Señalar proyección 3D]*

Esta proyección 3D visualiza los primeros 3 PCs. Aunque vemos cierta separación, noten que el **Silhouette Score en 2D es solo 0.13** – bastante bajo.

*[Pausa para énfasis]*

¿Esto significa que PCA falló? **No**. Al contrario, confirma que la separabilidad **NO está en proyecciones visuales simples**, sino en las **8 dimensiones combinadas** del espacio PCA. De hecho, como veremos en la siguiente pregunta, el AUC de 0.95 en 8D valida que capturamos la información discriminante esencial.

*[Avanzar]*

---

## **[SLIDE 9 - PREGUNTA 3: MODELOS PREDICTIVOS]** (1 min 30 seg)

**Pregunta 3:** *¿Podemos predecir tipos específicos de ataque usando componentes principales?*

Entrenamos **4 modelos de Regresión Logística binaria** con estrategia One-vs-Rest: cada modelo predice un tipo de ataque versus el resto.

Las features son los **8 componentes principales** obtenidos en P2.

*[Pausa]*

Los resultados son excelentes:
- **AUC promedio: 0.9496** – clasificación como "excelente" según estándares académicos

Detallando por categoría:
- **DoS: AUC 0.9902** – prácticamente perfecto
- **Probe: AUC 0.9868** – también excelente
- **R2L: AUC 0.9593** – muy bueno, considerando el desbalance
- **U2R: AUC 0.8619** – bueno, *notable dada la escasez extrema de datos* (solo 11 instancias)

*[Señalar curvas ROC]*

Estas curvas ROC visualizan el trade-off entre sensibilidad y especificidad. Todas están muy por encima de la línea diagonal (clasificador aleatorio), confirmando poder discriminante robusto.

Además, validamos que **8 PCs vs 13 PCs** tienen desempeño casi idéntico (diferencia <0.5%), confirmando nuestra decisión de parsimonia.

*[Avanzar]*

---

## **[SLIDE 10 - DESEMPEÑO DETALLADO]** (1 min)

*[Señalar tabla si es visible, sino referirse a texto]*

Analizando métricas complementarias:

**Listos para producción:**
- DoS y Probe tienen **F1-Score mayor a 0.79**, indicando excelente balance entre precisión y recall. Estos modelos son **deployables** con mínimo ajuste.

**Requieren ajustes:**
- R2L tiene AUC excelente (0.96) pero **F1 bajo (0.15)**. Esto es la **paradoja del desbalance**: el modelo discrimina bien (AUC alto), pero el threshold por defecto de 0.5 es inadecuado cuando la clase minoritaria es 0.8%. Ajustando el threshold según el costo de falsos positivos/negativos, el F1 mejoraría significativamente.

**Necesitan mejoras:**
- U2R con solo 11 instancias (0.04%) es el caso extremo. Incluso alcanzar AUC 0.86 con tan pocos ejemplos es notable. Técnicas de **oversampling como SMOTE** o simplemente conseguir más datos mejorarían sustancialmente la detección.

*[Avanzar]*

---

## **[SLIDE 11 - COHERENCIA METODOLÓGICA]** (1 min)

Esta es una de mis slides favoritas porque muestra la **coherencia metodológica** del estudio completo.

*[Señalar diagrama de flujo]*

Observen cómo cada pregunta alimenta la siguiente:

**P1 - Kruskal-Wallis:**
- *Objetivo:* Identificar variables discriminantes
- *Resultado:* ε² > 0.5 en todas las variables clave
- *Salida:* 5 variables con alta separabilidad

**P2 - PCA:**
- *Objetivo:* Capturar esa información en componentes ortogonales
- *Resultado:* 8 PCs capturan 71.85% varianza
- *Salida:* Features comprimidas sin pérdida crítica

**P3 - Regresión Logística:**
- *Objetivo:* Explotar esos componentes para clasificación
- *Resultado:* AUC = 0.9496 promedio
- *Salida:* Modelo predictivo viable y parsimonioso

*[Pausa]*

Las variables que mostraron mayor Epsilon-squared en P1 son precisamente las que tienen **loadings altos** en los componentes de P2, y esos componentes tienen **coeficientes altos** en los modelos de P3.

Es un flujo metodológico perfectamente coherente de hipótesis a predicción.

*[Avanzar]*

---

## **[SLIDE 12 - CONCLUSIONES]** (1 min 30 seg)

Resumiendo, este estudio valida tres conclusiones fundamentales:

**CONCLUSIÓN 1: Huellas Digitales Estadísticas Robustas**

Los ataques de red **SÍ** presentan firmas estadísticas distinguibles. Con p-valores menores a 1×10⁻¹⁰ y tamaños de efecto ε² > 0.49, cada tipo de ataque tiene un perfil característico identificable.

**CONCLUSIÓN 2: Compresión Dimensional Efectiva**

PCA logra reducir 75% la dimensionalidad preservando 71.85% de varianza sin pérdida de información crítica para clasificación. La parsimonia es **viable y beneficiosa**.

**CONCLUSIÓN 3: Modelos Predictivos Viables**

Regresión Logística con 8 PCs alcanza AUC = 0.9496, con DoS y Probe listos para producción, y R2L/U2R mejorables mediante técnicas estándar de balanceo.

*[Pausa dramática, contacto visual con audiencia]*

**Conclusión final:**

El **análisis estadístico del comportamiento del tráfico de red** es un enfoque **complementario viable** a los IDS tradicionales. No lo reemplaza – los sistemas de firmas siguen siendo excelentes para amenazas conocidas – pero **complementa** su capacidad de detección identificando anomalías estadísticas sin requerir conocimiento previo de ataques específicos.

Este estudio demuestra que **"conocer lo normal"** puede ser tan valioso como **"conocer lo malicioso"**.

*[Avanzar]*

---

## **[SLIDE 13 - PREGUNTAS]** (30 seg)

Gracias por su atención. Quedo a disposición para responder cualquier pregunta sobre la metodología, resultados o implicaciones del estudio.

*[Sonreír, contacto visual, brazos abiertos – postura receptiva]*

---

# 🎯 CONSEJOS DE DELIVERY

## **Timing por Sección:**
- Slides 1-3 (Intro + Dataset): 3 min
- Slides 4-5 (EDA): 2 min
- Slides 6-7 (P1): 2 min 45 seg
- Slide 8 (P2): 1 min 45 seg
- Slides 9-10 (P3): 2 min 30 seg
- Slides 11-12 (Coherencia + Conclusiones): 2 min 30 seg
- Slide 13 (Cierre): 30 seg
**TOTAL: 11 min 30 seg** → Perfecto para 10-12 min con margen

## **Lenguaje Corporal:**
- **Contacto visual:** 70% del tiempo distribuido equitativamente
- **Gestos:** Señalar slides cuando digas "observen", "aquí vemos"
- **Pausas dramáticas:** Antes de resultados clave (p-valores, AUC)
- **Movimiento:** No estático, pero tampoco inquieto
- **Postura:** Abierta, confiada, no cruzar brazos

## **Tono de Voz:**
- **Velocidad:** 130-150 palabras/min (ni lento ni acelerado)
- **Volumen:** Proyectar sin gritar
- **Énfasis:** En palabras clave (cero valores faltantes, p<10⁻¹⁰, AUC 0.95)
- **Variación:** No monotonía – modular para mantener atención

## **Manejo de Preguntas:**
- **Escuchar completamente** antes de responder
- **Reformular** si no entendiste: "¿Me pregunta si...?"
- **No bluffear:** Si no sabes, admitir honestamente
- **Conectar con el trabajo:** "Interesante pregunta, justo en la Slide X vimos..."

---

# 📝 CHECKLIST PRE-EXPOSICIÓN

- [ ] **Practicar 3 veces completas** con cronómetro
- [ ] **Corregir Slide 4** (título truncado)
- [ ] **Corregir Slide 8** (inconsistencia 5 vs 8 PCs)
- [ ] **Verificar Slide 10** (tabla legible)
- [ ] **Imprimir notas** en tarjetas pequeñas (bullet points clave)
- [ ] **Hidratar** antes (agua, no café en exceso)
- [ ] **Llegar 10 min antes** para probar proyector/laptop
- [ ] **Respirar profundo** antes de empezar

---

¡Éxito en tu exposición! 🚀

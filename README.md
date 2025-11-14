# 🏥 Desafío de Machine Learning en Economía de la Salud

**Universidad Nacional del Oeste (UNO) - 2025**  
**Materia:** Aplicaciones en Ciencia de Datos  
**Modalidad:** Trabajo en grupos de 3 personas

---

## 📋 Índice

1. [Descripción del Desafío](#descripción-del-desafío)
2. [Objetivo](#objetivo)
3. [El Dilema Estratégico: COVID](#el-dilema-estratégico-covid)
4. [Datos Proporcionados](#datos-proporcionados)
5. [Estructura del Proyecto](#estructura-del-proyecto)
6. [Instalación y Setup](#instalación-y-setup)
7. [Cómo Ejecutar el Pipeline](#cómo-ejecutar-el-pipeline)
8. [Qué Deben Entregar](#qué-deben-entregar)
9. [Criterios de Evaluación](#criterios-de-evaluación)
10. [Recursos y Documentación](#recursos-y-documentación)
11. [FAQ](#faq)

---

## 🎯 Descripción del Desafío

En este desafío aplicarán técnicas de Machine Learning para predecir el **gasto de bolsillo (Out-of-Pocket) per cápita en PPP** que los ciudadanos realizarán en salud durante el año **2022**, utilizando datos históricos de ~78 países del período 2000-2021.

El desafío combina:
- 📊 **Análisis de datos** de economía de la salud
- 🤖 **Machine Learning** con Gradient Boosting (LightGBM)
- 💡 **Feature Engineering creativo** basado en teoría económica
- ⚖️ **Decisiones estratégicas** sobre uso de datos COVID
- 📝 **Interpretación económica** de resultados

---

## 🎓 Objetivo

Desarrollar un modelo predictivo que:

1. **Prediga con la mayor precisión posible** el gasto de bolsillo en salud para 2022
2. **Incorpore variables económicamente significativas** que ustedes creen (feature engineering)
3. **Tome decisiones fundamentadas** sobre el uso de datos de los años COVID (2020-2021)
4. **Interprete los resultados** conectándolos con teoría de economía de la salud

**Métrica de Performance:** RMSE (Root Mean Squared Error) sobre conjunto de test

---

## ⚠️ El Dilema Estratégico: COVID

### El Problema

Los datos incluyen los años 2020-2021 (período COVID-19), que representan un **shock exógeno** sin precedentes en el sistema de salud mundial. Ustedes deben decidir:

**¿Usar datos COVID o descartarlos?**

### Las Opciones

Tienen dos parámetros clave en `0_HEALTH_YML.yml` que determinan su estrategia:

1. **`presente`**: ¿Cuál es el último año CON DATOS que usan?
   - `2021` = Incluyen datos hasta 2021 (con COVID completo)
   - `2020` = Incluyen datos hasta 2020 (con COVID parcial)
   - `2019` = NO incluyen ningún dato COVID
   - `2018` = Excluyen datos recientes

2. **`orden_lead`**: ¿Cuántos años hacia el futuro predicen?
   - `1` = Predecir 1 año adelante
   - `2` = Predecir 2 años adelante
   - `3` = Predecir 3 años adelante
   - `4` = Predecir 4 años adelante

### Ejemplos de Configuraciones

| Estrategia | presente | orden_lead | Train hasta | Predice | Datos COVID |
|------------|----------|------------|-------------|---------|-------------|
| **Maximalista** | 2021 | 1 | 2020 | 2022 | ✅ USA 2020-2021 |
| **Conservadora** | 2019 | 3 | 2016 | 2022 | ❌ Descarta 2020-2021 |
| **Intermedia** | 2020 | 2 | 2018 | 2022 | ⚠️ USA 2020, descarta 2021 |
| **Prudente** | 2018 | 4 | 2014 | 2022 | ❌ Descarta 2019-2021 |

### Trade-offs

#### Estrategia Maximalista (usar COVID)
**✅ Ventajas:**
- Más datos para entrenar
- Información más reciente
- Captura tendencias actuales

**❌ Desventajas:**
- COVID puede distorsionar relaciones entre variables
- Shock puede no ser representativo de 2022
- Riesgo de overfitting a datos atípicos

#### Estrategia Conservadora (descartar COVID)
**✅ Ventajas:**
- Datos más "estables" sin shocks
- Relaciones entre variables más predecibles
- Menor riesgo de aprender patrones no generalizables

**❌ Desventajas:**
- Menos datos para entrenar
- Información menos reciente
- Puede perder señales importantes de cambios estructurales

### 📌 Decisión Complementaria: excluir años

Además de `presente` y `orden_lead`, pueden configurar `excluir` en la sección `training_strategy` del YML para **eliminar años específicos del entrenamiento** (pero no del dataset).

Ejemplo:
```yaml
train:
  excluir: [2020, 2021]  # Excluye COVID del entrenamiento
```

**Esta decisión vale 15% de la nota final** y debe ser documentada en el informe.

---

## 📦 Datos Proporcionados

### Dataset Principal: `dataset_desafio.csv`

- **Países:** ~78 países válidos (excluidos 12 problemáticos)
- **Período:** 2000-2021 (22 años)
- **Variables:** ~200 indicadores del World Bank (WDI)
- **Target:** `hf3_ppp_pc` (gasto de bolsillo PPP per cápita)
- **Estructura:** Panel data (Country Code, year, region, income, variables...)

### Variables Importantes Incluidas

**Economía:**
- `NY.GDP.PCAP.PP.CD` - PIB per cápita PPP
- `NY.GDP.MKTP.KD.ZG` - Crecimiento del PIB
- Inflación, desempleo, comercio, etc.

**Salud:**
- `SP.DYN.LE00.IN` - Expectativa de vida al nacer
- `SH.XPD.CHEX.GD.ZS` - Gasto en salud como % del PIB
- `SH.XPD.CHEX.PC.CD` - Gasto en salud per cápita
- `SP.DYN.IMRT.IN` - Mortalidad infantil
- `SH.MED.BEDS.ZS` - Camas de hospital por 1000 habitantes

**Demográficas:**
- `SP.POP.TOTL` - Población total
- `SP.POP.65UP.TO.ZS` - % Población mayor de 65 años
- `SP.URB.TOTL.IN.ZS` - % Población urbana

**Metadatos:**
- `region` - Región WHO (AFR, AMR, EMR, EUR, SEAR, WPR)
- `income` - Nivel de ingreso (Low, Lower-middle, Upper-middle, High)

### ⚠️ Importante: Target Ausente para 2022

El dataset NO incluye valores de `hf3_ppp_pc` para 2022. Ese es el valor que deben predecir.

### Diccionario de Variables

Ver `dataset/diccionario_variables.md` para descripciones detalladas de cada variable.

---

## 📁 Estructura del Proyecto

```
health_economics_challenge/
├── README.md                              # Este archivo
├── 00_CONTEXTO_CONTINUIDAD_PROYECTO.md   # Contexto técnico (referencia)
│
├── dataset/
│   ├── dataset_desafio.csv                # Dataset limpio para ustedes
│   ├── diccionario_variables.md           # Descripción de variables
│   └── metadata_paises.csv                # Info de países
│
├── codigo_base/
│   ├── 0_HEALTH_YML.yml                   # ⚙️ CONFIGURACIÓN (deben modificar)
│   ├── 0_HEALTH_EXE.R                     # Script ejecutor principal
│   ├── 01_FE_health_ALUMNO.R              # 📝 FEATURE ENGINEERING (deben completar)
│   ├── 02_TS_health.R                     # Training Strategy
│   ├── 03_HT_health.R                     # Hyperparameter Tuning
│   └── 04_ZZ_health.R                     # Predicción final (TODO)
│
├── documentacion/
│   ├── 01_guia_instalacion.md             # Setup R, librerías
│   ├── 02_guia_estrategia_covid.md        # Análisis del dilema COVID
│   ├── 03_guia_feature_engineering.md     # Hints para crear variables
│   ├── 04_guia_interpretacion.md          # Cómo interpretar importancia
│   └── 05_FAQ_tecnico.md                  # Problemas comunes
│
├── evaluacion/
│   ├── rubrica_evaluacion.md              # Criterios de evaluación
│   ├── checklist_entrega.md               # Qué entregar
│   └── ejemplos_analisis.md               # Ejemplos de buen análisis
│
└── exp/                                    # Aquí se guardan resultados (se crea automáticamente)
```

---

## 🔧 Instalación y Setup

### Paso 1: Instalar R y RStudio

Ver `documentacion/01_guia_instalacion.md` para instrucciones detalladas.

### Paso 2: Instalar Librerías Necesarias

```r
# Copiar y ejecutar en R:
install.packages(c(
  "data.table",      # Manipulación eficiente de datos
  "lightgbm",        # Gradient Boosting
  "yaml",            # Lectura de configuración
  "mlrMBO",          # Optimización bayesiana
  "DiceKriging",     # Soporte para mlrMBO
  "rlist",           # Utilidades para listas
  "lubridate",       # Manejo de fechas
  "primes"           # Números primos (para canaritos)
))
```

### Paso 3: Ajustar Path del Proyecto

Editar `0_HEALTH_YML.yml` línea 1:
```yaml
environment:
  base_dir: "C:/RUTA/A/TU/CARPETA/health_economics_challenge"  # ← Cambiar esta ruta
```

---

## ▶️ Cómo Ejecutar el Pipeline

### Paso 1: Configurar Estrategia (YML)

Editar `codigo_base/0_HEALTH_YML.yml`:

```yaml
feature_engineering:
  const:
    orden_lead: 1      # ← COMPLETAR: 1, 2, 3, o 4
    presente: 2021     # ← COMPLETAR: 2018, 2019, 2020, o 2021

training_strategy:
  param:
    train:
      excluir: []      # ← COMPLETAR: [] o [2020, 2021] u otra combinación
```

### Paso 2: Crear Variables (Feature Engineering)

Editar `codigo_base/01_FE_health_ALUMNO.R`:

Completar la función `AgregarVariables()`:

```r
AgregarVariables <- function(dataset) {
  gc()
  
  # ========================================
  # AQUÍ CREAN SUS VARIABLES
  # ========================================
  
  # Ejemplo: Ratio de eficiencia en salud
  dataset[, eficiencia_salud := SP.DYN.LE00.IN / SH.XPD.CHEX.PC.CD]
  
  # Ejemplo: Dummy para crisis económica 2008
  dataset[, crisis_2008 := ifelse(year %in% 2008:2009, 1, 0)]
  
  # ... MÁS VARIABLES CREADAS POR USTEDES ...
  
  # ========================================
  # LÓGICA DE SEGURIDAD (NO MODIFICAR)
  # ========================================
  
  # [Código de seguridad ya incluido en el archivo]
  
  return(dataset)
}
```

### Paso 3: Ejecutar Pipeline Completo

En RStudio, abrir y ejecutar:

```r
source("codigo_base/0_HEALTH_EXE.R")
```

**Tiempo estimado:** 30-60 minutos (depende del hardware)

### Paso 4: Analizar Resultados

Los resultados se guardan en:
```
exp/[nombre_experimento]/
├── 01_FE/                    # Dataset con feature engineering
├── 02_TS/                    # Datos train/validate/test
└── 03_HT/
    ├── modelo_final_lgb.rds         # Modelo entrenado
    ├── tb_importancia.txt            # ⭐ Importancia de variables
    ├── BO_log.txt                    # Log de optimización
    └── predicciones_presente.csv     # ⭐ Predicciones para 2022
```

**Archivos clave para el informe:**
- `tb_importancia.txt` - Ver qué variables son más importantes
- `predicciones_presente.csv` - Sus predicciones finales
- `BO_log.txt` - RMSE del mejor modelo

---

## 📤 Qué Deben Entregar

### 1. Código (30%)

- `01_FE_health_ALUMNO.R` con función `AgregarVariables()` completa
- `0_HEALTH_YML.yml` con configuración elegida
- Comentarios explicando razonamiento económico de variables

### 2. Predicciones (15%)

- `predicciones_2022.csv` - Predicciones finales para cada país

### 3. Análisis de Importancia (25%)

- Top 20 variables más importantes (desde `tb_importancia.txt`)
- Interpretación económica: ¿por qué esas variables predicen mejor?
- Análisis de sus variables creadas: ¿aparecen en el top? ¿por qué?

### 4. Informe Ejecutivo (30%)

Documento estructurado con:

#### 1. Decisión de Estrategia (15% de nota total)
- Configuración elegida (presente, orden_lead, excluir)
- Justificación del trade-off
- Análisis de sensibilidad (opcional pero valorado)

#### 2. Feature Engineering (20%)
- Descripción de variables creadas
- Justificación teórica económica
- Impacto en performance

#### 3. Resultados y Performance (50%)
- Métricas del modelo (RMSE)
- Análisis de importancia de variables
- Interpretación económica

#### 4. Conclusiones (15%)
- Insights principales
- Limitaciones del modelo
- Recomendaciones futuras

**Extensión sugerida:** 8-12 páginas

Ver `evaluacion/checklist_entrega.md` para detalles completos.

---

## 📊 Criterios de Evaluación

| Dimensión | Peso | Descripción |
|-----------|------|-------------|
| **Decisión Estratégica COVID** | 15% | Justificación documentada de presente/orden_lead/excluir |
| **Feature Engineering - Originalidad** | 15% | Variables novedosas y bien fundamentadas |
| **Feature Engineering - Teoría** | 15% | Conexión con economía de la salud |
| **Feature Engineering - Impacto** | 10% | Mejora en RMSE vs baseline |
| **Análisis de Importancia** | 15% | Interpretación económica de variables top |
| **Código y Documentación** | 10% | Claridad, comentarios, reproducibilidad |
| **Predicciones** | 10% | Calidad y completitud |
| **Informe Ejecutivo** | 10% | Estructura, claridad, profundidad |

Ver `evaluacion/rubrica_evaluacion.md` para criterios detallados.

---

## 📚 Recursos y Documentación

### Guías Técnicas

1. **[Guía de Instalación](documentacion/01_guia_instalacion.md)**  
   Setup completo de R, librerías y estructura de proyecto

2. **[Guía del Dilema COVID](documentacion/02_guia_estrategia_covid.md)**  
   Análisis profundo de trade-offs y configuraciones ejemplo

3. **[Guía de Feature Engineering](documentacion/03_guia_feature_engineering.md)**  
   Hints conceptuales sobre economía de la salud (sin dar soluciones)

4. **[Guía de Interpretación](documentacion/04_guia_interpretacion.md)**  
   Cómo leer `tb_importancia.txt` y conectar con teoría económica

5. **[FAQ Técnico](documentacion/05_FAQ_tecnico.md)**  
   Soluciones a problemas comunes de ejecución

### Referencias de Economía de la Salud

- **Out-of-Pocket Payments:** Gastos directos de los ciudadanos en servicios de salud no cubiertos por seguros o sistemas públicos
- **PPP (Purchasing Power Parity):** Ajuste por poder adquisitivo para comparar entre países
- **Universal Health Coverage (UHC):** Meta de OMS de reducir OOP al <20% del gasto total en salud

### Lectura Recomendada (Opcional)

- WHO Global Health Expenditure Database
- World Bank WDI Documentation
- Artículos sobre financial protection en salud

---

## ❓ FAQ

### ¿Puedo usar librerías adicionales de R?
Sí, pero deben documentar qué instalar en su README de entrega.

### ¿Cómo sé si mi configuración es buena?
Comparen el RMSE en validación. Menor RMSE = mejor modelo.

### ¿Puedo modificar los scripts 02_TS, 03_HT?
NO. Solo deben modificar `01_FE_health_ALUMNO.R` y `0_HEALTH_YML.yml`.

### ¿Qué pasa si el pipeline falla?
Consulten `documentacion/05_FAQ_tecnico.md`. Si el problema persiste, consulten al docente.

### ¿Cuántas configuraciones debo probar?
Mínimo 2 (una con COVID, una sin COVID) para comparar. Más configuraciones = mejor análisis.

### ¿Las variables creadas deben ser complejas?
No necesariamente. Una variable simple pero bien fundamentada económicamente vale más que una compleja sin sentido.

### ¿Cómo cito el dataset?
```
World Bank. (2024). World Development Indicators. 
https://databank.worldbank.org/source/world-development-indicators
```

---

## 📞 Contacto y Soporte

**Docente:** Francisco Fernández  
**Institución:** Universidad Nacional del Oeste (UNO)  
**Año:** 2025

Para consultas técnicas:
1. Revisar `documentacion/05_FAQ_tecnico.md`
2. Consultar en clase o por email

---

## 🚀 ¡Buena Suerte!

Este desafío combina conocimientos de:
- ✅ Machine Learning
- ✅ Economía de la Salud
- ✅ Análisis de Datos
- ✅ Pensamiento Crítico

**Recuerden:** El objetivo NO es solo obtener el mejor RMSE, sino **entender y comunicar** qué factores predicen el gasto de bolsillo en salud y por qué.

---

**Última actualización:** Noviembre 2025  
**Versión:** 1.0

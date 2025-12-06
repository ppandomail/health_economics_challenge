# TP Desafío ML Salud

## 1. Configuración de Estrategia (en yml)

* orden_lead: 1 -> Predecir el año siguiente (desde 2021 predecir 2022)
* presente: 2021
* excluir: [2020, 2021] -> Excluir estos años por pandemia, para que no genere sesgo en el resultado dadas los índices seleccionados 

## 2. Variables

* **Índice de envejecimiento**
  * INDICE_ENVEJECIMIENTO <- SP.POP.65UP.TO.ZS / SP.POP.0014.TO.ZS
  * Determinar la relación del índice con el gasto en salud

* **Índice Compuesto de Cobertura Educativa Normalizado por PIB**
  * COBERTURA_PROMEDIO <-  (SE.PRM.ENRR + SE.SEC.ENRR + SE.TER.ENRR) / 3 -> Calculo promedio de las 3 coberturas
  * INDICE_NORMALIZADO = COBERTURA_PROMEDIO / PIB_per_capita. -> Normaliza dividiendo por PIB per cápita (para evitar sesgos de escala)
  * COB_EDU_PROM_Z = (INDICE_NORMALIZADO - media_indice) / sd_indice  -> estandariza con z-score por país para análisis comparativos
  * Medir la cobertura educativa (primaria, secundaria y terciaria) ajustada al nivel económico del país para comparar equidad y eficiencia
  
* **Ratio gasto público/privado**
  * RATIO_GASTO <- NE.CON.GOVT.ZS / NE.CON.PRVT.ZS
  * Mide la intensidad del financiamiento estatal relativo al privado por país

* **Intensidad de Emisiones por Concentración Urbana**
  * INTENS_EMISIONES_CONCENTR_URB <- EN.GHG.CO2.PC.CE.AR5 / EN.URB.MCTY.TL.ZS
  * Indica la cantidad de emisiones de CO2 per cápita que un país produce por cada punto porcentual de su población que vive en megaciudades

## 3. Ejecución del pipeline

## 4. Análisis de resultados

* En 2022, el gasto en salud mostró una fuerte relación con el índice de envejecimiento
  * a mayor proporción de adultos mayores, mayor presión sobre el sistema sanitario y aumento del gasto público y privado en salud
* 
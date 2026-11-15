# Predicción de la mortalidad por insuficiencia cardíaca

<p align="center">
  <img src="assets/heartbeat.gif" alt="Monitor cardíaco animado" width="100%">
</p>

Este proyecto intenta predecir si un paciente con insuficiencia cardíaca fallecerá durante el periodo de seguimiento a partir de sus datos clínicos. Para ello se comparan tres modelos lineales de clasificación (**Naive Bayes gaussiano**, **LDA** y **regresión logística**) y se analiza en qué casos funciona mejor cada uno según sus supuestos.

## Datos

Se usa el conjunto de datos [Heart Failure Clinical Records](https://archive.ics.uci.edu/dataset/519/heart+failure+clinical+records) del repositorio UCI:

- **299 pacientes** y **12 variables clínicas**: edad, fracción de eyección, creatinina sérica, sodio, plaquetas, anemia, diabetes, hipertensión, sexo, tabaquismo, etc.
- **Variable objetivo:** `DEATH_EVENT`, que vale 1 si el paciente fallece durante el seguimiento.
- **Clases desbalanceadas:** un 68 % de los pacientes sobrevive y un 32 % fallece.

## Metodología

1. **Preprocesamiento**
   - Se elimina `time` (días de seguimiento), porque es una fuga de información: los pacientes que fallecen pronto tienen, por construcción, un seguimiento corto.
   - Partición estratificada 70 % / 30 % en entrenamiento y test.
   - Se aplica el logaritmo a las dos variables más asimétricas (`creatinine_phosphokinase` y `serum_creatinine`). Así se obtienen dos versiones de los datos, la original y la logarítmica.
   - Estandarización ajustada solo con los datos de entrenamiento.
2. **Evaluación**
   - **Métrica:** F1 de la clase "fallece". La *accuracy* engaña con clases desbalanceadas.
   - **Error de generalización:** validación cruzada de 10 particiones. Es el único criterio para elegir el modelo.
   - **Test:** se usa solo para comprobar que no hay sobreajuste.
3. **Modelos**
   - **Naive Bayes gaussiano:** supone independencia y normalidad.
   - **LDA:** tiene en cuenta las correlaciones mediante una covarianza común.
   - **Regresión logística:** sin supuestos sobre la distribución de los datos. Se buscan sus hiperparámetros (penalización `l1`/`l2` y `C`) y se prueba también con `class_weight="balanced"`.

## Resultados

| Modelo | Datos | F1 validación cruzada | F1 test |
|---|---|---|---|
| Naive Bayes | Original | 0.401 +- 0.198 | 0.167 |
| Naive Bayes | Logaritmo | 0.553 +- 0.146 | 0.400 |
| LDA | Original | 0.584 +- 0.194 | 0.293 |
| LDA | Logaritmo | 0.571 +- 0.187 | 0.489 |
| Regresión logística | Original | 0.624 +- 0.177 | 0.333 |
| Regresión logística | Logaritmo | 0.612 +- 0.151 | 0.478 |
| Regresión logística balanceada | Original | **0.663 +- 0.114** | 0.548 |
| Regresión logística balanceada | Logaritmo | 0.659 +- 0.120 | 0.531 |

El **modelo elegido** es la **regresión logística balanceada sobre los datos con logaritmo**. Tiene prácticamente el mismo F1 que el mejor, pero gracias a la penalización `l1` solo utiliza **dos variables**: la **fracción de eyección** y la **creatinina sérica**.

**Conclusiones principales**

- El logaritmo es decisivo para Naive Bayes y casi irrelevante para la regresión logística, lo que encaja con lo que cada modelo depende de la normalidad.
- El desbalanceo de clases es lo que más limita el rendimiento; los pesos de clase son la mejora más clara.
- El rendimiento final es moderado (F1 ≈ 0.66): con estas variables las dos clases se solapan mucho.


## Cómo ejecutarlo

Requiere Python 3.12 o superior.

```bash
pip install -r requirements.txt
jupyter notebook heart_failure_prediction.ipynb
```

## Licencia de los datos

Los datos provienen de Chicco, D. y Jurman, G. (2020), *Machine learning can predict survival of patients with heart failure from serum creatinine and ejection fraction alone*, BMC Medical Informatics and Decision Making, y se distribuyen en el repositorio UCI bajo licencia [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).

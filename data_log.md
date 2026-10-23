# Registro de decisiones (data log)

Registro de decisiones de datos y modelado del Proyecto Final Integrador (detección de fraude). Las cifras citadas salen de `deteccion_fraude.ipynb` y están en `results/`.

## Datos

| Decisión | Motivo |
|---|---|
| Se mantiene el corte cronológico original (`fraudTrain.csv` / `fraudTest.csv`) | Evaluación realista hacia adelante; el test no interviene en selección ni ajuste de umbrales; después de la evaluación final se reutiliza solo para diagnósticos descriptivos |
| Sin imputación ni eliminación de duplicados | 0 nulos y 0 duplicados en ambos archivos |
| `amt`: se aplica `log1p` y no se hace winsorizing | Los montos extremos (hasta 28.949 USD) son compras legítimas de `travel`, no errores; los árboles no necesitan el recorte |
| Se excluyen `first`, `last`, `street`, `trans_num`, `cc_num`, `unix_time`, `zip`, `city`, `state`, coordenadas crudas | PII o identificadores; las coordenadas se resumen en `dist_km` |
| `merchant` y `job` con codificación por frecuencia | Alta cardinalidad (693 y 494); en CV se ajusta solo con la ventana de entrenamiento de cada fold y en el ajuste final solo con train; no usa etiquetas |
| `category` y `gender` en one-hot / binaria | Baja cardinalidad |
| `dob` → edad al momento de la transacción | Variable disponible al autorizar |

## Variables históricas por tarjeta (devolución del tutor)

- Solo usan transacciones **anteriores** a la evaluada: `cumcount`, sumas acumuladas menos la fila actual, `rolling(..., closed='left')`.
- Las primeras transacciones del test usan el historial real de la tarjeta en train (es pasado).
- No se usan etiquetas pasadas (`is_fraud`) como variable: en producción llegan con demora.
- Verificación automática en el notebook: las variables calculadas con datos truncados al 2019-09-01 coinciden con las calculadas con todos los datos; un control positivo con "futuro" sí cambia (`results/verificacion_sin_fuga.csv`).

## Validación y selección

- `TimeSeriesSplit` de 4 ventanas expansivas dentro de train; desbalance tratado dentro de cada fold (`class_weight` / `scale_pos_weight`); sin remuestreo del test.
- Red neuronal (`MLPClassifier`, 64-32, `max_iter=300`): sin `class_weight`; se usan `sample_weight` por fold con una razón legítimas/fraudes análoga a la de los otros modelos, y sin `early_stopping`, que reservaría una fracción aleatoria del train y rompería el orden temporal. Las 12 ejecuciones de CV convergieron antes del tope, entre 60 y 130 iteraciones (`results/mlp_convergencia.csv`); con el tope anterior de 100, 4 de 12 no convergían. Origen: propuesta de un integrante del equipo (PR #1); queda segunda en CV y en test, XGBoost sigue seleccionado.
- Semillas: XGBoost y MLP se promedian sobre 3 semillas en CV; en test se reentrenan con 3 semillas, cada una con su propio umbral (`results/seeds_test.csv`). PR-AUC y ahorro neto son estables; precisión y falsas alarmas del umbral por costo varían bastante (XGBoost: precisión ≈ 72–80 %), por lo que se informan como rangos.
- Versiones fijadas en `requirements.txt`: el umbral operativo de XGBoost y la API de pandas (`groupby.rolling`) cambian entre versiones; con otra versión de XGBoost el umbral por costo (y con él precisión y falsas alarmas) puede diferir, mientras que PR-AUC es estable.
- Criterio de selección fijado **antes** de mirar el test: mayor PR-AUC medio en CV temporal; empate si está a menos de un desvío del mejor; ante empate gana el modelo más simple (logística < Random Forest < XGBoost < MLP). **Salvedad:** el MLP y su lugar en el orden de desempate se agregaron después de conocer los resultados de test de los otros tres modelos. El resultado no depende de ello (el MLP queda por debajo del umbral de empate). El resultado de la ablación de `n_prev` también se agregó después de ver SHAP; es descriptivo y no interviene en la selección.
- Umbral por costo elegido sobre las predicciones fuera de muestra del último fold de CV, nunca sobre el test.
- Punto operativo alternativo: revisar el 0,5 % de las transacciones con mayor score (independiente de la escala del score).

## Supuestos a confirmar

- Costo por falsa alarma: **10 USD** (sensibilidad con 5, 25 y 50 USD). A confirmar con el tutor/negocio.
- Un fraude marcado se bloquea en tiempo real y evita su monto íntegro.
- La anualización del ahorro es proporcional al período de test, sin crecimiento.
- Licencia del dataset: CC0: Public Domain (campo de licencia de Kaggle). El simulador Sparkov que lo generó tiene licencia MIT.

## Límites

- Datos sintéticos (simulador Sparkov): los resultados no son representativos de producción.
- Cambio de régimen entre train y test (tasa 0,58 % → 0,39 %): se reporta y se analiza; no se corrige modificando el test.

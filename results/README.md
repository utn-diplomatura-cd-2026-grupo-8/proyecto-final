# results/

Tablas y figuras generadas por `deteccion_fraude.ipynb` (se sobrescriben al ejecutarlo). Todas las cifras de `informe-final.md` salen de aquí.

| Archivo | Contenido |
|---|---|
| `eda_calidad.csv` | Tamaño, nulos, duplicados, cardinalidades y resumen de `amt` en train y test |
| `eda_h1_*.csv`, `eda_h2_categoria.csv`, `eda_h3_*.csv`, `eda_distancia.csv`, `fig_eda.png` | EDA por hipótesis (hora, categoría, monto, distancia) |
| `drift_mensual.csv`, `drift_categoria.csv`, `drift_descomposicion.csv`, `drift_psi.csv`, `fig_drift.png` | Análisis del cambio de tasa entre train y test |
| `verificacion_sin_fuga.csv` | Prueba anti-fuga: variables históricas idénticas con datos truncados |
| `cv_temporal_folds.csv`, `cv_temporal.csv`, `seleccion_modelo.csv` | Validación temporal y regla de selección ex ante |
| `model_metrics.csv`, `matriz_confusion.csv`, `fig_pr_ganancia.png`, `fig_confusion.png` | Evaluación final en test (modelos y baselines) |
| `temporal_metrics.csv`, `fig_temporal.png` | Desempeño mes a mes en test |
| `shap_global.csv`, `shap_local_case.csv`, `fig_shap_global.png`, `fig_shap_local.png`, `ablaciones.csv` | Interpretación |
| `escenario_economico.csv` | Ahorro neto por escenario y sensibilidad al costo de falsa alarma |
| `auditoria_grupos.csv` | Auditoría por género y edad |
| `resumen.csv` | Cifras clave citadas en el resumen ejecutivo |

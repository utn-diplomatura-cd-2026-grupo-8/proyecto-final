# Proyecto Final Integrador – Detección de fraude con tarjetas de crédito

Diplomado en Ciencia de Datos y Análisis Avanzado – Grupo 8.

Este repositorio contiene la entrega final: el informe (`informe-final.pdf`), el notebook que genera todas sus cifras (`deteccion_fraude.ipynb`) y las tablas y figuras en `results/`.

## Resultado principal

Evaluación única sobre `fraudTest.csv` (jun.–dic. 2020), umbral elegido por costo en validación temporal (supuesto: 10 USD por falsa alarma):

| Métrica | Regla de negocio | XGBoost |
|---|---:|---:|
| PR-AUC (piso 0,39 %) | – | 0,972 |
| Fraude capturado (% del monto) | 83,3 % | 98,8 % |
| Recall / precisión | 62,9 % / 28,5 % | 96,6 % / 75,9 % |
| Falsas alarmas por día | 17,5 | 3,4 |

Los datos son **sintéticos** (simulador Sparkov) y sus patrones son muy regulares: los valores absolutos no son representativos de producción. Ver el informe para el análisis de drift, la verificación anti-fuga, las ablaciones y la auditoría por grupos.

## Descarga del dataset

El dataset **no está incluido en el repositorio** (~212 MB comprimido, ~500 MB descomprimido), pero **no hace falta descargarlo a mano**: la primera celda del notebook lo hace sola.

- Si `dataset/fraudTrain.csv` y `dataset/fraudTest.csv` ya existen, no hace nada.
- Si no, usa `dataset.zip` de la raíz del proyecto, y si tampoco está, lo descarga de Kaggle (<https://www.kaggle.com/api/v1/datasets/download/kartik2112/fraud-detection>; es un dataset público y no requiere cuenta ni credenciales) y lo descomprime en `dataset/` (ignorada por git).
- Necesita conexión a internet solo la primera vez. Si la descarga falla (por ejemplo, sin red o detrás de un proxy), el notebook muestra un error claro: bajá el zip desde <https://www.kaggle.com/datasets/kartik2112/fraud-detection> y guardalo como `dataset.zip` en la raíz.

## Estructura

```
.
├── README.md
├── informe-final.pdf          # Informe final
├── deteccion_fraude.ipynb    # Notebook final, ejecutado de punta a punta
├── data_log.md               # Registro de decisiones y supuestos
├── requirements.txt
├── results/                  # Tablas y figuras que cita el informe
│   └── README.md
├── dataset.zip               # Se descarga automáticamente al ejecutar el notebook (no versionado)
└── dataset/                  # Generado al descomprimir (no versionado)
```

## Ejecución

Python 3.11+ (probado con 3.13).

### macOS / Linux

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
jupyter notebook deteccion_fraude.ipynb
```

### Windows (PowerShell)

```powershell
py -m venv .venv
.venv\Scripts\Activate.ps1          # si PowerShell bloquea el script: Set-ExecutionPolicy -Scope Process RemoteSigned
pip install -r requirements.txt
jupyter notebook deteccion_fraude.ipynb
```

En CMD, la activación es `.venv\Scripts\activate.bat`.

### Dependencia de OpenMP (XGBoost)

XGBoost necesita un runtime OpenMP del sistema. Si al hacer `import xgboost` aparece un error como `libomp.dylib could not be loaded` o `vcomp140.dll not found`, instalalo según el sistema:

| Sistema | Solución |
|---|---|
| macOS | `brew install libomp` |
| Windows | Instalar el [Microsoft Visual C++ Redistributable (2015–2022, x64)](https://aka.ms/vs/17/release/vc_redist.x64.exe), que incluye `vcomp140.dll`, y reiniciar la terminal |
| Linux | Normalmente ya está; si falta: `sudo apt install libgomp1` (Debian/Ubuntu) |

Verificación rápida: `python -c "import xgboost; print(xgboost.__version__)"`.

### Ejecución de punta a punta desde la terminal

Funciona igual en los tres sistemas (≈ 12 minutos; sobrescribe `results/`):

```bash
jupyter nbconvert --to notebook --execute deteccion_fraude.ipynb --output deteccion_fraude.ipynb
```

Ejecutá los comandos desde la raíz del repositorio: el notebook usa rutas relativas (`dataset/`, `results/`).

## Notas

- Semilla fija (`SEED = 42`); rutas relativas a la raíz del repositorio.
- El test no interviene en la selección ni en el ajuste de escaladores, codificadores o umbrales; los diagnósticos posteriores se identifican como tales.
- El dataset se publica en Kaggle con licencia CC0: Public Domain; igualmente no se versiona en el repositorio por su tamaño.

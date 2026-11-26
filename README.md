# Tesis2 — Órtesis vibroacústica para la detección preliminar de gonartrosis (VAG + Machine Learning)

Repositorio de la tesis de bachiller en Ingeniería Mecatrónica (UTEC) y del artículo científico derivado:

> **Órtesis pasiva portátil con sensores vibroacústicos para la detección preliminar y no invasiva de gonartrosis mediante Máquina de Vectores de Soporte (SVM)**
> *Portable Passive Orthosis with Vibroacoustic Sensors for the Preliminary and Non-Invasive Detection of Gonarthrosis Using Vibroarthrography (VAG) and a Support Vector Machine (SVM)*

**Autores:** Andrea Antuane Flores Leyva, Rafael Giordano Fuentes Bonnett
**Coautor (artículo):** Mahdi Tavakoli (University of Alberta)
**Asesor:** Deyby Huamanchahua Canchanya (UTEC)

---

## 1. ¿De qué trata el proyecto?

La gonartrosis (osteoartritis de rodilla) suele diagnosticarse tarde, cuando la radiografía ya muestra daño estructural. La **vibroartrografía (VAG)** registra las vibraciones y sonidos que produce la rodilla al moverse, que cambian cuando el cartílago se degrada.

El proyecto integra:

1. **Órtesis pasiva impresa en 3D** (PLA + TPU, neopreno), diseñada a partir del escaneo 3D de una rodilla.
2. **Electrónica embebida:** ESP32 DevKit V1, micrófonos MEMS INMP441 (I²S), almacenamiento en microSD (`.wav`) y transmisión por WiFi.
3. **Procesamiento de señales:** eliminación de DC, normalización y un banco de cuatro filtros pasa-banda (Butterworth, Chebyshev I, Chebyshev II y Elíptico).
4. **Machine Learning:** comparación de SVM, KNN, Random Forest y una red neuronal MLP para clasificar las señales en *sano* y *gonartrosis*.
5. **Interfaz gráfica** (Flask + SQLite + Tailwind + Chart.js), en el repositorio [APP_TESIS](https://github.com/Gibonn24/APP_TESIS).

Este repositorio contiene los **datos de características**, los **notebooks de entrenamiento y evaluación** y, como submódulos, los **proyectos LaTeX (Overleaf)** de la tesis y del artículo.

## 2. Estructura del repositorio

```
Tesis2/
├── Data/                         # Características extraídas de las señales VAG (CSV)
│   ├── butter/                   # Señales filtradas con Butterworth
│   ├── cheby1/                   # Chebyshev tipo I
│   ├── cheby2/                   # Chebyshev tipo II
│   └── ellip/                    # Elíptico
├── butter copy.ipynb             # Entrenamiento y evaluación con el filtro Butterworth
├── cheby1 copy.ipynb             # Ídem, Chebyshev I
├── cheby2 copy.ipynb             # Ídem, Chebyshev II
├── ellip copy.ipynb              # Ídem, Elíptico
├── tesis/                        # Submódulo → proyecto Overleaf de la tesis (main.tex)
├── paper/                        # Submódulo → proyecto Overleaf del artículo (template.tex, plantilla MDPI)
└── PLAN_MEJORAS_ARTICULO.md      # Plan de correcciones del artículo (observaciones del asesor + revisión técnica)
```

## 3. Datos

Cada carpeta de `Data/` tiene 4 archivos CSV, uno por sujeto. Cada fila es una grabación de un ciclo de flexo-extensión de rodilla (~5 s) con dos canales, izquierdo (`_L`) y derecho (`_R`).

| Archivo (ej. Butterworth) | Sujeto | Condición | Grabaciones | Uso en los notebooks |
|---|---|---|---|---|
| `features_butter_sano.csv` | 1 | Sin gonartrosis | 60 | Entrenamiento |
| `features_butter3_diagnosticado.csv` | 3 | Gonartrosis leve | 67 | Entrenamiento |
| `features_butter4_sano.csv` | 2 | Sin gonartrosis | 40 | Prueba |
| `features_butter_diagnosticado.csv` | 4 | Gonartrosis moderada | 41 | Prueba |

Total: **208 grabaciones de 4 sujetos**. El entrenamiento y la prueba usan **sujetos distintos**, así que el modelo se evalúa con personas que no vio.

**Columnas:** `Filename`, y por canal (`_L`, `_R`): `rms_amp` (amplitud RMS), `peak_freq` (frecuencia pico, no se usa en el modelo), `spectral_entropy` (entropía espectral), `zcr` (tasa de cruces por cero) y `mean_freq` (frecuencia media).

> Las grabaciones `.wav` originales y el script que aplica los filtros y extrae las características (generado en MATLAB) **todavía no están en el repositorio**. Ver [PLAN_MEJORAS_ARTICULO.md](PLAN_MEJORAS_ARTICULO.md), hallazgo C9.

## 4. Flujo de procesamiento

```
.wav (ESP32 + INMP441, 2 canales)
  └─► eliminación de DC + normalización
        └─► filtro pasa-banda 15–17 kHz  (Butterworth | Chebyshev I | Chebyshev II | Elíptico)
              └─► características por canal: RMS, entropía espectral, ZCR, frecuencia media
                    └─► CSV en Data/<filtro>/
                          └─► notebook:
                                1. Fusión bilateral por "máxima severidad":
                                   max(L,R) para RMS, ZCR y frecuencia media; min(L,R) para la entropía
                                2. Características derivadas: ratio_rms_freq = RMS / frecuencia media, log_rms = log(1+RMS)
                                3. StandardScaler ajustado solo con el entrenamiento
                                4. GridSearchCV (5 pliegues, refit = accuracy) para SVM, KNN, Random Forest y MLP
                                5. Evaluación en los sujetos de prueba: accuracy, precisión, recall, F1, AUC,
                                   matriz de confusión, curvas ROC y Precision–Recall
```

## 5. Cómo ejecutar

### Clonar (con los submódulos de Overleaf)

```bash
git clone --recurse-submodules https://github.com/Gibonn24/Tesis2.git
```

Los submódulos `tesis/` y `paper/` apuntan a proyectos privados de Overleaf. Para descargarlos necesitas acceso a esos proyectos y un *token de Git* de Overleaf (Account Settings → Git Integration). Al pedir credenciales, el usuario es `git` y la contraseña es el token. Sin acceso, puedes clonar solo el código y los datos:

```bash
git clone https://github.com/Gibonn24/Tesis2.git
```

### Entorno

Probado con Python 3.12:

```bash
pip install numpy pandas scikit-learn matplotlib seaborn jupyter
jupyter notebook
```

Abre el notebook del filtro que quieras y ejecuta **Kernel → Restart & Run All**. Ejecutar las celdas en orden es importante, porque varias celdas reutilizan variables (`grid_svm`, `grid_knn`, etc.).

### Trabajar con los documentos LaTeX

```bash
git submodule update --remote --merge            # traer lo último de Overleaf
cd tesis && git add . && git commit -m "..." && git push && cd ..   # subir cambios a Overleaf
git add tesis paper && git commit -m "Actualizar submodulos"       # registrar la versión en Tesis2
```

## 6. Repositorios y documentos relacionados

- **Interfaz gráfica (VAG-Analyzer):** <https://github.com/Gibonn24/APP_TESIS>
- **Tesis (LaTeX, Overleaf):** submódulo `tesis/`
- **Artículo (LaTeX, plantilla MDPI):** submódulo `paper/`

## 7. Privacidad y ética

Las grabaciones provienen de voluntarios que firmaron un consentimiento informado. Los CSV no deben contener nombres de participantes. Antes de publicar el repositorio o los datos, hay que anonimizar las rutas de la columna `Filename` (ver el plan, hallazgo C10).

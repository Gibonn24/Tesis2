# Plan de mejoras del artículo científico

**Artículo:** *Portable Passive Orthosis with Vibroacoustic Sensors for the Preliminary and Non-Invasive Detection of Gonarthrosis Using Vibroarthrography (VAG) and a Support Vector Machine (SVM)* — `paper/template.tex` (plantilla MDPI)
**Fuentes revisadas:** tesis completa (`tesis/`), artículo completo (`paper/`), los 4 notebooks y los 16 CSV de `Data/`, la Hoja 2 de observaciones del asesor (la Hoja 1 está marcada como obsoleta) y la carpeta de PDFs de referencias en Drive.
**Fecha de la revisión:** 25/11/2026

---

## 0. Resumen ejecutivo

1. **Observaciones del asesor (Hoja 2):** 19 en total. Las 1–3 ya están hechas (autores, título con VAG, palabras clave) y faltan **16 (de la 4 a la 19)**.
2. **Antes de atenderlas hay que corregir los resultados.** Al revisar el código encontré errores que cambian los números de las tablas, la elección del "mejor filtro" y del "mejor modelo", el resumen y las conclusiones. Varias observaciones del asesor (14, 16, 17, 19) dependen de esos números, así que primero va el **Bloque 0**.
3. **Orden recomendado:** Bloque 0 (resultados) → Bloque A (metodología: 7, 8, 9, 10, 11, 12) → Bloque B (resultados: 15, 16, 14, 17, 18, 13) → Bloque C (literatura y discusión: 4, 6, 5, 19) → Bloque D (cierre: resumen, conclusiones y secciones obligatorias de MDPI).

---

## 1. Bloque 0 — Hallazgos críticos que no están en la lista del asesor

Ninguno de estos puntos aparece en la Hoja 2, pero un revisor de *Sensors* o *Applied Sciences* los encontraría. Conviene contárselos al asesor antes de seguir.

### C1. Nombres de modelos intercambiados en la evaluación final
En la celda "4. EVALUACIÓN COMPARATIVA FINAL" de los notebooks, el diccionario `optimized_models` asigna mal los modelos:

| Notebook | Qué dice el código | Consecuencia |
|---|---|---|
| `butter copy.ipynb` | `"Red Neuronal (MLP)": grid_svm…` y `"SVM": grid_mlp…` | El "SVM ganador" (acc 0.8025, recall 1.0, AUC 0.8982) **es la red neuronal MLP** |
| `cheby2 copy.ipynb` | `"SVM": grid_knn…`, `"KNN": grid_mlp…`, `"Red Neuronal": grid_svm…` | El "SVM" (acc 0.8519) **es el KNN** |
| `cheby1 copy.ipynb` | `"Random Forest": grid_svm…`, `"SVM": grid_rf…` | El "SVM" **es el Random Forest** |
| `ellip copy.ipynb` | Correcto | — |

Por eso la **Tabla 8, las Figuras 34–36 y la conclusión "SVM con recall = 1.0"** del artículo (y de la tesis) describen en realidad a otro modelo.

**Confirmado con una réplica exacta** de los notebooks (mismo orden de filas, mismas grillas y semillas): con datos Butterworth, la red neuronal MLP (tanh, capas 100-50-25, alpha 0, Adam, lr 0.001) da en test 0.8025 / 0.7193 / 1.0000 / 0.8367 / 0.8982 con matriz TN=24, FP=16, FN=0, TP=41, idéntica a la Fig. 36. El **SVM real** (RBF, C = 10, γ = 0.01) da 0.7901 / 0.7500 / 0.8780 / 0.8090 / 0.9107.

### C2. Las tablas por filtro no corresponden a su filtro

| Tabla del artículo | De dónde salen realmente los números |
|---|---|
| Tabla 3 "Butterworth" (SVM 0.9605…) | Del notebook **cheby1**, que entrena y prueba con los sujetos **al revés** que los demás (entrena con S4 y prueba con S3), y además con SVM y KNN intercambiados |
| Tabla 4 "Chebyshev I" (SVM 0.7883…) | Del notebook **butter** |
| Tabla 5 "Chebyshev II" | Del notebook cheby2, pero con los modelos permutados (el 0.7726 de la fila "SVM" es del KNN) |
| Tabla 6 "Elíptico" | Correcta |

La réplica exacta lo confirma: los valores coinciden al cuarto decimal. Además, con la división de `cheby1` los modelos **fallan en test** (exactitud 0.55–0.61 y AUC ≈ 0.5), aunque su CV sea 0.96. En `cheby1`, la tabla de la celda 28 también muestra para "SVM" los hiperparámetros del KNN. Eso indica que las celdas se ejecutaron fuera de orden. La conclusión **"Butterworth supera en más de 14 % al segundo mejor filtro" no tiene sustento.**

### C3. El resumen y las conclusiones citan números que no existen en el trabajo
El *Abstract* (línea 112) y las *Conclusions* (líneas 1071–1075) dicen:
- que el **filtro elíptico** es el mejor, mientras que la sección 3.4.5 concluye que es Butterworth;
- que la exactitud es **94.4 %**, la sensibilidad **95.2 %** y la especificidad **93.6 %**, valores que **no aparecen en ningún notebook** ni en la sección de resultados (que reporta 0.8025);
- que hay **aislamiento con espuma acústica** de poliuretano, que no se describe en la metodología.

Estos números hay que **eliminarlos** y reemplazarlos por los resultados corregidos del Bloque 0. Así como están, son un problema de integridad.

> **Actualización (25/11/2026):** como medida provisional, el resumen (inglés y la versión comentada en español) y las conclusiones del artículo ya se igualaron a la tesis: Butterworth como mejor filtro (0.96 / 0.95), SVM con 80.25 % de exactitud, 71.93 % de precisión, recall 1.00, F1 0.8367 y AUC 0.8982, y autonomía de 28.5 h. Se quitaron la espuma acústica y el 94.4/95.2/93.6 %. **Ojo:** esos números de la tesis siguen afectados por C1 y C2, así que habrá que actualizarlos otra vez cuando termine el Bloque 0.

### C4. Evaluación con 4 sujetos y confusión por "identidad del sujeto"
- La base tiene **4 personas** (1 sana y 1 con OA para entrenar; 1 sana y 1 con OA para probar). El test mide si el modelo distingue a **dos personas concretas**, no a sanos de enfermos en general.
- Las Tablas 3–6 son de **validación cruzada dentro de los 2 sujetos de entrenamiento**. Los pliegues mezclan grabaciones de la misma persona (validación *record-wise*), lo que infla las métricas [Saeb et al., 2017].
- **Prueba hecha:** intenté distinguir a dos personas con la **misma** condición (S1 sano vs S2 sano; S3 OA vs S4 OA) usando las mismas características. Se logra entre **75 % y 86 %**, igual o más que sano vs OA (**69–80 %**). Es decir, las características capturan **quién es la persona** al menos tanto como **si tiene OA**.

**Qué hacer:**
1. **Ideal:** grabar a más participantes (meta mínima de 10 por grupo, idealmente 20 o más) y evaluar con *leave-one-subject-out*.
2. **Si no hay tiempo para nuevas grabaciones:** presentar el trabajo como **estudio piloto de factibilidad técnica**. Reportar los resultados por sujeto, mostrar la prueba de identidad como limitación y quitar afirmaciones como "herramienta de tamizaje eficaz".

### C5. Resultados corregidos (reanálisis independiente)
Volví a correr el mismo pipeline (mismas 6 características, mismo escalado, mismas grillas) **con las etiquetas correctas y la misma división de sujetos para los 4 filtros**. Los resultados en el **conjunto de prueba** son:

**División A** (entrena con S1 sano + S3 OA, prueba con S2 sano + S4 OA; n_test = 81):

| Filtro | Modelo | CV acc | Test acc | Recall | Especificidad | F1 | AUC |
|---|---|---|---|---|---|---|---|
| Butterworth | SVM (lineal, C=10) | 0.774 | 0.691 | 0.902 | 0.475 | 0.748 | 0.883 |
| Butterworth | KNN | 0.751 | 0.741 | 0.707 | 0.775 | 0.734 | 0.878 |
| Butterworth | RF | 0.711 | 0.790 | 0.854 | 0.725 | 0.805 | 0.886 |
| Butterworth | MLP | 0.719 | 0.728 | 0.512 | 0.950 | 0.656 | 0.769 |
| Chebyshev I | SVM | 0.759 | 0.728 | 0.854 | 0.600 | 0.761 | 0.856 |
| Chebyshev I | KNN | 0.735 | 0.753 | 0.805 | 0.700 | 0.767 | 0.870 |
| Chebyshev I | RF | 0.727 | 0.728 | 0.805 | 0.650 | 0.750 | 0.813 |
| Chebyshev I | MLP | 0.711 | 0.691 | 0.439 | 0.950 | 0.590 | 0.753 |
| Chebyshev II | SVM (RBF, C=1, γ=0.1) | 0.751 | **0.840** | 0.878 | 0.800 | 0.847 | **0.919** |
| Chebyshev II | KNN | 0.727 | **0.852** | 0.829 | 0.875 | 0.850 | 0.912 |
| Chebyshev II | RF | 0.703 | 0.815 | 0.829 | 0.800 | 0.819 | 0.902 |
| Chebyshev II | MLP | 0.727 | 0.654 | 0.976 | 0.325 | 0.741 | 0.871 |
| Elíptico | SVM | 0.751 | 0.728 | 0.854 | 0.600 | 0.761 | 0.856 |
| Elíptico | KNN | 0.743 | 0.790 | 0.878 | 0.700 | 0.809 | 0.902 |
| Elíptico | RF | 0.711 | 0.704 | 0.854 | 0.550 | 0.745 | 0.831 |
| Elíptico | MLP | 0.719 | 0.704 | 0.463 | 0.950 | 0.613 | 0.754 |

**División B** (al revés: entrena con S2 + S4 y prueba con S1 + S3): la exactitud de prueba cae a **0.68–0.76** con todos los filtros, y el modelo "ganador" cambia.

**Conclusión honesta:** con 4 sujetos, la exactitud en sujetos nuevos está entre **~0.65 y 0.85**. **Ningún filtro ni modelo gana de forma consistente**: depende de qué personas quedan en el entrenamiento. Con una sola característica (regresión logística) se obtiene entre 0.57 y 0.70.

> El script del reanálisis está fuera del repo (en la carpeta temporal de la sesión). Conviene incorporarlo al repositorio para que estos números sean reproducibles.

### C6. Banda de 15–17 kHz: justificación débil y posible artefacto
- El **INMP441** tiene respuesta plana de **60 Hz a 15 kHz**, con el punto de −3 dB en 15 kHz ([TDK InvenSense](https://invensense.tdk.com/digital/inmp441)). La banda que se analiza está **en el límite superior del micrófono o por encima de él**.
- El propio marco teórico del artículo y la tesis dicen que la información útil de la VAG está por **debajo de 1 kHz**: 50–200 Hz [fuente3], <100 Hz en rodillas sanas y 300–600 Hz en patológicas [fuente2]. Befrui et al. graban a 16 kHz, es decir, solo ven hasta 8 kHz.
- En la figura del filtro Butterworth (Fig. 23), el **espectrograma muestra una línea continua de energía en ~15.5 kHz durante toda la grabación**, también cuando la rodilla no produce eventos. Eso sugiere un **tono estacionario (posible ruido electrónico o de reloj I²S)** y no crepitación.
- Las características lo reflejan: `mean_freq` ≈ 15 900–16 000 Hz y `zcr` ≈ 31 900 ≈ 2 × 16 kHz en todos los sujetos. Las diferencias entre clases son de apenas ~30–80 Hz.

**Qué hacer:**
1. Grabar **"registros nulos"**: con la órtesis puesta y la pierna quieta, con el micrófono sin contacto y con el equipo sobre la mesa. Así se ve si el tono de ~15.5 kHz existe sin movimiento, que es justamente el riesgo de *shortcut learning* que describe [fuente13].
2. Repetir la extracción en **bandas fisiológicas** (por ejemplo 50–1000 Hz y 1–5 kHz) como análisis de sensibilidad.
3. En el artículo, **justificar la banda con datos**: PSD promedio por clase y SNR por banda. Si no se puede justificar, cambiarla.
4. Aclarar que el INMP441 es un micrófono **de aire**, no de contacto, y explicar cómo se acopla a la piel.

### C7. Inconsistencias entre el texto y el código
| Dice el artículo | Dice el código / la realidad | Dónde corregir |
|---|---|---|
| "stratified `train_test_split` … `stratify=y`" (3.4.3) | No se usa: la división es por sujeto con la columna `partition` | §3.4.3 |
| "la métrica principal será recall y accuracy" | `refit='accuracy'` en todos los modelos | §3.4.4 |
| KNN: "k = 7, uniform, euclidean … mejor resultado en k = 6" | butter: k = 22, manhattan; cheby1: k = 6, manhattan, distance | §4.4.2 |
| "Phase 5 – System Evaluation…; Phase 6 – UI" (lista de fases) | Solo existen §3.1–3.5, y la 3.5 es la interfaz | §3 (ver obs. 7) |
| Tabla 2: sujeto 4 con 42 muestras | 41 (total 208; test = 81) | Tabla 2 |
| "4 características" + fusión | El modelo usa **6** (agrega `ratio_rms_freq` y `log_rms`), y `log_rms` es redundante con RMS | §3.4.2 |
| IMU MPU6050 y ángulo de flexión | El ángulo no se usa en el modelo y la IMU no aparece en el prototipo de resultados | §3.2.4 y §4 |
| No se reportan la frecuencia de muestreo, el orden de los filtros ni el *ripple*/atenuación | Las figuras de MATLAB llegan hasta 24 kHz, lo que sugiere fs ≈ 48 kHz (**confirmar** con el firmware o el script) | §3.3.3 |
| "Since the scope does not include clinical diagnosis" | Se concluye que es "herramienta de tamizaje eficaz" | §3 y Conclusiones |
| Fusión L/R "de ambas extremidades" | Hay que aclarar si L/R son **dos micrófonos en la misma rodilla** o **ambas rodillas** | §3.4.2 |
| Subsección 4.4 duplicada (Figs. 31–33 = 28–30) | Falta la subsección de la red neuronal (MLP) | líneas 842–873 |

### C8. Lo que se dice de las referencias no coincide con los PDF
- **[fuente2] Befrui 2018**: el artículo dice "accuracy ~90 %". El PDF dice **especificidad ≈ 0.8 con sensibilidad 0.75**, con 59 rodillas asintomáticas y 40 con OA, SVM lineal y muestreo a 16 kHz. La tabla de la tesis también dice "Precisión ~90 %".
- **[fuente23] Ewart 2023**: el artículo dice "wavelets + SVM y RF, >85 %". El PDF dice **regresión logística con validación *leave-one-knee-out***, 84–85 % de exactitud, AUC 0.92–0.93 y 44 rodillas de 32 participantes. Las páginas en el .bib también están mal (469–470 son números de resumen; el suplemento es 669–717).
- **Qué hacer:** antes de enviar, comparar **cada afirmación numérica** de la revisión de literatura con su PDF. La plantilla de la tabla comparativa del punto 19 sirve para eso.

### C9. Reproducibilidad
- No están en el repo ni las **grabaciones `.wav`** ni el **script MATLAB** de filtrado y extracción de características (las figuras de los filtros son de MATLAB). Tampoco los diagnósticos ni el *firmware* del ESP32.
- Hay 4 notebooks casi idénticos que se fueron desincronizando entre sí. Conviene dejar **uno solo, parametrizado por filtro**, con semillas fijas, y ejecutarlo siempre con *Restart & Run All*.
- MDPI exige una declaración de **disponibilidad de datos y código**.

### C10. Privacidad y ética
- Las rutas de los CSV contienen el nombre de un voluntario (carpeta `sano_<nombre>`), y en el Overleaf de la tesis hay un PDF de consentimiento informado con nombre propio (`tesis/images/resultados/` y `tesis/secciones/`). Hay que **anonimizar** (S1–S4) antes de publicar el repositorio o enviar el artículo.
- MDPI pide **Institutional Review Board Statement** e **Informed Consent Statement** en estudios con personas. En la Hoja 1 el asesor sugirió quitar las "consideraciones éticas", pero la revista igual exige estas declaraciones. Hay que **consultarlo con el asesor**: puede ser necesario pedir una aprobación de ética o una exención formal.

### C11. Requisitos formales de MDPI
- `\documentclass[journal,…]`: falta elegir la revista (`sensors` o `applsci`).
- `\AuthorNames` (línea 86) no incluye a Mahdi Tavakoli. Además, en `\Author` aparece "Mahdin".
- Están comentadas: *Author Contributions, Funding, Institutional Review Board Statement, Informed Consent Statement, Data Availability Statement, Conflicts of Interest, Abbreviations*.
- `referencias.bib`: tiene una `}` suelta en la línea 301, la entrada `fuente34` con caracteres corruptos y tipos de entrada que `mdpi.bst` no reconoce (`@standard` en IEC61672 y `@online` en fuente27).

### Pasos del Bloque 0 (en orden)
1. **Reunión con el asesor (30 min):** presentar C1–C6 y decidir entre (a) grabar más sujetos o (b) presentarlo como estudio piloto. Esta decisión cambia el título, el resumen, los resultados y la discusión.
2. **Recuperar** los `.wav` y el script MATLAB y subirlos al repo, o a Drive si pesan mucho, con nombres anónimos.
3. **Unificar los notebooks** en uno solo (`pipeline.ipynb`) parametrizado por filtro: misma división, etiquetas correctas, semillas fijas, validación por sujeto. Debe guardar las tablas en CSV y las figuras en PNG **en inglés** dentro de `paper/Definitions/`.
4. **Registros nulos** y análisis de bandas (C6).
5. Regenerar todas las tablas y figuras de resultados. Recién entonces trabajar las observaciones 14, 16, 17 y 19.

**Esfuerzo estimado:** 3–5 días sin nuevas grabaciones; +2–4 semanas si se graba a más participantes.

---

## 2. Observaciones del asesor (Hoja 2) — plan detallado una por una

Las referencias de línea son de `paper/template.tex` y la numeración de figuras y tablas es la del PDF compilado actual.

### Bloque A — Metodología

#### Obs. 7 — Diagrama de la VDI 2221 y coherencia de las fases
**Qué pide:** incluir el diagrama de la VDI 2221, explicar si el flujo de la Fig. 6 es la VDI 2221 o una adaptación propia, y asegurar que todas las fases existan en el documento.
**Estado actual:** la Fig. 6 (`Workflow.png`, línea 254) no se explica. La lista de las líneas 259–267 tiene **6 fases** (5 = *System Evaluation*, 6 = *UI*), pero solo hay secciones para 5 (§3.5 = UI). No se cita la VDI 2221.
**Pasos:**
1. Redibujar en draw.io el **esquema genérico de la VDI 2221**, que va de la clarificación de la tarea al diseño detallado (requisitos → estructura de funciones → principios de solución → módulos → diseño preliminar → diseño definitivo → documentación). Citar la norma [VDI 2221 Blatt 1:2019] y Pahl & Beitz, *Engineering Design* (Springer, 2007).
2. Agregar una **tabla de correspondencia** con las columnas "Etapa VDI 2221 → Fase del proyecto → Sección del artículo".
3. Escribir un párrafo que diga explícitamente: *"Fig. 6 is an adaptation of VDI 2221 to a mechatronic–biomedical system; stages X and Y were merged/added because…"*.
4. Unificar la cantidad de fases. **Opción recomendada:** 6 fases, agregando una **§3.5 System Evaluation and Statistical Analysis** (validación por sujeto, métricas, intervalos de confianza) y pasando la UI a §3.6. Actualizar el texto de la línea 448 ("Section 3.5") con `\ref`.
5. En la Fig. 6, marcar cada fase con un rectángulo punteado y su número (ya hecho en la Hoja 1; verificar que coincida con la nueva numeración).
6. Citar 1–2 artículos que usan la VDI 2221 en órtesis, por ejemplo [Bioengineering 2022, 9(7), 287](https://www.mdpi.com/2306-5354/9/7/287) y [Eng. Proc. 83(1), 23](https://www.mdpi.com/2673-4591/83/1/23).

**Listo cuando:** existe una figura de la VDI 2221 con cita, una tabla de correspondencia y el mismo número de fases en la lista, en la figura y en las secciones.
**Esfuerzo:** 1 día.

#### Obs. 8 — Información sobre la rodilla y la gonartrosis antes de la Fase 1
**Qué pide:** biomecánica de la rodilla y explicación de la gonartrosis, con figuras o tablas, antes de la Fase 1 o como subtítulo antes de la metodología.
**Pasos:**
1. Crear **§3.1 "Background: Knee Biomechanics and Gonarthrosis"** (o una §2 aparte antes de *Methodology*) y renumerar las fases.
2. Contenido, adaptado del Capítulo 2 de la tesis:
   - anatomía funcional: articulaciones tibiofemoral y patelofemoral, cartílago, meniscos;
   - movimientos y rangos: flexión 0–135°, extensión 0°, rango funcional para sentarse y levantarse. Esto también resuelve un pendiente de la Hoja 1 ("tabla de ángulos de movimiento");
   - fisiopatología de la OA y **escala Kellgren–Lawrence** (citar Kellgren & Lawrence, *Ann Rheum Dis* 1957);
   - **por qué la rodilla "suena":** crepitaciones y bandas de frecuencia reportadas (fuente2, fuente3, fuente10). Esto justifica la elección de la banda (ver C6).
3. Figuras: (a) esquema de la rodilla con los ejes de flexo-extensión y (b) grados KL. **Redibujarlas** o pedir permiso, porque MDPI requiere permiso para reproducir figuras de terceros (por ejemplo `kl_clasificacion.png` y `comparacion.png` de la tesis).
4. Tabla de **rangos de movimiento** (actividad → ángulo de rodilla) con cita.

**Listo cuando:** existe la nueva subsección con 1–2 figuras propias o redibujadas, una tabla de ROM y al menos 6 citas de artículos.
**Esfuerzo:** 1–1.5 días.

#### Obs. 9 — Requerimientos computacionales en la tabla de requerimientos
**Estado actual:** la Tabla 1 (líneas 278–311) tiene 4 bloques: electrónica, acondicionamiento, órtesis e interfaz.
**Pasos:** agregar un bloque **"Computational / Data Processing"** con requerimientos medibles:
- frecuencia de muestreo ≥ X kHz y resolución de 24 bits (dato real del INMP441);
- formato `.wav` sin compresión y almacenamiento ≥ N grabaciones;
- tiempo de procesamiento y clasificación < 2 min por sesión (ya se menciona en el texto);
- transmisión WiFi y HTTP POST a la interfaz;
- *pipeline* reproducible: Python 3.12 y scikit-learn 1.6, semillas fijas;
- evaluación sobre sujetos no vistos.

Idealmente, convertir la tabla al formato de lista de requerimientos de la VDI 2221: ID, requerimiento, tipo (exigencia o deseo) y valor.
**Esfuerzo:** 2 h.

#### Obs. 10 — Diagrama de bloques o de flujo que relacione los subsistemas
**Estado actual:** la Fig. 7 ("General functional diagram", línea 330) no deja ver cómo se conectan las partes mecánica, electrónica y computacional.
**Pasos:**
1. Rehacerla con **carriles (*swimlanes*)** por dominio: Mecánico (órtesis, sujeción, acople) → Electrónico (micrófonos, IMU, ESP32, SD, batería) → Computacional (filtrado, características, modelo) → Interfaz (Flask/BD/alerta).
2. Usar tres tipos de flecha con leyenda: **energía**, **señal** y **datos/información**, como en el análisis funcional de la VDI 2221.
3. Escribir un párrafo que recorra el diagrama de izquierda a derecha.

**Esfuerzo:** 0.5 día.

#### Obs. 11 — Matriz morfológica, citas de las metodologías y Figs. 9–11
**Pasos:**
1. **Fig. 8 (matriz morfológica):** pasarla a una **tabla LaTeX solo con texto**, con subfunciones en las filas (1–7) y alternativas A–E en las columnas. Marcar con colores o símbolos la combinación S1 a S5.
2. **Citas:** VDI 2221, VDI 2225 (Blatt 3), Zwicky (método morfológico) y Pahl & Beitz, más **artículos de revista que las usen** (ver la obs. 7; buscar en MDPI con `"VDI 2225"`). También queda resuelto el pendiente de la Hoja 1 "citar VDI 2221/2225".
3. **Figs. 9–10 (evaluación técnica y económica):** quitar la fila con el nombre antiguo del proyecto o actualizarla al título nuevo. Mejor aún, pasarlas a **tablas LaTeX**, que se leen mejor y quedan en inglés.
4. **Fig. 11 (diagrama técnico–económico):**
   - reemplazar las "X" por **círculos rellenos** con la etiqueta S1…S5;
   - explicar el método con las ecuaciones: $X_i = \sum g_j p_{ij} / (p_{max}\sum g_j)$, $Y_i$ análogo, la recta $Y = X$ como equilibrio y el punto ideal (1, 1). La mejor solución es la **más cercana a (1, 1)** o la de mayor $s_i = \sqrt{X_i Y_i}$ (fuerza de la solución según la VDI 2225). Reportar el valor para cada solución en una tabla pequeña.

**Esfuerzo:** 1 día.

#### Obs. 12 — "?" en la cita de §3.3.1 y entorno controlado
**Causa probable del "?":** la entrada `IEC61672` es de tipo `@standard`, que `paper/Definitions/mdpi.bst` **no define** (solo tiene `manual`, `misc` y `techreport`, entre otros). Revisar el log de Overleaf (*Logs and output files*) para confirmarlo.
**Pasos:**
1. Cambiar `@standard{IEC61672,…}` por `@manual{IEC61672, title={…}, organization={International Electrotechnical Commission}, year={2013}, note={IEC 61672-1}}`. Revisar también `@online{fuente27}`, que conviene cambiar a `@misc` con `howpublished` y `url`.
2. Agregar al texto de la Fig. 13 (diagrama temporal) un párrafo sobre las **condiciones controladas**: sala cerrada y silenciosa, nivel de ruido ambiental **medido** en dB(A) (si no se midió, medirlo ahora con un sonómetro o decir que no se midió), participante sentado sin apoyo de pies, la misma silla y la misma hora del día, sin otros equipos encendidos, lubricación de piezas móviles, la misma colocación de la órtesis y un operador entrenado.
3. Mencionar los "registros nulos" de C6 como control de ruido.

**Esfuerzo:** 2 h, más la medición.

### Bloque B — Resultados (hacer después del Bloque 0)

#### Obs. 15 — Etapa mecánica y electrónica completas
**Pasos (mecánica):**
1. **Diagrama de bloques del proceso mecánico:** escaneo 3D (iReal 2E) → limpieza de la malla STL → CAD paramétrico en Fusion 360 → selección de material → impresión 3D (PLA/TPU, parámetros de impresión) → ensamble con neopreno y hebillas → prueba de ajuste en 3 usuarios.
2. **Figura compuesta de tres paneles:** (a) escaneo (`frontal_escaneada.png`), (b) CAD (`ortfusion.png`) y (c) órtesis implementada (`ortimpre.png`). Esto reemplaza las Figs. 17–20, que el asesor pide juntar.
3. **Vista explosionada** de la órtesis desde Fusion 360 (*Animation → Explode*), con globos numerados y una tabla de piezas: pieza, material y cantidad.
4. **Selección de material:**
   - tabla comparativa de PLA, PETG, ABS y TPU con resistencia a tracción, módulo de Young, elongación, densidad, costo, facilidad de impresión y contacto con la piel (con citas a fichas técnicas o artículos);
   - **análisis de resistencia por elementos finitos** en Fusion 360 (*Simulation → Static Stress*) de la pieza crítica, la bisagra o lengüeta, con la carga de sujeción y de flexión estimada. Reportar el esfuerzo de von Mises máximo, el factor de seguridad y el desplazamiento para PLA y para la alternativa.
   - Si se puede, citar el artículo de órtesis impresa que usa VDI 2221 con caracterización de material ([Eng. Proc. 83(1), 23](https://www.mdpi.com/2673-4591/83/1/23)).

**Pasos (electrónica, hoy prácticamente ausente en el artículo):**
1. **Esquemático:** ya existe en la tesis (`tesis/images/resultados/esquemáticopcb.png` y `Esquematico.png`). Traducirlo y llevarlo a `paper/Definitions/`.
2. **PCB:** diseño (`disenopcb.png`) e integración en la órtesis (`integraortpcb.png`). Describir número de capas, dimensiones, software usado, conectores, alimentación (batería Li-ion de 3.7 V, regulador) y pulsador/LED.
3. **Diagrama de flujo del firmware:** inicio → WiFi → espera del pulsador → LED encendido → grabación I²S de 5 s a fs = ? → escritura del `.wav` en la SD → envío por WiFi → LED apagado.
4. **Conexionado:** tabla de pines del ESP32 contra INMP441 (SCK, WS, SD), SD (SPI) e IMU (I²C).
5. Corregir "I\textsubscript{2}S" (líneas 453 y otras) por "I\textsuperscript{2}S".

**Esfuerzo:** 2–3 días (FEA y vista explosionada incluidas).

#### Obs. 16 — Explicar las Figs. 23–26 y tabla cuantitativa de filtros
**Pasos:**
1. Unir las 4 figuras en **una sola de 2×2** con paneles (a)–(d), regenerada **en inglés** (hoy dicen "Señal filtrada", "Tiempo", "Frecuencia"). Ver también la obs. 18.
2. Un párrafo por panel que explique señal temporal, espectro, PSD de Welch y espectrograma, y qué diferencia se ve entre los filtros. Mencionar la línea de ~15.5 kHz del espectrograma (C6).
3. **Tabla cuantitativa de filtros** (columnas): orden, *ripple* en banda de paso (dB), atenuación en banda de rechazo (dB), ancho de transición, retardo de grupo máximo en la banda, **energía retenida en la banda (%)**, **correlación con la señal original**, **SNR antes y después**, y **exactitud de prueba del mejor modelo** (de C5). Se calcula con los `.wav` y el script recuperado en el Bloque 0.
4. Agregar la **respuesta en frecuencia** (magnitud y fase) de los 4 filtros superpuestos en una figura (lo que el asesor llama "gráficas del proceso de filtrado").
5. Citar en cada filtro un trabajo que lo haya usado en VAG o en bioseñales (pendiente de la Hoja 1).

**Esfuerzo:** 1–1.5 días.

#### Obs. 14 — Tablas consecutivas 3–6 y gráficas del proceso de filtrado
**Pasos:**
1. Reemplazar las Tablas 3–6 por **una sola tabla** (filtro × modelo) con CV acc, test acc, recall, especificidad, F1 y AUC, como la de C5 pero con los números finales del Bloque 0. Marcar en negrita el mejor valor por columna.
2. Agregar **1–2 párrafos de interpretación** después de la tabla: qué filtro y qué modelo, cuánta diferencia hay, si la diferencia es significativa (prueba de McNemar o intervalos de confianza por *bootstrap*) y la advertencia sobre los 4 sujetos.
3. Las gráficas del proceso de filtrado quedan cubiertas en la obs. 16 (punto 4).
4. Opcional: un *heatmap* de test acc (filtro × modelo) como figura resumen.

**Esfuerzo:** 0.5 día, después del Bloque 0.

#### Obs. 17 — Juntar las Figs. 28 y 31, 29 y 32, 30 y 33
**Diagnóstico:** **no son figuras para comparar, son duplicados.** La subsección "Machine Learning Classification Model" (SVM, KNN y RF) está **copiada dos veces** (líneas 809–840 y 842–873). Las etiquetas `fig:mapaSVM`, `fig:KNN` y `fig:RF` están duplicadas.
**Pasos:**
1. **Borrar las líneas 842–873.**
2. Agregar la subsección faltante de **red neuronal (MLP)** con su figura (`tesis/images/resultados/rn.png`, regenerada en inglés).
3. Hacer **una figura 2×2** con la búsqueda de hiperparámetros: (a) SVM por kernel, (b) KNN según k, (c) RF según n_estimators y profundidad y (d) MLP según arquitectura. Regenerarla desde el notebook unificado, en inglés y con las etiquetas correctas (las actuales vienen del notebook cheby1, ver C2).
4. Responder al asesor explicando que eran duplicados.

**Esfuerzo:** 0.5 día.

#### Obs. 18 — Fig. 36 en español y revisión de todas las figuras
Figuras con texto en español detectadas:
- **Fig. 36** `matrizConfusion.png`: "Matriz de Confusión", "Sano/Enfermo", "Aciertos/Errores".
- **Fig. 35** `ROC.png`: "Curva ROC", "Tasa de Falsos Positivos…".
- **Fig. 28** `svm2.png`: "Comparación de Kernels", "Exactitud Promedio".
- Muy probablemente también `CurvaPre.png`, `knnnew.png`, `rf.png` y `rn.png`, que salen de los mismos notebooks.
- **Figs. 23–26** `Filtro *.jpg` (MATLAB): "Señal filtrada", "Tiempo", "Frecuencia", "Espectro de Magnitud", "Espectrograma".

**Pasos:**
1. En el notebook unificado, poner todos los textos de las gráficas en inglés: *Healthy/OA, True/Predicted label, False Positive Rate, Accuracy*, etc. Guardar con `plt.savefig('paper/Definitions/<nombre>.png', dpi=300, bbox_inches='tight')`.
2. Regenerar en MATLAB las figuras de los filtros con etiquetas en inglés.
3. Revisar una por una las 36 figuras y las tablas: título dentro de la imagen (quitarlo, porque basta el *caption*), idioma, resolución ≥ 300 dpi y unidades.
4. Lista de control: Fig. 9–11 (evaluaciones, ya en inglés), Fig. 13, Fig. 21 y Fig. 27 (corriente).

**Esfuerzo:** 0.5 día.

#### Obs. 13 — Mencionar la Fig. 16 y todas las figuras y tablas en el texto
Encontré **15 etiquetas que nunca se citan** en el texto:
`fig:fuente6, fig:diagrama_de_bloques, fig:matriz_morfologica, fig:evaluacion_tecnica, fig:evaluacion_economica, fig:evaluacion_tecnico-economica, fig:solucion_seleccionada, fig:Disp (Fig. 16), fig:tomadatos1, fig:filtro1–4, fig:analisiscorr, tab:svm_final`.

**Pasos:**
1. Agregar `Figure~\ref{...}` o `Table~\ref{...}` en una oración que **interprete** cada una. Ejemplo para la Fig. 16: *"Figure~\ref{fig:Disp} shows the pairwise scatter plots and kernel density estimates of the four fused features…"*.
2. Verificar con este comando, que debe devolver vacío:
   ```powershell
   python -c "import re;t=open('paper/template.tex',encoding='utf-8').read();L=re.findall(r'\\label\{([^}]+)\}',t);R=set(re.findall(r'\\ref\{([^}]+)\}',t));print([l for l in L if l not in R])"
   ```
3. Eliminar los labels duplicados (obs. 17) y los labels con caracteres no ASCII (`fig:acondicionamiento_señal` → `fig:signal_processing`).

**Esfuerzo:** 2 h. Hacerlo al final del Bloque B.

### Bloque C — Literatura, introducción y discusión

#### Obs. 4 — Al menos ~40 referencias, en su mayoría artículos de revista (de preferencia de la revista destino)
**Estado actual:** el artículo cita **25 referencias**, de las cuales solo **13 son artículos de revista**. El resto son libros (Oppenheim, Proakis, Webster, Rangayyan), una norma, un reporte de la OMS, *preprints* de arXiv, un resumen de congreso y actas.
**Pasos:**
1. **Elegir la revista destino ahora**, porque condiciona las citas preferidas y el formato. Recomendación: **Sensors** (publica mucho sobre VAG y emisión acústica de rodilla) o **Applied Sciences**. Fijarla en `\documentclass[sensors,…]`.
2. **Reutilizar las referencias de la tesis que ya tienen PDF en Drive** y no se citan en el artículo: fuente3, 7, 8, 9, 10 (*Sensors* 2025), 12, 14, 15, 18, 19, 21, 24, 25 y **49** (*"A novel automatic knee OA detection method based on VAG signals"*, que está en Drive pero no en el `.bib`). Son unas 14 más.
3. **Agregar artículos de MDPI** encontrados en la búsqueda (verificar el DOI de cada uno antes de citar):
   - *Sensors* 2022, 22(6), 2176 — Diagnostics of Articular Cartilage Damage Based on Generated Acoustic Signals Using ANN—Part I (<https://www.mdpi.com/1424-8220/22/6/2176>)
   - *Sensors* 2020, 20(7), 1998 — Reliability of Vibroarthrography to Assess Knee Joint Sounds in Motion (<https://www.mdpi.com/1424-8220/20/7/1998>)
   - *Sensors* 2022, 22(23), 9027 — Test-Retest Reliability of Acoustic Emission Sensing of the Knee during Physical Tasks (<https://www.mdpi.com/1424-8220/22/23/9027>)
   - *Sensors* 2020, 20(15), 4138 — Vibration Characterization of the Human Knee Joint in Audible Frequencies (<https://www.mdpi.com/1424-8220/20/15/4138>)
   - *Sensors* 2021, 21(19), 6495 — The Assessment of the Condition of Knee Joint Surfaces with Acoustic Emission Analysis (<https://www.mdpi.com/1424-8220/21/19/6495>)
   - *Sensors* 2022, 22(15), 5549 — Using Nonlinear Vibroarthrographic Parameters for Age-Related Changes Assessment in Knee Arthrokinematics (<https://www.mdpi.com/1424-8220/22/15/5549>)
   - *Sensors* 2019, 19(12), 2683 — A Glove-Based Form Factor for Collecting Joint Acoustic Emissions (<https://www.mdpi.com/1424-8220/19/12/2683>)
   - *Applied Sciences* 2019, 9(5), 908 — Developmental Screening System for Patient Vibration Signals with Knee Disorder (<https://www.mdpi.com/2076-3417/9/5/908>)
   - *Entropy* 2013, 15(4), 1375 — Classification of Knee Joint Vibration Signals Using Bivariate Feature Distribution Estimation… (<https://www.mdpi.com/1099-4300/15/4/1375>)
   - *Diagnostics* 2022, 12(10), 2362 — Detection and Classification of Knee Osteoarthritis (<https://www.mdpi.com/2075-4418/12/10/2362>)
   - *Sensors* 2026, 26(11), 3563 — Wearable Sensors and Artificial Intelligence for Ecological Knee Osteoarthritis Assessment (<https://www.mdpi.com/1424-8220/26/11/3563>)
   - Para el diseño: *Bioengineering* 2022, 9(7), 287; *Applied Sciences* 2020, 10(16), 5459; *Applied Sciences* 2026, 16(4), 2035 (órtesis activa de rodilla).
4. **Artículos clave fuera de MDPI** (revista, verificar el DOI):
   - Teague et al., *IEEE Trans. Biomed. Eng.* 2016 — sensado de emisiones acústicas de rodilla con dispositivo vestible
   - Saeb et al., *GigaScience* 2017, doi:10.1093/gigascience/gix019 — validación por sujeto vs por registro (clave para C4)
   - Little et al., *GigaScience* 2017, doi:10.1093/gigascience/gix020 — comentario sobre estrategias de validación cruzada
   - Varoquaux, *NeuroImage* 2018 — errores de la validación cruzada con muestras pequeñas
   - Kellgren & Lawrence, *Ann. Rheum. Dis.* 1957 — escala KL
   - Hunter & Bierma-Zeinstra, *Lancet* 2019 — revisión sobre osteoartritis
   - Rangayyan & Wu 2008 (*Med. Biol. Eng. Comput.*) y Krishnan et al. 2000 (*IEEE TBME*) — VAG clásica
   - Mascaro et al. 2009 (*Med. Eng. Phys.*) y Schlüter et al. 2019 (*PLoS One*) — emisión acústica en OA
5. **Quitar o reducir** las fuentes que no son artículos: noticias (fuente33), sitio de la OMS (fuente27; reemplazar por GBD 2023, que ya está), libros de ML (fuente39, 41) y tesis. Los libros de DSP pueden quedarse 1–2 veces.
6. Corregir los errores del `.bib` (C8 y C11) y unificar los nombres de revista abreviados según ISO 4 (lo pide la plantilla MDPI).
7. Subir a Drive los PDF que faltan, con el número de la entrada. Hoy **faltan PDFs** de: 27 (web), 33 (web), 43, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, IEC61672, WHO2018, Oppenheim, Proakis 2006, Webster y el *dataset* de Kaggle.

**Meta:** entre 40 y 50 referencias, al menos 32 de ellas artículos de revista, de los cuales 10 o más de MDPI (*Sensors* o *Applied Sciences*).
**Esfuerzo:** 2–3 días, incluyendo leer los resúmenes.

#### Obs. 6 — Revisión crítica de literatura en 4 párrafos, sin figuras
**Estado actual:** §2 (líneas 183–231) resume autor por autor e incluye **5 figuras ajenas** (Figs. 1–5).
**Pasos:**
1. **Eliminar las Figs. 1–5**, que además requieren permiso de reproducción.
2. Reescribir la sección en **4 párrafos temáticos**, cada uno con 8–15 citas agrupadas:
   - **P1 — Contexto y tendencia:** limitaciones de las imágenes (radiografía, MRI, ecografía) y crecimiento de la VAG y las emisiones acústicas como biomarcador. Citas: fuente4, 16, 20, 21, 24, Teague 2016 y Hunter 2019.
   - **P2 — Síntesis de hallazgos:** agrupar por sensor (acelerómetro, piezoeléctrico, micrófono de contacto o MEMS), por protocolo (flexo-extensión, sit-to-stand, marcha) y por clasificador (SVM, RF, ANN, LR), con los rangos de desempeño reportados. Citas: fuente1, 2, 10, 12, 17, 19, 23, 35, 49 y Sensors 2022, 22(6), 2176.
   - **P3 — Análisis crítico:** muestras pequeñas (muchos estudios tienen menos de 10 personas por grupo), falta de protocolos estandarizados, confiabilidad test-retest (Sensors 2020/1998; 2022/9027), *shortcut learning* y confusión por identidad del sujeto (fuente13, Saeb 2017), validación por registro y equipos de laboratorio poco portables.
   - **P4 — Vacío y objetivo:** falta un sistema **vestible, de bajo costo y reproducible** que integre adquisición, procesamiento y alerta, y que se evalúe con sujetos no vistos. Este es el objetivo del artículo.
3. Opcional, recomendado: una **tabla comparativa** de estudios (autor, n de sujetos, sensor, fs, banda, características, clasificador, esquema de validación, resultado), que sirve también para la discusión de la obs. 19. Corregir ahí los datos de fuente2 y fuente23 (C8).

**Esfuerzo:** 1.5 días.

#### Obs. 5 — Último párrafo de la introducción más específico
**Estado actual (línea 181):** el párrafo es genérico ("high-precision tool…", "gait…", "remote monitoring").
**Pasos:** reescribirlo con datos concretos del proyecto:
- órtesis pasiva impresa en 3D (PLA/TPU) a partir de un escaneo de rodilla;
- 2 micrófonos MEMS INMP441 + ESP32, grabación de 5 s (indicar la fs confirmada en el Bloque 0), protocolo de flexo-extensión sentado y transmisión por WiFi;
- banco de 4 filtros IIR y 4 características espectrales y temporales, con fusión bilateral;
- comparación de 4 clasificadores con **evaluación en sujetos no vistos**;
- interfaz web de alerta;
- lista de **contribuciones** (3–4 viñetas o una oración enumerada);
- quitar "gait", porque no se probó la marcha, y "high-precision".

**Esfuerzo:** 1 h, después del Bloque 0.

#### Obs. 19 — Sección de Discusión antes de las Conclusiones
**Estado actual:** está comentada (línea 1064: `%\section{Discusión de resultados}`).
**Pasos:** crear **§5 Discussion** con esta estructura (1.5–2 páginas, todo en párrafos):
1. **Hallazgos principales**, con los números corregidos.
2. **Comparación con la literatura:** tabla y párrafos frente a fuente1 (86.8 %), fuente2 (sens 0.75 / espec 0.8), fuente23 (84–85 %, AUC 0.92–0.93, LOOCV), fuente35 (87 %), fuente19 (~95 %) y fuente10. Explicar las diferencias: número de sujetos, tipo de sensor, banda analizada, esquema de validación y movimiento.
3. **Discusión por fase** (como pidió la Hoja 1): mecánica (ajuste en 3 usuarios), electrónica (autonomía de 28.5 h y su cita), procesamiento (filtros y banda, C6), ML (validación por sujeto, C4) e interfaz.
4. **Aportes:** costo bajo (incluir el costo total de materiales), portabilidad, pipeline abierto y comparación sistemática de filtros.
5. **Limitaciones:** 4 sujetos, confusión por identidad, una sola sesión sin test-retest, banda en el límite del micrófono, IMU no usada en el modelo, ruido ambiental no medido (si aplica) y etiquetas clínicas (grado KL solo en los sujetos con OA).
6. **Trabajo futuro:** más participantes estratificados por KL, *leave-one-subject-out*, bandas < 1 kHz, uso del ángulo de la IMU para segmentar flexión y extensión, sensores de contacto (piezo o acelerómetro) y validación clínica.

**Esfuerzo:** 1.5 días.

### Bloque D — Cierre

#### D1. Resumen (*Abstract*)
Reescribirlo **desde cero** con los números corregidos, en 200–250 palabras y con la estructura introducción, objetivo, metodología, resultados (con números) y conclusión (pendiente de la Hoja 1). Quitar el 94.4/95.2/93.6 %, el "elliptic filter optimized SNR" y la "acoustic foam" (C3).

#### D2. Conclusiones
Reescribirlas en párrafos (no viñetas), coherentes con la discusión y sin afirmar eficacia clínica.

#### D3. Secciones obligatorias de MDPI (C11)
*Author Contributions* (taxonomía CRediT), *Funding*, *Institutional Review Board Statement*, *Informed Consent Statement*, *Data Availability Statement* (enlace al repo y los datos anonimizados), *Conflicts of Interest* y *Abbreviations* (VAG, OA, KL, SVM, KNN, RF, MLP, MEMS, PSD, ZCR, ROC, AUC, IMU, PCB, I²S…). Si se usó IA generativa, declararlo en *Acknowledgments* según la política de MDPI.

#### D4. Revisión final
- [ ] Compila sin *warnings* de referencias ni "??".
- [ ] Todas las figuras y tablas se citan (script de la obs. 13).
- [ ] Todo el texto de las figuras está en inglés y a 300 dpi.
- [ ] Mismo número de fases en el texto, la figura y las secciones.
- [ ] Los números del resumen, los resultados, la discusión y las conclusiones son idénticos y salen de `pipeline.ipynb`.
- [ ] Entre 40 y 50 referencias, en su mayoría artículos, con DOI verificado.
- [ ] Datos anonimizados y declaraciones de ética resueltas con el asesor.
- [ ] Tiempo presente en la metodología (pendiente de la Hoja 1).

---

## 3. Cronograma sugerido (2 personas)

| Semana | Bloque | Entregable |
|---|---|---|
| 1 | **0** | Reunión con el asesor; `.wav` y script en el repo; notebook unificado con etiquetas correctas; registros nulos; tabla de resultados definitiva |
| 2 | **A** | Obs. 7, 8, 9, 10, 11 y 12 hechas en Overleaf |
| 3 | **B** | Obs. 15 (FEA, vista explosionada, electrónica), 16, 14, 17, 18 y 13 |
| 4 | **C** | Obs. 4 (refs), 6 (revisión en 4 párrafos), 5 (intro) y 19 (discusión) |
| 5 | **D** | Resumen, conclusiones, secciones MDPI y revisión final → enviar al asesor |

Si se decide grabar más participantes, el Bloque 0 se alarga 2–4 semanas. Mientras tanto se pueden adelantar los Bloques A y B (salvo las obs. 14, 16 y 17).

## 4. Pendientes de la Hoja 1 (obsoleta) que siguen vigentes
Ya están integrados en los puntos anteriores: citar VDI 2221/2225 (obs. 11), citar cada filtro (obs. 16), citar el consumo energético (obs. 19 y Discusión), llevar el diseño de la órtesis a la Fase 2 (obs. 15), agregar la tabla de ángulos de la rodilla (obs. 8), escribir un párrafo por figura de filtro (obs. 16) y reducir las tesis como referencia (obs. 4).

## 5. Nota sobre la tesis
Los errores C1–C3 y C7 también están en la tesis: Capítulo 4 (Tablas por filtro), Capítulo de Resultados (validación final "SVM") y Conclusiones. Si la tesis todavía admite correcciones o una fe de erratas, conviene aplicar las mismas correcciones.

# Laboratorio 6 — Análisis de redes sociales (YouTube)

**Universidad del Valle de Guatemala** · Facultad de Ingeniería
Departamento de Ciencias de la Computación · **CC3084 Data Science** · Semestre II 2026

Análisis de la estructura de participación de usuarios en YouTube a partir de dos conjuntos de
datos: 293 videos y 406 comentarios recolectados sobre contenido guatemalteco.

> **Estado actual: AVANCE — secciones 1 a 4 completas.**
> Secciones 5 a 10 pendientes para la entrega final.

---

## Contenido entregado en este avance

| Sección | Tema | Estado |
|---|---|---|
| 1 | Carga, comprensión e integración de los datos | ✅ |
| 2 | Calidad, limpieza y preprocesamiento | ✅ |
| 3 | Análisis exploratorio | ✅ |
| 4 | Construcción de la red bipartita autor–video | ✅ |
| 5 | Proyecciones de la red | ⏳ |
| 6 | Topología y fragmentación | ⏳ |
| 7 | Comunidades | ⏳ |
| 8 | Nodos centrales y participantes puente | ⏳ |
| 9 | Análisis de contenido y sentimiento | ⏳ |
| 10 | Interpretación, limitaciones y conclusiones | ⏳ |

---

## Estructura del repositorio

```
Laboratorio-6-Data-Science-/
├── youtube_videos.csv                       # Datos originales (293 x 20) — no se modifican
├── youtube_comments.csv                     # Datos originales (406 x 17) — no se modifican
│
├── Laboratorio_6_Avance_Secciones_1-4.ipynb # ★ ENTREGABLE: análisis, código y narrativa
│
├── outputs/
│   ├── tables/                              # 48 tablas de resultados en CSV
│   └── figures/                             # 20 figuras PNG (200 dpi) — no versionadas
│
├── data/processed/
│   ├── comentarios_limpios.csv              # Comentarios con texto_original / texto_limpio
│   ├── videos_limpios.csv                   # Videos con identificadores normalizados
│   ├── red_bipartita_autor_video.gexf       # Red para Gephi
│   └── red_bipartita_autor_video.graphml    # Red en GraphML
│
├── requirements.txt
└── README.md
```

### Qué se versiona y qué no

Todo lo anterior **se regenera íntegramente** al ejecutar el notebook. El criterio para versionarlo
o no es el siguiente:

| Ruta | ¿En el repo? | Motivo |
|---|---|---|
| `outputs/figures/` | **No** (en `.gitignore`) | Las 20 figuras ya están embebidas dentro del `.ipynb`; versionarlas duplicaría 4.5 MB sin aportar nada |
| `outputs/tables/` | Sí | Incluye la tabla de nodos y la de aristas, que son entregables evaluados (§4.3); ocupa 226 KB |
| `data/processed/` | Sí | Permite abrir la red en Gephi (`.gexf`) sin ejecutar nada; ocupa 916 KB |

> ⚠️ **El notebook debe subirse con sus salidas ejecutadas.** Contiene las figuras y tablas
> embebidas, de modo que se visualiza completo en GitHub sin ejecutarlo. Si se usa *Clear All
> Outputs* antes de un commit, se pierden también las figuras.

---

## Instalación

Requiere **Python 3.11** o superior.

```bash
pip install -r requirements.txt
```

Después de instalar, descargue una sola vez la lista de palabras vacías en español de NLTK:

```bash
python -c "import nltk; nltk.download('stopwords')"
```

---

## Cómo reproducir el análisis

**Ejecute siempre desde la raíz del repositorio**, que es donde están los archivos CSV.

```bash
jupyter lab Laboratorio_6_Avance_Secciones_1-4.ipynb
```

Luego *Run → Restart Kernel and Run All Cells*. Tarda unos 2–3 minutos y escribe en
`outputs/tables/`, `outputs/figures/` y `data/processed/`.

**Si más adelante se necesita un informe en PDF**, se genera desde el mismo notebook:

```bash
jupyter nbconvert --to html --no-input --embed-images --output Informe_Avance_Lab6 Laboratorio_6_Avance_Secciones_1-4.ipynb
```

El PDF se obtiene imprimiendo ese HTML desde el navegador (*Imprimir → Guardar como PDF*). Quitar
`--no-input` incluye además todo el código.

---

## Reproducibilidad

- **Semilla fija** (`SEED = 42`) para todos los *layouts* de red: las figuras son idénticas entre
  ejecuciones.
- **Los datos originales nunca se modifican.** Toda transformación crea columnas o archivos nuevos.
- **Codificación `utf-8-sig`** en todas las lecturas: los CSV originales llevan BOM y leerlos como
  `utf-8` simple corrompería el nombre de la primera columna y rompería los cruces por `video_id`.
- Cada tabla y figura del análisis tiene un nombre estable (`t<sección>_<tema>` y
  `f<sección>_<tema>`) que permite rastrearla hasta la celda que la generó.

---

## Decisiones metodológicas relevantes

Documentadas en detalle en el notebook; se resumen aquí las que condicionan la lectura de todos
los resultados.

### `reply_count` NO define aristas entre usuarios

Los datos indican cuántas respuestas recibió un comentario, pero **no quién las escribió**. Por
eso no existe —ni puede construirse— ninguna red autor–autor directa. La única relación de red
observable es **autor → comentó en → video**, que es la que sustenta la red bipartita.

### Cobertura parcial de los comentarios

Sólo **19 de los 293 videos (6.5 %)** tienen comentarios recolectados. Un video sin comentarios
en estos datos refleja **ausencia de datos**, no ausencia de participación. Los 105 videos
recuperados mediante la estrategia `official_gov` (36 % del catálogo) no aportaron ni un solo
comentario. Todos los análisis de participación y de red se restringen explícitamente al
subconjunto de 19 videos.

### Los identificadores nunca se sustituyen por nombres

`video_id`, `channel_id`, `comment_id` y `author_channel_id` se conservan como llaves en todo el
análisis. Los nombres y *handles* se normalizan sólo como etiquetas de visualización, porque son
mutables en YouTube.

### Tres versiones del texto

| Columna | Contenido | Uso |
|---|---|---|
| `texto_original` | Sin modificación alguna | Auditoría y análisis de sentimiento |
| `texto_norm` | Minúsculas, sin URL/menciones/emojis/puntuación/números | Inspección intermedia |
| `texto_limpio` | Lemas sin palabras vacías ni acentos | Frecuencias, bigramas, nubes |

Los emojis, URL, hashtags y menciones se **extraen a columnas propias** en lugar de descartarse.

### Lematización con `simplemma` en lugar de spaCy

Se evaluó primero spaCy con el modelo `es_core_news_sm`, pero su carga falla en el equipo de
desarrollo por una directiva de Control de aplicaciones de Windows que bloquea la extensión
compilada. `simplemma` es una alternativa en Python puro, basada en diccionario, que produce lemas
equivalentes para este vocabulario y garantiza que el notebook sea ejecutable por cualquier
integrante del grupo.

---

## Principales hallazgos del avance

1. **Integración perfecta, cobertura muy parcial.** Los 406 comentarios cruzan al 100 % con el
   archivo de videos, pero sólo alcanzan a 19 de 293 videos.

2. **Los datos son limpios pero desbalanceados.** Sin duplicados ni identificadores inconsistentes;
   las correspondencias ID ↔ nombre ↔ handle son biunívocas. Los problemas reales son de
   *cobertura* y *diseño de muestreo*, no de calidad de registro.

3. **La participación se concentra en videos y canales, pero se dispersa entre autores.** Un video
   reúne el 39.7 % de los comentarios y un canal el 63.1 % (Gini 0.66), mientras que los autores
   están poco concentrados (Gini 0.16): la mayoría escribe un único comentario.

4. **Visibilidad y participación coinciden en el orden, no en la intensidad.** ρ de Spearman = 0.81
   entre visualizaciones y comentarios, pero la tasa de participación varía en un factor de 240×
   entre videos.

5. **La red bipartita es dispersa y fragmentada.** 351 nodos, 343 aristas, densidad bipartita
   0.054, 10 componentes conexas, componente mayor con el 81.5 % de los nodos. **323 de 332 autores
   (97.3 %) participan en un solo video.**

6. **Sólo 9 autores conectan la red.** Cuatro de ellos cruzan fronteras de canal, pero los nueve —
   sin excepción — pasan por Quorum o por el Gobierno, los dos emisores más intensamente
   muestreados: la conectividad observada depende en gran medida del diseño de recolección.

---

## Limitaciones

Ningún resultado puede generalizarse a "los usuarios de YouTube" ni a "la población de Guatemala".
La muestra está dominada por un canal de periodismo político (63 % de los comentarios). Los
conteos de visualizaciones y "me gusta" son una fotografía del instante de recolección — algo
verificado empíricamente en el notebook, donde `view_count` y `view_count_text` difieren
ligeramente por haberse capturado en momentos distintos. Las fechas de los comentarios son
relativas ("hace 1 año") y no admiten conversión a fecha exacta.

---

## Datos

| Archivo | Filas | Columnas | Llave primaria |
|---|---:|---:|---|
| `youtube_videos.csv` | 293 | 20 | `video_id` |
| `youtube_comments.csv` | 406 | 17 | `comment_id` |

Relación entre archivos: `youtube_comments.video_id → youtube_videos.video_id` (integridad
referencial verificada al 100 %).

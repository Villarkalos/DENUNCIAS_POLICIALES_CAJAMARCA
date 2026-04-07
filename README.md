# 🔍 Análisis de Denuncias Policiales — Cajamarca & Norte del Perú (2018–2026)

> **Herramientas:** Python · Pandas · GeoPandas · Matplotlib · Seaborn · imageio · Power BI · DAX  
> **Datos:** Dataset oficial de denuncias policiales del Perú (Ene 2018 – Feb 2026)

---

## 📌 ¿De qué trata este proyecto?

Este repositorio contiene **dos análisis complementarios** sobre el mismo dataset de denuncias policiales:

| Proyecto | Alcance | Herramienta | Archivo |
|---|---|---|---|
| **EDA — Cajamarca** | Solo la región de Cajamarca, enfoque exploratorio profundo | Python (Jupyter) | `Final.ipynb` |
| **Dashboard — Norte del Perú** | Cajamarca + La Libertad + Lambayeque + vista nacional | Power BI | `BI_PERU_DENUNCIAS.pbix` |

La lógica del proyecto: **primero explorar en profundidad la región local → luego escalar el análisis comparativo a nivel macro.**

---

## 📁 Estructura del repositorio

```
📦 denuncias-peru-analysis/
├── 📓 Final.ipynb                    # EDA completo en Python — Cajamarca
├── 📊 BI_PERU_DENUNCIAS.pbix         # Dashboard Power BI — Norte del Perú
├── 📂 graficos/
│   ├── 01_evolucion_total.png
│   ├── 02_top10_modalidades.png
│   └── 03_heatmap_provincias.png
├── 📂 gif/
│   ├── Total_Denuncias_Cajamarca.gif
│   ├── Robos_Cajamarca.gif
│   ├── Extorsion_Cajamarca.gif
│   └── Mapa_Geo_Total_Cajamarca.gif
└── 📄 README.md
```

---

## 🐍 PROYECTO 1 — EDA en Python: Cajamarca

### ¿Qué analiza?
Exploración profunda de las denuncias policiales **únicamente en Cajamarca** — a nivel de 13 provincias, modalidades delictivas y evolución 2018–2026.

**Dataset:** 15,814 registros · Columnas: AÑO, DEPARTAMENTO, PROVINCIA, MODALIDAD, CANTIDAD

---

### Paso a paso

#### Paso 1 — Instalar dependencias
```bash
pip install geopandas imageio[ffmpeg]
```
Librerías del proyecto: `pandas`, `numpy`, `matplotlib`, `seaborn`, `geopandas`, `imageio`

#### Paso 2 — Configuración visual
```python
BG_COLOR   = '#0d1b2a'   # fondo oscuro marino
TEXT_COLOR = 'white'
LINE_COLOR = '#00b4d8'   # cyan para líneas
```
Paleta oscura consistente en todos los gráficos.

#### Paso 3 — Carga y limpieza del dataset
```python
df = pd.read_csv('DATASET_DENUNCIAS.csv', encoding='latin1', sep=None, engine='python')
df.columns = df.columns.str.strip().str.upper()
# Renombrado de columnas, corrección de tildes (Ã³→ó), casteo numérico
```
Salida: `✅ Dataset procesado: 15,814 registros.`

#### Paso 4 — Evolución histórica (gráfico de línea + área)
- `df.groupby('ANIO')['cantidad'].sum()` → serie temporal
- `ax.fill_between()` para el área bajo la curva
- Anotaciones de valores en cada punto
- Guardado: `graficos/01_evolucion_total.png`

#### Paso 5 — Top 10 modalidades delictivas (barras horizontales)
- Agrupación por `P_MODALIDADES`, suma total histórica
- Gradiente de color `YlOrRd` de menor a mayor intensidad
- Guardado: `graficos/02_top10_modalidades.png`

#### Paso 6 — Heatmap de provincias × año
- Pivot table: filas = provincias, columnas = años (2018–2026)
- `sns.heatmap()` con cmap `YlOrRd` y anotaciones numéricas
- Identifica qué provincia concentra más casos en cada período
- Guardado: `graficos/03_heatmap_provincias.png`

#### Paso 7 — Bar chart race animado (GIF por ranking de provincias)
```python
def crear_gif_modalidad(modalidad_filtro, nombre_gif, top_n=10):
    # Genera un frame PNG por cada año
    # Paleta COLOR_MAP con tab20 — color consistente por provincia
    # xlim fijo al máximo histórico para comparabilidad
    # Concatena frames en GIF a 1.2 fps con imageio
```
GIFs generados:
- `Total_Denuncias_Cajamarca.gif` → todas las modalidades
- `Robos_Cajamarca.gif` → solo robos
- `Extorsion_Cajamarca.gif` → solo extorsión

#### Paso 8 — Mapa geográfico coroplético animado
```python
url_geojson = "https://raw.githubusercontent.com/juaneladio/peru-geojson/master/peru_provincial_simple.geojson"
mapa_peru = gpd.read_file(url_geojson)
# Normalización unicode para unir nombres de provincia del CSV con el GeoJSON
# vmax fijo al máximo histórico → escala de color comparable entre años
```
GIFs por modalidad: Total · Robo · Estafa · Extorsión

---

## 📊 PROYECTO 2 — Dashboard Power BI: Norte del Perú

### ¿Qué analiza?
Un dashboard interactivo que cruza **Cajamarca, La Libertad y Lambayeque** dentro del contexto nacional, sobre **6,966,237 denuncias** registradas en todo el Perú (2018–2026).

---

### Datos reales por departamento (total histórico 2018–2026)

| Departamento | **Total** | Violencia | Hurto | Robo | Estafa | Extorsión | Homicidio |
|---|---|---|---|---|---|---|---|
| **Cajamarca** | **171,501** | 47,180 | 37,733 | 7,400 | 3,377 | 1,405 | 312 |
| **La Libertad** | **364,034** | 77,945 | 72,896 | 45,199 | 11,448 | 19,250 | 1,338 |
| **Lambayeque** | **407,663** | 82,773 | 103,649 | 43,420 | 11,225 | 3,899 | 357 |

### Evolución año a año

| Año | Cajamarca | La Libertad | Lambayeque |
|---|---|---|---|
| 2018 | 16,181 | 36,256 | 43,694 |
| 2019 | 19,109 | 43,650 | 43,250 |
| 2020 | 17,561 | 32,048 | 32,372 |
| 2021 | 20,644 | 36,249 | 40,814 |
| 2022 | 22,522 | 47,433 | 51,978 |
| 2023 | 25,551 | 56,880 | 62,767 |
| 2024 | 25,697 | 57,479 | 62,989 |
| 2025 | 21,386 | 47,946 | 60,459 |

> 📌 La caída de 2020 en los tres departamentos refleja el efecto del confinamiento por la pandemia.

---

### Paso a paso Power BI

#### Paso 9 — Modelo de datos (esquema estrella)
- **Tabla de hechos:** `DATASET_Denuncias_Policiales_Ene 2018 a Feb 2026`
- **Dimensiones:** `Calendario` · `Ubicación` · `Modalidad` · `geodir-ubigeo-inei`
- Relaciones por claves de departamento / provincia / distrito / año-mes

#### Paso 10 — Medidas DAX (37 medidas en 5 carpetas)

| Carpeta | Medidas clave |
|---|---|
| 📊 Base | Total Denuncias · Promedio Mensual · % del Total General |
| 📈 Variación Anual | Denuncias Año Anterior · Variación Absoluta · Variación % Anual* |
| 🚨 Por Modalidad | Violencia · Robo · Hurto · Extorsión · Homicidio · Estafa |
| 🗺️ Mapa | Denuncias por Provincia · Categoría Mapa (4 rangos de color) |
| 🃏 HTML Cards | KPIs dinámicos con HTML renderer para cada región y comparativa |

*La variación % es "inteligente": si hay filtro de año activo, lo usa directamente; si no, compara el año más reciente con +6 meses de datos vs el año anterior completo.

#### Paso 11 — Páginas del dashboard
1. **Perú General** — KPI nacional con tendencia 2018–2026 y top departamento
2. **Cajamarca** — total, variación anual, top delitos, mapa provincial coroplético
3. **La Libertad** — análisis con alerta de extorsión (la más alta del norte)
4. **Lambayeque** — lidera en hurto y en volumen total regional
5. **Comparativa** — los 3 departamentos lado a lado + conclusiones ejecutivas

---

## 🔑 Hallazgos clave

- **Lambayeque** lidera el norte con **407,663 denuncias** totales y el mayor volumen de hurtos (103,649)
- **La Libertad** tiene la extorsión más alta de los tres: **19,250 casos** (5.3% de sus denuncias)
- **Cajamarca** es la más pequeña en volumen pero la **violencia contra la mujer** representa el **27.5%** de sus denuncias
- El año **2020 cayó en los tres** departamentos por el confinamiento
- Tendencia creciente 2021–2024, con descenso en 2025 (datos aún parciales en 2026)

---

## 🚀 Cómo ejecutar el notebook

```bash
git clone https://github.com/TU_USUARIO/denuncias-peru-analysis.git
cd denuncias-peru-analysis
pip install pandas numpy matplotlib seaborn geopandas imageio[ffmpeg]
```
Coloca `DATASET_DENUNCIAS.csv` en la raíz y ejecuta `Final.ipynb` en Jupyter o Google Colab.

---

## 🗃️ Fuente de datos
Dataset oficial de denuncias policiales — [datos.gob.pe](https://datos.gob.pe) · Ministerio del Interior del Perú.

---

## 👤 Autor
Desarrollado desde **Cajamarca, Perú** 🇵🇪 — porque los datos de tu región también merecen ser analizados.

*"No basta con que los datos existan — alguien tiene que analizarlos."*

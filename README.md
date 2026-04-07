# 🔍 Análisis de Denuncias Policiales — Cajamarca & Norte del Perú (2018–2026)

> **Fuente de datos:** [Denuncias Policiales — MININTER · datos.gob.pe](https://www.datosabiertos.gob.pe/dataset/denuncias-policiales-1)  
> **Licencia del dataset:** Open Data Commons Attribution License  
> **Herramientas:** Python · Pandas · GeoPandas · Matplotlib · Seaborn · imageio · Power BI · DAX

---

## 📌 ¿De qué trata este proyecto?

Este repositorio contiene **dos análisis complementarios** sobre el mismo dataset oficial de denuncias policiales del Perú:

| Proyecto | Alcance | Herramienta | Archivo |
|---|---|---|---|
| **EDA — Cajamarca** | Análisis exploratorio profundo solo de la región Cajamarca | Python (Jupyter) | `Final.ipynb` |
| **Dashboard — Norte del Perú** | Cajamarca + La Libertad + Lambayeque + vista nacional | Power BI | `BI PERU DENUNCIAS.pbix` |

---

## 📁 Estructura del repositorio

```
📦 denuncias-peru-analysis/
│
├── 📂 IMG/
│   ├── 📂 GIF/
│   │   ├── Total_Denuncias_Cajamarca.gif
│   │   ├── Robos_Cajamarca.gif
│   │   ├── Extorsion_Cajamarca.gif
│   │   ├── Mapa_Geo_Total_Cajamarca.gif
│   │   ├── Mapa_Geo_Robo_Cajamarca.gif
│   │   ├── Mapa_Geo_Estafa_Cajamarca.gif
│   │   └── Mapa_Geo_Extorsión_Cajamarca.gif
│   │
│   └── 📂 gráficos/
│       ├── 01_evolucion_total.png
│       ├── 02_top10_modalidades.png
│       └── 03_heatmap_provincias.png
│
├── 📂 PERÚ_DB/
│   └── DICCIONARIO_DATOS_Denuncias.xlsx
│
├── 📊 BI PERU DENUNCIAS.pbix
├── 📄 DATASET_DENUNCIAS.csv          ← dataset filtrado solo Cajamarca (usado en el notebook)
├── 📄 PERU.csv                       ← dataset nacional completo (usado en Power BI)
├── 📓 Final.ipynb
└── 📄 README.md
```

---

## 🐍 PROYECTO 1 — EDA en Python: Cajamarca

### Dataset
**`DATASET_DENUNCIAS.csv`** — registros de denuncias filtrados para la región Cajamarca.  
Columnas del CSV: `AÑO`, `PROVINCIA`, `MODALIDAD`, `CANTIDAD`  
Resultado tras limpieza: **15,814 registros**.

---

### Paso 1 — Instalación de dependencias

```bash
pip install geopandas imageio[ffmpeg]
```

Librerías importadas en el notebook:
```python
import pandas as pd
import numpy as np
import os
import matplotlib.pyplot as plt
import matplotlib.ticker as mticker
import seaborn as sns
import imageio.v2 as imageio
import geopandas as gpd
from IPython.display import Image, display
```

Configuración global de estilo:
```python
BG_COLOR   = '#0d1b2a'
TEXT_COLOR = 'white'
LINE_COLOR = '#00b4d8'

plt.rcParams['font.family'] = 'DejaVu Sans'
plt.rcParams['figure.dpi'] = 120
```

---

### Paso 2 — Carga y limpieza del dataset

```python
df_raw = pd.read_csv('DATASET_DENUNCIAS.csv', encoding='latin1', sep=None, engine='python')
df_raw.columns = df_raw.columns.str.strip().str.upper()

rename_map = {
    'AÑO': 'ANIO',
    'PROVINCIA': 'PROV_HECHO',
    'MODALIDAD': 'P_MODALIDADES',
    'CANTIDAD': 'cantidad'
}
df = df_raw.rename(columns=rename_map).copy()

# Normalización de texto y corrección de encoding
df['P_MODALIDADES'] = df['P_MODALIDADES'].astype(str).str.strip().str.title()
df['P_MODALIDADES'] = df['P_MODALIDADES'].str.replace('Ã³', 'ó').str.replace('Ã', 'í')
df['PROV_HECHO']    = df['PROV_HECHO'].astype(str).str.strip().str.title()

# Casteo numérico con manejo de nulos
df['ANIO']     = pd.to_numeric(df['ANIO'],     errors='coerce').fillna(0).astype(int)
df['cantidad'] = pd.to_numeric(df['cantidad'], errors='coerce').fillna(0).astype(int)
```
Salida: `✅ Dataset procesado: 15,814 registros.`

---

### Paso 3 — Gráfico de evolución histórica → `IMG/gráficos/01_evolucion_total.png`

```python
por_anio = df.groupby('ANIO')['cantidad'].sum().reset_index()

ax.fill_between(por_anio['ANIO'], por_anio['cantidad'], alpha=0.25, color=LINE_COLOR)
ax.plot(por_anio['ANIO'], por_anio['cantidad'], color=LINE_COLOR, linewidth=2.5, marker='o')

# Anotación del valor en cada punto
for x, y in zip(por_anio['ANIO'], por_anio['cantidad']):
    ax.annotate(f'{y:,}', (x, y), textcoords='offset points', xytext=(0, 10), ...)

plt.savefig('graficos/01_evolucion_total.png', dpi=150, bbox_inches='tight', facecolor=BG_COLOR)
```

---

### Paso 4 — Top 10 modalidades delictivas → `IMG/gráficos/02_top10_modalidades.png`

```python
top10 = df.groupby('P_MODALIDADES')['cantidad'].sum().sort_values().tail(10)

bars = ax.barh(top10.index, top10.values,
               color=plt.cm.YlOrRd(np.linspace(0.4, 1.0, 10)), height=0.65)

plt.savefig('graficos/02_top10_modalidades.png', dpi=150, bbox_inches='tight', facecolor=BG_COLOR)
```

---

### Paso 5 — Heatmap de provincias × año → `IMG/gráficos/03_heatmap_provincias.png`

```python
pivot_prov = (df.groupby(['PROV_HECHO', 'ANIO'])['cantidad']
                .sum().reset_index()
                .pivot(index='PROV_HECHO', columns='ANIO', values='cantidad')
                .fillna(0))

# Ordenado por total descendente
pivot_prov = pivot_prov.loc[pivot_prov.sum(axis=1).sort_values(ascending=False).index]

sns.heatmap(pivot_prov, cmap='YlOrRd', annot=True, fmt='.0f',
            linewidths=0.5, linecolor=BG_COLOR, ax=ax)

plt.savefig('graficos/03_heatmap_provincias.png', dpi=150, bbox_inches='tight', facecolor=BG_COLOR)
```

---

### Paso 6 — Bar chart race animado → `IMG/GIF/`

Función `crear_gif_modalidad(modalidad_filtro, nombre_gif, top_n=10)`:
- Calcula el `xlim` máximo histórico **antes** de generar los frames, para mantener escala fija
- Genera un frame PNG por año con barras horizontales de las top 10 provincias
- Asigna color fijo por provincia con `COLOR_MAP` (paleta `tab20`)
- Imprime el año como marca de agua semitransparente en cada frame
- Concatena los frames en GIF a **1.2 fps** con `imageio.mimwrite`
- Elimina los frames temporales tras generar el GIF

GIFs generados:
```python
crear_gif_modalidad(None,      'Total_Denuncias_Cajamarca')  # todas las modalidades
crear_gif_modalidad('Robo',    'Robos_Cajamarca')
crear_gif_modalidad('Extorsi', 'Extorsion_Cajamarca')
```

---

### Paso 7 — Mapa coroplético geográfico animado → `IMG/GIF/`

```python
url_geojson = "https://raw.githubusercontent.com/juaneladio/peru-geojson/master/peru_provincial_simple.geojson"
mapa_peru = gpd.read_file(url_geojson)
```

- Filtra el GeoJSON a las provincias de Cajamarca usando la columna `DEPARTAMENTO` del CSV
- Normalización unicode (`unicodedata.normalize('NFKD', ...)`) para cruzar nombres de provincia entre el CSV y el GeoJSON
- `vmax` fijo al máximo histórico para que la escala de color sea comparable entre años
- Un frame por año → GIF a 1.2 fps

GIFs generados:
```python
crear_gif_mapa_geografico(None,        'Mapa_Geo_Total_Cajamarca')
crear_gif_mapa_geografico('Robo',      'Mapa_Geo_Robo_Cajamarca')
crear_gif_mapa_geografico('Estafa',    'Mapa_Geo_Estafa_Cajamarca')
crear_gif_mapa_geografico('Extorsión', 'Mapa_Geo_Extorsión_Cajamarca')
```

---

## 📊 PROYECTO 2 — Dashboard Power BI: Norte del Perú

### Dataset
**`PERU.csv`** — dataset nacional completo de denuncias policiales (Ene 2018 – Feb 2026), fuente: [MININTER · datos.gob.pe](https://www.datosabiertos.gob.pe/dataset/denuncias-policiales-1).

---

### Modelo de datos

**Tabla de hechos:**
`DATASET_Denuncias_Policiales_Ene 2018 a Feb 2026`

| Columna | Tipo |
|---|---|
| AÑO | Int64 |
| MES | Int64 |
| DEPARTAMENTO | String |
| PROVINCIA | String |
| DISTRITO | String |
| UBIGEO_HECHO | Int64 |
| MODALIDAD | String |
| CANTIDAD | Int64 |
| Fecha *(calculada)* | DateTime — combina AÑO + MES para relacionar con el Calendario |

**Tablas de dimensión:**

`Calendario` — Fecha · Año · Mes · Nombre Mes · Mes Corto · Trimestre · Semestre · Año-Mes  
`Ubicación` — Ubigeo · Departamento · Provincia · Distrito · Región · Ubicacion Completa *(calculada)*  
`Modalidad` — ID Modalidad · Modalidad · Grupo Delito · Prioridad  
`geodir-ubigeo-inei` — tabla auxiliar de ubigeos para el mapa

---

### Medidas DAX (37 en total)

**📊 Carpeta Base**
- `Total Denuncias` — suma total en el contexto seleccionado
- `Promedio Mensual` — promedio de denuncias por mes en el período
- `% del Total General` — participación porcentual sobre el total absoluto del dataset

**📈 Carpeta Variación Anual**
- `Denuncias Año Anterior` — total del año inmediatamente anterior al seleccionado
- `Variación Absoluta` — diferencia absoluta vs año anterior
- `Variación % Anual` — variación inteligente: si hay filtro de año activo lo usa; si no, compara el año más reciente con más de 6 meses de datos vs el anterior (evita comparar años parciales con completos)

**🚨 Carpeta Por Modalidad**
- `Denuncias Violencia` · `Denuncias Robo` · `Denuncias Hurto`
- `Denuncias Extorsión` · `Denuncias Homicidio` · `Denuncias Estafa`

**🗺️ Carpeta Mapa**
- `Denuncias por Provincia (mapa)` — agrupada por provincia, no afectada por filtro de distrito
- `Categoria Mapa` — categoría de color para mapa coroplético con 4 rangos visuales

**🃏 Carpeta HTML Cards** (visuales renderizados con HTML Viewer)
- KPI General, Tendencia, Top Delitos para: Perú · Cajamarca · La Libertad · Lambayeque
- KPI Comparativa con los 3 departamentos lado a lado
- Página de conclusiones ejecutivas a página completa
- Portada

**Medidas de navegación/slicer:** `Departamento.` · `Provincia.` · `Distrito.`

---

### Páginas del dashboard

1. **Perú General** — resumen nacional con KPI principal, tendencia 2018–2026 y top departamento
2. **Cajamarca** — total, variación anual dinámica, top delitos, mapa coroplético provincial
3. **La Libertad** — análisis con distribución por tipo de delito y tendencia anual
4. **Lambayeque** — distribución por tipo de delito y tendencia anual
5. **Comparativa** — los 3 departamentos lado a lado con evolución año a año e insights
6. **Conclusiones** — página ejecutiva a pantalla completa

---

### Datos reales por departamento (total histórico 2018–2026)

| Departamento | **Total** | Violencia | Hurto | Robo | Estafa | Extorsión | Homicidio |
|---|---|---|---|---|---|---|---|
| **Cajamarca** | **171,501** | 47,180 | 37,733 | 7,400 | 3,377 | 1,405 | 312 |
| **La Libertad** | **364,034** | 77,945 | 72,896 | 45,199 | 11,448 | 19,250 | 1,338 |
| **Lambayeque** | **407,663** | 82,773 | 103,649 | 43,420 | 11,225 | 3,899 | 357 |

---

## 🚀 Cómo ejecutar el notebook

```bash
git clone https://github.com/TU_USUARIO/denuncias-peru-analysis.git
cd denuncias-peru-analysis
pip install pandas numpy matplotlib seaborn geopandas imageio[ffmpeg]
```

Asegúrate de que `DATASET_DENUNCIAS.csv` esté en la raíz del proyecto, luego abre `Final.ipynb` en Jupyter o Google Colab.

---

## 🗃️ Fuente de datos

[Denuncias Policiales — Plataforma Nacional de Datos Abiertos](https://www.datosabiertos.gob.pe/dataset/denuncias-policiales-1)  
Ministerio del Interior del Perú (MININTER) · Licencia: Open Data Commons Attribution License

---

## 👤 Autor

Desarrollado por **Richard Villar** 🇵🇪

*"No basta con que los datos existan — alguien tiene que analizarlos."*

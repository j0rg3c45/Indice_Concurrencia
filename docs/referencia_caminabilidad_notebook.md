# Referencia Técnica — Notebook de Caminabilidad Roosevelt v2

**Notebook:** `notebooks_py/caminabilidad_roosevelt_v2.ipynb`  
**Equipo:** ITT Cali Inteligente · Gobierno de Datos  
**Fecha de documentación:** Mayo 2026

---

## 1. Objetivo del Notebook

Calcular las métricas de caminabilidad para el área de influencia de la intervención en Av. Roosevelt, usando la red peatonal de OpenStreetMap. Incluye comparación con territorios espejo (Calle 5 y Calle 7).

---

## 2. Carga de Datos desde Repositorios Git

### 2.1 Repositorio principal

| Campo | Valor |
|-------|-------|
| URL | `https://github.com/j0rg3c45/indice-caminabilidad-roosevelt.git` |
| Nombre local | `indice-caminabilidad-roosevelt` |
| Detección de entorno | Si está en `/content` (Colab), clona el repo. Si es local, usa `Path(os.getcwd()).parent` |

### 2.2 Estructura de datos dentro del repo

```
data/
├── itt_roosevelt/
│   └── Roosevelt/
│       └── Geojson_Roosevelt/          ← DATA_DIR principal
│           ├── Geojson_tramos_Roosevelt_Buffer_100.geojson
│           ├── Geojson_tramos_Roosevelt.geojson
│           ├── BD_SINIESTROS_2023_2025_COMUNA_BARRIO_4326_Roosevelt.geojson
│           ├── COMPARENDOS_2023_2025_Roosevelt.geojson
│           ├── HOMICIDIOS_2023_2025_Roosevelt.geojson
│           ├── HURTOS_2023_2025_Roosevelt.geojson
│           ├── Sedes_educativas_oficiales_Roosevelt.geojson
│           ├── VBG_2025_Roosevelt.geojson
│           └── VIOLENCIA_INTRAFAMILIAR_2023_2025_Roosevelt.geojson
└── Informacion_espejo/
    ├── geojson_espejo_poligonos.zip     ← Se descomprime automáticamente
    └── geojson_espejo_poligonos/
        ├── calle_5_area_Espejo_Bf100.geojson
        └── calle_7_area_Espejo_Bf100.geojson
```

### 2.3 Método de búsqueda de archivos

Se usa una función `find_file(base_dir, pattern)` con regex para localizar archivos GeoJSON de forma flexible (case-insensitive), lo que permite tolerancia a variaciones en nombres de archivo.

### 2.4 Diccionario de rutas (PATHS)

```python
PATHS = {
    'poligono_buffer': find_file(DATA_DIR, r'Geojson_tramos_Roosevelt_Buffer_100'),
    'poligono_tramos': find_file(DATA_DIR, r'Geojson_tramos_Roosevelt\.geojson'),
    'siniestros':      find_file(DATA_DIR, r'BD_SINIESTROS'),
    'comparendos':     find_file(DATA_DIR, r'COMPARENDOS'),
    'homicidios':      find_file(DATA_DIR, r'HOMICIDIOS'),
    'hurtos':          find_file(DATA_DIR, r'HURTOS'),
    'sedes':           find_file(DATA_DIR, r'Sedes_educativas'),
    'vbg':             find_file(DATA_DIR, r'VBG'),
    'vif':             find_file(DATA_DIR, r'VIOLENCIA_INTRAFAMILIAR'),
}
```

---

## 3. Transformación de Datos y Mapeo Geográfico

### 3.1 Sistema de Referencia de Coordenadas (CRS)

| CRS | Uso |
|-----|-----|
| **EPSG:4326** (WGS84) | Sistema de trabajo para toda la información geoespacial, visualización y OSMnx |
| **EPSG:3116** (Colombia) | Solo para cálculos de área y distancia (proyectado, unidades en metros) |

### 3.2 Normalización de CRS en carga

Todos los datasets se normalizan a WGS84 al cargar:

```python
if gdf.crs is None:
    gdf = gdf.set_crs('EPSG:4326')
elif gdf.crs.to_epsg() != 4326:
    gdf = gdf.to_crs('EPSG:4326')
```

### 3.3 Polígono de intervención

- **Fuente:** `Geojson_tramos_Roosevelt_Buffer_100.geojson`
- **Tipo:** Buffer de 100m a cada lado del eje vial
- **Área:** 429,014 m² (42.90 ha)
- **Longitud del corredor:** ~1,642 m
- Se carga en WGS84 y se proyecta a EPSG:3116 solo para calcular área

### 3.4 Red peatonal desde OpenStreetMap

- **Librería:** OSMnx (`ox.graph_from_polygon`)
- **Tipo de red:** `walk` (peatonal)
- **Input:** Polígono en WGS84 (requerido por OSMnx)
- **Output:** Grafo NetworkX con nodos y aristas georreferenciados
- **Exportación:** Se guarda como GeoJSON en la carpeta de datos

```python
G = ox.graph_from_polygon(polygon_wgs84, network_type='walk')
```

### 3.5 Cálculo de métricas de caminabilidad

Se usa `ox.basic_stats(G, area=area_m2)` con el área en m² (EPSG:3116) para obtener:

| Métrica | Valor línea base | Descripción |
|---------|-----------------|-------------|
| Intersecciones peatonales | 232 | Puntos donde se cruzan caminos peatonales |
| Longitud red peatonal | 31.78 km | Total de segmentos peatonales |
| Longitud promedio segmento | 41.7 m | Promedio de largo de cada segmento |
| Densidad de calle | 74.07 km/km² | Longitud de red por unidad de área |
| Nodos OSM | 246 | Total de nodos en el grafo |
| Segmentos OSM | 762 | Total de aristas en el grafo |

### 3.6 Indicadores complementarios (densidad de eventos)

Se calculan densidades por hectárea para cada dataset:

```python
densidad = n_registros / (area_m2 / 10000)  # registros por hectárea
```

| Dataset | Registros | Densidad (por ha) |
|---------|-----------|-------------------|
| Siniestros | 72 | 1.68 |
| Comparendos | 1,300 | 30.30 |
| Homicidios | 5 | 0.12 |
| Hurtos | 730 | 17.02 |
| Sedes educativas | 2 | 0.05 |
| VBG | 4 | 0.09 |
| VIF | 24 | 0.56 |

---

## 4. Territorios Espejo

### 4.1 Zonas de comparación

| Zona | Polígono | Área |
|------|----------|------|
| Calle 5 | `calle_5_area_Espejo_Bf100.geojson` | 270,543 m² (27.05 ha) |
| Calle 7 | `calle_7_area_Espejo_Bf100.geojson` | 372,785 m² (37.28 ha) |

### 4.2 Métricas de caminabilidad por zona espejo

| Métrica | Roosevelt | Calle 5 | Calle 7 |
|---------|-----------|---------|---------|
| Nodos | 246 | 298 | 408 |
| Segmentos | 762 | 740 | 1,094 |
| Intersecciones | 232 | 288 | 395 |
| Longitud red (km) | 31.78 | 23.33 | 32.64 |
| Long. promedio segmento (m) | 41.7 | 31.5 | 29.8 |
| Densidad calle (km/km²) | 74.07 | 86.25 | 87.56 |
| Densidad intersecciones (int/km²) | ~541 | 1,065 | 1,060 |

---

## 5. Mapas Implementados

### 5.1 Mapa estático de red peatonal (Celda 8)

- **Librería:** OSMnx (`ox.plot_graph`)
- **Contenido:** Red peatonal con nodos y aristas
- **Estilo:** Fondo oscuro, nodos rojos, aristas verdes
- **Exportación:** PNG en `outputs/figures/`

### 5.2 Mapa interactivo con capas (Celda 9)

- **Librería:** Folium
- **Capas base disponibles:**
  - CartoDB Claro (positron)
  - OpenStreetMap
  - Esri Satélite
  - Google Streets
  - Google Satélite
- **Capas de datos (overlays):**
  - Polígono de intervención (Buffer 100m) — azul/verde
  - Tramos Roosevelt (eje) — rojo
  - Red peatonal OSM (líneas) — verde azulado
  - Nodos/intersecciones — rojo
  - Vértices de líneas — gris oscuro
  - Siniestros, Comparendos, Homicidios, Hurtos, Sedes, VBG, VIF — colores diferenciados
  - Polígonos de zonas espejo (Calle 5 y Calle 7) — rojo y azul con borde punteado
- **Control de capas:** `folium.LayerControl()` para activar/desactivar

### 5.3 Mapa de red peatonal alta resolución (Gráfico 1)

- Versión de alta resolución del mapa estático
- Incluye nodos, aristas y conectividad
- Exportado a PNG con DPI alto

### 5.4 Mapas estáticos comparativos (Celda 15)

- Subplots con la red peatonal de Roosevelt + cada zona espejo
- Permite comparación visual de la morfología de la red

---

## 6. Tablas Implementadas

### 6.1 Tabla de métricas de caminabilidad (Celda 6)

Métricas principales de la red peatonal de Roosevelt.

### 6.2 Tabla de indicadores complementarios (Celda 10)

Conteo y densidad por hectárea de cada dataset de eventos.

### 6.3 Tabla comparativa Roosevelt vs Espejo (Celda 10B)

DataFrame con densidad por hectárea comparando las tres zonas.

### 6.4 Tabla resumen de métricas urbanas (Gráfico 6)

Tabla consolidada con todas las métricas en formato visual (matplotlib table).

### 6.5 Tabla de comparación final (Celda 16)

Comparación completa de métricas de caminabilidad entre Roosevelt y territorios espejo.

---

## 7. Gráficos Implementados

### 7.1 Gráfico comparativo de indicadores (Celda 10B)

- **Tipo:** Barras agrupadas (2 subplots)
- **Subplot 1:** Total de registros por dataset y zona
- **Subplot 2:** Densidad por hectárea por dataset y zona
- **Colores:** Verde (Roosevelt), Rojo (Calle 5), Azul (Calle 7)

### 7.2 Heatmap de densidad de intersecciones (Gráfico 2)

- **Tipo:** Kernel Density Estimation (KDE)
- **Librería:** `scipy.stats.gaussian_kde`
- **Contenido:** Mapa de calor de concentración de intersecciones peatonales
- **Uso:** Identificar zonas de alta/baja conectividad

### 7.3 Histograma de longitud de segmentos (Gráfico 3)

- **Tipo:** Histograma
- **Variable:** Longitud de cada segmento peatonal
- **Uso:** Distribución de tamaños de manzana/cuadra

### 7.4 Boxplot de segmentos peatonales (Gráfico 4)

- **Tipo:** Boxplot
- **Variable:** Longitud de segmentos
- **Uso:** Identificar outliers y dispersión

### 7.5 Scatter plot de conectividad (Gráfico 5)

- **Tipo:** Scatter plot
- **Ejes:** Longitud promedio de segmento vs Densidad de intersecciones
- **Uso:** Posicionar morfológicamente a Roosevelt respecto a benchmarks

### 7.6 Gráfico comparativo de caminabilidad por zona (Celda 14B)

- **Tipo:** Barras agrupadas
- **Contenido:** Métricas de caminabilidad para Roosevelt + zonas espejo
- **Métricas:** Intersecciones, longitud red, densidad calle, densidad intersecciones

### 7.7 Scatter plot comparativo de conectividad (Celda 16)

- **Tipo:** Scatter plot
- **Contenido:** Roosevelt vs zonas espejo en el espacio longitud-densidad
- **Uso:** Comparar posición morfológica entre zonas

---

## 8. Exportación de Resultados

### 8.1 CSV de línea base (Celda 11)

- **Archivo:** `outputs/results/roosevelt_caminabilidad_linea_base.csv`
- **Contenido:** Todas las métricas de caminabilidad + indicadores complementarios
- **Metadatos incluidos:** fecha_medicion, poligono, crs_trabajo, crs_calculo, fuente, momento

### 8.2 CSV comparativo (Celda 17)

- **Archivo:** `outputs/results/comparacion_roosevelt_vs_espejo.csv`
- **Contenido:** Tabla comparativa de métricas entre Roosevelt y territorios espejo

### 8.3 Red peatonal GeoJSON

- **Archivo:** `data/itt_roosevelt/Roosevelt/Geojson_Roosevelt/red_peatonal_osm_roosevelt.geojson`
- **Contenido:** Aristas de la red peatonal descargada de OSM

### 8.4 Figuras PNG

- **Directorio:** `outputs/figures/`
- **Archivos:** Todos los gráficos estáticos exportados con DPI alto

---

## 9. Dependencias

```
osmnx
geopandas
matplotlib
pandas
numpy
folium
seaborn
scipy
```

---

## 10. Notas para Replicación

1. El notebook detecta automáticamente si está en Colab o local
2. En Colab, clona el repositorio completo antes de ejecutar
3. Los archivos ZIP de zonas espejo se descomprimen automáticamente
4. OSMnx requiere conexión a internet para descargar la red peatonal
5. Todos los CRS se normalizan a WGS84 al cargar; EPSG:3116 solo se usa para cálculos métricos
6. Para medición post-intervención: cambiar `momento` a `post_intervencion` y re-ejecutar
7. **Mapas Folium en VS Code/Jupyter:** Si aparece "Make this Notebook Trusted to load map", ir a File → Trust Notebook (o clic en "Trust" en la barra superior). Es una acción manual única por notebook que habilita la ejecución de HTML/JS embebido.

# Contexto Maestro para Agente — Índice de Concurrencia

Este archivo resume el contexto más importante del repositorio para que un agente pueda trabajar con buen criterio metodológico y operativo.

## 1. Objetivo del proyecto

Calcular un **Índice de Concurrencia** para el corredor de Av. Roosevelt (Cali, Colombia) usando POIs de Overture Maps como proxy de actividad económica y presencia comercial en el territorio.

El índice permite:
- Establecer una **línea base pre-intervención** de la vitalidad económica del corredor.
- Comparar con mediciones **post-intervención** para evaluar impacto de obras públicas.
- Complementar el ITT con una lectura de densidad y diversidad comercial.

## 2. Reglas metodológicas principales

- La fuente de datos es **Overture Maps Foundation** (release 2026-04-15).
- El polígono de análisis es el **buffer de 100m** alrededor del eje de Av. Roosevelt.
- Los POIs se filtran espacialmente al polígono exacto (no solo al bounding box).
- Se clasifican en **13 macrocategorías** temáticas para análisis agregado.
- El momento de medición se registra como metadato (`linea_base_pre_intervencion`).
- Para comparaciones futuras, se repite el proceso con el release más reciente de Overture Maps.

## 3. Datos clave

- **474 POIs** dentro del polígono exacto (de 896 en el bounding box).
- Cada POI tiene: nombre, categoría Overture, confianza, lat, lon, macrocategoría.
- Metadatos de trazabilidad: fecha_extraccion, poligono, fuente, momento, release_overture.

## 4. Pipeline de procesamiento

El notebook (`notebooks_py/roosevelt_overture_poi.ipynb`) ejecuta:
1. Carga automática de datos desde repositorios Git (detecta Colab vs local)
2. Carga del polígono de intervención desde el repo `indice-caminabilidad-roosevelt`
3. Descarga de POIs vía CLI de Overture Maps usando bounding box calculado del polígono
4. Filtrado espacial al polígono exacto (`within`)
5. Extracción de atributos (nombre, categoría, confianza)
6. Clasificación en 13 macrocategorías
7. Mapa interactivo Folium multicapa (5 tile layers + POIs por categoría con popups)
8. Mapa estático matplotlib con dispersión por macrocategoría
9. Exportación a CSV y GeoJSON con metadatos de trazabilidad

## 5. Estructura del repositorio

```
Indice_Concurrencia/
├── agent/              → Configuración del agente (este directorio)
│   ├── context/        → Contexto del proyecto y glosario
│   ├── knowledge_base/ → Guías metodológicas
│   └── prompts/        → System prompt del agente
├── data/               → Datos de entrada y salida
│   ├── roosevelt_overture_poi.csv
│   ├── roosevelt_overture_poi (1).geojson
│   └── info_geo/       → Información geográfica complementaria
│       ├── Comunas.zip
│       └── geojson_comunas/Comunas.geojson
├── notebooks_py/       → Notebooks de análisis
│   ├── roosevelt_overture_poi.ipynb      → Notebook principal de POIs
│   └── caminabilidad_roosevelt_v2.ipynb  → Notebook de caminabilidad
├── outputs/            → Resultados (mapas, reportes)
├── docs/               → Documentación del proyecto
│   ├── metodologia.md
│   └── referencia_caminabilidad_notebook.md
├── README.md
├── requirements.txt
└── environment.yml
```

## 5.1 Repositorios Git relacionados

| Repositorio | URL | Uso |
|-------------|-----|-----|
| Índice de Concurrencia (este) | `https://github.com/j0rg3c45/Indice_Concurrencia.git` | Código, datos y documentación del índice |
| Caminabilidad Roosevelt | `https://github.com/j0rg3c45/indice-caminabilidad-roosevelt.git` | Fuente del polígono y datos geoespaciales del corredor |

## 6. Próximos pasos del proyecto

- Definir fórmula final del Índice de Concurrencia (densidad, diversidad, confianza promedio).
- Establecer umbrales o benchmarks para interpretar el índice usando los territorios espejo como referencia.
- Automatizar la descarga periódica para seguimiento temporal.
- Integrar con el ITT como indicador complementario de Entorno Urbano.
- Repetir medición post-intervención para evaluar impacto.
- Consolidar comparación con territorios espejo en un reporte ejecutivo.

## 7. Precauciones para el agente

- No confundir este proyecto con el ITT principal (que tiene su propio repositorio).
- Los datos de Overture Maps incluyen lugares con presencia digital aunque no tengan matrícula mercantil.
- El campo `confianza` de Overture Maps varía entre 0 y 1; valores bajos (<0.3) indican menor certeza.
- La categorización de Overture puede tener errores; la macrocategoría es una agrupación propia del equipo.
- No asumir que la cantidad de POIs equivale directamente a actividad económica real; es un proxy.

## 8. Dónde vive el conocimiento

Para responder bien sobre este proyecto, un agente debe leer en este orden:

1. `agent/context/contexto_proyecto.md` (resumen ejecutivo)
2. `agent/knowledge_base/metodologia_indice_concurrencia.md`
3. `agent/context/glosario.md`
4. `agent/context/zonas_estudio.md`
5. `docs/referencia_caminabilidad_notebook.md`
6. `notebooks_py/roosevelt_overture_poi.ipynb`
7. `data/roosevelt_overture_poi.csv`

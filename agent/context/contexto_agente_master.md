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
1. Descarga de POIs vía CLI de Overture Maps
2. Filtrado espacial al polígono exacto
3. Extracción de atributos (nombre, categoría, confianza)
4. Clasificación en macrocategorías
5. Visualización cartográfica
6. Exportación a CSV y GeoJSON con metadatos

## 5. Estructura del repositorio

```
Indice_Concurrencia/
├── agent/              → Configuración del agente (este directorio)
│   ├── context/        → Contexto del proyecto y glosario
│   ├── knowledge_base/ → Guías metodológicas
│   └── prompts/        → System prompt del agente
├── data/               → Datos de entrada (CSV, GeoJSON)
├── notebooks_py/       → Notebooks de análisis
├── outputs/            → Resultados (mapas, reportes)
├── docs/               → Documentación y metodología
├── README.md
├── requirements.txt
└── environment.yml
```

## 6. Próximos pasos del proyecto

- Definir fórmula del Índice de Concurrencia (densidad, diversidad, confianza promedio).
- Establecer umbrales o benchmarks para interpretar el índice.
- Automatizar la descarga periódica para seguimiento temporal.
- Integrar con el ITT como indicador complementario de Entorno Urbano.
- Repetir medición post-intervención para evaluar impacto.

## 7. Precauciones para el agente

- No confundir este proyecto con el ITT principal (que tiene su propio repositorio).
- Los datos de Overture Maps incluyen lugares con presencia digital aunque no tengan matrícula mercantil.
- El campo `confianza` de Overture Maps varía entre 0 y 1; valores bajos (<0.3) indican menor certeza.
- La categorización de Overture puede tener errores; la macrocategoría es una agrupación propia del equipo.
- No asumir que la cantidad de POIs equivale directamente a actividad económica real; es un proxy.

## 8. Dónde vive el conocimiento

Para responder bien sobre este proyecto, un agente debe leer en este orden:

1. `agent/context/contexto_proyecto.md` (este archivo es el resumen ejecutivo)
2. `agent/knowledge_base/metodologia_indice_concurrencia.md`
3. `agent/context/glosario.md`
4. `notebooks_py/roosevelt_overture_poi.ipynb`
5. `data/roosevelt_overture_poi.csv`

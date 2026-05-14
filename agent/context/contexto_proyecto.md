# Contexto del Proyecto — Índice de Concurrencia

## Organización

**ITT Cali Inteligente · Equipo de Gobierno de Datos**

## Objetivo del proyecto

Construir un **Índice de Concurrencia** para el corredor de intervención de la Av. Roosevelt en Cali, Colombia, usando puntos de interés (POI) de Overture Maps Foundation como proxy de actividad económica y presencia comercial.

## ¿Qué mide el Índice de Concurrencia?

El índice busca cuantificar la **densidad, diversidad y vitalidad** de la actividad económica y de servicios en un territorio, medida a través de la cantidad y variedad de establecimientos con presencia digital (negocios, servicios, equipamiento urbano).

## Relación con el ITT

El Índice de Concurrencia es un **insumo complementario** al Índice de Transformación Territorial (ITT). Mientras el ITT mide transformación territorial integral (seguridad, movilidad, entorno urbano, educación, cohesión social), el Índice de Concurrencia aporta una lectura específica sobre la **vitalidad económica del corredor** antes y después de la intervención urbana.

## Zona de estudio

- **Corredor:** Avenida Roosevelt, Cali, Colombia
- **Polígono:** Buffer de 100 metros alrededor del eje vial
- **Bounding box:** (-76.54235038, 3.42514711, -76.53685514, 3.43761420)
- **Área aproximada:** 42.9 hectáreas
- **CRS:** EPSG:4326

## Fuente de datos

- **Overture Maps Foundation** (Meta, Microsoft, Amazon, TomTom)
- Tipo de dato: `place` (POIs / puntos de interés)
- Release usado: 2026-04-15
- Formato: GeoJSON descargado vía CLI `overturemaps`
- Ventajas: sin API Key, sin costo, datos abiertos, actualización mensual

## Momento de medición

- **Línea base:** Pre-intervención (mayo 2026)
- **Comparación futura:** Post-intervención (cambiar `momento` a `post_intervencion` y repetir con release más reciente)

## Datos disponibles

| Archivo | Ubicación | Descripción |
|---------|-----------|-------------|
| `roosevelt_overture_poi.csv` | `data/` | 474 POIs filtrados al polígono exacto, con categorías y macrocategorías |
| `roosevelt_overture_poi (1).geojson` | `data/` | GeoJSON completo de los POIs filtrados |
| `roosevelt_overture_poi_mapa (1).png` | `outputs/` | Mapa de dispersión por macrocategoría |

## Pipeline actual (notebook)

1. Instalar dependencias (`overturemaps`, `geopandas`, `shapely`)
2. Definir bounding box del corredor Roosevelt
3. Descargar POIs desde Overture Maps CLI
4. Cargar y explorar datos descargados (896 POIs en bbox)
5. Filtrar al polígono exacto del corredor (474 POIs dentro)
6. Extraer nombre, categoría, confianza y coordenadas
7. Agrupar en 13 macrocategorías temáticas
8. Visualizar mapa con dispersión por macrocategoría
9. Exportar CSV limpio con metadatos para datalake
10. Descargar archivos (CSV, GeoJSON, PNG)

## Macrocategorías definidas

| Macrocategoría | Ejemplos de categorías incluidas |
|----------------|----------------------------------|
| Gastronomía y bebidas | restaurant, bar, bakery, coffee, fast_food |
| Salud y bienestar | health, medical, dental, hospital, pharmacy |
| Comercio y tiendas | store, shop, supermarket, clothing, electronics |
| Educación y formación | school, university, language, dance, music |
| Alojamiento | hotel, hostel, accommodation |
| Servicios profesionales | professional, financial, insurance, marketing, legal |
| Automotriz y transporte | automotive, car, motorcycle, gas_station |
| Belleza y estética | beauty, spa, salon, barber |
| Deporte y recreación | sport, gym, fitness, park, stadium |
| Religioso | church, religious, cathedral |
| Comunitario e institucional | community, social, nonprofit, government |
| Cultura y entretenimiento | art, museum, event, concert, travel |
| Otros | categorías no clasificadas |

## Resultados de línea base (mayo 2026)

- **Total POIs en polígono exacto:** 474
- **Distribución principal:**
  - Gastronomía y bebidas: ~80 lugares
  - Salud y bienestar: ~50 lugares
  - Comercio y tiendas: ~45 lugares
  - Automotriz y transporte: ~40 lugares
  - Servicios profesionales: ~35 lugares
  - Educación y formación: ~30 lugares

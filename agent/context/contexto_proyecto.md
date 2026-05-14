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
| `Comunas.zip` | `data/info_geo/` | Polígonos de comunas de Cali (ZIP) |
| `Comunas.geojson` | `data/info_geo/geojson_comunas/` | Polígonos de comunas extraídos |
| `roosevelt_overture_poi_mapa (1).png` | `outputs/` | Mapa de dispersión por macrocategoría |

## Pipeline actual (notebook)

El notebook `notebooks_py/roosevelt_overture_poi.ipynb` ejecuta:

1. Verificar e instalar dependencias (`overturemaps`, `geopandas`, `folium`, etc.)
2. Detectar entorno (Colab vs local) y clonar repos Git automáticamente
3. Cargar polígono del corredor desde repo `indice-caminabilidad-roosevelt`
4. Calcular bounding box automáticamente desde el polígono
5. Descargar POIs desde Overture Maps CLI
6. Filtrar al polígono exacto del corredor (474 POIs dentro)
7. Extraer nombre, categoría, confianza y coordenadas
8. Agrupar en 13 macrocategorías temáticas
9. Generar mapa interactivo Folium multicapa (5 tile layers, POIs por categoría con popups)
10. Generar mapa estático matplotlib con dispersión por macrocategoría
11. Exportar CSV limpio con metadatos para datalake

### Repositorios Git utilizados

| Repositorio | URL | Datos que aporta |
|-------------|-----|------------------|
| Índice de Concurrencia | `https://github.com/j0rg3c45/Indice_Concurrencia.git` | Proyecto principal, outputs |
| Caminabilidad Roosevelt | `https://github.com/j0rg3c45/indice-caminabilidad-roosevelt.git` | Polígono del corredor, datos geoespaciales |

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
  - Otros: ~87 lugares
  - Gastronomía y bebidas: ~68 lugares
  - Comercio y tiendas: ~56 lugares
  - Automotriz y transporte: ~51 lugares
  - Educación y formación: ~39 lugares
  - Salud y bienestar: ~36 lugares
  - Servicios profesionales: ~33 lugares
  - Cultura y entretenimiento: ~30 lugares

## Territorios espejo

El notebook incluye comparación con dos corredores espejo (Calle 5 y Calle 7) cuyos polígonos se obtienen del repo de caminabilidad. Para cada zona espejo se:
1. Descarga POIs de Overture Maps
2. Filtra al polígono exacto
3. Clasifica en las mismas 13 macrocategorías
4. Compara con Roosevelt mediante gráficos de barras agrupadas, totales y confianza promedio

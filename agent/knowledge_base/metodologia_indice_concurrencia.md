# Metodología — Índice de Concurrencia

## Guía técnica para cálculo, interpretación y seguimiento

**Versión:** 1.0  
**Última actualización:** Mayo 2026  
**Equipo:** ITT Cali Inteligente · Gobierno de Datos  
**Propósito:** Documento de referencia para el cálculo del Índice de Concurrencia basado en POIs de Overture Maps.

---

## 1. Definición

El **Índice de Concurrencia** es una métrica compuesta que cuantifica la vitalidad económica y de servicios de un territorio, medida a través de la densidad, diversidad y calidad de los puntos de interés (POI) con presencia digital.

Opera como un **proxy de actividad económica territorial** que permite:
- Establecer líneas base antes de intervenciones urbanas
- Medir cambios en la oferta comercial y de servicios post-intervención
- Comparar corredores o zonas entre sí

---

## 2. Fuente de datos

### Overture Maps Foundation

- **Qué es:** Dataset open data de POIs mantenido por Meta, Microsoft, Amazon y TomTom
- **Tipo de dato:** `place` (puntos de interés georreferenciados)
- **Cobertura:** Global
- **Actualización:** Mensual (releases versionados)
- **Acceso:** CLI `overturemaps` (sin API Key, sin costo)
- **Formato:** GeoJSON

### Ventajas sobre Google Maps

- Sin API Key ni solicitud de acceso
- Sin costo
- Datos descargables en formato abierto
- Actualización mensual documentada
- Incluye lugares con presencia digital aunque no tengan matrícula mercantil

### Limitaciones

- No todos los negocios tienen presencia digital
- El campo `confianza` varía significativamente (0.09 a 0.98)
- La categorización puede tener errores o ser genérica
- No mide flujo de personas ni ingresos reales

---

## 3. Proceso de extracción de datos

### 3.1 Definir polígono de análisis

- Generar buffer de 100m alrededor del eje vial
- Exportar como GeoJSON con CRS EPSG:4326
- Calcular bounding box para la descarga inicial

### 3.2 Descargar POIs

```bash
overturemaps download \
  --bbox=lon_min,lat_min,lon_max,lat_max \
  -f geojson \
  --type=place \
  -o output_raw.geojson
```

### 3.3 Filtrar al polígono exacto

El bounding box descarga más POIs de los necesarios. Se aplica un filtro espacial (`within`) para retener solo los que caen dentro del polígono exacto.

### 3.4 Extraer atributos

De cada POI se extraen:
- **nombre:** campo `names.primary`
- **categoría:** campo `categories.primary`
- **confianza:** campo `confidence` (0-1)
- **coordenadas:** geometría del punto (lat, lon)

### 3.5 Clasificar en macrocategorías

Se aplica una función de agrupación que mapea las ~150 categorías de Overture a 13 macrocategorías temáticas definidas por el equipo.

---

## 4. Componentes del Índice de Concurrencia (propuesta)

### 4.1 IC v1 — Índice original (3 componentes)

```
IC_v1 = 0.40 × Densidad_area_norm + 0.40 × Diversidad_norm + 0.20 × Confianza_norm
```

| Componente | Fórmula cruda | Peso | ref_min | ref_max |
|-----------|---------------|------|---------|---------|
| Densidad por área | Total_POIs / Área_ha | 40% | 2 | 25 |
| Diversidad (Shannon) | (H / ln(13)) × 100 | 40% | 30 | 95 |
| Confianza | media(campo confidence) | 20% | 0.30 | 0.80 |

### 4.2 IC v2 — Índice mejorado (4 componentes, con dato poblacional)

```
IC_v2 = 0.30 × Densidad_area_norm + 0.30 × Diversidad_norm + 0.20 × Densidad_pob_norm + 0.20 × Confianza_norm
```

| Componente | Fórmula cruda | Peso | ref_min | ref_max |
|-----------|---------------|------|---------|---------|
| Densidad por área | Total_POIs / Área_ha | 30% | 2 | 25 |
| Diversidad (Shannon) | (H / ln(13)) × 100 | 30% | 30 | 95 |
| Densidad poblacional | Total_POIs / (Población / 1000) | 20% | 5 | 80 |
| Confianza | media(campo confidence) | 20% | 0.30 | 0.80 |

### 4.3 Notas sobre las fórmulas

**Normalización (aplica a ambas versiones):**
```
score = clamp((valor - ref_min) / (ref_max - ref_min) × 100, 0, 100)
```
- `clamp` acota el resultado al rango [0, 100]
- Si `valor < ref_min` → score = 0
- Si `valor > ref_max` → score = 100

**Índice de Shannon (diversidad):**
```
H = -Σ (pi × ln(pi))
```
- `pi` = proporción de POIs en la macrocategoría `i`
- `H_max = ln(13)` (13 macrocategorías definidas)
- Se normaliza: `diversidad = (H / H_max) × 100`
- Valor 0 = todos los POIs en una sola categoría
- Valor 100 = distribución perfectamente uniforme

**Densidad poblacional (IC v2):**
- Fuente: `Personas_por_hogar_según_barrio_2016.zip` (shapefile de 334 barrios)
- Se buscan barrios que intersectan con el polígono de la zona
- Si la población directa es < 500 (datos parciales), se usa la comuna completa
- Dato de 2016 — debe actualizarse cuando haya censo más reciente
- Interpretación: más POIs por habitante = mayor oferta comercial relativa

**Criterios de calibración de refs:**
- `ref_min` = valor aspiracional o mínimo razonable para corredores urbanos de Cali
- `ref_max` = valor alto pero alcanzable en zonas comerciales consolidadas
- Los refs son fijos y documentados, no calculados dinámicamente de los datos
- Consistentes con la metodología del ITT (ver `Guia_ITT_Metodologia_Notebook.md`)

---

## 5. Normalización de componentes

Para hacer comparables los componentes, se normalizan a escala 0-100 con refs fijos:

| Componente | ref_min | ref_max | Justificación |
|-----------|---------|---------|---------------|
| Densidad (POIs/ha) | 2 | 25 | Rango razonable para corredores urbanos de Cali |
| Diversidad (Shannon norm) | 30 | 95 | Mínimo aceptable vs. distribución casi perfecta |
| Confianza promedio | 0.3 | 0.8 | Rango observado en zonas urbanas consolidadas |

```
score = clamp((valor - ref_min) / (ref_max - ref_min) × 100, 0, 100)
```

---

## 6. Interpretación del Índice

| Rango IC | Nivel | Descripción |
|----------|-------|-------------|
| 0 – 30 | Bajo | Poca actividad económica visible. Zona residencial o en declive |
| 30 – 50 | Moderado | Actividad económica presente pero limitada en densidad o diversidad |
| 50 – 70 | Alto | Corredor activo con buena mezcla de usos y servicios |
| 70 – 100 | Muy alto | Zona de alta vitalidad económica, diversa y consolidada |

---

## 7. Seguimiento temporal

### Protocolo de medición periódica

1. **Frecuencia:** Cada 6 meses (o al menos pre y post intervención)
2. **Mismo polígono:** Usar exactamente el mismo GeoJSON de referencia
3. **Registrar release:** Anotar qué release de Overture Maps se usó
4. **Comparar:**
   - Δ Total POIs (¿aumentaron o disminuyeron?)
   - Δ Distribución por macrocategoría (¿cambió la composición?)
   - Δ Confianza promedio (¿mejoró la calidad de los datos?)
   - Δ Índice compuesto

### Metadatos obligatorios por medición

| Campo | Descripción |
|-------|-------------|
| fecha_extraccion | Fecha en que se ejecutó la descarga |
| poligono | Nombre del polígono usado |
| fuente | "Overture Maps Foundation" |
| momento | "linea_base_pre_intervencion" o "post_intervencion" |
| release_overture | Versión del release (ej: "2026-04-15") |

---

## 8. Integración con el ITT

El Índice de Concurrencia puede integrarse al ITT como:

- **Indicador complementario de Entorno Urbano:** aporta lectura de vitalidad económica
- **Indicador independiente:** se reporta junto al ITT pero sin afectar su cálculo
- **Proxy de transformación económica:** si el IC sube post-intervención, sugiere impacto positivo en la economía local

La decisión de integración depende del equipo y de la validación con datos reales post-intervención.

---

## 8.1 Comparación con territorios espejo

El notebook incluye descarga y análisis de POIs para dos territorios espejo (Calle 5 y Calle 7) que sirven como referencia comparativa.

### Proceso para zonas espejo

1. Cargar polígonos desde `data/Informacion_espejo/geojson_espejo_poligonos/` (repo caminabilidad)
2. Descargar POIs de Overture Maps para cada zona
3. Filtrar al polígono exacto
4. Clasificar en las mismas 13 macrocategorías
5. Comparar con Roosevelt

### Gráficos comparativos generados

| Gráfico | Tipo | Contenido |
|---------|------|-----------|
| Barras agrupadas por macrocategoría | barh agrupado | Cantidad de POIs por categoría, 3 zonas lado a lado |
| Total de POIs por zona | barras verticales | Comparación directa del volumen total |
| Confianza promedio por zona | barras verticales | Calidad/certeza de los datos por zona |

### Interpretación

- Si Roosevelt tiene **menos POIs** que los espejo → menor vitalidad económica relativa
- Si Roosevelt tiene **menor diversidad** → concentración en pocas categorías
- Si Roosevelt tiene **menor confianza** → datos menos consolidados (negocios más nuevos o informales)
- Post-intervención: si Roosevelt **sube** y los espejo se mantienen → evidencia de impacto positivo

---

## 9. Limitaciones y precauciones

| Limitación | Implicación |
|-----------|-------------|
| No mide flujo de personas | Un lugar puede existir pero tener poca clientela |
| Sesgo digital | Negocios informales sin presencia digital no aparecen |
| Confianza variable | POIs con confianza <0.3 pueden ser falsos positivos |
| Categorización imperfecta | La macrocategoría es una aproximación del equipo |
| No mide ingresos | Más POIs ≠ más ingresos necesariamente |
| Dependencia de Overture | Cambios en la metodología de Overture afectan la comparabilidad |

---

## 10. Errores comunes a evitar

| Error | Consecuencia | Corrección |
|-------|-------------|-----------|
| Usar bounding box sin filtro espacial | Incluir POIs fuera del corredor | Siempre filtrar con `within(polígono)` |
| Comparar releases muy distantes | Cambios en cobertura de Overture, no del territorio | Documentar release y considerar efecto plataforma |
| Ignorar campo confianza | Contar lugares que probablemente no existen | Filtrar o ponderar por confianza |
| Asumir causalidad | "Las obras causaron más POIs" | Solo se puede hablar de correlación temporal |
| No registrar metadatos | Imposible reproducir o comparar | Siempre exportar con fecha, release y momento |

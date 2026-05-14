# Metodología — Índice de Concurrencia

## Objetivo

Cuantificar la vitalidad económica y de servicios del corredor de Av. Roosevelt (Cali) usando POIs de Overture Maps como proxy de actividad comercial, permitiendo comparaciones pre y post intervención urbana.

## Fuentes de datos

| Fuente | Tipo | Frecuencia | Acceso |
|--------|------|-----------|--------|
| Overture Maps Foundation | POIs georreferenciados | Release mensual | CLI `overturemaps` (open data) |
| Polígono Roosevelt Buffer 100m | GeoJSON del corredor | Fijo | Generado por el equipo |

## Proceso metodológico

1. **Descarga:** POIs tipo `place` dentro del bounding box del corredor vía CLI de Overture Maps
2. **Filtrado espacial:** Retener solo POIs dentro del polígono exacto (buffer 100m)
3. **Extracción de atributos:** nombre, categoría, confianza, coordenadas
4. **Clasificación:** Agrupación en 13 macrocategorías temáticas
5. **Cálculo del índice:** Densidad + Diversidad (Shannon) + Confianza promedio
6. **Exportación:** CSV con metadatos de trazabilidad para el datalake

## Variables e indicadores

| Componente | Fórmula | Peso sugerido |
|-----------|---------|---------------|
| Densidad | Total POIs / Área (ha) | 40% |
| Diversidad | Shannon normalizado (H / ln(13)) × 100 | 40% |
| Confianza | Media del campo `confidence` de Overture | 20% |

## Clasificación del índice

| Rango | Nivel | Interpretación |
|-------|-------|----------------|
| 0 – 30 | Bajo | Poca actividad económica visible |
| 30 – 50 | Moderado | Actividad presente pero limitada |
| 50 – 70 | Alto | Corredor activo con buena mezcla de usos |
| 70 – 100 | Muy alto | Alta vitalidad económica, diversa y consolidada |

## Limitaciones

- Los POIs reflejan presencia digital, no necesariamente actividad económica real
- Negocios informales sin presencia digital no aparecen en Overture Maps
- El campo `confianza` varía significativamente; valores <0.3 son poco confiables
- No mide flujo de personas, ingresos ni empleo
- Cambios en la metodología de Overture Maps pueden afectar comparabilidad temporal
- La relación entre POIs y transformación territorial es correlacional, no causal

## Comparación con territorios espejo

Se comparan los resultados de Roosevelt con dos corredores espejo (Calle 5 y Calle 7) para contextualizar los valores y establecer benchmarks.

| Aspecto comparado | Descripción |
|-------------------|-------------|
| Total de POIs | Volumen absoluto de actividad económica |
| Distribución por macrocategoría | Composición del tejido comercial |
| Confianza promedio | Calidad/madurez de la presencia digital |
| Diversidad (Shannon) | Equilibrio entre categorías |

Los polígonos espejo se obtienen del repositorio `indice-caminabilidad-roosevelt` y se procesan con el mismo pipeline que Roosevelt.

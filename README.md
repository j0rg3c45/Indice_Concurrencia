# Índice de Concurrencia — Av. Roosevelt

**ITT Cali Inteligente · Equipo de Gobierno de Datos**

## Descripción

Proyecto para el cálculo del **Índice de Concurrencia** del corredor de Av. Roosevelt (Cali, Colombia), usando puntos de interés (POI) de Overture Maps Foundation como proxy de actividad económica y presencia comercial.

El índice permite establecer una línea base pre-intervención y comparar con mediciones post-intervención para evaluar el impacto de las obras públicas en la vitalidad económica del corredor.

## Datos de línea base (mayo 2026)

- **474 POIs** dentro del polígono exacto (buffer 100m del eje vial)
- **13 macrocategorías** temáticas
- **Release Overture Maps:** 2026-04-15
- **Fuente:** Overture Maps Foundation (Meta, Microsoft, Amazon, TomTom)

## Estructura del proyecto

```
Indice_Concurrencia/
│
├── README.md
├── requirements.txt
├── environment.yml
├── .gitignore
│
├── agent/              # Configuración del agente de IA
│   ├── context/        # Contexto del proyecto y glosario
│   ├── knowledge_base/ # Guías metodológicas
│   └── prompts/        # System prompt del agente
├── data/               # Datos de entrada (CSV, GeoJSON)
├── notebooks_py/       # Notebooks de análisis
├── outputs/            # Resultados (mapas, reportes)
├── docs/               # Documentación del proyecto
│   └── metodologia.md
```

## Instalación

```bash
pip install -r requirements.txt
```

o con conda:

```bash
conda env create -f environment.yml
```

## Uso

El notebook principal (`notebooks_py/roosevelt_overture_poi.ipynb`) ejecuta el pipeline completo:
1. Descarga POIs desde Overture Maps
2. Filtra al polígono exacto del corredor
3. Clasifica en macrocategorías
4. Genera mapa de dispersión
5. Exporta CSV limpio con metadatos

## Metodología

Ver `docs/metodologia.md` para la guía completa de cálculo e interpretación del índice.

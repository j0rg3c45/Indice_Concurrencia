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
├── data/               # Datos de entrada y salida
│   ├── roosevelt_overture_poi.csv
│   ├── roosevelt_overture_poi (1).geojson
│   └── info_geo/       # Información geográfica complementaria (comunas)
├── notebooks_py/       # Notebooks de análisis
│   ├── roosevelt_overture_poi.ipynb      # Notebook principal de POIs
│   └── caminabilidad_roosevelt_v2.ipynb  # Notebook de caminabilidad
├── outputs/            # Resultados (mapas, reportes)
├── docs/               # Documentación del proyecto
│   ├── metodologia.md
│   └── referencia_caminabilidad_notebook.md
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
1. Clona automáticamente los repositorios de datos (detecta Colab vs local)
2. Carga el polígono del corredor desde el repo de caminabilidad
3. Descarga POIs desde Overture Maps
4. Filtra al polígono exacto del corredor
5. Clasifica en macrocategorías
6. Genera mapa interactivo Folium multicapa
7. Genera mapa estático matplotlib
8. Exporta CSV limpio con metadatos

### Repositorios relacionados

| Repositorio | Uso |
|-------------|-----|
| [Indice_Concurrencia](https://github.com/j0rg3c45/Indice_Concurrencia.git) | Este proyecto |
| [indice-caminabilidad-roosevelt](https://github.com/j0rg3c45/indice-caminabilidad-roosevelt.git) | Fuente del polígono y datos geoespaciales |

## Metodología

Ver `docs/metodologia.md` para la guía completa de cálculo e interpretación del índice.

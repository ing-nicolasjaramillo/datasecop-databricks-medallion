# Pipeline de contratación pública en Azure Databricks

Pipeline de datos que toma los procesos de contratación pública del Quindío en 2025, publicados en SECOP II, y los transforma en información lista para analizar, usando la arquitectura Medallion (Bronze → Silver → Gold).

## Arquitectura

API datos.gov.co → **Bronze** (datos crudos) → **Silver** (datos limpios) → **Gold** (métricas)

| Capa | Qué contiene | Registros |
|---|---|---|
| Bronze | Datos tal como llegan de la API, con origen y fecha de carga | 31.552 |
| Silver | Un registro por proceso, sin duplicados y con formatos corregidos | 29.410 válidos + 508 en cuarentena |
| Gold | Tablas resumen para análisis | 6 tablas |

## Tecnologías

Azure Databricks · PySpark · SQL · Delta Lake · Unity Catalog · API Socrata (datos.gov.co) · GitHub

## Principales hallazgos

- **El 94 % de los procesos no tiene competencia:** contratación directa y régimen especial concentran casi todos los procesos y el 65 % del valor.
- **Mínima cuantía es la modalidad con más ahorro (10,9 %)** y más competencia (3,15 ofertas en promedio).
- **En contratación directa con ofertas, el 82,5 % de los procesos tuvo un único oferente.**
- **Febrero concentra el 22 % del valor contratado del año.**

![Procesos por mes](docs/procesos_mensual.png)

## Calidad de datos

- Se eliminaron 1.467 registros duplicados (4,6 %) presentes en la fuente.
- 48 procesos tenían varios valores de adjudicación sin forma de asociarlos a cada proveedor. Se marcaron y su valor se dejó vacío en lugar de adivinarlo.
- Los registros sin fecha o con precio base en cero se separaron en una tabla de cuarentena.

## Limitaciones

- El ranking de proveedores solo incluye procesos competitivos; la contratación directa no registra adjudicación en esta fuente.
- El porcentaje de procesos "sin ofertas" en modalidades competitivas puede incluir procesos en curso o desiertos.

## Estructura

notebooks/
├── 00_setup            Creación de esquemas y volumen
├── 01_bronze_ingesta   Descarga desde la API y análisis de duplicados
├── 02_silver_limpieza  Limpieza, validación y MERGE
├── 03_gold_metricas    Métricas de negocio
└── 04_delta_features   Historial de versiones y recuperación de errores
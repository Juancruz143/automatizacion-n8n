# Automatización de datos con n8n: API REST y pipeline de migración

Dos flujos de **n8n** desarrollados para *Minería de Datos y Big Data*, Licenciatura en Ciencia de Datos (UCA, 2026):

1. Un **backend REST** para una aplicación de gestión agropecuaria.
2. Un **pipeline ETL** que migra planillas Excel históricas heterogéneas a una base unificada.

Para importarlos: en n8n, *Workflows → Import from File* y elegir el JSON de [`workflows/`](workflows).

---

## 1. Backend de gestión reproductiva del rodeo

[`workflows/backend_gestion_agropecuaria.json`](workflows/backend_gestion_agropecuaria.json) · 34 nodos

Proyecto grupal por equipos (Equipo 2, backend). La API expone el registro de **tactos** (diagnósticos de preñez) y **pariciones** de un rodeo bovino. La persistencia está en Google Sheets y el frontend (React + Vite) está desacoplado.

| Método | Endpoint | Qué hace |
|---|---|---|
| `POST` | `/tactos` | Registra un tacto: valida campos, verifica que la caravana exista, guarda y avisa por email |
| `POST` | `/pariciones` | Registra una parición con las mismas validaciones |
| `GET` | `/tactos` | Lista los tactos |
| `GET` | `/pariciones` | Lista las pariciones |
| `GET` | `/tactos/positivos` | Filtra los tactos con preñez confirmada |

Flujo de cada alta:

```
Webhook → Validar campos ─(faltan)→ 400
              │
              └→ Buscar caravana en la maestra ─(no existe)→ 404
                        │
                        └→ Regla de negocio → Guardar en Sheets → Email → 200
```

Respuestas HTTP explícitas (200, 400 y 404) según un contrato de API acordado entre equipos.

## 2. Pipeline de migración de Excel históricos

[`workflows/pipeline_migracion_excel_historicos.json`](workflows/pipeline_migracion_excel_historicos.json) · 11 nodos

Caso: una fundación de salud comunitaria con años de registros en **planillas Excel con formatos distintos** (columnas con otros nombres, DNI con y sin puntos, fechas y "Sí/No" escritos de varias formas).

```
Trigger (manual o semanal) → Ingesta heterogénea → Normalización → Deduplicación por DNI
      → ¿Registro válido? ─ sí → Carga incremental (upsert persona + vínculo) → Reporte de cargados
                          └ no → Log de errores para revisión manual       → Reporte de errores
```

- **Normalización:** unifica nombres de columnas, limpia DNI, estandariza fechas y valores sí/no.
- **Deduplicación** por DNI dentro de cada ejecución.
- **Carga incremental idempotente:** upsert por DNI (persona) y por DNI + centro (vínculo).
- **Tolerancia a errores:** los registros inválidos se separan para auditoría sin cortar el flujo.
- **Programación semanal** (domingos a las 3 a. m.) además del disparo manual.

> Los datos de ejemplo son **ficticios** y la carga usa la memoria del workflow. En producción se reemplazan la ingesta por lectura real de Excel (Drive u OneDrive) y la carga por Postgres o Supabase.

## Stack

n8n · JavaScript (nodos Code) · Webhooks / API REST · Google Sheets · Gmail

---

**Juan Cruz Montero** · Estudiante de Lic. en Ciencia de Datos (UCA) · [LinkedIn](https://www.linkedin.com/in/juan-cruz-montero-a14474431)

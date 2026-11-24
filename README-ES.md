# Observability training: Movie Store checkout

## Contexto

Este repositorio contiene un escenario **sintético de entrenamiento SRE** para investigar lentitud y fallas en el flujo de checkout de " Movie Store". Los datos permiten correlacionar métricas técnicas, resultados de negocio y logs para practicar análisis de incidentes con Grafana y Elasticsearch.

El escenario incluye dos ambientes:

- **`dev` (10 de diciembre de 2026):** señales de advertencia antes de producción. Entre 19:35 y 19:44 UTC aumentan gradualmente la latencia, la carga y el uso del pool de base de datos; entre 19:45 y 19:49 se simula mitigación.
- **`prod` (11 de diciembre de 2026):** operación normal hasta las 19:29 UTC; incidente de checkout desde las **19:30 hasta las 19:44 UTC**, seguido de mitigación desde las 19:45.
- **`prod` (12 de diciembre de 2026):** datos de referencia en estado normal.

Durante el incidente de producción coinciden errores HTTP 5xx, latencia p95 elevada, agotamiento del pool de base de datos y un deterioro de los resultados de checkout. A las 19:30 UTC se registra también una alerta `CRITICAL` que resume esas señales. La coincidencia sirve para investigar una hipótesis; por sí sola no demuestra la causa raíz.

> Todos los datos son generados para capacitación, no representan telemetría real. Los timestamps usan UTC (`Z`).

## Archivos de datos

| Archivo | Contenido |
| --- | --- |
| [`sre-lab-metrics.csv`](./sre-lab-metrics.csv) | Métricas técnicas por minuto: tasa de solicitudes, porcentaje de 5xx, latencia p95, CPU, conexiones del pool y timeouts de base de datos. |
| [`sre-lab-business-metrics.csv`](./sre-lab-business-metrics.csv) | Métricas de negocio por minuto: checkouts iniciados y completados, carritos abandonados y pagos intentados y rechazados. |
| [`sre-lab-logs.csv`](./sre-lab-logs.csv) | Muestras de eventos de checkout y una alerta crítica. Contiene cinco eventos de solicitud por intervalo de métricas, más la alerta. |

Los archivos de métricas comparten `timestamp`, `service`, `environment` y `phase`, así que pueden relacionarse directamente por timestamp. Los logs usan los campos `@timestamp`, `service.name` y `deployment.environment`; los eventos de solicitud pueden identificarse con `event.kind=event` y la alerta con `event.kind=alert`.

### Fases del escenario

- `normal`: valores de referencia.
- `warning`: degradación gradual en `dev`.
- `incident`: falla de producción.
- `mitigation`: recuperación simulada después de la degradación o del incidente.

Las métricas tienen 45 puntos por ventana diaria, de 19:20 a 20:04 UTC, a intervalos de un minuto. Los logs son muestras sintéticas, no un registro exhaustivo de todas las solicitudes; por eso sus conteos no deben interpretarse como el total de tráfico real.

## Uso para el ejercicio

1. Compara los valores normales de `prod` con el periodo del incidente.
2. Busca qué señales aumentan primero en `dev` y si podrían servir como indicadores preventivos.
3. Correlaciona latencia, errores 5xx y saturación del pool con abandono de carritos, rechazos de pago y checkouts completados.
4. Revisa si las señales mejoran durante la mitigación y cuánto tardan los indicadores de negocio en recuperarse.
5. Separa hechos observados, hipótesis y datos que aún harían falta para afirmar una causa raíz.

## Grafana

Los CSV de métricas pueden explorarse con Infinity como fuente de datos CSV. El archivo [`grafana-dashboard-requirements.csv`](./grafana-dashboard-requirements.csv) contiene una guía de paneles y umbrales; [`grafana-prompts.txt`](./grafana-prompts.txt) reúne instrucciones generales para crear dashboards, y [`grafana-ai-prompts.txt`](./grafana-ai-prompts.txt) incluye prompts para Grafana clasificados por calidad e intención, incluidos ejemplos ambiguos y pruebas de robustez.

Los dashboards exportados disponibles están en [`dashboards/`](./dashboards/). Verifica y configura la fuente de datos del dashboard en tu propia instancia antes de importarlo.

## Elasticsearch

El archivo [`elastic-esql.txt`](./elastic-esql.txt) contiene consultas ES|QL de ejemplo para agrupar errores, analizar intervalos de tiempo e investigar saturación del pool. El archivo [`elastic-prompts.txt`](./elastic-prompts.txt) reúne prompts para explorar los logs, cargar CSV y crear dashboards en Kibana. El campo `event.kind` permite separar alertas de eventos de solicitudes para que no se mezclen en los agregados.

Antes de ejecutar las consultas, ingiere el CSV de logs en un índice o data stream llamado `sre-lab-logs*`, o adapta ese nombre al destino que uses.

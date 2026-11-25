# Observability training: Movie Store checkout

## Context

This repository contains a **synthetic SRE training scenario** for investigating slowness and failures in the "Movie Store" checkout flow. The data enables correlation of technical metrics, business outcomes, and logs to practice incident analysis with Grafana and Elasticsearch.

The scenario includes two environments:

- **`dev` (December 10, 2026):** warning signals before production. From 19:35 to 19:44 UTC, latency, load, and database pool usage increase gradually; mitigation is simulated from 19:45 to 19:49.
- **`prod` (December 11, 2026):** normal operations until 19:29 UTC; checkout incident from **19:30 to 19:44 UTC**, followed by mitigation starting at 19:45.
- **`prod` (December 12, 2026):** reference data in a normal state.

During the production incident, HTTP 5xx errors, elevated p95 latency, database pool exhaustion, and deterioration of checkout outcomes coincide. At 19:30 UTC, a `CRITICAL` alert also records a summary of these signals. The coincidence helps investigate a hypothesis but does not, by itself, prove the root cause.

> All data is generated for training and does not represent real telemetry. Timestamps use UTC (`Z`).

## Data files

| File | Content |
| --- | --- |
| [`sre-lab-metrics.csv`](./sre-lab-metrics.csv) | Per-minute technical metrics: request rate, 5xx percentage, p95 latency, CPU, pool connections, and database timeouts. |
| [`sre-lab-business-metrics.csv`](./sre-lab-business-metrics.csv) | Per-minute business metrics: started and completed checkouts, abandoned carts, and attempted and rejected payments. |
| [`sre-lab-logs.csv`](./sre-lab-logs.csv) | Samples of checkout events and one critical alert. Contains five request events per metrics interval, plus the alert. |

The metrics files share `timestamp`, `service`, `environment`, and `phase`, so they can be correlated directly by timestamp. The logs use the fields `@timestamp`, `service.name`, and `deployment.environment`; request events can be identified with `event.kind=event`, and the alert with `event.kind=alert`.

### Scenario phases

- `normal`: reference values.
- `warning`: gradual degradation in `dev`.
- `incident`: production failure.
- `mitigation`: simulated recovery after degradation or an incident.

The metrics contain 45 points per daily window, from 19:20 to 20:04 UTC, at one-minute intervals. The logs are synthetic samples, not an exhaustive record of every request; therefore, their counts must not be interpreted as the total real traffic.

## Exercise usage

1. Compare the normal `prod` values with the incident period.
2. Determine which signals increase first in `dev` and whether they could serve as preventive indicators.
3. Correlate latency, 5xx errors, and pool saturation with cart abandonment, payment rejections, and completed checkouts.
4. Check whether the signals improve during mitigation and how long the business indicators take to recover.
5. Separate observed facts, hypotheses, and additional data needed to establish a root cause.

## Grafana

The metrics CSV files can be explored using the Infinity CSV data source. The [`grafana-dashboard-requirements.csv`](./grafana-dashboard-requirements.csv) file contains a guide to panels and thresholds. The [`grafana-prompts.txt`](./prompts/grafana-prompts.txt) and [`grafana-prompts-es.txt`](./prompts/grafana-prompts-es.txt) files contain Grafana prompts classified by quality and intent, including ambiguous examples and robustness tests.

The exported dashboards are available in [`dashboards/`](./dashboards/). Verify and configure the dashboard data source in your own instance before importing it.

## Elasticsearch

The [`elastic-esql.txt`](./prompts/elastic-esql.txt) file contains example ES|QL queries for grouping errors, analyzing time ranges, and investigating pool saturation. The [`elastic-prompts.txt`](./prompts/elastic-prompts.txt) file contains prompts for exploring logs, uploading CSV files, and creating Kibana dashboards. The `event.kind` field allows alerts to be separated from request events so they are not mixed in aggregations.

Before running the queries, ingest the logs CSV into an index or data stream named `sre-lab-logs*`, or adapt that name to the destination you use.

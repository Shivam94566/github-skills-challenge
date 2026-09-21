# AIOps Anomaly Detection Pipeline

## 1. Scenario

This project demonstrates a lightweight AIOps workflow for monitoring a
`payment-service`.

The system receives operational telemetry containing service metrics and logs.
It detects abnormal behaviour and converts detected anomalies into events that
are passed through an event producer, an in-memory event topic, and an event
consumer.

The purpose of the AIOps pipeline is to automatically identify unusual service
behaviour and make the detected issues available as structured anomaly events.

---

## 2. Operational Problem

A payment service may experience:

- High response time
- High CPU utilization
- High memory utilization
- Payment or database timeout errors

Manually checking these metrics and logs can make it difficult to identify
problems quickly.

This pipeline provides a simple automated anomaly-detection workflow.

---

## 3. Operational Data

The operational data is stored in:

`data/service_data.json`

The dataset contains 10 observations for the `payment-service`.

### Metric fields

- `response_time_ms` - service response time in milliseconds
- `cpu_percent` - CPU utilization percentage
- `memory_percent` - memory utilization percentage

### Log fields

- `log_level` - log severity level
- `message` - service log message

### Timestamp

- `timestamp` - time at which the observation was recorded

---

## 4. Data Observations

Most observations show normal payment-service behaviour.

Normal observations have:

- Response time around 120–150 ms
- CPU utilization around 42–50%
- Memory utilization around 51–57%
- `INFO` log level
- Successful payment-processing messages

Two observations contain abnormal behaviour:

| Timestamp | Response Time | CPU | Memory | Log Level | Observation |
|---|---:|---:|---:|---|---|
| 2026-09-20 10:05 | 610 ms | 75% | 70% | ERROR | Payment service timeout |
| 2026-09-20 10:06 | 640 ms | 94% | 91% | ERROR | Database connection timeout |

---

## 5. Anomaly Detection

The `AnomalyDetector` checks the following thresholds:

- Response time > 500 ms
- CPU utilization > 80%
- Memory utilization > 80%
- `ERROR` log level

A detected anomaly contains:

- Timestamp
- Service
- Anomaly type
- Detection reasons
- Original source record

### Detection Results

Two anomalies were detected.

#### Anomaly 1

Timestamp:

`2026-09-20T10:05:00`

Reasons:

- High response time
- Error log detected

#### Anomaly 2

Timestamp:

`2026-09-20T10:06:00`

Reasons:

- High response time
- High CPU utilization
- High memory utilization
- Error log detected

The supplied dataset did not produce false-positive anomaly results during the
final pipeline execution.

---

## 6. Event Flow

The implemented architecture follows this flow:

```text
Operational Data
       |
       v
AnomalyDetector
       |
       v
EventProducer
       |
       v
service-events EventTopic
       |
       v
EventConsumer
       |
       v
AIOps Pipeline Result
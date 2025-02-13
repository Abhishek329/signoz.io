---
id: traefik-observability
title: Traefik Observability
description: Tutorial to export Traefik metrics and traces to SigNoz.
---
import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

import TraefikMetrics from '../shared/traefik-metrics-list.md'

### Overview

In this tutorial, we will see how to export metrics and traces of Traefik to SigNoz.
Visualizing Traefik metrics and traces will help you to understand the performance
of services running behind Traefik and troubleshoot issues.

### Prerequisites

- Traefik v3.0 or above
- Must have SigNoz running. You can follow the [installation guide][1] to install SigNoz.
- Must have SigNoz OtelCollector accessible from Traefik
- If you don’t already have a SigNoz Cloud account, you can sign up [here][4].

## Export Traefik Metrics and Traces to SigNoz

Based on how you are running SigNoz (e.g. SigNoz Cloud, in an independent VM or Kubernetes cluster),
you have to provide the address to send data from the above receivers.

<Tabs>
<TabItem value="signoz-cloud" label="SigNoz Cloud" default>

In this section, we will see how to export Traefik metrics and traces to SigNoz Cloud.

For metrics, we will have to set the following CLI flags in Traefik:

- `--metrics.openTelemetry=true`
- `--metrics.openTelemetry.grpc=true`
- `--metrics.openTelemetry.address=ingest.{region}.signoz.cloud:443`
- `--metrics.openTelemetry.insecure=false`
- `--metrics.openTelemetry.headers.signoz-access-token=SIGNOZ_INGESTION_KEY`

For traces, we will have to set the following CLI flags in Traefik:

- `--tracing.openTelemetry=true`
- `--tracing.openTelemetry.grpc=true`
- `--tracing.openTelemetry.address=ingest.{region}.signoz.cloud:443`
- `--tracing.openTelemetry.insecure=false`
- `--tracing.openTelemetry.headers.signoz-access-token=SIGNOZ_INGESTION_KEY`

We will take an example `docker-compose.yaml` with a simple `hello-app`
running behind Traefik.

_docker-compose.yaml_

```yaml {13-14,18-19}
version: '3'
services:
  reverse-proxy:
    image: traefik:v3.0.0-beta3
    extra_hosts:
      - signoz:host-gateway
    command:
      - --api.insecure=true
      - --providers.docker
      - --metrics.openTelemetry=true
      - --metrics.openTelemetry.grpc=true
      - --metrics.openTelemetry.insecure=false
      - --metrics.openTelemetry.address=ingest.{region}.signoz.cloud:443
      - --metrics.openTelemetry.headers.signoz-access-token=SIGNOZ_INGESTION_KEY
      - --tracing.openTelemetry=true
      - --tracing.openTelemetry.grpc=true
      - --tracing.openTelemetry.insecure=false
      - --tracing.openTelemetry.address=ingest.{region}.signoz.cloud:443
      - --tracing.openTelemetry.headers.signoz-access-token=SIGNOZ_INGESTION_KEY
    ports:
      - "80:80"
      - "8080:8080"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
  hello-app:
    image: gcr.io/google-samples/hello-app:2.0
    environment:
      - PORT=8080
    labels:
      traefik.enable: true
      traefik.http.routers.hello-app.rule: Host(`hello-app.docker.localhost`)
      traefik.http.routers.hello-app.entrypoints: http
      traefik.http.routers.hello-app.service: hello-app
```

Notes:
- Replace `SIGNOZ_INGESTION_KEY` with the one provided by SigNoz.
- Replace `{region}` with the region of your SigNoz Cloud instance.
  Refer to the table below for the region-specific endpoints:

  | Region	| Endpoint                   |
  | ------- | -------------------------- |
  | US      | ingest.us.signoz.cloud:443 |
  | IN      | ingest.in.signoz.cloud:443 |
  | EU      | ingest.eu.signoz.cloud:443 |

</TabItem>
<TabItem value="self-host" label="Self-Host">

In this section, we will see how to export Traefik metrics and traces to SigNoz.

For metrics, we will have to set the following CLI flags in Traefik:

- `--metrics.openTelemetry=true`
- `--metrics.openTelemetry.grpc=true`
- `--metrics.openTelemetry.address=<SigNoz OtelCollector IP>:4317`
- `--metrics.openTelemetry.insecure=true`

For traces, we will have to set the following CLI flags in Traefik:

- `--tracing.openTelemetry=true`
- `--tracing.openTelemetry.grpc=true`
- `--tracing.openTelemetry.address=<SigNoz OtelCollector IP>:4317`
- `--tracing.openTelemetry.insecure=true`

Note: Replace `<SigNoz OtelCollector IP>` with the IP address or hostname of the host running SigNoz OtelCollector.

We will take an example `docker-compose.yaml` with a simple `hello-app`
running behind Traefik.

_docker-compose.yaml_

```yaml {13,17}
version: '3'
services:
  reverse-proxy:
    image: traefik:v3.0.0-beta3
    extra_hosts:
      - signoz:host-gateway
    command:
      - --api.insecure=true
      - --providers.docker
      - --metrics.openTelemetry=true
      - --metrics.openTelemetry.grpc=true
      - --metrics.openTelemetry.insecure=true
      - --metrics.openTelemetry.address=signoz:4317
      - --tracing.openTelemetry=true
      - --tracing.openTelemetry.grpc=true
      - --tracing.openTelemetry.insecure=true
      - --tracing.openTelemetry.address=signoz:4317
    ports:
      - "80:80"
      - "8080:8080"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
  hello-app:
    image: gcr.io/google-samples/hello-app:2.0
    environment:
      - PORT=8080
    labels:
      traefik.enable: true
      traefik.http.routers.hello-app.rule: Host(`hello-app.docker.localhost`)
      traefik.http.routers.hello-app.entrypoints: http
      traefik.http.routers.hello-app.service: hello-app
```

:::info
In case SigNoz is not running on the same host, you will have to replace `signoz`
with the IP address of the host running SigNoz.
:::

</TabItem>
</Tabs>

To start the services, run the following command:

```bash
docker-compose up -d
```

We will visit the `hello-app` service to generate some traffic.

```bash
curl -H Host:hello-app.docker.localhost http://127.0.0.1
```

Now, we will visit the SigNoz UI to see the traces and metrics.

![Traefik Traces](/img/docs/tutorial/traefik-traces.png)

To plot metrics generated from **Traefik**, follow the instructions
given in the docs [here][2].

Check out the [List of metrics from Traefik][3].

### List of Metrics

<TraefikMetrics />

---

[1]: https://signoz.io/docs/install/
[2]: https://signoz.io/docs/userguide/dashboards/
[3]: #list-of-metrics
[4]: https://signoz.io/teams/

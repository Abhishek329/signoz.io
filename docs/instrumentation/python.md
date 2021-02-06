---
id: python
title: OpenTelemetry Python Instrumentation
---

Get up and running with OpenTelemetry in just a few quick steps! The setup process consists of two phases--getting OpenTelemetry installed and configured, and then validating that configuration to ensure that data is being sent as expected. This guide explains how to download, install, and run OpenTelemetry in Python.

**Requirements**
- Python 3.4 or newer
- An app to add OpenTelemetry to

We follow [OpenTelemetry python instrumentation library](https://github.com/open-telemetry/opentelemetry-python/tree/master/opentelemetry-instrumentation). **We shall be exporting data in Jaeger Thrift protocol.**

```console
pip install opentelemetry-instrumentation
pip install opentelemetry-exporter-jaeger
```


This package provides a couple of commands that help automatically instruments a program:

```console
opentelemetry-bootstrap --action=install
```
This commands inspects the active Python site-packages and figures out which instrumentation packages the user might want to install and installs them for you.

#### opentelemetry-instrument
```console
OTEL_SERVICE_NAME=<service name> OTEL_EXPORTER_JAEGER_ENDPOINT="http://<IP of SigNoz Backend>:14268/api/traces" opentelemetry-instrument -e jaeger python program.py
```
Basically keep your run command after `-e jaeger` part:
```
OTEL_SERVICE_NAME=<service name> OTEL_EXPORTER_JAEGER_ENDPOINT="http://<IP of SigNoz Backend>:14268/api/traces" opentelemetry-instrument -e jaeger <your run command>
```

:::caution
Remember to allow incoming requests to port 14268 of machine where SigNoz backend is hosted
:::

### Troubleshooting your installation
If spans are not being reported to SigNoz, try running in debug mode by setting `OTEL_LOG_LEVEL=debug`:

```console
OTEL_LOG_LEVEL=debug OTEL_SERVICE_NAME=<service name> OTEL_EXPORTER_JAEGER_ENDPOINT="http://<IP of SigNoz Backend>:14268/api/traces" opentelemetry-instrument -e jaeger python program.py
```

The debug log level will print out the configuration information. It will also emit every span to the console, which should look something like:
```
Span {
  attributes: {},
  links: [],
  events: [],
  status: { code: 0 },
  endTime: [ 1597810686, 885498645 ],
  _ended: true,
  _duration: [ 0, 43333 ],
  name: 'bar',
  spanContext: {
    traceId: 'eca3cc297720bd705e734f4941bca45a',
    spanId: '891016e5f8c134ad',
    traceFlags: 1,
    traceState: undefined
  },
  parentSpanId: 'cff3a2c6bfd4bbef',
  kind: 0,
  startTime: [ 1597810686, 885455312 ],
  resource: Resource { labels: [Object] },
  instrumentationLibrary: { name: 'example', version: '*' },
  _logger: ConsoleLogger {
    debug: [Function],
    info: [Function],
    warn: [Function],
    error: [Function]
  },
  _traceParams: {
    numberOfAttributesPerSpan: 32,
    numberOfLinksPerSpan: 32,
    numberOfEventsPerSpan: 128
  },
  _spanProcessor: MultiSpanProcessor { _spanProcessors: [Array] }
},
```
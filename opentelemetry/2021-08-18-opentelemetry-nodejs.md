---
title: Monitor your Nodejs application with OpenTelemetry and SigNoz
slug: nodejs
date: 2021-08-18
tags: [opentelemetry, javascript-monitoring]
author: Ankit Anand
author_title: SigNoz Team
author_url: https://github.com/ankit01-oss
author_image_url: https://avatars.githubusercontent.com/u/83692067?v=4
description: In this article, learn how to setup application monitoring for Node.js apps with OpenTelemetry and SigNoz.
image: /img/blog/2021/08/opentelemetry_nodejs.jpeg
keywords:
  - opentelemetry
  - opentelemetry javascript
  - opentelemetry nodejs
  - distributed tracing
  - observability
  - nodejs monitoring
  - nodejs instrumentation
  - signoz
---

OpenTelemetry can auto-instrument many common modules for a Javascript application. The telemetry data captured can then be sent to SigNoz for analysis and visualization.

<!--truncate-->

import Screenshot from "@theme/Screenshot"

<Screenshot
  alt="Monitor your Nodejs applications with SigNoz"
  height={500}
  src="/img/blog/common/signoz_charts_application_metrics.png"
  width={700}
/>
OpenTelemetry is a set of tools, APIs, and SDKs used to instrument applications to create and manage telemetry data(Logs, metrics, and traces). For any distributed system based on microservice architecture, it's an operational challenge to solve performance issues quickly.

Telemetry data helps engineering teams to troubleshoot issues across services and identify the root causes. In other words, telemetry data powers observability for your distributed applications.

Steps to get started with OpenTelemetry for a Nodejs application:

- Installing SigNoz
- Installing sample Nodejs app
- Set up OpenTelemetry and send data to SigNoz

## Installing SigNoz

You can get started with SigNoz using just three commands at your terminal if you have Docker installed. You can read about other deployment options from [SigNoz documentation](https://signoz.io/docs/deployment/docker/).

```
git clone https://github.com/SigNoz/signoz.git
cd signoz/deploy/
./install.sh
```

You will have an option to choose between ClickHouse or Kafka + Druid as a storage option. Trying out SigNoz with ClickHouse database takes less than 1.5GB of memory, and for this tutorial, we will use that option.

When you are done installing SigNoz, you can access the UI at: http://localhost:3000

The application list shown in the dashboard is from a sample app called HOT R.O.D that comes bundled with the SigNoz installation package.

<Screenshot
  alt="SigNoz dashboard"
  height={500}
  src="/img/blog/common/signoz_dashboard_homepage.png"
  title="SigNoz dashboard"
  width={700}
/>

## Install sample Nodejs application

You need to ensure that you have **Node.js version 12 or newer**. You can download the latest version of Node.js [here](https://nodejs.org/en/download/). For the sample application, let's create a basic 'hello world' express.js application.

Steps to get the app set up and running:

1. Make a directory and install express<br></br>
   Make a directory for your sample app on your machine. Then open up the terminal, navigate to the directory path and install express with the following command:
   ```
   npm i express
   ```
2. Setup server.js<br></br>
   Create a file called 'server.js' in your directory and with any text editor setup your 'Hello World' file with the code below:

   ```
   const express = require('express');

   const app = express();

   app.get('/hello', (req, res) => {
   res.status(200).send('Hello World');
   });

   app.listen(9090);
   ```

3. Boot up the server with the following command on the terminal:

   ```
   node server.js
   ```

   You can check if your app is working by visiting: http://localhost:9090/hello

   Once you are finished checking, exit the localhost on your terminal.

## Set up OpenTelemetry and send data to SigNoz

1. In the same directory path at the terminal, install the OpenTelemetry launcher package with this command:

   ```
   npm install lightstep-opentelemetry-launcher-node
   ```

   The OpenTelemetry launcher makes getting started with OpenTelemetry easier by reducing configuration boilerplate.

2. To use OpenTelemetry, you need to start the OpenTelemetry SDK before loading your application. By initializing OpenTelemetry first, we enable OpenTelemetry to apply available instrumentation and auto-detect packages before the application starts to run. To do that, go to your directory and create a new file named, "server_init.js". This will act as the new entry point for your app. Paste the following code in the file:

   ```
   const {
    lightstep,
    opentelemetry,
   } = require('lightstep-opentelemetry-launcher-node');

   const sdk = lightstep.configureOpenTelemetry();

   sdk.start().then(() => {
    require('./server');
   });

   function shutdown() {
    sdk.shutdown().then(
      () => console.log("SDK shut down successfully"),
      (err) => console.log("Error shutting down SDK", err),
    ).finally(() => process.exit(0))
   };

   process.on('exit', shutdown);
   process.on('SIGINT', shutdown);
   process.on('SIGTERM', shutdown);
   ```

3. Once the file is created, you only need to run one last command at your terminal, which passes the necessary environment variables. Here, you also set SigNoz as your backend analysis tool.

   ```
   OTEL_EXPORTER_OTLP_SPAN_ENDPOINT="http://<IP of SigNoz Backend>:55681/v1/trace" OTEL_METRICS_EXPORTER=none LS_SERVICE_NAME=<service name> node server_init.js
   ```

   Replacing the placeholders in the above command for local host:

   `IP of SigNoz Backend`: localhost (since we are running SigNoz on our local host)

   `service name` : sample_app (you can give whatever name that suits you)

   So the final command is:

   ```
   OTEL_EXPORTER_OTLP_SPAN_ENDPOINT="http://localhost:55681/v1/trace" OTEL_METRICS_EXPORTER=none LS_SERVICE_NAME=sample_app node server_init.js
   ```

And, congratulations! You have instrumented your sample Node.js app. You can now access the SigNoz dashboard at http://localhost:3000 to monitor your app for performance metrics.
<Screenshot
  alt="Sample nodejs app in the applications monitored"
  height={500}
  src="/img/blog/2021/08/opentelemetry_nodejs_signoz_dashboard.png"
  title="Sample_app in the list of applications monitored"
  width={700}
/>

## Metrics and Traces of the Nodejs application

SigNoz makes it easy to visualize metrics and traces captured through OpenTelemetry instrumentation.

SigNoz comes with out of box RED metrics charts and visualization. RED metrics stands for:

- Rate of requests
- Error rate of requests
- Duration taken by requests

<Screenshot
  alt="Sample nodejs app in the applications monitored"
  height={500}
  src="/img/blog/common/signoz_charts_application_metrics.png"
  title="Measure things like application latency, requests per sec, error percentage and see your top endpoints"
  width={700}
/>

You can then choose a particular timestamp where latency is high to drill down to traces around that timestamp.

<Screenshot
      alt="See traces, and apply powerful filters on trace data"
      height={500}
      src="/img/blog/common/signoz_list_of_traces_hc.png"
      title="View of traces at a particular timestamp"
      width={700}
/>

You can use flamegraphs to exactly identify the issue causing the latency.

<Screenshot
      alt="Flamegraphs for distributed tracing"
      height={500}
      src="/img/blog/common/signoz_flamegraphs.png"
      title="Flamegraphs showing exact duration taken by each spans - a concept of distributed tracing"
      width={700}
/>

## Conclusion

OpenTelemetry makes it very convenient to instrument your Nodejs application. You can then use an open-source APM tool like SigNoz to analyze the performance of your app. As SigNoz offers a full-stack observability tool, you don't have to use multiple tools for your monitoring needs.

You can try out SigNoz by visiting its GitHub repo 👇<br></br>

[![SigNoz GitHub repo](/img/blog/common/signoz_github.png)](https://github.com/SigNoz/signoz)

If you want to read more about SigNoz 👇<br></br>

[Golang Application Performance Monitoring with SigNoz](https://signoz.io/blog/monitoring-your-go-application-with-signoz/)

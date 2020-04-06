## Welcome to the OpenTelemetry Glitch Collector Workshop!

In this Glitch workshop, we will configure the OpenTelemetry Collector. Note
this workshop is a prerequisite to the other language-specific instrumentation
workshops. A basic configuration YAML file can be found at
`otel-collector-config.yaml` with instruction on how to configure listed below.
Have suggestions on how to improve this lab? PRs welcomed!

## Collector Service

This Collector can be configured to listen on a variety of ports depending on
needs. For this workshop, we need to enable the `zipkin` receiver and have it
listen on port `3000` (443 when accessing from outside glitch). For every
request zipkin span receives, it should log the span information and optionally
send to a reachable back-end.

## Running the Collector

The Collector is available at
https://glitch.com/edit/#!/signalfx-otel-workshop-collector. By default, it is
configured to collect its own Prometheus metrics and log them locally. From the
Glitch site, you should select the name of the Glitch project (top left) and
select `Remix Project`. You will now have a new Glitch project. The name of the
project is listed in the top left of the window.

To run this workshop locally, you'll need Make to be able to run
the service. Run `make run` and check the log output.

## Enabling tracing

Your task is to enable tracing configuration for the [OpenTelemetry
Collector](https://github.com/open-telemetry/opentelemetry-collector). If you get
stuck, check out the `otel-collector-config-complete.yaml`.

### 1. Enable the Zipkin receiver

```diff
receivers:
+  zipkin:
+    endpoint: 0.0.0.0:3000
```

Note: Receivers default to an endpoint of `127.0.0.1`. In containerized
environments, you will need to change the endpoint to `0.0.0.0`. Also, while
receivers come with default ports, these can be changed as desired. Given
Glitch exposes port 3000 (as 443), we will set the Zipkin receiver to listen on
port `3000`.

### 2. Enable a tracing pipeline

```diff
service:
  extensions: [pprof, zpages, health_check]
  pipelines:
    metrics:
      receivers: [prometheus]
      exporters: [logging]
+    traces:
+      receivers: [zipkin]
+      processors: [batch, queued_retry]
+      exporters: [logging]
```

Note: While you can enable multiple receivers, Glitch only allows you to expose
a single port. In addition, it does not appear that Glitch supports HTTP/2
today and as such, it is not possible to enable gRPC receivers.

We can run the Collector again and this time it should listen for Zipkin spans
on port `3000`.

### 3. Optionally configure your back-end(s) of choice

Many open-source and commercial back-ends are available today. For more details see:

* https://github.com/open-telemetry/opentelemetry-collector/tree/master/exporter
* https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/master/exporter

---
title: Tracing FAQ
keywords: data, distributed tracing, jaeger, zipkin, spans
tags: [tracing]
sidebar: doc_sidebar
permalink: tracing_faq.html
summary: Get answers to your questions about Wavefront distributed tracing
---
This page has some special tips to help you instrument your applications and send trace data to Wavefront.

## How do I send custom span level RED metrics?

For details on how to send span-level RED metrics using a custom tracing port, see [Instrument Your Application with Wavefront Sender SDKs](tracing_instrumenting_frameworks.html#instrument-your-application-with-wavefront-sender-sdks).

## How do I filter RED metrics using custom span tags?

If you want to filter RED metrics data using a non-default span tag, propagate the tag as a custom span tag to the RED metrics. See [Custom Span-Level Tags for RED Metrics](tracing_customize_spans_and_alerts.html).

## How do I customize the application name (Zipkin or Jaeger)?

To override the default application name when using [zipkin-js](https://github.com/openzipkin/zipkin-js), OpenTracing Zipkin/Jaeger libraries, or OpenTelemetry Zipkin/Jaeger libraries, use any of the following options:

* Add the following span tag to the application logic.
  ```
  application=<applicationName>
  ```

* If you are using the Wavefront proxy and don't want to modify your code, open the [`<wavefront_config_path>`](proxies_configuring.html#paths)`/wavefront.conf` file, and update or add the following configuration.
  * Zipkin
    ```
    traceZipkinApplicationName = <Enter_Application_Name>
    ```
  * Jaeger
    ```
    traceJaegerApplicationName = <Enter_Application_Name>
    ```
  {% include note.html content="Only one application can send traces to a proxy because there’s only one application name per proxy. "%}
  
## How do I emit spans in the Wavefront format?

If you are not using a [Wavefront SDK](wavefront_sdks.html), OpenTracing library, or OpenTelemetry library, you can still send spans to Wavefront using the Wavefront proxy.

Send spans to the Wavefront proxy's `customTracingListenerPorts` port in the Wavefront spans format. Open the [`<wavefront_config_path>`](proxies_configuring.html#paths)`/wavefront.conf` file, and update or add the `customTracingListenerPorts` property.

Given below is an example of the Wavefront span format:
```
wavefrontSender.sendSpan("getAllUsers", System.currentTimeMillis(), 343, "localhost",
      UUID.fromString("7b3bf470-9456-11e8-9eb6-529269fb1459"),
      UUID.fromString("0313bafe-9457-11e8-9eb6-529269fb1459"),
      ImmutableList.<UUID>builder().add(UUID.fromString(
        "2f64e538-9457-11e8-9eb6-529269fb1459")).build(), null,
      ImmutableList.<Pair<String, String>>builder().
        add(new Pair<>("application", "Wavefront")).
        add(new Pair<>("service", "myService")).
        add(new Pair<>("http.method", "GET")).build(), null);
```

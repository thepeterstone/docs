---
title: Instrumenting Your App for Tracing
keywords: data, distributed tracing, OpenTelemetry
tags: [tracing]
sidebar: doc_sidebar
permalink: tracing_instrumenting_frameworks.html
summary: Set up your application to send metrics, histograms, and trace data to Wavefront.
---

You instrument your application so that [trace data](tracing_basics.html) from different parts of the stack are sent to Wavefront. Instrumentation enables you to trace a request from end to end across multiple distributed services, guided by key metrics from your application. After instrumentation, you can use our [tracing UI](tracing_ui_overview.html) to visualize a request as a trace that consists of a hierarchy of spans. This visualization helps you pinpoint where the request is spending most of its time, and discover problems.

You instrument each microservice in your application with one or more [Wavefront observability SDKs](wavefront_sdks.html). This page:
* Helps you choose the SDK(s) or integration
* Directs you to the setup steps for each SDK or integration

## Step 1. Prepare to Send Data to Wavefront

Choose one of the following ways to send metrics, histograms, and trace data from your application to the Wavefront service:
* **Direct Ingestion**. To get up and running quickly, use direct ingestion to send data directly to the Wavefront service.
* **Wavefront proxy**. For any production environment, we recommend a Wavefront proxy to forward data from your application to the Wavefront service. [Using a proxy](direct_ingestion.html#proxy-or-direct-ingestion) provides resilience to internet outages, control over data queuing and filtering, and more.

Watch [this video](https://youtu.be/Lrm8UuxrsqA) for some background on proxy vs. direct ingestion.

### To Prepare for Direct Ingestion

1. Identify the URL of your Wavefront instance. This is the URL you connect to when you log in to Wavefront, typically something like `https://mywavefront.wavefront.com`.
2. [Obtain an API token](wavefront_api.html#generating-an-api-token).


### To Prepare a Wavefront Proxy

1. On the host that will run the proxy, [install the proxy](proxies_installing.html#proxy-installation).
    {{site.data.alerts.note}}
      <ul>
      <li>If you want to use span logs, you need proxy version 5.0 or later.</li>
      <li>If you are using a Sender SDK to send data to Wavefront, you need proxy version 9.0 or later.</li>
      </ul>
    {{site.data.alerts.end}}
2. On the proxy host, open the proxy configuration file `wavefront.conf` for editing. The [path to the file](proxies_configuring.html#paths) depends on the host OS.
3. In the `wavefront.conf` file, find and uncomment the [listener-port property](proxies_installing.html#set-the-listener-port-for-metrics-histograms-and-traces) for each listener port you want to enable. The following example enables the default/recommended listener ports for metrics, histogram distributions, and trace data:
    ```
    pushListenerPorts=2878
    ...
    histogramDistListenerPorts=2878
    ...
    traceListenerPorts=30000
    ```
4. Consider setting up [trace sampling](trace_data_sampling.html) by [configuring the proxy with a sampling strategy](trace_data_sampling.html#setting-up-explicit-sampling-through-the-proxy).
5. Save the `wavefront.conf` file.
6. [Start the proxy](proxies_installing.html#starting-and-stopping-a-proxy).


## Step 2. Get Data Flowing into Wavefront

Wavefront provides SDKs that implement the [OpenTracing](https://opentracing.io) specification in many languages. You can use a Wavefront OpenTracing SDK to collect custom trace data that you define for your service, for example, to augment an auto-instrumented framework or to replace a 3rd party OpenTracing-compliant library.

{% include note.html content="If you can not find the SDK you were looking for, see all the [SDKs provided by Wavefront](wavefront_sdks.html#what-do-you-want-to-collect)." %}

{% include tip.html content="Wavefront can only retrieve up to 1000 spans for a given trace, and you only see up to 1000 spans when you [drill down into spans](tracing_ui_overview.html#drill-down-into-spans-and-view-metrics-and-span-logs) via the Tracing browser. Therefore, as a best practice and for optimal performance, configure your application to have less than 1000 spans in a trace." %}

### Instrument Your Application with OpenTracing SDKs

Choose the Wavefront OpenTracing SDK for your microservice's programming language, and click the link to go to its `README` file on GitHub:

<div class="row">
 <div class="col-md-3 col-sm-6">
     <div class="panel panel-default text-center">
         <div class="panel-body">
            <a href="https://github.com/wavefrontHQ/wavefront-opentracing-sdk-java">
            <img src="/images/icons_svg_java.png" alt="Java logo">
            </a>
         </div>
     </div>
 </div>
 <div class="col-md-3 col-sm-6">
     <div class="panel panel-default text-center">
         <div class="panel-body">
            <a href="https://github.com/wavefrontHQ/wavefront-opentracing-sdk-python">
            <img src="/images/icons_svg_phython.png" alt="Python">
            </a>
         </div>
     </div>
 </div>
 <div class="col-md-3 col-sm-6">
     <div class="panel panel-default text-center">
         <div class="panel-body">
            <a href="https://github.com/wavefrontHQ/wavefront-opentracing-sdk-go">
            <img src="/images/icons_svg_go.png" alt="Go">
            </a>
         </div>
     </div>
 </div>
 <div class="col-md-3 col-sm-6">
        <div class="panel panel-default text-center">
            <div class="panel-body">
               <a href="https://github.com/wavefrontHQ/wavefront-opentracing-sdk-csharp">
               <img src="/images/icons_svg_.net.png" alt="Net">
               </a>
            </div>
        </div>
    </div>
  </div>

### Instrument Your OpenTracing Java Application Without Writing Code

If you need application observability, but don't want to instrument code for your Java microservices, use the [Wavefront Java Tracing Agent](https://github.com/wavefrontHQ/wavefront-opentracing-bundle-java). For more information, see [this blog post on the Wavefront Java Tracing Agent](https://www.wavefront.com/wavefront-tracing-agent-for-java/).

<div class="row">
   <div class="col-md-3 col-sm-6">
       <div class="panel panel-default text-center">
           <div class="panel-body">
              <a href="https://github.com/wavefrontHQ/wavefront-opentracing-bundle-java">
              <img src="/images/icons_svg_java_tracing_agent.png" alt="Java tracing agent">
              </a>
           </div>
       </div>
   </div>
 </div>

### Send Trace Data to Wavefront via Applications Integrated with Jaeger or Zipkin

If you have already instrumented your application with Jaeger or Zipkin follow the steps given below:
  1. Collect traces send them to Wavefront using the following integrations.

      <div class="row">
       <div class="col-md-3 col-sm-6">
           <div class="panel panel-default text-center">
               <div class="panel-body">
                  <a href="jaeger.html">
                  <img src="/images/icons_svg_jaeger.png" alt="Jaeger" class="center">
                  </a>
               </div>
           </div>
       </div>
       <div class="col-md-3 col-sm-6">
           <div class="panel panel-default text-center">
               <div class="panel-body">
                  <a href="zipkin.html">
                  <img src="/images/icons_svg_zipkin.png" alt="Zipkin" class="center">
                  </a>
               </div>
           </div>
       </div>
     </div>

     
     {% include note.html content="You can use OpenTracing or OpenTelemetry (OpenTracing and OpenCensus have merged to form OpenTelemetry) to send traces to Wavefront using the Jaeger or Zipkin integration. See [OpenTelemetry](opentelemetry.html#sending-trace-data-to-wavefront) for details." %}
     
 2. Optionally, add custom tags, applications names, or use an alternative for the Jaeger or Zipkin integration. See [Using Jaeger or Zipkin with Wavefront](tracing_integrations.html) for details.

After your recompiled application starts running, start [exploring your custom trace data](tracing_ui_overview.html) and the [metrics and histograms that are automatically derived](trace_data_details.html#red-metrics) from your trace data.

### Instrument Your Application with Wavefront Sender SDKs

For maximum flexibility, you can use the Wavefront Sender SDKs. See [SDKs for Sending Raw Data to Wavefront](wavefront_sdks.html#sdks-for-sending-raw-data-to-wavefront) for background.

<div class="row">
 <div class="col-md-2 col-sm-6">
     <div class="panel panel-default text-center">
         <div class="panel-body">
            <a href="https://github.com/wavefrontHQ/wavefront-sdk-java">
            <img src="/images/icons_svg_java.png" alt="Java logo">
            </a>
         </div>
     </div>
 </div>
 <div class="col-md-2 col-sm-6">
     <div class="panel panel-default text-center">
         <div class="panel-body">
            <a href="https://github.com/wavefrontHQ/wavefront-sdk-python">
            <img src="/images/icons_svg_phython.png" alt="Python">
            </a>
         </div>
     </div>
 </div>
 <div class="col-md-2 col-sm-6">
     <div class="panel panel-default text-center">
         <div class="panel-body">
            <a href="https://github.com/wavefrontHQ/wavefront-sdk-go">
            <img src="/images/icons_svg_go.png" alt="Go">
            </a>
         </div>
     </div>
 </div>
 <div class="col-md-2 col-sm-6">
     <div class="panel panel-default text-center">
         <div class="panel-body">
            <a href="https://github.com/wavefrontHQ/wavefront-sdk-csharp">
            <img src="/images/icons_svg_.net.png" alt="Net">
            </a>
         </div>
     </div>
 </div>
 <div class="col-md-2 col-sm-6">
     <div class="panel panel-default text-center">
         <div class="panel-body">
            <a href="https://github.com/wavefrontHQ/wavefront-sdk-cpp">
            <img src="/images/icons_cplus.png" alt="C++">
            </a>
         </div>
     </div>
 </div>
</div>

When you use a Sender SDK, you won’t see span-level RED metrics by default. This section explains how to send span-level RED metrics using a custom tracing port.

1. [Prepare to send data via the Wavefront proxy](#to-prepare-a-wavefront-proxy).
    {% include note.html content="You need proxy version 9.0 or later."%}
1. Configure your application to send data via the Wavefront Proxy. See the SDK’s README file for details.
1. Specify the port or a comma-separated list of ports that you want to send the trace data using the `customTracingListenerPorts` configuration on your [`<wavefront_config_path>`](proxies_configuring.html#paths)`/wavefront.conf` file.
  ```xml
  ## port for sending custom spans using the sender sdk
  ## you can use a port that you prefer. in this example port 30001 is used
  customTracingListenerPorts=30001
  ```
1. When you configure the `wavefront sender` on your application as explained in the SDK’s README file, define the port you want to send the data so that span level RED metrics will be gathered from your application.
  <br/>Example: Configuring your Java application to send trace data via the `tracingPort`.
    ```java
    // set up wavefront sender for Proxy based ingestion
    WavefrontSender wavefrontSender = new WavefrontClient.Builder(proxyHost).
            tracingPort(30001). // customTracingListenerPorts configured in your wavefront.conf file
            metricsPort(metricsPort).
            distributionPort(distributionPort).
            messageSizeBytes(messageSizeInBytes).
            batchSize(batchSize).
            flushIntervalSeconds(flushIntervalSeconds).
            maxQueueSize(queueSize).
            build(); // Returns a WavefrontClient

    // now send distributed tracing spans as below
    wavefrontSender.sendSpan("getAllUsers", 1552949776000L, 343, "localhost",
          UUID.fromString(UUID.randomUUID()),
          UUID.fromString(UUID.randomUUID()),
          ImmutableList.<UUID>builder().add(UUID.fromString(
            "2f64e538-9457-11e8-9eb6-529269fb1459")).build(), null,
          ImmutableList.<Pair<String, String>>builder().
            add(new Pair<>("application", "Wavefront")).
            add(new Pair<>("http.method", "GET")).build(), null);
    ```

## Next Steps

Examine traces, spans, and RED metric sent by your application. See [Visualizing Traces, Spans, and RED Metrics](tracing_ui_overview.html) for details.

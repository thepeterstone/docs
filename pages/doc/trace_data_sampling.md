---
title: Trace Sampling
keywords: data, distributed tracing
tags: [tracing]
sidebar: doc_sidebar
permalink: trace_data_sampling.html
summary: Learn about sampling for Wavefront trace data.
---

A cloud-scale web application generates a very large number of [traces](tracing_basics.html#wavefront-trace-data). Wavefront supports sampling to reduce the volume of stored trace data: 

* Wavefront automatically performs intelligent sampling on the traces that it receives, and retains only those traces that are most likely to be informative.  
* You can further reduce the trace data that Wavefront receives by adding explicit sampling strategies.

Sampling can give you a good idea of how your application is behaving. In addition, sampling can: 
* Reduce the amount of storage required for trace data, and lowering your monthly costs.
* Filter out "noise" traces so you can see what's important.
* Limit the performance impact on network bandwidth and application response times.

{% include note.html content="You can configure Wavefront to keep spans in storage for 7 or 30 days. Contact [support@wavefront.com](mailto:support@wavefront.com) to configure your span storage settings."  %}

## Wavefront Intelligent Sampling

Wavefront automatically performs intelligent sampling to reduce the volume of ingested traces. The goals of intelligent sampling are to retain traces that are likely to be informative, and to discard traces that are redundant or otherwise not worth inspecting. 

Intelligent sampling gives preference to: 

* Traces that are abnormally long, as compared to other traces for the same endpoint. 
* Traces that contain at least one individual span that is abnormally long, as compared to other spans for the same operation.
* Traces that contain at least one span in which an error occurred.

Wavefront uses proprietary algorithms to decide which traces to retain (sample) and which traces to discard (not sample). When analyzing whether a trace is worth retaining, Wavefront compares the trace's characteristics to a historical context that is composed of similar traces. The historical context is based on the [RED metrics](trace_data_details.html#trace-sampling-and-derived-red-metrics) that Wavefront derives from the entire set of trace data that your application has emitted before any sampling occurs. This enables Wavefront to determine whether an analyzed trace is a true outlier.

Intelligent sampling applies to entire traces after Wavefront receives them. If you have set up an [explicit sampling strategy](#explicit-sampling-strategies), then the output of your explicit sampling strategy is the input to intelligent sampling. 

Intelligent sampling is performed by the Wavefront service itself, not by the proxy or by an instrumented application. Consequently, intelligent sampling does not place any additional processing burden on your proxies or applications. Intelligent sampling does not add to your total cost of operation (TCO). If you already use one or more proxies to ingest your time-series data, you can start ingesting and sampling trace data without adding more hardware to support more proxies. 

{% include note.html content="If you are troubleshooting and need specific spans in wavefront, annotate spans with `debug=true`. Make sure to remove the annotation once you are done troubleshooting and make sure not to overuse the annotation as Wavefront intelligent sampling gives preference to unique spans." %}

You can [monitor](wavefront_monitoring.html#using-internal-metrics-to-optimize-performance) your span storage by checking the following internal metrics. If you have set up sampling, these metrics report the number of spans after sampling takes place.
<table width="100%">
<colgroup>
<col width="50%"/>
<col width="50%"/>
</colgroup>
<thead>
<tr><th>Metric</th><th>Description</th></tr>
</thead>
<tbody>
<tr>
<td markdown="span">`~collector.tracing.spans.reported`</td>
<td markdown="span">Number of spans per second being sent via a Wavefront proxy.</td>
</tr>
<tr>
<td markdown="span">`~collector.direct-ingestion.tracing.spans.reported`</td>
<td markdown="span">Number of spans per second being sent directly to the Wavefront service (direct ingestion).</td>
</tr>
</tbody>
</table>


## Explicit Sampling Strategies

A explicit sampling strategy is a mechanism for selecting which traces to forward to Wavefront. Wavefront supports the following explicit sampling strategies: 

<table>
<colgroup>
<col width="25%"/>
<col width="75%"/>
</colgroup>
<thead>
<tr><th>Sampling Strategy</th><th>Description</th></tr>
</thead>
<tbody>
<tr>
<td markdown="span">Rate-based sampling</td>
<td markdown="span">Sends N percent of the generated traces to Wavefront. Sometimes called probabilistic sampling. For example, a sampling rate of 10% causes 1 out of 10 traces to be sent and ingested.</td>
</tr>
<tr>
<td markdown="span">Duration-based sampling</td>
<td markdown="span"> Sends spans to Wavefront only if they are longer than N milliseconds. For example, a sampling duration of 45 sends spans to Wavefront only if they are longer than 45 milliseconds.</td>
</tr>
<tr>
<td markdown="span">Error-based sampling</td>
<td markdown="span"> Sends a span to Wavefront if it contains an error. A span contains an error if it is associated with the span tag `error=true`. </td>
</tr>
</tbody>
</table>

**Note:** You can query and visualize only the traces and spans that Wavefront has actually received and ingested. If you set up an explicit sampling strategy that severely reduces the volume of ingested trace data, you could end up with queries that produce no results.

### Ways to Set Up Explicit Sampling Strategies 

You can set up an explicit sampling strategy using either of the following methods:

* [Configure sampling on a Wavefront proxy](#setting-up-explicit-sampling-through-the-proxy).  
* [Configure sampling in your instrumented application code](#setting-up-explicit-sampling-in-your-code).  

Choose the [Wavefront proxy](proxies_installing.html) for sampling when you want to:
* Use a single sampling strategy to coordinate the sampling for all applications that use the same proxy. 
* Configure sampling with minimal effort.
* Improve the likelihood of ingesting complete traces. 

Choose sampling in your instrumented code when you want to:
* Reduce the performance impact of span reporting on your application. 
* Use [direct ingestion](direct_ingestion.html) (no proxy).
* Configure sampling on a per-process basis, for example, when you expect spans from the services in different processes to have different characteristics.

### Complete vs. Partial Traces

An ingested trace can be complete (a trace ingested with all of its member spans) or partial (a trace that is missing one or more spans). The completeness of the traces in a sample depends in part on the sampling strategy:

* Rate-based sampling attempts to send complete traces. That is, the sampler selects the specified percentage of trace IDs, and then sends all of the spans that belong to each selected trace. 

* Duration-based sampling considers only individual spans. That is, the sampler selects all spans of an appropriate duration, regardless of whether they form complete traces.

Partial traces can also occur in the following situations:
* If a span contains an error. Each such span is sent individually, without the other spans in the same trace.
* If a trace has spans from multiple services, and you set up different sampling rates for those services. 

### Result of Combining Explicit Sampling Strategies

You can combine rate-based sampling and duration-based sampling in the same service. Doing so causes Wavefront to ingest the union of the spans that are selected by each sampler.

For example, suppose you set the sampling rate to 20% and the sampling duration to 45ms for the same service. This causes Wavefront to receive:
* 20% of the traces generated by that service, regardless of the length of their spans.
* Any additional spans outside of that 20% that are longer than 45ms. 

As a result, the ingested sample will contain somewhat more than 20% of the generated traces, with some spans that are shorter than 45ms.

{% include note.html content="A span that contains an error is always sent to Wavefront, the regardless of the span's duration or whether it falls in a specified sampling percentage. " %}

## Setting Up Explicit Sampling Through the Proxy

You can set up explicit sampling strategies through a [Wavefront proxy](proxies.html) by adding the sampling properties to the proxy's configuration file.
{% include note.html content="Explicit sampling through a proxy is supported in Wavefront proxy version 4.34 and above. If you have an older version, make sure to [upgrade your proxy to the latest version](proxies_installing.html#upgrade-a-proxy)."  %}

1. On the proxy host, open the proxy configuration file `wavefront.conf` for editing. The [path to the file](proxies_configuring.html#paths) depends on the host. 
2. Add the [traceSamplingRate](proxies_configuring.html#tracing-proxy-properties-and-examples) property, the [traceSamplingDuration](proxies_configuring.html#tracing-proxy-properties-and-examples) property, or both to the `wavefront.conf` file. In the following example, the `traceSamplingRate` property sends 10% of the spans that make up the trace, to Wavefront and the `traceSamplingDuration` property sets the minimum sampling duration to 45 milliseconds: 
    ```
    # Number from 0.0 to 1.0
    traceSamplingRate=.1
    ...
    traceSamplingDuration=45
    ```
    {% include important.html content="If you have more than one proxy, each proxy must have the same value for the `traceSamplingRate` property. If different proxies send different percentages of spans to Wavefront, you get incomplete traces."%}
3. Save the `wavefront.conf` file. 
4. [Start the proxy](proxies_installing.html#starting-and-stopping-a-proxy).

## Setting Up Explicit Sampling in Your Code

You can set up explicit sampling strategies in application code that is built with one of the following [Wavefront observability SDKs](wavefront_sdks.html):

* The Wavefront OpenTracing SDK
* Any Wavefront observability SDK that depends on the Wavefront OpenTracing SDK

You set up a sampling strategy by configuring a Wavefront Tracer with a Sampler object. You create one Sampler for each sampling strategy. See the README file for the Wavefront observability SDK you are using.  


<!---
<table>
<colgroup>
<col width="18%"/>
<col width="50%"/>
<col width="32%"/>
</colgroup>
<thead>
<tr><th>Menu</th><th>Description</th><th>Example</th></tr>
</thead>
<tbody>
<tr>
<td markdown="span"> </td>
<td markdown="span"> </td>
<td markdown="span"> </td>
</tr>
</tbody>
</table>


--->

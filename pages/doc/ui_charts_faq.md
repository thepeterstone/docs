---
title: Wavefront Charts FAQ (v2)
tags: [getting started, dashboards, charts]
sidebar: doc_sidebar
permalink: ui_charts_faq.html
summary: Learn chart customization from the experts.
---

[Create and Customize Charts](ui_charts_v2.html) describes most of the things you need to know to get started. This page has some special tips and tricks to help you create the user experience you're after.

{% include shared/badge.html content="You must have [Dashboard permission](permissions_overview.html) to save a chart to a dashboard. If you do not have permission, UI menu selections and buttons required to perform the task are not visible." %}

<!--- Consider including Improve Display Speed with Sampling Option here --->

## How Do I Set Up Color Mapping?

Color mapping is a powerful way to get users' attention when there's a problem. We support color mapping for the following charts:
* Single stat
* Topk
* Node map

The following example shows how to use color mapping with a single stat chart.

<table style="width: 100%;">
<tbody>
<tr>
<td width="50%">
<ol>
<li>In the Data tab, specify the metrics to monitor and give a subtitle. </li>
<li>In the Sparkline tab:</li>
<ol><li>Select whether to apply the color to the Text or the Background and change <strong>Show Sparkline</strong> to <strong>No Line</strong> if you want a solid color chart. </li>
<li>Click the <strong>+</strong> next to <strong>Color to Value Mapping</strong> and chage the values and colors to fit your use case. In the example on the right, we show green for values under 75, yellow for values under 80, and red for values over 80. </li>
<li>Save your chart. </li></ol>
</ol></td>
<td width="50%"><img src="/images/color_mapping_faq.png" alt="create dashboard"></td>
</tr>
</tbody>
</table>

## How Do Drilldown Links Work?


A drilldown link sends users to a target dashboard when they click on a chart. We support drilldown links for the following chart types:
* Single stat
* Topk
* Node map

You can optionally pass along a constant, point tag value, or other value to be used in the target dashboard. Here's an example:

Suppose your users monitor 2 dashboards:
* Dashboard 1 consists of a set of single stat charts that monitor important values and change color as critical thresholds are crossed for an availability zone. Each chart is for one availability zone only. Each chart sets the `az` point tag to show only the value for that zone, for example:

  ![query for drilldown](images/drilldown_0.png)

* Dashboard 2 allows users to get details about the different availability zones. A variable (Availability Zone) is defined for that dashboard, and users can select a value for that variable.

* Inside dashboard 1, you've defined a drilldown link for each single stat chart that:
  - Goes to dashboard 2 when the user clicks one of the single-stat charts.
  - Passes the value of the `az` point tag in as the value of the `az` variable.
  **Note** A variable value that matches to the point tag value must exist in dashboard 2. However, the point tag name and the variable name do not have to match.

  ![drilldown_definition](images/drilldown_1.png)

* When the user clicks on a chart in dashboard 1 because it shows a critical value, the user is redirected to dashboard 2, and the variable is preset to show the environment that has the problem.

  ![drilldown_target](images/drilldown_2.png)



## Any Other Doc (or Videos)?

* Get the details about each chart type from the [Chart Reference](ui_chart_reference_v2.html).
* Send [a link to a chart](ui_sharing.html#share-a-link-to-a-dashboard-or-chart) to a coworker (or to the customer success team if you need help).
* [Embed a chart](ui_sharing.html#embed-a-chart-in-other-uis) outside Wavefront.
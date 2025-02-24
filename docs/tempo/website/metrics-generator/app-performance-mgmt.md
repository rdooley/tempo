---
title: Application performance management
menuTitle: APM dashboard
weight: 200
---

# Application performance management dashboard

<span style="background-color:#f3f973;">This is an experimental feature. For more information about how to enable it, continue reading.</span>

Application performance management (APM) uses data gathered from services, agents, systems, and microservices to visualize and identify areas of concern.

The Grafana APM dashboard visualizes the span metrics (traces data for rates, error rates, and durations (RED)) and service graphs.
Once the requirements are set up, this pre-configured dashboard is immediately available.  

You can use the APM dashboard to:

* Discover spans which are consistently erroring and the rates at which they occur
* Get an overview of the overall rate of span calls throughout your services
* Determine how long the slowest queries in your service take to complete
* Examine all traces that contain spans of particular interest based on rate, error and duration values (RED signals)

<p align="center"><img src="../../getting-started/apm-overview.png" alt="APM dashboard"></p>

## Requirements to enable the APM dashboard

You have to enable span metrics and service graph generation on the Grafana backend so metrics that are generated as traces are ingested.

To use the APM dashboard, you need:

* Tempo or Grafana Cloud Traces with either 1) the metrics generator enabled and configured or 2) the Grafana Agent enabled and configured to send data to a Prometheus-compatible metrics store
* [Services graphs]({{< relref "../metrics-generator/service_graphs/#how-to-run" >}}), which are enabled by default in Grafana
* [APM table enabled](https://grafana.com/docs/grafana/latest/datasources/tempo/) within the service graph tab (use the `tempoApmTable` feature flag in Grafana to enable)
* [Span metrics]({{< relref "../metrics-generator/span_metrics/#how-to-run" >}}) enabled in your Tempo data source configuration

The APM dashboard can be generated with the metrics-generator or Grafana Agent.

For information on how to configure these features, refer to the [Grafana Tempo data sources documentation](https://grafana.com/docs/grafana/latest/datasources/tempo/).

## APM dashboard

Using this dashboard, you can see the top five spans with a type of server (listed in the `Name` column).  
You can refine any of this data using the filters.
Selecting any of the data points lets you see more specific data.

The APM dashboard provides a span metrics visualization (APM table, screen section 2) and service graph (screen section 3). In addition, you can use the filters (screen section 1) to customize the data displayed.

<p align="center"><img src="../apm-overview-numbered.png" alt="APM dashboard with numbered sections"></p>

Any information in the APM table that has an underline can be selected to show more detailed information.
You can also select any node in the service graph to display additional information.
In the dashboard shown below, the `Ingester.QueryStream` span has a request rate of `144220.22` requests per second.
The `/cortex.Ingester/Query` span has the highest request rate.

### Error rate example

Let’s say we want to learn more about why `cortex.Ingester` has the highest error rates.
Selecting the second row of the Error rate column displays details about the span metrics in a new window on the right side.

<p align="center"><img src="../apm-error-rate-example.png" alt="APM error rate example"></p>

The metrics query used to generate the data appears in the **Metrics browser** field.

<p align="center"><img src="../apm-error-example-editor.png" alt="APM error example query editor"></p>

## Span metrics table

The span metrics, also called the APM table, are generated by the metrics-generator or the Grafana Agent.
These metrics are created from ingested tracing data, including RED metrics.

Span metrics generate two metrics:

* A counter that computes requests
* A histogram that tracks the distribution of durations of all requests

For information about span metrics and how they are calculated, refer to the [Span metrics documentation]({{< relref "../metrics-generator/span_metrics.md" >}}).

<p align="center"><img src="../apm-span-metrics.png" alt="APM span metrics table"></p>

### APM table contents

The table contains seven columns with five column headings. Selecting a heading sorts the data by ascending or descending values.

| **Column** | **Explanation** | **PromQL query for span** |
| --- | --- | --- |
| Name | Use the span name. OTel semantic conventions generally expect the span name to be some kind of low cardinality indicator of the http route or database function being performed. | N/A |
| Rate | LCD gauge (horizontal bar graph). Instances per second of the span. Clicking this field can jump to the appropriate metrics. |`sum(rate(  traces_spanmetrics_calls_total{ span_name="", <filters> }[$__range]))` |
| Error Rate | Number and LCD gauge (horizontal bar graph). Clicking this field shows more detailed metrics. | `sum(rate(  traces_spanmetrics_calls_total{ span_name="",   span_status="STATUS_CODE_ERROR", <filters> }[$__range]))` |
| Duration | p90 duration: 90% of all occurrences of this span complete within this time. Clicking this field shows the appropriate metrics. | `histogram_quantile(.9, sum(rate(  traces_spanmetrics_duration_seconds_bucket{ span_name="", span_status="STATUS_CODE_ERROR",  <filters> }[$__range]) by (le))` |
| Links | Provide links to example traces given the span name and other applied filters. Link to a search for all spans with the same name from the same Tempo data source. | N/A |

## Service graphs

A service graph (node graph) is a visual representation of the interrelationships between various services.
Service graphs help to understand the structure of a distributed system, and the connections and dependencies between its components.

Service graphs infer the topology of a distributed system, provide a high level overview of the health of your system, and a historic view of a system’s topology.
Service graphs show error rates and latencies, among other relevant data.
The service graph layout can be the default or grid. 

<p align="center"><img src="../apm-service-graph-web.png" alt="APM service graph with a connected node layout"></p>

The grid layout changes the service graph to a series of rows and columns. 

<p align="center"><img src="../apm-service-graph-rows.png" alt="APM service graph with grid layout"></p>

If you are using the metrics-generator, then it processes traces and generates service graphs in the form of time series metrics like:

```yaml
tempo_service_graph_request_total{client="app", server="db"} 20
```

For information about service graphs and how they are calculated, refer to the [Service Graphs documentation]({{< relref "../metrics-generator/service_graphs.md" >}}).

## Use filters to reveal details

The APM dashboard uses service graphs and span metrics to provide a gateway to your tracing information.
This dashboard is derived from a fixed set of metrics queries.
These underlying queries can not be changed.
However, you can choose which traces are included in the metrics query by filtering.

You can explore data by clicking on selectable items or by using filters.

### Selecting items or nodes for more detail

Clicking on selectable items, such as underlined text in the APM table or nodes on the service graph, lets you reveal specific details based upon your selection.

In the APM table, you can select items in the **Rate**, **Error Rate**, **Duration (p90)**, and **Links** columns. Choosing one of these items provides details about the span metrics.

<p align="center"><img src="../apm-rate-drilldown.png" alt="APM table with rate drill-down"></p>


You can view request rate, request histogram, failed request rate, and traces for any node in the service graph.
To view more information, select the node in the service graph and then choose an option from the popup.
For details on navigating the service graph, refer to the [Node graph panel](https://grafana.com/docs/grafana/latest/visualizations/node-graph/) documentation.

<p align="center"><img src="../apm-service-graph-drilldown.png" alt="APM service graph with drill-down"></p>


### Filter with metric queries

Using the filters at the top of the screen, you can narrow the data set based upon span attributes (key-value pairs or labels).
The filters build a query to refine what is shown in the service graph and span metrics.
You can add one or more label filters.

To use the filters:

1. At the top of the Service Graph, select the text box after **Filter** to display a list of available labels. In this case, **server** is selected. <br /><img src="../apm-query-filter-label.png" alt="APM service graph with grid layout">

1. Select or search for a value for the label. In this case, the value of **server** is equal to **tempo-ingester**. The default operator is equals (=). <br /> <img src="../apm-query-filter-value1.png" alt="APM select value for label">

1. Optional: Change the operator by selecting **=** and choosing a new option from the drop-down. <br /><img src="../apm-filter-operator.png" alt="APM filter operators">


1. Optional: Add additional key-value pairs to refine the data set. Any subsequent label filters use AND, which requires both key-value pairs to be presents for matches.

1. Select **Run query**.

Filters can be removed by selecting the filter drop-down and choosing **– remove filter –**.

<p align="center"><img src="../apm-remove-filter.png" alt="APM remove filters"></p>

In the example below, each field or label represents a key-value pair. Number 1 selects a service as the label whose value is `Go-http-client` (2). The second key-value pair has a client as a label whose value is `02e807`.

<p align="center"><img src="../apm-filter-example-numbered.png" alt="APM filter example with numbers"></p>

If your metrics queries are too specific, they may not return any results. 

Updating the filter to be less specific returns a result. In this case, the results show only span metrics data associated with the `span_name` label with a value of `/base.Ruler/Rules`. No service graph data was available.

<p align="center"><img src="../apm-filter-example2.png" alt="APM filter example with one results"></p>
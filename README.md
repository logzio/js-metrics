## Logz.io nodejs metrics sdk

This topic includes instructions on how to send custom metrics to Logz.io from your Node.js application.

The included example uses the [OpenTelemetry JS SDK](https://github.com/open-telemetry/opentelemetry-js) and its based on [OpenTelemetry exporter collector proto](https://github.com/open-telemetry/opentelemetry-js/tree/main/packages/opentelemetry-exporter-collector-proto).

**Before you begin, you'll need**:
Node 14 or higher

**Note** This project works best with logzio as metrics backend, but its compatible with all backends that support `prometheuesrmotewrite` format

## Quick start

Install the package:

```
npm install logzio-nodejs-metrics-sdk@0.4.0
npm install @opentelemetry/sdk-metrics-base@0.27.0
```

Set the variables in the following code snippet:

| Environment variable | Description                                                                                                                                                                     |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| url                  | The Logz.io Listener URL for for your region, configured to use port **8052** for http traffic, or port **8053** for https traffic. For example - https://listener.logz.io:8053 |
| token                | Your Logz.io Prometheus Metrics account token.                                                                                                                                  |

```js
const { MeterProvider, PeriodicExportingMetricReader } = require('@opentelemetry/sdk-metrics');
const sdk = require('logzio-nodejs-metrics-sdk');

const collectorOptions = {
    url: '<<url>>',
    headers: {
        Authorization: 'Bearer <<token>>',
    },
};
// Initialize the exporter
const metricExporter = new sdk.RemoteWriteExporter(collectorOptions);

// Initialize the meter provider
const meter = new MeterProvider({
    readers: [
        new PeriodicExportingMetricReader(
            {
                exporter: metricExporter, 
                exportIntervalMillis: 1000
            })
        ],
}).getMeter('example-exporter');

// Create your first counter metric
const requestCounter = meter.createCounter('Counter', {
    description: 'Example of a Counter',
});
// Define some labels for your metrics
const labels = { environment: 'prod' };
// Record some value
requestCounter.add(1, labels);
// In logzio Metrics you will see the following metric:
// Counter_total{environment: 'prod'} 1.0
```

### Types of metric instruments

For more information, see the OpenTelemetry [documentation](https://github.com/open-telemetry/opentelemetry-specification/blob/main/specification/metrics/api.md).

| Name          | Behavior                                                                                                               |
| ------------- | ---------------------------------------------------------------------------------------------------------------------- |
| Counter       | Metric value can only go up or be reset to 0, calculated per `counter.Add(context,value,labels)` request.              |
| UpDownCounter | Metric value can arbitrarily increment or decrement, calculated per `updowncounter.Add(context,value,labels)` request. |
| Histogram     | Metric values captured by the `histogram.Record(context,value,labels)` function, calculated per request.               |

### More examples

First Initialize the exporter and meter provider:

```js
const { MeterProvider, PeriodicExportingMetricReader } = require('@opentelemetry/sdk-metrics');
const sdk = require('logzio-nodejs-metrics-sdk');

const collectorOptions = {
    url: '<<url>>',
    headers: {
        Authorization: 'Bearer <<token>>',
    },
};
// Initialize the exporter
const metricExporter = new sdk.RemoteWriteExporter(collectorOptions);

// Initialize the meter provider
const meter = new MeterProvider({
    readers: [
        new PeriodicExportingMetricReader(
            {
                exporter: metricExporter, 
                exportIntervalMillis: 1000
            })
        ],
}).getMeter('example-exporter');
```

Then create different types of metrics

#### UpDownCounter:

```js
// Create UpDownCounter metric
const upDownCounter = meter.createUpDownCounter('UpDownCounter', {
    description: 'Example of a UpDownCounter',
});
// Define some labels for your metrics
const labels = { environment: 'prod' };
// Record some values
upDownCounter.add(5, labels);
upDownCounter.add(-1, labels);
// In logzio you will see the following metric:
// UpDownCounter{environment: 'prod'} 4.0
```

#### Histogram:

```js
// Create ValueRecorder metric
const histogram = meter.createHistogram('test_histogram', {
    description: 'Example of a histogram',
});
// Define some labels for your metrics
const labels = { environment: 'prod' };
// Record some values
histogram.record(30, labels);
histogram.record(20), labels;
// In logzio you will see the following metrics:
// test_histogram_sum{environment: 'prod'} 50.0
// test_histogram_count{environment: 'prod'} 2.0
// test_histogram_avg{environment: 'prod'} 25.0
```

## Update log
**0.5.0**
- Update dependencies versions
  - Upgrade to OTEL packages in version `1.26.0`
- Update docs 
- Drop support for Nodejs `12.*`

**0.4.0**

Breaking changes:
-   Drop support for Nodejs `8.*` `10.*`

Changelog:
-   Fix bug which makes it crash if no metrics created/written before the first interval (@chapost1)
-   Using `axios` instead of `requestretry` (@chapost1)

**0.3.0**

-   Add github action for auto publish to npm
-   Add option to update TimeUnixNano in metrics from Exporter

**0.2.0**

-   Update otel dependencies and naming conventions
-   Update docs
-   Fix exporting modules names

**0.1.0**

-   Initial Release

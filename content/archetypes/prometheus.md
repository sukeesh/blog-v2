---
layout: post
date: 2020-03-05
author: "Sukeesh"
title: "Setting up and Troubleshooting Prometheus in Kubernetes"
---

----------
### Prometheus

Prometheus is an open-source systems monitoring and alerting toolkit. It scrapes metrics from instrumented jobs, either directly or via an intermediary push gateway for short-lived jobs. It stores all scraped samples locally and runs rules over this data to either aggregate and record new time series from existing data or generate alerts. [link](https://prometheus.io/docs/introduction/overview/)


I tried to make this blog post as simple as possible and it has all the basic components required to setup Prometheus successfully in your microservice.

----

### Architecture

Ignore this section if it's too confusing to understand

![Prom](https://prometheus.io/assets/architecture.png "Architecture") [ref](https://prometheus.io/docs/introduction/overview/)

----

### Prometheus Client (Python)

```py
from prometheus_client import start_http_server, Counter
import random
import time

c = Counter('count', 'Description of counter')

if __name__ == '__main__':
    start_http_server(8000)
    while True:
      c.inc(0.1)
      time.sleep(random.random())
```

Above is a simple http server which has a Prometheus Counter implemented. Counter is incremented by `0.1` everytime `c.inc(0.1)` is called. Run the above program and try to curl `localhost:8000/metrics` and you should see the following metrics being served.


```sh
$ curl localhost:8000/metrics

# HELP python_info Python platform information
# TYPE python_info gauge
python_info{implementation="CPython",major="3",minor="7",patchlevel="6",version="3.7.6"} 1.0
# HELP python_gc_collected_objects Objects collected during gc
# TYPE python_gc_collected_objects histogram
# HELP python_gc_uncollectable_objects Uncollectable object found during GC
# TYPE python_gc_uncollectable_objects histogram
# HELP python_gc_duration_seconds Time spent in garbage collection
# TYPE python_gc_duration_seconds histogram
# HELP count_total Some description
# TYPE count_total counter
count_total 0.7
# TYPE count_created gauge
count_created 1583416778.013898
```

---------------
### Tip

To check if above metrics are in valid format, you can use `promtool` to validate

```yaml
$ curl localhost:8000/metrics | promtool check metrics
```
-----------------------

### Storage

Prometheus includes a local on-disk time series database, but also optionally integrates with remote storage systems.

Here, let's focus on local storage and we need to provide some dir path to Prometheus to write metrics to and for that to write, we need to set environment variable `prometheus_multiproc_dir` (default) to some temporary dir like `/tmp`.


Some examples of remote storage include [Prometheus Pushgateway](https://github.com/prometheus/pushgateway). Used to allow ephemeral and batch jobs to expose their metrics to Prometheus. Since these kinds of jobs may not exist long enough to be scraped, they can instead push their metrics to a Pushgateway. The Pushgateway then exposes these metrics to Prometheus.

-------------------

### Prometheus scrape

Now, that these metrics are being served, we somehow need to tell Prometheus server to scrape these metrics from these pods.

For the above purpose, we need to have per pod Prometheus annotations which allow a fine control of the scraping process. These annotations need to be part of pod metadata. 

Annotations required:

- `prometheus.io/scrape` Default config will scrape all pods (true/false)
- `prometheus.io/path` If the metrics path is not /metrics, define it with this annotation.
- `prometheus.io/port`

-------
### Example

In our case, K8s config will look like below

```yaml
...
spec:
  template:
    metadata:
      annotations:
        prometheus.io/scrape: 'true'
        prometheus.io/port: '8000'
  containers:
    env:
    - name: prometheus_multiproc_dir
      value: /tmp
...
```
-----

### Conclusion

Above should export all the metrics on pods to prometheus server. You can check on Prometheus targets and verify if these are being exported.

Now, this data can be consumed to show visualizations like on Grafana. :tada:


If still stuck while setting up, Hire me on [Codementor.io](http://codementor.io/sukeesh) :wink:

## Getting Prometheus
Download the latest binary release of Prometheus for your platform from:

https://github.com/prometheus/prometheus/releases

Extract the contents into a new directory and change to that directory.

```
wget https://github.com/prometheus/prometheus/releases/download/v2.16.0/prometheus-2.16.0.linux-amd64.tar.gz
tar zxf prometheus-2.16.0.linux-amd64.tar.gz
cd prometheus-2.16.0.linux-amd64/
```

## Configuring Prometheus to monitor itself

Take a look at the included example `prometheus.yml` configuration file. It
configures global options, as well as a single job to scrape metrics from: the
Prometheus server itself.

Prometheus collects metrics from monitored targets by scraping metrics HTTP
endpoints on these targets. Since Prometheus also exposes data in the same
manner about itself, it may also be used to scrape and monitor its own health.
While a Prometheus server which collects only data about itself is not very
useful in practice, it is a good starting example.

## Starting Prometheus
Start Prometheus. By default, Prometheus reads its config from a file
called `prometheus.yml` in the current working directory, and it
stores its database in a sub-directory called `data`, again relative
to the current working directory. Both behaviors can be changed using
the flags `-config.file` or `-storage.tsdb.path`, respectively.

```
./prometheus
```

Prometheus should start up and it should show the targets it scrapes at
[http://localhost:9090/targets](http://localhost:9090/targets). You
will find [http://localhost:9090/metrics](http://localhost:9090/metrics) in the
list of scraped targets. Give Prometheus a couple of seconds to start
collecting data about itself from its own HTTP metrics endpoint.

You can also verify that Prometheus is serving metrics about itself by
navigating to its metrics exposure endpoint:
[http://localhost:9090/metrics](http://localhost:9090/metrics).

## Using the expression browser
The query interface at
[http://localhost:9090/](http://localhost:9090/) allows you to
explore metric data collected by the Prometheus server. At the moment, the
server is only scraping itself. The collected metrics are already quite
interesting, though.  The *Console* tab shows the most recent value of metrics,
while the *Graph* tab plots values over time. The latter can be quite expensive
(for both the server and the browser). It is in general a good idea to try
potentially expensive expressions in the *Console* tab first. Take a bit of
time to play with the expression browser. Suggestions:

* Evaluate `prometheus_tsdb_head_samples_appended_total`, which shows you
  the total number of ingested samples over the lifetime of the server. In the
  *Graph* tab, it will show as steadily increasing.
* The expression `prometheus_tsdb_head_samples_appended_total[1m]`
  evaluates to all sample values of the metric in the last minute. It cannot be
  plotted as a graph, but in the *Console* tab, you see a list of the values with
  (Unix) timestamp.
* `rate(prometheus_tsdb_head_samples_appended_total[1m])` calculates the
  rate (increase per second) over the 1m timeframe. In other words, it tells you
  how many samples per second your server is ingesting. This expression can be
  plotted nicely, and it will become more interesting as you add more targets.

## Start the node exporter
The node exporter is a server that exposes system statistics about the machine
it is running on as Prometheus metrics.

Download the latest node exporter binary release for your platform from:

https://github.com/prometheus/node_exporter/releases

Beware that the majority of the node exporter's functionality is
Linux-specific, so its exposed metrics will be significantly reduced when
running it on other platforms.

Linux example:

```
wget https://github.com/prometheus/node_exporter/releases/download/v0.18.1/node_exporter-0.18.1.linux-amd64.tar.gz
tar zxf node_exporter-0.18.1.linux-amd64.tar.gz
```

Start the node exporter:

```
cd node_exporter-0.18.1.linux-amd64/
./node_exporter
```

## Configure Prometheus to monitor the node exporter

Add the following job configuration to the `scrape_configs:` section
in `prometheus.yml` to monitor both your own and the demo node
exporter:

```
  - job_name: 'node'
    static_configs:
    - targets: ['localhost:9100', 'demo.robustperception.io:9100']
```

Send your Prometheus server a `SIGHUP` to initiate a reload of the configuration:

```
killall -HUP prometheus
```

Then check the *Status* page of your Prometheus server to make sure the node
exporter is scraped correctly. Shortly after, a whole lot of interesting
metrics will show up in the expression browser, each of them starting with
`node_`. (Reload the page to see them in the autocompletion.) As an example,
have a look at `node_cpu_seconds_total`.

The node exporter has a whole lot of modules to export machine
metrics. Have a look at the
[README.md](https://github.com/prometheus/node_exporter) to get an
idea. While Prometheus is particularly good at collecting service
metrics, correlating those with system metrics from individual
machines can be immensely helpful.  (Perhaps that one task that showed
high latency yesterday was scheduled on a node with a lot of competing
disk operations?)

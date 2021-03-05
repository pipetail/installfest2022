## Define alert for HTTP server

Add the following rules file into `prometheus.yml`:

```
rule_files:
   - "alerts.yml"
```

Now create `alerts.yml` in the same directory as `prometheus.yml` with this context:

```
groups:
- name: http
  rules:
  - alert: ApacheDown
    expr: apache_up == 0
    for: 30s
    labels:
      severity: page
    annotations:
      summary: "Apache httpd on {{ $labels.instance }} down"
```

Check the alerting rules:

```
./promtool check rules alerts.yml
```

Send your Prometheus server a `SIGHUP` to initiate a reload of the configuration:

```
killall -HUP prometheus
```

Now you'll the the alert in "Alerts" section of prometheus web interface.

Try stopping the Apache HTTP server and alert should be fired.

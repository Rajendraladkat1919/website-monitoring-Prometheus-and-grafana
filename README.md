# website-monitoring-Prometheus-and-grafana


# Introduction:
This repo contains docker compose setup using grafana, prometheus and blackbox which helps in monitoring the various website provided inside `prometheus.yml` config file.
All the modules are added inside the `blackbox.yaml` configuration file.
The blackbox exporter allows blackbox probing of endpoints over HTTP, HTTPS, DNS, TCP, ICMP and gRPC.

# Running this software on local mac machine.

Note. we need docker compose setup in your local machine while using this utility.

1. clone this repo on your local machine and change directory to `website-monitoring-Prometheus-and-grafana`
   ```
    git clone https://github.com/Rajendraladkat1919/website-monitoring-Prometheus-and-grafana.git


    cd website-monitoring-Prometheus-and-grafana
    ```

# configuration

2. Then make the neccessary changes inside `blackbox.yaml` and `prometheus.yaml` before running the docker compose up command.

# Prometheus Configuration
The blackbox exporter needs to be passed the target as a parameter, this can be done with relabelling.
'''
Example config:

scrape_configs:
  - job_name: 'blackbox'
    metrics_path: /probe
    params:
      module: [http_2xx]  # Look for a HTTP 200 response.
    static_configs:
      - targets:
        - http://prometheus.io    # Target to probe with http.
        - https://prometheus.io   # Target to probe with https.
        - http://example.com:8080 # Target to probe with http on port 8080.
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 127.0.0.1:9115  # The blackbox exporter's real hostname:port.
        
'''
HTTP probes can accept an additional hostname parameter that will set Host header and TLS SNI. This can be especially useful with dns_sd_config:
```
scrape_configs:
  - job_name: blackbox_all
    metrics_path: /probe
    params:
      module: [ http_2xx ]  # Look for a HTTP 200 response.
    dns_sd_configs:
      - names:
          - example.com
          - prometheus.io
        type: A
        port: 443
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
        replacement: https://$1/  # Make probe URL be like https://1.2.3.4:443/
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 127.0.0.1:9115  # The blackbox exporter's real hostname:port.
      - source_labels: [__meta_dns_name]
        target_label: __param_hostname  # Make domain name become 'Host' header for probe requests
      - source_labels: [__meta_dns_name]
        target_label: vhost  # and store it in 'vhost' label
```
# Permissions

The ICMP probe requires elevated privileges to function:

Windows: Administrator privileges are required.
Linux: either a user with a group within net.ipv4.ping_group_range, the CAP_NET_RAW capability or the root user is required.
Your distribution may configure net.ipv4.ping_group_range by default in /etc/sysctl.conf or similar. If not you can set net.ipv4.ping_group_range = 0 2147483647 to allow any user the ability to use ping.
Alternatively the capability can be set by executing setcap cap_net_raw+ep blackbox_exporter
BSD: root user is required.
OS X: No additional privileges are needed.

3. Once both the configuration file is ready with configuration changes do the docker-compose up -d

 ```
    ➜  website-monitoring-Prometheus-and-grafana git:(monitoring) ✗ docker-compose up -d
    Creating network "website-monitoring-prometheus-and-grafana_default" with the default driver
    Creating prometheus ... done
    Creating blackbox   ... done
    Creating grafana    ... done
```
The above containers will be running in your local machine.

# Prometheus
Let’s navigate to Prometheus address http://localhost:9090/targets we should see all the black-box in the running status.

# Grafana
 Let’s navigate to Prometheus address http://localhost:3000 we should see the Grafana login screen as follows


The Login credentials of Grafana are as follows.
Username : admin
Password : admin

4. # Adding datasource inside the grafana.


    1. On the left side, we can see the Setting icon, click on the icon, and then click on data sources.
    Then select Prometheus.
    2. Add the localhost address or Prometheus server address with the listening port. Select HTTP Method method to GET then save and test.

    3. Import Prometheus
        Now click on the + icon and select the import option.
        Import Dashboard with id 7587.



# Remove the setup
```
    website-monitoring-Prometheus-and-grafana git:(monitoring) ✗ docker-compose down
Stopping grafana    ... done
Stopping blackbox   ... done
Stopping prometheus ... done
Removing grafana    ... done
Removing blackbox   ... done
Removing prometheus ... done
```

# store grafana and prometheus data

 We are using bind mount where we are using local hostpath to store the grafana and promethus data on host machine.
 when setup is up and running it will create two folder grafana and prometheus in same git repo.

# in order to configure data source in grafana check grafana documentation link.
    https://grafana.com/tutorials/
# STRAX_Grafana

This README assumes that you are using Prysm to run a STRAX node and a validator client on a __linux__ Ubuntu or Debian VPS or local server.  
1. Configured GETH an execution node using an execution-layer client.
2. Configure a Beacon node using Prysm, a consensus-layer client.
3. Configure a Validator client and stake STRAX using Prysm.
   ** Guide here: [StratisEVM](https://github.com/stratisproject/StratisEVM).

Install the following on the same system/server:
1. Install Prometheus.
  Prometheus must first be installed to fetch the data from the beacon node and validator for Grafana to display.
    1. [Official guide and installation for Prometheues here:](https://prometheus.io/docs/prometheus/latest/getting_started/).
    2. Extract the Prometheus archive and enter it's new directory.
       ```shell
       tar xvfz prometheus-*.tar.gz
       cd prometheus-*
       ```
    3. Locate and open/edit the YML file:
       ```shell
       sudo nano /etc/prometheus/prometheus.yml
       ```
    4. YML config:
       ```shell
       global:
         scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
         evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
       # scrape_timeout is set to the global default (10s).
       
       # Scrape configuration containing endpoints to scrape:
       scrape_configs:
         - job_name: 'validator'
           static_configs:
             - targets: ['localhost:8081']
         - job_name: 'beacon node'
           static_configs:
             - targets: ['localhost:8080']
         - job_name: 'node_exporter'
           static_configs:
             - targets: ['localhost:9100']
         - job_name: 'slasher'
           static_configs:
             - targets: ['localhost:8082']
        ```
       ** Note: Running a 'slasher' job isn't mandatory for staking, only people that are running a slasher can find the metrics at the port 8082. For those that don't run a slasher, all instructions that follow remain correct.


3. Install Grafana.
4. Install Node Exporter.

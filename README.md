# STRAX_Grafana

This README assumes that you are using Prysm to run a STRAX node and a validator client on a __linux__ Ubuntu or Debian VPS or local server.  
1. Configured GETH an execution node using an execution-layer client.
2. Configure a Beacon node using Prysm, a consensus-layer client.
3. Configure a Validator client and stake STRAX using Prysm.
   ** Guide here: [StratisEVM](https://github.com/stratisproject/StratisEVM).

Install the following on the same system/server:
1. Install Prometheus.
  Prometheus must first be installed to fetch the data from the beacon node and validator for Grafana to display.
    1. [Official guid and installation for Prometheues here:](https://prometheus.io/docs/prometheus/latest/getting_started/).
    2. Extract the Prometheus archive and enter it's new directory.
       ```shell
       tar xvfz prometheus-*.tar.gz
       cd prometheus-*
       ```
    3. Locate and open/edit the YML file:
       ```shell
       sudo nano /etc/prometheus/prometheus.yml
       ```
       




   Note: Running a slasher isn't mandatory for staking, only people that are running a slasher can find the metrics at the port 8082. For those that don't run a slasher, all instructions that follow remain correct.


3. Install Grafana.
4. Install Node Exporter.

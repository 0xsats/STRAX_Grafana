# STRAX_Grafana

This README assumes that: 
1. You are using Prysm to run a STRAX node and a validator client on a __linux__ Ubuntu or Debian VPS or local server.
   1. Configured GETH an execution node using an execution-layer client.
   2. Configure a Beacon node using Prysm, a consensus-layer client.
   3. Configure a Validator client and stake STRAX using Prysm.

   ** Node configuration guide here: [StratisEVM](https://github.com/stratisproject/StratisEVM).
2. Installed GNU Screen (Details below).
3. Configured your firewall (Details below).

### Screen
Screen or GNU Screen is a terminal multiplexer. It allows you to run multiple terminal processes and sessions in the background without the need to keep the terminal windows open.
1. Install:
`sudo apt install screen` 

2. Start a Screen named session: 
   `screen -S session_name`

   Example: `screen -S Geth`
   
   Then run the client/prossess: `./geth --...`
   
   Then take session to background (detach) with `CTRL`+`A`+`D `.

   Use `screen -ls` for list of sessions.
   
   Output:
   ```shell
   There are screens on:
          1234.Geth      	(Date/Time)	(Detached)
          1235.Beacon	    (Date/Time)	(Detached)
          1236.Validator	(Date/Time)	(Detached)
   2 Sockets in /run/screens/S-linuxize.
   ```
  3. To bring back session terminal window:
     Use `screen -r session_id`. session_id=1234 for Geth client terminal obtained from `screen -ls`.

     Details and Guides for using Screen here: [How To Setup & Use Linux Screen](https://linuxize.com/post/how-to-use-linux-screen/).
   
### Uncomplicated Firewall (UFW)
Uncomplicated Firewall (UFW) is an easy to use front-end to iptables for firewall management utilities. 
[How To Setup a Firewall with UFW on an Ubuntu and Debian Cloud Server](https://www.digitalocean.com/community/tutorials/how-to-setup-a-firewall-with-ufw-on-an-ubuntu-and-debian-cloud-server).   




   
Install the following on the same system/server:
### Prometheus
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

### Grafana
3. Install Grafana.

### Node Exporter
4. Install Node Exporter.

# STRAX_Grafana

This README assumes that: 
1. You are using Prysm to run a STRAX node and a validator client on a __linux__ Ubuntu or Debian VPS or local server.
   1. Configured GETH an execution node using an execution-layer client.
   2. Configure a Beacon node using Prysm, a consensus-layer client.
   3. Configure a Validator client and stake STRAX using Prysm.
   
   ** Node configuration guide here: [StratisEVM](https://github.com/stratisproject/StratisEVM).
   
3. Installed GNU Screen (Details below) (OPTIONAL) or use **systemd** (preffered).
4. Configured your firewall (Details below).

### Screen
Screen or GNU Screen is a terminal multiplexer. It allows you to run multiple terminal processes and sessions in the background without the need to keep the terminal windows open.
1. Install:
`sudo apt install screen` 

2. Start a Screen named session: 
   `screen -S <session_name>`

   Example: `screen -S Geth`
   
   Then run the client/processes: `./geth --...`
   
   Then take session to background (Detach) with: `CTRL`+`A`+`D `.

   Use `screen -ls` for list of sessions.
   
   Output:
   ```shell
   There are screens on:
          1234.Geth      	(Date/Time)	(Detached)
          1235.Beacon	    (Date/Time)	(Detached)
          1236.Validator	(Date/Time)	(Detached)
   2 Sockets in /run/screens/S-linuxize.
   ```
  3. To bring back session terminal window (Reattach):
     Use `screen -r <session_id>`. <session_id>=1234 for Geth client terminal obtained from `screen -ls`.

     Details and Guides for using GNU Screen here: [How To Setup & Use Linux Screen](https://linuxize.com/post/how-to-use-linux-screen/).


### SYSTEMD:
Using systemd for a Validator Node
You can run your validator as a systemd process so that it will automatically restart on server reboots or crashes (and helps to avoid getting slashed!).

Before following this guide you should have already set up your validator by following the How to validate article above.

First create a new unit file called geth.service in `/etc/systemd/system/`.

`nano /etc/systemd/system/geth.service`

In this unit file you will write the commands that you want to run on server boot / restart.  

```shell
[Unit]
Description=geth

[Service]
ExecStart=PATH_TO_geth_BIN --validator --name SHOW_ON_TELEMETRY
Restart=always
RestartSec=7s

[Install]
WantedBy=multi-user.target
```  

**DANGER:**
```It is recommended to delay the restart of a node with RestartSec in the case of node crashes. It's possible that when a node crashes, consensus votes in GRANDPA aren't persisted to disk. In this case, there is potential to equivocate when immediately restarting. What can happen is the node will not recognize votes that didn't make it to disk, and will then cast conflicting votes. Delaying the restart will allow the network to progress past potentially conflicting votes, at which point other nodes will not accept them.```

**Notes:**
- Modify the [Service] Section. Find the [Service] section in the service file, and add or modify the following lines:

```
Restart=always
RestartSec=3
```

Restart=always. This Ensures that the service will always restart, no matter how it was stopped or crashed.
RestartSec=7s. This ensures that the service will wait for 7 seconds before it restarts. This can be adjusted based on your needs.

- Save and Close. Save the file and exit the text editor. If you’re using nano, press CTRL + X followed by Y (to confirm saving) and then press Enter.
- Reload systemd. For systemd to recognize the changes you made to the service file, you need to reload its configuration:

`sudo systemctl daemon-reload`  

- Test the Configuration. Stop the service and observe if it restarts automatically:

`sudo systemctl stop example.service`

- After a few seconds (based on the RestartSec value you set), the service should restart itself.
- Enable the Service. If you want your service to start on boot and utilize the auto-restart feature.
`sudo systemctl enable example.service`

To enable this to autostart on bootup run:

`systemctl enable geth.service`

Start it manually with:

`systemctl start geth.service`

You can check that it's working with:

`systemctl status geth.service`

You can tail the logs with journalctl like so:

`journalctl -f -u geth`

** That’s it! You’ve successfully set up your Linux service to restart automatically. Regularly monitoring logs and service statuses will ensure that everything runs smoothly.  

   
### Uncomplicated Firewall (UFW)
For ease of managment of your firewall settings.
1. __Always__ make sure to allow SSH for firewall. `sudo ufw allow ssh`
2.  Check status `sudo ufw status`
[How To Setup a Firewall with UFW on an Ubuntu and Debian Cloud Server](https://www.digitalocean.com/community/tutorials/how-to-setup-a-firewall-with-ufw-on-an-ubuntu-and-debian-cloud-server).     


  

### After all clients are running without issues, Install the following on the same system/server:
### Prometheus
1. Install Prometheus.
  Prometheus must first be installed to fetch the data from the beacon node and validator for Grafana to display.
    1. [Official guide and installation for Prometheues here:](https://prometheus.io/docs/prometheus/latest/getting_started/).
    2. Extract the Prometheus archive and enter it's new directory.
       ```shell
       tar -xvfz prometheus-*.tar.gz
       cd prometheus-*
       ```
    3. Locate and open/edit the YML file:
       ```shell
       sudo nano /Directory/to/prometheus.yml
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
        ````
       `CTRL O`-`Enter`save.`CTRL X` exit editor.

    5. Start a GNU Screen for Prometheues: `screen -S Prometheues`
    6. Run Prometheues from within its directory: `./prometheus`
       If errors regarding "bind: address already in use", check <PID> with `lsof -i :9090` then kill with `sudo kill -9 <pid>` then start Prometheues again.
    7. Once Prometheus is running smoothly Detach: `CTRL`+`A`+`D`
    8. Navigate to `http://194.233.70.161:9090/graph` in a browser or directly to `http://194.233.70.161:9090/targets` to check . It will present a page similar to this:
     ![alt text](https://github.com/0xsats/STRAX_Grafana/blob/main/img/prometheus.png)
   9. All green means no issues with connections, firewalls... etc.  
      
**Note on other ruuning services when port is used by other application:**
```shell
msg="Error starting web server, exiting gracefully" err="listen tcp 0.0.0.0:9090: bind: address already in use"
```
you can use `lsof -i :<port_num#>` to get the `pid` and then use `sudo kill -9 <pid>` to kill the program and free up the port so you can run service again.  


### Grafana
Grafana must now be installed to provide the graphical component of the data analytics.  
3. Install Grafana:
   1. [Download Grafana](https://grafana.com/grafana/download) and install it.
      
   2. Open http://localhost:3000 in a browser. By default, the username and the password to this panel are both ‘admin’.
    
   3. Create a data source and choose Prometheus, then enter in the URL field http://localhost:9090 (VPS/Server IP).
      
   4. Click on Save & Test.
   5. A green notification saying 'Successfully queried the Prometheus API.' should now be visible.  

   **any errors check ports/firewalls.



### Node Exporter  
Node exporter allows Prometheus to record system data, such as CPU utilization, memory use, CPU temperature, and disk usage.  
4. Download / Install [Node Exporter](https://prometheus.io/download/#node_exporter).  
** Run node_exporter on its own GNU Screen.  

## Grafana Dashboards  
![alt text](https://github.com/0xsats/STRAX_Grafana/blob/main/img/strax_grafana.png)  


## Pre-Configured Dashboards:
To import pre-configured dashboards, copy the their JSON contents and paste them in them in Garfana:   
`__Goto__ Dashboards > New > Import > 'Import via dashboard JSON model' >= Paste JSON`.  

![alt text](https://github.com/0xsats/STRAX_Grafana/blob/main/img/importgrafanadashboard.png)  


**Note these 3rd party dashboards have many missing parts and should be used as examples only**


1. Node Exporter Full Dashboard.
   
   [JSON File](https://raw.githubusercontent.com/rfmoz/grafana-dashboards/master/prometheus/node-exporter-full.json)
   
   ![alt text](https://github.com/0xsats/STRAX_Grafana/blob/main/img/Node%20Exporter%20Full.png)
   
   
2. Teku Dashboard (closest yet) [github](https://github.com/Consensys/teku/):
   
   [JSON file](https://raw.githubusercontent.com/Consensys/teku/master/dashboard/teku-dashboard-grafana.json)
   
   ![alt text](https://github.com/0xsats/STRAX_Grafana/blob/main/img/Tekudashboard.png)
   
   
3. https://grafana.com/grafana/dashboards/?search=validator

4. Grafana Dashboard for Stratis from ETH validators:
   ![alt text](https://github.com/0xsats/STRAX_Grafana/blob/main/img/strax_grafana.png)
   [JSON File](https://github.com/0xsats/STRAX_Grafana/blob/main/strax_dashboard.json)




## Still working on Dashboard..

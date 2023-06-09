# Node Monitoring

Here is a quick guide on how to monitor your node using Grafana and Prometheus.

## Official documentation

Official documentation is located here: https://grafana.com/docs/grafana-cloud/quickstart/noagent_linuxnode/  

> **Warning**
> This guide is made by myself, if you encounter any error, please refer to the official documentation

## Grafana

First things first, you need to create your Grafana account and get your API key:  
https://grafana.com/docs/grafana-cloud/quickstart/  


Get the latest node_exporter package  
```
wget https://github.com/prometheus/node_exporter/releases/download/v1.5.0/node_exporter-1.5.0.linux-amd64.tar.gz
```

Extract
```
tar xvfz node_exporter-1.5.0.linux-amd64.tar.gz
```

Move into folder
```
cd node_exporter-1.5.0.linux-amd64/
```

Add executable permissions
```
chmod +x node_exporter
```

Move it in **/usr/local/bin**, rename it **grafana-agent**, then start it
```
mv node_exporter /usr/local/bin/grafana-agent
grafana-agent
```

In an another terminal, try this command to check if everything if working properly
```
curl http://localhost:9100/metrics
```

If you have results, you can close this terminal, and Ctrl-C the other one to stop it. Then we will create a service (in case there is an error, everything will restart correctly)  

Create an user for Grafana
```
sudo useradd --no-create-home --shell /bin/false grafana-agent
```

Create service
```
sudo tee <<EOF >/dev/null /etc/systemd/system/grafana-agent.service
[Unit]
Description=Grafana Agent

[Service]
User=grafana-agent
ExecStart=/usr/local/bin/grafana-agent
Restart=always
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

To enable the service to run automatically on every reboot, and start it:
```
sudo systemctl enable grafana-agent.service
sudo systemctl start grafana-agent.service
```

Now, try this command again to check that everything is working properly
```
curl http://localhost:9100/metrics
```

## Prometheus

Download Prometheus
```
wget https://github.com/prometheus/prometheus/releases/download/v2.43.0/prometheus-2.43.0.linux-amd64.tar.gz
```

Extract it
```
tar xvf prometheus-2.43.0.linux-amd64.tar.gz
```

Move into folder
```
cd prometheus-2.43.0.linux-amd64
```

Then, you need your Grafana Cloud username and the API key you created earlier. To confirm your username and URL, on the Grafana website, first navigate to the Cloud Portal, then from the Prometheus box, click Send Metrics.  

<img src="img/cloud_portal.png">

Create the Prometheus configuration file, **DON'T FORGET TO REPLACE VALUES**
```
sudo tee <<EOF >/dev/null $HOME/prometheus-2.43.0.linux-amd64/prometheus.yml
global:
  scrape_interval: 60s

scrape_configs:
  - job_name: node
    static_configs:
      - targets: ['localhost:9100']

remote_write:
  - url: 'Your Metrics instance remote_write endpoint'
    basic_auth:
      username: 'your grafana username'
      password: 'your Grafana API key'
EOF
```

Start Prometheus
```
./prometheus --config.file=./prometheus.yml
```

Within minutes, metrics should begin to be available in Grafana Cloud. To test this, use the Explore feature. Click Explore in the sidebar to start. This takes you to the Explore page, which looks like this.  

<img src="img/quickstarts-explorepage.png">  

Now we will create a service for Prometheus, exactly what we did with Grafana

Create service
```
sudo tee <<EOF >/dev/null /etc/systemd/system/prometheus-agent.service
[Unit]
Description=Prometheus Agent

[Service]
User=$USER
WorkingDirectory=$HOME/prometheus-2.43.0.linux-amd64/
ExecStart=$HOME/prometheus-2.43.0.linux-amd64/prometheus --config.file=./prometheus.yml
Restart=always
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

To enable the service to run automatically on every reboot, and start it
```
sudo systemctl enable prometheus-agent.service
sudo systemctl start prometheus-agent.service
```

Check if you have logs again, with systemctl, or directly with the Grafana dashboard
```
sudo journalctl -u prometheus-agent.service -f
```

And voilà :tada: ! Last thing to do is to configure a nice dashboard for metrics.  
In Grafana, click Dashboards in the left-side menu to go to the Dashboards page.

Click New and select Import in the dropdown. Enter the ID number 10180 into the box and click Load.  

<img src="img/quickstarts-importeddashboard.png">

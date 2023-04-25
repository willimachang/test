# Install the SNMP Exporter
Description
Now to manually install the SNMP Exporter.

I am going to manually install the latest version of the SNMP Exporter.

To see the latest versions, you can visit,

https://github.com/prometheus/snmp_exporter/releases

Select the file appropriate for your operating system.

I will download snmp_exporter-0.19.0.linux-amd64.tar.gz


wget https://github.com/prometheus/snmp_exporter/releases/download/v0.19.0/snmp_exporter-0.19.0.linux-amd64.tar.gz
Untar it,


tar xzf snmp_exporter-0.19.0.linux-amd64.tar.gz
CD into the new folder that was created and copy the files to the /usr/local/bin/ folder


cd snmp_exporter-0.19.0.linux-amd64
ls -lh
cp ./snmp_exporter /usr/local/bin/snmp_exporter
cp ./snmp.yml /usr/local/bin/snmp.yml
cd /usr/local/bin/
See the snmp_exporter help


./snmp_exporter -h
Create a new user (if it doesn't already exist)


sudo useradd --system prometheus
Create a new file called snmp-exporter.service


sudo nano /etc/systemd/system/snmp-exporter.service
Add the script and save


[Unit]
Description=Prometheus SNMP Exporter Service
After=network.target

[Service]
Type=simple
User=prometheus
ExecStart=/usr/local/bin/snmp_exporter --config.file="/usr/local/bin/snmp.yml"

[Install]
WantedBy=multi-user.target
Now start and check the service is running.


systemctl daemon-reload
sudo service snmp-exporter start
sudo service snmp-exporter status
Visit in the browser http://[ip or domain]:9116

Now to add the configuration to the prometheus.yml


sudo nano /etc/prometheus/prometheus.yml
Add the following to the end of the existing YML.


  - job_name: snmp
    metrics_path: /snmp
    params:
      module: [if_mib]
    static_configs:
      - targets:
        - 127.0.0.1
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 127.0.0.1:9116 # URL as shown on the UI
Save and check changes to the config are syntactically correct


promtool check config /etc/prometheus/prometheus.yml
And if OK, then restart the Prometheus service.


sudo service prometheus restart
sudo service prometheus status

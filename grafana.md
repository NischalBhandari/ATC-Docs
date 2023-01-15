### Installing Grafana

For details please see grafana docs

```
Sudo yum-config-manager –add-repo https://packages.grafana.com/enterprise/rpm

```


```

[grafana]
name=grafana
baseurl=https://packages.grafana.com/enterprise/rpm
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://packages.grafana.com/gpg.key
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt



Sudo yum install grafana-enterprise


sudo systemctl daemon-reload
sudo systemctl start grafana-server
sudo systemctl status grafana-server

```

In order for Traffic Portal users to see Grafana graphs, Grafana will need to allow anonymous access. Information on how to configure anonymous access can be found on the configuration page of the Grafana Website.

Traffic Portal uses custom dashboards to display information about individual Delivery Services or Cache Groups. In order for the custom graphs to display correctly, the Javascript files in traffic_stats/grafana/ need to be in the /usr/share/grafana/public/dashboards/ directory on the Grafana server. If your Grafana server is the same as your Traffic Stats server the RPM install process will take care of putting the files in place. If your Grafana server is different from your Traffic Stats server, you will need to manually copy the files to the correct directory

### Configuring Traffic Portal for Traffic Stats
The InfluxDB servers need to be added to Traffic Portal with a Profile that has the Type InfluxDB. Make sure to use port 8086 in the configuration.

The traffic stats server should be added to Traffic Ops with a Profile that has the Type TRAFFIC_STATS.

Parameters for which stats will be collected are added with the release, but any changes can be made via Parameters that are assigned to the Traffic Stats Profile.

### To change the parameters that are monitored  see in Traffic portal parameters 
 traffic_stats.config


 More details

 https://traffic-control-cdn.readthedocs.io/en/latest/admin/traffic_stats.html




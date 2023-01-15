### Install Traffic stats

```
Sudo yum install traffic_stats….rpm

```


### Add Influxdb to Repository

```
cat <<EOF | tee /etc/yum.repos.d/influxdb.repo
[influxdb]
name = InfluxDB Repository - RHEL \$releasever
baseurl = https://repos.influxdata.com/centos/\$releasever/\$basearch/stable
enabled = 1
gpgcheck = 1
gpgkey = https://repos.influxdata.com/influxdb.key
EOF
```

### Install influxdb

```
yum install influxdb

```

For influx cli become a root user  
Then type influx

```
sudo su

influx

```


Here you can create a user 
```
CREATE USER username WITH PASSWORD 'password' WITH ALL PRIVILEGES
```


Then go into /opt/traffic_stats/nifluxdb_tools/

```
Sudo ./create_ts_databases -user username -password password
```

Also configuration files can be found in /etc/influxdb/influxdb.conf(for http)

#### Influxdb needs a different permission to start on port 80 

```
sudo setcap CAP_NET_BIND_SERVICE=+eip /path/to/binary
```

#### To start influx cli with the new port with 80 and https
```
influx -ssl -port 80 -host trafficmonitor-infra.datahub.com.np
```

Note : 
The traffic_stats.cfg file contains the default retention policy for the influxdb databases such as cache_stats deliveryservice stats etc. Be careful to align the default policy in this and also influxdb as traffic stats only writes on the default retention policy


Now to change the retention policy of the influxdb databases change the duration of the monthly retention policy from 1 month to 3 month.


Note : For seeing the space used by influxdb database

du -sh /var/lib/influxdb/data/<db name>


Now after see the grafana.md

More details https://traffic-control-cdn.readthedocs.io/en/latest/admin/traffic_stats.html




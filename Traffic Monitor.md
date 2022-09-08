# Traffic Monitor

**Traffic Monitor is an HTTP service that monitors the cache servers in a CDN for a variety of metrics.** These metrics are for use in determining the overall “health” of a given cache server and the related Delivery Services. A given CDN can operate a number of Traffic Monitors, from a number of geographically diverse locations, to prevent false positives caused by network problems at a given site. Traffic Monitors operate independently, but use the state of other Traffic Monitors in conjunction with their own state to provide a consistent view of CDN cache server health to downstream applications such as Traffic Router. Health Protocol governs the cache server and Delivery Service availability. Traffic Monitor provides a view into CDN health using several RESTful JSON endpoints, which are consumed by other Traffic Monitors and downstream components such as Traffic Router. **Traffic Monitor is also responsible for serving the overall CDN configuration to Traffic Router, which ensures that the configuration of these two critical components remain synchronized as operational and health related changes propagate through the CDN**.

### Cache Monitoring

Traffic Monitor polls all cache servers configured with a status of REPORTED or ADMIN_DOWN at an interval specified as a configuration parameter in Traffic Ops. 
If the cache server is set to **ADMIN_DOWN** it is marked as unavailable but still polled for availability and statistics. If the cache server is explicitly configured with a status of **ONLINE** or **OFFLINE**, it is not polled by Traffic Monitor and presented to Traffic Router as configured, regardless of actual availability. 

Traffic Monitor polls traffic servers (ATS) at a specified interval on a specefic url. 
For eg: edge01-infra.dathaub.com.np/_stats
Currently traffic monitor polls the traffic server for stats using the HTTP STATS plugin in ATS.

The plugins provide information about
<li>Throughput (e.g. bytes in, bytes out, etc).</li>

<li>Transactions (e.g. number of 2xx, 3xx, 4xx responses, etc).</li>

<li>Connections (e.g. from clients, to parents, origins, etc).</li>

<li>Cache performance (e.g.: hits, misses, refreshes, etc).</li>

<li>Storage performance (e.g.: writes, reads, frags, directories, etc).</li>

<li>System performance (e.g: load average, network interface throughput, etc).</li>
</ol>

Many of the application-level statistics are available at the global or aggregate level, some at the Delivery Service level. Traffic Monitor uses the system-level performance to determine the overall health of the cache server by evaluating network throughput and load against values configured in Traffic Ops. Traffic Monitor also uses throughput and transaction statistics at the Delivery Service level to determine Delivery Service health. 

 If the Delivery Service statistics exceed the configured thresholds, the Delivery Service is marked as unavailable, and Traffic Router will start sending clients to the overflow destinations for that Delivery Service, but the cache server remains available to serve other content.

## HEALTH PROTOCOL

### Optimistic Health Protocol
Redundant Traffic Monitor servers operate independently from each other but take the state of other Traffic Monitors into account when asked for health state information. 
Upon startup or configuration change in Traffic Ops, in addition to cache servers, Traffic Monitor begins polling its peer Traffic Monitors whose state is set to ONLINE. Each ONLINE Traffic Monitor polls all of its peers at a configurable interval and saves the peer’s state for later use. When polling its peers, Traffic Monitor asks for the raw health state from each respective peer, which is strictly that instance’s view of the CDN’s health. When any ONLINE Traffic Monitor is asked for CDN health by a downstream component, such as Traffic Router, the component gets the Health Protocol-influenced version of CDN health (non-raw view). 
In operation of the Health Protocol, Traffic Monitor takes all health states from all peers, including the locally known health state, and serves an optimistic outlook to the requesting client. **This means that, for example, if three of the four Traffic Monitors see a given cache server or Delivery Service as exceeding its thresholds and unavailable, it is still considered available.** Only if all Traffic Monitors agree that the given object is unavailable is that state propagated to downstream components. 


# Installing Traffic Monitor

The following are hard requirements requirements for Traffic Monitor to operate:

CentOS 7 or later

Successful install of Traffic Ops (usually on a separate machine)

Administrative access to the Traffic Ops (usually on a separate machine)

These are the recommended hardware specifications for a production deployment of Traffic Monitor:

8 CPUs

16GB of RAM

It is also recommended that you know the geographic coordinates and/or mailing address of the site where the Traffic Monitor machine lives for optimal performance

## Steps for configuring Traffic Monitor
<ol>
<li>Enter the Traffic Monitor server into Traffic Portal</li>

<li>Make sure the FQDN of the Traffic Monitor is resolvable in DNS.</li>
<li>Install Traffic Monitor, either from source or by installing a traffic_monitor-version string.rpm package generated by the instructions in Building Traffic Control with yum(8) or rpm(8)</li>
<li>Configure Traffic Monitor according to Configuring Traffic Monitor</li>
<li>Start Traffic Monitor, usually by starting its systemd(1) service</li>
<li>Verify Traffic Monitor is running by e.g. opening your preferred web browser to port 80 on the Traffic Monitor host.</li>
</ol>


## Configuring Traffic Monitor

As defined above for configuring Traffic Monitor there are two important files
traffic_ops.cfg and traffic_monitor.cfg
For details please visit 

https://traffic-control-cdn.readthedocs.io/en/latest/admin/traffic_monitor.html#configuring-traffic-monitor


## Configuring Traffic Portal For Traffic Monitor

**Create a Traffic Monitor Profile in Traffic Portal**

    1. Name : RASCAL_Traffic-Monitor
    2. CDN : NAME_OF_CDN
    3. TYPE: TM_PROFILE
    4. ROUTING DISABLED : false
    5. DESCRIPTION: Description of the profile


**Parameters for Traffic Monitor**


1. This parameter is created so that the traffic monitor knows which type of monitoring plugin is to used to monitor Apache Traffic Servers

    1. Name: health.polling.format
    2. Config File: rascal.properties
    3. Value: stats_over_http
    4. secure: false

2. This parameter is created so that the traffic monitor knows what url to expect for monitoring from Apache Traffic servers.
    1. Name: health.polling.url
    2. Config File: rascal.properties
    3. Value: http://${hostname}/_stats
    4. Secure: false

3. This parameter is created so that traffic monitor knows interval in which to poll its peers
    1. Name: peers.polling.interval
    2. Config File: rascal-config.txt
    3. Value: 10000
    4. Secure: false

4. This parameter is created so that traffic monitor knows interval in which to poll ATS's
    1. Name: health.polling.interval
    2. Config File: rascal-config.txt
    3. Value: 10000
    4. Secure: false


#### Note for adding servers in Traffic Portal ! important Points
When adding a server in traffic portal keep in mind that the terms
Hostname and Domain Name

**FQDN = Hostname + Domain Name**

Also keep the server as **REPORTED** for it to be reported for monitoring

**Also For traffic Monitor to be working properly at least one ATS should be running.**

Error : If unable to cache server this trafficmonitor (TRAFFIC-MONITOR)
*This is the issue of hostname. The hostname should be pingable  add an etc/hosts file to ping 
TRAFFIC-MONITOR (127.0.0.1)*



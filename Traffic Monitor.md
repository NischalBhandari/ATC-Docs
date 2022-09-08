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
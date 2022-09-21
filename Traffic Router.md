# Traffic Router

Traffic Router’s function is to send clients to the most optimal cache server. ‘Optimal’ in this case is based on a number of factors:

<li>Distance between the cache server and the client (not necessarily measured in physical distance, but quite often in layer 3 network hops). Less network distance between the client and cache server yields better performance and lower network load. Traffic Router helps clients connect to the best-performing cache server for their location at the lowest network cost.</li>

<li>
Availability of cache servers and the system processing/network load on the cache servers. A common issue in Internet and television distribution scenarios is having many clients attempting to retrieve the same content at the same time. Traffic Router helps clients route around overloaded or purposely disabled cache servers.
</li>

<li>
Availability of content on a particular cache server. Reusing of content through “cache hits” is the most important performance gain a CDN can offer. Traffic Router sends clients to the cache server that is most likely to already have the desired content.
</li>


Traffic routing options are often configured at the Delivery Service level.

## DNS Content Routing

For a DNS Delivery Service the client might receive a URL such as http://video.demo1.mycdn.ciab.test/. When the LDNS is resolving this video.demo1.mycdn.ciab.test hostname to an IP address, it ends at Traffic Router because it is the authoritative DNS server for mycdn.ciab.test and the domains below it, and subsequently responds with a list of IP addresses from the eligible cache servers based on the location of the LDNS. When responding, Traffic Router does not know the actual client IP address or the path that the client is going to request. The decision on what cache server IP address (or list of cache server IP addresses) to return is solely based on the location of the LDNS and the health of the cache servers. The client then connects to port 80 (HTTP) or port 443 (HTTPS) on the cache server, and sends the Host: video.demo1.mycdn.ciab.test header. The configuration of the cache server includes the “remap rule” http://video.demo1.mycdn.ciab.test http://origin.infra.ciab.test to map the routed name to an Origin hostname.


## HTTP Content Routing
For an HTTP Delivery Service the client might receive a URL such as http://video.demo1.mycdn.ciab.test/. The LDNS resolves this video.demo1.mycdn.ciab.test to an IP address, but in this case Traffic Router returns its own IP address. The client opens a connection to port 80 (HTTP) or port 443 (HTTPS) on the Traffic Router’s IP address, and sends its request.

 Example Client Request to Traffic Router
*GET / HTTP/1.1
Host: video.demo1.mycdn.ciab.test
Accept: */* *

Traffic Router uses an HTTP 302 Found response to redirect the client to the best cache server.

*HTTP/1.1 302 Found
Location: http://edge.demo1.mycdn.ciab.test/
Content-Length: 0
Date: Tue, 13 Jan 2015 20:01:41 GMT*


In this case Traffic Router has access to more information when selecting a cache server because it has a full HTTP request instead of just a hostname. Traffic Router can be configured to select a cache server based on any of the following parts of the HTTP request:

The client’s IP address.

The URL the client is requesting.

All HTTP/1.1 headers.

The client follows the redirect and performs a DNS request for the IP address for edge.demo1.mycdn.ciab.test, and normal HTTP steps follow, except the sending of the Host: header when connected to the cache is Host: edge.demo1.mycdn.ciab.test, and the configuration of the cache server includes the “remap rule” (e.g. http://edge.demo1.mycdn.ciab.test http://origin.infra.ciab.test). Traffic Router sends all requests for the same path in a Delivery Service to the same cache server in a Cache Group using consistent hashing, in this case all cache servers in a Cache Group are not carrying the same content, and there is a much larger combined cache in the Cache Group. In many cases DNS content routing is the best possible option, especially in cases where the client is receiving small objects from the CDN like images and web pages. Traffic Router is redundant and horizontally scalable by adding more instances into the DNS hierarchy using NS records.



# Installing Traffic Router

**Requirements**
1. CentOS 7 or later

2. 4 CPUs

3. 8GB of RAM

4. Successful install of Traffic Ops (usually on another machine)

5. Successful install of Traffic Monitor (usually on another machine)

6. Administrative access to Traffic Ops


Setup Process

1. If no suitable Profile exists, create a new Profile for Traffic Router via the + button on the Profiles page in Traffic Portal
2. Enter the Traffic Router server into Traffic Portal on the Servers page (or via the Traffic Ops API), assign to it a Traffic Router Profile, and ensure that its status is set to ONLINE.
3. Enter the Traffic Router server into Traffic Portal on the Servers page (or via the Traffic Ops API), assign to it a Traffic Router Profile, and ensure that its status is set to ONLINE.
4. Extract the tomcat RPM (sudo yum install tomcat)
5. Extract the Traffic Router RPM
6. Edit the traffic_monitor.properties and specify the correct online Traffic Monitor(s) for your CDN.
   ``` https://traffic-control-cdn.readthedocs.io/en/latest/admin/traffic_router.html#tr-config-files
   
   traffic_monitor.properties
        URL that should normally point to this file, e.g. traffic_monitor.properties=file:/opt/traffic_router/conf/traffic_monitor.properties

    traffic_monitor.properties.reload.period
        Period to wait (in milliseconds) between reloading this file, e.g. traffic_monitor.properties.reload.period=60000
   
    ```
7. Start Traffic Router. This is normally done by starting its systemd(1) service. systemctl start traffic_router , and test DNS lookups against that server to be sure it’s resolving properly. with e.g. dig or curl. Also, because previously taken CDN Snapshots will be cached, they need to be removed manually to actually be reloaded. This file should be located at /opt/traffic_router/db/cr-config.json. This should be done before starting or restarting Traffic Router.
    ``` 
        systemctl start traffic_router
        dig @localhost mycdn.ciab.test

        ; <<>> DiG 9.9.4-RedHat-9.9.4-72.el7 <<>> @localhost mycdn.ciab.test
        ; (2 servers found)
        ;; global options: +cmd
        ;; Got answer:
        ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 27109
        ;; flags: qr aa rd; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 0
        ;; WARNING: recursion requested but not available

        ;; QUESTION SECTION:
        ;mycdn.ciab.test.       IN  A

        ;; AUTHORITY SECTION:
        mycdn.ciab.test.    30  IN  SOA trafficrouter.infra.ciab.test. twelve_monkeys.mycdn.ciab.test. 2019010918 28800 7200 604800 30

        ;; Query time: 28 msec
        ;; SERVER: ::1#53(::1)
        ;; WHEN: Wed Jan 09 21:27:57 UTC 2019
        ;; MSG SIZE  rcvd: 104

    ```

8. Perform a CDN Snapshot.

9. Ensure that the parent domain (e.g.: cdn.local) for the CDN’s top level domain (e.g.: ciab.cdn.local) contains a delegation (Name Server records) for the new Traffic Router, and that the value specified matches the FQDN of the Traffic Router.

### Configuring Traffic Router

Changed in version 1.5: Many of the configuration files under /opt/traffic_router/conf are now only needed to override the default configuration values for Traffic Router. Most of the given default values will work well for any CDN. Critical values that must be changed are hostnames and credentials for communicating with other Traffic Control components such as Traffic Ops and Traffic Monitor. Pre-existing installations that store configuration files under /opt/traffic_router/conf will still be used and honored for Traffic Router 1.5 onward.

For the most part, the configuration files and Parameters used by Traffic Router are used to bring it online and start communicating with various Traffic Control components. Once Traffic Router is successfully communicating with Traffic Control, configuration should mostly be performed in Traffic Portal, and will be distributed throughout Traffic Control via CDN Snapshot process.

** For more information on configuring please visit **
https://traffic-control-cdn.readthedocs.io/en/latest/admin/traffic_router.html


# Notes


The parameters must be linked to a profile
Which name must start with ccr- (for legacy reasons) and choose type of TR_PROFILE

**You must also install lighthttpd to install  GeoLite2-City.mmdb.gz and czmap.json**


After the traffic monitor is up and running it will provide traffic router with the cr-config.json

Then you must assign a delivery service in the traffic portal for its dns zone file to be available in the traffic router var/auto-zones

Then we will be able to dig @localhost mycdn.ciab.test

For the CDN to have changes CDN snapshot must be performed.

Ensure that parent domain (cdn.local) for the CDN’s top level domain ciab.cdn.local contains a delegation to the new Traffic ROuter and that the value specified matches the FQDN of the Traffic Router

A profile for Traffic Router must be setup 
**Here very important things are Parameters in traffic portal**

1. geolocation.polling.interval  CRConfig.json  20000  false 
2. coverazezone.polling.interval  CRConfig.json   20000  false 
3. certificates.polling.interval  CRConfig.json   5000  false 
4. geolocation.polling.url   CRConfig.json   http://trafficrouter.infra.nischal.rest:8080/GeoLite2-City.mmdb.gz false 
5. coveragezone.polling.url  CRConfig.json  http://trafficrouter.infra.nischa.rest:8080/czmap.json false 
6. tld.soa.expire   CRConfig.json   604800  false 
7. tld.ttls.A    CRConfig.json   3600 false 
8. domain_name    CRConfig.json   mycdn.nischal.rest   false 
9. tld.soa.refresh   CRConfig.json  3600 false 
10. tld.ttls.NS   CRConfig.json   3600 false 
11. api.port      server.xml   3333 false 
12. secure.api.port   server.xml  3443 false 
13. api.cach-control.max-age  CRConfig.json  10  false 

** The parameters  above must be linked to a profile
Which name must start with ccr- (for legacy reasons) and choose type of TR_PROFILE **

    
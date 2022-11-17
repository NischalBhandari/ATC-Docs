Many websites are hosted in cpanel with name based virtual hosting.For these kind of websites it is essential that the Host header be preserved as without this the hosting (origin) server might not be able to serve the request from the traffic server.

This can be managed via a configuration in the records.config file 
i.e. proxy.config.url_remap.pristine_host_hdr

But what can we do for in a per remap rule condition i.e. one remap rule(or delivery service)
requires the pristine host heade while others do not need it and should specefically be turned off in certain conditions. This is where tslua comes  to our rescue

You need to install TSLUA as described in the documentation. Then we can create a file in the Apache Traffic Server for eg. elecsol.lua, datahub.lua etc. 
In this lua file we need to add a these lines 
```
function do_remap()
    ts.http.config_int_set(TS_LUA_CONFIG_URL_REMAP_PRISTINE_HOST_HDR, 1)
    return 0
end


```
Note: The above program can be used to do a lot of things . Please read about it here 
https://docs.trafficserver.apache.org/en/latest/admin-guide/plugins/lua.en.html#ts-http-config-int-set

This file can now be saved as elecsol.lua or datahub.lua or any other file name you like or that describes the delivery service

We need to add this in the delivery service configuration page with the following line 

```
@plugin=/opt/trafficserver/libexec/trafficserver/tslua.so  @pparam=/var/test/elecsol.lua
```

Here we are using the tslua.so named plugin to implement lua programming and using the file created earlier in the ATS to configure a per remap rule basis remapping configuration.


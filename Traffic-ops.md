# Traffic OPS
Traffic Ops is the tool for administration (configuration and monitoring) of all components in a Traffic Control CDN. Traffic Portal uses the Traffic Ops API to manage servers, Cache Groups, Delivery Services, etc. In many cases, a configuration change requires propagation to several, or even all, cache servers and only explicitly after or before the same change propagates to Traffic Router. Traffic Ops takes care of this required consistency between the different components and their configuration.

Traffic Ops uses a PostgreSQL database to store the configuration information, and a combination of the Mojolicious framework and Go to provide the Traffic Ops API. Not all configuration data is in this database however; for sensitive data like private SSL keys or token-based authentication shared secrets, **Traffic Vault** is used as a separate, key/value store, allowing administrators to harden the Traffic Vault server better from a security perspective (i.e only allow Traffic Ops to access it, verifying authenticity with a certificate). The Traffic Ops server, by design, needs to be accessible from all the other servers in the Traffic Control CDN.

**Traffic Ops generates all the application-specific configuration files for the cache servers and other servers**. The cache servers and other servers check in with Traffic Ops at a regular interval to see if updated configuration files require application. On cache servers this is done by the ORT script.


# Installing Traffic OPS

Requirements for installing Traffic OPS

The user must have the following for a successful minimal install:

1. CentOS 7 or later

2. Two machines - physical or virtual -, each with at least two (v)CPUs, 4GB of RAM, and 20 GB of disk space

3. Access to CentOS Base and EPEL yum(8) repositories

4. Access to The Comprehensive Perl Archive Network (CPAN)


## Guide For installing Traffic Ops

1. Install PostgreSQL Database. For a production install it is best to install PostgreSQL on its own server/virtual machine.
   ``` yum update -y
   yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
   yum install -y postgresql13-server
   su - postgres -c '/usr/pgsql-13/bin/initdb -A md5 -W' #-W forces the user to provide a superuser (postgres) password ```
2. Edit /var/lib/pgsql/13/data/pg_hba.conf to allow the Traffic Ops instance to access the PostgreSQL server. For example, if the IP address of the machine to be used as the Traffic Ops host is 192.0.2.1 add the line host  all   all     192.0.2.1/32 md5 to the appropriate section of this file.
3. Edit the /var/lib/pgsql/13/data/postgresql.conf file to add the appropriate listen_addresses or listen_addresses = '*', set timezone = 'UTC', and start the database
    ``` systemctl enable postgresql-13
    systemctl start postgresql-13
    systemctl status postgresql-13 ```
4. Build a traffic_ops-version string.rpm file using the instructions under the Building Traffic Control page - or download a pre-built release from the Apache Continuous Integration server.
5. Install a PostgreSQL client on the Traffic Ops host
  ``` yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm ```
6. Install the Traffic Ops RPM. The Traffic Ops RPM file should have been built in an earlier step.
    ``` yum install -y ./dist/traffic_ops-3.0.0-xxxx.yyyyyyy.el7.x86_64.rpm ```
7. Login to the Database from the Traffic Ops machine. At this point you should be able to login from the Traffic Ops (hostname to in the example) host to the PostgreSQL (hostname pg in the example) host
   ```  psql -h pg -U postgres ```
8. Create the user and database. By default, Traffic Ops will expect to connect as the traffic_ops user to the traffic_ops database.
   ```to-# psql -U postgres -h pg -c "CREATE USER traffic_ops WITH ENCRYPTED PASSWORD 'tcr0cks';"
        Password for user postgres:
        CREATE ROLE
        to-# createdb traffic_ops --owner traffic_ops -U postgres -h pg
        Password:
        to-#
    ```
9.  Now, run the following command as the root user (or with sudo(8)): /opt/traffic_ops/install/bin/postinstall. Some additional files will be installed, and then it will proceed with the next phase of the install, where it will ask you about the local environment for your CDN. Please make sure you remember all your answers and verify that the database answers match the information previously used to create the database.
 ```to-# /opt/traffic_ops/install/bin/postinstall
...

===========/opt/traffic_ops/app/conf/production/database.conf===========
Database type [Pg]:
Database type: Pg
Database name [traffic_ops]:
Database name: traffic_ops
Database server hostname IP or FQDN [localhost]: pg
Database server hostname IP or FQDN: pg
Database port number [5432]:
Database port number: 5432
Traffic Ops database user [traffic_ops]:
Traffic Ops database user: traffic_ops
Password for Traffic Ops database user:
Re-Enter Password for Traffic Ops database user:
Writing json to /opt/traffic_ops/app/conf/production/database.conf
Database configuration has been saved
===========/opt/traffic_ops/app/db/dbconf.yml===========
Database server root (admin) user [postgres]:
Database server root (admin) user: postgres
Password for database server admin:
Re-Enter Password for database server admin:
===========/opt/traffic_ops/app/conf/cdn.conf===========
Generate a new secret? [yes]:
Generate a new secret?: yes
Number of secrets to keep? [10]:
Number of secrets to keep?: 10
Not setting up ldap
===========/opt/traffic_ops/install/data/json/users.json===========
Administration username for Traffic Ops [admin]:
Administration username for Traffic Ops: admin
Password for the admin user:
Re-Enter Password for the admin user:
Writing json to /opt/traffic_ops/install/data/json/users.json
===========/opt/traffic_ops/install/data/json/openssl_configuration.json===========
Do you want to generate a certificate? [yes]:
Country Name (2 letter code): US
State or Province Name (full name): CO
Locality Name (eg, city): Denver
Organization Name (eg, company): Super CDN, Inc
Organizational Unit Name (eg, section):
Common Name (eg, your name or your server's hostname):
RSA Passphrase:
Re-Enter RSA Passphrase:
===========/opt/traffic_ops/install/data/json/profiles.json===========
Traffic Ops url [https://localhost]:
Traffic Ops url: https://localhost
Human-readable CDN Name.  (No whitespace, please) [kabletown_cdn]: blue_cdn
Human-readable CDN Name.  (No whitespace, please): blue_cdn
DNS sub-domain for which your CDN is authoritative [cdn1.kabletown.net]: blue-cdn.supercdn.net
DNS sub-domain for which your CDN is authoritative: blue-cdn.supercdn.net
Writing json to /opt/traffic_ops/install/data/json/profiles.json

... much SQL output skipped

Starting Traffic Ops
Restarting traffic_ops (via systemctl):                    [  OK  ]
Waiting for Traffic Ops to restart
Success! Postinstall complete.

```



# Traffic Server

Apache Traffic Server™ provides a high-performance and scalable software solution for both forward and reverse proxying of HTTP/HTTPS traffic, and may be configured to run in either or both modes simultaneously. This Getting Started guide explains the basic steps an administrator new to Traffic Server will need to perform to get the software up and running in a minimal configuration as quickly as possible.


**Origin Server**
The server which generates the content you wish to proxy (and optionally cache) with Traffic Server. In a forward proxy configuration, the origin server may be any remote server to which a proxied client attempts to connect. In a reverse proxy configuration, the origin servers are typically a known set of servers for which you are using Traffic Server as a performance-enhancing caching layer.

**Reverse Proxy**
A reverse proxy appears to outside users as if it were the origin server, though it does not generate the content itself. Instead, it intercepts the requests and, based on the configured rules and contents of its cache, will either serve a cached copy of the requested content itself, or forward the request to the origin server, possibly caching the content returned for use with future requests.

**Forward Proxy**(Not important in this context)
A forward proxy brokers access to external resources, intercepting all matching outbound traffic from a network. Forward proxies may be used to speed up external access for locations with slow connections (by caching the external resources and using those cached copies to service requests directly in the future), or may be used to restrict or monitor external access.

**Transparent Proxy**(Not important in this context)
A transparent proxy may be either a reverse or forward proxy (though nearly all reverse proxies are deployed transparently), the defining feature being the use of network routing to send requests through the proxy without any need for clients to configure themselves to do so, and often without the ability for those clients to bypass the proxy.


## Installation

Note: ATS is not available in centos7 as of now . It is however available in centos8.

There are two ways to install  ATS. One is installing from repository another is building by yourself. By building yourself you can modify the build. For centos7 we can build from source.

### Installing from repository

**Ubuntu**
```
    sudo apt-get install trafficserver
```
**RHEL / CentOS**
```
    wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
    sudo rpm -Uvh epel-release-latest-7*.rpm

    sudo yum install trafficserver
```

### Installing from source

You will need following packages.

**pkgconfig, libtool ,C++ compiler (gcc >= 4.3 or clang > 3.0), GNU make, OpenSSL or BoringSSL, pcre, libcap, flex (for TPROXY), hwloc, lua, zlib, curses (for traffic_top), curl (for traffic_top)**

To build Traffic server
```
git

autoconf

automake
```

#### Process



Note that only CentOS6 and later is supported on current releases, and for ATS v7.0.0 and forward, you will need e.g. devtoolset-3 on CentOS6.
The following packages must be installed:
```
$ sudo yum install gcc gcc-c++ pkgconfig pcre-devel tcl-devel expat-devel openssl-devel

```
On CentOS6, you also need

```
$ sudo yum install perl-ExtUtils-MakeMaker
```

It's also recommended that the following packages are installed, but they are not required (and they come pre-installed on many systems):
```
$ sudo make
```

If the unwind library is available, install this to get support for the crash log feature:
```
$ sudo yum install libunwind libunwind-devel
```

For building out of source, you also need:
```
$ sudo yum install autoconf automake libtool
```

###### ATS v9.0
ATS v9.0.0 drops CentOS 6 support because of minimum openssl version (v1.0.2). If you want, you can build OpenSSL (v1.0.2 or later) from source and use it for ATS 9.


##### ATS v8.0
For ATS v8.0.0 and later, a compiler with support for C++17 is required. You can install this using devtoolset-7 on both CentOS 6 and 7:
```
$ sudo yum install centos-release-scl
sudo yum install yum-utils
$ sudo yum-config-manager --enable rhel-server-rhscl-7-rpms
$ sudo yum install devtoolset-7

``` 
#####  Optionally, install ASAN
```
$ sudo yum install devtoolset-7-libasan-devel.x86_64

```
This installs a sufficiently modern compiler but it is not available by default. You will need to run the enable script that is part of the devtoolset. This will be
```
/opt/rh/devtoolset-6/enable
or
/opt/rh/devtoolset-7/enable

```

This needs to run in your shell, not in a subshell, which means running the command

```
source /opt/rh/devtoolset-7/enable

```

either by hand or in your .bash-rc. You can tell if this has been done correctly by running the command

```
which g++

```

Valid output should look like
**/opt/rh/devtoolset-7/root/usr/bin/g++**


If the path returned in not in the devtoolset directory structure you are not using the correct compiler.
Or, if you rather use LLVM 5.x, you can install that from EPEL with the following:
```
$ sudo yum install llvm5.0 llvm5.0-libs

```

##### GIT

To use Git, you must install the appropriate packages:
```$ sudo yum install git```

##### autoreconf
The first time you build trafficserver, you must run autoreconf to create the ./configure script:
```$ autoreconf -i```

##### configure and build
For more details, see the Building page.
```
$ ./configure && make
```


Clone from repository 
```
git clone https://git-wip-us.apache.org/repos/asf/trafficserver.git

git clone https://github.com/apache/trafficserver
```

Change the directory to the cloned directory

```
cd trafficserver/
```

First create a group called tserver
```
Sudo groupadd -g 175 tserver
sudo useradd -g 175 -u 176 -d /var/empty -s /sbin/nologin tserver

**Here -d gives the home directory for the user 
-s gives the shell for the account**

```

If you have cloned the repository from Git, you will need to generate the configure script before proceeding:

```
autoreconf -if
```

Install Luajit

Clone luajit
```
git clone https://luajit.org/git/luajit.git

```

Then make the luajit
```
Make
Sudo make install

```

**Then the header files are present in /usr/loca/include/ **


Traffic Server uses the standard configure script method of configuring the source tree for building. A full list of available options may always be obtained by running the following in the base directory of your unpackaged archive or Git working copy:

```
./configure --prefix=/opt/ats --with-user=tserver --enable-experimental-plugins –with-luajit=/usr/local

```

#### In case of errors ###
```
If you get errors in building -libtscpputil.so in ld


This error can be found using command 
/opt/rh/devtoolset-7/root/usr/libexec/gcc/x86_64-redhat-linux/7/ld -ltscpputil –verbose

** Create a symlink **
Sudo ln -s /opt/ats/lib/libtscpputil.so /usr/lib64


```


Once the source tree has been configured, you may proceed on to building with the generated Makefiles. The make check command may be used to perform sanity checks on the resulting build, prior to installation, and it is recommended that you use this

```
make
make check

```


With the source built and checked, you may now install all of the binaries, header files, documentation, and other artifacts to their final locations on your system.

```
sudo make install

```



##### Note Extra:

How to install with openssl 1.1.1


Download repo from openssl

Configure with 
./config --prefix=/usr --openssldir=/etc/ssl --libdir=lib
Make
Sudo make install

Ln -s /usr/lib/libssl.so.3 /usr/lib64
Ln -s /usr/lib/libcrypto.so.3 /usr/lib64
















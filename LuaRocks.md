# Lua Rocks and Using Lua to use scripts in ATS

Installing luarocks
```
wget https://luarocks.org/releases/luarocks-3.9.0.tar.gz

tar zxpf luarocks-3.9.0.tar.gz

Cd luarocks


../configure --with-lua-include=/usr/local/include/luajit-2.1/

Make

Sudo make install


```

**To find packages**

```
Sudo luarocks search luajwt

```

**To find installed packages**

```
Sudo luarocks list
```

If some package like lua-cjson does not work

**For lua-cjson**

Build the lua-cjson again with the source file
Adding the LDFLAGS -lluajit-5.1 in the Makefile
Then build and install from it

https://github.com/apache/trafficserver/issues/5158


**For lbase64**

Add in makefile
LDFLAGS= -lluajit-5.1

**Build Lua Crypto**

First build openssl 

Download openssl by cloning it from source
```
./config --prefix=/usr/local/ssl --openssldir=/usr/local/ssl --libdir=lib
```

Then you can build luacrypto

```
 ./configure OPENSSL_LIBS=/usr/local/ssl/ OPENSSL_CFLAGS=-I/usr/local/ssl/include/openssl LUA_CFLAGS=-I/usr/local/include/luajit-2.1 LUA_LIBS=/usr/local/bin

 ```

Then install with luarocks
```
luarocks install --verbose 

```


**To install luacrypto with another openssl**

Install the openssl in a folder for example /usr/local/ssl-old/

```
luarocks unpack luacrypto
sudo luarocks make luacrypto-0.3.2-2.rockspec  LDFLAGS=-lluajit-5.1 LUA_LIBS=-L/usr/local/lib/ OPENSSL_INCDIR=/usr/local/ssl-old/include OPENSSL_LIBDIR=/usr/local/ssl-old/lib/

```








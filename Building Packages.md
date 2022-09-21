# Building Packages

First you must install GO

Install go 1.7 in Linux

1. curl -LO https://storage.googleapis.com/golang/go1.7.linux-amd64.tar.gz

2. sudo tar -C /usr/local -xvzf go1.7.linux-amd64.tar.gz

3. mkdir -p ~/projects/{bin,pkg,src}

4. **sudo vi /etc/profile.d/path.sh**
    ```
    export PATH=$PATH:/usr/local/go/bin
    ```

5. **vi ~/.bash_profile**
    ```
    export GOBIN="$HOME/projects/bin"
    export GOPATH="$HOME/projects/src"
    ```
6. source /etc/profile && source ~/.bash_profile

7. Check if go is working
    vi ~/projects/src/hello.go
    ```
    package main

    import "fmt"

    func main() {
        fmt.Printf("Hello, World!\n")
    }
    ```

    go install $GOPATH/hello.go

    go install $GOPATH/hello.go


## Installing Traffic Control

**Downloading Traffic Control**
If any local work on Traffic Monitor, Traffic Router Golang, Grove or Traffic Ops is to be done, it is highly recommended that the Traffic Control repository be downloaded inside the $GOPATH directory. Specifically, the best location is $GOPATH/src/github.com/apache/trafficcontrol. Cloning the repository outside of this location will require either linking the actual directory to that point, or moving/copying it there.


**Install Docker and Docker Compose**

**First install docker
**
```
sudo yum install -y yum-utils

sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

sudo yum install docker-ce docker-ce-cli containerd.io

sudo systemctl start docker
sudo docker run hello-world
 sudo usermod -aG docker $USER
 newgrp docker

```

**Install docker-compose(>=2.0)**
```
DOCKER_CONFIG=${DOCKER_CONFIG:-$HOME/.docker}
 $ mkdir -p $DOCKER_CONFIG/cli-plugins
 $ curl -SL https://github.com/docker/compose/releases/download/v2.2.3/docker-compose-linux-x86_64 -o $DOCKER_CONFIG/cli-plugins/docker-compose
chmod +x $DOCKER_CONFIG/cli-plugins/docker-compose
Sudo cp docker-compose /usr/local/bin
```

**Finally build the packages**
Then inside the $GOPATH/src/github.com/apache/trafficcontrol type the following commands
```
./pkg -7 -v 
```
For centos 7 packages


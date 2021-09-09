# Development Tools

Before you start writing a Java SCORE, you need to install Java Development Kit(JDK) 11 or later version, goloop CLI 
tool, goloop server,and gradle distribution. Goloop CLI tool will be used to interact with blockchain. You can either 
use goloop local server or testnet to deploy and interact with your scores. We will use gradle to create workspace. You 
also need a text editor or IDE - for example [Intellij IDEA](https://www.jetbrains.com/idea/download/).

## Installing JDK
* **MacOS**
```shell
$ brew tap AdoptOpenJDK/openjdk
$ brew install --cask adoptopenjdk11
```
* **Ubuntu**
```shell
$ sudo apt install openjdk-11-jdk
```

## Installing goloop CLI
* Install make
```shell
$ sudo apt install make
```
* Install [golang 1.14+](https://golang.org/doc/install)
* Install RocksDB 6.22+
  * MacOS 
    ```shell
    $ brew update
    $ brew install rocksdb
    ```
  * Ubuntu
    ```shell
    $ sudo apt-get update
    $ sudo apt-get -y install build-essential libgflags-dev libsnappy-dev zlib1g-dev libbz2-dev liblz4-dev libzstd-dev
    $ git clone https://github.com/facebook/rocksdb.git
    $ cd rocksdb
    $ make shared_lib
    $ sudo cp --preserve=links ./librocksdb.* /usr/lib/
    $ sudo cp -r ./include/rocksdb/ /usr/include/
    ```

* Clone [goloop repo](https://github.com/icon-project/goloop)
```shell
$ git clone git@github.com:icon-project/goloop.git
```
* Build goloop CLI 
```shell
$ cd goloop
$ make goloop
```

Goloop binary file is located in `./bin/`. Add this in your `PATH`

TODO: Easy installation of goloop CLI

## Installing Goloop Local Node

* [Download and Install Docker](https://docs.docker.com/get-docker/)
* Build gochain-icon-image
```shell
$ git clone git@github.com:icon-project/goloop.git
$ cd goloop
$ make gochain-icon-image
```
If the command runs successfully, it generates the docker image like the following.

```shell
$ docker images goloop/gochain-icon

REPOSITORY            TAG       IMAGE ID       CREATED          SIZE
goloop/gochain-icon   latest    f674a8a67fa6   12 minutes ago   732MB
```

* Starting the gochain-local-node using [gochain-local](https://github.com/icon-project/gochain-local)

```shell
# Clone the gochain-local
$ git clone git@github.com:icon-project/gochain-local.git
$ cd gochain-local

# Start the node
$ ./run_gochain.sh start
>>> START iconee 9082 latest
1ee7a7941a4db785ad4aa54df314a82d8bdea6592af647bb414ddade28c7bbe4

$ docker ps
CONTAINER ID   IMAGE                        COMMAND                  CREATED          STATUS          PORTS                                                           NAMES
1ee7a7941a4d   goloop/gochain-icon:latest   "/entrypoint /bin/sh…"   27 seconds ago   Up 26 seconds   8080/tcp, 9080/tcp, 0.0.0.0:9082->9082/tcp, :::9082->9082/tcp   gochain-iconee

```

* Log messages will be generated at `./chain/iconee.log`
```shell
$ tail -f ./chain/iconee.log 
T|20210908-18:24:58.606122|b6b5|9f8312|NM|protocolhandler.go:275 Broadcast {0x0200} 148 0 sub=consensus
W|20210908-18:24:58.606307|b6b5|9f8312|CS|consensus.go:1126 sendVote: NotAvailable
D|20210908-18:24:58.606482|b6b5|9f8312|CS|consensus.go:321 enterStep {Height:76 Round:0 Step:stepPrecommitWait}
D|20210908-18:24:58.606616|b6b5|9f8312|CS|consensus.go:321 enterStep {Height:76 Round:0 Step:stepCommit}
I|20210908-18:24:58.609374|b6b5|9f8312|SV|transition.go:687 finalizeResult() total=62.351µs world=59.967µs receipts=2.384µs
D|20210908-18:24:58.609416|b6b5|9f8312|SV|transactionmanager.go:86 TM.NotifyFinalized waiters_before=0 waiters_after=0
D|20210908-18:24:58.609673|b6b5|9f8312|BM|manager.go:1120 Finalize(719151717355e6d630f90139da96676312ae133da902e86a92f8ce7634f84447)
D|20210908-18:24:58.609723|b6b5|9f8312|SV|manager.go:573 Validators:1 RoundLimitFactor:3 --> RoundLimit:1
```

* Stop the container
```shell
$ ./run_gochain.sh stop
>>> STOP gochain-iconee
gochain-iconee
gochain-iconee
```

TODO: Installation using Docker Compose

## Installing gradle
Using package manager

**Ubuntu**
* Install sdk in Ubuntu
```shell
$ curl -s "https://get.sdkman.io" | bash
```
* Install gradle
```shell
$ sdk install gradle 7.2
```

**MacOS**
```shell
$ brew install gradle
```

* Verify Installation
```shell
$ gradle -v
```
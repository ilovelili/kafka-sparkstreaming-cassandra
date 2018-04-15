# Docker container for Kafka - Spark streaming - Cassandra

This Dockerfile sets up a complete streaming environment for experimenting with Kafka, Spark streaming (PySpark), and Cassandra. It installs

* Kafka 0.10.2.1
* Spark 2.1.1 for Scala 2.11
* Cassandra 3.7

It additionnally installs

* Anaconda distribution 4.4.0 for Python 2.7.10
* Jupyter notebook for Python

## Quick start-up guide

Run container using [DockerHub image](https://hub.docker.com/r/ilovelili/kafka-sparkstreaming-cassandra)

```bash
docker run -p 4040:4040 -p 8888:8888 -p 23:22 -ti --privileged ilovelili/kafka-sparkstreaming-cassandra
```

Note that any changes you make in the notebook will be lost once you exit de container. In order to keep the changes, it is necessary put your notebooks in a folder on your host, that you share with the container, using for example

```bash
docker run -v `pwd`:/home/guest/host -p 4040:4040 -p 8888:8888 -p 23:22 -ti --privileged ilovelili/kafka-sparkstreaming-cassandra
```

Note:

* The "-v `pwd`:/home/guest/host" shares the local folder (i.e. folder containing Dockerfile, ipynb files, etc...) on your computer - the 'host') with the container in the '/home/guest/host' folder.
* Port are shared as follows:
  * 4040 bridges to Spark UI
  * 8888 bridges to the Jupyter Notebook
  * 23 bridges to SSH

SSH allows to get a connection to the container

```bash
ssh -p 23 guest@containerIP
```

where `containerIP` is the IP of th container (127.0.0.1 on Linux, `docker-machine ls` on Windows and Mac). Password is `guest`.

### Start services

Once run, you are logged in as root in the container. Run the startup_script.sh (in /usr/bin) to start

* SSH server. You can connect to the container using user 'guest' and password 'guest'
* Cassandra
* Zookeeper server
* Kafka server

```bash
startup_script.sh
```

### Connect, create Cassandra table, open notebook and start streaming

Connect as user 'guest' and go to 'host' folder (shared with the host)

```bash
su guest
```

Start Jupyter notebook

```bash
notebook
```

and connect from your browser at port host:8888 (where 'host' is the IP for your host. If run locally on your computer, this should be 127.0.0.1 or 192.168.99.100, check Docker documentation)

#### Start Kafka producer

Open kafkaSendDataPy.ipynb and run all cells.

#### Start Kafka receiver

Open kafkaReceiveAndSaveToCassandraPy.ipynb and run cells up to start streaming. Check in subsequent cells that Cassandra collects data properly.

#### Connect to Spark UI

It is available in your browser at port 4040

### Container configuration details

The container is based on CentOS 6 Linux distribution. The main steps of the building process are

* Install some common Linux tools (wget, unzip, tar, ssh tools, ...), and Java (1.8)
* Create a guest user (UID important for sharing folders with host!, see below), and install Spark and sbt, Kafka, Anaconda and Jupyter notbooks for the guest user
* Go back to root user, and install startup script (for starting SSH and Cassandra services), sentenv.sh script to set up environment variables (JAVA, Kafka, Spark, ...), spark-default.conf, and Cassandra

### User UID

In the Dockerfile, the line

```bash
RUN useradd guest -u 1000
```

creates the user under which the container will be run as a guest user. The username is 'guest', with password 'guest', and the '-u' parameter sets the linux UID for that user.

In order to make sharing of folders easier between the container and your host, **make sure this UID matches your user UID on the host**. You can see what your host UID is with

```bash
echo $UID
```

## Build and running the container from scratch

### Clone this repository

```bash
git clone https://github.com/ilovelili/kafka-sparkstreaming-cassandra
```

### Build

From Dockerfile folder, run

```bash
docker build -t kafka-sparkstreaming-cassandra .
```

It may take about 30 minutes to complete.

### Run

```bash
docker run -v `pwd`:/home/guest/host -p 4040:4040 -p 8888:8888 -p 23:22 -ti --privileged kafka-sparkstreaming-cassandra
```
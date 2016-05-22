# cassandra-spark-analytics
Supercharge your analysis of Cassandra data with Apache Spark

## Introduction

[Apache Cassandra](http://cassandra.apache.org/) is a fantastic scalable, fault-tolerant NoSQL database, however it is abhorrently hard to query your data outside the realms of what your datamodel will allow. [Apache Spark](http://spark.apache.org/) is a fast and general engine for large-scale data processing. 

In this guide I intend to demonstrate the power of Spark to provide analysis of your Cassandra data in ways that were previously thought impossible. This guide is aimed at anyone who is interested in data analysis.

## Requirements

This guide has been tested with the software versions outlined below. Older versions may work, test at your own risk. 

This guide was developed using [CentOS 7](https://www.centos.org/).

| Software | Version |
| --- | --- |
| [Cassandra](http://cassandra.apache.org/) | `2.2.5` |
| [Cqlsh](https://pypi.python.org/pypi/cqlsh/4.1.1) | `4.1.1` |
| [Cassandra Spark Connector](https://github.com/datastax/spark-cassandra-connector) | `1.5` |
| [Spark](http://spark.apache.org/) | `1.6.1` |
| [Docker](https://www.docker.com/) | `1.11.1` |
| [Docker-compose](https://docs.docker.com/compose/) | `1.7.1` |
| [Python](https://www.python.org/) | `2.7.5` |

## Setup

**Throughout this guide it will assume your working directory path is the base of this repo, YMMV.**

### Docker:
Firstly we need `docker` and `docker-compose`  installed and running. A good startup guide can be viewed [here](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-centos-7).

Check docker is functioning correctly by issuing command `docker ps`.

### Cassandra:

#### Start Cassandra container

We will start Cassandra with the included `compose/cassandra.yml` instruction file. This file can be edited if you know what you are doing but the defaults are fairly sane for this demo. 

The Cassandra Thrift port (9160) will bind to `127.0.0.1:9160` and we will use this to connect to cassandra via `cqlsh` from the host.

`docker-compose -f compose/cassandra.yml up -d`

Check the container has started with `docker ps`.

#### Install Cqlsh

Install `cqlsh` with command `pip install --user cqlsh==4.1.1`.

Test you can access cassandra correctly: `cqlsh 127.0.0.1`

#### Create schema

Create the Cassandra schema with:

`cqlsh 127.0.0.1 -f schema/spark_demo.cql`

Import the test dataset (use your relative path):

`echo "use spark_demo; COPY person_data (id, first_name, last_name, email, gender, ip_address) FROM 'schema/spark_demo_data.csv' WITH HEADER=true;" | cqlsh 127.0.0.1`

If it is sucessful you should get some output like: `1000 rows imported in 0.477 seconds.`

I generated the demo data using [Mockaroo](https://www.mockaroo.com/schemas/download).

```
echo "SELECT * FROM spark_demo.person_data;" | cqlsh 127.0.0.1

 id  | email                  | first_name | gender | ip_address    | last_name
-----+------------------------+------------+--------+---------------+-----------
 769 | esanchezlc@comcast.net |     Ernest |   Male | 165.66.44.126 |   Sanchez

...
```

### Spark

We will start Spark with the included `compose/spark.yml` instruction file. You will need to edit this file to set the relative path to `cassandra-spark-analytics/scripts` this directory will be mounted into the running Spark container to give Spark access to the processing scripts.

The Spark instructions are located in `docker/spark/Dockerfile` if you want to roll your own image.

#### Start Spark container

`docker-compose -f compose/spark.yml up -d`

Check the container has started with `docker ps`.

#### Execute a Spark job

For this demo we are going to group and count all the first names in our `person_data` table. In `cassandra-spark-analytics/scripts` is `count.py`. You will need to alter `CASSANDRA_DOCKER_IP` and insert the IP address of the running cassandra container:

`conf.set("spark.cassandra.connection.host", "CASSANDRA_DOCKER_IP")`

To get the Cassandra container's IP address run: 

`docker inspect --format '{{ .NetworkSettings.IPAddress }}' cassandra`

Once the IP address is updated you can execute the Spark job:

`docker exec -ti spark /spark/bin/spark-submit /jobs/count.py`

When the script finished executing you will be left with output like below: 

```
+----------+-----+
|first_name|count|
+----------+-----+
|    Brenda|   12|
|    Sharon|   12|
|   Deborah|   12|
|     Kelly|   11|
|     Diana|   10|
|    Carlos|   10|
|     Irene|    9|
|   Heather|    9|
|    Louise|    9|
|    Andrea|    9|
|   Jeffrey|    9|
|   Michael|    9|
|    Victor|    9|
|     Terry|    8|
|     Steve|    8|
|    Ernest|    8|
|     Larry|    8|
|      Lois|    8|
|      John|    8|
|   Richard|    8|
|     Louis|    8|
|     Ralph|    8|
|    Joseph|    8|
|      Mary|    8|
|     Billy|    8|
+----------+-----+
only showing top 25 rows
```

That concludes this demo. If you come up with anything cool submit a PR!

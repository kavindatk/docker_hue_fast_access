# Part 5: GUI-Based Querying and Fast Data Retrieval

<br/><br/>
<p align="center">
<picture>
  <img alt="docker" src="https://github.com/kavindatk/docker_hue_fast_access/blob/main/images/hue_logo.png" width="200" height="125">
</picture>

<picture>
  <img alt="docker" src="https://github.com/kavindatk/presto_spark_hue/blob/main/images/spark_logo.png" width="200" height="125">
</picture>

<picture>
  <img alt="docker" src="https://github.com/kavindatk/docker_hue_fast_access/blob/main/images/presto_logo.JPG" width="200" height="125">
</picture>
</p>



<br/><br/>

So far, we have successfully set up a <b>3-NameNode Hadoop cluster with Hive and Tez</b>.

Now we’re moving forward with the <b>installation of Presto and Hue</b> on our <b>3-NameNode Hadoop cluster</b>.

Since <b>Impala is a Cloudera proprietary product</b>, we’ve chosen to use <b>Presto</b> as our fast SQL query engine.

In the next steps, I’ll show you how to <b>install Presto</b>.

Going forward, I also plan to <b>integrate Hue with Presto, Hive, Spark, and other open-source tools</b>.
The <b>Hue configuration</b> shared below will include settings for connecting to these components — some of which are covered in previous articles.
<br/><br/>


# Presto 

<picture>
  <img alt="docker" src="https://github.com/kavindatk/docker_hue_fast_access/blob/main/images/presto_logo.JPG" width="200" height="100">
</picture>



## Step 1: Install Presto on Ubuntu

The following steps will guide you through the Presto installation process on an Ubuntu environment.


### 1.1 Download Presto and set Environment Variables

```bash
wget https://repo1.maven.org/maven2/com/facebook/presto/presto-server/0.293/presto-server-0.293.tar.gz
tar -xvzf presto-server-0.293.tar.gz
mv presto-server-0.293 presto
wget https://repo1.maven.org/maven2/com/facebook/presto/presto-cli/0.293/presto-cli-0.293-executable.jar -O /opt/presto/bin/presto # This need to put in bin dir
```

```bash
nano ~/.bashrc
```

```bash
# Presto Related Option
export PRESTO_HOME=/opt/presto
export PATH=$PATH:$PRESTO_HOME/bin
```

```bash
source ~/.bashrc
```

### 1.2 Configure Presto 

```bash
cd presto
mkdir etc data
cd etc
```

#### Add or modify following files

* node.properties

```xml
node.environment=production
node.id=presto-node-1 # This should be unique for each server
node.data-dir=/opt/presto/data
```

* jvm.config

```xml
-server
-Xmx16G
-XX:+UseG1GC
-XX:G1HeapRegionSize=32M
-XX:+UseGCOverheadLimit
-XX:+ExplicitGCInvokesConcurrent
-XX:+HeapDumpOnOutOfMemoryError
-XX:+ExitOnOutOfMemoryError
-Djdk.attach.allowAttachSelf=true
```

* config.properties

** IF Coordinator: 

```xml
coordinator=true
node-scheduler.include-coordinator=false
http-server.http.port=8090
query.max-memory=5GB
query.max-memory-per-node=1GB
discovery-server.enabled=true
discovery.uri=http://bigdataproxy:9090 # This is HAPROXY for HA mode
```

** IF Worker: 

```xml
coordinator=false
http-server.http.port=8090
query.max-memory=5GB
query.max-memory-per-node=1GB
discovery.uri=http://bigdataproxy:9090 # This is HAPROXY for HA mode
```

* catalog/hive.properties

```bash
mkdir catalog
cd catalog
nano hive.properties
```

```xml
connector.name=hive-hadoop2
hive.metastore.uri=thrift://bigdataproxy:9083

hive.config.resources=/opt/presto/etc/hadoop/hdfs-site.xml,/opt/presto/etc/hadoop/core-site.xml

# Force managed tables by default
hive.non-managed-table-writes-enabled=true
hive.non-managed-table-creates-enabled=true
```

* hadoop/ #hadoop config for presto

```bash
mkdir hadoop
cd haoop
<copy hadoop core-site.xml , hdfs-site.xml and yarn-sie.xml and modified for presto , this need if you have HA setup> 
```

Modified only core-site.xml to HA proxy via HDFS access

```xml
      <property>
              <name>fs.defaultFS</name>
              <value>hdfs://bigdataproxy:8020</value>
      </property>
```


### 1.3 Start Presto Service

On Each Node , run following command 

```bash
launcher start
```


You can stop and check status via 

```bash
launcher stop/status
```

###  Verify Installation

Log into Presto via cli ot browser 


Browser : http://172.27.41.131:8090/ui/

CLI :

```bash
presto --server bigdataproxy:9090 --catalog hive --schema default
```

```sql
SELECT * FROM system.runtime.nodes; # Check running node list

SHOW TABLES FROM hive.<schema name>;

DESCRIBE hive.<schema>.<table>;
or
SHOW COLUMNS FROM  hive.<schema>.<table>;


CREATE SCHEMA <schema>;

SHOW CATALOGS;
or
SHOW SCHEMAS FROM hive;

CREATE TABLE hive.<schema>.<table> (
    id INT,
    name VARCHAR,
    city VARCHAR
);

INSERT INTO hive.<schema>.<table> (id, name, city)
VALUES (1, 'Alice', 'New York'), (2, 'Bob', 'London');	

CREATE TABLE hive.<schema>.<table> (
    product_id INT,
    product_name VARCHAR,
    price DOUBLE
)
WITH (
    format = 'ORC', -- Or 'PARQUET', 'CSV', etc.
    external_location = '<hdfs path>' 
);

INSERT INTO hive.<schema>.<table> (product_id, product_name, price)
VALUES (101, 'Laptop', 1200.00), (102, 'Mouse', 25.50);
```

<picture>
  <img alt="docker" src="https://github.com/kavindatk/docker_hue_fast_access/blob/main/images/presto_check.JPG" width="600" height="200">
</picture>

<br/><br/>

# Spark 

<picture>
  <img alt="docker" src="https://github.com/kavindatk/docker_hue_fast_access/blob/main/images/spark_logo.png" width="200" height="100">
</picture>


## Step 1: Install Spark on Ubuntu

The following steps will guide you through the Spark installation process on an Ubuntu environment.


### 1.1 Download Presto and set Environment Variables

```bash
wget https://dlcdn.apache.org/spark/spark-3.5.6/spark-3.5.6-bin-hadoop3.tgz
tar -xvzf spark-3.5.6-bin-hadoop3.tgz
mv spark-3.5.6-bin-hadoop3 spark
mv spark /opt/
```

```bash
nano ~/.bashrc
```

```bash
# Spark Related Option
export SPARK_HOME=/opt/spark
export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin
export PYSPARK_PYTHON=/usr/bin/python3
```

```bash
source ~/.bashrc
```

### 1.2 Configure Spark 

```bash
cd spark
cd conf
cp spark-env.sh.template spark-env.sh
cp spark-defaults.conf.template spark-defaults.conf
cp log4j2.properties.template log4j2.properties
```

### spark-env.sh

#### For Master : 

```bash
# Spark HA Configureations


# Java Home
export JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64

# Spark Configuration
export SPARK_MASTER_HOST=mst01 #Replace with master hostname
export SPARK_MASTER_PORT=7077
export SPARK_MASTER_WEBUI_PORT=8880

# Worker Configuration
export SPARK_WORKER_CORES=4
export SPARK_WORKER_MEMORY=4g
export SPARK_WORKER_PORT=7078
export SPARK_WORKER_WEBUI_PORT=8081

# History Server
export SPARK_HISTORY_OPTS="-Dspark.history.ui.port=18080 -Dspark.history.fs.logDirectory=hdfs://hadoop-cluster/spark-logs"

# Daemon Memory
export SPARK_DAEMON_MEMORY=1g

# Hadoop Configuration
export HADOOP_CONF_DIR=$HADOOP_CONF_DIR
export YARN_CONF_DIR=/etc/hadoop/etc/hadoop

# Hive Configuration
export HIVE_CONF_DIR=/etc/hive/conf

# Spark Classpath
#export SPARK_DIST_CLASSPATH=$(hadoop classpath)
export SPARK_DIST_CLASSPATH=$(/opt/hadoop/bin/hadoop classpath)
```

#### For Workers :

```bash
# Spark HA Configureations


# Java Home
export JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64

# Worker Configuration
export SPARK_WORKER_CORES=4
export SPARK_WORKER_MEMORY=4g
export SPARK_WORKER_PORT=7078
export SPARK_WORKER_WEBUI_PORT=8081

# History Server
export SPARK_HISTORY_OPTS="-Dspark.history.ui.port=18080 -Dspark.history.fs.logDirectory=hdfs://hadoop-cluster/spark-logs"

# Daemon Memory
export SPARK_DAEMON_MEMORY=1g

# Hadoop Configuration
export HADOOP_CONF_DIR=$HADOOP_CONF_DIR
export YARN_CONF_DIR=/etc/hadoop/etc/hadoop

# Hive Configuration
export HIVE_CONF_DIR=/etc/hive/conf

# Spark Classpath
#export SPARK_DIST_CLASSPATH=$(hadoop classpath)
export SPARK_DIST_CLASSPATH=$(/opt/hadoop/bin/hadoop classpath)
```


### spark-defaults.conf

```bash
# Spark Master HA Configuration
spark.master                    spark://mst01:7077,mst02:7077,mst03:7077

# Zookeeper for HA coordination (using your existing ZK cluster)
spark.deploy.recoveryMode       ZOOKEEPER
spark.deploy.zookeeper.url      mst01:2181,mst02:2181,mst03:2181
spark.deploy.zookeeper.dir      /spark-ha

# History Server Configuration
spark.eventLog.enabled          true
spark.eventLog.dir              hdfs://hadoop-cluster/spark-logs
spark.history.fs.logDirectory   hdfs://hadoop-cluster/spark-logs
spark.history.ui.port           18080

# Hive Integration
spark.sql.hive.metastore.uris   thrift://bigdataproxy:9083
spark.sql.warehouse.dir         hdfs://hadoop-cluster/user/hive/warehouse
spark.sql.catalogImplementation hive

# Thrift Server Configuration
spark.sql.hive.thriftServer.singleSession   false
spark.sql.thrift.server.port                10001

# Resource Configuration
spark.executor.memory           2g
spark.executor.cores            2
spark.driver.memory             1g
spark.driver.cores              1

# Serialization
spark.serializer                org.apache.spark.serializer.KryoSerializer

# Dynamic Allocation (Optional)
spark.dynamicAllocation.enabled             true
spark.dynamicAllocation.minExecutors        1
spark.dynamicAllocation.maxExecutors        10
spark.dynamicAllocation.initialExecutors    2

```


### workers

```bash
slv01
slv02
```

### HDFS Folder for Spark event logging and the history server


```bash
hdfs dfs -mkdir -p /spark-logs
```

<br/>

Once the above modifications are complete, you need to <b>distribute the Spark folder and its configuration files to all master and worker nodes in the cluster</b>.
This ensures that <b>Spark is properly installed and configured on every node</b>, and all nodes can work together as part of the same Spark cluster.

<br/>

### 1.3 Start Spark Service

On Each Master Node 

```bash
start-master.sh
start-history-server.sh
start-thriftserver.sh --hiveconf hive.server2.thrift.port=10001

stop-master.sh
stop-history-server.sh
stop-thriftserver.sh
```


On Each Worker/Slave Node 

```bash
start-worker.sh  spark://mst01:7077,mst02:7077,mst03:7077

stop-worker.sh  
```


###  Monitor Services and Verify the Cluster

Once all services are started, you can monitor the cluster status using the HAProxy web interface.

The following steps show how to:

* Verify the overall health of the cluster
* Check Hive connectivity
* Ensure that all components are running and communicating correctly

Run following command on any server 

```bash
pyspark \
  --master spark://mst01:7077,mst02:7077,mst03:7077 \
  --conf spark.deploy.recoveryMode=ZOOKEEPER \
  --conf spark.deploy.zookeeper.url=mst01:2181,mst02:2181,mst03:2181 \
  --conf spark.deploy.zookeeper.dir=/spark-ha
```

```python

import pyspark
from pyspark.sql import SparkSession
from pyspark.sql import functions as f
from pyspark.sql.types import StringType , ArrayType , IntegerType

spark = SparkSession.builder.master("spark://mst01:7077,mst02:7077,mst03:7077").appName("Test").enableHiveSupport().getOrCreate()

df = spark.read.table("presto_db.presto_web")

df.printSchema()

df.show()
```

#### PySpark Terminal


<picture>
  <img alt="docker" src="https://github.com/kavindatk/presto_spark_hue/blob/main/images/verfy_spark.JPG" width="800" height="400">
</picture>


<br/>

#### Spark Worker Web 

You can test the Spark master failover by shutting down the currently active Spark master node.

If everything is configured correctly, Spark will automatically switch to the next available master node, ensuring high availability.


<picture>
  <img alt="docker" src="https://github.com/kavindatk/presto_spark_hue/blob/main/images/spark_worker.JPG" width="800" height="400">
</picture>


<br/>

#### Spark History Server


<picture>
  <img alt="docker" src="https://github.com/kavindatk/presto_spark_hue/blob/main/images/spark_history.JPG" width="800" height="400">
</picture>

<br/>

# HUE

<picture>
  <img alt="docker" src="https://github.com/kavindatk/presto_spark_hue/blob/main/images/hue_logo.png" width="200" height="100">
</picture>


## Step 1: Install HUE on Ubuntu

The following steps will guide you through the <b>HUE installation process on an Ubuntu environment</b>.

In this setup, I’m using the <b>official HUE Docker image</b> to install and run HUE.

Since this involves Docker and HAProxy, I’ve written <b> separate articles </b> covering those topics in detail.
You can refer to them to get a better understanding:

👉 [Docker and HAProxy](https://github.com/kavindatk/haproxy_docker_setup)

<br/>

### 1.1 Download Presto and set Environment Variables

```bash
docker pull gethue/hue:latest
```


### 1.2 Configure Presto 

Create Hue user and hue database in mariadb , log into any mariadb database (We have already setup the galera cluster)


```sql
CREATE DATABASE hue;
CREATE USER 'hue'@'%' IDENTIFIED BY 'hue';
GRANT ALL PRIVILEGES ON hue.* TO 'hue'@'%';
FLUSH PRIVILEGES;
EXIT;
```

<br>

<picture>
  <img alt="docker" src="https://github.com/kavindatk/presto_spark_hue/blob/main/images/hue_db_user.JPG" width="600" height="400">
</picture>

<br/>

### 1.2 Configure Hue to Connect with Hive, Presto and SparkSQL

For set up configurations, 'hue.ini' file should be edited

(https://github.com/cloudera/hue/blob/master/desktop/conf.dist/hue.ini) 

```bash
sudo mkdir /opt/hue/
sudo mkdir /opt/hue/conf
cd /opt/hue/conf/
<download and copy hue.ini to conf path>
chmod 777 hue.ini
sudo chown -R hadoop:hadoop /opt/hue/
```

#### Hue Configuration (hue.ini)

The modification of the hue.ini file can be quite lengthy, so I’ve uploaded the full configuration file to the GitHub repository for easy reference.

🔗 Please refer to the file in the repo to see all settings.

<br/>  

👉 [hue.ini](https://github.com/kavindatk/presto_spark_hue/blob/main/hue_config/conf/hue.ini)
    

Since we are using a 3-NameNode Hadoop setup, make sure to update specific sections of the file with the correct NameNode IP addresses or hostnames, especially in places related to HDFS, YARN, Hive, and Spark integrations.

### 1.3 Start HUE Docker Containner 

```bash
docker run -d -p 8888:8888   --name hue   -v /opt/hue/conf/hue.ini:/usr/share/hue/desktop/conf/hue.ini   --network host   hue-custom
```

###  Monitor Services and Verify the Hue

In the web browser , you can access HUE interface by entering following URL

```xml
http://mst01:8888/hue/editor?editor=67
http://mst02:8888/hue/editor?editor=67
http://mst03:8888/hue/editor?editor=67

or if you enabled access via HAProxy

http://bigdataproxy:8888/hue/editor?editor=67 
```


<picture>
  <img alt="docker" src="https://github.com/kavindatk/presto_spark_hue/blob/main/images/hue_web.png" width="700" height="300">
</picture>


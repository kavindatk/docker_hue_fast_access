# GUI-Based Querying and Fast Data Retrieval

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

* hadoop/<hadoop config for presto>

```bash
mkdir hadoop
cd haoop
<copy hadoop core-site.xml , hdfs-site.xml and yarn-sie.xml and modified for presto , this need if you have HA setup> 
```

Modfied only core-site.xml to HA proxy via HDFS access

```xml
      <property>
              <name>fs.defaultFS</name>
              <value>hdfs://bigdatacluster:8020</value>
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
wget https://downloads.apache.org/spark/spark-3.5.6/spark-3.5.6-bin-without-hadoop.tgz
tar -xvzf spark-3.5.6-bin-without-hadoop.tgz
mv spark-3.5.6-bin-without-hadoop spark
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
```

### spark-env.sh

#### For Master : 

```bash
export SPARK_MASTER_HOST=mst01 # For other NNs place their hostname
export SPARK_MASTER_PORT=7077
export SPARK_MASTER_WEBUI_PORT=8880
export SPARK_DAEMON_JAVA_OPTS="-Dspark.deploy.recoveryMode=ZOOKEEPER \
  -Dspark.deploy.zookeeper.url=zk1:2181,zk2:2181,zk3:2181 \
  -Dspark.deploy.zookeeper.dir=/spark"
export HADOOP_CONF_DIR=/opt/hadoop/etc/hadoop
export SPARK_DIST_CLASSPATH=$(hadoop classpath)
export JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64
```

#### For Workers :

```bash
export SPARK_DAEMON_JAVA_OPTS="-Dspark.deploy.recoveryMode=ZOOKEEPER \
  -Dspark.deploy.zookeeper.url=zk1:2181,zk2:2181,zk3:2181 \
  -Dspark.deploy.zookeeper.dir=/spark"
export HADOOP_CONF_DIR=/opt/hadoop/etc/hadoop
export SPARK_DIST_CLASSPATH=$(hadoop classpath)
export JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64
```


### spark-defaults.conf

```bash

spark.master                    spark://bigdataproxy:7077
spark.submit.deployMode         cluster
spark.sql.catalogImplementation hive
spark.eventLog.enabled          true
spark.eventLog.dir              hdfs:///spark-logs
spark.history.fs.logDirectory   hdfs:///spark-logs
```

### HDFS Folder for Spark event logging and the history server


```bash
hdfs dfs -mkdir -p /spark-logs
```

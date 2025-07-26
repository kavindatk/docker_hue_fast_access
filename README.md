# GUI-Based Querying and Fast Data Retrieval

<br/><br/>
<p align="center">
<picture>
  <img alt="docker" src="https://github.com/kavindatk/docker_hue_fast_access/blob/main/images/docker.jpg" width="" height="125">
</picture>
  
<picture>
  <img alt="docker" src="https://github.com/kavindatk/docker_hue_fast_access/blob/main/images/hue_logo.png" width="200" height="125">
</picture>

<picture>
  <img alt="docker" src="https://github.com/kavindatk/docker_hue_fast_access/blob/main/images/presto_logo.JPG" width="200" height="125">
</picture>
</p>

<br/><br/>

So far, we have successfully set up a <b>3-NameNode Hadoop cluster with Hive and Tez</b>.

Now, we’ll enhance the setup by adding:

* GUI-based tools for easier querying
* Fast SQL engines like Impala or Presto for faster data access and analytics

Before we proceed with these tools, I will first <b>install Docker</b> on all three NameNodes. This will help us easily deploy and manage GUI tools and other services.
The steps below show how to <b>install Docker on the 3 NameNode</b> setup.
<br/><br/>


# Docker

<picture>
  <img alt="docker" src="https://github.com/kavindatk/docker_hue_fast_access/blob/main/images/docker.jpg" width="" height="125">
</picture>

## Step 1: Install Docker on Ubuntu

The following steps will guide you through the Docker installation process on an Ubuntu environment.


### 1.1 Update package index and install prerequisites:

```bash
sudo apt update
sudo apt install apt-transport-https ca-certificates curl gnupg lsb-release
```

### 1.2 Add Docker's official GPG key:

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

### 1.3 Add Docker repository:

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### 1.4 Install Docker Engine:

```bash
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io
sudo apt install docker-compose-plugin
sudo apt install docker-compose

sudo apt install python-is-python3 # For Presto
```

### Verify Installation

```bash
# Start and Enable service
sudo systemctl start docker
sudo systemctl enable docker
sudo systemctl status docker

# Add current user to docker group
sudo usermod -aG docker $USER

# Check Docker version
docker --version

# Check Docker Compose version
docker compose version
# or if using standalone version:
docker-compose --version

# Test Docker installation
sudo docker run hello-world
```

<picture>
  <img alt="docker" src="https://github.com/kavindatk/docker_hue_fast_access/blob/main/images/docker_log.JPG" width="800" height="400">
</picture>



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
``


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


browser : http://172.27.41.131:8090/ui/

cli :

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

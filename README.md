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
  <img alt="docker" src="https://github.com/kavindatk/docker_hue_fast_access/blob/main/images/presto_logo.JPG" width="200" height="125">
</picture>



## Step 1: Install Presto on Ubuntu

The following steps will guide you through the Presto installation process on an Ubuntu environment.


### 1.1 Download Presto and set Environment Variables

```bash
wget https://repo1.maven.org/maven2/com/facebook/presto/presto-server/0.293/presto-server-0.293.tar.gz
tar -xvzf presto-server-0.293.tar.gz
mv presto-server-0.293 presto
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
```

* config.properties

** IF Coordinator: 

```xml
coordinator=true
node-scheduler.include-coordinator=false
http-server.http.port=8080
query.max-memory=5GB
query.max-memory-per-node=1GB
discovery-server.enabled=true
discovery.uri=http://bigdataproxy:8080 # This is HAPROXY for HA mode
```

** IF Worker: 

```xml
coordinator=false
http-server.http.port=8080
query.max-memory=5GB
query.max-memory-per-node=1GB
discovery.uri=http://bigdataproxy:8080 # This is HAPROXY for HA mode
```

* catalog/hive.properties

```bash
mkdir catalog
cd catalog
```

```xml
connector.name=hive-hadoop2
hive.metastore.uri=thrift://bigdataproxy:9083 # This is HAPROXY for HA mode
hive.config.resources=/etc/hadoop/conf/core-site.xml,/etc/hadoop/conf/hdfs-site.xml
```


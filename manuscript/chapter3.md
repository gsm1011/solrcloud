# Configure Solr in Cloud mode

## Create `zookeeper` quorum

A quorum is a replicated group of zookeeper instances. All servers in the quorum
have copies of the same configuration files. 

that determines the status of a
clustered application. If the majority zookeeper instance is not maintained, the
quorum based application is down. Here majority means larger than half of the
instances are up, e.g. in a 5 instance zookeeper quorum, 3 will be the
majority. So, if at least 3 instances are up, the cluster is up, otherwise the
cluster is down. Normally, we set the number of zookeeper instances to be odd,
for e.g. `3, 5, ..., 2N + 1` etc. 

### Installing zookeeper

First download zookeeper from
[the zookeeper website](http://zookeeper.apache.org/releases.html) and put it to
`/usr/local/` directory. You can use the `wget` command to download the file
too. 

`$ wget <zookeeper-download-url>` 

Decompress the file and create symbolic link: 

	# cd /usr/local
	# cp ~/zookeeper-3.4.8.tar.gz .
	# tar xvf zookeeper-3.4.8.tar.gz
	# ln -s zookeeper-3.4.8 zookeeper

So far, zookeeper has been installed into `/usr/local/zookeeper` directory. In
order for each user to run zookeeper commands directly, path
`/usr/local/zookeeper/bin` needs to be added to the `PATH` environment
variable. E.g., we can add the following line into the `~/.bashrc` file: 

	export PATH=/usr/local/zookeeper/bin:$PATH
	
By default, we will run zookeeper server in the bin directory, the configuration
file named `zoo.cfg` under the `/usr/local/zookeeper/conf` directory will be
used. We can explicitely specify the config files to use when starting the
zookeeper server. 

The content of the `zoo.cfg` file is shown below: 

```
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/tmp/zookeeper/0
clientPort=2181
maxClientCnxns=60
#autopurge.snapRetainCount=3
#autopurge.purgeInterval=1
```

The `time` and `limit` parameters should be tuned against the network status and
other configurations of the cluster. The data directory specified by `dataDir`
will be used by zookeeper to store the metadata maintained by the zookeeper
quorum. If there is only one zookeeper instance in the quorum, the configuration
is enough. But if more than one zookeeper instances are needed for the quorum,
following changes need to be made: 

1. Under each instance's data directory, create file `myid`, with the id as the
   content. 
2. Configure the quorum, specifically, add content similar to the content below
   into the configuration file of each zookeeper instance: 
   
```
server.1=server1:2888:3888
server.2=server2:2888:3888
server.3=server3:2888:3888
```

Here, `server`, `server2` and `server3`are the hostnames of the zookeeper
instances, these names should be resolvable by each server. The port number
`2888` is used by zookeeper instances to communicate with each other, and port
number `3888` is used for zookeeper leader election.

### Setup a zookeeper quorum with 3 instances on the same server

Create a seed zookeeper configuration file from the sample file
`zoo_sample.cfg`. 

	# cd /usr/local/zookeeper/conf
	# cp zoo_sample.cfg zoo.cfg
	
The content of file `zoo.cfg` is: 

```
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/opt/zookeeper/0
clientPort=2181
#maxClientCnxns=60
#autopurge.snapRetainCount=3
#autopurge.purgeInterval=1
server.1=localhost:2888:3888
server.2=localhost:2889:3889
server.3=localhost:2890:3890
```

From this seed configuration file, we can create 3 seperate config files,
`zoo1.cfg`, `zoo2.cfg` and `zoo3.cfg`. Each of these files should have different
`dataDir` and `clientPort`, everything else should be the same as the seed
config file `zoo.cfg`. 

	# grep dataDir *cfg
	zoo.cfg:dataDir=/opt/zookeeper/0
	zoo1.cfg:dataDir=/opt/zookeeper/1
	zoo2.cfg:dataDir=/opt/zookeeper/2
	zoo3.cfg:dataDir=/opt/zookeeper/3
	
	# grep clientPort *cfg
	zoo.cfg:clientPort=2181
	zoo1.cfg:clientPort=2182
	zoo2.cfg:clientPort=2183
	zoo3.cfg:clientPort=2184
	
Now, let's create the data directory and create the `myid` file for each
instance. 

	# mkdir -pv /opt/zookeeper/{1,2,3}
	# touch /opt/zookeeper/{1,2,3}/myid
	# echo 1 > /opt/zookeeper/1/myid
	# echo 2 > /opt/zookeeper/2/myid
	# echo 3 > /opt/zookeeper/3/myid
	# tree /opt/zookeeper/ --charset=ascii
	/opt/zookeeper/
	|-- 1
	|   `-- myid
	|-- 2
	|   `-- myid
	`-- 3
	    `-- myid
		
	3 directories, 3 files

Next, let's start each zookeeper instance. 

	# cd /usr/local/zookeeper
	# bin/zkServer.sh start conf/zoo1.cfg
	# bin/zkServer.sh start conf/zoo2.cfg
	# bin/zkServer.sh start conf/zoo3.cfg
	# netstat -an | grep 218
	tcp46      0      0  *.2182                 *.*                    LISTEN
	tcp46      0      0  *.2183                 *.*                    LISTEN
	tcp46      0      0  *.2184                 *.*                    LISTEN

### Setup a zookeeper quorum with 3 instances on different servers

First, you need to have zookeeper installed on all the servers, for example, you
can install under the `/usr/local` directory.

Let's use `server1`, `server2` and `server3` as the server names. And in order
for the servers to resolve these hostnames, we can put the following entries
into the `/etc/hosts` file on each server. 

```
192.168.1.1 server1
192.168.1.2 server2
192.168.1.3 server3
```

The zookeeper configuration file will have following content: 

```
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/opt/zookeeper/data
clientPort=2181
#maxClientCnxns=60
#autopurge.snapRetainCount=3
#autopurge.purgeInterval=1
server.1=server1:2888:3888
server.2=server2:2888:3888
server.3=server3:2888:3888
```

Remember, don't forget to create a `myid` file under the `/opt/zookeeper/data`
directory.

Once the configuration changes are done, we can start the zookeeper instances on
all the three servers. 

	$ zkServer.sh /usr/local/zookeeper/conf/zoo.cfg

### Exploring zookeeper with zkCli.sh

**zkCli.sh** is a bash script under directory `$ZOOKEEPER_HOME/bin`. We can use
this tool to verify the configuration and make changes if necessary. 

**zkcli.sh** is tool shipped with solr installation, it is mainly used for solr
specific changes on zookeeper. By default, this file is locate at
`${SOLR_HOME}/server/scripts/cloud-scripts` directory.

To connect to the zookeeper server, we use `zkCli.sh` script. 

	$ zkCli.sh localhost:2181
	
**Note:** 
The hostname and port is optional, it will defaults to `localhost:2181`. But it
is handy if you want to connect to a zookeeper instance on a different `server`
or `port`. 

After we connect to the zookeeper command line, we can execute commands to
check the configuration files on zookeeper, e.g. the solr configs directory and
collections directory etc. 

	[zk: localhost:2181(CONNECTED) 0] ls /configs
	[gettingstarted]
	[zk: localhost:2181(CONNECTED) 1] ls /live_nodes
	[172.22.220.118:8983_solr, 172.22.220.118:7574_solr, 172.22.220.118:8985_solr]

The `/configs` directory contains the configurations for solr collections and
cores. The `/collections` directory contains collections and cores. And the
`/live_nodes` directory contains a list of nodes in the solr cluster as shown in
the previous command. 

Directory structure of a new solr node. 

```
node3
`-- solr
    |-- solr.xml
	`-- zoo.cfg
		
1 directory, 2 files
```

## Upload a `config` for solr `collection`

In solr standalone mode, we know that we need to tell solr where `solr_home` is
when starting a solr instance, and all the cores on that solr instance will
locate under the `solr_home` directory. But in solrcloud, the solr configuration
files and collection metadata are stored in zookeeper, we need to upload the
`config` directory before creating solr collections. 

	$ ./server/scripts/cloud-scripts/zkcli.sh -zkhost localhost:2181 -cmd
    upconfig -confdir solr/conf -confname conf1

The command will upload the solr collection configuration files in `solr/conf`
into zookeeper with name as conf1. We can verify this command with the
`zkCli.sh` command.

	[zk: localhost:2181(CONNECTED) 0] ls /configs
	[conf1]
	
Now, if we create a new collection, we can use this configuration. For example,
we can create a collection called `c1` with the following command. 

	$ solr create_collection -c c1 -n conf1 -shards 2 -replicationFactor 2
	
The following command confirms that the collection has been created
successfully. 

	[zk: localhost:2181(CONNECTED) 2] ls /collections
	[c1]
	
Alternatively, we can use the `zkcli.sh` to list the collections. 

	$ zkcli.sh -zkhost localhost:2181 -cmd list
	...
	DATA:
	    {"configName":"conf1"}
	  /collections/c1/state.json (0)
	   DATA:
	        {"c1":{
	             "replicationFactor":"2",
	             "router":{"name":"compositeId"},
	             "maxShardsPerNode":"2",
	             "autoAddReplicas":"false",
	             "shards":{
	               "shard1":{
	                  "range":"80000000-ffffffff",
	                  "state":"active",
	                  "replicas":{}},
	               "shard2":{
	                  "range":"0-7fffffff",
	                  "state":"active",
	                  "replicas":{}}}}}
	...
	

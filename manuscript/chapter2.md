# Configure Solr in standalone mode

In order to create solr in standalone mode, we need to create a directory for
`<solr_home>`. This directory should contain a file called `solr.xml`.

	$ cd $HOME
	$ mkdir standalone
	$ cp solr/server/solr/solr.xml standalone/
	$ solr/bin/solr start -s standalone/

Now, we can check the status of the solr instance

	$ bin/solr status
	
	Found 1 Solr nodes:
	Solr process 52337 running on port 8983
	{
	  "solr_home":"/Users/c05495a/solrcloud/solr/standalone",
	    "version":"6.1.0 4726c5b2d2efa9ba160b608d46a977d0a6b83f94 - jpountz -
	    2016-06-13 09:46:58",
	      "startTime":"2016-08-05T14:12:46.753Z",
	        "uptime":"0 days, 0 hours, 26 minutes, 32 seconds",
		  "memory":"83.5 MB (%4.3) of 1.9 GB"
	}

If that's what you get, congratulations, you just started solr in stand alone mode. 

Next, we can go to the solr admin web interface to manage our solr
instance. The URL of the instance is located
[here](http://localhost:8983/solr/).

## Create your first solr core

A solr core is corresponding, in large sense, to a table in
`RDMBS`. It consists of a number of configuration files and
libraries. Specifically, file `core.properties` is the main
configuration file for the core. For example, it defines the name of
the core, the schema file etc. By default, the core directory will
contain two directories, `conf/` and `data/`. The `conf/` directory
contains all the core related configuration files, mainly in `.xml`
format. And `data/` directory contains the core index files and
transaction log files. 

Now the structure of the `standalone/` directory will be similar to
the following: 

```
standalone/
|-- README.md
|-- core1
|   |-- conf
|   |   |-- admin-extra.html
|   |   |-- admin-extra.menu-bottom.html
|   |   |-- admin-extra.menu-top.html
|   |   |-- clustering
|   |   |   `-- carrot2
|   |   |       |-- kmeans-attributes.xml
|   |   |       |-- lingo-attributes.xml
|   |   |       `-- stc-attributes.xml
|   |   |-- currency.xml
|   |   |-- db-data-config.xml
|   |   |-- elevate.xml
|   |   |-- lang
|   |   |   |-- contractions_ca.txt
|   |   |   |-- ... (omited for brevity)
|   |   |   `-- userdict_ja.txt
|   |   |-- mapping-FoldToASCII.txt
|   |   |-- mapping-ISOLatin1Accent.txt
|   |   |-- protwords.txt
|   |   |-- schema.xml
|   |   |-- solrconfig.xml
|   |   |-- spellings.txt
|   |   |-- stopwords.txt
|   |   |-- synonyms.txt
|   |   |-- update-script.js
|   |   `-- xslt
|   |       |-- example.xsl
|   |       |-- example_atom.xsl
|   |       |-- example_rss.xsl
|   |       |-- luke.xsl
|   |       `-- updateXml.xsl
|   |-- core.properties
|   |-- data
|   |   |-- index
|   |   |   |-- segments_8
|   |   |   `-- write.lock
|   |   `-- tlog
|   |       |-- tlog.0000000000000000000
|   |       |-- ... (Omitted for brevity)
|   |       `-- tlog.0000000000000000007
|   `-- lib
`-- solr.xml
```

**Warning:**
Done create file `core.properties`, it will be generated automatically,
otherwise you will get an error saying that the core already exist. 

Now go to the [web UI](http://localhost:8983/solr), then click `Core Admin`
on the left panel, next click the `Add Core` button on the top, enter the
`name` and `instanceDir` and keep `dataDir`, `config` and `schema` as
default. The `Name` properties will be written to file `core.properties`
under the `instanceDir` directory. By default you can use the same name for
both values, but that's not required. For example, if you have a directory
called `core1_inst` for core `core`, you can specify the `instanceDir`
value to be `core1_inst` while the name to be `core1`.

**Note:**
Remember that, all those core instance directories should be located within
`${solr_home}` directory. 

If everything works fine so far, congrats, you just created your first
solr core in solr standalone mode. Next, let's load some data into the
newly created core and run some query over that.

## Loading data into solr core

We can use the `$SOLR_HOME/bin/post` tool to load some data into the newly
create solr core. 

	$ bin/post -c core1 http://lucene.apache.org/solr -recursive 1 -deplay 1

After it is down, we can check the size of data directory: 

	$ du -sh data
	2.0M     data

The command output tells that we have successfully loaded data into the
core. Actually, you can also check the content of this directory. The
`data/index/` and `data/tlog` directory now contains some files. 

## Querying the core

After loading the data, we can query it either using the web UI or from
command line using `curl`. Another option is to use some GUI tools like
`postman`. 

From the web UI, select `core` from the drop down list, then click query,
you will see a number of options for a query. By default, it will `/select`
all the documents from the core. By clicking the `Execute Query` at the
bottom, you will be able to retrieve some documents you just posted. 

After the query is executed, you will see a URL like
`http://localhost:8983/solr/core1/select?indent=on&q=*:*&wt=json` on top of
the result. Now, you can use curl to submit a query using this URL: 

	$ curl "http://localhost:8983/solr/core1/select?indent=on&q=*:*&wt=json"

After some testing, we can delete the sample data so that we can add real
and useful data later. To delete the content, the easiest way is to go to
the core web UI and click `Documents`, use `Document Type` as `XML` and in
the `Document(s)` field paste `<delete><query>*:*</query></delete>` then
click `Submit Document`. If everything is correct, you will see response
similar to the following. 

```
Status: success
Response:
{
  "responseHeader": {
      "status": 0,
	     "QTime": 13
	}
}
```
To confirm deletion, you can run the query again, it should give you output
similar to the following, note that the value of `numFound` is `0`. 

```
{
  "responseHeader":{
      "status":0,
	     "QTime":0,
		 params":{
		       "q":"*:*",
			ndent":"on",
			  "wt":"json"}},
			onse":{"numFound":0,"start":0,"docs":[] }}
```


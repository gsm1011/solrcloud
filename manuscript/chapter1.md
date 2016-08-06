# Installing Solr

## Downloading solr
Download solr from the [website](http://lucene.apache.org/solr/downloads.html).

## Install solr

Solr installation is very simple, you can untar the file and create symbolic link. 

	# cd /usr/local/
	# tar xvf solr-6.1.0.tgz
	# ln -s solr-6.1.0 solr

Solr will be installed in `/usr/local/solr`

In order to be able to access solr from anywhere, you can add the following
to your `.bashrc` file.

	export SOLR_HOME=/usr/local/solr
	export PATH=$PATH:${SOLR_HOME}/bin

To test the installation, you can run the following command with normal 

	$ solr version
	6.1.0

So far, you have successfully installed solr on one server. If you want to
install solr on multiple servers, it is suggested that you create some
script or use automation tools such as ansible, chef etc. to automate the
installation process.

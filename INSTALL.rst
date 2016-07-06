Building JMXToolkit
===================

The project comes with the configuration files for the current distribution,
so one can build the project as is - or - query a specific set of JMX enabled
hosts and build a system that matches those hosts. This allows for example
to create different instances to graph clusters with different release levels.

To build the project with the current JMX properties file simply run::

    $ mvn package

It builds a jar file containing all that is needed for the current release
of HBase. The jar file can be run like this:

    $ java -jar target/jmxtoolkit-2.0-toolkit.jar

Add the parameters as per the README file to execute the examples listed there.

vvv OUTDATED - v1.0 ONLY vvv

See the section about Cacti below for how to proceed from this point.

Custom Building
===============

The above builds the jar file against the current known JMX attributes and
operations of this release level. Sometimes though you may have custom setups
that have other attributes or operations (such as an older release level) or
you want to add custom Nagios type checks (see README.txt) or use a password
to secure your JMX setup.

You can build a jar using an immediate JMX discovery process. It scans the
named hosts and creates a custom properties file. Here HOSTNAME1 must point
to a master server and HOSTNAME2 to a region server and/or Hadoop data node.

Please note that one password is assumed for the access control rights::

    $ ant -DPASSWORD=mypass -DHOSTNAME1=master.foo.com -DHOSTNAME2=slave.foo.com \
     create-properties

Once done you can jar the custom properties file up with the rest of the
project::

    $ ant jar

Or do both in one step like so::

    $ ant -DPASSWORD=mypass -DHOSTNAME1=master.foo.com -DHOSTNAME2=slave.foo.com \
     create-properties jar

Finally there is an option to change the prefix that is used to name the
custom properties and resulting jar file; usually the prefix is the current
release version, for example "hbase-0.20.4". With the example below you build a
"mycluster-jmxtoolkit.jar" using a "mycluster-jmx.properties" file internally.
Also note that the scripts are automatically renamed as well and adjusted to
use this custom prefixed build. This allows to run more than one setup on one
Cacti instance for example, since everything is named differently. Here the
build command::

    $ ant -DPASSWORD=mypass -DHOSTNAME1=master.foo.com -DHOSTNAME2=slave.foo.com \
     -Dproperties.prefix=mycluster create-properties jar

Cacti Install
=============

Once the jar is build it can be used on a Cacti server. First the jar is copied
over the network into the Cacti scripts directory (which can vary between
installs, so make sure you know what you are doing). Next extract the scripts::

    $ cd $CACTI_HOME/scripts
    $ unzip hbase-0.20.4-jmxtoolkit.jar bin/*
    $ chmod +x bin/*

Once the scripts are in place test the basic functionality::

    $ bin/jmxtkcacti-hbase-0.20.4.sh slave.foo.com hadoopFSDatasetState

    Remaining:552378507264 Capacity:1880106897408 DfsUsed:1244164909593

The jar also includes a set of Cacti templates that you can import into it and
use as a starting point to graph various values exposed by Hadoop's and HBase's
JMX MBeans.

The templates where kindly provided by Ed Capriolo, more on the topic can be
found at http://www.jointhegrid.com/hadoop/

Especially the installation instructions may be useful to set up JMX first:

  http://www.jointhegrid.com/svn/hadoop-cacti-jtg/trunk/doc/INSTALL.txt

Please note that as of HBase 0.20.3 the hbase-env.sh already contains commented
out JMX export statements so that setting up JMX for HBase is simply done by
commenting out those few lines::

    export HBASE_JMX_BASE="-Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false"
    export HBASE_MASTER_OPTS="$HBASE_MASTER_OPTS $HBASE_JMX_BASE -Dcom.sun.management.jmxremote.port=10101"
    export HBASE_REGIONSERVER_OPTS="$HBASE_REGIONSERVER_OPTS $HBASE_JMX_BASE -Dcom.sun.management.jmxremote.port=10102"

Also note that the default setup is not employing any password authentication
as currently the HBase UI also does not have any security built in yet and
therefore is open on the LAN level.

Nagios Install
==============

With Nagios you can follow the same steps described above in the Cacti install.
You can then wire the Nagios checks to the supplied check script. If you have
checks defined in the properties file then you only specify the object and
attribute or operation to query. If not then you can specify the check within
Nagios like so::

    $ bin/jmxtknagios-hbase-0.20.4.sh master.foo.com hadoopFSNamesystemState \
    TotalLoad "0|2:WARN%20%7B0%7D:1500:<|1:FAIL%20%7B0%7D:2000:<"

    FAIL 1,501

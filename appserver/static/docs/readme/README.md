## Monitoring of Pivotal CloudFoundry with JMX

* Version : 2.0.4
* Date : January 2014
* Author : Damien Dallimore, ddallimore@splunk.com

## Overview

This app provides the means to :

* connect to any local or remote JVM's JMX server, either via the remote JMX interface,attaching to a local JVM process or using an MX4J HTTP based connector.
* query any MBean running on that server
* extract any MBean attributes (simple, composite or tabular)
* invoke MBean operations
* listen for MBean notifications and declare custom notification filters
* write attributes and operation results out in a default key/value format, or plugin your own custom format, for SPLUNK indexing and searching
* declare clusters of JVM's for large scale JVM deployments
* manage the JMX Modular Input programmatically via the REST API

Tested to date against Hotspot ,JRockit and IBM J9 JVM's.

Currently supports the following  JMX Connectors :
 
* rmi  (JSR160 Standard Implementation and MX4J's JSR160 Implementation)
* iiop (JSR160 Standard Implementation and MX4J's JSR160 Implementation)
* local (MX4J only)
* soap (MX4J only)
* hessian (MX4J only)
* burlap (MX4J only)


## Dependencies

* Splunk 5.0+ , if you only want to use as a JMX Modular Input
* Splunk 6.0+ , if you want to use the Simple XML dashboards also
* Java Runtime 1.6+
* Supported on all Splunk platforms : Windows, Linux, MacOS, Solaris, FreeBSD, HP-UX, AIX

## Setup

* Optionally set your JAVA_HOME environment variable to the root directory of you JRE installation.If you don't set this , the input will look for a default installed java executable on the path.
* Untar the release to your $SPLUNK_HOME/etc/apps directory
* Restart Splunk

## Configuration

The data collection logic is implemented as a modular input.

You can configure your JMX inputs via Manager->DataInputs->JMX

You will need to configure your JMX config file, this is the config file where you specify JMX server(s), MBeans and MBean attributes/operations/notifications.
You can create as many config files,with any name, as you want and place them in the jmx_cf/bin/config directory

In your setup stanzas, you can optionally specify an alternate config file directory location relative to SPLUNK_HOME ie: etc/apps/someotherapp
This might come in handy if you want to create another Splunk App that depends on this JMX App , and ship your own config files.

The app sets up a default input stanza for you. To get started you should edit the default config.xml file and then enable the stanza. 

### CONFIG.XML

Note : also refer to the configuration schema documentation in the docs directory

Included in the distribution is a comprehensive sample config.xml
This sample file queries various java.lang MBeans and obtains attributes
You can use this as a basis for creating your own config files for whatever MBeans and attributes/operations you need to poll across your environment
To try out this sample config.xml file , just set the values for your environment in the jmxserver element , and you should be good to go.

"jmxpoller" is the root xml element

A custom formatter can be specified.Only 1 may be specified for the entire configuration.
This is a Java implementation of the "com.dtdsoftware.splunk.formatter.Formatter" interface.
The formatter class must be placed on the java classpath.
If the optional formatter declaration is omitted, then the default formatter will be used.

Refer to the javadoc API in the docs directory for viewing bundled Formatter implementations or the signature of the interfaces for creating your own custom implementations.

1 or more "jmxserver" elements can be included in this XML defintion ie:  if you need to accesss multiple different jmx sources within the same scheduled execution
Each "jmxserver" element you declare will be run in parallel.

"jmxuser" and "jmxpass" are optional , if not specified, they will be ignored

"host" can be a hostname, dns alias or ip address. 

"jmxport" is the JMX server port to connect to.

"protocol"(default is rmi) is the jmx service protocol to use : rmi, iiop and mx4j specific protocols soap, hessian etc..

For more advanced finer grained control over the format of the jmx service URL you can also specify the "stubSource"(default is jndi), "encodedStub" and url "lookupPath"(default is jmxrmi).
Refer to the schema docs for more details.

Or, to facilitate complete user flexibility, you can enter the full jmx service url as a raw string using the "jmxServiceURL" attribute.

This is raw jmx service url in standard format "service:jmx:protocol:sap"

If this attribute is set, it will override any other set connection related attributes("host", "jmxport", "protocol", "stubSource", "encodedStub" and "lookupPath")

You can also attach directly to a locally running JVM process by specifying the Process ID in the "pid" attribute.
In this case, the "host", "jmxport", "jmxuser","jmxpass" attributes will not be used.
The host value will default to the localhost's name.

For more dynamic flexibility,rather than specifying a static Process ID value in the config xml , you can also specfy the name of a PID file that contains the JVM's Process ID.
You set this in the "pidFile" attribute.
Many long running Java processes output the PID value to a file, particularly if using a JVM Service Wrapper such as Tanuki.
Each time the mod input poller runs, it will dynamically inspect the value of the PID from this file.
This way, you dont have to alter the config.xml each time you do a JVM restart.

You can also specify the path to a command to execute to output the JVM's PID.
You set this in the "pidCommand" attribute.
This command might be a custom script that looks for the JVM process and extracts the PID to STD OUT.
On Linux you could do this with a simple script that uses ps, grep and awk.

Example : ps -eafH | grep "java" | grep -v "grep" | awk '{print $2}'

If the pidCommand outputs multiple PIDs on multiple lines , then Splunk for JMX will handle this by spawning additional JMXServer elements internally in memory.

Look at the example xml config file in the bin/config directory for usage examples.

Groups of jmxserver's that share that same mbeans can be grouped together in a cluster element, so that you only have to declare the MBeans/MBean attributes/operations once ie: a cluster of JEE appservers, a cluster of Hadoop or Cassandra nodes etc...This will be useful for enterprises with very large scale JVM deployments.
MBean inheritance is supported, so you can have mbeans defined that are common to the cluster, and mbeans that are
specific to a cluster member.

At index time the "host" field is extracted and transformed into the SPLUNK internal host value.

"jvmDescription" is just meta data, useful for searching on in SPLUNK where you might have multiple JVM's on the same host

By default, no timestamp is added , instead relying on the SPLUNK index time as the event time.
However you may  customise the 3 bundled formatters and configure them to prepend a timestamp if you wish.
See the examples in the default config.xml file

For MBean definitions , standard JMX object name wildcard patterns * and ? are supported for the "domain" and "properties" attributes 
http://download.oracle.com/javase/1,5.0/docs/api/javax/management/ObjectName.html

If no values are specified for the "domain" and "properties" attributes , the value will default to the * wildcard

The MBean's canonical objectname will be written out to SPLUNK.

If you would like the components of the MBean canonical objectname broken out into individual fields, then there is a custom formatter available which you can declare in your config XML file to achieve this :

''<formatter className="com.dtdsoftware.splunk.formatter.TokenizedMBeanNameFormatter" />''

Single, composite and tabular MBean attributes are supported.

You can set the "dumpAllAttributes" value to true and all of an MBean's attributes will be extracted.

ie: dump all the attributes in the java.lang domain
''<mbean domain="java.lang" properties="*" dumpAllAttributes="true"/>''

ie: dump all the attributes in all of the cassandra domains
''<mbean domain="org.apache.cassandra.*" properties="*" dumpAllAttributes="true" />''

Or you can specify each individual attribute you want to extract using the "attribute" element.

For attributes that are multi level ie: composite and tabular attributes , then you can use a ":" delimited notation for specifying the attribute name.
See examples of this in config.xml

MBean operation invocation is supported.
You can declare operations that return a result or not , take arguments or not.
Operation overloading is supported for methods of the same name with different signatures.

The following parameter types are supported :

* int
* float
* double
* long
* short
* byte
* boolean
* char
* string

Internally these get autoboxed into their respective Object counterparts.

See config_operations_example.xml for usage scenarios.

MBean notification listening is supported simply by adding a "<notificationListener />" element as a child of any MBean.
And you can declare custom notification filters "<notificationListener filterImplementationClass=com.foo.MyFilter/>" .
The filter is the canonical name of a class implementing the javax.management.NotificationFilter interface.

See config_notifications.xml for an example configuration.

#### Example output line (default formatter)

host=damien-laptop,jvmDescription="My test JVM",mbean="java.lang:type=Threading",count="47"

### SUPPORTED RETURN DATA TYPES FOR OPERATIONS AND ATTRIBUTES

Simple, Composite and Tabular MBean data structures are supported.

The value obtained from the Attribute or as the return type of an Operation can be :

1. Any primitive type(Object wrapped)
2. String
3. Array
4. List
5. Map
6. Set

Collection types will be recursively deeply inspected, so you can have Collections of Collections etc..

### JVM "ATTACH" NATIVE LIBRARYS

When connecting directly to local JVM using a process ID, native librarys are used. You shouldn't need to do anything, they are loaded automatically.

Windows : JRE_HOME/bin/attach.dll
Linux : JDK_HOME/jre/lib/i386/libattach.so

NOTE : For some reason the Linux "attach" library is only packaged in the JRE that is part of a JDK install. Weird. So if you don't have libattach.so , you can get it from the JDK and copy it into the jre lib directory.


### JMX SERVERS

To set up a JMX server for remote access via jvm system properties at startup, see this :
http://download.oracle.com/javase/1.5.0/docs/guide/management/agent.html#remote

### MX4J SUPPORT

If you wish to use MX4J as your JMX implementation for remote connectors(rmi and iiop) or use any of the MX4J specific JMX connectors(soap, burlap, hessian, local), then it is simply a matter of setting the USE_MX4J variable in the jmx_cf/bin/jmx.py script to True.

For more details about MX4J browse here : http://mx4j.sourceforge.net

Note : If using any of the HTTPS connectors(soap+ssl, hessian+ssl, burlap+ssl), the root certification authority must be present in the trusted certificates, normally stored in the "$JAVA_HOME/jre/lib/security/cacerts" file. 

### JVM HEAP Settings

The JMX modular input executes all of the stanzas in a single JVM instance.
If you need to boost HEAP size , then you can adjust the variables MIN_HEAP and MAX_HEAP in in the jmx_cf/bin/jmx.py script

### ADDING JAVA CLASSES TO THE APPLICATION CLASSPATH

You can dump a jar file in the "jmx_cf/bin/lib/" directory and it will be automatically loaded. 
Why would you want to do this ? Well perhaps you have created a custom formatter implementation as described above.

### ADDING JAVA CLASSES TO THE BOOT CLASSPATH

Any classes that you need prepended to the java bootclasspath should be put in "jmx_cf/bin/lib/boot".
Why would you want to do this ? Well perhaps you are targeting a JVM with some proprietary JMX logic that requires additional libraries on the JMX client side ie: using the IIOP connector with IBM  Websphere products is one such scenario I've encountered.


## Logging

Any runtime errors will get written to $SPLUNK_HOME/var/log/splunk/splunkd.log


## Troubleshooting

* check your firewall setup
* check the correct spelling/case of your MBeans and MBean attribute/operation definitions
* check your hostname and port
* check your user and pass
* check that your JVM remote JMX access is correctly setup , can you connect using JConsole ?
* check your process ID if trying to attach to a local JVM
* ensure your XML config adheres to the XSD
* look for errors in the log files or via Splunk search in the _internal index
* check for JMX connection timeouts
* check that the MBean server is not adding/removing domains and mbeans whilst you are concurrently polling them
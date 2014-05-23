#splunk-pcf-monitor

A splunk app for monitoring and alerting of Pivotal Cloud Foundry using Pivotal OpsMetrics.

splunk-pcf-monitor is based off of the work done by [damiendallimore](https://github.com/damiendallimore/SPLUNK4JMX)

## Setup
These directions assume you already have a installed splunk server to work from.  If you do not, you can get splunk and installation instructions from:
[Splunk](http://www.splunk.com/download)

This product has been tested with Splunk 6.0 and 6.1.  If you are using older versions some modification may be required.
 
## Procedure
- Download this project as a zip file.  Change to tar.gz?
- Login to splunk the splunk UI.  Select "manage apps" and choose to install app from file.  Point to the previously downloaded .tar.gz file
- Once the app is installed, log in to your splunk server and edit the config files to point to your OpsMetrics installation
- edit /opt/splunk/etc/apps/jmx_cf/bin/config/example_config.xml to your opsmetric and save as new file
  	/opt/splunk/etc/apps/jmx_cf/bin/config
		poc_pcf_config.xml - <jmxserver host="10.32.97.28" jvmDescription="CF POC OpsMetrics" jmxport="44444" jmxuser="jmw" jmxpass="jmw">

- In the splunk UI, go to settings -> data inputs -> JMX -> add new JMX input
	JMX Input Name "Prod PCF"
	Config File Name "prod_pcf_config.xml"
	Set sourcetype -> manual
	Polling Frequency -> 120
	Source type -> jmx
	Moresettings, index -> jmx_cf


add email
	sed -ie 's/userid@domain.com/mjseid@monsanto.com/g' /opt/splunk/etc/apps/jmx_cf/local/savedsearches.conf

go to splunk ui ->system settings->email alert settings
	mail host -> mail.monsanto.com


Here is an illustration:

![Raspberry Pi Wiring](doc/rpi-wiring.jpg)


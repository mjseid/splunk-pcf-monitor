#splunk-pcf-monitor

A splunk app for monitoring and alerting of Pivotal Cloud Foundry using Pivotal OpsMetrics.

Splunk-pcf-monitor is based off of the work done by [damiendallimore](https://github.com/damiendallimore/SPLUNK4JMX)

## Setup
These directions assume you already have an installed splunk server to work from.  If you do not, you can get splunk and its installation instructions from:
[Splunk](http://www.splunk.com/download)

This product has been tested with Splunk 6.0 and 6.1.  If you are using older versions some modification may be required.
 
## Procedure
- Download this project as a zip file.  Change to tar.gz?
- Login to splunk the splunk UI.  Select "manage apps" and choose to install app from file.  Point to the previously downloaded .tar.gz file
- Once the app is installed, log in to your splunk server and edit the config files to point to your OpsMetrics installation
- edit /opt/splunk/etc/apps/jmx_cf/bin/config/example_config.xml to your opsmetric and save as new file
```
cp /opt/splunk/etc/apps/jmx_cf/bin/config/example_config.xml /opt/splunk/etc/apps/jmx_cf/bin/config/prod_pcf_config.xml
vi /opt/splunk/etc/apps/jmx_cf/bin/config/prod_pcf_config.xml
```
The only line you have to change will look like this:
```
<jmxserver host="<IP_OF_YOUR_OPSMETRICS_VM>" jvmDescription="<DESCRIPTION_OF_YOUR_PCF_ENVIONRMENT>" jmxport="44444" jmxuser="<User you selected for PCF OPSMETRICS>" jmxpass="<PASSWORD_YOU_SELECTED_FOR_OPSMETRICS>">
```
- In the splunk UI, go to settings -> data inputs -> JMX -> add new JMX input.  Here is an example of a new input for monitoring aproduction PCF deployment:
<table>
  <tr>
    <th>JMX Input Name</th><th>Prod PCF</th>
  </tr>
  <tr>
    <td>Config File Name</td><td>prod_pcf_config.xml</td>
  </tr>
  <tr>
    <td>Set SourceType</td><td>manual</td>
  </tr>
  <tr>
    <td>Polling Frequency</td><td>60</td>
  </tr>
  <tr>
    <td>Source Type</td><td>jmx</td>
  </tr>
  <tr>
    <td>More Settings -> index</td><td>jmx_cf</td>
  </tr>
</table>

- Repeat the following two steps for any additional PCF enviornments you would like to monitor.


## Alerting
This app comes with some pre-defined alerts out of the box.  To recieve emails based on these alerts, you must do the following:
- Ensure that splunk is configured to send email alerts.  This includes setting the correct mail host and any required credentials in the splunk system settings.
- You can perform a bulk update of the email address for predefined alerts by logging on to the splunk server and editing the following file:
```
sed -ie 's/userid@domain.com/myid@mydomain.com/g' /opt/splunk/etc/apps/jmx_cf/local/savedsearches.conf

```
where myid@mydomain.com is the email (or comma seperated list of emails) you wish to send alerts to 


Here is an illustration:

![Raspberry Pi Wiring](doc/rpi-wiring.jpg)


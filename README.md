logstash-vmware
===============

Logstash configs and filters for handling ESXi and vSphere 5.1+ messages.


## Files

1. [Logstash Parser](https://github.com/harrytruman/logstash-vmware/blob/master/logstash-parser.conf): Filters and message mutations for ESX and vCenter messages.

2. [Logstash Forwarder](https://github.com/harrytruman/logstash-vmware/blob/master/logstash-forwarder.conf): Central forwarder and environment tagging of messages.

3. Coming soon: nxlog config

## Log Message Workflow

#### Shipping
Log messages get shipped to the Logstash collector/forwarder by standard syslog on ESX hosts (and vCenter appliances) or nxlog on Windows. 

#### Forwarding
The central Logstash forwarding instance receives messages, tags them as ESX or vCenter, and forwards them to Redis.

#### Parsing
The Logstash parsing instance receives messages from Redis, performs tag-based filtering and parsing, and sends them to Elasticsearch for indexing.

## Examples:

The vast majority of messages are parsed properly with just a few filters:

Message:
````
<166>2013-12-23T21:14:04.070Z hostname.com Vpxa: [50775B90 verbose 'VpxaHalCnxHostagent'
opID=WFU-feba478c] [WaitForUpdatesDone] Completed callback
````

Filter:
````
<%{POSINT:syslog_pri}>%{TIMESTAMP_ISO8601:time} %{SYSLOGHOST:hostname} %{SYSLOGPROG}:
(?<messagebody>(?<esxi_system_info>(?:\[%{DATA:esxi_thread_id} %{DATA:esxi_loglevel}
\'%{DATA:esxi_service}\'\ ?%{DATA:esxi_opID}])) \[%{DATA:esxi_service_info}]\
(?<esxi_message>(%{GREEDYDATA})))
````

Results:
````
{
"syslog_pri":         "166"
"time":               "2013-12-23T21:14:04.070Z"
"hostname":           "hostname.com"
"program":            "Vpxa"
"esxi_system_info":   [50775B90 verbose 'VpxaHalCnxHostagent' opID=WFU-feba478c]"
"esxi_thread_id":     "50775B90"
"esxi_loglevel":      "verbose"
"esxi_service":       "VpxaHalCnxHostagent"
"esxi_opID":          "opID=WFU-feba478c"
"esxi_service_info":  "WaitForUpdatesDone"
"esxi_message":       "Completed callback"
"messagebody":        [50775B90 verbose 'VpxaHalCnxHostagent' opID=WFU-feba478c] [WaitForUpdatesDone] Completed callback"
}
````

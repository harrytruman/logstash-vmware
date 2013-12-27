logstash-vmware
===============

###Logstash configs and filters for handling ESXi and vSphere 5.x+ messages.

If you're familiar with logs from VMware systems, you'll probably already know that their log formats are rather...diverse.  These patterns are the result of my searching and finding only one other Logstash config for parsing message from anything related to VMware.  I needed a central logging system to handle messages specifically from my ESXi 5.x hosts and vCenter running on Windows Server 2008/2012 (ugh).

Credit to [Martin Seener](https://gist.github.com/martinseener) for his [Grok ESXi 5.x Pattern](https://gist.github.com/martinseener/5238576).  His pattern was the only one I found for *anything* involving VMware so I heavily modified it to meet my needs.  They're not the cleanest and they're certainly not the most well-written, but they do the job very nicely.  If you find them useful, feel free to suggest improvements!

## Configs

1. [Logstash Parser](https://github.com/harrytruman/logstash-vmware/blob/master/logstash-parser.conf): Performs tag-based filtering/parsing and sends them to Elasticsearch for indexing.

2. [Logstash Forwarder](https://github.com/harrytruman/logstash-vmware/blob/master/logstash-forwarder.conf): Central forwarder, environment tagging of messages, and forwarding to Redis.

3. [NXLOG Config](https://github.com/harrytruman/logstash-vmware/blob/master/nxlog.conf): Ships messages from Windows to Logstash.

## Filter Examples:

The vast majority of messages are parsed properly with just a few filters:

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

From this message:
````
<166>2013-12-23T21:14:04.070Z hostname.com Vpxa: [50775B90 verbose 'VpxaHalCnxHostagent' opID=WFU-feba478c] [WaitForUpdatesDone] Completed callback
````

Parsed by this filter:
````
<%{POSINT:syslog_pri}>%{TIMESTAMP_ISO8601:time} %{SYSLOGHOST:hostname} %{SYSLOGPROG}: (?<messagebody>(?<esxi_system_info>(?:\[%{DATA:esxi_thread_id} %{DATA:esxi_loglevel} \'%{DATA:esxi_service}\'\ ?%{DATA:esxi_opID}])) \[%{DATA:esxi_service_info}]\ (?<esxi_message>(%{GREEDYDATA})))
````

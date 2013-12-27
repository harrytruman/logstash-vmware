logstash-vmware
===============

###Logstash configs and filters for handling ESXi and vSphere 5.x+ messages.

If you're familiar with logs from VMware systems, you'll probably already know that their log formats are rather...diverse.  These patterns are the result of my searching and finding only one other Logstash config for parsing message from anything related to VMware.  I needed a central logging system to handle messages specifically from my ESXi 5.x hosts and vCenter running on Windows Server 2008/2012 (ugh).

Credit to [Martin Seener](https://gist.github.com/martinseener) for his [Grok ESXi 5.x Pattern](https://gist.github.com/martinseener/5238576).  His pattern was the only one I found for *anything* involving VMware so I heavily modified it to meet my needs.  They're not the cleanest and they're certainly not the most well-written, but they do the job very nicely.  If you find them useful, feel free to suggest improvements!

## Configs

1. [Logstash Parser](https://github.com/harrytruman/logstash-vmware/blob/master/logstash-parser.conf): Performs tag-based filtering/parsing and sends them to Elasticsearch for indexing.

2. [Logstash Forwarder](https://github.com/harrytruman/logstash-vmware/blob/master/logstash-forwarder.conf): Central forwarder, environment tagging of messages, and forwarding to Redis.

3. [NXLOG Config](https://github.com/harrytruman/logstash-vmware/blob/master/nxlog.conf): Ships messages from Windows to Logstash.

## Filter Examples

This message:

````
<166>2013-12-27T16:12:57.896Z esxcac01094n01.corp.costco.com Vpxa: [507F9B90 verbose 'vpxavpxaInvtHost' opID=WFU-e579383e] [HostChanged] Found update for tracked MoRef vim.HostSystem:ha-host\n
````

Parsed by this filter:

````
<%{POSINT:syslog_pri}>%{TIMESTAMP_ISO8601:time} %{SYSLOGHOST:hostname} %{SYSLOGPROG}: (?<messagebody>(?<esxi_system_info>(?:\[%{DATA:esxi_thread_id} %{DATA:esxi_loglevel} \'%{DATA:esxi_service}\'\ ?%{DATA:esxi_opID}])) \[%{DATA:esxi_service_info}]\ (?<esxi_message>(%{GREEDYDATA})))
````

Index and displayed like this:

````json
{
  "_index": "logstash-2013.12.27",
  "_type": "logs",
  "_id": "Y9sWc0EnQ5q5TRVle-Xwgg",
  "_score": null,
  "_source": {
    "@timestamp": "2013-12-27T16:12:57.896Z",
    "tags": [ "esx", "np" ],
    "syslog_pri": "166",
    "time": "2013-12-27T16:12:57.896Z",
    "hostname": "hostname.com",
    "program": "Vpxa",
    "esxi_system_info": "[507F9B90 verbose 'vpxavpxaInvtHost' opID=WFU-e579383e]",
    "esxi_thread_id": "507F9B90",
    "esxi_loglevel": "verbose",
    "esxi_service": "vpxavpxaInvtHost",
    "esxi_opID": "opID=WFU-e579383e",
    "esxi_service_info": "HostChanged",
    "esxi_message": "Found update for tracked MoRef vim.HostSystem:ha-host",
    "syslog_severity_code": 6,
    "syslog_facility_code": 20,
    "syslog_facility": "local4",
    "syslog_severity": "informational",
    "ip_address": "172.24.69.100",
    "message-raw": "<166>2013-12-27T16:12:57.896Z hostname.com Vpxa: [507F9B90 verbose 'vpxavpxaInvtHost'
      opID=WFU-e579383e] [HostChanged] Found update for tracked MoRef vim.HostSystem:ha-host\n",
    "message": "[507F9B90 verbose 'vpxavpxaInvtHost' opID=WFU-e579383e] [HostChanged] Found update for
      tracked MoRef vim.HostSystem:ha-host"
  },
  "sort": [ 1388160777896 ]
}
````

<a href="http://imgur.com/2dA4WGI"><img src="http://i.imgur.com/2dA4WGI.png" title="Screenshot from Kibana" /></a>

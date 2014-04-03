Logstash with ESXi and vCenter
===============

###Logstash configs and filters for handling ESXi and vSphere 5.x+ messages.

Credit to [Martin Seener](https://gist.github.com/martinseener) for his [Grok ESXi 5.x Pattern](https://gist.github.com/martinseener/5238576).  His pattern was the only one I found for *anything* involving VMware, so I heavily modified it to meet my needs.  They're not the cleanest and they're certainly not the most well-written, but they do the job very nicely.  If you find them useful, feel free to suggest improvements!

## Configs

1. [Logstash](https://github.com/harrytruman/logstash-vmware/blob/master/logstash.conf): Retrieves messages from Redis. Performs tag-based filtering/parsing and sends them to Elasticsearch for indexing.

2. [Logstash Forwarder](https://github.com/harrytruman/logstash-vmware/blob/master/logstash-forwarder.conf): Central forwarder; environment tagging of messages and forwarding to Redis.

3. [Logstash Shipper](https://github.com/harrytruman/logstash-vmware/blob/master/logstash-vcenter.conf): Ships messages from Windows to the Logstash forwarder.

## Filter Examples

This message:
````
<166>2013-12-27T16:12:57.896Z hostname.com Vpxa: [507F9B90 verbose 'vpxavpxaInvtHost' opID=WFU-e579383e] [HostChanged] Found update for tracked MoRef vim.HostSystem:ha-host\n
````

Parsed by this filter:
````
"message", "<%{POSINT:syslog_pri}>%{TIMESTAMP_ISO8601:@timestamp} %{SYSLOGHOST:hostname} %{SYSLOGPROG:message_program}: (?<message-body>(?<message_system_info>(?:\[%{DATA:message_thread_id} %{DATA:syslog_level} \'%{DATA:message_service}\'\ ?%{DATA:message_opID}])) \[%{DATA:message_service_info}]\ (?<message-syslog>(%{GREEDYDATA})))",
````

Index and displayed like this (formatted for readability):
````json
{
  "_index": 				"logstash-2014.03.21",
  "_type": 					"logs",
  "_id": 					"LeKbd5UrRuaK6lTSmWStDw",
  "_score": 				null,
  "_source": 				{
    "@timestamp": 				"2014-03-21T12:15:03.221-07:00",
    "tags": 					[ "esx" ],
    "syslog_pri": 				"166",
    "message_program": 			"Vpxa",
    "message-body": 			"[7D6B0B90 verbose 'hostdvm' opID=WFU-87a0f82b] [VpxaHalVmHostagent] 3: GuestInfo changed 'guest.disk'",
    "message_system_info": 		"[7D6B0B90 verbose 'hostdvm' opID=WFU-87a0f82b]",
    "message_thread_id": 		"7D6B0B90",
    "syslog_level": 			"verbose",
    "message_service": 			"hostdvm",
    "message_opID": 			"opID=WFU-87a0f82b",
    "message_service_info": 	"VpxaHalVmHostagent",
    "message-syslog": 			"3: GuestInfo changed 'guest.disk'",
    "syslog_severity_code": 	6,
    "syslog_facility_code": 	20,
    "syslog_facility": 			"local4",
    "syslog_severity": 			"informational",
    "syslog_source-IP": 		"<ip_address>",
    "syslog_source-hostname": 	"<source_fqdn>",
    "message-raw": 				"<166>2014-03-21T19:15:03.206Z <source_fqdn> Vpxa: [7D6B0B90 verbose 'hostdvm' opID=WFU-87a0f82b] [VpxaHalVmHostagent] 3: GuestInfo changed 'guest.disk'\n"
  							},
  "sort": 					[ 1395429303221 ]
}
````
Note: I need to update this screenshot. But it's very similar:

<a href="http://imgur.com/2dA4WGI"><img src="http://i.imgur.com/2dA4WGI.png" title="Screenshot from Kibana" /></a>

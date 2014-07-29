Failed Login Alerting
===============

This is a very simple example of configuring email alerts (or whatever else you want) for failed login attempts. Source messages are from ESXi and vCenter, respectively:

	<164>2014-06-24T18:00:36.225Z host.domain.com Hostd: Rejected password for user alfred from 172.23.221.4
	
	2014-07-18T05:06:06.445-07:00 [04916 error 'authvpxdUser' opID=23bd4053] Failed to authenticate user <alfred>

Grok filters parse and tag messages, a throttle filter sets a threshold to occur after 3 messages within 5 minutes. and email outputs get triggered if all those conditions are met.

````puppet
filter {
  if "Rejected" in [message] {
    grok {
      match => [ "message", "<%{POSINT:syslog_pri}>%{TIMESTAMP_ISO8601:@timestamp} %{SYSLOGHOST:hostname} %{SYSLOGPROG:message_program}: (?<message-body>(?<user-fail>Rejected password for user (?<user-name>(%{USERNAME})) from (?<user-ip>(%{GREEDYDATA}))))" ]
      add_tag => [ "metric" ]
    }
  }
  if "Failed to authenticate user" in [message] {
    grok {
      match => [
        "message", "%{TIMESTAMP_ISO8601:@timestamp} (?<message-body>(?<message_system_info>(?:\[%{DATA:message_thread_id} %{DATA:syslog_level} \'%{DATA:message_service}\'\ ?%{DATA:message_opID}])) (?<message-syslog>(?<user-fail>Failed to authenticate user \<(?<user-domain>(%{GREEDYDATA}))\\(?<user-name>(%{USERNAME}))\>)))",
        "message", "%{TIMESTAMP_ISO8601:@timestamp} (?<message-body>(?<message_system_info>(?:\[%{DATA:message_thread_id} %{DATA:syslog_level} \'%{DATA:message_service}\'\ ?%{DATA:message_opID}])) (?<message-syslog>(?<user-fail>Failed to authenticate user \<(?<user-name>(%{USERNAME}))\>)))"
      ]
      add_tag => [ "metric" ]
    }
  }
  if "metric" in [tags] {
    throttle {
      before_count => 3
      after_count => 3
      period => 300
      key => "%{user-name}"
      add_tag => "throttled"
    }
  }
}

output {
  if "metric" in [tags] and "throttled" not in [tags] {
    email {
      from => "logstash@domain.com"
      to => "whoever@domain.com"
      subject => "Failed Login"
      body => "User: %{user-name}\nHost: %{host}\n%{message}"
    }
  }
}
````

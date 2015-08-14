.. _logstash:

Logstash
==============

The logstash event processing pipeline has three stages: input -> filters -> output.

* input  genenrate events

* filters modify events

* output ship events elsewhere

Inputs and outputs support codecs that enable you to encode or decode the data as it enters or exits the pipeline without having to use a separate filter.

To configure logstash, you create a config that specifies which plugins you want to use and settings for each plugin.

Configuration
-------------------

::

  # {{ansible_managed}}
  # 
  
  # input section
  input {
    
  {% if logstash_whoami == "shipper" %}
    # settings for plugin: file as a shipper
    #============================================
    file {
      path => {{default_syslog}}
  	type => "syslog"
    }
    file {
      path => {{default_authlog}}
  	type => "authlog"
    }
    #=============================================
  {% elif logstash_whoami == "indexer"  %}
    # settings for plugin: redis as an indexer
    #============================================
    redis {
      data_type => "pattern_channel"
  	key => "{{redis_key}}"
  	host => "{{read_from_redis_addr}}"
  	port => "{{read_from_redis_port}}"
    }
    #=============================================
  {% endif %}
  }
  
  # filter section
  filter {
  {% if logstash_whoami == "shipper" %}
    # apply filter rules on shipper end
    #=============================================
    if [type] == "syslog" or [type] == "authlog" {
      grok {
  	  match => { "message" => "%{SYSLOGTIMESTAMP:syslog_timestamp} %{SYSLOGHOST:syslog_hostname} %{DATA:syslog_program}(?:\[%{POSINT:syslog_pid}\])?: %{GREEDYDATA:syslog_message}" }
  	  add_field => [ "received_at", "%{@timestamp}" ]
  	  add_field => [ "received_from", "%{host}" ]
  	}
  	syslog_pri {}
  	date {
  	  match => [ "syslog_timestamp", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
  	}
    }
    #=============================================
  {% endif %}
  }
  
  # output section
  output {
  {% if logstash_whoami == "shipper" %}
    # settings for plugin: redis as a shipper
    #============================================
    redis {
      data_type => "channel"
  	key =>  "{{redis_key}}"
  	host => "{{write_to_redis_addr}}"
  	port => {{write_to_redis_port}}
    }
    #============================================
  {% elif logstash_whoami == "indexer" %}
    # settings for plugin: elasticsearch as an indexer
    #============================================
    elasticsearch {
      host => "{{els_addr}}"
  	protocol => "http"
    }
    #============================================
  {% endif %}
  
  }

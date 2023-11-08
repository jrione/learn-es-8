## introducing logstash

• logstash parses, transforms, and filters data as it passes through.
• it can derive structure from unstructured data
• it can anonymize personal data or exclude it entirely
• it can do geo-location lookups
• it can scale across many nodes
• it guarantees at-least-once delivery
• it absorbs throughput from load spikes
See https://www.elastic.co/guide/en/logstash/current/filter-plugins.html for the huge list of filter plugins.

### logstash with log file
sample pipeline:
```
input {
	file {
		path => "/home/student/access_log“
		start_position => "beginning"
	}
}
filter {
	grok {
		match => { "message" => "%{COMBINEDAPACHELOG}" }
	}
	date {
		match => [ "timestamp", "dd/MMM/yyyy:HH:mm:ss Z" ]
	}
}
output {
	elasticsearch {
		hosts => ["89.233.105.243:9200"]
		user => "logstash_internal"
        password => "changeme"
	}
	stdout {
		codec => rubydebug
	}
}
```

### logstash with Mysql

smaple pipeline:
```
input {
	jdbc {
		jdbc_connection_string => "jdbc:mysql://89.233.105.243:3306/bo"
		jdbc_user => "eci2"
		jdbc_password => "eci2.4321"
		jdbc_driver_library => "/sample/mysql-connector-java-8.0.15/mysql-connector-java-8.0.15.jar"
		jdbc_driver_class => "com.mysql.jdbc.Driver"
		statement => "SELECT * FROM audit_logs"
	}
}
output {
	elasticsearch {
		id => "bo_audit_logs"
      	index => "bo_audit_logs"
		hosts => ["elasticsearch:9200"]
		user => "logstash_internal"
        password => "changeme"
	}
	stdout {
		codec => rubydebug
	}
}
```

### logstash with CSV

sample pipeline:
```
input {
  file {
    path => "/sample/csv-schema-short-numerical.csv"
    start_position => "beginning"
    sincedb_path => "/dev/null"
  }
}
filter {
  csv {
      separator => ","
      skip_header => "true"
      columns => ["id","timestamp","paymentType","name","gender","ip_address","purpose","country","age"]
  }
}
output {
   elasticsearch {
     hosts => ["elasticsearch:9200"]
	user => "logstash_internal"
    password => "changeme"
     index => "demo-csv"
  }

stdout {}

}
```

### logstash with Kafka

sample pipeline:
```
input {
	kafka {
		bootstrap_servers => ["ws.absys.ninja:9092"]
		topics => ["kafka-logs"]
	}
}
filter {
	json {
		source => "message"
	}
}
output {
	elasticsearch {
		id => "kafka_logs"
      	index => "kafka_logs"
		hosts => ["elasticsearch:9200"]
		user => "logstash_internal"
        password => "changeme"
	}
	stdout {
		codec => rubydebug
	}
}
```
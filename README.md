# elastic-efk-apm

# elastic

```
helm upgrade --install elastic bitnami/elasticsearch -f values.yaml

kibanaEnabled: true

heapSize: 512m (coordination-only, data, master, ingest)

ingest:
 enabled: true
  
last line
kibana:
  elasticsearch:
    hosts:
      - '{{ include "elasticsearch.coordinating.fullname" . }}'
    port: 9200  
```

# fluentd

```
k apply -f elasticsearch-output.yaml

k apply -f nginx-log-parse.yaml

helm upgrade --install fluentd bitnami/fluentd -f values.yaml
```

# fluentd values;

```
configMap: nginx-log-parser
  
    extraEnv:
    - name: ELASTICSEARCH_HOST
      value: elastic-coordinating-only 
    - name: ELASTICSEARCH_PORT
      value: "9200"
 ```

# fluentd nginx log parse;

```
<source> 
      @type tail
      path /var/log/containers/*nginx*.log
      exclude_path /var/log/containers/*ngress*
      pos_file /opt/bitnami/fluentd/logs/buffers/fluentd-nginx.pos 
      tag nginx_access_logs
      format none
      read_from_head true
      add_remote_addr true 
    </source>

    <filter nginx_access_logs>
      @type parser
      format json
      key_name message
    </filter>


    <filter nginx_access_logs>
      @type parser
      format /^(?<remote>[^ ]*) (?<host>[^ ]*) (?<user>[^ ]*) \[(?<time_format>[^\]]*)\] "(?<method>\S+)(?: +(?<path>[^\"]*?)(?: +\S*)?)?" (?<code>[^ ]*) (?<size>[^ ]*)(?: "(?<referer>[^\"]*)" "(?<agent>[^\"]*)"(?:\s+(?<http_x_forwarded_for>[^ ]+))?)?(.*)$/
      time_format %d/%b/%Y:%H:%M:%S %z
      key_name log
    </filter>
```

# apm-server not

```
helm install apm elastic/apm-server --version 7.15.0
k edit cm apm-apm-server-config

    output.elasticsearch:
      hosts: ["http://elastic-coordinating-only.efk-apm.svc.cluster.local:9200"]
```

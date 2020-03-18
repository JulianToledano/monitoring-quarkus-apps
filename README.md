# Monitoring $ Logs

Monitoring of apps and logs

`docker run -d -v /:/rootfs:ro -v /var/run:/var/run:ro -v /sys:/sys:ro -v /var/lib/docker/:/var/lib/docker:ro -v /dev/disk/:/dev/disk:ro -p 8888:8080 --name cadvisor  google/cadvisor`

`docker run -d -p 9090:9090 -v ~/Documents/training/monitoring/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus`

`docker run -d --name elasticsearch --net monitoring -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:7.6.1`

`docker run -d --name kibana --net monitoring -p 5601:5601 docker.elastic.co/kibana/kibana:7.6.1`

`docker run -d --name logstash --net monitoring -p 5000:5000 -p 9600:9600 -p 12201:12201/udp -v ~/Documents/training/monitoring/logstash/pipelines:/usr/share/logstash/pipeline docker.elastic.co/logstash/logstash:7.6.1`

rate(container_cpu_usage_seconds_total{name="person-postgres"}[1m])
container_memory_usage_bytes{name="person-postgres"}
rate(container_network_transmit_bytes_total[1m])
rate(container_network_receive_bytes_total[1m])
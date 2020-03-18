# Monitoring & logs

Hello!! :wave::wave:

This repo aims to create a system to centralize all logs and metrics from microprocesses running quarkus apps.

For that purpose the following stack of software is needed:
 * Prometheus :fire: (for monitoring)
 * cAdvisor  :owl: (for monitoring container and system metrics)
 * elastickseach
 * logstash
 * kibana

 # Docker-compose :whale:

 Runing the docker compose of this repo should get all the above services up and runing.
  * `docker-compose up`

**Note**: check the configuration and change it if some ports are already taken.

# Prometheus :fire:

[Prometheus](https://prometheus.io/) as said in their docs:
>What is Prometheus?
>
>Prometheus is an open-source systems monitoring and alerting toolkit originally built at SoundCloud. Since its inception in 2012, many companies and organizations have adopted Prometheus, and the project has a very active developer and user community. It is now a standalone open source project and maintained independently of any company. To emphasize this, and to clarify the project's governance structure, Prometheus joined the Cloud Native Computing Foundation in 2016 as the second hosted project, after Kubernetes.

Once the docker-compose is up you can check your local prometheus from [here](localhost:9090).

Remember to add any target you want to check at [`prometheus.yml`](prometheus/prometheus.yml)

### Examples of use
 Here are some commands to check your containers:
 * `rate(container_cpu_usage_seconds_total{name="--name"}[1m])`
 * `container_memory_usage_bytes{name="--name"}`
 * `rate(container_network_transmit_bytes_total[1m])`
 * `rate(container_network_receive_bytes_total[1m])`

# cAdvisor :owl:
[cAdvisor](https://github.com/google/cadvisor) is a tool for monitoring resources and performance of containers. As they say in ther repo:

>cAdvisor (Container Advisor) provides container users an understanding of the resource usage and performance characteristics of their running containers. It is a running daemon that collects, aggregates, processes, and exports information about running containers. Specifically, for each container it keeps resource isolation parameters, historical resource usage, histograms of complete historical resource usage and network statistics. This data is exported by container and machine-wide.
>
>cAdvisor has native support for Docker containers and should support just about any other container type out of the box. We strive for support across the board so feel free to open an issue if that is not the case. cAdvisor's container abstraction is based on lmctfy's so containers are inherently nested hierarchically.

Once the docker-compose is up you can check your local cAdvisor from [here](localhost:8888).

# ELK

[ELK](https://www.elastic.co/what-is/elk-stack) is the acronym of ElasticSearch, Logstash and Kibana. The core of elastic.
## ElasticSearch
[ElasticSearch](https://www.elastic.co/) is a search and analytics engine.
## Logstash
[Logstash](https://www.elastic.co/logstash) is a data collection engine that can collect data from different [sources](https://www.elastic.co/guide/en/logstash/current/input-plugins.html) and output them to your choosen [stash](https://www.elastic.co/guide/en/logstash/current/input-plugins.html).

In this case we'll use just the [gelf plugin](https://www.elastic.co/guide/en/logstash/current/plugins-inputs-gelf.html) as the input and [elasticsearch server](https://www.elastic.co/guide/en/logstash/current/plugins-outputs-elasticsearch.html) as output. You can check this configuration in the [gelf.com](logstash/pipelines/gelf.com)
## Kibana
[Kibana](https://www.elastic.co/kibana) is a frontend application that sits on top of the Elastic Stack, providing search and data visualization capabilities for data indexed in Elasticsearch. 
Monitoring of apps and logs.

You can check your kibana dashboard [here](localhost:5601).

**Note**: kibana takes from 1 to 5 minute to get up, so don't get nervous if it seems that doesn't work at the beggining.

# Set up your quarkus apps

In order to be able to use this stack your `quarkus` apps must fulfill two requisites:
 * Publis some metrics at `/metrics` so Prometheus can acces them.
 * Forward the logs to logstash with `gelf`

To do so, add the following plugins to your pom:
```xml
    <dependency>
      <groupId>io.quarkus</groupId>
      <artifactId>quarkus-smallrye-metrics</artifactId>
    </dependency>
    <dependency>
    <groupId>io.quarkus</groupId>
      <artifactId>quarkus-logging-gelf</artifactId>
      <version>1.1.0.Final</version>
    </dependency>
```
## quarkus-smallrye-metrics
[This plugin](https://quarkus.io/guides/microprofile-metrics) use the [eclipse microprofile](https://wiki.eclipse.org/MicroProfile) to gather various metrics and statistics that provide insights into what is happening inside the application.

All the metrics you take with it will be published at `/metrics` where prometheus can reach them.

The following code snipet using microprofile checks the time of processing a request and count how many times that endpoint is called.
```java
package ...

import org.eclipse.microprofile.metrics.MetricUnits;
import org.eclipse.microprofile.metrics.annotation.Counted;
import org.eclipse.microprofile.metrics.annotation.Timed;

...{
    @GET
    @Counted(name = "restCountRetrieve", description = "How many times a person had been retrieve.")
    @Timed(name = "restTimeRetrieve", description = "A measure of how long it takes to retrieve a person.", unit = MetricUnits.MILLISECONDS)
    @Produces(MediaType.TEXT_PLAIN)
    @Path("/retrieve/{name}")
    public Person getPerson(@PathParam String name) {
        return personService.getByName(name);
    }
...}
```

## quarkus-logging-gelf
[Gelf](https://docs.graylog.org/en/3.2/pages/gelf.html) (Graylog Extended Log Format) is used because integrates well with `logstash` and is not neccesary to truncate message as with `syslog`. Check [this quarkus guide](https://quarkus.io/guides/centralized-log-management) if you want to read more about the quarkus integration with gelf.

The following code snipets show you how configure it and use with quarkus.

### application.properties
```xml
quarkus.log.handler.gelf.enabled=true
quarkus.log.handler.gelf.host=logstash
quarkus.log.handler.gelf.port=12201
```
**Note**: change host to `localhost` if not running in a container.
### Java code
```java
package ...

import org.jboss.logging.Logger;

...{
    
    private static final Logger LOG = Logger.getLogger(ConsumerPersonResource.class);

    ...{
        LOG.info("Connecting to final endpoint to insert a person...");
    ...
    }
...
}
```

With the configuration in the `application.properties` all logs are forwarded to logstash.


# Further work

1. Connect logstash to elk so all metrics can be accessed in the same place.
    * [Metricbeat](https://www.elastic.co/beats/metricbeat)
    * [Analyze your Prometheus metrics at scale](https://www.elastic.co/what-is/prometheus-monitoring)
2. Create alerts in Prometheus
    * [Alerting Rules](https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/)
3. Custom metrics with microprofile
4. Research about [this bug](https://github.com/JulianToledano/monitoring-quarkus-apps/issues/1)
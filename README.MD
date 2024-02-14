# Docker Grafana STACK
## How to use it with your SpringBoot ?
1. Configure `spring.application.name` inside `application.yml / application.properties` for search purposes in grafana.
2. Add `logback-spring.xml` to `/resources` folder - (remember to modify loki url according to your config)
    ```<?xml version="1.0" encoding="UTF-8"?>
    <configuration>
        <include resource="org/springframework/boot/logging/logback/base.xml"/>
        <springProperty scope="context" name="appName" source="spring.application.name"/>
    
        <appender name="LOKI" class="com.github.loki4j.logback.Loki4jAppender">
            <http>
                <url>http://localhost:3100/loki/api/v1/push</url>
            </http>
            <format>
                <label>
                    <pattern>application=${appName},host=${HOSTNAME},level=%level</pattern>
                </label>
                <message>
                    <pattern>${FILE_LOG_PATTERN}</pattern>
                </message>
                <sortByTime>true</sortByTime>
            </format>
        </appender>
    
        <root level="INFO">
            <appender-ref ref="LOKI"/>
        </root>
    </configuration>
    ```
3. Add dependencies to `pom.xml` 
- without TEMPO 
    ```<dependency>
		<dependency>
			<groupId>io.micrometer</groupId>
			<artifactId>micrometer-registry-prometheus</artifactId>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>com.github.loki4j</groupId>
			<artifactId>loki-logback-appender</artifactId>
			<version>1.3.2</version>
		</dependency>
	```
- with Tempo tracing (multi node enviroments)
    ```<dependency>
			<groupId>io.micrometer</groupId>
			<artifactId>micrometer-tracing-bridge-brave</artifactId>
		</dependency>
		<dependency>
			<groupId>io.micrometer</groupId>
			<artifactId>micrometer-registry-prometheus</artifactId>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>com.github.loki4j</groupId>
			<artifactId>loki-logback-appender</artifactId>
			<version>1.3.2</version>
		</dependency>
		<dependency>
			<groupId>io.zipkin.reporter2</groupId>
			<artifactId>zipkin-reporter-brave</artifactId>
		</dependency>
		<dependency>
			<groupId>net.ttddyy.observation</groupId>
			<artifactId>datasource-micrometer-spring-boot</artifactId>
			<version>1.0.1</version>
		</dependency>
    ```
4. Use `docker-compose.yml` form `/docker` folder - `docker compose up -d`

## Grafana UI
Open Grafana dashboard `http://localhost:3000/explore` and choose datasource

### Logs - use Loki 
set label filters `application` = yours SpringBoot application name from `spring.application.name`  


### Metrics - use Prometheus
choose metric, add filters, set interval 


## Advanced configuration
### Basic components configuration 
If you want to change basic settings like:
- ports
- volumes
- config files

you should check `docker-compose.yml` and `.env` files
### Grafana
Visualize logs and charts form metrics for your application

#### Configuration
Default Grafana configuration contains settings for LOKI, PROMETHEUS and TEMPO.
If you want change or add more data sources for your grafana check `/grafana/datasources.yml` 
Remember to update configurations after changing services ports in `docker-compose.yml`
#### Dashboard configurations
You can add custom dashboard configuration for your grafana by placing `dashboard.json` in `/grafana` directory.

### Prometheus
Gather metrics from you application

#### Configuration
Default Prometheus configuration placed in `/prometheus/prometheus.yml` contains one basic service - application.
If you want to monitor more than one service you should add it in `scrape_configs` section in yaml file:

    scrape_configs:
    ...
      - job_name: 'service-2'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['host.docker.internal:8081'] ## only for demo purposes don't use host.docker.internal in production```


### Tempo
Add tracking data for logs and metrics from your application

#### Configuration
For additional tempo configuration check `/tempo/tempo.yml`

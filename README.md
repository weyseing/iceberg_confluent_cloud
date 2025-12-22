# Setup
- **Remove interceptor class**  
    - Do not set Confluent interceptor classes—these are not in the community image.
    - For `ksqldb-server`, set these in `docker-compose.yml`:
```yaml
KSQL_PRODUCER_INTERCEPTOR_CLASSES: ""
KSQL_CONSUMER_INTERCEPTOR_CLASSES: ""
```

- **Comment out to use default partition 12** due to Confluent-image bugs
```yaml
# CONTROL_CENTER_INTERNAL_TOPICS_PARTITIONS: 1
# CONTROL_CENTER_MONITORING_INTERCEPTOR_TOPIC_PARTITIONS: 1
```

- **Start all services with**
```sh
docker-compose up -d

# ----- OR ----- 

docker compose up -d broker
# Wait for broker to be healthy
docker compose up -d schema-registry
# Wait for schema-registry to be healthy
docker compose up -d ksqldb-server
# Wait for ksqldb-server to be healthy
docker compose up -d rest-proxy
# Wait for rest-proxy to be healthy
docker compose up -d ksqldb-cli
# Wait for ksqldb-cli to be healthy
docker compose up -d prometheus
# Wait for prometheus to be healthy
docker compose up -d alertmanager
# Wait for alertmanager to be healthy
docker compose up -d control-center
```

# Expose Kafka to Public Network
- **Turn on Ngrok tunnel**
```bash
ngrok tcp 9094 
```

- **Update broker configuration in `docker-compose.yml`**
    - Must update **Ngrok endpoint**
```properties
# 1. Identity: Maps custom tag to protocol
KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: 'CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT,NGROK:PLAINTEXT'
# 2. Port: Port Kafka opens to listen for traffic
KAFKA_LISTENERS: 'PLAINTEXT://broker:29092,CONTROLLER://broker:29093,PLAINTEXT_HOST://0.0.0.0:9092,NGROK://0.0.0.0:9094'
# 3. Address: The URL/Port Kafka tells clients to use
KAFKA_ADVERTISED_LISTENERS: 'PLAINTEXT://broker:29092,PLAINTEXT_HOST://localhost:9092,NGROK://0.tcp.ap.ngrok.io:10517'
```

- **Recreate docker to reflect configuration**
```bash
docker compose up -d --force-recreate
```

- **Test via Kcat**
    - Must update **Ngrok endpoint**
    - Might need to use **own hotstpot wifi**
```shell
kcat -b 0.tcp.ap.ngrok.io:10517 -L
```
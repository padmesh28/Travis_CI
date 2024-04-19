

helm upgrade <release-name> bitnami/postgresql --set postgresqlConfiguration.wal_level=logical,postgresqlConfiguration.max_replication_slots=4,postgresqlConfiguration.max_wal_senders=4

bash
Copy code
psql -U <username> -d <database_name>
Then, run the SQL commands:
sql
Copy code
CREATE USER debezium REPLICATION LOGIN CONNECTION LIMIT 2 ENCRYPTED PASSWORD 'debezium_pass';
GRANT SELECT ON ALL TABLES IN SCHEMA public TO debezium;

CREATE USER debezium WITH REPLICATION LOGIN ENCRYPTED PASSWORD 'your_password_here';
GRANT SELECT ON ALL TABLES IN SCHEMA public TO debezium;

kubectl exec -it <postgres-pod-name> -- psql -U debezium -d postgres
kubectl get secret <secret-name> -o jsonpath="{.data.postgresql-password}" | base64 --decode
kubectl get secret <secret-name> -o jsonpath="{.data.postgresql-password}" | base64 --decode


FROM debezium/connect:latest
USER root
RUN confluent-hub install --no-prompt debezium/debezium-connector-postgresql:latest
USER kafka



kafkaConnect:
  enabled: true
  image:
    repository: debezium/connect
    tag: latest
  externalAccess:
    enabled: true
  configuration:
    bootstrap.servers: "KAFKA_BOOTSTRAP_SERVERS"  # Replace with your Kafka service URL
    group.id: kafka-connect
    key.converter: org.apache.kafka.connect.json.JsonConverter
    value.converter: org.apache.kafka.connect.json.JsonConverter
    key.converter.schemas.enable: true
    value.converter.schemas.enable: true
    config.storage.replication.factor: 1
    offset.storage.replication.factor: 1
    status.storage.replication.factor: 1

helm upgrade my-kafka bitnami/kafka -f values.yaml

{
  "name": "postgres-connector",
  "config": {
    "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
    "tasks.max": "1",
    "database.hostname": "postgres-host",
    "database.port": "5432",
    "database.user": "postgres",
    "database.password": "password",
    "database.dbname": "yourdb",
    "database.server.name": "dbserver1",
    "schema.include.list": "public",
    "plugin.name": "pgoutput",
    "publication.autocreate.mode": "filtered",
    "snapshot.mode": "initial"
  }
}


curl -X POST -H "Content-Type: application/json" --data @debezium-connector.json http://kafka-connect:8083/connectors
curl http://kafka-connect:8083/connectors/postgres-connector/status


kubectl run kafka-client -n kafka-connect-tutorial --restart='Never' --image bitnami/kafka:latest --requests='cpu=100m,memory=128Mi' --limits='cpu=200m,memory=256Mi' --command -- sleep infinity




kubectl run kafka-client -n kafka-connect-tutorial --restart='Never' --image bitnami/kafka:latest --command -- sleep infinity
kubectl exec -it kafka-client -n kafka-connect-tutorial -- kafka-topics --bootstrap-server dev-sandbox-kafka--service:9092 --topic connect-offsets --create --partitions 1 --replication-factor 1


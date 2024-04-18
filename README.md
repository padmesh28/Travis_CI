

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

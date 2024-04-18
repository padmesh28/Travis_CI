helm upgrade <release-name> bitnami/postgresql --set postgresqlConfiguration.wal_level=logical,postgresqlConfiguration.max_replication_slots=4,postgresqlConfiguration.max_wal_senders=4

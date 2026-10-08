# Apache Kafka on Wodby

What Wodby sets up for this Kafka service. It runs the official `apache/kafka` image, configured entirely through environment variables set by the manifest.

## What is configured

- One node in KRaft mode acting as both broker and controller (`KAFKA_PROCESS_ROLES`). There is no ZooKeeper, and the service is not scalable: the controller quorum is fixed to this single node.
- The cluster ID is generated once by Wodby (token `cluster_id`) and passed as `CLUSTER_ID`.
- Replication factors of the internal offsets and transaction topics are set to `1`. A topic created with a replication factor above 1 cannot be satisfied.
- The broker listens on `9092` without TLS and without authentication (`PLAINTEXT`). Port `9093` is the controller listener, used only inside the container.

## How applications reach it

- Bootstrap server: `<app service name>:9092`, security protocol `PLAINTEXT`. The broker advertises exactly this address (`KAFKA_ADVERTISED_LISTENERS`), so clients must be able to resolve the app service name: this works for services in the same environment.
- The port is private: no public route is created for it. No username, password or certificate exists.
- A service linked to this one receives the host and port in variables named by the linking service.

## Changing configuration

Broker settings are environment variables on this service in the image's `KAFKA_*` form (a broker property in upper case with dots replaced by underscores). Do not mount a `server.properties`. Changing the listener, advertised-listener, node ID or quorum variables breaks the single-node setup.

## Data

Log segments and KRaft metadata are in `/var/lib/kafka/data/logs` (`KAFKA_LOG_DIRS`) on the `data` volume, which is mounted at `/var/lib/kafka/data`. The manifest declares no backups, imports or actions.

## Check the result

From this service's container:

- `/opt/kafka/bin/kafka-topics.sh --bootstrap-server localhost:9092 --list`
- `/opt/kafka/bin/kafka-broker-api-versions.sh --bootstrap-server localhost:9092` answers when the broker is up.

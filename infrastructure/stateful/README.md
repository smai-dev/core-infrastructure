# Stateful Platform Foundation

The platform stateful foundation currently runs on the dedicated data-ops
host outside Kubernetes.

This keeps persistent state independent from K3s recovery.

## Services

### PostgreSQL

Purpose:

- relational application state;
- workflow metadata;
- execution metadata;
- future tenant and platform metadata.

Persistent data:

    /srv/data/postgresql

Service:

    postgresql

Port:

    5432

Internal DNS:

    postgres.core.home.arpa

Remote connections must use TLS and SCRAM authentication.

Permanent application roles and databases are created only when the
corresponding application exists.

### NATS

Purpose:

- publish/subscribe;
- request/reply;
- event distribution;
- durable event streams through JetStream.

Persistent state:

    /srv/data/nats

Configuration:

    /etc/nats/nats-server.conf

Secrets:

    /etc/nats/nats-server.env

Service:

    nats-server

Client port:

    4222

Monitoring:

    127.0.0.1:8222

Internal DNS:

    nats.core.home.arpa

NATS authentication secrets must never be stored in Git.

## Kubernetes integration

The existence of a stateful service does not automatically grant Kubernetes
workloads access to it.

Platform namespaces retain default-deny NetworkPolicy.

When a workload needs PostgreSQL or NATS, it must receive:

1. an explicit workload identity;
2. workload-specific credentials;
3. an explicit NetworkPolicy egress rule;
4. an explicit service/database/subject permission model.

## Backup

Stateful backups are generated on data-ops under:

    /srv/data/backups/stateful

They contain:

- PostgreSQL logical dump;
- PostgreSQL configuration;
- NATS configuration;
- JetStream persistent state.

The newest copies are transferred off-host to control-edge under:

    /var/backups/core-data-ops/stateful

Retention is currently three backup generations.

## Object storage

S3-compatible object storage is intentionally deferred until a real workload
requires it.

The choice of implementation should be evaluated separately from the
relational and messaging foundations.

## Future evolution

The current single-host stateful layer prioritises simplicity and recovery.

Later growth may introduce:

- PostgreSQL replication;
- NATS clustering;
- S3-compatible object storage;
- additional data-ops nodes;
- cloud-managed equivalents;
- automated host provisioning.

Those changes should extend, rather than replace, the current service
contracts.

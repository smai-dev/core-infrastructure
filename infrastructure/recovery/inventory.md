# Recovery Inventory

## control-edge

Role:

- K3s control plane
- Flux administration
- WireGuard endpoint

Local backup:

    /var/backups/core-control-edge/

Off-host backup:

    data-ops:/srv/data/backups/control-edge/

## data-ops

Role:

- Forgejo
- internal DNS
- Caddy
- persistent shared data services

Local backup:

    /srv/data/backups/data-ops/

Off-host backup:

    control-edge:/var/backups/core-data-ops/

## factory

Role:

- K3s worker
- factory workloads

Persistent filesystem:

    /srv/factory-data

Persistent application-data backup policy will be added when stateful
factory services are introduced.

## trust

Role:

- K3s worker reserved for trust-sensitive workloads

Persistent filesystem:

    /srv/trust-data

Scheduling invariant:

    dedicated=trust:NoSchedule

Persistent application-data backup policy will be added when stateful
trust services are introduced.

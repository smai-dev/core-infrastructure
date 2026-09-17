# Recovery Baseline

This directory documents the minimum recovery baseline for the core platform.

Backup archives, private keys, K3s tokens, kubeconfigs, WireGuard secrets,
Forgejo credentials, and other operational secrets must never be committed
to this repository.

## Recovery model

The platform recovery dependency is:

    data-ops
        |
        | internal DNS + Forgejo
        v
    core/core-infrastructure
        |
        v
       Flux
        |
        v
       K3s
        |
        +-- control-edge
        +-- factory
        +-- trust

Forgejo remains outside Kubernetes so recovery of the Git source does not
depend on the Kubernetes cluster being operational.

## Protected systems

### control-edge

The recovery archive contains the state required to recover the K3s control
plane and administrative network configuration, including:

- K3s server state
- K3s server token and configuration
- Kubernetes administrative configuration
- WireGuard configuration

Primary local backup location:

    /var/backups/core-control-edge/

Off-host copy:

    data-ops:/srv/data/backups/control-edge/

### data-ops

The recovery archive protects infrastructure services hosted outside
Kubernetes, including:

- Forgejo repositories and application state
- Forgejo configuration
- internal DNS configuration
- Caddy configuration
- Caddy internal PKI state
- SSH configuration relevant to Forgejo access
- relevant systemd service overrides

Primary local backup location:

    /srv/data/backups/data-ops/

Off-host copy:

    control-edge:/var/backups/core-data-ops/

## Recovery order

Recover the platform in the following order:

    1. Recover data-ops
            |
            v
    2. Restore internal DNS
            |
            v
    3. Restore Caddy
            |
            v
    4. Restore Forgejo
            |
            v
    5. Verify core/core-infrastructure is available
            |
            v
    6. Recover control-edge and K3s
            |
            v
    7. Verify Flux can reach Forgejo
            |
            v
    8. Allow Flux to reconcile the cluster
            |
            v
    9. Rejoin factory and trust if required
            |
            v
    10. Restore persistent platform data

## Backup verification

Every recovery archive must have a SHA-256 checksum.

Verify with:

    sha256sum -c <archive>.sha256

The expected result is:

    <archive>: OK

The archive must also be readable:

    tar -tzf <archive> >/dev/null && echo "archive: readable"

## Security

Recovery archives contain sensitive infrastructure state.

They must:

- remain outside Git;
- be readable only by trusted administrators;
- normally use mode 0600;
- have at least one copy stored off the machine being protected.

## GitOps recovery principle

Git stores desired Kubernetes configuration but is not a complete system
backup.

Recovery requires:

    Git desired state
            +
    host and cluster recovery state
            +
    persistent application data

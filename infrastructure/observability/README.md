# Observability Baseline

The current platform uses a lightweight operational observability baseline.

A full metrics, dashboard, logging, and tracing stack is intentionally deferred
until real platform workloads justify the additional resource and operational
cost.

## Current visibility

### Kubernetes node health

    kubectl get nodes -o wide

Healthy nodes should report Ready.

Important node conditions include:

    Ready=True
    MemoryPressure=False
    DiskPressure=False
    PIDPressure=False

### Workload health

    kubectl get pods -A
    kubectl get deployments,statefulsets,daemonsets -A

Inspect unhealthy workloads using:

    kubectl describe pod <pod>
    kubectl logs <pod>

### Resource usage

K3s Metrics Server provides live CPU and memory observations.

    kubectl top nodes
    kubectl top pods -A --sort-by=memory
    kubectl top pods -A --sort-by=cpu
    kubectl top pods -A --containers

These metrics are intended for current operational visibility and are not a
long-term historical metrics store.

### Kubernetes warnings

    kubectl events -A --types=Warning

Warnings must be interpreted in context. Historical or temporary warnings do
not necessarily indicate a current platform failure.

### Flux

    flux check
    flux get sources git -A
    flux get sources oci -A
    flux get kustomizations -A
    flux get helmreleases -A

Git sources, OCI sources, Kustomizations, and HelmReleases should normally
report Ready=True.

### Certificates

    kubectl get clusterissuer
    kubectl get certificate -A

Persistent issuers and certificates should normally report Ready=True.

Certificate status exposes validity and renewal information. The core root CA
is intentionally managed differently because its automatic renewal is
disabled.

### Host services

Kubernetes does not monitor host-level services such as Forgejo, Caddy and
dnsmasq.

Use systemd and journald on the corresponding host:

    systemctl --failed
    systemctl is-active <service>
    journalctl -u <service>

### Persistent storage

Filesystem capacity is currently checked directly on the host using:

    df -h

Important persistent filesystems include:

    /srv/factory-data
    /srv/trust-data
    /srv/data

### Backups

The backup timers should normally report:

    Active: active (waiting)

The backup services themselves are one-shot units and are normally inactive
between scheduled runs.

Local and off-host backup copies should both be checked periodically.

## Troubleshooting order

Use the following order where possible:

    status
      |
      v
    describe
      |
      v
    events
      |
      v
    logs
      |
      v
    host investigation if required

Avoid changing infrastructure while still diagnosing the failure.

## Deferred observability capabilities

The following are intentionally deferred:

- historical metrics storage;
- dashboards;
- centralized log aggregation;
- distributed tracing;
- automated alert routing;
- service-level objectives;
- long-term capacity trends.

These capabilities may later be introduced in the observability namespace
when application workloads and operational requirements justify them.

## Future monitoring architecture

A future expansion may include:

    Prometheus or compatible metrics backend
    Grafana
    centralized log collection
    OpenTelemetry
    alerting

Any monitoring component deployed into the observability namespace must
receive explicit NetworkPolicy and RBAC permissions rather than bypassing the
existing security baseline.

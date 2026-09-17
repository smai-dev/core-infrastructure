# Scheduling

## Trust node

The `trust` worker is reserved for trust-sensitive workloads.

Required node taint:

```text
dedicated=trust:NoSchedule
```

Verify:

```bash
kubectl get node trust -o jsonpath='{.spec.taints}{"\n"}'
```

Restore if missing:

```bash
kubectl taint nodes trust dedicated=trust:NoSchedule --overwrite
```

Only workloads intentionally assigned to the trust node should declare the
matching toleration.

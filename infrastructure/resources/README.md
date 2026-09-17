# Resource Governance

This directory defines the baseline CPU and memory resource conventions for
platform workloads.

## Principles

1. Every long-running workload should explicitly declare realistic CPU and
   memory requests.

2. A namespace LimitRange provides fallback requests when a container omits
   them.

3. The current fallback requests are:

       cpu: 100m
       memory: 128Mi

4. These values are scheduling defaults, not recommended values for every
   workload.

5. Workloads should override the defaults based on their actual requirements.

6. CPU limits are not applied globally. They should be introduced only when
   there is a reason to constrain CPU consumption.

7. Memory limits are not applied globally. Workload-specific memory limits
   should be chosen using observed or expected memory behaviour.

8. ResourceQuota is intentionally deferred until real workload demand,
   namespace ownership, or tenancy requirements justify aggregate limits.

9. GPU and other extended resources should be declared explicitly by the
   workloads that require them.

## Example small service

    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        memory: 512Mi

## Example larger workload

    resources:
      requests:
        cpu: 1
        memory: 2Gi
      limits:
        memory: 4Gi

These examples are illustrative only. Actual values should be selected for
each workload based on its behaviour and operational requirements.

## ResourceQuota

No CPU or memory ResourceQuota is applied at the current platform stage.

Quotas may be introduced later when the platform has measurable workload
demand or multiple tenants or teams sharing cluster capacity.

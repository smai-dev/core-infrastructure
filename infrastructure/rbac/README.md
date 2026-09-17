# RBAC and ServiceAccount Baseline

This directory defines the Kubernetes identity and authorization baseline for
platform workloads.

## Principles

1. The default ServiceAccount must not receive application permissions.

2. Automatic ServiceAccount token mounting is disabled on the default
   ServiceAccount in platform namespaces.

3. Workloads that require Kubernetes API access must use a dedicated
   ServiceAccount.

4. ServiceAccounts should normally be named after the workload they represent.

5. Permissions should be namespace-scoped with Role and RoleBinding whenever
   possible.

6. ClusterRole and ClusterRoleBinding should be introduced only when a workload
   genuinely requires cluster-scoped access.

7. RBAC rules should use explicit resources and verbs rather than wildcards.

8. Access to Secrets, workload creation, ServiceAccount tokens, impersonation,
   admission configuration, and other privilege-escalation-sensitive resources
   must be explicitly justified.

## Workload without Kubernetes API access

A workload that does not need the Kubernetes API should use a dedicated
ServiceAccount with token automounting disabled:

    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: example-service
      namespace: platform
    automountServiceAccountToken: false

The Pod should explicitly reference it:

    spec:
      serviceAccountName: example-service

## Workload requiring Kubernetes API access

If a workload genuinely requires Kubernetes API access:

1. create a dedicated ServiceAccount;
2. define the minimum Role required;
3. bind the Role only to that ServiceAccount;
4. allow token mounting only for that workload;
5. verify permissions with `kubectl auth can-i`.

Example authorization structure:

    ServiceAccount
          |
          v
        Role
          |
          v
    RoleBinding

Avoid granting application permissions to the namespace default
ServiceAccount.

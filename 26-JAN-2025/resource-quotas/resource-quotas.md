# Resource Quotas

- When several users or teams share a cluster with a fixed number of nodes, there is a concern that one team could use more than its fair share of resources.
- Resource quotas are a tool for administrators to address this concern.
- A resource quota, defined by a **_ResourceQuota_** object, provides constraints that limit aggregate resource consumption per namespace.
- It can limit the quantity of objects that can be created in a namespace by type, as well as the total amount of compute resources that may be consumed by resources in that namespace.

## Resource quotas work like this:

- Different teams work in different namespaces. This can be enforced with [RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/).
- The administrator creates one **ResourceQuota** for each namespace.
- Users create resources (pods, services, etc.) in the namespace, and the quota system tracks usage to ensure it does not exceed hard resource limits defined in a **ResourceQuota**.
- If creating or updating a resource violates a quota constraint, the request will fail with HTTP status code <mark>403 FORBIDDEN</mark> with a message explaining the constraint that would have been violated.


# OpenShift Resource Management – Summary

## 1. Resource Requests & Limits (per Pod/Container)

Applied directly in a **Deployment/DeploymentConfig**:

```yaml
resources:
  requests:
    cpu: "200m"       # Guaranteed minimum
    memory: "256Mi"
  limits:
    cpu: "500m"       # Maximum allowed usage
    memory: "512Mi"
```

* **Requests** = reserved resources; scheduler guarantees them.
* **Limits** = hard ceiling; container is throttled (CPU) or OOMKilled (Memory) if exceeded.

---

Example:

```shell
oc set resources deployment httpd \
  --limits=cpu=500m,memory=512Mi \
  --requests=cpu=200m,memory=256Mi
```

## 2. LimitRange (Namespace-level)

* Sets **defaults**, **minimums**, and **maximums** for requests/limits.
* Applied automatically if a Pod/Container does not specify resources.
* Prevents out-of-bounds values.

Example:

```yaml
kind: LimitRange
spec:
  limits:
  - type: Container
    defaultRequest:
      cpu: "100m"
      memory: "128Mi"
    max:
      cpu: "2"
      memory: "2Gi"
```

---

## 3. ResourceQuota (Namespace-level)

* Defines the **total budget** for a namespace.
* Ensures aggregate consumption stays under a limit.
* Can require that every Pod specifies requests/limits.

Example:

```yaml
kind: ResourceQuota
spec:
  hard:
    requests.cpu: "4"
    requests.memory: "8Gi"
    limits.cpu: "8"
    limits.memory: "16Gi"
```

---

## 4. Interaction: Who “wins”?

* **LimitRange** → Validates each Pod/Container individually.

  * Adds defaults and enforces min/max.
* **ResourceQuota** → Validates namespace totals.

  * Rejects new Pods if quota is exceeded.

👉 **Order of evaluation**:

1. Deployment sets `requests/limits` (or LimitRange defaults them).
2. LimitRange validates values are within allowed min/max.
3. ResourceQuota checks namespace-wide totals.
4. Pod is admitted only if both checks pass.

---

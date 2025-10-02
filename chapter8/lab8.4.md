# Lab 8.4

Il seguente comando, attribuisce al service account configmap-reloader-sa i permessi di edit a tutte le risorse presenti nel namespace appsec-api partendo dal namespace "configmap-reloader"
```shell
oc policy add-role-to-user edit \
   system:serviceaccount:configmap-reloader:configmap-reloader-sa \
   --rolebinding-name=reloader-edit \
   -n appsec-api
```   

La stessa operazione può essere eseguita da console creando un RoleBinding a Livello di cluster con questi settaggi:

## Permessi più granulari

Vediamo adesso come ottenere una configurazione più granulare per un **ServiceAccount** in OpenShift che deve:
- Leggere le **ConfigMap** dal namespace `appsec-api`
- Eseguire il **rollout dei Deployment** nel namespace `appsec-api`

Evitando di concedere permessi troppo ampi come con il ruolo `edit`.

---

## 1. ServiceAccount

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: configmap-reloader-sa
  namespace: configmap-reloader
```

---

## 2. Role (Read-only sulle ConfigMap)

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: reloader-read-configmaps
  namespace: appsec-api
rules:
  - apiGroups: [""]
    resources: ["configmaps"]
    verbs: ["get", "list", "watch"]
```

---

## 3. Role (Permessi per rollout dei Deployment)

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: reloader-rollout-deployments
  namespace: appsec-api
rules:
  - apiGroups: ["apps"]
    resources: ["deployments"]
    verbs: ["get", "list", "watch", "patch", "update"]
```

> 🔎 Nota: puoi restringere ulteriormente i permessi limitando a specifici Deployment con `resourceNames`.

---

## 4. RoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: reloader-read-cm
  namespace: appsec-api
subjects:
  - kind: ServiceAccount
    name: configmap-reloader-sa
    namespace: configmap-reloader
roleRef:
  kind: Role
  name: reloader-read-configmaps
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: reloader-rollout-deploy
  namespace: appsec-api
subjects:
  - kind: ServiceAccount
    name: configmap-reloader-sa
    namespace: configmap-reloader
roleRef:
  kind: Role
  name: reloader-rollout-deployments
  apiGroup: rbac.authorization.k8s.io
```

---

## ✅ Benefici

- Il ServiceAccount **non ha più ruolo `edit` completo**.
- Ha solo i permessi **strettamente necessari**:
  - **Read ConfigMap**
  - **Patch/Update Deployment per rollout**

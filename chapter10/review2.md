
# Lab: compreview-apps

```bash
lab start compreview-apps
```

## Setup directory

```bash
mkdir -p ~/DO280/labs/compreview-apps
mkdir -p ~/DO280/labs/compreview-apps/project-cleaner
mkdir -p ~/DO280/labs/compreview-apps/beeper-api
cd ~/DO280/labs/compreview-apps
```

---

## Login e creazione namespace

```bash
oc login -u admin -p redhatocp https://api.ocp4.example.com:6443
oc create namespace workshop-support
oc label namespace workshop-support category=support
oc project workshop-support
```

```bash
oc adm policy add-cluster-role-to-group admin workshop-support
```

---

## Resource Quota

```bash
oc create quota workshop-support \
  --hard=limits.cpu=4,limits.memory=4Gi,requests.cpu=3500m,requests.memory=3Gi
```

---

## LimitRange

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: workshop-support
  namespace: workshop-support
spec:
  limits:
    - default:
        cpu: 300m
        memory: 400Mi
      defaultRequest:
        cpu: 100m
        memory: 250Mi
      type: Container
```

```bash
oc apply -f limitrange.yaml
```

---

# Project Cleaner

## Service Account

```bash
oc create sa project-cleaner-sa
cd ~/DO280/labs/compreview-apps/project-cleaner
```

## Cluster Role

```bash
oc apply -f cluster-role.yaml
```

```bash
oc adm policy add-cluster-role-to-user project-cleaner -z project-cleaner-sa
```

```bash
oc login -u do280-support -p redhat
```

---

## CronJob

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: project-cleaner
  namespace: workshop-support
spec:
  schedule: "*/1 * * * *"
  concurrencyPolicy: Forbid
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: Never
          serviceAccountName: project-cleaner-sa
          containers:
            - name: project-cleaner
              image: registry.ocp4.example.com:8443/redhattraining/do280-project-cleaner:v1.1
              imagePullPolicy: Always
              env:
                - name: PROJECT_TAG
                  value: "workshop"
                - name: EXPIRATION_SECONDS
                  value: "10"
              resources:
                limits:
                  cpu: 100m
                  memory: 200Mi
```

```bash
oc apply -f cron-job.yaml
```

---

## Test creazione progetto

```bash
oc new-project clean-test
oc project workshop-support
oc get jobs,pods
```

---

# Beeper API

```bash
cd ~/DO280/labs/compreview-apps/beeper-api
```

## Deploy Database

```bash
oc apply -f beeper-db.yaml
oc get pod -l app=beeper-db
```

---

## TLS Secret

```bash
oc create secret tls beeper-api-cert \
  --cert certs/beeper-api.pem \
  --key certs/beeper-api.key
```

---

## Deployment

```bash
oc apply -f deployment.yaml
```

---

## Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: beeper-api
  namespace: workshop-support
spec:
  selector:
    app: beeper-api
  ports:
    - port: 443
      targetPort: 8080
      name: https
```

```bash
oc apply -f service.yaml
```

---

## Route

```bash
oc create route passthrough beeper-api-https \
  --service beeper-api \
  --hostname beeper-api.apps.ocp4.example.com
```

---

## Test API

```bash
curl -s --cacert certs/ca.pem \
  https://beeper-api.apps.ocp4.example.com/api/beeps; echo
```

---

## Test connettività DB

```bash
oc debug --to-namespace="workshop-support" -- \
  nc -z -v beeper-db.workshop-support.svc.cluster.local 5432
```

---

## POST Beep

```bash
curl -s --cacert certs/ca.pem -X POST \
  https://beeper-api.apps.ocp4.example.com/api/beep \
  -H 'Content-Type: application/json' \
  -d '{ "username": "user1", "content": "first message" }'
```

---

# Network Policy Database

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: database-policy
  namespace: workshop-support
spec:
  podSelector:
    matchLabels:
      app: beeper-db
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              category: support
          podSelector:
            matchLabels:
              app: beeper-api
      ports:
        - protocol: TCP
          port: 5432
```

```bash
oc apply -f db-networkpolicy.yaml
```

---

## Test API

```bash
curl -s --cacert certs/ca.pem \
  https://beeper-api.apps.ocp4.example.com/api/beeps; echo
```

---

## Test connettività API interna

```bash
oc debug --to-namespace="workshop-support" -- \
  nc -z -v beeper-api.workshop-support.svc.cluster.local 443
```

---

# Network Policy Ingress API

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: beeper-api-ingresspolicy
  namespace: workshop-support
spec:
  podSelector: {}
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              policy-group.network.openshift.io/ingress: ""
      ports:
        - protocol: TCP
          port: 8080
```

```bash
oc apply -f beeper-api-ingresspolicy.yaml
```

---

## Test

```bash
oc debug --to-namespace="workshop-support" -- \
  nc -z -v beeper-api.workshop-support.svc.cluster.local 443

curl -s --cacert certs/ca.pem \
  https://beeper-api.apps.ocp4.example.com/api/beeps; echo
```


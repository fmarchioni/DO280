
# Lab: compreview-package

```bash
lab start compreview-package
```

## Helm Repository

```bash
helm repo list
```

```bash
helm repo add  do280-repo http://helm.ocp4.example.com/charts
```

```bash
helm search repo
```

---

## Login e creazione progetto

```bash
oc login -u developer -p developer  https://api.ocp4.example.com:6443
```

```bash
oc new-project compreview-package
```

---

## Deploy MySQL Chart

```bash
helm install roster-database do280-repo/mysql-persistent
```

```bash
watch oc get pods
```

---

## Navigazione directory Kustomize

```bash
cd DO280/labs/compreview-package/
```

```bash
tree
```

---

## Verifica deployment base

```bash
cat roster/base/roster-deployment.yaml
```

---

## Render overlay production

```bash
oc kustomize roster/overlays/production/
```

---

## Deploy overlay production

```bash
oc apply -k roster/overlays/production/
```

```bash
watch oc get pods
```

---

## Verifica route

```bash
oc get route
```

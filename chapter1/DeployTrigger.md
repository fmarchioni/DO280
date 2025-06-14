# 🛠️ Effetti delle modifiche dirette ed indirette di un Deployment in OpenShift

Questo esempio dimostra che modificare direttamente un `Deployment` causa un **redeploy automatico**, mentre modificare una `ConfigMap` associata **non** lo fa automaticamente, a meno che non si usi un `DeploymentConfig` o un webhook trigger.

---

## ✅ Login al cluster

```bash
oc login -u developer -p developer https://api.ocp4.example.com:6443/
```

---

## 🚀 Creazione di un Deployment

```bash
oc create deployment httpd \
  --image=image-registry.openshift-image-registry.svc:5000/openshift/httpd:latest
```

Crea un Deployment chiamato `httpd` basato sull'immagine HTTPD presente nel registry interno di OpenShift.

---

## 🧱 Creazione di una ConfigMap

```bash
oc create configmap httpd-config --from-literal=NAME="ApacheWebServer"
```

La `ConfigMap` `httpd-config` contiene un semplice valore testuale.

---

## 🔗 Associazione della ConfigMap come variabili d'ambiente

```bash
oc set env deployment/httpd --from=configmap/httpd-config
```

Questo comando collega la `ConfigMap` al Deployment come variabili d’ambiente. Tuttavia, **non crea un trigger implicito** sul contenuto della ConfigMap.

---

## 🔍 Verifica dei trigger associati

```bash
oc set triggers deploy/httpd
```

Visualizza i trigger configurati. Di default, solo il trigger `config` (modifica diretta del Deployment) è attivo.  
**Le modifiche alla ConfigMap non sono trigger automatici.**

---

## 🔄 Modifica diretta del Deployment → **Redeploy automatico**

```bash
oc edit deploy/httpd
```

Modificando ad esempio l’immagine o una variabile nel Deployment, si innesca immediatamente un nuovo rollout (`deployment.apps/httpd restarted`).

---

## ⚠️ Modifica indiretta (es. ConfigMap) → **Nessun redeploy**

```bash
oc edit cm/httpd-config
```

Anche se la ConfigMap è usata nel Deployment, modificarla **non** innesca un redeploy automatico.  
Le modifiche avranno effetto **solo su nuovi pod**, oppure si dovrà forzare un nuovo deploy con:

```bash
oc rollout restart deployment/httpd
```

---

## 📌 Conclusione

| Tipo di modifica             | Innesca redeploy? |
|-----------------------------|-------------------|
| Modifica diretta del Deployment | ✅ Sì              |
| Modifica della ConfigMap         | ❌ No              |

Per ottenere un comportamento di redeploy automatico su risorse come `ConfigMap` o `Secret`, considera l’uso di `DeploymentConfig` con `configChange` trigger, oppure implementa meccanismi di restart gestiti esternamente.

---

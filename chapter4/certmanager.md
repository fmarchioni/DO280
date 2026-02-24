
# 🔐 Demo Cert-Manager in OpenShift

## 1. Introduzione

In questa demo mostriamo come **Cert-Manager Operator** semplifica la gestione dei certificati TLS in OpenShift:

* Automatizza la creazione, il rinnovo e la distribuzione dei certificati.
* Gestisce i certificati per Pod, Route e altri oggetti Kubernetes.
* Riduce il rischio di errori manuali con chiavi e certificati.
* Supporta diversi provider di certificati: self-signed, Let’s Encrypt, HashiCorp Vault, ecc.

**Pre-requisiti: Installa il Cert-Manager Operator (Disponibile nel Lab DO288)**

---

## 2. ClusterIssuer self-signed

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: selfsigned-issuer
spec:
  selfSigned: {}
```

**Spiegazione:**

* Un `ClusterIssuer` è una risorsa cluster-scoped che può emettere certificati in qualsiasi namespace.
* Qui usiamo `selfSigned` per generare certificati internamente al cluster, senza dipendere da CA esterne.
* Questo è utile per ambienti di test o demo, dove non vogliamo usare certificati pubblici.

---

## 3. Certificate per il Pod

Crea il progetto mydemo:


```bash
oc new-project mydemo
```

Crea il Certificate:

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: demo-cert
  namespace: mydemo
spec:
  secretName: demo-tls
  commonName: demo.example.com
  duration: 24h
  renewBefore: 12h
  issuerRef:
    name: selfsigned-issuer
    kind: ClusterIssuer
```

**Spiegazione:**

* La risorsa `Certificate` rappresenta il certificato che vogliamo usare.
* Il `secretName: demo-tls` indica dove Cert-Manager salverà il certificato e la chiave privata.
* `issuerRef` punta al `ClusterIssuer` che firmerà il certificato.
* `duration` e `renewBefore` definiscono il periodo di validità e il momento di rinnovo automatico.

💡 Nota didattica: quando esegui `oc get certificate demo-cert -n mydemo`, vedi `READY=True` solo quando il Secret è stato creato con successo.

---

## 4. Pod che usa il certificato

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: tls-demo-pod
  namespace: mydemo
spec:
  containers:
  - name: httpd
    image: image-registry.openshift-image-registry.svc:5000/openshift/httpd
    command: ["/bin/bash","-c"]
    args: ["ls -l /certs; sleep infinity"]
    volumeMounts:
    - name: tls-certs
      mountPath: "/certs"
      readOnly: true
  volumes:
  - name: tls-certs
    secret:
      secretName: demo-tls
```

**Spiegazione:**

* Montiamo il Secret `demo-tls` nel Pod sotto `/certs`.
* Il container `httpd` può così usare il certificato TLS per comunicazioni sicure.
* Questa configurazione è completamente automatizzata: non dobbiamo copiare manualmente certificati o chiavi nei Pod.

---

## 5. Verifica dello stato del certificato

```bash
oc get certificate demo-cert -n mydemo
NAME        READY   SECRET     AGE
demo-cert   True    demo-tls   13s
```

* `READY=True` indica che Cert-Manager ha generato il certificato e creato il Secret.
* Ora il Pod può montare il Secret e usare TLS senza ulteriori operazioni manuali.

---

## 6. Vantaggi della demo

1. **Automazione completa**: creazione e rinnovo certificati gestiti dall’Operator.
2. **Sicurezza**: le chiavi private non sono mai scritte manualmente, riducendo il rischio di errore.
3. **Portabilità**: lo stesso approccio funziona per qualsiasi Pod o servizio nel namespace.
4. **Visibilità**: con `oc get certificate` e `oc describe certificate` si può monitorare facilmente lo stato dei certificati.

---

## 7. Esempio di flessibilità: certificati Let’s Encrypt

Cert-Manager può anche interfacciarsi con provider pubblici come Let’s Encrypt:

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-issuer
spec:
  acme:
    email: studente@example.com
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt-key
    solvers:
    - http01:
        ingress:
          class: nginx
```



---

Vuoi che faccia anche una **versione completa della demo con due Pod comunicanti via TLS e una Route HTTPS** pronta per la classe? Sarebbe un ottimo modo per mostrare cifratura reale e flessibilità.

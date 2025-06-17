
# Creazione di un certificato client di emergenza per un utente amministratore in OpenShift

Come creare un certificato client per un account amministrativo (`admin-backdoor`) da utilizzare nel caso in cui il provider di identità (IdP) principale non funzioni. L'utente deve far parte del gruppo `backdoor-administrators` e avere il ruolo `cluster-admin`. Il certificato avrà una validità di una settimana.

---

## 1. Creare il gruppo `backdoor-administrators`

```bash
oc adm groups new backdoor-administrators
```

## 2. Assegnare il ruolo `cluster-admin` al gruppo

```bash
oc adm policy add-cluster-role-to-group \
  cluster-admin backdoor-administrators
```

## 3. Creare una directory per contenere il certificato

```bash
mkdir admin-cert
```

## 4. Generare una richiesta di firma del certificato (CSR) con OpenSSL

```bash
openssl req -newkey rsa:4096 -nodes \
  -keyout tls.key -subj "/O=backdoor-administrators/CN=admin-backdoor" \
  -out admin-cert/admin-backdoor-auth.csr
```

## 5. Creare la risorsa YAML per la richiesta di firma (CSR)

```bash
cat << EOF >> admin-cert/admin-backdoor-csr.yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: admin-backdoor-access
spec:
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 604800  # una settimana
  request: $(base64 -w0 admin-cert/admin-backdoor-auth.csr)
  usages:
  - client auth
EOF
```

## 6. Creare la CSR nel cluster OpenShift

```bash
oc create -f admin-cert/admin-backdoor-csr.yaml
```

## 7. Verificare lo stato della CSR

```bash
oc get csr admin-backdoor-access
```

## 8. Approvare la CSR

```bash
oc adm certificate approve admin-backdoor-access
```

## 9. Verificare di nuovo lo stato della CSR

```bash
oc get csr admin-backdoor-access
```

## 10. Estrarre il certificato firmato e salvarlo

```bash
oc get csr admin-backdoor-access \
  -o jsonpath='{.status.certificate}' | base64 -d > admin-cert/admin-backdoor-access.crt
```

## 11. Creare un file `kubeconfig` con le credenziali dell'utente `admin-backdoor`

```bash
oc config set-credentials admin-backdoor \
  --client-certificate admin-cert/admin-backdoor-access.crt --client-key tls.key \
  --embed-certs --kubeconfig admin-cert/admin-backdoor.config
```

## 12. Recuperare il certificato pubblico del server API

```bash
openssl s_client -showcerts -connect api.ocp4.example.com:6443 </dev/null 2>/dev/null | \
openssl x509 -outform PEM > ocp-apiserver-cert.crt
```

## 13. Impostare le informazioni del cluster nel kubeconfig

```bash
oc config set-cluster \
  $(oc config view -o jsonpath='{.clusters[0].name}') \
  --certificate-authority ocp-apiserver-cert.crt --embed-certs=true \
  --server https://api.ocp4.example.com:6443 \
  --kubeconfig admin-cert/admin-backdoor.config
```

## 14. Creare il contesto per l'utente `admin-backdoor`

```bash
oc config set-context admin-backdoor \
  --cluster $(oc config view -o jsonpath='{.clusters[0].name}') \
  --namespace default --user admin-backdoor \
  --kubeconfig admin-cert/admin-backdoor.config
```

## 15. Usare il contesto `admin-backdoor`

```bash
oc config use-context admin-backdoor \
  --kubeconfig admin-cert/admin-backdoor.config
```

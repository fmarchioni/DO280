# Installazione di MetalLB su OpenShift Lab

Questa guida spiega come installare MetalLB nel laboratorio OpenShift del corso, partendo dal lab `non-http-lb`.

---

## Step 1: Disinstallazione

Disinstalla l'operatore MetalLB e i relativi operand:

Puoi eseguirlo da Console Web o in alternativa da CLI con:

```bash
# Disinstalla l'operatore MetalLB (sostituisci con il nome effettivo del subscription/operator)
oc delete subscription metallb-operator -n metallb-system
oc delete csv <nome-del-csv> -n metallb-system

# Rimuovi anche gli eventuali operand ancora presenti
oc delete metallb metallb -n metallb-system
```

---

## Step 2: Installa l'Operator

```bash
# Usa l'interfaccia web o applica un Subscription per l'operatore MetalLB nel namespace metallb-system
# Esempio (modifica il nome dell'operator source se necessario)
oc apply -f - <<EOF
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: metallb-operator
  namespace: metallb-system
spec:
  channel: stable
  name: metallb-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
EOF
```

---

## Step 3: Installa MetalLB Operand

```bash
# Crea l'istanza MetalLB
oc apply -f - <<EOF
apiVersion: metallb.io/v1beta1
kind: MetalLB
metadata:
  name: metallb
  namespace: metallb-system
EOF
```

---

## Step 4: IPAddressPool

```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: gls-metallb-ipaddresspool
  namespace: metallb-system
  labels:
    zone: gls
spec:
  addresses:
    - 192.168.50.20-192.168.50.21
  autoAssign: true
  avoidBuggyIPs: false
```

```bash
oc apply -f ipaddresspool.yaml
```

---

## Step 5: L2Advertisement

```yaml
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: gls-metallb-l2advertisement
  namespace: metallb-system
spec:
  ipAddressPools:
    - gls-metallb-ipaddresspool
```

```bash
oc apply -f l2advertisement.yaml
```

---

## Comandi utili

```bash
# Verifica stato MetalLB
oc get metallb -n metallb-system -o yaml

# Descrivi IPAddressPool
oc describe ipaddresspool gls-metallb-ipaddresspool -n metallb-system

# Trova tutti i LoadBalancer
oc get services -A | grep LoadBalancer
```

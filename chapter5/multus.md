 
# Utilizzo di Multus in un cluster OpenShift multinodo: attenzione alla schedulazione

Quando si utilizza **Multus CNI** in un cluster **OpenShift** per connettere un pod a una rete secondaria tramite una `NetworkAttachmentDefinition` (NAD), bisogna prestare attenzione a **quale nodo** ospita il pod.

## 🧨 Possibili problemi

La configurazione di una `NetworkAttachmentDefinition` spesso fa riferimento a un'interfaccia di rete specifica (es. `eth1`, `ens1f0`, `macvlan0`, ecc.) presente solo su alcuni nodi del cluster. Se un pod viene schedulato su un nodo **che non ha l'interfaccia richiesta**, il risultato sarà:

- Il pod rimane in stato `Init` o `ContainerCreating`
- I log mostrano errori del tipo:
```

Error adding container to network ... no such device

````

## ✅ Soluzioni consigliate

### 1. Limitare i nodi con `nodeSelector` o `affinity`

Assicurarsi che i pod siano programmati **solo sui nodi che hanno la rete corretta**. Ad esempio, se solo alcuni nodi sono configurati con la rete NAD:

```yaml
spec:
template:
  spec:
    nodeSelector:
      network-role/multus-enabled: "true"
````

> Ricordarsi di aggiungere l'etichetta ai nodi corretti:

```bash
oc label node <nome-nodo> network-role/multus-enabled=true
```

### 2. Usare `topology` nella NAD (solo alcune CNI lo supportano)

Alcune CNI avanzate permettono di specificare nodi compatibili direttamente nella definizione NAD. Tuttavia, **non è uno standard universale**.

### 3. Verificare manualmente i nodi

Per controllare se un nodo ha l'interfaccia desiderata, usare:

```bash
oc debug node/<node-name>
chroot /host
ip link show
``` 

> **Nota:** OpenShift non schedula automaticamente i pod in base alla disponibilità delle reti secondarie. È compito dell'utente o dell'amministratore guidare la schedulazione in modo corretto.

## 📎 Risorse utili

* [Documentazione ufficiale Multus CNI](https://github.com/k8snetworkplumbingwg/multus-cni)
* [Guida Red Hat OpenShift Networking](https://docs.openshift.com/container-platform/latest/networking/understanding-networking.html)
 

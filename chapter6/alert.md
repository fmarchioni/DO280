# Creazione di un Alert Rule su OpenShift

Questa guida mostra come creare una *PrometheusRule* su OpenShift che
genera un alert quando l'uso della memoria nei Control Plane supera il
70% per almeno 5 minuti.

------------------------------------------------------------------------

## 📘 PrometheusRule di esempio

``` yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: master-memory-usage-high
  namespace: openshift-monitoring
spec:
  groups:
  - name: master-memory.rules
    rules:
    - alert: MasterMemoryUsageHigh
      expr: (1 - sum(node_memory_MemFree_bytes + node_memory_Buffers_bytes + node_memory_Cached_bytes and on (instance) label_replace(kube_node_role{role="master"}, "instance", "$1", "node", "(.+)")) / sum(node_memory_MemTotal_bytes and on (instance) label_replace(kube_node_role{role="master"}, "instance", "$1", "node", "(.+)"))) * 100 > 70
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "High memory usage on master nodes"
        description: "Master nodes are using more than 70% of available memory for at least 5 minutes."
```

Salvare il file come **rule.yml** e applicarlo con:

``` bash
oc create -f rule.yml
```

------------------------------------------------------------------------

## ➡️ Configurazione di un Receiver (es. Slack)

1.  Accedi alla console web di OpenShift.
2.  Vai su:\
    **Administration → Cluster Settings → Configuration → Alertmanager**
3.  Clicca su **Create Receiver**
4.  Inserisci lo Slack API URL, ad esempio:\
    `https://hooks.slack.com/services/T01CRLKMHT5/B09SRLHSGR1/XXXXXXXXXXXXXXXXXXXXXX`\
    (Puoi trovarlo su https://api.slack.com/apps)
5.  Inserisci il canale Slack.
6.  Aggiungi la routing label definita nella AlertRule, ad esempio:\
    `severity: warning`

------------------------------------------------------------------------

## ✅ Alert configurato!

Ora gli alert con livello di severità *warning* verranno inviati al tuo
canale Slack.

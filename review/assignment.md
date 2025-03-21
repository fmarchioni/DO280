# Task Assignment

## Installazione Helm Charts WildFly

### Obiettivi
- Deploy dell'applicazione Microprofile Config utilizzando Helm Charts
- Customizzazione della configurazione con ConfigMap
- Scaling & LimitsRange dell'applicazione
- Implementazione di Health Checks

---

## Dettaglio dei singoli task

### 1. Creazione del progetto
- Creare un progetto OpenShift chiamato `wildfly-demo`.

### 2. Installazione Helm Charts WildFly
- Aggiungere il repository Helm: https://docs.wildfly.org/wildfly-charts/

### 3. Deploy dell'applicazione Microprofile Config
- Installare l'applicazione dal seguente Helm Chart:
  - [Helm Chart](https://github.com/wildfly/quickstart/blob/main/microprofile-config/charts/helm.yaml)  
  - **Nota**: Utilizzare `raw.githubusercontent.com` per il file YAML.

- Verificare che l’applicazione sia disponibile all'indirizzo:
  - `https://microprofile-config-app-wildfly-demo.apps.ocp4.example.com/config/value`

### 4. Configurazione tramite ConfigMap
- Creare una ConfigMap con la seguente chiave/valore:
  ```bash
  CONFIG_PROP="Hello D280!"
  ```
- Configurare il deployment per utilizzare questa ConfigMap.
- Verificare che l'applicazione restituisca il valore corretto interrogando:
  - `https://microprofile-config-app-wildfly-demo.apps.ocp4.example.com/config/value`

### 5. Impostazione dei Limiti di Risorse
- Creare un LimitRange per i Pods con:
  ```yaml
  default:
    memory: 512Mi
  defaultRequest:
    memory: 256Mi
  ```
- Determinare il numero massimo di Pods scalabili con questo LimitRange.
- Scalare i Pods a 0.

### 6. Comandi Utili
- Visualizzare gli eventi ordinati temporalmente:
  ```bash
  oc get event --sort-by=metadata.creationTimestamp
  ```
- Controllare il consumo delle risorse dei nodi:
  ```bash
  oc adm top nodes
  ```
- Scalare il deployment:
  ```bash
  oc scale deployment microprofile-config-app --replicas X
  ```

### 7. Esposizione del servizio Microprofile Health
- Installare il servizio `microprofile-health` dal seguente Helm Chart:
  - [Helm Chart](https://raw.githubusercontent.com/wildfly/quickstart/refs/heads/main/microprofile-health/charts/helm.yaml)

- L'applicazione espone lo stato di health sulla porta `9990`, ma il servizio è mappato sulla porta `8080`.
- Modificare il servizio per esporre correttamente la porta `9990`.
- Aggiornare o ricreare la rotta.
- Modificare il servizio con il comando:
  ```bash
  oc edit service microprofile-health
  ```
- Interrogare l'endpoint `/health` per verificare lo stato:
  ```bash
  curl -k https://microprofile-health-wildfly-demo.apps.ocp4.example.com/health
  ```

# Differenza tra `ResourceQuota` e `LimitRange`

Di seguito è riportato un diagramma che evidenzia le differenze tra `ResourceQuota` e `LimitRange` in Kubernetes/OpenShift.

```mermaid
graph LR
    A[ResourceQuota] --> B[Limita risorse a livello di namespace]
    A --> C[Definisce limiti globali di utilizzo delle risorse]
    B --> D[Impedisce l'uso eccessivo di risorse nel namespace]
    C --> E[Limita risorse come CPU e memoria per tutti i pod]
    
    F[LimitRange] --> G[Definisce limiti per singoli oggetti (es. Pod, Container)]
    F --> H[Imposta valori di default per le risorse]
    G --> I[Controlla risorse minime e massime per i container]
    H --> J[Imposta risorse richieste e limiti per i container]

    B --> K[Impedisce la creazione di nuovi pod se superano i limiti]
    E --> L[Limita risorse a livello di namespace, impedendo l'allocazione illimitata]
    G --> M[Limita le risorse per ogni pod/container nel namespace]
    J --> N[Controlla che i container non possano usare più risorse del consentito]

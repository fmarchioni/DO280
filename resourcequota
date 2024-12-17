# Differenza tra `Requests` e `Limits` in un `ResourceQuota`

Di seguito è riportato un diagramma che mostra la differenza tra `requests` e `limits` in un `ResourceQuota` in Kubernetes/OpenShift.

```mermaid
graph TD
    A[ResourceQuota] --> B[Requests]
    A --> C[Limits]
    B --> D[Risorse minime garantite]
    B --> E[Utilizzati per il scheduling]
    C --> F[Massimo utilizzo consentito]
    C --> G[Utilizzati per throttling/terminazione]
    D --> H[Garantisce 256Mi per il Pod1]
    F --> I[Limita l'uso a 512Mi per il Pod1]
    E --> J[Il pod non può essere schedulato se il nodo non ha risorse]
    G --> K[Pod rallentato se supera il limite di CPU]
    G --> L[Pod terminato se supera il limite di memoria]

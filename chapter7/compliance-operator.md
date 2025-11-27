# Operators Review -- Compliance Operator

## Avvio del laboratorio

Per espandere il lab:

``` bash
lab start operators-review
```

------------------------------------------------------------------------

## Installare il Compliance Operator

Installa il Compliance Operator dalla Console Web (più rapido).

Oppure da CLI:

``` bash
oc create -f ~/DO280/labs/operators-review/scan-setting-binding.yaml
```

------------------------------------------------------------------------

## Verificare lo stato dello scanning

Attendi che lo scanning sia completato:

``` bash
oc get complianceremediation -n openshift-compliance
```

Controlla quali check sono falliti:

``` bash
oc get compliancecheckresult -n openshift-compliance | grep FAIL
```

------------------------------------------------------------------------

## Abilitare il fix automatico

Per abilitare l'applicazione automatica dei remediation, configura lo
**ScanSetting** con `autofix`:

``` yaml
apiVersion: compliance.openshift.io/v1alpha1
profiles:
  - apiGroup: compliance.openshift.io/v1alpha1
    name: rhcos4-moderate
    kind: Profile
settingsRef:
  apiGroup: compliance.openshift.io/v1alpha1
  name: default   # <-- Imposta lo ScanSetting con autofix
  kind: ScanSetting
kind: ScanSettingBinding
metadata:
  name: nist-moderate
  namespace: openshift-compliance
```

------------------------------------------------------------------------

## Applicare manualmente un remediation

Per applicare un remediation specifico:

``` bash
oc patch complianceremediation rhcos4-moderate-master-audit-rules-dac-modification-chmod   -n openshift-compliance   --type merge   -p '{"spec":{"apply": true}}'
```

⚠️ **Attenzione:** questa operazione potrebbe richiedere il riavvio del
nodo.

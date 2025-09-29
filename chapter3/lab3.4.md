# Una giornata nel cluster OpenShift

## Mail al Cluster Admin

**Oggetto:** Richiesta creazione progetto, assegnazione privilegi e gruppi applicativi  

Ciao,  

per policy aziendale, abbiamo deciso di **disabilitare la creazione dei progetti da parte dei teams applicativi**. Da adesso in poi ti verrà richiesta la creazione delle singole progettualità.  
Segue la prima richiesta di intervento che ti apriamo:  

1. **Creazione del progetto**  
   - Creare un nuovo progetto con nome `auth-rbac`.  

2. **Assegnazione privilegi**  
   - Garantire all’utente `leader` i privilegi di amministratore del progetto appena creato (`auth-rbac`).  

3. **Creazione gruppi applicativi**  
   - Creare un gruppo chiamato `dev-group` e aggiungere al suo interno l’utente `developer`.  
   - Creare un secondo gruppo chiamato `qa-group` e aggiungere al suo interno l’utente `qa-engineer`.  

In sintesi, l’obiettivo è avere un nuovo progetto chiamato **auth-rbac**, amministrato dall’utente **leader**, con due gruppi applicativi:  
- **dev-group** → include l’utente *developer*  
- **qa-group** → include l’utente *qa-engineer*  

Grazie per il supporto!  

Cordiali saluti,  
Il tuo affezionato Team Leader 

---

## Mail al Project Manager leader

**Oggetto:** Assegnazione privilegi al progetto auth-rbac  

Ciao,  

come utente **leader**, ti chiediamo di assegnare i seguenti privilegi sul progetto **auth-rbac**:  
- Concedere al gruppo **dev-group** i privilegi di **scrittura**.  
- Concedere al gruppo **qa-group** i privilegi di **sola lettura**.  

Grazie per la collaborazione!  

Cordiali saluti,  

Il tuo affezionato Team Leader

---

## Mail allo sviluppatore developer

**Oggetto:** Deployment Apache HTTP Server su auth-rbac  

Ciao,  

come utente **developer**, ti chiediamo di effettuare il **deployment di un server Apache HTTP (versione 2.4)** nel progetto **auth-rbac**, utilizzando l’immagine standard fornita da OpenShift.  

Grazie per il supporto!  

Cordiali saluti,  
Il tuo affezionato Team Leader

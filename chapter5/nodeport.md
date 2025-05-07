Ecco il testo riformattato in **Markdown**, con **commenti esplicativi** passo-passo per spiegare l'uso di `NodePort` in OpenShift:

---

# ⚙️ Configurazione accesso MySQL tramite NodePort

Questo esempio mostra come esporre un'istanza MySQL su OpenShift usando un **Deployment** e un **servizio NodePort**, per permettere l'accesso da fuori al cluster.

---

## 1. Crea il Deployment MySQL

```bash
oc create deployment mysql-server \
  --image=registry.ocp4.example.com:8443/rhel9/mysql-80
```

> Crea un Deployment chiamato `mysql-server` usando l'immagine MySQL personalizzata.

---

## 2. Imposta le variabili d'ambiente necessarie

```bash
oc set env deployment/mysql-server \
  MYSQL_USER=developer \
  MYSQL_PASSWORD=developer \
  MYSQL_DATABASE=sampledb
```

> Configura utente, password e database iniziale per il container MySQL.

---

## 3. Esporre il servizio con NodePort

```bash
oc expose deployment mysql-server --port=3306 --type=NodePort
```

> Questo comando crea un `Service` di tipo **NodePort**, che permette connessioni esterne al cluster verso la porta 3306.

---

## 4. Verifica i servizi esposti

```bash
oc get services
```

Esempio di output:

```
NAME        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
mysql       ClusterIP   172.30.77.232    <none>        3306/TCP         2m58s
mysqlnode   NodePort    172.30.225.121   <none>        3306:30225/TCP   16s
                                                 ↑
                                   # Porta interna: 3306
                                   # Porta NodePort esposta: 30225
```

---

## 5. Ottieni l’IP interno del nodo

```bash
oc get nodes -o wide
```

Output d'esempio:

```
NAME       STATUS   ROLES                         AGE    VERSION           INTERNAL-IP     EXTERNAL-IP   OS-IMAGE     ...
master01   Ready    control-plane,master,worker   470d   v1.27.6+f67aeb3   192.168.50.10   <none>         RHEL 9       ...
                                              ↑
                              # Questo è l’indirizzo IP da usare per il test
```

---

## 6. Connessione al database MySQL da fuori il cluster

```bash
mysql -h 192.168.50.10 -P 30225 -u developer -p
```



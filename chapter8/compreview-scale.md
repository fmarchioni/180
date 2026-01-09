# Soluzione Lab compreview-scale

```bash
# Avvia il laboratorio "compreview-scale"
lab start compreview-scale

# Login come admin per operazioni preliminari richieste dal lab
oc login -u admin -p redhatocp

# Rimuove un deployment residuo nel progetto di load testing
oc delete deploy computeprime -n compreview-scale-load

# Login come developer per eseguire il resto del laboratorio
oc login -u developer -p developer https://api.ocp4.example.com:6443

# Seleziona il progetto compreview-scale
oc project compreview-scale

# Visualizza i pod attivi nel progetto
oc get pods

# Mostra i log del pod del database (quotesdb)
oc logs quotesdb-78f9b9bc87-d6lpp

# Imposta variabili d'ambiente nel deployment del database
oc set env deployment/quotesdb \
  MYSQL_USER=operator1 \
  MYSQL_PASSWORD=redhat123 \
  MYSQL_DATABASE=quotes

# Mostra i dettagli del deployment aggiornato
oc get deployment/quotesdb -o wide

# Aggiorna l'immagine del container MySQL nel deployment
oc set image deployment/quotesdb \
  mysql-80=registry.ocp4.example.com:8443/rhel9/mysql-80:1-237

# Verifica che l'immagine sia stata aggiornata
oc get deployment/quotesdb -o wide

# Controlla lo stato dei pod dopo l'aggiornamento dell'immagine
oc get pods

# Imposta la readiness probe per il database usando mysqladmin ping
oc set probe deployment/quotesdb --readiness -- mysqladmin ping

# Imposta la liveness probe per il database
oc set probe deployment/quotesdb --liveness -- mysqladmin ping

# Imposta richieste e limiti di CPU/memoria per il database
oc set resources deployment/quotesdb \
  --requests cpu=200m,memory=256Mi \
  --limits cpu=500m,memory=1Gi

# Imposta la readiness probe per il frontend (endpoint /status)
oc set probe deployment/frontend --readiness --get-url http://:8000/status

# Imposta la liveness probe per il frontend (endpoint /env)
oc set probe deployment/frontend --liveness --get-url http://:8000/env

# Imposta richieste e limiti di CPU/memoria per il frontend
oc set resources deployment/frontend \
  --requests cpu=200m,memory=256Mi \
  --limits cpu=500m,memory=512Mi

# Scala il frontend a 3 repliche
oc scale deployment/frontend --replicas 3

# Verifica che i pod del frontend siano stati scalati
oc get pods

# Mostra le route disponibili nel progetto
oc get route

# Testa l'applicazione tramite la route pubblica
curl http://frontend-compreview-scale.apps.ocp4.example.com

# Grade del Lab
lab grade compreview-scale

# Finish del Lab
lab finish compreview-scale
```

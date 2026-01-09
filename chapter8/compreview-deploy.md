# Soluzione del lab compreview-deploy

```bash
# Avvia il laboratorio "compreview-deploy"
lab start compreview-deploy

# Effettua il login come utente developer
oc login -u developer -p developer https://api.ocp4.example.com:6443

# Crea un nuovo progetto chiamato "review"
oc new-project review

# Crea un ImageStreamTag chiamato mysql8:1 basato sull'immagine remota
oc create istag mysql8:1 --from-image registry.ocp4.example.com:8443/rhel9/mysql-80:1-228

# Abilita l'image lookup per l'ImageStream mysql8
oc set image-lookup mysql8

# Mostra lo stato dell'image lookup per tutte le immagini
oc set image-lookup

# Crea un secret con i parametri del database
oc create secret generic dbparams \
  --from-literal user=operator1 \
  --from-literal password=redhat123 \
  --from-literal database=quotesdb

# Crea un deployment chiamato quotesdb usando l'immagine mysql8:1, ma senza avviarlo
oc create deployment quotesdb --image mysql8:1 --replicas 0

# Visualizza i dettagli del deployment quotesdb
oc get deployment quotesdb -o wide

# Imposta un trigger di aggiornamento basato sull'immagine mysql8:1
oc set triggers deployment/quotesdb --from-image mysql8:1 --containers mysql8

# Esporta le variabili d'ambiente dal secret nel deployment (prefisso MYSQL_)
oc set env deployment/quotesdb --from secret/dbparams --prefix MYSQL_

# Aggiunge un volume PVC al deployment per la persistenza dei dati MySQL
oc set volumes deployment/quotesdb --add \
  --claim-class lvms-vg1 \
  --claim-size 2Gi \
  --mount-path /var/lib/mysql

# Scala il deployment a 1 replica per avviare il database
oc scale deployment/quotesdb --replicas 1

# Espone il deployment come servizio sulla porta 3306
oc expose deployment quotesdb --port 3306

# Mostra i dettagli del servizio quotesdb
oc describe service quotesdb

# Crea il deployment frontend basato sull'immagine famous-quotes, ma senza avviarlo
oc create deployment frontend \
  --image registry.ocp4.example.com:8443/redhattraining/famous-quotes:2-42 \
  --replicas 0

# Esporta le variabili d'ambiente dal secret nel deployment frontend (prefisso QUOTES_)
oc set env deployment/frontend --from secret/dbparams --prefix QUOTES_

# Imposta la variabile QUOTES_HOSTNAME per puntare al servizio del database
oc set env deployment/frontend QUOTES_HOSTNAME=quotesdb

# Scala il deployment frontend a 1 replica
oc scale deployment/frontend --replicas 1

# Espone il deployment frontend sulla porta 8000
oc expose deployment frontend --port 8000

# Crea una route pubblica per il servizio frontend
oc expose service frontend

# Mostra le route disponibili nel progetto
oc get route

# Testa l'applicazione tramite la route pubblica
curl http://frontend-review.apps.ocp4.example.com

# Valuta il laboratorio
lab grade compreview-deploy

# Termina il laboratorio
lab finish compreview-deploy
```

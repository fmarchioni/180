# Utilizzo dei Change Triggers con gli ImageStream su OpenShift

L'obiettivo è configurare un'applicazione su OpenShift utilizzando gli **ImageStream** e i **Change Triggers**. Dovrete creare una configurazione che consenta di aggiornare automaticamente un deployment quando cambia il riferimento dell'immagine associata all'ImageStream.

## Requisiti

1. **Creazione di un progetto**: Dovete lavorare in un progetto dedicato chiamato `httpd-triggers`.
2. **Gestione degli ImageStream**: Dovete creare un ImageStream che faccia riferimento a due diverse versioni di un'immagine HTTPD, disponibili nei seguenti percorsi:
   - `registry.ocp4.example.com:8443/ubi8/httpd-24:1-209`
   - `registry.ocp4.example.com:8443/ubi8/httpd-24:1-215`
3. **Configurazione del deployment**: Dovete configurare un deployment che utilizzi l'ImageStream e aggiungere un trigger di cambio immagine.
4. **Aggiornamento automatico**: Dimostrare che il deployment si aggiorna automaticamente quando l'ImageStream viene modificato per puntare a una nuova immagine.
5. **Rollback**: Dimostrare la capacità di tornare alla versione precedente dell'immagine.

### 1. Creazione del Progetto
Create un nuovo progetto su OpenShift chiamato `httpd-triggers`. Questo sarà il namespace in cui lavorerete.

Suggerimento:
```
$ oc new-project ....
```

### 2. Creazione dell'ImageStream
Create un ImageStream chiamato `httpd-app` nel progetto `httpd-triggers`. Questo ImageStream sarà utilizzato per gestire le versioni delle immagini HTTPD.

Suggerimento:
```
$ oc create imagestream ....
```

### 3. Configurazione del Tag dell'ImageStream
Aggiungete un tag all'ImageStream per puntare alla prima versione dell'immagine HTTPD (`1-209`). Abilitate inoltre la policy di lookup locale per l'ImageStream.

Suggerimenti:
```
$ oc tag ....   httpd-app:latest
$ oc set image-lookup .....
```

### 4. Creazione del Deployment
Create un deployment chiamato `httpd-server` che utilizzi l'immagine associata all'ImageStream `httpd-app:latest`. Configurate il deployment per avere almeno 1 replica.

Suggerimento:
```
$ oc create deployment .....
```

Verificate lo stato del deployment per assicurarvi che sia stato creato correttamente:
```
$ oc get deployment .....
```

### 5. Configurazione del Trigger di Cambio Immagine
Aggiungete al deployment un trigger che scateni automaticamente un rollout quando cambia il tag `latest` dell'ImageStream `httpd-app`.

Suggerimento:
```
$ oc set triggers deployment/httpd-server ....
```

### 6. Aggiornamento dell'ImageStream e Verifica del Rollout Automatico
Modificate il tag `latest` dell'ImageStream per puntare alla nuova versione dell'immagine HTTPD (`1-215`). Osservate il comportamento del deployment e verificate che venga aggiornato automaticamente.

Suggerimenti:
```
$ oc tag ..... httpd-app:latest
$ oc rollout status deployment/httpd-server
```

Verificate lo stato aggiornato del deployment:
```
$ oc get deployment .....
```

### 7. Rollback alla Versione Precedente
Tornate alla versione precedente dell'immagine HTTPD (`1-209`) modificando nuovamente il tag `latest` dell'ImageStream. Osservate il comportamento del deployment e verificate che venga effettuato automaticamente il rollback.

Suggerimenti:
```
$ oc tag ..... httpd-app:latest
$ oc rollout status deployment/httpd-server
```

### 8. Verifica Finale degli ImageStream e Deployment
Esaminate i dettagli dell'ImageStream per verificare la cronologia delle immagini e assicuratevi che il rollback sia stato completato correttamente.

Suggerimenti:
```
$ oc describe is httpd-app
$ oc get ....
```

.

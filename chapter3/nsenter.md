## Demo di `nsenter`su OpenShift

In questa dimostrazione vediamo come eseguire un comando **non presente all’interno di un container** (`netstat`) sfruttando l’accesso al **node** e lo strumento `nsenter`.
L’idea è entrare nel **network namespace del processo del container**, aggirando i limiti dell’immagine minimal.

---

### 1️⃣ Creazione del Pod di test

Creiamo un pod basato su un’immagine UBI minimale che espone un web server Apache.

```bash
$ oc run web-server --image registry.access.redhat.com/ubi10/httpd-24
```

Accediamo alla shell del pod.

```bash
$ oc rsh web-server
```

---

### 2️⃣ Tentativo di eseguire `netstat` nel container

Proviamo a eseguire direttamente `netstat` nel pod.

```bash
$ oc exec web-server -- netstat
```

Risultato:

```text
executable file `netstat` not found in $PATH: No such file or directory
command terminated with exit code 255
```

👉 L’immagine è minimale e **non contiene `netstat`**.

---

### 3️⃣ Accesso al cluster come amministratore

Per operare a livello di nodo è necessario un utente con privilegi amministrativi.

```bash
$ oc login -u admin -p redhatocp
```

---

### 4️⃣ Accesso al nodo che ospita il Pod

Avviamo una sessione di debug sul nodo (master in questo esempio).

```bash
$ oc debug node/master01
```

Entriamo nel filesystem del nodo.

```bash
# chroot /host
```

---

### 5️⃣ Individuazione del container tramite CRI-O

Elenchiamo i container attivi e identifichiamo quello relativo al pod `web-server`.

```bash
# crictl ps | grep "web-server"
```

Output:

```text
9b78bcbd666fd   registry.access.redhat.com/ubi10/httpd-24@sha256:...   Running   web-server
```

Annotiamo l’**ID del container**:

```text
9b78bcbd666fd
```

---

### 6️⃣ Recupero del PID del processo del container

Ispezioniamo il container per ottenere il PID del processo principale.

```bash
# crictl inspect 9b78bcbd666fd | grep pid
```

Output rilevante:

```json
"pid": 47798
```

Impostiamo il PID come variabile.

```bash
PID=47798
```

---

### 7️⃣ Accesso al network namespace con `nsenter`

Utilizziamo `nsenter` per entrare nel **network namespace** del processo del container ed eseguire `netstat` direttamente dal nodo.

```bash
# nsenter -t $PID -n netstat -tulpn
```

---

### 8️⃣ Output di `netstat`

```text
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State    PID/Program name
tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN   47798/httpd
tcp        0      0 0.0.0.0:8443            0.0.0.0:*               LISTEN   47798/httpd
```

✅ Abbiamo ora visibilità completa sulle porte in ascolto del container, **senza installare nulla nel pod**.


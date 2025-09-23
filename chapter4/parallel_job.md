 
---

### Esempio con `completionMode: Indexed`

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: job-indicizzato
spec:
  completions: 4
  parallelism: 2
  completionMode: Indexed
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: worker
        image: busybox
        command:
          - sh
          - -c
          - |
            echo "Ciao, sono l'indice $JOB_COMPLETION_INDEX"
            # Simula elaborazione di un dataset
            echo "Elaboro file parte-${JOB_COMPLETION_INDEX}.dat"
            sleep 10
```

---

### Cosa succede qui

* `completionMode: Indexed` → ogni Pod avrà una variabile d'ambiente **`JOB_COMPLETION_INDEX`** (0, 1, 2, 3).
* `completions: 4` → il Job eseguirà **4 Pod in totale**, uno per ciascun indice.
* `parallelism: 2` → al massimo **2 Pod alla volta**.
* Dentro al container puoi usare `$JOB_COMPLETION_INDEX` per fare qualcosa di diverso per ogni Pod (ad esempio elaborare un file diverso, processare un range di dati, ecc.).

---

### Risultato

Esecuzione (semplificata):

* Parte **Pod 0** e **Pod 1** (insieme).
* Quando uno dei due termina, parte **Pod 2**.
* Quando ne termina un altro, parte **Pod 3**.
* Quando tutti i 4 Pod terminano con successo → **Job completato** ✅

---

### Quando usarlo

* **Elaborazione di batch di dati** → ogni Pod prende un chunk del dataset.
* **Render distribuito** → ogni Pod fa il rendering di un frame diverso.
* **Testing parallelo** → ogni Pod esegue una suite di test diversa.

---

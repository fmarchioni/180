# Differenze tra `oc debug` e `oc run`

In OpenShift, i comandi `oc debug` e `oc run` sono utili per gestire e testare i pod, ma svolgono ruoli differenti e sono utilizzati in contesti diversi. Vediamo le principali differenze tra i due:

## `oc run`
Il comando `oc run` viene utilizzato per **creare un nuovo pod** basato su una specifica immagine Docker. È utile per eseguire rapidamente un container senza la necessità di definire un file di configurazione complesso come un Deployment o un Pod YAML.

- **Utilizzo principale**: Creare un nuovo pod a partire da un'immagine specificata.
- **Contesto**: Viene eseguito come utente definito nell'immagine (di solito l'utente predefinito del container).
- **Quando usarlo**: Quando desideri eseguire un container temporaneo per test o debug veloce.
- **Comportamento di default**: Viene eseguito con l'utente predefinito definito nel container. Se l'immagine non specifica un utente, sarà l'utente `root` (UID=0).

## `oc debug`
Il comando `oc debug` è utilizzato per avviare un pod di debug **basato su un pod esistente**. Crea un nuovo container in un pod già esistente e ti fornisce un ambiente di debugging che puoi usare per investigare o testare il comportamento del pod.

- **Utilizzo principale**: Fornisce un ambiente di debugging per interagire con un pod già esistente.
- **Contesto**: Usa il contesto di esecuzione dell'utente che eseguiva il pod originale, ma può essere configurato per eseguire come root se necessario.
- **Quando usarlo**: Quando desideri debuggare un pod già in esecuzione che potrebbe non essere più responsive
- **Comportamento di default**: Può usare l'utente del pod originale, ma può essere modificato per eseguire come root se specificato.

## Differenze principali:
- **`oc run`** crea un nuovo pod, mentre **`oc debug`** ti permette di entrare in un pod esistente per fare debugging.
- **`oc run`** è utile per test rapidi, mentre **`oc debug`** è pensato per il troubleshooting e il debug di pod già esistenti.
- Con **`oc debug`** puoi scegliere di eseguire come root, se necessario, mentre con **`oc run`** l'utente è determinato dal container o dall'immagine Docker.

## Script di dimostrazione

### Esecuzione con utente `admin`:
1. **Login come admin**:
   ```bash
   oc login -u admin -p redhatocp
   ```

2. **Creazione di un pod con `oc run`**:
   ```bash
   oc run example --image=image-registry.openshift-image-registry.svc:5000/openshift/httpd:latest
   ```

3. **Verifica dell'utente all'interno del pod**:
   ```bash
   sh-4.4$ id
   uid=1001(default) gid=0(root) groups=0(root)
   ```
Anche se il Pod è stato creato con un cluster-admin, non avrà un id di root
   ```bash
   oc delete pod example
   ```

### Esecuzione con utente `developer`:
1. **Login come developer**:
   ```bash
   oc login -u developer -p developer
   ```
   
2. **Creazione di un pod con `oc run`**:
   ```bash
   oc run example --image=image-registry.openshift-image-registry.svc:5000/openshift/httpd:latest
   ```

3. **Accesso al pod con `oc rsh`**:
   ```bash
   oc rsh example
   sh-4.4$ id
   uid=1000760000(1000760000) gid=0(root) groups=0(root),1000760000
   ```

4. **Debugging del pod con `oc debug`**:
   ```bash
   oc debug pod/example
   sh-4.4$ id
   uid=1000760000(1000760000) gid=0(root) groups=0(root),1000760000
   ```

### Esecuzione con utente `admin` con privilegi root:

1. **Login come admin**:
   ```bash
   oc login -u admin -p redhatocp
   ```

2. **Debugging del pod come root con `oc debug`**:
   ```bash
   oc debug pod/example --as-root
   sh-4.4# id
   uid=0(root) gid=0(root) groups=0(root),1000760000
   ```

### Conclusione:
- Il comando **`oc run`** è utile per creare un nuovo pod in modo rapido, mentre **`oc debug`** è pensato per il troubleshooting di pod già esistenti, permettendo anche l'accesso come root se necessario.
```

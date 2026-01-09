# Demo Horizontal Pod Autoscaler (HPA) su OpenShift

Questo documento raccoglie gli appunti operativi per una **demo live** sull’Horizontal Pod Autoscaler (HPA) in OpenShift, usando un’applicazione basata su **httpd**.

---

## 🎯 Obiettivo della demo

Dimostrare che:

* l’HPA scala i pod in base all’utilizzo della CPU
* l’aumento del traffico HTTP genera uno scale-up
* la riduzione del traffico porta allo scale-down (con comportamento controllato)

---

## 1️⃣ Pre-requisiti

* Deployment chiamato **httpd**
* Service associato (es. **example**)
* Route esposta verso l’esterno

> L’applicazione deve essere raggiungibile via HTTP tramite una Route OpenShift.

---

## 2️⃣ Configurazione delle risorse (requests e limits)

Per facilitare lo scaling durante la demo, impostiamo **requests CPU basse**.

```bash
oc set resources deployment/httpd \
  --containers=httpd \
  --requests=cpu=100m,memory=128Mi \
  --limits=cpu=500m,memory=256Mi
```

📌 Nota didattica:

> L’HPA scala in base alla **percentuale di utilizzo delle CPU requests**, non dei limits.

---

## 3️⃣ Script di generazione del carico

Inserire l’URL della Route nella variabile `ROUTE`:

```bash
export ROUTE=http://example.apps.cluster.example.com
```

Script di carico (eseguibile da shell):

```bash
while true; do
  curl -s $ROUTE > /dev/null
  sleep 0.05
done
```

Salvare lo script (ad esempio `script.sh`) ed eseguirlo:

```bash
bash script.sh
```

---

## 4️⃣ Monitoraggio dello stato dell’HPA

In un altro terminale, osservare il comportamento dell’autoscaler:

```bash
oc get hpa -w
```

Colonna chiave:

```
TARGETS  (es. 80%/50%)
```

* primo valore → utilizzo CPU corrente
* secondo valore → soglia configurata

---

## 5️⃣ Interruzione del carico

Interrompere lo script di carico (`CTRL+C`) e osservare il comportamento di scale-down.

👉 Di default lo scale-down è lento (stabilization window elevata).

---

## 6️⃣ Accelerare lo scale-down (solo per demo)

Per rendere lo **shrink** visibile rapidamente, riduciamo la `stabilizationWindowSeconds` dell’HPA:

```bash
oc patch hpa httpd-demo-hpa --type='merge' -p '
spec:
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 15
      policies:
      - type: Percent
        value: 50
        periodSeconds: 15
'
```

Effetto:

* dopo 15 secondi di carico basso → scale-down
* rimozione fino al 50% dei pod ogni 15 secondi

⚠️ Nota importante:

> Questa configurazione è **solo per demo**. In produzione lo scale-down deve essere più conservativo.

---

## 7️⃣ Comandi utili durante la demo

```bash
oc get pods -w
```

```bash
oc describe hpa httpd-demo-hpa
```

```bash
oc top pods
```

 

Comandi disponibili con `oc` per cancellare risorse in OpenShift, con esempi pratici per ogni scenario.  

---

## **1. Eliminare una risorsa specifica per nome**  
Se conosci il nome della risorsa, puoi eliminarla direttamente.  
```sh
oc delete <tipo-di-risorsa> <nome>
```
**Esempio:**  
```sh
oc delete pod my-pod
oc delete deployment my-deployment
oc delete service my-service
```

---

## **2. Eliminare tutte le risorse di un determinato tipo**  
Se vuoi eliminare tutti i pod, deployment, o altre risorse di un tipo specifico:  
```sh
oc delete <tipo-di-risorsa> --all
```
**Esempio:**  
```sh
oc delete pods --all
oc delete deployments --all
oc delete services --all
```

---

## **3. Eliminare una risorsa usando le etichette (labels)**  
Se una risorsa è stata creata con un'etichetta specifica, puoi filtrarla e cancellarla.  
```sh
oc delete <tipo-di-risorsa> -l <chiave>=<valore>
```
**Esempio:**  
```sh
oc delete pods -l app=my-app
oc delete deployments -l env=staging
oc delete services -l owner=teamA
```

---

## **4. Eliminare risorse multiple contemporaneamente**  
Puoi specificare più risorse in un unico comando.  
```sh
oc delete <tipo1>/<nome1> <tipo2>/<nome2>
```
**Esempio:**  
```sh
oc delete pod/my-pod deployment/my-deployment
```

Oppure eliminare più risorse dello stesso tipo elencandole:  
```sh
oc delete pod my-pod1 my-pod2 my-pod3
```

---

## **5. Eliminare una risorsa in un determinato namespace**  
Se la risorsa è in un namespace specifico, puoi indicarlo esplicitamente.  

```sh
oc delete <tipo-di-risorsa> <nome> -n <namespace>
```
**Esempio:**  
```sh
oc delete pod my-pod -n my-namespace
```

Per eliminare tutte le risorse di un tipo in un namespace:  
```sh
oc delete pods --all -n my-namespace
```

---

## **6. Eliminare usando il Manifest YAML usato per crearle**:
**Esempio: **  
```sh
oc delete -f resource.yaml
```

---

## **7. Forzare la cancellazione di una risorsa**  
Se una risorsa rimane bloccata in stato "Terminating", puoi forzare la cancellazione.  
```sh
oc delete <tipo-di-risorsa> <nome> --grace-period=0 --force
```
**Esempio:**  
```sh
oc delete pod my-stuck-pod --grace-period=0 --force
```

Se il pod rimane ancora in stato *Terminating*, puoi provare a eliminare la finalizer:  
```sh
oc patch pod my-stuck-pod -p '{"metadata":{"finalizers":null}}'
```

---

## **8. Eliminare un progetto intero**  
Se vuoi eliminare tutto in un namespace, puoi cancellare il progetto.  
```sh
oc delete project my-namespace
```

---
 

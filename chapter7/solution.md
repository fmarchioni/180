```
$ oc new-project httpd-triggers
```

```
$ oc create imagestream httpd-app
imagestream.image.openshift.io/httpd-app created
```

```
$ oc tag registry.ocp4.example.com:8443/ubi8/httpd-24:1-209 httpd-app:latest

Tag httpd-app:latest set to registry.ocp4.example.com:8443/ubi8/httpd-24:1-209.
```

```
$ oc set image-lookup httpd-app
imagestream.image.openshift.io/httpd-app image lookup updated
```

```
$ oc get imagestream
NAME        IMAGE REPOSITORY                                                            TAGS     UPDATED
httpd-app   image-registry.openshift-image-registry.svc:5000/httpd-triggers/httpd-app   latest   14 seconds ago
```

```
$ oc create deployment httpd-server --image=httpd-app:latest 
deployment.apps/httpd-server created
```

```
$ oc get deployment httpd-server -o wide
NAME           READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                                                                                                 SELECTOR
httpd-server   1/1     1            1           28s   httpd-app    registry.ocp4.example.com:8443/ubi8/httpd-24@sha256:b1e3c572516d19108428beca8a9da373a8cc7228832b4655eaed4d0c66def876   app=httpd-server
```

```
$ oc set triggers deployment/httpd-server   --from-image=httpd-app:latest --containers=httpd-app
deployment.apps/httpd-server triggers updated
```

```
$ oc tag registry.ocp4.example.com:8443/ubi8/httpd-24:1-215 httpd-app:latest
Tag httpd-app:latest set to registry.ocp4.example.com:8443/ubi8/httpd-24:1-215.
```

```
$ oc get deployment httpd-server -o wide
NAME           READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                                                                                                 SELECTOR
httpd-server   1/1     1            1           80s   httpd-app    registry.ocp4.example.com:8443/ubi8/httpd-24@sha256:91ad55f6ee89d7f59a428c6afe36a74c670d82045cf71d2750dc992b1654fd83   app=httpd-server
```

```
$ oc rollout status deployment/httpd-server 
deployment "httpd-server" successfully rolled out
```

```
$ oc describe is httpd-app
Name:                   httpd-app
Namespace:              httpd-triggers
Created:                3 minutes ago
Labels:                 <none>
Annotations:            openshift.io/image.dockerRepositoryCheck=2025-04-01T17:32:49Z
Image Repository:       image-registry.openshift-image-registry.svc:5000/httpd-triggers/httpd-app
Image Lookup:           local=true
Unique Images:          2
Tags:                   1

latest
  tagged from registry.ocp4.example.com:8443/ubi8/httpd-24:1-215

  * registry.ocp4.example.com:8443/ubi8/httpd-24@sha256:91ad55f6ee89d7f59a428c6afe36a74c670d82045cf71d2750dc992b1654fd83
      2 minutes ago
    registry.ocp4.example.com:8443/ubi8/httpd-24@sha256:b1e3c572516d19108428beca8a9da373a8cc7228832b4655eaed4d0c66def876
      3 minutes ago
```

```
$ oc tag registry.ocp4.example.com:8443/ubi8/httpd-24:1-209 httpd-app:latest
Tag httpd-app:latest set to registry.ocp4.example.com:8443/ubi8/httpd-24:1-209.
```

```
$ oc get deployment httpd-server -o wide
NAME           READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS   IMAGES                                                                                                                 SELECTOR
httpd-server   1/1     1            1           3m46s   httpd-app    registry.ocp4.example.com:8443/ubi8/httpd-24@sha256:b1e3c572516d19108428beca8a9da373a8cc7228832b4655eaed4d0c66def876   app=httpd-server
```

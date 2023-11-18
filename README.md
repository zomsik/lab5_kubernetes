1. Stworzenie namesapce i resource qouta poleceniem:
```sh
kubectl apply -f zad5.yaml
```
W wyniku otrzymamy:
```sh
namespace/zad5 created
resourcequota/zad5qouta created
```

2. Stworzenie poda z punktu 2 zadania
```sh
kubectl create -f worker.yaml --namespace=zad5
```
W wyniku otrzymamy:
```sh
pod/worker created
```

3. Zmodyfikowany plik yaml został umieszczony w pliku polecenie3.yaml. Treba było zmodyfikować ograniczenia na wykorzystywane zasoby oraz dodać namespace, w którym mają zostać utworzone Deployment i Service.

Stworzenie obiektów poleceniem:
```sh
kubectl create -f polecenie3.yaml
```

W wyniku otrzymujemy:
```sh
deployment.apps/php-apache created
service/php-apache created
```
4. Tworzymy autoscalera w pliku polecenie4.yaml

Polecenie:
```sh
kubectl apply -f polecenie4.yaml
```

Otrzymujemy:
```sh
horizontalpodautoscaler.autoscaling/php-apache created
```

maxReplicas ustaliłem na 6 z następującego powodu:
W namespace mamy limity cpu: "2000m" i ramu: 1.5Gi
w deploymencie limity to memory: 250Mi i cpu: 250m

2000m / 250m = 8

1.5Gi / 250Mi = 6

Limitem jest mniejsza wartość (w tym przypadku ram), zatem maksymalnie 6 replik można ustalić, aby działały przy maksymalnym obciążeniu.


Weryfikacja poleceniami:
```sh
$  kubectl get pods -n zad5
NAME                          READY   STATUS    RESTARTS   AGE
php-apache-74dccfb695-vs5h2   1/1     Running   0          109m
worker                        1/1     Running   0          113m
```

```sh
kubectl get hpa -n zad5
NAME         REFERENCE               TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   <unknown>/50%   1         6         1          10m
```

```sh
$ kubectl describe deployments -n zad5
Name:                   php-apache
Namespace:              zad5
CreationTimestamp:      Sat, 18 Nov 2023 12:16:31 +0100
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               run=php-apache
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  run=php-apache
  Containers:
   php-apache:
    Image:      registry.k8s.io/hpa-example
    Port:       80/TCP
    Host Port:  0/TCP
    Limits:
      cpu:     250m
      memory:  250Mi
    Requests:
      cpu:        150m
      memory:     150Mi
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   php-apache-74dccfb695 (1/1 replicas created)
Events:          <none>
```


6. Nieudana próba sprawdzenia obciążenia, najprawdopodbniej związana z działaniem na systemie windows:
```sh
zomsik@DESKTOP-NRQBN2R MINGW64 ~/OneDrive/studia/2 sem mag/kubernetes/lab5
$ kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://10.109.4.111/; done"
kubectl get hpa php-apache --watch
pod "load-generator" deleted
pod default/load-generator terminated (ContainerCannotRun)
failed to create task for container: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: exec: "C:/Program Files/Git/usr/bin/sh": stat C:/Program Files/Git/usr/bin/sh: no such file or directory: unknown
```

```sh
zomsik@DESKTOP-NRQBN2R MINGW64 ~/Downloads/httpd-2.4.58-win64-VS17/Apache24/bin
$ ./ab -n 1000 -c 10 http://10.109.4.111/
This is ApacheBench, Version 2.3 <$Revision: 1903618 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 10.109.4.111 (be patient)
```

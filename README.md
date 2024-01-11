# Horizontal Pod Autoscalar (HPA)

- Horizontal Scaling means increasing and decreasing the number of replicas (Pods)
- This can help our applications scale out to meet increased demand or scale in when resources are not needed.
- When we set a target CPU utilizations percentage, the HPA scales our applications in or out to try to meet that target.
- HPA needs kubernetes ```metric-server``` to verify metric of pods
- We do not need to deploy or install the HPA on our cluster to being scaling our applications, its out of the box available as a default kubernetes API resource.
- Default cooldown period is 5 mins
- Supports for configurable scaling behaviour.

Kubernetes Cluster: Minikube

step: 
1. Start the minikube
2. Deploy metric-server as we are using minikube, will enable that addons
3. clone the repo for manifest files
4. Deploy deployment, svc and hpa from manifest directory
```
minikube start --memory=4098 --driver=hyperkit
minikube ip
minikube addons enable metrics-server
git clone https://github.com/vivekbangare/kubernetes-hpa.git
```
```
k apply -f manifest/deployment.yaml 
deployment.apps/php-apache created

k get pods
NAME                          READY   STATUS    RESTARTS   AGE
php-apache-5b56f9df94-trpsk   1/1     Running   0          55s

k apply -f manifest/service.yaml 
service/php-apache created

k get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP   10m
php-apache   ClusterIP   10.100.75.128   <none>        80/TCP    3s
```

```
k autoscale deployment php-apache --cpu-percent=50 --min=1 --max=5

or 

k apply -f manifest/hpa.yaml


k get hpa
NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   0%/50%    1         5         1          27s
```

Here our condition is : If CPU goes beyond 50% utilization then its should increase it replicas and if CPU is below 50% then decrease the replicas, max replicas count should be 5 and minimum replica count should be 1.

5. Its time for testing our HPA, will try to genrate load on our php app

### Increase the load
#### Run this in a separate terminal
```
kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"
```

# Run this in a separate terminal
```
kubectl get hpa php-apache --watch
NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   0%/50%    1         5         1          102s
php-apache   Deployment/php-apache   193%/50%   1         5         1          3m1s
php-apache   Deployment/php-apache   193%/50%   1         5         4          3m16s
php-apache   Deployment/php-apache   25%/50%    1         5         4          4m1s
php-apache   Deployment/php-apache   0%/50%     1         5         4          5m1s
php-apache   Deployment/php-apache   0%/50%     1         5         4          8m46s
php-apache   Deployment/php-apache   0%/50%     1         5         1          9m1s
```

# Run this in a separate terminal
```
k get deployments.apps  --watch
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
php-apache   4/4     4            4           8m28s
php-apache   4/1     4            4           12m
php-apache   4/1     4            4           12m
php-apache   1/1     1            1           12m
```

from the above output we can see that when we genrates load, pod count(replicas) increases autmatically and when load is decrease the pod count(replicas) also decreasese.
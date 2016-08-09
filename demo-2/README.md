## Create namespace

```
kubectl create namespace demo
```

## Start MySQL

```
kubectl --namespace=demo create -f mysql-deployment.yaml 
kubectl --namespace=demo create -f mysql-service.yaml 

# Test MySQL
kubectl --namespace=demo run mysql-client --tty --stdin --rm --restart=Never --image=mysql:5.7.14 -- /bin/bash
```

## Start Wordpress

```
kubectl --namespace=demo create -f wordpress-deployment.yaml 
kubectl --namespace=demo create -f wordpress-service.yaml 

kubectl --namespace=demo describe service wordpress
```

## What do we have so far?

```
kubectl --namespace=demo get deployment,rs,pod,svc
```

## Scale Wordpress

```
kubectl --namespace=demo scale deployment/wordpress --replicas=3
```

## Upgrade Wordpress

```
# Monitor endpoints
lendpoits=""; while sleep 1; do endpoints=$(kubectl --namespace=demo describe service wordpress | grep Endpoints); test "$endpoints" != "$lendpoits" && { echo "$endpoints"; lendpoits="$endpoints"; }; done 

kubectl --namespace=demo set image deployment/wordpress wordpress=wordpress:4.5.3-apache
```


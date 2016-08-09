## Create Wordpress internal service


```
kubectl --namespace=demo create -f wordpress-service-internal.yaml 
```

## Create Varnish ConfigMap

```
kubectl --namespace=demo create -f varnish-configmap.yaml 
```

## Create Varnish Deployment

```
kubectl --namespace=demo create -f varnish-deployment.yaml 
```

## Update Wordpress Public service

```
kubectl --namespace=demo apply -f wordpress-service.yaml 
```

## Check Header to confirm it worked

```
http HEAD $(kubectl --namespace=demo get svc wordpress -o 'jsonpath={.Status.LoadBalancer.Ingress[0].Hostname}')
```

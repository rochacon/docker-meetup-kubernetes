# Subindo um blog

Para nosso exemplo de aplicação vamos subir um blog Wordpress com um banco MySQL.

## Preparando o ambiente: namespace

Vamos criar um novo namespace para agruparmos nossos serviços.

```
kubectl create namespace demo
```

## Iniciando o MySQL

Para subirmos nosso blog primeiro precisamos do nosso banco de dados.
Usando os manifestos de [`Deployment`](mysql-deployment.yaml) e [`Service`](mysql-service.yaml) vamos subir o banco MySQL para nosso blog.

```
kubectl --namespace=demo create -f mysql-deployment.yaml 
kubectl --namespace=demo create -f mysql-service.yaml 

# Test MySQL
kubectl --namespace=demo run mysql-client --tty --stdin --rm --restart=Never --image=mysql:5.7.14 -- mysql -u root -p -h mysql
```

## Iniciando o Wordpress

Agora que temos nosso banco de dados funcionando, vamos subir nosso blog.
Usando os manifestos de [`Deployment`](wordpress-deployment.yaml) e [`Service`](wordpress-service.yaml) vamos subir uma instância do Wordpress.

```
kubectl --namespace=demo create -f wordpress-deployment.yaml 
kubectl --namespace=demo create -f wordpress-service.yaml 
```

Como usamos um serviço do tipo `LoadBalancer` e nosso cluster está configurado com integração com a AWS, o Kubernetes irá criar um ELB para nossa aplicação ser acessível pelo mundo. Para pegarmos o endereço deste ELB podemos usar o comando `kubectl describe`:

```
kubectl --namespace=demo describe service wordpress
```

## O que já temos rodando em nosso namespace?

Com o seguinte comando iremos listar todos os `Deployments`, `ReplicaSets`, `Pods` e `Services` do nosso namespace.

```
kubectl --namespace=demo get deployment,rs,pod,svc
```

## Escalando nosso blog

Para aguentarmos o alto tráfego no nosso blog iremos aumentar a quantidade de instância do Wordpress que temos rodando.

Para acompanharmos o que o Kubernetes está fazendo vamos monitorar os endpoints (`Pods`) que nosso `Service` do Wordpress está apontando.

```
# Monitor endpoints
lendpoits=""; while sleep 1; do endpoints=$(kubectl --namespace=demo describe service wordpress | grep Endpoints); test "$endpoints" != "$lendpoits" && { echo "$endpoints"; lendpoits="$endpoints"; }; done 
```

Em outro terminal vamos agora aumentar a quantidade de replicas para 3:

```
kubectl --namespace=demo scale deployment/wordpress --replicas=3
```

Nota: O Wordpress por padrão guarda as sessões em disco, aumentando o número de replicas irá causar problemas para nossos usuários. Para o demo não ficar desnecessariamente complicado, vamos ignorar isto por enquanto.


## Upgrade Wordpress

Estamos usando a versão `4.5.2` do Wordpress, mas já existe uma nova versão disponível. Vamos atualizar a versão do nosso blog.
Dica: mantenha o monitor de endpoints rodando para acompanhar a troca dos `Pods` disponíveis no `Service`.

```
kubectl --namespace=demo set image deployment/wordpress wordpress=wordpress:4.5.3-apache
```


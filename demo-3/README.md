# Aguentando mais tráfego: adicionando uma camada de cache no blog.

Nosso blog está fazendo muito sucesso e temos muitos acessos simultâneos, para melhorar a performance do nosso blog vamos adicionar uma camada de cache usando o [Varnish](http://varnish-cache.org).

Como estamos usando o Kubernetes conseguimos fazer esta mudança sem a necessidade de mexer em nosso load balancer público e/ou registros de DNS, incluindo a camada de cache de forma transparente para nosso usuários.

## Primeiro, vamos criar um serviço interno para o Wordpress

Vamos criar um `Service` interno para o Wordpress, este [`Service`](wordpress-service-internal.yaml) será usado como o *backend* do nosso Varnish.

```
kubectl --namespace=demo create -f wordpress-service-internal.yaml 
```

## Criando a configuração (`ConfigMap`) para nosso Varnish

O Varnish usa um arquivo `.vcl` para sua configuração. No Kubernetes podemos usar [`ConfigMap`](varnish-configmap.yaml) para guardarmos esta configuração fora de nossa imagem Docker do Varnish.

```
kubectl --namespace=demo create -f varnish-configmap.yaml 
```

## Subindo o Varnish

Vamos agora subir o Varnish, para isto usaremos este [`Deployment`](varnish-deployment.yaml).

```
kubectl --namespace=demo create -f varnish-deployment.yaml 
```

Ótimo, agora temos nosso varnish rodando, hora de colocar ele como ponto de entrada do nosso `Service`.

## Atualizando o `Service` público do Wordpress

Agora vamos mudar o nosso `Service` público, aquele que criou o ELB no demo anterior, para apontar para os Pods do nosso Varnish. Isto é feito atualizando os labels do seletor do nosso [`Service`](wordpress-service.yaml).

```
kubectl --namespace=demo apply -f wordpress-service.yaml 
```

## Verificando que funcionou

Além de vermos que os endpoints do nosso `Service` público são agora os pods do Varnish, podemos olhar os headers da resposta HTTP do nosso blog. Usando o [HTTPie](https://github.com/jkbrzt/httpie) faremos uma requisição `HEAD` e procuraremos o header `Via` para confirmar que as requisições estão agora sendo servidas através do nosso Varnish.

```
http HEAD $(kubectl --namespace=demo get svc wordpress -o 'jsonpath={.Status.LoadBalancer.Ingress[0].Hostname}')
```

Pronto! Adicionamos nossa camada de cache, sem downtime e transparente para nossos usuários, agora podemos aguentar todo o tráfego que recebemos tranquilamente.

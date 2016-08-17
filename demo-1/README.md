# Kubernetes Hello World

Já temos um cluster funcionando na AWS e configuramos o `kubectl` para acessar este cluster, vamos rodar nossos primeiros comandos no Kubernetes.

## Listando os nós do cluster

```
kubectl get nodes
```

## Hello world

Rodando um pod que irá se limpar no final da execução.

```
kubectl run hello-k8s --stdin --rm --restart=Never --image=hello-world
```


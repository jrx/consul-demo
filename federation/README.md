# Failover

[Configuration Entry Kind: Service Resolver | Consul by HashiCorp](https://www.consul.io/docs/connect/config-entries/service-resolver#failover)

```sh
cd ~/test/consul/src/consul-demo

# proxy default
kubectl apply -f federation/eks-blue/service-default.yaml --context eks-blue
kubectl apply -f federation/eks-red/service-default.yaml --context eks-red

```

> We have a service "foo" in namespace "foo" in datacenter "dc1"

```sh
kubectl create namespace foo --context eks-blue
kubectl apply -f federation/eks-blue/app-backend.yaml --namespace foo --context eks-blue
```

> We have a service "bar" in namespace "bar" in datacenter "dc1" that has service "foo" in namespace "foo" in datacenter "dc1" as an upstream.

```sh
kubectl create namespace bar --context eks-blue
kubectl apply -f federation/eks-blue/app-frontend.yaml --namespace bar --context eks-blue

# intentions
kubectl apply -f federation/eks-blue/service-intentions.yaml --context eks-blue

kubectl port-forward service/bar 9090:9090 --namespace bar --context eks-blue
```

> How can we add a failover to the "foo" service. The service it should fail over to is named "foo" but lives in namespace "foo" and in datacenter "dc2".


```sh
kubectl create namespace foo --context eks-red
kubectl apply -f federation/eks-red/app-backend.yaml --namespace foo --context eks-red
```

```sh
# test failover
kubectl scale --replicas=0 deploy foo --namespace foo --context eks-blue
# or
kubectl delete deploy foo --namespace foo --context eks-blue
```

```sh
# clean-up
kubectl delete -f federation/eks-blue/app-backend.yaml --namespace foo --context eks-blue
kubectl delete -f federation/eks-red/app-backend.yaml --namespace foo --context eks-red
kubectl delete -f federation/eks-blue/app-frontend.yaml --namespace bar --context eks-blue
kubectl delete namespace foo --context eks-blue
kubectl delete namespace foo --context eks-red
kubectl delete namespace bar --context eks-blue
```

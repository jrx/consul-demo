# Failover

[Configuration Entry Kind: Service Resolver | Consul by HashiCorp](https://www.consul.io/docs/connect/config-entries/service-resolver#other-datacenters)

```sh
cd ~/test/consul/src/consul-demo

# proxy default
kubectl apply -f virtualservice/eks-blue/service-default.yaml --context eks-blue

# intentions
kubectl apply -f virtualservice/eks-blue/service-intentions.yaml --context eks-blue
```

## dc 2

```sh
kubectl apply -f virtualservice/eks-red/app-backend.yaml --context eks-red
```

## dc 1

```sh
# backend - bar
kubectl apply -f virtualservice/eks-blue/app-backend.yaml --context eks-blue
# or
kubectl apply -f virtualservice/eks-blue/app-backend-tp.yaml --context eks-blue

# virtual service
kubectl apply -f virtualservice/eks-blue/app-frontend.yaml --context eks-blue
# or 
kubectl apply -f virtualservice/eks-blue/app-frontend-tp.yaml --context eks-blue

# expose frontend
kubectl port-forward service/foo 9090:9090 --context eks-blue
```

```sh
# test 
kubectl get ServiceResolver --context eks-blue
kubectl delete ServiceResolver bar-dc2 --context eks-blue

# clean-up
kubectl delete -f virtualservice/eks-blue/ --context eks-blue
kubectl delete -f virtualservice/eks-red/ --context eks-red
```

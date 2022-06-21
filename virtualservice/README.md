# Failover

[Configuration Entry Kind: Service Resolver | Consul by HashiCorp](https://www.consul.io/docs/connect/config-entries/service-resolver#other-datacenters)

```sh
cd ~/test/consul/src/consul-demo

# proxy default
kubectl apply -f virtualservice/eks-blue/service-default.yaml --context eks-blue

# intentions
kubectl apply -f virtualservice/eks-blue/service-intentions.yaml --context eks-blue
```

```sh
kubectl apply -f virtualservice/eks-blue/app-backend.yaml --context eks-blue
```

```sh
kubectl apply -f virtualservice/eks-blue/app-frontend.yaml --context eks-blue
kubectl port-forward service/bar 9090:9090 --context eks-blue
```

```sh
kubectl apply -f virtualservice/eks-red/app-backend.yaml --context eks-red
```

```sh
# clean-up
kubectl delete -f virtualservice/eks-blue/app-backend.yaml --context eks-blue
kubectl delete -f virtualservice/eks-red/app-backend.yaml --context eks-red
kubectl delete -f virtualservice/eks-blue/app-frontend.yaml --context eks-blue
```
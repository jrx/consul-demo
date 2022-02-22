# Setup Prometheus/Grafana

## Install Kube-Prometheus

```sh
cd ~/test/kubernetes/src
git clone git@github.com:prometheus-operator/kube-prometheus.git
```

```sh
cd ~/test/kubernetes/src/kube-prometheus

# Create the namespace and CRDs, and then wait for them to be available before creating the remaining resources
kubectl create -f manifests/setup
until kubectl get servicemonitors --all-namespaces ; do date; sleep 1; echo ""; done
kubectl create -f manifests/

# Prometheus
kubectl --namespace monitoring port-forward svc/prometheus-k8s 9090

# Grafana
kubectl --namespace monitoring port-forward svc/grafana 3000

# Alert Manager
kubectl --namespace monitoring port-forward svc/alertmanager-main 9093

# clean-up
kubectl delete --ignore-not-found=true -f manifests/ -f manifests/setup
```

## Create Consul Token fro Prometheus

```sh
kubectl port-forward service/consul-server 8501:8501

cd ~/test/consul/src/consul-demo/

export CONSUL_HTTP_TOKEN=$(kubectl get secrets/consul-bootstrap-acl-token --template={{.data.token}} | base64 -d)
export CONSUL_HTTP_ADDR=https://127.0.0.1:8501
export CONSUL_HTTP_SSL_VERIFY=false

cat <<EOF | tee /tmp/agent-read.hcl
agent_prefix "" {
  policy = "read"
}
EOF

consul acl policy create -name=agent-read -rules=@/tmp/agent-read.hcl

export PROM_TOKEN=$(consul acl token create \
  -description "Token for Prometheus" \
  -policy-name agent-read \
  -format=json | jq -r .SecretID)

# Test
curl -k --header "X-Consul-Token: $PROM_TOKEN" \
  https://localhost:8501/v1/agent/metrics?format=prometheus

# Create secret
kubectl create secret generic consul-read-agent-token \
  --from-literal="token=$PROM_TOKEN" \
  --namespace="monitoring"
```

## Configure Prometheus

```sh
kubectl apply -f ./monitoring/
```

## Grafana Dashboards

- **13396** [Consul Server Monitoring dashboard for Grafana | Grafana Labs](https://grafana.com/grafana/dashboards/13396)
- 10642 [Consul Monitoring dashboard for Grafana | Grafana Labs](https://grafana.com/grafana/dashboards/10642)
- 6693 [Envoy Proxy dashboard for Grafana | Grafana Labs](https://grafana.com/grafana/dashboards/6693)
- 11021 [Envoy Clusters dashboard for Grafana | Grafana Labs](https://grafana.com/grafana/dashboards/11021)
- 11022  [Envoy Global dashboard for Grafana | Grafana Labs](https://grafana.com/grafana/dashboards/11022)
- 12904 [Hashicorp Vault dashboard for Grafana | Grafana Labs](https://grafana.com/grafana/dashboards/12904)

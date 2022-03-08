# Terminating Gateway

Configure Terminating Gateways
[Terminating Gateways - Kubernetes | Consul by HashiCorp](https://www.consul.io/docs/k8s/connect/terminating-gateways)

```sh
cd ~/test/consul/src/consul-demo/

kubectl config use-context eks-blue

kubectl port-forward service/consul-server 8501:8501

# set address
export CONSUL_HTTP_ADDR=https://127.0.0.1:8501
export CONSUL_HTTP_SSL_VERIFY=false

# retrieve bootstrap token
export CONSUL_HTTP_TOKEN=$(kubectl get secrets/consul-bootstrap-acl-token --template={{.data.token}} | base64 -d)

# set partition
export CONSUL_PARTITION=default
```

## Register External Service

```sh
cat > /tmp/external.json <<EOF
{
  "Node": "legacy_node",
  "Partition": "$CONSUL_PARTITION",
  "Address": "example.com",
  "NodeMeta": {
    "external-node": "true",
    "external-probe": "true"
  },
  "Service": {
    "ID": "example-https",
    "Service": "example-https",
    "Port": 443
  }
}
EOF
```

```sh
# create
curl --request PUT \
  --header "X-Consul-Token: $CONSUL_HTTP_TOKEN" \
  --data @/tmp/external.json -k $CONSUL_HTTP_ADDR/v1/catalog/register

# list
curl --request GET \
  --header "X-Consul-Token: $CONSUL_HTTP_TOKEN" \
  -k $CONSUL_HTTP_ADDR/v1/catalog/nodes | jq .
```

## Update ACL Token

```json
cat > /tmp/write-policy.hcl <<EOF
service "example-https" {
  policy = "write"
}
EOF
```

```sh
consul acl policy create \
  -name "example-https-write-policy" \
  -rules @/tmp/write-policy.hcl

consul acl token list -format=json | jq -c '.[] | select(.Description | contains("terminating-gateway-terminating-gateway-token Token"))' | jq -r .AccessorID

consul acl token update -id $(!!) \
  -policy-name example-https-write-policy \
  -merge-policies \
  -merge-roles \
  -merge-service-identities
```

## Deploy

```sh
kubectl apply -f ./terminating-gateway/
```

## Test

```sh
kubectl rollout status deploy static-client --watch
kubectl exec deploy/static-client \
  -- curl -vvvs -H "Host: example.com" http://localhost:1234/
```

## Clean-Up

```sh
# delete
cat > /tmp/external.json <<EOF
{
  "Node": "legacy_node"
}
EOF

curl --request PUT \
  --header "X-Consul-Token: $CONSUL_HTTP_TOKEN" \
  --data @/tmp/external.json -k $CONSUL_HTTP_ADDR/v1/catalog/deregister
```

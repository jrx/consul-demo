# canary test

```sh
EXTERNAL_IP=$(kubectl get service/consul-ingress-gateway \
  -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')

echo "Connecting to \"$EXTERNAL_IP\""

while true; do
curl -s \
   -H "Host: webapp.ingress.consul" \
  "http://$EXTERNAL_IP:8080/"
sleep 1
done

curl \
   -H "Host: webapp.ingress.consul" \
   -H "x-version: 3" \
  "http://$EXTERNAL_IP:8080/"
```

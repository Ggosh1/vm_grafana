kubectl apply -f simple-api-svc.yaml
kubectl apply -f scrape-simple-api.yaml
kubectl apply -f scrape-ingress-nginx.yaml

helm upgrade vm-agent victoria-metrics/victoria-metrics-agent \
  --namespace default \
  --reuse-values \
  -f override-values.yaml

kubectl rollout restart deploy/vm-agent-victoria-metrics-agent -n default

kubectl label vmservicescrape scrape-simple-api \
  app.kubernetes.io/instance=vm-agent -n default --overwrite
kubectl label vmservicescrape scrape-ingress-nginx \
  app.kubernetes.io/instance=vm-agent -n ingress-nginx --overwrite

kubectl rollout restart deploy/vm-agent-victoria-metrics-agent -n default

kubectl port-forward svc/vmagent-svc 8429:8429 -n default &
curl -s http://localhost:8429/targets | grep simple-api -A2
curl -s http://localhost:8429/targets | grep ingress-nginx -A2

kubectl port-forward svc/simple-api-svc 8080:80 -n default &
curl http://localhost:8080/actuator/prometheus | head

kubectl port-forward svc/vm-single-victoria-metrics-single-server 8428:8428 -n default &
curl -s "http://localhost:8428/api/v1/query?query=rate(http_server_requests_seconds_count{job=\"simple-api\"}[1m])" | jq .

Data Source: http://localhost:8429
Импорт дашбордов




## Redis
helm repo add bitnami https://charts.bitnami.com/bitnami 
helm repo update










helm uninstall my-redis --namespace redis



helm install my-redis bitnami/redis -f redis-values.yaml --namespace test --create-namespace

helm install redis . -f ../../../redis-values.yaml --namespace rofl
helm upgrade redis . -f ../../../redis-values.yaml --namespace rofl
helm uninstall redis --namespace rofl
helm install my-redis . -f ../../../redis-values.yaml --namespace rofl
helm upgrade redis . -f ../../../redis-values.yaml --namespace rofl
helm upgrade my-redis . -f ../../../redis-values.yaml --namespace rofl
helm uninstall my-redis --namespace rofl
helm install my-redis . -f ../../../redis-values.yaml --namespace rofl



git clone https://github.com/bitnami/charts
cd charts/bitnami/redis
helm dependency update

helm package .

перекинуть по ssh (scp)

helm install my-redis ./redis-X.Y.Z.tgz

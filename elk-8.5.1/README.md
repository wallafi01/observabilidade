k8s-fluentd
Kubernetes logging mechanism using EFK (Elastic search, Kibana and Fluentd)

Prerequisites
Docker setup: https://docs.docker.com/engine/install/

Kubectl installation: https://kubernetes.io/docs/tasks/tool

helm setup - https://helm.sh/docs/intro/install/


Step 1: Create Namespace

kubectl create namespace efk

Step 2: Add Elastic Helm Repository

helm repo add elastic https://helm.elastic.co
helm repo update


Step 3: Install Elasticsearch


8.5.1:

helm install elasticsearch elastic/elasticsearch --version 8.5.1 -n efk --values values-es.yaml

Step 4: Install Elasticsearch with Persistence Disabled

If you want to disable persistence for Elasticsearch (Note: This is suitable for testing purposes only):

helm install elasticsearch elastic/elasticsearch --version 7.17.3 -n efk --set persistence.enabled=false,replicas=1

Step 5: Check the Installation

helm list -n efk-monitoring
kubectl get pods -n efk-monitoring
kubectl get svc -n efk-monitoring



Step 6: Install Kibana


8.5.1

helm install kibana elastic/kibana --version 8.5.1 -n efk --values values-ki.yaml

kubectl get secrets --namespace=efk elasticsearch-master-credentials -ojsonpath='{.data.password}' | base64 -d 


Step 7: Verify Kibana Installation

kubectl get pods -n efk-monitoring
Step 8: Apply Fluentd Configuration

Apply the Fluentd ConfigMap and DaemonSet YAML files:

cd /fluent-bit/
helm upgrade --install fluent-bit . --namespace=efk

kubectl -n efk get all

kubectl -n efk get configmap,svc

kubectl -n efk edit configmap fluent-bit

    edit HHTTP_Passwd

kubectl -n efk rollout restart daemonset.apps/fluent-bit


Step 10: Add Dapr Helm Repository (Optional)

helm repo add dapr https://dapr.github.io/helm-charts/
helm repo update
Step 11: Install Dapr (Optional) Install Dapr with JSON formatted logs enabled:

helm install dapr dapr/dapr --namespace efk-monitoring --set global.logAsJson=true

Step 12: Port Forward Kibana

To access Kibana web interface:

kubectl port-forward svc/kibana-kibana 5601 -n efk-monitoring

Step13: Deploy the sample application

kubectl apply -f k8s-app.yml

kubectl get pods

kubectl get svc

kubectl logs <pod-name>

kubectl port-forward svc/myapp 8080

Delete all the resources
kubectl delete -f k8s-app.yml

kubectl delete -f fluentd-config-map.yaml

kubectl delete -f fluentd-rbac.yaml

helm list -n efk-monitoring

helm uninstall elasticsearch -n efk

helm uninstall kibana -n efk

###3resolvendo elasticsearch erro


https://github.com/elastic/helm-charts/issues/909

https://faun.pub/how-to-forward-kubernetes-logs-to-elasticsearch-elk-using-fluent-bit-and-visualize-it-by-kibana-4b95c881ae26
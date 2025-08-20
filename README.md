# prometheus-grafana-minikube
A test project that shows the working of Monitoring and Alerting using Prometheus and Grafana, about the health and stats of a Flask app deployed on Minikube

## Steps to install Prometheus on MiniKube and configure it to track metrics from your App
1. Install and set up **Prometheus** using **Helm**
* `helm repo add prometheus-community https://prometheus-community.github.io/helm-chart`
* `helm repo update`
* `helm search repo prometheus`
* `helm install prometheus prometheus-community/prometheus --namespace monitoring --create-namespace`
  
2. Access Prometheus dashboard in minikube
* Find the Prometheus service in `kubectl get svc -n monitoring`
* Expsoe Prometheus locally: `kubectl port-forward -n monitoring svc/prometheus-server 9090:80`
* Check `localhost:9090` for Prometheus UI

3. Configure Prometheus to scrape the Flask app
* Get the Flask app ServiceIP: `kubectl get svc` and look for the app name, note its ClusterIP and Port.
* Update the Prometheus ConfigMap and add the Flask app as a scrate target: `kubectl edit configmap prometheus-server -n monitoring`
* Add the `job_name` and `target` under `scrape_config` section.

4. Restarting Prometheus on Minikube
* Get the deployment name from `kubectl get deploy -n monitoring` and look for *prometheus-server*
* Restart the deployment: `kubectl rollout restart deployment prometheus-server -n monitoring`
* Verify the restart: `kubectl get pods -n monitoring`
* After restart, port forward the Prometheus dashboard `kubectl port-forward -n monitoring svc/prometheus-server 9090:90`
* Verify Scraping: seach for the metric in use on the Prometheus Dashboard (*i.e., total_api_requests_total)


## Steps to install Grafana on Minikube and set it up with Prometheus, create dashboards
1. Install Grafana on Minikube Cluster
* `helm repo add grafana https://grafana.github.io/helm-charts`
* `helm repo update`
* `helm install grafana grafana/grafana -n monitoring --create-namespace`
Note: Grafana, just like Prometheus is also installed on *monitoring* namespace.
2. Log into Grafan
* bash: `kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo`

3. Add Prometheus as a Data Source
* Once logged into Grafana, navigate to Connection -> Data Sources -> Add Data Source > Prometheus > (use the cluster IP of Prometheus) -> Save and Test
* Create dashboards.
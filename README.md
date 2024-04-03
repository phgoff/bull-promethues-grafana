**Prerequisites**

- Docker Desktop with Kubernetes.

**Steps**

1. **Deploy Bull Exporter**

   - Apply the yaml configuration to Kubernetes cluster:
     ```bash
     kubectl apply -f bull-monitoring.yaml
     ```

2. **Install Prometheus**

   Helm:

   - **Add the Prometheus charts repository:**

     ```bash
     helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
     ```

   - **Install Prometheus (we'll name our release `my-prometheus`):**
     ```bash
     helm install my-prometheus prometheus-community/prometheus
     ```

3. **Configure Prometheus**

   - **Get Prometheus service address:**

     ```bash
     kubectl get services --namespace=default
     ```

     Look for a service named something like `my-prometheus` and its ClusterIP.

   - **Update `prometheus.yml`:**

     ```bash
     kubectl edit configmap my-prometheus-server --namespace=default
     ```

     Update a config map configuration to tell Prometheus to scrape metrics from the Bull Exporter:

     ```yaml
     scrape_configs:
       - job_name: "bull-exporter"
         static_configs:
           - targets: ["bull-exporter.default.svc.cluster.local:9538"] # Replace with the correct service name and namespace if needed
     ```

4. **Install Grafana**

   - **Add the Grafana charts repository:**

     ```bash
     helm repo add grafana https://grafana.github.io/helm-charts
     ```

   - **Install Grafana (we'll call this release `my-grafana`):**
     ```bash
     helm install my-grafana grafana/grafana
     ```

5. **Access Grafana**

   - **Port-forward Grafana to local machine:**

     ```bash
     kubectl port-forward service/my-grafana 3000:80 --namespace=default
     ```

   - **Access Grafana:** Go to http://localhost:3000.

   - **Get the Admin Password**

     Run the following command to get the 'admin' password:

     ```bash
     kubectl get secret --namespace default my-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
     ```

6. **Import Bull Dashboard**

   - Visit Grafana Dashboards ([https://grafana.com/grafana/dashboards/](https://grafana.com/grafana/dashboards/)) and search for "Bull Queue Prometheus" dashboard.

7. **Configure Data Source**
   - Add a Prometheus data source pointing to Prometheus service address from step 3 (e.g., http://my-prometheus:80).

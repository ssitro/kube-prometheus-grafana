# kube-prometheus-grafana

### Set up Google Cloud K8s cluster
* Install google cloud CLI(gcloud)
* Log in to GCP accoutn

    `gcloud auth login`

* Init your project(be sure to choose the right project and region)

    `gcloud init`

* Get the configuration of you K8s cluster

    `gcloud container clusters get-credentials <cluster-name>`

### Deploy Prometheus and Grafana using Helm
1. Create namespace `monitoring`

    `kubectl create namespace monitoring`

2. Install kube-prometheus-stack using helm chart
    
    ```
    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

    helm repo update

    helm install kps prometheus-community/kube-prometheus-stack -n monitoring

    ```
4. Check if Grafana and Prometheus are working.
    
    Check pods:
    
    `kubectl get pods -n monitoring`

    Check services:

    `kubectl get svc -n monitoring`

5. Check Grafana and Prometheus web interface.

    Get the name of Grafana service (it gonna look like this *\<your helm install name\>-grafana*):

    `kubectl get svc -n monitoring`

    `kubectl port-forward svc/kps-grafana -n monitoring 3000:3000`

    Open Grafana web page : http://127.0.0.1:3000

    User: admin
    
    Password: prom-operator

    You can change password in the settings->Users->admin->edit password:

    ![here](https://imgur.com/rmCmemM.png)

    ![users](https://imgur.com/pIFza0U.png)

    ![pass](https://imgur.com/uXehGnI.png)

    Now check Prometheus web page.
    Get the name of Prometheus service:

    `kubectl get svc -n monitoring`

    `kubectl port-forward svc/kps-kube-prometheus-stack-prometheus -n monitoring 9090` 
    
    (you can port-forward in the new terminal window or you can just terminate previous port-forward with *Ctrl-C*)

    Go to http://127.0.0.1:9090

    It will look like this:

    ![](https://imgur.com/2PKRDG3.png)

### Deploy loadbalancer for the Grafana and BlackBox
    
* Clone the repo
    
    ```
    git clone git@github.com:day4me/kube-prometheus-grafana.git
    ```

* Deploy loadbalancer
    ```
    cd kube-prometheus-grafana
    kubectl apply -f grafana_lb.yml

    ### Get the public IP(external)
    kubectl get service -n monitoring grafana-lb
    ```
    You can now open Grafana Web-UI without portforward 

* Deploy BlackBox Exporter
    
    `kubectl create secret generic additional-scrape-configs --from-file=additional_config.yml --dry-run -oyaml > additional-scrape-configs.yml
    `

    Next, apply the generated kubernetes manifest

    `kubectl apply -f additional-scrape-configs.yml -n monitoring
    `

    Finally, you need to add additional configuration to your Prometheus config:

    ```
    kubectl --namespace=monitoring edit prometheuses kps-kube-prometheus-stack-prometheus
    ```

    You'll see something like this:

    ```
    apiVersion: monitoring.coreos.com/v1
    kind: Prometheus
    metadata:
    name: prometheus
    labels:
        prometheus: prometheus
    spec:
    replicas: 2
    serviceAccountName: prometheus
    serviceMonitorSelector:
        matchLabels:
        team: frontend
    ```

    Add 
    ```
    additionalScrapeConfigs:
      key: additional_config.yml
      name: additional-scrape-configs
    ```

    after `spec:`

    like this:

    ![](https://i.imgur.com/O5ARbT2.png)

    Save it and close it.

    Your new target will appear here(Prometheus): http://127.0.0.1:9090/targets

    For example: 

    BlackBox

    ![](https://imgur.com/iowFBTy.png)

### Add Blackbox exporter dashboard to Grafana

* Press import

    ![](https://i.imgur.com/65lmfFN.png)

* Paste this code: 7587

* Press load, and then choose Prometheus datasource and Import.
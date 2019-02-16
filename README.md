# kubernetes-monitoring

## Deploy Prometheus

#### Install helm client 
Go to: https://docs.helm.sh/using_helm/#installing-helm or \
`$ curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > get_helm.sh`\
`$ chmod 700 get_helm.sh`\
`$ ./get_helm.sh`
#### Apply the tiller RBAC policy 
`$ kubectl apply -f tiller-rbac.yaml`
#### Initilize tiller on the Kubernetes cluster
`$ helm init --service-account tiller`

#### Create a storage class for dynamic persistant volume provisioning
`$ kubectl apply -f k8s-monitoring-sc.yaml`

#### Create a namespace 
`$ kubectl create namespace k8s-monitoring`\
#### Switch position to namespace monitoring
`$ kubectl config set-context $(kubectl config current-context) --namespace=k8s-monitoring`

****Note: If pulling images from a private registry, open prometheus/values.yaml with vi or another editor and update image locations; the default images locations are:

`prom/pushgateway:v0.6.0` \
`prom/alertmanager:v0.15.3` \
`jimmidyson/configmap-reload:tag:v0.2.2` \
`busybox:latest` \
`quay.io/coreos/kube-state-metrics:v1.5.0` \
`prom/node-exporter:v0.17.0` \
`prom/prometheus:v2.7.1` \
`prom/pushgateway:v0.6.0` \

If the registy requires auth, set
`imagePullSecrets:` \
`name: "image-pull-secret" `

#### Install the prometheus helm chart
`helm install --name=prometheus ./prometheus`

#### Identify the prometheus-server EXTERNAL-IP
`$ kubectl get svc |grep prometheus-server`

#### Open a web browser to the Prometheus web UI
http://*PROMETHEUS_SERVICE_EXTERNAL-IP*

Select > **Status** > **Targets**\
Review the status Data Collection services. If not all service immediately report "UP", refresh the page a few times. 

## Deploy Grafana

****Note: Before running the next command, open grafana/values.yaml with vi or another editor and update image location and tag; the default images locations are:

`grafana/grafana:tag: 5.4.3`\
`appropriate/curl:latest`\
`busybox:1.30.0`

#### Install the Grafana Helm chart
`helm install --name=grafana ./grafana`

#### Verify all pods transition to "Running"
`$ kubectl get pods`

#### Collect and record the Admin secret
`$ kubectl get secret --namespace k8s-monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo`\

Record the Secret Ex. NRXNs9WS0gK5oeRTU0NNWyNlc1xoOT49mEiO4aen

#### Identify the grafana service EXTERNAL-IP
`$ kubectl get svc |grep grafana`

#### Open a web browser, navigate to the grafana web UI, and login
http://*GRAFANA_SERVICE_EXTERNAL-IP*

**User Name:** admin

**Pasword:** *SECRET*

#### Configure the prometheus plug-in 
Select > *Add data source*
Select > Prometheus

**Name:** *Prometheus*

**URL:** http://*PROMETHEUS_SERVICE_EXTERNAL-IP*

Select > **Save and Test**

#### Import the Kubernetes Dashboard

Select > **"+"** in left pane then **"Import"**

If the app has Internet Access, enter ID **6417** then click outside of the field; otherwise, upload the .json file located in the repo.

In the Options table, select the *Prometheus* drop-down and select **Prometheus** 
Select > *Import* 

You will redirect to the new dashboard. 

# kubernetes-monitoring

Deploy Prometheus

kubectl create namespace monitoring
kubectl config set-context $(kubectl config current-context) --namespace=monitoring 
kubectl apply -f prometheus.yaml 
kubectl create -f ./prometheus-config-map.yaml

Note: Change Private Registry to Public Image Location: /prom/prometheus/

kubectl create  -f prometheus-deployment.yaml
kubectl get deployments
kubectl create -f prometheus-service.yaml

kubectl get svc |grep prometheus-service
Open web browser to http://<prometheus-service EXTERNAL-IP>:8080
Select > Status > Targets
Verify Data Collection

Deploy Grafana

kubectl apply -f tiller-rbac.yaml 
helm init --service-account tiller

helm install --name grafana-app --namespace monitoring stable/grafana
helm ls

kubectl get secret --namespace monitoring grafana-app -o jsonpath="{.data.admin-password}" | base64 --decode ; echoRecord Password
Record Secret Ex. NRXNs9WS0gK5oeRTU0NNWyNlc1xoOT49mEiO4aen

kubectl create -f grafana-service.yaml
kubectl get svc |grep grafana-service

Open web browser to http:<grafana-service EXTERNAL-IP>:3000
User Name: admin
Pasword: <Output from Above>
   
 
Select > Add data source
 	Name: Prometheus
 	Type: Prometheus
 	URL: http://<prometheus-service EXTERNAL-IP>:8080
Select > Save and Test
Select > "+" in left pane then "Import"

If the app has Internet Access, enter ID "1621" then click outside of the field. If no Internet Access, you can upload .json file.
Select > Import
In the Options table, select the Prometheus drop-down and select Prometheus
Select > Import

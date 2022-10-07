# **Helm Installations**

## **Helm - Supported Kubernetes Versions**
https://helm.sh/docs/topics/version_skew/
```
Helm Version	Supported Kubernetes Versions
3.5.x	        1.20.x - 1.17.x
```

## **Install Helm**
```
wget https://get.helm.sh/helm-v3.5.4-linux-amd64.tar.gz
tar -zxvf helm-v3.5.4-linux-amd64.tar.gz
mv linux-amd64/helm /usr/local/bin/helm
```
---
# **Kubernetes Dashboard**
## **Install Dashboard**
```
helm repo add k8s-dashboard https://kubernetes.github.io/dashboard
helm repo update
```

## **Create Admin Service Account**
```
kubectl create serviceaccount admin
```

## **Get Auth Token**
### Get the secret associated with the Service Account
```
get serviceaccounts admin -o yaml
```
### Get the Bearer Token
```
kubectl describe secret [secret]
```

## **Bind Cluster Admin Role to Service Account**
### Edit cluster-admin Cluster Role Binding
```
kubectl edit clusterrolebinding cluster-admin
```

### Add the Service Account as a Subject
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: system:masters
```
```
- kind: ServiceAccount
  name: admin
  namespace: default
```
---

# **Grafana**
### Create SC & PVs
```
kubectl create -f sc-grafana.yaml
kubectl create -f pv-grafana.yaml
```

### Install Grafana
```
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```
```
helm install grafana grafana/grafana --version 6.33.0 -f ./values-grafana.yaml
```
### Get Grafana Login Credentials
```
username - admin
password - kubectl get secret --namespace default my-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```
--- 

# **Prometheus, Kube State Metrics & Node Exporters**
### **Add Helm Repo**
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```
## **Kube State Metrics**
```
helm install kube-state-metrics prometheus-community/kube-state-metrics -f ./values-kube-state-metrics.yaml -n kube-system
```
## **Node Exporters**
```
helm install node-exporter prometheus-community/prometheus-node-exporter -f values-prometheus-node-exporter.yaml
```
## **Prometheus**
### If any previous Prometheus Installations
```
helm delete prometheus
kubectl delete pv pv-prometheus-alert-manager.yaml
kubectl delete pv pv-prometheus-server.yaml
```
### Create SCs & PVs
```
kubectl create -f sc-prometheus-alert-manager.yaml
kubectl create -f sc-prometheus-server.yaml
kubectl create -f pv-prometheus-alert-manager.yaml
kubectl create -f pv-prometheus-server.yaml
```
### Install
```
helm install prometheus prometheus-community/prometheus -f ./values-prometheus.yaml
```
---

# **ElasticSearch**
## **Generate Certificates & Secrets**
### Get the P12 Certificate File for the 'elasticsearch-master' DNS
```
docker run --name elastic-helm-charts-certs -i -w /app docker.elastic.co/elasticsearch/elasticsearch:7.17.3 \
/bin/sh -c " \
elasticsearch-certutil ca -s --out /app/elastic-stack-ca.p12 --pass '' && \
elasticsearch-certutil cert -s --name elasticsearch-master --dns elasticsearch-master --ca /app/elastic-stack-ca.p12 --pass '' --ca-pass '' --out /app/elastic-certificates.p12"
```
```
docker cp elastic-helm-charts-certs:/app/elastic-certificates.p12 ./
```
```
docker rm -f elastic-helm-charts-certs
```

### Create Certificates & Keys
```
openssl pkcs12 -nodes -passin pass:'' -in elastic-certificates.p12 -out elastic-certificate.pem
```
```
openssl x509 -outform der -in elastic-certificate.pem -out elastic-certificate.crt
```

### Create Secrets
```
kubectl create secret generic elastic-certificates --from-file=elastic-certificates.p12
kubectl create secret generic elastic-certificate-pem --from-file=elastic-certificate.pem
kubectl create secret generic elastic-certificate-crt --from-file=elastic-certificate.crt
```
```
kubectl create secret generic elastic-credentials --from-literal=password='elastic12345' --from-literal=username=elastic
```
## **Install Elasticsearch**
### Add Helm Repo
```
helm repo add elastic https://helm.elastic.co
helm repo update
```

### Create SCs & PVs
```
kubectl create -f sc-elasticsearch.yaml
kubectl create -f pv-elasticsearch.yaml
```

### Install
```
helm install elasticsearch elastic/elasticsearch --version 7.17.3 -f values-elasticsearch.yaml
```
---

# **Kibana**
### Create Secret
```
encryptionkey=$(docker run --rm busybox:1.31.1 /bin/sh -c "< /dev/urandom tr -dc _A-Za-z0-9 | head -c50")
```
```
kubectl create secret generic kibana --from-literal=encryptionkey=$encryptionkey
```

### Install
```
helm install kibana elastic/kibana -f ./values-kibana.yaml --version 7.17.3
```
---

# **Logstash**
### Create SCs & PVs
```
kubectl create -f sc-logstash.yaml
kubectl create -f pv-logstash.yaml
```

### Install
```
helm install kibana elastic/kibana -f ./values-kibana.yaml --version 7.17.3
```
---

# **FileBeat**
### Install
```
helm install filebeat elastic/filebeat -f ./values-filebeat-alert-monitor.yaml --version 7.17.3
```
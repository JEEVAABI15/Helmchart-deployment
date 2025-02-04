# Deploying Jenkins, SonarQube, and Nexus on AKS with ACR Integration

## Prerequisites
Before deploying Jenkins, SonarQube, and Nexus on Azure Kubernetes Service (AKS), ensure you have the following prerequisites:

- **Azure Kubernetes Service (AKS) Cluster**
- **Azure Container Registry (ACR)**
- **Helm 3 installed**
- **kubectl configured to access your AKS cluster**

## 1. Create Namespaces
Create a namespace for each service:

```bash
kubectl create namespace jenkins  
kubectl create namespace sonarqube  
kubectl create namespace nexus  
```

## 2. Configure ACR Authentication
Ensure your AKS cluster has permission to pull images from ACR by creating a Kubernetes secret for ACR authentication:

```bash
kubectl create secret docker-registry acr-secret \
  --docker-server=<ACR-LOGIN-SERVER> \
  --docker-username=<ACR-USERNAME> \
  --docker-password=<ACR-PASSWORD> \
  -n <namespace>
```

Add the `imagePullSecrets` section to your Helm values file:

```yaml
imagePullSecrets:
  - name: acr-secret
```

## 3. Deploy Helm Charts
Navigate to the respective Helm chart directories and install the charts:

```bash
# Jenkins  
cd jenkins-chart  
helm install jenkins . -n jenkins  

# SonarQube  
cd ../sonarqube-chart  
helm install sonarqube . -n sonarqube  

# Nexus  
cd ../nexus-chart  
helm install nexus . -n nexus  
```

Check the status of the pods for each service:

```bash
kubectl get pods -n <namespace>
```

Get the service details:

```bash
kubectl get svc -n <namespace>
```

## 4. Port Forwarding (For SonarQube and Nexus)
If you are not using a LoadBalancer, you can access services via port-forwarding:

```bash
# SonarQube (Default port: 9000)  
kubectl port-forward svc/sonarqube 9000:9000 -n sonarqube  

# Nexus (Default port: 8081)  
kubectl port-forward svc/nexus 8081:8081 -n nexus  
```

You can then access them in your browser:

- **SonarQube**: [http://localhost:9000](http://localhost:9000)
- **Nexus**: [http://localhost:8081](http://localhost:8081)

## 5. Use LoadBalancer for Jenkins
For Jenkins, update your Helm chart or values file to use a LoadBalancer service type:

```yaml
service:
  type: LoadBalancer
  port: 8080
```

After deploying Jenkins with LoadBalancer, get the service details:

```bash
kubectl get svc -n jenkins
```

Access Jenkins at the `EXTERNAL-IP` of the service:

```text
http://<Jenkins-External-IP>:8080
```

## 6. Verify Deployment
Check if all services are running:

```bash
kubectl get pods --all-namespaces
```

## Notes
- Replace `<ACR-LOGIN-SERVER>`, `<ACR-USERNAME>`, and `<ACR-PASSWORD>` with your actual ACR credentials.
- Ensure your AKS cluster has the correct IAM roles and permissions to pull images from ACR.
- Modify the Helm values as needed to customize deployment settings.


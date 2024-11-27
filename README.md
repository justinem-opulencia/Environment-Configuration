# 1. Create dev and prod environment and configurations.
## 1-a. Define the development and production environments using namespaces.

```
kubectl create namespace production
```

## 1-b.Configure Environment-Specific ConfigMaps.

### PRODUCTION CONFIG
```
kubectl create configmap app-config \
  --from-literal=APP_ENV=production \
  --namespace=production
```

## 1-c. Define sensitive information (like database passwords) as Secrets
### PROD SECRETS
```
kubectl create secret generic app-secret \
  --from-literal=DB_PASSWORD=prodpassword123 \
  --namespace=production
```
## 1-d. View all secrets per environment.
```
kubectl get secrets -n production
```
## 1-e. To view the metadata of the secret let us use the describe function:
```
kubectl describe secret app-secret -n production
```
## 1-f. As we can see, the password is not readable by humans. To decode let us run:
```
kubectl get secret app-secret -n production -o jsonpath="{.data.DB_PASSWORD}" | base64 --decode
```
# 2. Deploying deployments to production environment.
## 2-a. Create a generic deployment YAML template. Save it as app-deployment-template.yaml:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-app
  namespace: PLACEHOLDER_NAMESPACE
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-app
  template:
    metadata:
      labels:
        app: nginx-app
    spec:
      containers:
      - name: nginx
        image: nginx
        env:
        - name: APP_ENV
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_ENV
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: DB_PASSWORD
```
## 2-b. Deploy to production environment:
```
sed -e 's/PLACEHOLDER_NAMESPACE/production/' \
    -e 's/replicas: .*$/replicas: 3/' app-deployment-template.yaml > prod-deployment.yaml

kubectl apply -f prod-deployment.yaml
```
## 2-c. Expose the production environment.
```
kubectl expose deployment nginx-app \
  --type=NodePort \
  --name=nginx-service \
  --port=80 \
  --target-port=80 \
  --namespace=production
```
## 2-d. Verify the deployment and access:
```
kubectl get pods -n production
kubectl get service -n production
```
## 2-e. Access the development app:
```
kubectl exec -it <pod-name> -n production -- /bin/bash
echo $APP_ENV
echo $DB_PASSWORD
```

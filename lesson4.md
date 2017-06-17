# Creating Deployments
```sh
kubectl create -f deployments/auth.yaml
kubectl describe deployments auth
kubectl create -f services/auth.yaml
kubectl create -f deployments/hello.yaml
kubectl create -f services/hello.yaml
kubectl create configmap nginx-frontend-conf --from-file=nginx/frontend.conf
kubectl create -f deployments/frontend.yaml
kubectl create -f services/frontend.yaml
```


# Scaling
```sh
kubectl get services frontend
kubectl get replicasets
kubectl get pods -l "app=hello,track=stable"
vim deployments/hello.yaml:6 # change replicas number
kubectl apply -f deployments/hello.yaml
```

We have single frontend service which is profing traffic to three pods of hello


# Rolling update
```sh
vim deployments/auth:15 # change version
kubectl apply -f deployments/auth.yaml
kubectl describe deployments auth
kubectl get pods
kubectl describe pods auth-3648574242-bs7vq
```

# Setting up k8s
### Use project directory
```sh
cd $GOPATH/src/github/com/udacity/ud615/kubernetes
```

or if you are in the course repository:

```sh
cd kubernetes
```

Note: At any time you can clean up by running the cleanup.sh script

### Provision a Kubernetes Cluster with GKE using gcloud

To complete the work in this course you going to need some tools. Kubernetes can be configured with many options and add-ons, but can be time consuming to bootstrap from the ground up. In this section you will bootstrap Kubernetes using Google Container Engine (GKE).

GKE is a hosted Kubernetes by Google. GKE clusters can be customized and supports different machine types, number of nodes, and network settings.

Use the following command to create your cluster for use for the rest of this course.

```sh
gcloud container clusters create k0
```


# Kubernetes Intro Demo
### Launch a single instance:
```sh
kubectl run nginx --image=nginx:1.10.0
```
### Get pods
```sh
kubectl get pods
```

### Expose nginx
```sh
kubectl expose deployment nginx --port 80 --type LoadBalancer
```

### List services
```sh
kubectl get services
```

### Kubernetes cheat sheet
We just went over a lot and we know you’re probably a little overwhelmed. Fear not! We’ll be going over each of these concepts, over the next two lessons. And you can always come back to this demo -- if you need to watch it again.

To help out, here’s a Kubernetes command cheat sheet. http://kubernetes.io/docs/user-guide/kubectl-cheatsheet/


# Creating Pods
### Explore config file
```sh
cat pods/monolith.yaml
```

### Create the monolith pod
```sh
kubectl create -f pods/monolith.yaml
```

### Examine pods
```sh
kubectl get pods
```ls

It may take a few seconds before the monolith pod is up and running as the monolith container image needs to be pulled from the Docker Hub before we can run it.

Use the kubectl describe command to get more information about the monolith pod.

```sh
kubectl describe pods monolith
```

# Interacting With Pods

Cloud shell 1: set up port-forwarding
```sh
kubectl port-forward monolith 10080:80
```

Open new Cloud Shell session 2
```sh
curl http://127.0.0.1:10080
curl http://127.0.0.1:10080/secure
```

Cloud shell 2 - log in
```sh
curl -u user http://127.0.0.1:10080/login # or TOKEN=$(curl 127.0.0.1:10080/login -u user | jq -r '.token')
curl -H "Authorization: Bearer $TOKEN" http://127.0.0.1:10080/secure
```

View logs
```sh
kubectl logs monolith
kubectl logs -f monolith
```

In Cloud Shell 3
```sh
curl http://127.0.0.1:10080
```

You can use the kubectl exec command to run an interactive shell inside the monolith Pod. This can come in handy when you want to troubleshoot from within a container:

kubectl exec monolith --stdin --tty -c monolith /bin/sh
For example, once we have a shell into the monolith container we can test external connectivity using the ping command.

ping -c 3 google.com
When you’re done with the interactive shell be sure to logout.

exit

# Secrets and configmaps

Create secret on master
```sh
kubectl create secret generic tls-certs --from-file=tls/
```

Move secrets to the pods
```sh
kubectl create -f pods/secure-monolith.yaml
```

# Creating Secrets
```sh
ls tls
```

The cert.pem and key.pem files will be used to secure traffic on the monolith server and the ca.pem will be used by HTTP clients as the CA to trust. Since the certs being used by the monolith server where signed by the CA represented by ca.pem, HTTP clients that trust ca.pem will be able to validate the SSL connection to the monolith server.

### Use kubectl

to create the tls-certs secret from the TLS certificates stored under the tls directory:

```sh
kubectl create secret generic tls-certs --from-file=tls/
```

kubectl will create a key for each file in the tls directory under the tls-certs secret bucket. Use the kubectl describe command to verify that:

```sh
kubectl describe secrets tls-certs
```

Next we need to create a configmap entry for the proxy.conf nginx configuration file using the kubectl create configmap command:

```sh
kubectl create configmap nginx-proxy-conf --from-file=nginx/proxy.conf
```
Use the kubectl describe configmap command to get more details about the nginx-proxy-conf configmap entry:

```sh
kubectl describe configmap nginx-proxy-conf

```

### TLS and SSL
TLS and SSL can be confusing topics. Here’s a good primer for understanding the basics: https://en.wikipedia.org/wiki/Transport_Layer_Security


# Accessing A Secure HTTPS Endpoint
```sh
cat pods/secure-monolith.yaml
```

### Create the secure-monolith Pod using kubectl.
```sh
kubectl create -f pods/secure-monolith.yaml
kubectl get pods secure-monolith

kubectl port-forward secure-monolith 10443:443

curl --cacert tls/ca.pem https://127.0.0.1:10443

kubectl logs -c nginx secure-monolith
```

# Creating a Service

```sh
kubectl create -f services/monolith.yaml
gcloud compute firewall-rules create allow-monolith-nodeport --allow=tcp:31000
gcloud compute instances list
```

# Adding Labels to pods
```sh
kubectl get pods -l "app=monolith,secure=enabled"
kubectl describe pods secure-monolith | grep Labels
kubectl label pods secure-monolith "secure=enabled"
kubectl describe services monolith | grep Endpoints
gcloud compute instances list
curl -k https://104.199.52.96:31000
```

# Creating a Service
a Service is a method for exposing a network application that is running as one or more Pods in your cluster. Different from Pod, Service is a long running object. A service has an IP address and has stable fixed port. We can attach a Service to a Pod.

The Pod has a <code>label</code> which is a key value pair. The Service has a <code>selector</code> which is a key value pair to. The Service pointing to the Pod based on the match the key value pair on the <code>selector</code> and <code>lable</code>.

<img src="images/service-pod.png" alt=""  width="75%"/>

If a Service we create is designed to be access from internal cluster (such as microservice internal communication), we set the type to <code>ClusterIP</code>. On the other hand if we design it to be accessible from outside cluster, set the type to <code>NodePort</code>/.

The port that is allowed in Kubernetes cluster is greater than 30000

Let's create a <code>service.yaml</code>
``` yaml
apiVersion: v1
kind: Service
metadata:
  name: fleetman-webapp
  namespace: default
spec:
  # This defines which pods are going to be represented by this service
  # The service becomes a network endpoint for either other services
  # or maybe external users to connect to (browser)
  selector:
    app: webapp # # can be any selector according to the label defined in pod ex: myapp:webapp
  ports:
  - name: http
    port: 80
    # the nodeport should be greater than 30000
    nodePort: 30080 
  type: NodePort
```

``` bash
kubectl apply -f service.yaml
```
Once the service created, check the status of serice and get ip of node (in this case we use minikube).

<img src="images/access-app-from-outside.png" alt="" width="75%"/>

Then open browser:
http://192.168.59.100:30080

<img src="images/open-app-0.png" alt="" width="75%"/>


# Updating Version of App

Let's assume we want to update app to a new release, from <code>richardchesterwood/k8s-fleetman-webapp-angular:release0</code> to <code>richardchesterwood/k8s-fleetman-webapp-angular:release0-5</code>

In this example we will add additional label called <code>release</code> and a new Pod named <code>webapp-release-0-5</code> which represent a new release app.

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
  labels:
    app: webapp # can be any label ex, myapp: webapp
    release: "0" # should be a string
spec:
  containers:
  - name: webapp
    image: richardchesterwood/k8s-fleetman-webapp-angular:release0

---
apiVersion: v1
kind: Pod
metadata:
  name: webapp-release-0-5
  labels:
    app: webapp # can be any label ex, myapp: webapp
    release: "0-5" # should be a string
spec:
  containers:
  - name: webapp
    image: richardchesterwood/k8s-fleetman-webapp-angular:release0-5
```
For service, we also add additional selector
``` yaml
apiVersion: v1
kind: Service
metadata:
  name: fleetman-webapp
  namespace: default
spec:
  # This defines which pods are going to be represented by this service
  # The service becomes a network endpoint for either other services
  # or maybe external users to connect to (browser)
  selector:
    app: webapp # # can be any selector according to the label defined in pod ex: myapp:webapp
    release: "0" # if we change this to "0-5" the service will point to app version 0-5. It is very fast without downtime.
  ports:
  - name: http
    port: 80
    # the nodeport should be greater than 30000
    nodePort: 30080 
  type: NodePort
```

Let's apply the <code>pod.yaml</code> and <code>service.yaml</code>
``` bash
kubectl apply -f pod.yaml 
kubectl apply -f service.yaml 
```

Once the second Pod <code>webapp-release-0-5</code> up, we can change the <code>release</code> value from <code>"0"</code> to <code>"0-5"</code> in the service. The changes in release in Service will pointing it to app version 0-5. It is very fast without downtime.

<img src="images/service-pod-change-selector.png" alt="" width="75%"/>

We can check further in the detail service information

<img src="images/service-info-change-selector.png" alt="" width="75%"/>

> ### Tips
> We can search/list Pod by filtering on the command
> ``` bash
> kubectl get pod --show-labels -l release=0-5
> ```

If we refresh/force reload browser (http://192.168.59.100:30080), it will load the new release of app (version 0-5).


# Service Port, TargetPort, and NodePort
There are several different port configurations for Kubernetes services:

- **Port** exposes the Kubernetes service on the specified port within the cluster. Other pods within the cluster can communicate with this server on the specified port.

- **TargetPort** is the port on which the service will send requests to, that your pod will be listening on. Your application in the container will need to be listening on this port also.

- **NodePort** exposes a service externally to the cluster by means of the target nodes IP address and the NodePort. NodePort is the default setting if the port field is not specified.


Beside may have the 3 kind of ports, Kubernates service may have multiple port as well. Here is an example of service which has multiple port.

``` yaml

kind: Service
apiVersion: v1
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
      # From inside the cluster, if you hit the my-service:8089 the traffic is routed to 8080 of the container(targetPort)
      # From outside the cluster, if you hit host_ip:30475 the traffic is routed to 8080 of the container(targetPort)
    - name: http
      protocol: TCP
      targetPort: 8080
      port: 8089
      nodePort: 30475

    - name: metrics
      protocol: TCP
      targetPort: 5555
      port: 5555
      nodePort: 31261
     
     # From inside the cluster, if you hit my-service:8443 then it is redirected to 8085 of the container(targetPort)
     # From outside the cluster, if you hit host_ip:30013 then it is redirected to 8085 of the container(targetPort)
    - name: health
      protocol: TCP
      targetPort: 8085
      port: 8443 
      nodePort: 30013
```
#  About
Kubernetes provides probes (health checks) to monitor and act on the state of Pods (Containers) and to make sure only healthy Pods serve traffic. With help of Probes, we can control when a pod should be deemed started, ready for service, or live to serve traffic.

Probes are used to detect:
- Containers that haven’t started yet and can’t serve traffic.
- Containers that are overwhelmed and can’t serve additional traffic.
- Containers that are completely dead and not serving any traffic.


Kubernetes gives you three types of health checks probes:
- Liveness
- Readiness
- Startup

# Ways to check Pods/Containers’ health using Probes
Kubelet can check a Pods’ health in three ways. Each probe must define exactly one of these mechanisms:

### HTTP (Http Get)
- We define a Port number along with the URL. Kubernetes pings this path, and if it gets an HTTP response in the 200 or 300 range, it marks the app as healthy. Otherwise, it is marked as unhealthy.
- HTTP probes are the most common type of custom probe.
- Even if your app isn’t an HTTP server, you can create a lightweight HTTP server inside your app to respond to the liveness probe.

``` yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-http

spec:
  containers:
  - name: liveness
    image: registry.k8s.io/liveness
    args:
    - /server
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
        httpHeaders:
        - name: Custom-Header
          value: Awesome
      initialDelaySeconds: 3
      periodSeconds: 3

```
### TCP (Tcp Socket)
- Kubernetes tries to establish a TCP connection on the specified port. If it can establish a connection, the container is considered healthy; if it can’t it is considered unhealthy.
- TCP probes come in handy if you have a scenario where HTTP probes or Command probes don’t work well.
- For example, a gRPC or FTP service is a prime candidate for this type of probe.

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: goproxy
  labels:
    app: goproxy
spec:
  containers:
  - name: goproxy
    image: registry.k8s.io/goproxy:0.1
    ports:
    - containerPort: 8080
    readinessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 10
    livenessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 20
```

### Command (Exec Command)
- Kubernetes runs a command inside the container. If the command returns with exit code 0, then the container is marked as healthy. Otherwise, it is marked unhealthy.
- This type of probe is useful when you can’t or don’t want to run an HTTP server, but can run a command that can check whether or not your app is healthy.

``` yaml
```

# References
- https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/
- https://medium.com/devops-mojo/kubernetes-probes-liveness-readiness-startup-overview-introduction-to-probes-types-configure-health-checks-206ff7c24487
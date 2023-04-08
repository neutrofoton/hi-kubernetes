# Microservice 
In this lab, we will use example provided by [DickyChesterwood](https://github.com/DickChesterwood/k8s-fleetman/tree/release0-reconstruction-branch). The docker images are available on [Docker Hub](https://hub.docker.com/u/richardchesterwood)

The scenario as shown in this diagram
 <img src="images/microservice-fleetman.png" alt="" width="75%"/>

### Deploying Queue (ActiveMQ)
First of all we will create <code>active-mq.yaml</code>
``` yaml
# ActiveMQ Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: queue
spec:
  selector:
    matchLabels:
      app: queue
  replicas: 1
  template: 
    metadata:
      labels:
        app: queue
    spec:
      containers:
      - name: queue
        image: richardchesterwood/k8s-fleetman-queue:release1

---
# ActiveMQ Service
apiVersion: v1
kind: Service
metadata:
  name: fleetman-queue

spec:
  # This defines which pods are going to be represented by this Service
  # The service becomes a network endpoint for either other services
  # or maybe external users to connect to (eg browser)
  selector:
    app: queue

  ports:
    - name: http
      port: 8161
      nodePort: 30010

    - name: endpoint
      port: 61616

  type: NodePort
```

``` bash
kubectl apply -f active-mq.yaml
```

<img src="images/active-mq-deploy-svc.png" alt=""/>

Now open in browser http://192.168.59.104:30010/

<img src="images/active-mq-ui.png" alt="" width="75%"/>

### Deploying Position Simulator

In the diagram, the position simulator is not designed to be accessed from outside the cluster. So it should be isolated and no port needed.

``` yaml
# Position Simulator Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: position-simulator
spec:
  selector:
    matchLabels:
      app: position-simulator
  replicas: 1
  template: # template for the pods
    metadata:
      labels:
        app: position-simulator
    spec:
      containers:
      - name: position-simulator
        image: richardchesterwood/k8s-fleetman-position-simulator:release1
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: production-microservice
```

Save the above yaml as <code>position-simulator.yaml</code> and run apply command.
``` bash
kubectl apply -f position-simulator.yaml
```

Once the position simulator running, we should see queue record in the ActiveMQ dashboard.

<img src="images/active-mq-queue.png" alt="" width="75%"/>

# Investigating Kubernates Log
If we get an error on resource we can chek
``` bash
kubectl logs position-simulator-56686f95f8-xx2t8

# follow the log
kubectl logs -f position-simulator-56686f95f8-xx2t8
```

# References
1. https://github.com/DickChesterwood/k8s-fleetman/tree/release0-reconstruction-branch
2. https://www.nginx.com/blog/microservices-at-netflix-architectural-best-practices/
# Creating a Pod

Let's create a simple <code>pod.yaml</code> using a docker image [richardchesterwood/k8s-fleetman-position-tracker](https://hub.docker.com/r/richardchesterwood/k8s-fleetman-webapp-angular)

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - name: webapp
    image: richardchesterwood/k8s-fleetman-webapp-angular:release0
```

To create a Pod, run the following command
``` bash

# apply will create if it does not exist, and update if it exists
kubectl apply -f pod.yaml 
```

We can check list of pods by running
``` bash
kubectl get pods
```

> We can also check all resources at once
> ``` bash
> kubectl get all
> ```

We can also delete pods
``` bash
# the actual pod.yaml on your local working directory will not be deleted
kubectl delete -f pod.yaml
```

If we get an error we can check the clue in
``` bash
kubectl describe pod {POD_NAME}
```

we also check the log
``` bash
kubectl logs --tail=[number] -p {POD_NAME}
```

Pod is not designed to be accessed from outside the cluster. But we still able to connect and execute command inside the pod.

``` bash
kubectl exec {POD_NAME} {COMMAND}
kubectl exe webapp ls
```

We could also get a shell interraction inside the container in the pod.
``` bash

# execute shell command
kubectl -it exec {POD_NAME} sh
kubectl -it exec webapp sh

```

Inside the container we can do 
``` bash
# for the example
wget http://localhost:80
```
# References
1. https://kubernetes.io/docs/concepts/workloads/pods/
2. https://signoz.io/blog/kubectl-logs-tail/
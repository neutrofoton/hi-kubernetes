# Installations

## Installing kubectl
Installation steps of <code>kubectl</code> can be found [here](https://kubernetes.io/docs/tasks/tools/)

``` bash
# install with brew
brew install kubectl

# check the version
kubectl version --client

# or the new command check version
kubectl version --output=json
```
## Installing minikube
Installation steps of <code>minikube</code> can be found [here](https://minikube.sigs.k8s.io/docs/start/)

``` bash
# install with brew
brew install minikube

# check the installation
minikube version
```

```
minikube version: v1.29.0
commit: ddac20b4b34a9c8c857fc602203b6ba2679794d3
```

Starting the <code>minikube</code>
``` bash
# starting minikube
minikube start

# starting minikube on windows with hyperv
minikube start --driver=hyperv

```
> The environtiment machine I use:
> 1. Macos Monterey 12.6.3
> 2. VirtualBox-6.1.42 for Macos (intel)
> 3. Microsoft Windows 10 Pro
> 
> On Macos, I got an issue when using version 7.0 of virtual box. The issue was <code>["The host-only adapter we just created is not visible"]( https://github.com/kubernetes/minikube/issues/15377)</code>. It is solved by downgrading the VirtualBox version to 6.1.42.
> 
>Meanwhile, on Windows 10 Pro no issue I got by using parameter <code>--driver=hyperv</code>. Don't forget to run the terminal as <code>Administrator</code> otherwise it may get permission issue.


When we successfully starting minikube, checking the version will give us server and client information

``` bash
kubectl version --output=json
```

``` json
{
  "clientVersion": {
    "major": "1",
    "minor": "25",
    "gitVersion": "v1.25.2",
    "gitCommit": "5835544ca568b757a8ecae5c153f317e5736700e",
    "gitTreeState": "clean",
    "buildDate": "2022-09-21T14:33:49Z",
    "goVersion": "go1.19.1",
    "compiler": "gc",
    "platform": "darwin/amd64"
  },
  "kustomizeVersion": "v4.5.7",
  "serverVersion": {
    "major": "1",
    "minor": "26",
    "gitVersion": "v1.26.1",
    "gitCommit": "8f94681cd294aa8cfd3407b8191f6c70214973a4",
    "gitTreeState": "clean",
    "buildDate": "2023-01-18T15:51:25Z",
    "goVersion": "go1.19.5",
    "compiler": "gc",
    "platform": "linux/amd64"
  }
}
```

If we want to access app deployed in the minikube cluster, we use the IP of minikube which can be checked using:
``` bash
minikube ip
```

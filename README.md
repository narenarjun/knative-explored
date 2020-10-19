# Knative Explored 🔰🔰🔰🔰

## Knative serving

Knative Serving builds on Kubernetes to support deploying and serving of applications
and functions as serverless containers.
Serving is easy to get started with and scales to support advanced scenarios.

# Preparing the cluster with knative

Installing all the Knative Serving related components.

### Tools needed:

* kubectl 
* istioctl
* kn [cli for knative which is under active development.]


### Knative crd installed :

```bash
kubectl apply -f ./install/knative-serving/serving-crds.yaml
```

### Knative core installed:

```bash
kubectl apply -f ./install/knative-serving/serving-core.yaml
```

### Installing a network layer for knative:

Istio is choosen for this Cluster network layer.

```bash
kubectl apply -f ./install/knative-serving/istio-release.yaml
```

> ### 📚 Note:
> you don't have to choose `istio` for your cluster network layer, there are other options such as `gloo`, `contour`, `kong`. learn more and pick one from this [installing page.](https://knative.dev/docs/install/any-kubernetes-cluster/#serving_networking)


### Service Mesh

Installing Istio for service mesh in the cluster.

Automatic sidecar injection is set to `autoInject: enabled` in operator configuration.
So, any namespace with the label `istio-injection=enabled` will have servicemesh sidecar injected automatically.

```yaml
global:
  proxy:
    autoInject: enabled
```

```bash
istioctl mainfest apply -f ./install/istio/istio-minimal-operator.yaml
```

### Configure DNS

For this instance, `magic DNS` is installed, [xip.io](http://xip.io/)

```bash
kubectl apply ./install/knative-serving/serving-default-domain.yaml
```
> ###  What is xip.io?
>xip.io is a magic domain name that provides wildcard DNS
>for any IP address.

### verifying everything is installed.

We want all the pods to be to a status of `Running` or `Completed`, which ensures knative installed smoothly.

```bash
>>$ kubectl get pods -n knative-serving
NAME                                READY   STATUS      RESTARTS   AGE
controller-7859944464-lmmp5         1/1     Running     0          45h
autoscaler-598bc966b4-ncqtx         1/1     Running     0          45h
webhook-dfc6974b-bpg97              1/1     Running     0          45h
activator-7dc7bc9d8-xhs4r           1/1     Running     0          45h
networking-istio-74c8f76b5c-ddvpn   1/1     Running     0          45h
istio-webhook-7f74d9f849-nmsvc      1/1     Running     0          45h
default-domain-drjth                0/1     Completed   0          44h

```
> ### Note
> The age shows 45h, since i got it deployed and checked before writing the readme, so it very long. When you set and run `kubectl get pods -n knative-serving`, the age will be much less since you set that just then.

## Deployments

Create a namespace for these deployments to come.

```bash
kubctl create ns knplayground
```

Labeling the namespace for automatic istio service mesh sidecar injection.

```bash
kubectl label ns knplayground istio-injection=enabled
```

Now,Lets deploy some sample app and see the knative in action.

```bash
kubectl apply -f ./serving-yamls/service.yaml
```

Get the created  knative service details with

* `kubectl`:
```bash
kubectl get ksvc
```
or 

* `kn`:
```bash
kn service ls
```

`curl` the url shown in the result.

#### revision

let's update our knative deployment.

```bash
kubectl apply -f  ./serving-yamls/service-env.yaml
```
let's see the revisions via 

* `kubectl`:
```bash
kubectl get revisions
```
or 

* `kn` :
```bash
kn revision ls
```

`curl` the url to see the result.
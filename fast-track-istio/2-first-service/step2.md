You can define custom labels for all Kubernetes objects, which are essentially key-value pairs. Istio relies on a specific label named `istio-injection` to decide the namespace on which the Envoy proxies are applied. This label supports two values. Setting the label's value to _enabled_ will instruct Istio to deploy sidecars for all the pods of your service automatically. On the other hand, setting this value to _disabled_ means that Istio will not inject sidecars automatically; however, it won’t affect the services within the namespace that have sidecars attached to them.

The journey to virtualization in enterprises is an incremental process. What it means is that in most of the cases, organizations would want to gradually migrate their existing workloads running on Kubernetes to the service mesh. In such cases, you will have to inject Envoy sidecars to existing services manually.

To simulate this scenario, let’s deploy a service to our cluster with automatic sidecar injection turned off. Explore the specification in the `my-worksop\book-club.yaml` file that we will use to deploy a stateless service to our cluster.

Let's now deploy this service to our cluster with the following command.

`kubectl apply -f my-workshop/book-club.yaml`{{execute}}

Once deployed, you can check the containers created in the pods of this service with the following command.

`kubectl get pods --selector app=bookclub -n fast-track-istio -o jsonpath={.items[*].spec.containers[*].name}`{{execute}}

The previous command might not display results immediately if the pods are not yet created. If you don't receive any results, wait for some time and try executing the previous command again.

A _NodePort_ service can be accessed through any machine on the cluster through the specified `nodePort`. Let's explore our service by navigating to the following URL.

https://[[HOST_SUBDOMAIN]]-30001-[[KATACODA_HOST]].environments.katacoda.com/

> Note: Currently, only the home page of the application is operational because we haven't yet deployed all the supporting microservices that this application requires. We will deploy all the services to the mesh to make the application fully operational in the next exercise.

Let's bring this service to the mesh now.

## Inject sidecar

To bring a service to the mesh, we will use _istioctl_ to inject a sidecar to the _bookclub_ application. The `kube-inject` command of istioctl can modify a deployment specification to provision sidecars for supported resources and leave the rest, such as secrets and policies.

Execute the following command to view the specification generated by the _istioctl_ CLI for our service.

`istioctl kube-inject -f my-workshop/book-club.yaml`{{execute}}

The generated specification instructs Kubernetes to provision two additional containers for each replica requested by us. The first container provisioned by Kubernetes in the deployment sequence is the _istio-init_ container, a special container that must execute to a conclusion before any other containers can be provisioned. Next, the _istio-proxy_ and the application containers are provisioned as expected.

By using some bash magic, we can generate the specification and execute it with a single command.

`istioctl kube-inject -f my-workshop/book-club.yaml | kubectl apply -f -`{{execute}}

After a couple of seconds, let's enumerate the containers again.

`kubectl get pods --selector app=bookclub -n fast-track-istio -o jsonpath={.items[*].spec.containers[*].name}`{{execute}}

The _istio-proxy_ container is the sidecar that intercepts manages the traffic en route to and from your application.

Now that you understand how sidecars work. Let's recreate the service with automatic sidecar injection. You can delete the namespace and its services with the following command.

`kubectl delete namespace fast-track-istio`{{execute}}

Let's create the service again in the next step.

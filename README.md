# vmware-automation

**Install a kubernetes cluster on AWS.**
 
 We can use kops.
 Kops, is an open source project used to set up Kubernetes clusters easily and swiftly. It's considered the “kubectl” 
 way of creating clusters. Kops allows deployment of highly available Kubernetes clusters on AWS and Google (GCP) clouds.
 I followed this tutorial and installed a 2 nodes cluster [link](https://conpilar.es/como-crear-un-cluster-de-kubernetes-con-kops/)
 
**Install ingress on the cluster**

Ingress is a Kubernetes resource type, that can be applied just like other resources. Its purpose is to define routing cluster-external requests to cluster-internal services. 
An Ingress will map URLs (hostname and path) to cluster-internal services.
We will use nginx as ingress.

<pre><code>
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.0.0/deploy/static/provider/cloud/deploy.yaml


Which will generate the following output:
namespace/ingress-nginx created
serviceaccount/ingress-nginx created
configmap/ingress-nginx-controller created
clusterrole.rbac.authorization.k8s.io/ingress-nginx created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx created
role.rbac.authorization.k8s.io/ingress-nginx created
rolebinding.rbac.authorization.k8s.io/ingress-nginx created
service/ingress-nginx-controller-admission created
service/ingress-nginx-controller created
deployment.apps/ingress-nginx-controller created
ingressclass.networking.k8s.io/nginx created
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission created
serviceaccount/ingress-nginx-admission created
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
role.rbac.authorization.k8s.io/ingress-nginx-admission created
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
job.batch/ingress-nginx-admission-create created
job.batch/ingress-nginx-admission-patch created


Validate the controller is running:
kubectl get pods --all-namespaces -l app.kubernetes.io/name=ingress-nginx
</pre></code>



# vmware-automation

**Install a kubernetes cluster on AWS.**
 
 We can use kops.
 Kops, is an open source project used to set up Kubernetes clusters easily and swiftly. It's considered the “kubectl” 
 way of creating clusters. Kops allows deployment of highly available Kubernetes clusters on AWS and Google (GCP) clouds.
 I installed a cluster composed by two nodes in eu-west-1 and running in one availibity zone.
 [link](https://conpilar.es/como-crear-un-cluster-de-kubernetes-con-kops/)
 
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
![alt text](https://github.com/joanbm91/vmware-automation/blob/main/images/ingress-nginx%20controller.PNG)

A new load balancer should be created on AWS. We should point our domain to the dns.
I'm using the domain plexustv.net and I added a CNAME entry.
<p>
test.plexustv.net       CNAME         a99479daaa3114ea6b774ac0e1f12bf5-2079315275.eu-west-1.elb.amazonaws.com 
</p>

**Create a S3 bucket where the files will be stored**
Create the bucket, I'm working on region eu-west-1 and I called the bucket: juanborrask8sproxy
Enable website hosting.
Add a index.html example file.
Copy the S3 bucket url, http://juanborrask8sproxy.s3-website-eu-west-1.amazonaws.com


**Create a kubernetess service**

<pre><code>kubectl -f apply s3-ingress-service.yaml</pre></code>

<pre><code>
kind: Service
apiVersion: v1
metadata:
  name: s3-bucket
  namespace: juan-borras2
spec:
  type: ExternalName
  externalName: juanborrask8sproxy.s3-website-eu-west-1.amazonaws.com
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 80

</pre></code>

Describe the service

![alt text](https://github.com/joanbm91/vmware-automation/blob/main/images/describe%20service.PNG)

**Create a ingress resource**

<pre><code>kubectl -f apply s3-ingress-rule.yaml</pre></code>

<pre><code>
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/upstream-vhost: juanborrask8sproxy.s3-website-eu-west-1.amazonaws.com
    nginx.ingress.kubernetes.io/from-to-www-redirect: "true"
  name: ingres-s3-bucket
  namespace: juan-borras2
spec:
  rules:
  - host: test.plexustv.net
    http:
      paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: s3-bucket
              port:
                number: 80
</pre></code>        

List ingres configuration:
![alt text](https://github.com/joanbm91/vmware-automation/blob/main/images/describe%20ingres.PNG)

**Check the access**

Accessing to test.plexustv.net shows the index.html uploaded to the S3 bucket.
For troubleshooting check the nginx ingress controller logs.

<pre><code>kubectl logs ingress-nginx-controller-fd7bb8d66-tn6ps -n ingress-nginx</pre></code>
![alt text](https://github.com/joanbm91/vmware-automation/blob/main/images/access%20logs.PNG)

![alt text](https://github.com/joanbm91/vmware-automation/blob/main/images/result.PNG)

**Add basic security**
We can add basic security with kubernetes secrets.

First of all encrypt a user password with htpasswd (requires apache2-utils)
<pre><code>htpasswd -c auth admin</pre></code>

Convert htpasswd into a secret, store it at the same namespace where the ingress its configured.
<pre><code>kubectl create secret generic basic-auth --from-file=auth -n juan-borras2</pre></code>

Check the stored secret
<pre><code>kubectl get secret basic-auth -o yaml</pre></code>
![alt text](https://github.com/joanbm91/vmware-automation/blob/main/images/auth.PNG)

Modify the ingress resource adding the following lines:
<pre><code> 
nginx.ingress.kubernetes.io/auth-type: basic
# name of the secret that contains the user
nginx.ingress.kubernetes.io/auth-secret: basic-auth
nginx.ingress.kubernetes.io/auth-realm: 'Authentication Required'
</pre></code> 

Test the access, a login prompt should appear asking credentials.

![alt text](https://github.com/joanbm91/vmware-automation/blob/main/images/login.PNG)


**Remember destroy the kops cluster :)**
<pre><code> kops delete cluster clustername --yes</pre></code> 

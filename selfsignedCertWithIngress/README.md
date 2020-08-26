# LaunchK8 Application in Minikube using K8 Ingress and Self Signed Certificate
In this section, We will see you how we can  leverage K8 Ingress and we can launch our kubernetes application in minicluster using a self signed certificate.

## Step 1: Launch minikube cluster with addons Ingress.
Check whether the minikube Ingress is enabled by default. Type `minikube addons list`, 
![https://github.com/jchowdhary/k8IngressWithCerts/blob/master/selfsignedCertWithIngress/minikube_addon_list.JPG](https://github.com/jchowdhary/k8IngressWithCerts/blob/master/selfsignedCertWithIngress/minikube_addon_list.JPG)

If they are not enabled by default,execute this command `minikube addons enable ingress`
an use any kubernetes deployment, Here..I have taken the kubernetes deployment from open source project from https://github.com/DickChesterwood/k8s-fleetman.
Pl. see the artifact

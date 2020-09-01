# LaunchK8 Application in Minikube using K8 Ingress and Self Signed Certificate
In this section, We will see you how we can  leverage K8 Ingress and we can launch our kubernetes application in minicluster using a self signed certificate.

## Step 1: Launch minikube cluster with addons Ingress.
Check whether the minikube Ingress is enabled by default. Type `minikube addons list`, 
![https://github.com/jchowdhary/k8IngressWithCerts/blob/master/selfsignedCertWithIngress/minikube_addon_list.JPG](https://github.com/jchowdhary/k8IngressWithCerts/blob/master/selfsignedCertWithIngress/minikube_addon_list.JPG)

If they are not enabled by default,execute this command `minikube addons enable ingress`. This will enable the minikube Ingress when minikube starts.
Start your minikube cluster.

## Step 2: Launch your K8 Application
In this example,we have taken an open source kubernetes application from Richard Chesterwood site.Pl. refer the open source project from https://github.com/DickChesterwood/k8s-fleetman. We launch this application using the kubectl command `kubectl apply -f 1-application-no-istio.yaml`. This yaml deployment will launch fleetman-webapp pods and services.
You can see the screen what all workloads and services are launched by this YAML.

We see that fleetman-webapp service has ClusterIP configured, so we can expose the endpoint to outside world. We will use Kubernetes Ingress which is packaged with our minikube cluster.

## Step 3: Configure your K8 Ingress
We create an Kubernetes Ingress YAML file which will configure the backend service and port. The Ingress external IP will be the same as `minikube ip`. We have given a DNS name "fleetman-webapp.com" here. Update the hosts file `\etc\hosts\`  with the IP{same as minikube ip} and this DNS{fleetman-webapp.com} name.
![https://github.com/jchowdhary/k8IngressWithCerts/blob/master/selfsignedCertWithIngress/K8Ingress.JPG](https://github.com/jchowdhary/k8IngressWithCerts/blob/master/selfsignedCertWithIngress/K8Ingress.JPG)
NB: Specify the backend service name and service  port properly.

## Step 4: Launch the K8 Ingress
Exceute the command `kubectl apply -f 2-fleetman-ingress.yaml`. Once done,pl. launch your browser and open the URL `http://fleetman-webapp.com`.This will launch your application in your browser.

## Step 5: Configure HTTPS with Self Signed Certificate.

Point 

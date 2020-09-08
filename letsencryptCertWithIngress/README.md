# LaunchK8 Application in Minikube using Lets Encrypt CA signed Certificate Using CertManager 
In this section, We will create the workable solution on how we can automatically provision Let's Encrypt CA signed certifcates on our Kubernetes Cluster. To simplify things, we will workout this example inside a Minikube..However, you can leverage this example in any Kubernetes Cluster in any cloud vendor.

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
We need to run our application in a trusted mode, so we would need certificates.. In this example, we will understand how we can create self signed certificates and how we manually use the certificate in our kubernetes application and launch our application in a secured TLS channel.

## Step 6: Create a Self Signed certificate using OpenSSL
We would be using OpenSSL to create a self signed certificate. If you do not have OpenSSL, you can download it from https://www.openssl.org/source/.
`openssl req -x509 -newkey rsa:4096 -sha256 -nodes -keyout fleetman.key -out fleetman.crt -subj '//CN=fleetman-webapp.com' -days 365`
Pl. execute the above command to create <i>fleetman.key</i> and <i>fleetman.crt</i>. You can find the same attached with this artifact as well.

## Step 7: Create a Kubernetes TLS secret from the self signed certificate and key.
Kubernetes save the secret credentials in K8 Secret. Here in this example,we will create a TLS secret and configure in our minikube kubernetes cluster.
Execute the command, `kubectl create secret tls fleetman-webapp-secret --cert=fleetman.crt --key=fleetman.key`
This will create a secret named as <i> fleetman-webapp-secret </i>. You can confirm the secret by typing `kubectl get secret -o yaml`. This will dump out all the certificate related information, token string, certificate metadata information, public key, signed encrypted algorithm details.

## Step 8: Configure the K8 Ingress to work with K8 secret to launch the application in HTTPS
Once, the secret is created, we need to configure our existing Ingress(<i> fleetman-ingress.yaml </i> so that application can listen in 443 secured port.
We need to edit our existing yaml by introducing the below lines.

![https://github.com/jchowdhary/k8IngressWithCerts/blob/master/selfsignedCertWithIngress/Ingress.JPG](https://github.com/jchowdhary/k8IngressWithCerts/blob/master/selfsignedCertWithIngress/Ingress.JPG)

If you see the screenshot above, we have mentioned <i> fleetman-webapp-secret </i> in the <i> <tls>..</i> section.Once changes are done, execute
`kubectl apply -f fleetman-ingress.yaml ` to apply the Ingress in our minikube cluster.
  
## Step 9. Launch your application in the browser.
Launch the browser and open the url <i> https://fleetman-webapp.com </i>. You will see your Kubernetes Application is launched in HTTPs mode.

![https://github.com/jchowdhary/k8IngressWithCerts/blob/master/selfsignedCertWithIngress/kubernetesApp.JPG](https://github.com/jchowdhary/k8IngressWithCerts/blob/master/selfsignedCertWithIngress/kubernetesApp.JPG)

In this example, we have seen how we can manually introduce a self signed certificate in our Kubernetes Application.

In the upcoming section,we will see how we can automate certificate provisioing using self signed certificate and CA Root Let's Encrypt using CertManager.


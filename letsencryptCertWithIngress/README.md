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

## Step 3: Launch your Ingress
Since, we need to launch the application externally, we would be using Nginx Ingress Controller here. This Nginx Ingress Controller will program a front-end Load Balancer to enable Ingress Configurations.First, we would launch the Ingress Resource and then apply the Nginx Ingress Controller on it.

From your CLI, type '`kubectl apply -f fleetman-ingress-nginxcontroller.yaml` and execute. As per the screenshot below, we will create an Ingress. As of now, we did not define the `TLS` tag yet..We can verify sucessfull Ingress creation by entering `kubectl get ing` in the CLI prompt.

![https://github.com/jchowdhary/k8IngressWithCerts/blob/master/letsencryptCertWithIngress/fleetman-ingress-nginxcontroller.jpg](https://github.com/jchowdhary/k8IngressWithCerts/blob/master/letsencryptCertWithIngress/fleetman-ingress-nginxcontroller.jpg)

<i>NB: Specify the backend service name and service  port properly.Also,ignore the `annotation` metadata tag with cert-manager, we will revisit it later.</i>

## Step 4: Configure NGINX Ingress Controller and Check the External IP in K8 Ingress
Execute the command `kubectl apply -f ingress-nginx-controller.yaml`. This will configure the kubernetes cluster with `ingress-nginx` namespace, configmaps, clusterrolebinding, rolebindings,serviceaccounts and deployments. You can find the YAML from https://www.nginx.com/products/nginx/kubernetes-ingress-controller/

Once sucessfully done,pl. revisit and verify the Ingress external IP now.. As per the screesnhot below,you can see `Address` section updated with an external IP.

![https://github.com/jchowdhary/k8IngressWithCerts/blob/master/letsencryptCertWithIngress/fleetman-ingress-ip.JPG](https://github.com/jchowdhary/k8IngressWithCerts/blob/master/letsencryptCertWithIngress/fleetman-ingress-ip.JPG)</br>
<i>NB:In your cloud enviornment, this IP is a external IP which can be accessed from outside the cluster. So,we can be reserve this IP and give a DNS name. Since, we are doing in minikube, we will not be doing anything here with the host file as we did with the manual provisioing of certificate example.one in the last example as we would want this application to be accessed externally from anywhere.Here, we will use an utility called NGROK i.e. https://ngrok.com/ </i>

## Step 5: Download Cert-Manager Certificate Issuer
We would use a 3rd party utility i.e. Cert-Manager to issue CA signed certificates. Pl. visit https://github.com/jetstack/cert-manager. This is a very useful tool from where we can generate certificates for Staging, Production and also use Vaults as well. In this example, we are using Let's Encrypt Staging certificate for our application. This Cert-Manager will automatically provision your certificates and secrets. We need not worry anymore for expired certificates and only self signed certificates.

![https://github.com/jchowdhary/k8IngressWithCerts/blob/master/letsencryptCertWithIngress/high-level-overview.svg](https://github.com/jchowdhary/k8IngressWithCerts/blob/master/letsencryptCertWithIngress/high-level-overview.svg)

## Step 6: Configure Cert Manager
All resources (the CustomResourceDefinitions, cert-manager, namespace, and the webhook component) are included in a single YAML manifest file. 
Install the CustomResourceDefinitions and cert-manager itself:
<br><i> For Kubernetes 1.16+ </i>
<br>`$ kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v1.0.1/cert-manager.yaml`
<br><i> For Kubernetes <1.16 </i>
<br>`$ kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v1.0.1/cert-manager-legacy.yaml`

Check the `namespaces` created in the kubernetes cluster. Execute `kubectl get ns`, you might find something like below screenshot.
You will see namespaces like `cert-manager`,`ingress-nginx` with the other namespaces.

![https://github.com/jchowdhary/k8IngressWithCerts/blob/master/letsencryptCertWithIngress/nslist.JPG](https://github.com/jchowdhary/k8IngressWithCerts/blob/master/letsencryptCertWithIngress/nslist.JPG)

Once cert-manager has been deployed, you must configure Issuer or ClusterIssuer resources which represent certificate authorities.Issuer can work in multiple clusters and ClusterIssuer works in the specific cluster where it is deployed. This will be created in the cert-manager namespace. We will see in Step 8.

## Step 7: Execute NGROK for tunneling Ingress
We have already seen that Kuberntes Ingress when described , it outputs external IP. In a real Cloud world,this IP should have been reserved and given a DNS name so that it is available to external world. For Minikube, we can use NGROK so that local IP can be tunnelled to external world and we can reference the IP in the Let's Encrypt Certificate Issuance. NGROK will make to believe the Let's Encrypt that this DNS really exists and hence we get the CA signed certificate.

Install NGROK from https://ngrok.com/ and execute the command, `NGROK http <Address which is exposed by Ingress>`. In my case, when i did `kubectl get ing`, I got the IP as `192.168.99.100`. Once NGROK executed, you will see similar screenshot as below.

![https://github.com/jchowdhary/k8IngressWithCerts/blob/master/letsencryptCertWithIngress/ngrok_capture.jpg](https://github.com/jchowdhary/k8IngressWithCerts/blob/master/letsencryptCertWithIngress/ngrok_capture.jpg)

Now,update the Ingress YAML file with the below TLS ,HOST and secret name information and apply the command `kubectl apply -f fleetman-ingress-nginxcontroller.yaml`.

![https://github.com/jchowdhary/k8IngressWithCerts/blob/master/letsencryptCertWithIngress/ngrokhost_ingress.jpg](https://github.com/jchowdhary/k8IngressWithCerts/blob/master/letsencryptCertWithIngress/ngrokhost_ingress.jpg)

NB: We see that as per my NGROK capture , host name is coming as <i>fbe0de55e3bf.ngrok.io</i>. But this will be different when you execute ngrok.We also see that we have given a random secret name <i>fleetman-webapp-secret</i>.But at this point, there is no certificate and secret created.

## Step 8: Launch ClusterIssuer from CertManager.
Execute, `kubectl apply -f fleetman-clusterissuer-certmanager.yaml`. This will automatically provision Let's Encrypt Staging certificate and also kubernetes secret as well.
If you describe your certificate you will see `fleetman-webapp-secret` certificate is created and also a Kubernetes Secret with the same name.

In my lab,when i describe my certificate and secret which are automatically provisioned,i get the below screenshot

![https://github.com/jchowdhary/k8IngressWithCerts/blob/master/letsencryptCertWithIngress/cert1.jpg](https://github.com/jchowdhary/k8IngressWithCerts/blob/master/letsencryptCertWithIngress/cert1.jpg)

![https://github.com/jchowdhary/k8IngressWithCerts/blob/master/letsencryptCertWithIngress/cert2.jpg](https://github.com/jchowdhary/k8IngressWithCerts/blob/master/letsencryptCertWithIngress/cert2.jpg)

# Step 9. Launch your CA signed HTTPS application in the browser.
Launch the browser and open the url and type the IP which is genereated when we executed the ngrok command. Or, you can find the external IP from your secret as well..In our case we are launching the browser with `https://05408122f22a.ngrok.io`

![https://github.com/jchowdhary/k8IngressWithCerts/blob/master/letsencryptCertWithIngress/kubernetesApp.JPG](https://github.com/jchowdhary/k8IngressWithCerts/blob/master/letsencryptCertWithIngress/kubernetesApp.JPG)

In this example, we have seen how we can atutomatically provision a CA trusted signed certificate in both MiniKube and Cloud World.{<i>In Cloud World,we do not use NGROK,we reserve the IP generated by Kubernetes Ingress.</i>}

In the upcoming section,we will see how we can use authentication with Istio Service Mesh


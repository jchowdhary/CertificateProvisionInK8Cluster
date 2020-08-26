# k8IngressWithCerts
In this repo, I will describe how we can use certificates with Kubernetes Ingress. Here,I will describe the below things
1. How we can migrate our K8 application deployment from HTTP to HTTPS using a self signed certificate.
2. How we can auto renew the expired self signed certificates in K8 deployment without downtime.
3. How we can auto renew CA root certificate in K8 Deployment using Cert-Manager without downtime.
4. How we can leverage certificates from Cloud Vendor.

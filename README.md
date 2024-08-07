# Gateway API Demo

## Prerequisites

* Install cert-manager helm chart (optional)
* Install external dns helm chart (optional)
* Kubernetes or Cloud provider Load Balancer Service 

* Clone this repo

```
git clone https://github.com/syseleven/gateway-api-demo.git
```

* Install Gateway API CRDs

```
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.1.0/experimental-install.yaml
```

* Install the cilium helm chart. This is the minimum configuration. You can also use the cilium values file in this repo (optional)

```
helm upgrade cilium cilium/cilium \
    --namespace kube-system \
    --reuse-values \
    --set kubeProxyReplacement=true \
    --set gatewayAPI.enabled=true
```

* Install the Cert-Manager helm chart (optional). Make sure to create an issuer or a clusterissuer after the installation. [Cert-Manager Docs]( https://cert-manager.io/docs/configuration/acme) 

```
helm upgrade --install cert-manager --namespace cert-manager --create-namespace -f cert-manager-helm/cert-manager-values.yaml jetstack/cert-manager
```

* Install the External DNS helm chart (optional). Make sure to configure your DNS API key in the external-dns-values.yaml first. [External-DNS Docs](https://kubernetes-sigs.github.io/external-dns/#the-latest-release
)

```
helm upgrade --install external-dns --namespace external-dns  --create-namespace -f external-dns/external-dns-values.yaml bitnami/external-dns
```
 
* Install the Gateway API controller. Remove the HTTPS section if you are only using HTTP routes. Change the domain, certificate and clusterissuer if you are using HTTPS. You will need a Load Balancer Service for this to succeed 
 
```
kubectl create -f gateway-api/gateway.yaml
```

* Wait for the Load Balancer IP to be ready
* Create the demo web app v1

```
kubectl create -f webapp/webapp-v1.deploy.yaml
```

* Create an HTTP route. Change the domain name first.

```
kubectl create -f gateway-api/http-route.yaml
```

* Test if the route is working. If you are using external dns wait a few minutes for the domain name to get added to the dns server

```
curl http://example.com
```

* Update the HTTP route with a redirect and a HTTPS route

```
kubectl apply -f gateway-api/https-redirect.yaml
```

* Add the second demo web app v2

```
kubectl create -f webapp/webapp-v2.deploy.yaml
```

* Add a second service and traffic spliting to the existing routes

```
kubectl apply -f gateway-api/https-redirect-split.yaml
```

* Test both web apps and ensure that the traffic spliting is working (green / red app)

```
open http://example.com
```
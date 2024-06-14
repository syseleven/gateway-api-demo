# Gateway API Demo

## Prerequisites

* Install cert-manager helm chart (optional)
* Install external dns helm chart (optional)
* Kubernetes or Cloud provider Load Balancer Service 
* Install Gateway API CRDs

```kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/gateway-api/v1.0.0/config/crd/standard/gateway.networking.k8s.io_gatewayclasses.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/gateway-api/v1.0.0/config/crd/standard/gateway.networking.k8s.io_gateways.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/gateway-api/v1.0.0/config/crd/standard/gateway.networking.k8s.io_httproutes.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/gateway-api/v1.0.0/config/crd/standard/gateway.networking.k8s.io_referencegrants.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/gateway-api/v1.0.0/config/crd/experimental/gateway.networking.k8s.io_grpcroutes.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/gateway-api/v1.0.0/config/crd/experimental/gateway.networking.k8s.io_tlsroutes.yaml
```

* Install cilium helm chart. This is the minimum configuration

```
helm upgrade cilium cilium/cilium \
    --namespace kube-system \
    --reuse-values \
    --set kubeProxyReplacement=true \
    --set gatewayAPI.enabled=true
```

* You can also use the cilium values file in this repo (optional)
* Clone this repo
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
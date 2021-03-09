# Create k8s Certificates With Lets Encrypt

Lets create a Kubernetes cluster to play with using kind
```
kind create cluster --name certmanager --image kindest/node:v1.19.1
```



Install cert-manager
```
curl -LO https://github.com/jetstack/cert-manager/releases/download/v1.0.4/cert-manager.yaml
mv cert-manager.yaml cert-manager-1.0.4.yaml
kubectl create ns cert-manager
kubectl apply --validate=false -f cert-manager-1.0.4.yaml
```

Let's deploy an Ingress controller:
```
kubectl create ns ingress-nginx

kubectl -n ingress-nginx apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.41.2/deploy/static/provider/cloud/deploy.yaml

kubectl -n ingress-nginx get pods

# When you run locally
kubectl -n ingress-nginx --address 0.0.0.0 port-forward svc/ingress-nginx-controller 80
kubectl -n ingress-nginx --address 0.0.0.0 port-forward svc/ingress-nginx-controller 443
```
We should be able to access NGINX in the browser and see a 404 Not Found page: http://localhost/ This indicates there are no routes to / and the ingress controller is running


Deploy the issuer
```
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-cluster-issuer
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: your-email@email.com
    privateKeySecretRef:
      name: letsencrypt-cluster-issuer-key
    solvers:
    - http01:
       ingress:
         class: nginx
         
         
# check the issuer
kubectl describe clusterissuer letsencrypt-cluster-issuer
> status ready
```



Deploy Ingress (but first the certificate)
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: "nginx"
  name: example-app
spec:
  tls:
  - hosts:
    - marcel.guru
    secretName: example-app-tls  # secret that will be used (must be created, see below)
  rules:
  - host: marcel.guru  # your dns
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service: 
            name: example-service
            port: 
              number: 80
```



Deploy certificate template
```
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: example-app
  namespace: default  #same as your app
spec:
  dnsNames:
    - marcel.guru   # your dns
  secretName: example-app-tls  # secret will be created
  issuerRef:
    name: letsencrypt-cluster-issuer
    kind: ClusterIssuer
    
    
    
# check the cert has been issued 
kubectl describe certificate example-app
```

```
Now a new secret was created automatically: Type: kubernetes.io/tls name ...-tls
```



## Renew A Certificate

```
kubectl get orders
# delete the order

kubectl get secrets
# delete the tls secret


```





















# What is the Ingress?

The Ingress is a Kubernetes resource that lets you configure an HTTP load balancer for applications running on Kubernetes, represented by one or more [Services](https://kubernetes.io/docs/concepts/services-networking/service/). Such a load balancer is necessary to deliver those applications to clients outside of the Kubernetes cluster.

The Ingress resource supports the following features:
* **Content-based routing**:
    * *Host-based routing*. For example, routing requests with the host header `foo.example.com` to one group of services and the host header `bar.example.com` to another group.
    * *Path-based routing*. For example, routing requests with the URI that starts with `/serviceA` to service A and requests with the URI that starts with `/serviceB` to service B.

See the [Ingress User Guide](http://kubernetes.io/docs/user-guide/ingress/) to learn more about the Ingress resource.

## What is the Ingress Controller?

The Ingress controller is an application that runs in a cluster and configures an HTTP load balancer according to Ingress resources. The load balancer can be a software load balancer running in the cluster or a hardware or cloud load balancer running externally. Different load balancers require different Ingress controller implementations. 

In the case of NGINX, the Ingress controller is deployed in a pod along with the load balancer.


# Installing the Ingress Controller In AWS 

## 1. Clone Kubernetes Nginx Ingress Manifests into server where you have kubectl

```
$ git clone https://github.com/MithunTechnologiesDevOps/kubernetes-ingress.git

$ cd kubernetes-ingress/deployments
```
## 2. Create a Namespace And SA

```
 $ kubectl apply -f common/ns-and-sa.yaml
```
## 3. Create RBAC, Default Secret And Config Map

```
 $ kubectl apply -f common/
```

## 4. Deploy the Ingress Controller

We include two options for deploying the Ingress controller:
 * *Deployment*. Use a Deployment if you plan to dynamically change the number of Ingress controller replicas.
 * *DaemonSet*. Use a DaemonSet for deploying the Ingress controller on every node or a subset of nodes.

### 4.1 Create a DaemonSet

When you run the Ingress Controller by using a DaemonSet, Kubernetes will create an Ingress controller pod on every node of the cluster.

```
 $ kubectl apply -f daemon-set/nginx-ingress.yaml
 ```

## 5. Check that the Ingress Controller is Running

Check that the Ingress Controller is Running
Run the following command to make sure that the Ingress controller pods are running:
```
$ kubectl get pods --namespace=nginx-ingress
```

## 6. Get Access to the Ingress Controller

 **If you created a daemonset**, ports 80 and 443 of the Ingress controller container are mapped to the same ports of the node where the container is running. To access the Ingress controller, use those ports and an IP address of any node of the cluster where the Ingress controller is running.


### 6.1 Service with the Type LoadBalancer

 Create a service with the type **LoadBalancer**. Kubernetes will allocate and configure a cloud load balancer for load balancing the Ingress controller pods.

**For AWS, run:**
```
$ kubectl apply -f service/loadbalancer-aws-elb.yaml
```

To get the DNS name of the ELB, run:
```
$ kubectl describe svc nginx-ingress --namespace=nginx-ingress
```

`OR`

```
kubectl get svc -n nginx-ingress 
```

You can resolve the DNS name into an IP address using `nslookup`:
```
$ nslookup <dns-name>
```


# 7. Ingress Resource:

### 5.1 Define path based or host based routing rules for your services.

### Single DNS Sample with host and servcie place holders
``` yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: <name>
  namespace: <namespace>
spec:
  ingressClassName: nginx
  rules:
  - host: <domainName>
    http:
      paths:
      - pathType: Prefix
        path: "/<Path>"
        backend:
          service:
            name: <serviceName>
            port:
              number: <servicePort>
``` 

### Multiple DNS Sample with hosts and servcies place holders
``` yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: <name>
  namespace: <namespace>
spec:
  ingressClassName: nginx
  rules:
  - host: <domainName>
    http:
      paths:
      - pathType: Prefix
        path: "/<Path>"
        backend:
          service:
            name: <serviceName>
            port:
              number: <servicePort>
  - host: <domainName>
    http:
      paths:
      - pathType: Prefix
        path: "/<Path>"
        backend:
          service:
            name: <serviceName>
            port:
              number: <servicePort>	
``` 		  

### Path Based Routing Example
``` yaml		  
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: <name>
  namespace: <nsname>
spec:
  ingressClassName: nginx
  rules:
  - host: <domain>
    http:
      paths:
      - pathType: Prefix
        path: "/<Path>"
        backend:
          service:
            name: <serviceName>
            port:
             number: <servicePort>
      - pathType: Prefix
        path: "/<path>"
        backend:
          service:
            name: <servcieName>
            port:
              number: <servicePort>
``` 


`Make sure you have services created in K8's with type ClusterIP for your applications. Which your are defining in Ingress Resource`.

## Uninstall the Ingress Controller

 Delete the `nginx-ingress` namespace to uninstall the Ingress controller along with all the auxiliary resources that were created:
 ```
 $ kubectl delete namespace nginx-ingress
 ```

 **Note**: If RBAC is enabled on your cluster and you completed step 2, you will need to remove the ClusterRole and ClusterRoleBinding created in that step:

 ```
 $ kubectl delete clusterrole nginx-ingress
 $ kubectl delete clusterrolebinding nginx-ingress
 ```

## Ingress with Https Using Self Signed Certificates:

### Generate self signed certificates
```
$ openssl req -x509 -nodes -days 365 -newkey rsa:2048 -out mithun-ingress-tls.crt -keyout mithun-ingress-tls.key -subj "/CN=javawebapp.mithuntechdevops.co.in/O=mithun-ingress-tls"

# Create secret for with your certificate .key & .crt file

$ kubectl create secret tls mithun-ingress-tls --namespace default --key mithun-ingress-tls.key --cert mithun-ingress-tls.crt
```
### Mention tls/ssl(certificate) details in ingress
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: mithuntechappingressrule
  namespace: test-ns
spec:
  ingressClassName: nginx
  tls:
  - hosts:
      - mithuntechdevops.co.in
    secretName: mithun-ingress-tls
  rules:
  - host: mithuntechdevops.co.in
    http:
      paths:
      - pathType: Prefix
        path: "/java-web-app"
        backend:
          service:
            name: javawebappsvc
            port:
              number: 80
      - pathType: Prefix
        path: "/maven-web-application"
        backend:
          service:
            name: mavenwebappsvc
            port:
              number: 80
```

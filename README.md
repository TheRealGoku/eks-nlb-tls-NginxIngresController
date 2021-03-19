# TLS Termination on NLB for EKS Nginx ingress controller

## Advantages:

1. You could use the certificates which are present in ACM / IAM. No accidental certificate key exposure at kubernetes / worker node level.

2. NLB will do the heavy lifting of [TLS Termination](https://aws.amazon.com/blogs/aws/new-tls-termination-for-network-load-balancers/
), Improved performance for worker nodes.

3. HTTP to HTTPS redirection at nginx ingress controller as the controller will be participating in dataplane traffic routing.

---
### Steps:

1. [Download](https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.44.0/deploy/static/provider/aws/deploy-tls-termination.yaml) the lastest nginx ingress deployment manifest file. Please note that at the time of writing this article, the controller version which I have used is v0.44.0


2. Under the Annotation of "# Source: ingress-nginx/templates/controller-service.yaml" Replace the all existing annotation content in that section with the following annotation.
```
   service.beta.kubernetes.io/aws-load-balancer-backend-protocol: "http"
   service.beta.kubernetes.io/aws-load-balancer-ssl-cert: "xxxxxxxARN\_\_HERExxxxxx"
   service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
   service.beta.kubernetes.io/aws-load-balancer-ssl-ports: "443"
```
3. Configure appropriate Certificate ARN on  ```service.beta.kubernetes.io/aws-load-balancer-ssl-cert``` field.

4. Now under the ```# Source: ingress-nginx/templates/controller-deployment.yaml``` section in the downloaded manifest, change the kind from Deployment to DaemonSet to avoid any SPOF

5. Search for "proxy-real-ip-cidr" in the manifest and remove that line or configure appropriate value incase if you require the same.

6. Now the manifest is ready to get deployed to spin up Nginx Ingress controller, use ```kubectl apply -f modified-file-name.yaml ```

### Note:

1. In addition to the above steps, I have attached the complete modified version of the configuration yaml file which I have used in my environment, In the attached file, The certificate ARN content has been omitted intentionally, Please feel free to use the file after configuring the ARN value.

2. A diff check screenshot on what has been changed to a existing manifest which has been downloaded also attached for future reference.

    Once, the modified manifest file applied, you could proceed further with creating ingress object in the respective namespaces to associate with your services.

---
### More Information:

1. Network Loadbalancer with nginx ingress controller - https://aws.amazon.com/blogs/opensource/network-load-balancer-nginx-ingress-controller-eks/
2. Discussion on Enabling TLS offloading at NLB with type Loadbalancer - https://github.com/kubernetes/kubernetes/issues/73297
3. Ingres-nginx - https://kubernetes.github.io/ingress-nginx/deploy/#aws

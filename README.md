# Simple Python Flask Multi-Page Dockerized Image Application

Build the image using the following command

```bash
$ docker build -t image-app:latest .
```

Run the Docker container using the command shown below.

```bash
$ docker run -d -p 5000:5000 image-app
```

The application will be accessible at http:127.0.0.1:5000 or if you are using boot2docker then first find ip address using `$ boot2docker ip` and the use the ip `http://<host_ip>:5000`

Application should be running on Port 5000



For Ingress Controller:
1. First apply ingress.yaml file to create ingress resource.
2. Next step would be to deploy/install ingress controller. I used following commands to install ingress controller on GCE-GKE cluster:
   a. First, your user needs to have cluster-admin permissions on the cluster. This can be done with the following command:
       kubectl create clusterrolebinding cluster-admin-binding \
  --clusterrole cluster-admin \
  --user $(gcloud config get-value account)
    b. Then, the ingress controller can be installed like this:
         kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.12.0/deploy/static/provider/cloud/deploy.yaml
3. Note: when I tried to access the log of the pod created in 2.b above(while creating ingress resource) using following command:
   kubectl get pods -A | grep nginx
   kubectl logs podName -n ingress-nginx| grep example
   above cmd logs gave me following error:
   Kubernetes IngressClass | error="ingress does not contain a valid IngressClass"

   In order to resolve above error, I used  following steps:
   Step 1 — Create IngressClass resource
     First we have to create IngressClass ingress resource. (Check if there is any kubectl get ingressclass -A -o yaml )
       apiVersion: networking.k8s.io/v1
       kind: IngressClass
       metadata:
         name: nginx
       spec:
	        # use your controller name.
         controller: k8s.io/ingress-nginx
   Step 2 — Create/Update Ingress
      Update ingress resource spec.ingressClassName with ingress class name nginx. This should resolve the problem.
       apiVersion: networking.k8s.io/v1
       kind: Ingress
       metadata:
        name: minimal-ingress
        annotations:
            nginx.ingress.kubernetes.io/rewrite-target: /
        spec:
           ingressClassName: nginx
           rules:
           - http:
               paths:
               - path: /testpath
               pathType: Prefix
               backend:
                 service:
                   name: test
                   port:
                     number: 80
   4. Add address from 'kubectl get ingress -n your-namespace' at the following path file to match it to the foo.bar.com mentioned in ingress.yaml file. eg. I added: 34.69.185.52 foo.bar.com at the end of the following file path:
        sudo nano /etc/hosts
      Note: This step is only necessary if you do not have real https path like in production. Since I am only doing this for practice using http so I had to fake this mapping.
    5. after step for you can use curl or ping at foo.bar.com like:
         ping foo.bar.com
         curl -L foo.bar.com/bar -v


     Commands to delete all the ingress-controller resources:
	kubectl get all -n ingress-nginx
	kubectl delete all --all -n ingress-nginx
	kubectl get all -n ingress-nginx
	kubectl get ingressClass
	kubectl get ingressClass -A
	kubectl delete ingressClass nginx
	kubectl get ingressClass -A
	kubectl get ValidatingWebhookConfiguration
	kubectl delete ValidatingWebhookConfiguration ingress-nginx-admission
	kubectl get ValidatingWebhookConfiguration
	kubectl get all
	kubectl get all -n yourNamespace
	kubectl get ingress -A

   	 
   
       

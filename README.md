# kubernetes-workshop
Labs for K8s workshops

Download the repo:
git clone https://github.com/sashajeltuhin/kubernetes-workshop.git

Navigate to the root folder of the repo.

On Mac:
  chmod 600 ssh/cluster.pem
  run ./provisionket do create

On Windows:
  give full permissions to the current user for ssh/cluster.pem writable 
  open GitBash
  run ./provision64win do create
  
Provide Digital Ocean API token when prompted  
  
After the provisionioning is done, ssh into the bootstrap node and navigate to folder /ket


chmod 600 kubernetes-workshop/ssh/cluster.pem
change the path to ssh file in  /ket/kismatic-cluster.yaml
./kismatic install apply -f kismatic-cluster.yaml


Create backend service:
kubectl apply -f backend

Create api service:
kubectl apply -f api

Create web service:
kubectl apply -f web


Create ingress resource:
kubectl apply -f ingress.yaml




Test API:
curl <Ip of worker1>:<NodePort of geo-api-svc>/api/testMap/32.983312,-84.343748%3B33.983311,-84.333732


Test website:
Add record to /etc/hosts:
<Ip of worker1>	myk8sworkshop.com

Open the browser, navigate to http://myk8sworkshop.com

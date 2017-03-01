# kubernetes-workshop
Labs for K8s workshops

Download the repo:
git clone https://github.com/sashajeltuhin/kubernetes-workshop.git

Navigate to the root folder of the repo.

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
138.197.148.37:32447

Test website:
Add record to /etc/hosts:
<Ip of worker1>	myk8sworkshop.com

Open the browser, navigate to http://myk8sworkshop.com
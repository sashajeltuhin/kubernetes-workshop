# Kubernetes Workshop

We run this lab on Digital Ocean. If you do not have an account with Digital Ocean, go to [this page] (https://www.digitalocean.com) and click on Create Account at the bottom of the screen. During the process, you will be prompted to
provide a credit card. Do not worry, the lab will not cost you a penny. Digital Ocean offers a $10 credit to the new users.
This amount is more than sufficient for this lab and beyond. Make sure you pick the account without any resource limitations.
After you opened the account click on the API tab and generate a new access token. Save the token string in a local file for future use: you will need it a couple of times during the lab.

## Provisioning and prep work

Download this repo:
```git clone https://github.com/sashajeltuhin/kubernetes-workshop.git```

On Mac:
  * Navigate to the root folder of the repo
  * chmod 600 ssh/cluster.pem
  * run ./provision do create

On Windows:
  * give full permissions to the current user for ssh/cluster.pem writable 
  * download GitBash from [here](https://git-scm.com/downloads)
  * open GitBash
  * Navigate to the root folder of the repo
  * run ./provision64win do create
  

Provide Digital Ocean API token that you saved earlier, when prompted.

By default, the provisioner will create 4 VMs: 1 etcd, 1 master, 1 worker and 1 bootstrap node, which we will use to run the lab from.
At the very end of the provisioning process, we prepare the bootstrap node for you to start the orchestration of the kubernetes cluster. Here is the list of commands that we run for your reference, in case you need to re-run them manually:

```mkdir -p /ket &&
cd /ket &&
sudo apt-get update -y &&
wget --no-check-certificate -O - https://github.com/apprenda/kismatic/releases/download/v1.2.1/kismatic-v1.2.1-linux-amd64.tar.gz | tar -zx && 
sudo apt-get -y install git build-essential &&
sudo apt-get install -qq python2.7 && ln -s /usr/bin/python2.7 /usr/bin/python &&
git clone https://github.com/sashajeltuhin/kubernetes-workshop.git &&
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl &&
chmod +x ./kubectl &&
sudo mv ./kubectl /usr/local/bin/kubectl```


## Orchestrate Kubernetes  
After the provisionioning is complete, ssh into the bootstrap node and navigate to folder `/ket`.
Run `chmod 600 kubernetes-workshop/ssh/cluster.pem`

To standup a Kubernetes cluster, we use a set of ansible scripts driven by a plan file. The file is generated for you during the provisioning process.
* Open `/ket/kismatic-cluster.yaml for editing.
* Inspect the generated content and change the path to the ssh file to `/ket/kubernetes-workshop/ssh/cluster.pem`
* Save and close the plan file
* Run the following command:
`./kismatic install apply -f kismatic-cluster.yaml`
It takes about 5 minutes to produce a cluster of this configuration.

## Start using `kubectl`. During the cluster provisioning we generated a configuration file that is required for us to use **kubectl**. We need to copy it to the location where **kubectl** expects it.
* run `makedir -p ~/.kube`
* run `cp generated/kubeconfig ~/.kube/config

Now we can communicate with kubernetes:

* Check the cluster info
`kubectl cluster-info`

* Inspect the nodes
`kubectl get nodes`

## Application deployment


To change the region, redeploy the osrm-api pod with reference to the corresponding *.pbf file. Refer to
[GeoFabrik pages](http://download.geofabrik.de/index.html) for the complete list of supported regions

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

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
  * `chmod 600 ssh/cluster.pem`
  * run `./provision do create`

On Windows:
  * give full permissions to the current user for ssh/cluster.pem writable 
  * download GitBash from [here](https://git-scm.com/downloads)
  * open GitBash
  * Navigate to the root folder of the repo
  * run `./provision64win do create`
  

Provide Digital Ocean API token that you saved earlier, when prompted.

By default, the provisioner will create 4 VMs: 1 etcd, 1 master, 1 worker and 1 bootstrap node, which we will use to run the lab from.
At the very end of the provisioning process, we prepare the bootstrap node for you to start the orchestration of the kubernetes cluster. Refer to [this script file] (https://github.com/sashajeltuhin/kubernetes-workshop/blob/master/digitalocean/scripts/bootinit.sh) for the list of commands that we run in the bootstrap node


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

## Start using `kubectl`. 
During the cluster provisioning we generated a configuration file that is required for us to use **kubectl**. We need to copy it to the location where **kubectl** expects it.
* run `makedir -p ~/.kube`
* run `cp generated/kubeconfig ~/.kube/config`

Now we can communicate with kubernetes:

* Check the cluster info
`kubectl cluster-info`

* Inspect the nodes
`kubectl get nodes`

## Application deployment

The application provided in this repo has 3 components. The lab demonstrates how these components can communicate with each other via **kubernetes services** and how end users can access the application via kubernetes **ingress**.

### Overview
![Multi Pod App](https://github.com/sashajeltuhin/kubernetes-workshop/blob/master/app.png "Multi Pod App")

To deploy the app, you can run the files individually or by folder.
Run `cd kubernetes-workshop`

To deploy backend osrm-api component:
`kubectl apply -f backend`

To deploy geoapi component:
`kubectl apply -f api`

To deploy geoweb component:
`kubectl apply -f web`


To enable external access to the app, create ingress resource:
`kubectl apply -f ingress.yaml`


You can delete the deployment and redeploy with `--record` flag. This will enable version history.
Run `kubectl rollout history deployment/<name of deployment>` to view the revisions and reasons for the updates.

To test deployment, open `backend/osrm-api.yaml` and change the last command parameter, which is the url to regional geo data file (pbf). The default is US Georgia. You can locate URLs to other desired regions at [GeoFabrik site] (http://download.geofabrik.de)
After you change the file, run
`kubectl apply -f backend`

Alternatively to change deployment and cause the rollout, run:
`kubectl edit deployment osrm-api`, make the change to the command parameter and save. This will initiate a rollout update for the deployment.


Test API:
curl <Ip of worker1>:<NodePort of geo-api-svc>/api/testMap/32.983312,-84.343748%3B33.983311,-84.333732


Test website:
Add record to /etc/hosts:
<Ip of worker1>	myk8sworkshop.com

Open the browser, navigate to http://myk8sworkshop.com

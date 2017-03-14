# Kubernetes Workshop

We run this lab on Digital Ocean. If you do not have an account with Digital Ocean, go to [this page](https://www.digitalocean.com) and click on Create Account at the bottom of the screen. During the process, you will be prompted to
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
  * give full permissions to the current user for ssh/cluster.pem
  * download GitBash using one of 2 options
    * from [here](https://git-scm.com/downloads)
    * using powershell as Administrator:
      * Run `Set-ExecutionPolicy unrestricted`
      * check version: `$PSVersionTable.PSVersion`
        * for v2, run: 
        `$ iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))`
        * for v3+, run 
        `$ iwr https://chocolatey.org/install.ps1 -UseBasicParsing | iex`
      * choco install git.install -y
  * open GitBash
  * Navigate to the root folder of the repo
  * run `./provision64win do create`
  

Provide Digital Ocean API token that you saved earlier, when prompted.

By default, the provisioner will create 4 VMs: 1 etcd, 1 master, 1 worker and 1 bootstrap node, which we will use to run the lab from.
At the very end of the provisioning process, we prepare the bootstrap node for you to start the orchestration of the kubernetes cluster. Refer to [this script file](https://github.com/sashajeltuhin/kubernetes-workshop/blob/master/digitalocean/scripts/bootinit.sh) for the list of commands that we run in the bootstrap node


## Orchestrate Kubernetes  
After the provisionioning is complete, ssh into the bootstrap node and navigate to folder `/ket`.

Run `chmod 600 kubernetes-workshop/ssh/cluster.pem`

To standup a Kubernetes cluster, we use a set of ansible scripts driven by a plan file. The file is generated for you during the provisioning process.
* Open `/ket/kismatic-cluster.yaml` for editing.
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

* The **geoweb** component is an Angular2 app running on nginx.
* The **geoapi** component is a NodeJS service that communicates to various external services: Google
geolocation service and the backend service of this application - **osrm-api**.
* The **osrm-api** component is a routing engine for OpenStreetMaps project, see [this project](https://github.com/Project-OSRM/osrm-backend). It is implemented in C++. Among other things, it provides shortest routes between 2+ geographical locations.

### Deployment

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

To check the readiness of the application look up the deployment status or inspect the pods:

`kubectl rollout status deployment/geoweb`

`kubectl get pods`

When the deployment is complete from the Kubernetes point of view, the output will look similar to this:

`kubectl rollout status deployment/geoweb`

```
	deployment "geoweb" successfully rolled out
```

`kubectl get pods`
``` 
	geoapi-662170153-jnv66      1/1       Running   0          1h
	geoweb-1477820572-r6558     1/1       Running   0          1h
	osrm-api-3969967481-wcp1q   1/1       Running   0          25m
```


You can delete the deployment and redeploy with `--record` flag. This will enable version history.
Run `kubectl rollout history deployment/<name of deployment>` to view the revisions and reasons for the updates.

#### Accessing the deployed application
You can access the app using the NodePort of the service.
Look up the geoweb service:

`kubectl get svc geoweb`

You will get an output similar to this:

`geoweb    172.17.30.138   <nodes>       80:31587/TCP   1h`

**31587** is the port on the host that can be used to access the service.

Open your browser and navigate to `http://<Public IP of Worker1>:<NodePort of geoweb service>`



Alternatively, you can use a more user-friendly method - *ingress*.

Add record to /etc/hosts:
`<Public IP of worker1>	  myk8sworkshop.com`
Open the browser, navigate to http://myk8sworkshop.com


#### Scale the deployment

To scale a deployment you can use at least 2 following options.

Let's increase the number of *geoweb* pods

`kubectl scale deployments geoweb --replicas 2`

Alternatively you can change the deployment yaml file:

`kubectl edit deployment/geoweb`, which will open the yaml of the pod in the preferred editor. After saving the changes, the deployment will be automatically updated.
To invoke the preferred editor, you can change the default setting like this:

`KUBE_EDITOR="nano" kubectl edit deployment/geoweb`

Check the rollout history:

`kubectl rollout history deployment/geoweb`

The number of revisions should remain unchanged, as scaling is not considered to be an upgrade.

#### Modify the deployment. Lab 1
To force a new rollout revision, a part of the pod spec has to be changed.
In this lab we will add environmental variables to the **geoapi** deployment. These variables will
control some of the text on the web page. Modify the **env** section of the spec in `api/geo-api.yaml` as follows:

```
env:
        - name:  'GEO-KEY'
          value:  AIzaSyDZszKJ0a72ED1ragcd0s3Eks3QI2wc6-I
        - name:  'GEO_TITLE'
          value:  Your preferred page title
        - name:  'GEO_PLACEHOLDER'
          value:  Your preferred prompt

```

After you make the change, run
`kubectl apply -f geoapi`


Alternatively to change deployment and cause the rollout, run:
`kubectl edit deployment/geo-api`, make the change and save. This will initiate a rollout update for the deployment.

Refresh the browser. Your new labels should appear on the first page of the web app.


#### Modify the deployment. Lab 2 

To force a new rollout revision, a part of the pod spec has to be changed. We can for example switch the routing engine to use a different region.
 Open `backend/osrm-api.yaml` and change the last command parameter, which is the url to regional
geo data file (pbf). The current image points to Florida. You can locate URLs to other desired regions at [GeoFabrik site](http://download.geofabrik.de).
Some examples:

`Georgia: "http://download.geofabrik.de/north-america/us/georgia-latest.osm.pbf"`

`Berlin: "http://download.geofabrik.de/europe/germany/berlin-latest.osm.pbf" `

`Florida: "http://download.geofabrik.de/north-america/us/florida-latest.osm.pbf"`


After you change the file, run
`kubectl apply -f backend`


Alternatively to change deployment and cause the rollout, run:
`kubectl edit deployment/osrm-api`, make the change to the command parameter and save. This will initiate a rollout update for the deployment.

#### Notes on Readiness 

As soon as the *geoapi* and *geoweb* deployments are declared by Kuberenetes as *Ready*, it becomes possible for end users to interact with some parts of the deployed application. For example, the user
can look up places of interest and see their locations on the map.
However, the backend component takes a long time to initialize. Depending on the volume of geo data it needs to process, it may take up to 15-20 minutes. 
The osrm-api pod will appear as Ready, but when the app tries to communicate with the pod, it receives the following error. This will happen when the user clicks on the Route button to map the shortest way between the selected points.

`500 - Internal Server Error Unable to look up location. Error: connect ECONNREFUSED 172.17.102.229:5000`

We can use the *ReadinessProbe* section of the pod spec to help Kubernetes understand when the pod is actually ready to receive traffic.
In our case the probe will be defined as follows:

``` 
        readinessProbe:
          httpGet:
            path: /route/v1/driving/-82.6609185%2C27.8859695%3B-82.5370781%2C27.9834776?steps=true
            port: 5000
          initialDelaySeconds: 30
          timeoutSeconds: 10
```

Edit the osrm-api deployment and add the ReadinessProbe block under the name property of the container:

`kubectl edit deployment/osrm-api`

```
      containers:
      - name: osrm-api
        readinessProbe:
          httpGet:
            path: /route/v1/driving/-82.6609185%2C27.8859695%3B-82.5370781%2C27.9834776?steps=true
            port: 5000
          initialDelaySeconds: 30
          timeoutSeconds: 10
```


Now, until the osrm-api initialization is truly complete, the pod will show the Not-Ready state, the deployment will show as incomplete and any attempts to reach the service will result in a different error, not an internal crash.

`500 - Internal Server Error Unable to look up location. Error: connect EHOSTUNREACH 172.17.102.229:5000`




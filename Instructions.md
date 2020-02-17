# Flux Installation and Implementation

#### Pre-Requisites:
Fork or make a copy of the Demonstration Repository : `.../flux-demo`

Steps required:

* Install K8s (self explanatory)
* Install and configure fluxctl (for Flux deployment) - not using Helm
* Configure Flux to connect to Git Repo
* Update deployment mainifest in Git Repo
* Update Container image and Sync
* Configuration drift and Sync

## Install Flux
Install Fluxctl - https://docs.fluxcd.io/en/1.18.0/references/fluxctl.html

Install Fluxcd https://docs.fluxcd.io/en/1.18.0/tutorials/get-started.html

## Confiugre Flux for Repo

* Create a Namespace
```
kubectl create ns <namespace>

export FLUX_FORWARD_NAMESPACE=<namespace>
fluxctl list-workloads
```

* Install Fluxcd to create connection to your Git Repo
```
export GHUSER="<username>"
export REPO="gitops-demo"
export NS="flux"
fluxctl install \
--git-user=${GHUSER} \
--git-email=${GHUSER}@users.noreply.github.com \
--git-url=git@github.com:${GHUSER}/${REPO} \
--namespace=${NS} | kubectl apply -f -
```

## Create SSH key for adding to your GitHub repository
```
fluxctl identity
```
Open GitHub, navigate to the Repository added when you installed Fluxcd, go to Setting > Deploy keys, click on Add deploy key, give it a Title, check Allow write access, paste the public key and click Add key.

## Update deployment mainifest in Git Repo

Open up your GitRepo that has the `deployment.yaml` file
Scroll down to the section as below and change the APP_VERSION number
```
        env:
        - name: APP_NAME
          value: myfluxdemo.K8s.GitOps
        - name: APP_VERSION
          value: v1.0.5
```
Save and Committ the change to your Repo
Flux will (within 5 minutes) update your Deployment

To test from a localhost we can use the "Port-forward" command within Kubernetes:

```
kubectl get pods -n <namespace>
copy the pod name
kubectl port-forward <podname> 8080:80 -n <namespace>
```
Open another terminal:
```
curl -s -i http://localhost:8080
```

## Update Container image and Sync

Let's now make a modification to the Docker image and upload to our Docker hub repository.  For this we will modify the `main.py` file within the `flaskapp` directory.  
* Update the `main.py` file - change `Hello World` to something else
```
    response = "%s - %s.%s\n" %('Flux World', appname, appversion)
```
* Create a new Docker file and upload to Docker (with another incremental version number)
* Wait 5 minutes - Flux will now automatically deploy new Image

## Configuration drift and Sync

Now lets test what happens when there is a manual change to the running configuration

```
kubectl scale deployment/fluxdemo --replicas=4 -n <namespace>
```

Now let's watch the pods for a few minutes and see what happens.  We will see the additional pods for a period of time, but within 5 minutes we will see the excess number of pods terminate.  Therefore, flux has brought the configuration back to the declared state of deployment currently held within Git.

```
kubectl get po -n <namespace> --watch
``` 

Re-running the same command again you can see that this view has now filtered down to the just one running pod. 

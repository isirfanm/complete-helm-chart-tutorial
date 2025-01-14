# Complete Helm Chart Tutorial: From Beginner to Expert Guide

From YouTube video: https://youtu.be/DQk8HOVlumI?si=9OgkBIptXUOihSSQ

## Environment

Operating System: Ubuntu 24.04

## Setup

### Install microk8s

Install microk8s using snap.

```bash
sudo snap install microk8s --classic
```

MicroK8s creates a group to enable seamless usage of commands which require admin privilege. To add your current user to the group and gain access to the `.kube` caching directory, run the following three commands: 

```bash
sudo usermod -a -G microk8s $USER
mkdir -p ~/.kube
chmod 0700 ~/.kube
```

You will also need to re-enter the session for the group update to take place:

```bash
su - $USER
```

Check microk8s cluster status.

```bash
microk8s status --wait-ready
```

We can use `microk8s kubectl` to access cluster.

```bash
microk8s kubectl get nodes
```

Optionally, we can create bash alias on `~/.bash_aliases`.

```bash
alias kubectl='microk8s kubectl'
```

Another option is to use install and setup `kubectl` CLI. 

```bash
# install kubectl
sudo snap install kubectl --classic

# setup cluster configuration
microk8s config > ~/kube/config

# test cluster configuration
kubectl get nodes
```

### Install Helm

Install `helm` using snap.

```bash
sudo snap install helm --classic
```

Check whether helm has been correctly installed.

```bash
# version of helm
helm version

# list releases 
helm list

# list repo
helm repo list
```

## First Helm Chart

### Create A New Chart Project

Creat chart with helm create command.

```bash
helm create mychart
```

Helm will create project folder `mychart` and basic project structure.

```bash
# show structure of the project folder
tree mychart
```

```
mychart/
├── charts
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── NOTES.txt
│   ├── serviceaccount.yaml
│   ├── service.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml
```

The bootstraped project will deploy nginx to kubernetes. We can see the deployment image configuration in `values.yaml` file:

```yaml
# This sets the container image more information can be found here: https://kubernetes.io/docs/concepts/containers/images/
image:
  repository: nginx
  # This sets the pull policy for images.
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion.
  tag: ""
```

### Installing Chart

We can install a chart with command:

```bash
helm install $RELEASE_NAME $CHART_NAME
```

`CHART_NAME` is the chart we want to install to kubernetes and `RELEASE_NAME` is the name used by helm to manage our installation. We can install a chart several times with different release names. For example, we can install our `mychart` chart for development, staging, and production with release names: `mychart-dev`, `mychart-stg`, and `mychart-prod`.

Install our `mychart` chart to kubernetes.

```bash
# make sure current directory is the parent of mychart folder
cd path_to_parent_of_mychart_project_directory

# list current directory
ls 
# the mychart project directory should be listed on ls output

# install mychart chart with mychart-dev as release name
helm install mychart-dev mychart
```

### List Installed Chart

We can view list of helm release with command:

```bash
helm list
```

The command will show our `my-chart-dev` release.

```
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
mychart-dev     default         1               2025-01-14 12:13:09.760425102 +0700 WIB deployed        mychart-0.1.0   1.16.0  
```

### List Kubernetes Resources

After make sure that our release is successfuly installed, let's check on kubernetes.

```bash
kubectl get all 
```

```
NAME                               READY   STATUS    RESTARTS   AGE
pod/mychart-dev-74b758c789-p4rqx   1/1     Running   0          4m37s

NAME                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
service/kubernetes    ClusterIP   10.152.183.1     <none>        443/TCP   93m
service/mychart-dev   ClusterIP   10.152.183.150   <none>        80/TCP    4m38s

NAME                          READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mychart-dev   1/1     1            1           4m38s

NAME                                     DESIRED   CURRENT   READY   AGE
replicaset.apps/mychart-dev-74b758c789   1         1         1       4m37s
```

The chart will create kubernetes resources for our mychart-dev release:
- Pod: pod/mychart-dev-74b758c789-p4rqx
- Service: service/mychart-dev
- Deployment: deployment.apps/mychart-dev
- Replica Set: replicaset.apps/mychart-dev-74b758c789

### Update Service Type on Installed Release

We can update our helm release with helm upgrade command.

```bash
helm upgrade $RELEASE_NAME $CHART_NAME
```

Let's modify service port to Node Port. We can do that by modify service configuration in `values.yaml`:

```yaml
# This is for setting up a service more information can be found here: https://kubernetes.io/docs/concepts/services-networking/service/
service:
  # This sets the service type more information can be found here: https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types
  type: ClusterIP
  # This sets the ports more information can be found here: https://kubernetes.io/docs/concepts/services-networking/service/#field-spec-ports
  port: 80
```

Change `service.type` value to `NodePort` and `service.port` value to `8080`:

```yaml
# This is for setting up a service more information can be found here: https://kubernetes.io/docs/concepts/services-networking/service/
service:
  # This sets the service type more information can be found here: https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types
  type: NodePort
  # This sets the ports more information can be found here: https://kubernetes.io/docs/concepts/services-networking/service/#field-spec-ports
  port: 8080
```

Update the helm release with the modified `values.yaml`.

```bash
helm upgrade mychart-dev mychart
```

Check the release.

```bash
helm list
```

```
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
mychart-dev     default         2               2025-01-14 12:36:13.474858928 +0700 WIB deployed        mychart-0.1.0   1.16.0 
```

If the release is successfuly upgraded, the `REVISION` number of our release will increase by one.

Next, check the kubernetes service.

```bash
kubectl get service
```

```
NAME          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
kubernetes    ClusterIP   10.152.183.1     <none>        443/TCP    117m
mychart-dev   ClusterIP   10.152.183.150   <none>        8080/TCP   28m
```

As your can see, the service port has been changed. 

Open browser and open `SERVICE_IP:8080` to access the nginx deployment, ex: `http://10.152.183.150:8080`

### Show Release History

We can view release history with command:

```bash
helm history $RELEASE_NAME
```

Example:

```bash
# show list of release
helm list

# show release history
helm history mychart-dev
```

```
REVISION        UPDATED                         STATUS          CHART           APP VERSION     DESCRIPTION     
1               Tue Jan 14 12:13:09 2025        superseded      mychart-0.1.0   1.16.0          Install complete
2               Tue Jan 14 12:36:13 2025        superseded      mychart-0.1.0   1.16.0          Upgrade complete
3               Tue Jan 14 12:41:49 2025        deployed        mychart-0.1.0   1.16.0          Upgrade complete
```

### Rollback Helm Release

We can rollback helm release to previous revision number.

```bash
helm rollback $RELEASE_NAME $TARGET_REVISION_NUMBER
```

Example:

```bash
helm rollback mychart-dev 1
```

Check kubernetes service.

```bash
kubectl get service
```

```
NAME          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
kubernetes    ClusterIP   10.152.183.1     <none>        443/TCP   135m
mychart-dev   ClusterIP   10.152.183.150   <none>        80/TCP    46m
```

As you can see, the kubernetes service has been rollbacked to older release version.

Check the release history.

```bash
helm history mychart-dev
```

```
REVISION        UPDATED                         STATUS          CHART           APP VERSION     DESCRIPTION     
1               Tue Jan 14 12:13:09 2025        superseded      mychart-0.1.0   1.16.0          Install complete
2               Tue Jan 14 12:36:13 2025        superseded      mychart-0.1.0   1.16.0          Upgrade complete
3               Tue Jan 14 12:41:49 2025        superseded      mychart-0.1.0   1.16.0          Upgrade complete
4               Tue Jan 14 12:58:33 2025        superseded      mychart-0.1.0   1.16.0          Rollback to 1   
5               Tue Jan 14 13:01:19 2025        deployed        mychart-0.1.0   1.16.0          Rollback to 3 
```

Each rollback operation will create a new revision.

### Debug And Dry Run

Validate chart before install.

```bash
helm install --dry-run --debug $RELEASE_NAME $CHART_NAME
```

Example:

```bash
helm install --dry-run --debug mychart-dev mychart
```

The command will print rendered yaml and error message if exist.

With this command option we can make sure that the chart and it's configuration are correct before installing the chart to kubernetes.

### Render Helm Template

We can render Helm Template to verify the output yaml of the chart.

```bash
helm template $CHART_NAME
```

Example:

```bash
helm template mychart
```

### Helm lint

Find any errors or misconfiguration.

```bash
helm lint $CHART_NAME
```

Example:

```bash
helm lint mychart
```

```
==> Linting mychart
[INFO] Chart.yaml: icon is recommended

1 chart(s) linted, 0 chart(s) failed
```

### Uninstall Release

We can uninstall a installed release with helm uninstall command.

```bash
helm uninstall $RELEASE_NAME 
```

Example:

```bash
helm uninstall mychart-dev 
```

```
release "mychart-dev" uninstalled
```

Check helm release.

```bash
helm list 
```

```
NAME    NAMESPACE       REVISION        UPDATED STATUS  CHART   APP VERSION
```

The release has been uninstalled.

```bash 
kubectl get all
```

```
NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.152.183.1   <none>        443/TCP   164m
```

All mychart-dev kubernetes resources has been deleted.

## Custom Helm Chart

We will build:
- Python API Application
- Application Docker Image
- Helm Chart for Application
- Deploy Application with Helm Chart

### Download Application Source Code

Download source code from github:

```bash
wget -O python-flask-rest-api-project.zip https://github.com/rahulwagh/python-flask-rest-api-project/archive/refs/heads/main.zip
```

Unzip the source code.

```bash
unzip python-flask-rest-api-project.zip
```

Structure of python api project:

```bash
tree unzip python-flask-rest-api-project-main
```

```
python-flask-rest-api-project-main
├── dockerfile
├── main.py
├── README.md
└── requirements.txt
```

### Setup Python Virtual Environment

Enter python project directory.

```bash
cd python-flask-rest-api-project-main
```

Create virtual environment with pyenv and activate.

```bash
# create virtualenv
pyenv virtualenv 3.12.8 flask

# activate the virtualenv
pyenv activate flask
```

Install pip requirements.

```bash
pip install -r requirements.txt
```

### Run API

Run python flask application.

```bash
python -m main
```

```
 * Serving Flask app 'main'
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:9001
 * Running on http://192.168.1.6:9001
Press CTRL+C to quit
```

Use curl to test the API.

```bash
curl localhost:9001/hello
```

```
{"data":"Hello World"}
```

The API is working and we got json response from API.


```
 * Serving Flask app 'main'
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:9001
 * Running on http://192.168.1.6:9001
Press CTRL+C to quit
127.0.0.1 - - [14/Jan/2025 14:08:32] "GET / HTTP/1.1" 404 -
127.0.0.1 - - [14/Jan/2025 14:08:46] "GET /hello HTTP/1.1" 200 -
```

`Ctr + C` to close the application.

### Build Docker Image

Build docker image for the application.

```bash
docker build -t python-rest-api .
```

```
 => => extracting sha256:cc48f13b5f0f44b2e298de83a94a99fe7abdfb3335fe9b7811b8f764abb1a4ac                                                       0.8s
 => => extracting sha256:5a98c896c047f960c5fd29d44fa778899a68e7ebfb6a6a4f2a3fbf7baa902f6a                                                       0.0s
 => [internal] load build context                                                                                                               0.0s
 => => transferring context: 1.62kB                                                                                                             0.0s
 => [2/4] WORKDIR /app                                                                                                                          0.6s
 => [3/4] COPY . /app                                                                                                                           0.0s
 => [4/4] RUN pip --no-cache-dir install -r requirements.txt                                                                                    6.0s
 => exporting to image                                                                                                                          0.2s
 => => exporting layers                                                                                                                         0.2s
 => => writing image sha256:36d618873b0a3ed40064be770f181c0aa38923e9270a0503bb7fc09628fcfd3e                                                    0.0s
 => => naming to docker.io/library/python-rest-api 
```

Show docker image on local.

```bash
docker image list
```

```
REPOSITORY                    TAG                    IMAGE ID       CREATED         SIZE
python-rest-api               latest                 36d618873b0a   3 minutes ago   1.01GB
```

We need to import the docker image into our microk8s cluster. To do that first we export the docker image and then import it into microk8s cluster.

Export docker image: 

```bash
docker save python-rest-api > python-rest-api.tar
```

Import image tar file into microk8s cluster.

```bash
microk8s ctr image import python-rest-api.tar
```

Make sure that the image is imported successfuly. 

```bash
microk8s ctr image ls
```

```
REF                                                                                                       TYPE                                                      DIGEST                                                                  SIZE      PLATFORMS                                                                     LABELS 
docker.io/library/python-rest-api:latest                                                                  application/vnd.oci.image.manifest.v1+json                sha256:8d9a00f95f8e8621a461d0bae98988db68e78e302758fb6832ac5e21c989abbc 981.5 MiB linux/amd64                                                                   io.cri-containerd.image=managed                         
```

### Create New Helm Chart Project

Go to parent directory.

```bash
# go to parent directory
cd ..

# list directory
ls
```
```
mychart  python-flask-rest-api-project-main
```

Create new helm chart.

```bash
helm create python-rest-api-project
```
```
Creating python-rest-api-project
```

Show chart project structure:


```bash
tree python-rest-api-project
```
```
python-rest-api-project
├── charts
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── NOTES.txt
│   ├── serviceaccount.yaml
│   ├── service.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml

4 directories, 10 files
```

### Set Application Version

Open `Chart.yaml` file and edit `appVersion` to `1.0.0`.

```yaml
# This is the version number of the application being deployed. This version number should be
# incremented each time you make changes to the application. Versions are not expected to
# follow Semantic Versioning. They should reflect the version the application is using.
# It is recommended to use it with quotes.
appVersion: "1.0.0"
```

This is metadata for our application version.

### Change Docker Image

Open and edit `values.yaml` file to change image repository and tag:
- repository: python-rest-api
- tag: latest

```yaml
# This sets the container image more information can be found here: https://kubernetes.io/docs/concepts/containers/images/
image:
  repository: python-rest-api
  # This sets the pull policy for images.
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion.
  tag: latest

```

### Change Service Port

Open and edit `values.yaml` file to change kubernetes service configuration `service.type` and `service.port`:
- type: NodePort
- port: 9001

```yaml
# This is for setting up a service more information can be found here: https://kubernetes.io/docs/concepts/services-networking/service/
service:
  # This sets the service type more information can be found here: https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types
  type: NodePort
  # This sets the ports more information can be found here: https://kubernetes.io/docs/concepts/services-networking/service/#field-spec-ports
  port: 9001
```

### Change Kubernetes Liveness Probe

We need to change kubernetes deployment probe to accomodate the python application.

Open and Edit `values.yaml`:
- `resources.livenessProbe.httpGet.path`: /hello
- `resources.readinessProbe.httpGet.path`: /hello

```yaml
# This is to setup the liveness and readiness probes more information can be found here: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/
livenessProbe:
  httpGet:
    path: /hello
    port: http
readinessProbe:
  httpGet:
    path: /hello
    port: http

```

### Install Helm Chart

Deploy the chart to kubernetes.

```bash
helm install python-rest-api-project-dev python-rest-api-project
```

```
NAME: python-rest-api-project-dev
LAST DEPLOYED: Tue Jan 14 16:23:35 2025
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services python-rest-api-project-dev)
  export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT
```

Show helm release list:

```bash
helm list
```

```
NAME                            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                           APP VERSION
python-rest-api-project-dev     default         1               2025-01-14 16:23:35.6178963 +0700 WIB   deployed        python-rest-api-project-0.1.0   1.0.0  
```

Check created kubernetes resources.

```bash
kubectl get all 
```

```
NAME                                               READY   STATUS    RESTARTS   AGE
pod/python-rest-api-project-dev-7698996b59-nn7kd   1/1     Running   0          3s

NAME                                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/kubernetes                    ClusterIP   10.152.183.1     <none>        443/TCP          5h39m
service/python-rest-api-project-dev   NodePort    10.152.183.241   <none>        9001:30250/TCP   4s

NAME                                          READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/python-rest-api-project-dev   1/1     1            1           4s

NAME                                                     DESIRED   CURRENT   READY   AGE
replicaset.apps/python-rest-api-project-dev-7698996b59   1         1         1       4s
```

The chart is deployed to kubernetes successfuly.

Test application api with curl:

```bash
curl http://10.152.183.241:9001/hello
```

```
{"data":"Hello World"}
```

Call to API works successfuly as expected.

## Helmfile

Github: https://github.com/helmfile/helmfile 

Helmfile is a declarative spec for deploying helm charts. With Helmfile you can:

- Keep a directory of chart value files and maintain changes in version control.
- Apply CI/CD to configuration changes.
- Periodically sync to avoid skew in environments.


Here are some key benfits of using Helmfile -

- You can bundle several Helm Charts into a Single Helmfile to manage your kubernetes eco system
- Helmfile helps you to keep isolation between the different environments(developemnt, staging, production)
- It can help you to identify the differences between the new changes which you want to apply against the existing running deployment inside kubernetes cluster
- Helmfile uses the Go Templates which lets you templatify your Helmfile and also you can use Sprig Library with functions - requiredEnv, exec, readFile, toYaml, fromYaml, setValueAtPath, get, tpl, required, fetchSecretValue, expandSecretRefs
- With the help of HelmFile you can deploy multi-tier applications inside kubernetes cluster.

### Install Helmfile

Create installation binary.

```bash
# go to installation directory
cd ~/apps

# make directory
mkdir helmfile

# enter directory
cd helmfile
```

Download binary.

```bash
wget https://github.com/helmfile/helmfile/releases/download/v1.0.0-rc.8/helmfile_1.0.0-rc.8_linux_amd64.tar.gz
```

Extract binary.

```bash
tar -xzvf helmfile_1.0.0-rc.8_linux_amd64.tar.gz
```

Create and move binary to bin directory.

```bash
# create bin folder
mkdir bin

# move binary to bin folder
mv helmfile bin/
```

Add binary directory to PATH environment variable by adding these lines to `~/.bashrc` file.

```
# Helmfile Path 
export PATH="/home/irfan/apps/helmfile/bin:$PATH"
```

Restart/reopen terminal and check helm file CLI.

```bash
helmfile -v
```

```
helmfile version 1.0.0-rc.8
```

Helmfile is installed successfully.

### Create Helm Project

```bash
helm create helloworld
```

### Create helmfile

Create `helmfile.yaml` file with content:

```yaml
releases:
  - name: helloworld
    chart: helloworld
    installed: true
```

### Deploy with helmfile

Deploy chart(s) with helmfile.

```bash
helmfile sync
```

```
Building dependency release=helloworld, chart=helloworld
Upgrading release=helloworld, chart=helloworld, namespace=
Release "helloworld" does not exist. Installing it now.
NAME: helloworld
LAST DEPLOYED: Tue Jan 14 17:56:14 2025
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=helloworld,app.kubernetes.io/instance=helloworld" -o jsonpath="{.items[0].metadata.name}")
  export CONTAINER_PORT=$(kubectl get pod --namespace default $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace default port-forward $POD_NAME 8080:$CONTAINER_PORT

Listing releases matching ^helloworld$
helloworld      default         1               2025-01-14 17:56:14.870367794 +0700 WIB deployed        helloworld-0.1.0        1.16.0     


UPDATED RELEASES:
NAME         NAMESPACE   CHART        VERSION   DURATION
helloworld               helloworld   0.1.0           0s
```

The releases that defined in `helmfile.yaml` will be deployed. 

Check helm releases.

```bash
helm list
```

```
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
helloworld      default         2               2025-01-14 18:01:22.580371401 +0700 WIB deployed        helloworld-0.1.0        1.16.0   
```

Check kubernetes resources:

```bash
kubectl get all
```

```
NAME                             READY   STATUS    RESTARTS   AGE
pod/helloworld-d885c4655-w7m4r   1/1     Running   0          74s

NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
service/helloworld   ClusterIP   10.152.183.143   <none>        80/TCP    75s
service/kubernetes   ClusterIP   10.152.183.1     <none>        443/TCP   7h13m

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/helloworld   1/1     1            1           74s

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/helloworld-d885c4655   1         1         1       74s
```

### Remove deployment with helmfile

Edit field `installed` in the configuration file `helmfile.yaml` with `false` value.

```yaml
releases:
  - name: helloworld
    chart: helloworld
    installed: false
```

After that, run helmfile sync command.

```bash
helmfile sync
```

Check helm releases.

```bash
helm list
```

```
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION  
```

Check kubernetes resources:

```bash
kubectl get all
```

```
NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.152.183.1   <none>        443/TCP   7h21m
```

### Using Repository 

We can use github repository to add charts.

Add helm-git plugin:

```bash
helm plugin install https://github.com/aslafy-z/helm-git --version 1.3.0
```

Add repositories configuration to `helmfile.yaml` file.

Example `helmfile.yaml`:

```yaml
repositories:
  - name: helloworld-repo
    url: git+https://github.com/isirfanm/complete-helm-chart-tutorial@helloworld
releases:
  - name: helloworld
    chart: helloworld-repo/helloworld
    installed: true 
```

## Using Helm Repo

We can add helm repository and use charts from that repository.

### Search Repository

Search repository for wordpress from Artifact Hub: https://artifacthub.io/. Enter search keyword and open the chart page. 

In the chart page, click Install button to view installation instruction. 

Example:

```bash
# Add repository
helm repo add bitnami https://charts.bitnami.com/bitnami

# Install chart
helm install my-wordpress bitnami/wordpress --version 24.1.6
```

### Add Repository

Add new repository to helm and install chart.

```bash
# Add repository
helm repo add bitnami https://charts.bitnami.com/bitnami

# Install chart
helm install my-wordpress bitnami/wordpress --version 24.1.6
```

## Helm Hook

We can use helm hook to perform some task on helm install/upgrade/rollback processes. Available hook can be found on the hook documentation: https://helm.sh/docs/topics/charts_hooks/#the-available-hooks

| Annotation Value | Description |
| -----------------|-------------|
|pre-install	     | Executes after templates are rendered, but before any resources are created in Kubernetes|
|post-install      | Executes after all resources are loaded into Kubernetes|
|pre-delete        | Executes on a deletion request before any resources are deleted from Kubernetes|
|post-delete       | Executes on a deletion request after all of the release's resources have been deleted|
|pre-upgrade       | Executes on an upgrade request after templates are rendered, but before any resources are updated
|post-upgrade      | Executes on an upgrade request after all resources have been upgraded|
|pre-rollback      | Executes on a rollback request after templates are rendered, but before any resources are rolled back
|post-rollback     | Executes on a rollback request after all resources have been modified|
|test              | Executes when the Helm test subcommand is invoked ( view test docs)|

Example `helloworld/templates/hooks/post-install-job.yaml`:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: "{{ .Release.Name }}"
  labels:
    app.kubernetes.io/managed-by: {{ .Release.Service | quote }}
    app.kubernetes.io/instance: {{ .Release.Name | quote }}
    app.kubernetes.io/version: {{ .Chart.AppVersion }}
    helm.sh/chart: "{{ .Chart.Name }}-{{ .Chart.Version }}"
  annotations:
    # This is what defines this resource as a hook. Without this line, the
    # job is considered part of the release.
    "helm.sh/hook": post-install
    "helm.sh/hook-weight": "-5"
    "helm.sh/hook-delete-policy": hook-succeeded
spec:
  template:
    metadata:
      name: "{{ .Release.Name }}"
      labels:
        app.kubernetes.io/managed-by: {{ .Release.Service | quote }}
        app.kubernetes.io/instance: {{ .Release.Name | quote }}
        helm.sh/chart: "{{ .Chart.Name }}-{{ .Chart.Version }}"
    spec:
      restartPolicy: Never
      containers:
      - name: post-install-job
        image: "alpine:3.3"
        command: ["/bin/sleep","{{ default "10" .Values.sleepyTime }}"]
```

It is possible to define a weight for a hook which will help build a deterministic executing order. Weights are defined using the following annotation:

```yaml
annotations:
  "helm.sh/hook-weight": "5"
```
Hook weights can be positive or negative numbers but must be represented as strings. When Helm starts the execution cycle of hooks of a particular Kind it will sort those hooks in ascending order.

## Helm Test

We can use helm test to do testing on our installed helm chart. Documentation: https://helm.sh/docs/topics/chart_tests/ 

Example `helloworld/templates/tests/test-connection.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "helloworld.fullname" . }}-test-connection"
  labels:
    {{- include "helloworld.labels" . | nindent 4 }}
  annotations:
    "helm.sh/hook": test
spec:
  containers:
    - name: wget
      image: busybox
      command: ['wget']
      args: ['{{ include "helloworld.fullname" . }}:{{ .Values.service.port }}']
  restartPolicy: Never
```

Install helm chart:

```bash
helm install helloworld-dev helloworld
```

Run helm test to a helm release:

```bash
helm test helloworld-dev
```

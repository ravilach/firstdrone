# Welcome to your first Drone project!

Feel free to leverage this project as your first Drone project. Can learn
about the basics about [Drone](https://drone.io/) from the project site
and the [project itself](https://github.com/drone/drone). 

A video walkthrough of the Drone installation steps can be found on the
Harness Blog: [Your First Drone Installation, Build, and Push](https://harness.io/2020/08/your-first-harness-ci-installation/) 

This is a simple structure of a Go Lang class and a Dockerfile to create an image
of the Go Lang class. The Drone.yaml is wired to Drone out-of-box events.  

* Main Go
* Dockerfile
* Drone.yaml 

Make sure in your Drone.yaml to edit the Docker Registry to be yours. The
example `repo: rlachhman/myrepo` needs to be updated to yours. 

Below are the commands to install the [Drone Server](https://docs.drone.io/server/overview/) and [Drone Runner](https://docs.drone.io/runner/overview/)
into Kubernetes. 

## Drone Docker Server Pull and Run

```
#Pull Server
sudo docker pull drone/drone:1

#Run Server
sudo docker run \
  --volume=/var/lib/drone:/data \
  --env=DRONE_GITHUB_CLIENT_ID=yourID \
  --env=DRONE_GITHUB_CLIENT_SECRET=yourSecret \
  --env=DRONE_RPC_SECRET=yourRPC \
  --env=DRONE_SERVER_HOST=yourAddressOrIP \
  --env=DRONE_SERVER_PROTO=http \
  --env=DRONE_USER_CREATE=username:yourGitHubUser,admin:true \
  --publish=80:80 \
  --publish=443:443 \
  --restart=always \
  --detach=true \
  --name=drone \
  drone/drone:1
```
## Drone Kubernetes Runner Role Binding
Save to drone_role.yaml then
*kubectl apply -f drone_role.yaml*

```
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: drone
rules:
- apiGroups:
  - ""
  resources:
  - secrets
  verbs:
  - create
  - delete
- apiGroups:
  - ""
  resources:
  - pods
  - pods/log
  verbs:
  - get
  - create
  - delete
  - list
  - watch
  - update

---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: drone
  namespace: default
subjects:
- kind: ServiceAccount
  name: default
  namespace: default
roleRef:
  kind: Role
  name: drone
  apiGroup: rbac.authorization.k8s.io

```

## Drone Kubernetes Runner Deployment
Save to drone_deployment.yaml then
*kubectl apply -f drone_deployment.yaml*

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: drone
  labels:
    app.kubernetes.io/name: drone
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: drone
  template:
    metadata:
      labels:
        app.kubernetes.io/name: drone
    spec:
      containers:
      - name: runner
        image: drone/drone-runner-kube:latest
        ports:
        - containerPort: 3000
        env:
        - name: DRONE_RPC_HOST
          value: yourServerHostOrIP
        - name: DRONE_RPC_PROTO
          value: http
        - name: DRONE_RPC_SECRET
          value: yourRPC

```

## Some Helper Methods
A few helper items. Can remove the Drone Server and leverage the CLI to repair/recreate the webooks
in the repositories. 

### Remove Drone
```
sudo docker ps
sudo docker kill dronePodName
sudo docker container rm drone

```

### Install CLI
```
curl -L https://github.com/drone/drone-cli/releases/latest/download/drone_linux_amd64.tar.gz | tar zx
sudo install -t /usr/local/bin drone

export DRONE_SERVER=yourServerHostOrIP
export DRONE_TOKEN=yourAuthToken

```

### Repair Repository
This will need the Drone CLI with an admin user. 
For example this repo is refered as "ravilach/firstdrone"
```
#drone repo info ravilach/firstdrone
drone repo info your/repo
drone repo repair your/repo

```
Thanks for checking out the example!

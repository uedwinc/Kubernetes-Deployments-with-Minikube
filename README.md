# Kubernetes with Minikube

This project involves:

- Setting up minikube for kubernetes
- Carrying out basic kubectl operations
- Creating a MongoDB and Mongo-Express deployments

---

## Setup

1. Prerequisite - 
    - Make sure virtualization is enabled on your system
    - Install oracle virtual box

2. Install minikube (https://minikube.sigs.k8s.io/docs/start/) with chocolatey (https://community.chocolatey.org/packages?q=minikube)

    ```
    choco install minikube
    ```

3. Then do ```minkube start``` (use `minkube start --help` to see customization options)

    ```
    minikube start --driver=virtualbox --cpus=2 --memory=2g
    ```

- To ignore virtual machine check, use:

    ```
    minikube start --driver=virtualbox --cpus=2 --memory=2g --no-vtx-check
    ```

4. Confirm minikube and kubectl installation:

    `minikube version`

    `kubectl version`

![minikube installed]()

5. kubectl crud (create read update delete) operations

    `kubectl get svc`

    `kubectl get replicaset`

    `kubectl get deployment`

    `kubectl get nodes`

    `kubectl get pods`

![kubectl cmds]()

- To create deployment:

    `kubectl create deploy nginx-depl --image=nginx`

![ngx depl img]()

- To check / edit deployment file

    `kubectl edit deploy nginx-depl`

- To inspect your pods or troubleshoot

    `kubectl logs pod-name`

    `kubectl describe pod pod-name`

    `kubectl describe deploy deployment-name`

    `kubectl describe service name-of-service`

- To get more info about pods

    `kubectl get pods -o wide`

![0 wide]()

- To access or log into your pods

    `kubectl exec -it name-of-pod -- bin/bash`

![exec]()

- To delete deployment

    `kubectl delete deploy name-of-deployment`

![kube delete]()

### Creating Deployment Using a yaml file

- To create deployment and service, you have to apply the deployment file and the service file

    `kubectl apply -f path-to-file`

![apply file]()

![k svc]()

- To delete deployment created with file (delete with both deployment file and service file)

    `kubectl delete -f path-to-deploy-file`

---


## Mongodb Deployment on Minikube

- We need to create yaml files for our deployments, service and secrets (keys for accessing the deployment)

1. Secret file

- Defined in [mongo-secret.yaml]()

- Contains base64 encripted key for mongodb defined in the environmental variable of the deployment file
    + Open git bash:
        + Do `echo -n specify-username | base64`
        + Do `echo -n specify-password | base64`
    + the encrypted values generated here are passed in as username and password in the mongo-secret.yaml file.

- First, create the namespace defined in the yaml file (I will use 'mongo' as namespace)

    ```
    kubectl create ns mongo
    ```

- Run the secrets file

    ```
    kubectl apply -f mongo-secret.yaml
    ```

- Do `kubectl get secret -n mongo` to confirm

2. Deployment and Service file

- Defined in [mongo-deployment.yaml]()

- Service:
    This is used to expose the deployment to external communication like the mongo-express

- Run the file

```
kubectl apply -f mongo-deployment.yaml
```

![apply mongo]()

- Confirm deployments

    `kubectl get deployment -n mongo`

    `kubectl get svc -n mongo`

![depl and svc]()

    `kubectl describe svc mongodb-service -n mongo`

![describe svc]()

    `kubectl get pods -n mongo -o wide`

![wide]()

## Mongo-Express Deployment

1. Configmap file

- Defined in [mongo-configmap.yaml]()

- This is used to connect to the database

- Start the configmap process

    ```
    kubectl apply -f mongo-configmap.yaml
    ```

- Do `kubectl get configmap -n mongo` to confirm

![config]()

2. Deployment and Service file

- Defined in [mongo-express.yaml]()

- Run the file

    ```
    kubectl apply -f mongo-express.yaml
    ```

![mx apply]()

- Confirm

    `kubectl get pods -n mongo`

    `kubectl get svc -n mongo`

    `kubectl describe svc mongo-express-service -n mongo`

![mg express]()

3. External Access

- Do `minikube service mongo-express-service -n mongo`

- This will open up your browser to mongo-express

- Create database and collections


## Ingress to expose the deployment

- This removes the need for a nodePort in the ip address

1. Enable the ingress add-on

- Do `kubectl get ns` to see available namespaces (ns = namespace)
    + Namespaces helps to isolate resources within a cluster
    + So, it is important to have created mongo-express in the same namespace you will create ingress in
    + `kubectl get all -n name-of-namespace` will tell you which services were created inside the namespace.

- To enable / create an ingress namespace, use `minikube addons enable ingress`

2. Create a yaml file for ingress, defined in [mongo-express-ingress.yaml]()

```
kubectl apply -f mongo-express-ingress.yaml
```

- Do `kubectl get ingress -n mongo` to confirm

3. Inorder to access the host, edit the hosts file

    + For linux, `vi /etc/hosts`

    + For windows:
        - Run notepad as administrator
        - Go to file > open
        - Next, under localdisk > windows, go to System32 > drivers > etc
        - Change file type dropdown from Text documents to All files
        - Then open the hosts file
    
    + In a new line, enter the host ip gotten from `kubectl get ingress` and the hostname to be mapped, that is, mongo-express.com

    + Save and close.

4. Now, enter "mongo-express.com" on the browser to open the mongo-express page.
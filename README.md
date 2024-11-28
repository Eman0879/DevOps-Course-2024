# Deploying a Simple MERN Application Using Kubernetes 


This guide explains the step-by-step process of deploying a simple MERN application on Kubernetes using Minikube. The application uses MongoDB and Express containers available on Docker Hub.

## Prerequisites

### Install Required Tools
1. Install Visual Studio Code (VS Code) for writing YAML files and managing code.
2. Install Docker Desktop from the [offical website](https://docs.docker.com/desktop/setup/install/windows-install/).
3. Install kubectl on windows using Chocolatey:
   
   ```bash
   choco install kubernetes-cli 
   ```
4. Install Minikube using Chocolatey:
   
   ```bash
   choco install minikube
   ```
 ### Verify Installation 
 To check if kubectl is properly installed, run:
  
  ```bash
  kubectl --help
  ```

## Steps to Deploy the Application 

### Step 1: Create a Secret
Create a secret.yaml file in VS Code to store the username and password for MongoDB. A Kubernetes Secret is used to store sensitive information, such as passwords, tokens, or keys.
Ensure the username and password are base64-encoded. For example:

```bash
apiVersion: v1
kind: Secret
metadata:
  name: mongo-secret
data:
  mongo-user: <base64-encoded-username>
  mongo-password: <base64-encoded-password>

```

### Step 2: Create a ConfigMap 
Write a mongo-config.yaml file in VS Code to define non-confidential configuration data as key-value pairs. This will be consumed by the Pods. A ConfigMap is an API object used to store non-confidential data in key-value pairs. Pods can consume ConfigMaps as environment variables, command-line arguments, or as configuration files in a volume.

```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: mongo-config
data:
  mongo-url: mongo-service
```

### Step 3: Create MongoDB Deployment 
Create a mongo-app.yaml file for MongoDB deployment. Kubernetes Deployments manage Pods and ReplicaSets, ensuring scalability and availability.
We are using the environment variables ME_CONFIG_MONGODB_ADMINUSERNAME and ME_CONFIG_MONGODB_ADMINPASSWORD given on the docker hub for the official docker image of mongo. 

Reference the secret created in Step 1 for MongoDB credentials. For example:

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-deployment
  labels:
    app: mongo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongo
  template:
    metadata:
      labels:
        app: mongo
    spec:
      containers:
      - name: mongo
        image: mongo:6.0
        ports:
        - containerPort: 27017
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          valueFrom:
            secretKeyRef:
              name: mongo-secret
              key: mongo-user
        - name: MONGO_INITDB_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mongo-secret
              key: mongo-password  

```


### Step 4: Create MongoDB Service
Write a mongo-service.yaml file to expose the MongoDB deployment as a service. Services allow Pods to communicate within the cluster.

```bash
apiVersion: v1
kind: Service
metadata:
  name: mongo-service
spec:
  selector:
    app : mongo
  ports:
    - protocol: TCP
      port: 27017
      targetPort: 27017

```

### Step 5: Create Mongo-Express Deployment 
Write a web-app.yaml file for deploying the Mongo-Express app. Mongo-Express is a web-based interface for managing MongoDB.In this yaml file everything is similar to mongo-app.yaml except that this will also contain information for the nodeport.
In this file we are going to use the environment variables given on docker hub for the official image of mongo-express ME_CONFIG_MONGODB_ADMINUSERNAME 
And  ME_CONFIG_MONGODB_ADMINPASSWORD.

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-deployment
  labels:
    app: webapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: mongo-express
        image: mongo-express:latest
        ports:
        - containerPort: 8081
        env:
        - name: ME_CONFIG_MONGODB_ADMINUSERNAME 
          valueFrom:
            secretKeyRef:
              name: mongo-secret
              key: mongo-user
        - name: ME_CONFIG_MONGODB_ADMINPASSWORD 
          valueFrom:
            secretKeyRef:
              name: mongo-secret
              key: mongo-password   
        - name: ME_CONFIG_MONGODB_SERVER 
          value: "mongo-service"
```


### Step 6: Expose Mongo-Express 
In the same web-app.yaml file, expose Mongo-Express using a NodePort to access the application from outside the cluster.

```bash
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
spec:
  type: NodePort
  selector: 
    app : webapp
  ports:
    - protocol: TCP
      port: 8081
      targetPort: 8081
      nodePort: 30200

```

### Step 7: Start Minikube
This will set up a single-node kubernetes cluster on local machine for deployment and testing.

```bash
minikube start
```

### Step 8: Deploy a Resource
Apply the configuration in the following order:

```bash
kubectl apply -f secret.yaml
kubectl apply -f mongo-config.yaml
kubectl apply -f mongo-app.yaml
kubectl apply -f web-app.yaml
```

### Step 9: Verify Pods
Check the status of Pods to ensure everything is running:

```bash
kubectl get pods
```
![getPods](https://github.com/user-attachments/assets/bf278dd3-780b-443b-a0f9-85b7c2d4c058)

### Step 10: Verify Services
Check the services running in your cluster:

```bash 
kubectl get svc

```

![svc](https://github.com/user-attachments/assets/4ab446c2-05fe-4993-94e4-0d8244b23290)


### Step 11: Access the Application
Expose the web application:

```bash
minikube service webapp-service
```
![Dep2](https://github.com/user-attachments/assets/cf8482a6-ee17-455a-93ac-f3f97b842a34)

This will open the Mongo-Express app in your browser.The webpage will prompt for a username and password because we have used the latest tag of the mongo-express image that comes a default username and password. The username and password is: admin and pass.

![webpage](https://github.com/user-attachments/assets/6a6425f9-4ceb-4f30-8b26-6c13934d811d)

### Success 
After successfully logging into Mongo-Express, you should see the web interface to manage your MongoDB instance. Congratulations, you've deployed a simple MERN application using Kubernetes!


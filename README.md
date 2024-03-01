# cloud-native-monitoring-app
<img width="568" alt="image" src="https://github.com/134420173-myseneca/cloud-native-monitoring-app/assets/39117847/76ce0bd1-5d71-44c2-9582-0fa205cfabae">

## **Part 1: Deploying the Flask application locally**

### **Step 1: Clone the code**

Clone the code from the repository:

```
git clone https://github.com/134420173-myseneca/cloud-native-monitoring-app.git
```

### **Step 2: Install dependencies**

The application uses the **`psutil`** and **`Flask`, Plotly, boto3** libraries. Install them using pip:

```
pip3 install -r requirements.txt
```

### **Step 3: Run the application**

To run the application, navigate to the root directory of the project and execute the following command:

```
python3 app.py
```
if running app on  cloud9 aws instance open port in security group for cloud instance and tru to access app with cloud9 instance public ip with port 50000 e.g http://34.207.216.95:5000
or
This will start the Flask server on **`localhost:5000`**. Navigate to [http://localhost:5000/](http://localhost:5000/) on your browser to access the application.


## **Part 2: Dockerizing the Flask application**

### **Step 1: Create a Dockerfile**

Create a **`Dockerfile`** in the root directory of the project with the following contents:

```
# Use the official Python image as the base image
FROM python:3.9-slim-buster

# Set the working directory in the container
WORKDIR /app

# Copy the requirements file to the working directory
COPY requirements.txt .

RUN pip3 install --no-cache-dir -r requirements.txt

# Copy the application code to the working directory
COPY . .

# Set the environment variables for the Flask app
ENV FLASK_RUN_HOST=0.0.0.0

# Expose the port on which the Flask app will run
EXPOSE 5000

# Start the Flask app when the container is run
CMD ["flask", "run"]
```

### **Step 2: Build the Docker image**

To build the Docker image, execute the following command:

```
docker build -t my-flask-app .
```

### **Step 3: Run the Docker container**

To run the Docker container, execute the following command:

```
docker run -p 5000:5000 my-flask-app
```

This will start the Flask server in a Docker container on **`localhost:5000`**. Navigate to [http://localhost:5000/](http://localhost:5000/) on your browser to access the application.

## **Part 3: Pushing the Docker image to ECR**

### **Step 1: Create an ECR repository**

Create an ECR repository using Python:

```
import boto3

# Create an ECR client
ecr_client = boto3.client('ecr')

# Create a new ECR repository
repository_name = 'my-ecr-repo'
response = ecr_client.create_repository(repositoryName=repository_name)

# Print the repository URI
repository_uri = response['repository']['repositoryUri']
print(repository_uri)
```
or

##### Create an ECR repository manually with below command

```
# Replace 'my-ecr-repo' with your desired repository name
aws ecr create-repository --repository-name my-ecr-repo

```

### **Step 2: Push the Docker image to ECR**

Push the Docker image to ECR using the push commands on the console:


Retrieve an authentication token and authenticate your Docker client to your registry.
Use the AWS CLI:
```
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 063129952770.dkr.ecr.us-east-1.amazonaws.com
```
Note: If you receive an error using the AWS CLI, make sure that you have the latest version of the AWS CLI and Docker installed.
Build your Docker image using the following command. For information on building a Docker file from scratch see the instructions here . You can skip this step if your image is already built:
```
docker build -t my_monitoring_app_image .
```
After the build completes, tag your image so you can push the image to this repository:
```
docker tag my_monitoring_app_image:latest 063129952770.dkr.ecr.us-east-1.amazonaws.com/my_monitoring_app_image:latest
```
Run the following command to push this image to your newly created AWS repository:
```
docker push 063129952770.dkr.ecr.us-east-1.amazonaws.com/my_monitoring_app_image:latest
```

## **Part 4: Creating an EKS cluster and deploying the app using Python**

### **Step 1: Create an EKS cluster**

Create an EKS cluster and add node group

### **Step 2: Create a node group**

Create a node group in the EKS cluster.

### **Step 3: Create deployment and service**

```jsx
from kubernetes import client, config

# Load Kubernetes configuration
config.load_kube_config()

# Create a Kubernetes API client
api_client = client.ApiClient()

# Define the deployment
deployment = client.V1Deployment(
    metadata=client.V1ObjectMeta(name="my-flask-app"),
    spec=client.V1DeploymentSpec(
        replicas=1,
        selector=client.V1LabelSelector(
            match_labels={"app": "my-flask-app"}
        ),
        template=client.V1PodTemplateSpec(
            metadata=client.V1ObjectMeta(
                labels={"app": "my-flask-app"}
            ),
            spec=client.V1PodSpec(
                containers=[
                    client.V1Container(
                        name="my-flask-container",
                        image="568373317874.dkr.ecr.us-east-1.amazonaws.com/my-cloud-native-repo:latest",
                        ports=[client.V1ContainerPort(container_port=5000)]
                    )
                ]
            )
        )
    )
)

# Create the deployment
api_instance = client.AppsV1Api(api_client)
api_instance.create_namespaced_deployment(
    namespace="default",
    body=deployment
)

# Define the service
service = client.V1Service(
    metadata=client.V1ObjectMeta(name="my-flask-service"),
    spec=client.V1ServiceSpec(
        selector={"app": "my-flask-app"},
        ports=[client.V1ServicePort(port=5000)]
    )
)

# Create the service
api_instance = client.CoreV1Api(api_client)
api_instance.create_namespaced_service(
    namespace="default",
    body=service
)
```

make sure to edit the name of the image on line 25 with your image Uri.

- Once you run this file by running “python3 eks.py” deployment and service will be created.
- Check by running following commands:

```jsx
kubectl get deployment -n default (check deployments)
kubectl get service -n default (check service)
kubectl get pods -n default (to check the pods)
```

Once your pod is up and running, run the port-forward to expose the service

```bash
kubectl port-forward service/<service_name> 5000:5000
```
## **Part 5: Optional Creating kubernetes cluster with kind and deploying the app using deployment and service yaml files**

change permission for init_kind.sh and excute script to create kubenetes cluster with kind

```bash
chmod +x init_kind.sh
./init_kind.sh
```
This command applies the configuration defined in the deployment.yaml file to Kubernetes. 


```bash
kubectl apply -f deployment.yaml
```
This command applies the configuration defined in the nodeservice.yaml file to Kubernetes.
```bash
kubectl apply -f nodeservice.yaml
```
Retrieves information about the nodes in the Kubernetes cluster
```bash
kubectl get pods
```
```bash
voclabs:~/environment/cloud-native-monitoring-app (main) $ kubectl get pods
NAME                            READY   STATUS    RESTARTS   AGE
my-flask-app-7fdf956447-q7scz   1/1     Running   0          10m
```
Access the application through browser  https://3.81.92.209:30076

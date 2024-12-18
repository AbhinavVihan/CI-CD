# Node.js CI/CD Pipeline

This repository contains a Node.js application with a CI/CD pipeline set up using GitHub Actions. The pipeline builds the application, creates a Docker image, deploys it to a Kubernetes cluster, and sends notifications about the deployment status.

## Approach

### 1. **CI/CD Pipeline Overview**

The pipeline is broken into the following stages:

1. **Build**:
   - Set up Node.js environment.
   - Install dependencies.
   - Run tests to ensure the application is working as expected.

2. **Docker Image Build & Push**:
   - Build a Docker image from the Node.js app.
   - Tag the image with a unique identifier (such as the commit SHA).
   - Push the image to Docker Hub.

3. **Kubernetes Deployment**:
   - Deploy the Docker image to a Kubernetes cluster (using Minikube for local development or another Kubernetes environment).
   - Update the Kubernetes Deployment with the new Docker image.

4. **Email Notification**:
   - Send success or failure notifications to a specified email address.

### 2. **Pipeline Flow**

The GitHub Actions workflow is triggered on every push or pull request to the `release` branch.

#### Jobs in the Workflow:

- **Build**:
  - Set up Node.js.
  - Install dependencies using `npm install`.
  - Run tests with `npm test`.
  - Build a Docker image using the `Dockerfile` in the root of the repository.

- **Deploy**:
  - Login to Docker Hub.
  - Push the Docker image to Docker Hub.
  - Deploy the image to Kubernetes using a Kubernetes deployment file.

- **Notify**:
  - Send an email notification on deployment success or failure using a mail service like Mailgun or Gmail.

### 3. **Environment Variables and Secrets**

The following secrets should be set up in your GitHub repository to enable the pipeline to work:

- **DOCKER_USERNAME**: Your Docker Hub username.
- **DOCKER_PASSWORD**: Your Docker Hub password or access token.
- **GMAIL_USERNAME**: Your Gmail address (if using Gmail).
- **GMAIL_PASSWORD**: Your Gmail app password (generated through Google’s 2FA and App Passwords feature).

### 4. **Deployment to Kubernetes**

The deployment uses a Kubernetes configuration defined in the `kubernetes/deployment.yml` file. The configuration includes:

- **Deployment**: Describes the desired state of the application, including the Docker image to use, the number of replicas, and the container ports.
- **Service**: Exposes the application using a `NodePort` to allow external access.

### 5. **Email Notifications**

You will receive email notifications on the following events:
- **Deployment Success**: When the deployment to Kubernetes is successful.
- **Deployment Failure**: When the deployment to Kubernetes fails.

You can configure email notifications using:
- **Gmail**: SMTP service to send email notifications.

### 6. **Kubernetes Deployment Configuration**

The `kubernetes/deployment.yml` file includes the necessary configuration to deploy the application to Kubernetes. The Docker image is dynamically updated based on the GitHub SHA of the commit.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: abhik0/my-app:${IMAGE_TAG}  # Dynamically set image tag
        ports:
        - containerPort: 3000

---
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: NodePort

# Deploy Two-Tier Flask App Using Kubernetes

This guide explains how to deploy a two-tier Flask application with MySQL using Kubernetes. It includes setting up persistent storage, Kubernetes Deployments, and exposing services.

## Prerequisites

- Docker installed
- Kubernetes cluster (Minikube, MicroK8s, or any cloud-managed Kubernetes service)
- kubectl configured

---

## Step 1: Clone the Repository and Prepare Files

First, clone the repository containing the Flask application:
```sh
git clone <your-github-repo-url>
cd mypro
```

Ensure your Flask application has the necessary dependencies and database setup.

### Why Create `requirements.txt`?
The `requirements.txt` file contains all the dependencies required for the Flask application to run. These will be installed inside the container.

### Create `requirements.txt`
```sh
Flask==2.0.1
Flask-MySQLdb==0.2.0
mysql-connector-python
requests==2.26.0
Werkzeug==2.2.2
```

### Why Create `message.sql`?
The `message.sql` file contains the SQL commands to create the necessary database table for storing messages.

### Create `message.sql`
```sql
CREATE TABLE IF NOT EXISTS messages (
    id INT AUTO_INCREMENT PRIMARY KEY,
    message TEXT
);
```

### Why Create `mysqldata` Directory?
The `mysqldata` directory is created to provide persistent storage for MySQL data. This prevents data loss when the MySQL container restarts.

Create a directory for MySQL data persistence:
```sh
mkdir mysqldata
```

---

## Step 2: MySQL Deployment

### `mysql-deployment.yml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
  labels:
    app: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql:latest
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: "admin"
            - name: MYSQL_DATABASE
              value: "mydb"
            - name: MYSQL_USER
              value: "admin"
            - name: MYSQL_PASSWORD
              value: "admin"
          ports:
            - containerPort: 3306
          volumeMounts:
            - name: mysqldata
              mountPath: /var/lib/mysql
      volumes:
        - name: mysqldata
          persistentVolumeClaim:
            claimName: mysql-pvc
```

### `mysql-svc.yml`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  selector:
    app: mysql
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
  clusterIP: None
```

### `mysql-pv.yml`
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
spec:
  capacity:
    storage: 256Mi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: "/c/Users/admin/Desktop/mypro/mysqldata"
```

### `mysql-pvc.yml`
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 256Mi
```

---

## Step 3: Flask Application Deployment

### `two-tier-app-deployment.yml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: two-tier-app
  labels:
    app: two-tier-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: two-tier-app
  template:
    metadata:
      labels:
        app: two-tier-app
    spec:
      containers:
        - name: my-flaskapp
          image: nitinbijlwan/flaskapp:latest
          env:
            - name: MYSQL_HOST
              value: "mysql"
            - name: MYSQL_PASSWORD
              value: "admin"
            - name: MYSQL_USER
              value: "root"
            - name: MYSQL_DB
              value: "mydb"
          ports:
            - containerPort: 5000
          imagePullPolicy: Always
```

### `two-tier-app-svc.yml`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: two-tier-app-service
spec:
  selector:
    app: two-tier-app
  ports:
    - protocol: TCP
      port: 5000
      targetPort: 5000
      nodePort: 30004
  type: NodePort
```

---

## Step 4: Deploy to Kubernetes

Apply the manifests:
```sh
kubectl apply -f mysql-pv.yml
kubectl apply -f mysql-pvc.yml
kubectl apply -f mysql-deployment.yml
kubectl apply -f mysql-svc.yml
kubectl apply -f two-tier-app-deployment.yml
kubectl apply -f two-tier-app-svc.yml
```

Check if the pods are running:
```sh
kubectl get pods
```

Get the NodePort of the application service:
```sh
kubectl get svc two-tier-app-service
```

Access the Flask app:
```sh
curl http://<NODE-IP>:30004
```

kubectl exec -it mysql-7f8dbdd66-b6667  -- bash
mysql -u root -p
SHOW DATABASES;
USE mydb;
SHOW TABLES;
SELECT * FROM messages;





---

## Step 5: Cleanup

To delete the resources:
```sh
kubectl delete -f two-tier-app-deployment.yml
kubectl delete -f two-tier-app-svc.yml
kubectl delete -f mysql-deployment.yml
kubectl delete -f mysql-svc.yml
kubectl delete -f mysql-pv.yml
kubectl delete -f mysql-pvc.yml
```

---





# Deploying Flask App on Kubernetes with AWS Load Balancer

## Prerequisites
- AWS Account
- Kubernetes Cluster running on AWS (EKS or self-managed)
- `kubectl` and `awscli` installed
- AWS IAM permissions to create ALB

## Steps to Deploy Flask App with AWS ALB



### Step 1: Create an Application Load Balancer (ALB) in AWS
1. Sign in to AWS Management Console.
2. Navigate to **EC2 Dashboard** > **Load Balancers**.
3. Click **Create Load Balancer** and select **Application Load Balancer**.
4. Set **Name** (e.g., `my-app-alb`) and select **Internet-facing**.
5. Choose **VPC** and at least two **subnets** in different availability zones.
6. Under **Security Groups**, select an existing group or create a new one allowing HTTP/HTTPS traffic.
7. In **Listeners**, add HTTP (Port 80) and configure it to forward traffic to a **Target Group**.
8. Click **Create Target Group**, set type to **Instance or IP**, and configure it with the correct Kubernetes node instances.
9. Register Kubernetes worker nodes as **targets** in the target group.
10. Review and create the ALB.

### Step 2: Modify and Apply MySQL Service for External Access
Modify `mysql-svc.yml` to use AWS Load Balancer:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "external"
spec:
  selector:
    app: mysql
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
  type: LoadBalancer
```
Apply the service:
```sh
kubectl apply -f mysql-svc.yml
```

### Step 3: Retrieve the External Load Balancer URL
```sh
kubectl get svc mysql
```

### Step 4: Access the Flask App
Open your browser and visit:
```sh
http://<EXTERNAL-IP>:5000
```



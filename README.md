# Kubernetes-Postgress-Deploy
Deploying PostgreSQL on Kubernetes

###Add the following into postgres-configmap.yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: postgres
data:
  POSTGRES_DB: mypostgres_production

##Apply the configmap by running "kubectl apply -f postgres-configmap.yaml"

###Then run the following command:
kubectl create secret generic postgres \
--from-literal=POSTGRES_USER="root" \
--from-literal=POSTGRES_PASSWORD="the-super-secret-password"

###Save the following into postgres-pv.yaml file:
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pv-claim
  labels:
    app: postgres
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi

### Now save the Deployment Manifest "postgres-deployment.yaml"

apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres  
    spec:
      containers:
      - name: postgres
        image: postgres:12.4-alpine
        ports:
          - containerPort: 5432
        envFrom:
          - secretRef:
              name: postgres-secrets
          - configMapRef:
              name: postgres-configmap  
        volumeMounts:
        - name: postgres-database-storage
          mountPath: /var/lib/pgsql/data
      volumes:
      - name: postgres-database-storage
        persistentVolumeClaim:
          claimName: postgres-pv-claim


## TO Apply the deployment - run "kubectl apply -f postgres-deployment.yaml"
###Create a postgres-service.yaml:

apiVersion: v1
kind: Service
metadata:
  name: postgres
spec:
  selector:
    app: postgress
  ports:
  - protocol: TCP
    port: 5432
  clusterIP: none
  
  

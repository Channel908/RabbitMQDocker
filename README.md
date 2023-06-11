# RabbitMQDocker

# Sql Kubernetes Installation

## SQL sa password secret
```
kubectl create secret generic mssql --from-literal=SA_PASSWORD=Abc123!!
```
## Apply Persistent Volume Claim

mssql-persist.yaml
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mssql-claim
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 200Mi
```
```
kubectl apply -f mssql-persist.yaml
```

## Apply SQL image, load balancer & cluster ip service

mssql-depl.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mssql-depl
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mssql
  template:
    metadata:
      labels:
        app: mssql
    spec:
      containers:
        - name: mssql
          image: mcr.microsoft.com/mssql/server:2017-latest
          ports:
            - containerPort: 1433
          env:
          - name: MSSQL_PID
            value: "Express"
          - name: ACCEPT_EULA
            value: "Y"
          - name: SA_PASSWORD
            valueFrom:
              secretKeyRef:
                name: mssql
                key: SA_PASSWORD
          volumeMounts:
          - mountPath: /var/opt/mssql/data
            name: mssqldb
      volumes:
      - name: mssqldb
        persistentVolumeClaim:
          claimName: mssql-claim
---
apiVersion: v1
kind: Service
metadata:
  name: mssql-clusterip-srv
spec:
  type: ClusterIP
  selector:
    app: mssql
  ports:
  - name: mssql
    protocol: TCP
    port: 1433
    targetPort: 1433
---
apiVersion: v1
kind: Service
metadata:
  name: mssql-loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: mssql
  ports:
  - protocol: TCP
    port: 1433
    targetPort: 1433
```
```
kubectl apply -f mssql-depl.yaml
```
## Connect
you should now be able to connect 
Server Name: localhost or computername or host name
Authentication: SQL Server Authentication
Login: sa
Pasword: Abc123!1 (secret key SA_PASSWORD value)

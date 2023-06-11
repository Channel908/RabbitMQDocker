# RabbitMQ Docker Kubernetes Installation

```
## Apply RabbitMQ Image, Cluster IP and Load balancer

mssql-persist.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rabbitmq-depl
spec:
  replicas: 1
  selector:
    matchLabels:
      app: rabbitmq
  template:
    metadata:
      labels:
        app: rabbitmq
    spec:
      containers:
        - name: rabbitmq
          image: rabbitmq:3-management
          ports:
            - containerPort: 15672
              name: rbmq-mgmt-port
            - containerPort: 5672
              name: rbmq-msg-port
---
apiVersion: v1
kind: Service
metadata:
  name: rabbitmq-clusterip-srv
spec:
  type: ClusterIP
  selector:
    app: rabbitmq
  ports:
  - name: rbmq-mgmt-port
    protocol: TCP
    port: 15672
    targetPort: 15672
  - name: rbmq-msg-port
    protocol: TCP
    port: 5672
    targetPort: 5672
---
apiVersion: v1
kind: Service
metadata:
  name: rabbitmq-loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: rabbitmq
  ports:
  - name: rbmq-mgmt-port
    protocol: TCP
    port: 15672
    targetPort: 15672
  - name: rbmq-msg-port
    protocol: TCP
    port: 5672
    targetPort: 5672


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

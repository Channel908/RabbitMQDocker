# RabbitMQ Docker Kubernetes Installation

## Apply the Image, Cluster IP & Load Balancer

rabbitmq-deploy.yaml
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
kubectl apply -f rabbitmq-depl.yaml
```
Or apply it via the Docker Dashboard user interface
(see the Docker Dasboard guids https://github.com/Channel908/DockerUI)

## Launch the user interface in your browser 
http://localhost:15672/

```
http://localhost:15672/
```
Username: guest
password: guest



# Documentation

Documentation about the microservices contained in the organisation.

To congfigure ingress:

```bash
kubectl apply -f ingress.yaml 
```

To connect to AKS to resource group **PolnilniceRG**, to cluster **polnilnice**, use:

```bash
az aks get-credentials --resoource-group PolnilniceRG --name polnilnice
```



## Running localy: prerequisites

Create network **rsonet** if there is none:

```bash
docker network create rsonet
```

Run Consul Docker container:
```bash
docker run -d --name consul -p 8500:8500 --network rsonet consul
```



## Uporabniki microservice

### Prerequisites

Running **pg-uporabniki** container:

```bash
docker run -d --name pg-uporabniki -e POSTGRES_USER=dbuser -e POSTGRES_PASSWORD=postgres -e POSTGRES_DB=customer -p 5432:5432 --network rsonet postgres:13
```

### Build and run Docker containers

Building **uporabniki** container:
```bash
docker build -f .\Dockerfile_with_maven_build -t uporabniki:latest .
```

Run:
```bash
docker run -p 8080:8080 --network rsonet -e KUMULUZEE_DATASOURCES0_CONNECTIONURL=jdbc:postgresql://pg-uporabniki:5432/customer -e KUMULUZEE_CONFIG_CONSUL_AGENT=http://consul:8500 --name uporabniki-instance uporabniki
```

### Deploy to Kubernetes

```bash
kubectl apply -f ./k8s/uporabniki-deployment.yaml
```

Make sure to select the right tag for the image to be used.




## Polnilnice microservice

### Prerequisites

```bash
docker run -d --name pg-polnilnice -e POSTGRES_USER=dbuser -e POSTGRES_PASSWORD=postgres -e POSTGRES_DB=polnilnice-data -p 5433:5432 postgres:13
```

Create Docker container.
```bash
docker run -p 8081:8080 --network rsonet -e KUMULUZEE_DATASOURCES0_CONNECTIONURL=jdbc:postgresql://192.168.99.100:5433/polnilnice-data -e KUMULUZEE_CONFIG_CONSUL_AGENT=http://192.168.99.100:8500 --name polnilnice-instance polnilnice
```

Start database and polnilnice-ms containers
```bash
docker start pg-polnilnice
docker start polnilnice-instance
```

### Deploy to Kubernetes

```bash
kubectl apply -f ./k8s/polnilnice-deployment.yaml
```

Make sure to select the right tag for the image to be used.




## Računi microservice

### Prerequisites

Create database container
```bash
docker run -d --name pg-racuni -e POSTGRES_USER=dbuser -e POSTGRES_PASSWORD=postgres -e POSTGRES_DB=racuni -p 5434:5432 postgres:13
```
Create docker container 
```shell
docker run -p 8082:8080 --network rsonet -e KUMULUZEE_DATASOURCES0_CONNECTIONURL=jdbc:postgresql://192.168.99.100:5434/racuni -e KUMULUZEE_CONFIG_CONSUL_AGENT=http://192.168.99.100:8500 --name placila-instance placila
```
Start database and admin-ms containers
```shell
docker start pg-racuni
docker start placila-instance
```

### Deploy to Kubernetes

```bash
kubectl apply -f ./k8s/placila-deployment.yaml
```

Make sure to select the right tag for the image to be used.




## Admin microservice

### Prerequisites

Running pg-admin postgres container (available on port 5435):
```shell
docker run -d --name pg-admin -e POSTGRES_USER=dbuser -e POSTGRES_PASSWORD=postgres -e POSTGRES_DB=admin -p 5435:5432 --network rsonet postgres:13
```
Or simply start the container if it already exsits:
```shell
docker start pg-admin
```

### Building the docker container

Clean and package the IntelliJ project
```shell
mvn clean package
```
Build the docker image locally
```shell
docker build -f .\Dockerfile_with_maven_build -t admin:latest .
```
### Running the docker container

#### Run docker container (available on port 8085)
Pass database and consul agent url's as parameters
```shell
docker run -p 8083:8080 --network rsonet -e KUMULUZEE_DATASOURCES0_CONNECTIONURL=jdbc:postgresql://pg-admin:5435/admin -e KUMULUZEE_CONFIG_CONSUL_AGENT=http://consul:8500 --name admin-instance admin
```
Use variables from .yaml file in api module.
```shell
docker run -p 8083:8080 --network rsonet --name admin-instance admin
```
Or simply run if you already created the container.
```shell
docker start admin-instance
```

### Deploy to Kubernetes

```bash
kubectl apply -f ./k8s/admin-deployment.yaml
```

Make sure to select the right tag for the image to be used.



## Iskanje microservice

This microservice sends emails whenever a new charging station is added.

### Build and run Docker containers

Building **novice** container:
```bash
docker build -f .\Dockerfile_with_maven_build -t iskanje:latest .
```

Run:
```bash
docker run -p 8084:8080 --network rsonet -e KUMULUZEE_CONFIG_CONSUL_AGENT=http://consul:8500 --name iskanje-instance novice
```

### Usage

Input the values inside Consul. Those values will be used when sending emails to subscribers.

#### Deploy to Kubernetes

```bash
kubectl apply -f ./k8s/iskanje-deployment.yaml
```

Make sure to select the right tag for the image to be used.




## Novice microservice

This microservice sends emails whenever a new charging station is added.

### Build and run Docker containers

Building **novice** container:
```bash
docker build -f .\Dockerfile_with_maven_build -t novice:latest .
```

Run:
```bash
docker run -p 8085:8080 --network rsonet -e KUMULUZEE_CONFIG_CONSUL_AGENT=http://consul:8500 --name novice-instance novice
```

### Usage

Input the values inside Consul. Those values will be used when sending emails to subscribers.

#### Deploy to Kubernetes

```bash
kubectl apply -f ./k8s/novice-deployment.yaml
```

Make sure to select the right tag for the image to be used.
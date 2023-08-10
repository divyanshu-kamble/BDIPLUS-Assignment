# Example Voting App

A simple distributed application running across multiple Docker containers.

## Introduction
This solution uses Python, Node.js, .NET, with Redis for messaging and Postgres for storage.

## Architecture

![Architecture diagram](architecture.excalidraw.png)

* A front-end web app in [Python](/vote) which lets you vote between two options
* A [Redis](https://hub.docker.com/_/redis/) which collects new votes
* A [.NET](/worker/) worker which consumes votes and stores them in…
* A [Postgres](https://hub.docker.com/_/postgres/) database backed by a Docker volume
* A [Node.js](/result) web app which shows the results of the voting in real time

##File Structure
The `3-tier Application` folder consist of all the application codebase. With the Jenkinsfile for CI build which pushes the image to docker registry in the repository divyanshuk.
The `Deployment` folder consist of 2 ways through which we can deploy the application -
1. Kubernets manifest files
2. Helm Charts

##How to Run the Application
####vote:
`docker build -t divyanshuk/vote:01 .`

####worker:
`docker build -t divyanshuk/worker:01 .`

####result:
`docker build -t divyanshuk/result:01 .`
---------
The sequence order must be followed as the application depend on each other for linking.
####Redis container running
    `docker run -d -h redis -p 6379:6379 --name redis redis:latest`
---------
####Postgres container running
    `docker run -d -h db --name postgres -e POSTGRES_DB=db -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=postgres -p 5432:5432 postgres:15-alpine`
---------
####Vote container running
    `docker run -d -p 5000:80 --link redis --name vote vote:1.0`
---------
####Worker container running
    `docker run -d --link postgres --link redis --name worker worker:1.0`
---------
####Result container running
    `docker run -d -p 5001:80 --link postgres --name result result:1.0`
---------
The `vote` app will be running at [http://localhost:5000](http://localhost:5000)
The `results` will be at [http://localhost:5001](http://localhost:5001).

## Run the app in Kubernetes
####helm deployment:

`helm upgrade --install voting-app /Deployments/helm-deployments/voting-application/ --atomic --values ${WORKSPACE}/environments/DEV/helm-deployments/voting-application/values.yaml --set deploy.vote_app.image.imageTag=${vote_app_tag} --set deploy.result_app.image.imageTag=${result_app_tag} --set deploy.worker_app.image.imageTag=${worker_app_tag} --namespace ${NAMESPACE}-voting-app`
------
####k8s deployment:
The `Deployment` directory consist of `Kubernetes` directory which consist of manifest files for all the microservices you can use `kubectl apply -f` with the directory name to deploy it on the cluster.

Currently there is no dns host so the application could only be accessed with port forwarding.
`kubectl port-forward vote-app [LOCAL_PORT:]REMOTE_PORT`
`kubectl port-forward result-app [LOCAL_PORT:]REMOTE_PORT`

## Notes

The voting application only accepts one vote per client browser. It does not register additional votes if a vote has already been submitted from a client.

This isn't an example of a properly architected perfectly designed distributed app... it's just a simple
example of the various types of pieces and languages you might see (queues, persistent data, etc), and how to
deal with them in Docker at a basic level.
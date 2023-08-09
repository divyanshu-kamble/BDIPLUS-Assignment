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
The `Deployment` folder consist of 3 ways through which we can deploy the application -
1. Ansible
2. Kubernets manifest files
3. Helm Charts

##Running the Application
The `vote` app will be running at [http://localhost:5000](http://localhost:5000)
The `results` will be at [http://localhost:5001](http://localhost:5001).

## Run the app in Kubernetes
Currently there is no dns host so the application could only be accessed with port forwarding.

## Notes

The voting application only accepts one vote per client browser. It does not register additional votes if a vote has already been submitted from a client.

This isn't an example of a properly architected perfectly designed distributed app... it's just a simple
example of the various types of pieces and languages you might see (queues, persistent data, etc), and how to
deal with them in Docker at a basic level.
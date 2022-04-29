# Deploying a Flask API and MySQL server on Kubernetes

This repo contains code that 
1) Deploys a MySQL server on a Kubernetes cluster
2) Attaches a persistent volume to it, so the data remains contained if pods are restarting
3) Deploys a Flask API to add, delete and modify users in the MySQL database

## Prerequisites
1. Have `Docker` and the `Kubernetes CLI` (`kubectl`) installed together with `Minikube` (https://kubernetes.io/docs/tasks/tools/)

## Getting started
1. Clone the repository
2. Configure `Docker` to use the `Docker daemon` in your kubernetes cluster via your terminal: `eval $(minikube docker-env)`
3. Pull the latest mysql image from `Dockerhub`: `Docker pull mysql`
4. Build a kubernetes-api image with the Dockerfile in this repo: `Docker build . -t flask-api`

## Secrets
`Kubernetes Secrets` can store and manage sensitive information. For this example we will define a password for the
`root` user of the `MySQL` server using the `Opaque` secret type. For more info: https://kubernetes.io/docs/concepts/configuration/secret/

1. Encode your password in your terminal: `echo -n super-secret-passwod | base64`
2. Add the output to the `flakapi-secrets.yml` file at the `db_root_password` field

## Deployments
Get the secrets, persistent volume in place and apply the deployments for the `MySQL` database and `Flask API`

1. Add the secrets to your `kubernetes cluster`: `kubectl apply -f flaskapi-secrets.yml, kubectl apply -f mysql-secrets.yml`
2. Add mysql db by applying configmap, this will create a database in mysql pod if it does not exist `kubectl apply -f mysql-configmap.yml`
3. Create the `persistent volume` and `persistent volume claim` for the database: `kubectl apply -f mysql-pv.yml`
4. Create the `MySQL` deployment: `kubectl apply -f mysql-deployment.yml`
5. Create the `Flask API` deployment: `kubectl apply -f flaskapp-deployment.yml`

You can check the status of the pods, services and deployments.

## Expose the API
The API can be accessed by exposing it using minikube: `minikube service flask-service`. This will return a `URL`. If you paste this to your browser you will see the `hello world` message. You can use this `service_URL` to make requests to the `API`

## Start making requests
Now you can use the `API` to `CRUD` your database
1. add a user: `curl -H "Content-Type: application/json" -d '{"name": "<user_name>", "email": "<user_email>", "pwd": "<user_password>"}' <service_URL>/create`
2. get all users: `curl <service_URL>/users`
3. get information of a specific user: `curl <service_URL>/user/<user_id>`
4. delete a user by user_id: `curl -H "Content-Type: application/json" <service_URL>/delete/<user_id>`
5. update a user's information: `curl -H "Content-Type: application/json" -d {"name": "<user_name>", "email": "<user_email>", "pwd": "<user_password>", "user_id": <user_id>} <service_URL>/update`

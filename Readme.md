# Docker - Kubernetes WordPress

This project was created in order to explain how to setup a wordpress installation with docker, then, move it to kubernetes

## What do we need?

 - [docker](https://www.docker.com/products/docker-desktop)

## Usage

Run the following:

```bash
docker-compose up -d
```
To bring it down and preserve your data

```bash
docker-compose down
```

To bring down all

```bash
docker-compose down --volumes
```
So, to explain it, we connect to the wordpress instance by [http://127.0.0.1:8000/](http://127.0.0.1:8000/), as we see on the compose file, we are connecting our 8000 port to the 80 port that's inside the container.

```yml
     ports:
       - "8000:80"
```
we are creating the environment variables that the wordpress image needs, and saying to docker that wordpress depend on the mysql image creation.

On the mysql side, we are doing something similar (talking about the ports and env vars), but in this case we are preserving our data in a local volume

```yml
     environment:
       MYSQL_ROOT_PASSWORD: somewordpress
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: wordpress
```

![alt text](https://github.com/yosoyvilla/docker-k8s-wordpress/blob/develop/docker-compose.png?raw=true)

## Tips

 - You can translate from docker compose files to kubernetes files using [Kompose](https://kubernetes.io/docs/tasks/configure-pod-container/translate-compose-kubernetes/)
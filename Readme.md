# Docker - Vault - Kubernetes WordPress

This project was created in order to explain how to set up a Wordpress installation with docker, then, move it to Kubernetes.

## What do we need?

 - [docker](https://www.docker.com/products/docker-desktop)
 - [minikube](https://kubernetes.io/docs/setup/learning-environment/minikube/#installation)

## Installing Vault (Our secrets manager)

we will use the Filesystem backend to store secrets at rest.

this method storage the data on the filesystem using a standard directory structure. It can be used for durable single server situations, or to develop locally where durability is not critical.

## Installing and unsealing Vault

In order to start working with Vault, we will create our infra:

```bash
docker-compose up -d --build
```

Then, start a bash session into the running container:

```bash
docker-compose exec vault bash
```

initialize Vault:

```bash
vault operator init
```

This command initializes the vault server, that means Vault's storage backend is prepared to receive data.

During this initialization, Vault generates an in-memory master key and applies [Shamir's secret sharing](https://en.wikipedia.org/wiki/Shamir%27s_Secret_Sharing) algorithm to disassemble that master key into a configuration number of key shares such that a configurable subset of those key shares must come together to regenerate the master key. In other words, instead of distributing this master key as a single key to an operator, Vault uses an algorithm known as Shamir's Secret Sharing to split the key into shards. [Vault Architecture](https://www.vaultproject.io/docs/internals/architecture)

Note that vault operator init command cannot be run against already-initialized Vault cluster.

Take note of the unseal keys and the initial root token. We will need to provide three of the unseal keys every time the Vault server is re-sealed or restarted.

Now we can unseal Vault using three of the keys:

```bash
vault operator unseal <the_key>
```

Once a Vault is unsealed, it remains unsealed until one of two things happens: either it is resealed via the API or the server is restarted.

Let's authenticate using the root token:

```bash
vault login <root_key>
```

## Auditing

First, we need to enable an Audit Device which keeps a detailed log of all requests and response to Vault:

```bash
vault audit enable file file_path=/vault/logs/audit.log
```

if you want to see the logs:

```bash
vault audit list
```

## Secrets

There are two types of secrets in Vault:

  - [Static Secrets](https://www.vaultproject.io/docs/secrets): Like Redis or Memcahched, those have refresh intervals but they do not expire unless explicitly revoked.
  - [Dynamic Secrets](https://www.vaultproject.io/docs/secrets): Generated on demand, they don't exists until they are accessed.

we can create our secrets using: CLI, API or UI.

In this example we will use UI:

![Secret Creation](https://github.com/yosoyvilla/docker-k8s-wordpress/blob/develop/create-secret.png?raw=true)


## Usage

In order to list your secrets:

```bash
docker secret ls
```

To bring it up:

```bash
docker-compose up -d
```
To bring it down and preserve your data

```bash
docker-compose down
```

To bring down all (removing conatainers, data, networks)

```bash
docker-compose down --volumes
```
So, to explain it, we connect to the WordPress instance by [http://127.0.0.1:8000/](http://127.0.0.1:8000/), as we see on the compose file, we are connecting our 8000 port to the 80 port that's inside the container.

```yml
     ports:
       - "8000:80"
```
We are creating the environment variables that the WordPress image needs, and saying to docker that WordPress depends on the MySQL image creation.

On the MySQL side, we are doing something similar (talking about the ports and env vars), but in this case, we are preserving our data in a local volume

```yml
     environment:
       MYSQL_ROOT_PASSWORD: somewordpress
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: wordpress
```

![Docker Diagram](https://github.com/yosoyvilla/docker-k8s-wordpress/blob/develop/docker-compose.png?raw=true)

## Tips

 - You can translate from docker-compose files to Kubernetes files using [Kompose](https://kubernetes.io/docs/tasks/configure-pod-container/translate-compose-kubernetes/), please, keep in mind that you should modify the generated files in order to adapt it to what you really need.
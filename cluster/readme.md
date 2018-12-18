Cluster Environment
===================

The cluster environment is served using a Kubernetes cluster. To make your lives easier, we suggest to use a managed Kuberentes cluster from a cloud provider. This configuration is tests with the Kubernetes cluster from [Digital Ocean](https://www.digitalocean.com/products/kubernetes/), but in theory it should work with all other cloud providers as well.

A managed Kubernetes cluster has the advantages, that you don't have to setup and maintain the cluster with all its nodes. You only have to tell the cluster manager, which services should it serve and on how many nodes the application should be running on.

## Table of content

- [Prerequisites](#prerequisites)
- [Set up a Load Balancer and Traefik](#set-up-a-load-balancer-and-traefik)
- [Set up a Service](#set-up-a-service)

## Prerequisites

- Kubernetes cluster
- Docker registry (private or public)

## Set up a Load Balancer and Traefik

When running the applications inside a cluster, a load balancer is required to forward the requests to all nodes that take part of the cluster. In addition, Traefik is used as an ingress controller to then route the requests coming from the load balancer to the appropriate services.

1. Execute the following command to create a namespace:

    ```bash
    kubectl create namespace traefik
    ```

2. Copy the file `cluster/traefik-load-balancer-remote-example.yml` to `cluster/traefik-load-balancer-remote.yml` and adjust the following line with your email address:

    ```yml
    [acme]
      email = "your@email.com"
    ```

3. Execute the following command to create the load balancer:

    ```bash
    kubectl apply -f cluster/traefik-load-balancer-remote.yml
    ```

4. Point the domain name of the Traefik dashboard service (ex. traefik.aboutbits.it) to the IP of the load balancer created in the previous step.

    - You receive the IP of the load balancer at the administration page of your cloud provider.
    - Wait until the DNS changes are available, before going on to the next step.

5. Copy the file `cluster/traefik-dashboard-remote-example.yml` to `cluster/traefik-dashboard-remote.yml`.

6. Create a htpasswd file with all user and their credentials, that should have access to the Traefik dashboard. Place the file under `secrets/traefik/auth`.

    ```bash
    touch secrets/traefik/auth
    htpasswd -B secrets/traefik/auth alex
    ```

    Note: On Ubuntu the package `apache2-utils` is required to run the commands.

7. Add the htpasswd file to the cluster by executing the following command:

    ```bash
    kubectl create secret generic traefik-auth --from-file=auth=secrets/traefik/auth --namespace=traefik --dry-run -o yaml | kubectl apply -f -
    ```

8. Start the Traefik dashboard by executing the following command:

    ```bash
    kubectl apply -f cluster/traefik-dashboard-remote.yml
    ```

## Set up a Service

After the initial setup of the load balancer and Traefik, we can then configure and run a service inside the cluster. An example configuration of a service, that should be running inside the cluster, can be found in the directory `example`. Copy the files to your project and adjust the configuration. In addition, the images of the containers should already be stored in an accessible Docker registry.

1. Execute the following command to create a namespace:

    ```bash
    kubectl create namespace example
    ```

2. Create a secret with the credentials of the private Docker registry, so that Kubernetes can access and pull the images from the registry:

    ```bash
    kubectl create secret docker-registry gitlab-registry --docker-server=https://registry.gitlab.com  --docker-email=<your-email> --docker-username=<your-username> --docker-password=<your-password> --namespace=example
    ```

    Note: This is only necessary if you store the images on a private registry.

3. Adjust the following lines of the file `example/kubernetes-remote.yml` and set the right domain name of the service:

    ```yml
    rules:
    - host: example.aboutbits.it
    ```

4. Point the domain name of the service (ex. example.aboutbits.it) to the IP of the load balancer created previously.

    - You receive the IP of the load balancer at the administration page of your cloud provider.
    - Wait until the DNS changes are available, before going on to the next step.

5. Start the service by executing the following command:

    ```bash
    kubectl apply -f example/kubernetes-remote.yml
    ```

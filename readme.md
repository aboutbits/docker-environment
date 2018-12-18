Docker Environment
==================

This project includes a Docker setup for all your environments, so that your are able to use the same containers for your development, testing and production environment.

It provides an easy to use development configuration, where you can quickly add and remove services. Also running multiple services that are all accessible through the ports 80 and 443 of your local machine is possible.

In addition, it provides a predefined configuration for a Kubernetes cluster that can host your containers. It is tested with a Kuberenetes cluster provided by [Digital Ocean](https://www.digitalocean.com/products/kubernetes/), but in theory it should work also with other cloud providers.

## Table of content

- [Prerequisites](#prerequisites)
- [Local Environmet](#local-environment)
- [Cluster Environment](#cluster-environment)
- [Information](#information)

## Local Environment

The setup of the local development environment consists of two parts. One part are the services that you develop by yourself and which serve a specific application. The other part is a reverse proxy, that binds to your local ports 80 and 443, and distributes all incoming requests to the appropriate services.

Therefore, this project uses Traefik as a reverse proxy and provides already a predefined setup and configuration for the tool. Then, Traefik scans automatically all the running Docker containers connected to a prespecified Docker network, registers them based on their labeled domains and redirects all request to the appropriate containers. In addition, the usage of SSL certificates is already preconfigured into the proxy.

The detailed documentation about the local environment can be found [here](local/readme.md).

## Cluster Environment

The cluster environment is served using a Kubernetes cluster. To make your lives easier, we suggest to use a managed Kuberentes cluster from a cloud provider. This configuration is tests with the Kubernetes cluster from [Digital Ocean](https://www.digitalocean.com/products/kubernetes/), but in theory it should work with all other cloud providers as well.

A managed Kubernetes cluster has the advantages, that you don't have to setup and maintain the cluster with all its nodes. You only have to tell the cluster manager, which services should it serve and on how many nodes the application should be running on.

The detailed documentation about the cluster environment can be found [here](cluster/readme.md).

## Information

About Bits is a company based in South Tyrol, Italy. You can find more information about us on [our website](https://aboutbits.it).

### Support

For support, please contact [info@aboutbits.it](mailto:info@aboutbits.it).

### Credits

- [Alex Lanz](https://github.com/alexlanz)
- [All Contributors](../../contributors)

### License

The MIT License (MIT). Please see [License File](license.md) for more information.

Local Environment
=================

The setup of the local development environment consists of two parts. One part are the services that you develop by yourself and which serve a specific application. The other part is a reverse proxy, that binds to your local ports 80 and 443, and distributes all incoming requests to the appropriate services.

Therefore, this project uses Traefik as a reverse proxy and provides already a predefined setup and configuration for the tool. Then, Traefik scans automatically all the running Docker containers connected to a prespecified Docker network, registers them based on their labeled domains and redirects all request to the appropriate containers. In addition, the usage of SSL certificates is already preconfigured into the proxy.

## Table of content

- [Prerequisites](#prerequisites)
- [Set up Traefik](#set-up-traefik)
- [Set up a Service](#set-up-a-service)
- [Running Traefik](#running-traefik)
    - [Start Traefik](#start-traefik)
    - [Reload Traefik](#reload-traefik)
    - [Stop Traefik](#stop-traefik)
- [Documentation](#documentation)

## Prerequisites

- Docker (or Docker for Mac/Windows)

## Set up Traefik

The Treafik container is a small and lightweight container, that automatically detects new services when they were started and redirects all the requests to the appropriate containers. The setup steps for a service can be found in the next section. In this section, we will guide you through the required steps to configure this Traefik container. Don't be afraid to keep the Traefik container always running on your machine. It does not need much resources and it restarts automatically if you don't stop it explicitly.

1. Edit the `/etc/hosts` file and point the domain for accessing the Traefik dashboard to localhost:

    ```
    127.0.0.1 traefik.aboutbits.local
    ```

2. Copy the file `local/docker-compose-example.yml` to `local/docker-compose.yml` and adjust the following lines with your desired domain name for the Traefik dashboard:

    ```yml
    labels:
      - traefik.frontend.rule=Host:traefik.aboutbits.local
      - traefik.port=8080
    ```

3. Paste the SSL certificate of the domain for accessing the Traefik dashboard in the folder `certificates`.

    Note: if you are searching for a solution to use certificates on your local development, check out the [certificate authority tools](https://github.com/aboutbits/certificate-authority-tools) that help you create your own certificate authorities. Usinge these tools you can generate and sign your own certificates that you then can use for development.

4. Copy the file `local/traefik-example.toml` to `local/traefik.toml` and adjust the certificate paths with the files that you copied in the previous step:

    ```toml
    [entryPoints.https.tls]
      [[entryPoints.https.tls.certificates]]
      keyFile = "/var/certificates/traefik.aboutbits.local.key"
      certFile = "/var/certificates/traefik.aboutbits.local.pem"
    ```

5. Create a Docker network for the Traefik proxy by executing the following command:

    ```bash
    cd local && docker network create proxy
    ```

6. Run the Traefik container by executing the following command:

    ```bash
    cd local && docker-compose up --detach
    ```

7. You can now access the Traefik dashboard through your browser by accessing the configured domain name.

## Set up a Service

Normally you would configure a project with a Docker container bound to a specific port (ex. 80, 443). Using this port you can then access the application. However, what if you want to start two projects locally and you want to bind them both on the same port (ex. 80, 443). This is not possible using the default Docker pratices. Therefore, we have to use Traefik as a proxy.

We can easily set up a project and don't bind the port of the project to the ports of our machine. Instead, we can connect a container to a prespecified Docker network and give it additional parameters using labels. These labels indicate details like the domain name of the service. An example project can be found in the folder `example`. Inside the file `example/docker-compose.yml` there are two things to keep in mind:

- Use an external and a internal network. Only create the containers that should be accessed from the Traefik container to the `proxy` network and connect all other services together internally by assigning them to the `internal` network.

- Add the labels to the container that should be accessed through the Traefik container and configure the host, port and backend name.

After that, you have to modify the Traefik configuration with some details about the service:

1. Edit the `/etc/hosts` file and point the domain of the service to localhost:

    ```
    127.0.0.1 traefik.aboutbits.local example.aboutbits.local
    ```

2. Paste the SSL certificate for the domain in the folder `certificates`.

    Note: if you are searching for a solution to use certificates on your local development, check out the [certificate authority tools](https://github.com/aboutbits/certificate-authority-tools) that help you create your own certificate authorities. Usinge these tools you can generate and sign your own certificates that you then can use for development.

3. Edit the file `local/traefik.toml` and add the new certificates that you copied in the previous step:

    ```toml
    [entryPoints.https.tls]
      [[entryPoints.https.tls.certificates]]
      keyFile = "/var/certificates/traefik.aboutbits.local.key"
      certFile = "/var/certificates/traefik.aboutbits.local.pem"
      [[entryPoints.https.tls.certificates]]
      keyFile = "/var/certificates/service.aboutbits.local.key"
      certFile = "/var/certificates/service.aboutbits.local.pem"
    ```

4. Reload the Traefik container by executing the following command:

    ```bash
    cd local && docker-compose up --detach --force-recreate
    ```

5. After these steps you can start the Docker containers of your service.

## Running Traefik

To start, reload or stop the Traefik container, simply execute one of the following command in your terminal:

### Start Traefik

```bash
cd local && docker-compose up --detach
```

### Reload Traefik

```bash
cd local && docker-compose up --detach --force-recreate
```

### Stop Traefik

```bash
cd local && docker-compose down
```

## Documentation

Additional information about the usage of Traefik with Docker can be found [here](https://docs.traefik.io/configuration/backends/docker/). There you can find additional configuration settings and possible security concerns, that should be considered.

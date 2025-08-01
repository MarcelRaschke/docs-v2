---
title: Install and run InfluxDB 3 Explorer
description: >
  Use [Docker](https://docker.com) to install and run **InfluxDB 3 Explorer**.
menu:
  influxdb3_explorer:
    name: Install Explorer
weight: 2
---

Use [Docker](https://docker.com) to install and run **InfluxDB 3 Explorer**.

<!-- BEGIN TOC -->
- [Run the InfluxDB 3 Explorer Docker container](#run-the-influxdb-3-explorer-docker-container)
- [Enable TLS/SSL (HTTPS)](#enable-tlsssl-https)
- [Pre-configure InfluxDB connection settings](#pre-configure-influxdb-connection-settings)
- [Run in query or admin mode](#run-in-query-or-admin-mode)
  - [Run in query mode](#run-in-query-mode)
  - [Run in admin mode](#run-in-admin-mode)
- [Environment Variables](#environment-variables)
- [Volume Reference](#volume-reference)
- [Exposed Ports](#exposed-ports)
  - [Custom port mapping](#custom-port-mapping)
<!-- END TOC -->

## Run the InfluxDB 3 Explorer Docker container

1.  **Install Docker**

    If you haven't already, install [Docker](https://docs.docker.com/engine/) or
    [Docker Desktop](https://docs.docker.com/desktop/).

2.  **Pull the {{% product-name %}} Docker image**

    ```bash
    influxdata/influxdb3-ui:{{% latest-patch %}}
    ```

3.  **Create local directories** _(optional)_

    {{% product-name %}} can mount the following directories on your local
    machine:

    | Directory  | Description                                                                                       | Permissions |
    | :--------- | :------------------------------------------------------------------------------------------------ | :---------: |
    | `./db`     | Stores {{% product-name %}} application data                                                      |     700     |
    | `./config` | Stores [pre-configured InfluxDB connection settings](#pre-configure-influxdb-connection-settings) |     755     |
    | `./ssl`    | Stores TLS/SSL certificates _(Required when [using TLS/SSL](#enable-tlsssl-https))_               |     755     |

    > [!Important]
    > If you don't create and mount a local `./db` directory, {{% product-name %}}
    > stores application data in the container's file system.
    > This data will be lost when the container is deleted.

    To create these directories with the appropriate permissions, run the
    following commands from your current working directory:

    ```bash
    mkdir -m 700 ./db
    mkdir -m 755 ./config
    mkdir -m 755 ./ssl
    ```

4.  **Run the {{% product-name %}} container**

    Use the `docker run` command to start the {{% product-name %}} container.
    Include the following:

    - Port mappings:
      - `8888` to `80` (or `443` if using TLS/SSL)
      - `8889` to `8888`
    - Mounted volumes:
      - `$(pwd)/db:/db:rw`
      - `$(pwd)/config:/app-root/config:ro`
      - `$(pwd)/ssl:/etc/nginx/ssl:ro`
    - Any of the available [environment variables](#environment-variables)
      
      > [!Note]
      > To persist sessions across container restarts, see the detailed instructions
      > on setting the [`SESSION_SECRET_KEY` environment variable](#session_secret_key).

    ```bash
    docker run --detach \
      --name influxdb3-explorer \
      --publish 8888:80 \
      --publish 8889:8888 \
      --volume $(pwd)/config:/app-root/config:ro \
      --volume $(pwd)/db:/db:rw \
      --volume $(pwd)/ssl:/etc/nginx/ssl:ro \
      influxdata/influxdb3-ui:{{% latest-patch %}} \
      --mode=admin
    ```

5.  **Access the {{% product-name %}} user interface (UI) at <http://localhost:8888>**.

---

## Enable TLS/SSL (HTTPS)

To enable TLS/SSL, mount valid certificate and key files into the container:

1.  **Place your TLS/SSL certificate files your local `./ssl` directory**

    Required files:

    - Certificate: `server.crt` or `fullchain.pem`
    - Private key: `server.key`

2.  **When running your container, mount the SSL directory and map port 443 to port 8888**
   
    Include the following options when running your Docker container:

    ```sh
    --volume $(pwd)/ssl:/etc/nginx/ssl:ro \
    --publish 8888:443
    ```

The nginx web server automatically uses certificate files when they are present
in the mounted path.

> [!Note]
> You can use a custom location for certificate and key files.
> Use the [`SSL_CERT_PATH`](#ssl_cert_path) and [`SSL_KEY_PATH`](#ssl_key_path)
> environment variables to identify the custom location.
> Also update the SSL directory volume mount path inside the container.


---

## Pre-configure InfluxDB connection settings

You can predefine InfluxDB connection settings using a `config.json` file.

{{% code-placeholders "INFLUXDB3_HOST|INFLUXDB_DATABASE_NAME|INFLUXDB3_AUTH_TOKEN|INFLUXDB3_SERVER_NAME" %}}

1.  **Create a `config.json` file in your local `./config` directory**

    ```json
    {
      "DEFAULT_INFLUX_SERVER": "INFLUXDB3_HOST",
      "DEFAULT_INFLUX_DATABASE": "INFLUXDB_DATABASE_NAME",
      "DEFAULT_API_TOKEN": "INFLUXDB3_AUTH_TOKEN",
      "DEFAULT_SERVER_NAME": "INFLUXDB3_SERVER_NAME"
    }
    ```

    > [!Important]
    > If connecting to an InfluxDB 3 Core or Enterprise instance running on
    > localhost (outside of the container), use the internal Docker network to
    > in your InfluxDB 3 host value--for example:
    >
    > ```txt
    > http://host.docker.internal:8181
    > ```

2.  **Mount the configuration directory**

    Include the following option when running your Docker container:

    ```sh
    --volume $(pwd)/config:/app-root/config:ro
    ```

{{% /code-placeholders %}}

These settings will be used as defaults when the container starts.

---

## Run in query or admin mode

{{% product-name %}} has the following operational modes:

- **Query mode (default):** Read-only UI and query interface
- **Admin mode:** Full UI and API access for administrators

You can control the operational mode using the `--mode=` option in your
`docker run` command (after the image name).

### Run in query mode {note="(default)"}

```sh
docker run \
  ...
  --mode=query
```

### Run in admin mode

```sh
docker run \
  ...
  --mode=admin
```

If `--mode` is not set, the container defaults to query mode.

---

## Environment Variables

Use the following environment variables to customize {{% product-name %}} settings
in your container.

- [DATABASE_URL](#database_url)
- [SESSION_SECRET_KEY](#session_secret_key)
- [SSL_CERT_PATH](#ssl_cert_path)
- [SSL_KEY_PATH](#ssl_key_path)

### DATABASE_URL

Path to SQLite DB inside container. The default is `/db/sqlite.db`.

{{< expand-wrapper >}}
{{% expand "View `DATABASE_URL` example" %}}
<!-- pytest.mark.skip -->

```bash
docker run --detach \
  # ...
  --volume $(pwd)/db:/custom/db-path:rw \
  --env DATABASE_URL=/custom/db-path/sqlite.db \
  influxdata/influxdb3-ui:{{% latest-patch %}}
```
{{% /expand %}}
{{< /expand-wrapper >}}

### SESSION_SECRET_KEY

Specifies the secret key for session management. If none is provided, Explorer
uses a random 32-byte hex string as the session secret key.

{{< expand-wrapper >}}
{{% expand "View `SESSION_SECRET_KEY` example" %}}
<!-- pytest.mark.skip -->

```bash
docker run --detach \
  # ...
  --env SESSION_SECRET_KEY=xxX0Xx000xX0XxxxX0Xx000xX0XxX00x \
  influxdata/influxdb3-ui:{{% latest-patch %}}
```
{{% /expand %}}
{{< /expand-wrapper >}}

> [!Important]
> #### Always set SESSION_SECRET_KEY in production
>
> When you restart the container, {{% product-name %}} generates a new key if
> not explicitly set. For production use cases, always set the `SESSION_SECRET_KEY`
> environment variable to persist sessions across restarts.

### SSL_CERT_PATH

Defines the path to the SSL certificate file inside the container.
Default is `/etc/nginx/ssl/cert.pem`.

{{< expand-wrapper >}}
{{% expand "View `SSL_CERT_PATH` example" %}}
<!-- pytest.mark.skip -->

```bash
docker run --detach \
  # ...
  --volume $(pwd)/ssl:/custom/ssl:ro \
  --env SSL_CERT_PATH=/custom/ssl/cert.pem \
  influxdata/influxdb3-ui:{{% latest-patch %}}
```
{{% /expand %}}
{{< /expand-wrapper >}}

### SSL_KEY_PATH

Defines the path to the SSL private key file inside the container.
Default is `/etc/nginx/ssl/key.pem`.

{{< expand-wrapper >}}
{{% expand "View `SSL_KEY_PATH` example" %}}
<!-- pytest.mark.skip -->

```bash
docker run --detach \
  # ...
  --volume $(pwd)/ssl:/custom/ssl:ro \
  --env SSL_KEY_PATH=/custom/ssl/key.pem \
  influxdata/influxdb3-ui:{{% latest-patch %}}
```
{{% /expand %}}
{{< /expand-wrapper >}}

## Volume Reference

| Container Path       | Purpose                      | Host Example               |
|----------------------|------------------------------|----------------------------|
| `/db`                | SQLite DB storage            | `./db`                     |
| `/app-root/config`   | JSON config for defaults     | `./config`                 |
| `/etc/nginx/ssl`     | SSL certs for HTTPS          | `./ssl`                    |

## Exposed Ports

| Port | Protocol | Purpose                 |
|------|----------|-------------------------|
| 80   | HTTP     | Web access (unencrypted) |
| 443  | HTTPS    | Web access (encrypted)   |

### Custom port mapping

```sh
# Map ports to custom host values
--publish 8888:80 --publish 8443:443
```

# docker-nginx-certbot-proxy

A lightweight companion container for the nginx-proxy. It allows the creation/renewal of CertBot(Let's Encrypt) certificates automatically.

## Usage

First, you must declare a writable volumes to create/renew CertBot certificates. The default path is `/usr/local/nginx/certs`. See and modify the `.env` file if you want to make some changes.

Then, create the network `nginx-proxy` if you want to use  multiple sets of services.

```sh
$ docker create network nginx-proxy
```

After that, run containers via docker-compose:

```sh
$ docker-compose up -d
```

Three containers will be up, check it:

```console
$ docker-compose ps
    Name                   Command               State                    Ports
-------------------------------------------------------------------------------------------------
nginx-certbot   /bin/bash /app/entrypoint. ...   Up
nginx-gen       /usr/local/bin/docker-gen  ...   Up
nginx-root      nginx -g daemon off;             Up      0.0.0.0:443->443/tcp, 0.0.0.0:80->80/tcp
```

NOTE: The first time this container is launched it generates a new Diffie-Hellman group file. This process can take several minutes to complete (be patient).

Finally, start any containers you want proxied with a env var `VIRTUAL_HOST=subdomain.youdomain.com`

Example:

```sh
$ docker run -d --net=nginx-proxy \
--name example-app \
-e "VIRTUAL_HOST=example.com,www.example.com" \
nginx:alpine
```

The containers being proxied must expose the port to be proxied, either by using the `EXPOSE` directive in their `Dockerfile` or by using the `--expose` flag to `docker run` or `docker create`.

### Multiple Ports

If your container exposes multiple ports, nginx-proxy will default to the service running on port 80. If you need to specify a different port, you can set a `VIRTUAL_PORT` env var to select a different one. If your container only exposes one port and it has a `VIRTUAL_HOST` env var set, that port will be selected.

Example:

```sh
$ docker run -d --net=nginx-proxy \
--name example-app \
-e "VIRTUAL_HOST=example.com,www.example.com" \
-e "VIRTUAL_PORT=8080" \
bingozb/springboot
```

### CertBot

All you need is to add the environment `LETSENCRYPT_HOST` and `LETSENCRYPT_EMAIL`.

Example:

```sh
$ docker run -d --net=nginx-proxy \
--name example-app \
-e "VIRTUAL_PORT=8080" \
-e "VIRTUAL_HOST=example.com,www.example.com" \
-e "LETSENCRYPT_HOST=example.com,www.example.com" \
-e "LETSENCRYPT_EMAIL=your@mail.com" \
bingozb/springboot
```

To use the Certbot service to automatically create a valid certificate for virtual host(s), declare the `LETSENCRYPT_HOST` environment variable in each to-be-proxied application containers.

The `LETSENCRYPT_HOST` variable most likely needs to be set to the same value as the `VIRTUAL_HOST` variable and must be publicly reachable domains. Specify multiple hosts with a comma delimiter.

## Correlation

- docker-letsencrypt-nginx-proxy-companion https://github.com/bingozb/docker-letsencrypt-nginx-proxy-companion
- docker-gen https://github.com/bingozb/docker-gen
- nginx-proxy https://github.com/bingozb/nginx-proxy


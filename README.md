# NGINX Config for Subdomains in Local Development (SSL Version)

A simple NGINX config for setting up sub domains in local development.

The target environment is linux operating systems. If you are on Windows or Mac remove `extra_hosts` from `docker-compose.yaml`, as this value is set automatically.

By default `nginx.conf` is set to route `dev.localhost:4000` to `localhost:3000`.

## Prerequisites

You need to have docker & docker-compose installed on your machine.

## Installation

### Clone

```bash
git clone https://github.com/nvme0/nginx-config-local-dev.git

```

### Modify docker-compose.yaml

On a linux OS, get your host IP:

```bash
ifconfig | grep -E "([0-9]{1,3}\.){3}[0-9]{1,3}" | grep -v 127.0.0.1 | awk '{ print $2 }' | cut -f2 -d: | head -n1
```

In `docker-compose.yaml` under `extra_hosts`, change `127.18.0.1` to your host IP.

**Remove `extra_hosts` if your are on Windows or MacOS.**

#### (optional) Modify the default port

By default the NGINX server will be accessable on port `4000` of your local machine. To change this, under `ports` change `4000` to your desired port.

### Modify nginx.conf

### Add subdomain to /etc/hosts

Add `127.0.0.1 dev.localhost` to `/etc/hosts`. Change `dev` to your subdomain.

### Setting Up SSL

Create a self-signed key & certificate pair with OpenSSL:

```bash
openssl req -x509 -nodes -new -sha256 -days 1024 -newkey rsa:2048 -keyout RootCA.key -out RootCA.pem -subj "/C=US/CN=Example-Root-CA"
```

```bash
openssl x509 -outform pem -in RootCA.pem -out RootCA.crt
```

(optional) Modify alt_names to domains.ext

```bash
[alt_names]
DNS.1 = localhost
DNS.2 = dev.localhost
DNS.3 = ...
```

```bash
openssl req -new -nodes -newkey rsa:2048 -keyout ./ssl/private/localhost.key -out localhost.csr -subj "/C=US/ST=YourState/L=YourCity/O=Example-Certificates/CN=localhost.local"
```

```bash
openssl x509 -req -sha256 -days 1024 -in localhost.csr -CA RootCA.pem -CAkey RootCA.key -CAcreateserial -extfile domains.ext -out ./ssl/certs/localhost.crt
```

Change the owner of `localhost.key` to `www-data`

```bash
sudo chown www-data ./ssl/private/localhost.key
```

### Trust the local CA

#### Firefox

Import the certificate by going to `about:preferences#privacy` > `View Certificates` > `Authorities` > `Import...` > `RootCA.pem` > Modify trust settings and confirm.

Sources: <https://gist.github.com/cecilemuller/9492b848eb8fe46d462abeb26656c4f8>

#### Chrome

Import the certificate by going to `chrome://settings/certificates` > `Authorities` > `Import` > Modify trust settings and confirm.

Restart Chrome Browser to get green lock.

## Usage

Start your dev server on `localhost:3000`, then spin up the NGINX server with docker-compose.

```bash
docker-compose up -d
```

Start your dev server, and navigate to `dev.localhost:4000` in your browser. It should serve your dev server on `localhost:3000`.

## Troubleshooting

Check your logs `logs/error.log`.

### host not found in upstream "host.docker.internal:3000"

If you get the following error in `logs/error.log`:

```bash
2020/08/09 14:45:35 [emerg] 1#1: host not found in upstream "host.docker.internal:3000" in /etc/nginx/conf.d/nginx.conf:2
```

Then either your dev server is not running on the specified port, or your host IP in `docker-compose.yaml` -> `extra_hosts` is incorrect.

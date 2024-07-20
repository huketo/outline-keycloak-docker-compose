# Outline(notion alternative) + Keycloak(OIDC), Docker compose with Nginx

🚧 Change variables in the `.env.sample` to meet your requirements.

## Deploy

Create docker network:

`docker network create keycloak-network`

`docker network create outline-network`

### Deploy Keycloak:

```bash
cd keycloak
docker-compose -p keycloak up -d
```

Create a realm, client, and user.

**realm** and **client** are case-sensitive, name these `outline`.

Create a client

1. Client type: `OpenID Connect`
2. Client ID: `outline`
3. Client authentication: `on`
4. Authentication flow: uncheck all, leave `Standard flow` checked
5. Set URLs:
  - Root URL: `https://outline.example.com`
  - Home URL: `https://outline.example.com`
  - Valid Redirect URIs: `https://outline.example.com/auth/oidc.callback`

🛠️ Get `Client secret` value on the `Credentials` tab of the `Client`

Specify the `OUTLINE_OIDC_CLIENT_SECRET` in the `.env` file.

Create a user on Keycloak for Outline.

### Deploy Outline:

Sets the Outline to communicate outside the container.

Check Keycloak's docker gateway IP address and update the `OUTLINE_OIDC_DOCKER_GATEWAY` in the `.env` file.

Change Hostname for `outline.example.com`

`127.0.0.1 outline.example.com`

```bash
sudo nano /etc/hosts
```

Deploy Outline with Docker Compose:

```bash
cd outline
docker-compose -p outline up -d
```

## Nginx

### Create certificates using Certbot

```bash
certbot certonly --nginx -d auth.example.com
certbot certonly --nginx -d outline.example.com
certbot certonly --nginx -d minio.example.com
certbot certonly --nginx -d minio-console.example.com
```

### Create configuration files for Nginx

`keycloak/auth.conf`, `outline/outline.conf`, `outline/minio.conf`, `outline/minio-console.conf` are the sample configuration files for Nginx.

Copy these files to `/etc/nginx/sites-available/` and create a symbolic link to `/etc/nginx/sites-enabled/`.

```bash
cp keycloak/auth.conf /etc/nginx/sites-available/auth.example.com
cp outline/outline.conf /etc/nginx/sites-available/outline.example.com
cp outline/minio.conf /etc/nginx/sites-available/minio.example.com
cp outline/minio-console.conf /etc/nginx/sites-available/minio-console.example.com
```

```bash
ln -s /etc/nginx/sites-available/auth.example.com /etc/nginx/sites-enabled/auth.example.com
ln -s /etc/nginx/sites-available/outline.example.com /etc/nginx/sites-enabled/outline.example.com
ln -s /etc/nginx/sites-available/minio.example.com /etc/nginx/sites-enabled/minio.example.com
ln -s /etc/nginx/sites-available/minio-console.example.com /etc/nginx/sites-enabled/minio-console.example.com
```

### Reload Nginx

Check the configuration files:
```bash
nginx -t
```

Reload Nginx:
```bash
sudo systemctl reload nginx
```
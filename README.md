# Dev Server

## Utils
- Traefik
- Pomerus
- OpenVSCode server

### Traefik
Traefik is used as an ingress for all our web service and will add SSL certificate with Let's Encrypt

### Pomerus
Pomerus is used as an internal proxy that add an authentification layer for accessing internal web service like OpenVSCode Server

### OpenVSCode Server
I use VSCode server for developping on the go on my iPad.

## Installation
### Traefik

#### Create local user
```
groupadd traefik
useradd \
    -g traefik \
    --no-user-group \
    --home-dir /opt/traefik \
    --no-create-home \
    --shell /usr/sbin/nologin \
    --system traefik
```

#### Install config
```
cp -r traefik/opt/etc /opt/traefik
chown -R root. /opt/

# Add local acme store for traefik
touch /opt/traefik/etc/acme.json
chown traefik. /opt/traefik/etc/acme.json
chmod 0600 /opt/traefik/etc/acme.json
```

#### Create systemd service
```
cp pomerium/etc/systemd/system/traefik.service /etc/systemd/system/
chown root. /etc/systemd/system/traefik.service

systemctl enable traefik.service
systemctl start traefik.service
```

### Pomerium
#### Install Pomerium binary
```
export POMERIUM_VERSION=v0.15.3
mkdir -p /opt/pomerium/bin
curl -L https://github.com/pomerium/pomerium/releases/download/${POMERIUM_VERSION}/pomerium-linux-amd64.tar.gz -o /opt/pomerium/bin/pomerium.tar.gz
tar xzf /tmp/pomerium.tar.gz -C /opt/pomerium/bin
```

#### Create local user
```
groupadd pomerium
useradd \
    -g pomerium \
    --no-user-group \
    --home-dir /opt/pomerium \
    --no-create-home \
    --shell /usr/sbin/nologin \
    --system pomerium
```

#### Prepare Pomerium config
For Pomerium, we need to configure the auth provider, in my case, I use Github.
If you only want to authorise a list of user email, we need to create an oauth application, but if we want to validate that a user is in a group of an organisation, we also need a Personal Access Token.

We also need to define a callback URL for the oauth validation in the `authenticate_service_url`.
[GitHub as an identity provider for Pomerium](# https://www.pomerium.com/docs/identity-providers/github.html)

And, we need to configure some routes and policy for them:
```
routes:
  - from: https://public_url
    to: http://localhost:3000
    allow_websockets: true
    policy:
      - allow:
          or:
            - email:
                is: my-github-email
```

#### Install Pomerium config
```
cp -r pomerium/opt/etc /opt/pomerium
chown -R root. /opt/pomerium
```

#### Create systemd service
```
cp pomerium/etc/systemd/system/pomerium.service /etc/systemd/system/
chown root. /etc/systemd/system/pomerium.service

systemctl enable pomerium.service
systemctl start pomerium.service
```

### OpenVSCode Server
#### Install systemd service
```
cp etc/systemd/system/vscode-server.service /etc/systemd/system/
systemctl enable vscode-server.service
systemctl start vscode-server.service
```
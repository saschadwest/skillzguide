# Final 

## Architecture

<p>
 I completed creating the architecture which encompassed NGINX, UNICORN, FLASK, VAULT inside Docker, and placing VAULT behind NGINX with (TLS). This was a incredibly deep learning curve because I have never done anything like this before and I wasn't even able reference a person to help because nobody knew how to do it. 
</p>

## New learnings
<p>
I was unable to unable to use VAULT effectively becuase the requirement was to use dockerize VAULT. Unlike all other projects, documentation that dockerized VAULT behind NGINX was always incomplete and caused a tremendous amount of heart break because my system kept crashing over and over. I constantly had to create a new image which taught me the vaule of versioning.
</p>

<p>
Basically I had to sit down and learn how to use Docker. I watched many hours of docker videos and completed various smaller projects before I had an understanding of the what/whys/hows of docker. Once I understood docker then it was time to add VAULT to docker. From there I had to learn the configuration of VAULT through Docker and also I had to develop a greater understaning of NGINX configuration so that I can connect Vault behind NGINX and still maintain TLS. 
</p>

## Configuring VAULT OPTIONS

### Docker

### install vault
 ```
 docker pull vault
 ```
 ### Create an Image for NGINX and keep it perpetually running
 
```
docker run -d -t --cap-add=IPC_LOCK -e 'VAULT_LOCAL_CONFIG={"storage": {"file": {"path": "/vault/file"}}, "listener": [{"tcp": { "address": "0.0.0.0:8200", "tls_disable": true}}], "default_lease_ttl": "168h", "max_lease_ttl": "720h", "ui": true}' -p 8200:8200 vault server
```

### Harden Vault with TLS

<p>
/etc/vault.hcl

We’re using the storage type file (previously termed back end), that ui has been set to true, without this we will have only the CLI and APIs

The Listener configuration defines our specific configurations for TCP settings, without the hard IP configuration we will listen on all IP addresses which is problematic for a multi IP system. By default we will also not enable TLS and will not look to use a certificate and private key.

The api_addr argument has been defined, this needs a full URL definition, including port, if this is not set, one is assumed from the IP set in the listener

The two lease settings define how long issued tokens from Vault to clients are valid for
</p>

```
storage "file" {
        path = "/opt/vault"
}

ui = true

listener "tcp" {
        address = "vault.skillzguide.com"
        tls_disable = 0
        tls_cert_file = "/etc/ssl/certs/vault.cer"
        tls_key_file = "/etc/ssl/private/vault.key"
}

max_lease_ttl = "10h"
default_lease_ttl = "10h"
api_addr = "https://164.92.92.186:8200"

```

<p>
set permissions on vault.hcl so service account can interact with it
</p>

```
# Set user and group ownership
sudo chown vault:vault /etc/vault.hcl

# Set file system permissions, mode 640
sudo chmod 640 /etc/vault.hcl

```

<p>
Demonize Vault
</p>

```
# Create a new unit file for the new vault.service Unit Service
sudo nano /etc/systemd/system/vault.service
```
<p> INSIDE FILE</p>

```
[Unit]
Description=Hashicorp Vault Service
Documentation=https://vaultproject.io/docs/
After=network.target
ConditionFileNotEmpty=/etc/vault.hcl

[Service]
User=vault
Group=vault
ExecStart=/usr/local/bin/vault server -config=/etc/vault.hcl
ExecReload=/usr/local/bin/kill --signal HUP $MAINPID
CapabilityBoundingSet=CAP_SYSLOG CAP_IPC_LOCK
AmbientCapabilities=CAP_IPC_LOCK
SecureBits=keep-caps
NoNewPrivileges=yes
KillSignal=SIGINT

[Install]
WantedBy=multi-user.target
```
<p> Verifying services run </p>

```
# Reload systemd daemon
sudo systemctl daemon-reload

# Start vault service
sudo systemctl start vault

# Verify status
sudo systemctl status vault

```

### Vault set up

<p> I created 5 key shares & 2 key thresholds. Initialized with inital root token </p>

### VAULT behind NGNIX
<p> /etc/nginx/default </p>

<p> 
This places vault behind nginx via 443. Vault has its own TLS certs
</p>

```
server {
        listen 443;
        server_name vault.skillzguide.com;

        ssl on;
        ssl_certificate /etc/letsencrypt/live/vault.skillzguide.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/vault.skillzguide.com/privkey.pem;

        ssl_prefer_server_ciphers on;
        ssl_session_timeout 1d;
        ssl_session_cache shared:SSL:50m;
        ssl_session_tickets off;

        location / {
                proxy_pass http://127.0.0.0:8200;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto https;
        }
}
```

### Clarification 

<p> Reverse proxy is in /etc/nginx/sites-available-project
</p>

```
server {
        listen 80;
        listen 443;
        server_name  164.92.92.186;
        return 301 https://www.skillzguide.com$request_uri;
}


server {
        listen 164.92.92.186;
        server_name skillzguide.com;
        return 301 https://www.skillzguide.com$request_uri;
}

```

## Inconclusion

<p> We have NGINX load balancer/reverse proxy as our ingress and outgress to FLASK via Gunicorn. Behind NGINX we also have VAULT which is out credential manager.
</p>

<p> I learned a ton about dev ops and networking. I relied completely on documentation and got through this by myself. It was daunting, anxiety inducing and challening but I overcame and developed confidence. I feel that I can get anything and that I am ready for new challenges in the real world. This opportunity is way harder than the capstone as I am greatful to have particpated </p>

<p>
This directly relates to security becuase its opens the door to alot of possibilities as networking and configurion is important to know. This project touched upon IAM, Networking, Sys admin, dev ops, security, documentation and a little bit of scripting.
</p>


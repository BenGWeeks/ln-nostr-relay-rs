# nostr-relay-ln

This is a [nostr](https://github.com/nostr-protocol/nostr) relay, written in Rust. It currently supports the entire relay protocol, and persists data with SQLite.

This is a fork of [https://github.com/scsibug/nostr-rs-relay](https://github.com/scsibug/nostr-rs-relay).

## Features

[NIPs](https://github.com/nostr-protocol/nips) with a relay-specific implementation are listed here.

- [x] NIP-01: [Basic protocol flow description](https://github.com/nostr-protocol/nips/blob/master/01.md)
  * Core event model
  * Hide old metadata events
  * Id/Author prefix search
- [x] NIP-02: [Contact List and Petnames](https://github.com/nostr-protocol/nips/blob/master/02.md)
- [ ] NIP-03: [OpenTimestamps Attestations for Events](https://github.com/nostr-protocol/nips/blob/master/03.md)
- [x] NIP-05: [Mapping Nostr keys to DNS-based internet identifiers](https://github.com/nostr-protocol/nips/blob/master/05.md)
- [x] NIP-09: [Event Deletion](https://github.com/nostr-protocol/nips/blob/master/09.md)
- [x] NIP-11: [Relay Information Document](https://github.com/nostr-protocol/nips/blob/master/11.md)
- [x] NIP-12: [Generic Tag Queries](https://github.com/nostr-protocol/nips/blob/master/12.md)
- [x] NIP-15: [End of Stored Events Notice](https://github.com/nostr-protocol/nips/blob/master/15.md)
- [x] NIP-16: [Event Treatment](https://github.com/nostr-protocol/nips/blob/master/16.md)
- [x] NIP-20: [Command Results](https://github.com/nostr-protocol/nips/blob/master/20.md)
- [x] NIP-22: [Event `created_at` limits](https://github.com/nostr-protocol/nips/blob/master/22.md) (_future-dated events only_)
- [x] NIP-26: [Event Delegation](https://github.com/nostr-protocol/nips/blob/master/26.md)

## Hardware / Virtual Machine

Using a virtual machine allows you to scale your relay as the amount of data and users increases with time. For a private-ish, friends/family relay, you can start with an x64 based architecture, 2 GB RAM, 30 GB SSD. Make sure you choose Ubuntu Server 20.04 or 22.04. Generate a SSH keypair file, make a note of the name, and save it locally. Ensure that the server is available via SSH (22), HTTP (80), and HTTPS (443).

## DNS

Login to your DNS provider and set an A record for `nostr.example.com` to point to the public IP address of the VM you just created. You can confirm the DNS propogation is successful by running:

```console
$ ping nostr.example.com
123.456.789.100
```

## Installation and configuration

### 1. Connect to the server via SSH

To connect the the machine, run the following SSH command (on Windows, by opening a command window as an Administrator, and changing directory to where you saved the SSH keypair file) and run:

```console
$ ssh -i <SSH-keypair-filename>.pem <SSH-keypair-name>@<IP-address>
```

### 2. Update system packages

To update your Ubuntu system packages, run the following:

```console
$ sudo apt update
$ sudo apt upgrade -y
```

### 3. Install required dependencies

To install the required dependencies, such as the `nginx` web server, `certbotto` to generate SSL certificates, `python3-certbot-nginx` to serve HTTPS content, `apt-transport-https` to xxxxxxxxx, `ca-certificates` to be a Certificate Authority on the Ubuntu server, `curl` to be able to transmit data using various protocols using the command line, and `software-properties-common` to yyyyyyyyyy.

```console
$ sudo apt install -y nginx certbot python3-certbot-nginx
$ sudo apt install apt-transport-https ca-certificates curl software-properties-common
```

To check the installation of some of the dependencies, run:

```
$ nginx -V
$ certbot --version
```

### 3. Setup Docker GPG key

```console
$ sudo mkdir -p /etc/apt/keyrings
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

### 4. Setup `apt` Docker repository

```
$ echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### 5. Install Docker

To setup Docker, run the following:

```console
$ sudo apt update
$ sudo apt-get install -y docker-ce
```

To check Docker is installed correctly run:

```console
$ docker --version
```

### 6. Make user to own directory

```console
$ sudo groupadd --gid 1000 appuser
$ sudo useradd --uid 1000 --gid appuser appuser
```

### 7. Make directory for relay database to live

```console
$ sudo mkdir /nostr-data
$ sudo mkdir /nostr-data/data
$ sudo chown -R 1000:1000 /nostr-data
```

### 8. Configure the relay

To make changes to the relay, we need to edit the [`config.toml`](config.toml) file. This file gives options include rate-limiting, event size limits, and network address settings. To make the changes required, run:

```console
$ sudo nano /nostr-data/config.toml
```

Within this file, you will want to make changes to the following lines (replacing with your relay details and your public key in HEX format):

```
name = "nostr-example"
description = "A newly created nostr-rs-relay from nostr.example.com."
pubkey = "971615b70ad9ec896f8d5ba0f2d01652f1dfe5f9ced81ac9469ca7facefad68b"
mode = "passive"
port = 7000
relay_url = "wss://nostr.example.com/"
```

### 9. Start the relay

To start the relay, run the following command:

```console
$ sudo docker run -it -d \
 -p 7000:7000 \
 --mount src=/nostr-data/config.toml,target=/usr/src/app/config.toml,type=bind \
 --mount src=/nostr-data/data,target=/usr/src/app/db,type=bind \
 --restart unless-stopped \
 scsibug/nostr-rs-relay:latest
```

To check for success, run the following command:

```console
curl localhost:7000
```

You should see `Please use a Nostr client to connect`.

### 10. Configure reverse proxy

To make changes to default Nginx configuration, run:

```console
$ sudo nano /etc/nginx/sites-available/default
```

And insert the following (replacing `nostr.example.com` with your domain):

```
server{
    server_name nostr.example.com;
    location / {
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        proxy_pass http://127.0.0.1:7000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

Now restart Nginx by running:

```console
$ sudo systemctl restart nginx
```

### 11. Setup SSL

Request an SSL certificate from letsencrypt/certbot by running the following (replace nostr.example.com with yours):

```console
$ sudo snap install --classic certbot
$ sudo certbot --nginx -d nostr.example.com
```

### 12. Test the WebSocket and HTTPS connections

To test the WebSocket connection, navigate to [https://websocketking.com](https://websocketking.com),and enter `wss://nostr.example.com` (replacing with your sub-domain details), and click `Connect`. You should see you are successfully connected.

To test the HTTPS connection, navigate to `https://nostr.example.com` (replacing with your sub-domain details). It should say `Please use a Nostr client to connect`.

### 13. Add your relay to your client

Finally, in your Nostr client, add your newly created relay `wss://nostr.example.com` (replacing with your sub-domain details). Submit some posts :-)

### 14. Query your relay

To query the relay, we need to install SQLite3, by running:

```console
$ sudo apt install sqlite3
```

You can then query your relay, by running commands such as:

```console
$ sqlite3 /nostr-data/data/nostr.db
$ select count(*) from event;
5
$ select count(distinct author) from event;
1
```

## Next Steps

You may want to consider hardening your VM against common attack types by performing actions such as:

- Hardning the SSH server
- Installing a firewall

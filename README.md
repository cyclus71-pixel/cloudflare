# Cloudflare Tunnel Setup on Linux

**This quick guide will help you set up a Cloudflare Tunnel on your Linux server !**

Cloudflare Tunnel provides you with a secure way to connect your resources to the web without a publicly routable IP address. With Tunnel, you do not send traffic to an external IP, instead, a lightweight daemon in your infrastructure (`cloudflared`) creates outbound-only connections to Cloudflare’s edge. Cloudflare Tunnel can connect HTTP web servers, SSH servers, remote desktops, and other protocols safely to Cloudflare. This way, your origins can serve traffic through Cloudflare without being vulnerable to attacks that bypass Cloudflare.
#1 Source
([Source](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/))
#2 My Source
([My Source](https://github.com/cyclus71-pixel/cloudflare.git))

## Getting Started

### Pre-requisites

* A Cloudflare account
* A domain managed by Cloudflare DNS
* A Linux server (in this example a Debian and Raspberry Pi 4)

### 1. Install `cloudflared`

Find the url of the `cloudflared` binary compatible with you architecture here :
https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/installation

> The `lscpu` command will give you the architecture of the system.

Once you have the binary downloaded, copy it to `/usr/local/bin/cloudflared` or add it to your `PATH`. Finally make it executable using `chmod`.

```sh
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-arm
sudo cp ./cloudflared-linux-arm /usr/local/bin/cloudflared
sudo chmod +x /usr/local/bin/cloudflared
cloudflared -v
```

### 2. Authentification

```sh
cloudflared tunnel login
```
This command will output an url to authenticate your Cloudflare account.

- Open the URL in a browser and login with your Cloudflare credentials
- This will create a certificate for the tunnel

### 3. Create tunnel

```sh
cloudflared tunnel create <tunelName>
```

### 4. Edit tunnel configuation

> You can find the tunnel uuid of the tunnel with the `cloudflared tunnel list` command.

Open the `~/.cloudflared/config.yml` file and add the following lines:

```yml
tunnel: <tunnel-uuid>
credentials-file: /home/pi/.cloudflared/<tunnel-uuid>.json 

ingress:
  - hostname: <your-domain>
    service: hello_world

  - service: http_status:404
```

Or Debbian
```yml
hostname: <host-local>
tunnel: <tunnel-uuid>
credentials-file: /home/pi/.cloudflared/<tunnel-uuid>.json 

```
Examples of Ingress settings are shown bellow. More information about the configuration settings can be found here : https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/configuration/configuration-file/

### 5. Create a new DNS record

When you add a hostname to the tunnel, you must create a DNS record (CNAME) in cloudflare to link this host to the tunnel. You can do this with the following command:
#On Raspberry Pi 4
```
cloudflared tunnel route dns <tunnel-uuid> <your-domain>
```

#On Debbian
```
cloudflared tunnel route dns <tunnelName> <your-domain>
```

> This step is required for each hostname you want to bind to the tunnel.

### 6. Start tunnel

```
cloudflared tunnel run <tunnel-name>
```

To run cloudflared automatically as a service, you can use the following commands :

```
cloudflared --config ~/.cloudflared/config.yml service install
sudo systemctl start cloudflared
sudo systemctl status cloudflared
```
This will create a new config file at `/etc/cloudflared/config.yml`. This is the file you must edit to change the tunnel configuration used by the service.

After editing the config file, don't forget to restart the service

```
sudo systemctl restart cloudflared
```

[Learn more about this here](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/run-tunnel/as-a-service/linux/).

### 7. Test tunnel

Navigate to the tunnel URL in your browser. You should see the Hello World page. You can now update the configuration file to bind hosts to different services.


## Ingress Settings

Their is no need to create a new tunnel for each host names you want to bind with the local server. Several ingress settings can be used to achieve this.

```yml
ingress:
  # Bind host to cloudflare test page
  - hostname: test.example.com
    service: hello_world

  # Bind host to a local http web server 
  - hostname: example.com
    service: https://localhost:8000

  # Rules can match the request's path to a regular expression:
  - hostname: static.example.com
    path: /*.(jpg|png|css|js)
    service: https://localhost:8001

  # Rules can match the request's hostname to a wildcard character:
  - hostname: '*.example.com'
    service: https://localhost:8002

  # Default catch-all rule (return a 404 if no host match the request):
  - service: http_status:404
```

:warning: The last rule you list in the config file must be a catch-all rule that matches all traffic.

Other type of port forwarding can be achieved, like TCP or SSH. More about ingress settings can be found here : https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/configuration/configuration-file/ingress/ 

---

## SSH Tunnel 

You can connect to machines over SSH through the tunnel using Cloudflare’s Zero Trust platform.

> An help page for SSH tunneling setup can be found here : https://blog.cloudflare.com/ssh-raspberry-pi-400-cloudflare-tunnel-auditable-terminal/

1. If not already done, you must create a **Cloudflare for Teams** organisation.

The first step is to visit https://dash.teams.cloudflare.com/ and following the setup guide.

2. In the Cloudflare for Teams dashboard [create a new "Self-hosted" Application](https://developers.cloudflare.com/cloudflare-one/tutorials/ssh/#create-a-zero-trust-policy) and follow the instructions to create a new Zero Trust policy.

4. If you want to enable SSH acces in a Browser-rendered terminal, you must enable it in the app's setings (in the **cloudeflared settings** section). Example [here](https://developers.cloudflare.com/cloudflare-one/applications/non-http/).


3. Create a new rule in the ingress section of the tunnel configuration (on your machine).

```yml
  - hostname: ssh.example.dev
    service: ssh://localhost:22
```
Don't forget to add a CNAME for your host (with `cloudflared tunnel route dns`), and to restart the tunnel / service.

5. Connect to the tunnel

If you enabled browser-rendered terminal, you can connect to the tunnel by navigating to the host url in your browser. You'll be prompted to enter your credentials.

To access the tunnel from a remote client without using the browser, you must use `cloudflared access` on the remote client. [Learn more here](https://developers.cloudflare.com/cloudflare-one/tutorials/ssh/#native-terminal).

---

## Access policy

You can create and manage access policies to your tunnels using Cloudflare’s Zero Trust platform. You must create new self hosted applications and policies in *Cloudflare for Teams* for each host you want to protect.

https://developers.cloudflare.com/cloudflare-one/applications/configure-apps/self-hosted-apps/

---

## Useful links

* [Tunnel Useful commands](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-useful-commands/)
* [Tunnel Guide](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-guide/)
* [Tunnel Configuration](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/configuration/configuration-file/)
* [Cloudflare’s Zero Trust platform Tutorials](https://developers.cloudflare.com/cloudflare-one/tutorials/)

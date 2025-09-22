Cloudflare Tunnel Setup on Linux
This quick guide will help you set up a Cloudflare Tunnel on your Linux server !

Cloudflare Tunnel provides you with a secure way to connect your resources to the web without a publicly routable IP address. With Tunnel, you do not send traffic to an external IP, instead, a lightweight daemon in your infrastructure (cloudflared) creates outbound-only connections to Cloudflare’s edge. Cloudflare Tunnel can connect HTTP web servers, SSH servers, remote desktops, and other protocols safely to Cloudflare. This way, your origins can serve traffic through Cloudflare without being vulnerable to attacks that bypass Cloudflare. (Source)

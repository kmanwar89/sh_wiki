---
Title: Intro to Reverse Proxies
---

### **Reverse proxies and YOU**

What can a reverse proxy do for *you*?

Many services that are self-hosted have a web UI. If you have many of these services running, a reverse proxy can be a central daemon that handles requests for all of these various backends.

Let's say that you have three different services running:

- Node.js forum NodeBB that runs on port 8008
- Navidrome music streaming server on port 3000 through a local machine using IP 192.168.1.125
- ownCloud web UI on port 8800 on a local machine with an IP of 192.168.1.123

You have a domain name that you want to use for these but don't want an ugly URL like http://mydomain.com:8008. To complicate things further, these services are all on different hosts.

A reverse proxy can be configured to accept requests for this domain and redirect them to a different host or port.

To make your URLs pretty, the reverse proxy can be configured to redirect requests on your domain based on a folder name to a different service on your local network.

- http://mydomain.com/forum/ --> http://localhost:8008/
- http://mydomain.com/music/ --> http://192.168.1.125:3000/
- http://mydomain.com/cloud/ --> http://192.168.1.123:8800/

As you can see, it is much nicer to reach these services through a single domain and folder than to use their port and host individually.

You can even couple this with a self-hosted VPN so that these requests can be proxied to different services on different networks in different locations. All you need to do is to make sure the proxy and the services are on the same VPN and to use the VPN IP addresses.

You don't have to use folders either. You can use subdomains as well such as music.mydomain.com, cloud.mydomain.com, and forum.mydomain.com respectively. It's all up to you and how you want to structure your services.


### **Popular Reverse Proxies**

This wiki already contains steps on how to set up *nginx*, which is one of the more popular reverse proxies and may be encountered in enterprise environments, however it is not the only option. Some other options include:

- Nginx Proxy Manager (GUI front-end to NGINX)
- Caddy (can be used as both an HTTP server or as a reverse proxy, so it is included here as well)
- Traefik

The general concept of these is the same as mentioned above - they allow you to host multiple services from behind a single IP address, they allow for flexibility and control and, most importantly, they offer *security* by handling SSL for you. A brief list of features to help you decide:

|            | Graphical  interface | File-based  configuration | Handle SSL  automatically | Route traffic  based on labels | Used in Enterprise Deployments | Allow configuration of subdomains and  subdirectories | Easy to read configuration | Integrate directly into docker compose files |
|:----------:|:--------------------:|:-------------------------:|:-------------------------:|:------------------------------:|:------------------------------:|:-----------------------------------------------------:|:--------------------------:|:--------------------------------------------:|
|     NPM    |           X          |                           |             X             |                                |                                |                           X                           |                            |                                              |
|   Traefik  |                      |             X             |             X             |                X               |                                |                           X                           |              X             |                       X                      |
|    Caddy   |                      |             X             |             X             |                                |                                |                           X                           |              X             |                                              |
| Pure NGINX |                      |             X             |             X             |                                |                X               |                           X                           |                            |                                              |
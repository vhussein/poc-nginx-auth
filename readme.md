# Purpose

To evaluate the use of basic authentication in NGINX to serve different upstream & downstream. The high level requirement is to have NGINX as the reverse proxy to serve to different downstream (old & new) as well as 2 different upstreams (BAU & new) with basic authentication.

> https://docs.nginx.com/nginx/admin-guide/security-controls/configuring-http-basic-authentication/'

# End-to-End Nginx Docker Setup with Basic Authentication

## Directory Structure
```plaintext
poc-nginx-auth/
│
├── docker-compose.yml
├── central-node/
│   ├── nginx.conf
│   ├── htpasswd_api1
│   └── htpasswd_api2
├── upstream-node1/
│   ├── nginx.conf
│   └── index.html
├── upstream-node2/
    ├── nginx.conf
    └── index.html
```

## Overview
This setup consists of three Nginx nodes configured using Docker Compose:
1. **Central Node**: Acts as the entry point for incoming requests. It supports:
   - Two virtual hosts (`api1.localhost` (existing) and `api2.localhost` (new)).
   - Proxies requests to two upstream nodes.
   - Implements basic authentication for each virtual host.

2. **Upstream Nodes**:
   - **Node 1**: Serves requests for `api1.localhost` with its own content.
   - **Node 2**: Serves requests for `api2.localhost` with different content.

---

## Central Node
The central node:
- Listens on port **8080**.
- Routes requests based on the `Host` header:
  - Requests to `api1.localhost` are proxied to Upstream Node 1.
  - Requests to `api2.localhost` are proxied to Upstream Node 2.
- Implements **basic authentication**:
  - Requires credentials from the `htpasswd` files for each virtual host:
    - `htpasswd_api1` for `api1.localhost`.
    - `htpasswd_api2` for `api2.localhost`.

---

## Upstream Nodes
Each upstream node:
- Listens on its own local Nginx instance. In our setup, will follow BAU & new.
- Serves static content (`index.html`) specific to its endpoint:
  - **Upstream Node 1**: Returns a "Hello from Upstream Node 1" message.
  - **Upstream Node 2**: Returns a "Hello from Upstream Node 2" message.

---

## Basic Authentication
- **`htpasswd_api1`**: Contains credentials for `api1.localhost`.
- **`htpasswd_api2`**: Contains credentials for `api2.localhost`.

### Steps to Generate `.htpasswd` Files
1. Install the `htpasswd` tool on the server. For simplicity of this POC, I've created using an online tool (https://www.web2generators.com/apache-tools/htpasswd-generator)
2. Generate credentials:
   - For API 1:
     ```bash
     htpasswd -c central-node/htpasswd_api1 user1
     ```
   - For API 2:
     ```bash
     htpasswd -c central-node/htpasswd_api2 user2
     ```
3. These files are mounted into the central node container.

---

## Routing Behavior
1. **Request Flow**:
   - Incoming requests arrive at the **Central Node** on port `8080`.
   - Based on the `Host` header, the central node proxies the request to:
     - **Upstream Node 1** for `api1.localhost`.
     - **Upstream Node 2** for `api2.localhost`.

2. **Authentication**:
   - Each virtual host requires credentials to proceed.

---

## Verification
### Running the docker compose
In the terminal, run the following command from  root to start the docker nodes:
```bash
docker-compose up -d
```

### Accessing the APIs
1. Add the following entries to your `/etc/hosts` file (optional):
- `127.0.0.1 api1.localhost` 
- `127.0.0.1 api2.localhost`

2. Test with a browser or `curl`:
- API 1:
  ```bash
  curl -u user1:<password> http://api1.localhost:8080
  
  OR
  
  curl --location 'http://api1.localhost:8080/' --header 'Authorization: Basic dXNlcjE6ODROZ05nMmtJYUVQY29OV0c2V0M='
  ```
- API 2:
  ```bash
  curl -u user2:<password> http://api2.localhost:8080

  OR

  curl --location 'http://api2.localhost:8080' --header 'Authorization: Basic dXNlcjI6VTl6anFHMmNDTmtKQzFtRVZReFU='
  ```

---

## Summary
This setup achieves:
- A central entry point for routing requests to two different upstream nodes.
- Basic authentication for secure access to each endpoint.
- Independent upstream nodes, each serving unique content.

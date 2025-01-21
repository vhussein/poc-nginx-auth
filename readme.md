# Purpose

To evaluate the use of basic authentication in NGINX to serve different upstream & downstream.

> https://docs.nginx.com/nginx/admin-guide/security-controls/configuring-http-basic-authentication/'

# Implementation

poc-nginx-auth/
│
├── docker-compose.yml
├── site1/
│   ├── default.conf
│   ├── htpasswd
│   └── index.html
├── site2/
│   ├── default.conf
│   ├── htpasswd
│   └── index.html

# Directory Structure Explanation

## Root Directory (`poc-nginx-auth/`)
This is the main directory for the project, containing the necessary files and subdirectories to set up and run the application.

### `docker-compose.yml`
- Defines the Docker services for the project.
- Configures two Nginx containers (`site1` and `site2`) with:
  - Mounted configuration files (`default.conf` for each site).
  - Separate content files (`index.html`).
  - Authentication files (`htpasswd`).
- Exposes unique ports (`8081` for `site1` and `8082` for `site2`) to access the services.

---

## Subdirectory: `site1/`
This directory contains files for the first Nginx virtual host.

- **`default.conf`**
  - The Nginx configuration file for `site1`.
  - Configures the web server to:
    - Serve content from `/usr/share/nginx/html` (where `index.html` is mounted).
    - Require basic authentication using the `htpasswd` file.
  
- **`htpasswd`**
  - Contains hashed credentials for basic authentication for `site1`.
  - Created using the `htpasswd` command.
  - Specifies usernames and passwords required to access the `site1` endpoint.

- **`index.html`**
  - A simple HTML file that serves as the "Hello World" page for `site1`.
  - Displays a greeting message when accessed via the web browser.

---

## Subdirectory: `site2/`
This directory contains files for the second Nginx virtual host. The structure is identical to `site1/` but serves as an independent site.

- **`default.conf`**
  - The Nginx configuration file for `site2`.
  - Similar to `site1/default.conf` but operates for `site2`.

- **`htpasswd`**
  - Contains hashed credentials for basic authentication for `site2`.
  - Separate from `site1/htpasswd`, allowing unique usernames and passwords.

- **`index.html`**
  - A simple HTML file that serves as the "Hello World" page for `site2`.
  - Displays a greeting message specific to `site2`.

---

## Summary
This setup achieves:
1. **Two distinct Nginx virtual hosts** (`site1` and `site2`), each serving unique content.
2. **Basic authentication** for each site, ensuring restricted access with separate credentials.
3. Port forwarding to enable accessing both sites on different ports locally (`8081` for `site1` and `8082` for `site2`).


## Testing

Using curl to test with the basic authentication in the header. You can decode the value using online base64decoder tool.

`curl --location 'http://localhost:8081/' --header 'Authorization: Basic dXNlcjE6ODROZ05nMmtJYUVQY29OV0c2V0M='`

`curl --location 'http://localhost:8082/' --header 'Authorization: Basic dXNlcjI6VTl6anFHMmNDTmtKQzFtRVZReFU='`
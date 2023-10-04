# Nginx Dynamic Reverse Proxy Configuration

## Introduction

This repository contains Nginx configuration file (`nginx.conf`) that is designed to serve as a dynamic reverse proxy. This configuration is specifically tailored for secure Kubernetes environments where nodes do not have direct internet access. The proxy allows pods within the Kubernetes cluster to access external APIs through this intermediary Nginx server.

## Usage

### Deployment

1. Apply the Nginx configuration to your Kubernetes environment.
2. Ensure that the necessary certificates are provided (replace `/opt/acme/certs/example.com/fullchain.cer` and `/opt/acme/certs/example.com/private.key` with your actual certificate paths).

### Access

To access external APIs from within your Kubernetes pods, you can make HTTP requests to the Nginx reverse proxy. For example, to call the Deezer API endpoint:

```bash
curl https://otpxy.example.com/deezerdevs-deezer.p.rapidapi.com/infos
```

instead of

```bash
curl https://deezerdevs-deezer.p.rapidapi.com/infos
```

This will route your request through the Nginx server, allowing pods to access external resources securely. The proxy will forward the request to the original API endpoint.

Please replace `https://otpxy.example.com/deezerdevs-deezer.p.rapidapi.com/infos` with the desired API endpoint.

### Configuration Details

#### resolver

The `resolver` directive specifies the DNS server to use for resolving domain names. In this example, it is set to Google's DNS server (`8.8.8.8`).

#### server

- `listen 443 ssl`: Specifies that Nginx should listen for incoming connections on port 443 using SSL.
- `server_name otpxy.example.com`: Defines the server name.
- `proxy_read_timeout 6000;`: Sets the maximum time to read the response from the proxied server.
- `proxy_connect_timeout 6000;`: Sets the maximum time to establish a connection with the proxied server.
- `proxy_send_timeout 6000;`: Sets the maximum time to send data to the proxied server.

#### location

- `if ($request_uri ~ ^\/(.*)$) { ... }`: Extracts the URI path and sets it as the upstream URL.
- `if ($upstream_url ~ ^https:\/\/([^\/]+)) { ... }`: Extracts the host name from the upstream URL.

#### proxy_pass

- `proxy_pass $upstream_url;`: Passes the request to the upstream server.

#### Headers

- `add_header X-Request-URI $request_uri;`: Adds the request URI as a header.
- `add_header X-Upstream-URL $upstream_url;`: Adds the upstream URL as a header.

#### SSL Configuration

- `ssl_certificate /opt/acme/certs/example.com/fullchain.cer;`: Specifies the SSL certificate file path.
- `ssl_certificate_key /opt/acme/certs/example.com/private.key;`: Specifies the SSL certificate key file path.

## Important Notes

- Ensure that you replace the certificate paths with your actual certificate paths.
- Customize the `server_name` directive to match your domain.
- Make sure to deploy this configuration in a secure Kubernetes environment where direct internet access from nodes is restricted.

## License

This project is licensed under the [MIT License](LICENSE).

# TLS Termination Proxy

[![Docker Pulls][pulls-badge]][docker-hub]

A docker image that provides a simple TLS/SSL termination proxy to be
used in front of another running docker container with a web server.

It is using [Pound][pound] for TLS/SSL termination. Pound is a small
reverse proxy, load balancer and HTTPS front-end for Web servers. Of
course there various other good pieces of software out there, that can
do the same job just as well, for instance [HAProxy][haproxy] and
[nginx][nginx].

## Usage

### Run the container

When started, this container will listen on port 443 for incoming
HTTPS requests. Requests will then be forwarded via HTTP to another
container, the actual web server. Address/hostname and port of the
upstream server have to configured using environment variables.

The easiest way is to simply link the two containers together and
configure the name of the web server container as upstream server
address. A file containing the SSL certificate(s) and the private key
must be mounted as a volume. It is expected to be found at the
location /cert.pem by the container.

```
docker run \
    -e HTTPS_UPSTREAM_SERVER_ADDRESS=othercontainer \
    -e HTTPS_UPSTREAM_SERVER_PORT=80 \
    -e TIMEOUT=15 \
    -v /path/to/cert.pem:/cert.pem:ro \
    --link othercontainer:othercontainer \
    mnuessler/tls-termination-proxy
```

#### Kubernetes support

When running this image on Kubernetes, the volume hosting the certificate has to be
a directory (cannot be the file itself).
The following command shows the alternative settings to support kubernetes volumes:

```
docker run \
    -e HTTPS_UPSTREAM_SERVER_ADDRESS=othercontainer \
    -e HTTPS_UPSTREAM_SERVER_PORT=80 \
    -e CERT_PATH=/certs/cert.pem \
    -e TIMEOUT=15 \
    -v /path/to/certs/:/certs/:ro \
    --link othercontainer:othercontainer \
    mnuessler/tls-termination-proxy
```

Notice that `CERT_PATH` has to be configured and the volume has to point to a directory, not
the certificate file itself.
If `CERT_PATH` is not configured, it will default to `/cert.pem`.

### Build the image

```
docker build -t mnuessler/tls-termination-proxy .
```

Or simply pull it from [Docker Hub][docker-hub]:

```
docker pull mnuessler/tls-termination-proxy
```

### Troubleshooting

* *Container fails to start with error "Port is supported only for
  INET/INET6 back-ends"*: When HTTPS_UPSTREAM_SERVER_ADDRESS is set to
  a hostname (i.e. container name) and that hostname cannot be
  resolved to an IP address on startup, then Pound will assume that it
  represents the path for a Unix-domain socket. In that case, the
  configuration option for port becomes invalid. Solution: Make sure
  that the value configured for server address is a valid hostname. If
  you are using a container name, make sure the container is linked.

[pulls-badge]: https://img.shields.io/docker/pulls/mnuessler/tls-termination-proxy.svg?maxAge=3600
[docker-hub]: https://registry.hub.docker.com/u/mnuessler/tls-termination-proxy/
[pound]: http://www.apsis.ch/pound
[haproxy]: http://www.haproxy.org/
[nginx]: http://nginx.org/


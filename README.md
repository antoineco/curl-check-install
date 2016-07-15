# cURL, check, install

I found myself repeating the same pattern in my Dockerfiles over and over again:

1. Install cURL from available package repositories
2. Use cURL to download an executable or archive from Github
3. Check that the downloaded file matches the expected checksum
4. Move the file to PATH
5. Purge cURL

So I made a dumb script to automate that, which I push to my usual base image.

It currently works on Debian and its forks only, feel free to extend it for other distributions.

## Usage

```
curl-check-install -u url [-n filename] [-d dest] [-x] [-c checksum] [-p program]

Options:
  -u	URL of the file
  -n	Name of the file once installed (default: same as source)
  -d	Directory to move the file to (default: '/usr/local/bin')
  -x	Make the file executable
  -c	File checksum
  -p	Program used for checking file checksum (default: 'sha1sum')
  -h	Display help
```

## Examples

Install the [tini](https://github.com/krallin/tini/) container init using a Dockerfile:

````dockerfile
FROM debian

ENV TINI_VERSION 0.9.0
ENV TINI_SHA1 fa23d1e20732501c3bb8eeeca423c89ac80ed452

RUN curl-check-install \
      -u https://github.com/krallin/tini/releases/download/v${TINI_VERSION}/tini-static \
      -n tini \
      -c $TINI_SHA1 \
      -x
```

Install the [docker](https://github.com/docker/docker/) binary using a Dockerfile:

````dockerfile
FROM debian

ENV DOCKER_VERSION 1.11.2
ENV DOCKER_SHA256 8c2e0c35e3cda11706f54b2d46c2521a6e9026a7b13c7d4b8ae1f3a706fc55e1

RUN curl-check-install \
      -u https://get.docker.com/builds/Linux/x86_64/docker-${DOCKER_VERSION}.tgz \
      -n docker.tgz \
      -d /tmp \
      -c $DOCKER_SHA256 \
      -p sha256sum \
    && tar -xzvf /tmp/docker.tgz -C /usr/local/bin/ --strip 1 \
    && rm /tmp/docker.tgz
```

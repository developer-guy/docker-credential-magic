# docker-credential-magic

[![GitHub Actions status](https://github.com/docker-credential-magic/docker-credential-magic/workflows/build/badge.svg)](https://github.com/docker-credential-magic/docker-credential-magic/actions?query=workflow%3Abuild+)

![docker-credential-magic](./docker-credential-magic.png)

- [Overview](#overview)
- [Installation](#installation)
- [Usage](#usage)
  - [`docker-credential-magic`](#docker-credential-magic)
  - [`docker-credential-magician`](#docker-credential-magician)

## Overview

This repo contains the source for two separate tools:

- `docker-credential-magic` - credential helper which proxies auth to other helpers based on domain name
- `docker-credential-magician` - tool to augment images with various credential helpers (including `magic`)

The following third-party Docker credential helpers are currently supported:

- [`acr-env`](https://github.com/chrismellard/docker-credential-acr-env) - for Azure Container Registry (ACR)
- [`ecr-login`](https://github.com/awslabs/amazon-ecr-credential-helper) - for Amazon Elastic Container Registry (ECR)
- [`gcr`](https://github.com/GoogleCloudPlatform/docker-credential-gcr) - for Google Container Registry (GCR),
  Google Artifact Registry (GAR)

## Installation

Download [latest release](https://github.com/docker-credential-magic/docker-credential-magic/releases/latest) tarball
for your system and install both tools manually:

```
cat docker-credential-magic*.tar.gz | tar x -C /usr/local/bin 'docker-credential-magic*'
```

## Usage

### `docker-credential-magic`

The following example shows how `docker-credential-magic` can be used to
proxy auth to `docker-credential-gcr`, based on the detection of a `*.gcr.io` domain:

*Note: Example requires [`docker-credential-gcr`](https://github.com/GoogleCloudPlatform/docker-credential-gcr)
to be pre-installed*

```
$ export GOOGLE_APPLICATION_CREDENTIALS="${PWD}/service-account-key.json"
```

```
$ echo "us.gcr.io" | docker-credential-magic get
{"ServerURL":"us.gcr.io","Username":"_dcgcr_token","Secret":"*****"}
```

### `docker-credential-magician`

The following example shows how `docker-credential-magician` can be used to
(1) augment the [`cosign`](https://github.com/sigstore/cosign) image with
various credential helpers, (2) set the default credential store to `magic`,
and (3) push the new image to a registry running at `localhost:5000`:

```
$ docker-credential-magician mutate \
    gcr.io/projectsigstore/cosign/ci/cosign:v0.5.0 \
    -t localhost:5000/cosign:v0.5.0-magic
2021/07/29 17:06:59 Pulling gcr.io/projectsigstore/cosign/ci/cosign:v0.5.0 ...
2021/07/29 17:07:01 Adding /opt/magic/etc/aws.yml ...
2021/07/29 17:07:01 Adding /opt/magic/etc/azure.yml ...
2021/07/29 17:07:01 Adding /opt/magic/etc/gcp.yml ...
2021/07/29 17:07:01 Adding /opt/magic/bin/docker-credential-ecr-login ...
2021/07/29 17:07:01 Adding /opt/magic/bin/docker-credential-acr-env ...
2021/07/29 17:07:01 Adding /opt/magic/bin/docker-credential-gcr ...
2021/07/29 17:07:01 Adding /opt/magic/bin/docker-credential-magic ...
2021/07/29 17:07:01 Adding /opt/magic/config.json ...
2021/07/29 17:07:02 Prepending PATH with /opt/magic/bin ...
2021/07/29 17:07:02 Setting DOCKER_CONFIG to /opt/magic ...
2021/07/29 17:07:02 Setting DOCKER_CREDENTIAL_MAGIC_CONFIG to /opt/magic ...
2021/07/29 17:07:02 Pushing image to localhost:5000/cosign:v0.5.0-magic ...
```

```
$ docker run --rm --entrypoint sh \
    localhost:5000/cosign:v0.5.0-magic \
    -c 'ls -lah /opt/magic/etc && \
        ls -lah /opt/magic/bin &&
        env | grep DOCKER_ &&
        cat $DOCKER_CONFIG/config.json'
total 20K
drwxr-xr-x    2 root     root        4.0K Jul 29 21:00 .
drwxr-xr-x    4 root     root        4.0K Jul 29 21:00 ..
-r-xr-xr-x    1 root     root          45 Jan  1  1970 aws.yml
-r-xr-xr-x    1 root     root          40 Jan  1  1970 azure.yml
-r-xr-xr-x    1 root     root          44 Jan  1  1970 gcp.yml
total 25M
drwxr-xr-x    2 root     root        4.0K Jul 29 21:00 .
drwxr-xr-x    4 root     root        4.0K Jul 29 21:00 ..
-r-xr-xr-x    1 root     root        8.7M Jan  1  1970 docker-credential-acr-env
-r-xr-xr-x    1 root     root        7.8M Jan  1  1970 docker-credential-ecr-login
-r-xr-xr-x    1 root     root        5.6M Jan  1  1970 docker-credential-gcr
-r-xr-xr-x    1 root     root        3.0M Jan  1  1970 docker-credential-magic
DOCKER_CREDENTIAL_MAGIC_CONFIG=/opt/magic
DOCKER_CONFIG=/opt/magic
{"credsStore":"magic"}
```

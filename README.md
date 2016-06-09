# Nomad Docker Wrapper

## Description

Small wrapper to enable running arbitrary docker run commands in [nomad](https://www.nomadproject.io/).
This is useful when you require features that are not supported by the nomad docker task driver.

## Usage

```sh
sudo NOMAD_DOCKER_CONTAINER_NAME=http ./nomad-docker-wrapper \
-p 8000:8000 python:alpine python -m http.server&
```

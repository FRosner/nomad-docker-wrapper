# Nomad Docker Wrapper

## Description

Small wrapper to enable running arbitrary docker run commands in [nomad](https://www.nomadproject.io/).
This is useful when you require features that are not supported by the nomad docker task driver.

## Usage

```sh
sudo NOMAD_DOCKER_CONTAINER_NAME=python ./nomad-docker-wrapper python:alpine python -m http.server
```

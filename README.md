# Nomad Docker Wrapper

[![Build Status](https://travis-ci.org/FRosner/nomad-docker-wrapper.svg?branch=master)](https://travis-ci.org/FRosner/nomad-docker-wrapper)

## Description

Small wrapper to enable running arbitrary docker run commands in [nomad](https://www.nomadproject.io/).
This is useful when you require features that are not supported by the nomad docker task driver.

## Usage

You can use the wrapper script as a job artifact to run your docker container with the specified arguments. Using a named container will ensure that it gets removed properly the next time you start it again. You have to provide `NOMAD_DOCKER_CONTAINER_NAME` as an environment variable.

If you want to use a custom docker registry, please provide the `NOMAD_DOCKER_REGISTRY_URL` alongside with the credentials. For details, see the configuration section.

If you are using the "latest" tag of an image and want to pull it before running, you can specify the image to pull by setting `NOMAD_DOCKER_PULL_COMMAND`.

Below are examples that start a simple python web server inside a container and expose the respective port. While it is technically possible to use the wrapper from the shell, it is not very useful. The intended use case is within nomad.

### Shell

```sh
NOMAD_DOCKER_CONTAINER_NAME=http-server \
bash <(curl -s https://raw.githubusercontent.com/FRosner/nomad-docker-wrapper/master/nomad-docker-wrapper) \
-p 8000:8000 python:alpine python -m http.server
```

### Nomad

```hcl
job "http-job" {
  datacenters = ["dc1"]
  group "http-group" {
    task "http-task" {
      driver = "raw_exec"

      artifact {
        source = "https://raw.githubusercontent.com/FRosner/nomad-docker-wrapper/master/nomad-docker-wrapper"
      }

      env {
        NOMAD_DOCKER_CONTAINER_NAME = "${NOMAD_TASK_NAME}"
      }

      config {
        command = "nomad-docker-wrapper"
        args = ["-p", "8000:8000", "python:alpine", "python", "-m", "http.server"]
      }

      resources {
        cpu = 500
        memory = 128
      }
    }
  }
}
```

## Configuration

You can use environment variables to configure the wrapper. Below is a list of supported variables. The ones marked with * are mandatory.

| Variable | Description |
| -------- | ----------- |
| `NOMAD_DOCKER_CONTAINER_NAME`* | Container name to be used on the docker host. |
| `NOMAD_DOCKER_REGISTRY_URL` | URL of your private docker registry. If this is set, the wrapper will attempt to login. |
| `NOMAD_DOCKER_REGISTRY_USER` | User to use for logging into your private docker registry. |
| `NOMAD_DOCKER_REGISTRY_PASSWORD` | Password to use for logging into your private docker registry. |
| `NOMAD_DOCKER_REGISTRY_EMAIL` | Email address to use for logging into your private docker registry. |
| `NOMAD_DOCKER_PULL_COMMAND` | Arguments to pass to `docker pull` which will get execute before `docker run`. |

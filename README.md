# Nomad Docker Wrapper

[![Build Status](https://travis-ci.org/FRosner/nomad-docker-wrapper.svg?branch=master)](https://travis-ci.org/FRosner/nomad-docker-wrapper)

## Description

Small wrapper to enable running arbitrary docker run commands in [nomad](https://www.nomadproject.io/).
This is useful when you require features that are not supported by the nomad docker task driver.

## Usage

You can use the wrapper script as a job artifact to run your docker container with the specified arguments. Using a named container will ensure that it gets removed properly the next time you start it again. You have to provide `NOMAD_DOCKER_CONTAINER_NAME` as an environment variable.

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

# Nomad Docker Wrapper

[![Build Status](https://travis-ci.org/FRosner/nomad-docker-wrapper.svg?branch=master)](https://travis-ci.org/FRosner/nomad-docker-wrapper)

## Description

Small wrapper to enable running arbitrary docker run commands in [nomad](https://www.nomadproject.io/).
This is useful when you require features that are not supported by the nomad docker task driver.

## Usage

You can use the wrapper script as an artifact to start run your docker container with the specified arguments. Below is an example of a nomad job that runs a simple python web server inside a container.

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

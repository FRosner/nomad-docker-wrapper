# DEPRECATION NOTE
This tool is no longer required as Nomad now supports all major features within the native Docker driver.

# Nomad Docker Wrapper

[![Build Status](https://travis-ci.org/FRosner/nomad-docker-wrapper.svg?branch=master)](https://travis-ci.org/FRosner/nomad-docker-wrapper)

## Description

Small wrapper to enable running arbitrary docker run commands in [Nomad](https://www.nomadproject.io/).
This is useful when you require features that are not supported by the Nomad docker task driver.

## Usage

You can use the wrapper script as a job artifact to run your docker container with the specified arguments.

### Docker Container Name

Using a named container will ensure that it gets removed properly the next time you start it again. You have to provide `NOMAD_DOCKER_CONTAINER_NAME` as an environment variable.

### Docker Pull Before Run

If you are using the "latest" tag of an image and want to pull it before running, you can specify the image to pull by setting `NOMAD_DOCKER_PULL_COMMAND`.

### Environment Variable Replacement

If you want to use environment variables inside your docker command that are passed by Nomad (e.g. dynamic ports), you can include them as a String and rely on the wrapper to replace them. To accomplish this, just set `NOMAD_DOCKER_ENVSUBST` to any value.

*Please make sure that `envsubst` is installed on your system when using this option. It is part of the `gettext` package: `apt-get install gettext`.*

### Private Docker Registry

If you want to use a custom docker registry, please provide the `NOMAD_DOCKER_REGISTRY_URL` alongside with the credentials. For details, see the configuration section.

*CAUTION: The credentials will be passed as environment variables to the Nomad job. Everyone who can query for these environment variables through the Nomad API is able to see your registry credentials. Better practice is to make sure all Nomad nodes are logged into the docker registry already.*

### Artifact Download

If you want to download an artifact before launching your job, you can configure this using the `NOMAD_DOCKER_ARTIFACT_*` environment variables.
While the functionality is somewhat overlapping with the [artifact stanza](https://www.nomadproject.io/docs/job-specification/artifact.html) from Nomad, it was introduced due to the lack of the possibility to
provide basic authentication credentials.

Nomad Docker Wrapper will download the specified resource to the specified location before executing the job.
If you plan to use it inside the job, you should mount the target folder.
If the file is a tar, tar gzip or tar bzip2 compressed file, it will also be uncompressed for you.

## Examples

Below are examples that start a simple python web server inside a container and expose the respective port. While it is technically possible to use the wrapper from the shell, it is not very useful. The intended use case is within Nomad.

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
        NOMAD_DOCKER_ENVSUBST = 1
      }

      config {
        command = "nomad-docker-wrapper"
        args = ["-p", "$NOMAD_PORT_http:8000", "python:alpine", "python", "-m", "http.server"]
      }

      resources {
        cpu = 500
        memory = 128
        network {
          mbits = 10
          port "http" {}
        }
      }
    }
  }
}
```

## Configuration

You can use environment variables to configure the wrapper. Below is a list of supported variables. The ones marked with * are mandatory.

### General Options

| Variable | Description |
| -------- | ----------- |
| `NOMAD_DOCKER_CONTAINER_NAME`* | Container name to be used on the docker host. |
| `NOMAD_DOCKER_ENVSUBST` | Set this to apply `envsubst` to your docker command before running it. |

### Docker Registry Interaction

| Variable | Description |
| -------- | ----------- |
| `NOMAD_DOCKER_PULL_COMMAND` | Arguments to pass to `docker pull` which will get execute before `docker run`. |
| `NOMAD_DOCKER_REGISTRY_URL` | URL of your private docker registry. If this is set, the wrapper will attempt to login. |
| `NOMAD_DOCKER_REGISTRY_USER` | User to use for logging into your private docker registry. |
| `NOMAD_DOCKER_REGISTRY_PASSWORD` | Password to use for logging into your private docker registry. |
| `NOMAD_DOCKER_REGISTRY_EMAIL` | Email address to use for logging into your private docker registry. |

### Artifact Download

| Variable | Description |
| -------- | ----------- |
| `NOMAD_DOCKER_ARTIFACT_SOURCE` | URL to wget for downloading an artifact before executing the job. |
| `NOMAD_DOCKER_ARTIFACT_TARGET` | Folder to download the artifact to. |
| `NOMAD_DOCKER_ARTIFACT_USER` | (Optional) user name for HTTP basic authentication. |
| `NOMAD_DOCKER_ARTIFACT_PASSWORD` | (Optional) password for HTTP basic authentication. |


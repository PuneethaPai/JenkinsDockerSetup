# Jenkins Docker Setup

This repo contains required docker configurations to help quick and reproducible setup of Jenkins.

Jenkins installation [page](https://www.jenkins.io/doc/book/installing/docker/) provides us with more detailed explaination of the same and this repo just convert the same set of instructions into easier portable solution using `docker-compose`.

# Usage:

```bash
$ docker-compose up --build
Creating JenkinsDocker ... done
Creating JenkinsServer ... done
```

## Details:

The [`docker-compose.yaml`](./docker-compose.yaml) file defines following components:

- jenkins network: Defines underlying bridge network.
- jenkins server: This is jenkins server container definition.
- jenkins docker: A `docker:dind` container to enable running docker inside docker.

### Jenkins Network:

```yaml
networks:
  jenkins:
    driver: bridge # Defines bridge network to be used by services defined later.
```

### Jenkins Docker:

```yaml
services:
  jenkins_docker:
    image: docker:dind
    networks:
      jenkins:
        aliases:
          - docker # Defines to use jenkins network defined above also under the alias name `docker`.
    container_name: JenkinsDocker
    privileged: true
    environment:
      - DOCKER_TLS_CERTDIR=/certs
    ports:
      - "2376:2376" # Exposes docker serveer port 2376 to be used by jenkins server container at "tcp://docker:2376".
    volumes:
      - ./jenkins-docker-certs:/certs/client # Docker client certs.
      - ./jenkins-data:/var/jenkins_home # Preserves Jenkins data like job definitions, credentials, build logs, etc.
      - ./extras:/extras # Any extra data or files you want to cache between server restart can be saved here `/extras/`.
```

### Jenkins Server:

```yaml
services:
  jenkins_server:
    build:
      context: # Build container from the custom Dockerfile defined in the repo.
    networks:
      - jenkins # Use jenkins network defined earlier
    container_name: JenkinsServer
    restart: always
    environment: # Define docker env variable to connect to docker-engine defined in JenkinsDocker container.
      - DOCKER_HOST=tcp://docker:2376
      - DOCKER_CERT_PATH=/certs/client
      - DOCKER_TLS_VERIFY=1
    ports:
      - "8080:8080" # For UI
      - "50000:50000" # For API
    volumes:
      - ./jenkins-data:/var/jenkins_home:rw # Docker client certs.
      - ./jenkins-docker-certs:/certs/client:ro # Preserves Jenkins data like job definitions, credentials, build logs, etc.
      - ./extras:/extras:rw # Any extra data or files you want to cache between server restart can be saved here `/extras/`.
```

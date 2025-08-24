# Docker

## Install docker

- [Install Docker Engine on Debian](https://docs.docker.com/engine/install/debian/)

## Docker Usage

### Run shell inside container


In case the container is already running:
```
docker exec -it <mycontainer> sh
```

In case we want to run a shell in a new container:
```
docker run -ti <image> /bin/bash
```

- [How do I get into a Docker container's shell?](https://stackoverflow.com/a/30173220)

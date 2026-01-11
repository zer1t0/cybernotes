# Docker / Podman

Docker is well known containers tool.

Podman is container management tool that shares the same API as Docker, so you
can run any docker command by replacing `docker` by `podman`, but also allows
users to create containers without requiring root permissions.

## Install podman

```
sudo apt install podman
```

## Usage

### Remove all containers

```
docker rm $(docker ps -a -q)
```

- [Stop and remove all docker containers](https://stackoverflow.com/a/51316176)


### Run shell inside container

In case the container is already running:
```
podman exec -it <mycontainer> sh
```

In case we want to run a shell in a new container:
```
podman run -ti <image> /bin/bash
```

- [How do I get into a Docker container's shell?](https://stackoverflow.com/a/30173220)


### Stop all containers

```
docker stop $(docker ps -a -q)
```

- [Stop and remove all docker containers](https://stackoverflow.com/a/51316176)


## Where are the volumes stored

Volumes are folders under `/var/lib/docker/volumes/`
```
sudo ls -lah /var/lib/docker/volumes/
```

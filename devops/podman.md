# Podman

Podman is container management tool that shares the same API as Docker, so you
can run any docker command by replacing `docker` by `podman`, but also allows
users to create containers without requiring root permissions.

## Install podman

```
sudo apt install podman
```

## Podman Usage

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

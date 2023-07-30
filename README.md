# herz-podman
A repository where I store my podman stuff.

In case you are not familiar with [Podman](https://podman.io/), it's a drop-in replacement for Docker that can be used serviceless and rootless and can make use of Kubernetes Pods. You can literally copy Docker commands and replace `docker` with `podman`, and generally it should work, aside from a few Podman particularities.

There's also [podman-compose](https://github.com/containers/podman-compose) which is a drop-in replacement for docker-compose.

Podman can also generate Kubernetes Pod files with `podman generate kube` for easy deployment with `podman play kube`, and systemd service files that can easily be used to have your service run on boot with user permissions.

## How to manage these containers

### podman-compose

Alter the values in env, then run:

```bash
podman-compose --project-name wikijs --in-pod 1 --env-file env up --detach
```

### podman-play-kube

Alter the values in wikijs-configmap.yaml, then run:

```bash
podman play kube wikijs-kube.yaml --configmap wikijs-configmap.yaml
```

You may also specify the host port to be used with play:

```bash
podman play kube wikijs-kube.yaml --configmap wikijs-configmap.yaml --publish 3500:3000
```

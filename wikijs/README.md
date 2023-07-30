## WikiJS Deployment

[WikiJS](https://js.wiki/) is a selfhosted Markdown wiki written in Vue and Node.

### podman-compose

Install `aardvark-dns`. Uncomment the database options, alter the values in env, then run:

```bash
podman-compose --project-name wikijs --in-pod 1 --env-file env up --detach
```

Set `external: true` in compose.yaml if you want to create the desired volume
instead of letting podman-compose do it. In this case, the service name
(e.g. dbdata) is only used to be referenced inside the compose.yaml, while
`name:` value will be used for the pre-existing volume that podman will search for.

### podman-play-kube

Uncomment the database and configmap options you want to use, then run:

```bash
podman play kube wikijs-kube.yaml --configmap wikijs-configmap.yaml
```

You may also specify the host port to be used for a specific container port
(port 3000 from WikiJS is exposed as 3500 on the host):

```bash
podman play kube wikijs-kube.yaml --configmap wikijs-configmap.yaml --publish 3500:3000
```

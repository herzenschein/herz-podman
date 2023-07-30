## Owncast Deployment

[Owncast](https://owncast.online/) is a selfhosted, single-user alternative
to streaming services like Twitch, written in Go and React.

After deployment, use localhost:8080/admin to access the administrative interface,
Ctrl+Shift+R to trigger basic authentication, then the login should be
`admin` + `abc123`. Change the password afterwards.

### podman-compose

Alter the values in env, then run:

```bash
podman-compose --project-name owncast --in-pod 1 --env-file env up --detach
```

Set `external: true` in compose.yaml if you want to create the desired volume
instead of letting podman-compose do it. In this case, the service name
(e.g. data) is only used to be referenced inside the compose.yaml, while the
`name:` value will be used for the pre-existing volume that podman will search for.

### podman-play-kube

Run:

```bash
podman play kube owncast-kube.yaml
```

You may also specify the host port to be used for a specific container port
(for example, port 8080 from WikiJS exposed as 80 on the host and port 1935
exposed as port 2000 on the host):

```bash
podman play kube owncast-kube.yaml --publish 80:8080 --publish 2000:1935
```

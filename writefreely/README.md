## WriteFreely Deployment (WIP)

[WriteFreely](https://writefreely.org/) is a selfhosted, Fediverse-enabled
blog platform for writers, written in Go.

This is a WIP. With these steps, the current error that I get is:

```
ERROR: 2023/07/30 22:19:13 database.go:808: Failed selecting from collections: Error 1146: Table 'writefreelydb.collections' doesn't exist
```

Furthermore, WriteFreely only pings once to see if the database is available.
This does not match well with the async behavior of docker-compose and
podman-compose. Ideally upstream should add a few pings like what WikiJS does
(1 ping every 3 seconds for a max of 10 pings).

### podman-compose

First, create a dummy config file and give it 777 permissions for now so that
the container is able to access it:

```bash
touch writefreely-config.ini
chmod 777 writefreely-config.ini
```

TODO: improve permission handling

Generate a config file in the current directory using a temporary WriteFreely
container:

```bash
podman run --name writefreely --rm --interactive --tty --mount type=bind,source=./writefreely-config.ini,destination=/go/config.ini,rw docker.io/writeas/writefreely:latest config generate
```

Finally, run:

```bash
podman-compose --project-name writefreely --in-pod 1 up --detach
```

The writefreely-web container will fail because it pinged once. Just start it again:

```bash
podman start writefreely-web
```

Set `external: true` in compose.yaml if you want to create the desired volume
instead of letting podman-compose do it. In this case, the service name
(e.g. data) is only used to be referenced inside the compose.yaml, while the
`name:` value will be used for the pre-existing volume that podman will search for.

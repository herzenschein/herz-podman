## WriteFreely Deployment (WIP)

[WriteFreely](https://writefreely.org/) is a selfhosted, Fediverse-enabled
blog platform for writers, written in Go.

This is a WIP. With these steps, the current error that I get is:

```
LC_ALL=C curl http://localhost:8080
curl: (56) Recv failure: Connection reset by peer
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
podman run --name writefreely --rm --interactive --tty --mount type=bind,source=./writefreely-config.ini,destination=/go/config.ini,rw,relabel=private docker.io/writeas/writefreely:latest config generate
```

Change the ini file to set your website. Make sure that the user, password and
database are correct and that the host matches the database service name,
as in the example file provided here.

Finally, run:

```bash
podman-compose --project-name writefreely --in-pod 1 up --detach
```

You will need to initialize the database using a disposable writefreely container
connecting to the database. Specifying the network is needed here:

```bash
podman run --pod pod_writefreely --network writefreely_writefreely --name writefreely --rm --interactive --tty --mount type=bind,source=./writefreely-config.ini,destination=/go/config.ini,rw docker.io/writeas/writefreely:latest db init
```

If the database is not initialized, you get
`Error 1146: Table 'writefreelydb.collections' doesn't exist`.

The writefreely-web container will have failed because it pinged once before
the database was fully initialized. Just start it again:

```bash
podman start writefreely-web
```

Set `external: true` in compose.yaml if you want to create the desired volume
instead of letting podman-compose do it. In this case, the service name
(e.g. data) is only used to be referenced inside the compose.yaml, while the
`name:` value will be used for the pre-existing volume that podman will search for.

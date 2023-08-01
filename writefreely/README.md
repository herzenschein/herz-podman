## WriteFreely Deployment (WIP)

[WriteFreely](https://writefreely.org/) is a selfhosted, Fediverse-enabled
blog platform for writers, written in Go.

### podman-compose

First, create a dummy config file and give it 777 permissions for now so that
the container is able to access it:

```bash
touch writefreely-config.ini
chmod 606 writefreely-config.ini
```

Generate a config file in the current directory using a temporary WriteFreely
container:

```bash
podman run --name writefreely --rm --interactive --tty --mount type=bind,source=./writefreely-config.ini,destination=/go/config.ini,rw,relabel=private docker.io/writeas/writefreely:latest config generate
```

Change the ini file to set your website. Make sure that the user, password and
database follow the compose file and that the host matches the database
service name, as in the example file provided here.

Finally, run:

```bash
podman-compose --project-name writefreely --in-pod 1 up --detach
```

You will need to initialize the database using a disposable writefreely container
connecting to the database. Specifying the network is needed here:

```bash
podman run --pod pod_writefreely --network writefreely_writefreely --name writefreely --rm --interactive --tty --mount type=bind,source=./writefreely-config.ini,destination=/go/config.ini,rw docker.io/writeas/writefreely:latest db init
```

WriteFreely only pings once to see if the database is available.
This does not match well with the async behavior of docker-compose and
podman-compose. Ideally upstream should add a few pings like what WikiJS does
(1 ping every 3 seconds for a max of 10 pings).

Because of that, the writefreely-web container will have failed because it
pinged once before the database was fully initialized. Just start it again:

```bash
podman start writefreely-web
```

Set `external: true` in compose.yaml if you want to create the desired volume
instead of letting podman-compose do it. In this case, the service name
(e.g. data) is only used to be referenced inside the compose.yaml, while the
`name:` value will be used for the pre-existing volume that podman will search for.

---

## Errors you might encounter

WriteFreely will attempt to generate keys if you do not provide your own with
`writefreely generate keys`. If you fail to mount the volume correctly, you will see:

```
ERROR: 2023/07/31 02:52:02 main.go:120: init keys: open keys/email.aes256: no such file or directory
```

If you fail to initialize the database, you will see:

```
Error 1146: Table 'writefreelydb.collections' doesn't exist.
```

You should use 0.0.0.0 as your bind in the configuration file. Otherwise,
you will see:

```
curl http://localhost:8080
curl: (56) Recv failure: Connection reset by peer
```

The first registered user will be admin. If your config file contains
`open_registration=false` and `single_user=true`, you will see "Page not found".

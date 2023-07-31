## WriteFreely Deployment (WIP)

[WriteFreely](https://writefreely.org/) is a selfhosted, Fediverse-enabled
blog platform for writers, written in Go.

This is a WIP.

```
ERROR: 2023/07/31 04:24:04 database.go:808: Failed selecting from collections: Error 1054: Unknown column 'post_signature' in 'field list'
```

This seems to be an upstream database error.

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

The `schema.sql` file in this folder is from
`wget https://raw.githubusercontent.com/writefreely/writefreely/develop/schema.sql`.
It is mounted inside `/tmp/schema.sql` in the database container.

Finally, run:

```bash
podman-compose --project-name writefreely --in-pod 1 up --detach
```

To write the schema.sql to the database, run:

```bash
podman-compose exec writefreely-db sh -c 'exec mariadb --user=writefreely --password=writefreelypass writefreelydb < /tmp/schema.sql'
```

Without it, you get `Error 1146: Table 'writefreelydb.collections' doesn't exist`.

The writefreely-web container will fail because it pinged once before
the database finished initializing. Just start it again:

```bash
podman start writefreely-web
```

Set `external: true` in compose.yaml if you want to create the desired volume
instead of letting podman-compose do it. In this case, the service name
(e.g. data) is only used to be referenced inside the compose.yaml, while the
`name:` value will be used for the pre-existing volume that podman will search for.

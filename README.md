# dokku omnisci [![Build Status](https://img.shields.io/github/actions/workflow/status/dokku/dokku-omnisci/ci.yml?branch=master&style=flat-square "Build Status")](https://github.com/dokku/dokku-omnisci/actions/workflows/ci.yml?query=branch%3Amaster) [![IRC Network](https://img.shields.io/badge/irc-libera-blue.svg?style=flat-square "IRC Libera")](https://webchat.libera.chat/?channels=dokku)

Official omnisci plugin for dokku. Currently defaults to installing [omnisci/core-os-cpu v5.10.2](https://hub.docker.com/r/omnisci/core-os-cpu/).

## Requirements

- dokku 0.19.x+
- docker 1.8.x

## Installation

```shell
# on 0.19.x+
sudo dokku plugin:install https://github.com/dokku/dokku-omnisci.git --name omnisci
```

## Commands

```
omnisci:app-links <app>                            # list all omnisci service links for a given app
omnisci:backup-set-public-key-encryption <service> <public-key-id> # set GPG Public Key encryption for all future backups of omnisci service
omnisci:backup-unset-public-key-encryption <service> # unset GPG Public Key encryption for future backups of the omnisci service
omnisci:connect <service>                          # connect to the service via the omnisci connection tool
omnisci:create <service> [--create-flags...]       # create a omnisci service
omnisci:destroy <service> [-f|--force]             # delete the omnisci service/data/container if there are no links left
omnisci:enter <service>                            # enter or run a command in a running omnisci service container
omnisci:exists <service>                           # check if the omnisci service exists
omnisci:expose <service> <ports...>                # expose a omnisci service on custom host:port if provided (random port on the 0.0.0.0 interface if otherwise unspecified)
omnisci:info <service> [--single-info-flag]        # print the service information
omnisci:link <service> <app> [--link-flags...]     # link the omnisci service to the app
omnisci:linked <service> <app>                     # check if the omnisci service is linked to an app
omnisci:links <service>                            # list all apps linked to the omnisci service
omnisci:list                                       # list all omnisci services
omnisci:logs <service> [-t|--tail] <tail-num-optional> # print the most recent log(s) for this service
omnisci:pause <service>                            # pause a running omnisci service
omnisci:promote <service> <app>                    # promote service <service> as OMNISCI_URL in <app>
omnisci:restart <service>                          # graceful shutdown and restart of the omnisci service container
omnisci:set <service> <key> <value>                # set or clear a property for a service
omnisci:start <service>                            # start a previously stopped omnisci service
omnisci:stop <service>                             # stop a running omnisci service
omnisci:unexpose <service>                         # unexpose a previously exposed omnisci service
omnisci:unlink <service> <app>                     # unlink the omnisci service from the app
omnisci:upgrade <service> [--upgrade-flags...]     # upgrade service <service> to the specified versions
```

## Usage

Help for any commands can be displayed by specifying the command as an argument to omnisci:help. Plugin help output in conjunction with any files in the `docs/` folder is used to generate the plugin documentation. Please consult the `omnisci:help` command for any undocumented commands.

### Basic Usage

### create a omnisci service

```shell
# usage
dokku omnisci:create <service> [--create-flags...]
```

flags:

- `-c|--config-options "--args --go=here"`: extra arguments to pass to the container create command (default: `None`)
- `-C|--custom-env "USER=alpha;HOST=beta"`: semi-colon delimited environment variables to start the service with
- `-i|--image IMAGE`: the image name to start the service with
- `-I|--image-version IMAGE_VERSION`: the image version to start the service with
- `-m|--memory MEMORY`: container memory limit in megabytes (default: unlimited)
- `-N|--initial-network INITIAL_NETWORK`: the initial network to attach the service to
- `-p|--password PASSWORD`: override the user-level service password
- `-P|--post-create-network NETWORKS`: a comma-separated list of networks to attach the service container to after service creation
- `-r|--root-password PASSWORD`: override the root-level service password
- `-S|--post-start-network NETWORKS`: a comma-separated list of networks to attach the service container to after service start
- `-s|--shm-size SHM_SIZE`: override shared memory size for omnisci docker container

Create a omnisci service named lollipop:

```shell
dokku omnisci:create lollipop
```

You can also specify the image and image version to use for the service. It *must* be compatible with the omnisci/core-os-cpu image.

```shell
export OMNISCI_IMAGE="omnisci/core-os-cpu"
export OMNISCI_IMAGE_VERSION="${PLUGIN_IMAGE_VERSION}"
dokku omnisci:create lollipop
```

You can also specify custom environment variables to start the omnisci service in semicolon-separated form.

```shell
export OMNISCI_CUSTOM_ENV="USER=alpha;HOST=beta"
dokku omnisci:create lollipop
```

### print the service information

```shell
# usage
dokku omnisci:info <service> [--single-info-flag]
```

flags:

- `--config-dir`: show the service configuration directory
- `--data-dir`: show the service data directory
- `--dsn`: show the service DSN
- `--exposed-ports`: show service exposed ports
- `--id`: show the service container id
- `--internal-ip`: show the service internal ip
- `--initial-network`: show the initial network being connected to
- `--links`: show the service app links
- `--post-create-network`: show the networks to attach to after service container creation
- `--post-start-network`: show the networks to attach to after service container start
- `--service-root`: show the service root directory
- `--status`: show the service running status
- `--version`: show the service image version

Get connection information as follows:

```shell
dokku omnisci:info lollipop
```

You can also retrieve a specific piece of service info via flags:

```shell
dokku omnisci:info lollipop --config-dir
dokku omnisci:info lollipop --data-dir
dokku omnisci:info lollipop --dsn
dokku omnisci:info lollipop --exposed-ports
dokku omnisci:info lollipop --id
dokku omnisci:info lollipop --internal-ip
dokku omnisci:info lollipop --initial-network
dokku omnisci:info lollipop --links
dokku omnisci:info lollipop --post-create-network
dokku omnisci:info lollipop --post-start-network
dokku omnisci:info lollipop --service-root
dokku omnisci:info lollipop --status
dokku omnisci:info lollipop --version
```

### list all omnisci services

```shell
# usage
dokku omnisci:list
```

List all services:

```shell
dokku omnisci:list
```

### print the most recent log(s) for this service

```shell
# usage
dokku omnisci:logs <service> [-t|--tail] <tail-num-optional>
```

flags:

- `-t|--tail [<tail-num>]`: do not stop when end of the logs are reached and wait for additional output

You can tail logs for a particular service:

```shell
dokku omnisci:logs lollipop
```

By default, logs will not be tailed, but you can do this with the --tail flag:

```shell
dokku omnisci:logs lollipop --tail
```

The default tail setting is to show all logs, but an initial count can also be specified:

```shell
dokku omnisci:logs lollipop --tail 5
```

### link the omnisci service to the app

```shell
# usage
dokku omnisci:link <service> <app> [--link-flags...]
```

flags:

- `-a|--alias "BLUE_DATABASE"`: an alternative alias to use for linking to an app via environment variable
- `-q|--querystring "pool=5"`: ampersand delimited querystring arguments to append to the service link
- `-n|--no-restart "false"`: whether or not to restart the app on link (default: true)

A omnisci service can be linked to a container. This will use native docker links via the docker-options plugin. Here we link it to our `playground` app.

> NOTE: this will restart your app

```shell
dokku omnisci:link lollipop playground
```

The following environment variables will be set automatically by docker (not on the app itself, so they won’t be listed when calling dokku config):

```
DOKKU_OMNISCI_LOLLIPOP_NAME=/lollipop/DATABASE
DOKKU_OMNISCI_LOLLIPOP_PORT=tcp://172.17.0.1:6274
DOKKU_OMNISCI_LOLLIPOP_PORT_6274_TCP=tcp://172.17.0.1:6274
DOKKU_OMNISCI_LOLLIPOP_PORT_6274_TCP_PROTO=tcp
DOKKU_OMNISCI_LOLLIPOP_PORT_6274_TCP_PORT=6274
DOKKU_OMNISCI_LOLLIPOP_PORT_6274_TCP_ADDR=172.17.0.1
```

The following will be set on the linked application by default:

```
OMNISCI_URL=omnisci://lollipop:SOME_PASSWORD@dokku-omnisci-lollipop:6274/lollipop
```

The host exposed here only works internally in docker containers. If you want your container to be reachable from outside, you should use the `expose` subcommand. Another service can be linked to your app:

```shell
dokku omnisci:link other_service playground
```

It is possible to change the protocol for `OMNISCI_URL` by setting the environment variable `OMNISCI_DATABASE_SCHEME` on the app. Doing so will after linking will cause the plugin to think the service is not linked, and we advise you to unlink before proceeding.

```shell
dokku config:set playground OMNISCI_DATABASE_SCHEME=omnisci2
dokku omnisci:link lollipop playground
```

This will cause `OMNISCI_URL` to be set as:

```
omnisci2://lollipop:SOME_PASSWORD@dokku-omnisci-lollipop:6274/lollipop
```

### unlink the omnisci service from the app

```shell
# usage
dokku omnisci:unlink <service> <app>
```

flags:

- `-n|--no-restart "false"`: whether or not to restart the app on unlink (default: true)

You can unlink a omnisci service:

> NOTE: this will restart your app and unset related environment variables

```shell
dokku omnisci:unlink lollipop playground
```

### set or clear a property for a service

```shell
# usage
dokku omnisci:set <service> <key> <value>
```

Set the network to attach after the service container is started:

```shell
dokku omnisci:set lollipop post-create-network custom-network
```

Set multiple networks:

```shell
dokku omnisci:set lollipop post-create-network custom-network,other-network
```

Unset the post-create-network value:

```shell
dokku omnisci:set lollipop post-create-network
```

### Service Lifecycle

The lifecycle of each service can be managed through the following commands:

### connect to the service via the omnisci connection tool

```shell
# usage
dokku omnisci:connect <service>
```

Connect to the service via the omnisci connection tool:

> NOTE: disconnecting from ssh while running this command may leave zombie processes due to moby/moby#9098

```shell
dokku omnisci:connect lollipop
```

### enter or run a command in a running omnisci service container

```shell
# usage
dokku omnisci:enter <service>
```

A bash prompt can be opened against a running service. Filesystem changes will not be saved to disk.

> NOTE: disconnecting from ssh while running this command may leave zombie processes due to moby/moby#9098

```shell
dokku omnisci:enter lollipop
```

You may also run a command directly against the service. Filesystem changes will not be saved to disk.

```shell
dokku omnisci:enter lollipop touch /tmp/test
```

### expose a omnisci service on custom host:port if provided (random port on the 0.0.0.0 interface if otherwise unspecified)

```shell
# usage
dokku omnisci:expose <service> <ports...>
```

Expose the service on the service's normal ports, allowing access to it from the public interface (`0.0.0.0`):

```shell
dokku omnisci:expose lollipop 6274 6278
```

Expose the service on the service's normal ports, with the first on a specified ip address (127.0.0.1):

```shell
dokku omnisci:expose lollipop 127.0.0.1:6274 6278
```

### unexpose a previously exposed omnisci service

```shell
# usage
dokku omnisci:unexpose <service>
```

Unexpose the service, removing access to it from the public interface (`0.0.0.0`):

```shell
dokku omnisci:unexpose lollipop
```

### promote service <service> as OMNISCI_URL in <app>

```shell
# usage
dokku omnisci:promote <service> <app>
```

If you have a omnisci service linked to an app and try to link another omnisci service another link environment variable will be generated automatically:

```
DOKKU_OMNISCI_BLUE_URL=omnisci://other_service:ANOTHER_PASSWORD@dokku-omnisci-other-service:6274/other_service
```

You can promote the new service to be the primary one:

> NOTE: this will restart your app

```shell
dokku omnisci:promote other_service playground
```

This will replace `OMNISCI_URL` with the url from other_service and generate another environment variable to hold the previous value if necessary. You could end up with the following for example:

```
OMNISCI_URL=omnisci://other_service:ANOTHER_PASSWORD@dokku-omnisci-other-service:6274/other_service
DOKKU_OMNISCI_BLUE_URL=omnisci://other_service:ANOTHER_PASSWORD@dokku-omnisci-other-service:6274/other_service
DOKKU_OMNISCI_SILVER_URL=omnisci://lollipop:SOME_PASSWORD@dokku-omnisci-lollipop:6274/lollipop
```

### start a previously stopped omnisci service

```shell
# usage
dokku omnisci:start <service>
```

Start the service:

```shell
dokku omnisci:start lollipop
```

### stop a running omnisci service

```shell
# usage
dokku omnisci:stop <service>
```

Stop the service and removes the running container:

```shell
dokku omnisci:stop lollipop
```

### pause a running omnisci service

```shell
# usage
dokku omnisci:pause <service>
```

Pause the running container for the service:

```shell
dokku omnisci:pause lollipop
```

### graceful shutdown and restart of the omnisci service container

```shell
# usage
dokku omnisci:restart <service>
```

Restart the service:

```shell
dokku omnisci:restart lollipop
```

### upgrade service <service> to the specified versions

```shell
# usage
dokku omnisci:upgrade <service> [--upgrade-flags...]
```

flags:

- `-c|--config-options "--args --go=here"`: extra arguments to pass to the container create command (default: `None`)
- `-C|--custom-env "USER=alpha;HOST=beta"`: semi-colon delimited environment variables to start the service with
- `-i|--image IMAGE`: the image name to start the service with
- `-I|--image-version IMAGE_VERSION`: the image version to start the service with
- `-N|--initial-network INITIAL_NETWORK`: the initial network to attach the service to
- `-P|--post-create-network NETWORKS`: a comma-separated list of networks to attach the service container to after service creation
- `-R|--restart-apps "true"`: whether or not to force an app restart (default: false)
- `-S|--post-start-network NETWORKS`: a comma-separated list of networks to attach the service container to after service start
- `-s|--shm-size SHM_SIZE`: override shared memory size for omnisci docker container

You can upgrade an existing service to a new image or image-version:

```shell
dokku omnisci:upgrade lollipop
```

### Service Automation

Service scripting can be executed using the following commands:

### list all omnisci service links for a given app

```shell
# usage
dokku omnisci:app-links <app>
```

List all omnisci services that are linked to the `playground` app.

```shell
dokku omnisci:app-links playground
```

### check if the omnisci service exists

```shell
# usage
dokku omnisci:exists <service>
```

Here we check if the lollipop omnisci service exists.

```shell
dokku omnisci:exists lollipop
```

### check if the omnisci service is linked to an app

```shell
# usage
dokku omnisci:linked <service> <app>
```

Here we check if the lollipop omnisci service is linked to the `playground` app.

```shell
dokku omnisci:linked lollipop playground
```

### list all apps linked to the omnisci service

```shell
# usage
dokku omnisci:links <service>
```

List all apps linked to the `lollipop` omnisci service.

```shell
dokku omnisci:links lollipop
```
### Backups

Datastore backups are supported via AWS S3 and S3 compatible services like [minio](https://github.com/minio/minio).

You may skip the `backup-auth` step if your dokku install is running within EC2 and has access to the bucket via an IAM profile. In that case, use the `--use-iam` option with the `backup` command.

If both passphrase and public key forms of encryption are set, the public key encryption will take precedence.

The underlying core backup script is present [here](https://github.com/dokku/docker-s3backup/blob/main/backup.sh).

Backups can be performed using the backup commands:

### set GPG Public Key encryption for all future backups of omnisci service

```shell
# usage
dokku omnisci:backup-set-public-key-encryption <service> <public-key-id>
```

Set the `GPG` Public Key for encrypting backups:

```shell
dokku omnisci:backup-set-public-key-encryption lollipop
```

This method currently requires the <public-key-id> to be present on the keyserver `keyserver.ubuntu.com`:

### unset GPG Public Key encryption for future backups of the omnisci service

```shell
# usage
dokku omnisci:backup-unset-public-key-encryption <service>
```

Unset the `GPG` Public Key encryption for backups:

```shell
dokku omnisci:backup-unset-public-key-encryption lollipop
```

### Disabling `docker image pull` calls

If you wish to disable the `docker image pull` calls that the plugin triggers, you may set the `OMNISCI_DISABLE_PULL` environment variable to `true`. Once disabled, you will need to pull the service image you wish to deploy as shown in the `stderr` output.

Please ensure the proper images are in place when `docker image pull` is disabled.

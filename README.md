# dokku omnisci [![Build Status](https://img.shields.io/circleci/project/github/dokku/dokku-omnisci.svg?branch=master&style=flat-square "Build Status")](https://circleci.com/gh/dokku/dokku-omnisci/tree/master) [![IRC Network](https://img.shields.io/badge/irc-freenode-blue.svg?style=flat-square "IRC Freenode")](https://webchat.freenode.net/?channels=dokku)

Official omnisci plugin for dokku. Currently defaults to installing [omnisci/core-os-cpu v5.1.2](https://hub.docker.com/r/omnisci/core-os-cpu/).

## Requirements

- dokku 0.19.x+
- docker 1.8.x

## Installation

```shell
# on 0.19.x+
sudo dokku plugin:install https://github.com/dokku/dokku-omnisci.git omnisci
```

## Commands

```
omnisci:app-links <app>                        # list all omnisci service links for a given app
omnisci:connect <service>                      # connect to the service via the omnisci connection tool
omnisci:create <service> [--create-flags...]   # create a omnisci service
omnisci:destroy <service> [-f|--force]         # delete the omnisci service/data/container if there are no links left
omnisci:enter <service>                        # enter or run a command in a running omnisci service container
omnisci:exists <service>                       # check if the omnisci service exists
omnisci:expose <service> <ports...>            # expose a omnisci service on custom port if provided (random port otherwise)
omnisci:info <service> [--single-info-flag]    # print the service information
omnisci:link <service> <app> [--link-flags...] # link the omnisci service to the app
omnisci:linked <service> <app>                 # check if the omnisci service is linked to an app
omnisci:links <service>                        # list all apps linked to the omnisci service
omnisci:list                                   # list all omnisci services
omnisci:logs <service> [-t|--tail]             # print the most recent log(s) for this service
omnisci:promote <service> <app>                # promote service <service> as OMNISCI_URL in <app>
omnisci:restart <service>                      # graceful shutdown and restart of the omnisci service container
omnisci:start <service>                        # start a previously stopped omnisci service
omnisci:stop <service>                         # stop a running omnisci service
omnisci:unexpose <service>                     # unexpose a previously exposed omnisci service
omnisci:unlink <service> <app>                 # unlink the omnisci service from the app
omnisci:upgrade <service> [--upgrade-flags...] # upgrade service <service> to the specified versions
```

## Usage

Help for any commands can be displayed by specifying the command as an argument to omnisci:help. Please consult the `omnisci:help` command for any undocumented commands.

### Basic Usage

### create a omnisci service

```shell
# usage
dokku omnisci:create <service> [--create-flags...]
```

flags:

- `-C|--custom-env "USER=alpha;HOST=beta"`: semi-colon delimited environment variables to start the service with
- `-i|--image IMAGE`: the image name to start the service with
- `-I|--image-version IMAGE_VERSION`: the image version to start the service with
- `-p|--password PASSWORD`: override the user-level service password
- `-r|--root-password PASSWORD`: override the root-level service password

Create a omnisci service named lolipop:

```shell
dokku omnisci:create lolipop
```

You can also specify the image and image version to use for the service. It *must* be compatible with the omnisci/core-os-cpu image. 

```shell
export OMNISCI_IMAGE="omnisci/core-os-cpu"
export OMNISCI_IMAGE_VERSION="${PLUGIN_IMAGE_VERSION}"
dokku omnisci:create lolipop
```

You can also specify custom environment variables to start the omnisci service in semi-colon separated form. 

```shell
export OMNISCI_CUSTOM_ENV="USER=alpha;HOST=beta"
dokku omnisci:create lolipop
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
- `--links`: show the service app links
- `--service-root`: show the service root directory
- `--status`: show the service running status
- `--version`: show the service image version

Get connection information as follows:

```shell
dokku omnisci:info lolipop
```

You can also retrieve a specific piece of service info via flags:

```shell
dokku omnisci:info lolipop --config-dir
dokku omnisci:info lolipop --data-dir
dokku omnisci:info lolipop --dsn
dokku omnisci:info lolipop --exposed-ports
dokku omnisci:info lolipop --id
dokku omnisci:info lolipop --internal-ip
dokku omnisci:info lolipop --links
dokku omnisci:info lolipop --service-root
dokku omnisci:info lolipop --status
dokku omnisci:info lolipop --version
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
dokku omnisci:logs <service> [-t|--tail]
```

flags:

- `-t|--tail`: do not stop when end of the logs are reached and wait for additional output

You can tail logs for a particular service:

```shell
dokku omnisci:logs lolipop
```

By default, logs will not be tailed, but you can do this with the --tail flag:

```shell
dokku omnisci:logs lolipop --tail
```

### link the omnisci service to the app

```shell
# usage
dokku omnisci:link <service> <app> [--link-flags...]
```

flags:

- `-a|--alias "BLUE_DATABASE"`: an alternative alias to use for linking to an app via environment variable
- `-q|--querystring "pool=5"`: ampersand delimited querystring arguments to append to the service link

A omnisci service can be linked to a container. This will use native docker links via the docker-options plugin. Here we link it to our 'playground' app. 

> NOTE: this will restart your app

```shell
dokku omnisci:link lolipop playground
```

The following environment variables will be set automatically by docker (not on the app itself, so they won’t be listed when calling dokku config):

```
DOKKU_OMNISCI_LOLIPOP_NAME=/lolipop/DATABASE
DOKKU_OMNISCI_LOLIPOP_PORT=tcp://172.17.0.1:6274
DOKKU_OMNISCI_LOLIPOP_PORT_6274_TCP=tcp://172.17.0.1:6274
DOKKU_OMNISCI_LOLIPOP_PORT_6274_TCP_PROTO=tcp
DOKKU_OMNISCI_LOLIPOP_PORT_6274_TCP_PORT=6274
DOKKU_OMNISCI_LOLIPOP_PORT_6274_TCP_ADDR=172.17.0.1
```

The following will be set on the linked application by default:

```
OMNISCI_URL=omnisci://lolipop:SOME_PASSWORD@dokku-omnisci-lolipop:6274/lolipop
```

The host exposed here only works internally in docker containers. If you want your container to be reachable from outside, you should use the 'expose' subcommand. Another service can be linked to your app:

```shell
dokku omnisci:link other_service playground
```

It is possible to change the protocol for `OMNISCI_URL` by setting the environment variable `OMNISCI_DATABASE_SCHEME` on the app. Doing so will after linking will cause the plugin to think the service is not linked, and we advise you to unlink before proceeding. 

```shell
dokku config:set playground OMNISCI_DATABASE_SCHEME=omnisci2
dokku omnisci:link lolipop playground
```

This will cause `OMNISCI_URL` to be set as:

```
omnisci2://lolipop:SOME_PASSWORD@dokku-omnisci-lolipop:6274/lolipop
```

### unlink the omnisci service from the app

```shell
# usage
dokku omnisci:unlink <service> <app>
```

You can unlink a omnisci service:

> NOTE: this will restart your app and unset related environment variables

```shell
dokku omnisci:unlink lolipop playground
```

### Service Lifecycle

The lifecycle of each service can be managed through the following commands:

### connect to the service via the omnisci connection tool

```shell
# usage
dokku omnisci:connect <service>
```

Connect to the service via the omnisci connection tool:

```shell
dokku omnisci:connect lolipop
```

### enter or run a command in a running omnisci service container

```shell
# usage
dokku omnisci:enter <service>
```

A bash prompt can be opened against a running service. Filesystem changes will not be saved to disk. 

```shell
dokku omnisci:enter lolipop
```

You may also run a command directly against the service. Filesystem changes will not be saved to disk. 

```shell
dokku omnisci:enter lolipop touch /tmp/test
```

### expose a omnisci service on custom port if provided (random port otherwise)

```shell
# usage
dokku omnisci:expose <service> <ports...>
```

Expose the service on the service's normal ports, allowing access to it from the public interface (`0.0.0.0`):

```shell
dokku omnisci:expose lolipop 6274 6278
```

### unexpose a previously exposed omnisci service

```shell
# usage
dokku omnisci:unexpose <service>
```

Unexpose the service, removing access to it from the public interface (`0.0.0.0`):

```shell
dokku omnisci:unexpose lolipop
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
DOKKU_OMNISCI_SILVER_URL=omnisci://lolipop:SOME_PASSWORD@dokku-omnisci-lolipop:6274/lolipop
```

### start a previously stopped omnisci service

```shell
# usage
dokku omnisci:start <service>
```

Start the service:

```shell
dokku omnisci:start lolipop
```

### stop a running omnisci service

```shell
# usage
dokku omnisci:stop <service>
```

Stop the service and the running container:

```shell
dokku omnisci:stop lolipop
```

### graceful shutdown and restart of the omnisci service container

```shell
# usage
dokku omnisci:restart <service>
```

Restart the service:

```shell
dokku omnisci:restart lolipop
```

### upgrade service <service> to the specified versions

```shell
# usage
dokku omnisci:upgrade <service> [--upgrade-flags...]
```

flags:

- `-C|--custom-env "USER=alpha;HOST=beta"`: semi-colon delimited environment variables to start the service with
- `-i|--image IMAGE`: the image name to start the service with
- `-I|--image-version IMAGE_VERSION`: the image version to start the service with
- `-R|--restart-apps "true"`: whether to force an app restart

You can upgrade an existing service to a new image or image-version:

```shell
dokku omnisci:upgrade lolipop
```

### Service Automation

Service scripting can be executed using the following commands:

### list all omnisci service links for a given app

```shell
# usage
dokku omnisci:app-links <app>
```

List all omnisci services that are linked to the 'playground' app. 

```shell
dokku omnisci:app-links playground
```

### check if the omnisci service exists

```shell
# usage
dokku omnisci:exists <service>
```

Here we check if the lolipop omnisci service exists. 

```shell
dokku omnisci:exists lolipop
```

### check if the omnisci service is linked to an app

```shell
# usage
dokku omnisci:linked <service> <app>
```

Here we check if the lolipop omnisci service is linked to the 'playground' app. 

```shell
dokku omnisci:linked lolipop playground
```

### list all apps linked to the omnisci service

```shell
# usage
dokku omnisci:links <service>
```

List all apps linked to the 'lolipop' omnisci service. 

```shell
dokku omnisci:links lolipop
```

### Disabling `docker pull` calls

If you wish to disable the `docker pull` calls that the plugin triggers, you may set the `OMNISCI_DISABLE_PULL` environment variable to `true`. Once disabled, you will need to pull the service image you wish to deploy as shown in the `stderr` output.

Please ensure the proper images are in place when `docker pull` is disabled.
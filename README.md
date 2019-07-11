# dokku omnisci [![Build Status](https://img.shields.io/travis/dokku/dokku-omnisci.svg?branch=master "Build Status")](https://travis-ci.org/dokku/dokku-omnisci) [![IRC Network](https://img.shields.io/badge/irc-freenode-blue.svg "IRC Freenode")](https://webchat.freenode.net/?channels=dokku)

Official omnisci plugin for dokku. Currently defaults to installing [omnisci/core-os-cpu v4.7.1](https://hub.docker.com/_/omnisci/core-os-cpu/).

## requirements

- dokku 0.12.x+
- docker 1.8.x

## installation

```shell
# on 0.12.x+
sudo dokku plugin:install https://github.com/dokku/dokku-omnisci.git omnisci
```

## commands

```
omnisci:app-links <app>          List all omnisci service links for a given app
omnisci:backup <name> <bucket> (--use-iam) Create a backup of the omnisci service to an existing s3 bucket
omnisci:backup-auth <name> <aws_access_key_id> <aws_secret_access_key> (<aws_default_region>) (<aws_signature_version>) (<endpoint_url>) Sets up authentication for backups on the omnisci service
omnisci:backup-deauth <name>     Removes backup authentication for the omnisci service
omnisci:backup-schedule <name> <schedule> <bucket> Schedules a backup of the omnisci service
omnisci:backup-schedule-cat <name> Cat the contents of the configured backup cronfile for the service
omnisci:backup-set-encryption <name> <passphrase> Set a GPG passphrase for backups
omnisci:backup-unschedule <name> Unschedules the backup of the omnisci service
omnisci:backup-unset-encryption <name> Removes backup encryption for future backups of the omnisci service
omnisci:clone <name> <new-name>  Create container <new-name> then copy data from <name> into <new-name>
omnisci:connect <name>           Connect via omnisci to a omnisci service
omnisci:create <name>            Create a omnisci service with environment variables
omnisci:destroy <name>           Delete the service, delete the data and stop its container if there are no links left
omnisci:enter <name> [command]   Enter or run a command in a running omnisci service container
omnisci:exists <service>         Check if the omnisci service exists
omnisci:export <name> > <file>   Export a dump of the omnisci service database
omnisci:expose <name> [port]     Expose a omnisci service on custom port if provided (random port otherwise)
omnisci:import <name> < <file>   Import a dump into the omnisci service database
omnisci:info <name>              Print the connection information
omnisci:link <name> <app>        Link the omnisci service to the app
omnisci:linked <name> <app>      Check if the omnisci service is linked to an app
omnisci:list                     List all omnisci services
omnisci:logs <name> [-t]         Print the most recent log(s) for this service
omnisci:promote <name> <app>     Promote service <name> as DATABASE_URL in <app>
omnisci:restart <name>           Graceful shutdown and restart of the omnisci service container
omnisci:start <name>             Start a previously stopped omnisci service
omnisci:stop <name>              Stop a running omnisci service
omnisci:unexpose <name>          Unexpose a previously exposed omnisci service
omnisci:unlink <name> <app>      Unlink the omnisci service from the app
omnisci:upgrade <name>           Upgrade service <service> to the specified version
```

## usage

```shell
# create a omnisci service named lolipop
dokku omnisci:create lolipop

# you can also specify the image and image
# version to use for the service
# it *must* be compatible with the
# official omnisci image
export OMNISCI_IMAGE="omnisci"
export OMNISCI_IMAGE_VERSION="v4.7.0"
dokku omnisci:create lolipop

# you can also specify custom environment
# variables to start the omnisci service
# in semi-colon separated form
export OMNISCI_CUSTOM_ENV="USER=alpha;HOST=beta"
dokku omnisci:create lolipop

# get connection information as follows
dokku omnisci:info lolipop

# you can also retrieve a specific piece of service info via flags
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

# a bash prompt can be opened against a running service
# filesystem changes will not be saved to disk
dokku omnisci:enter lolipop

# you may also run a command directly against the service
# filesystem changes will not be saved to disk
dokku omnisci:enter lolipop ls -lah /

# a omnisci service can be linked to a
# container this will use native docker
# links via the docker-options plugin
# here we link it to our 'playground' app
# NOTE: this will restart your app
dokku omnisci:link lolipop playground

# the following environment variables will be set automatically by docker (not
# on the app itself, so they won’t be listed when calling dokku config)
#
#   DOKKU_OMNISCI_LOLIPOP_NAME=/lolipop/DATABASE
#   DOKKU_OMNISCI_LOLIPOP_PORT=tcp://172.17.0.1:6274
#   DOKKU_OMNISCI_LOLIPOP_PORT_6274_TCP=tcp://172.17.0.1:6274
#   DOKKU_OMNISCI_LOLIPOP_PORT_6274_TCP_PROTO=tcp
#   DOKKU_OMNISCI_LOLIPOP_PORT_6274_TCP_PORT=6274
#   DOKKU_OMNISCI_LOLIPOP_PORT_6274_TCP_ADDR=172.17.0.1
#
# and the following will be set on the linked application by default
#
#   DATABASE_URL=omnisci://omnisci:SOME_PASSWORD@dokku-omnisci-lolipop:6274/lolipop
#
# NOTE: the host exposed here only works internally in docker containers. If
# you want your container to be reachable from outside, you should use `expose`.

# another service can be linked to your app
dokku omnisci:link other_service playground

# since DATABASE_URL is already in use, another environment variable will be
# generated automatically
#
#   DOKKU_OMNISCI_BLUE_URL=omnisci://omnisci:ANOTHER_PASSWORD@dokku-omnisci-other-service:6274/other_service

# you can then promote the new service to be the primary one
# NOTE: this will restart your app
dokku omnisci:promote other_service playground

# this will replace DATABASE_URL with the url from other_service and generate
# another environment variable to hold the previous value if necessary.
# you could end up with the following for example:
#
#   DATABASE_URL=omnisci://omnisci:ANOTHER_PASSWORD@dokku-omnisci-other_service:6274/other_service
#   DOKKU_OMNISCI_BLUE_URL=omnisci://omnisci:ANOTHER_PASSWORD@dokku-omnisci-other-service:6274/other_service
#   DOKKU_OMNISCI_SILVER_URL=omnisci://omnisci:SOME_PASSWORD@dokku-omnisci-lolipop:6274/lolipop

# you can also unlink a omnisci service
# NOTE: this will restart your app and unset related environment variables
dokku omnisci:unlink lolipop playground

# you can tail logs for a particular service
dokku omnisci:logs lolipop
dokku omnisci:logs lolipop -t # to tail

# you can dump the database
dokku omnisci:export lolipop > lolipop.sql

# you can import a dump
dokku omnisci:import lolipop < database.sql

# you can clone an existing database to a new one
dokku omnisci:clone lolipop new_database

# finally, you can destroy the container
dokku omnisci:destroy lolipop
```

## Changing database adapter

It's possible to change the protocol for DATABASE_URL by setting
the environment variable OMNISCI_DATABASE_SCHEME on the app:

```
dokku config:set playground OMNISCI_DATABASE_SCHEME=omnisci2
dokku omnisci:link lolipop playground
```

Will cause DATABASE_URL to be set as
omnisci2://omnisci:SOME_PASSWORD@dokku-omnisci-lolipop:6274/lolipop

CAUTION: Changing OMNISCI_DATABASE_SCHEME after linking will cause dokku to
believe the service is not linked when attempting to use `dokku omnisci:unlink`
or `dokku omnisci:promote`.
You should be able to fix this by

- Changing DATABASE_URL manually to the new value.

OR

- Set OMNISCI_DATABASE_SCHEME back to its original setting
- Unlink the service
- Change OMNISCI_DATABASE_SCHEME to the desired setting
- Relink the service

## Disabling `docker pull` calls

If you wish to disable the `docker pull` calls that the plugin triggers, you may set the `OMNISCI_DISABLE_PULL` environment variable to `true`. Once disabled, you will need to pull the service image you wish to deploy as shown in the `stderr` output.

Please ensure the proper images are in place when `docker pull` is disabled.

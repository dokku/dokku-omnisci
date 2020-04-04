# dokku omnisci [![Build Status](https://img.shields.io/travis/dokku/dokku-omnisci.svg?branch=master "Build Status")](https://travis-ci.org/dokku/dokku-omnisci) [![IRC Network](https://img.shields.io/badge/irc-freenode-blue.svg "IRC Freenode")](https://webchat.freenode.net/?channels=dokku)

Official omnisci plugin for dokku. Currently defaults to installing [omnisci v5.1.2](https://hub.docker.com/_/omnisci/).

## Requirements

- dokku 0.12.x+
- docker 1.8.x

## Installation

```shell
# on 0.12.x+
sudo dokku plugin:install https://github.com/dokku/dokku-omnisci.git omnisci
```

## Commands

```
omnisci:app-links <app>                            # list all omnisci service links for a given app
omnisci:backup <service> <bucket-name> [--use-iam] # creates a backup of the omnisci service to an existing s3 bucket
omnisci:backup-auth <service> <aws-access-key-id> <aws-secret-access-key> <aws-default-region> <aws-signature-version> <endpoint-url> # sets up authentication for backups on the omnisci service
omnisci:backup-deauth <service>                    # removes backup authentication for the omnisci service
omnisci:backup-schedule <service> <schedule> <bucket-name> [--use-iam] # schedules a backup of the omnisci service
omnisci:backup-schedule-cat <service>              # cat the contents of the configured backup cronfile for the service
omnisci:backup-set-encryption <service> <passphrase> # sets encryption for all future backups of omnisci service
omnisci:backup-unschedule <service>                # unschedules the backup of the omnisci service
omnisci:backup-unset-encryption <service>          # unsets encryption for future backups of the omnisci service
omnisci:clone <service> <new-service> [--clone-flags...] # create container <new-name> then copy data from <name> into <new-name>
omnisci:connect <service>                          # connect to the service via the omnisci connection tool
omnisci:create <service> [--create-flags...]       # create a omnisci service
omnisci:destroy <service> [-f|--force]             # delete the omnisci service/data/container if there are no links left
omnisci:enter <service>                            # enter or run a command in a running omnisci service container
omnisci:exists <service>                           # check if the omnisci service exists
omnisci:export <service>                           # export a dump of the omnisci service database
omnisci:expose <service> <ports...>                # expose a omnisci service on custom port if provided (random port otherwise)
omnisci:import <service>                           # import a dump into the omnisci service database
omnisci:info <service> [--single-info-flag]        # print the service information
omnisci:link <service> <app> [--link-flags...]     # link the omnisci service to the app
omnisci:linked <service> <app>                     # check if the omnisci service is linked to an app
omnisci:links <service>                            # list all apps linked to the omnisci service
omnisci:list                                       # list all omnisci services
omnisci:logs <service> [-t|--tail]                 # print the most recent log(s) for this service
omnisci:promote <service> <app>                    # promote service <service> as DATABASE_URL in <app>
omnisci:restart <service>                          # graceful shutdown and restart of the omnisci service container
omnisci:start <service>                            # start a previously stopped omnisci service
omnisci:stop <service>                             # stop a running omnisci service
omnisci:unexpose <service>                         # unexpose a previously exposed omnisci service
omnisci:unlink <service> <app>                     # unlink the omnisci service from the app
omnisci:upgrade <service> [--upgrade-flags...]     # upgrade service <service> to the specified versions
```

## Usage

Help for any commands can be displayed by specifying the command as an argument to omnisci:help. Please consult the `omnisci:help` command for any undocumented commands.

### Basic Usage

### create a omnisci service

```shell
# usage
dokku omnisci:create <service> [--create-flags...]
```

Create a omnisci service named lolipop:

```shell
dokku omnisci:create lolipop
```

You can also specify the image and image version to use for the service. It *must* be compatible with the ${plugin_image} image.

```shell
export DATABASE_IMAGE="${PLUGIN_IMAGE}"
export DATABASE_IMAGE_VERSION="${PLUGIN_IMAGE_VERSION}"
dokku omnisci:create lolipop
```

You can also specify custom environment variables to start the omnisci service in semi-colon separated form.

```shell
export DATABASE_CUSTOM_ENV="USER=alpha;HOST=beta"
dokku omnisci:create lolipop
```

### print the service information

```shell
# usage
dokku omnisci:info <service> [--single-info-flag]
```

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

A omnisci service can be linked to a container. This will use native docker links via the docker-options plugin. Here we link it to our 'playground' app.

> NOTE: this will restart your app

```shell
dokku omnisci:link lolipop playground
```

The following environment variables will be set automatically by docker (not on the app itself, so they won’t be listed when calling dokku config):

```
DOKKU_DATABASE_LOLIPOP_NAME=/lolipop/DATABASE
DOKKU_DATABASE_LOLIPOP_PORT=tcp://172.17.0.1:6274
DOKKU_DATABASE_LOLIPOP_PORT_6274_TCP=tcp://172.17.0.1:6274
DOKKU_DATABASE_LOLIPOP_PORT_6274_TCP_PROTO=tcp
DOKKU_DATABASE_LOLIPOP_PORT_6274_TCP_PORT=6274
DOKKU_DATABASE_LOLIPOP_PORT_6274_TCP_ADDR=172.17.0.1
```

The following will be set on the linked application by default:

```
DATABASE_URL=omnisci://lolipop:SOME_PASSWORD@dokku-omnisci-lolipop:6274/lolipop
```

The host exposed here only works internally in docker containers. If you want your container to be reachable from outside, you should use the 'expose' subcommand. Another service can be linked to your app:

```shell
dokku omnisci:link other_service playground
```

It is possible to change the protocol for database_url by setting the environment variable database_database_scheme on the app. Doing so will after linking will cause the plugin to think the service is not linked, and we advise you to unlink before proceeding.

```shell
dokku config:set playground DATABASE_DATABASE_SCHEME=omnisci2
dokku omnisci:link lolipop playground
```

This will cause database_url to be set as:

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

Expose the service on the service's normal ports, allowing access to it from the public interface (0. 0. 0. 0):

```shell
dokku omnisci:expose lolipop ${PLUGIN_DATASTORE_PORTS[@]}
```

### unexpose a previously exposed omnisci service

```shell
# usage
dokku omnisci:unexpose <service>
```

Unexpose the service, removing access to it from the public interface (0. 0. 0. 0):

```shell
dokku omnisci:unexpose lolipop
```

### promote service <service> as DATABASE_URL in <app>

```shell
# usage
dokku omnisci:promote <service> <app>
```

If you have a omnisci service linked to an app and try to link another omnisci service another link environment variable will be generated automatically:

```
DOKKU_DATABASE_BLUE_URL=omnisci://other_service:ANOTHER_PASSWORD@dokku-omnisci-other-service:6274/other_service
```

You can promote the new service to be the primary one:

> NOTE: this will restart your app

```shell
dokku omnisci:promote other_service playground
```

This will replace database_url with the url from other_service and generate another environment variable to hold the previous value if necessary. You could end up with the following for example:

```
DATABASE_URL=omnisci://other_service:ANOTHER_PASSWORD@dokku-omnisci-other-service:6274/other_service
DOKKU_DATABASE_BLUE_URL=omnisci://other_service:ANOTHER_PASSWORD@dokku-omnisci-other-service:6274/other_service
DOKKU_DATABASE_SILVER_URL=omnisci://lolipop:SOME_PASSWORD@dokku-omnisci-lolipop:6274/lolipop
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

### create container <new-name> then copy data from <name> into <new-name>

```shell
# usage
dokku omnisci:clone <service> <new-service> [--clone-flags...]
```

You can clone an existing service to a new one:

```shell
dokku omnisci:clone lolipop lolipop-2
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

### Data Management

The underlying service data can be imported and exported with the following commands:

### import a dump into the omnisci service database

```shell
# usage
dokku omnisci:import <service>
```

Import a datastore dump:

```shell
dokku omnisci:import lolipop < database.dump
```

### export a dump of the omnisci service database

```shell
# usage
dokku omnisci:export <service>
```

By default, datastore output is exported to stdout:

```shell
dokku omnisci:export lolipop
```

You can redirect this output to a file:

```shell
dokku omnisci:export lolipop > lolipop.dump
```

### Backups

Datastore backups are supported via AWS S3 and S3 compatible services like [minio](https://github.com/minio/minio).

You may skip the `backup-auth` step if your dokku install is running within EC2 and has access to the bucket via an IAM profile. In that case, use the `--use-iam` option with the `backup` command.

Backups can be performed using the backup commands:

### sets up authentication for backups on the omnisci service

```shell
# usage
dokku omnisci:backup-auth <service> <aws-access-key-id> <aws-secret-access-key> <aws-default-region> <aws-signature-version> <endpoint-url>
```

Setup s3 backup authentication:

```shell
dokku omnisci:backup-auth lolipop AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY
```

Setup s3 backup authentication with different region:

```shell
dokku omnisci:backup-auth lolipop AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY AWS_REGION
```

Setup s3 backup authentication with different signature version and endpoint:

```shell
dokku omnisci:backup-auth lolipop AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY AWS_REGION AWS_SIGNATURE_VERSION ENDPOINT_URL
```

More specific example for minio auth:

```shell
dokku omnisci:backup-auth lolipop MINIO_ACCESS_KEY_ID MINIO_SECRET_ACCESS_KEY us-east-1 s3v4 https://YOURMINIOSERVICE
```

### removes backup authentication for the omnisci service

```shell
# usage
dokku omnisci:backup-deauth <service>
```

Remove s3 authentication:

```shell
dokku omnisci:backup-deauth lolipop
```

### creates a backup of the omnisci service to an existing s3 bucket

```shell
# usage
dokku omnisci:backup <service> <bucket-name> [--use-iam]
```

Backup the 'lolipop' service to the 'my-s3-bucket' bucket on aws:

```shell
dokku omnisci:backup lolipop my-s3-bucket --use-iam
```

### sets encryption for all future backups of omnisci service

```shell
# usage
dokku omnisci:backup-set-encryption <service> <passphrase>
```

Set a gpg passphrase for backups:

```shell
dokku omnisci:backup-set-encryption lolipop
```

### unsets encryption for future backups of the omnisci service

```shell
# usage
dokku omnisci:backup-unset-encryption <service>
```

Unset a gpg encryption key for backups:

```shell
dokku omnisci:backup-unset-encryption lolipop
```

### schedules a backup of the omnisci service

```shell
# usage
dokku omnisci:backup-schedule <service> <schedule> <bucket-name> [--use-iam]
```

Schedule a backup:

> 'schedule' is a crontab expression, eg. "0 3 * * *" for each day at 3am

```shell
dokku omnisci:backup-schedule lolipop "0 3 * * *" my-s3-bucket
```

Schedule a backup and authenticate via iam:

```shell
dokku omnisci:backup-schedule lolipop "0 3 * * *" my-s3-bucket --use-iam
```

### cat the contents of the configured backup cronfile for the service

```shell
# usage
dokku omnisci:backup-schedule-cat <service>
```

Cat the contents of the configured backup cronfile for the service:

```shell
dokku omnisci:backup-schedule-cat lolipop
```

### unschedules the backup of the omnisci service

```shell
# usage
dokku omnisci:backup-unschedule <service>
```

Remove the scheduled backup from cron:

```shell
dokku omnisci:backup-unschedule lolipop
```

### Disabling `docker pull` calls

If you wish to disable the `docker pull` calls that the plugin triggers, you may set the `OMNISCI_DISABLE_PULL` environment variable to `true`. Once disabled, you will need to pull the service image you wish to deploy as shown in the `stderr` output.

Please ensure the proper images are in place when `docker pull` is disabled.
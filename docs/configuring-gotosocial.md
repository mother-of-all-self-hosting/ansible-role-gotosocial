<!--
SPDX-FileCopyrightText: 2020 - 2024 MDAD project contributors
SPDX-FileCopyrightText: 2020 - 2024 Slavi Pantaleev
SPDX-FileCopyrightText: 2020 Aaron Raimist
SPDX-FileCopyrightText: 2020 Chris van Dijk
SPDX-FileCopyrightText: 2020 Dominik Zajac
SPDX-FileCopyrightText: 2020 Mickaël Cornière
SPDX-FileCopyrightText: 2022 François Darveau
SPDX-FileCopyrightText: 2022 Julian Foad
SPDX-FileCopyrightText: 2022 Warren Bailey
SPDX-FileCopyrightText: 2023 Antonis Christofides
SPDX-FileCopyrightText: 2023 Felix Stupp
SPDX-FileCopyrightText: 2023 Pierre 'McFly' Marty
SPDX-FileCopyrightText: 2024 - 2025 Suguru Hirahara

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Setting up Docmost

This is an [Ansible](https://www.ansible.com/) role which installs [Docmost](https://docmost.com/) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

Docmost is a free and open-source collaborative wiki and documentation software, designed for seamless real-time collaboration. It can be used to manage a wiki, a knowledge base, project documentation, etc. It has various functions such as granular permissions management system, page history to track changes of articles, etc. It also supports diagramming tools like Draw.io, Excalidraw and Mermaid.

See the project's [documentation](https://docmost.com/docs/) to learn what Docmost does and why it might be useful to you.

## Prerequisites

To run a Docmost instance it is necessary to prepare a [Redis](https://redis.io/) server for managing a metadata database.

If you are looking for an Ansible role for Redis, you can check out [this role (ansible-role-redis)](https://github.com/mother-of-all-self-hosting/ansible-role-redis) maintained by the [Mother-of-All-Self-Hosting (MASH)](https://github.com/mother-of-all-self-hosting) team. Note that the team recommends to have a look at [this role (ansible-role-valkey)](https://github.com/mother-of-all-self-hosting/ansible-role-valkey) for [Valkey](https://valkey.io/) instead.

## Adjusting the playbook configuration

To enable Docmost with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# docmost                                                              #
#                                                                      #
########################################################################

docmost_enabled: true

########################################################################
#                                                                      #
# /docmost                                                             #
#                                                                      #
########################################################################
```

### Set the hostname

To enable the Docmost instance you need to set the hostname as well. To do so, add the following configuration to your `vars.yml` file. Make sure to replace `example.com` with your own value.

```yaml
docmost_hostname: "example.com"
```

After adjusting the hostname, make sure to adjust your DNS records to point the domain to your server.

**Note**: hosting Docmost under a subpath (by configuring the `docmost_path_prefix` variable) does not seem to be possible due to Docmost's technical limitations.

### Set variables for connecting to a Redis server

As described above, it is necessary to set up a [Redis](https://redis.io/) server for managing a metadata database of a Docmost instance. You can use either KeyDB or Valkey alternatively.

Having configured it, you need to add and adjust the following configuration to your `vars.yml` file, so that the Docmost instance will connect to the server:

```yaml
docmost_redis_username: ''
docmost_redis_password: ''
docmost_redis_host: YOUR_REDIS_SERVER_HOSTNAME_HERE
docmost_redis_port: 6379
docmost_redis_dbnumber: ''
```

Make sure to replace `YOUR_REDIS_SERVER_HOSTNAME_HERE` with the hostname of your Redis server. If the Redis server runs on the same host as Docmost, set `localhost`.

### Configure a storage backend

The service provides these storage backend options: local filesystem (default) and Amazon S3 compatible object storage.

#### Local filesystem (default)

**By default this role removes uploaded files when uninstalling the service**. In order to make those files persistent, you need to add a Docker volume to mount in the container, so that the directory for storing files is shared with the host machine.

To add the volume, prepare a directory on the host machine and add the following configuration to your `vars.yml` file:

```yaml
docmost_data_path: /path/on/the/host
```

Make sure permissions of the directory specified to `/path/on/the/host`.

#### Amazon S3 compatible object storage

To use Amazon S3 or a S3 compatible object storage, add the following configuration to your `vars.yml` file (adapt to your needs):

```yaml
docmost_environment_variable_storage_driver: s3

# Set a S3 access key ID
docmost_environment_variable_aws_s3_access_key_id: ''

# Set a S3 secret access key ID
docmost_environment_variable_aws_s3_secret_access_key: ''

# Set the the region where your S3 bucket is located
docmost_environment_variable_aws_s3_region: ''

# Set a S3 bucket name to use
docmost_environment_variable_aws_s3_bucket: ''

# The endpoint URL for your S3 service (optional; set if using a S3 compatible storage like Wasabi and Storj)
docmost_environment_variable_aws_s3_endpoint: ''

# Control whether to force path style URLs (https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/Config.html#s3ForcePathStyle-property) for S3 objects
docmost_environment_variable_aws_s3_force_path_style: false
```

### Configure the mailer

You can configure a mailer for functions such as user invitation. Docmost supports a SMTP server (default) and Postmark. To set it up, add the following common configuration and settings specific to SMTP server or Postmark to your `vars.yml` file as below (adapt to your needs):

```yaml
docmost_mailer_enabled: true

# Set the email address that emails will be sent from
docmost_environment_variable_mail_from_address: hello@example.com

# Set the name that emails will be sent from
docmost_environment_variable_mail_from_name: docmost
```

#### Use SMTP server (default)

To use a SMTP server, add the following configuration to your `vars.yml` file:

```yaml
# Set the hostname of the SMTP server
docmost_environment_variable_smtp_host: 127.0.0.1

# Set the port to use for the SMTP server
docmost_environment_variable_smtp_port: 587

# Set the username for the SMTP server
docmost_environment_variable_smtp_username: ''

# Set the password for the SMTP server
docmost_environment_variable_smtp_password: ''

# Control whether TLS is used when connecting to the server
docmost_environment_variable_smtp_secure: false

# Control whether SSL errors are ignored
docmost_environment_variable_smtp_ignoretls: false
```

⚠️ **Note**: without setting an authentication method such as DKIM, SPF, and DMARC for your hostname, emails are most likely to be quarantined as spam at recipient's mail servers. If you have set up a mail server with the [MASH project's exim-relay Ansible role](https://github.com/mother-of-all-self-hosting/ansible-role-exim-relay), you can enable DKIM signing with it. Refer [its documentation](https://github.com/mother-of-all-self-hosting/ansible-role-exim-relay/blob/main/docs/configuring-exim-relay.md#enable-dkim-support-optional) for details.

#### Use Postmark

To use Postmark, add the following configuration to your `vars.yml` file:

```yaml
docmost_environment_variable_mail_driver: postmark

# Set the token for Postmark
docmost_environment_variable_postmark_token: ''
```

### Extending the configuration

There are some additional things you may wish to configure about the component.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `docmost_environment_variables_additional_variables` variable

For a complete list of Docmost's config options that you could put in `docmost_environment_variables_additional_variables`, see its [environment variables](https://docmost.com/docs/self-hosting/environment-variables).

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, Docmost becomes available at the specified hostname like `https://example.com`.

To get started, go to the URL on a web browser and create a first workspace by inputting required information. For an email address, make sure to input your own email address, not the one specified to `docmost_environment_variable_mail_from_address`.

## Troubleshooting

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH and running `journalctl -fu docmost` (or how you/your playbook named the service, e.g. `mash-docmost`).

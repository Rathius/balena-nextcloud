# balena-nextcloud

nextcloud stack for aarch64 on balenaCloud

DO NOT USE - this is a work in progress and requires substituting environment variables at build time

<https://forums.balena.io/t/environment-variables-are-not-working-at-build-time/5176>

## Requirements

* RaspberryPi3 or a similar aarch64 device supported by BalenaCloud
* Custom domain name with Cloudflare DNS (eg. nextcloud.mydomain.com)

## Getting Started

To get started you'll first need to sign up for a free balenaCloud account and flash your device.

<https://www.balena.io/docs/learn/getting-started>

## Deployment

Once your account is set up, deployment is carried out by downloading the project and pushing it to your device either via Git or the balena CLI.

### Application Environment Variables

Application envionment variables apply to all services within the application, and can be applied fleet-wide to apply to multiple devices.

|Name|Example|Purpose|
|---|---|---|
|`TRAEFIK_LOG_LEVEL`|`DEBUG`|Log level set to traefik logs. (Default: `ERROR`)|
|`TRAEFIK_CERTIFICATESRESOLVERS_MYDNSCHALLENGE_ACME_EMAIL`|`foo@bar.com`|Email address used for registration.|
|`TRAEFIK_CERTIFICATESRESOLVERS_MYDNSCHALLENGE_ACME_CASERVER`|`https://acme-staging-v02.api.letsencrypt.org/directory`|CA server to use. (Default: `https://acme-v02.api.letsencrypt.org/directory`)|
|`TRAEFIK_PROVIDERS_DOCKER_DEFAULTRULE`|``Host(`{{ normalize .Name }}.mydomain.com`)``|Default rule. (Default: ``Host(`{{ normalize .Name }}`))``|
|`CF_API_EMAIL`|`foo@bar.com`|Cloudflare account email.|
|`CF_API_KEY`|`b9841238feb177a84330febba8a83208921177bffe733`|Cloudflare global API key.|
|`NEXTCLOUD_ADMIN_USER`|`admin`|Name of the Nextcloud admin user.|
|`NEXTCLOUD_ADMIN_PASSWORD`|`my-secret-pw`|Password for the Nextcloud admin user.|
|`NEXTCLOUD_TRUSTED_DOMAINS`|`nextcloud.mydomain.com,*.balena-devices.com`|Optional space-separated list of domains|
|`MYSQL_DATABASE`|`nextcloud`|This variable is optional and allows you to specify the name of a database to be created on image startup. If a user/password was supplied (see below) then that user will be granted superuser access (corresponding to GRANT ALL) to this database.|
|`MYSQL_USER`|`nextcloud`|These variables are optional, used in conjunction to create a new user and to set that user's password. This user will be granted superuser permissions (see above) for the database specified by the MYSQL_DATABASE variable. Both variables are required for a user to be created.|
|`MYSQL_PASSWORD`|`nextcloud`|These variables are optional, used in conjunction to create a new user and to set that user's password. This user will be granted superuser permissions (see above) for the database specified by the MYSQL_DATABASE variable. Both variables are required for a user to be created.|
|`MYSQL_ROOT_PASSWORD`|`my-secret-pw`|This variable is mandatory and specifies the password that will be set for the MariaDB root superuser account.|

## Usage

_TODO_

## Author

Kyle Harding <https://klutchell.dev>

## Acknowledgments

* <https://hub.docker.com/_/nextcloud/>
* <https://hub.docker.com/_/mariadb/>
* <https://hub.docker.com/_/traefik/>
* <https://go-acme.github.io/lego/dns/cloudflare/>

## License

[MIT License](./LICENSE)

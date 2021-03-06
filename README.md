# balena-nextcloud

nextcloud stack for balenaCloud

## Requirements

- Raspberry Pi 4 or a similar x64 device supported by BalenaCloud
- (optional) Custom domain name with DNS pointing to your balena device (eg. nextcloud.example.com)
- (optional) USB storage device
- (optional) Network storage share

## Getting Started

To get started you'll first need to sign up for a free balenaCloud account and flash your device.

<https://www.balena.io/docs/learn/getting-started>

## Deployment

Once your account is set up, deployment is carried out by downloading the project and pushing it to your device either via Git or the balenaCLI.

### Application Environment Variables

Application envionment variables apply to all services within the application, and can be applied fleet-wide to apply to multiple devices.

|Name|Example|Purpose|
|---|---|---|
|`NEXTCLOUD_TRUSTED_DOMAINS`|`<server-ip> nextcloud.example.com *.balena-devices.com`|space-separated list of trusted domains for remote access|
|`MYSQL_ROOT_PASSWORD`|`********`|password that will be set for the MariaDB root account|
|`MYSQL_PASSWORD`|`********`|password that will be set for the MariaDB nextcloud account|
|`TRAEFIK_PROVIDERS_DOCKER_DEFAULTRULE`|``Host(`nextcloud.example.com`)``|provide your custom domain here|
|`TRAEFIK_CERTIFICATESRESOLVERS_TLSCHALLENGE_ACME_EMAIL`|`foo@bar.com`|email address to use for ACME registration|
|`TRAEFIK_CERTIFICATESRESOLVERS_TLSCHALLENGE_ACME_CASERVER`|`https://acme-staging-v02.api.letsencrypt.org/directory`|(optional) specify a different CA server to use|
|`TZ`|`America/Toronto`|(optional) inform services of the [timezone](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) in your location|
|`EXTRA_MOUNT`|`//192.168.8.1/sda1`|(optional) path to mount `/mnt/storage` on startup|
|`EXTRA_MOUNT_OPTS`|`vers=1.0,username=guest`|(optional) options to mount `/mnt/storage` on startup|

## Usage

### prepare external storage

Connect to the `Host OS` Terminal and run the following:

```bash
# g - create a new empty GPT partition table
# n - add a new partition
# p - primary partition
# 1 - partition number 1
# default - start at beginning of disk
# default - extend partition to end of disk
# w - write the partition table
printf "g\nn\np\n1\n\n\nw\n" | fdisk /dev/sda
mkfs.ext4 /dev/sda1
```

Restart the `nextcloud` service and any supported partitions will be mounted at `/media/{UUID}`.

Add the storage location in the Nextcloud dashboard under Settings -> Administration -> External Storages -> Add Storage -> Local.

The system path to the mount location(s) are printed in the logs.

### fix nextcloud proxy warnings

Connect to the `nextcloud` terminal and run the following:

```bash
# turn on nextcloud maintenance mode
sudo -u www-data php /var/www/html/occ maintenance:mode --on

# example: get/set trusted domains (optional)
sudo -u www-data php /var/www/html/occ config:system:get trusted_domains
sudo -u www-data php /var/www/html/occ config:system:set trusted_domains 0 --value='192.168.8.6'
sudo -u www-data php /var/www/html/occ config:system:set trusted_domains 1 --value='nextcloud.example.com'
sudo -u www-data php /var/www/html/occ config:system:set trusted_domains 2 --value='*.balena-devices.com'
sudo -u www-data php /var/www/html/occ config:system:set trusted_domains 3 --value='nextcloud.lan'

# example: get/set trusted proxies (optional)
sudo -u www-data php /var/www/html/occ config:system:get trusted_proxies
sudo -u www-data php /var/www/html/occ config:system:set trusted_proxies 0 --value='traefik'
sudo -u www-data php /var/www/html/occ config:system:set trusted_proxies 1 --value='localhost'

# example: get/set overwrite values (optional)
sudo -u www-data php /var/www/html/occ config:system:get overwrite.cli.url
sudo -u www-data php /var/www/html/occ config:system:set overwrite.cli.url --value='https://nextcloud.example.com/'

sudo -u www-data php /var/www/html/occ config:system:get overwritehost
sudo -u www-data php /var/www/html/occ config:system:set overwritehost --value='nextcloud.example.com'

sudo -u www-data php /var/www/html/occ config:system:get overwriteprotocol
sudo -u www-data php /var/www/html/occ config:system:set overwriteprotocol --value='https'

# turn off nextcloud maintenance mode
sudo -u www-data php /var/www/html/occ maintenance:mode --off
```

- <https://docs.nextcloud.com/server/latest/admin_manual/configuration_server/reverse_proxy_configuration.html>
- <https://docs.nextcloud.com/server/latest/admin_manual/configuration_server/occ_command.html>

### fix nextcloud database warnings

Connect to the `nextcloud` terminal and run the following:

```bash
# turn on nextcloud maintenance mode
sudo -u www-data php /var/www/html/occ maintenance:mode --on

# fix nextcloud database warnings
sudo -u www-data php /var/www/html/occ db:add-missing-indices
sudo -u www-data php /var/www/html/occ db:convert-filecache-bigint

# turn off nextcloud maintenance mode
sudo -u www-data php /var/www/html/occ maintenance:mode --off
```

- <https://docs.nextcloud.com/server/latest/admin_manual/configuration_database/linux_database_configuration.html>
- <https://docs.nextcloud.com/server/latest/admin_manual/configuration_database/bigint_identifiers.html>
- <https://docs.nextcloud.com/server/latest/admin_manual/configuration_server/occ_command.html>

### enable redis

Connect to the `nextcloud` terminal and run the following:

```bash
# turn on nextcloud maintenance mode
sudo -u www-data php /var/www/html/occ maintenance:mode --on

# enable redis service in config
sudo -u www-data php occ config:system:set redis host --value=redis
sudo -u www-data php occ config:system:set redis port --value=6379
sudo -u www-data php occ config:system:set memcache.distributed --value="\OC\Memcache\Redis"
sudo -u www-data php occ config:system:set memcache.locking --value="\OC\Memcache\Redis"

# turn off nextcloud maintenance mode
sudo -u www-data php /var/www/html/occ maintenance:mode --off
```

- <https://docs.nextcloud.com/server/latest/admin_manual/configuration_server/caching_configuration.html#id2>
- <https://docs.nextcloud.com/server/latest/admin_manual/configuration_server/occ_command.html>

### enable duplicati

Connect to `http://<device-ip>:8200` and configure a new backup using any online service you prefer as the Destination and `/source` as Source Data.

## Contributing

Please open an issue or submit a pull request with any features, fixes, or changes.

## Author

Kyle Harding <https://klutchell.dev>

[Buy me a beer](https://kyles-tip-jar.myshopify.com/cart/31356319498262:1?channel=buy_button)

[Buy me a craft beer](https://kyles-tip-jar.myshopify.com/cart/31356317859862:1?channel=buy_button)

## Acknowledgments

- <https://hub.docker.com/_/nextcloud/>
- <https://hub.docker.com/_/mariadb/>
- <https://hub.docker.com/_/redis/>
- <https://hub.docker.com/_/traefik/>
- <https://hub.docker.com/r/linuxserver/duplicati>

## License

[MIT License](./LICENSE)

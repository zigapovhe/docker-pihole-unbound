# Pi-Hole + Unbound

## Description

This Docker deployment runs both Pi-Hole and Unbound in a single container. 

Warning: This image is meant for users with dualstack IPs (IPv4 + IPv6). If you don't have native IPv6, consider forking this repo, or manually changing /etc/unbound/unbound.conf.d/pi-hole.conf file.

The base image for the container is the [official Pi-Hole container](https://hub.docker.com/r/pihole/pihole), with an extra build step added to install the Unbound resolver directly into to the container based on [instructions provided directly by the Pi-Hole team](https://docs.pi-hole.net/guides/unbound/).

## Usage

First create a `.env` file to substitute variables for your deployment. 

Then create a folder on the host machine where you want to store your unbound config. Copy the unbound-pihole.conf file into this folder and make your changes. 
> 

### Recommended Variables

Vars and descriptions replicated from the [official pihole container](https://github.com/pi-hole/docker-pi-hole/):

| Variable | Default | Value | Description |
| -------- | ------- | ----- | ---------- |
| `TZ` | UTC | `<Timezone>` | Set your [timezone](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) to make sure logs rotate at local midnight instead of at UTC midnight.
| `WEBPASSWORD` | random | `<Admin password>` | http://pi.hole/admin password. Run `docker logs pihole \| grep random` to find your random pass.
| `FTLCONF_REPLY_ADDR4` | unset | `<Host's IP>` | Set to your server's LAN IP, used by web block modes and lighttpd bind address.

### Optional Variables

| Variable | Default | Value | Description |
| -------- | ------- | ----- | ---------- |
| `ADMIN_EMAIL` | unset | email address | Set an administrative contact address for the Block Page |
| `PIHOLE_DNS_` |  `8.8.8.8;8.8.4.4` | IPs delimited by `;` | Upstream DNS server(s) for Pi-hole to forward queries to, seperated by a semicolon <br/> (supports non-standard ports with `#[port number]`) e.g `127.0.0.1#5053;8.8.8.8;8.8.4.4` |
| `DNSSEC` | `false` | `<"true"\|"false">` | Enable DNSSEC support |
| `DNS_BOGUS_PRIV` | `true` |`<"true"\|"false">`| Never forward reverse lookups for private ranges |
| `DNS_FQDN_REQUIRED` | `true` | `<"true"\|"false">`| Never forward non-FQDNs |
| `REV_SERVER` | `false` | `<"true"\|"false">` | Enable DNS conditional forwarding for device name resolution |
| `REV_SERVER_DOMAIN` | unset | Network Domain | If conditional forwarding is enabled, set the domain of the local network router |
| `REV_SERVER_TARGET` | unset | Router's IP | If conditional forwarding is enabled, set the IP of the local network router |
| `REV_SERVER_CIDR` | unset | Reverse DNS | If conditional forwarding is enabled, set the reverse DNS zone (e.g. `192.168.0.0/24`) |
| `DHCP_ACTIVE` | `false` | `<"true"\|"false">` | Enable DHCP server. Static DHCP leases can be configured with a custom `/etc/dnsmasq.d/04-pihole-static-dhcp.conf`
| `DHCP_START` | unset | `<Start IP>` | Start of the range of IP addresses to hand out by the DHCP server (mandatory if DHCP server is enabled).
| `DHCP_END` | unset | `<End IP>` | End of the range of IP addresses to hand out by the DHCP server (mandatory if DHCP server is enabled).
| `DHCP_ROUTER` | unset | `<Router's IP>` | Router (gateway) IP address sent by the DHCP server (mandatory if DHCP server is enabled).
| `DHCP_LEASETIME` | 24 | `<hours>` | DHCP lease time in hours.
| `PIHOLE_DOMAIN` | `lan` | `<domain>` | Domain name sent by the DHCP server.
| `DHCP_IPv6` | `false` | `<"true"\|"false">` | Enable DHCP server IPv6 support (SLAAC + RA).
| `DHCP_rapid_commit` | `false` | `<"true"\|"false">` | Enable DHCPv4 rapid commit (fast address assignment).
| `VIRTUAL_HOST` | `$ServerIP` | `<Custom Hostname>` | What your web server 'virtual host' is, accessing admin through this Hostname/IP allows you to make changes to the whitelist / blacklists in addition to the default 'http://pi.hole/admin/' address
| `IPv6:` | `true` | `<"true"\|"false">` | For unraid compatibility, strips out all the IPv6 configuration from DNS/Web services when false.
| `TEMPERATUREUNIT` | `c` | `<c\|k\|f>` | Set preferred temperature unit to `c`: Celsius, `k`: Kelvin, or `f` Fahrenheit units.
| `WEBUIBOXEDLAYOUT` | `boxed` | `<boxed\|traditional>` | Use boxed layout (helpful when working on large screens)
| `QUERY_LOGGING` | `true` | `<"true"\|"false">` | Enable query logging or not.
| `WEBTHEME` | `default-light` | `<"default-dark"\|"default-darker"\|"default-light"\|"default-auto"\|"lcars">`| User interface theme to use.
| `WEBPASSWORD_FILE`| unset | `<Docker secret path>` |Set an Admin password using [Docker secrets](https://docs.docker.com/engine/swarm/secrets/). If `WEBPASSWORD` is set, `WEBPASSWORD_FILE` is ignored. If `WEBPASSWORD` is empty, and `WEBPASSWORD_FILE` is set to a valid readable file path, then `WEBPASSWORD` will be set to the contents of `WEBPASSWORD_FILE`.
| `UNBOUND_CONFIG_MOUNT` | unset | `<Mount unbound config>` | Volume mount for path on host machine (`eg. './opt-unbound/:/opt/unbound/'. You should not change :/opt/unbound/`)

### Advanced Variables
| Variable | Default | Value | Description |
| -------- | ------- | ----- | ---------- |
| `INTERFACE` | unset | `<NIC>` | The default works fine with our basic example docker run commands.  If you're trying to use DHCP with `--net host` mode then you may have to customize this or DNSMASQ_LISTENING.
| `DNSMASQ_LISTENING` | unset | `<local\|all\|single>` | `local` listens on all local subnets, `all` permits listening on internet origin subnets in addition to local, `single` listens only on the interface specified.
| `WEB_PORT` | unset | `<PORT>` | **This will break the 'webpage blocked' functionality of Pi-hole** however it may help advanced setups like those running synology or `--net=host` docker argument.  This guide explains how to restore webpage blocked functionality using a linux router DNAT rule: [Alternative Synology installation method](https://discourse.pi-hole.net/t/alternative-synology-installation-method/5454?u=diginc)
| `SKIPGRAVITYONBOOT` | unset | `<unset\|1>` | Use this option to skip updating the Gravity Database when booting up the container.  By default this environment variable is not set so the Gravity Database will be updated when the container starts up.  Setting this environment variable to 1 (or anything) will cause the Gravity Database to not be updated when container starts up.
| `CORS_HOSTS` | unset | `<FQDNs delimited by ,>` | List of domains/subdomains on which CORS is allowed. Wildcards are not supported. Eg: `CORS_HOSTS: domain.com,home.domain.com,www.domain.com`.
| `CUSTOM_CACHE_SIZE` | `10000` | Number | Set the cache size for dnsmasq. Useful for increasing the default cache size or to set it to 0. Note that when `DNSSEC` is "true", then this setting is ignored.
| `FTLCONF_[SETTING]` | unset | As per documentation | Customize pihole-FTL.conf with settings described in the [FTLDNS Configuration page](https://docs.pi-hole.net/ftldns/configfile/). For example, to customize REPLY_ADDR6, ensure you have the `FTLCONF_REPLY_ADDR6` environment variable set.

### Experimental Variables
| Variable | Default | Value | Description |
| -------- | ------- | ----- | ---------- |
| `DNSMASQ_USER` | unset | `<pihole\|root>` | Allows changing the user that FTLDNS runs as. Default: `pihole`

Example `.env` file in the same directory as your `docker-compose.yaml` file:

```
FTLCONF_REPLY_ADDR4=192.168.1.10
TZ=Europe/Ljubljana
ADMIN_EMAIL=name.surname@gmail.com
WEBPASSWORD=QWERTY123456asdfASDF
WEBUIBOXEDLAYOUT=boxed
WEBTHEME=default-dark
REV_SERVER=false
DHCP_ACTIVE=true
DHCP_START=192.168.1.100
DHCP_END=192.168.1.251
DHCP_ROUTER=192.168.1.1
DHCP_LEASETIME=24
HOSTNAME=pihole
DOMAIN_NAME=pihole.local
UNBOUND_CONFIG_MOUNT=./opt-unbound/:/opt/unbound/
```

### Using Portainer stacks?

Portainer stacks are a little weird and don't want you to declare your named volumes, so remove this block from the top of the `docker-compose.yaml` file before copy/pasting into Portainer's stack editor:

```yaml
volumes:
  etc_pihole-unbound:
  etc_pihole_dnsmasq-unbound:
```

### Running the stack

```bash
docker-compose up -d
```

> If using Portainer, just paste the `docker-compose.yaml` contents into the stack config and add your *environment variables* directly in the UI.

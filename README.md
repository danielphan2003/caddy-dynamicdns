Dynamic DNS app for Caddy
=========================

This is a simple Caddy app that keeps your DNS pointed to your machine; especially useful if your IP address is not static.

It simply queries a service (an "IP source") for your public IP address every so often and if it changes, it updates the DNS records with your configured provider. It supports multiple IPs, including IPv4 and IPv6, as well as redundant IP sources.

IP sources and DNS providers are modular. This app comes with IP source modules. However, you'll need to plug in [a DNS provider module from caddy-dns](https://github.com/caddy-dns) so that your DNS records can be updated.


### Minimal example config

Caddyfile config ([global options](https://caddyserver.com/docs/caddyfile/options)):

```
{
	dynamic_dns {
		provider cloudflare {env.CLOUDFLARE_API_TOKEN}
		domains {
			example.com
		}
	}
}
```

Equivalent JSON config:

```json
{
	"apps": {
		"dynamic_dns": {
			"domains": {
				"example.com": ["@"]
			},
			"dns_provider": {
				"name": "cloudflare",
				"api_token": "{env.CLOUDFLARE_API_TOKEN}"
			}
		}
	}
}
```

This updates DNS records for `example.com` via Cloudflare's API. (Notice how the DNS zone is separate from record names/subdomains.)


### Complex example config

Here's a more filled-out config, will all the options used.

This config prefers to get the IP address locally via UPnP (if edge router has UPnP enabled, of course), but if that fails, will fall back to querying `icanhazip.com` for the IP address. It then updates records for `example.com`, `www.example.com`, and `subdomain.example.net`. Notice how the zones and subdomains are separate; this eliminates ambiguity since we don't have to try to be clever and figure out the zone via recursive, authoritative DNS lookups. We also check every 5 minutes instead of 30 minutes (default).

Note that it's redundant to specify both IP versions in the config, since the default is to enable both IPv4 and IPv6. It's purpose is to allow disabling one or the other if your server is only reachable via one of the versions. It's included in this config example for posterity.

Caddyfile config ([global options](https://caddyserver.com/docs/caddyfile/options)):

```
{
	dynamic_dns {
		provider cloudflare {env.CLOUDFLARE_API_TOKEN}
		domains {
			example.com @ www
			example.net subdomain
		}
		ip_source upnp
		ip_source simple_http https://icanhazip.com
		ip_source simple_http https://api64.ipify.org
		check_interval 5m
		versions ipv4 ipv6
	}
}
```

Equivalent JSON config:

```json
{
	"apps": {
		"dynamic_dns": {
			"dns_provider": {
				"name": "cloudflare",
				"api_token": "{env.CLOUDFLARE_API_TOKEN}"
			},
			"domains": {
				"example.com": ["@", "www"],
				"example.net": ["subdomain"]
			},
			"ip_sources": [
				{
					"source": "upnp"
				},
				{
					"source": "simple_http",
					"endpoints": ["https://icanhazip.com", "https://api64.ipify.org"]
				}
			],
			"check_interval": "5m",
			"versions": ["ipv4", "ipv6"]
		}
	}
}
```


### Disabling IPv6

To disable IPv6 lookups, specify only IPv4 as the version you want enabled.

Caddyfile config:

```
{
	dynamic_dns {
		provider cloudflare {env.CLOUDFLARE_API_TOKEN}
		domains {
			example.com
		}
		versions ipv4
	}
}
```

Equivalent JSON config:

```json
{
	"apps": {
		"dynamic_dns": {
			"domains": {
				"example.com": ["@"]
			},
			"dns_provider": {
				"name": "cloudflare",
				"api_token": "{env.CLOUDFLARE_API_TOKEN}"
			},
			"versions": ["ipv4"]
		}
	}
}
```
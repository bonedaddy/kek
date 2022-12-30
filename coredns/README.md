# CoreDNS

To install with the unbound recursive resolver we need to install the `libunbound-dev` dependency.

# Reverse Lookups

https://github.com/coredns/coredns/issues/2977

# Docker


The actual dockerfile used:

```Dockerfile
FROM debian:stable-slim

RUN apt-get update && apt-get -uy upgrade
RUN apt-get -y install ca-certificates && update-ca-certificates

FROM scratch

COPY --from=0 /etc/ssl/certs /etc/ssl/certs
ADD coredns /coredns

EXPOSE 53 53/udp
ENTRYPOINT ["/coredns"]
```

# Overview


```
Specifying a Protocol
Currently CoreDNS accepts four different protocols: DNS, DNS over TLS (DoT), DNS over HTTP/2 (DoH) and DNS over gRPC. You can specify what a server should accept in the server configuration by prefixing a zone name with a scheme.

dns:// for plain DNS (the default if no scheme is specified).
tls:// for DNS over TLS, see RFC 7858.
https:// for DNS over HTTPS, see RFC 8484.
grpc:// for DNS over gRPC.
```

## Recursive Resolvers

CoreDNS does not have a native (i.e. written in Go) recursive resolver, but there is an (external) plugin that utilizes libunbound. For this setup to work, you first have to recompile CoreDNS and enable the unbound plugin. Super quick primer here (you must have the CoreDNS source installed):

Add unbound:github.com/coredns/unbound to plugin.cfg.
Do a go generate, followed by make.
Note: the unbound plugin needs cgo to be compiled, which also means the coredns binary is now linked against libunbound and not a static binary anymore.

Assuming this worked, you can then enable unbound with the following Corefile:

```
. {
    unbound
    cache
    log
}
```

cache has been included, because the (internal) cache from unbound is disabled to allow the cache’s metrics to works just like normal.

# DNS Records

## SOA

```
ns1.dnsimple.com admin.dnsimple.com 2013022001 86400 7200 604800 300
```

* The primary name server for the domain, which is ns1.dnsimple.com or the first name server in the vanity name server list.
* The responsible party for the domain: admin.dnsimple.com.
* A timestamp that changes whenever you update your domain.
* The number of seconds before the zone should be refreshed.
* The number of seconds before a failed refresh should be retried.
* The upper limit in seconds before a zone is considered no longer authoritative.
* The negative result TTL (for example, how long a resolver should consider a negative result for a subdomain to be valid before retrying).

## Zone File


Format defined in the following RFCs:

* RFC-1035
* RFC-2308

Links:

* http://www-lor.int-evry.fr/cgi-ph/rfc?1035&fmt=txt
* https://www.zytrax.com/books/dns/ch6/mydomain.html
* https://help.dyn.com/how-to-format-a-zone-file/
* https://www.zytrax.com/books/dns/ch6/reverse-map.html

## Examples

```
$ORIGIN dns...... ; this symbol starts a comment
$TTL 3600 ; this sets the default ttl
@	IN	SOA sns.dns.icann.org. noc.dns.icann.org. (
				2020032102 ; serial -  RFC-1912 format
				7200       ; refresh (2 hours)
				3600       ; retry (1 hour)
				1209600    ; expire (2 weeks)
				3600       ; minimum (1 hour)
				)

	3600 IN NS a.iana-servers.net.
	3600 IN NS b.iana-servers.net.

; start records

www     	IN A		127.0.0.1
lighthouse1 IN A		....
lighthouse2 IN A 		....
test    	IN CNAME 	www
```

```
dig @localhost -p 5356 ns1.dns.....
```

# Configuration

## systemd-resolved setups

If your setup uses systemd-resolved, it can be a bit troubling to run a DNS server, so you need to make sure that you properly override resolved to point to the correct host.

You'll need to `rm /etc/resolv.conf` and create a new file containing the appropriate IP for the nameserver you want to use. Then you'll need to do `chattr +i /etc/resolv.conf` to prevent anyone from being able to modify the contents
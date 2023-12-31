# Configuration DNS on Ubuntu 22.04

https://www.ionos.com/digitalguide/server/configuration/change-dns-server-on-ubuntu/#:~:text=Step%201%3A%20Launch%20the%20system,server%20connection%20for%20a%20moment.


# Dynamic bind updates

https://stackoverflow.com/questions/76623550/permissions-error-when-creating-an-a-record


# Update DNS with Terraform Provider
```shell
echo /etc/bind/zones/** rw, > /etc/apparmor.d/local/usr.sbin.named
chown bind:bind -R /etc/bind
setcap 'cap_net_bind_service=+ep' /usr/sbin/named
```
https://unix.stackexchange.com/questions/710321/bind9-dynamic-zone-updates-are-denied-by-apparmor-in-debian11

Also https://dnns.no/dynamic-dns-with-bind-and-nsupdate.html

/etc/apparmor.d/user.sbin.named
```text
...
  /etc/bind/** r,
  /etc/bind/pz/* rw,
  /var/lib/bind/** rw,
  /var/lib/bind/ rw,
  /var/cache/bind/** lrw,
  /var/cache/bind/ rw,
```

named.conf
```text
include "/etc/bind/named.conf.key";
include "/etc/bind/named.conf.options";
include "/etc/bind/named.conf.local";
include "/etc/bind/named.conf.default-zones";
```

named.conf.key
```text
key "tsig-key" {
	algorithm hmac-sha256;
	secret "********8";
};
```

named.conf.local
```text
include "/etc/bind/zones.rfc1918";
zone "byteworksinc.com" {
    type master;
    file "/etc/bind/pz/db.byteworksinc.com";
    allow-transfer { 10.0.0.8; };
    also-notify { 10.0.0.8; };
    update-policy {
       grant tsig-key zonesub any;
    };
};
```

named.conf.options
```text
options {
	directory "/var/cache/bind";

	forwarders {
	 	8.8.8.8;
		1.1.1.1;
	};

	dnssec-validation auto;
	listen-on-v6 { any; };
};
```

# DNS server configuration

## Modules required:

- bind9
- bind9utils
- bind9-doc

## File /etc/bind/named.conf.options

This file contains all the configurations included the ip addresses allowed to perform the query.  
In this file are also specified the security measures,
in this server they are not set up, in order to ease the msdos attack.

```bash
//options {
//	directory "/var/cache/bind";

	// If there is a firewall between you and nameservers you want
	// to talk to, you may need to fix the firewall to allow multiple
	// ports to talk.  See http://www.kb.cert.org/vuls/id/800113

	// If your ISP provided one or more IP addresses for stable
	// nameservers, you probably want to use them as forwarders.
	// Uncomment the following block, and insert the addresses replacing
	// the all-0's placeholder.

	// forwarders {
	// 	0.0.0.0;
	// };

	//========================================================================
	// If BIND logs error messages about the root key being expired,
	// you will need to update your keys.  See https://www.isc.org/bind-keys
	//========================================================================
//	dnssec-validation auto;

//	listen-on-v6 { any; };
// };
// This file contains all the configuration options for the DNS server.

// If there is a firewall between you and nameservers you want
// to talk to, you may need to fix the firewall to allow multiple
// ports to talk.  See http://www.kb.cert.org/vuls/id/800113

// If your ISP provided one or more IP addresses for stable
// nameservers, you probably want to use them as forwarders.
// Uncomment the following block, and insert the addresses replacing
// the all-0's placeholder.

//========================================================================
// If BIND logs error messages about the root key being expired,
// you will need to update your keys.  See https://www.isc.org/bind-keys
//========================================================================

//************************************************************************************
// ACL's define a address_match_list e.g. IP address(es), which can then be referenced
// (used) in a number of statements and the view clause(s). acl's MUST be defined before
// they are referenced in any statement or clause. For this reason they are usually defined
// first in the named.conf file. 'acl-name' is an arbitrary (but unique) quoted string defining
// the specific list. The 'acl-name' is the method used to subsequently reference the particular
// list. Any number of acl's may be defined.

/*
 * Deny transfers by default except for the listed hosts.
 * If we have other name servers, place them here.
 */
 acl "acl_trusted_transfer" {
       127.0.0.0/8;        // localhost (RFC 3330) - Loopback-Device addresses    127.0.0.0 - 127.255.255.255
       192.168.0.0/16;     // Private Network (RFC 1918) - e. e. LAN              192.168.0.0 - 192.168.255.255
       10.0.0.0/8;         // Private Network (RFC 1918) - e. g. VPN              10.0.0.0 - 10.255.255.255
 };

/*
 * You might put in here some ips which are allowed to use the cache or
 * recursive queries
 */
 acl "acl_trusted_clients" {
       127.0.0.0/8;        // localhost (RFC 3330) - Loopback-Device addresses    127.0.0.0 - 127.255.255.255
       192.168.0.0/16;     // Private Network (RFC 1918) - e. e. LAN              192.168.0.0 - 192.168.255.255
       10.0.0.0/8;         // Private Network (RFC 1918) - e. g. VPN              10.0.0.0 - 10.255.255.255
 };

//********************************************************************************
options {
        /*
         * Is a quoted string defining the absolute path for the server e.g. "/var/named".
         * All subsequent relative paths use this base directory. If no directory options
         * is specified the directory from which BIND was loaded is used.
         */
        directory "/var/cache/bind";

        /*
         * Is a quoted string and allows you to define where the pid (Process Identifier)
         * used by BIND is written. If not present it is distribution or OS specific
         * typically /var/run/named.pid or /etc/named.pid. It may be defined using an
         * absolute path or relative to the directory parameter.
         */
        pid-file "/var/run/named/named.pid";

        /*
         * Specifies the string that will be returned to a version.bind query when using
         * the chaos class only. version_string is a quoted string, for example, "get lost"
         * or something equally to the point. We tend to use it in all named.conf files to
         * avoid giving out a version number such that an attacker can exploit known
         * version-specific weaknesses.
         */
        version "not currently available";

        /*
         * Turns on BIND to listen for IPv6 queries. If this statement is not present and the
         * server supports IPv6 (only or in dual stack mode) the server will listen for IPv6 on
         * port 53 on all server interfaces. If the OS supports RFC 3493 and RFC 3542 compliant
         * IPv6 sockets and the address_match_list uses the special any name then a single listen
         * is issued to the wildcard address. If the OS does not support this feature a socket is
         * opened for every required address and port. The port default is 53.
         * Multiple listen-on-v6 statements are allowed.
         */
        listen-on-v6 { none; };

        /* Defines the port and IP address(es) on which BIND will listen for incoming queries.
         * The default is port 53 on all server interfaces.
         * Multiple listen-on statements are allowed.
         */
        listen-on { any; };

        /* Notify behaviour is applicable to both master zones (with 'type master;')
         * and slave zones (with 'type slave;') and if set to 'yes' (the default) then,
         * when a zone is loaded or changed, for example, after a zone transfer, NOTIFY
         * messages are sent to the name servers defined in the NS records for the zone
         * (except itself and the 'Primary Master' name server defined in the SOA record)
         * and to any IPs listed in any also-notify statement.
         * If set to 'no' NOTIFY messages are not sent.
         * If set to 'explicit' NOTIFY is only sent to those IP(s) listed in an also-notify statement.
         */
        notify no;

        /*
         * Defines an match list of IP address(es) which are allowed
         * to issue queries to the server.
         * Only trusted addresses are allowed to perform queries.
         * We will allow anyone to query our master zones below.
         * This prevent becoming a public free DNS server.
         */
        allow-query {
                acl_trusted_clients;
        };

        /*
         * Defines an match list of IP address(es) which are allowed to
         * issue queries that access the local query cache.
         * Only trusted addresses are allowed to use query cache.
         */
        allow-query-cache {
                acl_trusted_clients;
        };

        /*
         * Defines a match list of IP address(es) which are allowed to
         * issue recursive queries to the server.
         * Only trusted addresses are allowed to use recursion.
         */
         allow-recursion {
                acl_trusted_clients;
         };

        /*
         * Dfines a match list e.g. IP address(es) that are allowed to transfer
         * the zone information from the server (master or slave for the zone).
         * The default behaviour is to allow zone transfers to any host.
         */
        allow-transfer {
		acl_trusted_clients;
        };

        /*
         * Defines an match list of host IP address(es) that are allowed
         * to submit dynamic updates for master zones, and thus this
         * statement enables Dynamic DNS.
         */
        allow-update {
                acl_trusted_clients;
        };

        /*
         * Indicates that a resolver (a caching or caching-only name server) will attempt
         * to validate replies from DNSSEC enabled (signed) zones. To perform this task
         * the server alos needs either a valid trusted-keys clause (containing one or more
         * trusted-anchors or a managed-keys clause.
         * Since 9.5 the default value is dnssec-validation yes;
         */
        dnssec-validation no;

        /*
         * If auth-nxdomain is 'yes' allows the server to answer authoritatively
         * (the AA bit is set) when returning NXDOMAIN (domain does not exist) answers,
         * if 'no' (the default) the server will not answer authoritatively.
         */
        auth-nxdomain no; # conform to RFC1035

        /*
         * By default empty-zones-enable is set to 'yes' which means that
         * reverse queries for IPv4 and IPv6 addresses covered by RFCs 1918,
         * 4193, 5737 and 6598 (as well as IPv6 local address (locally assigned),
         * IPv6 link local addresses, the IPv6 loopback address and the IPv6 unknown address)
         * but which is not not covered by a locally defined zone clause will automatically
         * return an NXDOMAIN response from the local name server. This prevents reverse map queries
         * to such addresses escaping to the DNS hierarchy where they are simply noise and increase
         * the already high level of query pollution caused by mis-configuration.
         */
        empty-zones-enable yes;

        /*
         * If recursion is set to 'yes' (the default) the server will always provide
         * recursive query behaviour if requested by the client (resolver).
         * If set to 'no' the server will only provide iterative query behaviour -
         * normally resulting in a referral. If the answer to the query already
         * exists in the cache it will be returned irrespective of the value of this statement.
         * This statement essentially controls caching behaviour in the server.
         */
        recursion yes;

        /*
         * additional-from-auth and additional-from-cache control the behaviour when
         * zones have additional (out-of-zone) data or when following CNAME or DNAME records.
         * These options are for used for configuring authoritative-only (non-caching) servers
         * and are only effective if recursion no is specified in a global options clause or
         * in a view clause. The default in both cases is yes.
         */
        //additional-from-auth no;
        //additional-from-cache no;

        /*
         * Defines a list of IP address(es) and optional port numbers
         * to which queries will be forwarded.
         */
        forwarders {
                // Router DNS
                // 192.168.2.1

                // Google Public DNS
                8.8.8.8;
                8.8.4.4;

                // OpenDNS
                208.67.222.222;
                208.67.220.220;
        };

};

```

## File /etc/bind/named.conf.local

This file is typically used to define local DNS zones for a private domain. We will update this file to include our forward and reverse DNS zones.  
During the test it is configured using private NS.

```bash
 GNU nano 6.2   /etc/bind/named.conf.local
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

zone "ediproject.com" {
    type primary;
    file "/etc/bind/zones/db.ediproject.com"; # zone file path
};

zone "128.10.in-addr.arpa" {
    type primary;
    file "/etc/bind/zones/db.10.128";  # 10.128.0.0/16 subnet
};
```

## File /etc/bind/zones/db.ediproject.com

/etc/bind/zones/db.ediproject.com
It contains all informations about the domain name, including all resource records ...
This domain has :

- 7 Name servers
- 6 Mail servers
- 2 IPv4 addresses for the domain ediproject.com.

```bash
  GNU nano 6.2    /etc/bind/zones/db.ediproject.com
$TTL    604800
@       IN      SOA     ns1.ediproject.com. admin.ediproject.com. (
                  3     ; Serial
             604800     ; Refresh
              86400     ; Retry
            2419200     ; Expire
             604800 )   ; Negative Cache TTL
;
; name servers - NS records
@     IN      NS        davide.ediproject.com.
@     IN      NS        cristian.ediproject.com.
@     IN      NS        matteo.ediproject.com.
@     IN      NS        andrea.ediproject.com.
@     IN      NS        karim.ediproject.com.
@     IN      NS        www.ediproject.com.
@     IN      NS        ns2.ediproject.com.
@     IN      MX  10    ediproject.com
@     IN      MX  8     email.ediproject.com
@     IN      MX  6     email2.ediproject.com
@     IN      MX  3     email3.ediproject.com
@     IN      MX  2     email4.ediproject.com
@     IN      MX  1     email5.ediproject.com

ns1.ediproject.com.          IN      A       10.128.10.11
davide.ediproject.com.       IN      A       10.128.10.13
karim.ediproject.com.        IN      A       10.128.10.12
andrea.ediproject.com.       IN      A       10.128.10.14
matteo.ediproject.com.       IN      A       10.128.10.15
cristian.ediproject.com.     IN      A       10.128.10.16
ediproject.com.              IN      A       10.128.10.17
ediproject.com.              IN      A       10.128.10.18
www.ediproject.com.          IN      A       10.128.10.19
ns2.ediproject.com.          IN      A       10.128.10.20
```

## File /etc/bind/zones/db.10.128

It contains all informations about the reverse name server

```bash
  GNU nano 6.2    /etc/bind/zones/db.10.128
$TTL    604800
@       IN      SOA     ediproject.com. admin.ediproject.com. (
                              3         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
; name servers
      IN      NS      ns1.project.com.


```

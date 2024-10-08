# freebsd_dns

[![quality](https://img.shields.io/ansible/quality/27910)](https://galaxy.ansible.com/vbotka/freebsd_dns)
[![Build Status](https://app.travis-ci.com/vbotka/ansible-freebsd-dns.svg?branch=master)](https://app.travis-ci.com/vbotka/ansible-freebsd-dns)
[![GitHub tag](https://img.shields.io/github/v/tag/vbotka/ansible-freebsd-dns)](https://github.com/vbotka/ansible-freebsd-dns/tags)

[Ansible role.](https://galaxy.ansible.com/vbotka/freebsd_dns/) FreeBSD. Install and configure DNS.

Feel free to [share your feedback and report issues](https://github.com/vbotka/ansible-freebsd-dns/issues).

[Contributions are welcome](https://github.com/firstcontributions/first-contributions).


## Requirements

### Collections

* community.general

### Recommended

* [YAZVS (Yet Another Zone Validation Script)](https://galaxy.ansible.com/vbotka/yazvs/)


## Role Variables

See the defaults and examples in vars.

* By default the service *named* and *dnssec* are disabled.

```yaml
bsd_named_enable: false
bsd_named_conf_dnssec_enable: false
```

* Keys are needed to enable DNSSEC (see the workflow below).

* *dnssec-keygen* binary is needed to generate the keys.

* See samples in *vars*


### Recreate named.conf

Set 'bsd_named_conf_recreate=true' to either:

* copy *named.conf.sample* to *named.conf* when 'bsd_named_conf_recreate_from_sample=true' (default), or

* create *named.conf* from a template when 'bsd_named_conf_recreate_from_sample=false' (default 'bsd_named_conf_recreate_template=named-forward-only.j2')

Set 'bsd_named_conf_recreate=false' (default) to make the tasks *named.yml* idempotent.


## Workflow

1) Change shell to /bin/sh if necessary

```bash
shell>  ansible host -e 'ansible_shell_type=csh ansible_shell_executable=/bin/csh' -a 'sudo pw usermod admin -s /bin/sh'
```

2) Install the role

```bash
shell> ansible-galaxy role install vbotka.freebsd_dns
```

3) Change variables

4) Create the playbook and check the syntax

```bash
shell> cat freebsd-dns.yml
- hosts: ns1.example.com
  roles:
    - vbotka.freebsd_dns
    
shell> ansible-playbook freebsd-dns.yml --syntax-check
```

5) If DNSSEC is enabled create keys as described in <TODO>. See [Domain Name System (DNS)](https://docs.freebsd.org/en/books/handbook/network-servers/#network-dns).
Create the directories first if necessary. See how to run the playbook below.

Example:

```bash
shell> cd /usr/local/etc/namedb/keys
shell> dnssec-keygen -f KSK -a RSASHA256 -b 2048 -n ZONE example.com
shell> ln -s Kexample.com.+008+20191.key Kexample.com.KSK.key
shell> ln -s Kexample.com.+008+20191.private Kexample.com.KSK.private
shell> dnssec-keygen -a RSASHA256 -b 2048 -n ZONE example.com
shell> ln -s Kexample.com.+008+35529.key Kexample.com.ZSK.key
shell> ln -s Kexample.com.+008+35529.private Kexample.com.ZSK.private
shell> chown bind K*
```

6) Configure the zones

Example of master zone

```yaml
sd_named_conf_zone:
 - zone: example.com
   type: master
   reverse: true
   zone_ip: 10.1.0.10
   zone_in: 0.1.10
   primary: ns1.example.net
   primary_ip: 192.168.1.11
   secondary: ns2.example.net
   secondary_ip: 192.168.1.12
   admin: admin.example.com
   serial: '2016102401'
   refresh: '10800'
   retry: '3600'
   expire: '1209600'
   negative: '300'
   server:
     - ns1.example.net
     - ns2.example.net
   mx:
     - {server: srv.example.com, priority: '10'}
   host:
     - {host: srv, ip: '10.1.0.10'}
   alias:
     - www
     - mail
```

Example of slave zone

```yaml
sd_named_conf_zone:
 - zone: example.com
   type: slave
   masters: 192.168.1.11;
   reverse: true
   zone_in: 0.1.10
```

7) Run the playbook

Test the syntax

```yaml
shell> ansible-playbook freebsd-dns.yml --syntax-check
```

Take a look at the variables

```yaml
shell> ansible-playbook freebsd-dns.yml -t bsd_dns_debug -e bsd_dns_debug=true
```

Install packages

```yaml
shell> ansible-playbook freebsd-dns.yml -t bsd_dns_packages -e bsd_dns_install=true
```
Create directories

```yaml
shell> ansible-playbook freebsd-dns.yml -t bsd_dns_dirs
```

Dry run and see what will be configured

```yaml
shell> ansible-playbook freebsd-dns.yml --check --diff
```

If this is what you want run the play

```yaml
shell> ansible-playbook freebsd-dns.yml
```

8) Sign the zones, reload the server and test the server as described in <TODO>. The zones can be signed when the DNSSEC keys are included in the zone files.

Sign the zone. Change to the *keys* directory. Otherwise, full path to the keys is needed.

```bash
shell> cd /usr/local/etc/namedb/keys
shell> dnssec-signzone -o example.com -k Kexample.com.KSK /usr/local/etc/namedb/master/example.com Kexample.com.ZSK.key
shell> /usr/local/etc/rc.d/named reload
```

Test the server.

```bash
shell> dig @resolver +dnssec se ds 
```

9) Update registrar DS records

- [Add a DS record](https://uk.godaddy.com/help/add-a-ds-record-23865)

```bash
shell> dig type48 example.com
```

- [Domain Name System Security (DNSSEC) Algorithm Numbers](http://www.iana.org/assignments/dns-sec-alg-numbers/dns-sec-alg-numbers.xhtml)
- [DS records at Godaddy with .co tld](https://lists.opendnssec.org/pipermail/opendnssec-user/2015-April/003288.html)
- [How To Setup DNSSEC on an Authoritative BIND DNS Server](https://www.digitalocean.com/community/tutorials/how-to-setup-dnssec-on-an-authoritative-bind-dns-server--2)

10) Consider to test the server with

- [dnscheck.iis.se](http://dnscheck.iis.se/)
- [DNS Check in Pingdom Tools](http://dnscheck.pingdom.com/)
- [Pingdom. Failed to deliver email. Can safely ignore this failure.](http://serverfault.com/questions/748923/how-can-i-fix-these-soa-dns-problems)
- [Verisign LABS](http://dnssec-debugger.verisignlabs.com/)
- [DNS VIZ](http://dnsviz.net/)


## NOTES

See: [isc.org DNSSEC search results](https://www.isc.org/search/?s=dnssec)


## Ansible lint

Use the configuration file *.ansible-lint.local* when running
*ansible-lint*. Some rules might be disabled and some warnings might
be ignored. See the notes in the configuration file.

```bash
shell> ansible-lint -c .ansible-lint.local
```


## TODO

- automate creation of the keys
- automate signing of the zones
- automate testing of the server


## References

- [BIND Release Notes](https://kb.isc.org/docs/a-note-about-bind-release-notes)
- [USENIX: DNSSEC in 6 minutes](http://static.usenix.org/event/lisa08/dnssec_bof.pdf)
- [BSD: Domain Name System (DNS)](https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/network-dns.html)
- [ISC: BIND 9 Administrator Reference Manual](https://ftp.isc.org/isc/bind9/cur/9.19/doc/arm/html/)
- [Zytrax: DNS for Rocket Scientists](http://www.zytrax.com/books/dns/)
- [Zytrax: BIND (Berkeley Internet Name Domain)](http://www.zytrax.com/books/dns/ch5/)
- [Zytrax: DNS BIND Zone Transfers and Updates](http://www.zytrax.com/books/dns/ch7/xfer.html)
- [Zytrax: Define a DMARC Record](http://www.zytrax.com/books/dns/ch9/dmarc.html)
- [Zytrax: Define an SPF Record](http://www.zytrax.com/books/dns/ch9/spf.html)
- [OpenSPF: Sender Policy Framework](http://www.open-spf.org/)

- [IBM: Configuring a forward only name server](https://www.ibm.com/docs/en/aix/7.1?topic=resolution-configuring-forward-only-name-server)
- [Invalid TLD error when changing nameservers in godaddy](https://www.howtoforge.com/community/threads/invalid-tld-error-when-changing-nameservers-in-godaddy.62932/)
- [www.dnssec.net](http://www.dnssec.net/)
- [FreeBSD man named-checkconf(8)](https://www.freebsd.org/cgi/man.cgi?query=named-checkconf&sektion=8)
- [FreeBSD man named-checkzone(8)](https://www.freebsd.org/cgi/man.cgi?query=named-checkzone&sektion=8)


## License

[![license](https://img.shields.io/badge/license-BSD-red.svg)](https://www.freebsd.org/doc/en/articles/bsdl-gpl/article.html)


## Author Information

[Vladimir Botka](https://botka.info)

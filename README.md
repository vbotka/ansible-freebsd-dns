# freebsd_dns

[![quality](https://img.shields.io/ansible/quality/27910)](https://galaxy.ansible.com/vbotka/freebsd_dns)[![Build Status](https://travis-ci.org/vbotka/ansible-freebsd-dns.svg?branch=master)](https://travis-ci.org/vbotka/ansible-freebsd-dns)

[Ansible role.](https://galaxy.ansible.com/vbotka/freebsd_dns/) FreeBSD. Configure DNS.

Feel free to [share your feedback and report issues](https://github.com/vbotka/ansible-freebsd-dns/issues).

[Contributions are welcome](https://github.com/firstcontributions/first-contributions).


## Requirements

### Collections

* community.general

### Recommended

* [YAZVS (Yet Another Zone Validation Script)](https://galaxy.ansible.com/vbotka/yazvs/)


## Role Variables

See the defaults and examples in vars.

By default the service *named* and *dnssec* are disabled.


```yaml
bsd_named_enable: false
bsd_named_conf_dnssec_enable: 'no'
bsd_named_conf_dnssec_validation: 'no'
```

- Keys are needed to enable DNSSEC (see workflow).
- *dnssec-keygen* binary is needed to generate the keys.


## Workflow

1) Change shell to /bin/sh

```bash
shell>  ansible host -e 'ansible_shell_type=csh ansible_shell_executable=/bin/csh' -a 'sudo pw usermod admin -s /bin/sh'
```

2) Install role

```bash
shell> ansible-galaxy install vbotka.freebsd_dns
```

3) Change variables

```bash
shell> editor vbotka.freebsd_dns/vars/main.yml
```

4) Create and run the playbook

```bash
shell> cat freebsd-dns.yml
- hosts: ns1.example.com
  roles:
    - vbotka.freebsd_dns
    
shell> ansible-playbook freebsd-dns.yml
```

5) If DNSSEC is enabled create keys as described in [Authoritative DNS Server Configuration](http://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/network-dns.html#dns-dnssec-auth)

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
   reverse: "yes"
   zone_ip: 10.1.0.10
   zone_in: 0.1.10
   primary: ns1.example.net
   primary_ip: 192.168.1.11
   secondary: ns2.example.net
   secondary_ip: 192.168.1.12
   admin: admin.example.com
   serial: 2016102401
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
   reverse: "yes"
   zone_in: 0.1.10
```

7) Run the playbook

```yaml
shell> ansible-playbook freebsd-dns.yml
```

8) Sign the zones, reload the server and test the server as described in [Authoritative DNS Server Configuration](http://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/network-dns.html#dns-dnssec-auth). The zones can be signed when the DNSSEC keys are included in the zone files.

Sign the zone. Change to the *keys* directory. Otherwise full path to the keys is needed.

```bash
shell> cd /usr/local/etc/namedb/keys
shell> dnssec-signzone -o example.com -k Kexample.com.KSK /usr/local/etc/namedb/master/example.com Kexample.com.ZSK.key
shell> /usr/local/etc/rc.d/named reload
```

Test the server.

```
shell> dig @resolver +dnssec se ds 
```

9) Update registrar DS records

- [Add a DS record](https://uk.godaddy.com/help/add-a-ds-record-23865)
- ["Method used for encrypting the public key"](https://www.edge-cloud.net/2014/06/16/practical-guide-dns-based-authentication-named-entities-dane/) can be found using the command

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
- [Veisign LABS](http://dnssec-debugger.verisignlabs.com/)
- [DNS VIZ](http://dnsviz.net/)


## NOTES

- [In-line Signing](https://deepthought.isc.org/article/AA-00711/0/In-line-Signing-With-NSEC3-in-BIND-9.9-A-Walk-through.html)
  works with the slave as expected, but not with the master.
- Keys from master are copied to the slave manually.


## TODO

- automate creation of the keys
- automate signing of the zones
- automate testing of the server


## References

- [BIND 9.11.3 Release Notes](https://kb.isc.org/article/AA-01597/0/BIND-9.11.3-Release-Notes.html)
- [USENIX: DNSSEC in 6 minutes](http://static.usenix.org/event/lisa08/dnssec_bof.pdf)
- [BSD: Domain Name System (DNS)](https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/network-dns.html)
- [BSD: Authoritative DNS Server Configuration](http://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/network-dns.html#dns-dnssec-auth)
- [ISC: BIND 9 Configuration Reference](https://ftp.isc.org/isc/bind9/cur/9.10/doc/arm/Bv9ARM.ch06.html)
- [ISC: Inline Signing in ISC BIND 9.9.0](https://kb.isc.org/article/AA-00626/0/Inline-Signing-in-ISC-BIND-9.9.0-Examples.html)
- [ISC: BIND DNSSEC Guide](https://users.isc.org/~jreed/dnssec-guide/dnssec-guide.html)
- [Zytrax: DNS for Rocket Scientists](http://www.zytrax.com/books/dns/)
- [Zytrax: BIND (Berkeley Internet Name Domain)](http://www.zytrax.com/books/dns/ch5/)
- [Zytrax: DNS BIND Zone Transfers and Updates](http://www.zytrax.com/books/dns/ch7/xfer.html)
- [Zytrax: Define a DMARC Record](http://www.zytrax.com/books/dns/ch9/dmarc.html)
- [Zytrax: Define an SPF Record](http://www.zytrax.com/books/dns/ch9/spf.html)
- [OpenSPF: How a receiving mail server uses SPF](http://www.openspf.org/FAQ/Examples)

- [Enabling DNSSec in Bind](http://networking.ringofsaturn.com/Unix/dnssec.php)
- [Invalid TLD error when changing nameservers in godaddy](https://www.howtoforge.com/community/threads/invalid-tld-error-when-changing-nameservers-in-godaddy.62932/)
- [www.bind9.net](http://www.bind9.net/)
- [www.dnssec.net](http://www.dnssec.net/)
- [FreeBSD man named-checkconf(8)](https://www.freebsd.org/cgi/man.cgi?query=named-checkconf&sektion=8)
- [FreeBSD man named-checkzone(8)](https://www.freebsd.org/cgi/man.cgi?query=named-checkzone&sektion=8)


## License

[![license](https://img.shields.io/badge/license-BSD-red.svg)](https://www.freebsd.org/doc/en/articles/bsdl-gpl/article.html)


## Author Information

[Vladimir Botka](https://botka.info)

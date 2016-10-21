freebsd-dns
===========

[![Build Status](https://travis-ci.org/vbotka/ansible-freebsd-dns.svg?branch=master)](https://travis-ci.org/vbotka/ansible-freebsd-dns)

[Ansible role.](https://galaxy.ansible.com/vbotka/freebsd-dns/) Configure DNS in FreeBSD.


Requirements
------------

No requiremenst.


Variables
---------

TBD (Check the defaults).


Workflow
--------

1) Change shell to /bin/sh.

```
ansible do-bsd-test -e 'ansible_shell_type=csh ansible_shell_executable=/bin/csh' -a 'sudo pw usermod freebsd -s /bin/sh'
```

2) Install role.

```
ansible-galaxy install vbotka.freebsd-dns
```

3) Fit variables. Zones can be configured when DNSSEC keys are available. *dnssec-keygen* binary is needed to generate the keys.

```
~/.ansible/roles/vbotka.freebsd-dns/vars/main.yml
```

4) Create and run the playbook.

```
> cat ~/.ansible/playbooks/freebsd-dns.yml
---
- hosts: do-bsd-tetst
  become: yes
  become_method: sudo
  roles:
    - role: vbotka.ansible-freebsd-dns
    
> ansible-playbook ~/.ansible/playbooks/freebsd-dns.yml
```

5) Create keys as described in [Authoritative DNS Server Configuration](http://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/network-dns.html#dns-dnssec-auth) and configure the zones

Example:

```
> cd /usr/local/etc/namedb/keys
> dnssec-keygen -f KSK -a RSASHA256 -b 2048 -n ZONE example.com
> ln -s Kexample.com.+008+20191.key Kexample.com.KSK.key
> ln -s Knetng.co.+008+20191.private Knetng.co.KSK.private
> dnssec-keygen -a RSASHA256 -b 2048 -n ZONE example.com
> ln -s Kexample.com.+008+35529.key Kexample.com.ZSK.key
> ln -s Kexample.com.+008+35529.private Kexample.com.ZSK.private
> chown bind K*
```  

6) Example of bsd_named_conf_zone

```
bsd_named_conf_zone:
  - zone: "example.com"
    reverse: "yes"
    type: "master"
    primary: "ns1.example.com"
    ip: "192.168.1.1"
    in: "1.168.192"
    localhost: "127.0.0.1"
    admin: "admin.example.com"
    serial: "2016101801"
    refresh: "10800"
    retry: "3600"
    expire: "604800"
    negative: "300"
    server: [ "ns1.example.com", "ns2.example.com"]
    mx:
      - { server: "mx1.example.com", priority: "10" }
      - { server: "mx2.example.com", priority: "20" }
    host:
      - { host: "srv", ip: "192.168.1.1", ip24: "1" }
      - { host: "ns1", ip: "192.168.1.2", ip24: "2" }
      - { host: "ms2", ip: "192.168.1.3", ip24: "3" }
      - { host: "mx1", ip: "192.168.1.4", ip24: "4" }
      - { host: "mx2", ip: "192.168.1.5", ip24: "5" }
    alias: [ "www", "nfs", "ftp" ]
    dnssec_KSK_public_key: "Kexample.com.KSK.key"
    dnssec_ZSK_public_key: "Kexample.com.ZSK.key"
```

7) Run the playbook

```
ansible-playbook ~/.ansible/playbooks/freebsd-dns.yml
```

8) Sign the zones, reload the server and test the server as described in [Authoritative DNS Server Configuration](http://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/network-dns.html#dns-dnssec-auth). The zones can be signed when the DNSSEC keys are included in the zone files.

Sign the zone. Change to the *keys* directory. Otherwise full path to the keys is needed.

```
> cd /usr/local/etc/namedb/keys
> dnssec-signzone -o example.com -k Kexample.com.KSK /usr/local/etc/namedb/master/example.com  Kexample.com.ZSK.key
> /usr/local/etc/rc.d/named reload
```

Test the server.

```
> dig @resolver +dnssec se ds 
```

TODO
----
- automate creation of the keys
- automate signing of the zones
- automate testing of the server


References
----------

- [DNSSEC in 6 minutes](http://static.usenix.org/event/lisa08/dnssec_bof.pdf)
- [Domain Name System (DNS)](https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/network-dns.html)
- [Authoritative DNS Server Configuration](http://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/network-dns.html#dns-dnssec-auth)
- [ISC: BIND 9 Configuration Reference](https://ftp.isc.org/isc/bind9/cur/9.10/doc/arm/Bv9ARM.ch06.html)
- [Zytrax: BIND (Berkeley Internet Name Domain)](http://www.zytrax.com/books/dns/ch5/)
- [DNS for Rocket Scientists](http://www.zytrax.com/books/dns/)
- [www.bind9.net](http://www.bind9.net/)
- [www.dnssec.net](http://www.dnssec.net/)
- [Enabling DNSSec in Bind](http://networking.ringofsaturn.com/Unix/dnssec.php)


License
-------

[![license](https://img.shields.io/badge/license-BSD-red.svg)](https://www.freebsd.org/doc/en/articles/bsdl-gpl/article.html)



Author Information
------------------

[Vladimir Botka](https://botka.link)

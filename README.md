freebsd-dns
===========

[![Build Status](https://travis-ci.org/vbotka/ansible-freebsd-dns.svg?branch=master)](https://travis-ci.org/vbotka/ansible-freebsd-dns)

Configure DNS in FreeBSD.


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
ansible-galaxy install vbotka.ansible-freebsd-dns
```

3) Fit variables.

```
~/.ansible/roles/vbotka.ansible-freebsd-dns/vars/main.yml
```

4) Create playbook.

```
> cat ~/.ansible/playbooks/freebsd-dns.yml
---
- hosts: do-bsd-tetst
  become: yes
  become_method: sudo
  roles:
    - role: vbotka.ansible-freebsd-dns
```

5) Configure the zones <TBD>.

```
ansible-playbook ~/.ansible/playbooks/freebsd-dns.yml
```


References
----------

- [Domain Name System (DNS)](https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/network-dns.html)


License
-------

[![license](https://img.shields.io/badge/license-BSD-red.svg)](https://www.freebsd.org/doc/en/articles/bsdl-gpl/article.html)



Author Information
------------------

[Vladimir Botka](https://botka.link)

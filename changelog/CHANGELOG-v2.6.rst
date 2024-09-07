====================================
vbotka.freebsd_dns 2.6 Release Notes
====================================

.. contents:: Topics


2.6.2
=====

Release Summary
---------------
Maintenance update.

Major Changes
-------------

Minor Changes
-------------
* Update tests/test.yml playbook


2.6.1
=====

Release Summary
---------------
Ansible 2.17 maintenance update.

Major Changes
-------------
* Add supported 14.1

Minor Changes
-------------
* Update README.
* Update skip_list in local ansible-lint config.
* Update debug, sanity, zones
* Update ANSIBLE MANAGED BLOCK markers
* Add var bsd_dns_role_version
* Add vars +bsd_named_conf_master, bsd_named_conf_slave, bsd_named_conf_dirs
* Create dirs in loop
* Tags removed: bsd_dns_create_named_chrootdir,
  bsd_dns_create_keydir_chrooted, bsd_dns_create_keydir,
  bsd_dns_create_workingdir


2.6.0
=====

Release Summary
---------------
Ansible 2.16 update. Update to dns/bind918

Major Changes
-------------
* Support FreeBSD 13.3. and 14.0
* Update to dns/bind918

Minor Changes
-------------
* Update ansible lint config.
* Use default rules in local ansible-lint
* Fix Ansible lint.
* Update README.
* Formatting travis.yml
* Update debug.yml formatting

Bugfixes
--------

Breaking Changes / Porting Guide
--------------------------------

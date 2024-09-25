========================================
vbotka.freebsd_wpa_cli 2.6 Release Notes
========================================

.. contents:: Topics
# BEGIN Commits 2.6.1
- Update python 3.11 in .travis.yml
- Format meta/main.yml
- Ansible 2.17 maintenance update including docs update
- Update changelog.
- Add 14.1 supported and tested.
- Rename templates
- Fix templates docs/annotation
- Start devel 2.6.1
# END Commits 2.6.1
# BEGIN Release notes 2.6.1
2.6.1
=====
Release Summary
---------------
Major Changes
-------------
Minor Changes
-------------
- Update python 3.11 in .travis.yml
- Format meta/main.yml
- Ansible 2.17 maintenance update including docs update
- Update changelog.
- Add 14.1 supported and tested.
- Rename templates
- Fix templates docs/annotation
- Start devel 2.6.1

Bugfixes
--------
Breaking Changes / Porting Guide
--------------------------------
# END Release notes 2.6.1


2.6.1
=====

Release Summary
---------------
Ansible 2.17 maintenance update including docs update.

Major Changes
-------------
* Support 14.1

Minor Changes
-------------
* Bump docs version.
* Update docs.
* Update README.
* Update default template wpacli_action_script_template
* Update debug
* Add var wpacli_versions_tested_rel
* Add var wpacli_role_version

Bugfixes
--------
* Fix templates docs/annotation

Breaking Changes / Porting Guide
--------------------------------
* Rename template 1.0.0-wpa_action.sh.j2 to wpa_action-1.0.0.sh.j2
* Rename template 1.1.0-wpa_action.sh.j2 to wpa_action-1.1.0.sh.j2


2.6.0
=====

Release Summary
---------------
Ansible 2.16 update. Update docs.

Major Changes
-------------
* Supported FreeBSD 13.3 and 14.0

Minor Changes
-------------
* Update Ansible lint config.
* Update README.
* Fix Ansible lint.
* Bump docs 2.6.0
* Update docs.

Bugfixes
--------

Breaking Changes / Porting Guide
--------------------------------

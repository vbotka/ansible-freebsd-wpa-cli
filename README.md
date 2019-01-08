freebsd_wpa_cli
===============

[![Build Status](https://travis-ci.org/vbotka/ansible-freebsd-wpa-cli.svg?branch=master)](https://travis-ci.org/vbotka/ansible-freebsd-wpa-cli)

[Ansible role.](https://galaxy.ansible.com/vbotka/freebsd_wpa_cli/) FreeBSD. Use wpa_cli action_file to configure wlan devices. Configuration of RC system.


Requirements
------------

None.

Recommended
-----------

- [freebsd_postinstall](https://galaxy.ansible.com/vbotka/freebsd_postinstall) task [wpa_supplicant](https://github.com/vbotka/ansible-freebsd-postinstall/blob/master/tasks/wpasupplicant.yml)
- [freebsd_network](https://galaxy.ansible.com/vbotka/freebsd_network)


Role Variables
--------------

Review defaults and examples in vars.

Dependencies
------------

None.

Example playbooks
-----------------

1) Configure wpa_supplicant.

```
# cat freebsd-postinstall.yml
- hosts: router
  roles:
    - vbotka.freebsd_postinstall


# ansible-playbook freebsd-postinstall.yml -t fp_wpasupplicant
```

2) Configure wpa_cli and network.

```
# cat freebsd-wpacli.yml

- hosts: router
  roles:
    - vbotka.freebsd_wpa_cli
    - vbotka.freebsd_network

# ansible-playbook freebsd-wpacli.yml
```

Details
-------

- [*wpa_cli*](https://www.freebsd.org/cgi/man.cgi?wpa_cli) is an utility developed, built and packaged together with [*wpa_supplicant*](https://w1.fi/).
- *wpa_cli* is installed in the base system together with *wpa_supplicant*.
- *wpa_cli* can run in the background, listen to the events from *wpa_supplicant* and execute programmable actions (wpa_cli -B -i wlan0 -a action_file.sh).
- *wpa_cli* provides reliable synchronous method to configure DHCP and routing of wireless adapters. See example of *action_file.sh* below. See also [template](https://github.com/vbotka/ansible-freebsd-wpa-cli/blob/master/templates/wpa_action.sh.j2) to be installed.

```
#!/bin/sh
ifname=$1
cmd=$2
if [ "$cmd" = "CONNECTED" ]; then
    /etc/rc.d/dhclient forcestart $ifname
    /etc/rc.d/routing restart
fi
if [ "$cmd" = "DISCONNECTED" ]; then
    /etc/rc.d/dhclient forcestop $ifname
    /etc/rc.d/routing restart
```

To control *wpa_cli* rc script */etc/rc.d/wpa_cli* is created from [template](https://github.com/vbotka/ansible-freebsd-wpa-cli/blob/master/templates/wpa_cli.j2)

```
#!/bin/sh

# PROVIDE: wpa_cli
# REQUIRE: mountcritremote
# KEYWORD: nojail nostart

. /etc/rc.subr
. /etc/network.subr

name="wpa_cli"
desc="Frontend to WPA/802.11i Supplicant for wireless network
devices. Run in daemon mode executing the action file based on events
from wpa_supplicant"
rcvar=

ifn="$2"
if [ -z "$ifn" ]; then
	return 1
fi

load_rc_config $name

command="${wpa_cli_program}"
pidfile="/var/run/${name}/${ifn}.pid"
command_args="-B -i $ifn -P $pidfile -p ${wpa_cli_ctrl_interface} -a ${wpa_cli_action_file}"
required_files="${wpa_cli_action_file}"

run_rc_command "$1"
```

*wpa_cli* is started and stopped from *network.subr* . See [patch](https://github.com/vbotka/ansible-freebsd-wpa-cli/blob/master/files/network.subr.patch)

```
# grep -A 1 -B 3 wpa_cli /etc/network.subr
	if wpaif $1; then
		/etc/rc.d/wpa_supplicant start $1
		_cfg=0		# XXX: not sure this should count
		/etc/rc.d/wpa_cli start $1
	elif hostapif $1; then
--
	_cfg=1

	if wpaif $1; then
		/etc/rc.d/wpa_cli stop $1
		/etc/rc.d/wpa_supplicant stop $1
```

Following default variables are added to */etc/defaults* . See [patch](https://github.com/vbotka/ansible-freebsd-wpa-cli/blob/master/files/rc.conf.patch)

```
# grep -r wpa_cli /etc/defaults/
/etc/defaults/rc.conf:wpa_cli_program="/usr/sbin/wpa_cli"
/etc/defaults/rc.conf:wpa_cli_ctrl_interface="/var/run/wpa_supplicant"
/etc/defaults/rc.conf:wpa_cli_action_file="/root/bin/wpa_action.sh"
```

When the *dhclient* is controlled by wpa_cli, ifconfig must by configured in rc.conf to control *wpa_supplicant* only. Options [DHCP and SYNCDHCP](https://www.freebsd.org/doc/handbook/network-wireless.html) would start unwanted additional *dhclient*.

```
ifconfig_wlan0="WPA"
```

Service *netif* than starts/restarts and stops both wpa_supplicant and wpa_cli

```

# ps ax | grep wpa
 4161  -  Ss      0:00.65 /usr/local/sbin/wpa_supplicant -s -B -i wlan0 -c /etc/wpa_supplicant.conf.wlan0 -D bsd -P /var/run/wpa_supplicant/wlan0.pid
 4171  -  Ss      0:00.44 /usr/local/sbin/wpa_cli -B -i wlan0 -P /var/run/wpa_cli/wlan0.pid -p /var/run/wpa_supplicant -a /root/bin/wpa_action.sh
```

License
-------

[![license](https://img.shields.io/badge/license-BSD-red.svg)](https://www.freebsd.org/doc/en/articles/bsdl-gpl/article.html)


Author Information
------------------

[Vladimir Botka](https://botka.link)

References
----------

- [hostapd and wpa_supplicant](https://w1.fi/)
- [Practical rc.d scripting in BSD](https://www.freebsd.org/doc/en/articles/rc-scripting/index.html)

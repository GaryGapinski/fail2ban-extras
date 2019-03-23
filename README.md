# fail2ban-extras

Extra fail2ban filters for [Fail2Ban](https://github.com/fail2ban/fail2ban).

## Filters

The following filters (and associated jail definitions) are available:

- sshd-extra ([Documentation](filter.d/sshd-extra.md))
- postfix-extra
- ufw-extra

## Usage

For a particular jail
- Place related filter definition file  (`filter.d/<jailname>.conf`) in `/etc/fail2ban/filter.d`
- Place related jail definition file (`jail.d/<jailname>.conf`) in `/etc/fail2ban/jail.d`
- Place related jail modification file (`jail.d/<jailname>.local`) in `/etc/fail2ban/jail.d`
- Restart fail2ban
- Observe fail2ban.log file for correct operation


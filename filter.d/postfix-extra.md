# postfix-extra filter

This [filter](postfix-extra.conf) matches Postfix log messages
associated with commonly-encountered Internet-borne exploits.

This is not meant to replace the default Postfix filter (`postfix.conf`), which should be used in conjunction with this one.

The following (partial) Postfix `main.cf` configuration has been used for testing the filter.

```

### smtpd (service) settings
### some settings in helo, sender, relay, recipient are duplicative
### with smtpd_delay_reject = yes they are deferred until RCPT TO
###
## reduce smtpd_timeout
smtpd_timeout = 15s

## disable the VRFY command
disable_vrfy_command = yes

## smtp restrictions
# delay reject until RCPT TO
smtpd_delay_reject = yes

## HELO/EHLO restrictions:
# require HELO/EHLO
smtpd_helo_required = yes
# punish dalliers
smtp_helo_timeout = 3s
smtpd_helo_restrictions =
        # allow trusted networks
        permit_mynetworks,
        # HELO/EHLO must be FQDN
        reject_non_fqdn_helo_hostname,
        # HELO/EHLO must not be malformed
        reject_invalid_helo_hostname,
        # HELO/EHLO must have good DNS
        reject_unknown_helo_hostname,
        permit

## Sender restrictions:
smtpd_sender_restrictions =
        # allow trusted networks
        permit_mynetworks,
        # use master.cf to allow permit_sasl_authenticated for submission
        # permit_sasl_authenticated,
        # sender must have FQDN 
        reject_non_fqdn_sender,
        # sender must have good DNS
        reject_unknown_sender_domain,
        # permit if previous did not permit or reject
        permit

## Relay restrictions
smtpd_relay_restrictions =
        # allow trusted networks
        permit_mynetworks,
        # use master.cf to allow permit_sasl_authenticated for submission
        # permit_sasl_authenticated,
        defer_unauth_destination,
        # permit if previous did not permit or reject
        permit

## Recipient restrictions:
smtpd_recipient_restrictions =
        # allow trusted networks
        permit_mynetworks,
        # use master.cf to allow permit_sasl_authenticated for submission
        # permit_sasl_authenticated,
        #!!! open relay prevention (based on $mydestination)
        #!!! reject_unauth_destination should appear before other
        #!!! restrictions which could permit unwanted acceptance
        #!!! see http://www.postfix.org/SMTPD_ACCESS_README.html#danger
        reject_unauth_destination,
        # following restrictions may duplicate those of helo, sender, and relay
        # sender must have FQDN 
        reject_non_fqdn_hostname,
        # must be FQDN
        reject_non_fqdn_sender,
        # recipient must be FQDN
        reject_non_fqdn_recipient,
        # sender must have good DNS
        reject_unknown_sender_domain,
        # recipient domain must have good DNS
        reject_unknown_recipient_domain,
        # reject bad behavior
        reject_unauth_pipelining,
        # place desired RBLs here
        #--
        # RBLs
        #--
        # perform SPF check
        check_policy_service unix:private/policy-spf,
        # permit if previous did not allow or reject
        permit

```

As well, SMTP AUTH is **disabled** for smtpd on port 25 in `master.cf`.
```
#
# Postfix master process configuration file.  For details on the format
# of the file, see the master(5) manual page (command: "man 5 master" or
# on-line: http://www.postfix.org/master.5.html).
#
# Do not forget to execute "postfix reload" after editing this file.
#
# ==========================================================================
# service type  private unpriv  chroot  wakeup  maxproc command + args
#               (yes)   (yes)   (no)    (never) (100)
# ==========================================================================
smtp      inet  n       -       y       -       -       smtpd
  -o smtpd_sasl_auth_enable=no
```
SMTP AUTH *is* enabled for submission (port 587).
 
## Usage

- The filter definition — [postfix-extra.conf](../filter.d/postfix-extra.conf) —
should be placed in `/etc/fail2ban/filter.d`.
- The jail definition — [postfix-extra.conf](../jail.d/postfix-extra.conf) —
should be placed in `/etc/fail2ban/jail.d`.
- The jail definition must be enabled using
[postfix-extra.local](../jail.d/postfix-extra.local) (or an alternative configuration file)
which should be placed in `/etc/fail2ban/jail.d`.
The variant banaction, port (all ports), and bantime are arbitrary adjustments and are optional.
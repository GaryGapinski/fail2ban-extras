# postfix-extra filter

This [filter](postfix-extra.conf) matches Postfix log messages
associated with commonly-encountered Internet-borne exploits.

This is not meant to replace the default Postfix filter (`postfix.conf`), which should be used in conjunction with this one.

The following (partial) Postfix configuration has been used for testing the filter.

```
### HELO/EHLO restrictions:
smtpd_delay_reject = yes
smtpd_helo_required = yes
smtpd_helo_restrictions =
        permit_mynetworks,
        reject_non_fqdn_helo_hostname,
        reject_invalid_helo_hostname,
        reject_unknown_helo_hostname,
        permit

### Sender restrictions:
smtpd_sender_restrictions =
        permit_mynetworks,
        # permit_sasl_authenticated,
        reject_non_fqdn_sender,
        reject_unknown_sender_domain,
        permit

### Relay restrictions
smtpd_relay_restrictions =
        permit_mynetworks,
        permit_sasl_authenticated,
        defer_unauth_destination,
        permit

### Recipient restrictions:
smtpd_recipient_restrictions =
        permit_mynetworks,
        # permit_sasl_authenticated,
        reject_invalid_hostname,
        reject_non_fqdn_hostname,
        reject_non_fqdn_sender,
        reject_non_fqdn_recipient,
        reject_unknown_sender_domain,
        reject_unknown_recipient_domain,
        reject_unauth_pipelining,
        reject_unauth_destination,
        # place desired RBLs here
        check_policy_service unix:private/policy-spf,
        permit
```

## Usage

- The filter definition — [postfix-extra.conf](../filter.d/postfix-extra.conf) —
should be placed in `/etc/fail2ban/filter.d`.
- The jail definition — [postfix-extra.conf](../jail.d/postfix-extra.conf) —
should be placed in `/etc/fail2ban/jail.d`.
- The jail definition must be enabled using
[postfix-extra.local](../jail.d/postfix-extra.local) (or an alternative configuration file)
which should be placed in `/etc/fail2ban/jail.d`.
The variant banaction, port (all ports), and bantime are arbitrary adjustments and are optional.
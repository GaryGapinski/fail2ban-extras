# sshd-extra filter
This [filter](sshd-extra.conf) matches commonly-encountered sshd log messages
associated with Internet-borne ssh exploits.

**NB**: A modified `sshd_config` file must be used to exploit some of these
filter expressions.
LogLevel VERBOSE is needed for some of the filter regexes.
Restricting cryptographic algorithms deflects many exploits.

A partial sshd_config as an example:
```
# increase logging level
LogLevel VERBOSE

# allow public key authentication
PubkeyAuthentication yes
# to the exclusion of all other authentication methods
PasswordAuthentication no
ChallengeResponseAuthentication no
GSSAPIAuthentication no
HostbasedAuthentication no

# login must be accomplished within 5 seconds
LoginGraceTime 5s
# no need to dally when PubkeyAuthentication is the sole method

# limit unauthenticated connections
MaxStartups 1
# this presumes an SSH service which is not handling multiple simultaneous connections from desired clients

# restrict host key algorithms
HostKeyAlgorithms ssh-ed25519
HostKey /etc/ssh/ssh_host_ed25519_key

# restrict key exchange algorithms
KexAlgorithms curve25519-sha256
# this has the added benefit of removing DH_GEX
# DH_GEX warrants custom moduli not commonly customized

# restrict ciphers
Ciphers chacha20-poly1305@openssh.com

# restrict MACs
MACs hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com
```
This can typically be placed at the end of the default `sshd_config` file.

Those sshd_config settings of course require that both the server and the (desired) client are compatible
implementations of the OpenSSH protocol.
Ensure that desired clients will be accommodated lest Self Denial occur.

This configuration — deliberately — does not include algorithms which are
[mandatory-to-implement](https://tools.ietf.org/html/rfc4253)
(e.g., 3des-cbc, diffie-hellman-group1-sha1, hmac-sha1) or recommended.
Requisite implementation does not entail requisite usage.

This configuration
— particularly the restrictions on host key, key exchange, cipher, and MAC algorithms —
has been observed to effectively deny over 80% of unwanted arrivees
(which at the the time of this writing predominantly identify as `SSH-2.0-libssh-0.6.3`
and for which I have no good explanation of ban efficacy).
These FIN ACK as soon as they see the server key exchange init
(observed as `Connection closed by … [preauth]`).
The ones which get farther send client key exchange init and then the server FIN ACKs
(observed as `Unable to negotiate … no matching key exchange method found`).

## Usage

The filter definition — [../filter.d/sshd-extra.conf](../filter.d/sshd-extra.conf) —
should be placed in `/etc/fail2ban/filter.d`.

The jail definition — [sshd-extra.conf](sshd-extra.conf) —
should be placed in `/etc/fail2ban/jail.d`.

The jail definition can be tweaked using an additional j`ailname.local` (not supplied here).
Example `sshd-extra.local` (to be placed in `/etc/fail2ban/jail.d`):
```
[sshd-extra]
# use IP sets for banning
# NB: ipset may not be installed by default
banaction = iptables-ipset-proto6-allports
port      = all

# increase bantime
bantime   = 1h
# bantime is typically 10m by default
```
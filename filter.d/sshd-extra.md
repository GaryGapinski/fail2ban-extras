# sshd-extra filter
This [filter](sshd-extra.conf) matches sshd log messages
associated with commonly-encountered Internet-borne ssh exploits.

**NB**: A modified `sshd_config` file must be used to exploit some of these
filter expressions.
LogLevel VERBOSE is needed for some of the filter regexes.
Restricting cryptographic algorithms deflects many exploits.

**NB**: These settings deliberately curtail interoperability by
removing algorithms commonly required by some client implementations.

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
# this has the added benefit of removing finite field Diffie-Hellman key exchange
# which warrants deliberately generated custom moduli

# restrict ciphers
Ciphers chacha20-poly1305@openssh.com

# restrict MACs
MACs hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com
```
This can typically be placed at the end of the default `sshd_config` file.

Those sshd_config settings of course require that both the server and the (desired) client are compatible
implementations of the OpenSSH protocol.
Ensure that desired clients will be accommodated lest Self Denial occur.
[https://ssh-comparison.quendi.de/](https://ssh-comparison.quendi.de/) has a nice comparison of implementations.

This configuration — deliberately — does not include algorithms which are
[mandatory-to-implement](https://tools.ietf.org/html/rfc4253)
(e.g., ssh-rsa, 3des-cbc, diffie-hellman-group1-sha1, hmac-sha1) or recommended.
Requisite implementation does not entail requisite usage.
If finite field DH must be used for interoperability, consider using groups defined in 
[RFC 8268](https://tools.ietf.org/html/rfc8268) with the possible addition of `diffie-hellman-group14-sha1`.
And generate custom moduli for them.

This configuration
— particularly the restrictions on host key, key exchange, cipher, and MAC algorithms —
has been observed to effectively deny over 80% of unwanted arrivées
(which at the the time of this writing predominantly identify as `SSH-2.0-libssh-0.6.3`¹).
These FIN ACK as soon as they see the server key exchange init
(observed as `Connection closed by … [preauth]`).
The ones which get farther send client key exchange init and then the server FIN ACKs
(observed as `Unable to negotiate … no matching key exchange method found`).
Those which manage a key exchange then fail SSH authentication.

¹ Routinely seen:
- host_key_algorithms: `ssh-rsa,ssh-dss`
- kex_algorithms: `diffie-hellman-group-exchange-sha256,diffie-hellman-group-exchange-sha1,diffie-hellman-group14-sha1,diffie-hellman-group1-sha1`

## Usage

- The filter definition — [sshd-extra.conf](../filter.d/sshd-extra.conf) —
should be placed in `/etc/fail2ban/filter.d`.
- The jail definition — [sshd-extra.conf](../jail.d/sshd-extra.conf) —
should be placed in `/etc/fail2ban/jail.d`.
- The jail definition must be enabled using
[sshd-extra.local](../jail.d/sshd-extra.local) (or an alternative configuration file)
which should be placed in `/etc/fail2ban/jail.d`.
The variant banaction, port (all ports), and bantime are arbitrary adjustments and are optional.
# sshd-extra filter

This filter matches commonly-encountered sshd log messages
associated with Internet-borne ssh exploits.

**NB**: A modified `sshd_config` file must be used to accompany some of these 
filter expressions.

A rather severe sshd_config as an example:
```
…
# increase logging level
LogLevel VERBOSE

# prevent root logins
PermitRootLogin no

# allow public key authentication
PubkeyAuthentication yes

# disallow password authentication
PasswordAuthentication no

# disallow challenge-response authentication
ChallengeResponseAuthentication no

# limit unauthenticated connections
MaxStartups 1

# login must be accomplished within 5 seconds
LoginGraceTime 5s

# restrict host keys and types
HostKey /etc/ssh/ssh_host_ed25519_key
HostKeyAlgorithms ssh-ed25519

# restrict key exchange algorithms
KexAlgorithms curve25519-sha256@libssh.org

# restrict ciphers
Ciphers chacha20-poly1305@openssh.com

# restrict MACs
MACs hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com
…
```

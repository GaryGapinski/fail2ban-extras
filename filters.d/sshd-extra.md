# sshd-extra filter

This filter matches commonly-encountered sshd log messages
associated with Internet-borne ssh exploits.

**NB**: A modified `sshd_config` file must be used to accompany some of these 
filter expressions.

A rather severe (partial) sshd_config as an example:
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
# this has the added benefit of removeg DH_GEX

# restrict ciphers
Ciphers chacha20-poly1305@openssh.com

# restrict MACs
MACs hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com
…
```

That combination, particularly the restrictions on host keys, key exchange, 
ciphers, and MACs
seems effective against over 80% of arrivees 
(who at the the time of this writing identify as `SSH-2.0-libssh-0.6.3`
for which I have no good explanation).
They FIN ACK as soon as they see the server key exchange init
(observed as `Connection closed by <address> port <port> [preauth]` in `auth.log`).
The ones who get farther encounter
`no matching key exchange method found`.

Example jail configuration:


```
[sshd-extra]
enabled  = true
port     = ssh
filter   = sshd-extra
logpath  = /var/log/auth.log
maxretry = 1
banaction = iptables-ipset-proto6-allports
```


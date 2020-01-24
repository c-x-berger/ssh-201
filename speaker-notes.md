# SSH 201 - Speaker's Notes
## Certificates and Signing
- both host and client keys can be signed by a CA, a bit like HTTPS
- benefits:
  - key expiration
  - as long as a server / client trusts the CA, no need to update authorized_keys / known_hosts

### Use
- generating a CA cert: `ssh-keygen -t ed25519 -f file-name-ca -C "Comment!"`

#### Sign a host
`ssh-keygen -s file-name-ca -h -n host.name -V +52w -I host.name-key keyfile.pub`

- sign `-h`ost key `keyfile.pub` with `file-name-ca` for `host.name`, and keep `-V`alid for 52 weeks
  - the `-I` arg is used for logging etc
- writes the certificate to `$(pwd)/${keyfile}-cert.pub`, may need root
- use a shiny new host key cert with `HostCertificate /where/is/${keyfile}-cert.pub` in `sshd_config`
- client must trust CA with `@cert-authority whatever-hosts $(cat file-name-ca.pub)` in `known_hosts`

#### Sign a user
`ssh-keygen -s file-name-ca -n your,usernames -V +52w -I user key.pub`
- server must trust CA with `TrustedUserCAKeys /path/to/file-name-ca.pub`

## The Menu
- get help with <kbd>Enter</kbd>, <kbd>~</kbd>, <kbd>?</kbd>
  - Sequence, not combination
- Once you input one, you can essentially perform as many as you can without hitting <kbd>Enter</kbd>
- List of immediately interesting options:
  - `~C`: add port forwards (demo with `-L 8000:localhost:443`)
  - `~#`: list forwarded connections, a bit verbose but can give an idea with some close reading
  - `~V`/`~v`: Increase/decrease verbosity, if you suspect SSH itself is misbehaving

## Multiplexing
- Keep one TCP/IP connection and use it for concurrent SSH sessions
- Skips "login" process being performed repeatedly
- Requires `ssh_config` setup for each host to use it on  
```apache
Host github
  Hostname github.com
  # user@host:port
  ControlPath ~/.ssh/controls/%r@%h:%p
  # use master connection iff available
  ControlMaster auto
  # keep master connection alive for 10 minutes after last session ends
  ControlPersist 10m
```
- Useful for repeatedly connecting to a host, as speed benefits only matter after first connection
- To manually close the socket, use `ssh -O stop user@host`
- Can't use The Menu's "live" port forwarding (`~C`)
  - `ssh -O forward -L 8080:localhost:80 server.example.org` forwards port 80 on `example.org` to port 8080 on the client
  - `ssh -O cancel -L 8080:localhost:80 server.example.org` cancels above

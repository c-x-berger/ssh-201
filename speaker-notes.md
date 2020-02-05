# SSH 201 - Speaker's Notes
- go through opening topics (`scp`, keys) as quick as possible

## Certificates and Signing
- both host and client keys can be signed by a CA, a bit like HTTPS
- benefits:
  - key expiration
  - as long as the server / client trusts the CA, no need to update authorized_keys / known_hosts

### Use
- generating a CA cert: `ssh-keygen -t ed25519 -f file-name-ca -C "Comment!"`

#### Sign a host
`ssh-keygen -s file-name-ca -h -n host.name -V +52w -I host.name-key keyfile.pub`

- sign `-h`ost key `keyfile.pub` with `file-name-ca` for `host.name`, and keep `-V`alid for 52 weeks
  - the `-I` arg is used for logging etc
- writes cert next to `keyfile.pub` as `keyfile-cert.pub`. Will probably need root
- use a shiny new host key cert with `HostCertificate /where/is/${keyfile}-cert.pub` in `sshd_config`
- client may trust CA with `@cert-authority whatever-hosts $(cat file-name-ca.pub)` in `known_hosts`

#### Sign a user
`ssh-keygen -s file-name-ca -n your,usernames -V +52w -I user key.pub`
- server must trust CA with `TrustedUserCAKeys /path/to/file-name-ca.pub`

## Tunneling / Forwarding
[Thank you, Stack Exchange](https://unix.stackexchange.com/a/118650/125869)

Regular tunnels map a local port to a remote port. Traffic destined for that
local port is therefore forwarded to the mapped remote port.  
The remote host can forward the traffic to yet another host if need be
(`-L localport:farfaraway:farport nearhost`).

A reverse tunnel maps a remote port to a local port. So traffic destined for
that port on **the remote machine** is instead directed to a local port.  
Your local host can send the traffic to another host if need be
(`-R remoteport:nearhost:nearport remotehost`).

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

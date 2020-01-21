## The Menu
- get help with <kbd>Enter</kbd>, <kbd>~</kbd>, <kbd>?</kbd>
  - Sequence, not combination
  - On a lot of keyboards, this expands to <kbd>Enter</kbd>, <kbd>Shift</kbd> + <kbd>`</kbd>, <kbd>Shift</kbd> + <kbd>/</kbd>
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

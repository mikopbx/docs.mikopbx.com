# Debugging behind NAT

It is relevant if the PhpStorm development environment works for NAT, and PBX is available at a public address.

* "**pbx.address.com**" - the address of MikoPBX

On the PC where PhpStorm is running, run the command:

```
ssh -R 9003:127.0.0.1:9003 root@pbx.address.com
```

This forwards the PBX-side port 9003 to the port PhpStorm listens on locally. Xdebug 3 uses **9003** by default, on both ends.

next, run debugging on MikoPBX:

```
# debugging services
pbx-console debug WorkerCdr 127.0.0.1

# debugging CLI php scripts
pbx-console xdebug enable-cli 127.0.0.1

# debugging php-agi scripts
pbx-console xdebug enable-agi 127.0.0.1

# debugging the web interface and REST API
pbx-console xdebug enable-www 127.0.0.1
```

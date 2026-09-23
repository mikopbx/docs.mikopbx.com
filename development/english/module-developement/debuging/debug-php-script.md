# Debug PHP Script

MikoPBX ships the Xdebug 3 extension (`xdebug.so`) but keeps it switched off: the file `/etc/php.d/15-xdebug.ini.disabled` is not loaded by PHP. The `pbx-console xdebug` commands create a real `/etc/php.d/15-xdebug.ini` for you, so you never have to edit the PHP configuration by hand.

To enable debugging of CLI PHP scripts:

```
pbx-console xdebug enable-cli 192.168.0.12
```

For debugging PHP scripts of the web interface (admin cabinet and REST API):

```
pbx-console xdebug enable-www 192.168.0.12
```

* **192.168.0.12** — IP address of the PC from which debugging is performed. PhpStorm must be running on that PC and listening for incoming connections.
* The generated configuration uses **`xdebug.client_port=9003`** — the Xdebug 3 default. Set the same port in PhpStorm.
* `xdebug.start_with_request=yes` is written, so **every** request starts a debug session. No browser extension and no `XDEBUG_SESSION` cookie are required.
* `enable-www` restarts `php-fpm` to pick up the new configuration; the web interface is unavailable for a few seconds.

Re-running the command with a different IP address only rewrites the `xdebug.client_host` line. To switch debugging off again, delete `/etc/php.d/15-xdebug.ini` and restart `php-fpm` (`monit restart php-fpm`).

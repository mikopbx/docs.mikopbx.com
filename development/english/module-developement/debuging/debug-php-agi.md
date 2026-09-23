# Debug PHP-AGI

AGI scripts are executed by Asterisk, so the debug configuration has to be in place _before_ Asterisk starts a new dialplan channel.

Run on the PBX:

```
pbx-console xdebug enable-agi 192.168.1.65
```

* **192.168.1.65** — IP address of the PC where PhpStorm is running.
* The command creates or updates `/etc/php.d/15-xdebug.ini` (the same file as `enable-cli`, with `xdebug.client_port=9003` and `xdebug.start_with_request=yes`) and then restarts the Asterisk daemon with `monit restart asterisk`.

Be careful! Restarting Asterisk drops all active calls, and a breakpoint held inside an AGI script blocks the channel that is executing it. Do not do this on a production PBX.

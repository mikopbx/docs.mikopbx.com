# Debuging

The most complete information on debugging in PHP Storm is presented on the [official website](https://www.jetbrains.com/help/phpstorm/debugging-with-phpstorm-ultimate-guide.html).

In this section we will describe the main features of debugging MikoPBX

* [Configuring IDE](configuring-ide.md)
* [Debug PHP-AGI](debug-php-agi.md)
* [Debug PHP Worker](debug-php-worker.md)
* [Debug PHP Script](debug-php-script.md)
* [Debugging behind NAT](debugging-behind-nat.md)

MikoPBX uses **Xdebug 3**. The extension is shipped with the firmware but disabled by default; the `pbx-console xdebug` and `pbx-console debug` commands enable it and write `/etc/php.d/15-xdebug.ini`. The debug port is **9003** everywhere — set the same port in PhpStorm.

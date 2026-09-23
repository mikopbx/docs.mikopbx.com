# Debug PHP Worker

At the beginning of work, several key workers are launched on the PBX:

* **WorkerApiCommands** - Service for processing REST API requests
* **WorkerCallEvents** - Service for recording raw CDR data
* **WorkerCdr** - Service for recording the final CDR data
* **WorkerModelsEvents** - Service for processing changes in PBX settings.
* **WorkerNotifyByEmail** - Email notification service.

To simplify debugging of these php services, the command has been added:

```
pbx-console debug WorkerCdr 192.168.0.12
```

* **WorkerCdr** — service name. The script is located by `find /usr/www/src/ -name "<name>.php"`, so any worker class name works, including workers shipped by modules and `WorkerApiCommands`, which lives in `src/PBXCoreREST/Workers/`.
* **192.168.0.12** — IP address of the PC from which debugging is performed. PhpStorm must be running on that PC.

What the command does, in order:

1. stops `crond` (`pbx-console cron stop`), so the worker is not respawned behind your back;
2. stops the running instance of the worker (`pbx-console service <name>`);
3. creates or updates `/etc/php.d/15-xdebug.ini` with `xdebug.client_host=<ip>` and `xdebug.client_port=9003`;
4. starts the worker in the foreground with `php -dxdebug.mode=debug -dxdebug.start_with_request=yes -f <path> start`.

The worker stays in the foreground until you interrupt it. Afterwards restore normal operation with `pbx-console cron start` — otherwise the workers will not be restarted automatically.

Set the PhpStorm debug port to **9003** (the Xdebug 3 default).

---
description: Published on 2026-08-03
---

# MikoPBX 2026.3.40

**New Features:**

* Module operations (install, update, enable, disable, remove) now run in the background and survive service restarts; on failure the module is automatically rolled back to its previous working version together with its settings.
* Module operation progress is shown on the button and is restored after a page reload; if an operation stalls, the interface reports it instead of waiting forever.
* Fresh installations get a modern default codec set: opus, g722, alaw, ulaw, g729, plus video h265, h264, vp9 and vp8. Existing systems keep the administrator's choice on upgrade.
* Added support for DNS SRV and NAPTR records for SIP providers that spread traffic across several servers.

**Improvements:**

* The PBX no longer accumulates memory while idle: instead of growing past 350 MB per day it stays around 65 MB.
* SSH setting changes (port, password login restriction) take effect immediately, without a reboot.
* System setting changes apply instantly and are no longer overridden by outdated values after a service restart.
* Updating from an image shows clear console output, and on single-disk systems a warning is displayed before the update starts.
* Provider status is reported accurately: no more false "registration lost" records, list and 24-hour timeline colours now match, and a registered but unreachable trunk is marked yellow.
* Disabling or deleting an outbound trunk now tells the operator to drop the registration, so incoming calls no longer arrive on a switched-off trunk.
* The security-update warning for a module is now shown only to systems running a version below the fixed one.

**Bug Fixes:**

* Updating from an image on single-disk systems no longer wipes the partition holding call recordings and data.
* Incoming calls: the line number (DID) and caller number are extracted correctly from the configured rules, and calls are no longer rejected after a cold start before the provider addresses are known.
* An incoming route no longer loses its provider when edited.
* Call recordings: after a failed transfer to a queue the recording is kept in full, and resumed conversation audio ends up in the final file.
* Employee mobile number: short internal numbers are now accepted, and a custom dial string is no longer overwritten when the number changes.
* Disabled modules no longer start their background processes, and their log files are rotated correctly again.
* A single faulty module no longer stops the background processes of all other modules.
* Fixed a hang during dial plan reload and restored the correct order of outbound routes.
* Coupon activation no longer reports a false "coupon already activated" error, and a successful system update is no longer shown as a failure.
* On virtual servers, serial port detection no longer freezes the system: hundreds of stuck processes and the blank welcome screen are gone.
* Module logos and screenshots are shown again in the module marketplace, and the menu group dropdown in module settings works again.
* Restored DTMF tone transmission on legacy H.323 gateways.

**Security:**

* The per-employee "do not record calls" setting works again: previously such conversations were recorded despite the setting.
* Login with a hardware security key (YubiKey and similar) now works; keys registered earlier must be registered again.
* Users with restricted permissions can set employee passwords again — strong passwords are no longer rejected as weak.
* Firewall: an address blocked only for the management connection no longer blocks phone registration; enabling or disabling protection, as well as edits to the allowed and blocked address lists for web access, apply without a reboot.
* Hardened protection against forged data in incoming calls and against malicious content inside installed module archives.

---
description: Published on 2026-08-03
---

# MikoPBX 2026.3.40

MikoPBX 2026.3.40 makes day-to-day PBX administration more reliable. Module operations now run in the background and recover cleanly from failures, image-based upgrades are safer on single-disk systems, and Asterisk uses significantly less memory. This release also improves SIP provider handling, call routing and recordings, Passkeys, and network access rules.

### Reliable module operations

MikoPBX now handles installing, updating, enabling, disabling, and removing modules in a dedicated background process. The operation continues if the REST API worker restarts and does not depend on the browser page remaining open. Its current stage and progress appear directly on the action button, while the complete state and any error are stored in a persistent operations journal.

If the page is reloaded or the browser misses the completion event, the interface restores the current state from the journal. After a period without events, it checks the operation automatically and reports a stalled process instead of displaying an endless progress indicator. Only one module operation can run at a time, preventing conflicting changes.

Module installation and updates are now atomic. The archive is unpacked into a staging directory and validated before the new version replaces the old one. If any stage fails, MikoPBX restores the previous module version, its settings, and its enabled or disabled state. A supervisor retry cannot damage a module that has already been restored.

Disabled modules no longer restart their background workers or leave orphaned processes behind. A fault in one module no longer removes the other module and system workers from supervision, and logs produced by long-running module processes continue to rotate correctly.

### SIP providers and DNS

DNS support for SIP providers has been expanded. Asterisk now uses an asynchronous resolver for A, AAAA, SRV, and NAPTR records, respects DNS record TTLs, and can fail over between multiple provider nodes. This is particularly useful for operators that publish SIP endpoints through SRV or NAPTR records for load distribution and redundancy.

Incoming calls are no longer rejected during a cold start while the DNS cache is still being populated. The generated configuration keeps a stable route based on the provider hostname and reconciles PJSIP with the dial plan after the hostname has been resolved.

Provider monitoring no longer records false **Registration lost** events after a worker or Redis restart. Status colors are now consistent between the provider list and the 24-hour timeline: disabled providers are gray and providers that lose registration are red. A registered trunk that stops responding to OPTIONS requests is shown as yellow with an **Unreachable** status.

When an outbound SIP provider is disabled or deleted, MikoPBX sends an unregister request before reloading the configuration. The upstream operator can therefore remove the active binding immediately and stop sending incoming calls to a trunk that has already been switched off.

Fresh installations now use an updated default codec set. Opus, G.722, alaw, ulaw, and G.729 are enabled for audio, while H.265, H.264, VP9, and VP8 are enabled for video. GSM remains enabled for system prompts. Upgrading an existing PBX does not change the codecs selected by the administrator.

### Safer image-based upgrades

Image-based upgrades have been fixed for systems where the operating system and Storage partition share one physical disk. Previously, recreating the partition table could move or reformat the Storage partition together with call recordings and other user data.

Before the upgrade, MikoPBX detects the single-disk layout, records the position of the Storage partition, and validates its filesystem. After writing the image, the partition is restored at the same location with the same UUID. If validation fails, the process stops for recovery and never attempts to format the partition automatically.

The web interface displays a dedicated warning before a single-disk upgrade. Although the Storage partition is preserved automatically, creating a backup beforehand is strongly recommended. Console output is also clearer: routine messages from disk-partitioning tools are hidden while important warnings and errors remain visible.

### Stability and memory usage

Asterisk now starts with the jemalloc memory allocator, which returns released memory to the operating system more effectively. On the test PBX, idle memory usage stabilized at approximately 65 MB instead of climbing from 64 MB to more than 350 MB over a day. If a compatible jemalloc library is unavailable, MikoPBX automatically limits the number of memory arenas used by the standard glibc allocator.

Serial port detection no longer opens `/dev/ttyS*` and `/dev/ttyAMA*` devices in blocking mode. On some VPS and virtual machine platforms, a port with no physical hardware appeared to be available but blocked the detection process indefinitely. This could create hundreds of stuck processes, increase system load, and leave the console welcome screen blank.

System settings are now reloaded into the cache from the database during startup. Stale Redis values can therefore no longer override saved settings after services restart. SSH changes, including the listening port and disabling password login, take effect immediately because the Dropbear and Monit configurations are reloaded without rebooting the PBX.

Dial plan reloads are now serialized with PJSIP configuration updates so that conflicting reloads cannot run at the same time. An additional Asterisk health check allows Monit to detect a stalled reload and recover the service automatically.

### Call routing and recordings

DID and CallerID extraction with regular expressions in incoming routes now works correctly. The dial plan previously received only a match flag instead of the extracted number, which could send a call to the wrong route. A dedicated AGI script now extracts the number, and both the input and result are validated before they are used in the dial plan.

Editing an incoming route no longer removes its selected provider. Outbound route sorting has also been fixed for systems with ten or more rules: priorities are handled as numbers, so priority 10 no longer appears before priority 2. Reordering routes now triggers one consolidated reload instead of a separate reload for every rule.

After a failed attended transfer, call recording resumes into the same split audio tracks used when the recording started. When a call is transferred to a queue that rings several employees in parallel, the CDR row for the employee who answered is preserved and the resumed conversation is included in the final recording.

The per-employee **Call recording** setting is applied correctly again. When recording is disabled for an extension, its calls are not recorded even if global call recording is enabled on the PBX.

Short internal numbers can now be saved in the employee mobile number field. A custom dial string is no longer overwritten when the mobile number changes; when no custom dial string has been entered, MikoPBX continues to fill it automatically.

### Security and access control

Hardware Passkey registration, including YubiKey, has been fixed. MikoPBX now requires discoverable credentials that a browser can offer when signing in without a username. Hardware keys enrolled on an earlier release that do not appear on the login page must be removed from the settings and enrolled again.

Users with restricted permissions can assign strong employee passwords again. Password-strength helper endpoints are available to every authenticated role, and the interface performs a local check if the REST API is temporarily unavailable instead of rejecting a password because of a network error.

In Docker environments, changes to network filters, trusted addresses, and the Firewall or Fail2Ban state now take effect without a reboot. An address blocked only for AMI is no longer added to the SIP deny ACL and therefore does not prevent phones from registering. ACL updates use lightweight Asterisk module reloads and do not interrupt active calls.

Validation of module installation archives has been hardened. The installer rejects absolute paths, backslash path separators, attempts to escape the staging directory, and symbolic links stored inside ZIP archives. Partially extracted files and temporary API response files are removed when an error occurs.

Critical module update warnings now use the minimum secure version. A warning is shown only when an installed module is older than the release containing the security fix, and disappears automatically after the module has been updated.

### Web interface fixes

The system update page can once again retrieve the list of available releases through REST API v3. The response is now read using the current API format, so the updates table no longer remains empty after a successful check.

Activating a coupon in License Management no longer sends a duplicate request or displays a false **Coupon already activated** message after a successful operation.

The module marketplace once again displays logos and screenshots hosted on the new MikoPBX CDN. The menu group dropdown works again in module settings, and enable or disable controls correctly show an operation that is still in progress.

---
title: Solutions
author: Funzi
---

# Solutions and Troubleshooting

## `dnf` always claims to require a restart

### Symptom

Even though the system has been restarted recently, e.g., after a Kernel update, `dnf needs-restarting -r` always
returns with exit code `1`.

```
# dnf needs-restarting -r
Core libraries or services have been updated since boot-up:
  * dbus
  * dbus-daemon
  * glibc
  * linux-firmware
  * systemd

Reboot is required to fully utilize these updates.
More information: https://access.redhat.com/solutions/27943
```

**This affects both Rocky Linux 8 and 9.** However,
a [patch for 9.3 is to be released soon](https://github.com/rpm-software-management/dnf-plugins-core/pull/468).

### Cause

The `dnf` plugin uses the `mtime` of `/proc/1` which is set to start of epoch due to lack of RTC on Raspberry Pi. The
correct system time is set later at boot once network connectivity is established and an NTP server is reachable.
Compared to 1970, every package update is after the last reboot which causes the wrong result.

### Solution

Until 9.3 is released, or if the system is still on
8.x, [the patch](https://github.com/rpm-software-management/dnf-plugins-core/pull/468/commits/b8132e1e04fb44c9ef8ab7dbcea2d2b3b920be2c)
can be manually applied to all versions.

```
# patch /usr/lib/python3.6/site-packages/dnf-plugins/needs_restarting.py <<EOF
--- a/plugins/needs_restarting.py
+++ b/plugins/needs_restarting.py
@@ -34,6 +34,7 @@ import functools
 import os
 import re
 import stat
+import time
 
 
 # For which package updates we should recommend a reboot
@@ -199,7 +200,28 @@ class ProcessStart(object):
 
     @staticmethod
     def get_boot_time():
-        return int(os.stat('/proc/1').st_mtime)
+        """
+        We have two sources from which to derive the boot time. These values vary
+        depending on containerization, existence of a Real Time Clock, etc.
+        For our purposes we want the latest derived value.
+        - st_mtime of /proc/1
+             Reflects the time the first process was run after booting
+             This works for all known cases except machines without
+             a RTC - they awake at the start of the epoch.
+        - /proc/uptime
+             Seconds field of /proc/uptime subtracted from the current time
+             Works for machines without RTC iff the current time is reasonably correct.
+             Does not work on containers which share their kernel with the
+             host - there the host kernel uptime is returned
+        """
+
+        proc_1_boot_time = int(os.stat('/proc/1').st_mtime)
+        if os.path.isfile('/proc/uptime'):
+            with open('/proc/uptime', 'rb') as f:
+                uptime = f.readline().strip().split()[0].strip()
+                proc_uptime_boot_time = int(time.time() - float(uptime))
+                return max(proc_1_boot_time, proc_uptime_boot_time)
+        return proc_1_boot_time
 
     @staticmethod
     def get_sc_clk_tck():
-- 
2.41.0
EOF
```

It is advisable to `version-lock` the plugin version until it's fixed upstream. On 8.x, that might be the only solutions
if the patch is not backported to 8.x upstream.  
**Use at your own risk!**

```
# dnf versionlock add python3-dnf-plugins-core
```


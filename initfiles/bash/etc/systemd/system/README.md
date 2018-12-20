Systemd scripts for HPCC-Systems Platform
=========================================

  The primary tool for starting and stopping the entire platform on a target
machine using systemd is the `hpccsystems-platform.target`.  This target script
is entirely dependent on the `environment.xml` file located on the server, and
gets generated using the `generate-hpccsystems-target.sh` script that is located
in `/opt/HPCCSystems/sbin` on standard installations.  This target script will
need to be generated any time you modify the environment.xml on the local
machine.  This can be done manually by an administrator, or by enabling the
`hpcc-environment-monitor.path` systemd script which we provide.

Manual regeneration of hpccsystems-platform.target
--------------------------------------------------

  The hpccsystems-platform.target script contains a requirement list for every
instance of a component declared in the environment.xml file.  The
hpccsystems-platform.target can be manually generated by an administrator by
invoking the `generate-hpccsystems-target.sh` script on a target system.  It
should be noted, that this will overwrite any existing
hpccsystems-platform.target currently on the system, so any old components that
were declared previously in the environment.xml, that no longer are declared,
cannot be start/stopped with the new hpccsystems-platform.target.  It is
therefore recommended that an administrator wishing to manually (or
programatically through a script) update the hpccsystems-platform.target, first
run `systemctl stop hpccsystems-platform.target` to properly shut down
hpccsystems components before the hpccsystems-platform.target changes.

Automatic regeneration of hpccsystems-platform.target
-----------------------------------------------------

  The hpccsystems-platform.target script contains a requirement list for every
instance of a component which is explicitly declared in the `environment.xml`
file.  In order to generate this required components list, the
`generate-hpccsystems-target.sh` script is invoked.  This can be done
automatically by enabling the `hpcc-environemnt-monitor.path` systemd script.
This path script will call a corresponding service script of the same name
whenever `environment.xml` is modified.

  When an administrator attempts to regenerate the hpccsystems-platform.target,
by modifying the `environment.xml` file when `hpcc-environment-monitor.path` is
enabled, the `hpcc-environment-monitor.service` script will first attempt to
shut down any hpccsystems-platform.target that is currently running on the
target system.  This is so components that were previously declared and possibly
no longer exist in the environment won't be orphaned on the target system.
After the `generate-hpccsystems-target.sh` script is done invoking the
hpccsystems configgen utility, the appropriate service scripts will be generated
from templates and a fresh hpccsystems-platform.target will exist under
`/opt/HPCCSystems/etc/systemd/system` and symlinked to `/etc/systemd/system`
where appropriate.

Systemd ulimits --------------

  If an administrator wishes to set custom ulimits for specific hpccsystems
components, it is recommended that they do so in the templates located in
`/opt/HPCCSystems/etc/systemd/system` so that they persist across regenerations
of the hpccsystems-platform.target and component service scripts.  The syntax
for modifying the user limits can be found in `man systemd.exec`.
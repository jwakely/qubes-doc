---
layout: doc
title: Config Files
permalink: /doc/config-files/
redirect_from:
- /en/doc/config-files/
- /doc/ConfigFiles/
- "/doc/UserDoc/ConfigFiles/"
- "/wiki/UserDoc/ConfigFiles/"
---

Configuration Files
===================

Qubes-specific VM config files
------------------------------

These files are placed in /rw, which survives a VM restart.
That way, they can be used to customize a single VM instead of all VMs based on the same template. 
The scripts here all run as root.

-   `/rw/config/rc.local` - script runs at VM startup.
    Good place to change some service settings, replace config files with its copy stored in /rw/config, etc.
    Example usage:

    ~~~
    # Store bluetooth keys in /rw to keep them across VM restarts
    rm -rf /var/lib/bluetooth 
    ln -s /rw/config/var-lib-bluetooth /var/lib/bluetooth
    ~~~

-   `/rw/config/qubes-ip-change-hook` - script runs in NetVM after every external IP change and on "hardware" link status change.

-   In ProxyVMs (or AppVMs with `qubes-firewall` service enabled), scripts placed in the following directories will be executed in the listed order followed by `qubes-firewall-user-script` after each firewall update.
    Good place to write own custom firewall rules.

    ~~~
    /etc/qubes/qubes-firewall.d
    /rw/config/qubes-firewall.d
    /rw/config/qubes-firewall-user-script
    ~~~

-   `/rw/config/suspend-module-blacklist` - list of modules (one per line) to be unloaded before system goes to sleep.
    The file is used only in a VM with PCI devices attached.
    Intended for use with problematic device drivers.

- In NetVMs/ProxyVMs, scripts placed in `/rw/config/network-hooks.d` will be ran when configuring Qubes interfaces. For each script, the `command`, `vif`, `vif_type` and `ip` is passed as arguments (see `/etc/xen/scripts/vif-route-qubes`). For example, consider an PV AppVM `work` with IP `10.137.0.100` and `sys-firewall` as NetVM. Assuming it's Xen domain id is arbitrary `12` then, the following script located at `/rw/config/network-hooks.d/hook-100.sh` in `sys-firewall`:
    ~~~
    #!/bin/bash

    command="$1"
    vif="$2"
    vif_type="$3"
    ip="$4"

    if [ "$ip" == '10.137.0.100' ]; then
        case "$command" in
            online)
                ip route add 192.168.0.100 via 10.137.0.100
                ;;
            offline)
                ip route del 192.168.0.100
                ;;
        esac
    fi
    ~~~

  will be executed with arguments `online vif12.0 vif 10.137.0.100` when starting `work`. Please note that in case of HVM, the script will be called twice - once with vif_type `vif`, then with vif_type `vif_ioemu` (and different interface names). As long as the ioemu interface exists, it should be preferred (up to the hook script). When VM decide to use PV interface (vif_type `vif`), the ioemu one will be unplugged.

Note that scripts need to be executable (chmod +x) to be used.

Also, take a look at [bind-dirs](/doc/bind-dirs) for instructions on how to easily modify arbitrary system files in an AppVM and have those changes persist.


GUI and audio configuration in dom0
-----------------------------------

The GUI configuration file `/etc/qubes/guid.conf` in one of a few not managed by qubes-prefs or the Qubes Manager tool.
Sample config (included in default installation):

~~~
# Sample configuration file for Qubes GUI daemon
#  For syntax go http://www.hyperrealm.com/libconfig/libconfig_manual.html

global: {
  # default values
  #allow_fullscreen = false;
  #allow_utf8_titles = false;
  #secure_copy_sequence = "Ctrl-Shift-c";
  #secure_paste_sequence = "Ctrl-Shift-v";
  #windows_count_limit = 500;
  #audio_low_latency = false;
};

# most of setting can be set per-VM basis

VM: {
  work: {
    #allow_utf8_titles = true;
  };
  video-vm: {
    #allow_fullscreen = true;
  };
};
~~~

Currently supported settings:

-   `allow_fullscreen` - allow VM to request its windows to go fullscreen (without any colorful frame).

    **Note:** Regardless of this setting, you can always put a window into fullscreen mode in Xfce4 using the trusted window manager by right-clicking on a window's title bar and selecting "Fullscreen".
    This functionality should still be considered safe, since a VM window still can't voluntarily enter fullscreen mode.
    The user must select this option from the trusted window manager in dom0.
    To exit fullscreen mode from here, press `alt` + `space` to bring up the title bar menu again, then select "Leave Fullscreen".

-   `allow_utf8_titles` - allow the use of UTF-8 in window titles; otherwise, non-ASCII characters are replaced by an underscore.

-   `secure_copy_sequence` and `secure_paste_sequence` - key sequences used to trigger secure copy and paste.

-   `windows_count_limit` - limit on concurrent windows.

-   `audio_low_latency` - force low-latency audio mode (about 40ms compared to 200-500ms by default).
    Note that this will cause much higher CPU usage in dom0.


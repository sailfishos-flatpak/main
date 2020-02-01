# Application development using Flatpak

This is to describe shortly some aspects regarding packaging of
Flatpak for ARM/i386 required for running Flatpak applications on
Sailfish OS. This is based on
https://github.com/flatpak/flatpak/issues/5 and QEMU instructions.

For general introduction of application development and packaging
using Flatpak elsewhere.


## Setting up environment

In the following, we will setup environment for ARM. For i386, you probably don't need to setup QEMU. 

1. Install QEMU with user static support. For Gentoo, specify
`static-user` use flag of `app-emulation/qemu` and set
```
QEMU_SOFTMMU_TARGETS="aarch64 arm i386 x86_64"
QEMU_USER_TARGETS="aarch64 arm i386"
```
in make.conf. For Fedora, install `qemu-system-arm qemu-user-static`.

2. If configuration is not provided out of the box, as it is for some distributions, add
`/etc/binfmt.d/qemu-arm-static.conf`:
```
:qemu-arm:M::\x7fELF\x01\x01\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x02\x00\x28\x00:\xff\xff\xff\xff\xff\xff\xff\x00\x00\x00\x00\x00\x00\x00\x00\x00\xfe\xff\xff\xff:/usr/bin/qemu-arm:F
```
For aarch64, add `/etc/binfmt.d/qemu-aarch64-static.conf`:
```
:aarch64:M::\x7fELF\x02\x01\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x02\x00\xb7:\xff\xff\xff\xff\xff\xff\xff\xfc\xff\xff\xff\xff\xff\xff\xff\xff\xfe\xff\xff:/usr/bin/qemu-aarch64:F
```

You may need to register a handler in host machine (don't register a
handler that matches the host machine). See instructions in
https://wiki.gentoo.org/wiki/Embedded_Handbook/General/Compiling_with_qemu_user_chroot .

3. Restart systemd-binfmt service: `sudo systemctl restart systemd-binfmt.service`

4. Install required platform and SDK:
```
flatpak install --user org.kde.Platform/arm/5.12 org.kde.Sdk/arm/5.12
```

5. Test installation by running
```
flatpak run --command=sh --arch=arm org.kde.Platform/arm/5.12
```

If you get into the shell, all works as expected. See below for known issues.


## Compilation and packaging

Compose JSON or YAML describing the build process. Look for
introduction elsewhere. For examples, see packaging scripts for
[Flathub](https://github.com/flathub) and
[KDE](https://phabricator.kde.org/source/flatpak-kde-applications/).

For use of locally changed sources, use something like 

```json
            "sources": [ {
                "type": "dir",
                "path": "../plasma-angelfish",
                "skip": [".git", ".flatpak-builder", ".cache"]
            } ]
```

for sources. Keep Flatpak packaging script and code in separate folder
or be sure to include `.flatpak-builder` into skipped folders. Note
that the source is relative to JSON path.

Compile and package the application:

```
flatpak-builder --arch=arm --repo=../flatpak --force-clean ../flatpak-build-desktop org.kde.mobile.angelfish.json
flatpak build-bundle --arch=arm ../flatpak angelfish.flatpak org.kde.mobile.angelfish
```

After that you will find file `angelfish.flatpak` in your current
folder. Transfer that to device and install using

```
flatpak install --user angelfish.flatpak
```

After that, it can be run by

```
flatpak-runner org.kde.mobile.angelfish
```

Assuming that the application has .desktop installed, you can configure its environment using `flatpak-runner`. Just 
start the runner from application grid and configure the application environment in it.


## Use of Sailfish keyboard

Maliit plugin should be mounted by flatpak-runner automatically. No need to adjust the sources.


## Known issues

When compiling for ARM, there are issues imposed by either absence or broken handling of statx syscall. In practice,
it means that we cannot use `qmake` in the projects compiled against KDE 5.12 Sdk and neither `qmake` nor `cmake` in
KDE 5.13. As is, we are limited to `cmake` on 5.12 right now. Symptoms are errors like "Project ERROR: Unknown module(s) in QT: core dbus" while compiling the projects.

Corresponding issues are filed and are listed below:

* https://bugs.kde.org/show_bug.cgi?id=416262
* https://bugs.launchpad.net/qemu/+bug/1861341
* https://invent.kde.org/kde/flatpak-kde-runtime/issues/2


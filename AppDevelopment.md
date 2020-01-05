# Application development using Flatpak

This is to describe shortly some aspects regarding packaging of
Flatpak for ARM/i386 required for running Flatpak applications on
Sailfish OS. This is based on
https://github.com/flatpak/flatpak/issues/5 and QEMU instructions.

For general introduction of application development and packaging
using Flatpak elsewhere.


## Setting up environment

In the following, we will setup environment for ARM. Similar procedure
should work for i386.

1. Install QEMU with user static support. For Gentoo, specify
`static-user` use flag of `app-emulation/qemu` and set
```
QEMU_SOFTMMU_TARGETS="aarch64 arm i386 x86_64"
QEMU_USER_TARGETS="aarch64 arm i386"
```
in make.conf. For Fedora, install `qemu-system-arm qemu-user-static`.

2. If configuration is not provided out of the box, add
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

If you get into the shell, all works as expected.


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


## Use of Qt Virtual Keyboard

As we don't have support for Sailfish keyboard nor QtVirtualKeyboard
by included compositor, you may have to create input panel as
described in
https://doc.qt.io/qt-5/qtvirtualkeyboard-deployment-guide.html#creating-inputpanel. In
practice, for Kirigami applications, it requires modification of the main QML with Kirigami.ApplicationWindow.
Namely, addition of

```
import QtQuick.VirtualKeyboard 2.1
```

in the header section and addition of

```qml
    InputPanel {
        id: inputPanel
        anchors.left: parent.left
        anchors.right: parent.right
        y: Qt.inputMethod.visible ? parent.height - inputPanel.height : parent.height
        visible: Qt.inputMethod.visible
    }    
```

just before closing Kirigami.ApplicationWindow definition. As Kirigami
follows Qt input methods, window content should be shifted on opening
of the keyboard.

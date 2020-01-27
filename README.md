# Flatpak support for Sailfish OS

This is the repository for documentation and for filing issues against current implementation
of Flatpak support for Sailfish OS.


## Status

Flatpak support requires changes in
[libhybris](https://github.com/libhybris/libhybris/pull/433), support
in kernel, and wrapper program
[Flatpak Runner](https://github.com/sailfishos-flatpak/flatpak-runner). As
a result of these dependencies, the support is device-specific.

Current issues are listed at
https://github.com/sailfishos-flatpak/main/issues and, for Flatpak
Runner, at https://github.com/sailfishos-flatpak/flatpak-runner/issues
. Notable limitations are due to limited support by Wayland compositor
and keyboard input. In practice, KDE Plasma Mobile applications are
supported best while they may require adjustment to include
QtVirtualKeyboard support.

In general, running Flatpak applications and their management is
similar to other Linux platforms. Exception is in use of Flatpak
Runner for executing the applications. This allows to provide
orientation information to the running applications and work around
some limitations of the current Sailfish Lipstick.


## Supported devices

* Sony Xperia XZ2, XZ2c, XZ3 via unofficial port


## Installation and usage

Assuming that your device has support for Flatpak, you need to enable
repository with the Flatpak packages and install `flatpak` with
`flatpak-runner`:

```
ssu addrepo rinigus-flatpak http://repo.merproject.org/obs/home:/rinigus:/flatpak/sailfish_latest_armv7hl/
devel-su zypper ref
devel-su zypper in flatpak flatpak-runner flatpak-maliit-plugin
```

Start `flatpak-runner` either from app grid (shown as Flatpak) or from terminal (`flatpak-runner`). 
This will generate Hybris extension that is used to run applications. Note that the extension 
has to be refreshed after libhybris update (usually after Sailfish OS update). This can be done 
in `flatpak-runner` under Extension pulley menu action.

Reboot to finish installation.

Currently, it is only supported if you install and use Flatpaks from your home folder. This will avoid
filling up system storage and will simplify of application installation. Add Flatpak repositories, 
such as Flathub:

```
flatpak remote-add --user --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

For KDE apps:
```
curl -O https://distribute.kde.org/kdeapps.flatpakrepo
flatpak remote-add --user --if-not-exists kdeapps kdeapps.flatpakrepo
```

After that, install applications with `flatpak install --user appname` in terminal. It will ask few
questions and install the application with its dependencies.

To get applications recognized by Lipstick, run `flatpak-runner`. The runner will also allow you to
set application-specific and default environment variables, Qt scaling and perceived DPI.

To run applications, either launch them from app grid or run with 

```
flatpak-runner org.kde.mobile.angelfish
```

## Application development

For description of application development, see separate
[HOWTO](AppDevelopment.md).

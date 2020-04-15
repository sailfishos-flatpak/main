# Flatpak support for Sailfish OS

This is the repository for documentation and for filing issues against current implementation
of Flatpak support for Sailfish OS.


## Status

Flatpak support requires changes in
[libhybris](https://github.com/libhybris/libhybris/pull/433, merged in 0.0.5.34), support
in kernel (PR_SET_NO_NEW_PRIVS, officially from 3.5), and wrapper program
[Flatpak Runner](https://github.com/sailfishos-flatpak/flatpak-runner). As
a result of these dependencies, the support is device-specific.

Current issues are listed at
https://github.com/sailfishos-flatpak/main/issues and, for Flatpak
Runner, at https://github.com/sailfishos-flatpak/flatpak-runner/issues
. Notable limitations are due to limited support by Wayland compositor. 
In practice, KDE Plasma Mobile applications are
supported best.

In general, running Flatpak applications and their management is
similar to other Linux platforms. Exception is in use of Flatpak
Runner for executing the applications. This allows to provide
orientation information to the running applications and work around
some limitations of the current Sailfish Lipstick.


## Supported devices

* Kernel version >= 3.5 (PR_SET_NO_NEW_PRIVS, maybe possible to backport)
* libhybris version >= 0.0.5.34


## Installation and usage

Assuming that your device has support for Flatpak, you need to enable
repository with the Flatpak packages and install `flatpak` with
`flatpak-runner`:

```
devel-su ssu addrepo rinigus-flatpak http://repo.merproject.org/obs/home:/rinigus:/flatpak/sailfish_latest_armv7hl/
devel-su pkcon refresh
devel-su pkcon install flatpak-runner
```

Start `flatpak-runner` either from app grid (shown as Flatpak) or from terminal (`flatpak-runner` without root). 
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
flatpak remote-add --user --if-not-exists kdeapps http://distribute.kde.org/kdeapps.flatpakrepo
```

After that, install applications with `flatpak install --user appname` in terminal. It will ask few
questions and install the application with its dependencies.

For Sailfish OS keyboard support, install 

```
flatpak install --user org.kde.PlatformInputContexts.MaliitSailfishOS//5.14
```

Replace with `//5.12` in the end if you wish to use 5.12 KDE runtime.


To get applications recognized by Lipstick, run `flatpak-runner`. The runner will also allow you to
set application-specific and default environment variables, Qt scaling and perceived DPI.

To run applications, either launch them from app grid or run with 

```
flatpak-runner org.kde.mobile.angelfish
```

## App specific tips

### Difference between DPI and scale

As Sailfish devices are HiDPI, there are some times issues with presenting content. By default, Flatpak Runner sets DPI 
through Qt environment variable to force the same DPI as on the device. Alternatively, its possible to use scaling, as in
openGL to stretch the output. For Qt apps, usually, there is no big difference on how to approach it as long as DPI times 
scale gives the same value. However, there are exceptions and its best to experiment with these settings if the app looks
too big ot too small. In such cases, you could reduce or increase overall scale x dpi to make it feel right.

DPI and scaling can be changed in Flapak Runner, either globally or for specific app.

### QtWebengine and related applications: Angelfish

It seems that QtWebengine (Qt 5.14) overshoots at high DPI values wih the scaling. So, I suggest to increase 
scaling and reduce DPI. On my device with ~430DPI, I am using 3x scaling and 120 DPI. This results in 
360DPI/430DPI reduction in overall size which is my preference in this case.

Angelfish allows to setup your navigation bar buttons in Settings. While the default is based on KDE 
requirements, its easy to make it into the bar that we are used to. In my case, I have: tabs, back, address, 
and reload buttons. Menu buttons are hidden.

To get into the main and context menus (left and right menus by default), all you have to do is to 
slide your finger along bottom border of the screen. Depending on direction, you will get either main or 
context menu. As, at least in my case, menus were mainly used to access back and reload buttons, I don't 
have much use for them if the buttons are out. Note that the menu buttons will appear in landscape mode 
as there is more screen space over there.

Consider also using `Reduce window when keyboard is shown` to scale app window when the keyboard is shown. Its useful
when input entries are shown under the keyboard. For example, find in page entry.

### Mirage

Similar to Angelfish, scale 3x and DPI 130 is used. It maybe due to older Qt that is used (5.12 for flatpaks at the 
moment of writing).

Its recommended to use `Reduce window when keyboard is shown` to be able to see your message while you type it.

### Telegram Desktop

To make org.telegram.dekstop working add following environment variable: `QT_QPA_PLATFORM=wayland`. To set, 
in `flatpak-runner`, add new variable and specify its name as `QT_QPA_PLATFORM` and value as `wayland`.

## Application development

For description of application development, see separate
[HOWTO](AppDevelopment.md).

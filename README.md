# Birdtray — Wayland / Flatpak fork

> **This is a personal fork of [gyunaev/birdtray](https://github.com/gyunaev/birdtray) with patches to make hide/show work on KDE Plasma Wayland and with Flatpak Thunderbird.**
> The upstream project targets X11. These changes are local and have not been submitted upstream.
> See the [Wayland / Flatpak Thunderbird](#wayland--flatpak-thunderbird) section below for what was changed and why.

# Birdtray is a system tray new mail notification for Thunderbird, which does not require extensions. [![Build](https://github.com/gyunaev/birdtray/actions/workflows/main.yml/badge.svg)](https://github.com/gyunaev/birdtray/actions/workflows/main.yml)

Birdtray is a free system tray notification for new mail for Thunderbird. It supports Linux and Windows (credit for adding and maintaining Windows support goes to @Abestanis). Patches to support other platforms are welcome.

## Features

- Shows the unread email counter in the Thunderbird system tray icon;

- Optionally can animate the Thunderbird system tray icon if new mail is received;

- You can snooze new mail notifications for a specific time period;

- Birdtray checks the unread e-mail status directly by reading the Thunderbird email mork database. This means it does not need any extensions, and thus is immune to any future extension API changes in Thunderbird;

- Starting from version 0.2 if you click on Birdtray icon, it can hide the Thunderbird window, and restore it. There is also context menu for that (this currently only works on Linux);

- You can configure which accounts you want to check for unread emails on;

- You can choose different font colors for different email accounts. This allows you, for example, to have blue unread count for personal emails, red unread count for work emails, and green unread count if both folders have unread mail.

- Can launch Thunderbird when Birdtray starts, and terminate it when Birdtray quits (configurable).

- You can choose the tray icon, or use Thunderbird original icon;

- Can monitor that Thunderbird is running, and indicate it if you accidentally closed it;

- Has configurable "New Email" functionality, allowing pre-configured email templates.


## Building

To build Birdtray from source, you would need the following components:

- A C++ compiler
- Cmake
- Qt 6.2 or higher;
- libX11-devel

To build, please do the following:

```shell script
mkdir build
cd build
cmake -DCMAKE_BUILD_TYPE=Release ..
cmake --build .
```

Launch the `./birdtray` executable from the build directory.

## Installation

Run `cmake --build . --target install` to install Birdtray.
On Unix systems, you can configure the install location by running
`cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr ..` before the command above.
On Windows, the command will build a graphical installer and execute it.
It requires [NSIS](https://nsis.sourceforge.io/Main_Page) to be installed on your system.
It is recommended for Windows users to use the
[precompiled installers for the latest release](https://github.com/gyunaev/birdtray/releases/latest).

## Usage

Once started, Birdtray will show the Thunderbird icon in system tray.

Right-click on this icon, and click Settings. Go to Monitoring tab ans select the Thunderbird MSF file for the mailbox you'd like to monitor. You can specify different notification colors for each mailbox. Birdtray will show the new email count using this color if only this folder has new mail. If more than one folder has new mail, the default color will be used.

Then select the font and default color (which will be used if more than one monitored folder has new mail).

You can also enable birdtray to start Thunderbird when you start Birdtray, or enable show/hide Thunderbird when the system tray icon is clicked, in settings.

Once you change settings, often you need to restart birdtray for the new settings to take effect.

### Configuration File Location
*Birdtray configuration is stored on a per-user basis, where the location differs depending on environment, as follows:*

#### Linux Package Installation
`$HOME/.config/birdtray-config.json`

#### Linux Flatpak Installation
`$HOME/.var/app/com.ulduzsoft.Birdtray/config/ulduzsoft/birdtray-config.json`

#### Windows Installation
`%LocalAppData%\ulduzsoft\birdtray\birdtray-config.json`

## Troubleshooting

If Birdtray shows the wrong number of unread messages, it can be caused by a corrupt mork file.
This can often be fixed by using the `Repair` functionality in Thunderbird in the mail folder settings.

Generally Birdtray expects a spec-compliant desktop manager. If you're using a barebone or non-standard/light/simple desktop manager, it is very likely that some features of Birdtray will not work properly. Most likely candidates are hiding and restoring Thunderbird window(s) - including their position and state. But sometimes even a system tray icon isn't shown. Linux Mint with Cinnamon seem to be one particularly troublesome distro which reports many issues.

### Wayland / Flatpak Thunderbird

On a pure Wayland session (e.g. KDE Plasma Wayland), `QNativeInterface::QX11Application` returns `nullptr`, so the traditional X11 window-detection path cannot find Thunderbird's window handle. The same applies when Thunderbird is installed as a **Flatpak**: its process runs inside a `bwrap` sandbox, so `argv[0]` is `bwrap` rather than `thunderbird`, causing the previous `/proc` scan to miss it.

The following fixes were applied to enable full hide/show support on Wayland + Flatpak Thunderbird:

1. **Process-based fallback detection** (`isThunderbirdProcessRunning()`): when the X11 root window is unavailable, `lookup()` now scans all `/proc/<PID>/cmdline` entries and checks *every* argument (not just `argv[0]`) for the configured process name. This correctly detects Flatpak Thunderbird even though `argv[0]` is `bwrap`.

2. **`mProcessOnly` mode**: a new flag marks when Thunderbird was detected via `/proc` but not via X11. `isValid()`, `isHidden()`, `checkWindow()`, and `lookup()` all branch on this flag so the tray icon behaves correctly without a window handle.

3. **KWin D-Bus scripting for hide/show** (`kwinScriptMinimize()`): on Wayland, window management is delegated to KWin via `qdbus6 org.kde.KWin`. A temporary JavaScript file is written to `/tmp`, loaded as a KWin script, and run asynchronously.
   - **Hide**: sets `skipTaskbar = true`, `skipPager = true`, `minimized = true` — removes the window from the taskbar and panel pager.
   - **Show**: sets `skipTaskbar = false`, `skipPager = false`, `minimized = false`, and assigns `workspace.activeWindow` to raise and focus the window.

These changes are in effect when building from source on a Wayland session with KDE Plasma. The `qdbus6` binary must be available (provided by `qttools6` / `qt6-tools`). Email monitoring continues to work identically to the X11 path.

## Submitting bugs and feature requests

Please use Github issue tracker. Please attach the log output, if relevant. It could be obtained from Settings -> Advanced (tab) -> Show Log Window (button) -> copy-paste from it into bug report.

### Translations

Translations are maintained by the community.
If you want to add a translation, you can follow [this guide](https://github.com/gyunaev/birdtray/wiki/Add-a-new-translation)
and if you want to edit an existing translation, read [this page](https://github.com/gyunaev/birdtray/wiki/Edit-an-existing-translation).

## Author and license

Birdtray is written by George Yunaev, and is licensed under GPLv3 license.

![birdtray-settings](screenshots/birdtray-settings.png)

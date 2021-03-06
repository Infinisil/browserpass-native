# Browserpass - native messaging host

This is a host application for [browserpass](https://github.com/browserpass/browserpass-extension) browser extension providing it access to your password store. The communication is handled through [Native Messaging API](https://developer.chrome.com/extensions/nativeMessaging).

## Table of Contents

-   [Installation](#installation)
    -   [Install via package manager](#install-via-package-manager)
    -   [Install manually](#install-manually)
        -   [Install on Nix / NixOS](#install-on-nix--nixos)
        -   [Install on Windows](#install-on-windows)
        -   [Install on Windows through WSL](#install-on-windows-through-wsl)
    -   [Configure browsers](#configure-browsers)
        -   [Configure browsers on Windows](#configure-browsers-on-windows)
-   [Building the app](#building-the-app)
    -   [Build locally](#build-locally)
    -   [Build using Docker](#build-using-docker)
-   [Updates](#updates)
-   [FAQ](#faq)
    -   [Hints for configuring gpg](#hints-for-configuring-gpg)
-   [Contributing](#contributing)

## Installation

### Install via package manager

The following operating systems provide a browserpass package that can be installed using a package manager:

-   TODO
-   TODO

Once the package is installed, **refer to the section [Configure browsers](#configure-browsers)**.

If your OS is not listed above, proceed with the manual installation steps below.

### Install manually

Download [the latest Github release](https://github.com/browserpass/browserpass-native/releases), choose either the source code archive (if you want to compile the app yourself) or an archive for your operating system (it contains a pre-built binary).

All release files are signed with [this PGP key](https://keybase.io/maximbaz). To verify the signature of a given file, use `$ gpg --verify <file>.sig`.

It should report:

```
gpg: Signature made ...
gpg:                using RSA key 8053EB88879A68CB4873D32B011FDC52DA839335
gpg: Good signature from "Maxim Baz <...>"
gpg:                 aka ...
Primary key fingerprint: EB4F 9E5A 60D3 2232 BB52  150C 12C8 7A28 FEAC 6B20
     Subkey fingerprint: 8053 EB88 879A 68CB 4873  D32B 011F DC52 DA83 9335
```

Unpack the archive. If you decided to compile the application yourself, refer to the [Building the app](#building-the-app) section on how to do so. Once complete, continue with the steps below.

Configure the hosts json files using `make configure` and then install the app using `sudo make install` (if you compiled it using `make browserpass`) or `sudo make BIN=browserpass-XXXX install` (if you downloaded a release with pre-built binary). Both `configure` and `install` targets respect `PREFIX` and `DESTDIR` parameters if you want to customize the install location (e.g. to avoid `sudo`).

Finally proceed to the [Configure browsers](#configure-browsers) section.

#### Install on Nix / NixOS

If you wish to have a stateless setup, make sure you have this in your `/etc/nixos/configuration.nix` and rebuild your system:

```nix
{ pkgs, ... }: {
  programs.browserpass.enable = true;
  environment.systemPackages = with pkgs; [
    # All of these browsers will work with it
    chromium
    firefox
    google-chrome
    vivaldi
  ];
}
```

Note: `firefox*-bin` versions do _not_ work statelessly. If you require such firefox versions, use the stateful setup as described below.

For a stateful setup, install Browserpass with:

```
nix-env -iA nixpkgs.browserpass
```

Then link the necessary files (see [Configure browsers](#configure-browsers) section and refer to Makefile in particular), but use the `~/.nix-profile` folder as the source of json files (for example, instead of `/usr/lib/browserpass/hosts/chromium/com.github.browserpass.native.json` as in `hosts-chromium-user` make goal you would use `~/.nix-profile/usr/lib/browserpass/hosts/chromium/com.github.browserpass.native.json`.

#### Install on Windows

The Makefile currently does not support Windows, so instead of `sudo make install` you'd have to do a bit of a manual work.

First, copy the contents of the extracted `browserpass-windows64` folder to a permanent location where you want Browserpass to be installed, for the sake of example let's suppose it is `C:\Program Files\Browserpass\`.

Then edit the hosts json files (in our example `C:\Program Files\Browserpass\browser-files\*-host.json`) and replace `%%replace%%` with a full path to `browserpass-windows64.exe` (in our example `C:\\Program Files\\Browserpass\\browserpass-windows64.exe`).

Finally proceed to the [Configure browsers on Windows](#configure-browsers-on-windows) section.

#### Install on Windows through WSL

If you want to use WSL instead, follow Linux installation steps, then create `%localappdata%\Browserpass\browserpass-wsl.bat` with the following contents:

```
@echo off
bash -c /usr/bin/browserpass-linux64
```

Then edit the hosts json files (in our example `C:\Program Files\Browserpass\browser-files\*-host.json`) and replace `%%replace%%` with a full path to `browserpass-wsl.bat` you've just created.

Finally proceed to the [Configure browsers on Windows](#configure-browsers-on-windows) section.

Remember to check [Hints for configuring gpg](#hints-for-configuring-gpg) on how to configure pinentry to unlock your PGP key.

### Configure browsers

The Makefile (which is also available in `/usr/lib/browserpass/`, if you installed via package manager) contains the following `make` goals to configure the browsers you use:

| Command                    | Description                                                                |
| -------------------------- | -------------------------------------------------------------------------- |
| `sudo make hosts-chromium` | Configure browserpass for Chromium browser, system-wide                    |
| `make hosts-chromium-user` | Configure browserpass for Chromium browser, for the current user only      |
| `sudo make hosts-chrome`   | Configure browserpass for Google Chrome browser, system-wide               |
| `make hosts-chrome-user`   | Configure browserpass for Google Chrome browser, for the current user only |
| `sudo make hosts-vivaldi`  | Configure browserpass for Vivaldi browser, system-wide                     |
| `make hosts-vivaldi-user`  | Configure browserpass for Vivaldi browser, for the current user only       |
| `sudo make hosts-brave`    | Configure browserpass for Brave browser, system-wide                       |
| `make hosts-brave-user`    | Configure browserpass for Brave browser, for the current user only         |
| `sudo make hosts-firefox`  | Configure browserpass for Firefox browser, system-wide                     |
| `make hosts-firefox-user`  | Configure browserpass for Firefox browser, for the current user only       |

In addition, Chromium-based browsers support the following `make` goals:

| Command                       | Description                                                                                  |
| ----------------------------- | -------------------------------------------------------------------------------------------- |
| `sudo make policies-chromium` | Automatically install browser extension for Chromium browser, system-wide                    |
| `make policies-chromium-user` | Automatically install browser extension for Chromium browser, for the current user only      |
| `sudo make policies-chrome`   | Automatically install browser extension for Google Chrome browser, system-wide               |
| `make policies-chrome-user`   | Automatically install browser extension for Google Chrome browser, for the current user only |
| `sudo make policies-brave`    | Automatically install browser extension for Brave browser, system-wide                       |
| `make policies-brave-user`    | Automatically install browser extension for Brave browser, for the current user only         |

#### Configure browsers on Windows

The Makefile currently does not support Windows, so instead of the make goals shown above you'd have to do a bit of a manual work.

Open `regedit` and create a browser-specific subkey, it can be under `HKEY_CURRENT_USER` (`hkcu`) or `HKEY_LOCAL_MACHINE` (`hklm`) depending if you want to configure Browserpass only for your user or for all users respectively:

1. Google Chrome: `hkcu:\Software\Google\Chrome\NativeMessagingHosts\com.github.browserpass.native`
1. Firefox: `hkcu:\Software\Mozilla\NativeMessagingHosts\com.github.browserpass.native`

Inside this subkey create a new property called `(Default)` with the value of the full path to the browser-specific hosts json file, for example:

1. Google Chrome: `C:\Program Files\Browserpass\browser-files\chromium-host.json`
1. Firefox: `C:\Program Files\Browserpass\browser-files\firefox-host.json`

You can automate all of these steps by running the following commands in PowerShell:

```powershell
# Google Chrome
New-Item -Path "hkcu:\Software\Google\Chrome\NativeMessagingHosts\com.github.browserpass.native" -force
New-ItemProperty -Path "hkcu:\Software\Google\Chrome\NativeMessagingHosts\com.github.browserpass.native" -Name "(Default)" -Value "C:\Program Files\Browserpass\browser-files\chromium-host.json"

# Firefox
New-Item -Path "hkcu:\Software\Mozilla\NativeMessagingHosts\com.github.browserpass.native" -force
New-ItemProperty -Path "hkcu:\Software\Mozilla\NativeMessagingHosts\com.github.browserpass.native" -Name "(Default)" -Value "C:\Program Files\Browserpass\browser-files\firefox-host.json"
```

For other browsers, please explore the registry to find the correct location, and peek into Makefile for inspiration.

## Building the app

### Build locally

Make sure you have the latest stable Go installed.

The following `make` goals are available (check Makefile for more details):

| Command                      | Description                         |
| ---------------------------- | ----------------------------------- |
| `make` or `make all`         | Compile the app and run tests       |
| `make browserpass`           | Compile the app for your OS         |
| `make browserpass-linux64`   | Compile the app for Linux 64-bit    |
| `make browserpass-windows64` | Compile the app for Windows 64-bit  |
| `make browserpass-darwin64`  | Compile the app for Mac OS X 64-bit |
| `make browserpass-openbsd64` | Compile the app for OpenBSD 64-bit  |
| `make browserpass-freebsd64` | Compile the app for FreeBSD 64-bit  |
| `make test`                  | Run tests                           |

### Build using Docker

First build the docker image using the following command in the project root:

```shell
docker build -t browserpass-native .
```

The entry point in the docker image is the `make` command. To run it:

```shell
docker run --rm -v "$(pwd)":/src browserpass-native
```

Specify `make` goal(s) as the last parameter, for example:

```shell
docker run --rm -v "$(pwd)":/src browserpass-native test
```

Refer to the list of available `make` goals above.

## Updates

If you installed the app using a package manager for your OS, you will likely update it in the same way.

If you installed manually, repeat the steps in the [Install manually](#install-manually) section.

## FAQ

### Hints for configuring gpg

First make sure `gpg` and some `pinentry` are installed.

-   on macOS many people succeeded with `pinentry-mac`
-   on Windows WSL people succeded with [pinentry-wsl-ps1](https://github.com/diablodale/pinentry-wsl-ps1)

Make sure your pinentry program is configured in `~/.gnupg/gpg-agent.conf`:

```
pinentry-program /full/path/to/pinentry
```

If Browserpass is unable to locate the proper `gpg` binary, try configuring a full path to your `gpg` in the browser extension settings or in `.browserpass.json` file in the root of your password store:

```json
{
    "gpgPath": "/full/path/to/gpg"
}
```

## Contributing

1. Fork [the repo](https://github.com/browserpass/browserpass-extension)
2. Create your feature branch
    - `git checkout -b my-new-feature`
3. Commit your changes
    - `git commit -am 'Add some feature'`
4. Push the branch
    - `git push origin my-new-feature`
5. Create a new pull request

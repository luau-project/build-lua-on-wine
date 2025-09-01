# Setup MinGW to build Lua on Wine

In a machine running Debian (current stable, `trixie`), this guide shows how to setup MinGW ([https://sourceforge.net/projects/mingw/](https://sourceforge.net/projects/mingw/)) on Wine to build Lua from source.

> [!IMPORTANT]
> 
> This tutorial was written in a system with `amd64` architecture. Other architectures were not tested.

## Table of Contents

* [Prerequisites](#prerequisites)
* [Install Wine](#install-wine)
* [Configure Wine for Lua and MinGW](#configure-wine-for-lua-and-mingw)
    * [Initial Wine configuration](#initial-wine-configuration)
    * [Create a symbolic link pointing to Lua development directory](#create-a-symbolic-link-pointing-to-lua-development-directory)
    * [Set permanent helper environment variables on Wine](#set-permanent-helper-environment-variables-on-wine)
* [Setup MinGW on Wine](#setup-mingw-on-wine)
* [Build and test Lua on Wine](#build-and-test-lua-on-wine)

## Prerequisites

To follow this guide, we need:

* basic skills to use the terminal;
* internet connection.

Moreover, we are going to employ well-known softwares:

* `wget` to download `MinGW`;
* `tar` to extract `MinGW` archives.

Most likely, these softwares come preinstalled on most Unix-like distributions. In case they are not installed on your system, install them:

```bash
sudo apt install -y tar wget
```

## Install Wine

1. To run 32-bit applications, Wine needs `i386` packages. Then, add the `i386` architecture:

    ```bash
    sudo dpkg --add-architecture i386
    ```

2. Update list of available packages:

    ```bash
    sudo apt update
    ```

3. Finally, install the packages:

    ```bash
    sudo apt install -y wine wine32 wine64
    ```

    <img width="1036" height="653" alt="01-install-wine" src="https://github.com/user-attachments/assets/37493abb-fde7-4926-a843-18c36b899e34" />

## Configure Wine for Lua and MinGW

### Initial Wine configuration

> [!NOTE]
> 
> This guide assumes that the reader has an entry-level knowledge about Windows. Thus, we install Wine in a separate directory under `$HOME` customized to have `MinGW` tools available in the `PATH` environment variable, not interfering with other toolchains. Feel free to adapt this guide to your skill level.

1. Since we are going to install the *whole* Wine-MinGW tools in a separate directory, I highly recommend you to set a permanent shell variable on your system to hold the full path for such directory (*you may need to replace `$HOME/.bashrc` below by the shell file, in case it is not `bash`*):

    ```bash
    echo 'export WINE_MINGW=$HOME/wine-MinGW' >> $HOME/.bashrc
    ```

2. Reload the variables on your current shell:

    ```bash
    source $HOME/.bashrc
    ```

3. Test the assigned variable:

    ```bash
    echo $WINE_MINGW
    ```

    <img width="1036" height="653" alt="02-initial-wine-config" src="https://github.com/user-attachments/assets/d29a06ca-efc0-46fe-b3a7-d5c280fe95d2" />

4. Initialize Wine in the directory pointed by `$WINE_MINGW` using Windows 10 as target version:

    ```bash
    WINEPREFIX=$WINE_MINGW winecfg /v win10
    ```

    <img width="1036" height="653" alt="03-wine-initialization" src="https://github.com/user-attachments/assets/b3086c4b-f0dc-4c5e-bbb3-76130ba403e0" />

> [!TIP]
> 
> The errors shown in the command execution are not relevant.

5. List the contents of `$WINE_MINGW` directory:

    ```bash
    ls -la $WINE_MINGW
    ```

    <img width="1036" height="653" alt="04-wine-mingw-contents" src="https://github.com/user-attachments/assets/e5ad9e54-e783-4738-88bb-03b478bd346f" />

    In the image, it is worth to note a folder named `drive_c` - equivalent to the root `C:\` on Windows - and the file `user.reg` used to store registry entries (*Windows uses the registry to store configuration info about the system*).

### Create a symbolic link pointing to Lua development directory

1. For this step, we are going to place a symbolic link on `$WINE_MINGW/drive_c/lua` pointing to Lua development directory. This has the same effect to have the development directory at `C:\lua` on Windows. I'll assume the development directory is stored at `$HOME/lua-5.4.8` (**CHANGE** this to the real path on your system):

    ```bash
    ln -s $HOME/lua-5.4.8 $WINE_MINGW/drive_c/lua
    ```

2. List the contents of Lua development directory on Wine:

    ```bash
    WINEPREFIX=$WINE_MINGW wine cmd /C dir C:\\lua
    ```

    <img width="1036" height="653" alt="05-lua-dev-dir-contents" src="https://github.com/user-attachments/assets/40fb52cc-39ce-41eb-a396-e72fb3dd3bf8" />

> [!NOTE]
> 
> The last command is equivalent to run `dir C:\lua` on `cmd`. Since arguments are passed on our Unix shell, we have to use double backslash (`\\`) to forward a single backslash to `cmd`.

### Set permanent helper environment variables on Wine

We are going to set two helper environment variables for the current user on Wine:

* adjust `PATH` to have the directory of MinGW binaries (`gcc`, `mingw32-make`, etc);
* set `LUA_DIR` to hold Lua development directory.

To do that, we have to:

1. Open the file `$WINE_MINGW/user.reg` on your text editor of choice. I'll use the `nano` editor for this task:

    ```bash
    nano $WINE_MINGW/user.reg
    ```

2. Scroll down to find `[Environment]`:

    <img width="1036" height="653" alt="06-env-section" src="https://github.com/user-attachments/assets/7a4d756f-f909-4ad3-8650-b95903d940d7" />

3. In the `[Environment]` section, add the lines below:

    ```bash
    "PATH"="C:\\MinGW\\bin"
    "LUA_DIR"="C:\\lua"
    ```

    <img width="1036" height="653" alt="07-changed-env-section" src="https://github.com/user-attachments/assets/8c459098-b687-47a3-a977-851896fb891d" />

4. Save the changes;

5. Test the assigned environment variables:

    ```bash
    WINEPREFIX=$WINE_MINGW wine cmd /C echo %PATH%
    WINEPREFIX=$WINE_MINGW wine cmd /C echo %LUA_DIR%
    WINEPREFIX=$WINE_MINGW wine cmd /C dir %LUA_DIR%
    ```

    <img width="1036" height="653" alt="08-test-env-vars" src="https://github.com/user-attachments/assets/856f6bf2-9182-4ec3-b707-37ad51d0c6a2" />

> [!TIP]
> 
> In the output of `PATH` environment variable, it is different compared to what we set, because `PATH` is a special environment variable and Windows merges the values of system paths with user paths.

## Setup MinGW on Wine

1. We are going to install MinGW ([https://sourceforge.net/projects/mingw/](https://sourceforge.net/projects/mingw/)) on Wine through a shell script. Save the following script to a file named `setup-MinGW.sh`:

    ```bash
    target_dir="$1"

    if ! [ -d "${target_dir}" ]; then
        mkdir -p "${target_dir}";

        if ! [ $? = 0 ]; then
            echo "Failed to create directory ${target_dir}";
            exit 1;
        fi
    fi

    base_url="https://sourceforge.net/projects/mingw/files/Installer/mingw-get/mingw-get-0.6.2-beta-20131004-1"

    for file in \
        mingw-get-setup-0.6.2-mingw32-beta-20131004-1-xml.tar.xz \
        mingw-get-0.6.2-mingw32-beta-20131004-1-bin.tar.xz \
        ; do

        target_file="${target_dir}/${file}";

        if ! [ -e "${target_file}" ]; then
            wget "-P${target_dir}" "${base_url}/${file}";

            if ! [ $? = 0 ]; then
                echo "Failed to download ${file}";
                exit 1;
            fi
        fi

        tar -C "${target_dir}" -xf "${target_file}";

        if ! [ $? = 0 ]; then
            echo "Failed to extract ${file}";
            exit 1;
        fi
    done
    ```

2. Run the script to download `MinGW` and extract it at `$WINE_MINGW/drive_c/MinGW`, equivalent to `C:\MinGW` on Windows:

    ```bash
    sh setup-MinGW.sh $WINE_MINGW/drive_c/MinGW
    ```

    <img width="1036" height="653" alt="09-download-extract-mingw" src="https://github.com/user-attachments/assets/58a3a387-f9a1-4988-8ff4-2ef3d826aa6d" />

3. Test the `mingw-get` tool:

    ```bash
    WINEPREFIX=$WINE_MINGW wine cmd /C mingw-get --help
    ```

    <img width="1036" height="653" alt="10-test-mingw-get" src="https://github.com/user-attachments/assets/a37e4388-bee6-49bc-a9e2-17e95ebbae7b" />

4. Update `MinGW` catalogue:

    ```bash
    WINEPREFIX=$WINE_MINGW wine cmd /C mingw-get update
    ```

    <img width="1036" height="653" alt="11-update-mingw-catalogue" src="https://github.com/user-attachments/assets/bcc5a2d8-8633-455d-b510-e3bdead0240d" />

5. Install the packages `mingw32-gcc` and `mingw32-make` to have `GCC`, `GNU Make` and all its dependencies:

    ```bash
    WINEPREFIX=$WINE_MINGW wine cmd /C mingw-get install --all-related mingw32-gcc mingw32-make
    ```

    <img width="1036" height="653" alt="12-install-packages" src="https://github.com/user-attachments/assets/9d1f036f-9c92-4136-9053-82f192d9a541" />

6. Test `gcc` and `mingw32-make`:

    ```bash
    WINEPREFIX=$WINE_MINGW wine cmd /C mingw32-gcc --version
    WINEPREFIX=$WINE_MINGW wine cmd /C mingw32-make --version
    ```

    <img width="1036" height="653" alt="13-test-gcc-and-make" src="https://github.com/user-attachments/assets/1dbb457d-7081-44ad-ba19-a610cb41724d" />

## Build and test Lua on Wine

> [!NOTE]
> 
> All the previous steps are part of setup operations. Thus, they only need to be done once.

1. Finally, build Lua on Wine with `gcc` through `mingw32-make`:

    ```bash
    WINEPREFIX=$WINE_MINGW wine cmd /C mingw32-make -C %LUA_DIR% mingw
    ```

    <img width="1036" height="653" alt="14-build-lua-on-wine" src="https://github.com/user-attachments/assets/ac121cce-f165-48ab-8179-fc482c8b875e" />

2. Test the built interpreter:

    ```bash
    WINEPREFIX=$WINE_MINGW wine cmd /C %LUA_DIR%\\src\\lua.exe -v
    ```

    <img width="1036" height="653" alt="15-test-lua-interpreter" src="https://github.com/user-attachments/assets/aa05c98a-d5c6-4014-8d92-0ddba3bd4c09" />

---

[Back to TOC](#table-of-contents) | [Back to home](../README.md)
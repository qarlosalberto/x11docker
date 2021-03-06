# x11docker: ![x11docker logo](x11docker.png) Run GUI applications in docker
## Avoid X security leaks and improve container security

Graphical applications and desktops in docker are similar in usage to a Virtual
Machine. They are isolated from host in several ways. 
It is possible to run applications that would not run on host due to missing 
dependencies. 
For example, you can run latest development versions or outdated 
versions of applications, or even multiple versions at the same time.

Practical differences to a VM: 
Docker containers need much less resources. 
x11docker discardes containers after use. 
Persistant data and configuration storage is done with shared folders. 
Persistant container system changes can be done in Dockerfile. 
System changes in running containers are discarded after use.

[x11docker wiki](https://github.com/mviereck/x11docker/wiki) provides some Howto's for basic setups without x11docker.

 - Avoids X security leaks by running [additional X servers](#choice-of-x-servers-and-wayland-compositors).
 - Improves container [security](#security):
   - Restricts container capabilities to bare minimum.
   - Container user is same as host user to avoid root in container.
 - No dependencies inside of docker images.
 - No obliging [dependencies](#dependencies) on host beside X and docker. Recommended: `xpra` and `Xephyr`.
 - [Wayland](#wayland) support.
 - [Optional features](#options): 
   - [Persistent data storage](#shared-folders-and-home-in-container) with shared host folders and a persistant `HOME` in container.
   - [Sound](#sound) with pulseaudio or ALSA.
   - [Hardware acceleration](#hardware-acceleration) for OpenGL.
   - [Clipboard](#clipboard) sharing.
   - [Printing](#printer) through CUPS.
   - [Webcam](#webcam) support.
   - [Language locale](#language-locales) creation.
 - Remote access with [SSH](https://github.com/mviereck/x11docker/wiki/Remote-access-with-SSH), [VNC](https://github.com/mviereck/x11docker/wiki/VNC) or [HTML5 in browser](https://github.com/mviereck/x11docker/wiki/Container-applications-running-in-Browser-with-HTML5) possible.
 - Supports [init systems](#init-system) `tini`, `runit`, `openrc`, `SysVinit` and `systemd` in container.
 - Developed on debian 9. Tested on fedora 28, CentOS 7, openSUSE 42.3, Ubuntu 18.04, Manjaro 17, Mageia 6 and Arch Linux. Runs also on MS Windows in [MSYS2, Cygwin and WSL](#msys2-cygwin-and-wsl-on-ms-windows).
 - Easy to use. [Examples](#examples): 
   - `x11docker jess/cathode`
   - `x11docker --desktop --size 320x240 x11docker/lxde`
 
![x11docker-gui screenshot](/../screenshots/screenshot-retroterm.png?raw=true "Cathode retro term in docker") ![LXDE in xpra](/../screenshots/screenshot-lxde-small.png?raw=true "LXDE desktop in docker")

# GUI for x11docker
`x11docker-gui` is an optional graphical frontend for `x11docker`. It runs from console, too.
 - `x11docker-gui` needs package `kaptain`. If your distribution misses it, look at [kaptain repository](https://github.com/mviereck/kaptain). 
 - If `kaptain` is not installed on your system, `x11docker-gui` uses image [`x11docker/kaptain`](https://hub.docker.com/r/x11docker/kaptain/). 

![x11docker-gui screenshot](/../screenshots/x11docker-gui.png?raw=true "GUI for x11docker")

# Terminal usage
Just type `x11docker IMAGENAME [IMAGECOMMAND]`. Get an [overview of options](https://github.com/mviereck/x11docker/wiki/x11docker-options-overview) with `x11docker --help`. For desktop environments in image add option `--desktop` (or short option `-d`).
General syntax:
```
To run a docker image with new X server (auto-choosing X server):
  x11docker [OPTIONS] IMAGE [COMMAND]
  x11docker [OPTIONS] -- IMAGE [COMMAND [ARG1 ARG2 ...]]
  x11docker [OPTIONS] -- DOCKER_RUN_OPTIONS -- IMAGE [COMMAND [ARG1 ARG2 ...]]
To run a host application on a new X server:
  x11docker [OPTIONS] --exe COMMAND
  x11docker [OPTIONS] --exe -- COMMAND [ARG1 ARG2 ...]
To run only a new empty X server:
  x11docker [OPTIONS]
```

# Installation
### Minimal installation
For a first test, you can run with `bash x11docker` respective `bash x11docker-gui`. 
For manual installation, make x11docker executable with `chmod +x x11docker` and move it to `/usr/bin`.
### Installation options
As root, you can install, update and remove x11docker on your system:
 - `x11docker --install` : install x11docker and x11docker-gui from current directory. 
 - `x11docker --update` : download and install latest [release](https://github.com/mviereck/x11docker/releases) from github.
 - `x11docker --update-master` : download and install latest master version from github.
 - `x11docker --remove` : remove all files installed by x11docker.
 
Copies `x11docker` and `x11docker-gui` to `/usr/bin`. Creates an icon in `/usr/share/icons`. Creates `x11docker.desktop` in `/usr/share/applications`. Copies `README.md`, `CHANGELOG.md` and `LICENSE.txt` to `/usr/share/doc/x11docker`.
### Shortest way for first installation:
```
wget https://raw.githubusercontent.com/mviereck/x11docker/master/x11docker -O /tmp/x11docker
sudo bash /tmp/x11docker --update
rm /tmp/x11docker
```

# Options
Description of some commonly used options. Get an [overview of all options](https://github.com/mviereck/x11docker/wiki/x11docker-options-overview) with `x11docker --help`.
## Desktop or seamless mode
x11docker assumes that you want to run a single application in seamless mode, i.e. a single window on your regular desktop. If you want to run a desktop environment in image, add option `--desktop`. If you don't specify a [desired X server](#choice-of-x-servers-and-wayland-compositors), x11docker chooses the best matching one depending on chosen options and installed dependencies.
  - Seamless mode is supported with options `--xpra` and `--nxagent`. As a fallback insecure option `--hostdisplay` is possible.
    - If neither `xpra` nor `nxagent` are installed, but x11docker finds a desktop capable X server like `Xephyr`, it avoids insecure option `--hostdisplay` and runs Xephyr with a host window manager.
    - You can specify a host window manager with option `--wm WINDOWMANAGER`, for example `--wm openbox`.
  - Desktop mode with `--desktop` is supported with all X server options except `--hostdisplay`. If available, x11docker prefers `Xephyr` and `nxagent` 
## Shared folders and HOME in container
Changes in a running docker image are lost, the created docker container will be discarded. For persistent data storage you can share host directories:
 - Option `--home` creates a host directory in `~/x11docker/IMAGENAME` that is shared with the container and mounted as home directory. Files in container home and configuration changes will persist. 
 - Option `--sharedir DIR` mounts a host directory at the same location in container without setting `HOME`.
 - Option `--homedir DIR` is similar to `--home` but allows you to specify a custom host directory for data storage.
 - Special cases for `$HOME`:
   - `--homedir $HOME` will use your host home as container home. Discouraged, use with care.
   - `--sharedir $HOME` will mount your host home as a subfolder of container home. 
 
For persistant changes of image system, adjust Dockerfile and rebuild. To add custom applications to x11docker example images, you can create a new Dockerfile based on them. Example:
```
# xfce desktop with Midori internet browser
FROM x11docker/xfce
RUN apt-get update && apt-get install -y midori
```
## Hardware acceleration
Hardware acceleration for OpenGL is possible with option `--gpu`. 
 - This will work out of the box in most cases with open source drivers on host. Otherwise have a look at [Dependencies](#dependencies). 
 - x11docker wiki provides some [background information about hardware acceleration for docker containers.](https://github.com/mviereck/x11docker/wiki/Hardware-acceleration)
## Clipboard
Clipboard sharing is possible with option `--clipboard`. 
 - Image clips are possible with `--xpra` and `--hostdisplay`. 
 - Some X server options need package `xclip` on host.
## Sound
Sound is possible with options `--pulseaudio` and `--alsa`. 
(Some background information is given in [x11docker wiki about sound for docker containers.](https://github.com/mviereck/x11docker/wiki/Pulseaudio-sound-over-TCP-or-with-shared-socket)
 - For pulseaudio sound with `--pulseaudio` you need `pulseaudio` on host and in image.
 - For ALSA sound with `--alsa` you can specify the desired sound card with e.g. `--env ALSA_CARD=Generic`. Get a list of available sound cards with `aplay -l`.
## Webcam
Webcams on host can be shared with option `--webcam`.
 - If webcam application in image fails, install `mesa-utils` (debian) or `mesa-demos` (arch) in image. 
 - `cheese` is not recommended. It needs `--systemd` and `--privileged`. Privileged setup is a no-go.
 - `guvcview` needs `--pulseaudio` or `--alsa`.
## Printer
Printers on host can be provided to container with option `--printer`. 
 - It needs CUPS on host, the default printer server for most linux distributions. 
 - The container needs package `libcups2` (debian) or `libcups` (arch).
## Language locales
You have two possibilities to set [language locale](https://wiki.archlinux.org/index.php/locale) in docker image. 
 - For support of chinese, japanese and korean characters install a font like `fonts-arphic-uming` in image.
### Language locale created offhand
x11docker provides option `--lang $LANG` for flexible language locale settings. 
 - x11docker will check on container startup if the desired locale is already present in image and enable it. 
 - If x11docker does not find the locale, it creates it on container startup.
   - Debian images need package `locales`. 
   - x11docker will only look for or create `UTF-8`/`utf8` locales. 
 - Examples: `--lang de` for German, `--lang zh_CN` for Chinese, `--lang ru` for Russian, `--lang $LANG` for your host locale.
### Language locale precompiled in image
You can choose between already installed language locales in image setting environment variable `LANG`, e.g. in image with `ENV LANG=en_US.utf8` or with x11docker option `--env LANG=en_US.utf8`.  
 - Already installed locales in image can be checked with `docker run IMAGENAME locale -a`. 
 - Example to create a language locale in image:
```
RUN apt-get install -y locales
ENV LANG en_US.utf8
RUN localedef --verbose --force -i en_US -f UTF-8 en_US.utf8 || echo "localedef exit code: $?"
```
 
# Troubleshooting
For troubleshooting, run `x11docker` or `x11docker-gui` in a terminal. 
 - x11docker shows warnings if something is insecure, missing or going wrong. 
 - Use option `--verbose` to see logfile output, too.
   - Option `--debug` can provide additional informations.
   - Use options `--stdout --stderr --silent` to get application output only.
   - You can find the latest dispatched logfile at `~/.cache/x11docker/x11docker.log`.
 - Make sure your x11docker version is up to date with `x11docker --update` (latest release) or `x11docker --update-master` (latest beta).
 - Some applications need more privileges or capabilities than x11docker provides as default.
   - Reduce container isolation with options `--hostipc --hostnet --cap-default --sys-admin` and try again. If the application runs, reduce this insecure options to encircle the issue.
   - You can run container application as root with `--user=root`.
 - Get help in the [issue tracker](https://github.com/mviereck/x11docker/issues). 
   - Most times it makes sense to store the `--verbose`output (or `x11docker.log`) at [pastebin](https://pastebin.com/).

# Dependencies
x11docker can run with standard system utilities without additional dependencies on host or in image. As a core it only needs an `X` server and, of course, [`docker`](https://www.docker.com/) to run docker images on X.

x11docker checks dependencies for chosen options on startup and shows terminal messages if some are missing. 

_Basics:_
 - If no additional X server is installed, only less isolated option `--hostdisplay` will work out of the box within X, and option `--xorg` from console. (To use `--xorg` within X, look at [setup for option --xorg](#setup-for-option---xorg)).
 - As a **well working base** for convenience and security, it is recommended to install [`xpra`](http://xpra.org/) (seamless mode) and `Xephyr` (nested desktop mode).
 - Already installed on most systems with an X server: `xrandr`, `xauth` and `xdpyinfo`.
 
_Advanced usage:_
 - **Clipboard** sharing with option `--clipboard` needs `xclip`. (Not needed for `--xpra`, `--nxagent` and `--hostdisplay`). Image clipboard sharing is possible with `--xpra` and `--hostdisplay`.
 - **Sound**:
   - Option `--alsa` has no dependencies. 
     - You can install ALSA libraries in image to support virtual devices (debian images: `libasound2`).
   - Option `--pulseaudio` needs `pulseaudio` on host _and_ in image. 
 - **Hardware acceleration** with option `--gpu`
   - Works best with open source drivers on host and OpenGL/Mesa in image. In most cases everything will work out of the box with just setting `--gpu`.
   - To provide good X isolation: Install `Xwayland` and `weston` for desktop mode, and additionally `xpra` and `xdotool` for seamless mode. Without these, you can still use `--gpu` with `--hostdisplay` and `--xorg`.
   - Packages for OpenGL/Mesa in image:
     - debian and Ubuntu images: `mesa-utils mesa-utils-extra`.
     - CentOS and fedora images: `glx-utils mesa-dri-drivers`
     - Alpine and NixOS images: `mesa-demos mesa-dri-ati mesa-dri-intel mesa-dri-nouveau mesa-dri-swrast`
     - Arch Linux images: `mesa-demos`
   - Proprietary closed source drivers from NVIDIA corporation need some manual setup and have some restrictions. Consider to use free `nouveau` driver instead.
     - x11docker can automatically install closed source nvidia drivers in container at every container startup. It gives some setup instructions in terminal output.
     - The image should contain `modprobe` (package `kmod`) and `xz`. x11docker installs them if they are missing, but that slows down container startup.
     - You need the very same driver version as on host. It must not be a `deb` or `rpm` package but an `NVIDIA_[...].run` file. Store it at one of the following locations:
       - `~/.local/share/x11docker` (current user only)
       - `/usr/local/share/x11docker` (system wide)
     - Look at [NVIDIA driver download page](https://http.download.nvidia.com/) or try the direct download link provided in x11docker terminal output.
     - Closed source driver installation fails on image systems that are not based on `glibc`. This affects especially [Alpine](https://alpinelinux.org/) based images. 
       NVIDIA corporation does not provide the source code that would allow you to use your hardware with different systems.
     - Alternativly, you can install a driver version matching your host setup in image yourself. Note that this image will not be portable anymore.
 
_Rarer needed dependencies for special options:_
 - `--nxagent` needs `nxagent`. Faster and more flexible than `xpra` and `Xephyr`, but less reliable.
 - `--kwin` and `--kwin-xwayland` need `kwin_wayland`, included in modern `kwin` packages.
 - `--xdummy` needs dummy video driver `xserver-xorg-video-dummy` (debian) or `xorg-x11-drv-dummy` (fedora).
 - `--xvfb` needs `Xvfb`
 - `--xfishtank` needs `xfishtank` to show a fish tank.
 - `--dbus` is needed only for QT5 application in Wayland. It needs `dbus` or `dbus-launch` in image.
 - `--starter` needs `xdg-user-dir` to locate your `Desktop` folder for starter icons.
 - `--install`, `--update` and `--remove` need `wget`, `unzip` and `xdg-icon-resource`.
   
_List of all host packages for all possible x11docker options (debian package names):_
 - `xserver-xephyr xpra weston xwayland nxagent kwin xvfb xserver-xorg-video-dummy xauth xclip xdpyinfo xrandr xfishtank xdg-utils xdotool unzip wget`, further (deeper surgery in system): `pulseaudio xserver-xorg-legacy`.

![x11docker-gui dependencies screenshot](/../screenshots/x11docker-dependencies.png?raw=true)

# Password prompt
root permissions are needed only to run docker. X servers run as unprivileged user.

_Running x11docker as unprivileged user:_
 - x11docker checks whether docker needs a password to run and whether `su` or `sudo` are needed to get root privileges. A password prompt appears if needed.
 - If that check fails and does not match your setup, use option `--pw FRONTEND`. `FRONTEND` can be one of `su sudo gksu gksudo lxsu lxsudo kdesu kdesudo beesu pkexec` or `none`.

_Running x11docker as root:_
 - Commands other than `docker` are executed as unprivileged user determined with [`logname`](http://pubs.opengroup.org/onlinepubs/9699919799/utilities/logname.html). (You  can specify another host user with `--hostuser USER`).
 - Unfortunately, some systems do not provide `DISPLAY` and `XAUTHORITY` for root, but needed for nested X servers like Xephyr.    
   - Tools like `gksu` or `gksudo` can help. 
   - Some `sudo` implementations provide `-E` to keep the user environment: `sudo -E x11docker [...]`.

# Security 
Scope of x11docker is to run dockered GUI applications while preserving and improving container isolation.
Core concept is:
   - Run a second X server to avoid [X security leaks](http://www.windowsecurity.com/whitepapers/unix_security/Securing_X_Windows.html).
     - This in opposite to widespread solutions that share host X socket of display :0, thus breaking container isolation, allowing keylogging and remote host control. (x11docker provides this with option `--hostdisplay`).
     - Authentication is done with MIT-MAGIC-COOKIE, stored separate from file `~/.Xauthority`.
   - Create container user similar to host user to [avoid root in container](http://blog.dscpl.com.au/2015/12/don-run-as-root-inside-of-docker.html).
     - You can also specify another user with `--user=USERNAME` or a non-existing one with `--user=UID:GID`.
     - If you want root permissions in container, use option `--sudouser` that allows `su` and `sudo` with password `x11docker`. Alternatively, you can run with `--user=root`. 
   - Disables possible root password and deletes entries in `/etc/sudoers`.
   - Reduce [container capabilities](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities) to bare minimum.
     - Uses docker run options `--cap-drop=ALL --security-opt=no-new-privileges`. 
     - This restriction can be disabled with x11docker option `--cap-default` or reduced with `--sudouser`.

_Weaknesses:_
 - If docker daemon runs with `--selinux-enabled`, SELinux restrictions are degraded for x11docker containers with docker run option `--security-opt label=type:container_runtime_t` to allow access to new X unix socket. A more restrictive solution is desirable.
   Compare: [SELinux and docker: allow access to X unix socket in /tmp/.X11-unix](https://unix.stackexchange.com/questions/386767/selinux-and-docker-allow-access-to-x-unix-socket-in-tmp-x11-unix)
 - User namespace remapping is disabled to allow options `--home` and `--homedir` without file ownership issues. (Though, this is less a problem as x11docker already avoids root in container).

### Options degrading container isolation
x11docker shows warning messages in terminal if chosen options degrade container isolation.

_Most important:_
  - `--hostdisplay` shares host X socket of display :0 instead of running a second X server. 
    - Danger of abuse is reduced providing so-called untrusted cookies, but do not rely on this. 
    - If additionally using `--gpu` or `--clipboard`, option `--hostipc` and trusted cookies are enabled and no protection against X security leaks is left. 
    - If you don't care about container isolation, `x11docker --hostdisplay --gpu` is an insecure, but quite fast setup without any overhead.
  - `--gpu` allows access to GPU hardware. This can be abused to get window content from host ([palinopsia bug](https://hsmr.cc/palinopsia/)) and makes [GPU rootkits](https://github.com/x0r1/jellyfish) possible.
  - `--pulseaudio` and `--alsa` allow catching audio output and microphone input from host.
  
_Rather special options reducing security, but not needed for regular use:_
  - `--sudouser` allows `su` and `sudo` with password `x11docker`for container user. If an application breaks out of container, it can do anything. Allows many container capabilties that x11docker would drop otherwise.
  - `--cap-default` disables x11docker's container security hardening and falls back to default docker container capabilities.
  - `--dbus-system`, `--systemd`, `--sysvinit`, `--openrc` and `--runit` allow some container capabilities that x11docker would drop otherwise. `--systemd` also shares access to `/sys/fs/cgroup`.
  - `--hostipc` sets docker run option `--ipc=host`. (Allows MIT-SHM / shared memory. Disables IPC namespacing.)
  - `--hostnet` sets docker run option `--net=host`. (Shares host network stack. Disables network namespacing. Container can spy on network traffic.)
   
# Choice of X servers and Wayland compositors
If no X server option is specified, x11docker automatically chooses one depending on installed dependencies and on given or missing options `--desktop`, `--gpu` and `--wayland`. 
 - For single applications, x11docker prefers `--xpra`. Alternativly, it tries `--nxagent`.
 - With option `--desktop`, x11docker assumes a desktop environment in image and prefers `--xephyr`. 
 - With option `--gpu` for hardware acceleration, x11docker prefers `--xpra-xwayland` for single applications, or `--weston-xwayland` for desktop environments. 
 - If none of above can be started due to missing dependencies, x11docker uses `--hostdisplay` or `--xorg`.
 - With option `--wayland`, x11docker creates a Wayland environment without X. See also chapter [Wayland](#wayland).
 
 To run an X server entirely in docker, look at [x11docker/xwayland](https://github.com/mviereck/dockerfile-x11docker-xwayland).
 
![x11docker-gui server screenshot](/../screenshots/x11docker-server.png?raw=true)

## Wayland
To run  [Wayland](https://wayland.freedesktop.org/) instead of an X server x11docker provides options `--wayland`, `--weston`, `--kwin` and `--hostwayland`.
 - Option `--wayland` automatically sets up a Wayland environment with some related environment variables.
 - Options `--kwin` and `--weston` run Wayland compositors `kwin_wayland` or `weston`.
   - For QT5 applications without option `--wayland` you need to manually add options `--dbus`  and `--env QT_QPA_PLATFORM=wayland`.
 - Option `--hostwayland` can run single applications on host Wayland desktops like Gnome 3, KDE 5 and [Sway](https://github.com/swaywm/sway).
 - Example: KDE `konsole` terminal on Wayland:
 ```
x11docker --wayland x11docker/plasma konsole
```

## Setup for option --xorg
 - Option `--xorg` runs from console without additional setup. 
 - To run a second core Xorg server from within an already running X session, you have two possibilities:
   - Run x11docker as root, or
   - Edit or create file `/etc/X11/Xwrapper.config` and replace line:
```
allowed_users=console
```
with lines:
```
allowed_users=anybody
needs_root_rights=yes
```
On debian 9 and Ubuntu 16.04 you need to install package `xserver-xorg-legacy`. 
(Depending on your hardware and system setup, you may not need line `needs_root_rights=yes`).

# Init system
x11docker supports init systems as PID 1 in container. Init in container solves the [zombie reaping issue](https://blog.phusion.nl/2015/01/20/docker-and-the-pid-1-zombie-reaping-problem/).

## tini
As default, x11docker uses docker built-in [`tini`](https://github.com/krallin/tini) with docker run option `--init`.
 - You can disable init in container with option `--no-init`. 
 - On some distributions docker's init `/usr/bin/docker-init` is missing in docker package. Compare [#23](https://github.com/mviereck/x11docker/issues/23#issuecomment-386817295). 
   To provide a replacement, download `tini-static` from https://github.com/krallin/tini and store it at one of following locations:
   - `~/local/share/x11docker`
   - `/usr/local/share/x11docker`

Example installation code:
```
mkdir -p ~/.local/share/x11docker
cd ~/.local/share/x11docker
wget https://github.com/krallin/tini/releases/download/v0.18.0/tini-static
chmod +x tini-static
```
      
## systemd, SysVinit, runit, OpenRC
x11docker sets up the init system to run desired command. No special setup is needed beside installing the init system in image. Installing `dbus` in image is recommended.
 - `--systemd`: [systemd](https://wiki.debian.org/systemd) in container.
   - To get a faster startup, it helps to look for services that fail to start in container and to mask them in image with `systemctl mask servicename`.
   - Tested with fedora, debian and Arch Linux images. Debian 10 images run well; debian 9 images additionally need insecure option `--sys-admin`.
   - Image example based on debian buster: [x11docker/cinnamon](https://hub.docker.com/r/x11docker/cinnamon/)
 - `--runit`: [runit](https://wiki.voidlinux.org/Runit) in container.
   - For a bit faster startup, failing services can be disabled by deleting their softlinks in `/etc/runit/runsvdir/default`.
   - Image examples based on [Void Linux](https://www.voidlinux.org/): [x11docker/enlightenment](https://hub.docker.com/r/x11docker/enlightenment/) and [x11docker/lumina](https://hub.docker.com/r/x11docker/lumina/).
 - `--openrc`: [OpenRC](https://wiki.gentoo.org/wiki/OpenRC) in container.
   - cgroup usage possible with option `--sharecgroup`.
   - Image example based on [Alpine Linux](https://alpinelinux.org/): [x11docker/fvwm](https://hub.docker.com/r/x11docker/fvwm/)
 - `--sysvinit`: [SysVinit](https://wiki.archlinux.org/index.php/SysVinit) in container.
   - Tested with [devuan](https://devuan.org/) images from [gitlab/paddy-hack](https://gitlab.com/paddy-hack/devuan/container_registry).

## elogind
x11docker automatically supports `elogind` in container with init system options `--dbus-system`, `--sysvinit`, `--runit` and `--openrc`. Set option `--sharecgroup` to allow `elogind` in container.
 - If your host does not run with `elogind` (but e.g. with `systemd`), x11docker needs an elogind cgroup mountpoint at `/sys/fs/cgroup/elogind`. Run x11docker with root privileges to automatically create it.
 - Same goes for `elogind` on host and `systemd` in container; a cgroup mountpoint for `systemd` must be created. x11docker does this automatically if it runs as root.
 - If you want to manually set up cgroup:
   - Create elogind cgroup mountpoint on a systemd host:
```
mount -o remount,rw cgroup /sys/fs/cgroup  # remove write protection
mkdir -p /sys/fs/cgroup/elogind
mount -t cgroup cgroup /sys/fs/cgroup/elogind -o none,name=elogind
mount -o remount,ro cgroup /sys/fs/cgroup  # restore write protection
```
   - Create a systemd cgroup mountpoint on an elogind host:
```
mkdir -p /sys/fs/cgroup/systemd
mount -t cgroup cgroup /sys/fs/cgroup/systemd -o none,name=systemd
```

## dbus
Some desktop environments and applications need a running dbus daemon and/or dbus user session. 
 - use `--dbus-system` to run dbus system daemon. This includes option `--dbus`.
 - use `--dbus` to run image command with `dbus-launch` (fallback: `dbus-run-session`) for a dbus user session.
 
# MSYS2, Cygwin and WSL on MS Windows
x11docker runs on MS Windows in [MSYS2](https://www.msys2.org/), [Cygwin](https://www.cygwin.com/) 
and [WSL (Windows subsystem for Linux)](https://docs.microsoft.com/en-us/windows/wsl/about).
 - Install X server [`VcXsrv`](https://sourceforge.net/projects/vcxsrv/) on Windows into `C:/Program Files/VcXsrv` (option `--vcxsrv`).
 - Cygwin/X also provides `Xwin` (option `--xwin`). Install `xinit` package in Cygwin.
 - For sound with option `--pulseaudio` install Cygwin in `C:/cygwin64` with package `pulseaudio`. It runs in MSYS2 and WSL, too.
 - Error messages like `./x11docker: line 2: $'\r': command not found` indicate a wrong line ending conversion from git. Run `dos2unix x11docker`.
 
# Examples
Some image examples can be found on docker hub: https://hub.docker.com/u/x11docker/

 - Single GUI application in container: 
   - Terminal: `x11docker x11docker/xfce xfce4-terminal`
   - [Telegram messenger](https://telegram.org/) with persistant `HOME` for configuration storage: 
     - `x11docker --home xorilog/telegram`
   - Fractal generator [XaoS](https://github.com/patrick-nw/xaos): `x11docker patricknw/xaos`
   - GLXgears with hardware acceleration: `x11docker --gpu x11docker/xfce glxgears`
   - Firefox with shared Download folder: `x11docker --hostipc --sharedir $HOME/Downloads jess/firefox` (Option `--hostipc` avoids tab crashes. Better avoid them in `about:config` setting `browser.tabs.remote.autostart` to `false`).
   - Chromium browser: `x11docker -- jess/chromium --no-sandbox`
   - [Tor browser](https://www.torproject.org/projects/torbrowser.html): `x11docker jess/tor-browser`
   - VLC media player with shared Video folder and pulseaudio sound: 
     - `x11docker --pulseaudio --sharedir=$HOME/Videos jess/vlc`
   - [Kodi](https://kodi.tv/): `x11docker --gpu erichough/kodi` For setup look at [ehough/docker-kodi](https://github.com/ehough/docker-kodi).
   
 - Desktop in container: 
   - Minimal images:
     - FVWM: `x11docker --desktop x11docker/fvwm` (based on [alpine](https://alpinelinux.org/), 22.5 MB)
     - fluxbox: `x11docker --desktop x11docker/fluxbox` (based on debian, 87 MB)

   - Lightweight, small image:
     - [Lumina](https://lumina-desktop.org): `x11docker --desktop x11docker/lumina` (based on [Void Linux](https://www.voidlinux.org/))
     - LXDE: `x11docker --desktop x11docker/lxde`
     - LXQt: `x11docker --desktop x11docker/lxqt`
     - Xfce: `x11docker --desktop x11docker/xfce`
     - [CDE Common Desktop Environment](https://en.wikipedia.org/wiki/Common_Desktop_Environment): `x11docker --desktop --hostnet x11docker/cde`
     
   - Medium:
     - Mate: `x11docker --desktop x11docker/mate`
     - Enlightenment: `x11docker --desktop --gpu --runit x11docker/enlightenment` (Based on [Void Linux](https://www.voidlinux.org/))
     - [Trinity](https://www.trinitydesktop.org/) (successor of KDE 3): `x11docker --desktop x11docker/trinity`
     
   - Heavy, option `--gpu` recommended:
     - Cinnamon: `x11docker --desktop --gpu --dbus-system x11docker/cinnamon`
     - [deepin](https://www.deepin.org/en/dde/): `x11docker --desktop --gpu --systemd x11docker/deepin`
     - KDE Plasma: `x11docker --desktop --gpu x11docker/plasma`
     - KDE Plasma as nested Wayland compositor: 
       - `x11docker --hostdisplay --gpu x11docker/plasma startplasmacompositor`
     
   - LXDE desktop with wine and a persistent home folder to preserve installed Windows applications, with pulseaudio sound and hardware acceleration: 
     - `x11docker --desktop --home --pulseaudio --gpu x11docker/lxde-wine`
  
## Screenshots
Sample screenshots can be found in [screenshot branch](https://github.com/mviereck/x11docker/tree/screenshots)

`x11docker --desktop x11docker/lxde-wine`
![screenshot](https://raw.githubusercontent.com/mviereck/x11docker/screenshots/screenshot-lxde-wine.png "LXDE desktop in docker")

`x11docker --desktop --gpu x11docker/plasma`
![screenshot](https://raw.githubusercontent.com/mviereck/x11docker/screenshots/screenshot-plasma.png "KDE plasma desktop in docker")

`x11docker --desktop x11docker/lxqt`
![screenshot](https://raw.githubusercontent.com/mviereck/x11docker/screenshots/screenshot-lxqt.png "LXQT desktop in docker"))

`x11docker --desktop --systemd --gpu x11docker/deepin`
![screenshot](https://raw.githubusercontent.com/mviereck/x11docker/screenshots/screenshot-deepin.png "deepin desktop in docker")

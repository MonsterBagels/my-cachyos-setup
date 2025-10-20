# CachyOS Install
This is meant to collect my notes as I try to move on from Windows on my gaming PC. 

<ins>A few handy links:</ins>
- [CachyOS Wiki](https://wiki.cachyos.org/)
- [CachyOS Forum](https://discuss.cachyos.org/)
- [Linux Gaming Subreddit FAQ](https://www.reddit.com/r/linux_gaming/wiki/faq/)

## Fresh Install

[PCPartPicker parts list](https://pcpartpicker.com/list/zFjmKq)

My PC has two M.2 SSD's, one with windows 11 and one with CachyOS. For the install I used the following options:

**Boot Manager**: I went with **rEFInd** because I don't need BIOS support and wanted something customizable.

**Filesystem**: I went with the default **BTRFS** because it includes snapshot functionality and several other [features](https://itsfoss.com/btrfs/)

**Desktop Environment**: **KDE Plasma** is straight forward and supports Wayland.

I used the [wiki](https://wiki.cachyos.org/configuration/secure_boot_setup/) guide to configure the OS.

## Configuration

### 1. BTRFS Assistant
I installed this first so that I could create a snapshot of a fresh install in case I needed to roll back to it for any reason.

### 2. Secure Boot Setup
This is required so that CachyOS plays nicely with my Windows 11 install.

### 3. Post Install Recommendations
Ran into an issue setting the WiFi region. It turns out that my onboard WiFi & BT adapter [doesn't have a working linux driver](https://www.reddit.com/r/homelab/comments/1iw23f3/anybody_know_if_mediatek_tp_link_7927_wifi_7_is/). WiFi is okay but I need Bluetooth for my controller so I bought a TP-Link UB500 which does have a linux driver. I also enabled global menu, updated tldr, and installed the appimage manager.

### 4. Boot Manager Configuration
Setting up rEFInd is straight forward as it autodetects OS installs. I ended up going with the [refind-theme-regular](https://github.com/bobafetthotmail/refind-theme-regular) theme but others can be found [here](https://refind-themes-collection.netlify.app/). I only had to set the resolution to max in refind.conf and rename the CachyOS and Windows icons so the correct ones were used.

### 5. Gaming
The only other thing I did here beyond what is called out in the wiki is to set the default proton version to native version of cachyos proton.

> Steam > Settings > Compatibility > Default Compatibility tool: proton-cachyos (native package)

### 6. General System Tweaks
I just disabled split lock mitigate.

### 7. Scheduler
I'm using scx_lavd and the gaming profile.

### 8. Peripheral Setup
I mostly stuck with [this YouTube video](https://www.youtube.com/watch?v=uIRs-zh3nGI) for guidance. 

- Keyboard & Mouse
  - I installed the [OpenRazer Driver](https://openrazer.github.io/) and the [Polychromatic front end](https://polychromatic.app/).
  - It doesn't recognize my keyboard or mouse dock so will have to [add them myself](https://github.com/openrazer/openrazer/blob/master/DEVELOPMENT.md) or wait for support to be added to OpenRazer.
- Controller (Sony PlayStation Dualsense)
  - Bluetooth pairing wizard errors during connection but the controller is paired when you test it.
- Streamdeck
  - Works using the [StreamController](https://github.com/StreamController/StreamController) app.

[Gamescope setup video](https://www.youtube.com/watch?v=wcs7JsMLHFY)

[List of games supporting Nvidia DLSS](https://www.nvidia.com/en-us/geforce/news/nvidia-rtx-games-engines-apps/)

### 9. Sound Setup

I like to use virtual sources to individually adjust sound levels while gaming with a podcast, music, or discord playing also. CachyOS has [pipewire](https://wiki.archlinux.org/title/PipeWire) installed so it's not terrible. I created three virtual devices using [this tutorial](https://gitlab.freedesktop.org/pipewire/pipewire/-/wikis/Virtual-Devices#virtual-devices) and named them media, browser, and voice. Each of these need a .conf file in the `/etc/pipewire/pipewire.conf.d/` directory. 

```
context.objects = [
    {   factory = adapter
        args = {
            factory.name     = support.null-audio-sink
            node.name        = "media"
            media.class      = Audio/Sink
            audio.position   = [ FL FR ]
            monitor.channel-volumes = true
            monitor.passthrough = true
            adapter.auto-port-config = {
                mode = dsp
                monitor = true
                position = preserve
            }
        }
    }
]
```

You then assign apps to each virtual device using the sound panel in system settings and then draw connections between the virtual devices and the selected output device, my monitor in this case, using an app called [Cable](https://github.com/magillos/Cable) which is in AUR.

![Cables setup](https://github.com/MonsterBagels/my-cachyos-setup/blob/main/assets/cable_example.png)

<ins> **Launch Option Setup** </ins>

Structure:
```
<env variables> <wrappers> %command% <application arguments>
```

+ `<env variables>` sets hardware options
+ `<wrappers>` are applications or scripts to modify the running application
+ `%command%` will be replaced by steam with the application path
+ `<application arguments>` are used by the application itself

<ins>**Environment Variables**</ins>
+ `PROTON_DLSS_UPGRADE=1`: Upgrades DLSS to the latest version
+ `PROTON_DLSS_INDICATOR=1`: Enable DLSS indicator
+ `MANGOHUD=1`: Enable MangoHUD (possibly Vulkan only, could also try mangohud as a wrapper)

<ins>**Wrappers**</ins>
+ `game-performance`: Enable performance mode while game is active
+ `dlss-swapper`: CachyOS specifric script to force games using DLSS2 or later to use latest preset.
  + If dlss-swapper is not working or causing issues try updating game’s DLSS implementation manually by replacing nvngx_dlss.dll with an up-to-date version and using the dlss-swapper-dll wrapper script instead.
+ `gamescope`: Enable gamescope, use `gamescope --help` in terminal to view options

<ins>Example</ins>
Here's what I'm using in Elden ring.

```
game-performance gamescope -h 2160 -w 3840 -r 240 -b -f --adaptive-sync --mangoapp --hdr-enabled -- %command%
```

+ game-performance - enable performance mode in OS
+ gamescope: enable gamescope
  + -h 2160 -w 3840: sets 4k resolution
  + -r 240: sets fps limit of 240 Hz
  + -b -f: sets borderless fullscreen
  + --adaptive-sync: enable adaptive sync
  + --mangoapp: When using gamescope must enable mango this way
  + --hdr-enabled: enable HDR, must also have it turned on in system settings 

### Sched-ext Tutorial
~~I stuck with scx_bpfland and since I'm mostly gaming  I set the profile to Gaming~~ Reverted profile back to auto.


 
## Other Apps
+ balenaEtcher 
+ Cider - Apple Music player (built from tar)
+ Chromium (CachyOS Package Manager)
+ Discord (cachyOS Package Manager)
+ Github Desktop (Octopi)
+ GNU (cachyOS Package Manager)
+ MangoHUD / GOverlay
  + [Setup video](https://www.youtube.com/watch?v=KSQrfWXHPDs)
+ PLEX Desktop (Snap)
+ Pocket Casts - Podcast Player (Snap)
+ ProtonPlus / ProtonUp-Qt (Octopi)
+ Syncthing - local file syncing to server (Octopi)
+ Thunderbird - Email client (cachyOS Package Manager)
+ VLC (cachyOS Package Manager)

## Useful Commands
**Restart to UEFI**
```
systemctl reboot --firmware-setup
```

**Activating Python Virtual Environments**
```
source .venv/bin/activate.fish
```

**Restart pipewire**
```
systemctl --user restart pipewire pipewire-pulse wireplumber
```

**Information on Command Line Arguments**
```
tldr <argument>
```

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

### BTRFS Assistant
I installed this first so that I could create a snapshot of a fresh install in case I needed to roll back to it for any reason.

### Secure Boot Setup
This is required so that CachyOS plays nicely with my Windows 11 install.

### Post Install Recommendations
Ran into an issue setting the WiFi region. It turns out that my onboard WiFi & BT adapter [doesn't have a working linux driver](https://www.reddit.com/r/homelab/comments/1iw23f3/anybody_know_if_mediatek_tp_link_7927_wifi_7_is/). WiFi is okay but I need Bluetooth for my controller so I bought a TP-Link UB500 which does have a linux driver.

### Boot Manager Configuration
Setting up rEFInd is straight forward as it autodetects OS installs. I ended up going with the [refind-theme-regular](https://github.com/bobafetthotmail/refind-theme-regular) theme but others can be found [here](https://refind-themes-collection.netlify.app/). I only had to set the resolution to max in refind.conf and rename the CachyOS and Windows icons so the correct ones were used.

### Gaming
The only other thing I did here beyond what is called out in the wiki is to set the default proton version to native version of cachyos proton.

> Steam > Settings > Compatibility > Default Compatibility tool: proton-cachyos (native package)

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

### General System Tweaks
I did a couple of the tweaks listed here including changing the AMD 3D V-Cache Optimizer to Driver in the BIOS and then setting it to cache in the OS but running the grep code didn't seem to do anything so not sure. I also disabled split lock mitigate. 

### Sched-ext Tutorial
~~I stuck with scx_bpfland and since I'm mostly gaming  I set the profile to Gaming~~ Reverted profile back to auto.

### Peripheral Setup
I mostly stuck with [this YouTube video](https://www.youtube.com/watch?v=uIRs-zh3nGI) for guidance. 

- Mouse (Razer Cobra Pro)
  - I installed the [OpenRazer Driver](https://openrazer.github.io/) and the [Polychromatic front end](https://polychromatic.app/).
  - It didn't recognize the charging dock but that was the only issue.
- Keyboard (Keychron Q5 MAX)
  - No software necessary.
  - Uses [Launcher](https://launcher.keychron.com) webtool for programming. Wasn't able to get it working on Chromium so will need to program using Windows.
- Headphones (Arctis Nova Pro Wirelss)
  - While not as feature rich as Steelseries' GG software, [Arctis Manager](https://github.com/elegos/Linux-Arctis-Manager) gets the job done, through providing device data and Game/Voice channels for balancing.
- Controller (Sony PlayStation Dualsense)
  - Had difficulty getting the controller to sync with the default Bluetooth manager so I installed [Blueman](https://github.com/blueman-project/blueman) and that worked after a controller reset.
  - Bluetooth manager gives an error saying it isn't connected but controller will connect and work afterwards
- Streamdeck
  - Works using the [StreamController](https://github.com/StreamController/StreamController) app.
 
## Other Apps
+ Cider - Apple Music player (built from tar)
+ Chromium (CachyOS Package Manager)
+ Discord (cachyOS Package Manager)
+ Github Desktop (Octopi)
+ GNU (cachyOS Package Manager)
+ PLEX Desktop (Snap)
+ Pocket Casts - Podcast Player (Snap)
+ ProtonPlus / ProtonUp-Qt (Octopi)
+ VLC (cachyOS Package Manager)

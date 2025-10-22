# CachyOS Install
This is meant to collect my notes as I try to move on from Windows on my gaming PC. 

<ins>A few handy links:</ins>
- [CachyOS Wiki](https://wiki.cachyos.org/)
- [CachyOS Forum](https://discuss.cachyos.org/)
- [Linux Gaming Subreddit FAQ](https://www.reddit.com/r/linux_gaming/wiki/faq/)

## Fresh Install

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
Setting up rEFInd is straight forward as it autodetects OS installs.

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
- Case Lighting
  - Doesn't seem to work in chromium, will need to adjust using Windows partition. 

### 9. Sound Setup

I like to use virtual surround when gaming so I made a virtual device using [this tutorial](https://www.youtube.com/watch?v=tymRFhUiXVQ). Here is the code you paste into the config file, and the dolby atmos wave file I used is in the asset folder.

```
{ name = libpipewire-module-filter-chain
        flags = [ nofail ]
        args = {
            node.description = "Virtual Surround Sink"
            media.name       = "Virtual Surround Sink"
            filter.graph = {
                nodes = [
                    # duplicate inputs
                    { type = builtin label = copy name = copyFL  }
                    { type = builtin label = copy name = copyFR  }
                    { type = builtin label = copy name = copyFC  }
                    { type = builtin label = copy name = copyRL  }
                    { type = builtin label = copy name = copyRR  }
                    { type = builtin label = copy name = copySL  }
                    { type = builtin label = copy name = copySR  }
                    { type = builtin label = copy name = copyLFE }

                    # apply hrir - HeSuVi 14-channel WAV (not the *-.wav variants) (note: */44/* in HeSuVi are the same, but resampled to 44100)
                    { type = builtin label = convolver name = convFL_L config = { filename = "hrir_hesuvi/hrir.wav" channel =  0 } }
                    { type = builtin label = convolver name = convFL_R config = { filename = "hrir_hesuvi/hrir.wav" channel =  1 } }
                    { type = builtin label = convolver name = convSL_L config = { filename = "hrir_hesuvi/hrir.wav" channel =  2 } }
                    { type = builtin label = convolver name = convSL_R config = { filename = "hrir_hesuvi/hrir.wav" channel =  3 } }
                    { type = builtin label = convolver name = convRL_L config = { filename = "hrir_hesuvi/hrir.wav" channel =  4 } }
                    { type = builtin label = convolver name = convRL_R config = { filename = "hrir_hesuvi/hrir.wav" channel =  5 } }
                    { type = builtin label = convolver name = convFC_L config = { filename = "hrir_hesuvi/hrir.wav" channel =  6 } }
                    { type = builtin label = convolver name = convFR_R config = { filename = "hrir_hesuvi/hrir.wav" channel =  7 } }
                    { type = builtin label = convolver name = convFR_L config = { filename = "hrir_hesuvi/hrir.wav" channel =  8 } }
                    { type = builtin label = convolver name = convSR_R config = { filename = "hrir_hesuvi/hrir.wav" channel =  9 } }
                    { type = builtin label = convolver name = convSR_L config = { filename = "hrir_hesuvi/hrir.wav" channel = 10 } }
                    { type = builtin label = convolver name = convRR_R config = { filename = "hrir_hesuvi/hrir.wav" channel = 11 } }
                    { type = builtin label = convolver name = convRR_L config = { filename = "hrir_hesuvi/hrir.wav" channel = 12 } }
                    { type = builtin label = convolver name = convFC_R config = { filename = "hrir_hesuvi/hrir.wav" channel = 13 } }

                    # treat LFE as FC
                    { type = builtin label = convolver name = convLFE_L config = { filename = "hrir_hesuvi/hrir.wav" channel =  6 } }
                    { type = builtin label = convolver name = convLFE_R config = { filename = "hrir_hesuvi/hrir.wav" channel = 13 } }

                    # stereo output
                    { type = builtin label = mixer name = mixL }
                    { type = builtin label = mixer name = mixR }
                ]
                links = [
                    # input
                    { output = "copyFL:Out"  input="convFL_L:In"  }
                    { output = "copyFL:Out"  input="convFL_R:In"  }
                    { output = "copySL:Out"  input="convSL_L:In"  }
                    { output = "copySL:Out"  input="convSL_R:In"  }
                    { output = "copyRL:Out"  input="convRL_L:In"  }
                    { output = "copyRL:Out"  input="convRL_R:In"  }
                    { output = "copyFC:Out"  input="convFC_L:In"  }
                    { output = "copyFR:Out"  input="convFR_R:In"  }
                    { output = "copyFR:Out"  input="convFR_L:In"  }
                    { output = "copySR:Out"  input="convSR_R:In"  }
                    { output = "copySR:Out"  input="convSR_L:In"  }
                    { output = "copyRR:Out"  input="convRR_R:In"  }
                    { output = "copyRR:Out"  input="convRR_L:In"  }
                    { output = "copyFC:Out"  input="convFC_R:In"  }
                    { output = "copyLFE:Out" input="convLFE_L:In" }
                    { output = "copyLFE:Out" input="convLFE_R:In" }

                    # output
                    { output = "convFL_L:Out"  input="mixL:In 1" }
                    { output = "convFL_R:Out"  input="mixR:In 1" }
                    { output = "convSL_L:Out"  input="mixL:In 2" }
                    { output = "convSL_R:Out"  input="mixR:In 2" }
                    { output = "convRL_L:Out"  input="mixL:In 3" }
                    { output = "convRL_R:Out"  input="mixR:In 3" }
                    { output = "convFC_L:Out"  input="mixL:In 4" }
                    { output = "convFC_R:Out"  input="mixR:In 4" }
                    { output = "convFR_R:Out"  input="mixR:In 5" }
                    { output = "convFR_L:Out"  input="mixL:In 5" }
                    { output = "convSR_R:Out"  input="mixR:In 6" }
                    { output = "convSR_L:Out"  input="mixL:In 6" }
                    { output = "convRR_R:Out"  input="mixR:In 7" }
                    { output = "convRR_L:Out"  input="mixL:In 7" }
                    { output = "convLFE_R:Out" input="mixR:In 8" }
                    { output = "convLFE_L:Out" input="mixL:In 8" }
                ]
                inputs  = [ "copyFL:In" "copyFR:In" "copyFC:In" "copyLFE:In" "copyRL:In" "copyRR:In", "copySL:In", "copySR:In" ]
                outputs = [ "mixL:Out" "mixR:Out" ]
            }
            capture.props = {
                node.name      = "effect_input.virtual-surround-7.1-hesuvi"
                media.class    = Audio/Sink
                audio.channels = 8
                audio.position = [ FL FR FC LFE RL RR SL SR ]
            }
            playback.props = {
                node.name      = "effect_output.virtual-surround-7.1-hesuvi"
                node.passive   = true
                audio.channels = 2
                audio.position = [ FL FR ]
            }
        }
    }
```

I like to use virtual sources to individually adjust sound levels while gaming with a podcast, music, or discord playing also. CachyOS has [pipewire](https://wiki.archlinux.org/title/PipeWire) installed so it's not terrible. I created three virtual devices using [this tutorial](https://gitlab.freedesktop.org/pipewire/pipewire/-/wikis/Virtual-Devices#virtual-devices) and named them media, game, and chat. Instead of individual .conf files listed in the tutorial I added the code below to the pipewire.conf file created in the virtual surround setup, specifically in the context.objects area.

```
    {   factory = adapter
        args = {
            factory.name     = support.null-audio-sink
            node.name        = "Media"
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
    {   factory = adapter
        args = {
            factory.name     = support.null-audio-sink
            node.name        = "Chat"
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
    {   factory = adapter
        args = {
            factory.name     = support.null-audio-sink
            node.name        = "Game"
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
```

You then assign apps to each virtual device using the sound panel in system settings and then draw connections between the virtual devices and the selected output device, my monitor in this case, using an app called [Cable](https://github.com/magillos/Cable) which is in AUR. You can make connections from the virtual devices to the hdmi output and then save the profile and set it to autoconnect at start. Individual apps can be set to each virtual device in the desktop sound menu.

![Cables setup](https://github.com/MonsterBagels/my-cachyos-setup/blob/main/assets/cable_example.png)

## Application Installs
+ [balenaEtcher](https://etcher.balena.io/)
+ [Cider](https://taproom.cider.sh/downloads) - Apple Music player
+ [Ungoogled Chromium](https://github.com/ungoogled-software/ungoogled-chromium)
  + CachyOS Hello
+ Discord
+ Github Desktop
+ LACT - GPU Overclocking
  + `sudo systemctl enable --now lactd` to enable at startup
+ MangoHUD / GOverlay
  + [Setup video](https://www.youtube.com/watch?v=KSQrfWXHPDs)
+ OpenRGB
  + To enable probes:
  + ```
    sudo modprobe i2c-dev
    sudo modprobe i2c-piix4
    ```
  + To auto enable probes at boot (AMD):
  + ```
    sudo touch /etc/modules-load.d/i2c.conf
    sudo sh -c 'echo "i2c-dev" >> /etc/modules-load.d/i2c.conf'
    sudo sh -c 'echo "i2c-piix4" >> /etc/modules-load.d/i2c.conf'
    ```
+ PLEX Desktop
  + [Snap](https://snapcraft.io/plex-desktop)
+ Pocket Casts - Podcast Player
  + [Snap](https://snapcraft.io/pocket-casts)
+ Sublime Text - IDE
  + [GitHub Copilot](https://packagecontrol.io/packages/LSP-copilot) installation
+ Syncthing - local file syncing to server
  + `sudo systemctl enable syncthing@darren` to enable syncthing at startup
  + `sudo ufw allow syncthing` to add syncthing to firewall
+ VLC

## Desktop Customization
I went all in on [Catppuccin](https://catppuccin.com/), there are themes for so much available.

## Gaming Setup

### Launch Option Setup

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
+ `MANGOHUD=1`: Enable MangoHUD (if using gamescope, enable it inside of gamescope instead)

<ins>**Wrappers**</ins>
+ `game-performance`: Enable performance mode while game is active
+ `dlss-swapper`: CachyOS specifric script to force games using DLSS2 or later to use latest preset.
  + [List of games supporting Nvidia DLSS](https://www.nvidia.com/en-us/geforce/news/nvidia-rtx-games-engines-apps/)
  + If dlss-swapper is not working or causing issues try updating game’s DLSS implementation manually by replacing nvngx_dlss.dll with an up-to-date version and using the dlss-swapper-dll wrapper script instead.
+ `gamescope`: Enable gamescope, use `gamescope --help` in terminal to view options

### Gamescope
[Gamescope setup video](https://www.youtube.com/watch?v=wcs7JsMLHFY)

[Gamescope command line generator](https://rfrench3.github.io/scopebuddy-gui/web-ui/index.html)

[Using ReShade with Gamescope](https://www.reddit.com/r/linux_gaming/comments/1mha29v/how_do_you_use_reshade_with_gamescope/)
+ [ReShade HDR Shaders](https://github.com/EndlesslyFlowering/ReShade_HDR_shaders)

Use `gamescope --help` in konsole to see a full list of options.

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
**If asked to use yay to install AUR package, use paru instead**

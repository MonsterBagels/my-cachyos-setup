# CachyOS Install
This is meant to collect my notes as I try to move on from Windows on my gaming PC. 

## General links:
- [CachyOS Wiki](https://wiki.cachyos.org/)
- [CachyOS Forum](https://discuss.cachyos.org/)
- [Pacman Tips & Tricks](https://wiki.archlinux.org/title/Pacman/Tips_and_tricks)
- [Linux Gaming Subreddit FAQ](https://www.reddit.com/r/linux_gaming/wiki/faq/)

## Hardware Configuration
- My onboard WiFi & BT adapter [doesn't have a working linux driver](https://www.reddit.com/r/homelab/comments/1iw23f3/anybody_know_if_mediatek_tp_link_7927_wifi_7_is/) so using a TP-Link UB500 for Bluetooth.
- Using [OpenRazer Driver](https://openrazer.github.io/) and [Polychromatic front end](https://polychromatic.app/) for mouse and keyboard.
- Using [StreamController](https://github.com/StreamController/StreamController) for my Stream Deck.
- Using [OpenRGB](https://openrgb.org/) for case lighting.
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


## OS Configuration
- Trying out the scx_lavd scheduler and the gaming profile.

## Applications
- [balenaEtcher](https://etcher.balena.io/)
- [Cider](https://taproom.cider.sh)
- Gparted
- [LACT](https://github.com/ilya-zlobintsev/LACT)
- [PLEX Desktop](https://snapcraft.io/plex-desktop)
- [Pocketcasts](https://snapcraft.io/install/pocket-casts/arch)
- Sublime Text
  - [GitHub Copilot](https://packagecontrol.io/packages/LSP-copilot) installation
- Syncthing - local file syncing to server
  - `sudo systemctl enable syncthing@darren` to enable syncthing at startup
  - `sudo ufw allow syncthing` to add syncthing to firewall
- Virtualbox
  - [CachyOS Specifics](https://discuss.cachyos.org/t/info-on-how-to-install-virtualbox-on-cachyos/2488/13)
  - [Arch Wiki](https://wiki.archlinux.org/title/VirtualBox)

## Gaming
- [List of Environment Variables](https://wiki.cachyos.org/configuration/gaming/#environment-variables)
- [Mangohud Setup Video](https://www.youtube.com/watch?v=KSQrfWXHPDs)

## Useful Commands

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
`
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

**Perform clean uninstall of software package and unused dependencies**
```
sudo pacman -Rns
```

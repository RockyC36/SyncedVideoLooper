![synced video looper](https://github.com/RockyC36/SyncedVideoLooper/assets/72821598/53dbb833-0118-48ff-80c6-cdf1e1544651)
# SyncedVideoLooper
A simple video looper that you can remotely update across the Internet using Syncthing.
***
This is an instruction guide for a "no-frills" video looper that you can remotely update across the Internet using [Syncthing](https://syncthing.net/). I use it to display ads and informational content on some televisions at a local restaurant using a couple of [Raspberry Pi Zero 2W](https://www.raspberrypi.com/products/raspberry-pi-zero-2-w/) SBCs. To update the content, I just drop the new video file into my local Syncthing shared folder and it gets pushed to all of the Pi's automatically.

## Disclaimers
1. These instructions assume that you have general working knowledge of Raspberry Pi SBCs and are comfortable using terminal commands. 
2. The instructions are designed around **one Raspberry Pi per display**. Output to two displays is possible, but I have not researched or tested it.
3. I have only done this on Raspberry Pi Zero 2W SBCs, though there's no reason these instructions shouldn't work on higher-end models.
4. I do this as a hobby, so I can’t provide technical support other than to direct you to official instructions and guides.
5. If you use firewalls, proxies, or need more advanced sync features, be sure to check out the [Syncthing Wiki](https://docs.syncthing.net/users/index.html).
   
## You Will Need:
- A networked computer running Windows, macOS, or Linux
- 1 Raspberry Pi SBC and 1 Micro-SD Card per display
- [Raspberry Pi Imager](https://www.raspberrypi.com/software/) or [Balena Etcher](https://etcher.balena.io/) to write the SD-Card

# Instructions
## 1. Install and configure Syncthing on your local computer
### Using whatever method is appropriate for your OS, install Syncthing on your local computer. 
- On **Windows**, use the [Syncthing Windows Installer](https://github.com/Bill-Stewart/SyncthingWindowsSetup/)
- On **macOS**, use the [Syncthing macOS Installer](https://github.com/syncthing/syncthing-macos), or install via [Homebrew](https://brew.sh)
### Linux Installation
- On **Debian** or a Debian derivative, you can use the four steps listed below under the heading **3. Install Syncthing on the Pi**, or install via [Homebrew](https://brew.sh)
- On other distros, look for Syncthing in your distro’s package manager, or install via [Homebrew](https://brew.sh)
- Syncthing is also available as a [Flatpak](https://flathub.org/apps/search?q=syncthing) bundled with a tray icon app, and there are other tray icon apps for Linux as well, but the tray icon really isn’t necessary.
### Homebrew
If you install Syncthing via Homebrew, be sure to enable the Syncthing service as described in the notes.
### Local Configuration
Once Syncthing is installed, use the [getting started guide](https://docs.syncthing.net/intro/getting-started.html) for a detailed walk-through on how to set it up and share a folder. Your local computer is now ready to sync. Next, we set up the Raspberry Pi.

## 2. Install Raspberry Pi OS and VLC on the Pi
1. Install **Raspberry Pi OS Lite 32-bit** on your Pi's SD card. It's available from within **Raspberry Pi Imager**. If the Imager fails to write the SD-Card successfully, you can [download the ISO](https://www.raspberrypi.com/software/operating-systems/) directly from Raspberry Pi and flash the SD-Card using **Balena Etcher** instead.
2. Insert the SD-Card into the Pi. If your Pi uses Ethernet, connect the LAN cable.
3. Boot the Pi and let it do its thing. The Pi will prompt you to create a user account and connect to Wi-Fi unless you pre-configured settings using RPi Imager.  Once that's done, you should be left at a prompt. 
4. Run `raspi-config` and enable automatic login. Also configure your Pi’s hostname, network, language, or other settings you need. If configuring Wi-Fi with `raspi-config` fails, you can configure Wi-Fi with `nmtui` instead.
5. Update the Pi. `sudo apt update && sudo apt upgrade`. If you're on a Zero 2W, go have a coffee. The Pi may appear to freeze at times, but be patient.
6. Install VLC, which will also install X11 and other dependencies so the Pi can display the video: `sudo apt install vlc`.

## 3. Install Syncthing on the Pi:
1. Add the release PGP keys:
   ```
   sudo mkdir -p /etc/apt/keyrings
   sudo curl -L -o /etc/apt/keyrings/syncthing-archive-keyring.gpg https://syncthing.net/release-key.gpg
   ```
2. Add the "stable" channel to your APT sources. Yes, that whole thing is one command:
 ```
echo "deb [signed-by=/etc/apt/keyrings/syncthing-archive-keyring.gpg] https://apt.syncthing.net/ syncthing stable" | sudo tee /etc/apt/sources.list.d/syncthing.list
```
3. Update and install Syncthing: `sudo apt update` and `sudo apt install syncthing`
4. Configure the Syncthing service, replacing "username" with the name of the Pi user that you set up in Step 2: `sudo systemctl enable syncthing@username.service` and then `sudo systemctl start syncthing@username.service`

## 4. Configure Syncthing for headless access:
Syncthing requires you to configure the local host via a locally generated web page, but since we have no web browser installed on the Pi, we configure Syncthing to allow us to access it from another computer on the LAN.

1. Navigate to `/home/username/.local/state/syncthing`. Edit `config.xml` using your favorite text editor.
2. Change the IP address to `0.0.0.0` to enable remote access to the GUI.
3. Save and close, then `sudo systemctl restart syncthing@username.service`, remembering to replace "username" with the name of the user you created in Step 2.
4. Get your Pi's IP address. You can use `ip addr` or `ifconfig` to display it. 

## 5. Configure Syncthing and perform your first sync
1. View the Pi’s Syncthing web page from your main computer (or any other computer on the LAN) using a browser with the IP address you noted in Step 4 above: `<Pi-ip-address>:8384`
2. Go to **Settings** and give yourself the same username and password as on your main computer.
3. Click on the **Add Remote Device** button. Your Pi's Device ID should already be present in the window. Click on the ID to select it, type in your Pi's name below, then click **Save**. 
4. After a moment, you will see a notification on the Pi's configuration page that your main computer wants to connect. Choose **Add Device**, then click **Save**.
5. On your main computer, copy the video you want to share to the Pi into the shared folder. It should begin syncing momentarily.
## 6. Test play the video on the Pi
On the Pi, use VLC to play the video to confirm operation. Navigate to the shared folder location and run the command to launch VLC: `vlc filename.m4v`

## 7. Move the Pi to the remote location and set up
If you're using Wi-Fi at the remote location, you'll have to bring a keyboard for the Pi so you can reconfigure the network settings at the remote location. Use `raspi-config` to reconfigure the Wi-fi and reboot.

## 8. Configure automatic playback
1. Edit the Pi’s `.bashrc` file and add the command to autoplay the movie at boot to the end of the file: `cvlc /home/username/sharedfoldername/filename.m4v --quiet --loop --no-video-title`. The `cvlc` command removes the playback controls. To mute the audio, add `--noaudio` to the command.
2. Reboot the Pi. The video should play automatically.
### You’re done! Enjoy your video kiosk player!
***
### Playing multiple videos in a row
Multiple file video playback is possible using a playlist in VLC instead of the file name in the command to start VLC. I have not tested this, however, so YMMV. See the [VLC command-line wiki](https://wiki.videolan.org/VLC_command-line_help) for more information.

### Playback on two displays, effects, filters, scaling, and other stuff.
VLC allows for playback on multiple displays is possible, but is outside the scope of this guide. See the [VLC command-line wiki](https://wiki.videolan.org/VLC_command-line_help) for more information.

### Updating the video file
To update the video on the Pi, just overwrite the video file on the host computer’s shared folder. Syncthing will push the updated file to the Pi, and VLC will play the update file on the next loop.

### Syncing Multiple Pi’s
When syncing multiple Pi’s be sure to give each one a unique name when setting it up. You use the exact same configuration procedure as above for each Pi.
To save time when setting up multiple Pi’s you can clone the SD card right after installing VLC, so the only steps required would be to install and configure Syncthing on each Pi you want to use.



<p align="center">
  <img src="https://raw.githubusercontent.com/github/explore/main/topics/fedora/fedora.png" width="100" alt="Fedora logo">
</p>

<h1 align="center">🦦 Fedora Setup Guide</h1>

<p align="center">
  <a href="https://getfedora.org/">
    <img src="https://img.shields.io/badge/Fedora-43+-blue.svg?style=for-the-badge&logo=fedora" alt="Fedora badge">
  </a>
  <a href="">
    <img src="https://img.shields.io/badge/Maintained%3F-no-brightgreen.svg?style=for-the-badge" alt="Maintenance badge">
  </a>
</p>

<p align="center">
  A personal and practical guide for configuring Fedora Linux for gaming 🕹️, productivity 💻, and daily use ☕  
</p>


---

# 👋 Hey there!

Since you’re here, you’ve probably just finished installing **Fedora** and are staring at a fresh desktop thinking, *“What’s next?”* — I’ve been there too. Fedora can use a little fine-tuning after a clean install 😊. That’s why I put together this guide: a collection of everything I learned and gathered from various sources when I made the switch to Fedora.

> **Note:** This isn’t an official guide — just my personal setup notes and preferences. Feel free to skip whatever doesn’t suit you. Just make sure to read everything before copying and pasting commands.

---

### 💡 Quick heads-up about commands:

- `sudo` — run as administrator (you’ll be asked for your password).  
- `-y` flag — automatically answers “yes” to all prompts, so you won’t need to keep pressing Enter.  

⚠️ **Tip:** Copy-paste is convenient, but always read and understand what you’re running first!

---

## 🔥 First Things First

### RPM Fusion - The Good Stuff

Alright — the first thing to know is that Fedora ships with a pretty minimal setup due to legal and licensing restrictions. That’s where RPM Fusion

```bash
# Get the free repository (most stuff you need)
sudo dnf install -y \
  https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm

# Get the nonfree repository (NVIDIA drivers, some codecs)
sudo dnf install -y \
  https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm

# For Terra:
 sudo dnf install --nogpgcheck --repofrompath 'terra,https://repos.fyralabs.com/terra$releasever' terra-release

# Update everything so it all plays nice together
sudo dnf group upgrade core -y
sudo dnf check-update
```
<details>
<summary>🤔 What's the difference between free and nonfree?</summary>

- **Free**: Open-source software that Fedora can legally include and redistribute.
- **Nonfree**: Proprietary software (like NVIDIA drivers or certain codecs) that Fedora can’t ship by default because of licensing restrictions.

</details>

---

### Updates (Boring but Needed)

Yeah, I get it — updates aren’t exactly exciting. But trust me, this should be your first step. Fresh installs almost always come with outdated packages, so it’s best to get everything up to date right away.

```bash
# Update everything
sudo dnf update -y
```
```bash
# If it updated the kernel, reboot
sudo reboot
```

> 💡 **Tip**: Make a habit of running `sudo dnf update` weekly or monthly :).

---

### Firmware Updates

Your hardware likely has newer firmware updates available — and yes, they do matter. Keeping your firmware up to date can improve things like Wi-Fi stability, battery life, and overall system performance.

```bash
# See what can be updated
sudo fwupdmgr get-devices

# Refresh the firmware database
sudo fwupdmgr refresh --force

# Check for updates
sudo fwupdmgr get-updates

# Apply them
sudo fwupdmgr update
```

> ⚠️ **Note**: After updating your firmware, you’ll need to reboot — no way around it. Just go ahead and do it.

---

### Give Your Computer a Name

This one’s purely cosmetic, but it helps make your desktop feel like yours. Choose something you like — have a little fun with it!

```bash
# Replace with whatever you want
sudo hostnamectl set-hostname hungry-beast
```

---

## 📦 Getting More Software

### Flathub Setup

Fedora includes a limited version of Flatpak by default, which doesn’t give you access to most apps. To fix that, enable Flathub — it’s where all the real Flatpak apps live.

```bash
# Remove the Fedora repo
flatpak remote-delete fedora

# Add the real Flathub
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo

# Update everything
flatpak update --appstream
```

---

## 🎮 Graphics Drivers

### NVIDIA (The Tricky One)

> ⚠️ **WARNING**: NVIDIA on Linux is… well, complicated. For now, this method works most of the time — but if you run into issues, don’t worry, you’re definitely not alone. Welcome to the club.

#### 📋 Before you start:

- Disable Secure Boot in your BIOS (or, if you’re feeling adventurous, learn to sign kernel modules yourself 😉).
- Ensure your system is fully updated and rebooted.
- Have a backup plan — seriously, I’m not responsible if things go sideways.

```bash
# Install kernel headers and dev tools
sudo dnf install -y kernel-devel kernel-headers gcc make dkms acpid \
  libglvnd-glx libglvnd-opengl libglvnd-devel pkgconfig

# Enable RPM Fusion (if not already done)
sudo dnf install -y \
  https://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm \
  https://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
```

> 📌 **Special Note for RTX 4000 and Newer Series:**
> 
> If you have an NVIDIA 4000-series (or newer) GPU, Fedora requires a build macro to be set before installing the driver. This ensures the open-source kernel module is properly enabled.

```bash
# Set open kernel module macro (one-time step for RTX 4000+)
sudo sh -c 'echo "%_with_kmod_nvidia_open 1" > /etc/rpm/macros.nvidia-kmod'
```

#### Install the NVIDIA driver:

```bash
sudo dnf install -y akmod-nvidia xorg-x11-drv-nvidia-cuda
```

Now just wait. Seriously — building the module can take anywhere from 5 to 15 minutes, depending on your system.
You can monitor progress with:

```bash
sudo journalctl -f -u akmods
```

Once done:

```bash
sudo reboot
```

#### ✅ Check if it worked:

```bash
nvidia-smi
```

You should now see your GPU information. If it doesn’t show up, try the following:

```bash
# Force rebuild and try again
sudo akmods --force --kernels $(uname -r)
sudo reboot
```

<details>
<summary>🆘 Help! I'm stuck at 800×600 / black screen / terminal!</summary>

If your NVIDIA module didn’t build correctly, here’s how to fix it:

1. **Boot into an older kernel**:
   - At the GRUB menu, choose “Advanced Options”
   - Select the previous kernel version to boot into it
   
2. **Force rebuild**:
   ```bash
   sudo akmods --kernels $(uname -r) --rebuild
   sudo reboot
   ```

3. **Still broken?** Check the logs:
   ```bash
   sudo journalctl -u akmods
   dmesg | grep -i nvidia
   ```

</details>

> 💡 **Tip**: After kernel updates — especially if you have a newer GPU — you may need to manually rebuild the NVIDIA module:
> 
> ```bash
> sudo akmods --kernels $(uname -r) --rebuild
> sudo reboot
> ```

---

### AMD & Intel (The Easy Ones)

These usually work out of the box, but let’s tweak them to work even better.

#### Both AMD & Intel:

```bash
# Basic drivers and Vulkan support
sudo dnf install -y mesa-dri-drivers mesa-vulkan-drivers vulkan-loader mesa-libGLU
```

#### AMD Only:

```bash
# AMD video acceleration (makes videos smoother)
sudo dnf install -y mesa-va-drivers-freeworld mesa-vdpau-drivers-freeworld
```

#### Intel (Newer GPUs):

```bash
# Intel video acceleration (for newer Intel GPUs)
sudo dnf install -y intel-media-driver
```

#### Intel (Older GPUs):

```bash
# Intel video acceleration (for Grandfather Intel GPUs)
sudo dnf install -y libva-intel-driver
```

> ✅ **That's it!** AMD and Intel GPUs are generally plug-and-play — no extra setup needed.

---

## 🎵 Making Media Work

### Video Codecs (So Everything Plays)

Fedora comes with almost no media codecs by default due to patent and licensing restrictions. This will fix that and let you play all your media.

```bash
sudo dnf4 group install multimedia
sudo dnf swap 'ffmpeg-free' 'ffmpeg' --allowerasing # Switch to full FFMPEG.
sudo dnf upgrade @multimedia --setopt="install_weak_deps=False" --exclude=PackageKit-gstreamer-plugin # Installs gstreamer components. Required if you use Gnome Videos and other dependent applications.
sudo dnf group install -y sound-and-video # Installs useful Sound and Video complementary packages.
```

---

### Hardware Acceleration

This enables hardware acceleration, so video playback uses your GPU instead of putting all the load on your CPU.

```bash
# Install VA-API stuff
sudo dnf install -y ffmpeg-libs libva libva-utils
```

**NVIDIA users, add this too:**

```bash
sudo dnf install -y libva-nvidia-driver
```

---

### Firefox Video Fix

Firefox needs a small tweak to properly handle H.264 videos.

```bash
# Install the Cisco codec (it's free but weird licensing)
sudo dnf install -y openh264 gstreamer1-plugin-openh264 mozilla-openh264

# Enable the Cisco repo
sudo dnf config-manager --set-enabled fedora-cisco-openh264
sudo dnf update -y
```

> ⚠️ **Important**: Restart Firefox, then go to about:addons and make sure the OpenH264 plugin is enabled.

---

## 🔧 Useful Stuff

### Archive Support

Because sooner or later, you’re going to need to extract a .rar file`.

```bash
sudo dnf install -y p7zip p7zip-plugins unrar btop
```

---

### Microsoft Fonts (Unfortunately Still Needed)

Without these, web pages and documents can look awkward or broken, so it’s worth installing them.

```bash
# Install dependencies
sudo dnf install -y curl cabextract xorg-x11-font-utils fontconfig

# Install the fonts
sudo rpm -i --nodigest --nosignature https://downloads.sourceforge.net/project/mscorefonts2/rpms/msttcore-fonts-installer-2.6-1.noarch.rpm

# Update font cache
sudo fc-cache -fv
```

> Yep, we still need Arial and Times New Roman — blame Microsoft 😉.

---

### AppImage Support

Some applications are only distributed as AppImages. This step ensures they run smoothly on your system.

```bash
# Install FUSE
sudo dnf install -y fuse fuse-libs
```
Optional: AppImage Manager — it’s actually pretty handy for organizing and running AppImages.
```bash
flatpak install -y flathub it.mijorus.gearlever
```

---

### Default Firefox start page.

The tweak below will make the start page the default firefox start page instead of [this](https://fedoraproject.org/start)

```bash
sudo rm -f /usr/lib64/firefox/browser/defaults/preferences/firefox-redhat-default-prefs.js
```

---

## Terminal setup

#### Terminal Theme

For me, it’s just better and fancyer to use Starship.

```bash
curl -sS https://starship.rs/install.sh | sh
```

---

#### Terminal shell

I like to use Fish shell.

```bash
sudo dnf install fish -y
```

So lets make it default shell.

```bash
chsh --shell /bin/fish
```

---


## ⚡ Making Things Faster

### Faster Boots

This one’s simple to do and gives a noticeable improvement.

```bash
sudo systemctl disable NetworkManager-wait-online.service
```

> 💡 **Why?** This service waits for network connections during boot, but for most users, it just adds unnecessary delay.

---

### Dual Boot Time Fix

If you’re dual-booting with Windows, this fixes the wrong system clock issue.

```bash
# Tell Fedora to use UTC for hardware clock
sudo timedatectl set-local-rtc 0 --adjust-system-clock
```

> 🤓 **Explanation**: Windows sets the hardware clock to local time, while Linux expects UTC. This tweak makes Linux play nicely with Windows.

---

## 💾 Backup Your Stuff

### System Snapshots

If you’re using Btrfs (the default on newer Fedora), you can create snapshots easily with Btrfs Assistant.

```bash
sudo dnf install -y btrfs-assistant btrbk snapper
```

#### Setup:

Run `btrfs-assistant` to set up snapshots through a user-friendly GUI.

#### Enable automatic snapshots:

```bash
sudo systemctl enable --now snapper-timeline.timer
sudo systemctl enable --now snapper-cleanup.timer
```

>  ⚠️ **Important**: By default, this only backs up system files — your personal files aren’t included unless you adjust the settings.

---

### Personal Files

Use Déjà Dup for your documents, photos, etc.

From fedora repos
```bash
# Install it
sudo dnf install -y deja-dup
```

Or get it from Flathub
```bash
flatpak install -y flathub org.gnome.DejaDup
```

> 💡 **Tip**: Always back up to external storage or the cloud, never to the same disk.

---

## 🎮 Gaming Setup

### Steam and Gaming

[Steam](https://store.steampowered.com/) works great on Fedora thanks to Proton.

```bash
sudo dnf install -y steam
```

And that’s it — seriously. Most Windows games should just work now.

> 📌 **Note**: If you have a newer NVIDIA GPU and Steam won’t launch or behaves oddly, it’s likely due to missing dependencies. The fix: remove the RPM version and install the Flatpak instead:

```bash
# Remove Steam RPM version 
sudo dnf remove steam

# Install Steam Flatpak version 
flatpak install -y flathub com.valvesoftware.Steam
```
### Lutris

[Lutris](https://lutris.net/) is perfect for managing non-Steam games, emulators, and older titles.

```bash
sudo dnf install -y lutris
```

Or get the Flatpak version:

```bash
flatpak install -y flathub net.lutris.Lutris
```

### MangoHud

[MangoHud](https://github.com/flightlessmango/MangoHud) is an overlay that shows FPS, CPU/GPU temps, and more.

```bash
sudo dnf install -y mangohud
```

---

## 🖥️ Desktop Environment

### GNOME

#### Stop GNOME Software autostart

This app launches at every boot and just sits in memory, eating resources. I always remove it:

```bash
sudo rm /etc/xdg/autostart/org.gnome.Software.desktop
```

>   ⚠️ **Note**: If you remove GNOME Software from autostart, update notifications will be disabled. You’ll need to either check for updates manually or rely on the auto-update setup described earlier.

---

#### Get GNOME Tweaks

You literally can’t use GNOME properly without this. Want to add minimize/maximize buttons? This is where you do it:

```bash
sudo dnf install -y gnome-tweaks
```

---

#### Extra themes (if you're into that)

I stick with the default, but some people like to tweak these settings — your call.

```bash
sudo dnf install -y gnome-themes-extra
```

---

#### Extension Manager is essential

Don’t even think about skipping this one — it makes managing GNOME extensions actually easy:

```bash
flatpak install -y flathub com.mattjakeman.ExtensionManager
```

---

#### My must-have extensions:

>  💡 **Disclaimer**: It’s just my personal preference — do what works for you :)

- **Auto Accent Colour** - Automatically set the GNOME accent colour based on the user's background.
- **Bluetooth Quick Connect** - This extension allows paired Bluetooth devices to be connected and disconnected via the GNOME system menu, Shows battery status and more.
- **Blur my Shell** - Adds a blur look to different parts of the GNOME Shell, including the top panel, dash and overview.
- **Clipboard Indicator** - The most popular clipboard manager for GNOME, with over 2M downloads. Check the Github page for a full list of features.
- **Compiz windows effect** - Compiz wobbly windows effect thanks to compiz plugin engine.
- **Dash to Dock** - A dock for the Gnome Shell.
- **Emoji Copy** - Emoji copy is a versatile extension designed to simplify emoji selection and clipboard management.
- **Just Perfection** - Tweak Tool to Customize GNOME Shell, Change the Behavior and Disable UI Elements.
- **Media Controls** - Show controls and information of the currently playing media in the panel.

---

## 🧹 Keeping Things Clean

### System Cleanup

Do this occasionally to free up space.

```bash
# Clean package cache
sudo dnf clean all

# Remove orphaned packages
sudo dnf autoremove -y
```

---

## 👏 Thanks

This guide exists because I got frustrated figuring everything out on my own, and I want to help others get started with Linux too.

---

<div align="center">

### ⚠️ Disclaimer

This isn’t official Fedora documentation — it’s just what works for me. Your results may vary. Always back up important data before making major changes, and don’t blame me if something breaks (though you’re welcome to ask for help).

---

**Have a nice day ;)**

</div>

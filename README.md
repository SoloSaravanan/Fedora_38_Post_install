# Fedora_38_Post_install

# Configure dnf

```sh
sudo nano /etc/dnf/dnf.conf
```

Paste below

```sh
max_parallel_downloads=10 
countme=false
installonly_limit=2
```

# Update system and reboot

```sh
sudo dnf upgrade -y
```

# Setting umask to 077

```sh
umask 077
sudo sed -i 's/umask 002/umask 077/g' /etc/bashrc
sudo sed -i 's/umask 022/umask 077/g' /etc/bashrc
```

# Debloat some apps

```sh
sudo dnf remove akregator mediawriter kmag kgpg qt5-qdbusviewer kcharselect kcolorchooser dragon kmines kmahjongg kpat kruler kmousetool kmouth konversation krdc kfind kaddressbook kmail kontact korganizer ktnef libreoffice-* kf5-akonadi-* adcli anaconda* dmidecode bluez-cu ps hyperv* alsa-sof-firmware virtualbox-guest-additions fedora-bookmarks mailcap open-vm-tools rsync samba-client libertas-usb8388-firmware linux-firmware xorg-x11-drv-vmware sos kpartx dos2unix sssd  spice-vdagent perl-IO-Socket-SSL openrgb inkscape zram-generator plasma-vault plasma-browser-integration plasma-thunderbolt cyrus-sasl-plain traceroute mtr realmd teamd tcpdump nmap-ncat geoclue2 openconnect openvpn ppp pptp
```

# Run Updates

```sh
sudo dnf autoremove -y
sudo fwupdmgr get-devices
sudo fwupdmgr refresh --force
sudo fwupdmgr get-updates -y
sudo fwupdmgr update -y
```

# Setup RPMFusion 

```sh
sudo dnf install -y https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm 
sudo dnf groupupdate core -y
```

# Disable Baloo

```sh
balooctl disable
balooctl purge
sudo rm -rf .local/share/baloo/
```

# Disable Krunner

```sh
sudo chmod -x /usr/bin/krunner
```

# Disbale IPV6

```sh
sudo nano /etc/sysctl.conf
```

Paste below

```sh
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
```

# Disable rsyslog

```sh
sudo service rsyslog stop
sudo systemctl disable rsyslog
```

# Disable hibernate

```sh
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
```

# Install KVM

```sh
sudo dnf install @virtualization
sudo sed -i 's/#unix_sock_group = "libvirt"/unix_sock_group = "libvirt"/g' /etc/libvirt/libvirtd.conf
sudo sed -i 's/#unix_sock_rw_perms = "0770"/unix_sock_rw_perms = "0770"/g' /etc/libvirt/libvirtd.conf
sudo systemctl enable libvirtd
sudo usermod -aG libvirt $(whoami)
```

# Harden the Kernel with Kicksecure's patches

```sh
sudo curl https://raw.githubusercontent.com/Kicksecure/security-misc/master/etc/modprobe.d/30_security-misc.conf -o /etc/modprobe.d/30_security-misc.conf
sudo curl https://raw.githubusercontent.com/Kicksecure/security-misc/master/etc/sysctl.d/30_security-misc.conf -o /etc/sysctl.d/30_security-misc.conf
sudo curl https://raw.githubusercontent.com/Kicksecure/security-misc/master/etc/sysctl.d/30_silent-kernel-printk.conf -o /etc/sysctl.d/30_silent-kernel-printk.conf 
```

# Enable bluetooth from Kicksecure's patches

```sh
sudo sed -i 's/install bluetooth/#install bluetooth/g' /etc/modprobe.d/30_security-misc.conf
sudo sed -i 's/install btusb/#install btusb/g' /etc/modprobe.d/30_security-misc.conf
```

 # Enable Kicksecure CPU mitigations 

```sh
sudo curl https://raw.githubusercontent.com/Kicksecure/security-misc/master/etc/default/grub.d/40_cpu_mitigations.cfg -o /etc/grub.d/40_cpu_mitigations.cfg
sudo curl https://raw.githubusercontent.com/Kicksecure/security-misc/master/etc/default/grub.d/40_distrust_cpu.cfg -o /etc/grub.d/40_distrust_cpu.cfg
sudo curl https://raw.githubusercontent.com/Kicksecure/security-misc/master/etc/default/grub.d/40_enable_iommu.cfg -o /etc/grub.d/40_enable_iommu.cfg 
```

# Divested's brace patches

```sh
sudo mkdir -p /etc/systemd/system/NetworkManager.service.d
sudo curl https://gitlab.com/divested/brace/-/raw/master/brace/usr/lib/systemd/system/NetworkManager.service.d/99-brace.conf -o /etc/systemd/system/NetworkManager.service.d/99-brace.conf
sudo mkdir -p /etc/systemd/system/irqbalance.service.d
sudo curl https://gitlab.com/divested/brace/-/raw/master/brace/usr/lib/systemd/system/irqbalance.service.d/99-brace.conf -o /etc/systemd/system/irqbalance.service.d/99-brace.conf 
```

# NTS instead of NTP

```sh
sudo curl https://raw.githubusercontent.com/GrapheneOS/infrastructure/main/chrony.conf -o /etc/chrony.conf
```

# Setup Firewalld's Rules

```sh
sudo firewall-cmd --permanent --remove-port=1025-65535/udp
sudo firewall-cmd --permanent --remove-port=1025-65535/tcp
sudo firewall-cmd --permanent --remove-service=mdns
sudo firewall-cmd --permanent --remove-service=ssh
sudo firewall-cmd --permanent --remove-service=samba-client
sudo firewall-cmd --permanent --add-port=1714-1764/udp
sudo firewall-cmd --permanent --add-port=1714-1764/tcp
sudo firewall-cmd --reload
```

# Randomize MAC address and disable static hostname

```sh
sudo nano /etc/NetworkManager/conf.d/00-macrandomize.conf
```

Paste below

```sh
[main]
hostname-mode=none

[device]
wifi.scan-rand-mac-address=yes

[connection]
wifi.cloned-mac-address=random
ethernet.cloned-mac-address=random
```

```sh
sudo systemctl restart NetworkManager
sudo hostnamectl hostname "localhost"
```

# Install Multimedia Codecs

```sh
sudo dnf install -y libavcodec-free libavdevice-free libavfilter-free libavutil-free libavformat-free libpostproc-free libswscale-free libswresample-free --allowerasing
rpm -e --nodeps libavcodec-free libavdevice-free libavfilter-free libavutil-free libavformat-free libpostproc-free libswscale-free libswresample-free
```

```sh
sudo dnf swap mesa-va-drivers-freeworld mesa-va-drivers
sudo dnf swap mesa-vdpau-drivers-freeworld mesa-vdpau-drivers
```

```sh
sudo dnf install -y faad2-libs gpac-libs gstreamer1-plugins-bad-freeworld gstreamer1-plugins-ugly gstreamer1-plugin-libav libavdevice libdca libde265 libfreeaptx libheif libftl librtmp live555 mjpegtools-libs mlt mlt-freeworld opencore-amr pipewire-codec-aptx svt-hevc-libs vo-amrwbenc libmediainfo mediainfo x264 x264-libs x265 x265-libs xvidcore vlc intel-media-driver libva-intel-driver nvidia-vaapi-driver mesa-va-drivers-freeworld mesa-vdpau-drivers-freeworld ffmpeg ffmpeg-libs compat-ffmpeg4 ffmpegthumbs --best --allowerasing
```

```sh
pkcon refresh force
pkcon update -y rpmfusion-nonfree-release fedora-repos
pkcon refresh force
pkcon install -y faad2-libs gpac-libs gstreamer1-plugins-bad-freeworld gstreamer1-plugins-ugly gstreamer1-plugin-libav libavdevice libdca libde265 libfreeaptx libheif
pkcon install -y libftl librtmp live555 mjpegtools-libs mlt mlt-freeworld opencore-amr pipewire-codec-aptx svt-hevc-libs vo-amrwbenc libmediainfo mediainfo
pkcon install -y x264 x264-libs x265 x265-libs xvidcore vlc
pkcon install -y faad2-libs gstreamer1-plugins-bad-freeworld gstreamer1-plugins-ugly gstreamer1-plugin-libav libdca libde265 librtmp mjpegtools-libs --filter ~arch
pkcon install -y opencore-amr vo-amrwbenc x264-libs x265-libs --filter ~arch
pkcon install -y intel-media-driver libva-intel-driver nvidia-vaapi-driver mesa-va-drivers-freeworld mesa-vdpau-drivers-freeworld
pkcon install -y ffmpeg
pkcon install -y ffmpeg-libs compat-ffmpeg4
pkcon install -y ffmpeg-libs --filter ~arch
pkcon install -y ffmpegthumbs
```

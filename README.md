# Fedora_40_Post_install

# Configure dnf

```sh
sudo sed -i '8 amax_parallel_downloads=10 ' /etc/dnf/dnf.conf
sudo sed -i '9 acountme=false' /etc/dnf/dnf.conf
```

# Update system and reboot

```sh
sudo dnf upgrade -y
sudo dnf autoremove -y
sudo fwupdmgr get-devices
sudo fwupdmgr refresh --force
sudo fwupdmgr get-updates -y
sudo fwupdmgr update -y
```

# Debloat some apps

```sh
sudo dnf remove dragon k3b kaddressbook kaddressbook-libs kdeaccessibility* kdepim-addons kdepim-runtime kdepim-runtime-libs kdiagram kf5-akonadi-calendar kf5-akonadi-mime kf5-akonadi-notes kf5-akonadi-search kf5-calendarsupport kf5-eventviews kf5-incidenceeditor kf5-kcalendarcore kf5-kcalendarutils kf5-kdav kf5-kidentitymanagement kf5-kimap kf5-kitinerary kf5-kldap kf5-kmailtransport kf5-kmailtransport-akonadi kf5-kmbox kf5-kontactinterface kf5-kpimtextedit kf5-kpkpass kf5-kross-core kf5-ksmtp kf5-ktnef kf5-libgravatar kf5-libkdepim kf5-libkleo kf5-libksane kf5-libksieve kf5-mailcommon kf5-mailimporter kf5-mailimporter-akonadi kf5-messagelib kf5-pimcommon kf5-pimcommon-akonadi kio-gdrive kipi-plugins kmahjongg kmail kmail-account-wizard kmail-libs kmines kolourpaint kolourpaint-libs kontact kontact-libs konversation korganizer korganizer-libs kpat krdc krdc-libs krfb krfb-libs krusader ktorrent kwrite libkdegames libkgapi libkmahjongg libkmahjongg-data libkolabxml libphonenumber libwinpr mpage pim-data-exporter pim-data-exporter-libs pim-sieve-editor plasma-welcome qgpgme qt5-qtwebengine-freeworld qtkeychain-qt5 scim* system-config-printer system-config-services system-config-users unoconv xsane xsane-gimp abrt-desktop akregator mediawriter kmag kgpg qt5-qdbusviewer kcharselect kcolorchooser kruler kmousetool kmouth kfind libreoffice-* kf5-akonadi-* adcli anaconda* dmidecode bluez-cups hyperv* virtualbox-guest-additions fedora-bookmarks mailcap open-vm-tools rsync samba-client libertas-usb8388-firmware xorg-x11-drv-vmware sos kpartx dos2unix sssd  spice-vdagent openrgb inkscape zram-generator plasma-vault plasma-browser-integration plasma-thunderbolt cyrus-sasl-plain traceroute mtr realmd teamd tcpdump nmap-ncat openconnect openvpn ppp pptp
```

# Setup RPMFusion 

```sh
sudo dnf install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
sudo dnf groupupdate core -y
```

# Install Multimedia Codecs

# AMD

```sh
sudo dnf swap mesa-va-drivers mesa-va-drivers-freeworld
sudo dnf swap mesa-vdpau-drivers mesa-vdpau-drivers-freeworld
```

# Intel

```sh
sudo dnf swap libva-intel-media-driver intel-media-driver --allowerasing
```

```sh
sudo dnf groupupdate 'core' 'multimedia' 'sound-and-video' --exclude=zram-generator-defaults --setopt='install_weak_deps=False' --exclude='PackageKit-gstreamer-plugin' --allowerasing && sync 
sudo dnf swap 'ffmpeg-free' 'ffmpeg' --allowerasing
sudo dnf config-manager --set-enabled fedora-cisco-openh264
sudo dnf install gstreamer1-plugins-{bad-\*,good-\*,base} openh264 mozilla-openh264 gstreamer1-plugin-openh264 gstreamer1-libav --exclude=gstreamer1-plugins-bad-free-devel ffmpeg ffmpeg-libs libva libva-utils gstreamer-ffmpeg --exclude=gstreamer1-plugins-bad-free-opencv
sudo dnf install libva-nvidia-driver.{i686,x86_64}
sudo dnf install lame\* --exclude=lame-devel
sudo dnf group upgrade --with-optional Multimedia
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

# Disbale IPV6 and tweak some vm

```sh
sudo nano /etc/sysctl.conf
```

Paste below

```sh
kernel.printk = 0 0 0 0
vm.dirty_bytes=50331648
vm.dirty_background_bytes=16777216
vm.dirty_background_ratio=0
vm.dirty_ratio=0
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

# Setting umask to 077

```sh
umask 077
sudo sed -i 's/umask 002/umask 077/g' /etc/bashrc
sudo sed -i 's/umask 022/umask 077/g' /etc/bashrc
```

# Install KVM

```sh
sudo dnf install @virtualization
sudo systemctl start libvirtd
sudo systemctl enable libvirtd
sudo sed -i 's/#unix_sock_group = "libvirt"/unix_sock_group = "libvirt"/g' /etc/libvirt/libvirtd.conf
sudo sed -i 's/#unix_sock_rw_perms = "0770"/unix_sock_rw_perms = "0770"/g' /etc/libvirt/libvirtd.conf
sudo usermod -a -G libvirt $(whoami)
```

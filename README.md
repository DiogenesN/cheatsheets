# 🐧 Linux CheatSheet

A personal collection of useful Linux commands, fixes, and tutorials.

## 📚 The Wiki Page
➡️ https://github.com/DiogenesN/cheatsheets/wiki/%23-%F0%9F%90%A7-Linux-CheatSheet

---

## 📑 Table of Contents
- 🛠️ [Linux System Fixes](#linux-system-fixes)
- 📘 [Tutorials](#tutorials)
- 🔤 [Text Processing](#text-processing-sed-awk-and-others)
- 🧩 [Miscellaneous](#miscellaneous)

---

## Linux System Fixes
Commands and quick fixes for common Linux issues.

```bash
# Fix broken packages (Debian/Ubuntu)
	sudo apt --fix-broken install

# Reconfigure dpkg
	sudo dpkg --configure -a

# Fix ownership/permissions
	sudo chown -R $USER:$USER /path/to/directory

# Reset file permissions (basic example)
	chmod -R 755 /path/to/directory

# IF the system is unstable and has some misfunctions, you can try the ACPI fix:
	sudo strings /sys/firmware/acpi/tables/DSDT | grep -i windows
	output:
		...
		Windows 2006
		Windows 2009
		Windows 2012

	pick the latest version from the list
	then add this line to the kernel parameters lile this:

	sudo nano /etc/default/grub

	modify this line:
	GRUB_CMDLINE_LINUX="acpi_osi=! acpi_osi=\"Windows 2012\""

	sudo update-grub

# Recover grub/efi with chroot
	sudo mkdir -p /mnt/boot/efi
	sudo mount /dev/nvme0n1p2 /mnt/
	sudo mount /dev/nvme0n1p1 /mnt/boot/efi
	sudo mount -t proc none /mnt/proc
	sudo mount -o bind /sys /mnt/sys
	sudo mount -o bind /run /mnt/run
	sudo mount --rbind /dev/ /mnt/dev/
	sudo mount --rbind /sys/firmware/efi/efivars/ /mnt/sys/firmware/efi/efivars

	chroot /mnt

	export LC_ALL="C"

	(optionalt if you need internet connection)
	sudo nano /etc/resolv.conf
	nameserver 8.8.8.8
	nameserver 8.8.4.4

	sudo grub-install /dev/nvme0n1

	exit

	sudo umount /mnt/boot/efi && \
	sudo umount /mnt/proc && \
	sudo umount /mnt/sys && \
	sudo umount /mnt/run && \
	sudo umount /mnt/sys/firmware/efi/efivars

# Updating to a new kernel no longer updates kernel-headers
# that's why the NVIDIA modules are not being built
# here is how to fix it with dkms
	Automatically:
	1) get the kernel version:

	dpkg -l | grep linux-image
	...
	ii  linux-image-6.12.57+deb13-amd64
	ii  linux-image-6.12.63+deb13-amd64
	ii  linux-image-6.12.69+deb13-amd64
	ii  linux-image-amd64

	2) install headers:
	sudo apt install linux-headers-6.12.69+deb13-amd64

	3) install
	sudo dkms autoinstall -k 6.12.69+deb13-amd64

	4) verify
	sudo dkms status

	you should see this:
		nvidia/current-550.163.01, 6.12.69+deb13-amd64, x86_64: installed


	Manually:
	1) build:
	sudo dkms build -m nvidia -v current-550.163.01 -k 6.12.69+deb13-amd64

	3) install:

	sudo dkms install -m nvidia -v current-550.163.01 -k 6.12.69+deb13-amd64

# Fix ethernet if not working
	sudo mv /usr/lib/NetworkManager/conf.d/10-globally-managed-devices.conf  /usr/lib/NetworkManager/conf.d/10-globally-managed-devices.conf_orig

	sudo touch /usr/lib/NetworkManager/conf.d/10-globally-managed-devices.conf

	sudo service network-manager restart

# My HP notebook is randomely shutting down and this is the fix
	sudo nano /etc/default/grub

	added the following parameters:
	GRUB_CMDLINE_LINUX_DEFAULT="processor.max_cstate=0 intel_idle.max_cstate=2 ..."

	sudo update-grub

# Fix raven AMD GPU firmware
	the error was:
	Jul 19 20:46:59 home kernel: amdgpu 0000:08:00.0: firmware: failed to load amdgpu/raven_ta.bin (-2)

	the missing firmware was: raven_ta.bin

	downloaded raven_ta.bin from here: https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/tree/amdgpu

	sudo cp raven_ta.bin /lib/firmware/amdgpu

	sudo update-initramfs -u

# Fix dependency hell with dpkg
	sudo dpkg --force-all -i ./packagename.deb

# If you're stuck in grub command line and can't boot
	ls
	(hd0) (hd0,msdos2) (hd0,msdos1)

	ls (hd0,msdos1)/boot
	grub vmlinuz-3.10.0-1127.13.1.el7.x86_64 initramfs-3.10.0-1127.13.1.el7.x86_64.img
	set root=(hd0,msdos1)
	set prefix=(hd0,msdos1)/grub2
	linux (hd0,msdos1)/boot/vmlinuz-3.10.0-1127.13.1.el7.x86_64 root=/dev/sda1
	initrd (hd0,msdos1)/boot/initramfs-3.10.0-1127.13.1.el7.x86_64.img

# Fix for old intel graphics, slow boot and wrong resolution
	GRUB_CMDLINE_LINE_LINUX_DEFAULT="quiet splash video=SVIDEO-1:d"

# Fix if you forgot the login credentials
	init=/bin/bash or single (for systemd)
	mount: /: /dev/sda1 already mounted on /.
	mount -o remount --rw /dev/sda1
	passwd (change the password)
	ls /home (to see available users)
	pick a username and run: 
	passwd username (change the desired password, this will change the password for this user)
	su mxlinux
	systemctl reboot (for systemd) or reboot -f (for sysvinit)
	now you should be able to login with your newly created password!

# Fix pipewire, pulseaudio cracking noise, sound
	mkdir -p ~/.config/pipewire/pipewire.conf.d/
	nano ~/.config/pipewire/pipewire.conf.d/low-latency.conf
	context.properties = {
		default.clock.rate        = 48000
		default.clock.quantum     = 1024
		default.clock.min-quantum = 256
	}

	systemctl --user daemon-reexec
	systemctl --user restart pipewire pipewire-pulse wireplumber

# Fix fan spin on suspend
	cat /sys/power/mem_sleep

	if it shows:

	[s2idle] deep

	then do:

	echo deep | sudo tee /sys/power/mem_sleep

	alternatively add this to grub parameters /etc/default/grub

	mem_sleep_default=deep

	sudo update-grub

# Fix wifi doesn't autoconnect
	cat /etc/NetworkManager/NetworkManager.conf

	[main]
	plugins=ifupdown,keyfile

	[ifupdown]
	managed=false


	[device]
	wifi.scan-rand-mac-address=no

# Fix wifi adapter not showing in the list
	how to fix wifi not showing up in network manager:

	journalctl -k | grep -i firmware

	for me it showed:

	ath10k_pci 0000:02:00.0: firmware: failed to load ath10k/cal-pci-0000:02:00.0.bin (-2)
	ath10k_pci 0000:02:00.0: firmware: direct-loading firmware ath10k/QCA6174/hw3.0/firmware-6.bin
	ath10k_pci 0000:02:00.0: firmware ver WLAN.RM.4.4.1-00157-QCARMSWPZ-1 api 6 features wowlan,ignore-otp,mfp crc32 90eebefb
	ath10k_pci 0000:02:00.0: firmware: direct-loading firmware ath10k/QCA6174/hw3.0/board-2.bin

	even though a few lines show that it failed toload the firmware, the latter ones successfully loaded the firmware

	once the fiemware loaded successfully let us see the driver in use:

	lspci -nnk | grep -iEA3 "net"

	output:
	02:00.0 Network controller [0280]: Qualcomm Atheros QCA6174 802.11ac Wireless Network Adapter [168c:003e] (rev 32)
		Subsystem: Lite-On Communications Inc QCA6174 802.11ac Wireless Network Adapter [11ad:0807]
		Kernel driver in use: ath10k_pci
		Kernel modules: ath10k_pci
	03:00.0 Ethernet controller [0200]: Realtek Semiconductor Co., Ltd. RTL8111/8168/8411 PCI Express Gigabit Ethernet Controller [10ec:8168] (rev 15)
		Subsystem: Acer Incorporated [ALI] RTL8111/8168/8411 PCI Express Gigabit Ethernet Controller [1025:1127]
		Kernel driver in use: r8169
		Kernel modules: r8169

	so Kernel driver in use: ath10k_pci, everything looks good here
	let us see if the driver is loaded:

	lsmod | grep ath10k

	output:
	ath10k_pci             49152  0
	ath10k_core           438272  1 ath10k_pci
	ath                    36864  1 ath10k_core
	mac80211             1085440  1 ath10k_core
	cfg80211             1056768  3 ath,mac80211,ath10k_core

	all is good here too
	let's see if it isn't blocked by the switch:

	sudo apt install rfkill     (use tethering for that)
	sudo rfkill list

	0: acer-wireless: Wireless LAN
		Soft blocked: yes
		Hard blocked: no
	1: acer-bluetooth: Bluetooth
		Soft blocked: yes
		Hard blocked: no
	2: phy0: Wireless LAN
		Soft blocked: yes
		Hard blocked: no
	3: hci0: Bluetooth
		Soft blocked: yes
		Hard blocked: no

	sudo rfkill unblock wlan
	sudo rfkill list

	0: acer-wireless: Wireless LAN
		Soft blocked: no
		Hard blocked: no
	1: acer-bluetooth: Bluetooth
		Soft blocked: yes
		Hard blocked: no
	2: phy0: Wireless LAN
		Soft blocked: yes
		Hard blocked: no
	3: hci0: Bluetooth
		Soft blocked: yes
		Hard blocked: no

	systemctl reboot

	VERY IMPORTANT! rfkill will still show it as Soft blocked, but now you can right click on the nm-applet and you will see the option to Enable Wi-Fi

# Fix older version of Call of Duty not starting
	MESA_EXTENSION_MAX_YEAR=2003 DRI_PRIME=0 wine CoDSP.exe

# Fix for Far Cry 3
	install via winetricks

	d3dcompiler_42 to 47
	d3dx10
	d3dx10_43
	d3dx11_42
	d3dx11_43

	winetricks corefonts

	run with -opengl

	in wintricks choose:

	glsl = enabled
	gsm = 3
	psm = 3
	renderer = gl
	videomemorysize = 1024 (then in regedit change it to 4096)
	vsm = 3

# Fix wine russian locale
	locale -a

	C
	C.utf8
	en_US.utf8
	POSIX

	sudo dpkg-reconfigure locales

	pick ru_RU.UTF-8
	hit space
	hit enter

	LANG=ru_RU.UTF-8 wine ...

# Install msi with wine
	wine msiexec /i whatever-filename.msi

	Alternatively:

	wine start whatever-filename.msi


```

## Tutorials
Step-by-step guides for performing tasks in Linux.

```bash
# Record the the system audio with different formats
	find your monitor source
	pactl list sources short

	You will see something like:
	alsa_output.pci-0000_00_1f.3.analog-stereo.monitor

	Record system audio
	ffmpeg -f pulse -i alsa_output.pci-0000_00_1f.3.analog-stereo.monitor output.wav

	Lower CPU usage (compressed format)
	Instead of WAV (raw), encode directly:
	ffmpeg -f pulse -i alsa_output.pci-0000_00_1f.3.analog-stereo.monitor -c:a flac output.flac

	or even lighter, this avoids huge files + reduces disk I/O:
	ffmpeg -f pulse -i alsa_output.pci-0000_00_1f.3.analog-stereo.monitor -c:a libmp3lame -b:a 192k output.mp3.

	Even more efficient (no resampling)
	ffmpeg -f pulse -i alsa_output.pci-0000_00_1f.3.analog-stereo.monitor -acodec copy output.mka

# Record the screen with audio
	wf-recorder --no-damage --framerate 30 --muxer mp4 -a

# How to kill, simulate ok and cancel
	kill -USR1 `pgrep yad`

	"kill -USR1 $YAD_PID" wil acts like pressing OK button
	"kill -USR2 $YAD_PID" wil acts like pressing Cancel button

	yad --text="Testing how abort a program" \
		--button='Ends-with-USR1:              \
		    bash -c "echo -n Leaving with USR1 returns\  &&
		        kill -USR1 $YAD_PID"'        \
		--button='Ends-with-USR2:              \
		    bash -c "echo -n Leaving with USR1 returns\  &&
		        kill -USR2 $YAD_PID"'; echo $?

# Generate Debian iso with xorriso
	xorriso -outdev debian-live-10.0.0-amd64-xfce+nonfree.iso -volid d-live -padding 0 -compliance no_emul_toc -map /tmp/debian-live-10.0.0-amd64-xfce+nonfree / -chmod 0755 / -- -boot_image isolinux dir=/isolinux -boot_image isolinux system_area=/usr/lib/ISOLINUX/isohdpfx.bin -boot_image any next -boot_image any efi_path=boot/grub/efi.img -boot_image isolinux partition_entry=gpt_basdat

# Extract the iso content
	xorriso -osirrox on -indev debian-7.6.0-amd64-CD-1.iso -extract / debian_cd1

	xorriso -osirrox on -indev debian-live-10.0.0-amd64-xfce+nonfree.iso -extract / debian-live-10.0.0-amd64-xfce+nonfree

# WebCam test with ffmpeg
	ffmpeg -f v4l2 -input_format mjpeg -video_size 640x480 -i /dev/video0 -f null -

# Cut CPU power to prevent overheating
	sudo apt install tlp
	sudo vi /etc/default/tlp

	uncomment:

	CPU_SCALING_GOVERNOR_ON_AC=powersave
	CPU_SCALING_GOVERNOR_ON_BAT=powersave

	change

	#CPU_MAX_PERF_ON_AC=100

	to

	CPU_MAX_PERF_ON_AC=50

	save, reboot

# Things to do after installing Debian
	sudo dpkg --add-architecture i386
	sudo apt update && sudo apt upgrade

	sudo nano /etc/environment

	WINEESYNC=1
	MESA_NO_ERROR=1
	XCURSOR_SIZE=48
	WLR_WL_OUTPUTS=1
	GTK_USE_PORTAL=1
	QT_ACCESSIBILITY=1
	GDMSESSION=wayland
	WLR_SESSION=logind
	GDK_BACKEND=wayland
	MOZ_ENABLE_WAYLAND=1
	LIBVA_DRIVER_NAME=iHD
	LIBSEAT_BACKEND=logind
	CLUTTER_BACKEND=wayland
	QT_QPA_PLATFORM=wayland
	VAAPI_MPEG4_ENABLED=true
	WINE_LARGE_ADDRESS_AWARE=1
	XDG_SESSION_DESKTOP=woodland
	WINE_FORCE_LARGE_ADDRESS_AWARE=1
	WLR_SCENE_DISABLE_DIRECT_SCANOUT=0
	ADW_DEBUG_COLOR_SCHEME=prefer-dark
	__EGL_VENDOR_LIBRARY_FILENAMES=/usr/share/glvnd/egl_vendor.d/50_mesa.json

# Limit memory usage
	systemd-run --scope -p MemoryHigh=5M glxgears

	this will give only 5 MB of RAM for glxgears, if it exceeds the 5 MB then it will swap
	you can specify M or G for gigabytes

# Tweak system performance\
	systemctl enable systemd-sysctl
	sudo nano /etc/sysctl.d/00-sysctl.conf

	# ===============================
	# SECURITY HARDENING
	# ===============================
	net.ipv4.conf.all.rp_filter = 1
	net.ipv4.conf.default.rp_filter = 1
	net.ipv4.conf.all.accept_source_route = 0
	net.ipv4.conf.default.accept_source_route = 0
	net.ipv6.conf.all.accept_source_route = 0
	net.ipv6.conf.default.accept_source_route = 0
	net.ipv4.conf.all.accept_redirects = 0
	net.ipv4.conf.default.accept_redirects = 0
	net.ipv6.conf.all.accept_redirects = 0
	net.ipv6.conf.default.accept_redirects = 0
	net.ipv4.conf.all.send_redirects = 0
	net.ipv4.conf.default.send_redirects = 0
	net.ipv4.conf.all.log_martians = 1
	net.ipv4.conf.default.log_martians = 1
	net.ipv4.tcp_rfc1337 = 1
	net.ipv4.tcp_syncookies = 1
	kernel.kptr_restrict = 2
	kernel.dmesg_restrict = 1

	# ===============================
	# POWER SAVING & LAPTOP TWEAKS
	# ===============================
	vm.laptop_mode = 5			# slight delay in disk flush
	vm.dirty_writeback_centisecs = 1500	# 15 seconds
	vm.dirty_expire_centisecs = 1500
	vm.dirty_ratio = 10
	vm.dirty_background_ratio = 5

	# ===============================
	# MEMORY MANAGEMENT
	# ===============================
	vm.swappiness = 10
	vm.vfs_cache_pressure = 100
	vm.oom_kill_allocating_task = 1
	vm.memory_failure_recovery = 1

	# ===============================
	# NETWORK PERFORMANCE
	# ===============================
	net.core.somaxconn = 4096
	net.core.netdev_max_backlog = 10000
	net.core.rmem_default = 262144
	net.core.rmem_max = 16777216
	net.core.wmem_default = 262144
	net.core.wmem_max = 16777216
	net.ipv4.tcp_rmem = 4096 87380 16777216
	net.ipv4.tcp_wmem = 4096 65536 16777216
	net.ipv4.tcp_congestion_control = bbr	# Only if available
	net.ipv4.tcp_fin_timeout = 15
	net.ipv4.tcp_keepalive_time = 300
	net.ipv4.tcp_keepalive_intvl = 15
	net.ipv4.tcp_keepalive_probes = 5
	net.ipv4.tcp_slow_start_after_idle = 0
	net.ipv4.tcp_tw_reuse = 1
	net.ipv4.tcp_max_syn_backlog = 4096

	# ===============================
	# FILE SYSTEM
	# ===============================
	fs.file-max = 2097152
	fs.inotify.max_user_instances = 1024
	fs.inotify.max_user_watches = 524288

	# ===============================
	# MISC
	# ===============================
	kernel.watchdog = 0
	kernel.nmi_watchdog = 0
	dev.i915.perf_stream_paranoid = 0	# Allows tools like intel_gpu_top

# Manipulations with squashfs
	##ADD A FOLDER TO SQUASHFS

	say your squashfs file is called filesystem.squashfs

	copy filesystem.squashfs to a dir and in that dir: mkdir squashfs-root

	inside squashfs-root copy whatever folder with files you want to add

	in that dir where your both filesystem.squashfs and squashfs-root/ (with all the needed dirs) run:

	sudo mksquashfs squashfs-root/ filesystem.squashfs <<<<(this will add the content of the squashfs-root/ to filesystem.squashfs, sudo is recommended for copying some system files)

	to verify if the newly files are there, run:

	sudo mount filesystem.squashfs /mnt

	go to /mnt and verify

	to unmount:

	sudo umount /mnt

# Speaker test
	speaker-test -c2 -t wav

# Sort directories before files
	dconf-editor

	org
	  gtk
		settings
		  file-chooser
		     sort-directories-first

# Change display manager logind, greetd
	sudo apt install greetd tuigreet seatd
	sudo nano /usr/local/bin/start-woodland

	#!/bin/sh
	exec dbus-run-session -- /usr/local/bin/woodland

	sudo chmod +x /usr/local/bin/start-woodland

	sudo nano /etc/greetd/config.toml

	[terminal]
	# The VT to run the greeter on. Can be "next", "current" or a number
	# designating the VT.
	vt = 1

	# The default session, also known as the greeter.
	[default_session]

	# `agreety` is the bundled agetty/login-lookalike. You can replace `/bin/sh`
	# with whatever you want started, such as `sway`.
	command = "/usr/local/bin/start-woodland"
	# if using wlgreet
	#command = "sway --config /etc/greetd/sway-config"

	# The user to run the command as. The privileges this user must have depends
	# on the greeter. A graphical greeter may for example require the user to be
	# in the `video` group.
	user = "diogenes"

	sudo usermod -aG input diogenes
	sudo systemctl disable getty@tty1
	sudo systemctl enable greetd

# Set BFQ scheduler
	cat /sys/block/sda/queue/scheduler
	[mq-deadline] none

	sudo nano /etc/modules
	bfq

	sudo nano /etc/udev/rules.d/60-scheduler.rules
	ACTION=="add|change", KERNEL=="nvme[0-9]*", ATTR{queue/scheduler}="bfq"
	ACTION=="add|change", KERNEL=="sd[a-z]|mmcblk[0-9]*", ATTR{queue/rotational}=="0", ATTR{queue/scheduler}="bfq"
	ACTION=="add|change", KERNEL=="sd[a-z]", ATTR{queue/rotational}=="1", ATTR{queue/scheduler}="bfq"

	cat /sys/block/sda/queue/scheduler
	mq-deadline [bfq] none

# Record both input and output
	1) pactl load-module module-loopback
	2) start recording with audacity
	3) open pavucontrol > Recording
	4) in 'ALSA plug-in [audacity]' change 'Built-in Audio Analog Stereo' to 'Monitor of Built-in Audio Analog Stereo'
	now it should record both mic and background 

# Make QT apps dark theme
	QT_STYLE_OVERRIDE=Adwaita-Dark chessx

# QEMU boot legacy and EFI
	for legacy:
	sudo qemu-system-x86_64 -machine accel=kvm:tcg -m 4512 -hda /dev/sdc
	for UEFI:
	sudo qemu-system-x86_64 --bios /usr/share/qemu/OVMF.fd -m 4096 -enable-kvm -vga virtio -hda /dev/sdc

# QEMU boot an USB drive
	sudo apt install qemu-system-x86
	sudo qemu-system-x86_64 -machine accel=kvm:tcg -m 4512 -hda /dev/sdc

# QEMU boot and ISO
	qemu-system-x86_64 -accel kvm -boot d -cdrom '/path/debian-live-12.0.0-amd64-xfce.iso' -m 4096

# Replacing all your files on GitLab in one go
	1. Clone your repo
	git clone https://gitlab.com/Diogeness/woodland.git

	cd woodland/

	2. Remove all existing files
	git rm -r *

	3. Copy your new files into this folder
	cp -r /home/diogenes/Documents/woodland-2.1.0/* .

	4. Add everything
	git add .

	5. Commit changes
	git commit -m "Updated to version 2.1.0"

	6. Push
	git push origin main

# Find PID by process name
	ps axf | grep 'yad --timeout-indicator=top' | grep -v grep

	output:
	29453 pts/1    Sl+    0:00  \_ yad --timeout-indicator=top --timeout=444 --posx=1 --posy=1 --on-top --no-buttons --skip-taskbar --width=900 --undecorated

	# Print the first word (in this case process ID)
	ps xeww | grep DISPLAY=:1 | head -n1 |  awk  '{print $1}'

# Run an app with NVIDIA
	GBM_BACKEND=nvidia-drm __NV_PRIME_RENDER_OFFLOAD=1 __GLX_VENDOR_LIBRARY_NAME=nvidia

# Nouveau performance tweak
	sudo nano /etc/default/grub
	GRUB_CMDLINE_LINUX_DEFAULT="nouveau.config=NvClkMode=10"

# Create a MULTIBOOT universal USB drive:
	sudo apt install gir1.2-udisks-2.0 grub-efi-ia32-bin grub-pc-bin grub-efi-amd64-bin grub2-common dosfstools udisks2 psmisc util-linux parted coreutils e2fsprogs

	umount /dev/sdX*
	sudo dd if=/dev/zero of=/dev/sdX bs=1k count=2048 oflag=dsync status=progress
	sudo parted -s /dev/sdX mklabel msdos
	sudo parted -s -a optimal /dev/sdX mkpart primary fat32 1MiB 100MiB
	sudo parted -s /dev/sdX toggle 1 boot
	sudo parted -s -a optimal /dev/sdX mkpart primary ext4 100MiB 100%
	udisksctl unmount -b /dev/sdX'1' 2>/dev/null
	udisksctl unmount -b /dev/sdX'2' 2>/dev/null
	sudo mkfs.vfat -F 32 -v -I -n "BOOT" /dev/sdX'1'
	sudo mkfs.ext4 -q -F -L "LIVEISOS" /dev/sdX'2'
	mkdir /tmp/usb-bootgrub
	sudo mount /dev/sdX1 /tmp/usb-bootgrub

	sudo grub-install --removable --no-nvram --no-uefi-secure-boot --efi-directory="/tmp/usb-bootgrub" --boot-directory="/tmp/usb-bootgrub/boot" --target=i386-efi

	sudo grub-install --removable --no-nvram --no-uefi-secure-boot --efi-directory="/tmp/usb-bootgrub" --boot-directory="/tmp/usb-bootgrub/boot" --target=x86_64-efi

	sudo grub-install --removable --no-floppy --boot-directory="/tmp/usb-bootgrub/boot" --target=i386-pc /dev/sdX
	sudo umount /tmp/usb-bootgrub
	udisksctl power-off -b /dev/sdX

	re-attach your usb drive and copy your iso into LIVEISOS

	and make a file grub.cfg with the appropriate menu entries in usbdevice/BOOT/boot/grub
	example of grub.cfg

	set timeout=10
	set default=0
	insmod search_fs_uuid
	search --no-floppy --set=isopart --fs-uuid "38e76c8f-c0a7-4ec0-8269-80c8568a90a0"

	menuentry 'debian-live-13.2.0-amd64-kde.iso' {
		set isofile='/debian-live-13.2.0-amd64-kde.iso'
		insmod loopback
		loopback loop ($isopart)$isofile
		linux (loop)/live/vmlinuz-6.12.57+deb13-amd64 boot=live findiso=$isofile noprompt noeject noswap config quiet splash
		initrd (loop)/live/initrd.img-6.12.57+deb13-amd64
	}

	menuentry 'xubuntu-20.04-desktop-amd64.iso' {
		set isofile='/xubuntu-20.04-desktop-amd64.iso'
		insmod loopback
		loopback loop ($isopart)$isofile
		linux (loop)/casper/vmlinuz boot=casper iso-scan/filename=$isofile noprompt noeject quiet splash
		initrd (loop)/casper/initrd
	}

	menuentry 'MX-Linux19-Customers.iso' {
		set isofile='/MX-Linux19-Customers.iso'
		insmod loopback
		loopback loop ($isopart)$isofile
		linux (loop)/antiX/vmlinuz fromiso=$isofile antiX=MLX
		initrd (loop)/antiX/initrd.gz
	}

	menuentry 'antiX-19_386-full.iso' {
		set isofile='/antiX-19_386-full.iso'
		insmod loopback
		loopback loop ($isopart)$isofile
		linux (loop)/antiX/vmlinuz fromiso=$isofile antiX=MLX
		initrd (loop)/antiX/initrd.gz
	}

	menuentry 'ubuntu-22.04-desktop-amd64.iso' {
		set isofile='/ubuntu-22.04-desktop-amd64.iso'
		insmod loopback
		loopback loop ($isopart)$isofile
		linux (loop)/casper/vmlinuz boot=casper iso-scan/filename=$isofile noprompt noeject quiet splash
		initrd (loop)/casper/initrd
	}

	menuentry 'devuan_chimaera_4.0.2_amd64_desktop-live.iso' {
		set isofile='/devuan_chimaera_4.0.2_amd64_desktop-live.iso'
		insmod loopback
		loopback loop ($isopart)$isofile
		linux (loop)/live/vmlinuz boot=live findiso=$isofile noprompt noeject noswap config quiet splash
		initrd (loop)/live/initrd.img
	}

	menuentry 'debian-live-12.4.0-amd64-xfce.iso' {
		set isofile='/debian-live-12.4.0-amd64-xfce.iso'
		insmod loopback
		loopback loop ($isopart)$isofile
		linux (loop)/live/vmlinuz-6.1.0-15-amd64 boot=live findiso=$isofile noprompt noeject noswap config quiet splash
		initrd (loop)/live/initrd.img-6.1.0-15-amd64
	}

# hotspot create
	sudo apt install dnsmasq
	sudo systemctl disable dnsmasq && sudo systemctl mask dnsmasq
	sudo systemctl stop dnsmasq
	nmcli -s dev wifi hotspot con-name diohotspot
	nmcli con down diohotspot

	after that connect to it like this:
	nmcli con up diohotspot
	nmcli dev wifi show-password (to see the password)

# Remove wine file associations
	Disabling winemenubuilder.exe altogether will prevent wine from hijacking your file associations,
	but it will also prevent it from creating menu entries for newly installed software, which may be
	an undesired behavior. The better solution is this:

	Remove existing wine hijacks (from wine FAQ):

	rm -f ~/.local/share/mime/packages/x-wine*
	rm -f ~/.local/share/applications/wine-extension*
	rm -f ~/.local/share/icons/hicolor/*/*/application-x-wine-extension*
	rm -f ~/.local/share/mime/application/x-wine-extension* 

	Edit  /usr/share/wine/wine/wine.inf (as root), find the [Services] section:
	pkexec env DISPLAY=$DISPLAY XAUTHORITY=$XAUTHORITY mousepad /usr/share/wine/wine/wine.inf

		[Services]
		HKLM,%CurrentVersion%\RunServices,"winemenubuilder",2,"%11%\winemenubuilder.exe -a -r"
		...

		and edit it so it says:

		[Services]
		HKLM,%CurrentVersion%\RunServices,"winemenubuilder",2,"%11%\winemenubuilder.exe -r"
		...

	namely, to start winemenubuilder.exe without the -a switch. This will prevent updating
	file associations on new user accounts (or with new WINEPREFIXes).

	Edit your $WINEPREFIX/system.reg file (if it exists) in similar fashion. Where it says
	mousepad /home/diogenes/.wine/system.reg

		[Software\\Microsoft\\Windows\\CurrentVersion\\RunServices]
		"winemenubuilder"="C:\\windows\\system32\\winemenubuilder.exe -a -r"

		remove the -a switch. (By default, WINEPREFIX=$HOME/.wine.)

	This will prevent wine from stealing your preferred mimeapps, but the winemenubuilder will still
	run and create convenient desktop entries for your Windoze software.

# Change default http mime, mimetype handler
	nano ~/.local/share/applications/mimeinfo.cache

	change:
	x-scheme-handler/http=brave-browser-wayland.desktop;
	x-scheme-handler/https=brave-browser-wayland.desktop;

	to:
	x-scheme-handler/http=org.gnome.Epiphany.desktop;
	x-scheme-handler/https=org.gnome.Epiphany.desktop;

	it will have effect immediately after saving the file

# Set dark mode for desktop
	gsettings set org.gnome.desktop.interface color-scheme 'prefer-dark'

# Resize GIFs with command line
	convert cat-leaf.gif -coalesce -resize 220x160 -fuzz 10% -layers Optimize cat.gif
	gifsicle -O3 --colors 16 cat.gif -o cat2.gif

# Clean resources
	Use echo 1 to free page cache:
	echo 1 > /proc/sys/vm/drop_caches

	Use echo 2 to free dentries and inodes:
	echo 2 > /proc/sys/vm/drop_caches

	Use echo 3 to free page cache, dentries, and inodes:
	echo 3 > /proc/sys/vm/drop_caches

	to clean the swap:
	sudo swapoff -a && sudo swapon -a

# CHange default cursor theme
	extract your theme to /usr/share/icons
	sudo update-alternatives --config x-cursor-theme

	this will show you the registered themes and their priorities, the one that has higher priority number, that one will be default (auto mode), in my case it was:

	*0            /usr/share/icons/Adwaita/cursor.theme       90        manual mode
	 1            /usr/share/icons/DMZ-Black/cursor.theme     30        manual mode
	 2            /usr/share/icons/DMZ-White/cursor.theme     50        manual mode

	this means that Adwaita is the default and my newly extracted theme (/usr/share/icons/Edge-Cursor) was not even in the list, to add it to the list run:

	sudo update-alternatives --install /usr/share/icons/default/index.theme x-cursor-theme /usr/share/icons/Edge-Cursor/cursor.theme 100

	to make it default, you need to give it hightr priority number, in my case i put: 100
	it should be set now to default, to check, tun:

	sudo update-alternatives --config x-cursor-theme

	pick the numbr and hit enter

	reboot

# Find logs from other boots
	run: journalctl --list-boots
	find there the ID of your last boot and run: journalctl -b IDNUMBER -e

	example:

	journalctl --list-boots

	IDX BOOT ID                          FIRST ENTRY                 LAST ENTRY                 
	  0 730e7ee2f2aa498a8f29c7545445c95b Sat 2024-02-24 08:36:56 EET Sat 2024-02-24 09:32:26 EET
	  
	then run:

	journalctl -b 730e7ee2f2aa498a8f29c7545445c95b -e

# Command to find all .doc files in a given directory and copy them to taeget directory
	find . -type f -name '*.doc' -exec cp --target-directory=/home/diogenes/Desktop/ {} +

# Make grep always highlight the result
	nano .bashrc
	alias grep='grep --color=always'

# Change CPU governor (temporary)
	echo performance | sudo tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor

# Check current CPU governor
	cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor

# View available governors
	cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_available_governors

# Add a nice terminal promps
	sudo apt install powerline fonts-powerline python3-powerline

	add to your .bashrc
	__powerline() {
		# Unicode symbols
		readonly GIT_BRANCH_CHANGED_SYMBOL='+'
		readonly GIT_NEED_PULL_SYMBOL='⇣'
		readonly GIT_NEED_PUSH_SYMBOL='⇡'
		readonly PS_SYMBOL='🐧'

		# Solarized colorscheme
		readonly BG_BASE00="\\[$(tput setab 11)\\]"
		readonly BG_BASE01="\\[$(tput setab 10)\\]"
		readonly BG_BASE02="\\[$(tput setab 0)\\]"
		readonly BG_BASE03="\\[$(tput setab 8)\\]"
		readonly BG_BASE0="\\[$(tput setab 12)\\]"
		readonly BG_BASE1="\\[$(tput setab 14)\\]"
		readonly BG_BASE2="\\[$(tput setab 7)\\]"
		readonly BG_BASE3="\\[$(tput setab 15)\\]"
		readonly BG_BLUE="\\[$(tput setab 4)\\]"
		readonly BG_COLOR1="\\[\\e[48;5;240m\\]"
		readonly BG_COLOR2="\\[\\e[48;5;238m\\]"
		readonly BG_COLOR3="\\[\\e[48;5;238m\\]"
		readonly BG_COLOR4="\\[\\e[48;5;31m\\]"
		readonly BG_COLOR5="\\[\\e[48;5;31m\\]"
		readonly BG_COLOR6="\\[\\e[48;5;237m\\]"
		readonly BG_COLOR7="\\[\\e[48;5;237m\\]"
		readonly BG_COLOR8="\\[\\e[48;5;161m\\]"
		readonly BG_COLOR9="\\[\\e[48;5;161m\\]"
		readonly BG_CYAN="\\[$(tput setab 6)\\]"
		readonly BG_GREEN="\\[$(tput setab 2)\\]"
		readonly BG_MAGENTA="\\[$(tput setab 5)\\]"
		readonly BG_ORANGE="\\[$(tput setab 9)\\]"
		readonly BG_RED="\\[$(tput setab 1)\\]"
		readonly BG_VIOLET="\\[$(tput setab 13)\\]"
		readonly BG_YELLOW="\\[$(tput setab 3)\\]"
		readonly BOLD="\\[$(tput bold)\\]"
		readonly DIM="\\[$(tput dim)\\]"
		readonly FG_BASE00="\\[$(tput setaf 11)\\]"
		readonly FG_BASE01="\\[$(tput setaf 10)\\]"
		readonly FG_BASE02="\\[$(tput setaf 0)\\]"
		readonly FG_BASE03="\\[$(tput setaf 8)\\]"
		readonly FG_BASE0="\\[$(tput setaf 12)\\]"
		readonly FG_BASE1="\\[$(tput setaf 14)\\]"
		readonly FG_BASE2="\\[$(tput setaf 7)\\]"
		readonly FG_BASE3="\\[$(tput setaf 15)\\]"
		readonly FG_BLUE="\\[$(tput setaf 4)\\]"
		readonly FG_COLOR1="\\[\\e[38;5;250m\\]"
		readonly FG_COLOR2="\\[\\e[38;5;240m\\]"
		readonly FG_COLOR3="\\[\\e[38;5;250m\\]"
		readonly FG_COLOR4="\\[\\e[38;5;238m\\]"
		readonly FG_COLOR6="\\[\\e[38;5;31m\\]"
		readonly FG_COLOR7="\\[\\e[38;5;250m\\]"
		readonly FG_COLOR8="\\[\\e[38;5;237m\\]"
		readonly FG_COLOR9="\\[\\e[38;5;161m\\]"
		readonly FG_CYAN="\\[$(tput setaf 6)\\]"
		readonly FG_GREEN="\\[$(tput setaf 2)\\]"
		readonly FG_MAGENTA="\\[$(tput setaf 5)\\]"
		readonly FG_ORANGE="\\[$(tput setaf 9)\\]"
		readonly FG_RED="\\[$(tput setaf 1)\\]"
		readonly FG_VIOLET="\\[$(tput setaf 13)\\]"
		readonly FG_YELLOW="\\[$(tput setaf 3)\\]"
		readonly RESET="\\[$(tput sgr0)\\]"
		readonly REVERSE="\\[$(tput rev)\\]"

		__git_info() {
		    # no .git directory
			[ -d .git ] || return

		    local aheadN
		    local behindN
		    local branch
		    local marks
		    local stats

		    # get current branch name or short SHA1 hash for detached head
		    branch="$(git symbolic-ref --short HEAD 2>/dev/null || git describe --tags --always 2>/dev/null)"
		    [ -n "$branch" ] || return  # git branch not found

		    # how many commits local branch is ahead/behind of remote?
		    stats="$(git status --porcelain --branch | grep '^##' | grep -o '\[.\+\]$')"
		    aheadN="$(echo "$stats" | grep -o 'ahead \d\+' | grep -o '\d\+')"
		    behindN="$(echo "$stats" | grep -o 'behind \d\+' | grep -o '\d\+')"
		    [ -n "$aheadN" ] && marks+=" $GIT_NEED_PUSH_SYMBOL$aheadN"
		    [ -n "$behindN" ] && marks+=" $GIT_NEED_PULL_SYMBOL$behindN"

		    # print the git branch segment without a trailing newline
		    # branch is modified?
		    if [ -n "$(git status --porcelain)" ]; then
		        printf "%s" "${BG_COLOR8}$RESET$BG_COLOR8 $branch$marks $FG_COLOR9"
		    else
		        printf "%s" "${BG_BLUE}$RESET$BG_BLUE $branch$marks $RESET$FG_BLUE"
		    fi
		}

		ps1() {
		    # Check the exit code of the previous command and display different
		    # colors in the prompt accordingly.
		    if [ "$?" -eq 0 ]; then
		        local BG_EXIT="$BG_GREEN"
		        local FG_EXIT="$FG_GREEN"
		    else
		        local BG_EXIT="$BG_RED"
		        local FG_EXIT="$FG_RED"
		    fi

		    PS1="$FG_BASE3"
		    PS1+="$BG_COLOR5 \\w "
		    PS1+="$RESET${FG_COLOR6}"
		    PS1+="$(__git_info)"
		    PS1+="$BG_EXIT$RESET"
		    PS1+="$BG_EXIT$FG_BASE3 ${PS_SYMBOL} ${RESET}${FG_EXIT}${RESET} "
		}

		PROMPT_COMMAND=ps1
	}

	__powerline
	unset __powerline

# Create a .deb package
	mkdir -p myapp-2.1.0/DEBIAN
	mkdir -p myapp-2.1.0/usr/local/bin
	touch myapp-2.1.0/DEBIAN/control
	nano myapp-2.1.0/DEBIAN/control

	Package: myapp
	Version: 2.1.0
	Architecture: amd64
	Maintainer: Diogenes
	Installed-Size: 344
	Depends: libpixman-1-0, libxkbcommon0, libwayland-bin, libwlroots-0.18, wayland-protocols, libwayland-server0
	Provides: wayland-compositor
	Section: wayland
	Priority: optional
	Description: A minimal but functional wlroots-based Wayland compositor.

	Ctrl+s (save)
	Ctrl+x (exit)

	(now copy your binary to 'myapp-2.1.0/usr/local/bin'

	dpkg-deb --build --root-owner-group myapp-2.1.0

# Command to copy all the files with no ovewriting existent files
	sudo cp -r --copy-contents --no-clobber /path/to/copy/* /destination/path/

# Limit CPU usage for a application
	sudo nano /etc/polkit-1/rules.d/50-systemd-manage-units.rules

	polkit.addRule(function(action, subject) {
		if (action.id == "org.freedesktop.systemd1.manage-units" &&
			subject.isInGroup("sudo")) {
			return polkit.Result.YES;
		}
	});

	systemd-run -p CPUQuota=7% --scope appname

# Set consistent cursor theme
	gsettings set org.gnome.desktop.interface cursor-theme Adwaita
	gsettings set org.gnome.desktop.interface cursor-size 48

# Add subtitles to your video with ffmpeg
	create a file: subs.srt

	1
	00:00:00,000 --> 00:00:02,500
	Hello everyone.

	2
	00:00:02,600 --> 00:00:05,000
	Welcome to the UNI Community.

	3
	00:00:05,100 --> 00:00:07,200
	We’re glad to have you here!

	ffmpeg -i input.mp4 -vf "subtitles=your_subs.srt:force_style='FontName=Arial,FontSize=24,PrimaryColour=&HFFFFFF&'" -c:a copy output_with_subs.mp4

# Extract the last frame from your video with ffmpeg
	ffmpeg -i input.mp4 -vf reverse -vframes 1 last.png

	this one will extract and scale to 700x1000
	ffmpeg -i 2.mp4 -vf "reverse,scale=700:1000" -vframes 1 last.pn

	this will extract a frame at the given time/second
	ffmpeg -ss 00:00:03 -i 2.mp4 -frames:v 1 frame.png

# Reverse your video and show down with ffmepg
	reverse video:
	ffmpeg -i input.mp4 -vf reverse -an output_reverse.mp4

	slow motion
	ffmpeg -i input.mp4 -filter:v "setpts=3*PTS" -an output.mp4

	slow down video slow motion:
	ffmpeg -i input.mp4 -filter:v "setpts=2.5*PTS" -an output.mp4

	2.5 means 2.5× slower (4 seconds × 2.5 ≈ 10 seconds).
	-an removes audio (or you can stretch it too, but it sounds unnatural unless processed).

# Find installed packages using dpkg
	dpkg -l fire\* | grep ^.i
	dpkg -l unatt\* | grep ^.i

# Commit a pull request
	1) clone the repo:
	git clone https://github.com/DiogenesN/woodland.git

	2) change directory:
	cd woodland

	3) Find the PR number
	On GitHub, the PR title will look like: Makefile: restructuring #6
	so '#6' is the PR NUMBER:
	git fetch origin pull/<PR NUMBER>/head:pr-test

	4) Switch to the PR branch (Now you are on the PR code, not your main branch):
	git checkout pr-test

	5) test if the PR did not introduce any regression by building and running the project:
	chmod +x ./configure 
	./configure
	make clean
	make run

	6) see exactly what changed:
	git diff main..pr-test

	7) look if nothing weird here, no missing libs:
	ldd ./woodland

	8) change back to the main branch:
	git checkout main

	9) remove the testing branch:
	git branch -D pr-test

	10) Click on 'Merge pull-request'

# Check if something is blacklisted
	grep -r micro /etc/modprobe.d/*

# Show text containing the given key in all dirs and subdirs using egrep
	grep -R "keyword" -R /target/directory/*

# Add or remove your user to a group
	1) list all available groups:
	cut -d: -f1 /etc/group | sort

	2) add yourself to a group:
	sudo usermod -a -G examplegroup exampleusername

	example:
	sudo usermod -a -G systemd-journal,winbindd_priv diogenes
	relogin

	3) to remove yourself from a group:
	sudo gpasswd -d user group
	relogin

# If your startup is broken and you can't login
	1) boot your system and as soon as you see the grub menu, press 'e'
	2) navigate to the line that starts with linux
	3) add this parameter:
	systemd.unit=multi-user.target
	4) press: ctrl+x

# Add hidden timeout for grub
	sudo nano /etc/default/grub

	GRUB_HIDDEN_TIMEOUT=0
	GRUB_HIDDEN_TIMEOUT_QUIET=true

	this will save the last used choice
	GRUB_DEFAULT=saved
	GRUB_SAVEDEFAULT=true

	sudo update-grub

# dpkg list all the paths a deb package has installed
	cat /var/lib/dpkg/info/dioliveusb.list

	/.
	/opt
	/opt/scripts
	/opt/scripts/dioliveusb.sh
	/usr
	/usr/local
	/usr/local/share
	/usr/local/share/applications
	/usr/local/share/applications/dioliveusb.desktop
	/usr/local/share/icons
	/usr/local/share/icons/hicolor
	/usr/local/share/icons/hicolor/128x128
	/usr/local/share/icons/hicolor/128x128/apps
	/usr/local/share/icons/hicolor/128x128/apps/dioliveusb.png

# Silent boot, supress boot messages
	sudo systemctl edit getty@.service

	YOU NEED TO PUT YOUR LINES RIGHT BETWEEN THOSE TWO COMMENTED LINES!

	## Editing /etc/systemd/system/getty@.service.d/override.conf
	### Anything between here and the comment below will become the contents of the drop-in file

	[Service]
	ExecStart=
	ExecStart=-/sbin/agetty -o '-p -- \\u' --nonewline --noissue --noclear --nohostname - $TERM

	### Edits below this comment will be discarded

	sudo nano /etc/issue
	remove:
	Debian GNU/Linux bookworm/sid \n \l

	sudo nano /etc/default/grub
	GRUB_CMDLINE_LINUX_DEFAULT="quiet splash loglevel=3  rd.systemd.show_status=auto rd.udev.log-priority=3 vt.global_cursor_default=0"
	sudo update-grub

	sudo nano /etc/grub.d/10_linux
	comment:
	#    message="$(gettext_printf "Loading Linux %s ..." ${version})"
	#      message="$(gettext_printf "Loading initial ramdisk ...")"
	remove lines:
	echo	'$(echo "$message" | grub_quote)'
	echo	'$(echo "$message" | grub_quote)'
	sudo update-grub

# Skip filesystem check on boot
	sudo nano /etc/default/grub
	GRUB_CMDLINE_LINUX_DEFAULT="quiet splash fsck.mode=skip"
	sudo update-grub

```

## Text Processing (sed, awk, and others)

Useful commands for parsing and manipulating text.

```bash
# Different string manipulaitons
	#checks if string contains only numbers

	=~ ^[0-9]+$



	#checks if string contains only numbers and letters

	=~ ^[0-9a-zA-Z]+$



	#checks if string contains special characters

	== *['!'@#\$%^\&*()_+]*

# Check if a string contains only numbers
	echo 1234 | grep -E "^[[:digit:]]{1,}$" && echo "contains only numbers" || echo "not only numbers"

	output

	1234
	contains only numbers

# Add numbers plus result with leding zeros
	new=$(awk "BEGIN { printf \"%.02d\n\", 02 + 1 }")

	echo $new

	04

	\"%.02d\n\",  allows to get the output with leading zero 04, 05, 06 etc.

# Add quotes to the beginning and the end
    cat /tmp/applist2|sed '1s/^/"/;s/$/"/' - adds q at the beginning and each end
    cat /tmp/applist2|sed "\$s/\$/'/"      - adds quote only at the end
    
    cat /tmp/applist2|sed '1s/^/"/;$s/$/"/'
    out:
    "one
    two
    three"

# Adds slash at the end
	cat /tmp/LOOPMOUNTPOINTWITHQUOTESlive
	output:

	/media/diogenes/d-live nf 10.4.0 ci amd64

	cat /tmp/LOOPMOUNTPOINTWITHQUOTESlive | sed -e 's/$/\/live/'
	output:

	/media/diogenes/d-live nf 10.4.0 ci amd64/live

	this | sed -e 's/$/\/live/' adds "/live" after the last word in the line

# Add a character after each word
	ls /opt/dioalarmclock/tones

	output

	appointed.ogg            goes-without-saying.ogg  out-of-space-dog.ogg
	breaking-some-glass.ogg  good-things-happen.ogg   piece-of-cake.ogg
	capisci.ogg              got-it-done.ogg          plucky.ogg
	case-closed.ogg          inflicted.ogg            pristine.ogg
	closure.ogg              isnt-it.ogg              quite-impressed.ogg
	confident.ogg            jingle-bells-sms.ogg     serious-strike.ogg
	consequence.ogg          juntos.ogg               solemn.ogg
	definite.ogg             just-saying.ogg          swiftly.ogg
	done-for-you.ogg         maybe-one-day.ogg        that-was-quick.ogg
	dont-think-so.ogg        not-bad.ogg              thweep.ogg

	ls /opt/dioalarmclock/tones | sed -e :a -e '$!N; s/\n/|/; ta'

	output

	appointed.ogg|breaking-some-glass.ogg|capisci.ogg|case-closed.ogg|closure.ogg|confident.ogg|consequence.ogg|definite.ogg|done-for-you.ogg|dont-think-so.ogg|eventually.ogg|goes-without-saying.ogg|good-things-happen.ogg|got-it-done.ogg|inflicted.ogg|isnt-it.ogg|jingle-bells-sms.ogg|juntos.ogg|just-saying.ogg|maybe-one-day.ogg|not-bad.ogg|oringz-w468.ogg|out-of-space-dog.ogg|piece-of-cake.ogg|plucky.ogg|pristine.ogg|quite-impressed.ogg|serious-strike.ogg|solemn.ogg|swiftly.ogg|that-was-quick.ogg|thweep.ogg

	| sed -e :a -e '$!N; s/\n/|/; ta' adds '|' after each word

# Add word to the end of the line
	sed -e '2s/$/Debian/' textfile.txt
	this will add word Debian to the end of the second line of the textfile.txt

# Start from a given word and print till the end
	echo one two three and more

	output
	one two three and more

	echo one two three and more | awk '{a="";for (i=3;i<=NF;i++){a=a" "$i}print a}'

	output
	 three and more

	| awk '{a="";for (i=3;i<=NF;i++){a=a" "$i}print a}' <---------------- will print till the 3rd word till the end

	i=3 <---------------- 3 is the 3rd word

# cat pipe pattern
	sed -e "2s/$/$(cat /tmp/usbdev1)/" /home/diogenes/newappend.sh
	this will put the output of cat /tmp/usbdev1 at the end of the 2nd line in /home/diogenes/newappend.sh

# Comment out all the uncommented line
	sed -e 's/^\([^#].*\)/# \1/g'  will comment out all the uncommented lines only

# Count the number of slashes
	pwd
	/home/diogenes

	pwd | awk -F"/" '{print NF-1}'
	2

# Cut the third character
	echo '15:00200909'

	output:
	15:00200909

	echo '15:00200909' | cut --complement -c3

	output
	1500200909

	the | cut --complement -c3 cut the 3rd character which is ":"

# Print only n characters from a string
	echo '15:00200909'

	output:
	15:00200909

	echo '15:00200909' | cut --complement -c3

	output
	1500200909

	the | cut --complement -c3 cut the 3rd character which is ":"

# grep mountpoints
	grep "^/dev/sdc2 " /proc/mounts | cut -d ' ' -f 2

	/media/diogenes/DIOLVISO

# If finds pattern then print to the first empty line
	sed -n '/xubuntu-18.04-desktop-amd64.iso/,/^$/p' grub.cfg

	output:

	menuentry 'xubuntu-18.04-desktop-amd64.iso' {
		set isofile='/xubuntu-18.04-desktop-amd64.iso'
		loopback loop (hd0,4)$isofile
		linux (loop)/casper/vmlinuz file=/cdrom/preseed/xubuntu.seed boot=casper iso-scan/filename=$isofile quiet splash ---
		initrd (loop)/casper/initrd.lz
	}


	in a grub.cfg it finds the line that includes (NOT necessarily begins with) xubuntu-18.04-desktop-amd64.iso and prints the further content down till it finds an empty new line  

	^ = beginning
	$ = end
	p = print

# Insert a character after a given character
	sed 's/\<Hello World\>/& really/' file.txt

	this will parse file.txt and wherever it finds "Hello World" it inserts the word "really"
	after "hello World" so the end result is "Hello World really"

# Insert character at the beginning of the given line
	cat /tmp/titlesong | cut -c 9-

	output:
	Ты Далеко

	cat /tmp/titlesong | cut -c 9- | sed '1s/./#&/'

	output:
	#Ты Далеко

	sed '1s/./#&/' inserts a # at the beginning of first line adjust 1s to 2s and it will
	insert at the beginning of the 2nd line

# Insert character to the end of the line
	cat blabla

	01.bla bla bla

	cat blabla | sed -e 's/$/.mp3/'

	appends .mp3 to the end of the line
	01.bla bla bla.mp3

# Insert a new line after mactched pattern
	cat .config/dfiles/dirs

	/usr/share/icons/Lyra-blue-dark-diogenes/22x22/places/folder.png
	Videos
	1001
	Oct
	2
	16:25

	cat .config/dfiles/dirs | sed '/^Videos.*/i New Line'

	/usr/share/icons/Lyra-blue-dark-diogenes/22x22/places/folder.png
	New Line
	Videos
	1001
	Oct
	2
	16:25

	sed '/^Videos.*/i New Line' inserts a new line with text "New Line" above the Videos word

# Insert new line at the end and beginning
	egrep -r 'Name' /home/diogenes/.local/share/applications/* /usr/local/share/applications/* /usr/share/applications/* | grep -F -v "Name[" | grep -F -v "#" | grep -F -w Name | sed 's:.*=::' | awk '{$1=$1;print}' | sed '1s/^/"/;$s/$/"/'

# Insert placeholder instead of text pattern
	IMPORTANT!

	IF YOU WANT YOUR SED $PLACEHOLDER TO WORK THEN USE DOUBLE QUOTES!!!!!

	EXAMPLE: sed "/$GRUBENTRYTOREMOVE/,/^$/{d;}" 

	GRUBENTRYTOREMOVE=sometext

	echo $GRUBENTRYTOREMOVE

	output:

	sometext

	here is how you use $GRUBENTRYTOREMOVE placeholder with sed:

	sed "/$GRUBENTRYTOREMOVE/,/^$/{d;}" $MOUNTPOINTDIOBOOTPATH

	this ^^^^^^ with look for "sometext" in $MOUNTPOINTDIOBOOTPATH and if it finds the text
	then it will remove the line that contains the text and also all the lines below till
	it reaches a new blank line then it stops

# Print after last slash
	echo "/home/diogenes/fake_distros/Antix-Linux-desktop-32bit.iso" | sed 's:.*/::'

	output:
	Antix-Linux-desktop-32bit.iso

	| sed 's:.*/::' this prints only the last word after the last / or delete2 everything until the last slash

# Print first line
	sed -n 1p prints the first line only

# Print the last character of a string
	echo str |{ read; echo "${REPLY#${REPLY%?}}";}
	r

	this will print only the last character of a string, in out case the string is "str" and the last character is "r"

	# Print last line
	ls

	1
	2
	3

	ls | sed '$!d'

	3

	| sed '$!d' will print the last file/line in a directory

# Print only file extension
	cat Documents/PROJECTS/dfiles/full_path

	/home/diogenes/20-intel.conf

	cat Documents/PROJECTS/dfiles/full_path | sed 's/.*\.//'

	conf

	sed 's/.*\.//' prints only the extension

# Print only last n characters
	echo 1234 | tr -d " \t\n\r" | tail -c 2

	output
	34

	tail -c 2 with print the last 2 characters, tr -d " \t\n\r" removes whitespaces

# Print only several characters from the beginning
	cat $HOME/.config/dioliveusb/zenoutorig

		#!/bin/bash
		zenity --list --column 'Select' --column 'USB Device' --radiolist true

	head -c 3  $HOME/.config/dioliveusb/zenoutorig

		#!/

	head -c 3 will print only the first 3 characters from $HOME/.config/dioliveusb/zenoutorig

	# Print penultimate line
		cat /tmp/.directoryname | sed 'x; ${s/.*/&,/;p;x}; 2d'

		#FvwmScript-DMelodyPlayerWindow,3586206
		/home/diogenes/Music/ENIGMATIC
		/home/diogenes/Music/Kamal - Voices (2022) [FLAC]
		/home/diogenes/Music/Arms and Sleepers
		/home/diogenes/Music/Akira Kosemura
		/home/diogenes/Music/Гости из Будущего
		/home/diogenes/Music/Кино (Виктор Цой) - Дискография (1982-1990) [Remastered]
		#end,

		cat /tmp/.directoryname | tail -3 | head -1

		/home/diogenes/Music/Кино (Виктор Цой) - Дискография (1982-1990) [Remastered]

		| tail -3 | head -1 - prints the second line from the end

# Print nth line from the file
	sed -n 4p file.txt

	this will print the 4th line from file.txt

# Print nth line from the output
	ls /path/to/file | awk 'NR==3 {print; exit}'

# Print only nth word from the output
	echo "one two three" | awk  '{print $2}' | tr -d " \t\n\r"

	output

	two

	awk  '{print $2}' prints only the 2nd word from a strin

# Print until given character
	xrandr | grep \* | awk '{print $1}'

	output

	1920x1080

	now i want it to print only the number until x (1920)
	xrandr | grep \* | awk '{print $1}' | cut -d x -f 1

	output

	1920

	| cut -d x -f 1 will print the text until first x found

# Adds quotes to the line
	cat /tmp/usbdev1

	sdd 14.9G Generic Flash_Disk

	sed 's/^/"/;s/$/"/' /tmp/usbdev1

	"sdd 14.9G Generic Flash_Disk"

# Remove all lines that start with given sign or word
	cat Documents/PROJECTS/dfiles/usbdevicesmountpaths
	OUTPUT:
	dfiles.sh /media/diogenes/days & sleep 3 ;
	 & sleep 3 ;
	 & sleep 3 ;
	 
	cat Documents/PROJECTS/dfiles/usbdevicesmountpaths | sed '/^ &/d'
	OUTPUT:
	dfiles.sh /media/diogenes/days & sleep 3 ;

	sed '/^ &/d' removes all the lines that start with " &" so it removed:
	 & sleep 3 ;
	 & sleep 3 ;

# Remove all slashes from a file
	pwd 
	/home/diogenes
	pwd | sed 's%/[^/]*$%/%'
	/home/

	pwd | sed 's%/[^/]*$%/%' | sed 's/.$//'
	/home

	pwd 
	/home/diogenes

	pwd | sed 's%/[^/]*$%/%'
	/home/

	remove all backslashes from a file
	cat file
	output:

	one/
	teo/
	three/

	sed 's/\\//g' file
	output

	one
	teo
	three

	sed 's/\\//g'  removes all the slashes

# Remove blank spaces, whitespaces
	lsblk -o KNAME,UUID | grep sdd2
	output:
	sdd2  6d69906a-5972-41f1-927c-b6e9aa2ea212

	lsblk -o KNAME,UUID | grep sdd2 | cut -d' ' -f2-
	output:
	 6d69906a-5972-41f1-927c-b6e9aa2ea212 (notice there is a space at the beginning of the output)

	lsblk -o KNAME,UUID | grep sdd2 | cut -d' ' -f2- | sed 's/^[[:space:]]*//g'
	output:
	6d69906a-5972-41f1-927c-b6e9aa2ea212

	adding | sed 's/^[[:space:]]*//g' eliminates the space from the beginning of the file

	this one might be better: 

	echo "123 " | tr -d " \t\n\r"

	123

# Remove a block of text if matching the given pattern
# for instance grub.cfg contains
	menuentry 'xubuntu-16.04.3-desktop-amd64.iso' {
		set isofile='/xubuntu-16.04.3-desktop-amd64.iso'
		loopback loop (hd0,4)$isofile
		linux (loop)/casper/vmlinuz.efi file=/cdrom/preseed/xubuntu.seed boot=casper iso-scan/filename=$isofile quiet splash ---
		initrd (loop)/casper/initrd.lz
	}

	menuentry 'xubuntu-18.04-desktop-amd64.iso' {
		set isofile='/xubuntu-18.04-desktop-amd64.iso'
		loopback loop (hd0,4)$isofile
		linux (loop)/casper/vmlinuz file=/cdrom/preseed/xubuntu.seed boot=casper iso-scan/filename=$isofile quiet splash ---
		initrd (loop)/casper/initrd.lz
	}

	now we run:
	sed '/xubuntu-18.04-desktop-amd64.iso/,/^$/{d;}' grub.cfg

	and it will delete the entire block that contains 'xubuntu-18.04-desktop-amd64.iso' till the next new line

# Remove first 5 characters from each line
	cat test.txt

	output:

	   1 #! /bin/bash
		2 # -*- mode: sh -*-
		3 
		4 
		5 KEY=$RANDOM
		6 
		7 res1=$(mktemp --tmpdir term-tab1.XXXXXXXX)
		8 res2=$(mktemp --tmpdir term-tab2.XXXXXXXX)
		9 res3=$(mktemp --tmpdir term-tab3.XXXXXXXX)


	sed 's/^.....//' test.txt

	output:

	#! /bin/bash
	 # -*- mode: sh -*-
	 
	 
	 KEY=$RANDOM
	 
	 res1=$(mktemp --tmpdir term-tab1.XXXXXXXX)
	 res2=$(mktemp --tmpdir term-tab2.XXXXXXXX)
	 res3=$(mktemp --tmpdir term-tab3.XXXXXXXX)
	 
	NOTE! sed 's/^.....//' removed the first 5 characters from each line

# Remove everything after the last stash
	xdotool getactivewindow getwindowname (click on mousepad window)

	out:

	~/Documents/C/diomenu/main.c - Mousepad

	xdotool getactivewindow getwindowname | sed 's![^/]*$!!'

	out:

	~/Documents/C/diomenu/

# Removes everything after colon
	sed 's/\:.*$//' - removes everything after :

	normal output:

	hello/world:name

	sed 's/\:.*$//'

	output:

	hello/world

# Remove file extension
	find $HOME/.local/share/applications /usr/local/share/applications /usr/share/applications -iname "*.desktop" | sed 's:.*/::' | sort

	swayreminder.desktop
	swifi.desktop
	termit.desktop

	find $HOME/.local/share/applications /usr/local/share/applications /usr/share/applications -iname "*.desktop" | sed 's:.*/::' | sort | sed 's/\.[^.]*$//'

	swayreminder
	swifi
	termit

	find $HOME/.local/share/applications /usr/local/share/applications /usr/share/applications -iname "*.desktop" | sed 's:.*/::' | sort | grep -Po '.*(?=\.)'

	swayreminder
	swifi
	termit

	so either sed 's/\.[^.]*$//' or grep -Po '.*(?=\.)' removes the file extensions

# Remove the first line
	tail -n +2 removed the first line

# Remove first n characters from a file
	lsblk -o KNAME,UUID | grep sdd2

	output: sdd2  6d69906a-5972-41f1-927c-b6e9aa2ea212

	lsblk -o KNAME,UUID | grep sdd2 | cut -c 5-

	output:   6d69906a-5972-41f1-927c-b6e9aa2ea212

	this | cut -c 5 - will cut the first 5 characters from the beginning of the file including spaces

# Remove the last character
	cat $HOME/.config/dioliveusb/zenoutorig

	#!/bin/bash
	zenity --list --column 'Select' --column 'USB Devices' --radiolist true

	sed '2s/.....$//'   $HOME/.config/dioliveusb/zenoutorig

	#!/bin/bash
	zenity --list --column 'Select' --column 'USB Devices' --radiolist

	in the command sed '2s/.....$//' 2s = means the second line .....$ each dot removes one character from the last word of the last line

	also this: sed 's/.$//'

	echo "me  " (two leading spaces)
	out:
	me  (with two leading spaces)

	echo "me  " | sed 's/.$//'
	out:
	me (with one leading space)

# Remove the last given word
	cat $CONFDIR/full_path
	/home/diogenes/ Text me when you get home #Shorts.mp4 3M Sep 25 16:00

	cat $CONFDIR/full_path | sed -r 's/\s+\S+\s+\S+\s+\S+\s+\S+\s*$//'
	/home/diogenes/ Text me when you get home #Shorts.mp4

	sed -r 's/\s+\S+\s+\S+\s+\S+\s+\S+\s*$//' removed 3M Sep 25 16:00 last 4 words

# Remove quotes
	echo " 'hey' "

	output
	 'hey' 

	echo " 'hey' " | tr -d "'"
	output
	 hey

# Remove the exact given word from string
	echo one two | sed 's/one//g'

	output
	 two

	echo one two | sed 's/two//g'

	output
	one 

	echo onetwo | sed 's/one//g'

	output
	two

	| sed 's/one//g' <------------ will remove the given word "one" even if it is inside a long text line onetwothree

	| tr -d "'"  removes the single quote '"' to remove doulbe quots

# Remove the first word in a text
	lsblk -o KNAME,UUID | grep sdd2

	output: sdd2  6d69906a-5972-41f1-927c-b6e9aa2ea212

	lsblk -o KNAME,UUID | grep sdd2 | cut -d' ' -f2-

	output:  6d69906a-5972-41f1-927c-b6e9aa2ea212

	this | cut -d' ' -f2- will remove the first word in a text (not n characters)

# Remove the last character
	echo abc | sed 's/.$//'

	output
	ab

	| sed 's/.$//'    removes the last character from a line

# Remove the list above pattern match
	cat .config/dfiles/dirs

	/usr/share/icons/Lyra-blue-dark-diogenes/22x22/places/folder.png
	Videos
	1001
	Oct
	2
	16:25

	cat .config/dfiles/dirs | sed -n '/Videos/{x;d;};1h;1!{x;p;};${x;p;}'

	Videos
	1001
	Oct
	2
	16:25

	sed -n '/Videos/{x;d;};1h;1!{x;p;};${x;p;}' removes the line above the ord Videos

# Remove the blank spaces at the end
	cat file

	output:
	"something "

	cat file | sed 's/ *$//g'

	output
	"something"

	sed 's/ *$//g' removes the trailing space the space that comes after the last character
	awk '{$1=$1;print}' - removes both trailing and leading white spaces

# Remove word occurence from output
	cat /tmp/dlauncherNAME

	"org.gnome.Epiphany"
	"org.gnome.Evince"
	"org.gnome.Evince-previewer"
	"org.gnome.FileRoller"
	"org.gnome.gedit"
	"org.gnome.Nautilus"
	"org.gnome.Polari"
	"org.gnome.seahorse.Application"
	"org.gnome.Terminal"

	cat /tmp/dlauncherNAME | awk '{gsub("org.gnome", "");print}'

	".Epiphany"
	".Evince"
	".Evince-previewer"
	".FileRoller"
	".gedit"
	".Nautilus"
	".Polari"
	".seahorse.Application"
	".Terminal"

	awk '{gsub("org.gnome", "");print}' will remove the org.gnome from the output

# Replae second line in a file
	cat replaceline

	output

	one
	two
	three

	sed '2s/.*/one/' replaceline

	output

	one
	one
	three

	sed '2s/.*/one/'  replaced the second line from file replaceline with one

# Replace char with space
	cat /home/diogenes/.config/dmelody/directories
	out:
	'/home/diogenes/Music/Arms and Sleepers'//'/home/diogenes/Music/Arms and Sleepers 2'

	cat /home/diogenes/.config/dmelody/directories | sed -E 's/\/\// /g'
	out:
	'/home/diogenes/Music/Arms and Sleepers' '/home/diogenes/Music/Arms and Sleepers 2'

	so sed -E 's/\/\// /g' or sed 's/\/\// /g' replaced the double // with space

# Replace empty line with word
	cat .config/dmelody/dplaylist2
	out:

	19
	11 - Jigsaw.mp3
	Mimi Page
	Jigsaw

	Other
	0:03:49

	sed 's/^$/None/' .config/dmelody/dplaylist2
	out:

	19
	11 - Jigsaw.mp3
	Mimi Page
	Jigsaw
	None
	Other
	0:03:49

	sed 's/^$/None/' replaces the empty new line with word None

# Replace empty line with comma
	cat /tmp/blabla

	01.bla bla bla

	#replace empty space with comma
	cat /tmp/blabla | sed "s/ /,/g"

	01.bla,bla,bla

	#replace empty space with _
	cat /tmp/blabla | sed "s/ /_/g"
	01.bla_bla_bla

# Replace first char
	echo $SECONDSTOADD
	64

	echo $SECONDSTOADD | sed 's/./0/'
	04

	| sed 's/./0/' replacesthe first char with 0

	to replace an nth character:

	cat somefile

	20201021085460

	cat somefile | sed -E -e 's/(.{12}).(.*)/\1X\2/'

	202010210854X0

	| sed -E -e 's/(.{12}).(.*)/\1X\2/'   replaced 6 with X

# Replace last line
	cat sedlastlinereplacetest

	output
	remove the text and write your note here

	created on:
	NN

	cat sedlastlinereplacetest | sed "$ s/.*/h/g"

	output
	remove the text and write your note here

	created on:
	h

	sed "$ s/.*/h/g" replaced the last line NN with h

	| sed '1 s/.*/h/g' will replace the first one

# Sort list into string
	awk 'BEGIN {for (i=0; i<= 3; i++) print i}'

	output
	0
	1
	2
	3

	awk 'BEGIN {for (i=0; i<= 3; i++) print i}' | sed -e :a -e '$!N; s/\n/|/; ta'

	output
	0|1|2|3

# Sort the list of paths by the basenames names after the last slash
	patharray=( $(cat /tmp/dlauncherPATH) )

	"/home/diogenes/.local/share/applications/audacity.desktop"
	"/home/diogenes/.local/share/applications/brave-wayland-browser.desktop"
	"/home/diogenes/.local/share/applications/etherape.desktop"
	"/home/diogenes/.local/share/applications/farcry3d11.desktop"
	"/home/diogenes/.local/share/applications/gparted.desktop"
	"/home/diogenes/.local/share/applications/io.github.Hexchat.desktop"
	"/home/diogenes/.local/share/applications/jasf.desktop"
	"/home/diogenes/.local/share/applications/justcause2.desktop"
	"/home/diogenes/.local/share/applications/org.thonny.Thonny.desktop"
	"/home/diogenes/.local/share/applications/swayreminder.desktop"
	"/home/diogenes/.local/share/applications/viber.desktop"
	"/home/diogenes/.local/share/applications/wine/Programs/AnyDesk/AnyDesk.desktop"
	"/home/diogenes/.local/share/applications/wine/Programs/AnyDesk/Uninstall AnyDesk.desktop"
	"/usr/local/share/applications/dmelody.desktop"
	"/usr/local/share/applications/gimp.desktop"
	"/usr/local/share/applications/swifi.desktop"
	"/usr/local/share/applications/wine.desktop"
	"/usr/share/applications/AcetoneISO.desktop"

	printf '%s\n' "${patharray[@]}" | awk '{ print $NF, $0 }' FS='/' OFS='/' | sort | cut -d'/' -f2-

	/usr/share/applications/AcetoneISO.desktop
	/home/diogenes/.local/share/applications/wine/Programs/AnyDesk/AnyDesk.desktop
	/home/diogenes/.local/share/applications/audacity.desktop
	/usr/share/applications/audacity.desktop
	/usr/share/applications/blueman-adapters.desktop
	/usr/share/applications/blueman-manager.desktop
	/usr/share/applications/brave-browser.desktop
	/home/diogenes/.local/share/applications/brave-wayland-browser.desktop
	/usr/share/applications/ca.desrt.dconf-editor.desktop
	/usr/share/applications/calf.desktop
	/usr/share/applications/com.github.wwmm.easyeffects.desktop
	/usr/share/applications/display-im6.q16.desktop
	/usr/local/share/applications/dmelody.desktop
	/home/diogenes/.local/share/applications/etherape.desktop
	/usr/share/applications/etherape.desktop
	/home/diogenes/.local/share/applications/farcry3d11.desktop
	/usr/share/applications/footclient.desktop
	/usr/share/applications/foot.desktop
	/usr/share/applications/foot-server.desktop
	/usr/share/applications/gcr-prompter.desktop
	/usr/share/applications/gcr-viewer.desktop
	/usr/local/share/applications/gimp.desktop

	this will sort the list of paths by the basenames (names after the last slash)

# Sort output in columns
	printf "%${w}s\n" 1 2 3

	out:
	1
	2
	3

# Command pipe, use the output of one command as argument for another command
	in a directory we have these files:

	file1
	file2
	file3

	cat file1
	#output
	file2

	#so the file1 containts one line that reads file2

	cat file1 | xargs xdg-open

	now this command gets the output of file1 (output is "file2") and uses it as the argument for xdg-open
	so this command will run as if you would run: xdg-open file2

# sed: replace text in a file
	sed -i 's/old/new/g' file.txt

# awk: print second column
	awk '{print $2}' file.txt

# cut: extract first field
	cut -d':' -f1 /etc/passwd

# sort + uniq: count occurrences
	sort file.txt | uniq -c

# grep: search for a pattern
	grep "search_term" file.txt

# Remove all the lines in a file that start with hashtags (#)
	grep -v "^#" file.txt

```

## Miscellaneous

Other helpful commands and tricks.

```bash
# Show disk usage of current directory
	du -sh *

# Find large files (>100MB)
	find / -type f -size +100M 2>/dev/null

# Show running processes
	ps aux | less

# Show system info
	uname -a

# Check memory usage
	free -h

```

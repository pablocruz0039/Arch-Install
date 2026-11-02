# Arch Linux VM Install Notes

Manual install following the [Arch Installation Guide](https://wiki.archlinux.org/title/Installation_guide)

1. Get the .iso from https://archlinux.org/download/
	1. Pick a US mirror
	2. Save the `archlinux-x86_64.iso` file
2. Create a New Virtual Machine in VirtualBox
	1. Name: ArchVM
	2. Type: Linux, Version: Arch Linux (64-bit)
	3. 4096 MB RAM, 2 CPUs
	4. 30 GB disk, dynamically allocated
	5. Check **Enable EFI** (Settings > System > Motherboard)
	6. Video memory to max (Settings > Display)
	7. Start the VM and pick "Arch Linux install medium"

3. Partition the disk
	1. `fdisk /dev/sda`
	2. `g` for a new GPT table
	3. Partition 1 (EFI)
		1. `n`, Enter, Enter, `+1G`
		2. `t`, then `1` for type EFI System
	4. Partition 2 (root)
		1. `n`, Enter, Enter, Enter (uses the rest of the disk)
	5. `p` to check, then `w` to write and quit
	6. **Mistake:** typed `t` at the "First sector" prompt, got "Value out of range", then pressed Enter through the rest. Partition 1 took the whole disk and `n` said "No enough free sectors available"
		1. Fix: `d` to delete the partition and redo `n` with `+1G` at the **Last sector** prompt
		2. Nothing is saved until `w`, so this was easy to undo
4. Format partitions
	1. `mkfs.fat -F32 /dev/sda1`
	2. `mkfs.ext4 /dev/sda2`
5. Mount file systems
	1. `mount /dev/sda2 /mnt`
	2. `mount --mkdir /dev/sda1 /mnt/boot`
		1. Mount root first, then boot
6. Install packages
	1. `pacstrap -K /mnt base linux linux-firmware sudo nano vim networkmanager grub efibootmgr base-devel git`
		1. Includes sudo, NetworkManager, and the bootloader packages now so I don't forget them later
7. Fstab
	1. `genfstab -U /mnt >> /mnt/etc/fstab`
	2. `cat /mnt/etc/fstab` to check it has both partitions
8. Chroot
	1. `arch-chroot /mnt`
9. Localization
	1. `ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime`
	2. `hwclock --systohc`
	3. `nano /etc/locale.gen` and uncomment `en_US.UTF-8 UTF-8`
	4. `locale-gen`
	5. `echo "LANG=en_US.UTF-8" > /etc/locale.conf`
10. Hostname and root password
	1. `echo "ArchVM" > /etc/hostname`
	2. `passwd`
		1. Nothing shows on screen while typing the password, that is normal
11. Bootloader
	1. `grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB`
	2. `grub-mkconfig -o /boot/grub/grub.cfg`
	3. `systemctl enable NetworkManager`
		1. Without this there is no internet after reboot
12. Reboot
	1. `exit`
	2. `umount -R /mnt`
	3. `reboot`
	4. Remove the ISO (Devices > Optical Drives > Remove disk from virtual drive), or it boots the installer again
	5. Got the `ArchVM login:` prompt, logged in as root
13. Make user accounts with sudo
	1. `useradd -m -G wheel -s /bin/bash Pablo`
	2. `passwd Pablo`
	3. Repeated for codi
	4. `EDITOR=nano visudo`
		1. Uncomment `%wheel ALL=(ALL:ALL) ALL`
	5. Test: `su - Pablo` then `sudo whoami`
		1. Printed `root`, so sudo works
14. Update the system
	1. `pacman -Syu`
15. Install the desktop (LXQt)
	1. `pacman -S xorg-server lxqt sddm breeze-icons xdg-utils qterminal firefox`
	2. `systemctl enable sddm`
16. Boot straight into the GUI
	1. `systemctl set-default graphical.target`
	2. `systemctl get-default`
		1. Should print `graphical.target`
17. Install zsh
	1. `pacman -S zsh zsh-completions`
	2. `chsh -s /usr/bin/zsh Pablo`
	3. `chsh -s /usr/bin/zsh codi`
	4. First login shows the zsh setup wizard, pressed `0` to create an empty `.zshrc`
	5. The prompt changed from `$` to `%`, which means zsh is running
18. Install SSH
	1. `pacman -S openssh`
	2. `systemctl enable --now sshd`
	3. `systemctl status sshd`
		1. Should say active (running)
	1. `nano /etc/pacman.conf` and uncomment `Color`
		1. Makes pacman output colored (done)
	2. **Mistake:** typed the `alias` and `autoload` lines straight into the root prompt instead of saving them in a file
		1. `~/.zshrc` is a file, not a command, so it said "No such file or directory"
		2. `autoload: command not found` because I was still in bash, and `autoload` is zsh only
		3. Fix: install zsh first, then edit `~/.zshrc` with nano
	3. `nano ~/.zshrc` and add:
		1. `alias ls='ls --color=auto'`
		2. `alias grep='grep --color=auto'`
		3. `autoload -U colors && colors`
		4. `PS1="%{$fg[green]%}%n@%m %{$fg[blue]%}%~ %{$reset_color%}%# "`
	4. Save with Ctrl+O, Enter, Ctrl+X
	5. `source ~/.zshrc`
	6. TODO: repeat for codi
19. Make aliases (add to `~/.zshrc`)
	1. `alias ll='ls -lah'`
	2. `alias update='sudo pacman -Syu'`
	3. `alias ..='cd ..'`
	4. `alias ports='ss -tulpn'`
	5. `alias myip='ip -brief addr'`
	6. `source ~/.zshrc` to load them
	7. `alias` to check which aliases are active
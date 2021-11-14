
# Boot Process

1. Bios -- execute MBR
2. MBR(Master Boot Record) -- executes GRUB(windows)/DOS(linux)
3. GRUG(Grand unified Bootloader) --executes kernel
4. Kernel -- executes Init
5. Init(sbin/init) -- run execute level program
6. Run level -- run level programs from /etc/rc.d/rc*.d
7. User prompt

## BIOS
- BIOS -A firmware(interaction b/n software and harware)
- Runs the POST(power-on self-test) to sure everything is fine. It gives the "beep" sound if everything is fine else "bebeepbebeep" with a beep-code to identify the problem
- Checks for bootable device(flash drive with the os) and hand over the computer to the os
- BIOS is stored in the BIOS chip (with manufacture settings). It's non-volatile
- The settings you input (fan speed, date and time, boot sequence) goes to the CMOS chip which is volatile(needs power) from the CMOS chip. When removed, the BIOS is reset to maufacturers settings and you loose all data(date/time settings, fan speed, boot sequence, etc)
- Limitation: The system that BIOS uses to start up the computer(MBR) can only handle partitions less than 2 TB

## UEFI
- UEFI(Unified Extensible Firmware Interface) -- a modern replacement of BIOS offers more support for large partitions

## Grub(legacy) and Grub2
- GRUB -- Take over from BIOS/UEFI at boot time, load itself, load the Linux kernel into memory, and turn over the execution to the kernel
- Tells the computer how to boot up and how to mount different partitions
- Grub -- on boot, Grub asks you to press something in some amount of secods time. It has `menu.lst` and `grub.conf` file in `/boot/grub` directory
- Grub2 -- boots straight to the login, unless you press long press shift to go into settings. It only has `grub.cfg` in `/boot/grub/` directory
- If you do get a kernel panic, or something wrong with your kernel system, reboot, long press shift, you'll see previous successful Grub files(kernels), roll back to a previous one and boot from there. You can then later fix the problematic Grub file(current one)

## Boot methods
- Systems boot from these boot methods
1. PXE(booting from network) pre-boot execution environment
2. iPXE(advanced of PXE)
3. CD
4. USB
5. ISO
- PXE -- uses tftp to download the boot file from the network(local)
- iPXE -- users http to download the boot files
- Hardward(BIOS/UEFI) vs Sofware

- At boot time, you dont' want to boot all or certain modules. These modules are in the `blacklist.conf` file in `/etc/modprobe.d/blacklist.conf/`
- Examples of such modules: `usbkbd, usbmouse, evbug, eth1394`

## Modules
- They're tools to manipulate the kernel modules
1. `insmod` -- (insert module), uses full path of the kernel module you want to install, no dependency checking, fails with no explanation
2. `modprobe` -- more efficient, takes only name of the module, fetch dependencies using `depmod`, uses isnmod under the hood. 
Example installation: `modprobe thunderbolt-net`
3. `depmod` -- Tool for finding module dependencies
- kernel module directory: `/lib/modules/<numb-generic>/kernel/drivers/net`
- list all module in the running kernel: `lsmod`
Removing module
- Remove module dependency first: `rmmod thunderbolt-net` // remove dependency first
- Remove dependency: `rmmod thunderbolt`


# GPT and MBR
- These are different ways of chopping your drive into pieces that can be recognized and mounted onto the system

- `MBR(Master Boot Record)` --
Raw device -- `/dev/sda/`
Partition -- `/dev/sda1/` and `/dev/sda2/`
- limited to `2TB` and 4 primary partitions
- Can only handle 4 primary partitions
- firmware interface -- BIOS

- `GPT(GUID Partition Table)` -- newer and feature-rich
** GUID - globally uniques identifier
- Does same partition, but scatter the partitions around for rundundancy
- fireware interface -- UEFI
- up to 128 primary partitions

# Filesystem Hierarchy
- Directory ~/dev/, /proc/partitions/
- Create partition table (with GPT or MBR) using `fdisk /dev/sda/` where `sda` can be the name of your device
- See all partitions

```bash
$ sudo fdisk -l
```
- os -- an interface b/n user and hardware
- kernel -- an interface b/n software and hardware

## LVM vs Standard Partition
- You have 3 disks of each having 1TB storage
- Standard partition -- partitions each separate disks 
- LVM -- aggregates all 3 disks, each is a physical volume (PV) is added to one or more volume groups (VG) and then you can carve out logical volume(LG) from the pool for usage
- `PV + PV + PV = VG`
- Create your desirable storage from `VG` called `LV`
```bash
Storage space is managed by combining or pooling the capacity of the available drives. With traditional storage, three 1 TB disks are handled individually. With LVM, those same three disks are considered to be 3 TB of aggregated storage capacity
```
- Unlike RAID, PVs need not to be the same size or disks that are the same speed

### Process
- First make your disk a physical volume (PV), ready to be added to a VG

```bash
$ sudo pvcreate /dev/sda1           # Make partition 1 of sda disk a PV
$ sudo pvcreate /dev/sda            # Make entire sda disk a PV
$ sudo pvdisplay                    # Display all physical volumes(PV)
```

- Create volume groups(VG) for the PVs to aggregate them
Syntax: `sudo vgcreate <name-of-vg> <PV-members>` with 

```bash
$ sudo vgcreate vg00 /dev/sdb1 /dev/sdb2          # With /dev/sdb1 and /dev/sdb2 as members
```

- Then, you can now create a logical volume(LV) from the VG by dividing the VG and use it just like a partition. Use the `lvcreate` command. Options are `-n(name of LV), -L(size in G or T)`
Syntax: `lvcreate -L size -n lvname vgname`
```bash
$ sudo lvcreate -L 2GB -n sales-lv vg-awesome
```

- Now, the LV, sales-lv just created needs a `filesystem` and `mount point`


- Create `filesystem` on new LV(or partition)

```bash
$ sudo mkfs.ext4 /dev/vg-awesome/sales-lv
```

- Create a mount point

```bash
$ mkdir /salestorage            # Make directory to serve as mount point
$ sudo mount /dev/vg-awesome/sales-lv  /salestorage
```

- Confirm the storage capacity is available

```bash
$ sudo du -h /salestorage
```

- Other benefits
1. You can extend/reduce volume groups(VGs) with `vgextend/vgreduce`

2. You can extend/reduce logical volumes(LGs) with `lvextend/lvreduce` by first unmounting the filesystem


## RAID
- Redundant Array of Independent Disks
1. RAID0 -- Writing data across all disks(disks) at the same time. 
- It's fast
- Data is is not replicated across disks. so no redundancy

2. RAID1 -- Writing data into one disks and data replicated(mirrored) to other disks
- Low performance but provides high redundancy and high-availability

3. RAID5 -- Uses minimum of 3 drives(disks) with parity information distributed among the drives

- /etc/mdadm/mdadm.conf     # To install the RAID configuration to picked up during boot time to create it

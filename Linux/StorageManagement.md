# Storage
- [Storage and File System Basics](#storage-and-file-system-basics)
- [Partition Management](#partition-management)
- [Logical Volume Manager](#logical-volume-manager)
- [Archiving, Backup & Recovery](#archiving-backup--recovery)
- [Storage Troubleshooting](#storage-troubleshooting)
- [Remote File System Access](#remote-file-system-access)


## Storage and File System Basics
* **Section overview**
    * This section will offer a brief overview over storage and file system basics
    * It covers the Linux file system, inodes, hard and symbolic linking, how to localize files, and the essential of RAID configuration

### The Linux File System

* To read up on the Linux file system:
    * `man hier`

`/` - Root
- The root directory is the top-level directory in the Linux filesystem hierarchy.

`/bin` - Essential Binaries
- Contains binaries or executables that are essential to the entire OS.
- Commands like `gzip`, `curl`, `ls` are stored here.
- All users can by default run these 

`/sbin` - System Binaries
- Contains system binaries or executables for the superuser (root).
- Example: `mount`.

`/lib` - Libraries
- Shared libraries for essential binaries in `/bin` and `/sbin` are stored here.

`/usr` - User Directory
- Contains non-essential binaries or executables intended for the end user.
- Includes:
  - `/usr/bin`: Non-essential binaries.
  - `/usr/sbin`: Non-essential system binaries.
  - `/usr/lib`: Libraries for `/usr/bin` and `/usr/sbin`.
- `/usr/local`:
  - Contains manually compiled binaries in `/usr/local/bin`.
  - Provides a safe place to avoid conflicts with system-installed packages.
- Path and Precedence
  - All these binaries are mapped together using the `PATH` environment variable (`$PATH`).
  - `which` command shows the location of a binary and gives precedence to `/usr/bin` if the command exists in multiple places.

`/etc` - Editable Text Config
- Stores system configuration files (e.g., `.config` files).

`/home` - User Files
- Contains user-specific files and configurations.

`/boot` - Boot Files
- Contains files required to boot the system:
  - `initrd`
  - The Linux kernel:
    - `vmlinux`: Uncompressed kernel.
    - `vmlinuz`: Compressed kernel.

`/dev` - Device Files
- Provides access to hardware and drivers as if they are files.

`/opt` - Optional Software
- Stores optional or add-on software.

`/var` - Variable Files
- Contains files that change while the OS is running:
  - Logs are stored here.

- `/tmp` - Temporary Files
- Stores temporary files.
- Files in `/tmp` are non-persistent between reboots.

`/proc` - Process Filesystem
- An illusionary filesystem created in memory by the Linux kernel.
- Used to keep track of running processes.

### inodes
* **inode** - index node 
    - it is an object storing meta deta about a file or directory on a given file system
    - It contains:
      - Last time of change, access, modification.
      - Owner/permission data.
      - block-level location data, aka physical disk location of the object data.
    - Directory is a list of names assigned to inodes:
      - Contains entries for itself, its parents, and its children.
    - A disk can run out of inodes before running out of disk space, which prevents new file creation.
    - Inodes can be used to delete strangely named files that won’t tab-complete.
    - **inode related commands**
        - `ls -i` - See the inode number.
        - `find . -inum <inode number>`: Find file by inode number.
        - `df -ih` - see inode usage on a partition level

### Hard and Symbolic Links
- **Hard Link**
    - A link to another file’s inode.
    - Can only occur within the same file system.
    - `ls -la`: Displays hard link count (second column).
    - Directories have a minimum of 2 links: one for itself and one for its parent.
    - Files have a minimum of 1 link.
    - Create hard links with `ln <sourcefile> <destfile>`.
    - Check links with `ls -li`.
    - **Note**: Hard links are not allowed for directories.
  
- **Symbolic (Soft) Link**
    - A link to another file name.
    - Can span across different file systems.
    - Akin to shortcuts in Windows.
    - Useful for linking files between file systems.
    - Create soft links with `ln -s <sourcefile> <destfile>`.

### File localization
* `type`
    - Determines if a command is a shell built-in, alias, or external command.

* `whereis`
    - Outputs the location of a command and its man pages.

* `locate`
    - Locate a file quickly (requires `updatedb`)
    - options:
        - `-r` - use regex
        - `-c` - display or count the bnumber of matching entries found
        - `-e` - return only files that exist at the time of search
        - `-I` - ignmore the casing in file names or paths

* `updatedb`
    - Must be run regularly in order to effectively use `locate`

* `/etc/updatedb.conf`
    - Configuration for `updatedb`, which defines locations not to search.
    - or add them to the $PRUNEPATH variable

- `find` 
    * combine the results with:
        * `-print`      - display the location of the files
        * `-exec`       - execute the comamnd that follows 
        * `-ok`         - execute the command that follow interactively
        * `-delete`     - delete the files found
        * `-fprint`     - tore the results in a target file 

* `which`
    * locate the location of an executable

* `whereis` 
    * Locate the binary, source, and manual page files for a command.
    * gives more info than `which`

- `stat`
    - Display file or file system status 
    - Output includes details like: 
    -   File name 
    -   File size 
    -   Blocks allocated 
    -   IO block size 
    -   File type 
    -   Device ID 
    -   Inode number 
    -   Links 
    -   Access, Modify, and Change timestamps 

### RAID
* RAID types
    * Striping
    * Mirroring
    * Parity
* **RAID configurations**
    * RAID 0 - Striping 
    * RAID 1 - Mirroring
    * RAID 5 - Parity, min. 3 drives
    * RAID 6 - 5+1, min. 4 drives
    * RAID 10 - 1+0, striped mirrors. min. 4 drives

* `mdadm`
    * tool for managing software-based RAID arrays
    * options:
        * `-F` - Activate monitor mode
        * `-f` - Mark specified device
        * `-r` - Remove specified device
        * `-a` - Add device as hot-spare

* `/proc/mdstat`
    * offers information on the current RAID configuration 
    * contains a snapshot of the kernel's RAID/ md state
 

### Swap management
* Swap is a way of emulating RAM on the disk 
* Useful for when you're low on RAM

* `dd if=/dev/zero of=/swapfile bs=1M count=1024`

* `mkswap <file>`
    * Create swap space on a storage partition
    * options:
        * `-c` - check for bad sectors before making the swap space
        * `-p` - Set the page size to be used
        * `-L` - Active swap space using labels applied to partition or file systems

* `swapon <file>`
    * Activate a swap partition

* `swapoff <file>`
    * Deactivate a swap partition

* swap partition
    * `fdisk <disk>`
    * `t` - change the type
    * select a partition
    * enter hex code `82` - this changes the partition type to swap
    * `w` - to write the changes
    * `swapon 


## Partition management

### The storage structure is governed by **three** main layers:
* **Partition table** (GPT/MBT)
    * AKA disklabel or label
    * This defines how a disk is partitioned
    * it is the same for the whole disk 
    * Examples: MBR (old, limited to 2TB) and GPT (modern, supports larger disks and more partitions).
    * Created using `parted mklabel`. See below

* **Partition**
    * Section of the storage drive that logically acts as a separate drive 
    * **Partition types**
        * Primary   - contain one file system or logical drive. Sometimes called a Volume. 
            * Boot partition and swap space are normally created in the primary partition
        * Extended  - contains several file systems which are referrred to as logical drives
        * Logical   - partitioned and allocated as an independent unit and functions as a seperate drive

* **File system**
    * A data structure used by an OS to store, retrieve, organize,  and manage files and riectories on storage devices 
    * Common file systems
        * ext2/3/4
            * Native Linux
        * XFS
            * high-performance journalisting file system
            * fast recovery
            * handles large files very well
        * BTRFS
            * supports huge volumes, 16 exabytes
    * Other file systems
        * ZFS
            * Comparable to btrfs, but not supported by the Linux Kernel, only through fuse
            * see VFS
        * FAT
            * older
            * compatible with most other OS
        * ntfs-ng
            * Windows NRFS compatible file system
        * NFS
            * Network file system
            * Default for file sharing 
        * CIFS
            * Common Internet File System
            * USed to access Windows shares

    * **VFS** - virtual file system
        * VFS is a generic abstraction of all file systems
        * It sits between the kernel and the actual file systems, mediating communication between both
        * It translates the real file system's details over to the kernel
        * Attached to VFS are all the different file system modules
            * XFS, ext4, btrfs
            * One special file system module is fuse - file system in user space
                * fuse enables working with file systems that are not directly supported by the Linux kernel, such as zfs 
        * They will be recognizible by their file system labels
            * These can be up to 16 characters long
            * `e2label` - see or change ext-based file systems
            * `xfs_admin` - same but for XFS-based file systems 

### Managing partitions and file systems 
* `findmnt` - find mount
    * Show all mounts on the system

### Managing disks and partitions
* `lsblk` - list block
    - Lists information about storage devices, such as
        - Device and partition name
        - `MAJ:MIN` - Major Number (Maj): Identifies the device type (e.g., 8 is for SCSI/SATA disks) and Minor Number (Min): Differentiates individual devices or partitions under the same major number.
        - size of the device or partition
        - `RO` - Read-Only value. `0` for read-write and `1` for read-only
        - mount points
    - Partially same info as `fdisk -l`
    - Options:
        - `-a` - list empty devices
        - `-r` - use raw output format
        - `-f` - output info about filesystems, e.g. UUID 
        - `-l` - output list instead of tree
        - `-m` - display device permission info
    * Common disk block device names include:
        * sda - first SCSI/SATA disk
        * vda - virtual disk that uses the KVM virtio driver
        * xvda - virtual disk in a Xen environment
        * nvme0n1 - first nvme disk 
    
* `blkid` - block id
    * print the UUID of devices 
    * this is used to add the devices to `/etc/fstab` 

* `fdisk </dev>` / `gdisk </dev>`
    * Manage partition tables and partitions on a hard disk
    * `fdisk` is for MBR, `gdisk` for GPT
    * **options:**
        * `-l` - list partitions 
        * `-b` - Specify number of drive sectors
        * `-H` - specify number of drive heads
        * `-S` - specify number of sectors per tracks
        * `-s` - print partition size in blocks
    * within the fdisk tool, there are several options:
        * `g` - create gpt disk label type
        * `p` - print disk partition info
        * `n` - create a new partition table
        * `w` - write the changes
        * `q` - cancel and quit

* `parted`
    * Newer standard for table and partition management of disks 
    * Benefits over `fdisk`
        * in contrast to `fdisk`, `parted` allows for resizing of partitions
        * works better for GPT
        * works with large disks
        * allows scripting
    * `mklabel gpt /dev/sd#` - create a new disk / partition table
    * `mkpart <name> <FS-type> <start> <end>` - create a partition
        * <start> and <end> can be in (kilo/mega/giga/tera)bytes or in percentages (10%, 20%, etc)
        * Note: specifiying the filesystem is purely administrative. It can be left out or mismatch the specification by the `mkfs.` command. See below
    * `resizepart <part number> <new size>` - resize a partition

* `partprobe` 
    * Inform the OS of partition table changes
    * This updates the kernels with changes that now exist within the partition table
    * It eliminates the necessity to reboot the system

* `/proc/partitions`
    * contains info about each partition attached to the system
    * contains 
        * major     - class of device
        * minor     - divides partitions into physical devices
        * blocks    - number of physical blocks the partition takes up
        * name      
    * similar to `lsblk  `

* `mount [options] <dev> <mountpoint>`  
    * mount a file system
    * `-t` - specify target
    * `--mkdir` - create directory if not exists
    * `-a` - mount all file systems defined in `/etc/fstab`
    * options are normally added in the `/etc/fstab` file
        * auto      - device must be mounted automatically
        * noauto    - device should not be mounted automatically
        * nouser    - Only root user can mount a device or file system
        * user      - all users can mount 
        * exec      - allow binaries in a file system to be executed
        * noexec    - do not allow binaries in a file system to be executed
        * ro        - mount file system as read only
        * rw        - mount file system with read/write permissions
        * sync      - input and output operations should be done synchronously
        * async     - input and output operations can be done asynchronously

* `umount <mount point>` - unmount
    * unmount a file system
    * options
        * `-f` - force
        * `-l` - perform a lazy unmount. File system is detached from a hierarchy, but refereneces to that file system are not cleaned up until the file system is not being used
        * `-R` - recursively unmounts the directories mount points
        * `-t <fs type>` - unmount all file system types specified
        * `-O` - unmount only the file systems with specified options in the `/etc/fstab` file
        * `-fake` - dryrun it

### Managing file systems
* `wipefs` - wipe file system
    * Wipe the file system using the `--all </dev/>` option
    * It does not wipe the files themselves
        * Use `shred` or `dd` for that

* `mkfs.<type> /dev/sd#` - make file system
    - Build a filesystem on a hard disk partition
        - e.g. `mkfs.ext2/3/4` / `mkfs.xfs` / `mkfs.btrfs`
    - Some options:
        - `-v` - verbose
        - `-c` - check for bad blocks before building the file system

* `/etc/fstab` - file system table
    - Dictates how mounting happens at startup.
    - Describes what devices are mounted, where, and with which options.
    - **components**
        * File system
            * name or UUID of the partition
            * UUID can be found using `blkid`
        * mount points
            * location on the file system where it needs to be mounted
        * type
            * type of file system used by the partition
        * options 
            * set of comma-separated options that will be activated when the file system is mounted
        - **dump**: If enabled, the dump command makes a backup.
        - **pass**: Used by `fsck` to determine the order of file system checks at boot:
            - `0`: No check.
            - `1`: Root disk.
            - `2`: After root disk

* `/etc/crypttab` 
    * A file containing information about encrypted devices and partitions that must be unlocked and mounted on system boot

* `/etc/mtab` 
    * reports the status of currently mounted file systems

* `/proc/mounts`
    * Same but more accurate and includes more up-to-date info on file systems

* `smartctl` - SMART control
    * Self-Monitoring, Analysis, and Reporting Technology
    * Tool for tracking disk drive health, predict failures, and log error and self-tests
    * `--health` - print overall health 
    * `-t (short|long)` - do a disk drive test 
    * `-all` - print all info, including test result

* **ext-specific file system management tools**
    * `e2fsck`
        * file system check

    * `fsck [options] <dev/fs name>`- file system check
        - Check the integrity of a filesystem or repair it
        - **Note**: cannot run on mounted disks.
        - `-a` - immediately repair all damaged blocks
        - `-r` - repair

    * `resize2fs [options] <dev> [<size>]`
        * resize ext2, ext3, or ext4 **file systems**
        * leaving out the size, defaults to going max size
        
    * `tune2fs` 
        - Used to change variable file system parameters
            * add/ remove reserved blocks
            * alter reserve block counts
            * specify number of mounts between chjecks
            * specify the time interval between checks
            * etc  
        - `tune2fs -l` - List all options
        - `tune2fs -l <disk>` - List all details of the disk.
        - `tune2fs <disk> -L <volume name>`: Assign a volume name to a disk.

    * `dumpe2fs` 
        - Displays all super block and block group information about a disk
        - Includes block and superblock data, and metadata about the file system

* **How to's**
    * **So in order to take a used disk and repurpose it:**
        * wipe disk using `dd` (see below). Wiping the file system without deleting the files `wipefs` works as well 
        * create a new drive label with `parted mklabel` and a new partition using `parted mkpart`
        * format the partition `mkfs.<type>`
        * add formatted partition to `/etc/fstab` so it can configured by the system to run at boot up
        * run `partprobe` if needing to use the drive immediately without rebooting
        * mount it using `mount`

    * **Remove swap partition and expand root partition**
        * `swapoff <swap space>`
        * `fdisk` 
            * `d <swap partition>`
        * `parted` - you cannot resize partition in `fdisk`
            * `resizepart <partition number> <100%>` - now the partition is resizes
        * `resize2fs <dev>` - after expanding the partition, the file system must be expanded


### XFS-specific file system management tools
* `xfs_info`        - display details 
* `xfs_admin`       - change the parameters 
* `xfs_metadump`    -  Copy the superblock metadata to a file 
* `xfs_growfs`      -  expand the XFS file system to fill the drive size
* `xfs_copy`        -  copy the contents of the XFS file system to another location
* `xfs_repair`      -  repair and recover a corrupt XFS file system
* `xfs_db`          -  Debug the XFS file system



## Logical Volume Manager
* LVM - Logical Volume Manager
    * Tool for mapping whole physical devices and partitions into one or more virtual containers called **volume groups**
    * Within these volume groups, are one or more logical volumes
    * It is analogous to Storage Spaces in Windows
    * So
        * Bunch of disks
        * Partitions on those disks
        * Physical volumes
        * Volume Group comprising of multiple of these physical volumes
        * Virtual Volumes carved from the Volume Group
        * File Systems created from these Virtual Volumes
    * `/dev/mapper` contains all the logical volumes on a given system managed by LVM

* Physical volume tools
    * `pvscan`      - scan for all physical devices being used as physical volumes
    * `pvcreate`    - Initializes a drive or partition to use as a physical volume
        * It all starts with this
        * If creating a physical volume from a physical drive, the partitions and label must be wiped first
    * `pvdisplay`   - lists attributes of physical volumes
    * `pvchange`    - changes attributes of a physical volumes
    * `pvs`         - displays information about physical volumes
    * `pvck`        - checks the metadata of physical volumes
    * `pvremove`    - removes physical volumes

* Volume group tools
    * `vgscan`      - scan all physical devices for volume groups
    * `vgcreate`    - creates volumes groups
    * `vgdisplay`   - lists attributes of volume groups
    * `vgchange`    - changes attributes of volume groups
    * `vgs`         - displays information about volume groups
    * `vgck`        - checks the metadata of volume groups
    * `vgrename`    - renames a volume group
    * `vgreduce`    - removes physical volumes from a group to reduce its size
    * `vgextend`    - add physical volumes from a group to enlarge its size
    * `vgmerge`     - merges two volume groups
    * `vgsplit`     - splits a volume group into two
    * `vgremove`    - removes volume groups

* Logical volume tools
    * `lvscan`      - scan for all physical devices for logical volumes
    * `lvcreate`    - creates logical volumes in a volume group 
    * `lvdisplay`   - lists attributes of logical volumes
    * `lvchange`    - changes attributes of a physical volumes
    * `lvs`         - displays information about logical volumes
    * `lvrename`    - renames a logical volume
    * `lvreduce`    - reduces the size of logical volumes
    * `lvextend`    - extends the size of logical volumes 
    * `lvresize`    - resizes logical volumes
    * `lvremove`    - removes a logical volume


## Archiving, Backup & Recovery

- **`tar`** - Tape Archiver
    - Bundles together multiple files into a single tarball 
    - Common options:
        - `-c` - Create an archive.
        - `-x` - Extract file
        - `-t` - List archive contents.
        - `-f` - Specify the filename.
        - `-v` - Verbose (list files being processed).
        - `-z` - Compress or decompress with gzip
        - `-C` - Specify the output file
    - **Examples:**
        * `tar -cvf <file>.tar <files>` - Create a tarball from specified files  
        * `tar -xvf <file>.tar` - Extract **all** files in tarball
            * `tar -xvf <file>.tar <files>` - Extract only the mentioned files from tarball
            * `tar -xvf <file>.tar -C <directory>` - Extract the tarball into the specified directory 
        * `tar -tvf <file>.tar` - List files in tarball

* **Compression tools**:
    - **`gzip`**
        - Most common utility
        - Nice balance between compression ratio and take to compress
        - Compression tool that produces files with `.gz` extension 
        - Use with `tar` with `-z` option
            - `tar -czvf <file>.tar <files>` - archive and compress
        
    * **`bzip2`**
        * Better compression, but takes longer 
        * `-j` - when used as a option for `tar`
            * `tar -cjvf <file>.tar <files>` - archive and compress
             
    * **`xz`**
        * Best compression, but slow, like 10x slower than `gzip` for a +-20% better compression
        * `-J` - when used as a option for `tar`
            * `tar -cJvf <file>.tar <files>` - archive and compress
            
- **`dd`** - data duplicator / disk destroyer
    * Commonly used for disk imaging, data recovery, and even wiping data securely
    * Command components
        * `if={file name}` - Specify input file 
        * `of={file name}` - Specify output file
        * `bs={bytes}`     - Specify total block size to read and write in bytes
        * `count={count}`  - Specify the number of blocks to be written. Optional
        * `status={level}` - Specify information to print to standard error
    * Command examples
        * `dd if=/dev/sdX of=/path/to/backup.img bs=4M status=progress` 
            * Create a disk image
        * `dd if=/dev/zero of=/dev/sd# bs=1M status=progress`
            * Secure wipe a disk  
        * `dd if=/dev/sda of=/backup.bak bs=446 count=1`
            * Copy or backup a disk to a file
        * `dd if=/dev/sda of=/dev/sdb`
            * Copy or backup a disk to another disk   
    * Other things you do with `dd`
        * Converting files to lowercase
        * Disk performance tests
            * measuring read / write speed


* `mirrorvg` - Mirror volume group
    * 




## Storage Troubleshooting


- **`df`**: - display filesystem
    * Print filesystem usage.
    * `-h` - human-readable format (e.g., MiBs, GiBs).
    * `-i` - show inodes 

* `du` - disk usage 
    * `-h` - human readable 
    * `-s` - show only total 

* **`quotas`** 
    * Package for managing user quotas
    * `quotacheck`: Scan a file system for disk usage and create/repair quota files.
    * `edquota`: Edit the disk quota for a user.
        - Specifies disk usage, inode usage, and the specific file system to which it applies.
    * `quotaon`: Enable quotas and apply the configuration set in `edquota`.
    * To implement quotas:
        * modify `/etc/fstab` file
        * add `,usrquota` after `defaults`


 
 ## Remote File System Access





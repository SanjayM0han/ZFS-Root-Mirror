# ZFS-Root-Mirror
**Installation on Ubuntu 24.04**
This adds a mirror to an existing Ubuntu ZFS boot drive

## HOW

### Initial Ubuntu installation

+ Install Ubuntu Desktop 24.04 

+ For the "**Erase disk and install Ubuntu**" option, click "**Advanced Features**" and choose "**Experimental ZFS**"

+ Continue install as normal and boot into Ubuntu....

### Add second drive

All work will be done from CLI. Open a Terminal.

+ Update Ubuntu:
    
      sudo apt update && sudo apt dist-upgrade
+ Find the names of your two disks , 
The first disk will have three / four partitions, (Four , if you have swap memmory)
  
      ls -l /dev/disk/by-id

![VirtualBox_ubuntu_05_07_2024_11_49_11](https://github.com/kashinathshabu/ZFS-Root-Mirror/assets/67222565/7231b13d-ea7b-4794-bd6b-b8620c48c84f)
![VirtualBox_ubuntu_05_07_2024_10_46_39](https://github.com/kashinathshabu/ZFS-Root-Mirror/assets/67222565/b7af13ee-f8f6-4307-8baf-45c4cd6a9c01)

+ Let's set variables for those disk paths so we can refer to them in the following

      DISK1=/dev/disk/by-id/scsi-disk1
      DISK2=/dev/disk/by-id/scsi-disk2
![VirtualBox_ubuntu_05_07_2024_11_36_44(2)](https://github.com/kashinathshabu/ZFS-Root-Mirror/assets/67222565/ad6a7bfe-3945-4375-bec7-823516c1d136)



### Create partitions on second drive

+ List partitions , you expect to see three or four of them 

      sudo sgdisk -p $DISK1
![VirtualBox_ubuntu_05_07_2024_11_36_44](https://github.com/kashinathshabu/ZFS-Root-Mirror/assets/67222565/b6b096e7-7232-4d60-bd8e-e5a9ce823e88)

If three:
```
BIOS-Boot or EFI
boot
root
```

If four:
```
BIOS-Boot or EFI
swap
boot
root
```

+ Copy partition table from disk 1 to disk 2: 

      sudo sgdisk -R$DISK2 $DISK1

### Mirror boot pool
+ Confirm that disk 1 partition 2 is the device in the **bpool** by comparing "Partition by-ID" to the device name shown in zpool status: 
`sudo sgdisk -i2 $DISK1` and `zpool status bpool`

+ Add that partition to the pool:  *for example* `sudo zpool attach bpool DISK-PART2 /dev/disk/by-id/DISK_PART2`
  
        sudo zpool attach bpool EXISTING-ID /dev/disk/by-id/DISK2-part2, 

+ Verify with `zpool status bpool`. You expect to see mirror-0 now, which has been resilvered
  ![bpool](https://github.com/kashinathshabu/ZFS-Root-Mirror/assets/67222565/c4707a16-0ecb-4cf9-bd87-fbc802930406)


### Mirror root pool

+ Confirm that disk 1 partition 3 is the device in the **rpool** by comparing "Partition by-ID" to the device nmae shown in zpool status:
  `sudo sgdisk -i3 $DISK1` and `zpool status rpool`
  
+ Add that partition to the pool: , *for example* `sudo zpool attach rpool DISK-PART3 /dev/disk/by-id/DISK-PART3`

          sudo zpool attach rpool EXISTING-ID /dev/disk/by-id/DISK2-PART3
  
+ Verify with `zpool status` rpool. You expect to see mirror-0 now, which either is resilvering or has been resilvered
![rpool](https://github.com/kashinathshabu/ZFS-Root-Mirror/assets/67222565/0e45bf41-0fd8-4970-9fbe-96e6d867a2e4)


### Install GRUB
Install GRUB on both the disk by `sudo grub-install /dev/sda` and `sudo grub-install /dev/sdb`

### DONE !!
*Try removing HDD1 and boot the machine  or remove HDD1 on the live running machine and test does it automatically works with HDD2*

# ZFS-Root-Mirror

This adds a mirror to an existing Ubuntu ZFS boot drive

## HOW

### Initial Ubuntu installation

• Install from an Ubuntu Desktop 20.04 or later install USB. Ubuntu Server does not offer ZFS boot disk

• For the "Erase disk and install Ubuntu" option, click "Advanced Features" and choose "Experimental ZFS"

• Continue install as normal and boot into Ubuntu****

### Add second drive

All work will be done from CLI. Open a Terminal.

Update Ubuntu:
    
    sudo apt update && sudo apt dist-upgrade
Find the names of your two disks , 
The first disk will have three / four partitions, (Four , if you have swap memmory)
  
    ls -l /dev/disk/by-id. 

![VirtualBox_ubuntu_05_07_2024_10_46_39](https://github.com/kashinathshabu/ZFS-Root-Mirror/assets/67222565/79304947-f4a5-413c-8174-bfe2c5aa8935)



  
  Let's set variables for those disk paths so we can refer to them in the following

    DISK1=/dev/disk/by-id/scsi-disk1
    DISK2=/dev/disk/by-id/scsi-disk2

### Create partitions on second drive

List partitions , you expect to see three or four of them 

    sudo sgdisk -p $DISK1
Copy partition table from disk 1 to disk 2: 

    sudo sgdisk -R$DISK2 $DISK1

### Mirror boot pool
Confirm that disk 1 partition 2 is the device in the bpool by comparing "Partition by-ID" to the device name shown in zpool status: 

sudo sgdisk -i2 $DISK1 and zpool status bpool

Get GUID of partition 3 on disk 2: 

sudo sgdisk -i3 $DISK2

Add that partition to the pool: 

sudo zpool attach bpool EXISTING-UID /dev/disk/by-partuuid/DISK2-PART3-GUID, 

for example sudo zpool attach bpool ac78ee0c-2d8d-3641-97dc-eb8b50abd492 /dev/disk/by-partuuid/8e1830b3-4e59-459c-9c02-a09c80428052

Verify with zpool status bpool. You expect to see mirror-0 now, which has been resilvered



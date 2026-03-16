Task 1: Check Current Storage

```
Run: lsblk, pvs, vgs, lvs, df -h

$ lsblk => list blocks

$ pvs => show physical volumes

$ vgs => show volume groups

$ lvs => show logical volume 

$ df -h
```
<img width="757" height="260" alt="image" src="https://github.com/user-attachments/assets/cd03d210-2479-4e6e-b0dc-d4f4ee94f9ba" />
<img width="757" height="212" alt="image" src="https://github.com/user-attachments/assets/45459e9d-eac7-4003-9dc8-03d686277a69" />

Task 2: Create Physical Volume

```
$ pvcreate /dev/xvdx /dev/xvdy /dev/xvdz

$pvs

```
<img width="757" height="297" alt="image" src="https://github.com/user-attachments/assets/28ae1535-c257-4b81-bfcb-5323e3d60576" />
<img width="757" height="259" alt="image" src="https://github.com/user-attachments/assets/0dcddf64-b9dc-4571-9325-83bb8bd9c8f4" />



Task 3: Create Volume Group

```
$ vgcreate umesh-vg /dev/xvdx /dev/xvdy

Volume group "umesh-vg" successfully created

$ vgs

VG       #PV #LV #SN Attr   VSize  VFree 
umesh-vg   2   0   0 wz--n- 21.99g 21.99g

```
<img width="757" height="154" alt="image" src="https://github.com/user-attachments/assets/410add27-6d0f-475f-827c-ffff226f0d6a" />


Task 4: Create Logical Volume

```
$ lvcreate -L 10G -n umesh-lv umesh-vg

$ lvs

$ pvdisplay => physical volume information

$ vgdisplay => volume group information

$ lvdisplay => logical volume information
```
<img width="647" height="54" alt="image" src="https://github.com/user-attachments/assets/251948f2-f46d-4403-8845-8ce1ecbc3a17" />

<img width="1051" height="75" alt="image" src="https://github.com/user-attachments/assets/0b2118b6-8937-4d73-8312-42b0cc36a6aa" />

Task 5: Format and Mount

```
$ mkfs.ext4 /dev/umesh-vg/umesh-lv

mke2fs 1.47.0 (5-Feb-2023)
Creating filesystem with 2621440 4k blocks and 655360 inodes
Filesystem UUID: db4eb75c-a68b-4815-9b56-ec752ef11932
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Allocating group tables: done                            
Writing inode tables: done                            
Writing superblocks and filesystem accounting information: done

$ mkdir -p /mnt/umesh-mount

$ mount /dev/umesh-vg/umesh-lv /mnt/umesh-mount

$ df -h /mnt/umesh-mount

xvdx                  202:5888  0   10G  0 disk 
xvdy                  202:6144  0   12G  0 disk 
└─umesh--vg-umesh--lv 252:0     0   10G  0 lvm  /mnt/umesh-mount
xvdz                  202:6400  0   14G  0 disk 

```
<img width="1769" height="635" alt="image" src="https://github.com/user-attachments/assets/e4275156-c77a-4da0-98ed-e6bce6b77c28" />

<img width="953" height="236" alt="image" src="https://github.com/user-attachments/assets/48e57228-33f0-453a-a725-f32bc0102693" />

<img width="953" height="134" alt="image" src="https://github.com/user-attachments/assets/6c8a6f0e-30ee-41ba-be6e-602878345a6a" />

Task 6: Extend the Volume

```
$ lvextend -L +5G /dev/umesh-vg/umesh-lv 

$ resize2fs /dev/umesh-vg-umesh-lv

$ df -h /mnt/umesh-mount

```
<img width="1294" height="360" alt="image" src="https://github.com/user-attachments/assets/3c30c3ee-df93-4384-9f44-a51258711fb4" />

<img width="1294" height="360" alt="image" src="https://github.com/user-attachments/assets/54444a7e-1226-4293-8e7d-80290c8fc10c" />


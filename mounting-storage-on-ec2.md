# Mounting and modifying storage on an EC2 instance

I set up an Ubuntu 18.04 server with 9 TB additional EBS volume, which is what
we'll use in this example.

## Setting up the storage added when configuring the instance

Log in to the instance. Make the additional storage available as /opt:

     ubuntu@ip-10-20-30-400:~$ lsblk                           # output shows extra 9 TB storage is at nvme0n1
     ubuntu@ip-10-20-30-400:~$ sudo mkfs -t xfs /dev/nvme0n1   # prep as xfs
     ubuntu@ip-10-20-30-400:~$ sudo mkdir /opt                 # if needed
     ubuntu@ip-10-20-30-400:~$ sudo mount /dev/nvme0n1 /opt    # mount as /opt

To ensure the mount is preserved on reboot we'll modify /etc/fstab. First, run
lsblk again

     ubuntu@ip-10-20-30-400:~$ sudo lsblk -o +UUID

and harvest the UUID from the output, in this case
abcd131c-1234-451e-8d34-ec98989891ae. Update /etc/fstab:

     ubuntu@ip-10-20-30-400:~$ sudo vi /etc/fstab

Add the line

     UUID=abcd131c-1234-451e-8d34-ec98989891ae /opt xfs defaults,nofail 0 2

## Resizing attached storage

I had to resize the EBS volume from 9 TB to 12 TB. I'm grateful to Ryan D. Smith
for making the process clear for me. The steps were:

1.  On the host, unmount the volume.
2.  From the AWS Console, expand the volume.
3.  On the host, expand the filesystem.
4.  Remount the volume.

### Unmount the volume

Use lsblk to see what we're dealing with:

     ubuntu@ip-10-20-30-400:~$ lsblk
     NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
     loop0         7:0    0   18M  1 loop /snap/amazon-ssm-agent/1566
     loop1         7:1    0 96.6M  1 loop /snap/core/9804
     loop2         7:2    0 28.1M  1 loop /snap/amazon-ssm-agent/2012
     loop4         7:4    0 97.1M  1 loop /snap/core/9993
     nvme0n1     259:0    0  8.8T  0 disk /opt
     nvme1n1     259:1    0    8G  0 disk
     └─nvme1n1p1 259:2    0    8G  0 part /

/dev/nvme0n1 has the mount point /opt. Unmount the volume:

     ubuntu@ip-10-20-30-400:~$ sudo umount --verbose /opt
     umount: /opt unmounted

### Expand the volume

Go to the AWS console's EC2 dashboard. In the navigation menu (left side of the
page) select Elastic Block Store >> Volumes, then:

*   Click on the target volume.
*   Under the Actions menu, select Modify Volume.
*   Modify the volume.

The update can take a while to complete, in this example about ten hours. Its
status in the AWS Console will be marked as "in-use - optimizing."

There's guidance at
https://docs.aws.amazon.com/cli/latest/reference/ec2/modify-volume.html from
AWS.

### Expand the filesystem

Once the volume is expanded, the associated Linux filesystem needs to be
expanded too. On the host, remount the volume at /opt. Since we configured the
instance to mount the volume after a reboot, it's easiest to do

     ubuntu@ip-10-20-30-400:~$ sudo reboot

and to log in again.

The volume is now mounted at /opt, and we see a difference between lsblk and df
in the space reported as being available:

     ubuntu@ip-10-20-30-400:~$ lsblk
     NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
     loop0         7:0    0 28.1M  1 loop /snap/amazon-ssm-agent/2012
     loop1         7:1    0 97.1M  1 loop /snap/core/9993
     loop2         7:2    0   18M  1 loop /snap/amazon-ssm-agent/1566
     loop3         7:3    0 96.6M  1 loop /snap/core/9804
     nvme0n1     259:0    0 11.7T  0 disk /opt
     nvme1n1     259:1    0    8G  0 disk
     └─nvme1n1p1 259:2    0    8G  0 part /

     ubuntu@ip-10-20-30-400:~$ df -h /opt
     Filesystem      Size  Used Avail Use% Mounted on
     /dev/nvme0n1    8.8T  2.5T  6.4T  28% /opt

Reconcile this using xfs\_growfs:

     ubuntu@ip-10-20-30-400:~$ sudo xfs_growfs -d /opt
     meta-data=/dev/nvme0n1           isize=512    agcount=12, agsize=196608000 blks
      . . .

     ubuntu@ip-10-20-30-400:~$ df -h /opt
     Filesystem      Size  Used Avail Use% Mounted on
     /dev/nvme0n1     12T  2.5T  9.3T  21% /opt

See
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/recognize-expanded-volume-linux.html
for more info, particularly if you've set up partitions.

Should be good now.

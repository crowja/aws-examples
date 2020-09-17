# Mounting storage on an EC2 instance

## On the instance, set up storage added when configuring the instance

An Ubuntu 18.04 server has been set up with 10 TB additional EBS storage. First
make it available as /opt.

     ubuntu@ip-10-20-30-400:~$ lsblk                           # output shows extra 10 TB storage is at nvme0n1
     ubuntu@ip-10-20-30-400:~$ sudo mkfs -t xfs /dev/nvme0n1   # prep as xfs
     ubuntu@ip-10-20-30-400:~$ sudo mkdir /opt                 # if needed
     ubuntu@ip-10-20-30-400:~$ sudo mount /dev/nvme0n1 /opt    # mount as /opt

Now ensure the mount is preserved on reboot. Run lsblk:

     ubuntu@ip-10-20-30-400:~$ sudo lsblk -o +UUID

Harvest the UUID, for example xxxx131c-1234-451e-8d34-ec98989891ae, and update
/etc/fstab as follows:

     ubuntu@ip-10-20-30-400:~$ sudo vi /etc/fstab

and add the line:

     UUID=xxxx131c-1234-451e-8d34-ec98989891ae /opt xfs defaults,nofail 0 2

## Resizing attached storage

I had to resize the EBS volume. I'm grateful to Ryan D. Smith for making the
process clear for me. The basic steps are:

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
page) select Elastic Block Store >> Volumes. Then:

*   Click on the target volume.
*   Under the Actions menu, select Modify Volume.
*   Modify the volume.

The update can take a while to complete. Its status will be marked as "in-use -
optimizing."

Guidance from AWS at
https://docs.aws.amazon.com/cli/latest/reference/ec2/modify-volume.html.

### Expand the filesystem

Once the volume is expanded, the associated Linux filesystem needs to be
expanded too. On the host ***TBD***.

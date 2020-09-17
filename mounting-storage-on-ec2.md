# Mounting storage on an EC2 instance

## On the instance, set up storage added when configuring the instance

An Ubuntu 18.04 server has been set up with 10 TB additional EBS storage. First
make it available as /opt.

     $ lsblk                           # output shows extra 10 TB storage is at nvme0n1
     $ sudo mkfs -t xfs /dev/nvme0n1   # prep as xfs
     $ sudo mkdir /opt                 # if needed
     $ sudo mount /dev/nvme0n1 /opt    # mount as /opt

Now ensure the mount is preserved on reboot. Run lsblk:

     $ sudo lsblk -o +UUID

Harvest the UUID, for example xxxx131c-1234-451e-8d34-ec98989891ae, and update
/etc/fstab as follows:

     $ sudo vi /etc/fstab

and add the line:

     UUID=xxxx131c-1234-451e-8d34-ec98989891ae /opt xfs defaults,nofail 0 2


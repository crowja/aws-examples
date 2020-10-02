# Basic software on an EC2 instance

These notes are for an Ubuntu system.

## Basic software

Install a few helpers

     $ sudo apt update
     $ sudo apt upgrade
     $ sudo apt install build-essential
     $ sudo apt install manpages-dev
     $ sudo apt install unzip
     $ sudo apt install screen
     $ sudo apt install zlib1g-dev

If useful to have Miniconda

     $ wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh

Make the download executable and run it. Update conda:

     $ conda update conda

## To change the instance's hostname

     $ sudo vi /etc/hostname
     [Change the entry to the new hostname]
     . . .
     $ sudo vi /etc/cloud/cloud.cfg
     [Change "preserve_hostname: false" to "preserve_hostname: true"]

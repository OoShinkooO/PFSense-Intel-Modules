# Background Information

This is a repository for the compiled intel drivers for PFsense 2.7.0
PFsense Uses Kernel Source 14-Current with branch 0c59e0b4e581

## Steps to reproduce

1. Download FreeBSD either ISO or VM-Image

- For ISO Images: <https://download.freebsd.org/snapshots/amd64/amd64/ISO-IMAGES/14.0/>
- For VM Images: <https://download.freebsd.org/snapshots/VM-IMAGES/14.0-CURRENT/amd64/Latest/>

(I used VM Image: [FreeBSD-14.0-CURRENT-amd64.vmdk.xz](https://download.freebsd.org/snapshots/VM-IMAGES/14.0-CURRENT/amd64/Latest/FreeBSD-14.0-CURRENT-amd64.vmdk.xz))

### Configure FreeBSD

#### Creating Virtual Machine

1. Expand the Physical hard drive from 4 GB, to 10 GB in which ever virtualization software.
2. Turn on Virtual Machine and access Single User Mode (S)
3. Press Enter for shell path
4. List Partitions

        gpart show
        # Look for Partition Labeled "freebsd-ufs"

5. extend "freebsd-ufs" partition to full size 10G

        gpart resize -i 4 -s 8G da0

6. Extend "freebsd-ufs" filesystem

        growfs /
        yes

7. reboot

        reboot

8. create user account

        adduser
        #Add User Account to group wheel

9. Enable SSHD

        echo "sshd_enable=\"YES\"" >> /etc/rc.conf
        /etc/rc.d/sshd start

10. SSH into Machine to finishin working on it.

### Download Kernel Source Code

1. Switch to root

        su -

2. Install git

        pkg install git
        # The package management tool is not yet installed on your system.
        # Do you want to fetch and install it now? [y/N]: y

        # The process will require 239 MiB more space.
        # 46 MiB to be downloaded.

        # Proceed with this action? [y/N]: y

3. Download Kernel Source and change to that directory for with following command:  

        git clone -b main  https://git.freebsd.org/src.git /usr/src && cd /usr/src

4. Check out Kernel Source to the appproiate version.  

        git checkout 0c59e0b4e581

5. exit out

        exit
        # Prompt should change from root to regular user

### Download Driver Source Code and Patch

1. Make a directory within your home called Downloads and cd into it.

        mkdir ~/Downloads && cd ~/Downloads

2. Download the Driver and extract it:

        fetch https://downloadmirror.intel.com/732263/igb-2.5.24.tar.gz && tar -xvzf igb-2.5.24.tar.gz && cd igb-2.5.24/src

3. Download Patch:

        fetch https://cgit.freebsd.org/ports/plain/net/intel-igb-kmod/files/patch-if__igb.c

4. Apply Patch:

        patch if_igb.c patch-if__igb.c 

5. Run Make to create the file:

        make

### Transfer files from FreeBSD Machine to PFsense

1. Copy File using a different terminal locally:

        scp $username@$ipaddress:~/Downloads/igb-2.5.24/src/if_igb.ko .

2. Upload file to PFsense Machine:

        scp ./if_igb.ko root@$pfsense:/boot/kernel/

### Update Pfsense Configuration to use Kernel Module:

1. Login into PFSense machine:

        ssh root@pfsenseip
        # Choose Option 8 for command prompt

2. Create / update /boot/loader.conf.local to load kernel module.

        echo 'if_igb_load="YES"' >> /boot/loader.conf.local

3. Reboot and check with kldstat to make sure kernel module is loaded

        reboot
        ssh root@pfsenseip
        kdlstat

4. Exit out of PFsense

        exit
        #Choice Option 0 to exit

## Sources

<https://ostechnix.com/how-to-enable-ssh-on-freebsd/>  
<https://github.com/MonkWho/pfatt/issues/67>  
<https://forums.freebsd.org/threads/unable-to-compile-network-driver-under-freebsd14-0-current.89396/>  
<https://www.reddit.com/r/PFSENSE/comments/14mdp9c/pfsense_ce_270_software_and_pfsense_plus_23051/>

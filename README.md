# ansible-disks-aws
This role formats a list of disks with a single partition each that fills up 100% of the disk. Works on RedHat and Debian.

##### Instructions
1. Define a var `partitions` with an entry for each disk that you wish to provision.  The structure should be as follows:
```
partitions:
  - number: "1"           #Number of the partition
    device: "/dev/xvdg"   #Disk on which the partition will be made
    diskstate: "present"
    fstype: "ext4"        #Defines disk filesystem
    mountpoint: "/u01"    #Ensures mountpoint dir exists
    mountstate: "mounted" #Mounts/unmounts partition and updates fstab
```


#####  Task Overview
1. Gathers facts, including "ansible_product_name" which shows the AWS instance type (e.g. c5.large)
2. Grabs the first part of the ansible_product_name string which corresponds to the AWS instance family) and checks whether it's in `nvmelist`, a list of all instance types using NVMe disks as of May 2019 (See below for why this is needed)
    - If true: sets var `nvme_bit` to `p`
    - If false: leaves var `nvme_bit` as an empty string
3. Iterates through the list entries in partitions to provision disks.


##### Assumptions
1. A "partitions" list of dictionaries is defined.



##### Standard vs NVMe SSDs

This role also supports NVMe SSDs using the "nvme_bit" variable. For normal SSDs, partition names are composed of [block device name] + [partition number], e.g.:
```
sdg   + 1  = sdg1
xvdb  + 2  = xvdb2
```
For NVMe disks, there is an extra "p" to denote the partition. Partition names are thus created by concatenating [block device name] + ["p"] + [partition number], e.g.
```
nvme0n1 + p + 2 = nvme0n1p2
nvme1n1 + p + 1 = nvme1n1p1
```
By default nvme_bit is an empty string.  Ansible will check the EC2 instance family, and if it is in a family that uses NVMe disks, it will set nvme_bit to "p".


# Taking a Snapshot of a Bare-Metal Instance in OpenStack to create a image (in Ubuntu 14.04)

Taking Snapshot of a Bare-Metal Instance in OpenStack is a little convoluted process as opposed to taking snapshots of a Virtual Machine from a OpenStack dashboard

Snapshots of a Instance/VM are usually taken to save the state of the instance and use that instance as a image to further spin-off different instances. For example, if one wants to install some kind of software like Docker/DevStack onto a Bare-Metal instance and want to share this software with others in a cloud setting. It will be very easy by taking snapshot of that particular Bare-Metal instance and creating a image using it. So whenever someone else wants to use it, this image can be used to spin an instance.  

In the below guide Google's Machine Learning library --> TensorFlow is used as an example.
https://www.tensorflow.org/ 

Below is a step-by-step guide which explains the process of creating an image by taking snapshot of a Bare-Metal Instance

## Step 1. Logging onto a Bare-Metal instance using ssh
```
ssh -i ~/.ssh/public_key.pub username@ipaddress
```

## Step 2. Change to a root user
```
sudo -i
```

## Step 3. update and upgrade the ubuntu packages 
```
apt-get update
apt-get upgrade
```

## Step 4. Installing TensorFlow library
```
apt-get install python-pip python-dev
sudo pip install --upgrade 
https://storage.googleapis.com/tensorflow/linux/cpu/tensorflow-0.5.0-cp27-none-linux_x86_64.whl
```

## Step 5. Taking a snapshot of the Bare Metal Instance
#### Below command creates a snapshot.tar file in /tmp/ folder
```
tar cf /tmp/snapshot.tar / --selinux --acls --xattrs --numeric-owner --one-file-system --exclude=/tmp/* --exclude=/proc/* --exclude=/boot/extlinux 
```
### Verify if the ```snapshot.tar``` is created
```
cd /tmp
ls
```

## Step 6. Install the libguests-tools 
(BELOW STEPS ARE IMPORTANT)
```
apt-get install libguestfs-tools (A pink screen will appear while installation, select the option as "yes")
apt-get install kvm libvirt-bin virt-manager virt-viewer virt-top virt-what
apt-get install ubuntu-virt virt-top virt-what
update-guestfs-appliance
```

## Step 7. Below command creates a qcow2 image
```
virt-make-fs --format=qcow2 --type=ext4 --label=img-rootfs /tmp/snapshot.tar /tmp/snapshot.qcow2
```
### Now verify in the tmp folder again if a file called snapshot.qcow2 is being created
```
cd /tmp
ls
```

## Step 8. Take a snapshot of the image
```
virt-sysprep -a /tmp/snapshot.qcow2
```

## Step 9. Compress the created qcow2 image
```
qemu-img convert /tmp/snapshot.qcow2 -O qcow2 /tmp/snapshot_compressed.qcow2 -c
```

## Step 10. Install the glance-client (if it is not available)
```
apt-get install python-glanceclient
```

## Step 11. The final steps are to upload your snapshot image to OpenStack Glance. First, visit the Access & Security tab in the OpenStack web interface and "Download OpenStack RC File". Copy this file to your instance and source it. 
```
vi openrc.sh 
```
(copy and paste the openrc file inside this file and save it)
```
chmod +x openrc.sh 
source openrc.sh 
```

*** (Just incase enter the password to your Bare-Metal password )***

## Step 12. Then simply use the glance client program to view your image. 
### First, find the uuids of the kernel and ramdisk by typing: 
```
 glance image-list
```
---
ID |Name|Disk Format|Container Format|Size|Status|
---|----|-----------|----------------|----|------|
 8ad7e4f4-7204-4aec-bd9d-4348ee5e7ddf | ubuntu-14-04               | qcow2       | bare             | 336656384  | active |
| 02eded7f-1ea8-4011-afe3-51460b30c2e4 | ubuntu-14-04-initrd        | ari         | ari              | 24696115   | active|
| bbe9bbc8-d90c-41d6-9cbd-85a6fce43bb2 | ubuntu-14-04-kernel        | aki         | aki              | 5820640    | active|

---

### copy the aki id  -> bbe9bbc8-d90c-41d6-9cbd-85a6fce43bb2 and paste it in place of the kernel_id just like shown in below command.
### copy the ari id  -> 02eded7f-1ea8-4011-afe3-51460b30c2e4 and paste it in place of the ram_id just like shown in below command

## Step 13. Create 
```
glance image-create --name my-snapshot --disk-format qcow2 --container-format bare --property kernel_id=bbe9bbc8-d90c-41d6-9cbd-85a6fce43bb2 --property ramdisk_id=02eded7f-1ea8-4011-afe3-51460b30c2e4 < /tmp/snapshot_compressed.qcow2
```
 



















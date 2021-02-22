# challenge
example project for creation of 2 VM Centos8 with Docker e Swarm, with 40GB each for Doker partition


## prerequisites

 Linux SO with KVM
 
### 1-Step prepare VM Centos 8 template image
I started from Centos 8 Cloud image but it is only 8GB so I use cloud-init script to customize the image to ssh into with my rsa key.
following simple step to reproduce the image (qcow2) :
```
   #set your work directory
   WORKDIR=<<your_workdir_path>>
   
   #set your VM name
   VM=Centos-8
   
   #create directory named as image in your workdir
   mkdir -vp $WORKDIR/$VM
   
   #enter inside
   cd $WORKDIR/$VM

   #Download image from centos cloud image
   wget https://cloud.centos.org/centos/8/x86_64/images/CentOS-8-GenericCloud-8.3.2011-20201204.2.x86_64.qcow2

   mv CentOS-8-GenericCloud-8.3.2011-20201204.2.x86_64.qcow2 $VM.qcow2

   #set LIBGUESTFS_BACKEND env variable
   export LIBGUESTFS_BACKEND=direct
   
   #create 60GB allocation for VM in $VM.new.image file
   qemu-img create -f qcow2 -o preallocation=metadata $VM.new.image 60G

   #resize the image into new image
   virt-resize --quiet --expand /dev/sda1 $VM.qcow2 $VM.new.image
   
   #rename new image overwriting the original one
   mv $VM.new.image $VM.qcow2
   
   #create cloud-init iso file with system customization
   mkisofs -o $VM-cidata.iso -V cidata -J -r user-data meta-data
   
   #create virsh pool
   virsh pool-create-as --name $VM --type dir --target $D/$VM
   
   #install nad start VM mounting iso generated in previous step as cdrom for cloud-init
   virt-install --import --name $VM --memory 4096 --vcpus 2 --cpu host --disk $VM.qcow2,format=qcow2,bus=virtio --disk $VM-cidata.iso,device=cdrom --network bridge=virbr0,model=virtio --os-type=linux --os-variant=centos8 --graphics spice --noautoconsole
   
   #install vagrant
   sudo dnf install vagrant
   
   #install vagrant-libvirt for KVM
   sudo dnf install vagrant-libvirt
   
   #dowload utility to convert qcow2 row image into vagrant box
   wget https://raw.githubusercontent.com/vagrant-libvirt/vagrant-libvirt/master/tools/create_box.sh
   
   #give it permission
   chmod 775 create_box.sh 
   
   #create box file
   ./create_box.sh $VM.qcow2
   
   #import created box into vagrant box modify your VM name from "gcryptolo/Centos-8" into your name
   vagrant box add gcryptolo/Centos-8 $VM.box
   
   #init the image (change to your VM name)
   vagrant init gcryptolo/Centos-8
 ```

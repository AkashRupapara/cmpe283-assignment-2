# cmpe283-assignment-2
The task is carried out on a Google Cloud VM that supports nested virtualization (`—enable-nested-virtualization`).

Configuration:
  - CPU PLatform: Intel Cascade Lake 
  - Machine Type: n2-standard-8

Assignment 2 modifies the behavior of the cpuid instruction for the following cases:
  - CPUID lead node(%eax=0x4ffffffe)
    - Return the high 32 bits of the total time spent processing all exits in %ebx and %ecx. Return values are measured in processor cycles, across all           vCPUs.
  - CPUID leaf node(%eax=0x4fffffff)
    - Return the total number of exits for all types in %eax.
  
<h2> Team Details: </h2> 
   - `Done by Myself`
   
<h2>Describe in detail the steps used to complete the assignment:</h2>
   - Download kernel from https://www.kernel.org/  and extract source code (Run following commands)
  
```
sudo apt-get install wget
wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.0.7.tar.xz
tar xvf linux-6.0.7.tar.xz
```

- Install required packages before intiating installation:
```
sudo apt-get install git fakeroot build-essential ncurses-dev xz-utils libssl-dev bc flex libelf-dev bison
```
- Configure kernel
```
cd linux-6.0.7
cp -v /boot/config-$(uname -r) .config
```

- Build the kernel
```
sudo make modules
sudo make
sudo make modules_install
sudo make install
```
- Bootloader is automatically updated because of the make install and now we can reboot the system and check the kernel version
```
sudo reboot
uname -mrs
```
<img width="441" alt="Screen Shot 2022-12-05 at 7 24 33 AM" src="https://user-images.githubusercontent.com/29530515/205539152-91c80941-4f9e-4243-8ce8-7dcf63bb07e2.png">

- Make code changes and overwrite in given location below.(vmx.c and cpuid.c)
```
/linux-6.0.7/arch/x86/kvm/vmx/vmx.c
/linux-6.0.7/arch/x86/kvm/cpuid.c
```

- Build and install modules again
```
sudo make modules && sudo make modules_install
```
- Load newly build modules
```
sudo rmmod kvm_intel
sudo rmmod kvm
sudo modprode kvm
sudo modprobe kvm_intel
```

- For testing the functionality, a nested VM was created in the GCP VM. The steps to create an nested VM are as follows:

  - Download the Ubuntu cloud image.img(QEMU compatible image) file from this [ubuntu cloud images site] in GCP VM(https://cloud-images.ubuntu.com/). 
  ```
  wget https://cloud-images.ubuntu.com/bionic/current/bionic-server-cloudimg-amd64.img
  sudo apt update && sudo apt install qemu-kvm -y
  ```
  
  - Go to the folder where the.img file was downloaded. There is no default password for the virtual machine included with this Ubuntu cloud image. So, follow these instructions (terminal) to modify the password and log in to the virtual machine:
 ```
  sudo apt-get install cloud-image-utils

  cat >user-data <<EOF
    #cloud-config
    password: newpass #new password here
    chpasswd: { expire: False }
    ssh_pwauth: True
    EOF

  cloud-localds user-data.img user-data
 ```
 - Run following command to initiate nested vm. Set configuration : `username: ubuntu` and `password: newpass`
 ```
 sudo qemu-system-x86_64 -enable-kvm -hda bionic-server-cloudimg-amd64.img -drive "file=user-data.img,format=raw" -m 512 -curses -nographic
 ```
 
 - install utility:
 ```
  sudo apt-get update
  sudo apt-get install cpuid
 ```
 
 - IMPORTANT: open two terminals:
    - the GCP VM terminal(T1)
    - the nested VM terminal(logged in)(T2)
    
<h2> Testing feature </h2>

- Testing the CPUID functionality for `%eax=0x4ffffffe`
  - T2: 
  ```
  sudo cpuid -l 0x4ffffffe
  ```
  - <img width="639" alt="Screen Shot 2022-12-05 at 8 01 30 AM" src="https://user-images.githubusercontent.com/29530515/205540961-7ed7a694-9719-44ad-b675-17288486672e.png">
  T1: 
  ```
  sudo dmesg
  ```
  - <img width="408" alt="Screen Shot 2022-12-05 at 8 02 44 AM" src="https://user-images.githubusercontent.com/29530515/205540999-794e3b68-dda5-4538-ace8-8bfc17464d2e.png">

- Testing the CPUID functionality for `%eax=0x4fffffff`
  - T2: 
  ```
  sudo cpuid -l 0x4fffffff
  ```
  - <img width="648" alt="Screen Shot 2022-12-05 at 8 00 18 AM" src="https://user-images.githubusercontent.com/29530515/205541145-00632e89-8345-40ee-ba19-0661ba0c6ac7.png">
  - T1: 
  ```
  sudo dmesg
  ```
  - <img width="372" alt="Screen Shot 2022-12-05 at 8 01 00 AM" src="https://user-images.githubusercontent.com/29530515/205541317-d8a097f4-77b6-43f1-963e-396e1eb6657d.png">


  

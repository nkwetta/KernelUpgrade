# KernelUpgrade from 3.1 to latest EL-Repo kernel release 

## 1.Pre-req: 
  1.  OS version : CentOS Linux release 7.1.1503 (Core)
  2.  Kernel version: 3.10.0-229.14.1.el7.x86_64

## 2.Setting up the ELRepo
      rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
      rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm

## 3.Updating kernel
       yum --enablerepo=elrepo-kernel install kernel-ml -y

# KernelUpgrade from 3.1 to specific 3.x release 
================
## 1.Pre-req: Depende
  1.  OS version : CentOS Linux release 7.1.1503 (Core)
  2.  Kernel version: 3.10.0-229.14.1.el7.x86_64

## 2.Installing dependencies
      yum install gcc ncurses ncurses-devel libbsd bc -y

## 3.Update whole system
      yum update -y

## 4.Downloading the package
      cd /tmp/
      wget https://www.kernel.org/pub/linux/kernel/v3.x/linux-3.18.4.tar.xz
## 5.Package Extraction
      tar -xf linux-3.18.4.tar.xz -C /usr/src/
      cd /usr/src/linux-3.18.4/

## 6.Configuration option
  1. `make menuconfig` 
    (manual configuration to kernel , if you are not aware about the GUI options, double hit Esc key and save the configuration. It would generate a .config file to be processed)

  2. `make oldconfig` 
    (make latest kernel with old confiuration)

## 7.Specific configuraiton required for CRIU feature for docker

      sed -i.bak  's/\(\s*CONFIG_CHECKPOINT_RESTORE.*\)/#\1/g;s/\(\s*CONFIG_NAMESPACES.*\)/#\1/g;s/\(\s*CONFIG_UTS_NS.*\)/#\1/g;s/\(\s*CONFIG_IPC_NS.*\)/#\1/g;s/\(\s*CONFIG_PID_NS.*\)/#\1/g;s/\(\s*CONFIG_NET_NS.*\)/#\1/g;s/\(\s*CONFIG_FHANDLE.*\)/#\1/g;s/\(\s*CONFIG_EVENTFD.*\)/#\1/g;s/\(\s*CONFIG_EPOLL.*\)/#\1/g;s/\(\s*CONFIG_UNIX_DIAG.*\)/#\1/g;s/\(\s*CONFIG_INET_DIAG.*\)/#\1/g;s/\(\s*CONFIG_INET_UDP_DIAG.*\)/#\1/g;s/\(\s*CONFIG_PACKET_DIAG.*\)/#\1/g;s/\(\s*CONFIG_NETLINK_DIAG.*\)/#\1/g;s/\(\s*CONFIG_INOTIFY_USER.*\)/#\1/g;s/\(\s*CONFIG_IA32_EMULATION.*\)/#\1/g;s/\(\s*CONFIG_EXPERT.*\)/#\1/g;s/\(\s*CONFIG_EMBEDDED.*\)/#\1/g' .config
      
      echo -e  "CONFIG_CHECKPOINT_RESTORE=y\nCONFIG_NAMESPACES=y\nCONFIG_UTS_NS=y\nCONFIG_IPC_NS=y\nCONFIG_PID_NS=y\nCONFIG_NET_NS=y\nCONFIG_FHANDLE=y\nCONFIG_EVENTFD=y\nCONFIG_EPOLL=y\nCONFIG_UNIX_DIAG=y\nCONFIG_INET_DIAG=y\nCONFIG_INET_UDP_DIAG=y\nCONFIG_PACKET_DIAG=y\nCONFIG_NETLINK_DIAG=y\nCONFIG_INOTIFY_USER=y\nCONFIG_IA32_EMULATION=y\nCONFIG_EXPERT=y\nCONFIG_EMBEDDED=y" >> .config
      
      
## 8.Compile kernel
      make

## 9. Install new kernel
      make modules_install install
      
## 10. Verifying Kernel
      uname -r
      
** You might need to set the default kernel at the time of boot, Alternatively you can specify the index of kernel to be default at boot time **

      egrep ^menuentry /etc/grub2.cfg | cut -f 2 -d \'

Output: 

      CentOS Linux (4.2.3-1.el7.elrepo.x86_64) 7 (Core)
      CentOS Linux (3.10.0-229.14.1.el7.x86_64) 7 (Core)
      CentOS Linux, with Linux 3.10.0-123.el7.x86_64
      CentOS Linux, with Linux 0-rescue-ee31ffedfc2a48d4b437f776818a7cd2

`grub2-set-default 0` # for kernel 4.2.3-1.el7.elrepo.x86_64 to be default at boot time
`grub2-set-default 1` # for kernel 3.10.0-229.14.1.el7.x86_64 to be default at boot time

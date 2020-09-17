# Steps/Commands to compile and boot into a custom kernel on RHCOS node:
  
  # cd /var/home/core
  
  # cat dockfile
  FROM fedora:latest
  
  RUN dnf install -y kexec-tools make gcc git openssl-devel bison flex autoconf automake bc
  # 
  
  *** Setup container image to build the kernel ***
  # podman build -t fedora:build -f dockfile
  
  # cp /lib/modules/`uname -r`/config .
  
  # podman run -it -v /var/home/core:/root --privileged=true -- fedora:build bash
  
  *** Build the kernel in container ***
  # cd /root
  
  # git clone --depth=1 git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
  
  # cp config linux/.config
  
  # cd linux
  
  # make oldconfig
  
  # make vmlinux; make modules
  
  # make INSTALL_MOD_STRIP=1 INSTALL_MOD_PATH=/root modules_install
  
  # cp vmlinux /root/vmlinux-<kver>
  
  # cp /sbin/kexec /root
  
  # exit
  
  *** Back to RHCOS host ***
  
  *** To build initrd image, pick dracut options from existing RHCOS initrd in /boot/ostree and use the same options,
  *** plus "--kmoddir '/var/home/core/lib/modules/5.9.0-rc5'" to find the modules since we could not install kernel modules
  *** at the usual place as /lib/modules directory is read-only.  ***
  # mkdir /tmp/dracut
  # dracut --reproducible --gzip -v --add 'ostree' --tmpdir '/tmp/dracut' -f --add 'ignition' --no-hostonly --add-drivers 'mptspi vmw_pvscsi' --omit-drivers 'nouveau' --omit 'nfs' --add 'iscsi' --add 'ifcfg' --add 'fips' --kmoddir '/var/home/core/lib/modules/<kver>' initramfs-<kver>.img <kver>
  
  ** Ready to load and kexec into the just built kernel ***
  # ./kexec -l --append="`cat /proc/cmdline`" --initrd=initramfs-<kver>.img vmlinux-<kver>
  
  # ./kexec -e
  
  Note: After booting past initrd, 'modprobe' would fail to find modules as the modules for the custom kernel could not be placed in
        /lib/modules directory (read-only) where they are looks for. Three options to workaround it:
             a. Use alias "modprobe -d /var/home/core" for "modprobe"
             b. Build the missing kernel modules into the initrd with add-drivers option
             c. Compile the modules into the kernel (=y instead of =m)


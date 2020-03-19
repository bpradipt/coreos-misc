# Create user_data file

Create a file named `user_data` and copy the below content

```
{
  "ignition": { "version": "2.2.0" },
  "passwd": {
    "users": [
      {
        "name": "core",
        "passwordHash": "$1$7X2OoGIo$SnfQ1fDrODPzpSo3sNEsj/",
        "sshAuthorizedKeys": [
          "ssh-rsa asdasdasdasdasdasdasdadasxyzzzzzzzzzzz"
        ]
      }
    ]
  }
}
```

The `passwordHash` is for default password `passw0rd`.
For generating your own `passwordHash` you can use the command
`openssl passwd -1`.

Change the `sshAuthorizedKeys` to your SSH public key.


# Create ISO
Run the following command on a Linux host

```
mkdir -p /tmp/new-drive/openstack/latest
cp user_data /tmp/new-drive/openstack/latest/user_data
mkisofs -R -V config-2 -o configdrive.iso /tmp/new-drive
rm -rf /tmp/new-drive
```

# Create CoreOS VM with Config drive attached

Here is a sample libvirt XML snippet to attach config drive

```
 <disk type='file' device='cdrom'>
      <driver name='qemu' type='raw'/>
      <source file='/var/lib/libvirt/images/configdrive.iso'/>
      <target dev='sda' />
 </disk>
```

# Complete Libvirt XML for KVM on Power (ppc64le)

```
<domain type='kvm' >
  <name>rhcos</name>
  <memory unit='KiB'>16777216</memory>
  <currentMemory unit='KiB'>16777216</currentMemory>
  <vcpu placement='static'>2</vcpu>
  <resource>
    <partition>/machine</partition>
  </resource>
  <os>
    <type arch='ppc64le' machine='pseries-rhel7.6.0'>hvm</type>
    <boot dev='hd'/>
    <boot dev='network'/>
  </os>
  <clock offset='utc'/>
  <on_poweroff>destroy</on_poweroff>
  <on_reboot>restart</on_reboot>
  <on_crash>destroy</on_crash>
  <devices>
    <emulator>/usr/libexec/qemu-kvm</emulator>
    <disk type='file' device='disk'>
      <driver name='qemu' type='qcow2'/>
      <source file='/var/lib/libvirt/images/rhcos-openstack.qcow2'/>
      <target dev='vda' bus='virtio'/>
    </disk>
    <disk type='file' device='cdrom'>
      <driver name='qemu' type='raw'/>
      <source file='/var/lib/libvirt/images/configdrive.iso'/>
      <target dev='sda' />
   </disk>
   <interface type='bridge'>
      <source bridge='virbr1'/>
      <model type='virtio'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x01' function='0x0'/>
    </interface>
    <serial type='pty'>
      <source path='/dev/pts/17'/>
      <target type='spapr-vio-serial' port='0'>
        <model name='spapr-vty'/>
      </target>
      <alias name='serial0'/>
      <address type='spapr-vio' reg='0x30000000'/>
    </serial>
    <console type='pty' tty='/dev/pts/17'>
      <source path='/dev/pts/17'/>
      <target type='serial' port='0'/>
          <alias name='serial0'/>
      <address type='spapr-vio' reg='0x30000000'/>
    </console>
   <input type='keyboard' bus='usb'>
      <alias name='input0'/>
      <address type='usb' bus='0' port='1'/>
   </input>
  </devices>
</domain>
```

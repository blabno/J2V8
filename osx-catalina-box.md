# Build osx-catalina Vagrant box

Overview

1. Create VirtualBox VM with macOS (https://github.com/myspaghetti/macos-virtualbox)
   1. Change `system_integrity_protection='10'` to `system_integrity_protection='77'` in `macos-guest-virtualbox.sh`.
2. Tweak the VM for Vagrant
3. Convert VirtualBox VM to Vagrant box

## Create /vagrant directory

The root filesystem is read-only. Solution is describe at https://github.com/myspaghetti/macos-virtualbox/issues/607 and summarized here:

1. Reboot virtual machine, press ESC to enter boot menu
2. If the boot menu appears, select "Boot Manager" and
then "EFI Internal Shell" and then allow the startup.nsh script to execute
automatically, applying the NVRAM variables before booting macOS
3. Once standard login screen shows up use terminal: 
4. sudo mount -uw /
5. mkdir /Users/vagrant/Documents/vagrant
6. sudo ln -s /Users/vagrant/Documents/vagrant /vagrant

## Vagrant group

1. Open `System preferences->Users & groups`
2. At the bottom left corner click the lock icon to unlock settings.
3. Hit + to add new group. Change value of `New account` select field to `Group`
4. Enter `vagrant` as the group name and create the group 
5. Select the group on the list (below list of users) and add vagrant user to the vagrant group.
6. At the bottom left corner click the lock icon to lock settings.

## SSH

1. Open `System preferences->Sharing`
2. Select `Remote login`
3. Select `Allow access` to `All users`
4. In VirtualBox settings of the virtual machine open `Network->Adapter 1->Advanced->Port forwarding`
5. Add entry: Name: SSH, Host port: 2222, Guest port: 22

Use terminal of your host system.

```bash
ssh vagrant@localhost -p 2222
sudo su
echo "UseDNS no" >> /etc/ssh/sshd_config
exit
```

## Sudoers

Use terminal of your host system.

```bash
ssh vagrant@localhost -p 2222
sudo echo "vagrant ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/vagrant

```

## Vagrant ssh key

Use terminal of your host system.

```bash
ssh vagrant@localhost -p 2222 "mkdir ~/.ssh"
vagrant global-status
ssh-keygen -y -f ~/.vagrant.d/insecure_private_key > /tmp/authorized_keys
scp -P 2222 /tmp/authorized_keys vagrant@localhost:~/.ssh/
```

## XCode
              
Download Xcode_12.4 from Apple Store https://developer.apple.com/download/all/?q=xcode%2012.4.
Use terminal of your host system.

```bash
scp -P 2222 Xcode_12.4.xip vagrant@localhost:/tmp/
ssh vagrant@localhost -p 2222 "xip -x /tmp/Xcode_12.4.xip"
ssh vagrant@localhost -p 2222 "mv Xcode.app /Applications/"
ssh vagrant@localhost -p 2222 "sudo xcodebuild -license accept"
```
                                          
Shutdown VM.

```bash
vagrant package --base macOS --output osx-catalina.box
vagrant box add osx-catalina.box --name osx-catalina 
```

parentTopic:: [[Computer]]

# Preparing custom box for Vagrant
Some docs say `package` command is only for VirtualBox and `repackage` is for everything else. However, `repackage` does not exist anymore in latest versions of Vagrant. Wtf?

## Repackage
Easier method.
- Create a `Vagrantfile`, use some minimal box as the image
- Set `config.ssh.insert_key=false`! It's important. Without it Vagrant will mess up the ssh keys and it won't work after repackaging.
- Run `vagrant up`
- Perform setup you want with the machine
- For minimal Ubuntu, this is a very lightweight way to get gui: `sudo apt-get install lxdm xinit i3 -y`.
- Shutdown the machine
- Note the VM name in VirtualBox, e.g. `deploy_ubuntu2204_1707866371369_28268`.
- Run `vagrant package --base deploy_ubuntu2204_1707866371369_28268 --output MyGreatMachine.box`.

## From scratch
I've not done this yet. This is the harder method. [This](https://developer.hashicorp.com/vagrant/docs/boxes/base) is probably worth reading. What I know is that we should install naked OS and take care of these things:
- username=vagrant password=vagrant  
- install dkms build-essential linux-headers-generic (not sure if neccessary)
- shorten grub timeout  
- install guest additions  
- setup ssh server

# Clearing cache
Some Vagrantfile changes may not be honoured because of caches. This command can clear them up, so we start with a clean slate. All of the VMs in VirtualBox will become unused by Vagrant. New ones will be generated. Be sure to remove them to not waste space.
```
vagrant halt && vagrant global-status --prune && rm -rf .vagrant/
```

# Where vagrant keeps files
1. `~/.vagrant.d` - Vagrant home (location changed by `VAGRANT_HOME` env). It contains various metadata, but mainly base boxes. They can get pretty large - windows box is ~8.5GiB.
2. `<cwd>/.vagrant` - Cached metadata for VMs run in current directory. Not very large - about 50-100KiB.
3. `~/vm` - VirtualBox home. Vagrant creates virtual machines, but never destroys them, so it may take quite a lot of space because of disk cloning.
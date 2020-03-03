**FreeBSD thinJail setup my way**

The idea behind this setup is to separate the thinjail's base structure and system configuration into different paths but within /jails ZFS dataset, mainly to make it easier to export individual parts of the structure through NFS.
With this setup the system image template goes into /jails/structure/thinjail, jail specific RW skeleton folder to /jails/thinjails,  fstab in /jails/fstab, and ultimately base + skeleton folders are mounted in /jails/mnt.

#### Create thinjail base template ####
```
sudo mkdir -p /jails/structure/thinjail/base/12.1-RELEASE
```
#### Copy image into thinjail's base folder ####
```
sudo cp -R /jails/releases/12.1-RELEASE/root/* /jails/structure/thinjail/base/12.1-RELEASE/
```
#### Create skeleton folder structure ####
```
sudo mkdir -p /jails/structure/thinjail/skeleton/12.1-RELEASE/usr
sudo mv /jails/structure/thinjail/base/12.1-RELEASE/etc /jails/structure/thinjail/skeleton/12.1-RELEASE/etc
sudo mv /jails/structure/thinjail/base/12.1-RELEASE/usr/local /jails/structure/thinjail/skeleton/12.1-RELEASE/usr/local
sudo mv /jails/structure/thinjail/base/12.1-RELEASE/var /jails/structure/thinjail/skeleton/12.1-RELEASE/var
sudo mv /jails/structure/thinjail/base/12.1-RELEASE/tmp /jails/structure/thinjail/skeleton/12.1-RELEASE/tmp
sudo mv /jails/structure/thinjail/base/12.1-RELEASE/root /jails/structure/thinjail/skeleton/12.1-RELEASE/root
```
#### Template skeleton structure symlinking ####
```
cd /jails/structure/thinjail/base/12.1-RELEASE
sudo mkdir skeleton
sudo ln -s skeleton/etc etc
sudo ln -s skeleton/home home
sudo ln -s skeleton/root root
sudo ln -s skeleton/usr/local usr/local
sudo ln -s skeleton/tmp tmp
sudo ln -s skeleton/var var
```
#### Creating and copying skeleton template to thinjail1 local folder ####
```
sudo mkdir -p /jails/thinjails/thinjail1
sudo cp -R /jails/structure/thinjail/skeleton/12.1-RELEASE/* /jails/thinjails/thinjail1
echo hostname=\"thinjail1\" > rc.conf; sudo mv rc.conf /jails/thinjails/thinjail1/etc/
```
#### Create folder where base and skeleton will be mounted to ####
```
sudo mkdir -p /jails/mnt/thinjail1
```
#### Create jail's fstab folder ####
```
sudo mkdir /jails/fstab
```
#### thinjail1's fstab ####
> /jails/fstab/thinjail1.fstab
```
# this is thinJail's base template path       # thinjail local mount
/jails/structure/thinjail/base/12.1-RELEASE /jails/mnt/thinjail1/  nullfs  ro  0 0
# Skeleton folder structure                   # same as above
/jails/thinjails/thinjail1 /jails/mnt/thinjail1/skeleton  nullfs  rw  0 0
```

#### jail.conf ####
```
thinjail1 {
    path          = "/jails/mnt/thinjail1";
    host.hostname = "thinjail1";
    $epair        = "epair3";  # must be unique in every jail
    mount.fstab    = "/jails/fstab/thinjail1.fstab";
    exec.poststart = "ifconfig epair3a 192.168.99.1 netmask 255.255.255.0";
    exec.consolelog = "/jails/logs/thinjail1/console.log";
 }
 ```
If all went well, a mount from the FreeBSD host with thinjail1 running should look as following 
```
[admin@lockdown ~]$ mount |grep thinjail1
/jails/structure/thinjail/base/12.1-RELEASE on /jails/mnt/thinjail1 (nullfs, local, read-only)
/jails/thinjails/thinjail1 on /jails/mnt/thinjail1/skeleton (nullfs, local)
devfs on /jails/mnt/thinjail1/dev (devfs, local, multilabel)
[admin@lockdown ~]$ 
```
While from thinjail1 only / appears as mounted and in RO, though skeleton is writable
```
[admin@lockdown ~]$ sudo jexec thinjail1 mount
/jails/structure/thinjail/base/12.1-RELEASE on / (nullfs, local, read-only)
[admin@lockdown ~]$ 
```
Thanks to [citia](https://github.com/clinta/clinta.github.io/blob/2b28a7d626eff467e44ce18dd1000aa2c279a329/_posts/2015-08-09-freebsd-jails-the-hard-way.md) for the idea. 

# Hosting a Django project on Linode

#### Configure hostname

```
echo "hostname" > /etc/hostname
hostname -F /etc/hostname

nano /etc/hosts
```
Add:
`ip-address   hostname.example.com  hostname`

#### Configure timezone

`dpkg-reconfigure tzdata`

#### Add normal user

`adduser example_user`

Add to sudo group: `adduser example_user sudo`

#### Setup 

On Mac:

`ssh-keygen -b 4096`

On Linux:

`mkdir ~/.ssh`
`nano ~/.ssh/authorized_keys`

On Mac:

`scp ~/.ssh/id_rsa.pub example_user@ip_address:~/.ssh/authorized_keys`

Then you can log in without having to enter a password!



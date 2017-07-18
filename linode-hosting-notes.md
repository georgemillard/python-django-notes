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




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

#### Setup SSH keys

On Mac:

`ssh-keygen -b 4096`

On Linux:

`mkdir ~/.ssh`
`nano ~/.ssh/authorized_keys`

On Mac:

`scp ~/.ssh/id_rsa.pub example_user@ip_address:~/.ssh/authorized_keys`

Then you can log in without having to enter a password!

#### SSH Demon settings

`sudo nano /etc/ssh/sshd_config`

Set `PermitRootLogin` and `PasswordAuthentication` to `no`

Then `sudo service ssh restart`

#### Check running processes

`sudo netstat -tulpn`

`ps aux | less`

`pgrep <process_name>

End a process with `sudo apt-get purge program_name`

#### Fail2Ban

Default settings `/etc/fail2ban/fail2ban.conf`

Override by copying and editing to `/etc/fail2ban/fail2ban.local`

Default logs `/var/log/fail2ban.log`

To run commands in a client `sudo fail2ban-client -i`

Commands include `status, start, reload, stop, get logtarget, exit...`

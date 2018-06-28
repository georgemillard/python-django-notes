### Installing SSL certificates with letsencrypt and certbot

[Useful tutorial here](https://www.digitalocean.com/community/tutorials/how-to-set-up-let-s-encrypt-certificates-for-multiple-apache-virtual-hosts-on-ubuntu-14-04)

#### Install certbot
```
sudo add-apt-repository ppa:certbot/certbot

sudo apt-get update

sudo apt-get install python-certbot-apache
```

#### Install certificates
Best to install all subdomains along with root domain

`sudo certbot --apache -d example.com -d www.example.com`

#### Delete a cerificate 
Note this will cause apache errors, as references to the cart are not removed. Best to a2dissite the site first.

`certbot delete`

#### Check renewal status

```
sudo apt-get ssl-cert-check
su root
cd /etc/letsencrypt/live
ssl-cert-check <my_domain>/cert.pem
```


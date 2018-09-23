# Baseline Configuration

## Ubuntu 16.04.4 LTS Initial Setup

### Change root password
```
Changing password for root.
(current) UNIX password: 
Enter new UNIX password: 
Retype new UNIX password: 
```

### Add new user and set password
```bash
# adduser xxx
# chgpasswd 
```

### Update the firewall
```bash
$ ufw app list
Available applications:
  OpenSSH
$ sudo ufw allow OpenSSH
Rules updated
Rules updated (v6)
$ sudo ufw enable
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
$ sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere                  
OpenSSH (v6)               ALLOW       Anywhere (v6)             
```

## Install Tomcat Application Server

### Install Java (for Tomcat)
```bash
$ sudo apt-get update
$ sudo apt-get install default-jdk
```

### Install Tomcat
- Create a `tomcat` group
- Create a `tomcat` user
- Download a Tomcat distribution to `/tmp`
- Extract the distribution to `/opt/tomcat`
- Adjust the permissions of `/opt/tomcat`
```bash
$ sudo groupadd tomcat
$ sudo useradd -s /bin/false -g tomcat -d /opt/tomcat tomcat
$ cd /tmp
/tmp$ curl -O http://www-us.apache.org/dist/tomcat/tomcat-8/v8.5.33/bin/apache-tomcat-8.5.33.tar.gz
/tmp$ sudo mkdir /opt/tomcat
/tmp$ sudo tar xzvf apache-tomcat-8*tar.gz -C /opt/tomcat --strip-components=1
/tmp$ cd /opt/tomcat/
/opt/tomcat$ sudo chgrp -R tomcat /opt/tomcat
/opt/tomcat$ sudo chmod -R g+r conf
/opt/tomcat$ sudo chmod g+x conf
/opt/tomcat$ sudo chown  -R tomcat webapps/ work/ temp/ logs
```

### Update Tomcat service
- Get location of JAVA 
- Update service with JAVA location
- Start Tomcat
```bash
$ sudo update-java-alternatives -l
java-1.8.0-openjdk-amd64       1081       /usr/lib/jvm/java-1.8.0-openjdk-amd64
$ sudo nano /etc/systemd/system/tomcat.service
$ sudo systemctl daemon-reload
$ sudo systemctl start tomcat
$ sudo systemctl status tomcat
```

### Configure firewall to allow port 8080 (Tomcat)
```bash
$ sudo ufw allow 8080
Rule added
Rule added (v6)
```

### Configure Tomcat users
``` bash
$ sudo nano /opt/tomcat/webapps/host-manager/META-INF/context.xml
$ sudo nano /opt/tomcat/conf/tomcat-users.xml
$ sudo systemctl restart tomcat
$ sudo systemctl status tomcat

```

## Apache Web Server

### Install Apache
```bash
$ sudo apt-get update
$ sudo apt-get install apache2
```

### Setting ServerName
```bash
$ sudo nano /etc/apache2/apache2.conf 
$ sudo apache2ctl configtest
$ sudo systemctl restart apache2
```

### Adjust the firewall to allow web traffic
```bash
$ sudo ufw app list
$ sudo ufw app info "Apache Full"
$ sudo ufw allow in "Apache Full"
```

### Setting ServerName
```bash
$ sudo nano /etc/apache2/apache2.conf 
$ sudo apache2ctl configtest
$ sudo systemctl restart apache2
```

### Install Let\'s Encrypt
```bash
$ sudo add-apt-repository ppa:certbot/certbot
$ sudo apt-get update
$ sudo apt-get install python-certbot-apache
```

### Setup the SSL certificate
```bash
$ sudo certbot --apache -d emxsys.net
$ sudo certbot renew --dry-run
```

### Install Apache module for reverse proxy for Tomcat
```bash
$ sudo apt-get update
$ sudo apt-get install libapache2-mod-jk
$ sudo nano /etc/libapache2-mod-jk/workers.properties 
```

### Adjust Apache virtual host to proxy with mod_jk
```bash
$ sudo nano /etc/apache2/sites-available/default-ssl.conf 
$ sudo apache2ctl -S
$ sudo nano /etc/apache2/sites-available/000-default-le-ssl.conf 
$ sudo apache2ctl -S
$ sudo systemctl restart apache2
```

### Adding CORS support: Header set Access-Control-Allow-Origin "*"
```bash
$ sudo nano /etc/apache2/sites-available/000-default-le-ssl.conf 
$ sudo nano /etc/apache2/sites-available/000-default.conf 
$ sudo apache2ctl configtest
```

### Enable Apache modules
```shell
$ sudo a2enmod headers
$ sudo a2enmod proxy
$ sudo a2enmod proxy_http
$ sudo a2enmod proxy_html
$ sudo a2enmod rewrite
$ sudo a2enmod deflate
$ sudo a2enmod proxy_connect
$ sudo a2enmod xml2enc
$ sudo systemctl restart apache2
$ sudo systemctl status apache2
```
### Install PostFix for send only
```shell
sudo apt-get update
sudo apt install mailutils
sudo ufw app list
sudo ufw app info 'Postfix'
sudo ufw app info 'Postfix SMTPS'
sudo ufw app info 'Postfix Submission'
sudo ufw allow 'Postfix Submission'
sudo ufw status
```
#### To reconfigure
```shell
sudo dpkg-reconfigure postfix
```
#### To setup 
```shell
sudo nano /etc/postfix/main.cf
sudo systemctl status postfix
```

### Install LogWatch
```shell
sudo apt-get update
sudo apt-get install logwatch
sudo nano /usr/share/logwatch/default.conf/logwatch.conf 
```
#### Review daily report job
```shell
cat /etc/cron.daily/00logwatch 
```
#### Test
```shell
sudo logwatch --detail med --service all --range "since 24 hours ago for those hours" | less  
```
#### Test email report
```shell
sudo /usr/sbin/logwatch --output mail
```

### Install Fail2Ban
```shell
sudo apt-get update
sudo apt-get install fail2ban
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local 
sudo systemctl restart fail2ban
```

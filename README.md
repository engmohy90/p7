# udacity-p7-Linux-Server-Configuration

## content
- [overview](#overview)
- [software installed](#software-installed)
- [step by step](step-by-step)

IP address, URL, summary of software installed, summary of configurations made, and a list of third-party resources used to completed this project.


## overview

- this project is a linux configuration to deploy web app via apache using cloud instance (cloud vm)(vps)
<table>
 <tr>
 <td>IP address</dt>
 <td>104.198.139.131</td>
 </tr>
  <tr>
 <td>ssh port</dt>
 <td>2200</td>
 </tr>
 <tr>
 <td>URL</dt>
 <td>http://mohymenu.tk</td>
 </tr>
 </table>
 
 ## software installed 
- apache2 
- postgresql
- tree
## step by step
#### install ubunut
- add user grader  ``` useradd grader ```
- give grader root privilege with out password 
```
vim /etc/sudors.d/grader 
#insert in fill grader 
grader ALL=(ALL:ALL) NOPASSWD:ALL
```
#### enable ssh connect for grader

- from any pc terminal 
``` ssh-keygen ```
will ask you for path 
type ```~/grader```
will genrate two files
grader which is key 
and grader.pub wich is public key
copy public key conntent and 
touch file authorized_keys at this path ```/home/grader/.ssh```
past puplic key into the touched file 
- change ssh port to 2200 stop root remote connect
force key authorization over password
```
sudo vim  /etc/ssh/sshd_config
change Posrt 22 to any number here we will make it 2200.
change PermitRootLogin yes to 
PermitRootLogin no
# Change to no to disable tunnelled clear text passwords
PasswordAuthentication no

```
- try to connect to the machine via ssh 
```
ssh grader@machineIP -p 2200 -i path-to-grader(the key you generated)

```
- you can find and download my key at this repo
#### setting up firewall 
1- ```sudo ufw default deny incoming```
2- ```sudo ufw default allow outgoing```
3- ```sudo ufw allow 2200/tcp ```  >>> ssh allow 
4- ```sudo ufw allow www    ```>>> http allow 
5- ```sudo ufw allow 123/udp ```>>> NTP allow
6- ```sudo ufw enable```

#### installing postgress
- ```sudo apt-get install postgresql```
- if you have problem installing go to this [link](https://www.postgresql.org/download/linux/ubuntu/)
- add user and make database
```
$ sudo -su postgres
$psql
>CREATE USER graderdb WITH PASSWORD 'pasword';
>CREATE DATABASE catalogdb;
>GRANT ALL ON DATABASE catalogdb TO  mohy ;
>/q
$exit
```
- now change your flask app connection to data base as like 

```
url = "postgresql://graderdb:pasword@localhost:5432/catalogdb"
engine = create_engine(url)
```
## installing apache2 and configure it to run flask project 
- install apache
``` 
sudo apt-get install apache2
```
- install mod_wsgi 
```
sudo apt-get install libapache2-mod-wsgi
```
- install flask and all dependancy need to run catalog.py 
- test your flask:
```
python catalog.py
```
- open from your browser 127.0.0.1:5000
### configure mode-wsgi

- copy your project to /var/www/FlaskApp
- create file start.wsgi
```
sudo vim /var/www/FlaskApp/start.wsgi
```
- insert in this file python code which import your app opject from flask 
```
import sys


sys.path.insert(0, "/var/www/flaskapp/production")

from catalog import app as application
```
- your directory should look like this 

```
├── flaskapp
|   ├── production
│   │   ├── catalog.py
│   │   ├── catalog.pyc
│   │   ├── database_setup.py
│   │   ├── database_setup.pyc
│   │   ├── static
│   │   │   ├── face.jpg
│   │   │   ├── login.js
│   │   │   ├── main.css
│   │   │   ├── main.js
│   │   │   ├── signin.png
│   │   │   └── signup.js
│   │   └── templates
│   │       ├── base.html
│   │       ├── delete.html
│   │       ├── edit.html
│   │       ├── index.html
│   │       ├── iteminfo.html
│   │       ├── items.html
│   │       ├── login.html
│   │       ├── mainpage.html
│   │       ├── newC.html
│   │       ├── newitem.html
│   │       ├── profile.html
│   │       └── signup.html
│   |
│   └── start.wsgi

```
- onpen your flask app and remove app.run() or make sure it is under if __name__ == "__main__":
- touch file at /etc/apache2/site-avaliable/
``` 
sudo vim /etc/apache2/site-avaliable/catalog.conf
```
- configure mod_wsgi by insert this text on catalog.conf
```
<VirtualHost *:80>
    
    WSGIDaemonProcess catalog user=www-data group=www-data threads=5
    WSGIScriptAlias / /var/www/flaskapp/start.wsgi
    <Directory /var/www/flaskapp>
        Require all granted
    </Directory>
    alias /static /var/www/flaskapp/production/static
    <Directory /var/www/flaskapp/production/static/>
        Require all granted
    </Directory>
</VirtualHost>
```
- Enable the virtual host with the following command
```
sudo a2ensite catalog
sudo service apache2 reload
```

#### upgrade system packages 
```
sudo apt-get update 
sudo apt-get upgrade
```






 

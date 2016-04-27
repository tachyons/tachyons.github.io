---
layout: single
title: "Deploying Fedena Using Ubuntu 14.04"
modified:
categories:
excerpt:
comments: true
tags: ['fedena','14.04','ngnix','passenger']
image:
  feature:
date: 2015-11-18T09:30:13+05:30
---

* Install Ruby Dependancies

```bash
sudo apt-get update
sudo apt-get install git-core curl zlib1g-dev build-essential libssl-dev libreadline-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt1-dev libcurl4-openssl-dev python-software-properties
```

* Install Ruby Using RVM

```bash
sudo apt-get install libgdbm-dev libncurses5-dev automake libtool bison libffi-dev
curl -L https://get.rvm.io | bash -s stable
source ~/.rvm/scripts/rvm
echo "source ~/.rvm/scripts/rvm" >> ~/.bashrc
rvm install 1.8.7
rvm use 1.8.7 --default
ruby -v
```

* Setting up MySQL server
Fedena uses mysql, so run,

```bash
sudo apt-get install libmysqlclient-dev mysql-server
```
Do remember the mysql password you set during this step, it is required later

* Checking Out modified version version of fedena

I made some modifications in project fedena to make it easily installable

```bash

git clone https://github.com/tachyons/project_fedena.git
cd project_fedena
cp config/database.yml.example config/database.yml
bundle install
rake db:create
bundle exec rake fedena:plugins:install_all
```

* Installing passenger and nginix

```bash
# Install passenger PGP key and add HTTPS support for APT
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 561F9B9CAC40B2F7
sudo apt-get install -y apt-transport-https ca-certificates

# Add passenger APT repository
sudo sh -c 'echo deb https://oss-binaries.phusionpassenger.com/apt/passenger trusty main > /etc/apt/sources.list.d/passenger.list'
sudo apt-get update

# Install Passenger + Nginx
sudo apt-get install -y nginx-extras passenger
```

Edit /etc/nginx/nginx.conf and uncomment passenger_root and passenger_ruby. For example, you may see this:

```bash
# passenger_root /some-filename/locations.ini;
# passenger_ruby /usr/bin/passenger_free_ruby;
```
Remove the '#' characters, like this:

```bash
passenger_root /some-filename/locations.ini;
passenger_ruby /usr/bin/passenger_free_ruby;
```
Now you can validate the installation using the command

```bash
sudo passenger-config validate-install
```

now create an entry in /etc/ngnix/sites.enabled

```bash
sudo vim /etc/nginx/sites-enabled/project_fedena
```
Now modify and paste following text,ie edit your server name and project path

```bash
server {
    listen       80;
    server_name  fedena.example.com;
    location / {
    	root   /your/fedena/directory/public;
     	passenger_enabled on;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }
  	error_page 413 /413.html;
   	location = /413.html{
       root   html;
       allow all;
    }
 }

```
you can verify above configuration by adding `127.0.1.1	fedena.example.com
` in `/etc/hosts`

Now restart the ngnix server and check fedena.example.com

```bash
sudo service ngnix restart
```

## References
* https://www.phusionpassenger.com/library/install/nginx/install/oss/trusty/
* http://aboobacker.in/installing-project-fedena-in-14-04/

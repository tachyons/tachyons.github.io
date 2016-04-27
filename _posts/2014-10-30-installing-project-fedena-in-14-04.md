---
layout: single
title: "Installing Project Fedena in Ubuntu 14.04"
modified:
categories:
excerpt:
comments: true
tags: []
image:
  feature:
date: 2014-10-30T16:36:20+05:30
---
Modified version of Fedena Installation guide at project fedena website

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

* Install Rails 2.3.5

```bash
gem install rails -v 2.3.5
```

* Setting up MySQL server
Fedena uses mysql, so run,

```bash
sudo apt-get install libmysqlclient-dev mysql-server
```
Do remember the mysql password you set during this step, it is required later

* Clone Fedena Source Code
```bash

git clone https://github.com/projectfedena/fedena.git
```

* Setup your database details in the database.yml


Open the file database.yml in the config folder of the fedena soucre. Change the following details:

database: fedena - The name of the database you want to use for fedena

username: root - Mysql username for fedena

password: mypass - The password for the above mysql user


* Install the rest of the gems

```bash
gem uninstall -i ~/.rvm/gems/ruby-1.8.7-head@global rake
gem install rake -v 0.8.7
gem install declarative_authorization -v 0.5.1
gem install i18n -v 0.4.2
gem install mysql
gem install rush -v 0.6.8
gem update --system 1.3.7
```
* Set up Fedena databases

From the Fedena source directory in terminal run,

```bash
rake db:create
rake db:migrate
```

followed by,

```bash
rake fedena:plugins:install_all
```

* Set up pdf setings

```bash
sudo apt-get install wkhtmltopdf
cd config/initializers
cp wicked_pdf.rb.example wicked_pdf.rb
vi wicked_pdf.rb
```
now change :wkhtmltopdf => '/opt/wkhtmltopdf', to :wkhtmltopdf => '/usr/bin/wkhtmltopdf',
save the file

* Setup Email

```bash
cd config
cp smtp_settings.yml.example smtp_settings.yml
vi smtp_settings.yml
```
Add your settings and save the file

* Setup Sms

```bash
cd config
vi sms_settings.yml
```
Add your settings and save the file


* Change permissions for scripts

From the same directory grant executable permissions for the files in script directory by,

```bash
chmod +x script/*
```
* Run the inbuilt server

If everything went fine till now, you are ready to run fedena server by running the following from fedena source folder,

```bash
script/server
```

Note: This guide is for setting development environment , for fedena deployment refer [Deploying Fedena in 14.04](http://aboobacker.in/deploying-fedena-using-ubuntu-14-04/).

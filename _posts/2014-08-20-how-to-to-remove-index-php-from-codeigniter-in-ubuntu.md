---
layout: single
title: "How to to Remove index.php From Codeigniter in Ubuntu"
modified:
categories:
comments: true
excerpt:
tags: []
image:
  feature:
date: 2014-08-20T06:54:15+05:30
---

* Install code igniter
* Enable mode rewrie

```ruby
$sudo a2enmod rewrite
```
* Add the .htaccess files with contents

```ruby
RewriteBase /your_project_name/
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond $1 !^(index\.php|images|install|css|robots\.txt)
RewriteRule .* index.php/$0 [PT,L]
Options -Indexes
```
* Open the file and replace AllowOverride None by AllowOverride All

```ruby
sudo gedit /etc/apache2/sites-available/default
```
* Restart apache server

```ruby
$sudo /etc/init.d/apache2 restart
```


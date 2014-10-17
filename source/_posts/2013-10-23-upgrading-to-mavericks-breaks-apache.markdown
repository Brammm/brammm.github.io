---
layout: post
title: "Upgrading to Mac OS X Mavericks Breaks Apache"
date: 2013-10-23 16:02
comments: true
categories: Tip
---
I decided to upgrade to Mavericks today. Things went pretty smooth, right up until trying to continue work. I don't use Vagrant or MAMP, but rather the built in Apache2 (with PHP, etc installed via Homebrew). 

My VirtualHosts in `/etc/apache2/extra/httpd-vhosts.conf` were kept, but going to http://localhost/ took me to the default "It Works!" page. I quickly discovered that the httpd.conf file was reset. Stuff I had to do to fix it below.
<!-- more -->

- In `httpd.conf`, uncomment `Include /private/etc/apache2/extra/httpd-vhosts.conf`
- In `httpd.conf`, include the path to mod_php, in my case `LoadModule php5_module /usr/local/Cellar/php54/5.4.19/libexec/apache2/libphp5.so`
- Create a file `<yourusername>.conf` in `/etc/apache2/users` with content
``` text /etc/apache2/users/bram.conf
    <Directory "/path/to/your/sites/">
    Options Indexes MultiViews FollowSymLinks
    AllowOverride All
    Order allow,deny
    Allow from all
    </Directory>
```
- Chown the file `sudo chown root:wheel /etc/apache2/users/<yourusername>.conf`
- Restart Apache with `sudo apachectl restart`

That fixed it for me.
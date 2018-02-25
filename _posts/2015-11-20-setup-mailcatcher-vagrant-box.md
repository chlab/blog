---
layout: post
title: Setup Mailcatcher in Vagrant box
categories:
- Tools
tags:
- php
- vagrant
---
I am currently refactoring some mailings in an application and as you probably know, debugging mails is a PITA. Luckily I stumbled across [Mailcatcher](http://mailcatcher.me/ "Mailcatcher"), which runs a simple mail server and can catch mails you send to it and display them in a web gui.

Since it runs on localhost, it's a bit tricky to set it up in a Vagrant box, where your box probably has it's own hostname (e.g. *myapp.dev*). This blog post is written for Ubuntu 14.04 with PHP and apache.

Here's how to set up Mailcatcher in your Vagrant box.

### Provisioning
First you need to provision your Vagrant box to install Mailcatcher. If you don't already have a provisioning script, you can add one to your Vagrantfile like this:

{% highlight ruby %}
config.vm.provision :shell, path: "vagrant/provision.sh"
{% endhighlight %}

Then create vagrant/provision.sh in your project directory:

{% highlight bash %}
#!/bin/bash

# install mailcatcher (http://mailcatcher.me/)
apt-get install -y ruby-dev libsqlite3-dev
gem install mailcatcher
# enable apache proxy modules to configure a reverse proxy to mailcatchers webfrontend
sudo a2enmod proxy proxy_http proxy_wstunnel

# replace sendmail path in php.ini with catchmail path
CATCHMAIL="$(which catchmail)"
sed -i "s|;sendmail_path\s=.*|sendmail_path = $CATCHMAIL|" /etc/php5/apache2/php.ini

# restart apache
apachectl restart
{% endhighlight %}

Note that I replace the default sendmail path in the php.ini* for apache with Mailcatcher's own *catchmail*, which causes all mails to be sent there. If you also send mails from the command line (e.g. cronjobs), you'll also need to replace sendmail in */etc/php5/cli/php.ini*.

If you're sending your mails with SMTP, you have to configure *smtp_port* instead.

This script is only run automatically when your box is first setup. You can run it manually by calling:

{% highlight bash %}vagrant provision{% endhighlight %}

\*you could also just configure this in your apache config like:

{% highlight bash %}php_admin_value sendmail_path "/usr/local/bin/catchmail"{% endhighlight %}

But I feel more comfortable knowing that it's in my php.ini and won't suddenly start sending mails if I change around my apache config and suddenly that rule is in the wrong Directory or something.

### Starting Mailcatcher on 'vagrant up'
Just add this to your Vagrantfile to start Mailcatcher when you run vagrant up:

{% highlight ruby %}
config.vm.provision :shell, inline: "mailcatcher", run: "always"
{% endhighlight %}

### Accessing Mailcatcher web gui
Since the Mailcatcher web gui, where you can view all captured mails, runs on localhost, it's not accessible via your app or website frontend by default. This is how I configured my Apache virtualhost conf to enable this (only relevant parts):

{% highlight bash %}
ServerName myapp.dev

# proxy pass mailcatcher to internal webserver
ProxyRequests Off
ProxyPass /mailcatcher http://localhost:1080
ProxyPass /assets http://localhost:1080/assets
ProxyPass /messages ws://localhost:1080/messages
{% endhighlight %}

Then restart your apache and you should be able to access *myapp.dev/mailcatcher* in the browser.  Now any mails your app sends should be instantly displayed there instead of actually being sent.

---
layout: post
title: Control LEGO Power Functions trains with Raspberry Pi
---

# Control LEGO Power Functions trains with Raspberry Pi

## Introduction

@todo

## Prerequisites

First, make sure that **vim** is installed (or your favourite text editor of choice):

```
sudo vim
```

Next, make sure that **package information** is up-to-date:

```
sudo apt-get update
```

And finally, make sure that **virtualenv** is installed, so that we can create Python virtual environments.

```
sudo apt-get install virtualenv
```

Okay, now we are good to go!

## Install Apache

Apache is one of the most popular web servers in the world. We will install Apache on the Raspberry Pi and deploy the Flask project to it.

Install Apache with the following command:

```
sudo apt-get install apache2 libapache2-mod-wsgi-py3
```

After that, move to the public HTML directory of Apache:

```
cd /var/www/html
```

Clone the Flask project inside this directory using the following command:

```
sudo git clone https://github.com/braek/legoirblaster.git
```

Make sure that the user **pi** and group **pi** are owners of that directory and its content:

```
sudo chown pi:pi legoirblaster
```

Move inside the project directory:

```
cd legoirblaster
```

Now, create a Python virtual environment using Python 3 and no site packages:

```
virtualenv -p python3 --no-site-packages venv
```

Activate the newly created virtual environment:

```
source venv/bin/activate
```

Now, install all the Python dependencies from the **requirements.txt** file:

```
pip install -r requirements.txt
```

You can deactivate the virtual environment now:

```
deactivate
```

Create a new Apache site configuration:

```
sudo vim /etc/apache2/sites-available/legoirblaster.conf
```

And paste the following content in that file:

```
<VirtualHost *:80>
    WSGIDaemonProcess legoirblaster python-home=/var/www/html/legoirblaster/venv user=pi group=pi threads=5
    WSGIScriptAlias / /var/www/html/legoirblaster/wsgi.py
    Alias /static /var/www/html/legoirblaster/legoirblaster/static
    <Directory /var/www/html/legoirblaster>
        WSGIProcessGroup legoirblaster
        WSGIApplicationGroup %{GLOBAL}
        Require all granted
    </Directory>
</VirtualHost>
```

Disable the default Apache site:

```
sudo a2dissite 000-default
```

Enable the new Apache site:

```
sudo a2ensite legoirblaster
```

And restart Apache:

```
sudo service apache2 restart
```

When you now open a browser, you should be able to see the interface on http://localhost/.

## Deploy Flask project to Apache

@todo

## Install LIRC

First, install LIRC with this command:

```
sudo apt-get install lirc
```

Next, open the **/etc/modules** file:

```
sudo vim /etc/modules
```

And add the following lines:

```
lirc_dev
lirc_rpi
```

*Lots of tutorials and blog posts say that you need to configure the output pin here, but that is actually not necessary.*

Open the **/etc/lirc/hardware.conf** file:

```
sudo vim /etc/lirc/hardware.conf
```

Make sure that it contains the following key-value pairs:

```
LIRCD_ARGS="--uinput"
DRIVER="default"
DEVICE="/dev/lirc0"
MODULES="lirc_rpi"
```

Next, open the **/boot/config.txt** file:

```
sudo vim /boot/config.txt
```

Uncomment the following line:

```
#dtoverlay=lirc-rpi
```

And change it into:

```
dtoverlay=lirc-rpi:gpio_out_pin=22
```

*Note that the GPIO output pin is configured here instead of in the **/etc/modules** file.*

Now, open the **sudo vim /etc/lirc/lircd.conf** file:

```
sudo vim /etc/lirc/lircd.conf
```

To add the LEGO IR Remote here:

```
include "/var/www/html/legoirblaster/lirc/LEGO_Single_Output.conf"
```

We are almost there, but there is still one problem that we haven't encountered yet. You need to start the LIRC device **/dev/lirc0** every time manually when the Raspberri Pi boots.

We can fix this however by opening the **/etc/rc.local** file:

```
sudo vim /etc/rc.local
```

And adding this line:

```
sudo lircd -d /dev/lirc0
```

Right before this line:

```
exit 0
```

Make sure that the line with the **exit** statement is still the last line of this file.

Now, reboot the Raspberry Pi:

```
sudo reboot
```
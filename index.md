# HOW TO INSTALL OWNCLOUD

This guide helps you to install an ownCloud instance on Ubuntu 20.4 manually. ownCloud is a software that helps you to sync, share, and collaborate content. 
You can work on data easily from anywhere and on any device.  
Prerequisites
Ensure that you have a fresh installation of Ubuntu 20.4 LTS with SSH. ownCloud requires a [web server](https://doc.owncloud.com/server/10.6/admin_manual/installation/manual_installation/manual_installation_apache.html),
a [database](https://doc.owncloud.com/server/10.6/admin_manual/installation/manual_installation/manual_installation_db.html), and a [PHP version](https://doc.owncloud.com/server/10.6/admin_manual/installation/manual_installation/manual_installation_prerequisites.html#php-version) to function properly. 

[![setup_server](./media/set_up.PNG)](#set-up-installation-environment)
[![install_cloud](./media/install.PNG)](#install-owncloud-binaries)
[![configure_server](./media/configure.PNG)](#configure-your-owncloud)
[![connect_server](./media/connect.PNG)](#connect-to-your-owncloud)

## Set up installation environment

Before you begin, prepare your server with the required packages to set up the installation environment:
- [Update required package](#update-required-package)
- [Create an occ helper script](#create-occ-helper-script)
- [Install required packages](#install-required-packages)
- [Configure Apache](#configure-apache)
- [Configure database](#configure-database)

### Update required package
Firstly, ensure that you have all the required packages installed and the PHP version available in the APT repository. 
In addition, check whether the installed packages are up to date:

```
apt update && \
apt upgrade -y

```

### Create occ helper script
Create a helper script to simplify the running occ commands:
```
FILE="/usr/local/bin/occ"
/bin/cat <<EOM >$FILE
#! /bin/bash
cd /var/www/owncloud
sudo -u www-data /usr/bin/php /var/www/owncloud/occ "\$@"
EOM
Make thehelper script executable:
chmod +x /usr/local/bin/occ
```
 
### Install required packages
 
```
 apt install -y \
  apache2 \
  libapache2-mod-php \
  mariadb-server \
  openssl \
  php-imagick php-common php-curl \
  php-gd php-imap php-intl \
  php-json php-mbstring php-mysql \
  php-ssh2 php-xml php-zip \
  php-apcu php-redis redis-server \
  wget
```

The php-smbclient package was removed from the official repository. It is available in the ondrej/php repository (ppa).
Add ondrej’s ppa with these commands:
```
sudo add-apt-repository ppa:ondrej/php
sudo apt-get update
echo "deb http://ppa.launchpad.net/ondrej/php/ubuntu $(lsb_release -cs) main" | tee /etc/apt/sources.list.d/php.list
```

Now, add the key:
```
apt-key adv --recv-keys --keyserver hkps://keyserver.ubuntu.com:443 4F4EA0AAE5267A6C
```

You can install the recommended packages using the following command:
```
apt install -y \
ssh bzip2 rsync curl jq \
inetutils-ping smbclient\
php-smbclient coreutils php-ldap
```

### Configure Apache
- Change the document root:
    ```
    sed -i "s#html#owncloud#" /etc/apache2/sites-available/000-default.conf
    service apache2 restart
    ```

- Create a virtual host configuration:

    ```
    FILE="/etc/apache2/sites-available/owncloud.conf"
    /bin/cat <<EOM >$FILE
    Alias /owncloud "/var/www/owncloud/"

    <Directory /var/www/owncloud/>
    Options +FollowSymlinks
    AllowOverride All

    <IfModule mod_dav.c>
      Dav off
    </IfModule>

    SetEnv HOME /var/www/owncloud
    SetEnv HTTP_HOME /var/www/owncloud
    </Directory>
    EOM
    ```

- Enable the virtual host configuration
    ```
    a2ensite owncloud.conf
    service apache2 reload
    ```

### Configure database
Configure your database:

```
mysql -u root -e "CREATE DATABASE IF NOT EXISTS owncloud; \
GRANT ALL PRIVILEGES ON owncloud.* \
  TO owncloud@localhost \
  IDENTIFIED BY 'password'";
```
Enable the Apache modules:

```
echo "Enabling Apache Modules"
a2enmod dir env headers mime rewrite setenvif
service apache2 reload
```

> **Note:**
>
> This guide assumes that you are connected as the root user and your ownCloud directory is located in /var/www/owncloud/

## Install ownCloud binaries
This section helps you to install the ownCloud server quickly.  

### Download ownCloud

1.	Navigate to the [ownCloud download page](https://owncloud.com/download-server/) and select the required package. You can download either the .tar.bz2 or .zip archive. Change to a directory where you want to install ownCloud. 
2.	To download, copy the link of the selected file and run following command:

    ```
    cd /var/www/
    wget https://download.owncloud.org/community/owncloud-complete-yyyymmdd.tar.bz2
    ```

3.	Extract the archive contents, run the unpacking command for your tar archive, and set permissions:

    ```tar -xjf owncloud-10.6.0.tar.bz2 && \
      chown -R www-data. owncloud
    ```
    > **Note:**
    >
    > This guide assumes that you are connected as the root user and your ownCloud directory is located in /var/www/owncloud/

The .tar package unpacks to a single owncloud directory. Copy the ownCloud directory to its final destination. When you are running the Apache HTTP server, you may safely install your ownCloud in your Apache document root.

After restarting Apache, you must complete your installation by running either the Graphical Installation Wizard or the command line with the occ command.
### Install ownCloud

Install ownCloud using the following command:
```
     occ maintenance:install \
    --database "mysql" \
    --database-name "owncloud" \
    --database-user "owncloud" \
    --database-pass "password" \
    --admin-user "admin" \
    --admin-pass "admin"
```

Your ownCloud instance is now ready to use.

To finalize the installation, you can also use the installation wizard. For more information, see [Graphical Installation Wizard](https://doc.owncloud.com/server/10.6/admin_manual/installation/installation_wizard.html).

## Configure your ownCloud

After installing ownCloud successfully, it is recommended to perform some post-installation tasks. 
These tasks help you to configure background jobs or improve performance by caching.

To access ownCloud using a URL, you must define your ownCloud URL in the config.php file, 
under the trusted_domains setting. This setting is important when you are changing or moving to a new domain name. 

For more information, see how to configure your ownCloud.

This guide assumes that the ownCloud server is accessible using the default route /owncloud. 
If you want, you [can change your ownCloud URL](configure-owncloud.md#change-your-owncloud-url) in your webserver configuration.

> **Note:**
>
> On CentOS/Fedora/Red Hat, edit /etc/httpd/conf.d/owncloud.conf, /var/www/html/owncloud/config/config.php, and then restart Apache.

## Connect to your ownCloud 

Synchronize the files in your local shared directories to the server and other devices using your ownCloud. 
You can connect to your ownCloud server either using any Web browser or any of the following:

- [ownCloud Desktop Synchronization Client](connect-to-owncloud.md#owncloud-desktop-synchronization-client)
- [ownCloud Mobile App](connect-to-owncloud.md#owncloud-mobile-app)

Access your ownCloud account, share files, create content, [add users](configure-owncloud.md#create-user) and so on.





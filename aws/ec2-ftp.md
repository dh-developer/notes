### FTP Setup in Amazon EC2 instance

#### Step #1: Install vsftpd

> SSH to your EC2 server. Type:


    sudo yum install vsftpd

#### Step #2: Open up the FTP ports on your EC2 instance

> Next, you'll need to open up the FTP ports on your EC2 server. Log in to the AWS EC2 Management Console and select Security Groups from the navigation tree on the left. Select the security group assigned to your EC2 instance. Select the Inbound tab and add port range 20-21

> Also add port range 1024-1048

Type | Protocol | Port Range | Source 
------------- | ------------- | ------------- | -------------
SSH  | TCP  |  22  |  ip/32
HTTP  | TCP  |  80  |  0.0.0.0/0
Custom TCP Rule  | TCP  |  20 - 21  |  0.0.0.0/0
Custom TCP Rule  | TCP  |  1024 - 1048  |  0.0.0.0/0

#### Step #3: Make updates to the vsftpd.conf file

> Edit your vsftpd conf file by typing:

    sudo nano /etc/vsftpd/vsftpd.conf
    
> Disable anonymous FTP by changing this line:

    anonymous_enable=YES
to

    anonymous_enable=NO

> Then add the following lines to the bottom of the vsftpd.conf file:

    pasv_enable=YES
    pasv_min_port=1024
    pasv_max_port=1048
    pasv_address=<Public IP of your instance>

> Your vsftpd.conf file should look something like the following - except make sure to replace the pasv_address with your public facing IP address:

```
pam_service_name=vsftpd
userlist_enable=YES
tcp_wrappers=YES

# Additional configuration
pasv_enable=YES
pasv_min_port=1024
pasv_max_port=1048
pasv_address=xx-xxx-xxx-xx
local_root=/var/www/html
```

#### Step #4: Restart vsftpd

> Restart vsftpd by typing:

    sudo /etc/init.d/vsftpd restart

#### Step #5: Create an FTP user

> If you take a peek at /etc/vsftpd/user_list, you'll see the following:

```
# vsftpd userlist
# If userlist_deny=NO, only allow users in this file
# If userlist_deny=YES (default), never allow users in this file, and
# do not even prompt for a password.
# Note that the default vsftpd pam config also checks /etc/vsftpd/ftpusers
# for users that are denied.
root
bin
daemon
adm
lp
sync
shutdown
halt
mail
news
uucp
operator
games
nobody
```

> This is basically saying, "Don't allow these users FTP access." vsftpd will allow FTP access to any user not on this list.

> So, in order to create a new FTP account, you may need to create a new user on your server. (Or, if you already have a user account that's not listed in /etc/vsftpd/user_list, you can skip to the next step.)

> Creating a new user on an EC2 instance is pretty simple. For example, to create the user `gunjan`, type:

    sudo adduser gunjan-ftp
    sudo passwd gunjan-ftp

#### Step #6: Restricting users to their home directories

> At this point, your FTP users are not restricted to their home directories. That's not very secure, but we can fix it pretty easily.

> Edit your vsftpd conf file again by typing:

    sudo nano /etc/vsftpd/vsftpd.conf

> Un-comment out the line:

    chroot_local_user=YES
    
> Restart the vsftpd server again like so:

    sudo /etc/init.d/vsftpd restart

#### Surviving a reboot

> vsftpd doesn't automatically start when your server boots. If you're like me, that means that after rebooting your EC2 instance, you'll feel a moment of terror when FTP seems to be broken - but in reality, it's just not running!. Here's a handy way to fix that:

    sudo chkconfig --level 345 vsftpd on

_Alternatively, if you are using redhat, another way to manage your services is by using this nifty graphic user interface to control which services should automatically start: `sudo ntsysv`_

#### To change the default FTP upload folder

> Edit

    edit /etc/vsftpd/vsftpd.conf

> Create a new entry at the bottom of the page:

    local_root=/var/www/html

> To apply read, write, delete permission to the files under folder so that you can manage using a FTP device

    sudo find /var/www/html -type d -exec chmod 755 {} \;
    
#### If still doesn't working

> It will not be ok until you add your user to the group www by the following commands:

    sudo usermod -a -G www <USER>

> _Note that you will probably need to add the user you created to the "FTP" usergroup:_

    gpasswd -a <usr> ftp

**This documented is generated based on http://stackoverflow.com/questions/7052875/setting-up-ftp-on-amazon-cloud-server?answertab=votes#answer-11404078**

### Wordpress Plugin Updates

Add ```define('FS_METHOD', 'direct');``` to the bottom of wp-config.php

Barnyard2 is a way to store and process the binary outputs from Snort into a MySQL database.

# Update, Upgrade, and Reboot
Before we actually get our hands into the Snort (S) sources, we need to make sure our system is up to date. We can do this by issuing the commands below.

    sudo apt-get update
    sudo apt-get upgrade -y
    sudo reboot
 
# Pre-install configuration
If you don't have MySQL installed you can install it with the following command, 

    sudo apt-get install -y mysql-server libmysqlclient-dev mysql-client autoconf libtool

If you don't have the network intrusion detection system (IDS) Snort installed and configured, please consult the documentation [installation documentation](https://www.vultr.com/docs/how-to-configure-snort-on-debian)

#Setting up Barnyard2
In order to install Barnyard we need to grab the source from [Barnyard2's github page](https://github.com/firnsy/barnyard2).

    cd /usr/src
    sudo git clone https://github.com/firnsy/barnyard2 barnyard_src
    cd barnyard_src

Now that we have the source for barnyard we need to `autoreconf` barnyard.

    sudo autoreconf -fvi -I ./m4

#### Update system library references

Once that is finished have to make a symlink to the dumbnet library as dnet.

    sudo ln -s /usr/include/dumbnet.h /usr/include/dnet.h

Because we essentially made a new system library we have to update the system's library cache. This can be done by issuing the following command:

    sudo ldconfig

#### Configuring Barnyard2 for MySQL
This part is important because it depends on whether or not your system is a 64 bit system or a 32 bit system.

If you are unsure as to whether or not your system is 64 bit or 32 bit, you can either use `uname -m` or `arch` to achieve this.

    cd /usr/src/barnyard_src
    ./configure --with-mysql --with-mysql-libraries=/usr/lib/YOUR-ARCH-HERE-linux-gnu

So that configuration should look like `./configure --with-mysql --with-mysql-libraries=/usr/lib/x86_64-linux-gnu`

    make
    sudo make install

#### Copying configurations 
In order to set up barnyard properly and let it work with our system we need to copy over our configuration files. Also, please note, while I tested this I had to create the log directory for barnyard2 otherwise running it would fail. 

    sudo cp etc/barnyard2.conf /etc/snort
    sudo mkdir /var/log/barnyard2
    sudo chown snort.snort /var/log/barnyard2
    sudo touch /var/log/snort/barnyard2.bookmark
    sudo chown snort.snort /var/log/snort/barnyard2.bookmark

#### Creating the database

Now that our barnyard instance has been mostly set up we need to create and associate a database with our setup.

     mysql -u root -p
     create database snort;
     use snort;
     source /usr/src/barnyard_src/schemas/create_mysql
     CREATE USER 'snort'@'localhost' IDENTIFIED BY 'MYPASSWORD';
     grant create, insert, select, delete, update on snort.* to snort@localhost;
     exit;


#### Configuring barnyard for use with MySQL
In case you didn't happen to change the password in the above command, you can reset the password by re-entering the mysql command and entering

    SET PASSWORD FOR 'snort'@'localhost' = PASSWORD( 'MYPASSWORD' );

At the very bottom of your `/etc/snort/barnyard2.conf` file add the following and edit the password to what you set above. 

    output database: log, mysql, user=snort password=MYPASSWORD dbname=snort host=localhost

For security purposes, we need to lock down our barnyard.conf file because it contains your database password in cleartext.

    sudo chmod o-r /etc/snort/barnyard2.conf

# Testing
You can test snort by having it run in alert mode using your config file.

    sudo /usr/local/bin/snort -q -u snort -g snort -c /etc/snort/snort.conf -i eth0

Once snort is running, open another terminal and ping that system's address, you should be able to see the messages on your main terminal. 

Now that you have some data in your snort logs, you should be able to test barnyard against it.

    sudo barnyard2 -c /etc/snort/barnyard2.conf -d /var/log/snort -f snort.u2 -w /var/log/snort/barnyard2.bookmark -g snort -u snort

These flags basically mean the following.

    -c   specifies the config file.
    -d   is the snort output directory
    -f    specifies the file to look for.
    -w   specifies the bookmark file.
    -u / -g   tells barnyard to run as a specific user and group.

After starting barnyard, once `Waiting for new data` appears you can quit the application by pressing `ctrl + c` now to check your MySQL database by logging back into the MySQL server and selecting all from the `event` table in your `snort` database.

    mysql -u snort -p snort
    select count(*) from event;

As long as the count is more than 0 everything worked properly!

However, if the count IS 0, you're probably either pinging your system from a system that matches a whitelisted ip. If that is the case, try pinging your system from out side your network and to make sure that is exposed to the outside world.

**Congratulations, you now have a way to read and keep track of your detected intrusions.**

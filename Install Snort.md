# Background
  **Q:**  What is Snort? 

  **A:** Snort is a *free* network intrusion detection system (IDS). In less official terms, it lets you to monitor your network for suspicious activity *in real time*. 


  **Q:** Is my OS supported?

  **A:** Currently, Snort has packages for Fedora, CentOS, FreeBSD, and windows based systems. Exact installation methods vary between OSes. We will be installing directly from the source files for Snort (S). So this means we can support many more distros.
**TL;DR** *Probably*

# Update, Upgrade, and Reboot
Before we actually get our hands into the Snort (S) sources, we need to make sure our system is up to date. We can do this by issuing the commands below.

    sudo apt-get update
    sudo apt-get upgrade -y
    sudo reboot
 
# Pre-install configuration
Once your system has rebooted, we need to install a number of packages to make sure that we can install SBPP. I was able to figure out a number of the packages that were needed so I have the base command below.

    sudo apt-get install flex bison build-essential checkinstall libpcap-dev libnet1-dev libpcre3-dev libmysqlclient15-dev libnetfilter-queue-dev iptables-dev libdumbnet-dev -y

Once all of the packages are installed you will need to create a temporary directory for your sources files, they can be where ever you want, we'll have it in `/usr/src/snort_src`. So to create this folder you'll need to be root or have sudo permissions, root just makes it easier.
 
    sudo mkdir /usr/src/snort_src
    cd /usr/src/snort_src

# Installing the Data AcQuisition library (DAQ)
Just before we can get the source for Snort we need to install the DAQ. It's fairly simple, to install you can just type whats below

Download the file from here using the command, 

    wget https://www.snort.org/downloads/snort/daq-2.0.6.tar.gz

Extract the files from the tar ball

    tar xvfz daq-2.0.6.tar.gz

Change into the DAQ directory

    cd daq-2.0.6

Configure and install the DAQ

    ./configure; make; sudo make install

That last line, will execute `./configure` first. Then it will execute `make`. Lastly it will execute `make install`. We use the shorter syntax here just to save a little bit on typing.

# Installing Sort.

We want to make sure we're in the `/usr/src/snort_src` directory again, so be sure to change into that directory with

    cd /usr/src/snort_src

Now that we are in the directory for the sources, we should download the `tar.gz` file for the source. At the time of this writing, (09/24/2015) the most recent version of Snort is 2.9.7.5

    wget https://www.snort.org/downloads/snort/snort-2.9.7.5.tar.gz

The commands to actually install snort are very similar to the ones used in the DAQ, but they have different options.

Extract the Snort source files
    
    tar xvfz snort-2.9.7.5.tar.gz

Change into the source directory

    cd snort-2.9.7.5

Configure and install the sources

     ./configure --enable-sourcefire; make; sudo make install

### Post-install of Snort
Once we have Snort installed, we need to make sure that our shared libraries are up to date. We can do this using the command 
   
    sudo ldconfig

After we do that, we now need to make sure that snort is installed, we can do this by typing

    snort --version


If this command does not work you will need to create the symlink you can do this by typing

    sudo ln -s /usr/local/bin/snort /usr/sbin/snort

The resulting output should be something like this....

       ,,_     -*> Snort! <*-
      o"  )~   Version 2.9.7.5 GRE (Build 262)
       ''''    By Martin Roesch & The Snort Team: http://www.snort.org/contact#team
               Copyright (C) 2014-2015 Cisco and/or its affiliates. All rights reserved.
               Copyright (C) 1998-2013 Sourcefire, Inc., et al.
               Using libpcap version 1.6.2
               Using PCRE version: 8.35 2014-04-04
               Using ZLIB version: 1.2.8

# Un-rooting Snort
Now that we have snort installed, we don't want it running as root, so we need to create a Snort user and group. To create a new user and group we can use these two commands.

    sudo groupadd snort
    sudo useradd snort -r -s /sbin/nologin -c SNORT_IDS -g snort

Since we're installing the program using the source, we need to create the configuration files and the rules for snort.

    sudo mkdir /etc/snort
    sudo mkdir /etc/snort/rules
    sudo mkdir /etc/snort/preproc_rules
    sudo touch /etc/snort/rules/white_list.rules /etc/snort/rules/black_list.rules /etc/snort/rules/local.rules

After we create the directories and the rules, we now need to create the log directory

    sudo mkdir /var/log/snort

and lastly, before we can add any rules we need a place to store any of the dynamic rules,

    sudo mkdir /usr/local/lib/snort_dynamicrules

Then once we have everything created we need to make sure that they all have the proper permissions.

    sudo chmod -R 5775 /etc/snort
    sudo chmod -R 5775 /var/log/snort
    sudo chmod -R 5775 /usr/local/lib/snort_dynamicrules
    sudo chown -R snort:snort /etc/snort
    sudo chown -R snort:snort /var/log/snort
    sudo chown -R snort:snort /usr/local/lib/snort_dynamicrules

## Setting up the config files

To save a bunch of time and to keep from having to copy and paste everything, lets just copy all of the files in the configuration directory.

    sudo cp /usr/src/snort_src/snort*/etc/*.conf* /etc/snort
    sudo cp /usr/src/snort_src/snort*/etc/*.map /etc/snort

but now that the config files are there, you can do one of two things. 

  *  You can enable Barnyard2
  *  Or you can just leave the config files alone and selectively enable the desired rules

Either way, you're still going to want to change a few things.

## Configuration

Here is where you may need a little help. In the `/etc/snort/snort.conf` file, you will need to change the variable `HOME_NET` it should be set to your internal network's ip block so it won't log your own network's attempts to log into the server. This may be `10.0.0.0/24` or `192.168.1.0/8`. So on line 45 of `/etc/snort/snort.conf` change the value of `HOME_NET` to that value of your network's ip block. 

ex.

    ipvar HOME_NET 192.168.1.0/8




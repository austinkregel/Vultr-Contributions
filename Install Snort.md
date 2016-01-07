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

    sudo apt-get install flex bison build-essential checkinstall libpcap-dev libnet1-dev libpcre3-dev libnetfilter-queue-dev iptables-dev libdumbnet-dev -y

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

Now that we are in the directory for the sources, we should download the `tar.gz` file for the source. At the time of this writing, (01/07/2016) the most recent version of Snort is 2.9.8.0

    wget https://www.snort.org/downloads/snort/snort-2.9.8.0.tar.gz

The commands to actually install snort are very similar to the ones used in the DAQ, but they have different options.

Extract the Snort source files
    
    tar xvfz snort-2.9.8.0.tar.gz

Change into the source directory

    cd snort-2.9.8.0

Configure and install the sources

     ./configure --enable-sourcefire; make; sudo make install

### Post-install of Snort
Once we have Snort installed, we need to make sure that our shared libraries are up to date. We can do this using the command 

    sudo ldconfig

After we do that, we now need to make sure that snort is installed, we can do this by typing

    snort --version


If this command does not work you will need to create the symlink you can do this by typing

    sudo ln -s /usr/local/bin/snort /usr/sbin/snort
    snort --version

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

Here is where you may need a little help. In the `/etc/snort/snort.conf` file, you will need to change the variable `HOME_NET` it should be set to your internal network's ip block so it won't log your own network's attempts to log into the server. This may be `10.0.0.0/24` or `192.168.0.0/16`. So on line 45 of `/etc/snort/snort.conf` change the variable `HOME_NET` to that value of your network's ip block. 

For me it's :

    ipvar HOME_NET 192.168.0.0/16

Then you'll have to set the `EXTERNAL_NET` variable to 

    any
  
Which just turns EXERNAL_NET into whatever your HOME_NET isn't.

#### Setting the rules

Now that a large majority of the system is set up we need to configure our rules for this little piggy. Somewhere around line 104 in your `/etc/snort/snort.conf` file, you should see a var declartion and the variables `RULE_PATH`, `SO_RULE_PATH`, `PREPROC_RULE_PATH`, `WHITE_LIST_PATH`, and `BLACK_LIST_PATH`. Now their values should be paths we set up in `Unrooting Snort`.

    var RULE_PATH /etc/snort/rules
    var SO_RULE_PATH /etc/snort/so_rules
    var PREPROC_RULE_PATH /etc/snort/preproc_rules
    var WHITE_LIST_PATH /etc/snort/rules
    var BLACK_LIST_PATH /etc/snort/rules

Once those values are set, we have to make sure you *delete or comment out the current rules starting on about line 548*.

Now lets check to make sure that your config is all set up properly. You can do that by using the flowing command.

     # snort -T -c /etc/snort/snort.conf
     
You should be able to get something like the following (truncated for brevity)
     Running in Test mode
     
             --== Initializing Snort ==--
     Initializing Output Plugins!
     Initializing Preprocessors!
     Initializing Plug-ins!
     .....
     Rule application order: activation->dynamic->pass->drop->sdrop->reject->alert->log
     Verifying Preprocessor Configurations!
     
             --== Initialization Complete ==--
     
        ,,_     -*> Snort! <*-
       o"  )~   Version 2.9.8.0 GRE (Build 229) 
        ''''    By Martin Roesch & The Snort Team: http://www.snort.org/contact#team
                Copyright (C) 2014-2015 Cisco and/or its affiliates. All rights reserved.
                Copyright (C) 1998-2013 Sourcefire, Inc., et al.
                Using libpcap version 1.7.4
                Using PCRE version: 8.35 2014-04-04
                Using ZLIB version: 1.2.8
     
                Rules Engine: SF_SNORT_DETECTION_ENGINE  Version 2.4  <Build 1>
                Preprocessor Object: SF_IMAP  Version 1.0  <Build 1>
                Preprocessor Object: SF_FTPTELNET  Version 1.2  <Build 13>
                Preprocessor Object: SF_SIP  Version 1.1  <Build 1>
                Preprocessor Object: SF_REPUTATION  Version 1.1  <Build 1>
                Preprocessor Object: SF_POP  Version 1.0  <Build 1>
                Preprocessor Object: SF_DCERPC2  Version 1.0  <Build 3>
                Preprocessor Object: SF_SDF  Version 1.1  <Build 1>
                Preprocessor Object: SF_GTP  Version 1.1  <Build 1>
                Preprocessor Object: SF_DNS  Version 1.1  <Build 4>
                Preprocessor Object: SF_SSH  Version 1.1  <Build 3>
                Preprocessor Object: SF_DNP3  Version 1.1  <Build 1>
                Preprocessor Object: SF_SSLPP  Version 1.1  <Build 4>
                Preprocessor Object: SF_SMTP  Version 1.1  <Build 9>
                Preprocessor Object: SF_MODBUS  Version 1.1  <Build 1>
     
     Snort successfully validated the configuration!
     Snort exiting

Now that we are all configured and we don't have any errors, we're ready to start testing Snort.

## Testing Snort

By far the easiest way to test Snort would be by enabling the local.rules. This is just a place to set your custom rules for your implementation of Snort.

If you noticed in the snort.conf file, some where around line 546, you should have 

    include $RULE_PATH/local.rules

If you don't have it, please add it around 546. Once that is all set up you'll need to set up the local.rules file for testing. Just something small, I'll just have it keep track of a ping request (ICMP request). You can do that by adding in the following line to your local.rules file.

     alert icmp any any -> $HOME_NET any (msg:"ICMP test"; sid:10000001; rev:001;)
     
Once you have that in your file, save it, and then lets get to the testing.

## Getting to the testing
The following command will start snort and it will print "fast mode" alerts, as the user snort, under the group snort, using the config `/etc/snort/snort.conf`, and it will listen on the network interface `eno1`. You will need to change eno1 to whatever network interface your system is listening on.

    $ sudo /usr/local/bin/snort -A console -q -u snort -g snort -c /etc/snort/snort.conf -i eno1

Once you have it running, ping that computer. You should start to see something that looks like the following.

    01/07−16:03:30.611173 [∗∗] [1:10000001:0] ICMP test [∗∗] [Priority: 0] {ICMP} 192.168.1.105 −> 192.168.1.104
    01/07−16:03:31.612174 [∗∗] [1:10000001:0] ICMP test [∗∗] [Priority: 0] {ICMP} 192.168.1.104 −> 192.168.1.105
    01/07−16:03:31.612202 [∗∗] [1:10000001:0] ICMP test [∗∗] [Priority: 0] {ICMP} 192.168.1.105 −> 192.168.1.104
    ˆC∗∗∗ Caught Int−Signal

You can press ctrl+c to exit the program, and that's it. Snort is all set up. 

## Sidenote:
I want to note that there are some public rules made by the community you can download from [snort.org](https://www.snort.org/downloads#rule-downloads) under the "Community" tab, look for 'Snort', then just under that there is a community link. Download that, extract it, and look for the `community.rules` file.

You'll have to uncomment most of the file, but the rules they have there are really use full.

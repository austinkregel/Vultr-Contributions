Having only one user, which is root, can be **dangerous**. So let's fix that. Vultr provides us with the freedom to do as we please with our users and our servers. Let's make use of this by adding a user with **sudo** access instead of **direct root access**.

## Add a new user

We need to first connect to the server with root so that we have adequate permissions. Once connected, add another user account.
 
    # useradd <username>

Replace `<username>` with the desired username.

That command will add the user to the list of users on the system, and create a corresponding group (if the group doesn't exist).

## Edit the hostname

With the default hostname set, sudo with throw the following error.
 
     sudo: unable to resolve host vultr.guest

We can prevent this error by changing the hostname. Your hostname is located after the "@" symbol in the shell. For example, `root@<hostname> ~#`.

Edit both `/etc/hosts/` and `/etc/hostname` to update your hostname. Save the files when you're done editing them.

    nano /etc/hosts
    nano /etc/hostname

## Restart the server

You must restart the server for the hostname change to take effect. Once the server finishes rebooting, log back in to continue.

    reboot

## Add a sudo entry

Even if you're root, you will need to run the "sudo" part of the following command.

    sudo visudo

This will bring up a file with a bit of default info. What you need to look for is the following section:

    # Cmnd alias specification

    # User privilege specification
    root    ALL=(ALL:ALL) ALL

Find where "root" appears in the list shown above, and add the following. Substitute the appropriate variables.

    <username> ALL=(ALL:ALL) ALL

Upon saving this file, the editor will try to save it as a "tmp" file. Erase the ".tmp" from the line and then press "enter".

## Finish setting up user account

We are almost finished. A few more minor steps.

Create the home folder for the new user.

    mkdir /home/<username>

Give the new user permissions in that folder.

    chown <username>:<usergroup> /home/<username> -R

Set a password for the new user.

    sudo passwd <username>

Add the following to the `/etc/passwd` file.

    <username>:x:1000:1000::/home/<username>:/bin/bash

Reboot the server again for these changes to take effect.

     reboot 

## Using sudo

At this point, your new user account is ready-to-go with sudo access. You may log into the server with this new account and use `sudo` when necessary.

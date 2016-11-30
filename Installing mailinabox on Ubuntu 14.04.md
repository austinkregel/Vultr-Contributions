## Important pre-deployment notice
In order to install [mailinabox](https://mailinabox.email) on a Vultr server the VPS **MUST** 

 * Have **at least 1024MB of RAM**
 * Must be running Ubuntu **14.04** any version of 14.04 will do, but it must be 14.04. 
    * (At the time of this writing Ubuntu 14.04 is the only supported OS)
 * It's STRONGLY recommended that you have a domain and a subdomain.
    * I personally use a `mail` prefix do my domain. So if you have `example.com` I would use `mail.example.com`

## Deploying the new server
 
* Once you're signed in, click the plus button in the top right corner. 
* Select your desired location
* Select Ubuntu then be sure to select the **14.04** version.
* Select the server size that has **at least** 1024MB of RAM
* Lastly, click `Deploy Now`

## Post Server Configuration

Once the server has finished installing, grab the server's IP and update the subdomain's record to that IP. Allow 24-48 hours to update globally; however, if you are using Vultr's DNS it shouldn't take more than a moment or two to update.

# The hard stuff

Now for installing [mailinabox](https://mailinabox.email)... :D

To do the installation it's just once simple command.

    curl -s https://mailinabox.email/setup.sh | sudo bash

## Installation

1.  your new email. It should be the main account for your email server. Make note that what ever email you choose it will be the server's defacto email. Mailinabox will also make an alias for 3 very common address (abuse, postmaster, administrator) that will forward mail to your defacto email.

2. the hostname of the server. Delete the current value and enter the mail subdomain that we configured earlier. For example `mail.example.com`. Then press enter.

3.  select your (general) timezone, and the desired region for your timezone. Then press enter
Mailinabox will execute an `apt-get update` in the background and start installing the necessary components. 

4. After a little while you'll be prompted for your defacto email account.

5. Once it finishes it will display a link, something along the lines of `https://mail.example.com/admin`. Visit your URL and from that interface you can add users, update your server's DNS entries, Sync your calendars, contacts, your OwnCloud installation, and the webroot for your domains.


Congratulations! You now manage your own mail, dns, cloud, and web server!

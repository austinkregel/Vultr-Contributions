##__Step 1__
Login to ssh, and then copy and paste the following command,

**Note this is about a 300MB installation**

    sudo apt-get install apache2 mysql-server php5-mysql php5 libapache2-mod-php5 php5-mcrypt php5-cli php5-cgi php5-curl php5-gd php5-geoip php5-dbg php5-json php5-mysql phpmyadmin -y 

##__Step 2__
Enjoy your installation. That's right... If you were able to follow the prompts that appear in the shell, you're done! If you want to test your connection, just visit the IP Address you used SSH to connect to. 

For example, if you used the following command, `user@10.0.0.2` your IP would be `10.0.0.2`, visit that address in your web browser (http://10.0.0.2) and you should be greeted with the basic apache2 installation file.

##__Note__

Your document root is located in 

    /var/www/html

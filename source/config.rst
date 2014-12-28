.. _config:

########################################################################
Configuring Odoo Installation
########################################################################

************************************************************************
Create infomobius unix account
************************************************************************

This step creates infomobius unix account. This account will be used to
store various things like

* Thai Fonts
* Flask or Back up script
* Installation Script
* Any other non-odoo stuff

Steps::

    sudo su - 
    adduser infomobius
    su infomobius


************************************************************************
Installing Thai Fonts
************************************************************************

* Clone a repo from github ``sudo git clone git://github.com/pwarawit/thaifonts.git``
* Copy fonts from to /usr/share/fonts
* refresh font ``sudo fc-cache -f -v``
* Modify the customfonts.py file 
* restart odoo server
* clone a repo ``sudo git clone git://github.com/pwarawit/thaifonts.git``

************************************************************************
Create Custom Addons directory
************************************************************************

``/home/odoo/addons``

Create new folder to keep odoo custom addons under odoo ::

    mkdir /home/odoo/custom_addons
    cd /home/odoo/custom_addons
    git clone git://github.com/pwarawit/mdc_adblock.git
    git clone git://github.com/pwarawit/web_printscreen_zb.git
    git clone git://github.com/pwarawit/mdc_custom.git
    git clone git://github.com/pwarawit/mdc_discount.git
    
Modify odoo-server.conf file ::

    sudo vi /etc/odoo-server.conf

Add /home/odoo/custom_addons into addons_path (separate by comma)
Then need to restart odoo server.
	

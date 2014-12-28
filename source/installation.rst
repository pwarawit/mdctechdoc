.. _installation:

########################################################################
Installation of ModernCare ERP system
########################################################################
Last Update: |today|

.. toctree::
   :maxdepth: 2
   
This guide describes steps to install OpenERP for ModernCare. 

************************************************************************
Requirements
************************************************************************
- Google Account with developer option enabled


************************************************************************
Create New Instance
************************************************************************
First step is to create new GCE (Google Compute Engine) instance. 
This is a virtual machine hosted by Google Cloud Platform.

#. Log on with your Google Account
#. Goto Developer Console http://console.developers.google.com
#. Create new Google project (if not already exist). Each project will have its own billing.

    * Project Name: ``MDC ERP``
    * Project ID: ``mdc-erp``

#. Create new Compute Engine Instance
    * Goto **Compute** --> **Compute Engine** --> **VM Instance**
    * Then click ``Create an instance``
    * On ``Create a new instance`` screen 
        - Name: ``mdc-erp-instance-1``
        - Allow HTTP Traffic: ``checked``
        - Allow HTTPS Traffic: ``checked``
        - Zone : Any of the ``asia`` zone
        - Machine Type: ``n1-standard-1 (1 vCPU, 3.8 GB memory)``
        - Boot Source: ``New disk from image``
        - Image: ``centos7-v20141205`` *It can be other date*
        - Boot Disk Type: ``Standard Persistent Disk``
        - Delete boot disk when instance is deleted: ``unchecked`` *This is important*
        - Networking: ``Ephemeral``
    * Then click ``Create``


#. After a while, new instance will be created. Take note of the External IP address assigned to it. 


.. note:: 
    We can later on changed the IP address assigned to particular instance. 


************************************************************************
Modify instance firewall rule
************************************************************************
This step is required to allow outside communication to access the instance via http port 8069 used by odoo. 

#. Go to the instance detail page
#. Under ``Network`` section, there is a like for network rule called ``defult``, click it.
#. In the ``Firewall rules`` section, locate the rule called ``default-allow-http``, expand it.
#. Click ``Edit`` inside the ``default-allow-http`` firewall rule.
#. On the dialog of ``Edit firewall rule "default-allow-http"`` change protocols & ports to ``tcp:80,8069``, and save. 


************************************************************************
Access the instance using SSH
************************************************************************
The easiest way to access to the running instance is through the SSH via web browser. 
In the VM Instances screen with the list of instance, on the right-most there is connect column 
with **SSH** displayed.
Select ``Open in browser windows`` opiton will launch another browser window with SSH session to the instance
under your Google user account of with **sudo** priviledge.


************************************************************************
Installing Odoo 7 into CentOS7
************************************************************************
After `Create New Instance`_ and `Access the instance using SSH`_ , now it is time to start intsalling Odoo into it.

#. Switch user to root ::

    sudo su - 


#. Update yum database ::

    yum -y update 

#. Create 2 strong passwords ::

    ODOO_POSTGRES_PASSWORD=`< /dev/urandom tr -dc _A-Z-a-z-0-9 | head -c${1:-20};echo;`
    ODOO_DB_ADMIN_PASSWORD=`< /dev/urandom tr -dc _A-Z-a-z-0-9 | head -c${1:-20};echo;`

#. Install PostgreSQL, initialize database and enabling service to start when reboot ::

    yum -y install postgresql-server postgresql-devel
    postgresql-setup initdb
    systemctl enable postgresql.service

#. Configure PostgreSQL so it will accept connection using MD5 ::

    cp /var/lib/pgsql/data/pg_hba.conf /var/lib/pgsql/data/pg_hba.conf.orig
    sed -i "/^host/s/ident/md5/g" /var/lib/pgsql/data/pg_hba.conf

#. Start Postgres Server and add odoo user to Postgres ::

    systemctl start postgresql.service
    echo -e "$ODOO_POSTGRES_PASSWORD\n$ODOO_POSTGRES_PASSWORD\n" | su - postgres -c "createuser --createdb --username postgres --no-createrole --no-superuser --pwprompt odoo"

#. Install some yum packages that will be required before installying Python virtual environment ::

    yum -y install wget gcc zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libffi-devel libxslt libxslt-devel libxml2 libxml2-devel openldap-devel libjpeg-turbo-devel openjpeg-devel libtiff-devel libyaml-devel python-virtualenv git libpng12 libXext xorg-x11-fonts-Type1

#. Install *wkhtmltopdf* via rpm - this package needed to create PDF in odoo ::

    rpm -ivh http://downloads.sourceforge.net/project/wkhtmltopdf/0.12.1/wkhtmltox-0.12.1_linux-centos6-amd64.rpm
    ln -s /usr/local/bin/wkhtmltopdf /usr/bin/

#. Install Microsoft core fonts ::

    rpm -ivh http://dl.fedoraproject.org/pub/epel/7/x86_64/l/libmspack-0.4-0.2.alpha.el7.x86_64.rpm
    rpm -ivh http://dl.fedoraproject.org/pub/epel/7/x86_64/c/cabextract-1.4-4.el7.x86_64.rpm
    rpm -ivh https://downloads.sourceforge.net/project/mscorefonts2/rpms/msttcore-fonts-installer-2.6-1.noarch.rpm

#. Create odoo Unix user ::

    adduser odoo

#. Create some odoo directories and make odoo its owner ::

    DIR="/var/run/odoo /var/log/odoo /opt/odoo"
    for NAME in $DIR
    do
    if [ ! -d $NAME ]; then
    mkdir $NAME
    chown odoo.odoo $NAME
    fi
    done

#. Log on as odoo ::

    su - odoo

#. Create virtual environment under odoo user and activate it ::

    /bin/virtualenv odoo
    source odoo/bin/activate

#. Install Python modules under odoo virtual environment ::

    pip install http://download.gna.org/pychart/PyChart-1.39.tar.gz
    pip install babel
    pip install docutils
    pip install feedparser
    pip install gdata
    pip install Jinja2
    pip install mako
    pip install mock
    pip install psutil
    pip install psycopg2
    pip install pydot
    pip install python-dateutil
    pip install python-openid
    pip install pytz
    pip install pywebdav
    pip install pyyaml
    pip install reportlab
    pip install simplejson
    pip install unittest2
    pip install vatnumber
    pip install vobject
    pip install werkzeug
    pip install xlwt
    pip install pyopenssl
    pip install lxml
    pip install python-ldap
    pip install pillow
    pip install decorator
    pip install requests
    pip install pyPdf
    pip install wkhtmltopdf
    pip install passlib

#. Install odoo 7 from github (under odoo name) and then exit odoo Unix user, return to root ::

    cd /opt
    git clone https://github.com/odoo/odoo.git --branch 7.0
    chown -R odoo.odoo odoo
    exit

#. Create odoo-server.conf file ::

    cat > /etc/odoo-server.conf << EOF
    [options]
    ; This is the password that allows database operations:
    admin_passwd = $ODOO_DB_ADMIN_PASSWORD
    ; DATABASE OPTIONS
    db_host = localhost
    db_port = 5432
    db_user = odoo
    db_password = $ODOO_POSTGRES_PASSWORD
    ; MISC SETTINGS
    addons_path = /opt/odoo/addons
    without-demo=all
    no-xmlrpc = True
    no-xmlrpcs = True
    no-netrpc = True
    ; LOG SETTINGS
    logfile = /var/log/odoo/odoo-server.log
    log_handler = werkzeug:WARNING
    log_level = warn
    no-logrotate = True
    
    EOF

#. Enable Log Rotation using CentOS built in feature ::

    cat > /etc/logrotate.d/odoo-server << EOF
    /var/log/odoo/*.log {
        copytruncate
        missingok
        notifempty
    }
    EOF

#. Create start/stop script ::

    cat > /usr/lib/systemd/system/odoo.service << EOF
    [Unit]
    Description=Odoo Open Source ERP and CRM
    After=network.target postgresql.service
    
    [Service]
    Type=forking
    User=odoo
    Group=odoo
    Environment="ENVDIR=/home/odoo/odoo"
    ExecStart=/bin/bash -c "cd /home/odoo; /bin/virtualenv -q odoo; source odoo/bin/activate; /usr/bin/odoo-server --config=/etc/odoo-server.conf &"
    
    [Install]
    WantedBy=multi-user.target
    
    EOF
    
    ln -s /opt/odoo/openerp-server /usr/bin/odoo-server
    systemctl enable odoo

#. Add firewall rule and start odoo server ::

    firewall-cmd --zone=public --add-port=8069/tcp --permanent
    firewall-cmd --reload
    
    systemctl start odoo

#. Write down your odoo admin password (you'll need it to create odoo database) ::

    echo $ODOO_DB_ADMIN_PASSWORD

At this point you should be able to access the site via ``http://your.ip:8069``. You should log on and change the admin password. 

Configuration
=============

Configuring getfiles
--------------------
To support email reporting, you will need to setup a smtp account. So in the directory of getfiles create a file called settings.json and insert the following with the proper changes.

.. code-block:: JSON

   {
       "smtp":{
           "host": "fqdn.domain.com",
           "port": "465",
           "username": "username@domain.com",
           "password": "Password123"
	     },
       "send":{
           "name": "Monitoring",
           "from": "monitoring@domain.com",
           "to": "destination@domain.com"
	     },
       "logs":{
           "directory": "/tmp/"
	     }
   }

Additionnally you can set the default ftp and smb settings

.. code-block:: JSON

   {
			 "ftp":{
	         "host": "fqdn.domain.com",
	         "port": "465",
	         "username": "username",
	         "password": "Password123"
	     },
	 		"smb":{
	         "host": "fqdn.domain.com",
	         "share": "share1",
	         "destination": "/subdirectory/",
	         "username": "username",
	         "password": "Password123",
					 "version": "1.0"
	     }
   }

Configuring a cron for multiple transfers
-----------------------------------------
To run as a cron, you have multiple options. If you only have one transfer to process, I would suggest you add them directly to your crontab. But if you have multiple transfers to do, I recommend creating a script like this to process them.

.. code-block:: bash

   #!/bin/bash

   # Verify if the file-system is currently in READ-ONLY state
   FILE=/tmp/test.txt;touch ${FILE};if [ -f ${FILE} ]; then
     cd /home/${USER}/bin/getfiles # Location of getfiles
     bash getfiles -vecrd destination@domain.com -f "host1.domain.com 21 username password" -s "host share1 destination username password"
     bash getfiles -vecrd destination@domain.com -f "host2.domain.com 21 username password" -s "host share2 destination username password"
   else
     # Reboot the host if the file-system is READ-ONLY
	   sudo reboot
   fi

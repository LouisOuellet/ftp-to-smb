# ftp-to-smb
FTP to SMB file transfer script. Including email reporting. I wrote this script because I needed a way to transfer files from various ftp accounts to network shares on a Windows network. Since ADDC does not allow to map a ftp drive at logon, this was my next solution so I could deploy those files to many users centrally. My other solution was to go at each desk and setup the FTP on each machine locally.

## Requirements (non-root user)
For the script to work with a non-root user, you will need to add some parameters to your sudoers file.

``` bash
echo "username ALL= NOPASSWD: /bin/umount" | sudo tee -a /etc/sudoers
echo "username ALL= NOPASSWD: /bin/mount" | sudo tee -a /etc/sudoers
echo "username ALL= NOPASSWD: /bin/curlftpfs" | sudo tee -a /etc/sudoers
echo "username ALL= NOPASSWD: /sbin/reboot" | sudo tee -a /etc/sudoers
```

These are required so that the script can mount both SMB and FTP shares as the local user to process the file transfer.

## Configuring getfiles
To support email reporting, you will need to setup a smtp account. So in the directory of getfiles create a file called settings.json and insert the following with the proper changes.

``` json
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
```
Additionnally you can set the default ftp and smb settings

``` json
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
        "password": "Password123"
    }
}
```


## Configuring a cron for multiple transfers
To run as a cron, you have multiple options. If you only have one transfer to process, I would suggest you add them directly to your crontab. But if you have multiple transfers to do, I recommend creating a script like this to process them.

``` bash
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
```
